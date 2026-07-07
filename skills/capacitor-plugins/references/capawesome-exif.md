# Exif

Read, write, and remove EXIF metadata from image files without re-encoding the pixel data.

**Package:** `@capawesome/capacitor-exif`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-exif
npx cap sync
```

On Web, all methods reject as unimplemented.

## Configuration

### Android

This plugin uses the following project variable (defined in your app's `variables.gradle`):

- `$androidxExifInterfaceVersion` version of `androidx.exifinterface:exifinterface` (default: `1.4.2`)

### iOS

No additional configuration is required.

## Usage

### Read EXIF tags

```typescript
import { Exif } from '@capawesome/capacitor-exif';

const { tags } = await Exif.readExif({ path: 'file:///path/to/photo.jpg' });
console.log(tags.gpsLatitude, tags.gpsLongitude, tags.make, tags.model);
// Every tag is optional and only present if it exists in the file.
```

### Write EXIF tags in place

```typescript
await Exif.writeExif({
  path: 'file:///path/to/photo.jpg',
  tags: {
    dateTimeOriginal: '2026:01:15 10:30:00', // EXIF format YYYY:MM:DD HH:MM:SS
    gpsLatitude: 51.503364, // Must be provided together with gpsLongitude.
    gpsLongitude: -0.127625,
  },
});
```

### Strip metadata before upload (privacy)

```typescript
// Removes GPS and all other EXIF metadata in place; keeps orientation by default.
await Exif.removeExif({ path: 'file:///path/to/photo.jpg' });
```

## Notes

- `path` accepts local file paths and `file://` URIs.
- **Lossless guarantee**: the pixel data is never decoded or re-encoded, so image quality is unaffected. On Android the `ExifInterface` library rewrites only the metadata segments; on iOS `CGImageDestinationCopyImageSource` copies the compressed data unchanged and replaces only the metadata.
- `writeExif(...)` updates only the provided tags; all other tags remain unchanged.
- `removeExif(...)` keeps the `orientation` tag by default (`keepOrientation: true`) so images are not suddenly displayed rotated.
- `gpsLatitude`/`gpsLongitude` are signed decimal degrees and must be written together.
- **Read** works for any platform-supported format. **Write & remove** are lossless in-place operations supported only per this matrix:

  | Format    | Read Android/iOS | Write & Remove Android | Write & Remove iOS |
  | --------- | ---------- | ---------------------- | ------------------ |
  | JPEG      | ✅ / ✅    | ✅                     | ✅                 |
  | PNG       | ✅ / ✅    | ✅                     | ❌                 |
  | WebP      | ✅ / ✅    | ✅                     | ❌                 |
  | HEIC/HEIF | ✅ / ✅    | ❌                     | ✅                 |
  | DNG/RAW   | ✅ / ✅    | ❌                     | ❌                 |

  Calling `writeExif(...)`/`removeExif(...)` on an unsupported format rejects with `UNSUPPORTED_FORMAT`.
- **HEIC strip caveats (iOS)**: HEIC stores orientation at the container level, so `orientation` cannot be changed and is always kept even with `keepOrientation: false`. HEIC files may retain `software` and `dateTimeOriginal` after `removeExif(...)`, but GPS and device-identity tags (`make`, `model`, `lensModel`) are always removed.
- **Android caveat**: vendor-specific maker notes and other unknown tags may survive `removeExif(...)`, since `ExifInterface` only removes known tags.
- `ErrorCode` values: `FILE_NOT_FOUND`, `READ_FAILED`, `UNSUPPORTED_FORMAT`, `WRITE_FAILED`.
