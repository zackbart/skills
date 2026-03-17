# NanoMDM Storage Backends & Schemas

## Storage Interfaces (storage/)

### AllStorage (the union of all storage interfaces)

```go
type AllStorage interface {
    ServiceStore
    PushStore
    PushCertStore
    CommandEnqueuer
    CertAuthStore
    CertAuthRetriever
    StoreMigrator
    TokenUpdateTallyStore
    PushCertStorer
}
```

### ServiceStore

```go
type ServiceStore interface {
    CheckinStore
    CommandAndReportResultsStore
    BootstrapTokenStore
}
```

### CheckinStore

```go
type CheckinStore interface {
    StoreAuthenticate(r *mdm.Request, msg *mdm.Authenticate) error
    StoreTokenUpdate(r *mdm.Request, msg *mdm.TokenUpdate) error
    Disable(r *mdm.Request) error
    UserAuthenticateStore
}

type UserAuthenticateStore interface {
    StoreUserAuthenticate(r *mdm.Request, msg *mdm.UserAuthenticate) error
}
```

### CommandAndReportResultsStore

```go
type CommandAndReportResultsStore interface {
    StoreCommandReport(r *mdm.Request, report *mdm.CommandResults) error
    RetrieveNextCommand(r *mdm.Request, skipNotNow bool) (*mdm.Command, error)
    ClearQueue(r *mdm.Request) error
}
```

### CommandEnqueuer

```go
type CommandEnqueuer interface {
    EnqueueCommand(ctx context.Context, id []string, cmd *mdm.Command) (map[string]error, error)
}
```

### PushStore

```go
type PushStore interface {
    RetrievePushInfo(ctx context.Context, ids []string) (map[string]*mdm.Push, error)
}
```

### PushCertStore & PushCertStorer

```go
type PushCertStore interface {
    IsPushCertStale(ctx context.Context, topic string, staleToken string) (bool, error)
    RetrievePushCert(ctx context.Context, topic string) (cert *tls.Certificate, staleToken string, err error)
}

type PushCertStorer interface {
    StorePushCert(ctx context.Context, pemCert, pemKey []byte) error
}
```

### CertAuthStore & CertAuthRetriever

```go
type CertAuthStore interface {
    HasCertHash(r *mdm.Request, hash string) (bool, error)
    EnrollmentHasCertHash(r *mdm.Request, hash string) (bool, error)
    IsCertHashAssociated(r *mdm.Request, hash string) (bool, error)
    AssociateCertHash(r *mdm.Request, hash string) error
}

type CertAuthRetriever interface {
    EnrollmentFromHash(ctx context.Context, hash string) (string, error)
}
```

### BootstrapTokenStore

```go
type BootstrapTokenStore interface {
    StoreBootstrapToken(r *mdm.Request, msg *mdm.SetBootstrapToken) error
    RetrieveBootstrapToken(r *mdm.Request, msg *mdm.GetBootstrapToken) (*mdm.BootstrapToken, error)
}
```

### TokenUpdateTallyStore

```go
type TokenUpdateTallyStore interface {
    RetrieveTokenUpdateTally(ctx context.Context, id string) (int, error)
}
```

### StoreMigrator

```go
type StoreMigrator interface {
    RetrieveMigrationCheckins(context.Context, chan<- interface{}) error
}
```

## MySQL Backend (storage/mysql/)

### Configuration

```go
store, err := mysql.New(
    mysql.WithDSN("user:password@/dbname"),
    mysql.WithLogger(logger),
    mysql.WithDeleteCommands(),  // enable command cleanup after response
)
```

CLI: `-storage mysql -storage-dsn user:pass/dbname -storage-options delete=1`

Requires MySQL 8.0.19+.

### Schema (storage/mysql/schema.sql)

