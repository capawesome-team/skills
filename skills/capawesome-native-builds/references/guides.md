# Advanced Guides

## Auto-Increment Build Numbers

Use `CI_BUILD_NUMBER` with Trapeze to automatically increment build numbers.

### Prerequisites

Install Trapeze and set up the build script (see `references/configuration.md`, "Native Configuration Overwriting" section).

### Configuration

Create `capawesome.yml` in the project root:

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

The default value `1` is used for local development. In Capawesome Cloud, `CI_BUILD_NUMBER` auto-increments with each build.

### Combine with Version Name

```yaml
vars:
  VERSION_NAME:
  CI_BUILD_NUMBER:
    default: 1

platforms:
  android:
    versionName: $VERSION_NAME
    versionCode: $CI_BUILD_NUMBER
  ios:
    version: $VERSION_NAME
    buildNumber: $CI_BUILD_NUMBER
```

Set `VERSION_NAME` (e.g., `1.0.0`) in the Capawesome Cloud environment.

### Custom Starting Number

Set the next build number in [App Settings](https://console.cloud.capawesome.io/apps/_/settings). Must be at least 1 higher than the previous build number.

Alternatively, add an offset in the build script:

```json
{
  "scripts": {
    "capawesome:build": "CI_BUILD_NUMBER=$(($CI_BUILD_NUMBER + 1000)); if [ \"$CI_PLATFORM\" = \"ios\" ] || [ \"$CI_PLATFORM\" = \"android\" ]; then npx trapeze run capawesome.yml -y --$CI_PLATFORM; fi && npm run build"
  }
}
```

Replace `1000` with the desired offset.

## Custom iOS Provisioning Profiles for Multiple Targets

For apps with multiple targets using non-standard bundle ID patterns (e.g., VPN apps with separate extension bundle IDs), set `IOS_PROVISIONING_PROFILE_MAP`:

1. Add environment variable `IOS_PROVISIONING_PROFILE_MAP` to the build environment.
2. Set value to a JSON object mapping bundle IDs to target names:

```json
{"com.example.wireguard":"WireGuardExtension","com.example.tunnelprovider":"PacketTunnel"}
```

Automatic detection works for standard patterns where extension bundle IDs are prefixed with the main app bundle ID (e.g., `com.example.app.widget`).

## Private npm Packages

To install private npm packages during builds:

1. Add an environment secret (e.g., `NPM_REGISTRY_TOKEN`) with the auth token from the npm registry provider.

2. Create `.npmrc` in the project root:

```
@your-scope:registry=https://your-registry.example.com
//your-registry.example.com/:_authToken=${NPM_REGISTRY_TOKEN}
```

Replace `@your-scope`, the registry URL, and `${NPM_REGISTRY_TOKEN}` with actual values.

### Separate Local and CI Config

If local development breaks with the new `.npmrc`:

1. Create `.npmrc.capawesome` with the registry config.
2. Add environment variable `NPM_CONFIG_USERCONFIG` with value `/tmp/capawesome/repo/.npmrc.capawesome`.

## Override Java Version

Capawesome Cloud auto-detects Java version from the Android Gradle Plugin version:

| Android Gradle Plugin | Java Version |
| --------------------- | ------------ |
| 8.0.0 – 8.7.1        | 17           |
| >= 8.7.2              | 21           |

To override, set `JAVA_VERSION` environment variable to the major version (e.g., `17` or `21`).

## Web Build Script

Capawesome Cloud determines the web build script in this order:

1. `capawesome:build` script in `package.json` (if defined)
2. `build` script in `package.json`
3. Build fails if neither exists

Use `capawesome:build` for CI-specific build configurations:

```json
{
  "scripts": {
    "build": "ionic build",
    "capawesome:build": "ionic build --prod"
  }
}
```

The `webBuildCommand` in `capawesome.config.json` overrides automatic script detection.
