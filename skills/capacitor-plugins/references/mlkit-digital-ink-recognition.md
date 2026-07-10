# MLKit Digital Ink Recognition

Unofficial Capacitor plugin for ML Kit Digital Ink Recognition. Recognizes handwritten text and hand-drawn shapes from stroke coordinates on the device.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/digital-ink-recognition
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variable: `mlkitDigitalInkRecognitionVersion` (default: `19.0.0`).

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { DigitalInkRecognition } from '@capacitor-mlkit/digital-ink-recognition';
import type { Stroke } from '@capacitor-mlkit/digital-ink-recognition';

await DigitalInkRecognition.downloadModel({ languageTag: 'en-US' });

const strokes: Stroke[] = [{ points: [{ x: 100, y: 50, t: Date.now() }] }];

const { candidates } = await DigitalInkRecognition.recognize({
  languageTag: 'en-US',
  strokes,
  writingArea: { width: 300, height: 100 },
});
console.log(candidates[0]?.text);

const { languageTags } = await DigitalInkRecognition.getDownloadedModels();
await DigitalInkRecognition.deleteDownloadedModel({ languageTag: 'en-US' });
```

## Notes

- A per-language model must be downloaded before recognition. Languages are raw BCP-47 tags (e.g. `en-US`).
- Use the shape model tag `zxx-Zsym-x-autodraw` to recognize hand-drawn shapes, or `zxx-Zsym-x-emoji` for emojis.
- `recognize()` throws `ErrorCode.ModelNotDownloaded` if the model is missing and `ErrorCode.UnsupportedLanguageTag` for unknown tags.
