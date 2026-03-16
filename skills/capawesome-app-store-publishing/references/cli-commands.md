# CLI Commands Reference

Full reference for `@capawesome/cli` commands related to App Store Publishing.

## Destinations

### `apps:destinations:create`

Create a new destination for an app.

```bash
npx @capawesome/cli apps:destinations:create [options]
```

**Options:**

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

### `apps:destinations:list`

List all destinations for an app.

```bash
npx @capawesome/cli apps:destinations:list [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--json` | Output in JSON format. |
| `--limit` | Maximum number of destinations to return. |
| `--offset` | Offset to start returning destinations from. |
| `--platform` | Filter by platform (`android` or `ios`). |

### `apps:destinations:get`

Get a destination from an app.

```bash
npx @capawesome/cli apps:destinations:get [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--destination-id` | The ID of the destination. |
| `--json` | Output in JSON format. |
| `--name` | The name of the destination. |
| `--platform` | The platform (`android` or `ios`). |

### `apps:destinations:update`

Update an existing destination.

```bash
npx @capawesome/cli apps:destinations:update [options]
```

**Options:**

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

### `apps:destinations:delete`

Delete a destination from an app.

```bash
npx @capawesome/cli apps:destinations:delete [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--destination-id` | The ID of the destination. |
| `--name` | The name of the destination. |
| `--platform` | The platform (`android` or `ios`). |
| `--yes, -y` | Skip confirmation prompt. |

## Deployments

### `apps:deployments:create`

Create a new app deployment.

```bash
npx @capawesome/cli apps:deployments:create [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--build-id` | The ID of the build to deploy. Alternative to `--build-number`. |
| `--build-number` | The build number to deploy. Alternative to `--build-id`. |
| `--channel` | The name of the channel to deploy to (Web only). |
| `--destination` | The name of the destination to deploy to (Android/iOS). |
| `--detached` | Exit immediately without waiting for completion. |

### `apps:deployments:cancel`

Cancel an ongoing app deployment.

```bash
npx @capawesome/cli apps:deployments:cancel [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--deployment-id` | The ID of the deployment to cancel. |

### `apps:deployments:logs`

Display the logs for an ongoing or completed deployment.

```bash
npx @capawesome/cli apps:deployments:logs [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--deployment-id` | The ID of the deployment. |

## Builds (Relevant Options)

### `apps:builds:create`

Create a new app build. When `--destination` is provided, the build is automatically deployed upon completion.

```bash
npx @capawesome/cli apps:builds:create [options]
```

**Relevant options for App Store Publishing:**

| Option | Description |
|--------|-------------|
| `--app-id` | The ID of the app. |
| `--platform` | The platform (`android` or `ios`). |
| `--type` | Build type. iOS: `app-store`. Android: `release`. |
| `--git-ref` | Git reference (branch, tag, or commit SHA) to build. |
| `--destination` | The name of the destination to deploy to upon completion. |
| `--certificate` | The name of the certificate to use. |
| `--environment` | The name of the environment to use. |
| `--detached` | Exit immediately without waiting for completion. |
| `--json` | Output in JSON format. |
| `--yes, -y` | Skip confirmation prompts. |
