# Validation and Serialization

Fastify uses a schema-based approach for validation and serialization. Request validation is handled by **Ajv v8** (JSON Schema Draft 7), and response serialization is handled by **fast-json-stringify**.

Validation is applied only when the request `Content-Type` is `application/json`, unless the `content` property is used in the body schema to specify per-content-type schemas.

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Adding Shared Schemas](#adding-shared-schemas)
   1. [$ref Patterns](#ref-patterns)
   2. [Retrieving Schemas](#retrieving-schemas)
3. [Validation](#validation)
   1. [Supported Locations](#supported-locations)
   2. [Complete Schema Example](#complete-schema-example)
   3. [Body Content-Type Validation](#body-content-type-validation)
   4. [Ajv Coercion for Querystring Arrays](#ajv-coercion-for-querystring-arrays)
   5. [Custom Validator Per HTTP Part](#custom-validator-per-http-part)
4. [Ajv Configuration](#ajv-configuration)
   1. [Default Settings](#default-settings)
   2. [WARNING: allErrors and DoS](#warning-allerrors-and-dos)
   3. [Ajv Plugins](#ajv-plugins)
5. [Custom Validators](#custom-validators)
   1. [setValidatorCompiler](#setvalidatorcompiler)
   2. [Joi Example](#joi-example)
   3. [Yup Example](#yup-example)
   4. [Best Practices for Custom Validators](#best-practices-for-custom-validators)
6. [Validation Error Messages](#validation-error-messages)
   1. [schemaErrorFormatter](#schemaerrorformatter)
   2. [Custom errorHandler for Validation Context](#custom-errorhandler-for-validation-context)
   3. [ajv-errors for Per-Property Messages](#ajv-errors-for-per-property-messages)
   4. [ajv-i18n for Localization](#ajv-i18n-for-localization)
7. [Serialization](#serialization)
   1. [Response Schema by Status Code](#response-schema-by-status-code)
   2. [Content-Type Specific Response Schemas](#content-type-specific-response-schemas)
   3. [Performance](#performance)
   4. [Data Filtering](#data-filtering)
8. [Serializer Compiler](#serializer-compiler)
9. [JSON Schema Support Table](#json-schema-support-table)
10. [Validation Error Response Format](#validation-error-response-format)
    1. [attachValidation](#attachvalidation)
11. [Security Warnings](#security-warnings)
    1. [Schema Definitions Use new Function()](#schema-definitions-use-new-function)
    2. [$async Ajv Feature](#async-ajv-feature)

---

## Core Concepts

- **Validation**: Incoming requests are validated against JSON Schema definitions for body, querystring, params, and headers.
- **Serialization**: Outgoing responses are serialized using `fast-json-stringify` when a response schema is defined, yielding a 10-20% throughput improvement over `JSON.stringify`.
- **Shared schemas**: Reusable schemas registered via `addSchema()` can be referenced with `$ref` across routes.
- Response schemas also prevent **accidental disclosure** of sensitive fields — only properties declared in the schema are included in the response.

---

## Adding Shared Schemas

Register reusable schemas on the Fastify instance:

```js
fastify.addSchema({
  $id: 'User',
  type: 'object',
  properties: {
    id: { type: 'integer' },
    name: { type: 'string' },
    email: { type: 'string', format: 'email' },
  },
})

fastify.addSchema({
  $id: 'Address',
  type: 'object',
  properties: {
    street: { type: 'string' },
    city: { type: 'string' },
    zip: { type: 'string' },
  },
})
```

### $ref Patterns

Shared schemas can be referenced using several `$ref` patterns:

| Pattern | Description |
|---|---|
| `User#` | Reference a shared schema by its `$id` |
| `User#/properties/name` | Reference a specific property within a shared schema |
| `#/definitions/foo` | Reference a definition within the current schema |
| `http://example.com/schemas/user.json#` | Absolute URI reference |
| `http://example.com/schemas/user.json#/properties/name` | Absolute URI with JSON pointer |

```js
// Reference shared schema by $id
fastify.post('/users', {
  schema: {
    body: { $ref: 'User#' },
    response: {
      200: { $ref: 'User#' },
    },
  },
  handler: async (request, reply) => { /* ... */ },
})

// Reference with local definitions
fastify.addSchema({
  $id: 'OrderSchema',
  type: 'object',
  definitions: {
    lineItem: {
      type: 'object',
      properties: {
        product: { type: 'string' },
        quantity: { type: 'integer' },
      },
    },
  },
  properties: {
    id: { type: 'integer' },
    items: {
      type: 'array',
      items: { $ref: '#/definitions/lineItem' },
    },
  },
})
```

### Retrieving Schemas

```js
// Get all shared schemas in the current encapsulation context
const schemas = fastify.getSchemas()
// Returns: { User: { ... }, Address: { ... } }

// Get a specific schema by $id
const userSchema = fastify.getSchema('User')
```

Both methods are **encapsulated** — they only return schemas visible in the current plugin context.

---

## Validation

### Supported Locations

Schemas can be defined for four HTTP request parts:

- **`body`** — request body (application/json by default)
- **`querystring`** (or **`query`**) — URL query parameters
- **`params`** — URL path parameters
- **`headers`** — request headers (note: header names are lowercased)

### Complete Schema Example

```js
fastify.post('/items/:category', {
  schema: {
    params: {
      type: 'object',
      required: ['category'],
      properties: {
        category: { type: 'string', enum: ['electronics', 'clothing', 'food'] },
      },
    },
    querystring: {
      type: 'object',
      properties: {
        limit: { type: 'integer', minimum: 1, maximum: 100, default: 10 },
        offset: { type: 'integer', minimum: 0, default: 0 },
        tags: { type: 'array', items: { type: 'string' } },
      },
    },
    headers: {
      type: 'object',
      properties: {
        'x-request-id': { type: 'string', format: 'uuid' },
      },
    },
    body: {
      type: 'object',
      required: ['name', 'price'],
      properties: {
        name: { type: 'string', minLength: 1, maxLength: 255 },
        price: { type: 'number', minimum: 0 },
        description: { type: 'string', nullable: true },
        metadata: {
          oneOf: [
            { type: 'object', properties: { color: { type: 'string' } } },
            { type: 'object', properties: { size: { type: 'string' } } },
          ],
        },
        status: {
          type: 'string',
          enum: ['draft', 'published', 'archived'],
          default: 'draft',
        },
        tags: {
          type: 'array',
          items: { type: 'string' },
          minItems: 0,
          maxItems: 10,
          uniqueItems: true,
        },
        restricted: { not: { type: 'string', const: 'banned' } },
      },
      additionalProperties: false,
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
    const item = await createItem(request.params.category, request.body)
    return reply.code(201).send(item)
  },
})
```

### Body Content-Type Validation

Validate the body differently depending on the request `Content-Type`:

```js
fastify.post('/data', {
  schema: {
    body: {
      content: {
        'application/json': {
          schema: {
            type: 'object',
            required: ['name'],
            properties: {
              name: { type: 'string' },
              value: { type: 'number' },
            },
          },
        },
        'text/plain': {
          schema: {
            type: 'string',
          },
        },
      },
    },
  },
  handler: async (request, reply) => {
    return { received: request.body }
  },
})
```

When using the `content` property, validation applies to the matching `Content-Type` — this is the mechanism for validating non-JSON body content types.

### Ajv Coercion for Querystring Arrays

Because querystrings are parsed as strings, Ajv's `coerceTypes: 'array'` (enabled by default) handles single-value-to-array coercion:

```
# Single value coerced to array
GET /items?tags=javascript
# request.query.tags → ['javascript']

# Multiple values remain an array
GET /items?tags=javascript&tags=node
# request.query.tags → ['javascript', 'node']
```

This works when the schema declares the property as `{ type: 'array', items: { type: 'string' } }`.

### Custom Validator Per HTTP Part

You can set a different validator compiler for a specific HTTP part:

```js
const customQueryValidator = new Ajv({ coerceTypes: false })

fastify.post('/strict', {
  schema: {
    querystring: {
      type: 'object',
      properties: { page: { type: 'integer' } },
    },
    body: {
      type: 'object',
      properties: { name: { type: 'string' } },
    },
  },
  validatorCompiler: ({ schema, method, url, httpPart }) => {
    if (httpPart === 'querystring') {
      return customQueryValidator.compile(schema)
    }
    // Fall back to default for other parts
    return fastify.validatorCompiler({ schema, method, url, httpPart })
  },
  handler: async (request, reply) => { /* ... */ },
})
```

---

## Ajv Configuration

### Default Settings

Fastify creates Ajv with these defaults:

```js
{
  coerceTypes: 'array',       // coerce types, including wrapping scalars into arrays
  useDefaults: true,          // apply default values from schema
  removeAdditional: true,     // remove properties not in schema
  uriResolver: require('fast-uri'),
  addUsedSchema: false,       // prevent $id conflicts
  allErrors: false,           // stop on first error (security default)
}
```

### WARNING: allErrors and DoS

Setting `allErrors: true` makes Ajv validate all properties even after finding an error. This can be exploited as a **Denial of Service** vector with specially crafted payloads. Fastify defaults to `false` for security.

### Ajv Plugins

Add Ajv plugins via the `ajv` factory option:

```js
import Fastify from 'fastify'
import ajvFormats from 'ajv-formats'

const fastify = Fastify({
  ajv: {
    customOptions: {
      // Override default Ajv options
      removeAdditional: false,
      coerceTypes: true, // true instead of 'array'
    },
    plugins: [
      ajvFormats,                    // add format validations
      [ajvKeywords, { keywords: ['transform'] }], // plugin with options as tuple
    ],
  },
})
```

---

## Custom Validators

### setValidatorCompiler

Replace the default Ajv-based validator with a custom implementation:

```js
fastify.setValidatorCompiler(({ schema, method, url, httpPart }) => {
  return (data) => {
    // Must return { value } on success or { error } on failure
    // The error object should have a message property
    return { value: data }
    // OR
    return { error: new Error('Validation failed') }
  }
})
```

### Joi Example

```js
import Joi from 'joi'

fastify.post('/users', {
  schema: {
    body: Joi.object({
      name: Joi.string().required(),
      email: Joi.string().email().required(),
      age: Joi.number().integer().min(0),
    }),
  },
  validatorCompiler: ({ schema }) => {
    return (data) => {
      const { error, value } = schema.validate(data, { abortEarly: false })
      if (error) {
        return { error }
      }
      return { value }
    }
  },
  handler: async (request, reply) => {
    return { user: request.body }
  },
})
```

### Yup Example

```js
import * as yup from 'yup'

fastify.post('/products', {
  schema: {
    body: yup.object({
      name: yup.string().required(),
      price: yup.number().positive().required(),
    }),
  },
  validatorCompiler: ({ schema }) => {
    return (data) => {
      try {
        const value = schema.validateSync(data, { abortEarly: false })
        return { value }
      } catch (error) {
        return { error }
      }
    }
  },
  handler: async (request, reply) => {
    return { product: request.body }
  },
})
```

### Best Practices for Custom Validators

- **Always return `{ value }` or `{ error }`** — never throw from the validator function. Use try-catch internally.
- All validation errors are assigned **`.statusCode = 400`** automatically by Fastify.
- The returned `error` object's `message` property is used in the response.

---

## Validation Error Messages

### schemaErrorFormatter

Customize the validation error message format globally or per-route:

```js
// Global
const fastify = Fastify({
  schemaErrorFormatter: (errors, dataVar) => {
    const message = errors.map(e => `${dataVar}${e.instancePath} ${e.message}`).join(', ')
    return new Error(message)
  },
})

// Per-route (via setSchemaErrorFormatter in a plugin)
fastify.setSchemaErrorFormatter((errors, dataVar) => {
  const message = errors
    .map(e => `${dataVar}${e.instancePath} ${e.message}`)
    .join('; ')
  return new Error(message)
})
```

Parameters:
- `errors` — array of Ajv error objects
- `dataVar` — string identifying the validated part (e.g., `"body"`, `"querystring"`)

### Custom errorHandler for Validation Context

Detect validation errors in the error handler:

```js
fastify.setErrorHandler((error, request, reply) => {
  if (error.validation) {
    // error.validation contains the Ajv errors array
    // error.validationContext is 'body', 'querystring', 'params', or 'headers'
    reply.status(400).send({
      statusCode: 400,
      error: 'Validation Error',
      context: error.validationContext,
      details: error.validation.map(e => ({
        path: e.instancePath,
        message: e.message,
        params: e.params,
      })),
    })
    return
  }
  reply.status(500).send({ error: 'Internal Server Error' })
})
```

### ajv-errors for Per-Property Messages

Use the `ajv-errors` plugin for custom error messages directly in the schema:

```js
import Fastify from 'fastify'
import ajvErrors from 'ajv-errors'

const fastify = Fastify({
  ajv: {
    customOptions: { allErrors: true }, // required by ajv-errors (accept DoS risk)
    plugins: [ajvErrors],
  },
})

fastify.post('/register', {
  schema: {
    body: {
      type: 'object',
      required: ['email', 'password'],
      properties: {
        email: {
          type: 'string',
          format: 'email',
          errorMessage: 'Must be a valid email address',
        },
        password: {
          type: 'string',
          minLength: 8,
          errorMessage: {
            type: 'Password must be a string',
            minLength: 'Password must be at least 8 characters',
          },
        },
      },
      errorMessage: {
        required: {
          email: 'Email is required',
          password: 'Password is required',
        },
      },
    },
  },
  handler: async (request, reply) => {
    return { registered: true }
  },
})
```

### ajv-i18n for Localization

```js
import localize from 'ajv-i18n'

fastify.setErrorHandler((error, request, reply) => {
  if (error.validation) {
    // Localize error messages based on Accept-Language header
    const lang = request.headers['accept-language']?.startsWith('es') ? 'es' : 'en'
    if (lang !== 'en') {
      localize.es(error.validation)
    }
    reply.status(400).send({
      statusCode: 400,
      message: error.validation.map(e => e.message).join(', '),
    })
    return
  }
  reply.send(error)
})
```

---

## Serialization

### Response Schema by Status Code

Define response schemas keyed by individual status code, status code range, or `default`:

```js
fastify.get('/items/:id', {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
          secret: { type: 'string' }, // will be included because it is in the schema
        },
      },
      201: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          created: { type: 'boolean' },
        },
      },
      '2xx': {
        type: 'object',
        properties: {
          success: { type: 'boolean' },
        },
      },
      '4xx': {
        type: 'object',
        properties: {
          statusCode: { type: 'integer' },
          error: { type: 'string' },
          message: { type: 'string' },
        },
      },
      default: {
        type: 'object',
        properties: {
          statusCode: { type: 'integer' },
          message: { type: 'string' },
        },
      },
    },
  },
  handler: async (request, reply) => {
    const item = await getItem(request.params.id)
    if (!item) {
      return reply.code(404).send({ statusCode: 404, error: 'Not Found', message: 'Item not found' })
    }
    return item
  },
})
```

Status code matching priority: exact code (e.g., `200`) > range (e.g., `'2xx'`) > `'default'`.

### Content-Type Specific Response Schemas

Define different response serialization per content type:

```js
fastify.get('/report', {
  schema: {
    response: {
      200: {
        content: {
          'application/json': {
            schema: {
              type: 'object',
              properties: {
                title: { type: 'string' },
                data: { type: 'array', items: { type: 'number' } },
              },
            },
          },
          'text/csv': {
            schema: {
              type: 'string',
            },
          },
        },
      },
    },
  },
  handler: async (request, reply) => {
    // Content-type specific serialization is applied based on reply type
    return { title: 'Report', data: [1, 2, 3] }
  },
})
```

### Performance

When a response schema is defined, Fastify uses `fast-json-stringify` instead of `JSON.stringify`. This provides a **10-20% throughput improvement** because the serializer is compiled ahead of time based on the schema and avoids runtime type checking.

### Data Filtering

Response schemas also serve as a security measure. Properties not declared in the response schema are **excluded from the response**, preventing accidental disclosure of sensitive data like passwords, internal IDs, or tokens.

```js
// Even if handler returns { id: 1, name: 'Alice', passwordHash: '...' },
// only id and name are sent to the client
fastify.get('/user', {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
          // passwordHash is NOT listed, so it is stripped
        },
      },
    },
  },
  handler: async () => getUserFromDb(),
})
```

---

## Serializer Compiler

Replace the default `fast-json-stringify` based serializer:

```js
fastify.setSerializerCompiler(({ schema, method, url, httpStatus, contentType }) => {
  return (data) => JSON.stringify(data)
})
```

Parameters passed to the factory:
- `schema` — the response schema for the given status code
- `method` — HTTP method
- `url` — route URL
- `httpStatus` — the response status code (e.g., `200`, `'2xx'`)
- `contentType` — the response content type (when content-type specific schemas are used)

Per-route override:

```js
fastify.get('/custom', {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: { message: { type: 'string' } },
      },
    },
  },
  serializerCompiler: ({ schema }) => {
    return (data) => JSON.stringify({ wrapped: data })
  },
  handler: async () => ({ message: 'hello' }),
})
```

---

## JSON Schema Support Table

| $ref Target | Validator | Serializer |
|---|---|---|
| `$ref` to `$id` (e.g., `User#`) | Supported | Supported |
| `$ref` to `/definitions` (e.g., `#/definitions/foo`) | Supported | Supported |
| `$ref` to shared schema `$id` (e.g., `http://example.com/user.json#`) | Supported | Supported |
| `$ref` to shared schema `/definitions` (e.g., `http://example.com/defs.json#/definitions/bar`) | Supported | Supported |

Both the validator (Ajv) and the serializer (fast-json-stringify) fully support `$ref` resolution across all patterns.

---

## Validation Error Response Format

The default validation error response:

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "body should have required property 'name'"
}
```

Multiple errors (when `allErrors` is enabled):

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "body/name must NOT have fewer than 1 characters, body must have required property 'email'"
}
```

### attachValidation

Use the `attachValidation` route option to handle validation errors in the handler instead of triggering the error handler:

```js
fastify.post('/users', {
  attachValidation: true,
  schema: {
    body: {
      type: 'object',
      required: ['name'],
      properties: {
        name: { type: 'string' },
      },
    },
  },
  handler: async (request, reply) => {
    if (request.validationError) {
      // Access raw Ajv validation error
      // request.validationError.validation — array of Ajv error objects
      // request.validationError.validationContext — 'body', 'querystring', etc.
      return reply.code(422).send({
        error: 'Unprocessable Entity',
        details: request.validationError.validation,
      })
    }
    return { created: request.body.name }
  },
})
```

---

## Security Warnings

### Schema Definitions Use new Function()

Fastify's schema compilation (both Ajv and fast-json-stringify) uses `new Function()` internally to generate optimized validation and serialization functions. **Never accept user-provided schemas** — this would allow arbitrary code execution.

### $async Ajv Feature

The `$async` keyword in Ajv enables asynchronous validation (e.g., checking a value against a database). This should **not** be used for initial request validation as it creates a **Denial of Service** risk — async operations during validation block the request pipeline and can be exploited to exhaust server resources.
