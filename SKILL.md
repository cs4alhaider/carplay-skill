---
name: carplay-skill
description: >
  Comprehensive guide for building Apple CarPlay apps, widgets, and Live Activities using the CarPlay framework.
  Use this skill whenever the user mentions CarPlay, CPTemplate, CarPlay entitlements, in-car app development,
  CarPlay Ultra, CarPlay Dashboard, instrument cluster, CarPlay navigation apps, CarPlay audio apps,
  CarPlay communication apps, driving task apps, EV charging apps, fueling apps, parking apps,
  quick food ordering apps, voice-based conversational apps, public safety apps,
  CPListTemplate, CPMapTemplate, CPGridTemplate, CPTabBarTemplate, CPNowPlayingTemplate,
  CPSearchTemplate, CPPointOfInterestTemplate, CPInterfaceController, CPNavigationSession,
  CarPlay Simulator, widgets or Live Activities in CarPlay, or CarPlay app submission.
  Based on Apple's official CarPlay Developer Guide (February 2026).
license: MIT
compatibility: Requires Xcode with iOS SDK. Designed for Claude Code or similar coding-assistant products.
metadata:
  author: Abdullah Alhaider
  version: "1.0.0"
  source: "Apple CarPlay Developer Guide, February 2026"
  repository: "https://github.com/cs4alhaider/carplay-skill"
---

# CarPlay Development Skill

This skill provides authoritative guidance for building CarPlay apps, widgets, and Live Activities based on Apple's official CarPlay Developer Guide (February 2026).

## Quick Reference: What Can You Build?

CarPlay supports three integration points — **widgets**, **Live Activities**, and **CarPlay apps** — each with different requirements.

### Widgets & Live Activities (No CarPlay Entitlement Required)

Your app does **not** need to be a CarPlay app to show widgets or Live Activities in CarPlay.

- **Widgets**: Support the `systemSmall` widget family. Appear in CarPlay Dashboard. Interactive on touchscreens. Supported in CarPlay Ultra and iOS 26+.
- **Live Activities**: Support the `small` activity family (same as Apple Watch Smart Stack). Shown in CarPlay Dashboard or as a notification. Supported in iOS 26+ and CarPlay Ultra.

For details, see [references/widgets-and-live-activities.md](references/widgets-and-live-activities.md).

### CarPlay App Categories

Apps require a category-specific entitlement from Apple. Supported categories:

| Category | Entitlement Key | Min iOS |
|---|---|---|
| Audio | `com.apple.developer.carplay-audio` | 14 |
| Communication | `com.apple.developer.carplay-communication` | 14 |
| Driving Task | `com.apple.developer.carplay-driving-task` | 16 |
| EV Charging | `com.apple.developer.carplay-charging` | 14 |
| Fueling | `com.apple.developer.carplay-fueling` | 16 |
| Navigation | `com.apple.developer.carplay-maps` | 12 |
| Parking | `com.apple.developer.carplay-parking` | 14 |
| Public Safety | `com.apple.developer.carplay-public-safety` | 14 |
| Quick Food Ordering | `com.apple.developer.carplay-quick-ordering` | 14 |
| Voice Conversational | `com.apple.developer.carplay-voice-based-conversation` | 26.4 |

EV charging and fueling entitlements may be combined in a single app.

For the full entitlement and provisioning walkthrough, see [references/entitlements.md](references/entitlements.md).

## Architecture Overview

CarPlay apps use a **Model-View-Controller** pattern where:

- **Your app** selects which template to show (controller) and provides data (model).
- **iOS** renders the template on the CarPlay screen (view).

Your app uses the `CarPlay` framework to present UI via fixed templates. iOS handles layout across different screen resolutions and input hardware (touchscreens, knobs, touch pads).

All CarPlay apps must adopt **UIScene** and declare a CarPlay scene. The scene delegate conforms to `CPTemplateApplicationSceneDelegate`.

### Minimal Startup Example

```swift
import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?

    func templateApplicationScene(
        _ templateApplicationScene: CPTemplateApplicationScene,
        didConnect interfaceController: CPInterfaceController
    ) {
        self.interfaceController = interfaceController
        let listTemplate = CPListTemplate(title: "Home", sections: [])
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }

    func templateApplicationScene(
        _ templateApplicationScene: CPTemplateApplicationScene,
        didDisconnect interfaceController: CPInterfaceController
    ) {
        self.interfaceController = nil
    }
}
```

### Info.plist Scene Manifest

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UISceneConfigurations</key>
    <dict>
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>UIWindowScene</string>
                <key>UISceneConfigurationName</key>
                <string>DeviceScene</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).AppWindowSceneDelegate</string>
            </dict>
        </array>
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlayScene</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).CarPlaySceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

## Templates

Templates are the building blocks of CarPlay UI. Each app category supports specific templates — using an unsupported one triggers a runtime exception.

| Template | Audio | Comms | Driving/Voice | EV/Fuel/Park/QFO | Safety | Nav |
|---|---|---|---|---|---|---|
| Action Sheet | ✓* | ✓ | ✓ | ✓ | ✓ | ✓ |
| Alert | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Grid | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| List | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Tab Bar | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Information | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Point of Interest | — | — | — | ✓ | ✓ | ✓ |
| Now Playing | ✓ | ✓* | — | — | — | ✓ |
| Contact | — | ✓ | — | — | ✓ | — |
| Map | — | — | — | — | — | ✓ |
| Search | — | — | — | — | — | ✓ |
| Voice Control | — | — | ✓** | — | — | ✓ |

