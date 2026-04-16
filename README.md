# skills

Agent skills for interacting with Capawesome, Capacitor and the Ionic Framework.

<div class="capawesome-z29o10a">
  <a href="https://cloud.capawesome.io/" target="_blank">
    <img alt="Deliver Live Updates to your Capacitor app with Capawesome Cloud" src="https://cloud.capawesome.io/assets/banners/cloud-build-and-deploy-capacitor-apps.png?t=1" />
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

Missing a feature? Just [open an issue](https://github.com/capawesome-team/skills/issues/new) and we'll take a look!

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
| `capacitor-core` | App creation, development, upgrades, expert reference, and plugin setup. |
| `capacitor-integrations` | Push notifications (FCM) and in-app purchases. |
| `capacitor-frameworks` | Angular, React, and Vue patterns for Capacitor. |
| `capacitor-plugin-dev` | Create, upgrade, and add SPM support to Capacitor plugins. |
| `ionic-core` | App creation, development, upgrades, and expert reference. |
| `ionic-frameworks` | Angular, React, and Vue patterns for Ionic. |
| `capawesome-cloud` | CLI setup, native builds, live updates, and app store publishing. |
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

Migrate from Ionic Appflow or Ionic Enterprise SDK plugins:

```plaintext
Migrate my app from Ionic Appflow to Capawesome Cloud.
```

## Skills

### Capacitor

| Skill | Description |
| ----- | ----------- |
| [`capacitor-angular`](./skills/capacitor-angular/) | Angular-specific patterns and best practices for Capacitor app development. |
| [`capacitor-app-creation`](./skills/capacitor-app-creation/) | Create a new Capacitor app from scratch with platform setup and optional integrations. |
| [`capacitor-app-development`](./skills/capacitor-app-development/) | General Capacitor app development — core concepts, CLI usage, app configuration, troubleshooting, and best practices. |
| [`capacitor-app-upgrades`](./skills/capacitor-app-upgrades/) | Upgrade a Capacitor app to a newer major version. |
| [`capacitor-expert`](./skills/capacitor-expert/) | Comprehensive Capacitor expert reference — core concepts, CLI, plugins, framework integration, best practices, and Capawesome Cloud. |
| [`capacitor-in-app-purchases`](./skills/capacitor-in-app-purchases/) | Set up in-app purchases and subscriptions in Capacitor apps. |
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
| [`capawesome-cloud`](./skills/capawesome-cloud/) | Set up and manage native builds, live updates, and app store publishing using Capawesome Cloud. |

### Ionic

| Skill | Description |
| ----- | ----------- |
| [`ionic-angular`](./skills/ionic-angular/) | Angular-specific Ionic development patterns — navigation, lifecycle hooks, forms, and standalone vs NgModule architecture. |
| [`ionic-app-creation`](./skills/ionic-app-creation/) | Create a new Ionic app with framework integration, Capacitor setup, and Tailwind CSS. |
| [`ionic-app-development`](./skills/ionic-app-development/) | General Ionic Framework development — core concepts, component reference, CLI usage, layout, theming, and troubleshooting. |
| [`ionic-app-upgrades`](./skills/ionic-app-upgrades/) | Upgrade an Ionic app to a newer major version (4 through 8). |
| [`ionic-expert`](./skills/ionic-expert/) | Comprehensive Ionic Framework expert skill — core concepts, components, theming, lifecycle, navigation, and framework-specific patterns. |
| [`ionic-react`](./skills/ionic-react/) | React-specific Ionic development patterns — components, IonReactRouter, lifecycle hooks, and state management. |
| [`ionic-vue`](./skills/ionic-vue/) | Vue-specific Ionic development patterns — components, navigation, lifecycle hooks, and composables. |

### Ionic Appflow

| Skill | Description |
| ----- | ----------- |
| [`ionic-appflow-migration`](./skills/ionic-appflow-migration/) | Migrate from Ionic Appflow to Capawesome Cloud. |
| [`ionic-enterprise-sdk-migration`](./skills/ionic-enterprise-sdk-migration/) | Migrate from discontinued Ionic Enterprise SDK plugins to Capawesome alternatives. |

## License

See [LICENSE](https://github.com/capawesome-team/skills/blob/main/LICENSE).
