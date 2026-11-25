# Linkzly Android SDK

[![JitPack](https://jitpack.io/v/Linkzly/linkzly-android-sdk.svg)](https://jitpack.io/#Linkzly/linkzly-android-sdk)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Android](https://img.shields.io/badge/platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-21-orange.svg)](https://developer.android.com/about/versions/lollipop)

The official Android SDK for [Linkzly](https://linkzly.com) - a powerful mobile app attribution and deep linking platform.

**SDK version:** 1.0.11

Full documentation: https://docs.linkzly.com/docs/sdk-android

Guides: [Tracking events](https://docs.linkzly.com/docs/sdk-tracking-events) | [Tracking purchases and refunds](https://docs.linkzly.com/docs/sdk-tracking-purchases) | [Integrations setup](https://docs.linkzly.com/docs/sdk-integrations-setup)

## Using an AI coding agent

If you use an AI coding agent (Claude Code, Codex, Cursor, GitHub Copilot, Antigravity), give it the Linkzly skill and ask it to add the SDK to your app: `npx skills add Linkzly/skills`. The skill carries these instructions, cut to your platform, and rules for the SDK behaviours that are easy to get wrong. It asks you for your SDK key and never makes one up. See [Set up with an AI agent](https://docs.linkzly.com/docs/sdk-ai-agents).

## Features

- **Attribution Tracking** - Install tracking with Google Play Install Referrer API, app open tracking, automatic session management
- **Deep Linking** - Standard deep links (URL schemes, App Links), deferred deep linking, smart link parameter extraction
- **Custom Event Tracking** - Track in-app events with custom parameters, batch tracking, purchase and refund tracking
- **Affiliate Attribution** - Capture affiliate click IDs from deep links, server-to-server attribution, 30-day storage with expiry
- **Push Notifications** - Device push token registration, optional Firebase Cloud Messaging (FCM) broadcast topic
- **Gaming Tracking** - Dedicated gaming event pipeline, player identification, session tracking, gaming-specific attribution
- **Privacy Controls** - User opt-in/opt-out, separate advertising identifier switch
- **Offline Event Queue** - Automatic event queuing when offline, manual flush, pending event inspection
- **Debug Controls** - `LinkzlySDKDebug` batching overrides for debuggable app builds
- **Production Ready** - ProGuard/R8 compatible, lifecycle-aware, coroutine-based async operations

## Requirements

| Component | Value |
|-----------|-------|
| `minSdk` | 21 (5.0 Lollipop), as declared in the SDK's manifest |
| Built and tested with | Android Gradle Plugin 8.2.2, Gradle 8.2, Kotlin 1.9.20, `compileSdk` 34, Java 17 |

The minimum is the value the SDK declares; it is not a tested floor.

**Language Support:** Kotlin (primary, all examples) and Java (fully compatible).

## Prerequisites

Before integrating the SDK, set up your app in the Linkzly Console:

1. Go to **Dashboard > Apps** and click "Register App"
2. Enter your Android **Package Name** and **SHA-256 Fingerprints**
3. Choose a verification method (Hosted recommended for quick start)
4. Copy your **SDK Key** from the post-creation wizard or from Manage app > Overview > SDK Configuration

Your SDK key starts with `slk_` followed by at least 40 hexadecimal characters, and uniquely identifies your app. `configure()` rejects any other format.

## Getting Your SDK Key

Your SDK key (`slk_` prefix) authenticates your app with Linkzly's servers. You can find it in:

- **Post-creation wizard**: Displayed prominently right after creating your app
- **Dashboard > Apps > Manage app > Overview > SDK Configuration**: Click the eye icon to reveal, or copy directly

> **Note**: Each app has a unique SDK key. Do not share keys between different apps.

## Installation

### Step 1: Add JitPack Repository

Add the JitPack repository to your project's `settings.gradle` (or root `build.gradle` for older setups):

**settings.gradle (Kotlin DSL):**
```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

**settings.gradle (Groovy):**
```gradle
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}
```

### Step 2: Add the Dependency

Add the SDK to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.github.Linkzly:linkzly-android-sdk:1.0.11")
}
```

Or in Groovy (`build.gradle`):

```gradle
dependencies {
    implementation 'com.github.Linkzly:linkzly-android-sdk:1.0.11'
}
```

### Step 3: Sync Project

Sync your Gradle files to download the SDK.

## Quick Start

### 1. Configure AndroidManifest.xml

Add your Application class and required intent filters:

```xml
<application
    android:name=".MyApplication"
    ...>

    <activity
        android:name=".MainActivity"
        android:launchMode="singleTask"
        android:exported="true">

        <!-- Launcher intent -->
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>

        <!-- Custom URL Scheme (for testing) -->
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="yourapp" />
        </intent-filter>

        <!-- App Links (for production) -->
        <intent-filter android:autoVerify="true">
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data
                android:scheme="https"
                android:host="yourdomain.com" />
        </intent-filter>
    </activity>
</application>
```

**Required Permissions:**
The SDK automatically includes these permissions (no manual action needed):
- `INTERNET` - For API communication
- `ACCESS_NETWORK_STATE` - For network connectivity checks
- `READ_PHONE_STATE` - Declared in the SDK's manifest

The SDK's dependencies also merge in `com.google.android.gms.permission.AD_ID` and the Play Install Referrer service permission.

### 2. Initialize SDK

In your `Application` class:

```kotlin
import android.app.Application
import com.linkzly.sdk.LinkzlySDK
import com.linkzly.sdk.models.Environment

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        LinkzlySDK.configure(
            context = this,
            sdkKey = "slk_your_key_from_console",  // Get this from Dashboard > Apps > Manage app
            environment = Environment.PRODUCTION
        )
    }
}
```

`Environment` has three values: `PRODUCTION` (the default), `STAGING` and `DEVELOPMENT`. The first `configure()` call wins. Install, open and session tracking start by themselves; there is nothing else to call.

`STAGING` and `DEVELOPMENT` send to separate Linkzly backends, so events sent with them never show in your console. Use `PRODUCTION` with the key from your console.

**Don't forget to register your Application class in AndroidManifest.xml:**
```xml
<application android:name=".MyApplication" ...>
```

### 3. Track Events

**Register the name first.** A custom event is kept only if its name is registered for the app in the console. An unregistered name is dropped, and on the default batched path the call and the flush still report success. See [Register each name in the console](https://docs.linkzly.com/docs/sdk-tracking-events#register-each-name-in-the-console).

```kotlin
LinkzlySDK.trackEvent("signup_completed", mapOf("method" to "email"))
```

### 4. Handle Deep Links

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val deepLinkData = LinkzlySDK.handleAppLink(intent)
        deepLinkData?.let { data ->
            val path = data.path
            val productId = data.getStringParameter("id")
            navigateTo(path, productId)
        }
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        val deepLinkData = LinkzlySDK.handleAppLink(intent)
        deepLinkData?.let { data ->
            navigateTo(data.path, data.getStringParameter("id"))
        }
    }
}
```

### 5. Set User ID

```kotlin
// After login
LinkzlySDK.setUserID("user_12345")

// After logout
LinkzlySDK.setUserID("")
```

## Setup Verification

After integration, verify the SDK is working correctly:

**1. Check SDK Initialization:**
```bash
adb logcat | grep LinkzlySDK
```

Look for:
```
"LinkzlySDK configured successfully"
"Tracking install event" (first launch only)
```

**2. Verify Event Tracking:**

Register `test_event` for the app first (see [Register each name in the console](https://docs.linkzly.com/docs/sdk-tracking-events#register-each-name-in-the-console)). Linkzly drops an event whose name is not registered.

```kotlin
LinkzlySDK.trackEvent("test_event", mapOf("test" to true))
```

Logcat should show:
```
D/LinkzlySDK: Tracking event: test_event with parameters: {test=true}
D/LinkzlySDK: Event tracked successfully
```

This log only shows that the SDK sent the event. It does not show that Linkzly kept it: an unregistered name is dropped.

**3. Test Deep Links with ADB:**
```bash
# For custom URL scheme
adb shell am start -W -a android.intent.action.VIEW \
  -d "yourapp://product?id=123" \
  com.yourcompany.yourapp

# For App Links (HTTPS)
adb shell am start -W -a android.intent.action.VIEW \
  -d "https://yourdomain.com/product?id=123" \
  com.yourcompany.yourapp
```

**Setup Verification Checklist:**
- [ ] SDK added to dependencies and synced
- [ ] Application class created and registered in AndroidManifest
- [ ] SDK initializes on app launch (check Logcat)
- [ ] First install event tracked automatically
- [ ] Custom events tracked successfully, once their names are registered for the app
- [ ] Intent filters added to MainActivity in AndroidManifest
- [ ] Deep links open your app (test with ADB)
- [ ] Deep link data extracted correctly
- [ ] `android:launchMode="singleTask"` set on MainActivity

## API Reference - Core

### Configuration

```kotlin
LinkzlySDK.configure(
    context: Context,
    sdkKey: String,
    environment: Environment = Environment.PRODUCTION
)
```

Call this once in your `Application.onCreate()`. `Environment` has three values: `PRODUCTION`, `STAGING` and `DEVELOPMENT`.

### Deep Linking

```kotlin
// Parse deep link data from an intent
val deepLinkData: DeepLinkData? = LinkzlySDK.handleAppLink(intent)
```

**DeepLinkData** provides type-safe parameter access:

```kotlin
deepLinkData?.url           // Full URL string
deepLinkData?.path          // URL path component
deepLinkData?.smartLinkId   // Linkzly smart link ID
deepLinkData?.clickId       // Attribution click ID

// Type-safe parameter getters
deepLinkData?.getStringParameter("name")    // String?
deepLinkData?.getIntParameter("count")      // Int?
deepLinkData?.getDoubleParameter("price")   // Double?
deepLinkData?.getBooleanParameter("active") // Boolean (defaults to false)
deepLinkData?.getParameter("custom")        // JsonElement?
```

### Attribution

```kotlin
// The SDK already sends the install on the first launch of a new install, so calling
// trackInstall on a new install sends a second install event. On Android this is
// currently the only way to get the deferred deep link data in code: call it once,
// behind your own first-launch flag (the flag stops a third event, not the second).
LinkzlySDK.trackInstall { result ->
    result.onSuccess { deepLinkData ->
        deepLinkData?.let { navigateTo(it.path) }
    }
}

// Track app open: the SDK already sends opens itself, so each explicit call sends one more open event.
// Call it only when you need the DeepLinkData it returns.
LinkzlySDK.trackOpen { result ->
    result.onSuccess { deepLinkData ->
        deepLinkData?.let { handleDeepLink(it) }
    }
}
```

The SDK already sends opens itself (when the app returns to the foreground after more than 30 seconds). Each explicit `trackOpen` call sends one more open event; call it only when you need the `DeepLinkData` it returns. The install is also sent automatically (see the note above on `trackInstall`).

### Event Tracking

**Register the name first.** A custom event is kept only if its name is registered for the app in the console. An unregistered name is dropped, and on the default batched path the call and the flush still report success. See [Register each name in the console](https://docs.linkzly.com/docs/sdk-tracking-events#register-each-name-in-the-console).

```kotlin
// Track a custom event
LinkzlySDK.trackEvent("level_completed", mapOf(
    "level" to 5,
    "score" to 12500
))

// Track a purchase: revenue, currency and transactionId are required.
// transactionId is the deduplication key. revenueType is "iap", "subscription" or "other".
// environment is "production" or "sandbox". The SDK sends only what you pass: without it the
// purchase still counts as revenue, but Linkzly may hold it back from your ad networks.
// See https://docs.linkzly.com/docs/sdk-tracking-purchases#purchase-environment
LinkzlySDK.trackPurchase(
    parameters = mapOf(
        "revenue" to 9.99,
        "currency" to "USD",
        "transactionId" to "order-12345",
        "productId" to "premium_monthly",
        "revenueType" to "subscription",
        "environment" to "production" // "sandbox" in the builds you test purchases with
    ),
    callback = { result ->  // optional
        result.onSuccess { Log.d("Linkzly", "Purchase tracked") }
    }
)

// Track a refund: reuse the ORIGINAL transactionId so it nets out the purchase
LinkzlySDK.trackRefund(
    parameters = mapOf(
        "revenue" to 9.99,
        "currency" to "USD",
        "transactionId" to "order-12345"
    )
)

// Track multiple events in a batch (each map has eventType, eventName and customData keys)
LinkzlySDK.trackEventBatch(
    events = listOf(
        mapOf("eventType" to "custom", "eventName" to "level_started", "customData" to mapOf("level" to 1)),
        mapOf("eventType" to "custom", "eventName" to "level_completed", "customData" to mapOf("level" to 1))
    ),
    callback = { result ->
        result.onSuccess { Log.d("Linkzly", "Batch tracked") }
    }
)
```

See [Tracking events](https://docs.linkzly.com/docs/sdk-tracking-events) and [Tracking purchases and refunds](https://docs.linkzly.com/docs/sdk-tracking-purchases) for the full guides. Purchases are recorded only when Revenue & LTV is enabled for your organisation.

### Sessions

Sessions are automatic. The SDK observes `ProcessLifecycleOwner`: a session starts each time the app comes to the foreground and ends when it goes to the background. There are no session methods to call.

### User Management

```kotlin
// Set user ID (after login)
LinkzlySDK.setUserID("user_12345")

// Get current user ID
val userId: String? = LinkzlySDK.getUserID()

// Clear user ID (after logout)
LinkzlySDK.setUserID("")
```

> **Note:** There is no separate `clearUserID()` method. Pass an empty string to `setUserID("")` to clear.

### Visitor ID

A unique visitor ID is auto-generated on first launch and persists across sessions.

```kotlin
// Get the visitor ID
val visitorId: String = LinkzlySDK.getVisitorID()

// Reset visitor ID (generates a new one)
LinkzlySDK.resetVisitorID()
```

### Event Queue

Events are queued locally and flushed automatically. The queue is saved to SharedPreferences, so it survives an app restart, and holds up to 1,000 events (the oldest is dropped when it is full). For manual control:

```kotlin
// Flush pending events immediately
LinkzlySDK.flushEvents { result ->
    result.onSuccess { Log.d("Linkzly", "Events flushed") }
}

// Get the number of pending events
val count: Int = LinkzlySDK.getPendingEventCount()

// Get the list of pending events. Only works when SDK logging is enabled
// (Environment.STAGING or DEVELOPMENT); with PRODUCTION it logs
// "getPendingEvents() requires loggingEnabled=true" and returns an empty list.
val events: List<QueuedEvent> = LinkzlySDK.getPendingEvents()
```

### Privacy Controls

```kotlin
// Stop custom events, purchases, refunds and trackEventBatch
// (install, open and session-end events are not affected; no effect before configure())
LinkzlySDK.setTrackingEnabled(false)
val isEnabled: Boolean = LinkzlySDK.isTrackingEnabled()

// Disable/enable advertising identifier (GAID) collection
LinkzlySDK.setAdvertisingTrackingEnabled(false)
val isAdEnabled: Boolean = LinkzlySDK.isAdvertisingTrackingEnabled()
```

`setTrackingEnabled(false)` stops custom events, purchases, refunds and `trackEventBatch`. It does not stop the install, open and session-end events the SDK sends itself, and it has no effect before `configure`. Advertising tracking controls only affect GAID collection while allowing other event tracking to continue.

## Affiliate Attribution Tracking

The SDK captures affiliate attribution data from deep links and stores it for server-to-server (S2S) attribution.

### Capturing Attribution

Call `captureAffiliateAttribution` when your app receives a deep link:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    // Capture affiliate data from the incoming URI
    val captured = LinkzlySDK.captureAffiliateAttribution(intent.data)
    if (captured) {
        Log.d("Linkzly", "Affiliate attribution captured")
    }
}

override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    intent.data?.let { LinkzlySDK.captureAffiliateAttribution(it) }
}
```

### Retrieving Attribution

```kotlin
// Check if affiliate attribution exists
if (LinkzlySDK.hasAffiliateAttribution()) {
    // Get the full attribution object
    val attribution: AffiliateAttribution = LinkzlySDK.getAffiliateAttribution()
    // attribution.clickId, attribution.programId, attribution.affiliateId,
    // attribution.timestamp, attribution.hasAttribution, attribution.source

    // Or get just the click ID for S2S reporting
    val clickId: String? = LinkzlySDK.getAffiliateClickId()
}

// Clear stored attribution
LinkzlySDK.clearAffiliateAttribution()
```

**AffiliateAttribution** fields:

| Field | Type | Description |
|-------|------|-------------|
| `clickId` | `String?` | The affiliate click ID |
| `programId` | `String?` | The affiliate program ID |
| `affiliateId` | `String?` | The affiliate partner ID |
| `timestamp` | `Long?` | When the attribution was captured |
| `hasAttribution` | `Boolean` | Whether valid attribution exists |
| `source` | `AffiliateAttributionSource` | `DEEP_LINK`, `STORED`, or `NONE` |

### Server-to-Server (S2S) Example

```kotlin
// After a conversion event (e.g., purchase), send the click ID to your server
val clickId = LinkzlySDK.getAffiliateClickId()
if (clickId != null) {
    yourApi.reportConversion(
        clickId = clickId,
        eventName = "purchase",
        revenue = 29.99
    )
}
```

### Storage and Expiry

- Attribution data is stored in encrypted SharedPreferences
- Default expiry: **30 days** from capture
- Expired attribution is automatically cleared on next access
- Call `clearAffiliateAttribution()` to manually remove stored data

## Push Notification Support

The SDK exposes **two independent push features** — most apps that target individual users want the first one:

| Feature | Methods | What it does | When to use |
|---|---|---|---|
| **Device token registration** | `setNotificationToken` / `getNotificationToken` / `hasNotificationToken` / `clearNotificationToken` | Registers this device's **Firebase Cloud Messaging (FCM) registration token** in Linkzly's device registry so campaigns can target the specific device/user. Devices registered through the SDK are reached through Firebase Cloud Messaging. | You want Linkzly to send (or target) notifications to individual devices/users. |
| **Broadcast topic subscription** | `initializePush` / `disablePush` | Subscribes the device to its **per-(Smart App, platform) FCM topic** for "send to All" campaigns. The topic name comes from a successful `setNotificationToken` registration — `initializePush()` alone is a no-op until then. FCM-only, via runtime reflection. | You need broadcast-to-everyone campaigns and use Firebase Cloud Messaging. |

The two are not mutually exclusive, but they solve different problems. Start with **device token registration** below.

### Registering a device push token

`setNotificationToken` records the device's FCM token in Linkzly's device registry (`/api/sdk/devices/register`). Call it whenever Firebase vends or refreshes a token — the SDK throttles network calls (it only re-registers when the token, user, or app version changes, or after 7 days), so it is safe to call on every launch.

```kotlin
class MyFirebaseMessagingService : FirebaseMessagingService() {
    override fun onNewToken(token: String) {
        LinkzlySDK.setNotificationToken(token)
    }
}
```

You can also push the current token at startup, e.g. from `FirebaseMessaging.getInstance().token`:

```kotlin
FirebaseMessaging.getInstance().token.addOnSuccessListener { token ->
    LinkzlySDK.setNotificationToken(token)
}
```

**Binding to a user:** when you call `LinkzlySDK.setUserID(...)`, the SDK automatically re-registers the stored token against the new user id, so campaigns can target that user.

**On logout / notifications disabled:** clear the token. This removes it locally and revokes it server-side.

```kotlin
LinkzlySDK.clearNotificationToken()
```

**Inspecting state:**

```kotlin
val token: String? = LinkzlySDK.getNotificationToken()
val has: Boolean = LinkzlySDK.hasNotificationToken()
```

**The app passes the token.** The SDK never reads it from Firebase. An empty token is ignored. If the server does not acknowledge a registration, the token is kept and registration is retried on the next trigger or app launch. Before `configure` has run, `setNotificationToken` is ignored: a warning is logged and nothing is stored. An app that delays `configure` calls it again afterwards with the current token.

`clearNotificationToken()` is the durable opt-out: it removes the token locally, revokes it on the server and unsubscribes the broadcast topic. `disablePush()` only unsubscribes the topic.

### Token methods

| Method | Returns | Description |
|--------|---------|-------------|
| `setNotificationToken(token: String)` | `Unit` | Registers the device's FCM token in Linkzly's device registry (throttled) |
| `getNotificationToken()` | `String?` | Returns the currently stored push token, if any |
| `hasNotificationToken()` | `Boolean` | Checks whether a push token is stored |
| `clearNotificationToken()` | `Unit` | Clears the token locally and revokes it server-side |

### Broadcast topic subscription (optional, FCM-only)

Use this only if you want "send to All" broadcast campaigns and your app uses Firebase Cloud Messaging.

#### Prerequisites

- Firebase Cloud Messaging integrated in your app (`com.google.firebase:firebase-messaging`)
- A valid `google-services.json` in your app module
- Linkzly SDK configured and initialized

> **Note:** `initializePush()` and `disablePush()` are **Firebase Cloud Messaging only**. They subscribe/unsubscribe the device to its server-assigned FCM broadcast topic (`linkzly_broadcast_<smartAppId>_android`) using runtime reflection. **Broadcast reach requires a successful `setNotificationToken()` registration first** — the topic name is returned by `/api/sdk/devices/register`. Calling `initializePush()` alone does not receive broadcasts.
>
> If your app also uses another push provider for its own delivery, you do **not** need to call these methods for it. Devices registered through the SDK are reached through Firebase Cloud Messaging.

#### Setup

Most apps do not need this call: after a token registration succeeds the SDK subscribes to the topic by itself, and `initializePush()` is the explicit re-subscribe call. If you do call it, call it once on app startup, after both Firebase and Linkzly SDK are configured:

```kotlin
// In your Application.onCreate() or main Activity
LinkzlySDK.configure(this, "slk_your_sdk_key")

// After Firebase is initialized
val success = LinkzlySDK.initializePush()
if (success) {
    Log.d("MyApp", "Push notifications enabled")
} else {
    Log.w("MyApp", "Firebase Messaging not available")
}
```

#### Disabling Push

```kotlin
LinkzlySDK.disablePush()
```

#### How It Works

- `initializePush()` subscribes the device to a Linkzly broadcast topic via FCM
- No Firebase dependency is added to the Linkzly SDK itself -- it uses runtime reflection
- If Firebase Messaging is not available, the method returns `false` safely (no crash)
- Push campaigns sent from Linkzly Console targeting "All" users are delivered via this topic

#### Troubleshooting

- `initializePush()` returns `false` -- no broadcast topic is stored yet (call `setNotificationToken` first and wait for the registration to succeed), or Firebase Messaging is not in your app
- Notifications not received -- Ensure Firebase is initialized before calling `initializePush()`
- Ensure `google-services.json` is present and valid, and the device has Google Play Services installed

### Push notification opens

Linkzly counts a device as having opened a push campaign when its user taps the notification. The campaign page in the Linkzly console shows the number. This needs Android SDK 1.0.11 or later.

**Automatic.** You write no code. The SDK counts taps on notifications that Android shows itself from an FCM notification message (the system tray, with the app in the background or closed), including the tap that launches the app.

- Only notifications sent by Linkzly count. Their data carries `campaign_id`, or `flow_delivery_id` for triggered pushes.
- Nothing is sent while analytics consent is denied (`setConsent`).
- A tap that arrives before `configure` is held in memory and sent after it.

If a tap can bring a running `singleTop` or `singleTask` activity to the front, Android hands the tap to `onNewIntent`, which the SDK cannot see by itself. Pass it on from there:

```kotlin
override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    LinkzlySDK.handleNotificationIntent(intent)
}
```

**Explicit call.** Call `trackNotificationOpen` from your own tap handling when your app draws the notification itself, and for action buttons. That means any notification your code shows (from your `FirebaseMessagingService`, with `NotificationCompat`, Notifee or a similar library), whether the app is open, in the background or closed, and any action button. Pass the push's data as it arrived and, for a button, its id. A copy cut down to the fields your app routes on can lose `campaign_id`, and then nothing is counted:

```kotlin
// Tap on the notification itself
LinkzlySDK.trackNotificationOpen(remoteMessage.data)

// Tap on an action button
LinkzlySDK.trackNotificationOpen(remoteMessage.data, actionId = "accept")

// Or from the tap's Intent, when your code put the data on it as top-level String extras
LinkzlySDK.trackNotificationOpen(intent)
```

The signatures are `trackNotificationOpen(data: Map<String, Any?>?, actionId: String? = null)` and `trackNotificationOpen(intent: Intent?, actionId: String? = null)`. Only String values are read. The Intent overload reads only top-level extras; when your tap Intent nests the data in a bundle, pass that data as a map instead. A tap that reaches the SDK twice within 5 seconds, once automatically and once through this call, is counted once, so it is safe to call on every tap. Called before `configure`, the call does nothing.

To check a build, use a build that configures Linkzly with the staging or development environment (the detailed notification-open lines below are logged only then; `SDK not configured - trackNotificationOpen ignored` is logged in every environment), run `adb logcat -s LinkzlySDK`, and tap a Linkzly push once with the app closed and once with it in the background. `Push open reported` means the server accepted the report; confirm the count on the campaign page in the console for that same environment. `Notification data has no campaign_id or flow_delivery_id` means the data passed to `trackNotificationOpen` was not the push's own. `Analytics consent is denied` and `Notification open already reported for this tap` explain a report that was skipped on purpose. `SDK not configured - trackNotificationOpen ignored` means the call came before `configure`. No line at all is inconclusive: the call may not have run, or it was given something that holds no data the SDK can read.

**Turning automatic capture off.** Add this to the `<application>` element of your `AndroidManifest.xml`. The explicit call still works.

```xml
<meta-data
    android:name="com.linkzly.sdk.auto_track_notification_opens"
    android:value="false" />
```

## Gaming Tracking Module

The gaming tracking module provides a dedicated event pipeline optimized for game telemetry with automatic batching, retry logic, and offline support.

### Configuration

```kotlin
// Option 1: Using GamingOptions (recommended)
LinkzlySDK.configureGamingTracking(
    context = this,
    options = LinkzlyGamingTracking.GamingOptions(
        apiKey = "your-gaming-api-key",
        organizationId = "your-org-id",
        gameId = "your-game-id",
        gameVersion = "1.2.0",              // optional, default ""
        debug = false,                       // optional, default false
        maxBatchSize = 100,                  // optional, default 100
        flushIntervalMs = 5_000,             // optional, default 5000
        maxRetries = 3,                      // optional, default 3
        maxQueueSize = 10_000,               // optional, default 10000
        sessionTimeoutMs = 30 * 60 * 1000,   // optional, default 30 min
        autoSessionTracking = true,          // optional, default true
        signingSecret = null                 // optional, HMAC signing
    )
)

// Option 2: Shorthand
LinkzlySDK.configureGamingTracking(
    context = this,
    apiKey = "your-gaming-api-key",
    organizationId = "your-org-id",
    gameId = "your-game-id",
    environment = Environment.PRODUCTION     // optional, default PRODUCTION
)
```

### Player Identification

```kotlin
// Identify the current player
LinkzlySDK.identifyGamingPlayer(
    playerId = "player_abc123",
    traits = mapOf(                  // optional
        "level" to 42,
        "vip_tier" to "gold"
    )
)
```

### Tracking Gaming Events

```kotlin
// Track a gaming event (batched automatically)
LinkzlySDK.trackGamingEvent(
    eventType = "level_complete",
    data = mapOf(
        "level" to 15,
        "score" to 48000,
        "stars" to 3,
        "time_seconds" to 127
    )
)

// Track with immediate flush (for critical events)
LinkzlySDK.trackGamingEventImmediate(
    eventType = "purchase",
    data = mapOf(
        "item" to "gem_pack_100",
        "price" to 4.99,
        "currency" to "USD"
    )
)
```

A gaming `purchase` event is game telemetry and is not recorded as revenue; report revenue with `trackPurchase`.

### Gaming Sessions

Sessions are tracked automatically when `autoSessionTracking` is enabled. For manual control:

```kotlin
LinkzlySDK.startGamingSession()
LinkzlySDK.endGamingSession()
```

### Gaming Attribution

Link game installs and opens to marketing campaigns:

```kotlin
// Set attribution data (e.g., from a deep link or install referrer)
LinkzlySDK.setGamingAttribution(
    clickId = "click_abc123",
    deferredDeepLink = "yourapp://game/promo?bonus=100",
    metadata = mapOf("campaign" to "summer_sale")  // optional
)

// Clear attribution
LinkzlySDK.clearGamingAttribution()
```

### Gaming Event Queue

```kotlin
// Manually flush the gaming event queue
LinkzlySDK.flushGamingEvents { result ->
    result?.onSuccess { Log.d("Gaming", "Events flushed") }
}

// Get pending gaming event count
val count: Int = LinkzlySDK.getGamingPendingEventCount()

// Check if a batch is currently being sent
val inflight: Boolean = LinkzlySDK.hasGamingInflightBatch()

// Reset all gaming tracking state (clears queue, player ID, attribution)
LinkzlySDK.resetGamingTracking()
```

### GamingOptions Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `apiKey` | `String` | required | Gaming API key |
| `organizationId` | `String` | required | Organization identifier |
| `gameId` | `String` | required | Game identifier |
| `baseUrl` | `String` | `"https://gaming.linkzly.com"` | API base URL |
| `endpointPath` | `String` | `"/api/v1/gaming/events"` | Events endpoint path |
| `sdkVersion` | `String` | `"1.0.0"` | SDK version string |
| `gameVersion` | `String` | `""` | Your game version |
| `includeTraits` | `Boolean` | `false` | Include player traits in events |
| `debug` | `Boolean` | `false` | Enable debug logging |
| `maxBatchSize` | `Int` | `100` | Max events per batch |
| `maxBatchBytes` | `Int` | `524288` (512 KB) | Max batch size in bytes |
| `flushIntervalMs` | `Int` | `5000` | Auto-flush interval (ms) |
| `maxRetries` | `Int` | `3` | Max retry attempts per batch |
| `retryDelayMs` | `Int` | `1000` | Delay between retries (ms) |
| `maxQueueSize` | `Int` | `10000` | Max queued events |
| `sessionTimeoutMs` | `Int` | `1800000` (30 min) | Session timeout (ms) |
| `autoSessionTracking` | `Boolean` | `true` | Auto track sessions on lifecycle |
| `signingSecret` | `String?` | `null` | HMAC signing secret for requests |

## Debug and Batching Controls

`LinkzlySDKDebug` (package `com.linkzly.sdk.utils`) changes how the event queue batches and flushes, so you can watch events arrive without waiting for the default timers. It is for development builds only.

**When they take effect.** Each method first checks `isAppDebuggable(context)`, which reads your app's `FLAG_DEBUGGABLE` (the app's build type, not the SDK library's). In a debuggable build the values you set override the server-driven configuration and the defaults. In a non-debuggable build the setters log a warning and do nothing, the getters return `null`, and the queue ignores saved overrides. Do not ship calls to these methods in a release build.

**Strategies.** The strategy is a string:

| Strategy | Effect |
|---|---|
| `"smart"` | Only custom events are batched; install, open and purchase events are sent immediately. This is the default. |
| `"all"` | Every event type is batched. |
| `"instant"` | Nothing is batched; every event is sent immediately. |
| `"custom"` | Same as `"smart"`. |

The `BatchingStrategy` class in the SDK's `models` package is deprecated and is not used by these methods.

**Limits and defaults.** Batch size accepts 1 to 100 (default 20). Flush interval is in seconds and accepts 10 to 300 (default 30). Out-of-range values throw `IllegalArgumentException`.

```kotlin
import com.linkzly.sdk.utils.LinkzlySDKDebug

LinkzlySDKDebug.setBatchingStrategy(context, "all")
LinkzlySDKDebug.setBatchSize(context, 5)
LinkzlySDKDebug.setFlushInterval(context, 15)

LinkzlySDKDebug.printDebugConfig(context)        // logs the current values
val strategy: String? = LinkzlySDKDebug.getDebugStrategy(context)
val size: Int? = LinkzlySDKDebug.getDebugBatchSize(context)
val interval: Long? = LinkzlySDKDebug.getDebugFlushInterval(context)

// Set batch size, flush interval and (optionally) strategy in one call
LinkzlySDKDebug.simulateServerConfig(context, batchSize = 10, flushInterval = 30, strategy = "all")

LinkzlySDKDebug.resetDebugConfig(context)        // back to defaults
val debuggable: Boolean = LinkzlySDKDebug.isAppDebuggable(context)
```

After `simulateServerConfig`, restart the SDK or call `flushEvents` to apply the change immediately. Its `ttl` parameter is unused.

| Method | Description |
|--------|-------------|
| `setBatchingStrategy(context, strategy)` | Overrides the batching strategy |
| `setBatchSize(context, size)` | Overrides the batch size (1 to 100) |
| `setFlushInterval(context, intervalSeconds)` | Overrides the flush interval (10 to 300 seconds) |
| `simulateServerConfig(context, batchSize, flushInterval, ttl, strategy)` | Sets batch size, flush interval and optionally strategy at once |
| `resetDebugConfig(context)` | Clears every override |
| `printDebugConfig(context)` | Logs the current overrides |
| `getDebugStrategy(context)` | Returns the strategy override, or `null` |
| `getDebugBatchSize(context)` | Returns the batch size override, or `null` |
| `getDebugFlushInterval(context)` | Returns the flush interval override, or `null` |
| `isAppDebuggable(context)` | Returns `true` when the host app is debuggable |

## App Links Setup

Android App Links allow your app to handle `https://` URLs directly without a disambiguation dialog.

### Using Linkzly Hosted Verification (Recommended)

If you chose "Hosted Verification" when creating your app in the console, Linkzly automatically hosts your `assetlinks.json` file. You only need to add the intent-filter to your `AndroidManifest.xml`:

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="{your-prefix}.linkz.ly" />
</intent-filter>
```

Replace `{your-prefix}` with the brand prefix you chose in the console. No manual file hosting required.

### Self-Hosted Verification

If you chose "Custom Domain Verification", follow these steps:

#### 1. Create assetlinks.json

Host the following file at `https://yourdomain.com/.well-known/assetlinks.json`:

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.yourcompany.yourapp",
    "sha256_cert_fingerprints": [
      "YOUR:SHA256:CERTIFICATE:FINGERPRINT:HERE"
    ]
  }
}]
```

### 2. Get Your Certificate Fingerprint

```bash
# For debug builds
keytool -list -v -keystore ~/.android/debug.keystore \
  -alias androiddebugkey -storepass android -keypass android

