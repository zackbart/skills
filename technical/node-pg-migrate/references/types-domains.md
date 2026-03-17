# Types & Domains

## createType

```
pgm.createType(type_name, values)
```

Alias: `addType`. Reverse: `dropType`.

- For **enum types**: `values` is `string[]`
- For **composite types**: `values` is an object of `{ attribute_name: type_string }`

### Examples

```js
// Enum type
pgm.createType('status_enum', ['draft', 'published', 'archived']);

// Composite type
pgm.createType('address', {
  street: 'text',
  city: 'text',
  state: 'varchar(2)',
  zip: 'varchar(10)',
});
```

## dropType

```
pgm.dropType(type_name)
```

## renameType

```
pgm.renameType(type_name, new_type_name)
```

Reverse: renames back.

## addTypeAttribute

```
pgm.addTypeAttribute(type_name, attribute_name, attribute_type)
```

Adds an attribute to a composite type. Reverse: `dropTypeAttribute`.

## dropTypeAttribute

```
pgm.dropTypeAttribute(type_name, attribute_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Skip if attribute doesn't exist |

## setTypeAttribute

```
pgm.setTypeAttribute(type_name, attribute_name, attribute_type)
```

Changes the data type of an existing composite type attribute.

## addTypeValue

```
pgm.addTypeValue(type_name, value, options)
```

Adds a value to an existing enum type. **Not reversible** — set `exports.down = false`.

**Requires `pgm.noTransaction()`** when the enum was created in a previous migration.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifNotExists` | boolean | Skip if value exists (default false) |
| `before` | string | Insert before this value |
| `after` | string | Insert after this value |

### Example

```js
exports.up = (pgm) => {
  pgm.noTransaction();
  pgm.addTypeValue('status_enum', 'pending', { before: 'published' });
};

exports.down = false;
```

## renameTypeAttribute

```
pgm.renameTypeAttribute(type_name, attribute_name, new_attribute_name)
```

## renameTypeValue

```
pgm.renameTypeValue(type_name, value, new_value)
```

Changes an enum value's label.

---

## createDomain

```
pgm.createDomain(domain_name, type, options)
```

Reverse: `dropDomain`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `default` | string | Default value |
| `collation` | string | Collation |
| `notNull` | boolean | NOT NULL constraint |
| `check` | string | Check constraint SQL |
| `constraintName` | string | Name for the constraint |

### Examples

```js
pgm.createDomain('email_address', 'text', {
  notNull: true,
  check: "VALUE ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'",
});

pgm.createDomain('positive_integer', 'integer', {
  check: 'VALUE > 0',
  constraintName: 'positive_check',
});
```

## dropDomain

```
pgm.dropDomain(domain_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## alterDomain

```
pgm.alterDomain(domain_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `default` | string | New default value |
| `collation` | string | New collation |
| `notNull` | boolean | Set NOT NULL |
| `allowNull` | boolean | Drop NOT NULL |
| `check` | string | New check constraint |
| `constraintName` | string | Constraint name |

## renameDomain

```
pgm.renameDomain(old_domain_name, new_domain_name)
```

Reverse: renames back.
