# Ionic Angular Lifecycle

Ionic page lifecycle hooks and their relationship with Angular lifecycle hooks.

## Ionic Page Events

Ionic provides four lifecycle hooks for page components routed via `<ion-router-outlet>`. These hooks only fire on components directly mapped to a route, not on child components.

| Hook                  | Fires When                                                       |
| --------------------- | ---------------------------------------------------------------- |
| `ionViewWillEnter`    | The page is about to animate into view (before transition).      |
| `ionViewDidEnter`     | The page has finished animating into view (after transition).    |
| `ionViewWillLeave`    | The page is about to animate out of view (before transition).    |
| `ionViewDidLeave`     | The page has finished animating out of view (after transition).  |

## Execution Order

When navigating to a page for the first time:

1. `ngOnInit` — Angular initializes the component (fires **once**).
2. `ionViewWillEnter` — Page is about to enter.
3. `ionViewDidEnter` — Page has entered.

When navigating away:

4. `ionViewWillLeave` — Page is about to leave.
5. `ionViewDidLeave` — Page has left.

When returning to a cached page (e.g., navigating back):

1. `ionViewWillEnter` — fires again.
2. `ionViewDidEnter` — fires again.

`ngOnInit` does **not** fire again because Ionic keeps the component in the DOM.

## Key Difference from Angular

Angular's `ngOnInit` fires once when the component is created, and `ngOnDestroy` fires when it is destroyed. Ionic's navigation keeps pages in the DOM after navigating away (for smooth back-navigation animations). Therefore:

- `ngOnInit` fires only on first visit.
- `ngOnDestroy` fires only when the page is popped from the navigation stack (e.g., back navigation removes it).
- `ionViewWillEnter` / `ionViewDidEnter` fire on **every** visit (first and subsequent).

Use Ionic lifecycle hooks for logic that must run on every page visit (e.g., refreshing data). Use `ngOnInit` for one-time setup.

## Implementation

### Standalone Component

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { ViewWillEnter, ViewDidEnter, ViewWillLeave, ViewDidLeave } from '@ionic/angular';
import { IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/angular/standalone';

@Component({
  selector: 'app-home',
  template: `
    <ion-header>
      <ion-toolbar>
        <ion-title>Home</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <p>{{ message }}</p>
    </ion-content>
  `,
  standalone: true,
  imports: [IonHeader, IonToolbar, IonTitle, IonContent],
})
export class HomePage
  implements OnInit, OnDestroy, ViewWillEnter, ViewDidEnter, ViewWillLeave, ViewDidLeave
{
  message = '';

  ngOnInit() {
    // One-time setup. Fires once when the component is first created.
  }

  ionViewWillEnter() {
    // Fires every time the page is about to become visible.
    // Use for refreshing data.
  }

  ionViewDidEnter() {
    // Fires every time the page has fully transitioned in.
    // Use for starting animations or focus management.
  }

  ionViewWillLeave() {
    // Fires every time the page is about to leave.
    // Use for pausing tasks, saving draft state.
  }

  ionViewDidLeave() {
    // Fires every time the page has fully transitioned out.
    // Use for cleanup that should happen after the page is no longer visible.
  }

  ngOnDestroy() {
    // Fires when the page is removed from the DOM (popped from nav stack).
    // Use for final cleanup: unsubscribe observables, remove listeners.
  }
}
```

### NgModule Component

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { ViewWillEnter, ViewDidEnter, ViewWillLeave, ViewDidLeave } from '@ionic/angular';

@Component({
  selector: 'app-home',
  templateUrl: './home.page.html',
})
export class HomePage
  implements OnInit, OnDestroy, ViewWillEnter, ViewDidEnter, ViewWillLeave, ViewDidLeave
{
  ngOnInit() {}
  ionViewWillEnter() {}
  ionViewDidEnter() {}
  ionViewWillLeave() {}
  ionViewDidLeave() {}
  ngOnDestroy() {}
}
```

The interfaces (`ViewWillEnter`, `ViewDidEnter`, `ViewWillLeave`, `ViewDidLeave`) are imported from `@ionic/angular` in both standalone and NgModule architectures. They are optional but recommended for type checking.

## Common Patterns

### Refreshing Data on Every Visit

```typescript
ionViewWillEnter() {
  this.loadItems();
}

private async loadItems() {
  this.items = await this.itemService.getAll();
}
```

### Pausing and Resuming Subscriptions

```typescript
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

export class DashboardPage implements ViewWillEnter, ViewWillLeave, OnDestroy {
  private leave$ = new Subject<void>();
  private destroy$ = new Subject<void>();

  ionViewWillEnter() {
    this.dataService.stream$
      .pipe(takeUntil(this.leave$))
      .subscribe((data) => {
        this.data = data;
      });
  }

  ionViewWillLeave() {
    this.leave$.next();
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### Deferring Work Until After Animation

If data loading causes jank during the page transition animation, move it to `ionViewDidEnter` instead of `ionViewWillEnter`:

```typescript
ionViewDidEnter() {
  // Heavy work here does not block the page transition animation.
  this.loadHeavyData();
}
```
