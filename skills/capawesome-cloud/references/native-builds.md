# Native Builds

Build native iOS and Android apps in the cloud using Capawesome Cloud.

## Contents

- Connect Git Repository
- Upload Signing Certificates
- Configure Environments
- Configure Build Settings
- Trigger a Build
- Monitor and Download Build Artifacts
- Configure Native Configuration Overwriting
- Set Up CI/CD

## Connect Git Repository

**This step requires the Capawesome Cloud Console** — no CLI command is available for Git connection.

Instruct the user to:

1. Navigate to the [Git](https://console.cloud.capawesome.io/apps/_/git) page of the app in the Capawesome Cloud Console.
2. In the **Git Providers** section, select the Git provider (GitHub, GitLab, Bitbucket, etc.) and click **Connect** to authorize access. Skip if already connected.
3. In the **Git Repositories** section, select the Git provider from the dropdown, choose the repository owner, and select the repository.
4. Click **Save**.

## Upload Signing Certificates

Skip this step if the user only wants to create unsigned builds (`debug` on Android, `simulator` on iOS).

### Android

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

If the user does not have a keystore, read `certificates-android.md` for creation instructions.

### iOS

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

Read `certificates-ios.md` to determine if the user already has a `.p12` file and provisioning profile, and guide them through obtaining the missing files if needed.

## Configure Environments (Optional)

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

Read `environments.md` for default CI variables, reserved variables (`JAVA_VERSION`, `NODE_VERSION`, `XCODE_VERSION`, etc.), secrets, and ad-hoc environments.

## Configure Build Settings (Optional)

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

Read `build-configuration.md` for all options including pnpm/Yarn setup and web build script priority.

## Trigger a Build

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

Read `build-types.md` for all available build types. Read `build-stacks.md` for available stacks and software versions.

## Monitor and Download Build Artifacts

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

## Configure Native Configuration Overwriting (Optional)

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

Read `build-configuration.md` for full Trapeze setup and environment variable usage. Read `build-guides.md` for auto-incrementing build numbers with version name management.

## Set Up CI/CD (Optional)

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

Read `cli-commands.md` for the full CLI reference including `--json` output for capturing build IDs.

## Advanced Topics

Read `build-guides.md` for:
- Auto-incrementing build numbers
- Custom iOS provisioning profiles for multi-target apps
- Private npm package configuration
- Overriding Java version
- Web build script configuration

Read `build-troubleshooting.md` for common errors and fixes.
