# Sparkle Sandboxing Guide

Sparkle 2 supports sandboxed macOS applications via XPC Services. Sparkle 1 does not support sandboxing.

## Required XPC Services

Sparkle bundles two XPC Services inside the framework:

| Service | Purpose | Required? |
|---------|---------|-----------|
| `Installer.xpc` | Installs updates outside app sandbox | **Yes** — all sandboxed apps |
| `Downloader.xpc` | Downloads updates without network entitlement | Only if app lacks `com.apple.security.network.client` |

## Setup Steps

### 1. Enable Installer XPC Service (Required)

Add to Info.plist:
```xml
<key>SUEnableInstallerLauncherService</key>
<true/>
```

### 2. Add Communication Entitlements (Required)

Add to your `.entitlements` file:
```xml
<key>com.apple.security.temporary-exception.mach-lookup.global-name</key>
<array>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spks</string>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spki</string>
</array>
```

`$(PRODUCT_BUNDLE_IDENTIFIER)` is auto-substituted by Xcode.

### 3. Enable Downloader XPC Service (Optional)

Only needed if your sandboxed app does NOT have the `com.apple.security.network.client` entitlement.

Add to Info.plist:
```xml
<key>SUEnableDownloaderService</key>
<true/>
```

**Downloader XPC Service drawbacks:**
- Falls back to deprecated `WebView` for release notes (WKWebView doesn't work without network entitlement)
- Version-adaptive HTML release notes not supported
- External content in release notes may not load
- Not sandboxed by default (as of Sparkle 2.6)

**Recommendation:** Consider enabling `com.apple.security.network.client` entitlement instead.

## Code Signing

### Standard Workflow (Recommended)

Use Xcode's Archive & Export workflow (Product > Archive > Distribute App > Developer ID). Xcode handles re-signing XPC Services and helpers automatically.

### Manual Re-signing

If you need manual re-signing (custom build workflows):

```sh
# Re-sign XPC Services
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/XPCServices/Installer.xpc

# Downloader (Sparkle >= 2.6)
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime --preserve-metadata=entitlements Sparkle.framework/Versions/B/XPCServices/Downloader.xpc

# Helper tools
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/Autoupdate
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/Updater.app

# Framework itself (last)
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework
```

**Important:** Do NOT use `--deep` for code signing — it breaks Downloader XPC Service entitlements.

## Removing XPC Services (Non-Sandboxed Apps)

If you don't sandbox your app, you can strip XPC Services to save space. Add a build phase script:

```bash
#!/bin/bash

# Only run for install / archive builds
if [ "$ACTION" != "install" ]; then
    exit 0
fi

APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"
SPARKLE_FRAMEWORK="${APP_PATH}/Contents/Frameworks/Sparkle.framework"

PATHS_TO_REMOVE=(
    # Remove Downloader XPC if not needed
    "${SPARKLE_FRAMEWORK}/Versions/B/XPCServices/Downloader.xpc"

    # Uncomment to remove ALL XPC Services (non-sandboxed apps only)
    #"${SPARKLE_FRAMEWORK}/Versions/B/XPCServices"
    #"${SPARKLE_FRAMEWORK}/XPCServices"
)

for p in "${PATHS_TO_REMOVE[@]}"; do
    if [ -e "$p" ] || [ -L "$p" ]; then
        rm -rf "$p"
    fi
done

# Re-sign framework after modification
codesign --force --sign "$EXPANDED_CODE_SIGN_IDENTITY" --preserve-metadata "$SPARKLE_FRAMEWORK"

exit 0
```

## Debugging

If Xcode can't attach to XPC Service processes, edit your Scheme and disable "Debug XPC services used by app", or test detached from Xcode. The distributed XPC Services have Hardened Runtime enabled which prevents debugging.
