# iOS Device Supervision Reference

## Table of Contents

- [What Is Supervision](#what-is-supervision)
- [How to Supervise Devices](#how-to-supervise-devices)
- [Supervised-Only Commands](#supervised-only-commands)
- [Supervised-Only Payloads](#supervised-only-payloads)
- [Supervised-Only Restrictions (Major Categories)](#supervised-only-restrictions-major-categories)
- [Unsupervised Capabilities](#unsupervised-capabilities)
- [Checking Supervision Status](#checking-supervision-status)

## What Is Supervision

Supervision marks an iOS/iPadOS device as organization-owned, granting the MDM server enhanced management control. Supervised devices display a message in Settings: "This iPhone is supervised and managed by [Organization]."

Supervision is a prerequisite for many MDM capabilities -- commands, restrictions, and payloads that are unavailable on unsupervised devices.

## How to Supervise Devices

### Automated Device Enrollment (ADE)

- Via Apple Business Manager or Apple School Manager
- Device is enrolled during initial setup (Setup Assistant)
- Wireless, zero-touch -- device automatically becomes supervised
- Organization purchases or registers devices with Apple
- The recommended method for organization-owned devices

### Apple Configurator for Mac

- USB connection required
- Can supervise any iPhone/iPad by erasing it
- Creates a supervision identity tied to the Mac
- Useful for devices not purchased through ADE

### Apple Configurator for iPhone

- Proximity-based (NFC/Bluetooth)
- Can add devices to Apple Business Manager
- iOS 16+

## Supervised-Only Commands

These MDM commands only work on supervised devices:

| Command | Platform | Notes |
|---------|----------|-------|
| RestartDevice | iOS, iPadOS, tvOS | NotifyUser option available |
| ShutDownDevice | iOS, iPadOS | |
| EnableLostMode | iOS, iPadOS | Locks device, shows message |
| DisableLostMode | iOS, iPadOS | |
| DeviceLocation | iOS, iPadOS | Returns lat/long |
| PlayLostModeSound | iOS, iPadOS | |
| ActivationLockBypassCode | iOS, iPadOS | |
| ClearActivationLockBypassCode | iOS, iPadOS | |
| ScheduleOSUpdate | iOS, iPadOS, tvOS | Download/install updates |
| DeleteUser | macOS (Apple silicon) | |

Commands that do NOT require supervision: DeviceLock, EraseDevice, ClearPasscode, DeviceInformation, SecurityInfo, InstallProfile, RemoveProfile, InstallApplication, RemoveApplication, all query commands.

## Supervised-Only Payloads

| Payload | Identifier | Why supervised |
|---------|-----------|---------------|
| App Lock (Single App Mode) | com.apple.app.lock | Locks device to one app |
| Home Screen Layout | com.apple.homescreenlayout | Forces icon arrangement |
| Global HTTP Proxy | com.apple.proxy.http.global | Routes all traffic through proxy |
| Shared Device Configuration | com.apple.shareddeviceconfiguration | Shared iPad settings |

PayloadRemovalDisallowed in the profile wrapper also requires supervision on iOS/tvOS.

## Supervised-Only Restrictions (Major Categories)

Over 80 restriction keys require supervision. The major categories:

**App control:**

- allowListedAppBundleIDs / blockedAppBundleIDs (allowlist/blocklist apps)
- allowRemoveSystemApps
- allowHideApps, allowLockApps (iOS 18+)

**Safari and browser:**

- allowSafari (disable entirely)
- allowDefaultBrowserModification

**Communication:**

- allowChat (iMessage), allowRCS, allowAirDrop
- allowCallRecordingInPhone (iOS 18.2+)

**Content:**

- allowBookstore, allowPodcasts, allowNews
- allowGameCenter

**Device settings:**

- allowDeviceNameModification, allowWallpaperModification
- allowBluetoothModification, allowNotificationsModification
- allowCellularPlanModification, allowPasscodeModification
- allowFingerprintModification, allowEraseContentAndSettings

**Security:**

- forceWiFiWhitelisting (only join managed WiFi)
- allowUSBRestrictedMode, allowConfigurationProfileInstallation
- allowVPNCreation

**AI (iOS 18+):**

- allowGenmoji, allowImagePlayground, allowWritingTools
- allowAppleIntelligence, allowExternalIntelligence

## Unsupervised Capabilities

What you CAN do without supervision:

- Install/remove configuration profiles
- Install/remove apps (via MDM command)
- Lock the device, erase the device, clear passcode
- Query all device information
- Set passcode policy, WiFi, VPN, email, certificate payloads
- Web content filter, DNS settings
- Most restriction keys related to iCloud, camera, screenshots, backups
- Managed Open In (allowOpenFromManagedToUnmanaged)

The key limitation: users can remove the MDM profile on unsupervised devices, which removes all management. On supervised devices, profile removal can be prevented.

## Checking Supervision Status

Query via DeviceInformation command with the `IsSupervised` query key. Returns boolean.

Also available via DDM status: device properties include supervision status.

In code, check the `IsSupervised` field in the DeviceInformation response before attempting supervised-only operations.
