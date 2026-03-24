# Android Plugin Implementation

## Architecture

A Capacitor Android plugin consists of a single Java or Kotlin class that extends `com.getcapacitor.Plugin` and uses the `@CapacitorPlugin` annotation. The file lives in `android/src/main/java/<package-path>/`.

For separation of concerns, create a separate implementation class for the platform logic and a plugin class for the bridge.

## Plugin Class

```java
package io.capawesome.capacitorexample;

import com.getcapacitor.JSObject;
import com.getcapacitor.Plugin;
import com.getcapacitor.PluginCall;
import com.getcapacitor.PluginMethod;
import com.getcapacitor.annotation.CapacitorPlugin;

@CapacitorPlugin(name = "Example")
public class ExamplePlugin extends Plugin {

    private Example implementation = new Example();

    @PluginMethod()
    public void echo(PluginCall call) {
        String value = call.getString("value");
        JSObject ret = new JSObject();
        ret.put("value", implementation.echo(value));
        call.resolve(ret);
    }
}
```

The `name` in `@CapacitorPlugin(name = "Example")` must match the first argument of `registerPlugin()` in `src/index.ts`.

## Implementation Class

```java
package io.capawesome.capacitorexample;

public class Example {

    public String echo(String value) {
        System.out.println(value);
        return value;
    }
}
```

## Using Kotlin

The default scaffold generates Java. To convert to Kotlin:

1. Right-click the Java file in Android Studio.
2. Select **Code > Convert Java File to Kotlin File**.
3. Review and fix any conversion issues.

Kotlin example:

```kotlin
package io.capawesome.capacitorexample

import com.getcapacitor.JSObject
import com.getcapacitor.Plugin
import com.getcapacitor.PluginCall
import com.getcapacitor.PluginMethod
import com.getcapacitor.annotation.CapacitorPlugin

@CapacitorPlugin(name = "Example")
class ExamplePlugin : Plugin() {

    private val implementation = Example()

    @PluginMethod
    fun echo(call: PluginCall) {
        val value = call.getString("value") ?: ""
        val ret = JSObject()
        ret.put("value", implementation.echo(value))
        call.resolve(ret)
    }
}
```

## Method Annotations

### `@PluginMethod()`

Default return type is `PluginMethod.RETURN_PROMISE` (resolves with a value).

| Return Type | Annotation | Usage |
| --- | --- | --- |
| Promise (resolves with value) | `@PluginMethod()` | Most common. Method resolves or rejects once. |
| None (void) | `@PluginMethod(returnType = PluginMethod.RETURN_NONE)` | Fire-and-forget. No response to JavaScript. |
| Callback (repeated) | `@PluginMethod(returnType = PluginMethod.RETURN_CALLBACK)` | Method can send data multiple times. |

## Reading Call Data

```java
String name = call.getString("name", "default");
Integer count = call.getInt("count", 0);
Float rate = call.getFloat("rate", 1.0f);
Boolean enabled = call.getBoolean("enabled", false);
JSObject config = call.getObject("config", new JSObject());
JSArray items = call.getArray("items", new JSArray());
```

## Returning Data

### Success

```java
JSObject result = new JSObject();
result.put("value", "Hello");
result.put("count", 42);
JSObject nested = new JSObject();
nested.put("id", "abc123");
result.put("nested", nested);
call.resolve(result);
```

### Void Success

```java
call.resolve();
```

### Error

```java
call.reject("Something went wrong");
```

With an error code:

```java
call.reject("File not found", "FILE_NOT_FOUND");
```

With an exception:

```java
call.reject("Unexpected error", "UNEXPECTED_ERROR", exception);
```

### Platform Not Supported

```java
call.unavailable("This feature requires Android API 26 or later.");
```

### Not Implemented

```java
call.unimplemented("This method is not implemented on Android.");
```

## Events

Emit events to JavaScript listeners:

```java
JSObject data = new JSObject();
data.put("status", "active");
notifyListeners("statusChange", data);
```

