# Text Zoom

Get and set the text zoom of the WebView to respect the user's font size preferences.

**Package:** `@capawesome/capacitor-text-zoom`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-text-zoom
npx cap sync
```

On Web, all methods reject as unimplemented.

## Configuration

### iOS

The text zoom is implemented via the `-webkit-text-size-adjust` CSS property, which only takes effect when the WebView runs in mobile content mode. On iPad the WebView defaults to desktop content mode, so `setZoom(...)` has no visible effect unless you set the following in `capacitor.config`:

```json
{
  "ios": {
    "preferredContentMode": "mobile"
  }
}
```

## Usage

```typescript
import { TextZoom } from '@capawesome/capacitor-text-zoom';

// Read the current WebView zoom (1.0 = 100%).
const { zoom } = await TextZoom.getZoom();

// Read the OS font size preference and apply it.
const { zoom: preferred } = await TextZoom.getPreferredZoom();
await TextZoom.setZoom({ zoom: preferred });

// Set a custom zoom (must be greater than 0).
await TextZoom.setZoom({ zoom: 1.5 });
```

## Notes

- `getPreferredZoom()` returns the zoom derived from the operating system's font size settings; pass it to `setZoom(...)` to honor the user's preference.
- The zoom is not persisted across app restarts. Persist it with a preferences plugin (e.g. `@capacitor/preferences`) and re-apply it on app launch if needed.
- `zoom` uses `1.0` for `100%` (no zoom); it must be greater than `0`.
