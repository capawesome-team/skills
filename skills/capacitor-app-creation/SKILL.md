---
name: capacitor-app-creation
description: "Guides the agent through creating a new Capacitor app from scratch. Covers project scaffolding with the Capacitor CLI, configuring the app (appId, appName, webDir), adding native platforms (iOS, Android), and syncing. Includes decision points for Ionic Framework integration, live updates, and CI/CD setup. Do not use for upgrading existing Capacitor apps, migrating from other frameworks, or plugin installation."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-app-creation
---

# Capacitor App Creation

Create a new Capacitor app from scratch, including project scaffolding, platform setup, and optional integrations.

## Prerequisites

1. **Node.js 22+** installed. Verify with `node --version`.
2. For **iOS**: macOS with Xcode 26.0+ installed. Install Xcode Command Line Tools: `xcode-select --install`.
3. For **Android**: Android Studio 2025.2.1+ installed with Android SDK (API 24+).

## Agent Behavior

- **Guide step-by-step.** Walk the user through the process one step at a time. Never present multiple unrelated questions at once.
- **One decision at a time.** When a step requires user input, ask that single question, wait for the answer, then continue.
- **Present clear options.** Provide concrete choices (e.g., "Do you want to add the iOS platform? (yes/no)") instead of open-ended questions.

## Procedures

### Step 1: Check for Ionic Framework

**Before** creating the app, ask the user:

> Do you want to use the Ionic Framework for UI components and theming? (yes/no)

If the user answers **yes**, stop this skill and switch to the **`ionic-app-creation`** skill instead. If **no**, continue with Step 2.

### Step 2: Create the Web App

Ask the user which web framework they want to use. Present these options:

1. **Angular**
2. **React**
3. **Vue**
4. **None** (vanilla JS/TS or existing web app)

Based on the selection, scaffold the web project:

#### Angular

```bash
npm install -g @angular/cli
ng new my-app
cd my-app
```

The default web asset directory for Angular is `dist/my-app/browser`. Adjust the directory name based on the actual project name.

#### React

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
```

The default web asset directory for React (Vite) is `dist`.

#### Vue

```bash
npm create vite@latest my-app -- --template vue-ts
cd my-app
npm install
```

The default web asset directory for Vue (Vite) is `dist`.

#### None (vanilla or existing web app)

If the user has an existing web app, confirm it meets these requirements:

1. A `package.json` file exists at the project root.
2. A build output directory exists (e.g., `dist`, `www`, `build`).
3. An `index.html` file exists at the root of the build output directory.
4. The `index.html` file contains a `<head>` tag (required for Capacitor plugins to work).

If the user wants a vanilla project:

```bash
mkdir my-app && cd my-app
npm init -y
mkdir www
```

Then create `www/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>My App</title>
</head>
<body>
  <h1>My App</h1>
</body>
</html>
```

The web asset directory is `www`.

### Step 3: Install Capacitor

Navigate to the project root directory and install the Capacitor core packages:

```bash
npm install @capacitor/core
npm install -D @capacitor/cli
```

### Step 4: Initialize Capacitor

Run the Capacitor init command with the app name, app ID, and web asset directory:

```bash
npx cap init <appName> <appID> --web-dir <webDir>
```

- **`<appName>`**: Ask the user for the display name of the app (e.g., `"My App"`).
- **`<appID>`**: Ask the user for the app ID in reverse-domain format (e.g., `com.example.myapp`).
- **`<webDir>`**: Use the web asset directory determined in Step 2 (e.g., `dist`, `www`, `dist/my-app/browser`).

This creates a `capacitor.config.ts` file in the project root.

### Step 5: Build the Web App

Before adding native platforms, build the web app so the output directory exists:

#### Angular

```bash
ng build
```

#### React / Vue (Vite)

```bash
npm run build
```

#### Vanilla

No build step needed if using a static `www/` directory.

### Step 6: Add Native Platforms

Ask the user which platforms to add:

> Which platforms do you want to add? (android/ios/both)

#### Add Android

```bash
npm install @capacitor/android
npx cap add android
```

This creates the `android/` directory with a native Android project.

#### Add iOS

```bash
npm install @capacitor/ios
npx cap add ios
```

This creates the `ios/` directory with a native iOS project. By default, Capacitor 8 uses Swift Package Manager (SPM) for iOS dependency management.

### Step 7: Sync the Project

Sync the web assets and native dependencies:

```bash
npx cap sync
```

This copies the built web assets into each native platform project and installs native dependencies (Gradle for Android, SPM for iOS).

### Step 8: Run the App

Run the app on a connected device or emulator/simulator:

#### Android

```bash
npx cap run android
```

This prompts for a target device (emulator or physical device).

#### iOS

```bash
npx cap run ios
```

This prompts for a target simulator or physical device.

### Step 9: Post-Creation Options

After the app is running, ask the user:

> Do you want to set up Live Updates for over-the-air updates? (yes/no)

If **yes**, refer the user to the **`capawesome-cloud`** skill for live update setup.

Then ask:

> Do you want to set up CI/CD for native builds and app store publishing? (yes/no)

If **yes**, refer the user to the **`capawesome-cloud`** skill for CI/CD setup.

## Error Handling

- **`npx cap init` fails with "already initialized"**: A `capacitor.config.ts` or `capacitor.config.json` file already exists. The project is already a Capacitor project. Delete the existing config file if re-initialization is intended.
- **`npx cap add` fails with "platform already exists"**: The `android/` or `ios/` directory already exists. Remove the directory first: `rm -rf android` or `rm -rf ios`, then re-run `npx cap add`.
- **`npx cap sync` fails with "could not find the web assets directory"**: The web app has not been built yet, or the `webDir` in `capacitor.config.ts` points to a non-existent directory. Run the web app build command first (e.g., `npm run build`, `ng build`).
- **iOS build fails with "no such module"**: Run `npx cap sync ios` to ensure SPM dependencies are resolved.
- **Android build fails with SDK errors**: Verify that Android SDK is installed and `ANDROID_HOME` or `ANDROID_SDK_ROOT` environment variable is set. Open Android Studio SDK Manager to install missing SDK versions.
- **`index.html` missing `<head>` tag**: Capacitor plugins inject scripts into the `<head>` tag. Ensure the `index.html` in the web asset directory contains a `<head>` element.

## Related Skills

- **`ionic-app-creation`** — Create a new Ionic app with UI components and theming instead of a plain Capacitor app.
- **`capacitor-app-development`** — General Capacitor app development guidance after app creation.
- **`capacitor-angular`** — Angular-specific patterns and best practices for Capacitor apps.
- **`capacitor-react`** — React-specific patterns and best practices for Capacitor apps.
- **`capacitor-vue`** — Vue-specific patterns and best practices for Capacitor apps.
- **`capacitor-plugins`** — Install and configure Capacitor plugins after app creation.
- **`capawesome-cloud`** — Set up live updates, native builds, and CI/CD for the app.
