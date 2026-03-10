# Kysely Examples Reference

Complete code examples from the official Kysely documentation. Organized by operation type.

## Table of Contents

1. [SELECT](#select)
2. [WHERE](#where)
3. [JOIN](#join)
4. [INSERT](#insert)
5. [UPDATE](#update)
6. [DELETE](#delete)
7. [MERGE](#merge)
8. [Transactions](#transactions)
9. [CTE (Common Table Expressions)](#cte)

---

## SELECT

### Single Column

```typescript
const persons = await db
  .selectFrom('person')
  .select('id')
  .where('first_name', '=', 'Arnold')
  .execute()
```

### Column with Table Qualifier

When selecting from multiple tables, qualify column names with the table:

```typescript
const persons = await db
  .selectFrom(['person', 'pet'])
  .select('person.id')
  .execute()
```

### Multiple Columns

Pass an array to `select()`:

```typescript
const persons = await db
  .selectFrom('person')
  .select(['person.id', 'first_name'])
  .execute()
```

### Aliases

Append `as alias_name` to table names or column names:

```typescript
const persons = await db
  .selectFrom('person as p')
  .select([
    'first_name as fn',
    'p.last_name as ln'
  ])
  .execute()
```

### Complex Selections

For subqueries, raw SQL, or computed expressions, use a callback. Every non-column expression needs `.as()`:

```typescript
const persons = await db
  .selectFrom('person')
  .select((eb) => [
    'id',
    // Correlated subquery
    eb.selectFrom('pet')
      .select('pet.name')
      .whereRef('pet.owner_id', '=', 'person.id')
      .limit(1)
      .as('first_pet_name'),
    // OR expression
    eb.or([
      eb('first_name', '=', 'Jennifer'),
      eb('first_name', '=', 'Arnold')
    ]).as('is_jennifer_or_arnold'),
    // Raw SQL
    sql<string>`concat(${eb.ref('first_name')}, ' ', ${eb.ref('last_name')})`.as('full_name'),
    // Static values
    eb.val('hello').as('greeting'),
    eb.lit(42).as('magic_number'),
  ])
  .execute()
```

### Not Null Assertions

When you know a value isn't null but Kysely can't infer it:

```typescript
// Method 1: $notNull() on expressions
const result = await db
  .selectFrom('person')
  .select((eb) => [
    'id',
    jsonObjectFrom(
      eb.selectFrom('pet')
        .select(['pet.id as pet_id', 'pet.name'])
        .whereRef('pet.owner_id', '=', 'person.id')
        .where('pet.is_favorite', '=', true)
    ).$notNull().as('favorite_pet')
  ])
  .execute()

// Method 2: $narrowType for output type narrowing
const result = await db
  .selectFrom('person')
  .selectAll()
  .where('last_name', 'is not', null)
  .$narrowType<{ last_name: NotNull }>()
  .execute()
```

### Function Calls

```typescript
const result = await db
  .selectFrom('person')
  .innerJoin('pet', 'pet.owner_id', 'person.id')
  .select(({ fn, val, ref }) => [
    'person.id',

    // Built-in function shortcut
    fn.count<number>('pet.id').as('pet_count'),

    // Call any function with fn()
    fn<string>('concat', [ref('first_name'), val(' '), ref('last_name')]).as('full_name'),

    // Aggregate function
    fn.agg<string[]>('array_agg', ['pet.name']).as('pet_names'),

    // Raw SQL alternative
    sql<string>`concat(${ref('first_name')}, ' ', ${ref('last_name')})`.as('full_name_raw'),
  ])
  .groupBy('person.id')
  .execute()
```

### Distinct

```typescript
const persons = await db.selectFrom('person')
  .select('first_name')
  .distinct()
  .execute()
```

### Distinct On (PostgreSQL)

```typescript
const persons = await db.selectFrom('person')
  .innerJoin('pet', 'pet.owner_id', 'person.id')
  .where('pet.name', '=', 'Doggo')
  .distinctOn('person.id')
  .selectAll('person')
  .execute()
```

### All Columns

```typescript
// All columns from all tables in the query
const persons = await db.selectFrom('person').selectAll().execute()

// All columns from a specific table
const persons = await db.selectFrom('person').selectAll('person').execute()
```

### Nested Array (Relations)

Fetch related rows as a JSON array using dialect-specific helpers:

```typescript
import { jsonArrayFrom } from 'kysely/helpers/postgres' // or /mysql or /sqlite

const result = await db
  .selectFrom('person')
  .select((eb) => [
    'id',
    jsonArrayFrom(
      eb.selectFrom('pet')
        .select(['pet.id as pet_id', 'pet.name'])
        .whereRef('pet.owner_id', '=', 'person.id')
        .orderBy('pet.name')
    ).as('pets')
  ])
  .execute()
```

> **Important:** If your dialect returns JSON as strings (not parsed objects), add `ParseJSONResultsPlugin` to your Kysely instance.

### Nested Object (Relations)

Fetch a single related row as a JSON object:

```typescript
import { jsonObjectFrom } from 'kysely/helpers/postgres'

const result = await db
  .selectFrom('person')
  .select((eb) => [
    'id',
    jsonObjectFrom(
      eb.selectFrom('pet')
        .select(['pet.id as pet_id', 'pet.name'])
        .whereRef('pet.owner_id', '=', 'person.id')
        .where('pet.is_favorite', '=', true)
    ).as('favorite_pet')
  ])
  .execute()
```

### Generic / Dynamic Queries

When table or column names aren't known at compile time, use the `dynamic` module:

```typescript
async function getRowByColumn<
  T extends keyof Database,
  C extends keyof Database[T] & string,
  V extends SelectType<Database[T][C]>,
>(t: T, c: C, v: V) {
  const { table, ref } = db.dynamic
  return await db
    .selectFrom(table(t).as('t'))
    .selectAll()
    .where(ref(c), '=', v)
    .orderBy('t.id')
    .executeTakeFirstOrThrow()
}

const person = await getRowByColumn('person', 'first_name', 'Arnold')
```

---

## WHERE

### Simple Where (AND)

Multiple `.where()` calls are combined with AND:

```typescript
const person = await db
  .selectFrom('person')
  .selectAll()
  .where('first_name', '=', 'Jennifer')
  .where('age', '>', 40)
  .executeTakeFirst()
```

### Where In

```typescript
const persons = await db
  .selectFrom('person')
  .selectAll()
  .where('id', 'in', [1, 2, 3])
  .execute()
```

### Object Filter

Use `eb.and()` with an object for simple equality filters:

```typescript
const persons = await db
  .selectFrom('person')
  .selectAll()
  .where((eb) => eb.and({
    first_name: 'Jennifer',
    last_name: eb.ref('first_name')
  }))
  .execute()
```

### OR Where

Two approaches:

```typescript
// Approach 1: eb.or() with array
const persons = await db
  .selectFrom('person')
  .selectAll()
  .where((eb) => eb.or([
    eb('first_name', '=', 'Jennifer'),
    eb('first_name', '=', 'Sylvester')
  ]))
  .execute()

// Approach 2: chaining .or()
const persons = await db
  .selectFrom('person')
  .selectAll()
  .where((eb) =>
    eb('last_name', '=', 'Aniston').or('last_name', '=', 'Stallone')
  )
  .execute()
```

### Conditional Where

The query builder is immutable, so reassign when adding conditions:

```typescript
let query = db.selectFrom('person').selectAll()

if (firstName) {
  query = query.where('first_name', '=', firstName)
}

if (under18 || over60) {
  query = query.where((eb) => {
    const ors: Expression<SqlBool>[] = []
    if (under18) ors.push(eb('age', '<', 18))
    if (over60) ors.push(eb('age', '>', 60))
    return eb.or(ors)
  })
}

const persons = await query.execute()
```

### Complex Where

Use the callback form with destructured helpers for nested logic:

```typescript
const persons = await db
  .selectFrom('person')
  .selectAll('person')
  .where(({ eb, or, and, not, exists, selectFrom }) => and([
    or([
      eb('first_name', '=', firstName),
      eb('age', '<', maxAge)
    ]),
    not(exists(
      selectFrom('pet')
        .select('pet.id')
        .whereRef('pet.owner_id', '=', 'person.id')
    ))
  ]))
  .execute()
```

---

## JOIN

### Simple Inner Join

Select must come **after** the join so joined table columns are available:

```typescript
const result = await db
  .selectFrom('person')
  .innerJoin('pet', 'pet.owner_id', 'person.id')
  .select(['person.id', 'pet.name as pet_name'])
  .execute()
```

### Aliased Join

```typescript
await db.selectFrom('person')
  .innerJoin('pet as p', 'p.owner_id', 'person.id')
  .where('p.name', '=', 'Doggo')
  .selectAll()
  .execute()
```

### Complex Join

Pass a function as the second argument for complex ON conditions:

```typescript
await db.selectFrom('person')
  .innerJoin(
    'pet',
    (join) => join
      .onRef('pet.owner_id', '=', 'person.id')
      .on('pet.name', '=', 'Doggo')
      .on((eb) => eb.or([
        eb('person.age', '>', 18),
        eb('person.age', '<', 100)
      ]))
  )
  .selectAll()
  .execute()
```

### Subquery Join

Two callbacks — first for the subquery, second for join conditions:

```typescript
const result = await db.selectFrom('person')
  .innerJoin(
    (eb) => eb
      .selectFrom('pet')
      .select(['owner_id as owner', 'name'])
      .where('name', '=', 'Doggo')
      .as('doggos'),
    (join) => join
      .onRef('doggos.owner', '=', 'person.id'),
  )
  .selectAll('doggos')
  .execute()
```

> All join types are available: `innerJoin`, `leftJoin`, `rightJoin`, `fullJoin`.

---

## INSERT

### Single Row

```typescript
const result = await db
  .insertInto('person')
  .values({
    first_name: 'Jennifer',
    last_name: 'Aniston',
    age: 40,
  })
  .executeTakeFirst()

console.log(result.insertId) // Only available on MySQL/SQLite
```

### Multiple Rows

```typescript
await db
  .insertInto('person')
  .values([
    { first_name: 'Jennifer', last_name: 'Aniston', age: 40 },
    { first_name: 'Arnold', last_name: 'Schwarzenegger', age: 70 },
  ])
  .execute()
```

> PostgreSQL supports array inserts natively. Other dialects may vary.

### Returning Data (PostgreSQL)

`returning` works just like `select`:

```typescript
const result = await db
  .insertInto('person')
  .values({
    first_name: 'Jennifer',
    last_name: 'Aniston',
    age: 40,
  })
  .returning(['id', 'first_name as name'])
  .executeTakeFirstOrThrow()
```

### Complex Values

Use a callback in `.values()` for expressions, refs, and subqueries:

```typescript
const result = await db
  .insertInto('person')
  .values(({ ref, selectFrom, fn }) => ({
    first_name: 'Jennifer',
    last_name: sql<string>`concat(${ani}, ${ston})`,
    middle_name: ref('first_name'),
    age: selectFrom('person')
      .select(fn.avg<number>('age').as('avg_age')),
  }))
  .executeTakeFirst()
```

### Insert from Subquery (INSERT INTO ... SELECT FROM)

```typescript
const result = await db.insertInto('person')
  .columns(['first_name', 'last_name', 'age'])
  .expression((eb) => eb
    .selectFrom('pet')
    .select((eb) => [
      'pet.name',
      eb.val('Petson').as('last_name'),
      eb.lit(7).as('age'),
    ])
  )
  .execute()
```

---

## UPDATE

### Single Row

```typescript
const result = await db
  .updateTable('person')
  .set({
    first_name: 'Jennifer',
    last_name: 'Aniston',
  })
  .where('id', '=', 1)
  .executeTakeFirst()
```

### Complex Values

Use a callback in `.set()` for arithmetic, subqueries, etc.:

```typescript
await db
  .updateTable('person')
  .set((eb) => ({
    age: eb('age', '+', 1),
    first_name: eb.selectFrom('pet').select('name').limit(1),
    last_name: 'updated',
  }))
  .where('id', '=', 1)
  .executeTakeFirst()
```

### MySQL Multi-Table Update

MySQL allows joining tables in updates:

```typescript
const result = await db
  .updateTable(['person', 'pet'])
  .set('person.first_name', 'Updated person')
  .set('pet.name', 'Updated doggo')
  .whereRef('person.id', '=', 'pet.owner_id')
  .where('person.id', '=', 1)
  .executeTakeFirst()
```

> The `innerJoin` etc. methods on `UpdateQueryBuilder` are for PostgreSQL's `UPDATE SET FROM JOIN` pattern only.

---

## DELETE

### Single Row

```typescript
const result = await db
  .deleteFrom('person')
  .where('person.id', '=', 1)
  .executeTakeFirst()

console.log(result.numDeletedRows)
```

---

## MERGE

### Based on Source Row Existence

```typescript
const result = await db
  .mergeInto('person')
  .using('pet', 'pet.owner_id', 'person.id')
  .whenMatchedAnd('person.has_pets', '!=', 'Y')
  .thenUpdateSet({ has_pets: 'Y' })
  .whenNotMatchedBySourceAnd('person.has_pets', '=', 'Y')
  .thenUpdateSet({ has_pets: 'N' })
  .executeTakeFirstOrThrow()

console.log(result.numChangedRows)
```

### Temporary Changes Table (Upsert Pattern)

```typescript
const result = await db
  .mergeInto('wine as target')
  .using('wine_stock_change as source', 'source.wine_name', 'target.name')
  .whenNotMatchedAnd('source.stock_delta', '>', 0)
  .thenInsertValues(({ ref }) => ({
    name: ref('source.wine_name'),
    stock: ref('source.stock_delta'),
  }))
  .whenMatchedAnd(
    (eb) => eb('target.stock', '+', eb.ref('source.stock_delta')),
    '>',
    0,
  )
  .thenUpdateSet('stock', (eb) =>
    eb('target.stock', '+', eb.ref('source.stock_delta')),
  )
  .whenMatched()
  .thenDelete()
  .executeTakeFirstOrThrow()
```

---

## Transactions

### Simple Transaction

Auto-commits on success, auto-rolls back on exception:

```typescript
const result = await db.transaction().execute(async (trx) => {
  const person = await trx
    .insertInto('person')
    .values({ first_name: 'Jennifer', last_name: 'Aniston', age: 40 })
    .returning('id')
    .executeTakeFirstOrThrow()

  return await trx
    .insertInto('pet')
    .values({ owner_id: person.id, name: 'Catto', species: 'cat' })
    .returningAll()
    .executeTakeFirstOrThrow()
})
```

### Controlled Transaction

Manual commit/rollback:

```typescript
await db.startTransaction().execute(async (trx) => {
  try {
    await trx.insertInto('person').values({...}).execute()
    await trx.insertInto('pet').values({...}).execute()
    await trx.commit().execute()
  } catch (error) {
    await trx.rollback().execute()
  }
})
```

### Controlled Transaction with Savepoints

```typescript
await db.startTransaction().execute(async (trx) => {
  try {
    await trx.insertInto('person').values({...}).execute()

    await trx.savepoint('after_person').execute()
    try {
      await trx.insertInto('pet').values({...}).execute()
      await trx.insertInto('toy').values({...}).execute()
    } catch {
      await trx.rollbackToSavepoint('after_person').execute()
    }
    await trx.releaseSavepoint('after_person').execute()

    await trx.insertInto('audit').values({...}).execute()
    await trx.commit().execute()
  } catch {
    await trx.rollback().execute()
  }
})
```

---

## CTE

### Simple CTE Selects

Chain `.with()` to define CTEs, then query them:

```typescript
const result = await db
  .with('jennifers', (qb) =>
    qb.selectFrom('person')
      .select(['id', 'age'])
      .where('first_name', '=', 'Jennifer')
  )
  .with('adult_jennifers', (qb) =>
    qb.selectFrom('jennifers')
      .select(['id', 'age'])
      .where('age', '>', 18)
  )
  .selectFrom('adult_jennifers')
  .selectAll()
  .where('age', '<', 60)
  .execute()
```

### CTEs with INSERT/UPDATE/DELETE (PostgreSQL)

PostgreSQL allows modification queries inside CTEs:

```typescript
const result = await db
  .with('new_person', (qb) =>
    qb.insertInto('person')
      .values({ first_name: 'Jennifer', last_name: 'Aniston', age: 40 })
      .returning(['id'])
  )
  .with('new_pet', (qb) =>
    qb.insertInto('pet')
      .values((eb) => ({
        owner_id: eb.selectFrom('new_person').select('id'),
        name: 'Catto',
        species: 'cat',
      }))
      .returning(['id'])
  )
  .selectFrom(['new_person', 'new_pet'])
  .select(['new_person.id as person_id', 'new_pet.id as pet_id'])
  .executeTakeFirstOrThrow()
```
