# Action Sheet

Present a native action sheet with a list of buttons.

**Package:** `@capawesome/capacitor-action-sheet`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-action-sheet
npx cap sync
```

## Configuration

### Android

#### Variables

This plugin uses the following project variable (defined in your app's `android/variables.gradle` file):

- `$androidxMaterialVersion` version of `com.google.android.material:material` (default: `1.12.0`)

## Usage

```typescript
import { ActionSheet, ActionSheetButtonStyle } from '@capawesome/capacitor-action-sheet';

const { index, canceled } = await ActionSheet.showActions({
  title: 'Photo Options',
  message: 'Select an option to perform.',
  options: [
    { title: 'Upload' },
    { title: 'Share' },
    { title: 'Delete', style: ActionSheetButtonStyle.Destructive },
    { title: 'Cancel', style: ActionSheetButtonStyle.Cancel },
  ],
  cancelable: true, // Dismiss by tapping outside or pressing back (Android). Default: true.
});
// index: zero-based index of the selected button, or -1 if canceled without a cancel button
// canceled: true if dismissed or a Cancel-style button was selected
```

`ActionSheetButtonStyle` values: `DEFAULT`, `DESTRUCTIVE`, `CANCEL`.

## Notes

- Only available on Android and iOS; there is no Web implementation.
- The `Cancel`-style button should be the last entry in the `options` array.
- On iPad the action sheet is automatically anchored as a popover.
- On iOS the sheet can always be dismissed by tapping outside (a system behavior); `cancelable` only controls the outside-tap/back-button dismissal on Android.
- API-compatible with `@capacitor/action-sheet` (same enum values), plus a `canceled` result field, a `cancelable` option, and rendered `message`/button styles on Android.
