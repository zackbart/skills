# Error Handling

Fastify automatically catches both synchronous and asynchronous errors thrown in route handlers and converts them to a `500 Internal Server Error` response.

## Automatic Error Catching

```js
// Sync throw — caught automatically
fastify.get('/sync-error', (request, reply) => {
  throw new Error('Something broke')
  // Response: 500 {"statusCode":500,"error":"Internal Server Error","message":"Something broke"}
})

// Async rejection — caught automatically
fastify.get('/async-error', async (request, reply) => {
  throw new Error('Async failure')
  // Response: 500 {"statusCode":500,"error":"Internal Server Error","message":"Async failure"}
})

// Promise rejection — caught automatically
fastify.get('/promise-error', (request, reply) => {
  return Promise.reject(new Error('Promise failed'))
})
```

### Always Throw Error Instances

Always throw `Error` objects, never primitives. Throwing a string or number produces unhelpful stack traces and breaks error handler expectations:

```js
// WRONG — no stack trace, hard to debug
throw 'something failed'
throw 404
throw { message: 'not found' }

// CORRECT
throw new Error('something failed')

// For HTTP errors, use Fastify's built-in createError or set statusCode
const error = new Error('Not found')
error.statusCode = 404
throw error
```

## Custom Error Handler

### setErrorHandler

Register a custom error handler for the current encapsulation scope:

```js
fastify.setErrorHandler(function (error, request, reply) {
  request.log.error({ err: error }, 'Error occurred')

  // Set status code from the error, default to 500
  const statusCode = error.statusCode || 500

  reply.status(statusCode).send({
    statusCode,
    error: error.name,
    message: error.message
  })
})
```

The error handler receives three arguments:
- `error` — the Error object
- `request` — the Fastify Request object
- `reply` — the Fastify Reply object

### reply.send(data) in Error Handler

`reply.send(data)` in an error handler works exactly like in a route handler. You can send JSON, strings, streams, or buffers:

```js
fastify.setErrorHandler(function (error, request, reply) {
  if (error.statusCode === 404) {
    reply.status(404).send({ error: 'Resource not found' })
    return
  }

  // In production, hide internal error details
  reply.status(500).send({
    statusCode: 500,
    message: process.env.NODE_ENV === 'production'
      ? 'Internal Server Error'
      : error.message
  })
})
```

### Encapsulated Error Handlers

Error handlers are scoped to their encapsulation context:

```js
// Root error handler
fastify.setErrorHandler(function (error, request, reply) {
  reply.status(500).send({ error: 'Root handler caught this' })
})

fastify.register(async function apiRoutes(fastify) {
  // API-specific error handler
  fastify.setErrorHandler(function (error, request, reply) {
    reply.status(error.statusCode || 500).send({
      error: error.message,
      code: error.code
    })
  })

  fastify.get('/fail', async () => {
    throw new Error('API error')
    // Handled by apiRoutes error handler, NOT root
  })
})

fastify.get('/root-fail', async () => {
  throw new Error('Root error')
  // Handled by root error handler
})
```

## Error Propagation

### Child to Parent Error Handler

If you throw inside an error handler, the error propagates to the parent context's error handler:

```js
fastify.setErrorHandler(function (error, request, reply) {
  // Parent catches errors thrown in child error handlers
  request.log.error('Parent error handler caught:', error.message)
  reply.status(500).send({ error: 'Caught by parent' })
})

fastify.register(async function child(fastify) {
  fastify.setErrorHandler(function (error, request, reply) {
    // Re-throw to propagate to parent
    if (error.statusCode >= 500) {
      throw error  // Goes to parent error handler
    }
    reply.status(error.statusCode).send({ error: error.message })
  })
})
```

If the root error handler throws, Fastify sends a generic `500` response.

## onError Hook

The `onError` hook runs **before** `setErrorHandler`. It is for observation/logging and **cannot change the response**. The error is still passed to the error handler afterward.

```js
fastify.addHook('onError', async (request, reply, error) => {
  // Log, track metrics, report to Sentry, etc.
  request.log.error({ err: error }, 'onError hook')
  // Cannot modify the reply here — it continues to setErrorHandler
})
```

Execution order when an error occurs:
1. `onError` hook(s) execute
2. `setErrorHandler` executes
3. Response is sent to client

## Validation Errors

When schema validation fails, Fastify calls the error handler with special properties on the error object.

### error.validation

An array of validation error objects from Ajv:

```js
fastify.setErrorHandler(function (error, request, reply) {
  if (error.validation) {
    // This is a validation error
    reply.status(400).send({
      statusCode: 400,
      error: 'Validation Error',
      message: error.message,
      details: error.validation
      // [{ keyword: 'required', params: { missingProperty: 'name' }, message: "must have required property 'name'" }]
    })
    return
  }

  // Other errors
  reply.status(error.statusCode || 500).send({
    statusCode: error.statusCode || 500,
    message: error.message
  })
})
```

### error.validationContext

Indicates where validation failed. One of: `body`, `params`, `querystring`, `headers`, `response`.

