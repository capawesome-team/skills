# iOS Troubleshooting

Common iOS issues and their solutions.

## Build Issues

### `pod install` Fails / CocoaPods Incompatible Versions

**Symptom**: `cd ios/App && pod install` fails or `CocoaPods could not find compatible versions for pod "..."`.

**Solutions**:
1. Delete `Podfile.lock` and `Pods` directory, then reinstall with repo update:
   ```bash
   cd ios/App && rm -rf Pods Podfile.lock && pod install --repo-update
   ```
2. Update the CocoaPods spec repo: `pod repo update`.
3. Ensure the correct Ruby and CocoaPods versions are installed: `pod --version`.
4. If using an M1/M2/M3 Mac, run via Rosetta if needed: `arch -x86_64 pod install`.
5. If the problem persists, check that all dependencies in the `Podfile` use compatible versions of the same pod (e.g., all Firebase plugins should use the same Firebase SDK version).

### Signing Errors

**Symptom**: Build fails with "Signing for App requires a development team" or "No signing certificate found."

**Solutions**:
1. Open `ios/App/App.xcodeproj` in Xcode.
2. Select the **App** target > **Signing & Capabilities** tab.
3. Enable **Automatically manage signing**.
4. Select a development team.
5. Verify the Bundle Identifier matches the one registered in the Apple Developer portal.

### Deployment Target Mismatch

**Symptom**: Build fails with "The iOS deployment target is set to X.X, but the range of supported deployment target versions is Y.Y to Z.Z."

**Solution**: Update the deployment target in `ios/App/App.xcodeproj/project.pbxproj`. Find all occurrences of `IPHONEOS_DEPLOYMENT_TARGET` and set to the required minimum:

- Capacitor 7: `15.0`
- Capacitor 8: `16.0`

```diff
- IPHONEOS_DEPLOYMENT_TARGET = 14.0;
+ IPHONEOS_DEPLOYMENT_TARGET = 16.0;
```

Also update in `ios/App/Podfile` if using CocoaPods:

```diff
- platform :ios, '14.0'
+ platform :ios, '16.0'
```

Then run: `cd ios/App && pod install`.

### SPM Package Resolution Fails

**Symptom**: Xcode shows "Package resolution failed" or "Cannot resolve package dependencies."

**Solutions**:
1. In Xcode: File > Packages > Reset Package Caches.
2. In Xcode: File > Packages > Resolve Package Versions.
3. Delete the SPM cache: `rm -rf ~/Library/Caches/org.swift.swiftpm` and `rm -rf ios/App/App.xcodeproj/project.xcworkspace/xcshareddata/swiftpm/`.
4. Close Xcode, run `npx cap sync ios`, then reopen.

### `could not find module 'Capacitor' for target 'x86_64-apple-ios-simulator'`

**Symptom**: Build fails because the `Capacitor` module is not available for the `x86_64` simulator architecture.

**Solution**: Add a `post_install` hook to `ios/App/Podfile`:

```ruby
post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['ONLY_ACTIVE_ARCH'] = 'NO'
            config.build_settings['EXCLUDED_ARCHS[sdk=iphonesimulator*]'] = 'arm64'
        end
    end
end
```

Then run: `cd ios/App && pod install`.

### `value of type 'WKWebView' has no member 'isInspectable'`

**Symptom**: Build fails with this error when using an outdated Xcode version.

**Solution**: Capacitor 6+ requires Xcode 15.0 or later. Update Xcode to the latest version.

If this occurs in GitHub Actions, update the runner:

```diff
- runs-on: macos-latest
+ runs-on: macos-14
```

Note: `macos-latest` is not always the actual latest macOS version.

### Module Not Found

**Symptom**: `No such module 'Capacitor'` or `No such module 'PluginName'`.

**Solutions**:
1. Run `npx cap sync ios`.
2. If using CocoaPods: `cd ios/App && pod install`.
3. In Xcode: Product > Clean Build Folder (Shift+Cmd+K).
4. Close and reopen Xcode.
5. Verify the plugin's iOS implementation exists in `node_modules/<plugin>/ios/`.

## Runtime Issues

### Blank Screen

A blank screen can have many causes.

#### With Live Reload

1. The development server must bind to all network interfaces, not just `localhost`. Use the `--external` flag:
   ```bash
   ionic cap run ios -l --external
   ```
2. Verify the device can access the development server by opening the URL in the device's browser. The device must be on the same network and not blocked by a firewall.
3. Check that Content Security Policy (CSP) or other security mechanisms are not blocking content — open Safari Web Inspector and check the Console.

