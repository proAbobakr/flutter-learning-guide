# Section 2: Project Structure

## Introduction

Understanding Flutter's project structure is crucial for organizing your code effectively. While Android projects follow a specific structure with `app`, `res`, `manifest`, Flutter has a different but intuitive organization.

## Creating a New Project

```bash
# Create new Flutter project
flutter create my_app

# Create with specific organization
flutter create --org com.example my_app

# Create with specific platforms
flutter create --platforms=android,ios,web my_app
```

## Flutter Project Structure

```
my_app/
├── android/                 # Android-specific code
│   ├── app/
│   │   ├── src/
│   │   │   └── main/
│   │   │       ├── AndroidManifest.xml
│   │   │       ├── kotlin/
│   │   │       └── res/
│   │   └── build.gradle
│   └── build.gradle
├── ios/                     # iOS-specific code
│   ├── Runner/
│   │   ├── Info.plist
│   │   └── AppDelegate.swift
│   └── Runner.xcodeproj
├── lib/                     # Main Dart code (YOUR WORK HERE)
│   └── main.dart
├── test/                    # Unit and widget tests
│   └── widget_test.dart
├── web/                     # Web-specific files
├── linux/                   # Linux-specific files
├── macos/                   # macOS-specific files
├── windows/                 # Windows-specific files
├── assets/                  # Images, fonts, etc. (create this)
│   ├── images/
│   └── fonts/
├── pubspec.yaml            # Dependencies and assets (like build.gradle)
├── pubspec.lock            # Locked dependency versions
├── analysis_options.yaml   # Linter rules
└── README.md
```

## Comparison with Android Project

| Android | Flutter | Purpose |
|---------|---------|---------|
| `app/src/main/java|kotlin/` | `lib/` | Main source code |
| `app/src/main/res/` | `assets/` | Resources (images, fonts) |
| `build.gradle` | `pubspec.yaml` | Dependencies |
| `AndroidManifest.xml` | `android/app/src/main/AndroidManifest.xml` | App configuration |
| `app/src/test/` | `test/` | Unit tests |
| `app/src/androidTest/` | `integration_test/` | Integration tests |

## The `lib/` Directory (Your Main Workspace)

This is where you'll spend most of your time. Here's a recommended structure:

```
lib/
├── main.dart                    # App entry point
├── app.dart                     # Root widget
├── core/
│   ├── constants/
│   │   ├── app_colors.dart
│   │   ├── app_strings.dart
│   │   └── app_routes.dart
│   ├── config/
│   │   └── app_config.dart
│   ├── theme/
│   │   └── app_theme.dart
│   └── utils/
│       ├── helpers.dart
│       └── validators.dart
├── data/
│   ├── models/
│   │   ├── user.dart
│   │   └── product.dart
│   ├── repositories/
│   │   ├── user_repository.dart
│   │   └── product_repository.dart
│   ├── data_sources/
│   │   ├── remote/
│   │   │   └── api_service.dart
│   │   └── local/
│   │       └── database_service.dart
│   └── dto/                     # Data Transfer Objects
│       └── user_dto.dart
├── domain/
│   ├── entities/
│   │   └── user_entity.dart
│   ├── repositories/
│   │   └── user_repository.dart (interface)
│   └── use_cases/
│       └── get_user_use_case.dart
├── presentation/
│   ├── screens/
│   │   ├── home/
│   │   │   ├── home_screen.dart
│   │   │   ├── home_viewmodel.dart
│   │   │   └── widgets/
│   │   │       ├── home_header.dart
│   │   │       └── home_list.dart
│   │   ├── profile/
│   │   │   ├── profile_screen.dart
│   │   │   └── profile_viewmodel.dart
│   │   └── login/
│   │       ├── login_screen.dart
│   │       └── login_viewmodel.dart
│   └── widgets/
│       ├── common/
│       │   ├── custom_button.dart
│       │   ├── custom_text_field.dart
│       │   └── loading_indicator.dart
│       └── dialogs/
│           └── error_dialog.dart
└── services/
    ├── navigation_service.dart
    ├── analytics_service.dart
    └── notification_service.dart
```

