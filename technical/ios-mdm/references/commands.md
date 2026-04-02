# iOS MDM Commands Reference

## Table of Contents

- [Device Information Queries](#device-information-queries)
- [App Management](#app-management)
- [Profile Management](#profile-management)
- [Security Commands](#security-commands)
- [Device Control](#device-control)
- [Software Update](#software-update)
- [Activation Lock](#activation-lock)

## Device Information Queries

### DeviceInformation

Queries device attributes. Takes a `Queries` array of string keys specifying which values to return.

**Query keys by category:**

| Category | Keys |
|----------|------|
| Device | UDID, SerialNumber, DeviceName, Model, ModelName, ModelNumber, ProductName |
| OS | OSVersion, BuildVersion, SoftwareUpdateDeviceID |
| Hardware | DeviceCapacity, AvailableDeviceCapacity, BatteryLevel, CellularTechnology |
| Status | IsSupervised, IsMultiUser, IsActivationLockEnabled, IsCloudBackupEnabled |
| Network | WiFiMAC, BluetoothMAC, EthernetMAC |
| Security | PushToken, TimeZone, IsDeviceLocatorServiceEnabled |

**Command:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
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
</plist>
```

**Response (QueryResponses dict):**

```xml
<dict>
  <key>QueryResponses</key>
  <dict>
    <key>UDID</key>
    <string>ab1cd234-ef56-7890-ab12-cd34ef567890</string>
    <key>SerialNumber</key>
    <string>C0EXAMPLE123</string>
    <key>DeviceName</key>
    <string>John's iPhone</string>
    <key>OSVersion</key>
    <string>17.4.1</string>
    <key>BatteryLevel</key>
    <real>0.85</real>
    <key>IsSupervised</key>
    <true/>
  </dict>
</dict>
```

### SecurityInfo

Returns security-related information about the device. No additional parameters required.

**RequestType:** `SecurityInfo`

**Response keys:** HardwareEncryptionCaps, PasscodePresent, PasscodeCompliant, PasscodeCompliantWithProfiles, IsPasscodeLockGracePeriodEnforced, FDE_Enabled, FDE_HasPersonalRecoveryKey, FDE_HasInstitutionalRecoveryKey, FDE_PersonalRecoveryKeyCMS, FDE_PersonalRecoveryKeyDeviceKey, FirewallSettings (BlockAllIncoming, StealthMode, IsEnabled), ManagementStatus (EnrolledViaDEP, UserApprovedMDM).

### CertificateList

Returns an array of certificates installed on the device.

**RequestType:** `CertificateList`

**Response:** Array of dictionaries, each containing:
- **CommonName** (string) — subject common name
- **IsIdentity** (boolean) — whether the certificate includes a private key
- **Data** (data) — DER-encoded certificate bytes

## App Management

### InstallApplication

Installs an app from the App Store or an enterprise manifest URL.

**Key parameters:**
- **iTunesStoreID** (integer) — App Store app ID for Store apps
- **ManifestURL** (string) — HTTPS URL to enterprise app manifest plist
- **ManagementFlags** (integer) — bitmask: `1` = remove app on unenroll, `4` = prevent backup of app data
- **Options** (dict) — contains `PurchaseMethod` (0 = user-based VPP, 1 = device-based VPP)
- **ChangeManagementState** (string) — set to `Managed` to take management of an already-installed app (supervised only)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>RequestType</key>
  <string>InstallApplication</string>
  <key>iTunesStoreID</key>
  <integer>361309726</integer>
  <key>ManagementFlags</key>
  <integer>1</integer>
  <key>Options</key>
  <dict>
    <key>PurchaseMethod</key>
    <integer>1</integer>
  </dict>
</dict>
</plist>
```

### RemoveApplication

Removes a managed application by bundle identifier.

**RequestType:** `RemoveApplication`

**Parameters:**
- **Identifier** (string) — the app's bundle ID (e.g., `com.example.app`)

Only works for MDM-managed apps. On unsupervised devices, only apps installed by MDM can be removed. On supervised devices, any managed app can be removed.

### InstalledApplicationList

Returns all apps installed on the device.

**RequestType:** `InstalledApplicationList`

**Response:** Array of dictionaries with: Identifier, Name, ShortVersion, Version, BundleSize, DynamicSize, IsValidated.

### ManagedApplicationList

Returns only MDM-managed apps with their management status.

**RequestType:** `ManagedApplicationList`

**Response:** Dictionary keyed by bundle ID, each containing: Status (Managed, ManagedButUninstalled, UserInstalledOverride), ManagementFlags, HasConfiguration, HasFeedback, IsValidated.

## Profile Management

### InstallProfile

Installs a configuration profile on the device.

**Parameters:**
- **Payload** (data) — the complete mobileconfig profile, base64-encoded

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>RequestType</key>
  <string>InstallProfile</string>
  <key>Payload</key>
  <data>
    <!-- Base64-encoded .mobileconfig profile data -->
    TUlJQ2...
  </data>
</dict>
</plist>
```

### RemoveProfile

Removes a previously installed configuration profile.

**RequestType:** `RemoveProfile`

**Parameters:**
- **Identifier** (string) — the PayloadIdentifier of the profile to remove

Only profiles installed by MDM can be removed by MDM. On supervised devices, all profiles can be removed.

### ProfileList

Returns all installed configuration profiles.

**RequestType:** `ProfileList`

**Response:** Array of dictionaries with: PayloadIdentifier, PayloadUUID, PayloadDisplayName, PayloadDescription, PayloadOrganization, HasRemovalPasscode, IsEncrypted, IsManaged, PayloadContent (array of payload dictionaries within the profile).

## Security Commands

### DeviceLock

Immediately locks the device.

**RequestType:** `DeviceLock`

**Parameters:**
- **Message** (string, optional) — message displayed on the lock screen
- **PhoneNumber** (string, optional) — phone number displayed on the lock screen
- **PIN** (string, optional) — 6-character unlock PIN, macOS only

Does **not** require supervision. Works on all enrolled devices.

### EraseDevice

Remotely wipes the device to factory settings.

**RequestType:** `EraseDevice`

**Parameters:**
- **PreserveDataPlan** (boolean, iOS 11+) — keep cellular data plan through wipe
- **DisallowProximitySetup** (boolean, iOS 11.3+) — prevent Quick Start after erase
- **PIN** (string, macOS only) — 6-character EFI lock PIN
- **ReturnToService** (dict, iOS 17+) — auto re-enroll after erase:
  - **WiFiProfileData** (data) — base64 Wi-Fi profile for network connectivity
  - **MDMProfileData** (data) — base64 MDM enrollment profile
  - **IsSupervised** (boolean) — whether the device should be supervised after re-enrollment

Does **not** require supervision. This is a destructive operation.

### ClearPasscode

Clears the device passcode so the user can set a new one.

**RequestType:** `ClearPasscode`

**Parameters:**
- **UnlockToken** (data) — the escrow keybag token received during the TokenUpdate check-in message

iOS and iPadOS only. Not available on macOS.

### EnableLostMode

Locks the device into Lost Mode with a custom message and enables location tracking. **Supervised only.**

**RequestType:** `EnableLostMode`

**Parameters:**
- **Message** (string) — message displayed on the lock screen
- **PhoneNumber** (string) — phone number displayed on the lock screen
- **Footnote** (string) — additional text at bottom of lock screen

### DisableLostMode

Exits Lost Mode on the device. **Supervised only.**

**RequestType:** `DisableLostMode`

### DeviceLocation

Returns the device's current location while in Lost Mode. **Supervised only.**

**RequestType:** `DeviceLocation`

**Response:** Latitude, Longitude, HorizontalAccuracy, VerticalAccuracy, Altitude, Speed, Course, Timestamp.

### PlayLostModeSound

Plays an audible sound on a device in Lost Mode. **Supervised only.**

**RequestType:** `PlayLostModeSound`

## Device Control

### RestartDevice

Restarts the device. **Supervised only** on iOS/iPadOS.

**RequestType:** `RestartDevice`

**Parameters:**
- **NotifyUser** (boolean) — if true, warns the user before restarting
- **RebuildKernelCache** (boolean, macOS only) — rebuild the kernel cache during restart

### ShutDownDevice

Shuts down the device. **Supervised only** on iOS/iPadOS.

**RequestType:** `ShutDownDevice`

### Settings

A multi-purpose command that applies one or more device settings at once. Takes a `Settings` array where each entry is a dictionary with an `Item` key indicating the setting type.

**RequestType:** `Settings`

**Available settings (Item values):**
- **VoiceRoaming** — enable/disable voice roaming (supervised)
- **DataRoaming** — enable/disable data roaming (supervised)
- **PersonalHotspot** — enable/disable personal hotspot (supervised)
- **DeviceName** — set device name (supervised)
- **Wallpaper** — set lock screen or home screen wallpaper (supervised)
- **MaximumResidentUsers** — set max users on Shared iPad
- **DiagnosticSubmission** — enable/disable diagnostic data submission (supervised)
- **AppAnalytics** — enable/disable app analytics sharing (supervised)

**Example — set device name:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>RequestType</key>
  <string>Settings</string>
  <key>Settings</key>
  <array>
    <dict>
      <key>Item</key>
      <string>DeviceName</string>
      <key>DeviceName</key>
      <string>Lobby iPad 01</string>
    </dict>
  </array>
</dict>
</plist>
```

## Software Update

All software update commands require **supervision** on iOS/iPadOS.

### ScheduleOSUpdate

Schedules a software update with a specified install action.

**RequestType:** `ScheduleOSUpdate`

**Parameters:**
- **Updates** (array) — each entry contains:
  - **ProductKey** (string) — the update's product key from AvailableOSUpdates
  - **ProductVersion** (string, iOS 17.4+) — target OS version (alternative to ProductKey)
  - **InstallAction** (string):
    - `Default` — download and prompt
    - `DownloadOnly` — download but do not install
    - `InstallASAP` — download and install immediately, force restart
    - `NotifyOnly` — notify user that update is available
    - `InstallLater` — download and install at a convenient time

### AvailableOSUpdates

Returns a list of available software updates for the device.

**RequestType:** `AvailableOSUpdates`

**Response:** Array of dictionaries with: ProductKey, ProductName, Version, Build, DownloadSize, InstallSize, IsCritical, IsConfigurationDataUpdate, RestartRequired, AllowsInstallLater.

### OSUpdateStatus

Returns the current status of software updates on the device.

**RequestType:** `OSUpdateStatus`

**Response:** Array of dictionaries with: ProductKey, IsDownloaded, DownloadPercentComplete, Status (Idle, Downloading, Installing), MaxDeferrals, DeferralsRemaining.

## Activation Lock

### ActivationLockBypassCode

Retrieves the Activation Lock bypass code for a supervised device. The MDM server stores this code to bypass Activation Lock during device wipe or re-provisioning.

**RequestType:** `ActivationLockBypassCode`

**Supervised only.** The response contains the bypass code in the `ActivationLockBypassCode` key.

### ClearActivationLockBypassCode

Clears a previously escrowed Activation Lock bypass code from Apple's servers.

**RequestType:** `ClearActivationLockBypassCode`

**Supervised only.** Use this when decommissioning a device to ensure the old bypass code is invalidated.

---

*Based on Apple's MDM Protocol Reference documentation. All commands are sent as plist XML payloads from the MDM server to the device via APNs push notification and HTTPS callback.*
