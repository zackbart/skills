# Tables & Columns

## createTable

```
pgm.createTable(tablename, columns, options)
```

Creates a new table. Reverse: `dropTable`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `temporary` | boolean | Create as temporary table |
| `ifNotExists` | boolean | Skip if table already exists |
| `inherits` | Name | Parent table to inherit from |
| `constraints` | object | Table-level constraints (same as addConstraint expression) |
| `like` | Name or object | Create table based on existing table structure |
| `comment` | string | Table comment |
| `unlogged` | boolean | Create as unlogged table |

**Like options (when `like` is an object):**

| Option | Type | Values |
|--------|------|--------|
| `including` | string or string[] | COMMENTS, CONSTRAINTS, DEFAULTS, IDENTITY, INDEXES, STATISTICS, STORAGE, ALL |
| `excluding` | string or string[] | Same as `including` |

## dropTable

```
pgm.dropTable(tablename, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## renameTable

```
pgm.renameTable(tablename, new_tablename)
```

Reverse: renames back.

## alterTable

```
pgm.alterTable(tablename, options)
```

**Options:**

| Option | Type | Values |
|--------|------|--------|
| `levelSecurity` | string | DISABLE, ENABLE, FORCE, NO FORCE |
| `unlogged` | boolean | Set/unset UNLOGGED |

---

## Column Definitions

When creating tables or adding columns, column values can be:

- **A string** — treated as the type: `{ age: 'integer' }`
- **`'id'`** — shorthand for `{ type: 'serial', primaryKey: true }`
- **An object** with these properties:

| Option | Type | Description |
|--------|------|-------------|
| `type` | string | PostgreSQL data type |
| `collation` | string | Collation for the column |
| `unique` | boolean | Add unique constraint |
| `primaryKey` | boolean | Mark as primary key |
| `notNull` | boolean | Add NOT NULL constraint |
| `default` | string | Default value (use `pgm.func()` for SQL expressions) |
| `check` | string | Check constraint SQL expression |
| `references` | Name | Foreign key table reference |
| `referencesConstraintName` | string | Name for the FK constraint |
| `referencesConstraintComment` | string | Comment on the FK constraint |
| `onDelete` | string | ON DELETE action (CASCADE, SET NULL, RESTRICT, etc.) |
| `onUpdate` | string | ON UPDATE action |
| `match` | string | FULL or SIMPLE |
| `deferrable` | boolean | Make constraint deferrable |
| `deferred` | boolean | Set constraint initially deferred |
| `comment` | string | Column comment |
| `expressionGenerated` | string | Generated column expression |
| `sequenceGenerated` | object | Identity column configuration |
| `precedence` | string | ALWAYS or BY DEFAULT (for identity columns) |

### Type Aliases

| Alias | PostgreSQL Type |
|-------|----------------|
| `'id'` | `serial PRIMARY KEY` |
| `'int'` | `integer` |
| `'string'` | `text` |
| `'float'` | `real` |
| `'double'` | `double precision` |
| `'datetime'` | `timestamp` |
| `'bool'` | `boolean` |

### Custom Shorthands

Define reusable column definitions via the `shorthands` export:

```js
exports.shorthands = {
  created_at: {
    type: 'timestamp',
    notNull: true,
    default: new PgLiteral('current_timestamp'),
  },
};

// Then use in columns:
pgm.createTable('users', {
  id: 'id',
  created_at: 'created_at', // expands to the shorthand definition
});
```

## addColumns

```
pgm.addColumns(tablename, new_columns, options)
```

Alias: `addColumn`. Reverse: `dropColumns`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifNotExists` | boolean | Skip columns that already exist |

## dropColumns

```
pgm.dropColumns(tablename, columns, options)
```

Alias: `dropColumn`.

- `columns`: string[], object, or string

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Skip if column doesn't exist |
| `cascade` | boolean | Drop dependent objects |

## renameColumn

```
pgm.renameColumn(tablename, old_column_name, new_column_name)
```

Reverse: renames back.

## alterColumn

```
pgm.alterColumn(tablename, column_name, column_options)
```

**Column options:**

| Option | Type | Description |
|--------|------|-------------|
| `default` | string or null | Set or drop default (null drops) |
| `type` | string | Change data type |
| `notNull` | boolean | Set NOT NULL |
| `allowNull` | boolean | Drop NOT NULL |
| `using` | string | USING expression for type change |
| `collation` | string | Change collation |
| `comment` | string | Set column comment |
| `sequenceGenerated` | object, null, or false | Set/drop identity column |
