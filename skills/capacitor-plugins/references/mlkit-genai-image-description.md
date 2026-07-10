# MLKit GenAI Image Description

Unofficial Capacitor plugin for ML Kit GenAI Image Description. Generates a natural-language description of an image on the device via Gemini Nano.

**Platforms:** Android

## Installation

```bash
npm install @capacitor-mlkit/genai-image-description
npx cap sync
```

## Configuration

### Android

Requires `minSdkVersion` 26 or higher in `android/variables.gradle`.

Optional `variables.gradle` variable: `mlkitGenaiImageDescriptionVersion` (default: `1.0.0-beta1`).

### iOS

Not supported. All methods reject as unavailable on iOS (and unimplemented on the Web).

## Usage

```typescript
import { GenAiImageDescription, FeatureStatus } from '@capacitor-mlkit/genai-image-description';

const { featureStatus } = await GenAiImageDescription.checkFeatureStatus();
if (featureStatus === FeatureStatus.Downloadable) {
  await GenAiImageDescription.addListener('downloadProgress', (event) => console.log(event.totalBytesDownloaded));
  await GenAiImageDescription.downloadFeature();
}

await GenAiImageDescription.addListener('inferenceProgress', (event) => console.log(event.text));
const { description } = await GenAiImageDescription.describeImage({ path: 'file:///path/to/image.jpg' });
console.log(description);
```

## Notes

- The underlying ML Kit GenAI Image Description API is in **beta**.
- Powered by Gemini Nano via Android AICore. Requires a Gemini-Nano-capable device with a locked bootloader; unsupported devices report `FeatureStatus.Unavailable`.
- The feature model must be downloaded once via `downloadFeature()`; track progress with the `downloadProgress` event and stream partial output with `inferenceProgress`.
- Descriptions are only available in English.
