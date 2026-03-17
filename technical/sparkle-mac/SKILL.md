---
name: sparkle-mac
description: >
  Sparkle macOS auto-update framework expert. Use this skill when the user is integrating Sparkle
  into a macOS app, configuring automatic updates, creating appcasts, signing updates with EdDSA,
  setting up SwiftUI or Cocoa updater controllers, sandboxing with XPC Services, publishing delta
  updates, handling package (pkg) installs, adding update preferences UI, implementing gentle
  update reminders, configuring channels for beta/staging, customizing Info.plist update settings,
  or troubleshooting Sparkle update flows. Also trigger when code imports Sparkle, references
  SPUUpdater, SPUStandardUpdaterController, SUAppcastItem, SPUUserDriver, or mentions
  generate_appcast, sign_update, SUFeedURL, SUPublicEDKey, or appcast XML. Covers Sparkle 2.x.
---

# Sparkle macOS Auto-Update Framework (v2.x)

You are an expert at integrating the Sparkle framework for macOS application auto-updates. This skill references the **Sparkle 2.x** official documentation.

## Core Principles

1. **Security first** — Always use HTTPS for appcast feeds. Sign updates with EdDSA (ed25519). Notarize and code-sign apps via Apple Developer ID. Never store signing keys on the update server.
2. **Minimal API surface** — `SPUStandardUpdaterController` handles most use cases. Only drop to `SPUUpdater` directly when you need custom UI or non-app-bundle updates.
3. **Info.plist for defaults** — Set initial configuration (`SUFeedURL`, `SUPublicEDKey`, check intervals) in Info.plist. Only use runtime APIs when responding to user setting changes.
4. **Appcast-driven** — Updates are described via RSS-based appcast XML. Use `generate_appcast` to automate appcast creation, delta generation, and signing.
5. **Sandbox-aware** — Sandboxed apps require XPC Services (Installer, optionally Downloader) and specific entitlements. Non-sandboxed apps can optionally strip XPC Services to save space.

## How to Use This Skill

Before generating code, load the relevant reference file(s):

- `references/setup-and-integration.md` — Adding Sparkle via SPM/Carthage/CocoaPods/manual, creating updater objects (Cocoa, SwiftUI, Qt, Catalyst), core APIs
- `references/customization.md` — All Info.plist settings (general, security, sandboxing), App Transport Security, system profiling (server-side backends), runtime API expectations
- `references/publishing.md` — Archiving apps, signing updates, appcast XML format, delta updates, channels, versioning, release notes (HTML/markdown/plain text/localized/adaptive), phased rollouts, critical/major/informational updates, extending appcast with custom XML
- `references/sandboxing.md` — XPC Services setup, entitlements, code signing for sandboxed apps, removing XPC Services
- `references/advanced.md` — Gentle update reminders, preferences UI (Cocoa/SwiftUI/Qt), custom user interfaces (SPUUserDriver), sparkle-cli command-line updater, non-app bundle updates (plug-ins), package updates, upgrading from Sparkle 1

## Quick Reference

### Key Classes

| Class | Purpose |
|-------|---------|
| `SPUStandardUpdaterController` | Convenience controller for standard UI + main bundle updates. Use in nibs or instantiate in code. |
| `SPUUpdater` | Core updater engine. Use directly for custom UI or non-app-bundle updates. |
| `SPUStandardUserDriver` | Default UI driver. Handles update alerts, download progress, release notes. |
| `SUAppcastItem` | Represents a single update item from the appcast feed. |
| `SPUUpdaterDelegate` | Delegate for updater behavior customization (feed URL, channels, update filtering). |
| `SPUStandardUserDriverDelegate` | Delegate for UI customization (gentle reminders, release notes formatting). |

### Essential Info.plist Keys

| Key | Type | Purpose |
|-----|------|---------|
| `SUFeedURL` | String | URL to your appcast XML feed |
| `SUPublicEDKey` | String | Base64-encoded public EdDSA key for update verification |
| `SUEnableAutomaticChecks` | Boolean | Enable/disable automatic update checks without prompting user |
| `SUAutomaticallyUpdate` | Boolean | Auto-download and install updates silently (default: NO) |
| `SUScheduledCheckInterval` | Number | Seconds between checks (default: 86400 = 1 day, min: 3600) |

### Sparkle CLI Tools

