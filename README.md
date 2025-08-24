# Build Configuration


## Plugins

``` kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.kotlin.kapt)
    alias(libs.plugins.kotlin.parcelize)
}
```

-   **android.application**: Required for building Android applications.
-   **kotlin.android**: Adds Kotlin language support.
-   **kotlin.serialization**: Enables Kotlin Serialization support.
-   **kotlin.kapt**: Kotlin Annotation Processing
-   **kotlin.parcelize**: Simplifies implementing `Parcelable` classes.

------------------------------------------------------------------------

## Android Configuration

### General Settings

``` kotlin
android {
    namespace = "com.andrew.mark.crf23"
    compileSdk = 36
}
```

-   **namespace**: The application's package namespace.
-   **compileSdk = 36**: Targets Android API level 36 (Android 16
    features available).

### Default Configuration

``` kotlin
defaultConfig {
    applicationId = "com.andrew.mark.crf23"
    minSdk = 26
    targetSdk = 36
    versionCode = 11
    versionName = "11.0"
    ndk {
        abiFilters.add("armeabi-v7a")
        abiFilters.add("arm64-v8a")
    }
}
```

-   **applicationId**: Unique identifier for the application.
-   **minSdk = 26**: Supports Android 8.0 (API 26) and above.
-   **targetSdk = 36**: Optimized for Android 16.
-   **versionCode / versionName**: Internal and display versioning.
-   **abiFilters**: Includes only ARM architectures (armeabi-v7a,
    arm64-v8a).

------------------------------------------------------------------------

## Signing Configuration

``` kotlin
signingConfigs {
    create("release") {
        storeFile = file(localProperties.getProperty("storeFile"))
        storePassword = localProperties.getProperty("storePassword")
        keyAlias = localProperties.getProperty("keyAlias")
        keyPassword = localProperties.getProperty("keyPassword")
        enableV1Signing = true
        enableV2Signing = true
        enableV3Signing = true
    }
}
```

-   **Release signing**: Reads credentials from `local.properties`.
-   **V1/V2/V3 Signing**: Supports legacy and modern Android versions.

------------------------------------------------------------------------

## Build Types

``` kotlin
buildTypes {
    debug {
        isShrinkResources = false
        isMinifyEnabled = false
        proguardFiles(...)
    }

    release {
        signingConfig = signingConfigs.getByName("release")
        isShrinkResources = true
        isMinifyEnabled = true
        isDebuggable = false
        proguardFiles(...)
    }
}
```

-   **debug**: No code/resource shrinking, debuggable.
-   **release**:
    -   **Minification & Shrinking enabled** (R8).
    -   **Debugging disabled**.
    -   **ProGuard/R8 rules applied**.

------------------------------------------------------------------------

## Compile Options

``` kotlin
compileOptions {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
}

kotlinOptions {
    jvmTarget = "11"
}
```

-   Supports **Java 11** and **Kotlin JVM 11**.

------------------------------------------------------------------------

## Build Features

``` kotlin
buildFeatures {
    viewBinding = true
    buildConfig = true
    dataBinding = true
}
```

-   **viewBinding**: Type-safe view access.
-   **dataBinding**: XML-to-ViewModel binding.
-   **buildConfig**: Generates BuildConfig constants.

------------------------------------------------------------------------

## Packaging

``` kotlin
packaging {
    resources {
        excludes += "DebugProbesKt.bin"
    }
}
```

-   Excludes **DebugProbesKt.bin** from release builds (coroutines debug
    artifact).

------------------------------------------------------------------------

## Dependencies

``` kotlin
dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.activity)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)

    // Dependency Injection
    implementation(libs.koin.android)

    // CameraX
    implementation(libs.androidx.camera.core)
    implementation(libs.androidx.camera.view)
    implementation(libs.androidx.camera.camera2)
    implementation(libs.androidx.camera.lifecycle)

    // Ktor HTTP Server
    implementation(libs.ktor.server.cio)
    implementation(libs.ktor.server.websockets)
    implementation(libs.ktor.server.content.negotiation)
    implementation(libs.ktor.serialization.kotlinx.json)

    // JSON/Gson
    implementation(libs.gson)

    // Image Loading
    implementation(libs.glide)

    // Local Modules
    implementation(project("::jsonviewer"))
    implementation(project("::bzyuv"))
}
```

-   **AndroidX & Material**: Core Android libraries and UI components.
-   **Koin**: Dependency injection framework.
-   **CameraX**: Modern camera APIs.
-   **Ktor**: Embedded HTTP/WebSocket server support.
-   **Gson**: JSON serialization/deserialization.
-   **Glide**: Image loading and caching.
-   **Local Modules**: Internal project modules (`jsonviewer`, `bzyuv`).

