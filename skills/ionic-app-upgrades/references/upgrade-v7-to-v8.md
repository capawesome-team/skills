# Upgrade: Ionic 7 to 8

## Step 1: Update Framework Dependencies

Determine the framework in use and install the correct packages.

### Angular

Ionic 8 requires Angular 16+. Update Angular first using the [Angular Update Guide](https://angular.dev/update-guide), then install Ionic:

```bash
npm install @ionic/angular@latest
```

If using `@ionic/angular-server` and `@ionic/angular-toolkit`:

```bash
npm install @ionic/angular-server@latest @ionic/angular-toolkit@11
```

Note: `@ionic/angular-toolkit@11` requires Angular 17 as a minimum. If on Angular 16, use `@ionic/angular-toolkit@10`.

#### Update import for `IonBackButtonDelegate`

```diff
-import { IonBackButtonDelegate } from '@ionic/angular';
+import { IonBackButton } from '@ionic/angular';
```

### React

Ionic 8 requires React 17+. Install Ionic:

```bash
npm install @ionic/react@8 @ionic/react-router@8
```

### Vue

Ionic 8 requires Vue 3.0.6+. Install Ionic:

```bash
npm install @ionic/vue@8 @ionic/vue-router@8
```

### Core (Vanilla JS)

```bash
npm install @ionic/core@8
```

## Step 2: Update Browser Support

Update `browserslist` or `.browserslistrc`:

```
Chrome >=89
ChromeAndroid >=89
Firefox >=75
Edge >=89
Safari >=15
iOS >=15
```

## Step 3: Migrate Form Controls (Required)

Legacy form control syntax has been removed. Migrate **all** instances of `ion-checkbox`, `ion-input`, `ion-radio`, `ion-select`, `ion-textarea`, and `ion-toggle` to the modern syntax.

Remove the `legacy` property from all form control components if present:

```diff
-<ion-input legacy="true" label="Name"></ion-input>
+<ion-input label="Name"></ion-input>
```

Ensure all form controls use the modern label pattern (label as a property or slot) rather than wrapping in `ion-item` with a separate `ion-label`:

```diff
-<ion-item>
-  <ion-label>Name</ion-label>
-  <ion-input></ion-input>
-</ion-item>
+<ion-item>
+  <ion-input label="Name"></ion-input>
+</ion-item>
```

Or with the label slot:

```diff
-<ion-item>
-  <ion-label>Name</ion-label>
-  <ion-input></ion-input>
-</ion-item>
+<ion-item>
+  <ion-input>
+    <div slot="label">Name</div>
+  </ion-input>
+</ion-item>
```

Apply this pattern to all six form control components.

## Step 4: Update Component Changes

### Input

Remove the `size` and `accept` properties. Use CSS for input width instead of `size`:

```diff
-<ion-input size="20" accept="image/*"></ion-input>
+<ion-input style="width: 200px;"></ion-input>
```

### Item

Remove the following properties from `ion-item`. These have been moved to `ion-input` and `ion-textarea`:

- `counter` — use `counter` on `ion-input` or `ion-textarea` instead
- `counterFormatter` — use `counterFormatter` on `ion-input` or `ion-textarea` instead
- `helper` — use `helperText` on `ion-input` or `ion-textarea` instead
- `error` — use `errorText` on `ion-input` or `ion-textarea` instead
- `fill` — use `fill` on `ion-input`, `ion-textarea`, or `ion-select` instead
- `shape` — use `shape` on `ion-input`, `ion-textarea`, or `ion-select` instead

```diff
-<ion-item fill="outline" shape="round" counter="true" helper="Enter your name" error="Name is required">
-  <ion-label>Name</ion-label>
-  <ion-input></ion-input>
-</ion-item>
+<ion-item>
+  <ion-input
+    label="Name"
+    fill="outline"
+    shape="round"
+    counter="true"
+    helperText="Enter your name"
+    errorText="Name is required"
+  ></ion-input>
+</ion-item>
```

### Nav

The `getLength()` method now returns `Promise<number>` instead of `number`. Await the call:

```diff
-const length = nav.getLength();
+const length = await nav.getLength();
```

### Picker

The legacy `ion-picker` has been moved to `ion-picker-legacy`. A new inline `ion-picker` component is available.

If using the legacy picker, update the component name:

```diff
-<ion-picker></ion-picker>
+<ion-picker-legacy></ion-picker-legacy>
```

For controller-based usage:

```diff
-import { PickerController } from '@ionic/angular';
+import { PickerLegacyController } from '@ionic/angular';
```

Note: `ion-picker-legacy` will be removed in a future release. Plan to migrate to the new inline `ion-picker` component.

### Toast

Remove `cssClass` from `ToastButton`. Use the `button` CSS Shadow Part instead:

```diff
-const toast = await toastController.create({
-  buttons: [{ text: 'OK', cssClass: 'custom-button' }]
-});
+const toast = await toastController.create({
+  buttons: [{ text: 'OK' }]
+});
```

```css
ion-toast::part(button) {
  /* custom styles here */
}
```

## Step 5: Apply Recommended Changes (Optional)

These changes are recommended but not strictly required:

### Light Palette

Remove default color variables from `src/theme/variables.scss` that duplicate the built-in palette. These are now imported automatically via `core.css`. Keep only custom color overrides.

### Dark Palette

Replace manual dark palette definitions with imported CSS files.

**Angular — in `src/global.scss`:**

```diff
+@import '@ionic/angular/css/palettes/dark.system.css';
```

Update any dark palette selectors from `body` to `:root`:

```diff
-body.dark {
+:root.dark {
   --ion-color-primary: #428cff;
 }
```

### Step Color Tokens

The unified `--ion-color-step-*` tokens have been split into separate background and text tokens:

For background usage, rename `--ion-color-step-[number]` to `--ion-background-color-step-[number]`.

For text usage, rename `--ion-color-step-[number]` to `--ion-text-color-step-[number]` **and subtract the number from 1000**:

```diff
-:root {
-  --ion-color-step-400: #666;
-}
+:root {
+  --ion-text-color-step-600: #666;
+}
```

### Dynamic Font

The `--ion-default-dynamic-font` variable has been replaced with `--ion-dynamic-font`. Dynamic font scaling is enabled by default. To disable:

```css
:root {
  --ion-dynamic-font: initial;
}
```

### Angular CSS Import Order

In `angular.json`, reorder style imports so `global.scss` comes before `variables.scss`:

```diff
 "styles": [
-  "src/theme/variables.scss",
-  "src/global.scss"
+  "src/global.scss",
+  "src/theme/variables.scss"
 ]
```

## Step 6: Build and Test

```bash
npm run build
```

Fix any TypeScript or template errors caused by the changes above.

## Error Handling

* If form controls render incorrectly or show errors about `legacy` property, complete the migration to modern form control syntax as described in Step 3.
* If `ion-item` property errors appear (`fill`, `shape`, `counter`, etc.), move these properties to the inner `ion-input`, `ion-textarea`, or `ion-select` component.
* If `getLength()` returns a Promise unexpectedly, add `await` to the call.
* If `ion-picker` behaves differently, rename to `ion-picker-legacy` to retain old behavior temporarily.
* If dark mode styles break, check that dark palette selectors target `:root` instead of `body`.
* If step color tokens appear wrong, update them to the new split background/text tokens with the 1000-complement calculation for text.
