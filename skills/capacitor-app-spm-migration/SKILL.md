---
name: capacitor-app-spm-migration
description: "Guides the agent through migrating an existing Capacitor app project from CocoaPods to Swift Package Manager (SPM) for iOS dependency management. Covers prerequisite checks, inventorying installed Capacitor plugins, backing up customized iOS project files (Info.plist, AppDelegate.swift, Assets.xcassets, Base.lproj, App.entitlements, GoogleService-Info.plist, .xcconfig files, signing configuration), deleting the existing `ios/` folder, re-scaffolding with `npx cap add ios --packagemanager SPM`, restoring preserved files, re-syncing plugins, and verifying the build. Performs all migration steps manually — does not use the interactive `npx cap spm-migration-assistant` command. Do not use for Capacitor plugin projects, app projects already on SPM, app projects without an existing `ios/` folder, or non-Capacitor mobile frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-app-spm-migration
---

# Capacitor App: CocoaPods → SPM Migration

Migrate an existing Capacitor app project's iOS dependency management from CocoaPods to Swift Package Manager (SPM). The migration is destructive — the existing `ios/` folder is deleted and re-scaffolded, then user-authored files are restored from a temporary backup.

## Prerequisites

| Requirement | Version |
| ----------- | ------- |
| Capacitor   | 6+      |
| Node.js     | 18+     |
| Xcode       | 15+     |
| CocoaPods   | installed (only used for verification before deletion) |

The project must be a Capacitor **app** (not a plugin) and must currently use CocoaPods (`ios/App/Podfile` exists).

## Procedures

### Step 1: Verify Project State

1. Verify the project is a Capacitor **app**, not a plugin. Confirm `package.json` lists `@capacitor/ios` in `dependencies` and the project root contains a `capacitor.config.*` file.
2. Verify the project currently uses CocoaPods. The file `ios/App/Podfile` must exist. If it does not exist and `ios/App/CapApp-SPM/Package.swift` exists, the project is already on SPM — abort and inform the user.
3. Read `@capacitor/core` version from `package.json` (`dependencies` or `devDependencies`). Confirm the major version is **6 or higher**. If lower, abort and direct the user to the `capacitor-app-upgrades` skill first.
4. Verify the working tree is clean. Run `git status --porcelain` — if the output is non-empty, instruct the user to commit or stash changes before proceeding. The migration is destructive and must be reversible via `git checkout`.

### Step 2: Inventory Installed Capacitor Plugins

Read `package.json` and record every installed Capacitor plugin under `dependencies`. Match these prefixes:

- `@capacitor/*` (official plugins)
- `@capawesome/*` and `@capawesome-team/*`
- `@capacitor-community/*`
- `@capacitor-firebase/*`
- `@capacitor-mlkit/*`
- `@revenuecat/*`
- Any other package with `"capacitor"` in its `package.json` `keywords` array

Record each plugin's package name and installed version. This list is used in Step 8 to verify all plugins resolve under SPM after re-sync.

### Step 3: Identify Files to Preserve

Build a checklist of files in `ios/` that contain user-authored content and must survive the re-scaffold. For each path below, check whether the file exists. Record only the paths that exist.

| Path (relative to project root) | Purpose |
| ------------------------------- | ------- |
| `ios/App/App/Info.plist` | App permissions, URL schemes, bundle config, custom keys |
| `ios/App/App/AppDelegate.swift` | Custom app lifecycle code |
| `ios/App/App/SceneDelegate.swift` | Custom scene lifecycle code (if present) |
| `ios/App/App/Assets.xcassets/` | App icon, splash assets, color sets |
| `ios/App/App/Base.lproj/` | LaunchScreen and Main storyboards |
| `ios/App/App/App.entitlements` | Push, App Groups, Associated Domains, Keychain Sharing |
| `ios/App/App/GoogleService-Info.plist` | Firebase configuration (if Firebase plugins are installed) |
| `ios/App/App/*.xcconfig` | Build configuration overrides |
| `ios/App/App.xcodeproj/project.pbxproj` | Source for extracting signing values in Step 5 |

