# Ionic Angular Navigation

Angular Router integration with Ionic components for page navigation, tabs, and modals.

## Routing Setup

### Standalone (Angular 17+)

Define routes in `src/app/app.routes.ts`:

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    redirectTo: 'home',
    pathMatch: 'full',
  },
  {
    path: 'home',
    loadComponent: () => import('./home/home.page').then((m) => m.HomePage),
  },
  {
    path: 'detail/:id',
    loadComponent: () => import('./detail/detail.page').then((m) => m.DetailPage),
  },
];
```

Register in `src/app/app.config.ts`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideIonicAngular } from '@ionic/angular/standalone';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes), provideIonicAngular({})],
};
```

### NgModule

Define routes in `src/app/app-routing.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { PreloadAllModules, RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: '',
    redirectTo: 'home',
    pathMatch: 'full',
  },
  {
    path: 'home',
    loadChildren: () => import('./home/home.module').then((m) => m.HomePageModule),
  },
  {
    path: 'detail/:id',
    loadChildren: () => import('./detail/detail.module').then((m) => m.DetailPageModule),
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

## Root Component Layout

The root component (`src/app/app.component.html`) must contain `<ion-router-outlet>`:

```html
<ion-app>
  <ion-router-outlet></ion-router-outlet>
</ion-app>
```

For standalone apps, import `IonApp` and `IonRouterOutlet` in `app.component.ts`:

```typescript
import { Component } from '@angular/core';
import { IonApp, IonRouterOutlet } from '@ionic/angular/standalone';

@Component({
  selector: 'app-root',
  template: '<ion-app><ion-router-outlet></ion-router-outlet></ion-app>',
  standalone: true,
  imports: [IonApp, IonRouterOutlet],
})
export class AppComponent {}
```

## Template-Based Navigation

Use `routerLink` on Ionic buttons and items:

```html
<ion-button routerLink="/detail/42">View Detail</ion-button>

<ion-item routerLink="/detail/42" routerDirection="forward" detail>
  <ion-label>Item 42</ion-label>
</ion-item>
```

The `routerDirection` attribute controls the page transition animation:
- `"forward"` — slide-in-from-right animation.
- `"back"` — slide-in-from-left animation.
- `"root"` — no animation (instant swap).

Import `RouterLink` in standalone components:

```typescript
import { RouterLink } from '@angular/router';
import { IonButton } from '@ionic/angular/standalone';

@Component({
  // ...
  standalone: true,
  imports: [IonButton, RouterLink],
})
```

## Programmatic Navigation

Inject `NavController` from `@ionic/angular` for animated navigation, or `Router` from `@angular/router` for standard navigation:

```typescript
import { Component, inject } from '@angular/core';
import { NavController } from '@ionic/angular';

@Component({ /* ... */ })
export class HomePage {
  private navCtrl = inject(NavController);

  goToDetail(id: number) {
    this.navCtrl.navigateForward(`/detail/${id}`);
  }

  goBack() {
    this.navCtrl.back();
  }