```js
fastify.setErrorHandler(function (error, request, reply) {
  if (error.validation) {
    const context = error.validationContext  // 'body', 'params', etc.
    reply.status(400).send({
      statusCode: 400,
      error: `Validation failed in ${context}`,
      details: error.validation.map(v => ({
        field: v.instancePath || v.params?.missingProperty,
        message: v.message
      }))
    })
    return
  }

  reply.status(error.statusCode || 500).send({ message: error.message })
})
```

## setNotFoundHandler

Register a custom 404 handler. Supports `preHandler` and `preValidation` hooks:

```js
fastify.setNotFoundHandler({
  preHandler: async (request, reply) => {
    // Runs before the not-found handler
    // Useful for authentication checks on 404 routes
  }
}, function (request, reply) {
  reply.status(404).send({
    statusCode: 404,
    error: 'Not Found',
    message: `Route ${request.method}:${request.url} not found`
  })
})
```

### Scoped Not Found Handlers

```js
fastify.register(async function apiRoutes(fastify) {
  fastify.setNotFoundHandler(function (request, reply) {
    reply.status(404).send({
      statusCode: 404,
      error: 'API endpoint not found',
      path: request.url
    })
  })
}, { prefix: '/api' })
```

## Error Codes (errorCodes)

Import Fastify error codes for programmatic error checking:

```js
import { errorCodes } from 'fastify'

fastify.setErrorHandler(function (error, request, reply) {
  if (error instanceof errorCodes.FST_ERR_BAD_STATUS_CODE) {
    // Handle specific error type
  }
})
```

### 10 Most Common Error Codes

| Error Code | Description |
|-----------|-------------|
| `FST_ERR_BAD_STATUS_CODE` | Status code is not valid (not an integer, or outside 100–599) |
| `FST_ERR_DEC_ALREADY_PRESENT` | A decorator with the same name is already registered in this scope |
| `FST_ERR_DEC_MISSING_DEPENDENCY` | A decorator dependency is missing — register the dependency first |
| `FST_ERR_DEC_UNDECLARED` | Attempted to `getDecorator`/`setDecorator` for a name that was never declared |
| `FST_ERR_HOOK_INVALID_HANDLER` | Hook handler is not a function |
| `FST_ERR_CTP_ALREADY_PRESENT` | Content type parser already registered for this content type |
| `FST_ERR_CTP_INVALID_TYPE` | Content type is not a string |
| `FST_ERR_SCH_VALIDATION_BUILD` | Schema failed to compile (invalid JSON Schema) |
| `FST_ERR_VALIDATION` | Request validation failed against the route schema |
| `FST_ERR_REP_ALREADY_SENT` | Reply was already sent — trying to send again |

### Creating Custom Errors with Fastify's createError

```js
import createError from '@fastify/error'

const NotFoundError = createError('ERR_NOT_FOUND', 'Resource %s not found', 404)
const UnauthorizedError = createError('ERR_UNAUTHORIZED', 'Authentication required', 401)
const ForbiddenError = createError('ERR_FORBIDDEN', '%s is not allowed', 403)

fastify.get('/users/:id', async (request) => {
  const user = await findUser(request.params.id)
  if (!user) {
    throw new NotFoundError(request.params.id)
    // Response: 404 {"statusCode":404,"code":"ERR_NOT_FOUND","message":"Resource 42 not found"}
  }
  return user
})
```

`@fastify/error` creates error constructors with:
- `statusCode` — HTTP status code
- `code` — machine-readable error code
- `message` — supports `%s` and `%d` format placeholders

## Complete Error Handling Example

```js
import Fastify from 'fastify'
import createError from '@fastify/error'

const AppError = createError('APP_ERROR', '%s', 500)
const NotFoundError = createError('NOT_FOUND', '%s not found', 404)
const ValidationError = createError('VALIDATION_ERROR', 'Invalid input: %s', 400)

const fastify = Fastify({ logger: true })

// Global onError hook for metrics/monitoring
fastify.addHook('onError', async (request, reply, error) => {
  // Report to error tracking service
  request.log.error({
    err: error,
    url: request.url,
    method: request.method
  }, 'Request error')
})

// Root error handler
fastify.setErrorHandler(function (error, request, reply) {
  // Validation errors
  if (error.validation) {
    reply.status(400).send({
      statusCode: 400,
      code: 'VALIDATION_ERROR',
      message: 'Request validation failed',
      details: error.validation,
      validationContext: error.validationContext
    })
    return
  }

  // Known application errors
  const statusCode = error.statusCode ?? 500

  reply.status(statusCode).send({
    statusCode,
    code: error.code ?? 'INTERNAL_ERROR',
    message: statusCode < 500 ? error.message : 'Internal Server Error'
  })
})

// Custom 404 handler
fastify.setNotFoundHandler(function (request, reply) {
  reply.status(404).send({
    statusCode: 404,
    code: 'ROUTE_NOT_FOUND',
    message: `${request.method} ${request.url} does not exist`
  })
})

await fastify.listen({ port: 3000 })
```
