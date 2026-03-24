---
name: capacitor-angular
description: "Guides the agent through Angular-specific patterns for Capacitor app development. Covers project structure, adding Capacitor to Angular projects, using Capacitor plugins in Angular services and components, NgZone integration for plugin event listeners, lifecycle hook patterns, dependency injection, routing with deep links, and environment-based platform detection. Do not use for creating a new Capacitor app from scratch, upgrading Capacitor versions, installing specific plugins, Ionic Framework setup, or non-Angular frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-angular
---

# Capacitor with Angular

Angular-specific patterns and best practices for Capacitor app development — project structure, services, lifecycle hooks, NgZone integration, and plugin usage.

## Prerequisites

1. **Capacitor 6, 7, or 8** app with Angular 16+.
2. Node.js and npm installed.
3. Angular CLI installed (`npm install -g @angular/cli`).
4. For iOS: Xcode installed.
5. For Android: Android Studio installed.

## Agent Behavior

- **Auto-detect before asking.** Check the project for `angular.json`, `package.json`, `capacitor.config.ts` or `capacitor.config.json`, and existing directory structure. Only ask the user when something cannot be detected.
- **Guide step-by-step.** Walk the user through the process one step at a time.
- **Adapt to project style.** Detect whether the project uses standalone components or NgModule-based architecture and adapt code examples accordingly.

## Procedures

### Step 1: Analyze the Project

Auto-detect the following by reading project files:

1. **Angular version**: Read `@angular/core` version from `package.json`.
2. **Capacitor version**: Read `@capacitor/core` version from `package.json`. If not present, Capacitor has not been added yet — proceed to Step 2.
3. **Architecture style**: Check `src/main.ts` for `bootstrapApplication` (standalone) vs. `platformBrowserDynamic().bootstrapModule` (NgModule). Check `angular.json` for further confirmation.
4. **Platforms**: Check which directories exist (`android/`, `ios/`).
5. **Capacitor config format**: Check for `capacitor.config.ts` (TypeScript) or `capacitor.config.json` (JSON).
6. **Build output directory**: Read `outputPath` from `angular.json` under `projects > <project-name> > architect > build > options > outputPath`. This is needed for Capacitor's `webDir` setting.

### Step 2: Add Capacitor to an Angular Project

Skip if `@capacitor/core` is already in `package.json`.

1. Install Capacitor core and CLI:

   ```bash
   npm install @capacitor/core
   npm install -D @capacitor/cli
   ```

2. Initialize Capacitor:

   ```bash
   npx cap init
   ```

   When prompted, set the **web directory** to the Angular build output path detected in Step 1. For Angular 17+ with the application builder, this is typically `dist/<project-name>/browser`. For older Angular versions, it is typically `dist/<project-name>`.

3. Verify the `webDir` value in the generated `capacitor.config.ts` or `capacitor.config.json` matches the Angular build output path. If incorrect, update it:

   **`capacitor.config.ts`:**
   ```typescript
   import type { CapacitorConfig } from '@capacitor/cli';

   const config: CapacitorConfig = {
     appId: 'com.example.app',
     appName: 'my-app',
     webDir: 'dist/my-app/browser',
   };

   export default config;
   ```

   **`capacitor.config.json`:**
   ```json
   {
     "appId": "com.example.app",
     "appName": "my-app",
     "webDir": "dist/my-app/browser"
   }
   ```

4. Build the Angular app and add platforms:

   ```bash
   ng build
   npm install @capacitor/android @capacitor/ios
   npx cap add android
   npx cap add ios
   npx cap sync
   ```

### Step 3: Project Structure

A Capacitor Angular project has this structure:

```
my-app/
├── android/                  # Android native project
├── ios/                      # iOS native project
├── src/
│   ├── app/
│   │   ├── app.component.ts
│   │   ├── app.config.ts     # Standalone: app configuration
│   │   ├── app.module.ts     # NgModule: root module
│   │   ├── app.routes.ts     # Routing configuration
│   │   └── services/         # Angular services for Capacitor plugins
│   ├── environments/
│   │   ├── environment.ts
│   │   └── environment.prod.ts
│   ├── index.html
│   └── main.ts
├── angular.json
├── capacitor.config.ts       # or capacitor.config.json
├── package.json
└── tsconfig.json
```

