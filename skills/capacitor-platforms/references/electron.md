# Electron Platform Reference

Package: [`@capawesome/capacitor-electron`](https://github.com/capawesome-team/capacitor-electron) (MIT). Maximum plugin compatibility and web-bundle update support, at the cost of larger binaries (bundled Chromium).

## Compatibility

Supports **Capacitor 6+** and **Electron 28+** (no upper Electron bound — update Electron in `electron/package.json` and run `npm install` there). On Capacitor 6 and 7, the Capacitor CLI ignores the exit code of platform hooks, so a failing `npx cap sync` still reports success — check the log output for `[capacitor-electron]` errors. Capacitor 8 fails the command properly.

## Scaffold Layout

`npx cap add @capawesome/capacitor-electron` creates an `electron/` directory containing only user-owned files:

| File | Purpose |
| --- | --- |
| `main.ts` | ~5 lines: imports the runtime and starts the app |
| `capacitor.electron.config.ts` | Typed platform options (window, CSP, deep links, hooks) |
| `electron-builder.config.js` | Packaging configuration |
| `package.json` | Electron/electron-builder/TypeScript devDependencies and the `build`/`start`/`pack` scripts |
| `tsconfig.json` | TypeScript configuration for `main.ts` and the platform config |
| `assets/` | App icons |

All runtime logic lives in the npm package and updates via `npm update`.

After `cap add`, install the Electron dependencies once (`cd electron && npm install && cd ..`) and add a `postinstall` script to the app's root `package.json` so they stay in sync with the main app dependencies:

```json
{
  "scripts": {
    "postinstall": "cd electron && npm ci && cd .."
  }
}
```

## Configuration

Typed options in `electron/capacitor.electron.config.ts`:

```typescript
import { defineConfig } from '@capawesome/capacitor-electron/config';

export default defineConfig({
  window: {
    width: 1200,
    height: 800,
    minWidth: 800,
    minHeight: 600,
  },
  deepLinks: {
    scheme: 'myapp',
  },
});
```

Available top-level options: `scheme` (default `'capacitor-electron'`), `hostname` (default `'localhost'`), `window` (size/min/max, `backgroundColor`, `fullscreen`, `statePersistence`, `titleBarStyle`), `csp` (`policy` and `devPolicy` overrides), `deepLinks` (`scheme`), `singleInstance` (default `true`), and `hooks`.

Extension happens through typed options and the hooks nested under the `hooks` key (`beforeReady`, `windowFactory`, `onWindowCreated`) — never by owning runtime code. The security defaults (sandboxed renderer, context isolation, validated IPC) are not configurable; `windowFactory` must not weaken the mandatory security window options.

## Live Reload

Set `server.url` in the Capacitor config to the dev server and run the platform:

```typescript
// capacitor.config.ts
const config: CapacitorConfig = {
  server: {
    url: 'http://localhost:5173',
  },
};
```

```bash
npx vite &
npx cap run @capawesome/capacitor-electron
```

Dev mode applies a documented, dev-only CSP relaxation (inline scripts, eval, websockets) and reconnects automatically when the dev server restarts. Remove `server.url` to serve built web assets again.

## Deep Links

Declare the scheme in the platform configuration (see above) and listen with the standard `@capacitor/app` plugin:

```typescript
import { App } from '@capacitor/app';

await App.addListener('appUrlOpen', ({ url }) => {
  console.log('App opened with URL:', url);
});
```

Deep links opened while the app runs are routed to the running instance (single instance is enforced by default); the launch URL is available via `App.getLaunchUrl()`.

## Plugin Support

- Plugins with a **web implementation** work unchanged via automatic fallback.
- Plugins with a dedicated **electron implementation** get native (Node) capability.

Plugins declare their electron implementation via `package.json`:

```json
{
  "capacitor": {
    "electron": { "src": "electron" }
  }
}
```

The implementation is an ES module at `<src>/dist/plugin.mjs` exporting plugin classes with a static `__capacitorElectronPlugin` property (`{ name: string; methods: string[] }`). The static property is the contract, so plugins need no build-time dependency on the platform package (an optional `defineElectronPlugin` helper is exported from `@capawesome/capacitor-electron/plugin`). A declared method missing on the class fails loudly at boot. Implementations written for the old `@capacitor-community/electron` platform are **not** loaded.

## Packaging

```bash
cd electron && npm run pack
```

The `pack` script runs compile (`tsc`), **vendor** (`capacitor-electron vendor` — copies the platform runtime, every plugin's electron implementation, and their dependency closure into `electron/vendor/`, mapped to `node_modules` inside the packaged app), and `electron-builder`. Code signing, notarization, and targets (dmg/msi/nsis/AppImage/deb) are standard electron-builder configuration in the user-owned `electron-builder.config.js`. Electron Forge is a supported alternative: run `capacitor-electron vendor` before packaging and include `vendor/node_modules` as the app's `node_modules`.

Native Node addons are **not** rebuilt automatically — the vendor step detects and reports them, but they must be rebuilt against Electron's ABI (e.g. with `@electron/rebuild`).

## App Updates

Two independent layers:

- **Binary updates** (Electron, runtime, native modules): use [`electron-updater`](https://www.electron.build/auto-update), wired in the user-owned `main.ts`.
- **Web-bundle updates**: the platform ships the serving primitive only — plugin implementations can access `services.bundles` (`getActiveBundlePath()`, `setActiveBundle(dir | null)`, `notifyBootReady()`) with a failed-boot rollback watchdog that reverts to the previous bundle if the renderer never signals boot-ready. A full OTA product (download, channels, verification) is deliberately not included.

## Migration from `@capacitor-community/electron`

The platform replaces `@capacitor-community/electron`. Key differences: the scaffold is minimal and user-owned (no runtime code to maintain), and plugin implementations use the contract above — old-platform implementations are not loaded, but web implementations keep working via the fallback. Follow the Migration section of the [README](https://github.com/capawesome-team/capacitor-electron) for the step-by-step procedure.
