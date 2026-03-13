# Audio & Communication Apps

Guide for building CarPlay audio apps and communication apps.

## Table of Contents

1. [Audio Apps Overview](#audio-apps-overview)
2. [Audio Session Handling](#audio-session-handling)
3. [Now Playing Template Setup](#now-playing-template-setup)
4. [Recording Restrictions](#recording-restrictions)
5. [Communication Apps Overview](#communication-apps-overview)
6. [SiriKit Messaging Intents](#sirikit-messaging-intents)
7. [VoIP Calling with CallKit](#voip-calling-with-callkit)

---

## Audio Apps Overview

CarPlay audio apps use the `com.apple.developer.carplay-audio` entitlement and the CarPlay framework. They provide audio playback services — music, podcasts, audiobooks, radio, etc.

**Key rules:**
- Never show song lyrics on the CarPlay screen.
- The app must be primarily designed to provide audio playback.
- All flows must be meaningful while driving.

**Supported templates**: Action Sheet (iOS 17+), Alert, Grid, List, Tab Bar, Information, Now Playing.

**Template depth limit**: 5 (including root).

**Tab bar limit**: 4 tabs.

## Audio Session Handling

### Activation Timing

Only activate your audio session when you are ready to play audio. Activating too early stops the car's FM radio and other audio sources.

```swift
// WRONG — don't activate at app launch
func templateApplicationScene(_ scene: CPTemplateApplicationScene,
    didConnect interfaceController: CPInterfaceController) {
    try? AVAudioSession.sharedInstance().setActive(true) // BAD
}

// RIGHT — activate when user selects content to play
func startPlayback() {
    try? AVAudioSession.sharedInstance().setActive(true)
    audioPlayer.play()
}
```

### Deactivation

When playback ends or pauses for an extended time, deactivate:

```swift
func stopPlayback() {
    audioPlayer.stop()
    try? AVAudioSession.sharedInstance().setActive(false,
        options: .notifyOthersOnDeactivation)
}
```

## Now Playing Template Setup

The now playing template is a shared singleton. Configure it when your interface controller connects, because iOS can present it at any time (e.g., from the CarPlay home screen).

```swift
import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?

    func templateApplicationScene(
        _ scene: CPTemplateApplicationScene,
        didConnect interfaceController: CPInterfaceController
    ) {
        self.interfaceController = interfaceController

        // Configure now playing immediately
        let nowPlaying = CPNowPlayingTemplate.shared
        let rateButton = CPNowPlayingPlaybackRateButton { _ in
            // Toggle rate: 1x → 1.5x → 2x → 1x
        }
        let repeatButton = CPNowPlayingRepeatButton { _ in
            // Toggle repeat mode
        }
        nowPlaying.updateNowPlayingButtons([rateButton, repeatButton])
        nowPlaying.isUpNextButtonEnabled = true
        nowPlaying.upNextTitle = "Playing Next"

        // Set up root template
        let listTemplate = buildBrowseList()
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }
}
```

### Responding to "Up Next"

When the user taps "Playing Next", push a list template with the upcoming queue:

```swift
extension CarPlaySceneDelegate: CPNowPlayingTemplateObserver {
    func nowPlayingTemplateUpNextButtonTapped(_ template: CPNowPlayingTemplate) {
        let queueItems = playbackQueue.map { track in
            CPListItem(text: track.title, detailText: track.artist)
        }
        let queueTemplate = CPListTemplate(
            title: "Playing Next",
            sections: [CPListSection(items: queueItems)]
        )
        interfaceController?.pushTemplate(queueTemplate, animated: true)
    }
}
```

### Populating MPNowPlayingInfoCenter

CarPlay reads from `MPNowPlayingInfoCenter` for track metadata. Keep it updated:

```swift
import MediaPlayer

func updateNowPlayingInfo(track: Track) {
    var info = [String: Any]()
    info[MPMediaItemPropertyTitle] = track.title
    info[MPMediaItemPropertyArtist] = track.artist
    info[MPMediaItemPropertyPlaybackDuration] = track.duration
    info[MPNowPlayingInfoPropertyElapsedPlaybackTime] = audioPlayer.currentTime
    info[MPNowPlayingInfoPropertyPlaybackRate] = audioPlayer.rate

    if let artwork = track.artworkImage {
        info[MPMediaItemPropertyArtwork] = MPMediaItemArtwork(
            boundsSize: artwork.size
        ) { _ in artwork }
    }

    MPNowPlayingInfoCenter.default().nowPlayingInfo = info
}
```

## Recording Restrictions

Recording is **not supported** while in CarPlay. Do not activate an audio session with recording enabled — it affects other audio sources and the car's voice assistant/phone calls.

**Exception**: Navigation and voice-based conversational apps may use recording for voice input, but **only** in conjunction with the voice control template.

```swift
// While in CarPlay, configure WITHOUT recording
let session = AVAudioSession.sharedInstance()
try session.setCategory(.playback, mode: .default)
// NOT .playAndRecord
```

## Communication Apps Overview

Communication apps use the `com.apple.developer.carplay-communication` entitlement. They provide short-form messaging and/or VoIP calling.

**Requirements:**
- Must provide either short-form text messaging, VoIP calling, or both.
- Email is **not** considered short-form messaging and is not permitted.
- Never show the content of messages, texts, or emails on the CarPlay screen.

**Supported templates**: Action Sheet, Alert, Grid, List, Tab Bar, Information, Now Playing (iOS 17+), Contact.

**Template depth limit**: 5.

## SiriKit Messaging Intents

Communication apps providing text messaging must support all three of these SiriKit intents:

### 1. Send a Message

```swift
import Intents

class SendMessageIntentHandler: NSObject, INSendMessageIntentHandling {
    func handle(intent: INSendMessageIntent,
                completion: @escaping (INSendMessageIntentResponse) -> Void) {
        guard let recipients = intent.recipients,
              let content = intent.content else {
            completion(INSendMessageIntentResponse(code: .failure, userActivity: nil))
            return
        }
        // Send the message via your service
        MessageService.send(content: content, to: recipients) { success in
            let code: INSendMessageIntentResponseCode = success ? .success : .failure
            completion(INSendMessageIntentResponse(code: code, userActivity: nil))
        }
    }
}
```

### 2. Search for Messages

```swift
class SearchForMessagesIntentHandler: NSObject, INSearchForMessagesIntentHandling {
    func handle(intent: INSearchForMessagesIntent,
                completion: @escaping (INSearchForMessagesIntentResponse) -> Void) {
        let messages = MessageService.search(
            sender: intent.sender,
            dateRange: intent.dateTimeRange
        )
        let response = INSearchForMessagesIntentResponse(code: .success, userActivity: nil)
        response.messages = messages.map { $0.toINMessage() }
        completion(response)
    }
}
```

### 3. Set Message Attribute

```swift
class SetMessageAttributeIntentHandler: NSObject, INSetMessageAttributeIntentHandling {
    func handle(intent: INSetMessageAttributeIntent,
                completion: @escaping (INSetMessageAttributeIntentResponse) -> Void) {
        guard let identifier = intent.identifiers?.first else {
            completion(INSetMessageAttributeIntentResponse(code: .failure, userActivity: nil))
            return
        }
        MessageService.setAttribute(intent.attribute, for: identifier)
        completion(INSetMessageAttributeIntentResponse(code: .success, userActivity: nil))
    }
}
```

## VoIP Calling with CallKit

Communication apps providing VoIP calling must support CallKit and the following SiriKit intent:

### Start a Call

```swift
class StartCallIntentHandler: NSObject, INStartCallIntentHandling {
    func handle(intent: INStartCallIntent,
                completion: @escaping (INStartCallIntentResponse) -> Void) {
        guard let contact = intent.contacts?.first else {
            completion(INStartCallIntentResponse(code: .failure, userActivity: nil))
            return
        }
        // Initiate VoIP call via CallKit
        CallService.startCall(to: contact) { success in
            let code: INStartCallIntentResponseCode = success ? .success : .failure
            completion(INStartCallIntentResponse(code: code, userActivity: nil))
        }
    }
}
```

### CallKit Integration

Your app must integrate with CallKit for proper call handling in CarPlay:

```swift
import CallKit

class CallManager: NSObject, CXProviderDelegate {
    let provider: CXProvider
    let callController = CXCallController()

    override init() {
        let config = CXProviderConfiguration()
        config.supportsVideo = false
        config.supportedHandleTypes = [.phoneNumber, .generic]
        provider = CXProvider(configuration: config)
        super.init()
        provider.setDelegate(self, queue: nil)
    }

    func startOutgoingCall(handle: String) {
        let uuid = UUID()
        let handle = CXHandle(type: .generic, value: handle)
        let action = CXStartCallAction(call: uuid, handle: handle)
        callController.request(CXTransaction(action: action)) { error in
            if let error = error {
                print("Failed to start call: \(error)")
            }
        }
    }

    func providerDidReset(_ provider: CXProvider) {
        // End all ongoing calls
    }
}
```
