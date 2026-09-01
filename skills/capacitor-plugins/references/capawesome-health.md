# Health

Capacitor plugin to read, write and aggregate health data via Apple HealthKit and Android Health Connect with a single, strictly typed API. Supports individual record reads, calendar-aware aggregation, workouts and writing.

**Package:** `@capawesome-team/capacitor-health`
**Platforms:** Android, iOS
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-health
npx cap sync
```

## Configuration

### Android

#### Minimum SDK Version

Health Connect requires `minSdkVersion` `26`. In `android/variables.gradle`, set:

```groovy
ext {
    minSdkVersion = 26
}
```

#### Permissions

Health Connect permissions must be declared by the app, not the plugin. Declare **only** the permissions for the data types the app actually uses — Google Play reviews every declared health permission.

Add to `android/app/src/main/AndroidManifest.xml` before or after the `application` tag:

```xml
<!-- Add ONLY the permissions for the data types your app actually uses! -->
<uses-permission android:name="android.permission.health.READ_STEPS" />
<uses-permission android:name="android.permission.health.READ_HEART_RATE" />
<uses-permission android:name="android.permission.health.WRITE_WEIGHT" />
```

Read permission names are prefixed with `android.permission.health.`: `READ_ACTIVE_CALORIES_BURNED`, `READ_TOTAL_CALORIES_BURNED`, `READ_BLOOD_GLUCOSE`, `READ_BLOOD_PRESSURE`, `READ_BODY_FAT`, `READ_BODY_TEMPERATURE`, `READ_DISTANCE`, `READ_FLOORS_CLIMBED`, `READ_HEART_RATE`, `READ_HEART_RATE_VARIABILITY`, `READ_HEIGHT`, `READ_HYDRATION`, `READ_OXYGEN_SATURATION`, `READ_RESPIRATORY_RATE`, `READ_RESTING_HEART_RATE`, `READ_SLEEP`, `READ_STEPS`, `READ_VO2_MAX`, `READ_WEIGHT`. Write permissions exist **only** for the writable data types: `WRITE_BLOOD_GLUCOSE`, `WRITE_BLOOD_PRESSURE`, `WRITE_HEIGHT`, `WRITE_HYDRATION`, `WRITE_STEPS`, `WRITE_WEIGHT`. The `WORKOUT` data type uses `android.permission.health.READ_EXERCISE` / `WRITE_EXERCISE`; writing a workout with totals additionally requires `WRITE_DISTANCE` and `WRITE_ACTIVE_CALORIES_BURNED`, and reading workout totals additionally requires `READ_DISTANCE` and `READ_ACTIVE_CALORIES_BURNED`.

#### Privacy Policy

Health Connect requires every app to explain how it uses health data. The plugin already ships the activity that handles the Health Connect privacy policy intents — only the URL must be configured.

Add **inside** the `application` tag in `android/app/src/main/AndroidManifest.xml`:

```xml
<meta-data
    android:name="io.capawesome.capacitorjs.plugins.health.PRIVACY_POLICY_URL"
    android:value="https://example.com/privacy-policy" />
```

#### Variables

Optional overrides in `android/variables.gradle` for dependency conflicts:

- `healthConnectVersion` — `androidx.health.connect:connect-client` (default: `1.1.0`)
- `kotlinVersion` — `org.jetbrains.kotlin:kotlin-gradle-plugin` (default: `2.1.20`)
- `kotlinxCoroutinesVersion` — `org.jetbrains.kotlinx:kotlinx-coroutines-android` (default: `1.10.2`)

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

#### Google Play Health Apps Declaration

Every app integrating with Health Connect must be approved by Google Play or it will be rejected. In the [Google Play Console](https://play.google.com/console/), open **Monitor and improve** → **Policy and programs** → **App content** and complete the **Health apps** declaration (integration, requested permission types, use case) before rolling out a release.

### iOS

#### Capability

The app needs the HealthKit entitlement in `ios/App/App/App.entitlements`:

```xml
<key>com.apple.developer.healthkit</key>
<true/>
```

If the app has no entitlements file yet, add the **HealthKit** capability in Xcode instead (select the `App` target, open the **Signing & Capabilities** tab). Creating the file by hand only takes effect once the project references it through the `CODE_SIGN_ENTITLEMENTS` build setting, which Xcode sets up when the capability is added.

#### Privacy Descriptions

Add to `ios/App/App/Info.plist`:

```xml
<key>NSHealthShareUsageDescription</key>
<string>The app needs access to your health data to show your activity and progress.</string>
<key>NSHealthUpdateUsageDescription</key>
<string>The app writes the health data you log back to Apple Health.</string>
```

`NSHealthShareUsageDescription` is required for reading, `NSHealthUpdateUsageDescription` for writing. If a required key is missing, `requestPermissions(...)` rejects with a clear error instead of crashing the app.

## Usage

### Check availability and install Health Connect

```typescript
import { Health } from '@capawesome-team/capacitor-health';

