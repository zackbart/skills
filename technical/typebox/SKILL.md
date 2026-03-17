---
name: typebox
description: >
  TypeBox JSON Schema type builder assistant for TypeScript. Use when the user writes schemas with
  TypeBox, imports from 'typebox' or '@sinclair/typebox', defines runtime types using Type.Object /
  Type.String / Type.Number / Type.Array etc., validates data with Value.Check / Value.Parse, uses
  Type.Static for inference, builds codecs with Type.Codec, creates modules with Type.Module, or
  asks about JSON Schema generation in TypeScript. Also trigger when the user mentions typebox,
  asks how to validate data at runtime with type inference, or writes code using Type.* / Value.*
  patterns. Covers all Type builders, Value operations, Compile, Schema, Script, Format, System,
  Codec, Refine, Module, and Generic types.
---

# TypeBox — JSON Schema Type Builder

You are an expert at building JSON Schema types with static TypeScript inference using TypeBox. TypeBox creates in-memory JSON Schema objects that infer as TypeScript types, providing both compile-time type safety and runtime validation.

## Core Principles

1. **JSON Schema under the hood** — Every `Type.*` call returns a plain JSON Schema object. `Type.Object()` produces `{ type: 'object', ... }`, `Type.String()` produces `{ type: 'string' }`, etc. This means TypeBox schemas are portable and interoperable with any JSON Schema tooling.
2. **Static inference via `Type.Static<T>`** — Extract TypeScript types from any schema. The inferred types match what the TypeScript compiler would produce for equivalent type declarations.
3. **Validation is separate from type building** — Use the `Value` module (`typebox/value`) for runtime operations (Check, Parse, Create, Errors). Use `Compile` (`typebox/compile`) for high-performance JIT validation.
4. **V1 uses default imports** — `import Type from 'typebox'`, `import Value from 'typebox/value'`, `import Schema from 'typebox/schema'`. The legacy `@sinclair/typebox` package uses named imports.
5. **Options on last argument** — Every type function accepts an optional options object as the last parameter for JSON Schema annotations (min, max, format, default, etc.).

## How to Use This Skill

When the user needs TypeBox help, consult the reference files for detailed patterns:

- **`references/types.md`** — All Type.* builders: primitives, objects, arrays, unions, intersects, tuples, literals, enums, records, template literals, and every utility type (Pick, Omit, Partial, Required, KeyOf, Index, Mapped, Conditional, Exclude, Extract)
- **`references/value.md`** — Value module operations: Check, Parse, Assert, Create, Clean, Convert, Default, Decode, Encode, Clone, Diff, Patch, Hash, Equal, Errors, Mutate, Pointer, Repair, Pipeline
- **`references/advanced.md`** — Module system, Generics, Script DSL, Codec, Refine, Cyclic types, Compile, Schema, Format, System/Settings, migration from 0.34.x

Read the relevant reference file before generating code.

## Quick Reference

### Import Paths (V1)

```typescript
import Type from 'typebox'                          // Type builders + Static
import Value from 'typebox/value'                    // Runtime value operations
import Schema from 'typebox/schema'                  // Low-level JSON Schema validation
import { Compile } from 'typebox/compile'            // JIT schema compiler
import Format from 'typebox/format'                  // String format validators
import { Settings, Locale, Memory } from 'typebox/system' // Configuration
```

### Legacy Imports (0.34.x)

```typescript
import { Type, Static } from '@sinclair/typebox'
import { Value } from '@sinclair/typebox/value'
```

### Building Types

```typescript
import Type from 'typebox'

const User = Type.Object({                           // JSON Schema object
  id: Type.Number(),
  name: Type.String({ minLength: 1 }),
  email: Type.String({ format: 'email' }),
  role: Type.Union([Type.Literal('admin'), Type.Literal('user')]),
  tags: Type.Array(Type.String()),
  metadata: Type.Optional(Type.Record(Type.String(), Type.Unknown())),
})

type User = Type.Static<typeof User>                 // Inferred TypeScript type
```

