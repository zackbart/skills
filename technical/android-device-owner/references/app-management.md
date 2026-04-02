# App Management

## setApplicationHidden -- Hide Apps Completely (API 21+)

Makes an app invisible and completely non-functional. The app disappears from the launcher, cannot run, cannot receive broadcasts, and cannot be started by other apps. App data and APK are preserved on disk. Unhiding restores the app to its previous state.

```kotlin
// Hide an app
dpm.setApplicationHidden(adminName, "com.example.blocked", true)

// Unhide an app
dpm.setApplicationHidden(adminName, "com.example.blocked", false)

// Check status
val isHidden = dpm.isApplicationHidden(adminName, "com.example.blocked")
```

**Callers:** Device owner, profile owner, or delegate with `DELEGATION_PACKAGE_ACCESS`

**Behavior details:**
- Hidden apps remain installed -- `PackageManager.getInstalledPackages()` still lists them
- Hidden apps cannot start activities, services, or receive broadcasts
- Data is fully preserved; unhiding restores the app completely
- Hiding an app that is currently in the foreground force-stops it
- System apps can be hidden, but hiding critical system packages (SystemUI, Settings, Phone, keyboard, Google Play Services) will make the device unusable -- always safeguard these

## setPackagesSuspended -- Gray Out Apps (API 24+)

Suspended apps remain visible in the launcher but are grayed out. Attempting to launch shows a system dialog. Notifications are suppressed. The user can see the app exists but cannot interact with it.

```kotlin
// Suspend apps (returns array of packages that couldn't be suspended)
val failed = dpm.setPackagesSuspended(adminName,
    arrayOf("com.example.app1", "com.example.app2"), true)

// Unsuspend
dpm.setPackagesSuspended(adminName, arrayOf("com.example.app1"), false)

// Check status
val isSuspended = dpm.isPackageSuspended(adminName, "com.example.app1")
```

**Callers:** Device owner (API 26+), profile owner (API 24+), or delegate with `DELEGATION_PACKAGE_ACCESS`

**Key differences from hidden:**

| Aspect | Hidden | Suspended |
|--------|--------|-----------|
| Visible in launcher | No | Yes (grayed out) |
| Can launch | No | No (shows dialog) |
| Notifications | Suppressed | Suppressed |
| User awareness | App seems gone | User sees it's restricted |
| API level | 21+ | 24+ |

**When to use which:**
- **Hidden** for apps the user should not know about or that should be completely invisible (parental controls, managed deployments)
- **Suspended** for temporary restrictions where the user should see the app exists but is currently unavailable (time-based controls, compliance enforcement)

## setUninstallBlocked -- Prevent Removal (API 21+)

```kotlin
dpm.setUninstallBlocked(adminName, "com.example.required", true)

// Check status
val isBlocked = dpm.isUninstallBlocked(adminName, "com.example.required")
```

**Callers:** Device owner, profile owner, or delegate with `DELEGATION_BLOCK_UNINSTALL`

The uninstall option is removed from Settings and the launcher context menu. Essential for protecting your DPC app and mandatory managed apps.

## enableSystemApp -- Restore Disabled System Apps (API 21+)

During managed provisioning, many system apps are disabled by default. Use this to re-enable them.

```kotlin
// By package name
dpm.enableSystemApp(adminName, "com.android.calculator2")

// By intent -- re-enable all apps that handle a specific intent
val count = dpm.enableSystemApp(adminName, Intent(Intent.ACTION_VIEW).apply {
    addCategory(Intent.CATEGORY_BROWSABLE)
    data = Uri.parse("https://example.com")
})
// Returns count of apps enabled
```

**Callers:** Device owner, profile owner, or delegate with `DELEGATION_ENABLE_SYSTEM_APP`

## Silent App Installation via PackageInstaller (API 23+)

Device owners bypass the user confirmation dialog entirely. The install proceeds silently.