Key points:
- The `android/` and `ios/` directories contain native projects and should be committed to version control.
- The `src/` directory contains the Angular app, which is the web layer of the Capacitor app.
- Capacitor plugins are called from Angular services or components inside `src/app/`.

### Step 4: Using Capacitor Plugins in Angular

Capacitor plugins are plain TypeScript APIs. Import and call them directly in Angular components or services.

#### Direct Usage in a Component

```typescript
import { Component } from '@angular/core';
import { Geolocation } from '@capacitor/geolocation';

@Component({
  selector: 'app-location',
  template: `
    <div>
      <p>Latitude: {{ latitude }}</p>
      <p>Longitude: {{ longitude }}</p>
      <button (click)="getCurrentPosition()">Get Location</button>
    </div>
  `,
  standalone: true,
})
export class LocationComponent {
  latitude: number | null = null;
  longitude: number | null = null;

  async getCurrentPosition() {
    const position = await Geolocation.getCurrentPosition();
    this.latitude = position.coords.latitude;
    this.longitude = position.coords.longitude;
  }
}
```

#### Wrapping Plugins in Angular Services (Recommended)

Wrapping Capacitor plugins in Angular services provides dependency injection, testability, and a single place to handle platform differences:

```typescript
import { Injectable } from '@angular/core';
import { Camera, CameraResultType, CameraSource, Photo } from '@capacitor/camera';
import { Capacitor } from '@capacitor/core';

@Injectable({
  providedIn: 'root',
})
export class CameraService {
  async takePhoto(): Promise<Photo> {
    return Camera.getPhoto({
      quality: 90,
      allowEditing: false,
      resultType: CameraResultType.Uri,
      source: CameraSource.Camera,
    });
  }

  async pickFromGallery(): Promise<Photo> {
    return Camera.getPhoto({
      quality: 90,
      allowEditing: false,
      resultType: CameraResultType.Uri,
      source: CameraSource.Photos,
    });
  }

  isNativePlatform(): boolean {
    return Capacitor.isNativePlatform();
  }
}
```

Use the service in a component:

```typescript
import { Component, inject } from '@angular/core';
import { CameraService } from '../services/camera.service';

@Component({
  selector: 'app-photo',
  template: `
    <button (click)="takePhoto()">Take Photo</button>
    <img *ngIf="photoUrl" [src]="photoUrl" alt="Captured photo" />
  `,
  standalone: true,
})
export class PhotoComponent {
  private cameraService = inject(CameraService);
  photoUrl: string | null = null;

  async takePhoto() {
    const photo = await this.cameraService.takePhoto();
    this.photoUrl = photo.webPath ?? null;
  }
}
```

### Step 5: NgZone Integration for Plugin Event Listeners

Capacitor plugin event listeners run **outside** Angular's `NgZone` execution context. When a plugin listener updates component state, Angular's change detection does **not** automatically trigger. Wrap the handler logic in `NgZone.run()` to fix this.

**Without NgZone (broken — UI does not update):**

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Network, ConnectionStatus } from '@capacitor/network';
import { PluginListenerHandle } from '@capacitor/core';

@Component({
  selector: 'app-network',
  template: `<p>Status: {{ networkStatus }}</p>`,
  standalone: true,
})
export class NetworkComponent implements OnInit, OnDestroy {
  networkStatus = 'Unknown';
  private listenerHandle: PluginListenerHandle | null = null;

  async ngOnInit() {
    // BUG: This callback runs outside NgZone — the template will not update.
    this.listenerHandle = await Network.addListener('networkStatusChange', (status) => {
      this.networkStatus = status.connected ? 'Online' : 'Offline';
    });
  }

  async ngOnDestroy() {
    await this.listenerHandle?.remove();
  }
}
```

**With NgZone (correct — UI updates properly):**

```typescript
import { Component, NgZone, OnInit, OnDestroy, inject } from '@angular/core';
import { Network, ConnectionStatus } from '@capacitor/network';
import { PluginListenerHandle } from '@capacitor/core';

