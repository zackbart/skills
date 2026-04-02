---
name: android-device-owner
description: >
  Android Device Owner Mode (Device Policy Controller) assistant. Use when working with
  DevicePolicyManager, DeviceAdminReceiver, device owner provisioning, app hiding/suspension,
  web content filtering, kiosk/lock task mode, user restrictions, always-on VPN, silent app
  install, factory reset protection, or any Android Enterprise fully-managed device feature.
  Also triggers when code imports 'android.app.admin.DevicePolicyManager'. Use this skill
  whenever the user mentions Android enterprise management, MDM, device policy, managed devices,
  parental controls on Android, app blocking/hiding on Android, or building a DPC app -- even if
  they don't explicitly say "device owner mode."
metadata:
  version: "1.0.0"
  source: developer.android.com/work
---

# Android Device Owner Mode

You are an expert at building Android Device Policy Controller (DPC) apps that use Device Owner Mode for full device management. This skill references the official Android Enterprise documentation and DevicePolicyManager APIs.

## Important: DPC Allowlist (2025)

Google Play Protect now enforces an allowlist of approved DPCs during provisioning. Unapproved custom DPCs are blocked with a "harmful app" warning. The Play EMM API is closed to new registrations. For new enterprise management projects, Google recommends the Android Management API. Custom DPCs provisioned via ADB (`dpm set-device-owner`) still work for development and sideloaded deployments.

## Core Concepts

### Device Owner vs Profile Owner vs Device Admin

| Mode | Scope | Provisioning | Use Case |
|------|-------|-------------|----------|
| **Device Owner** | Full device control | Factory reset required, set during initial setup | Company-owned, fully managed devices |
| **Profile Owner** | Work profile only | Can be added to already-provisioned device | BYOD with work/personal separation |
| **Device Admin** | **Deprecated** (removed API 29) | Runtime activation | Legacy -- do not use for new projects |

Device Owner is the most powerful mode. It can hide/show apps, filter web content, enforce VPN, set user restrictions, silently install apps, control system updates, and perform factory reset -- all without user consent.

### DevicePolicyManager (DPM) -- The Central API

All device owner operations go through `DevicePolicyManager`. Get a reference:

```kotlin
val dpm = context.getSystemService(Context.DEVICE_POLICY_SERVICE) as DevicePolicyManager
val adminName = ComponentName(context, MyDeviceAdminReceiver::class.java)

// Guard all DPM calls with this check
if (dpm.isDeviceOwnerApp(context.packageName)) {
    // Safe to call device owner APIs
}
```

### DeviceAdminReceiver -- The Entry Point

Every DPC must declare a `DeviceAdminReceiver` subclass:

```kotlin
class MyDeviceAdminReceiver : DeviceAdminReceiver() {
    override fun onEnabled(context: Context, intent: Intent) {
        // Device admin activated -- apply initial restrictions
    }
    override fun onProfileProvisioningComplete(context: Context, intent: Intent) {
        // Provisioning finished -- set up policies
    }
}
```

Manifest registration:

```xml
<receiver android:name=".MyDeviceAdminReceiver"
    android:permission="android.permission.BIND_DEVICE_ADMIN"
    android:exported="true">
    <intent-filter>
        <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
        <action android:name="android.app.action.PROFILE_PROVISIONING_COMPLETE" />
    </intent-filter>
    <meta-data android:name="android.app.device_admin"
        android:resource="@xml/device_admin" />
</receiver>
```

Device admin XML (`res/xml/device_admin.xml`):

```xml
<device-admin>
    <uses-policies>
        <limit-password />
        <watch-login />
        <reset-password />
        <force-lock />
        <wipe-data />
        <expire-password />
        <encrypted-storage />
        <disable-camera />
        <disable-keyguard-features />
    </uses-policies>
</device-admin>
```

## How to Use This Skill

Before generating code, load the relevant reference file(s):

- **Provisioning & setup**: `cat references/provisioning.md`
- **App hiding, suspension & install**: `cat references/app-management.md`
- **Website blocking & DNS filtering**: `cat references/web-filtering.md`
- **Kiosk & lock task mode**: `cat references/kiosk-mode.md`
- **DISALLOW_* restrictions**: `cat references/user-restrictions.md`
- **Security, FRP & wipe**: `cat references/security.md`

## Quick API Reference

### App Management (API 21+)

| Method | Effect |
|--------|--------|
| `setApplicationHidden(admin, pkg, true)` | App invisible, cannot run, data preserved |
| `setPackagesSuspended(admin, pkgs, true)` | App grayed out, visible but non-functional (API 24+) |
| `setUninstallBlocked(admin, pkg, true)` | Prevents user from uninstalling |
| `enableSystemApp(admin, pkg)` | Re-enables system app disabled during provisioning |
| `isApplicationHidden(admin, pkg)` | Check if app is hidden |

### Web Content Filtering

No OS-level URL filtering API exists. Three approaches:

1. **Chrome managed configs** -- `setApplicationRestrictions()` with `URLBlocklist`/`URLAllowlist` keys. Simplest, Chrome-only.
2. **VPN-based DNS filtering** -- Custom `VpnService` intercepting DNS queries. Comprehensive, covers all apps.
3. **Proxy** -- `setRecommendedGlobalProxy()` for HTTP proxy filtering.

Combine with `setAlwaysOnVpnPackage(admin, vpnPkg, lockdown=true)` (API 24+) and `DISALLOW_CONFIG_VPN` to enforce.

### User Restrictions

