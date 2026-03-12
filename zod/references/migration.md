# Zod v3 to v4 Migration Guide

## Installation

```bash
npm install zod@^4.0.0
```

Community codemod available: [zod-v3-to-v4](https://github.com/nicoespeon/zod-v3-to-v4)

## Error Customization (Highest Impact)

### `message` → `error`

```typescript
// v3
z.string().min(5, { message: "Too short." });
// v4
z.string().min(5, { error: "Too short." });
```

`message` still works but is deprecated.

### `invalid_type_error` / `required_error` → `error` function

```typescript
// v3
z.string({
  required_error: "Required",
  invalid_type_error: "Not a string",
});
// v4
z.string({
  error: (issue) => issue.input === undefined ? "Required" : "Not a string"
});
```

### `errorMap` → `error` function

```typescript
// v3
z.string({
  errorMap: (issue, ctx) => {
    if (issue.code === "too_small") return { message: `Min ${issue.minimum}` };
    return { message: ctx.defaultError };
  },
});
// v4
z.string({
  error: (issue) => {
    if (issue.code === "too_small") return `Min ${issue.minimum}`;
    // return undefined to defer to default
  },
});
```

Error functions can now return a plain `string` or `undefined`.

### Error Precedence Changed

In v3, per-parse error maps had higher priority than schema-level. In v4, schema-level wins:

```typescript
const schema = z.string({ error: "Schema-level" });
schema.parse(12, { error: () => "Per-parse" });
// v3: "Per-parse"
// v4: "Schema-level"
```

### `.format()` / `.flatten()` Deprecated

```typescript
// v3
error.format();
error.flatten();
// v4
z.treeifyError(error);    // replaces .format()
z.flattenError(error);    // replaces .flatten()
```

### `.addIssue()` / `.addIssues()` Deprecated

Push directly to `err.issues` array instead.

## ZodError Issue Types

Dramatically streamlined. v4 issue types:

| v4 | v3 equivalent |
|---|---|
| `$ZodIssueInvalidType` | `ZodInvalidTypeIssue` |
| `$ZodIssueTooBig` | `ZodTooBigIssue` |
| `$ZodIssueTooSmall` | `ZodTooSmallIssue` |
| `$ZodIssueInvalidStringFormat` | `ZodInvalidStringIssue` |
| `$ZodIssueNotMultipleOf` | `ZodNotMultipleOfIssue` |
| `$ZodIssueUnrecognizedKeys` | `ZodUnrecognizedKeysIssue` |
| `$ZodIssueInvalidValue` | `ZodInvalidEnumValueIssue` + `ZodInvalidLiteralIssue` (merged) |
| `$ZodIssueInvalidUnion` | `ZodInvalidUnionIssue` |
| `$ZodIssueInvalidKey` | new (for z.record/z.map) |
| `$ZodIssueInvalidElement` | new (for z.map/z.set) |
| `$ZodIssueCustom` | `ZodCustomIssue` |

**Removed**: `ZodInvalidUnionDiscriminatorIssue` (throws Error at schema creation), `ZodInvalidArgumentsIssue`, `ZodInvalidReturnTypeIssue` (z.function throws ZodError directly), `ZodInvalidDateIssue` (merged into invalid_type), `ZodInvalidIntersectionTypesIssue` (throws regular Error), `ZodNotFiniteIssue` (infinite values are invalid_type).

## String Formats: Methods → Top-Level

```typescript
// v3 (deprecated)
z.string().email();
z.string().uuid();
z.string().url();
z.string().ip();
z.string().cidr();

// v4
z.email();
z.uuid();         // stricter RFC 9562/4122 validation
z.guid();         // permissive UUID-like (8-4-4-4-12 hex)
z.url();
z.ipv4();         // separate (no z.string().ip())
z.ipv6();
z.cidrv4();       // separate (no z.string().cidr())
z.cidrv6();
```

Method forms still work but are deprecated and will be removed in v5.

### z.uuid() is Stricter

v4 validates variant bits per RFC spec. Use `z.guid()` for the old permissive behavior.

### z.base64url() No Padding

Padding is no longer allowed in `z.base64url()`.

## z.number() Changes

- `Infinity` and `-Infinity` are no longer valid
- `.int()` only accepts safe integers (MIN_SAFE_INTEGER to MAX_SAFE_INTEGER)
- `.safe()` behaves identically to `.int()` (no longer accepts floats)

## z.coerce Changes

Input type of all `z.coerce` schemas is now `unknown` (was the target type in v3).

## z.object() Changes

### `.strict()` / `.passthrough()` Deprecated

```typescript
// v3
z.object({}).strict();
z.object({}).passthrough();
// v4
z.strictObject({});
z.looseObject({});
```

Methods still work but are considered legacy.

### `.strip()` Deprecated, `.nonstrict()` Removed

Use `z.object(A.shape)` to convert a strict object to regular.

### `.deepPartial()` Removed

No direct replacement. Was an anti-pattern.

### `.merge()` Deprecated → Use `.extend()` or Spread

```typescript
// v3
A.merge(B);
// v4
A.extend(B.shape);
// or (best tsc performance):
z.object({ ...A.shape, ...B.shape });
```

### `z.unknown()` / `z.any()` No Longer Optional in Objects

```typescript
z.object({ a: z.any(), b: z.unknown() });
// v3: { a?: any; b?: unknown }
// v4: { a: any; b: unknown }
```

### Defaults Applied in Optional Fields

```typescript
z.object({ a: z.string().default("tuna").optional() }).parse({});
// v3: {}
// v4: { a: "tuna" }
```

## z.enum() Absorbs z.nativeEnum()

```typescript
// v3
z.nativeEnum(MyEnum);
// v4
z.enum(MyEnum);
```

`z.nativeEnum()` is deprecated.

Removed properties: `.Enum`, `.Values` (use `.enum` only).

## z.record() Requires Two Arguments

```typescript
// v3
z.record(z.string());        // ✅
// v4
z.record(z.string());        // ❌
z.record(z.string(), z.string()); // ✅
```

### Enum Keys Now Exhaustive

```typescript
z.record(z.enum(["a","b"]), z.number());
// v3: { a?: number; b?: number }
// v4: { a: number; b: number }

// For optional keys, use z.partialRecord():
z.partialRecord(z.enum(["a","b"]), z.number());
```

## .default() Behavior Changed

Default value must match OUTPUT type (short-circuits parsing):

```typescript
// v3: default matches INPUT type, gets parsed
z.string().transform(val => val.length).default("tuna");
// parse(undefined) => 4

// v4: default matches OUTPUT type, returned directly
z.string().transform(val => val.length).default(0);
// parse(undefined) => 0

// v4: use .prefault() for v3 behavior
z.string().transform(val => val.length).prefault("tuna");
// parse(undefined) => 4
```

## z.array().nonempty() Type Change

```typescript
z.array(z.string()).nonempty();
// v3 type: [string, ...string[]]
// v4 type: string[] (behaves like .min(1))

// For tuple-style non-empty:
z.tuple([z.string()], z.string()); // [string, ...string[]]
```

## z.function() Redesigned

No longer a Zod schema. New API:

```typescript
// v3
z.function().args(z.string()).returns(z.number());
// v4
z.function({ input: [z.string()], output: z.number() });
```

Use `.implement()` / `.implementAsync()` to create functions.

## z.promise() Deprecated

Just `await` the value before parsing.

## .refine() Changes

### Type Predicates Ignored

```typescript
z.unknown().refine((val): val is string => typeof val === "string");
// v3 inferred type: string
// v4 inferred type: unknown
```

### ctx.path Removed

No longer available in `.superRefine()` callbacks.

### Function as Second Argument Removed

```typescript
// v3 (removed)
z.string().refine(val => val.length > 10, val => ({ message: `${val} too short` }));
```

## z.literal() — No Symbol Support

Symbols are no longer valid literal values.

## z.intersection() — Error Change

Unmergeable intersection results throw a regular `Error` instead of `ZodError`.

## Dropped APIs

| API | Status |
|---|---|
| `z.ostring()`, `z.onumber()`, etc. | Removed |
| `ZodClass.create()` static methods | Removed |
| `.formErrors` | Removed (use `z.flattenError()`) |
| `ZodTypeAny` | Use `z.ZodType` |

## Internal Changes (For Library Authors)

### Generics Updated

```typescript
// v3
class ZodType<Output, Def extends z.ZodTypeDef, Input = Output> {}
// v4
class ZodType<Output = unknown, Input = unknown> {}
```

`Input` now defaults to `unknown` (was `Output`).

### ._def → ._zod.def

```typescript
// v3
schema._def;
// v4
schema._zod.def;
```

### ZodEffects Removed

Refinements stored inside schemas. Transforms use `ZodTransform` + `ZodPipe`:

```typescript
z.string().transform(val => val.length);
// v4 type: ZodPipe<ZodString, ZodTransform>
```

### ZodPreprocess → ZodPipe

```typescript
z.preprocess(fn, schema);
// v4 type: ZodPipe<ZodTransform, Schema>
```

### ZodBranded Removed

Branding modifies inferred type directly.

### z.core Namespace

Shared types between Zod and Zod Mini:

```typescript
import * as z from "zod";
z.core.$ZodError;
z.core.$ZodType;
// etc.
```

---

## Zod Mini

Tree-shakable functional variant sharing `zod/v4/core`:

```typescript
import * as z from "zod/mini";

// Functional wrappers instead of methods
z.optional(z.string());
z.union([z.string(), z.number()]);
z.extend(z.object({ ... }), { age: z.number() });

// .check() for refinements
z.array(z.number()).check(
  z.minLength(5),
  z.maxLength(10),
  z.refine(arr => arr.includes(5))
);

// Parsing methods are identical
z.string().parse("asdf");
z.string().safeParse("asdf");
```

Core bundle: 1.88kb gzipped (vs 5.36kb regular, 12.47kb v3).

Zod Mini does **not** load any locale by default.

---

## Library Author Guide

### Peer Dependencies

```json
{
  "peerDependencies": {
    "zod": "^3.25.0 || ^4.0.0"
  }
}
```

### Import from Stable Subpaths Only

- `"zod/v3"` — Zod 3 permalink
- `"zod/v4/core"` — Zod 4 Core (works with both Zod and Zod Mini)
- **Do NOT** import from `"zod"` (changes between major versions), `"zod/v4"`, or `"zod/v4/mini"`

### Detect Zod Version at Runtime

```typescript
if ("_zod" in schema) {
  // Zod 4
  schema._zod.def;
} else {
  // Zod 3
  schema._def;
}
```

### Accept User Schemas Correctly

```typescript
import * as z4 from "zod/v4/core";

// ❌ Wrong — loses subclass info
function bad<T>(schema: z4.$ZodType<T>) { return schema; }

// ✅ Correct — preserves subclass
function good<T extends z4.$ZodType>(schema: T) { return schema; }

// Constrain to specific subclass
function objectOnly<T extends z4.$ZodObject>(schema: T) { return schema; }

// Constrain output type
function stringsOnly<T extends z4.$ZodType<string>>(schema: T) { return schema; }
```

### Parse with Core

```typescript
import * as z4 from "zod/v4/core";
z4.parse(schema, data);
z4.safeParse(schema, data);
```

### Consider Standard Schema

If you only need black-box validation (not Zod-specific features), use [Standard Schema](https://standardschema.dev/) for broader library compatibility.
