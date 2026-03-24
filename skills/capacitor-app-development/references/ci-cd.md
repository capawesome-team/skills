# CI/CD for Capacitor Apps

Set up continuous integration and continuous deployment for Capacitor apps.

## Capawesome Cloud (Recommended)

Capawesome Cloud provides a managed CI/CD solution for Capacitor apps with native builds, live updates, and app store publishing.

For full setup and usage, use the **`capawesome-cloud`** skill.

Key features:
- Cloud-based native builds for iOS and Android (no local Xcode/Android Studio required for builds).
- Live updates (OTA) to push web layer changes without app store review.
- Automated app store publishing to Apple App Store and Google Play Store.
- Signing certificate management.
- Environment variables and secrets.
- CI/CD integration via the `@capawesome/cli`.

## Self-Hosted CI/CD

For self-hosted CI/CD pipelines (GitHub Actions, GitLab CI, Bitbucket Pipelines, etc.), the general workflow is:

### Web Build

```bash
npm ci
npm run build
```

### Android Build

```bash
npm ci
npm run build
npx cap sync android
cd android && ./gradlew assembleRelease
```

The APK is at `android/app/build/outputs/apk/release/app-release.apk`.

For AAB (Android App Bundle):

```bash
cd android && ./gradlew bundleRelease
```

The AAB is at `android/app/build/outputs/bundle/release/app-release.aab`.

Android builds require:
- Java (JDK 17 for Capacitor 7, JDK 21 for Capacitor 8).
- Android SDK and build tools.
- A signing keystore for release builds.

### iOS Build

```bash
npm ci
npm run build
npx cap sync ios
cd ios/App && xcodebuild -workspace App.xcworkspace -scheme App -configuration Release -archivePath build/App.xcarchive archive
cd ios/App && xcodebuild -exportArchive -archivePath build/App.xcarchive -exportOptionsPlist ExportOptions.plist -exportPath build/output
```

iOS builds require:
- macOS runner.
- Xcode.
- Apple signing certificate and provisioning profile installed on the runner.
- An `ExportOptions.plist` file specifying the export method and signing identity.

### Live Updates via CI/CD

Upload web bundles to Capawesome Cloud from CI:

```bash
npm ci
npm run build
npx @capawesome/cli login --token $CAPAWESOME_TOKEN
npx @capawesome/cli apps:bundles:create --app-id $APP_ID --path dist/
```

For full live update setup, use the **`capawesome-cloud`** skill.

## Automated Configuration with Trapeze

Use Trapeze (`@trapezedev/configure`) to automate native project configuration changes in CI/CD. This is useful for:

- Setting version numbers and build numbers per environment.
- Configuring different app IDs for staging vs. production.
- Injecting environment-specific values.

Define configuration in a YAML file (e.g., `trapeze.yaml`):

```yaml
platforms:
  android:
    versionName: $VERSION
    versionCode: $BUILD_NUMBER
    manifest:
      - file: AndroidManifest.xml
        target: manifest
        attrs:
          package: $APP_ID
  ios:
    version: $VERSION
    buildNumber: $BUILD_NUMBER
    targets:
      App:
        bundleId: $APP_ID
```

Apply in CI:

```bash
npx @trapezedev/configure run trapeze.yaml --diff
```

## Environment Variables

Use environment variables to manage secrets and configuration across environments. Never hardcode secrets in source code.

Common CI/CD environment variables for Capacitor projects:

| Variable | Purpose |
| -------- | ------- |
| `APP_ID` | Application identifier |
| `VERSION` | App version string |
| `BUILD_NUMBER` | Build number (auto-incremented) |
| `KEYSTORE_PATH` | Path to Android signing keystore |
| `KEYSTORE_PASSWORD` | Android keystore password |
| `KEY_ALIAS` | Android key alias |
| `KEY_PASSWORD` | Android key alias password |
| `CAPAWESOME_TOKEN` | Capawesome Cloud auth token |
