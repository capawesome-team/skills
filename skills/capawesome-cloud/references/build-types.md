# Build Types

Build types determine how the app is compiled, signed, and distributed.

## Android

| Type      | Signed With      | Use Case                                       |
| --------- | ---------------- | ---------------------------------------------- |
| `debug`   | Debug keystore   | Development/testing, not optimized              |
| `release` | Release keystore | Production, required for Google Play Store      |
| Custom    | As configured    | Set `ANDROID_BUILD_TYPE` env var (e.g., `staging`) |

### Custom Build Types

To use a custom Android build type (e.g., `staging` or `qa`), set the `ANDROID_BUILD_TYPE` environment variable to the custom type name. The type must be defined in the app's `build.gradle` file.

### Product Flavors

To build a specific Android product flavor, set the `ANDROID_FLAVOR` environment variable (e.g., `free` or `paid`).

## iOS

| Type          | Signed With               | Use Case                                                 |
| ------------- | ------------------------- | -------------------------------------------------------- |
| `simulator`   | None                      | iOS Simulator testing, cannot install on real devices     |
| `development` | Development certificate   | Physical device testing, devices must be registered       |
| `ad-hoc`      | Distribution certificate  | Beta testing, up to 100 registered devices                |
| `app-store`   | Distribution certificate  | App Store/TestFlight submission (auto-uploaded to TestFlight) |
| `enterprise`  | Enterprise certificate    | In-house distribution (requires Enterprise Program)       |

### Custom Scheme

To build with a specific iOS scheme, set the `IOS_SCHEME` environment variable.

## Selecting a Build Type

When triggering a build via CLI:

```bash
# Android debug build
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type debug --git-ref main

# Android release build
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref main --certificate "Production Android Key"

# iOS simulator build
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --type simulator --git-ref main

# iOS App Store build
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --type app-store --git-ref main --certificate "Production iOS Certificate"
```

For initial testing, use `debug` (Android) or `simulator` (iOS) to skip certificate setup.
