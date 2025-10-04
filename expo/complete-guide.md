# Complete Expo Development Guide

This comprehensive guide covers everything you need to know about Expo React Native development, from setup to deployment.

## 📚 Table of Contents

1. [Prerequisites & Environment Setup](#prerequisites--environment-setup)
2. [Project Creation](#project-creation)
3. [Blank Template - JavaScript](#blank-template---javascript)
4. [Blank Template - TypeScript](#blank-template---typescript)
5. [Development Workflow](#development-workflow)
6. [Building and Deployment](#building-and-deployment)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites & Environment Setup

### System Requirements

- **Operating System**: macOS, Windows, or Linux
- **Node.js**: Version 18 or later (LTS recommended)
- **Package Manager**: npm (comes with Node.js) or yarn
- **Code Editor**: VS Code (recommended)
- **Mobile Device**: iOS or Android device for testing

### Step 1: Install Node.js

```bash
# Check if Node.js is installed
node --version
npm --version

# If not installed, download from nodejs.org
# Install LTS version (recommended)
```

### Step 2: Install Expo CLI (Latest Method)

**Important**: The global `@expo/cli` is deprecated. Use `npx` instead:

```bash
# ❌ OLD WAY (deprecated)
# npm install -g @expo/cli

# ✅ NEW WAY (recommended)
# No global installation needed - use npx
npx create-expo-app@latest --help
```

### Step 3: Install Expo Go App

- **iOS**: App Store → Search "Expo Go" → Install
- **Android**: Google Play Store → Search "Expo Go" → Install

### Step 4: Install VS Code Extensions

```bash
# Install recommended extensions
code --install-extension ms-vscode.vscode-typescript-next
code --install-extension bradlc.vscode-tailwindcss
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode.vscode-json
code --install-extension ms-vscode.vscode-eslint
```

### Step 5: Verify Setup

```bash
# Test the setup
npx create-expo-app@latest TestProject --template blank
cd TestProject
npx expo start
```

---

## Project Creation

### Available Templates (2024)

| Template            | Command                 | Description                      |
| ------------------- | ----------------------- | -------------------------------- |
| **Blank (JS)**      | `blank`                 | Minimal JavaScript setup         |
| **Blank (TS)**      | `blank-typescript`      | Minimal TypeScript setup         |
| **Tabs (JS)**       | `tabs`                  | Tab navigation with JavaScript   |
| **Tabs (TS)**       | `tabs-typescript`       | Tab navigation with TypeScript   |
| **Navigation (JS)** | `navigation`            | Stack navigation with JavaScript |
| **Navigation (TS)** | `navigation-typescript` | Stack navigation with TypeScript |

### Method 1: Interactive Setup (Recommended)

```bash
# Create project with interactive prompts
npx create-expo-app@latest MyApp

# Follow prompts:
# ? What would you like your app to be called? › MyApp
# ? What type of app are you making? ›
#   ❯ App (JavaScript)
#     App (TypeScript)
#     App (JavaScript + TypeScript)
#     App (JavaScript + TypeScript + EAS)
#     App (JavaScript + EAS)
#     App (TypeScript + EAS)
#     App (JavaScript + TypeScript + EAS + Web)
#     App (JavaScript + EAS + Web)
#     App (TypeScript + EAS + Web)
#     App (JavaScript + TypeScript + EAS + Web + NativeWind)
#     App (JavaScript + EAS + Web + NativeWind)
#     App (TypeScript + EAS + Web + NativeWind)
```

### Method 2: Direct Template Specification

```bash
# JavaScript Blank Template
npx create-expo-app@latest MyApp --template blank

# TypeScript Blank Template
npx create-expo-app@latest MyApp --template blank-typescript

# Tab Navigation (JavaScript)
npx create-expo-app@latest MyApp --template tabs

# Tab Navigation (TypeScript)
npx create-expo-app@latest MyApp --template tabs-typescript

# Stack Navigation (JavaScript)
npx create-expo-app@latest MyApp --template navigation

# Stack Navigation (TypeScript)
npx create-expo-app@latest MyApp --template navigation-typescript
```

### Project Structure

```
MyApp/
├── .expo/                 # Expo configuration (auto-generated)
├── assets/               # Images, fonts, and other assets
│   ├── images/
│   └── fonts/
├── node_modules/         # Dependencies
├── .gitignore           # Git ignore rules
├── App.js               # Main app component (or App.tsx)
├── app.json             # Expo configuration
├── babel.config.js      # Babel configuration
├── package.json         # Project dependencies and scripts
└── README.md           # Project documentation
```

---

## Blank Template - JavaScript

### Creating JavaScript Project

```bash
# Create blank JavaScript project
npx create-expo-app@latest MyJSApp --template blank

# Navigate to project
cd MyJSApp

# Start development server
npx expo start
```

### Default App.js Structure

```javascript
import { StatusBar } from "expo-status-bar";
import { StyleSheet, Text, View } from "react-native";

export default function App() {
  return (
    <View style={styles.container}>
      <Text>Open up App.js to start working on your app!</Text>
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
    alignItems: "center",
    justifyContent: "center",
  },
});
```

### Key Features

- **React Native Components**: Basic View, Text, StyleSheet
- **Expo StatusBar**: Cross-platform status bar management
- **Hot Reloading**: Automatic refresh on file changes
- **Cross-platform**: Works on iOS, Android, and Web

### Adding Dependencies

```bash
# Add regular npm packages
npm install package-name

# Add Expo SDK packages (recommended)
npx expo install expo-package-name

# Add development dependencies
npm install --save-dev package-name
```

### Common JavaScript Patterns

```javascript
// State management with useState
import { useState } from "react";
import { View, Text, Button } from "react-native";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Text>Count: {count}</Text>
      <Button title="Increment" onPress={() => setCount(count + 1)} />
    </View>
  );
}
```

---

## Blank Template - TypeScript

### Creating TypeScript Project

```bash
# Create blank TypeScript project
npx create-expo-app@latest MyTSApp --template blank-typescript

# Navigate to project
cd MyTSApp

# Start development server
npx expo start
```

### Default App.tsx Structure

```typescript
import { StatusBar } from "expo-status-bar";
import { StyleSheet, Text, View } from "react-native";

export default function App(): JSX.Element {
  return (
    <View style={styles.container}>
      <Text>Open up App.tsx to start working on your app!</Text>
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
    alignItems: "center",
    justifyContent: "center",
  },
});
```

### TypeScript Configuration

The project includes `tsconfig.json`:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "allowSyntheticDefaultImports": true,
    "jsx": "react-native",
    "lib": ["dom", "esnext"],
    "moduleResolution": "node",
    "noEmit": true,
    "skipLibCheck": true,
    "resolveJsonModule": true
  }
}
```

### TypeScript Features

- **Type Safety**: Compile-time error checking
- **IntelliSense**: Better IDE support and autocomplete
- **Interface Definitions**: Define component props and state types
- **Generic Types**: Reusable type definitions

### Common TypeScript Patterns

```typescript
// Interface definitions
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserCardProps {
  user: User;
  onPress: (user: User) => void;
}

// Component with typed props
import React from "react";
import { View, Text, TouchableOpacity } from "react-native";

const UserCard: React.FC<UserCardProps> = ({ user, onPress }) => {
  return (
    <TouchableOpacity onPress={() => onPress(user)}>
      <View>
        <Text>{user.name}</Text>
        <Text>{user.email}</Text>
      </View>
    </TouchableOpacity>
  );
};

export default UserCard;
```

---

## Development Workflow

### Running Your Project

```bash
# Start development server
npx expo start

# Start with specific platform
npx expo start --ios
npx expo start --android
npx expo start --web

# Start with tunnel (for remote access)
npx expo start --tunnel

# Start with clear cache
npx expo start --clear
```

### Testing on Devices

1. **Physical Device**:

   - Install Expo Go app
   - Scan QR code from terminal
   - App loads on device

2. **iOS Simulator** (macOS only):

   ```bash
   npx expo start --ios
   ```

3. **Android Emulator**:

   ```bash
   npx expo start --android
   ```

4. **Web Browser**:
   ```bash
   npx expo start --web
   ```

### Development Features

- **Hot Reloading**: Automatic refresh on file changes
- **Fast Refresh**: Preserves component state during updates
- **Error Overlay**: Clear error messages and stack traces
- **Debug Menu**: Shake device or press Cmd+D (iOS) / Cmd+M (Android)

### Package Management

```bash
# Install packages
npm install package-name

# Install Expo SDK packages (recommended)
npx expo install expo-camera expo-location

# Update packages
npm update

# Remove packages
npm uninstall package-name
```

### Configuration Files

#### app.json

```json
{
  "expo": {
    "name": "MyApp",
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
      "favicon": "./assets/favicon.png",
      "bundler": "metro"
    }
  }
}
```

#### package.json Scripts

```json
{
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web"
  }
}
```

---

## Building and Deployment

### EAS Build (Recommended)

```bash
# Install EAS CLI
npm install -g @expo/eas-cli

# Login to Expo
eas login

# Configure EAS
eas build:configure

# Build for platforms
eas build --platform ios
eas build --platform android
eas build --platform all
```

### Local Builds

```bash
# Build for web
npx expo export --platform web

# Build for iOS (requires Xcode)
npx expo run:ios

# Build for Android (requires Android Studio)
npx expo run:android
```

### App Store Deployment

1. **iOS App Store**:

   ```bash
   # Build for App Store
   eas build --platform ios --profile production

   # Submit to App Store
   eas submit --platform ios
   ```

2. **Google Play Store**:

   ```bash
   # Build for Play Store
   eas build --platform android --profile production

   # Submit to Play Store
   eas submit --platform android
   ```

---

## Troubleshooting

### Common Issues

#### 1. Metro Bundler Issues

```bash
# Clear Metro cache
npx expo start --clear

# Reset Metro cache
npx expo start --reset-cache

# Clear all caches
npx expo start --clear --reset-cache
```

#### 2. Port Already in Use

```bash
# Use different port
npx expo start --port 8082

# Kill process using port
npx kill-port 8081
```

#### 3. Dependencies Issues

```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear npm cache
npm cache clean --force
```

#### 4. Expo Go Connection Issues

```bash
# Use tunnel mode
npx expo start --tunnel

# Use localhost mode
npx expo start --localhost

# Check network connectivity
ping expo.dev
```

#### 5. TypeScript Issues

```bash
# Check TypeScript configuration
npx tsc --noEmit

# Restart TypeScript server in VS Code
# Cmd+Shift+P → "TypeScript: Restart TS Server"
```

#### 6. Build Issues

```bash
# Clear build cache
eas build --clear-cache

# Check build logs
eas build:list
eas build:view [BUILD_ID]
```

### Debug Commands

```bash
# Check Expo CLI version
npx expo --version

# Check project configuration
npx expo config

# Check for updates
npx expo install --fix

# Doctor command (check for issues)
npx expo doctor
```

### Performance Issues

```bash
# Enable performance monitoring
npx expo start --dev-client

# Check bundle size
npx expo export --platform web --output-dir ./dist
npx expo export --platform android --output-dir ./dist
```

---

## Quick Reference

### Essential Commands

```bash
# Create new project
npx create-expo-app@latest MyApp --template blank

# Start development
npx expo start

# Install packages
npx expo install package-name

# Build for production
eas build --platform all

# Submit to stores
eas submit --platform all
```

### File Structure

```
MyApp/
├── App.js/tsx           # Main app component
├── app.json            # Expo configuration
├── package.json        # Dependencies
├── assets/             # Images, fonts
├── components/         # Reusable components
├── screens/           # Screen components
├── navigation/        # Navigation setup
└── utils/            # Utility functions
```

### Useful Resources

- [Expo Documentation](https://docs.expo.dev/)
- [React Native Documentation](https://reactnative.dev/)
- [Expo GitHub](https://github.com/expo/expo)
- [Expo Community](https://forums.expo.dev/)

---

**Happy coding with Expo! 🚀**

This guide covers everything from basic setup to advanced deployment. Keep this as your reference for all Expo development needs.
