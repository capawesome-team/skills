# CLI Commands Reference

Install: `npm install -g @capawesome/cli@latest`
Usage: `npx @capawesome/cli <command> [options]`

## Contents

- Authentication
- App Management
- Build Commands
- Certificate Commands
- Environment Commands
- Destination Commands
- Deployment Commands
- Live Update Bundle Commands
- Channel Commands
- Device Commands
- Utility

## Authentication

### login

```bash
npx @capawesome/cli login
npx @capawesome/cli login --token <token>   # CI/CD
```

### logout

```bash
npx @capawesome/cli logout
```

### whoami

```bash
npx @capawesome/cli whoami
```

## App Management

### apps:create

```bash
npx @capawesome/cli apps:create [--name <name>] [--organization-id <id>]
```

### apps:delete

```bash
npx @capawesome/cli apps:delete [--app-id <id>] [--yes]
```

## Build Commands

### apps:builds:create

Create a new native or web build.

```bash
npx @capawesome/cli apps:builds:create [options]
```

Options:
- `--app-id`: App ID
- `--platform`: `android`, `ios`, or `web`
- `--type`: Build type
  - Android: `debug`, `release`
  - iOS: `simulator`, `development`, `ad-hoc`, `app-store`, `enterprise`
- `--git-ref`: Git branch, tag, or commit SHA
- `--certificate`: Name of the signing certificate to use
- `--environment`: Name of the environment to use
- `--destination`: Deployment destination (Android/iOS only)
- `--channel`: Channel to deploy to (Web only)
- `--stack`: Build stack (`macos-sequoia` or `macos-tahoe`)
- `--apk`: Download APK after build (Android, optional file path)
- `--aab`: Download AAB after build (Android, optional file path)
- `--ipa`: Download IPA after build (iOS, optional file path)
- `--zip`: Download zip after build (Web, optional file path)
- `--json`: Output in JSON format (includes build ID)
- `--detached`: Exit immediately without waiting for build to complete
- `--yes, -y`: Skip confirmation prompts

### apps:builds:cancel

```bash
npx @capawesome/cli apps:builds:cancel --app-id <APP_ID> --build-id <BUILD_ID>
```

### apps:builds:download

Download build artifacts.

```bash
npx @capawesome/cli apps:builds:download [options]
```

Options:
- `--app-id`: App ID
- `--build-id`: Build ID
- `--apk`: Download APK (Android, optional file path)
- `--aab`: Download AAB (Android, optional file path)
- `--ipa`: Download IPA (iOS, optional file path)
- `--zip`: Download zip (Web, optional file path)

### apps:builds:logs

Display build logs.

```bash
npx @capawesome/cli apps:builds:logs --app-id <APP_ID> --build-id <BUILD_ID>
```

## Certificate Commands

### apps:certificates:create

```bash
npx @capawesome/cli apps:certificates:create [options]
```

Options:
- `--app-id`: App ID
- `--name`: Certificate name
- `--platform`: `android`, `ios`, or `web`
- `--type`: `development` or `production`
- `--file`: Path to certificate file (`.jks`/`.keystore` for Android, `.p12` for iOS)
- `--password`: Certificate/keystore password
- `--key-alias`: Key alias (Android only)
- `--key-password`: Key password (Android only)
- `--provisioning-profile`: Path to `.mobileprovision` file (iOS, repeatable)
- `--yes, -y`: Skip optional prompts

### apps:certificates:list

```bash
npx @capawesome/cli apps:certificates:list [--app-id <id>] [--platform <platform>] [--type <type>] [--json]
```

### apps:certificates:get

```bash
npx @capawesome/cli apps:certificates:get [--app-id <id>] [--certificate-id <id>] [--name <name>] [--platform <platform>] [--json]
```

### apps:certificates:update

```bash
npx @capawesome/cli apps:certificates:update [--app-id <id>] [--certificate-id <id>] [--name <name>] [--type <type>] [--password <pw>] [--key-alias <alias>] [--key-password <pw>]
```

### apps:certificates:delete

```bash
npx @capawesome/cli apps:certificates:delete [--app-id <id>] [--certificate-id <id>] [--name <name>] [--platform <platform>] [--yes]
```

## Environment Commands

### apps:environments:create

```bash
npx @capawesome/cli apps:environments:create --app-id <APP_ID> --name <NAME>
```

### apps:environments:list

