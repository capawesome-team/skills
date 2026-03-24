# Project Configuration

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

## Options

| Option | Required | Description | Default |
|--------|----------|-------------|---------|
| `cloud.apps[].appId` | Yes | Capawesome Cloud App ID (UUID from App Settings). | — |
| `cloud.apps[].baseDir` | No | Base directory of the app within the repo. All commands run from here. | Repo root |
| `cloud.apps[].dependencyInstallCommand` | No | Command to install dependencies. | `npm install` |
| `cloud.apps[].webBuildCommand` | No | Command to build web assets. | `npm run build` |

## Web Build Script Priority

Capawesome Cloud determines the web build script in this order:

1. `capawesome:build` script in `package.json` (if defined)
2. `build` script in `package.json`
3. Build fails if neither exists

The `webBuildCommand` in `capawesome.config.json` overrides this detection.

## Examples

### Monorepo

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

### pnpm

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

### Yarn

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
