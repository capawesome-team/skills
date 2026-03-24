---
name: capacitor-vue
description: "Guides the agent through Vue-specific patterns for Capacitor app development. Covers Vue 3 Composition API with Capacitor plugins, custom composables for native features, reactive plugin state, lifecycle hook patterns, Vue Router deep link integration, platform detection, PWA Elements setup, Quasar Framework integration, and Nuxt integration. Do not use for creating a new Capacitor app from scratch, upgrading Capacitor versions, installing specific plugins, Ionic Framework with Vue setup, or non-Vue frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-vue
---

# Capacitor with Vue

Vue-specific patterns and best practices for Capacitor app development — Composition API, composables, reactivity, lifecycle hooks, Vue Router integration, and framework-specific guidance for Quasar and Nuxt.

## Prerequisites

1. **Capacitor 6, 7, or 8** app with Vue 3.
2. Node.js and npm installed.
3. For iOS: Xcode installed.
4. For Android: Android Studio installed.

## Agent Behavior

- **Auto-detect before asking.** Check the project for `vite.config.ts`, `vite.config.js`, `quasar.config.js`, `quasar.config.ts`, `nuxt.config.ts`, `package.json`, `capacitor.config.ts` or `capacitor.config.json`, and existing directory structure. Only ask the user when something cannot be detected.
- **Guide step-by-step.** Walk the user through the process one step at a time.
- **Detect the meta-framework.** Determine whether the project uses plain Vue (Vite), Quasar, or Nuxt, and adapt instructions accordingly.

## Procedures

### Step 1: Analyze the Project

Auto-detect the following by reading project files:

1. **Vue version**: Read `vue` version from `package.json`.
2. **Capacitor version**: Read `@capacitor/core` version from `package.json`. If not present, Capacitor has not been added yet — proceed to Step 2.
3. **Meta-framework**: Detect the framework by checking for these files in order:
   - `quasar.config.js` or `quasar.config.ts` — **Quasar** project. Proceed to `references/quasar.md`.
   - `nuxt.config.ts` or `nuxt.config.js` — **Nuxt** project. Proceed to `references/nuxt.md`.
   - `vite.config.ts` or `vite.config.js` — **Plain Vue (Vite)** project. Continue with the steps below.
4. **Platforms**: Check which directories exist (`android/`, `ios/`).
5. **Capacitor config format**: Check for `capacitor.config.ts` (TypeScript) or `capacitor.config.json` (JSON).
6. **Build output directory**: Read `build.outDir` from `vite.config.ts` or `vite.config.js`. The default is `dist`.

### Step 2: Add Capacitor to a Vue Project

Skip if `@capacitor/core` is already in `package.json`. Skip if the project uses Quasar (Quasar has its own Capacitor integration — see `references/quasar.md`).

1. Install Capacitor core and CLI:

   ```bash
   npm install @capacitor/core
   npm install -D @capacitor/cli
   ```

2. Initialize Capacitor:

   ```bash
   npx cap init
   ```

   When prompted, set the **web directory** to the Vue build output path detected in Step 1. For Vite-based Vue projects, this is typically `dist`.

3. Verify the `webDir` value in the generated `capacitor.config.ts` or `capacitor.config.json` matches the Vue build output path. If incorrect, update it:

   **`capacitor.config.ts`:**
   ```typescript
   import type { CapacitorConfig } from '@capacitor/cli';

   const config: CapacitorConfig = {
     appId: 'com.example.app',
     appName: 'my-app',
     webDir: 'dist',
   };

   export default config;
   ```

   **`capacitor.config.json`:**
   ```json
   {
     "appId": "com.example.app",
     "appName": "my-app",
     "webDir": "dist"
   }
   ```

4. Build the Vue app and add platforms:

   ```bash
   npm run build
   npm install @capacitor/android @capacitor/ios
   npx cap add android
   npx cap add ios
   npx cap sync
   ```

### Step 3: Project Structure

A Capacitor Vue project (Vite-based) has this structure:

```
my-app/
├── android/                  # Android native project
├── ios/                      # iOS native project
├── public/
├── src/
│   ├── assets/
│   ├── components/
│   ├── composables/          # Custom composables for Capacitor plugins
│   ├── router/
│   │   └── index.ts          # Vue Router configuration
│   ├── views/
│   ├── App.vue
│   └── main.ts
├── capacitor.config.ts       # or capacitor.config.json
├── index.html
├── package.json
├── tsconfig.json
└── vite.config.ts
```

Key points:
- The `android/` and `ios/` directories contain native projects and should be committed to version control.
- The `src/` directory contains the Vue app, which is the web layer of the Capacitor app.
- Place custom composables that wrap Capacitor plugins in `src/composables/`.
- Capacitor plugins are called from Vue components or composables inside `src/`.

### Step 4: Using Capacitor Plugins in Vue

Capacitor plugins are plain TypeScript APIs. Import and call them directly in Vue components using the Composition API.

#### Direct Usage in a Component

