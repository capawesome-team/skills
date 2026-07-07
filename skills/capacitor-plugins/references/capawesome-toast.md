# Toast

Show a toast with a short text message.

**Package:** `@capawesome/capacitor-toast`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-toast
npx cap sync
```

No additional permissions or configuration are required on any platform. The Web implementation is self-contained with no additional peer dependencies.

## Usage

```typescript
import { Toast, ToastDuration, ToastPosition } from '@capawesome/capacitor-toast';

// Minimal toast.
await Toast.show({ text: 'Hello World!' });

// Toast with duration and position.
await Toast.show({
  text: 'Saved successfully.',
  duration: ToastDuration.Long,
  position: ToastPosition.Top,
});
```

## Notes

- `ToastDuration` values: `ToastDuration.Short` (default, ~2000 ms), `ToastDuration.Long` (~3500 ms).
- `ToastPosition` values: `ToastPosition.Bottom` (default), `ToastPosition.Center`, `ToastPosition.Top`.
- On Android 12 and newer, the operating system ignores the requested position and always shows toasts at the bottom of the screen. This is a system restriction, not a plugin bug.
