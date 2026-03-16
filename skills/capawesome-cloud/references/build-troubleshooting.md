# Troubleshooting

Common native build errors and their solutions.

## Error: `invalid source release: 21`

**Cause:** Wrong Java version for the build.

**Fix:** Set `JAVA_VERSION` environment variable to a supported version (e.g., `17` or `21`). For Capacitor 7, Java 21 is the default. See `build-stacks.md` for available versions.

## Error: `JavaScript heap out of memory`

**Cause:** Node.js runs out of memory during the web build step. Default heap limit is ~2 GB.

**Fix (Option 1):** Add `NODE_OPTIONS` as an environment variable with value `--max-old-space-size=4096` (4 GB). Try `8192` (8 GB) if still insufficient.

**Fix (Option 2):** Set `NODE_OPTIONS` in the build script in `package.json`:

```json
"capawesome:build": "NODE_OPTIONS='--max-old-space-size=4096' npm run build"
```

## Build fails with authentication error

**Cause:** CLI not authenticated or token expired.

**Fix:** Re-authenticate:

```bash
npx @capawesome/cli login
```

For CI/CD, ensure `CAPAWESOME_CLOUD_TOKEN` is set and valid:

```bash
npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
```

## Build fails with missing signing certificate

**Cause:** No certificate uploaded or wrong certificate name specified.

**Fix:** List existing certificates and verify:

```bash
npx @capawesome/cli apps:certificates:list --app-id <APP_ID> --json
```

Upload a certificate if missing. See `certificates-android.md` or `certificates-ios.md`.

## Build fails with expired provisioning profile

**Cause:** iOS provisioning profile has expired (typically after one year).

**Fix:** Regenerate the provisioning profile in the [Apple Developer Portal](https://developer.apple.com/account), download, and re-upload to Capawesome Cloud. See `certificates-ios.md`.

## iOS build fails with code signing error

**Cause:** Certificate and provisioning profile mismatch, or wrong certificate type.

**Fix:**
- Ensure the certificate type matches the build type (`development` cert for `development`/`simulator` builds, `production` cert for `ad-hoc`/`app-store`/`enterprise` builds).
- Ensure the provisioning profile matches the app's bundle ID.
- For multi-target apps, ensure all targets have matching provisioning profiles or set `IOS_PROVISIONING_PROFILE_MAP`.

## Web build step fails

**Cause:** Missing or incorrect build script.

**Fix:** Ensure `package.json` has either a `capawesome:build` or `build` script. If using `capawesome.config.json`, verify `webBuildCommand` is correct. See `build-configuration.md`.

## Android build fails with Gradle error

**Cause:** Java version incompatibility with the Android Gradle Plugin.

**Fix:** Check the Gradle Plugin version in `android/build.gradle` and set the correct `JAVA_VERSION` environment variable. See `build-guides.md` (Override Java Version section).
