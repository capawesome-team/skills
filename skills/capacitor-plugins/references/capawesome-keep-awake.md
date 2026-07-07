# Keep Awake

Keep the screen awake to prevent it from dimming or turning off.

**Package:** `@capawesome/capacitor-keep-awake`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-keep-awake
npx cap sync
```

No additional permissions or configuration are required on any platform.

## Usage

```typescript
import { KeepAwake } from '@capawesome/capacitor-keep-awake';

// Keep the screen awake.
await KeepAwake.keepAwake();

// Restore the default sleep behavior.
await KeepAwake.allowSleep();

// Check state and availability.
const { keptAwake } = await KeepAwake.isKeptAwake();
const { available } = await KeepAwake.isAvailable();
```

## Notes

- Keeping the screen awake only affects your app, not a system-wide setting.
- The screen is kept awake until `allowSleep()` is called or the app is restarted.
- On Web, the plugin uses the [Screen Wake Lock API](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API). `isAvailable()` depends on browser support; on Android and iOS it always returns `true`.
- On Web, the browser automatically releases the wake lock when the tab becomes hidden. The plugin re-acquires it when the tab becomes visible again, unless `allowSleep()` was called in the meantime.
