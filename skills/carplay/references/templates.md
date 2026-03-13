# CarPlay Templates

Comprehensive reference for all CarPlay UI templates, their usage, and code examples.

## Table of Contents

1. [Action Sheet Template](#action-sheet-template)
2. [Alert Template](#alert-template)
3. [Contact Template](#contact-template)
4. [Grid Template](#grid-template)
5. [Information Template](#information-template)
6. [List Template](#list-template)
7. [Now Playing Template](#now-playing-template)
8. [Point of Interest Template](#point-of-interest-template)
9. [Tab Bar Template](#tab-bar-template)
10. [Voice Control Template](#voice-control-template)
11. [Template Depth Limits](#template-depth-limits)

---

## Action Sheet Template

A modal alert that appears in response to a control or action, presenting two or more contextual choices. Consists of a title, message, and buttons.

**Use for**: initiating tasks, confirming destructive operations.

Available in all app categories. Audio apps require iOS 17+.

```swift
import CarPlay

let action1 = CPAlertAction(title: "Delete", style: .destructive) { _ in
    // Handle deletion
}
let action2 = CPAlertAction(title: "Cancel", style: .cancel) { _ in
    // Dismiss
}
let actionSheet = CPActionSheetTemplate(
    title: "Remove Download?",
    message: "This will free up space on your device.",
    actions: [action1, action2]
)
interfaceController.presentTemplate(actionSheet, animated: true)
```

## Alert Template

Modal alerts convey important information about your app's state. Consists of a title and one or more buttons. You can provide multiple title variants of varying length — CarPlay picks the one that best fits.

Available in all app categories.

```swift
import CarPlay

let okAction = CPAlertAction(title: "OK", style: .default) { _ in
    // Dismiss alert
}
let alert = CPAlertTemplate(
    titleVariants: [
        "Unable to connect to your account. Please try again later.",
        "Connection failed. Try again later.",
        "Connection failed"
    ],
    actions: [okAction]
)
interfaceController.presentTemplate(alert, animated: true)
```

## Contact Template

Presents information about a person or business: image, title, subtitle, and action buttons. Action buttons let users perform tasks like calling or messaging.

You can optionally provide a bar button to activate Siri for composing a message.

Available in communication, public safety, and navigation apps.

```swift
import CarPlay

let callAction = CPContactCallButton { button in
    // Initiate VoIP call
}
let messageAction = CPContactMessageButton { button in
    // Compose message via Siri
}
let contact = CPContactTemplate(contact: CPContact(
    name: "Alice Smith",
    image: UIImage(named: "alice")!,
    subtitle: "Mobile",
    informativeText: nil,
    actions: [callAction, messageAction]
))
interfaceController.pushTemplate(contact, animated: true)
```

## Grid Template

A menu presenting up to 8 choices, each with an icon and title. Includes a navigation bar with title, leading buttons, and trailing buttons.

**Use for**: letting users select from a fixed list of items.

Available in all app categories.

```swift
import CarPlay

let gridButton1 = CPGridButton(
    titleVariants: ["Favorites"],
    image: UIImage(systemName: "star.fill")!
) { _ in
    // Navigate to favorites
}
let gridButton2 = CPGridButton(
    titleVariants: ["Recent"],
    image: UIImage(systemName: "clock.fill")!
) { _ in
    // Navigate to recents
}
let grid = CPGridTemplate(
    title: "Browse",
    gridButtons: [gridButton1, gridButton2]
)
interfaceController.pushTemplate(grid, animated: true)
```

## Information Template

A specific list style presenting a limited number of static labels with optional footer buttons. Labels appear in one or two columns. Supports leading and trailing navigation bar buttons (iOS 16+).

**Use for**: showing key summary information (e.g., charging station availability, order summary with pick-up location and time).

Available in audio, communication, driving task/voice, EV/fuel/park/QFO, and public safety apps. **Not** navigation.

```swift
import CarPlay

let item1 = CPInformationItem(title: "Station", detail: "Elm Street Charger")
let item2 = CPInformationItem(title: "Availability", detail: "2 of 4 available")
let item3 = CPInformationItem(title: "Price", detail: "$0.35/kWh")

let startAction = CPTextButton(title: "Start Charging", textStyle: .confirm) { _ in
    // Begin charging session
}
let infoTemplate = CPInformationTemplate(
    title: "Charger Details",
    layout: .leading,
    items: [item1, item2, item3],
    actions: [startAction]
)
interfaceController.pushTemplate(infoTemplate, animated: true)
```

## List Template

Presents data as a scrolling, single-column table divided into sections. The most versatile template.

### Item Types

- **Item** (`CPListItem`): standard icon + text.
- **Image row item** (`CPListImageRowItem`): horizontal series of images. With iOS 26+, supports 5 element styles: row, card, condensed, grid, and image grid.
- **Message item**: contact/conversation display for communication apps (iOS 26+).
- **Assistant cell**: Siri prompt for media playback or calls. Can appear at top or bottom.

### Pinned Elements (iOS 26+)

Grid elements with image and title that always appear at the top. Communication apps can add unread indicators.

### Dynamic List Limits

Some cars limit lists to 12 items. Check `CPListTemplate.maximumSectionCount` and always handle truncation gracefully.

```swift
import CarPlay

// Basic list with a section
let item = CPListItem(text: "Highway to Heaven", detailText: "Podcast • 45 min")
item.handler = { [weak self] item, completion in
    self?.startPlayback(for: item)
    self?.interfaceController?.pushTemplate(
        CPNowPlayingTemplate.shared, animated: true
    )
    completion()
}

let assistantCell = CPAssistantCellConfiguration(
    position: .top,
    visibility: .always,
    assistantAction: .playMedia
)

let section = CPListSection(items: [item])
let listTemplate = CPListTemplate(title: "Episodes", sections: [section])
listTemplate.assistantCellConfiguration = assistantCell

interfaceController.pushTemplate(listTemplate, animated: true)
```

### Image Row Item Example

```swift
import CarPlay

let images = (1...5).compactMap { UIImage(named: "album_\($0)") }
let imageRow = CPListImageRowItem(text: "Recently Played", images: images)
imageRow.listImageRowHandler = { item, index, completion in
    // User tapped image at index
    completion()
}

let section = CPListSection(items: [imageRow])
let listTemplate = CPListTemplate(title: "Library", sections: [section])
```

## Now Playing Template

Displays currently playing audio: title, artist, elapsed time, album artwork, and playback control buttons.

### Key Behaviors

- **Shared instance**: access via `CPNowPlayingTemplate.shared`.
- Accessible directly from the CarPlay home screen or the now playing button in your app's navigation bar.
- Must be populated at all times — iOS can present it at any time.
- Only a `CPListTemplate` may be pushed on top (e.g., for "Playing Next" queue).
- Customizable playback controls, elapsed time indicator (fixed-length or open-ended/live stream).

Available in audio, communication (iOS 17+), and navigation apps.

```swift
import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    func templateApplicationScene(
        _ templateApplicationScene: CPTemplateApplicationScene,
        didConnect interfaceController: CPInterfaceController
    ) {
        let nowPlaying = CPNowPlayingTemplate.shared
        let rateButton = CPNowPlayingPlaybackRateButton { button in
            // Cycle playback rate
        }
        let shuffleButton = CPNowPlayingShuffleButton { button in
            // Toggle shuffle
        }
        nowPlaying.updateNowPlayingButtons([rateButton, shuffleButton])
    }
}
```

### Sports Mode (iOS 18.4+)

The now playing template supports a sports mode for live or recorded sporting events involving 2 teams. The sports presentation includes playback controls plus team images, scores, a countdown/count-up clock, possession indicators, standings, and more.

CarPlay automatically counts the event clock up or down from the point you provide. Update sports mode metadata at any point to adjust scores or other fields.

**Sports mode metadata fields**:
- Background artwork (image)
- Event status text (array of text) and status image
- Event clock (count up, count down, or paused)
- Per-team: name, logo (image or text), standings, score, possession indicator, favorite indicator

## Point of Interest Template

Lets users browse nearby locations on a MapKit-powered map and choose one. Includes an overlay with up to 12 locations with customizable pin images. iOS 16+ supports a larger pin for the selected location.

Available in EV charging, fueling, parking, quick food ordering, public safety, and navigation apps.

```swift
import CarPlay
import MapKit

let poi1 = CPPointOfInterest(
    location: MKMapItem(placemark: MKPlacemark(
        coordinate: CLLocationCoordinate2D(latitude: 37.334, longitude: -122.009)
    )),
    title: "Apple Park Charger",
    subtitle: "4 available",
    summary: "Level 2 • $0.35/kWh",
    detailTitle: "Apple Park Charger",
    detailSubtitle: "4 of 8 available",
    detailSummary: "Open 24 hours",
    pinImage: UIImage(systemName: "bolt.fill")
)
poi1.primaryButton = CPTextButton(title: "Navigate", textStyle: .confirm) { _ in
    // Start navigation
}

let poiTemplate = CPPointOfInterestTemplate(
    title: "Nearby Chargers",
    pointsOfInterest: [poi1],
    selectedIndex: 0
)
interfaceController.pushTemplate(poiTemplate, animated: true)
```

## Tab Bar Template

A container for other templates, each occupying one tab. Supports text or image titles (SF Symbols recommended). Tabs can show a small red indicator for actionable/ephemeral state.

### Tab Limits

- Audio apps: up to **4** tabs.
- All other categories: up to **5** tabs.

Query the maximum from iOS at runtime — avoid hardcoding.

When playing audio, CarPlay shows a "now playing" button in the top right. This button may not appear if you have more than 4 tabs.

```swift
import CarPlay

let favoritesTab = CPListTemplate(title: "Favorites", sections: [])
favoritesTab.tabSystemItem = .favorites
favoritesTab.tabImage = UIImage(systemName: "star.fill")

let recentsTab = CPListTemplate(title: "Recents", sections: [])
recentsTab.tabImage = UIImage(systemName: "clock.fill")

let searchTab = CPListTemplate(title: "Search", sections: [])
searchTab.tabImage = UIImage(systemName: "magnifyingglass")

let tabBar = CPTabBarTemplate(templates: [favoritesTab, recentsTab, searchTab])
interfaceController.setRootTemplate(tabBar, animated: true)
```

## Voice Control Template

Provides visual feedback for voice-based services. Available in navigation and voice-based conversational apps (iOS 26.4+).

Navigation apps must limit voice control to navigation functions. The voice control screen must be displayed while voice-based services are active.

iOS 26.4+ supports up to 4 action buttons, plus leading/trailing navigation bar buttons.

All other app categories must use SiriKit or Siri Shortcuts for voice features.

```swift
import CarPlay

let voiceTemplate = CPVoiceControlTemplate(
    voiceControlStates: [
        CPVoiceControlState(
            titleVariants: ["Listening..."],
            image: UIImage(systemName: "mic.fill"),
            identifier: "listening"
        ),
        CPVoiceControlState(
            titleVariants: ["Processing..."],
            image: UIImage(systemName: "waveform"),
            identifier: "processing"
        )
    ]
)
interfaceController.presentTemplate(voiceTemplate, animated: true)
```

## Template Depth Limits

The number of templates you can push onto the screen is limited by app category:

| Category | Max Depth |
|---|---|
| Audio, Communication, EV Charging, Parking, Public Safety, Navigation | 5 |
| Fueling, Voice-Based Conversational | 3 |
| Driving Task, Quick Food Ordering (iOS ≤ 26.3) | 2 |
| Driving Task, Quick Food Ordering (iOS 26.4+) | 3 |

These limits include the root template. Exceeding them causes a runtime exception.
