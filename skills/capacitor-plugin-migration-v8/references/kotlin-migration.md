# Kotlin 2.2 Migration

Capacitor 8 plugins using Kotlin should update from Kotlin 1.x to `2.2.20`. This is a major version upgrade with breaking changes.

## Key Breaking Changes

### `kotlinOptions{}` block removed

The `kotlinOptions{}` block in Gradle is deprecated and raises an error in Kotlin 2.2. Replace with `compilerOptions{}`:

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

The `kotlin {}` block must be at the top level of `build.gradle`, not inside `android {}`.
Use `JvmTarget.JVM_21` if using Java 21 (recommended), or match the Java version.

### `kotlin-android-extensions` plugin removed

The `kotlin-android-extensions` plugin is no longer available. Replace with:
- `kotlin-parcelize` plugin for `Parcelable` implementation.
- Android Jetpack's view bindings for synthetic view access.

## Full Migration Reference

For a complete list of breaking changes, see the [Kotlin 2.2.0 release notes](https://kotlinlang.org/docs/whatsnew22.html).
