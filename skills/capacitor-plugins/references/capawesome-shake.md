# Shake

Detect physical shake gestures on the device.

**Package:** `@capawesome/capacitor-shake`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-shake
npx cap sync
```

## Usage

### Watch for shake gestures

```typescript
import { Shake } from '@capawesome/capacitor-shake';

await Shake.addListener('shake', () => {
  console.log('Shake detected!');
});

await Shake.startWatching({ sensitivity: 'medium' });
```

### Stop watching

```typescript
import { Shake } from '@capawesome/capacitor-shake';

await Shake.stopWatching();
await Shake.removeAllListeners();
```

## Notes

- The `shake` event is only emitted between `startWatching()` and `stopWatching()`. The sensor is only active while watching, keeping it battery-friendly.
- `sensitivity` controls how strong a shake must be (default: `'medium'`):
  - `'light'`: a gentle shake is enough.
  - `'medium'`: a moderate shake is required.
  - `'hard'`: only a strong shake triggers an event.
- On Web, all methods reject as unimplemented.
