# Live Update Plugin API Reference

Import: `import { LiveUpdate } from "@capawesome/capacitor-live-update";`

## Methods

### ready()

```typescript
ready() => Promise<ReadyResult>
```

Notify the plugin that the app is ready. Call as early as possible to prevent rollback. Returns `{ currentBundleId, previousBundleId, rollback }`.

### sync(options?)

```typescript
sync(options?: { channel?: string }) => Promise<{ nextBundleId: string | null }>
```

Fetch the latest bundle from Capawesome Cloud, download it, and set it as the next bundle. Call `reload()` or restart the app to apply. Returns `null` if up-to-date.

### reload()

```typescript
reload() => Promise<void>
```

Reload the app to apply the next bundle immediately.

### reset()

```typescript
reset() => Promise<void>
```

Reset to the default (built-in) bundle. Call `reload()` or restart to apply.

### fetchLatestBundle(options?)

```typescript
fetchLatestBundle(options?: { channel?: string }) => Promise<FetchLatestBundleResult>
```

Fetch metadata about the latest bundle without downloading. Returns `{ bundleId, artifactType, downloadUrl, customProperties, channel, checksum, signature }`. `bundleId` is `null` if no bundle available.

### downloadBundle(options)

```typescript
downloadBundle(options: {
  url: string;
  bundleId: string;
  artifactType?: 'zip' | 'manifest';
  checksum?: string;
  signature?: string;
}) => Promise<void>
```

Download a bundle from a URL.

### setNextBundle(options)

```typescript
setNextBundle(options: { bundleId: string | null }) => Promise<void>
```

Set the bundle to use after next reload/restart. Pass `null` to use the default bundle.

### deleteBundle(options)

```typescript
deleteBundle(options: { bundleId: string }) => Promise<void>
```

Delete a downloaded bundle.

### getBundles()

```typescript
getBundles() => Promise<{ bundleIds: string[] }>
```

Get all available bundle identifiers.

### getDownloadedBundles()

```typescript
getDownloadedBundles() => Promise<{ bundleIds: string[] }>
```

Get all downloaded bundle identifiers. Since 7.4.0.

### getCurrentBundle()

```typescript
getCurrentBundle() => Promise<{ bundleId: string | null }>
```

Get the current active bundle ID. `null` means default bundle.

### getNextBundle()

```typescript
getNextBundle() => Promise<{ bundleId: string | null }>
```

Get the next bundle ID. `null` means default bundle.

### setChannel(options)

```typescript
setChannel(options: { channel: string | null }) => Promise<void>
```

Set the update channel. Persisted across restarts. Pass `null` to remove.

### getChannel()

```typescript
getChannel() => Promise<{ channel: string | null }>
```

Get the currently resolved channel.

### fetchChannels(options?)

```typescript
fetchChannels(options?: { limit?: number; offset?: number; query?: string }) => Promise<{ channels: Array<{ id: string; name: string }> }>
```

Fetch available channels (requires channel discovery enabled). Since 8.2.0.

### setCustomId(options)

```typescript
setCustomId(options: { customId: string | null }) => Promise<void>
```

Set a custom device identifier.

### getCustomId()

```typescript
getCustomId() => Promise<{ customId: string | null }>
```

### getDeviceId()

```typescript
getDeviceId() => Promise<{ deviceId: string }>
```

Get the unique device identifier.

### getVersionCode()

```typescript
getVersionCode() => Promise<{ versionCode: string }>
```

Get the native version code (`versionCode` on Android, `CFBundleVersion` on iOS).

### getVersionName()

```typescript
getVersionName() => Promise<{ versionName: string }>
```

Get the native version name (`versionName` on Android, `CFBundleShortVersionString` on iOS).

### isSyncing()

```typescript
isSyncing() => Promise<{ syncing: boolean }>
```

Check if a sync is in progress. Since 7.4.0.

### setConfig(options)

```typescript
setConfig(options: { appId?: string | null }) => Promise<void>
```

Update plugin config at runtime. Persisted across restarts. Auto-reset on native app update. Since 7.4.0.

### getConfig()

```typescript
getConfig() => Promise<{ appId: string | null; autoUpdateStrategy: 'none' | 'background' }>
```

Get current runtime config. Since 7.4.0.

### resetConfig()

```typescript
resetConfig() => Promise<void>
```

Reset runtime config to Capacitor config values. Since 7.4.0.

### getBlockedBundles()

```typescript
getBlockedBundles() => Promise<{ bundleIds: string[] }>
```

Get blocked bundle IDs. Since 7.4.0.

### clearBlockedBundles()

```typescript
clearBlockedBundles() => Promise<void>
```

Clear all blocked bundles. Since 7.4.0.

## Event Listeners

### downloadBundleProgress

```typescript
LiveUpdate.addListener('downloadBundleProgress', (event) => {
  // event: { bundleId, progress, downloadedBytes, totalBytes }
  console.log(`Download: ${Math.round(event.progress * 100)}%`);
});
```

### nextBundleSet

```typescript
LiveUpdate.addListener('nextBundleSet', (event) => {
  // event: { bundleId: string | null }
  console.log(`Next bundle set: ${event.bundleId}`);
});
```

### reloaded

```typescript
LiveUpdate.addListener('reloaded', () => {
  console.log('App was reloaded');
});
```

### removeAllListeners()

```typescript
LiveUpdate.removeAllListeners() => Promise<void>
```
