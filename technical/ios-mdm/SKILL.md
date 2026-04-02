---
name: ios-mdm
description: >
  Apple iOS/iPadOS MDM (Mobile Device Management) protocol expert. Use this skill whenever
  the user is building or working with an MDM server that manages Apple devices, constructing
  MDM command plists, building configuration profiles or mobileconfig files, working with
  payload types (restrictions, WiFi, VPN, web content filter, passcode policy, certificates,
  app lock, DNS settings), querying device information, managing apps via MDM, implementing
  enrollment flows, handling check-in messages (Authenticate, TokenUpdate, CheckOut), using
  APNs push for MDM wake-up, working with Declarative Device Management (DDM) declarations
  and status subscriptions, or dealing with supervised vs unsupervised device capabilities.
  Also trigger when code constructs plist XML for MDM commands, references RequestType or
  CommandUUID fields, mentions com.apple.mdm or com.apple.applicationaccess payloads, or
  the user asks about what an MDM server can do with iOS devices, what data it can read,
  or what restrictions it can enforce. This skill covers the Apple MDM protocol, not any
  specific MDM server implementation -- see the nanomdm skill for server-side details.
metadata:
  version: "1.0.0"
  source: developer.apple.com/documentation/devicemanagement
---

# Apple iOS/iPadOS MDM Protocol

You are an expert at the Apple MDM protocol for managing iOS and iPadOS devices. This skill covers the protocol itself — commands, profiles, payloads, DDM, and supervision — grounded in Apple's official documentation and the `apple/device-management` GitHub repository.

For server-side implementation details (NanoMDM endpoints, storage backends, middleware), see the **nanomdm** skill.

## Core Principles

1. **Push-then-poll architecture** — The MDM server never sends commands directly. It sends a silent APNs push to wake the device, then the device connects to the ServerURL to fetch queued commands. The device drives the connection.
2. **Commands are raw plist XML** — Every MDM command is a property list with a `CommandUUID` and a `Command` dictionary containing a `RequestType`. Responses include a `Status` field (Acknowledged, Error, CommandFormatError, Idle, NotNow).
3. **Profiles are layered** — A configuration profile is an outer plist wrapper (PayloadType "Configuration") containing a `PayloadContent` array of individual payload dictionaries. Each payload has its own type, UUID, and identifier.
4. **Supervision unlocks power** — Many restrictions, commands, and payloads require the device to be supervised (enrolled via ADE or Apple Configurator). Always check supervision requirements before assuming a capability is available.
5. **DDM is the future** — Declarative Device Management shifts the device from passive (server pushes) to autonomous (device applies declarations and reports status proactively). It coexists with traditional MDM and is Apple's recommended path for new capabilities like software update enforcement.
6. **Separate profiles by function** — Apple recommends one profile per concern (WiFi in one, VPN in another, restrictions in a third) rather than bundling everything into a single profile. This allows granular removal and replacement.

## How to Use This Skill

Before generating code or constructing payloads, read the relevant reference file(s):

- **`references/protocol.md`** — Enrollment flow, check-in messages, command-response cycle, APNs push, MDM payload keys, AccessRights bitmask
- **`references/commands.md`** — Complete MDM command list with XML structures, DeviceInformation query keys, security commands, app management, profile management
- **`references/profiles.md`** — Configuration profile XML structure, top-level keys, common payload keys, major payload type identifiers and their key fields
- **`references/restrictions.md`** — Full restriction matrix: what requires supervision, app controls, content, communication, security, AI/ML restrictions (iOS 18+)
- **`references/ddm.md`** — Declarative Device Management: declarations, activations, assets, status channel, predicate expressions, configuration types
- **`references/supervision.md`** — What supervision enables, how devices get supervised, supervised-only commands and payloads

## Quick Reference

### MDM Communication Cycle

```
1. Server queues a command for the device
2. Server sends silent APNs push (using stored Token + PushMagic)
3. Device receives push, connects to ServerURL via HTTPS
4. Server delivers command plist
5. Device processes command, returns response plist
6. Repeat until server responds with empty body (no more commands)
```

### Command Plist Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>CommandUUID</key>
  <string>0001_DeviceInformation</string>
  <key>Command</key>
  <dict>
    <key>RequestType</key>
    <string>DeviceInformation</string>
    <key>Queries</key>
    <array>
      <string>UDID</string>
      <string>SerialNumber</string>
      <string>DeviceName</string>
      <string>OSVersion</string>
      <string>BatteryLevel</string>
      <string>IsSupervised</string>
    </array>
  </dict>
