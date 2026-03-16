# Environments

Environment variables customize native builds by providing configuration values, secrets, and build parameters.

## Default CI Variables

Automatically set for every build. Cannot be overridden.

| Name                    | Type      | Description                                          |
| ----------------------- | --------- | ---------------------------------------------------- |
| `CI`                    | `boolean` | Always `true` in CI builds.                          |
| `CI_APP_ID`             | `string`  | The app ID.                                          |
| `CI_BUILD_ID`           | `string`  | Unique build identifier.                             |
| `CI_BUILD_NUMBER`       | `string`  | Sequential build number (auto-increments).           |
| `CI_GIT_COMMIT_SHA`     | `string`  | SHA hash of the Git commit.                          |
| `CI_GIT_COMMIT_MESSAGE` | `string`  | Commit message.                                      |
| `CI_GIT_REFERENCE`      | `string`  | Git branch or tag name.                              |
| `CI_PLATFORM`           | `string`  | Target platform (`ios` or `android`).                |

## Reserved Environment Variables

Can be set to customize build behavior.

| Name                           | Type     | Description                                                                |
| ------------------------------ | -------- | -------------------------------------------------------------------------- |
| `ANDROID_BUILD_TYPE`           | `string` | Custom Android build type (e.g., `staging`).                               |
| `ANDROID_FLAVOR`               | `string` | Android product flavor.                                                    |
| `IOS_PROVISIONING_PROFILE_MAP` | `string` | JSON mapping bundle IDs to target names for multi-target apps.             |
| `IOS_SCHEME`                   | `string` | iOS scheme name.                                                           |
| `NODE_VERSION`                 | `int`    | Node.js major version (e.g., `22` or `24`).                               |
| `JAVA_VERSION`                 | `int`    | Java major version (e.g., `17` or `21`).                                   |
| `XCODE_VERSION`                | `int`    | Xcode major version (e.g., `16` or `26`).                                  |

## Custom Environments

Custom environments group variables/secrets for different build scenarios (e.g., development, staging, production).

### Create via CLI

```bash
npx @capawesome/cli apps:environments:create --app-id <APP_ID> --name staging
```

### Set Variables via CLI

```bash
npx @capawesome/cli apps:environments:set \
  --app-id <APP_ID> \
  --environment-id <ENV_ID> \
  --variable "API_URL=https://api.staging.example.com" \
  --variable "FEATURE_FLAG=true"
```

Bulk import from a `.env` file:

```bash
npx @capawesome/cli apps:environments:set \
  --app-id <APP_ID> \
  --environment-id <ENV_ID> \
  --variable-file .env
```

### Set Secrets via CLI

```bash
npx @capawesome/cli apps:environments:set \
  --app-id <APP_ID> \
  --environment-id <ENV_ID> \
  --secret "API_KEY=sk-abc123" \
  --secret "NPM_TOKEN=npm_xyz"
```

Bulk import from a `.env` file:

```bash
npx @capawesome/cli apps:environments:set \
  --app-id <APP_ID> \
  --environment-id <ENV_ID> \
  --secret-file .env.secrets
```

Secrets are encrypted at rest and in transit and never displayed in build logs.

### Unset Variables/Secrets

```bash
npx @capawesome/cli apps:environments:unset \
  --app-id <APP_ID> \
  --environment-id <ENV_ID> \
  --variable API_URL \
  --secret API_KEY
```

### List Environments

```bash
npx @capawesome/cli apps:environments:list --app-id <APP_ID> --json
```

### Delete an Environment

```bash
npx @capawesome/cli apps:environments:delete --app-id <APP_ID> --name staging --yes
```

### Set Default Environment

To auto-select an environment for every build, navigate to [App Settings](https://console.cloud.capawesome.io/apps/_/settings), enable **Default Environment** in the "Builds" section, select the environment, and save.

## Ad-hoc Environment Variables

Temporary variables set when creating a single build. Not persisted.

Useful for:
- Troubleshooting (e.g., testing a different `NODE_VERSION`)
- Overriding native configuration values for a single build

Set via Console when creating a build: enter `KEY=VALUE` pairs in the "Ad-hoc Environment" field.

Set via CLI using the `--ad-hoc-env` flag on `apps:builds:create` (if available).

**Warning:** Ad-hoc variables are **not encrypted** and may appear in build logs. Do not use for sensitive data — use environment secrets instead.

## Reserved Name Restrictions

The variable name `CI` and any name starting with `CI_` are reserved and cannot be overridden by custom or ad-hoc environments.
