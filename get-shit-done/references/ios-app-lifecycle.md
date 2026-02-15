# iOS App Lifecycle

Reference for GSD agents to understand iOS app lifecycle, scene management, and background execution. Agents use this to plan features that depend on app state transitions, background processing, or multi-window support.

**Complements:** `ios-frameworks.md` (which lists frameworks), this file explains **when and how** they activate.

---

## App Structure

Every SwiftUI app starts with a single `@main` struct conforming to `App`:

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [Item.self])
    }
}
```

**Key points:**
- `WindowGroup` is the standard scene type for iOS apps. Each window is a scene instance.
- The `App` struct is the single entry point — there is no `AppDelegate` by default.
- Use `.modelContainer`, `.environment`, and `.environmentObject` modifiers at the `WindowGroup` level to inject dependencies available to the entire app.

### AppDelegate Bridge

When you need `AppDelegate` functionality (push notification registration, third-party SDK initialization, universal links), use `UIApplicationDelegateAdaptor`:

```swift
@main
struct MyApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

class AppDelegate: NSObject, UIApplicationDelegate {
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil
    ) -> Bool {
        // Push notification registration, SDK init
        return true
    }

    func application(
        _ application: UIApplication,
        didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
    ) {
        // Send token to server
    }
}
```

**Use `UIApplicationDelegateAdaptor` only when required.** Prefer SwiftUI-native modifiers (`.onOpenURL`, `.handlesExternalEvents`) when possible.

---

## Scene Phases

SwiftUI provides `ScenePhase` to observe app state transitions. This replaces the old `UIApplicationDelegate` lifecycle methods.

```swift
@Environment(\.scenePhase) private var scenePhase
```

| Phase | Meaning | Typical Use |
|-------|---------|-------------|
| `.active` | App is in foreground and receiving events | Normal operation |
| `.inactive` | App is visible but not receiving events (e.g., during app switcher, incoming call) | Pause timers, stop animations |
| `.background` | App is not visible | Save state, release resources, schedule background tasks |

### Observing Scene Phase Changes

**At the View level** — for view-specific reactions:

```swift
struct ContentView: View {
    @Environment(\.scenePhase) private var scenePhase

    var body: some View {
        Text(String(localized: "content.greeting"))
            .onChange(of: scenePhase) { oldPhase, newPhase in
                switch newPhase {
                case .active:
                    // Resume data refresh
                    break
                case .inactive:
                    // Pause ongoing work
                    break
                case .background:
                    // Save draft, persist state
                    break
                @unknown default:
                    break
                }
            }
    }
}
```

**At the App level** — for app-wide reactions:

```swift
@main
struct MyApp: App {
    @Environment(\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { oldPhase, newPhase in
            if newPhase == .background {
                // Schedule background refresh
                scheduleAppRefresh()
            }
        }
    }
}
```

**For GSD agents:** When planning features that involve state saving, draft persistence, or data synchronization, verify that `scenePhase` transitions are handled. The verifier should check that `.background` transitions trigger state preservation.

---

## State Restoration

SwiftUI provides `@SceneStorage` for automatic scene-level state persistence across app launches:

```swift
struct EditorView: View {
    @SceneStorage("editor.selectedTab") private var selectedTab = 0
    @SceneStorage("editor.draftText") private var draftText = ""

    var body: some View {
        TabView(selection: $selectedTab) {
            // tabs
        }
    }
}
```

**Rules:**
- `@SceneStorage` persists per-scene (each window has independent state)
- Supports basic types: `String`, `Int`, `Double`, `Bool`, `URL`, `Data`
- Do NOT store sensitive data in `@SceneStorage` — it is not encrypted
- Do NOT store large data — it is meant for UI state, not model data
- Use SwiftData or Keychain for persistent model data and secrets

---

## Background Execution

iOS strictly controls background execution. Apps cannot run indefinitely in the background — they get seconds to minutes depending on the task type.

### Background Tasks Framework

Use `BackgroundTasks` for scheduled work:

```swift
import BackgroundTasks

// 1. Register in App init or didFinishLaunching
BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.app.refresh",
    using: nil
) { task in
    handleAppRefresh(task: task as! BGAppRefreshTask)
}

// 2. Schedule when entering background
func scheduleAppRefresh() {
    let request = BGAppRefreshTaskRequest(identifier: "com.app.refresh")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)
    try? BGTaskScheduler.shared.submit(request)
}

// 3. Handle the task
func handleAppRefresh(task: BGAppRefreshTask) {
    scheduleAppRefresh() // Schedule next refresh

    let operation = Task {
        // Fetch new data
        await refreshData()
        task.setTaskCompleted(success: true)
    }

    task.expirationHandler = {
        operation.cancel()
    }
}
```

| Task Type | Class | Duration | Use Case |
|-----------|-------|----------|----------|
| App Refresh | `BGAppRefreshTask` | ~30 seconds | Fetch latest data, sync small payloads |
| Processing | `BGProcessingTask` | Minutes (system-managed) | Database maintenance, ML model updates, large sync |

### Info.plist Registration

Background tasks MUST be declared in Info.plist:

```xml
<key>BGTaskSchedulerPermittedIdentifiers</key>
<array>
    <string>com.app.refresh</string>
    <string>com.app.processing</string>
