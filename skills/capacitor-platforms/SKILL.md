---
name: capacitor-platforms
description: "Guides the agent through adding, configuring, and using the Capawesome desktop platforms for Capacitor — @capawesome/capacitor-electron and @capawesome/capacitor-tauri. Covers choosing between Electron and Tauri, installation and scaffolding, sync/run workflows, live reload, deep links, plugin compatibility (Electron plugin implementations and web fallback; Tauri plugin tiers), packaging, and app updates. Do not use for the Android or iOS platforms, installing individual Capacitor plugins, migrating Capacitor apps or plugins to newer versions, or non-Capacitor desktop frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-platforms
---

# Capacitor Platforms (Desktop)

Bring a Capacitor app to macOS, Windows, and Linux with the Capawesome desktop platforms: [`@capawesome/capacitor-electron`](https://github.com/capawesome-team/capacitor-electron) and [`@capawesome/capacitor-tauri`](https://github.com/capawesome-team/capacitor-tauri). Both use the standard Capacitor workflow (`cap add` / `cap sync` / `cap run`), report `Capacitor.isNativePlatform()` as `true`, and deliver deep links through the standard `@capacitor/app` events.

## Prerequisites

| Requirement | Electron | Tauri |
| --- | --- | --- |
| Node.js | LTS (18+) | LTS (18+) |
| Capacitor | 6+ | 8+ |
| Electron | >= 28 (installed by the scaffold) | — |
| Rust toolchain + system dependencies | — | Required — see the [Tauri prerequisites guide](https://v2.tauri.app/start/prerequisites/) |

On Capacitor 6 and 7, the Capacitor CLI ignores the exit code of platform hooks — a failing `npx cap sync` still reports success. Check the log output for `[capacitor-electron]` errors. Capacitor 8 fails the command properly.

## Agent Behavior

- **Auto-detect before asking.** Read `package.json` and the Capacitor config to determine installed platforms and plugins before prompting the user.
- **Help choose the platform first.** If the user has not committed to Electron or Tauri, apply Step 1 — plugin requirements usually decide it.
- **Always use the full package name** with Capacitor CLI commands (e.g. `npx cap sync @capawesome/capacitor-electron`). A bare `npx cap sync electron` or `npx cap sync tauri` silently does nothing.
- **Check plugin compatibility early.** On Tauri, list the app's Capacitor plugins and verify each is covered by a web implementation or a curated shim (see `references/tauri.md`) before recommending it.

## Procedures

### Step 1: Choose a Platform

| The app needs… | Choose |
| --- | --- |
| Reuse of Capacitor plugins with native (Node) desktop implementations | **Electron** |
| Web-bundle over-the-air updates | **Electron** (ships a bundle-serving primitive; Tauri compiles web assets into the binary — only full signed binary updates) |
| A single, predictable bundled Chromium across all OSes | **Electron** |
| Smallest binaries (~3–10 MB vs ~85–120 MB) and lowest memory use | **Tauri** |
| A deny-by-default security model with a Rust core | **Tauri** |
| No Rust toolchain in dev/CI | **Electron** |

Tauri's trade-offs are inherent, not bugs: no Node runtime (so no arbitrary Capacitor plugin reuse), three different system webviews (WKWebView/WebView2/WebKitGTK), no web-bundle OTA, and a required Rust toolchain. If in doubt, read the Honest Limitations section of the [Tauri platform README](https://github.com/capawesome-team/capacitor-tauri).

### Step 2: Install and Add the Platform

Electron:

```bash
npm install @capawesome/capacitor-electron
npx cap add @capawesome/capacitor-electron
cd electron && npm install && cd ..
```

Then add a `postinstall` script to the app's root `package.json` so the Electron dependencies are always installed together with the main app dependencies:

```json
{
  "scripts": {
    "postinstall": "cd electron && npm ci && cd .."
  }
}
```

Tauri (verify `rustc --version` works first; if not, stop and walk the user through the [Tauri prerequisites](https://v2.tauri.app/start/prerequisites/)):

```bash
npm install @capawesome/capacitor-tauri
npx cap add @capawesome/capacitor-tauri
```

Both scaffolds contain only files the user owns (config, icons, a few-line entry point); all runtime logic lives in the versioned package/crate and updates via `npm update` / `cargo update`.

### Step 3: Sync and Run

```bash
npx cap sync @capawesome/capacitor-electron   # or @capawesome/capacitor-tauri
npx cap run @capawesome/capacitor-electron    # or @capawesome/capacitor-tauri
```

Electron sync copies web assets and regenerates the plugin manifest. Tauri sync additionally scans the app's Capacitor plugin dependencies and generates a deny-by-default capability file, the curated `tauri-plugin-*` crate dependencies, and the Rust plugin registrations.

For live reload on either platform, set `server.url` in the Capacitor config to the local dev server and run the platform — see the platform reference for details.

### Step 4: Configure

- Electron: typed options in `electron/capacitor.electron.config.ts` (window, CSP, deep links, hooks) — see `references/electron.md`.
- Tauri: `src-tauri/tauri.conf.json` (window, CSP, deep links via `plugins.deep-link.desktop.schemes`, bundle) — see `references/tauri.md`.

### Step 5: Package for Distribution

- Electron: `cd electron && npm run pack` (electron-builder; vendors the runtime and plugin implementations automatically) — see `references/electron.md`.
- Tauri: `npx cap sync @capawesome/capacitor-tauri && npx tauri build` (native installers per OS) — see `references/tauri.md`.

## References

- `references/electron.md` — scaffold layout, configuration, live reload, deep links, plugin support and plugin development contract, packaging and vendoring, app updates, migration from `@capacitor-community/electron`.
- `references/tauri.md` — Rust prerequisites, scaffold layout, sync-time codegen, the plugin tier model (built-in / curated / web fallback), configuration, live reload, deep links, packaging, honest limitations.
