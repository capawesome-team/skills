# Volume

Control the volume and observe hardware volume button presses.

**Package:** `@capawesome/capacitor-volume`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-volume
npx cap sync
```

## Usage

### Get and set the volume

```typescript
import { Volume, VolumeStream } from '@capawesome/capacitor-volume';

const { volume } = await Volume.getVolume(); // 0 - 1
const { volume: ringVolume } = await Volume.getVolume({ stream: VolumeStream.Ring }); // Android only

await Volume.setVolume({ volume: 0.5 });
```

### Watch hardware volume buttons

```typescript
import { Volume } from '@capawesome/capacitor-volume';

await Volume.addListener('volumeButtonPressed', event => {
  console.log('Volume button pressed:', event.direction); // 'up' | 'down'
});

await Volume.addListener('volumeChange', event => {
  console.log('Volume changed:', event.volume);
});

await Volume.startWatching({ suppressVolumeChange: true });

const { watching } = await Volume.isWatching();

await Volume.stopWatching();
await Volume.removeAllListeners();
```

## Notes

- The volume level is a value between `0` and `1`. The `volumeButtonPressed` and `volumeChange` events are only emitted while watching.
- **Android**: The volume can be read and set per audio stream via the `stream` option (`VolumeStream.Music` by default; also `Alarm`, `Notification`, `Ring`, `System`, `VoiceCall`). If Do Not Disturb is active, changing the ring, notification, or system stream requires **Do Not Disturb access**; otherwise the call rejects with the `DO_NOT_DISTURB_ACCESS_REQUIRED` error code. Direct the user to the settings screen via the `ACTION_NOTIFICATION_POLICY_ACCESS_SETTINGS` intent.
- **iOS**: There is no public API to set the system volume directly. The plugin uses a hidden system volume view to change the **media volume**, which is the only volume that can be read and set. The `stream` option is ignored.
- **Button watching**: On both platforms, presses are only detected while the app is in the foreground. `suppressVolumeChange` keeps the volume unchanged and hides the system volume indicator while watching (default: `false`). On iOS, the volume is reset after each press (nudged away from the min/max edges if needed) and the original volume is restored when watching stops; volume changes from other sources (e.g. Control Center) are also reported.
- On Web, all methods reject as unimplemented.
