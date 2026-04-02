# Logging

Fastify uses [Pino](https://getpino.io/) as its default logger. Logging is disabled by default and must be explicitly enabled.

## Table of Contents

1. [Enabling the Logger](#enabling-the-logger)
2. [Environment-Specific Configuration](#environment-specific-configuration)
3. [Using the Logger](#using-the-logger)
   1. [Server-Level Logging](#server-level-logging)
   2. [Request-Level Logging](#request-level-logging)
4. [Logger Options](#logger-options)
   1. [level](#level)
   2. [file](#file)
   3. [stream](#stream)
   4. [transport](#transport)
5. [Request ID Tracking](#request-id-tracking)
   1. [requestIdHeader](#requestidheader)
   2. [genReqId](#genreqid)
   3. [requestIdLogLabel](#requestidloglabel)
6. [Custom Serializers](#custom-serializers)
   1. [Default Serializers](#default-serializers)
   2. [Custom req/res Serializers](#custom-reqres-serializers)
   3. [WARNING: Body Cannot Be Serialized in req Serializer](#warning-body-cannot-be-serialized-in-req-serializer)
   4. [Log Body via preHandler Hook](#log-body-via-prehandler-hook)
7. [Custom Logger Instances](#custom-logger-instances)
   1. [Using a Non-Pino Logger](#using-a-non-pino-logger)
8. [Log Redaction for Sensitive Data](#log-redaction-for-sensitive-data)
   1. [Advanced Redaction](#advanced-redaction)
   2. [Redacting in Child Loggers](#redacting-in-child-loggers)
9. [Per-Plugin Log Level](#per-plugin-log-level)
10. [Per-Route Log Level](#per-route-log-level)

---

## Enabling the Logger

```js
import Fastify from 'fastify'

// Basic — defaults to level 'info'
const fastify = Fastify({ logger: true })

// With level
const fastify = Fastify({
  logger: { level: 'info' }
})

// Disabled (default)
const fastify = Fastify({ logger: false })
```

## Environment-Specific Configuration

```js
const envToLogger = {
  development: {
    transport: {
      target: 'pino-pretty',
      options: {
        translateTime: 'HH:MM:ss Z',
        ignore: 'pid,hostname'
      }
    }
  },
  production: true,
  test: false
}

const fastify = Fastify({
  logger: envToLogger[process.env.NODE_ENV] ?? true
})
```

Install `pino-pretty` as a dev dependency:

```bash
npm install -D pino-pretty
```

## Using the Logger

### Server-Level Logging

```js
fastify.log.info('Server starting')
fastify.log.error({ err: new Error('failed') }, 'Operation failed')
fastify.log.debug({ data: someObject }, 'Debug info')
fastify.log.warn('Deprecation warning')
fastify.log.fatal('Unrecoverable error')
fastify.log.trace('Detailed trace info')
```

### Request-Level Logging

Request-scoped logging automatically includes the request ID in every log line:

```js
fastify.get('/users', async (request, reply) => {
  request.log.info('Fetching users')
  // Output: {"reqId":"req-1","msg":"Fetching users",...}

  request.log.warn({ userId: 42 }, 'User not found')
  // Output: {"reqId":"req-1","userId":42,"msg":"User not found",...}

  return { users: [] }
})
```

Always prefer `request.log` over `fastify.log` inside route handlers — it provides automatic request correlation.

## Logger Options

### level

Log levels in order: `trace`, `debug`, `info`, `warn`, `error`, `fatal`, `silent`.

```js
const fastify = Fastify({
  logger: { level: 'debug' }
})
```

### file

Write logs to a file:

```js
const fastify = Fastify({
  logger: {
    level: 'info',
    file: '/var/log/app.log'
  }
})
```

### stream

Provide a custom writable stream:

```js
import { createWriteStream } from 'fs'

const logStream = createWriteStream('/var/log/app.log', { flags: 'a' })

const fastify = Fastify({
  logger: {
    level: 'info',
    stream: logStream
  }
})
```

### transport

Pino v7+ transports for log processing (formatting, shipping, etc.):

```js
const fastify = Fastify({
  logger: {
    transport: {
      target: 'pino-pretty'
    }
  }
})

// Multiple targets
const fastify = Fastify({
  logger: {
    transport: {
      targets: [
        {
          target: 'pino-pretty',
          options: { destination: 1 },  // stdout
          level: 'info'
        },
        {
          target: 'pino/file',
          options: { destination: '/var/log/app.log' },
          level: 'error'
        }
      ]
    }
  }
})
```

## Request ID Tracking

Every request is assigned a unique ID for correlation across log lines.

### requestIdHeader

Use an existing header as the request ID (e.g., from a load balancer or API gateway):

```js
const fastify = Fastify({
  logger: true,
  requestIdHeader: 'x-request-id'  // Default: 'request-id'
})
```

If the header is not present, Fastify generates one automatically.

### genReqId

Custom request ID generator:

```js
import { randomUUID } from 'crypto'

const fastify = Fastify({
  logger: true,
  genReqId: (request) => randomUUID()
})
```

The default generator produces `req-1`, `req-2`, etc. (incrementing counter).

### requestIdLogLabel

Change the key name in log output:

```js
const fastify = Fastify({
  logger: true,
  requestIdLogLabel: 'traceId'  // Default: 'reqId'
})
// Output: {"traceId":"req-1","msg":"..."}
```

## Custom Serializers

Serializers control how `req` and `res` objects are represented in log output.

### Default Serializers

Fastify includes default serializers for `req` and `res`:
- `req`: logs `method`, `url`, `hostname`, `remoteAddress`, `remotePort`
- `res`: logs `statusCode`

### Custom req/res Serializers

```js
const fastify = Fastify({
  logger: {
    serializers: {
      req(request) {
        return {
          method: request.method,
          url: request.url,
          hostname: request.hostname,
          remoteAddress: request.ip,
          // Add custom fields
          userAgent: request.headers['user-agent']
        }
      },
      res(reply) {
        return {
          statusCode: reply.statusCode,
          // Add custom fields
          contentLength: reply.getHeader('content-length')
        }
      }
    }
  }
})
```

### WARNING: Body Cannot Be Serialized in req Serializer

The request body is **not yet parsed** when the `req` serializer runs. Attempting to access `request.body` in the serializer will return `undefined` or `null`.

```js
// WRONG — body is not available here
serializers: {
  req(request) {
    return {
      method: request.method,
      url: request.url,
      body: request.body  // Always undefined/null!
    }
  }
}
```

### Log Body via preHandler Hook

To log the request body, use a `preHandler` hook (body has been parsed at this stage):

```js
fastify.addHook('preHandler', async (request, reply) => {
  if (request.body) {
    request.log.info({ body: request.body }, 'Request body')
  }
})
```

For specific routes only:

```js
fastify.post('/users', {
  preHandler: async (request, reply) => {
    request.log.info({ body: request.body }, 'Creating user')
  }
}, async (request, reply) => {
  // Handle request
})
```

## Custom Logger Instances

Use the `loggerInstance` option to pass a pre-configured logger:

```js
import pino from 'pino'

const customLogger = pino({
  level: 'info',
  redact: ['req.headers.authorization'],
  transport: {
    target: 'pino-pretty'
  }
})

const fastify = Fastify({
  loggerInstance: customLogger
})
```

The custom logger must implement the Pino interface: `info()`, `error()`, `debug()`, `fatal()`, `warn()`, `trace()`, `child()`.

### Using a Non-Pino Logger

Any logger conforming to the Pino interface works. Minimal interface:

```js
const myLogger = {
  info: (msg) => console.log(msg),
  error: (msg) => console.error(msg),
  debug: (msg) => console.debug(msg),
  fatal: (msg) => console.error(msg),
  warn: (msg) => console.warn(msg),
  trace: (msg) => console.trace(msg),
  child: () => myLogger,  // Must return a logger instance
  level: 'info'
}

const fastify = Fastify({ loggerInstance: myLogger })
```

## Log Redaction for Sensitive Data

Pino supports built-in redaction of sensitive fields:

```js
const fastify = Fastify({
  logger: {
    level: 'info',
    redact: ['req.headers.authorization', 'req.headers.cookie']
  }
})
// Output: {"req":{"headers":{"authorization":"[Redacted]","cookie":"[Redacted]"}}}
```

### Advanced Redaction

```js
const fastify = Fastify({
  logger: {
    redact: {
      paths: [
        'req.headers.authorization',
        'req.headers.cookie',
        'body.password',
        'body.creditCard',
        '*.secret'               // Wildcard — any key named 'secret'
      ],
      censor: '***REDACTED***',   // Custom replacement string
      remove: false               // true to remove the key entirely
    }
  }
})
```

### Redacting in Child Loggers

Redaction is configured at the root logger. Child loggers (like `request.log`) inherit the redaction rules automatically.

## Per-Plugin Log Level

Override the log level for a specific plugin scope:

```js
fastify.register(import('./routes/health.js'), {
  logLevel: 'warn'  // Only warn and above for health checks
})

fastify.register(import('./routes/api.js'), {
  logLevel: 'debug'  // Verbose logging for API routes
})
```

## Per-Route Log Level

```js
fastify.get('/health', {
  logLevel: 'silent'  // Suppress all logging for this route
}, async () => {
  return { status: 'ok' }
})

fastify.get('/debug-endpoint', {
  logLevel: 'trace'
}, async (request) => {
  request.log.trace('Detailed trace for this route')
  return { debug: true }
})
```
