---
name: sparkle-win
description: >
  WinSparkle auto-update framework for Windows desktop apps. Use this skill when the user is
  integrating WinSparkle into a Windows application, configuring automatic updates via appcast feeds,
  signing updates with EdDSA, setting up win_sparkle_init/cleanup lifecycle, P/Invoke bindings for
  C#/.NET, configuring registry settings, custom HTTP headers, installer arguments for InnoSetup/MSI/NSIS,
  or troubleshooting WinSparkle update flows. Also trigger when code includes winsparkle.h, references
  win_sparkle_* functions, WinSparkle.dll, or appcast XML targeting Windows.
---

# WinSparkle

Plug-and-forget software update library for Windows applications. C API compatible with all Windows compilers. Shares appcast feed format with macOS Sparkle framework.

## How to Use This Skill

- **Getting started**: Read the Quick Start section below for your language (C/C++ or C#/.NET)
- **Complete API with all function signatures**: Read `references/api.md`
- **Migrating from DSA to EdDSA signing**: Read `references/eddsa-migration.md`

## Core Principles

- **Configuration before initialization**: All configuration functions (`set_appcast_url`, `set_eddsa_public_key`, `set_app_details`, etc.) must be called before `win_sparkle_init()`
- **EdDSA signing required**: Updates must be cryptographically signed with Ed25519; unsigned updates will be rejected
- **HTTPS everywhere**: The appcast URL, release notes links, and download URLs must all use HTTPS to prevent MITM attacks
- **Non-blocking checks**: All update check functions (`check_update_with_ui`, `check_update_without_ui`, etc.) return immediately; UI appears asynchronously on a background thread
- **Minimal first-launch footprint**: WinSparkle does nothing on first launch to avoid distorting the user's first impression of the app

## Quick Start (C/C++)

```c
#include <winsparkle.h>

// After main window is shown:
win_sparkle_set_appcast_url("https://example.com/appcast.xml");
win_sparkle_init();

// On app exit:
win_sparkle_cleanup();
```

## Quick Start (C#/.NET)

```csharp
using System.Runtime.InteropServices;

class WinSparkle
{
    [DllImport("WinSparkle.dll", CallingConvention = CallingConvention.Cdecl)]
    public static extern void win_sparkle_init();

    [DllImport("WinSparkle.dll", CallingConvention = CallingConvention.Cdecl)]
    public static extern void win_sparkle_cleanup();

    [DllImport("WinSparkle.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
    public static extern void win_sparkle_set_appcast_url(string url);

    [DllImport("WinSparkle.dll", CharSet = CharSet.Unicode, CallingConvention = CallingConvention.Cdecl)]
    public static extern void win_sparkle_set_app_details(string company_name, string app_name, string app_version);

    [DllImport("WinSparkle.dll", CharSet = CharSet.Ansi, CallingConvention = CallingConvention.Cdecl)]
    public static extern void win_sparkle_set_registry_path(string path);

    [DllImport("WinSparkle.dll", CallingConvention = CallingConvention.Cdecl)]
    public static extern void win_sparkle_check_update_with_ui();
}
```

## Installation

- Download prebuilt `WinSparkle.dll` from [GitHub releases](https://github.com/vslavik/winsparkle/releases)
- Or use NuGet: `WinSparkle` package
- Available for x86, x64, and arm64
- Single self-contained DLL with no external dependencies (C runtime linked statically)
- Visual C++ auto-links `WinSparkle.lib` via pragma; other compilers need manual import library setup

## Application Metadata

WinSparkle reads version info from VERSIONINFO resources. Set these `StringFileInfo` fields:

- `ProductName`
- `ProductVersion`
- `CompanyName`

Or provide via API: `win_sparkle_set_app_details(company, app, version)`

For C#, use `AssemblyFileVersion` in `AssemblyInfo.cs`:
```csharp
[assembly: AssemblyFileVersion("1.0.7.0")]
```

## Initialization Rules

1. Call configuration functions (`set_appcast_url`, `set_app_details`, etc.) **before** `win_sparkle_init()`
2. Call `win_sparkle_init()` after the main window is shown (WinSparkle may display UI immediately)
3. Call `win_sparkle_cleanup()` on app exit

## EdDSA Signing (Required)

Updates must be cryptographically signed. EdDSA (Ed25519) is the current standard.

### Generate Keys

```bash
winsparkle-tool generate-key --file private.key
# Output: Public key: pXAx0wfi8kGbeQln11+V4R3tCepSuLXeo7LkOeudc/U=
```

### Set Public Key

As Windows resource:
```
EdDSAPub EDDSA {"pXAx0wfi8kGbeQln11+V4R3tCepSuLXeo7LkOeudc/U="}
```

Or via API:
```c
win_sparkle_set_eddsa_public_key("pXAx0wfi8kGbeQln11+V4R3tCepSuLXeo7LkOeudc/U=");
```

### Sign Updates

```bash
winsparkle-tool sign --private-key-file private.key Updater.exe
# Output: sparkle:edSignature="JhQ69mg..." length="1736832"
```

Add output as `sparkle:edSignature` attribute on `<enclosure>` in appcast.

## Appcast URL Configuration

The appcast URL tells WinSparkle where to fetch update information. There are two ways to set it:

### Option 1: API call (recommended)

```c
win_sparkle_set_appcast_url("https://example.com/appcast.xml");
```

Must be called **before** `win_sparkle_init()`.

### Option 2: Windows resource

If `win_sparkle_set_appcast_url()` is not called, WinSparkle falls back to reading a Windows resource named `"FeedURL"` of type `"APPCAST"` from the executable:

```rc
// In your .rc resource file:
FeedURL APPCAST "https://example.com/appcast.xml"
```

### Custom HTTP headers

Add authentication tokens or custom headers sent with both appcast checks and update downloads:

```c
win_sparkle_set_http_header("Authorization", "Bearer my-token");
win_sparkle_set_http_header("X-App-Edition", "pro");
// Clear all custom headers:
win_sparkle_clear_http_headers();
```

## How Update Checking Works

1. **Automatic checks**: After `win_sparkle_init()`, WinSparkle fetches the appcast URL on a background thread at the configured interval (default: daily, minimum: hourly). No UI is shown unless an update is found.
2. **Manual checks**: `win_sparkle_check_update_with_ui()` fetches the appcast immediately with a progress spinner. Shows "no update" if current, or the update dialog if a newer version exists. Ignores "skip this version".
3. **Silent manual checks**: `win_sparkle_check_update_without_ui()` fetches the appcast silently. Only shows UI if an update is found. Respects "skip this version".
4. **Auto-install checks**: `win_sparkle_check_update_with_ui_and_install()` fetches the appcast and automatically downloads + installs if an update is found, skipping the user prompt.

All check functions return immediately (non-blocking). The appcast is always fetched over the network from the configured URL. WinSparkle compares `sparkle:version` in the feed against the app's `ProductVersion` (from VERSIONINFO) or the build version set via `win_sparkle_set_app_build_version()`.

### Version comparison

- By default, compares appcast `sparkle:version` against the app's `ProductVersion` from VERSIONINFO resources
- If `win_sparkle_set_app_build_version()` is used, the build version is compared against `sparkle:version`, and `sparkle:shortVersionString` is used for display in the update dialog

### Security: HTTPS everywhere

All parts of the update chain **must** use HTTPS:
- The appcast feed URL itself
- Release notes linked from the appcast (`sparkle:releaseNotesLink`)
- The download URL in `<enclosure url="...">`

Using HTTP exposes users to MITM attacks (hiding updates, tampering with downloads).

## Appcast Feed Format

RSS 2.0 feed with Sparkle extensions. Host the XML file on any HTTPS web server.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:sparkle="http://www.andymatuschak.org/xml-namespaces/sparkle">
  <channel>
    <title>My App Updates</title>
    <description>Most recent updates</description>
    <language>en</language>
    <item>
      <title>Version 2.0.0</title>
      <sparkle:version>2.0.0</sparkle:version>
      <sparkle:releaseNotesLink>https://example.com/notes.html</sparkle:releaseNotesLink>
      <pubDate>Fri, 06 Feb 2026 22:49:00 +0100</pubDate>
      <enclosure url="https://example.com/setup.exe"
                 sparkle:edSignature="JhQ69mgRxjNxS35z..."
                 length="1736832"
                 type="application/octet-stream" />
    </item>
  </channel>
</rss>
```

### Platform-Specific Updates

Use `sparkle:os` attribute on `<enclosure>`:

| Value | Target |
|-------|--------|
| `windows` | Any Windows |
| `windows-x86` | 32-bit Windows |
| `windows-x64` | 64-bit Windows (requires 64-bit DLL) |
| `windows-arm64` | ARM64 Windows |
| `macos` | macOS (for shared feeds) |

### Installer Arguments

Use `sparkle:installerArguments` on `<enclosure>`:

| Installer | Arguments | Notes |
|-----------|-----------|-------|
| InnoSetup | `/SILENT /SP- /NOICONS` | Shows progress and errors only |
| MSI | `/passive` | Unattended, progress bar only |
| NSIS | `/S` | Silent mode |

### Minimum OS Version

```xml
<sparkle:minimumSystemVersion>10.0</sparkle:minimumSystemVersion>
```

Values: `10.0` (Win10), `6.2` (Win8), `10.0.22000` (Win11). Format: `major[.minor[.build]]`.

### Critical Updates

```xml
<sparkle:tags>
  <sparkle:criticalupdate/>
</sparkle:tags>
```

Hides Skip and Remind Me Later buttons, and the close button on the dialog.

### Multiple Items and Version History

The appcast can contain multiple `<item>` elements. WinSparkle picks the newest compatible version:

```xml
<channel>
  <title>My App Updates</title>
  <item>
    <title>Version 3.0.0</title>
    <sparkle:version>3.0.0</sparkle:version>
    <sparkle:minimumSystemVersion>10.0</sparkle:minimumSystemVersion>
    <enclosure url="https://example.com/setup-3.0.exe"
               sparkle:edSignature="..."
               sparkle:os="windows-x64"
               sparkle:installerArguments="/SILENT /SP- /NOICONS"
               length="2048000"
               type="application/octet-stream" />
  </item>
  <item>
    <title>Version 2.5.0</title>
    <sparkle:version>2.5.0</sparkle:version>
    <enclosure url="https://example.com/setup-2.5.exe"
               sparkle:edSignature="..."
               sparkle:os="windows"
               length="1536000"
               type="application/octet-stream" />
  </item>
</channel>
```

### Publishing the Appcast

Upload the XML file to any HTTPS web server. No special server-side logic needed — it's a static file. Update the XML whenever you release a new version, adding a new `<item>` at the top.

## API Reference

See `references/api.md` for the complete API.

### Core Lifecycle

| Function | Description |
|----------|-------------|
| `win_sparkle_init()` | Start WinSparkle, begin automatic checks |
| `win_sparkle_cleanup()` | Shut down, cancel pending operations |

### Configuration (call before `init`)

| Function | Description |
|----------|-------------|
| `win_sparkle_set_appcast_url(url)` | Set appcast feed URL |
| `win_sparkle_set_eddsa_public_key(key)` | Set EdDSA public key (base64) |
| `win_sparkle_set_app_details(company, app, version)` | Override VERSIONINFO metadata |
| `win_sparkle_set_app_build_version(build)` | Set internal build version |
| `win_sparkle_set_registry_path(path)` | Custom registry path for settings |
| `win_sparkle_set_lang(lang)` | Set UI language (ISO 639 code) |
| `win_sparkle_set_langid(langid)` | Set UI language (Win32 LANGID) |
| `win_sparkle_set_http_header(name, value)` | Add custom HTTP header |
| `win_sparkle_clear_http_headers()` | Clear custom HTTP headers |
| `win_sparkle_set_config_methods(methods)` | Override registry with custom config storage |

### Update Check Settings

| Function | Description |
|----------|-------------|
| `win_sparkle_set_automatic_check_for_updates(state)` | Enable (1) or disable (0) auto-checks |
| `win_sparkle_get_automatic_check_for_updates()` | Get auto-check state |
| `win_sparkle_set_update_check_interval(seconds)` | Set interval (min 3600 = 1 hour) |
| `win_sparkle_get_update_check_interval()` | Get interval (default: 86400 = 1 day) |
| `win_sparkle_get_last_check_time()` | Get last check timestamp (-1 if never) |

### Manual Update Checks

| Function | Description |
|----------|-------------|
| `win_sparkle_check_update_with_ui()` | Check with full UI (ignores skip version) |
| `win_sparkle_check_update_with_ui_and_install()` | Check and auto-install (no prompt) |
| `win_sparkle_check_update_without_ui()` | Silent check, UI only if update found |

### Callbacks (call before `init`)

| Function | Description |
|----------|-------------|
| `win_sparkle_set_error_callback(cb)` | On updater error |
| `win_sparkle_set_can_shutdown_callback(cb)` | Query if app can shut down (return TRUE/FALSE) |
| `win_sparkle_set_shutdown_request_callback(cb)` | Request app shutdown for install |
| `win_sparkle_set_did_find_update_callback(cb)` | Update found |
| `win_sparkle_set_did_not_find_update_callback(cb)` | No update found |
| `win_sparkle_set_update_cancelled_callback(cb)` | User cancelled update |
| `win_sparkle_set_update_skipped_callback(cb)` | User clicked Skip |
| `win_sparkle_set_update_postponed_callback(cb)` | User clicked Remind Me Later |
| `win_sparkle_set_update_dismissed_callback(cb)` | Dialog closed (any reason) |
| `win_sparkle_set_user_run_installer_callback(cb)` | Payload ready, return 1 if handled |

## Registry Settings

Stored in `HKCU\Software\<vendor>\<app>\WinSparkle` (falls back to HKLM).

| Key | Type | Description |
|-----|------|-------------|
| `CheckForUpdates` | bool | Whether to auto-check |
| `LastCheckTime` | time_t | Timestamp of last check |
| `UpdateInterval` | int | Check interval in seconds |
| `SkipThisVersion` | string | Version string to skip |
| `DidRunOnce` | bool | Has app been launched before |

## User Experience

- **First launch**: WinSparkle does nothing (doesn't distort first impression)
- **Second launch**: Asks user if they want automatic update checks
- **Subsequent launches**: Silent background checks; shows update dialog only when new version found
- Update dialog shows release notes (HTML) and offers: Install Update, Skip This Version, Remind Me Later

## Migrating DSA to EdDSA

See `references/eddsa-migration.md` for step-by-step migration guide.

## Language Bindings

| Language | Link |
|----------|------|
| C#/.NET | P/Invoke (see Quick Start above) |
| Python | [pywinsparkle](https://pypi.org/project/pywinsparkle/) |
| Go | [go-winsparkle](https://github.com/abemedia/go-winsparkle) |
| Pascal | Bundled in WinSparkle source |

## Common Pitfalls

- **Calling `win_sparkle_init()` before configuration functions**: The appcast URL, EdDSA public key, and other settings will be silently ignored if set after initialization
- **Using HTTP instead of HTTPS**: Serving the appcast, release notes, or download URL over HTTP exposes users to man-in-the-middle attacks that can hide updates or tamper with downloads
- **Not implementing `can_shutdown_callback`**: Without this callback, the app may force-quit during an update installation, causing users to lose unsaved work
- **Version mismatch between VERSIONINFO and appcast**: The `ProductVersion` in your executable's VERSIONINFO resource and `sparkle:version` in the appcast must align, or WinSparkle won't detect available updates

## Resources

- [GitHub Repository](https://github.com/vslavik/winsparkle)
- [Website](https://winsparkle.org)
- [Wiki](https://github.com/vslavik/winsparkle/wiki)
- [NuGet Package](https://www.nuget.org/packages/WinSparkle/)
- [Sparkle (macOS equivalent)](https://sparkle-project.org/)
