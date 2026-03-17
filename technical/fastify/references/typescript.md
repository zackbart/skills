# Fastify TypeScript Reference (v5.8.x)

## Getting Started

```bash
npm init -y && npm i fastify && npm i -D typescript @types/node
npx tsc --init
```

Recommended `tsconfig.json` settings:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "strict": true,
    "outDir": "./dist",
    "declaration": true,
    "sourceMap": true
  }
}
```

## Basic Server Setup

```ts
import Fastify from 'fastify'

const server = Fastify({ logger: true })

server.get('/', async () => {
  return { hello: 'world' }
})

server.listen({ port: 3000 }, (err, address) => {
  if (err) {
    server.log.error(err)
    process.exit(1)
  }
})
```

## Route Generics

Define request/reply shapes using the `RouteGenericInterface` slots: `Body`, `Querystring`, `Params`, `Headers`, and `Reply`.

```ts
interface IQuerystring {
  username: string
  password: string
}

interface IHeaders {
  'h-Custom': string
}

interface IReply {
  200: { token: string }
  302: { url: string }
  '4xx': { error: string }
}

server.get<{
  Querystring: IQuerystring
  Headers: IHeaders
  Reply: IReply
}>(
  '/auth',
  {
    schema: {
      querystring: qsSchema,
      headers: headersSchema,
      response: responseSchema
    }
  },
  async (request, reply) => {
    const { username, password } = request.query
    const customerHeader = request.headers['h-Custom']
    reply.code(200).send({ token: 'jwt' })
  }
)
```

### Body and Params Generics

```ts
interface IBody {
  name: string
  age: number
}

interface IParams {
  id: string
}

server.post<{ Body: IBody; Params: IParams }>(
  '/user/:id',
  async (request, reply) => {
    const { id } = request.params   // typed as string
    const { name, age } = request.body // typed
    reply.send({ id, name, age })
  }
)
```

## Type Providers

Type providers automatically infer request/reply types from your schema definitions, eliminating the need for separate interface declarations.

Available providers:
- `@fastify/type-provider-typebox` — uses TypeBox for runtime + compile-time schemas
- `@fastify/type-provider-json-schema-to-ts` — infers types from JSON Schema objects

### TypeBox Example

```bash
npm i @sinclair/typebox @fastify/type-provider-typebox
```

```ts
import Fastify from 'fastify'
import { Type, Static } from '@sinclair/typebox'
import { TypeBoxTypeProvider } from '@fastify/type-provider-typebox'

const User = Type.Object({
  name: Type.String(),
  age: Type.Number()
})
type UserType = Static<typeof User>

const server = Fastify().withTypeProvider<TypeBoxTypeProvider>()

server.post<{ Body: UserType }>('/user', {
  schema: { body: User }
}, async (request) => {
  const { name, age } = request.body // fully typed
  return { name, age }
})
```

#### TypeBox with Route-Level Schemas

```ts
server.get('/items', {
  schema: {
    querystring: Type.Object({
      limit: Type.Optional(Type.Number({ default: 10 })),
      offset: Type.Optional(Type.Number({ default: 0 }))
    }),
    response: {
      200: Type.Array(Type.Object({
        id: Type.String(),
        name: Type.String()
      }))
    }
  }
}, async (request) => {
  const { limit, offset } = request.query // typed as { limit?: number; offset?: number }
  return []
})
```

### json-schema-to-ts Example

```bash
npm i @fastify/type-provider-json-schema-to-ts
```

```ts
import Fastify from 'fastify'
import { JsonSchemaToTsProvider } from '@fastify/type-provider-json-schema-to-ts'

const server = Fastify().withTypeProvider<JsonSchemaToTsProvider>()

server.get('/route', {
  schema: {
    querystring: {
      type: 'object',
      properties: {
        name: { type: 'string' }
      },
      required: ['name']
    } as const
  }
}, async (request) => {
  request.query.name // typed as string
})
```

**Important:** Use `as const` on JSON schema objects so TypeScript captures literal types for inference.

## Creating Typed Plugins

### Async Plugin with Declaration Merging

```ts
import { FastifyPluginAsync } from 'fastify'
import fp from 'fastify-plugin'

// Extend the FastifyInstance type with your decoration
declare module 'fastify' {
  interface FastifyInstance {
    myUtil: () => string
  }
}

const myPlugin: FastifyPluginAsync = async (fastify) => {
  fastify.decorate('myUtil', () => 'hello')
}

export default fp(myPlugin, { name: 'my-plugin' })
```

### Plugin with Options

```ts
import { FastifyPluginAsync } from 'fastify'
import fp from 'fastify-plugin'

interface MyPluginOptions {
  prefix: string
  enabled?: boolean
}

const myPlugin: FastifyPluginAsync<MyPluginOptions> = async (fastify, opts) => {
  const { prefix, enabled = true } = opts
  if (enabled) {
    fastify.decorate('prefix', prefix)
  }
}

