# Screen Brightness

Read and control the screen brightness.

**Package:** `@capawesome/capacitor-screen-brightness`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-screen-brightness
npx cap sync
```

## Usage

### Get, set, and reset brightness

```typescript
import { ScreenBrightness } from '@capawesome/capacitor-screen-brightness';

const { brightness } = await ScreenBrightness.getBrightness(); // 0.0 (darkest) - 1.0 (brightest)

await ScreenBrightness.setBrightness({ brightness: 1 });

await ScreenBrightness.resetBrightness(); // Android only
```

## Notes

- `brightness` is a value between `0.0` (darkest) and `1.0` (brightest).
- **Android**: `setBrightness()` only affects the current app window and does not require the `WRITE_SETTINGS` permission. The change is reverted automatically as soon as the app is closed. Use `resetBrightness()` to hand control back to the OS while the app is running.
- **iOS**: `setBrightness()` changes the **system** brightness and the change **persists** after the app is closed. There is no system API to hand control back to the OS, so `resetBrightness()` is not available on iOS and rejects as unimplemented.
- On Web, all methods reject as unimplemented.