Additionally, scan `ios/App/App/` for **any other `.swift` files** beyond `AppDelegate.swift` and `SceneDelegate.swift`. These are user-authored Swift sources (e.g., custom Notification Service Extensions, custom view controllers) and must also be preserved.

If the project contains a Notification Service Extension or other Xcode targets under `ios/App/` (e.g., `ios/App/NotificationService/`), record those directories as well.

### Step 4: Back Up Preserved Files

1. Create the backup directory:

   ```bash
   mkdir -p .spm-migration-backup
   ```

2. Add `.spm-migration-backup/` to `.gitignore` to prevent accidental commits:

   ```diff
    # iOS files
   +.spm-migration-backup/
   ```

3. Copy each file recorded in Step 3 into `.spm-migration-backup/`, preserving the relative path. For example:

   ```bash
   mkdir -p .spm-migration-backup/ios/App/App
   cp ios/App/App/Info.plist .spm-migration-backup/ios/App/App/Info.plist
   cp ios/App/App/AppDelegate.swift .spm-migration-backup/ios/App/App/AppDelegate.swift
   cp -R ios/App/App/Assets.xcassets .spm-migration-backup/ios/App/App/Assets.xcassets
   cp -R ios/App/App/Base.lproj .spm-migration-backup/ios/App/App/Base.lproj
   ```

   Repeat for every file/directory identified in Step 3. Use `cp -R` for directories.

4. Always copy the full `ios/App/App.xcodeproj/project.pbxproj` to `.spm-migration-backup/ios/App/App.xcodeproj/project.pbxproj`, even though the file itself will not be restored. Step 5 reads it to extract signing values, and Step 11 may reference it for capability reconfiguration.

### Step 5: Extract Signing & Bundle Configuration

Read `.spm-migration-backup/ios/App/App.xcodeproj/project.pbxproj` and extract the values of the following build settings from the `App` target's `XCBuildConfiguration` blocks (both `Debug` and `Release`). Search for each key and record the assigned value:

- `PRODUCT_BUNDLE_IDENTIFIER`
- `DEVELOPMENT_TEAM`
- `CODE_SIGN_STYLE` (e.g., `Automatic` or `Manual`)
- `PROVISIONING_PROFILE_SPECIFIER` (only if `CODE_SIGN_STYLE = Manual`)
- `CODE_SIGN_IDENTITY` (only if `CODE_SIGN_STYLE = Manual`)
- `MARKETING_VERSION`
- `CURRENT_PROJECT_VERSION`
- `IPHONEOS_DEPLOYMENT_TARGET`
- `SWIFT_VERSION`

Also record the list of capabilities by reading `.spm-migration-backup/ios/App/App/App.entitlements` (if present). Note all top-level keys (e.g., `aps-environment`, `com.apple.security.application-groups`, `com.apple.developer.associated-domains`).

Save the extracted values in memory for use in Step 9.

### Step 6: Delete the `ios/` Folder

Confirm with the user that the working tree is committed (re-run `git status --porcelain` if necessary) and that `.spm-migration-backup/` contains every file from Step 3. Then run:

```bash
rm -rf ios
```

### Step 7: Re-scaffold iOS with SPM

Run:

```bash
npx cap add ios --packagemanager SPM
```

This creates a new `ios/` directory with the SPM-based scaffold. The new structure includes `ios/App/CapApp-SPM/Package.swift` instead of `ios/App/Podfile`.

### Step 8: Restore Preserved Files

Copy each backed-up file from `.spm-migration-backup/` back to its original path under `ios/`, **overwriting** the scaffolded defaults. For example:

```bash
cp .spm-migration-backup/ios/App/App/Info.plist ios/App/App/Info.plist
cp .spm-migration-backup/ios/App/App/AppDelegate.swift ios/App/App/AppDelegate.swift
cp -R .spm-migration-backup/ios/App/App/Assets.xcassets ios/App/App/Assets.xcassets
cp -R .spm-migration-backup/ios/App/App/Base.lproj ios/App/App/Base.lproj
```

