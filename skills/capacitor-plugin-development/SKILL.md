---
name: capacitor-plugin-development
description: "Guides the agent through creating and maintaining Capacitor plugins from scratch. Covers scaffolding a new plugin project, designing the TypeScript API, implementing native iOS (Swift) and Android (Java/Kotlin) bridges, implementing the web layer, defining TypeScript type definitions, plugin configuration values, plugin hooks, development workflow with local testing, documentation generation, and publishing to npm. Do not use for installing existing plugins into an app, upgrading existing plugins to newer Capacitor versions, adding SPM support to plugins, or non-Capacitor plugin frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-plugin-development
---

# Capacitor Plugin Development

Create and maintain Capacitor plugins — scaffolding, native API bridging (iOS/Android), type definitions, testing, and publishing.

## Prerequisites

| Requirement | Version |
| --- | --- |
| Node.js | LTS (18+) |
| npm | 6+ |
| Xcode | 15+ (for iOS) |
| Android Studio | Hedgehog 2023.1.1+ (for Android) |
| Capacitor | 6+ |

## Agent Behavior

- **Auto-detect before asking.** Read `package.json`, inspect the directory structure, and infer the plugin's current state before prompting the user.
- **Guide step-by-step.** Walk the user through one phase at a time. Do not present multiple unrelated tasks simultaneously.
- **Skip what exists.** If the plugin is already scaffolded, skip scaffolding. If the iOS implementation already exists, skip to the next platform.
- **Ask which platforms to target.** If the user does not specify, implement all three (iOS, Android, Web).

## Procedures

### Step 1: Determine the Task

Ask the user what they want to do. Common tasks:

1. **Create a new plugin from scratch** — proceed to Step 2.
2. **Add a new method to an existing plugin** — skip to Step 5.
3. **Add a new platform implementation** — skip to the relevant platform step (Step 6, 7, or 8).
4. **Set up plugin configuration** — read `references/plugin-configuration.md` and apply.
5. **Set up plugin hooks** — read `references/testing-and-workflow.md` (Plugin Hooks section) and apply.
6. **Publish the plugin** — skip to Step 10.

If the user's intent is clear from context, skip this question and proceed directly.

### Step 2: Scaffold the Plugin

Read `references/scaffolding.md` and guide the user through generating a new plugin project using the Capacitor plugin generator.

Gather the following from the user (or infer from context):

- Plugin npm package name
- Android package ID
- Plugin class name
- Repository URL
- License
- Description

Run the generator and verify the scaffold with `npm run verify`.

### Step 3: Design the TypeScript API

Read `references/designing-api.md` and guide the user through defining the plugin interface.

1. Ask the user what methods the plugin should expose and what data each method accepts/returns.
2. Define the plugin interface and all supporting types in `src/definitions.ts`.
3. Register the plugin in `src/index.ts` using `registerPlugin()`.

Ensure:
- Every method and property has JSDoc comments with `@since` tags.
- Options and result types are defined as separate interfaces.
- String union types are used instead of TypeScript enums.

### Step 4: Implement the Web Layer

Read `references/web-guide.md` and implement the web layer in `src/web.ts`.

1. Extend `WebPlugin` and implement the plugin interface.
2. For methods that use Web APIs, implement the browser-based logic.
3. For methods that have no web equivalent, throw `this.unimplemented()`.
4. For methods that require a web API not available in all browsers, check for the API's existence and throw `this.unavailable()` if missing.

After implementation, build and verify:

```bash
npm run build
npm run verify:web
```

### Step 5: Define Method Signatures Across Platforms

Before implementing native code, determine the method types for each plugin method:

| Type | TypeScript | iOS | Android |
| --- | --- | --- | --- |
| Returns a value | `Promise<T>` | `CAPPluginReturnPromise` | `@PluginMethod()` |
| Returns void | `Promise<void>` | `CAPPluginReturnNone` | `@PluginMethod(returnType = PluginMethod.RETURN_NONE)` |
| Callback (repeated) | `Promise<string>` | `CAPPluginReturnCallback` | `@PluginMethod(returnType = PluginMethod.RETURN_CALLBACK)` |

Most methods use the "returns a value" type. Use "callback" only for continuous data streams (e.g., geolocation watching).

### Step 6: Implement the iOS Plugin

Read `references/ios-guide.md` and implement the iOS layer.

1. Create or update the **implementation class** (e.g., `ios/Sources/<ClassName>Plugin/Example.swift`) with the platform logic. Extend `NSObject` and mark methods with `@objc`.
2. Create or update the **plugin class** (e.g., `ios/Sources/<ClassName>Plugin/ExamplePlugin.swift`):
   - Extend `CAPPlugin` and conform to `CAPBridgedPlugin`.
   - Set `identifier`, `jsName`, and `pluginMethods` properties.
   - Implement each `@objc func` method, reading data from `CAPPluginCall` and calling `resolve()`, `reject()`, `unavailable()`, or `unimplemented()`.
