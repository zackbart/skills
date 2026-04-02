# TypeBox Type Builders Reference

## Table of Contents

1. [Primitive Types](#primitive-types)
2. [Literal and Enum Types](#literal-and-enum-types)
3. [Composite Types](#composite-types)
4. [Modifiers](#modifiers)
5. [Utility Types](#utility-types)
6. [Reference and Recursive Types](#reference-and-recursive-types)
7. [Function Types](#function-types)
8. [Async and Iterator Types](#async-and-iterator-types)
9. [String Manipulation Types](#string-manipulation-types)
10. [Escape Hatch](#escape-hatch)
11. [Schema Options](#schema-options)
12. [Type Guards](#type-guards)

## Primitive Types

### Type.String

Creates a String type.

```typescript
const T = Type.String()                              // { type: 'string' }
type T = Type.Static<typeof T>                       // string
```

**Options (TStringOptions):**
- `format` — A string format (e.g., `'email'`, `'uuid'`, or any registered format)
- `minLength` — Minimum number of characters (non-negative integer)
- `maxLength` — Maximum number of characters (non-negative integer)
- `pattern` — Regular expression pattern the string must match (string or RegExp)

```typescript
const Email = Type.String({ format: 'email' })       // { type: 'string', format: 'email' }
const Short = Type.String({ minLength: 1, maxLength: 50 })
const Hex = Type.String({ pattern: '^[0-9a-f]+$' })
```

**Guard:** `Type.IsString(value)` — type assertion: `value is TString`

### Type.Number

Creates a Number type.

```typescript
const T = Type.Number()                              // { type: 'number' }
type T = Type.Static<typeof T>                       // number
```

**Options (TNumberOptions):**
- `minimum` — Inclusive lower limit (value >= this)
- `maximum` — Inclusive upper limit (value <= this)
- `exclusiveMinimum` — Exclusive lower bound (value > this)
- `exclusiveMaximum` — Exclusive upper bound (value < this)
- `multipleOf` — Value must be a multiple of this number

```typescript
const Percentage = Type.Number({ minimum: 0, maximum: 100 })
const Positive = Type.Number({ exclusiveMinimum: 0 })
```

**Guard:** `Type.IsNumber(value)` — type assertion: `value is TNumber`

### Type.Integer

Creates an Integer type.

```typescript
const T = Type.Integer()                             // { type: 'integer' }
type T = Type.Static<typeof T>                       // number
```

Same options as Type.Number.

### Type.Boolean

```typescript
const T = Type.Boolean()                             // { type: 'boolean' }
type T = Type.Static<typeof T>                       // boolean
```

### Type.Null

```typescript
const T = Type.Null()                                // { type: 'null' }
type T = Type.Static<typeof T>                       // null
```

### Type.BigInt

```typescript
const T = Type.BigInt()                              // { type: 'bigint' }
type T = Type.Static<typeof T>                       // bigint
```

### Type.Symbol

```typescript
const T = Type.Symbol()                              // { type: 'symbol' }
type T = Type.Static<typeof T>                       // symbol
```

### Type.Undefined

```typescript
const T = Type.Undefined()                           // { type: 'undefined' }
type T = Type.Static<typeof T>                       // undefined
```

### Type.Void

```typescript
const T = Type.Void()                                // { type: 'void' }
type T = Type.Static<typeof T>                       // void
```

### Type.Any

```typescript
const T = Type.Any()                                 // {}
type T = Type.Static<typeof T>                       // any
```

### Type.Unknown

```typescript
const T = Type.Unknown()                             // {}
type T = Type.Static<typeof T>                       // unknown
```

### Type.Never

```typescript
const T = Type.Never()                               // { not: {} }
type T = Type.Static<typeof T>                       // never
```

## Literal and Enum Types

### Type.Literal

Creates a Literal type. Supports BigInt, Boolean, Number, and String values.

```typescript
Type.Literal('hello')                                // { type: 'string', const: 'hello' }
Type.Literal(42)                                     // { type: 'number', const: 42 }
Type.Literal(true)                                   // { type: 'boolean', const: true }
Type.Literal(1n)                                     // { type: 'bigint', const: 1n }
```

**Guard:** `Type.IsLiteral(value)` — type assertion: `value is TLiteral<TLiteralValue>`

### Type.Enum

Creates an Enum type from an array of values.

```typescript
const T = Type.Enum(['A', 'B', 'C'])                 // { enum: ['A', 'B', 'C'] }
type T = Type.Static<typeof T>                       // 'A' | 'B' | 'C'
```

**Guard:** `Type.IsEnum(value)`

### Type.TemplateLiteral

Creates a TemplateLiteral type.

```typescript
const T = Type.TemplateLiteral('user-${"admin"|"member"}')
// { type: 'string', pattern: '^user-(admin|member)$' }
type T = Type.Static<typeof T>                       // 'user-admin' | 'user-member'
```

**Guard:** `Type.IsTemplateLiteral(value)` — type assertion: `value is TTemplateLiteral`

## Composite Types

### Type.Object

Creates an Object type.

```typescript
const T = Type.Object({                              // { type: 'object',
  x: Type.Number(),                                  //   required: ['x', 'y'],
  y: Type.Number(),                                  //   properties: { x: {...}, y: {...} }
})                                                   // }

type T = Type.Static<typeof T>                       // { x: number; y: number }
```

**Options (TObjectOptions):**
- `additionalProperties` — Whether additional properties are allowed (boolean or schema)
- `minProperties` / `maxProperties` — Property count boundaries
- `dependentRequired` — Properties that must be present if a given property is present
- `patternProperties` — Schemas applied to properties matching regex patterns
- `propertyNames` — Schema that all property names must validate against

**Guard:** `Type.IsObject(value)` — type assertion: `value is TObject`

### Type.Array

Creates an Array type.

```typescript
const T = Type.Array(Type.Number())                  // { type: 'array', items: { type: 'number' } }
type T = Type.Static<typeof T>                       // number[]
```

**Options (TArrayOptions):**
- `minItems` / `maxItems` — Array length boundaries
- `uniqueItems` — All items must be unique (boolean)
- `contains` — Schema that at least one item must satisfy
- `minContains` / `maxContains` — Bounds for `contains` matches
- `prefixItems` — Array of schemas for items at corresponding positions

**Guard:** `Type.IsArray(value)` — type assertion: `value is TArray`

### Type.Tuple

Creates a Tuple type.

```typescript
const T = Type.Tuple([Type.String(), Type.Number()])
// { type: 'array', additionalItems: false, minItems: 2, items: [...] }
type T = Type.Static<typeof T>                       // [string, number]
```

**Guard:** `Type.IsTuple(value)` — type assertion: `value is TTuple`

### Type.Union

Creates a Union type.

```typescript
const T = Type.Union([Type.String(), Type.Number()])
// { anyOf: [{ type: 'string' }, { type: 'number' }] }
type T = Type.Static<typeof T>                       // string | number
```

**Guard:** `Type.IsUnion(value)` — type assertion: `value is TUnion`

### Type.Intersect

Creates an Intersect type.

```typescript
const A = Type.Object({ x: Type.Number() })
const B = Type.Object({ y: Type.Number() })
const T = Type.Intersect([A, B])                     // { allOf: [A, B] }
type T = Type.Static<typeof T>                       // { x: number } & { y: number }
```

**Options (TIntersectOptions):**
- `unevaluatedProperties` — Controls properties not covered by any schema in the intersection. Set to `false` to prevent additional properties.

**Guard:** `Type.IsIntersect(value)` — type assertion: `value is TIntersect`

### Type.Record

Creates a Record type.

```typescript
const T = Type.Record(Type.String(), Type.Number())
// { type: 'object', patternProperties: { '^.*$': { type: 'number' } } }
type T = Type.Static<typeof T>                       // Record<string, number>
```

**Guard:** `Type.IsRecord(value)` — type assertion: `value is TRecord`

## Modifiers

### Type.Immutable

Applies an `~immutable` modifier to an Array or Tuple. Infers `readonly T[]` for arrays and `readonly [...]` for tuples. Inference only — no effect on validation.

```typescript
const T = Type.Immutable(                            // { type: 'array',
  Type.Array(Type.Number())                          //   items: { type: 'number' },
)                                                    //   '~immutable': true }

type T = Type.Static<typeof T>                       // readonly number[]
```

```typescript
const T = Type.Immutable(                            // readonly [string, number]
  Type.Tuple([Type.String(), Type.Number()])
)
type T = Type.Static<typeof T>                       // readonly [string, number]
```

Can be applied to any type, but TypeBox only interprets inference on Array and Tuple.

**Guard:** `Type.IsImmutable(value)` — type assertion: `value is TImmutable`

### Type.Optional

Applies an optional `?` modifier to a type. Adds `'~optional': true` to the schema.

```typescript
const T = Type.Object({
  x: Type.Number(),
  y: Type.Optional(Type.Number()),                   // y?: number
})
```

**Guard:** `Type.IsOptional(value)`

### Type.Readonly

Applies a `readonly` modifier. This is for type inference only — no effect on validation.

```typescript
const T = Type.Object({
  id: Type.Readonly(Type.Number()),                  // readonly id: number
})
```

**Guard:** `Type.IsReadonly(value)`

### Type.ReadonlyType

Alias for TypeScript's `Readonly<T>`. Makes all Object properties `readonly`, or marks Array/Tuple as immutable.

```typescript
// On Object: wraps each property in TReadonly
const T = Type.Object({ x: Type.Number(), y: Type.Number() })
const S = Type.ReadonlyType(T)                       // { x: TReadonly<TNumber>, y: TReadonly<TNumber> }

// On Array/Tuple: wraps in TImmutable
const U = Type.Tuple([Type.Number(), Type.String()])
const V = Type.ReadonlyType(U)                       // TImmutable<TTuple<[TNumber, TString]>>
type V = Type.Static<typeof V>                       // readonly [number, string]
```

## Utility Types

### Type.NonNullable

Discards `Undefined` and `Null` variants from a Union type (like TypeScript's `NonNullable<T>`).

```typescript
const T = Type.Union([Type.String(), Type.Undefined(), Type.Null()])
const S = Type.NonNullable(T)                        // TString
```

### Type.Exclude

Excludes types from Left that extend Right (like TypeScript's `Exclude<T, U>`).

```typescript
const L = Type.Union([Type.Literal('hello'), Type.Literal('world'), Type.Literal(1), Type.Literal(2)])
const R = Type.String()
const S = Type.Exclude(L, R)                         // TUnion<[TLiteral<1>, TLiteral<2>]>
```

### Type.Extract

Extracts types from Left that extend Right (like TypeScript's `Extract<T, U>`).

```typescript
const L = Type.Union([Type.Literal('hello'), Type.Literal('world'), Type.Literal(1), Type.Literal(2)])
const R = Type.String()
const S = Type.Extract(L, R)                         // TUnion<[TLiteral<'hello'>, TLiteral<'world'>]>
```

### Type.Partial

Makes all properties of an Object optional (like TypeScript's `Partial<T>`).

```typescript
const T = Type.Object({ x: Type.Number(), y: Type.Number() })
const S = Type.Partial(T)
type S = Type.Static<typeof S>                       // { x?: number; y?: number }
```

### Type.Required

Makes all properties of an Object required (like TypeScript's `Required<T>`).

```typescript
const S = Type.Required(PartialType)
```

### Type.Pick

Picks property keys from an Object type.

```typescript
const T = Type.Object({ x: Type.Number(), y: Type.Number(), z: Type.Number() })
const S = Type.Pick(T, Type.Union([Type.Literal('x'), Type.Literal('y')]))
type S = Type.Static<typeof S>                       // { x: number; y: number }
```

### Type.Omit

Omits keys from an Object type.

```typescript
const S = Type.Omit(T, Type.Union([Type.Literal('z')]))
type S = Type.Static<typeof S>                       // { x: number; y: number }
```

### Type.KeyOf

Extracts keys from an Object or Tuple as a Union of Literals.

```typescript
const T = Type.Object({ x: Type.Number(), y: Type.Number(), z: Type.Number() })
const S = Type.KeyOf(T)                              // TUnion<[TLiteral<'x'>, TLiteral<'y'>, TLiteral<'z'>]>
```

### Type.Index

Performs an indexed access lookup on a type (like `T[K]`).

```typescript
const T = Type.Object({
  x: Type.Literal(1),
  y: Type.Literal(2),
  z: Type.Literal(3)
})
const S = Type.Index(T, Type.Union([Type.Literal('x'), Type.Literal('y')]))
// S: TUnion<[TLiteral<1>, TLiteral<2>]>
```

### Type.Evaluate

Evaluates logical type expressions — simplifies Union to broadest disjoint types and Intersect to narrowest types.

```typescript
Type.Evaluate(Type.Union([Type.Literal(1), Type.Number()]))
// { type: 'number' } — Number subsumes Literal(1)

Type.Evaluate(Type.Intersect([Type.Literal(1), Type.Number()]))
// { const: 1 } — Literal(1) is narrower than Number
```

### Type.Instantiate

Instantiates interior Ref types embedded within a type. Accepts a context object to source referenced types.

```typescript
const X = Type.Literal(1)
const Y = Type.Literal(2)
const Z = Type.Literal(3)

const T = Type.Object({
  x: Type.Ref('X'),
  y: Type.Ref('Y'),
  z: Type.Ref('Z'),
})

const S = Type.Instantiate({ X, Y, Z }, T)           // TObject<{ x: TLiteral<1>, y: TLiteral<2>, z: TLiteral<3> }>
```

## Reference and Recursive Types

### Type.Ref

Creates a reference to another type by identifier.

```typescript
const T = Type.Ref('A')                              // { $ref: 'A' }
type T = Type.Static<typeof T>                       // unknown
```

**Guard:** `Type.IsRef(value)` — type assertion: `value is TRef`

### Type.Cyclic

Creates a Cyclic type for recursive data structures.

```typescript
const Node = Type.Cyclic(
  Type.Object({
    value: Type.String(),
    children: Type.Array(Type.Ref('Node'))
  }),
  { $id: 'Node' }
)
```

Generates JSON Schema with `$defs` and `$ref` for recursive references.

**Guard:** `Type.IsCyclic(value)`

### Type.This

Creates a self-reference within a type using `{ $ref: '#' }`. Supported for top-level Object and Interface types only.

Warning: TypeBox does not transform `#` references when embedding Objects within other Objects — use with care.

```typescript
const T = Type.Object({                              // { type: 'object',
  items: Type.Array(Type.This())                     //   required: ['items'],
})                                                   //   properties: { items: { $ref: '#' } } }

type T = Type.Static<typeof T>                       // type T = object
```

**Guard:** `Type.IsThis(value)` — type assertion: `value is TThis`

### Type.Interface

Creates an Object with heritage (inheritance).

```typescript
const A = Type.Interface([], { a: Type.Number() })   // { a: number }
const B = Type.Interface([A], { b: Type.Number() })  // { a: number, b: number }
```

## Function Types

### Type.Function

```typescript
const T = Type.Function([Type.String()], Type.Number())
type T = Type.Static<typeof T>                       // (arg0: string) => number
```

### Type.Constructor

```typescript
const T = Type.Constructor([Type.String()], Type.Object({ name: Type.String() }))
```

### Type.Parameters / Type.ReturnType / Type.ConstructorParameters / Type.InstanceType

Utility types that mirror TypeScript's built-in utility types for function/constructor types.

## Async and Iterator Types

### Type.Promise

```typescript
const T = Type.Promise(Type.String())
type T = Type.Static<typeof T>                       // Promise<string>
```

### Type.Iterator / Type.AsyncIterator

```typescript
const T = Type.Iterator(Type.Number())               // Iterator<number>
const S = Type.AsyncIterator(Type.Number())          // AsyncIterator<number>
```

### Type.Awaited

```typescript
const T = Type.Awaited(Type.Promise(Type.String()))  // string
```

## String Manipulation Types

### Type.Capitalize / Type.Uncapitalize / Type.Uppercase / Type.Lowercase

```typescript
Type.Capitalize(Type.Literal('hello'))               // 'Hello'
Type.Uppercase(Type.Literal('hello'))                // 'HELLO'
Type.Lowercase(Type.Literal('HELLO'))                // 'hello'
Type.Uncapitalize(Type.Literal('Hello'))             // 'hello'
```

## Escape Hatch

### Type.Unsafe

Create custom schematics with user-defined inference. Named "Unsafe" because the schema may not match the inferred type.

```typescript
const T = Type.Unsafe<string>({ type: 'number' })   // Schema says number, TS says string
type T = Type.Static<typeof T>                       // string
```

## Schema Options

Options can be passed as the last argument on any type:

```typescript
const T = Type.Number({ minimum: 10, maximum: 100 })
const S = Type.String({ format: 'email' })
const U = Type.Array(Type.String(), { minItems: 1, maxItems: 10 })
```

Or using the Options function:

```typescript
const T = Type.Options(Type.Number(), { minimum: 10, maximum: 100 })
```

## Type Guards

Every type has a corresponding guard function:

```typescript
Type.IsString(value)          // value is TString
Type.IsNumber(value)          // value is TNumber
Type.IsObject(value)          // value is TObject
Type.IsArray(value)           // value is TArray
Type.IsUnion(value)           // value is TUnion
Type.IsIntersect(value)       // value is TIntersect
Type.IsLiteral(value)         // value is TLiteral
Type.IsEnum(value)            // value is TEnum
Type.IsTuple(value)           // value is TTuple
Type.IsRecord(value)          // value is TRecord
Type.IsRef(value)             // value is TRef
Type.IsCyclic(value)          // value is TCyclic
Type.IsOptional(value)        // value is TOptional
Type.IsReadonly(value)        // value is TReadonly
Type.IsImmutable(value)       // value is TImmutable
Type.IsTemplateLiteral(value) // value is TTemplateLiteral
Type.IsGeneric(value)         // value is TGeneric
Type.IsIdentifier(value)      // value is TIdentifier
Type.IsCall(value)            // value is TCall
Type.IsInfer(value)           // value is TInfer
Type.IsRest(value)            // value is TRest
Type.IsThis(value)            // value is TThis
```
