---
name: node-pg-migrate
description: >
  PostgreSQL migration expert using node-pg-migrate. Use this skill whenever the user is writing
  database migrations with node-pg-migrate, creating or altering tables, columns, indexes,
  constraints, functions, triggers, views, types, sequences, schemas, roles, policies, extensions,
  or any other PostgreSQL DDL operation through migrations. Also trigger when code imports
  'node-pg-migrate', uses pgm.createTable/addColumns/createIndex patterns, references
  MigrationBuilder type, or mentions pg-migrate CLI commands (up, down, create, redo). Activate
  for questions about migration file structure, async migrations, automatic down migrations,
  transaction control, row-level security policies, grants, operator classes, domains, casts,
  or the pgm.sql/pgm.func/pgm.db utilities. Also trigger for programmatic API usage (runner
  function), troubleshooting SSL/connection issues, or TypeScript/ESM setup. Covers node-pg-migrate
  v9.x (9.0.0-alpha.6).
---

# node-pg-migrate (v9.0.0-alpha.6)

You are an expert at writing PostgreSQL database migrations with node-pg-migrate. Generate correct, production-ready migration code using the `pgm` MigrationBuilder API. This skill references the **node-pg-migrate v9.x** documentation.

## Core Principles

1. **Migrations are declarative** — Calling `pgm.createTable()`, `pgm.addColumns()`, etc. queues SQL commands; they don't execute immediately. The framework runs them in order after your function returns.
2. **Reversibility matters** — Write both `up` and `down` functions. Omit `down` only when all operations are auto-reversible. Set `exports.down = false` for intentionally irreversible migrations.
3. **Transactions by default** — Migrations run inside a transaction. Call `pgm.noTransaction()` for operations that can't run in transactions (e.g., `CREATE INDEX CONCURRENTLY`, `addTypeValue` on existing enums).
4. **Schema-qualified names** — Pass `{ schema: 'myschema', name: 'mytable' }` objects instead of strings when working with non-public schemas.
5. **Type shortcuts** — Use `'id'` for `{ type: 'serial', primaryKey: true }`, and string aliases like `'int'`, `'string'`, `'float'`, `'double'`, `'datetime'`, `'bool'` for common types.
6. **Shorthand inheritance** — Shorthands defined in earlier migrations are inherited by all subsequent migrations. Override a shorthand by redefining it in a later migration.
7. **TypeScript out of the box** — TypeScript and modern JavaScript are supported via the jiti loader. Use `-j ts` to generate `.ts` migration files. No manual compilation needed.

## How to Use This Skill

Before generating migration code, load the relevant reference file(s):

- `references/tables-columns.md` — createTable, dropTable, renameTable, alterTable, column definitions, addColumns, dropColumns, renameColumn, alterColumn, type shortcuts, shorthands
- `references/constraints-indexes.md` — addConstraint, dropConstraint, renameConstraint, createIndex, dropIndex, foreign keys, unique, check, exclude constraints
- `references/functions-triggers.md` — createFunction, dropFunction, renameFunction, createTrigger, dropTrigger, renameTrigger, function params, trigger options
- `references/schemas-sequences.md` — createSchema, dropSchema, renameSchema, createSequence, dropSequence, alterSequence, renameSequence
- `references/views.md` — createView, dropView, alterView, renameView, createMaterializedView, dropMaterializedView, alterMaterializedView, refreshMaterializedView
- `references/types-domains.md` — createType (enum and composite), dropType, renameType, addTypeValue, addTypeAttribute, createDomain, dropDomain, alterDomain
- `references/roles-policies-grants.md` — createRole, dropRole, alterRole, createPolicy, dropPolicy, alterPolicy, grantRoles, revokeRoles, grantOnTables, grantOnSchemas
- `references/operators.md` — createOperator, dropOperator, createOperatorClass, createOperatorFamily, addToOperatorFamily
- `references/extensions-misc.md` — createExtension, dropExtension, createCast, dropCast, pgm.sql(), pgm.func(), pgm.db, pgm.noTransaction()
- `references/cli.md` — CLI commands (create, up, down, redo), flags, configuration, environment variables
- `references/programmatic-api.md` — Runner function for using node-pg-migrate programmatically, all runner options (databaseUrl, dbClient, dryRun, etc.)
- `references/troubleshooting.md` — SSL connections, case sensitivity, connection issues, transaction gotchas, environment variables

## Quick Reference

### Migration File Structure

