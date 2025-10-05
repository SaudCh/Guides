# Expo Prebuild Guide

This comprehensive guide covers everything about Expo prebuild, including what it is, how to use it, configuration options, and best practices for managing native code in Expo projects.

## 📚 Table of Contents

1. [Understanding Expo Prebuild](#understanding-expo-prebuild)
2. [Prebuild Commands](#prebuild-commands)
3. [Prebuild Configuration](#prebuild-configuration)
4. [Native Code Management](#native-code-management)
5. [Prebuild Workflows](#prebuild-workflows)
6. [Custom Native Code](#custom-native-code)
7. [Prebuild vs Development Builds](#prebuild-vs-development-builds)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## Understanding Expo Prebuild

### What is Expo Prebuild?

Expo prebuild is a tool that generates native iOS and Android projects from your Expo configuration. It allows you to:

- **Generate native projects** from your Expo app configuration
- **Customize native code** while maintaining Expo's managed workflow
- **Add custom native modules** and configurations
- **Maintain full control** over native iOS and Android code
- **Use Expo CLI** for development while having native project access

### When to Use Prebuild

#### Use Prebuild When:

- **Adding custom native modules** not available in Expo SDK
- **Customizing native configurations** (Info.plist, AndroidManifest.xml)
- **Adding custom native code** or libraries
- **Integrating with existing native projects**
- **Using third-party native libraries**
- **Customizing build configurations**

#### Don't Use Prebuild When:

- **Using only Expo SDK modules** (use managed workflow)
- **Simple apps** without custom native requirements
- **Quick prototyping** and development
- **Learning Expo** for the first time

### Prebuild vs Managed Workflow

| Feature                   | Managed Workflow | Prebuild   |
| ------------------------- | ---------------- | ---------- |
| **Native Code Access**    | ❌ No            | ✅ Full    |
| **Custom Native Modules** | ❌ Limited       | ✅ Full    |
| **Build Configuration**   | ❌ Limited       | ✅ Full    |
| **Development Speed**     | ⚡ Fast          | 🐌 Slower  |
| **Complexity**            | 🟢 Simple        | 🟡 Complex |
| **Expo Go Compatible**    | ✅ Yes           | ❌ No      |
| **Custom Configurations** | ❌ Limited       | ✅ Full    |

---

## Prebuild Commands

### Basic Prebuild Commands

#### Generate Native Projects

```bash
# Generate native projects for all platforms
npx expo prebuild

# Generate for specific platform
npx expo prebuild --platform ios
npx expo prebuild --platform android

# Generate with clean slate
npx expo prebuild --clean
```

#### Prebuild with Options

```bash
# Generate with specific template
npx expo prebuild --template bare-minimum

# Generate with custom output directory
npx expo prebuild --output-dir ./native

# Generate with non-interactive mode
npx expo prebuild --non-interactive
```

#### Prebuild Management

```bash
# Check prebuild status
npx expo prebuild --check

# Clean prebuild files
npx expo prebuild --clean

# Regenerate native projects
npx expo prebuild --regenerate
```

### Advanced Prebuild Commands

#### Prebuild with Configuration

```bash
# Prebuild with specific configuration
npx expo prebuild --config app.config.js

# Prebuild with environment variables
npx expo prebuild --env production

# Prebuild with custom plugins
npx expo prebuild --plugins expo-camera,expo-location
```

#### Prebuild Debugging

```bash
# Prebuild with verbose output
npx expo prebuild --verbose

# Prebuild with debug mode
npx expo prebuild --debug

# Prebuild with dry run
npx expo prebuild --dry-run
```

---

## Prebuild Configuration

### Basic Prebuild Configuration

#### app.json Configuration

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.yourapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.yourcompany.yourapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": ["expo-camera", "expo-location"]
  }
}
```

#### app.config.js Configuration

```javascript
export default {
  expo: {
    name: "My App",
    slug: "my-app",
    version: "1.0.0",
    orientation: "portrait",
    icon: "./assets/icon.png",
    userInterfaceStyle: "light",
    splash: {
      image: "./assets/splash.png",
      resizeMode: "contain",
      backgroundColor: "#ffffff",
    },
    assetBundlePatterns: ["**/*"],
    ios: {
      supportsTablet: true,
      bundleIdentifier: "com.yourcompany.yourapp",
    },
    android: {
      adaptiveIcon: {
        foregroundImage: "./assets/adaptive-icon.png",
        backgroundColor: "#ffffff",
      },
      package: "com.yourcompany.yourapp",
    },
    web: {
      favicon: "./assets/favicon.png",
    },
    plugins: ["expo-camera", "expo-location"],
  },
};
```

### Advanced Prebuild Configuration

#### Custom Native Configurations

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.yourapp",
      "infoPlist": {
        "NSCameraUsageDescription": "This app needs access to camera to take photos",
        "NSLocationWhenInUseUsageDescription": "This app needs location access for features",
        "CFBundleDisplayName": "My App"
      },
      "config": {
        "usesNonExemptEncryption": false
      }
    },
    "android": {
      "package": "com.yourcompany.yourapp",
      "permissions": [
        "CAMERA",
        "ACCESS_FINE_LOCATION",
        "WRITE_EXTERNAL_STORAGE"
      ],
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      }
    },
    "plugins": [
      "expo-camera",
      "expo-location",
      [
        "expo-build-properties",
        {
          "ios": {
            "deploymentTarget": "11.0"
          },
          "android": {
            "compileSdkVersion": 31,
            "targetSdkVersion": 31,
            "buildToolsVersion": "31.0.0"
          }
        }
      ]
    ]
  }
}
```

#### Environment-Specific Configuration

```javascript
const IS_DEV = process.env.APP_VARIANT === "development";

export default {
  expo: {
    name: IS_DEV ? "My App (Dev)" : "My App",
    slug: "my-app",
    version: "1.0.0",
    ios: {
      bundleIdentifier: IS_DEV
        ? "com.yourcompany.yourapp.dev"
        : "com.yourcompany.yourapp",
    },
    android: {
      package: IS_DEV
        ? "com.yourcompany.yourapp.dev"
        : "com.yourcompany.yourapp",
    },
    plugins: ["expo-camera", "expo-location"],
  },
};
```

---

## Native Code Management

### Generated Project Structure

After running prebuild, you'll have:

```
my-app/
├── android/                 # Android native project
│   ├── app/
│   ├── gradle/
│   ├── build.gradle
│   └── settings.gradle
├── ios/                     # iOS native project
│   ├── MyApp/
│   ├── MyApp.xcodeproj/
│   └── Podfile
├── app.json                 # Expo configuration
├── package.json             # Dependencies
└── App.js                   # Main app component
```

### Customizing Native Code

#### iOS Customizations

```bash
# Navigate to iOS project
cd ios

# Open in Xcode
open MyApp.xcodeproj

# Customize Info.plist
# Add custom native code
# Configure build settings
```

#### Android Customizations

```bash
# Navigate to Android project
cd android

# Open in Android Studio
# Customize AndroidManifest.xml
# Add custom native code
# Configure build.gradle
```

### Adding Custom Native Modules

#### iOS Native Module

```objc
// ios/MyApp/MyCustomModule.m
#import "MyCustomModule.h"
#import <React/RCTLog.h>

@implementation MyCustomModule

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(showAlert:(NSString *)message)
{
  dispatch_async(dispatch_get_main_queue(), ^{
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Alert" message:message preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil];
    [alert addAction:okAction];

    UIViewController *rootViewController = [UIApplication sharedApplication].delegate.window.rootViewController;
    [rootViewController presentViewController:alert animated:YES completion:nil];
  });
}

@end
```

#### Android Native Module

```java
// android/app/src/main/java/com/yourapp/MyCustomModule.java
package com.yourapp;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.UiThreadUtil;

public class MyCustomModule extends ReactContextBaseJavaModule {

    public MyCustomModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    @Override
    public String getName() {
        return "MyCustomModule";
    }

    @ReactMethod
    public void showAlert(String message) {
        UiThreadUtil.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // Show alert dialog
            }
        });
    }
}
```

---

## Prebuild Workflows

### Development Workflow

#### 1. Initial Setup

```bash
# Create new Expo project
npx create-expo-app MyApp --template bare-minimum

# Navigate to project
cd MyApp

# Install dependencies
npm install
```

#### 2. Configure Project

```bash
# Configure app.json or app.config.js
# Add plugins and native configurations

# Run prebuild
npx expo prebuild
```

#### 3. Development

```bash
# Start development server
npx expo start --dev-client

# Make changes to native code
# Test on device or simulator
```

#### 4. Building

```bash
# Build for development
eas build --platform ios --profile development

# Build for production
eas build --platform all --profile production
```

### Production Workflow

#### 1. Prebuild for Production

```bash
# Clean prebuild
npx expo prebuild --clean

# Prebuild with production config
npx expo prebuild --env production
```

#### 2. Build Production App

```bash
# Build for App Store
eas build --platform ios --profile production

# Build for Play Store
eas build --platform android --profile production
```

#### 3. Submit to Stores

```bash
# Submit to App Store
eas submit --platform ios --profile production

# Submit to Play Store
eas submit --platform android --profile production
```

---

## Custom Native Code

### Adding Custom Native Libraries

#### iOS Custom Library

```bash
# Add to Podfile
cd ios
echo 'pod "MyCustomLibrary", "~> 1.0.0"' >> Podfile

# Install pods
pod install
```

#### Android Custom Library

```bash
# Add to build.gradle
cd android/app
# Add to dependencies in build.gradle
implementation 'com.example:my-custom-library:1.0.0'
```

### Custom Native Configurations

#### iOS Custom Configuration

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "This app needs access to camera",
        "NSLocationWhenInUseUsageDescription": "This app needs location access",
        "CFBundleDisplayName": "My App",
        "UIBackgroundModes": ["background-fetch", "background-processing"]
      },
      "config": {
        "usesNonExemptEncryption": false
      }
    }
  }
}
```

#### Android Custom Configuration

```json
{
  "expo": {
    "android": {
      "permissions": [
        "CAMERA",
        "ACCESS_FINE_LOCATION",
        "WRITE_EXTERNAL_STORAGE",
        "RECORD_AUDIO"
      ],
      "intentFilters": [
        {
          "action": "VIEW",
          "data": [
            {
              "scheme": "https",
              "host": "yourapp.com"
            }
          ],
          "category": ["BROWSABLE", "DEFAULT"]
        }
      ]
    }
  }
}
```

---

## Prebuild vs Development Builds

### Comparison

| Feature                   | Prebuild         | Development Build |
| ------------------------- | ---------------- | ----------------- |
| **Native Code Access**    | ✅ Full          | ✅ Full           |
| **Custom Native Modules** | ✅ Full          | ✅ Full           |
| **Build Time**            | ⏱️ 10-15 minutes | ⏱️ 10-15 minutes  |
| **Development Speed**     | 🐌 Slower        | 🐌 Slower         |
| **Expo Go Compatible**    | ❌ No            | ❌ No             |
| **Custom Configurations** | ✅ Full          | ✅ Full           |
| **Native Project Access** | ✅ Yes           | ✅ Yes            |
| **Complexity**            | 🟡 Complex       | 🟡 Complex        |

### When to Use Each

#### Use Prebuild When:

- **Starting from scratch** with custom native requirements
- **Need full control** over native project structure
- **Integrating with existing** native projects
- **Custom build configurations** are required

#### Use Development Builds When:

- **Already have** a managed Expo project
- **Need to add** custom native modules
- **Want to maintain** Expo's managed workflow
- **Quick addition** of native functionality

---

## Troubleshooting

### Common Prebuild Issues

#### 1. Prebuild Fails

```bash
# Check for errors
npx expo prebuild --verbose

# Clean and retry
npx expo prebuild --clean

# Check configuration
npx expo prebuild --check
```

#### 2. Native Project Issues

```bash
# Regenerate native projects
npx expo prebuild --regenerate

# Clean and rebuild
npx expo prebuild --clean
npx expo prebuild
```

#### 3. Build Issues

```bash
# Check build configuration
eas build:configure

# Clear build cache
eas build --platform all --clear-cache

# Check build logs
eas build:logs [BUILD_ID]
```

#### 4. Plugin Issues

```bash
# Check plugin compatibility
npx expo install --check

# Update plugins
npx expo install --fix

# Remove problematic plugins
npm uninstall problematic-plugin
```

### Debug Commands

```bash
# Check prebuild status
npx expo prebuild --check

# Verbose prebuild output
npx expo prebuild --verbose

# Debug mode
npx expo prebuild --debug

# Dry run
npx expo prebuild --dry-run
```

---

## Best Practices

### Prebuild Best Practices

#### 1. Use Version Control

```bash
# Commit native projects
git add ios/ android/
git commit -m "Add native projects from prebuild"

# Use .gitignore for generated files
echo "ios/build/" >> .gitignore
echo "android/app/build/" >> .gitignore
```

#### 2. Configuration Management

```javascript
// Use app.config.js for dynamic configuration
const IS_DEV = process.env.APP_VARIANT === "development";

export default {
  expo: {
    name: IS_DEV ? "My App (Dev)" : "My App",
    // ... rest of configuration
  },
};
```

#### 3. Plugin Management

```json
{
  "expo": {
    "plugins": [
      "expo-camera",
      "expo-location",
      [
        "expo-build-properties",
        {
          "ios": {
            "deploymentTarget": "11.0"
          },
          "android": {
            "compileSdkVersion": 31
          }
        }
      ]
    ]
  }
}
```

#### 4. Native Code Organization

```
ios/
├── MyApp/
│   ├── AppDelegate.m
│   ├── Info.plist
│   └── CustomModules/
│       ├── MyCustomModule.h
│       └── MyCustomModule.m
└── Podfile

android/
├── app/
│   ├── src/main/java/com/yourapp/
│   │   ├── MainActivity.java
│   │   └── CustomModules/
│   │       └── MyCustomModule.java
│   └── build.gradle
└── build.gradle
```

### Development Workflow

#### 1. Start with Managed Workflow

```bash
# Create managed project first
npx create-expo-app MyApp

# Develop with Expo Go
npx expo start
```

#### 2. Add Prebuild When Needed

```bash
# When you need native modules
npx expo prebuild

# Continue development
npx expo start --dev-client
```

#### 3. Maintain Both Workflows

```bash
# Keep managed workflow for simple changes
npx expo start

# Use prebuild for native changes
npx expo start --dev-client
```

---

## Quick Reference

### Essential Commands

```bash
# Generate native projects
npx expo prebuild

# Clean and regenerate
npx expo prebuild --clean

# Check prebuild status
npx expo prebuild --check

# Start development
npx expo start --dev-client

# Build for production
eas build --platform all --profile production
```

### Configuration Files

| File                                       | Purpose               | When to Edit   |
| ------------------------------------------ | --------------------- | -------------- |
| `app.json`                                 | Basic configuration   | Always         |
| `app.config.js`                            | Dynamic configuration | When needed    |
| `ios/Info.plist`                           | iOS native config     | After prebuild |
| `android/app/src/main/AndroidManifest.xml` | Android native config | After prebuild |

### Workflow Comparison

| Workflow              | Use Case             | Commands                                            |
| --------------------- | -------------------- | --------------------------------------------------- |
| **Managed**           | Simple apps, Expo Go | `npx expo start`                                    |
| **Prebuild**          | Custom native code   | `npx expo prebuild` + `npx expo start --dev-client` |
| **Development Build** | Add native modules   | `eas build --profile development`                   |

---

**Your Expo prebuild setup is now complete! Build powerful native apps with full control! 🚀**
