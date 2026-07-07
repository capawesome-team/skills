# Battery

Capacitor plugin to access battery information: battery level, battery state, low power mode, and change events.

**Package:** `@capawesome/capacitor-battery`

**Platforms:** Android, iOS, Web (partial)

## Installation

```bash
npm install @capawesome/capacitor-battery
npx cap sync
```

No additional configuration or permissions are required on Android or iOS.

## Usage

```typescript
import { Battery } from '@capawesome/capacitor-battery';

const getBatteryLevel = async () => {
  const { level } = await Battery.getBatteryLevel();
  return level; // 0.0 – 1.0
};

const getBatteryState = async () => {
  const { state } = await Battery.getBatteryState();
  return state; // 'charging' | 'full' | 'unplugged' | 'unknown'
};

const isLowPowerModeEnabled = async () => {
  const { enabled } = await Battery.isLowPowerModeEnabled();
  return enabled;
};

const addListeners = async () => {
  await Battery.addListener('batteryLevelChange', event => {
    console.log('Battery level changed:', event.level);
  });
  await Battery.addListener('batteryStateChange', event => {
    console.log('Battery state changed:', event.state);
  });
  await Battery.addListener('lowPowerModeChange', event => {
    console.log('Low power mode changed:', event.enabled);
  });
};
```

## Notes

- `getBatteryLevel()`, `getBatteryState()`, and their change listeners are available on Android, iOS, and Web. `isLowPowerModeEnabled()` and `lowPowerModeChange` are Android and iOS only.
- The device is only observed while at least one listener is attached.
- Android: level/state are read from the sticky `ACTION_BATTERY_CHANGED` broadcast; low power mode reflects the power saver mode.
- iOS: the battery level is not available on the iOS Simulator, so `getBatteryLevel()` rejects there (use a real device); low power mode reflects Low Power Mode.
- Web: level and state are only available in browsers that implement the [Battery Status API](https://developer.mozilla.org/en-US/docs/Web/API/Battery_Status_API) (Chromium-based browsers). Low power mode is not available on Web.
