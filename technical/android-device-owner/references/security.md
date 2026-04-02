# Security, FRP & Device Control

## Factory Reset Protection (API 30+)

Control which Google Accounts can provision the device after a factory reset:

```kotlin
val policy = FactoryResetProtectionPolicy.Builder()
    .setFactoryResetProtectionAccounts(listOf("admin@company.com"))
    .setFactoryResetProtectionEnabled(true)
    .build()
dpm.setFactoryResetProtectionPolicy(adminName, policy)

// Notify GMS of the change
val intent = Intent("com.google.android.gms.auth.FRP_CONFIG_CHANGED")
intent.setPackage("com.google.android.gms")
context.sendBroadcast(intent)
```

Pass `null` to disable FRP (falls back to accounts on the personal profile).

## Remote Wipe / Factory Reset

```kotlin
// Basic factory reset
dpm.wipeData(0)

// With options
dpm.wipeData(
    DevicePolicyManager.WIPE_EXTERNAL_STORAGE or  // Also wipe SD card
    DevicePolicyManager.WIPE_RESET_PROTECTION_DATA or  // Clear FRP data
    DevicePolicyManager.WIPE_EUICC or  // Erase eSIM profile
    DevicePolicyManager.WIPE_SILENTLY  // No user warning
)

// With reason (API 28+)
dpm.wipeData(0, "Remote wipe requested by admin")
```

**Flags:**

| Flag | Effect |
|------|--------|
| `WIPE_EXTERNAL_STORAGE` | Also wipe external/SD storage |
| `WIPE_RESET_PROTECTION_DATA` | Clear FRP data (device owner only) |
| `WIPE_EUICC` | Erase eSIM profile |
| `WIPE_SILENTLY` | No user-facing warning dialog |

## Runtime Permissions (API 23+)

Silently grant or deny runtime permissions for any app:

```kotlin
// Auto-grant camera permission
dpm.setPermissionGrantState(adminName, "com.example.app",
    Manifest.permission.CAMERA,
    DevicePolicyManager.PERMISSION_GRANT_STATE_GRANTED)

// Auto-deny location
dpm.setPermissionGrantState(adminName, "com.example.app",
    Manifest.permission.ACCESS_FINE_LOCATION,
    DevicePolicyManager.PERMISSION_GRANT_STATE_DENIED)

// Reset to user-managed
dpm.setPermissionGrantState(adminName, "com.example.app",
    Manifest.permission.CAMERA,
    DevicePolicyManager.PERMISSION_GRANT_STATE_DEFAULT)

// Check current state
val state = dpm.getPermissionGrantState(adminName, "com.example.app",
    Manifest.permission.CAMERA)
```

**Grant states:**
- `PERMISSION_GRANT_STATE_DEFAULT` (0) -- user manages via UI
- `PERMISSION_GRANT_STATE_GRANTED` (1) -- auto-granted, user cannot revoke
- `PERMISSION_GRANT_STATE_DENIED` (2) -- auto-denied, user cannot grant

**Callers:** Device owner, profile owner, or delegate with `DELEGATION_PERMISSION_GRANT`

## Battery Optimization Exemption

Ensure your DPC app runs continuously without being killed by Doze/battery optimization:

```kotlin
val pm = context.getSystemService(Context.POWER_SERVICE) as PowerManager
if (!pm.isIgnoringBatteryOptimizations(context.packageName)) {
    // Request exemption via intent
    val intent = Intent(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS).apply {
        data = Uri.parse("package:${context.packageName}")
    }
    startActivity(intent)
}
```

