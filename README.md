# Capawesome Agent Skills

[![Install via skills.sh](https://img.shields.io/badge/skills.sh-install-green)](https://skills.sh/capawesome-team/skills)

Agent skills for interacting with Capawesome, Capacitor and the Ionic Framework.

See the [documentation](https://capawesome.io/docs/ai/skills/) for installation instructions, the full list of skills, and answers to common questions.

<div class="capawesome-z29o10a">
  <a href="https://capawesome.io/" target="_blank">
    <img alt="Deliver Live Updates to your Capacitor app with Capawesome Cloud" src="https://capawesome.io/assets/banners/cloud-build-and-deploy-capacitor-apps.png?t=1" />
  </a>
</div>

## Features

- 🤖 **Agent-optimized**: Every instruction is unambiguous and machine-actionable for AI agents.
- 📱 **Cross-platform**: Covers both Android and iOS platform specifics.
- 🔄 **Capacitor Upgrades**: Upgrade Capacitor apps and plugins across major versions (4 → 8).
- 📦 **Plugin Setup**: Install and configure 160+ Capacitor plugins from official and community sources.
- ☁️ **Cloud Integration**: Set up native builds, live updates, and app store publishing with Capawesome Cloud.
- 🚀 **Automated First**: Tries automated upgrade before falling back to manual steps.
- 🐛 **Error Handling**: Common issues and fixes included in every skill.
- 🔁 **Up-to-date**: Always supports the latest Capacitor version.
- 🔌 **MCP-aware**: Detects the hosted [MCP Servers](#mcp-servers) and prefers them for Capawesome, Capacitor, and Ionic Framework documentation.

Missing a feature? Just [open an issue](https://github.com/capawesome-team/skills/issues/new) and we'll take a look!

## MCP Servers

Capawesome also hosts three **[MCP Servers](https://capawesome.io/docs/ai/mcp/)**. They serve the documentation straight from the source — so it never goes stale — and, in the case of the Capawesome server, the full [Capawesome Cloud](https://capawesome.io/cloud/) management API. There is nothing to install and nothing to keep up to date:

| Server | Endpoint | Serves |
| ------ | -------- | ------ |
| [Capawesome](https://capawesome.io/docs/ai/mcp/capawesome/) | `https://mcp.capawesome.io/mcp` | The Capawesome plugins, the Capawesome CLI, and Capawesome Cloud. |
| [Capacitor](https://capawesome.io/docs/ai/mcp/capacitor/) (unofficial) | `https://capacitor-mcp.capawesome.io/mcp` | The official Capacitor documentation, the official and community plugin list, and the Capacitor posts from the Ionic blog. |
| [Ionic Framework](https://capawesome.io/docs/ai/mcp/ionic-framework/) (unofficial) | `https://ionic-framework-mcp.capawesome.io/mcp` | The official Ionic Framework documentation, the component API reference, and the usage examples per framework. |

Add them to Claude Code:

```bash
claude mcp add --transport http capawesome "https://mcp.capawesome.io/mcp"
claude mcp add --transport http capacitor "https://capacitor-mcp.capawesome.io/mcp"
claude mcp add --transport http ionic-framework "https://ionic-framework-mcp.capawesome.io/mcp"
```

None of the documentation tools need an account or a token. To add the Capawesome Cloud tools, create an [API token](https://console.cloud.capawesome.io/settings/tokens) and use:

```bash
claude mcp add --transport http capawesome "https://mcp.capawesome.io/mcp?toolsets=all" \
  --header "Authorization: Bearer YOUR_TOKEN"
```

For Claude Desktop, Cursor, VS Code, Windsurf, Zed, and other MCP clients, see the [`capawesome-mcp`](./skills/capawesome-mcp/), [`capacitor-mcp`](./skills/capacitor-mcp/), and [`ionic-framework-mcp`](./skills/ionic-framework-mcp/) skills.

### MCP Servers or skills?

Use both — they cover different ground:

| | MCP Servers | Skills |
| --- | :---: | :---: |
| Capawesome plugin, CLI, and Cloud documentation | ✅ always current | ✅ bundled |
| Capacitor Firebase and Capacitor MLKit plugin documentation | ✅ always current | ✅ bundled |
| Official and community Capacitor plugins | ✅ Capacitor MCP Server | ✅ |
| RevenueCat plugins | ❌ | ✅ |
| Ionic Framework component API and usage examples | ✅ Ionic Framework MCP Server | ✅ bundled |
| Capacitor and Ionic upgrade and migration procedures | ✅ reference only | ✅ step by step |
| Ionic Appflow, Ionic Enterprise SDK, and Capgo migrations | ❌ | ✅ |
| Managing Capawesome Cloud apps, builds, and deployments | ✅ | via the CLI |
| Works without a network round trip | ❌ | ✅ |

The skills check for the MCP Servers and prefer them for documentation whenever they are connected, falling back to their bundled reference files when they are not.

## Installation

### skills.sh

To install all skills at once, use the [skills](https://skills.sh/) command:

```bash
npx skills add capawesome-team/skills
```

### Claude Code Plugin Marketplace

Alternatively, if you're using Claude Code, you can add the [Claude Code Plugin Marketplace](https://code.claude.com/docs/en/plugins) and install individual plugins:

```bash
claude plugin marketplace add capawesome-team/skills
```

Available plugins:

| Plugin | Description |
| ------ | ----------- |
| `capacitor-core` | App creation, development, upgrades, expert reference, plugin setup, desktop platforms, and MCP server setup. |
| `capacitor-integrations` | Push notifications (FCM) and in-app purchases. |
| `capacitor-frameworks` | Angular, React, and Vue patterns for Capacitor. |
| `capacitor-plugin-dev` | Create, upgrade, and add SPM support to Capacitor plugins. |
| `ionic-core` | App creation, development, upgrades, expert reference, and MCP server setup. |
| `ionic-frameworks` | Angular, React, and Vue patterns for Ionic. |
| `capawesome-cloud` | MCP server, CLI setup, native builds, live updates, and app store publishing. |
| `ionic-migrations` | Migrate from Ionic Appflow and Ionic Enterprise SDK plugins. |

Install a plugin:

```bash
claude plugin install capacitor-core@capawesome-skills
```

## Update

To regularly update the skills to the latest version, run:

```bash
npx skills update
```

## Usage

After installing the skills, you can use them in your agent. Here are some examples...

Upgrade a Capacitor app across major versions:

```plaintext
Upgrade my Capacitor app from version 5 to version 7.
```

Install and configure any of 160+ supported Capacitor plugins:

```plaintext
Install and configure the @capawesome/capacitor-file-picker plugin in my project.
```

Set up live updates, native builds, or app store publishing with Capawesome Cloud:

```plaintext
Set up Capacitor Live Updates with Capawesome Cloud in my project.
```

```plaintext
Set up native builds with Capawesome Cloud for my Capacitor app.
```

Add push notifications with Firebase Cloud Messaging:

```plaintext
Set up push notifications with Firebase Cloud Messaging in my Capacitor app.
```

Upgrade a Capacitor plugin to a newer major version:

```plaintext
Upgrade my Capacitor plugin to version 7.
```

Migrate from Ionic Appflow, Ionic Enterprise SDK plugins, or Capgo:

```plaintext
Migrate my app from Ionic Appflow to Capawesome Cloud.
```

```plaintext
Migrate my app from Capgo to Capawesome Cloud.
```

## Skills

### Capacitor

| Skill | Description |
| ----- | ----------- |
| [`capacitor-angular`](./skills/capacitor-angular/) | Angular-specific patterns and best practices for Capacitor app development. |
| [`capacitor-app-creation`](./skills/capacitor-app-creation/) | Create a new Capacitor app from scratch with platform setup and optional integrations. |
| [`capacitor-app-development`](./skills/capacitor-app-development/) | General Capacitor app development — core concepts, CLI usage, app configuration, troubleshooting, and best practices. |
| [`capacitor-app-spm-migration`](./skills/capacitor-app-spm-migration/) | Migrate a Capacitor app's iOS dependency management from CocoaPods to Swift Package Manager (SPM). |
| [`capacitor-app-upgrades`](./skills/capacitor-app-upgrades/) | Upgrade a Capacitor app to a newer major version. |
| [`capacitor-expert`](./skills/capacitor-expert/) | Comprehensive Capacitor expert reference — core concepts, CLI, plugins, framework integration, best practices, and Capawesome Cloud. |
| [`capacitor-in-app-purchases`](./skills/capacitor-in-app-purchases/) | Set up in-app purchases and subscriptions in Capacitor apps. |
| [`capacitor-mcp`](./skills/capacitor-mcp/) | Connect an MCP client to the hosted Capacitor MCP Server for the current Capacitor documentation and plugin list. |
| [`capacitor-platforms`](./skills/capacitor-platforms/) | Add and use the Capawesome desktop platforms (Electron and Tauri) in Capacitor apps. |
| [`capacitor-plugin-development`](./skills/capacitor-plugin-development/) | Create and maintain Capacitor plugins from scratch. |
| [`capacitor-plugin-spm-support`](./skills/capacitor-plugin-spm-support/) | Add Swift Package Manager (SPM) support to a Capacitor plugin. |
| [`capacitor-plugin-upgrades`](./skills/capacitor-plugin-upgrades/) | Upgrade a Capacitor plugin to a newer major version. |
| [`capacitor-plugins`](./skills/capacitor-plugins/) | Install, configure, and use Capacitor plugins from official and community sources. |
| [`capacitor-push-notifications`](./skills/capacitor-push-notifications/) | Set up and use push notifications in Capacitor apps using Firebase Cloud Messaging. |
| [`capacitor-react`](./skills/capacitor-react/) | React-specific patterns and best practices for Capacitor app development. |
| [`capacitor-vue`](./skills/capacitor-vue/) | Vue-specific patterns and best practices for Capacitor app development. |

### Capawesome

| Skill | Description |
| ----- | ----------- |
| [`capawesome-cli`](./skills/capawesome-cli/) | Install, configure, and use the Capawesome CLI for authentication, project linking, and CI/CD integration. |
| [`capawesome-cloud`](./skills/capawesome-cloud/) | Set up and manage native builds, live updates, and app store publishing for Capacitor and Cordova apps using Capawesome Cloud. |
| [`capawesome-mcp`](./skills/capawesome-mcp/) | Connect an MCP client to the hosted Capawesome MCP Server for always-current documentation and Capawesome Cloud management. |

### Capgo

| Skill | Description |
| ----- | ----------- |
| [`capgo-cloud-migration`](./skills/capgo-cloud-migration/) | Migrate from Capgo to Capawesome Cloud. |

### Ionic

| Skill | Description |
| ----- | ----------- |
| [`ionic-angular`](./skills/ionic-angular/) | Angular-specific Ionic development patterns — navigation, lifecycle hooks, forms, and standalone vs NgModule architecture. |
| [`ionic-app-creation`](./skills/ionic-app-creation/) | Create a new Ionic app with framework integration, Capacitor setup, and Tailwind CSS. |
| [`ionic-app-development`](./skills/ionic-app-development/) | General Ionic Framework development — core concepts, component reference, CLI usage, layout, theming, and troubleshooting. |
| [`ionic-app-upgrades`](./skills/ionic-app-upgrades/) | Upgrade an Ionic app to a newer major version (4 through 8). |
| [`ionic-expert`](./skills/ionic-expert/) | Comprehensive Ionic Framework expert skill — core concepts, components, theming, lifecycle, navigation, and framework-specific patterns. |
| [`ionic-framework-mcp`](./skills/ionic-framework-mcp/) | Connect an MCP client to the hosted Ionic Framework MCP Server for the current component API reference and usage examples. |
| [`ionic-react`](./skills/ionic-react/) | React-specific Ionic development patterns — components, IonReactRouter, lifecycle hooks, and state management. |
| [`ionic-vue`](./skills/ionic-vue/) | Vue-specific Ionic development patterns — components, navigation, lifecycle hooks, and composables. |

### Ionic Appflow

| Skill | Description |
| ----- | ----------- |
| [`ionic-appflow-migration`](./skills/ionic-appflow-migration/) | Migrate from Ionic Appflow to Capawesome Cloud. |
| [`ionic-enterprise-sdk-migration`](./skills/ionic-enterprise-sdk-migration/) | Migrate from discontinued Ionic Enterprise SDK plugins to Capawesome alternatives. |

## License

See [LICENSE](https://github.com/capawesome-team/skills/blob/main/LICENSE).
