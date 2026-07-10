# MLKit GenAI Proofreading

Unofficial Capacitor plugin for ML Kit GenAI Proofreading. Corrects grammar and spelling in short text on the device via Gemini Nano.

**Platforms:** Android

## Installation

```bash
npm install @capacitor-mlkit/genai-proofreading
npx cap sync
```

## Configuration

### Android

Requires `minSdkVersion` 26 or higher in `android/variables.gradle`.

Optional `variables.gradle` variable: `mlkitGenaiProofreadingVersion` (default: `1.0.0-beta1`).

### iOS

Not supported. All methods reject as unavailable on iOS (and unimplemented on the Web).

## Usage

```typescript
import { GenAiProofreading, FeatureStatus, InputType, Language } from '@capacitor-mlkit/genai-proofreading';

const options = { inputType: InputType.Keyboard, language: Language.English };

const { featureStatus } = await GenAiProofreading.checkFeatureStatus(options);
if (featureStatus === FeatureStatus.Downloadable) {
  await GenAiProofreading.addListener('downloadProgress', (event) => console.log(event.totalBytesDownloaded));
  await GenAiProofreading.downloadFeature(options);
}

await GenAiProofreading.addListener('inferenceProgress', (event) => console.log(event.text));
const { results } = await GenAiProofreading.proofread({ text: 'Capacitor is an open source natvie runtime.', ...options });
console.log(results); // sorted by confidence, highest first
```

## Notes

- The underlying ML Kit GenAI Proofreading API is in **beta**.
- Powered by Gemini Nano via Android AICore. Requires a Gemini-Nano-capable device with a locked bootloader; unsupported devices report `FeatureStatus.Unavailable`.
- The feature model must be downloaded once via `downloadFeature()`; track progress with the `downloadProgress` event and stream partial output with `inferenceProgress`.
- `inputType` (`Keyboard`/`Voice`) and `language` (`English`, `French`, `German`, `Italian`, `Japanese`, `Korean`, `Spanish`) configure proofreading.
