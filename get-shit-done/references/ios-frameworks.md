# iOS Native Frameworks Reference

Opinionated guide to Apple's native frameworks for iOS development. Prefer native over third-party. Prefer modern APIs over legacy.

**Preference levels:** `primary` = default choice | `secondary` = specific use cases | `legacy` = avoid in new code | `conditional` = platform/version gated

---

## UI Frameworks

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **SwiftUI** | primary | All UI — views, navigation, animations, layout |
| **UIKit** | secondary | Camera (UIImagePickerController), complex gestures, drag-and-drop with fine control, embedding in SwiftUI via UIViewRepresentable |

**Use SwiftUI** for all new screens and components. It is the only UI framework Apple actively evolves.

**Use UIKit** only when SwiftUI has no equivalent API (e.g., certain camera flows, document scanners) or when wrapping legacy code. Never start a new screen in UIKit.

**Do NOT use:** AppKit (macOS only), Interface Builder / Storyboards (dead-end tooling).

---

## Data & Persistence

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **SwiftData** | primary | Structured local persistence with Swift-native models |
| **Core Data** | legacy | Same purpose as SwiftData but Objective-C era API |
| **UserDefaults** | secondary | Simple key-value preferences (booleans, strings, small numbers) |
| **Keychain Services** | primary | Secrets — tokens, passwords, API keys |
| **CloudKit** | secondary | iCloud sync, public/private databases, sharing |
| **FileManager** | secondary | File system operations — reading/writing documents, caches |

**Use SwiftData** for any model that needs querying, relationships, or persistence. It uses `@Model` macro and integrates directly with SwiftUI.

**Use Core Data** only when maintaining existing code. Do not introduce Core Data in new projects.

**Use UserDefaults** for simple preferences only. Never store arrays of complex objects, images, or anything > 1KB.

**Use Keychain** for ALL secrets. Never store tokens or passwords in UserDefaults.

**Do NOT use:** Realm, SQLite directly (unless extreme performance requirements with millions of rows), Firebase Firestore (prefer CloudKit for Apple ecosystem).

---

## Networking

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **URLSession** | primary | HTTP/HTTPS requests, downloads, uploads, WebSocket |
| **Network.framework** | secondary | Low-level TCP/UDP, connection monitoring, path evaluation |
| **MultipeerConnectivity** | secondary | Peer-to-peer discovery and communication (nearby devices) |

**Use URLSession** for all REST/GraphQL API calls. Use `async/await` overloads (`data(for:)`, `bytes(for:)`). Build a thin API client — do not use Alamofire.

**Use Network.framework** for network path monitoring (`NWPathMonitor`) to detect connectivity changes. Also use for custom protocol implementations.

**Do NOT use:** Alamofire (URLSession is sufficient), third-party WebSocket libraries (URLSession supports WebSocket natively).

---

## Concurrency

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **Swift Concurrency** | primary | async/await, actors, TaskGroups, AsyncSequence |
| **Combine** | secondary | Reactive streams, publisher/subscriber pattern |
| **Grand Central Dispatch** | secondary | Low-level queue management, DispatchSemaphore |

**Use Swift Concurrency** for everything. `async/await` for sequential async work. `TaskGroup` for parallel work. `@MainActor` for UI-bound code. `actor` for thread-safe mutable state.

**Use Combine** when you need reactive operators like `debounce`, `throttle`, `combineLatest`, or when interfacing with APIs that return publishers. Combine is not deprecated — it is secondary to async/await for most use cases, but remains the right tool for reactive streams.

**Use GCD** only for `DispatchQueue.main.async` bridging in legacy code or very specific low-level scenarios. Prefer `@MainActor` instead.

**Do NOT use:** RxSwift (dead ecosystem), third-party async libraries, OperationQueue (use TaskGroup).

---

## Location & Maps

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **CoreLocation** | primary | GPS location, heading, region monitoring, geocoding |
| **MapKit** | primary | Map views, annotations, overlays, directions |

**Use CoreLocation** for all location needs. Use `CLLocationManager` with the new `async` APIs. Always request minimum necessary authorization (`whenInUse` over `always`).

**Use MapKit** for maps. SwiftUI `Map` view is the default. Use `MKMapView` via UIViewRepresentable only for unsupported features.

**Do NOT use:** Google Maps SDK (unnecessary dependency, MapKit is sufficient for most apps), third-party geocoding.

---

## Media

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **AVFoundation** | primary | Audio/video playback, recording, editing, camera capture |
| **PhotosUI** | primary | Photo picker (`PHPickerViewController`), photo library access |
| **CoreImage** | secondary | Image filters, face detection in images |
| **Vision** | secondary | Image analysis — text recognition (OCR), barcode detection, object tracking |
| **AVKit** | primary | Video player UI (`VideoPlayer` in SwiftUI) |

**Use PhotosUI** for picking photos/videos. Use `PhotosPicker` in SwiftUI — never build a custom photo picker.

**Use AVFoundation** for custom camera interfaces, audio recording, and video editing.

**Use Vision** for on-device ML tasks: OCR (`VNRecognizeTextRequest`), barcode scanning, image classification.

