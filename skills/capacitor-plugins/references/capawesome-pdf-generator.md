# PDF Generator

Generate paginated PDF files from an HTML string or a URL.

**Package:** `@capawesome/capacitor-pdf-generator`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-pdf-generator
npx cap sync
```

No additional permissions or configuration are required. On Web, all methods reject as unimplemented.

## Usage

### Generate from HTML

```typescript
import { PdfGenerator, Orientation, PageSize } from '@capawesome/capacitor-pdf-generator';

const { path } = await PdfGenerator.generateFromHtml({
  html: '<h1>Hello World</h1>',
  fileName: 'invoice.pdf',
  baseUrl: 'https://capawesome.io/', // Resolves relative images/stylesheets.
  orientation: Orientation.Portrait,
  pageSize: PageSize.A4,
  timeout: 10000,
});
// path points to a file in the cache directory.
```

### Generate from URL

```typescript
import { PdfGenerator, PageSize } from '@capawesome/capacitor-pdf-generator';

const { path } = await PdfGenerator.generateFromUrl({
  url: 'https://capawesome.io/',
  fileName: 'page.pdf',
  pageSize: PageSize.A4,
});
```

## Notes

- The HTML is rendered in a hidden WebView (Android: Android Print Framework; iOS: WKWebView + UIKit) before the PDF is produced.
- `timeout` (default `30000` ms) caps how long the plugin waits for the page to finish loading. On expiry the call rejects with the `TIMEOUT` error code.
- The generated file is written to the cache directory and deleted on the next app launch. Move it to a permanent location (e.g. Filesystem `rename(...)`) if you want to keep it, or hand `path` to the PDF Viewer, Printer, File Opener, or Share plugin.
- `Orientation` values: `Orientation.Portrait` (default), `Orientation.Landscape`.
- `PageSize` values: `PageSize.A3`, `PageSize.A4` (default), `PageSize.A5`, `PageSize.Letter`.
- `ErrorCode` values: `GENERATION_FAILED`, `LOAD_FAILED`, `TIMEOUT`.
