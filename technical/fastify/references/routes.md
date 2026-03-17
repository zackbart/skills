# Fastify v5 Routes Reference

Comprehensive routes reference for Fastify v5.8.x. All examples from official documentation.

## Table of Contents

1. [Full Declaration](#full-declaration)
2. [Route Options](#route-options)
3. [Shorthand Declaration](#shorthand-declaration)
4. [URL Building](#url-building)
5. [Async/Await Patterns](#asyncawait-patterns)
6. [Promise Resolution Rules](#promise-resolution-rules)
7. [Route Prefixing](#route-prefixing)
8. [Config](#config)
9. [Constraints](#constraints)
10. [Custom Log Level and Serializers](#custom-log-level-and-serializers)

---

## Full Declaration

```js
fastify.route({
  method: 'GET',
  url: '/users/:id',
  schema: {
    querystring: {
      type: 'object',
      properties: {
        fields: { type: 'string' }
      }
    },
    params: {
      type: 'object',
      properties: {
        id: { type: 'string' }
      },
      required: ['id']
    },
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'string' },
          name: { type: 'string' },
          email: { type: 'string' }
        }
      }
    }
  },
  handler: async (request, reply) => {
    const { id } = request.params
    return { id, name: 'Jane', email: 'jane@example.com' }
  }
})
```

---

## Route Options

Complete list of all route options:

### Core Options

| Option | Type | Description |
|---|---|---|
| `method` | `String \| Array` | HTTP method(s). Supported: `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `PATCH`, `SEARCH`, `TRACE`, `PROPFIND`, `PROPPATCH`, `MKCOL`, `COPY`, `MOVE`, `LOCK`, `UNLOCK` |
| `url` / `path` | `String` | Route path. `url` and `path` are aliases |
| `handler` | `Function` | `(request, reply) => {}` — handles the request |
| `exposeHeadRoute` | `Boolean` | Auto-creates a HEAD route for every GET route. Default: `true` on the server, configurable per route |
| `bodyLimit` | `Number` | Max body size in bytes for this route. Overrides server-level `bodyLimit` |
| `handlerTimeout` | `Number` | Route handler timeout in milliseconds |
| `logLevel` | `String` | Log level for this route (`'info'`, `'warn'`, `'error'`, `'fatal'`, `'trace'`, `'debug'`, `'silent'`) |
| `logSerializers` | `Object` | Custom log serializers scoped to this route |
| `config` | `Object` | Custom configuration object accessible via `request.routeOptions.config` |
| `version` | `String` | Semver version constraint (matched via `Accept-Version` header) |
| `constraints` | `Object` | Route constraints (version, host, custom) |
| `prefixTrailingSlash` | `String` | Controls trailing slash behavior with prefixes: `'both'` (default), `'slash'`, `'no-slash'` |

### Schema Options

| Option | Type | Description |
|---|---|---|
| `schema.body` | `Object` | Validates request body (POST/PUT/PATCH) |
| `schema.querystring` / `schema.query` | `Object` | Validates query string parameters. `querystring` and `query` are aliases |
| `schema.params` | `Object` | Validates URL parameters |
| `schema.response` | `Object` | Validates and serializes response by status code |
| `schema.headers` | `Object` | Validates request headers |

```js
fastify.route({
  method: 'POST',
  url: '/users',
  schema: {
    body: {
      type: 'object',
      required: ['name', 'email'],
      properties: {
        name: { type: 'string' },
        email: { type: 'string', format: 'email' }
      }
    },
    querystring: {
      type: 'object',
      properties: {
        notify: { type: 'boolean' }
      }
    },
    params: {
      type: 'object',
      properties: {
        orgId: { type: 'string' }
      }
    },
    headers: {
      type: 'object',
      properties: {
        'x-request-id': { type: 'string' }
      }
    },
    response: {
      201: {
        type: 'object',
        properties: {
          id: { type: 'string' },
          name: { type: 'string' }
        }
      },
      400: {
        type: 'object',
        properties: {
          message: { type: 'string' }
        }
      }
    }
  },
  handler: async (request, reply) => {
    const user = await createUser(request.body)
    reply.code(201)
    return user
  }
})
```

### Error Handling Options

| Option | Type | Description |
|---|---|---|
| `errorHandler` | `Function` | Custom error handler scoped to this route |
| `attachValidation` | `Boolean` | When `true`, validation errors attach to `request.validationError` instead of returning 400 |
| `schemaErrorFormatter` | `Function` | Custom formatter for schema validation errors |

```js
fastify.route({
  method: 'POST',
  url: '/data',
  attachValidation: true,
  schema: {
    body: {
      type: 'object',
      required: ['value'],
      properties: {
        value: { type: 'number' }
      }
    }
  },
  handler: async (request, reply) => {
    if (request.validationError) {
      // Handle validation error manually
      return { error: request.validationError.message }
    }
    return { received: request.body.value }
  }
})

// Custom schemaErrorFormatter
fastify.route({
  method: 'POST',
  url: '/items',
  schemaErrorFormatter: (errors, dataVar) => {
    return new Error(`Validation failed for ${dataVar}: ${errors.map(e => e.message).join(', ')}`)
  },
  schema: {
    body: {
      type: 'object',
      required: ['name'],
      properties: {
        name: { type: 'string' }
      }
    }
  },
  handler: async (request, reply) => {
    return { ok: true }
  }
})
```

### Logging Options

| Option | Type | Description |
|---|---|---|
| `childLoggerFactory` | `Function` | Custom factory for the child logger bound to the request |

```js
fastify.route({
  method: 'GET',
  url: '/resource',
  childLoggerFactory: function (logger, bindings, opts, rawReq) {
    const child = logger.child(bindings, opts)
    child.customProperty = 'foo'
    return child
  },
  handler: async (request, reply) => {
    request.log.info('Using custom child logger')
    return { ok: true }
  }
})
```

### Lifecycle Hook Options

All lifecycle hooks can be set per-route. Each accepts a function or array of functions.

| Option | Phase | Signature |
|---|---|---|
| `onRequest` | Before parsing | `(request, reply, done)` or `async (request, reply)` |
| `preParsing` | Before body parsing | `(request, reply, payload, done)` or `async (request, reply, payload)` |
| `preValidation` | Before schema validation | `(request, reply, done)` or `async (request, reply)` |
| `preHandler` | Before handler | `(request, reply, done)` or `async (request, reply)` |
| `preSerialization` | Before response serialization | `(request, reply, payload, done)` or `async (request, reply, payload)` |
| `onSend` | Before sending response | `(request, reply, payload, done)` or `async (request, reply, payload)` |
| `onResponse` | After response sent | `(request, reply, done)` or `async (request, reply)` |
| `onTimeout` | On request timeout | `(request, reply, done)` or `async (request, reply)` |
| `onError` | On error | `(request, reply, error, done)` or `async (request, reply, error)` |

```js
fastify.route({
  method: 'GET',
  url: '/protected',
  onRequest: async (request, reply) => {
    const token = request.headers['authorization']
    if (!token) {
      reply.code(401)
      throw new Error('Unauthorized')
    }
  },
  preHandler: [
    async (request, reply) => {
      // First hook
      request.user = await validateToken(request.headers['authorization'])
    },
    async (request, reply) => {
      // Second hook — runs after first
      await checkPermissions(request.user)
    }
  ],
  preSerialization: async (request, reply, payload) => {
    // Modify payload before serialization; must return modified payload
    return { ...payload, timestamp: Date.now() }
  },
  onSend: async (request, reply, payload) => {
    // Modify serialized payload (string); must return modified payload
    return payload
  },
  onResponse: async (request, reply) => {
    // Fires after response is sent — no further reply possible
    request.log.info(`Response sent in ${reply.elapsedTime}ms`)
  },
  onTimeout: async (request, reply) => {
    request.log.warn('Request timed out')
  },
  onError: async (request, reply, error) => {
    request.log.error({ err: error }, 'Route error occurred')
  },
  handler: async (request, reply) => {
    return { user: request.user }
  }
})
```

### Compiler Options

| Option | Type | Description |
|---|---|---|
| `validatorCompiler` | `Function` | Custom validator compiler for this route |
| `serializerCompiler` | `Function` | Custom serializer compiler for this route |

```js
fastify.route({
  method: 'POST',
  url: '/custom-validation',
  validatorCompiler: ({ schema, method, url, httpPart }) => {
    return (data) => {
      // Return { value } on success or { error } on failure
      if (data.name && typeof data.name === 'string') {
        return { value: data }
      }
      return { error: new Error('name is required and must be a string') }
    }
  },
  serializerCompiler: ({ schema, method, url, httpStatus, contentType }) => {
    return (data) => JSON.stringify(data)
  },
  schema: {
    body: {
      type: 'object',
      properties: {
        name: { type: 'string' }
      }
    },
    response: {
      200: {
        type: 'object',
        properties: {
          ok: { type: 'boolean' }
        }
      }
    }
  },
  handler: async (request, reply) => {
    return { ok: true }
  }
})
```

---

## Shorthand Declaration

All shorthand methods follow the same pattern:

```js
fastify.get(path, [options], handler)
fastify.post(path, [options], handler)
fastify.put(path, [options], handler)
fastify.delete(path, [options], handler)
fastify.patch(path, [options], handler)
fastify.options(path, [options], handler)
fastify.head(path, [options], handler)
fastify.all(path, [options], handler)  // Matches all HTTP methods
```

### Without options:

```js
fastify.get('/ping', async (request, reply) => {
  return { pong: true }
})
```

### With options:

```js
fastify.post('/users', {
  schema: {
    body: {
      type: 'object',
      required: ['name'],
      properties: {
        name: { type: 'string' }
      }
    },
    response: {
      201: {
        type: 'object',
        properties: {
          id: { type: 'string' },
          name: { type: 'string' }
        }
      }
    }
  },
  preHandler: async (request, reply) => {
    await authenticate(request)
  }
}, async (request, reply) => {
  const user = await createUser(request.body)
  reply.code(201)
  return user
})
```

### Handler in options object:

```js
fastify.get('/items', {
  schema: {
    response: {
      200: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            id: { type: 'number' },
            name: { type: 'string' }
          }
        }
      }
    }
  },
  handler: async (request, reply) => {
    return getItems()
  }
})
```

---

## URL Building

### Parametric Routes

```js
// Single parameter
fastify.get('/users/:id', async (request, reply) => {
  const { id } = request.params  // { id: 'abc123' }
  return { id }
})

// Multiple parameters in one segment using dash separator
fastify.get('/near/:lat-:lng/radius/:r', async (request, reply) => {
  // GET /near/15.3-40.2/radius/5
  const { lat, lng, r } = request.params  // { lat: '15.3', lng: '40.2', r: '5' }
  return { lat, lng, r }
})
```

### Wildcard Routes

```js
// Matches /files/any/path/here
fastify.get('/files/*', async (request, reply) => {
  const path = request.params['*']  // 'any/path/here'
  return { path }
})
```

### RegExp Routes

```js
// Only matches numeric file names with .png extension
fastify.get('/images/:file(^\\d+).png', async (request, reply) => {
  // GET /images/123.png → { file: '123' }
  // GET /images/abc.png → 404
  return { file: request.params.file }
})
```

### Optional Parameters

```js
// Matches both /posts and /posts/123
fastify.get('/posts/:id?', async (request, reply) => {
  const { id } = request.params
  if (id) {
    return getPost(id)
  }
  return getAllPosts()
})
```

### Double Colon Escape

Use `::` to produce a literal `:` in the URL (not treated as a parameter):

```js
// Registers route: /name:verb
fastify.get('/name::verb', async (request, reply) => {
  // Accessible at GET /name:verb
  return { ok: true }
})
```

---

## Async/Await Patterns

### Return value directly (auto-sent)

```js
fastify.get('/data', async (request, reply) => {
  const data = await fetchData()
  return data  // Automatically sent as response
})
```

### Using reply.send() in async — MUST return reply

```js
fastify.get('/data', async (request, reply) => {
  const data = await fetchData()
  reply.send(data)
  return reply  // REQUIRED — must return reply when using reply.send() in async
})
```

### Awaiting reply for callback-based APIs

```js
fastify.get('/stream', async (request, reply) => {
  const stream = fs.createReadStream('file.txt')
  reply.type('text/plain').send(stream)
  await reply  // Wait for the response to be sent
})
```

### WARNING: return value takes precedence over reply.send()

```js
// BAD — will cause FST_ERR_PROMISE_NOT_FULFILLED
fastify.get('/broken', async (request, reply) => {
  reply.send({ via: 'send' })
  return { via: 'return' }  // This takes precedence, reply.send is ignored
})
```

### WARNING: undefined cannot be returned

```js
// BAD — undefined return causes an error
fastify.get('/bad', async (request, reply) => {
  // Implicit undefined return — will throw
})

// GOOD — always return a value or use reply.send() + return reply
fastify.get('/good', async (request, reply) => {
  reply.code(204)
  return ''  // Return empty string for no-content
})
```

---

## Promise Resolution Rules

### Rule 1: Using reply.send()

When using `reply.send()` in an async handler, you MUST return `reply` or `await reply`:

```js
// CORRECT
fastify.get('/a', async (request, reply) => {
  reply.send({ hello: 'world' })
  return reply
})

// CORRECT
fastify.get('/b', async (request, reply) => {
  reply.send({ hello: 'world' })
  await reply
})
```

### Rule 2: Using return value

When returning a value directly, DO NOT also call `reply.send()`:

```js
// CORRECT
fastify.get('/c', async (request, reply) => {
  return { hello: 'world' }
})

// WRONG — do not mix return value with reply.send()
fastify.get('/d', async (request, reply) => {
  reply.send({ hello: 'world' })
  return { hello: 'world' }  // Conflict!
})
```

---

## Route Prefixing

### Basic prefix with register

```js
// routes/v1/users.js
async function userRoutes(fastify, options) {
  // This becomes GET /v1/users
  fastify.get('/users', async (request, reply) => {
    return getUsers()
  })

  // This becomes GET /v1/users/:id
  fastify.get('/users/:id', async (request, reply) => {
    return getUser(request.params.id)
  })
}

module.exports = userRoutes

// app.js
fastify.register(require('./routes/v1/users'), { prefix: '/v1' })
fastify.register(require('./routes/v2/users'), { prefix: '/v2' })
```

### Nested prefixes

```js
// Prefixes are concatenated
fastify.register(function (api, opts, done) {
  // prefix: /api

  api.register(function (v1, opts, done) {
    // prefix: /api/v1

    v1.get('/users', async () => {
      // GET /api/v1/users
      return getUsers()
    })
    done()
  }, { prefix: '/v1' })

  done()
}, { prefix: '/api' })
```

### fastify-plugin workaround

When using `fastify-plugin` (which breaks encapsulation), the prefix from the parent register is preserved:

```js
const fp = require('fastify-plugin')

// This plugin shares the parent context but still gets the prefix
const routes = fp(async function (fastify, opts) {
  fastify.get('/hello', async () => {
    return { hello: 'world' }
  })
})

// GET /api/hello
fastify.register(routes, { prefix: '/api' })
```

### Root route behavior with prefixes

```js
async function routes(fastify, opts) {
  // With prefix '/api', this becomes GET /api and GET /api/
  fastify.get('/', async () => {
    return { root: true }
  })
}

fastify.register(routes, { prefix: '/api' })
```

### prefixTrailingSlash option

Controls how trailing slashes are handled when a prefix is applied:

```js
// 'both' (default) — registers both /prefix and /prefix/
fastify.register(function (childA, opts, done) {
  childA.get('/', { prefixTrailingSlash: 'both' }, async () => {
    return { slash: 'both' }
  })
  // Matches: /api and /api/
  done()
}, { prefix: '/api' })

// 'slash' — only registers /prefix/
fastify.register(function (childB, opts, done) {
  childB.get('/', { prefixTrailingSlash: 'slash' }, async () => {
    return { slash: 'only' }
  })
  // Matches: /api/ only
  done()
}, { prefix: '/api' })

// 'no-slash' — only registers /prefix
fastify.register(function (childC, opts, done) {
  childC.get('/', { prefixTrailingSlash: 'no-slash' }, async () => {
    return { slash: 'none' }
  })
  // Matches: /api only
  done()
}, { prefix: '/api' })
```

---

## Config

Attach custom configuration to a route, accessible in hooks and handlers:

```js
fastify.get('/en', {
  config: {
    output: 'hello!'
  }
}, async (request, reply) => {
  // Access via reply.routeOptions.config
  return reply.routeOptions.config.output  // 'hello!'
})

fastify.get('/it', {
  config: {
    output: 'ciao!'
  }
}, async (request, reply) => {
  return reply.routeOptions.config.output  // 'ciao!'
})
```

### Accessing config in hooks

```js
fastify.addHook('onRequest', async (request, reply) => {
  const routeConfig = reply.routeOptions.config
  request.log.info({ config: routeConfig }, 'Route config')
})
```

### Config is merged with internal Fastify config

```js
fastify.get('/info', {
  config: {
    greeting: 'hello'
  }
}, async (request, reply) => {
  const config = reply.routeOptions.config
  // config.greeting === 'hello'
  // config.url, config.method also available (set by Fastify internally)
  return config
})
```

---

## Constraints

### Version Constraints

Uses the `Accept-Version` header with semver matching:

```js
fastify.route({
  method: 'GET',
  url: '/resource',
  constraints: { version: '1.0.0' },
  handler: async (request, reply) => {
    return { version: '1.0.0', data: 'original' }
  }
})

fastify.route({
  method: 'GET',
  url: '/resource',
  constraints: { version: '2.0.0' },
  handler: async (request, reply) => {
    return { version: '2.0.0', data: 'updated' }
  }
})
// GET /resource with Accept-Version: 1.x → version 1 handler
// GET /resource with Accept-Version: 2.x → version 2 handler
```

> **WARNING**: When using version constraints, set the `Vary` response header to include `Accept-Version`. This prevents cache poisoning where a cached v1 response is served for a v2 request.

```js
fastify.addHook('onSend', async (request, reply) => {
  if (request.headers['accept-version']) {
    reply.header('Vary', 'Accept-Version')
  }
})
```

### Host Constraints

Match routes to specific hosts (string or RegExp):

```js
fastify.route({
  method: 'GET',
  url: '/',
  constraints: { host: 'api.example.com' },
  handler: async (request, reply) => {
    return { host: 'api' }
  }
})

fastify.route({
  method: 'GET',
  url: '/',
  constraints: { host: 'admin.example.com' },
  handler: async (request, reply) => {
    return { host: 'admin' }
  }
})

// RegExp host constraint
fastify.route({
  method: 'GET',
  url: '/',
  constraints: { host: /.*\.example\.com/ },
  handler: async (request, reply) => {
    return { host: 'wildcard' }
  }
})
```

### Async Custom Constraints

Define custom constraints with async `deriveConstraint`:

```js
const Fastify = require('fastify')

const fastify = Fastify({
  constraints: {
    tenantId: {
      // Name of the constraint
      name: 'tenantId',
      // Storage strategy
      storage: function () {
        const map = new Map()
        return {
          get: (key) => map.get(key),
          set: (key, store) => map.set(key, store),
          del: (key) => map.delete(key),
          empty: () => map.clear()
        }
      },
      // Async constraint derivation from the request
      deriveConstraint: async (req) => {
        const token = req.headers['authorization']
        const tenant = await lookupTenant(token)
        return tenant.id
      },
      // Whether constraint must match a registered value
      mustMatchWhenDerived: true
    }
  }
})

fastify.route({
  method: 'GET',
  url: '/data',
  constraints: { tenantId: 'tenant-a' },
  handler: async (request, reply) => {
    return { tenant: 'a', data: [] }
  }
})

fastify.route({
  method: 'GET',
  url: '/data',
  constraints: { tenantId: 'tenant-b' },
  handler: async (request, reply) => {
    return { tenant: 'b', data: [] }
  }
})
```

> **WARNING**: Async custom constraints with `deriveConstraint` have a performance impact. The async function is called for every request that matches the URL, even before the constraint is checked. Use synchronous constraints when possible.

---

## Custom Log Level and Serializers

### Per-route log level

```js
// Only logs warnings and above for this route
fastify.get('/health', {
  logLevel: 'warn'
}, async (request, reply) => {
  request.log.info('This will NOT be logged')
  request.log.warn('This WILL be logged')
  return { status: 'ok' }
})

// Verbose logging for debugging a specific route
fastify.get('/debug-endpoint', {
  logLevel: 'trace'
}, async (request, reply) => {
  request.log.trace('Detailed trace info')
  return { debug: true }
})

// Silence logging entirely for noisy routes
fastify.get('/metrics', {
  logLevel: 'silent'
}, async (request, reply) => {
  return getMetrics()
})
```

### Per-route log serializers

Custom serializers override the default serializers for a specific route:

```js
fastify.get('/user/:id', {
  logSerializers: {
    req (request) {
      return {
        method: request.method,
        url: request.url,
        userId: request.params.id
      }
    },
    res (reply) {
      return {
        statusCode: reply.statusCode
      }
    }
  }
}, async (request, reply) => {
  request.log.info({ req: request }, 'User request')
  const user = await getUser(request.params.id)
  return user
})
```

### Combining log level and serializers

```js
fastify.route({
  method: 'POST',
  url: '/orders',
  logLevel: 'info',
  logSerializers: {
    req (request) {
      return {
        method: request.method,
        url: request.url,
        bodyKeys: Object.keys(request.body || {}),
        userAgent: request.headers['user-agent']
      }
    }
  },
  handler: async (request, reply) => {
    request.log.info({ req: request }, 'New order request')
    const order = await createOrder(request.body)
    reply.code(201)
    return order
  }
})
```
