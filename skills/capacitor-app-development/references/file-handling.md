# File Handling Best Practices

Handle files efficiently in Capacitor apps without causing memory issues.

## The Core Problem

Loading files into the WebView as base64 strings or data URLs consumes excessive memory and can cause out-of-memory crashes, especially with large files (images, videos, PDFs). The WebView's JavaScript heap has limited memory.

## Best Practices

### Reading Files

**Do not** use the Filesystem plugin's `readFile()` to load large files as base64:

```typescript
// BAD — loads entire file into WebView memory as base64
const result = await Filesystem.readFile({ path: 'photo.jpg', directory: Directory.Data });
const base64 = result.data; // Large string in JS heap
```

**Instead**, use `Capacitor.convertFileSrc()` with the Fetch API to load files as blobs:

```typescript
import { Capacitor } from '@capacitor/core';
import { Filesystem, Directory } from '@capacitor/filesystem';

// Get the native file URI
const file = await Filesystem.getUri({ path: 'photo.jpg', directory: Directory.Data });

// Convert to a web-accessible URL
const webUrl = Capacitor.convertFileSrc(file.uri);

// Load as a blob (memory-efficient — uses disk backing automatically)
const response = await fetch(webUrl);
const blob = await response.blob();
```

### Displaying Files

Convert native file paths to web-accessible URLs using `Capacitor.convertFileSrc()`, then use as `src` attributes:

```typescript
import { Capacitor } from '@capacitor/core';

const webUrl = Capacitor.convertFileSrc(nativeFilePath);

// Set as image source
const img = document.querySelector('img');
img.src = webUrl;

// Or in a framework template
// Angular: <img [src]="webUrl">
// React: <img src={webUrl} />
// Vue: <img :src="webUrl">
```

This avoids loading the file content into JavaScript memory entirely.

### Downloading Files

Use the Filesystem plugin's `downloadFile()` to write directly to device storage:

```typescript
import { Filesystem, Directory } from '@capacitor/filesystem';

await Filesystem.downloadFile({
  url: 'https://example.com/large-file.pdf',
  path: 'downloads/file.pdf',
  directory: Directory.Data,
});
```

This bypasses the WebView completely — the native layer handles the download.

### Uploading Files

Use the Fetch API with `FormData` after reading the file as a blob:

```typescript
import { Capacitor } from '@capacitor/core';
import { Filesystem, Directory } from '@capacitor/filesystem';

const file = await Filesystem.getUri({ path: 'photo.jpg', directory: Directory.Data });
const webUrl = Capacitor.convertFileSrc(file.uri);

const response = await fetch(webUrl);
const blob = await response.blob();

const formData = new FormData();
formData.append('file', blob, 'photo.jpg');

await fetch('https://api.example.com/upload', {
  method: 'POST',
  body: formData,
});
```

### Opening Files

Use a file opener plugin to open files in external apps (e.g., PDF viewer, image gallery):

- `@capawesome-team/capacitor-file-opener` (recommended)
- `@capacitor-community/file-opener`

For plugin installation and configuration, use the `capacitor-plugins` skill.

### Picking Files

Use a file picker plugin to let the user select files without loading them into the WebView:

- `@capawesome/capacitor-file-picker` (recommended)

The picker returns file paths or URIs. Convert them with `Capacitor.convertFileSrc()` for display, or read them as blobs for upload.

For plugin installation and configuration, use the `capacitor-plugins` skill.

## Filesystem Directories

The `@capacitor/filesystem` plugin provides these directory constants:

| Directory | Description | Persists Across Updates |
| --------- | ----------- | ----------------------- |
| `Directory.Data` | App-specific data directory | Yes |
| `Directory.Documents` | User documents (iOS: backed up to iCloud) | Yes |
| `Directory.Library` | App library directory (iOS only) | Yes |
| `Directory.Cache` | Temporary cache (OS may clear) | No |
| `Directory.External` | External storage (Android only) | Yes |
| `Directory.ExternalStorage` | Shared external storage (Android only, requires permissions) | Yes |

## Summary

| Operation | Approach |
| --------- | -------- |
| Read large files | `convertFileSrc()` + `fetch()` as blob |
| Display images/videos | `convertFileSrc()` as `src` attribute |
| Download files | `Filesystem.downloadFile()` |
| Upload files | `convertFileSrc()` + `fetch()` blob + `FormData` |
| Open in external app | File opener plugin |
| Pick files | File picker plugin |
