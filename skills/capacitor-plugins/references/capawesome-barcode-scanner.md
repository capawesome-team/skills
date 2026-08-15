# Barcode Scanner

Capacitor plugin for scanning barcodes and QR codes with a themeable native fullscreen scanner or an embedded camera view. Supports 13 barcode formats and reading barcodes from image files.

**Package:** `@capawesome-team/capacitor-barcode-scanner`
**Platforms:** Android, iOS, Web
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-barcode-scanner
npx cap sync
```

## Configuration

### Android

The plugin declares the camera permission in its own manifest, so no changes to `android/app/src/main/AndroidManifest.xml` are required.

#### Variables

Optionally define these variables in `android/variables.gradle` to override the default dependency versions (useful when other plugins cause dependency conflicts):

```groovy
androidxCameraVersion = '1.6.1'
mlkitBarcodeScanningVersion = '17.3.0'
```

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Privacy Descriptions

Add to `ios/App/App/Info.plist`:

```xml
<key>NSCameraUsageDescription</key>
<string>The app needs access to the camera to scan barcodes.</string>
```

If the key is missing, `scan(...)` and `startScan(...)` reject with a clear error message.

### Web

The web implementation uses the [`BarcodeDetector`](https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector) API, which is not available in all browsers. For browsers without built-in support, install the `barcode-detector` polyfill (it is intentionally not bundled with the plugin) and import it once in your app entry file:

```typescript
import 'barcode-detector/side-effects';
```

## Usage

### Check availability and request permissions

```typescript
import { BarcodeScanner } from '@capawesome-team/capacitor-barcode-scanner';

const { available } = await BarcodeScanner.isAvailable();
const { camera } = await BarcodeScanner.requestPermissions();
if (camera === 'denied') {
  await BarcodeScanner.openSettings();
}
```

### Scan with the fullscreen scanner

```typescript
import {
  BarcodeFormat,
  BarcodeScanner,
} from '@capawesome-team/capacitor-barcode-scanner';

const { barcodes } = await BarcodeScanner.scan({
  batch: false,
  formats: [BarcodeFormat.QrCode, BarcodeFormat.Ean13],
  ui: { accentColor: '#59C7F9', title: 'Scan Barcode' },
});
const barcode = barcodes[0];
```

Set `batch: true` to collect multiple barcodes in one session and resolve with all of them when the user taps the done button.

### Scan with the embedded camera view

```typescript
import {
  BarcodeScanner,
  LensFacing,
} from '@capawesome-team/capacitor-barcode-scanner';

const getScanFrame = () => {
  const rect = document.querySelector('#scanner').getBoundingClientRect();
  return { x: rect.x, y: rect.y, width: rect.width, height: rect.height };
};

await BarcodeScanner.addListener('barcodesScanned', (event) => {
  console.log('Scanned barcodes:', event.barcodes);
});
await BarcodeScanner.startScan({
  frame: getScanFrame(),
  lensFacing: LensFacing.Back,
});
window.addEventListener('resize', () =>
  BarcodeScanner.setScanFrame({ frame: getScanFrame() }),
);

// Later:
await BarcodeScanner.stopScan();
await BarcodeScanner.removeAllListeners();
```

### Overlay HTML elements over the embedded camera view

Set `placement` to `PreviewPlacement.Behind` to render the camera preview behind the web view. The preview is only visible where the app is transparent, so the placeholder element, all its ancestors and the `body` must have a transparent background over the frame area:

```css
body,
#scanner {
  background: transparent;
}
```

With Ionic Framework UI components, also set `--background: transparent` on the surrounding `ion-content` element.

```typescript
import {
  BarcodeScanner,
  PreviewPlacement,
} from '@capawesome-team/capacitor-barcode-scanner';

await BarcodeScanner.startScan({
  frame: getScanFrame(),
  placement: PreviewPlacement.Behind,
});
```

### Read barcodes from an image

```typescript
import { BarcodeScanner } from '@capawesome-team/capacitor-barcode-scanner';

const { barcodes } = await BarcodeScanner.readBarcodesFromImage({ path });
```

On Web, `path` must be a URL that can be fetched by the browser.

## Notes

- `scan(...)`, `openSettings()` and torch/zoom control (`setTorchEnabled(...)`, `setZoomRatio(...)`, `getZoomRatioRange()`) are Android and iOS only. `startScan(...)` and `readBarcodesFromImage(...)` also work on Web.
- Error codes: `SCAN_CANCELED` (user closed the fullscreen scanner), `ALREADY_SCANNING` (session already active), `NOT_SCANNING` (no active session).
- Events: `barcodesScanned` and `scanError`. `pauseScan()` / `resumeScan()` stop and restart detection without stopping the camera preview.
- `startScan(...)` options: `detectionArea` (region of interest inside the frame), `duplicateTimeout` (default `1500` ms before the same barcode is emitted again), `formats`, `frame`, `lensFacing`, `placement` (default `PreviewPlacement.Above`).
- `ui` options for `scan(...)`: `accentColor`, `beep` (default `false`), `hapticFeedback` (default `true`), `instructions`, `showFlipCameraButton` (default `false`), `showTorchButton` (default `true`), `title`. Only barcodes fully inside the viewfinder are detected.
- Always set `formats` to only the formats you need to improve detection performance.
- `Barcode` result: `displayValue`, `rawValue`, `format`, `cornerPoints` (CSS pixels relative to the scan frame, the screen, or the image), `bytes` (Android only, `null` on iOS and Web).
- `BarcodeFormat` members: `Aztec`, `Codabar`, `Code39`, `Code93`, `Code128`, `DataMatrix`, `Ean8`, `Ean13`, `Itf`, `Pdf417`, `QrCode`, `UpcA`, `UpcE`. On iOS, camera scanning of Codabar requires iOS 15.4+. On Web, supported formats depend on the browser or polyfill.
- Scanning works offline: Android uses the bundled ML Kit model (no Google Play services required, increases app size by a few megabytes), iOS uses AVFoundation and Vision (no third-party pods).
- Structured payload parsing (WiFi credentials, vCard, etc.) is not supported. Parse `rawValue` yourself.