  goToRoot() {
    this.navCtrl.navigateRoot('/home');
  }
}
```

`NavController` methods:
- `navigateForward(url)` — push with forward animation.
- `navigateBack(url)` — navigate with back animation.
- `navigateRoot(url)` — set root page (no animation).
- `back()` — pop the current page.

For standalone, import `NavController` from `@ionic/angular/standalone`.

## Reading Route Parameters

```typescript
import { Component, OnInit, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({ /* ... */ })
export class DetailPage implements OnInit {
  private route = inject(ActivatedRoute);
  itemId: string | null = null;

  ngOnInit() {
    this.itemId = this.route.snapshot.paramMap.get('id');
  }
}
```

## Tab-Based Navigation

### Route Configuration

Define a parent route for tabs with child routes per tab. Each tab has its own `<ion-router-outlet>` via nested routing:

```typescript
const routes: Routes = [
  {
    path: 'tabs',
    loadComponent: () => import('./tabs/tabs.page').then((m) => m.TabsPage),
    children: [
      {
        path: 'home',
        loadComponent: () => import('./home/home.page').then((m) => m.HomePage),
      },
      {
        path: 'settings',
        loadComponent: () => import('./settings/settings.page').then((m) => m.SettingsPage),
      },
      {
        path: '',
        redirectTo: 'home',
        pathMatch: 'full',
      },
    ],
  },
  {
    path: '',
    redirectTo: 'tabs/home',
    pathMatch: 'full',
  },
];
```

### Tabs Page Template

`src/app/tabs/tabs.page.html`:

```html
<ion-tabs>
  <ion-tab-bar slot="bottom">
    <ion-tab-button tab="home">
      <ion-icon name="home-outline"></ion-icon>
      <ion-label>Home</ion-label>
    </ion-tab-button>
    <ion-tab-button tab="settings">
      <ion-icon name="settings-outline"></ion-icon>
      <ion-label>Settings</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

The `tab` attribute on `<ion-tab-button>` must match the child route `path`.

### Tab Navigation Rules

- Each tab maintains its own navigation stack independently.
- Never navigate between tabs programmatically. Users switch tabs via the tab bar only.
- To share a view across tabs, use a modal instead of routing to another tab's page.

## Side Menu Navigation

### Route Configuration

```typescript
const routes: Routes = [
  {
    path: '',
    loadComponent: () => import('./menu/menu.page').then((m) => m.MenuPage),
    children: [
      {
        path: 'home',
        loadComponent: () => import('./home/home.page').then((m) => m.HomePage),
      },
      {
        path: 'profile',
        loadComponent: () => import('./profile/profile.page').then((m) => m.ProfilePage),
      },
      {
        path: '',
        redirectTo: 'home',
        pathMatch: 'full',
      },
    ],
  },
];
```

### Menu Page Template

`src/app/menu/menu.page.html`:

```html
<ion-menu contentId="main-content">
  <ion-header>
    <ion-toolbar>
      <ion-title>Menu</ion-title>
    </ion-toolbar>
  </ion-header>
  <ion-content>
    <ion-list>
      <ion-menu-toggle auto-hide="false">
        <ion-item routerLink="/home" routerDirection="root">
          <ion-icon name="home-outline" slot="start"></ion-icon>
          <ion-label>Home</ion-label>
        </ion-item>
      </ion-menu-toggle>
      <ion-menu-toggle auto-hide="false">
        <ion-item routerLink="/profile" routerDirection="root">
          <ion-icon name="person-outline" slot="start"></ion-icon>
          <ion-label>Profile</ion-label>
        </ion-item>
      </ion-menu-toggle>
    </ion-list>
  </ion-content>
</ion-menu>

<ion-router-outlet id="main-content"></ion-router-outlet>
```

Wrap each menu item in `<ion-menu-toggle>` so the menu closes after selection. Set `auto-hide="false"` to always show the toggle.

## Route Guards

Angular route guards work normally with Ionic routes. Use functional guards (Angular 15.2+):

```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }
  return router.createUrlTree(['/login']);
};
```

Apply the guard to a route:

```typescript
{
  path: 'profile',
  loadComponent: () => import('./profile/profile.page').then((m) => m.ProfilePage),
  canActivate: [authGuard],
}
```

## Modals

Open modals using `ModalController`:

```typescript
import { Component, inject } from '@angular/core';
import { ModalController } from '@ionic/angular';
import { EditModalComponent } from '../edit-modal/edit-modal.component';

@Component({ /* ... */ })
export class DetailPage {
  private modalCtrl = inject(ModalController);

  async openEditModal() {
    const modal = await this.modalCtrl.create({
      component: EditModalComponent,
      componentProps: {
        itemId: 42,
      },
    });
    await modal.present();

    const { data, role } = await modal.onDidDismiss();
    if (role === 'confirm') {
      // Handle confirmed data
    }
  }
}
```

Dismiss from inside the modal:

```typescript
import { Component, Input, inject } from '@angular/core';
import { ModalController } from '@ionic/angular';

@Component({ /* ... */ })
export class EditModalComponent {
  @Input() itemId!: number;
  private modalCtrl = inject(ModalController);

  confirm() {
    this.modalCtrl.dismiss({ saved: true }, 'confirm');
  }

  cancel() {
    this.modalCtrl.dismiss(null, 'cancel');
  }
}
```

For standalone, import `ModalController` from `@ionic/angular/standalone`.

## Linear vs. Non-Linear Routing

**Linear routing**: A predictable forward/back sequence (e.g., list -> detail -> edit). The back button always returns to the previous page. Works well with `NavController.navigateForward()` and `NavController.back()`.

**Non-linear routing**: Multiple independent navigation stacks (e.g., tabs). The back button returns within the current tab's stack, not the global history. Each tab maintains its own `ion-router-outlet`.

When the app uses tabs or split-pane layouts, use non-linear routing. For simple page flows, use linear routing.
