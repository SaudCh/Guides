# EAS Build & Deployment Guide

This comprehensive guide covers everything about EAS (Expo Application Services) including build configuration, deployment, environment variables, and production builds.

## 📚 Table of Contents

1. [Understanding EAS](#understanding-eas)
2. [EAS Configuration (eas.json)](#eas-configuration-easjson)
3. [Build Profiles](#build-profiles)
4. [Environment Variables](#environment-variables)
5. [Build Commands](#build-commands)
6. [APK Production Builds](#apk-production-builds)
7. [iOS Production Builds](#ios-production-builds)
8. [Web Builds](#web-builds)
9. [Deployment Strategies](#deployment-strategies)
10. [Troubleshooting](#troubleshooting)

---

## Understanding EAS

### What is EAS?

EAS (Expo Application Services) is a cloud-based service that provides:

- **EAS Build**: Cloud-based builds for iOS, Android, and Web
- **EAS Submit**: Automated submission to app stores
- **EAS Update**: Over-the-air updates for your apps
- **EAS CLI**: Command-line tools for managing your projects

### EAS vs Expo CLI

| Feature       | Expo CLI                | EAS                        |
| ------------- | ----------------------- | -------------------------- |
| **Builds**    | Local only              | Cloud-based                |
| **iOS**       | Requires Xcode          | No Xcode needed            |
| **Android**   | Requires Android Studio | No Android Studio needed   |
| **APK**       | Local build only        | Cloud APK builds           |
| **App Store** | Manual submission       | Automated submission       |
| **Updates**   | Basic OTA               | Advanced OTA with channels |

---

## EAS Configuration (eas.json)

### Basic eas.json Structure

```json
{
  "cli": {
    "version": ">= 3.0.0"
  },
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

### Complete eas.json Example

```json
{
  "cli": {
    "version": ">= 3.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "apk"
      }
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "distribution": "store",
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "aab"
      }
    },
    "staging": {
      "distribution": "internal",
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "apk"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890",
        "appleTeamId": "ABCD123456"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "internal"
      }
    }
  }
}
```

---

## Build Profiles

### Development Profile

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "resourceClass": "m-medium",
        "simulator": true
      },
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleDebug"
      }
    }
  }
}
```

### Preview Profile

```json
{
  "build": {
    "preview": {
      "distribution": "internal",
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "apk"
      }
    }
  }
}
```

### Production Profile

```json
{
  "build": {
    "production": {
      "distribution": "store",
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "buildType": "aab"
      }
    }
  }
}
```

### Custom Profile

```json
{
  "build": {
    "custom": {
      "distribution": "internal",
      "ios": {
        "resourceClass": "m-large"
      },
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleRelease"
      }
    }
  }
}
```

---

## Environment Variables

### Setting Environment Variables

#### Method 1: .env Files

```bash
# .env.development
EXPO_PUBLIC_API_URL=https://dev-api.example.com
EXPO_PUBLIC_APP_VERSION=1.0.0-dev
EXPO_PUBLIC_DEBUG=true

# .env.production
EXPO_PUBLIC_API_URL=https://api.example.com
EXPO_PUBLIC_APP_VERSION=1.0.0
EXPO_PUBLIC_DEBUG=false
```

#### Method 2: EAS Build Environment Variables

```bash
# Set environment variables for builds
eas build:configure

# Set variables for specific profiles
eas secret:create --scope project --name API_URL --value "https://api.example.com"
eas secret:create --scope project --name APP_VERSION --value "1.0.0"
```

#### Method 3: Build Profile Environment Variables

```json
{
  "build": {
    "production": {
      "env": {
        "EXPO_PUBLIC_API_URL": "https://api.example.com",
        "EXPO_PUBLIC_APP_VERSION": "1.0.0",
        "EXPO_PUBLIC_DEBUG": "false"
      }
    }
  }
}
```

### Accessing Environment Variables

```javascript
// In your app code
import Constants from "expo-constants";

// Access public environment variables
const apiUrl =
  Constants.expoConfig.extra?.apiUrl || process.env.EXPO_PUBLIC_API_URL;
const appVersion =
  Constants.expoConfig.extra?.appVersion || process.env.EXPO_PUBLIC_APP_VERSION;
const isDebug =
  Constants.expoConfig.extra?.debug || process.env.EXPO_PUBLIC_DEBUG === "true";

console.log("API URL:", apiUrl);
console.log("App Version:", appVersion);
console.log("Debug Mode:", isDebug);
```

### Environment-Specific Configuration

```javascript
// config.js
const config = {
  development: {
    apiUrl: process.env.EXPO_PUBLIC_API_URL || "https://dev-api.example.com",
    appVersion: process.env.EXPO_PUBLIC_APP_VERSION || "1.0.0-dev",
    debug: process.env.EXPO_PUBLIC_DEBUG === "true" || true,
  },
  staging: {
    apiUrl:
      process.env.EXPO_PUBLIC_API_URL || "https://staging-api.example.com",
    appVersion: process.env.EXPO_PUBLIC_APP_VERSION || "1.0.0-staging",
    debug: process.env.EXPO_PUBLIC_DEBUG === "true" || true,
  },
  production: {
    apiUrl: process.env.EXPO_PUBLIC_API_URL || "https://api.example.com",
    appVersion: process.env.EXPO_PUBLIC_APP_VERSION || "1.0.0",
    debug: process.env.EXPO_PUBLIC_DEBUG === "true" || false,
  },
};

const environment = __DEV__ ? "development" : "production";
export default config[environment];
```

---

## Build Commands

### Basic Build Commands

```bash
# Build for all platforms
eas build --platform all

# Build for specific platform
eas build --platform ios
eas build --platform android
eas build --platform web

# Build with specific profile
eas build --profile production
eas build --profile development
eas build --profile preview
```

### Advanced Build Commands

```bash
# Build with specific channel
eas build --channel production

# Build with local changes
eas build --local

# Build with non-interactive mode
eas build --non-interactive

# Build with specific message
eas build --message "Version 1.0.0 release"

# Build with specific version
eas build --platform android --profile production --message "v1.0.0"
```

### Build Status and Management

```bash
# List all builds
eas build:list

# View specific build
eas build:view [BUILD_ID]

# Cancel build
eas build:cancel [BUILD_ID]

# Download build
eas build:download [BUILD_ID]

# View build logs
eas build:logs [BUILD_ID]
```

---

## APK Production Builds

### APK Build Configuration

```json
{
  "build": {
    "production-apk": {
      "distribution": "internal",
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleRelease"
      }
    }
  }
}
```

### Building APK

```bash
# Build APK for production
eas build --platform android --profile production-apk

# Build APK for preview
eas build --platform android --profile preview

# Build APK with specific version
eas build --platform android --profile production-apk --message "v1.0.0 APK"
```

### APK Build Options

```json
{
  "build": {
    "production-apk": {
      "distribution": "internal",
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleRelease",
        "keystore": {
          "keystorePath": "./keystore.jks",
          "keystorePassword": "your-keystore-password",
          "keyAlias": "your-key-alias",
          "keyPassword": "your-key-password"
        }
      }
    }
  }
}
```

### APK Signing

```bash
# Generate keystore
keytool -genkey -v -keystore keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias your-key-alias

# Build signed APK
eas build --platform android --profile production-apk
```

---

## iOS Production Builds

### iOS Build Configuration

```json
{
  "build": {
    "production-ios": {
      "distribution": "store",
      "ios": {
        "resourceClass": "m-medium",
        "buildConfiguration": "Release"
      }
    }
  }
}
```

### Building iOS App

```bash
# Build iOS app for App Store
eas build --platform ios --profile production-ios

# Build iOS app for TestFlight
eas build --platform ios --profile production-ios --message "TestFlight build"
```

### iOS Code Signing

```bash
# Configure iOS credentials
eas credentials:configure -p ios

# Manage certificates
eas credentials:list -p ios

# Create new certificate
eas credentials:create -p ios
```

---

## Web Builds

### Web Build Configuration

```json
{
  "build": {
    "production-web": {
      "distribution": "internal",
      "web": {
        "bundler": "metro"
      }
    }
  }
}
```

### Building Web App

```bash
# Build web app
eas build --platform web --profile production-web

# Build web app with specific output
eas build --platform web --profile production-web --output-dir ./dist
```

### Web Build Options

```json
{
  "build": {
    "production-web": {
      "distribution": "internal",
      "web": {
        "bundler": "metro",
        "outputDir": "./dist",
        "publicPath": "/"
      }
    }
  }
}
```

---

## Deployment Strategies

### Staged Deployment

```bash
# 1. Build for staging
eas build --platform all --profile staging

# 2. Test staging build
# Deploy to staging environment

# 3. Build for production
eas build --platform all --profile production

# 4. Deploy to production
eas submit --platform all --profile production
```

### Blue-Green Deployment

```bash
# 1. Build new version
eas build --platform all --profile production --message "v1.1.0"

# 2. Deploy to staging
eas submit --platform all --profile staging

# 3. Test staging
# Verify functionality

# 4. Deploy to production
eas submit --platform all --profile production
```

### Canary Deployment

```bash
# 1. Build canary version
eas build --platform all --profile canary --message "v1.1.0-canary"

# 2. Deploy to limited users
eas submit --platform all --profile canary

# 3. Monitor metrics
# Check for issues

# 4. Full deployment
eas submit --platform all --profile production
```

---

## Submit Commands

### Basic Submit Commands

```bash
# Submit to all platforms
eas submit --platform all

# Submit to specific platform
eas submit --platform ios
eas submit --platform android

# Submit with specific profile
eas submit --platform all --profile production
```

### Advanced Submit Commands

```bash
# Submit with specific build
eas submit --platform android --id [BUILD_ID]

# Submit with non-interactive mode
eas submit --platform all --non-interactive

# Submit with specific message
eas submit --platform all --message "Version 1.0.0 release"
```

### Submit Configuration

```json
{
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890",
        "appleTeamId": "ABCD123456"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "internal"
      }
    }
  }
}
```

---

## Troubleshooting

### Common Build Issues

#### 1. Build Timeout

```bash
# Increase build timeout
eas build --platform android --profile production --timeout 3600
```

#### 2. Memory Issues

```json
{
  "build": {
    "production": {
      "ios": {
        "resourceClass": "m-large"
      },
      "android": {
        "resourceClass": "m-large"
      }
    }
  }
}
```

#### 3. Build Failures

```bash
# Check build logs
eas build:logs [BUILD_ID]

# Retry build
eas build --platform android --profile production --retry

# Clear build cache
eas build --platform android --profile production --clear-cache
```

### Environment Variable Issues

#### 1. Missing Variables

```bash
# Check environment variables
eas secret:list

# Set missing variables
eas secret:create --scope project --name API_URL --value "https://api.example.com"
```

#### 2. Variable Not Accessible

```javascript
// Make sure to use EXPO_PUBLIC_ prefix
const apiUrl = process.env.EXPO_PUBLIC_API_URL; // ✅ Correct
const apiUrl = process.env.API_URL; // ❌ Wrong
```

### Build Optimization

#### 1. Reduce Build Time

```json
{
  "build": {
    "production": {
      "ios": {
        "resourceClass": "m-medium"
      },
      "android": {
        "resourceClass": "m-medium"
      }
    }
  }
}
```

#### 2. Optimize Bundle Size

```json
{
  "expo": {
    "assetBundlePatterns": ["assets/images/*", "assets/fonts/*"]
  }
}
```

---

## Best Practices

### Build Strategy

1. **Use Development Builds** for testing
2. **Use Preview Builds** for internal testing
3. **Use Production Builds** for app stores
4. **Test on Multiple Devices** before release
5. **Monitor Build Logs** for issues

### Environment Management

1. **Use .env Files** for local development
2. **Use EAS Secrets** for sensitive data
3. **Use Build Profiles** for different environments
4. **Validate Environment Variables** before builds
5. **Keep Secrets Secure** and rotate regularly

### Deployment Strategy

1. **Staged Deployment** for major releases
2. **Blue-Green Deployment** for zero downtime
3. **Canary Deployment** for risk mitigation
4. **Monitor Metrics** after deployment
5. **Have Rollback Plan** ready

---

## Quick Reference

### Essential Commands

```bash
# Configure EAS
eas build:configure

# Build for production
eas build --platform all --profile production

# Submit to app stores
eas submit --platform all --profile production

# List builds
eas build:list

# View build
eas build:view [BUILD_ID]

# Download build
eas build:download [BUILD_ID]
```

### Build Profiles

| Profile         | Platform | Distribution | Use Case            |
| --------------- | -------- | ------------ | ------------------- |
| **development** | All      | Internal     | Development testing |
| **preview**     | All      | Internal     | Internal testing    |
| **production**  | All      | Store        | App store release   |
| **staging**     | All      | Internal     | Staging environment |

### Environment Variables

| Variable                  | Description  | Example                   |
| ------------------------- | ------------ | ------------------------- |
| `EXPO_PUBLIC_API_URL`     | API endpoint | `https://api.example.com` |
| `EXPO_PUBLIC_APP_VERSION` | App version  | `1.0.0`                   |
| `EXPO_PUBLIC_DEBUG`       | Debug mode   | `true`/`false`            |

---

**Your EAS build and deployment setup is now complete! Build and deploy amazing apps! 🚀**
