---
name: capacitor-app-migration-v6
description: Guides the agent through migrating a Capacitor 5 app to Capacitor 6. Covers updating dependencies, Android project variables, Gradle configuration, iOS deployment target, custom plugin registration, androidScheme migration, and official plugin breaking changes. Supports both automated migration via the Capacitor CLI and manual step-by-step migration. Do not use for plugin library migration or non-Capacitor mobile frameworks.
---

# Capacitor App Migration v6

Migrate a Capacitor 5 app project to Capacitor 6.

## Prerequisites

Before proceeding, verify:

1. The project is a **Capacitor 5** app.
2. **Node.js 18+** is installed (required for Capacitor 6).
3. **Xcode 15.0+** is installed (for iOS).
4. **Android Studio Hedgehog | 2023.1.1+** is installed (for Android).

## Procedures

### Step 1: Attempt Automated Migration

```bash
npm i -D @capacitor/cli@latest-6
npx cap migrate
```

If the automated migration completes successfully, skip to **Step 8** to update plugins.
If any steps fail, continue with the manual steps below.

### Step 2: Update Capacitor Dependencies

```bash
npm i @capacitor/core@latest-6
npm i -D @capacitor/cli@latest-6
npm i @capacitor/android@latest-6 @capacitor/ios@latest-6
```

### Step 3: Update Android Project Variables

In `android/variables.gradle`:

```groovy
minSdkVersion = 22
compileSdkVersion = 34
targetSdkVersion = 34
androidxActivityVersion = '1.8.0'
androidxAppCompatVersion = '1.6.1'
androidxCoordinatorLayoutVersion = '1.2.0'
androidxCoreVersion = '1.12.0'
androidxFragmentVersion = '1.6.2'
coreSplashScreenVersion = '1.0.1'
androidxWebkitVersion = '1.9.0'
junitVersion = '4.13.2'
androidxJunitVersion = '1.1.5'
androidxEspressoCoreVersion = '3.5.1'
cordovaAndroidVersion = '10.1.1'
```

### Step 4: Update Android Gradle Configuration

#### 4a: Update Gradle plugin to 8.2.1

In `android/build.gradle`:

```diff
 dependencies {
-    classpath 'com.android.tools.build:gradle:8.0.0'
+    classpath 'com.android.tools.build:gradle:8.2.1'
 }
```

#### 4b: Update Google Services plugin (if used)

```diff
-classpath 'com.google.gms:google-services:4.3.15'
+classpath 'com.google.gms:google-services:4.4.0'
```

#### 4c: Update Gradle wrapper to 8.2.1

In `android/gradle/wrapper/gradle-wrapper.properties`:

```diff
-distributionUrl=https\://services.gradle.org/distributions/gradle-8.0.2-all.zip
+distributionUrl=https\://services.gradle.org/distributions/gradle-8.2.1-all.zip
```

#### 4d: Update Kotlin version (if used)

Update `kotlin_version` to `'1.9.10'`.

### Step 5: Handle androidScheme Migration

In Capacitor 6, `https` is the default `androidScheme`. To avoid data loss from cookies/localStorage:

- If `androidScheme` was **not** previously set, add `androidScheme: "http"` to the Capacitor config.
- If `androidScheme` was already set to `"https"`, it can be safely removed.

```typescript
{
  server: {
    androidScheme: "http"
  }
}
```

### Step 6: Handle iOS Breaking Changes

#### 6a: Register custom plugins

Plugin classes are no longer automatically registered. npm-installed plugins are handled by the CLI. For local custom plugins, create a [custom view controller and register the plugins manually](https://capacitorjs.com/docs/ios/custom-code#register-the-plugin).

#### 6b: Zooming disabled by default

iOS apps are no longer zoomable by default (matching Android). To re-enable, set `zoomEnabled: true` in the Capacitor config.

### Step 7: Update Official Plugins

Update all `@capacitor/*` plugins to v6.

Read `references/plugin-breaking-changes.md` for specific breaking changes per plugin.

### Step 8: Sync and Test

```bash
npx cap sync
npx cap run android
npx cap run ios
```

## Error Handling

* If `npx cap migrate` fails partially, apply the failed steps manually.
* If Android build fails, run **Tools > AGP Upgrade Assistant** in Android Studio.
* If `addListener` calls fail to compile, update the code — `addListener` now only returns a `Promise` (remove `& PluginListenerHandle` if present).
