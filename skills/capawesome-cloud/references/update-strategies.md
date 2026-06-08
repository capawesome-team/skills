# Update Strategies

The snippets below use the **Capacitor** API. For **Cordova**, the same strategies apply with three differences: access the plugin via `cordova.plugins.LiveUpdate`, run startup code inside a `deviceready` listener, and configure the strategy via the `AUTO_UPDATE_STRATEGY` preference in `config.xml`. See [Cordova Variants](#cordova-variants) at the end for ready-to-use examples.

## Background

Set `autoUpdateStrategy: "background"` in the plugin config (Capacitor) or `AUTO_UPDATE_STRATEGY=background` in `config.xml` (Cordova). No additional code needed.

Behavior:
- Checks for updates at app startup and on resume (if last check >15 min ago).
- Downloads in the background.
- Applies on next app launch.

To force a check during development, force-close and restart the app.

## Always Latest (Recommended)

Combines background updates with a user prompt when an update is ready.

```typescript
import { LiveUpdate } from "@capawesome/capacitor-live-update";

void LiveUpdate.ready();

LiveUpdate.addListener("nextBundleSet", async (event) => {
  if (event.bundleId) {
    const shouldReload = confirm("A new update is available. Install now?");
    if (shouldReload) {
      await LiveUpdate.reload();
    }
  }
});
```

Set `autoUpdateStrategy: "background"` in the config.

## Force Update

Show a splash/loading screen while downloading, then reload immediately.

```typescript
import { LiveUpdate } from "@capawesome/capacitor-live-update";
import { SplashScreen } from "@capacitor/splash-screen";

void LiveUpdate.ready();

const checkForUpdate = async () => {
  const { nextBundleId } = await LiveUpdate.sync();
  if (nextBundleId) {
    await SplashScreen.show();
    await LiveUpdate.reload();
  }
};

void checkForUpdate();
```

Set `autoUpdateStrategy: "none"` in the config.

## Manual Sync (Capacitor 6)

Capacitor 6 does not support `autoUpdateStrategy`. Implement sync manually:

```typescript
import { App } from "@capacitor/app";
import { LiveUpdate } from "@capawesome/capacitor-live-update";

void LiveUpdate.ready();

App.addListener("resume", async () => {
  const { nextBundleId } = await LiveUpdate.sync();
  if (nextBundleId) {
    const shouldReload = confirm("A new update is available. Install now?");
    if (shouldReload) {
      await LiveUpdate.reload();
    }
  }
});
```

## Advanced: Manual Fetch + Download

For full control over the update flow without using `sync()`:

```typescript
import { LiveUpdate } from "@capawesome/capacitor-live-update";

const checkAndApply = async () => {
  const { bundleId, downloadUrl, artifactType, checksum, signature } =
    await LiveUpdate.fetchLatestBundle();

  if (!bundleId) return;

  const { bundleId: currentId } = await LiveUpdate.getCurrentBundle();
  if (bundleId === currentId) return;

  await LiveUpdate.downloadBundle({
    url: downloadUrl,
    bundleId,
    artifactType,
    checksum,
    signature,
  });

  await LiveUpdate.setNextBundle({ bundleId });
  await LiveUpdate.reload();
};
```

Set `autoUpdateStrategy: "none"` when using this approach.

## Cordova Variants

For Cordova, run startup logic inside a `deviceready` listener and access the plugin via `cordova.plugins.LiveUpdate`.

### Always Latest (Cordova)

Set `AUTO_UPDATE_STRATEGY=background` in `config.xml`.

```javascript
document.addEventListener("deviceready", async () => {
  await cordova.plugins.LiveUpdate.ready();

  cordova.plugins.LiveUpdate.addListener("nextBundleSet", async (event) => {
    if (event.bundleId) {
      const shouldReload = confirm("A new update is available. Install now?");
      if (shouldReload) {
        await cordova.plugins.LiveUpdate.reload();
      }
    }
  });
});
```

### Manual Sync (Cordova)

Set `AUTO_UPDATE_STRATEGY=none` in `config.xml` and sync on resume:

```javascript
document.addEventListener("deviceready", async () => {
  await cordova.plugins.LiveUpdate.ready();

  document.addEventListener("resume", async () => {
    const { nextBundleId } = await cordova.plugins.LiveUpdate.sync();
    if (nextBundleId) {
      const shouldReload = confirm("A new update is available. Install now?");
      if (shouldReload) {
        await cordova.plugins.LiveUpdate.reload();
      }
    }
  });
});
```