</dict>
</plist>
```

### Response Status Values

| Status | Meaning |
|--------|---------|
| `Acknowledged` | Command processed successfully |
| `Error` | Command failed (check ErrorChain) |
| `CommandFormatError` | Malformed command plist |
| `Idle` | No pending command (device checking in) |
| `NotNow` | Device busy, will retry later |

### Profile Wrapper Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>PayloadType</key>
  <string>Configuration</string>
  <key>PayloadVersion</key>
  <integer>1</integer>
  <key>PayloadIdentifier</key>
  <string>com.example.wifi-profile</string>
  <key>PayloadUUID</key>
  <string>A1B2C3D4-E5F6-7890-ABCD-EF1234567890</string>
  <key>PayloadDisplayName</key>
  <string>Corporate WiFi</string>
  <key>PayloadContent</key>
  <array>
    <!-- Individual payload dictionaries here -->
  </array>
</dict>
</plist>
```

### Key Command Categories

| Category | Commands | Notes |
|----------|----------|-------|
| Device info | `DeviceInformation`, `SecurityInfo`, `CertificateList` | Query device state |
| Apps | `InstallApplication`, `RemoveApplication`, `InstalledApplicationList`, `ManagedApplicationList` | App Store or enterprise apps |
| Profiles | `InstallProfile`, `RemoveProfile`, `ProfileList` | Configuration profile management |
| Security | `DeviceLock`, `EraseDevice`, `ClearPasscode` | No supervision required |
| Lost Mode | `EnableLostMode`, `DisableLostMode`, `DeviceLocation` | Supervised only |
| Updates | `ScheduleOSUpdate`, `AvailableOSUpdates`, `OSUpdateStatus` | Supervised only |
| Control | `RestartDevice`, `ShutDownDevice` | Supervised only on iOS |
| Settings | `Settings` | Modify device settings |

### Key Payload Types

| Payload | Identifier | Supervised? |
|---------|-----------|-------------|
| Restrictions | `com.apple.applicationaccess` | Partial (80+ keys need supervision) |
| WiFi | `com.apple.wifi.managed` | No |
| VPN | `com.apple.vpn.managed` | No |
| Per-App VPN | `com.apple.vpn.managed.applayer` | No |
| Web Content Filter | `com.apple.webcontent-filter` | No |
| Passcode Policy | `com.apple.mobiledevice.passwordpolicy` | No |
| App Lock | `com.apple.app.lock` | Yes (ADE only) |
| DNS Settings | `com.apple.dnsSettings.managed` | No |
| Certificate (PKCS12) | `com.apple.security.pkcs12` | No |
| Certificate (PEM) | `com.apple.security.pem` | No |
| Certificate (SCEP) | `com.apple.security.scep` | No |
| Global HTTP Proxy | `com.apple.proxy.http.global` | Yes |
| Home Screen Layout | `com.apple.homescreenlayout` | Yes |
| Notifications | `com.apple.notificationsettings` | No |
| Domains | `com.apple.domains` | No |
| Web Clip | `com.apple.webClip.managed` | No |

### AccessRights Bitmask

| Value | Permission |
|-------|-----------|
| 1 | Inspect configuration profiles |
| 2 | Install/remove configuration profiles |
| 4 | Device lock and passcode removal |
| 8 | Device erase |
| 16 | Query device information |
| 32 | Query network information |
| 64 | Inspect provisioning profiles |
| 128 | Install/remove provisioning profiles |
| 256 | Inspect installed applications |
| 512 | Restriction-related queries |
| 1024 | Security-related queries |
| 2048 | Manipulate settings |
| 4096 | App management |

Full access = **8191** (all bits set).

## Common Mistakes

- **Sending commands directly instead of via APNs** — The MDM server cannot push commands to the device. It must send a silent APNs notification; the device then connects to fetch commands.
- **Assuming all restrictions work without supervision** — Over 80 restriction keys require supervision. Always check the restriction matrix before deploying a restrictions profile.
- **Using AccessRights 4095 instead of 8191** — 4095 omits app management (bit 4096). Use 8191 for full access.
- **Bundling all payloads in one profile** — If you need to update WiFi settings, you'd have to replace the entire profile. Separate profiles by function.
- **Forgetting PayloadUUID on inner payloads** — Every payload dictionary inside PayloadContent needs its own unique PayloadUUID, not just the outer profile.
- **Ignoring NotNow responses** — The device returns NotNow when it can't process a command right now (e.g., during setup). The server should re-queue the command, not discard it.
- **Not checking PayloadRemovalDisallowed requires supervision** — On iOS/tvOS, preventing profile removal requires the device to be supervised. On unsupervised devices, users can always remove MDM-installed profiles.