# For release builds
keytool -list -v -keystore your-release-key.jks \
  -alias your-alias
```

Copy the `SHA256` fingerprint from the output.

### 3. Host the File

- The file must be served over HTTPS
- Content-Type must be `application/json`
- The file must be accessible without redirects
- Include fingerprints for both debug and release certificates during development

### 4. Verify Configuration

```bash
# Check if App Links are verified on the device
adb shell pm get-app-links com.yourcompany.yourapp

# Manually trigger verification
adb shell pm verify-app-links --re-verify com.yourcompany.yourapp
```

The `android:autoVerify="true"` attribute in your intent filter triggers automatic verification at install time.

## Advanced Usage

### Custom Event Parameter Types

The `parameters` map accepts `Map<String, Any>` and supports these value types:

```kotlin
LinkzlySDK.trackEvent("complex_event", mapOf(
    "string_val" to "hello",       // String
    "int_val" to 42,               // Int
    "double_val" to 3.14,          // Double
    "bool_val" to true,            // Boolean
    "long_val" to 1234567890L      // Long
))
```

### Deep Link Data Access Patterns

```kotlin
val data = LinkzlySDK.handleAppLink(intent)

// Pattern 1: Null-safe chaining
val productId = data?.getStringParameter("id") ?: return

// Pattern 2: Full extraction
data?.let { deepLink ->
    val path = deepLink.path           // e.g., "/product"
    val id = deepLink.getIntParameter("id")
    val price = deepLink.getDoubleParameter("price")
    val featured = deepLink.getBooleanParameter("featured")

    when (path) {
        "/product" -> showProduct(id)
        "/category" -> showCategory(deepLink.getStringParameter("name"))
        "/promo" -> applyPromo(deepLink.getStringParameter("code"))
    }
}

