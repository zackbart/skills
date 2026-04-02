# iOS MDM Protocol Reference

## Table of Contents

- [Enrollment Flow](#enrollment-flow)
- [Check-in Protocol](#check-in-protocol)
- [Command-Response Cycle](#command-response-cycle)
- [APNs Push Mechanism](#apns-push-mechanism)
- [MDM Enrollment Payload](#mdm-enrollment-payload-comapplemdm)
- [Enrollment Types](#enrollment-types)

## Enrollment Flow

### 1. Automated Device Enrollment (ADE)

Devices registered in Apple Business Manager (or Apple School Manager) enroll automatically during Setup Assistant. The flow:

1. Device activates and contacts Apple's device enrollment service.
2. Apple returns the enrollment profile URL for the assigned MDM server.
3. Device downloads the MDM enrollment profile from the server.
4. Device installs the profile, generates an identity certificate, and begins the check-in protocol.
5. The device is **supervised**, giving the MDM server full management authority including silent app install, restrictions, and kiosk mode.

ADE enrollment survives factory reset -- the device re-enrolls on next activation.

### 2. Account-Driven Enrollment

Introduced in iOS 15 / macOS 14. The user initiates enrollment from Settings (or System Settings on macOS):

1. User goes to **Settings > General > VPN & Device Management > Sign in to Work or School Account**.
2. User authenticates with their organization credentials (federated via the identity provider).
3. The identity provider redirects to the MDM server's enrollment URL.
4. Device installs the MDM profile with either BYOD or ADDE (Account-Driven Device Enrollment) scope.

BYOD enrollment uses a **Managed Apple Account** and cryptographically separates personal and managed data. ADDE provides full device management similar to ADE.

### 3. Manual Profile Installation

The simplest method -- a user downloads and installs a `.mobileconfig` file:

1. User navigates to an enrollment URL in Safari or receives the profile via email.
2. Safari downloads the profile; user is directed to **Settings > General > Profiles** to review it.
3. User taps **Install** and authenticates with their device passcode.
4. Device installs the MDM enrollment profile and begins check-in.

The device is **not supervised** and the user can remove the profile at any time.

## Check-in Protocol

The device sends HTTP PUT requests to the `CheckInURL` (falls back to `ServerURL` if `CheckInURL` is not specified in the enrollment payload). The request body is a property list. The `MessageType` key identifies the message.

### Authenticate

First message sent after enrollment profile installation.

Key fields: `MessageType` ("Authenticate"), `UDID`, `Topic`, `BuildVersion`, `ProductName`, `SerialNumber`, `Model`, `IMEI`, `OSVersion`.

The server should respond with HTTP 200 to continue enrollment or 410 to reject it.

### TokenUpdate

Sent immediately after Authenticate and whenever the device's push token changes. Delivers the values the server needs to send push notifications:

- **PushMagic** -- a UUID string the server must include in every APNs push payload.
- **Token** -- a base64-encoded 32-byte device push token for APNs delivery.
- **UnlockToken** -- base64-encoded escrow key for clearing the device passcode (device channel only).

The server must store `Token` and `PushMagic` to be able to wake the device. TokenUpdate can arrive at any time to refresh these values.

### CheckOut

Sent when the MDM profile is removed from the device, but **only** if `CheckOutWhenRemoved` is set to `true` in the enrollment payload. Contains `MessageType` ("CheckOut"), `UDID`, and `Topic`. After this message the server should stop sending pushes.

### SetBootstrapToken / GetBootstrapToken

macOS only. Used to escrow and retrieve the Bootstrap Token for Apple Silicon and T2 Macs. The `SetBootstrapToken` message contains the base64-encoded token; `GetBootstrapToken` requests it back from the server.

### DeclarativeManagement

Sent when the device supports Declarative Device Management (DDM). Signals that the device is ready to receive declarations. The server responds with the DDM synchronization tokens.

### UserAuthenticate

macOS and Shared iPad. Sent when a user logs in and per-user MDM is configured. The server can challenge the user or accept the authentication. On Shared iPad, this enables user-scoped management.

## Command-Response Cycle

The MDM protocol follows a pull model -- the server cannot push commands directly. Instead:

1. **Server queues a command** internally (not sent to device yet).
2. **Server sends a silent APNs push** to the device using the stored `Token` and `PushMagic`.
3. **Device receives the push** and opens an HTTPS PUT connection to `ServerURL`.
4. **Server responds with a command plist** containing a `CommandUUID` and a `Command` dictionary.
5. **Device executes the command** and sends the result back as an HTTPS PUT to `ServerURL`.
6. **Server can pipeline** -- respond to the result with the next command, or send an empty body (HTTP 200 with no content) to signal "no more commands."
7. **Device may return `NotNow`** if it cannot process the command at this time (e.g., device is locked). The server should re-queue and retry later.

### Command Plist Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CommandUUID</key>
    <string>a1b2c3d4-e5f6-7890-abcd-ef1234567890</string>
    <key>Command</key>
    <dict>
        <key>RequestType</key>
        <string>DeviceInformation</string>
        <key>Queries</key>
        <array>
            <string>DeviceName</string>
            <string>OSVersion</string>
            <string>SerialNumber</string>
        </array>
    </dict>
</dict>
</plist>
```

### Response Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>UDID</key>
    <string>ab12cd34-ef56-7890-abcd-1234567890ab</string>
    <key>Status</key>
    <string>Acknowledged</string>
    <key>CommandUUID</key>
    <string>a1b2c3d4-e5f6-7890-abcd-ef1234567890</string>
    <key>QueryResponses</key>
    <dict>
        <key>DeviceName</key>
        <string>John's iPhone</string>
        <key>OSVersion</key>
        <string>18.4</string>
        <key>SerialNumber</key>
        <string>DNXXXX1234</string>
    </dict>
</dict>
</plist>
```

### Status Values

| Status | Meaning |
|---|---|
| `Acknowledged` | Command executed successfully; response contains result data |
| `Error` | Command failed; response contains `ErrorChain` array with details |
| `CommandFormatError` | Command plist was malformed or contained invalid keys |
| `Idle` | Device has no pending command to report on (initial connect) |
| `NotNow` | Device cannot process now; server should re-queue for later |

## APNs Push Mechanism

MDM uses **silent push notifications** via Apple Push Notification service. The push carries no user-visible alert -- it only wakes the device to contact the MDM server.

- The **push certificate** (obtained from Apple's Push Certificates Portal) must match the `Topic` string in the MDM enrollment payload.
- The **Token** is a 32-byte device-specific push token, delivered via `TokenUpdate`, base64-encoded.
- The **PushMagic** is a UUID string assigned by the device during `TokenUpdate`.

### Push Payload Format

The APNs payload must contain the `mdm` key set to the device's `PushMagic`:

```json
{
    "mdm": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

This is the **entire** payload. No `aps` dictionary, no `alert`, no `badge`. The APNs request uses the push topic from the MDM certificate.

### Failure Handling

- If the push fails (device offline, token expired), the device will still contact the server when it comes online via its built-in periodic check-in schedule.
- If APNs returns an invalid token response, the server should wait for a new `TokenUpdate` from the device.
- Push tokens can change after OS updates, device restores, or profile re-installations.

## MDM Enrollment Payload (com.apple.mdm)

The MDM payload is embedded in the enrollment profile and configures the device's MDM client.

| Key | Type | Required | Description |
|---|---|---|---|
| `ServerURL` | String | Yes | URL the device contacts for commands (HTTPS) |
| `CheckInURL` | String | No | URL for check-in messages; defaults to `ServerURL` |
| `Topic` | String | Yes | APNs push topic, format `com.apple.mgmt.External.<UUID>` |
| `IdentityCertificateUUID` | String | Yes | UUID of the identity certificate payload used for client TLS auth |
| `AccessRights` | Integer | Yes | Bitmask of management rights; must not be 0 |
| `SignMessage` | Boolean | No | If true, device adds `Mdm-Signature` header (CMS signed) to requests |
| `CheckOutWhenRemoved` | Boolean | No | If true, device sends CheckOut on profile removal |
| `ServerCapabilities` | Array | No | Strings like `com.apple.mdm.per-user-connections`, `com.apple.mdm.bootstraptoken` |
| `UseDevelopmentAPNS` | Boolean | No | If true, use APNs sandbox environment |
| `EnrollmentMode` | String | No | `BYOD` for user enrollment or `ADDE` for account-driven device enrollment |
| `RequiredAppIDForMDM` | Integer | No | App Store ID of an app that must be installed for enrollment to proceed |

### AccessRights Bitmask

| Value | Permission |
|---|---|
| 1 | Inspect configuration profiles |
| 2 | Install and remove configuration profiles |
| 4 | Remove MDM profile (device lock and wipe still available) |
| 8 | Inspect device lock passcode |
| 16 | Wipe device |
| 32 | Query device information (IMEI, serial, etc.) |
| 64 | Query network information (WiFi MAC, IP, etc.) |
| 128 | Install and remove provisioning profiles |
| 256 | Install applications |
| 512 | Manage app settings and configuration |
| 1024 | Manage app restrictions (e.g., data loss prevention) |
| 2048 | Manage media |
| 4096 | Change device settings |

**Full access** = 8191 (sum of all values: 1 + 2 + 4 + 8 + 16 + 32 + 64 + 128 + 256 + 512 + 1024 + 2048 + 4096).

For User Enrollment (BYOD), the server should not request rights that apply only to supervised/device-level management.

## Enrollment Types

The enrollment type determines the device identity format and management scope.

| Type | ID Format | Scope |
|---|---|---|
| Device channel | `UDID` | Full device management |
| User channel (macOS) | `UDID:UserID` | Per-user management on macOS |
| User Enrollment (BYOD) | `EnrollmentID` (opaque, not the UDID) | Managed data separation, limited rights |
| Shared iPad | `UDID:ShortName` | Per-user sessions on supervised Shared iPad |

- **Device channel** is used for ADE, manual enrollment, and ADDE. The server receives the device UDID and has full management authority (subject to `AccessRights`).
- **User channel** applies on macOS when per-user MDM connections are configured via `ServerCapabilities`. Commands target a specific user's context.
- **User Enrollment** (BYOD) uses a cryptographic `EnrollmentID` instead of the UDID to protect user privacy. The MDM server cannot see the real device identifier and has a restricted command set.
- **Shared iPad** enrollment uses the device UDID combined with the user's Managed Apple Account short name, enabling user-scoped policies on shared devices.
