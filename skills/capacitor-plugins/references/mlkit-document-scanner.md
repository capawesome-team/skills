# MLKit Document Scanner

Unofficial Capacitor plugin for ML Kit Document Scanner. Opens the ML Kit document scanning UI to capture and export documents as images or PDF.

**Platforms:** Android

## Installation

```bash
npm install @capacitor-mlkit/document-scanner
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variable: `mlkitDocumentScannerVersion` (default: `16.0.0`) for `com.google.android.gms:play-services-mlkit-document-scanner`.

### iOS

Not supported. All methods reject as unavailable on iOS (and unimplemented on the Web).

## Usage

```typescript
import { DocumentScanner, GoogleDocumentScannerModuleInstallState } from '@capacitor-mlkit/document-scanner';

const { available } = await DocumentScanner.isGoogleDocumentScannerModuleAvailable();
if (!available) {
  await DocumentScanner.addListener('googleDocumentScannerModuleInstallProgress', (event) => {
    if (event.state === GoogleDocumentScannerModuleInstallState.COMPLETED) {
      console.log('Module installed');
    }
  });
  await DocumentScanner.installGoogleDocumentScannerModule();
}

const { scannedImages, pdf } = await DocumentScanner.scanDocument({
  galleryImportAllowed: true,
  pageLimit: 5,
  resultFormats: 'JPEG_PDF',
  scannerMode: 'FULL',
});
console.log(scannedImages, pdf?.uri, pdf?.pageCount);
```

## Notes

- Android-only. Requires API level 21+ and a device with at least 1.7 GB total RAM (otherwise rejects with an `UNSUPPORTED` error).
- The scanner models and UI are downloaded on demand by Google Play services; use `installGoogleDocumentScannerModule()` and the `googleDocumentScannerModuleInstallProgress` event to manage installation.
- `resultFormats` (`'JPEG' | 'PDF' | 'JPEG_PDF'`) and `scannerMode` (`'FULL' | 'BASE' | 'BASE_WITH_FILTER'`) are string-literal unions.
