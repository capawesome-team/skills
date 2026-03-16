# skills

🤖 Agent skills for interacting with Capawesome, Capacitor and the Ionic Framework.

## Installation

To install a skill, use the [skills](https://skills.sh/) command:

```bash
npx skills add capawesome-team/skills
```

## Usage

After installing the skills, you can use them in your agent. For example, to use the `capacitor-live-update` skill, you can prompt your agent like this:

```plaintext
Use the `capacitor-live-update` skill to help me set up Capacitor Live Updates in my project.
```

## Skills

### Capacitor Migrations

| Skill | Description |
| ----- | ----------- |
| [`capacitor-app-migration`](./skills/capacitor-app-migration/) | Migrate a Capacitor app to a newer major version (4→5, 5→6, 6→7, or 7→8). |
| [`capacitor-plugin-migration`](./skills/capacitor-plugin-migration/) | Migrate a Capacitor plugin to a newer major version (4→5, 5→6, 6→7, or 7→8). |

### Capacitor Plugins

| Skill | Description |
| ----- | ----------- |
| [`capacitor-live-update`](./skills/capacitor-live-update/) | Set up and manage Capacitor Live Updates using Capawesome Cloud. |

## License

See [LICENSE](https://github.com/capawesome-team/skills/blob/main/LICENSE).