Repeat for every file/directory backed up in Step 4, **except** `ios/App/App.xcodeproj/project.pbxproj` — that file is **not** restored. The new `project.pbxproj` is required for SPM integration; signing values from the old one are reapplied via Step 9.

If any preserved file did not exist in the new scaffold (e.g., `App.entitlements`, `GoogleService-Info.plist`, custom Swift files), the file must additionally be added to the Xcode project membership in Step 10.

### Step 9: Reapply Signing & Bundle Configuration

Edit the new `ios/App/App.xcodeproj/project.pbxproj` and reapply the values extracted in Step 5. For each setting, find every occurrence in the `App` target's `XCBuildConfiguration` blocks and replace the scaffolded value with the recorded value.

Apply this diff pattern for each setting (example for `PRODUCT_BUNDLE_IDENTIFIER`):

```diff
-				PRODUCT_BUNDLE_IDENTIFIER = com.example.app;
+				PRODUCT_BUNDLE_IDENTIFIER = <RECORDED_BUNDLE_ID>;
```

Settings to reapply (use `replace_all` semantics — update **all** occurrences across `Debug` and `Release` configurations):

- `PRODUCT_BUNDLE_IDENTIFIER`
- `DEVELOPMENT_TEAM`
- `CODE_SIGN_STYLE`
- `PROVISIONING_PROFILE_SPECIFIER` (only if manual signing)
- `CODE_SIGN_IDENTITY` (only if manual signing)
- `MARKETING_VERSION`
- `CURRENT_PROJECT_VERSION`

If `IPHONEOS_DEPLOYMENT_TARGET` in the recorded values is **higher** than the scaffolded value, update it. If lower, keep the scaffolded value (the new scaffold targets a supported minimum).

### Step 10: Add Restored Custom Files to the Xcode Project

The new `project.pbxproj` only references files that the scaffold created. Files restored in Step 8 that did **not** exist in the scaffold (e.g., `App.entitlements`, `GoogleService-Info.plist`, custom Swift files, Notification Service Extension targets) are present on disk but invisible to Xcode until they are added to the project's `PBXBuildFile`, `PBXFileReference`, and `PBXGroup` entries.

For each custom file that was not part of the scaffold, instruct the user to perform the following one-time action in Xcode (the SPM scaffold has no `App.xcworkspace` — open `App.xcodeproj` directly):

1. Open the project: `npx cap open ios` (or `open ios/App/App.xcodeproj`).
2. In the Xcode Project Navigator, right-click the `App` group and choose **Add Files to "App"…**.
3. Select each restored custom file (e.g., `App.entitlements`, `GoogleService-Info.plist`, custom `.swift` files).
4. Ensure **Copy items if needed** is **unchecked** and the `App` target is **checked**.

For the `App.entitlements` file specifically, also set the `CODE_SIGN_ENTITLEMENTS` build setting in `ios/App/App.xcodeproj/project.pbxproj`. Apply this diff pattern in **both** `Debug` and `Release` `XCBuildConfiguration` blocks for the `App` target:

```diff
 				CODE_SIGN_STYLE = Automatic;
+				CODE_SIGN_ENTITLEMENTS = App/App.entitlements;
 				CURRENT_PROJECT_VERSION = 1;
```

If the project had additional Xcode targets (e.g., a Notification Service Extension under `ios/App/NotificationService/`), the user must recreate those targets in Xcode. SPM scaffolding does not regenerate non-`App` targets. Inform the user explicitly.

### Step 11: Reconfigure Capabilities

For each capability key recorded from `App.entitlements` in Step 5, verify the corresponding capability is enabled in Xcode:

1. Open `ios/App/App.xcodeproj` via `npx cap open ios`.
2. Select the `App` target → **Signing & Capabilities** tab.
3. For each entitlement key, add the matching capability via the **+ Capability** button:
   - `aps-environment` → **Push Notifications**
   - `com.apple.security.application-groups` → **App Groups**
   - `com.apple.developer.associated-domains` → **Associated Domains**
   - `keychain-access-groups` → **Keychain Sharing**
   - `com.apple.developer.in-app-payments` → **Apple Pay**
   - `com.apple.developer.icloud-services` → **iCloud**

