# Rook Steps Tracker

This SDK enables apps to extract steps from phones.

## Features

* Get current day total steps.

## Installation

![Maven Central](https://img.shields.io/maven-central/v/com.rookmotion.android/rook-sdk?style=for-the-badge&logo=gradle&label=maven&color=7200F7)

In your **build.gradle** (app module) add the required dependencies.

```groovy
implementation 'com.rookmotion.android:rook-sdk:version'
```

## Getting started

### Android configuration

In your build.gradle (app) set your min and target sdk version like below:

```groovy
minSdk 26
targetSdk 34
```

#### Obfuscation

If you are using obfuscation consider the following:

In your gradle.properties (Project level) add the following to disable R8 full mode:

```properties
android.enableR8.fullMode=false
```

If you want to enable full mode add the following rules to proguard-rules.pro:

```text
# Keep generic signature of Call, Response (R8 full mode strips signatures from non-kept items).
-keep,allowobfuscation,allowshrinking interface retrofit2.Call
-keep,allowobfuscation,allowshrinking class retrofit2.Response

# With R8 full mode generic signatures are stripped for classes that are not
# kept. Suspend functions are wrapped in continuations where the type argument
# is used.
-keep,allowobfuscation,allowshrinking class kotlin.coroutines.Continuation
```

#### Included permissions

This SDK will add the following permissions to your manifest:

* RECEIVE_BOOT_COMPLETED
* ACTIVITY_RECOGNITION
* POST_NOTIFICATIONS
* FOREGROUND_SERVICE
* FOREGROUND_SERVICE_HEALTH

### Logging

Go to the main [Logging](README.md#logging) section to configure logs.

## Usage

### Initialize

Go to the main [Initialize](README.md#initialize) and [Update userID](README.md#update-userid) sections to initialize.

### Check availability

Before proceeding further, you need to ensure the user's device has the required sensors. Call `isAvailable` providing a
Context:

```kotlin
StepsTracker.isAvailable(context)
```

### Check permissions

To check permissions call `hasPermissions` providing a Context:

```kotlin
StepsTracker.hasPermissions(context)
```

### Request permissions

To request permissions call `requestPermissions` providing a Context:

```kotlin
StepsTracker.requestPermissions(context)
```

### Steps Tracker Configuration

#### Notification

The steps tracker works using a Foreground service which requires a notification to be permanently displayed, although
from Android 13 (SDK 33) this notification can be dismissed without finishing the service associated with it, then the
service will be displayed in the active apps section (This may vary depending on device brand).

The notification has
an [Icon](https://fonts.google.com/icons?selected=Material%20Symbols%20Rounded%3Adirections_walk%3AFILL%400%3Bwght%40400%3BGRAD%400%3Bopsz%4024),
a Title ("Steps service") and Content ("Tracking your steps…") preconfigured

To use your own resources you need to reference them in the **AndroidManifest.xml** file:

```xml

<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application>
        <meta-data
            android:name="io.tryrook.rook.realtime.STEPS_SERVICE_NOTIFICATION_ICON"
            android:resource="@drawable/my_custom_icon"/>

        <meta-data
            android:name="io.tryrook.rook.realtime.STEPS_SERVICE_NOTIFICATION_TITLE"
            android:resource="@string/my_custom_title"/>

        <meta-data
            android:name="io.tryrook.rook.realtime.STEPS_SERVICE_NOTIFICATION_CONTENT"
            android:resource="@string/my_custom_content"/>
    </application>
</manifest>
```

#### Tracker controls

To start tracking steps call `start` providing a Context:

```kotlin
StepsTracker.start(context)
```

To stop tracking steps call `stop` providing a Context:

```kotlin
StepsTracker.stop(context)
```

#### Recommendations

Calling `start`/ `stop` when the service is active/inactive will do nothing, however you can check if the service is
active with `isActive`:

```kotlin
StepsTracker.isActive()
```

#### Get today step count

Call `getTodaySteps` to obtain the last stored step count of the current day:

```kotlin
StepsTracker.getTodaySteps()
```

This value is updated every time the phone detects a step taken and will be reset to zero every day. You can call this
function as many times as you like, although we recommend to put a delay of 1000-3000 milliseconds between each call.

```kotlin
scope.launch {
    while (isActive) {
        val todaySteps = StepsTracker.getTodaySteps()

        // Use todaySteps to update state

        delay(3000) // 1000 to 3000 milliseconds
    }
}
```

#### Observe today step count (Experimental)

Call `observeTodaySteps` to get a **SharedFlow** which receives updates of the current day's total steps. Upon
subscribing, you will get the last emitted value.

```kotlin
scope.launch {
    // This element (class, method or field) is not in stable state yet. It may be renamed, changed or even removed in a future version.
    @OptIn(Experimental::class) 
    StepsTracker.observeTodaySteps().collect {
        // Update your UI
    }
}
```

#### Auto start

After a call to `StepsTracker.start()` if the device is restarted the Foreground service will start after the user
unlocks their device for the first time (This may vary depending on device brand). This behaviour will be stopped when
calling `StepsTracker.stop()`
