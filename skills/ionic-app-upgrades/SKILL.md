---
name: ionic-app-upgrades
description: "Guides the agent through upgrading an Ionic Framework app to a newer major version. Supports upgrades from Ionic 4 through 8, including multi-version jumps. Covers framework-specific migration steps for Angular, React, and Vue, component breaking changes, CSS variable updates, and browser support changes. Do not use for Capacitor version upgrades (use capacitor-app-upgrades instead), for plugin library upgrades, or for non-Ionic UI frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/ionic-app-upgrades
---

# Ionic App Upgrade

Upgrade an Ionic Framework app to a newer major version (4 to 5, 5 to 6, 6 to 7, or 7 to 8).

## Prerequisites

| Target Version | Angular  | React | Vue     |
| -------------- | -------- | ----- | ------- |
| 5              | 8.2+     | 16+   | N/A     |
| 6              | 12+      | 17+   | 3.0.6+  |
| 7              | 14+      | 17+   | 3.0.6+  |
| 8              | 16+      | 17+   | 3.0.6+  |

## Procedures

### Step 1: Detect Current Version

Read `@ionic/angular`, `@ionic/react`, or `@ionic/vue` version from `package.json` (`dependencies` or `devDependencies`). Determine the current major version and which framework is in use (Angular, React, Vue, or Core/Standalone).

Ask the user for the target version. Default to the latest (8) if not specified.

### Step 2: Execute Upgrade

For each major version jump between the current and target version, apply the corresponding upgrade guide **sequentially**:

| Current to Target | Reference                            |
| ----------------- | ------------------------------------ |
| 4 to 5            | `references/upgrade-v4-to-v5.md`     |
| 5 to 6            | `references/upgrade-v5-to-v6.md`     |
| 6 to 7            | `references/upgrade-v6-to-v7.md`     |
| 7 to 8            | `references/upgrade-v7-to-v8.md`     |

For multi-version jumps (e.g., 5 to 8), apply each upgrade in order:
1. Read and apply `references/upgrade-v5-to-v6.md`
2. Build the project (`npm run build`), fix any errors, and verify the app runs
3. Read and apply `references/upgrade-v6-to-v7.md`
4. Build the project (`npm run build`), fix any errors, and verify the app runs
5. Read and apply `references/upgrade-v7-to-v8.md`
6. Build the project (`npm run build`), fix any errors, and verify the app runs

Do **not** skip intermediate versions.

### Step 3: Final Verification

After completing all upgrade steps:

```bash
npm run build
```

If the project uses Capacitor, also run:

```bash
npx cap sync
```

## Error Handling

* If `npm run build` fails after an upgrade step, check the compiler errors. Most errors are caused by removed component properties, renamed CSS variables, or changed event handler signatures documented in the reference files.
* If Jest or unit tests fail, check the `transformIgnorePatterns` configuration — Ionic 6+ ships as ES Modules and requires Babel/Jest configuration updates.
* If CSS styles break after upgrade, check for removed CSS Shadow Parts and renamed CSS variables in the reference file for that version.
* If a multi-version upgrade fails mid-way, fix the current version step before proceeding to the next.
* If the project also uses Capacitor and platform builds fail, the Capacitor version may also need upgrading — use the `capacitor-app-upgrades` skill.

## Related Skills

- **`capacitor-app-upgrades`** — If Capacitor also needs upgrading alongside Ionic, use this skill for the Capacitor-specific upgrade steps.
- **`ionic-app-development`** — General Ionic development guidance.
- **`capacitor-plugins`** — For installing or reconfiguring plugins after the app upgrade.
