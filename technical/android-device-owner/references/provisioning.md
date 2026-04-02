# Provisioning & Setup

## Prerequisites

- Device must be **factory reset** (or in initial setup wizard) -- no user accounts can exist
- `USER_SETUP_COMPLETE` must not be set
- The DPC app must be installed on the device before setting device owner

## Provisioning Methods

### ADB (Development Only)

```bash
# Install the DPC APK
adb install path/to/dpc.apk

# Set as device owner
adb shell dpm set-device-owner com.example.dpc/.receiver.MyDeviceAdminReceiver
```

The component name must match the fully-qualified `DeviceAdminReceiver` class declared in the manifest.

**Common errors:**
- "Not allowed to set the device owner because there are already some accounts on the device" -- factory reset first
- "Not allowed to set the device owner because the user is already set up" -- factory reset first
- "Trying to set the device owner, but device owner is already set" -- already provisioned

### QR Code Provisioning (Production)

Tap the setup wizard screen 6 times to open the QR reader. The QR code encodes a UTF-8 JSON payload:

```json
{
  "android.app.extra.PROVISIONING_DEVICE_ADMIN_COMPONENT_NAME":
    "com.example.dpc/.receiver.MyDeviceAdminReceiver",
  "android.app.extra.PROVISIONING_DEVICE_ADMIN_SIGNATURE_CHECKSUM":
    "gJD2YwtOiWJHkSMkkIfLRlj-quNqG1fb6v100QmzM9w=",
  "android.app.extra.PROVISIONING_DEVICE_ADMIN_PACKAGE_DOWNLOAD_LOCATION":
    "https://example.com/dpc.apk",
  "android.app.extra.PROVISIONING_SKIP_ENCRYPTION": false,
  "android.app.extra.PROVISIONING_WIFI_SSID": "SetupNetwork",
  "android.app.extra.PROVISIONING_WIFI_PASSWORD": "password123",
  "android.app.extra.PROVISIONING_WIFI_SECURITY_TYPE": "WPA",
  "android.app.extra.PROVISIONING_ADMIN_EXTRAS_BUNDLE": {
    "custom_key": "custom_value"
  }
}
```

The signature checksum is a URL-safe base64 SHA-256 hash of the APK signing certificate. Generate it:

```bash
# Extract cert from APK and compute checksum
apksigner verify --print-certs dpc.apk
# Or use the signing keystore
keytool -exportcert -alias key0 -keystore keystore.jks | openssl dgst -sha256 -binary | openssl base64 | tr '+/' '-_' | tr -d '='
```

### NFC Provisioning

Bump two NFC-enabled devices together during setup. The provisioning device sends an NFC intent with the same extras as QR. Android 10+ supports EAP WiFi config and certificates via NFC.

### Zero-Touch Enrollment

OEM partnership-based. Device auto-enrolls on first boot. Configured via the zero-touch portal (partner.android.com/zerotouch). Supports both device owner and profile owner. DPCs set via zero-touch receive secure-hardware-attested device IDs (IMEI, serial number).

### Managed Google Play Accounts

Enter `afw#DPC_IDENTIFIER` in the email field during setup to trigger managed provisioning. Requires the DPC to be published on managed Google Play.

## Provisioning Extras

The DPC receives provisioning extras via `DeviceAdminReceiver.onProfileProvisioningComplete()` intent:

```kotlin
override fun onProfileProvisioningComplete(context: Context, intent: Intent) {
    val extras = intent.getParcelableExtra<PersistableBundle>(
        DevicePolicyManager.EXTRA_PROVISIONING_ADMIN_EXTRAS_BUNDLE)
    // Extract custom configuration from extras
    val serverUrl = extras?.getString("server_url")
}
```

## Removing Device Owner

### Programmatic (From Within DPC)

```kotlin
// Clear all enforcement first
clearAllRestrictions()
unhideAllApps()
stopVpn()

// Then remove device owner status
dpm.clearDeviceOwnerApp(context.packageName)
```

**Important:** Clear all restrictions and unhide all apps BEFORE calling `clearDeviceOwnerApp()`. Once device owner status is removed, you lose the ability to undo policy changes.

### Via ADB (Development)

```bash
adb shell dpm remove-active-admin com.example.dpc/.receiver.MyDeviceAdminReceiver
```

### Factory Reset

```kotlin
dpm.wipeData(0) // Factory reset, removes device owner
// With flags:
dpm.wipeData(DevicePolicyManager.WIPE_EXTERNAL_STORAGE or
    DevicePolicyManager.WIPE_RESET_PROTECTION_DATA)
```

## TestDPC

Google's reference DPC for development and testing:
- GitHub: `github.com/googlesamples/android-testdpc`
- Available on Google Play
- Supports all device owner and profile owner features
- Useful for testing policy behavior before implementing in your own DPC