## Comparing Architectures

### Android (Clean Architecture)
```
com.example.app/
├── data/
│   ├── model/
│   ├── repository/
│   └── source/
├── domain/
│   ├── entity/
│   ├── repository/
│   └── usecase/
└── presentation/
    ├── ui/
    ├── viewmodel/
    └── adapter/
```

### Flutter (Similar Clean Architecture)
```
lib/
├── data/
│   ├── models/
│   ├── repositories/
│   └── data_sources/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── use_cases/
└── presentation/
    ├── screens/
    ├── widgets/
    └── viewmodels/
```

**Key Takeaway**: The architecture concepts translate directly!

## pubspec.yaml - The Heart of Your Project

Think of `pubspec.yaml` as Flutter's `build.gradle`. Here's a comprehensive example:

```yaml
name: my_app
description: A Flutter application
publish_to: 'none'  # Prevent publishing to pub.dev
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'
  flutter: '>=3.10.0'

dependencies:
  flutter:
    sdk: flutter

  # UI
  cupertino_icons: ^1.0.6

  # State Management
  provider: ^6.1.1
  # OR riverpod: ^2.4.9
  # OR bloc: ^8.1.2
  # OR get: ^4.6.6

  # Network
  dio: ^5.4.0
  http: ^1.1.2
  retrofit: ^4.0.3

  # JSON Serialization
  json_annotation: ^4.8.1
  freezed_annotation: ^2.4.1

  # Local Storage
  shared_preferences: ^2.2.2
  hive: ^2.2.3
  sqflite: ^2.3.0

  # Navigation
  go_router: ^13.0.0

  # Dependency Injection
  get_it: ^7.6.4
  injectable: ^2.3.2

  # Utilities
  intl: ^0.18.1  # Internationalization
  logger: ^2.0.2
  equatable: ^2.0.5

dev_dependencies:
  flutter_test:
    sdk: flutter

  # Linting
  flutter_lints: ^3.0.1

  # Code Generation
  build_runner: ^2.4.7
  json_serializable: ^6.7.1
  freezed: ^2.4.6
  injectable_generator: ^2.4.1
  retrofit_generator: ^8.0.6

  # Testing
  mockito: ^5.4.4
  bloc_test: ^9.1.5

flutter:
  uses-material-design: true

  # Assets
  assets:
    - assets/images/
    - assets/icons/
    - assets/config/

  # Fonts
  fonts:
    - family: Roboto
      fonts:
        - asset: assets/fonts/Roboto-Regular.ttf
        - asset: assets/fonts/Roboto-Bold.ttf
          weight: 700
```

## Android-Specific Configuration

### AndroidManifest.xml

Located at `android/app/src/main/AndroidManifest.xml`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.CAMERA"/>

    <application
        android:label="My App"
        android:name="${applicationName}"
        android:icon="@mipmap/ic_launcher">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:launchMode="singleTop"
            android:theme="@style/LaunchTheme"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|smallestScreenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
            android:hardwareAccelerated="true"
            android:windowSoftInputMode="adjustResize">

            <!-- Deep linking -->
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:scheme="https"
                    android:host="example.com" />
            </intent-filter>

            <meta-data
                android:name="io.flutter.embedding.android.NormalTheme"
                android:resource="@style/NormalTheme" />

            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>

        <meta-data
            android:name="flutterEmbedding"
            android:value="2" />
    </application>
</manifest>
```

### build.gradle (app level)

Located at `android/app/build.gradle`:

```gradle
android {
    namespace "com.example.myapp"
    compileSdkVersion 34
    ndkVersion flutter.ndkVersion

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = '1.8'
    }

    defaultConfig {
        applicationId "com.example.myapp"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName

        // MultiDex support
        multiDexEnabled true
    }

    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            applicationIdSuffix ".debug"
            debuggable true
        }
    }

    flavorDimensions "environment"
    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
        }
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
        }
        production {
            dimension "environment"
        }
    }
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"

    // Add native Android dependencies here if needed
    implementation 'com.google.android.material:material:1.11.0'
}
```

## File Organization Best Practices

### 1. Feature-First Approach

```
lib/
├── features/
│   ├── authentication/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── home/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── profile/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── shared/
    ├── widgets/
    ├── utils/
    └── constants/
