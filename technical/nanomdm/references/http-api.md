# NanoMDM HTTP API & Endpoints

## Table of Contents

1. [API Authentication](#api-authentication)
2. [MDM Protocol Endpoints](#mdm-protocol-endpoints)
3. [API Endpoints](#api-endpoints)
4. [HTTP Handler Functions](#http-handler-functions)
5. [Webhook Event Format](#webhook-event-format)
6. [OpenAPI Spec](#openapi-spec)
7. [Request Adapter](#request-adapter)
8. [Service Request Dispatcher](#service-request-dispatcher)

## API Authentication

All `/v1/` API endpoints use HTTP Basic Authentication:
- Username: `nanomdm`
- Password: value of `-api` flag

If `-api` is not set, all API endpoints are disabled.

## MDM Protocol Endpoints

### PUT /mdm

Primary MDM endpoint. Corresponds to `ServerURL` in enrollment profile. Handles both check-in and command report requests by default.

Content-Type detection:
- `application/x-apple-aspen-mdm-checkin` -> Check-in handler
- Everything else -> Command report handler

If `-checkin` flag is set, `/mdm` only handles command reports.

### PUT /checkin

Separate check-in endpoint. Only active when `-checkin` flag is set. Corresponds to `CheckInURL` in enrollment profile.

### Certificate Extraction

Certificates are extracted via middleware in this priority:
1. **`-cert-header` flag**: Extract from HTTP header (RFC 9440 `:base64:` format or URL-escaped PEM)
2. **Default**: Verify `Mdm-Signature` header (PKCS7 detached signature)

After extraction, the certificate is verified against the CA pool (`-ca` flag).

## API Endpoints

### PUT /v1/pushcert

Upload APNs push certificate and private key.

**Request**: Raw PEM certificate + PEM private key concatenated (not JSON). Private key must be unencrypted.

```bash
cat push.pem push.key | curl -T - -u nanomdm:APIKEY 'http://host:9000/v1/pushcert'
```

**Response** (200):
```json
{
    "topic": "com.apple.mgmt.External.e3b8ceac-1f18-2c8e-8a63-dd17d99435d9",
    "not_after": "2026-01-07T04:04:46Z"
}
```

**Errors**: 400 (bad cert/key), 401 (auth), 500 (storage error)

### GET /v1/push/{id,...}

Send APNs push notifications to one or more enrollment IDs (comma-separated).

```bash
# Single push
curl -u nanomdm:APIKEY 'http://host:9000/v1/push/UUID1'

# Multi push
curl -u nanomdm:APIKEY 'http://host:9000/v1/push/UUID1,UUID2'
```

**Response** (200/207/500):
```json
{
    "status": {
        "UUID1": {
            "push_result": "8B16D295-AB2C-EAB9-90FF-8615C0DFBB08"
        },
        "UUID2": {
            "push_error": "push data missing for id"
        }
    }
}
```

HTTP status codes: 200 (all succeeded), 207 (partial), 500 (all failed).

### PUT /v1/enqueue/{id,...}[?nopush=1]

Enqueue a raw MDM command plist to one or more enrollment IDs.

**Request**: Raw XML plist command body.

```bash
# Enqueue with push
./cmdr.py SecurityInfo | curl -T - -u nanomdm:APIKEY 'http://host:9000/v1/enqueue/UUID1'

# Enqueue without push
./cmdr.py ProfileList | curl -T - -u nanomdm:APIKEY 'http://host:9000/v1/enqueue/UUID1?nopush=1'

# Multi-target
./cmdr.py DeviceInformation | curl -T - -u nanomdm:APIKEY 'http://host:9000/v1/enqueue/UUID1,UUID2'
```

**Response** (200/207/500):
```json
{
    "status": {
        "UUID1": {
            "push_result": "16C80450-B79F-E23B-F99B-0810179F244E"
        }
    },
    "command_uuid": "1ec2a267-1b32-4843-8ba0-2b06e80565c4",
    "request_type": "ProfileList"
}
```

With `?nopush=1`:
```json
{
    "no_push": true,
    "command_uuid": "598544b5-b681-4ce2-8914-ba7f45ff5c02",
    "request_type": "CertificateList"
}
```

### POST /v1/escrowkeyunlock

Activation Lock bypass via Apple's escrowKeyUnlock API. Uses APNs certificate for mTLS with Apple.

**Request**: `application/x-www-form-urlencoded` with parameters:

| Parameter | Required | In |
|-----------|----------|-----|
| topic | Yes | form |
| serial | Yes | form -> query |
| productType | Yes | form -> query |
| escrowKey | Yes | form -> body |
| orgName | Yes | form -> body |
| guid | Yes | form -> body |
| imei | Cellular only | form -> query |
| imei2 | Dual-SIM only | form -> query |
| meid | Cellular only | form -> query |

```bash
curl -u nanomdm:APIKEY \
    -d "topic=com.apple.mgmt.External.xxx" \
    -d "serial=C8TJ500QF1MN" \
    -d "productType=MacBookPro17,1" \
    -d "escrowKey=3UM43-PUYVY-QYD1-UVCC-HEHJ-FKA4" \
    -d "orgName=Acme Inc" \
    -d "guid=12346" \
    'http://host:9000/v1/escrowkeyunlock'
```

Response is proxied directly from Apple's endpoint.

### PUT /migration

Enrollment migration endpoint. Only active with `-migration` flag. Accepts raw Authenticate and TokenUpdate plists. Uses API authentication but bypasses certificate validation.

```bash
# Used by nano2nano tool
curl -T authenticate.plist -u nanomdm:APIKEY 'http://host:9000/migration'
curl -T tokenupdate.plist -u nanomdm:APIKEY 'http://host:9000/migration'
```

### GET /version

Returns server version. No authentication required.

```json
{"version": "v0.9.0"}
```

### /authproxy/

MDM-authenticated reverse proxy. Only active with `-auth-proxy-url` flag. Strips `/authproxy/` prefix and proxies to target URL. Authenticates using the same MDM certificate chain.

Headers forwarded to target: `X-Enrollment-ID`, `X-Trace-ID`.

Example: Request to `/authproxy/foo/bar` with `-auth-proxy-url http://backend:9008` proxies to `http://backend:9008/foo/bar`.

## HTTP Handler Functions

### MDM Handlers (http/mdm/)

```go
// Combined check-in + command handler (default)
httpmdm.CheckinAndCommandHandler(service, logger) http.HandlerFunc

// Separate handlers (when -checkin flag used)
httpmdm.CheckinHandler(service, logger) http.HandlerFunc
httpmdm.CommandAndReportResultsHandler(service, logger) http.HandlerFunc
```

### Certificate Middleware (http/mdm/)

```go
// Extract cert from Mdm-Signature header (PKCS7)
httpmdm.CertExtractMdmSignatureMiddleware(next, verifier, opts...)

// Extract cert from custom HTTP header
httpmdm.CertExtractPEMHeaderMiddleware(next, headerName, logger)

// Extract cert from TLS peer certificate
httpmdm.CertExtractTLSMiddleware(next, logger)

// Verify extracted cert against CA pool
httpmdm.CertVerifyMiddleware(next, verifier, logger)

// Look up enrollment ID from cert hash
httpmdm.CertWithEnrollmentIDMiddleware(next, hasher, store, enforce, logger)

// Get cert from context
httpmdm.GetCert(ctx) *x509.Certificate

// Get enrollment ID from context
httpmdm.GetEnrollmentID(ctx) string
```

### API Handlers (http/api/)

```go
// Register all v1 API handlers
httpapi.HandleAPIv1(prefix, mux, logger, store, pusher)

// Individual handlers
httpapi.PushHandler(pusher, logger)
httpapi.PushToIDsHandler(pusher, logger, idGetter)
httpapi.RawCommandEnqueueHandler(enqueuer, pusher, logger)
httpapi.RawCommandEnqueueToIDsHandler(enqueuer, pusher, logger, idGetter)
httpapi.NewStorePushCertHandler(storage, logger)
httpapi.NewEscrowKeyUnlockHandler(store, client, logger)

// ID extraction from URL path
httpapi.PathIDGetter(r *http.Request) ([]string, error)
```

## Webhook Event Format

JSON POST to configured webhook URL. Compatible with MicroMDM webhook format.

```json
{
    "topic": "mdm.TokenUpdate",
    "created_at": "2024-01-15T10:30:00Z",
    "event_id": "abc123",
    "checkin_event": {
        "ids": {
            "id": "UUID",
            "parent_id": null,
            "type": "Device"
        },
        "enrollment_id": null,
        "udid": "UUID",
        "raw_payload": "base64-encoded-plist",
        "token_update_tally": 1,
        "url_params": {}
    }
}
```

For command reports (`mdm.Connect`):
```json
{
    "topic": "mdm.Connect",
    "created_at": "2024-01-15T10:30:00Z",
    "acknowledge_event": {
        "ids": {"id": "UUID", "type": "Device"},
        "status": "Acknowledged",
        "command_uuid": "uuid-string",
        "raw_payload": "base64-encoded-plist",
        "url_params": {}
    }
}
```

HMAC: When `-webhook-hmac-key` is set, SHA-256 HMAC of the body is included in the `X-Hmac-Signature` header (Base-64 encoded).

## OpenAPI Spec

Full OpenAPI 3.0 specification is available at `docs/openapi.yaml` in the repository. Key schemas:

- `APIResult`: Push/enqueue response with per-enrollment status
- `PushCertResponse`: Push cert upload response with topic and expiry
- `ErrorResponse`: Simple error string wrapper

## Request Adapter

```go
// Convert HTTP request to MDM request
mdmReq := httpmdm.RequestFromHTTP(r)
// Sets context, certificate (from context), and URL query params
```

## Service Request Dispatcher

```go
// Dispatch check-in to appropriate service method
respBytes, err := service.CheckinRequest(svc, mdmReq, bodyBytes)

// Dispatch command report
respBytes, err := service.CommandAndReportResultsRequest(svc, mdmReq, bodyBytes)
```

These functions decode the plist body and call the appropriate service method based on `MessageType`.
