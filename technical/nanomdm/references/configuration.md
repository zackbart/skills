# NanoMDM Configuration & CLI Flags

## Table of Contents

1. [Environment Variables](#environment-variables)
2. [nanomdm Server Flags](#nanomdm-server-flags)
3. [Storage Backend Options](#storage-backend-options)
4. [nano2nano Migration Tool Flags](#nano2nano-migration-tool-flags)
5. [Docker Configuration](#docker-configuration)
6. [Build Configuration (Makefile)](#build-configuration-makefile)
7. [Enrollment Profile Template](#enrollment-profile-template)
8. [cmdr.py Command Generator](#cmdrpy-command-generator)
9. [Example Configurations](#example-configurations)

## Environment Variables

All flags can also be set via environment variables prefixed with `NANOMDM_`. Flags take precedence over env vars. Env vars listed in brackets below.

## nanomdm Server Flags

### Required

| Flag | Env Var | Description |
|------|---------|-------------|
| `-ca <path>` | `NANOMDM_CA` | Path to PEM CA cert(s) for device identity validation. **Required.** |

### API & Endpoints

| Flag | Env Var | Default | Description |
|------|---------|---------|-------------|
| `-api <key>` | `NANOMDM_API` | (empty) | API key for API endpoints. HTTP Basic auth user=`nanomdm`. Omit to disable all API. |
| `-listen <addr>` | `NANOMDM_LISTEN` | `:9000` | HTTP listen address |
| `-disable-mdm` | `NANOMDM_DISABLE_MDM` | false | Disable MDM endpoint (API-only mode) |
| `-checkin` | `NANOMDM_CHECKIN` | false | Enable separate `/checkin` endpoint |
| `-migration` | `NANOMDM_MIGRATION` | false | Enable `/migration` endpoint |

### Certificate Handling

| Flag | Env Var | Description |
|------|---------|-------------|
| `-ca <path>` | `NANOMDM_CA` | Path to PEM CA cert(s) |
| `-intermediate <path>` | `NANOMDM_INTERMEDIATE` | Path to PEM intermediate cert(s) |
| `-cert-header <name>` | `NANOMDM_CERT_HEADER` | HTTP header for client cert (RFC 9440 or URL-escaped PEM). Overrides Mdm-Signature. |

### Storage

| Flag | Env Var | Default | Description |
|------|---------|---------|-------------|
| `-storage <name>` | `NANOMDM_STORAGE` | `filekv` | Storage backend name |
| `-storage-dsn <dsn>` | `NANOMDM_STORAGE_DSN` | `dbkv` | Data source name (connection string or path) |
| `-storage-options <opts>` | `NANOMDM_STORAGE_OPTIONS` | (empty) | Comma-separated key=value options |

Storage backend names: `filekv`, `mysql`, `pgsql`, `inmem`, `file`, `diskv`

Multiple backends: specify multiple `-storage`/`-storage-dsn`/`-storage-options` flag sets.

### Webhook

| Flag | Env Var | Description |
|------|---------|-------------|
| `-webhook-url <url>` | `NANOMDM_WEBHOOK_URL` | URL for MDM event webhooks |
| `-webhook-hmac-key <key>` | `NANOMDM_WEBHOOK_HMAC_KEY` | SHA-256 HMAC key for webhook body signing |

### Declarative Management

| Flag | Env Var | Description |
|------|---------|-------------|
| `-dm <url>` | `NANOMDM_DM` | Base URL for DM requests (should have trailing slash) |
| `-dm-send-hmac-key <key>` | `NANOMDM_DM_SEND_HMAC_KEY` | HMAC key for outgoing DM requests |
| `-dm-recv-hmac-key <key>` | `NANOMDM_DM_RECV_HMAC_KEY` | HMAC key for verifying DM responses |

### Authentication Proxy

| Flag | Env Var | Description |
|------|---------|-------------|
| `-auth-proxy-url <url>` | `NANOMDM_AUTH_PROXY_URL` | Reverse proxy target for MDM-authenticated requests at `/authproxy/` |

### User Authentication

| Flag | Env Var | Description |
|------|---------|-------------|
| `-ua-zl-dc` | `NANOMDM_UA_ZL_DC` | Reply with zero-length DigestChallenge for UserAuthenticate (enables user channel management) |

### Enrollment Migration

| Flag | Env Var | Description |
|------|---------|-------------|
| `-retro` | `NANOMDM_RETRO` | Allow retroactive certificate-auth association for migrated enrollments |

### Debugging

| Flag | Env Var | Description |
|------|---------|-------------|
| `-debug` | `NANOMDM_DEBUG` | Enable debug logging |
| `-dump` | `NANOMDM_DUMP` | Dump raw MDM request/response plists to stdout |

### Other

| Flag | Description |
|------|-------------|
| `-version` | Print version and exit |

## Storage Backend Options

### MySQL Options

```
-storage mysql -storage-dsn user:pass/dbname -storage-options delete=1
```

- `delete=1`: Enable command/response cleanup after device acknowledges. Saves disk space.

DSN format: [go-sql-driver format](https://github.com/go-sql-driver/mysql#dsn-data-source-name) e.g., `user:password@tcp(host:3306)/dbname`

### PostgreSQL Options

```
-storage pgsql -storage-dsn postgres://user:pass@host:5432/dbname -storage-options delete=1
```

- `delete=1`: Same as MySQL -- cleanup after acknowledge.

DSN format: [lib/pq format](https://pkg.go.dev/github.com/lib/pq#pkg-overview) e.g., `postgres://user:pass@host/dbname?sslmode=disable`

### File (deprecated) Options

```
-storage file -storage-dsn /path/to/db -storage-options enable_deprecated=1
```

- `enable_deprecated=1`: Required to enable this deprecated backend.

### FileKV Options

```
-storage filekv -storage-dsn /path/to/db
```

No additional options. Default DSN is `dbkv`.

### In-Memory Options

```
-storage inmem
```

No DSN or options. All data lost on process exit.

### Multi-Storage

```
-storage filekv -storage-dsn dbkv -storage mysql -storage-dsn user:pass/dbname
```

Flags are paired by order. Empty options must still be specified for backends without options. First backend is authoritative.

## nano2nano Migration Tool Flags

| Flag | Description |
|------|-------------|
| `-storage <name>` | Source storage backend |
| `-storage-dsn <dsn>` | Source DSN |
| `-storage-options <opts>` | Source storage options |
| `-url <url>` | NanoMDM migration endpoint URL (e.g., `http://host:9000/migration`) |
| `-key <key>` | NanoMDM API key |
| `-debug` | Debug logging |
| `-version` | Print version |

Example:
```bash
nano2nano -storage file -storage-dsn db -url 'http://host:9000/migration' -key APIKEY -debug
```

Migration sends Authenticate and TokenUpdate messages to the target NanoMDM instance. Order matters: Authenticate must precede TokenUpdate, device channel must precede user channel.

## Docker Configuration

```dockerfile
FROM gcr.io/distroless/static
EXPOSE 9000
VOLUME ["/app/dbkv", "/app/db"]
ENTRYPOINT ["/app/nanomdm"]
```

```bash
docker pull ghcr.io/micromdm/nanomdm:latest
docker run -v /host/db:/app/dbkv ghcr.io/micromdm/nanomdm:latest \
    -ca /path/to/ca.pem -api APIKEY -storage filekv -storage-dsn /app/dbkv
```

## Build Configuration (Makefile)

```bash
make              # Build for current OS/arch
make release      # Cross-compile all platforms (darwin/linux amd64/arm64, windows)
make test         # Run tests with race detector
```

Version is injected via `-ldflags "-X main.version=$(VERSION)"` from `git describe --tags --always --dirty`.

Cross-compile targets: `darwin-amd64`, `darwin-arm64`, `linux-amd64`, `linux-arm64`, `linux-arm`, `windows-amd64`.

## Enrollment Profile Template

The repo provides `docs/enroll.mobileconfig` as a reference. Key fields:

```xml
<!-- SCEP payload — generates device identity certificate -->
<dict>
    <key>PayloadContent</key>
    <dict>
        <key>Key Type</key><string>RSA</string>
        <key>Challenge</key><string>SCEP-CHALLENGE-HERE</string>
        <key>Key Usage</key><integer>5</integer>
        <key>Keysize</key><integer>2048</integer>
        <key>URL</key><string>https://mdm.example.org/scep</string>
    </dict>
    <key>PayloadType</key><string>com.apple.security.scep</string>
    <key>PayloadUUID</key><string>CB90E976-AD44-4B69-8108-8095E6260978</string>
</dict>

<!-- MDM payload -->
<dict>
    <key>AccessRights</key><integer>8191</integer>  <!-- All rights -->
    <key>CheckOutWhenRemoved</key><true/>
    <key>IdentityCertificateUUID</key>
    <string>CB90E976-AD44-4B69-8108-8095E6260978</string>  <!-- Links to SCEP PayloadUUID -->
    <key>ServerURL</key><string>https://mdm.example.org/mdm</string>
    <key>SignMessage</key><true/>  <!-- Enables Mdm-Signature header -->
    <key>Topic</key><string>com.apple.mgmt.External.YOUR-PUSH-TOPIC-HERE</string>
    <key>ServerCapabilities</key>
    <array>
        <string>com.apple.mdm.per-user-connections</string>  <!-- User channel support -->
        <string>com.apple.mdm.bootstraptoken</string>        <!-- Bootstrap token escrow -->
        <string>com.apple.mdm.token</string>                 <!-- GetToken (watch pairing) -->
    </array>
</dict>
```

**AccessRights values** (bitfield, 8191 = all):

| Bit | Value | Right |
|-----|-------|-------|
| 0 | 1 | Allow inspection of installed profiles |
| 1 | 2 | Allow installation/removal of profiles |
| 2 | 4 | Allow device lock and passcode removal |
| 3 | 8 | Allow device erase |
| 4 | 16 | Allow query of device information |
| 5 | 32 | Allow query of network information |
| 6 | 64 | Allow inspection of installed provisioning profiles |
| 7 | 128 | Allow installation/removal of provisioning profiles |
| 8 | 256 | Allow inspection of installed applications |
| 9 | 512 | Allow restriction-related queries/commands |
| 10 | 1024 | Allow security-related queries/commands |
| 11 | 2048 | Allow manipulation of settings |
| 12 | 4096 | Allow app management |

## cmdr.py Command Generator

Located at `tools/cmdr.py`. Generates raw plist MDM commands with auto-generated CommandUUID.

### Simple commands (no arguments)

```bash
./cmdr.py ProfileList              ./cmdr.py RestartDevice
./cmdr.py ProvisioningProfileList  ./cmdr.py ShutDownDevice
./cmdr.py SecurityInfo             ./cmdr.py StopMirroring
./cmdr.py ClearRestrictionsPassword ./cmdr.py UserList
./cmdr.py LogOutUser               ./cmdr.py PlayLostModeSound
./cmdr.py DisableLostMode          ./cmdr.py DeviceLocation
./cmdr.py ManagedMediaList         ./cmdr.py DeviceConfigured
./cmdr.py AvailableOSUpdates       ./cmdr.py NSExtensionMappings
./cmdr.py OSUpdateStatus           ./cmdr.py EnableRemoteDesktop
./cmdr.py DisableRemoteDesktop     ./cmdr.py ActivationLockBypassCode
./cmdr.py ScheduleOSUpdateScan
```

### Commands with arguments

```bash
# DeviceInformation with optional queries
./cmdr.py DeviceInformation Model OSVersion SerialNumber

# Install/Remove profiles
./cmdr.py InstallProfile /path/to/profile.mobileconfig
./cmdr.py RemoveProfile com.example.profile.identifier

# OS updates
./cmdr.py ScheduleOSUpdate InstallASAP --version 15.0 --priority High
./cmdr.py ScheduleOSUpdate InstallLater --deferrals 3

# Account configuration (macOS)
./cmdr.py AccountConfig -f "John Doe" -u "jdoe" --lock

# Settings (bootstrap token)
./cmdr.py Settings --allowbst

# Certificate list
./cmdr.py CertificateList --managedonly

# Erase/Lock (CAUTION: destructive)
./cmdr.py EraseDevice 123456
./cmdr.py DeviceLock 123456

# Arbitrary command by RequestType
./cmdr.py command InstalledApplicationList

# Random read-only command (for testing)
./cmdr.py -r
```

### Flags

| Flag | Description |
|------|-------------|
| `-u UUID` | Custom CommandUUID (default: auto-generated) |
| `-r` | Random read-only command (SecurityInfo, CertificateList, ProfileList, ProvisioningProfileList) |

## Example Configurations

### Minimal (development)

```bash
nanomdm -ca ca.pem -api nanomdm -debug
```

### Production with MySQL

```bash
nanomdm \
    -ca /etc/nanomdm/ca.pem \
    -intermediate /etc/nanomdm/intermediate.pem \
    -api "$API_KEY" \
    -storage mysql \
    -storage-dsn "nanomdm:$DB_PASS@tcp(db.example.com:3306)/nanomdm" \
    -storage-options delete=1 \
    -webhook-url https://mdm-backend.example.com/webhook \
    -webhook-hmac-key "$WEBHOOK_SECRET" \
    -cert-header X-Client-Cert \
    -listen :9000
```

### With Declarative Management

```bash
nanomdm \
    -ca ca.pem \
    -api nanomdm \
    -dm https://ddm-server.example.com/ \
    -dm-send-hmac-key "$DM_HMAC_KEY" \
    -dm-recv-hmac-key "$DM_HMAC_KEY"
```

### API-Only Mode

```bash
nanomdm \
    -ca ca.pem \
    -api nanomdm \
    -disable-mdm \
    -storage mysql \
    -storage-dsn "nanomdm:pass@/nanomdm"
```
