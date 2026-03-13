# Assets & Notifications

Guide for preparing CarPlay image assets and implementing notifications.

## Table of Contents

1. [Image Assets](#image-assets)
2. [Asset Size Reference](#asset-size-reference)
3. [SF Symbols](#sf-symbols)
4. [Programmatic Image Assets](#programmatic-image-assets)
5. [Runtime Scale Detection](#runtime-scale-detection)
6. [List Item Image Sizes](#list-item-image-sizes)
7. [Notifications](#notifications)
8. [Communication Notifications](#communication-notifications)

---

## Image Assets

CarPlay supports multiple scales (2x and 3x) and both light and dark interfaces. Create all variants for each asset.

### Xcode Asset Catalog

Turn on **CarPlay assets** in Xcode to populate CarPlay 2x and 3x image wells in your asset catalog.

### Tips

- Test with CarPlay Simulator to verify appearance under different screen resolutions, scale factors, and light/dark styles.
- Always provide both light and dark variants for all images.

## Asset Size Reference

| Element | Max Points | Max 3x Pixels | Max 2x Pixels |
|---|---|---|---|
| Contact action button | 50 × 50 | 150 × 150 | 100 × 100 |
| Grid icon | 40 × 40 | 120 × 120 | 80 × 80 |
| Now playing action button | 20 × 20 | 60 × 60 | 40 × 40 |
| Tab bar icon | 24 × 24 | 72 × 72 | 48 × 48 |

### Navigation Maneuver Symbols

| Element | Max Points | Max 3x | Max 2x |
|---|---|---|---|
| 1st maneuver (1-line layout) | 50 × 50 | 150 × 150 | 100 × 100 |
| 1st maneuver (2-line layout) | 120 × 50 | 360 × 150 | 240 × 100 |
| 2nd maneuver (with instructions) | 18 × 18 | 54 × 54 | 36 × 36 |
| 2nd maneuver (symbol only) | 120 × 18 | 360 × 54 | 240 × 36 |
| Dashboard junction image | 140 × 100 | 420 × 300 | 280 × 200 |

## SF Symbols

Use SF Symbols for tab bar icons and similar UI elements for seamless integration with the system font.

```swift
let tabImage = UIImage(systemName: "music.note.list")
let mapButton = UIImage(systemName: "plus.magnifyingglass")
```

## Programmatic Image Assets

If you create assets programmatically, use `UIImageAsset` to combine light and dark variants:

```swift
let imageAsset = UIImageAsset()

let lightImage = UIImage(named: "icon-light")!
let darkImage = UIImage(named: "icon-dark")!

let lightTraits = UITraitCollection(userInterfaceStyle: .light)
let darkTraits = UITraitCollection(userInterfaceStyle: .dark)

imageAsset.register(lightImage, with: lightTraits)
imageAsset.register(darkImage, with: darkTraits)

// Use the asset — it automatically resolves the correct variant
let resolvedImage = imageAsset.image(with: traitCollection)
```

## Runtime Scale Detection

Get the CarPlay screen scale at runtime using the car trait collection:

```swift
// Get scale for the car's screen (not the iPhone screen)
let carScale = templateApplicationScene.carTraitCollection.displayScale
```

> **Important**: Do not use other parameters from `carTraitCollection`. Only use it for the display scale.

## List Item Image Sizes

Use the maximum image size properties to provide correctly sized images:

```swift
// For CPListItem
let maxSize = CPListItem.maximumImageSize
// Provide images matching this resolution

// For CPListImageRowItem
let maxRowSize = CPListImageRowItem.maximumImageSize
```

## Notifications

Notifications are supported in:
- Communication apps
- EV Charging apps
- Parking apps
- Public Safety apps
- Driving Task apps (iOS 18.4+)

Notifications should be used **sparingly** and reserved for important tasks required while driving. Do not use notifications for iPhone-only features. In general, notifications are **not read aloud** in CarPlay.

> **Note**: Route guidance notifications in navigation apps are handled by the CarPlay framework and are separate from standard notifications.

### Request Authorization

Include the `carPlay` option when requesting notification authorization:

```swift
let options: UNAuthorizationOptions = [.badge, .sound, .alert, .carPlay]
let center = UNUserNotificationCenter.current()
center.requestAuthorization(options: options) { granted, error in
    // Enable or disable features based on authorization
}
```

Users can show/hide your notifications in CarPlay via Settings. Gracefully disable notification features if declined.

### Create a CarPlay Notification Category

Enable CarPlay for specific notification categories using `allowInCarPlay`:

```swift
let category = UNNotificationCategory(
    identifier: "MESSAGE_RECEIVED",
    actions: [/* your actions */],
    intentIdentifiers: [],
    options: [.allowInCarPlay]
)
UNUserNotificationCenter.current().setNotificationCategories([category])
```

Ensure local or remote notifications use the same category identifier.

## Communication Notifications

For CarPlay communication apps, also refer to Apple's documentation on [Implementing Communication Notifications](https://developer.apple.com/documentation/usernotifications/implementing-communication-notifications).

**Key rule**: In CarPlay, notifications must only include sender and group name in the title and subtitle. The message content must **never** be shown.
