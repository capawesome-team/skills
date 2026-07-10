# MLKit Object Detection

Unofficial Capacitor plugin for ML Kit Object Detection and Tracking. Detects and classifies objects in an image on the device.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/object-detection
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variable: `mlkitObjectDetectionVersion` (default: `17.0.2`).

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { ObjectDetection } from '@capacitor-mlkit/object-detection';

const { detectedObjects } = await ObjectDetection.processImage({
  path: 'path/to/image.jpg',
  shouldEnableClassification: true,
  shouldEnableMultipleObjects: true,
});
for (const object of detectedObjects) {
  console.log(object.boundingBox, object.trackingId, object.labels);
}
```

## Notes

- `processImage()` is only available on Android and iOS; the Web platform is not supported by the underlying ML Kit SDK.
- `shouldEnableClassification` (default `false`) adds coarse labels (`Fashion good`, `Food`, `Home good`, `Place`, `Plant`).
- `shouldEnableMultipleObjects` (default `false`) enables detection of up to five objects per image.