------------------------------------------------------------------------

## Security & Optimization Notes

-   **R8 enabled**: Code minification, obfuscation, and resource
    shrinking active in release builds.
-   **DebugProbes removed**: No coroutine debug artifacts in release.
-   **Multi-ABI (ARMv7 + ARM64)**: Optimized for ARM devices; x86 not
    included.
-   **V1/V2/V3 Signing**: Ensures compatibility across Android versions.

------------------------------------------------------------------------


# Android Manifest

## Device Capabilities

``` xml
<uses-feature
    android:name="android.hardware.camera"
    android:required="true" />
```

-   **Camera required**: The app **won't install** on devices without a
    camera and will be **filtered out** in Play Store for devices
    lacking this hardware.
-   If you want the app to install on devices without a camera (but
    degrade gracefully), set `android:required="false"` and gate
    features at runtime.

------------------------------------------------------------------------

## Permissions

``` xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_SPECIAL_USE" />
```

-   **CAMERA**: Required to open and use the camera. **Runtime
    permission** on Android 6.0+ (API 23+).
-   **INTERNET**: Needed for any network I/O (HTTP/WebSocket, etc.).
-   **ACCESS_WIFI_STATE**: Read Wi-Fi connection state; often used to
    optimize networking or diagnostics.
-   **POST_NOTIFICATIONS**: Required to show notifications on Android
    13+ (API 33+). This is a **runtime permission** (user can deny).
-   **FOREGROUND_SERVICE**: Allows starting a Foreground Service (FGS)
    with a persistent user-visible notification.
-   **FOREGROUND_SERVICE_SPECIAL_USE** (API 34+): Declares use of the
    **`specialUse`** foreground service type (see service section).
    Using this type may require **Play policy justification**; prefer a
    specific FGS type.

> **Runtime UX note:** Request CAMERA and POST_NOTIFICATIONS at runtime
> with clear rationale UI; handle denial and "Don't ask again"
> gracefully.

------------------------------------------------------------------------

## Application Node

``` xml
<application
    android:name=".App"
    android:allowBackup="true"
    android:dataExtractionRules="@xml/data_extraction_rules"
    android:fullBackupContent="@xml/backup_rules"
    android:hardwareAccelerated="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:screenOrientation="portrait"
    android:supportsRtl="true"
    android:theme="@style/Theme.CRF23">
```

-   **`android:name=".App"`**: Custom `Application` subclass used for
    process-wide init (DI etc.).
-   **Backup & Data Export**
    -   `allowBackup="true"` enables system backups (ADB, device
        transfer).\
    -   `dataExtractionRules` / `fullBackupContent` point to XMLs that
        **whitelist/blacklist** data for backup/restore and
        user-initiated data export. Ensure sensitive data is excluded.
-   **`hardwareAccelerated="true"`**: Enables GPU acceleration
    (recommended for camera/graphics).
-   **RTL**: `supportsRtl="true"` opts into right-to-left layout
    mirroring when device language is RTL.

------------------------------------------------------------------------

## Activities

### Launcher / Settings

``` xml
<activity
    android:name=".presentation.settings.SettingsActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

-   **Launcher activity**: Entry point shown in the launcher.\
-   **`exported="true"`** is mandatory because it has an intent filter
    and must be launchable by the system.

### Camera UI

``` xml
<activity
    android:name=".presentation.camera.CameraActivity"
    android:exported="false"
    android:noHistory="true" />
```

-   **Not exported**: Only the app can start it (safer for sensitive
    camera flows).
-   **`noHistory="true"`**: Activity is not kept in the back stack after
    the user leaves (prevents navigating back into an old capture
    session; also reduces exposure of sensitive camera views).

### Result Screen

``` xml
<activity
    android:name=".presentation.result.ResultsActivity"
    android:exported="false" />
```

-   Internal activity used to display results; not launchable by other
    apps.

------------------------------------------------------------------------

## Foreground Service

``` xml
<service
    android:name=".service.HttpServerForegroundService"
    android:exported="false"
    android:foregroundServiceType="specialUse"
    android:stopWithTask="false" />
