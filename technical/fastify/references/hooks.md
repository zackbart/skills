# Fastify v5.8.x Hooks Reference

Hooks let you intercept specific points in the request/response lifecycle or application lifecycle. Register hooks with `fastify.addHook()`.

## Table of Contents

1. [Request/Reply Hooks (Execution Order)](#requestreply-hooks-execution-order)
   1. [onRequest](#onrequest)
   2. [preParsing](#preparsing)
   3. [preValidation](#prevalidation)
   4. [preHandler](#prehandler)
   5. [preSerialization](#preserialization)
   6. [onError](#onerror)
   7. [onSend](#onsend)
   8. [onResponse](#onresponse)
   9. [onTimeout](#ontimeout)
   10. [onRequestAbort](#onrequestabort)
2. [Error Handling from Hooks](#error-handling-from-hooks)
3. [Responding from a Hook](#responding-from-a-hook)
4. [Application Hooks](#application-hooks)
   1. [onReady](#onready)
   2. [onListen](#onlisten)
   3. [onClose](#onclose)
   4. [preClose](#preclose)
   5. [onRoute](#onroute)
   6. [onRegister](#onregister)
5. [Scope and Encapsulation](#scope-and-encapsulation)
6. [Route-Level Hooks](#route-level-hooks)
7. [Diagnostics Channel Hooks](#diagnostics-channel-hooks)
   1. [Initialization Event](#initialization-event)
   2. [Request TracingChannel Events](#request-tracingchannel-events)
8. [Common Patterns](#common-patterns)
   1. [Authentication Hook](#authentication-hook)
   2. [Request Logging with Timing](#request-logging-with-timing)
   3. [Payload Decompression in preParsing](#payload-decompression-in-preparsing)

---

## Request/Reply Hooks (Execution Order)

Hooks execute in this order for each request:

```
onRequest -> preParsing -> preValidation -> preHandler -> [handler]
  -> preSerialization -> onSend -> onResponse
```

If an error occurs: `onError` runs before the error handler, then `onSend -> onResponse`.

---

### onRequest

```js
fastify.addHook('onRequest', async (request, reply) => {
  // request.body is ALWAYS undefined here — body has not been parsed yet
  // Useful for authentication, rate limiting, request decoration
})
```

- First hook in the lifecycle after routing
- `request.body` is always `undefined` at this point
- Can modify `request` properties (e.g., add custom headers, set decorations)

### preParsing

```js
fastify.addHook('preParsing', async (request, reply, payload) => {
  // payload is the raw request stream
  // Must return a stream if modifying
  const newPayload = new PassThrough()
  pump(payload, transform, newPayload)

  // Add receivedEncodedLength so Fastify can check bodyLimit correctly
  newPayload.receivedEncodedLength = payload.receivedEncodedLength || 0

  return newPayload
})
```

- `payload` is the incoming request stream
- If you return a new stream, you **must** add a `receivedEncodedLength` property to it
- The size of the returned stream is checked against `bodyLimit`
- If you do not modify the payload, do not return anything

### preValidation

```js
fastify.addHook('preValidation', async (request, reply) => {
  // Body has been parsed — you can modify it before schema validation
  request.body = { ...request.body, importantKey: 'value' }
})
```

- Runs after body parsing but before schema validation
- Last chance to modify `request.body` before validation runs
- Useful for injecting default values or normalizing data

### preHandler

```js
fastify.addHook('preHandler', async (request, reply) => {
  // Body is parsed and validated
  // Runs just before the route handler
  // Good for authorization checks, loading resources
})
```

- Runs after validation succeeds, immediately before the route handler
- `request.body`, `request.query`, and `request.params` are all validated at this point

### preSerialization

```js
fastify.addHook('preSerialization', async (request, reply, payload) => {
  // Modify the response payload before it is serialized
  return { wrapped: payload, timestamp: Date.now() }
})
```

- Called before the response payload is serialized to JSON
- **NOT called** if the payload is a `string`, `Buffer`, `stream`, or `null`
- Must return the modified payload
- If you do not modify the payload, do not return anything

### onError

```js
fastify.addHook('onError', async (request, reply, error) => {
  // For custom error logging/tracking only
  // You CANNOT change the error from here
  // You CANNOT call reply.send() from here
  this.log.error({ err: error, req: request }, 'request errored')
})
```

- Runs **before** `setErrorHandler`
- Intended for custom error logging, metrics, or error tracking services
- You **cannot** change the error or send a response from this hook
- If you need to modify the error response, use `setErrorHandler` instead

### onSend

```js
fastify.addHook('onSend', async (request, reply, payload) => {
  // payload is the serialized response body (string)
  // Can replace the payload — must return string, Buffer, stream,
  // ReadableStream, Response, or null
  const newPayload = payload.replace('sensitive', '***')
  return newPayload
})
```

- Runs after serialization, before the response is sent
- `payload` is the serialized body (usually a string)
- Returned value **must** be `string`, `Buffer`, `stream`, `ReadableStream`, `Response`, or `null`
- To remove the body, return `null` and set `reply.code(204)` with appropriate content-length

### onResponse

```js
fastify.addHook('onResponse', async (request, reply) => {
  // Response has already been sent to the client
  // Useful for metrics, logging, cleanup
  const responseTime = reply.elapsedTime
  this.log.info({ url: request.url, statusCode: reply.statusCode, responseTime }, 'request completed')
})
```

- The response has **already been sent** — you cannot modify it
- Use for metrics collection, access logging, or resource cleanup

### onTimeout

```js
fastify.addHook('onTimeout', async (request, reply) => {
  // The request socket has timed out (connectionTimeout)
  this.log.warn({ url: request.url }, 'request timed out')
})
```

- Triggered by socket-level timeout (`connectionTimeout` server option)
- For **application-level** timeouts, use `handlerTimeout` (route option) combined with `request.signal` (an `AbortSignal`)

```js
// Application-level timeout pattern
fastify.route({
  method: 'GET',
  url: '/slow',
  config: { handlerTimeout: 5000 },
  handler: async (request, reply) => {
    const result = await longOperation({ signal: request.signal })
    return result
  }
})
```

### onRequestAbort

```js
fastify.addHook('onRequestAbort', async (request) => {
  // Client closed the connection before the full request was processed
  // Useful for aborting expensive operations, cleaning up resources
  this.log.warn({ url: request.url }, 'client aborted request')
})
```

- Fires when the client closes the connection before the request is fully processed
- Only receives `request` (no `reply` parameter)
- Use to cancel in-progress database queries, file operations, etc.

---

## Error Handling from Hooks

### Async hooks — throw the error

```js
fastify.addHook('preHandler', async (request, reply) => {
  if (!request.headers.authorization) {
    const err = new Error('Unauthorized')
    err.statusCode = 401
    throw err
  }
})
```

### Callback hooks — pass error to done()

```js
fastify.addHook('preHandler', (request, reply, done) => {
  if (!request.headers.authorization) {
    reply.code(401)
    done(new Error('Unauthorized'))
    return
  }
  done()
})
```

- Use `reply.code(statusCode)` before `done(error)` to set a custom HTTP status code
- Setting `error.statusCode` also works for async hooks

---

## Responding from a Hook

To send a response early (short-circuit the lifecycle):

```js
fastify.addHook('onRequest', async (request, reply) => {
  // Call reply.send() then return reply
  reply.code(304)
  reply.send('')
  return reply  // REQUIRED — signals Fastify to stop the lifecycle
})
```

- You **must** call `reply.send()` and then `return reply`
- The hook chain stops — remaining hooks and the route handler are skipped
- **NEVER** mix callback style (`done()`) with async/Promise — pick one

```js
// WRONG — do not mix
fastify.addHook('onRequest', async (request, reply, done) => {
  done() // NEVER do this in an async function
})
```

---

## Application Hooks

### onReady

```js
fastify.addHook('onReady', async () => {
  // All plugins loaded, routes registered
  // Server is NOT yet listening
  // Good for: warm caches, verify connections, seed data
  await warmUpCache()
})
```

- Fires after all plugins are loaded but before the server starts listening
- Cannot access `request` or `reply`

### onListen

```js
fastify.addHook('onListen', async () => {
  // Server is now listening for connections
  // NOT called when using fastify.inject() or fastify.ready()
  console.log(`Server listening on ${fastify.server.address().port}`)
})
```

- Only fires when the server actually binds to a port
- Not triggered by `inject()` or `ready()` (test scenarios)

### onClose

```js
fastify.addHook('onClose', async (instance) => {
  // HTTP server has stopped, connections are drained
  // Safe to clean up: close DB pools, flush logs, etc.
  await instance.db.close()
})
```

- Called after the HTTP server stops and all connections are drained
- **Child plugin hooks run before parent hooks** (bottom-up teardown)
- Safe for final cleanup: close database pools, flush buffers, release resources

### preClose

```js
fastify.addHook('preClose', async () => {
  // Server is rejecting new requests with 503 but still listening
  // Use for graceful shutdown of WebSocket connections, SSE streams
  for (const ws of activeWebSockets) {
    ws.close(1001, 'Server shutting down')
  }
})
```

- Server responds with `503 Service Unavailable` to new requests but is still listening
- Use to gracefully shut down long-lived connections (WebSockets, SSE, streaming)
- Runs before `onClose`

### onRoute

```js
fastify.addHook('onRoute', (routeOptions) => {
  // Sync hook — called when a route is registered
  // routeOptions is mutable — you can modify the route config
  console.log(`Registered: ${routeOptions.method} ${routeOptions.url}`)

  // Example: add a shared preHandler to all routes
  if (!routeOptions.preHandler) {
    routeOptions.preHandler = []
  }
  routeOptions.preHandler.push(sharedAuthHook)
})
```

- **Synchronous** — no `done` callback or async
- Called once per route registration, not per request
- `routeOptions` is mutable — you can modify the route before it is finalized
- Encapsulated — only sees routes registered in the same plugin scope

### onRegister

```js
fastify.addHook('onRegister', (instance, opts) => {
  // Called when a new plugin is registered
  // Runs BEFORE the plugin code executes
  // instance is the new encapsulated context
  instance.decorate('pluginMeta', { registeredAt: Date.now() })
})
```

- Fires when a plugin is registered, before the plugin code runs
- `instance` is the new encapsulated Fastify context for that plugin
- `opts` are the options passed to `fastify.register(plugin, opts)`

---

## Scope and Encapsulation

- **All hooks except `onClose` are encapsulated** — they only apply to routes registered in the same plugin scope (and child scopes)
- `onClose` runs for all registered instances regardless of scope

### `this` Context

```js
fastify.addHook('onRequest', function (request, reply, done) {
  // `this` is the Fastify context where the ROUTE was registered
  this.log.info('hook running')
  done()
})
```

- `this` is bound to the Fastify instance of the plugin where the **route** was registered
- **Arrow functions break `this` binding** — use regular functions if you need `this`

```js
// Arrow function — `this` is NOT the Fastify instance
fastify.addHook('onRequest', async (request, reply) => {
  // `this` is the surrounding lexical scope, NOT Fastify
})
```

---

## Route-Level Hooks

Hooks can be set directly on individual routes:

```js
fastify.route({
  method: 'GET',
  url: '/protected',
  onRequest: async (request, reply) => {
    // Route-specific onRequest
    await verifyApiKey(request)
  },
  preValidation: async (request, reply) => {
    // Route-specific preValidation
  },
  preHandler: [checkAuth, loadUser],  // Can be an array of functions
  preSerialization: async (request, reply, payload) => {
    return { data: payload }
  },
  handler: async (request, reply) => {
    return { user: request.user }
  }
})
```

- Route-level hooks **always execute after** shared (application-level) hooks
- Can be a single function or an array of functions
- Available route-level hooks: `onRequest`, `preParsing`, `preValidation`, `preHandler`, `preSerialization`, `onSend`, `onResponse`, `onError`, `onTimeout`, `onRequestAbort`

### Shorthand syntax

```js
fastify.get('/admin', {
  onRequest: [rateLimit, authenticate],
  preHandler: authorize
}, async (request, reply) => {
  return { admin: true }
})
```

---

## Diagnostics Channel Hooks

Fastify publishes events via Node.js `diagnostics_channel` for observability tooling.

### Initialization Event

```js
import diagnostics_channel from 'node:diagnostics_channel'

diagnostics_channel.subscribe('fastify.initialization', ({ fastify }) => {
  // Fastify instance created
})
```

### Request TracingChannel Events

Per-request tracing events follow the `TracingChannel` pattern:

```js
import { tracingChannel } from 'node:diagnostics_channel'

const requestChannel = tracingChannel('fastify.request.handler')

// Each event receives: { request, reply, route: { url, method } }

requestChannel.subscribe({
  start({ request, reply, route }) {
    // Handler is about to execute
    console.log(`${route.method} ${route.url} starting`)
  },
  end({ request, reply, route }) {
    // Handler completed synchronously
  },
  asyncStart({ request, reply, route }) {
    // Async handler resumed
  },
  asyncEnd({ request, reply, route }) {
    // Async handler completed
  },
  error({ request, reply, route, error }) {
    // Handler threw an error
    console.error(`${route.method} ${route.url} error:`, error)
  }
})
```

Channel names:
- `tracing:fastify.request.handler:start`
- `tracing:fastify.request.handler:end`
- `tracing:fastify.request.handler:asyncStart`
- `tracing:fastify.request.handler:asyncEnd`
- `tracing:fastify.request.handler:error`

---

## Common Patterns

### Authentication Hook

```js
async function authenticate(request, reply) {
  try {
    const token = request.headers.authorization?.replace('Bearer ', '')
    if (!token) throw new Error('Missing token')
    request.user = await verifyJwt(token)
  } catch (err) {
    reply.code(401).send({ error: 'Unauthorized' })
    return reply
  }
}

fastify.addHook('onRequest', authenticate)
```

### Request Logging with Timing

```js
fastify.addHook('onRequest', async (request) => {
  request.startTime = process.hrtime.bigint()
})

fastify.addHook('onResponse', async (request, reply) => {
  const duration = Number(process.hrtime.bigint() - request.startTime) / 1e6
  request.log.info({ duration, statusCode: reply.statusCode }, 'request completed')
})
```

### Payload Decompression in preParsing

```js
import { createGunzip } from 'node:zlib'

fastify.addHook('preParsing', async (request, reply, payload) => {
  if (request.headers['content-encoding'] === 'gzip') {
    const gunzip = createGunzip()
    gunzip.receivedEncodedLength = payload.receivedEncodedLength || 0
    payload.pipe(gunzip)
    return gunzip
  }
})
```
