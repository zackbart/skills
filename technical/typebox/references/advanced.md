# TypeBox Advanced Features Reference

## Table of Contents

1. [Module System](#module-system)
2. [Generic Types](#generic-types)
3. [Mapped Types](#mapped-types)
4. [Conditional Types](#conditional-types)
5. [Script Evaluation Types](#script-evaluation-types)
6. [Codec (Bi-directional Transforms)](#codec-bi-directional-transforms)
7. [Refine (Custom Validation Logic)](#refine-custom-validation-logic)
8. [Compile (JIT Compiler)](#compile-jit-compiler)
9. [Schema Module (Low-Level JSON Schema Validation)](#schema-module-low-level-json-schema-validation)
10. [Script DSL](#script-dsl)
11. [Format Module](#format-module)
12. [System Module](#system-module)
13. [Migration from @sinclair/typebox (0.34.x)](#migration-from-sinclairtypebox-034x)

## Module System

Type.Module is an advanced compositing system for referential types. It performs reference inlining, cyclic type resolution, and dead code elimination.

### Key Capabilities

- **Order-Independent Referencing** — Types can reference other types before they're defined using `Type.Ref()` within a Module scope
- **Automatic Inlining** — Referenced types are cloned into the referring type, producing fully self-contained schemas
- **Cyclic Detection** — Self-referential types are automatically identified and transformed into `TCyclic` instances
- **Dead Code Elimination** — Unused definitions are excluded from the final output

```typescript
import Type from 'typebox'

const Module = Type.Module({
  User: Type.Object({
    id: Type.Number(),
    name: Type.String(),
    posts: Type.Array(Type.Ref('Post')),             // Forward reference
  }),
  Post: Type.Object({
    id: Type.Number(),
    title: Type.String(),
    author: Type.Ref('User'),                        // Back reference (cyclic)
  }),
})
```

## Generic Types

Type.Generic creates generic types that can be instantiated with Type.Call.

Note: This is a Script evaluation action.

```typescript
const ArrayOf = Type.Generic(
  [Type.Parameter('T')],
  Type.Array(Type.Ref('T'))
)

// Instantiate the generic
const Numbers = Type.Call(ArrayOf, [Type.Number()])
type Numbers = Type.Static<typeof Numbers>           // number[]
```

**Guard:** `Type.IsGeneric(value)` — type assertion: `value is TGeneric`

## Mapped Types

Type.Mapped applies a mapped operation to a type. This is a Script evaluation action.

```typescript
const T = Type.Object({
  x: Type.Number(),
  y: Type.Number(),
  z: Type.Number()
})

// Make all properties nullable
const S = Type.Mapped(
  Type.Identifier('K'),
  Type.KeyOf(T),
  Type.Union([Type.Index(T, Type.Ref('K')), Type.Null()])
)
type S = Type.Static<typeof S>                       // { x: number | null; y: number | null; z: number | null }
```

## Conditional Types

Type.Conditional creates conditional type expressions. This is a Script evaluation action.

```typescript
const L = Type.Literal(1)
const R = Type.Number()

const T = Type.Conditional(L, R,
  Type.Literal(true),                                // true branch
  Type.Literal(false)                                // false branch
)
// Result: TLiteral<true> — because Literal(1) extends Number
```

## Script Evaluation Types

These types are Script evaluation actions — they use TypeBox's internal scripting engine.

### Type.Identifier

Creates a named identifier for use in Mapped types and other Script expressions.

```typescript
const T = Type.Identifier('A')                       // { type: 'identifier', name: 'A' }
```

**Guard:** `Type.IsIdentifier(value)` — type assertion: `value is TIdentifier`

### Type.Call

Invokes a Generic type with concrete type arguments.

```typescript
const G = Type.Generic([Type.Parameter('T')], Type.Array(Type.Ref('T')))
const S = Type.Call(G, [Type.Number()])               // TArray<TNumber>
```

**Guard:** `Type.IsCall(value)` — type assertion: `value is TCall`

### Type.Infer

Creates an Infer instruction to extract types in conditional expressions (like TypeScript's `infer` keyword).

```typescript
const T = Type.Object({ x: Type.Literal(1), y: Type.Literal(2) })

const S = Type.Conditional(T, Type.Object({
  x: Type.Infer('X'),
  y: Type.Infer('Y'),
}), Type.Tuple([
  Type.Ref('X'),
  Type.Ref('Y')
]), Type.Never())
// Result: TTuple<[TLiteral<1>, TLiteral<2>]>
```

**Guard:** `Type.IsInfer(value)` — type assertion: `value is TInfer`

### Type.Rest

Creates a Rest type from a Tuple.

```typescript
const T = Type.Rest(Type.Tuple([Type.Literal(1), Type.Literal(2)]))
type T = Type.Static<typeof T>                       // [1, 2][]
```

**Guard:** `Type.IsRest(value)` — type assertion: `value is TRest`

## Codec (Bi-directional Transforms)

Type.Codec applies Decode/Encode transforms to a type.

```typescript
const Timestamp = Type.Codec(Type.Number())
  .Decode(value => new Date(value))                  // number → Date
  .Encode(value => value.getTime())                  // Date → number

type Decoded = Type.StaticDecode<typeof Timestamp>   // Date
type Encoded = Type.StaticEncode<typeof Timestamp>   // number
```

Use with Value.Decode and Value.Encode:

```typescript
import Value from 'typebox/value'

const T = Type.Object({ date: Timestamp })

// Decode: number → Date
const decoded = Value.Decode(T, { date: 1234567890 })

// Encode: Date → number
const encoded = Value.Encode(T, { date: new Date() })
```

The codec is stored in a `'~codec'` property on the schema.

### Type.Decode (Uni-directional)

Applies a decode-only transform. The encode direction throws "not implemented".

```typescript
const Hex = Type.Decode(Type.Number(),               // Decode: number → hex string
  value => value.toString(16).toUpperCase()
)

const D = Value.Decode(Hex, 12345678)                // 'BC614E'
const E = Value.Encode(Hex, D)                       // throws Error('not implemented')
```

### Type.Encode (Uni-directional)

Applies an encode-only transform. The decode direction throws "not implemented".

Warning: Encode callbacks receive `unknown` because TypeBox cannot statically derive the decoded state. Apply runtime checks inside the callback.

```typescript
const T = Type.Encode(Type.Number(),
  (value: unknown) => value === 'hello' ? 1 : 0
)

const E = Value.Encode(T, 'hello')                   // 1
const D = Value.Decode(T, 12345)                     // throws Error('not implemented')
```

## Refine (Custom Validation Logic)

Type.Refine applies explicit validation logic to a type.

**Warning:** Refine embeds runtime logic that cannot be serialized as JSON Schema. Only use when schemas don't need to be shared across systems.

```typescript
const T = Type.Refine(
  Type.Object({ x: Type.Number(), y: Type.Number() }),
  (value) => {
    if (value.x !== value.y) {
      return { message: 'x and y must be equal' }
    }
  }
)

// Validation failure returns:
// keyword: '~refine'
// schemaPath: '#/~refine'
```

## Compile (JIT Compiler)

The Compile module provides high-performance JIT-compiled validation.

```typescript
import { Compile } from 'typebox/compile'

const T = Type.Object({
  x: Type.Number(),
  y: Type.Number(),
  z: Type.Number()
})

const C = Compile(T)

// Validate and return typed result
const result = C.Parse({ x: 1, y: 2, z: 3 })
```

Use Compile when you need maximum validation throughput. The compiled validator is reusable across multiple Parse calls.

## Schema Module (Low-Level JSON Schema Validation)

A standalone JSON Schema validation system supporting Drafts 3 through 2020-12. Decoupled from Type.* and Value.* modules. Designed as an ultra-lightweight, high-performance alternative to Ajv.

```typescript
import Schema from 'typebox/schema'

// Works with raw JSON Schema — no TypeBox types needed
const C = Schema.Compile({
  type: 'object',
  required: ['x', 'y', 'z'],
  properties: {
    x: { type: 'number' },
    y: { type: 'number' },
    z: { type: 'number' }
  }
})

const result = C.Parse({ x: 0, y: 0, z: 0 })
```

Can also be used with TypeBox schemas since they produce standard JSON Schema:

```typescript
const T = Type.Object({ x: Type.Number() })
const C = Schema.Compile(T)
const valid = C.Check({ x: 1 })
```

## Script DSL

TypeBox includes a runtime TypeScript DSL engine that transforms TypeScript syntax into JSON Schema.

```typescript
// Basic script
const T = Type.Script(`{
  x: number,
  y: string,
  z: boolean
}`)

// Reflect the result
T.type                                               // 'object'
T.required                                           // ['x', 'y', 'z']
T.properties                                         // { x: ..., y: ..., z: ... }

// Computed types with references
const S = Type.Script({ T }, `{
  [K in keyof T]: T[K] | null
}`)

// Type inference works
type S = Type.Static<typeof S>
// { x: number | null; y: string | null; z: boolean | null }
```

## Format Module

Validates string formats referenced in JSON Schema spec (Defined Formats 7.3).

```typescript
import Format from 'typebox/format'

Format.IsEmail('user@domain.com')                    // true
```

## System Module

Core configuration for TypeBox.

```typescript
import { Settings, Locale, Memory } from 'typebox/system'
```

### Settings

```typescript
// Enable corrective parsing
Settings.Set({ correctiveParse: true })

// Reset to defaults
Settings.Reset()
```

### Locale

Configure error message language/format.

### Memory

Interact with the TypeBox memory management system.

## Migration from @sinclair/typebox (0.34.x)

Most 0.34.x types are compatible with V1. Both packages can run side by side.

### Key Changes

| 0.34.x | V1 |
|--------|-----|
| `import { Type } from '@sinclair/typebox'` | `import Type from 'typebox'` |
| `import { Value } from '@sinclair/typebox/value'` | `import Value from 'typebox/value'` |
| `import { TypeCompiler } from '@sinclair/typebox/compiler'` | `import { Compile } from 'typebox/compile'` |
| Named exports | Default exports |
| `Static<typeof T>` from package | `Type.Static<typeof T>` |

### Compatibility Example

```typescript
import { Type } from '@sinclair/typebox'             // 0.34.x
import { Static } from 'typebox'                     // V1

// Legacy types work with modern inference
const T = Type.Object({ x: Type.Number() })
type T = Static<typeof T>

// Legacy types work with modern validation
import Schema from 'typebox/schema'
Schema.Compile(T).Check({ x: 1 })
```

Full migration guide: https://github.com/sinclairzx81/typebox/blob/main/changelog/1.0.0-migration.md
