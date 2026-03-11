# Plugins & Encapsulation

## Registering Plugins

`fastify.register(plugin, [options])` creates a new encapsulated scope. Every call to `register` creates a child context that inherits from the parent but cannot modify it.

```js
fastify.register(import('./routes/users.js'))
fastify.register(import('./routes/posts.js'))
```

### Plugin Function Signature

A plugin is a function that receives the Fastify instance and options:

```js
// Callback pattern
function myPlugin(fastify, opts, done) {
  // Add routes, decorators, hooks, etc.
  done()
}

// Async pattern (preferred)
async function myPlugin(fastify, opts) {
  // No done callback needed
}
```

### ESM Support

Fastify supports ESM plugins natively:

```js
// my-plugin.mjs
export default async function myPlugin(fastify, opts) {
  fastify.get('/hello', async () => ({ hello: 'world' }))
}

// Named export also works
export async function myPlugin(fastify, opts) {
  fastify.get('/hello', async () => ({ hello: 'world' }))
}
```

Register with dynamic import:

```js
fastify.register(import('./my-plugin.mjs'))
```

## Plugin Options

### Standard Options

```js
fastify.register(myPlugin, {
  logLevel: 'warn',           // Override log level for this scope
  logSerializers: {            // Custom serializers for this scope
    req(req) { return { url: req.url } }
  },
  prefix: '/api/v1'           // Prefix all routes in this plugin
})
```

### Route Prefixing

The `prefix` option prepends a path segment to all routes inside the plugin:

```js
async function usersRoutes(fastify, opts) {
  // GET /api/v1/users
  fastify.get('/users', async () => ({ users: [] }))

  // GET /api/v1/users/:id
  fastify.get('/users/:id', async (request) => {
    return { id: request.params.id }
  })
}

fastify.register(usersRoutes, { prefix: '/api/v1' })
```

Nested prefixes stack:

```js
async function v1Routes(fastify, opts) {
  fastify.register(usersRoutes, { prefix: '/users' })
  // Results in /api/v1/users
}

fastify.register(v1Routes, { prefix: '/api/v1' })
```

### Options as Function

When you need access to parent decorators at registration time, pass options as a function:

```js
fastify.decorate('config', { db: 'postgres://...' })

fastify.register(dbPlugin, parent => {
  return { connectionString: parent.config.db }
})
```

## Error Handling with after/ready/listen

### after()

Runs a callback after the current plugin (and its sub-plugins) finish loading:

```js
fastify
  .register(plugin1)
  .after(err => {
    if (err) throw err
    // plugin1 is now loaded
  })
  .register(plugin2)
```

With async/await:

```js
await fastify.register(plugin1)
// plugin1 is now loaded
await fastify.register(plugin2)
```

### ready()

Fires when all plugins have loaded:

```js
fastify.ready(err => {
  if (err) throw err
  // All plugins loaded, server not yet listening
})

// Or with async/await
await fastify.ready()
```

### listen()

Starts the server after calling `ready()` internally:

```js
await fastify.listen({ port: 3000, host: '0.0.0.0' })
```

## Encapsulation Model

### DAG Structure

Fastify's plugin system forms a Directed Acyclic Graph (DAG). Each `register` call creates a new node in the graph.

```
Root
├── Plugin A (child of Root)
│   ├── Plugin A1 (grandchild of Root, child of A)
│   └── Plugin A2 (grandchild of Root, child of A)
└── Plugin B (child of Root)
```

### Access Rules

- **Parent to child**: A parent context CANNOT access decorators, hooks, or routes defined in child plugins.
- **Child to parent**: A child context CAN access everything from its parent (inheritance).
- **Sibling isolation**: Sibling plugins CANNOT access each other's context.
- **Grandchild access**: Grandchildren inherit from both parent and grandparent.

```js
const fastify = Fastify()

fastify.decorate('rootLevel', 'available everywhere')

fastify.register(async function pluginA(instance) {
  instance.decorate('pluginAOnly', 'only in A and its children')

  instance.register(async function pluginA1(child) {
    console.log(child.rootLevel)    // 'available everywhere' — inherited from root
    console.log(child.pluginAOnly)  // 'only in A and its children' — inherited from parent
  })
})

fastify.register(async function pluginB(instance) {
  console.log(instance.rootLevel)    // 'available everywhere' — inherited from root
  console.log(instance.pluginAOnly)  // undefined — sibling isolation
})
```

### Encapsulation Visualized

```
Root (decorators: rootLevel)
├── Plugin A (decorators: rootLevel + pluginAOnly)
│   └── Plugin A1 (decorators: rootLevel + pluginAOnly)
└── Plugin B (decorators: rootLevel)
```

## Breaking Encapsulation with fastify-plugin

`fastify-plugin` makes a plugin's decorators, hooks, and routes visible to the parent context. It "breaks out" of the encapsulation boundary.

