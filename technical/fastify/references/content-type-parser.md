# Content Type Parsers

Fastify includes built-in parsers for `application/json` and `text/plain` (both UTF-8). For other content types, register custom parsers.

## Default Parsers

- `application/json` — parsed with `JSON.parse`, returns JavaScript object
- `text/plain` — returned as a UTF-8 string

All other content types produce a `FST_ERR_CTP_INVALID_MEDIA_TYPE` error (415 Unsupported Media Type) unless a custom parser is registered.

## addContentTypeParser

### Signature

```js
fastify.addContentTypeParser(contentType, [opts], parserFunction)
```

- `contentType` — `string`, `string[]`, or `RegExp`
- `opts` — optional configuration
- `parserFunction` — `(request, payload, done)` or async `(request, payload)`

### String Content Type

```js
fastify.addContentTypeParser('application/xml', function (request, payload, done) {
  let data = ''
  payload.on('data', chunk => { data += chunk })
  payload.on('end', () => {
    done(null, data)
  })
})
```

### Array of Content Types

```js
fastify.addContentTypeParser(
  ['application/x-www-form-urlencoded', 'multipart/form-data'],
  function (request, payload, done) {
    // Handle both content types
    done(null, payload)
  }
)
```

### RegExp Content Type

```js
// Match all text/* types
fastify.addContentTypeParser(/^text\/.*/, function (request, payload, done) {
  let data = ''
  payload.on('data', chunk => { data += chunk })
  payload.on('end', () => done(null, data))
})
```

### Async Parser Support

```js
fastify.addContentTypeParser('application/xml', async function (request, payload) {
  let data = ''
  for await (const chunk of payload) {
    data += chunk
  }
  return parseXml(data)
})
```

## Parser Options

### parseAs

Tells Fastify to collect the request body into a `string` or `Buffer` before calling the parser, instead of passing the raw stream:

```js
// Body collected as string before parser runs
fastify.addContentTypeParser('application/xml', { parseAs: 'string' }, function (request, body, done) {
  // 'body' is a string, not a stream
  try {
    const parsed = parseXml(body)
    done(null, parsed)
  } catch (err) {
    done(err)
  }
})

// Body collected as Buffer
fastify.addContentTypeParser('application/octet-stream', { parseAs: 'buffer' }, function (request, body, done) {
  // 'body' is a Buffer
  done(null, body)
})
```

### bodyLimit

Set a per-parser body size limit (overrides the global `bodyLimit`):

```js
// Allow up to 10MB for file uploads, even if global limit is 1MB
fastify.addContentTypeParser('application/octet-stream', {
  parseAs: 'buffer',
  bodyLimit: 10 * 1024 * 1024  // 10MB
}, function (request, body, done) {
  done(null, body)
})

// Restrict JSON to 256KB
fastify.addContentTypeParser('application/json', {
  parseAs: 'string',
  bodyLimit: 256 * 1024  // 256KB
}, function (request, body, done) {
  try {
    done(null, JSON.parse(body))
  } catch (err) {
    err.statusCode = 400
    done(err)
  }
})
```

## Checking and Removing Parsers

### hasContentTypeParser

```js
if (fastify.hasContentTypeParser('application/xml')) {
  // Parser already registered
}
```

### removeContentTypeParser

Remove one or more parsers:

```js
// Remove single parser
fastify.removeContentTypeParser('text/plain')

// Remove multiple
fastify.removeContentTypeParser(['text/plain', 'application/json'])
```

### removeAllContentTypeParsers

Remove all registered parsers, including the built-in JSON and text/plain parsers:

```js
fastify.removeAllContentTypeParsers()

// Now you must register your own, or all requests with bodies will fail
fastify.addContentTypeParser('application/json', { parseAs: 'string' }, function (request, body, done) {
  try {
    done(null, JSON.parse(body))
  } catch (err) {
    err.statusCode = 400
    done(err)
  }
})
```

## Catch-All Parser

Register a parser for `'*'` to handle any content type that does not have a specific parser:

```js
fastify.addContentTypeParser('*', function (request, payload, done) {
  let data = ''
  payload.on('data', chunk => { data += chunk })
  payload.on('end', () => {
    done(null, data)
  })
})
```

Useful for proxying or when you want to handle the raw body yourself.

## Piping Request Streams

When you do NOT use `parseAs`, the `payload` parameter is the raw Node.js `IncomingMessage` stream. You can pipe it directly:

```js
import { createWriteStream } from 'fs'

fastify.addContentTypeParser('application/octet-stream', function (request, payload, done) {
  const writeStream = createWriteStream('/tmp/upload')
  payload.pipe(writeStream)

  writeStream.on('finish', () => done(null))
  writeStream.on('error', (err) => done(err))
})
```

