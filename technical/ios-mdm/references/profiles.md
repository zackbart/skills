# iOS Configuration Profiles Reference

## Table of Contents

- [Profile Structure](#profile-structure)
- [WiFi Payload](#wifi-payload-comapplewifimanaged)
- [VPN Payload](#vpn-payload-comapplevpnmanaged)
- [Web Content Filter](#web-content-filter-comapplewebcontent-filter)
- [Passcode Policy](#passcode-policy-comapplemobiledevicepasswordpolicy)
- [App Lock](#app-lock-comappleapplock--supervised--ade-only)
- [DNS Settings](#dns-settings-comapplednssettingsmanaged)
- [Certificate Payloads](#certificate-payloads)
- [Other Notable Payloads](#other-notable-payloads)
- [Profile Management Best Practices](#profile-management-best-practices)

## Profile Structure

A configuration profile is an XML property list (plist) file with MIME type `application/x-apple-aspen-config`. The top-level dictionary defines the profile itself, and the `PayloadContent` array holds one or more payload dictionaries.

### Top-Level Keys

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `PayloadType` | String | Yes | Must be `"Configuration"` |
| `PayloadVersion` | Integer | Yes | Must be `1` |
| `PayloadIdentifier` | String | Yes | Reverse-DNS identifier, e.g. `com.example.profile.wifi` |
| `PayloadUUID` | String | Yes | Globally unique UUID for this profile |
| `PayloadDisplayName` | String | No | Human-readable name shown to the user |
| `PayloadDescription` | String | No | Description shown during install |
| `PayloadOrganization` | String | No | Organization name shown during install |
| `PayloadRemovalDisallowed` | Boolean | No | If `true`, profile cannot be removed (supervised only on iOS) |
| `PayloadScope` | String | No | `"User"` or `"System"` (macOS only; iOS profiles are always device-scoped) |
| `RemovalDate` | Date | No | Date the profile is automatically removed |
| `DurationUntilRemoval` | Float | No | Seconds until automatic removal |
| `TargetDeviceType` | Integer | No | `5` = iPhone/iPod, `6` = iPad |
| `ConsentText` | Dictionary | No | Localized consent text, keyed by locale (e.g. `"en"`, `"default"`) |
| `PayloadContent` | Array | Yes | Array of payload dictionaries |

### Common Payload Keys

Every payload dictionary within `PayloadContent` must include:

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `PayloadType` | String | Yes | Payload-specific type identifier |
| `PayloadVersion` | Integer | Yes | Typically `1` |
| `PayloadIdentifier` | String | Yes | Unique identifier for this payload |
| `PayloadUUID` | String | Yes | Globally unique UUID for this payload |
| `PayloadDisplayName` | String | No | Human-readable name for this payload |

### Example: Complete Profile with WiFi Payload

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>PayloadType</key>
  <string>Configuration</string>
  <key>PayloadVersion</key>
  <integer>1</integer>
  <key>PayloadIdentifier</key>
  <string>com.example.profile.wifi</string>
  <key>PayloadUUID</key>
  <string>A1B2C3D4-E5F6-7890-ABCD-EF1234567890</string>
  <key>PayloadDisplayName</key>
  <string>Corporate WiFi</string>
  <key>PayloadDescription</key>
  <string>Configures WiFi access for the corporate network.</string>
  <key>PayloadOrganization</key>
  <string>Example Corp</string>
  <key>PayloadContent</key>
  <array>
    <dict>
      <key>PayloadType</key>
      <string>com.apple.wifi.managed</string>
      <key>PayloadVersion</key>
      <integer>1</integer>
      <key>PayloadIdentifier</key>
      <string>com.example.profile.wifi.managed</string>
      <key>PayloadUUID</key>
      <string>B2C3D4E5-F6A7-8901-BCDE-F12345678901</string>
      <key>PayloadDisplayName</key>
      <string>Corporate WiFi Settings</string>
      <key>SSID_STR</key>
      <string>CorpNet</string>
      <key>HIDDEN_NETWORK</key>
      <false/>
      <key>AutoJoin</key>
      <true/>
      <key>EncryptionType</key>
      <string>WPA2</string>
      <key>Password</key>
      <string>SecurePassword123</string>
    </dict>
  </array>
</dict>
</plist>
```

## WiFi Payload (com.apple.wifi.managed)

### Key Fields

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `SSID_STR` | String | Yes | Network SSID |
| `HIDDEN_NETWORK` | Boolean | No | `true` if network is not broadcasting |
| `AutoJoin` | Boolean | No | Automatically join this network (default `true`) |
| `EncryptionType` | String | No | `WEP`, `WPA`, `WPA2`, `WPA3`, `Any`, or `None` |
| `Password` | String | No | Network password (for personal networks) |
| `ProxyType` | String | No | `None`, `Manual`, or `Auto` |
| `ProxyServer` | String | No | Proxy hostname (when `ProxyType` is `Manual`) |
| `ProxyPort` | Integer | No | Proxy port number |

### Enterprise Configuration

For 802.1X enterprise networks, include an `EAPClientConfiguration` dictionary:

| Key | Type | Description |
|-----|------|-------------|
| `AcceptEAPTypes` | Array of Integer | EAP methods: `13` = TLS, `21` = TTLS, `25` = PEAP, `43` = EAP-FAST |
| `UserName` | String | Username for authentication |
| `PayloadCertificateAnchorUUID` | Array of String | UUIDs of trusted anchor certificates (from Certificate payloads) |
| `TLSTrustedServerNames` | Array of String | Expected RADIUS server common names |

### Example: WPA2 Enterprise with EAP-TLS

```xml
<dict>
  <key>PayloadType</key>
  <string>com.apple.wifi.managed</string>
  <key>PayloadVersion</key>
  <integer>1</integer>
  <key>PayloadIdentifier</key>
  <string>com.example.wifi.enterprise</string>
  <key>PayloadUUID</key>
  <string>C3D4E5F6-A7B8-9012-CDEF-123456789012</string>
  <key>SSID_STR</key>
  <string>CorpSecure</string>
  <key>EncryptionType</key>
  <string>WPA2</string>
  <key>EAPClientConfiguration</key>
  <dict>
    <key>AcceptEAPTypes</key>
    <array>
      <integer>13</integer>
    </array>
    <key>PayloadCertificateAnchorUUID</key>
    <array>
      <string>D4E5F6A7-B8C9-0123-DEFA-234567890123</string>
    </array>
    <key>TLSTrustedServerNames</key>
    <array>
      <string>radius.example.com</string>
    </array>
  </dict>
</dict>
```

## VPN Payload (com.apple.vpn.managed)

### VPN Types

| `VPNType` | Description |
|-----------|-------------|
| `L2TP` | L2TP over IPSec |
| `IPSec` | Cisco IPSec |
| `IKEv2` | IKEv2 (recommended for modern deployments) |
| `AlwaysOn` | Always On VPN (supervised devices only) |

### IKEv2 Configuration Keys

| Key | Type | Description |
|-----|------|-------------|
| `RemoteAddress` | String | VPN server hostname or IP |
| `RemoteIdentifier` | String | Server identity (FQDN, UserFQDN, Address, or ASN1DN) |
| `LocalIdentifier` | String | Client identity sent to the server |
| `AuthenticationMethod` | String | `SharedSecret`, `Certificate`, or `None` |
| `SharedSecret` | String | Pre-shared key (when using SharedSecret auth) |
| `PayloadCertificateUUID` | String | UUID of identity certificate payload (when using Certificate auth) |

### On Demand Rules

The `OnDemandRules` array controls automatic VPN connection. Each rule dictionary contains:

| Key | Type | Description |
|-----|------|-------------|
| `Action` | String | `Connect`, `Disconnect`, `EvaluateConnection`, or `Ignore` |
| `SSIDMatch` | Array of String | Match when connected to these WiFi SSIDs |
| `DNSDomainMatch` | Array of String | Match when device's DNS search domain matches |
| `InterfaceTypeMatch` | String | `WiFi`, `Cellular`, or `Ethernet` |
| `URLStringProbe` | String | URL to probe; if unreachable, the rule matches |

Rules are evaluated in order; the first match wins. Always include a final catch-all rule.

### Always On VPN (Supervised Only)

Available only on supervised devices. Uses separate tunnel configurations for cellular (`TunnelConfigurations`) and WiFi. Ensures all IP traffic is routed through the VPN. Supports `ServiceExceptions` for captive portal detection and voicemail.

### Per-App VPN (com.apple.vpn.managed.applayer)

Routes traffic from specific managed apps through the VPN tunnel:

| Key | Type | Description |
|-----|------|-------------|
| `VPNUUID` | String | UUID reference to the parent VPN payload |
| `SafariDomains` | Array of String | Domains that trigger the per-app VPN in Safari |
| `DesignatedRequirement` | String | Code signing requirement for apps using this VPN |

## Web Content Filter (com.apple.webcontent-filter)

### FilterType: BuiltIn

Apple's built-in content filter (Safari only):

| Key | Type | Description |
|-----|------|-------------|
| `FilterType` | String | `BuiltIn` |
| `AutoFilterEnabled` | Boolean | Enable automatic content filtering |
| `PermittedURLs` | Array of String | URLs always allowed regardless of filter (allowlist) |
| `WhitelistedBookmarks` | Array of Dict | Bookmarks added to Safari that bypass the filter |
| `BlacklistedURLs` | Array of String | URLs that are always blocked |

> **Note:** The BuiltIn filter only applies to Safari. It does not filter traffic from other apps or browsers.

### FilterType: Plugin

Third-party content filter that can inspect all network traffic:

| Key | Type | Description |
|-----|------|-------------|
| `FilterType` | String | `Plugin` |
| `FilterBrowsers` | Boolean | Filter WebKit-based browser traffic |
| `FilterSockets` | Boolean | Filter socket-level (non-browser) traffic |
| `FilterDataProviderBundleIdentifier` | String | Bundle ID of the filter data provider app extension |
| `FilterDataProviderDesignatedRequirement` | String | Code signing requirement for the filter extension |
| `ServerAddress` | String | Server hostname for the filter service |
| `UserName` | String | Username for filter service authentication |
| `Password` | String | Password for filter service authentication |
| `Organization` | String | Organization name passed to the filter provider |
| `FilterPackets` | Boolean | Enable packet-level filtering (network extension) |

## Passcode Policy (com.apple.mobiledevice.passwordpolicy)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `allowSimple` | Boolean | `true` | Allow simple passcodes (e.g. 1234, abcd) |
| `forcePIN` | Boolean | `false` | Require a passcode on the device |
| `maxFailedAttempts` | Integer | 11 | Failed attempts before device wipe (2--11) |
| `maxInactivity` | Integer | none | Minutes of inactivity before auto-lock |
| `maxPINAgeInDays` | Integer | none | Days until passcode must be changed |
| `minComplexChars` | Integer | 0 | Minimum number of complex (non-alphanumeric) characters |
| `minLength` | Integer | 0 | Minimum passcode length |
| `requireAlphanumeric` | Boolean | `false` | Require at least one letter and one number |
| `pinHistory` | Integer | none | Number of unique passcodes before reuse is allowed |
| `maxGracePeriod` | Integer | 0 | Minutes the device can be unlocked without passcode after locking |
| `changeAtNextAuth` | Boolean | `false` | Force passcode change at next authentication |

## App Lock (com.apple.app.lock) -- Supervised + ADE Only

Locks the device to a single app (Single App Mode). Requires supervised device enrolled through Automated Device Enrollment (ADE).

### Configuration

The payload contains an `App` dictionary:

| Key | Type | Description |
|-----|------|-------------|
| `Identifier` | String | Bundle ID of the app to lock to (e.g. `com.example.kiosk`) |

### Options Dictionary

Boolean keys to disable hardware controls while in Single App Mode:

- `DisableTouch` -- Disable touch input on the screen
- `DisableDeviceRotation` -- Lock screen orientation
- `DisableVolumeButtons` -- Disable hardware volume buttons
- `DisableRingerSwitch` -- Disable the ringer/silent switch
- `DisableSleepWakeButton` -- Disable the side/power button
- `DisableAutoLock` -- Prevent the device from auto-locking
- `EnableVoiceOver` -- Turn on VoiceOver accessibility
- `EnableZoom` -- Turn on Zoom accessibility
- `EnableAssistiveTouch` -- Turn on AssistiveTouch
- `EnableInvertColors` -- Turn on color inversion
- `EnableMonoAudio` -- Turn on mono audio output
- `EnableSpeakSelection` -- Turn on Speak Selection

### Autonomous Single App Mode

The `UserEnabledOptions` dictionary allows the app itself to enter and exit Single App Mode programmatically (via the `UIAccessibilityRequestGuidedAccessSession` API). Uses the same boolean keys as `Options`.

## DNS Settings (com.apple.dnsSettings.managed)

Configures encrypted DNS (DNS over HTTPS or DNS over TLS).

| Key | Type | Description |
|-----|------|-------------|
| `DNSProtocol` | String | `HTTPS` (DoH) or `TLS` (DoT) |
| `ServerURL` | String | DoH server URL (required when `DNSProtocol` is `HTTPS`) |
| `ServerName` | String | DoT server hostname (required when `DNSProtocol` is `TLS`) |
| `ServerAddresses` | Array of String | IP addresses of DNS servers (bootstrap addresses) |
| `SupplementalMatchDomains` | Array of String | Domains resolved using this DNS configuration (empty = all domains) |
| `OnDemandRules` | Array of Dict | Rules to selectively enable/disable DNS settings by network |

## Certificate Payloads

### com.apple.security.pem -- CA and Root Certificates

| Key | Type | Description |
|-----|------|-------------|
| `PayloadCertificateFileName` | String | Filename for the certificate (display purposes) |
| `PayloadContent` | Data | DER-encoded certificate data |

### com.apple.security.pkcs12 -- Identity Certificates

| Key | Type | Description |
|-----|------|-------------|
| `PayloadCertificateFileName` | String | Filename for the certificate |
| `PayloadContent` | Data | PKCS#12 (.p12) data containing certificate and private key |
| `Password` | String | Password to decrypt the PKCS#12 container |

### com.apple.security.scep -- SCEP Enrollment

| Key | Type | Description |
|-----|------|-------------|
| `URL` | String | SCEP server URL |
| `Name` | String | CA name (as understood by the SCEP server) |
| `Subject` | Array of Array | X.500 subject in OID/value pairs, e.g. `[["O", "Example"], ["CN", "Device"]]` |
| `Challenge` | String | SCEP challenge password |
| `KeySize` | Integer | Key size in bits (`1024`, `2048`, `4096`) |
| `KeyType` | String | `RSA` |
| `KeyUsage` | Integer | Bitmask: `1` = signing, `4` = encryption, `5` = both |

### com.apple.security.acme -- ACME Certificate Issuance

Automates certificate issuance through an ACME server. Requires `DirectoryURL`, `ClientIdentifier`, `Subject`, and `KeySize`/`KeyType`. The device completes the ACME challenge directly with the server.

## Other Notable Payloads

**Email / Exchange (com.apple.eas.account, com.apple.mail.managed)**
Configures Exchange ActiveSync or IMAP/POP email accounts. Keys include `EmailAddress`, `Host`, `IncomingMailServerHostName`, `OutgoingMailServerHostName`, `IncomingMailServerAuthentication`, and OAuth support.

**Notifications (com.apple.notificationsettings)**
Per-app notification control. The `NotificationSettings` array contains dictionaries with `BundleIdentifier`, `NotificationsEnabled`, `ShowInNotificationCenter`, `ShowInLockScreen`, `AlertType` (0=none, 1=banner, 2=modal), and `BadgesEnabled`.

**Home Screen Layout (com.apple.homescreenlayout)** -- Supervised only
Forces icon arrangement on the home screen. Specify pages as arrays of arrays containing bundle IDs. Supports folders and dock layout.

**Domains (com.apple.domains)**
Defines managed Safari domains and associated domains. `WebDomains` specifies domains treated as managed (documents from these domains are managed). `SafariPasswordAutoFillDomains` controls AutoFill behavior.

**Web Clip (com.apple.webClip.managed)**
Creates a home screen shortcut to a URL. Keys include `URL`, `Label`, `Icon` (PNG data), `IsRemovable`, and `FullScreen`.

**Global HTTP Proxy (com.apple.proxy.http.global)** -- Supervised only
Routes all HTTP/HTTPS traffic through a proxy. Keys include `ProxyType` (`Manual` or `Auto`), `ProxyServer`, `ProxyServerPort`, `ProxyPACURL`, and `ProxyCaptiveLoginAllowed`.

**SSO Extension (com.apple.extensiblesso)**
Configures the Extensible SSO framework. Supports `Redirect` and `Credential` types. Keys include `ExtensionIdentifier`, `TeamIdentifier`, `URLs`, `Realm`, and `Hosts`. Used with Kerberos SSO extensions and third-party identity providers.

**Cellular (com.apple.cellular)**
Configures APN settings for cellular data. Keys include `APNs` array with `DefaultProtocolMask`, `Name`, `AuthenticationType`, `Username`, and `Password`.

**Shared Device Configuration (com.apple.shareddeviceconfiguration)** -- Shared iPad
Configures Shared iPad behavior including `QuotaSize` (per-user storage), `ResidentUsers` (max cached users), and `OnlyTemporaryUsers` for guest mode.

## Profile Management Best Practices

- **One profile per concern**: Keep WiFi, VPN, restrictions, and certificates in separate profiles. This allows independent updates and removal without affecting other configurations.
- **Use unique PayloadIdentifier values**: Installing a profile with the same `PayloadIdentifier` as an existing profile replaces the old one entirely. This is the mechanism for updating profiles.
- **PayloadRemovalDisallowed limitations**: On iOS and tvOS, this key only prevents removal on supervised devices. On unsupervised devices, users can always remove profiles (though MDM can re-push them).
- **MDM-installed profiles and backup/restore**: Profiles installed by MDM are not included in device backups. After a backup/restore, the device must re-enroll in MDM to receive profiles again.
- **Sign profiles for trust**: Use a code signing certificate from the Apple Developer portal to sign profiles. Signed profiles show a "Verified" badge during installation and prevent tampering.
- **Use PayloadScope appropriately**: On macOS, `System`-scoped profiles apply device-wide, while `User`-scoped profiles apply per-user. iOS profiles are always device-scoped.
- **Test with Apple Configurator**: Before deploying profiles at scale via MDM, validate them with Apple Configurator 2 on a test device.
- **Profile expiration**: Use `RemovalDate` or `DurationUntilRemoval` for temporary configurations such as guest WiFi access or time-limited VPN credentials.
