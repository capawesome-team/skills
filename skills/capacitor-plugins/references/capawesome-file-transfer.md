# File Transfer

Capacitor plugin for reliable background file uploads and downloads that survive the app being backgrounded. Task-based transfers with pause/resume, progress events, retries, and a persisted task store.

**Package:** `@capawesome-team/capacitor-file-transfer`
**Platforms:** Android, iOS
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-file-transfer
npx cap sync
```

## Configuration

### Android

No changes to `android/app/src/main/AndroidManifest.xml` are required. The plugin declares the `INTERNET`, `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_DATA_SYNC` and `POST_NOTIFICATIONS` permissions and the `dataSync` foreground service in its own manifest.

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
-keep class okhttp3.** { *; }
-keep class okio.** { *; }
```

#### Variables

Optionally define in `android/variables.gradle` to change the dependency version:

- `okhttpVersion` — version of `com.squareup.okhttp3:okhttp` (default: `4.12.0`)

### iOS

#### Background URLSession Handler

The plugin uses a background `URLSession`. Add the following extension to `ios/App/App/AppDelegate.swift` so iOS can forward the completion handler when a transfer finishes while the app is not running. Without it, background transfers still complete, but the final events may be delivered late:

```swift
import Foundation

extension AppDelegate {
    func application(
        _ application: UIApplication,
        handleEventsForBackgroundURLSession identifier: String,
        completionHandler: @escaping () -> Void
    ) {
        NotificationCenter.default.post(
            name: Notification.Name("io.capawesome.capacitorjs.plugins.filetransfer.handleEventsForBackgroundURLSession"),
            object: completionHandler
        )
    }
}
```

## Usage

### Download a file

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

const { id } = await FileTransfer.startDownload({
  url: 'https://example.com/file.zip',
  path: '/path/to/destination/file.zip',
  headers: { Authorization: 'Bearer <token>' },
  network: 'unmetered',
  maxRetries: 3,
  androidNotification: { title: 'Downloading file', progress: true },
});
```

### Upload a file

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

// `multipart/form-data` upload
const { id } = await FileTransfer.startUpload({
  url: 'https://example.com/upload',
  path: '/path/to/source/file.jpg',
  fileField: 'file',
  mimeType: 'image/jpeg',
  formFields: { albumId: '42' },
});

// Raw body upload, e.g. to an S3 presigned URL
const { id: binaryId } = await FileTransfer.startUpload({
  url: 'https://example.com/presigned-url',
  path: '/path/to/source/file.jpg',
  method: 'PUT',
  uploadType: 'binary',
});
```

### Listen for transfer events

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

await FileTransfer.addListener('transferProgress', (event) => console.log(event.id, event.bytes, event.totalBytes));
await FileTransfer.addListener('transferCompleted', (event) => console.log(event.id, event.path, event.responseBody));
await FileTransfer.addListener('transferFailed', (event) => console.error(event.id, event.errorCode, event.message));
```

### Manage and retrieve transfers

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

await FileTransfer.pauseTransferById({ id });
await FileTransfer.resumeTransferById({ id });
await FileTransfer.cancelTransferById({ id });

const { transfer } = await FileTransfer.getTransferById({ id });
const { transfers } = await FileTransfer.getTransfers();
```

## Notes

- Unlike the official `@capacitor/file-transfer` plugin, transfers continue while the app is in the background (Android `dataSync` foreground service, iOS background `URLSession`) and the plugin adds pause/resume that survives process death, a persisted task store (`getTransferById`, `getTransfers`), `transferCompleted`/`transferFailed` events, automatic retries, and network constraints.
- `startDownload()` and `startUpload()` resolve immediately with `{ id }`. Observe the outcome via the `transferCompleted` and `transferFailed` events. A `Transfer` has `id`, `type` (`download`, `upload`), `state` (`pending`, `running`, `paused`, `completed`, `failed`, `canceled`), `url`, `path`, `bytes` and `totalBytes`.
- `transferProgress` is throttled to roughly one event every 100 ms per transfer. `transferCompleted` and `transferFailed` events that occur while no listener is registered are retained and delivered once a listener is added.
- Only downloads can be paused. `pauseTransferById()` rejects with the `TRANSFER_NOT_PAUSABLE` error code for uploads and non-resumable downloads. Resuming requires the server to support the HTTP `Range` header. `cancelTransferById()` deletes partially transferred data, so the transfer cannot be resumed afterwards.
- Download options: `url`, `path`, `headers`, `method` (`'GET' | 'POST'`, default `'GET'`), `network` (`'any' | 'unmetered'`, default `'any'`), `resumable` (default `true`), `maxRetries` (default `0`), `androidNotification`.
- Upload options: `url`, `path`, `method` (`'POST' | 'PUT'`, default `'POST'`), `uploadType` (`'binary' | 'multipart'`, default `'multipart'`), `fileField` (default `'file'`), `mimeType`, `formFields`, `headers`, `network`, `maxRetries`, `androidNotification`.
- `androidNotification` (Android only) accepts `title`, `text`, `channelName` (default `'File Transfer'`) and `progress` (default `false`). The foreground service notification is always shown while a transfer runs; `progress: true` adds a separate per-transfer progress notification.
- On Android 13+, the progress notification is only shown if `requestPermissions()` returns `notifications: 'granted'`. Transfers run either way. On Android 12 and older, on iOS and on the web, permissions resolve as `granted` without prompting.
- On the web, `startDownload()`, `startUpload()`, `pauseTransferById()` and `resumeTransferById()` reject as unavailable; the remaining methods report that no transfer exists.
- If the app is force-quit, Android restores the transfer as `failed` (downloads remain resumable) and iOS cancels it.
