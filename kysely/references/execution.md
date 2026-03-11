# Kysely Execution Reference

Execution internals and debugging patterns from Kysely execution docs and API docs.

## Table of Contents

1. [Execution Pipeline](#execution-pipeline)
2. [Compile Without Executing](#compile-without-executing)
3. [Execute a Compiled Query](#execute-a-compiled-query)
4. [Inspect Query Plans](#inspect-query-plans)
5. [Stream Large Result Sets](#stream-large-result-sets)
6. [Plugin Hook Points](#plugin-hook-points)
7. [Lifecycle Mental Model](#lifecycle-mental-model)
8. [Debugging Checklist](#debugging-checklist)
9. [Common Mistakes](#common-mistakes)

---

## Execution Pipeline

Kysely query builders are immutable objects that build an operation tree (AST). Execution follows this flow:

1. Build query AST with fluent APIs (`selectFrom`, `where`, `join`, ...).
2. Apply plugin query transforms (`transformQuery`).
3. Compile to SQL + parameter array for the active dialect.
4. Execute through the dialect driver/adapter.
5. Apply plugin result transforms (`transformResult`).

This model explains why plugin ordering, dialect, and driver behavior can change outcomes even when query builder code looks the same.

---

## Compile Without Executing

Use `.compile()` to inspect generated SQL and parameters without touching the database.

```typescript
const compiled = db
  .selectFrom('person')
  .select(['id', 'first_name'])
  .where('id', '=', personId)
  .compile()

console.log(compiled.sql)
console.log(compiled.parameters)
```

Use this first for:

- wrong SQL shape
- wrong bindings/order
- dialect-specific SQL differences

---

## Execute a Compiled Query

If you split build and execute phases, run compiled queries with `executeQuery`.

```typescript
const query = db
  .selectFrom('person')
  .selectAll()
  .where('id', '=', personId)
  .compile()

const result = await db.executeQuery(query)
```

This is useful when query construction and execution happen in different layers.

---

## Inspect Query Plans

Use `.explain()` for plan-level debugging and index verification.

```typescript
const plan = await db
  .selectFrom('person')
  .where('email', '=', email)
  .selectAll()
  .explain('json')

console.log(plan)
```

Prefer `explain` when SQL is correct but performance is poor.

---

## Stream Large Result Sets

Use `.stream()` for large datasets to avoid loading all rows into memory.

```typescript
for await (const row of db
  .selectFrom('event_log')
  .selectAll()
  .orderBy('id')
  .stream()) {
  // process row incrementally
}
```

Use batching or checkpoints if processing is long-running.

---

## Plugin Hook Points

Plugins are the right extension point when behavior should apply consistently across many queries.

Common uses:

- auto-tenant filters
- query tracing and correlation IDs
- result post-processing
- custom SQL transforms

Hook points:

- `transformQuery(args)` before compilation/execution
- `transformResult(args)` after driver returns rows

When deciding between a plugin and local query code:

- choose local query code for one-off query logic
- choose plugin when policy must apply globally and consistently

---

## Lifecycle Mental Model

Use this quick diagnostic ladder:

1. Builder stage: is the query tree what you intend?
2. Compile stage: is generated SQL/parameterization correct?
3. Driver stage: is the dialect/driver changing runtime types?
4. Result stage: is a plugin transforming rows?

When a bug appears, find the first stage where observed output diverges from expectation.

---

## Debugging Checklist

- Capture and inspect `.compile()` output.
- Run the SQL directly in the database to isolate driver/plugin effects.
- Compare behavior with plugins disabled.
- Validate runtime row shapes against TypeScript assumptions.
- Use `.explain()` for slow queries.
- Switch to `.stream()` for high-cardinality reads.

---

## Common Mistakes

- Trusting TypeScript types as runtime guarantees.
- Debugging performance only at app code level without `explain`.
- Loading huge tables with `.execute()` instead of `.stream()`.
- Using plugins for one-off query logic.
- Not checking compiled SQL when diagnosing dialect-specific issues.
