# System WebView

Detect an outdated Android System WebView and guide users to update it.

**Package:** `@capawesome/capacitor-system-webview`
**Platforms:** Android

## Installation

```bash
npm install @capawesome/capacitor-system-webview
npx cap sync
```

### Android

This plugin will use the following project variables (defined in your app's `variables.gradle` file):

- `$androidxWebkitVersion` version of `androidx.webkit:webkit` (default: `1.14.0`)

## Usage

### Prompt for an update when required

```typescript
import { Dialog } from '@capacitor/dialog';
import { SystemWebView } from '@capawesome/capacitor-system-webview';

const promptForUpdateIfRequired = async () => {
  const { required } = await SystemWebView.isUpdateRequired({
    minMajorVersion: 105, // Minimum Chromium major version your app's bundle needs.
  });
  if (!required) {
    return;
  }
  const { value } = await Dialog.confirm({
    title: 'Update required',
    message: 'Your Android System WebView is outdated. Please update it.',
    okButtonTitle: 'Update',
  });
  if (value) {
    await SystemWebView.openAppStore();
  }
};
```

### Read provider info

```typescript
import { SystemWebView } from '@capawesome/capacitor-system-webview';

const { packageName, versionName, majorVersion } = await SystemWebView.getInfo();
// e.g. "com.google.android.webview", "126.0.6478.122", 126
```

## Notes

- **Android-only.** On iOS and Web all methods reject as unimplemented. iOS has no implementation by design: `WKWebView` is part of the OS and updates with iOS itself, so there is nothing to detect or fix.
- On Android, the app runs inside the System WebView, which updates independently of the OS. Devices can run an ancient WebView when auto-updates are disabled, the device is offline, or a custom ROM is used — causing missing modern web features and hard-to-diagnose breakage.
- The plugin never hardcodes an "outdated" threshold. Your app declares the minimum Chromium major version it needs via `isUpdateRequired({ minMajorVersion })`. Rough baselines: `97+` Array `findLast`, `105+` CSS `:has()` + container queries, `112+` native CSS nesting, `114+` `text-wrap: balance`.
- `getInfo()` reports the **active** provider. `majorVersion` is the integer before the first dot of `versionName`; it is `null` for OEM WebView forks whose `versionName` is not a Chromium-style version.
- `openAppStore()` opens the Play Store listing of the **active** WebView provider (which may be a non-Google provider such as an OEM or Chrome-based one, depending on the device), not a fixed package.
- A WebView update only takes effect the next time the app process starts. The running app keeps using the old WebView until restarted.
- Error codes: `OPEN_FAILED` (Play Store could not be opened), `VERSION_UNPARSEABLE` (version name not Chromium-style), `WEB_VIEW_PACKAGE_UNAVAILABLE` (active WebView package could not be determined, e.g. on custom ROMs or when the System WebView is disabled).
