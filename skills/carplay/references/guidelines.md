# CarPlay Guidelines

Complete set of Apple's CarPlay app guidelines. Violating these will result in App Store rejection.

## Table of Contents

1. [Guidelines for Widgets](#guidelines-for-widgets)
2. [Guidelines for All CarPlay Apps](#guidelines-for-all-carplay-apps)
3. [Audio Apps](#audio-apps)
4. [Communication Apps](#communication-apps)
5. [Driving Task Apps](#driving-task-apps)
6. [EV Charging Apps](#ev-charging-apps)
7. [Fueling Apps](#fueling-apps)
8. [Parking Apps](#parking-apps)
9. [Public Safety Apps](#public-safety-apps)
10. [Navigation Apps](#navigation-apps)
11. [Quick Food Ordering Apps](#quick-food-ordering-apps)
12. [Voice-Based Conversational Apps](#voice-based-conversational-apps)

---

## Guidelines for Widgets

Set CarPlay as a disfavored location if your widget:
- Is a game or requires extensive user interaction (more than ~6 taps/refreshes).
- Is non-functional in the car (e.g., relies on data protection classes A or B).
- Primarily exists to launch an app that is not a CarPlay app.

## Guidelines for All CarPlay Apps

1. Must be designed primarily to provide the specified feature for the category.
2. **Never** instruct people to pick up their iPhone. You can notify about error conditions (e.g., login required) without asking them to manipulate the phone.
3. All flows must be possible without interacting with iPhone.
4. All flows must be meaningful while driving. Do not include unrelated features (e.g., unrelated settings, maintenance).
5. No gaming or social networking.
6. **Never** show the content of messages, texts, or emails on the CarPlay screen.
7. Use templates for their intended purpose only. Populate them with specified information types only.
8. All voice interaction must be handled using SiriKit (exceptions: navigation and voice-based conversational apps).

## Audio Apps

1. **Never** show song lyrics on the CarPlay screen.

## Communication Apps

1. Must provide short-form text messaging, VoIP calling, or both. Email is **not** permitted.
2. Messaging apps must support all 3 SiriKit intents:
   - `INSendMessageIntent` (send a message)
   - `INSearchForMessagesIntent` (request a list of messages)
   - `INSetMessageAttributeIntent` (modify message attributes)
3. VoIP calling apps must support CallKit and `INStartCallIntent`.

## Driving Task Apps

1. Must enable tasks people need to do **while driving** — tasks that actually help with the drive, not just tasks done while driving.
2. Must use provided templates only. No custom maps, real-time video, or other UI.
3. Do not show CarPlay UI for non-driving tasks (account setup, detailed settings).
4. Do not refresh data items more than once every **10 seconds**.
5. Do not refresh POI template data more than once every **60 seconds**.
6. Do not create POI-focused location finder apps. Driving task apps must accomplish tasks, not just find locations.
7. Use cases outside the vehicle environment are not permitted.

## EV Charging Apps

1. Must provide meaningful driving-relevant functionality — not just a list of EV chargers.
2. When showing locations on a map, only expose EV charger locations.

## Fueling Apps

1. Must provide meaningful driving-relevant functionality — not just a list of fueling stations.
2. When showing locations on a map, only expose fueling station locations.

## Parking Apps

1. Must provide meaningful driving-relevant functionality — not just a list of parking locations.
2. When showing locations on a map, only expose parking locations.

## Public Safety Apps

1. Must be designed primarily to assist public safety organizations (firefighters, law enforcement, ambulance) in the performance of their responsibilities: dispatch, routing, vehicle and location search.

## Navigation Apps

1. Must provide turn-by-turn directions with upcoming maneuvers.
2. The base view is exclusively for the map. No windows, alerts, panels, overlays, or UI elements in the base view. Lane guidance goes in the secondary maneuver template, not the base view.
3. Use each template for its intended purpose. Maneuver images must represent maneuvers, not other content.
4. Provide a way to enter panning mode via a map button (drag gestures are unavailable in some vehicles).
5. Touch gestures must only be used for their intended map purpose (pan, zoom, pitch, rotate).
6. **Immediately** terminate route guidance when requested (e.g., when the car's built-in nav starts).
7. Handle audio correctly. Voice prompts must work alongside the car's audio system. Do not needlessly activate audio sessions.
8. Ensure your map is appropriate in each supported country.
9. Be open and responsive to feedback from Apple and automakers.
10. Voice control must be limited to navigation features.

## Quick Food Ordering Apps

1. Must be Quick Service Restaurant (QSR) apps designed primarily for driving-oriented orders (drive thru, pick up). Not general retail apps (supermarkets, curbside pickup).
2. Provide meaningful driving-relevant functionality — not just a list of store locations.
3. Simplified ordering only. No full menu. Show recent orders or favorites, limited to 12 items each.
4. When showing locations on a map, only expose your QSR locations.

## Voice-Based Conversational Apps

1. Primary modality must be voice upon launch. Must respond to questions/requests and perform actions.
2. Only hold an audio session open when voice features are actively being used.
3. Optimize for voice interaction in the driving environment. Do not show text or imagery in response to queries.
