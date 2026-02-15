# iOS Permissions

Reference for GSD agents to understand the iOS permission model. Agents use this to verify that features requiring user permissions have proper Info.plist declarations, runtime request flows, and graceful denial handling.

**Complements:** `ios-frameworks.md` (which lists frameworks by category), this file covers the **permission layer** those frameworks require.

---

## How iOS Permissions Work

iOS uses a two-layer permission system:

1. **Info.plist declaration** — The app declares WHY it needs access via a usage description key. Without this key, the app crashes at runtime when requesting the permission.
2. **Runtime request** — The app asks the user for permission at the moment it needs access. The system shows a dialog with the usage description from Info.plist.

**The user decides.** They can grant, deny, or revoke permissions at any time in Settings. Apps MUST handle all three states gracefully.

---

## Info.plist Privacy Keys

Every permission requires a usage description string in Info.plist. These strings MUST be localized (use InfoPlist.strings or String Catalog).

### Common Permissions

| Permission | Info.plist Key | Framework | When Required |
|-----------|---------------|-----------|---------------|
| Camera | `NSCameraUsageDescription` | AVFoundation | Photo/video capture, QR scanning |
| Photo Library (read) | `NSPhotoLibraryUsageDescription` | PhotosUI | Browsing full photo library |
| Photo Library (add) | `NSPhotoLibraryAddUsageDescription` | Photos | Saving images/videos only (no read access) |
| Microphone | `NSMicrophoneUsageDescription` | AVFoundation | Audio recording, voice messages, video calls |
| Location (when in use) | `NSLocationWhenInUseUsageDescription` | CoreLocation | GPS while app is in foreground |
| Location (always) | `NSLocationAlwaysAndWhenInUseUsageDescription` | CoreLocation | GPS in background (geofencing, tracking) |
| Contacts | `NSContactsUsageDescription` | Contacts | Reading/writing contacts |
| Calendars (full access) | `NSCalendarsFullAccessUsageDescription` | EventKit | Reading/writing calendar events |
| Reminders (full access) | `NSRemindersFullAccessUsageDescription` | EventKit | Reading/writing reminders |
| Bluetooth | `NSBluetoothAlwaysUsageDescription` | CoreBluetooth | BLE device communication |
| Face ID | `NSFaceIDUsageDescription` | LocalAuthentication | Biometric authentication |
| Health | `NSHealthShareUsageDescription` | HealthKit | Reading health data |
| Health (write) | `NSHealthUpdateUsageDescription` | HealthKit | Writing health data |
| Motion | `NSMotionUsageDescription` | CoreMotion | Accelerometer, gyroscope, pedometer |
| Speech Recognition | `NSSpeechRecognitionUsageDescription` | Speech | On-device speech-to-text |
| Tracking | `NSUserTrackingUsageDescription` | AppTrackingTransparency | Cross-app tracking (IDFA) |
| Local Network | `NSLocalNetworkUsageDescription` | Network | Discovering devices on local network |

### Notification Permission

Notifications use a different mechanism — no Info.plist key required, but runtime authorization is mandatory:

```swift
let center = UNUserNotificationCenter.current()
let granted = try await center.requestAuthorization(options: [.alert, .badge, .sound])
```

---

## Runtime Permission Requests

### General Pattern

```swift
// 1. Check current status
let status = AVCaptureDevice.authorizationStatus(for: .video)

switch status {
case .notDetermined:
    // 2. Request permission (first time only)
    let granted = await AVCaptureDevice.requestAccess(for: .video)
    if granted {
        // Proceed with camera
    } else {
        // Show explanation, offer Settings link
    }
case .authorized:
    // Proceed with camera
case .denied, .restricted:
    // Show explanation, offer Settings link
@unknown default:
    break
}
```

### Framework-Specific Request APIs

