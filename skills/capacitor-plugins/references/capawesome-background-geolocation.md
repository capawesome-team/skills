# Background Geolocation

Capacitor plugin for reliable background geolocation tracking. Provides one-shot positions, background watch sessions with an Android foreground service, and native HTTP sync of positions to your own server.

**Package:** `@capawesome-team/capacitor-background-geolocation`
**Platforms:** Android, iOS
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-background-geolocation
npx cap sync
```

## Configuration

### Android

#### Permissions

The plugin already declares the location and foreground service permissions in its own manifest. To receive position updates while the app is in the **background**, add the following **before or after** the `application` tag in `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

Only declare it if the app really needs background tracking: Google Play requires a policy declaration and review for it.

#### Variables

Set `playServicesLocationVersion` in `android/variables.gradle` to change the version of `com.google.android.gms:play-services-location` (default: `21.4.0`).

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Info.plist

Add to `ios/App/App/Info.plist`:

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app needs access to your location to track your position.</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>The app needs access to your location to track your position, even while the app is in the background.</string>
<!-- Required only for position updates while the app is in the background -->
<key>UIBackgroundModes</key>
<array>
  <string>location</string>
</array>
<!-- Required only for requestTemporaryFullAccuracy(...): one entry per purpose key -->
<key>NSLocationTemporaryUsageDescriptionDictionary</key>
<dict>
  <key>navigation</key>
  <string>The app needs your precise location to provide turn-by-turn navigation.</string>
</dict>
```

## Usage

### Check and request permissions

```typescript
import { BackgroundGeolocation } from '@capawesome-team/capacitor-background-geolocation';

await BackgroundGeolocation.requestPermissions({ permissions: ['location', 'notifications'] });
// Request the background permission in a separate, second call.
await BackgroundGeolocation.requestPermissions({ permissions: ['backgroundLocation'] });
const status = await BackgroundGeolocation.checkPermissions();
```

### Get the current position

```typescript
import { Accuracy, BackgroundGeolocation } from '@capawesome-team/capacitor-background-geolocation';

const { position } = await BackgroundGeolocation.getCurrentPosition({ accuracy: Accuracy.High, timeout: 10000 });
```

### Watch the position

```typescript
import { Accuracy, ActivityType, BackgroundGeolocation } from '@capawesome-team/capacitor-background-geolocation';

await BackgroundGeolocation.addListener('positionChange', ({ position }) => console.log(position));
await BackgroundGeolocation.addListener('positionError', ({ code, message }) => console.error(code, message));
await BackgroundGeolocation.startWatching({
  accuracy: Accuracy.High,
  distanceFilter: 10,
  androidInterval: 5000,
  androidNotification: { title: 'Location Tracking', text: 'Your location is being tracked.' },
  iosActivityType: ActivityType.Fitness,
});

const { watching } = await BackgroundGeolocation.isWatching();
await BackgroundGeolocation.stopWatching();
```

### Sync positions to a server

```typescript
import { BackgroundGeolocation } from '@capawesome-team/capacitor-background-geolocation';

await BackgroundGeolocation.addListener('syncFailed', ({ statusCode, message }) => console.error(statusCode, message));
await BackgroundGeolocation.startWatching({
  androidNotification: { title: 'Location Tracking', text: 'Your location is being tracked.' },
  sync: {
    url: 'https://api.example.com/positions',
    batchSize: 100,
    headers: { Authorization: 'Bearer eyJhbGciOi...' },
    extras: { userId: 'abc' },
  },
});

const { pendingCount, droppedCount, lastSyncedAt } = await BackgroundGeolocation.getSyncStatus();
await BackgroundGeolocation.triggerSync();
await BackgroundGeolocation.clearSyncQueue();
```

## Notes

- The `backgroundLocation` permission must be requested in a **second, separate** call after `location` was granted. On Android 11+ the user is taken to the app location settings and must select `Allow all the time`; on iOS the system shows the upgrade prompt from `While Using the App` to `Always`.
- Use `openSettings()` when a permission was permanently denied, and `requestTemporaryFullAccuracy({ purposeKey: 'navigation' })` (iOS only) to upgrade reduced accuracy for the app session.
- `androidNotification` is **required** on Android because the watch session runs in a foreground service. Options: `title`, `text`, `channelName`, `color` (hex), `icon` (drawable name).
- Only one watch session can be active at a time. `startWatching()` rejects with `ALREADY_WATCHING` otherwise. Call `stopWatching()` before starting a session with different options.
- Events: `positionChange`, `positionError`, `syncFailed` (remove with `removeAllListeners()`). Error codes: `ALREADY_WATCHING`, `LOCATION_SERVICES_DISABLED`, `PERMISSION_DENIED`, `POSITION_UNAVAILABLE`, `TIMEOUT`.
- `startWatching()` tuning: `accuracy` (`Accuracy.Low` | `Balanced` | `High`), `distanceFilter` (meters), `androidInterval` (ms), `iosActivityType`, `iosPausesAutomatically`, `iosShowBackgroundIndicator`, `androidForceLocationManager` (platform location manager for devices without Google Play services).
- `Position` fields: `latitude`, `longitude`, `accuracy`, `altitude`, `altitudeAccuracy`, `bearing`, `speed`, `timestamp`, `simulated` (mock provider flag, always `null` on iOS).
- HTTP sync `POST`s `{ positions: [...], extras: {...} }` where each position carries an extra `id`. Delivery is at-least-once, so deduplicate by `id` per device on the server. The `INTERNET` permission it needs is already declared by the Capacitor app template.
- Sync responses: `2xx` acknowledges and deletes the batch; `408`, `429`, `5xx`, network errors and timeouts retry with exponential backoff (5s up to 10min, 30s request timeout); **any other status code drops the batch permanently**. Answer with a retryable code (e.g. `503`) when temporarily unavailable.
- Sync options: `url`, `batchSize` (default `100`, set to `1` for instant upload), `flushInterval` (default `60000`), `maxQueueSize` (default `10000`, oldest dropped first), `maxAge`, `headers`, `extras`. The queue is a local SQLite database that survives app restarts and is only uploaded while a watch session with a `sync` configuration is active.
- Tracking stops when the user force-quits the app on both platforms. Use the Geofences plugin if the app must be relaunched after termination.