This is the only step that requires Xcode interaction. Inform the user explicitly that this step must be done manually.

### Step 12: Re-sync Plugins

Run:

```bash
npm install
npx cap sync ios
```

`npx cap sync ios` updates `ios/App/CapApp-SPM/Package.swift` to include every Capacitor plugin recorded in Step 2. Watch for warnings — any plugin without SPM support will be reported here.

If a plugin is reported as **incompatible with SPM**, inform the user. The user must either:
- Wait for upstream SPM support from the plugin maintainer, or
- Fork the plugin and add SPM support using the `capacitor-plugin-spm-support` skill, or
- Remove the plugin from the project.

### Step 13: Verify the Build

1. Open the project in Xcode and resolve SPM packages:

   ```bash
   npx cap open ios
   ```

   In Xcode, **File → Packages → Resolve Package Versions** (or wait for automatic resolution on open).

2. Build for a simulator from the command line to confirm everything compiles:

   ```bash
   npx cap run ios
   ```

3. Verify the app launches and that previously working features (push notifications, deep links, Firebase, etc.) still function.

### Step 14: Cleanup

Once the build is verified and the app runs correctly:

1. Delete the backup directory:

   ```bash
   rm -rf .spm-migration-backup
   ```

2. Remove the `.spm-migration-backup/` entry from `.gitignore`.

3. Verify no CocoaPods artifacts remain:

   ```bash
   find ios -name 'Podfile*' -o -name 'Pods' -o -name '*.xcworkspace'
   ```

   If any results appear, delete them. The SPM workflow uses `App.xcodeproj` directly — there is no `App.xcworkspace`.

4. Commit the migration:

   ```bash
   git add -A
   git commit -m "chore(ios): migrate from CocoaPods to Swift Package Manager"
   ```

## Error Handling

- **`npx cap add ios --packagemanager SPM` fails with "ios platform already exists"** — Step 6 was not completed. Re-run `rm -rf ios` and retry.
- **`npx cap sync ios` reports a plugin without SPM support** — that plugin cannot be used under SPM until it ships a `Package.swift`. Inform the user and pause the migration; do not silently drop the plugin.
- **Build fails with "Module 'X' not found"** — the plugin's SPM package was not resolved. Run **File → Packages → Reset Package Caches** in Xcode, then **File → Packages → Resolve Package Versions**.
- **Build fails with code signing errors** — the values reapplied in Step 9 are missing or incorrect. Re-read `.spm-migration-backup/ios/App/App.xcodeproj/project.pbxproj` and confirm `DEVELOPMENT_TEAM` and `PRODUCT_BUNDLE_IDENTIFIER` were applied to **both** `Debug` and `Release` configurations.
- **Push notifications stop working after migration** — the Push Notifications capability was not re-added in Step 11, or `App.entitlements` is not linked via `CODE_SIGN_ENTITLEMENTS` (Step 10).
- **Firebase initializes but no data appears** — `GoogleService-Info.plist` was restored to disk but not added to the Xcode project's target membership in Step 10.
- **Custom Notification Service Extension target is missing** — non-`App` targets are not regenerated by `npx cap add ios`. Recreate the target manually in Xcode and re-add its source files.
- **The user reports they want to revert** — because Step 1 verified a clean working tree, the entire migration can be undone with `git checkout -- .` followed by `git clean -fd ios .spm-migration-backup`.

## Related Skills

- **`capacitor-plugin-spm-support`** — For adding SPM support to a Capacitor **plugin** (not an app). Use this if Step 12 reports a plugin without SPM support and the user controls that plugin's source.
- **`capacitor-plugins`** — For installing or reconfiguring Capacitor plugins after the migration.
- **`capacitor-app-upgrades`** — If the project is on Capacitor 5 or earlier, upgrade it to Capacitor 6+ first using this skill, then return here.
- **`capacitor-app-development`** — For general Capacitor app development topics, troubleshooting, and best practices after the migration.
