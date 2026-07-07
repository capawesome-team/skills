# Gyroscope

Read the device's gyroscope sensor (rotation rate around the x, y, and z axes).

**Package:** `@capawesome/capacitor-gyroscope`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-gyroscope
npx cap sync
```

## Configuration

### Android

#### Proguard

If you are using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Privacy Descriptions

Add the `NSMotionUsageDescription` key to `ios/App/App/Info.plist`:

```xml
<key>NSMotionUsageDescription</key>
<string>The app needs to access the motion activity.</string>
```

## Usage

### Get a single measurement

```typescript
import { Gyroscope } from '@capawesome/capacitor-gyroscope';

const { x, y, z } = await Gyroscope.getMeasurement();
```

### Continuous measurement updates

```typescript
import { Gyroscope } from '@capawesome/capacitor-gyroscope';

await Gyroscope.addListener('measurement', measurement => {
  console.log(measurement.x, measurement.y, measurement.z);
});
await Gyroscope.startMeasurementUpdates();
// Later:
await Gyroscope.stopMeasurementUpdates();
await Gyroscope.removeAllListeners();
```

### Availability and permissions

```typescript
import { Gyroscope } from '@capawesome/capacitor-gyroscope';

const { isAvailable } = await Gyroscope.isAvailable();
let status = await Gyroscope.checkPermissions();
if (status.gyroscope !== 'granted') {
  status = await Gyroscope.requestPermissions();
}
```

## Notes

- Rotation rate values `x`, `y`, and `z` are in radians per second (rad/s).
- `getMeasurement()` returns the most recent measurement from the sensor.
- The permission status (`status.gyroscope`) can be `'prompt'`, `'prompt-with-rationale'`, `'granted'`, `'denied'`, or `'limited'`.
