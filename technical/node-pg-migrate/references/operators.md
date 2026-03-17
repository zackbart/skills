# Operators

## createOperator

```
pgm.createOperator(operator_name, options)
```

Reverse: `dropOperator`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `procedure` | Name | Function implementing the operator |
| `left` | Name | Left operand type |
| `right` | Name | Right operand type |
| `commutator` | Name | Commutator operator |
| `negator` | Name | Negator operator |
| `restrict` | Name | Restriction selectivity function |
| `join` | Name | Join selectivity function |
| `hashes` | boolean | Supports hash joins |
| `merges` | boolean | Supports merge joins |

## dropOperator

```
pgm.dropOperator(operator_name, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |
| `cascade` | boolean | Drop dependent objects |
| `left` | Name | Left operand type |
| `right` | Name | Right operand type |

---

## createOperatorClass

```
pgm.createOperatorClass(operator_class_name, type, index_method, operator_list, options)
```

Reverse: `dropOperatorClass`.

- `type`: Data type for the operator class
- `index_method`: btree, hash, gist, gin, etc.
- `operator_list`: Array of operator/function definitions

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `default` | boolean | Make default for this type |
| `family` | string | Operator family to belong to |

## dropOperatorClass

```
pgm.dropOperatorClass(operator_class_name, index_method, options)
```

**Options:**

| Option | Type |
|--------|------|
| `ifExists` | boolean |
| `cascade` | boolean |

## renameOperatorClass

```
pgm.renameOperatorClass(old_name, index_method, new_name)
```

---

## createOperatorFamily

```
pgm.createOperatorFamily(operator_family_name, index_method)
```

Reverse: `dropOperatorFamily`.

## dropOperatorFamily

```
pgm.dropOperatorFamily(operator_family_name, index_method, options)
```

**Options:**

| Option | Type |
|--------|------|
| `ifExists` | boolean |
| `cascade` | boolean |

## renameOperatorFamily

```
pgm.renameOperatorFamily(old_name, index_method, new_name)
```

## addToOperatorFamily

```
pgm.addToOperatorFamily(operator_family_name, index_method, operator_list)
```

Reverse: `removeFromOperatorFamily`.

## removeFromOperatorFamily

```
pgm.removeFromOperatorFamily(operator_family_name, index_method, operator_list)
```

---

## Operator List Entry Format

Each entry in `operator_list` is an object:

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "function" or "operator" |
| `number` | number | Strategy or support number |
| `name` | Name | Operator or function name |
| `params` | array | Argument type(s) |
