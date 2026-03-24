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
- **Always quote the `description` field** in YAML front matter with double quotes to prevent YAML parsing errors (colons and other special characters in unquoted values break parsers).
- Upgrade skills try automated upgrade first (`npx cap migrate`), then provide manual fallback steps.
- Upgrade skills cover both Android and iOS platform specifics.
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

- `capacitor-app-upgrades` — upgrading a Capacitor **app** (covers all versions, routes to per-version reference files)
- `capacitor-plugin-upgrades` — upgrading a Capacitor **plugin** (covers all versions, routes to per-version reference files)
- Other skills are named after the product/feature they cover

## Planning New Skills

- **Keep the number of skills small.** Before creating a new skill, check if an existing skill can be extended to cover the new topic. Prefer combining related functionality into one skill over splitting it across many.
- **Cross-link related skills.** Skills should reference each other where relevant to allow agents to easily navigate between related topics (e.g., a live-updates skill linking to the native-builds skill and vice versa).
- **Check for cross-references after every change.** When adding or modifying skill content, always check whether the new content should link to other skills or whether other skills should link back. Add a `## Related Skills` section at the end of each skill with bullet points referencing related skills.

## Skills Workflow

**Always use the `skill-creator` skill** (via the Skill tool) when creating, modifying, or restructuring any skill. This ensures all skills follow the agentskills.io spec and maintain consistent structure and metadata.

## README.md

When skills are added or removed, update the skills table in `README.md` to reflect the change.
