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

### Configure the Android notification

Call once on every app start, before the first transfer, and again when the user changes the language:

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

await FileTransfer.setConfig({
  androidNotification: { title: 'Downloads', channelName: 'Downloads' },
});
```

### Download a file

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

const { id } = await FileTransfer.startDownload({
  url: 'https://example.com/file.zip',
  path: '/path/to/destination/file.zip',
  title: 'Quarterly report',
  headers: { Authorization: 'Bearer <token>' },
  network: 'unmetered',
  maxRetries: 3,
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
await FileTransfer.addListener('transferInterrupted', (event) => console.warn(event.id, event.reason));
```

### Manage and retrieve transfers

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

await FileTransfer.pauseTransferById({ id });
await FileTransfer.resumeTransferById({ id }); // resumes `paused`, `interrupted` and `failed` transfers
await FileTransfer.cancelTransferById({ id });

const { transfer } = await FileTransfer.getTransferById({ id });
const { transfers } = await FileTransfer.getTransfers();
```

### Resume interrupted transfers on app start

```typescript
import { FileTransfer } from '@capawesome-team/capacitor-file-transfer';

const { transfers } = await FileTransfer.getTransfers();
for (const transfer of transfers) {
  if (transfer.state === 'interrupted') {
    await FileTransfer.resumeTransferById({ id: transfer.id });
  }
}
```

## Notes

- Unlike the official `@capacitor/file-transfer` plugin, transfers continue while the app is in the background (Android `dataSync` foreground service, iOS background `URLSession`) and the plugin adds pause/resume that survives process death, a persisted task store (`getTransferById`, `getTransfers`), `transferCompleted`/`transferFailed`/`transferInterrupted` events, automatic retries, and network constraints.
- `startDownload()` and `startUpload()` resolve immediately with `{ id }`. Observe the outcome via the `transferCompleted`, `transferFailed` and `transferInterrupted` events. A `Transfer` has `id`, `type` (`download`, `upload`), `state` (`pending`, `running`, `paused`, `interrupted`, `completed`, `failed`, `canceled`), `title`, `url`, `path`, `bytes`, `totalBytes`, `errorCode` (set when `failed`) and `interruptionReason` (set when `interrupted`). Completed and canceled transfers are removed on the next app start.
- `transferProgress` is throttled to roughly one event every 100 ms per transfer. `transferCompleted`, `transferFailed` and `transferInterrupted` events that occur while no listener is registered are retained and delivered once a listener is added.
- `paused` is set by the app, `interrupted` by the system. A transfer becomes `interrupted` with `interruptionReason: 'appTerminated'` when the app process is terminated (OS kill, force-quit, device restart) and with `'timeLimitReached'` on Android 15+ when the `dataSync` foreground service has used its 6 hours per day; such transfers can be resumed once the app is in the foreground again. Interrupted transfers are not failures and are not retried automatically; resume them with `resumeTransferById()`. `failed` is reserved for errors of the transfer itself (network error after the last retry, HTTP error response) and carries an `errorCode`.
- Only downloads can be paused. `pauseTransferById()` rejects with the `TRANSFER_NOT_PAUSABLE` error code for uploads and non-resumable downloads. `resumeTransferById()` resumes `paused`, `interrupted` and `failed` transfers and resolves without effect for any other state. Resuming requires the server to support the HTTP `Range` header on Android; on iOS, the resume data of the stopped task is used. `cancelTransferById()` cancels an unfinished transfer and deletes partially transferred data, so the transfer cannot be resumed afterwards.
- Download options: `url`, `path`, `title` (default: file name of `path`), `hidden` (default `false`), `headers`, `method` (`'GET' | 'POST'`, default `'GET'`), `network` (`'any' | 'unmetered'`, default `'any'`), `resumable` (default `true`), `maxRetries` (default `0`).
- Upload options: `url`, `path`, `title`, `hidden`, `method` (`'POST' | 'PUT'`, default `'POST'`), `uploadType` (`'binary' | 'multipart'`, default `'multipart'`), `fileField` (default `'file'`), `mimeType`, `formFields`, `headers`, `network`, `maxRetries`.
- Android shows one notification while transfers run, because Android requires a foreground service to have one. It lists every running transfer by its `title` and shows their combined progress. `setConfig({ androidNotification: { title, channelName } })` sets its title and channel name (both default to `'File Transfer'`); the configuration replaces the previous one, omitted properties fall back to their defaults, and it is kept only until the app restarts. `hidden: true` hides a transfer from the notification. On Android 12+, a transfer that starts while the app is in the background (e.g. after waiting for an unmetered network) runs without the foreground service and its notification.
- On Android 13+, the notification is only shown if `requestPermissions()` returns `notifications: 'granted'`. Transfers run either way. On Android 12 and older, on iOS and on the web, permissions resolve as `granted` without prompting.
- On the web, `startDownload()`, `startUpload()`, `pauseTransferById()` and `resumeTransferById()` reject as unavailable; the remaining methods report that no transfer exists.
- Breaking changes in 0.2.0: the per-transfer `androidNotification` option of `startDownload()` and `startUpload()` was removed in favor of `setConfig()` plus `title` and `hidden`, and transfers stopped by the system are restored as `interrupted` instead of `failed`. A force-quit on iOS is no longer retried automatically via `maxRetries`; the transfer is restored as `interrupted` on relaunch and resumed from the task's resume data.
