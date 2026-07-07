# Light Sensor

Read the device's ambient light sensor in lux.

**Package:** `@capawesome/capacitor-light-sensor`
**Platforms:** Android

## Installation

```bash
npm install @capawesome/capacitor-light-sensor
npx cap sync
```

This plugin is Android-only. iOS provides no public API to read the ambient light sensor, so it is not implemented there (the closest signal is screen brightness via the Screen Brightness plugin). On iOS and Web, all methods reject as unimplemented.

### Android

If you use Proguard, add the following rule to your `proguard-rules.pro` file:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

## Usage

### One-off measurement and availability

```typescript
import { LightSensor } from '@capawesome/capacitor-light-sensor';

const { available } = await LightSensor.isAvailable();

const measurement = await LightSensor.getMeasurement();
console.log('Illuminance:', measurement.illuminance); // lux (lx)
```

### Continuous updates

```typescript
await LightSensor.addListener('measurement', measurement => {
  console.log('Illuminance:', measurement.illuminance);
});
await LightSensor.startMeasurementUpdates();

// Later:
await LightSensor.stopMeasurementUpdates();
await LightSensor.removeAllListeners();
```

## Notes

- `getMeasurement()` returns the most recent measurement; `Measurement.illuminance` is the ambient light level in lux (lx).
- Call `startMeasurementUpdates()` to begin emitting `measurement` events and `stopMeasurementUpdates()` to stop them.
- Always check `isAvailable()` first — not every Android device has an ambient light sensor.
- No error codes are defined for this plugin.
