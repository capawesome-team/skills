# skills

🤖 Agent skills for interacting with Capawesome, Capacitor and the Ionic Framework.

## Installation

To install a skill, use the [skills](https://skills.sh/) command:

```bash
npx skills add capawesome-team/skills
```

## Usage

After installing the skills, you can use them in your agent. For example, to use the `capawesome-cloud` skill, you can prompt your agent like this:

```plaintext
Use the `capawesome-cloud` skill to help me set up Capacitor Live Updates in my project.
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
