# Sparkle Advanced Features

## Gentle Update Reminders (Sparkle 2.2+)

Sparkle's standard UI shows updates at opportune times:
- On app launch or after recent interaction with updater
- When system has been idle (no display-sleep power assertions)
- When user returns to your app from another app (not while actively using it)

Background (dockless) apps won't steal focus from other windows (except at launch).

### Implementing Gentle Reminders

Implement `SPUStandardUserDriverDelegate`:

```swift
// Declare support for gentle reminders
var supportsGentleScheduledUpdateReminders: Bool {
    return true
}

// Override Sparkle's default update presentation
func standardUserDriverShouldHandleShowingScheduledUpdate(_ update: SUAppcastItem, andInImmediateFocus immediateFocus: Bool) -> Bool {
    // Let Sparkle handle immediate-focus updates (e.g., near launch)
    // Handle non-immediate updates ourselves with gentle UI
    return immediateFocus
}

// Add custom UI when update is available
func standardUserDriverWillHandleShowingUpdate(_ handleShowingUpdate: Bool, forUpdate update: SUAppcastItem, state: SPUUserUpdateState) {
    guard !handleShowingUpdate else { return }
    // Add badge, titlebar button, notification, etc.
}

// User gave attention to the update
func standardUserDriverDidReceiveUserAttention(forUpdate update: SUAppcastItem) {
    // Clear badges/notifications
}

// Update session finished
func standardUserDriverWillFinishUpdateSession() {
    // Clean up gentle UI indicators
}
```

### Window Titlebar Button Example

```swift
func standardUserDriverWillHandleShowingUpdate(_ handleShowingUpdate: Bool, forUpdate update: SUAppcastItem, state: SPUUserUpdateState) {
    guard !handleShowingUpdate else { return }

    let updateButton = NSButton(frame: NSMakeRect(0, 0, 120, 100))
    updateButton.title = "v\(update.displayVersionString) Available"
    updateButton.bezelStyle = .recessed
    updateButton.target = updaterController
    updateButton.action = #selector(updaterController.checkForUpdates(_:))

    let accessoryViewController = NSTitlebarAccessoryViewController()
    accessoryViewController.layoutAttribute = .right
    accessoryViewController.view = updateButton

    window.addTitlebarAccessoryViewController(accessoryViewController)
    titlebarAccessoryViewController = accessoryViewController
}
```

### Background App with Dock Badge + Notification

For dockless apps, bring app to Dock and post notification:

```swift
func standardUserDriverWillHandleShowingUpdate(_ handleShowingUpdate: Bool, forUpdate update: SUAppcastItem, state: SPUUserUpdateState) {
    // Bring to foreground
    NSApp.setActivationPolicy(.regular)

    if !state.userInitiated {
        NSApp.dockTile.badgeLabel = "1"

        let content = UNMutableNotificationContent()
        content.title = "A new update is available"
        content.body = "Version \(update.displayVersionString) is now available"

        let request = UNNotificationRequest(identifier: "UpdateCheck", content: content, trigger: nil)
        UNUserNotificationCenter.current().add(request)
    }
}

func standardUserDriverWillFinishUpdateSession() {
    NSApp.setActivationPolicy(.accessory)
}
```

### Testing Gentle Reminders

```sh
# Test immediate check (near launch)
defaults delete my-bundle-id SULastCheckTime

# Test background check (~30-60s after launch)
defaults write my-bundle-id SULastCheckTime -date "$(date -v-1d -v+30S)"
```

---

## Preferences UI

### SwiftUI Settings

```swift
struct UpdaterSettingsView: View {
    private let updater: SPUUpdater

    @State private var automaticallyChecksForUpdates: Bool
    @State private var automaticallyDownloadsUpdates: Bool

    init(updater: SPUUpdater) {
        self.updater = updater
        self.automaticallyChecksForUpdates = updater.automaticallyChecksForUpdates
        self.automaticallyDownloadsUpdates = updater.automaticallyDownloadsUpdates
    }

    var body: some View {
        VStack {
            Toggle("Automatically check for updates", isOn: $automaticallyChecksForUpdates)
                .onChange(of: automaticallyChecksForUpdates) { newValue in
                    updater.automaticallyChecksForUpdates = newValue
                }

            Toggle("Automatically download updates", isOn: $automaticallyDownloadsUpdates)
                .disabled(!automaticallyChecksForUpdates)
                .onChange(of: automaticallyDownloadsUpdates) { newValue in
                    updater.automaticallyDownloadsUpdates = newValue
                }
        }.padding()
    }
}

// Add to App body:
Settings {
    UpdaterSettingsView(updater: updaterController.updater)
}
```