// Pattern 3: Create DeepLinkData from a map (for testing)
val testData = DeepLinkData.fromMap(
    url = "yourapp://product?id=123",
    path = "/product",
    parameters = mapOf("id" to "123", "name" to "Test Product")
)
```

### Lifecycle Integration

The SDK observes `ProcessLifecycleOwner` for automatic lifecycle tracking. If you need to track app state changes yourself:

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        LinkzlySDK.configure(this, "slk_your_key_from_console")

        // SDK automatically handles:
        // - Install tracking on first launch
        // - Open tracking on subsequent launches
        // - Session start/end on foreground/background transitions
        // - Event queue flush on app background
    }
}
```

## Privacy & Compliance

### Consent and opt-out

```kotlin
// Call after configure(): before then setTrackingEnabled has no effect
// (stops custom events, purchases, refunds and trackEventBatch; not install, open or session-end events)
fun onUserConsentUpdated(hasConsent: Boolean) {
    LinkzlySDK.setTrackingEnabled(hasConsent)
    LinkzlySDK.setAdvertisingTrackingEnabled(hasConsent)
}

// Complete opt-out flow
fun handleOptOut() {
    LinkzlySDK.setTrackingEnabled(false)
    LinkzlySDK.setAdvertisingTrackingEnabled(false)
    LinkzlySDK.setUserID("")
    LinkzlySDK.resetVisitorID()
    LinkzlySDK.clearAffiliateAttribution()
}
```

