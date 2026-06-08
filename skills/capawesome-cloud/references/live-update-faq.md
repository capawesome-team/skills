# Frequently Asked Questions

## Compliance

### Are Live Updates compliant with Apple App Store policies?

Yes. The Apple Developer Program License Agreement allows interpreted code to be downloaded as long as it does not change the primary purpose of the app or bypass OS security features. Live Updates only update the web layer.

### Are Live Updates compliant with Google Play policies?

Yes. Google Play's Device and Network Abuse policy restriction on self-updating does not apply to JavaScript running in a webview or browser. Live Updates only modify the web layer.

## Updates

### What are binary-compatible changes?

Changes that only affect web assets (HTML, CSS, JS, images) and do not require a native update. Native code changes (Java, Swift, CocoaPods, Gradle) are NOT binary-compatible and require an app store submission.

### What is the maximum bundle size?

1 GB on Capawesome Cloud. To work with larger bundles:
1. Reduce bundle size by removing source maps and optimizing assets.
2. Use `manifest` artifact type (1 GB limit applies per file, not the whole bundle).
3. Self-host the bundle.

### What happens when a bundle is deleted?

Devices that already downloaded the bundle continue using it. The previous bundle becomes the latest in the channel.

### How to automatically delete old bundles?

1. Set a bundle limit per channel (oldest auto-deleted when limit reached).
2. Set an expiration date per bundle with `--expires-in-days`.

## Channels

### What happens when a channel is deleted?

All bundles in the channel are deleted. Devices with existing bundles continue using them.

## Devices

### How are Monthly Active Users (MAU) counted?

A MAU is a unique device that has synced with Capawesome Cloud in the current month, identified by a plugin-generated unique ID.

### What happens at the MAU limit?

New devices stop receiving updates. Existing devices continue receiving updates. An email notification is sent.

### What happens at the storage limit?

New uploads are blocked. Delete old bundles or upgrade the plan.

## Privacy

### Is PII collected?

No. Only device-generated identifiers are sent to Capawesome Cloud for MAU counting.

## Development

### Why are local changes not visible after installing a live update?

Once a live update bundle is active, it takes precedence over the default bundle. Options:
1. Call `LiveUpdate.reset()` then reload.
2. Reinstall the app.
3. Increase the native version code (Capacitor auto-resets on native update).

### Do not use Live Reload during testing

The Live Reload development feature uses a dev server, not the local file system. Disable Live Reload when testing live updates:

```bash
# Capacitor
npx ionic cap run android --open
# Cordova
cordova run android
```

## Cordova

### Does the Cordova plugin require a specific WebView scheme?

Yes. `@capawesome/cordova-live-update` requires the **default custom WebView scheme** (`https://localhost` on Android, `app://localhost` on iOS). It does **not** work if `config.xml` sets `<preference name="Scheme" value="file" />` or `<preference name="AndroidInsecureFileModeEnabled" value="true" />`. It does not require `cordova-plugin-ionic-webview`.

### Why don't my Cordova preference changes take effect?

Cordova reads plugin `<preference>` values only when a platform is added. After changing any Live Update preference, run `cordova platform rm <platform>` followed by `cordova platform add <platform>`.

### Is the Cordova API the same as Capacitor?

Yes — full API parity. Access the plugin via `cordova.plugins.LiveUpdate` after `deviceready`. See `live-update-plugin-api.md`.
