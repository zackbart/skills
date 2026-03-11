# Decorators

Decorators extend the Fastify server instance, Request, or Reply objects with custom properties and methods. They are the primary mechanism for sharing functionality across your application.

## Server Instance Decorators

### decorate(name, value, [dependencies])

Adds a property or method to the Fastify server instance:

```js
// Simple value
fastify.decorate('config', { db: 'postgres://localhost/mydb' })

// Method
fastify.decorate('utility', function () {
  // 'this' is the Fastify instance
  return this.config.db
})

// Async method
fastify.decorate('fetchData', async function (id) {
  return await this.db.query('SELECT * FROM items WHERE id = $1', [id])
})
```

Access anywhere in the same encapsulation scope:

```js
fastify.get('/info', async function (request, reply) {
  return { db: this.config.db }
})
```

## Request Decorators

### decorateRequest(name, value, [dependencies])

Adds a property to the Request object. **Cannot use reference types (objects, arrays, functions) as the initial value** because the same reference would be shared across all requests, causing cross-request data leaks.

```js
// CORRECT — use null as initial value for objects
fastify.decorateRequest('user', null)

// CORRECT — primitive types are fine
fastify.decorateRequest('requestStartTime', 0)
fastify.decorateRequest('isAuthenticated', false)
fastify.decorateRequest('tenantId', '')

// WRONG — reference type shared across ALL requests
fastify.decorateRequest('user', { id: null, name: null })  // BUG: shared object!
fastify.decorateRequest('tags', [])                         // BUG: shared array!
```

### Proper Initialization with onRequest Hook

For per-request objects, initialize in a hook:

```js
fastify.decorateRequest('user', null)

fastify.addHook('onRequest', async (request, reply) => {
  // Each request gets its own object
  request.user = { id: null, name: null, roles: [] }
})
```

### Getter/Setter Pattern

Use getter/setter for computed or lazily-initialized values:

```js
fastify.decorateRequest('userAgent', {
  getter() {
    return this.headers['user-agent'] || 'unknown'
  }
})

// Access in route — computed on demand
fastify.get('/info', async (request) => {
  return { ua: request.userAgent }
})
```

The getter function receives no arguments. `this` is the Request object. This pattern avoids storing a value on every request when it may not be needed.

### setDecorator for Request

`setDecorator` provides runtime safety when setting request decorator values. It validates the decorator exists and the value is compatible:

```js
fastify.decorateRequest('session', null)

fastify.addHook('onRequest', async (request) => {
  request.setDecorator('session', await loadSession(request))
})
```

If the decorator was never declared, `setDecorator` throws `FST_ERR_DEC_UNDECLARED` instead of silently adding a property (which would bypass V8 optimization).

## Reply Decorators

### decorateReply(name, value, [dependencies])

Same rules as request decorators — **no reference types as initial values**.

```js
// CORRECT
fastify.decorateReply('startTime', 0)
fastify.decorateReply('metadata', null)

// WRONG
fastify.decorateReply('metadata', { cached: false })  // BUG: shared reference

// Initialize per-request in a hook
fastify.addHook('onRequest', async (request, reply) => {
  reply.metadata = { cached: false, servedAt: Date.now() }
})
```

## Checking for Decorators

### hasDecorator / hasRequestDecorator / hasReplyDecorator

Check if a decorator exists in the current encapsulation context:

```js
if (fastify.hasDecorator('db')) {
  // Server-level decorator exists
}

if (fastify.hasRequestDecorator('user')) {
  // Request-level decorator exists
}

if (fastify.hasReplyDecorator('startTime')) {
  // Reply-level decorator exists
}
```

Useful in plugins to check if a dependency has been registered:

```js
async function myPlugin(fastify, opts) {
  if (!fastify.hasDecorator('db')) {
    throw new Error('This plugin requires the db decorator. Register the db plugin first.')
  }

  fastify.get('/items', async function () {
    return this.db.query('SELECT * FROM items')
  })
}
```

## Getting Decorators

### getDecorator(name)

Retrieves the current value of a server decorator. Throws `FST_ERR_DEC_UNDECLARED` if the decorator does not exist.

```js
fastify.decorate('config', { port: 3000 })

const config = fastify.getDecorator('config')
// { port: 3000 }

// Throws FST_ERR_DEC_UNDECLARED
fastify.getDecorator('nonexistent')
```

### setDecorator(name, value)

Updates the value of an existing server decorator. Throws `FST_ERR_DEC_UNDECLARED` if the decorator does not exist.

