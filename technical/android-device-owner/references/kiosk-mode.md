# Kiosk Mode / Lock Task Mode

Lock task mode pins an app (or set of apps) to the screen, preventing the user from navigating away. Combined with device owner restrictions, it creates a full kiosk experience.

## Setup Overview

1. Set device owner
2. Allowlist kiosk app(s) via `setLockTaskPackages()`
3. Configure UI features via `setLockTaskFeatures()` (API 28+)
4. Set your app as the persistent home activity
5. Start lock task mode
6. Apply lockdown user restrictions

## Allowlisting Apps (API 21+)

```kotlin
// Allowlist apps that can run in lock task mode
dpm.setLockTaskPackages(adminName, arrayOf(
    "com.example.kiosk",
    "com.android.settings",  // Optional: allow settings access
    "com.android.chrome"     // Optional: allow browser
))

// Query allowlisted packages
val packages = dpm.getLockTaskPackages(adminName)

// Check if a specific app can enter lock task
val permitted = dpm.isLockTaskPermitted("com.example.kiosk")
```

## Configuring UI Features (API 28+)

```kotlin
dpm.setLockTaskFeatures(adminName,
    DevicePolicyManager.LOCK_TASK_FEATURE_SYSTEM_INFO or    // Status bar info
    DevicePolicyManager.LOCK_TASK_FEATURE_HOME or           // Home button
    DevicePolicyManager.LOCK_TASK_FEATURE_GLOBAL_ACTIONS    // Power button dialog
)
```

**Feature flags:**

| Flag | Effect | Notes |
|------|--------|-------|
| `LOCK_TASK_FEATURE_NONE` | Disable all UI features | Most restrictive |
| `LOCK_TASK_FEATURE_SYSTEM_INFO` | Status bar: connectivity, battery, sound | |
| `LOCK_TASK_FEATURE_HOME` | Show home button | Required for NOTIFICATIONS and OVERVIEW |
| `LOCK_TASK_FEATURE_OVERVIEW` | Show recents/overview button | Requires HOME |
| `LOCK_TASK_FEATURE_NOTIFICATIONS` | Notification shade and heads-up | Requires HOME |
| `LOCK_TASK_FEATURE_GLOBAL_ACTIONS` | Power button long-press dialog | **Default ON** |
| `LOCK_TASK_FEATURE_KEYGUARD` | Enable lock screen | |

Features persist across lock task sessions and apply immediately if already in lock task mode.

## Starting Lock Task Mode

### Method 1: Manifest attribute (Recommended)

Set on the activity that should auto-enter lock task:

```xml
<activity android:name=".KioskActivity"
    android:lockTaskMode="if_whitelisted" />
```

Values:
- `normal` -- default, never enters lock task
- `if_whitelisted` -- enters lock task automatically when launched, if allowlisted
- `always` -- always enters lock task (only for device owner apps)
- `never` -- never enters lock task, even if allowlisted

### Method 2: ActivityOptions (API 28+)

Launch any allowlisted app into lock task mode from your DPC:

```kotlin
val options = ActivityOptions.makeBasic()
options.setLockTaskEnabled(true)
val intent = Intent(context, KioskActivity::class.java)
context.startActivity(intent, options.toBundle())
```

### Method 3: Activity.startLockTask() (Pre-API 28)

Call from the foreground activity:

```kotlin
override fun onResume() {
    super.onResume()
    if (dpm.isLockTaskPermitted(packageName)) {
        startLockTask()
    }
}
```

## Stopping Lock Task Mode

```kotlin
// Method 1: Remove from allowlist (API 23+)
dpm.setLockTaskPackages(adminName, emptyArray())

// Method 2: From the activity itself
activity.stopLockTask()
```

## Setting as Home App

Make your kiosk app the default home/launcher:

```kotlin
val filter = IntentFilter(Intent.ACTION_MAIN).apply {
    addCategory(Intent.CATEGORY_HOME)
    addCategory(Intent.CATEGORY_DEFAULT)
}
val activity = ComponentName(context, KioskActivity::class.java)
dpm.addPersistentPreferredActivity(adminName, filter, activity)
```

This overrides the user's launcher choice. The device always returns to your kiosk app when pressing Home.

