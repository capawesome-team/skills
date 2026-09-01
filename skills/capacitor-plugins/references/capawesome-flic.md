# Flic

Unofficial Capacitor plugin for Flic smart buttons: pair, connect, and receive button events.

**Package:** `@capawesome/capacitor-flic`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/flic/

## Installation

```bash
npm install @capawesome/capacitor-flic
npx cap sync
```

## Configuration

### Android

#### Permissions

No configuration is required. The plugin declares all required Bluetooth permissions itself.

#### Repositories

The `flic2lib-android` library is resolved from [JitPack](https://jitpack.io/), which the plugin declares in its own `build.gradle`. If your project restricts repository declarations to the settings file (e.g. `dependencyResolutionManagement` with `FAIL_ON_PROJECT_REPOS`), add JitPack to `android/settings.gradle`:

```groovy
dependencyResolutionManagement {
    repositories {
        maven { url 'https://jitpack.io' }
    }
}
```

#### Variables

Defined in your app's `variables.gradle`:

- `$flic2libVersion` version of `com.github.50ButtonsEach:flic2lib-android` (default: `2.0.1`)

### iOS

#### Privacy Descriptions

**Required.** Add to `ios/App/App/Info.plist`:

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>The app needs access to Bluetooth to communicate with your Flic buttons.</string>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>The app needs access to Bluetooth to communicate with your Flic buttons.</string>
```

#### Background Mode

To receive button events while the app is in the background, enable the `Uses Bluetooth LE accessories` background mode in **Signing & Capabilities** and call `initialize({ iosBackground: true })`.

## Usage

### Initialize

```typescript
import { Flic } from '@capawesome/capacitor-flic';

await Flic.initialize({ iosBackground: true });
```

### Pair a new button

```typescript
import { Flic, ScanStatus } from '@capawesome/capacitor-flic';

await Flic.addListener('scanStatusChanged', (event) => {
  if (event.status === ScanStatus.Discovered) {
    console.log('Button discovered. Keep holding it...');
  }
});

const { button } = await Flic.startScan();
```

### Connect paired buttons

```typescript
import { Flic } from '@capawesome/capacitor-flic';

const { buttons } = await Flic.getButtons();
for (const button of buttons) {
  await Flic.connectButtonById({ id: button.id });
}

await Flic.disconnectButtonById({ id: buttons[0].id });
await Flic.forgetButtonById({ id: buttons[0].id });
```

### Listen for button events

```typescript
import { Flic } from '@capawesome/capacitor-flic';

await Flic.addListener('buttonSingleClick', (event) => {
  console.log('Clicked:', event.buttonId, event.wasQueued);
});
await Flic.addListener('buttonDoubleClick', (event) => {
  console.log('Double clicked:', event.buttonId);
});
await Flic.addListener('buttonHold', (event) => {
  console.log('Held down:', event.buttonId);
});
```

### Permissions

```typescript
import { Flic } from '@capawesome/capacitor-flic';

let status = await Flic.checkPermissions();
if (status.bluetoothScan !== 'granted') {
  status = await Flic.requestPermissions();
}
```

## Notes

- `initialize(...)` must be called before every other method, as early as possible after app launch to minimize the delay of queued button events. On iOS it triggers the Bluetooth permission prompt.
- To pair a button, press and hold it for at least 6 seconds while `startScan()` runs. Only one scan can run at a time; `stopScan()` rejects the pending `startScan()` call.
- `connectButtonById(...)` resolves immediately — the connection is established once the button is available and never times out. Wait for `buttonConnected` and `buttonReady`.
- After an app restart, every paired button must be connected again.
- Events carry `wasQueued: true` if they occurred while the button was disconnected.
- `Button.batteryVoltage` is only present once a sample has been taken; hint at a battery change below `2.65` volts.
- If `Button.isUnpaired` is true (e.g. after a factory reset), call `forgetButtonById(...)` and pair the button again.
- Android permissions: `BLUETOOTH_SCAN`/`BLUETOOTH_CONNECT` (Android 12+) and `ACCESS_FINE_LOCATION` (Android 11 and earlier). On iOS, `requestPermissions()` only returns the current status.
- Background delivery on Android requires the app process to stay alive — consider the Android Foreground Service plugin.
- Supports Flic 2 buttons. For the Flic Duo, only the big button's events are delivered.
- Not available on web. This project is not affiliated with Shortcut Labs AB.