```

### 2. Layer-First Approach

```
lib/
├── data/
│   ├── authentication/
│   ├── home/
│   └── profile/
├── domain/
│   ├── authentication/
│   ├── home/
│   └── profile/
└── presentation/
    ├── authentication/
    ├── home/
    └── profile/
```

**Recommendation**: Feature-first is better for larger apps, layer-first for smaller ones.

## Entry Point: main.dart

**Android:**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

**Flutter:**
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const HomeScreen(),
    );
  }
}
```

## Environment-Specific Configuration

Create different entry points for different environments:

```
lib/
├── main_dev.dart
├── main_staging.dart
├── main_prod.dart
└── app.dart
```

**main_dev.dart:**
```dart
import 'package:flutter/material.dart';
import 'app.dart';
import 'core/config/app_config.dart';

void main() {
  AppConfig.initialize(
    apiBaseUrl: 'https://dev-api.example.com',
    environment: Environment.dev,
  );

  runApp(const MyApp());
}
```

**Run specific environment:**
```bash
flutter run -t lib/main_dev.dart
flutter build apk -t lib/main_prod.dart
```

## Asset Management

### Android
```
res/
├── drawable/
├── mipmap/
└── values/
```

### Flutter
```
assets/
├── images/
│   ├── logo.png
│   ├── 2.0x/
│   │   └── logo.png
│   └── 3.0x/
│       └── logo.png
└── fonts/
```

**pubspec.yaml:**
```yaml
flutter:
  assets:
    - assets/images/
    - assets/icons/

  fonts:
    - family: CustomFont
      fonts:
        - asset: assets/fonts/CustomFont-Regular.ttf
        - asset: assets/fonts/CustomFont-Bold.ttf
          weight: 700
```

**Usage:**
```dart
// Images
Image.asset('assets/images/logo.png')

// Fonts (applied in theme)
TextStyle(fontFamily: 'CustomFont', fontWeight: FontWeight.bold)
```

## Configuration Files

### analysis_options.yaml

Flutter's equivalent to Android's lint configuration:

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # Style
    prefer_const_constructors: true
    prefer_const_literals_to_create_immutables: true
    prefer_final_fields: true
    prefer_final_locals: true

    # Error prevention
    avoid_print: true
    avoid_unnecessary_containers: true
    sized_box_for_whitespace: true

    # Formatting
    always_put_required_named_parameters_first: true
    always_use_package_imports: true

    # Documentation
    public_member_api_docs: false  # Enable for libraries

analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"

  errors:
    invalid_annotation_target: ignore
```

## Platform Channels (Native Integration)

When you need native Android code:

**Android (MainActivity.kt):**
```kotlin
class MainActivity: FlutterActivity() {
    private val CHANNEL = "com.example.app/native"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setMethodCallHandler { call, result ->
                when (call.method) {
                    "getNativeData" -> {
                        result.success("Data from Android")
                    }
                    else -> result.notImplemented()
                }
            }
    }
}
```

**Flutter:**
```dart
import 'package:flutter/services.dart';

class NativeService {
  static const platform = MethodChannel('com.example.app/native');

  Future<String> getNativeData() async {
    try {
      final String result = await platform.invokeMethod('getNativeData');
      return result;
    } on PlatformException catch (e) {
      return "Failed: ${e.message}";
    }
  }
}
```

## Quick Reference

| Task | Android | Flutter |
|------|---------|---------|
| Add dependency | `build.gradle` | `pubspec.yaml` |
| Add permission | `AndroidManifest.xml` | `AndroidManifest.xml` (in android/) |
| Add image | `res/drawable/` | `assets/images/` + pubspec.yaml |
| App icon | `res/mipmap/` | Use flutter_launcher_icons package |
| Signing config | `build.gradle` | `android/app/build.gradle` |
| ProGuard | `proguard-rules.pro` | Same location in android/ |

## Next Steps

Now that you understand the project structure, let's learn about the app lifecycle in [Section 3: App Lifecycle](./03-app-lifecycle.md).
