# Capacitor Platforms

Capacitor supports multiple target platforms. Each platform has its own native project and build tooling.

## Android

- **Native project**: `android/` directory, opened with Android Studio.
- **Language**: Kotlin (default for Capacitor 7+) or Java.
- **Build system**: Gradle.
- **WebView**: Android System WebView (Chromium-based), updated via Google Play.
- **Minimum SDK**: Varies by Capacitor version (API 23 for Cap 7, API 24 for Cap 8).

Add the Android platform:

```bash
npx cap add android
```

Build and run:

```bash
npx cap run android
```

Or open in Android Studio:

```bash
npx cap open android
```

## iOS

- **Native project**: `ios/` directory, opened with Xcode.
- **Language**: Swift.
- **Build system**: Xcode Build System.
- **Dependency manager**: Swift Package Manager (default for Capacitor 8+) or CocoaPods.
- **WebView**: WKWebView.
- **Minimum deployment target**: Varies by Capacitor version (iOS 14 for Cap 7, iOS 16 for Cap 8).

Add the iOS platform:

```bash
npx cap add ios
```

Build and run:

```bash
npx cap run ios
```

Or open in Xcode:

```bash
npx cap open ios
```

## Progressive Web App (PWA)

Capacitor apps can run as PWAs in the browser without any native wrapper. The web app is served directly and uses standard Web APIs.

- No native project directory is created.
- Plugins with web implementations work automatically. Plugins without web implementations throw "not implemented" errors at runtime.
- Use `@capacitor/pwa-elements` for UI fallbacks (e.g., camera, toast) on the web:

```bash
npm install @capacitor/pwa-elements
```

Then register the elements in the app's entry file:

```typescript
import { defineCustomElements } from '@capacitor/pwa-elements/loader';
defineCustomElements(window);
```

To build for PWA, configure a service worker and web manifest using the framework's PWA tooling (e.g., `@angular/pwa`, `vite-plugin-pwa`, `next-pwa`).

## Electron (Desktop)

Capacitor apps can target desktop platforms (macOS, Windows, Linux) via the `@capacitor-community/electron` package.

- **Requirement**: Capacitor 5.4.0 or higher.
- **Native project**: `electron/` directory.
- Web plugins work automatically. Native plugins from iOS/Android are **not** available — use Electron-specific plugin implementations instead.

Add the Electron platform:

```bash
npm install @capacitor-community/electron
npx cap add @capacitor-community/electron
npx cap open @capacitor-community/electron
```

Electron support is community-maintained and may not cover all Capacitor plugin APIs. Evaluate plugin availability before committing to a desktop target.

## Platform Detection at Runtime

Use `Capacitor.getPlatform()` to detect the current platform at runtime:

```typescript
import { Capacitor } from '@capacitor/core';

const platform = Capacitor.getPlatform();
// Returns 'android', 'ios', or 'web'

if (Capacitor.isNativePlatform()) {
  // Running on Android or iOS
}
```

Use platform detection to conditionally execute platform-specific code or to provide fallback behavior on the web.
