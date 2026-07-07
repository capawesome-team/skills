# Settings Launcher

Open system settings screens, your app's settings, or your app's notification settings.

**Package:** `@capawesome/capacitor-settings-launcher`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-settings-launcher
npx cap sync
```

No additional permissions or configuration are required. On Web, all methods reject as unimplemented.

## Usage

### Open your app's settings or notification settings

```typescript
import { SettingsLauncher } from '@capawesome/capacitor-settings-launcher';

// Recommended way to guide users to grant a previously denied permission.
await SettingsLauncher.openAppSettings();

// Open the notification settings of your app (iOS requires iOS 16+).
await SettingsLauncher.openNotificationSettings();
```

### Open a specific Android system settings screen

```typescript
import { SettingsLauncher, AndroidSettingsPage } from '@capawesome/capacitor-settings-launcher';

await SettingsLauncher.openAndroidSettings({
  page: AndroidSettingsPage.Wifi,
});
```

## Notes

- `openAndroidSettings(...)` is Android-only. `openAppSettings()` and `openNotificationSettings()` are available on Android and iOS.
- On iOS there is no public API to deep-link into arbitrary system settings screens, so the plugin can only open your app's own settings (`openAppSettings`) and, on iOS 16+, its notification settings (`openNotificationSettings`). On older iOS versions `openNotificationSettings()` rejects as unavailable.
- Some `AndroidSettingsPage` screens do not exist on every device because manufacturers remove or replace them. In that case `openAndroidSettings(...)` rejects with the `PAGE_NOT_SUPPORTED` error code — always handle it gracefully rather than assuming a page is present.
- `AndroidSettingsPage` members (their string values are SCREAMING_SNAKE, e.g. `AndroidSettingsPage.Wifi` = `'WIFI'`) include `Accessibility`, `AirplaneMode`, `Apn`, `BatterySaver`, `Bluetooth`, `Captioning`, `CastSettings`, `DataRoaming`, `Date`, `Development`, `DeviceInfo`, `Display`, `Dream`, `Home`, `IgnoreBatteryOptimization`, `InputMethod`, `InternalStorage`, `Locale`, `Location`, `ManageAllApplications`, `ManageApplications`, `MemoryCard`, `Network`, `Nfc`, `NfcPayment`, `NotificationListener`, `Printing`, `Privacy`, `Search`, `Security`, `Sound`, `Sync`, `Usage`, `UserDictionary`, `VoiceInput`, `Vpn`, `Wifi`, `Wireless`.
- `ErrorCode` values: `OPEN_FAILED`, `PAGE_NOT_SUPPORTED`.
