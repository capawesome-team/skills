# Privacy Screen

Hide sensitive app content in the app switcher, block screenshots, and detect when a screenshot is taken.

**Package:** `@capawesome/capacitor-privacy-screen`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-privacy-screen
npx cap sync
```

On Web, all methods reject as unimplemented.

## Configuration

### Android

The plugin declares the `android.permission.DETECT_SCREEN_CAPTURE` permission in its own `AndroidManifest.xml`. This install-time permission is required to detect screenshots on Android 14 (API level 34) and newer. It is added automatically and requires no further configuration.

## Usage

```typescript
import { PrivacyScreen } from '@capawesome/capacitor-privacy-screen';

// Enable the privacy screen.
await PrivacyScreen.enable({
  ios: {
    preventScreenshots: true, // iOS only; opt-in, unofficial technique
  },
});

// Disable it again.
await PrivacyScreen.disable();

// Check the current state.
const { enabled } = await PrivacyScreen.isEnabled();

// Get notified when the user takes a screenshot (cannot be prevented).
await PrivacyScreen.addListener('screenshotTaken', () => {
  console.log('The user took a screenshot.');
});
await PrivacyScreen.removeAllListeners();
```

## Notes

- Android: `enable()` sets the `FLAG_SECURE` window flag, which hides content in the app switcher and blocks both screenshots and screen recordings with a single flag.
- Android: screenshot detection (`screenshotTaken`) is only available on Android 14 (API level 34) and newer; on older versions the event is never emitted.
- iOS: `enable()` installs an overlay that hides content in the app switcher. There is no official API to block screenshots; when `ios.preventScreenshots` is enabled, the plugin uses an unofficial secure text field technique that may stop working in future iOS versions.
- The `screenshotTaken` event only fires after the screenshot has been taken and therefore cannot be prevented.
