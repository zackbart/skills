# HTTP/2 & Middleware

## HTTP/2 Support

Fastify has native HTTP/2 support via Node.js's built-in `http2` module.

### HTTPS with HTTP/2

The standard production setup uses HTTP/2 over TLS (h2):

```js
import Fastify from 'fastify'
import { readFileSync } from 'fs'

const fastify = Fastify({
  http2: true,
  https: {
    key: readFileSync('./certs/server.key'),
    cert: readFileSync('./certs/server.crt'),
    // Optional: CA certificate for mutual TLS
    // ca: readFileSync('./certs/ca.crt')
  },
  logger: true
})

fastify.get('/', async () => {
  return { protocol: 'h2', secure: true }
})

await fastify.listen({ port: 3000 })
// Access: https://localhost:3000
```

### ALPN Negotiation with allowHTTP1

Allow both HTTP/1.1 and HTTP/2 clients on the same port. The protocol is negotiated via TLS ALPN (Application-Layer Protocol Negotiation):

```js
const fastify = Fastify({
  http2: true,
  https: {
    allowHTTP1: true,  // Fallback to HTTP/1.1 for older clients
    key: readFileSync('./certs/server.key'),
    cert: readFileSync('./certs/server.crt')
  }
})

fastify.get('/', async (request) => {
  return {
    protocol: request.raw.httpVersion  // '2.0' or '1.1'
  }
})
```

This is the recommended approach for public-facing services — HTTP/2-capable clients use h2, while older clients fall back to HTTP/1.1 transparently.

### Plain Text HTTP/2 (h2c)

For internal microservices behind a reverse proxy, you can use unencrypted HTTP/2 (h2c). No TLS certificates needed:

```js
const fastify = Fastify({
  http2: true,
  // No https option — plain text HTTP/2
  logger: true
})

fastify.get('/', async () => {
  return { protocol: 'h2c', secure: false }
})

await fastify.listen({ port: 3000 })
```

**Warning**: h2c is for trusted networks only (service mesh, internal APIs). Browsers do not support h2c — they require TLS for HTTP/2.

### Testing HTTP/2

Use `npx h2url` to test HTTP/2 endpoints from the command line:

```bash
# Test HTTPS HTTP/2
npx h2url https://localhost:3000/

# Test plain text h2c
npx h2url http://localhost:3000/
```

### HTTP/2 Server Push

Fastify supports HTTP/2 server push via the raw `stream`:

```js
fastify.get('/', async (request, reply) => {
  // Push a resource before the client requests it
  const stream = reply.raw.stream
  if (stream && stream.pushAllowed) {
    stream.pushStream({ ':path': '/styles.css' }, (err, pushStream) => {
      if (!err) {
        pushStream.respond({ ':status': 200, 'content-type': 'text/css' })
        pushStream.end('body { color: red; }')
      }
    })
  }

  reply.type('text/html')
  return '<html><link rel="stylesheet" href="/styles.css"><body>Hello</body></html>'
})
```

### HTTP/2 Route Constraints

You can constrain routes to specific HTTP/2 sessions or settings if needed, but in most cases routes work identically between HTTP/1.1 and HTTP/2.

## Middleware

### Not Supported Out of the Box

Since Fastify v3, Express-style middleware (`(req, res, next)`) is **not supported** natively. Fastify uses its own hook system instead. To use Express or Connect middleware, install an adapter plugin.

### @fastify/express — Full Express Compatibility

Provides full Express middleware and routing compatibility:

```js
import Fastify from 'fastify'
import fastifyExpress from '@fastify/express'
import cors from 'cors'
import helmet from 'helmet'

const fastify = Fastify({ logger: true })

await fastify.register(fastifyExpress)

// Now you can use Express middleware
fastify.use(cors({ origin: true }))
fastify.use(helmet())

// Express middleware still works alongside Fastify routes
fastify.get('/', async () => {
  return { hello: 'world' }
})

await fastify.listen({ port: 3000 })
```

### @fastify/middie — Simpler Middleware Support

A lighter alternative when you only need basic middleware support (no Express routing):

