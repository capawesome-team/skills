# Navigation and Routing in Ionic Vue

Ionic Vue uses Vue Router with Ionic-specific components (`IonRouterOutlet`, `IonTabs`) for mobile-optimized page transitions and stack-based navigation.

## Router Setup

Create the router using `createRouter` from `@ionic/vue-router` (not from `vue-router` directly):

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from '@ionic/vue-router';
import { RouteRecordRaw } from 'vue-router';
import HomePage from '@/views/HomePage.vue';

const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    redirect: '/home',
  },
  {
    path: '/home',
    name: 'Home',
    component: HomePage,
  },
];

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
});

export default router;
```

**Important:** Always import `createRouter` from `@ionic/vue-router`, not from `vue-router`. The Ionic version wraps Vue Router to enable page transitions and stack management.

## Declarative Navigation with router-link

Use the `router-link` attribute on any Ionic component for declarative navigation:

```html
<ion-button router-link="/detail">Go to Detail</ion-button>
```

Control the transition direction and animation with additional attributes:

```html
<ion-button
  router-link="/page2"
  router-direction="forward"
  :router-animation="myAnimation"
>
  Navigate Forward
</ion-button>
```

- `router-direction`: `"forward"` | `"back"` | `"root"` — controls the page transition direction.
- `router-animation`: An `AnimationBuilder` function for custom transitions.

## Programmatic Navigation with useIonRouter

The `useIonRouter` composable provides programmatic navigation with full transition control:

```vue
<script setup lang="ts">
import { useIonRouter } from '@ionic/vue';

const ionRouter = useIonRouter();

const goToDetail = () => {
  ionRouter.push('/detail');
};

const goBack = () => {
  if (ionRouter.canGoBack()) {
    ionRouter.back();
  }
};

const replaceRoute = () => {
  ionRouter.replace('/home');
};

const navigateWithAnimation = () => {
  ionRouter.navigate('/page2', 'forward', 'replace', customAnimation);
};
</script>
```

Methods:
- `push(path, animation?)` — Navigate forward to a new route.
- `replace(path, animation?)` — Replace the current route without adding to history.
- `back(animation?)` — Navigate back one step.
- `forward(animation?)` — Navigate forward one step.
- `navigate(path, direction, action, animation?)` — Full control over navigation.
- `canGoBack()` — Returns `true` if there is a previous page in the navigation stack.

**Constraint:** `useIonRouter` calls Vue's `inject()` internally. It must be called inside `setup()` or `<script setup>`.

## Standard Vue Router Navigation

Vue Router's `useRouter` composable also works:

```vue
<script setup lang="ts">
import { useRouter } from 'vue-router';

const router = useRouter();

const goToDetail = () => {
  router.push('/detail');
};
</script>
```

Use `useRoute` to access the current route and its parameters:

```vue
<script setup lang="ts">
import { useRoute } from 'vue-router';

const route = useRoute();
const id = route.params.id;
</script>
```

**Limitation:** `router.go(n)` navigates by history steps but only works with linear routing. Do not use it in tab-based or nested outlet layouts.

## URL Parameters

Define dynamic route segments with colon syntax:

```typescript
const routes: Array<RouteRecordRaw> = [
  {
    path: '/detail/:id',
    name: 'Detail',
    component: () => import('@/views/DetailPage.vue'),
  },
];
```

Access parameters in the component:

```vue
<script setup lang="ts">
import { useRoute } from 'vue-router';

const route = useRoute();
const { id } = route.params;
</script>
```

## Lazy Loading

Load route components on demand to reduce the initial bundle size:

```typescript
const routes: Array<RouteRecordRaw> = [
  {
    path: '/detail',
    name: 'Detail',
    component: () => import('@/views/DetailPage.vue'),
  },
];
```

## Linear vs. Non-Linear Routing

- **Linear routing:** Sequential forward/back navigation (A -> B -> C). `router.go()` works. Suitable for simple flows.
- **Non-linear routing:** Tabs and nested outlets where back navigation does not follow strict history order. `router.go()` does not work. Required for tab-based interfaces.

Default to linear routing. Switch to non-linear only when using tabs or nested `IonRouterOutlet` instances.

## Shared URLs vs. Nested Routes

### Shared URLs

Routes share URL path segments but are defined at the same level:

```typescript
const routes: Array<RouteRecordRaw> = [
  {
    path: '/dashboard',
    component: DashboardMainPage,
  },
  {
    path: '/dashboard/stats',
    component: DashboardStatsPage,
  },
];
```

### Nested Routes

Routes defined as children of a parent, requiring the parent component to render an `IonRouterOutlet`:

```typescript
const routes: Array<RouteRecordRaw> = [
  {
    path: '/dashboard/:id',
    component: DashboardRouterOutlet,
    children: [
      {
        path: '',
        component: DashboardMainPage,
      },
      {
        path: 'stats',
        component: DashboardStatsPage,
      },
    ],
  },
];
```

Default to shared URLs. Use nested routes primarily for tabs.

## Tab-Based Routing

Tab routes use nested routes with `IonTabs`:

```typescript
// src/router/index.ts
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    redirect: '/tabs/tab1',
  },
  {
    path: '/tabs/',
    component: TabsPage,
    children: [
      {
        path: '',
        redirect: 'tab1',
      },
      {
        path: 'tab1',
        component: () => import('@/views/Tab1Page.vue'),
      },
      {
        path: 'tab2',
        component: () => import('@/views/Tab2Page.vue'),
      },
      {
        path: 'tab3',
        component: () => import('@/views/Tab3Page.vue'),
      },
    ],
  },
];
```

The tabs container component:

```vue
<!-- src/views/TabsPage.vue -->
<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet></ion-router-outlet>
      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="tab1" href="/tabs/tab1">
          <ion-icon :icon="triangle" />
          <ion-label>Tab 1</ion-label>
        </ion-tab-button>
        <ion-tab-button tab="tab2" href="/tabs/tab2">
          <ion-icon :icon="ellipse" />
          <ion-label>Tab 2</ion-label>
        </ion-tab-button>
        <ion-tab-button tab="tab3" href="/tabs/tab3">
          <ion-icon :icon="square" />
          <ion-label>Tab 3</ion-label>
        </ion-tab-button>
      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonIcon,
  IonLabel,
  IonPage,
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs,
} from '@ionic/vue';
import { ellipse, square, triangle } from 'ionicons/icons';
</script>
```

### Child Routes Within Tabs

Add child routes as siblings under the tab parent:

```typescript
{
  path: '/tabs/',
  component: TabsPage,
  children: [
    {
      path: 'tab1',
      component: () => import('@/views/Tab1Page.vue'),
    },
    {
      path: 'tab1/detail/:id',
      component: () => import('@/views/Tab1DetailPage.vue'),
    },
  ],
}
```

This keeps Tab 1 selected in the tab bar while rendering the detail page.

### Tab Routing Rules

- Each tab maintains an independent navigation stack.
- Never route between tabs programmatically from within a tab's content. Only the tab bar buttons should switch tabs.
- If a view is needed across multiple tabs, use `IonModal` instead of cross-tab routing.
- To reuse a component in multiple tabs, define duplicate routes pointing to the same component under each tab.

## Router History Modes

- `createWebHistory` (recommended) — Uses the HTML5 History API. Clean URLs without a hash.
- `createWebHashHistory` — Adds `#` to URLs. Useful when server-side URL rewriting is not available.
- `createMemoryHistory` — In-memory history. Used for server-side rendering (SSR).
