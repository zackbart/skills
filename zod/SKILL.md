---
name: zod
description: >
  Zod v4 TypeScript-first validation library assistant. Use when the user writes schemas with Zod,
  imports from 'zod' or 'zod/mini', defines validation schemas using z.object / z.string / z.number /
  z.array / z.enum etc., parses data with .parse / .safeParse, uses z.infer for type inference, builds
  discriminated unions, converts to JSON Schema with z.toJSONSchema, customizes errors, defines
  recursive objects, uses z.coerce for input coercion, creates codecs with z.codec, uses string
  formats like z.email / z.uuid / z.url, or asks about Zod migration from v3 to v4. Also trigger
  when the user mentions zod, asks how to validate data with TypeScript inference, writes code using
  z.* patterns, uses Zod with forms (react-hook-form), tRPC, or AI structured outputs. Covers all
  schema types, refinements, transforms, pipes, error handling, JSON Schema, metadata/registries,
  Zod Mini, codecs, and the v3-to-v4 migration.
---

# Zod v4 — TypeScript-First Validation

> **Docs version**: Based on Zod v4.x (stable) documentation as of 2025-07-10, including features through v4.2 (codecs, numeric record keys, brand direction). Source: https://zod.dev

You are an expert at data validation using Zod v4. Zod is a TypeScript-first schema declaration and validation library that infers static types from runtime schemas, providing both compile-time safety and runtime validation with zero external dependencies.

## Core Principles

1. **TypeScript-first** — Every schema infers a static TypeScript type via `z.infer<typeof schema>`. Input and output types can diverge (use `z.input<>` / `z.output<>`).
2. **Immutable API** — All schema methods return a new instance. Schemas are never mutated in place.
3. **Parse, don't validate** — Use `.parse()` (throws `ZodError`) or `.safeParse()` (returns `{ success, data/error }`) to validate data. Parsed results are deep clones.
4. **Refinements live inside schemas** — Unlike v3, `.refine()` does not wrap schemas in `ZodEffects`. You can freely chain `.refine()` with `.min()`, `.max()`, etc.
5. **Top-level string formats** — Use `z.email()`, `z.uuid()`, `z.url()` etc. instead of `z.string().email()` (deprecated).
6. **Unified error param** — Use `error` (string or function) instead of v3's `message`, `invalid_type_error`, `required_error`, or `errorMap`.
7. **Requires TypeScript strict mode** and TS >= 5.5.

## How to Use This Skill

When the user needs Zod help, consult the reference files for detailed patterns:

- **`references/api.md`** — Complete schema API: primitives, strings, string formats, numbers, objects, arrays, tuples, unions, discriminated unions, records, maps, sets, enums, files, refinements, transforms, pipes, codecs, defaults, branded types, recursive objects, template literals, functions
- **`references/errors-and-metadata.md`** — Error customization (schema-level, per-parse, global), error formatting (`z.treeifyError`, `z.prettifyError`, `z.flattenError`), internationalization/locales, metadata registries (`.meta()`, `z.globalRegistry`, custom registries), JSON Schema conversion (`z.toJSONSchema`, `z.fromJSONSchema`)
- **`references/migration.md`** — Complete v3-to-v4 migration guide: breaking changes, renamed APIs, updated generics, Zod Mini, library author guidance, `zod/v4/core` for dual support

Read the relevant reference file before generating code.

## Quick Reference

### Installation & Import

```typescript
npm install zod    // v4 is the default

import * as z from "zod";       // full API
import * as z from "zod/mini";  // tree-shakable functional API (1.88kb gzipped core)
```

### Defining Schemas

```typescript
// Primitives
z.string();  z.number();  z.boolean();  z.bigint();
z.null();    z.undefined();  z.date();

// String formats (top-level, NOT z.string().email())
z.email();  z.uuid();  z.url();  z.ipv4();  z.ipv6();
z.iso.date();  z.iso.datetime();  z.iso.time();  z.iso.duration();

// Numbers & integers
z.int();  z.int32();  z.float32();  z.float64();

// Coercion
z.coerce.string();  z.coerce.number();  z.coerce.boolean();

// Objects
z.object({ name: z.string(), age: z.number() });
z.strictObject({ ... });   // errors on unknown keys
z.looseObject({ ... });    // passes through unknown keys

// Arrays, tuples, unions
z.array(z.string());
z.tuple([z.string(), z.number()]);
z.union([z.string(), z.number()]);
z.discriminatedUnion("status", [ ... ]);

// Records
z.record(z.string(), z.number());        // Record<string, number>
z.partialRecord(z.enum(["a","b"]), z.string());  // optional keys

// Enums (supports TS enums, objects, and string arrays)
z.enum(["A", "B", "C"]);
z.enum(MyTypeScriptEnum);   // replaces z.nativeEnum()

// Literals (single or multiple)
z.literal("hello");
z.literal([200, 201, 204]);
```

