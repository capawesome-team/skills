---
name: capacitor-plugin-migration-v8
description: Guides the agent through migrating a Capacitor 7 plugin to Capacitor 8. Covers updating Capacitor dependencies, Android SDK targets, Gradle plugin and wrapper versions, Gradle property syntax, Java version, Kotlin 2.2 migration, iOS deployment target, CocoaPods podspec, and Swift Package Manager configuration. Do not use for app project migration or non-Capacitor plugin frameworks.
---

# Capacitor Plugin Migration v8

Migrate a Capacitor 7 plugin to Capacitor 8.

## Prerequisites

Before proceeding, verify:

1. The project is a **Capacitor 7 plugin**.
2. **Node.js 22+** is installed (required for Capacitor 8).

## Procedures

### Step 1: Attempt Automated Migration

Run the official migration tool from the plugin's root directory:

```bash
npx @capacitor/plugin-migration-v7-to-v8@latest
```

If the automated migration completes successfully, skip to **Step 9**.
If any steps fail, continue with the manual steps below.

### Step 2: Update Capacitor Dependencies

In `package.json`:

- Update `@capacitor/cli`, `@capacitor/core`, `@capacitor/android`, and `@capacitor/ios` in `devDependencies` to `^8.0.0`.
- Update `@capacitor/core` in `peerDependencies` to `>=8.0.0`.

```bash
npm install
```

### Step 3: Update Android SDK Targets

In the plugin's `android/build.gradle`:

```diff
 android {
-    compileSdk project.hasProperty('compileSdkVersion') ? rootProject.ext.compileSdkVersion : 35
+    compileSdk = project.hasProperty('compileSdkVersion') ? rootProject.ext.compileSdkVersion : 36
     defaultConfig {
-        minSdkVersion project.hasProperty('minSdkVersion') ? rootProject.ext.minSdkVersion : 23
+        minSdkVersion = project.hasProperty('minSdkVersion') ? rootProject.ext.minSdkVersion : 24
-        targetSdkVersion project.hasProperty('targetSdkVersion') ? rootProject.ext.targetSdkVersion : 35
+        targetSdkVersion = project.hasProperty('targetSdkVersion') ? rootProject.ext.targetSdkVersion : 36
     }
 }
```

### Step 4: Update Android Plugin Variables

In `android/build.gradle`, update the `ext` block. Only add/update variables the plugin actually uses:

```groovy
ext {
    junitVersion = project.hasProperty('junitVersion') ? rootProject.ext.junitVersion : '4.13.2'
    androidxAppCompatVersion = project.hasProperty('androidxAppCompatVersion') ? rootProject.ext.androidxAppCompatVersion : '1.7.1'
    androidxJunitVersion = project.hasProperty('androidxJunitVersion') ? rootProject.ext.androidxJunitVersion : '1.3.0'
    androidxEspressoCoreVersion = project.hasProperty('androidxEspressoCoreVersion') ? rootProject.ext.androidxEspressoCoreVersion : '3.7.0'
}
```

Read `references/android-plugin-variables.md` for the full list of updated variable versions.

### Step 5: Update Gradle Plugin and Wrapper

#### 5a: Update Gradle plugin to 8.13.0

In `android/build.gradle`:

```diff
 dependencies {
-    classpath 'com.android.tools.build:gradle:8.7.2'
+    classpath 'com.android.tools.build:gradle:8.13.0'
 }
```

#### 5b: Update Gradle wrapper to 8.14.3

In `android/gradle/wrapper/gradle-wrapper.properties`:

```diff
-distributionUrl=https\://services.gradle.org/distributions/gradle-8.11.1-all.zip
+distributionUrl=https\://services.gradle.org/distributions/gradle-8.14.3-all.zip
```

#### 5c: Update Google Services plugin (if used)

```diff
-classpath 'com.google.gms:google-services:4.4.2'
+classpath 'com.google.gms:google-services:4.4.4'
```

### Step 6: Update Gradle Property Syntax

Replace all space-assignment syntax with `=` assignment in all `.gradle` files:

```diff
 repositories {
     maven {
-        url "https://plugins.gradle.org/m2/"
+        url = "https://plugins.gradle.org/m2/"
     }
 }

 android {
-    namespace 'com.example.plugin'
+    namespace = 'com.example.plugin'
 }

 lintOptions {
-    abortOnError false
+    abortOnError = false
 }
```

Method calls (like `mavenCentral()`) remain unchanged — do not add `=` to those.

### Step 7: Update Java and Kotlin Versions

#### 7a: Update to Java 21 (recommended)

In `android/build.gradle`:

```diff
 compileOptions {
-    sourceCompatibility JavaVersion.VERSION_17
+    sourceCompatibility JavaVersion.VERSION_21
-    targetCompatibility JavaVersion.VERSION_17
+    targetCompatibility JavaVersion.VERSION_21
 }
```

#### 7b: Update Kotlin version (if used)

If the plugin uses Kotlin, update the default version to `2.2.20`:

```diff
 buildscript {
-    ext.kotlin_version = project.hasProperty("kotlin_version") ? rootProject.ext.kotlin_version : '1.9.25'
+    ext.kotlin_version = project.hasProperty("kotlin_version") ? rootProject.ext.kotlin_version : '2.2.20'
 }
```

#### 7c: Migrate kotlinOptions to compilerOptions (if using Kotlin)

Read `references/kotlin-migration.md` for the full Kotlin 2.2 migration details.

Replace the deprecated `kotlinOptions` block:

```diff
+import org.jetbrains.kotlin.gradle.dsl.JvmTarget
+
 android {
-    kotlinOptions {
-        jvmTarget = '17'
-    }
 }

+kotlin {
+    compilerOptions {
+        jvmTarget = JvmTarget.JVM_21
+    }
+}
```

### Step 8: Update iOS Configuration

#### 8a: Update CocoaPods podspec

In the plugin's `.podspec` file:

```diff
-s.ios.deployment_target = '14.0'
+s.ios.deployment_target = '15.0'
```

#### 8b: Update Swift Package Manager (if SPM-compatible)

In `Package.swift`:

```diff
-    platforms: [.iOS(.v14)],
+    platforms: [.iOS(.v15)],
```

Update the Capacitor SPM dependency:

```diff
 dependencies: [
-    .package(url: "https://github.com/ionic-team/capacitor-swift-pm.git", from: "7.0.0")
+    .package(url: "https://github.com/ionic-team/capacitor-swift-pm.git", from: "8.0.0")
 ],
```

#### 8c: Update Podfile (if plugin has a test app)

```diff
-platform :ios, '14.0'
+platform :ios, '15.0'
```

#### 8d: Update Xcode project (if plugin uses old structure)

Set **iOS Deployment Target** to **15.0** for both the Project and all Targets in Xcode.

### Step 9: Sync and Test

```bash
npm install
npx cap sync
```

Build the plugin's test/example app on both platforms to verify.

## Error Handling

* If the automated migration tool (`@capacitor/plugin-migration-v7-to-v8`) fails, apply the manual steps above for the failing parts.
* If Android build fails with Gradle property syntax errors, search all `.gradle` files for property assignments without `=` and update them. Remember: method calls do not use `=`.
* If Kotlin compilation fails after updating to 2.2, read `references/kotlin-migration.md` for breaking changes.
* If iOS build fails, verify the deployment target is set to 15.0 in both the podspec and Package.swift (if applicable).
