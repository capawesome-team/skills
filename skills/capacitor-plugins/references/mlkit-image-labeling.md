# MLKit Image Labeling

Unofficial Capacitor plugin for ML Kit Image Labeling. Detects labels for objects, places, and activities in an image on the device.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/image-labeling
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variable: `mlkitImageLabelingVersion` (default: `17.0.9`).

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { ImageLabeling } from '@capacitor-mlkit/image-labeling';

const { labels } = await ImageLabeling.processImage({
  path: 'path/to/image.jpg',
  confidenceThreshold: 0.5,
});
for (const label of labels) {
  console.log(label.text, label.confidence);
}
```

## Notes

- `processImage()` is only available on Android and iOS; the Web platform is not supported by the underlying ML Kit SDK.
- `confidenceThreshold` (default `0.5`) filters out labels below the given confidence.
- Uses the default on-device model with 400+ labels; no model download required.
