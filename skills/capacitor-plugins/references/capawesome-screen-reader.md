# Screen Reader

Post accessibility announcements to the active screen reader and detect whether one is enabled.

**Package:** `@capawesome/capacitor-screen-reader`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-screen-reader
npx cap sync
```

No additional permissions or configuration are required on any platform.

## Usage

### Announce a message

```typescript
import { ScreenReader } from '@capawesome/capacitor-screen-reader';

await ScreenReader.announce({
  value: 'The item was added to your cart.',
  language: 'en', // Only available on Android.
});
```

### Detect and observe the screen reader state

```typescript
import { ScreenReader } from '@capawesome/capacitor-screen-reader';

const { enabled } = await ScreenReader.isEnabled();

await ScreenReader.addListener('stateChange', ({ enabled }) => {
  console.log('Screen reader enabled:', enabled);
});
```

## Notes

- This does not perform text-to-speech. `announce(...)` posts an announcement that is only read out when a screen reader (VoiceOver/TalkBack) is active. For real text-to-speech, use the Speech Synthesis plugin.
- On Web, the announcement is made through a visually hidden `aria-live` region, so it is only read out if the user has a screen reader running. `isEnabled()` and `stateChange` are not available on Web (they reject as unimplemented).
- On Android, `isEnabled()` reflects whether touch exploration (TalkBack) is enabled; on iOS, whether VoiceOver is running.
- The device is only observed while at least one `stateChange` listener is attached.
- The `language` option (BCP 47 tag) helps the screen reader pronounce the message and is only available on Android.