### Cocoa Bindings (NIB)

For "Automatically check for updates" checkbox:
- Bind Value to File's Owner with Model Key Path: `updater.automaticallyChecksForUpdates`

For update check interval popup:
- Set menu item tags to seconds (3600, 86400, 604800, 2629800)
- Bind Selected Tag to `updater.updateCheckInterval`
- Bind Enabled to `updater.automaticallyChecksForUpdates`

**Important:** Only set updater properties when the user changes settings, not at init. Properties are backed by NSUserDefaults — don't maintain duplicate defaults.

### Preference Caching Warning

macOS caches plist files in `~/Library/Preferences`. Don't edit them directly. Use `defaults` command for testing:
```sh
defaults read my-bundle-id
defaults write my-bundle-id SULastCheckTime -date "2024-01-01"
defaults delete my-bundle-id SULastCheckTime
```

---

## Package Updates

For apps with custom installation needs that can't use regular app bundles.

**Downsides vs regular updates:**
- Always requires user authorization (no silent installs)
- Slower relaunching and installation
- No delta update support
- No key rotation fallback
- No `generate_appcast` support

### Bare Package (Recommended — Sparkle 1.26+)

Serve flat `*.pkg` or `*.mpkg` directly without archiving. Avoids redundant compression.

### Archived Package

Archive contains `*.pkg` or `*.mpkg` at root. For Sparkle 2, add to appcast enclosure:
```xml
sparkle:installationType="package"
```

**Note:** Apps with daemons (`SMAppService`) or system extensions don't need package installers.

---

## Testing Sparkle

1. Use an old version of your app (decrease `CFBundleVersion` temporarily)
2. Run app, then quit (Sparkle waits until second launch by default)
3. Run again — update process should trigger
4. Clear last check time to test immediately:
```sh
defaults delete my-bundle-id SULastCheckTime
```
5. Check `Console.app` for detailed update logs
6. Keep `.dSYM` files for crash log symbolication

### Quick Testing Checklist

- [ ] `CFBundleVersion` is lower than update version
- [ ] `SUFeedURL` points to correct appcast
- [ ] `SUPublicEDKey` matches the signing key
- [ ] Appcast is accessible via HTTPS
- [ ] Update archive is properly signed
- [ ] Code signing is valid: `codesign --deep --verify <path-to-app>`

---

## Upgrading from Sparkle 1 to Sparkle 2

### Key Changes

| Sparkle 1 | Sparkle 2 |
|-----------|-----------|
| `SUUpdater` (singleton) | `SPUStandardUpdaterController` + `SPUUpdater` (no singletons) |
| `+[SUUpdater sharedUpdater]` | Store reference to controller/updater |
| `Versions/A/` | `Versions/B/` |
| Autoupdate (GUI) | Autoupdate (CLI) + Updater.app (GUI) |
| No sandbox support | XPC Services for sandbox |
| DSA signing | EdDSA (ed25519) signing |

### Migration Steps

