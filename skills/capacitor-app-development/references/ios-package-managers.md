# iOS Package Managers

Capacitor supports two dependency managers for iOS: Swift Package Manager (SPM) and CocoaPods.

## Swift Package Manager (SPM)

SPM is the default for new Capacitor 8+ projects. It is Apple's official package manager, integrated directly into Xcode.

### How to Detect SPM

- The `ios/App/Podfile` does **not** exist.
- Dependencies are managed in the Xcode project file (`ios/App/App.xcodeproj/project.pbxproj`).
- The `ios/App/App.xcworkspace` may still exist but is not required for SPM-based builds.

### Adding Dependencies with SPM

Capacitor plugins that support SPM are added automatically when running `npx cap sync`. The Capacitor CLI modifies the Xcode project to include SPM package references.

To manually add an SPM package:

1. Open `ios/App/App.xcodeproj` in Xcode.
2. Select the project in the navigator > **Package Dependencies** tab.
3. Click **+** to add a package.
4. Enter the package URL and select a version rule.

### Advantages

- No additional tooling required (built into Xcode).
- Faster dependency resolution than CocoaPods.
- No `Podfile` or `Podfile.lock` to maintain.
- No `pod install` step required.

### Limitations

- Not all Capacitor plugins support SPM yet. Check the plugin's documentation.
- Some third-party native SDKs only distribute via CocoaPods.

## CocoaPods

CocoaPods is the legacy dependency manager. It is the default for Capacitor 7 and earlier projects.

### How to Detect CocoaPods

- The `ios/App/Podfile` exists.
- The `ios/App/Podfile.lock` exists.
- The `ios/App/Pods/` directory exists (after installation).

### Installation

CocoaPods requires Ruby. Install via:

```bash
sudo gem install cocoapods
```

Or via Homebrew:

```bash
brew install cocoapods
```

### Adding Dependencies with CocoaPods

Dependencies are declared in `ios/App/Podfile`. Capacitor plugins are added automatically when running `npx cap sync`.

After any change to `Podfile`:

```bash
cd ios/App && pod install
```

Always open the `.xcworkspace` file (not `.xcodeproj`) when using CocoaPods:

```bash
npx cap open ios
# This opens the .xcworkspace automatically
```

### Common Issues

- **`pod install` fails with version conflicts**: Run `pod repo update` to refresh the local spec repository, then retry.
- **`pod install` hangs**: The CDN source may be slow. Add `source 'https://cdn.cocoapods.org/'` at the top of the `Podfile`.
- **Missing pods after `npx cap sync`**: Run `cd ios/App && pod install` manually.

## Migrating from CocoaPods to SPM

Capacitor 8 supports migrating existing projects from CocoaPods to SPM. This is done during the Capacitor 8 upgrade. For upgrade procedures, use the `capacitor-app-upgrades` skill.

Requirements for migration:
- All Capacitor plugins used in the project must support SPM.
- Third-party native SDKs must be available via SPM.

If any dependency does not support SPM, continue using CocoaPods. Capacitor 8 fully supports both.

## Checking Plugin SPM Support

To verify whether a plugin supports SPM, check for a `Package.swift` file in the plugin's iOS source directory. Plugins distributed via the `@capacitor/*`, `@capawesome/*`, and `@capacitor-firebase/*` namespaces generally support SPM for their latest major versions.

For adding SPM support to custom Capacitor plugins, use the `capacitor-plugin-spm-support` skill.
