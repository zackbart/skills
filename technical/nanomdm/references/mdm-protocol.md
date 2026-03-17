# NanoMDM MDM Protocol Types & Check-in Handling

## Core Types (mdm/)

### Enrollment

```go
type Enrollment struct {
    UDID             string `plist:",omitempty"`
    UserID           string `plist:",omitempty"`
    UserShortName    string `plist:",omitempty"`
    UserLongName     string `plist:",omitempty"`
    EnrollmentID     string `plist:",omitempty"`  // User Enrollment (BYOD)
    EnrollmentUserID string `plist:",omitempty"`  // User Enrollment user channel
}
```

### EnrollID (Normalized)

```go
type EnrollID struct {
    Type     EnrollType  // Device, User, UserEnrollmentDevice, UserEnrollment, SharediPad
    ID       string      // Normalized enrollment ID
    ParentID string      // Device channel ID (for user channels)
}
```

### EnrollType

```go
const (
    Device              = 1 + iota  // Standard device enrollment (macOS UDID, iOS UDID)
    User                            // macOS user channel (UDID:UserID)
    UserEnrollmentDevice            // iOS BYOD device channel (EnrollmentID)
    UserEnrollment                  // iOS BYOD user channel (EnrollmentID:EnrollmentUserID)
    SharediPad                      // Shared iPad (UDID:UserShortName/AppleID)
)
```

### Request

```go
type Request struct {
    *EnrollID                      // Populated by core service (normalized IDs)
    Certificate *x509.Certificate  // Device identity certificate
    Params      map[string]string  // URL query parameters
    ctx         context.Context
}

func NewRequestWithContext(ctx context.Context, cert *x509.Certificate) *Request
func (r *Request) Context() context.Context
func (r *Request) WithContext(ctx context.Context) *Request
```

### Push

```go
type Push struct {
    PushMagic string   // Magic string for APNs payload
    Token     hexData  // APNs device token (hex-encoded bytes)
    Topic     string   // APNs topic (from push certificate)
}
```

## Check-in Message Types

### Authenticate

First message sent by device during enrollment. Indicates a new or re-enrollment.

```go
type Authenticate struct {
    Enrollment
    MessageType
    Topic        string
    SerialNumber string `plist:",omitempty"`
    Raw          []byte `plist:"-"`
}
```

**Server behavior**: Store authenticate data, clear command queue, disable enrollment (until TokenUpdate).

### TokenUpdate

Sent after Authenticate to complete enrollment. Contains push notification credentials.

```go
type TokenUpdate struct {
    Enrollment
    MessageType
    Push                        // PushMagic, Token, Topic
    UnlockToken []byte `plist:",omitempty"`  // iOS device unlock token
    Raw         []byte `plist:"-"`
}
```

**Server behavior**: Store push info, enable enrollment, increment tally. For user channels, associate with parent device.

### CheckOut

Sent when device unenrolls from MDM.

```go
type CheckOut struct {
    Enrollment
    MessageType
    Raw []byte `plist:"-"`
}
```

**Server behavior**: Disable enrollment.

### UserAuthenticate

Sent for directory MDM user management. Two-phase protocol.

```go
type UserAuthenticate struct {
    Enrollment
    MessageType
    DigestResponse string `plist:",omitempty"`  // Empty on first message
    Raw            []byte `plist:"-"`
}
```

**Server behavior**: Default returns HTTP 410 (decline). With `-ua-zl-dc`, sends empty DigestChallenge on first message, empty response on second.

### SetBootstrapToken

Device escrows a Bootstrap Token.

```go
type SetBootstrapToken struct {
    Enrollment
    MessageType
    BootstrapToken  // Contains base64 token data
    Raw []byte `plist:"-"`
}
```

### GetBootstrapToken

Device requests previously escrowed Bootstrap Token.

```go
type GetBootstrapToken struct {
    Enrollment
    MessageType
    Raw []byte `plist:"-"`
}

type BootstrapToken struct {
    BootstrapToken b64Data  // Base64-encoded token bytes
}
```

### DeclarativeManagement

Declarative Device Management protocol messages.

```go
type DeclarativeManagement struct {
    Enrollment
    MessageType
    Data     []byte   // Request body (e.g., status report JSON)
    Endpoint string   // DM endpoint type (e.g., "status", "declaration-items")
    Raw      []byte `plist:"-"`
}
```

**Server behavior**: Proxied to external DM service via `-dm` URL flag.

### GetToken

Token exchange for watch pairing and other services.

```go
type GetToken struct {
    Enrollment
    MessageType
    TokenServiceType string             // e.g., "com.apple.watch.pairing"
    TokenParameters  *TokenParameters `plist:",omitempty"`
    Raw              []byte `plist:"-"`
}

type TokenParameters struct {
    PhoneUDID     string
    SecurityToken string
    WatchUDID     string
}

type GetTokenResponse struct {
    TokenData []byte
}
```

