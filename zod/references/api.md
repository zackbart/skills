# Zod v4 — Complete Schema API Reference

## Primitives

```typescript
import * as z from "zod";

z.string();
z.number();
z.bigint();
z.boolean();
z.symbol();
z.undefined();
z.null();
```

### Coercion

Coerce input to the appropriate type using the built-in JavaScript constructors:

```typescript
z.coerce.string();    // String(input)
z.coerce.number();    // Number(input)
z.coerce.boolean();   // Boolean(input) — falsy→false, truthy→true
z.coerce.bigint();    // BigInt(input)
```

Input type is `unknown` by default. Constrain with a generic:

```typescript
const B = z.coerce.number<number>();
type BInput = z.input<typeof B>; // number
```

## Literals

```typescript
z.literal("tuna");
z.literal(12);
z.literal(2n);
z.literal(true);

// Multiple values (new in v4)
z.literal([200, 201, 204]);

// Extract allowed values
z.literal(["red", "green"]).values; // Set<"red" | "green">
```

Use `z.null()` and `z.undefined()` for those JS literals.

## Strings

### Validations

```typescript
z.string().max(5);
z.string().min(5);
z.string().length(5);
z.string().regex(/^[a-z]+$/);
z.string().startsWith("aaa");
z.string().endsWith("zzz");
z.string().includes("---");
z.string().uppercase();
z.string().lowercase();
```

### Transforms (overwrites — type unchanged)

```typescript
z.string().trim();
z.string().toLowerCase();
z.string().toUpperCase();
z.string().normalize();
```

## String Formats

All formats are top-level functions. The method forms (`z.string().email()`) are deprecated.

```typescript
z.email();
z.uuid();
z.url();
z.httpUrl();         // http or https only
z.hostname();
z.emoji();           // single emoji character
z.base64();
z.base64url();
z.hex();
z.jwt();
z.nanoid();
z.cuid();
z.cuid2();
z.ulid();
z.ipv4();
z.ipv6();
z.mac();
z.cidrv4();
z.cidrv6();
z.hash("sha256");    // or "sha1", "sha384", "sha512", "md5"
z.iso.date();
z.iso.time();
z.iso.datetime();
z.iso.duration();
z.e164();            // phone numbers
```

### Email Options

```typescript
z.email();                                    // Zod default (Gmail-like rules)
z.email({ pattern: z.regexes.html5Email });   // browser input[type=email]
z.email({ pattern: z.regexes.rfc5322Email }); // RFC 5322
z.email({ pattern: z.regexes.unicodeEmail }); // Unicode-friendly
```

### UUID Options

```typescript
z.uuid();                    // any valid UUID (RFC 9562/4122)
z.uuid({ version: "v4" });  // specific version
z.uuidv4();                  // convenience
z.uuidv7();
z.guid();                    // any UUID-like 8-4-4-4-12 hex (less strict)
```

### URL Options

```typescript
z.url();                                   // any WHATWG URL
z.url({ hostname: /^example\.com$/ });     // constrain hostname
z.url({ protocol: /^https$/ });            // constrain protocol

// Web URLs specifically:
z.url({ protocol: /^https?$/, hostname: z.regexes.domain });
// or use the convenience:
z.httpUrl();
```

### ISO Datetime Options

```typescript
z.iso.datetime();                          // UTC only, no offsets
z.iso.datetime({ offset: true });          // allow timezone offsets
z.iso.datetime({ local: true });           // allow unqualified (no timezone)
z.iso.datetime({ precision: 0 });          // second precision only
z.iso.datetime({ precision: 3 });          // millisecond precision
z.iso.datetime({ precision: -1 });         // minute precision (no seconds)
```

### ISO Time Options

```typescript
z.iso.time();                // HH:MM[:SS[.s+]] — no offsets allowed
z.iso.time({ precision: 0 });  // HH:MM:SS
z.iso.time({ precision: 3 });  // HH:MM:SS.sss
```

### MAC Address Options

```typescript
z.mac();                         // colon-delimited, e.g. "00:1A:2B:3C:4D:5E"
z.mac({ delimiter: "-" });       // dash-delimited
```

### JWT Options

```typescript
z.jwt();
z.jwt({ alg: "HS256" });
```

### Hash Options

