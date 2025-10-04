# Expo Go Updates & OTA Guide

This guide covers everything about updating your Expo app in Expo Go, including OTA (Over-The-Air) updates, publishing changes, and managing different environments.

## 📚 Table of Contents

1. [Understanding Expo Go Updates](#understanding-expo-go-updates)
2. [Publishing Updates](#publishing-updates)
3. [OTA (Over-The-Air) Updates](#ota-over-the-air-updates)
4. [Environment Management](#environment-management)
5. [Update Channels](#update-channels)
6. [Rollback Strategies](#rollback-strategies)
7. [Troubleshooting Updates](#troubleshooting-updates)

---

## Understanding Expo Go Updates

### What are Expo Go Updates?

Expo Go updates allow you to:

- **Push JavaScript/TypeScript changes** without rebuilding the app
- **Update UI, logic, and assets** instantly
- **Test changes** on physical devices immediately
- **Deploy hotfixes** without app store approval

### How Expo Go Updates Work

1. **Development Server**: Your local `npx expo start` creates a development server
2. **QR Code**: Expo Go scans the QR code to connect to your server
3. **Live Updates**: Changes are pushed instantly to connected devices
4. **Production Updates**: Published updates are served from Expo's CDN

### Update Types

| Update Type     | Description          | Speed       | Use Case                    |
| --------------- | -------------------- | ----------- | --------------------------- |
| **Development** | Local server updates | Instant     | Development & testing       |
| **OTA**         | Published updates    | ~30 seconds | Production hotfixes         |
| **Native**      | App store updates    | Days/weeks  | Major changes, new features |

---

## Publishing Updates

### Basic Publishing

```bash
# Publish current changes
npx expo publish

# Publish with specific message
npx expo publish --message "Fix login bug"

# Publish to specific channel
npx expo publish --release-channel production
```

### Publishing with EAS Update (Recommended)

```bash
# Install EAS CLI
npm install -g @expo/eas-cli

# Login to Expo
eas login

# Configure EAS Update
eas update:configure

# Publish update
eas update --branch production --message "New feature release"

# Publish to specific platform
eas update --platform ios --branch production
eas update --platform android --branch production
```

### Publishing Workflow

```bash
# 1. Make your changes
# Edit your code files

# 2. Test locally
npx expo start

# 3. Publish update
npx expo publish --message "Add new feature"

# 4. Verify update
# Check Expo Go app for updates
```

---

## OTA (Over-The-Air) Updates

### What is OTA?

OTA updates allow you to:

- **Update JavaScript code** without rebuilding
- **Push instant updates** to users
- **Fix bugs quickly** in production
- **A/B test features** easily

### OTA Update Process

```bash
# 1. Make changes to your code
# Example: Update App.js
import { StatusBar } from "expo-status-bar";
import { StyleSheet, Text, View } from "react-native";

export default function App() {
  return (
    <View style={styles.container}>
      <Text>Updated App Version 1.1!</Text>
      <StatusBar style="auto" />
    </View>
  );
}

# 2. Publish the update
npx expo publish --message "Version 1.1 update"

# 3. Users receive update automatically
# Next time they open the app
```

### OTA Update Configuration

#### app.json Configuration

```json
{
  "expo": {
    "name": "MyApp",
    "slug": "my-app",
    "version": "1.0.0",
    "updates": {
      "enabled": true,
      "checkAutomatically": "ON_LOAD",
      "fallbackToCacheTimeout": 0
    },
    "runtimeVersion": "1.0.0"
  }
}
```

#### Update Settings Explained

```json
{
  "updates": {
    "enabled": true, // Enable OTA updates
    "checkAutomatically": "ON_LOAD", // Check for updates when app loads
    "fallbackToCacheTimeout": 0, // Timeout for fallback to cache
    "url": "https://u.expo.dev/..." // Custom update server (optional)
  }
}
```

### Checking for Updates Programmatically

```javascript
// App.js
import { useEffect } from 'react';
import * as Updates from 'expo-updates';

export default function App() {
  useEffect(() => {
    checkForUpdates();
  }, []);

  const checkForUpdates = async () => {
    try {
      const update = await Updates.checkForUpdateAsync();
      if (update.isAvailable) {
        await Updates.fetchUpdateAsync();
        await Updates.reloadAsync();
      }
    } catch (error) {
      console.log('Error checking for updates:', error);
    }
  };

  return (
    // Your app content
  );
}
```

---

## Environment Management

### Development Environment

```bash
# Start development server
npx expo start

# Publish to development
npx expo publish --release-channel development

# Or use EAS Update
eas update --branch development
```

### Staging Environment

```bash
# Publish to staging
npx expo publish --release-channel staging

# Or use EAS Update
eas update --branch staging
```

### Production Environment

```bash
# Publish to production
npx expo publish --release-channel production

# Or use EAS Update
eas update --branch production
```

### Environment-Specific Configuration

```javascript
// config.js
const config = {
  development: {
    apiUrl: "https://dev-api.example.com",
    debug: true,
  },
  staging: {
    apiUrl: "https://staging-api.example.com",
    debug: true,
  },
  production: {
    apiUrl: "https://api.example.com",
    debug: false,
  },
};

const environment = __DEV__ ? "development" : "production";
export default config[environment];
```

---

## Update Channels

### Understanding Channels

Channels allow you to:

- **Separate environments** (dev, staging, prod)
- **Test features** before production
- **Rollback** to previous versions
- **A/B test** different versions

### Channel Management

```bash
# List all channels
npx expo publish --help

# Publish to specific channel
npx expo publish --release-channel production

# Publish to multiple channels
npx expo publish --release-channel production
npx expo publish --release-channel staging
```

### EAS Update Channels

```bash
# Create new branch
eas update --branch feature-branch

# Publish to existing branch
eas update --branch production

# List all branches
eas update:list

# View branch details
eas update:view [UPDATE_ID]
```

---

## Rollback Strategies

### Immediate Rollback

```bash
# Rollback to previous version
npx expo publish --release-channel production --rollback

# Or with EAS Update
eas update --branch production --rollback
```

### Selective Rollback

```bash
# Rollback to specific version
npx expo publish --release-channel production --rollback-to [VERSION_ID]

# Or with EAS Update
eas update --branch production --rollback-to [UPDATE_ID]
```

### Emergency Rollback

```bash
# Emergency rollback (immediate)
npx expo publish --release-channel production --rollback --message "Emergency rollback"

# Verify rollback
npx expo publish --release-channel production --rollback --dry-run
```

---

## Troubleshooting Updates

### Common Issues

#### 1. Update Not Appearing

```bash
# Check if update was published
npx expo publish --release-channel production --dry-run

# Clear Expo Go cache
# In Expo Go: Settings → Clear Cache

# Force update check
# Shake device → Reload
```

#### 2. Update Fails to Load

```bash
# Check update status
npx expo publish --release-channel production --status

# Check for errors
npx expo publish --release-channel production --verbose
```

#### 3. Stuck on Loading Screen

```bash
# Clear all caches
npx expo start --clear

# Reset Metro bundler
npx expo start --reset-cache
```

### Debug Commands

```bash
# Check current version
npx expo publish --release-channel production --current

# List all versions
npx expo publish --release-channel production --list

# Check update status
npx expo publish --release-channel production --status
```

### EAS Update Debugging

```bash
# List all updates
eas update:list

# View specific update
eas update:view [UPDATE_ID]

# Check update status
eas update:status

# View update logs
eas update:logs [UPDATE_ID]
```

---

## Best Practices

### Update Strategy

1. **Test First**: Always test updates in development
2. **Staged Rollout**: Use staging environment before production
3. **Monitor**: Watch for errors and user feedback
4. **Rollback Plan**: Always have a rollback strategy
5. **Version Control**: Use meaningful commit messages

### Update Messages

```bash
# Good update messages
npx expo publish --message "Fix login bug in production"
npx expo publish --message "Add new feature: user profiles"
npx expo publish --message "Performance improvements"

# Bad update messages
npx expo publish --message "fix"
npx expo publish --message "update"
npx expo publish --message "changes"
```

### Update Frequency

- **Development**: As often as needed
- **Staging**: Daily or per feature
- **Production**: Weekly or per hotfix
- **Emergency**: As needed for critical bugs

---

## Advanced Features

### Conditional Updates

```javascript
// Only update on specific conditions
import * as Updates from "expo-updates";

const checkForUpdates = async () => {
  try {
    const update = await Updates.checkForUpdateAsync();
    if (update.isAvailable) {
      // Check if user is on WiFi
      const isWiFi = await checkWiFiConnection();
      if (isWiFi) {
        await Updates.fetchUpdateAsync();
        await Updates.reloadAsync();
      }
    }
  } catch (error) {
    console.log("Update check failed:", error);
  }
};
```

### Update Notifications

```javascript
// Notify users about updates
import { Alert } from "react-native";

const checkForUpdates = async () => {
  try {
    const update = await Updates.checkForUpdateAsync();
    if (update.isAvailable) {
      Alert.alert(
        "Update Available",
        "A new version is available. Would you like to update now?",
        [
          { text: "Later", style: "cancel" },
          {
            text: "Update",
            onPress: async () => {
              await Updates.fetchUpdateAsync();
              await Updates.reloadAsync();
            },
          },
        ]
      );
    }
  } catch (error) {
    console.log("Update check failed:", error);
  }
};
```

---

## Quick Reference

### Essential Commands

```bash
# Publish update
npx expo publish --message "Your update message"

# Publish to specific channel
npx expo publish --release-channel production

# Check update status
npx expo publish --release-channel production --status

# Rollback update
npx expo publish --release-channel production --rollback

# EAS Update commands
eas update --branch production --message "Update message"
eas update:list
eas update:view [UPDATE_ID]
```

### Update Flow

1. **Make Changes** → Edit your code
2. **Test Locally** → `npx expo start`
3. **Publish Update** → `npx expo publish`
4. **Verify Update** → Check in Expo Go
5. **Monitor** → Watch for issues
6. **Rollback if Needed** → `npx expo publish --rollback`

---

**Your Expo Go updates are now ready! Push changes instantly to your users! 🚀**
