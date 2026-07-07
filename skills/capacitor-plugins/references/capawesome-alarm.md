# Alarm

Create system alarms and timers via the platform clock apps and Apple's AlarmKit.

**Package:** `@capawesome/capacitor-alarm`
**Platforms:** Android, iOS (iOS 26+)

## Installation

```bash
npm install @capawesome/capacitor-alarm
npx cap sync
```

## Configuration

### Android

The plugin already declares the `com.android.alarm.permission.SET_ALARM` permission in its manifest. This is a normal permission granted automatically, so no additional configuration is required.

### iOS

This plugin uses Apple's [AlarmKit](https://developer.apple.com/documentation/alarmkit) framework, which requires **iOS 26 or later**. On older versions, all methods except `isAvailable()` reject as unavailable.

Add the `NSAlarmKitUsageDescription` key to `ios/App/App/Info.plist`:

```xml
<key>NSAlarmKitUsageDescription</key>
<string>The app uses alarms to notify you at the times you choose.</string>
```

## Usage

```typescript
import { Alarm, Weekday } from '@capawesome/capacitor-alarm';

// Check availability (Android: a clock app is installed; iOS: device runs iOS 26+).
const { available } = await Alarm.isAvailable();

// Permissions (Android always returns 'granted').
const { alarms } = await Alarm.requestPermissions();

// Create an alarm.
const { id } = await Alarm.createAlarm({
  hour: 6,
  minute: 30,
  label: 'Wake up',
  days: [Weekday.Monday, Weekday.Friday], // Omit for a one-off alarm.
});
```

```typescript
// iOS only: list and cancel app-owned alarms.
const { alarms } = await Alarm.getAlarms();
await Alarm.cancelAlarm({ id });

// Android only: create a timer / open the clock app's alarm list.
await Alarm.createTimer({ duration: 300, label: 'Tea' });
await Alarm.openAlarms();
```

`Weekday` values: `MONDAY`, `TUESDAY`, `WEDNESDAY`, `THURSDAY`, `FRIDAY`, `SATURDAY`, `SUNDAY`.
`ErrorCode` values: `NO_CLOCK_APP`, `PERMISSION_DENIED`.

## Notes

- The two platforms use fundamentally different alarm models, exposed honestly rather than hidden behind a false abstraction.
- **Android**: alarms and timers are created in the **system clock app** via the `AlarmClock` intent API (fire-and-forget). `createAlarm(...)` returns `id: null`, and `getAlarms()` and `cancelAlarm(...)` reject as unimplemented. `createTimer(...)` and `openAlarms()` are Android-only.
- **iOS (26+)**: alarms are created with AlarmKit and are **owned by the app**, so they can be listed with `getAlarms()` and canceled with `cancelAlarm(...)`. `createTimer(...)` and `openAlarms()` reject on iOS.
- Android-specific options: `android.skipUi` (create without showing the clock app UI, default `false`) and `android.vibrate` for alarms; `android.skipUi` for timers.
- Alarms are not local notifications: they break through Silent Mode and Focus and play sound until dismissed, but cannot carry custom content or actions.