```typescript
z.hash("sha256");                        // hex (default)
z.hash("sha256", { enc: "base64" });     // base64
z.hash("sha256", { enc: "base64url" });  // base64url
```

### Custom String Formats

```typescript
const coolId = z.stringFormat("cool-id", (val) => {
  return val.length === 100 && val.startsWith("cool-");
});
// or with regex:
z.stringFormat("cool-id", /^cool-[a-z0-9]{95}$/);
```

## Template Literals

```typescript
z.templateLiteral(["hello, ", z.string()]);
// `hello, ${string}`

z.templateLiteral([z.number(), z.enum(["px", "em", "rem"])]);
// `${number}px` | `${number}em` | `${number}rem`

z.templateLiteral([z.string().min(1), "@", z.string().max(64)]);
// `${string}@${string}` with refinements enforced
```

## Numbers

```typescript
z.number();              // any finite number (no NaN, no Infinity)

z.number().gt(5);
z.number().gte(5);       // alias: .min(5)
z.number().lt(5);
z.number().lte(5);       // alias: .max(5)
z.number().positive();   // > 0
z.number().nonnegative();
z.number().negative();
z.number().nonpositive();
z.number().multipleOf(5);  // alias: .step(5)

z.nan();                 // validates NaN specifically
```

### Integer & Numeric Formats

```typescript
z.int();       // safe integer range [MIN_SAFE_INTEGER, MAX_SAFE_INTEGER]
z.int32();     // [-2147483648, 2147483647]
z.uint32();    // [0, 4294967295]
z.float32();
z.float64();

// BigInt formats
z.int64();     // returns ZodBigInt
z.uint64();    // returns ZodBigInt
```

## BigInts

```typescript
z.bigint();
z.bigint().gt(5n);
z.bigint().gte(5n);    // alias: .min(5n)
z.bigint().lt(5n);
z.bigint().lte(5n);    // alias: .max(5n)
z.bigint().positive();
z.bigint().nonnegative();
z.bigint().negative();
z.bigint().nonpositive();
z.bigint().multipleOf(5n);
```

## Booleans

```typescript
z.boolean();
```

## Stringbools

Parse string boolean values from env vars or form data:

```typescript
z.stringbool();
// "true"/"1"/"yes"/"on"/"y"/"enabled" → true
// "false"/"0"/"no"/"off"/"n"/"disabled" → false

// Custom truthy/falsy
z.stringbool({
  truthy: ["yes", "true"],
  falsy: ["no", "false"],
});

// Case-sensitive
z.stringbool({ case: "sensitive" });
```

## Dates

```typescript
z.date();
z.date().min(new Date("1900-01-01"));
z.date().max(new Date());
```

## Enums

```typescript
// String array
z.enum(["Salmon", "Tuna", "Trout"]);

// Use `as const` for variables
const fish = ["Salmon", "Tuna"] as const;
z.enum(fish);

// TypeScript enum (replaces z.nativeEnum())
enum Color { Red = "red", Green = "green" }
z.enum(Color);

// Enum-like object
const Status = { Active: 0, Inactive: 1 } as const;
z.enum(Status);

// Methods
FishEnum.enum;                     // { Salmon: "Salmon", ... }
FishEnum.exclude(["Salmon"]);      // new enum without Salmon
FishEnum.extract(["Salmon"]);      // new enum with only Salmon
```

## Optionals / Nullables / Nullish

```typescript
z.optional(z.string());          // or z.string().optional()
z.nullable(z.string());          // or z.string().nullable()
z.nullish(z.string());           // optional + nullable

// Unwrap
optionalSchema.unwrap();         // inner schema
```

## Unknown / Any / Never

```typescript
z.any();
z.unknown();
z.never();
```

## Objects

```typescript
const Person = z.object({
  name: z.string(),
  age: z.number(),
});

// Unrecognized keys are stripped by default
Person.parse({ name: "Zack", age: 30, extra: true });
// => { name: "Zack", age: 30 }
```

### Strict & Loose Objects

```typescript
z.strictObject({ name: z.string() });   // errors on unknown keys
z.looseObject({ name: z.string() });    // passes through unknown keys
```

### Catchall

```typescript
z.object({ name: z.string() }).catchall(z.string());
// unknown keys must be strings
```

### Object Operations

