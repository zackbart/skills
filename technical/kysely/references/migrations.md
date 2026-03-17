# Kysely Migrations Reference

Migration patterns from the official Kysely migrations docs and API docs.

## Table of Contents

1. [Core Rule](#core-rule)
2. [Migration Provider Setup](#migration-provider-setup)
3. [Write Migration Files](#write-migration-files)
4. [Run Migrations Safely](#run-migrations-safely)
5. [Directional and Targeted Migration](#directional-and-targeted-migration)
6. [Rollback All Migrations](#rollback-all-migrations)
7. [Unordered Migration Support](#unordered-migration-support)
8. [Operational Tips](#operational-tips)
9. [Common Mistakes](#common-mistakes)

---

## Core Rule

Migration callbacks should use `Kysely<any>`, not your app database interface.

Kysely runs migration files against historical schema states. If you use your current app `Database` type in old migrations, type checks can break as schema evolves.

```typescript
import { Kysely } from 'kysely'

export async function up(db: Kysely<any>): Promise<void> {
  // schema changes here
}

export async function down(db: Kysely<any>): Promise<void> {
  // reverse changes here
}
```

---

## Migration Provider Setup

Use `FileMigrationProvider` for migration files on disk, and construct a `Migrator`.

```typescript
import { promises as fs } from 'node:fs'
import path from 'node:path'
import { FileMigrationProvider, Migrator } from 'kysely'

const migrator = new Migrator({
  db,
  provider: new FileMigrationProvider({
    fs,
    path,
    migrationFolder: path.join(process.cwd(), 'migrations'),
  }),
})
```

Run with:

```typescript
const { error, results } = await migrator.migrateToLatest()
```

---

## Write Migration Files

Create one file per migration with `up` and optional `down`.

```typescript
import { Kysely, sql } from 'kysely'

export async function up(db: Kysely<any>): Promise<void> {
  await db.schema
    .createTable('person')
    .addColumn('id', 'serial', (col) => col.primaryKey())
    .addColumn('first_name', 'varchar(255)', (col) => col.notNull())
    .execute()

  await db.schema
    .createIndex('person_first_name_idx')
    .on('person')
    .column('first_name')
    .execute()

  await sql`comment on table person is 'people table'`.execute(db)
}

export async function down(db: Kysely<any>): Promise<void> {
  await db.schema.dropTable('person').execute()
}
```

---

## Run Migrations Safely

Migrator methods return a result object. They do not always throw directly. Check `error` and inspect per-migration results.

```typescript
const { error, results } = await migrator.migrateToLatest()

results?.forEach((it) => {
  if (it.status === 'Success') {
    console.log(`migration "${it.migrationName}" executed successfully`)
  } else if (it.status === 'Error') {
    console.error(`failed to execute migration "${it.migrationName}"`)
  }
})

if (error) {
  console.error('failed to migrate')
  console.error(error)
  process.exit(1)
}
```

Use this same `error/results` handling pattern for all migrator methods.

---

## Directional and Targeted Migration

Use directional methods for controlled releases and rollback tooling.

### Migrate all pending

```typescript
await migrator.migrateToLatest()
```

### Apply one migration upward

```typescript
await migrator.migrateUp()
```

### Roll back one migration

```typescript
await migrator.migrateDown()
```

### Migrate to a specific migration

```typescript
await migrator.migrateTo('2026_03_10_add_invoice_table')
```

---

## Rollback All Migrations

Use `NO_MIGRATIONS` to target the empty state.

```typescript
import { NO_MIGRATIONS } from 'kysely'

await migrator.migrateTo(NO_MIGRATIONS)
```

---

## Unordered Migration Support

By default, migrations are expected to run in strict name order. If your workflow allows branch/merge reordering, enable unordered execution in migrator config.

```typescript
const migrator = new Migrator({
  db,
  provider,
  allowUnorderedMigrations: true,
})
```

Use this intentionally; strict ordering is safer when multiple teams touch schema.

---

## Operational Tips

- Use a dedicated migration command in CI/CD before app rollout.
- Run migrations with a single leader process to avoid concurrency races.
- Keep migrations small and reversible.
- Avoid mixing data backfills and structural changes unless required.
- Log `migrationName` and `status` for auditability.

---

## Common Mistakes

- Typing migration callbacks with your app `Database` instead of `Kysely<any>`.
- Ignoring the returned `error` field from migrator methods.
- Assuming every dialect supports identical DDL behavior.
- Adding out-of-order files without deciding whether to enable `allowUnorderedMigrations`.
- Shipping non-reversible migrations without documenting rollback plans.