| Permission | Check Status | Request Access |
|-----------|-------------|----------------|
| Camera | `AVCaptureDevice.authorizationStatus(for: .video)` | `AVCaptureDevice.requestAccess(for: .video)` |
| Microphone | `AVCaptureDevice.authorizationStatus(for: .audio)` | `AVCaptureDevice.requestAccess(for: .audio)` |
| Photos (full) | `PHPhotoLibrary.authorizationStatus(for: .readWrite)` | `PHPhotoLibrary.requestAuthorization(for: .readWrite)` |
| Photos (add only) | `PHPhotoLibrary.authorizationStatus(for: .addOnly)` | `PHPhotoLibrary.requestAuthorization(for: .addOnly)` |
| Location | `CLLocationManager().authorizationStatus` | `CLLocationManager().requestWhenInUseAuthorization()` |
| Contacts | `CNContactStore.authorizationStatus(for: .contacts)` | `CNContactStore().requestAccess(for: .contacts)` |
| Notifications | `UNUserNotificationCenter.current().notificationSettings()` | `UNUserNotificationCenter.current().requestAuthorization(options:)` |
| Tracking | `ATTrackingManager.trackingAuthorizationStatus` | `ATTrackingManager.requestTrackingAuthorization()` |
| HealthKit | `HKHealthStore().authorizationStatus(for: type)` | `HKHealthStore().requestAuthorization(toShare:read:)` |
| Face ID | N/A (evaluate directly) | `LAContext().evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics)` |

### PhotosPicker Exception

`PhotosPicker` in SwiftUI does NOT require any permission. It runs in a separate process and gives the app access only to selected photos. Prefer `PhotosPicker` over `PHPhotoLibrary` when the user just needs to pick photos:

```swift
import PhotosUI

struct ContentView: View {
    @State private var selectedItem: PhotosPickerItem?

    var body: some View {
        PhotosPicker(
            String(localized: "photos.selectButton"),
            selection: $selectedItem,
            matching: .images
        )
    }
}
```

No Info.plist key needed. No permission dialog shown.

---

## Permission States

Every permission has these possible states:

| State | Meaning | App Behavior |
|-------|---------|-------------|
| `.notDetermined` | User has never been asked | Request permission (dialog will appear) |
| `.authorized` | User granted access | Proceed with feature |
| `.denied` | User explicitly denied | Show explanation + Settings link |
| `.restricted` | Parental controls or MDM block access | Show explanation (user cannot change in Settings) |
| `.limited` | User granted partial access (Photos only) | Work with limited selection, offer to expand |
| `.provisional` | Notifications delivered quietly (no explicit grant) | Notifications appear in Notification Center only |

### Handling Denied Permissions

NEVER repeatedly ask for permission after denial — the system dialog only appears once for `.notDetermined`. After denial, guide the user to Settings:

```swift
func openAppSettings() {
    guard let settingsURL = URL(string: UIApplication.openSettingsURLString) else { return }
    UIApplication.shared.open(settingsURL)
}
```

Present a clear, localized explanation BEFORE sending the user to Settings:

```swift
struct PermissionDeniedView: View {
    let permissionName: String
    let reason: String

    var body: some View {
        ContentUnavailableView {
            Label(permissionName, systemImage: "lock.shield")
        } description: {
            Text(reason)
        } actions: {
            Button(String(localized: "permissions.openSettings")) {
                openAppSettings()
            }
        }
    }
}
```

---

## Best Practices

### 1. Ask at the Moment of Need

Request permission when the user taps a feature that requires it — never on app launch. This gives context for why the permission is needed.

```swift
// WRONG — asking on launch
struct MyApp: App {
    init() {
        CLLocationManager().requestWhenInUseAuthorization() // Why?
    }
}

// CORRECT — asking when user taps "Find Nearby"
Button(String(localized: "nearby.findButton")) {
    await requestLocationAndSearch()
}
```

### 2. Pre-Permission Explanation

For high-sensitivity permissions (location always, health, tracking), show a custom explanation screen BEFORE triggering the system dialog. This improves grant rates because the user understands the value:

