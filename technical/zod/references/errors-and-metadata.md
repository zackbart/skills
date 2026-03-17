# Zod v4 — Errors, Metadata, JSON Schema & Registries

## Error Customization

### Schema-Level Errors (Highest Priority)

Pass `error` as a string or function to any schema or check:

```typescript
// String shorthand
z.string("Not a string!");
z.string().min(5, "Too short!");
z.email("Invalid email!");
z.array(z.string(), "Not an array!");

// Object form
z.string({ error: "Bad!" });
z.string().min(5, { error: "Too short!" });
```

### Error Function (replaces v3 errorMap, invalid_type_error, required_error)

```typescript
z.string({
  error: (iss) => iss.input === undefined
    ? "This field is required"
    : "Not a string"
});
```

The `iss` object provides context:

```typescript
z.string({
  error: (iss) => {
    iss.code;   // issue code
    iss.input;  // the input data
    iss.inst;   // the schema/check that originated the issue
    iss.path;   // error path
  }
});

// Check-specific properties
z.string().min(5, {
  error: (iss) => {
    iss.minimum;    // the minimum value
    iss.inclusive;   // whether inclusive
    return `Must be at least ${iss.minimum} chars`;
  }
});
```

Return `undefined` to fall back to default message:

```typescript
z.int64({
  error: (issue) => {
    if (issue.code === "too_big") {
      return `Value must be <${issue.maximum}`;
    }
    return undefined; // defer to default
  }
});
```

### Per-Parse Errors (Lower Priority Than Schema-Level)

```typescript
schema.parse(data, {
  error: (iss) => "Per-parse custom error"
});
```

Schema-level errors take precedence:

```typescript
const schema = z.string({ error: "Schema wins" });
schema.parse(12, { error: () => "This loses" });
// Error message: "Schema wins"
```

### Global Error Map (Lowest Priority)

```typescript
z.config({
  customError: (iss) => {
    if (iss.code === "invalid_type") {
      return `Expected ${iss.expected}`;
    }
    if (iss.code === "too_small") {
      return `Minimum is ${iss.minimum}`;
    }
  }
});
```

### Error Precedence (High → Low)

1. Schema-level `error`
2. Per-parse `error`
3. Global `z.config({ customError })`
4. Locale error map

### Include Input in Issues

By default, input data is excluded from issues (privacy). Opt in:

```typescript
z.string().parse(12, { reportInput: true });
// Issue includes: { input: 12, ... }
```

## Internationalization (Locales)

```typescript
import * as z from "zod";

z.config(z.locales.en());  // English (loaded by default in regular zod)
z.config(z.locales.fr());  // French
z.config(z.locales.de());  // German
// etc.
```

Lazy loading:

```typescript
async function loadLocale(locale: string) {
  const { default: loc } = await import(`zod/v4/locales/${locale}.js`);
  z.config(loc());
}
await loadLocale("fr");
```

### Available Locales

ar, az, be, bg, ca, cs, da, de, en, eo, es, fa, fi, fr, frCA, he, hu, hy, id, is, it, ja, ka, km, ko, lt, mk, ms, nl, no, ota, ps, pl, pt, ru, sl, sv, ta, th, tr, uk, ur, uz, vi, zhCN, zhTW, yo

**Note**: Zod Mini does not load any locale by default — all errors default to "Invalid input".

## Error Formatting

### z.prettifyError()

Human-readable multi-line string:

```typescript
z.prettifyError(error);
// ✖ Unrecognized key: "extraField"
// ✖ Invalid input: expected string, received number
//   → at username
// ✖ Invalid input: expected number, received string
//   → at favoriteNumbers[1]
```

### z.treeifyError()

Nested tree structure mirroring the schema:

```typescript
const tree = z.treeifyError(error);
tree.errors;                                    // top-level errors
tree.properties?.username?.errors;              // field errors
tree.properties?.favoriteNumbers?.items?.[1]?.errors;  // nested array errors
```

### z.flattenError()

Shallow error object (good for flat form schemas):

```typescript
const flat = z.flattenError(error);
flat.formErrors;             // string[] — top-level errors
flat.fieldErrors;            // { [key: string]: string[] }
flat.fieldErrors.username;   // ["Invalid input: expected string..."]
```

## Issue Types

```typescript
z.core.$ZodIssueInvalidType
z.core.$ZodIssueTooBig
z.core.$ZodIssueTooSmall
z.core.$ZodIssueInvalidStringFormat
z.core.$ZodIssueNotMultipleOf
z.core.$ZodIssueUnrecognizedKeys
z.core.$ZodIssueInvalidValue
z.core.$ZodIssueInvalidUnion
z.core.$ZodIssueInvalidKey       // for z.record/z.map
z.core.$ZodIssueInvalidElement   // for z.map/z.set
z.core.$ZodIssueCustom
```

All issues share a base interface:

```typescript
interface $ZodIssueBase {
  readonly code?: string;
  readonly input?: unknown;
  readonly path: PropertyKey[];
  readonly message: string;
}
```

---

## Metadata & Registries

### .meta()

Register metadata in `z.globalRegistry`:

