# iOS MDM Restrictions Reference

## Table of Contents

- [Overview](#overview)
- [App Restrictions](#app-restrictions)
- [Safari and Browser](#safari-and-browser)
- [Content and Media](#content-and-media)
- [Communication](#communication)
- [iCloud and Data](#icloud-and-data)
- [Security and Privacy](#security-and-privacy)
- [Device Features](#device-features)
- [AI and Machine Learning](#ai-and-machine-learning)
- [Managed Open In](#managed-open-in)
- [Settings Command Restrictions](#settings-command-restrictions)
- [Key Concepts](#key-concepts)

## Overview

The Restrictions payload (`com.apple.applicationaccess`) controls what users can and cannot do on managed iOS/iPadOS devices. Key facts:

- Over 80 restriction keys require **supervision**
- All keys are **boolean** (`true` = allowed, `false` = restricted) unless noted
- Default value for most keys is `true` (feature allowed)
- Payload can be sent to User or Device channel

## App Restrictions

| Key | Supervised | Notes |
|-----|-----------|-------|
| `allowAppInstallation` | No | Disables App Store and prevents app installation |
| `allowUIAppInstallation` | Yes | Specifically hides the App Store UI |
| `allowAppRemoval` | No | Prevents removing apps from device |
| `allowInAppPurchases` | No | Disables in-app purchases |
| `allowAutomaticAppDownloads` | No | Stops auto-downloading apps purchased on other devices |
| `allowAppCellularDataModification` | No | Prevents changing per-app cellular data settings |
| `allowAppClips` | No | Disables App Clips |
| `allowListedAppBundleIDs` | Yes | **Array of strings.** App allowlist -- blocks all apps not in this list |
| `blockedAppBundleIDs` | Yes | **Array of strings.** App blocklist -- blocks only listed apps |
| `allowRemoveSystemApps` | Yes | Prevents removing built-in Apple apps |
| `allowHideApps` | Yes | iOS 18+. Prevents hiding apps on the Home Screen |
| `allowLockApps` | Yes | iOS 18+. Prevents locking apps behind Face ID/Touch ID |

## Safari and Browser

| Key | Supervised | Notes |
|-----|-----------|-------|
| `allowSafari` | Yes | Hides Safari entirely; requires supervision |
| `safariAllowAutoFill` | No | Disables AutoFill in Safari |
| `safariAllowJavaScript` | No | Disables JavaScript in Safari |
| `safariAllowPopups` | No | Blocks pop-up windows |
| `safariForceWarning` | No | Forces fraudulent website warning |
| `allowAutofillPasswordSharing` | Yes | Prevents sharing passwords via AutoFill |
| `allowWebDistributionAppInstallation` | Yes | Allows installing apps from alternative marketplaces (EU) |
| `allowDefaultBrowserModification` | Yes | iOS 18+. Prevents changing the default browser |

## Content and Media

| Key | Supervised | Notes |
|-----|-----------|-------|
| `allowExplicitContent` | No | Hides explicit music/video content |
| `allowBookstoreErotica` | No | Hides erotica from Book Store |
| `ratingApps` | No | **Integer.** App content rating level (0=none, 1000=all) |
| `ratingMovies` | No | **Integer.** Movie rating level |
| `ratingTVShows` | No | **Integer.** TV show rating level |
| `ratingRegion` | No | **String.** Two-letter country code for ratings (e.g., `us`, `gb`) |
| `allowMusicService` | No/Yes | No: disables Music service. Yes (supervised): fully blocks Music app |
| `allowRadioService` | No | Disables Apple Music Radio |
| `allowBookstore` | Yes | Hides Book Store |
| `allowPodcasts` | Yes | Hides Podcasts app |
| `allowNews` | Yes | Hides News app |
| `allowGameCenter` | Yes | Hides Game Center |
| `allowAddingGameCenterFriends` | No | Prevents adding Game Center friends |
| `allowMultiplayerGaming` | No | Disables multiplayer features |

## Communication

| Key | Supervised | Notes |
|-----|-----------|-------|
| `allowChat` | No/Yes | No: disables iMessage. Yes (supervised): fully disables iMessage |
| `allowRCS` | Yes | iOS 18+. Disables RCS messaging |
| `allowCallRecordingInPhone` | Yes | iOS 18.2+. Disables call recording in Phone app |
| `forceAirDropUnmanaged` | No | Treats AirDrop as unmanaged destination |
| `allowAirDrop` | Yes | Disables AirDrop entirely |
| `allowDefaultMessagingAppModification` | Yes | iOS 18+. Prevents changing default messaging app |
| `allowDefaultCallingAppModification` | Yes | iOS 18+. Prevents changing default calling app |

## iCloud and Data

| Key | Supervised | Notes |
|-----|-----------|-------|
| `allowCloudBackup` | No | Prevents iCloud Backup |
| `allowCloudDocumentSync` | No | Prevents iCloud Drive document sync |
| `allowCloudKeychainSync` | No | Prevents iCloud Keychain sync |
| `allowCloudPhotoLibrary` | No | Prevents iCloud Photo Library |
| `allowSharedStream` | No | Prevents Shared Photo Streams |
| `allowActivityContinuation` | No | Prevents Handoff |
| `forceCloudPrivateRelayDisabled` | Yes | Disables iCloud Private Relay |
| `allowManagedToWriteUnmanagedContacts` | Yes | Controls managed app contact access |
| `allowUnmanagedToReadManagedContacts` | Yes | Controls unmanaged app contact access |

## Security and Privacy

| Key | Supervised | Notes |
|-----|-----------|-------|
| `allowCamera` | No | Disables camera; hides Camera app |
| `allowScreenShot` | No | Prevents screenshots and screen recording |
| `allowRemoteScreenObservation` | No | Prevents AirPlay screen observation in Classroom |
| `forceEncryptedBackup` | No | Forces encrypted backups in Finder/iTunes |
| `allowFingerprintForUnlock` | No | Prevents biometric unlock |
| `allowPasscodeModification` | No/Yes | Supervised: prevents adding/changing/removing passcode |
| `allowDiagnosticSubmission` | No | Prevents sending diagnostics to Apple |
| `forceWiFiPowerOn` | Yes | Prevents turning off Wi-Fi |
| `allowVPNCreation` | Yes | Prevents creating VPN configurations |
| `allowPasswordAutoFill` | Yes | Disables password AutoFill |
| `allowPasswordProximityRequests` | Yes | Prevents requesting passwords from nearby devices |
| `allowPasswordSharing` | Yes | Prevents AirDrop password sharing |
| `allowFingerprintModification` | Yes | Prevents adding/removing biometric enrollments |
| `allowEraseContentAndSettings` | Yes | Prevents factory reset |
| `allowConfigurationProfileInstallation` | Yes | Prevents manual profile installation |
| `allowUSBRestrictedMode` | Yes | Controls USB accessory behavior when locked |
| `forceWiFiWhitelisting` | Yes | Device can only join MDM-managed Wi-Fi networks |

## Device Features

| Key | Supervised | Notes |
|-----|-----------|-------|
| `allowLockScreenNotificationsView` | No | Hides notifications on Lock Screen |
| `allowLockScreenTodayView` | No | Hides Today View on Lock Screen |
| `allowLockScreenControlCenter` | No | Hides Control Center on Lock Screen |
| `allowSpotlightInternetResults` | No | Prevents Spotlight web suggestions |
| `allowPersonalHotspotModification` | No | Prevents changing Personal Hotspot settings |
| `allowDeviceNameModification` | Yes | Prevents changing the device name |
| `allowWallpaperModification` | Yes | Prevents changing wallpaper |
| `allowBluetoothModification` | Yes | Prevents toggling Bluetooth |
| `allowNotificationsModification` | Yes | Prevents changing notification settings |
| `allowCellularPlanModification` | Yes | Prevents modifying cellular plan settings |
| `allowAutoUnlock` | Yes | Prevents Auto Unlock with Apple Watch |
| `forceOnDeviceOnlyDictation` | Yes | Dictation processed on-device only |
| `forceOnDeviceOnlyTranslation` | Yes | Translation processed on-device only |

## AI and Machine Learning

All keys in this section require **supervision** and **iOS 18+** unless noted.

| Key | Min Version | Notes |
|-----|------------|-------|
| `allowGenmoji` | iOS 18.0 | Disables Genmoji creation |
| `allowImagePlayground` | iOS 18.0 | Disables Image Playground |
| `allowImageWand` | iOS 18.0 | Disables Image Wand in Notes |
| `allowWritingTools` | iOS 18.0 | Disables Writing Tools system-wide |
| `allowSafariSummary` | iOS 18.0 | Disables web page summaries in Safari |
| `allowMailSmartReplies` | iOS 18.0 | Disables smart replies in Mail |
| `allowVisualIntelligence` | iOS 18.2 | Disables Visual Intelligence in Camera |
| `allowExternalIntelligence` | iOS 18.2 | Disables ChatGPT integration in Apple Intelligence |
| `allowAppleIntelligence` | iOS 18.4 | Master toggle; disables all Apple Intelligence features |

## Managed Open In

These keys control data flow between managed (work) and unmanaged (personal) apps:

| Key | Default | Effect when `false` |
|-----|---------|-------------------|
| `allowOpenFromManagedToUnmanaged` | `true` | Managed documents can only open in other managed apps |
| `allowOpenFromUnmanagedToManaged` | `true` | Unmanaged documents cannot open in managed apps |
| `forceAirDropUnmanaged` | `false` | When `true`, AirDrop is treated as an unmanaged drop target |

These are the foundation of corporate data loss prevention (DLP). Combine with per-app VPN and managed app configuration for full data containment.

## Settings Command Restrictions

These restrictions are applied via the **Settings MDM command**, not the Restrictions payload:

| Setting | Supervised | Notes |
|---------|-----------|-------|
| `VoiceRoaming` | No | Enable/disable voice roaming |
| `DataRoaming` | No | Enable/disable data roaming |
| `PersonalHotspot` | No | Enable/disable Personal Hotspot |
| `DeviceName` | Yes | Set the device name |
| `Wallpaper` | Yes | Set Lock Screen and Home Screen wallpaper |
| `DiagnosticSubmission` | No | Enable/disable diagnostic data sharing |
| `AppAnalytics` | No | Enable/disable app analytics sharing |
| `MaximumResidentUsers` | Yes | Shared iPad: max cached users |

## Key Concepts

- **Supervision**: Required for the majority of restriction keys. Devices are supervised during enrollment via Apple Business Manager (ABM) or Apple Configurator.
- **Payload scope**: Restrictions payload can target the device or user channel. Device-level restrictions apply regardless of which user is signed in.
- **Combining payloads**: When multiple Restrictions payloads are installed, the most restrictive value wins for each key.
- **Removal**: Removing a Restrictions profile restores the default (allowed) state for all keys it contained.
