# Troubleshooting

## SSL Connections

**Set PGSSLMODE environment variable:**
```bash
PGSSLMODE=require node-pg-migrate up
```

**Or append to DATABASE_URL:**
```
postgres://user:pass@host/db?ssl=true
```

**Self-signed certificates (pg v8+):**
Self-signed certs are rejected by default. Use `--no-reject-unauthorized` CLI flag or configure SSL via JSON connection settings.

Do not combine `--no-reject-unauthorized` with `?ssl=true` in the URL — the URL parameter overrides the flag.

**Custom SSL in JSON config:**
```json
{
  "db": {
    "host": "myhost",
    "database": "mydb",
    "ssl": {
      "rejectUnauthorized": false,
      "ca": "-----BEGIN CERTIFICATE-----\n..."
    }
  }
}
```

## Case Sensitivity

PostgreSQL folds unquoted identifiers to lowercase. Quoted identifiers (`"MyTable"`) are case-sensitive.

Best practices:
- Use lowercase identifiers everywhere
- Enable `--decamelize` to auto-convert camelCase to snake_case
- Be consistent with quoting in raw SQL

## Connection Refused

**Check password special characters:** Characters like `@`, `#`, `%` in passwords can break `DATABASE_URL` parsing. URL-encode them or use individual connection parameters.

**Verify connectivity:**
```bash
telnet 127.0.0.1 5432
```

If connection fails, the PostgreSQL server is down or a firewall is blocking access.

## Transaction Errors

**`addTypeValue` fails inside transaction:**
When adding values to an enum type created in a previous migration, the operation fails inside a transaction.

Solution: Call `pgm.noTransaction()` at the start of the migration.

```js
exports.up = (pgm) => {
  pgm.noTransaction();
  pgm.addTypeValue('my_enum', 'new_value');
};
exports.down = false;
```

**`CREATE INDEX CONCURRENTLY` fails:**
Same issue — concurrent index creation cannot run inside a transaction.

```js
exports.up = (pgm) => {
  pgm.noTransaction();
  pgm.createIndex('large_table', 'column', { concurrently: true });
};
```

## Environment Variables

**Required (one of these must be set):**
- `DATABASE_URL` — Full connection string
- `PGHOST` + `PGUSER` + `PGDATABASE` — Individual parameters

**Supported PostgreSQL variables:**

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | Full connection string |
| `PGHOST` | Server hostname (default: localhost) |
| `PGUSER` | Username |
| `PGPASSWORD` | Password |
| `PGDATABASE` | Database name |
| `PGPORT` | Port (default: 5432) |
| `PGSSLMODE` | SSL mode (disable, allow, prefer, require, verify-ca, verify-full) |

## Advisory Lock Stuck

If a migration crashes or is killed, the advisory lock may persist for the database session. The lock is session-level and will release when the connection closes.

If migrations are stuck waiting for a lock:
1. Find the blocking session: `SELECT * FROM pg_stat_activity WHERE state = 'active';`
2. Terminate it: `SELECT pg_terminate_backend(pid);`
3. Or use `--no-lock` to bypass locking (use with caution)

## Migration Order Violations

By default, `--check-order` is enabled. If a migration file has a timestamp earlier than an already-executed migration, it will fail.

To disable: `--no-check-order` or set `checkOrder: false` in config.

## TypeScript Setup

TypeScript is supported out of the box via the jiti loader. No compilation step required.

```json
{
  "scripts": {
    "migrate": "node-pg-migrate -j ts"
  }
}
```

Works with npm, pnpm, yarn, and bun.