```vue
<script setup lang="ts">
import { ref } from 'vue';
import { Geolocation } from '@capacitor/geolocation';

const latitude = ref<number | null>(null);
const longitude = ref<number | null>(null);

async function getCurrentPosition() {
  const position = await Geolocation.getCurrentPosition();
  latitude.value = position.coords.latitude;
  longitude.value = position.coords.longitude;
}
</script>

<template>
  <div>
    <p>Latitude: {{ latitude }}</p>
    <p>Longitude: {{ longitude }}</p>
    <button @click="getCurrentPosition">Get Location</button>
  </div>
</template>
```

#### Wrapping Plugins in Composables (Recommended)

Wrapping Capacitor plugins in composables provides reusability, encapsulated reactive state, and automatic cleanup:

```typescript
// src/composables/useCamera.ts
import { ref } from 'vue';
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';
import type { Photo } from '@capacitor/camera';

export function useCamera() {
  const photo = ref<Photo | null>(null);
  const error = ref<string | null>(null);

  async function takePhoto(): Promise<void> {
    try {
      error.value = null;
      photo.value = await Camera.getPhoto({
        quality: 90,
        allowEditing: false,
        resultType: CameraResultType.Uri,
        source: CameraSource.Camera,
      });
    } catch (e) {
      error.value = e instanceof Error ? e.message : String(e);
    }
  }

  async function pickFromGallery(): Promise<void> {
    try {
      error.value = null;
      photo.value = await Camera.getPhoto({
        quality: 90,
        allowEditing: false,
        resultType: CameraResultType.Uri,
        source: CameraSource.Photos,
      });
    } catch (e) {
      error.value = e instanceof Error ? e.message : String(e);
    }
  }

  return {
    photo,
    error,
    takePhoto,
    pickFromGallery,
  };
}
```

Use the composable in a component:

```vue
<script setup lang="ts">
import { useCamera } from '@/composables/useCamera';

const { photo, error, takePhoto } = useCamera();
</script>

<template>
  <div>
    <button @click="takePhoto">Take Photo</button>
    <p v-if="error">Error: {{ error }}</p>
    <img v-if="photo?.webPath" :src="photo.webPath" alt="Captured photo" />
  </div>
</template>
```

### Step 5: Plugin Event Listeners with Lifecycle Hooks

Capacitor plugin event listeners must be registered in `onMounted` and removed in `onUnmounted` to prevent memory leaks. Vue's reactivity system picks up `ref` changes automatically, so there is no NgZone-equivalent issue — but cleanup is still critical.

#### Composable with Automatic Cleanup

```typescript
// src/composables/useNetwork.ts
import { ref, onMounted, onUnmounted } from 'vue';
import { Network } from '@capacitor/network';
import type { ConnectionStatus } from '@capacitor/network';
import type { PluginListenerHandle } from '@capacitor/core';

export function useNetwork() {
  const status = ref<ConnectionStatus | null>(null);
  let listenerHandle: PluginListenerHandle | null = null;

  onMounted(async () => {
    status.value = await Network.getStatus();
    listenerHandle = await Network.addListener('networkStatusChange', (newStatus) => {
      status.value = newStatus;
    });
  });

  onUnmounted(async () => {
    await listenerHandle?.remove();
  });

  return {
    status,
  };
}
```

Usage in a component:

```vue
<script setup lang="ts">
import { useNetwork } from '@/composables/useNetwork';

const { status } = useNetwork();
</script>

<template>
  <p v-if="status">Network: {{ status.connected ? 'Online' : 'Offline' }}</p>
</template>
```

#### App-Wide Listeners via App.vue

For listeners that should persist for the entire app lifecycle (e.g., app state changes), register them in `App.vue`:

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue';
import { App } from '@capacitor/app';
import type { PluginListenerHandle } from '@capacitor/core';
import { RouterView } from 'vue-router';

let appStateListener: PluginListenerHandle | null = null;

onMounted(async () => {
  appStateListener = await App.addListener('appStateChange', (state) => {
    console.log('App state changed. Is active:', state.isActive);
  });
});

onUnmounted(async () => {
  await appStateListener?.remove();
});
</script>

<template>
  <RouterView />
</template>
```

### Step 6: Platform Detection

Use `Capacitor.isNativePlatform()` and `Capacitor.getPlatform()` to conditionally run native-only code. Wrap this in a composable for reuse:

```typescript
// src/composables/usePlatform.ts
import { Capacitor } from '@capacitor/core';

export function usePlatform() {
  const platform = Capacitor.getPlatform() as 'web' | 'ios' | 'android';
  const isNative = Capacitor.isNativePlatform();
  const isIos = platform === 'ios';
  const isAndroid = platform === 'android';
  const isWeb = platform === 'web';

  return {
    platform,
    isNative,
    isIos,
    isAndroid,
    isWeb,
  };
}
```

Use it in components to show/hide native-only features:

```vue
<script setup lang="ts">
import { usePlatform } from '@/composables/usePlatform';

const { isNative } = usePlatform();
</script>

<template>
  <button v-if="isNative" @click="openNativeSettings()">Open Device Settings</button>
