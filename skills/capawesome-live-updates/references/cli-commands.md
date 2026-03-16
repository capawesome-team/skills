# Capawesome CLI Commands for Live Updates

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

## Build Commands (Cloud Builds)

### apps:builds:create

```bash
npx @capawesome/cli apps:builds:create [--app-id <id>] [--platform web] [--git-ref <ref>] [--channel <name>] [--json] [--yes]
```

### apps:deployments:create

```bash
npx @capawesome/cli apps:deployments:create [--app-id <id>] [--build-number <n>] [--channel <name>]
```

## Utility

### doctor

```bash
npx @capawesome/cli doctor
```

Print environment and CLI diagnostic information.
