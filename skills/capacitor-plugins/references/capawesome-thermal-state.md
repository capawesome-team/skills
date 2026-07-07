# Thermal State

Read the device thermal state and react before the OS throttles your app.

**Package:** `@capawesome/capacitor-thermal-state`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-thermal-state
npx cap sync
```

## Usage

### Read the thermal state and listen for changes

```typescript
import { ThermalState } from '@capawesome/capacitor-thermal-state';

const { state } = await ThermalState.getThermalState(); // 'nominal' | 'fair' | 'serious' | 'critical'

await ThermalState.addListener('thermalStateChange', event => {
  console.log(event.state);
});

await ThermalState.removeAllListeners();
```

## Notes

- The thermal state is one of `nominal`, `fair`, `serious`, or `critical`. Use it to progressively reduce your app's workload (e.g. lower the frame rate, pause prefetching, defer background/ML work) before the OS throttles it.
- **Android requires API level 29 (Android 10) or newer.** On older versions, `getThermalState()` rejects as unavailable and the `thermalStateChange` event is never emitted.
- The device is only observed while at least one listener is attached.
- On Web, all methods reject as unimplemented.