const { available, reason } = await Health.isAvailable();
if (!available && reason === 'health-connect-not-installed') {
  await Health.installHealthConnect();
}
```

### Request permissions

```typescript
import { DataType, Health } from '@capawesome-team/capacitor-health';

const { permissions } = await Health.requestPermissions({
  read: [DataType.Steps, DataType.HeartRate, DataType.Sleep],
  write: [DataType.Weight],
});
```

### Read records

```typescript
import { DataType, Health } from '@capawesome-team/capacitor-health';

const { records } = await Health.readRecords({
  dataType: DataType.Steps,
  startDate: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString(),
  endDate: new Date().toISOString(),
});
```

### Aggregate data

```typescript
import { DataType, Health } from '@capawesome-team/capacitor-health';

const { buckets } = await Health.aggregate({
  dataType: DataType.Steps,
  startDate: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString(),
  endDate: new Date().toISOString(),
  bucket: 'day',
  operations: ['sum'],
});
```

### Read workouts

```typescript
import { Health } from '@capawesome-team/capacitor-health';

const { workouts } = await Health.readWorkouts({
  startDate: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString(),
  endDate: new Date().toISOString(),
  limit: 10,
});
```

### Write a record

```typescript
import { DataType, Health } from '@capawesome-team/capacitor-health';

await Health.writeRecord({
  dataType: DataType.Weight,
  startDate: new Date().toISOString(),
  value: 71.5,
});
```

## Notes

- No plugin configuration in `capacitor.config.ts` is required.
- All methods reject with an unimplemented error on the web — there is no web API for health data.
- The `DataType` enum covers roughly 20 types across activity (steps, distance, floors climbed, active/total calories), vitals (heart rate, heart rate variability, resting heart rate, blood pressure, blood glucose, oxygen saturation, respiratory rate, body temperature, VO2 max), body measurements (weight, height, body fat), hydration, sleep and workouts. See the `DataType`, `WritableDataType`, `SleepStage` and `WorkoutType` types of the plugin for the full list and the fixed unit of each type.
- Each data type has a fixed unit (e.g. steps `count`, distance `m`, weight `kg`, hydration `L`, calories `kcal`, sleep duration `min`). `BLOOD_PRESSURE` uses `systolic`/`diastolic` instead of `value`.
- Writable data types are limited to `BLOOD_GLUCOSE`, `BLOOD_PRESSURE`, `HEIGHT`, `HYDRATION`, `STEPS`, `WEIGHT` and `WORKOUT`.
- `readRecords(...)` does not support `WORKOUT` (use `readWorkouts(...)`) and does not support `TOTAL_CALORIES` on iOS (use `aggregate(...)`).
- `aggregate(...)` supports only specific data type/operation combinations: `sum` for `ACTIVE_CALORIES`, `DISTANCE`, `FLOORS_CLIMBED`, `HYDRATION`, `STEPS`, `TOTAL_CALORIES`; `average`/`maximum`/`minimum` for `HEART_RATE`, `HEIGHT`, `RESTING_HEART_RATE`, `WEIGHT`. Any other combination rejects with the `INVALID_AGGREGATION` error code.
- Use `aggregate(...)` for totals — the platform deduplicates overlapping data from multiple sources. Summing `readRecords(...)` results manually double-counts data.
- iOS read permissions are reported as `prompt` before the first request and `unknown` afterwards, never `granted` — HealthKit hides read permission status. Design around the presence of data, not the permission status.
- Android read/write permissions are reported as `granted` or `prompt`; a never-requested and a denied permission are indistinguishable. After two denials, Health Connect ignores further requests and permissions can only be granted via `openSettings()`.
- Android reads are limited to the last 30 days before the permission was first granted; iOS has no time limit.
- Platform differences: `SLEEP` returns sessions with all stages on Android but one sample per stage on iOS; `HEART_RATE_VARIABILITY` is RMSSD on Android and SDNN on iOS (not comparable); `sourceName` is always `null` on Android (use `sourceBundleId`); workout totals require iOS 16+.
- `openSettings()` opens the Health Connect settings on Android and the Apple Health app on iOS.
- Apple reviews health apps against App Review Guideline 5.1.3: the app needs a genuine health/fitness feature, must not use health data for advertising or data mining, and must provide a privacy policy.
