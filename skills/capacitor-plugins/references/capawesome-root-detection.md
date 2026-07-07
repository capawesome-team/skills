# Root Detection

Detect rooted and jailbroken devices, emulators/simulators, and developer mode.

**Package:** `@capawesome/capacitor-root-detection`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-root-detection
npx cap sync
```

## Configuration

### Android

#### Variables

This plugin uses the following project variable (defined in your app's `android/app/variables.gradle` file):

- `$rootbeerVersion` version of `com.scottyab:rootbeer-lib` (default: `0.1.0`)

### iOS

#### Application Queries Schemes

`isRooted()` checks whether the `cydia://` URL scheme can be opened as part of its jailbreak heuristics. This step is optional — if the scheme is not added, `isRooted()` still works but skips the URL scheme check. To enable it, add the `cydia` scheme to the `LSApplicationQueriesSchemes` array in `ios/App/App/Info.plist`:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>cydia</string>
</array>
```

## Usage

### Detect root/jailbreak, emulator, and developer mode

```typescript
import { RootDetection } from '@capawesome/capacitor-root-detection';

const { rooted } = await RootDetection.isRooted();
const { emulator } = await RootDetection.isEmulator();
const { enabled } = await RootDetection.isDeveloperModeEnabled(); // Android only
```

## Notes

- These are **best-effort, client-side** checks that can be bypassed or spoofed by a determined attacker with control over a rooted/jailbroken device. Never rely on this plugin as the sole security measure.
- For **server-verifiable** device and app integrity, use the App Integrity plugin (Play Integrity API on Android, App Attest on iOS).
- `isDeveloperModeEnabled()` is only available on Android.
- `isRooted()` maps to root detection on Android and jailbreak detection on iOS; `isEmulator()` maps to emulator detection on Android and simulator detection on iOS.
- The `$rootbeerVersion` variable in `variables.gradle` can override the default `com.scottyab:rootbeer-lib` version.
