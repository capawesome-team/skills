# Android Edge-to-Edge Support

Configure Android apps to draw content behind the status bar and navigation bar for immersive, full-screen layouts.

## Overview

Edge-to-edge means the app content extends behind the system bars (status bar and navigation bar). The app is responsible for handling insets to prevent content from being obscured.

Starting with **Android 15 (API 35)**, edge-to-edge is enforced by the system. Apps targeting API 35+ are edge-to-edge by default.

## Capacitor 8+: Built-in Support

Capacitor 8 includes a built-in `SystemBars` plugin that handles edge-to-edge automatically. It manages status and navigation bar appearance and applies insets to the WebView.

The `SystemBars` plugin provides a public API for granular control:

```typescript
import { SystemBars } from '@capacitor/core';

// Set status bar style
await SystemBars.setStyle({ statusBar: 'DARK', navigationBar: 'LIGHT' });

// Hide system bars
await SystemBars.setVisible({ statusBar: false, navigationBar: false });
```

For apps using the `@capacitor/status-bar` plugin (Capacitor 7 and earlier), the two can be used together. Capacitor automatically applies the correct behavior based on the device's Android version.

### Disabling Built-in Inset Handling

If using a third-party edge-to-edge plugin (e.g., `@capawesome/capacitor-android-edge-to-edge-support` or `@capacitor-community/safe-area`), disable the built-in insets handling in `capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  plugins: {
    SystemBars: {
      insetsHandling: 'disable',
    },
  },
};
```

## Capacitor 7 and Earlier: Plugin Options

For Capacitor 7 and earlier, use one of these plugins:

### Option A: `@capawesome/capacitor-android-edge-to-edge-support`

This plugin enables edge-to-edge mode on Android while preserving traditional inset handling by applying proper insets to the WebView.

```bash
npm install @capawesome/capacitor-android-edge-to-edge-support
npx cap sync
```

Configuration in `capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  plugins: {
    EdgeToEdge: {
      backgroundColor: '#ffffff',
      statusBarColor: '#ffffff',
      navigationBarColor: '#ffffff',
    },
  },
};
```

API:

```typescript
import { EdgeToEdge } from '@capawesome/capacitor-android-edge-to-edge-support';

await EdgeToEdge.enable();
await EdgeToEdge.disable();
const insets = await EdgeToEdge.getInsets();
// insets = { top: 24, bottom: 48, left: 0, right: 0 }
```

For full plugin setup, use the `capacitor-plugins` skill.

### Option B: `@capacitor-community/safe-area`

This plugin handles both edge-to-edge and safe area insets. See `references/safe-area.md` for details.

## CSS Inset Handling

When edge-to-edge is enabled, use CSS environment variables to prevent content from being obscured by system bars:

```css
.content {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
```

For the `viewport-fit=cover` meta tag to work correctly, add it to `index.html`:

```html
<meta name="viewport" content="viewport-fit=cover, width=device-width, initial-scale=1.0">
```

**Note on Android WebView bug**: Chromium WebView versions below 140 have a bug where `env(safe-area-inset-*)` values may report incorrect values. The `@capacitor-community/safe-area` plugin includes a workaround for this issue.
