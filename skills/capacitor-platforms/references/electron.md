# Electron Platform Reference

Package: [`@capawesome/capacitor-electron`](https://github.com/capawesome-team/capacitor-electron) (MIT). Maximum plugin compatibility and web-bundle OTA, at the cost of larger binaries (bundled Chromium).

## Scaffold Layout

`npx cap add @capawesome/capacitor-electron` creates an `electron/` directory containing only user-owned files:

| File | Purpose |
| --- | --- |
| `main.ts` | ~5 lines: imports the runtime and starts the app |
| `capacitor.electron.config.ts` | Typed platform options (window, CSP, deep links, hooks) |
| `electron-builder.config.js` | Packaging configuration |
| `assets/` | App icons |

All runtime logic lives in the npm package and updates via `npm update`.

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

Extension happens through typed options and hooks (`windowFactory`, `beforeReady`, `onWindowCreated`, CSP overrides) — never by owning runtime code.

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

The implementation is an ES module at `<src>/dist/plugin.mjs` exporting plugin classes with static metadata (registration name + public API). The static property is the contract, so plugins need no build-time dependency on the platform package. Implementations written for the old `@capacitor-community/electron` platform are **not** loaded.

## Packaging

```bash
cd electron && npm run pack
```

The `pack` script runs compile (`tsc`), **vendor** (`capacitor-electron vendor` — copies the platform runtime, every plugin's electron implementation, and their dependency closure into `electron/vendor/`, mapped to `node_modules` inside the packaged app), and `electron-builder`. Code signing, notarization, and targets (dmg/msi/nsis/AppImage/deb) are standard electron-builder configuration in the user-owned `electron-builder.config.js`. Electron Forge is a supported alternative: run `capacitor-electron vendor` before packaging and include `vendor/node_modules` as the app's `node_modules`.

## Migration from `@capacitor-community/electron`

The platform replaces `@capacitor-community/electron`. Key differences: the scaffold is minimal and user-owned (no runtime code to maintain), and plugin implementations use the contract above — old-platform implementations are not loaded, but web implementations keep working via the fallback. Follow the Migration section of the [README](https://github.com/capawesome-team/capacitor-electron) for the step-by-step procedure.
