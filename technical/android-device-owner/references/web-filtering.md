# Web Content Filtering

Android has **no built-in OS-level URL filtering API**. Content filtering must be implemented through one of these approaches.

## Approach 1: Chrome Managed Configurations (Simplest)

Use `setApplicationRestrictions()` to push URL policies to Chrome. Chrome-only -- does not cover other browsers or in-app WebViews.

```kotlin
val restrictions = Bundle().apply {
    putStringArray("URLBlocklist", arrayOf(
        "https://www.facebook.com",
        "twitter.com",
        "*.tiktok.com",
        "reddit.com/r/funny"
    ))
    putStringArray("URLAllowlist", arrayOf(
        "https://allowed.example.com",
        "mail.google.com"
    ))
    // Optional: disable incognito to prevent bypass
    putString("IncognitoModeAvailability", "1")  // 0=available, 1=disabled, 2=forced
    // Optional: force SafeSearch
    putString("ForceGoogleSafeSearch", "true")
}
dpm.setApplicationRestrictions(adminName, "com.android.chrome", restrictions)
```

**URL format:** `[scheme://][.]host[:port][/path][@query]`

| Pattern | Blocks |
|---------|--------|
| `example.com` | All pages on example.com |
| `.example.com` | example.com and all subdomains |
| `https://example.com` | Only HTTPS on example.com |
| `example.com/path` | Specific path |
| `*` | All URLs (use with allowlist) |

**Key behaviors:**
- Allowlist takes precedence over blocklist
- Maximum ~1,000 entries per list (performance degrades beyond this)
- Keys renamed from `URLBlacklist`/`URLWhitelist` in Chrome 86+
- Does not filter in-app browsers, WebViews, or other browser apps

## Approach 2: VPN-Based DNS Filtering (Comprehensive)

Intercept all DNS queries via a custom `VpnService` and block at the DNS level. This covers all apps, not just Chrome.

### Architecture

```
App makes DNS query
  -> Android routes to VPN TUN interface
  -> Your VpnService reads DNS packet
  -> Check domain against block/allow list
  -> Blocked: respond with 0.0.0.0 (NXDOMAIN or null IP)
  -> Allowed: forward to upstream DNS (e.g., 8.8.8.8)
  -> Write response back to TUN
```

### VpnService API Overview

The Android `VpnService` API (API 14+) allows apps to create a virtual network interface (TUN) that intercepts device traffic. For DNS filtering, the VPN intercepts DNS queries and selectively blocks or forwards them.

Key `VpnService.Builder` methods:

```kotlin
class MyVpnService : VpnService() {
    fun setupTunnel(): ParcelFileDescriptor? {
        return Builder()
            .addAddress(tunnelAddress, prefixLength)  // VPN interface address
            .addDnsServer(dnsServerAddress)            // DNS server for the VPN
            .addRoute(routeAddress, prefixLength)      // Traffic to route through TUN
            .setBlocking(true)                         // Block reads until data available
            .setMtu(1500)                              // Standard MTU
            .establish()                               // Returns TUN file descriptor
    }
}
```

**Per-app VPN** (API 30+): Route only specific apps through the VPN:
```kotlin
Builder()
    .addAllowedApplication("com.example.browser")     // Only this app routes through VPN
    // OR
    .addDisallowedApplication("com.example.trusted")   // All apps except this one
```

**Reference implementations** for DNS-based filtering architecture:
- **Android ToyVPN sample** (`android/platform-development/samples/ToyVpn`) -- official Android VPN sample demonstrating `VpnService` usage
- **NetGuard** (`M66B/NetGuard` on GitHub) -- open-source no-root firewall using VPN-based traffic filtering

### DNS Filtering Considerations

When building a DNS filter via VPN:

- **Intercept DNS (UDP port 53)** on the TUN interface and inspect the query domain
- **Blocked domains**: Respond with `0.0.0.0` (A record) or `::` (AAAA record) so the connection fails cleanly
- **Allowed domains**: Forward the query to an upstream DNS resolver and return the real response
- **System-critical domains**: Always allow Google Play Services domains (`googleapis.com`, `gstatic.com`, `android.com`, `google.com`) to avoid breaking core device functionality
- **Thread safety**: DNS queries arrive on the VPN reader thread; policy updates come from other threads. Use thread-safe data structures (`ConcurrentHashMap`, `AtomicReference`, or `@Volatile`) for shared filter state

