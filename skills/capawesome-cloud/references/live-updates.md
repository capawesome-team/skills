# Live Updates

Set up OTA updates for Capacitor and Cordova apps using Capawesome Cloud.

The Live Update SDK is available for both **Capacitor** (`@capawesome/capacitor-live-update`) and **Cordova** (`@capawesome/cordova-live-update`) with full API parity. The setup procedure differs per framework, so follow the framework-specific guide.

## Detect the Framework

Determine which framework the app uses, then read the matching procedure:

1. **Capacitor** — `package.json` lists `@capacitor/core`, and a `capacitor.config.ts`/`capacitor.config.json` exists.
   → Read `live-updates-capacitor.md`.
2. **Cordova** — a `config.xml` exists at the project root with a `<widget>` element, and `package.json` has a `cordova` field (or `cordova-*` platforms).
   → Read `live-updates-cordova.md`.

If both indicators are present (rare), ask the user which framework to target.

## Shared References

These reference files cover both frameworks (each notes Capacitor and Cordova specifics where they differ):

- `live-update-configuration.md` — all configuration options and the Capacitor config ↔ Cordova preference mapping.
- `live-update-plugin-api.md` — the SDK API reference (methods, events, runtime access).
- `update-strategies.md` — Always Latest, Force Update, and Manual Sync strategies.
- `live-update-advanced-topics.md` — channels, versioned channels, rollbacks, code signing, gradual rollouts, self-hosting, delta updates, debugging, and limitations.
- `live-update-faq.md` — compliance, billing, and common questions.
- `live-update-ci-cd-integrations.md` — GitHub Actions, GitLab CI, Azure DevOps, and Bitbucket Pipelines examples.
