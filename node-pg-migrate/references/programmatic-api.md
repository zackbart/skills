# Programmatic API

node-pg-migrate exports a runner function for use in application code, scripts, and test setups.

## Basic Usage

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

## With Existing Client

```js
import { Client } from 'pg';
import { default as migrate } from 'node-pg-migrate';

const client = new Client({ connectionString: process.env.DATABASE_URL });
await client.connect();

try {
  await migrate({
    dbClient: client,
    direction: 'up',
    dir: 'migrations',
    migrationsTable: 'pgmigrations',
  });
} finally {
  await client.end(); // caller must close the connection
}
```

## Runner Options

### Connection

| Option | Type | Description |
|--------|------|-------------|
| `databaseUrl` | string or object | Connection string or pg.Client config object |
| `dbClient` | pg.Client | Pre-connected client (caller manages lifecycle) |

Do not use both `dbClient` and `databaseUrl` — they are mutually exclusive.

### Migration Location

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `dir` | string or string[] | `'migrations'` | Migration files directory or glob patterns (resolved from cwd) |
| `useGlob` | boolean | false | Enable glob pattern matching |
| `ignorePattern` | string or string[] | — | Regex or glob patterns to exclude files |
| `file` | string | — | Run a single specific migration file |

### Schema & Tracking

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `schema` | string or string[] | `'public'` | Target schema(s), sets search_path |
| `createSchema` | boolean | false | Auto-create schema if missing |
| `migrationsSchema` | string | (matches schema) | Schema for tracking table |
| `createMigrationsSchema` | boolean | false | Auto-create tracking schema |
| `migrationsTable` | string | `'pgmigrations'` | Migration tracking table name |

### Execution

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `direction` | string | — | `'up'` or `'down'` (required) |
| `count` | number | — | Number of migrations to run |
| `timestamp` | boolean | false | Interpret `count` as a timestamp |
| `checkOrder` | boolean | true | Validate migration execution order |
| `singleTransaction` | boolean | true | Wrap all pending migrations in one transaction |
| `noLock` | boolean | false | Disable advisory lock |
| `lockValue` | number | — | Custom advisory lock identifier |
| `fake` | boolean | false | Mark as run without executing |
| `dryRun` | boolean | false | Preview changes without applying |

### Output

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `log` | function | — | Custom logging function (replaces console) |
| `logger` | object | — | Logger with debug/info/warn/error methods |
| `verbose` | boolean | false | Print SQL and debug info |
| `decamelize` | boolean | false | Convert camelCase identifiers to snake_case |

## Examples

### Dry Run

```js
await migrate({
  databaseUrl: process.env.DATABASE_URL,
  direction: 'up',
  dir: 'migrations',
  dryRun: true,
  verbose: true,
  log: console.log,
});
```

### Run Down Migrations

```js
await migrate({
  databaseUrl: process.env.DATABASE_URL,
  direction: 'down',
  count: 1, // roll back one migration
  dir: 'migrations',
});
```

### In Test Setup

```js
beforeAll(async () => {
  await migrate({
    databaseUrl: process.env.TEST_DATABASE_URL,
    direction: 'up',
    dir: 'migrations',
    migrationsTable: 'pgmigrations',
    log: () => {}, // suppress output
  });
});

afterAll(async () => {
  await migrate({
    databaseUrl: process.env.TEST_DATABASE_URL,
    direction: 'down',
    count: Infinity,
    dir: 'migrations',
    log: () => {},
  });
});
```

### Multiple Migration Directories

```js
await migrate({
  databaseUrl: process.env.DATABASE_URL,
  direction: 'up',
  dir: ['migrations/core', 'migrations/tenant'],
  useGlob: true,
});
```
