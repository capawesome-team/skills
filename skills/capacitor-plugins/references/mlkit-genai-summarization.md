# MLKit GenAI Summarization

Unofficial Capacitor plugin for ML Kit GenAI Summarization. Summarizes text into bullet points on the device via Gemini Nano.

**Platforms:** Android

## Installation

```bash
npm install @capacitor-mlkit/genai-summarization
npx cap sync
```

## Configuration

### Android

Requires `minSdkVersion` 26 or higher in `android/variables.gradle`.

Optional `variables.gradle` variable: `mlkitGenaiSummarizationVersion` (default: `1.0.0-beta1`).

### iOS

Not supported. All methods reject as unavailable on iOS (and unimplemented on the Web).

## Usage

```typescript
import { GenAiSummarization, FeatureStatus, InputType, Language, OutputType } from '@capacitor-mlkit/genai-summarization';

const options = { inputType: InputType.Article, language: Language.English, outputType: OutputType.ThreeBullets };

const { featureStatus } = await GenAiSummarization.checkFeatureStatus(options);
if (featureStatus === FeatureStatus.Downloadable) {
  await GenAiSummarization.addListener('downloadProgress', (event) => console.log(event.totalBytesDownloaded));
  await GenAiSummarization.downloadFeature(options);
}

await GenAiSummarization.addListener('inferenceProgress', (event) => console.log(event.text));
const { summary } = await GenAiSummarization.summarize({ text: '...', ...options });
console.log(summary);
```

## Notes

- The underlying ML Kit GenAI Summarization API is in **beta**.
- Powered by Gemini Nano via Android AICore. Requires a Gemini-Nano-capable device with a locked bootloader; unsupported devices report `FeatureStatus.Unavailable`.
- The feature model must be downloaded once via `downloadFeature()`; track progress with the `downloadProgress` event and stream partial output with `inferenceProgress`.
- `inputType` (`Article`/`Conversation`), `language` (`English`/`Japanese`/`Korean`), and `outputType` (one/two/three bullets) configure the summary. Input should stay under ~4,000 tokens.