```swift
struct LocationExplanationView: View {
    let onContinue: () -> Void

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "location.circle.fill")
                .font(.system(size: 60))
                .foregroundStyle(.blue)
                .accessibilityHidden(true)
            Text(String(localized: "location.explanation.title"))
                .font(.title2.bold())
            Text(String(localized: "location.explanation.body"))
                .multilineTextAlignment(.center)
                .foregroundStyle(.secondary)
            Button(String(localized: "location.explanation.continue")) {
                onContinue()
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

### 3. Request Minimum Necessary Access

| Need | Request | NOT |
|------|---------|-----|
| Show user on map | `.requestWhenInUseAuthorization()` | `.requestAlwaysAuthorization()` |
| Save photo to library | `.requestAuthorization(for: .addOnly)` | `.requestAuthorization(for: .readWrite)` |
| Let user pick photos | `PhotosPicker` (no permission needed) | `PHPhotoLibrary.requestAuthorization()` |
| One-time location | `CLLocationManager` with delegate stop | Continuous location updates |

### 4. Localize Usage Description Strings

Info.plist usage descriptions appear in the system permission dialog. They MUST be localized:

**Using String Catalog (preferred):**

Create `InfoPlist.xcstrings` in your project. Add entries for each `NS*UsageDescription` key with translations per locale.

**Using InfoPlist.strings (legacy):**

```
// InfoPlist.strings (pt-BR)
"NSCameraUsageDescription" = "Precisamos da câmera para escanear documentos.";
"NSLocationWhenInUseUsageDescription" = "Sua localização é usada para encontrar estabelecimentos próximos.";
```

### 5. Graceful Degradation

Features MUST work (or degrade gracefully) when permissions are denied:

```swift
// Example: Map feature without location permission
struct MapView: View {
    @State private var locationAuthorized = false

    var body: some View {
        Map {
            if locationAuthorized {
                UserAnnotation()
            }
            // Always show points of interest regardless of location permission
            ForEach(pointsOfInterest) { poi in
                Marker(poi.name, coordinate: poi.coordinate)
            }
        }
    }
}
```

---

## Verification Checklist for GSD Agents

When a plan involves features that require permissions, agents MUST verify:

| Check | What to Verify |
|-------|---------------|
| Info.plist key exists | The `NS*UsageDescription` key is present for every permission the feature uses |
| Usage string is meaningful | Not empty, not placeholder — explains WHY the app needs access |
| Usage string is localized | Present in String Catalog or InfoPlist.strings for all supported locales |
| Runtime check before use | Code checks authorization status before accessing protected resource |
| Denial is handled | UI provides clear explanation and Settings link when permission is denied |
| Restricted is handled | UI handles `.restricted` state (user cannot change it) |
| Minimum access requested | App requests the least privileged level needed |
| No launch-time requests | Permissions are requested at moment of need, not in `init()` or `didFinishLaunching` |
| Capability is enabled | Required capabilities (e.g., Push Notifications, HealthKit, Background Modes) are enabled in Signing & Capabilities |

---

## Anti-Patterns

| Anti-Pattern | Problem | Correct Approach |
|-------------|---------|-----------------|
| Requesting all permissions on launch | User denies everything (no context) | Ask at moment of need |
| Missing Info.plist key | App crashes with `NSInternalInconsistencyException` | Always add usage description before shipping |
| Empty usage description string | App Store rejection | Write clear, specific reason |
| Not handling `.denied` state | Feature silently fails, user confused | Show explanation + Settings link |
| Using `PHPhotoLibrary` when `PhotosPicker` suffices | Unnecessary permission prompt | Use `PhotosPicker` for photo selection |
| Requesting `.always` location when `.whenInUse` is enough | User distrust, App Store scrutiny | Request minimum access level |
| Hardcoded English usage descriptions | Non-English users see English in system dialog | Localize via InfoPlist.xcstrings or InfoPlist.strings |
| Not re-checking status after returning from Settings | User may have changed permission while in Settings | Check status in `.onAppear` or `scenePhase == .active` |
