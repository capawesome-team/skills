# MLKit GenAI Prompt

Unofficial Capacitor plugin for the ML Kit GenAI Prompt API. Runs on-device text and multimodal prompts through Gemini Nano.

**Platforms:** Android

## Installation

```bash
npm install @capacitor-mlkit/genai-prompt
npx cap sync
```

## Configuration

### Android

Requires `minSdkVersion` 26 or higher in `android/variables.gradle`.

Optional `variables.gradle` variable: `mlkitGenaiPromptVersion` (default: `1.0.0-beta2`).

### iOS

Not supported. All methods reject as unavailable on iOS (and unimplemented on the Web).

## Usage

```typescript
import { GenAiPrompt, FeatureStatus } from '@capacitor-mlkit/genai-prompt';

const { featureStatus } = await GenAiPrompt.checkFeatureStatus();
if (featureStatus === FeatureStatus.Downloadable) {
  await GenAiPrompt.addListener('downloadProgress', (event) => console.log(event.totalBytesDownloaded));
  await GenAiPrompt.downloadFeature();
}

await GenAiPrompt.addListener('inferenceProgress', (event) => console.log(event.text));
const { text } = await GenAiPrompt.generateContent({ prompt: 'Write a short poem about the sea.' });
console.log(text);
```

## Notes

- The underlying ML Kit GenAI Prompt API is in **beta**.
- Powered by Gemini Nano via Android AICore. Requires a Gemini-Nano-capable device with a locked bootloader; unsupported devices report `FeatureStatus.Unavailable`.
- The feature model must be downloaded once via `downloadFeature()`; track progress with the `downloadProgress` event and stream partial output with `inferenceProgress`.
- Supports an optional `imagePath` for multimodal prompts. Prompts should stay under ~4,000 tokens; output is not moderated. Validated for English and Korean.
