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
- 🔄 **Capacitor Migrations**: Migrate Capacitor apps and plugins across major versions (4 → 8).
- 📦 **Plugin Setup**: Install and configure 160+ Capacitor plugins from official and community sources.
- ☁️ **Cloud Integration**: Set up native builds, live updates, and app store publishing with Capawesome Cloud.
- 🚀 **Automated First**: Tries automated migration before falling back to manual steps.
- 🐛 **Error Handling**: Common issues and fixes included in every skill.
- 🔁 **Up-to-date**: Always supports the latest Capacitor version.

Missing a feature? Just [open an issue](https://github.com/capawesome-team/skills/issues/new) and we'll take a look!

## Installation

To install a skill, use the [skills](https://skills.sh/) command:

```bash
npx skills add capawesome-team/skills
```

## Usage

After installing the skills, you can use them in your agent. Here are some examples...

Migrate a Capacitor app across major versions:

```plaintext
Migrate my Capacitor app from version 5 to version 7.
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

Migrate a Capacitor plugin to a newer major version:

```plaintext
Migrate my Capacitor plugin to version 7.
```

Migrate from Ionic Appflow or Ionic Enterprise SDK plugins:

```plaintext
Migrate my app from Ionic Appflow to Capawesome Cloud.
```

## Skills

### Capacitor

| Skill | Description |
| ----- | ----------- |
| [`capacitor-app-migrations`](./skills/capacitor-app-migrations/) | Migrate a Capacitor app to a newer major version. |
| [`capacitor-plugin-migrations`](./skills/capacitor-plugin-migrations/) | Migrate a Capacitor plugin to a newer major version. |
| [`capacitor-plugins`](./skills/capacitor-plugins/) | Install, configure, and use Capacitor plugins from official and community sources. |
| [`capacitor-push-notifications`](./skills/capacitor-push-notifications/) | Set up and use push notifications in Capacitor apps using Firebase Cloud Messaging. |

### Capawesome

| Skill | Description |
| ----- | ----------- |
| [`capawesome-cloud`](./skills/capawesome-cloud/) | Set up and manage native builds, live updates, and app store publishing using Capawesome Cloud. |

### Ionic

| Skill | Description |
| ----- | ----------- |
| [`ionic-appflow-migration`](./skills/ionic-appflow-migration/) | Migrate from Ionic Appflow to Capawesome Cloud. |
| [`ionic-enterprise-sdk-migration`](./skills/ionic-enterprise-sdk-migration/) | Migrate from discontinued Ionic Enterprise SDK plugins to Capawesome alternatives. |

## License

See [LICENSE](https://github.com/capawesome-team/skills/blob/main/LICENSE).