#### Without Live Reload

1. **Web assets not copied**: Run `npx cap sync ios`.
2. **Wrong `webDir`**: Verify `webDir` in `capacitor.config.ts` matches the web build output directory.
3. **Angular browserslist**: If using Angular, ensure `.browserslistrc` or `browserslist` includes the required browser versions. The Angular CLI uses this for code optimization.
4. **JavaScript error**: Open Safari Web Inspector (Develop > [Device] > [WebView]) and check the Console.

### Plugin Not Found at Runtime

**Symptom**: `"PluginName" plugin is not implemented` or `Capacitor plugin "PluginName" not found`.

**Solutions**:
1. Verify the plugin is installed and listed in `package.json`.
2. Run `npx cap sync ios`.
3. Make sure no conflicting plugin is installed (e.g., two different push notification plugins).
4. Make sure `WKAppBoundDomains` is **not** in `ios/App/App/Info.plist` — this key restricts domain access and breaks plugin injection into the WebView.
5. If using CocoaPods: Verify the plugin pod is in `ios/App/Podfile` and run `cd ios/App && pod install`. Ensure no warnings appear during sync.
6. If using SPM: Verify the package appears in Xcode under **Package Dependencies**.
7. Make sure to use the latest Node.js and npm version. Run `node -v` and `npm -v`.
8. Run `npx cap doctor` to verify Capacitor CLI versions.
9. Make sure only one version of Xcode is installed. Check with `xcode-select -p`.

### App Transport Security (ATS) Blocking HTTP

**Symptom**: Network requests to HTTP URLs fail silently or with ATS errors.

**Solution**: iOS blocks cleartext (HTTP) traffic by default. Options:
1. Use HTTPS for all API endpoints (recommended).
2. For development only, add an ATS exception in `ios/App/App/Info.plist`:

```diff
 <dict>
+    <key>NSAppTransportSecurity</key>
+    <dict>
+        <key>NSAllowsArbitraryLoads</key>
+        <true/>
+    </dict>
 </dict>
```

**Do not ship this in production.** For specific domains, use `NSExceptionDomains` instead.

### Keyboard Pushes Content Up

**Symptom**: The keyboard pushes the WebView content up, causing layout issues.

**Solution**: Configure the `@capacitor/keyboard` plugin in `capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  plugins: {
    Keyboard: {
      resize: 'none',           // 'body', 'ionic', 'native', 'none'
      resizeOnFullScreen: false, // Required when using edge-to-edge plugins
    },
  },
};
```

### Camera/Permission Issues

**Symptom**: Camera, photo library, or other permission-protected features do not work.

**Solutions**:
1. Verify usage descriptions are in `ios/App/App/Info.plist`:
   ```xml
   <key>NSCameraUsageDescription</key>
   <string>This app uses the camera to take photos.</string>
   <key>NSPhotoLibraryUsageDescription</key>
   <string>This app accesses the photo library to select photos.</string>
   ```
2. Each permission requires a human-readable usage description. Without it, the app crashes when requesting the permission.
3. If the user denied the permission, it cannot be re-requested. Guide them to Settings > [App] > Permissions.

### Push Notifications Not Received on Simulator

iOS simulators do not support push notifications. Test on a physical device. For push notification setup, use the `capacitor-push-notifications` skill.

## Debugging

### Safari Web Inspector

1. Set `ios.webContentsDebuggingEnabled: true` in `capacitor.config.ts`.
2. On the iOS device/simulator: Settings > Safari > Advanced > Web Inspector (enable).
3. On macOS: Safari > Settings > Advanced > Show features for web developers.
4. Run the app, then in Safari: Develop > [Device Name] > [WebView].

### Xcode Console

Native logs appear in Xcode's Debug area (View > Debug Area > Activate Console). Filter by `Capacitor` to see bridge-related messages.

### Common Log Messages

- `Loading app at capacitor://localhost` — Normal startup message.
- `Unable to determine the main bundle path` — Web assets not found. Run `npx cap sync`.
- `Plugin "X" is not implemented` — Plugin has no iOS implementation or is not installed.

## Other Issues

### Version 70 Error

**Symptom**: `Unable to find compatibility version string for object version '70'.`

This is caused by Xcode when adding an extension file to the project.

**Solution**: Open `ios/App/App.xcodeproj/project.pbxproj` and change `objectVersion` from `70` to `60`:

```diff
- objectVersion = 70;
+ objectVersion = 60;
```
