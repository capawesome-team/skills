# MLKit GenAI Rewriting

Unofficial Capacitor plugin for ML Kit GenAI Rewriting. Rewrites text in a different style or tone on the device via Gemini Nano.

**Platforms:** Android

## Installation

```bash
npm install @capacitor-mlkit/genai-rewriting
npx cap sync
```

## Configuration

### Android

Requires `minSdkVersion` 26 or higher in `android/variables.gradle`.

Optional `variables.gradle` variable: `mlkitGenaiRewritingVersion` (default: `1.0.0-beta1`).

### iOS

Not supported. All methods reject as unavailable on iOS (and unimplemented on the Web).

## Usage

```typescript
import { GenAiRewriting, FeatureStatus, Language, OutputType } from '@capacitor-mlkit/genai-rewriting';

const options = { language: Language.English, outputType: OutputType.Professional };

const { featureStatus } = await GenAiRewriting.checkFeatureStatus(options);
if (featureStatus === FeatureStatus.Downloadable) {
  await GenAiRewriting.addListener('downloadProgress', (event) => console.log(event.totalBytesDownloaded));
  await GenAiRewriting.downloadFeature(options);
}

await GenAiRewriting.addListener('inferenceProgress', (event) => console.log(event.text));
const { results } = await GenAiRewriting.rewrite({ text: 'hey can we meet later', ...options });
console.log(results); // sorted by confidence, highest first
```

## Notes

- The underlying ML Kit GenAI Rewriting API is in **beta**.
- Powered by Gemini Nano via Android AICore. Requires a Gemini-Nano-capable device with a locked bootloader; unsupported devices report `FeatureStatus.Unavailable`.
- The feature model must be downloaded once via `downloadFeature()`; track progress with the `downloadProgress` event and stream partial output with `inferenceProgress`.
- `outputType` styles: `Elaborate`, `Emojify`, `Shorten`, `Friendly`, `Professional`, `Rephrase`. Input should stay under ~256 tokens.
