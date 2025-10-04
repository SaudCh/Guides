# Expo app.json Configuration Guide

This comprehensive guide covers everything about configuring your Expo app using the `app.json` file, including all available options, best practices, and platform-specific settings.

## 📚 Table of Contents

1. [Understanding app.json](#understanding-appjson)
2. [Basic Configuration](#basic-configuration)
3. [Platform-Specific Settings](#platform-specific-settings)
4. [Advanced Configuration](#advanced-configuration)
5. [Environment Variables](#environment-variables)
6. [Build Configuration](#build-configuration)
7. [Publishing Configuration](#publishing-configuration)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

---

## Understanding app.json

### What is app.json?

The `app.json` file is the main configuration file for your Expo app. It defines:

- **App metadata** (name, version, description)
- **Platform settings** (iOS, Android, Web)
- **Build configuration** (icons, splash screens, permissions)
- **Publishing settings** (update channels, OTA configuration)
- **Development settings** (schemes, deep linking)

### File Structure

```json
{
  "expo": {
    // Main app configuration
  },
  "scripts": {
    // Custom scripts (optional)
  }
}
```

---

## Basic Configuration

### Essential Properties

```json
{
  "expo": {
    "name": "My Awesome App",
    "slug": "my-awesome-app",
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
      "supportsTablet": true
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      }
    },
    "web": {
      "favicon": "./assets/favicon.png"
    }
  }
}
```

### Property Explanations

| Property             | Description              | Required | Example                            |
| -------------------- | ------------------------ | -------- | ---------------------------------- |
| `name`               | Display name of your app | ✅       | `"My App"`                         |
| `slug`               | URL-friendly name        | ✅       | `"my-app"`                         |
| `version`            | App version              | ✅       | `"1.0.0"`                          |
| `orientation`        | Screen orientation       | ❌       | `"portrait"`, `"landscape"`        |
| `icon`               | App icon path            | ✅       | `"./assets/icon.png"`              |
| `userInterfaceStyle` | UI theme                 | ❌       | `"light"`, `"dark"`, `"automatic"` |

---

## Platform-Specific Settings

### iOS Configuration

```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.yourcompany.yourapp",
      "buildNumber": "1.0.0",
      "supportsTablet": true,
      "requireFullScreen": false,
      "infoPlist": {
        "CFBundleDisplayName": "My App",
        "NSCameraUsageDescription": "This app needs access to camera to take photos",
        "NSLocationWhenInUseUsageDescription": "This app needs location access for features"
      },
      "associatedDomains": ["applinks:yourapp.com"],
      "config": {
        "usesNonExemptEncryption": false
      }
    }
  }
}
```

### Android Configuration

```json
{
  "expo": {
    "android": {
      "package": "com.yourcompany.yourapp",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "permissions": [
        "CAMERA",
        "ACCESS_FINE_LOCATION",
        "WRITE_EXTERNAL_STORAGE"
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

### Web Configuration

```json
{
  "expo": {
    "web": {
      "favicon": "./assets/favicon.png",
      "name": "My Web App",
      "shortName": "MyApp",
      "lang": "en",
      "scope": "/",
      "themeColor": "#000000",
      "description": "My awesome web app",
      "dir": "ltr",
      "display": "standalone",
      "orientation": "portrait",
      "startUrl": "/",
      "backgroundColor": "#ffffff",
      "bundler": "metro"
    }
  }
}
```

---

## Advanced Configuration

### Updates Configuration

```json
{
  "expo": {
    "updates": {
      "enabled": true,
      "checkAutomatically": "ON_LOAD",
      "fallbackToCacheTimeout": 0,
      "url": "https://u.expo.dev/your-project-id"
    },
    "runtimeVersion": "1.0.0"
  }
}
```

### Notifications Configuration

```json
{
  "expo": {
    "notification": {
      "icon": "./assets/notification-icon.png",
      "color": "#ffffff",
      "androidMode": "default",
      "androidCollapsedTitle": "#{unread_notifications} new interactions"
    }
  }
}
```

### Deep Linking Configuration

```json
{
  "expo": {
    "scheme": "myapp",
    "linking": {
      "prefixes": ["myapp://", "https://yourapp.com"],
      "config": {
        "screens": {
          "Home": "",
          "Profile": "profile/:id",
          "Settings": "settings"
        }
      }
    }
  }
}
```

### Plugins Configuration

```json
{
  "expo": {
    "plugins": [
      "expo-camera",
      "expo-location",
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff"
        }
      ],
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

---

## Environment Variables

### Using Environment Variables

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "extra": {
      "apiUrl": "https://api.example.com",
      "environment": "production"
    }
  }
}
```

### Environment-Specific Configuration

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

### Accessing Environment Variables

```javascript
// In your app code
import Constants from "expo-constants";

const apiUrl = Constants.expoConfig.extra.apiUrl;
const environment = Constants.expoConfig.extra.environment;

console.log("API URL:", apiUrl);
console.log("Environment:", environment);
```

---

## Build Configuration

### EAS Build Configuration

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

### Build Profiles (eas.json)

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {
      "distribution": "store"
    }
  },
  "submit": {
    "production": {}
  }
}
```

### App Store Configuration

```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.yourcompany.yourapp",
      "buildNumber": "1.0.0",
      "config": {
        "usesNonExemptEncryption": false
      },
      "infoPlist": {
        "CFBundleDisplayName": "My App",
        "NSCameraUsageDescription": "This app needs access to camera"
      }
    },
    "android": {
      "package": "com.yourcompany.yourapp",
      "versionCode": 1,
      "permissions": ["CAMERA", "ACCESS_FINE_LOCATION"]
    }
  }
}
```

---

## Publishing Configuration

### Update Channels

```json
{
  "expo": {
    "updates": {
      "enabled": true,
      "checkAutomatically": "ON_LOAD",
      "fallbackToCacheTimeout": 0
    },
    "runtimeVersion": "1.0.0"
  }
}
```

### Publishing Settings

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "privacy": "public"
  }
}
```

---

## Best Practices

### File Organization

```
assets/
├── icon.png              # 1024x1024 app icon
├── splash.png            # 1284x2778 splash screen
├── adaptive-icon.png     # 1024x1024 Android adaptive icon
├── favicon.png           # 48x48 web favicon
└── notification-icon.png # 96x96 notification icon
```

### Icon Requirements

| Platform          | Size      | Format | Description                |
| ----------------- | --------- | ------ | -------------------------- |
| **iOS**           | 1024x1024 | PNG    | App icon (no transparency) |
| **Android**       | 1024x1024 | PNG    | Adaptive icon foreground   |
| **Web**           | 48x48     | PNG    | Favicon                    |
| **Notifications** | 96x96     | PNG    | Notification icon          |

### Splash Screen Requirements

| Platform    | Size      | Format | Description       |
| ----------- | --------- | ------ | ----------------- |
| **iOS**     | 1284x2778 | PNG    | iPhone 12 Pro Max |
| **Android** | 1284x2778 | PNG    | Standard splash   |
| **Web**     | 1284x2778 | PNG    | Web splash screen |

### Version Management

```json
{
  "expo": {
    "version": "1.0.0",
    "runtimeVersion": "1.0.0",
    "ios": {
      "buildNumber": "1.0.0"
    },
    "android": {
      "versionCode": 1
    }
  }
}
```

### Security Best Practices

```json
{
  "expo": {
    "ios": {
      "config": {
        "usesNonExemptEncryption": false
      },
      "infoPlist": {
        "NSAppTransportSecurity": {
          "NSAllowsArbitraryLoads": false
        }
      }
    },
    "android": {
      "permissions": ["CAMERA", "ACCESS_FINE_LOCATION"]
    }
  }
}
```

---

## Troubleshooting

### Common Issues

#### 1. Invalid Bundle Identifier

```json
// ❌ Wrong
{
  "expo": {
    "ios": {
      "bundleIdentifier": "my-app"
    }
  }
}

// ✅ Correct
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.yourcompany.yourapp"
    }
  }
}
```

#### 2. Missing Required Properties

```json
// ❌ Missing required properties
{
  "expo": {
    "name": "My App"
  }
}

// ✅ Complete configuration
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "icon": "./assets/icon.png"
  }
}
```

#### 3. Invalid Icon Paths

```json
// ❌ Wrong path
{
  "expo": {
    "icon": "assets/icon.png"
  }
}

