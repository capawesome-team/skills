# Standalone vs. NgModules

Ionic Angular supports two architectures for using components: standalone components (Ionic 7.5+) and NgModule-based imports.

## Detection

Check the project to determine which architecture is in use:

1. Read `src/main.ts`:
   - `bootstrapApplication(...)` indicates **standalone**.
   - `platformBrowserDynamic().bootstrapModule(...)` indicates **NgModule**.
2. Check for `src/app/app.module.ts`:
   - If it exists and imports `IonicModule`, the project uses **NgModule**.
   - If it does not exist, the project uses **standalone**.

## Standalone Architecture

### Setup

`src/main.ts`:

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, appConfig);
```

`src/app/app.config.ts`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideIonicAngular } from '@ionic/angular/standalone';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes), provideIonicAngular({})],
};
```

`provideIonicAngular({})` must always be called, even without custom configuration. It accepts an `IonicConfig` object for customization.

### Component Imports

Each standalone component must import the Ionic components it uses from `@ionic/angular/standalone`:

```typescript
import { Component } from '@angular/core';
import {
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonList,
  IonItem,
  IonLabel,
} from '@ionic/angular/standalone';

@Component({
  selector: 'app-home',
  template: `
    <ion-header>
      <ion-toolbar>
        <ion-title>Home</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list>
        <ion-item>
          <ion-label>Item 1</ion-label>
        </ion-item>
      </ion-list>
      <ion-button (click)="doSomething()">Click Me</ion-button>
    </ion-content>
  `,
  standalone: true,
  imports: [IonHeader, IonToolbar, IonTitle, IonContent, IonButton, IonList, IonItem, IonLabel],
})
export class HomePage {
  doSomething() {}
}
```

### Icon Registration

Icons must be registered with `addIcons()` from `ionicons`:

```typescript
import { Component } from '@angular/core';
import { IonIcon } from '@ionic/angular/standalone';
import { addIcons } from 'ionicons';
import { homeOutline, settingsOutline, personOutline } from 'ionicons/icons';

@Component({
  selector: 'app-menu',
  template: `
    <ion-icon name="home-outline"></ion-icon>
    <ion-icon name="settings-outline"></ion-icon>
  `,
  standalone: true,
  imports: [IonIcon],
})
export class MenuComponent {
  constructor() {
    addIcons({ homeOutline, settingsOutline, personOutline });
  }
}
```

The icon names in the template use kebab-case (`home-outline`), while the import names use camelCase (`homeOutline`). The `addIcons()` call maps them automatically.

### Controller Imports

Controllers (`ModalController`, `ActionSheetController`, `AlertController`, `LoadingController`, `PopoverController`, `ToastController`, `PickerController`) are imported from `@ionic/angular/standalone`:

```typescript
import { ModalController } from '@ionic/angular/standalone';
```

### Advantages

- Enables tree-shaking: the final bundle only includes Ionic components actually used.
- Supports Angular's ESBuild-based builder for faster builds.
- Aligns with Angular's recommended architecture (Angular 17+).

## NgModule Architecture

### Setup

`src/main.ts`:

```typescript
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```

`src/app/app.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { IonicModule } from '@ionic/angular';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, IonicModule.forRoot(), AppRoutingModule],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

`IonicModule.forRoot()` registers all Ionic components globally. No per-component imports needed.

### Page Modules

Each lazy-loaded page has its own module:

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { IonicModule } from '@ionic/angular';
import { FormsModule } from '@angular/forms';
import { HomePage } from './home.page';
import { HomePageRoutingModule } from './home-routing.module';

@NgModule({
  imports: [CommonModule, FormsModule, IonicModule, HomePageRoutingModule],
  declarations: [HomePage],
})
export class HomePageModule {}
```

### Advantages

- No manual per-component imports; all Ionic components are available everywhere.
- Works with older Angular versions (before Angular 17).

### Disadvantages

- No tree-shaking for Ionic components; all components are included in the bundle.
- Incompatible with ESBuild.
- More boilerplate (module files for every page).

## Migrating NgModules to Standalone

Ionic provides an automated migration schematic:

```bash
ng generate @ionic/angular-toolkit:standalone
```

This must be run as a single migration. Gradual (partial) migration is not supported.

The migration:
1. Updates `src/main.ts` to use `bootstrapApplication`.
2. Creates `src/app/app.config.ts` with `provideIonicAngular`.
3. Converts page modules to standalone components with explicit imports.
4. Removes `app.module.ts` and page module files.
5. Updates routing to use `loadComponent` instead of `loadChildren`.

After migration, verify:
- All Ionic component imports in standalone components reference `@ionic/angular/standalone`.
- Icons are registered with `addIcons()`.
- The app compiles and runs without errors.