The SDK sends nothing before `configure`: an app that must send nothing before the user consents delays `configure` until the user has consented, and uses `setTrackingEnabled(false)` afterwards to stop events, purchases and refunds.

### Data Collected

| Data | When | Purpose |
|------|------|---------|
| Device model, manufacturer | Every event | Device identification |
| Carrier name | Every event | User context |
| Android version, API level | Every event | Compatibility tracking |
| App version, package name | Every event | App identification |
| Screen size, locale, timezone | Every event | User context |
| Android ID | Every event | Device fingerprinting |
| Install Referrer | Install event | Attribution |
| GAID | Every event (when enabled) | Advertising attribution |
| LAT status | Every event | Privacy preference |

### Data NOT Collected

- Location data
- Contact information
- SMS/call logs
- Browsing history
- Photos or media
- No PII unless explicitly set via `setUserID()`

## ProGuard/R8

The SDK includes consumer ProGuard rules - **no additional configuration needed!**

If you encounter issues, add to your `proguard-rules.pro`:

```proguard
# Keep Linkzly SDK public API
-keep public class com.linkzly.sdk.LinkzlySDK { *; }
-keep public class com.linkzly.sdk.models.** { *; }
-keep public class com.linkzly.sdk.gaming.** { *; }

# Keep OkHttp (used internally)
-dontwarn okhttp3.**
-dontwarn okio.**
```

