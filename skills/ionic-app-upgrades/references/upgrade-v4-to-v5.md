# Upgrade: Ionic 4 to 5

## Step 1: Update Framework Dependencies

Determine the framework in use and install the correct packages.

### Angular

```bash
npm install @ionic/angular@5 @ionic/angular-toolkit@4
```

### React

```bash
npm install @ionic/react@5 @ionic/react-router@5 ionicons@5
```

### Core (Stencil / Vanilla JS)

```bash
npm install @ionic/core@5
```

## Step 2: Update Ionicons

Ionic 5 ships with Ionicons 5. Update the Ionicons package:

```bash
npm install ionicons@5
```

If using Ionicons directly, review icon name changes. Many icons were renamed or removed:

- `md-*` and `ios-*` prefixed icons have been removed. Use the unprefixed name (e.g., `md-close` becomes `close`).
- Some icons were renamed entirely. Check the [Ionicons 5 changelog](https://github.com/ionic-team/ionicons/blob/main/BREAKING.md) for the full mapping.

## Step 3: Update CSS Utilities

### Colors

The `ion-color-*` CSS utility classes have been renamed. Update all occurrences:

```diff
-<div class="ion-color ion-color-primary">
+<div class="ion-color-primary">
```

### CSS Custom Properties

Several component CSS custom properties were renamed for consistency:

#### Button

The `--margin-*` and `--padding-*` CSS variables on `ion-button` have been removed. Use standard CSS `margin` and `padding` properties instead:

```diff
 ion-button {
-  --margin-start: 8px;
-  --padding-start: 16px;
+  margin-inline-start: 8px;
+  padding-inline-start: 16px;
 }
```

## Step 4: Update Component API Changes

### Action Sheet

- The `title` property now renders above the subtitle. No code change required unless relying on visual ordering.

### Card

- `ion-card-header` no longer applies padding by default. Add padding manually if needed:

```diff
-<ion-card-header>
+<ion-card-header style="padding: 16px;">
```

### Header / Toolbar

- The `no-border` attribute on `ion-header` has been replaced with the `ion-no-border` class:

```diff
-<ion-header no-border>
+<ion-header class="ion-no-border">
```

### Input

- The `placeholder-color` CSS variable on `ion-input` is removed. Use `--placeholder-color` instead:

```diff
 ion-input {
-  placeholder-color: gray;
+  --placeholder-color: gray;
 }
```

### List Header

- `ion-list-header` now requires an `ion-label` child:

```diff
-<ion-list-header>Header Text</ion-list-header>
+<ion-list-header>
+  <ion-label>Header Text</ion-label>
+</ion-list-header>
```

### Menu

- `ion-menu` `side` attribute now defaults to `start` instead of `left`. In LTR layouts this is the same. No changes needed unless relying on `left`/`right` values explicitly.

### Range

- `ion-range` now requires `ion-label` in the main slot or as a property:

```diff
-<ion-range>
-  <span slot="start">Low</span>
-</ion-range>
+<ion-range>
+  <ion-label>Range</ion-label>
+</ion-range>
```

### Searchbar

- The `showCancelButton` property now accepts a string value instead of boolean:

```diff
-<ion-searchbar showCancelButton></ion-searchbar>
+<ion-searchbar show-cancel-button="focus"></ion-searchbar>
```

Valid values: `"never"`, `"focus"`, `"always"`.

### Tabs

- `ion-tab-button` must now be placed inside `ion-tab-bar`. Direct children of `ion-tabs` that are tab buttons must be wrapped:

```diff
 <ion-tabs>
-  <ion-tab-button tab="home">Home</ion-tab-button>
+  <ion-tab-bar>
+    <ion-tab-button tab="home">Home</ion-tab-button>
+  </ion-tab-bar>
 </ion-tabs>
```

### Toast

- The `showCloseButton` and `closeButtonText` properties have been replaced with the `buttons` property:

```diff
-const toast = await toastController.create({
-  message: 'Hello',
-  showCloseButton: true,
-  closeButtonText: 'Dismiss'
-});
+const toast = await toastController.create({
+  message: 'Hello',
+  buttons: [{ text: 'Dismiss', role: 'cancel' }]
+});
```

## Step 5: Update Browser Support

Update `browserslist` or `.browserslistrc` in the project root:

```
Chrome >=60
Firefox >=63
Edge >=79
Safari >=13
iOS >=13
```

## Step 6: Build and Test

```bash
npm run build
```

Fix any TypeScript or template errors caused by the changes above.

## Error Handling

* If Ionicons do not render, verify the icon names were updated from the `md-*`/`ios-*` prefixed format to the unprefixed format.
* If CSS styling breaks, search for removed CSS variables (e.g., `--margin-*`, `--padding-*` on buttons) and replace with standard CSS properties.
* If `showCancelButton` on `ion-searchbar` causes a type error, change the value from a boolean to a string (`"focus"`, `"never"`, or `"always"`).
