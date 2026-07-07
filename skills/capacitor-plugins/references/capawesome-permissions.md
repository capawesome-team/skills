# Permissions

Check and request device permissions with a single unified API.

**Package:** `@capawesome/capacitor-permissions`
**Platforms:** Android, iOS, Web (partial)

## Installation

```bash
npm install @capawesome/capacitor-permissions
npx cap sync
```

## Configuration

The plugin itself declares **no permissions** and requires **no Info.plist keys**. Your app must declare exactly the permissions it actually uses. If a permission is not declared, `request(...)` rejects with a clear error message.

### Android — `AndroidManifest.xml`

Add the manifest entries for the permissions you use (before or after the `application` tag):

| `Permission`      | Manifest entries                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------- |
| `BLUETOOTH`       | `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`                                                                          |
| `CALENDAR`        | `READ_CALENDAR`, `WRITE_CALENDAR`                                                                              |
| `CAMERA`          | `CAMERA`                                                                                                       |
| `CONTACTS`        | `READ_CONTACTS`, `WRITE_CONTACTS`                                                                              |
| `LOCATION`        | `ACCESS_COARSE_LOCATION`, `ACCESS_FINE_LOCATION`                                                               |
| `LOCATION_ALWAYS` | `ACCESS_BACKGROUND_LOCATION` (plus the `LOCATION` entries)                                                     |
| `MICROPHONE`      | `RECORD_AUDIO`                                                                                                 |
| `MOTION`          | `ACTIVITY_RECOGNITION`                                                                                         |
| `NOTIFICATIONS`   | `POST_NOTIFICATIONS`                                                                                           |
| `PHOTOS`          | `READ_MEDIA_IMAGES`, `READ_MEDIA_VISUAL_USER_SELECTED`, `READ_EXTERNAL_STORAGE` (`android:maxSdkVersion="32"`) |
| `REMINDERS`       | Not available on Android.                                                                                      |

All entries use the `android.permission.` prefix, e.g. `<uses-permission android:name="android.permission.CAMERA" />`.

### iOS — `ios/App/App/Info.plist`

Add the usage description keys for the permissions you request:

| `Permission`      | Info.plist keys                                                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------- |
| `BLUETOOTH`       | `NSBluetoothAlwaysUsageDescription`                                                                 |
| `CALENDAR`        | `NSCalendarsFullAccessUsageDescription` (iOS 17+), `NSCalendarsUsageDescription` (iOS 16 and older) |
| `CAMERA`          | `NSCameraUsageDescription`                                                                          |
| `CONTACTS`        | `NSContactsUsageDescription`                                                                        |
| `LOCATION`        | `NSLocationWhenInUseUsageDescription`                                                               |
| `LOCATION_ALWAYS` | `NSLocationAlwaysAndWhenInUseUsageDescription`, `NSLocationWhenInUseUsageDescription`               |
| `MICROPHONE`      | `NSMicrophoneUsageDescription`                                                                      |
| `MOTION`          | `NSMotionUsageDescription`                                                                          |
| `NOTIFICATIONS`   | None                                                                                                |
| `PHOTOS`          | `NSPhotoLibraryUsageDescription`                                                                    |
| `REMINDERS`       | `NSRemindersFullAccessUsageDescription` (iOS 17+), `NSRemindersUsageDescription` (iOS 16 and older) |

Because the plugin references the system frameworks of all supported permissions, App Store Connect may ask for additional usage descriptions on upload even for permissions you never request.

## Usage

```typescript
import { Permission, Permissions } from '@capawesome/capacitor-permissions';

// check(...) never displays a permission prompt.
const { statuses } = await Permissions.check({
  permissions: [Permission.Camera, Permission.Microphone],
});
// statuses: { permission, state }[] in the same order as requested.

const result = await Permissions.request({
  permissions: [Permission.Camera, Permission.Microphone],
});
```

`Permission` values: `BLUETOOTH`, `CALENDAR`, `CAMERA`, `CONTACTS`, `LOCATION`, `LOCATION_ALWAYS`, `MICROPHONE`, `MOTION`, `NOTIFICATIONS`, `PHOTOS`, `REMINDERS`.
`PermissionState` values: `'granted' | 'denied' | 'prompt' | 'prompt-with-rationale' | 'limited' | 'unavailable'`.
`ErrorCode` values: `REQUEST_FAILED`, `USAGE_DESCRIPTION_MISSING` (iOS only).

## Notes

- **API-level gates (Android)**: `BLUETOOTH` covers `BLUETOOTH_SCAN`/`BLUETOOTH_CONNECT` only on Android 12+ (else always `granted`); `MOTION` covers `ACTIVITY_RECOGNITION` only on Android 10+ (else `granted`); `NOTIFICATIONS` covers `POST_NOTIFICATIONS` only on Android 13+ (else `granted`); `LOCATION_ALWAYS` covers `ACCESS_BACKGROUND_LOCATION` on Android 10+ and otherwise behaves like `LOCATION`.
- **Limited state**: partial photo-library access is reported as `limited` on Android 14+ and iOS. On iOS 17+ write-only calendar access is `limited`; on iOS 18+ limited contacts access is `limited`.
- **Web is partial**: only `NOTIFICATIONS` can be requested; `CAMERA`, `LOCATION`, and `MICROPHONE` support `check(...)` but `request(...)` just returns the current state (browsers prompt when the web API is first used). All other permissions are always `unavailable` on Web.
- **`REMINDERS`** is iOS-only (always `unavailable` on Android and Web); **`REMINDERS`, `CALENDAR`, `CONTACTS`, `BLUETOOTH`, `MOTION`, `LOCATION_ALWAYS`, `PHOTOS`** are always `unavailable` on Web.
- **Background location**: request `LOCATION_ALWAYS` only after `LOCATION` is granted. On Android 11+ the user is sent to system settings rather than shown a prompt; on iOS the upgrade prompt is shown only once.
- **Recovering from `denied`**: a `denied` permission cannot be re-requested in-app — guide the user to app settings via the Settings Launcher plugin.
- App Tracking Transparency is deliberately not included; use the App Tracking Transparency plugin instead.
