# Advanced Topics

## Channels

Channels distribute different bundles to different user groups (e.g., `production`, `staging`, `beta`). A `default` channel is created automatically.

Create a channel:

```bash
npx @capawesome/cli apps:channels:create
```

Set a channel in the app:

```typescript
await LiveUpdate.setChannel({ channel: "beta" });
```

Or pass it to `sync()`:

```typescript
await LiveUpdate.sync({ channel: "production" });
```

Read `cli-commands.md` for all channel management commands.

## Versioned Channels

Restrict bundles to specific native versions by tying channel names to version codes.

**Android** (`android/app/build.gradle`):

```groovy
android {
    defaultConfig {
        resValue "string", "capawesome_live_update_default_channel", "production-" + defaultConfig.versionCode
    }
}
```

**iOS** (`ios/App/App/Info.plist`):

```xml
<key>CapawesomeLiveUpdateDefaultChannel</key>
<string>production-$(CURRENT_PROJECT_VERSION)</string>
```

## Bundle Versioning

Restrict bundles to native version ranges when uploading:

```bash
npx @capawesome/cli apps:liveupdates:upload --android-min 1 --android-max 5 --ios-min 1.0.0 --ios-max 1.0.5
```

## Rollbacks

Manual rollback via CLI:

```bash
npx @capawesome/cli apps:liveupdates:rollback --channel production --steps 1
```

Enable auto-blocking of rolled-back bundles:

```typescript
LiveUpdate: {
  autoBlockRolledBackBundles: true,
  readyTimeout: 10000,
}
```

## Gradual Rollouts

Upload a bundle with a rollout percentage:

```bash
npx @capawesome/cli apps:liveupdates:upload --rollout-percentage 10
```

Update the rollout:

```bash
npx @capawesome/cli apps:liveupdates:rollout --channel production --percentage 50
```

## Code Signing

Generate a signing key pair:

```bash
npx @capawesome/cli apps:liveupdates:generatesigningkey
```

Upload a signed bundle:

```bash
npx @capawesome/cli apps:liveupdates:upload --private-key private.pem
```

Configure the plugin with the public key:

```typescript
LiveUpdate: {
  publicKey: "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----",
}
```

## Self-Hosting

Register a self-hosted bundle URL:

```bash
npx @capawesome/cli apps:liveupdates:register --url https://example.com/bundle.zip
```

## CI/CD Integration

Read `live-update-ci-cd-integrations.md` for GitHub Actions, GitLab CI, Azure DevOps, and Bitbucket Pipelines examples.

## Delta Updates

Use manifest artifact type to download only changed files:

```bash
npx @capawesome/cli apps:liveupdates:upload --artifact-type manifest
```

Generate a manifest first if needed:

```bash
npx @capawesome/cli apps:liveupdates:generatemanifest --path dist
```

## Debugging

The Live Update SDK logs to Android Logcat and iOS Xcode console. View server-side logs in the [Capawesome Cloud Console](https://console.cloud.capawesome.io) under the "Logs" section of the app.

## Limitations

- Live updates only support **binary-compatible changes** (HTML, CSS, JS, images). Native code changes (Java, Swift, CocoaPods, Gradle) require a full app store submission.
- Maximum bundle size on Capawesome Cloud is **1 GB**. Use `manifest` artifact type for larger bundles.
- Live updates are compliant with both Apple App Store and Google Play policies.