*Action Sheet for Audio: iOS 17+. *Now Playing for Comms: iOS 17+. **Voice Control for driving/voice: iOS 26.4+.

**Template depth limits**: Audio, communication, EV charging, parking, public safety, and navigation apps: **5 templates**. Fueling and voice-based conversational: **3 templates**. Driving task and quick food ordering: **2** (iOS ≤ 26.3) or **3** (iOS 26.4+). These include the root template.

For detailed template descriptions, code examples, and usage patterns, see [references/templates.md](references/templates.md).

## Navigation Apps

Navigation apps have unique capabilities: base map view, route guidance panels, search, multitouch, voice prompts, instrument cluster support, and HUD metadata.

For the complete navigation app guide, see [references/navigation.md](references/navigation.md).

## Audio & Communication Apps

For audio session handling, now playing configuration, voice prompts, SiriKit intents, and communication app requirements, see [references/audio-and-communication.md](references/audio-and-communication.md).

## Guidelines

Apple enforces strict guidelines per app category. Violating these will cause App Store rejection. For the complete set of guidelines, see [references/guidelines.md](references/guidelines.md).

**Critical rules that apply to ALL CarPlay apps:**
1. Never instruct users to pick up their iPhone.
2. All flows must work without iPhone interaction.
3. All flows must be meaningful while driving.
4. No gaming or social networking.
5. Never show message/text/email content on screen.
6. Use templates only for their intended purpose.

## Assets

Prepare CarPlay image assets for both 2x and 3x scale factors, and for light and dark styles.

| Element | Max Points | Max 3x | Max 2x |
|---|---|---|---|
| Contact action button | 50×50 | 150×150 | 100×100 |
| Grid icon | 40×40 | 120×120 | 80×80 |
| Now playing action button | 20×20 | 60×60 | 40×40 |
| Tab bar icon | 24×24 | 72×72 | 48×48 |

Use SF Symbols for tab bar icons and similar elements. Use `UIImageAsset` to combine light/dark variants. Use `carTraitCollection` to get the car screen scale at runtime.

## Testing & Simulators

For complete testing guidance including CarPlay Simulator setup, Xcode Simulator configuration, display size recommendations, and instrument cluster testing, see [references/testing.md](references/testing.md).

**Quick start**: CarPlay Simulator is in the "Additional Tools for Xcode" package → Hardware folder. Connect iPhone via USB.

## Notifications

Notifications are supported in communication, EV charging, parking, public safety, and (iOS 18.4+) driving task apps. For details see [references/assets-and-notifications.md](references/assets-and-notifications.md).

## Locked iPhone Considerations

CarPlay is frequently used while iPhone is locked. You cannot access:
- Files saved with `NSFileProtectionComplete` or `NSFileProtectionCompleteUnlessOpen`.
- Keychain items with `kSecAttrAccessibleWhenUnlocked` or similar restricted attributes.

Test thoroughly with a locked iPhone via CarPlay Simulator.

## Launching Other Apps

Use `CPTemplateApplicationScene.open(_:options:completionHandler:)` with a URL to launch other apps on the CarPlay screen (not `UIApplication.shared.open`).

## Publishing

Submit via App Store Connect like any iOS app. Ensure compliance with the [CarPlay Guidelines](references/guidelines.md) and the CarPlay Entitlement Addendum.

## Sample Code from Apple

- **CarPlay Music** — Audio app with `CPNowPlayingTemplate` and `CPListTemplate`.
- **CarPlay Quick-Ordering** — QSR app with `CPPointOfInterestTemplate` and `CPListTemplate`.
- **Coastal Roads** — Navigation app with map rendering and `CPGridTemplate`/`CPListTemplate`.

Download from Apple's developer documentation site.

## Common Pitfalls

1. **Activating audio session too early** — stops the car's FM radio. Wait until you actually play audio.
2. **Recording while in CarPlay** — not supported (except voice input in navigation/voice-based conversational apps, and only with voice control template).
3. **Not handling locked iPhone** — data-protection-restricted files/keychain items will be inaccessible.
4. **Exceeding template depth** — causes runtime exceptions. Respect your category's limit.
5. **Hardcoding tab counts** — query the maximum from iOS; it may change in future versions.
6. **Missing panning button** — navigation apps that support panning must provide a map button for panning mode (some cars have no touchscreen).
7. **Not terminating guidance when asked** — if the car's native nav starts, your delegate receives a cancellation and you must stop immediately.
8. **Drawing UI in the base view** — the base view is exclusively for the map. Use templates for all UI elements.
9. **Refreshing data too frequently** — driving task apps: max once per 10 seconds for data items, once per 60 seconds for POI templates.
10. **Missing light/dark variants** — always provide both for all image assets and maneuver symbols.

## External References

- [Apple CarPlay Developer Portal](http://developer.apple.com/carplay)
- [Human Interface Guidelines: CarPlay](https://developer.apple.com/design/human-interface-guidelines/carplay)
- [Human Interface Guidelines: Widgets](https://developer.apple.com/design/human-interface-guidelines/widgets)
- [Human Interface Guidelines: Live Activities](https://developer.apple.com/design/human-interface-guidelines/live-activities)
- [CarPlay Framework API Reference](https://developer.apple.com/documentation/carplay)
- [SiriKit Documentation](https://developer.apple.com/documentation/sirikit)
- [CallKit Documentation](https://developer.apple.com/documentation/callkit)
- [Implementing Communication Notifications](https://developer.apple.com/documentation/usernotifications/implementing-communication-notifications)
