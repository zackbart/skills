# User Restrictions (DISALLOW_* Constants)

Device owners can set and clear user restrictions via:

```kotlin
dpm.addUserRestriction(adminName, UserManager.DISALLOW_INSTALL_UNKNOWN_SOURCES)
dpm.clearUserRestriction(adminName, UserManager.DISALLOW_FACTORY_RESET)
```

## App Installation & Management

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_INSTALL_APPS` | Block all app installation |
| `DISALLOW_UNINSTALL_APPS` | Block all app uninstallation |
| `DISALLOW_INSTALL_UNKNOWN_SOURCES` | Block sideloading (per-user) |
| `DISALLOW_INSTALL_UNKNOWN_SOURCES_GLOBALLY` | Block sideloading (device-wide) |
| `DISALLOW_APPS_CONTROL` | Hide "Force stop" and "Clear data" in Settings |

## Network & Connectivity

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_CONFIG_WIFI` | Block WiFi settings changes |
| `DISALLOW_CHANGE_WIFI_STATE` | Block programmatic WiFi changes |
| `DISALLOW_WIFI_TETHERING` | Block WiFi hotspot |
| `DISALLOW_WIFI_DIRECT` | Block WiFi Direct |
| `DISALLOW_ADD_WIFI_CONFIG` | Block adding new WiFi networks |
| `DISALLOW_SHARING_ADMIN_CONFIGURED_WIFI` | Block sharing admin WiFi via QR/NFC |
| `DISALLOW_CONFIG_VPN` | Block user VPN configuration |
| `DISALLOW_CONFIG_MOBILE_NETWORKS` | Block mobile network settings |
| `DISALLOW_CONFIG_TETHERING` | Block all tethering settings |
| `DISALLOW_NETWORK_RESET` | Block network settings reset |
| `DISALLOW_CONFIG_PRIVATE_DNS` | Block Private DNS (DoH/DoT) settings |
| `DISALLOW_DATA_ROAMING` | Disable data roaming (device owner only) |

## Bluetooth

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_CONFIG_BLUETOOTH` | Block Bluetooth settings |
| `DISALLOW_BLUETOOTH` | Disable Bluetooth entirely |
| `DISALLOW_BLUETOOTH_SHARING` | Block Bluetooth file sharing |

## Hardware & Peripherals

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_MOUNT_PHYSICAL_MEDIA` | Block external storage (USB, SD card) |
| `DISALLOW_USB_FILE_TRANSFER` | Block USB file transfer |
| `DISALLOW_ADJUST_VOLUME` | Lock volume settings |
| `DISALLOW_UNMUTE_MICROPHONE` | Keep microphone muted |
| `DISALLOW_UNMUTE_DEVICE` | Keep device muted |
| `DISALLOW_CAMERA` | Disable camera |
| `DISALLOW_MICROPHONE_TOGGLE` | Block microphone privacy toggle |
| `DISALLOW_CAMERA_TOGGLE` | Block camera privacy toggle |
| `DISALLOW_BIOMETRIC` | Block biometric enrollment |

## Radio & Wireless

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_AIRPLANE_MODE` | Block airplane mode |
| `DISALLOW_CELLULAR_2G` | Block 2G cellular connections |
| `DISALLOW_ULTRA_WIDEBAND_RADIO` | Block UWB radio |
| `DISALLOW_NEAR_FIELD_COMMUNICATION_RADIO` | Block NFC radio |
| `DISALLOW_CHANGE_NEAR_FIELD_COMMUNICATION_RADIO` | Block NFC toggle |
| `DISALLOW_THREAD_NETWORK` | Block Thread network |
| `DISALLOW_SIM_GLOBALLY` | Block SIM management device-wide |

## Display & Appearance

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_CONFIG_BRIGHTNESS` | Block brightness changes |
| `DISALLOW_AMBIENT_DISPLAY` | Block ambient display |
| `DISALLOW_CONFIG_SCREEN_TIMEOUT` | Block screen timeout changes |
| `DISALLOW_WALLPAPER` | Block wallpaper changes |
| `DISALLOW_SET_WALLPAPER` | Same as above (alias) |
| `DISALLOW_SET_USER_ICON` | Block user icon changes |

