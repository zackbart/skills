# Sparkle Customization — Info.plist Settings & Runtime APIs

## General Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `SUFeedURL` | String | — | URL of your appcast (e.g. `https://example.com/appcast.xml`). Set in Info.plist even if changed later programmatically. |
| `SUPublicEDKey` | String | — | Base64-encoded public EdDSA key from `generate_keys` tool. |
| `SUEnableAutomaticChecks` | Boolean | (unset) | When unset: user is prompted on second launch. `YES`: enables auto-checks without prompting. `NO`: disables without prompting. Override at runtime via `automaticallyChecksForUpdates`. |
| `SUScheduledCheckInterval` | Number | 86400 (1 day) | Seconds between auto-checks. Minimum: 3600 (1 hour). |
| `SUAutomaticallyUpdate` | Boolean | NO | Auto-download and silently install updates. Updates needing authorization won't auto-install. Override at runtime via `automaticallyDownloadsUpdates`. |
| `SUScheduledImpatientCheckInterval` | Number | 604800 (1 week) | After a background update is ready, seconds before prompting user to install. Must be > `SUScheduledCheckInterval`. (Sparkle 2.9+) |
| `SUAllowsAutomaticUpdates` | Boolean | (unset) | Controls whether users see the auto-update option. `NO`: disallows auto-updates entirely. `YES`: allows even if auto-checking is disabled. |
| `SUEnableSystemProfiling` | Boolean | NO | Enable anonymous system profiling. See [System Profiling docs](https://sparkle-project.org/documentation/system-profiling). |
| `SUShowReleaseNotes` | Boolean | YES | Set to `NO` to hide release notes in update alert. |
| `SUBundleName` | String | — | Alternative display name (e.g. if bundle name already has version number). |
| `SUDefaultsDomain` | String | — | Alternative NSUserDefaults domain (e.g. for App Group suite). |
| `SURelaunchHostBundle` | Boolean | NO | For plug-ins: relaunch host bundle instead of app bundle after update. |

## Security Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `SUVerifyUpdateBeforeExtraction` | Boolean | NO | Verify update signature before extracting. Requires EdDSA signing. Recommended for stronger validation. (Sparkle 2.7.3+) |
| `SURequireSignedFeed` | Boolean | NO | Validate appcast and release notes are signed. Requires `SUVerifyUpdateBeforeExtraction`. (Sparkle 2.9+) |
| `SUSignedFeedFailureExpirationInterval` | Number | 1728000 (20 days) | Seconds before feed signing failure expires. `0` = never expire. Failsafe for lost keys. |
| `SUAllowedURLSchemes` | Array of Strings | — | Custom URL schemes allowed in release notes (default: only `https`). (Sparkle 2.5+) |
| `SUEnableJavaScript` | Boolean | NO | Allow JavaScript in release notes HTML. |

## Sandboxing Settings

Only for sandboxed apps (with `com.apple.security.app-sandbox` entitlement):

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `SUEnableInstallerLauncherService` | Boolean | NO | **Required** for all sandboxed apps. Enables Installer XPC Service. |
| `SUEnableDownloaderService` | Boolean | NO | Only if app doesn't have `com.apple.security.network.client` entitlement. Enables Downloader XPC Service. |
| `SUEnableInstallerConnectionService` | Boolean | NO | Usually not needed (use mach lookup entitlement instead). |
| `SUEnableInstallerStatusService` | Boolean | NO | Usually not needed (use mach lookup entitlement instead). |

## App Transport Security

macOS blocks HTTP by default. You **must** either:
- **Use HTTPS** for appcast URL and all download URLs (recommended). Free certs via [Let's Encrypt](https://letsencrypt.org/) or AWS Certificate Manager.
- Add an ATS exception to Info.plist ([Apple docs](https://developer.apple.com/library/prerelease/mac/technotes/App-Transport-Security-Technote/))

Test updating even if already using HTTPS — ATS has additional requirements.

## System Profiling

When `SUEnableSystemProfiling` is `YES`, Sparkle sends anonymous system info as GET parameters on the feed URL:

**Collected data:** macOS version, CPU type/subtype, Mac model, CPU count/cores, 32/64-bit, CPU speed, RAM size, app name, app version, preferred language.

Data is submitted once per week. The feed URL receives parameters like:
```
https://example.org/appcast.xml?cpusubtype=4&ncpu=2&appName=App.app&cpuFreqMHz=1830...
```

**Server-side setup:** Point `SUFeedURL` to a server script (e.g. `profileInfo.php`) that collects stats and redirects/links to the actual appcast.

**Known backends:**
- PHP: [sparkle stats server](https://sparkle-project.org/files/php_sparkle_stats_server.zip)
- CakePHP: [JDSparkle](https://github.com/balthisar/JDSparkle)
- Rails: [Sparkler](https://github.com/mackuba/sparkler)
- PHP (no DB): [Sparkle-Posse](https://habilis.net/sparkle-posse/)

**Sparkle 2 delegate methods:**
- `feedParametersForUpdater:sendingSystemProfile:` — add custom parameters
- `allowedSystemProfileKeysForUpdater:` — limit which data is sent

## Runtime API Expectations

### Do
- Set initial config in Info.plist
- Only set `SPUUpdater` properties when user changes settings
- Use `SPUUpdaterDelegate.feedURLString(for:)` for dynamic feed URLs
- Use `resetUpdateCycleAfterShortDelay()` after user changes channels or feed

### Don't
- Don't set updater properties at initialization
- Don't maintain separate user defaults for properties already backed by NSUserDefaults
- Don't call `checkForUpdatesInBackground` manually (Sparkle schedules this automatically)
- Don't use deprecated `-setFeedURL:` — use delegate method instead
- If you must force a background check at launch, only call `checkForUpdatesInBackground` immediately after starting the updater AND only if `automaticallyChecksForUpdates` is `YES`
