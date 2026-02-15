# Stack Analysis Template — iOS

Analysis of the project's technology stack, dependencies, build tools, and deployment pipeline.

> **For the codebase mapper agent** — Fill this template when analyzing an existing iOS project's stack.

---

## Language & Runtime

**Primary Language:** Swift {version}
**Minimum Deployment Target:** iOS {version}+ (e.g., iOS 17.0)
**Supported Platforms:** {iPhone / iPad / Mac (Designed for iPad) / Mac Catalyst / visionOS}
**Xcode Version:** {version}+ required
**Swift Toolchain:** {default / custom — specify if using Swift nightly or specific toolchain}

### Language Features in Use
```
- [ ] Swift Concurrency (async/await, actors, Sendable)
- [ ] Swift Macros (@Observable, @Model, custom)
- [ ] Opaque/Existential Types (some/any)
- [ ] Result Builders (@ViewBuilder, custom)
- [ ] Property Wrappers (@State, @Binding, @Environment, custom)
- [ ] String Catalogs (Localizable.xcstrings)
- [ ] RegexBuilder
- [ ] Swift 6 strict concurrency mode
```

### Concurrency Model
```
Preference hierarchy:
1. Swift Concurrency (async/await, actors) — preferred
2. Combine — only for reactive streams (publisher/subscriber patterns)
3. GCD (Dispatch) — only for legacy/interop
```

---

## UI Framework

**Primary:** {SwiftUI / UIKit / Mixed}
**Navigation:** {NavigationStack / NavigationSplitView / UINavigationController / Coordinator pattern}
**Architecture:** {MVVM with @Observable / MVC / TCA / VIPER}

### SwiftUI Patterns
```
- [ ] @Observable (iOS 17+) vs ObservableObject (legacy)
- [ ] @Environment for dependency injection
- [ ] @Bindable for two-way bindings
- [ ] #Preview macro (not PreviewProvider)
- [ ] ViewModifiers for reusable styling
- [ ] Custom Shapes and Paths
- [ ] GeometryReader usage
- [ ] Animations (.animation, withAnimation, matchedGeometryEffect)
- [ ] Liquid Glass effects (iOS 26+)
```

### UIKit Integration (if mixed)
```
- [ ] UIViewRepresentable wrappers
- [ ] UIViewControllerRepresentable wrappers
- [ ] Hosting controllers (UIHostingController)
- [ ] Coordinator pattern for UIKit navigation
```

---

## Data & Persistence

**Primary Persistence:** {SwiftData / Core Data / UserDefaults / None}
**Cloud Sync:** {CloudKit / Firebase / Custom backend / None}
**Keychain:** {Used for: tokens, credentials, sensitive data / Not used}
**File Storage:** {FileManager usage: documents, caches, app groups / Not used}

### SwiftData Configuration (if used)
```
Models: {list @Model classes}
Container: {ModelContainer configuration — in-memory for previews, persistent for app}
Migration: {Lightweight / Custom VersionedSchema / None needed}
```

### Core Data Configuration (if used)
```
Model: {.xcdatamodeld name}
Stack: {NSPersistentContainer / NSPersistentCloudKitContainer}
Migration: {Lightweight / Custom mapping model}
```

---

## Networking

**HTTP Client:** {URLSession (native) / Alamofire / Custom}
**API Style:** {REST / GraphQL / gRPC / WebSocket}
**Base URL Configuration:** {Hardcoded / Environment-based / Server config}
**Authentication:** {Bearer token / Sign in with Apple / OAuth / API key / None}

### Network Layer Architecture
```
Pattern: {Direct URLSession / Protocol-based APIClient / Repository pattern}
Decoding: {JSONDecoder with custom strategies / Manual / Codable}
Error Handling: {Custom error types / URLError passthrough / Result type}
Caching: {URLCache / Custom / None}
Certificate Pinning: {Yes / No}
```

---

## Dependencies

**Manager:** {Swift Package Manager (SPM) / CocoaPods / Carthage / Mixed}

### SPM Dependencies (Package.swift or Xcode project)
```
| Package | Version | Purpose | Critical? |
|---------|---------|---------|-----------|
| {name} | {version constraint} | {what it does} | {Yes/No} |
```

### CocoaPods Dependencies (if used)
```
| Pod | Version | Purpose | Migration plan to SPM? |
|-----|---------|---------|----------------------|
| {name} | {version} | {what it does} | {Yes/No/N/A} |
```

### Vendor / Manual Dependencies
```
| Framework | Source | Purpose |
|-----------|--------|---------|
| {name} | {local .xcframework / binary} | {what it does} |
```

---

## Testing

**Unit Tests:** {Swift Testing / XCTest / Both}
**UI Tests:** {XCUITest / Not configured}
**Snapshot Tests:** {swift-snapshot-testing / None}
**Code Coverage:** {Enabled in scheme / Not configured}

