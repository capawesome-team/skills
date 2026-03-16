# CLI Commands for Native Builds

Install: `npm install -g @capawesome/cli@latest`
Usage: `npx @capawesome/cli <command> [options]`

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

Create a new native build.

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
