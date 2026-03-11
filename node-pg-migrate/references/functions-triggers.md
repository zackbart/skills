# Functions & Triggers

## createFunction

```
pgm.createFunction(function_name, function_params, function_options, definition)
```

Reverse: `dropFunction`.

**function_params**: array of strings or objects.

Object format:

| Property | Type | Description |
|----------|------|-------------|
| `mode` | string | IN, OUT, INOUT, or VARIADIC |
| `name` | string | Parameter name |
| `type` | string | Data type |
| `default` | string | Default value |

**function_options:**

| Option | Type | Values |
|--------|------|--------|
| `returns` | string | Return type (e.g., 'void', 'trigger', 'integer') |
| `language` | string | plpgsql, sql, plv8, etc. |
| `replace` | boolean | Use CREATE OR REPLACE |
| `window` | boolean | Mark as window function |
| `behavior` | string | IMMUTABLE, STABLE, or VOLATILE |
| `security` | string | INVOKER or DEFINER |
| `onNull` | boolean | RETURNS NULL ON NULL INPUT |
| `parallel` | string | UNSAFE, RESTRICTED, or SAFE |

### Examples

```js
// Simple SQL function
pgm.createFunction(
  'add_numbers',
  [
    { name: 'a', type: 'integer' },
    { name: 'b', type: 'integer' },
  ],
  { returns: 'integer', language: 'sql', behavior: 'IMMUTABLE' },
  'SELECT a + b'
);

// PL/pgSQL trigger function
pgm.createFunction(
  'update_updated_at',
  [],
  { returns: 'trigger', language: 'plpgsql', replace: true },
  `
  BEGIN
    NEW.updated_at = current_timestamp;
    RETURN NEW;
  END;
  `
);

// Function with string params
pgm.createFunction(
  'get_user_count',
  ['text'],
  { returns: 'integer', language: 'sql' },
  "SELECT count(*)::integer FROM users WHERE role = $1"
);
```

## dropFunction

```
pgm.dropFunction(function_name, function_params, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## renameFunction

```
pgm.renameFunction(old_function_name, function_params, new_function_name)
```

Reverse: renames back.

---

## createTrigger

```
pgm.createTrigger(table_name, trigger_name, trigger_options)
```

Reverse: `dropTrigger`.

**trigger_options:**

| Option | Type | Description |
|--------|------|-------------|
| `when` | string | BEFORE, AFTER, or INSTEAD OF |
| `operation` | string or string[] | INSERT, UPDATE [OF ...], DELETE, TRUNCATE |
| `constraint` | boolean | Create as constraint trigger |
| `function` | Name | Function to execute |
| `functionParams` | array | Arguments for the function |
| `level` | string | STATEMENT or ROW |
| `condition` | string | WHEN clause expression |
| `deferrable` | boolean | Make deferrable (constraint triggers only) |
| `deferred` | boolean | Initially deferred (constraint triggers only) |
| `definition` | string | Inline function body (auto-creates function with trigger name) |

### Examples

```js
// Trigger using existing function
pgm.createTrigger('users', 'users_updated_at', {
  when: 'BEFORE',
  operation: 'UPDATE',
  level: 'ROW',
  function: 'update_updated_at',
});

// Trigger on multiple operations
pgm.createTrigger('audit_log', 'track_changes', {
  when: 'AFTER',
  operation: ['INSERT', 'UPDATE', 'DELETE'],
  level: 'ROW',
  function: 'log_changes',
});

// Trigger with inline function definition
pgm.createTrigger('products', 'validate_price', {
  when: 'BEFORE',
  operation: ['INSERT', 'UPDATE'],
  level: 'ROW',
  definition: `
    BEGIN
      IF NEW.price < 0 THEN
        RAISE EXCEPTION 'Price must be non-negative';
      END IF;
      RETURN NEW;
    END;
  `,
});

// Conditional trigger
pgm.createTrigger('orders', 'notify_on_complete', {
  when: 'AFTER',
  operation: 'UPDATE',
  level: 'ROW',
  condition: "NEW.status = 'completed' AND OLD.status != 'completed'",
  function: 'notify_order_complete',
});
```

## dropTrigger

```
pgm.dropTrigger(table_name, trigger_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |

## renameTrigger

```
pgm.renameTrigger(table_name, old_trigger_name, new_trigger_name)
```

Reverse: renames back.