The event name must match the `eventName` parameter in the TypeScript `addListener()` method.

## Lifecycle Hooks

Override `load()` for setup when the plugin is first loaded, and `handleOnConfigurationChanged()` for device configuration changes:

```java
@Override
public void load() {
    // Initialize resources, register receivers, etc.
}

@Override
public void handleOnConfigurationChanged(Configuration newConfig) {
    super.handleOnConfigurationChanged(newConfig);
    // React to configuration changes (e.g., orientation)
}
```

## Launching Activities

Launch native screens using Android Intents:

```java
Intent intent = new Intent(getContext(), MyActivity.class);
startActivityForResult(call, intent, "myCallback");
```

Handle the result with `@ActivityCallback`:

```java
@ActivityCallback
private void myCallback(PluginCall call, ActivityResult result) {
    if (call == null) {
        return;
    }
    if (result.getResultCode() == Activity.RESULT_OK) {
        call.resolve();
    } else {
        call.reject("Activity cancelled");
    }
}
```

## Reading Plugin Configuration

Access values from the Capacitor config file:

```java
String style = getConfig().getString("style", "light");
int maxRetries = getConfig().getInt("maxRetries", 3);
```

## Best Practices

- Use `@NonNull` and `@Nullable` annotations to indicate nullability.
- Use primitive types instead of wrapper classes (e.g., `int` instead of `Integer`) when applicable to avoid unnecessary null checks.
- **Nullable vs optional properties in JSObject**: Use `JSONObject.NULL` to represent a nullable property with a null value (included in JSON output as `null`). Simply set to `null` or omit for optional properties (excluded from JSON output entirely).
- Use variables for dependency versions in `build.gradle` so app developers can override them in `variables.gradle`:
  ```groovy
  ext {
      androidxSqliteVersion = project.hasProperty('androidxSqliteVersion') ? rootProject.ext.androidxSqliteVersion : '2.4.0'
  }

  dependencies {
      implementation "androidx.sqlite:sqlite:$androidxSqliteVersion"
  }
  ```
  Document the variable in the plugin's README under an "Android Variables" section.
- Declare required permissions directly in the plugin's `AndroidManifest.xml` at `android/src/main/AndroidManifest.xml`. Optional permissions should be documented in the README for app developers to add manually.

## Permissions

If the plugin accesses protected resources, define permission aliases in the `@CapacitorPlugin` annotation:

```java
@CapacitorPlugin(
    name = "Example",
    permissions = {
        @Permission(
            alias = "camera",
            strings = { Manifest.permission.CAMERA }
        ),
        @Permission(
            alias = "photos",
            strings = {
                Manifest.permission.READ_MEDIA_IMAGES
            }
        )
    }
)
public class ExamplePlugin extends Plugin {

    @PluginMethod()
    public void checkPermissions(PluginCall call) {
        super.checkPermissions(call);
    }

    @PluginMethod()
    public void requestPermissions(PluginCall call) {
        super.requestPermissions(call);
    }

    @PermissionCallback
    private void permissionCallback(PluginCall call) {
        // Handle permission result
        checkPermissions(call);
    }
}
```

### Permission Methods

| Method | Description |
| --- | --- |
| `requestPermissionForAlias(alias, call, callbackName)` | Request a single permission alias. |
| `requestPermissionForAliases(aliases, call, callbackName)` | Request multiple permission aliases. |
| `requestAllPermissions(call, callbackName)` | Request all defined permission aliases. |
| `getPermissionState(alias)` | Get the current state of a permission alias. |

### Install-Time vs Runtime Permissions

- **Install-time permissions** (e.g., `INTERNET`, `ACCESS_NETWORK_STATE`): Add to `AndroidManifest.xml` in the plugin's `android/src/main/` directory.
- **Runtime permissions** (e.g., `CAMERA`, `ACCESS_FINE_LOCATION`): Define in the `@CapacitorPlugin` annotation and request at runtime. Document which permissions the app developer must add to their `AndroidManifest.xml`.
