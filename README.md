# Linkzly Android SDK

[![JitPack](https://jitpack.io/v/Linkzly/linkzly-android-sdk.svg)](https://jitpack.io/#Linkzly/linkzly-android-sdk)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Android](https://img.shields.io/badge/platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-21-orange.svg)](https://developer.android.com/about/versions/lollipop)

The official Android SDK for [Linkzly](https://linkzly.com) - a powerful mobile app attribution and deep linking platform.

## Features

- **Attribution Tracking** - Install tracking with Google Play Install Referrer API, app open tracking, automatic session management
- **Deep Linking** - Standard deep links (URL schemes, App Links), deferred deep linking, smart link parameter extraction
- **Custom Event Tracking** - Track in-app events with custom parameters, batch tracking, purchase tracking
- **Affiliate Attribution** - Capture affiliate click IDs from deep links, server-to-server attribution, 30-day storage with expiry
- **Push Notifications** - Firebase Cloud Messaging (FCM) integration, token management
- **Gaming Tracking** - Dedicated gaming event pipeline, player identification, session tracking, gaming-specific attribution
- **Privacy Controls** - GDPR/CCPA compliant, user opt-in/opt-out, advertising tracking controls, two-tier consent model
- **Offline Event Queue** - Automatic event queuing when offline, manual flush, pending event inspection
- **Production Ready** - ProGuard/R8 compatible, lightweight (< 100KB), coroutine-based async operations

## Requirements

| Component | Minimum Version | Recommended |
|-----------|----------------|-------------|
| Android SDK | API 21 (5.0 Lollipop) | API 33+ (13.0) |
| Target SDK | API 34 (14.0) | API 34+ |
| Java | 8+ | 11+ |
| Kotlin | 1.8+ | 1.9+ |
| Gradle | 7.0+ | 8.0+ |
| Android Studio | 2021.3.1+ | Latest |

**Language Support:** Kotlin (primary, all examples) and Java (fully compatible).

## Prerequisites

Before integrating the SDK, set up your app in the Linkzly Console:

1. Go to **Dashboard > Apps** and click "Register App"
2. Enter your Android **Package Name** and **SHA-256 Fingerprints**
3. Choose a verification method (Hosted recommended for quick start)
4. Copy your **SDK Key** from the post-creation wizard or from Manage App > Overview > SDK Configuration

Your SDK key starts with `slk_` and uniquely identifies your app.

## Getting Your SDK Key

Your SDK key (`slk_` prefix) authenticates your app with Linkzly's servers. You can find it in:

- **Post-creation wizard**: Displayed prominently right after creating your app
- **Dashboard > Apps > Manage App > Overview > SDK Configuration**: Click the eye icon to reveal, or copy directly

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

Add the SDK to your app's `build.gradle`:

```gradle
dependencies {
    implementation 'com.github.Linkzly:linkzly-android-sdk:1.0.1'
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
            sdkKey = "slk_your_key_from_console"  // Get this from Dashboard > Apps > Manage App,
            environment = Environment.STAGING  // Use PRODUCTION for release
        )
    }
}
```

**Don't forget to register your Application class in AndroidManifest.xml:**
```xml
<application android:name=".MyApplication" ...>
```

### 3. Track Events

```kotlin
LinkzlySDK.trackEvent("purchase_completed", mapOf(
    "product_id" to "abc123",
    "price" to 29.99,
    "currency" to "USD"
))
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

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        intent?.let {
            val deepLinkData = LinkzlySDK.handleAppLink(it)
            deepLinkData?.let { data ->
                navigateTo(data.path, data.getStringParameter("id"))
            }
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
```kotlin
LinkzlySDK.trackEvent("test_event", mapOf("test" to true))
```

Logcat should show:
```
D/LinkzlySDK: Tracking event: test_event with parameters: {test=true}
D/LinkzlySDK: Event tracked successfully
```

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
- [ ] Custom events tracked successfully
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

Call this once in your `Application.onCreate()`. The `Environment` enum supports `PRODUCTION` and `STAGING`.

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
// Track install (first launch) - returns deferred deep link data if available
LinkzlySDK.trackInstall { result ->
    result.onSuccess { deepLinkData ->
        deepLinkData?.let { navigateTo(it.path) }
    }
}

// Track app open
LinkzlySDK.trackOpen { result ->
    result.onSuccess { deepLinkData ->
        deepLinkData?.let { handleDeepLink(it) }
    }
}
```

Both install and open events are tracked automatically by the SDK's lifecycle observer. Use these methods when you need the callback (e.g., for deferred deep links).

### Event Tracking

```kotlin
// Track a custom event
LinkzlySDK.trackEvent("level_completed", mapOf(
    "level" to 5,
    "score" to 12500
))

// Track a purchase
LinkzlySDK.trackPurchase(
    parameters = mapOf(
        "amount" to 9.99,
        "currency" to "USD",
        "sku" to "premium_monthly"
    ),
    callback = { result ->  // optional
        result.onSuccess { Log.d("Linkzly", "Purchase tracked") }
    }
)

// Track multiple events in a batch
LinkzlySDK.trackEventBatch(
    events = listOf(event1, event2, event3),
    callback = { result ->
        result.onSuccess { Log.d("Linkzly", "Batch tracked") }
    }
)
```

### Session Management

The SDK tracks sessions automatically using `ProcessLifecycleOwner` with a 30-minute timeout. For manual control:

```kotlin
LinkzlySDK.startSession()
LinkzlySDK.endSession()
```

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

Events are queued locally and flushed automatically. For manual control:

```kotlin
// Flush pending events immediately
LinkzlySDK.flushEvents { result ->
    result.onSuccess { Log.d("Linkzly", "Events flushed") }
}

// Get the number of pending events
val count: Int = LinkzlySDK.getPendingEventCount()

// Get the list of pending events
val events: List<QueuedEvent> = LinkzlySDK.getPendingEvents()
```

### Privacy Controls

```kotlin
// Disable/enable all tracking
LinkzlySDK.setTrackingEnabled(false)
val isEnabled: Boolean = LinkzlySDK.isTrackingEnabled()

// Disable/enable advertising identifier (GAID) collection
LinkzlySDK.setAdvertisingTrackingEnabled(false)
val isAdEnabled: Boolean = LinkzlySDK.isAdvertisingTrackingEnabled()
```

When tracking is disabled, the SDK stops collecting and sending all events. Advertising tracking controls only affect GAID collection while allowing other event tracking to continue.

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

override fun onNewIntent(intent: Intent?) {
    super.onNewIntent(intent)
    intent?.data?.let { LinkzlySDK.captureAffiliateAttribution(it) }
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

    // Or get just the click ID for S2S postback
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

- Attribution data is stored in SharedPreferences
- Default expiry: **30 days** from capture
- Expired attribution is automatically cleared on next access
- Call `clearAffiliateAttribution()` to manually remove stored data

## Push Notification Support

### Prerequisites

- Firebase Cloud Messaging (FCM) integrated in your app
- A valid `google-services.json` in your app module

### Setup

```kotlin
// Initialize push notification support (registers FCM token with Linkzly)
val success = LinkzlySDK.initializePush()

// Disable push notifications (unregisters token)
val disabled = LinkzlySDK.disablePush()
```

Call `initializePush()` after `configure()`, typically in your `Application.onCreate()` or after the user opts in to notifications.

### Non-FCM Providers

If you use a push provider other than Firebase (e.g., Huawei Push Kit), handle token registration through your own integration and pass attribution data via custom events.

### Troubleshooting Push

- Ensure `google-services.json` is present and valid
- Verify Firebase dependencies are included in your `build.gradle`
- Check that the device has Google Play Services installed
- Confirm the FCM sender ID matches your Firebase project

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

### Manual Session Management

Override automatic session tracking for custom session boundaries:

```kotlin
// Start tracking a custom session (e.g., a game round)
LinkzlySDK.startSession()

// ... user activity ...

// End the custom session
LinkzlySDK.endSession()
```

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

### GDPR Implementation

```kotlin
// Check consent before enabling tracking
fun onUserConsentUpdated(hasConsent: Boolean) {
    LinkzlySDK.setTrackingEnabled(hasConsent)
    LinkzlySDK.setAdvertisingTrackingEnabled(hasConsent)
}

// Complete opt-out flow
fun handleGDPRDeletion() {
    LinkzlySDK.setTrackingEnabled(false)
    LinkzlySDK.setAdvertisingTrackingEnabled(false)
    LinkzlySDK.setUserID("")
    LinkzlySDK.resetVisitorID()
    LinkzlySDK.clearAffiliateAttribution()
}
```

### Data Collected

| Data | When | Purpose |
|------|------|---------|
| Device model, manufacturer | Every event | Device identification |
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

1. Confirm tracking is enabled: `LinkzlySDK.isTrackingEnabled()`
2. Check internet permission is granted
3. Verify device has network connectivity
4. Check pending event count: `LinkzlySDK.getPendingEventCount()`
5. Try manual flush: `LinkzlySDK.flushEvents { ... }`

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
3. Referrer data is only available for ~90 seconds after install
4. Test with: `adb shell am broadcast -a com.android.vending.INSTALL_REFERRER`

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

- Sessions start when the app enters the foreground
- Sessions end when the app enters the background
- A new session is created if the app was backgrounded for more than 30 minutes
- Session data is included with all events for attribution accuracy

## Performance

### Network Efficiency
- Events are batched to reduce HTTP requests
- Automatic retry with exponential backoff
- Offline queuing with flush on connectivity restore

### Battery & Resource Usage
- No background services or wake locks
- Lifecycle-aware observers (no polling)
- Lightweight serialization with kotlinx.serialization
- SDK size: < 100KB (excluding dependencies)

## FAQ

**Can I use this with React Native?**
Yes. Create a native module that bridges to `LinkzlySDK` methods. All methods are static and callable from Java/Kotlin bridge code.

**Does it work with Jetpack Compose?**
Yes. The SDK is lifecycle-aware and works with any UI framework. Call `handleAppLink(intent)` from your Activity regardless of whether you use Compose or Views.

**What happens when the device is offline?**
Events are queued locally in persistent storage. They are automatically sent when connectivity is restored. The queue holds up to 10,000 events.

**Does it work on Huawei devices (no Google Play Services)?**
Core event tracking and deep linking work. Install Referrer and GAID collection require Google Play Services and will be unavailable on Huawei devices.

**How large is the SDK?**
The SDK adds approximately 80-100KB to your APK (excluding shared dependencies like OkHttp and kotlinx.serialization).

## Integration Checklist

- [ ] Created app in Linkzly Console (Dashboard > Apps)
- [ ] Copied SDK key from console
- [ ] Added Linkzly SDK dependency via Gradle
- [ ] Called `LinkzlySDK.initialize()` in Application class
- [ ] Added `intent-filter` with `android:autoVerify="true"` in AndroidManifest.xml
- [ ] Implemented deep link handling in MainActivity
- [ ] Verified integration in Linkzly Console (Manage App > Integration tab)

## Support

- [Documentation](https://app.linkzly.com)
- [Issue Tracker](https://github.com/Linkzly/linkzly-android-sdk/issues)
- Email: support@linkzly.com

## Example Apps

- **Android Example**: [linkzly-android-sdk-example](https://github.com/Linkzly/linkzly-android-sdk-example)

## Related SDKs

- **iOS SDK**: [linkzly-ios-sdk](https://github.com/Linkzly/linkzly-ios-sdk)

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

**Made with care by the Linkzly Team**

[Website](https://linkzly.com) | [Documentation](https://docs.linkzly.com) | [GitHub](https://github.com/Linkzly) | [Issues](https://github.com/Linkzly/linkzly-android-sdk/issues) | [support@linkzly.com](mailto:support@linkzly.com)