## Troubleshooting

### SDK Not Initializing

1. Verify `configure()` is called in `Application.onCreate()`, not in an Activity
2. Check that the `sdkKey` is correct
3. Verify the Application class is registered in `AndroidManifest.xml`
4. Check Logcat for error messages: `adb logcat | grep LinkzlySDK`

### Events Not Tracking

1. For a custom event, confirm its name is registered for the app in the console: an unregistered name is dropped and the batched path still reports success
2. Confirm tracking is enabled: `LinkzlySDK.isTrackingEnabled()`
3. Check internet permission is granted
4. Verify device has network connectivity
5. Check pending event count: `LinkzlySDK.getPendingEventCount()`
6. Try manual flush: `LinkzlySDK.flushEvents { ... }`

### Deep Links Not Working

1. Verify intent filter has `VIEW` action with `DEFAULT` and `BROWSABLE` categories
2. Verify data scheme matches your test URL
3. Ensure `android:launchMode="singleTask"` is set on the activity
4. Handle `onNewIntent()` for when the activity is already running
5. Test with ADB:
   ```bash
   adb shell am start -W -a android.intent.action.VIEW \
     -d "yourapp://test" com.yourcompany.yourapp
   ```

### App Links Not Verifying

1. Confirm `assetlinks.json` is accessible at `https://yourdomain.com/.well-known/assetlinks.json`
2. Verify the SHA256 fingerprint matches your signing certificate
3. Check the file is served with `Content-Type: application/json` and no redirects
4. Re-verify: `adb shell pm verify-app-links --re-verify com.yourcompany.yourapp`
5. Check status: `adb shell pm get-app-links com.yourcompany.yourapp`

