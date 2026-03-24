# Ionic Vue Lifecycle Hooks

Ionic provides four lifecycle hooks that fire during page transitions managed by `IonRouterOutlet`. These are separate from Vue's standard lifecycle hooks (`onMounted`, `onUnmounted`, etc.).

## Why Ionic Lifecycle Hooks Exist

`IonRouterOutlet` keeps previously visited pages in the DOM rather than destroying them. This enables smooth back-navigation transitions and preserves scroll position and form state. Because pages are not destroyed, Vue's `onMounted` and `onUnmounted` do not fire on subsequent visits. Use Ionic lifecycle hooks to run logic every time a page is shown or hidden.

## The Four Hooks

| Hook | Fires When | Typical Use |
|------|-----------|-------------|
| `onIonViewWillEnter` | Page is about to animate into view | Load or refresh data from a service |
| `onIonViewDidEnter` | Page has finished animating into view | Start animations, focus inputs, begin timers |
| `onIonViewWillLeave` | Page is about to animate out of view | Save form state, unsubscribe from observables |
| `onIonViewDidLeave` | Page has finished animating out of view | Stop timers, clean up resources not needed off-screen |

## Composition API Usage (Recommended)

```vue
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>My Page</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <p>{{ message }}</p>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  onIonViewWillEnter,
  onIonViewDidEnter,
  onIonViewWillLeave,
  onIonViewDidLeave,
} from '@ionic/vue';

const message = ref('');

onIonViewWillEnter(() => {
  // Runs every time the page is about to be shown.
  // Good place to fetch fresh data.
  message.value = 'Loading...';
});

onIonViewDidEnter(() => {
  // Page is now fully visible.
  message.value = 'Page is active';
});

onIonViewWillLeave(() => {
  // Page is about to leave.
  // Save state or unsubscribe here.
});

onIonViewDidLeave(() => {
  // Page has fully left the view.
  // Clean up timers or heavy resources.
});
</script>
```

## Options API Usage

For components using the Options API, define the lifecycle methods as functions at the component root level:

```vue
<script>
import { defineComponent } from 'vue';
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar } from '@ionic/vue';

export default defineComponent({
  components: { IonContent, IonHeader, IonPage, IonTitle, IonToolbar },
  data() {
    return {
      message: '',
    };
  },
  ionViewWillEnter() {
    this.message = 'Loading...';
  },
  ionViewDidEnter() {
    this.message = 'Page is active';
  },
  ionViewWillLeave() {
    // cleanup
  },
  ionViewDidLeave() {
    // cleanup
  },
});
</script>
```

## Critical Requirement: IonPage Wrapper

Ionic lifecycle hooks **only fire** on components that have `IonPage` as their root template element. If `IonPage` is missing, the hooks silently do nothing.

```vue
<!-- CORRECT: IonPage is the root element -->
<template>
  <ion-page>
    <ion-content>...</ion-content>
  </ion-page>
</template>

<!-- INCORRECT: Missing IonPage — lifecycle hooks will not fire -->
<template>
  <ion-content>...</ion-content>
</template>
```

## Scope: Only Router-Mapped Components

Ionic lifecycle hooks only fire on components that are **directly mapped to a route** in the router configuration. They do **not** fire on child components rendered inside a page.

If a child component needs to respond to page lifecycle events, the parent page component should call a method or emit an event to the child.

## Combining with Vue Lifecycle Hooks

Vue's standard hooks still work but behave differently due to DOM caching:

| Vue Hook | Behavior with IonRouterOutlet |
|----------|-------------------------------|
| `onMounted` | Fires once when the page is first created. Does **not** fire again on subsequent visits. |
| `onUnmounted` | Fires only when the page is evicted from the DOM cache (not on every navigation away). |
| `onActivated` | Not used — `IonRouterOutlet` does not use Vue's `<keep-alive>`. |

**Recommendation:** Use `onMounted` for one-time setup (e.g., initializing a map). Use `onIonViewWillEnter` for data that must refresh every time the page is shown.