```sql
-- Core tables
CREATE TABLE devices (
    id VARCHAR(255) NOT NULL PRIMARY KEY,
    identity_cert TEXT NULL,
    serial_number VARCHAR(127) NULL,
    unlock_token MEDIUMBLOB NULL,
    unlock_token_at TIMESTAMP NULL,
    authenticate TEXT NOT NULL,
    authenticate_at TIMESTAMP NOT NULL,
    token_update TEXT NULL,
    token_update_at TIMESTAMP NULL,
    bootstrap_token_b64 TEXT NULL,
    bootstrap_token_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE users (
    id VARCHAR(255) NOT NULL,
    device_id VARCHAR(255) NOT NULL,
    user_short_name VARCHAR(255) NULL,
    user_long_name VARCHAR(255) NULL,
    token_update TEXT NULL,
    token_update_at TIMESTAMP NULL,
    user_authenticate TEXT NULL,
    user_authenticate_at TIMESTAMP NULL,
    user_authenticate_digest TEXT NULL,
    user_authenticate_digest_at TIMESTAMP NULL,
    PRIMARY KEY (id, device_id),
    FOREIGN KEY (device_id) REFERENCES devices (id) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE enrollments (
    id VARCHAR(255) NOT NULL PRIMARY KEY,
    device_id VARCHAR(255) NOT NULL,
    user_id VARCHAR(255) NULL,
    type VARCHAR(31) NOT NULL,
    topic VARCHAR(255) NOT NULL,
    push_magic VARCHAR(127) NOT NULL,
    token_hex VARCHAR(255) NOT NULL,
    enabled BOOLEAN NOT NULL DEFAULT 1,
    token_update_tally INTEGER NOT NULL DEFAULT 1,
    last_seen_at TIMESTAMP NOT NULL,
    FOREIGN KEY (device_id) REFERENCES devices (id) ON DELETE CASCADE
);

CREATE TABLE commands (
    command_uuid VARCHAR(127) NOT NULL PRIMARY KEY,
    request_type VARCHAR(63) NOT NULL,
    command MEDIUMTEXT NOT NULL
);

CREATE TABLE command_results (
    id VARCHAR(255) NOT NULL,
    command_uuid VARCHAR(127) NOT NULL,
    status VARCHAR(31) NOT NULL,
    result MEDIUMTEXT NOT NULL,
    not_now_at TIMESTAMP NULL,
    not_now_tally INTEGER NOT NULL DEFAULT 0,
    PRIMARY KEY (id, command_uuid),
    FOREIGN KEY (id) REFERENCES enrollments (id) ON DELETE CASCADE,
    FOREIGN KEY (command_uuid) REFERENCES commands (command_uuid) ON DELETE CASCADE
);

CREATE TABLE enrollment_queue (
    id VARCHAR(255) NOT NULL,
    command_uuid VARCHAR(127) NOT NULL,
    active BOOLEAN NOT NULL DEFAULT 1,
    priority TINYINT NOT NULL DEFAULT 0,
    PRIMARY KEY (id, command_uuid),
    FOREIGN KEY (id) REFERENCES enrollments (id) ON DELETE CASCADE,
    FOREIGN KEY (command_uuid) REFERENCES commands (command_uuid) ON DELETE CASCADE
);

-- Queue view
CREATE OR REPLACE VIEW view_queue AS
SELECT q.id, q.created_at, q.active, q.priority,
       c.command_uuid, c.request_type, c.command,
       r.updated_at AS result_updated_at, r.status, r.result
FROM enrollment_queue AS q
    INNER JOIN commands AS c ON q.command_uuid = c.command_uuid
    LEFT JOIN command_results r ON r.command_uuid = q.command_uuid AND r.id = q.id
ORDER BY q.priority DESC, q.created_at;

CREATE TABLE push_certs (
    topic VARCHAR(255) NOT NULL PRIMARY KEY,
    cert_pem TEXT NOT NULL,
    key_pem TEXT NOT NULL,
    stale_token INTEGER NOT NULL
);

CREATE TABLE cert_auth_associations (
    id VARCHAR(255) NOT NULL,
    sha256 CHAR(64) NOT NULL,
    PRIMARY KEY (id, sha256)
);
```

### Command Queue Behavior (MySQL)