```kotlin
val packageInstaller = context.packageManager.packageInstaller
val params = PackageInstaller.SessionParams(
    PackageInstaller.SessionParams.MODE_FULL_INSTALL)
val sessionId = packageInstaller.createSession(params)
val session = packageInstaller.openSession(sessionId)

// Write APK to session
val output = session.openWrite("apk", 0, -1)
apkInputStream.use { input -> input.copyTo(output, 2048) }
session.fsync(output)
output.close()

// Commit -- no user prompt for device owner
val intent = Intent(context, InstallStatusReceiver::class.java)
val pendingIntent = PendingIntent.getBroadcast(context, sessionId, intent,
    PendingIntent.FLAG_MUTABLE)
session.commit(pendingIntent.intentSender)
```

Handle the result:

```kotlin
class InstallStatusReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.getIntExtra(PackageInstaller.EXTRA_STATUS, -1)) {
            PackageInstaller.STATUS_SUCCESS -> { /* installed */ }
            PackageInstaller.STATUS_FAILURE -> {
                val msg = intent.getStringExtra(PackageInstaller.EXTRA_STATUS_MESSAGE)
            }
        }
    }
}
```

**Silent uninstall:**
```kotlin
packageInstaller.uninstall("com.example.app", statusReceiver)
```

## Cached APK Management (API 28+)

Keep APKs cached for quick reinstallation:

```kotlin
// Mark packages to keep cached after uninstall
dpm.setKeepUninstalledPackages(adminName, listOf("com.example.app"))

// Reinstall from cache
dpm.installExistingPackage(adminName, "com.example.app")

// Clear cache list
dpm.setKeepUninstalledPackages(adminName, emptyList())
```

## Delegation (API 26+)

Delegate specific DPM capabilities to other apps without giving them full device owner:

```kotlin
dpm.setDelegatedScopes(adminName, "com.example.helper", listOf(
    DevicePolicyManager.DELEGATION_PACKAGE_ACCESS,    // hide/suspend apps
    DevicePolicyManager.DELEGATION_BLOCK_UNINSTALL,   // block uninstall
    DevicePolicyManager.DELEGATION_PERMISSION_GRANT,  // grant permissions
))
```

**Available delegation scopes:**

| Scope | Capability |
|-------|-----------|
| `DELEGATION_CERT_INSTALL` | Install/manage certificates |
| `DELEGATION_APP_RESTRICTIONS` | Set managed configurations |
| `DELEGATION_BLOCK_UNINSTALL` | Block app uninstall |
| `DELEGATION_PERMISSION_GRANT` | Grant/deny runtime permissions |
| `DELEGATION_PACKAGE_ACCESS` | Hide/suspend/query packages |
| `DELEGATION_ENABLE_SYSTEM_APP` | Re-enable disabled system apps |
| `DELEGATION_KEEP_UNINSTALLED_PACKAGES` | Manage cached APKs |
| `DELEGATION_NETWORK_LOGGING` | Access network activity logs |
| `DELEGATION_SECURITY_LOGGING` | Access security logs |

The delegate app receives `ACTION_APPLICATION_DELEGATION_SCOPES_CHANGED` broadcast when scopes change.

## Querying Package States

```kotlin
// All installed packages
val packages = context.packageManager.getInstalledPackages(0)

// Check individual states
val hidden = dpm.isApplicationHidden(adminName, pkg)
val suspended = dpm.isPackageSuspended(adminName, pkg)
val uninstallBlocked = dpm.isUninstallBlocked(adminName, pkg)
```

## Managed Google Play

Control Play Store access via managed configurations on `com.android.vending`:

```kotlin
val restrictions = Bundle().apply {
    putString("allowed_accounts", "user@company.com")
    putBoolean("verify_apps:device_wide_unknown_source_block", true)
}
dpm.setApplicationRestrictions(adminName, "com.android.vending", restrictions)
```

Combine with `DISALLOW_INSTALL_UNKNOWN_SOURCES` user restriction to fully control app sources.
