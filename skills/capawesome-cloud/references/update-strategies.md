# Update Strategies

## Background

Set `autoUpdateStrategy: "background"` in the plugin config. No additional code needed.

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
