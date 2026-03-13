# CarPlay Development Skill

A comprehensive [Agent Skill](https://agentskills.io/) for building Apple CarPlay apps, widgets, and Live Activities. Based on Apple's official **CarPlay Developer Guide (February 2026)**.

## What This Skill Does

When activated in Claude Code (or any compatible agent), this skill provides:

- **Complete API guidance** for all CarPlay app categories: audio, navigation, communication, driving task, EV charging, fueling, parking, public safety, quick food ordering, and voice-based conversational apps.
- **Working Swift code examples** for every major API area — templates, scene delegates, entitlements, audio sessions, route guidance, SiriKit intents, and more.
- **Entitlement & provisioning walkthroughs** from request to Xcode configuration.
- **Template reference** covering all 12 CarPlay templates with usage rules and category compatibility.
- **Navigation app deep-dive** including base view, map template, route guidance flow, maneuver metadata, instrument cluster, HUD support, and voice prompts.
- **Widget & Live Activity support** for CarPlay Dashboard.
- **Apple's complete guidelines** per app category to avoid App Store rejection.
- **Testing guide** for CarPlay Simulator, Xcode Simulator, and real hardware.
- **Common pitfalls** and troubleshooting.

## When It Triggers

The skill activates when you ask about:

- CarPlay, CarPlay Ultra, CarPlay Dashboard
- Any `CPTemplate` class (`CPListTemplate`, `CPMapTemplate`, `CPGridTemplate`, etc.)
- CarPlay entitlements or provisioning
- In-car app development for iPhone
- CarPlay navigation, audio, communication, or any supported app category
- Widgets or Live Activities in CarPlay
- CarPlay Simulator or testing
- Instrument cluster or HUD metadata

## Installation

### Claude Code

```bash
claude skill install https://github.com/cs4alhaider/carplay-skill
```

### Manual

Clone or download this repository and place it in your skills directory:

```bash
git clone https://github.com/cs4alhaider/carplay-skill.git
# Then add the path to your agent's skill configuration
```

## Repository Structure

```
carplay-skill/
├── SKILL.md                              # Core skill file (overview + code examples)
├── README.md                             # This file
├── LICENSE                               # MIT License
├── .gitignore
└── references/
    ├── entitlements.md                   # Entitlement keys, provisioning, Xcode setup
    ├── templates.md                      # All 12 CarPlay templates with code examples
    ├── navigation.md                     # Navigation apps: maps, routing, HUD, clusters
    ├── audio-and-communication.md        # Audio sessions, now playing, SiriKit, CallKit
    ├── widgets-and-live-activities.md    # Widgets & Live Activities in CarPlay
    ├── guidelines.md                     # Apple's complete per-category guidelines
    ├── testing.md                        # Simulators, screen sizes, testing checklist
    └── assets-and-notifications.md       # Image assets, notifications, CarPlay categories
```

## Usage Examples

Once installed, just ask naturally:

- *"How do I add CarPlay support to my audio app?"*
- *"Show me how to set up a CPListTemplate with image rows"*
- *"What entitlement do I need for a CarPlay navigation app?"*
- *"Help me implement route guidance with maneuvers and trip estimates"*
- *"How do I configure voice prompts for my navigation app?"*
- *"What are the guidelines for CarPlay driving task apps?"*
- *"How do I test my CarPlay app in the simulator?"*
- *"Add a widget to my app that shows in CarPlay Dashboard"*

## Source

All content is derived from Apple's official **CarPlay Developer Guide** dated February 2026. External Apple documentation URLs are preserved as references throughout.

## Author

**Abdullah Alhaider** — [github.com/cs4alhaider](https://github.com/cs4alhaider)

## License

[MIT](LICENSE)
