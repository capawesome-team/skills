# Accessibility Preferences

Capacitor plugin for reading the user's system accessibility preferences (font scale, reduce motion, contrast, inverted colors).

**Package:** `@capawesome/capacitor-accessibility-preferences`

**Platforms:** Android, iOS, Web (partial)

## Installation

```bash
npm install @capawesome/capacitor-accessibility-preferences
npx cap sync
```

No additional permissions or configuration are required.

## Usage

```typescript
import { AccessibilityPreferences } from '@capawesome/capacitor-accessibility-preferences';

const getPreferences = async () => {
  const preferences = await AccessibilityPreferences.getPreferences();
  console.log(preferences.fontScale);
  console.log(preferences.isReduceMotionEnabled);
  console.log(preferences.isBoldTextEnabled);
  console.log(preferences.isInvertColorsEnabled);
  console.log(preferences.isHighContrastEnabled);
  console.log(preferences.isReduceTransparencyEnabled);
};
```

## Notes

- Fields that the current platform cannot provide are set to `null`.
- `fontScale`: available on Android and iOS; always `1.0` on Web (not exposed to web content). On iOS it is derived from the preferred content size category.
- `isReduceMotionEnabled`: available on all platforms (`boolean`).
- `isHighContrastEnabled`: available on all platforms; `null` where it cannot be read.
- `isBoldTextEnabled`: available on iOS and Android API 31+; `null` on Web and older Android.
- `isInvertColorsEnabled`: available on Android and iOS; `null` on Web.
- `isReduceTransparencyEnabled`: iOS only; `null` on Android and Web.
- CSS media queries (`prefers-reduced-motion`, `prefers-contrast`) already cover reduce-motion and contrast inside the WebView. This plugin's value is the fields CSS cannot see (`fontScale`, bold text, inverted colors) plus a single cross-platform API.
- Screen reader state (VoiceOver/TalkBack) is out of scope; use `@capacitor/screen-reader` for that.
