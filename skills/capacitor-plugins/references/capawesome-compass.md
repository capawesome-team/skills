# Compass

Capacitor plugin for reading the device compass heading, on demand or as continuous live updates.

**Package:** `@capawesome/capacitor-compass`

**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-compass
npx cap sync
```

All methods reject as unimplemented on Web.

## Configuration

### iOS

The magnetic heading is available without any permission. To also read the true (geographic) north heading, location services must be enabled and the `NSLocationWhenInUseUsageDescription` key must be added to `ios/App/App/Info.plist`:

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Your location is used to determine the true (geographic) north heading.</string>
```

If the key is missing or location permission is not granted, the `trueHeading` value is `null`.

## Usage

```typescript
import { Compass } from '@capawesome/capacitor-compass';

const isAvailable = async () => {
  const { available } = await Compass.isAvailable();
  return available;
};

const getHeading = async () => {
  const heading = await Compass.getHeading();
  console.log(heading.magneticHeading, heading.trueHeading, heading.accuracy);
  return heading;
};

const startHeadingUpdates = async () => {
  await Compass.addListener('headingChange', heading => {
    console.log(heading.magneticHeading);
  });
  await Compass.startHeadingUpdates();
};

const stopHeadingUpdates = async () => {
  await Compass.stopHeadingUpdates();
  await Compass.removeAllListeners();
};
```

## Notes

- Call `startHeadingUpdates()` to begin emitting `headingChange` events and `stopHeadingUpdates()` to stop.
- `magneticHeading` ranges from `0` to `360` degrees (`0` = magnetic north).
- `trueHeading` is always `null` on Android. On iOS it requires location services enabled and `NSLocationWhenInUseUsageDescription`; otherwise `null`.
- `accuracy` is the maximum deviation in degrees; a negative value or `null` means invalid or unknown.