For streaming to an external service:

```js
fastify.addContentTypeParser('application/octet-stream', async function (request, payload) {
  const response = await fetch('https://storage.example.com/upload', {
    method: 'PUT',
    body: payload,
    duplex: 'half'
  })
  return { uploaded: response.ok }
})
```

## Content Type Matching Order

When multiple parsers match a request's content type:

1. **Exact string match** is checked first
2. **RegExp patterns** are checked in **reverse registration order** (last registered RegExp wins)

```js
// Registered first — lower priority for RegExp
fastify.addContentTypeParser(/^application\/.*\+json$/, handler1)

// Registered second — higher priority for RegExp
fastify.addContentTypeParser(/^application\/.*/, handler2)

// Exact match — always wins over RegExp
fastify.addContentTypeParser('application/vnd.api+json', handler3)

// Request with Content-Type: application/vnd.api+json → handler3 (exact match)
// Request with Content-Type: application/xml → handler2 (last RegExp)
// Request with Content-Type: application/schema+json → handler2 (last RegExp wins)
```

## HTTP Method Behavior

### GET and HEAD

The request body is **never parsed** for GET and HEAD requests, regardless of the Content-Type header. Any body sent with these methods is silently ignored.

### OPTIONS and DELETE

The body is parsed **only if** the request includes a valid Content-Type header that matches a registered parser. If no Content-Type header is present, the body is not parsed.

```js
// This works — Content-Type is set
// DELETE /items/1 with Content-Type: application/json and body {"reason":"obsolete"}
fastify.delete('/items/:id', {
  schema: {
    body: {
      type: 'object',
      properties: {
        reason: { type: 'string' }
      }
    }
  }
}, async (request) => {
  return { deleted: request.params.id, reason: request.body?.reason }
})
```

## Encapsulation

Content type parsers are **encapsulated** within their `register` scope:

```js
fastify.register(async function xmlScope(fastify) {
  // This parser only applies to routes in this scope
  fastify.addContentTypeParser('application/xml', { parseAs: 'string' }, (req, body, done) => {
    done(null, parseXml(body))
  })

  fastify.post('/xml-endpoint', async (request) => {
    // request.body is parsed XML
    return { received: request.body }
  })
})

// Routes outside the scope do NOT have the XML parser
fastify.post('/json-only', async (request) => {
  // application/xml would return 415 here
  return request.body
})
```

## getDefaultJsonParser

Returns a parser function that behaves like Fastify's built-in JSON parser, with configurable prototype poisoning protection:

```js
const defaultJsonParser = fastify.getDefaultJsonParser('error', 'error')
```

Arguments:
- `onProtoPoisoning` — `'error'` | `'remove'` | `'ignore'` — what to do when `__proto__` is found
- `onConstructorPoisoning` — `'error'` | `'remove'` | `'ignore'` — what to do when `constructor` is found

```js
// Re-register JSON parser with strict prototype poisoning protection
fastify.removeContentTypeParser('application/json')

fastify.addContentTypeParser('application/json', { parseAs: 'string' },
  fastify.getDefaultJsonParser('error', 'error')
)
```

### Prototype Poisoning Options

| Value | Behavior |
|-------|----------|
| `'error'` | Throws a `400 Bad Request` error if the key is found |
| `'remove'` | Silently removes the key from the parsed object |
| `'ignore'` | Allows the key through (NOT recommended) |

## Complete Example: Multi-Format API

```js
import Fastify from 'fastify'

const fastify = Fastify({ logger: true })

// Override default JSON parser with stricter limits
fastify.removeContentTypeParser('application/json')
fastify.addContentTypeParser('application/json', {
  parseAs: 'string',
  bodyLimit: 1024 * 1024  // 1MB
}, (request, body, done) => {
  try {
    done(null, JSON.parse(body))
  } catch (err) {
    err.statusCode = 400
    done(err)
  }
})

// YAML support
fastify.addContentTypeParser('application/x-yaml', {
  parseAs: 'string'
}, async (request, body) => {
  const yaml = await import('yaml')
  return yaml.parse(body)
})

// MessagePack (binary)
fastify.addContentTypeParser('application/x-msgpack', {
  parseAs: 'buffer',
  bodyLimit: 5 * 1024 * 1024  // 5MB
}, async (request, body) => {
  const msgpack = await import('msgpackr')
  return msgpack.unpack(body)
})

// Catch-all for unknown types — reject with 415
fastify.addContentTypeParser('*', function (request, payload, done) {
  done(new Error('Unsupported content type'))
})

fastify.post('/data', async (request) => {
  // request.body is parsed regardless of format
  return { received: request.body }
})

await fastify.listen({ port: 3000 })
```
