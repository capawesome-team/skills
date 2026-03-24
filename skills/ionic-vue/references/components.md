# Ionic Vue Component Patterns

## Importing and Using Ionic Components

All Ionic components are imported from `@ionic/vue` and used in Vue templates with their kebab-case tag names:

```vue
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>My Page</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content class="ion-padding">
      <ion-button @click="handleClick">Click Me</ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonButton,
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/vue';

const handleClick = () => {
  console.log('Button clicked');
};
</script>
```

Each component must be explicitly imported. If a component is not imported, Vue will log a warning: `Failed to resolve component: ion-button`.

## IonPage — Required Root Wrapper

Every routed page component must use `IonPage` as its root template element:

```vue
<template>
  <ion-page>
    <!-- All page content goes here -->
  </ion-page>
</template>
```

`IonPage` is required for:
- Page transition animations to work correctly.
- Ionic CSS baseline styles to apply.
- Proper flexbox layout that prevents content from overlapping with `IonTabBar` or `IonHeader`.
- Ionic lifecycle hooks (`onIonViewWillEnter`, etc.) to fire.

Components rendered inside modals or popovers do **not** require `IonPage` unless they need layout dimension calculations.

## IonRouterOutlet

The container that renders routed page views:

```vue
<template>
  <ion-app>
    <ion-router-outlet />
  </ion-app>
</template>

<script setup lang="ts">
import { IonApp, IonRouterOutlet } from '@ionic/vue';
</script>
```

Key behaviors:
- Multiple pages can exist in the DOM at the same time (for transition animations and state preservation).
- Never place content directly inside `IonRouterOutlet`.
- Nested `IonRouterOutlet` instances are used for nested route hierarchies (e.g., inside tabs).

## Accessing Web Component Methods

Ionic components are Web Components. To call methods on them (e.g., `scrollToBottom` on `IonContent`), access the underlying DOM element via the `$el` property on the Vue ref:

```vue
<template>
  <ion-page>
    <ion-content ref="contentRef">
      <!-- content -->
    </ion-content>
    <ion-button @click="scrollToBottom">Scroll Down</ion-button>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { IonButton, IonContent, IonPage } from '@ionic/vue';

const contentRef = ref();

const scrollToBottom = () => {
  contentRef.value.$el.scrollToBottom(300);
};
</script>
```

**Important:** Call `contentRef.value.$el.scrollToBottom()`, not `contentRef.value.scrollToBottom()`. The Vue ref wraps the component instance; `$el` gives the actual Web Component DOM element.

## Using Icons (Ionicons)

Ionicons are included with Ionic Vue. Import icons as SVG references and bind them to the `icon` prop:

```vue
<template>
  <ion-icon :icon="heart" />
  <ion-icon :icon="heartOutline" />
</template>

<script setup lang="ts">
import { IonIcon } from '@ionic/vue';
import { heart, heartOutline } from 'ionicons/icons';
</script>
```

Platform-specific icons use the `md` and `ios` props:

```vue
<ion-icon :md="heart" :ios="heartOutline" />
```

Never pass icon names as strings. Always import the SVG reference from `ionicons/icons`.

## v-model on Ionic Components

Many Ionic form components support `v-model` for two-way data binding:

```vue
<template>
  <ion-input v-model="name" label="Name" label-placement="floating" />
  <ion-toggle v-model="darkMode">Dark Mode</ion-toggle>
  <ion-select v-model="color" label="Favorite Color" label-placement="floating">
    <ion-select-option value="red">Red</ion-select-option>
    <ion-select-option value="blue">Blue</ion-select-option>
  </ion-select>
  <ion-checkbox v-model="agreed">I agree</ion-checkbox>
  <ion-range v-model="volume" :min="0" :max="100" label="Volume" />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import {
  IonCheckbox,
  IonInput,
  IonRange,
  IonSelect,
  IonSelectOption,
  IonToggle,
} from '@ionic/vue';

const name = ref('');
const darkMode = ref(false);
const color = ref('red');
const agreed = ref(false);
const volume = ref(50);
</script>
```

`v-model` works because `@ionic/vue` provides Vue-specific wrappers around the underlying Web Components.

## Overlay Components (Modals, Action Sheets, Alerts, Popovers)

Ionic Vue supports inline overlay usage with `v-model` binding for visibility:

### IonModal

```vue
<template>
  <ion-page>
    <ion-content>
      <ion-button @click="isOpen = true">Open Modal</ion-button>

      <ion-modal :is-open="isOpen" @didDismiss="isOpen = false">
        <ion-header>
          <ion-toolbar>
            <ion-title>Modal Title</ion-title>
            <ion-buttons slot="end">
              <ion-button @click="isOpen = false">Close</ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content class="ion-padding">
          <p>Modal content goes here.</p>
        </ion-content>
      </ion-modal>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import {
  IonButton,
  IonButtons,
  IonContent,
  IonHeader,
  IonModal,
  IonPage,
  IonTitle,
  IonToolbar,
} from '@ionic/vue';

const isOpen = ref(false);
</script>
```

### IonModal with a Separate Component

Pass a component to the modal using the `component` prop or inline template. For complex modals, use a dedicated component:

```vue
<ion-modal :is-open="isOpen" @didDismiss="isOpen = false">
  <modal-content @close="isOpen = false" />
</ion-modal>
```

### IonAlert

```vue
<template>
  <ion-button @click="showAlert = true">Show Alert</ion-button>
  <ion-alert
    :is-open="showAlert"
    header="Confirm"
    message="Are you sure?"
    :buttons="alertButtons"
    @didDismiss="showAlert = false"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { IonAlert, IonButton } from '@ionic/vue';

const showAlert = ref(false);
const alertButtons = ['Cancel', 'OK'];
</script>
```

### IonActionSheet

```vue
<template>
  <ion-button @click="showActionSheet = true">Show Action Sheet</ion-button>
  <ion-action-sheet
    :is-open="showActionSheet"
    header="Actions"
    :buttons="actionSheetButtons"
    @didDismiss="showActionSheet = false"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { IonActionSheet, IonButton } from '@ionic/vue';

const showActionSheet = ref(false);
const actionSheetButtons = [
  { text: 'Delete', role: 'destructive' },
  { text: 'Share' },
  { text: 'Cancel', role: 'cancel' },
];
</script>
```

## Event Handling

Ionic components emit custom events. In Vue templates, use kebab-case for event names:

```vue
<ion-searchbar @ion-change="onSearchChange" />
<ion-infinite-scroll @ion-infinite="onInfiniteScroll" />
<ion-refresher slot="fixed" @ion-refresh="onRefresh" />
```

When using `addEventListener` programmatically (not recommended), event names must also be kebab-case:

```typescript
element.addEventListener('ion-modal-did-present', () => { /* ... */ });
```

## Performance: Using key with v-for

When rendering lists with `v-for` and Ionic components, always provide a stable `key`:

```vue
<ion-list>
  <ion-item v-for="item in items" :key="item.id">
    <ion-label>{{ item.name }}</ion-label>
  </ion-item>
</ion-list>
```

Use a unique property from the data object (e.g., `item.id`), not the array index. This allows Vue to efficiently update existing component instances rather than recreating them.