```bash
npx @capawesome/cli apps:environments:list --app-id <APP_ID> [--json]
```

### apps:environments:set

```bash
npx @capawesome/cli apps:environments:set [options]
```

Options:
- `--app-id`: App ID
- `--environment-id`: Environment ID
- `--variable`: `key=value` (repeatable)
- `--variable-file`: Path to `.env` file
- `--secret`: `key=value` (repeatable)
- `--secret-file`: Path to `.env` file with secrets

### apps:environments:unset

```bash
npx @capawesome/cli apps:environments:unset [options]
```

Options:
- `--app-id`: App ID
- `--environment-id`: Environment ID
- `--variable`: Key to unset (repeatable)
- `--secret`: Key to unset (repeatable)

### apps:environments:delete

```bash
npx @capawesome/cli apps:environments:delete --app-id <APP_ID> [--environment-id <id> | --name <name>] [--yes]
```

## Destination Commands

### apps:destinations:create

Create a new destination for an app.

```bash
npx @capawesome/cli apps:destinations:create [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--name` | The name of the destination. |
| `--platform` | The platform (`android` or `ios`). |
| `--android-build-artifact-type` | Android build artifact type (`aab` or `apk`). |
| `--android-package-name` | Android package name. |
| `--android-release-status` | Android release status (`completed` or `draft`). |
| `--google-play-track` | Google Play track (`internal`, `alpha`, `beta`, or `production`). |
| `--google-service-account-key-file` | Path to the Google service account key JSON file. |
| `--apple-api-key-file` | Path to the Apple API key (`.p8`) file. |
| `--apple-app-id` | Apple App ID (numeric). |
| `--apple-app-password` | Apple app-specific password. |
| `--apple-id` | Apple ID email address. |
| `--apple-issuer-id` | Apple Issuer ID. |
| `--apple-team-id` | Apple Team ID. |

### apps:destinations:list

```bash
npx @capawesome/cli apps:destinations:list [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--json` | Output in JSON format. |
| `--limit` | Maximum number of destinations to return. |
| `--offset` | Offset to start returning destinations from. |
| `--platform` | Filter by platform (`android` or `ios`). |

### apps:destinations:get

```bash
npx @capawesome/cli apps:destinations:get [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--destination-id` | The ID of the destination. |
| `--json` | Output in JSON format. |
| `--name` | The name of the destination. |
| `--platform` | The platform (`android` or `ios`). |

### apps:destinations:update

```bash
npx @capawesome/cli apps:destinations:update [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--destination-id` | The ID of the destination. |
| `--name` | The name of the destination. |
| `--android-build-artifact-type` | Android build artifact type (`aab` or `apk`). |
| `--android-package-name` | Android package name. |
| `--android-release-status` | Android release status (`completed` or `draft`). |
| `--google-play-track` | Google Play track. |
| `--apple-api-key-id` | Apple API Key ID. |
| `--apple-app-id` | Apple App ID. |
| `--apple-app-password` | Apple app-specific password. |
| `--apple-id` | Apple ID email. |
| `--apple-issuer-id` | Apple Issuer ID. |
| `--apple-team-id` | Apple Team ID. |

### apps:destinations:delete

```bash
npx @capawesome/cli apps:destinations:delete [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--destination-id` | The ID of the destination. |
| `--name` | The name of the destination. |
| `--platform` | The platform (`android` or `ios`). |
| `--yes, -y` | Skip confirmation prompt. |

## Deployment Commands

### apps:deployments:create

Create a new app deployment.

```bash
npx @capawesome/cli apps:deployments:create [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--build-id` | The ID of the build to deploy. Alternative to `--build-number`. |
| `--build-number` | The build number to deploy. Alternative to `--build-id`. |
| `--channel` | The name of the channel to deploy to (Web only). |
| `--destination` | The name of the destination to deploy to (Android/iOS). |
| `--detached` | Exit immediately without waiting for completion. |

### apps:deployments:cancel

```bash
npx @capawesome/cli apps:deployments:cancel [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--deployment-id` | The ID of the deployment to cancel. |

### apps:deployments:logs

```bash
npx @capawesome/cli apps:deployments:logs [options]
```