### Common Type Builders

| TypeBox | TypeScript | JSON Schema |
|---------|-----------|-------------|
| `Type.String()` | `string` | `{ type: 'string' }` |
| `Type.Number()` | `number` | `{ type: 'number' }` |
| `Type.Integer()` | `number` | `{ type: 'integer' }` |
| `Type.Boolean()` | `boolean` | `{ type: 'boolean' }` |
| `Type.Null()` | `null` | `{ type: 'null' }` |
| `Type.Literal('x')` | `'x'` | `{ type: 'string', const: 'x' }` |
| `Type.Array(T)` | `T[]` | `{ type: 'array', items: T }` |
| `Type.Object({...})` | `{ ... }` | `{ type: 'object', ... }` |
| `Type.Tuple([A, B])` | `[A, B]` | `{ type: 'array', ... }` |
| `Type.Union([A, B])` | `A \| B` | `{ anyOf: [A, B] }` |
| `Type.Intersect([A, B])` | `A & B` | `{ allOf: [A, B] }` |
| `Type.Record(K, V)` | `Record<K, V>` | `{ type: 'object', patternProperties: ... }` |
| `Type.Enum(['A','B'])` | `'A' \| 'B'` | `{ enum: ['A','B'] }` |
| `Type.Optional(T)` | `T?` | adds `~optional` modifier |
| `Type.Readonly(T)` | `readonly T` | adds `~readonly` modifier |

### Validation

```typescript
import Value from 'typebox/value'

// Check (returns boolean)
const valid = Value.Check(User, data)

// Parse (returns typed value or throws)
const user = Value.Parse(User, data)

// Assert (throws AssertError if invalid)
Value.Assert(User, data)

// Errors (get detailed validation errors)
if (!Value.Check(User, data)) {
  for (const error of Value.Errors(User, data)) {
    console.log(error.path, error.message)
  }
}
```

### High-Performance Compilation

```typescript
import { Compile } from 'typebox/compile'

const C = Compile(User)
const result = C.Parse(data)                         // JIT-compiled validation
```

### Utility Types

| TypeBox | TypeScript Equivalent |
|---------|----------------------|
| `Type.Partial(T)` | `Partial<T>` |
| `Type.Required(T)` | `Required<T>` |
| `Type.Pick(T, Keys)` | `Pick<T, Keys>` |
| `Type.Omit(T, Keys)` | `Omit<T, Keys>` |
| `Type.KeyOf(T)` | `keyof T` |
| `Type.Index(T, K)` | `T[K]` |
| `Type.Exclude(U, E)` | `Exclude<U, E>` |
| `Type.Extract(U, E)` | `Extract<U, E>` |

## Common Mistakes to Catch

- **Using named imports with V1** — V1 uses `import Type from 'typebox'` (default import), not `import { Type } from 'typebox'`. Named imports are for the legacy `@sinclair/typebox` package.
- **Calling Value.Errors on valid data** — Errors performs an exhaustive recheck. Only call it after a failed Check, and avoid it in hot paths on large datasets.
- **Forgetting that Optional/Readonly are modifiers, not schema types** — They add metadata markers (`~optional`, `~readonly`) but don't change the JSON Schema validation behavior. They affect TypeScript inference only.
- **Using Refine and expecting portability** — Refine embeds runtime validation logic that cannot be serialized as JSON Schema. Only use it when schemas don't need to be shared across systems.
- **Not wrapping Decode/Encode in try/catch** — These functions throw on invalid input. Always handle errors.
- **Expecting Value.Clean/Convert/Default to validate** — These functions do not validate. They return `unknown` and may produce invalid data. Always validate after transformation.
- **Confusing Value.Parse with Schema.Compile().Parse** — `Value.Parse` uses the Value module pipeline. `Schema.Compile().Parse` uses the low-level JSON Schema validator. Both work, but have different performance profiles and feature sets.
