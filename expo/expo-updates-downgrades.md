# Expo Updates & Downgrades Guide

This comprehensive guide covers everything about updating and downgrading Expo SDK, managing dependencies, fixing compatibility issues, and handling version-related problems.

## 📚 Table of Contents

1. [Understanding Expo Versions](#understanding-expo-versions)
2. [Updating Expo SDK](#updating-expo-sdk)
3. [Downgrading Expo SDK](#downgrading-expo-sdk)
4. [Dependency Management](#dependency-management)
5. [Compatibility Issues](#compatibility-issues)
6. [Migration Guides](#migration-guides)
7. [Troubleshooting](#troubleshooting)
8. [Best Practices](#best-practices)

---

## Understanding Expo Versions

### Expo SDK Versioning

Expo uses semantic versioning (SemVer) with the format `MAJOR.MINOR.PATCH`:

- **MAJOR**: Breaking changes, requires migration
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes, backward compatible

### Current Expo SDK Versions

| SDK Version | Release Date | Status        | React Native | Node.js |
| ----------- | ------------ | ------------- | ------------ | ------- |
| **SDK 50**  | Latest       | ✅ Current    | 0.73.x       | 18+     |
| **SDK 49**  | Previous     | ✅ Stable     | 0.72.x       | 18+     |
| **SDK 48**  | Older        | ⚠️ Deprecated | 0.71.x       | 16+     |
| **SDK 47**  | Legacy       | ❌ EOL        | 0.70.x       | 16+     |

### Version Compatibility

```json
{
  "expo": {
    "sdkVersion": "50.0.0",
    "runtimeVersion": "1.0.0"
  },
  "dependencies": {
    "expo": "~50.0.0",
    "react-native": "0.73.6"
  }
}
```

---

## Updating Expo SDK

### Check Current Version

```bash
# Check Expo CLI version
npx expo --version

# Check project SDK version
npx expo install --check

# Check all package versions
npm list expo
```

### Update to Latest Version

#### Method 1: Using Expo CLI (Recommended)

```bash
# Update to latest SDK
npx expo install --fix

# Update to specific SDK version
npx expo install --sdk-version 50

# Update with non-interactive mode
npx expo install --fix --non-interactive
```

#### Method 2: Manual Update

```bash
# Update Expo SDK
npm install expo@latest

# Update React Native
npm install react-native@latest

# Update Expo modules
npx expo install --fix
```

#### Method 3: Using EAS Update

```bash
# Update EAS CLI
npm install -g @expo/eas-cli

# Update project dependencies
eas update --fix
```

### Step-by-Step Update Process

#### 1. Backup Your Project

```bash
# Create backup branch
git checkout -b backup-before-update

# Commit current state
git add .
git commit -m "Backup before Expo update"

# Return to main branch
git checkout main
```

#### 2. Update Expo SDK

```bash
# Update to latest SDK
npx expo install --fix

# Verify update
npx expo install --check
```

#### 3. Update Dependencies

```bash
# Update all dependencies
npm update

# Update specific packages
npm install package-name@latest

# Update Expo modules
npx expo install --fix
```

#### 4. Test the Update

```bash
# Start development server
npx expo start

# Test on device
# Scan QR code with Expo Go

# Test on simulator
npx expo start --ios
npx expo start --android
```

#### 5. Fix Any Issues

```bash
# Check for errors
npx expo doctor

# Fix common issues
npx expo install --fix

# Clear cache if needed
npx expo start --clear
```

---

## Downgrading Expo SDK

### When to Downgrade

- **Breaking Changes**: New SDK has breaking changes
- **Compatibility Issues**: Dependencies not compatible
- **Stability Issues**: New version has bugs
- **Team Requirements**: Team needs specific version

### Downgrade Process

#### Method 1: Using Expo CLI

```bash
# Downgrade to specific SDK version
npx expo install --sdk-version 49

# Downgrade with fix
npx expo install --sdk-version 49 --fix
```

#### Method 2: Manual Downgrade

```bash
# Downgrade Expo SDK
npm install expo@49.0.0

# Downgrade React Native
npm install react-native@0.72.6

# Fix dependencies
npx expo install --fix
```

#### Method 3: Complete Downgrade

```bash
# Remove node_modules
rm -rf node_modules

# Remove package-lock.json
rm package-lock.json

# Install specific versions
npm install expo@49.0.0 react-native@0.72.6

# Install compatible dependencies
npx expo install --fix
```

### Step-by-Step Downgrade Process

#### 1. Check Current Version

```bash
# Check current SDK version
npx expo install --check

# Check package versions
npm list expo react-native
```

#### 2. Choose Target Version

```bash
# List available SDK versions
npm view expo versions --json

# Choose compatible version
# SDK 49: React Native 0.72.x
# SDK 48: React Native 0.71.x
# SDK 47: React Native 0.70.x
```

#### 3. Downgrade SDK

```bash
# Downgrade to SDK 49
npx expo install --sdk-version 49

# Verify downgrade
npx expo install --check
```

#### 4. Fix Dependencies

```bash
# Fix incompatible dependencies
npx expo install --fix

# Update package.json
npm install

# Clear cache
npx expo start --clear
```

#### 5. Test Downgrade

```bash
# Start development server
npx expo start

# Test functionality
# Verify all features work
```

---

## Dependency Management

### Understanding Dependencies

#### Expo Dependencies

```json
{
  "dependencies": {
    "expo": "~50.0.0",
    "react-native": "0.73.6",
    "expo-status-bar": "~1.11.1",
    "expo-splash-screen": "~0.20.5",
    "expo-constants": "~15.4.5"
  }
}
```

#### Third-Party Dependencies

```json
{
  "dependencies": {
    "react": "18.2.0",
    "react-native": "0.73.6",
    "expo": "~50.0.0",
    "expo-camera": "~14.0.1",
    "expo-location": "~16.5.5",
    "expo-notifications": "~0.27.6"
  }
}
```

### Managing Dependencies

#### Install Dependencies

```bash
# Install Expo module
npx expo install expo-camera

# Install regular npm package
npm install lodash

# Install development dependency
npm install --save-dev @types/lodash
```

#### Update Dependencies

```bash
# Update all dependencies
npm update

# Update specific package
npm install package-name@latest

# Update Expo modules
npx expo install --fix
```

#### Remove Dependencies

```bash
# Remove package
npm uninstall package-name

# Remove Expo module
npm uninstall expo-camera

# Clean up
npm install
```

### Dependency Conflicts

#### Common Conflicts

```bash
# Check for conflicts
npm ls

# Check for peer dependency warnings
npm install --verbose

# Check for outdated packages
npm outdated
```

#### Resolving Conflicts

```bash
# Force install
npm install --force

# Clear cache and reinstall
npm cache clean --force
rm -rf node_modules package-lock.json
npm install

# Use specific versions
npm install package-name@1.2.3
```

---

## Compatibility Issues

### Common Compatibility Issues

#### 1. React Native Version Mismatch

```bash
# Error: React Native version mismatch
# Solution: Update React Native
npx expo install react-native@latest
```

#### 2. Expo Module Compatibility

```bash
# Error: Expo module not compatible
# Solution: Update Expo modules
npx expo install --fix
```

#### 3. Node.js Version Issues

```bash
# Error: Node.js version not supported
# Solution: Update Node.js
# Check required version in Expo docs
```

#### 4. Metro Bundler Issues

```bash
# Error: Metro bundler issues
# Solution: Clear Metro cache
npx expo start --clear
```

### Fixing Compatibility Issues

#### Method 1: Automatic Fix

```bash
# Fix all compatibility issues
npx expo install --fix

# Check for issues
npx expo doctor
```

#### Method 2: Manual Fix

```bash
# Update specific packages
npm install expo@latest
npm install react-native@latest

# Update Expo modules
npx expo install --fix

# Clear cache
npx expo start --clear
```

#### Method 3: Complete Reset

```bash
# Remove all dependencies
rm -rf node_modules package-lock.json

# Reinstall everything
npm install
npx expo install --fix

# Clear all caches
npx expo start --clear
```

---

## Migration Guides

### SDK 49 to SDK 50

#### Breaking Changes

```bash
# Update to SDK 50
npx expo install --sdk-version 50

# Fix breaking changes
npx expo install --fix
```

#### Key Changes

- **React Native**: Updated to 0.73.x
- **New Features**: New Expo modules
- **Deprecated APIs**: Some APIs deprecated
- **Performance**: Improved performance

#### Migration Steps

```bash
# 1. Update SDK
npx expo install --sdk-version 50

# 2. Update dependencies
npx expo install --fix

# 3. Update code
# Replace deprecated APIs
# Update imports
# Fix breaking changes

# 4. Test migration
npx expo start
```

### SDK 48 to SDK 49

#### Breaking Changes

```bash
# Update to SDK 49
npx expo install --sdk-version 49

# Fix breaking changes
npx expo install --fix
```

#### Key Changes

- **React Native**: Updated to 0.72.x
- **New Features**: New Expo modules
- **Deprecated APIs**: Some APIs deprecated
- **Performance**: Improved performance

### SDK 47 to SDK 48

#### Breaking Changes

```bash
# Update to SDK 48
npx expo install --sdk-version 48

# Fix breaking changes
npx expo install --fix
```

#### Key Changes

- **React Native**: Updated to 0.71.x
- **New Features**: New Expo modules
- **Deprecated APIs**: Some APIs deprecated
- **Performance**: Improved performance

---

## Troubleshooting

### Common Update Issues

#### 1. Update Fails

```bash
# Check for errors
npx expo install --check

# Clear cache
npm cache clean --force

# Try again
npx expo install --fix
```

#### 2. Dependencies Not Compatible

```bash
# Check compatibility
npx expo doctor

# Fix dependencies
npx expo install --fix

# Update specific packages
npm install package-name@latest
```

#### 3. Build Issues

```bash
# Clear build cache
npx expo start --clear

# Check build logs
npx expo build:logs [BUILD_ID]

# Retry build
npx expo build --platform all --retry
```

#### 4. Runtime Errors

```bash
# Check for errors
npx expo start

# Clear Metro cache
npx expo start --clear

# Check dependencies
npx expo install --check
```

### Common Downgrade Issues

#### 1. Downgrade Fails

```bash
# Check current version
npx expo install --check

# Try specific version
npx expo install --sdk-version 49

# Clear cache
npm cache clean --force
```

#### 2. Dependencies Not Compatible

```bash
# Fix dependencies
npx expo install --fix

# Update package.json
npm install

# Clear cache
npx expo start --clear
```

#### 3. Build Issues

```bash
# Clear build cache
npx expo start --clear

# Check build logs
npx expo build:logs [BUILD_ID]

# Retry build
npx expo build --platform all --retry
```

### Debug Commands

```bash
# Check Expo version
npx expo --version

# Check project status
npx expo doctor

# Check dependencies
npx expo install --check

# Check for issues
npx expo start --verbose

# Clear all caches
npx expo start --clear --reset-cache
```

---

## Best Practices

### Update Strategy

#### 1. Regular Updates

```bash
# Update monthly
npx expo install --fix

# Update dependencies
npm update

# Test updates
npx expo start
```

#### 2. Major Updates

```bash
# Create backup
git checkout -b backup-before-update

# Update SDK
npx expo install --sdk-version 50

# Fix issues
npx expo install --fix

# Test thoroughly
npx expo start
```

#### 3. Emergency Updates

```bash
# Quick fix
npx expo install --fix

# Clear cache
npx expo start --clear

# Test immediately
npx expo start
```

### Version Management

#### 1. Pin Versions

```json
{
  "dependencies": {
    "expo": "~50.0.0",
    "react-native": "0.73.6"
  }
}
```

#### 2. Use Exact Versions

```json
{
  "dependencies": {
    "expo": "50.0.0",
    "react-native": "0.73.6"
  }
}
```

#### 3. Regular Updates

```bash
# Check for updates
npm outdated

# Update regularly
npm update

# Test updates
npx expo start
```

### Testing Strategy

#### 1. Test Before Update

```bash
# Test current version
npx expo start

# Test on device
# Test on simulator
# Test all features
```

#### 2. Test After Update

```bash
# Test updated version
npx expo start

# Test on device
# Test on simulator
# Test all features
```

#### 3. Rollback Plan

```bash
# Keep backup
git checkout backup-before-update

# Downgrade if needed
npx expo install --sdk-version 49

# Fix issues
npx expo install --fix
```

---

## Quick Reference

### Essential Commands

```bash
# Check current version
npx expo --version

# Update to latest
npx expo install --fix

# Update to specific version
npx expo install --sdk-version 50

# Check for issues
npx expo doctor

# Clear cache
npx expo start --clear

# Fix dependencies
npx expo install --fix
```

### Version Compatibility

| SDK Version | React Native | Node.js | Status        |
| ----------- | ------------ | ------- | ------------- |
| **50**      | 0.73.x       | 18+     | ✅ Current    |
| **49**      | 0.72.x       | 18+     | ✅ Stable     |
| **48**      | 0.71.x       | 16+     | ⚠️ Deprecated |
| **47**      | 0.70.x       | 16+     | ❌ EOL        |

### Update Checklist

- [ ] **Backup Project** - Create backup branch
- [ ] **Check Current Version** - Verify current SDK
- [ ] **Update SDK** - Use `npx expo install --fix`
- [ ] **Fix Dependencies** - Resolve conflicts
- [ ] **Test Update** - Verify functionality
- [ ] **Fix Issues** - Address any problems
- [ ] **Commit Changes** - Save working version

---

**Your Expo updates and downgrades are now managed! Keep your app up-to-date and running smoothly! 🚀**
