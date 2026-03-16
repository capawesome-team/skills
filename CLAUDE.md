# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of **agent skills** for [skills.sh](https://skills.sh/) that help AI assistants guide developers through Capacitor migrations and Capawesome plugin setup. Pure markdown — no build, lint, or test tooling.

## Repository Structure

```
skills/
  <skill-name>/
    SKILL.md            # Main skill file (front matter + procedures)
    references/         # Supporting docs referenced from SKILL.md
    assets/             # Reserved
    scripts/            # Reserved
```

## SKILL.md Format

Each `SKILL.md` has YAML front matter (`name`, `description`) followed by:

1. **Title** (H1) and brief description
2. **Prerequisites** — required tools/versions
3. **Procedures** — numbered step-by-step instructions with code blocks (bash, diff, typescript, groovy, xml)
4. **Error Handling** — common issues and fixes
5. Optional: **Advanced Topics**, **Debugging**, **Limitations**

Key conventions:
- Migration skills try automated migration first (`npx cap migrate`), then provide manual fallback steps.
- Migration skills cover both Android and iOS platform specifics.
- Reference files are cross-referenced from `SKILL.md` via relative paths.
- Skill names use kebab-case.

## Writing Agent-Optimized Skills

Skills are executed by AI agents, not humans. Every instruction must be unambiguous and machine-actionable:

- **Reference exact file paths** — e.g., `ios/App/App.xcodeproj/project.pbxproj`, not "in Xcode".
- **Never use IDE-only instructions** — agents can't click menus. Always provide the file path and the exact text/property to change.
- **Use diff blocks** for all file changes so the agent knows the exact before/after.
- **Specify scope** — e.g., "update **all** occurrences" or "update only the first occurrence".
- **Avoid vague language** — "set the deployment target" is bad. "In `project.pbxproj`, replace all `IPHONEOS_DEPLOYMENT_TARGET = 14.0;` with `IPHONEOS_DEPLOYMENT_TARGET = 15.0;`" is good.

## Naming Conventions

- `capacitor-app-migrations` — migrating a Capacitor **app** (covers all versions, routes to per-version reference files)
- `capacitor-plugin-migrations` — migrating a Capacitor **plugin** (covers all versions, routes to per-version reference files)
- Other skills are named after the product/feature they cover