### Test Structure
```
TestTarget/
├── UnitTests/
│   ├── ViewModelTests/    # @Suite, @Test, #expect
│   ├── ServiceTests/      # Network/data layer
│   └── ModelTests/        # Data model logic
├── IntegrationTests/      # Cross-layer tests
└── UITests/               # XCUITest flows
```

### Test Configuration
```
Scheme: {scheme name for testing}
Simulator: {default test destination — e.g., iPhone 16}
Parallel: {Yes / No}
Test Plans: {.xctestplan files / Default}
```

---

## Build & CI/CD

**Build System:** Xcode Build System (xcodebuild)
**CI/CD:** {Xcode Cloud / GitHub Actions / Fastlane / Bitrise / None}

### Build Configuration
```
Configurations: {Debug / Release / custom}
Schemes: {list schemes and their purposes}
Build Settings: {notable custom settings — OTHER_SWIFT_FLAGS, etc.}
Preprocessor Flags: {DEBUG, STAGING, etc.}
```

### Xcode Cloud / CI Pipeline (if configured)
```
Triggers: {branch push / PR / tag}
Workflows: {build / test / archive / distribute}
Post-actions: {TestFlight upload / App Store submission / Slack notification}
```

### Fastlane Configuration (if used)
```
Lanes: {list lanes — e.g., beta, release, test}
Match: {code signing via match / manual}
Pilot: {TestFlight distribution}
```

---

## Code Signing & Distribution

**Team:** {Apple Developer Team ID}
**Signing:** {Automatic / Manual}
**Provisioning:** {Xcode Managed / Fastlane Match / Manual profiles}
**Distribution:** {App Store / TestFlight / Ad Hoc / Enterprise}

### Certificates & Profiles
```
Development: {status}
Distribution: {status}
Push Notifications: {APNs key / certificate / not configured}
```

---

## Project Structure

**Organization:** {Single target / Multi-target / Multi-module SPM / Workspace}
**Architecture Layers:** {describe folder organization}

### Target Structure
```
| Target | Type | Purpose |
|--------|------|---------|
| {AppName} | App | Main application |
| {AppName}Tests | Unit Test | Swift Testing + XCTest |
| {AppName}UITests | UI Test | XCUITest flows |
| {WidgetExtension} | Widget | Home screen widgets |
| {NotificationExtension} | Notification | Rich push notifications |
```

### Module/Package Structure (if multi-module)
```
| Module | Dependencies | Purpose |
|--------|-------------|---------|
| Core | Foundation | Shared models, utilities |
| Networking | Core | API client, DTOs |
| Features | Core, Networking | Feature modules |
| DesignSystem | SwiftUI | Reusable UI components |
```

---

## App Capabilities & Entitlements

### Capabilities Enabled
```
- [ ] Push Notifications (APNs)
- [ ] Sign in with Apple
- [ ] iCloud (CloudKit / Key-value / Documents)
- [ ] App Groups (shared data between targets)
- [ ] Keychain Sharing
- [ ] Background Modes (fetch, processing, location, audio)
- [ ] HealthKit
- [ ] HomeKit
- [ ] In-App Purchase (StoreKit 2)
- [ ] Maps / Location Services
- [ ] Camera / Photo Library
- [ ] Siri / App Intents
- [ ] Associated Domains (universal links)
- [ ] Network Extensions
```

### Info.plist Privacy Keys
```
| Key | Value | Used by |
|-----|-------|---------|
| NSCameraUsageDescription | {description} | {feature} |
| NSLocationWhenInUseUsageDescription | {description} | {feature} |
| NSPhotoLibraryUsageDescription | {description} | {feature} |
```

---

## Environment & Configuration

**Configuration Files:**
```
- [ ] .xcconfig files (per-environment settings)
- [ ] Info.plist (app metadata, privacy keys)
- [ ] Entitlements file (capabilities)
- [ ] GoogleService-Info.plist (Firebase, if used)
- [ ] Secrets management (keychain / .xcconfig not in git / environment vars in CI)
```

**Environment Switching:**
```
Method: {Xcode schemes / build configurations / custom enum}
Environments: {Development / Staging / Production}
Base URLs: {how API endpoints change per environment}
```

---

## Accessibility

```
- [ ] VoiceOver labels on interactive elements
- [ ] Dynamic Type support (no fixed font sizes)
- [ ] Color contrast WCAG AA compliance
- [ ] Bold Text support
- [ ] Reduce Motion support
- [ ] Accessibility traits (.isButton, .isHeader, etc.)
- [ ] Accessibility actions (custom swipe actions)
- [ ] Accessibility audit passes in Xcode
```

---

## Notes

{Any additional observations about the stack — unusual patterns, migration plans, known tech debt, performance considerations, third-party SDK limitations}
