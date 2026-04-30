# Capacitor CLI

Command reference for the Capacitor CLI (`@capacitor/cli`). All commands are invoked via `npx cap <command>`.

## Commands

### `npx cap init`

Initialize Capacitor in an existing web project. Creates `capacitor.config.ts` and `package.json` entries.

```bash
npx cap init <appName> <appId>
```

- `appName`: The display name of the app (e.g., `"My App"`).
- `appId`: The app identifier in reverse-domain format (e.g., `com.example.myapp`).

### `npx cap add <platform>`

Add a native platform to the project.

```bash
npx cap add android
npx cap add ios
```

Creates the `android/` or `ios/` directory with a full native project.

### `npx cap sync`

Copy web assets and update native plugins and dependencies. This is the most commonly used command.

```bash
npx cap sync [platform]
```

Options:
- `--deployment` — Prevent deletion of `Podfile.lock` and use `pod install --deployment`.
- `--inline` — Inline JS source maps for Android WebView debugging.

Run `npx cap sync` after:
- Building web assets (`npm run build`)
- Installing or removing plugins
- Changing `capacitor.config.ts` or `capacitor.config.json`

### `npx cap copy`

Copy web assets only (no native dependency updates). Faster than `sync` when only web code has changed.

```bash
npx cap copy [platform]
```

### `npx cap update`

Update native plugins and dependencies only (no web asset copy).

```bash
npx cap update [platform]
```

### `npx cap run`

Build, sync, and deploy to a device or emulator.

```bash
npx cap run <platform> [options]
```

Options:
- `--list` — List available target devices.
- `--target <id>` — Deploy to a specific device by ID.
- `--target-name <name>` — Deploy to a device by name (e.g., `"iPhone 17 Pro"`).
- `--target-name-sdk-version <version>` — Target devices with a specific OS version.
- `--flavor <name>` — Android build flavor.
- `--scheme <name>` — iOS build scheme.
- `--configuration <name>` — iOS scheme configuration.
- `--live-reload` or `-l` — Enable live reload (see `references/live-reload.md`).
- `--host <host>` — Live reload host.
- `--port <port>` — Live reload port.
- `--https` — Use HTTPS for live reload.
- `--forwardPorts <port1:port2>` — Run `adb reverse` for Android live reload.
- `--no-sync` — Skip the sync step before building.

### `npx cap open`

Open the native project in the platform's IDE.

```bash
npx cap open android   # Opens Android Studio
npx cap open ios       # Opens Xcode
```

### `npx cap build`

Build the native project.

```bash
npx cap build <platform> [options]
```

Options:
- `--scheme <name>` — iOS scheme name.
- `--configuration <name>` — iOS build configuration (e.g., `Debug`, `Release`).
- `--flavor <name>` — Android flavor.
- `--androidreleasetype <type>` — Android release type (`AAB` or `APK`).
- `--signing <type>` — Android signing type.
- `--keystorepath <path>` — Android keystore path.
- `--keystorepass <pass>` — Android keystore password.
- `--keystorealias <alias>` — Android key alias.
- `--keystorealiaspass <pass>` — Android key alias password.

### `npx cap doctor`

Diagnose common configuration issues.

```bash
npx cap doctor [platform]
```

Checks environment, dependencies, and configuration. Use when something is not working as expected.

### `npx cap ls`

List installed Capacitor plugins.

```bash
npx cap ls [platform]
```

### `npx cap migrate`

Interactive command intended to migrate a project to a newer Capacitor version. **Do not invoke from an agent** — it requires interactive input and cannot be driven by a script. For all upgrade work, use the `capacitor-app-upgrades` skill, which performs the upgrade manually step-by-step.

## Lifecycle Hooks

Capacitor supports lifecycle hooks in `capacitor.config.ts` that run at specific points during CLI operations:

- `capacitor:copy:before` / `capacitor:copy:after`
- `capacitor:sync:before` / `capacitor:sync:after`
- `capacitor:update:before` / `capacitor:update:after`
- `capacitor:build:before` / `capacitor:build:after`
- `capacitor:run:before` / `capacitor:run:after`

Hooks are scripts defined in `package.json`:

```json
{
  "scripts": {
    "capacitor:sync:before": "echo 'Running before sync'"
  }
}
```
