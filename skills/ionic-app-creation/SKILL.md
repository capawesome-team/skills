---
name: ionic-app-creation
description: "Guides the agent through creating a new Ionic app using the Ionic CLI. Covers project scaffolding with ionic start, framework selection (Angular, React, Vue), template selection (blank, tabs, sidemenu), Capacitor integration, and optional Tailwind CSS setup. After app creation with Capacitor integration, delegates to the capacitor-app-creation skill for further Capacitor setup. Do not use for existing Ionic projects, migrating Ionic apps, upgrading Capacitor versions, or non-Ionic mobile frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/ionic-app-creation
---

# Ionic App Creation

Create a new Ionic app using the Ionic CLI — project scaffolding, framework integration (Angular, React, Vue), Capacitor integration, and Tailwind CSS setup.

## Prerequisites

1. **Node.js** (latest LTS) installed.
2. **Ionic CLI** installed globally (`npm install -g @ionic/cli`). If not installed, install it as part of the procedure.
3. For **iOS development**: Xcode installed.
4. For **Android development**: Android Studio installed.

## Agent Behavior

- **Ask before creating.** Before running `ionic start`, gather all configuration choices from the user. Present available options clearly and wait for confirmation.
- **One decision at a time.** When a step requires user input, ask that single question, wait for the answer, then continue.
- **Present clear options.** Provide concrete choices (e.g., "Which framework: Angular, React, or Vue?") instead of open-ended questions.
- **Guide step-by-step.** Walk the user through the process one step at a time. Never present multiple unrelated questions at once.

## Procedures

### Step 1: Verify the Ionic CLI

Check if the Ionic CLI is installed:

```bash
ionic --version
```

If the command fails or is not found, install it:

```bash
npm install -g @ionic/cli
```

If the user has the legacy `ionic` package installed, uninstall it first:

```bash
npm uninstall -g ionic
npm install -g @ionic/cli
```

### Step 2: Gather Project Configuration

Before creating the app, ask the user the following questions **one at a time**. Present the available options and defaults for each.

#### 2.1: Project Name

Ask the user for the project name. This determines the directory name and the project identifier.

- Example: `my-app`

#### 2.2: Framework

Ask the user which JavaScript framework to use:

| `--type` value        | Framework                          |
| --------------------- | ---------------------------------- |
| `angular`             | Angular (NgModules)                |
| `angular-standalone`  | Angular (Standalone Components)    |
| `react`               | React                              |
| `vue`                 | Vue                                |

Default: `angular`.

#### 2.3: Starter Template

Ask the user which starter template to use:

| Template   | Description                          |
| ---------- | ------------------------------------ |
| `blank`    | An empty project with a single page  |
| `tabs`     | A tabs-based layout                  |
| `sidemenu` | A side menu-based layout             |

The user can also run `ionic start --list` to see all available templates for the selected framework.

Default: `blank`.

#### 2.4: Capacitor Integration

Ask the user if Capacitor should be integrated:

- `--capacitor` — Include Capacitor integration (recommended for native mobile apps).

Default: Yes (include `--capacitor`).

#### 2.5: Package ID

If Capacitor integration was selected, ask the user for the bundle identifier in reverse-DNS notation:

- `--package-id=<id>` — e.g., `com.example.myapp`

This sets the app's bundle ID for both iOS and Android.

#### 2.6: Additional Options

Inform the user about these additional flags and ask if any should be applied:

| Flag               | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `--no-deps`        | Skip installing npm dependencies after project creation      |
| `--no-git`         | Do not initialize a Git repository                           |
| `--project-id=<slug>` | Custom slug for directory name and package name          |

Default: None of these flags are applied.

### Step 3: Create the App

Assemble the `ionic start` command from the collected configuration and run it:

```bash
ionic start <name> <template> --type=<framework> --capacitor --package-id=<package-id>
```

Example:

```bash
ionic start my-app blank --type=angular-standalone --capacitor --package-id=com.example.myapp
```

Wait for the command to complete. The Ionic CLI installs dependencies and scaffolds the project.

### Step 4: Verify the Project

Change into the project directory and verify the project was created correctly:

```bash
cd <name>
ionic serve
```

If `ionic serve` starts the development server without errors, the project was created successfully. Stop the server after verification.

### Step 5: Set Up Tailwind CSS (Optional)

Ask the user if they want to add Tailwind CSS to the project.

If yes, read `references/tailwind-css-setup.md` and apply the Tailwind CSS configuration for the user's chosen framework.

### Step 6: Continue with Capacitor Setup

If Capacitor integration was selected in Step 2.4, inform the user that the basic Ionic app with Capacitor has been created. The `ionic start --capacitor` command already installs `@capacitor/core`, `@capacitor/cli`, and creates `capacitor.config.ts`. Switch to the **`capacitor-app-creation`** skill and continue from **Step 5** (Build the Web App) onward to add native platforms, sync, run the app, and configure optional integrations (live updates, CI/CD).

If Capacitor was **not** selected, the skill is complete.

## Error Handling

- **`ionic: command not found`**: The Ionic CLI is not installed globally. Run `npm install -g @ionic/cli`.
- **Permission errors during global install**: The user may need to configure npm for elevated privilege-free global installs, or use `sudo npm install -g @ionic/cli`.
- **`ionic start` fails with network error**: Check internet connectivity. The CLI downloads starter templates from GitHub.
- **`ionic start` fails with "directory already exists"**: A directory with the chosen project name already exists. Either choose a different name or delete the existing directory.
- **`ionic serve` fails with port conflict**: Another process is using port 8100. Stop the conflicting process or run `ionic serve --port=<other-port>`.
- **Dependency installation fails**: Run `npm install` manually in the project directory. If it still fails, check for Node.js version incompatibilities.
- **Template not found**: Run `ionic start --list` to view all available templates for the selected framework. The user may have misspelled the template name.

## Related Skills

- **`capacitor-app-creation`** — Continue Capacitor setup after Ionic app creation (adding platforms, live updates, CI/CD).
- **`ionic-app-development`** — General Ionic development guidance.
- **`ionic-angular`** — Angular-specific Ionic patterns.
- **`ionic-react`** — React-specific Ionic patterns.
- **`ionic-vue`** — Vue-specific Ionic patterns.
- **`capawesome-cloud`** — Set up live updates, native builds, and app store publishing with Capawesome Cloud.
