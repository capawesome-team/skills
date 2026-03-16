# Android Plugin Variables (v8)

The following variables should be updated in the plugin's `android/build.gradle` `ext` block. Only add/update variables the plugin actually uses.

```groovy
ext {
    junitVersion = project.hasProperty('junitVersion') ? rootProject.ext.junitVersion : '4.13.2'
    androidxAppCompatVersion = project.hasProperty('androidxAppCompatVersion') ? rootProject.ext.androidxAppCompatVersion : '1.7.1'
    androidxJunitVersion = project.hasProperty('androidxJunitVersion') ? rootProject.ext.androidxJunitVersion : '1.3.0'
    androidxEspressoCoreVersion = project.hasProperty('androidxEspressoCoreVersion') ? rootProject.ext.androidxEspressoCoreVersion : '3.7.0'
    androidxActivityVersion = project.hasProperty('androidxActivityVersion') ? rootProject.ext.androidxActivityVersion : '1.11.0'
    androidxCoordinatorLayoutVersion = project.hasProperty('androidxCoordinatorLayoutVersion') ? rootProject.ext.androidxCoordinatorLayoutVersion : '1.3.0'
    androidxCoreVersion = project.hasProperty('androidxCoreVersion') ? rootProject.ext.androidxCoreVersion : '1.17.0'
    androidxFragmentVersion = project.hasProperty('androidxFragmentVersion') ? rootProject.ext.androidxFragmentVersion : '1.8.9'
    firebaseMessagingVersion = project.hasProperty('firebaseMessagingVersion') ? rootProject.ext.firebaseMessagingVersion : '25.0.1'
    androidxBrowserVersion = project.hasProperty('androidxBrowserVersion') ? rootProject.ext.androidxBrowserVersion : '1.9.0'
    androidxMaterialVersion = project.hasProperty('androidxMaterialVersion') ? rootProject.ext.androidxMaterialVersion : '1.13.0'
    androidxExifInterfaceVersion = project.hasProperty('androidxExifInterfaceVersion') ? rootProject.ext.androidxExifInterfaceVersion : '1.4.1'
    coreSplashScreenVersion = project.hasProperty('coreSplashScreenVersion') ? rootProject.ext.coreSplashScreenVersion : '1.2.0'
    androidxWebkitVersion = project.hasProperty('androidxWebkitVersion') ? rootProject.ext.androidxWebkitVersion : '1.14.0'
}
```

## Version Changes from v7

| Variable                            | v7 Value  | v8 Value  |
| ----------------------------------- | --------- | --------- |
| `androidxAppCompatVersion`          | `1.7.0`   | `1.7.1`   |
| `androidxJunitVersion`              | `1.2.1`   | `1.3.0`   |
| `androidxEspressoCoreVersion`       | `3.6.1`   | `3.7.0`   |
| `androidxActivityVersion`           | —         | `1.11.0`  |
| `androidxCoordinatorLayoutVersion`  | —         | `1.3.0`   |
| `androidxCoreVersion`               | —         | `1.17.0`  |
| `androidxFragmentVersion`           | —         | `1.8.9`   |
| `firebaseMessagingVersion`          | —         | `25.0.1`  |
| `androidxBrowserVersion`            | —         | `1.9.0`   |
| `androidxMaterialVersion`           | —         | `1.13.0`  |
| `androidxExifInterfaceVersion`      | —         | `1.4.1`   |
| `coreSplashScreenVersion`           | —         | `1.2.0`   |
| `androidxWebkitVersion`             | —         | `1.14.0`  |
