# File Manager

Capacitor plugin to manage files and directories with a URI-based API. Persists access to user-picked folders, runs copy, move and delete operations with progress and cancellation, calculates checksums, and inspects device and app storage.

**Package:** `@capawesome-team/capacitor-file-manager`
**Platforms:** Android, iOS, Web
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-file-manager
npx cap sync
```

## Configuration

No plugin configuration in `capacitor.config.ts` is required.

### Android

No storage permissions are required. App sandbox directories are accessible without any permission, and user-visible folders are accessed through persisted directories granted by the user in the system file picker.

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### Web

Files are stored in the Origin Private File System (OPFS), which is subject to the browser's storage eviction rules. Safari deletes all script-writable storage after seven days without user interaction when the app runs in a browser tab (home-screen web apps are exempt). To reduce the risk of eviction, call `navigator.storage.persist()` at a moment of meaningful user engagement (for example after sign-in):

```typescript
await navigator.storage.persist();
```

## Usage

### Write and read a file in the app's sandbox

```typescript
import { Directory, Encoding, FileManager } from '@capawesome-team/capacitor-file-manager';

const { uri } = await FileManager.getUri({ path: 'notes/todo.txt', directory: Directory.Data });
await FileManager.writeFile({ uri, data: 'Hello, World!', encoding: Encoding.Utf8, recursive: true });
const { data } = await FileManager.readFile({ uri, encoding: Encoding.Utf8 });
```

### Persist access to a user-picked directory

Requires version `8.1.0` or later of `@capawesome/capacitor-file-picker`, since `pickDirectory()` only returns a bookmark from that version on.

```typescript
import { FileManager } from '@capawesome-team/capacitor-file-manager';
import { FilePicker } from '@capawesome/capacitor-file-picker';

const result = await FilePicker.pickDirectory();
const { directory } = await FileManager.persistDirectoryAccess({
  uri: result.path,
  bookmark: result.bookmark,
});

// Call this on app start and use the returned URIs instead of storing them yourself.
const { directories } = await FileManager.getPersistedDirectories();
```

### Work inside a persisted directory

```typescript
import { Directory, FileManager } from '@capawesome-team/capacitor-file-manager';

const { uri: targetUri } = await FileManager.getUri({ path: 'exports/report.pdf', parentUri: directoryUri });
const { uri: sourceUri } = await FileManager.getUri({ path: 'report.pdf', directory: Directory.Cache });
await FileManager.copyFile({ uri: sourceUri, toUri: targetUri });

const { entries } = await FileManager.readDirectory({ uri: directoryUri });
```

### Copy a directory with progress and cancellation

```typescript
import { FileManager } from '@capawesome-team/capacitor-file-manager';

await FileManager.addListener('operationProgress', (event) => {
  console.log(`Processed ${event.processedFiles} of ${event.totalFiles} files`);
});
await FileManager.copyDirectory({ uri, toUri, id: 'my-copy-operation' });

await FileManager.cancelOperationById({ id: 'my-copy-operation' });
```

### Read a large file as a Blob and verify it

```typescript
import { ChecksumAlgorithm, FileManager } from '@capawesome-team/capacitor-file-manager';

const { blob } = await FileManager.readFileAsBlob({ uri });
const { checksum } = await FileManager.getFileChecksum({ uri, algorithm: ChecksumAlgorithm.Sha256 });
```

### Check the storage space

```typescript
import { FileManager } from '@capawesome-team/capacitor-file-manager';

const { freeBytes, totalBytes } = await FileManager.getDeviceStorageInfo();
const { cacheBytes } = await FileManager.getAppStorageInfo();
if (cacheBytes > 100_000_000) {
  await FileManager.clearCache();
}
```

## Notes

- Every file and directory method takes a `uri`. Build it with `getUri({ path, directory })` for sandbox directories or `getUri({ path, parentUri })` for a path relative to any directory URI. The entry does not need to exist.
- `Directory` members: `Cache`, `Data`, `Documents`, `External`, `ExternalCache`, `Library`, `LibraryNoCloud`, `Temporary`. Same names and values as in `@capacitor/filesystem`.
- Persisted directory URIs may change between app launches. Never store them yourself — call `getPersistedDirectories()` on app start; it also refreshes or prunes stale entries. Release access with `releaseDirectoryAccess({ uri })`.
- `persistDirectoryAccess()`, `getPersistedDirectories()`, `releaseDirectoryAccess()` and `getFileChecksum()` are only available on Android and iOS.
- `bookmark` in `persistDirectoryAccess()` is iOS-only; on Android only `uri` is used.
- `copyDirectory()`, `moveDirectory()` and `deleteDirectory()` emit `operationProgress` events and can be canceled via `cancelOperationById({ id })`. The canceled promise rejects with the `OPERATION_CANCELED` error code; already processed files remain in place.
- `deleteDirectory()` defaults to `recursive: false` and rejects with the `NOT_EMPTY` error code if the directory is not empty. `clearDirectory()` empties a directory while preserving persisted access to it.
- `readFile()` loads the whole file into memory. For large files use `readFileAsBlob()`; use `offset`/`length` only for targeted reads such as file headers. Never loop `readFile()` with an increasing offset.
- `encoding` defaults to `Encoding.Base64` for `readFile()`, `writeFile()` and `appendFile()`. Only `Encoding.Base64` and `Encoding.Utf8` exist.
- `writeFile()` overwrites the file unless `position` is set, and defaults to `recursive: false`. `truncateFile()` defaults to `size: 0`.
- Result URIs of `writeFile()`, `copyFile()` and `moveFile()` may differ from the requested URI, for example when the document provider renames the file to avoid a collision. Always use the returned `uri`.
- `readDirectory()` supports paging via `limit` and `offset`. Entries and `getMetadata()` expose `name`, `uri`, `type` (`EntryType.File` / `EntryType.Directory`), `size`, `mimeType`, `createdAt` and `modifiedAt`.
- `ChecksumAlgorithm` members: `Md5`, `Sha1`, `Sha256`. The checksum is returned as a lowercase hexadecimal string.
- Sandbox URIs (`file://`) can be passed to other Capawesome file plugins. Persisted directory URIs (`content://` on Android, security-scoped `file://` on iOS) and web URIs (`opfs://`) cannot — copy the file into a sandbox directory such as `Directory.Cache` first.
- Replaces the sandbox operations of `@capacitor/filesystem`. `checkPermissions()`/`requestPermissions()` are not needed, `Directory.ExternalStorage` is replaced by persisted directories, and `Encoding.ASCII`/`Encoding.UTF16` were dropped.