@Component({
  selector: 'app-network',
  template: `<p>Status: {{ networkStatus }}</p>`,
  standalone: true,
})
export class NetworkComponent implements OnInit, OnDestroy {
  private ngZone = inject(NgZone);
  networkStatus = 'Unknown';
  private listenerHandle: PluginListenerHandle | null = null;

  async ngOnInit() {
    this.listenerHandle = await Network.addListener('networkStatusChange', (status) => {
      this.ngZone.run(() => {
        this.networkStatus = status.connected ? 'Online' : 'Offline';
      });
    });
  }

  async ngOnDestroy() {
    await this.listenerHandle?.remove();
  }
}
```

**Rule:** Always use `NgZone.run()` inside Capacitor plugin event listener callbacks that update component or service state bound to templates.

### Step 6: Lifecycle Hook Patterns

Use Angular lifecycle hooks to manage Capacitor plugin listeners. Register listeners in `ngOnInit` and remove them in `ngOnDestroy` to prevent memory leaks.

#### Service-Based Listener Management

For app-wide listeners (e.g., network status, app state), use a service initialized at app startup:

```typescript
import { Injectable, NgZone, OnDestroy, inject } from '@angular/core';
import { App } from '@capacitor/app';
import { PluginListenerHandle } from '@capacitor/core';
import { BehaviorSubject } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class AppStateService implements OnDestroy {
  private ngZone = inject(NgZone);
  private listenerHandle: PluginListenerHandle | null = null;

  private isActiveSubject = new BehaviorSubject<boolean>(true);
  isActive$ = this.isActiveSubject.asObservable();

  constructor() {
    this.initListener();
  }

  private async initListener() {
    this.listenerHandle = await App.addListener('appStateChange', (state) => {
      this.ngZone.run(() => {
        this.isActiveSubject.next(state.isActive);
      });
    });
  }

  async ngOnDestroy() {
    await this.listenerHandle?.remove();
  }
}
```

Initialize the service at app startup to ensure it runs immediately. In **standalone** apps, use `APP_INITIALIZER` or inject it in the root component. In **NgModule** apps, inject it in `AppComponent`:

**Standalone (`app.config.ts`):**

```typescript
import { ApplicationConfig, APP_INITIALIZER } from '@angular/core';
import { AppStateService } from './services/app-state.service';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: (appStateService: AppStateService) => () => {},
      deps: [AppStateService],
      multi: true,
    },
  ],
};
```

**NgModule (`app.component.ts`):**

```typescript
import { Component } from '@angular/core';
import { AppStateService } from './services/app-state.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent {
  constructor(private appStateService: AppStateService) {}
}
```

### Step 7: Platform Detection

Use `Capacitor.isNativePlatform()` and `Capacitor.getPlatform()` to conditionally run native-only code:

```typescript
import { Injectable } from '@angular/core';
import { Capacitor } from '@capacitor/core';

@Injectable({
  providedIn: 'root',
})
export class PlatformService {
  isNative(): boolean {
    return Capacitor.isNativePlatform();
  }

  getPlatform(): 'web' | 'ios' | 'android' {
    return Capacitor.getPlatform() as 'web' | 'ios' | 'android';
  }

  isIos(): boolean {
    return Capacitor.getPlatform() === 'ios';
  }

  isAndroid(): boolean {
    return Capacitor.getPlatform() === 'android';
  }

  isWeb(): boolean {
    return Capacitor.getPlatform() === 'web';
  }
}
```

Use it in components to show/hide native-only features:

```typescript
import { Component, inject } from '@angular/core';
import { PlatformService } from '../services/platform.service';

@Component({
  selector: 'app-settings',
  template: `
    @if (platformService.isNative()) {
      <button (click)="openNativeSettings()">Open Device Settings</button>
    }
  `,
  standalone: true,
})
export class SettingsComponent {
  platformService = inject(PlatformService);

  openNativeSettings() {
    // Native-only logic
  }
}
```

### Step 8: Deep Link Routing

Handle deep links by mapping Capacitor's `App.addListener('appUrlOpen', ...)` event to Angular Router navigation:

```typescript
import { Injectable, NgZone, inject } from '@angular/core';
import { Router } from '@angular/router';
import { App } from '@capacitor/app';

