# Linkzly Android SDK

[![JitPack](https://jitpack.io/v/Linkzly/linkzly-android-sdk.svg)](https://jitpack.io/#Linkzly/linkzly-android-sdk)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Android](https://img.shields.io/badge/platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-21-orange.svg)](https://developer.android.com/about/versions/lollipop)

The official Android SDK for [Linkzly](https://linkzly.com) - a powerful mobile app attribution and deep linking platform.

## Features

✨ **Automatic Attribution Tracking**
- Install tracking with Google Play Install Referrer API
- App open tracking with lifecycle awareness
- Session management

🔗 **Deep Linking**
- Standard deep links (URL schemes, App Links)
- Deferred deep linking (install attribution)
- Smart link parameter extraction

📊 **Custom Event Tracking**
- Track in-app events with custom parameters
- User identification
- Full attribution data

🔒 **Privacy First**
- GDPR compliant
- User opt-in/opt-out controls
- Configurable data collection

⚡ **Production Ready**
- ProGuard/R8 compatible
- Lightweight (< 100KB)
- Coroutine-based async operations
- Comprehensive error handling

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
    implementation 'com.github.Linkzly:linkzly-android-sdk:1.0.0'
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
            sdkKey = "your-sdk-key-here",
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
// Custom events are tracked automatically, or manually:
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
            // Navigate based on deep link
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

That's it! 🎉

## Verify Your Setup

After integration, verify the SDK is working correctly:

**1. Check SDK Initialization:**
```bash
adb logcat | grep LinkzlySDK
```

Look for:
```
✅ "LinkzlySDK configured successfully"
✅ "Tracking install event" (first launch only)
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

## Requirements

| Component | Minimum Version | Recommended |
|-----------|----------------|-------------|
| Android SDK | API 21 (5.0 Lollipop) | API 33+ (13.0) |
| Target SDK | API 34 (14.0) | API 34+ |
| Java | 8+ | 11+ |
| Kotlin | 1.8+ | 1.9+ |
| Gradle | 7.0+ | 8.0+ |
| Android Studio | 2021.3.1+ | Latest |

**Language Support:**
- Kotlin (primary, all examples)
- Java (fully compatible)

## API Reference

### Lifecycle Events (Automatic)

The Android SDK automatically tracks application lifecycle events using `ProcessLifecycleOwner`.
*   **Install**: Tracked on the first launch.
*   **Open**: Tracked on every app launch.

### Session Management (Manual)

Use these methods to manually control session boundaries.

```kotlin
// Start a session
LinkzlySDK.startSession()

// End a session
LinkzlySDK.endSession()
```

### Commerce Events

Track revenue and in-app purchases.

```kotlin
// Track a purchase
val purchaseParams = mapOf(
    "amount" to 9.99,
    "currency" to "USD",
    "sku" to "premium_monthly"
)
LinkzlySDK.trackPurchase(parameters = purchaseParams)
```

### Custom Events

Track any user interaction.

```kotlin
// Track a custom event
LinkzlySDK.trackEvent("tutorial_completed", parameters = mapOf("step" to 5))
```

### Batch Tracking

```kotlin
// Track multiple events
LinkzlySDK.trackEventBatch(listOf(event1, event2))
```

### Deep Linking

**Standard Deep Links:**
```kotlin
// yourapp://product?id=123
val deepLinkData = LinkzlySDK.handleAppLink(intent)
val productId = deepLinkData?.getStringParameter("id") // "123"
```

**Deferred Deep Links:**
```kotlin
// User clicks link → installs app → opens app
LinkzlySDK.trackInstall { result ->
    result.onSuccess { deepLinkData ->
        // Navigate to originally intended destination
        deepLinkData?.let { navigateTo(it.path) }
    }
}
```

### User Management

```kotlin
// Set user ID (after login)
LinkzlySDK.setUserID("user_12345")

// Get current user ID
val userId = LinkzlySDK.getUserID()

// Clear user ID (after logout)
LinkzlySDK.setUserID("")
```

### Privacy Controls

```kotlin
// Disable tracking
LinkzlySDK.setTrackingEnabled(false)

// Check status
val isEnabled = LinkzlySDK.isTrackingEnabled()

// Respect user privacy choice
if (userConsentedToTracking) {
    LinkzlySDK.setTrackingEnabled(true)
} else {
    LinkzlySDK.setTrackingEnabled(false)
}
```

### Visitor ID

```kotlin
// Get unique visitor ID (auto-generated)
val visitorId = LinkzlySDK.getVisitorID()

// Reset visitor ID (e.g., on logout)
LinkzlySDK.resetVisitorID()
```

### Flush Events

```kotlin
// Manually flush pending events
LinkzlySDK.flushEvents { result ->
    result.onSuccess { Log.d("Linkzly", "Events flushed") }
}

// Get pending event count
val count = LinkzlySDK.getPendingEventCount()
```

## ProGuard/R8

The SDK includes consumer ProGuard rules - **no additional configuration needed!**

If you encounter issues, add to your `proguard-rules.pro`:

```proguard
# Keep Linkzly SDK public API
-keep public class com.linkzly.sdk.LinkzlySDK { *; }
-keep public class com.linkzly.sdk.models.** { *; }
```

## Privacy & GDPR

The SDK is designed with privacy in mind:

- ✅ User consent controls
- ✅ Opt-in/opt-out mechanisms
- ✅ Minimal data collection
- ✅ No PII without explicit user ID setting
- ✅ Transparent data collection

**Data Collected:**
- Device model and manufacturer
- Android version and API level
- App version and package name
- Screen size, locale, timezone
- Android ID
- Install Referrer data (from Google Play)

## Troubleshooting

### SDK Not Tracking Events

1. Verify SDK is configured in `Application.onCreate()`
2. Check internet permission is granted
3. Verify device has network connectivity
4. Confirm tracking is enabled: `LinkzlySDK.isTrackingEnabled()`

### Deep Links Not Working

1. Verify intent filter has `VIEW` action
2. Verify intent filter has `DEFAULT` and `BROWSABLE` categories
3. Verify data scheme matches your test URL
4. Test with ADB command

### Build Errors

**Duplicate class kotlin.collections.CollectionsKt:**
```gradle
// Update Kotlin version in build.gradle
buildscript {
    ext.kotlin_version = '1.9.0'
}
```

## Support

- 📚 [Documentation](https://app.linkzly.com)
- 🐛 [Issue Tracker](https://github.com/Linkzly/linkzly-android-sdk/issues)
- 📧 Email: support@linkzly.com

## Example Apps

- **Android Example**: [linkzly-android-sdk-example](https://github.com/Linkzly/linkzly-android-sdk-example)

## Related SDKs

- **iOS SDK**: [linkzly-ios-sdk](https://github.com/Linkzly/linkzly-ios-sdk)

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

**Made with ❤️ by the Linkzly Team**

[Website](https://linkzly.com) • [Documentation](https://docs.linkzly.com) • [GitHub](https://github.com/Linkzly) • [Issues](https://github.com/Linkzly/linkzly-android-sdk/issues) • [support@linkzly.com](mailto:support@linkzly.com)
