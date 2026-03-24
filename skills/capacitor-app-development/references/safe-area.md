# Android Safe Area Handling

Handle safe area insets on Android to prevent content from being obscured by system bars, notches, and rounded corners.

## Overview

On iOS and modern web browsers, `env(safe-area-inset-*)` CSS variables work correctly out of the box. On Android, the WebView's Chromium engine may report incorrect inset values (especially in versions below 140). This reference covers Android-specific solutions.

## Prerequisites

Add `viewport-fit=cover` to the viewport meta tag in `index.html`:

```html
<meta name="viewport" content="viewport-fit=cover, width=device-width, initial-scale=1.0">
```

## Option A: `@capacitor-community/safe-area` Plugin

This plugin detects the Chromium WebView version and applies a workaround automatically for versions with the inset bug.

### Installation

For Capacitor 8:

```bash
npm install @capacitor-community/safe-area
npx cap sync
```

For Capacitor 7:

```bash
npm install @capacitor-community/safe-area@^7.0.0
npx cap sync
```

### Android Configuration

Enable edge-to-edge in `android/app/src/main/java/.../MainActivity.java` (or `.kt`):

```diff
+ import androidx.activity.EdgeToEdge;

  public class MainActivity extends BridgeActivity {
+     @Override
+     protected void onCreate(Bundle savedInstanceState) {
+         EdgeToEdge.enable(this);
+         super.onCreate(savedInstanceState);
+     }
  }
```

Disable Capacitor's built-in insets handling in `capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  plugins: {
    SystemBars: {
      insetsHandling: 'disable',
    },
  },
};
```

### Plugin Configuration

```typescript
const config: CapacitorConfig = {
  plugins: {
    SafeArea: {
      detectViewportFitCoverChanges: true,
      initialViewportFitCover: true,
    },
  },
};
```

### Conflicts

When using `@capacitor-community/safe-area`:
- Remove `@capacitor/status-bar` if installed — the safe-area plugin manages system bar styling.
- Set `resizeOnFullScreen: false` for `@capacitor/keyboard` to avoid WebView resizing conflicts.
- Do not install `@capawesome/capacitor-android-edge-to-edge-support` alongside this plugin.

### API

```typescript
import { SafeArea } from '@capacitor-community/safe-area';

// Get current insets
const insets = await SafeArea.getInsets();
// { top: 24, bottom: 48, left: 0, right: 0 }

// Style system bars
await SafeArea.setSystemBarsStyle({ style: 'DARK' });

// Hide/show system bars
await SafeArea.hideSystemBars({ bars: 'BOTH' });
await SafeArea.showSystemBars({ bars: 'BOTH' });
```

For full plugin setup, use the `capacitor-plugins` skill.

## Option B: `@capawesome/capacitor-android-edge-to-edge-support`

See `references/edge-to-edge.md` for details on this plugin, which takes a different approach by applying insets to the WebView itself rather than exposing them via CSS.

## CSS Usage

After installing a safe area plugin, use standard CSS environment variables:

```css
:root {
  --safe-area-top: env(safe-area-inset-top);
  --safe-area-bottom: env(safe-area-inset-bottom);
  --safe-area-left: env(safe-area-inset-left);
  --safe-area-right: env(safe-area-inset-right);
}

header {
  padding-top: var(--safe-area-top);
}

footer {
  padding-bottom: var(--safe-area-bottom);
}

.tab-bar {
  padding-bottom: var(--safe-area-bottom);
}
```