```js
// migrations/1234567890_create-users.js

/** @type {import('node-pg-migrate').ColumnDefinitions} */
exports.shorthands = undefined;

/** @param {import('node-pg-migrate').MigrationBuilder} pgm */
exports.up = (pgm) => {
  pgm.createTable('users', {
    id: 'id',                               // serial primary key
    email: { type: 'varchar(255)', notNull: true, unique: true },
    name: { type: 'text', notNull: true },
    role: { type: 'varchar(50)', default: "'user'" },
    created_at: { type: 'timestamp', notNull: true, default: pgm.func('current_timestamp') },
  });
};

/** @param {import('node-pg-migrate').MigrationBuilder} pgm */
exports.down = (pgm) => {
  pgm.dropTable('users');
};
```

### TypeScript Migration

```ts
import type { MigrationBuilder, ColumnDefinitions } from 'node-pg-migrate';

export const shorthands: ColumnDefinitions | undefined = undefined;

export async function up(pgm: MigrationBuilder): Promise<void> {
  pgm.createTable('posts', {
    id: 'id',
    user_id: {
      type: 'integer',
      notNull: true,
      references: 'users',
      onDelete: 'CASCADE',
    },
    title: { type: 'text', notNull: true },
    body: { type: 'text' },
    published_at: { type: 'timestamp' },
    created_at: { type: 'timestamp', notNull: true, default: pgm.func('current_timestamp') },
  });

  pgm.createIndex('posts', 'user_id');
}

export async function down(pgm: MigrationBuilder): Promise<void> {
  pgm.dropTable('posts');
}
```

### Async Migration (Data Migration)

```js
exports.up = async (pgm) => {
  pgm.addColumns('users', { display_name: { type: 'text' } });

  // Run queued DDL first, then do data migration
  await pgm.db.query('UPDATE users SET display_name = name WHERE display_name IS NULL');
};
```

### Enum Type

```js
exports.up = (pgm) => {
  pgm.createType('status_enum', ['draft', 'published', 'archived']);
  pgm.addColumns('posts', {
    status: { type: 'status_enum', notNull: true, default: "'draft'" },
  });
};
```

### Adding Enum Values (No Transaction)

```js
exports.up = (pgm) => {
  pgm.noTransaction();
  pgm.addTypeValue('status_enum', 'pending', { before: 'published' });
};

exports.down = false; // enum values cannot be removed
```

### Concurrent Index

```js
exports.up = (pgm) => {
  pgm.noTransaction();
  pgm.createIndex('posts', 'title', { concurrently: true });
};
```

### Row-Level Security

```js
exports.up = (pgm) => {
  pgm.alterTable('documents', { levelSecurity: 'ENABLE' });
  pgm.createPolicy('documents', 'owner_access', {
    command: 'ALL',
    using: 'user_id = current_user_id()',
    check: 'user_id = current_user_id()',
  });
};
```

### Programmatic API

```js
import { default as migrate } from 'node-pg-migrate';

await migrate({
  databaseUrl: process.env.DATABASE_URL,
  direction: 'up',
  dir: 'migrations',
  migrationsTable: 'pgmigrations',
  log: console.log,
});
```

### Schema-Qualified Names

```js
pgm.createTable(
  { schema: 'my_schema', name: 'my_table' },
  { id: 'id', name: { type: 'text' } }
);
```

## Common Patterns

### Foreign Key with Constraint Name

```js
pgm.addConstraint('posts', 'fk_posts_user', {
  foreignKeys: {
    columns: 'user_id',
    references: 'users(id)',
    onDelete: 'CASCADE',
  },
});
```

### Composite Unique Index

```js
pgm.createIndex('user_roles', ['user_id', 'role_id'], { unique: true });
```

### Custom Shorthands

```js
exports.shorthands = {
  created_at: { type: 'timestamp', notNull: true, default: new PgLiteral('current_timestamp') },
  updated_at: { type: 'timestamp', notNull: true, default: new PgLiteral('current_timestamp') },
};
```

### Raw SQL

```js
pgm.sql(`
  CREATE OR REPLACE FUNCTION update_updated_at()
  RETURNS TRIGGER AS $$
  BEGIN
    NEW.updated_at = current_timestamp;
    RETURN NEW;
  END;
  $$ LANGUAGE plpgsql;
`);
```

## Key Warnings

- **String defaults need inner quotes** — `default: "'active'"` not `default: 'active'`. Use `pgm.func('current_timestamp')` for SQL expressions.
- **`addTypeValue` requires `noTransaction()`** — Adding values to an existing enum type created in a previous migration fails inside a transaction.
- **Concurrent operations require `noTransaction()`** — `CREATE INDEX CONCURRENTLY` cannot run inside a transaction.
- **Advisory locks** — node-pg-migrate uses advisory locks to prevent concurrent migrations. If a migration crashes, you may need to manually release the lock.
- **`pgm.db.query()` runs immediately** — Unlike other `pgm` methods that queue SQL, `pgm.db.query()` executes right away using the migration's connection.
- **Down migrations for `addTypeValue`** — Enum values cannot be removed in PostgreSQL. Set `exports.down = false` for these migrations.
