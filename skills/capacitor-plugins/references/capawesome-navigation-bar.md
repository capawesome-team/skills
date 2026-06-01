# Navigation Bar

Customize the appearance and visibility of the Android system navigation bar (background color, button style, divider color).

**Package:** `@capawesome/capacitor-navigation-bar`
**Platforms:** Android
**Capawesome Insiders:** No

> iOS does not expose a customizable system navigation bar, so all methods are Android-only.

## Installation

```bash
npm install @capawesome/capacitor-navigation-bar
npx cap sync
```

## Configuration

### capacitor.config (optional)

Configure the initial navigation bar appearance via `capacitor.config.ts`:

```typescript
/// <reference types="@capawesome/capacitor-navigation-bar" />

import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  plugins: {
    NavigationBar: {
      color: '#ffffff',
      dividerColor: '#d9d9d9',
      style: 'LIGHT',
    },
  },
};

export default config;
```

- `color`: hex color or `'transparent'`.
- `dividerColor`: hex color (Android 9+ only).
- `style`: `'LIGHT'`, `'DARK'`, or `'DEFAULT'`.

## Usage

```typescript
import { NavigationBar, Style } from '@capawesome/capacitor-navigation-bar';

await NavigationBar.setColor({ color: '#ffffff' });
await NavigationBar.setStyle({ style: Style.Light });

const { color } = await NavigationBar.getColor();
const { style } = await NavigationBar.getStyle();

await NavigationBar.hide();
await NavigationBar.show();
```

## Enums

- **`Style`**: `Light`, `Dark`, `Default`.

## Notes

- All methods are Android-only and throw an error on iOS.
- On Android 15+, edge-to-edge display is enforced. To color the navigation bar on Android 15+, use the `@capawesome/capacitor-android-edge-to-edge-support` plugin (see `references/capawesome-android-edge-to-edge-support.md`).
- The divider color is only configurable on Android 9 (API 28) and higher.
