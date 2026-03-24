# Splash Screens and App Icons

Generate and configure splash screens and app icons for Capacitor apps using `@capacitor/assets`.

## Installation

```bash
npm install @capacitor/assets --save-dev
```

## Source Asset Requirements

Create an `assets/` directory in the project root with the following files:

| File | Minimum Size | Purpose |
| ---- | ------------ | ------- |
| `icon-only.png` | 1024x1024 px | App icon without background (used for iOS) |
| `icon-foreground.png` | 1024x1024 px | Adaptive icon foreground layer (Android) |
| `icon-background.png` | 1024x1024 px | Adaptive icon background layer (Android) |
| `splash.png` | 2732x2732 px | Light mode splash screen |
| `splash-dark.png` | 2732x2732 px | Dark mode splash screen |

All files must be PNG or JPG format.

## Generate Assets

Run the generation command to create platform-specific assets:

```bash
npx capacitor-assets generate
```

This generates all required icon sizes and splash screen variants for each platform.

To generate for a specific platform only:

```bash
npx capacitor-assets generate --ios
npx capacitor-assets generate --android
npx capacitor-assets generate --pwa
```

## Android-Specific Notes

### Android 12+ Splash Screen

Android 12 and newer versions use a redesigned splash screen system that displays a smaller icon with a colored background, rather than a full-screen image. The splash screen is managed by the Android system and cannot be fully customized.

The `@capacitor/assets` tool automatically generates the correct adaptive icon and splash screen resources for Android 12+.

### Adaptive Icons

Android 8.0+ uses adaptive icons with separate foreground and background layers. The `icon-foreground.png` and `icon-background.png` files are used to generate these. Keep the main icon content within the center 66% of the `icon-foreground.png` to ensure it is not clipped by different device masks.

## iOS-Specific Notes

iOS uses a single icon at 1024x1024 and scales it for all required sizes. The `icon-only.png` is used as the source.

iOS splash screens use a Storyboard (`LaunchScreen.storyboard`) rather than static images. The `@capacitor/assets` tool updates this file automatically.

## Splash Screen Plugin

To control splash screen behavior at runtime (show duration, auto-hide, animations), use the `@capacitor/splash-screen` plugin. Configure it in `capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  plugins: {
    SplashScreen: {
      launchShowDuration: 2000,
      launchAutoHide: true,
      launchFadeOutDuration: 500,
      backgroundColor: '#ffffff',
      showSpinner: false,
    },
  },
};
```

To manually control the splash screen:

```typescript
import { SplashScreen } from '@capacitor/splash-screen';

// Hide the splash screen
await SplashScreen.hide();

// Show the splash screen
await SplashScreen.show({ autoHide: false });
```

For full plugin API, use the `capacitor-plugins` skill.
