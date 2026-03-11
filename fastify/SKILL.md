---
name: fastify
description: >
  Fastify web framework expert assistant. Use this skill whenever the user is building HTTP servers
  or APIs with Fastify, registering routes, defining JSON schemas for validation/serialization,
  using hooks (onRequest, preHandler, onSend, etc.), creating plugins, working with decorators,
  handling errors, configuring logging with Pino, setting up TypeScript with Fastify, using
  content type parsers, or asking about the Fastify request lifecycle. Also trigger when code
  imports 'fastify', uses fastify.get/post/route patterns, references FastifyInstance/FastifyRequest/
  FastifyReply types, or mentions encapsulation contexts. Activate for questions about Fastify
  plugins (@fastify/cors, @fastify/auth, etc.), HTTP2 support, middleware migration from Express,
  reply serialization, or schema-based validation with Ajv. This skill covers Fastify v5.8.x.
---

# Fastify Web Framework (v5.8.x)

You are an expert at building high-performance HTTP servers and APIs with Fastify. This skill references the **Fastify v5.8.x** official documentation.

## Core Principles

1. **Performance first** — Fastify is built for speed. Use JSON Schema for validation and serialization to get 10-20% throughput gains via `fast-json-stringify`.
2. **Schema-driven** — Define schemas for request body, querystring, params, headers, and response. Schemas enable validation, serialization, and documentation.
3. **Plugin architecture** — Everything is a plugin. Use `fastify.register()` to extend functionality. Plugins create encapsulation contexts (DAG structure).
4. **Encapsulation** — Decorators, hooks, and plugins are scoped. Child contexts inherit from parents but not vice versa. Use `fastify-plugin` to break encapsulation when needed.
5. **Lifecycle hooks** — The request lifecycle flows: Routing → onRequest → preParsing → Parsing → preValidation → Validation → preHandler → Handler → preSerialization → onSend → onResponse. Each hook point allows injection.

## How to Use This Skill

Before generating code, load the relevant reference file(s):

- `references/server.md` — Factory options, server instance methods (listen, close, register, inject, addSchema, setErrorHandler, setNotFoundHandler, decorators, etc.)
- `references/routes.md` — Route declaration (full and shorthand), URL building, parametric/wildcard routes, async/await patterns, route prefixing, constraints
- `references/request-reply.md` — Request object properties, Reply methods (code, header, send, redirect, hijack, type, serialization)
- `references/hooks.md` — All request/reply hooks (onRequest through onResponse) and application hooks (onReady, onListen, onClose, preClose, onRoute, onRegister), diagnostics channels
- `references/validation-serialization.md` — JSON Schema validation with Ajv, response serialization with fast-json-stringify, shared schemas ($ref), custom validators (Joi, Yup), schemaErrorFormatter
- `references/plugins-encapsulation.md` — Plugin creation, scope management, fastify-plugin, encapsulation contexts, ESM support
- `references/decorators.md` — decorate, decorateRequest, decorateReply, getter/setter patterns, getDecorator, setDecorator, dependency management
- `references/typescript.md` — TypeScript setup, generics, type providers (@fastify/type-provider-typebox, @fastify/type-provider-json-schema-to-ts), creating typed plugins
- `references/logging.md` — Pino logger setup, environment-specific config, serializers, log redaction, custom loggers
- `references/errors.md` — Error handling patterns, setErrorHandler, error codes (FST_ERR_*), error propagation
- `references/content-type-parser.md` — Custom parsers, catch-all, removeContentTypeParser, parseAs options
- `references/http2-middleware.md` — HTTP2 (secure and plain), ALPN negotiation, Express middleware via @fastify/express or @fastify/middie

## Quick Reference

### Creating a Server

```js
import Fastify from 'fastify'

const fastify = Fastify({
  logger: true,           // enable Pino logging
  bodyLimit: 1048576,     // 1 MiB (default)
  caseSensitive: true,    // default: true
  ignoreDuplicateSlashes: false,
  ignoreTrailingSlash: false,
  maxParamLength: 100,
})

// Register plugins
await fastify.register(import('./routes.js'), { prefix: '/api' })

// Start
await fastify.listen({ port: 3000, host: '0.0.0.0' })
```

