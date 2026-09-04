# Option Picker

Let the user pick an option from a list using a native picker: a wheel picker on iOS and a Material 3 dialog on Android.

**Package:** `@capawesome/capacitor-option-picker`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/option-picker/

## Installation

```bash
npm install @capawesome/capacitor-option-picker
npx cap sync
```

## Configuration

### Android

#### Variables

This plugin uses the following project variable (defined in your app's `android/variables.gradle` file):

- `$androidxMaterialVersion` version of `com.google.android.material:material` (default: `1.12.0`)

### iOS

No configuration is required.

## Usage

### Present a picker

```typescript
import { OptionPicker } from '@capawesome/capacitor-option-picker';

const { value } = await OptionPicker.present({
  title: 'Select a country',
  options: [
    { label: 'Germany', value: 'de' },
    { label: 'France', value: 'fr' },
    { label: 'Spain', value: 'es' },
  ],
  value: 'fr', // Preselected option. Defaults to the first option.
  theme: 'auto', // 'auto' | 'light' | 'dark'. Default: 'auto' (follows the system appearance).
  cancelButtonText: 'Cancel', // Default: 'Cancel'
  doneButtonText: 'Ok', // Default: 'Ok'
});
```

### Handle cancellation

```typescript
import { ErrorCode, OptionPicker } from '@capawesome/capacitor-option-picker';

try {
  const { value } = await OptionPicker.present({
    options: [
      { label: 'Small', value: 's' },
      { label: 'Medium', value: 'm' },
      { label: 'Large', value: 'l' },
    ],
  });
} catch (error) {
  if (error.code === ErrorCode.Canceled) {
    // The user tapped Cancel, tapped outside the picker or pressed the back button (Android).
  }
}
```

## Notes

- Only available on Android and iOS; there is no Web implementation. Use a regular HTML `<select>` element on the Web.
- Cancelling or dismissing the picker rejects the promise with the `CANCELED` error code. There is no `canceled` result flag.
- The option whose `value` matches the `value` option is preselected. If none matches, the first option is selected.
- Calling `present(...)` while a picker is already shown rejects with "A picker is already presented."
- Android: the dialog inherits the app theme if it is a Material Components or Material 3 theme. With an AppCompat theme (the Capacitor default) it uses a Material 3 theme with the system's dynamic colors on Android 12+; switch to a Material 3 app theme such as `Theme.Material3.DayNight.NoActionBar` to get brand colors.
- iOS: the picker is a `UIPickerView` in a bottom sheet with Cancel and Done buttons in the app's tint color.
- Use the Action Sheet plugin when the user chooses between a few actions; use the Option Picker when the user chooses a value, especially from a long list.
- `ErrorCode` values: `CANCELED`.
