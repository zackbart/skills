# NanoMDM Architecture & Service Interfaces

## Table of Contents

1. [Package Structure](#package-structure)
2. [Layered Architecture](#layered-architecture)
3. [Core Service Interfaces](#core-service-interfaces)
4. [Core NanoMDM Service (service/nanomdm/)](#core-nanomdm-service-servicenanomdm)
5. [Service Middleware](#service-middleware)
6. [Push System](#push-system)
7. [Certificate Verification (certverify/)](#certificate-verification-certverify)
8. [API Layer (api/)](#api-layer-api)
9. [Crypto Utilities (cryptoutil/)](#crypto-utilities-cryptoutil)
10. [Middleware Assembly in cmd/nanomdm](#middleware-assembly-in-cmdnanomdm)

## Package Structure

```
github.com/micromdm/nanomdm/
  mdm/              # MDM protocol types (Enrollment, Request, check-in messages, commands)
  service/          # Service interfaces and middleware
    nanomdm/        # Core service implementation (dispatches to storage)
    certauth/       # Certificate authentication middleware
    webhook/        # HTTP webhook callback service
    dump/           # Debug request dumper middleware
    dmhook/         # Declarative Management HTTP proxy
    multi/          # Multi-service dispatcher
  storage/          # Storage interfaces
    mysql/          # MySQL backend
    pgsql/          # PostgreSQL backend
    file/           # File-based backend (deprecated)
    kv/             # Key-value based backend (shared logic)
    inmem/          # In-memory backend (uses kv)
    diskv/          # Disk-based KV backend (uses kv + diskv)
    allmulti/        # Multi-storage dispatcher
  api/              # API types and push+enqueue logic
  push/             # APNs push interfaces and implementations
    nanopush/       # Native HTTP/2 APNs push provider
    buford/         # Buford-based APNs push provider
    service/        # Push service (retrieves certs, sends pushes)
  http/             # HTTP handlers
    mdm/            # MDM protocol HTTP handlers
    api/            # API HTTP handlers (v1)
    authproxy/      # MDM-authenticated reverse proxy
    hashbody/       # HMAC body hashing utilities
    escrowkeyunlock/ # Activation Lock bypass
  certverify/       # Certificate verification (pool, signature, fallback)
  cryptoutil/       # Crypto helpers (PEM, PKCS7, topic extraction)
  cmd/nanomdm/      # Main server binary
  cmd/nano2nano/    # Migration tool binary
  cli/              # CLI storage flag parsing
```

## Layered Architecture

```
                    HTTP Layer
                    ----------
                    http/mdm/       (MDM protocol handlers)
                    http/api/       (API handlers)
                    http/authproxy/ (reverse proxy)
                         |
                    Service Layer
                    -------------
                    service.CheckinAndCommandService interface
                         |
              +----------+---------+----------+
              |          |         |          |
            dump     certauth   multi     webhook
              |          |      /    \
              +----------+   nanomdm  webhook
                             |
                    Storage Layer
                    -------------
                    storage.ServiceStore interface
                    storage.CommandEnqueuer
                    storage.PushStore
                    storage.PushCertStore
                    storage.CertAuthStore
                         |
              +------+-------+------+------+
              |      |       |      |      |
            filekv  mysql   pgsql  inmem  diskv
```

## Core Service Interfaces

### service/service.go

```go
// Checkin represents the various check-in requests.
type Checkin interface {
    Authenticate(*mdm.Request, *mdm.Authenticate) error
    TokenUpdate(*mdm.Request, *mdm.TokenUpdate) error
    CheckOut(*mdm.Request, *mdm.CheckOut) error
    SetBootstrapToken(*mdm.Request, *mdm.SetBootstrapToken) error
    GetBootstrapToken(*mdm.Request, *mdm.GetBootstrapToken) (*mdm.BootstrapToken, error)
    UserAuthenticate
    DeclarativeManagement
    GetToken
}

type UserAuthenticate interface {
    UserAuthenticate(*mdm.Request, *mdm.UserAuthenticate) ([]byte, error)
}

type DeclarativeManagement interface {
    DeclarativeManagement(*mdm.Request, *mdm.DeclarativeManagement) ([]byte, error)
}

type GetToken interface {
    GetToken(*mdm.Request, *mdm.GetToken) (*mdm.GetTokenResponse, error)
}

type CommandAndReportResults interface {
    CommandAndReportResults(*mdm.Request, *mdm.CommandResults) (*mdm.Command, error)
}

// CheckinAndCommandService is the primary composable interface.
type CheckinAndCommandService interface {
    Checkin
    CommandAndReportResults
}
```

### NopService

`service.NopService` implements all `CheckinAndCommandService` methods as no-ops. Useful as a base for partial implementations.

## Core NanoMDM Service (service/nanomdm/)

The main service normalizes enrollment IDs and dispatches to storage:

```go
type Service struct {
    logger     log.Logger
    normalizer func(e *mdm.Enrollment) *mdm.EnrollID
    store      storage.ServiceStore
    dm         service.DeclarativeManagement  // optional DM handler
    ua         service.UserAuthenticate       // optional UA handler
    gt         service.GetToken               // optional GetToken handler
}

func New(store storage.ServiceStore, opts ...Option) *Service

// Options:
func WithLogger(logger log.Logger) Option
func WithDeclarativeManagement(dm service.DeclarativeManagement) Option
func WithUserAuthenticate(ua service.UserAuthenticate) Option
func WithGetToken(gt service.GetToken) Option
```

### Enrollment ID Normalization

The `normalize` function converts raw enrollment data to a canonical ID:

```go
func normalize(e *mdm.Enrollment) *mdm.EnrollID {
    r := e.Resolved()
    eid := &mdm.EnrollID{Type: r.Type, ID: r.DeviceChannelID}
    if r.IsUserChannel {
        eid.ID += ":" + r.UserChannelID
        eid.ParentID = r.DeviceChannelID
    }
    return eid
}
```

| Type | Platform | ID Format | Example |
|------|----------|-----------|---------|
| Device | macOS | `UUID` | `470E005B-17C1-4537-BBB3-0EBC340D432A` |
| User | macOS | `UUID:UUID` | `470E005B-...:F151140B-...` |
| Device | iOS | `UUID` | `8b3b8ba3783e9ade1dae4fbb944ab3afc0ce5b69` |
| UserEnrollment | iOS | `UUID` | `b318edb72b556059a013368e3150050c5f74a2c6` |
| SharediPad | iOS | `UUID:ShortName` | `68656c6c6f...:appleid@example.com` |

### Authenticate Flow

1. Setup request (normalize enrollment ID)
2. `store.StoreAuthenticate(r, message)` -- persist the authenticate message
3. `store.ClearQueue(r)` -- clear any queued commands (prevents stale commands after re-enrollment)
4. `store.Disable(r)` -- disable enrollment until TokenUpdate confirms it

### TokenUpdate Flow

1. Setup request
2. `store.StoreTokenUpdate(r, message)` -- stores push token, topic, push magic; enables enrollment

### CommandAndReportResults Flow

1. Setup request
2. `store.StoreCommandReport(r, results)` -- store the command result
3. `store.RetrieveNextCommand(r, skipNotNow)` -- get next queued command
4. Return command (or nil if queue empty)

## Service Middleware

### CertAuth (service/certauth/)

Certificate authentication middleware. Wraps another `CheckinAndCommandService`.

```go
func New(next CheckinAndCommandService, storage CertAuthStore, opts ...Option) *CertAuth

// Options:
func WithLogger(logger log.Logger) Option
func WithAllowRetroactive() Option  // allow migrated enrollments to fix associations
```

**Behavior:**
- On `Authenticate`: associates the device certificate hash with the enrollment (`associateNewEnrollment`)
- On all other messages: validates the certificate is associated with the enrollment (`validateAssociateExistingEnrollment`)
- Prevents certificate reuse across different enrollments (anti-spoofing)
- `HashCert(cert)` produces SHA-256 hex string of raw cert bytes

### Multi Service (service/multi/)

Dispatches to multiple services. First service is authoritative (returns values). Others run in goroutines.

```go
func New(logger log.Logger, svcs ...CheckinAndCommandService) *MultiService
```

Uses `ContextWithoutCancel` to prevent goroutine cancellation when the primary request context is done.

### Webhook (service/webhook/)

Sends JSON HTTP POST callbacks for MDM events.

```go
func New(url string, opts ...Option) *Webhook

// Options:
func WithTokenUpdateTalley(store storage.TokenUpdateTallyStore) Option
func WithClient(doer Doer) Option
func WithEventID(fn func(context.Context) string) Option
func WithHMACSecret(key []byte) Option  // SHA-256 HMAC in X-Hmac-Signature header
```

### Dump (service/dump/)

Debug middleware that writes raw request plists to an `io.Writer` (typically stdout).

```go
func New(next CheckinAndCommandService, w DumpWriter) *Dumper
```

### DMHook (service/dmhook/)

Proxies Declarative Management requests to an external HTTP endpoint.

```go
func New(urlPrefix string, opts ...Option) (*DMHook, error)

// Options:
func WithClient(client Doer) Option
func WithSetHMACSecret(key []byte) Option    // HMAC outgoing requests
func WithVerifyHMACSecret(key []byte) Option // Verify HMAC on responses
```

Headers set on proxy requests: `X-Enrollment-ID`, `X-Enrollment-Type`, `X-Enrollment-ParentID` (if non-empty).

### TokenMux (service/nanomdm/token.go)

Multiplexer for GetToken check-in messages by `TokenServiceType`:

```go
tokenMux := nanomdm.NewTokenMux()
tokenMux.Handle("com.apple.watch.pairing", handler)
```

### UAService (service/nanomdm/ua.go)

UserAuthenticate handler. By default returns HTTP 410 (declining user management). With `sendEmptyDigestChallenge=true`, implements the zero-length digest challenge protocol.

```go
func NewUAService(store storage.UserAuthenticateStore, sendEmptyDigestChallenge bool) *UAService
```

## Push System

### Interfaces (push/push.go)

```go
type Pusher interface {
    Push(context.Context, []string) (map[string]*Response, error)
}

type PushProvider interface {
    Push(context.Context, []*mdm.Push) (map[string]*Response, error)
}

type PushProviderFactory interface {
    NewPushProvider(*tls.Certificate) (PushProvider, error)
}
```

### PushService (push/service/)

The main push service. Retrieves push info from storage, gets/caches push providers per topic, and sends notifications.

```go
func New(store PushStore, certStore PushCertStore, providerFactory PushProviderFactory, logger log.Logger) *PushService
```

- Caches push providers per APNs topic
- Detects stale certificates and re-creates providers
- Supports multi-topic (multi-tenant) push
- Maps between enrollment IDs and push tokens

### NanoPush Provider (push/nanopush/)

Native HTTP/2 APNs push implementation.

```go
factory := nanopush.NewFactory(
    nanopush.WithExpiration(24 * time.Hour),
    nanopush.WithWorkers(5),
    nanopush.WithNewClient(customClientFn),
)
```

APNs endpoints: `https://api.push.apple.com` (production), `https://api.development.push.apple.com` (development).

Push payload: `{"mdm":"<PushMagic>"}` sent to `/3/device/<token>`.

### Buford Provider (push/buford/)

Alternative APNs provider using the `github.com/RobotsAndPencils/buford` library.

```go
factory := buford.NewPushProviderFactory(
    buford.WithWorkers(5),
    buford.WithExpiration(24 * time.Hour),
)
```

## Certificate Verification (certverify/)

Three verifier types:

```go
// PoolVerifier - Full X509 chain verification
verifier, _ := certverify.NewPoolVerifier(caPEM, intsPEM, x509.ExtKeyUsageClientAuth)

// SignatureVerifier - Signature-only check against single CA
verifier, _ := certverify.NewSignatureVerifier(rootPEM)

// FallbackVerifier - Try multiple verifiers, pass if any succeeds
verifier := certverify.NewFallbackVerifier(verifier1, verifier2)
```

## API Layer (api/)

### PushEnqueuer

Combined push and enqueue operations:

```go
pe, _ := api.NewPushEnqueuer(store, pusher, api.WithLogger(logger))
result, httpCode, err := pe.Push(ctx, ids)
result, httpCode, err := pe.EnqueueWithPush(ctx, command, ids, noPush)
result, httpCode, err := pe.RawCommandEnqueueWithPush(ctx, rawPlist, ids, noPush)
```

HTTP status codes: 200 (all success), 207 (partial), 500 (all failed).

### APIResult

```go
type APIResult struct {
    Status       map[string]EnrollmentResult `json:"status,omitempty"`
    NoPush       bool                        `json:"no_push,omitempty"`
    PushError    *Error                      `json:"push_error,omitempty"`
    EnqueueError *Error                      `json:"command_error,omitempty"`
    CommandUUID  string                      `json:"command_uuid,omitempty"`
    RequestType  string                      `json:"request_type,omitempty"`
}

type EnrollmentResult struct {
    PushError    *Error `json:"push_error,omitempty"`
    PushID       string `json:"push_result,omitempty"`
    EnqueueError *Error `json:"command_error,omitempty"`
}
```

## Crypto Utilities (cryptoutil/)

```go
// Extract APNs topic from certificate (UserID OID)
topic, err := cryptoutil.TopicFromCert(cert)

// Verify Mdm-Signature header (PKCS7 detached signature)
cert, err := cryptoutil.VerifyMdmSignature(headerBase64, body)

// PEM encode/decode
pemBytes := cryptoutil.PEMCertificate(derBytes)
cert, err := cryptoutil.DecodePEMCertificate(pemData)
```

## Middleware Assembly in cmd/nanomdm

The main binary assembles the full middleware chain:

```go
// 1. Create core service
nano := nanomdm.New(mdmStorage, nanoOpts...)

// 2. Wrap with webhook (via multi service)
webhookService := webhook.New(webhookURL, whOpts...)
mdmService = multi.New(logger, mdmService, webhookService)

// 3. Wrap with cert auth
mdmService = certauth.New(mdmService, mdmStorage, certAuthOpts...)

// 4. Wrap with dump (optional)
mdmService = dump.New(mdmService, os.Stdout)

// 5. Register HTTP handlers with cert extraction middleware
mux.Handle("/mdm", httpmdm.CheckinAndCommandHandler(mdmService, logger))
```
