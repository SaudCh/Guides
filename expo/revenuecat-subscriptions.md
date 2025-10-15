# RevenueCat Subscriptions in React Native Expo (2025 Guide)

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup and Configuration](#setup-and-configuration)
- [Installation](#installation)
- [RevenueCat Dashboard Configuration](#revenuecat-dashboard-configuration)
- [Implementation](#implementation)
- [Restore Purchases](#restore-purchases)
- [Checking Subscription Status](#checking-subscription-status)
- [Handling Subscription Events](#handling-subscription-events)
- [Testing](#testing)
- [Best Practices](#best-practices)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Migration Notes (2025)](#migration-notes-2025)

## Overview

RevenueCat is a powerful subscription management platform that simplifies in-app purchases (IAP) for mobile applications. As of 2025, RevenueCat supports:

- **Cross-platform subscriptions** (iOS, Android, Web)
- **Real-time subscription analytics**
- **Webhook integrations**
- **A/B testing for pricing**
- **Server-side receipt validation**
- **Customer support tools**
- **Apple App Store and Google Play Store integration**
- **Stripe for web subscriptions**

### Why Use RevenueCat?

1. **Single API** for iOS and Android subscriptions
2. **Server-side receipt validation** prevents fraud
3. **Real-time webhooks** for subscription events
4. **Analytics dashboard** with MRR, churn, and retention metrics
5. **No backend required** - RevenueCat handles everything
6. **Free tier** available (up to $2.5k monthly tracked revenue)

## Prerequisites

### 1. Expo Requirements

**Important:** As of 2025, RevenueCat works with both Expo managed workflow (using development builds) and bare workflow.

- **Expo SDK 50+** (recommended: SDK 51 or 52)
- **Development Build** or **Bare Workflow** (Expo Go does NOT support in-app purchases)
- **EAS Build** for creating development and production builds

### 2. App Store Setup

#### iOS (App Store Connect)

- Active Apple Developer Account ($99/year)
- App created in App Store Connect
- Paid Applications Agreement signed
- Bank and tax information completed
- Subscriptions/products created in App Store Connect

#### Android (Google Play Console)

- Google Play Developer Account ($25 one-time fee)
- App created in Google Play Console
- Merchant account set up
- Products/subscriptions created in Google Play Console

### 3. RevenueCat Account

- Free account at [revenuecat.com](https://www.revenuecat.com)
- Project created for your app
- API keys generated

## Setup and Configuration

### Step 1: Install Dependencies

```bash
# Install RevenueCat SDK
npx expo install react-native-purchases

# For Expo SDK 50+, the package is compatible out of the box
```

**Package versions (2025):**

- `react-native-purchases`: v7.x or higher
- Compatible with Expo SDK 50, 51, 52

### Step 2: Create Development Build

Since in-app purchases don't work in Expo Go, create a development build:

```bash
# Install EAS CLI if you haven't
npm install -g eas-cli

# Login to Expo
eas login

# Configure your project
eas build:configure

# Create development build for iOS
eas build --profile development --platform ios

# Create development build for Android
eas build --profile development --platform android
```

### Step 3: Configure app.json/app.config.js

Add necessary configurations for in-app purchases:

```json
{
  "expo": {
    "name": "YourApp",
    "slug": "your-app",
    "ios": {
      "bundleIdentifier": "com.yourcompany.yourapp",
      "supportsTablet": true,
      "config": {
        "usesNonExemptEncryption": false
      }
    },
    "android": {
      "package": "com.yourcompany.yourapp",
      "permissions": ["com.android.vending.BILLING"]
    },
    "plugins": [
      [
        "react-native-purchases",
        {
          "enablePurchaseListener": true
        }
      ]
    ]
  }
}
```

## RevenueCat Dashboard Configuration

### 1. Create a Project

1. Go to [app.revenuecat.com](https://app.revenuecat.com)
2. Click **+ New Project**
3. Enter project name and select currency

### 2. Configure iOS App

1. In your project, click **Configure** under iOS
2. Enter your **Bundle ID** (must match App Store Connect)
3. Upload **In-App Purchase Key (.p8 file)** from App Store Connect:
   - Go to App Store Connect → Users and Access → Keys
   - Generate In-App Purchase key
   - Download and upload to RevenueCat

### 3. Configure Android App

1. Click **Configure** under Android
2. Enter your **Package Name** (must match Google Play Console)
3. Upload **Service Account JSON**:
   - Go to Google Play Console → Setup → API access
   - Create or use existing service account
   - Grant "Financial data" permissions
   - Download JSON and upload to RevenueCat

### 4. Create Products

1. In RevenueCat dashboard, go to **Products**
2. Click **+ New Product**
3. Enter product details:
   - **Product ID**: Must match App Store/Play Store product ID
   - **Type**: Subscription or One-time purchase
   - **Description**: Internal note

### 5. Create Offerings

Offerings are how you group products for display in your app:

1. Go to **Offerings**
2. Click **+ New Offering**
3. Name it (e.g., "default", "premium", "seasonal")
4. Add packages (e.g., monthly, annual)
5. Set one as **Current Offering**

**Example Offering Structure:**

```
Offering: "default"
  └─ Package: "monthly" → monthly_subscription
  └─ Package: "annual" → annual_subscription
  └─ Package: "lifetime" → lifetime_purchase
```

## Implementation

### Initialize RevenueCat

Create a service file for RevenueCat operations:

**`src/services/RevenueCatService.js`**

```javascript
import Purchases, { LOG_LEVEL } from "react-native-purchases";
import { Platform } from "react-native";

// API Keys from RevenueCat dashboard
const REVENUECAT_API_KEY_IOS = "appl_xxxxxxxxxxxxxxx";
const REVENUECAT_API_KEY_ANDROID = "goog_xxxxxxxxxxxxxxx";

class RevenueCatService {
  constructor() {
    this.isConfigured = false;
  }

  /**
   * Initialize RevenueCat SDK
   * Call this once when app starts
   */
  async initialize(userId = null) {
    try {
      if (this.isConfigured) {
        console.log("RevenueCat already configured");
        return;
      }

      // Set debug logs for development (remove in production)
      if (__DEV__) {
        Purchases.setLogLevel(LOG_LEVEL.DEBUG);
      }

      // Configure with platform-specific API key
      const apiKey = Platform.select({
        ios: REVENUECAT_API_KEY_IOS,
        android: REVENUECAT_API_KEY_ANDROID,
      });

      Purchases.configure({ apiKey });

      // Identify user if you have a user ID
      if (userId) {
        await Purchases.logIn(userId);
      }

      this.isConfigured = true;
      console.log("RevenueCat configured successfully");
    } catch (error) {
      console.error("Error configuring RevenueCat:", error);
      throw error;
    }
  }

  /**
   * Get available offerings
   * Returns products organized by offerings
   */
  async getOfferings() {
    try {
      const offerings = await Purchases.getOfferings();

      if (
        offerings.current !== null &&
        offerings.current.availablePackages.length !== 0
      ) {
        // Display current offering with array of packages
        return {
          current: offerings.current,
          all: offerings.all,
        };
      } else {
        console.warn("No offerings available");
        return null;
      }
    } catch (error) {
      console.error("Error getting offerings:", error);
      throw error;
    }
  }

  /**
   * Purchase a package
   * @param {Package} packageToPurchase - Package from offerings
   */
  async purchasePackage(packageToPurchase) {
    try {
      const { customerInfo, productIdentifier } =
        await Purchases.purchasePackage(packageToPurchase);

      // Check if the user now has an active entitlement
      if (typeof customerInfo.entitlements.active["pro"] !== "undefined") {
        // Grant access to premium features
        return {
          success: true,
          customerInfo,
          productIdentifier,
        };
      }

      return {
        success: false,
        customerInfo,
        productIdentifier,
      };
    } catch (error) {
      if (
        error.code === Purchases.PURCHASES_ERROR_CODE.PURCHASE_CANCELLED_ERROR
      ) {
        // User cancelled the purchase
        console.log("Purchase cancelled by user");
        return { success: false, cancelled: true };
      } else {
        // Handle other errors
        console.error("Error purchasing package:", error);
        throw error;
      }
    }
  }

  /**
   * Restore previous purchases
   * Important for users who reinstalled the app or got a new device
   */
  async restorePurchases() {
    try {
      const customerInfo = await Purchases.restorePurchases();

      // Check if user has active entitlements
      const hasActiveSubscription =
        Object.keys(customerInfo.entitlements.active).length > 0;

      return {
        success: true,
        hasActiveSubscription,
        customerInfo,
      };
    } catch (error) {
      console.error("Error restoring purchases:", error);
      throw error;
    }
  }

  /**
   * Get current customer info
   * Use this to check subscription status
   */
  async getCustomerInfo() {
    try {
      const customerInfo = await Purchases.getCustomerInfo();
      return customerInfo;
    } catch (error) {
      console.error("Error getting customer info:", error);
      throw error;
    }
  }

  /**
   * Check if user has active subscription
   * @param {string} entitlementId - Entitlement identifier from RevenueCat
   */
  async hasActiveSubscription(entitlementId = "pro") {
    try {
      const customerInfo = await this.getCustomerInfo();
      return (
        typeof customerInfo.entitlements.active[entitlementId] !== "undefined"
      );
    } catch (error) {
      console.error("Error checking subscription:", error);
      return false;
    }
  }

  /**
   * Log in user (for cross-platform subscription syncing)
   * @param {string} userId - Your app's user ID
   */
  async logIn(userId) {
    try {
      const { customerInfo } = await Purchases.logIn(userId);
      return customerInfo;
    } catch (error) {
      console.error("Error logging in:", error);
      throw error;
    }
  }

  /**
   * Log out user
   */
  async logOut() {
    try {
      const customerInfo = await Purchases.logOut();
      return customerInfo;
    } catch (error) {
      console.error("Error logging out:", error);
      throw error;
    }
  }

  /**
   * Set custom attributes for the user
   * Useful for analytics and customer support
   */
  async setAttributes(attributes) {
    try {
      await Purchases.setAttributes(attributes);
    } catch (error) {
      console.error("Error setting attributes:", error);
    }
  }

  /**
   * Setup listener for subscription status changes
   * This is called automatically by RevenueCat when subscription status changes
   */
  setupPurchaseListener(callback) {
    Purchases.addCustomerInfoUpdateListener((customerInfo) => {
      // Handle subscription status changes
      callback(customerInfo);
    });
  }
}

export default new RevenueCatService();
```

### Initialize in App.js/App.tsx

```javascript
import React, { useEffect } from "react";
import { View, Text } from "react-native";
import RevenueCatService from "./src/services/RevenueCatService";

export default function App() {
  useEffect(() => {
    // Initialize RevenueCat when app starts
    const initRevenueCat = async () => {
      try {
        await RevenueCatService.initialize();

        // Optional: Setup listener for subscription changes
        RevenueCatService.setupPurchaseListener((customerInfo) => {
          console.log("Subscription status changed:", customerInfo);
          // Update your app state here
        });
      } catch (error) {
        console.error("Failed to initialize RevenueCat:", error);
      }
    };

    initRevenueCat();
  }, []);

  return (
    <View>
      <Text>Your App</Text>
    </View>
  );
}
```

### Paywall Component

Create a full-featured paywall screen:

**`src/screens/PaywallScreen.js`**

```javascript
import React, { useState, useEffect } from "react";
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  ActivityIndicator,
  Alert,
  ScrollView,
  Platform,
} from "react-native";
import RevenueCatService from "../services/RevenueCatService";

export default function PaywallScreen({ navigation }) {
  const [offerings, setOfferings] = useState(null);
  const [selectedPackage, setSelectedPackage] = useState(null);
  const [loading, setLoading] = useState(true);
  const [purchasing, setPurchasing] = useState(false);
  const [restoring, setRestoring] = useState(false);

  useEffect(() => {
    loadOfferings();
  }, []);

  const loadOfferings = async () => {
    try {
      setLoading(true);
      const result = await RevenueCatService.getOfferings();

      if (result && result.current) {
        setOfferings(result.current);
        // Pre-select the first package
        if (result.current.availablePackages.length > 0) {
          setSelectedPackage(result.current.availablePackages[0]);
        }
      } else {
        Alert.alert("Error", "No subscription packages available");
      }
    } catch (error) {
      Alert.alert("Error", "Failed to load subscriptions");
    } finally {
      setLoading(false);
    }
  };

  const handlePurchase = async () => {
    if (!selectedPackage) {
      Alert.alert("Error", "Please select a subscription package");
      return;
    }

    try {
      setPurchasing(true);
      const result = await RevenueCatService.purchasePackage(selectedPackage);

      if (result.cancelled) {
        // User cancelled, do nothing
        return;
      }

      if (result.success) {
        Alert.alert(
          "Success!",
          "Your subscription is now active. Enjoy premium features!",
          [
            {
              text: "OK",
              onPress: () => navigation.goBack(),
            },
          ]
        );
      } else {
        Alert.alert("Error", "Failed to activate subscription");
      }
    } catch (error) {
      Alert.alert("Error", error.message || "Failed to complete purchase");
    } finally {
      setPurchasing(false);
    }
  };

  const handleRestore = async () => {
    try {
      setRestoring(true);
      const result = await RevenueCatService.restorePurchases();

      if (result.hasActiveSubscription) {
        Alert.alert(
          "Success!",
          "Your purchases have been restored successfully!",
          [
            {
              text: "OK",
              onPress: () => navigation.goBack(),
            },
          ]
        );
      } else {
        Alert.alert(
          "No Purchases Found",
          "We could not find any previous purchases associated with your account."
        );
      }
    } catch (error) {
      Alert.alert("Error", "Failed to restore purchases. Please try again.");
    } finally {
      setRestoring(false);
    }
  };

  const formatPrice = (pkg) => {
    const product = pkg.product;
    return `${product.priceString}/${getPeriodString(product)}`;
  };

  const getPeriodString = (product) => {
    // Parse subscription period
    if (product.subscriptionPeriod) {
      const period = product.subscriptionPeriod;
      // Format based on period (P1M = monthly, P1Y = yearly, etc.)
      if (period.includes("M")) return "month";
      if (period.includes("Y")) return "year";
      if (period.includes("W")) return "week";
    }
    return "period";
  };

  const getPackageTitle = (identifier) => {
    // Package identifiers from RevenueCat
    const titles = {
      $rc_monthly: "Monthly",
      $rc_annual: "Annual",
      $rc_lifetime: "Lifetime",
      monthly: "Monthly",
      annual: "Annual",
      yearly: "Annual",
      lifetime: "Lifetime",
    };
    return titles[identifier] || identifier;
  };

  const getSavingsText = (pkg) => {
    // Calculate savings for annual vs monthly
    if (
      pkg.identifier.includes("annual") ||
      pkg.identifier.includes("yearly")
    ) {
      return "Save 40%";
    }
    return null;
  };

  if (loading) {
    return (
      <View style={styles.centered}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Loading subscriptions...</Text>
      </View>
    );
  }

  if (!offerings) {
    return (
      <View style={styles.centered}>
        <Text style={styles.errorText}>No subscriptions available</Text>
        <TouchableOpacity style={styles.retryButton} onPress={loadOfferings}>
          <Text style={styles.retryButtonText}>Retry</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <ScrollView
      style={styles.container}
      contentContainerStyle={styles.contentContainer}
    >
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.title}>Go Premium</Text>
        <Text style={styles.subtitle}>Unlock all features and content</Text>
      </View>

      {/* Features */}
      <View style={styles.featuresContainer}>
        <FeatureItem text="Ad-free experience" />
        <FeatureItem text="Access to all premium content" />
        <FeatureItem text="Priority customer support" />
        <FeatureItem text="Early access to new features" />
      </View>

      {/* Packages */}
      <View style={styles.packagesContainer}>
        {offerings.availablePackages.map((pkg) => {
          const isSelected = selectedPackage?.identifier === pkg.identifier;
          const savings = getSavingsText(pkg);

          return (
            <TouchableOpacity
              key={pkg.identifier}
              style={[
                styles.packageCard,
                isSelected && styles.packageCardSelected,
              ]}
              onPress={() => setSelectedPackage(pkg)}
            >
              {savings && (
                <View style={styles.savingsBadge}>
                  <Text style={styles.savingsBadgeText}>{savings}</Text>
                </View>
              )}

              <View style={styles.packageHeader}>
                <Text style={styles.packageTitle}>
                  {getPackageTitle(pkg.identifier)}
                </Text>
                <View style={styles.radioButton}>
                  {isSelected && <View style={styles.radioButtonInner} />}
                </View>
              </View>

              <Text style={styles.packagePrice}>{formatPrice(pkg)}</Text>

              {pkg.product.introPrice && (
                <Text style={styles.trialText}>
                  {pkg.product.introPrice.periodNumberOfUnits}{" "}
                  {pkg.product.introPrice.periodUnit} free trial
                </Text>
              )}
            </TouchableOpacity>
          );
        })}
      </View>

      {/* Subscribe Button */}
      <TouchableOpacity
        style={[
          styles.subscribeButton,
          purchasing && styles.subscribeButtonDisabled,
        ]}
        onPress={handlePurchase}
        disabled={purchasing}
      >
        {purchasing ? (
          <ActivityIndicator color="#FFFFFF" />
        ) : (
          <Text style={styles.subscribeButtonText}>Subscribe Now</Text>
        )}
      </TouchableOpacity>

      {/* Restore Button */}
      <TouchableOpacity
        style={styles.restoreButton}
        onPress={handleRestore}
        disabled={restoring}
      >
        {restoring ? (
          <ActivityIndicator color="#007AFF" size="small" />
        ) : (
          <Text style={styles.restoreButtonText}>Restore Purchases</Text>
        )}
      </TouchableOpacity>

      {/* Terms */}
      <View style={styles.termsContainer}>
        <Text style={styles.termsText}>
          Subscription automatically renews unless auto-renew is turned off at
          least 24 hours before the end of the current period.
        </Text>
        <View style={styles.linksContainer}>
          <TouchableOpacity>
            <Text style={styles.linkText}>Terms of Service</Text>
          </TouchableOpacity>
          <Text style={styles.linkSeparator}>•</Text>
          <TouchableOpacity>
            <Text style={styles.linkText}>Privacy Policy</Text>
          </TouchableOpacity>
        </View>
      </View>
    </ScrollView>
  );
}

const FeatureItem = ({ text }) => (
  <View style={styles.featureItem}>
    <Text style={styles.featureCheckmark}>✓</Text>
    <Text style={styles.featureText}>{text}</Text>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#FFFFFF",
  },
  contentContainer: {
    padding: 20,
    paddingBottom: 40,
  },
  centered: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    backgroundColor: "#FFFFFF",
  },
  header: {
    alignItems: "center",
    marginTop: 20,
    marginBottom: 30,
  },
  title: {
    fontSize: 32,
    fontWeight: "bold",
    color: "#000000",
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: "#666666",
  },
  featuresContainer: {
    marginBottom: 30,
  },
  featureItem: {
    flexDirection: "row",
    alignItems: "center",
    marginBottom: 12,
  },
  featureCheckmark: {
    fontSize: 20,
    color: "#34C759",
    marginRight: 12,
    fontWeight: "bold",
  },
  featureText: {
    fontSize: 16,
    color: "#333333",
  },
  packagesContainer: {
    marginBottom: 20,
  },
  packageCard: {
    borderWidth: 2,
    borderColor: "#E5E5E5",
    borderRadius: 12,
    padding: 16,
    marginBottom: 12,
    position: "relative",
  },
  packageCardSelected: {
    borderColor: "#007AFF",
    backgroundColor: "#F0F8FF",
  },
  savingsBadge: {
    position: "absolute",
    top: -8,
    right: 16,
    backgroundColor: "#FF3B30",
    paddingHorizontal: 12,
    paddingVertical: 4,
    borderRadius: 12,
  },
  savingsBadgeText: {
    color: "#FFFFFF",
    fontSize: 12,
    fontWeight: "bold",
  },
  packageHeader: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
    marginBottom: 8,
  },
  packageTitle: {
    fontSize: 20,
    fontWeight: "600",
    color: "#000000",
  },
  radioButton: {
    width: 24,
    height: 24,
    borderRadius: 12,
    borderWidth: 2,
    borderColor: "#007AFF",
    justifyContent: "center",
    alignItems: "center",
  },
  radioButtonInner: {
    width: 12,
    height: 12,
    borderRadius: 6,
    backgroundColor: "#007AFF",
  },
  packagePrice: {
    fontSize: 18,
    color: "#333333",
    marginBottom: 4,
  },
  trialText: {
    fontSize: 14,
    color: "#34C759",
    fontWeight: "500",
  },
  subscribeButton: {
    backgroundColor: "#007AFF",
    borderRadius: 12,
    padding: 16,
    alignItems: "center",
    marginBottom: 16,
  },
  subscribeButtonDisabled: {
    opacity: 0.6,
  },
  subscribeButtonText: {
    color: "#FFFFFF",
    fontSize: 18,
    fontWeight: "600",
  },
  restoreButton: {
    padding: 12,
    alignItems: "center",
    marginBottom: 20,
  },
  restoreButtonText: {
    color: "#007AFF",
    fontSize: 16,
    fontWeight: "500",
  },
  termsContainer: {
    marginTop: 20,
    alignItems: "center",
  },
  termsText: {
    fontSize: 12,
    color: "#999999",
    textAlign: "center",
    marginBottom: 12,
    lineHeight: 18,
  },
  linksContainer: {
    flexDirection: "row",
    alignItems: "center",
  },
  linkText: {
    fontSize: 12,
    color: "#007AFF",
  },
  linkSeparator: {
    marginHorizontal: 8,
    color: "#999999",
  },
  loadingText: {
    marginTop: 12,
    fontSize: 16,
    color: "#666666",
  },
  errorText: {
    fontSize: 16,
    color: "#FF3B30",
    marginBottom: 20,
  },
  retryButton: {
    backgroundColor: "#007AFF",
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  retryButtonText: {
    color: "#FFFFFF",
    fontSize: 16,
    fontWeight: "600",
  },
});
```

## Restore Purchases

### Why Restore Purchases is Important

1. **App Store Guidelines**: Apple requires apps to provide a way to restore purchases
2. **User Experience**: Users who reinstall the app or switch devices need to restore access
3. **Account Management**: Allows users to restore purchases without repurchasing

### Implementation

The restore functionality is already included in the `RevenueCatService` above. Here's a detailed explanation:

```javascript
/**
 * Restore previous purchases
 * This syncs the user's purchases from App Store/Play Store with RevenueCat
 */
async restorePurchases() {
  try {
    // This fetches the latest receipt from App Store/Play Store
    // and updates RevenueCat's database
    const customerInfo = await Purchases.restorePurchases();

    // Check if user has any active entitlements
    const hasActiveSubscription = Object.keys(customerInfo.entitlements.active).length > 0;

    return {
      success: true,
      hasActiveSubscription,
      customerInfo,
    };
  } catch (error) {
    console.error('Error restoring purchases:', error);
    throw error;
  }
}
```

### Where to Add Restore Button

1. **Paywall Screen**: Primary location (shown in example above)
2. **Settings Screen**: For existing users
3. **Login/Signup Screen**: After user logs in

### Restore Button in Settings

**`src/screens/SettingsScreen.js`**

```javascript
import React, { useState } from "react";
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ActivityIndicator,
  Alert,
} from "react-native";
import RevenueCatService from "../services/RevenueCatService";

export default function SettingsScreen() {
  const [restoring, setRestoring] = useState(false);

  const handleRestorePurchases = async () => {
    Alert.alert(
      "Restore Purchases",
      "This will restore your previous purchases from your App Store or Google Play account.",
      [
        {
          text: "Cancel",
          style: "cancel",
        },
        {
          text: "Restore",
          onPress: async () => {
            try {
              setRestoring(true);
              const result = await RevenueCatService.restorePurchases();

              if (result.hasActiveSubscription) {
                Alert.alert(
                  "Success!",
                  "Your purchases have been restored successfully!"
                );
              } else {
                Alert.alert(
                  "No Purchases Found",
                  "We could not find any previous purchases associated with your account."
                );
              }
            } catch (error) {
              Alert.alert(
                "Error",
                "Failed to restore purchases. Please try again later."
              );
            } finally {
              setRestoring(false);
            }
          },
        },
      ]
    );
  };

  return (
    <View style={styles.container}>
      {/* Other settings items... */}

      <TouchableOpacity
        style={styles.settingsItem}
        onPress={handleRestorePurchases}
        disabled={restoring}
      >
        <Text style={styles.settingsItemText}>Restore Purchases</Text>
        {restoring && <ActivityIndicator size="small" color="#007AFF" />}
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#F2F2F7",
  },
  settingsItem: {
    backgroundColor: "#FFFFFF",
    padding: 16,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: "#C6C6C8",
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
  },
  settingsItemText: {
    fontSize: 17,
    color: "#007AFF",
  },
});
```

### Automatic Restore on App Launch

For better UX, automatically check subscription status when app launches:

```javascript
useEffect(() => {
  const checkSubscriptionStatus = async () => {
    try {
      // This automatically syncs with the store
      const customerInfo = await RevenueCatService.getCustomerInfo();

      const isSubscribed =
        Object.keys(customerInfo.entitlements.active).length > 0;

      // Update app state
      setIsSubscribed(isSubscribed);
    } catch (error) {
      console.error("Error checking subscription:", error);
    }
  };

  checkSubscriptionStatus();
}, []);
```

## Checking Subscription Status

### Method 1: Check Entitlements

```javascript
import RevenueCatService from "./services/RevenueCatService";

const checkPremiumAccess = async () => {
  try {
    const customerInfo = await RevenueCatService.getCustomerInfo();

    // Check for specific entitlement (configured in RevenueCat dashboard)
    const hasPremium =
      typeof customerInfo.entitlements.active["pro"] !== "undefined";

    if (hasPremium) {
      // User has active subscription
      console.log("User is subscribed");
    } else {
      // User does not have active subscription
      console.log("User is not subscribed");
    }
  } catch (error) {
    console.error("Error checking subscription:", error);
  }
};
```

### Method 2: Using React Context

Create a subscription context for app-wide access:

**`src/context/SubscriptionContext.js`**

```javascript
import React, { createContext, useState, useEffect, useContext } from "react";
import RevenueCatService from "../services/RevenueCatService";

const SubscriptionContext = createContext();

export const useSubscription = () => {
  const context = useContext(SubscriptionContext);
  if (!context) {
    throw new Error("useSubscription must be used within SubscriptionProvider");
  }
  return context;
};

export const SubscriptionProvider = ({ children }) => {
  const [isSubscribed, setIsSubscribed] = useState(false);
  const [subscriptionInfo, setSubscriptionInfo] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    initializeSubscription();
    setupListener();
  }, []);

  const initializeSubscription = async () => {
    try {
      await checkSubscriptionStatus();
    } finally {
      setLoading(false);
    }
  };

  const checkSubscriptionStatus = async () => {
    try {
      const customerInfo = await RevenueCatService.getCustomerInfo();
      const hasActiveSubscription =
        Object.keys(customerInfo.entitlements.active).length > 0;

      setIsSubscribed(hasActiveSubscription);
      setSubscriptionInfo(customerInfo);
    } catch (error) {
      console.error("Error checking subscription:", error);
      setIsSubscribed(false);
    }
  };

  const setupListener = () => {
    RevenueCatService.setupPurchaseListener((customerInfo) => {
      const hasActiveSubscription =
        Object.keys(customerInfo.entitlements.active).length > 0;
      setIsSubscribed(hasActiveSubscription);
      setSubscriptionInfo(customerInfo);
    });
  };

  const value = {
    isSubscribed,
    subscriptionInfo,
    loading,
    checkSubscriptionStatus,
  };

  return (
    <SubscriptionContext.Provider value={value}>
      {children}
    </SubscriptionContext.Provider>
  );
};
```

### Use in App.js

```javascript
import React from "react";
import { NavigationContainer } from "@react-navigation/native";
import { SubscriptionProvider } from "./src/context/SubscriptionContext";
import RevenueCatService from "./src/services/RevenueCatService";

export default function App() {
  useEffect(() => {
    RevenueCatService.initialize();
  }, []);

  return (
    <SubscriptionProvider>
      <NavigationContainer>{/* Your navigation */}</NavigationContainer>
    </SubscriptionProvider>
  );
}
```

### Use in Components

```javascript
import React from "react";
import { View, Text, Button } from "react-native";
import { useSubscription } from "../context/SubscriptionContext";

export default function PremiumFeatureScreen({ navigation }) {
  const { isSubscribed, loading } = useSubscription();

  if (loading) {
    return <Text>Loading...</Text>;
  }

  if (!isSubscribed) {
    return (
      <View>
        <Text>This is a premium feature</Text>
        <Button
          title="Subscribe Now"
          onPress={() => navigation.navigate("Paywall")}
        />
      </View>
    );
  }

  return (
    <View>
      <Text>Premium Content Here!</Text>
    </View>
  );
}
```

## Handling Subscription Events

### Customer Info Update Listener

RevenueCat provides a listener that fires when subscription status changes:

```javascript
import Purchases from "react-native-purchases";

// Setup listener
Purchases.addCustomerInfoUpdateListener((customerInfo) => {
  // Called whenever subscription status changes

  // Check entitlements
  const activeEntitlements = customerInfo.entitlements.active;

  if (Object.keys(activeEntitlements).length > 0) {
    // User has active subscriptions
    console.log("Active subscriptions:", activeEntitlements);
  } else {
    // User has no active subscriptions
    console.log("No active subscriptions");
  }

  // Access specific entitlement
  const proEntitlement = activeEntitlements["pro"];
  if (proEntitlement) {
    console.log("Pro entitlement expires:", proEntitlement.expirationDate);
  }
});
```

### Webhook Events

Configure webhooks in RevenueCat dashboard to receive server-side events:

**Events Available (2025):**

- `INITIAL_PURCHASE` - First purchase
- `RENEWAL` - Subscription renewed
- `CANCELLATION` - User cancelled (still active until period ends)
- `UNCANCELLATION` - User re-enabled auto-renew
- `NON_RENEWING_PURCHASE` - One-time purchase
- `EXPIRATION` - Subscription expired
- `BILLING_ISSUE` - Payment failed
- `PRODUCT_CHANGE` - User upgraded/downgraded
- `TRANSFER` - Subscription transferred (family sharing)

Example webhook handler (Node.js backend):

```javascript
app.post("/revenuecat-webhook", express.json(), (req, res) => {
  const event = req.body;

  switch (event.type) {
    case "INITIAL_PURCHASE":
      // Handle new subscription
      console.log(`New subscriber: ${event.app_user_id}`);
      break;

    case "RENEWAL":
      // Handle renewal
      console.log(`Subscription renewed: ${event.app_user_id}`);
      break;

    case "CANCELLATION":
      // Handle cancellation
      console.log(`Subscription cancelled: ${event.app_user_id}`);
      break;

    case "EXPIRATION":
      // Handle expiration
      console.log(`Subscription expired: ${event.app_user_id}`);
      // Revoke access
      break;

    case "BILLING_ISSUE":
      // Handle billing issue
      console.log(`Billing issue: ${event.app_user_id}`);
      // Send notification to user
      break;
  }

  res.status(200).send("OK");
});
```

## Testing

### Test Environment Setup

1. **iOS Testing:**

   - Use Sandbox Apple ID (created in App Store Connect)
   - Subscriptions auto-renew every few minutes in sandbox
   - Can test multiple times with same account

2. **Android Testing:**
   - Add test account emails in Google Play Console
   - Use test products for testing
   - Can cancel and repurchase easily

### Testing Subscriptions

```javascript
// Enable debug logs
import Purchases, { LOG_LEVEL } from "react-native-purchases";

if (__DEV__) {
  Purchases.setLogLevel(LOG_LEVEL.DEBUG);
}
```

### Testing Checklist

- [ ] Purchase monthly subscription
- [ ] Purchase annual subscription
- [ ] Cancel subscription (in iOS Settings / Play Store)
- [ ] Restore purchases after reinstalling app
- [ ] Test subscription renewal
- [ ] Test grace period (payment failed)
- [ ] Test free trial
- [ ] Test introductory pricing
- [ ] Test upgrade/downgrade
- [ ] Test cross-platform sync (iOS → Android)

### Simulator/Emulator Testing

**Important:** You must use a real device for in-app purchase testing. Simulator/emulator will NOT work.

### RevenueCat Sandbox Testing

RevenueCat automatically detects sandbox purchases. View them in:

- RevenueCat Dashboard → Customers → Search by user ID

## Best Practices

### 1. User Identification

```javascript
// When user logs in
await RevenueCatService.logIn(user.id);

// When user logs out
await RevenueCatService.logOut();
```

### 2. Attribution

Set attribution data for marketing analytics:

```javascript
// Set attribution
await Purchases.setAttributes({
  campaign: "summer_sale",
  source: "instagram",
  user_type: "premium_trial",
});
```

### 3. Error Handling

Always handle errors gracefully:

```javascript
try {
  await RevenueCatService.purchasePackage(package);
} catch (error) {
  if (error.code === Purchases.PURCHASES_ERROR_CODE.PURCHASE_CANCELLED_ERROR) {
    // User cancelled - don't show error
  } else if (
    error.code === Purchases.PURCHASES_ERROR_CODE.STORE_PROBLEM_ERROR
  ) {
    Alert.alert("Store Error", "Unable to connect to store. Please try again.");
  } else if (
    error.code === Purchases.PURCHASES_ERROR_CODE.PURCHASE_NOT_ALLOWED_ERROR
  ) {
    Alert.alert("Not Allowed", "Purchases are not allowed on this device.");
  } else {
    Alert.alert("Error", "Failed to complete purchase.");
  }
}
```

### 4. Graceful Degradation

Always provide fallback when RevenueCat is unavailable:

```javascript
const [isSubscribed, setIsSubscribed] = useState(false);

try {
  const customerInfo = await RevenueCatService.getCustomerInfo();
  setIsSubscribed(Object.keys(customerInfo.entitlements.active).length > 0);
} catch (error) {
  // Fallback: Allow limited access or cache last known state
  const cachedStatus = await AsyncStorage.getItem("subscription_status");
  setIsSubscribed(cachedStatus === "true");
}
```

### 5. Cache Subscription Status

```javascript
// Cache subscription status locally
await AsyncStorage.setItem("subscription_status", isSubscribed.toString());
await AsyncStorage.setItem("subscription_check_time", Date.now().toString());

// Check cache first, then verify with RevenueCat
const cachedStatus = await AsyncStorage.getItem("subscription_status");
const checkTime = await AsyncStorage.getItem("subscription_check_time");

// If cache is recent (< 1 hour), use it
if (checkTime && Date.now() - parseInt(checkTime) < 3600000) {
  return cachedStatus === "true";
}

// Otherwise, fetch from RevenueCat
const customerInfo = await RevenueCatService.getCustomerInfo();
// Update cache...
```

### 6. Handle Offline Mode

```javascript
import NetInfo from "@react-native-community/netinfo";

const checkSubscription = async () => {
  const netInfo = await NetInfo.fetch();

  if (!netInfo.isConnected) {
    // Use cached subscription status
    const cached = await AsyncStorage.getItem("subscription_status");
    return cached === "true";
  }

  // Online - fetch from RevenueCat
  return await RevenueCatService.hasActiveSubscription();
};
```

## Common Issues and Solutions

### Issue 1: "No products found"

**Solution:**

- Verify products are created in App Store Connect / Google Play Console
- Ensure products are approved (App Store) or published (Play Store)
- Check Bundle ID / Package Name matches exactly
- Wait 1-2 hours after creating products for them to propagate
- Ensure RevenueCat product IDs match store product IDs exactly

### Issue 2: "Purchase failed with unknown error"

**Solution:**

- Check device is signed in to App Store / Play Store
- Verify payment method is valid
- For iOS: Use Sandbox Apple ID, not production account
- Check RevenueCat dashboard for detailed error logs

### Issue 3: Restore purchases not working

**Solution:**

- Ensure user is signed in with same Apple ID / Google account
- Call `restorePurchases()` which syncs with store
- Check if purchases were made in sandbox (can't restore to production)
- For iOS: Check if App-Specific Password is required

### Issue 4: Subscription not showing as active after purchase

**Solution:**

- Check entitlement configuration in RevenueCat dashboard
- Verify product is assigned to correct entitlement
- Wait a few seconds for receipt validation
- Check RevenueCat customer dashboard for purchase status

### Issue 5: Cross-platform sync not working

**Solution:**

- Ensure user is logged in with `Purchases.logIn(userId)`
- Use same user ID across platforms
- Call `getCustomerInfo()` to force sync
- Check RevenueCat dashboard shows both platform purchases

### Issue 6: Build fails after adding react-native-purchases

**Solution:**

- Run `npx pod-install` for iOS
- Clear cache: `rm -rf node_modules && npm install`
- For Expo: Ensure using development build, not Expo Go
- Rebuild with `eas build`

## Migration Notes (2025)

### What's New in 2025

1. **RevenueCat SDK v7+:**

   - Improved TypeScript support
   - Better error handling
   - Simplified API surface
   - Enhanced customer info model

2. **Expo SDK 50+:**

   - Native module support in managed workflow with dev builds
   - Better plugin system
   - Faster build times

3. **App Store Changes:**

   - Family Sharing support for subscriptions
   - External purchase links allowed (with 27% commission)
   - New subscription management UI

4. **Google Play Changes:**
   - User choice billing (alternative payment methods)
   - Improved subscription management
   - Better grace period handling

### Breaking Changes from Older Versions

If migrating from older RevenueCat SDK:

```javascript
// OLD (pre-v6)
Purchases.setup("api_key", "user_id");

// NEW (v7+)
Purchases.configure({ apiKey: "api_key" });
await Purchases.logIn("user_id");
```

```javascript
// OLD
const offerings = await Purchases.getOfferings();
const monthly = offerings.current.monthly;
await Purchases.purchaseProduct(monthly.product);

// NEW
const offerings = await Purchases.getOfferings();
const monthlyPackage = offerings.current.availablePackages[0];
await Purchases.purchasePackage(monthlyPackage);
```

### Expo Config Changes

```javascript
// OLD (Expo SDK 48)
{
  "plugins": ["react-native-purchases"]
}

// NEW (Expo SDK 50+)
{
  "plugins": [
    [
      "react-native-purchases",
      {
        "enablePurchaseListener": true
      }
    ]
  ]
}
```

## Additional Resources

### Official Documentation

- [RevenueCat Docs](https://docs.revenuecat.com/)
- [React Native SDK](https://docs.revenuecat.com/docs/reactnative)
- [Expo Documentation](https://docs.expo.dev/)

### Tools

- [RevenueCat Dashboard](https://app.revenuecat.com/)
- [App Store Connect](https://appstoreconnect.apple.com/)
- [Google Play Console](https://play.google.com/console/)

### Community

- [RevenueCat Community](https://community.revenuecat.com/)
- [Expo Discord](https://chat.expo.dev/)
- [RevenueCat GitHub](https://github.com/RevenueCat/react-native-purchases)

---

## Quick Start Checklist

- [ ] Install `react-native-purchases`
- [ ] Create development build (not Expo Go)
- [ ] Set up RevenueCat account and configure iOS/Android apps
- [ ] Create products in App Store Connect / Google Play Console
- [ ] Create products and offerings in RevenueCat dashboard
- [ ] Initialize RevenueCat in your app
- [ ] Implement paywall screen
- [ ] Add restore purchases functionality
- [ ] Test purchases with sandbox accounts
- [ ] Implement subscription status checking
- [ ] Add customer info listener
- [ ] Test restore purchases
- [ ] Configure webhooks (optional but recommended)
- [ ] Submit app for review

---

**Last Updated:** October 2025
**SDK Version:** react-native-purchases v7.x
**Expo SDK:** 50, 51, 52
