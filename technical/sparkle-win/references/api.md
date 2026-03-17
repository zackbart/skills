# WinSparkle Complete API Reference

Source: `winsparkle.h` from [github.com/vslavik/winsparkle](https://github.com/vslavik/winsparkle)

## Initialization

### `win_sparkle_init()`

```c
void win_sparkle_init();
```

Starts WinSparkle. Performs automatic update check if configured. Non-blocking; update UI appears asynchronously from a separate thread.

**Important**: Call after the app's main window is shown. WinSparkle may display UI immediately. All configuration functions must be called before this.

### `win_sparkle_cleanup()`

```c
void win_sparkle_cleanup();
```

Cleans up after WinSparkle. Cancels pending operations and stops helper threads. Call during application shutdown.

## Language Settings

### `win_sparkle_set_lang()`

```c
void win_sparkle_set_lang(const char *lang);
```

Sets UI language from ISO 639 language code with optional ISO 3166 country code.

**Examples**: `"fr"`, `"pt-PT"`, `"zh-Hans"`

Must be called before `win_sparkle_init()`.

### `win_sparkle_set_langid()`

```c
void win_sparkle_set_langid(unsigned short lang);
```

Sets UI language from Win32 LANGID code created by `MAKELANGID()` macro.

Must be called before `win_sparkle_init()`.

## Configuration

### `win_sparkle_set_appcast_url()`

```c
void win_sparkle_set_appcast_url(const char *url);
```

Sets the URL of the appcast feed. Supports HTTP/HTTPS (HTTPS strongly recommended).

If not called, WinSparkle reads the URL from a Windows resource named `"FeedURL"` of type `"APPCAST"`:

```rc
FeedURL APPCAST "https://example.com/appcast.xml"
```

Must be called before `win_sparkle_init()`.

### `win_sparkle_set_eddsa_public_key()`

```c
int win_sparkle_set_eddsa_public_key(const char *pubkey);
```

Sets EdDSA (Ed25519) public key in base64 format for verifying update signatures.

**Returns**: 1 on valid key, 0 otherwise.

If set, DSA keys and signatures are ignored. Can also be set via Windows resource:

```rc
EdDSAPub EDDSA {"pXAx0wfi8kGbeQln11+V4R3tCepSuLXeo7LkOeudc/U="}
```

### `win_sparkle_set_dsa_pub_pem()` (DEPRECATED)

```c
int win_sparkle_set_dsa_pub_pem(const char *dsa_pub_pem);
```

Sets DSA public key in PEM format. **Deprecated** — migrate to EdDSA.

**Returns**: 1 on valid key, 0 otherwise.

### `win_sparkle_set_app_details()`

```c
void win_sparkle_set_app_details(const wchar_t *company_name,
                                  const wchar_t *app_name,
                                  const wchar_t *app_version);
```

Sets application metadata. Normally read from VERSIONINFO resources. Also determines registry path: `HKCU\Software\<company_name>\<app_name>\WinSparkle`.

Must be called before `win_sparkle_init()`.

### `win_sparkle_set_app_build_version()`

```c
void win_sparkle_set_app_build_version(const wchar_t *build);
```

Sets internal build version number for more granular comparisons. When used, appcast items must include `sparkle:shortVersionString` attribute for display, while `sparkle:version` is used for comparison.

Must be called before `win_sparkle_init()`.

### `win_sparkle_set_registry_path()`

```c
void win_sparkle_set_registry_path(const char *path);
```

Sets custom registry path for WinSparkle settings. Relative to HKCU/HKLM.

**Example**: `"Software\\My App\\Updates"`

Must be called before `win_sparkle_init()`.

### `win_sparkle_set_config_methods()`

```c
typedef struct win_sparkle_config_methods_tag {
    int  (__cdecl *config_read)(const char *name, wchar_t *buf, size_t len, void *user_data);
    void (__cdecl *config_write)(const char *name, const wchar_t *value, void *user_data);
    void (__cdecl *config_delete)(const char *name, void *user_data);
    void *user_data;
} win_sparkle_config_methods_t;

void win_sparkle_set_config_methods(win_sparkle_config_methods_t *config_methods);
```

Overrides default registry-based configuration storage with custom read/write/delete functions. All callbacks must be thread-safe.

`config_read` returns 1 if value was read successfully, 0 otherwise.

Must be called before `win_sparkle_init()`.

### `win_sparkle_set_http_header()`

```c
void win_sparkle_set_http_header(const char *name, const char *value);
```

Adds a custom HTTP header sent with appcast checks and update downloads. Can be called multiple times for different headers.

### `win_sparkle_clear_http_headers()`

```c
void win_sparkle_clear_http_headers();
```

Clears all previously added custom HTTP headers.

## Update Check Settings

### `win_sparkle_set_automatic_check_for_updates()`

```c
void win_sparkle_set_automatic_check_for_updates(int state);
```

Enables (1) or disables (0) automatic update checking.

### `win_sparkle_get_automatic_check_for_updates()`

```c
int win_sparkle_get_automatic_check_for_updates();
```

**Returns**: 1 if enabled, 0 if disabled. Defaults to 0 on first launch (user is asked on second launch).

### `win_sparkle_set_update_check_interval()`

```c
void win_sparkle_set_update_check_interval(int interval);
```

Sets automatic update check interval in seconds. Minimum: 3600 (1 hour).

### `win_sparkle_get_update_check_interval()`

```c
int win_sparkle_get_update_check_interval();
```

**Returns**: Check interval in seconds. Default: 86400 (1 day).

### `win_sparkle_get_last_check_time()`

```c
time_t win_sparkle_get_last_check_time();
```

**Returns**: Timestamp of last update check, or -1 if never checked.

## Manual Update Checks

### `win_sparkle_check_update_with_ui()`

```c
void win_sparkle_check_update_with_ui();
```

Checks for updates with full progress UI. Shows "no update available" if none found, or standard update dialog if found. Ignores "skip this version" preference. Returns immediately (non-blocking).

Use for "Check for Updates" menu items.

### `win_sparkle_check_update_with_ui_and_install()`

```c
void win_sparkle_check_update_with_ui_and_install();
```

Checks for updates and automatically installs if found, skipping the user prompt. Returns immediately.

Use for applications that require the latest version (e.g., forced updates).

### `win_sparkle_check_update_without_ui()`

```c
void win_sparkle_check_update_without_ui();
```

Silent update check. Shows update dialog only if an update is found. Respects "skip this version" preference. Returns immediately.

Use for silent background checks outside the normal automatic schedule.

## Callbacks

All callbacks are called from a **non-main thread**. Implementations must be thread-safe.

All callback setters must be called **before** `win_sparkle_init()`.

### `win_sparkle_set_error_callback()`

```c
typedef void (__cdecl *win_sparkle_error_callback_t)();
void win_sparkle_set_error_callback(win_sparkle_error_callback_t callback);
```

Invoked when the updater encounters an error (network, parse, signature verification, etc.).

### `win_sparkle_set_can_shutdown_callback()`

```c
typedef int (__cdecl *win_sparkle_can_shutdown_callback_t)();
void win_sparkle_set_can_shutdown_callback(win_sparkle_can_shutdown_callback_t callback);
```

Called to query whether the application can safely shut down for the update. Return `TRUE` (non-zero) if safe, `FALSE` (0) otherwise.

### `win_sparkle_set_shutdown_request_callback()`

```c
typedef void (__cdecl *win_sparkle_shutdown_request_callback_t)();
void win_sparkle_set_shutdown_request_callback(win_sparkle_shutdown_request_callback_t callback);
```

Called to request application shutdown after the installer is launched. Only called if `can_shutdown_callback` returned `TRUE`.

### `win_sparkle_set_did_find_update_callback()`

```c
typedef void (__cdecl *win_sparkle_did_find_update_callback_t)();
void win_sparkle_set_did_find_update_callback(win_sparkle_did_find_update_callback_t callback);
```

Invoked when an update is found. Particularly useful with `check_update_with_ui_and_install()`.

### `win_sparkle_set_did_not_find_update_callback()`

```c
typedef void (__cdecl *win_sparkle_did_not_find_update_callback_t)();
void win_sparkle_set_did_not_find_update_callback(win_sparkle_did_not_find_update_callback_t callback);
```

Invoked when no update is found.

### `win_sparkle_set_update_cancelled_callback()`

```c
typedef void (__cdecl *win_sparkle_update_cancelled_callback_t)();
void win_sparkle_set_update_cancelled_callback(win_sparkle_update_cancelled_callback_t callback);
```

Invoked when the user cancels the update (closes dialog, clicks Skip, or cancels download). Not called on errors or when no update exists.

### `win_sparkle_set_update_skipped_callback()`

```c
typedef void (__cdecl *win_sparkle_update_skipped_callback_t)();
void win_sparkle_set_update_skipped_callback(win_sparkle_update_skipped_callback_t callback);
```

Invoked when user clicks "Skip This Version".

### `win_sparkle_set_update_postponed_callback()`

```c
typedef void (__cdecl *win_sparkle_update_postponed_callback_t)();
void win_sparkle_set_update_postponed_callback(win_sparkle_update_postponed_callback_t callback);
```

Invoked when user clicks "Remind Me Later".

### `win_sparkle_set_update_dismissed_callback()`

```c
typedef void (__cdecl *win_sparkle_update_dismissed_callback_t)();
void win_sparkle_set_update_dismissed_callback(win_sparkle_update_dismissed_callback_t callback);
```

Invoked when the update dialog is closed for any reason (including no updates found or error).

### `win_sparkle_set_user_run_installer_callback()`

```c
typedef int (__cdecl *win_sparkle_user_run_installer_callback_t)(const wchar_t *path);
void win_sparkle_set_user_run_installer_callback(win_sparkle_user_run_installer_callback_t callback);
```

Invoked when the update payload has been downloaded and is ready to run. The `path` parameter points to the downloaded file.

**Returns**:
- `1` — callback handled the installer launch
- `0` — WinSparkle should handle it with default behavior
- `WINSPARKLE_RETURN_ERROR` — an error occurred

## Complete Integration Example (C++)

```cpp
#include <winsparkle.h>
#include <windows.h>

// Callbacks
int __cdecl canShutdown() {
    // Check if there's unsaved work, etc.
    return TRUE;
}

void __cdecl shutdownRequest() {
    PostMessage(g_hMainWnd, WM_CLOSE, 0, 0);
}

void InitUpdater(HWND hMainWnd) {
    // Configure (all before init)
    win_sparkle_set_appcast_url("https://example.com/appcast.xml");
    win_sparkle_set_eddsa_public_key("pXAx0wfi8kGbeQln11+V4R3tCepSuLXeo7LkOeudc/U=");
    win_sparkle_set_automatic_check_for_updates(1);
    win_sparkle_set_update_check_interval(86400); // daily

    // Set shutdown callbacks
    win_sparkle_set_can_shutdown_callback(canShutdown);
    win_sparkle_set_shutdown_request_callback(shutdownRequest);

    // Initialize
    win_sparkle_init();
}

void CleanupUpdater() {
    win_sparkle_cleanup();
}

// Menu handler for "Check for Updates..."
void OnCheckForUpdates() {
    win_sparkle_check_update_with_ui();
}
```

## Complete Integration Example (C#)

```csharp
public partial class MainForm : Form
{
    public MainForm()
    {
        InitializeComponent();

        WinSparkle.win_sparkle_set_appcast_url("https://example.com/appcast.xml");
        WinSparkle.win_sparkle_init();
    }

    private void checkForUpdatesMenuItem_Click(object sender, EventArgs e)
    {
        WinSparkle.win_sparkle_check_update_with_ui();
    }

    protected override void OnFormClosed(FormClosedEventArgs e)
    {
        WinSparkle.win_sparkle_cleanup();
        base.OnFormClosed(e);
    }
}
```
