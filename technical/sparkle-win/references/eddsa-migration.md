# Migrating from DSA to EdDSA Signatures

WinSparkle deprecated DSA signatures in favor of EdDSA (Ed25519). This guide covers migrating without disrupting existing users.

## Step-by-Step Migration

### 1. Update WinSparkle in your app

Update to WinSparkle 0.9.0 or newer.

### 2. Replace DSA key with EdDSA key

Generate a new EdDSA keypair:

```bash
winsparkle-tool generate-key --file private.key
```

Replace `win_sparkle_set_dsa_pub_pem()` or `DSAPub` resource with EdDSA equivalent:

**API approach:**
```c
// Before (DSA - remove this)
win_sparkle_set_dsa_pub_pem("-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----");

// After (EdDSA)
win_sparkle_set_eddsa_public_key("pXAx0wfi8kGbeQln11+V4R3tCepSuLXeo7LkOeudc/U=");
```

**Resource approach:**
```rc
// Before (DSA - remove this)
DSAPub PEM "dsa_pub.pem"

// After (EdDSA)
EdDSAPub EDDSA {"pXAx0wfi8kGbeQln11+V4R3tCepSuLXeo7LkOeudc/U="}
```

### 3. Release the updated app

Ship the version with the EdDSA public key.

### 4. Dual-sign appcast enclosures

Add `sparkle:edSignature` **in addition to** existing `sparkle:dsaSignature`:

```xml
<enclosure url="https://example.com/setup.exe"
           sparkle:edSignature="JhQ69mgRxjNxS35z..."
           sparkle:dsaSignature="MEQCICh10Sof..."
           length="1736832"
           type="application/octet-stream" />
```

This ensures:
- Older app versions (with DSA key) validate using `dsaSignature`
- Newer app versions (with EdDSA key) validate using `edSignature`

## Dropping DSA Completely

Three options:

### Option A: Wait for natural adoption

Wait until most/all users have updated to the EdDSA-enabled version, then stop including `dsaSignature` in new appcast entries.

### Option B: Two-step forced migration

1. Change the appcast feed URL in the new (EdDSA) version of your app
2. Stop publishing updates to the old URL
3. Users on old versions update to the last DSA-signed version (old feed), then to latest (new feed)

### Option C: Keep dual-signing indefinitely

Continue including both signatures. WinSparkle ignores DSA keys and signatures when an EdDSA public key is configured, so dual-signing does not reduce security.

## Security Note

DSA (specifically DSA with SHA-1) has known security weaknesses. EdDSA with Ed25519 provides stronger security guarantees. Migration is strongly recommended.
