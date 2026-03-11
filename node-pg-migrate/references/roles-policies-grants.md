# Roles, Policies & Grants

## createRole

```
pgm.createRole(role_name, options)
```

Reverse: `dropRole`.

**Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `superuser` | boolean | false | Superuser privilege |
| `createdb` | boolean | false | Can create databases |
| `createrole` | boolean | false | Can create roles |
| `inherit` | boolean | true | Inherits privileges |
| `login` | boolean | false | Can log in |
| `replication` | boolean | false | Replication privilege |
| `bypassrls` | boolean | — | Bypass row-level security |
| `limit` | number | — | Connection limit |
| `password` | string | — | Role password |
| `encrypted` | boolean | true | Store password encrypted |
| `valid` | string | — | Password expiration timestamp |
| `inRole` | string or string[] | — | Existing roles to join |
| `role` | string or string[] | — | Roles added as members |
| `admin` | string or string[] | — | Roles added as admins |

### Examples

```js
pgm.createRole('app_user', {
  login: true,
  password: 'secure_password',
  valid: '2025-12-31',
});

pgm.createRole('readonly', {
  login: false,
  inRole: ['app_user'],
});
```

## dropRole

```
pgm.dropRole(role_name)
```

## alterRole

```
pgm.alterRole(role_name, options)
```

Same options as `createRole`.

## renameRole

```
pgm.renameRole(old_role_name, new_role_name)
```

---

## createPolicy

```
pgm.createPolicy(tableName, policyName, options)
```

Reverse: `dropPolicy`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `command` | string | ALL, SELECT, INSERT, UPDATE, or DELETE |
| `role` | string or string[] | Applicable role(s) |
| `using` | string | Visibility check SQL (rows visible to SELECT/UPDATE/DELETE) |
| `check` | string | New row validation SQL (rows allowed for INSERT/UPDATE) |

### Examples

```js
// Enable RLS first
pgm.alterTable('documents', { levelSecurity: 'ENABLE' });

// Owner can see own rows
pgm.createPolicy('documents', 'owner_select', {
  command: 'SELECT',
  role: 'app_user',
  using: 'owner_id = current_setting(\'app.user_id\')::integer',
});

// Owner can insert own rows
pgm.createPolicy('documents', 'owner_insert', {
  command: 'INSERT',
  role: 'app_user',
  check: 'owner_id = current_setting(\'app.user_id\')::integer',
});

// Admin can do everything
pgm.createPolicy('documents', 'admin_all', {
  command: 'ALL',
  role: 'admin',
  using: 'true',
  check: 'true',
});
```

## dropPolicy

```
pgm.dropPolicy(tableName, policyName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `ifExists` | boolean | Drop only if exists |

## alterPolicy

```
pgm.alterPolicy(tableName, policyName, options)
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `role` | string | Updated role(s) |
| `using` | string | Updated visibility expression |
| `check` | string | Updated validation expression |

## renamePolicy

```
pgm.renamePolicy(tableName, policyName, newPolicyName)
```

---

## grantRoles

```
pgm.grantRoles(roles_from, roles_to, options)
```

Reverse: `revokeRoles`.

**Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `withAdminOption` | boolean | false | Grant with admin option |
| `onlyAdminOption` | boolean | false | Grant only admin option |
| `cascade` | boolean | false | Cascade |

## revokeRoles

```
pgm.revokeRoles(roles, roles_from, options)
```

**Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `onlyAdminOption` | boolean | false | Revoke only admin option |
| `cascade` | boolean | false | Cascade |

## grantOnTables

```
pgm.grantOnTables(options)
```

Reverse: `revokeOnTables`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `tables` | Name or Name[] | Table name(s) |
| `schema` | string | Required when tables is ALL |
| `privileges` | string[] or 'ALL' | SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER |
| `roles` | Name or Name[] | Role(s) to grant to |
| `withGrantOption` | boolean | Allow re-granting (default false) |
| `cascade` | boolean | Cascade (default false) |

### Example

```js
pgm.grantOnTables({
  tables: ['users', 'posts'],
  privileges: ['SELECT', 'INSERT', 'UPDATE'],
  roles: 'app_user',
});
```

## revokeOnTables

```
pgm.revokeOnTables(options)
```

Same options as `grantOnTables`.

## grantOnSchemas

```
pgm.grantOnSchemas(options)
```

Reverse: `revokeOnSchemas`.

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `schemas` | Name or Name[] | Schema name(s) |
| `privileges` | string[] or 'ALL' | CREATE, USAGE |
| `roles` | Name or Name[] | Role(s) |
| `withGrantOption` | boolean | Default false |
| `onlyGrantOption` | boolean | Default false |
| `cascade` | boolean | Default false |

## revokeOnSchemas

```
pgm.revokeOnSchemas(options)
```

Same options as `grantOnSchemas`.
