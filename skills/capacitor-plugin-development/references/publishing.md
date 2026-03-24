# Publishing a Capacitor Plugin

## Pre-Publish Checklist

Before publishing, verify the following:

1. **Build succeeds**: Run `npm run build` and confirm no errors.
2. **All platforms verify**: Run `npm run verify` and confirm iOS, Android, and web all pass.
3. **Linting passes**: Run `npm run lint` and fix any issues.
4. **Documentation is generated**: Run `npm run docgen` to update the README API section.
5. **`package.json` is correct**:
   - `name` — the npm package name.
   - `version` — follows [semantic versioning](https://semver.org/).
   - `description` — clear one-line description.
   - `main` — points to `dist/plugin.cjs.js`.
   - `module` — points to `dist/esm/index.js`.
   - `types` — points to `dist/esm/index.d.ts`.
   - `files` — includes `dist/`, `ios/`, `android/`, and the `.podspec`.
   - `capacitor.ios.src` and `capacitor.android.src` — point to the native source directories.
   - `repository`, `author`, `license` — are set correctly.
6. **README** — contains usage instructions, installation steps, and the generated API docs.
7. **CHANGELOG** — documents changes for the version being published.

## `package.json` Capacitor Fields

The `capacitor` field in `package.json` tells the Capacitor CLI where to find native source:

```json
{
  "capacitor": {
    "ios": {
      "src": "ios"
    },
    "android": {
      "src": "android"
    }
  }
}
```

## Publishing to npm

```bash
npm publish
```

For scoped packages that should be public:

```bash
npm publish --access public
```

## Publishing Options

| Option | Description |
| --- | --- |
| **Public npm registry** | Default. Accessible to all. |
| **Private npm registry** | For internal/enterprise plugins. Configure `.npmrc`. |
| **GitHub Packages** | Publish to GitHub's npm registry. |
| **Local linking** | Use `npm install ../path/to/plugin` for internal use without publishing. |

## Version Management

Follow semantic versioning:

- **Patch** (`1.0.1`): Bug fixes, no API changes.
- **Minor** (`1.1.0`): New features, backward-compatible.
- **Major** (`2.0.0`): Breaking changes.

Bump the version before publishing:

```bash
npm version patch   # 1.0.0 -> 1.0.1
npm version minor   # 1.0.0 -> 1.1.0
npm version major   # 1.0.0 -> 2.0.0
```

## Capacitor Community Organization

For plugins that benefit the broader community, consider contributing to the [Capacitor Community](https://github.com/capacitor-community) GitHub organization. This provides:

- Shared maintenance across multiple contributors.
- Discoverability through the organization.
- Standardized tooling and CI pipelines.

## Post-Publish Verification

After publishing, verify the package installs correctly in a fresh Capacitor app:

```bash
npm install <plugin-package-name>
npx cap sync
```
