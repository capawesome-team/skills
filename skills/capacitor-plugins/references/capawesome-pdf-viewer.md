# PDF Viewer

Open a local PDF file in a fullscreen native viewer.

**Package:** `@capawesome/capacitor-pdf-viewer`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-pdf-viewer
npx cap sync
```

On Web, all methods reject as unimplemented. Browsers ship a built-in PDF viewer, so render a PDF with an `<iframe>` or `<object>` element instead.

## Configuration

### Android

#### Variables

This plugin uses the following project variable (defined in your app's `android/variables.gradle` file):

- `$androidPdfViewerVersion` version of `com.github.mhiew:android-pdf-viewer` (default: `3.2.0-beta.3`)

## Usage

### Open and close

```typescript
import { PdfViewer } from '@capawesome/capacitor-pdf-viewer';

await PdfViewer.open({
  path: 'file:///data/user/0/.../cache/document.pdf',
  title: 'Invoice',
  page: 3,
});

await PdfViewer.close();
```

### Open a password-protected PDF

```typescript
import { PdfViewer } from '@capawesome/capacitor-pdf-viewer';

await PdfViewer.open({
  path: 'file:///.../secret.pdf',
  password: 'secret',
});
```

### Listen for events

```typescript
import { PdfViewer } from '@capawesome/capacitor-pdf-viewer';

await PdfViewer.addListener('pageChange', ({ page }) => {
  console.log('Current page:', page);
});

await PdfViewer.addListener('closed', () => {
  console.log('Viewer closed');
});
```

## Notes

- Only local files are supported. Remote URLs must be downloaded first (e.g. Filesystem `downloadFile(...)`) and the local path passed to `open(...)`.
- Opening a new viewer while one is already open closes the previous one first.
- On Android, the underlying android-pdf-viewer library renders with Pdfium and bundles native libraries that add about 10-16 MB (uncompressed, across all ABIs) to your app. Publishing as an Android App Bundle ships only the current device's ABI, greatly reducing the download size. Text selection is not supported on Android.
- On iOS, the viewer uses PDFKit; no additional configuration is required.
- `ErrorCode` values: `FILE_NOT_FOUND`, `LOAD_FAILED`, `PASSWORD_INVALID`, `PASSWORD_REQUIRED`.