```kotlin
// Apply
dpm.addUserRestriction(adminName, UserManager.DISALLOW_INSTALL_UNKNOWN_SOURCES)
// Clear
dpm.clearUserRestriction(adminName, UserManager.DISALLOW_FACTORY_RESET)
```

Most-used restrictions: `DISALLOW_FACTORY_RESET`, `DISALLOW_SAFE_BOOT`, `DISALLOW_INSTALL_UNKNOWN_SOURCES`, `DISALLOW_ADD_USER`, `DISALLOW_DEBUGGING_FEATURES`, `DISALLOW_CONFIG_VPN`, `DISALLOW_USB_FILE_TRANSFER`.

### Lock Task / Kiosk Mode (API 21+, features API 28+)

```kotlin
dpm.setLockTaskPackages(adminName, arrayOf("com.example.kiosk"))
dpm.setLockTaskFeatures(adminName,
    DevicePolicyManager.LOCK_TASK_FEATURE_SYSTEM_INFO or
    DevicePolicyManager.LOCK_TASK_FEATURE_HOME)
```

### Device Control

| Method | API | Effect |
|--------|-----|--------|
| `reboot(admin)` | 24 | Remote reboot |
| `wipeData(flags)` | 8 | Factory reset |
| `clearDeviceOwnerApp(pkg)` | 21 | Remove device owner status |
| `setKeyguardDisabled(admin, true)` | 23 | Disable lock screen |
| `setStatusBarDisabled(admin, true)` | 23 | Disable status bar |
| `setGlobalSetting(admin, key, value)` | 21 | Modify system settings |
| `setPermissionGrantState(admin, pkg, perm, state)` | 23 | Auto-grant/deny runtime permissions |

### Runtime Permissions (API 23+)

```kotlin
dpm.setPermissionGrantState(adminName, "com.example.app",
    Manifest.permission.CAMERA,
    DevicePolicyManager.PERMISSION_GRANT_STATE_GRANTED)
```

Grant states: `PERMISSION_GRANT_STATE_DEFAULT` (user decides), `PERMISSION_GRANT_STATE_GRANTED` (auto-grant), `PERMISSION_GRANT_STATE_DENIED` (auto-deny).

## API Level Thresholds

| API Level | Android | Key Capabilities Added |
|-----------|---------|----------------------|
| 21 | 5.0 | Device Owner, setApplicationHidden, setUninstallBlocked, lock task |
| 23 | 6.0 | setPermissionGrantState, setKeyguardDisabled, setStatusBarDisabled, system update policy |
| 24 | 7.0 | setPackagesSuspended, setAlwaysOnVpnPackage, reboot, security logging |
| 26 | 8.0 | setDelegatedScopes, password reset token |
| 28 | 9.0 | setLockTaskFeatures, freeze periods, installExistingPackage, DISALLOW_SYSTEM_ERROR_DIALOGS |
| 30 | 11 | setFactoryResetProtectionPolicy |

## Common Patterns

### Protect Your DPC From Removal

```kotlin
dpm.setUninstallBlocked(adminName, context.packageName, true)
dpm.addUserRestriction(adminName, UserManager.DISALLOW_SAFE_BOOT)
dpm.addUserRestriction(adminName, UserManager.DISALLOW_FACTORY_RESET)
dpm.addUserRestriction(adminName, UserManager.DISALLOW_DEBUGGING_FEATURES) // release only
dpm.addUserRestriction(adminName, UserManager.DISALLOW_ADD_USER)
dpm.addUserRestriction(adminName, UserManager.DISALLOW_INSTALL_UNKNOWN_SOURCES)
```

### Exempt From Battery Optimization

DPC apps that enforce policies in the background need to be exempt from Doze battery optimization. The documented approach is to request exemption via an intent:

```kotlin
val pm = context.getSystemService(Context.POWER_SERVICE) as PowerManager
if (!pm.isIgnoringBatteryOptimizations(context.packageName)) {
    val intent = Intent(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS).apply {
        data = Uri.parse("package:${context.packageName}")
    }
    startActivity(intent)
}
```

Check status with `PowerManager.isIgnoringBatteryOptimizations()`. See the [Doze documentation](https://developer.android.com/training/monitoring-device-state/doze-standby) for details on what background work is restricted.

### Hide/Unhide Apps Safely

Per the Android docs, `setApplicationHidden()` makes an app invisible and non-functional while preserving its data. The app cannot run, receive broadcasts, or be started by other apps. Unhiding restores it fully.

```kotlin
// Hide an app
dpm.setApplicationHidden(adminName, "com.example.blocked", true)

// Unhide an app
dpm.setApplicationHidden(adminName, "com.example.blocked", false)

// Check current state
val isHidden = dpm.isApplicationHidden(adminName, "com.example.blocked")
```

**Important:** Do not hide critical system packages (SystemUI, Settings, Phone, keyboard, Google Play Services, Google Services Framework). Hiding these will make the device unusable. Use `enableSystemApp()` to re-enable system apps disabled during provisioning -- it is the only supported way to restore them.

### Managed Configurations for Third-Party Apps

Device owners can remotely configure any app that supports managed configurations via `setApplicationRestrictions()`:

```kotlin
val restrictions = Bundle().apply {
    putString("server_url", "https://mdm.example.com")
    putBoolean("require_auth", true)
}
dpm.setApplicationRestrictions(adminName, "com.example.managedapp", restrictions)
```

The target app reads these via `RestrictionsManager.getApplicationRestrictions()`. This is the official mechanism for configuring third-party apps (e.g., Chrome URL policies, email clients, VPN apps).
