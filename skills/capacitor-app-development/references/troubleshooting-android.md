# Android Troubleshooting

Common Android issues and their solutions.

## Build Issues

### Gradle Sync Failed

**Symptom**: Android Studio shows "Gradle sync failed" or `npx cap sync android` fails.

**Solutions**:
1. Ensure the correct JDK is installed and configured:
   - Capacitor 7: JDK 17.
   - Capacitor 8: JDK 21.
   Check: `java -version`. In Android Studio: File > Settings > Build, Execution, Deployment > Build Tools > Gradle > Gradle JDK.
2. Invalidate caches: File > Invalidate Caches in Android Studio.
3. Delete `.gradle` cache: `rm -rf ~/.gradle/caches && rm -rf android/.gradle`.
4. Check that `android/variables.gradle` has the correct SDK and tool versions.

### `compileSdkVersion` or `targetSdkVersion` Errors

**Symptom**: Build fails with messages about SDK version incompatibility.

**Solution**: Update `android/variables.gradle`:

```groovy
ext {
    minSdkVersion = 23          // Capacitor 7; 24 for Capacitor 8
    compileSdkVersion = 35
    targetSdkVersion = 35
    androidxActivityVersion = '1.9.0'
    androidxAppCompatVersion = '1.7.0'
    androidxCoordinatorLayoutVersion = '1.2.0'
    androidxCoreVersion = '1.15.0'
    androidxFragmentVersion = '1.8.1'
    junitVersion = '4.13.2'
    androidxJunitVersion = '1.2.1'
    androidxEspressoCoreVersion = '3.6.1'
    coreSplashScreenVersion = '1.0.1'
}
```

Exact values depend on the Capacitor version. Check the upgrade guide for the target version.

### Gradle Version Incompatibility Error

**Symptom**: `Minimum supported Gradle version is X.X. Current version is Y.Y.`

This occurs when the Android Gradle Plugin (AGP) version is incompatible with the installed Gradle version.

**Solution**: Update AGP in `android/build.gradle` to match the Capacitor version:

```groovy
// Capacitor 6
classpath 'com.android.tools.build:gradle:8.2.1'

// Capacitor 7/8
classpath 'com.android.tools.build:gradle:8.7.3'
```

And update the Gradle wrapper in `android/gradle/wrapper/gradle-wrapper.properties`:

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.11.1-all.zip
```

Use Android Studio's **Tools > AGP Upgrade Assistant** for automated AGP upgrades.

### Gradle Property Syntax Warnings (Capacitor 8+)

**Symptom**: Warnings like "Setting a Gradle property without `=` is deprecated."

**Solution**: Search all `.gradle` files in `android/` for property assignments without `=` and add the `=` sign:

```diff
- ext.androidxCoreVersion '1.15.0'
+ ext.androidxCoreVersion = '1.15.0'
```

### `JAVA_HOME` Not Set

**Symptom**: Build fails with `JAVA_HOME is not set` or `could not find tools.jar`.

**Solution**: Set `JAVA_HOME` to the JDK installation path:

```bash
# macOS (Zulu JDK 21 example)
export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-21.jdk/Contents/Home

# Or use Android Studio's bundled JDK
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
```

Add to `~/.zshrc` or `~/.bashrc` for persistence.

## Runtime Issues

### Blank Screen

A blank screen can have many causes.

#### With Live Reload

1. The development server must bind to all network interfaces, not just `localhost`. Use the `--external` flag:
   ```bash
   ionic cap run android -l --external
   ```
2. Verify the device can access the development server by opening the URL in the device's browser. The device must be on the same network and not blocked by a firewall.
3. Check that Content Security Policy (CSP) or other security mechanisms are not blocking content — open Chrome DevTools via `chrome://inspect` and check the Console.

#### Without Live Reload

1. **Web assets not copied**: Run `npx cap sync android`.
2. **Wrong `webDir`**: Verify `webDir` in `capacitor.config.ts` matches the web build output directory.
3. **Outdated WebView**: Capacitor requires Android System WebView version 60+. Check via device Settings > Developer options > WebView implementation.
4. **Angular browserslist**: If using Angular, ensure `.browserslistrc` or `browserslist` includes the required browser versions. The Angular CLI uses this for code optimization.
5. **JavaScript error**: Connect Chrome DevTools via `chrome://inspect` and check the Console for errors.

### Plugin Not Found at Runtime

**Symptom**: `"PluginName" plugin is not implemented` or `Capacitor plugin "PluginName" not found` in logcat.

**Solutions**:
1. Verify the plugin is installed and listed in `package.json`.
2. Run `npx cap sync android`.
3. In Android Studio: **File > Sync Project with Gradle Files**.
4. Make sure no conflicting plugin is installed (e.g., two different push notification plugins).
5. Check that the plugin's Android implementation exists in `node_modules/<plugin>/android/`.
6. Make sure to use Node.js 22 or higher. Run `node -v` to check.
7. Run `npx cap doctor` to verify Capacitor CLI versions.

### `net::ERR_CLEARTEXT_NOT_PERMITTED`

**Symptom**: Network requests to HTTP URLs fail.

**Solution**: Android 9+ blocks cleartext (HTTP) traffic by default. Options:
1. Use HTTPS for all API endpoints (recommended).
2. If HTTP is required for development, set `server.cleartext: true` in `capacitor.config.ts`.
3. For specific domains, add a network security config in `android/app/src/main/res/xml/network_security_config.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">10.0.2.2</domain>
    </domain-config>
</network-security-config>
```

And reference it in `AndroidManifest.xml`:

```diff
 <application
+    android:networkSecurityConfig="@xml/network_security_config"
     ...>
```

### Camera/Permission Issues

**Symptom**: Camera, geolocation, or other permission-protected features do not work.

**Solutions**:
1. Verify permissions are declared in `android/app/src/main/AndroidManifest.xml`.
2. On Android 13+ (API 33), runtime permissions must be requested explicitly. Call `requestPermissions()` before using the feature.
3. Check if the user denied the permission permanently. If so, guide them to Settings > Apps > [App] > Permissions.

### ProGuard / R8 Stripping Plugin Code

**Symptom**: Plugin works in debug but crashes in release builds.

**Solution**: Add ProGuard keep rules in `android/app/proguard-rules.pro`:

```
-keep class com.getcapacitor.** { *; }
-keep class your.plugin.package.** { *; }
```

## Debugging

### View Logs

```bash
adb logcat | grep -i capacitor
```

Or filter by tag in Android Studio's Logcat panel: `Capacitor`, `Capacitor/Plugin`, or the specific plugin tag.

### Remote WebView Debugging

1. Set `android.webContentsDebuggingEnabled: true` in `capacitor.config.ts`.
2. Run the app on a device or emulator.
3. Open `chrome://inspect` in Chrome.
4. Click **inspect** on the listed WebView.
