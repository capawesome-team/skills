---
name: ionic-angular
description: "Guides the agent through Angular-specific patterns for Ionic app development. Covers project structure, standalone vs NgModule architecture detection, Angular Router integration with Ionic navigation (tabs, side menu, modals), lazy loading, Ionic page lifecycle hooks, reactive forms with Ionic input components, Angular services for state management, route guards, performance optimization, and testing. Do not use for creating a new Ionic app from scratch, upgrading Ionic versions, general Ionic component usage unrelated to Angular, Capacitor plugin integration, or non-Angular frameworks (React, Vue)."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/ionic-angular
---

# Ionic Angular

Ionic development with Angular — project structure, Angular-specific components, navigation, lazy loading, forms, and lifecycle integration.

## Prerequisites

1. **Ionic Framework 7 or 8** with `@ionic/angular`.
2. **Angular 16+** (Angular 17+ recommended for standalone architecture).
3. Node.js and npm installed.
4. Ionic CLI installed (`npm install -g @ionic/cli`).

## Agent Behavior

- **Auto-detect architecture.** Check `src/main.ts` for `bootstrapApplication` (standalone) vs. `platformBrowserDynamic().bootstrapModule` (NgModule). Adapt all code examples to the detected architecture.
- **Guide step-by-step.** Walk the user through one topic at a time.
- **Adapt imports.** For standalone components, import each Ionic component from `@ionic/angular/standalone`. For NgModule components, import `IonicModule` in the page module.

## Procedures

### Step 1: Analyze the Project

Auto-detect the following by reading project files — do **not** ask the user for information that can be inferred:

1. **Ionic version**: Read `@ionic/angular` version from `package.json`.
2. **Angular version**: Read `@angular/core` version from `package.json`.
3. **Architecture**: Check `src/main.ts` for `bootstrapApplication` (standalone) or `platformBrowserDynamic().bootstrapModule` (NgModule). If `src/app/app.module.ts` exists and imports `IonicModule`, confirm NgModule.
4. **Capacitor**: Check if `@capacitor/core` is in `package.json`. Check for `android/` and `ios/` directories.
5. **Routing**: Check for `src/app/app.routes.ts` (standalone) or `src/app/app-routing.module.ts` (NgModule).
6. **Navigation pattern**: Check `src/app/` for a `tabs/` directory (tab-based) or `ion-menu` usage in templates (side menu).

### Step 2: Understand Project Structure

Read `references/project-structure.md` for the standard Ionic Angular directory layout.

Key points:
- **Standalone projects** have `app.config.ts`, `app.routes.ts`, and standalone page components.
- **NgModule projects** have `app.module.ts`, `app-routing.module.ts`, and per-page `*.module.ts` files.
- Ionic config lives in `ionic.config.json`.
- Theme variables are in `src/theme/variables.scss`.
- Global styles and Ionic CSS imports are in `src/global.scss`.

### Step 3: Understand Architecture Differences

Read `references/standalone-vs-ngmodules.md` for detailed differences.

Summary:

| Aspect            | Standalone                                      | NgModule                                    |
| ----------------- | ----------------------------------------------- | ------------------------------------------- |
| Bootstrap         | `bootstrapApplication` in `main.ts`             | `platformBrowserDynamic().bootstrapModule`   |
| Ionic setup       | `provideIonicAngular({})` in `app.config.ts`    | `IonicModule.forRoot()` in `app.module.ts`  |
| Component imports | Import each Ionic component per file             | `IonicModule` provides all globally         |
| Import source     | `@ionic/angular/standalone`                      | `@ionic/angular`                            |
| Lazy loading      | `loadComponent` in routes                        | `loadChildren` in routes                    |
| Icon registration | `addIcons()` from `ionicons` required            | Automatic                                   |
| Tree-shaking      | Yes                                              | No                                          |

### Step 4: Set Up Navigation

Read `references/navigation.md` for full navigation patterns.

Topics covered:
- **Route configuration** — defining routes with lazy loading for both architectures.
- **Root component layout** — `<ion-app>` with `<ion-router-outlet>`.
- **Template navigation** — `routerLink` with `routerDirection` for transition animations.
- **Programmatic navigation** — `NavController` (`navigateForward`, `navigateBack`, `navigateRoot`, `back`).
- **Route parameters** — reading params via `ActivatedRoute`.
- **Tab navigation** — `<ion-tabs>` with child routes per tab.
- **Side menu** — `<ion-menu>` with `<ion-menu-toggle>` and child routes.
- **Route guards** — functional guards (Angular 15.2+) with `CanActivateFn`.
- **Modals** — `ModalController` for overlay navigation with data passing.
- **Linear vs. non-linear routing** — when to use each pattern.

### Step 5: Implement Lifecycle Hooks

Read `references/lifecycle.md` for Ionic page lifecycle hook details.

