# PixLive

Build augmented reality experiences with the PixLive SDK by Vidinoti: synchronize content from PixLive Maker, display the AR camera view and react to recognized contexts.

**Package:** `@capawesome/capacitor-pixlive`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/pixlive/

## Installation

```bash
npm install @capawesome/capacitor-pixlive
npx cap sync
```

**Requires third-party setup:** the PixLive SDK is proprietary software by [Vidinoti](https://www.vidinoti.com/) and is **not bundled with the plugin**. A PixLive Maker account and license key are required, and the native SDK binaries must be added to the app project manually (see below).

## Configuration

### Android

#### SDK Setup

Copy the `vdarsdk-release.aar` file into `android/app/libs/`.

#### Permissions

No configuration is required. The plugin declares `CAMERA`, `ACCESS_FINE_LOCATION`, `POST_NOTIFICATIONS`, `BLUETOOTH`, `BLUETOOTH_CONNECT` and `BLUETOOTH_SCAN` itself.

### iOS

Only **CocoaPods** is supported as the iOS dependency manager. Swift Package Manager is not supported.

#### SDK Setup

Copy `VDARSDK.xcframework` into `ios/App/Frameworks/`.

### Plugin Configuration

In `capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  plugins: {
    Pixlive: {
      licenseKey: 'YOUR_LICENSE_KEY', // PixLive Maker license key
      apiUrl: 'https://ar.vidinoti.com/api/api.php', // optional, PixLive Maker API endpoint
      sdkUrl: 'https://sdk.vidinoti.com', // optional, PixLive SDK resource server
    },
  },
};
```

## Usage

### Initialize and request permissions

`initialize()` must be called before any other method. It initializes the SDK with the license key from the Capacitor configuration:

```typescript
import { Pixlive } from '@capawesome/capacitor-pixlive';

await Pixlive.initialize();

const status = await Pixlive.checkPermissions();
if (status.camera !== 'granted') {
  await Pixlive.requestPermissions({ permissions: ['camera'] });
}
```

### Synchronize content from PixLive Maker

```typescript
import { Pixlive } from '@capawesome/capacitor-pixlive';

await Pixlive.addListener('syncProgress', (event) => {
  console.log('Progress:', event.progress); // 0.0 - 1.0
});
await Pixlive.addListener('requireSync', (event) => {
  console.log('Sync required for tags:', event.tags);
});

await Pixlive.synchronize({ tags: [['my-tag']] });

const { contexts } = await Pixlive.getContexts();
```

### Display the AR view

```typescript
import { Pixlive } from '@capawesome/capacitor-pixlive';

await Pixlive.createARView({ x: 0, y: 0, width: 300, height: 400 });
await Pixlive.resizeARView({ x: 0, y: 0, width: 300, height: 600 });
await Pixlive.setARViewTouchEnabled({ enabled: true });
await Pixlive.setARViewTouchHole({ top: 0, bottom: 100, left: 0, right: 300 });
await Pixlive.destroyARView();
```

### React to recognized contexts and codes

```typescript
import { Pixlive } from '@capawesome/capacitor-pixlive';

await Pixlive.addListener('enterContext', (event) => {
  console.log('Entered context:', event.contextId);
});
await Pixlive.addListener('exitContext', (event) => {
  console.log('Exited context:', event.contextId);
});
await Pixlive.addListener('codeRecognize', (event) => {
  console.log('Scanned code:', event.code, event.type);
});
await Pixlive.addListener('eventFromContent', (event) => {
  console.log('Custom event:', event.name, event.params);
});
```

## Notes

- `initialize()` must be called first. Without a valid `licenseKey` in the Capacitor configuration, the SDK cannot be initialized.
- There is no web implementation — all methods rely on the native PixLive SDK and are only available on Android and iOS.
- Permission types: `camera`, `location`, `notifications`, plus `bluetooth` (iOS) or `bluetoothConnect` and `bluetoothScan` (Android). Add the matching iOS usage descriptions to `Info.plist` for the permissions the app requests.
- The AR view is a native view positioned by screen coordinates — keep it in sync with the layout via `resizeARView(...)`, and use `setARViewTouchHole(...)` to let touches pass through to the web view in a region.
- Contexts can be filtered with `enableContextsWithTags(...)` and triggered programmatically with `activateContext({ contextId })` / `stopContext()`.
- Location-based features: `getNearbyGPSPoints(...)`, `getGPSPointsInBoundingBox(...)`, `getNearbyBeacons()`, `startNearbyGPSDetection()` / `stopNearbyGPSDetection()` and `startGPSNotifications()` / `stopGPSNotifications()`.
- Other events: `presentAnnotations` and `hideAnnotations` for AR annotation visibility.
- `setInterfaceLanguage({ language })` changes the SDK interface language; `setNotificationsSupport({ enabled })` toggles local notification support.
- This is an unofficial plugin for the PixLive SDK by Vidinoti.