export default fp(myPlugin, { name: 'my-plugin' })
```

### Plugin with Type Provider

```ts
import { FastifyPluginAsyncTypebox } from '@fastify/type-provider-typebox'
import { Type } from '@sinclair/typebox'

const routes: FastifyPluginAsyncTypebox = async (fastify) => {
  fastify.get('/items', {
    schema: {
      response: {
        200: Type.Array(Type.Object({ id: Type.String() }))
      }
    }
  }, async () => {
    return [{ id: '1' }]
  })
}

export default routes
```

## Extending Request/Reply with Declaration Merging

Add custom properties to request or reply objects across the application:

```ts
declare module 'fastify' {
  interface FastifyRequest {
    user?: { id: string; name: string }
  }
  interface FastifyReply {
    sendSuccess: (data: unknown) => void
  }
}
```

Then decorate within a plugin:

```ts
import fp from 'fastify-plugin'
import { FastifyPluginAsync } from 'fastify'

const decorators: FastifyPluginAsync = async (fastify) => {
  fastify.decorateRequest('user', null)
  fastify.decorateReply('sendSuccess', function (data: unknown) {
    this.code(200).send({ success: true, data })
  })
}

export default fp(decorators, { name: 'decorators' })
```

## HTTP2 Types

```ts
import Fastify from 'fastify'
import type {
  FastifyInstance,
  FastifyHttp2SecureOptions
} from 'fastify'
import { readFileSync } from 'node:fs'
import type { Server, ServerResponse } from 'node:http2'

const serverOptions: FastifyHttp2SecureOptions<Server> = {
  http2: true,
  https: {
    key: readFileSync('key.pem'),
    cert: readFileSync('cert.pem')
  }
}

const server = Fastify(serverOptions)
```

## Key Types Reference

Core instance and lifecycle types:

| Type | Purpose |
|------|---------|
| `FastifyInstance` | The server instance |
| `FastifyRequest` | Incoming request object |
| `FastifyReply` | Outgoing reply object |
| `FastifyPluginAsync` | Async plugin function signature |
| `FastifyPluginCallback` | Callback-style plugin function signature |
| `FastifyPluginOptions` | Base interface for plugin options |
| `FastifySchema` | Schema definition object |
| `FastifySchemaCompiler` | Custom schema compiler function |

Route and handler types:

| Type | Purpose |
|------|---------|
| `RouteShorthandOptions` | Options for `.get()`, `.post()`, etc. |
| `RouteGenericInterface` | Generic slots for Body, Querystring, Params, Headers, Reply |
| `RouteHandlerMethod` | Handler function type |

Error and logging types:

| Type | Purpose |
|------|---------|
| `FastifyError` | Fastify-specific error type |
| `FastifyLoggerInstance` | Logger instance type |
| `FastifyBaseLogger` | Base logger interface |

Raw server types (for advanced use):

| Type | Purpose |
|------|---------|
| `RawServerDefault` | Default raw server (http.Server) |
| `RawRequestDefaultExpression` | Default raw request |
| `RawReplyDefaultExpression` | Default raw reply |

## Typed Hooks

```ts
import { FastifyPluginAsync } from 'fastify'

const plugin: FastifyPluginAsync = async (fastify) => {
  fastify.addHook('onRequest', async (request, reply) => {
    // request and reply are fully typed
    request.log.info('incoming request')
  })

  fastify.addHook('preHandler', async (request, reply) => {
    if (!request.headers.authorization) {
      reply.code(401).send({ error: 'Unauthorized' })
    }
  })

  fastify.addHook('onSend', async (request, reply, payload) => {
    // payload is typed as string | Buffer | null
    return payload
  })
}
```

## Typed Error Handling

```ts
import { FastifyError } from 'fastify'

server.setErrorHandler<FastifyError>(async (error, request, reply) => {
  request.log.error(error)
  reply.code(error.statusCode ?? 500).send({
    error: error.name,
    message: error.message,
    statusCode: error.statusCode ?? 500
  })
})

server.setNotFoundHandler(async (request, reply) => {
  reply.code(404).send({
    error: 'Not Found',
    message: `Route ${request.method} ${request.url} not found`
  })
})
```

## Tips

- Always install `@types/node` alongside TypeScript
- Use `as const` on JSON schema objects to get literal types for json-schema-to-ts inference
- `withTypeProvider<T>()` must be called before adding routes for type inference to work
- For plugins that break encapsulation (using `fastify-plugin`), use declaration merging to add types to `FastifyInstance`
- Use `FastifyPluginAsync` for async plugins, `FastifyPluginCallback` for callback-style
- When using TypeBox, prefer defining schemas at the route level for automatic inference rather than separate interfaces
- Use `FastifyPluginAsyncTypebox` from `@fastify/type-provider-typebox` for plugins with TypeBox type providers built in
- Type providers only infer types within the scope where `withTypeProvider` was called; child plugins need their own call or must use `FastifyPluginAsyncTypebox`
