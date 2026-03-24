---
name: ionic-react
description: "Guides the agent through Ionic Framework development with React — project structure, React-specific Ionic components, IonReactRouter and navigation patterns, Ionic lifecycle hooks (useIonViewWillEnter, useIonViewDidEnter, useIonViewWillLeave, useIonViewDidLeave), state management integration, and React-specific best practices for Ionic apps. Do not use for plain Capacitor React apps without Ionic (use capacitor-react), Ionic with Angular or Vue, creating a new Ionic app (use ionic-app-creation), upgrading Ionic to a newer version (use ionic-app-upgrades), or general Ionic component usage without React-specific context (use ionic-app-development)."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/ionic-react
---

# Ionic React

Develop Ionic Framework apps with React — project structure, IonReactRouter, React-specific components, lifecycle hooks, state management, and best practices.

## Prerequisites

1. **Ionic Framework 7 or 8** with `@ionic/react`.
2. **React 18** or later.
3. Node.js and npm installed.
4. For **iOS**: Xcode installed.
5. For **Android**: Android Studio installed.

## Agent Behavior

- **Auto-detect before asking.** Check the project for `package.json` dependencies (`@ionic/react`, `@ionic/react-router`, `react`, `@capacitor/core`), platforms (`android/`, `ios/`), build tools, and TypeScript usage. Only ask the user when something cannot be detected.
- **Guide step-by-step.** Walk the user through the process one step at a time. Never present multiple unrelated questions at once.
- **Adapt to the project.** Detect the existing code style (TypeScript vs. JavaScript, state management library, routing setup) and generate code that matches.

## Procedures

### Step 1: Analyze the Project

Auto-detect the following by reading project files:

1. **Ionic version**: Read `@ionic/react` version from `package.json`.
2. **React version**: Read `react` version from `package.json`.
3. **Capacitor version**: Read `@capacitor/core` version from `package.json` (if present).
4. **TypeScript**: Check if `tsconfig.json` exists and if `.tsx` files are used.
5. **Router**: Check if `@ionic/react-router` and `react-router-dom` are in `package.json`.
6. **State management**: Check `package.json` for `redux`, `@reduxjs/toolkit`, `zustand`, `jotai`, `@tanstack/react-query`, or similar.
7. **Platforms**: Check which directories exist (`android/`, `ios/`).
8. **Build tool**: Check for `vite.config.ts`, `angular.json`, `webpack.config.js`, etc.

### Step 2: Project Structure

A standard Ionic React project follows this structure:

```
project-root/
├── android/                  # Android native project (Capacitor)
├── ios/                      # iOS native project (Capacitor)
├── public/
├── src/
│   ├── components/           # Reusable UI components
│   ├── hooks/                # Custom React hooks
│   ├── pages/                # Page components (one per route)
│   ├── services/             # Service modules for API and native calls
│   ├── context/              # React context providers
│   ├── theme/
│   │   └── variables.css     # Ionic CSS custom properties
│   ├── App.tsx               # Root component with IonReactRouter
│   └── main.tsx              # Entry point with setupIonicReact()
├── capacitor.config.ts       # Capacitor configuration
├── ionic.config.json         # Ionic CLI configuration
├── package.json
├── tsconfig.json
└── vite.config.ts            # Or other bundler config
```

If the project does not follow this structure, adapt all guidance to the project's actual directory layout. Do **not** restructure the project unless the user explicitly asks.

### Step 3: App Initialization

The entry point must call `setupIonicReact()` before rendering the app. This function initializes the Ionic Framework for React.

Verify that `src/main.tsx` (or `src/index.tsx`) contains:

```typescript
import { setupIonicReact } from '@ionic/react';

setupIonicReact();
```

`setupIonicReact()` accepts an optional configuration object for global settings:

```typescript
setupIonicReact({
  mode: 'ios',          // Force iOS mode on all platforms ('ios' | 'md')
  rippleEffect: false,  // Disable Material Design ripple effect
  animated: true,       // Enable/disable all animations
});
```

If `setupIonicReact()` is missing or called after rendering, Ionic components will not function correctly.

### Step 4: Routing and Navigation

Read `references/routing.md` for complete routing patterns including:
- `IonReactRouter` setup
- `IonRouterOutlet` page management
- Tab-based navigation with `IonTabs`
- Side menu navigation with `IonMenu`
- Programmatic navigation with `useIonRouter`
- Route guards and route parameters

Key principles:
1. **Use `IonReactRouter`** instead of React Router's `BrowserRouter`. It is required for Ionic page transitions.
2. **Use `IonRouterOutlet`** to contain routes. It manages the page stack and transition animations.
3. **Pass `component` prop to `Route`** — do not use `render` or `children` inside `IonRouterOutlet`.
4. **Use `useIonRouter`** for programmatic navigation — it provides Ionic-aware navigation with transition animations.

### Step 5: Ionic Lifecycle Hooks

Read `references/lifecycle.md` for detailed lifecycle hook usage.

Ionic React pages stay mounted in the DOM after navigation. Standard React `useEffect` only fires on initial mount, not on every page visit. Use Ionic lifecycle hooks for per-visit logic:

| Hook                    | Use for                                                |
| ----------------------- | ------------------------------------------------------ |
| `useIonViewWillEnter`   | Refresh data before the page becomes visible           |
| `useIonViewDidEnter`    | Start animations or focus inputs after page is visible |
| `useIonViewWillLeave`   | Pause media, save draft state                          |
| `useIonViewDidLeave`    | Cleanup after page is fully hidden                     |

