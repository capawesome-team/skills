# Maps Launcher

Launch navigation apps (Google Maps, Apple Maps, Waze) with turn-by-turn directions.

**Package:** `@capawesome/capacitor-maps-launcher`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-maps-launcher
npx cap sync
```

## Configuration

### Android

The plugin declares the required package visibility `<queries>` for Google Maps and Waze in its own `AndroidManifest.xml`. No additional configuration is required.

### iOS

#### Application Queries Schemes

To detect and launch Google Maps and Waze, add their URL schemes to the `LSApplicationQueriesSchemes` array in `ios/App/App/Info.plist`. Without them, `getAvailableApps()` reports those apps as unavailable and `navigate(...)` rejects with the `APP_NOT_AVAILABLE` error code.

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>comgooglemaps</string>
  <string>waze</string>
</array>
```

Apple Maps is always available and requires no configuration.

## Usage

```typescript
import { MapsLauncher, NavigationApp } from '@capawesome/capacitor-maps-launcher';

// Navigate to coordinates with a specific app and travel mode.
await MapsLauncher.navigate({
  destination: { latitude: 37.3349, longitude: -122.009 },
  app: NavigationApp.GoogleMaps,
  travelMode: 'driving',
});

// Navigate to an address using the system default app.
await MapsLauncher.navigate({
  destination: { address: 'Apple Park, Cupertino, CA' },
});

// Query available and default apps.
const { apps } = await MapsLauncher.getAvailableApps();
const { app } = await MapsLauncher.getDefaultApp(); // Android only; may be null
```

## Notes

- Supported apps (`NavigationApp` enum): `AppleMaps` (`'appleMaps'`, iOS only), `GoogleMaps` (`'googleMaps'`), `Waze` (`'waze'`).
- A `destination` is defined either by `latitude`/`longitude` or by `address`, not both.
- If no `app` is provided, the system default behavior is used (a chooser on Android, Apple Maps on iOS).
- `getDefaultApp()` is Android-only and returns `null` if the default is not a curated app or no default is set.
- Travel modes (`'driving'` | `'walking'` | `'bicycling'` | `'transit'`, default `'driving'`) are best-effort per app: Apple Maps supports driving/walking/transit; Google Maps supports all four; Waze supports only driving and ignores the option.
- The `start` option is best-effort: Apple Maps supports it fully, Google Maps opens the directions preview instead of starting navigation, and Waze ignores it. If omitted, the device's current location is used.
- `ErrorCode` values: `APP_NOT_AVAILABLE`, `LAUNCH_FAILED`.
