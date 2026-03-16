# App Store Publishing

Automate app store submissions to TestFlight, Apple App Store, and Google Play Store using Capawesome Cloud.

## Contents

- Identify the Target Platform
- Create an Apple App Store Destination
- Create a Google Play Store Destination
- Verify Destination Creation
- Deploy a Build to a Destination
- Build and Deploy in One Step
- Managing Destinations

## Identify the Target Platform

Ask the user which platform to configure:

1. **Apple App Store (iOS)** — submits builds to TestFlight / App Store.
2. **Google Play Store (Android)** — submits builds to a Google Play track.
3. **Both** — configure both platforms sequentially.

If the user selects Apple App Store or both, proceed to Create an Apple App Store Destination.
If the user selects Google Play Store only, skip to Create a Google Play Store Destination.

## Create an Apple App Store Destination

### Obtain Apple Credentials

Ask the user which authentication method to use:

1. **API Key (recommended)** — uses a `.p8` key file, Key ID, and Issuer ID.
2. **Apple ID + Password** — uses Apple ID, app-specific password, and Apple App ID.

Read `apple-app-store-credentials.md` for detailed instructions on obtaining the required credentials for the chosen method.

### Create the Destination via CLI

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

Replace all `<PLACEHOLDER>` values with the credentials obtained above.

If the user selected both platforms, proceed to Create a Google Play Store Destination. Otherwise, skip to Verify Destination Creation.

## Create a Google Play Store Destination

### Obtain Google Play Credentials

Read `google-play-store-credentials.md` for detailed instructions on obtaining the required credentials.

The user needs:
- **Package name**: The Android application ID (e.g., `com.example.app`), found in `android/app/build.gradle`.
- **Service account JSON key file**: From Google Cloud Console and Google Play Console setup.

### Choose Track and Release Settings

Ask the user:

1. **Track**: `internal`, `alpha`, `beta`, or `production`.
2. **Publishing format**: `aab` (recommended, required for new apps) or `apk`.
3. **Release status**: `completed` (auto-publish) or `draft` (manual review before publishing).

For the first release on a track, use `draft`. For subsequent releases, `completed` is typical.

### Create the Destination via CLI

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

Replace all `<PLACEHOLDER>` values with the credentials and settings from above.

## Verify Destination Creation

List all destinations to confirm successful creation:

```bash
npx @capawesome/cli apps:destinations:list --app-id <APP_ID> --json
```

Verify the output contains the newly created destination(s) with the correct platform and name.

## Deploy a Build to a Destination

Deploying submits an existing Capawesome Cloud build to the configured app store destination.

### Identify the Build

The user needs either a **build ID** or a **build number** from a completed Capawesome Cloud build. If unknown, ask the user for it.

### Create the Deployment

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

### Verify the Deployment

If the deployment does not complete successfully, check the deployment logs:

```bash
npx @capawesome/cli apps:deployments:logs \
  --app-id <APP_ID> \
  --deployment-id <DEPLOYMENT_ID>
```

## Build and Deploy in One Step (Optional)

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

Read `cli-commands.md` for the full CLI reference for managing destinations and deployments.

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

Read `cli-commands.md` for all available update options.

### Delete a Destination

```bash
npx @capawesome/cli apps:destinations:delete \
  --app-id <APP_ID> \
  --name "<DESTINATION_NAME>" \
  --platform <ios|android>
```
