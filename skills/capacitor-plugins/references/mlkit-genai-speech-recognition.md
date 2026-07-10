# MLKit GenAI Speech Recognition

Unofficial Capacitor plugin for ML Kit GenAI Speech Recognition. Transcribes speech from the microphone on the device via Gemini Nano.

**Platforms:** Android

## Installation

```bash
npm install @capacitor-mlkit/genai-speech-recognition
npx cap sync
```

## Configuration

### Android

Requires `minSdkVersion` 26 or higher in `android/variables.gradle` (basic mode requires API level 31+).

The `android.permission.RECORD_AUDIO` permission is declared automatically by the plugin — no manual `AndroidManifest.xml` entry is needed.

Optional `variables.gradle` variables:

- `mlkitGenaiSpeechRecognitionVersion` (default: `1.0.0-alpha1`)
- `kotlinVersion` (default: `2.2.20`)

### iOS

Not supported. All methods reject as unavailable on iOS (and unimplemented on the Web).

## Usage

```typescript
import { GenAiSpeechRecognition, FeatureStatus, RecognitionMode } from '@capacitor-mlkit/genai-speech-recognition';

const options = { locale: 'en-US', mode: RecognitionMode.Basic };

const { featureStatus } = await GenAiSpeechRecognition.checkFeatureStatus(options);
if (featureStatus === FeatureStatus.Downloadable) {
  await GenAiSpeechRecognition.addListener('downloadProgress', (event) => console.log(event.totalBytesDownloaded));
  await GenAiSpeechRecognition.downloadFeature(options);
}

await GenAiSpeechRecognition.addListener('partialResult', (event) => console.log('partial', event.text));
await GenAiSpeechRecognition.addListener('finalResult', (event) => console.log('final', event.text));
await GenAiSpeechRecognition.addListener('error', (event) => console.error(event.message));

await GenAiSpeechRecognition.startRecognition(options); // auto-requests microphone permission
// later:
await GenAiSpeechRecognition.stopRecognition();
```

## Notes

- The underlying ML Kit GenAI Speech Recognition API is in **alpha**; the API surface and behavior may change with breaking changes.
- Powered by Gemini Nano via Android AICore. Requires a Gemini-Nano-capable device with a locked bootloader; unsupported devices report `FeatureStatus.Unavailable`.
- `RecognitionMode.Advanced` (GenAI model) is limited to a few devices (e.g. Pixel 10); `RecognitionMode.Basic` uses the traditional model and needs API level 31+.
- `startRecognition()` auto-requests the microphone permission; you can also manage it via `checkPermissions()` / `requestPermissions()`. Results stream through `partialResult` / `finalResult`, and failures through the `error` event.
