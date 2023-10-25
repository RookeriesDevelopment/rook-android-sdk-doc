# Rook SDK

This SDK enables apps to extract and upload data from Health Connect.

* This SDK was developed with our modular SDKs as a way to simplify their implementation:
    * [rook-users](https://github.com/RookeriesDevelopment/rook-android-sdks-docs/tree/main/rook-users)
    * [rook-health-connect](https://github.com/RookeriesDevelopment/rook-android-sdks-docs/tree/main/rook-health-connect)
    * [rook-transmission](https://github.com/RookeriesDevelopment/rook-android-sdks-docs/tree/main/rook-transmission)

## Features

* Get authorization
* Register users
* Extract health data (Health Connect)
* Upload health data (Health Connect)

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

* Health Connect will only work with devices of a minSdk 28 or later. The `minSdk 26` is to keep
  compatibility with other Rook SDKs that can be used with older SDKs.

In your **AndroidManifest.xml**, add a query for Health Connect:

```xml

<manifest>
    <queries>
        <package android:name="com.google.android.apps.healthdata"/>
    </queries>
</manifest>
```

Then declare the health permissions used by this SDK:

```text
<uses-permission android:name="android.permission.health.READ_SLEEP" />
<uses-permission android:name="android.permission.health.READ_STEPS" />
<uses-permission android:name="android.permission.health.READ_DISTANCE" />
<uses-permission android:name="android.permission.health.READ_FLOORS_CLIMBED" />
<uses-permission android:name="android.permission.health.READ_ELEVATION_GAINED" />
<uses-permission android:name="android.permission.health.READ_OXYGEN_SATURATION" />
<uses-permission android:name="android.permission.health.READ_VO2_MAX" />
<uses-permission android:name="android.permission.health.READ_TOTAL_CALORIES_BURNED" />
<uses-permission android:name="android.permission.health.READ_ACTIVE_CALORIES_BURNED" />
<uses-permission android:name="android.permission.health.READ_HEART_RATE" />
<uses-permission android:name="android.permission.health.READ_RESTING_HEART_RATE" />
<uses-permission android:name="android.permission.health.READ_HEART_RATE_VARIABILITY" />
<uses-permission android:name="android.permission.health.READ_EXERCISE" />
<uses-permission android:name="android.permission.health.READ_SPEED" />
<uses-permission android:name="android.permission.health.READ_WEIGHT" />
<uses-permission android:name="android.permission.health.READ_HEIGHT" />
<uses-permission android:name="android.permission.health.READ_BLOOD_GLUCOSE" />
<uses-permission android:name="android.permission.health.READ_BLOOD_PRESSURE" />
<uses-permission android:name="android.permission.health.READ_HYDRATION" />
<uses-permission android:name="android.permission.health.READ_BODY_TEMPERATURE" />
<uses-permission android:name="android.permission.health.READ_RESPIRATORY_RATE" />
<uses-permission android:name="android.permission.health.READ_NUTRITION" />
<uses-permission android:name="android.permission.health.READ_MENSTRUATION" />
<uses-permission android:name="android.permission.health.READ_POWER" />
```

### Privacy policy

Health Connect requires a privacy policy where you inform your users how you will handle and use their data.

To comply with this you'll need an intent filter. You can use a dedicated Activity, or if you are using a **Single
activity architecture**, you can use Deeplinks. You can find an example of the first approach in our demo app.

Inside the Activity that you use to display your app's privacy policy, add an intent filter for the Health Connect
permissions action:

```xml

<application>
  <!-- For supported versions through Android 13, create an activity to show the rationale
       of Health Connect permissions once users click the privacy policy link. -->
  <activity android:name=".features.healthconnect.privacypolicy.HCPrivacyPolicyActivity"
            android:enabled="true"
            android:exported="true">

    <intent-filter>
      <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE"/>
    </intent-filter>
  </activity>

  <!-- For versions starting Android 14, create an activity alias to show the rationale 
       of Health Connect permissions once users click the privacy policy link. -->
  <activity-alias
          android:name="ViewPermissionUsageActivity"
          android:exported="true"
          android:targetActivity=".features.healthconnect.privacypolicy.HCPrivacyPolicyActivity"
          android:permission="android.permission.START_VIEW_PERMISSION_USAGE">

    <intent-filter>
      <action android:name="android.intent.action.VIEW_PERMISSION_USAGE"/>
      <category android:name="android.intent.category.HEALTH_PERMISSIONS"/>
    </intent-filter>
  </activity-alias>
</application>
```

#### Request data access

When you are developing with the Health Connect SDK data access is unrestricted. In order to have data access when your
app is launched to the PlayStore you MUST complete the Developer Declaration Form, more
information [Here](https://developer.android.com/health-and-fitness/guides/health-connect/publish/request-access).

### Environment

The `RookEnvironment` enum allows to quickly configure the behaviour of **rook-sdk**, e.g. the
api used to communicate with ROOK servers.

Available environments:

* SANDBOX ➞ Use this during your app development process.
* PRODUCTION ➞ Use this ONLY when your app is published to the PlayStore.

You can use the `BuildConfig.DEBUG` property of you app to configure the environment:

```kotlin
val environment = if (BuildConfig.DEBUG) RookEnvironment.SANDBOX else RookEnvironment.PRODUCTION
```

### Logging

If you want to see the logs generated by this SDK, before `setConfiguration` call `enableLocalLogs`. You can use
the `BuildConfig.DEBUG` property of you app to configure logging:

```kotlin
if (BuildConfig.DEBUG) {
    rookConfigurationManager.enableLocalLogs()
}
```

## Usage

### Initialize

Create an instance of `RookConfigurationManager` providing a context:

```kotlin
val rookConfigurationManager = RookConfigurationManager(context)
```

Set a configuration and initialize. The `RookConfiguration` requires the following parameters:

* [clientUUID](https://docs.tryrook.io/docs/Definitions#client_uuid)
* [clientPassword](https://docsbeta.tryrook.io/docs/Definitions#client_password)
* [Environment](#environment)

```kotlin
val environment = if (BuildConfig.DEBUG) RookEnvironment.SANDBOX else RookEnvironment.PRODUCTION

val rookConfiguration = RookConfiguration(
  CLIENT_UUID,
  CLIENT_PASSWORD,
  environment,
)

if (BuildConfig.DEBUG) {
  rookConfigurationManager.enableLocalLogs() // MUST be called first if you want to enable native logs
}

rookConfigurationManager.setConfiguration(rookConfiguration)

val result = rookConfigurationManager.initRook()

result.fold(
    {
        // SDK initialized successfully
    },
    {
        // Error initializing SDK

        val error = when (it) {
            is MissingConfigurationException -> "MissingConfigurationException: ${it.message}"
            is TimeoutException -> "TimeoutException: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

#### Recommendations

When you initialize with `initRook` the SDK an HTTP request is made, so you should only initialize the sdk once. We
recommend to code the initialization in the Application's class or treat the `RookConfigurationManager` as a singleton
if you are using dependency injection.

```kotlin
class RookApplication : Application() {
    val rookConfigurationManager: RookConfigurationManager by lazy {
        RookConfigurationManager(context).apply {
            setConfiguration(rookConfiguration)
            initRook()
        }
    }
}
```

### Update userID

Update the [userID](https://docs.tryrook.io/docs/Definitions#user_id):

```kotlin
val result = rookConfigurationManager.updateUserID(userID)

result.fold(
    {
        // userID updated successfully
    },
    {
        // Error updating userID

        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is TimeoutException -> "TimeoutException: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

#### Recommendations

The `updateUserID` should be called as a part of your "Log in" and/or "Is Logged in" flow, when your users log out from
your app call `clearUserID` (this is optional as any call to `updateUserID` will override the previous userID).

```kotlin
rookConfigurationManager.clearUserID()
```

#### User timezone

Every time `updateUserID` completes successfully the timezone information will be updated, you can see the result of the
operation in the logs, when success you will see `Timezone information updated`
otherwise `Failed to update timezone information with error:`

In most cases the above behavior is more than enough. If you need to update the timezone information manually
call `syncUserTimeZone`:

```kotlin
val result = rookConfigurationManager.syncUserTimeZone()

result.fold(
    {
        // User timezone updated successfully
    },
    {
        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
            is TimeoutException -> "TimeoutException: ${it.message}"
            is HttpRequestException -> "HttpRequestException: ${it.message}"
            else -> it.localizedMessage
        }

        // Error updating user timezone
    }
)
```

### Check availability

Before proceeding further, you need to ensure the user's device is compatible with
Health Connect and check if the [APK](https://play.google.com/store/apps/details?id=com.google.android.apps.healthdata)
is installed.

Create a `RookHealthPermissionsManager` instance:

```kotlin
val rookHealthPermissionsManager = RookHealthPermissionsManager(configurationManager)
```

Call `checkAvailability` providing a context and take the corresponding actions:

| Status        | Description                                 | What to do                                      |
|---------------|---------------------------------------------|-------------------------------------------------|
| INSTALLED     | APK is installed                            | Proceed to check permissions                    |
| NOT_INSTALLED | APK is not installed                        | Prompt the user to install Health Connect.      |
| NOT_SUPPORTED | This device does not support Health Connect | Take the user out of the Health Connect section |

```kotlin
val message = when (RookHealthPermissionsManager.checkAvailability(context)) {
    AvailabilityStatus.INSTALLED -> "Health Connect is installed! You can skip the next step"
    AvailabilityStatus.NOT_INSTALLED -> "Health Connect is not installed. Please download from the Play Store"
    else -> "This device is not compatible with health connect. Please close the app"
}
```

### Check permissions

To check permissions call `checkPermissions` and provide a `HealthPermission`, available permissions:

* SLEEP - [Sleep Health](https://docs.tryrook.io/docs/Definitions/#sleep-health-data-pillar) Data Pillar permissions.
* PHYSICAL - [Physical Health](https://docs.tryrook.io/docs/Definitions/#body-health-data-pillar) Data Pillar
  permissions.
* BODY - [Body Health](https://docs.tryrook.io/docs/Definitions/#body-health-data-pillar) Data Pillar permissions.
* ALL - All Health Data Pillar permissions.

```kotlin
val result = rookHealthPermissionsManager.checkPermissions(HealthPermission.ALL)

result.fold(
    {
        val message = if (it) {
            "All permissions are granted! You can skip the next 2 steps"
        } else {
            "There are missing permissions. Please grant them"
        }
    },
    {
        // Error checking all permissions

        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
            is HealthConnectNotInstalledException -> "HealthConnectNotInstalledException: ${it.message}"
            is DeviceNotSupportedException -> "DeviceNotSupportedException: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

### Request permissions

Before requesting permissions you need to register the `RookHealthPermissionsManager` with an activity (ComponentActivity) or
fragment. Call `registerPermissionsRequestLauncher` providing an activity/fragment.

The following block of code MUST be called before your activity/fragment reaches the `resume` state preferably as part
of the `onCreate\onCreateView` function.

```kotlin
RookHealthPermissionsManager.registerPermissionsRequestLauncher(activity / fragment)
```

To request permissions call `launchPermissionsRequest` providing a `HealthPermission`.

```kotlin
RookHealthPermissionsManager.launchPermissionsRequest(HealthPermission.ALL)
```

**IMPORTANT**

Remember to unregister your activity/fragment when you don't need to request permissions anymore. Preferably as part
of the `onDestroy` function.

```kotlin
fun onDestroy() {
  RookHealthPermissionsManager.unregisterPermissionsRequestLauncher()
    super.onDestroy()
}
```

**Permissions denied**

If the user clicks cancel or navigates away from the permissions screen, Health Connect will take it as if the user
denied the permissions.

If the user 'denies' the permissions 2 times, your app will be blocked by Health Connect. This is
permanent and cannot be undone even if the user uninstalls your app.

When your app is blocked, any permissions request will be ignored.

To solve this problem, we recommend you also include an `Open Health Connect` button in your
permissions UI. This button will call `openHealthConnectSettings`, there your users can manually
grant permissions to your app.

```kotlin
val result = rookHealthPermissionsManager.openHealthConnectSettings()

result.fold(
    {
        // Health Connect was opened
    },
    {
        // Error opening Health Connect

        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
            is HealthConnectNotInstalledException -> "HealthConnectNotInstalledException: ${it.message}"
            is DeviceNotSupportedException -> "DeviceNotSupportedException: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

### Sync health data

There are 2 types of health data **Summaries** and **Events**.

| Health Data | Timezone | Oldest date of retrieval | Soonest date of retrieval | Class              |
|-------------|----------|--------------------------|---------------------------|--------------------|
| Summary     | UTC      | 29 days                  | Yesterday                 | RookSummaryManager |
| Event       | UTC      | 29 days                  | Today                     | RookEventManager   |

Create the required instances:

```kotlin
val rookSummaryManager = RookSummaryManager(configurationManager)
val rookEventManager = RookEventManager(configurationManager)
```

#### Sync summaries

To sync any type of summary, you need to provide a date. This date cannot be the current
day and cannot be older than 29 days. See the examples below:

| Current date | Provided date | Is valid?                          |
|--------------|---------------|------------------------------------|
| 2023-01-08   | 2023-01-08    | No, the date is from today         |
| 2023-01-08   | 2023-01-07    | Yes, the date is from yesterday    |
| 2023-01-08   | 2022-11-01    | No, the date is older than 29 days |
| 2023-01-08   | 2023-01-01    | Yes, the date is 7 days old        |

To get health data, call `sync_xxx_summary` and provide a LocalDate instance of the day you want to retrieve the data
from.

For example, if you want to sync yesterday's sleep summary, call `syncSleepSummary`.

```kotlin
val yesterday = LocalDate.now().minusDays(1)
val result = rookSummaryManager.syncSleepSummary(yesterday)

result.fold(
    {
        // Sleep summary synced successfully
    },
    {
        // Error syncing Sleep summary

        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
            is HealthConnectNotInstalledException -> "HealthConnectNotInstalledException: ${it.message}"
            is DeviceNotSupportedException -> "DeviceNotSupportedException: ${it.message}"
            is MissingPermissionsException -> "MissingPermissionsException: ${it.message}"
            is TimeoutException -> "TimeoutException: ${it.message}"
            is HttpRequestException -> "HttpRequestException: code: ${it.code} message: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

#### Sync pending summaries

When you call `sync_xxx_summary`, what happens it's that:

1. Health data is extracted from Health Connect
2. The extracted data is enqueued
3. The enqueued data is uploaded to ROOK servers.
4. If success the queued is cleared. Otherwise, the summary is stored.

To retry sending those stored summaries call `syncPendingSummaries`

```kotlin
val result = rookSummaryManager.syncPendingSummaries()

result.fold(
    {
        // Pending summaries synced successfully
    },
    {
        // Error syncing pending summaries

        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
            is TimeoutException -> "TimeoutException: ${it.message}"
            is HttpRequestException -> "HttpRequestException: code: ${it.code} message: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

#### Recommendations

Summaries are collections of health data from a past day, so you should not sync them more than once. Our recommendation
is that every time your app is opened you should call `should_sync_xxx_summaries_for` to check if you have synced
yesterday's summaries.

If the returned value is true sync summaries, otherwise call `syncPendingSummaries` to retry any failed summary upload.

```kotlin
/**
 * This function assumes that you have already checked availability and permissions.
 */
suspend fun syncSummaries() {
    val yesterday = LocalDate.now().minusDays(1)

    rookSummaryManager.shouldSyncSleepSummariesFor(yesterday).fold(
        { shouldSyncSleepSummariesForYesterday ->

            if (shouldSyncSleepSummariesForYesterday) {
                rookSummaryManager.syncSleepSummary(yesterday).fold(
                    {
                        // Sleep summary synced successfully
                    },
                    {
                        // Error syncing Sleep summary
                    }
                )
            } else {
                rookSummaryManager.syncPendingSummaries().fold(
                    {
                        // Pending summaries synced successfully
                    },
                    {
                        // Error syncing pending summaries
                    }
                )
            }
        },
        {
            // Error

            val error = when (it) {
                is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
                is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
                is HealthConnectNotInstalledException -> "HealthConnectNotInstalledException: ${it.message}"
                is DeviceNotSupportedException -> "DeviceNotSupportedException: ${it.message}"
                else -> it.localizedMessage
            }
        }
    )

    // Check for other types of summaries...
}
```

#### Sync events

To sync any type of event, you need to provide a date. This date cannot be older than 29 days. See the examples
below:

| Current date | Provided date | Is valid?                          |
|--------------|---------------|------------------------------------|
| 2023-01-08   | 2023-01-08    | Yes, the date is from today        |
| 2023-01-08   | 2023-01-07    | Yes, the date is from yesterday    |
| 2023-01-08   | 2022-11-01    | No, the date is older than 29 days |
| 2023-01-08   | 2023-01-01    | Yes, the date is 7 days old        |

To get health data, call `sync_xxx_events` and provide a LocalDate instance of the day you want to retrieve the data
from.

For example, if you want to sync today's physical events, call `syncPhysicalEvents`.

```kotlin
val result = rookEventManager.syncPhysicalEvents(today)

result.fold(
    {
        // Physical events synced successfully
    },
    {
        // Error syncing Physical events

        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
            is HealthConnectNotInstalledException -> "HealthConnectNotInstalledException: ${it.message}"
            is DeviceNotSupportedException -> "DeviceNotSupportedException: ${it.message}"
            is MissingPermissionsException -> "MissingPermissionsException: ${it.message}"
            is TimeoutException -> "TimeoutException: ${it.message}"
            is HttpRequestException -> "HttpRequestException: code: ${it.code} message: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

#### Sync pending events

When you call `sync_xxx_events`, what happens it's that:

1. Health data is extracted from Health Connect
2. The extracted data is enqueued
3. The enqueued data is uploaded to ROOK servers.
4. If success the queued is cleared. Otherwise, the events is stored.

To retry sending those stored events call `syncPendingEvents`

```kotlin
val result = rookEventManager.syncPendingEvents()

result.fold(
    {
        // Pending events synced successfully
    },
    {
        // Error syncing pending events

        val error = when (it) {
            is SDKNotInitializedException -> "SDKNotInitializedException: ${it.message}"
            is UserNotInitializedException -> "UserNotInitializedException: ${it.message}"
            is TimeoutException -> "TimeoutException: ${it.message}"
            is HttpRequestException -> "HttpRequestException: code: ${it.code} message: ${it.message}"
            else -> it.localizedMessage
        }
    }
)
```

#### Recommendations

Events are collections of health data divided in intervals of 1 hour, so you can/should sync them frequently. Our
recommendation is that every time your app is opened you should sync events.

```kotlin
/**
 * This function assumes that you have already checked availability and permissions.
 */
suspend fun syncEvents() {
    val today = LocalDate.now()

    rookEventManager.syncPhysicalEvents(today).fold(
        {
            // Physical events synced successfully
        },
        {
            // Error syncing pending events
        }
    )

    // Sync other types of events...

    // Finally you can call syncPendingEvents to retry all failed uploads 
    // rookEventManager.syncPendingEvents() (optional) 
}
```

* Be cautions of not syncing events an excessive amount of times, Health Connect has a daily usage limit and your app
  could be blocked for some hours or a full day.

## Other resources

* See a complete list of rook-sdk functions in
  the [Javadoc](https://javadoc.io/doc/com.rookmotion.android/rook-sdk/latest/com/rookmotion/rook/sdk/package-summary.html)
