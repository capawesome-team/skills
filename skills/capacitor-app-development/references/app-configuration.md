# App Configuration

Capacitor is configured through `capacitor.config.ts` (TypeScript) or `capacitor.config.json` (JSON) in the project root.

## Configuration File Formats

### TypeScript (recommended)

```typescript
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.example.myapp',
  appName: 'My App',
  webDir: 'dist',
};

export default config;
```

### JSON

```json
{
  "appId": "com.example.myapp",
  "appName": "My App",
  "webDir": "dist"
}
```

TypeScript is preferred because it enables IDE autocompletion and type checking for all configuration options.

## Core Properties

| Property | Type | Description |
| -------- | ---- | ----------- |
| `appId` | `string` | App identifier in reverse-domain format (e.g., `com.example.myapp`). Must match the native project's bundle/application ID. |
| `appName` | `string` | Display name of the app. |
| `webDir` | `string` | Directory containing the built web assets (e.g., `dist`, `www`, `build`). Relative to the project root. |
| `bundledWebRuntime` | `boolean` | Deprecated. Do not use. |

## Server Configuration

Control how the WebView loads content:

```typescript
const config: CapacitorConfig = {
  server: {
    url: 'http://192.168.1.68:8100',    // Load from a remote URL (for live reload)
    cleartext: true,                      // Allow HTTP (non-HTTPS) traffic
    androidScheme: 'https',               // URL scheme for Android (default: 'https')
    iosScheme: 'capacitor',               // URL scheme for iOS (default: 'capacitor')
    hostname: 'localhost',                 // Hostname for the local server
    allowNavigation: ['example.com'],      // Allowed navigation hosts
    errorPath: 'error.html',              // Custom error page
  },
};
```

- **`url`**: Override the default local file serving to load from a remote URL. Used primarily for live reload during development. **Do not commit to version control.**
- **`cleartext`**: Set to `true` to allow HTTP traffic. Required when `url` uses `http://`.
- **`androidScheme`**: The URL scheme used on Android. Default is `https`. Changing this affects CORS, localStorage, and cookie behavior.
- **`iosScheme`**: The URL scheme used on iOS. Default is `capacitor`.
- **`allowNavigation`**: List of hostnames or URL patterns the WebView is allowed to navigate to.

## Plugin Configuration

Configure plugin-specific options under the `plugins` key:

```typescript
const config: CapacitorConfig = {
  plugins: {
    SplashScreen: {
      launchShowDuration: 2000,
      backgroundColor: '#ffffff',
    },
    Keyboard: {
      resize: 'body',
      resizeOnFullScreen: true,
    },
    SystemBars: {
      insetsHandling: 'disable',
    },
  },
};
```

Each plugin documents its own configuration options. Refer to the plugin's documentation for available keys.

## Android-Specific Configuration

```typescript
const config: CapacitorConfig = {
  android: {
    path: 'android',                      // Path to the Android project
    buildOptions: {
      releaseType: 'AAB',                 // 'AAB' or 'APK'
      signingType: 'apksigner',
      keystorePath: '/path/to/keystore',
      keystorePassword: 'password',
      keystoreAlias: 'alias',
      keystoreAliasPassword: 'password',
    },
    flavor: 'production',                 // Default flavor
    allowMixedContent: false,             // Allow HTTP content in HTTPS WebView
    backgroundColor: '#ffffff',           // WebView background color
    loggingBehavior: 'debug',             // Logging level: 'none', 'debug', 'production'
    webContentsDebuggingEnabled: true,    // Enable Chrome DevTools for WebView (debug only)
  },
};
```

## iOS-Specific Configuration

```typescript
const config: CapacitorConfig = {
  ios: {
    path: 'ios',                          // Path to the iOS project
    scheme: 'App',                        // Xcode scheme
    backgroundColor: '#ffffff',           // WebView background color
    contentInset: 'automatic',            // 'automatic', 'scrollableAxes', 'never', 'always'
    limitsNavigationsToAppBoundDomains: false,
    preferredContentMode: 'mobile',       // 'mobile' or 'desktop'
    webContentsDebuggingEnabled: true,    // Enable Safari Web Inspector (debug only)
  },
};
```

## Environment-Specific Configuration

Use the TypeScript config format to conditionally apply settings based on environment variables:

```typescript
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.example.myapp',
  appName: 'My App',
  webDir: 'dist',
};

if (process.env.NODE_ENV === 'development') {
  config.server = {
    url: 'http://192.168.1.68:8100',
    cleartext: true,
  };
}

export default config;
```

This avoids accidentally shipping development settings in production builds.