```js
import Fastify from 'fastify'
import middie from '@fastify/middie'

const fastify = Fastify({ logger: true })

await fastify.register(middie)

// Use Connect-style middleware
fastify.use(function (req, res, next) {
  req.customProperty = 'hello'
  next()
})

fastify.get('/', async (request) => {
  return { value: request.raw.customProperty }
})

await fastify.listen({ port: 3000 })
```

### Middleware Encapsulation via register

Middleware registered inside a `register` scope only applies to routes in that scope:

```js
fastify.register(async function apiScope(fastify) {
  await fastify.register(middie)

  // This middleware ONLY applies to routes in apiScope
  fastify.use(function (req, res, next) {
    req.apiContext = true
    next()
  })

  fastify.get('/data', async (request) => {
    return { apiContext: request.raw.apiContext }  // true
  })
}, { prefix: '/api' })

// Routes outside do NOT have the middleware
fastify.get('/', async (request) => {
  return { apiContext: request.raw.apiContext }  // undefined
})
```

### No Access to Fastify Reply in Middleware

Middleware receives the **raw Node.js** `req` (IncomingMessage) and `res` (ServerResponse), NOT Fastify's Request and Reply objects:

```js
fastify.use(function (req, res, next) {
  // req is Node's IncomingMessage — NO request.body, request.params, etc.
  // res is Node's ServerResponse — NO reply.send(), reply.code(), etc.

  console.log(req.url)        // Works — raw Node.js property
  console.log(req.body)       // undefined — not available in middleware
  console.log(req.params)     // undefined — not available in middleware

  next()
})
```

To access Fastify's Request/Reply, use Fastify hooks instead:

```js
fastify.addHook('onRequest', async (request, reply) => {
  // request is Fastify Request — has .body, .params, .query, etc.
  // reply is Fastify Reply — has .send(), .code(), .header(), etc.
  request.log.info('Full Fastify access here')
})
```

### Path-Based Middleware Restriction

Restrict middleware to specific paths:

```js
await fastify.register(middie)

// Only runs for requests starting with /api
fastify.use('/api', function (req, res, next) {
  req.apiMiddlewareApplied = true
  next()
})

// Only runs for /admin paths
fastify.use('/admin', function (req, res, next) {
  // Check admin auth
  if (!req.headers['x-admin-token']) {
    res.statusCode = 403
    res.end(JSON.stringify({ error: 'Forbidden' }))
    return
  }
  next()
})
```

## Preferred Fastify Alternatives to Middleware

Instead of using Express middleware through adapters, prefer native Fastify plugins which are faster and fully integrated:

| Express Middleware | Fastify Plugin | Install |
|-------------------|---------------|---------|
| `helmet` | `@fastify/helmet` | `npm i @fastify/helmet` |
| `cors` | `@fastify/cors` | `npm i @fastify/cors` |
| `serve-static` | `@fastify/static` | `npm i @fastify/static` |
| `compression` | `@fastify/compress` | `npm i @fastify/compress` |
| `cookie-parser` | `@fastify/cookie` | `npm i @fastify/cookie` |
| `express-session` | `@fastify/session` | `npm i @fastify/session` |
| `multer` | `@fastify/multipart` | `npm i @fastify/multipart` |
| `express-rate-limit` | `@fastify/rate-limit` | `npm i @fastify/rate-limit` |

### Example: Native Fastify Plugins Instead of Middleware

```js
import Fastify from 'fastify'
import cors from '@fastify/cors'
import helmet from '@fastify/helmet'
import rateLimit from '@fastify/rate-limit'
import fastifyStatic from '@fastify/static'
import { join } from 'path'

const fastify = Fastify({ logger: true })

// CORS
await fastify.register(cors, {
  origin: ['https://example.com', 'https://app.example.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true
})

// Security headers
await fastify.register(helmet, {
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"]
    }
  }
})

// Rate limiting
await fastify.register(rateLimit, {
  max: 100,
  timeWindow: '1 minute'
})

// Static file serving
await fastify.register(fastifyStatic, {
  root: join(import.meta.dirname, 'public'),
  prefix: '/public/'
})

fastify.get('/', async () => {
  return { hello: 'world' }
})

await fastify.listen({ port: 3000 })
```

These native plugins are faster than Express middleware adapters because they skip the compatibility layer and integrate directly with Fastify's request/reply lifecycle.
