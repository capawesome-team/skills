# CLI Commands Reference

Reference for `@capawesome/cli` v4.9.0+.

Install: `npm install -g @capawesome/cli@latest`
Usage: `npx @capawesome/cli <command> [options]`

## Contents

- Authentication
- App Management
- Build Commands
- Certificate Commands
- Channel Commands
- Deployment Commands
- Destination Commands
- Device Commands
- Environment Commands
- Live Update Commands
- Organization Commands
- Utility
- Deprecated Commands

## Global Flags

| Flag | Description |
|------|-------------|
| `--help` | Show help for any command. |
| `--json` | Output in JSON format (supported by most read/create commands). |
| `--yes, -y` | Skip confirmation prompts. |
| `--detached` | Exit immediately without waiting for completion (builds and deployments). |

## Authentication

### login

```bash
npx @capawesome/cli login
npx @capawesome/cli login --token <TOKEN>   # CI/CD
```

| Option | Description |
|--------|-------------|
| `--token` | Token to use for authentication. |

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
npx @capawesome/cli apps:create [options]
```

| Option | Description |
|--------|-------------|
| `--name` | Name of the app. |
| `--organization-id` | ID of the organization to create the app in. |
| `--type` | Type of the app. Defaults to `capacitor`. |
| `--link` | Connect the created app to the local git repository. |
| `--json` | Output in JSON format. |
| `--yes, -y` | Skip all confirmation prompts. |

### apps:delete

```bash
npx @capawesome/cli apps:delete [--app-id <id>] [--yes]
```

### apps:get

Get an existing app.

```bash
npx @capawesome/cli apps:get [--app-id <id>] [--json]
```

### apps:list

Retrieve a list of existing apps.

```bash
npx @capawesome/cli apps:list [--organization-id <id>] [--json] [--limit <n>] [--offset <n>]
```

### apps:transfer

Transfer an app to another organization.

```bash
npx @capawesome/cli apps:transfer [--app-id <id>] [--organization-id <id>] [--yes]
```

### apps:link

Connect a git repository to an app. The repository information (provider, owner, and repository name) is automatically detected from the local git remote (`origin`).

```bash
npx @capawesome/cli apps:link [--app-id <id>]
```

### apps:unlink

Disconnect a git repository from an app.

```bash
npx @capawesome/cli apps:unlink [--app-id <id>] [--yes]
```

## Build Commands

### apps:builds:create

Create a new native or web build on Capawesome Cloud Runners.

```bash
npx @capawesome/cli apps:builds:create [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID to create the build for. |
| `--platform` | `android`, `ios`, or `web`. |
| `--type` | Build type. iOS: `simulator`, `development`, `ad-hoc`, `app-store`, `enterprise`. Android: `debug`, `release`. Web: no type required. |
| `--git-ref` | Git branch, tag, or commit SHA. Cannot be used with `--path` or `--url`. |
| `--path` | Path to local source files to upload. Cannot be used with `--git-ref` or `--url`. |
| `--url` | URL to a zip file to use as build source. Cannot be used with `--git-ref` or `--path`. |
| `--certificate` | Name of the signing certificate to use. |
| `--environment` | Name of the environment to use. |
| `--destination` | Destination to deploy to (Android/iOS only). |
| `--channel` | Channel to deploy to (Web only). |
| `--stack` | Build stack (e.g. `macos-sequoia`, `macos-tahoe`). |
| `--variable` | Ad hoc environment variable in `key=value` format (repeatable). |
| `--variable-file` | Path to a file containing ad hoc environment variables in `.env` format. |
| `--apk` | Download APK after build (Android, optional file path). |
| `--aab` | Download AAB after build (Android, optional file path). |
| `--ipa` | Download IPA after build (iOS, optional file path). |
| `--zip` | Download zip after build (Web, optional file path). |
| `--json` | Output in JSON format (includes build ID). |
| `--detached` | Exit immediately without waiting for the build to complete. |
| `--yes, -y` | Skip confirmation prompts. |

### apps:builds:cancel

```bash
npx @capawesome/cli apps:builds:cancel --app-id <APP_ID> --build-id <BUILD_ID>
```

### apps:builds:download

Download build artifacts.

```bash
npx @capawesome/cli apps:builds:download [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID the build belongs to. |
| `--build-id` | Build ID to download. |
| `--apk` | Download APK (Android, optional file path). |
| `--aab` | Download AAB (Android, optional file path). |
| `--ipa` | Download IPA (iOS, optional file path). |
| `--zip` | Download ZIP (Web, optional file path). |

### apps:builds:get

Get an existing app build.

```bash
npx @capawesome/cli apps:builds:get [--app-id <id>] [--build-id <id>] [--json]
```

### apps:builds:list

Retrieve a list of existing app builds.

```bash
npx @capawesome/cli apps:builds:list [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | ID of the app. |
| `--platform` | Platform to filter by. |
| `--number-as-string` | Build number (as string) to filter by. |
| `--limit` | Limit for pagination. |
| `--offset` | Offset for pagination. |
| `--json` | Output in JSON format. |

### apps:builds:logs

```bash
npx @capawesome/cli apps:builds:logs --app-id <APP_ID> --build-id <BUILD_ID>
```

## Certificate Commands

### apps:certificates:create

```bash
npx @capawesome/cli apps:certificates:create [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--name` | Certificate name. |
| `--platform` | `android`, `ios`, or `web`. |
| `--type` | `development` or `production`. |
| `--file` | Path to certificate file (`.jks`/`.keystore` for Android, `.p12` for iOS). |
| `--password` | Certificate/keystore password. |
| `--key-alias` | Key alias (Android only). |
| `--key-password` | Key password (Android only). |
| `--provisioning-profile` | Path to `.mobileprovision` file (iOS, repeatable). |
| `--yes, -y` | Skip confirmation prompts. |

### apps:certificates:list

```bash
npx @capawesome/cli apps:certificates:list [--app-id <id>] [--platform <platform>] [--type <type>] [--json] [--limit <n>] [--offset <n>]
```

### apps:certificates:get

```bash
npx @capawesome/cli apps:certificates:get [--app-id <id>] [--certificate-id <id>] [--name <name>] [--platform <platform>] [--type <type>] [--json]
```

### apps:certificates:update

```bash
npx @capawesome/cli apps:certificates:update [--app-id <id>] [--certificate-id <id>] [--name <name>] [--type <type>] [--password <pw>] [--key-alias <alias>] [--key-password <pw>]
```

### apps:certificates:delete

```bash
npx @capawesome/cli apps:certificates:delete [--app-id <id>] [--certificate-id <id>] [--name <name>] [--platform <platform>] [--type <type>] [--yes]
```

## Channel Commands

### apps:channels:create

```bash
npx @capawesome/cli apps:channels:create [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | ID of the app. |
| `--name` | Name of the channel. |
| `--protected` | Whether to protect the channel. Default `false`. |
| `--expires-in-days` | Number of days until the channel is automatically deleted. |
| `--ignore-errors` | Ignore errors (e.g. when the channel already exists). |

### apps:channels:delete

```bash
npx @capawesome/cli apps:channels:delete [--app-id <id>] [--channel-id <id>] [--name <name>] [--yes]
```

Either `--channel-id` or `--name` must be provided.

### apps:channels:get

```bash
npx @capawesome/cli apps:channels:get [--app-id <id>] [--channel-id <id>] [--name <name>] [--json]
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

## Deployment Commands

### apps:deployments:create

```bash
npx @capawesome/cli apps:deployments:create [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--build-id` | Build ID to deploy. Alternative to `--build-number`. |
| `--build-number` | Build number to deploy (e.g. `"1"`, `"42"`). Alternative to `--build-id`. |
| `--channel` | Channel to deploy to (Web only). |
| `--destination` | Destination to deploy to (Android/iOS). |
| `--detached` | Exit immediately without waiting for completion. |

### apps:deployments:cancel

```bash
npx @capawesome/cli apps:deployments:cancel --app-id <APP_ID> --deployment-id <DEPLOYMENT_ID>
```

### apps:deployments:get

Get an existing app deployment.

```bash
npx @capawesome/cli apps:deployments:get [--app-id <id>] [--deployment-id <id>] [--json]
```

### apps:deployments:list

Retrieve a list of existing app deployments.

```bash
npx @capawesome/cli apps:deployments:list [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | ID of the app. |
| `--build-id` | ID of the build to filter by. |
| `--channel-id` | ID of the channel to filter by. |
| `--destination-id` | ID of the destination to filter by. |
| `--limit` | Limit for pagination. |
| `--offset` | Offset for pagination. |
| `--json` | Output in JSON format. |

### apps:deployments:logs

```bash
npx @capawesome/cli apps:deployments:logs --app-id <APP_ID> --deployment-id <DEPLOYMENT_ID>
```

## Destination Commands

### apps:destinations:create

```bash
npx @capawesome/cli apps:destinations:create [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--name` | Destination name. |
| `--platform` | `android` or `ios`. |
| `--android-build-artifact-type` | `aab` or `apk`. |
| `--android-package-name` | Android package name. |
| `--android-release-status` | `completed` or `draft`. |
| `--google-play-track` | `internal`, `alpha`, `beta`, or `production`. |
| `--google-service-account-key-file` | Path to Google service account key JSON file. |
| `--apple-api-key-file` | Path to Apple API key (`.p8`) file. |
| `--apple-app-id` | Apple App ID (numeric). |
| `--apple-app-password` | Apple app-specific password. |
| `--apple-id` | Apple ID email address. |
| `--apple-issuer-id` | Apple Issuer ID. |
| `--apple-team-id` | Apple Team ID. |

### apps:destinations:list

```bash
npx @capawesome/cli apps:destinations:list [--app-id <id>] [--platform <platform>] [--json] [--limit <n>] [--offset <n>]
```

### apps:destinations:get

```bash
npx @capawesome/cli apps:destinations:get [--app-id <id>] [--destination-id <id>] [--name <name>] [--platform <platform>] [--json]
```

### apps:destinations:update

```bash
npx @capawesome/cli apps:destinations:update [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--destination-id` | Destination ID. |
| `--name` | Destination name. |
| `--android-build-artifact-type` | `aab` or `apk`. |
| `--android-package-name` | Android package name. |
| `--android-release-status` | `completed` or `draft`. |
| `--google-play-track` | Google Play track. |
| `--app-google-service-account-key-id` | App Google Service Account Key ID. |
| `--apple-api-key-id` | Apple API Key ID. |
| `--apple-app-id` | Apple App ID. |
| `--apple-app-password` | Apple app-specific password. |
| `--apple-id` | Apple ID email. |
| `--apple-issuer-id` | Apple Issuer ID. |
| `--apple-team-id` | Apple Team ID. |
| `--app-apple-api-key-id` | App Apple API Key ID. |

### apps:destinations:delete

```bash
npx @capawesome/cli apps:destinations:delete [--app-id <id>] [--destination-id <id>] [--name <name>] [--platform <platform>] [--yes]
```

## Device Commands

### apps:devices:delete

```bash
npx @capawesome/cli apps:devices:delete [--app-id <id>] [--device-id <id>] [--yes]
```

### apps:devices:forcechannel

Pin one or more devices to a specific channel.

```bash
npx @capawesome/cli apps:devices:forcechannel [--app-id <id>] [--device-id <id>] [--channel <name>]
```

`--device-id` can be specified multiple times.

### apps:devices:unforcechannel

Remove forced channel assignment from one or more devices.

```bash
npx @capawesome/cli apps:devices:unforcechannel [--app-id <id>] [--device-id <id>]
```

`--device-id` can be specified multiple times.

### apps:devices:probe

Test whether a device would receive a live update.

```bash
npx @capawesome/cli apps:devices:probe [--app-id <id>] [--device-id <id>] [--json]
```

## Environment Commands

### apps:environments:create

```bash
npx @capawesome/cli apps:environments:create --app-id <APP_ID> --name <NAME>
```

### apps:environments:list

```bash
npx @capawesome/cli apps:environments:list --app-id <APP_ID> [--json] [--limit <n>] [--offset <n>]
```

### apps:environments:set

```bash
npx @capawesome/cli apps:environments:set [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--environment-id` | Environment ID. |
| `--variable` | `key=value` (repeatable). |
| `--variable-file` | Path to `.env` file. |
| `--secret` | `key=value` (repeatable). |
| `--secret-file` | Path to `.env` file with secrets. |

### apps:environments:unset

```bash
npx @capawesome/cli apps:environments:unset [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--environment-id` | Environment ID. |
| `--variable` | Key to unset (repeatable). |
| `--secret` | Key to unset (repeatable). |

### apps:environments:delete

```bash
npx @capawesome/cli apps:environments:delete --app-id <APP_ID> [--environment-id <id> | --name <name>] [--yes]
```

## Live Update Commands

### apps:liveupdates:create

Create a new live update by building and deploying web assets using Capawesome Cloud Runners.

```bash
npx @capawesome/cli apps:liveupdates:create [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--channel` | Channel to deploy to (repeatable). |
| `--git-ref` | Git branch, tag, or commit SHA. Cannot be used with `--path` or `--url`. |
| `--path` | Path to local source files to upload. Cannot be used with `--git-ref` or `--url`. |
| `--url` | URL to a zip file to use as build source. Cannot be used with `--git-ref` or `--path`. |
| `--certificate` | Name of the certificate to use for the build. |
| `--environment` | Name of the environment to use for the build. |
| `--stack` | Build stack (e.g. `macos-sequoia`, `macos-tahoe`). |
| `--variable` | Ad hoc environment variable in `key=value` format (repeatable). |
| `--variable-file` | Path to a file containing ad hoc environment variables in `.env` format. |
| `--rollout-percentage` | Rollout percentage 0-100 (default: `100`). |
| `--android-min` | Minimum Android version code. |
| `--android-max` | Maximum Android version code. |
| `--android-eq` | Exact Android version code. |
| `--ios-min` | Minimum iOS version (`CFBundleVersion`). |
| `--ios-max` | Maximum iOS version. |
| `--ios-eq` | Exact iOS version. |
| `--custom-property` | `key=value` pairs (repeatable). |
| `--json` | Output in JSON format. |
| `--yes, -y` | Skip confirmation prompts. |

### apps:liveupdates:upload

Upload a locally built bundle and deploy to a channel.

```bash
npx @capawesome/cli apps:liveupdates:upload [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--path` | Path to bundle (web assets folder, e.g. `www`/`dist`, or zip file). |
| `--channel` | Channel to associate the bundle with. |
| `--artifact-type` | `zip` (default) or `manifest`. |
| `--private-key` | Private key file path (`.pem`) or content for code signing. |
| `--rollout-percentage` | 0-100 for gradual rollout. |
| `--expires-in-days` | Number of days until the bundle is automatically deleted. |
| `--git-ref` | Git reference (branch, tag, or commit SHA) to associate with the bundle. |
| `--android-min` | Minimum Android version code. |
| `--android-max` | Maximum Android version code. |
| `--android-eq` | Exact Android version code. |
| `--ios-min` | Minimum iOS version (`CFBundleVersion`). |
| `--ios-max` | Maximum iOS version. |
| `--ios-eq` | Exact iOS version. |
| `--custom-property` | `key=value` pairs (repeatable). |
| `--commit-message` | **Deprecated** — use `--git-ref`. |
| `--commit-ref` | **Deprecated** — use `--git-ref`. |
| `--commit-sha` | **Deprecated** — use `--git-ref`. |
| `--yes, -y` | Skip confirmation prompts. |

### apps:liveupdates:register

Register a self-hosted bundle URL for serving artifacts from your own infrastructure.

```bash
npx @capawesome/cli apps:liveupdates:register [options]
```

Same options as `apps:liveupdates:upload` (except `--artifact-type`), plus:

| Option | Description |
|--------|-------------|
| `--url` | HTTPS URL to the self-hosted bundle (required). |
| `--path` | Path to a zip file (used for code signing only, when `--private-key` is provided). |

### apps:liveupdates:bundle

Generate a manifest file and compress locally built web assets into a `bundle.zip` file.

```bash
npx @capawesome/cli apps:liveupdates:bundle [options]
```

| Option | Description |
|--------|-------------|
| `--input-path` | Path to the web assets directory. |
| `--output-path` | Output path for the generated artifact (default: `./bundle.zip`). |
| `--overwrite` | Overwrite output file if it already exists (default: `false`). |
| `--skip-manifest` | Skip manifest file generation (default: `false`). |

### apps:liveupdates:generatemanifest

Generate a manifest file for delta updates.

```bash
npx @capawesome/cli apps:liveupdates:generatemanifest --path <web-assets-path>
```

### apps:liveupdates:generatesigningkey

Generate an RSA key pair for code signing.

```bash
npx @capawesome/cli apps:liveupdates:generatesigningkey [--key-size 2048|3072|4096] [--public-key-path <path>] [--private-key-path <path>]
```

Defaults: `--key-size 2048`, `--public-key-path public.pem`, `--private-key-path private.pem`.

### apps:liveupdates:rollback

Rollback the active build in a channel to a previous build.

```bash
npx @capawesome/cli apps:liveupdates:rollback [--app-id <id>] [--channel <name>] [--steps 1-5]
```

### apps:liveupdates:rollout

Update the rollout percentage of the active build in a channel.

```bash
npx @capawesome/cli apps:liveupdates:rollout [--app-id <id>] [--channel <name>] [--percentage 0-100]
```

### apps:liveupdates:setnativeversions

Set native version constraints on a web build.

```bash
npx @capawesome/cli apps:liveupdates:setnativeversions [options]
```

| Option | Description |
|--------|-------------|
| `--app-id` | App ID. |
| `--build-id` | Build ID. |
| `--android-min` | Minimum Android version code. |
| `--android-max` | Maximum Android version code. |
| `--android-eq` | Exact Android version code. |
| `--ios-min` | Minimum iOS version (`CFBundleVersion`). |
| `--ios-max` | Maximum iOS version. |
| `--ios-eq` | Exact iOS version. |

## Organization Commands

### organizations:create

```bash
npx @capawesome/cli organizations:create [--name <name>]
```

### organizations:get

Get an existing organization.

```bash
npx @capawesome/cli organizations:get [--organization-id <id>] [--json]
```

### organizations:list

Retrieve a list of existing organizations.

```bash
npx @capawesome/cli organizations:list [--json] [--limit <n>] [--offset <n>]
```

## Utility

### doctor

Print environment and CLI diagnostic information.

```bash
npx @capawesome/cli doctor
```

## Deprecated Commands

These commands are still available but should not be used in new code:

| Command | Replacement |
|---------|-------------|
| `apps:bundles:create` | `apps:liveupdates:upload` (or `apps:liveupdates:create` / `apps:liveupdates:register`) |
| `apps:bundles:delete` | — (managed via Capawesome Cloud Console) |
| `apps:bundles:update` | `apps:liveupdates:rollout` / `apps:liveupdates:setnativeversions` |
| `manifests:generate` | `apps:liveupdates:generatemanifest` |
