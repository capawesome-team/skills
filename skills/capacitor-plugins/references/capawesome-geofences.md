# Geofences

Capacitor plugin for monitoring OS-managed geofences (region monitoring). Detects enter, exit, and dwell transitions even while the app is in the background or terminated.

**Package:** `@capawesome-team/capacitor-geofences`
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
npm install @capawesome-team/capacitor-geofences
npx cap sync
```

## Configuration

### Android

#### Permissions

The plugin already declares `ACCESS_FINE_LOCATION`, `POST_NOTIFICATIONS`, and `RECEIVE_BOOT_COMPLETED`. The background location permission is **not** declared by the plugin (Google Play policy) and must be added to `android/app/src/main/AndroidManifest.xml` before or after the `application` tag:

```xml
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

#### Variables

If needed, define these variables in `android/variables.gradle` to change the default dependency versions:

- `androidxWorkVersion` — version of `androidx.work:work-runtime` (default: `2.11.2`)
- `playServicesLocationVersion` — version of `com.google.android.gms:play-services-location` (default: `21.4.0`)

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Privacy Descriptions

Add to `ios/App/App/Info.plist`:

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app needs access to your location to monitor geofences.</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>The app needs access to your location to monitor geofences while it is in the background.</string>
```

If these keys are missing, `addGeofences(...)` and `requestPermissions(...)` reject with an error.

## Usage

### Check and request permissions

The `backgroundLocation` permission must be requested in a **second**, separate call after `location` has been granted:

```typescript
import { Geofences } from '@capawesome-team/capacitor-geofences';

const checkPermissions = async () => {
  return Geofences.checkPermissions();
};

const requestPermissions = async () => {
  let status = await Geofences.requestPermissions({ permissions: ['location'] });
  if (status.location === 'granted') {
    status = await Geofences.requestPermissions({ permissions: ['backgroundLocation'] });
  }
  await Geofences.requestPermissions({ permissions: ['notifications'] });
  return status;
};

const openSettings = async () => {
  await Geofences.openSettings();
};
```

### Add geofences

```typescript
import { Geofences } from '@capawesome-team/capacitor-geofences';

const addGeofences = async () => {
  const { ids } = await Geofences.addGeofences({
    geofences: [
      {
        latitude: 37.33182,
        longitude: -122.03118,
        radius: 200,
        notification: {
          title: 'Welcome',
          text: 'You have entered the area.',
        },
      },
    ],
  });
  return ids;
};
```

### Listen for geofence transitions

Register the listener as early as possible after app start so that transitions queued while the app was terminated are replayed.

```typescript
import { Geofences, TransitionType } from '@capawesome-team/capacitor-geofences';

const addListener = async () => {
  await Geofences.addListener('geofenceTransition', (event) => {
    if (event.transitionType === TransitionType.Enter) {
      console.log(`Entered the geofence ${event.id}.`);
    }
  });
};
```

### Retrieve and remove geofences

```typescript
import { Geofences } from '@capawesome-team/capacitor-geofences';

const { geofences } = await Geofences.getGeofences();
await Geofences.removeGeofences({ ids: ['1b8935d6-27b4-4a5c-9f0f-4a5c9f0f1b89'] });
await Geofences.removeAllGeofences();
```

### Sync transitions to a server

The configuration is persisted natively, so it only needs to be set once (e.g. after sign-in). Transitions are then uploaded even while the app is in the background or terminated.

```typescript
import { Geofences } from '@capawesome-team/capacitor-geofences';

const configureSync = async () => {
  await Geofences.addListener('syncFailed', (event) => {
    console.error('Upload failed: ', event.statusCode, event.message);
  });
  await Geofences.configureSync({
    url: 'https://api.example.com/transitions',
    headers: { Authorization: 'Bearer eyJhbGciOi...' },
    extras: { userId: 'abc' },
  });
};

const inspectQueue = async () => {
  const { pendingCount, droppedCount, lastSyncedAt } = await Geofences.getSyncStatus();
  await Geofences.triggerSync();
  await Geofences.clearSyncQueue();
  return { pendingCount, droppedCount, lastSyncedAt };
};
```

## Notes

- All methods are Android and iOS only. On the web they reject with an unimplemented error.
- Geofencing requires the **Always** authorization on iOS and `ACCESS_BACKGROUND_LOCATION` on Android. With only the foreground permission, `addGeofences(...)` rejects with `PERMISSION_DENIED`.
- Limits: Android allows 100 geofences per app, iOS 20 regions. Exceeding them rejects `addGeofences(...)` with `GEOFENCE_LIMIT_EXCEEDED`.
- Radius: at least 200 meters recommended on iOS (Apple), at least 100 meters on Android. iOS clamps the radius to `maximumRegionMonitoringDistance`.
- `Geofence` options: `id` (generated UUID if omitted), `latitude`, `longitude`, `radius`, `notifyOnEnter` (default `true`), `notifyOnExit` (default `true`), `notification`, plus Android-only `androidNotifyOnDwell` (default `false`), `androidLoiteringDelay`, and `androidExpirationDuration`.
- `TransitionType` values: `ENTER`, `EXIT`, `DWELL`. Dwell is Android only; `androidNotifyOnDwell` and `androidLoiteringDelay` are ignored on iOS.
- On Android an enter transition fires immediately if the device is already inside a newly added geofence; iOS only reports a transition once the boundary is crossed.
- `GeofenceTransitionEvent` provides `id`, `transitionType`, `timestamp`, `latitude`, `longitude`. On iOS `latitude` and `longitude` are always `null`.
- Terminated-app delivery: transitions are queued and replayed in order once the first `geofenceTransition` listener is registered. The replay buffer holds at most `100` transitions; the oldest are dropped when it overflows. A geofence `notification` is displayed natively regardless of app state.
- On Android, geofences are automatically re-registered after a device reboot or an app update. On iOS the monitored regions are persisted by the operating system.
- HTTP sync: transitions are `POST`ed as `{ "transitions": [...], "extras": {...} }` with `Content-Type: application/json; charset=utf-8`. Delivery is at-least-once — deduplicate by the transition `id` on the server.
- Sync response handling: `2xx` acknowledges and deletes; `408`, `429`, `5xx`, network errors, and timeouts retry with exponential backoff and emit `syncFailed`; **any other status code drops the transitions permanently** and emits `syncFailed`. Return a retryable status (e.g. `503`) if the server is temporarily unavailable.
- Sync queue: persisted across restarts and reboots, holds at most `1000` transitions, oldest dropped first. Request timeout is 30 seconds. `disableSync()` stops buffering but keeps already-queued transitions until `clearSyncQueue()` is called.
- Events: `geofenceTransition`, `syncFailed`. Use `removeAllListeners()` to detach all of them.
