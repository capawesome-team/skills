# Photo Manipulator

Headless image transforms like crop, resize, rotate, flip and format conversion, including HEIC/AVIF to JPEG.

**Package:** `@capawesome/capacitor-photo-manipulator`
**Platforms:** Android, iOS, Web (partial)

## Installation

```bash
npm install @capawesome/capacitor-photo-manipulator
npx cap sync
```

### Android

This plugin will use the following project variables (defined in your app's `variables.gradle` file):

- `$androidxExifInterfaceVersion` version of `androidx.exifinterface:exifinterface` (default: `1.4.1`)

## Usage

### Convert HEIC to JPEG

```typescript
import { PhotoManipulator, ImageFormat } from '@capawesome/capacitor-photo-manipulator';

const { path } = await PhotoManipulator.transform({
  path: 'file:///var/mobile/.../photo.heic',
  format: ImageFormat.Jpeg,
  quality: 0.9,
});
```

### Crop, resize, rotate and flip in one call

```typescript
import { PhotoManipulator } from '@capawesome/capacitor-photo-manipulator';

// Operations are always applied in a fixed order: crop -> resize -> rotate -> flip.
const { path, width, height } = await PhotoManipulator.transform({
  path: 'file:///var/mobile/.../photo.jpeg',
  crop: { x: 100, y: 100, width: 1080, height: 1080 },
  resize: { width: 256 }, // Aspect ratio preserved when only one dimension is given.
  rotate: 90, // Must be 90, 180 or 270.
  flipHorizontal: true,
});
```

### Read image info without decoding pixels

```typescript
import { PhotoManipulator } from '@capawesome/capacitor-photo-manipulator';

const { width, height, format } = await PhotoManipulator.getInfo({
  path: 'file:///var/mobile/.../photo.heic',
});
```

## Notes

- `path` accepts local file paths and `file://` URIs on Android and iOS; on the web any fetchable URL (`https://`, `blob:`, `data:`) works.
- Output is written to a **new file** in the cache directory and **deleted on the next app launch**. Move it to a permanent location to keep it (e.g. via the Filesystem plugin's `rename(...)`).
- The `crop → resize → rotate → flip` order is fixed. Chain multiple `transform(...)` calls if you need a different order.
- **Format enum:** `ImageFormat.Jpeg` (`'JPEG'`), `ImageFormat.Png` (`'PNG'`), `ImageFormat.Webp` (`'WEBP'`). Default is `ImageFormat.Jpeg`. `quality` (0–1, default `0.9`) only applies to JPEG and WebP.
- **Decode support by platform:** JPEG/PNG/WebP/GIF/BMP work everywhere. HEIC/HEIF: Android 9+ and iOS (not web). AVIF: Android 12+ and iOS 16+ (browser-dependent on web).
- **Encode support:** JPEG/PNG work everywhere. **WebP output is not supported on iOS** and rejects with the `UNSUPPORTED_FORMAT` error code (browser-dependent on web).
- Any unsupported decode/encode is rejected with `UNSUPPORTED_FORMAT`; other error codes are `FILE_NOT_FOUND` and `TRANSFORM_FAILED`. Branch on these to fall back gracefully.
- **Memory:** Providing a `resize` target enables downsampled decoding (`inSampleSize` on Android, `kCGImageSourceThumbnailMaxPixelSize` on iOS), so the full-resolution bitmap is never loaded. Always pass `resize` when you don't need full resolution. Transforms run one at a time on a background queue.
- **Metadata:** The source EXIF orientation is applied during decoding so output is always upright; all other metadata (EXIF, GPS) is stripped by re-encoding.
- On the web this is a partial Canvas-based implementation; HEIC/AVIF generally cannot be decoded in browsers, which is why the native implementations exist.
