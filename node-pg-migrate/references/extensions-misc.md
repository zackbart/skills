# Extensions, Casts & Miscellaneous

## createExtension

```
pgm.createExtension(extension, options)
```

Alias: `addExtension`. Reverse: `dropExtension`.

- `extension`: string or string[] (install multiple at once)

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifNotExists` | boolean | Skip if already installed |
| `schema` | string | Schema to install extension objects into |

### Examples

```js
pgm.createExtension('uuid-ossp', { ifNotExists: true });
pgm.createExtension(['pgcrypto', 'hstore']);
```

## dropExtension

```
pgm.dropExtension(extension, options)
```

- `extension`: string or string[]

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

---

## createCast

```
pgm.createCast(source_type, target_type, options)
```

Reverse: `dropCast`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `functionName` | string | Function to perform the cast |
| `argumentTypes` | array | Function argument types (defaults to source_type) |
| `inout` | boolean | Use standard I/O conversion |
| `as` | string | `assignment` or `implicit` |

## dropCast

```
pgm.dropCast(source_type, target_type, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |

---

## pgm.sql()

```
pgm.sql(sql, args)
```

Execute raw SQL. Supports basic mustache-style templating.

- `sql`: SQL string to execute
- `args` (optional): Object with key-value pairs for `{{key}}` substitution

### Examples

```js
// Raw SQL
pgm.sql('CREATE EXTENSION IF NOT EXISTS "uuid-ossp"');

// With templating
pgm.sql('ALTER TABLE {{table}} ADD COLUMN status text', { table: 'users' });

// Multi-statement
pgm.sql(`
  CREATE OR REPLACE FUNCTION notify_trigger()
  RETURNS trigger AS $$
  BEGIN
    PERFORM pg_notify('changes', NEW.id::text);
    RETURN NEW;
  END;
  $$ LANGUAGE plpgsql;
`);
```

## pgm.func()

```
pgm.func(sql)
```

Creates an unescaped SQL literal for use in column defaults and other expressions.

```js
// Use in column definitions
pgm.createTable('users', {
  id: { type: 'uuid', default: pgm.func('gen_random_uuid()'), primaryKey: true },
  created_at: { type: 'timestamp', default: pgm.func('current_timestamp') },
});
```

## pgm.db

Access the underlying database connection (same connection as the running migration).

- `pgm.db.query(sql)` — Execute a query, returns promise with full result
- `pgm.db.select(sql)` — Execute a query, returns promise with rows only

Both follow the [pg.Client.query](https://node-postgres.com/api/client#client-query) API.

**These execute immediately**, unlike other `pgm` methods that queue SQL.

### Examples

```js
exports.up = async (pgm) => {
  // Data migration: read then transform
  const { rows } = await pgm.db.query('SELECT id, first_name, last_name FROM users');

  for (const row of rows) {
    await pgm.db.query(
      'UPDATE users SET display_name = $1 WHERE id = $2',
      [`${row.first_name} ${row.last_name}`, row.id]
    );
  }
};
```

## pgm.noTransaction()

```
pgm.noTransaction()
```

Disables transaction wrapping for the current migration. Required for:

- `CREATE INDEX CONCURRENTLY`
- `addTypeValue` on enum types created in previous migrations
- Any DDL that PostgreSQL forbids inside transactions

When disabled, a failed migration will not fully roll back — some changes may persist.
