# Nuxt with Capacitor

Nuxt can be used with Capacitor, but requires specific configuration since Capacitor needs a static SPA build output, not Nuxt's default SSR output.

## Critical Requirement: Disable SSR

Capacitor loads a local `index.html` file on the device. Nuxt's default SSR mode generates server-rendered output that cannot run inside Capacitor's WebView. The project **must** be configured for static SPA generation.

In `nuxt.config.ts`, set:

```typescript
export default defineNuxtConfig({
  ssr: false,
});
```

This configures Nuxt to generate a client-side SPA in the `.output/public/` directory.

## Project Structure

```
my-nuxt-app/
├── android/                     # Android native project
├── ios/                         # iOS native project
├── .output/
│   └── public/                  # SPA build output (webDir target)
├── components/
├── composables/                 # Custom composables for Capacitor plugins
├── layouts/
├── pages/
├── plugins/                     # Nuxt plugins for initialization
├── app.vue
├── capacitor.config.ts          # or capacitor.config.json
├── nuxt.config.ts
└── package.json
```

## Adding Capacitor

1. Install Capacitor:

   ```bash
   npm install @capacitor/core
   npm install -D @capacitor/cli
   ```

2. Initialize Capacitor with the correct web directory:

   ```bash
   npx cap init
   ```

   Set `webDir` to `.output/public` when prompted.

3. Verify the Capacitor config:

   **`capacitor.config.ts`:**
   ```typescript
   import type { CapacitorConfig } from '@capacitor/cli';

   const config: CapacitorConfig = {
     appId: 'com.example.app',
     appName: 'my-nuxt-app',
     webDir: '.output/public',
   };

   export default config;
   ```

   **`capacitor.config.json`:**
   ```json
   {
     "appId": "com.example.app",
     "appName": "my-nuxt-app",
     "webDir": ".output/public"
   }
   ```

4. Build and add platforms:

   ```bash
   npx nuxi generate
   npm install @capacitor/android @capacitor/ios
   npx cap add android
   npx cap add ios
   npx cap sync
   ```

   Use `npx nuxi generate` (not `npx nuxi build`) to produce the static SPA output.

## Build and Sync Workflow

```bash
npx nuxi generate
npx cap sync
```

To run on a device or emulator:

```bash
npx cap run android
npx cap run ios
```

## Nuxt Plugins for Capacitor Initialization

Use Nuxt plugins for app-wide Capacitor initialization instead of modifying `app.vue` directly:

```typescript
// plugins/capacitor.client.ts
import { App } from '@capacitor/app';
import { Capacitor } from '@capacitor/core';

export default defineNuxtPlugin((nuxtApp) => {
  const router = useRouter();

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

The `.client.ts` suffix ensures this plugin only runs on the client side.

## PWA Elements in Nuxt

If the project needs `@ionic/pwa-elements`:

1. Install the package:

   ```bash
   npm install @ionic/pwa-elements
   ```

2. Create a client-side Nuxt plugin:

   ```typescript
   // plugins/pwa-elements.client.ts
   import { defineCustomElements } from '@ionic/pwa-elements/loader';

   export default defineNuxtPlugin(() => {
     defineCustomElements(window);
   });
   ```

## Composables in Nuxt

All composable patterns from the main skill (e.g., `useCamera`, `useNetwork`, `usePlatform`) work identically in Nuxt. Place them in the `composables/` directory — Nuxt auto-imports composables from this directory.

One difference: in Nuxt, composables in the `composables/` directory are auto-imported. There is no need to import them manually in components:

```vue
<script setup lang="ts">
// No import needed — Nuxt auto-imports from composables/
const { status } = useNetwork();
</script>

<template>
  <p v-if="status">Network: {{ status.connected ? 'Online' : 'Offline' }}</p>
</template>
```

## Nuxt-Specific Caveats

- **Always use `ssr: false`** in `nuxt.config.ts`. Without this, the build output is not compatible with Capacitor.
- **Use `npx nuxi generate`** for builds, not `npx nuxi build`. The `generate` command produces static files; `build` produces a server bundle.
- **The `webDir` is `.output/public`**, not `dist`. This is Nuxt's default static output directory when using `ssr: false` and `npx nuxi generate`.
- **Client-only plugins**: Use the `.client.ts` suffix for all Capacitor-related Nuxt plugins to prevent server-side execution during development.
- **`process.client` guard**: If Capacitor code runs outside a `.client.ts` plugin, wrap it in a `process.client` check to prevent SSR errors during development with `ssr: true` accidentally enabled.