**Server behavior**: Dispatched via TokenMux to registered handlers by TokenServiceType.

## Command Types

### Command

```go
type Command struct {
    CommandUUID string
    Command     struct {
        RequestType string
    }
    Raw []byte `plist:"-"`  // Original XML plist
}

func DecodeCommand(rawCommand []byte) (*Command, error)
```

### CommandResults

```go
type CommandResults struct {
    Enrollment
    CommandUUID string `plist:",omitempty"`
    Status      string  // "Idle", "Acknowledged", "Error", "CommandFormatError", "NotNow"
    ErrorChain  []ErrorChain `plist:",omitempty"`
    Raw         []byte `plist:"-"`
}

type ErrorChain struct {
    ErrorCode            int
    ErrorDomain          string
    LocalizedDescription string
    USEnglishDescription string
}

func DecodeCommandResults(rawResults []byte) (*CommandResults, error)
```

### Command Status Values

| Status | Meaning |
|--------|---------|
| `Idle` | Device checking in with no pending response. Queue next command. |
| `Acknowledged` | Command executed successfully. |
| `Error` | Command failed. Check ErrorChain. |
| `CommandFormatError` | Command plist malformed. |
| `NotNow` | Device busy, will retry later. Command stays in queue. |

## Check-in Decoding

```go
// DecodeCheckin automatically detects MessageType and returns typed struct
msg, err := mdm.DecodeCheckin(rawPlistBytes)
// msg is one of: *Authenticate, *TokenUpdate, *CheckOut, *UserAuthenticate,
//   *SetBootstrapToken, *GetBootstrapToken, *DeclarativeManagement, *GetToken
```

## Enrollment Resolution

`Enrollment.Resolved()` determines the enrollment type and channel:

```go
func (e *Enrollment) Resolved() *ResolvedEnrollment {
    // Priority: UDID -> EnrollmentID
    // If UDID set:
    //   - No UserID: Device channel
    //   - UserID == FFFFFFFF-...: Shared iPad (uses UserShortName)
    //   - UserID set: User channel
    // If EnrollmentID set (no UDID):
    //   - No EnrollmentUserID: UserEnrollmentDevice
    //   - EnrollmentUserID set: UserEnrollment
}
```

Shared iPad constant: `SharediPadUserID = "FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF"`

## Command Queue Flow

```
1. API receives command plist
   -> mdm.DecodeCommand(rawPlist) validates CommandUUID and RequestType
   -> storage.EnqueueCommand(ctx, ids, cmd) stores command + queue entries

2. APNs push sent to device
   -> push.Pusher.Push(ctx, ids) sends {"mdm":"<PushMagic>"} to APNs

3. Device connects to /mdm
   -> Reports previous command result (or Idle)
   -> storage.StoreCommandReport(r, results) stores result
   -> storage.RetrieveNextCommand(r, skipNotNow) gets next queued command
   -> Returns command plist (or empty response if queue empty)

4. Device processes command and connects again
   -> Reports result with Status (Acknowledged/Error/NotNow)
   -> Cycle continues until queue is empty (Idle)
```

### NotNow Handling

When a device responds with `NotNow`:
- The command stays in the queue
- On next check-in, `skipNotNow` parameter determines whether to skip NotNow commands
- `skipNotNow` is true when the current status report is `NotNow` (prevents re-sending the same command)

## Plist Command Format

MDM commands are raw Apple Plist XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Command</key>
    <dict>
        <key>RequestType</key>
        <string>SecurityInfo</string>
    </dict>
    <key>CommandUUID</key>
    <string>d1b7fdda-52e1-45d1-80e6-8bf3b5d76f17</string>
</dict>
</plist>
```

Use the `tools/cmdr.py` script to generate commands:
```bash
./cmdr.py SecurityInfo          # specific command
./cmdr.py -r                    # random read-only command
./cmdr.py DeviceInformation     # device info query
```

## Error Types

```go
var ErrUnrecognizedMessageType = errors.New("unrecognized MessageType")
var ErrInvalidCommandResult = errors.New("invalid command result")
var ErrInvalidCommand = errors.New("invalid command")
var ErrEmptyCommand = errors.New("empty command bytes")

type ParseError struct {
    Err     error
    Content []byte  // Raw content that failed to parse
}

type HTTPStatusError struct {
    Status int
    Err    error
}
```

## Migration

The `nano2nano` tool extracts Authenticate, TokenUpdate, and SetBootstrapToken messages from a source storage backend and sends them to a NanoMDM migration endpoint.

Key constraints:
- **Lossy**: Only transfers minimum data for push notifications and command queue
- **Order matters**: Authenticate before TokenUpdate, device before user channel
- **Same ServerURL required**: Source and target MDM must have identical ServerURL
- **iOS unlock tokens**: May be lost if not in latest TokenUpdate
- Use `-retro` flag on target to allow retroactive cert associations
