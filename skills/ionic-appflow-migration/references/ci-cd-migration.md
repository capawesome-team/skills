# CI/CD Migration

Replace Ionic Appflow CLI commands and credentials in external CI/CD pipelines with their Capawesome CLI equivalents.

Skip this guide if the project has no external CI/CD pipeline and relies on build automations only — the import already recreated the Appflow automations in Capawesome Cloud, so builds keep triggering on the same branches.

**Before editing any pipeline file:**

1. Collect the new Capawesome Cloud app IDs from the import summary (or `npx @capawesome/cli apps:list --json`). Every `--app-id` value changes — Appflow app IDs are not valid in Capawesome Cloud.
2. Check the `renames` entries of the import summary. Apps, certificates, channels, configurations, destinations, and automations that hit a name conflict were given a numeric suffix (e.g. `prod (2)`), and pipelines that reference them by name must use the new name.

## Command Mapping

| Appflow CLI Command | Capawesome CLI Equivalent |
|---|---|
| `appflow live-update upload-artifact` | `npx @capawesome/cli apps:liveupdates:upload` |
| `appflow live-update upload-artifact --signing-key <key>` | `npx @capawesome/cli apps:liveupdates:upload --private-key <key>` |
| `appflow live-update generate-signing-key` | `npx @capawesome/cli apps:liveupdates:generatesigningkey` |
| `appflow live-update create-channel` | `npx @capawesome/cli apps:channels:create` |
| `appflow live-update delete-channel` | `npx @capawesome/cli apps:channels:delete` |
| `appflow live-update list-channels` | `npx @capawesome/cli apps:channels:list` |
| `appflow live-update download-artifact` | `npx @capawesome/cli apps:builds:download --zip --build-id <BUILD_ID>` — downloads a web build artifact by build ID; there is no channel-based download command. Find the build ID via `apps:builds:list` or the Console |
| `appflow live-update set-native-versions` | `npx @capawesome/cli apps:liveupdates:setnativeversions --build-id <BUILD_ID>` with `--android-min`/`--android-max`/`--android-eq` and `--ios-min`/`--ios-max`/`--ios-eq`. Prefer versioned channels for new setups (see `live-update-sdk-migration.md`) |
| `appflow build android` / `appflow build ios` | `npx @capawesome/cli apps:builds:create --platform android` / `--platform ios` |
| `appflow build web` + `appflow deploy web` | `npx @capawesome/cli apps:liveupdates:create` — builds the web assets and deploys them to a channel in one command via Cloud Runners |
| `appflow deploy android` / `appflow deploy ios` | `npx @capawesome/cli apps:deployments:create` |

Legacy pipelines may invoke the same commands via the `ionic-cloud` binary instead of `appflow` — replace those the same way.

## 1. Replace Authentication

```diff
-# Ionic Appflow authentication
-IONIC_TOKEN: ${{ secrets.IONIC_TOKEN }}
+# Capawesome Cloud authentication
+- run: npx @capawesome/cli login --token ${{ secrets.CAPAWESOME_CLOUD_TOKEN }}
```

Generate a token in the [Capawesome Cloud Console](https://console.cloud.capawesome.io/settings/tokens) and store it as a CI/CD secret named `CAPAWESOME_CLOUD_TOKEN`. Appflow personal access tokens do not work with the Capawesome CLI.

## 2. Replace Build Commands

```diff
-appflow build android release --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA> --signing-cert="Android Release"
+npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref <COMMIT_SHA> --certificate "<CERTIFICATE_NAME>" --yes
```

```diff
-appflow build ios app-store --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA> --signing-cert="iOS Distribution"
+npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --type app-store --git-ref <COMMIT_SHA> --certificate "<CERTIFICATE_NAME>" --yes
```

`--git-ref` accepts a branch, tag, or commit SHA, so pass through the revision the pipeline was triggered for — never hardcode a branch. Add `--detached` for non-blocking builds in CI/CD pipelines.

## 3. Replace Live Update Upload Commands

Appflow deploys live updates in two steps (`appflow build web` followed by `appflow deploy web`). Replace both with a web build in the pipeline (e.g. `npm run build`) followed by a single upload of its output folder (e.g. `www` or `dist`) — `--path` is required in non-interactive environments:

```diff
-appflow build web --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA>
-appflow deploy web --app-id=<APPFLOW_APP_ID> --build-id=<BUILD_ID> --destination=production
+npm run build
+npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --channel production --path <WEB_ASSETS_DIR> --yes
```

Alternatively, to keep building the web assets in the cloud (the closest equivalent to Appflow's build-and-deploy pipeline), use `apps:liveupdates:create`, which builds and deploys in one command via Cloud Runners:

```diff
-appflow build web --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA>
-appflow deploy web --app-id=<APPFLOW_APP_ID> --build-id=<BUILD_ID> --destination=production
+npx @capawesome/cli apps:liveupdates:create --app-id <APP_ID> --channel production --git-ref <COMMIT_SHA> --yes
```

## 4. Replace App Store Deploy Commands

```diff
-appflow deploy android --app-id=<APPFLOW_APP_ID> --build-id=<BUILD_ID> --destination="Google Play"
+npx @capawesome/cli apps:deployments:create --app-id <APP_ID> --build-number <BUILD_NUMBER> --destination "<DESTINATION_NAME>"
```

## 5. Remove Ionic Appflow Dependencies

After verifying the new pipeline works:

1. Remove any `@ionic/appflow` or Appflow-related npm packages from `package.json`.
2. Remove Ionic Appflow environment variables from CI/CD secrets (e.g., `IONIC_TOKEN`).
3. Remove `appflow.config.json`, `.appflow.yaml`, or similar Appflow configuration files from the project root.
