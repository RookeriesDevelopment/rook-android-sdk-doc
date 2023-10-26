# Changelog

## 0.4.0

The following functions from `RookHealthPermissionsManager` have been converted to static functions:

* checkAvailability(context: Context)
* registerPermissionsRequestLauncher(componentActivity: ComponentActivity)
* registerPermissionsRequestLauncher(fragment: Fragment)
* unregisterPermissionsRequestLauncher()
* launchPermissionsRequest(permissions: HealthPermission)

## 0.3.0

### Android 14

This version will only compile against Android 14 (SDK 34). Please follow the steps to migrate

1. Update your gradle version to 8.1.2
2. Update your `compileSdk` and `targetSdk` to 34
3. Update your `sourceCompatibility`, `targetCompatibility` and `jvmTarget` to Java 17

### Changes

* Permissions functions have changed, see [Check permissions](README.md#check-permissions) for more information.
* Privacy policy configuration has changed, see [Privacy policy](README.md#privacy-policy) for more information.
* Updated data access links, see [Privacy policy](README.md#request-data-access) for more information.
* Added new obfuscation rules, see [Obfuscation](README.md#obfuscation) for more information.

## 0.2.0

* Changed `setLocalLoggingLevel` to `enableLocalLogs`, see [Logging](README.md#logging) for more information.
* Added `RookEnvironment` to configure internal behaviour, see [Environment](README.md#environment) for more
  information.
* Changed `RookConfiguration` constructor parameters, see [Initialize](README.md#initialize) for more information.
* Removed `NotAuthorizedException` each rook SDK will throw a personalized message.

## 0.1.1

* Added Time Zone sync, see [Update userID](README.md#update-userid) for more information.

## 0.1.0

* Initial release.