### Install Referrer Issues

1. Ensure the app is installed from Google Play (or use a test referrer)
2. The Install Referrer API requires Google Play Services
3. Without referrer data the install is still tracked, but without the referrer

### Build Errors

**Duplicate class kotlin.collections.CollectionsKt:**
```gradle
// Update Kotlin version in build.gradle
buildscript {
    ext.kotlin_version = '1.9.20'
}
```

**Gradle sync failed:**
- Verify JitPack repository is added to `settings.gradle` (not just `build.gradle`)
- Check your internet connection
- Try `./gradlew --refresh-dependencies`

## Architecture

```
+-------------------------------------------+
|        LinkzlySDK (Singleton)             |
|  Public API for initialization & events   |
+---------+-----------+---------------------+
          |           |
    +-----+-----+  +-+------------+
    | Attribution|  |   Network    |
    |  Service   |--|   Service    |
    |            |  |  (OkHttp)    |
    +-----+------+  +-----------+-+
          |                     |
    +-----+------+    +--------+--------+
    |  Install   |    | Gaming Tracking |
    |  Referrer  |    |   (Batched)     |
    +------------+    +-----------------+
    +------------+    +-----------------+
    |  Affiliate |    |  Push Service   |
    |  Service   |    |   (FCM)         |
    +------------+    +-----------------+
    +------------+
    | DeviceInfo |
    | (Fingerprint)|
    +------------+
```