## Security & System

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_CONFIG_CREDENTIALS` | Block credential settings |
| `DISALLOW_DEBUGGING_FEATURES` | Disable ADB debugging |
| `DISALLOW_FACTORY_RESET` | Block factory reset |
| `DISALLOW_SAFE_BOOT` | Block safe mode boot |
| `DISALLOW_OEM_UNLOCK` | Block OEM unlock |
| `DISALLOW_UNIFIED_PASSWORD` | Require separate work password |
| `DISALLOW_CONFIG_DATE_TIME` | Block date/time changes |
| `DISALLOW_CONFIG_LOCALE` | Block locale changes |
| `DISALLOW_CONFIG_DEFAULT_APPS` | Block default app changes |
| `DISALLOW_SYSTEM_ERROR_DIALOGS` | Suppress crash/ANR dialogs (API 28+) |
| `DISALLOW_CREATE_WINDOWS` | Block overlay windows/popups |
| `DISALLOW_FUN` | Block easter eggs |

## Users & Profiles

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_ADD_USER` | Block adding new users |
| `DISALLOW_REMOVE_USER` | Block removing users |
| `DISALLOW_REMOVE_MANAGED_PROFILE` | Block work profile removal |
| `DISALLOW_ADD_MANAGED_PROFILE` | Block adding work profiles |
| `DISALLOW_ADD_CLONE_PROFILE` | Block clone profiles |
| `DISALLOW_ADD_PRIVATE_PROFILE` | Block private profiles |
| `DISALLOW_USER_SWITCH` | Block user switching |
| `DISALLOW_GRANT_ADMIN` | Block granting admin to users |

## Data & Privacy

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_SHARE_LOCATION` | Disable location sharing |
| `DISALLOW_CONFIG_LOCATION` | Block location settings |
| `DISALLOW_CROSS_PROFILE_COPY_PASTE` | Block clipboard across profiles |
| `DISALLOW_SHARE_INTO_MANAGED_PROFILE` | Block sharing into work profile |
| `DISALLOW_OUTGOING_BEAM` | Block NFC beam |
| `DISALLOW_AUTOFILL` | Disable autofill |
| `DISALLOW_CONTENT_CAPTURE` | Disable content capture |
| `DISALLOW_CONTENT_SUGGESTIONS` | Disable content suggestions |
| `DISALLOW_ASSIST_CONTENT` | Disable assistant context |

## Communication

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_OUTGOING_CALLS` | Block outgoing calls |
| `DISALLOW_SMS` | Block SMS |
| `DISALLOW_CONFIG_CELL_BROADCASTS` | Block cell broadcast settings |

## Other

| Restriction | Effect |
|-------------|--------|
| `DISALLOW_MODIFY_ACCOUNTS` | Block adding/removing accounts |
| `DISALLOW_RECORD_AUDIO` | Block audio recording |
| `DISALLOW_RUN_IN_BACKGROUND` | Kill background processes |
| `DISALLOW_PRINTING` | Block printing |

## Documented Restriction Patterns

### Dedicated Device / Kiosk (from Android Dedicated Devices Cookbook)

The [Android dedicated devices cookbook](https://developer.android.com/work/dpc/dedicated-devices/cookbook) documents these restrictions for dedicated/kiosk devices:

```kotlin
// From developer.android.com/work/dpc/dedicated-devices/cookbook
arrayOf(
    UserManager.DISALLOW_FACTORY_RESET,
    UserManager.DISALLOW_SAFE_BOOT,
    UserManager.DISALLOW_ADD_USER,
    UserManager.DISALLOW_MOUNT_PHYSICAL_MEDIA,
    UserManager.DISALLOW_ADJUST_VOLUME,
).forEach { dpm.addUserRestriction(adminName, it) }
```

Additional restrictions to consider per the cookbook: `DISALLOW_SYSTEM_ERROR_DIALOGS` (API 28+) for suppressing crash dialogs, and `DISALLOW_CREATE_WINDOWS` to block overlay windows.

### Fully Managed Device (from Android Enterprise Documentation)

The [fully managed device documentation](https://developer.android.com/work/dpc/device-management) describes these as common restrictions for company-owned devices:

```kotlin
arrayOf(
    UserManager.DISALLOW_INSTALL_UNKNOWN_SOURCES,
    UserManager.DISALLOW_FACTORY_RESET,
    UserManager.DISALLOW_ADD_USER,
    UserManager.DISALLOW_DEBUGGING_FEATURES,
).forEach { dpm.addUserRestriction(adminName, it) }
```

Select additional restrictions based on your specific device management requirements. The full list above is organized by category to help identify relevant restrictions for your use case.