```typescript
const emailSchema = z.email().meta({
  id: "email_address",
  title: "Email address",
  description: "Your email address",
  examples: ["user@example.com"],
});

// Retrieve
emailSchema.meta(); // => { id: "email_address", ... }
```

Metadata is per-instance (immutable API returns new instances):

```typescript
const A = z.string().meta({ description: "A" });
const B = A.refine(_ => true);
B.meta(); // => undefined (B is a new instance)
```

### .describe()

Shorthand for `{ description: "..." }` (v3 compat):

```typescript
z.string().describe("A name");
// equivalent to: z.string().meta({ description: "A name" })
```

### z.globalRegistry

Accepts common JSON Schema metadata:

```typescript
interface GlobalMeta {
  id?: string;
  title?: string;
  description?: string;
  deprecated?: boolean;
  [k: string]: unknown;
}
```

Augment globally with declaration merging:

```typescript
// zod.d.ts
declare module "zod" {
  interface GlobalMeta {
    examples?: unknown[];
  }
}
export {}
```

### Custom Registries

```typescript
const myRegistry = z.registry<{ description: string }>();

myRegistry.add(schema, { description: "..." });
myRegistry.has(schema);    // boolean
myRegistry.get(schema);    // metadata
myRegistry.remove(schema);
myRegistry.clear();

// Inline with .register()
z.string().register(myRegistry, { description: "A name" });
```

Registries with no metadata type work as collections:

```typescript
const col = z.registry();
col.add(z.string());
```

### Advanced Registry Patterns

Reference inferred types in metadata:

```typescript
type MyMeta = { examples: z.$output[] };
const myRegistry = z.registry<MyMeta>();
myRegistry.add(z.string(), { examples: ["hello"] }); // ✅
myRegistry.add(z.number(), { examples: [1, 2] });    // ✅
```

Constrain schema types:

```typescript
const strOnly = z.registry<{ desc: string }, z.ZodString>();
strOnly.add(z.string(), { desc: "ok" }); // ✅
strOnly.add(z.number(), { desc: "no" }); // ❌ type error
```

**Note**: Duplicate `id` values in any registry throw an Error.

---

## JSON Schema Conversion

### z.toJSONSchema()

```typescript
import * as z from "zod";

z.toJSONSchema(z.object({ name: z.string(), age: z.number() }));
// => {
//   type: "object",
//   properties: { name: { type: "string" }, age: { type: "number" } },
//   required: ["name", "age"],
//   additionalProperties: false,
// }
```

### Options

```typescript
z.toJSONSchema(schema, {
  target: "draft-2020-12",    // default; or "draft-07", "draft-04", "openapi-3.0"
  io: "output",               // default; or "input"
  unrepresentable: "throw",   // default; or "any" (maps to {})
  cycles: "ref",               // default; or "throw"
  reused: "inline",            // default; or "ref" (extract to $defs)
  metadata: z.globalRegistry,  // or custom registry
  uri: (id) => `https://example.com/${id}.json`,
  override: (ctx) => {
    ctx.zodSchema;   // original Zod schema
    ctx.jsonSchema;  // generated JSON Schema (mutate directly)
  },
});
```

### Unrepresentable Types

These have no JSON Schema analog and throw by default:

```typescript
z.bigint(); z.int64(); z.symbol(); z.undefined(); z.void();
z.date(); z.map(); z.set(); z.transform(); z.nan(); z.custom();
```

Use `unrepresentable: "any"` to convert to `{}`, or `override` for custom handling:

```typescript
z.toJSONSchema(z.date(), {
  unrepresentable: "any",
  override: (ctx) => {
    if (ctx.zodSchema._zod.def.type === "date") {
      ctx.jsonSchema.type = "string";
      ctx.jsonSchema.format = "date-time";
    }
  },
});
```

### String Format Conversion

```typescript
z.email();          // => { type: "string", format: "email" }
z.iso.datetime();   // => { type: "string", format: "date-time" }
z.uuid();           // => { type: "string", format: "uuid" }
z.url();            // => { type: "string", format: "uri" }
z.base64();         // => { type: "string", contentEncoding: "base64" }
```

### Object Schema Conversion

- `z.object()` → `additionalProperties: false` (default)
- `z.looseObject()` → no `additionalProperties` constraint
- `z.strictObject()` → always `additionalProperties: false`
- `io: "input"` mode omits `additionalProperties`

### File Schema Conversion

```typescript
z.file();
// => { type: "string", format: "binary", contentEncoding: "binary" }

z.file().min(1).max(1024*1024).mime("image/png");
// => { ..., contentMediaType: "image/png", minLength: 1, maxLength: 1048576 }
```

### Registry-Based Multi-Schema Conversion

```typescript
z.globalRegistry.add(User, { id: "User" });
z.globalRegistry.add(Post, { id: "Post" });

z.toJSONSchema(z.globalRegistry);
// => { schemas: { User: { ... }, Post: { ... } } }
// Cross-references use $ref

z.toJSONSchema(z.globalRegistry, {
  uri: (id) => `https://api.example.com/schemas/${id}.json`
});
```

### z.fromJSONSchema() (Experimental)

```typescript
const zodSchema = z.fromJSONSchema({
  type: "object",
  properties: { name: { type: "string" }, age: { type: "number" } },
  required: ["name", "age"],
});
```
