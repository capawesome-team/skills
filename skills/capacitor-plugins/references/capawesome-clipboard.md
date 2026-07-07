# Clipboard

Read from and write to the system clipboard, including images, HTML, and URLs.

**Package:** `@capawesome/capacitor-clipboard`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-clipboard
npx cap sync
```

No additional permissions or configuration are required on any platform.

## Usage

```typescript
import { Clipboard, ClipboardContentType } from '@capawesome/capacitor-clipboard';

// Write exactly one of text, html, image, or url (html may be combined with text as fallback).
await Clipboard.write({ text: 'Hello World' });
await Clipboard.write({ html: '<b>Hello World</b>', text: 'Hello World' });
await Clipboard.write({ image: 'data:image/png;base64,iVBORw0KGgo...' });
await Clipboard.write({ url: 'https://capawesome.io' });
```

```typescript
const { type, value } = await Clipboard.read();
// type: ClipboardContentType — 'TEXT' | 'HTML' | 'IMAGE' | 'URL'
// value: string (images are returned as a Base64-encoded data URL)
```

`ClipboardContentType` values: `TEXT`, `HTML`, `IMAGE`, `URL`.
`ErrorCode` values: `EMPTY_CLIPBOARD`, `READ_FAILED`, `WRITE_FAILED`.

## Notes

- `write(...)` requires exactly one of `text`, `html`, `image`, or `url`. Optionally pair `html` with `text` for a plain-text fallback. `label` (describing the clipboard content) is Android-only.
- On Android 10+ (API 29+), the clipboard can only be read while the app is in the foreground with input focus; otherwise `read()` rejects.
- On Android 12+ (API 31+), the system shows a toast when the app reads clipboard content written by another app. This cannot be suppressed.
- On iOS 14+, reading the clipboard shows a system paste notification banner. This cannot be suppressed.
- Drop-in alternative to `@capacitor/clipboard` with real image support on Android and HTML support on all platforms. Note the property rename: `write({ string })` becomes `write({ text })`, and `read()` returns `type: 'TEXT'` instead of `'text/plain'`.
