<overview>
Property wrapper selection, @Observable patterns, @State/@Binding/@Environment/@Bindable usage, decision flowchart, and common mistakes. Read when choosing how to manage state or debugging state-related issues. Related: view-composition.md (view structure), navigation.md (navigation state).
</overview>

## Property Wrapper Decision Flowchart

```
Who OWNS this data?
├── This view → Is it a value type?
│   ├── YES → @State private var
│   └── NO (class) → @State private var (with @Observable class)
├── Parent view → Does child need to MODIFY it?
│   ├── YES → @Binding var
│   └── NO → let (pass as value)
├── Injected @Observable (child needs bindings) → @Bindable var
├── Environment/system → @Environment(\.key)
├── Observable in environment → @Environment(MyType.self)
└── UserDefaults → @AppStorage("key")
```

## @Observable (iOS 17+)

```swift
@Observable
@MainActor
final class ProfileViewModel {
    var profile: UserProfile?
    var isLoading = false
    var errorMessage: String?

    func load() async { /* ... */ }
}
```

**Rules:**
- Mark `@MainActor` (unless using Swift 6.2 default actor isolation)
- Use `@State` to own in a view (NOT @StateObject)
- Use `@ObservationIgnored` for properties that conflict with other wrappers

```swift
@Observable @MainActor
final class SettingsModel {
    @ObservationIgnored @AppStorage("username") var username = ""
}
```

## @State (View-Owned Value State)

```swift
@State private var isPresented = false
@State private var searchText = ""
@State private var viewModel = ProfileViewModel()  // Also for @Observable
```

**Always `private`.** @State is owned by the view — external code should not set it directly.

**Don't pass values as @State** — it only accepts the initial value and ignores subsequent updates from parent.

## @Binding (Two-Way Reference)

```swift
struct ToggleRow: View {
    @Binding var isOn: Bool

    var body: some View {
        Toggle("Setting", isOn: $isOn)
    }
}

// Parent passes binding
ToggleRow(isOn: $settings.notificationsEnabled)
```

## @Bindable (For Injected @Observable)

When a child view receives an @Observable object and needs to create bindings:

```swift
struct ProfileEditor: View {
    @Bindable var viewModel: ProfileViewModel  // NOT @State — parent owns it

    var body: some View {
        TextField("Name", text: $viewModel.name)
    }
}
```

## @Environment

```swift
// System values
@Environment(\.dismiss) private var dismiss
@Environment(\.modelContext) private var modelContext
@Environment(\.colorScheme) private var colorScheme

// Custom @Observable
@Environment(ThemeManager.self) private var theme

// Custom key with @Entry (iOS 17+)
extension EnvironmentValues {
    @Entry var accentTheme: Theme = .default
}
```

## Injection Patterns

```swift
// Inject @Observable at root
@main
struct MyApp: App {
    @State private var store = TaskStore()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(store)
        }
    }
}

// Consume in any descendant
@Environment(TaskStore.self) private var store
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `@StateObject` with `@Observable` | Use `@State` instead |
| Public `@State` | Always `private` |
| `@State var x = parentValue` (expects updates) | Use `let x` or `@Binding` |
| `@Published` with `@Observable` | Not needed — all properties observed automatically |
| Nested `ObservableObject` not updating | Use `@Observable` (iOS 17+) or pass nested object directly |
| `@EnvironmentObject` in new code | Use `@Environment(Type.self)` with `@Observable` |

## onChange Patterns

```swift
// iOS 17+ (two params)
.onChange(of: searchText) { oldValue, newValue in
    performSearch(newValue)
}

// iOS 17+ (no params, when old value not needed)
.onChange(of: selectedTab) {
    resetScroll()
}
```

Never use the deprecated single-parameter `onChange(of:perform:)`.
