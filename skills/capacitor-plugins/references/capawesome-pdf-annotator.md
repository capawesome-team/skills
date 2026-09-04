# PDF Annotator

Let the user annotate a local PDF file in a fullscreen native viewer with the platform's markup tools.

**Package:** `@capawesome/capacitor-pdf-annotator`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/pdf-annotator/

## Installation

```bash
npm install @capawesome/capacitor-pdf-annotator
npx cap sync
```

On Web, `isAvailable()` resolves to `false` and `open(...)` rejects as unimplemented.

## Configuration

### Android

#### SDK Extension Level

The Jetpack PDF library used by the plugin requires the app to be compiled against Android SDK 36 with SDK extension level 19 or higher. Add the `compileSdkExtension` property to the `android` block of `android/app/build.gradle`:

```groovy
android {
    compileSdk = rootProject.ext.compileSdkVersion
    compileSdkExtension = 19
    // ...
}
```

This requires the `Android SDK Platform 36-ext19` package (`sdkmanager "platforms;android-36-ext19"` or the SDK Manager of Android Studio with **Show Package Details** enabled). Apps compiled against Android SDK 37 or higher (Android Gradle Plugin 9.1 or higher) do not need to set an extension level.

#### Variables

Defined in your app's `variables.gradle`:

- `$androidxPdfVersion` version of `androidx.pdf:pdf-ink` and `androidx.pdf:pdf-viewer-fragment` (default: `1.0.0-beta01`)
- `$androidxMaterialVersion` version of `com.google.android.material:material` (default: `1.13.0`)

### iOS

No configuration is required. The plugin uses the Quick Look framework.

## Usage

### Check the availability

```typescript
import { PdfAnnotator } from '@capawesome/capacitor-pdf-annotator';

const { available } = await PdfAnnotator.isAvailable();
```

### Annotate a PDF document

```typescript
import { ErrorCode, PdfAnnotator } from '@capawesome/capacitor-pdf-annotator';

try {
  const { path } = await PdfAnnotator.open({
    path: 'file:///data/user/0/.../cache/document.pdf',
  });
  console.log('Annotated file:', path);
} catch (error) {
  if (error.code === ErrorCode.Canceled) {
    console.log('The user closed the viewer without saving.');
  }
}
```

### Keep the annotated file

```typescript
import { Directory, Filesystem } from '@capacitor/filesystem';
import { PdfAnnotator } from '@capawesome/capacitor-pdf-annotator';

const { path } = await PdfAnnotator.open({ path: 'file:///.../document.pdf' });
await Filesystem.copy({
  from: path,
  to: 'annotated.pdf',
  toDirectory: Directory.Documents,
});
```

## Notes

- Only local files are supported. Remote URLs must be downloaded first (e.g. Filesystem `downloadFile(...)`) and the local path passed to `open(...)`.
- The original file is never modified. `open(...)` resolves with the path of an annotated copy once the user saves and closes the viewer. The copy is stored in the cache directory and deleted on the next app launch, so move it to a permanent location to keep it.
- Closing the viewer without saving rejects with the `CANCELED` error code.
- On Android, annotating requires Android 11 (API level 30) or higher and a PDF system module with SDK extension level 18 or higher (ships with Android 16 QPR2, delivered to older devices via Google Play system updates). Always check `isAvailable()` first; `open(...)` rejects with `NOT_SUPPORTED` otherwise. The Android tools are pen, highlighter, eraser and undo/redo. The Jetpack PDF annotation APIs are in beta and marked experimental by Google.
- On iOS, the plugin presents the Quick Look markup UI (pen, highlighter, pencil, eraser, shapes, text, signature, undo/redo, Apple Pencil). The toolbar cannot be customized. Test on a real device, as the markup tools are not available on all simulator versions.
- `ErrorCode` values: `CANCELED`, `FILE_NOT_FOUND`, `LOAD_FAILED`, `NOT_SUPPORTED`, `SAVE_FAILED`.
