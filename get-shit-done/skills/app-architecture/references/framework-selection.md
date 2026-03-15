<overview>
Framework preference tables for every iOS domain. Purely tabular — lookup which framework to use for UI, data, networking, concurrency, location, media, auth, payments, testing, and more. Read when choosing between native and third-party options. Related: deployment-targets.md (availability gating), anti-patterns.md (banned patterns).
</overview>

## Framework Decision Hierarchy

When choosing between frameworks, apply these rules in order:

1. **Native over third-party** — Apple frameworks get first-class OS integration, performance, and long-term support
2. **Modern over legacy** — SwiftData over Core Data, Swift Testing over XCTest, App Intents over SiriKit
3. **Simple over complex** — UserDefaults for a boolean, not SwiftData. URLSession for REST, not a custom stack
4. **Privacy-first** — On-device processing (Vision, FoundationModels) over cloud APIs when possible

## UI Frameworks

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **SwiftUI** | primary | All UI — views, navigation, animations, layout |
| **UIKit** | secondary | Camera, complex gestures, drag-and-drop, UIViewRepresentable wrapping |

**Do NOT use:** AppKit (macOS only), Interface Builder / Storyboards.

## Data & Persistence

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **SwiftData** | primary | Structured local persistence with @Model macro |
| **Core Data** | legacy | Same purpose — only for maintaining existing code |
| **UserDefaults** | secondary | Simple key-value preferences (booleans, strings, small values) |
| **Keychain Services** | primary | Secrets — tokens, passwords, API keys |
| **CloudKit** | secondary | iCloud sync, public/private databases |
| **FileManager** | secondary | File system operations — documents, caches |

**Do NOT use:** Realm, SQLite directly (unless extreme perf needs), Firebase Firestore (prefer CloudKit).

## Networking

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **URLSession** | primary | HTTP/HTTPS requests, downloads, uploads, WebSocket |
| **Network.framework** | secondary | Low-level TCP/UDP, NWPathMonitor for connectivity |
| **MultipeerConnectivity** | secondary | Peer-to-peer nearby devices |

**Do NOT use:** Alamofire (URLSession is sufficient), third-party WebSocket libraries.

## Concurrency

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **Swift Concurrency** | primary | async/await, actors, TaskGroups, AsyncSequence |
| **Combine** | secondary | Reactive streams (debounce, throttle, combineLatest) |
| **Grand Central Dispatch** | legacy | Low-level queues — prefer @MainActor, Task |

**Do NOT use:** RxSwift, third-party async libraries, OperationQueue (use TaskGroup).

## Location & Maps

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **CoreLocation** | primary | GPS, heading, region monitoring, geocoding |
| **MapKit** | primary | Map views, annotations, overlays, directions |

**Do NOT use:** Google Maps SDK (MapKit is sufficient for most apps).

## Media

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **AVFoundation** | primary | Audio/video playback, recording, camera capture |
| **PhotosUI** | primary | PhotosPicker in SwiftUI — never build custom picker |
| **CoreImage** | secondary | Image filters, face detection |
| **Vision** | secondary | OCR, barcode scanning, image classification |
| **AVKit** | primary | VideoPlayer in SwiftUI |

**Do NOT use:** UIImagePickerController for photos (use PhotosUI), third-party image processing for basic filters.

## Notifications

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **UserNotifications** | primary | Local and push notifications |
| **ActivityKit** | secondary | Live Activities on Lock Screen and Dynamic Island |

**Do NOT use:** Third-party push wrappers, deprecated UILocalNotification.

## Authentication

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **AuthenticationServices** | primary | Sign in with Apple, passkeys, AutoFill credentials |
| **LocalAuthentication** | primary | Face ID, Touch ID, device passcode |

**Do NOT use:** Third-party auth SDKs as primary (Sign in with Apple first — App Store requirement).

## In-App Purchases

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **StoreKit 2** | primary | Subscriptions, consumables, transaction management |

**RevenueCat** is acceptable when you need subscription analytics or cross-platform billing. For pure iOS apps, StoreKit 2 is sufficient.

**Do NOT use:** Original StoreKit (StoreKit 1), old `/verifyReceipt` endpoint.

## System Integration

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **WidgetKit** | primary | Home Screen, Lock Screen, StandBy widgets |
| **App Intents** | primary | Siri, Shortcuts, Spotlight, interactive widgets |
| **BackgroundTasks** | primary | Background fetch and processing |
| **CoreSpotlight** | secondary | Index app content for Spotlight |

**Do NOT use:** SiriKit (legacy — use App Intents), old background modes.

## Testing

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **Swift Testing** | primary | Unit/integration tests — @Test, #expect, parameterized |
| **XCTest** | secondary | Performance tests, legacy compatibility |
| **XCUITest** | primary | UI automation tests |

**Do NOT use:** Quick/Nimble, third-party UI testing frameworks.

## Data Visualization

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **Swift Charts** | primary | Bar, line, area, point, rule, sector charts |

**Do NOT use:** Third-party charting libraries (Swift Charts covers most use cases).

## Security & Privacy

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **CryptoKit** | primary | Hashing, symmetric encryption, HMAC |
| **App Attest** | secondary | Device integrity verification |
| **Keychain Services** | primary | Secure token/password storage |

**Never disable ATS** in production. Use per-domain exceptions if needed.

## When Third-Party Is Acceptable

- **Firebase Analytics / Crashlytics** — No native equivalent for crash reporting at scale
- **Lottie** — Complex vector animations beyond SwiftUI
- **SDWebImage / Kingfisher** — Only for aggressive caching beyond AsyncImage
- **Fastlane** — CI/CD automation (complementary to Xcode Cloud)
- **RevenueCat** — Cross-platform subscription analytics

For everything else, start native. Only reach for third-party if you hit a concrete limitation.