@Injectable({
  providedIn: 'root',
})
export class DeepLinkService {
  private ngZone = inject(NgZone);
  private router = inject(Router);

  constructor() {
    this.initDeepLinkListener();
  }

  private async initDeepLinkListener() {
    await App.addListener('appUrlOpen', (event) => {
      this.ngZone.run(() => {
        const url = new URL(event.url);
        const path = url.pathname;

        // Navigate to the route matching the deep link path.
        // Adjust the path parsing logic to match the app's URL scheme.
        if (path) {
          this.router.navigateByUrl(path);
        }
      });
    });
  }
}
```

Initialize `DeepLinkService` at app startup (same pattern as Step 6 — via `APP_INITIALIZER` or root component injection).

### Step 9: Back Button Handling (Android)

Handle the Android hardware back button using `App.addListener('backButton', ...)`:

```typescript
import { Injectable, NgZone, inject } from '@angular/core';
import { Location } from '@angular/common';
import { App } from '@capacitor/app';
import { Capacitor } from '@capacitor/core';

@Injectable({
  providedIn: 'root',
})
export class BackButtonService {
  private ngZone = inject(NgZone);
  private location = inject(Location);

  constructor() {
    if (Capacitor.getPlatform() === 'android') {
      this.initBackButtonListener();
    }
  }

  private async initBackButtonListener() {
    await App.addListener('backButton', ({ canGoBack }) => {
      this.ngZone.run(() => {
        if (canGoBack) {
          this.location.back();
        } else {
          App.exitApp();
        }
      });
    });
  }
}
```

### Step 10: Build and Sync Workflow

After making changes to the Angular app, build and sync to native platforms:

```bash
ng build
npx cap sync
```

To run on a device or emulator:

```bash
npx cap run android
npx cap run ios
```

To open the native IDE for advanced configuration or debugging:

```bash
npx cap open android
npx cap open ios
```

For live reload during development:

```bash
npx cap run android --livereload --external
npx cap run ios --livereload --external
```

This starts `ng serve` internally and configures the native app to load from the development server.

## Error Handling

- **UI not updating from plugin listeners**: Wrap the listener callback body in `NgZone.run(() => { ... })`. This is the most common Angular-specific issue with Capacitor.
- **`webDir` mismatch**: If `npx cap sync` copies the wrong files, verify that `webDir` in `capacitor.config.ts` or `capacitor.config.json` matches the Angular build output path. For Angular 17+ with the application builder, the path is `dist/<project-name>/browser`. For older Angular versions, it is `dist/<project-name>`.
- **Plugin not found at runtime**: Run `npx cap sync` after installing any new plugin. Verify the plugin appears in `package.json` dependencies.
- **Memory leaks from listeners**: Always remove plugin listeners in `ngOnDestroy`. Store the `PluginListenerHandle` returned by `addListener` and call `handle.remove()` on destroy.
- **Deep links not working**: Verify the app URL scheme / universal links are configured in the native projects (`android/app/src/main/AndroidManifest.xml` for Android, `ios/App/App/Info.plist` and associated domain entitlement for iOS). Verify `DeepLinkService` is initialized at app startup.
- **Back button closes app unexpectedly**: Ensure the back button listener checks `canGoBack` before calling `App.exitApp()`. Only exit when there is no navigation history.
- **Build output empty after `ng build`**: Verify the `outputPath` in `angular.json` is correct. For Angular 17+, the default changed to `dist/<project-name>/browser` with the application builder.

## Related Skills

- **`capacitor-app-creation`** — Create a new Capacitor app from scratch.
- **`capacitor-app-development`** — General Capacitor development guidance not specific to Angular.
- **`capacitor-plugins`** — Install and configure Capacitor plugins from official and community sources.
- **`capacitor-react`** — React-specific patterns and best practices for Capacitor app development.
- **`ionic-angular`** — Ionic Framework with Angular (UI components, navigation, theming on top of Capacitor).
- **`capacitor-app-upgrades`** — Upgrade a Capacitor app to a newer major version.