## Lockdown Restrictions for Kiosk

Apply all of these for a secure kiosk:

```kotlin
val kioskRestrictions = arrayOf(
    UserManager.DISALLOW_FACTORY_RESET,
    UserManager.DISALLOW_SAFE_BOOT,
    UserManager.DISALLOW_MOUNT_PHYSICAL_MEDIA,
    UserManager.DISALLOW_ADD_USER,
    UserManager.DISALLOW_INSTALL_UNKNOWN_SOURCES,
    UserManager.DISALLOW_SYSTEM_ERROR_DIALOGS,  // API 28+ -- suppress crash/ANR dialogs
    UserManager.DISALLOW_CREATE_WINDOWS,         // Block overlays during lock task
)
kioskRestrictions.forEach { dpm.addUserRestriction(adminName, it) }
```

## Screen Management

```kotlin
// Stay on while plugged in
val pluggedInto = BatteryManager.BATTERY_PLUGGED_AC or
    BatteryManager.BATTERY_PLUGGED_USB or
    BatteryManager.BATTERY_PLUGGED_WIRELESS
dpm.setGlobalSetting(adminName,
    Settings.Global.STAY_ON_WHILE_PLUGGED_IN,
    pluggedInto.toString())

// Disable keyguard (lock screen)
dpm.setKeyguardDisabled(adminName, true)  // API 23+

// Disable status bar (prevents pulling down notification shade)
dpm.setStatusBarDisabled(adminName, true)  // API 23+

// Disable screenshots
dpm.setScreenCaptureDisabled(adminName, true)
```

## Handling App Crashes in Kiosk Mode

1. **Suppress crash dialogs:** `DISALLOW_SYSTEM_ERROR_DIALOGS` (API 28+)
2. **Detect lock task exit:** Override `DeviceAdminReceiver.onLockTaskModeExiting()`:

```kotlin
override fun onLockTaskModeExiting(context: Context, intent: Intent) {
    // Re-enter lock task mode or restart kiosk app
    val launchIntent = Intent(context, KioskActivity::class.java).apply {
        addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
    }
    context.startActivity(launchIntent)
}
```

3. **Persistent home activity:** `addPersistentPreferredActivity()` ensures the device always returns to your app.
4. **Block overlays:** `DISALLOW_CREATE_WINDOWS` prevents background services from creating windows that could overlay your kiosk UI.

## Complete Kiosk Setup Checklist

```kotlin
fun setupKioskMode() {
    // 1. Allowlist kiosk app
    dpm.setLockTaskPackages(adminName, arrayOf(context.packageName))

    // 2. Configure lock task UI
    dpm.setLockTaskFeatures(adminName,
        DevicePolicyManager.LOCK_TASK_FEATURE_SYSTEM_INFO or
        DevicePolicyManager.LOCK_TASK_FEATURE_GLOBAL_ACTIONS)

    // 3. Set as home app
    val filter = IntentFilter(Intent.ACTION_MAIN).apply {
        addCategory(Intent.CATEGORY_HOME)
        addCategory(Intent.CATEGORY_DEFAULT)
    }
    dpm.addPersistentPreferredActivity(adminName, filter,
        ComponentName(context, KioskActivity::class.java))

    // 4. Apply restrictions
    arrayOf(
        UserManager.DISALLOW_FACTORY_RESET,
        UserManager.DISALLOW_SAFE_BOOT,
        UserManager.DISALLOW_ADD_USER,
        UserManager.DISALLOW_MOUNT_PHYSICAL_MEDIA,
        UserManager.DISALLOW_INSTALL_UNKNOWN_SOURCES,
        UserManager.DISALLOW_SYSTEM_ERROR_DIALOGS,
    ).forEach { dpm.addUserRestriction(adminName, it) }

    // 5. Screen settings
    dpm.setKeyguardDisabled(adminName, true)
    dpm.setGlobalSetting(adminName,
        Settings.Global.STAY_ON_WHILE_PLUGGED_IN,
        (BatteryManager.BATTERY_PLUGGED_AC or BatteryManager.BATTERY_PLUGGED_USB).toString())

    // 6. Start lock task (if not using manifest attribute)
    // activity.startLockTask()
}
```
