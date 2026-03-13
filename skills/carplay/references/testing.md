# Testing CarPlay Apps

Guide for testing with CarPlay Simulator, Xcode Simulator, and real hardware.

## Table of Contents

1. [CarPlay Simulator](#carplay-simulator)
2. [Xcode Simulator](#xcode-simulator)
3. [Vehicle & Aftermarket Testing](#vehicle--aftermarket-testing)
4. [Recommended Screen Sizes](#recommended-screen-sizes)
5. [Instrument Cluster Testing](#instrument-cluster-testing)
6. [Metadata Testing](#metadata-testing)
7. [What to Test](#what-to-test)

---

## CarPlay Simulator

CarPlay Simulator is a standalone Mac app that simulates a car environment. It most closely matches real CarPlay behavior.

### Setup

1. Download "Additional Tools for Xcode" from [developer.apple.com/download/all](https://developer.apple.com/download/all/) (under "More Downloads").
2. Locate CarPlay Simulator in the **Hardware** folder.
3. Run it and connect iPhone via USB cable.
4. CarPlay starts on iPhone as if connected to a real car.

### Configuration

Click **Configure** to change display settings: resolution, scale factor, light/dark mode.

## Xcode Simulator

Xcode Simulator provides a second window that acts as the car's display.

### Setup

1. Launch Simulator.
2. Select **I/O → External Displays → CarPlay**.
3. A CarPlay screen window appears.

### Enabling Extra Options (Navigation Apps)

To test different display configurations in Xcode Simulator, run this Terminal command **before** launching:

```bash
defaults write com.apple.iphonesimulator CarPlayExtraOptions -bool YES
```

> **Note**: Xcode Simulator does not simulate the instrument cluster or show metadata.

### Limitations of Xcode Simulator

Xcode Simulator is good for rapid build/test cycles, but cannot test:

- **Locked iPhone** — most people use CarPlay with a locked phone.
- **Runtime scenarios** — switching between CarPlay and the car's built-in UI, connecting/disconnecting.
- **Concurrent audio** — other audio sources (e.g., FM radio) playing alongside your app.
- **Siri features**.
- **Instrument cluster displays**.

Use CarPlay Simulator or a real vehicle for these scenarios.

## Vehicle & Aftermarket Testing

You can test with an actual vehicle or an aftermarket head unit with a power supply. If using an aftermarket unit, choose one that supports **wireless CarPlay** so you can simultaneously connect iPhone to the head unit and to Xcode via cable.

## Recommended Screen Sizes

Test your navigation app map drawing with these configurations:

| Name | Resolution | Scale |
|---|---|---|
| Minimum (smallest possible) | 748 × 456 px | 2.0 |
| Standard (typical) | 800 × 480 px | 2.0 |
| High resolution (larger screens) | 1920 × 720 px | 3.0 |
| Portrait (vertical screens) | 900 × 1200 px | 3.0 |

In CarPlay Simulator, use **Configure** to change the display. In Xcode Simulator, enable extra options first (see above).

## Instrument Cluster Testing

Use CarPlay Simulator to test instrument cluster displays.

### Setup

1. Click **Configure → Cluster Display**.
2. Enable **Instrument Cluster Display enabled**.
3. Set scale factor, screen size, safe area, and safe area sizes.

### Recommended Configurations

| Name | Scale | Size | Safe Area Origin | Safe Area Size |
|---|---|---|---|---|
| Minimum | 3x | 300 × 200 | 0, 0 | 300 × 200 |
| Basic | 2x | 640 × 480 | 0, 0 | 640 × 480 |
| Widescreen (wide safe area) | 3x | 1920 × 720 | 420, 0 | 1080 × 720 |
| Widescreen (small safe area) | 2x | 1920 × 720 | 640, 120 | 640 × 480 |

## Metadata Testing

Use CarPlay Simulator to verify that your app supplies correct metadata for instrument cluster and HUD displays.

1. Start an active navigation session.
2. Click **Navigation** to view the next upcoming maneuver.
3. Click **Show More** to see the full sequence of upcoming maneuvers.
4. Inside the detail screen, click the table icons for **Maneuvers** or **Lane Guidances** for detailed info.

## What to Test

### All CarPlay Apps

- App launches correctly when connected to CarPlay.
- App works while iPhone is locked.
- Templates display correctly at different resolutions and scale factors.
- Light and dark mode both work.
- Connecting and disconnecting iPhone gracefully.
- Switching between CarPlay and the car's built-in UI.
- All error states display meaningful alerts (without instructing the user to pick up iPhone).
- Template depth does not exceed category limits.

### Audio Apps

- Audio session activates only when playback starts (not at launch).
- Now playing template is populated at all times.
- Playback works alongside car's FM radio (no premature interruption).
- Siri integration works for controlling playback.

### Navigation Apps

- Map renders correctly at all recommended screen sizes.
- Light and dark mode maps.
- Safe areas are respected.
- Panning mode works via map button (for knob-only cars).
- Route guidance flow: search → preview → route selection → guidance → end.
- Maneuvers display correctly, including rapid succession.
- Lane guidance renders in the second maneuver slot.
- Voice prompts mix correctly with car audio.
- Guidance terminates immediately when system requests cancellation.
- CarPlay Dashboard map renders correctly.
- Instrument cluster map follows the rules (minimal, detailed route view, heading up).
- Metadata appears correctly in CarPlay Simulator's navigation panel.

### Communication Apps

- SiriKit intents work: send message, search messages, set attributes.
- VoIP calls work via CallKit.
- Message content is never shown on CarPlay screen.
- Contact template displays and actions work.