```js
fastify.decorate('requestCount', 0)

fastify.addHook('onRequest', async () => {
  const current = fastify.getDecorator('requestCount')
  fastify.setDecorator('requestCount', current + 1)
})
```

## Dependencies Parameter

The third argument to any `decorate*` method is an array of decorator names that must exist before this decorator can be registered:

```js
fastify.decorate('db', pgPool)

// 'cache' depends on 'db' existing first
fastify.decorate('cache', new Cache(fastify.db), ['db'])

// Multiple dependencies
fastify.decorate('service', new Service(), ['db', 'cache'])
```

If a dependency is missing, Fastify throws `FST_ERR_DEC_MISSING_DEPENDENCY` with a message listing which decorators are missing.

```js
// Throws: FST_ERR_DEC_MISSING_DEPENDENCY: The decorator is missing dependency 'db'.
fastify.decorate('service', new Service(), ['db'])
```

## Encapsulation Rules

### Same Name in Different Contexts — OK

Each encapsulated scope gets its own decorator namespace:

```js
fastify.register(async function pluginA(instance) {
  instance.decorate('name', 'Plugin A')
  console.log(instance.name) // 'Plugin A'
})

fastify.register(async function pluginB(instance) {
  instance.decorate('name', 'Plugin B')  // No conflict — different scope
  console.log(instance.name) // 'Plugin B'
})
```

### Same Name in Same Context — Throws

Registering the same decorator name twice in the same scope throws `FST_ERR_DEC_ALREADY_PRESENT`:

```js
fastify.decorate('db', pool1)
fastify.decorate('db', pool2) // Throws FST_ERR_DEC_ALREADY_PRESENT
```

### Child Overriding Parent

A child context can shadow a parent decorator by decorating with the same name:

```js
fastify.decorate('config', { env: 'production' })

fastify.register(async function child(instance) {
  // Shadows the parent 'config' within this scope only
  instance.decorate('config', { env: 'test' })
  console.log(instance.config.env) // 'test'
})

console.log(fastify.config.env) // 'production' — parent is unaffected
```

## Arrow Functions and `this` Binding

Arrow functions do NOT have their own `this`, so they cannot access the Fastify instance via `this`:

```js
// WRONG — 'this' is not the Fastify instance
fastify.decorate('broken', () => {
  return this.config // 'this' is the outer scope, not Fastify
})

// CORRECT — use regular function
fastify.decorate('working', function () {
  return this.config // 'this' is the Fastify instance
})
```

Same applies in route handlers:

```js
// WRONG — arrow function loses 'this'
fastify.get('/info', async () => {
  return this.config // undefined or wrong 'this'
})

// CORRECT
fastify.get('/info', async function () {
  return this.config
})

// ALSO OK — use the request/reply params directly instead of 'this'
fastify.get('/info', async (request, reply) => {
  return request.server.config
})
```

## V8 Optimization

Fastify decorators are designed to work with V8's hidden classes optimization. Defining the "shape" of objects before instantiation allows V8 to optimize property access.

### Why Initial Values Matter

V8 creates hidden classes (internal type descriptors) for objects. When all instances of an object have the same set of properties added in the same order, V8 can optimize property access to near-array-index speed.

```js
// GOOD — shape defined upfront, V8 optimizes access
fastify.decorateRequest('user', null)    // Shape: { user: null }
fastify.decorateRequest('session', null) // Shape: { user: null, session: null }

// BAD — adding properties dynamically in handlers
fastify.addHook('onRequest', async (request) => {
  request.customProp = 'value'  // V8 sees a new shape, deoptimizes
})
```

### Initial Value Guidelines

| Type | Initial Value | Why |
|------|--------------|-----|
| String | `''` | Primitive, safe to share |
| Number | `0` | Primitive, safe to share |
| Boolean | `false` | Primitive, safe to share |
| Object | `null` | Reference type — initialize in hook |
| Array | `null` | Reference type — initialize in hook |
| Function | `null` | Reference type — initialize in hook |

### Complete Pattern

```js
// 1. Declare shape with safe initial values
fastify.decorateRequest('user', null)
fastify.decorateRequest('requestId', '')
fastify.decorateRequest('startTime', 0)
fastify.decorateRequest('isAdmin', false)

// 2. Initialize reference types per-request
fastify.addHook('onRequest', async (request, reply) => {
  request.startTime = Date.now()
  request.user = { id: null, name: null, roles: [] }
})

// 3. Use in routes
fastify.get('/profile', async (request) => {
  return {
    user: request.user,
    processingTime: Date.now() - request.startTime
  }
})
```
