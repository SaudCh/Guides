# App Store & Google Play Store Submission Guide

This comprehensive guide covers everything about submitting your Expo app to the App Store and Google Play Store, including preparation, submission process, and best practices.

## 📚 Table of Contents

1. [Pre-Submission Preparation](#pre-submission-preparation)
2. [App Store Submission (iOS)](#app-store-submission-ios)
3. [Google Play Store Submission (Android)](#google-play-store-submission-android)
4. [EAS Submit Automation](#eas-submit-automation)
5. [Store Listing Optimization](#store-listing-optimization)
6. [Review Process](#review-process)
7. [Post-Submission Management](#post-submission-management)
8. [Troubleshooting](#troubleshooting)

---

## Pre-Submission Preparation

### App Store Requirements Checklist

#### iOS App Store

- [ ] **App Store Connect Account** - Active developer account
- [ ] **App Bundle ID** - Unique identifier registered
- [ ] **App Icons** - All required sizes (1024x1024, 180x180, etc.)
- [ ] **Splash Screen** - Properly sized and designed
- [ ] **App Description** - Compelling and keyword-optimized
- [ ] **Screenshots** - All required device sizes
- [ ] **Privacy Policy** - Required for most apps
- [ ] **App Review Information** - Demo account and notes
- [ ] **Age Rating** - Appropriate content rating
- [ ] **App Store Guidelines** - Compliance verified

#### Google Play Store

- [ ] **Google Play Console Account** - Active developer account
- [ ] **Package Name** - Unique identifier registered
- [ ] **App Icons** - All required sizes (512x512, 192x192, etc.)
- [ ] **Feature Graphic** - 1024x500 promotional image
- [ ] **App Description** - Compelling and keyword-optimized
- [ ] **Screenshots** - All required device sizes
- [ ] **Privacy Policy** - Required for most apps
- [ ] **Content Rating** - Appropriate rating questionnaire
- [ ] **Target Audience** - Age and content targeting
- [ ] **Play Store Guidelines** - Compliance verified

### App Metadata Preparation

#### App Information

```json
{
  "expo": {
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "version": "1.0.0",
    "description": "A comprehensive mobile app that helps users manage their daily tasks efficiently.",
    "keywords": ["productivity", "task management", "mobile", "app"],
    "privacy": "public",
    "orientation": "portrait",
    "primaryColor": "#000000",
    "icon": "./assets/icon.png",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    }
  }
}
```

#### Store Listing Assets

```
assets/
├── app-store/
│   ├── icon-1024.png          # 1024x1024 app icon
│   ├── icon-180.png           # 180x180 app icon
│   ├── icon-167.png           # 167x167 app icon
│   ├── icon-152.png           # 152x152 app icon
│   ├── icon-120.png           # 120x120 app icon
│   ├── icon-87.png            # 87x87 app icon
│   ├── icon-80.png            # 80x80 app icon
│   ├── icon-76.png            # 76x76 app icon
│   ├── icon-60.png            # 60x60 app icon
│   ├── icon-58.png            # 58x58 app icon
│   ├── icon-40.png            # 40x40 app icon
│   ├── icon-29.png            # 29x29 app icon
│   ├── icon-20.png            # 20x20 app icon
│   ├── splash-2732.png        # 2732x2732 splash screen
│   ├── splash-2688.png        # 2688x2688 splash screen
│   ├── splash-2532.png        # 2532x2532 splash screen
│   ├── splash-2436.png        # 2436x2436 splash screen
│   ├── splash-2388.png        # 2388x2388 splash screen
│   ├── splash-2340.png        # 2340x2340 splash screen
│   ├── splash-2208.png        # 2208x2208 splash screen
│   ├── splash-2048.png        # 2048x2048 splash screen
│   ├── screenshot-6.7.png     # 6.7" iPhone screenshot
│   ├── screenshot-6.5.png     # 6.5" iPhone screenshot
│   ├── screenshot-5.5.png     # 5.5" iPhone screenshot
│   ├── screenshot-12.9.png    # 12.9" iPad screenshot
│   └── screenshot-11.png      # 11" iPad screenshot
└── play-store/
    ├── icon-512.png           # 512x512 app icon
    ├── icon-192.png           # 192x192 app icon
    ├── icon-144.png           # 144x144 app icon
    ├── icon-96.png            # 96x96 app icon
    ├── icon-72.png            # 72x72 app icon
    ├── icon-48.png            # 48x48 app icon
    ├── icon-36.png            # 36x36 app icon
    ├── feature-graphic.png    # 1024x500 feature graphic
    ├── screenshot-phone.png   # Phone screenshot
    ├── screenshot-tablet.png  # Tablet screenshot
    └── screenshot-tv.png      # TV screenshot (if applicable)
```

---

## App Store Submission (iOS)

### Step 1: Build Your App First

#### Create iOS Build

```bash
# Build for App Store
eas build --platform ios --profile production

# Check build status
eas build:list --platform ios

# Download build when ready
eas build:download [BUILD_ID]
```

#### Verify Build

- **Build Status**: Ensure build completed successfully
- **Build Size**: Check if build size is reasonable
- **Build Logs**: Review logs for any warnings or errors
- **Test Build**: Test the build on device if possible

### Step 2: App Store Connect Setup

#### Create App Record

1. **Login to App Store Connect**

   - Go to [appstoreconnect.apple.com](https://appstoreconnect.apple.com)
   - Sign in with your Apple ID

2. **Create New App**
   - Click "My Apps" → "+" → "New App"
   - Fill in app information:
     - **Platform**: iOS
     - **Name**: Your app name
     - **Primary Language**: English (or your primary language)
     - **Bundle ID**: Select from registered bundle IDs
     - **SKU**: Unique identifier (e.g., "myapp-ios-001")
     - **User Access**: Full Access

### Step 3: App Information Configuration

#### App Information

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
        "NSLocationWhenInUseUsageDescription": "This app needs location access for features",
        "NSPhotoLibraryUsageDescription": "This app needs access to photo library to save images"
      }
    }
  }
}
```

#### App Store Connect Configuration

- **App Name**: Display name in App Store
- **Subtitle**: Brief description (30 characters)
- **Category**: Primary and secondary categories
- **Content Rights**: Usage rights information
- **Age Rating**: Complete rating questionnaire
- **App Review Information**: Demo account and notes

### Step 4: Build and Upload

#### Using EAS Build

```bash
# Build for App Store
eas build --platform ios --profile production

# Submit to App Store
eas submit --platform ios --profile production
```

#### Manual Upload

```bash
# Build for App Store
eas build --platform ios --profile production

# Download build
eas build:download [BUILD_ID]

# Upload to App Store Connect
# Use Xcode or Application Loader
```

### Step 5: App Store Connect Configuration

#### App Information

- **Name**: My Awesome App
- **Subtitle**: Task Management Made Easy
- **Category**: Productivity
- **Secondary Category**: Business
- **Content Rights**: Usage rights information
- **Age Rating**: 4+ (or appropriate rating)

#### App Description

```
# App Description

## What's New
- New task management features
- Improved user interface
- Bug fixes and performance improvements

## Description
My Awesome App is a comprehensive task management solution that helps you organize your daily activities, set reminders, and track your progress. With an intuitive interface and powerful features, staying productive has never been easier.

## Key Features
- ✅ Task creation and management
- 🔔 Smart reminders and notifications
- 📊 Progress tracking and analytics
- 🔄 Sync across all your devices
- 🎨 Customizable themes and layouts
- 📱 Works offline and online

## Privacy
We respect your privacy and never share your personal data. All your tasks and information are stored securely on your device and can be synced to your iCloud account.

## Support
Need help? Contact us at support@yourapp.com or visit our website at https://yourapp.com/support
```

#### Keywords

```
productivity, task management, organization, reminders, todo, planner, efficiency, mobile, app
```

#### Screenshots

- **iPhone 6.7"**: 1290x2796 pixels
- **iPhone 6.5"**: 1242x2688 pixels
- **iPhone 5.5"**: 1242x2208 pixels
- **iPad Pro 12.9"**: 2048x2732 pixels
- **iPad Pro 11"**: 1668x2388 pixels

### Step 6: App Review Submission

#### Review Information

- **Demo Account**: Username and password for testing
- **Review Notes**: Special instructions for reviewers
- **Contact Information**: Your contact details
- **Review Guidelines**: Any specific requirements

#### Submit for Review

1. **Review All Information** - Verify all details are correct
2. **Submit for Review** - Click "Submit for Review"
3. **Wait for Review** - Typically 24-48 hours
4. **Monitor Status** - Check review status regularly

---

## Google Play Store Submission (Android)

### Step 1: Google Play Console Setup

#### Create App

1. **Login to Google Play Console**

   - Go to [play.google.com/console](https://play.google.com/console)
   - Sign in with your Google account

2. **Create New App**
   - Click "Create app"
   - Fill in app information:
     - **App name**: Your app name
     - **Default language**: English (or your primary language)
     - **App or game**: App
     - **Free or paid**: Free or Paid
     - **Declarations**: Complete required declarations

### Step 2: App Information Configuration

#### App Information

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
      ]
    }
  }
}
```

#### Google Play Console Configuration

- **App name**: Display name in Play Store
- **Short description**: Brief description (80 characters)
- **Full description**: Detailed description (4000 characters)
- **Category**: App category
- **Content rating**: Complete rating questionnaire
- **Target audience**: Age and content targeting

### Step 3: Build and Upload

#### Using EAS Build

```bash
# Build for Play Store
eas build --platform android --profile production

# Submit to Play Store
eas submit --platform android --profile production
```

#### Manual Upload

```bash
# Build for Play Store
eas build --platform android --profile production

# Download build
eas build:download [BUILD_ID]

# Upload to Play Console
# Use Play Console or Google Play Developer API
```

### Step 4: Play Console Configuration

#### App Information

- **App name**: My Awesome App
- **Short description**: Task Management Made Easy
- **Full description**: Detailed app description
- **Category**: Productivity
- **Content rating**: Everyone (or appropriate rating)
- **Target audience**: Adults 18+ (or appropriate)

#### App Description

```
# My Awesome App

## Short Description
Task Management Made Easy

## Full Description
My Awesome App is a comprehensive task management solution that helps you organize your daily activities, set reminders, and track your progress. With an intuitive interface and powerful features, staying productive has never been easier.

## Key Features
- ✅ Task creation and management
- 🔔 Smart reminders and notifications
- 📊 Progress tracking and analytics
- 🔄 Sync across all your devices
- 🎨 Customizable themes and layouts
- 📱 Works offline and online

## Privacy
We respect your privacy and never share your personal data. All your tasks and information are stored securely on your device and can be synced to your Google account.

## Support
Need help? Contact us at support@yourapp.com or visit our website at https://yourapp.com/support
```

#### Graphics

- **App icon**: 512x512 pixels
- **Feature graphic**: 1024x500 pixels
- **Screenshots**: Phone and tablet screenshots
- **TV banner**: 1280x720 pixels (if applicable)

### Step 5: Content Rating and Compliance

#### Content Rating

1. **Complete Rating Questionnaire**

   - Answer all questions honestly
   - Provide accurate content descriptions
   - Submit for rating

2. **Target Audience**
   - Select appropriate age range
   - Choose content categories
   - Set content filters

#### Compliance

- **Privacy Policy**: Required for most apps
- **Data Safety**: Complete data safety section
- **App Permissions**: Justify all permissions
- **Content Guidelines**: Ensure compliance

### Step 6: Release Management

#### Release Types

- **Internal Testing**: Test with internal team
- **Closed Testing**: Test with limited users
- **Open Testing**: Test with public users
- **Production**: Public release

#### Release Process

1. **Upload APK/AAB** - Upload your build
2. **Configure Release** - Set release notes and details
3. **Review Release** - Verify all information
4. **Publish Release** - Make it available to users

---

## EAS Submit Automation

### EAS Submit Configuration

#### eas.json Configuration

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

#### iOS Submission

```bash
# Submit to App Store
eas submit --platform ios --profile production

# Submit with specific build
eas submit --platform ios --id [BUILD_ID]

# Submit with non-interactive mode
eas submit --platform ios --non-interactive
```

#### Android Submission

```bash
# Submit to Play Store
eas submit --platform android --profile production

# Submit to specific track
eas submit --platform android --track internal

# Submit with specific build
eas submit --platform android --id [BUILD_ID]
```

### Service Account Setup (Android)

#### Create Service Account

1. **Go to Google Cloud Console**

   - Visit [console.cloud.google.com](https://console.cloud.google.com)
   - Select your project

2. **Create Service Account**

   - Go to IAM & Admin → Service Accounts
   - Click "Create Service Account"
   - Fill in details and create

3. **Generate Key**

   - Click on service account
   - Go to Keys tab
   - Click "Add Key" → "Create new key"
   - Download JSON key file

4. **Grant Permissions**
   - Add service account to Play Console
   - Grant appropriate permissions

---

## Store Listing Optimization

### App Store Optimization (ASO)

#### Keywords Optimization

- **Primary Keywords**: Most important terms
- **Secondary Keywords**: Supporting terms
- **Long-tail Keywords**: Specific phrases
- **Competitor Analysis**: Research competitor keywords
- **Keyword Density**: Use keywords naturally

#### Title Optimization

- **App Name**: Include primary keyword
- **Subtitle**: Use secondary keywords
- **Character Limits**: Stay within limits
- **Readability**: Make it easy to read
- **Uniqueness**: Stand out from competitors

#### Description Optimization

- **Hook**: Compelling opening sentence
- **Features**: List key features with benefits
- **Social Proof**: Include testimonials or ratings
- **Call to Action**: Encourage downloads
- **Keywords**: Include relevant keywords naturally

### Visual Assets Optimization

#### App Icon

- **Design**: Simple and recognizable
- **Colors**: Use brand colors
- **Text**: Avoid text in icon
- **Testing**: Test on different backgrounds
- **Consistency**: Match app design

#### Screenshots

- **First Screenshot**: Most important feature
- **Feature Highlights**: Show key features
- **User Interface**: Display clean UI
- **Text Overlays**: Add descriptive text
- **Device Frames**: Use appropriate device frames

---

## Review Process

### App Store Review

#### Review Timeline

- **Standard Review**: 24-48 hours
- **Expedited Review**: 2-4 hours (for critical bugs)
- **Rejected Apps**: 24-48 hours for resubmission

#### Common Rejection Reasons

- **Guideline Violations**: App Store guidelines
- **Metadata Issues**: Incorrect app information
- **Technical Issues**: App crashes or bugs
- **Design Issues**: Poor user experience
- **Content Issues**: Inappropriate content

#### Review Guidelines

- **Follow Guidelines**: Read and follow all guidelines
- **Test Thoroughly**: Test on multiple devices
- **Provide Demo Account**: Include test credentials
- **Clear Instructions**: Provide clear review notes
- **Be Responsive**: Respond to review feedback

### Google Play Review

#### Review Timeline

- **Standard Review**: 1-3 days
- **New Apps**: May take longer
- **Updates**: Usually faster
- **Rejected Apps**: 24-48 hours for resubmission

#### Common Rejection Reasons

- **Policy Violations**: Play Store policies
- **Technical Issues**: App crashes or bugs
- **Metadata Issues**: Incorrect app information
- **Content Issues**: Inappropriate content
- **Security Issues**: Security vulnerabilities

#### Review Guidelines

- **Follow Policies**: Read and follow all policies
- **Test Thoroughly**: Test on multiple devices
- **Provide Information**: Complete all required information
- **Be Honest**: Provide accurate descriptions
- **Be Responsive**: Respond to review feedback

---

## Post-Submission Management

### App Store Connect Management

#### App Updates

```bash
# Build new version
eas build --platform ios --profile production

# Submit update
eas submit --platform ios --profile production
```

#### Release Management

- **Version Control**: Increment version numbers
- **Release Notes**: Write compelling release notes
- **Phased Rollout**: Gradual release to users
- **Emergency Updates**: Quick fixes for critical issues

#### Analytics and Monitoring

- **App Analytics**: Track downloads and usage
- **Crash Reports**: Monitor app crashes
- **User Feedback**: Respond to user reviews
- **Performance**: Monitor app performance

### Google Play Console Management

#### App Updates

```bash
# Build new version
eas build --platform android --profile production

# Submit update
eas submit --platform android --profile production
```

#### Release Management

- **Version Control**: Increment version codes
- **Release Notes**: Write compelling release notes
- **Staged Rollout**: Gradual release to users
- **Emergency Updates**: Quick fixes for critical issues

#### Analytics and Monitoring

- **Play Console Analytics**: Track downloads and usage
- **Crash Reports**: Monitor app crashes
- **User Feedback**: Respond to user reviews
- **Performance**: Monitor app performance

---

## Troubleshooting

### Common Submission Issues

#### iOS Issues

```bash
# Check build status
eas build:list --platform ios

# View build logs
eas build:logs [BUILD_ID]

# Check app store connect
# Verify app information and status
```

#### Android Issues

```bash
# Check build status
eas build:list --platform android

# View build logs
eas build:logs [BUILD_ID]

# Check play console
# Verify app information and status
```

### Build Issues

#### Build Failures

```bash
# Retry build
eas build --platform all --profile production --retry

# Clear build cache
eas build --platform all --profile production --clear-cache

# Check build configuration
eas build:configure
```

#### Upload Issues

```bash
# Check upload status
eas submit:list

# Retry upload
eas submit --platform all --profile production --retry

# Check credentials
eas credentials:list
```

### Review Issues

#### App Rejection

1. **Read Rejection Reason** - Understand why app was rejected
2. **Fix Issues** - Address all mentioned problems
3. **Test Thoroughly** - Test on multiple devices
4. **Resubmit** - Submit updated version
5. **Monitor Status** - Track review progress

#### Review Delays

1. **Check Status** - Monitor review status
2. **Contact Support** - Reach out if needed
3. **Provide Information** - Supply additional details
4. **Be Patient** - Allow time for review

---

## Best Practices

### Pre-Submission

1. **Test Thoroughly** - Test on multiple devices and OS versions
2. **Follow Guidelines** - Read and follow all store guidelines
3. **Prepare Assets** - Create all required graphics and descriptions
4. **Review Metadata** - Verify all app information is correct
5. **Check Compliance** - Ensure app meets all requirements

### During Submission

1. **Provide Clear Information** - Write clear descriptions and notes
2. **Include Demo Account** - Provide test credentials if needed
3. **Be Honest** - Provide accurate information
4. **Follow Instructions** - Complete all required steps
5. **Double-Check** - Verify all information before submitting

### Post-Submission

1. **Monitor Status** - Check review progress regularly
2. **Respond Quickly** - Address any review feedback promptly
3. **Prepare Updates** - Plan for future updates
4. **Engage Users** - Respond to user reviews and feedback
5. **Monitor Performance** - Track app performance and analytics

---

## Quick Reference

### Essential Commands

```bash
# Build for production
eas build --platform all --profile production

# Submit to app stores
eas submit --platform all --profile production

# Check build status
eas build:list

# View build logs
eas build:logs [BUILD_ID]

# Check submission status
eas submit:list
```

### Store Requirements

| Requirement        | iOS            | Android        |
| ------------------ | -------------- | -------------- |
| **App Icon**       | 1024x1024      | 512x512        |
| **Screenshots**    | Multiple sizes | Phone + Tablet |
| **Description**    | 4000 chars     | 4000 chars     |
| **Privacy Policy** | Required       | Required       |
| **Age Rating**     | Required       | Required       |
| **Content Rating** | Required       | Required       |

---

**Your app is now ready for the App Store and Google Play Store! Submit and reach millions of users! 🚀**