3. If the plugin requires third-party iOS dependencies, add them to the `.podspec` file.
4. If the plugin requires permissions, implement `checkPermissions()` and `requestPermissions()`, and document the required `Info.plist` keys.

After implementation, verify:

```bash
npm run verify:ios
```

### Step 7: Implement the Android Plugin

Read `references/android-guide.md` and implement the Android layer.

1. Create or update the **implementation class** (e.g., `android/src/main/java/<package-path>/Example.java`) with the platform logic.
2. Create or update the **plugin class** (e.g., `android/src/main/java/<package-path>/ExamplePlugin.java`):
   - Extend `Plugin` and add `@CapacitorPlugin(name = "<JSName>")`.
   - Implement each `@PluginMethod` method, reading data from `PluginCall` and calling `resolve()`, `reject()`, `unavailable()`, or `unimplemented()`.
3. If the plugin requires third-party Android dependencies, add them to `android/build.gradle`.
4. If the plugin requires permissions:
   - Define permission aliases in the `@CapacitorPlugin` annotation.
   - Implement `checkPermissions()` and `requestPermissions()`.
   - Document which permissions the app developer must add to `AndroidManifest.xml`.

After implementation, verify:

```bash
npm run verify:android
```

### Step 8: Add Events (If Needed)

If the plugin emits events to JavaScript:

1. **TypeScript**: Add `addListener()` and `removeAllListeners()` methods to the plugin interface in `src/definitions.ts`. Define an event interface for each event type.
2. **Web**: Call `this.notifyListeners('eventName', data)` when the event occurs.
3. **iOS**: Call `self.notifyListeners("eventName", data: [...])` when the event occurs. Register observers in `load()` if listening to system notifications.
4. **Android**: Call `notifyListeners("eventName", jsObject)` when the event occurs. Override `handleOnConfigurationChanged()` or register broadcast receivers as needed.

Ensure the event name string is identical across all platforms and matches the `eventName` parameter in `addListener()`.

### Step 9: Generate Documentation and Verify

1. Add JSDoc comments to all methods and interfaces in `src/definitions.ts` if not already present.
2. Generate documentation:

```bash
npm run docgen
```

3. Run the full verification:

```bash
npm run verify
```

4. Lint and format:

```bash
npm run lint
npm run fmt
```

### Step 10: Publish

Read `references/publishing.md` and guide the user through the publishing process.

1. Verify `package.json` fields are correct (name, version, description, files, capacitor).
2. Run the pre-publish checklist.
3. Publish to npm:

```bash
npm publish --access public
```

## Error Handling

- **`npm init @capacitor/plugin@latest` fails**: Ensure Node.js LTS and npm 6+ are installed. Run `node -v` and `npm -v` to check.
- **`npm run verify:ios` fails with "module not found"**: Run `cd ios && pod install --repo-update && cd ..` to install CocoaPods dependencies.
- **`npm run verify:android` fails with build errors**: Open `android/` in Android Studio, sync Gradle, and check for missing dependencies or SDK version mismatches.
- **`npm run build` fails with TypeScript errors**: Check `src/definitions.ts` for type mismatches. Ensure the web implementation in `src/web.ts` correctly implements the plugin interface.
- **`registerPlugin()` name mismatch**: The first argument to `registerPlugin()` in `src/index.ts` must exactly match the `jsName` property on iOS and the `@CapacitorPlugin(name = "...")` value on Android. A mismatch causes the plugin to not load.
- **iOS: Methods not callable from JavaScript**: Ensure all plugin methods are marked with `@objc` and listed in the `pluginMethods` array of `CAPBridgedPlugin`.
- **Android: Methods not callable from JavaScript**: Ensure all plugin methods have the `@PluginMethod()` annotation and are `public`.
- **Events not received in JavaScript**: Verify the event name string is identical in the native `notifyListeners()` call and the TypeScript `addListener()` definition. Verify `addListener()` is called before the event fires.
- **Plugin configuration values not reading**: Verify the key names in `getConfig().getString("key")` match the keys in the Capacitor config file under `plugins.<PluginJSName>`.
- **`npm run docgen` produces empty output**: Ensure JSDoc comments are present on the plugin interface methods in `src/definitions.ts`, not on the implementation classes.

## Related Skills

- **`capacitor-plugin-spm-support`** — Add Swift Package Manager support to a Capacitor plugin.
- **`capacitor-plugin-upgrades`** — Upgrade a Capacitor plugin to a newer Capacitor major version.
- **`capacitor-plugins`** — Install and configure existing Capacitor plugins in an app project.
