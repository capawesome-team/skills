# App Configuration

## capawesome.config.json

Place `capawesome.config.json` in the project root for advanced build configuration (monorepos, subdirectory apps, custom build commands). Not required for standard project setups.

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

### Options

- `cloud.apps[].appId` **(required)**: Capawesome Cloud App ID (UUID from App Settings).
- `cloud.apps[].baseDir`: Base directory of the app within the repo. All commands run from here. Default: repo root.
- `cloud.apps[].dependencyInstallCommand`: Command to install dependencies. Default: `npm install`.
- `cloud.apps[].webBuildCommand`: Command to build web assets. Default: `npm run build`.

### Monorepo Example

```json
{
  "cloud": {
    "apps": [
      {
        "appId": "app-id-1",
        "baseDir": "packages/app1",
        "dependencyInstallCommand": "cd ../.. && npm install",
        "webBuildCommand": "cd ../.. && npm run build --workspace=app1"
      }
    ]
  }
}
```

### pnpm Example

```json
{
  "cloud": {
    "apps": [
      {
        "appId": "your-app-id",
        "dependencyInstallCommand": "npm install -g pnpm && pnpm install",
        "webBuildCommand": "pnpm build"
      }
    ]
  }
}
```

### Yarn Example

```json
{
  "cloud": {
    "apps": [
      {
        "appId": "your-app-id",
        "dependencyInstallCommand": "npm install -g yarn && yarn install",
        "webBuildCommand": "yarn build"
      }
    ]
  }
}
```

## Web Build Script Priority

Capawesome Cloud determines the web build script in this order:

1. `capawesome:build` script in `package.json` (if defined)
2. `build` script in `package.json`
3. Build fails if neither exists

The `webBuildCommand` in `capawesome.config.json` overrides this detection.

## Native Configuration Overwriting (Trapeze)

Use [Trapeze](https://trapeze.dev/) to overwrite native Android/iOS configurations before building (e.g., app ID, display name, version numbers).

### Install Trapeze

```bash
npm install --save-dev @trapezedev/configure
```

### Create capawesome.yml

Place `capawesome.yml` in the project root:

```yaml
platforms:
  android:
    packageName: com.example.myapp
    appName: My App
  ios:
    bundleId: com.example.myapp
    displayName: My App
```

### Add Build Script

Add to `package.json`:

```json
{
  "scripts": {
    "capawesome:build": "if [ \"$CI_PLATFORM\" = \"ios\" ] || [ \"$CI_PLATFORM\" = \"android\" ]; then npx trapeze run capawesome.yml -y --$CI_PLATFORM; fi && npm run build"
  }
}
```

The `capawesome:build` script runs Trapeze conditionally for iOS/Android before the web build. The `$CI_PLATFORM` variable is set automatically by Capawesome Cloud.

### Using Environment Variables in Trapeze

Define variables in `capawesome.yml`:

```yaml
vars:
  APP_ID:
  APP_NAME:

platforms:
  android:
    packageName: $APP_ID
    appName: $APP_NAME
  ios:
    bundleId: $APP_ID
    displayName: $APP_NAME
```

Set `APP_ID` and `APP_NAME` in the Capawesome Cloud environment. Different environments can have different values.

Default values can be specified:

```yaml
vars:
  APP_ID:
    default: com.example.default
```
