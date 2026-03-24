# Quasar Framework with Capacitor

Quasar has built-in Capacitor integration managed through the Quasar CLI. Do **not** install or configure Capacitor manually in a Quasar project — use the Quasar CLI commands instead.

## Project Structure

Quasar places the Capacitor project in `src-capacitor/` instead of the root:

```
my-quasar-app/
├── src/
│   ├── boot/                    # Boot files (initialization logic)
│   ├── components/
│   ├── composables/             # Custom composables for Capacitor plugins
│   ├── layouts/
│   ├── pages/
│   ├── router/
│   │   ├── index.ts
│   │   └── routes.ts
│   ├── App.vue
│   └── ...
├── src-capacitor/               # Capacitor project (managed by Quasar CLI)
│   ├── android/
│   ├── ios/
│   └── capacitor.config.json
├── quasar.config.js             # or quasar.config.ts
└── package.json
```

Key differences from plain Vue:
- The Capacitor native projects live under `src-capacitor/android/` and `src-capacitor/ios/`.
- The Capacitor config file is at `src-capacitor/capacitor.config.json`.
- Build and dev commands use the Quasar CLI, not `npx cap` directly.

## Adding Capacitor Mode

If the project does not yet have `src-capacitor/`:

```bash
quasar mode add capacitor
```

This creates the `src-capacitor/` directory and installs the required dependencies.

## Development

Start the development server with hot reload on a device or emulator:

```bash
quasar dev -m capacitor -T android
quasar dev -m capacitor -T ios
```

This automatically opens the native IDE (Android Studio or Xcode) where the app can be deployed to a device or emulator.

## Building for Production

```bash
quasar build -m capacitor -T android
quasar build -m capacitor -T ios
```

## Using Capacitor Plugins

Capacitor plugins work the same way as in plain Vue — import them and use them with the Composition API. The only difference is initialization patterns.

### Boot Files for App-Wide Plugin Initialization

Quasar uses **boot files** for initialization logic that must run before the app starts. Use boot files for app-wide Capacitor plugin setup instead of `App.vue`.

Create a boot file for Capacitor plugin initialization:

```typescript
// src/boot/capacitor.ts
import { boot } from 'quasar/wrappers';
import { App } from '@capacitor/app';
import { Capacitor } from '@capacitor/core';

export default boot(({ router }) => {
  // Deep link handling
  App.addListener('appUrlOpen', (event) => {
    const url = new URL(event.url);
    const path = url.pathname;
    if (path) {
      router.push(path);
    }
  });

  // Android back button handling
  if (Capacitor.getPlatform() === 'android') {
    App.addListener('backButton', ({ canGoBack }) => {
      if (canGoBack) {
        router.back();
      } else {
        App.exitApp();
      }
    });
  }
});
```

Register the boot file in `quasar.config.js` or `quasar.config.ts`:

```diff
- boot: [],
+ boot: ['capacitor'],
```

### PWA Elements in Quasar

If the project needs `@ionic/pwa-elements` for web fallback UI:

1. Install the package:

   ```bash
   npm install @ionic/pwa-elements
   ```

2. Create a boot file:

   ```typescript
   // src/boot/pwa-elements.ts
   import { boot } from 'quasar/wrappers';
   import { defineCustomElements } from '@ionic/pwa-elements/loader';

   export default boot(() => {
     defineCustomElements(window);
   });
   ```

3. Register the boot file in `quasar.config.js` or `quasar.config.ts`:

   ```diff
   - boot: ['capacitor'],
   + boot: ['capacitor', 'pwa-elements'],
   ```

## Quasar-Specific Configuration

### Back Button (Quasar Built-In)

Quasar provides built-in back button handling via `quasar.config.js` or `quasar.config.ts`:

```javascript
// quasar.config.js
framework: {
  config: {
    capacitor: {
      backButton: true,       // Enable Quasar's back button handler
      backButtonExit: true,   // Exit app when no history (can also be an array of routes: ['/login', '/home'])
    }
  }
}
```

When using Quasar's built-in back button handling, do **not** also register a custom `backButton` listener — they will conflict.

### Splash Screen

Quasar auto-hides the splash screen by default. To control this manually:

```javascript
// quasar.config.js
capacitor: {
  hideSplashscreen: false, // Disable auto-hide to control it manually
}
```

Then hide it manually in a boot file or component:

```typescript
import { SplashScreen } from '@capacitor/splash-screen';

await SplashScreen.hide();
```

## Composables in Quasar

All composable patterns from the main skill (e.g., `useCamera`, `useNetwork`, `usePlatform`) work identically in Quasar. Place them in `src/composables/`.

## Running Native Commands

To run Capacitor CLI commands directly (e.g., `npx cap sync`), execute them from inside the `src-capacitor/` directory:

```bash
cd src-capacitor && npx cap sync
```

Or use the Quasar CLI equivalents when available.
