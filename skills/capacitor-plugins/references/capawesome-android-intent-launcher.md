# Android Intent Launcher

Launch arbitrary Android intents and read their results.

**Package:** `@capawesome/capacitor-android-intent-launcher`
**Platforms:** Android

## Installation

```bash
npm install @capawesome/capacitor-android-intent-launcher
npx cap sync
```

This plugin is Android-only. On iOS and Web, all methods reject as unimplemented. No additional configuration is required.

## Usage

### Start an activity

```typescript
import { AndroidIntentLauncher } from '@capawesome/capacitor-android-intent-launcher';

const { resultCode, dataUri } = await AndroidIntentLauncher.startActivity({
  action: 'android.intent.action.VIEW',
  dataUri: 'https://capawesome.io',
});
// resultCode: -1 = RESULT_OK, 0 = RESULT_CANCELED, or a custom code.
```

### Attach extras, a MIME type, and target a specific component

```typescript
await AndroidIntentLauncher.startActivity({
  action: 'android.intent.action.SEND',
  type: 'text/plain',
  extras: { 'android.intent.extra.TEXT': 'Hello world!' },
});

// Explicit intent targeting a specific component.
await AndroidIntentLauncher.startActivity({
  packageName: 'com.example.app',
  className: 'com.example.app.MainActivity',
});
```

### Resolve an activity before launching

```typescript
const { canResolve } = await AndroidIntentLauncher.canResolveActivity({
  action: 'android.intent.action.VIEW',
  dataUri: 'https://capawesome.io',
});
```

## Notes

- `StartActivityOptions` fields: `action`, `categories` (string[]), `className`, `dataUri`, `extras` (only `string | number | boolean` values), `flags` (number, combine with bitwise OR), `packageName`, `type` (MIME type). `CanResolveActivityOptions` is the same shape.
- `startActivity(...)` uses `startActivityForResult(...)`, so it resolves once the launched activity finishes, returning `resultCode` and an optional `dataUri`.
- **Package visibility caveat**: On Android 11+ (API level 30+), package visibility restricts which apps and intents your app can see. Intents targeting other apps may need matching `<queries>` entries in your app's `AndroidManifest.xml`. Without them, `startActivity(...)` rejects with `ACTIVITY_NOT_FOUND` and `canResolveActivity(...)` returns `false`. The plugin cannot predeclare arbitrary intents on your behalf. Example:

  ```xml
  <manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <queries>
      <intent>
        <action android:name="android.intent.action.VIEW" />
        <data android:scheme="https" />
      </intent>
    </queries>
  </manifest>
  ```

- This is a last-resort escape hatch for system screens and app integrations that no dedicated plugin covers. Prefer a typed plugin where one exists (e.g. Settings Launcher, App Launcher).
- `ErrorCode` values: `ACTIVITY_NOT_FOUND`, `START_FAILED`.
