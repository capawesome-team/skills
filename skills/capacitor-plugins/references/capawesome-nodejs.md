# Node.js

Run a full Node.js runtime inside a Capacitor app and exchange messages with it from the web layer.

**Package:** `@capawesome/capacitor-nodejs`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/nodejs/

## Installation

```bash
npm install @capawesome/capacitor-nodejs
npx cap sync
```

The plugin embeds the [Node.js for Mobile Apps](https://github.com/nodejs-mobile/nodejs-mobile) runtime binaries. They are **not** part of the npm package: the Android binaries are downloaded by the Gradle build, the iOS binaries during `npm install` (macOS only). Set `CAPACITOR_NODEJS_SKIP_DOWNLOAD=1` to skip the iOS download.

## Configuration

### Android

#### Variables

Optionally define in `android/variables.gradle`:

- `$nodejsMobileVersion` version of the Node.js for Mobile Apps runtime (default: `18.20.4-capawesome.1`, a 16 KB page size compatible build)
- `$nodejsMobileAndroidUrl` download URL of the Android runtime binaries (default: GitHub release of `$nodejsMobileVersion`)
- `$nodejsMobileAndroidSha256` SHA-256 checksum of that download

### iOS

Only **Swift Package Manager** is supported. CocoaPods is not supported.

If dependencies were installed with `--ignore-scripts`, the iOS runtime binaries are missing. Download them manually before building:

```bash
node node_modules/@capawesome/capacitor-nodejs/scripts/postinstall.js
```

### Plugin Configuration

In `capacitor.config.ts`:

```typescript
/// <reference types="@capawesome/capacitor-nodejs" />

const config: CapacitorConfig = {
  plugins: {
    Nodejs: {
      nodeDir: 'custom-nodejs', // Node.js project dir, relative to `webDir` (default: 'nodejs')
      startMode: 'manual', // 'auto' | 'manual' (default: 'auto')
    },
  },
};
```

## Usage

### Set up the Node.js project

The plugin runs the Node.js project located in the `nodejs` directory (see `nodeDir`) **inside the Capacitor `webDir`**. Make sure the web build copies it there, e.g. by placing it in the `public` directory of the web project:

```
my-app
├── capacitor.config.json   // webDir: 'dist'
└── src
    └── public
        └── nodejs
            ├── package.json   // "main": "index.js"
            └── index.js
```

The `main` field of the project's `package.json` defines the script that is executed.

### Communicate from Node.js

Inside the Node.js script, the built-in `bridge` module provides the channel to the Capacitor app:

```js
const { app, channel } = require('bridge');

channel.on('my-event', (...args) => {
  channel.post('my-response', 'Hello from Node.js!');
});

// Writable directory for persistent file storage.
const dataDir = app.datadir();

app.on('pause', (pauseLock) => pauseLock.release());
app.on('resume', () => {});
```

### Exchange messages from the Capacitor app

```typescript
import { Nodejs } from '@capawesome/capacitor-nodejs';

await Nodejs.addListener('ready', () => console.log('Runtime is ready.'));
await Nodejs.addListener('message', (event) => {
  console.log('Received:', event.eventName, event.args);
});

const { ready } = await Nodejs.isReady();

await Nodejs.send({
  eventName: 'my-event',
  args: ['Hello from Capacitor!'],
});
```

### Start the runtime manually

```typescript
import { Nodejs } from '@capawesome/capacitor-nodejs';

// Only available if `startMode` is set to 'manual'.
await Nodejs.start({
  args: ['--option', 'value'],
  env: { MY_ENV_VAR: 'value' },
  script: 'custom-main.js',
});
```

## Notes

- The runtime is considered ready as soon as the Node.js project has required the `bridge` module. Wait for `isReady()`/the `ready` event before calling `send(...)`.
- Store persistent data in `app.datadir()`. The Node.js project directory itself may be overwritten during app updates.
- To use npm packages, run `npm install --omit=dev` inside the Node.js project directory before building the web project. Bundling the project into a single file (e.g. with esbuild) improves startup time.
- Message arguments must be JSON-serializable.
- **Single instance**: the runtime can be started only once per app launch — there is no `stop()` and no restart. Send a custom `shutdown` message instead if work must be stopped.
- `child_process` is not supported. `process.exit()` is not allowed by the App Store guidelines.
- On iOS the JS engine runs interpreter-only (no JIT), so execution is slower than on Android.
- Embedding the runtime increases the app size by several tens of megabytes per CPU architecture.
- Native addons work on Android only, and only as prebuilds (see `node-gyp-build`).
- The bundled Node.js version is `18.20.4`, the latest available from Node.js for Mobile Apps.