</array>
```

### Background Modes

Some features require continuous background execution. Enable in Xcode under Signing & Capabilities > Background Modes:

| Mode | Purpose | Example |
|------|---------|---------|
| Audio | Audio playback/recording in background | Music player, podcast app |
| Location updates | Continuous location tracking | Navigation, fitness tracking |
| Voice over IP | VoIP push notifications | Communication apps |
| Background fetch | Periodic content refresh | News app, email client |
| Remote notifications | Silent push to trigger work | Server-driven data sync |
| Background processing | Long-running tasks | Data migration, ML training |

**For GSD agents:** When a plan involves background work, verify:
1. Task identifier is registered in `BGTaskScheduler`
2. Identifier is declared in Info.plist `BGTaskSchedulerPermittedIdentifiers`
3. Appropriate background mode is enabled in capabilities
4. Expiration handler cancels work gracefully
5. Task calls `setTaskCompleted(success:)` in all code paths

---

## Push Notifications Lifecycle

Push notification handling spans multiple lifecycle stages:

### Registration Flow

```swift
// 1. Request permission (see ios-permissions.md)
let center = UNUserNotificationCenter.current()
let granted = try await center.requestAuthorization(options: [.alert, .badge, .sound])

// 2. Register for remote notifications (requires AppDelegate)
UIApplication.shared.registerForRemoteNotifications()

// 3. Receive token in AppDelegate
func application(
    _ application: UIApplication,
    didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
) {
    let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
    // Send token to your server
}
```

### Handling Notifications

```swift
// In App struct or AppDelegate
class NotificationDelegate: NSObject, UNUserNotificationCenterDelegate {
    // Called when notification arrives while app is in foreground
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification
    ) async -> UNNotificationPresentationOptions {
        return [.banner, .badge, .sound]
    }

    // Called when user taps notification
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse
    ) async {
        let userInfo = response.notification.request.content.userInfo
        // Navigate to relevant content
    }
}
```

### Silent Push Notifications

For server-triggered background data sync:

```swift
// AppDelegate
func application(
    _ application: UIApplication,
    didReceiveRemoteNotification userInfo: [AnyHashable: Any]
) async -> UIBackgroundFetchResult {
    // Fetch new data, return .newData, .noData, or .failed
    await syncData()
    return .newData
}
```

Requires "Remote notifications" background mode enabled.

---

## Deep Links and URL Handling

### Universal Links and Custom URL Schemes

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    // Handle deep link
                    Router.shared.handle(url)
                }
        }
    }
}
```

### Associated Domains

For universal links, configure in Signing & Capabilities:

```
applinks:example.com
```

And host an `apple-app-site-association` file on your server.

**For GSD agents:** When a plan involves deep linking, verify:
1. `.onOpenURL` handler exists at the appropriate navigation level
2. URL parsing is robust (handles malformed URLs gracefully)
3. Associated Domains capability is configured (for universal links)
4. The app navigates to the correct content after handling the URL

---

## Common Lifecycle Patterns for GSD Agents

### Pattern: Data Refresh on Foreground

```swift
.onChange(of: scenePhase) { _, newPhase in
    if newPhase == .active {
        Task { await viewModel.refreshIfStale() }
    }
}
```

Use when data may have changed while the app was backgrounded (social feeds, messages, notifications).

### Pattern: Save Draft on Background

```swift
.onChange(of: scenePhase) { _, newPhase in
    if newPhase == .background {
        viewModel.saveDraft()
    }
}
```

Use for any screen with unsaved user input (editors, forms, compose screens).

### Pattern: Pause/Resume Timers

```swift
.onChange(of: scenePhase) { _, newPhase in
    switch newPhase {
    case .active: timer.resume()
    case .inactive, .background: timer.pause()
    @unknown default: break
    }
}
```

Use for workout timers, countdowns, or polling intervals.

---

## Anti-Patterns

| Anti-Pattern | Problem | Correct Approach |
|-------------|---------|-----------------|
| Using `UIApplication.shared.state` in SwiftUI | Bypasses SwiftUI state management | Use `@Environment(\.scenePhase)` |
| Infinite background loops | App will be terminated by system | Use `BGTaskScheduler` with proper expiration |
| Saving state only in `applicationWillTerminate` | Not called reliably on modern iOS | Save on `.background` scene phase |
| Polling with `Timer` in background | Timer stops when backgrounded | Use `BGAppRefreshTask` or silent push |
| Heavy work in `scenePhase` `.inactive` | Inactive is transient (app switcher) | Do heavy work in `.background`, not `.inactive` |
| Ignoring `task.expirationHandler` | System kills the task ungracefully | Always set expiration handler and cancel work |
