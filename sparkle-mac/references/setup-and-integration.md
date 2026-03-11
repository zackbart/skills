# Sparkle Setup and Integration

## Adding Sparkle to Your Project

### Swift Package Manager (Recommended)

1. In Xcode: File > Add Packages...
2. Enter `https://github.com/sparkle-project/Sparkle` as the package repository URL
3. Choose default package options (Xcode auto-updates Sparkle 2 versions)

Tools location: `../artifacts/sparkle/Sparkle/bin/` (from Xcode's project navigator, right-click Sparkle package > Show in Finder, go up one folder from `checkouts`)

### Carthage

```
binary "https://sparkle-project.org/Carthage/Sparkle.json"
```

```sh
carthage update
```

- Drag `Carthage/Build/Mac/Sparkle.framework` into Xcode project
- Ensure target is checked in Add to targets
- Under General > Frameworks, Libraries, and Embedded Content: set Sparkle.framework to **Embed & Sign**
- Tools must be downloaded separately from [GitHub releases](https://github.com/sparkle-project/Sparkle/releases/latest)

**Important:** Only `binary` origin is supported with Carthage (Carthage strips code signing when building from source).

### CocoaPods (Deprecated)

```ruby
pod 'Sparkle'
use_frameworks!
```

### Manual Integration

1. Download from [GitHub releases](https://github.com/sparkle-project/Sparkle/releases/latest)
2. Drag `Sparkle.framework` into Xcode project (check "Copy items")
3. General > Frameworks, Libraries, and Embedded Content: **Embed & Sign**
4. Build Settings: set "Runpath Search Paths" to `@loader_path/../Frameworks`
5. Preserve symlinks and executable permissions when copying/packaging

### Library Validation Note

If Library Validation is enabled (required for notarization via Hardened Runtime):
- Sign with `Apple Development` certificate for development (requires Apple Developer Program)
- Or disable Library Validation for Debug configurations only
- Not an issue for distribution with Developer ID certificate

---

## Setting Up the Updater Object

### Option 1: Interface Builder (Cocoa NIB)

1. Open `MainMenu.xib`
2. View > Show Library > search "Object" > drag Object to sidebar
3. Select Object > View > Inspectors > Identity
4. Set Class to `SPUStandardUpdaterController`
5. Optionally add "Check for Updates..." menu item with target = controller, action = `checkForUpdates:`

### Option 2: Cocoa Programmatic

```swift
import Cocoa
import Sparkle

@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate {
    @IBOutlet var checkForUpdatesMenuItem: NSMenuItem!

    let updaterController: SPUStandardUpdaterController

    override init() {
        // Pass false to startingUpdater to start manually later via .startUpdater()
        updaterController = SPUStandardUpdaterController(startingUpdater: true, updaterDelegate: nil, userDriverDelegate: nil)
    }

    func applicationDidFinishLaunching(_ notification: Notification) {
        checkForUpdatesMenuItem.target = updaterController
        checkForUpdatesMenuItem.action = #selector(SPUStandardUpdaterController.checkForUpdates(_:))
    }
}
```

### Option 3: SwiftUI

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
        WindowGroup {
            ContentView()
        }
        .commands {
            CommandGroup(after: .appInfo) {
                CheckForUpdatesView(updater: updaterController.updater)
            }
        }
    }
}
```

### Option 4: Mac Catalyst

Sparkle must be loaded via an AppKit bundle plug-in:

```swift
import AppKit
import Foundation
import Sparkle

@objc class AppKitPlugin: NSObject, PlugIn {
    let updaterController: SPUStandardUpdaterController

    required override init() {
        updaterController = SPUStandardUpdaterController(startingUpdater: false, updaterDelegate: nil, userDriverDelegate: nil)
    }

    func startUpdater() {
        updaterController.startUpdater()
    }
}
```

**Catalyst notes:**
- HTML release notes not supported (WebKit views can't be used in Catalyst)
- Use markdown (macOS 12+, Sparkle 2.9) or plain text (Sparkle 2.4) release notes instead
- Or set `SUShowReleaseNotes` to `NO`
- Or implement a custom user interface with iOS WebKit view

### Option 5: Qt / External Build Systems

See the [Qt integration example](https://sparkle-project.org/documentation/programmatic-setup#create-an-updater-in-qt) for Objective-C++ integration using KVO for menu item state.

Lower-level requirements for external build systems:
- Compile `.mm` files with `-fobjc-arc`
- Link: `-framework Sparkle`
- Framework search path: `-F/path/to/dir/`
- Rpath: `-Wl,-rpath,@loader_path`
- Copy `Sparkle.framework` to `Contents/Frameworks/` preserving symlinks
- Set `CFBundleIdentifier`, `CFBundleVersion`, `CFBundleShortVersionString` in Info.plist
- Set `SUFeedURL` and `SUPublicEDKey` in Info.plist
- Set deployment target and architectures

---

## Core APIs

### SPUStandardUpdaterController

Use when updating your own app bundle with Sparkle's standard UI.

Key methods:
- `init(startingUpdater:updaterDelegate:userDriverDelegate:)` — Initialize controller
- `startUpdater()` — Start updater manually (if `startingUpdater: false`)
- `checkForUpdates(_:)` — User-initiated update check (use as menu item action)
- `.updater` — Access the underlying `SPUUpdater` instance

### SPUUpdater

Core updater engine. Access via `SPUStandardUpdaterController.updater` or instantiate directly for custom UI.

Key properties/methods:
- `canCheckForUpdates` — KVO-compliant property for menu item validation
- `checkForUpdates()` — User-initiated update check
- `checkForUpdatesInBackground()` — Background check (avoid calling manually)
- `automaticallyChecksForUpdates` — Get/set auto-check preference
- `automaticallyDownloadsUpdates` — Get/set auto-download preference
- `updateCheckInterval` — Get/set check interval
- `resetUpdateCycle()` — Reset scheduler (use after changing feed/channels)
- `resetUpdateCycleAfterShortDelay()` — Same but with a short delay
- `startUpdater()` / `startUpdater:` — Start the updater
- `clearFeedURLFromUserDefaults` — Clear custom feed URL from user defaults (Sparkle 2.4+)

### API Best Practices

1. **Set defaults in Info.plist**, not programmatically
2. **Only set runtime properties in response to user setting changes** (not at init)
3. **Don't maintain separate user defaults** for properties already backed by NSUserDefaults
4. **Don't call `checkForUpdatesInBackground` manually** — Sparkle schedules this automatically
5. **Use `resetUpdateCycleAfterShortDelay()`** after user changes feed URL or channels
6. **Use `feedURLString(for:)` delegate** instead of deprecated `-setFeedURL:`

---

## EdDSA Key Setup

### Generate Keys (One-Time)

```sh
./bin/generate_keys
```

This saves a private key to your login Keychain and prints the public key.

### Add Public Key to Info.plist

```xml
<key>SUPublicEDKey</key>
<string>pfIShU4dEXqPd5ObYNfDBiQWcXozk7estwzTnF9BamQ=</string>
```

### Key Management

- Keys are stored in macOS Keychain — lost if keychain/system is erased
- Export: `./bin/generate_keys -x private-key-file`
- Import: `./bin/generate_keys -f private-key-file`
- Run `./bin/generate_keys` again anytime to see the public key
- If keys are lost, use [key rotation](https://sparkle-project.org/documentation/#rotating-signing-keys) for Developer ID signed apps

### Key Rotation

For regular app bundles (not packages), if you both code-sign with Developer ID AND include EdDSA key:
- You can rotate either your Apple certificate OR your EdDSA keys (but not both at once)
- With `SUVerifyUpdateBeforeExtraction` enabled, EdDSA key changes require Developer ID signed DMG
