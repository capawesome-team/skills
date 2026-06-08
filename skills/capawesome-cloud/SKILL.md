---
name: capawesome-cloud
description: "Guides the agent through setting up and using Capawesome Cloud for Capacitor and Cordova apps. Covers three core workflows: (1) Native Builds — cloud builds for iOS and Android (Capacitor, Cordova, or native projects), signing certificates, environments, Trapeze configuration, and build artifacts; (2) Live Updates — OTA updates via the @capawesome/capacitor-live-update or @capawesome/cordova-live-update plugin, channels, versioning, rollbacks, and code signing; (3) App Store Publishing — automated submissions to Apple App Store (TestFlight) and Google Play Store. Includes CI/CD integration for all workflows. Do not use for non-Capacitor, non-Cordova mobile frameworks such as React Native or Flutter."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capawesome-cloud
---

# Capawesome Cloud

Set up and manage native builds, live updates, and app store publishing for Capacitor and Cordova apps using Capawesome Cloud.

## Prerequisites

1. A [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.
2. A **Capacitor 6/7/8** or **Cordova** app. Notes:
   - **Live Updates**: supported on both Capacitor and Cordova.
   - **Native Builds**: supported on Capacitor, Cordova, and native iOS/Android projects.
   - **App Store Publishing**: framework-agnostic — works with any native build produced by Native Builds.
3. Node.js and npm installed.
4. For **Native Builds**: The app must be in a Git repository (GitHub, GitLab, Bitbucket, or Azure DevOps).
5. For **Apple App Store Publishing**: An active [Apple Developer Program](https://developer.apple.com/programs/) membership and an app created in [App Store Connect](https://appstoreconnect.apple.com).
6. For **Google Play Store Publishing**: A [Google Play Developer](https://play.google.com/console) account and an app created in Google Play Console with at least one version uploaded manually.

## General Rules

Before running any `@capawesome/cli` command for the first time, run it with the `--help` flag to review all available options.

## Procedures

### Step 1: Authenticate with Capawesome Cloud

```bash
npx @capawesome/cli login
```

For CI/CD, use token-based auth:

```bash
npx @capawesome/cli login --token <token>
```

### Step 2: Create an App in Capawesome Cloud

Skip if the user already has an app ID.

```bash
npx @capawesome/cli apps:create
```

The CLI prompts for organization and app name, then outputs the **app ID** (UUID). Save for subsequent steps.

The app type defaults to `capacitor`. For a Cordova app, pass `--type cordova` (allowed values: `android`, `capacitor`, `cordova`, `ios`):

```bash
npx @capawesome/cli apps:create --type cordova
```

### Step 3: Identify Required Feature(s)

Ask the user which Capawesome Cloud feature(s) to set up:

1. **Native Builds** — Build iOS and Android apps in the cloud.
2. **Live Updates** — Push OTA web updates to deployed apps.
3. **App Store Publishing** — Automate submissions to Apple App Store or Google Play Store.

The user may select one or more features. Proceed to the corresponding sections below.

### Native Builds

Read `references/native-builds.md` for the full native builds setup and usage procedure. This covers:

- Connecting a Git repository
- Uploading signing certificates (Android keystores, iOS .p12 + provisioning profiles)
- Configuring environments and secrets
- Configuring build settings (monorepos, custom build commands)
- Triggering builds
- Monitoring and downloading build artifacts
- Configuring native configuration overwriting with Trapeze
- Setting up CI/CD pipelines

### Live Updates

Read `references/live-updates.md` first — it detects the framework (Capacitor or Cordova) and routes to the matching procedure (`live-updates-capacitor.md` or `live-updates-cordova.md`). Both procedures cover:

- Installing the Live Update plugin (`@capawesome/capacitor-live-update` for Capacitor, `@capawesome/cordova-live-update` for Cordova)
- Configuring the plugin (Capacitor config or Cordova `config.xml` preferences)
- Adding rollback protection
- Adding update logic (Always Latest, Manual Sync, Force Update)
- Configuring iOS Privacy Manifest
- Configuring version handling (versioned channels, versioned bundles)
- Testing the setup

### App Store Publishing

Read `references/app-store-publishing.md` for the full app store publishing setup and usage procedure. This covers:

- Creating Apple App Store destinations (API Key or Apple ID auth)
- Creating Google Play Store destinations
- Deploying builds to destinations
- Building and deploying in one step

## Error Handling

### Native Builds

- `invalid source release: 21` → Set `JAVA_VERSION` env var to `17` or `21`. Read `references/build-troubleshooting.md`.
- `JavaScript heap out of memory` → Set `NODE_OPTIONS` env var to `--max-old-space-size=4096`. Read `references/build-troubleshooting.md`.
- Authentication errors → Re-run `npx @capawesome/cli login`. For CI/CD, verify the token.
- Missing signing certificate → Upload via `apps:certificates:create`. Read `references/certificates-android.md` or `references/certificates-ios.md`.
- Expired provisioning profile → Regenerate in Apple Developer Portal and re-upload. Read `references/certificates-ios.md`.
- Web build step fails → Ensure `package.json` has `capawesome:build` or `build` script. Read `references/build-configuration.md`.

### Live Updates

- `npx cap sync` fails (Capacitor) → Verify plugin version matches Capacitor version in `package.json`.
- Bundles not applied → Ensure `ready()` is called before `readyTimeout`/`READY_TIMEOUT` expires.
- App reverts to default bundle after restart → `ready()` likely not called. Add it early in app init (in Cordova, inside the `deviceready` listener).
- Upload auth errors → Re-run `npx @capawesome/cli login`.
- Updates not detected with `autoUpdateStrategy: "background"` / `AUTO_UPDATE_STRATEGY=background` → Updates only checked if last check was >15 min ago. Force-close and restart.
- **Cordova:** preference changes have no effect → Cordova reads preferences only on platform add. Run `cordova platform rm <platform>` then `cordova platform add <platform>`.
- **Cordova:** updates never apply / WebView errors → The plugin requires the default custom WebView scheme. Ensure `config.xml` does not set `Scheme=file` or `AndroidInsecureFileModeEnabled=true`.
- Read `references/live-update-plugin-api.md` for the full SDK API reference.
- Read `references/live-update-faq.md` for compliance, billing, and limitations.

### App Store Publishing

- Authentication errors → Re-run `npx @capawesome/cli login`.
- Destination creation fails → Verify credentials. Read `references/apple-app-store-credentials.md` or `references/google-play-store-credentials.md`.
- iOS build not appearing in TestFlight → Build processing may take time. Common causes: build number not incremented, missing Privacy Descriptions in `Info.plist`, insufficient permissions.
- Google Play deployment fails → Ensure first version was uploaded manually. Verify service account has Release permissions.
- Deployment timeout → Use `--detached` flag and check logs with `apps:deployments:logs`.
- `apps:deployments:create` fails with "build not found" → Ensure the build completed successfully.
- Cancel a stuck deployment → `npx @capawesome/cli apps:deployments:cancel --app-id <APP_ID> --deployment-id <DEPLOYMENT_ID>`.

## Related Skills

- **`capacitor-app-development`** — For general Capacitor development topics, CI/CD patterns, and troubleshooting.
- **`capawesome-cli`** — For the full Capawesome CLI command reference, project configuration, and CI/CD integration details.
- **`capacitor-plugins`** — For installing and configuring Capacitor plugins, including the `@capawesome/capacitor-live-update` plugin.
- **`ionic-appflow-migration`** — For migrating existing Ionic Appflow projects to Capawesome Cloud.
- **`capacitor-in-app-purchases`** — For setting up in-app purchases and subscriptions before publishing to app stores.
