# Audio Session

Capacitor plugin to configure and observe the iOS [audio session](https://developer.apple.com/documentation/avfaudio/avaudiosession). Set the category, mode and options, activate/deactivate, route output, and observe interruption and route change events.

**Package:** `@capawesome/capacitor-audio-session`

**Platforms:** iOS

## Installation

```bash
npm install @capawesome/capacitor-audio-session
npx cap sync
```

All methods reject as unimplemented on Android and Web. No additional configuration is required.

## Usage

```typescript
import { AudioSession } from '@capawesome/capacitor-audio-session';

const configure = async () => {
  await AudioSession.configure({
    category: 'playback',
    mode: 'moviePlayback',
    options: {
      mixWithOthers: false,
    },
  });
};

const setActive = async () => {
  await AudioSession.setActive({ active: true });
};

const getCurrentOutputs = async () => {
  const { outputs } = await AudioSession.getCurrentOutputs();
  return outputs; // [{ portName, portType }]
};

const overrideOutput = async () => {
  await AudioSession.overrideOutput({ type: 'speaker' });
};

const addListeners = async () => {
  await AudioSession.addListener('interruption', event => {
    console.log('Interruption:', event.type, event.shouldResume);
  });
  await AudioSession.addListener('routeChange', event => {
    console.log('Route change:', event.reason, event.outputs);
  });
};
```

## Notes

- `category`: `'ambient' | 'multiRoute' | 'playAndRecord' | 'playback' | 'record' | 'soloAmbient'`.
- `mode`: `'default' | 'gameChat' | 'measurement' | 'moviePlayback' | 'spokenAudio' | 'videoChat' | 'videoRecording' | 'voiceChat' | 'voicePrompt'` (default `'default'`).
- `options` flags default to `false`: `allowAirPlay`, `allowBluetooth`, `allowBluetoothA2DP`, `defaultToSpeaker`, `duckOthers`, `interruptSpokenAudioAndMixWithOthers`, `mixWithOthers`.
- `overrideOutput({ type })` accepts `'default' | 'speaker'`.
- `setActive({ active, notifyOthersOnDeactivation })`; `notifyOthersOnDeactivation` defaults to `true`.
- `interruption` event: `type` is `'began' | 'ended'`; `shouldResume` is only `true` when `type` is `ended`.
- `routeChange` event `reason`: `'categoryChange' | 'newDeviceAvailable' | 'noSuitableRouteForCategory' | 'oldDeviceUnavailable' | 'override' | 'routeConfigurationChange' | 'unknown' | 'wakeFromSleep'`.
- The audio session is a single, app-wide object shared with other audio plugins (Audio Player, Audio Recorder, Speech Recognition) and the platform. The last write wins: `configure(...)` may override settings another plugin applied, and vice versa. Reconfigure when switching between playback, recording and other scenarios.
- Error codes: `ACTIVATION_FAILED`, `CONFIGURATION_FAILED`.
