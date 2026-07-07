# In-App Browser

Open URLs in the external browser, the system browser (Custom Tabs / `SFSafariViewController`), or an embedded web view.

**Package:** `@capawesome/capacitor-in-app-browser`
**Platforms:** Android, iOS, Web (Web supports `openInExternalBrowser` only)

## Installation

```bash
npm install @capawesome/capacitor-in-app-browser
npx cap sync
```

## Configuration

### Android

#### Variables

Optionally define in `android/variables.gradle`:

- `$androidxBrowserVersion` version of `androidx.browser:browser` (default: `1.9.0`)

#### Permissions

Only required if web pages opened in the embedded web view should access the camera or microphone. Add to `android/app/src/main/AndroidManifest.xml` before or after the `application` tag:

```xml
<!-- Required if web pages should be able to access the camera. -->
<uses-permission android:name="android.permission.CAMERA" />
<!-- Required if web pages should be able to access the microphone. -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

The permissions must also be granted before a web page requests access.

### iOS

#### Privacy Descriptions

Only required if web pages opened in the embedded web view should access the camera or microphone. Add to `ios/App/App/Info.plist`:

```xml
<key>NSCameraUsageDescription</key>
<string>The camera is used by websites opened in the in-app browser.</string>
<key>NSMicrophoneUsageDescription</key>
<string>The microphone is used by websites opened in the in-app browser.</string>
```

## Usage

### Open a URL in the three browser modes

```typescript
import { InAppBrowser } from '@capawesome/capacitor-in-app-browser';

// Default browser app of the device (available on Web too).
await InAppBrowser.openInExternalBrowser({ url: 'https://capawesome.io' });

// System browser (Custom Tabs on Android, SFSafariViewController on iOS).
await InAppBrowser.openInSystemBrowser({
  url: 'https://capawesome.io',
  android: { showTitle: true, toolbarColor: '#008080' },
  ios: { dismissButtonStyle: 'close', toolbarColor: '#008080' },
});

// Embedded web view with a native toolbar.
await InAppBrowser.openInWebView({
  url: 'https://capawesome.io',
  toolbar: { backgroundColor: '#008080', color: '#FFFFFF', showNavigationButtons: true },
});

await InAppBrowser.close();
```

### Execute a script and exchange messages (web view only)

```typescript
import { InAppBrowser } from '@capawesome/capacitor-in-app-browser';

const { result } = await InAppBrowser.executeScript({ script: 'document.title' });

// App -> web page (page listens for the `capacitorInAppBrowserMessage` window event).
await InAppBrowser.postMessage({ data: { name: 'Capawesome' } });

// Web page -> app via window.CapacitorInAppBrowser.postMessage(...).
await InAppBrowser.addListener('messageReceived', event => {
  console.log('Message received', event.data);
});
```

### Listen to navigation events and manage session data

```typescript
import { InAppBrowser } from '@capawesome/capacitor-in-app-browser';

await InAppBrowser.addListener('browserClosed', () => console.log('closed'));
await InAppBrowser.addListener('browserPageLoaded', () => console.log('loaded'));
await InAppBrowser.addListener('browserPageNavigationCompleted', event => {
  console.log('Navigated to', event.url);
});

await InAppBrowser.clearCache();
await InAppBrowser.clearSessionData();
const { cookies } = await InAppBrowser.getCookies({ url: 'https://capawesome.io' });
```

## Notes

- On Web, only `openInExternalBrowser(...)` is supported. All other methods are Android/iOS only.
- `executeScript(...)`, `postMessage(...)`, `browserPageNavigationCompleted`, and `messageReceived` are only available for browsers opened with `openInWebView(...)`.
- System browser: visited URLs cannot be tracked by design. On Android, `browserPageLoaded` is not emitted for the system browser, and `browserClosed` fires when the user returns to the app.
- Embedded web view: the `dataStore` option (`'isolated'` | `'shared'`) is iOS-only; on Android the web view always uses the app-global `shared` store. On iOS, hiding the toolbar removes the close button, so the browser can then only be closed via `close(...)`.
- External browser: no events are emitted and `close(...)` has no effect.
- `ErrorCode` values: `BROWSER_NOT_FOUND`, `NO_BROWSER_OPEN`.
