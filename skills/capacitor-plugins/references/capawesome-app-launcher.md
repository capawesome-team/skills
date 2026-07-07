# App Launcher

Check whether an app can be opened by URL scheme or package name, and open it.

**Package:** `@capawesome/capacitor-app-launcher`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-app-launcher
npx cap sync
```

## Configuration

### Android

Starting with Android 11 (API level 30), apps must declare the packages and URL schemes they want to check or open in the [`<queries>`](https://developer.android.com/training/package-visibility) element. Otherwise `canOpenUrl(...)` always resolves with `value: false` and `openUrl(...)` may resolve with `completed: false`.

Add an entry for **every** package name and URL scheme you pass to the plugin, in `android/app/src/main/AndroidManifest.xml` (before or after the `<application>` tag):

```xml
<queries>
    <!-- Required to check and open a specific package name. -->
    <package android:name="com.google.android.gm" />
    <!-- Required to check and open a specific URL scheme. -->
    <intent>
        <action android:name="android.intent.action.VIEW" />
        <data android:scheme="mailto" />
    </intent>
</queries>
```

### iOS

Every URL scheme you check with `canOpenUrl(...)` must be declared in the [`LSApplicationQueriesSchemes`](https://developer.apple.com/documentation/bundleresources/information_property_list/lsapplicationqueriesschemes) key. Otherwise `canOpenUrl(...)` always resolves with `value: false`.

Add an entry for **every** URL scheme you pass to the plugin, in `ios/App/App/Info.plist`:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>mailto</string>
</array>
```

## Usage

```typescript
import { AppLauncher } from '@capawesome/capacitor-app-launcher';

const { value } = await AppLauncher.canOpenUrl({ url: 'mailto:' });

const { completed } = await AppLauncher.openUrl({ url: 'mailto:' });
```

## Notes

- On iOS, `url` must be a URL scheme (e.g. `mailto:`). On Android, `url` may be a URL scheme or a package name (e.g. `com.google.android.gm`) for both methods.
- On Web, `canOpenUrl(...)` always resolves with `value: true` because the browser cannot determine whether a URL can be opened.
- Drop-in replacement for `@capacitor/app-launcher` (identical API); only the import and installation change.
