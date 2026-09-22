---
name: ionic-appflow-migration
description: "Guides the agent through migrating an existing Ionic Appflow project to Capawesome Cloud. Imports an Ionic Appflow organization export with the Capawesome CLI apps:import command — apps, signing certificates, environments, live update channels, native configurations, store destinations, build automations, and repository links — then covers the remaining work: swapping the Live Update SDK (@capacitor/live-updates or cordova-plugin-ionic), mapping configuration options, completing the manual follow-ups, updating CI/CD pipelines, and verifying builds, live updates, and store deployments. Includes a manual fallback for teams without Appflow organization admin access. Pure Cordova apps use @capawesome/cordova-live-update. Do not use for setting up Capawesome Cloud from scratch without an existing Appflow project, for non-Capacitor mobile frameworks, or for migrating Ionic Enterprise plugins (Auth Connect, Identity Vault, Secure Storage) — use the ionic-enterprise-sdk-migration skill instead."
license: MIT
compatibility: "Requires Node.js 18+, npm, network access, access to the project repository, the Capawesome CLI 4.20.0 or later, and a Capawesome Cloud account. The automated import additionally requires admin access to the Ionic Appflow organization."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/ionic-appflow-migration
---

# Ionic Appflow Migration

Migrate an existing Ionic Appflow project to Capawesome Cloud.

Most of the migration is automated: Ionic Appflow can export an organization as a zip file, and the Capawesome CLI imports it with a single command — apps, signing certificates, environments, live update channels, native configurations, store destinations, and build automations included. The remaining work is swapping the Live Update SDK in the app, connecting the Git provider, and updating CI/CD pipelines.

Ionic Appflow reaches **end of life on December 31, 2027**, and new customer sales already stopped in February 2025. Existing customers keep access until the shutdown date, but every Appflow project needs a migration plan — migrating early leaves room for a gradual, low-risk rollout.

