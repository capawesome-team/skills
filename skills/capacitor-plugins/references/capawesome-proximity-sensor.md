# Proximity Sensor

Read the device's proximity sensor to detect whether an object is close to the screen.

**Package:** `@capawesome/capacitor-proximity-sensor`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-proximity-sensor
npx cap sync
```

### Android

If you are using Proguard, add the following to your `proguard-rules.pro` file:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

## Usage

### Read a single measurement

```typescript
import { ProximitySensor } from '@capawesome/capacitor-proximity-sensor';

const { available } = await ProximitySensor.isAvailable();

const measurement = await ProximitySensor.getMeasurement();
console.log('Near:', measurement.near); // boolean
console.log('Distance:', measurement.distance); // cm, Android only; null on iOS
```

### Listen for continuous updates

```typescript
import { ProximitySensor } from '@capawesome/capacitor-proximity-sensor';

await ProximitySensor.addListener('measurement', (measurement) => {
  console.log('Near:', measurement.near);
  console.log('Distance:', measurement.distance);
});
await ProximitySensor.startMeasurementUpdates();

// Later:
await ProximitySensor.stopMeasurementUpdates();
await ProximitySensor.removeAllListeners();
```

## Notes

- `near` is a boolean available on both platforms. `distance` (centimeters) is **Android only** and is always `null` on iOS. Most devices only report near/far, so `distance` is typically `0` (near) or the sensor's maximum range (far).
- **iOS screen-dimming side effect:** Reading the proximity sensor requires enabling proximity monitoring via `UIDevice`. While monitoring is enabled, iOS automatically turns off the screen whenever an object (an ear, a hand) is close to the sensor — exactly like during a phone call. This is system-level behavior that cannot be disabled.
  - `getMeasurement()` enables monitoring, reads the state, then disables it again.
  - `startMeasurementUpdates()` keeps monitoring (and the dimming behavior) enabled until `stopMeasurementUpdates()` or `removeAllListeners()` is called. Only start updates when you actually want this behavior.
- On Android, reading the proximity sensor has no effect on the screen.
- All methods are only available on Android and iOS; there is no web implementation.
