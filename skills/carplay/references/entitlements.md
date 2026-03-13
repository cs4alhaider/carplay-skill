# Entitlements & Provisioning

This reference covers how to request, configure, and use CarPlay app entitlements.

## Table of Contents

1. [Requesting an Entitlement](#requesting-an-entitlement)
2. [Provisioning Profile Setup](#provisioning-profile-setup)
3. [Xcode Configuration](#xcode-configuration)
4. [Entitlement Keys Reference](#entitlement-keys-reference)
5. [Deprecated Entitlements](#deprecated-entitlements)

---

## Requesting an Entitlement

1. Go to [developer.apple.com/carplay](http://developer.apple.com/carplay).
2. Provide information about your app, including the entitlement category.
3. Agree to the **CarPlay Entitlement Addendum**.
4. Apple reviews your request. If approved, the entitlement is assigned to your Apple Developer account.

## Provisioning Profile Setup

After receiving the entitlement:

1. Log in at [developer.apple.com/account](https://developer.apple.com/account/).
2. Under **Certificates, IDs & Profiles**, select **Identifiers**.
3. Select your App ID (or create a new one).
4. Enable all necessary CarPlay app entitlements.
5. Click **Save**.
6. Go to **Provisioning Profiles** and create a new profile for the App ID.

Import the provisioning profile into Xcode. Both Xcode and Simulator require a provisioning profile that supports CarPlay.

## Xcode Configuration

Create (or update) an `Entitlements.plist` in your project. Add your entitlement key as a boolean:

```xml
<!-- Example: Audio app -->
<key>com.apple.developer.carplay-audio</key>
<true/>

<!-- Example: Navigation app -->
<key>com.apple.developer.carplay-maps</key>
<true/>

<!-- Example: Combined EV + Fueling app -->
<key>com.apple.developer.carplay-charging</key>
<true/>
<key>com.apple.developer.carplay-fueling</key>
<true/>
```

In Xcode:

1. Under **Signing & Capabilities**, turn off **Automatically manage signing**.
2. Under **Build Settings**, set **Code Signing Entitlements** to the path of your `Entitlements.plist`.

> **Important**: Once a CarPlay entitlement is added, your app icon appears on every CarPlay home screen. You cannot selectively show or hide it for certain users. Only publish with CarPlay support when you are ready.

## Entitlement Keys Reference

| Category | Entitlement Key | Min iOS |
|---|---|---|
| Audio (CarPlay framework) | `com.apple.developer.carplay-audio` | iOS 14 |
| Communication | `com.apple.developer.carplay-communication` | iOS 14 |
| Driving Task | `com.apple.developer.carplay-driving-task` | iOS 16 |
| EV Charging | `com.apple.developer.carplay-charging` | iOS 14 |
| Fueling | `com.apple.developer.carplay-fueling` | iOS 16 |
| Navigation | `com.apple.developer.carplay-maps` | iOS 12 |
| Parking | `com.apple.developer.carplay-parking` | iOS 14 |
| Public Safety | `com.apple.developer.carplay-public-safety` | iOS 14 |
| Quick Food Ordering | `com.apple.developer.carplay-quick-ordering` | iOS 14 |
| Voice-Based Conversational | `com.apple.developer.carplay-voice-based-conversation` | iOS 26.4 |

EV charging and fueling entitlements may be combined in a single app.

## Deprecated Entitlements

### Audio (Media Player Framework) — Deprecated

If your app must support iOS 13 and earlier, use the Media Player framework with:

```
com.apple.developer.playable-content
```

On iOS 14+, the CarPlay framework takes precedence if both are present. The Media Player approach works on later iOS versions but the UI is not customizable.

### Communication (Legacy) — Deprecated

For iOS 13 and earlier compatibility, also include:

```
com.apple.developer.carplay-messaging   <!-- for messaging -->
com.apple.developer.carplay-calling     <!-- for VoIP -->
```

Apps without the CarPlay framework entitlement still work on later iOS versions, but UI is not customizable.