The `ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` intent is the documented approach per [Doze and App Standby](https://developer.android.com/training/monitoring-device-state/doze-standby). Note that OEM-specific battery management features may apply additional restrictions beyond stock Android Doze -- test on target hardware.

## Security Logging (API 24+)

```kotlin
// Enable security logging
dpm.setSecurityLoggingEnabled(adminName, true)

// Retrieve logs (callback-based)
override fun onSecurityLogsAvailable(context: Context, intent: Intent) {
    val logs = dpm.retrieveSecurityLogs(adminName)
    logs?.forEach { event ->
        val tag = event.tag  // e.g., SecurityLog.TAG_ADB_SHELL_CMD
        val data = event.data
        val timestamp = event.timeNanos
    }
}
```

Log event tags include: `TAG_ADB_SHELL_CMD`, `TAG_ADB_SHELL_INTERACTIVE`, `TAG_APP_PROCESS_START`, `TAG_KEYGUARD_DISMISSED`, `TAG_KEYGUARD_SECURED`, `TAG_KEY_GENERATED`, `TAG_CERT_AUTHORITY_INSTALLED`, etc.

## Network Logging (API 26+)

```kotlin
dpm.setNetworkLoggingEnabled(adminName, true)

override fun onNetworkLogsAvailable(context: Context, intent: Intent,
    batchToken: Long, networkLogsCount: Int) {
    val events = dpm.retrieveNetworkLogs(adminName, batchToken)
    events?.forEach { event ->
        when (event) {
            is DnsEvent -> {
                val hostname = event.hostname
                val addresses = event.inetAddresses
                val packageName = event.packageName
            }
            is ConnectEvent -> {
                val ipAddress = event.inetAddress
                val port = event.port
                val packageName = event.packageName
            }
        }
    }
}
```

## Certificate Management (API 21+)

Install client certificates for enterprise WiFi/VPN:

```kotlin
// Install a certificate + private key
dpm.installKeyPair(adminName, privateKey, certChain, "my-cert-alias",
    DevicePolicyManager.INSTALLKEY_SET_USER_SELECTABLE)

// Grant an app access to the certificate
dpm.grantKeyPairToApp(adminName, "my-cert-alias", "com.example.app")

// Install a CA certificate (trusted by all apps)
dpm.installCaCert(adminName, caCertBytes)

// Remove a CA certificate
dpm.uninstallCaCert(adminName, caCertBytes)
```

**Callers:** Device owner, profile owner, or delegate with `DELEGATION_CERT_INSTALL`

## Password Management

### Password Reset Token (API 26+)

```kotlin
// Set a token (must be done before user sets a password)
val token = ByteArray(32).also { SecureRandom().nextBytes(it) }
dpm.setResetPasswordToken(adminName, token)

// Check if token is active
val isActive = dpm.isResetPasswordTokenActive(adminName)

// Reset password using token
dpm.resetPasswordWithToken(adminName, "new-pin-1234", token, 0)

// Clear the token
dpm.clearResetPasswordToken(adminName)
```

### Lock Screen Message (API 24+)

```kotlin
dpm.setDeviceOwnerLockScreenInfo(adminName,
    "This device is managed by Acme Corp.\nIf found, call 555-0100.")
```

## Remote Reboot (API 24+)

```kotlin
// Only works when phone is idle (not in a call)
dpm.reboot(adminName)
```

Throws `IllegalStateException` if the device is in a call (`CALL_STATE_IDLE` required).

## Bug Report (API 24+)

```kotlin
dpm.requestBugreport(adminName)

// Handle in DeviceAdminReceiver
override fun onBugreportShared(context: Context, intent: Intent, bugreportHash: String) {
    val bugreportUri = intent.data  // Content URI to the bug report
}
override fun onBugreportFailed(context: Context, intent: Intent, failureCode: Int) {
    // BUGREPORT_FAILURE_INCOMPLETE or BUGREPORT_FAILURE_FILE_NO_LONGER_AVAILABLE
}
override fun onBugreportSharingDeclined(context: Context, intent: Intent) {
    // User declined to share
}
```

## System Update Policy (API 23+)

```kotlin
// Auto-install immediately
dpm.setSystemUpdatePolicy(adminName,
    SystemUpdatePolicy.createAutomaticInstallPolicy())

// Install during maintenance window (2am-4am)
dpm.setSystemUpdatePolicy(adminName,
    SystemUpdatePolicy.createWindowedInstallPolicy(120, 240))  // minutes from midnight

// Postpone up to 30 days
dpm.setSystemUpdatePolicy(adminName,
    SystemUpdatePolicy.createPostponeInstallPolicy())

// Add freeze period (API 28+) -- no updates during this window
val policy = SystemUpdatePolicy.createAutomaticInstallPolicy()
policy.freezePeriods = listOf(
    FreezePeriod(MonthDay.of(12, 20), MonthDay.of(1, 5))  // Holiday freeze
)
dpm.setSystemUpdatePolicy(adminName, policy)
```

**Freeze period constraints:**
- Maximum 90 days per period
- Minimum 60-day gap between periods
- No overlapping periods
- Uses `MonthDay` for annual recurrence

## Always-On VPN (API 24+)

```kotlin
// Force all traffic through your VPN app
dpm.setAlwaysOnVpnPackage(adminName, "com.example.vpn", true)
// lockdown=true means NO traffic leaks if VPN disconnects

// Check current always-on VPN
val vpnPackage = dpm.getAlwaysOnVpnPackage(adminName)

// Prevent user from changing VPN
dpm.addUserRestriction(adminName, UserManager.DISALLOW_CONFIG_VPN)
```

## Global Settings

Device owners can modify system-wide settings:

```kotlin
// Disable Private DNS (prevent DoH/DoT bypass)
dpm.setGlobalSetting(adminName, "private_dns_mode", "off")

// Keep screen on while charging
dpm.setGlobalSetting(adminName, Settings.Global.STAY_ON_WHILE_PLUGGED_IN,
    (BatteryManager.BATTERY_PLUGGED_AC or BatteryManager.BATTERY_PLUGGED_USB).toString())

// Set custom lock screen message
dpm.setDeviceOwnerLockScreenInfo(adminName, "Managed device - contact IT")
```

**Note:** `setGlobalSetting()` can only modify settings that are explicitly documented as writable by device owners. Not all `Settings.Global` values are writable.
