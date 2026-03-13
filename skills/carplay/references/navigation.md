# CarPlay Navigation Apps

Complete guide for building CarPlay navigation apps with maps, route guidance, instrument cluster support, and HUD metadata.

## Table of Contents

1. [Supported Displays](#supported-displays)
2. [Base View](#base-view)
3. [Map Template](#map-template)
4. [Search Template](#search-template)
5. [Panels](#panels)
6. [Route Guidance Flow](#route-guidance-flow)
7. [Maneuvers & Trip Estimates](#maneuvers--trip-estimates)
8. [Lane Guidance](#lane-guidance)
9. [Multitouch](#multitouch)
10. [Keyboard & List Restrictions](#keyboard--list-restrictions)
11. [Voice Prompts](#voice-prompts)
12. [CarPlay Dashboard & Instrument Cluster](#carplay-dashboard--instrument-cluster)
13. [Metadata in Instrument Cluster or HUD](#metadata-in-instrument-cluster-or-hud)
14. [Maneuver Types Reference](#maneuver-types-reference)
15. [Scene Manifest for Navigation Apps](#scene-manifest-for-navigation-apps)

---

## Supported Displays

Navigation apps can appear in multiple locations:

| iOS Version | Center Display | Dashboard | Instrument Cluster | Metadata in Cluster/HUD |
|---|---|---|---|---|
| iOS 12 | ✓ | — | — | — |
| iOS 13.4 | ✓ | ✓ | — | — |
| iOS 16.4 | ✓ | ✓ | ✓ | — |
| iOS 17.4 | ✓ | ✓ | ✓ | ✓ |

Support all capabilities for a seamless experience across all vehicle configurations.

## Base View

All navigation apps start with a base view — this is where you draw your map. Create and attach it to the provided `CPWindow` when CarPlay starts.

**Rules:**
- The base view must be used **exclusively** to draw a map.
- Do not draw alerts, overlays, panels, or any other UI elements in the base view (e.g., no lane guidance in the base view).
- Your app will not receive direct tap or drag events in the base view.
- Must support varying aspect ratios, resolutions, and light/dark mode.

Use `contentStyle` on the CarPlay template application scene for current mode, and observe `contentStyleDidChange` in your scene delegate. Respect safe areas (portions not obscured by buttons).

## Map Template

The map template is a control overlay on the base view for manipulating the map. It includes:

- **Navigation bar**: up to 2 leading + 2 trailing buttons (icons or text). Auto-hides after inactivity by default.
- **Map buttons**: up to 4, shown as icons. Use for zoom, pan, etc.

### Panning Mode

Not all cars have touchscreens — some use knobs or touch pads. CarPlay provides a "panning mode" for these. If your app supports any panning, you **must** provide a pan button among the map buttons and respond to panning functions in `CPMapTemplate`.

```swift
import CarPlay

func templateApplicationScene(
    _ scene: CPTemplateApplicationScene,
    didConnect interfaceController: CPInterfaceController,
    to window: CPWindow
) {
    self.interfaceController = interfaceController
    self.carWindow = window

    // Set up the base view
    let mapViewController = MapViewController()
    window.rootViewController = mapViewController

    // Create map template with buttons
    let zoomIn = CPMapButton { _ in mapViewController.zoomIn() }
    zoomIn.image = UIImage(systemName: "plus.magnifyingglass")

    let zoomOut = CPMapButton { _ in mapViewController.zoomOut() }
    zoomOut.image = UIImage(systemName: "minus.magnifyingglass")

    let panButton = CPMapButton { [weak self] _ in
        self?.mapTemplate?.showPanningInterface(animated: true)
    }
    panButton.image = UIImage(systemName: "hand.draw")

    let mapTemplate = CPMapTemplate()
    mapTemplate.mapButtons = [zoomIn, zoomOut, panButton]
    mapTemplate.mapDelegate = self

    self.mapTemplate = mapTemplate
    interfaceController.setRootTemplate(mapTemplate, animated: false)
}
```

## Search Template

Displays a text field, search results, and keyboard. Respond to `updatedSearchText` to update results with `CPListItem` elements. Respond to `selectedResult` when an item is picked.

Many cars limit when the keyboard is shown — see [Keyboard & List Restrictions](#keyboard--list-restrictions).

## Panels

Navigation apps use panels overlaid on the map. You don't create them directly — use the provided APIs.

### Trip Preview Panel

Shows up to 12 potential destinations. Typically appears after a search. When previewed, show the trip visually on your map.

### Route Choice Panel

Shows potential routes for a trip. Each route needs a clear description. When a route is previewed, show it on the map.

### Guidance & Trip Estimate Panels

Show upcoming maneuvers and trip estimates during guidance. Normally one maneuver at a time; rapid successions may show two. The second maneuver slot can display lane guidance or junction images.

### Navigation Alert Panel

Shows important real-time feedback with optional decision buttons (up to 2). Includes image, title, subtitle, auto-dismiss duration. Shown as a notification when your app is in the background.

iOS 16+: longer subtitle text, no action buttons (simple close), custom button colors.

## Route Guidance Flow

All navigation apps follow this standard flow:

1. **Select destination** — via search, voice, list, or grid.
2. **Preview** — show trip preview with up to 12 trips. Supports disambiguation.
3. **Choose route & start** — driver confirms, optionally picks from multiple routes.
4. **View trip info** — real-time maneuvers, distance, and time estimates.
5. **End guidance** — driver arrives or cancels.
6. **Re-route** (optional, iOS 17.4+) — return to active guidance with a new route via `CPNavigationSession.resumeTrip`.

### Select Destination

```swift
// Push templates for destination selection
let searchTemplate = CPSearchTemplate()
searchTemplate.delegate = self
interfaceController.pushTemplate(searchTemplate, animated: true)
```

### Preview

```swift
let routeChoice = CPRouteChoice(
    summaryVariants: ["Via I-280 South", "Via I-280 S.", "I-280"],
    additionalInformationVariants: ["Traffic is light", "Light traffic"],
    selectionSummaryVariants: ["I-280 South — 25 min"]
)
let trip = CPTrip(
    origin: MKMapItem.forCurrentLocation(),
    destination: destinationMapItem,
    routeChoices: [routeChoice]
)
mapTemplate.showTripPreviews([trip])
mapTemplate.updateEstimates(
    CPTripEstimateStyle(
        distanceRemaining: Measurement(value: 12.5, unit: .miles),
        timeRemaining: 1500
    ),
    for: trip
)
```

### Start Guidance

```swift
// In your CPMapTemplateDelegate:
func mapTemplate(_ mapTemplate: CPMapTemplate, startedTrip trip: CPTrip,
                 using routeChoice: CPRouteChoice) {
    mapTemplate.hideTripPreviews()
    let session = mapTemplate.startNavigationSession(for: trip)
    self.navigationSession = session
    session.pauseTrip(for: .loading)

    // Calculate initial maneuvers, then call:
    // session.upcomingManeuvers = [maneuver1, maneuver2, ...]
}
```

### End Guidance

```swift
// When guidance ends:
navigationSession?.finishTrip()
// or
navigationSession?.cancelTrip()

// If system cancels (e.g., car's native nav starts):
func mapTemplateDidCancelNavigation(_ mapTemplate: CPMapTemplate) {
    navigationSession?.cancelTrip()
    // Stop guidance immediately
}
```

## Maneuvers & Trip Estimates

Each `CPManeuver` includes:

- **Symbol** (`symbolSet`): `CPImageSet` with light and dark variants.
- **Instruction** (`instructionVariants`): array of strings in descending length order. CarPlay picks the longest that fits.
- **Attributed instructions** (`attributedInstructionVariants`): optional, for embedded images (e.g., highway symbols). Always also provide plain `instructionVariants`.
- **Metadata**: maneuver type, state, junction type, traffic side, lane guidance — for instrument cluster/HUD.

Maintain at least one upcoming maneuver at all times. Provide a second for rapid succession.

### Maneuver Symbol Asset Sizes

| Element | Max Points | Max 3x | Max 2x |
|---|---|---|---|
| 1st maneuver (1 line) | 50×50 | 150×150 | 100×100 |
| 1st maneuver (2 lines) | 120×50 | 360×150 | 240×100 |
| 2nd maneuver (with instructions) | 18×18 | 54×54 | 36×36 |
| 2nd maneuver (symbol only) | 120×18 | 360×54 | 240×36 |
| Dashboard junction image | 140×100 | 420×300 | 280×200 |

### Updating Estimates

```swift
// Per-maneuver estimates
let estimates = CPTravelEstimates(
    distanceRemaining: Measurement(value: 0.7, unit: .miles),
    timeRemaining: 120
)
navigationSession?.updateEstimates(estimates, for: maneuver)

// Overall trip estimates
mapTemplate.updateEstimates(tripEstimates, for: trip)
```

Only update when significant changes occur (e.g., remaining minutes change).

### Notifications from Maneuvers

Respond to `shouldShowNotificationFor` delegate call to decide if a maneuver or navigation alert should appear as a notification when your app is in the background.

## Lane Guidance

Use the second maneuver to show lane guidance. Create a second `CPManeuver` with:
- `symbolSet` containing dark and light images at full width (max 120pt × 18pt).
- Empty array for `instructionVariants`.
- Return `CPManeuverDisplayStyle.symbolOnly` from `CPMapTemplateDelegate`.

## Multitouch

Vehicles supporting multitouch (including all CarPlay Ultra vehicles) send gesture callbacks to your `CPMapTemplate` (iOS 26+):

- **Zoom**: pinch, double tap (in), two-finger double tap (out).
- **Pitch**: two-finger slide up/down.
- **Rotate**: two-finger clockwise/counterclockwise.

## Keyboard & List Restrictions

Some cars disable the keyboard and reduce list lengths while driving. iOS handles this automatically, but you can observe changes via `CPSessionConfiguration.limitedUserInterfaces` to adjust your UI (e.g., disable a search icon).

## Voice Prompts

### Audio Session Configuration

```swift
let session = AVAudioSession.sharedInstance()
try session.setCategory(
    .playback,
    mode: .voicePrompt,
    options: [.interruptSpokenAudioAndMixWithOthers, .duckOthers]
)
```

- `interruptSpokenAudioAndMixWithOthers`: pauses podcasts/audiobooks, mixes with music.
- `duckOthers`: lowers other audio volume during prompts.

### Activation Rules

- Keep the session **deactivated** until a prompt is ready.
- Call `setActive(true)` only when playing. While active, music is ducked and spoken audio is paused.
- Call `setActive(false)` when done — don't hold for more than a few seconds between prompts.

### Prompt Style

Check `AVAudioSession.sharedInstance().promptStyle` before each prompt:

| Style | Action |
|---|---|
| `.none` | Don't play any sound |
| `.short` | Play a tone only |
| `.normal` | Play the full spoken prompt |

## CarPlay Dashboard & Instrument Cluster

### Declaring Support

In your scene manifest, set these keys to `true`:

- `CPSupportsDashboardNavigationScene`
- `CPSupportsInstrumentClusterNavigationScene`

### Scene Delegates

- **Dashboard**: conforms to `CPTemplateApplicationDashboardSceneDelegate`. Receives `CPDashboardController`.
- **Instrument Cluster**: conforms to `CPTemplateApplicationInstrumentClusterSceneDelegate`. Receives `CPInstrumentClusterController`.

### Drawing Maps

Use the provided windows to draw map content for each scene.

**Instrument cluster rules:**
- Draw a minimal map with minimal clutter.
- Show a detailed view of the upcoming route, not an overview.
- Current heading must face up (top of screen).
- Observe safe areas and light/dark mode.
- Respond to zoom events from the car.
- Check delegates for whether to draw compass or speed limit.
- Override `viewSafeAreaInsetsDidChange` and use `safeAreaLayoutGuide`.

**Dashboard buttons**: provide up to 2 `CPDashboardButton` instances to `CPDashboardController`. Shown when not actively navigating.

## Metadata in Instrument Cluster or HUD

Starting with iOS 17.4, supply metadata for maneuvers displayed in instrument clusters and HUDs.

### Enabling

Return `true` from `mapTemplateShouldProvideNavigationMetadata` in `CPMapTemplateDelegate`.

### Providing Maneuvers

Supply maneuvers with type and lane guidance when guidance starts. Use `add CPManeuver` and `add CPLane Guidance`. Provide as many as possible for better performance.

Set the **current road name** and update **maneuver state** as the vehicle progresses:

| State | Description |
|---|---|
| `continue` | Continue along route until next maneuver |
| `initial` | Maneuver is in the near future |
| `prepare` | Maneuver is in the immediate future |
| `execute` | Currently in the maneuver |

Required maneuver properties: `maneuverType`, `junctionType`, `trafficSide`.

### Junction Types

| Type | Description |
|---|---|
| `intersection` | Roads coming to a common point |
| `roundabout` | Roads exiting a roundabout |

### Traffic Side

| Side | Description |
|---|---|
| `right` | Right (anti-clockwise for roundabouts) |
| `left` | Left (clockwise for roundabouts) |

### Lane Status

| Status | Description |
|---|---|
| `notGood` | Vehicle should not take this lane |
| `good` | Vehicle can take this lane but may need to move |
| `preferred` | Best lane for upcoming maneuvers |

Lane angles: specify angles between -180° and +180°.

## Maneuver Types Reference

Full list of predefined `maneuverType` values:

- `noTurn` (default), `startRoute`, `startRouteWithUTurn`
- `straightAhead`, `followRoad`
- `leftTurn`, `rightTurn`, `slightLeftTurn`, `slightRightTurn`, `sharpLeftTurn`, `sharpRightTurn`
- `leftTurnAtEnd`, `rightTurnAtEnd`
- `keepLeft`, `keepRight`
- `uTurn`, `uTurnWhenPossible`, `uTurnAtRoundabout`
- `enterRoundabout`, `exitRoundabout`, `roundaboutExit1` through `roundaboutExit19`
- `onRamp`, `offRamp`, `highwayOffRampLeft`, `highwayOffRampRight`
- `changeHighway`, `changeHighwayLeft`, `changeHighwayRight`
- `enterFerry`, `exitFerry`, `changeFerry`
- `arriveAtDestination`, `arriveAtDestinationLeft`, `arriveAtDestinationRight`
- `arriveEndOfDirections`, `arriveEndOfNavigation`

## Scene Manifest for Navigation Apps

Full manifest supporting center display, Dashboard, and instrument cluster:

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>CPSupportsDashboardNavigationScene</key>
    <true/>
    <key>CPSupportsInstrumentClusterNavigationScene</key>
    <true/>
    <key>UIApplicationSupportsMultipleScenes</key>
    <true/>
    <key>UISceneConfigurations</key>
    <dict>
        <!-- Device scene -->
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>UIWindowScene</string>
                <key>UISceneConfigurationName</key>
                <string>Phone</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).AppWindowSceneDelegate</string>
            </dict>
        </array>
        <!-- Main CarPlay scene -->
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).CarPlaySceneDelegate</string>
            </dict>
        </array>
        <!-- Dashboard scene -->
        <key>CPTemplateApplicationDashboardSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationDashboardScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Dashboard</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).CarPlayDashboardSceneDelegate</string>
            </dict>
        </array>
        <!-- Instrument cluster scene -->
        <key>CPTemplateApplicationInstrumentClusterSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationInstrumentClusterScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Instrument-Cluster</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).CarPlayInstrumentClusterSceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```
