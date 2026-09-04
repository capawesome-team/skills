# Google Play Services

Check whether Google Play Services is available on the device, find out why it is not, and prompt the user to install, enable or update it.

**Package:** `@capawesome/capacitor-google-play-services`
**Platforms:** Android
**Documentation:** https://capawesome.io/docs/sdks/capacitor/google-play-services/

## Installation

```bash
npm install @capawesome/capacitor-google-play-services
npx cap sync
```

### Android

#### Variables

Defined in your app's `variables.gradle`:

- `$androidPlayServicesBaseVersion` version of `com.google.android.gms:play-services-base` (default: `18.10.1`)

## Usage

### Check the availability

```typescript
import { GooglePlayServices } from '@capawesome/capacitor-google-play-services';

const { available } = await GooglePlayServices.isAvailable();
```

### Detect devices without Google Play Services

```typescript
import { GooglePlayServices, Status } from '@capawesome/capacitor-google-play-services';

const { status } = await GooglePlayServices.getStatus();
if (status === Status.ServiceMissing) {
  // Device without Google Play Services (e.g. Huawei with HMS, Amazon Fire, AOSP builds)
} else if (status !== Status.Success) {
  // Installed but disabled, outdated, updating or invalid
}
```

### Make Google Play Services available

```typescript
import { ErrorCode, GooglePlayServices } from '@capawesome/capacitor-google-play-services';

try {
  await GooglePlayServices.makeAvailable();
} catch (error) {
  if (error.code === ErrorCode.Canceled) {
    console.log('The user dismissed the dialog.');
  }
}
```

### Get the version

```typescript
import { GooglePlayServices } from '@capawesome/capacitor-google-play-services';

const { version } = await GooglePlayServices.getVersion();
```

## Notes

- Android-only. All methods reject with the `UNIMPLEMENTED` error code on iOS and Web, so guard calls with `Capacitor.getPlatform() === 'android'` in cross-platform apps.
- `Status` values: `SUCCESS`, `SERVICE_MISSING`, `SERVICE_DISABLED`, `SERVICE_VERSION_UPDATE_REQUIRED`, `SERVICE_UPDATING`, `SERVICE_INVALID`. `isAvailable()` resolves to `true` only for `SUCCESS`.
- `getVersion()` resolves to `0` if Google Play Services is not installed.
- `makeAvailable()` shows Google's system dialog and resolves once Google Play Services is usable. It cannot install Google Play Services on devices without the Google Play Store (e.g. Huawei with HMS), so check `getStatus()` first and only call it for `SERVICE_DISABLED`, `SERVICE_VERSION_UPDATE_REQUIRED` or `SERVICE_UPDATING`. Dismissing the dialog rejects with the `CANCELED` error code; unresolvable cases reject with Google's error message.
- The availability check runs in the bundled `play-services-base` client library, so it does not crash on devices without Google Play Services.
- The `$androidPlayServicesBaseVersion` variable is shared with the App Update plugin, so one override applies to both.
- `ErrorCode` values: `CANCELED`.
