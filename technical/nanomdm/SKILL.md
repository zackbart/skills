---
name: nanomdm
description: >
  NanoMDM Apple MDM server assistant (v0.9.x). Use when working with NanoMDM
  (github.com/micromdm/nanomdm) — building MDM check-in handlers, command queues, APNs push,
  storage backends, service middleware, webhook callbacks, certificate auth, enrollment profiles,
  or the NanoMDM HTTP API. Also triggers when code imports 'github.com/micromdm/nanomdm'.
---

# NanoMDM Apple MDM Server (v0.9.x)

You are an expert at building and operating Apple MDM solutions with NanoMDM. This skill references the **NanoMDM v0.9.x** source code and documentation from `github.com/micromdm/nanomdm`.

## Core Principles

1. **Minimalist composable architecture** -- NanoMDM is a thin layer between HTTP handlers, a service interface layer, and storage abstractions. Each layer is independently composable.
2. **Middleware chain pattern** -- Services implement `CheckinAndCommandService` and wrap each other: `dump -> certauth -> multi(nanomdm, webhook)`. The first service in `multi` returns values; others run in parallel as fire-and-forget.
3. **Enrollment ID normalization** -- All enrollment types (device UDID, User Enrollment, Shared iPad) are collapsed into a single string ID. Device channel: `UUID`. User channel: `UUID:UUID`. Shared iPad: `UUID:ShortName`.
4. **Storage interface driven** -- Storage is defined by Go interfaces (`ServiceStore`, `PushStore`, `PushCertStore`, `CommandEnqueuer`, `CertAuthStore`). Multiple backends implement these interfaces.
5. **Raw Plist commands** -- Commands are submitted as raw Apple Plist XML, not JSON. Use the `cmdr.py` tool or construct plists directly.
6. **Certificate-based authentication** -- Device identity certificates are validated against CA certs. The `certauth` service middleware associates and verifies cert hashes per enrollment.

## How to Use This Skill

Before generating code, load the relevant reference file(s):

- **Architecture & service interfaces**: `references/architecture.md`
- **Storage backends & schemas**: `references/storage.md`
- **HTTP API & endpoints**: `references/http-api.md`
- **Configuration & CLI flags**: `references/configuration.md`
- **MDM protocol types & check-in handling**: `references/mdm-protocol.md`

## Quick Reference

### Module Path & Dependencies

```
module github.com/micromdm/nanomdm
go 1.18
```

Key dependencies: `github.com/micromdm/nanolib`, `github.com/micromdm/plist`, `github.com/go-sql-driver/mysql`, `github.com/lib/pq`, `github.com/smallstep/pkcs7`, `github.com/peterbourgon/diskv/v3`

### Core Service Interface

```go
// CheckinAndCommandService is the main composable interface
type CheckinAndCommandService interface {
    Checkin                  // Authenticate, TokenUpdate, CheckOut, SetBootstrapToken, GetBootstrapToken, UserAuthenticate, DeclarativeManagement, GetToken
    CommandAndReportResults  // CommandAndReportResults(r, results) (*Command, error)
}
```

### Service Middleware Chain (default cmd/nanomdm setup)

```
HTTP Request
  -> cert extraction (Mdm-Signature header or -cert-header)
  -> cert verification (CA pool)
  -> dump (optional, -dump flag)
  -> certauth (certificate-enrollment association)
  -> multi [nanomdm service, webhook service]
       -> nanomdm dispatches to storage
       -> webhook sends HTTP callbacks
```

### Storage Backends

| Backend | Flag | DSN Example | Notes |
|---------|------|-------------|-------|
| filekv | `-storage filekv` | `-storage-dsn dbkv` | Default. Uses diskv package internally. Zero deps. |
| mysql | `-storage mysql` | `-storage-dsn user:pass/dbname` | MySQL 8.0.19+. Run schema.sql first. |
| pgsql | `-storage pgsql` | `-storage-dsn postgres://user:pass@host/db` | PostgreSQL 9.5+. Run schema.sql first. |
| inmem | `-storage inmem` | (none) | Volatile in-memory. All data lost on exit. |
| file | `-storage file` | `-storage-dsn /path/to/db` | Deprecated. Requires `-storage-options enable_deprecated=1`. |
| multi | Multiple `-storage` flags | | First backend is authoritative; others are write-only. |