The official guide for this migration is [Migrating from Ionic Appflow](https://capawesome.io/docs/cloud/migrations/ionic-appflow/). Where it disagrees with this skill, follow the documentation.

## Prerequisites

1. A **Capacitor 6, 7, or 8** app currently using Ionic Appflow. For **pure Cordova apps** (without Capacitor), migrate live updates to the Cordova Live Update SDK ([`@capawesome/cordova-live-update`](https://github.com/capawesome-team/cordova-live-update)) instead — the import and all Capawesome Cloud steps apply to both.
2. Node.js 18+ and npm installed.
3. Access to the project's source code repository.
4. A [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.
5. **Admin access to the Ionic Appflow organization** — required to create the export. Without it, skip Steps 3 to 7 and use the [Manual Fallback](#manual-fallback-migrating-without-the-organization-export) instead.

## Hard Rule: Never Read the Export

The Ionic Appflow export is a complete credential dump — environment secrets, keystore and `.p12` passwords, provisioning profiles, Google service account private keys, and Apple app-specific passwords, all in plaintext. **Treat the export zip as opaque.** These rules override every other instruction in this skill:

- **NEVER unzip, extract, open, read, `cat`, `grep`, list, parse, or inspect** the export zip or any file inside it — including `manifest.json`, `app-detail.json`, any other JSON, keystores, `.p12` files, `.mobileprovision` files, and `json-key.json` — with any tool (shell commands, file reading, or code). Do **not** "check" its structure, and do **not** look up app names or IDs inside it.
- **NEVER print, log, paste, summarize, or store** any content of the export, and **NEVER ask the user to paste** export contents, secret values, or passwords into the chat.
- The **ONLY** allowed interaction with the export is passing its path to `npx @capawesome/cli apps:import --file <path>` and reading the CLI's own summary output. That summary contains non-sensitive values only (app names, resource counts, warnings, renames). Use `--dry-run` to discover the app names and IDs needed for `--include` — never the zip itself.
- The same rule applies to the [Manual Fallback](#manual-fallback-migrating-without-the-organization-export): **never open** keystores, `.p12` files, provisioning profiles, service account keys, or environment secret values. Ask the user for **file paths** and pass them to the CLI (e.g. `apps:certificates:create --file`, `apps:environments:set --secret-file`) without opening them. Passwords and secret values must be supplied by the user directly to the CLI — through its interactive prompts or the user's own shell — and must never be typed, echoed, or relayed by the agent.
- After the migration, **remind the user to delete the export zip**. The agent must not delete, move, copy, or rename it on its own.

## General Rules

- Before running any `@capawesome/cli` command for the first time, run it with the `--help` flag to review all available options.
- **Run the import exactly once.** A second run does not update existing apps — it creates renamed duplicates. Complete Step 2 (Git connection) before importing.
- Never commit the export zip, and never add it to the repository — not even temporarily.
- Do not remove Ionic Appflow configuration until the corresponding Capawesome Cloud feature is fully set up and verified. The import does not change anything in Ionic Appflow, so both platforms can run in parallel during the transition.
- Determine the Capacitor version from `package.json` (`@capacitor/core`) before making any changes — it affects which plugin version and update strategy to use.

## MCP Server

The [Capawesome MCP server](https://capawesome.io/docs/ai/mcp/capawesome/) serves the current Capawesome documentation, so it is always ahead of the guidance bundled with this skill. With an API token it also exposes the Capawesome Cloud management API.

- **If the Capawesome MCP tools are available**, call `search_docs` for the topic and read the matching page with `get_doc_page` before applying the guidance below. Where the two disagree, follow the documentation. The `cloud_*` tools can carry out the Capawesome Cloud steps in this skill directly — creating apps, triggering builds, deploying to channels and stores, rolling back, and diagnosing failed jobs — as an alternative to the Capawesome CLI.
- **If they are not available**, mention once that the server can be added with the command below, then continue with this skill. Never block on it.

```bash
claude mcp add --transport http capawesome "https://mcp.capawesome.io/mcp"
```

The documentation tools need no account and no token. See the `capawesome-mcp` skill for full setup, including the Capawesome Cloud tools.

## Procedures

### Step 1: Detect Ionic Appflow Usage

Scan the project to determine which Appflow features are in use. This decides which of the later steps apply — the import itself always covers the Capawesome Cloud side.

#### 1.1 Check for Live Updates

Search for these signals:

1. `@capacitor/live-updates` in `package.json` (dependencies or devDependencies).
2. `cordova-plugin-ionic` in `package.json` — this is the legacy Cordova SDK for Ionic Live Updates.
3. A `LiveUpdates` key inside the `plugins` object in `capacitor.config.ts` or `capacitor.config.json`.
4. Imports of `@capacitor/live-updates` or `cordova-plugin-ionic` in TypeScript/JavaScript source files.

If any signal is found, mark **Live Updates** as in use. Also record which SDK is in use (`@capacitor/live-updates` or `cordova-plugin-ionic`).

Record the current configuration values: `appId` (Ionic Appflow app ID), `autoUpdateMethod` (`background`, `always`, or `none`), `channel`, `enabled`, and `maxVersions`.

#### 1.2 Check for Native Builds

Search for these signals:

1. Appflow CLI build invocations in CI/CD configuration files (e.g., `.github/workflows/*.yml`, `.gitlab-ci.yml`, `bitrise.yml`, `Jenkinsfile`, `azure-pipelines.yml`). The Appflow CLI binary is `appflow` (legacy: `ionic-cloud`), so grep for `appflow build` and `ionic-cloud build`.
2. References to `dashboard.ionicframework.com` or `appflow.ionic.io` in CI/CD files or scripts.
3. An `appflow.config.json`, `.appflow.yaml`, or similar Appflow configuration file in the project root.

If any signal is found, mark **Native Builds** as in use.

#### 1.3 Check for App Store Publishing

Search for these signals:

1. Appflow CLI store deploy invocations in CI/CD configuration files — grep for `appflow deploy android`, `appflow deploy ios`, and the legacy `ionic-cloud deploy` (note: `appflow deploy web` deploys a live update to a channel and belongs to Live Updates, not App Store Publishing).
2. Appflow deploy destinations configured for app store submission (e.g., `appflow destination` commands or store destinations referenced via `--destination`).

If any signal is found, mark **App Store Publishing** as in use.

#### 1.4 Present Findings

Present the detected features to the user and ask:

1. Which features to migrate (all detected features or a subset).
2. Whether the user has **admin access to the Ionic Appflow organization**. If not, continue with the [Manual Fallback](#manual-fallback-migrating-without-the-organization-export).

### Step 2: Prepare Capawesome Cloud

#### 2.1 Install and Authenticate the CLI

```bash
npx @capawesome/cli login
```

The `apps:import` command requires **Capawesome CLI 4.20.0 or later**. Verify with `npx @capawesome/cli --version` and upgrade if the installed version is older.

#### 2.2 Connect the Git Provider (Before the Import)

The import links each app's Git repository, but only if the target organization already has a connection for that provider. Without it, the import warns and continues **without linking** those repositories, and each one has to be linked manually afterwards.

Ask the user to connect the provider used by the apps' repositories — GitHub, GitLab, Bitbucket, Azure DevOps, Gitea, or a Git server over HTTP/S — at organization level in the [Capawesome Cloud Console](https://console.cloud.capawesome.io/organizations/_/git). See [Integrations](https://capawesome.io/docs/cloud/integrations/) for provider-specific setup.

Wait for the user's confirmation before continuing. This cannot be done after the import without manual rework.

#### 2.3 Do Not Create Apps Manually

Do **not** run `apps:create`. The import creates every app from the export. An app created upfront causes a name conflict, and the imported app is renamed with a numeric suffix (e.g. `prod (2)`).

### Step 3: Export the Ionic Appflow Organization

The export cannot be triggered from the CLI. Instruct the user to perform these steps in the Appflow Dashboard:

1. Open the [Ionic Appflow Dashboard](https://dashboard.ionicframework.com) and select the organization.
2. Navigate to **Settings** → **Account**.
3. In the **Organization export** section, click **Export**.
4. Once the export completes, click **Download** to save the zip file.

The export contains **all apps of the organization**, not just the current project.

**The zip file contains sensitive data in plaintext** — environment secrets, signing certificates with their passwords, and store credentials. Tell the user to store it outside the repository, never commit it, and delete it after the migration.

Ask the user for the local path to the downloaded zip file and use it as `<EXPORT_ZIP>` below.

> **Hard rule:** Do not open, unzip, list, or inspect `<EXPORT_ZIP>` — pass the path to the CLI and nothing else.

### Step 4: Preview the Import

Always run a dry run first. It creates nothing and prints exactly what the real import would do:

```bash
npx @capawesome/cli apps:import --file <EXPORT_ZIP> --dry-run
```

> **Hard rule:** The dry run is the only way to learn what the export contains. Never unzip or read the export to look up app names, IDs, or any other value.

The CLI prompts for the target organization. Options:

| Option | Purpose |
|---|---|
| `--file <path>` | Path to the export file (`.zip`). Prompted if omitted. |
| `--organization-id <id>` | Target organization. Prompted if omitted; **required** in non-interactive environments. |
| `--include <apps>` | Import only specific apps, by name or ID from the export. Repeatable or comma-separated. Defaults to all apps. |
| `--ionic-app-type <capacitor\|cordova>` | Resolves apps exported with the ambiguous app type `ionic`. Prompted per app when omitted; **required** in non-interactive environments, otherwise those apps are skipped. |
| `--dry-run` | Preview without creating any resources. |
| `--json` | Print the summary as JSON. |

When `--include` is omitted and the export holds more than one app, the CLI prompts interactively which apps to import (all preselected). Because the export contains the whole organization, prefer `--include` when the user only wants specific apps:

```bash
npx @capawesome/cli apps:import --file <EXPORT_ZIP> --include "My App,My Other App" --dry-run
```

Review the printed table and warnings together with the user before continuing. Resolve anything that is cheaper to fix now than later — most importantly a missing Git connection (Step 2.2).

### Step 5: Run the Import

Run the same command without `--dry-run`:

```bash
npx @capawesome/cli apps:import --file <EXPORT_ZIP>
```

> **Hard rule:** The CLI reads the export — the agent never does. Work only from the printed summary.

The import creates, per app:

| Ionic Appflow | Capawesome Cloud | Notes |
|---|---|---|
| App | App | Name conflicts and case-only duplicates get a numeric suffix (e.g. `prod (2)`) |
| Live update channel | Channel | Same concept, same naming freedom |
| Environment (variables & secrets) | Environment | Variables and secrets are both imported |
| Signing certificate | Signing certificate | Keystores, `.p12` files, and provisioning profiles |
| Native config | Configuration | App name and bundle ID overrides |
| Destination | Destination | Store credentials incl. the Google service account key |
| Native and web build automation | Automation | A web automation with multiple channels becomes one automation per channel |
| Repository association | Git repository link | Requires the connected Git provider from Step 2.2 |
| Latest build number | Next build number | New builds continue the Appflow build numbering |

Run this command **once**. Re-running it creates a second set of renamed apps instead of updating the existing ones.

### Step 6: Review the Import Summary

The CLI prints a table of created resources per app, followed by per-app notes, renames, errors, and the Console URL of each created app. The exit code is `1` if any resource failed.

Walk through the output with the user and act on it:

| Message | Action |
|---|---|
| **Renames** (`... was renamed to ...`) | Note the new names. Every CI/CD workflow, script, or automation that references the old name must be updated (see Step 9). |
| **Automation webhooks were not imported** | Capawesome Cloud supports webhooks at the app level instead of per automation. Add them in Step 7. |
| **Native configurations contain Live Update plugin settings** | These belong in the app itself (`capacitor.config` / `config.xml`), not in a Capawesome Cloud configuration. Handled in Step 8. |
| **Web previews are not supported** | Capawesome Cloud has no web preview equivalent. Tell the user and drop the feature. |
| **Repository was not linked** | The Git provider is not connected. Connect it and link the repository manually in Step 7. |
| **React Native / Flutter apps were skipped** | Support is coming. Keep the export until then and re-run the import later with `--include` for those apps only. |
| **Unsupported build type / unknown reference skipped** | The automation was created without that value. Set it manually in the Console. |
| **Errors** for individual resources | Create the failed resource manually in the Console. Do **not** re-run the whole import. |

### Step 7: Complete the Manual Follow-Ups

These cannot be imported and need a one-time manual setup. Present the list to the user and offer to do the parts that are possible from the CLI:

1. **Link unlinked repositories** — connect the Git provider at organization level, then select the repository on the **Git repository** page of each app.
2. **Add app-level webhooks** — if Appflow automations used webhooks, set them up per app (see [Webhooks](https://capawesome.io/docs/cloud/webhooks/)).
3. **Add an App Store Connect API key** — the import carries over Apple ID credentials only. For automated publishing, add a `.p8` [App Store Connect API key](https://capawesome.io/docs/cloud/app-store-publishing/destinations/apple-app-store/) to the imported destination.
4. **Create API tokens for CI/CD** — Appflow personal access tokens do not work. Create [Capawesome Cloud API tokens](https://capawesome.io/docs/cloud/accounts/tokens/) instead (see Step 9).
5. **Re-create teams and members** — invite the team in the Capawesome Cloud Console.

Not migrated at all, by design: build and deployment history, live update deployment history, dependency caching settings, and web previews. The apps in Ionic Appflow stay untouched.

### Step 8: Swap the Live Update SDK

Skip this step if Live Updates was not detected or the user chose not to migrate it.

#### 8.1 Replace the Package

If the project uses `@capacitor/live-updates`:

```bash
npm uninstall @capacitor/live-updates
```

If the project uses the legacy Cordova SDK (`cordova-plugin-ionic`):

```bash
npm uninstall cordova-plugin-ionic
```

Install the Capawesome plugin version matching the project's Capacitor version:

- **Capacitor 8**: `npm install @capawesome/capacitor-live-update@latest`
- **Capacitor 7**: `npm install @capawesome/capacitor-live-update@v7-lts`
- **Capacitor 6**: `npm install @capawesome/capacitor-live-update@v6-lts`

For a pure Cordova app, install `@capawesome/cordova-live-update` instead and configure it via `<preference>` entries in `config.xml`.

#### 8.2 Update the Capacitor Configuration

Replace the `LiveUpdates` plugin config with `LiveUpdate` in `capacitor.config.ts` (or `.json`). Use the **Capawesome Cloud app ID** from the import summary — not the Appflow app ID. Map the options as follows:

| Ionic Appflow (`LiveUpdates`) | Capawesome Cloud (`LiveUpdate`) | Notes |
|---|---|---|
| `appId` | `appId` | Replace with the imported Capawesome Cloud app ID |
| `autoUpdateMethod: 'background'` | `autoUpdateStrategy: 'background'` | Same behavior |
| `autoUpdateMethod: 'always'` | `autoUpdateStrategy: 'background'` | Also add a `nextBundleSet` listener |
| `autoUpdateMethod: 'none'` | *(omit `autoUpdateStrategy`)* | Use manual sync code instead |
| `channel` | `defaultChannel` | Same value |
| `enabled` | *(remove)* | Not needed — controlled in code |
| `maxVersions` | `autoDeleteBundles: true` | Boolean instead of number |

```diff
 // capacitor.config.ts
 const config: CapacitorConfig = {
   plugins: {
-    LiveUpdates: {
-      appId: 'abc12345',
-      autoUpdateMethod: 'background',
-      channel: 'production',
-      maxVersions: 3
-    }
+    LiveUpdate: {
+      appId: '<CAPAWESOME_APP_ID>',
+      autoUpdateStrategy: 'background',
+      defaultChannel: 'production',
+      autoDeleteBundles: true,
+      autoBlockRolledBackBundles: true,
+      readyTimeout: 10000
+    }
   }
 };
```

If the import reported Live Update plugin settings inside a native configuration, apply those values here — Capawesome Cloud configurations only cover the app name and bundle ID.

#### 8.3 Update the Source Code

Read `references/live-update-sdk-migration.md` **before editing any source file**. It covers the import and API changes, the `sync()` return value, the per-strategy code (`background`, `always`, `none`, force update), rollback protection, native version compatibility, and the iOS privacy manifest.

For projects on the legacy Cordova SDK, read `references/cordova-sdk-migration.md` as well for the complete `Deploy` → `LiveUpdate` method mapping and the native configuration cleanup.

Finally, sync the native projects:

```bash
npx cap sync
```

### Step 9: Update CI/CD Pipelines

Skip this step if the project has no external CI/CD pipeline — the imported automations already trigger builds on the same branches.

Read `references/ci-cd-migration.md` and apply it: the full Appflow → Capawesome CLI command mapping, token-based authentication, and the removal of Appflow dependencies. Every `--app-id` changes, and resources renamed during the import must be referenced by their new name.

### Step 10: Test and Verify

#### 10.1 Verify Live Updates (if migrated)

Ask the user whether to test live update functionality. If accepted:

1. Make a small, visible change in the app (e.g., append " - Migration Test" to a heading).
2. Build the web assets: `npm run build`.
3. Check existing channels:
   ```bash
   npx @capawesome/cli apps:channels:list --app-id <APP_ID> --json
   ```
4. Create the target channel if missing:
   ```bash
   npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name <CHANNEL_NAME>
   ```
5. Upload the bundle:
   ```bash
   npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --channel <CHANNEL_NAME>
   ```
6. Revert the visible change, rebuild, and sync:
   ```bash
   npm run build && npx cap sync
   ```
7. Open the native project (`npx cap open ios` or `npx cap open android`) and run on a device or emulator.
8. Verify the live update is applied. With `autoUpdateStrategy: "background"`, wait for the update prompt or force-close and reopen the app, or switch away from the app and return after more than 15 minutes to trigger a check.

#### 10.2 Verify Native Builds (if migrated)

Trigger a test build and verify the artifact is produced:

```bash
npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --git-ref main
```

#### 10.3 Verify App Store Publishing (if migrated)

Deploy a test build to the configured destination and verify it appears in TestFlight or Google Play Console:

```bash
npx @capawesome/cli apps:deployments:create --app-id <APP_ID> --build-number <BUILD_NUMBER> --destination "<DESTINATION_NAME>"
```

#### 10.4 Clean Up

After all features are verified:

1. Confirm all Ionic Appflow SDK packages are removed from `package.json`.
2. Confirm all Ionic Appflow configuration is removed from `capacitor.config.ts`/`capacitor.config.json`.
3. Confirm all Ionic Appflow CI/CD commands and environment variables are removed.
4. **Remind the user to delete the export zip file** and any copies of it. The agent must not delete, move, or rename it itself.

## Manual Fallback: Migrating Without the Organization Export

Use this path only when the user has no admin access to the Ionic Appflow organization, or the export is unavailable. Everything is recreated by hand; Steps 1, 8, 9, and 10 stay the same.

> **Hard rule:** Credential files stay closed here too. Ask the user for **file paths** to keystores, `.p12` files, provisioning profiles, and service account keys, and pass those paths to the CLI (`--file`, `--provisioning-profiles`, `--google-service-account-key-file`, `--apple-api-key-file`, `--secret-file`) without opening them. Let the user enter passwords and secret values into the CLI's interactive prompts or run the command in their own shell — never type, echo, or relay a secret.

### F.1 Create the App

```bash
npx @capawesome/cli apps:create
```

Save the returned **app ID** (UUID) — it replaces the Appflow app ID in `capacitor.config` and in all CI/CD commands.

### F.2 Recreate Live Update Channels

Recreate each Appflow live update channel:

```bash
npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name <CHANNEL_NAME>
```

### F.3 Recreate Native Builds

Connect the Git repository, upload signing certificates, and configure environments and native configurations. Use the `capawesome-cloud` skill — follow the **Native Builds** section starting from "Connect Git Repository" and read its `references/native-builds.md` for the full procedure. The original values (keystore and `.p12` passwords, environment variables and secrets) live in the Appflow Dashboard — the **user** looks them up and enters them into the CLI prompts. Never ask for them in the chat.

### F.4 Recreate App Store Publishing

Create destinations and configure the Apple App Store and/or Google Play Store credentials. Use the `capawesome-cloud` skill — follow the **App Store Publishing** section and read its `references/app-store-publishing.md`.

### F.5 Recreate Automations

Recreate each Appflow build automation:

```bash
npx @capawesome/cli apps:automations:create --app-id <APP_ID>
```

Then continue with Step 8.

## Error Handling

### Import

- `The provided file does not look like an Ionic Appflow export` → Ask the user to confirm the file is the organization export downloaded from **Settings → Account**, not a re-zipped folder or a repository archive, and to re-download it if in doubt. Do **not** open the zip to check.
- `The export file ... has an unexpected format` / format version warning → Update the CLI (`npx @capawesome/cli@latest`) and re-run the dry run.
- `No apps found in the export that match the provided filters` → `--include` values must match an app **name or ID exactly**. Run the dry run without `--include` to have the CLI list the available apps.
- App skipped because of the ambiguous app type `ionic` → Pass `--ionic-app-type capacitor` or `--ionic-app-type cordova`, or run the import interactively to be prompted per app.
- App skipped as "not yet supported" (React Native, Flutter) → Expected. Keep the export and re-run the import for those apps with `--include` once support ships.
- Import finished with errors (exit code `1`) → Individual resources failed. Create them manually in the Console. Never re-run the full import to fix them — it duplicates the apps.
- Apps imported twice by mistake → Delete the duplicated apps in the Capawesome Cloud Console (`apps:delete`), then keep the originals.
- Repositories not linked → The organization had no matching Git connection at import time. Connect the provider, then link each repository on the app's **Git repository** page.

### Live Updates

- `npm uninstall @capacitor/live-updates` fails → The package may be listed under a different name. Search `package.json` for any package containing `live-updates` from the `@capacitor` scope.
- `npx cap sync` fails after plugin swap → Verify the installed `@capawesome/capacitor-live-update` version matches the Capacitor version in `package.json`.
- App reverts to default bundle after restart → `LiveUpdate.ready()` is likely not called early enough. Add it as the first call in app initialization.
- Updates not detected with `autoUpdateStrategy: "background"` → Updates are only checked if the last check was >15 minutes ago. Force-close and restart the app. Check device logs for Live Update SDK output.
- `LiveUpdates is not defined` or similar runtime errors → Ensure all imports were updated from `@capacitor/live-updates` to `@capawesome/capacitor-live-update` and the class name changed from `LiveUpdates` to `LiveUpdate`.
- `Deploy is not defined` → The project uses the legacy Cordova SDK (`cordova-plugin-ionic`). Read `references/cordova-sdk-migration.md` for the full method mapping.
- `activeApplicationPathChanged is not a property` or sync result checks fail → The Capawesome SDK returns `{ nextBundleId }` instead of `{ activeApplicationPathChanged }`. Update all sync result checks.
- `setConfig is not a function` → The installed plugin release is outdated. Update to the latest release of the matching dist-tag (`@latest` for Capacitor 8, `@v7-lts` for 7, `@v6-lts` for 6).

### Native Builds

- Build fails with authentication error → Re-run `npx @capawesome/cli login`. For CI/CD, verify the `CAPAWESOME_CLOUD_TOKEN` secret is set correctly.
- Build fails with missing signing certificate → The certificate may have failed to import. Upload it via `npx @capawesome/cli apps:certificates:create`. Read the `capawesome-cloud` skill's `references/certificates-android.md` or `references/certificates-ios.md`.
- Build fails with `invalid source release` → Set the `JAVA_VERSION` environment variable in the build environment. Read the `capawesome-cloud` skill's `references/build-troubleshooting.md`.

### App Store Publishing

- Destination creation fails → Verify credentials are correct. Read the `capawesome-cloud` skill's `references/apple-app-store-credentials.md` or `references/google-play-store-credentials.md`.
- Deployment fails with "build not found" → Ensure the build completed successfully before deploying.

### CI/CD

- Pipeline fails after migration → Compare the old and new pipeline configurations side by side. Check that `--app-id` uses the new Capawesome Cloud app ID, that renamed resources are referenced by their new name, and that `CAPAWESOME_CLOUD_TOKEN` is available as a secret.

## Related Skills

- **`capawesome-cloud`** — Referenced for Native Builds and App Store Publishing setup, and for the manual fallback.
- **`capawesome-cli`** — Installation, authentication, and CI/CD usage of the Capawesome CLI used throughout this skill.
- **`ionic-enterprise-sdk-migration`** — If the project also uses discontinued Ionic Enterprise SDK plugins (Auth Connect, Identity Vault, Secure Storage), use this skill to migrate them to Capawesome alternatives.
- **`capgo-cloud-migration`** — Use this skill instead if the project migrates from Capgo rather than Ionic Appflow.
- **`capawesome-mcp`** — Connect an MCP client to the hosted Capawesome MCP server for always-current documentation and Capawesome Cloud management.
