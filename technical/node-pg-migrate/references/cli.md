# CLI Reference (v9.0.0-alpha.6)

## Requirements

- Node.js 20.11+
- PostgreSQL 13+
- `pg` package (peer dependency)

## Installation

```bash
npm add pg
npm add -D node-pg-migrate
```

Add to `package.json`:
```json
{ "scripts": { "migrate": "node-pg-migrate" } }
```

## Database Connection

```bash
# Environment variable
DATABASE_URL=postgres://user:pass@localhost/dbname npm run migrate up

# .env file (auto-loaded via dotenv)
# DATABASE_URL=postgres://user:pass@localhost/dbname

# Custom env var name
npm run migrate up -d MY_DB_URL

# Config file (default.json via node-config)
# { "db": { "user": "postgres", "host": "localhost", "database": "mydb", "port": 5432 } }

# Custom config file
npm run migrate up -f my-config.json
```

Set `PGSSLMODE=require` for SSL connections.

## Commands

| Command | Description |
|---------|-------------|
| `create {name}` | Generate a new timestamped migration file |
| `up` | Run all pending migrations |
| `up {N}` | Run next N pending migrations |
| `down` | Roll back the last migration |
| `down {N}` | Roll back N migrations |
| `redo` | Roll back and re-run the last migration |
| `redo {N}` | Roll back and re-run N migrations |

## CLI Options

### Connection

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--database-url-var` | `-d` | `DATABASE_URL` | Environment variable for connection string |
| `--config-file` | `-f` | — | Path to JSON config file |
| `--config-value` | — | `db` | Config section name |
| `--envPath` | — | `.env` | Custom .env file path |
| `--reject-unauthorized` | — | undefined | SSL rejectUnauthorized setting |

### Migration Files

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--migrations-dir` | `-m` | `migrations` | Migration folder path (supports globs) |
| `--use-glob` | — | false | Enable glob matching for migration files |
| `--migration-filename-format` | — | `timestamp` | Prefix format: `utc`, `timestamp`, or `index` |
| `--migration-file-language` | `-j` | `js` | File type: `js`, `ts`, or `sql` |
| `--template-file-name` | — | — | Custom template file path |
| `--ignore-pattern` | — | — | Regex/glob to skip files |

### Schema & Tracking

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--schema` | `-s` | `public` | Target schema(s), sets search_path |
| `--create-schema` | — | false | Auto-create schema if missing |
| `--migrations-schema` | — | (matches --schema) | Schema for tracking table |
| `--create-migrations-schema` | — | false | Auto-create tracking schema |
| `--migrations-table` | `-t` | `pgmigrations` | Migration tracking table name |

### Execution Control

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--check-order` | — | true | Validate migration order |
| `--single-transaction` | — | true | Wrap all pending migrations in one transaction |
| `--no-lock` | — | false | Disable advisory lock |
| `--fake` | — | false | Mark as run without executing |
| `--timestamp` | — | false | Treat numeric args as timestamps |
| `--decamelize` | — | false | Convert camelCase to snake_case |
| `--verbose` | — | true | Print SQL and debug info |

## JSON Configuration

```json
{
  "db": {
    "user": "postgres",
    "host": "localhost",
    "database": "myapp",
    "password": "secret",
    "port": 5432
  },
  "schema": "public",
  "createSchema": false,
  "migrationsDir": "migrations",
  "migrationsSchema": "public",
  "createMigrationsSchema": false,
  "migrationsTable": "pgmigrations",
  "migrationFilenameFormat": "utc",
  "migrationFileLanguage": "ts",
  "checkOrder": true,
  "verbose": true,
  "decamelize": false
}
```

## Migration File Formats

### JavaScript (default)
```bash
npm run migrate create my-migration
# Creates: migrations/{timestamp}_my-migration.js
```

### TypeScript
```bash
npm run migrate create my-migration -j ts
# Creates: migrations/{timestamp}_my-migration.ts
```

### SQL
```bash
npm run migrate create my-migration -j sql
# Creates: migrations/{timestamp}_my-migration.sql
# (up and down separated by `-- Down Migration` comment)
```