**Do NOT use:** UIImagePickerController for photo selection (use PhotosUI), third-party image processing libraries for basic filters (use CoreImage).

---

## Notifications

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **UserNotifications** | primary | Local and push notifications — scheduling, content, actions |
| **ActivityKit** | secondary | Live Activities on Lock Screen and Dynamic Island |

**Use UserNotifications** for all notification needs. Use `UNUserNotificationCenter` for scheduling and handling.

**Use ActivityKit** for real-time status updates (delivery tracking, sports scores, timers). Requires `ActivityAttributes` and `ActivityContent`.

**Do NOT use:** third-party push notification wrappers (Apple's API is straightforward), deprecated `UILocalNotification`.

---

## Authentication

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **AuthenticationServices** | primary | Sign in with Apple, passkeys, AutoFill credentials |
| **LocalAuthentication** | primary | Face ID, Touch ID, device passcode verification |

**Use AuthenticationServices** for Sign in with Apple (required if you offer third-party login). Also use for passkey support and credential AutoFill.

**Use LocalAuthentication** for biometric authentication to protect sensitive app features. Use `LAContext` with `evaluatePolicy`.

**Do NOT use:** third-party auth SDKs as primary auth (offer Sign in with Apple first — App Store requirement), custom biometric UI.

---

## In-App Purchases

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **StoreKit 2** | primary | Subscriptions, consumables, non-consumables, transaction management |

**Use StoreKit 2** exclusively. It provides `async/await` APIs, `Transaction.currentEntitlements`, and `Product.SubscriptionInfo`. Handle all purchase flows, restoration, and receipt validation through StoreKit 2.

**Use StoreKit 2** as the default. **RevenueCat** is acceptable when the project needs subscription analytics dashboards or cross-platform billing (iOS + Android + web). For pure iOS apps with straightforward IAP, StoreKit 2 is sufficient.

**Do NOT use:** Original StoreKit (StoreKit 1), server-side receipt validation with Apple's old `/verifyReceipt` endpoint (use App Store Server API or StoreKit 2 on-device verification).

---

## System Integration

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **WidgetKit** | primary | Home Screen, Lock Screen, and StandBy widgets |
| **App Intents** | primary | Siri, Shortcuts, Spotlight integration, interactive widgets |
| **BackgroundTasks** | primary | Background fetch, processing tasks |
| **CoreSpotlight** | secondary | Index app content for Spotlight search |
| **ActivityKit** | secondary | Live Activities (also listed under Notifications) |

**Use WidgetKit** for all widgets. Use `TimelineProvider` for data, SwiftUI for rendering. Support multiple widget families.

**Use App Intents** for Siri and Shortcuts. Use `@Parameter`, `@Dependency`, and `AppEntity` to expose app functionality. Also required for interactive widgets and Control Center controls.

**Use BackgroundTasks** for scheduled background work. Register `BGAppRefreshTask` for periodic updates and `BGProcessingTask` for heavy work.

**Do NOT use:** SiriKit (legacy — replaced by App Intents), old background modes for fetch (use BackgroundTasks framework).

---

## Testing

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **Swift Testing** | primary | Unit and integration tests — `@Test`, `#expect`, parameterized tests |
| **XCTest** | secondary | Unit tests when Swift Testing lacks coverage, performance tests |
| **XCUITest** | primary | UI automation tests — tapping, swiping, verifying UI elements |

**Use Swift Testing** for all new unit/integration tests. Use `@Test` macro, `#expect` for assertions, `@Suite` for grouping. Supports parameterized tests with `arguments:`.

**Use XCTest** only for performance testing (`measure {}`) or when integrating with tools that require XCTest format.

**Use XCUITest** for end-to-end UI tests. Use accessibility identifiers for reliable element queries.

**Do NOT use:** Quick/Nimble (unnecessary with Swift Testing), third-party UI testing frameworks (XCUITest is sufficient).

---

## Data Visualization

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **Swift Charts** | primary | Charts and data visualization — bar, line, area, point, rule, sector charts |

**Use Swift Charts** for all charting and data visualization needs. It integrates natively with SwiftUI, supports accessibility out of the box (VoiceOver reads chart data), and handles Dark Mode, Dynamic Type, and localization automatically.

```swift
import Charts

Chart(salesData) { item in
    BarMark(
        x: .value("Month", item.month),
        y: .value("Sales", item.amount)
    )
}
```

**Do NOT use:** third-party charting libraries (Charts by danielgindi, SwiftUICharts, etc.) — Swift Charts covers the vast majority of use cases with better OS integration.

---

## Accessibility

SwiftUI has accessibility built in — semantic controls (Button, Toggle, Slider, Picker) get VoiceOver support automatically. Explicit APIs are needed when:

| API | When to Use |
|-----|-------------|
| `.accessibilityLabel(_:)` | Icon-only buttons, images that convey meaning, custom controls |
| `.accessibilityHint(_:)` | When the action isn't obvious from the label alone |
| `.accessibilityHidden(true)` | Decorative images, redundant text |
| `.accessibilityElement(children:)` | Group or combine child elements for VoiceOver |
| `AccessibilityNotification` | Post notifications to VoiceOver (e.g., `.announcement` for dynamic content changes, `.screenChanged` after navigation) |
| `AXCustomContent` | Provide additional context for VoiceOver on complex cells (e.g., "Price: $9.99, Rating: 4.5 stars") without cluttering the default label |

**Use Accessibility Inspector** (Xcode > Open Developer Tool) to audit every screen. Accessibility is mandatory — see `ios-swift-guidelines.md` for the full checklist.

---

## Security & Privacy

| Framework | Preference | Purpose |
|-----------|-----------|---------|
| **CryptoKit** | primary | Hashing (SHA256, SHA512), symmetric encryption (AES-GCM), HMAC, key agreement |
| **App Attest** | secondary | Device integrity verification, anti-fraud for sensitive server requests |
| **App Transport Security (ATS)** | mandatory | Enforces HTTPS for all network connections (enabled by default) |
| **Keychain Services** | primary | Secure storage for tokens, passwords, credentials |

**Use CryptoKit** for all cryptographic operations. It is fast, hardware-accelerated, and the API is safe by design (no buffer overflows, no memory leaks).

**Use App Attest** (`DCAppAttestService`) for fraud prevention in apps with sensitive server APIs (e.g., in-app currency, high-value transactions).

**Never disable ATS** (`NSAllowsArbitraryLoads`) in production. If a third-party API only supports HTTP, use per-domain exceptions in Info.plist — not a blanket override.

Security is a pillar of App Store Review. Apps with security flaws get rejected.

---

## Developer Tools

| Tool | Purpose |
|------|---------|
| **Instruments** | Performance profiling — Time Profiler, Allocations, Leaks, Network, Core Animation |
| **Xcode Previews** | Real-time SwiftUI view previews with `#Preview` macro |
| **Console.app** | View unified system logs from device/simulator |
| **Accessibility Inspector** | Audit VoiceOver labels, traits, and accessibility hierarchy |
| **Xcode Memory Graph** | Debug retain cycles and memory leaks visually |
| **Network Link Conditioner** | Simulate poor network conditions for testing |

**Use Instruments** regularly — profile before optimizing. Time Profiler for CPU, Allocations for memory, Leaks for retain cycles.

**Use Xcode Previews** for all SwiftUI development. Provide meaningful preview data. Use `#Preview` macro (not the old `PreviewProvider` protocol).

**Use Accessibility Inspector** to verify every screen — VoiceOver support is not optional.

---

## iOS 26+ Features (Conditional)

These features require iOS 26 or later. Gate with `if #available(iOS 26, *)` or set deployment target to iOS 26+.

| Framework / API | Preference | Purpose |
|----------------|-----------|---------|
| **Liquid Glass** (`.glassEffect`) | conditional | Translucent glass material for UI surfaces — new design language in iOS 26 |
| **FoundationModels** | conditional | On-device LLM inference using Apple Intelligence models |
| **Visual Intelligence** | conditional | Camera-based visual search and contextual actions |

### Liquid Glass

Use `.glassEffect` modifier for glass material surfaces. Replaces `.ultraThinMaterial` as the preferred translucent background. Applies to navigation bars, tab bars, and custom surfaces.

```swift
// iOS 26+
.glassEffect(.regular)
.glassEffect(.regular.interactive)
```

Only use when targeting iOS 26+. Fall back to `.ultraThinMaterial` for earlier versions.

### FoundationModels

On-device language model for text generation, summarization, and extraction. Runs entirely on-device — no network required, no data leaves the device.

```swift
// iOS 26+
import FoundationModels
let session = LanguageModelSession()
let response = try await session.respond(to: "Summarize this text")
```

Use for on-device text tasks where privacy is critical. Do NOT use as a replacement for server-side AI — the on-device model has limited capabilities compared to cloud models.

### Visual Intelligence

Camera-based contextual understanding. Identify objects, translate text, and get contextual actions from the camera feed. Available on devices with Apple Intelligence support.

Use when building camera-based search or contextual features. Do NOT use for basic barcode/QR scanning (use Vision framework instead).

---

## Framework Decision Hierarchy

When choosing between frameworks:

1. **Native over third-party** — Apple frameworks get first-class OS integration, performance, and long-term support
2. **Modern over legacy** — SwiftData over Core Data, Swift Testing over XCTest, App Intents over SiriKit, Swift Concurrency over Combine
3. **Simple over complex** — UserDefaults for a boolean, not SwiftData. URLSession for REST, not a custom networking stack
4. **Privacy-first** — On-device processing (Vision, FoundationModels) over cloud APIs when possible

### When third-party is acceptable

- **Firebase Analytics / Crashlytics** — No native equivalent for crash reporting and analytics at scale
- **Lottie** — Complex vector animations beyond what SwiftUI animations handle
- **SDWebImage / Kingfisher** — Only if you need aggressive image caching beyond `AsyncImage` capabilities
- **Fastlane** — CI/CD automation (complementary to Xcode Cloud, not a replacement)

For everything else, start with the native framework. Only reach for third-party if you hit a concrete limitation.
