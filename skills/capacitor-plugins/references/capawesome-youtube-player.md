# YouTube Player

Embed and control inline YouTube players positioned by CSS pixel frames on Android, iOS and Web.

**Package:** `@capawesome/capacitor-youtube-player`
**Platforms:** Android, iOS, Web
**Documentation:** https://capawesome.io/docs/sdks/capacitor/youtube-player/

## Installation

```bash
npm install @capawesome/capacitor-youtube-player
npx cap sync
```

No API key is required — the plugin builds on the official YouTube IFrame Player API via open-source wrappers.

## Configuration

No plugin configuration is required.

### Android

Uses the [android-youtube-player](https://github.com/PierfrancescoSoffritti/android-youtube-player) library.

#### Variables

Optionally define in `android/variables.gradle`:

- `$androidYoutubePlayerVersion` version of `com.pierfrancescosoffritti.androidyoutubeplayer:core` (default: `13.0.0`)

#### Permissions

No action required. The plugin adds `android.permission.ACCESS_NETWORK_STATE` to the app manifest itself; it is used to recover the player when the network connection is lost and restored.

### iOS

Uses the [YoutubePlayerView](https://github.com/mukeshydv/YoutubePlayerView) library. Both Swift Package Manager and CocoaPods are supported. No additional configuration is required.

### Web

The YouTube IFrame Player API is loaded from YouTube at runtime, so an active network connection is required. If the app enforces a Content Security Policy, allow the YouTube domains (e.g. `https://www.youtube.com` for `script-src` and `frame-src`).

## Usage

### Create a player

Measure a placeholder element in the layout and create the player at its position:

```typescript
import { YoutubePlayer } from '@capawesome/capacitor-youtube-player';

const rect = document
  .querySelector('#player-placeholder')
  .getBoundingClientRect();

await YoutubePlayer.createPlayer({
  id: 'my-player',
  frame: { x: rect.x, y: rect.y, width: rect.width, height: rect.height },
  videoId: 'dQw4w9WgXcQ',
  options: { mute: true, controls: true, start: 30 },
});
```

### Keep the frame in sync

On Android and iOS the player is a native view rendered **above** the web view. It is not part of the DOM and does not scroll or resize with the web content:

```typescript
import { YoutubePlayer } from '@capawesome/capacitor-youtube-player';

const syncPlayerFrame = async () => {
  const rect = document
    .querySelector('#player-placeholder')
    .getBoundingClientRect();
  await YoutubePlayer.setPlayerFrame({
    id: 'my-player',
    frame: { x: rect.x, y: rect.y, width: rect.width, height: rect.height },
  });
};

window.addEventListener('scroll', syncPlayerFrame);
window.addEventListener('resize', syncPlayerFrame);
```

With a framework-owned scroll container (e.g. `ion-content`), listen to that container's scroll event instead.

### Control playback

```typescript
import { YoutubePlayer } from '@capawesome/capacitor-youtube-player';

await YoutubePlayer.play({ id: 'my-player' });
await YoutubePlayer.seekTo({ id: 'my-player', seconds: 42 });
await YoutubePlayer.setVolume({ id: 'my-player', volume: 50 });
await YoutubePlayer.setPlaybackRate({ id: 'my-player', rate: 1.5 });
await YoutubePlayer.pause({ id: 'my-player' });

const { currentTime } = await YoutubePlayer.getCurrentTime({ id: 'my-player' });
const { duration } = await YoutubePlayer.getDuration({ id: 'my-player' });

await YoutubePlayer.loadVideo({ id: 'my-player', videoId: 'dQw4w9WgXcQ' });
await YoutubePlayer.removePlayer({ id: 'my-player' });
```

### Listen for events

```typescript
import { YoutubePlayer } from '@capawesome/capacitor-youtube-player';

await YoutubePlayer.addListener('playerReady', (event) => {
  console.log('Player ready:', event.id);
});
await YoutubePlayer.addListener('playerStateChange', (event) => {
  console.log('State:', event.state); // 'unstarted' | 'cued' | 'buffering' | 'playing' | 'paused' | 'ended'
});
await YoutubePlayer.addListener('playerError', (event) => {
  console.error('Error:', event.code); // 'video-not-found' | 'not-embeddable' | ...
});
```

## Notes

- **Minimum player size**: frames must be at least 200×200 CSS pixels, as required by the YouTube Terms of Service. Smaller frames are rejected with the `FRAME_INVALID` error code.
- HTML content can never overlap the player, since the native view is always rendered above the web view. The YouTube Terms of Service also prohibit obscuring the player.
- Background playback is prohibited by the YouTube Terms of Service, so the plugin pauses all players when the app moves to the background.
- On Web, `web.elementId` mounts the player into an existing DOM element, in which case `setPlayerFrame(...)` has no effect and no frame synchronization is needed. On Web the player is otherwise positioned with `position: fixed`.
- Multiple players can exist at the same time; every method takes the player `id`. If `id` is omitted in `createPlayer(...)`, a random one is generated and returned.
- The `fullscreen` option and the `fullscreenChange` event are not available on iOS.
- On iOS, `mute()` is emulated by setting the volume to `0`; `unmute()` restores the last volume set via `setVolume(...)` (or `100`). A `setVolume(...)` call while muted is only applied on the next `unmute()`.
- On iOS, `playbackRateChange` is only emitted for `setPlaybackRate(...)` calls. On Android, `getCurrentTime(...)` and `getDuration(...)` are answered from the most recent values pushed by the player.
- `currentTimeChange` fires about 10 times per second on Android and about twice per second on iOS and Web.
- Player options: `autoplay`, `ccLoadPolicy`, `controls`, `end`, `fullscreen`, `ivLoadPolicy`, `mute`, `rel`, `start`. `rate` must be one of `0.25`, `0.5`, `0.75`, `1`, `1.25`, `1.5`, `1.75` or `2`; `volume` is `0`–`100`.
- Using the plugin means agreeing to the YouTube Terms of Service and the YouTube API Services Terms of Service. This project is not affiliated with, endorsed by, sponsored by, or approved by Google LLC or YouTube LLC.
