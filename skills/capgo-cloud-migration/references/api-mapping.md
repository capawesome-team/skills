# Capgo → Capawesome API Mapping

Complete mapping from `@capgo/capacitor-updater` (the `CapacitorUpdater` class) to `@capawesome/capacitor-live-update` (the `LiveUpdate` class). Method signatures verified against `@capgo/capacitor-updater` v8.50.2 and the `@capawesome/capacitor-live-update` API reference.

## Methods

| Capgo (`CapacitorUpdater`) | Capawesome (`LiveUpdate`) | Notes |
|---|---|---|
| `notifyAppReady()` | `ready()` | Call as early as possible in app startup. Returns `{ currentBundleId, previousBundleId, rollback }` instead of `{ bundle }`. Rollback protection only applies if `readyTimeout` > 0 |
| `getLatest(options?)` | `fetchLatestBundle(options?)` | Both accept `{ channel }`. Result fields: `version` → `bundleId`, `url` → `downloadUrl`. Also returns `artifactType`, `checksum`, `signature`, `customProperties` |
| `download({ url, version, checksum?, sessionKey? })` | `downloadBundle({ url, bundleId, artifactType?, checksum?, signature? })` | Pass the values returned by `fetchLatestBundle()` |
| `next({ id })` | `setNextBundle({ bundleId })` | Applied on next reload/restart |
| `set({ id })` | `setNextBundle({ bundleId })` + `reload()` | Capgo's `set()` reloads immediately; replicate with two calls |
| `reload()` | `reload()` | Same behavior |
| `reset(options?)` | `reset()` + `reload()` | Capawesome resets to the built-in bundle and requires an explicit `reload()` (or restart). No `toLastSuccessful` / `usePendingBundle` equivalent |
| `list(options?)` | `getDownloadedBundles()` | Returns `{ bundleIds: string[] }` instead of `BundleInfo[]` objects |
| `delete({ id })` | `deleteBundle({ bundleId })` | |
| `current()` | `getCurrentBundle()` | Returns `{ bundleId }` (`null` = built-in bundle). For the native version, use `getVersionName()` / `getVersionCode()` |
| `getNextBundle()` | `getNextBundle()` | Returns `{ bundleId: string \| null }` instead of `BundleInfo \| null` |
| `setChannel({ channel, triggerAutoUpdate? })` | Prefer `sync({ channel })` or `fetchLatestBundle({ channel })`; use `setChannel({ channel })` only for a persistent assignment | No server-side "self-assign" opt-in required. `sync({ channel })` also covers `triggerAutoUpdate: true` |
| `unsetChannel(options?)` | `setChannel({ channel: null })` | |
| `getChannel()` | `getChannel()` | Returns `{ channel }` only (no `status`/`allowSet`) |
| `listChannels()` | `fetchChannels(options?)` | Requires channel discovery enabled in the Capawesome Cloud Console |
| `setCustomId({ customId })` | `setCustomId({ customId })` | Capawesome also accepts `null` to remove the custom ID |
| `getDeviceId()` | `getDeviceId()` | Same purpose |
| `getBuiltinVersion()` | `getVersionName()` | Native version name; use `getVersionCode()` for the version code |
| `getAppId()` / `setAppId({ appId })` | `getConfig()` / `setConfig({ appId })` | Persisted across restarts, auto-reset on native update. `resetConfig()` restores the Capacitor config values |
| `isAutoUpdateEnabled()` / `isAutoUpdateAvailable()` | `getConfig()` | Inspect the returned `autoUpdateStrategy` (`'none'` or `'background'`) |
| `triggerUpdateCheck()` | `sync(options?)` | `sync()` fetches, downloads, and sets the next bundle in one call; returns `{ nextBundleId }` |
| `getFailedUpdate()` | `getBlockedBundles()` | Approximate: with `autoBlockRolledBackBundles: true`, rolled-back bundles are blocked and listable; `clearBlockedBundles()` unblocks them |
| `setBundleError({ id })` | No equivalent | Bundle blocking is automatic via `autoBlockRolledBackBundles` |
| `setMultiDelay(...)` / `cancelDelay()` | No equivalent | Implement delay conditions in JavaScript before calling `setNextBundle()` / `reload()` |
| `setUpdateUrl(...)` / `setStatsUrl(...)` / `setChannelUrl(...)` | No runtime equivalent | Use the static `serverDomain` config option (e.g. for EU hosting) |
| `getPluginVersion()` | No equivalent | |
| `getMissingBundleFiles(...)` / `getBundleDownloadSize(...)` | No equivalent | Delta downloads are handled internally with `artifactType: 'manifest'` |
| `startPreviewSession(...)`, `listPreviews()`, `setPreview(...)`, `resetPreview()`, `deletePreview(...)`, `checkPreviewUpdate(...)`, `updatePreview(...)` | No equivalent | Preview sessions are Capgo-specific; use dedicated test channels instead |
| `setShakeMenu(...)` / `isShakeMenuEnabled()` / `setShakeChannelSelector(...)` / `isShakeChannelSelectorEnabled()` | No equivalent | Shake menu is Capgo-specific |

## Event Listeners

| Capgo event | Capawesome event | Notes |
|---|---|---|
| `download` | `downloadBundleProgress` | Payload: `{ bundleId, progress, downloadedBytes, totalBytes }` (`progress` is 0-1, not percent) |
| `setNext` | `nextBundleSet` | Payload: `{ bundleId: string \| null }` |
| `appReloaded` | `reloaded` | No payload |
| `noNeedUpdate`, `updateAvailable`, `updateCheckResult`, `downloadComplete` | No equivalent | Use the return values of `sync()` / `fetchLatestBundle()` instead |
| `updateFailed`, `downloadFailed` | No equivalent | Handle promise rejections of `sync()` / `downloadBundle()` |
| `breakingAvailable`, `majorAvailable` | No equivalent | Use native version constraints on upload (`--android-min` / `--ios-min`) to prevent incompatible updates |
| `set`, `appReady`, `channelPrivate`, `onFlexibleUpdateStateChange` | No equivalent | |

`removeAllListeners()` exists in both plugins.

## Result Type Changes

- Capgo identifies bundles by an internal `id` plus a semver `version`. Capawesome uses a single `bundleId` string (assigned by Capawesome Cloud on upload) — there is no separate semver bundle version.
- Capgo's `BundleInfo` objects (`{ id, version, downloaded, checksum, status }`) have no equivalent; Capawesome methods return plain `bundleId` strings.
- `sync()` (Capawesome only) returns `{ nextBundleId: string | null }` — `null` means the app is up to date.
