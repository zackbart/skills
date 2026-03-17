# Kysely Recipes Reference

Advanced patterns and best practices from the official Kysely documentation.

## Table of Contents

1. [Relations (Nested Data)](#relations)
2. [Reusable Helpers](#reusable-helpers)
3. [Data Types](#data-types)
4. [Raw SQL](#raw-sql)
5. [Splitting Query Building and Execution](#splitting-query-building-and-execution)
6. [Conditional Selects](#conditional-selects)
7. [Expressions](#expressions)
8. [Schemas](#schemas)
9. [Deduplicate Joins](#deduplicate-joins)
10. [Excessively Deep Types](#excessively-deep-types)
11. [Extending Kysely](#extending-kysely)
12. [Introspecting Metadata](#introspecting-metadata)
13. [Logging](#logging)

---

## Relations

Kysely is NOT an ORM — it has no concept of relations. But you can nest related data using database-native JSON capabilities with dialect-specific helpers.

### Import Paths

```typescript
// PostgreSQL
import { jsonArrayFrom, jsonObjectFrom } from 'kysely/helpers/postgres'

// MySQL (requires 8.0.14+)
import { jsonArrayFrom, jsonObjectFrom } from 'kysely/helpers/mysql'

// SQLite
import { jsonArrayFrom, jsonObjectFrom } from 'kysely/helpers/sqlite'
```

### One-to-Many (Nested Array)

```typescript
const persons = await db
  .selectFrom('person')
  .select((eb) => [
    'id',
    'first_name',
    jsonArrayFrom(
      eb.selectFrom('pet')
        .select(['pet.id', 'pet.name'])
        .whereRef('pet.owner_id', '=', 'person.id')
        .orderBy('pet.name')
    ).as('pets')
  ])
  .execute()
```

### One-to-One (Nested Object)

```typescript
const persons = await db
  .selectFrom('person')
  .select((eb) => [
    'id',
    jsonObjectFrom(
      eb.selectFrom('pet')
        .select(['pet.id', 'pet.name'])
        .whereRef('pet.owner_id', '=', 'person.id')
        .where('pet.is_favorite', '=', true)
    ).as('favorite_pet')
  ])
  .execute()
```

### Non-Null Relations

Use `$notNull()` when you know the relation always exists:

```typescript
jsonObjectFrom(
  eb.selectFrom('pet')
    .select(['pet.id', 'pet.name'])
    .whereRef('pet.owner_id', '=', 'person.id')
).$notNull().as('pet')
```

### Reusable Relation Helpers

Extract common relations into functions:

```typescript
function withPets(eb: ExpressionBuilder<Database, 'person'>) {
  return jsonArrayFrom(
    eb.selectFrom('pet')
      .select(['pet.id', 'pet.name'])
      .whereRef('pet.owner_id', '=', 'person.id')
      .orderBy('pet.name')
  ).as('pets')
}

// Usage
const result = await db
  .selectFrom('person')
  .select((eb) => ['id', 'first_name', withPets(eb)])
  .execute()
```

### JSON Parsing Plugin

If your driver returns JSON as strings (not objects), add the plugin:

```typescript
const db = new Kysely<Database>({
  dialect,
  plugins: [new ParseJSONResultsPlugin()],
})
```

---

## Reusable Helpers

The key principle: everything in Kysely is an `Expression<T>`. Build helpers that accept and return expressions for maximum composability.

### Simple Function Wrappers

```typescript
function upper(expr: Expression<string>): RawBuilder<string> {
  return sql<string>`upper(${expr})`
}

function lower(expr: Expression<string>): RawBuilder<string> {
  return sql<string>`lower(${expr})`
}

// Usage in queries
db.selectFrom('person')
  .select((eb) => [upper(eb.ref('first_name')).as('upper_name')])
```

### Subquery Helpers with ExpressionBuilder

```typescript
function petCountFor(ownerId: Expression<number>) {
  const eb = expressionBuilder<Database, never>()
  return eb
    .selectFrom('pet')
    .select((eb) => eb.fn.countAll<number>().as('count'))
    .where('owner_id', '=', ownerId)
}

// Usage
db.selectFrom('person')
  .select((eb) => [
    'first_name',
    petCountFor(eb.ref('person.id')).as('pet_count')
  ])
```

### Binary Expression Helpers

```typescript
function isAdult(age: Expression<number>): Expression<SqlBool> {
  const eb = expressionBuilder<Database, never>()
  return eb(age, '>=', 18)
}
```

### Preserving Nullability

Use generic constraints to pass through nullability:

```typescript
function lower<T extends string | null>(expr: Expression<T>): RawBuilder<T> {
  return sql<T>`lower(${expr})`
}
```

### Converting to Scalar

Use `$asScalar()` when a helper expects a scalar but you have a single-column query:

```typescript
const query = eb.selectFrom('pet').select('name').limit(1).$asScalar()
upper(query).as('upper_pet_name')
```

---

## Data Types

### TypeScript vs Runtime Types

TypeScript types in your Database interface are compile-time only. The actual runtime types come from your database driver. If you declare a column as `string` but the driver returns a `number`, you get a `number` at runtime.

### PostgreSQL Type Parsing

Use `pg-types` to configure how `pg` parses database types:

```typescript
import * as pg from 'pg'

const int8TypeId = 20
pg.types.setTypeParser(int8TypeId, (val) => parseInt(val, 10))
```

### MySQL Type Casting

Use `typeCast` in the mysql2 pool config:

```typescript
const db = new Kysely<Database>({
  dialect: new MysqlDialect({
    pool: createPool({
      typeCast(field, next) {
        if (field.type === 'TINY' && field.length === 1) {
          return field.string() === '1'
        }
        return next()
      }
    })
  })
})
```

### Type Generation Tools

- **kysely-codegen** — Generates TypeScript types from your database schema
- **kanel-kysely** — Alternative type generator

---

## Raw SQL

Use the `sql` template tag for raw SQL. Interpolated values are parameterized by default (safe from SQL injection).

### Basic Usage

```typescript
const result = await sql<Person>`select * from person where id = ${id}`.execute(db)
```

### Column References

Use `sql.ref()` to reference columns (not parameterized):

```typescript
sql`select * from person order by ${sql.ref('first_name')}`
```

### Table References

```typescript
sql`select * from ${sql.table('person')}`
```

### Unparameterized SQL (Use Carefully)

```typescript
sql`select * from ${sql.raw(tableName)}`
```

> Only use `sql.raw()` with trusted values — it's not parameterized and is vulnerable to SQL injection.

### Inside Query Builder

```typescript
db.selectFrom('person')
  .select(sql<string>`concat(first_name, ' ', last_name)`.as('full_name'))
  .where(sql<boolean>`age > ${minAge}`)
```

---

## Splitting Query Building and Execution

### Cold Kysely Instance (No Database Connection)

Use `DummyDriver` when you only need to build SQL, not execute it:

```typescript
import { Kysely, DummyDriver, PostgresAdapter, PostgresIntrospector, PostgresQueryCompiler } from 'kysely'

const db = new Kysely<Database>({
  dialect: {
    createAdapter: () => new PostgresAdapter(),
    createDriver: () => new DummyDriver(),
    createIntrospector: (db) => new PostgresIntrospector(db),
    createQueryCompiler: () => new PostgresQueryCompiler(),
  },
})
```

### Compiling Queries

```typescript
const compiledQuery = db
  .selectFrom('person')
  .select('first_name')
  .where('id', '=', id)
  .compile()

// compiledQuery.sql: "select first_name from person where id = $1"
// compiledQuery.parameters: [id]
```

### Type Inference from Compiled Queries

```typescript
import { InferResult } from 'kysely'

const query = db.selectFrom('person').select('first_name')
type Result = InferResult<typeof query> // { first_name: string }[]
```

### Executing Compiled Queries

On a "hot" (connected) Kysely instance:

```typescript
const results = await db.executeQuery(compiledQuery)
```

---

## Conditional Selects

### The Problem

TypeScript downcasts the query type on reassignment, losing type info about conditionally added selections.

### Solution 1: Separate Return Paths

```typescript
if (includeLastName) {
  return db.selectFrom('person').select(['first_name', 'last_name']).execute()
} else {
  return db.selectFrom('person').select('first_name').execute()
}
```

### Solution 2: $if Method (Recommended)

```typescript
const result = await db
  .selectFrom('person')
  .select('first_name')
  .$if(includeLastName, (qb) => qb.select('last_name'))
  .execute()
```

Conditionally selected fields become **optional** in the output type.

> `$if` also works for `where`, `groupBy`, `orderBy`, and other methods that don't change the query structure.

---

## Expressions

### The Expression Builder

`ExpressionBuilder<DB, TB>` is accessed via callbacks:

```typescript
db.selectFrom('person')
  .select((eb) => [
    eb.fn.count('id').as('count'),        // function call
    eb('age', '>', 18).as('is_adult'),    // binary expression
    eb.val('hello').as('greeting'),       // parameterized value
    eb.lit(42).as('number'),              // literal value
    eb.selectFrom('pet')                   // subquery
      .select(eb.fn.countAll().as('c'))
      .whereRef('pet.owner_id', '=', 'person.id')
      .as('pet_count'),
  ])
```

### Building Conditional Expressions

```typescript
db.selectFrom('person')
  .where((eb) => {
    const conditions: Expression<SqlBool>[] = []

    conditions.push(eb('active', '=', true))

    if (minAge) conditions.push(eb('age', '>=', minAge))
    if (maxAge) conditions.push(eb('age', '<=', maxAge))
    if (names.length) conditions.push(eb('first_name', 'in', names))

    return eb.and(conditions)
  })
```

### Global Expression Builder

For reusable helpers outside query callbacks:

```typescript
import { expressionBuilder } from 'kysely'

const eb = expressionBuilder<Database, 'person'>()
```

---

## Schemas

### Enumerable Schemas (Static)

Add schema-qualified table names to your Database interface:

```typescript
interface Database {
  'user_schema.user': UserTable
  'user_schema.permission': PermissionTable
  pet: PetTable  // default/public schema
}

db.selectFrom('user_schema.user').selectAll().execute()
```

### Dynamic Schemas (Multi-Tenant)

Use `withSchema()` to set a default schema:

```typescript
const tenantDb = db.withSchema(tenantSchemaName)

await tenantDb
  .selectFrom('user')   // resolves to tenantSchemaName.user
  .selectAll()
  .execute()
```

To reference tables outside the default schema (e.g., shared tables), declare them with explicit schema prefixes in the Database interface.

---

## Deduplicate Joins

When building dynamic queries, the same join might be added multiple times. Use `DeduplicateJoinsPlugin`:

```typescript
// Global
const db = new Kysely<Database>({
  dialect,
  plugins: [new DeduplicateJoinsPlugin()],
})

// Per-query
db.withPlugin(new DeduplicateJoinsPlugin())
  .selectFrom('person')
  .innerJoin('pet', 'pet.owner_id', 'person.id')
  .innerJoin('pet', 'pet.owner_id', 'person.id') // duplicate removed
  .selectAll()
  .execute()
```

> Not enabled by default because detecting identical joins is surprisingly difficult with complex subqueries.

---

## Excessively Deep Types

If TypeScript throws `TS2589: Type instantiation is excessively deep`, use `$assertType`:

```typescript
const result = await db
  .with('cte1', (qb) => qb.selectFrom('t1').select('col1'))
  .with('cte2', (qb) => qb.selectFrom('t2').select('col2'))
  // ... many more CTEs ...
  .with('cte12', (qb) =>
    qb.selectFrom('t12').select('col12')
      .$assertType<{ col12: string }>()  // helps TypeScript
  )
  .selectFrom('cte12')
  .selectAll()
  .execute()
```

The asserted type must structurally match the actual type — type safety is maintained.

---

## Extending Kysely

### Custom Expressions via sql Tag

```typescript
function json<T>(value: T): RawBuilder<T> {
  return sql`CAST(${JSON.stringify(value)} AS JSONB)`
}

// Usage
db.insertInto('person')
  .values({ metadata: json({ key: 'value' }) })
```

### Custom AliasedExpression (for SELECT/FROM)

The `RawBuilder.as()` method creates an `AliasedRawBuilder`:

```typescript
sql<number>`count(*)`.as('total')  // usable in .select()
```

### Module Augmentation

Add methods to Kysely's builders:

```typescript
declare module 'kysely' {
  interface CreateTableBuilder<TB extends string, C extends string> {
    addIdColumn(): CreateTableBuilder<TB, C | 'id'>
  }
}

CreateTableBuilder.prototype.addIdColumn = function () {
  return this.addColumn('id', 'uuid', (col) =>
    col.primaryKey().defaultTo(sql`gen_random_uuid()`)
  )
}
```

---

## Introspecting Metadata

Extract table/view metadata at runtime:

```typescript
const tables = await db.introspection.getTables()
console.log(tables)
// Returns TableMetadata[] with column info, types, etc.
```

---

## Logging

### Simple Logging

```typescript
const db = new Kysely<Database>({
  dialect,
  log: ['query', 'error'],  // logs all queries and errors
})
```

### Custom Logging

```typescript
const db = new Kysely<Database>({
  dialect,
  log(event) {
    if (event.level === 'error') {
      console.error('Query failed:', {
        durationMs: event.queryDurationMillis,
        error: event.error,
        sql: event.query.sql,
        params: event.query.parameters,
      })
    } else {
      console.log('Query executed:', {
        durationMs: event.queryDurationMillis,
        sql: event.query.sql,
      })
    }
  },
})
```

The `LogEvent` contains:
- `level`: `'query'` or `'error'`
- `query`: `CompiledQuery` with `.sql`, `.parameters`, and `.query` (syntax tree)
- `queryDurationMillis`: execution time
- `error`: the error object (only when level is `'error'`)