1. Replace `SUUpdater` nib class with `SPUStandardUpdaterController`
2. Replace `+[SUUpdater sharedUpdater]` calls with stored references
3. Generate EdDSA keys and add `SUPublicEDKey` to Info.plist
4. Update any code referencing Sparkle framework paths (`Versions/A` → `Versions/B`)
5. If sandboxed, follow the [sandboxing guide](https://sparkle-project.org/documentation/sandboxing)
6. If using package updates, add `sparkle:installationType="package"` to appcast enclosures

### Version History

| Version | Min macOS | Notable Changes |
|---------|-----------|-----------------|
| 2.9 | 10.13 | Signed feeds, markdown release notes, hardware requirements, impatient check interval |
| 2.8 | 10.13 | Refreshed UI, removed interactive pkg |
| 2.7 | 10.13 | Delta v4, Apple Archive support, deprecated custom version comparators |
| 2.6 | 10.13 | Downloader XPC no longer sandboxed by default |
| 2.5 | 10.13 | Adaptive HTML release notes, custom URL schemes |
| 2.4 | 10.13 | Plain text release notes, deprecated `-setFeedURL:`, aggressive symbol stripping |
| 2.3 | 10.13 | Requires macOS 10.13+, DSA-only no longer supported |
| 2.2 | 10.11 | Renamed XPC Services, gentle reminders |
| 2.1 | 10.11 | Delta v3, `ignoreSkippedUpgradesBelowVersion`, `belowVersion` for informational |
| 2.0 | 10.11 | Complete rewrite, sandbox support, channels, split API |

---

## Custom User Interfaces

For apps needing UI beyond Sparkle's standard interface or gentle reminders.

### SPUUserDriver Protocol

Create a class conforming to `SPUUserDriver` and pass it when instantiating `SPUUpdater` directly (instead of using `SPUStandardUpdaterController`).

**Reference implementations in Sparkle repo:**
- `SPUStandardUserDriver` — Standard UI
- `SUPopUpTitlebarUserDriver` — Titlebar popup alternative (Test App)
- `SPUCommandLineUserDriver` — CLI interface (sparkle-cli)

### Key Considerations

1. **Update permission:** Implement `showUpdatePermissionRequest:reply:` for the second-launch permission dialog. Test by deleting defaults:
   ```sh
   defaults delete my-bundle-id SUEnableAutomaticChecks
   defaults delete my-bundle-id SUAutomaticallyUpdate
   defaults delete my-bundle-id SUSendProfileInfo
   ```

2. **Authorization:** Some installs require admin credentials (package updates, apps owned by root, managed systems). Your UI must let users control when install happens to avoid unexpected auth prompts. Test:
   ```sh
   sudo chown root path-to-app-bundle
   ```

3. **Update states:** Handle multiple stages — not yet downloaded, already installing (auto-update), downloaded but needs auth, informational only.

4. **Bring back in focus:** Implement `showUpdateInFocus` (optional since Sparkle 2.8) if re-focusing the update UI makes sense.

**Important:** `SPUUserDriver` must present a UI. It cannot be used to silently install updates — use `SUAutomaticallyUpdate` for that.

---

## sparkle-cli (Command Line Updater)

Command line tool for updating Sparkle-based applications and bundles. Removed from binary distribution in Sparkle 2.9 — build from source if needed.

### Usage

```sh
./sparkle.app/Contents/MacOS/sparkle bundle [options]
```

### Key Options

| Option | Description |
|--------|-------------|
| `--check-immediately` | Check for updates now (not on schedule) |
| `--probe` | Check if update available without installing (exit 0 = available) |
| `--allow-major-upgrades` | Include major upgrades in probe/install |
| `--channels chan1,chan2` | Allowed Sparkle channels |
| `--feed-url url` | Override feed URL |
| `--interactive` | Allow authorization prompts |
| `--defer-install` | Install after app terminates (don't relaunch) |
| `--grant-automatic-checks` | Auto-grant update check permission |
| `--send-profile` | Send system profile info |
| `--user-agent-name name` | Custom User-Agent display name |
| `--verbose` | Enable verbose logging |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Update available (with `--probe`) or success |
| 2 | Major upgrade found (without `--allow-major-upgrades`) |
| 3 | Authorization required (without `--interactive`) |
| 5 | User cancelled authorization |
| 6 | Permission needed (without `--grant-automatic-checks`) |
| 8 | Installation failed (Gatekeeper/permissions) |

### Example

```sh
./sparkle.app/Contents/MacOS/sparkle --check-immediately /Applications/MyApp.app/
```

### Deployment

Customize `PRODUCT_BUNDLE_IDENTIFIER` in `ConfigSparkleTool.xcconfig` and build from source.

---

## Updating Non-App Bundles (Plug-ins, Preference Panes)

### Sparkle 2 Approach

Sparkle 2 distinguishes `hostBundle` (the bundle being updated) from `applicationBundle` (the app to terminate/relaunch). Instantiate `SPUUpdater` directly with the target bundle:

```swift
let updater = SPUUpdater(hostBundle: pluginBundle,
                         applicationBundle: hostAppBundle,
                         userDriver: myUserDriver,
                         delegate: self)
```

- Multiple updaters can coexist (same process or different processes)
- One updater can start an update that another resumes
- Use `SURelaunchHostBundle` Info.plist key to relaunch host bundle (e.g. System Settings prefpane)

### Plug-in Best Practices

- **Don't inject** Sparkle.framework into host process — use an out-of-process tool like sparkle-cli
- If plug-in inherits a sandbox, use a companion app with XPC Services
- For Sparkle 1 legacy: use `[SUUpdater updaterForBundle:]` or subclass `SUUpdater`
