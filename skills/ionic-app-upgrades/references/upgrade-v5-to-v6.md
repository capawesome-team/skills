# Upgrade: Ionic 5 to 6

## Step 1: Update Framework Dependencies

Determine the framework in use and install the correct packages.

### Angular

Ionic 6 requires Angular 12+. Update Angular first using the [Angular Update Guide](https://angular.dev/update-guide), then install Ionic:

```bash
npm install @ionic/angular@6
```

If using `@ionic/angular-server`:

```bash
npm install @ionic/angular-server@6
```

#### Remove `Config.set()` and `setupConfig`

`Config.set()` and `setupConfig` have been removed. Configure Ionic via `IonicModule.forRoot()` in the app module:

```diff
 // app.module.ts
-import { IonicModule } from '@ionic/angular';
-import { setupConfig } from '@ionic/angular';
-
-setupConfig({ mode: 'ios' });

 @NgModule({
   imports: [
-    IonicModule.forRoot()
+    IonicModule.forRoot({ mode: 'ios' })
   ]
 })
```

### React

Ionic 6 requires React 17+. Update React first, then install Ionic:

```bash
npm install react@latest react-dom@latest
npm install @ionic/react@6 @ionic/react-router@6
```

#### Replace `setupConfig` with `setupIonicReact`

```diff
-import { setupConfig } from '@ionic/react';
-setupConfig({ mode: 'ios' });
+import { setupIonicReact } from '@ionic/react';
+setupIonicReact({ mode: 'ios' });
```

#### Update controller imports

```diff
-import { menuController } from '@ionic/core';
+import { menuController } from '@ionic/core/components';
```

### Vue

Ionic 6 requires Vue 3.0.6+. Update Vue and Vue Router, then install Ionic:

```bash
npm install vue@3 vue-router@4
npm install @ionic/vue@6 @ionic/vue-router@6
```

#### Remove `setupConfig`

Configure during plugin installation instead:

```diff
 // main.ts
-import { IonicVue, setupConfig } from '@ionic/vue';
-setupConfig({ mode: 'ios' });
-app.use(IonicVue);
+import { IonicVue } from '@ionic/vue';
+app.use(IonicVue, { mode: 'ios' });
```

#### Rename types

```diff
-import { IonRouter } from '@ionic/vue';
+import { UseIonRouterResult } from '@ionic/vue';
```

```diff
-import { IonKeyboardRef } from '@ionic/vue';
+import { UseKeyboardResult } from '@ionic/vue';
```

#### Update overlay event listeners

Remove the `on` prefix from all overlay event listeners:

```diff
-<ion-modal @onWillPresent="handlePresent">
+<ion-modal @willPresent="handlePresent">
```

```diff
-<ion-modal @onDidPresent="handlePresent">
+<ion-modal @didPresent="handlePresent">
```

```diff
-<ion-modal @onWillDismiss="handleDismiss">
+<ion-modal @willDismiss="handleDismiss">
```

```diff
-<ion-modal @onDidDismiss="handleDismiss">
+<ion-modal @didDismiss="handleDismiss">
```

This applies to all overlay components: `ion-modal`, `ion-action-sheet`, `ion-alert`, `ion-loading`, `ion-picker`, `ion-popover`, `ion-toast`.

#### Add `ion-router-outlet` to `ion-tabs`

```diff
 <ion-tabs>
   <ion-tab-bar slot="bottom">
     <ion-tab-button tab="home">Home</ion-tab-button>
   </ion-tab-bar>
+  <ion-router-outlet></ion-router-outlet>
 </ion-tabs>
```

### Core (Vanilla JS)

```bash
npm install @ionic/core@6
```

## Step 2: Update Component API Changes

### Datetime

The `ion-datetime` component has been completely redesigned. Remove the following properties:

- `placeholder`
- `pickerOptions`
- `pickerFormat`
- `monthNames`, `monthShortNames`, `dayNames`, `dayShortNames`
- `displayFormat`, `displayTimezone`

Remove the `text` and `placeholder` CSS shadow parts.

Remove these CSS variables: `--padding-bottom`, `--padding-end`, `--padding-start`, `--padding-top`, `--placeholder-color`.

Remove `open()` method calls. Place `ion-datetime` inside `ion-modal` or `ion-popover` instead:

```diff
-<ion-datetime placeholder="Select date"></ion-datetime>
+<ion-modal>
+  <ion-datetime></ion-datetime>
+</ion-modal>
```

### Modal

`ion-modal` now uses Shadow DOM. Update custom styles to use CSS variables or `::part()` selectors:

```diff
-ion-modal .modal-wrapper {
-  background: white;
-}
-ion-modal ion-backdrop {
-  opacity: 0.5;
-}
+ion-modal::part(content) {
+  background: white;
+}
+ion-modal::part(backdrop) {
+  opacity: 0.5;
+}
```

### Popover

`ion-popover` now uses Shadow DOM. Update custom styles:

```diff
-ion-popover .popover-arrow {
-  display: block;
-}
-ion-popover ion-backdrop {
-  opacity: 0.3;
-}
-ion-popover .popover-content {
-  border-radius: 8px;
-}
+ion-popover::part(arrow) {
+  display: block;
+}
+ion-popover::part(backdrop) {
+  opacity: 0.3;
+}
+ion-popover::part(content) {
+  border-radius: 8px;
+}
```

### Input, Select, Textarea

Do not pass `null` to the `placeholder` property. Use `undefined` instead:

```diff
-<ion-input placeholder={null}></ion-input>
+<ion-input placeholder={undefined}></ion-input>
```

### Radio

The `RadioChangeEventDetail` interface has been removed. Use `RadioGroupChangeEventDetail` instead if needed.

### Icon

Ionic 6 ships with Ionicons 6. Review the [Ionicons 6 breaking changes](https://github.com/ionic-team/ionicons/blob/main/BREAKING.md) for icon name changes.

## Step 3: Update Browser Support

Update `browserslist` or `.browserslistrc`:

```
Chrome >=60
Firefox >=63
Edge >=79
Safari >=13
iOS >=13
```

## Step 4: Update Jest / Testing Configuration

Ionic 6 ships as ES Modules. Jest requires configuration to handle them. Add `transformIgnorePatterns` to the Jest configuration (`jest.config.js` or `package.json`):

**Angular projects:**

```json
{
  "transformIgnorePatterns": ["/node_modules/(?!@ionic/core|@stencil/core|ionicons)"]
}
```

**React projects (react-scripts test):**

Update the `test` script in `package.json`:

```diff
-"test": "react-scripts test"
+"test": "react-scripts test --transformIgnorePatterns 'node_modules/(?!(@ionic/react|@ionic/react-router|@ionic/core|@stencil/core|ionicons)/)'"
```

**Vue projects:**

```json
{
  "transformIgnorePatterns": ["/node_modules/(?!@ionic/vue|@ionic/vue-router|@ionic/core|@stencil/core|ionicons)"]
}
```

## Step 5: Build and Test

```bash
npm run build
```

Fix any TypeScript or template errors caused by the changes above.

## Error Handling

* If `Config.set()` or `setupConfig` errors appear, replace them with the framework-specific configuration approach documented above.
* If modal or popover styles break, migrate CSS selectors from class-based selectors to `::part()` selectors.
* If Jest tests fail with `SyntaxError: Cannot use import statement outside a module`, add the `transformIgnorePatterns` configuration shown above.
* If overlay event listeners stop firing in Vue, remove the `on` prefix (e.g., `@onWillPresent` becomes `@willPresent`).
* If datetime picker no longer works, the component was redesigned. Place `ion-datetime` inside `ion-modal` or `ion-popover`.
