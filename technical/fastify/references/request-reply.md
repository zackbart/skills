# Fastify Request & Reply Reference (v5.8.x)

Complete reference for the Request and Reply objects from the official Fastify documentation.

## Table of Contents

1. [Request Object](#request-object)
2. [Request Properties](#request-properties)
3. [Request Methods](#request-methods)
4. [Reply Object](#reply-object)
5. [Reply Properties](#reply-properties)
6. [Reply Methods](#reply-methods)
7. [send() Behavior](#send-behavior)
8. [Async/Await Patterns](#asyncawait-patterns)

---

## Request Object

The first parameter of the route handler, wrapping the core Node.js request with additional Fastify functionality.

```js
fastify.post('/users/:id', {
  schema: {
    body: {
      type: 'object',
      properties: { name: { type: 'string' } },
    },
    params: {
      type: 'object',
      properties: { id: { type: 'integer' } },
    },
    querystring: {
      type: 'object',
      properties: { fields: { type: 'string' } },
    },
  },
  handler: async (request, reply) => {
    request.log.info('handling request')
    return {
      id: request.params.id,
      body: request.body,
      query: request.query,
    }
  },
})
```

---

## Request Properties

### query

The parsed query string. Validated and coerced if a `querystring` schema is defined.

```js
// GET /search?q=fastify&limit=10
fastify.get('/search', {
  schema: {
    querystring: {
      type: 'object',
      properties: {
        q: { type: 'string' },
        limit: { type: 'integer', default: 20 },
      },
    },
  },
  handler: async (request, reply) => {
    const { q, limit } = request.query
    // q = 'fastify', limit = 10 (coerced to integer)
    return { q, limit }
  },
})
```

### body

The parsed request body. Only available for POST, PUT, PATCH, DELETE, and OPTIONS methods by default.

```js
fastify.post('/items', {
  schema: {
    body: {
      type: 'object',
      required: ['name'],
      properties: {
        name: { type: 'string' },
        price: { type: 'number' },
      },
    },
  },
  handler: async (request, reply) => {
    const { name, price } = request.body
    const item = await createItem({ name, price })
    return reply.code(201).send(item)
  },
})
```

### params

URL parameters parsed from the route path.

```js
// GET /users/42/posts/7
fastify.get('/users/:userId/posts/:postId', async (request, reply) => {
  const { userId, postId } = request.params
  return { userId, postId }
})
```

### headers (getter/setter)

Request headers object. Assigning to `request.headers` adds custom headers that merge with the originals. Schema validation may mutate the headers object. Use `request.raw.headers` to access unmodified original headers.

```js
fastify.addHook('preHandler', async (request, reply) => {
  // Add custom headers (merges with existing)
  request.headers = { 'x-custom-id': generateId() }

  // Read a header
  const contentType = request.headers['content-type']

  // Access original unmodified headers
  const rawHost = request.raw.headers.host
})
```

### raw

The underlying Node.js `http.IncomingMessage` (or `http2.Http2ServerRequest`).

```js
fastify.get('/raw-example', async (request, reply) => {
  const nodeReq = request.raw
  const httpVersion = nodeReq.httpVersion
  const rawHeaders = nodeReq.rawHeaders // alternating key/value array
  return { httpVersion, rawHeaderCount: rawHeaders.length }
})
```

### server

Reference to the Fastify server instance.

```js
fastify.decorateRequest('db', null)

fastify.addHook('onRequest', async (request, reply) => {
  request.db = request.server.db // access server decorators
})

fastify.get('/config', async (request, reply) => {
  const pluginList = request.server.printPlugins()
  return { pluginList }
})
```

### id

The request ID, generated automatically or provided via the `requestIdHeader` server option.

```js
fastify.get('/trace', async (request, reply) => {
  request.log.info({ reqId: request.id }, 'processing request')
  return { requestId: request.id }
})
```

### log

The logger instance for this request, automatically includes the request ID.

```js
fastify.get('/work', async (request, reply) => {
  request.log.info('starting work')
  request.log.debug({ userId: request.query.userId }, 'user context')

  try {
    const result = await doWork()
    request.log.info('work complete')
    return result
  } catch (err) {
    request.log.error(err, 'work failed')
    throw err
  }
})
```

### ip

The IP address of the incoming request. When `trustProxy` is enabled, uses `X-Forwarded-For`.

```js
fastify.get('/whoami', async (request, reply) => {
  return { ip: request.ip }
})
```

### ips

Array of IP addresses from `X-Forwarded-For` header when `trustProxy` is enabled. The client IP is first, each successive proxy appended.

```js
// Requires: Fastify({ trustProxy: true })
fastify.get('/proxy-chain', async (request, reply) => {
  return {
    clientIp: request.ips[0],
    proxyChain: request.ips,
  }
})
```

### host

The host header value. When `trustProxy` is enabled, uses `X-Forwarded-Host`.

```js
fastify.get('/host-info', async (request, reply) => {
  return { host: request.host }
})
```

### hostname

The host without the port number.

```js
fastify.get('/hostname-info', async (request, reply) => {
  return {
    hostname: request.hostname, // e.g. 'example.com'
    host: request.host,         // e.g. 'example.com:3000'
  }
})
```

### port

The port from the host header. When `trustProxy` is enabled, uses `X-Forwarded-Host`.

```js
fastify.get('/port-info', async (request, reply) => {
  return { port: request.port }
})
```

### protocol

The request protocol (`http` or `https`). When `trustProxy` is enabled, uses `X-Forwarded-Proto`.

```js
fastify.get('/proto', async (request, reply) => {
  return { protocol: request.protocol }
})
```

### method

The HTTP method of the request.

```js
fastify.all('/any-method', async (request, reply) => {
  return { method: request.method }
})
```

### url

The request URL (path + query string).

```js
fastify.get('/url-info', async (request, reply) => {
  return { url: request.url }
})
```

### originalUrl

The original URL before any rewriting by Fastify internals or plugins.

```js
fastify.get('/original', async (request, reply) => {
  return {
    url: request.url,
    originalUrl: request.originalUrl,
  }
})
```

### is404

Boolean indicating whether the request was routed through the not-found handler.

```js
fastify.setNotFoundHandler((request, reply) => {
  request.log.warn({ is404: request.is404 }, 'route not found')
  // request.is404 === true
  reply.code(404).send({ error: 'Not Found', path: request.url })
})
```

### socket

The underlying `net.Socket` (or `tls.TLSSocket`) for the connection.

```js
fastify.get('/socket-info', async (request, reply) => {
  const { remoteAddress, remotePort } = request.socket
  return { remoteAddress, remotePort }
})
```

### signal

An `AbortSignal` that aborts when the `handlerTimeout` is reached or the client disconnects. Created lazily on first access.

```js
fastify.get('/long-task', {
  config: { handlerTimeout: 5000 },
  handler: async (request, reply) => {
    // Pass signal to cancellable operations
    const result = await fetch('https://slow-api.example.com/data', {
      signal: request.signal,
    })

    // Or check manually
    if (request.signal.aborted) {
      throw new Error('Request was aborted')
    }

    return result.json()
  },
})

// Listen for abort
fastify.get('/abortable', async (request, reply) => {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => resolve({ done: true }), 3000)

    request.signal.addEventListener('abort', () => {
      clearTimeout(timer)
      reject(new Error('Client disconnected'))
    })
  })
})
```

### routeOptions

The resolved options for the matched route. Read-only.

```js
fastify.get('/info', {
  config: { rateLimit: 100 },
  bodyLimit: 1048576,
  logLevel: 'warn',
  schema: {
    response: {
      200: { type: 'object', properties: { ok: { type: 'boolean' } } },
    },
  },
  handler: async (request, reply) => {
    // Access route configuration inside hooks or handlers
    const opts = request.routeOptions
    return {
      method: opts.method,
      url: opts.url,
      bodyLimit: opts.bodyLimit,
      logLevel: opts.logLevel,
      customConfig: opts.config, // { rateLimit: 100 }
      hasSchema: !!opts.schema,
      handlerTimeout: opts.handlerTimeout,
      attachValidation: opts.attachValidation,
      exposeHeadRoute: opts.exposeHeadRoute,
      prefixTrailingSlash: opts.prefixTrailingSlash,
    }
  },
})
```

**Available routeOptions properties:**

| Property | Description |
|---|---|
| `bodyLimit` | Maximum body size for this route |
| `handlerTimeout` | Timeout for the handler in ms |
| `config` | Custom config object passed in route options |
| `method` | HTTP method(s) |
| `url` | The route pattern |
| `handler` | The route handler function |
| `attachValidation` | Whether to attach validation errors to request |
| `logLevel` | Log level for this route |
| `schema` | The route schema object |
| `version` | Route version constraint |
| `exposeHeadRoute` | Whether HEAD is auto-created for GET routes |
| `prefixTrailingSlash` | Trailing slash handling behavior |

---

## Request Methods

### getValidationFunction(schema | httpPart)

Returns a validation function for a given schema or HTTP part (`body`, `params`, `querystring`, `headers`). The returned function has an `.errors` property containing validation errors after execution.

```js
fastify.post('/validate', async (request, reply) => {
  // Get validator by HTTP part name
  const bodyValidator = request.getValidationFunction('body')
  const isValid = bodyValidator(request.body)
  if (!isValid) {
    return reply.code(400).send({
      errors: bodyValidator.errors,
    })
  }

  // Get validator by schema object
  const customSchema = {
    type: 'object',
    properties: {
      tag: { type: 'string', minLength: 1 },
    },
  }
  const customValidator = request.getValidationFunction(customSchema)
  const tagValid = customValidator({ tag: request.body.tag })
  if (!tagValid) {
    return reply.code(400).send({ errors: customValidator.errors })
  }

  return { valid: true }
})
```

### compileValidationSchema(schema, [httpPart])

Compiles a validation schema and caches it via WeakMap. Subsequent calls with the same schema object reference return the cached function.

```js
const addressSchema = {
  type: 'object',
  required: ['street', 'city'],
  properties: {
    street: { type: 'string' },
    city: { type: 'string' },
    zip: { type: 'string', pattern: '^[0-9]{5}$' },
  },
}

fastify.post('/orders', async (request, reply) => {
  // Compile once, cached on subsequent calls (WeakMap by schema reference)
  const validateAddress = request.compileValidationSchema(addressSchema)
  const isValid = validateAddress(request.body.shippingAddress)

  if (!isValid) {
    return reply.code(400).send({
      error: 'Invalid address',
      details: validateAddress.errors,
    })
  }

  // Compile with httpPart for context-specific compilation
  const validateBody = request.compileValidationSchema(
    addressSchema,
    'body'
  )

  return { addressValid: true }
})
```

### validateInput(data, schema | httpPart, [httpPart])

Validates input data against a schema or the schema of an HTTP part. Returns `true` if valid, or `false` with errors accessible on the validation function.

```js
fastify.post('/submit', {
  schema: {
    body: {
      type: 'object',
      properties: {
        email: { type: 'string', format: 'email' },
        age: { type: 'integer', minimum: 0 },
      },
    },
  },
  handler: async (request, reply) => {
    // Validate against the route's body schema
    const bodyValid = request.validateInput(request.body, 'body')

    // Validate arbitrary data against a custom schema
    const customSchema = {
      type: 'string',
      minLength: 3,
      maxLength: 50,
    }
    const nameValid = request.validateInput(request.body.name, customSchema)

    // Validate with both schema and httpPart context
    const contextValid = request.validateInput(
      request.body,
      customSchema,
      'body'
    )

    if (!bodyValid) {
      return reply.code(400).send({ error: 'Validation failed' })
    }

    return { valid: true }
  },
})
```

---

## Reply Object

The second parameter of the route handler, wrapping the core Node.js response.

---

## Reply Properties

### statusCode

The current HTTP status code. Defaults to `200`.

```js
fastify.get('/status', async (request, reply) => {
  reply.code(201)
  return { statusCode: reply.statusCode } // 201
})
```

### elapsedTime

Time elapsed in milliseconds since the request was received.

```js
fastify.addHook('onResponse', async (request, reply) => {
  request.log.info(
    { elapsed: reply.elapsedTime },
    `${request.method} ${request.url} completed in ${reply.elapsedTime}ms`
  )
})
```

### server

Reference to the Fastify server instance.

```js
fastify.get('/server-ref', async (request, reply) => {
  const routes = reply.server.printRoutes()
  return { routes }
})
```

### raw

The underlying Node.js `http.ServerResponse` (or `http2.Http2ServerResponse`).

```js
fastify.get('/raw-reply', async (request, reply) => {
  // Direct access to the Node.js response — usually not needed
  reply.raw.writeHead(200, { 'content-type': 'text/plain' })
  reply.raw.end('direct response')
  return reply
})
```

### sent

Boolean indicating whether the response has been sent. Once true, further calls to `reply.send()` will throw.

```js
fastify.get('/check-sent', async (request, reply) => {
  reply.send({ step: 1 })
  console.log(reply.sent) // true
  return reply
})
```

### log

Logger instance for this reply/request context.

```js
fastify.get('/log-reply', async (request, reply) => {
  reply.log.info('preparing response')
  return { ok: true }
})
```

### request

Reference back to the request object.

```js
fastify.addHook('onSend', async (request, reply, payload) => {
  reply.log.info({ method: reply.request.method }, 'sending response')
  return payload
})
```

---

## Reply Methods

### code(statusCode) / status(statusCode)

Set the HTTP status code. `.status()` is an alias for `.code()`. Both are chainable.

```js
fastify.post('/items', async (request, reply) => {
  const item = await createItem(request.body)
  return reply.code(201).send(item)
})

fastify.delete('/items/:id', async (request, reply) => {
  await deleteItem(request.params.id)
  return reply.status(204).send()
})
```

### header(key, value)

Set a single response header. For `set-cookie`, values accumulate rather than replace.

```js
fastify.get('/with-headers', async (request, reply) => {
  reply
    .header('x-request-id', request.id)
    .header('cache-control', 'no-cache')

  // set-cookie accumulates
  reply
    .header('set-cookie', 'session=abc; HttpOnly')
    .header('set-cookie', 'theme=dark; Path=/')

  return { ok: true }
})
```

### headers(object)

Set multiple response headers at once.

```js
fastify.get('/multi-headers', async (request, reply) => {
  reply.headers({
    'x-powered-by': 'Fastify',
    'x-request-id': request.id,
    'cache-control': 'public, max-age=3600',
  })
  return { ok: true }
})
```

### getHeader(key)

Get the value of a previously set response header.

```js
fastify.get('/get-header', async (request, reply) => {
  reply.header('x-custom', 'hello')
  const value = reply.getHeader('x-custom') // 'hello'
  return { customHeader: value }
})
```

### getHeaders()

Get all currently set response headers as an object.

```js
fastify.get('/all-headers', async (request, reply) => {
  reply
    .header('x-one', '1')
    .header('x-two', '2')

  const allHeaders = reply.getHeaders()
  // { 'x-one': '1', 'x-two': '2' }
  return { headers: allHeaders }
})
```

### removeHeader(key)

Remove a previously set response header.

```js
fastify.get('/remove-header', async (request, reply) => {
  reply.header('x-temp', 'value')
  reply.removeHeader('x-temp')
  const has = reply.hasHeader('x-temp') // false
  return { removed: !has }
})
```

### hasHeader(key)

Check whether a response header has been set.

```js
fastify.get('/has-header', async (request, reply) => {
  reply.header('x-exists', 'yes')
  return {
    exists: reply.hasHeader('x-exists'),     // true
    missing: reply.hasHeader('x-not-here'),  // false
  }
})
```

### writeEarlyHints(hints, callback)

Send HTTP 103 Early Hints to the client for preloading resources.

```js
fastify.get('/page', async (request, reply) => {
  // Send early hints for resource preloading
  await reply.writeEarlyHints({
    link: '</style.css>; rel=preload; as=style',
  })

  // Multiple links
  await reply.writeEarlyHints({
    link: [
      '</style.css>; rel=preload; as=style',
      '</script.js>; rel=preload; as=script',
    ],
  })

  // With callback style
  reply.writeEarlyHints(
    { link: '</font.woff2>; rel=preload; as=font; crossorigin' },
    () => { /* hints sent */ }
  )

  return { html: '<html>...</html>' }
})
```

### trailer(key, fn)

Register a trailer header. The function receives `reply` and `payload` and must return the trailer value. Only works with chunked transfer encoding.

```js
fastify.get('/with-trailer', async (request, reply) => {
  reply.trailer('x-checksum', (reply, payload) => {
    const hash = require('node:crypto')
      .createHash('md5')
      .update(payload)
      .digest('hex')
    return hash
  })

  reply.trailer('x-timing', (reply, payload) => {
    return `${reply.elapsedTime}ms`
  })

  return { data: 'content with trailers' }
})
```

### hasTrailer(key)

Check whether a trailer has been registered.

```js
fastify.get('/check-trailer', async (request, reply) => {
  reply.trailer('x-checksum', (reply, payload) => 'abc123')
  return {
    hasChecksum: reply.hasTrailer('x-checksum'), // true
    hasTiming: reply.hasTrailer('x-timing'),     // false
  }
})
```

### removeTrailer(key)

Remove a previously registered trailer.

```js
fastify.get('/remove-trailer', async (request, reply) => {
  reply.trailer('x-checksum', (reply, payload) => 'abc')
  reply.removeTrailer('x-checksum')
  return { removed: !reply.hasTrailer('x-checksum') } // true
})
```

### redirect(dest, [code])

Redirect the client. Default status is 302.

```js
fastify.get('/old-path', async (request, reply) => {
  return reply.redirect('/new-path')
  // 302 Found → Location: /new-path
})

fastify.get('/moved', async (request, reply) => {
  return reply.redirect('/permanent-home', 301)
  // 301 Moved Permanently → Location: /permanent-home
})

fastify.get('/external', async (request, reply) => {
  return reply.redirect('https://example.com', 303)
})

// With code set separately
fastify.get('/see-other', async (request, reply) => {
  return reply.code(303).redirect('/other')
})
```

### callNotFound()

Invoke the not-found handler for the current encapsulation context.

```js
fastify.get('/items/:id', async (request, reply) => {
  const item = await getItem(request.params.id)
  if (!item) {
    return reply.callNotFound()
  }
  return item
})
```

### type(contentType)

Set the `Content-Type` header. Shortcut for `reply.header('content-type', contentType)`.

```js
fastify.get('/html', async (request, reply) => {
  reply.type('text/html; charset=utf-8')
  return '<h1>Hello World</h1>'
})

fastify.get('/csv', async (request, reply) => {
  reply.type('text/csv')
  return 'name,age\nAlice,30\nBob,25'
})

fastify.get('/xml', async (request, reply) => {
  reply.type('application/xml')
  return '<root><item>Hello</item></root>'
})
```

### getSerializationFunction(schema | httpStatus, [contentType])

Retrieve a serialization function for a given schema or HTTP status code.

```js
fastify.get('/serialize', {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          name: { type: 'string' },
          secret: { type: 'string' },
        },
      },
    },
  },
  handler: async (request, reply) => {
    // Get the serializer for status 200
    const serialize200 = reply.getSerializationFunction(200)
    // Returns the fast-json-stringify function if a 200 schema exists

    // Get serializer by schema object
    const customSchema = {
      type: 'object',
      properties: { id: { type: 'integer' } },
    }
    const customSerializer = reply.getSerializationFunction(customSchema)

    return { name: 'Alice', secret: 'hidden-by-schema' }
  },
})
```

### compileSerializationSchema(schema, [httpStatus], [contentType])

Compile and cache a serialization schema using `fast-json-stringify`.

```js
const itemSchema = {
  type: 'object',
  properties: {
    id: { type: 'integer' },
    name: { type: 'string' },
  },
}

fastify.get('/items', async (request, reply) => {
  const serialize = reply.compileSerializationSchema(itemSchema)
  const items = await getItems()
  const serialized = serialize(items[0])

  reply.type('application/json')
  reply.send(serialized)
  return reply
})

// With httpStatus context
fastify.get('/items/:id', async (request, reply) => {
  const serialize = reply.compileSerializationSchema(itemSchema, 200)
  const item = await getItem(request.params.id)
  return JSON.parse(serialize(item))
})
```

### serializeInput(data, [schema | httpStatus], [httpStatus], [contentType])

Serialize data using a given schema or the schema for a specific HTTP status code.

```js
fastify.get('/manual-serialize', {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
        },
      },
    },
  },
  handler: async (request, reply) => {
    const data = { id: 1, name: 'Alice', secretField: 'stripped' }

    // Serialize using the 200 response schema
    const json = reply.serializeInput(data, 200)
    // '{"id":1,"name":"Alice"}' — secretField stripped by schema

    // Serialize using a custom schema
    const customSchema = {
      type: 'object',
      properties: { id: { type: 'integer' } },
    }
    const minimal = reply.serializeInput(data, customSchema)
    // '{"id":1}'

    reply.type('application/json').send(json)
    return reply
  },
})
```

### serializer(func)

Set a custom serializer function for this specific reply. Overrides the default JSON serialization.

```js
fastify.get('/custom-serialize', async (request, reply) => {
  reply.serializer((payload) => {
    // Custom serialization logic
    return JSON.stringify(payload, (key, value) => {
      if (key === 'password') return undefined
      return value
    })
  })

  return { name: 'Alice', password: 'secret' }
  // Response: {"name":"Alice"}
})
```

### hijack()

Take over the response, preventing Fastify from sending it. After calling `.hijack()`, all lifecycle hooks after the current one are skipped and the response must be sent manually via `reply.raw`.

```js
fastify.get('/sse', async (request, reply) => {
  reply.hijack()

  const raw = reply.raw
  raw.writeHead(200, {
    'content-type': 'text/event-stream',
    'cache-control': 'no-cache',
    connection: 'keep-alive',
  })

  // Send Server-Sent Events
  let counter = 0
  const interval = setInterval(() => {
    counter++
    raw.write(`data: ${JSON.stringify({ count: counter })}\n\n`)
    if (counter >= 5) {
      clearInterval(interval)
      raw.end()
    }
  }, 1000)

  request.signal.addEventListener('abort', () => {
    clearInterval(interval)
    raw.end()
  })
})
```

### send(data)

Send the response payload. Handles various data types automatically.

```js
// Object — serialized via fast-json-stringify (if schema) or JSON.stringify
fastify.get('/object', async (request, reply) => {
  reply.send({ hello: 'world' })
  return reply
})

// String — sent as text/plain (unless Content-Type already set)
fastify.get('/text', async (request, reply) => {
  reply.send('plain text response')
  return reply
})

// String with Content-Type preset — sent as-is
fastify.get('/html-send', async (request, reply) => {
  reply.type('text/html').send('<h1>Hello</h1>')
  return reply
})

// Buffer — sent as application/octet-stream
fastify.get('/buffer', async (request, reply) => {
  const buf = Buffer.from('binary data')
  reply.send(buf)
  return reply
})

// TypedArray — sent as application/octet-stream
fastify.get('/typed-array', async (request, reply) => {
  const arr = new Uint8Array([72, 101, 108, 108, 111])
  reply.send(arr)
  return reply
})

// Stream — sent as application/octet-stream
import { createReadStream } from 'node:fs'
fastify.get('/stream', async (request, reply) => {
  const stream = createReadStream('/path/to/file')
  reply.send(stream)
  return reply
})

// ReadableStream (Web Streams API) — treated like node streams
fastify.get('/web-stream', async (request, reply) => {
  const stream = new ReadableStream({
    start(controller) {
      controller.enqueue(new TextEncoder().encode('chunk 1'))
      controller.enqueue(new TextEncoder().encode('chunk 2'))
      controller.close()
    },
  })
  reply.send(stream)
  return reply
})

// Response (Web API) — status, headers, and body used directly
fastify.get('/web-response', async (request, reply) => {
  const response = new Response(JSON.stringify({ ok: true }), {
    status: 200,
    headers: { 'content-type': 'application/json' },
  })
  reply.send(response)
  return reply
})

// Error — creates JSON error response
fastify.get('/error', async (request, reply) => {
  const err = new Error('Something went wrong')
  err.statusCode = 503
  reply.send(err)
  return reply
  // Response: { "error": "Service Unavailable", "message": "Something went wrong", "statusCode": 503 }
})
```

### then(fulfilled, rejected)

Makes the reply object thenable/awaitable. Used internally to support returning values from async handlers.

```js
// This is what makes `return reply` work in async handlers.
// You generally don't call .then() directly, but it enables:
fastify.get('/awaitable', async (request, reply) => {
  reply.send({ ok: true })
  await reply // waits for the response to be sent
})
```

---

## send() Behavior

Detailed breakdown of how `reply.send()` handles each data type:

### Objects

Serialized via `fast-json-stringify` when a response schema is defined (faster, strips undeclared properties). Falls back to `JSON.stringify` when no schema exists. Content-Type defaults to `application/json`.

```js
fastify.get('/with-schema', {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
        },
      },
    },
  },
  handler: async (request, reply) => {
    // fast-json-stringify: faster, and `extra` is stripped
    return { id: 1, name: 'Alice', extra: 'removed' }
    // Response: {"id":1,"name":"Alice"}
  },
})

fastify.get('/no-schema', async (request, reply) => {
  // JSON.stringify: all properties included
  return { id: 1, name: 'Alice', extra: 'included' }
  // Response: {"id":1,"name":"Alice","extra":"included"}
})
```

### Strings

Sent as `text/plain; charset=utf-8` unless `Content-Type` has already been set. If Content-Type is `application/json`, the string is assumed to be pre-serialized JSON and sent as-is.

```js
fastify.get('/string-plain', async (request, reply) => {
  return 'Hello World'
  // Content-Type: text/plain; charset=utf-8
})

fastify.get('/string-json', async (request, reply) => {
  reply.type('application/json')
  return '{"pre":"serialized"}'
  // Sent as-is with Content-Type: application/json
})
```

### Streams

Piped to the response. Content-Type defaults to `application/octet-stream`. Errors on the stream are properly handled.

```js
import { createReadStream } from 'node:fs'
import { pipeline } from 'node:stream'

fastify.get('/download', async (request, reply) => {
  const stream = createReadStream('./file.zip')
  reply
    .type('application/zip')
    .header('content-disposition', 'attachment; filename="file.zip"')
    .send(stream)
  return reply
})
```

### Buffers and TypedArrays

Sent directly with `Content-Type: application/octet-stream` unless already set.

```js
fastify.get('/binary', async (request, reply) => {
  const data = Buffer.from([0x89, 0x50, 0x4e, 0x47]) // PNG header
  reply.type('image/png').send(data)
  return reply
})
```

### ReadableStream (Web Streams API)

Treated like Node.js streams, piped to the response.

```js
fastify.get('/web-readable', async (request, reply) => {
  const readable = new ReadableStream({
    async pull(controller) {
      const chunk = await getNextChunk()
      if (chunk) {
        controller.enqueue(new TextEncoder().encode(chunk))
      } else {
        controller.close()
      }
    },
  })
  reply.type('text/plain').send(readable)
  return reply
})
```

### Response (Web API)

The `Response` object's status code, headers, and body are used directly.

```js
fastify.get('/fetch-proxy', async (request, reply) => {
  // Proxy a fetch response directly
  const upstream = await fetch('https://api.example.com/data')
  reply.send(upstream)
  return reply
})
```

### Errors

Creates a JSON response with `{ error, code, message, statusCode }`. If `statusCode` on the error is less than 400, the response uses 500 instead. Triggers the error handler if one is set.

```js
fastify.get('/not-found', async (request, reply) => {
  const err = new Error('Item not found')
  err.statusCode = 404
  throw err
  // Response: { "statusCode": 404, "error": "Not Found", "message": "Item not found" }
})

fastify.get('/with-code', async (request, reply) => {
  const err = new Error('Rate limit exceeded')
  err.statusCode = 429
  err.code = 'RATE_LIMIT'
  throw err
  // Response: { "statusCode": 429, "code": "RATE_LIMIT", "error": "Too Many Requests", "message": "Rate limit exceeded" }
})

// statusCode < 400 gets promoted to 500
fastify.get('/bad-status', async (request, reply) => {
  const err = new Error('Oops')
  err.statusCode = 200 // invalid for an error
  throw err
  // Response uses statusCode: 500
})
```

---

## Async/Await Patterns

### Return value directly

The simplest pattern. The return value is serialized and sent automatically.

```js
fastify.get('/', async (request, reply) => {
  const data = await db.query('SELECT * FROM users')
  return data
})

fastify.get('/user/:id', async (request, reply) => {
  const user = await db.findUser(request.params.id)
  if (!user) {
    const err = new Error('User not found')
    err.statusCode = 404
    throw err
  }
  return user
})
```

### Throw errors

Thrown errors are caught by Fastify and routed to the error handler.

```js
fastify.get('/protected', async (request, reply) => {
  if (!request.headers.authorization) {
    const err = new Error('Missing authorization header')
    err.statusCode = 401
    throw err
  }

  const user = await verifyToken(request.headers.authorization)
  return { user }
})
```

### reply.send() with return reply

When using `reply.send()` in an async handler, always `return reply` to prevent double sends.

```js
import { createReadStream } from 'node:fs'

fastify.get('/file', async (request, reply) => {
  const stream = createReadStream('./data.csv')
  reply.type('text/csv').send(stream)
  return reply
})

fastify.get('/conditional', async (request, reply) => {
  if (request.query.redirect) {
    reply.redirect('/other')
    return reply
  }
  return { ok: true }
})

// Setting headers and using send
fastify.get('/headers-and-send', async (request, reply) => {
  reply
    .code(201)
    .header('x-custom', 'value')
    .send({ created: true })
  return reply
})
```

### Warning: do not mix patterns

```js
// BAD: returning a value after reply.send() causes ERR_FASTIFY_REPLY_ALREADY_SENT
fastify.get('/bad', async (request, reply) => {
  reply.send({ hello: 'world' })
  return { another: 'value' } // ERROR — reply already sent
})

// BAD: forgetting return reply after send
fastify.get('/also-bad', async (request, reply) => {
  reply.send({ hello: 'world' })
  // Implicit return undefined — Fastify tries to send undefined too
})

// GOOD: always return reply after using reply.send()
fastify.get('/good', async (request, reply) => {
  reply.send({ hello: 'world' })
  return reply
})

// GOOD: just return the value
fastify.get('/also-good', async (request, reply) => {
  return { hello: 'world' }
})
```
