# Document Scanner

Capacitor plugin for scanning documents with the native, full-screen scanner on Android (ML Kit) and iOS (VisionKit). Returns perspective-corrected JPEG images and an optional combined PDF.

**Package:** `@capawesome-team/capacitor-document-scanner`
**Platforms:** Android, iOS
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-document-scanner
npx cap sync
```

## Configuration

No `capacitor.config.ts` configuration is required for this plugin.

### Android

No `android/app/src/main/AndroidManifest.xml` changes are required. The Google-provided scanner activity requests the camera permission itself.

#### Variables

To change the default dependency version, add to `android/variables.gradle` inside the `ext` block:

```groovy
playServicesMlkitDocumentScannerVersion = '16.0.0'
```

This is only needed to resolve dependency conflicts with other plugins.

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Privacy Descriptions

Add to `ios/App/App/Info.plist` inside the top-level `dict`:

```xml
<key>NSCameraUsageDescription</key>
<string>The app needs access to the camera to scan documents.</string>
```

If the key is missing, `scanDocument(...)` rejects with a clear error message.

## Usage

### Check the availability

```typescript
import { DocumentScanner } from '@capawesome-team/capacitor-document-scanner';

const { available } = await DocumentScanner.isAvailable();
```

### Scan a document

```typescript
import { DocumentScanner } from '@capawesome-team/capacitor-document-scanner';

const { scannedImages } = await DocumentScanner.scanDocument({
  imageQuality: 80,
  pageLimit: 5,
});
```

### Generate a PDF document

```typescript
import { DocumentScanner } from '@capawesome-team/capacitor-document-scanner';

const { pdf } = await DocumentScanner.scanDocument({
  generatePdf: true,
});
```

### Configure the Android scanner user interface

```typescript
import { DocumentScanner, ScannerMode } from '@capawesome-team/capacitor-document-scanner';

const { scannedImages } = await DocumentScanner.scanDocument({
  androidScannerMode: ScannerMode.BaseWithFilter,
  androidGalleryImportAllowed: true,
});
```

## Notes

- `isAvailable()` and `scanDocument(...)` are only available on Android and iOS. On the web both methods reject with an unimplemented error.
- `scanDocument(...)` rejects with the `SCAN_CANCELED` error code if the user cancels the scan.
- `scanDocument(...)` options: `imageQuality` (0–100, default `100`), `pageLimit` (>= 1, default `10`), `generatePdf` (default `false`), `androidScannerMode` (default `ScannerMode.Full`), `androidGalleryImportAllowed` (default `false`, Android only).
- `ScannerMode` values: `Base` (crop and rotate), `BaseWithFilter` (adds grayscale and auto enhancement), `Full` (adds ML-based cleaning of stains, fingers and shadows).
- `scanDocument(...)` resolves with `scannedImages` (paths of the JPEG files) and `pdf` (path of the PDF, or `null` if `generatePdf` is not `true`).
- Scanned images and the PDF are stored in the app's cache directory. Stale plugin files are cleaned up when the plugin loads, so copy files to a persistent location if they must be kept.
- On iOS, `pageLimit` cannot stop the scanner; the returned pages are truncated to that value after scanning.
- On Android, the scanner is an on-demand Google Play services module that is downloaded the first time `scanDocument(...)` is called, so `isAvailable()` only reflects Google Play services availability.
- Unlike `@capacitor-mlkit/document-scanner` (Android-only, ML Kit based), this plugin provides one unified API across Android and iOS.
- Pass the resulting `pdf` path to `@capawesome/capacitor-pdf-viewer`, `@capawesome-team/capacitor-printer` or `@capawesome-team/capacitor-file-opener`.
