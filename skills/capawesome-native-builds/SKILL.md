---
name: capawesome-native-builds
description: Guides the agent through setting up and using Capawesome Cloud Native Builds to build native iOS and Android apps in the cloud. Covers creating apps, connecting Git repositories, uploading signing certificates (Android keystores, iOS .p12 and provisioning profiles), configuring environments and secrets, triggering builds via CLI, downloading artifacts, configuring Trapeze for native configuration overwriting, and setting up CI/CD pipelines. Do not use for live update management, app store publishing workflows, or non-Capacitor mobile frameworks.
---

# Capawesome Cloud Native Builds

Build native iOS and Android apps in the cloud using Capawesome Cloud.

## Prerequisites

1. A [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.
2. A Capacitor app in a Git repository (GitHub, GitLab, Bitbucket, or Azure DevOps).
3. Node.js and npm installed.

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

### Step 3: Connect Git Repository

**This step requires the Capawesome Cloud Console** — no CLI command is available for Git connection.

Instruct the user to:

1. Navigate to the [Git](https://console.cloud.capawesome.io/apps/_/git) page of the app in the Capawesome Cloud Console.
2. In the **Git Providers** section, select the Git provider (GitHub, GitLab, Bitbucket, etc.) and click **Connect** to authorize access. Skip if already connected.
3. In the **Git Repositories** section, select the Git provider from the dropdown, choose the repository owner, and select the repository.
4. Click **Save**.

### Step 4: Upload Signing Certificates

Skip this step if the user only wants to create unsigned builds (`debug` on Android, `simulator` on iOS).

#### Android

Upload a Java Keystore for signed Android builds:

```bash
npx @capawesome/cli apps:certificates:create \
  --app-id <APP_ID> \
  --name "Production Android Key" \
  --platform android \
  --type production \
  --file /path/to/my-release-key.jks \
  --password <KEYSTORE_PASSWORD> \
  --key-alias my-key-alias \
  --key-password <KEY_PASSWORD>
```

If the user does not have a keystore, read `references/certificates-android.md` for creation instructions.

#### iOS

Upload a `.p12` certificate and provisioning profile for signed iOS builds:

```bash
npx @capawesome/cli apps:certificates:create \
  --app-id <APP_ID> \
  --name "Production iOS Certificate" \
  --platform ios \
  --type production \
  --file /path/to/certificate.p12 \
  --password <CERTIFICATE_PASSWORD> \
  --provisioning-profile /path/to/profile.mobileprovision
```

If the user does not have a `.p12` file or provisioning profile, read `references/certificates-ios.md` for creation instructions.

### Step 5: Configure Environments (Optional)

Skip unless the user needs custom environment variables, secrets, or reserved variable overrides.

Create an environment:

```bash
npx @capawesome/cli apps:environments:create --app-id <APP_ID> --name production
```

Set variables and secrets:

```bash
npx @capawesome/cli apps:environments:set \
  --app-id <APP_ID> \
  --environment-id <ENV_ID> \
  --variable "API_URL=https://api.example.com" \
  --secret "API_KEY=sk-abc123"
```

Read `references/environments.md` for default CI variables, reserved variables (`JAVA_VERSION`, `NODE_VERSION`, `XCODE_VERSION`, etc.), secrets, and ad-hoc environments.

### Step 6: Configure Build Settings (Optional)

Skip for standard project setups where the app is in the repo root and uses `npm install` + `npm run build`.

For monorepos, subdirectory apps, or custom build commands, create `capawesome.config.json` in the project root:

```json
{
  "cloud": {
    "apps": [
      {
        "appId": "<APP_ID>",
        "baseDir": "apps/my-app",
        "dependencyInstallCommand": "npm install",
        "webBuildCommand": "npm run build"
      }
    ]
  }
}
```

Read `references/configuration.md` for all options including pnpm/Yarn setup and web build script priority.

### Step 7: Trigger a Build

```bash
npx @capawesome/cli apps:builds:create \
  --app-id <APP_ID> \
  --platform <android|ios> \
  --type <BUILD_TYPE> \
  --git-ref <BRANCH_OR_TAG> \
  --certificate "<CERTIFICATE_NAME>"
```

Examples:

```bash
# Android debug (no certificate needed)
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type debug --git-ref main

# Android release
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref main --certificate "Production Android Key"

# iOS simulator (no certificate needed)
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --type simulator --git-ref main

# iOS App Store
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --type app-store --git-ref main --certificate "Production iOS Certificate"
```

Add `--environment <NAME>` to use a custom environment. Add `--stack <STACK>` to select a build stack.

Read `references/build-types.md` for all available build types. Read `references/build-stacks.md` for available stacks and software versions.

### Step 8: Monitor and Download Build Artifacts

View build logs:

```bash
npx @capawesome/cli apps:builds:logs --app-id <APP_ID> --build-id <BUILD_ID>
```

Download artifacts after the build completes:

```bash
# Android
npx @capawesome/cli apps:builds:download --app-id <APP_ID> --build-id <BUILD_ID> --apk
npx @capawesome/cli apps:builds:download --app-id <APP_ID> --build-id <BUILD_ID> --aab

# iOS
npx @capawesome/cli apps:builds:download --app-id <APP_ID> --build-id <BUILD_ID> --ipa
```

Alternatively, use inline download flags on `apps:builds:create`:

```bash
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref main --certificate "Production Android Key" --apk ./app-release.apk
```

### Step 9: Configure Native Configuration Overwriting (Optional)

Skip unless the user needs to modify native project settings (app ID, display name, version numbers) dynamically per build.

1. Install Trapeze:

```bash
npm install --save-dev @trapezedev/configure
```

2. Create `capawesome.yml` in the project root:

```yaml
vars:
  CI_BUILD_NUMBER:
    default: 1

platforms:
  android:
    versionCode: $CI_BUILD_NUMBER
  ios:
    buildNumber: $CI_BUILD_NUMBER
```

3. Add the build script to `package.json`:

```json
{
  "scripts": {
    "capawesome:build": "if [ \"$CI_PLATFORM\" = \"ios\" ] || [ \"$CI_PLATFORM\" = \"android\" ]; then npx trapeze run capawesome.yml -y --$CI_PLATFORM; fi && npm run build"
  }
}
```

Read `references/configuration.md` for full Trapeze setup and environment variable usage. Read `references/guides.md` for auto-incrementing build numbers with version name management.

### Step 10: Set Up CI/CD (Optional)

Skip unless the user wants automated builds from CI/CD pipelines.

1. Generate a token in the [Capawesome Cloud Console](https://console.cloud.capawesome.io).
2. Store as a secret in the CI/CD platform (e.g., `CAPAWESOME_CLOUD_TOKEN`).
3. Authenticate and trigger builds:

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

Use `--detached` to exit immediately without waiting for the build to complete:

```bash
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref main --detached --yes
```

Read `references/cli-commands.md` for the full CLI reference including `--json` output for capturing build IDs.

## Error Handling

- **`invalid source release: 21`** → Set `JAVA_VERSION` env var to `17` or `21`. See `references/troubleshooting.md`.
- **`JavaScript heap out of memory`** → Set `NODE_OPTIONS` env var to `--max-old-space-size=4096`. See `references/troubleshooting.md`.
- **Authentication errors** → Re-run `npx @capawesome/cli login`. For CI/CD, verify the token.
- **Missing signing certificate** → Upload via `apps:certificates:create`. See `references/certificates-android.md` or `references/certificates-ios.md`.
- **Expired provisioning profile** → Regenerate in Apple Developer Portal and re-upload. See `references/certificates-ios.md`.
- **Web build step fails** → Ensure `package.json` has `capawesome:build` or `build` script. See `references/configuration.md`.

## Advanced Topics

Read `references/guides.md` for:
- Auto-incrementing build numbers
- Custom iOS provisioning profiles for multi-target apps
- Private npm package configuration
- Overriding Java version
- Web build script configuration

Read `references/troubleshooting.md` for common errors and fixes.