### Route with Full Schema

```js
fastify.post('/users', {
  schema: {
    body: {
      type: 'object',
      required: ['name', 'email'],
      properties: {
        name: { type: 'string' },
        email: { type: 'string', format: 'email' },
      },
    },
    response: {
      201: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
        },
      },
    },
  },
  handler: async (request, reply) => {
    const user = await createUser(request.body)
    return reply.code(201).send(user)
  },
})
```

### Plugin Pattern

```js
// my-plugin.js — encapsulated by default
export default async function myPlugin(fastify, opts) {
  fastify.decorate('myUtil', () => 'hello')
  fastify.get('/hello', async () => ({ hello: 'world' }))
}

// shared-plugin.js — breaks encapsulation with fastify-plugin
import fp from 'fastify-plugin'
export default fp(async function sharedPlugin(fastify, opts) {
  fastify.decorateRequest('user', null)
}, { name: 'shared-plugin' })
```

### Hooks

```js
// Authentication hook
fastify.addHook('preHandler', async (request, reply) => {
  const token = request.headers.authorization
  if (!token) {
    reply.code(401).send({ error: 'Unauthorized' })
    return reply // must return reply to stop lifecycle
  }
  request.user = await verifyToken(token)
})
```

### Error Handling

```js
fastify.setErrorHandler((error, request, reply) => {
  request.log.error(error)
  const statusCode = error.statusCode >= 400 ? error.statusCode : 500
  reply.status(statusCode).send({
    error: statusCode >= 500 ? 'Internal Server Error' : error.message,
  })
})
```

### TypeScript

```ts
import Fastify, { FastifyRequest, FastifyReply } from 'fastify'

interface IBody { name: string }
interface IParams { id: string }
interface IQuerystring { limit: number }

fastify.get<{ Params: IParams; Querystring: IQuerystring }>(
  '/users/:id',
  async (request, reply) => {
    const { id } = request.params    // typed as string
    const { limit } = request.query  // typed as number
    return { id, limit }
  }
)
```

## Common Patterns

### Decorator Initialization (Avoid Reference Types)

```js
// BAD: reference type shared across requests
fastify.decorateRequest('data', { shared: true })

// GOOD: value type + onRequest hook
fastify.decorateRequest('data', null)
fastify.addHook('onRequest', async (req) => {
  req.data = { perRequest: true }
})
```

### Shared Schemas with $ref

```js
fastify.addSchema({
  $id: 'User',
  type: 'object',
  properties: {
    id: { type: 'integer' },
    name: { type: 'string' },
    email: { type: 'string' },
  },
})

fastify.get('/users', {
  schema: {
    response: {
      200: {
        type: 'array',
        items: { $ref: 'User#' },
      },
    },
  },
  handler: async () => getUsers(),
})
```

### Graceful Shutdown

```js
const signals = ['SIGINT', 'SIGTERM']
for (const signal of signals) {
  process.on(signal, async () => {
    await fastify.close() // triggers preClose → server.close → onClose hooks
    process.exit(0)
  })
}
```

### Async/Await in Handlers

```js
// Return value directly (no reply.send needed)
fastify.get('/', async (request, reply) => {
  const data = await fetchData()
  return data // automatically serialized and sent
})

// When using reply.send(), must return reply
fastify.get('/stream', async (request, reply) => {
  const stream = getStream()
  reply.type('application/octet-stream').send(stream)
  return reply
})
```

## Key Warnings

- **Arrow functions** in handlers break `this` binding to FastifyInstance
- **Never mix callbacks and async** in hooks — causes double execution
- **`reply.send()` in async hooks** must be followed by `return reply`
- **Undeclared decorators** accessing via `req.foo` without `decorateRequest` causes V8 deoptimization
- **Schemas as code**: treat schema definitions as application code — never accept user-provided schemas (uses `new Function()`)
- **Validation with allErrors: true** can enable DoS attacks — Fastify defaults to `false`
