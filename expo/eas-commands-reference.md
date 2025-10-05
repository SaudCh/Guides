# EAS Commands Reference Guide

This comprehensive guide covers all EAS (Expo Application Services) commands, their usage, options, and detailed examples for managing your Expo projects.

## 📚 Table of Contents

1. [EAS CLI Installation & Setup](#eas-cli-installation--setup)
2. [EAS Build Commands](#eas-build-commands)
3. [Development Builds](#development-builds)
4. [EAS Submit Commands](#eas-submit-commands)
5. [EAS Update Commands](#eas-update-commands)
6. [EAS Credentials Commands](#eas-credentials-commands)
7. [EAS Project Commands](#eas-project-commands)
8. [EAS Configuration Commands](#eas-configuration-commands)
9. [EAS Webhook Commands](#eas-webhook-commands)
10. [EAS Device Commands](#eas-device-commands)
11. [Advanced EAS Commands](#advanced-eas-commands)
12. [EAS Command Options](#eas-command-options)
13. [Troubleshooting](#troubleshooting)

---

## EAS CLI Installation & Setup

### Installation

```bash
# Install EAS CLI globally
npm install -g @expo/eas-cli

# Verify installation
eas --version

# Update EAS CLI
npm install -g @expo/eas-cli@latest
```

### Authentication

```bash
# Login to EAS
eas login

# Logout from EAS
eas logout

# Check authentication status
eas whoami

# Login with specific account
eas login --username your-username
```

### Project Setup

```bash
# Initialize EAS in your project
eas build:configure

# Check project configuration
eas project:info

# Link to existing project
eas project:link
```

---

## EAS Build Commands

### Basic Build Commands

#### Build for All Platforms

```bash
# Build for all platforms (iOS, Android, Web)
eas build --platform all

# Build with specific profile
eas build --platform all --profile production

# Build with message
eas build --platform all --message "Version 1.0.0 release"
```

#### Build for Specific Platforms

```bash
# Build for iOS only
eas build --platform ios

# Build for Android only
eas build --platform android

# Build for Web only
eas build --platform web
```

#### Build with Profiles

```bash
# Build with development profile
eas build --platform all --profile development

# Build with preview profile
eas build --platform all --profile preview

# Build with production profile
eas build --platform all --profile production

# Build with custom profile
eas build --platform all --profile staging
```

### Advanced Build Commands

#### Build with Options

```bash
# Build with non-interactive mode
eas build --platform all --non-interactive

# Build with local changes
eas build --platform all --local

# Build with clear cache
eas build --platform all --clear-cache

# Build with specific channel
eas build --platform all --channel production
```

#### Build Management

```bash
# List all builds
eas build:list

# List builds for specific platform
eas build:list --platform ios

# List builds with limit
eas build:list --limit 10

# List builds with status filter
eas build:list --status finished
```

#### Build Information

```bash
# View specific build
eas build:view [BUILD_ID]

# View build with logs
eas build:view [BUILD_ID] --logs

# Download build
eas build:download [BUILD_ID]

# Download build to specific directory
eas build:download [BUILD_ID] --output-dir ./builds
```

#### Build Cancellation

```bash
# Cancel specific build
eas build:cancel [BUILD_ID]

# Cancel all pending builds
eas build:cancel --all
```

### Development Builds

#### What are Development Builds?

Development builds are custom builds of your app that include the Expo development client, allowing you to:

- **Test native code changes** without publishing
- **Use custom native modules** not available in Expo Go
- **Debug native functionality** with full access to device APIs
- **Test on physical devices** with development tools

#### Create Development Build

```bash
# Create development build for iOS
eas build --platform ios --profile development

# Create development build for Android
eas build --platform android --profile development

# Create development build for all platforms
eas build --platform all --profile development
```

#### Development Build Configuration

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

#### Install Development Build

```bash
# Download development build
eas build:download [BUILD_ID]

# Install on iOS device
# Use Xcode or Apple Configurator 2

# Install on Android device
adb install path/to/your-app.apk
```

#### Run Development Build

```bash
# Start development server
npx expo start --dev-client

# Start with specific platform
npx expo start --dev-client --ios
npx expo start --dev-client --android

# Start with tunnel
npx expo start --dev-client --tunnel
```

#### Development Build Features

- **Hot Reloading**: Instant code changes
- **Debug Menu**: Access to debugging tools
- **Native Modules**: Full access to native functionality
- **Custom Configurations**: Use custom app.json settings
- **Development Tools**: React Native debugger, Flipper, etc.

#### Advanced Development Build Configuration

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "resourceClass": "m-medium",
        "simulator": true,
        "buildConfiguration": "Debug",
        "scheme": "MyApp-Development"
      },
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleDebug",
        "buildVariant": "debug"
      },
      "env": {
        "EXPO_PUBLIC_DEBUG": "true",
        "EXPO_PUBLIC_API_URL": "https://dev-api.example.com"
      }
    }
  }
}
```

#### Development Build vs Expo Go

| Feature            | Development Build | Expo Go      |
| ------------------ | ----------------- | ------------ |
| **Native Modules** | ✅ Full access    | ❌ Limited   |
| **Custom Config**  | ✅ Full control   | ❌ Limited   |
| **Debug Tools**    | ✅ Full access    | ❌ Limited   |
| **Hot Reloading**  | ✅ Yes            | ✅ Yes       |
| **Build Time**     | ⏱️ 10-15 minutes  | ⚡ Instant   |
| **Installation**   | 📱 Manual install | 📱 App store |

#### Development Build Workflow

```bash
# 1. Create development build
eas build --platform ios --profile development

# 2. Download and install build
eas build:download [BUILD_ID]

# 3. Start development server
npx expo start --dev-client

# 4. Open development build on device
# Scan QR code or enter URL manually

# 5. Develop with full native access
# Make changes, test native modules, debug
```

#### Development Build Troubleshooting

```bash
# Development build not connecting
npx expo start --dev-client --clear

# Development build crashes
npx expo start --dev-client --reset-cache

# Development build outdated
eas build --platform ios --profile development --clear-cache

# Check development build status
eas build:list --platform ios --profile development
```

#### Development Build Best Practices

1. **Use for Native Development**

   ```bash
   # When you need native modules
   eas build --platform ios --profile development
   ```

2. **Test on Physical Devices**

   ```bash
   # Always test on real devices
   npx expo start --dev-client
   ```

3. **Use Environment Variables**

   ```json
   {
     "build": {
       "development": {
         "env": {
           "EXPO_PUBLIC_DEBUG": "true"
         }
       }
     }
   }
   ```

4. **Keep Builds Updated**
   ```bash
   # Rebuild when adding native modules
   eas build --platform all --profile development
   ```

### Build Configuration

#### Build Profiles (eas.json)

```json
{
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
    }
  }
}
```

---

## EAS Submit Commands

### Basic Submit Commands

#### Submit to App Stores

```bash
# Submit to all platforms
eas submit --platform all

# Submit to iOS App Store
eas submit --platform ios

# Submit to Google Play Store
eas submit --platform android
```

#### Submit with Profiles

```bash
# Submit with production profile
eas submit --platform all --profile production

# Submit with specific profile
eas submit --platform ios --profile production
```

### Advanced Submit Commands

#### Submit with Options

```bash
# Submit with non-interactive mode
eas submit --platform all --non-interactive

# Submit with specific build
eas submit --platform ios --id [BUILD_ID]

# Submit with message
eas submit --platform all --message "Version 1.0.0 submission"
```

#### Submit Management

```bash
# List all submissions
eas submit:list

# List submissions for specific platform
eas submit:list --platform ios

# View specific submission
eas submit:view [SUBMISSION_ID]

# View submission with logs
eas submit:view [SUBMISSION_ID] --logs
```

#### Submit Configuration

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

## EAS Update Commands

### Basic Update Commands

#### Publish Updates

```bash
# Publish update to default branch
eas update

# Publish update with message
eas update --message "Bug fixes and improvements"

# Publish update to specific branch
eas update --branch production

# Publish update to multiple branches
eas update --branch production --branch staging
```

#### Update with Options

```bash
# Publish update with non-interactive mode
eas update --non-interactive

# Publish update with local changes
eas update --local

# Publish update with clear cache
eas update --clear-cache
```

### Update Management

#### List Updates

```bash
# List all updates
eas update:list

# List updates for specific branch
eas update:list --branch production

# List updates with limit
eas update:list --limit 10

# List updates with status filter
eas update:list --status published
```

#### Update Information

```bash
# View specific update
eas update:view [UPDATE_ID]

# View update with logs
eas update:view [UPDATE_ID] --logs

# View update with details
eas update:view [UPDATE_ID] --details
```

#### Update Configuration

```bash
# Configure EAS Update
eas update:configure

# Check update configuration
eas update:status

# View update channels
eas update:channels
```

### Update Rollback

```bash
# Rollback to previous update
eas update:rollback --branch production

# Rollback to specific update
eas update:rollback --branch production --update-id [UPDATE_ID]

# Rollback with message
eas update:rollback --branch production --message "Emergency rollback"
```

---

## EAS Credentials Commands

### Credentials Management

#### List Credentials

```bash
# List all credentials
eas credentials:list

# List credentials for iOS
eas credentials:list --platform ios

# List credentials for Android
eas credentials:list --platform android
```

#### Configure Credentials

```bash
# Configure credentials for iOS
eas credentials:configure --platform ios

# Configure credentials for Android
eas credentials:configure --platform android

# Configure credentials for all platforms
eas credentials:configure --platform all
```

#### Create Credentials

```bash
# Create iOS distribution certificate
eas credentials:create --platform ios --type distribution

# Create iOS push notification key
eas credentials:create --platform ios --type push-notification

# Create Android keystore
eas credentials:create --platform android --type keystore
```

#### Delete Credentials

```bash
# Delete specific credential
eas credentials:delete [CREDENTIAL_ID]

# Delete credentials for platform
eas credentials:delete --platform ios --type distribution
```

---

## EAS Project Commands

### Project Management

#### Project Information

```bash
# View project information
eas project:info

# View project with details
eas project:info --details

# View project configuration
eas project:info --config
```

#### Project Linking

```bash
# Link to existing project
eas project:link

# Link with specific project ID
eas project:link --id [PROJECT_ID]

# Unlink from project
eas project:unlink
```

#### Project Creation

```bash
# Create new project
eas project:create

# Create project with name
eas project:create --name "My Awesome App"

# Create project with description
eas project:create --description "A comprehensive mobile app"
```

---

## EAS Configuration Commands

### Configuration Management

#### Build Configuration

```bash
# Configure build settings
eas build:configure

# Configure build with profile
eas build:configure --profile production

# Configure build for platform
eas build:configure --platform ios
```

#### Update Configuration

```bash
# Configure update settings
eas update:configure

# Configure update with branch
eas update:configure --branch production

# Configure update with channel
eas update:configure --channel production
```

#### Submit Configuration

```bash
# Configure submit settings
eas submit:configure

# Configure submit for platform
eas submit:configure --platform ios

# Configure submit with profile
eas submit:configure --profile production
```

---

## EAS Webhook Commands

### Webhook Management

#### List Webhooks

```bash
# List all webhooks
eas webhook:list

# List webhooks for project
eas webhook:list --project-id [PROJECT_ID]
```

#### Create Webhook

```bash
# Create webhook for build events
eas webhook:create --event build

# Create webhook for submit events
eas webhook:create --event submit

# Create webhook with URL
eas webhook:create --event build --url https://your-webhook-url.com
```

#### Delete Webhook

```bash
# Delete specific webhook
eas webhook:delete [WEBHOOK_ID]

# Delete webhook with confirmation
eas webhook:delete [WEBHOOK_ID] --confirm
```

---

## EAS Device Commands

### Device Management

#### List Devices

```bash
# List all devices
eas device:list

# List devices for platform
eas device:list --platform ios

# List devices with details
eas device:list --details
```

#### Create Device

```bash
# Create iOS device
eas device:create --platform ios

# Create Android device
eas device:create --platform android

# Create device with name
eas device:create --platform ios --name "iPhone 15 Pro"
```

#### Delete Device

```bash
# Delete specific device
eas device:delete [DEVICE_ID]

# Delete device with confirmation
eas device:delete [DEVICE_ID] --confirm
```

---

## Advanced EAS Commands

### Environment Variables

#### Set Environment Variables

```bash
# Set environment variable
eas secret:create --name API_URL --value "https://api.example.com"

# Set environment variable with scope
eas secret:create --scope project --name API_URL --value "https://api.example.com"

# Set environment variable with environment
eas secret:create --scope project --name API_URL --value "https://api.example.com" --environment production
```

#### List Environment Variables

```bash
# List all secrets
eas secret:list

# List secrets for project
eas secret:list --scope project

# List secrets with values
eas secret:list --scope project --show-values
```

#### Delete Environment Variables

```bash
# Delete specific secret
eas secret:delete [SECRET_ID]

# Delete secret with confirmation
eas secret:delete [SECRET_ID] --confirm
```

### Branch Management

#### List Branches

```bash
# List all branches
eas branch:list

# List branches for project
eas branch:list --project-id [PROJECT_ID]
```

#### Create Branch

```bash
# Create new branch
eas branch:create --name "feature-branch"

# Create branch with description
eas branch:create --name "feature-branch" --description "New feature implementation"
```

#### Delete Branch

```bash
# Delete specific branch
eas branch:delete [BRANCH_ID]

# Delete branch with confirmation
eas branch:delete [BRANCH_ID] --confirm
```

---

## EAS Command Options

### Global Options

```bash
# Non-interactive mode
eas [command] --non-interactive

# Verbose output
eas [command] --verbose

# Help for specific command
eas [command] --help

# Version information
eas --version
```

### Platform Options

```bash
# Specify platform
eas [command] --platform ios
eas [command] --platform android
eas [command] --platform web
eas [command] --platform all
```

### Profile Options

```bash
# Specify profile
eas [command] --profile development
eas [command] --profile preview
eas [command] --profile production
eas [command] --profile staging
```

### Output Options

```bash
# Specify output directory
eas [command] --output-dir ./builds

# Specify output format
eas [command] --format json

# Specify output file
eas [command] --output-file build-info.json
```

---

## Troubleshooting

### Common Issues

#### Authentication Issues

```bash
# Check authentication status
eas whoami

# Re-authenticate
eas logout
eas login

# Check project access
eas project:info
```

#### Build Issues

```bash
# Check build status
eas build:list

# View build logs
eas build:view [BUILD_ID] --logs

# Retry failed build
eas build --platform all --retry

# Clear build cache
eas build --platform all --clear-cache
```

#### Submit Issues

```bash
# Check submit status
eas submit:list

# View submit logs
eas submit:view [SUBMISSION_ID] --logs

# Retry failed submission
eas submit --platform all --retry
```

#### Update Issues

```bash
# Check update status
eas update:status

# View update logs
eas update:view [UPDATE_ID] --logs

# Rollback problematic update
eas update:rollback --branch production
```

### Debug Commands

```bash
# Enable debug mode
eas [command] --debug

# Verbose output
eas [command] --verbose

# Check configuration
eas project:info --config

# Validate configuration
eas build:configure --validate
```

---

## Best Practices

### Command Usage

#### 1. Use Specific Profiles

```bash
# Good: Use specific profile
eas build --platform ios --profile production

# Bad: Use default profile
eas build --platform ios
```

#### 2. Use Non-Interactive Mode for CI/CD

```bash
# Good: Non-interactive for automation
eas build --platform all --profile production --non-interactive

# Bad: Interactive mode in CI/CD
eas build --platform all --profile production
```

#### 3. Use Meaningful Messages

```bash
# Good: Descriptive message
eas update --message "Fix login bug in production"

# Bad: Generic message
eas update --message "update"
```

#### 4. Check Status Before Actions

```bash
# Good: Check before building
eas build:list
eas build --platform ios --profile production

# Bad: Build without checking
eas build --platform ios --profile production
```

### Configuration Management

#### 1. Use Version Control

```bash
# Commit eas.json
git add eas.json
git commit -m "Update EAS configuration"

# Tag releases
git tag v1.0.0
git push origin v1.0.0
```

#### 2. Use Environment Variables

```bash
# Set secrets
eas secret:create --name API_URL --value "https://api.example.com"

# Use in builds
eas build --platform all --profile production
```

#### 3. Use Profiles for Different Environments

```json
{
  "build": {
    "development": {
      "distribution": "internal",
      "env": {
        "EXPO_PUBLIC_API_URL": "https://dev-api.example.com"
      }
    },
    "production": {
      "distribution": "store",
      "env": {
        "EXPO_PUBLIC_API_URL": "https://api.example.com"
      }
    }
  }
}
```

---

## Quick Reference

### Essential Commands

```bash
# Build
eas build --platform all --profile production

# Submit
eas submit --platform all --profile production

# Update
eas update --branch production --message "Update message"

# Credentials
eas credentials:configure --platform all

# Project
eas project:info
```

### Command Structure

```bash
eas [command] [subcommand] [options]

# Examples
eas build --platform ios --profile production
eas submit --platform all --non-interactive
eas update --branch production --message "Update"
```

### Common Options

| Option              | Description          | Example                |
| ------------------- | -------------------- | ---------------------- |
| `--platform`        | Target platform      | `--platform ios`       |
| `--profile`         | Build profile        | `--profile production` |
| `--message`         | Build/update message | `--message "v1.0.0"`   |
| `--non-interactive` | Non-interactive mode | `--non-interactive`    |
| `--local`           | Use local changes    | `--local`              |
| `--clear-cache`     | Clear build cache    | `--clear-cache`        |

---

**Your EAS commands are now mastered! Build, submit, and update with confidence! 🚀**
