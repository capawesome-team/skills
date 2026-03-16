# Build Stacks

Build stacks define pre-configured software versions used during the native build process.

## Available Stacks

| Software  | macos-sequoia            | macos-tahoe            |
| --------- | ------------------------ | ---------------------- |
| CocoaPods | 1.16.2                   | 1.16.2                 |
| Fastlane  | 2.226.0                  | 2.229.1                |
| Java      | 17, 21 (default)         | 17, 21 (default)       |
| macOS     | 15                       | 26                     |
| npm       | 11.6.1                   | 11.6.2                 |
| Node.js   | 20, 22, 24 (default)     | 20, 22, 24 (default)   |
| Xcode     | 16.4, 26.0.1 (default)   | 26.2 (default)         |

## Selecting a Stack

Specify via CLI when creating a build:

```bash
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --type simulator --git-ref main --stack macos-sequoia
```

If not specified, the default stack is used.

## Overriding Software Versions

Override default versions by setting reserved environment variables:

- `JAVA_VERSION`: Major version (e.g., `17` or `21`)
- `NODE_VERSION`: Major version (e.g., `20`, `22`, or `24`)
- `XCODE_VERSION`: Major version (e.g., `16` or `26`)

Set these in a custom environment or as ad-hoc variables when creating a build.

## Automatic Java Version Detection

For Android builds, Capawesome Cloud auto-detects the Java version based on the Android Gradle Plugin version:

| Android Gradle Plugin | Java Version |
| --------------------- | ------------ |
| 8.0.0 – 8.7.1        | 17           |
| >= 8.7.2              | 21           |

Override by setting `JAVA_VERSION` if the automatic detection is incorrect.