Key rules:
- `ngOnInit` fires **once** (first creation). Use for one-time setup.
- `ionViewWillEnter` fires on **every** page visit. Use for refreshing data.
- `ionViewDidEnter` fires after the transition animation. Use for heavy work that would block animations.
- `ionViewWillLeave` fires before leaving. Use for cleanup, pausing subscriptions.
- `ngOnDestroy` fires when the page is popped from the navigation stack. Use for final cleanup.
- Implement interfaces `ViewWillEnter`, `ViewDidEnter`, `ViewWillLeave`, `ViewDidLeave` from `@ionic/angular`.

### Step 6: Build Forms

Read `references/forms.md` for form patterns with Ionic components.

Topics covered:
- **Reactive forms** — `FormBuilder`, `FormGroup`, `Validators` with Ionic input components.
- **Template-driven forms** — `[(ngModel)]` with `FormsModule`.
- **Ionic form components** — `ion-input`, `ion-textarea`, `ion-select`, `ion-checkbox`, `ion-toggle`, `ion-radio-group`, `ion-range`, `ion-datetime`.
- **Validation display** — `errorText` (Ionic 7+) or manual `<ion-note>` for older versions.
- **Label placement** — `labelPlacement` attribute (`floating`, `stacked`, `fixed`).

### Step 7: Manage Services and State

Read `references/services-and-state.md` for service and state management patterns.

Topics covered:
- **Injectable services** — `@Injectable({ providedIn: 'root' })`.
- **HTTP data fetching** — `HttpClient` setup for standalone and NgModule, CRUD service pattern.
- **Reactive state** — `BehaviorSubject` for shared observable state.
- **Angular signals** — `signal()` and `computed()` for reactive state (Angular 16+).
- **Loading indicators** — `LoadingController` for async operations.
- **Error feedback** — `ToastController` for user-facing error/success messages.
- **Environment config** — `environment.ts` / `environment.prod.ts` for API URLs.

### Step 8: Optimize Performance

Read `references/performance.md` for performance optimization techniques.

Topics covered:
- **Lazy loading** — `loadComponent` (standalone) and `loadChildren` (NgModule).
- **trackBy / track** — efficient list rendering with `*ngFor` and `@for`.
- **Virtual scrolling** — Angular CDK `cdk-virtual-scroll-viewport` for large lists.
- **OnPush change detection** — reduce unnecessary re-renders.
- **Preloading strategy** — `PreloadAllModules` for instant subsequent navigation.
- **Skeleton text** — `ion-skeleton-text` for perceived performance during loading.

### Step 9: Write Tests

Read `references/testing.md` for testing patterns.

Topics covered:
- **Unit testing pages** — `TestBed` setup for standalone and NgModule pages.
- **Testing services** — basic services and HTTP services with `HttpTestingController`.
- **Mocking dependencies** — `jasmine.createSpyObj` for service and controller mocks.
- **Testing navigation** — mocking `NavController`.
- **Testing guards** — `TestBed.runInInjectionContext` for functional guards.
- **E2E testing** — Cypress and Playwright setup.

## Error Handling

- **`NullInjectorError: No provider for IonRouterOutlet`**: In standalone apps, verify `provideIonicAngular({})` is called in `app.config.ts`. In NgModule apps, verify `IonicModule.forRoot()` is imported in `app.module.ts`.
- **Ionic components not rendering**: In standalone components, verify each Ionic component is imported from `@ionic/angular/standalone` and listed in the `imports` array. In NgModule components, verify `IonicModule` is imported in the page module.
- **Icons not showing (standalone)**: Verify `addIcons()` is called with the required icons from `ionicons/icons`, and `IonIcon` is imported from `@ionic/angular/standalone`.
- **`ionViewWillEnter` not firing**: Verify the component is directly routed via `<ion-router-outlet>`. Child components do not receive Ionic lifecycle events.
- **Page data not refreshing on back navigation**: Use `ionViewWillEnter` instead of `ngOnInit` for data loading. Ionic caches pages in the DOM, so `ngOnInit` only fires once.
- **Form validation errors not displaying**: For Ionic 7+, use `errorText` on `<ion-input>`. For older versions, manually check `control.invalid && control.touched` and show `<ion-note>`.
- **`routerLink` not working on Ionic components**: In standalone components, import `RouterLink` from `@angular/router` in the component's `imports` array.
- **Page transitions not animating**: Use `NavController` from `@ionic/angular` instead of Angular's `Router` for animated navigation. Alternatively, add `routerDirection` to `routerLink` elements.
- **Tab navigation broken**: Verify the `tab` attribute on `<ion-tab-button>` matches the child route `path`. Do not programmatically navigate between tabs.

## Related Skills

- **`capacitor-angular`** — Capacitor-specific Angular patterns (plugin services, NgZone, deep links, platform detection).
- **`ionic-app-development`** — General Ionic development (components, theming, CLI).
- **`ionic-app-creation`** — Create a new Ionic app from scratch.
- **`ionic-app-upgrades`** — Upgrade Ionic to a newer version.
