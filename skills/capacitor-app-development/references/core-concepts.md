# Core Concepts

How Capacitor works and its fundamental architecture.

## What Is Capacitor

Capacitor is a cross-platform native runtime that makes it possible to build web apps that run natively on iOS, Android, and the web (as a Progressive Web App). The web app runs in a native WebView, and Capacitor provides a bridge between the web layer and native APIs.

## Architecture

A Capacitor app consists of three layers:

1. **Web layer** — The HTML/CSS/JavaScript app that runs inside a native WebView. This is the standard web app built with any framework (Angular, React, Vue, plain JS, etc.).
2. **Native bridge** — A communication layer that connects the web layer to native platform APIs. When JavaScript calls a Capacitor plugin method, the bridge serializes the call, passes it to the native side, executes the native code, and returns the result back to JavaScript.
3. **Native layer** — Platform-specific code (Swift/Objective-C on iOS, Kotlin/Java on Android) that implements native functionality exposed through plugins.

## How the Native Bridge Works

1. The web app calls a plugin method, e.g., `Camera.getPhoto()`.
2. The Capacitor bridge serializes the method name and arguments into a message.
3. The message is sent to the native side via the WebView's native-to-JS communication channel.
4. The native plugin implementation receives the call, executes the platform-specific code (e.g., opens the camera), and returns the result.
5. The result is serialized back to JavaScript and resolved as a Promise.

Data passed across the bridge must be JSON-serializable. Binary data (images, files) should be passed as file paths or base64 strings, though file paths are preferred for performance (see `references/file-handling.md`).

## Plugins

Plugins are the primary extension mechanism. Each plugin exposes a JavaScript API that maps to native implementations on each platform. Capacitor provides:

- **Official plugins** (`@capacitor/*`) — Maintained by the Capacitor team.
- **Community plugins** (`@capacitor-community/*`) — Maintained by the community.
- **Third-party plugins** (e.g., `@capawesome/*`, `@capacitor-firebase/*`) — Maintained by third parties.

Plugins that have no native implementation for a given platform fall back to a web implementation if one is provided.

## Web Views

- **iOS**: Uses `WKWebView`.
- **Android**: Uses Android System WebView (Chromium-based).
- **Electron**: Uses Chromium (via Electron).

The WebView loads the web app from the device's local filesystem (not from a remote server by default), which provides fast load times and offline capability.

## Project Structure

A typical Capacitor project has this structure:

```
my-app/
  android/                  # Native Android project (Android Studio)
  ios/                      # Native iOS project (Xcode)
    App/
      App/                  # iOS app source files
      App.xcodeproj/        # Xcode project file
      Podfile                # CocoaPods dependencies (if using CocoaPods)
  src/                      # Web app source code
  www/ or dist/ or build/   # Built web assets (output directory)
  capacitor.config.ts       # Capacitor configuration
  package.json
```

The `android/` and `ios/` directories are full native projects. They can be opened in Android Studio and Xcode respectively for native development, debugging, and building.

## Key Concepts

### Capacitor Config

The `capacitor.config.ts` (or `capacitor.config.json`) file configures the app's behavior. See `references/app-configuration.md` for details.

### Sync

`npx cap sync` is the primary command for keeping the native projects in sync with the web project. It copies built web assets to the native projects and updates native dependencies. Run it after:

- Installing or removing plugins
- Changing `capacitor.config.ts`
- Building new web assets

### Native Project Ownership

Unlike some frameworks, Capacitor treats the native projects as source-controlled first-class citizens. The `android/` and `ios/` directories are committed to version control and can be modified directly. This allows full access to native APIs, custom native code, and third-party native SDKs.
