---
name: capacitor-app-migration-v5
description: Guides the agent through migrating a Capacitor 4 app to Capacitor 5. Covers updating dependencies, Android project variables, Gradle configuration, iOS deployment target, Jetifier removal, package namespace migration, and official plugin breaking changes. Supports both automated migration via the Capacitor CLI and manual step-by-step migration. Do not use for plugin library migration or non-Capacitor mobile frameworks.
---

# Capacitor App Migration v5

Migrate a Capacitor 4 app project to Capacitor 5.

## Prerequisites

Before proceeding, verify:

1. The project is a **Capacitor 4** app.
2. **Node.js 16+** is installed (required for Capacitor 5).
3. **Xcode 14.1+** is installed (for iOS).
4. **Android Studio Flamingo | 2022.2.1+** is installed (for Android — ships with Java 17).

## Procedures

### Step 1: Attempt Automated Migration

```bash
npm i -D @capacitor/cli@latest-5
npx cap migrate
```

If the automated migration completes successfully, skip to **Step 9** to update plugins.
If any steps fail, continue with the manual steps below.

### Step 2: Update Capacitor Dependencies

```bash
npm i @capacitor/core@latest-5
npm i -D @capacitor/cli@latest-5
npm i @capacitor/android@latest-5 @capacitor/ios@latest-5
```

### Step 3: Update Android Project Variables

In `android/variables.gradle`:

```groovy
minSdkVersion = 22
compileSdkVersion = 33
targetSdkVersion = 33
androidxActivityVersion = '1.7.0'
androidxAppCompatVersion = '1.6.1'
androidxCoordinatorLayoutVersion = '1.2.0'
androidxCoreVersion = '1.10.0'
androidxFragmentVersion = '1.5.6'
coreSplashScreenVersion = '1.0.0'
androidxWebkitVersion = '1.6.1'
junitVersion = '4.13.2'
androidxJunitVersion = '1.1.5'
androidxEspressoCoreVersion = '3.5.1'
cordovaAndroidVersion = '10.1.1'
```

### Step 4: Update Android Gradle Configuration

#### 4a: Update Gradle plugin to 8.0.0

In `android/build.gradle`:

```diff
 dependencies {
-    classpath 'com.android.tools.build:gradle:7.2.1'
+    classpath 'com.android.tools.build:gradle:8.0.0'
 }
```

#### 4b: Update Google Services plugin (if used)

```diff
-classpath 'com.google.gms:google-services:4.3.13'
+classpath 'com.google.gms:google-services:4.3.15'
```

#### 4c: Update Gradle wrapper to 8.0.2

In `android/gradle/wrapper/gradle-wrapper.properties`:

```diff
-distributionUrl=https\://services.gradle.org/distributions/gradle-7.4.2-all.zip
+distributionUrl=https\://services.gradle.org/distributions/gradle-8.0.2-all.zip
```

#### 4d: Disable Jetifier

In `android/gradle.properties`, remove the Jetifier line if no plugins still use old Android support libraries:

```diff
 android.useAndroidX=true
-# Automatically convert third-party libraries to use AndroidX
-android.enableJetifier=true
```

#### 4e: Move package to build.gradle

Remove `package` from `android/app/src/main/AndroidManifest.xml` and add `namespace` to `android/app/build.gradle`:

```diff
# AndroidManifest.xml
-<manifest xmlns:android="http://schemas.android.com/apk/res/android"
-    package="[YOUR_PACKAGE_ID]">
+<manifest xmlns:android="http://schemas.android.com/apk/res/android">
```

```diff
# app/build.gradle
 android {
+    namespace "[YOUR_PACKAGE_ID]"
     compileSdkVersion rootProject.ext.compileSdkVersion
```

#### 4f: Update Kotlin version (if used)

Update `kotlin_version` to `'1.8.20'`.

### Step 5: Set androidScheme

To prepare for Capacitor 6 where `https` becomes the default, explicitly set the scheme to `http` in the Capacitor config to avoid data loss:

```typescript
{
  server: {
    androidScheme: "http"
  }
}
```

### Step 6: Update iOS Configuration

#### 6a: Update .gitignore

```diff
-App/Podfile.lock
+App/output
```

#### 6b: Update App Icon

Xcode 14 supports a single 1024x1024 app icon. Remove unnecessary sizes from `AppIcon.appiconset`.

### Step 7: Update Official Plugins

Update all `@capacitor/*` plugins to v5.

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
* If Jetifier removal causes build errors, a dependency still uses old support libraries — re-enable Jetifier or update the dependency.
