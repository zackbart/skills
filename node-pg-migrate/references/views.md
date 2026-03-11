# Views & Materialized Views

## createView

```
pgm.createView(viewName, options, definition)
```

Reverse: `dropView`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `temporary` | boolean | Create as temporary view |
| `replace` | boolean | Use CREATE OR REPLACE |
| `recursive` | boolean | Create as RECURSIVE view |
| `columns` | string or string[] | Custom column names |
| `options` | object | View storage options (key-value pairs) |
| `checkOption` | string | CASCADED or LOCAL |

### Examples

```js
pgm.createView('active_users', { replace: true },
  "SELECT * FROM users WHERE deleted_at IS NULL"
);

pgm.createView('user_stats', {
  columns: ['user_id', 'post_count', 'comment_count'],
}, `
  SELECT u.id, COUNT(DISTINCT p.id), COUNT(DISTINCT c.id)
  FROM users u
  LEFT JOIN posts p ON p.user_id = u.id
  LEFT JOIN comments c ON c.user_id = u.id
  GROUP BY u.id
`);
```

## dropView

```
pgm.dropView(viewName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## alterView

```
pgm.alterView(viewName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `checkOption` | string or null | CASCADED, LOCAL, or null to reset |
| `options` | object | View options (null values reset) |

## alterViewColumn

```
pgm.alterViewColumn(viewName, columnName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `default` | string | Set column default |

## renameView

```
pgm.renameView(viewName, newViewName)
```

Reverse: renames back.

---

## createMaterializedView

```
pgm.createMaterializedView(viewName, options, definition)
```

Reverse: `dropMaterializedView`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifNotExists` | boolean | Skip if exists |
| `columns` | string or string[] | Custom column names |
| `tablespace` | string | Storage tablespace |
| `storageParameters` | object | Storage configuration (key-value) |
| `data` | boolean | WITH DATA or WITH NO DATA |

### Examples

```js
pgm.createMaterializedView('monthly_stats', { data: true }, `
  SELECT
    date_trunc('month', created_at) AS month,
    count(*) AS total_orders,
    sum(amount) AS total_revenue
  FROM orders
  GROUP BY 1
`);
```

## dropMaterializedView

```
pgm.dropMaterializedView(viewName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## alterMaterializedView

```
pgm.alterMaterializedView(viewName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `cluster` | string | Index to cluster on |
| `extension` | string | Dependent extension |
| `storageParameters` | object | Storage options (null values reset) |

## renameMaterializedView

```
pgm.renameMaterializedView(viewName, newViewName)
```

## renameMaterializedViewColumn

```
pgm.renameMaterializedViewColumn(viewName, columnName, newColumnName)
```

## refreshMaterializedView

```
pgm.refreshMaterializedView(viewName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `concurrently` | boolean | Refresh without locking (requires unique index) |
| `data` | boolean | WITH DATA or WITH NO DATA |