### Event Flow

1. **Event created** - `trackEvent()`, `trackPurchase()`, or lifecycle event
2. **Queued locally** - Added to persistent event queue
3. **Batched** - Events are grouped for efficient network usage
4. **Sent** - HTTP POST to Linkzly API with retry logic
5. **Confirmed** - Successfully sent events are removed from queue

### Session Management

- A session starts each time the app enters the foreground
- A session ends when the app enters the background, and pending events are flushed
- An open event is tracked when the app enters the foreground more than 30 seconds after the last foreground
- The session id is included with events

## Performance

### Network Efficiency
- Events are batched to reduce HTTP requests
- Automatic retry with exponential backoff
- Offline queuing with flush on connectivity restore

### Battery & Resource Usage
- No background services or wake locks
- Lifecycle-aware observers (no polling)
- Lightweight serialization with kotlinx.serialization

## FAQ

**Can I use this with React Native or Flutter?**
Linkzly publishes a React Native SDK and a Flutter plugin that wrap the native SDKs; see the documentation. Every `LinkzlySDK` method is `@JvmStatic`, so it is also callable from Java.

**Does it work with Jetpack Compose?**
Yes. The SDK is lifecycle-aware and works with any UI framework. Call `handleAppLink(intent)` from your Activity regardless of whether you use Compose or Views.

