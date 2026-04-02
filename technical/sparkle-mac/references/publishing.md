# Publishing Updates with Sparkle

## Table of Contents

1. [Archiving Your App](#archiving-your-app)
2. [Signing Updates](#signing-updates)
3. [Appcast Format](#appcast-format)
4. [Versioning](#versioning)
5. [System Requirements](#system-requirements)
6. [Major Upgrades](#major-upgrades)
7. [Critical Updates](#critical-updates)
8. [Informational Updates (Sparkle 2)](#informational-updates-sparkle-2)
9. [Phased Group Rollouts](#phased-group-rollouts)
10. [Channels (Sparkle 2)](#channels-sparkle-2)
11. [Setting Feed URL Programmatically](#setting-feed-url-programmatically)
12. [Release Notes](#release-notes)
13. [Extending the Appcast](#extending-the-appcast)
14. [Delta Updates](#delta-updates)

## Archiving Your App

Use Xcode: Product > Archive > Distribute App > Developer ID.

### Archive Formats

**DMG (Recommended for website distribution):**
- APFS formatted with lzfse compression recommended
- Add `/Applications` symlink to encourage users to copy app out
- Notarize and Developer ID sign

**ZIP:**
```sh
ditto -c -k --sequesterRsrc --keepParent MyApp.app MyApp.zip
```
Avoid placing anything but the app inside to minimize app translocation issues.

**Tar:**
```sh
tar --no-xattrs -cJf MyApp.tar.xz MyApp.app
```

**Apple Archive (.aar):** (Sparkle 2.7+ / macOS 10.15+)
Requires `SUVerifyUpdateBeforeExtraction` enabled. See `man aa`.

**Critical:** Always preserve symlinks. macOS frameworks use them and code signatures break without them.

Sparkle supports updating from: dmg, zip, tarballs, Apple Archives, and installer packages.

## Signing Updates

### Automatic (Recommended)

Signatures are auto-generated when using `generate_appcast`.

### Manual EdDSA Signing

```sh
# Sign an update archive
./bin/sign_update path_to_your_update.(zip|dmg|tar.*)

# Output:
# sparkle:edSignature="7cLA..." length="1623481"
```

For apps with `SURequireSignedFeed`, also sign release notes:
```sh
./bin/sign_update path_to_your_releasenotes.html
# Output:
# sparkle:edSignature="4KjvG1..." sparkle:length="1467"
```

## Appcast Format

### Generating Appcasts Automatically

1. Build and archive your app into a folder
2. Run:
```sh
./bin/generate_appcast /path/to/your/updates_folder/
```
3. Generates appcast XML, delta updates, and signatures
4. Upload archives, deltas, and appcast to your server

The tool auto-discovers the filename from `SUFeedURL`. If an `.html` or `.md` file shares the archive name, it becomes the `releaseNotesLink`.

If `SURequireSignedFeed` is enabled, `generate_appcast` also signs the appcast and release note files.

### Manual Appcast Item

```xml
<item>
    <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>2.0</sparkle:version>
    <sparkle:shortVersionString>2.0</sparkle:shortVersionString>
    <sparkle:releaseNotesLink>
        https://example.com/release_notes/app_2.0.html
    </sparkle:releaseNotesLink>
    <pubDate>Mon, 05 Oct 2015 19:20:11 +0000</pubDate>
    <enclosure url="https://example.com/downloads/app.dmg"
               sparkle:edSignature="7cLALFUH..."
               length="1623481"
               type="application/octet-stream" />
</item>
```

## Versioning

### Internal Build Numbers

Use `CFBundleVersion` for machine-readable internal versions and `CFBundleShortVersionString` for human-readable display:

```xml
<item>
    <sparkle:version>1248</sparkle:version>
    <sparkle:shortVersionString>1.5.1</sparkle:shortVersionString>
    ...
</item>
```

`CFBundleVersion` / `sparkle:version` must be numeric and incrementing.

## System Requirements

### Minimum macOS Version

```xml
<sparkle:minimumSystemVersion>10.13.0</sparkle:minimumSystemVersion>
```

Use three-part version (major.minor.patch). Also supports `sparkle:maximumSystemVersion`.

### Hardware Requirements (Sparkle 2.9+)

```xml
<sparkle:hardwareRequirements>arm64</sparkle:hardwareRequirements>
```

### Minimum Update Version (Sparkle 2.9+)

Require users to be on a specific version before updating:

```xml
<sparkle:minimumUpdateVersion>1.9</sparkle:minimumUpdateVersion>
```

## Major Upgrades

Prevent automatic installation for major/paid upgrades:

```xml
<sparkle:minimumAutoupdateVersion>2.0</sparkle:minimumAutoupdateVersion>
```

Users below this version will always see the update GUI. Sparkle 2 will prefer installing the latest minor release before the major one.

### Ignore Skipped Upgrades (Sparkle 2.1+)

Re-notify users who skipped previous major versions:

```xml
<sparkle:ignoreSkippedUpgradesBelowVersion>2.2</sparkle:ignoreSkippedUpgradesBelowVersion>
```

## Critical Updates

Cannot be skipped by users, shown more promptly:

```xml
<!-- Sparkle 2 top-level syntax -->
<sparkle:criticalUpdate></sparkle:criticalUpdate>

<!-- With version targeting (only critical for versions below 1.2.4) -->
<sparkle:criticalUpdate sparkle:version="1.2.4"></sparkle:criticalUpdate>
```

Legacy syntax (still works):
```xml
<sparkle:tags>
    <sparkle:criticalUpdate></sparkle:criticalUpdate>
</sparkle:tags>
```

## Informational Updates (Sparkle 2)

Show a download link instead of auto-installing:

```xml
<!-- Informational for all versions -->
<sparkle:informationalUpdate></sparkle:informationalUpdate>

<!-- Informational only for specific versions -->
<sparkle:informationalUpdate>
    <sparkle:version>1.2.3</sparkle:version>
</sparkle:informationalUpdate>

<!-- Informational for all versions below 1.0 (Sparkle 2.1+) -->
<sparkle:informationalUpdate>
    <sparkle:belowVersion>1.0</sparkle:belowVersion>
</sparkle:informationalUpdate>
```

## Phased Group Rollouts

Gradually distribute updates to user groups:

```xml
<pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
<sparkle:phasedRolloutInterval>86400</sparkle:phasedRolloutInterval>
```

- 86400 seconds = 1 day between groups
- 7 hardcoded groups = full rollout in 7 days
- Not applied to critical updates or manual checks
- Group ID stored in `SUUpdateGroupIdentifier` user default

## Channels (Sparkle 2)

Branch updates to specific channels (e.g., beta):

```xml
<item>
    <title>Version 2.0 (Beta 1)</title>
    <sparkle:channel>beta</sparkle:channel>
    <sparkle:version>20001</sparkle:version>
    <sparkle:shortVersionString>2.0b1</sparkle:shortVersionString>
</item>
```

Opt in via delegate:
```swift
func allowedChannels(for updater: SPUUpdater) -> Set<String> {
    return Set(["beta"])
}
```

Default channel cannot be excluded. Channels are for branching, not parallel releases.

## Setting Feed URL Programmatically

Use delegate method (recommended):
```swift
func feedURLString(for updater: SPUUpdater) -> String? {
    return UserDefaults.standard.bool(forKey: "beta") ? BETA_FEED_URL_STRING : STABLE_FEED_URL_STRING
}
```

Migrate from deprecated `-setFeedURL:` by calling `clearFeedURLFromUserDefaults` after starting updater (Sparkle 2.4+).

## Release Notes

### External Link
```xml
<sparkle:releaseNotesLink>https://example.com/release_notes/app_2.0.html</sparkle:releaseNotesLink>
```

### Embedded HTML
```xml
<description><![CDATA[
    <h2>New Features</h2>
    <ul><li>Feature one</li></ul>
]]>
</description>
```

### Embedded Plain Text (Sparkle 2.4+)
```xml
<description sparkle:format="plain-text">Bug fixes and improvements</description>
```

### Embedded Markdown (Sparkle 2.9+ / macOS 12+)
```xml
<description sparkle:format="markdown">## What's New
- Feature one
- Bug fix two
</description>
```

### Localized Release Notes
```xml
<sparkle:releaseNotesLink xml:lang="en">https://example.com/app/2.0.en.html</sparkle:releaseNotesLink>
<sparkle:releaseNotesLink xml:lang="de">https://example.com/app/2.0.de.html</sparkle:releaseNotesLink>
```

### Full Release Notes (Version History)
```xml
<sparkle:fullReleaseNotesLink>https://myproductwebsite.com/app/full-history/</sparkle:fullReleaseNotesLink>
```

### Adapting HTML Release Notes (Sparkle 2.5+)

Sparkle auto-adds `sparkle-installed-version` CSS class to elements with matching `data-sparkle-version`:

```html
<div data-sparkle-version="3">Version 1.2: Update icon</div>
<div data-sparkle-version="2">Version 1.1: Fix list</div>
```

```css
div.sparkle-installed-version { opacity: 0.5; }
div.sparkle-installed-version ~ div { display: none; }
```

## Extending the Appcast

Custom elements via XML namespaces:

```xml
<rss version="2.0" xmlns:sparkle="http://www.andymatuschak.org/xml-namespaces/sparkle" xmlns:myapp="https://myproductpage.com/">
    <channel>
        <item>
            <myapp:messageOfTheDay>Tip: you can do X</myapp:messageOfTheDay>
            ...
        </item>
    </channel>
</rss>
```

Parse via `SUAppcastItem.propertiesDictionary["myapp:messageOfTheDay"]`.

## Delta Updates

Sparkle supports binary delta updates — users download only what changed.

### Automatic Generation

`generate_appcast` automatically creates and signs delta updates.

### Manual Generation

```sh
# Create delta
BinaryDelta create --version=$FORMAT_VERSION path/to/old/MyApp.app path/to/new/MyApp.app patch.delta

# Verify
BinaryDelta apply path/to/old/MyApp.app path/to/patched/MyApp.app patch.delta
```

### Delta Format Versions

| Format | Min Sparkle | Key Changes |
|--------|-------------|-------------|
| 4 | 2.7 | Preserves bundle creation date, efficient hash verification |
| 3 | 2.1 | Custom container format, lzma compression, file rename tracking |
| 2 | 1.10 | Improved hash function |
| 1 | 1.5 | Initial format (libxar, bzip2, bsdiff) |

### Appcast Delta Item

```xml
<sparkle:deltas>
    <enclosure url="http://you.com/1.5-2.0.delta"
               sparkle:deltaFrom="1.5"
               length="1485"
               type="application/octet-stream"
               sparkle:edSignature="..." />
</sparkle:deltas>
```

### Tips for Smaller Deltas

- Don't make unnecessary file modifications
- Store changing content in separate files from static content
- Avoid renaming files/folders (format 3 has heuristics for this)
- Delta updates don't support ACLs or extended attributes with code signing info