### Lock the VPN

Prevent the user from disabling or changing the VPN:

```kotlin
// Force all traffic through VPN; lockdown=true means no traffic if VPN disconnects
dpm.setAlwaysOnVpnPackage(adminName, context.packageName, true)

// Prevent user from changing VPN settings
dpm.addUserRestriction(adminName, UserManager.DISALLOW_CONFIG_VPN)
```

### Prevent DNS Bypass

Private DNS (DoH/DoT) allows browsers to bypass your DNS filter. Disable it:

```kotlin
dpm.setGlobalSetting(adminName, "private_dns_mode", "off")
```

Without this, Chrome and other apps can resolve domains directly via HTTPS to Google/Cloudflare DNS, completely bypassing your VPN-based filter.

## Approach 3: HTTP Proxy (Limited)

Set a global proxy that points to a filtering proxy server:

```kotlin
val proxyInfo = ProxyInfo.buildDirectProxy("proxy.example.com", 8080)
dpm.setRecommendedGlobalProxy(adminName, proxyInfo)

// Or PAC file
val proxyInfo = ProxyInfo.buildPacProxy(Uri.parse("https://example.com/proxy.pac"))
dpm.setRecommendedGlobalProxy(adminName, proxyInfo)
```

**Limitations:**
- Only covers HTTP/HTTPS traffic that respects system proxy
- Many apps ignore the system proxy
- Does not cover DNS-level requests
- "Recommended" -- not enforced on all network types

## Comparison

| Approach | Coverage | Complexity | Bypass Risk |
|----------|----------|------------|-------------|
| Chrome managed configs | Chrome only | Low | High (other browsers) |
| VPN DNS filtering | All apps | High | Low (with lockdown) |
| HTTP proxy | HTTP/HTTPS only | Medium | Medium (apps can ignore) |

## Combining Approaches

The Android Enterprise documentation supports layering these documented APIs together:

1. **`setApplicationRestrictions()`** for Chrome URL policies (documented in [Chrome Enterprise policy list](https://chromeenterprise.google/policies/))
2. **`VpnService`** for device-wide DNS filtering (documented in [Android VpnService API](https://developer.android.com/reference/android/net/VpnService))
3. **`setAlwaysOnVpnPackage()` with lockdown** to enforce VPN (documented in [Android Enterprise networking](https://developer.android.com/work/dpc/network-telephony))
4. **`DISALLOW_CONFIG_VPN`** user restriction to prevent user changes (documented in [UserManager](https://developer.android.com/reference/android/os/UserManager))
5. **`setGlobalSetting("private_dns_mode", "off")`** to prevent DoH/DoT bypass (documented in [DevicePolicyManager.setGlobalSetting](https://developer.android.com/reference/android/app/admin/DevicePolicyManager#setGlobalSetting))

Each of these is independently documented. The combination provides defense-in-depth for content filtering scenarios.

## Additional Chrome Managed Configuration Keys

Beyond URL filtering, device owners can configure Chrome behavior:

| Key | Type | Values |
|-----|------|--------|
| `URLBlocklist` | String array | URL patterns to block |
| `URLAllowlist` | String array | URL patterns to allow (overrides blocklist) |
| `IncognitoModeAvailability` | String | "0" (available), "1" (disabled), "2" (forced) |
| `ForceGoogleSafeSearch` | String | "true" / "false" |
| `HomepageLocation` | String | URL for Chrome homepage |
| `BookmarkBarEnabled` | String | "true" / "false" |
| `DefaultSearchProviderEnabled` | String | "true" / "false" |
| `PasswordManagerEnabled` | String | "true" / "false" |
| `TranslateEnabled` | String | "true" / "false" |
| `DefaultBrowserSettingEnabled` | String | "true" / "false" |

Source: [Chrome Enterprise policy list](https://chromeenterprise.google/policies/) -- any Chrome policy can be pushed as a managed configuration key via `setApplicationRestrictions()`.
