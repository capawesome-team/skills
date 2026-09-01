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

The `geofenceTransition` event is a **live feed**: it is only delivered while the app is running with a listener registered. Transitions detected while the app was in the background or terminated are **not** replayed — read them from the queue (see below), otherwise they are lost.

```typescript
import { Geofences, TransitionType } from '@capawesome-team/capacitor-geofences';

const addListener = async () => {
  await Geofences.addListener('geofenceTransition', (event) => {
    if (event.transition.type === TransitionType.Enter) {
      console.log(`Entered the geofence ${event.transition.geofenceId}.`);
    }
  });
};
```

### Read queued transitions

The queue is the durable record of every transition, including the ones detected while the app was terminated. It is **opt-in**: nothing is stored until `setConfig(...)` is called with `maxSize` or `url`. Drain it on app start and whenever the app becomes active again.

```typescript
import { Geofences } from '@capawesome-team/capacitor-geofences';

await Geofences.setConfig({ maxSize: 1000 });

const drainQueue = async () => {
  let hasMore = true;
  while (hasMore) {
    const result = await Geofences.getQueuedTransitions({ limit: 1000 });
    if (!result.transitions.length) {
      break;
    }
    await persist(result.transitions);
    await Geofences.deleteQueuedTransitions({
      upToId: result.transitions[result.transitions.length - 1].id,
    });
    hasMore = result.hasMore;
  }
};

const inspectQueue = async () => {
  const { pendingCount, droppedCount, lastUploadedAt } = await Geofences.getQueueStatus();
  await Geofences.clearQueue();
  return { pendingCount, droppedCount, lastUploadedAt };
};
```

### Retrieve and remove geofences

```typescript
import { Geofences } from '@capawesome-team/capacitor-geofences';

const { geofences } = await Geofences.getGeofences();
await Geofences.removeGeofences({ ids: ['1b8935d6-27b4-4a5c-9f0f-4a5c9f0f1b89'] });
await Geofences.removeAllGeofences();
```

### Upload transitions to a server

The configuration is persisted natively, so it only needs to be set once (e.g. after sign-in). Transitions are then uploaded even while the app is in the background or terminated. `setConfig(...)` **replaces** the whole configuration, so pass every property that should be kept.

```typescript
import { Geofences } from '@capawesome-team/capacitor-geofences';

const configureUpload = async () => {
  await Geofences.addListener('uploadFailed', (event) => {
    console.error('Upload failed: ', event.statusCode, event.message);
  });
  await Geofences.setConfig({
    url: 'https://api.example.com/transitions',
    headers: { Authorization: 'Bearer eyJhbGciOi...' },
    extras: { userId: 'abc' },
  });
};

const changeSingleOption = async () => {
  const config = await Geofences.getConfig();
  await Geofences.setConfig({ ...config, maxSize: 5000 });
};

const triggerUpload = async () => {
  await Geofences.triggerUpload();
};

const disableUpload = async () => {
  await Geofences.resetConfig();
};
```

## Notes

- All methods are Android and iOS only. On the web they reject with an unimplemented error.
- Geofencing requires the **Always** authorization on iOS and `ACCESS_BACKGROUND_LOCATION` on Android. With only the foreground permission, `addGeofences(...)` rejects with `PERMISSION_DENIED`.
- Limits: Android allows 100 geofences per app, iOS 20 regions. Exceeding them rejects `addGeofences(...)` with `GEOFENCE_LIMIT_EXCEEDED`.
- `ErrorCode` values: `ADD_FAILED`, `GEOFENCE_LIMIT_EXCEEDED`, `PERMISSION_DENIED`, `REMOVE_FAILED`.
- Radius: at least 200 meters recommended on iOS (Apple), at least 100 meters on Android. iOS clamps the radius to `maximumRegionMonitoringDistance`.
- `Geofence` options: `id` (generated UUID if omitted), `latitude`, `longitude`, `radius`, `notifyOnEnter` (default `true`), `notifyOnExit` (default `true`), `notification`, plus Android-only `androidNotifyOnDwell` (default `false`), `androidLoiteringDelay`, and `androidExpirationDuration`.
- `TransitionType` values: `ENTER`, `EXIT`, `DWELL`. Dwell is Android only; `androidNotifyOnDwell` and `androidLoiteringDelay` are ignored on iOS.
- On Android an enter transition fires immediately if the device is already inside a newly added geofence; iOS only reports a transition once the boundary is crossed.
- `GeofenceTransitionEvent` nests everything under a single `transition` property with `geofenceId`, `type`, `timestamp`, `latitude`, `longitude`. On iOS `latitude` and `longitude` are always `null`. A `QueuedTransition` carries the same properties plus a numeric `id`.
- **Terminated-app delivery: the `geofenceTransition` event is a live feed and replays nothing.** An app that only registers a listener silently receives nothing after a cold start. Enable the queue via `setConfig(...)` and drain it with `getQueuedTransitions(...)` + `deleteQueuedTransitions(...)`, or the transitions are lost. A geofence `notification` is displayed natively regardless of app state.
- Queue: opt-in (transitions are stored as soon as `maxSize` or `url` is set), persisted across restarts and reboots, holds `maxSize` (default `1000`) transitions with the oldest dropped first. `getQueuedTransitions(...)` returns them oldest first (`limit` default `100`, `afterId` for keyset pagination), `deleteQueuedTransitions({ upToId })` deletes up to and including that id, `clearQueue()` deletes all of them. `resetConfig()` stops storing and uploading but keeps the already queued transitions.
- `setConfig(...)` **replaces** the whole configuration — every omitted property falls back to its default, so `setConfig({ maxSize: 5000 })` also disables the upload. Read the stored configuration with `getConfig()` and spread it to change a single property.
- The upload drains the same queue that `getQueuedTransitions(...)` reads. Use either the upload or an own drain loop, not both.
- HTTP upload: transitions are `POST`ed as `{ "transitions": [...], "extras": {...} }` with `Content-Type: application/json; charset=utf-8`. Each entry carries `id` (queue identifier, deduplication key) and `geofenceId` (geofence identifier). Delivery is at-least-once — deduplicate on the server by `id` together with a device or user identifier from `extras`.
- Upload response handling: the response body is always ignored. `2xx` acknowledges and deletes; `401`, `408`, `429`, `5xx`, network errors, and timeouts retry with exponential backoff and emit `uploadFailed`; **any other status code drops the transitions permanently** and emits `uploadFailed` (also counted in `droppedCount`). `401` is retried so an expired token can be replaced via `setConfig(...)`. Request timeout is 30 seconds.
- Transitions that were buffered by an earlier version are **not** migrated and can no longer be read.
- On Android, geofences are automatically re-registered after a device reboot or an app update. On iOS the monitored regions are persisted by the operating system.
- Events: `geofenceTransition`, `uploadFailed`. Use `removeAllListeners()` to detach all of them.
