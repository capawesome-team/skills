# Dialog

Display native alert, confirm, and prompt dialogs.

**Package:** `@capawesome/capacitor-dialog`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-dialog
npx cap sync
```

No additional permissions or configuration are required on any platform.

## Usage

```typescript
import { Dialog } from '@capawesome/capacitor-dialog';

// Alert: single button.
await Dialog.alert({
  title: 'Success',
  message: 'Your changes have been saved.',
  buttonTitle: 'Got it', // Default: 'OK'.
});

// Confirm: OK / Cancel.
const { value } = await Dialog.confirm({
  title: 'Confirm',
  message: 'Do you want to delete this item?',
  okButtonTitle: 'Yes', // Default: 'OK'.
  cancelButtonTitle: 'No', // Default: 'Cancel'.
});
// value: true if confirmed
```

```typescript
// Prompt: text input with confirm and cancel.
const { value, canceled } = await Dialog.prompt({
  title: 'Name',
  message: 'What is your name?',
  inputPlaceholder: 'Enter your name',
  inputText: 'John Doe', // Initial value.
});
// value: the entered text; canceled: true if dismissed via the cancel button
```

## Notes

- On Web, dialogs use the browser's built-in `alert`/`confirm`/`prompt`; titles, button titles, and the input placeholder cannot be customized and are ignored.
- `message` is required for all three methods.