Options:

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--deployment-id` | The ID of the deployment. |

## Live Update Bundle Commands

### apps:liveupdates:upload

Upload a locally built bundle and deploy to a channel.

```bash
npx @capawesome/cli apps:liveupdates:upload [options]
```

Options:
- `--app-id`: App ID
- `--path`: Path to web assets folder or zip
- `--channel`: Channel to deploy to
- `--artifact-type`: `zip` (default) or `manifest`
- `--private-key`: Private key file for code signing
- `--rollout-percentage`: 0-100 for gradual rollout
- `--android-min`, `--android-max`, `--android-eq`: Android version code constraints
- `--ios-min`, `--ios-max`, `--ios-eq`: iOS version constraints
- `--custom-property`: `key=value` pairs (repeatable)
- `--git-ref`: Git reference to associate
- `--expires-in-days`: Auto-delete after N days
- `--yes`: Skip prompts

### apps:liveupdates:register

Register a self-hosted bundle URL.

```bash
npx @capawesome/cli apps:liveupdates:register [options]
```

Options: Same as `upload` plus `--url <https://...>` (required).

### apps:liveupdates:bundle

Generate manifest and compress web assets into a zip.

```bash
npx @capawesome/cli apps:liveupdates:bundle [--input-path <path>] [--output-path <path>] [--overwrite] [--skip-manifest]
```

### apps:liveupdates:generatemanifest

Generate a manifest file for delta updates.

```bash
npx @capawesome/cli apps:liveupdates:generatemanifest --path <web-assets-path>
```

### apps:liveupdates:generatesigningkey

Generate RSA key pair for code signing.

```bash
npx @capawesome/cli apps:liveupdates:generatesigningkey [--key-size 2048|3072|4096] [--public-key-path <path>] [--private-key-path <path>]
```

### apps:liveupdates:rollback

Rollback to a previous deployment in a channel.

```bash
npx @capawesome/cli apps:liveupdates:rollback [--app-id <id>] [--channel <name>] [--steps 1-5]
```

### apps:liveupdates:rollout

Update rollout percentage for active build in a channel.

```bash
npx @capawesome/cli apps:liveupdates:rollout [--app-id <id>] [--channel <name>] [--percentage 0-100]
```

### apps:liveupdates:setnativeversions

Set native version constraints on a web build.

```bash
npx @capawesome/cli apps:liveupdates:setnativeversions [--app-id <id>] [--build-id <id>] [--android-min <code>] [--android-max <code>] [--ios-min <ver>] [--ios-max <ver>]
```

## Channel Commands

### apps:channels:create

```bash
npx @capawesome/cli apps:channels:create [--app-id <id>] [--name <name>] [--protected] [--ignore-errors]
```

### apps:channels:delete

```bash
npx @capawesome/cli apps:channels:delete [--app-id <id>] [--name <name>] [--yes]
```

### apps:channels:get

```bash
npx @capawesome/cli apps:channels:get [--app-id <id>] [--name <name>] [--json]
```

### apps:channels:list

```bash
npx @capawesome/cli apps:channels:list [--app-id <id>] [--json] [--limit <n>] [--offset <n>]
```

### apps:channels:pause

```bash
npx @capawesome/cli apps:channels:pause [--app-id <id>] [--channel <name>]
```

### apps:channels:resume

```bash
npx @capawesome/cli apps:channels:resume [--app-id <id>] [--channel <name>]
```

### apps:channels:update

```bash
npx @capawesome/cli apps:channels:update [--app-id <id>] [--channel-id <id>] [--name <name>] [--protected]
```

## Device Commands

### apps:devices:delete

```bash
npx @capawesome/cli apps:devices:delete [--app-id <id>] [--device-id <id>] [--yes]
```

### apps:devices:forcechannel

```bash
npx @capawesome/cli apps:devices:forcechannel [--app-id <id>] [--device-id <id>] [--channel <name>]
```

### apps:devices:unforcechannel

```bash
npx @capawesome/cli apps:devices:unforcechannel [--app-id <id>] [--device-id <id>]
```

## Utility

### doctor

```bash
npx @capawesome/cli doctor
```

Print environment and CLI diagnostic information.

## CI/CD Usage

For CI/CD pipelines, authenticate with a token:

```bash
npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
```

Use `--detached` to exit immediately after creating a build (useful for non-blocking pipelines):

```bash
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref main --certificate "Release Key" --detached --yes
```

Use `--json` to capture the build ID for downstream steps:

```bash
BUILD_OUTPUT=$(npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref main --json --yes)
```
