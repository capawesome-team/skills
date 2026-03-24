# CI/CD Integration

Use the Capawesome CLI in CI/CD pipelines with token-based authentication and non-interactive flags.

## Authentication

1. Generate a token in the [Capawesome Cloud Console](https://console.cloud.capawesome.io) under **Settings > Tokens**.
2. Store the token as a secret in the CI/CD platform (e.g., `CAPAWESOME_CLOUD_TOKEN`).
3. Authenticate in the pipeline:

```bash
npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
```

## CI/CD Flags

| Flag | Description |
|------|-------------|
| `--token <token>` | Authenticate non-interactively (used with `login`). |
| `--yes, -y` | Skip all confirmation prompts. |
| `--detached` | Exit immediately without waiting for build/deployment to complete. |
| `--json` | Output in JSON format for programmatic parsing. |

## Example: Trigger a Build

### Blocking (Wait for Completion)

```bash
npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
npx @capawesome/cli apps:builds:create \
  --app-id <APP_ID> \
  --platform android \
  --type release \
  --git-ref main \
  --certificate "Production Android Key" \
  --yes
```

### Non-Blocking (Detached)

```bash
npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
npx @capawesome/cli apps:builds:create \
  --app-id <APP_ID> \
  --platform android \
  --type release \
  --git-ref main \
  --certificate "Production Android Key" \
  --detached \
  --yes
```

## Capture Build ID with JSON Output

Use `--json` to capture the build ID for downstream steps:

```bash
BUILD_OUTPUT=$(npx @capawesome/cli apps:builds:create \
  --app-id <APP_ID> \
  --platform android \
  --type release \
  --git-ref main \
  --json \
  --yes)
```

Extract the build ID using `jq`:

```bash
BUILD_ID=$(echo "$BUILD_OUTPUT" | jq -r '.buildId')
```

Use the build ID in subsequent commands:

```bash
npx @capawesome/cli apps:builds:download --app-id <APP_ID> --build-id $BUILD_ID --apk ./app-release.apk
```

## Example: Upload a Live Update Bundle

```bash
npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
npx @capawesome/cli apps:liveupdates:upload \
  --app-id <APP_ID> \
  --path ./dist \
  --channel production \
  --yes
```

## Example: Build and Deploy to App Store

```bash
npx @capawesome/cli login --token $CAPAWESOME_CLOUD_TOKEN
npx @capawesome/cli apps:builds:create \
  --app-id <APP_ID> \
  --platform ios \
  --type app-store \
  --git-ref main \
  --certificate "Production iOS Certificate" \
  --destination "App Store" \
  --yes
```

## GitHub Actions Example

```yaml
name: Build and Deploy
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Authenticate with Capawesome Cloud
        run: npx @capawesome/cli login --token ${{ secrets.CAPAWESOME_CLOUD_TOKEN }}

      - name: Trigger Android build
        run: |
          npx @capawesome/cli apps:builds:create \
            --app-id ${{ vars.CAPAWESOME_APP_ID }} \
            --platform android \
            --type release \
            --git-ref ${{ github.sha }} \
            --certificate "Production Android Key" \
            --yes

      - name: Trigger iOS build
        run: |
          npx @capawesome/cli apps:builds:create \
            --app-id ${{ vars.CAPAWESOME_APP_ID }} \
            --platform ios \
            --type app-store \
            --git-ref ${{ github.sha }} \
            --certificate "Production iOS Certificate" \
            --yes
```