```typescript
Schema.shape;                    // { name: ZodString, age: ZodNumber }
Schema.keyof();                  // ZodEnum<["name", "age"]>

Schema.extend({ email: z.email() });
Schema.safeExtend({ name: z.string().min(5) }); // type-safe extend, preserves refinements

Schema.pick({ name: true });
Schema.omit({ age: true });

Schema.partial();                // all optional
Schema.partial({ age: true });   // selective
Schema.required();               // all required
Schema.required({ name: true }); // selective
```

### Extend via Spread (best tsc performance)

```typescript
const Extended = z.object({
  ...Base.shape,
  ...Other.shape,
  newField: z.string(),
});
```

## Recursive Objects

Use getters — no type casting needed in v4:

```typescript
const Category = z.object({
  name: z.string(),
  get subcategories() { return z.array(Category); }
});
type Category = z.infer<typeof Category>;
// { name: string; subcategories: Category[] }
```

Mutually recursive:

```typescript
const User = z.object({
  email: z.email(),
  get posts() { return z.array(Post); }
});
const Post = z.object({
  title: z.string(),
  get author() { return User; }
});
```

If TS gives `ts(7023)` error, add return type annotation:

```typescript
get subcategories(): z.ZodNullable<z.ZodArray<typeof Category>> {
  return z.nullable(z.array(Category));
}
```

## Arrays

```typescript
z.array(z.string());     // or z.string().array()
z.array(z.string()).min(5);
z.array(z.string()).max(5);
z.array(z.string()).length(5);

stringArray.unwrap();    // inner schema
```

## Tuples

```typescript
z.tuple([z.string(), z.number(), z.boolean()]);
// [string, number, boolean]

// With rest element
z.tuple([z.string()], z.number());
// [string, ...number[]]
```

## Unions

```typescript
z.union([z.string(), z.number()]);
// string | number

schema.options; // [ZodString, ZodNumber]
```

### Exclusive Unions (XOR)

```typescript
z.xor([
  z.object({ type: z.literal("card"), cardNumber: z.string() }),
  z.object({ type: z.literal("bank"), accountNumber: z.string() }),
]);
// Fails if zero OR multiple options match
```

### Discriminated Unions

```typescript
z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("failed"), error: z.string() }),
]);
```

Supports unions, pipes, and nesting as discriminator values. Discriminated unions compose:

```typescript
z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.discriminatedUnion("code", [
    BaseError.extend({ code: z.literal(400) }),
    BaseError.extend({ code: z.literal(500) }),
  ])
]);
```

## Intersections

```typescript
z.intersection(SchemaA, SchemaB);
// Prefer .extend() for objects
```

## Records

Two arguments required in v4 (key schema + value schema):

```typescript
z.record(z.string(), z.number());
// Record<string, number>

// Enum keys → exhaustive (all keys required)
z.record(z.enum(["a", "b", "c"]), z.number());
// { a: number; b: number; c: number }

// Partial record (optional keys)
z.partialRecord(z.enum(["a", "b"]), z.string());
// { a?: string; b?: string }

// Loose record (pass through non-matching keys)
z.looseRecord(z.string().regex(/_phone$/), z.e164());

// Numeric keys (v4.2+)
z.record(z.number(), z.string());
z.record(z.int().min(0).max(10), z.string());
```

## Maps & Sets

```typescript
z.map(z.string(), z.number());   // Map<string, number>

z.set(z.number());               // Set<number>
z.set(z.string()).min(5).max(10).size(5);
```

## Files

```typescript
z.file();
z.file().min(10_000);            // minimum size (bytes)
z.file().max(1_000_000);         // maximum size (bytes)
z.file().mime("image/png");
z.file().mime(["image/png", "image/jpeg"]);
```

## Refinements

### .refine()

```typescript
z.string().refine(val => val.length <= 255);

// With options
z.string().refine(val => val.length > 8, {
  error: "Too short!",
  abort: true,       // stop further checks on failure
  path: ["field"],   // custom error path
});
```

Async refinements:

```typescript
z.string().refine(async (id) => {
  return await db.exists(id);
});
// Must use .parseAsync()
```

### .superRefine()

Create multiple issues with specific codes:

```typescript
z.array(z.string()).superRefine((val, ctx) => {
  if (val.length !== new Set(val).size) {
    ctx.addIssue({ code: "custom", message: "No duplicates", input: val });
  }
});
```

### .check()

Low-level refinement API (used heavily in Zod Mini):

