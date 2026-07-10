# MLKit Text Recognition

Unofficial Capacitor plugin for ML Kit Text Recognition. Recognizes text in an image on the device, grouped into blocks, lines, and elements.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/text-recognition
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variables (default `16.0.1` each):

- `mlkitTextRecognitionVersion` (Latin)
- `mlkitTextRecognitionChineseVersion`
- `mlkitTextRecognitionDevanagariVersion`
- `mlkitTextRecognitionJapaneseVersion`
- `mlkitTextRecognitionKoreanVersion`

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { TextRecognition, Script } from '@capacitor-mlkit/text-recognition';

const { text, blocks } = await TextRecognition.processImage({
  path: 'path/to/image.jpg',
  script: Script.Latin,
});
console.log(text, blocks);
```

## Notes

- `processImage()` is only available on Android and iOS; the Web platform is not supported by the underlying ML Kit SDK.
- Supported scripts: `Latin`, `Chinese`, `Devanagari`, `Japanese`, `Korean` (default `Latin`).
- On Android, including additional script models increases the app binary size; only include the script dependencies you need.
