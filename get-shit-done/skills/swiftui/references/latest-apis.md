<overview>
Deprecated-to-modern API transitions organized by iOS version, plus iOS 26+ new features: FoundationModels, Charts 3D, toolbar features, and AttributedString styling. Read when using newer APIs, checking for deprecations, or exploring iOS 26+ capabilities. Related: animations.md (Liquid Glass), deployment-targets in app-architecture skill.
</overview>

## Deprecated → Modern API Map

### Always Use (Any iOS Version)

| Deprecated | Modern |
|-----------|--------|
| `.navigationBarTitle()` | `.navigationTitle()` |
| `.navigationBarItems()` | `.toolbar { ToolbarItem(...) }` |
| `.foregroundColor()` | `.foregroundStyle()` |
| `.cornerRadius()` | `.clipShape(.rect(cornerRadius:))` |
| `.animation(_:)` no value | `.animation(_:value:)` |
| `.actionSheet()` | `.confirmationDialog()` |
| `PreviewProvider` | `#Preview` |
| `.accessibility(label:)` | `.accessibilityLabel()` |
| `onTapGesture` for buttons | `Button` |

### iOS 16+

| Deprecated | Modern |
|-----------|--------|
| `NavigationView` | `NavigationStack` / `NavigationSplitView` |
| `.accentColor()` | `.tint()` |

### iOS 17+

| Deprecated | Modern |
|-----------|--------|
| `ObservableObject` + `@Published` | `@Observable` |
| `@StateObject` with Observable | `@State` |
| `@EnvironmentObject` | `@Environment(Type.self)` |
| `onChange(of:perform:)` one-param | `onChange(of:) { old, new in }` |
| `MagnificationGesture` | `MagnifyGesture` |
| `RotationGesture` | `RotateGesture` |
| `GeometryReader` (many uses) | `containerRelativeFrame()`, `visualEffect()`, `onGeometryChange()` |
| Manual `EnvironmentKey` | `@Entry` macro |

### iOS 18+

| New API | Purpose |
|---------|---------|
| `@Previewable` | `@State` directly in `#Preview` blocks |
| `Tab` type | Modern TabView with `.sidebarAdaptable` |

### iOS 26+

| New API | Purpose |
|---------|---------|
| `.glassEffect()` | Liquid Glass design |
| `GlassEffectContainer` | Grouped glass elements |
| `.buttonStyle(.glass)` | Glass button style |
| `Tab(role: .search)` | Search tab role |
| `tabBarMinimizeBehavior()` | Minimize tab bar |
| `ToolbarSpacer` | Fixed/flexible toolbar spacing |
| `searchToolbarBehavior(.minimize)` | Minimizable search |
| `sharedBackgroundVisibility()` | Per-item glass visibility |

## FoundationModels (iOS 26+)

On-device LLM for text generation, summarization, extraction:

```swift
#if canImport(FoundationModels)
import FoundationModels

@available(iOS 26, *)
func summarize(text: String) async throws -> String {
    // Check availability first
    guard case .available = SystemLanguageModel.default.availability else {
        throw ModelError.unavailable
    }

    let session = LanguageModelSession(instructions: "Summarize concisely.")
    let response = try await session.respond(to: text)
    return response.content
}
#endif
```

### Guided Generation

```swift
@Generable(description: "Recipe summary")
struct RecipeSummary {
    var title: String
    @Guide(description: "Cooking time in minutes", .range(1...480))
    var cookingTime: Int
    var difficulty: String
}

let recipe = try await session.respond(to: "Pasta carbonara", generating: RecipeSummary.self)
```

### Availability States

| State | Action |
|-------|--------|
| `.available` | Show feature |
| `.unavailable(.deviceNotEligible)` | Hide feature entirely |
| `.unavailable(.appleIntelligenceNotEnabled)` | Guide to enable |
| `.unavailable(.modelNotReady)` | Show "preparing" state |

## Charts 3D (iOS 26+)

```swift
Chart3D {
    SurfacePlot(x: "X", y: "Y", z: "Z") { x, y in
        sin(x) * cos(y)
    }
}
.chart3DPose($chartPose)  // Interactive rotation
```

Predefined poses: `.default`, `.front`, `.back`, `.top`, `.bottom`, `.left`, `.right`.

## New Toolbar Features (iOS 26+)

```swift
// Customizable toolbar with IDs
.toolbar(id: "main") {
    ToolbarItem(id: "share") { ShareButton() }
    ToolbarSpacer(.flexible)
    ToolbarItem(id: "more") { MoreButton() }
}

// Minimizable search
.searchable(text: $searchText)
.searchToolbarBehavior(.minimize)

// Matched transition from toolbar to sheet
.toolbar {
    ToolbarItem {
        Button("Show") { show = true }
            .matchedTransitionSource(id: "item", in: namespace)
    }
}
.sheet(isPresented: $show) {
    DetailView()
        .navigationTransition(.zoom(sourceID: "item", in: namespace))
}
```
