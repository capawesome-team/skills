# Ionic Angular Performance

Optimization techniques for Ionic Angular applications.

## Lazy Loading Routes

Load pages on demand to reduce the initial bundle size.

### Standalone (Angular 17+)

Use `loadComponent` in route definitions:

```typescript
const routes: Routes = [
  {
    path: 'home',
    loadComponent: () => import('./home/home.page').then((m) => m.HomePage),
  },
  {
    path: 'settings',
    loadComponent: () => import('./settings/settings.page').then((m) => m.SettingsPage),
  },
];
```

### NgModule

Use `loadChildren` to lazy-load page modules:

```typescript
const routes: Routes = [
  {
    path: 'home',
    loadChildren: () => import('./home/home.module').then((m) => m.HomePageModule),
  },
  {
    path: 'settings',
    loadChildren: () => import('./settings/settings.module').then((m) => m.SettingsPageModule),
  },
];
```

## trackBy for Lists

When rendering lists with `*ngFor` or `@for`, provide a track function so Angular reuses DOM elements instead of recreating them:

### @for (Angular 17+)

```html
<ion-list>
  @for (item of items; track item.id) {
    <ion-item>
      <ion-label>{{ item.name }}</ion-label>
    </ion-item>
  }
</ion-list>
```

### *ngFor

```html
<ion-list>
  <ion-item *ngFor="let item of items; trackBy: trackById">
    <ion-label>{{ item.name }}</ion-label>
  </ion-item>
</ion-list>
```

```typescript
trackById(index: number, item: Item): number {
  return item.id;
}
```

Without `trackBy` / `track`, Angular destroys and recreates every `<ion-item>` on each change detection cycle, causing visible jank on large lists.

## Virtual Scrolling

For long lists (100+ items), use `ion-virtual-scroll` (Ionic 7) or Angular CDK virtual scroll to render only visible items:

### Angular CDK Virtual Scroll

```bash
npm install @angular/cdk
```

```typescript
import { Component } from '@angular/core';
import { ScrollingModule } from '@angular/cdk/scrolling';
import { IonItem, IonLabel } from '@ionic/angular/standalone';

@Component({
  selector: 'app-long-list',
  template: `
    <cdk-virtual-scroll-viewport itemSize="48" class="ion-content-scroll-host">
      <ion-item *cdkVirtualFor="let item of items">
        <ion-label>{{ item.name }}</ion-label>
      </ion-item>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    cdk-virtual-scroll-viewport {
      height: 100%;
    }
  `],
  standalone: true,
  imports: [ScrollingModule, IonItem, IonLabel],
})
export class LongListComponent {
  items = Array.from({ length: 10000 }, (_, i) => ({ name: `Item ${i}` }));
}
```

## OnPush Change Detection

Use `ChangeDetectionStrategy.OnPush` on components to reduce change detection cycles. The component only re-renders when its `@Input` references change, an event handler fires, or an `async` pipe emits:

```typescript
import { Component, ChangeDetectionStrategy, Input } from '@angular/core';

@Component({
  selector: 'app-item-card',
  template: `
    <ion-card>
      <ion-card-header>
        <ion-card-title>{{ item.name }}</ion-card-title>
      </ion-card-header>
    </ion-card>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true,
  imports: [/* Ionic imports */],
})
export class ItemCardComponent {
  @Input() item!: { name: string };
}
```

## Preloading Strategy

Preload lazy-loaded routes in the background after the initial load completes:

```typescript
import { PreloadAllModules, provideRouter, withPreloading } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules)),
    provideIonicAngular({}),
  ],
};
```

For NgModule:

```typescript
RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
```

This loads all lazy routes in the background immediately after the app starts, so subsequent navigation is instant.

## Image Optimization

Use `NgOptimizedImage` (Angular 15+) for optimized image loading:

```typescript
import { NgOptimizedImage } from '@angular/common';

@Component({
  // ...
  standalone: true,
  imports: [NgOptimizedImage],
  template: `
    <img ngSrc="/assets/hero.jpg" width="400" height="300" priority />
  `,
})
```

- `priority` — disables lazy loading for above-the-fold images.
- Without `priority`, images are lazy-loaded by default.

## Avoiding Common Performance Pitfalls

1. **Do not call methods in templates for computed values.** Methods in templates execute on every change detection cycle. Use signals, computed properties, or pipes instead:

   ```html
   <!-- Bad: getTotal() runs on every change detection -->
   <p>{{ getTotal() }}</p>

   <!-- Good: use a signal or computed -->
   <p>{{ total() }}</p>

   <!-- Good: use a pipe -->
   <p>{{ items | sumPipe }}</p>
   ```

2. **Unsubscribe from observables.** Leaked subscriptions cause memory growth and unnecessary computation. Use `takeUntil`, `DestroyRef`, or the `async` pipe.

3. **Use `ion-skeleton-text` for perceived performance.** Show skeleton placeholders while data loads instead of a blank page or spinner:

   ```html
   @if (loading) {
     <ion-item>
       <ion-skeleton-text animated style="width: 60%"></ion-skeleton-text>
     </ion-item>
   } @else {
     <ion-item>
       <ion-label>{{ item.name }}</ion-label>
     </ion-item>
   }
   ```