// ✅ Correct path
{
  "expo": {
    "icon": "./assets/icon.png"
  }
}
```

### Validation Commands

```bash
# Validate app.json
npx expo config

# Check for errors
npx expo doctor

# Validate specific platform
npx expo config --platform ios
npx expo config --platform android
npx expo config --platform web
```

### Debug Configuration

```bash
# Show current configuration
npx expo config --type public

# Show private configuration
npx expo config --type private

# Show specific platform config
npx expo config --platform ios --type public
```

---

## Complete Example

### Full app.json Example

```json
{
  "expo": {
    "name": "My Awesome App",
    "slug": "my-awesome-app",
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
      "bundleIdentifier": "com.yourcompany.yourapp",
      "buildNumber": "1.0.0",
      "supportsTablet": true,
      "requireFullScreen": false,
      "infoPlist": {
        "CFBundleDisplayName": "My App",
        "NSCameraUsageDescription": "This app needs access to camera to take photos",
        "NSLocationWhenInUseUsageDescription": "This app needs location access for features"
      }
    },
    "android": {
      "package": "com.yourcompany.yourapp",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "permissions": ["CAMERA", "ACCESS_FINE_LOCATION"]
    },
    "web": {
      "favicon": "./assets/favicon.png",
      "name": "My Web App",
      "shortName": "MyApp",
      "themeColor": "#000000",
      "backgroundColor": "#ffffff"
    },
    "updates": {
      "enabled": true,
      "checkAutomatically": "ON_LOAD",
      "fallbackToCacheTimeout": 0
    },
    "runtimeVersion": "1.0.0",
    "scheme": "myapp",
    "plugins": ["expo-camera", "expo-location"],
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

---

## Quick Reference

### Essential Properties Checklist

- [ ] `name` - App display name
- [ ] `slug` - URL-friendly name
- [ ] `version` - App version
- [ ] `icon` - App icon path
- [ ] `splash` - Splash screen configuration
- [ ] `ios.bundleIdentifier` - iOS bundle ID
- [ ] `android.package` - Android package name
- [ ] `web.favicon` - Web favicon

### Platform-Specific Checklist

#### iOS

- [ ] `bundleIdentifier` - Unique bundle ID
- [ ] `buildNumber` - Build number
- [ ] `supportsTablet` - Tablet support
- [ ] `infoPlist` - iOS-specific settings

#### Android

- [ ] `package` - Package name
- [ ] `versionCode` - Version code
- [ ] `adaptiveIcon` - Adaptive icon
- [ ] `permissions` - Required permissions

#### Web

- [ ] `favicon` - Web favicon
- [ ] `name` - Web app name
- [ ] `themeColor` - Theme color
- [ ] `backgroundColor` - Background color

---

**Your app.json configuration is now complete! Build amazing Expo apps with proper configuration! 🚀**
