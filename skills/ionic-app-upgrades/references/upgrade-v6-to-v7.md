# Upgrade: Ionic 6 to 7

## Step 1: Update Framework Dependencies

Determine the framework in use and install the correct packages.

### Angular

Ionic 7 requires Angular 14+. Update Angular first using the [Angular Update Guide](https://angular.dev/update-guide), then install Ionic.

Ensure RxJS is at version 7.5.0 or higher:

```bash
npm install rxjs@7.5.0
```

Install Ionic 7:

```bash
npm install @ionic/angular@7
```

If using `@ionic/angular-server`:

```bash
npm install @ionic/angular-server@7
```

If using Angular 15+ and `@ionic/angular-toolkit`:

```bash
npm install @ionic/angular-toolkit@9
```

Note: `@ionic/angular-toolkit@9` requires Angular 15 as a minimum. If still on Angular 14, do not update the toolkit.

### React

Ionic 7 requires React 17+. Update React, then install Ionic:

```bash
npm install react@latest react-dom@latest
npm install @ionic/react@7 @ionic/react-router@7
```

### Vue

Ionic 7 requires Vue 3.0.6+. Update Vue and Vue Router, then install Ionic:

```bash
npm install vue@latest vue-router@latest
npm install @ionic/vue@7 @ionic/vue-router@7
```

### Core (Vanilla JS)

```bash
npm install @ionic/core@7
```

## Step 2: Update Browser Support

Update `browserslist` or `.browserslistrc`:

```
Chrome >=79
ChromeAndroid >=79
Firefox >=70
Edge >=79
Safari >=14
iOS >=14
```

## Step 3: Remove Deleted Types

The following attribute type interfaces have been removed. Replace all occurrences with `{ [key: string]: any }`:

- `ActionSheetAttributes`
- `AlertAttributes`
- `AlertTextareaAttributes`
- `AlertInputAttributes`
- `LoadingAttributes`
- `ModalAttributes`
- `PickerAttributes`
- `PopoverAttributes`
- `ToastAttributes`

```diff
-import { ModalAttributes } from '@ionic/angular';
-const attrs: ModalAttributes = { ... };
+const attrs: { [key: string]: any } = { ... };
```

## Step 4: Update Component Changes

### Checkbox

Rename CSS variables:

```diff
 ion-checkbox {
-  --background: #fff;
-  --background-checked: blue;
+  --checkbox-background: #fff;
+  --checkbox-background-checked: blue;
 }
```

### Datetime

- Remove empty string (`''`) assignments to the `value` property. Use `undefined` or remove the assignment.
- Remove code that accesses timezone information from the `value` property. `ion-datetime` does not manage time zones.

### Input

The `ionInput` event detail changed. Update event handlers to access `event.detail.value` instead of `event.detail`:

```diff
-handleInput(event: CustomEvent) {
-  const value = event.detail;
-}
+handleInput(event: CustomEvent) {
+  const value = event.detail.value;
+}
```

### Modal

- Remove `swipeToClose` property. Card modals are now swipeable by default.
- Remove explicit `canDismiss={undefined}` assignments. The property defaults to `true`.
- To prevent dismissal, set `canDismiss` to `false` or provide a callback function.

```diff
-<ion-modal swipeToClose={true} canDismiss={undefined}>
+<ion-modal>
```

### Picker

The `refresh` method on `ion-picker-column` has been removed. Use the `columns` property on `ion-picker` to refresh the view instead.

### Searchbar

Same as Input: update `ionInput` event handlers to use `event.detail.value`:

```diff
-handleSearch(event: CustomEvent) {
-  const value = event.detail;
-}
+handleSearch(event: CustomEvent) {
+  const value = event.detail.value;
+}
```

### Segment

Do not assign `null` to the `value` property. Use `''` or `undefined` instead:

```diff
-<ion-segment value={null}>
+<ion-segment value="">
```

### Slides

`ion-slides` and `ion-slide` have been **removed**. Migrate to [Swiper.js](https://swiperjs.com/) directly.

**Angular:** Follow the [Swiper Angular migration guide](https://ionicframework.com/docs/angular/slides).

**React:** Follow the [Swiper React migration guide](https://ionicframework.com/docs/react/slides).

**Vue:** Follow the [Swiper Vue migration guide](https://ionicframework.com/docs/vue/slides).

Remove all `ion-slides` and `ion-slide` imports and component usage. Replace with the Swiper equivalent for the framework in use.

### Textarea

Same as Input: update `ionInput` event handlers to use `event.detail.value`:

```diff
-handleTextarea(event: CustomEvent) {
-  const value = event.detail;
-}
+handleTextarea(event: CustomEvent) {
+  const value = event.detail.value;
+}
```

### Toggle

Rename CSS variables:

```diff
 ion-toggle {
-  --background: #ccc;
-  --background-checked: green;
+  --track-background: #ccc;
+  --track-background-checked: green;
 }
```

### Virtual Scroll

`ion-virtual-scroll` has been **removed**. Migrate to framework-native virtual scrolling solutions.

**Angular:** Use `@angular/cdk/scrolling` (`ScrollingModule` with `cdk-virtual-scroll-viewport`).

**React:** Use a library like `react-virtuoso` or `react-window`.

**Vue:** Use a library like `vue-virtual-scroller`.

Remove all `ion-virtual-scroll` imports and component usage.

## Step 5: Build and Test

```bash
npm run build
```

Fix any TypeScript or template errors caused by the changes above.

## Error Handling

* If `ion-slides` or `ion-slide` errors appear, the components were removed. Migrate to Swiper.js.
* If `ion-virtual-scroll` errors appear, the component was removed. Use a framework-native virtual scrolling solution.
* If `ionInput` event handlers return an object instead of a string, update handlers to read `event.detail.value` instead of `event.detail`.
* If checkbox or toggle styles break, rename the CSS variables (`--background` to `--checkbox-background` or `--track-background`).
* If `swipeToClose` causes a type error, remove the property entirely from modal usage.
* If attribute types (`ModalAttributes`, etc.) are not found, replace with `{ [key: string]: any }`.
