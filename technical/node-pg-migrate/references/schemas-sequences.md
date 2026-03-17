# Schemas & Sequences

## createSchema

```
pgm.createSchema(schema_name, options)
```

Reverse: `dropSchema`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifNotExists` | boolean | Skip if schema exists |
| `authorization` | string | Owner of the schema |

## dropSchema

```
pgm.dropSchema(schema_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop all contained objects |

## renameSchema

```
pgm.renameSchema(old_schema_name, new_schema_name)
```

Reverse: renames back.

---

## Sequence Options

Shared options for `createSequence` and `alterSequence`:

| Option | Type | Description |
|--------|------|-------------|
| `type` | string | Sequence data type |
| `increment` | number | Increment value |
| `minvalue` | number, null, or false | Minimum value (null/false = NO MINVALUE) |
| `maxvalue` | number, null, or false | Maximum value (null/false = NO MAXVALUE) |
| `start` | number | First value |
| `cache` | number | Pre-allocate count |
| `cycle` | boolean | CYCLE or NO CYCLE |
| `owner` | string, null, or false | Owner column (null/false removes ownership) |

## createSequence

```
pgm.createSequence(sequence_name, options)
```

Reverse: `dropSequence`.

Additional options beyond the shared ones:

| Option | Type | Description |
|--------|------|-------------|
| `temporary` | boolean | Create as temporary |
| `ifNotExists` | boolean | Skip if exists |

### Examples

```js
pgm.createSequence('invoice_number_seq', {
  start: 1000,
  increment: 1,
  minvalue: 1000,
  maxvalue: 9999999,
});

pgm.createSequence('order_seq', {
  type: 'smallint',
  cycle: true,
  cache: 10,
});
```

## dropSequence

```
pgm.dropSequence(sequence_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## alterSequence

```
pgm.alterSequence(sequence_name, options)
```

Additional option beyond the shared ones:

| Option | Type | Description |
|--------|------|-------------|
| `restart` | number or true | Restart at value (true uses `start` value) |

## renameSequence

```
pgm.renameSequence(old_sequence_name, new_sequence_name)
```

Reverse: renames back.
