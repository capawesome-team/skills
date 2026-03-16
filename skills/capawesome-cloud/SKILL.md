---
name: capawesome-cloud
description: Guides the agent through setting up and using Capawesome Cloud for Capacitor apps. Covers three core workflows: (1) Native Builds — cloud builds for iOS and Android, signing certificates, environments, Trapeze configuration, and build artifacts; (2) Live Updates — OTA updates via the @capawesome/capacitor-live-update plugin, channels, versioning, rollbacks, and code signing; (3) App Store Publishing — automated submissions to Apple App Store (TestFlight) and Google Play Store. Includes CI/CD integration for all workflows. Do not use for non-Capacitor mobile frameworks.
---

# Capawesome Cloud

Set up and manage native builds, live updates, and app store publishing for Capacitor apps using Capawesome Cloud.

## Prerequisites

1. A [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.
2. A **Capacitor 6, 7, or 8** app.
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

Read `references/live-updates.md` for the full live updates setup and usage procedure. This covers:

- Installing the `@capawesome/capacitor-live-update` plugin
- Configuring the plugin in Capacitor config
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

- `npx cap sync` fails → Verify plugin version matches Capacitor version in `package.json`.
- Bundles not applied → Ensure `LiveUpdate.ready()` is called before `readyTimeout` expires.
- App reverts to default bundle after restart → `ready()` likely not called. Add it early in app init.
- Upload auth errors → Re-run `npx @capawesome/cli login`.
- Updates not detected with `autoUpdateStrategy: "background"` → Updates only checked if last check was >15 min ago. Force-close and restart.
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
