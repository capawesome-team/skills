# Silent Mode

Detect whether the device is in silent mode and read the ringer mode.

**Package:** `@capawesome/capacitor-silent-mode`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-silent-mode
npx cap sync
```

## Usage

### Check silent mode and ringer mode

```typescript
import { SilentMode } from '@capawesome/capacitor-silent-mode';

const { silent } = await SilentMode.isSilent();

const { mode } = await SilentMode.getRingerMode(); // Android only: 'normal' | 'silent' | 'vibrate'
```

### Listen for silent mode changes

```typescript
import { SilentMode } from '@capawesome/capacitor-silent-mode';

await SilentMode.addListener('silentModeChange', event => {
  console.log('Silent mode changed:', event.silent);
});

await SilentMode.removeAllListeners();
```

## Notes

- **Android**: The silent state is derived from the ringer mode. The device is considered silent whenever the ringer mode is not `normal`, so **vibrate mode also counts as silent**. Use `getRingerMode()` to distinguish between `vibrate` and `silent`.
- `getRingerMode()` is only available on Android.
- **iOS**: There is no public API to read the ring/silent switch. The plugin uses a heuristic that plays a short muted system sound and measures how long it takes to complete, so detection may be inaccurate while other audio is playing. The `silentModeChange` listener polls this heuristic on a timer while the app is in the foreground and pauses in the background.
- The device is only observed while at least one listener is attached.
- On Web, all methods reject as unimplemented.
