# Grafana Faro

Unofficial Capacitor plugin for [Grafana Faro](https://grafana.com/oss/faro/) frontend observability. Capture logs, events, errors, measurements, and native crashes, with user/session/view metadata across Android, iOS, and Web.

**Package:** `@capawesome/capacitor-grafana-faro`
**Platforms:** Android, iOS, Web
**Capawesome Insiders:** No

> **Experimental:** This plugin is in early development and its API may change.

## Installation

```bash
npm install @capawesome/capacitor-grafana-faro @grafana/faro-web-sdk
npx cap sync
```

`@grafana/faro-web-sdk` is a required peer dependency.

## Configuration

### iOS

Native crash reporting depends on PLCrashReporter, which requires iOS 15.0+. Set the deployment target to `15.0`:

- In `ios/App/Podfile`, set `platform :ios, '15.0'`.
- In `ios/App/App.xcodeproj/project.pbxproj`, replace all `IPHONEOS_DEPLOYMENT_TARGET = <older>;` with `IPHONEOS_DEPLOYMENT_TARGET = 15.0;`.

> PLCrashReporter installs global crash handlers. Running this plugin alongside another crash reporter that also installs handlers (e.g. Firebase Crashlytics, Sentry) can cause one or both to lose crash reports.

### capacitor.config (optional)

The SDK can auto-initialize at startup if `url` is configured. Configure via `capacitor.config.ts`:

```typescript
/// <reference types="@capawesome/capacitor-grafana-faro" />

import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  plugins: {
    GrafanaFaro: {
      url: 'https://faro-collector-prod-us-central-0.grafana.net/collect/REPLACE_ME',
      appName: 'my-app',
      appEnvironment: 'production',
      appVersion: '1.0.0',
      appNamespace: 'com.example.app',
      apiKey: 'YOUR_API_KEY',
      instrumentations: {
        nativeCrashReporting: true,
        anrTracking: true,
      },
    },
  },
};

export default config;
```

Alternatively, call `GrafanaFaro.initialize()` at runtime instead of using Capacitor config.

## Usage

### Initialize (runtime)

When initializing at runtime, the app metadata is passed as a nested `app` object.

```typescript
import { GrafanaFaro } from '@capawesome/capacitor-grafana-faro';

await GrafanaFaro.initialize({
  app: {
    environment: 'production',
    name: 'my-app',
    version: '1.0.0',
  },
  instrumentations: {
    anrTracking: true,
    console: true,
    errors: true,
    nativeCrashReporting: true,
    performance: true,
    view: true,
    webVitals: true,
  },
  url: 'https://faro-collector-prod-us-central-0.grafana.net/collect/REPLACE_ME',
});
```

### Push logs, events, errors, and measurements

```typescript
import { GrafanaFaro } from '@capawesome/capacitor-grafana-faro';

await GrafanaFaro.pushLog({
  level: 'info',
  message: 'User pressed sign-in button',
});

await GrafanaFaro.pushEvent({
  name: 'sign_in_started',
  attributes: { provider: 'google' },
});

await GrafanaFaro.pushError({
  type: error.name,
  value: error.message,
});

await GrafanaFaro.pushMeasurement({
  type: 'sign_in_duration',
  values: { duration_ms: 320 },
});
```

### User, session, and view metadata

```typescript
import { GrafanaFaro } from '@capawesome/capacitor-grafana-faro';

await GrafanaFaro.setUser({ id: 'user-123', username: 'jane', email: 'jane@example.com' });
await GrafanaFaro.resetUser();

await GrafanaFaro.setSession({ attributes: { plan: 'premium' } });
const { session } = await GrafanaFaro.getSession();
await GrafanaFaro.resetSession();

await GrafanaFaro.setView({ name: 'SignInView' });
const { view } = await GrafanaFaro.getView();
```

### Pause and resume telemetry

```typescript
import { GrafanaFaro } from '@capawesome/capacitor-grafana-faro';

await GrafanaFaro.pause();
await GrafanaFaro.unpause();
```

## Notes

- Call `initialize()` before any other method (unless auto-initialized via Capacitor config).
- Native crash reporting uses PLCrashReporter on iOS and `ApplicationExitInfo` on Android. ANR detection is Android-only.
- Web auto-instrumentation captures uncaught errors, console output, Core Web Vitals, route changes, and network performance.
- Use `sessionSamplingRate` in the initialize options to control telemetry data volume.
- This project is not affiliated with Grafana Labs.
