# Haptics

Trigger haptic feedback: impacts, notifications, selection changes, custom patterns, and vibration.

**Package:** `@capawesome/capacitor-haptics`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-haptics
npx cap sync
```

## Configuration

### Android

The plugin already declares the `android.permission.VIBRATE` permission in its manifest, so no additional configuration is required.

## Usage

```typescript
import { Haptics, ImpactStyle, NotificationType } from '@capawesome/capacitor-haptics';

const { available } = await Haptics.isAvailable();

await Haptics.impact({ style: ImpactStyle.Medium }); // Default: Medium.
await Haptics.notification({ type: NotificationType.Success }); // Default: Success.
await Haptics.vibrate({ duration: 500 }); // Android/Web only; default 300ms.
```

```typescript
// Selection session (e.g. a scrolling picker). No-ops on Web.
await Haptics.selectionStart();
await Haptics.selectionChanged();
await Haptics.selectionEnd();

// Custom pattern (iOS Core Haptics; Android/Web approximated).
await Haptics.playPattern({
  events: [
    { time: 0, intensity: 1.0, sharpness: 0.5 }, // Transient tap.
    { time: 0.2, intensity: 0.6, sharpness: 0.7, duration: 0.5 }, // Continuous.
  ],
});
```

```typescript
import { Haptics, AndroidHapticType } from '@capawesome/capacitor-haptics';

// Android only: semantic view-system haptics that respect the user's settings.
await Haptics.performAndroidHaptic({ type: AndroidHapticType.Confirm });
```

Enum values: `ImpactStyle` — `LIGHT`, `MEDIUM`, `HEAVY`, `SOFT`, `RIGID`. `NotificationType` — `SUCCESS`, `WARNING`, `ERROR`. `AndroidHapticType` — `CLOCK_TICK`, `CONFIRM`, `CONTEXT_CLICK`, `KEYBOARD_TAP`, `LONG_PRESS`, `REJECT`, `TOGGLE_OFF`, `TOGGLE_ON`, `VIRTUAL_KEY`. `ErrorCode` — `PATTERN_PLAYBACK_FAILED`.

## Notes

- `impact()` and `notification()` map their style/type to a best-effort vibration effect on Android.
- `playPattern(...)`: `HapticEvent.intensity` (0-1) is required; `sharpness` (0-1, default 0.5) is iOS-only; `duration` (seconds) makes the event continuous rather than a transient tap; `time` is the relative start in seconds. On Android intensity is only respected on devices with amplitude control; on iOS it requires a device with haptic event playback (a Taptic Engine) or the call rejects as unavailable.
- `vibrate({ duration })` is only available on Android and Web (iOS ignores the duration).
- `selectionStart()`, `selectionChanged()`, and `selectionEnd()` do nothing on Web.
- `performAndroidHaptic(...)` is Android-only and uses semantic haptic feedback constants that honor the user's system haptic settings. Several types fall back to another effect on older Android versions (e.g. `TOGGLE_ON`/`TOGGLE_OFF` fall back to `CLOCK_TICK` on Android 13 and older).
