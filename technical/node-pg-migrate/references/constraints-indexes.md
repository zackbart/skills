# Constraints & Indexes

## addConstraint

```
pgm.addConstraint(tablename, constraint_name, expression)
```

Alias: `createConstraint`. Reverse: `dropConstraint`.

- `expression` can be a raw SQL string or a definition object.

**Constraint definition object:**

| Property | Type | Description |
|----------|------|-------------|
| `check` | string or string[] | Check constraint SQL expression(s) |
| `unique` | Name or Name[] | Column(s) for unique constraint |
| `primaryKey` | Name or Name[] | Column(s) for primary key |
| `exclude` | string | Exclusion constraint SQL |
| `deferrable` | boolean | Make constraint deferrable |
| `deferred` | boolean | Initially deferred |
| `comment` | string | Constraint comment |
| `foreignKeys` | object or object[] | Foreign key definition(s) |

**Foreign key definition:**

| Property | Type | Description |
|----------|------|-------------|
| `columns` | Name or Name[] | Source column(s) |
| `references` | Name | Target table(column) reference |
| `referencesConstraintName` | string | Name for the FK constraint |
| `referencesConstraintComment` | string | Comment on FK constraint |
| `onDelete` | string | CASCADE, SET NULL, RESTRICT, NO ACTION, SET DEFAULT |
| `onUpdate` | string | Same options as onDelete |
| `match` | string | FULL or SIMPLE |

### Examples

```js
// Simple unique constraint
pgm.addConstraint('users', 'unique_email', { unique: 'email' });

// Composite primary key
pgm.addConstraint('user_roles', 'pk_user_roles', {
  primaryKey: ['user_id', 'role_id'],
});

// Foreign key
pgm.addConstraint('posts', 'fk_posts_user', {
  foreignKeys: {
    columns: 'user_id',
    references: 'users(id)',
    onDelete: 'CASCADE',
  },
});

// Multiple foreign keys
pgm.addConstraint('order_items', 'fk_order_items', {
  foreignKeys: [
    { columns: 'order_id', references: 'orders(id)', onDelete: 'CASCADE' },
    { columns: 'product_id', references: 'products(id)', onDelete: 'RESTRICT' },
  ],
});

// Check constraint
pgm.addConstraint('products', 'check_price', {
  check: 'price > 0',
});

// Raw SQL constraint
pgm.addConstraint('users', 'unique_lower_email',
  'UNIQUE (lower(email))'
);
```

## dropConstraint

```
pgm.dropConstraint(tablename, constraint_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## renameConstraint

```
pgm.renameConstraint(tablename, old_constraint_name, new_constraint_name)
```

Reverse: renames back.

---

## createIndex

```
pgm.createIndex(tablename, columns, options)
```

Alias: `addIndex`. Reverse: `dropIndex`.

- `columns`: string, string[], or array of objects with `{ name, sort, opclass }`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `name` | string | Custom index name (auto-generated if omitted) |
| `unique` | boolean | Create unique index |
| `where` | string | Partial index WHERE clause |
| `concurrently` | boolean | Create without locking (requires `noTransaction()`) |
| `ifNotExists` | boolean | Skip if index exists |
| `method` | string | Index type: btree, hash, gist, spgist, gin |
| `include` | string or string[] | INCLUDE clause columns |
| `nulls` | string | "distinct" or "not distinct" (unique indexes only) |

### Examples

```js
// Simple index
pgm.createIndex('users', 'email');

// Composite index
pgm.createIndex('posts', ['user_id', 'created_at']);

// Unique partial index
pgm.createIndex('users', 'email', {
  unique: true,
  where: 'deleted_at IS NULL',
});

// GIN index for JSONB
pgm.createIndex('documents', 'data', { method: 'gin' });

// Index with sort order
pgm.createIndex('posts', [
  { name: 'user_id', sort: 'ASC' },
  { name: 'created_at', sort: 'DESC' },
]);

// Concurrent index (requires noTransaction)
pgm.noTransaction();
pgm.createIndex('large_table', 'column', { concurrently: true });

// Index with operator class
pgm.createIndex('posts', [
  { name: 'title', opclass: 'text_pattern_ops' },
]);
```

## dropIndex

```
pgm.dropIndex(tablename, columns, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `name` | string | Explicit index name to drop |
| `concurrently` | boolean | Drop without locking |
| `ifExists` | boolean | Skip if not found |
| `cascade` | boolean | Drop dependent objects |
