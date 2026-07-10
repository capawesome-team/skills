# MLKit Pose Detection

Unofficial Capacitor plugin for ML Kit Pose Detection. Detects the pose of a person in an image as 33 skeletal landmarks on the device.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/pose-detection
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variables:

- `mlkitPoseDetectionVersion` (default: `18.0.0-beta5`)
- `mlkitPoseDetectionAccurateVersion` (default: `18.0.0-beta5`)

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { PoseDetection, PerformanceMode } from '@capacitor-mlkit/pose-detection';

const { poses } = await PoseDetection.processImage({
  path: 'path/to/image.jpg',
  performanceMode: PerformanceMode.Base,
});
for (const pose of poses) {
  console.log(pose.landmarks); // 33 landmarks
}
```

## Notes

- The underlying ML Kit Pose Detection API is in **beta**.
- `processImage()` is only available on Android and iOS; the Web platform is not supported by the underlying ML Kit SDK.
- Returns at most one pose per image. `PerformanceMode.Accurate` is slower but more precise than `PerformanceMode.Base` (default).
