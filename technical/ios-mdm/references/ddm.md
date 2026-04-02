# Declarative Device Management (DDM) Reference

## Table of Contents

- [Overview](#overview)
- [Declaration Types](#declaration-types)
- [Status Channel](#status-channel)
- [Activation Predicates](#activation-predicates)
- [DDM Synchronization Protocol](#ddm-synchronization-protocol)
- [When to Use DDM vs Traditional MDM](#when-to-use-ddm-vs-traditional-mdm)

## Overview

DDM shifts device management from server-driven (push commands) to device-autonomous (apply declarations, report status proactively). Introduced at WWDC 2021 (iOS 15, macOS 12), expanded yearly. DDM coexists with traditional MDM -- it does not replace it.

Key differences:

- **Traditional MDM:** server pushes commands, device is passive, server polls for state.
- **DDM:** device receives declarations, applies them independently, proactively reports status changes.
- DDM configurations are applied atomically within activations.

The server's role changes from "tell the device what to do right now" to "describe the desired state and let the device converge to it."

## Declaration Types

DDM organizes everything into four declaration types: configurations, activations, assets, and management.

### Configurations (36+ types)

Configurations are the policies to apply. Major categories:

**Account configs:** account.caldav, account.carddav, account.exchange, account.google, account.ldap, account.mail, account.subscribed-calendar

**App management:** app.managed -- install and manage apps declaratively. Available iOS 17+. Supports required/optional app designation, app configuration, and feedback via status channel.

**Passcode & Security:** passcode.settings -- passcode requirements (Apple's recommended approach over the traditional passcode payload for iOS 17+). Also: security.certificate, security.identity, security.passkey.attestation.

**Software Update:** softwareupdate.settings -- control update behavior (deferrals, rapid security responses). softwareupdate.enforcement.specific -- force a specific OS version by a deadline with notifications. Apple's recommended approach for managed updates on iOS 17+.

**Device Settings:** keyboard.settings, siri.settings, intelligence.settings, external-intelligence.settings, math.settings, audio-accessory.settings, diskmanagement.settings.

**Safari:** safari.bookmarks, safari.extensions.settings, safari.settings.

**Legacy Bridge:** legacy -- wraps a traditional configuration profile as a declaration, allowing gradual DDM migration. legacy.interactive -- wraps profiles that require user interaction (e.g., WiFi with credentials).

**Management:** management.status-subscriptions -- subscribe to status items. management.test -- for testing DDM integration.

### Activations

Activations are atomic sets of configurations. One type exists: `simple`.

- References an array of configuration identifiers.
- Optional predicate expression for conditional activation.
- All-or-nothing: if any referenced configuration or asset is invalid, none activate.
- Many-to-many relationship with configurations -- a configuration can appear in multiple activations.
- A device can have multiple activations active simultaneously.

### Assets

Assets provide reference data for configurations (credentials, large data, per-user info).

- One-to-many: a single asset can be referenced by multiple configurations.
- Types: credential.certificate, credential.identity, credential.scep, credential.acme, data, useridentity.
- Assets are fetched separately from declarations, reducing payload size.

### Management

Organizational context declarations:

- **Organization details:** name and other identifying information.
- **Server capabilities:** tells the device which DDM features the server supports (the device adjusts behavior accordingly).
- **Properties:** custom key-value pairs the server defines. These are available in activation predicates for conditional logic.

## Status Channel

The device proactively reports status items the server has subscribed to. 48+ available status items across categories:

**Device identification:** device.identifier.serial-number, device.identifier.udid, device.model.family, device.model.identifier, device.model.marketing-name.

**OS info:** device.operating-system.version, device.operating-system.build-version, device.operating-system.family, device.operating-system.supplemental.build-version, device.operating-system.supplemental.version.

**Software update:** softwareupdate.install-state, softwareupdate.pending-version, softwareupdate.failure-reason, softwareupdate.install-reason.

**Security:** security.certificate.list, passcode.is-compliant, passcode.is-present.

**Apps:** app.managed.list -- list of managed apps and their states (installed, installing, failed, etc.).

**Battery:** device.power.battery-health.

**Management:** management.declarations -- current declaration state and errors. management.client-capabilities -- which DDM features the device supports.

**Disk:** diskmanagement.filevault.enabled (macOS only).

Subscribe via the management.status-subscriptions configuration. The server only receives updates for subscribed items. When a subscribed item changes, the device sends a StatusReport to the server without being prompted.

## Activation Predicates

Predicates allow conditional activation based on device state or custom properties. This enables a single set of declarations to serve an entire fleet, with each device activating only its relevant configurations.

### Management Properties

Set via the management.properties declaration:

- Supported types: Integer, String, Boolean, Array.
- Example: define a "department" property, then use predicates to activate different configurations per department.

### Predicate Evaluation

Predicates can evaluate:

- **Status items** (e.g., device OS version, device model family).
- **Management properties** (custom key-value pairs set by the server).
- **Logical operators:** AND, OR, NOT for combining conditions.

### Common Pattern

1. Server pushes all declarations for all departments/roles to every device.
2. Server sets management properties per device (e.g., department, role).
3. Activations use predicates to match on properties.
4. Each device activates only the configurations matching its properties.

This eliminates the need for server-side group targeting -- the device handles it locally.

## DDM Synchronization Protocol

The synchronization flow between server and device:

1. Server sends a DeclarativeManagement push command (or device checks in with a DeclarativeManagement message type).
2. Device fetches the declaration manifest -- a list of all declaration identifiers and server tokens.
3. Device compares the manifest with its local state.
4. Device fetches only changed or new declarations (token-based diffing).
5. Device validates and applies declarations atomically.
6. Device reports status for all subscribed items.
7. On any subsequent subscribed status change, device proactively sends a StatusReport without server prompting.

**Synchronization tokens** prevent unnecessary data transfer. Each declaration has a server token; the device caches it and only re-fetches when the token changes. The manifest itself also has a token for the same purpose.

**Error handling:** if a declaration fails validation, the device reports the error via the management.declarations status item. Other valid declarations continue to apply.

## When to Use DDM vs Traditional MDM

| Use Case | Recommended Approach |
|---|---|
| Software update enforcement | DDM: softwareupdate.enforcement.specific |
| Passcode policy | DDM: passcode.settings (iOS 17+) |
| App installation | DDM: app.managed (iOS 17+) |
| Account configuration | DDM: account.* declarations |
| Real-time status monitoring | DDM: status channel with subscriptions |
| Querying device info on demand | Traditional MDM: DeviceInformation command |
| One-time actions (lock, erase, restart) | Traditional MDM commands |
| Restrictions | Traditional MDM: Restrictions payload (no DDM equivalent yet) |
| WiFi / VPN / certificate profiles | DDM legacy bridge or traditional MDM |
| Conditional config by department/role | DDM: activation predicates with management properties |

**Migration strategy:** adopt DDM for new capabilities (software update enforcement, app management, passcode) while keeping traditional MDM for restrictions and one-time commands. Use the legacy bridge declaration to wrap existing profiles during gradual migration.

**Minimum versions for key DDM features:**

- Basic DDM support: iOS 15, macOS 12
- Software update enforcement: iOS 17, macOS 14
- App management: iOS 17, macOS 14
- Passcode settings: iOS 17, macOS 14
- Activation predicates: iOS 16, macOS 13
- Status channel: iOS 16, macOS 13
