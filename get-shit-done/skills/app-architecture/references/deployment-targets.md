<overview>
Deployment target gating for iOS 17 (base), iOS 18 (conditional), and iOS 26 (conditional). Availability check patterns, xcconfig setup, and per-version API inventory. Read when using newer APIs, setting project targets, or reviewing availability checks. Related: framework-selection.md (which framework per domain).
</overview>

## iOS 17 (Base)

Minimum deployment target. Everything in the GSD skill set works on iOS 17.

**Available APIs:**
- SwiftData (`@Model`, `ModelContainer`, `ModelContext`)
- `@Observable` macro (replaces `ObservableObject`)
- `#Preview` macro (replaces `PreviewProvider`)
- `NavigationStack`, `NavigationSplitView`
- Swift Testing (`@Test`, `@Suite`, `#expect`)
- `contentMargins`, `scrollTargetBehavior`
- `TipKit` for contextual tips
- `SwiftUI.Inspector`

## iOS 18 (Conditional — Significant Leap)

Major capability jump. Use `if #available(iOS 18, *)` for these APIs.

| API | Purpose |
|-----|---------|
| `MeshGradient` | Complex multi-point gradient rendering |
| `@Previewable` macro | Use `@State` directly inside `#Preview` blocks |
| Enhanced `TabView` | `Tab` type with `.sidebarAdaptable` style |
| SwiftData history tracking | `ModelContext.fetchHistory` |
| Custom data stores | `DataStoreConfiguration` |
| `#Expression` macro | Type-safe predicates |
| `AccessoryWidgetGroup` | Compose small widgets on Watch / Lock Screen |
| `CustomAnimation` protocol | Reusable, type-safe custom animations |
| `TextRenderer` | Custom text rendering pipeline |
| SwiftData class inheritance | Base + subclass both with `@Model` |

```swift
// @Previewable eliminates wrapper views in previews
#Preview {
    @Previewable @State var isOn = false
    Toggle("Setting", isOn: $isOn)
}

// MeshGradient with fallback
if #available(iOS 18, *) {
    MeshGradient(/* ... */)
} else {
    LinearGradient(/* fallback */)
}
```

## iOS 26 (Conditional — Liquid Glass Era)

Use `if #available(iOS 26, *)` or `#if canImport(FoundationModels)`.

| API | Purpose |
|-----|---------|
| **Liquid Glass** (`.glassEffect`) | New translucent design language for UI surfaces |
| **FoundationModels** | On-device LLM inference with Apple Intelligence |
| `@Codable` macro | Synthesized from `@Model` |
| **Visual Intelligence** | Camera-based contextual search and actions |
| Enhanced widget interactivity | More interactive widget controls |
| **AlarmKit** | Alarm scheduling, Live Activities for alarms |
| **Charts 3D** | `Chart3D`, `SurfacePlot`, camera projection |

```swift
// Liquid Glass
if #available(iOS 26, *) {
    content.glassEffect(.regular)
} else {
    content.background(.ultraThinMaterial)
}

// FoundationModels with compile-time guard
#if canImport(FoundationModels)
import FoundationModels

@available(iOS 26, *)
func summarize(text: String) async throws -> String {
    let session = LanguageModelSession()
    let response = try await session.respond(to: "Summarize: \(text)")
    return response.content
}
#endif
```

## Availability Check Patterns

### Runtime Check

```swift
if #available(iOS 18, *) {
    // Use iOS 18 API
} else {
    // Fallback
}
```

### Compile-Time Check (Optional Frameworks)

```swift
#if canImport(FoundationModels)
// iOS 26+ only code
#endif
```

### Mark Entire Declarations

```swift
@available(iOS 18, *)
struct MeshBackgroundView: View { /* ... */ }
```

### View Modifier Pattern

```swift
extension View {
    @ViewBuilder
    func glassBackground() -> some View {
        if #available(iOS 26, *) {
            self.glassEffect(.regular)
        } else {
            self.background(.ultraThinMaterial)
        }
    }
}
```

## Deployment Target in Xcode

Always set in xcconfig, not in Xcode GUI:

```
// Base.xcconfig
IPHONEOS_DEPLOYMENT_TARGET = 17.0
```

This ensures consistency across team members and CI.

## Decision Tree

```
Need feature from iOS 18+ or 26+?
├── YES → Is there a fallback for older versions?
│   ├── YES → Use #available with fallback
│   └── NO → Is the feature critical to the app?
│       ├── YES → Consider raising deployment target
│       └── NO → Skip it or use workaround
└── NO → Use iOS 17 APIs
```
