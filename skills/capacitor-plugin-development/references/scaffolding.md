# Scaffolding a New Capacitor Plugin

## Using the Plugin Generator

Run the official Capacitor plugin generator:

```bash
npm init @capacitor/plugin@latest
```

The generator prompts for the following values:

| Prompt | Description | Example |
| --- | --- | --- |
| **Plugin npm name** | The npm package name (scoped or unscoped). | `@capawesome/capacitor-example` |
| **Plugin id** | The Java/Kotlin package identifier for Android. | `io.capawesome.capacitorexample` |
| **Plugin class name** | The PascalCase class name used in native code. | `Example` |
| **Repository URL** | The Git repository URL. | `https://github.com/capawesome-team/capacitor-plugins` |
| **Author** | The package author. | `Capawesome` |
| **License** | The SPDX license identifier. | `MIT` |
| **Description** | A one-line package description. | `Capacitor plugin for example functionality.` |

Alternatively, pass options directly:

```bash
npx @capacitor/create-plugin \
  --name @capawesome/capacitor-example \
  --package-id io.capawesome.capacitorexample \
  --class-name Example \
  --repo "https://github.com/capawesome-team/capacitor-plugins" \
  --license "MIT" \
  --description "Capacitor plugin for example functionality."
```

## Generated Project Structure

After scaffolding, the plugin project has this structure:

```
<plugin-root>/
  android/
    src/main/java/<package-path>/
      ExamplePlugin.java        # Android plugin bridge
    build.gradle
  ios/
    Sources/ExamplePlugin/
      ExamplePlugin.swift       # iOS plugin bridge
      Example.swift             # iOS implementation class
    Tests/ExamplePluginTests/
      ExamplePluginTests.swift  # iOS test file
  src/
    definitions.ts              # TypeScript API definitions
    index.ts                    # Plugin registration and re-exports
    web.ts                      # Web/PWA implementation
  package.json
  tsconfig.json
  rollup.config.js
  <PluginName>.podspec          # CocoaPods spec for iOS
```

## Post-Scaffold Steps

After generation, install dependencies:

```bash
cd <plugin-root>
npm install
```

Verify the scaffold is correct:

```bash
npm run verify
```

This command builds the TypeScript, lints the code, and verifies the native projects compile.

## Available Package Scripts

The generated `package.json` includes these scripts:

| Script | Purpose |
| --- | --- |
| `npm run build` | Build the TypeScript source to ESM and bundle distributions. |
| `npm run verify` | Build, lint, and verify native projects. Combines `verify:ios`, `verify:android`, and `verify:web`. |
| `npm run lint` | Lint the TypeScript source with ESLint. |
| `npm run fmt` | Auto-format the TypeScript source with Prettier. |
| `npm run docgen` | Generate API documentation from JSDoc comments into the README. |
| `npm run verify:ios` | Verify the iOS project compiles. |
| `npm run verify:android` | Verify the Android project compiles. |
| `npm run verify:web` | Build and lint the web source. |