```typescript
z.array(z.number()).check(
  z.minLength(5),
  z.maxLength(10),
  z.refine(arr => arr.includes(5))
);
```

### Conditional Refinements (when)

```typescript
z.object({ password: z.string().min(8), confirm: z.string() })
  .refine(data => data.password === data.confirm, {
    error: "Passwords don't match",
    path: ["confirm"],
    when(payload) {
      return schema.pick({ password: true, confirm: true })
        .safeParse(payload.value).success;
    },
  });
```

### .overwrite()

Transform that doesn't change the inferred type (returns original class):

```typescript
z.number().overwrite(val => val ** 2).max(100);
// Still ZodNumber, not ZodPipe
```

`.trim()`, `.toLowerCase()`, `.toUpperCase()` use `.overwrite()` internally.

## Transforms

```typescript
// Standalone transform
z.transform(input => String(input));

// Chained (returns ZodPipe)
z.string().transform(val => val.length);

// Async
z.string().transform(async (id) => await db.getUserById(id));
// Must use .parseAsync()

// Error reporting inside transform
z.transform((val, ctx) => {
  if (typeof val !== "string") {
    ctx.issues.push({ code: "custom", message: "Expected string", input: val });
    return z.NEVER;
  }
  return val.length;
});
```

### .preprocess()

```typescript
z.preprocess(val => typeof val === "string" ? parseInt(val) : val, z.int());
```

## Pipes

Chain schemas together:

```typescript
z.string().pipe(z.coerce.number()).pipe(z.number().min(0));
```

## Codecs (Bidirectional Transforms)

```typescript
const stringToDate = z.codec(
  z.iso.datetime(),  // input schema
  z.date(),          // output schema
  {
    decode: (s) => new Date(s),
    encode: (d) => d.toISOString(),
  }
);

stringToDate.parse("2024-01-15T10:30:00.000Z"); // Date (decode/forward)
z.decode(stringToDate, "2024-01-15T...");        // Date (strongly typed input)
z.encode(stringToDate, new Date());               // ISO string (reverse)
```

## Defaults & Prefaults

```typescript
// Default: short-circuits parsing, value must match OUTPUT type
z.string().default("tuna");
z.number().default(Math.random);  // function for dynamic defaults

// Prefault: value is PARSED, must match INPUT type
z.string().transform(val => val.length).prefault("tuna");
// parse(undefined) => 4 (prefault is parsed through transform)
```

## Catch

Fallback value on validation failure:

```typescript
z.number().catch(42);
z.number().catch((ctx) => {
  ctx.error; // the ZodError
  return Math.random();
});
```

## Branded Types

Simulate nominal typing:

```typescript
const USD = z.string().brand<"USD">();
type USD = z.infer<typeof USD>; // string & z.$brand<"USD">

// Brand direction (v4.2+)
z.string().brand<"Cat", "out">();    // output branded (default)
z.string().brand<"Cat", "in">();     // input branded
z.string().brand<"Cat", "inout">(); // both branded
```

## Readonly

```typescript
z.object({ name: z.string() }).readonly();
// Readonly<{ name: string }> — result is Object.freeze()'d
```

## JSON

```typescript
z.json(); // validates any JSON-encodable value
```

## Functions

```typescript
const MyFn = z.function({
  input: [z.string()],   // array or ZodTuple
  output: z.number(),    // optional
});

const impl = MyFn.implement((input) => input.trim().length);
const asyncImpl = MyFn.implementAsync(async (input) => input.trim().length);
```

## Custom Types

```typescript
const px = z.custom<`${number}px`>(val =>
  typeof val === "string" ? /^\d+px$/.test(val) : false
);
```

## Instanceof

```typescript
z.instanceof(URL);
z.instanceof(URL).check(
  z.property("protocol", z.literal("https:" as string))
);
```

## .apply()

Incorporate external functions into method chains:

```typescript
schema.apply(externalFunction);
```

## Parsing Methods

```typescript
schema.parse(data);              // throws ZodError
schema.safeParse(data);          // { success, data/error }
await schema.parseAsync(data);
await schema.safeParseAsync(data);
```

## Type Inference

```typescript
type T = z.infer<typeof schema>;    // output type
type T = z.output<typeof schema>;   // same as infer
type T = z.input<typeof schema>;    // input type
```