```js
import fp from 'fastify-plugin'

// Without fastify-plugin: dbDecorator is scoped to child only
// With fastify-plugin: dbDecorator is added to the PARENT context
const dbPlugin = fp(async function (fastify, opts) {
  const db = await createConnection(opts.connectionString)
  fastify.decorate('db', db)
}, {
  name: 'my-db-plugin',           // Plugin name (for dependency resolution)
  fastify: '5.x',                 // Semver range for Fastify compatibility
  dependencies: []                 // Required plugins (by name)
})

export default dbPlugin
```

### skip-override Symbol

Under the hood, `fastify-plugin` sets `Symbol.for('skip-override')` on the plugin function. You can do this manually:

```js
function myPlugin(fastify, opts, done) {
  fastify.decorate('myThing', 'value')
  done()
}

myPlugin[Symbol.for('skip-override')] = true

export default myPlugin
```

### When to Use fastify-plugin

| Scenario | Use fastify-plugin? |
|----------|-------------------|
| Database connection shared across routes | Yes |
| Authentication decorator used everywhere | Yes |
| Route group with specific prefix | No |
| Plugin with private hooks/decorators | No |
| Utility decorators (e.g., hash function) | Yes |

## Sharing State Between Contexts

### Using fastify-plugin for Shared Decorators

```js
// db.js — shared database connection
import fp from 'fastify-plugin'

export default fp(async function (fastify, opts) {
  const pool = new Pool(opts)
  fastify.decorate('db', pool)

  fastify.addHook('onClose', async () => {
    await pool.end()
  })
})

// auth.js — shared auth utility
import fp from 'fastify-plugin'

export default fp(async function (fastify, opts) {
  fastify.decorate('authenticate', async function (request, reply) {
    try {
      await request.jwtVerify()
    } catch (err) {
      reply.send(err)
    }
  })
}, { dependencies: ['@fastify/jwt'] })
```

## Complete Example: REST API with Auth Scope vs Public Scope

```js
import Fastify from 'fastify'
import fp from 'fastify-plugin'
import fastifyJwt from '@fastify/jwt'

const fastify = Fastify({ logger: true })

// Shared plugin — available everywhere (uses fastify-plugin)
const dbPlugin = fp(async function (fastify, opts) {
  // Simulated DB connection
  const db = {
    users: [
      { id: 1, name: 'Alice', email: 'alice@example.com' },
      { id: 2, name: 'Bob', email: 'bob@example.com' }
    ],
    posts: [
      { id: 1, title: 'Hello World', authorId: 1, published: true },
      { id: 2, title: 'Draft Post', authorId: 2, published: false }
    ]
  }
  fastify.decorate('db', db)
})

await fastify.register(dbPlugin)
await fastify.register(fastifyJwt, { secret: 'supersecret' })

// Auth utility — shared via fastify-plugin
const authPlugin = fp(async function (fastify, opts) {
  fastify.decorate('authenticate', async function (request, reply) {
    try {
      await request.jwtVerify()
    } catch (err) {
      reply.send(err)
    }
  })
})

await fastify.register(authPlugin)

// ──── Public Scope ────
// No authentication required
fastify.register(async function publicRoutes(fastify, opts) {
  fastify.get('/posts', async (request) => {
    return fastify.db.posts.filter(p => p.published)
  })

  fastify.post('/login', async (request, reply) => {
    const { email } = request.body
    const user = fastify.db.users.find(u => u.email === email)
    if (!user) {
      reply.code(401).send({ error: 'Invalid credentials' })
      return
    }
    const token = fastify.jwt.sign({ id: user.id, email: user.email })
    return { token }
  })
}, { prefix: '/api/public' })

// ──── Authenticated Scope ────
// All routes require JWT
fastify.register(async function authRoutes(fastify, opts) {
  // This hook applies ONLY to routes in this scope
  fastify.addHook('onRequest', fastify.authenticate)

  fastify.get('/profile', async (request) => {
    const user = fastify.db.users.find(u => u.id === request.user.id)
    return user
  })

  fastify.get('/posts', async (request) => {
    // Authenticated users see all posts, including drafts
    return fastify.db.posts.filter(p => p.authorId === request.user.id)
  })

  fastify.post('/posts', async (request, reply) => {
    const post = {
      id: fastify.db.posts.length + 1,
      ...request.body,
      authorId: request.user.id
    }
    fastify.db.posts.push(post)
    reply.code(201).send(post)
  })
}, { prefix: '/api/admin' })

await fastify.listen({ port: 3000 })
```

This produces the following route structure:
- `GET /api/public/posts` — no auth, published posts only
- `POST /api/public/login` — no auth, returns JWT
- `GET /api/admin/profile` — requires JWT
- `GET /api/admin/posts` — requires JWT, user's posts including drafts
- `POST /api/admin/posts` — requires JWT, create post
