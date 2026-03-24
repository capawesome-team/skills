# Ionic Vue Composables and Utility Functions

Ionic Vue provides built-in composable functions that wrap common Ionic and Capacitor functionality for use in Vue's Composition API.

## useIonRouter

Returns the Ionic router instance with methods for navigation and page transitions.

```vue
<script setup lang="ts">
import { useIonRouter } from '@ionic/vue';

const ionRouter = useIonRouter();

const goToDetail = (id: string) => {
  ionRouter.push(`/detail/${id}`);
};

const goBack = () => {
  if (ionRouter.canGoBack()) {
    ionRouter.back();
  }
};
</script>
```

Methods:
- `push(path, animation?)` — Navigate forward.
- `replace(path, animation?)` — Replace current route.
- `back(animation?)` — Go back one page.
- `forward(animation?)` — Go forward one page.
- `navigate(path, direction, action, animation?)` — Full navigation control.
- `canGoBack()` — Check if back navigation is possible.

This composable can be used alongside Vue Router's `useRouter()`. Use `useIonRouter` when Ionic-specific transition control is needed. Use `useRouter` for standard Vue Router features (route guards, query params, named routes).

**Constraint:** Must be called inside `setup()` or `<script setup>`.

## useBackButton

Registers a handler for the Android hardware back button. Accepts a priority number to control execution order when multiple handlers exist.

```vue
<script setup lang="ts">
import { useBackButton, useIonRouter } from '@ionic/vue';

const ionRouter = useIonRouter();

useBackButton(10, (processNextHandler) => {
  if (ionRouter.canGoBack()) {
    ionRouter.back();
  } else {
    processNextHandler();
  }
});
</script>
```

- The first argument is the **priority** (higher = runs first).
- The callback receives `processNextHandler`, which delegates to the next lower-priority handler.
- Only fires in Capacitor or Cordova environments.
- Returns an object with an `unregister()` method to remove the handler.

## useKeyboard

Provides reactive state about the on-screen keyboard visibility and height.

```vue
<script setup lang="ts">
import { useKeyboard } from '@ionic/vue';

const { isOpen, keyboardHeight } = useKeyboard();
</script>

<template>
  <div>
    <p v-if="isOpen.value">Keyboard is open ({{ keyboardHeight.value }}px)</p>
  </div>
</template>
```

- `isOpen` — A Vue `ref<boolean>` indicating whether the keyboard is visible.
- `keyboardHeight` — A Vue `ref<number>` with the keyboard height in pixels.
- Returns an `unregister()` method to stop listening.

Only provides meaningful values on native platforms (iOS/Android) via Capacitor or Cordova.

## Ionic Lifecycle Hooks (Composables)

Four composable hooks for Ionic page lifecycle events. See `references/lifecycle.md` for full details.

```typescript
import {
  onIonViewWillEnter,
  onIonViewDidEnter,
  onIonViewWillLeave,
  onIonViewDidLeave,
} from '@ionic/vue';

onIonViewWillEnter(() => { /* page about to show */ });
onIonViewDidEnter(() => { /* page fully visible */ });
onIonViewWillLeave(() => { /* page about to hide */ });
onIonViewDidLeave(() => { /* page fully hidden */ });
```

## isPlatform

A standalone function (not a composable) for detecting the current platform:

```typescript
import { isPlatform } from '@ionic/vue';

if (isPlatform('ios')) {
  // iOS-specific logic
}

if (isPlatform('android')) {
  // Android-specific logic
}

if (isPlatform('hybrid')) {
  // Running inside Capacitor or Cordova
}

if (isPlatform('mobileweb')) {
  // Mobile browser, not a native app
}
```

Supported platform identifiers: `android`, `capacitor`, `cordova`, `desktop`, `electron`, `hybrid`, `ios`, `ipad`, `iphone`, `mobile`, `mobileweb`, `phablet`, `pwa`, `tablet`.

A single device can match multiple platforms (e.g., an iPad matches `ios`, `ipad`, `mobile`, `tablet`).

## getPlatforms

Returns an array of all platform identifiers that match the current device:

```typescript
import { getPlatforms } from '@ionic/vue';

const platforms = getPlatforms();
// e.g., ['ios', 'iphone', 'mobile', 'capacitor', 'hybrid']
```

## Custom Platform Detection

Override the default platform detection in the IonicVue configuration:

```typescript
// src/main.ts
import { IonicVue } from '@ionic/vue';

const app = createApp(App);
app.use(IonicVue, {
  platform: {
    desktop: (win: Window) => {
      const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
        win.navigator.userAgent
      );
      return !isMobile;
    },
  },
});
```