- **Enqueue**: INSERT into `commands` + INSERT into `enrollment_queue` (in transaction)
- **Retrieve next**: SELECT from `enrollment_queue` JOIN `commands` LEFT JOIN `command_results` WHERE active=1 AND (no result OR NotNow). Ordered by priority DESC, created_at ASC.
- **Store report**: On Idle: just update last_seen. On NotNow: UPSERT into command_results. Otherwise: UPSERT into command_results. With `delete=1`: DELETE queue entry, result, and orphaned command.
- **Clear queue**: SET active=0 for all active queue items for device and sub-enrollments.
- **NotNow handling**: `skipNotNow` parameter skips NotNow commands in retrieval.

## PostgreSQL Backend (storage/pgsql/)

### Configuration

```go
store, err := pgsql.New(
    pgsql.WithDSN("postgres://user:pass@host:5432/dbname"),
    pgsql.WithLogger(logger),
    pgsql.WithDeleteCommands(),
)
```

CLI: `-storage pgsql -storage-dsn postgres://user:pass@host/dbname -storage-options delete=1`

Requires PostgreSQL 9.5+ (for ON CONFLICT).

Schema is functionally identical to MySQL with PostgreSQL syntax. Uses triggers for `updated_at` timestamps and `BYTEA` for unlock tokens.

## Key-Value Backend (storage/kv/)

The `kv` package provides shared command queue and storage logic using abstract `kv.TxnCRUDBucket` interfaces. Both `inmem` and `diskv` (and `filekv`) backends build on this.

### KV Structure

```go
type KV struct {
    certAuth, queue, pushCert, users kv.TxnCRUDBucket
    devices, enrollments             kv.TxnBucketWithCRUD
}
```

Six separate "buckets" for different data domains. Uses dot-separated key naming (e.g., `enrollment_id.last_seen_at`).

### In-Memory Backend (storage/inmem/)

```go
store := inmem.New()  // All data volatile -- lost on process exit
```

CLI: `-storage inmem`

### Diskv / FileKV Backend (storage/diskv/)

```go
store := diskv.New("/path/to/db")
```

Uses `github.com/peterbourgon/diskv/v3` for persistent key-value storage on disk.

**CLI name**: `-storage filekv` (there is no `-storage diskv` option). The CLI `filekv` backend maps directly to `diskv.New(dsn)`. Default DSN is `dbkv`.

### File Backend (storage/file/) -- DEPRECATED

Legacy filesystem backend. Each enrollment gets a directory with files like:
- `Authenticate.plist`, `TokenUpdate.plist`, `UnlockToken.dat`
- `SerialNumber.txt`, `Identity.pem`, `Disabled`
- `BootstrapToken.dat`, `TokenUpdate.tally.txt`
- `Queue/`, `QueueNotNow/`, `QueueDone/`, `QueueInactive/` directories

CLI: `-storage file -storage-dsn /path/to/db -storage-options enable_deprecated=1`

## Multi-Storage (storage/allmulti/)

Dispatches all storage calls to multiple backends in parallel. First backend is authoritative (returns values); others are fire-and-forget (errors logged).

```go
store := allmulti.New(logger, store1, store2, store3)
```

CLI: Multiple `-storage`/`-storage-dsn` flag pairs.

**Warning**: Only useful if all backends have been used since initial enrollment. No sync/backfill mechanism exists.

## Common Storage Patterns

### StoreAuthenticate (all backends)

1. Store device identity cert, serial number, raw authenticate plist
2. Clear bootstrap token (invalidated on re-enrollment)
3. Reset token update tally to 0

### StoreTokenUpdate (all backends)

1. Store raw TokenUpdate for device or user channel
2. Store UnlockToken separately (if present, device channel only)
3. UPSERT enrollment record with push topic, push magic, token hex
4. Set enabled=true, increment token_update_tally
5. For user channels: associate sub-enrollment with parent device

### Disable (all backends)

1. Must be called on device channel only (ParentID must be empty)
2. Disables all enrollments for the device (including user channels)
3. Resets token_update_tally to 0
