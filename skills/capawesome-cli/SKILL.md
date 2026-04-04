---
name: capawesome-cli
description: "Guides the agent through installing, authenticating, configuring, and using the Capawesome CLI (@capawesome/cli). Covers installation, interactive and token-based authentication, project linking via capawesome.config.json, the full command reference (app management, native builds, live updates, certificates, environments, channels, deployments, destinations, devices), CI/CD integration with token auth and JSON output, and diagnostics via the doctor command. Do not use for Capawesome Cloud feature setup (native builds workflow, live updates workflow, app store publishing) — use the capawesome-cloud skill instead."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capawesome-cli
---

# Capawesome CLI

Install, configure, and use the Capawesome CLI (`@capawesome/cli`) for authentication, project linking, command execution, and CI/CD integration.

## Prerequisites

1. **Node.js** (v18 or later) and **npm** installed.
2. A [Capawesome Cloud](https://cloud.capawesome.io) account and organization.

## General Rules

Before running any `@capawesome/cli` command for the first time, run it with the `--help` flag to review all available options.

## Procedures

### Step 1: Install the CLI

Install globally:

```bash
npm install -g @capawesome/cli@latest
```

Alternatively, use `npx` to run commands without global installation:

```bash
npx @capawesome/cli <command>
```

Verify the installation:

```bash
npx @capawesome/cli doctor
```

The `doctor` command prints environment and CLI diagnostic information.

### Step 2: Authenticate

#### Interactive Login (Local Development)

```bash
npx @capawesome/cli login
```

This opens a browser-based authentication flow. After completing the flow, the CLI stores the session locally.

#### Token-Based Login (CI/CD)

1. Generate a token in the [Capawesome Cloud Console](https://console.cloud.capawesome.io) under **Settings > Tokens**.
2. Authenticate using the token:

```bash
npx @capawesome/cli login --token <TOKEN>
```

#### Verify Session

```bash
npx @capawesome/cli whoami
```

#### Log Out

```bash
npx @capawesome/cli logout
```

### Step 3: Create or Select an App

Skip if the user already has a Capawesome Cloud app ID.

Create a new app:

```bash
npx @capawesome/cli apps:create --name "My App" --organization-id <ORGANIZATION_ID>
```

The CLI outputs the **app ID** (UUID). Save it for subsequent commands.

### Step 4: Link a Project (Optional)

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

Read `references/project-configuration.md` for all configuration options including monorepo, pnpm, and Yarn setups.

### Step 5: Run CLI Commands

Read `references/commands.md` for the full command reference organized by category:

- **Authentication** — `login`, `logout`, `whoami`
- **App Management** — `apps:create`, `apps:delete`, `apps:transfer`, `apps:link`, `apps:unlink`
- **Build Commands** — `apps:builds:create`, `apps:builds:cancel`, `apps:builds:download`, `apps:builds:logs`
- **Certificate Commands** — `apps:certificates:create`, `apps:certificates:list`, `apps:certificates:get`, `apps:certificates:update`, `apps:certificates:delete`
- **Environment Commands** — `apps:environments:create`, `apps:environments:list`, `apps:environments:set`, `apps:environments:unset`, `apps:environments:delete`
- **Channel Commands** — `apps:channels:create`, `apps:channels:delete`, `apps:channels:get`, `apps:channels:list`, `apps:channels:pause`, `apps:channels:resume`, `apps:channels:update`
- **Live Update Commands** — `apps:liveupdates:create`, `apps:liveupdates:upload`, `apps:liveupdates:register`, `apps:liveupdates:bundle`, `apps:liveupdates:generatemanifest`, `apps:liveupdates:generatesigningkey`, `apps:liveupdates:rollback`, `apps:liveupdates:rollout`, `apps:liveupdates:setnativeversions`
- **Deployment Commands** — `apps:deployments:create`, `apps:deployments:cancel`, `apps:deployments:logs`
- **Destination Commands** — `apps:destinations:create`, `apps:destinations:list`, `apps:destinations:get`, `apps:destinations:update`, `apps:destinations:delete`
- **Device Commands** — `apps:devices:delete`, `apps:devices:forcechannel`, `apps:devices:unforcechannel`, `apps:devices:probe`
- **Organization Commands** — `organizations:create`
- **Utility** — `doctor`

### Step 6: Set Up CI/CD Integration (Optional)

Skip unless the user wants to run CLI commands in a CI/CD pipeline.

Read `references/ci-cd-integration.md` for the full CI/CD setup procedure covering:

- Token-based authentication
- Non-blocking builds with `--detached`
- Machine-readable output with `--json`
- Skipping confirmation prompts with `--yes`
- Example workflows for GitHub Actions and other CI platforms

## Error Handling

- **`command not found: @capawesome/cli`** — The CLI is not installed globally. Either install with `npm install -g @capawesome/cli@latest` or prefix commands with `npx`.
- **Authentication errors / `Not authenticated`** — Re-run `npx @capawesome/cli login`. For CI/CD, verify the token is valid and not expired.
- **`whoami` returns unexpected user** — Log out with `npx @capawesome/cli logout` and log in again with the correct account.
- **Command fails with missing options** — Run the command with `--help` to see all required and optional flags.
- **`doctor` reports issues** — Follow the diagnostic output to resolve environment problems (Node.js version, npm version, CLI version).
- **`capawesome.config.json` not detected** — Ensure the file is in the project root directory (same level as `package.json`). Verify the JSON is valid.

## Related Skills

- **`capawesome-cloud`** — For setting up and using Capawesome Cloud features (native builds, live updates, app store publishing). Uses the CLI as a tool but covers the full workflow.
- **`capacitor-plugins`** — For installing and configuring Capacitor plugins, including the `@capawesome/capacitor-live-update` plugin.
