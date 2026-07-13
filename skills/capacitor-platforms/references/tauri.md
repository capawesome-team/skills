# Tauri Platform Reference

Package: [`@capawesome/capacitor-tauri`](https://github.com/capawesome-team/capacitor-tauri) (MIT) + the [`capacitor-tauri`](https://crates.io/crates/capacitor-tauri) Rust crate (version-locked to the npm package). Lean, secure desktop apps with tiny system-webview binaries and a deny-by-default Rust core, with a deliberately scoped plugin story.

## Prerequisites

Tauri requires the **Rust toolchain** (1.77.2+) and platform system dependencies (WebKitGTK dev packages on Linux, MSVC Build Tools on Windows, Xcode Command Line Tools on macOS). Verify with `rustc --version`; if missing, follow the [Tauri prerequisites guide](https://v2.tauri.app/start/prerequisites/) before installing.

## Scaffold Layout

`npx cap add @capawesome/capacitor-tauri` creates a `src-tauri/` directory containing only user-owned files:

| File | Purpose |
| --- | --- |
| `src/main.rs` | ~3 lines: starts the app via the platform crate |
| `src/generated.rs` | Generated plugin registrations (regenerated at sync, gitignored) |
| `tauri.conf.json` | App configuration (window, CSP, deep links, bundle) |
| `capabilities/` | Permission grants (regenerated at sync) |
| `Cargo.toml` | Crate dependencies (curated plugin crates added at sync) |
| `build.rs` | Standard Tauri build script |
| `icons/` | App icons |

All runtime logic lives in the `capacitor-tauri` crate and updates via `cargo update`.

## Sync-Time Codegen

`npx cap sync @capawesome/capacitor-tauri` copies web assets (skipped when `server.url` is set) and then statically scans the app's Capacitor plugin dependencies to generate:

- a **deny-by-default capability file** (`capabilities/capacitor.json`) containing only the permissions the enabled curated plugins need,
- the matching `tauri-plugin-*` crate dependencies in the managed block of `Cargo.toml`,
- the Rust plugin registrations in `src/generated.rs`.

Detection scans the app `package.json` `dependencies` only — a curated plugin listed under `devDependencies` is **not** detected. Rerun sync after adding or removing Capacitor plugins.

## Plugin Tiers

| Tier | What | Examples |
| --- | --- | --- |
| Built-in | Implemented by the platform crate | `@capacitor/app` (commands + lifecycle/deep-link events) |
| Curated | Shims calling official `tauri-plugin-*` crates | Filesystem → `fs`, Preferences → `store`, Dialog → `dialog`, Local Notifications → `notification` |
| Web fallback | Any plugin with a web implementation, automatically | e.g. `@capacitor/device` |

Plugins needing native functionality beyond this require a bespoke Rust implementation (Tier 3). Before recommending Tauri, list the app's plugins and verify coverage; if a required plugin is uncovered, recommend the [Electron platform](https://github.com/capawesome-team/capacitor-electron) instead. Requests for new curated plugins go to the [issue tracker](https://github.com/capawesome-team/capacitor-tauri/issues).

`Capacitor.getPlatform()` returns `'tauri'` and `Capacitor.isNativePlatform()` returns `true`.

## Configuration Notes

- The scaffold ships a default CSP in `tauri.conf.json` (`default-src 'self'; img-src 'self' data: blob:; style-src 'self' 'unsafe-inline'`) — apps loading remote resources must extend it.
- The default window is labeled `main` and the generated capability file grants permissions to `["main"]` only — renaming or adding windows requires updating `capabilities/` manually.

## Live Reload

Set `server.url` in the Capacitor config to the dev server, then `npx cap run @capawesome/capacitor-tauri` — it launches `tauri dev` pointing at the dev server with full HMR.

## Deep Links

Declare the scheme in `src-tauri/tauri.conf.json` under `plugins.deep-link.desktop.schemes` and listen with the standard `@capacitor/app` plugin:

```typescript
import { App } from '@capacitor/app';

await App.addListener('appUrlOpen', ({ url }) => {
  console.log('App opened with URL:', url);
});
```

OS URL-scheme handlers are registered for packaged apps; single instance is enforced so links route to the running app.

## Packaging

```bash
npx cap sync @capawesome/capacitor-tauri
npx tauri build
```

Tauri's bundler produces native installers for the current OS (`.dmg`/`.app`, `.msi`/`.exe`, `.deb`/`.AppImage`). Signing, updater, and target configuration live in `src-tauri/tauri.conf.json` — see the [Tauri distribution guide](https://v2.tauri.app/distribute/).

## Honest Limitations

Inherent trade-offs, not bugs — communicate them before the user commits:

1. **No plugin-ecosystem reuse.** No Node runtime; only web implementations, curated shims, and bespoke Rust plugins work.
2. **Three webview engines.** WKWebView (macOS), WebView2 (Windows), WebKitGTK (Linux) behave differently — a real cross-engine testing burden, worst on Linux.
3. **No web-bundle OTA.** Web assets are compiled into the binary; only full signed binary updates (via `tauri-plugin-updater`) are possible.
4. **Rust toolchain required** for developers and CI, with longer first builds and large `target/` directories.