### Parsing

```typescript
schema.parse(data);           // throws ZodError on failure
schema.safeParse(data);       // { success: true, data } | { success: false, error }
await schema.parseAsync(data);       // for async refinements/transforms
await schema.safeParseAsync(data);
```

### Type Inference

```typescript
type User = z.infer<typeof UserSchema>;     // output type
type UserInput = z.input<typeof UserSchema>; // input type (may differ with transforms)
```

### Object Operations

```typescript
Schema.extend({ newField: z.string() });
Schema.pick({ name: true });
Schema.omit({ age: true });
Schema.partial();                  // all optional
Schema.partial({ age: true });     // selective
Schema.required();
Schema.keyof();                    // ZodEnum from keys
```

### Refinements & Transforms

```typescript
// Refinements (stay inside schema, don't wrap)
z.string().refine(val => val.includes("@"), { error: "Must contain @" });
z.string().superRefine((val, ctx) => { ctx.addIssue({ ... }); });

// Transforms
z.string().transform(val => val.length);  // string -> number
z.string().trim().toLowerCase();           // overwrites (type unchanged)

// Pipes
z.string().pipe(z.coerce.number()).pipe(z.number().min(0));
```

### Error Customization

```typescript
// Inline error
z.string({ error: "Must be a string" });
z.string().min(5, { error: "Too short" });

// Error function (replaces v3 errorMap, invalid_type_error, required_error)
z.string({ error: (iss) => iss.input === undefined ? "Required" : "Invalid" });

// Per-parse
schema.parse(data, { error: (iss) => "Custom" });

// Global
z.config({ customError: (iss) => "Global custom" });

// Locales
z.config(z.locales.en());  // or fr, de, es, ja, zhCN, etc.

// Formatting
z.prettifyError(error);     // human-readable string
z.treeifyError(error);      // nested tree structure
z.flattenError(error);      // { formErrors, fieldErrors }
```

### JSON Schema

```typescript
z.toJSONSchema(schema);                          // default: draft-2020-12
z.toJSONSchema(schema, { target: "draft-07" });  // or "draft-04", "openapi-3.0"
z.toJSONSchema(schema, { io: "input" });         // input type instead of output
z.fromJSONSchema(jsonSchema);                    // experimental: JSON Schema -> Zod
```

### Metadata & Registries

```typescript
schema.meta({ title: "Name", description: "User name" });  // adds to z.globalRegistry
schema.meta();  // retrieve metadata

const myRegistry = z.registry<{ description: string }>();
myRegistry.add(schema, { description: "..." });
```

### Recursive Objects (v4 native — no type casting needed)

```typescript
const Category = z.object({
  name: z.string(),
  get subcategories() { return z.array(Category); }
});
type Category = z.infer<typeof Category>;
// { name: string; subcategories: Category[] }
```

### Codecs (bidirectional transforms)

```typescript
const stringToDate = z.codec(z.iso.datetime(), z.date(), {
  decode: (s) => new Date(s),
  encode: (d) => d.toISOString(),
});
stringToDate.parse("2024-01-15T10:30:00.000Z");  // Date
z.encode(stringToDate, new Date());                // ISO string
```

### Zod Mini (tree-shakable variant)

```typescript
import * as z from "zod/mini";

z.optional(z.string());               // functional wrappers
z.extend(z.object({ ... }), { age: z.number() });
z.array(z.number()).check(z.minLength(5), z.maxLength(10));
```

## Key v4 Changes (vs v3)

| v3 | v4 |
|---|---|
| `z.string().email()` | `z.email()` (top-level) |
| `z.nativeEnum(E)` | `z.enum(E)` |
| `{ message: "..." }` | `{ error: "..." }` |
| `z.object({}).strict()` | `z.strictObject({})` |
| `z.object({}).passthrough()` | `z.looseObject({})` |
| `.merge()` | `.extend(other.shape)` or spread `{ ...A.shape, ...B.shape }` |
| `z.record(valueSchema)` | `z.record(keySchema, valueSchema)` (two args required) |
| `.format()` / `.flatten()` | `z.treeifyError()` / `z.flattenError()` |
| `._def` | `._zod.def` |
| `ZodEffects` (wrapping) | Refinements stored inside schemas |

## Performance

- 14x faster string parsing, 7x faster array parsing, 6.5x faster object parsing vs v3
- 100x reduction in `tsc` type instantiations
- Core bundle: 5.36kb gzipped (regular), 1.88kb gzipped (mini) vs 12.47kb in v3
