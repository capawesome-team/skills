---
name: capawesome-app-store-publishing
description: Guides the agent through setting up and managing App Store Publishing using Capawesome Cloud and the @capawesome/cli. Covers creating and configuring destinations for Apple App Store (TestFlight) and Google Play Store, obtaining required credentials, deploying builds to destinations, and managing deployments. Do not use for live update management, native build configuration, or non-Capacitor mobile frameworks.
---

# App Store Publishing

Automate app store submissions to TestFlight, Apple App Store, and Google Play Store using Capawesome Cloud.

## Prerequisites

1. [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.
2. An existing app in Capawesome Cloud (app ID required).
3. Node.js and npm installed.
4. For Apple App Store: An active [Apple Developer Program](https://developer.apple.com/programs/) membership and an app created in [App Store Connect](https://appstoreconnect.apple.com).
5. For Google Play Store: A [Google Play Developer](https://play.google.com/console) account and an app created in Google Play Console with at least one version uploaded manually.

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

### Step 2: Identify the Target Platform

Ask the user which platform to configure:

1. **Apple App Store (iOS)** — submits builds to TestFlight / App Store.
2. **Google Play Store (Android)** — submits builds to a Google Play track.
3. **Both** — configure both platforms sequentially.

If the user selects Apple App Store or both, proceed to Step 3.
If the user selects Google Play Store only, skip to Step 4.

### Step 3: Create an Apple App Store Destination

#### Step 3.1: Obtain Apple Credentials

Ask the user which authentication method to use:

1. **API Key (recommended)** — uses a `.p8` key file, Key ID, and Issuer ID.
2. **Apple ID + Password** — uses Apple ID, app-specific password, and Apple App ID.

Read `references/apple-app-store-credentials.md` for detailed instructions on obtaining the required credentials for the chosen method.

#### Step 3.2: Create the Destination via CLI

For **API Key** authentication:

```bash
npx @capawesome/cli apps:destinations:create \
  --app-id <APP_ID> \
  --name "<DESTINATION_NAME>" \
  --platform ios \
  --apple-team-id <TEAM_ID> \
  --apple-api-key-file <PATH_TO_P8_FILE> \
  --apple-issuer-id <ISSUER_ID>
```

For **Apple ID + Password** authentication:

```bash
npx @capawesome/cli apps:destinations:create \
  --app-id <APP_ID> \
  --name "<DESTINATION_NAME>" \
  --platform ios \
  --apple-team-id <TEAM_ID> \
  --apple-id <APPLE_ID_EMAIL> \
  --apple-app-id <APPLE_APP_ID> \
  --apple-app-password <APP_SPECIFIC_PASSWORD>
```

Replace all `<PLACEHOLDER>` values with the credentials obtained in Step 3.1.

If the user selected both platforms, proceed to Step 4. Otherwise, skip to Step 5.

### Step 4: Create a Google Play Store Destination

#### Step 4.1: Obtain Google Play Credentials

Read `references/google-play-store-credentials.md` for detailed instructions on obtaining the required credentials.

The user needs:
- **Package name**: The Android application ID (e.g., `com.example.app`), found in `android/app/build.gradle`.
- **Service account JSON key file**: From Google Cloud Console and Google Play Console setup.

#### Step 4.2: Choose Track and Release Settings

Ask the user:

1. **Track**: `internal`, `alpha`, `beta`, or `production`.
2. **Publishing format**: `aab` (recommended, required for new apps) or `apk`.
3. **Release status**: `completed` (auto-publish) or `draft` (manual review before publishing).

For the first release on a track, use `draft`. For subsequent releases, `completed` is typical.

#### Step 4.3: Create the Destination via CLI

```bash
npx @capawesome/cli apps:destinations:create \
  --app-id <APP_ID> \
  --name "<DESTINATION_NAME>" \
  --platform android \
  --android-package-name <PACKAGE_NAME> \
  --google-play-track <TRACK> \
  --android-build-artifact-type <aab|apk> \
  --android-release-status <completed|draft> \
  --google-service-account-key-file <PATH_TO_JSON_KEY>
```

Replace all `<PLACEHOLDER>` values with the credentials obtained in Step 4.1 and settings from Step 4.2.

### Step 5: Verify Destination Creation

List all destinations to confirm successful creation:

```bash
npx @capawesome/cli apps:destinations:list --app-id <APP_ID> --json
```

Verify the output contains the newly created destination(s) with the correct platform and name.

### Step 6: Deploy a Build to a Destination

Deploying submits an existing Capawesome Cloud build to the configured app store destination.

#### Step 6.1: Identify the Build

The user needs either a **build ID** or a **build number** from a completed Capawesome Cloud build. If unknown, ask the user for it.

#### Step 6.2: Create the Deployment

Using a build ID:

```bash
npx @capawesome/cli apps:deployments:create \
  --app-id <APP_ID> \
  --build-id <BUILD_ID> \
  --destination "<DESTINATION_NAME>"
```

Using a build number:

```bash
npx @capawesome/cli apps:deployments:create \
  --app-id <APP_ID> \
  --build-number <BUILD_NUMBER> \
  --destination "<DESTINATION_NAME>"
```

The CLI waits for the deployment to complete by default. Add `--detached` to exit immediately without waiting.

#### Step 6.3: Verify the Deployment

If the deployment does not complete successfully, check the deployment logs:

```bash
npx @capawesome/cli apps:deployments:logs \
  --app-id <APP_ID> \
  --deployment-id <DEPLOYMENT_ID>
```

### Step 7: Build and Deploy in One Step (Optional)

Combine building and deploying by passing the `--destination` flag to the build command:

```bash
npx @capawesome/cli apps:builds:create \
  --app-id <APP_ID> \
  --platform <ios|android> \
  --type <BUILD_TYPE> \
  --git-ref <GIT_REF> \
  --destination "<DESTINATION_NAME>" \
  --yes
```

For iOS App Store builds, use `--type app-store`. For Android release builds, use `--type release`.

This creates the build and automatically deploys it to the specified destination upon completion.

## Managing Destinations

Read `references/cli-commands.md` for the full CLI reference for managing destinations and deployments.

### List Destinations

```bash
npx @capawesome/cli apps:destinations:list --app-id <APP_ID> --json
```

### Update a Destination

```bash
npx @capawesome/cli apps:destinations:update \
  --app-id <APP_ID> \
  --name "<DESTINATION_NAME>" \
  --platform <ios|android> \
  [options]
```

Read `references/cli-commands.md` for all available update options.

### Delete a Destination

```bash
npx @capawesome/cli apps:destinations:delete \
  --app-id <APP_ID> \
  --name "<DESTINATION_NAME>" \
  --platform <ios|android>
```

## Error Handling

- Authentication errors → re-run `npx @capawesome/cli login`.
- Destination creation fails → verify all credentials are correct. Read `references/apple-app-store-credentials.md` or `references/google-play-store-credentials.md` for troubleshooting.
- iOS build not appearing in TestFlight → build processing may take time. Check email for processing errors from Apple. Common causes: build number not incremented, missing Privacy Descriptions in `Info.plist`, insufficient App Store Connect permissions (requires at least App Manager role).
- Google Play deployment fails → ensure the first app version was uploaded manually to Google Play Console. Verify the service account has Release permissions for the app.
- Deployment timeout → use `--detached` flag and check logs with `apps:deployments:logs`.
- `apps:deployments:create` fails with "build not found" → ensure the build completed successfully before deploying.
- Cancel a stuck deployment → `npx @capawesome/cli apps:deployments:cancel --app-id <APP_ID> --deployment-id <DEPLOYMENT_ID>`.
