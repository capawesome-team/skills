# iOS Plugin Implementation

## Architecture

A Capacitor iOS plugin consists of two Swift files:

1. **Implementation class** (e.g., `Example.swift`) — Contains the platform logic. Extends `NSObject`.
2. **Plugin class** (e.g., `ExamplePlugin.swift`) — Bridges JavaScript calls to the implementation class. Extends `CAPPlugin` and conforms to `CAPBridgedPlugin`.

Both files live in `ios/Sources/<ClassName>Plugin/` (in generated plugins) or `ios/Plugin/` (in older plugin structures).

## Implementation Class

The implementation class contains pure iOS logic with no Capacitor dependencies:

```swift
import Foundation

@objc public class Example: NSObject {
    @objc public func echo(_ value: String) -> String {
        print(value)
        return value
    }
}
```

Mark methods and the class with `@objc` so they are visible to the Objective-C runtime, which Capacitor uses internally.

## Plugin Class

The plugin class bridges JavaScript calls to the implementation:

```swift
import Foundation
import Capacitor

@objc(ExamplePlugin)
public class ExamplePlugin: CAPPlugin, CAPBridgedPlugin {
    public let identifier = "ExamplePlugin"
    public let jsName = "Example"
    public let pluginMethods: [CAPPluginMethod] = [
        CAPPluginMethod(name: "echo", returnType: CAPPluginReturnPromise)
    ]
    private let implementation = Example()

    @objc func echo(_ call: CAPPluginCall) {
        let value = call.getString("value") ?? ""
        call.resolve([
            "value": implementation.echo(value)
        ])
    }
}
```

### Required Properties (CAPBridgedPlugin)

| Property | Type | Description |
| --- | --- | --- |
| `identifier` | `String` | The plugin class name as a string (e.g., `"ExamplePlugin"`). |
| `jsName` | `String` | The JavaScript name matching the first argument of `registerPlugin()` in `src/index.ts`. |
| `pluginMethods` | `[CAPPluginMethod]` | Array of all methods exposed to JavaScript. |

### Method Return Types

| Return Type | Constant | Usage |
| --- | --- | --- |
| Promise (resolves with value) | `CAPPluginReturnPromise` | Most common. Method resolves or rejects once. |
| None (void) | `CAPPluginReturnNone` | Method performs an action but returns no data. |
| Callback (repeated) | `CAPPluginReturnCallback` | Method can send data multiple times (e.g., geolocation watch). |

## Reading Call Data

Access data passed from JavaScript using `CAPPluginCall` convenience methods:

```swift
let value = call.getString("value") ?? ""
let count = call.getInt("count") ?? 0
let rate = call.getFloat("rate") ?? 1.0
let isEnabled = call.getBool("isEnabled") ?? false
let config = call.getObject("config") ?? [:]
let items = call.getArray("items") ?? []
```

All getter methods return optionals. Always provide a default value or handle `nil`.

## Returning Data

### Success

```swift
call.resolve([
    "value": "Hello",
    "count": 42,
    "nested": [
        "id": "abc123"
    ]
])
```

### Void Success

```swift
call.resolve()
```

### Error

```swift
call.reject("Something went wrong")
```

With an error code and underlying error:

```swift
call.reject("File not found", "FILE_NOT_FOUND", error)
```

### Platform Not Supported

```swift
call.unavailable("This feature requires iOS 16.0 or later.")
```

### Not Implemented

```swift
call.unimplemented("This method is not implemented on iOS.")
```

## Events

Emit events to JavaScript listeners:

```swift
self.notifyListeners("statusChange", data: [
    "status": "active"
])
```

The event name must match the `eventName` parameter in the TypeScript `addListener()` method.

## Lifecycle Hooks

Override `load()` to run setup when the plugin is first loaded:

```swift
override public func load() {
    // Register observers, initialize resources, etc.
    NotificationCenter.default.addObserver(
        self,
        selector: #selector(handleOrientationChange),
        name: UIDevice.orientationDidChangeNotification,
        object: nil
    )
}
```

## Presenting Native UI

Access the app's main view controller to present native screens:

```swift
DispatchQueue.main.async {
    self.bridge?.viewController?.present(viewController, animated: true)
}
```

## Reading Plugin Configuration

Access values from the Capacitor config file (`capacitor.config.ts` or `capacitor.config.json`):

```swift
let style = getConfig().getString("style") ?? "light"
let maxRetries = getConfig().getInt("maxRetries") ?? 3
```

These correspond to values set under `plugins.<PluginJSName>` in the Capacitor config.

## Best Practices

- **Nullable vs optional properties**: Use `NSNull()` to represent a nullable property with a null value (included in JSON output as `null`). Simply set to `nil` or omit for optional properties (excluded from JSON output entirely).
  ```swift
  // Nullable — value is null but present in output
  result["value"] = value == nil ? NSNull() : value
  // Optional — omitted from output if nil
  result["value"] = value
  ```
- If CocoaPods is supported, all new Swift files must be registered in the `project.pbxproj` file of the Xcode project.
- If CocoaPods is supported, set `static_framework = true` in the podspec file so the plugin can be used as a static framework.

## CocoaPods Dependencies

Add third-party iOS dependencies in the `.podspec` file:

```ruby
s.dependency 'SomeLibrary', '~> 2.0'
```

## Swift Package Manager Dependencies

Add SPM dependencies in `Package.swift`. See the `capacitor-plugin-spm-support` skill for details on adding full SPM support.

## Permissions

If the plugin accesses protected resources (camera, location, contacts, etc.):

1. Document which `Info.plist` keys the app must include (e.g., `NSCameraUsageDescription`).
2. Implement `checkPermissions()` and `requestPermissions()` methods.

```swift
@objc func checkPermissions(_ call: CAPPluginCall) {
    let status: String
    switch AVCaptureDevice.authorizationStatus(for: .video) {
    case .authorized:
        status = "granted"
    case .denied:
        status = "denied"
    case .notDetermined:
        status = "prompt"
    default:
        status = "prompt"
    }
    call.resolve(["camera": status])
}

@objc func requestPermissions(_ call: CAPPluginCall) {
    AVCaptureDevice.requestAccess(for: .video) { granted in
        self.checkPermissions(call)
    }
}
```

Permission statuses must use the strings: `granted`, `denied`, `prompt`, or `prompt-with-rationale`.