> **Note**: There is no `-storage diskv` CLI option. The `filekv` backend uses the `diskv` Go package internally. The `inmem` backend also uses the shared `kv` package with in-memory buckets.

### HTTP API Endpoints

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/mdm` | PUT | Cert | MDM check-in and command reports (primary) |
| `/checkin` | PUT | Cert | Separate check-in endpoint (if `-checkin` flag) |
| `/v1/pushcert` | PUT | API key | Upload APNs push cert+key (PEM concatenated) |
| `/v1/push/{id,...}` | GET | API key | Send APNs push to enrollment ID(s) |
| `/v1/enqueue/{id,...}` | PUT | API key | Enqueue raw plist command. `?nopush=1` to skip push. |
| `/v1/escrowkeyunlock` | POST | API key | Activation Lock bypass via Apple API |
| `/migration` | PUT | API key | Enrollment migration endpoint (if `-migration` flag) |
| `/version` | GET | None | Server version JSON |
| `/authproxy/` | * | Cert | Reverse proxy with MDM cert auth (if `-auth-proxy-url`) |

### Webhook Event Topics

`mdm.Authenticate`, `mdm.TokenUpdate`, `mdm.CheckOut`, `mdm.Connect`, `mdm.UserAuthenticate`, `mdm.SetBootstrapToken`, `mdm.GetBootstrapToken`, `mdm.DeclarativeManagement`, `mdm.GetToken`

### Enrollment Types

```go
const (
    Device              = 1 + iota  // macOS UDID, iOS UDID
    User                            // macOS UserID (UDID:UserID)
    UserEnrollmentDevice            // iOS EnrollmentID (BYOD)
    UserEnrollment                  // iOS EnrollmentUserID
    SharediPad                      // Shared iPad (UDID:UserShortName)
)
```

### Common Operations

```bash
# Upload push certificate
cat push.pem push.key | curl -T - -u nanomdm:APIKEY 'http://host:9000/v1/pushcert'

# Send push notification
curl -u nanomdm:APIKEY 'http://host:9000/v1/push/ENROLLMENT-ID'

# Enqueue command
./cmdr.py SecurityInfo | curl -T - -u nanomdm:APIKEY 'http://host:9000/v1/enqueue/ENROLLMENT-ID'

# Enqueue without push
./cmdr.py ProfileList | curl -T - -u nanomdm:APIKEY 'http://host:9000/v1/enqueue/ENROLLMENT-ID?nopush=1'

# Multi-target push/enqueue (comma-separated IDs)
curl -u nanomdm:APIKEY 'http://host:9000/v1/push/ID1,ID2,ID3'
```

### Docker

```bash
docker pull ghcr.io/micromdm/nanomdm:latest
docker run ghcr.io/micromdm/nanomdm:latest -ca /path/to/ca.pem -api APIKEY
```

### Building from Source

```bash
make           # builds for current OS/arch
make release   # cross-compile all platforms
make test      # run tests
```

## Common Mistakes

- **Backend name confusion**: The CLI flag is `-storage filekv`, not `-storage diskv`. The FileKV backend uses the diskv Go package internally, but the storage name is `filekv`.
- **Multi-storage gotcha**: When using multiple `-storage` flags, only the first backend is authoritative and returns values. All other backends are write-only fire-and-forget -- errors from secondary backends are logged but not returned.
- **Webhook timing**: Webhooks are asynchronous and may arrive out-of-order. Do not rely on webhook delivery order matching the order of MDM events.
- **Certificate validation order**: Certificate verification against the CA pool happens before the `certauth` service middleware runs. Invalid or untrusted certificates fail early at the HTTP handler level and never reach the service chain.
