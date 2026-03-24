# Testing and Development Workflow

## Local Testing

### Link the Plugin to a Capacitor App

To test the plugin locally, install it from the local file system in a Capacitor app project:

```bash
cd /path/to/capacitor-app
npm install ../path/to/plugin-root
npx cap sync
```

This creates a symlink in `node_modules` pointing to the local plugin directory.

### Rebuild After Changes

After modifying the plugin TypeScript source, rebuild:

```bash
cd /path/to/plugin-root
npm run build
```

Then sync and run the app:

```bash
cd /path/to/capacitor-app
npx cap sync
npx cap run android
npx cap run ios
```

For native code changes (Swift/Java/Kotlin), no `npm run build` is needed. Just run `npx cap sync` and rebuild the native project.

### Unlink the Plugin

To remove the local plugin link:

```bash
cd /path/to/capacitor-app
npm uninstall <plugin-package-name>
```

## Verification

Run the built-in verification command to check that all platforms build correctly:

```bash
npm run verify
```

This runs `verify:ios`, `verify:android`, and `verify:web` in sequence.

To verify individual platforms:

```bash
npm run verify:ios
npm run verify:android
npm run verify:web
```

## Linting and Formatting

```bash
npm run lint       # Check for lint errors
npm run fmt        # Auto-format with Prettier
```

## Unit Tests (iOS)

The scaffold generates a test file at `ios/Tests/<ClassName>PluginTests/<ClassName>PluginTests.swift`. Run iOS tests via:

```bash
npm run verify:ios
```

Or open `ios/Plugin.xcworkspace` in Xcode and run the test target.

## Documentation Generation

Add JSDoc comments to all public methods and interfaces in `src/definitions.ts`:

```typescript
export interface ExamplePlugin {
  /**
   * Echo the provided value back.
   *
   * @since 1.0.0
   */
  echo(options: EchoOptions): Promise<EchoResult>;
}
```

Then generate the API documentation:

```bash
npm run docgen
```

This reads the JSDoc comments from `src/definitions.ts` and writes a formatted API table into the README.md between the `docgen` markers.

## Plugin Hooks

Starting with Capacitor 6.1, plugins can hook into Capacitor CLI commands by adding scripts to `package.json`:

```json
{
  "scripts": {
    "capacitor:copy:before": "node scripts/before-copy.js",
    "capacitor:copy:after": "node scripts/after-copy.js",
    "capacitor:update:before": "node scripts/before-update.js",
    "capacitor:update:after": "node scripts/after-update.js",
    "capacitor:sync:before": "node scripts/before-sync.js",
    "capacitor:sync:after": "node scripts/after-sync.js"
  }
}
```

Available hook events:

| Event | Triggered |
| --- | --- |
| `capacitor:copy:before` | Before `npx cap copy` copies web assets to native projects. |
| `capacitor:copy:after` | After `npx cap copy` completes. |
| `capacitor:update:before` | Before `npx cap update` updates native dependencies. |
| `capacitor:update:after` | After `npx cap update` completes. |
| `capacitor:sync:before` | Before `npx cap sync` (which runs copy + update). |
| `capacitor:sync:after` | After `npx cap sync` completes. |

The `$CAPACITOR_PLATFORM_NAME` environment variable is available in hook scripts and contains the platform being processed (e.g., `android`, `ios`).

## Configuration Values

Plugins can define runtime configuration values that app developers set in their Capacitor config file. Read `references/plugin-configuration.md` for details.
