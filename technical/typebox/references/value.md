# TypeBox Value Module Reference

The Value module provides runtime operations on JavaScript values using TypeBox schemas.

```typescript
import Value from 'typebox/value'
```

## Table of Contents

1. [Validation](#validation)
2. [Transformation](#transformation)
3. [Codec Operations](#codec-operations)
4. [Structural Operations](#structural-operations)
5. [JSON Pointer (RFC 6901)](#json-pointer-rfc-6901)
6. [Pipeline](#pipeline)

## Validation

### Value.Check

Returns `true` if the value matches the type. This is the primary validation function.

```typescript
const T = Type.Number()
Value.Check(T, 42)                                   // true
Value.Check(T, 'not-a-number')                       // false
```

### Value.Parse

Validates and returns a typed value. Throws a parse error if the value does not satisfy the type.

```typescript
const T = Type.Object({ x: Type.Number(), y: Type.Number(), z: Type.Number() })

const result = Value.Parse(T, { x: 1, y: 0, z: 0 }) // typed as { x: number, y: number, z: number }
```

**Corrective Parse Mode:**

Attempts to repair invalid values through a Convert → Default → Clean pipeline before failing. Useful for parsing environment variables.

```typescript
import { Settings } from 'typebox/system'

Settings.Set({ correctiveParse: true })

// Numeric input "12345" converts to string if schema expects string
const result = Value.Parse(Type.String(), 12345)     // "12345"

Settings.Reset()                                     // Reset when done
```

Performance note: corrective parsing impacts performance and is not recommended for high-throughput applications.

### Value.Assert

Throws `AssertError` if the value does not match the type.

```typescript
Value.Assert(T, 42)                                  // OK — no return value
Value.Assert(T, 'not a number')                      // throws AssertError
```

### Value.Errors

Returns validation errors for a value. Only call after a failed Check — this performs an exhaustive recheck.

```typescript
if (!Value.Check(User, data)) {
  for (const error of Value.Errors(User, data)) {
    console.log(error.keyword)                       // error type
    console.log(error.schemaPath)                    // path in schema
    console.log(error.instancePath)                  // path in value (e.g., '/x')
    console.log(error.message)                       // human-readable message
  }
}
```

Performance warning: avoid calling on large datasets in production hot paths. Reserve for development/debugging.

Use `Value.Pointer.Get(data, error.instancePath)` to retrieve the actual invalid value.

## Transformation

### Value.Create

Creates an instance of the given type using default values or zero values.

```typescript
const T = Type.Object({
  x: Type.Number({ default: 10 }),
  y: Type.Number()
})
Value.Create(T)                                      // { x: 10, y: 0 }
```

Ambiguity: if a type has constraints that Create cannot satisfy (e.g., `format: 'email'` on a string), it throws `CreateError`. Fix by adding a `default` annotation.

```typescript
// Ambiguous — Create doesn't know what email to generate
Type.String({ format: 'email' })

// Fixed — provide a default
Type.String({ format: 'email', default: 'user@example.com' })
```

### Value.Clean

Removes excess properties or elements from a value that aren't in the schema. Does not validate.

```typescript
const T = Type.Object({ x: Type.Number(), y: Type.Number() })
Value.Clean(T, { x: 1, y: 2, z: 3 })                // { x: 1, y: 2 }

const S = Type.Tuple([Type.Number(), Type.Number()])
Value.Clean(S, [1, 2, 3])                            // [1, 2]
```

Warning: returns `unknown`. May return invalid data if the input is itself invalid.

### Value.Convert

Transforms values to align with a type when feasible. Original value stays unmodified if conversion isn't possible.

```typescript
const T = Type.Object({ x: Type.Number() })
Value.Convert(T, { x: '3.14' })                     // { x: 3.14 }
Value.Convert(T, { x: 'not a number' })             // { x: 'not a number' } — unchanged
```

Warning: returns `unknown`. Does not validate.

### Value.Default

Applies default annotations to missing properties or elements.

```typescript
const Pagination = Type.Object({
  skip: Type.Number({ default: 0 }),
  take: Type.Number({ default: 10 })
})

Value.Default(Pagination, {})                        // { skip: 0, take: 10 }
Value.Default(Pagination, { skip: 20 })              // { skip: 20, take: 10 }
```

Warning: returns `unknown`. Does not validate.

### Value.Repair

Repairs a value to match the provided type. Prioritizes data retention. If the value already matches, no action is taken.

```typescript
// Converts null to default structure
Value.Repair(T, null)

// Fills missing properties
Value.Repair(T, { x: 1 })                           // adds y with default

// Removes excess properties (only when additionalProperties: false)
Value.Repair(Type.Object({ x: Type.Number() }, { additionalProperties: false }), { x: 1, y: 2 })
```

## Codec Operations

### Value.Decode

Executes Decode callbacks embedded in types (via Type.Codec). Runs a pipeline: Clone → Default → Convert → Clean → Assert → DecodeUnsafe.

```typescript
const Timestamp = Type.Codec(Type.Number())
  .Decode(value => new Date(value))
  .Encode(value => value.getTime())

const T = Type.Object({ date: Timestamp })

const result = Value.Decode(T, { date: 1234567890 })
// result.date is a Date object
```

Wrap in try/catch — throws on invalid input.

**Value.DecodeUnsafe** — Executes codec callbacks without the processing pipeline. Returns `unknown`. For performance-critical paths where you handle validation separately.

### Value.Encode

Executes Encode callbacks embedded in types. Pipeline: Clone → EncodeUnsafe → Default → Convert → Clean → Assert.

```typescript
const result = Value.Encode(T, { date: new Date('1970-01-01T00:00:12.345Z') })
// result.date is 12345 (number)
```

Wrap in try/catch — throws on invalid input.

**Value.EncodeUnsafe** — Direct encoding without pipeline. Returns `unknown`.

## Structural Operations

### Value.Clone

Clones a value. Like `structuredClone()` but also clones `Map`, `Set`, and `TypedArray`.

```typescript
const A = { x: 1, y: 2, z: 3 }
const B = Value.Clone(A)                             // deep copy
```

### Value.Diff

Computes Edit commands to transform one value into another.

```typescript
const L = { x: 1, y: 2, z: 3 }
const R = { y: 4, z: 5, w: 6 }

const edits = Value.Diff(L, R)
// [
//   { type: 'update', path: '/y', value: 4 },
//   { type: 'update', path: '/z', value: 5 },
//   { type: 'insert', path: '/w', value: 6 },
//   { type: 'delete', path: '/x' }
// ]
```

Useful for efficiently synchronizing large data structures — transmit only the edit commands instead of full state.

### Value.Patch

Applies a sequence of Edit commands (from Diff) to a value.

```typescript
const result = Value.Patch(L, edits)                 // { y: 4, z: 5, w: 6 }
```

### Value.Hash

Computes an `fnv1a-64` hash across all properties, elements, and values. Returns a 64-bit hex string.

```typescript
Value.Hash({ x: 1, y: 2, z: 3 })                    // '0834a0916e3e4db0'
Value.Hash({ x: 1, y: 4, z: 3 })                    // '279c16b78fba6600'
```

Warning: not standards-based. Use only for comparing two values, not for persistent storage.

### Value.Equal

Structural value comparison.

```typescript
Value.Equal({ x: 1, y: 2 }, { x: 1, y: 2 })        // true
Value.Equal({ x: 1 }, { x: 2 })                     // false
```

### Value.Mutate

Mutates a value while preserving interior object and array references. Useful in state-aware frameworks (React, etc.) where reference identity matters.

```typescript
const Y = { z: 1 }
const X = { y: Y }
const Z = { x: X }

Value.Mutate(Z, { x: { y: { z: 2 } } })

Z.x === X                                           // true — reference preserved
Z.x.y === Y                                         // true — reference preserved
Z.x.y.z                                             // 2 — value updated
```

## JSON Pointer (RFC 6901)

### Value.Pointer

Implements RFC 6901 JSON Pointer operations.

```typescript
const obj = { x: { y: 1 } }

Value.Pointer.Get(obj, '/x/y')                       // 1
Value.Pointer.Set(obj, '/x/y', 100)                  // sets value, creates intermediate paths
Value.Pointer.Has(obj, '/x/y')                       // true
Value.Pointer.Delete(obj, '/x/y')                    // removes value
```

## Pipeline

Compose multiple Value operations into a single callable function.

```typescript
import { Pipeline, Clone, Default, Convert, Clean, Parse } from 'typebox/value'

const ParsePipeline = Pipeline([
  (_context, _type, value) => Clone(value),
  (context, type, value) => Default(context, type, value),
  (context, type, value) => Convert(context, type, value),
  (context, type, value) => Clean(context, type, value),
  (context, type, value) => Parse(context, type, value),
])
```

Pipelines return `unknown` because the return type can't be statically determined. Wrap in a generic function for type inference:

```typescript
function parse<T extends TSchema>(type: T, value: unknown): Type.Static<T> {
  return ParsePipeline(type, value) as Type.Static<T>
}
```

Practical for building custom validation/transformation chains tailored to your application.