| Tool | Purpose |
|------|---------|
| `./bin/generate_keys` | Generate EdDSA keypair (stored in Keychain). Run once. |
| `./bin/generate_appcast` | Auto-generate appcast XML, delta updates, and signatures from a folder of archives |
| `./bin/sign_update` | Manually sign an update archive or release notes file with EdDSA |
| `BinaryDelta` | Manually create/apply binary delta patches between app versions |

### Cocoa Setup (Programmatic)

```swift
import Sparkle

@NSApplicationMain
class AppDelegate: NSObject, NSApplicationDelegate {
    @IBOutlet var checkForUpdatesMenuItem: NSMenuItem!
    let updaterController: SPUStandardUpdaterController

    override init() {
        updaterController = SPUStandardUpdaterController(startingUpdater: true, updaterDelegate: nil, userDriverDelegate: nil)
    }

    func applicationDidFinishLaunching(_ notification: Notification) {
        checkForUpdatesMenuItem.target = updaterController
        checkForUpdatesMenuItem.action = #selector(SPUStandardUpdaterController.checkForUpdates(_:))
    }
}
```

### SwiftUI Setup

```swift
import SwiftUI
import Sparkle

final class CheckForUpdatesViewModel: ObservableObject {
    @Published var canCheckForUpdates = false
    init(updater: SPUUpdater) {
        updater.publisher(for: \.canCheckForUpdates)
            .assign(to: &$canCheckForUpdates)
    }
}

struct CheckForUpdatesView: View {
    @ObservedObject private var checkForUpdatesViewModel: CheckForUpdatesViewModel
    private let updater: SPUUpdater

    init(updater: SPUUpdater) {
        self.updater = updater
        self.checkForUpdatesViewModel = CheckForUpdatesViewModel(updater: updater)
    }

    var body: some View {
        Button("Check for Updates...", action: updater.checkForUpdates)
            .disabled(!checkForUpdatesViewModel.canCheckForUpdates)
    }
}

@main
struct MyApp: App {
    private let updaterController: SPUStandardUpdaterController

    init() {
        updaterController = SPUStandardUpdaterController(startingUpdater: true, updaterDelegate: nil, userDriverDelegate: nil)
    }

    var body: some Scene {
        WindowGroup { ContentView() }
        .commands {
            CommandGroup(after: .appInfo) {
                CheckForUpdatesView(updater: updaterController.updater)
            }
        }
    }
}
```

### Appcast Item Template

```xml
<item>
    <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>2.0</sparkle:version>
    <sparkle:shortVersionString>2.0</sparkle:shortVersionString>
    <sparkle:releaseNotesLink>https://example.com/release_notes/app_2.0.html</sparkle:releaseNotesLink>
    <pubDate>Mon, 05 Oct 2015 19:20:11 +0000</pubDate>
    <enclosure url="https://example.com/downloads/app.dmg"
               sparkle:edSignature="..."
               length="1623481"
               type="application/octet-stream" />
</item>
```

### Archive Commands

```sh
# Create zip (recommended for zip distribution)
ditto -c -k --sequesterRsrc --keepParent MyApp.app MyApp.zip

# Create tar.xz (high compression)
tar --no-xattrs -cJf MyApp.tar.xz MyApp.app

# Generate appcast from update archives folder
./bin/generate_appcast /path/to/updates_folder/

# Sign an update manually
./bin/sign_update MyApp.zip
```

### Common Pitfalls

1. **Don't call `checkForUpdatesInBackground` manually** — Sparkle handles scheduled checks automatically. Only call it immediately after starting the updater if the user hasn't disabled auto-checks.
2. **Don't use `-setFeedURL:`** — It's deprecated. Use `SPUUpdaterDelegate.feedURLString(for:)` instead.
3. **Don't use `--deep` for code signing** — It breaks sandboxed XPC Services that need specific entitlements.
4. **Don't set runtime updater properties at init** — Only set `automaticallyChecksForUpdates`, `automaticallyDownloadsUpdates`, etc. in response to user setting changes, not at startup.
5. **CFBundleVersion must increment** — Sparkle uses this to compare versions. Must be machine-readable (numeric). Use `CFBundleShortVersionString` for human-readable display.
6. **Preserve symlinks in archives** — macOS frameworks use symlinks; breaking them invalidates code signatures.
7. **Test with old version** — Sparkle won't show updates if the running app has the same or newer version. Decrease `CFBundleVersion` temporarily for testing.