</template>
```

### Step 7: Deep Link Routing with Vue Router

Handle deep links by mapping Capacitor's `App.addListener('appUrlOpen', ...)` event to Vue Router navigation. Set this up in `App.vue` or a dedicated composable:

```typescript
// src/composables/useDeepLinks.ts
import { onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import { App } from '@capacitor/app';
import type { PluginListenerHandle } from '@capacitor/core';

export function useDeepLinks() {
  const router = useRouter();
  let listenerHandle: PluginListenerHandle | null = null;

  onMounted(async () => {
    listenerHandle = await App.addListener('appUrlOpen', (event) => {
      const url = new URL(event.url);
      const path = url.pathname;

      // Navigate to the route matching the deep link path.
      // Adjust the path parsing logic to match the app's URL scheme.
      if (path) {
        router.push(path);
      }
    });
  });

  onUnmounted(async () => {
    await listenerHandle?.remove();
  });
}
```

Use the composable in `App.vue`:

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { useDeepLinks } from '@/composables/useDeepLinks';

useDeepLinks();
</script>

<template>
  <RouterView />
</template>
```

### Step 8: Back Button Handling (Android)

Handle the Android hardware back button using `App.addListener('backButton', ...)` combined with Vue Router:

```typescript
// src/composables/useBackButton.ts
import { onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import { App } from '@capacitor/app';
import { Capacitor } from '@capacitor/core';
import type { PluginListenerHandle } from '@capacitor/core';

export function useBackButton() {
  const router = useRouter();
  let listenerHandle: PluginListenerHandle | null = null;

  onMounted(async () => {
    if (Capacitor.getPlatform() !== 'android') {
      return;
    }

    listenerHandle = await App.addListener('backButton', ({ canGoBack }) => {
      if (canGoBack) {
        router.back();
      } else {
        App.exitApp();
      }
    });
  });

  onUnmounted(async () => {
    await listenerHandle?.remove();
  });
}
```

Use the composable in `App.vue`:

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { useBackButton } from '@/composables/useBackButton';

useBackButton();
</script>

<template>
  <RouterView />
</template>
```

### Step 9: PWA Elements Setup

Some Capacitor plugins (e.g., Camera, Toast) require `@ionic/pwa-elements` for web fallback UI. If the project uses any of these plugins and targets the web:

1. Install PWA Elements:

   ```bash
   npm install @ionic/pwa-elements
   ```

2. Register the custom elements in `src/main.ts` **before** `createApp()`:

   ```typescript
   import { createApp } from 'vue';
   import { defineCustomElements } from '@ionic/pwa-elements/loader';
   import App from './App.vue';
   import router from './router';

   defineCustomElements(window);

   const app = createApp(App);
   app.use(router);
   app.mount('#app');
   ```

### Step 10: Build and Sync Workflow

After making changes to the Vue app, build and sync to native platforms:

```bash
npm run build
npx cap sync
```

To run on a device or emulator:

```bash
npx cap run android
npx cap run ios
```

To open the native IDE for advanced configuration or debugging:

```bash
npx cap open android
npx cap open ios
```

For live reload during development:

```bash
npx cap run android --livereload --external
npx cap run ios --livereload --external
```

This starts the Vite dev server internally and configures the native app to load from the development server.

## Error Handling

- **`webDir` mismatch**: If `npx cap sync` copies the wrong files, verify that `webDir` in `capacitor.config.ts` or `capacitor.config.json` matches the Vue build output path. For Vite-based Vue projects, the default output directory is `dist`.
- **Plugin not found at runtime**: Run `npx cap sync` after installing any new plugin. Verify the plugin appears in `package.json` dependencies.
- **Memory leaks from listeners**: Always remove plugin listeners in `onUnmounted`. Store the `PluginListenerHandle` returned by `addListener` and call `handle.remove()` on unmount.
- **Deep links not working**: Verify the app URL scheme or universal links are configured in the native projects (`android/app/src/main/AndroidManifest.xml` for Android, `ios/App/App/Info.plist` and associated domain entitlement for iOS). Verify `useDeepLinks()` is called in `App.vue`.
- **Back button closes app unexpectedly**: Ensure the back button listener checks `canGoBack` before calling `App.exitApp()`. Only exit when there is no navigation history.
- **PWA Elements not rendering on web**: Verify `defineCustomElements(window)` is called in `src/main.ts` before `createApp()`. Verify `@ionic/pwa-elements` is installed.
- **Quasar project — Capacitor not detected**: Use `quasar mode add capacitor` instead of manually installing Capacitor. See `references/quasar.md`.
- **Nuxt project — build output not compatible**: Nuxt's default SSR output is not compatible with Capacitor. The project must use `ssr: false` in `nuxt.config.ts` to generate a static SPA. See `references/nuxt.md`.

## Related Skills

- **`capacitor-app-creation`** — Create a new Capacitor app from scratch.
- **`capacitor-app-development`** — General Capacitor development guidance not specific to Vue.
- **`capacitor-plugins`** — Install and configure Capacitor plugins from official and community sources.
- **`ionic-vue`** — Ionic Framework with Vue (UI components, navigation, theming on top of Capacitor).
- **`capacitor-app-upgrades`** — Upgrade a Capacitor app to a newer major version.
