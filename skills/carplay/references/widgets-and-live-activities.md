# Widgets & Live Activities in CarPlay

Guide for enabling widgets and Live Activities in CarPlay. No CarPlay app entitlement is required for either.

## Table of Contents

1. [Widgets in CarPlay](#widgets-in-carplay)
2. [Disfavored Locations](#disfavored-locations)
3. [Widget Interaction & App Launching](#widget-interaction--app-launching)
4. [Live Activities in CarPlay](#live-activities-in-carplay)

---

## Widgets in CarPlay

Widgets appear to the left of CarPlay Dashboard (right in right-hand drive vehicles) and support interaction on touchscreen vehicles.

**Availability**: CarPlay Ultra, and iOS 26+ in standard CarPlay.

**Configuration**: Users customize widgets in Settings → General → CarPlay → [Vehicle].

### Enabling Your Widget

Support the `systemSmall` widget family:

```swift
struct MyWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: "com.example.mywidget", provider: MyProvider()) { entry in
            MyWidgetView(entry: entry)
        }
        .supportedFamilies([.systemSmall])
    }
}
```

### Design Considerations

- Widgets are optimized for each vehicle, including content margins.
- If you mark your background as removable, the widget background will not be shown in CarPlay.
- Follow [Human Interface Guidelines: Widgets](https://developer.apple.com/design/human-interface-guidelines/widgets).

## Disfavored Locations

If your widget is not functional or suitable for driving, set CarPlay as a disfavored location:

```swift
.disfavoredLocations([.carPlay], for: [.systemSmall])
```

**When to set CarPlay as disfavored:**

- Your widget is a game or requires extensive interaction (more than ~6 taps/refreshes).
- Your widget is non-functional in the car (e.g., relies on data protection classes A or B, which are inaccessible when iPhone is locked — the typical CarPlay scenario).
- Your widget's primary purpose is launching an iPhone app that is not a CarPlay app.

When disfavored, the widget appears grouped in Settings with an "not optimized for CarPlay" indicator. Users can still choose to show it, but interaction is disabled.

## Widget Interaction & App Launching

Your widget can only launch your app in CarPlay if your app is also a CarPlay app. Use standard mechanisms:

```swift
// In your widget view
struct MyWidgetView: View {
    var body: some View {
        Link(destination: URL(string: "myapp://open-feature")!) {
            Text("Open Feature")
        }
    }
}

// Or use widgetURL on the entire widget
struct MyWidgetView: View {
    var body: some View {
        VStack {
            Text("My Widget")
        }
        .widgetURL(URL(string: "myapp://dashboard")!)
    }
}
```

## Live Activities in CarPlay

Live Activities provide timely, real-time updates in CarPlay Dashboard or as notifications.

**Availability**: iOS 26+ in CarPlay and CarPlay Ultra.

### Enabling Your Live Activity

Support the `small` supplemental activity family. This is the same size as Live Activities in the Apple Watch Smart Stack:

```swift
struct MyLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: MyActivityAttributes.self) { context in
            // Lock screen / banner presentation
            MyLiveActivityView(context: context)
        } dynamicIsland: { context in
            DynamicIsland {
                // Expanded regions
            } compactLeading: {
                Image(systemName: "car.fill")
            } compactTrailing: {
                Text(context.state.eta)
            } minimal: {
                Image(systemName: "car.fill")
            }
        }
        .supplementalActivityFamilies([.small])
    }
}
```

### Fallback Behavior

If you don't support the `small` activity family, CarPlay shows the compact leading and compact trailing views from your Dynamic Island configuration instead.

### Apple Watch Reuse

If you already support Apple Watch Smart Stack with the `small` family, the same Live Activity works in CarPlay automatically.

For design guidance, see [Human Interface Guidelines: Live Activities](https://developer.apple.com/design/human-interface-guidelines/live-activities).