```

-   **Purpose**: Runs an HTTP/WebSocket server in the foreground
    (inferred from name), keeping the process alive while providing a
    user-visible notification.
-   **`exported="false"`**: Only your app can start/bind this service.
-   **`foregroundServiceType="specialUse"`**: Declares a
    **generic/special** FGS type (API 34+).
    -   Use only if your use case doesn't fit predefined types
        (`dataSync`, `mediaPlayback`, `connectedDevice`, `camera`,
        etc.).\
    -   **Play policy**: You may need to declare/justify this in Play
        Console. If possible, switch to a **specific** type to reduce
        policy friction (e.g., if the service interacts with the camera,
        consider `camera`; for active network transfer, consider
        `dataSync`).
-   **`stopWithTask="false"`**: The service **continues running** even
    if the app's task is removed (Recents swipe). Ensure users have an
    explicit **Stop** action (notification/action button) to avoid
    unexpected battery drain.


# ProGuard/R8 Rules -- Technical Documentation

This document explains the purpose of each rule in the
`proguard-rules.pro` file for this project.

------------------------------------------------------------------------

## **Third-Party Libraries**

### **JsonViewer**

``` proguard
-keep class co.pokeum.jsonviewer.xml.** { *; }
-keep class co.pokeum.jsonviewer.** { *; }
```

-   Prevents obfuscation and removal of all `JsonViewer` classes,
    ensuring runtime reflection or XML parsing works properly.

### **BZYUV Library**

``` proguard
-keep class com.luoye.bzyuvlib.** { *; }
-keep class com.luoye.bzyuvlib.BZYUVUtil { *; }
```

-   Preserves native YUV processing library classes and utilities to
    prevent JNI linkage errors.

------------------------------------------------------------------------

## **Ktor Framework**

### **General Ktor Classes**

``` proguard
-keep class io.ktor.** { *; }
-keep class io.ktor.server.** { *; }
-keep class io.ktor.client.** { *; }
-keep class io.ktor.util.** { *; }
```

-   Keeps Ktor server/client core classes to prevent runtime reflection
    issues in HTTP/WebSocket functionality.

### **Debug Detection**

``` proguard
-keep class io.ktor.util.debug.IntellijIdeaDebugDetector { *; }
-dontwarn io.ktor.util.debug.**
```

-   Retains the IDE debug detector class and suppresses warnings for
    other debug classes.

### **Static Content/File Serving**

``` proguard
-keep class io.ktor.server.http.content.** { *; }
```

-   Ensures static file serving components are retained for HTTP server
    features.

------------------------------------------------------------------------

## **Java Core and NIO**

``` proguard
-keep class com.sun.nio.file.** { *; }
-keep class java.nio.file.** { *; }
-dontwarn com.sun.nio.file.**
-dontwarn java.nio.file.**
```

-   Preserves Java NIO filesystem classes, including internal Sun
    classes, often used by Ktor and file operations.
-   Suppresses warnings for potentially missing `com.sun.nio.file`
    classes on certain devices.

------------------------------------------------------------------------

## **Java Management Classes**

``` proguard
-keep class java.lang.management.** { *; }
-dontwarn java.lang.management.**
```

-   Retains Java Management classes (thread/heap introspection),
    avoiding warnings when used by libraries like Ktor.

------------------------------------------------------------------------

## **Application Code**

### **Main App Package**

``` proguard
-keep class com.andrew.mark.crf23.** { *; }
```

-   Ensures **all project classes** remain intact to avoid breaking core
    logic.

### **Native Methods**

``` proguard
-keepclasseswithmembernames class * {
    native <methods>;
}
```

-   Keeps classes with `native` methods unchanged, ensuring JNI bindings
    remain valid.

------------------------------------------------------------------------

## **View/Data Binding**

### **ViewBinding**

``` proguard
-keep class * implements androidx.viewbinding.ViewBinding {
    public static *** bind(android.view.View);
    public static *** inflate(...);
}
```

-   Retains generated `ViewBinding` classes to support runtime view
    binding.

### **DataBinding**

``` proguard
-keep class * implements androidx.databinding.ViewBinding {
    public static *** bind(android.view.View);
    public static *** inflate(...);
}
```

-   Retains DataBinding-generated classes required by the DataBinding
    runtime.

------------------------------------------------------------------------

## **Kotlin Support**

### **Kotlin Serialization**

``` proguard
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt
```

-   Keeps annotations required by Kotlin Serialization (e.g.,
    `@Serializable`) and prevents unnecessary notes in the build log.

### **Kotlin Coroutines**

``` proguard
-keepnames class kotlinx.coroutines.internal.MainDispatcherFactory {}
-keepnames class kotlinx.coroutines.CoroutineExceptionHandler {}
```

-   Ensures critical Coroutine interfaces remain discoverable at
    runtime.

------------------------------------------------------------------------

## **AndroidX**

``` proguard
-keep class androidx.** { *; }
-keep interface androidx.** { *; }
```

-   Retains all `AndroidX` classes and interfaces to ensure framework
    integration (CameraX, lifecycle, etc.) works.