**What happens when the device is offline?**
Events are queued locally in persistent storage. They are automatically sent when connectivity is restored. The queue holds up to 1,000 events; when it is full, the oldest event is dropped.

**Does it work on Huawei devices (no Google Play Services)?**
Core event tracking and deep linking work. Install Referrer and GAID collection require Google Play Services and will be unavailable on Huawei devices.

## Integration Checklist

- [ ] Created app in Linkzly Console (Dashboard > Apps)
- [ ] Copied SDK key from console
- [ ] Added Linkzly SDK dependency (`1.0.11`) via Gradle
- [ ] Called `LinkzlySDK.configure(...)` in Application class
- [ ] Added `intent-filter` with `android:autoVerify="true"` in AndroidManifest.xml
- [ ] Implemented deep link handling in MainActivity
- [ ] Registered device push token via `setNotificationToken` (and cleared on logout)
- [ ] Verified integration in Linkzly Console (Manage app > Verification tab, the SDK connection card)

## Support

- [Documentation](https://docs.linkzly.com/docs/sdk-android)
- [Issue Tracker](https://github.com/Linkzly/linkzly-android-sdk/issues)
- Email: support@linkzly.com

## Related SDKs

- **iOS SDK**: [linkzly-ios-sdk](https://github.com/Linkzly/linkzly-ios-sdk)

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

**Made with care by the Linkzly Team**

[Website](https://linkzly.com) | [Documentation](https://docs.linkzly.com) | [GitHub](https://github.com/Linkzly) | [Issues](https://github.com/Linkzly/linkzly-android-sdk/issues) | [support@linkzly.com](mailto:support@linkzly.com)