These hooks only work in components that:
- Are rendered as the `component` of a `Route` inside `IonRouterOutlet`.
- Render `IonPage` as their root element.

### Step 6: React-Specific Ionic Hooks

Read `references/hooks.md` for all available hooks and usage examples.

Ionic React provides hooks for presenting overlays and controlling navigation without managing state manually:

| Hook                | Purpose                              |
| ------------------- | ------------------------------------ |
| `useIonAlert`       | Present alert dialogs                |
| `useIonToast`       | Show toast notifications             |
| `useIonActionSheet` | Present action sheets                |
| `useIonLoading`     | Show/dismiss loading indicators      |
| `useIonModal`       | Present modals programmatically      |
| `useIonPopover`     | Present popovers programmatically    |
| `useIonPicker`      | Present picker dialogs               |
| `useIonRouter`      | Navigate with Ionic animations       |

### Step 7: React-Specific Component Patterns

Read `references/components.md` for detailed component patterns including:
- Page structure with `IonPage`
- Inline overlays (modals, popovers) with `isOpen` binding
- Pull-to-refresh with `IonRefresher`
- Infinite scroll with `IonInfiniteScroll`
- Forms with Ionic input components

Key principles:
1. **Every page must render `IonPage` as root.** This is required for transitions and lifecycle hooks.
2. **Use `onIonInput` for text inputs** and `onIonChange` for select, toggle, checkbox, and range components.
3. **Access values via `e.detail.value`** (or `e.detail.checked` for toggles/checkboxes).
4. **Use inline overlays with `isOpen`** for simpler state management, or use overlay hooks (`useIonModal`, `useIonAlert`, etc.) for imperative usage.

### Step 8: State Management

Read `references/state-management.md` for patterns with React Context, Redux Toolkit, Zustand, and TanStack Query.

Key principles:
1. **Page caching affects state.** Since Ionic keeps pages mounted, state persists across navigations. Use `useIonViewWillEnter` to refresh stale data.
2. **Place providers outside `IonReactRouter`.** Context providers, Redux `Provider`, and `QueryClientProvider` should wrap the router so all pages have access.
3. **Do not add a state library unless needed.** For simple apps, React Context and `useState`/`useReducer` are sufficient.

### Step 9: Build and Run

After implementing changes:

```bash
npm run build
npx cap sync
npx cap run android
npx cap run ios
```

For development with live reload:

```bash
ionic serve
```

For native development with live reload:

```bash
ionic cap run android --livereload --external
ionic cap run ios --livereload --external
```

## Error Handling

- **Ionic lifecycle hooks not firing**: Verify that the component renders `IonPage` as its root element and is the `component` of a `Route` inside `IonRouterOutlet`. Lifecycle hooks do not fire in child components or components rendered via `render`/`children` props.
- **Page transitions not animating**: Ensure `IonReactRouter` is used instead of `BrowserRouter`. Ensure routes use the `component` prop, not `render` or `children`. Verify `IonRouterOutlet` is a direct child of `IonReactRouter` or `IonTabs`.
- **Stale data after navigating back**: Ionic keeps pages mounted in the DOM. Use `useIonViewWillEnter` to refresh data on every page visit instead of `useEffect` with `[]`.
- **Tab bar disappears on sub-page**: All routes (including detail routes) must be inside the `IonRouterOutlet` within `IonTabs`. Do not nest a separate `IonRouterOutlet` inside a tab page.
- **`setupIonicReact` errors or components not rendering**: Ensure `setupIonicReact()` is called before `ReactDOM.createRoot()` or `ReactDOM.render()` in the entry file. Ensure all Ionic CSS files are imported (see Ionic CSS imports in `src/App.tsx` or `src/main.tsx`).
- **`IonBackButton` not appearing**: `IonBackButton` only renders when there is navigation history. Set the `defaultHref` prop to provide a fallback route when the page is accessed directly.
- **Form input values not updating**: Use `onIonInput` for `IonInput`/`IonTextarea` (not `onChange`). Use `onIonChange` for `IonSelect`, `IonToggle`, `IonCheckbox`, `IonRange`. Access values via `e.detail.value`.
- **Modal or popover content not styled**: Ensure the overlay content is wrapped in `IonContent`. For modals, include `IonHeader` with `IonToolbar` if a toolbar is needed.
- **React strict mode double-mounting**: In development, React 18 strict mode mounts components twice. This can cause duplicate Ionic event listeners. Ensure cleanup functions properly remove listeners.
- **`useIonRouter` returns undefined**: The hook must be used inside a component that is a descendant of `IonReactRouter`. Verify the component tree includes `IonReactRouter` as an ancestor.
- **CSS custom properties not applying**: Ensure Ionic CSS files are imported in the correct order in `src/App.tsx` or `src/main.tsx`. The theme `variables.css` file must be imported after the core Ionic CSS.

## Related Skills

- **`ionic-app-development`** — General Ionic development covering components, theming, and CLI usage.
- **`ionic-app-creation`** — Create a new Ionic app from scratch.
- **`capacitor-react`** — Capacitor-specific React patterns for accessing native device features (without Ionic Framework).
- **`ionic-app-upgrades`** — Upgrade Ionic Framework to a newer version.
- **`capacitor-plugins`** — Install, configure, and use Capacitor plugins from official and community sources.
