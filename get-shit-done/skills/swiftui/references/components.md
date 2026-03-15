<overview>
Component cookbook: TabView, Form, Grid, ScrollView, theming system, controls, haptics, and loading states. Read when implementing specific UI components or building a design system. Related: navigation.md (TabView navigation), view-composition.md (extraction patterns).
</overview>

## TabView

```swift
TabView(selection: $selectedTab) {
    Tab("Home", systemImage: "house", value: .home) {
        HomeView()
    }
    Tab("Search", systemImage: "magnifyingglass", value: .search) {
        SearchView()
    }
    Tab("Profile", systemImage: "person", value: .profile) {
        ProfileView()
    }
}
```

iOS 18+: Use `Tab` type with `.sidebarAdaptable` style.
iOS 26+: `Tab(role: .search)`, `tabBarMinimizeBehavior()`, `tabViewBottomAccessory`.

## Form

```swift
Form {
    Section("Account") {
        TextField("Name", text: $name)
        TextField("Email", text: $email)
            .textContentType(.emailAddress)
            .keyboardType(.emailAddress)
    }

    Section("Preferences") {
        Toggle("Notifications", isOn: $notificationsEnabled)
        Picker("Theme", selection: $theme) {
            ForEach(Theme.allCases) { Text($0.rawValue).tag($0) }
        }
    }

    Section {
        Button("Delete Account", role: .destructive) { showDeleteAlert = true }
    }
}
```

## Grid (Lazy)

```swift
let columns = [GridItem(.adaptive(minimum: 150, maximum: 200))]

ScrollView {
    LazyVGrid(columns: columns, spacing: 16) {
        ForEach(items) { item in
            ItemCard(item: item)
        }
    }
    .padding()
}
```

### Fixed Columns

```swift
let columns = [
    GridItem(.flexible()),
    GridItem(.flexible()),
    GridItem(.flexible())
]
```

## ScrollView

```swift
ScrollView {
    LazyVStack(spacing: 12) {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
    .padding(.horizontal)
}
.refreshable { await loadItems() }
```

### Scroll Position (iOS 17+)

```swift
@State private var scrollPosition: Item.ID?

ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
    .scrollTargetLayout()
}
.scrollPosition(id: $scrollPosition)
```

## Theming System

```swift
@Observable @MainActor
final class Theme {
    var tintColor: Color = .blue
    var primaryBackground: Color = .white
    var labelColor: Color = .primary
}

// Inject at root
ContentView().environment(theme)

// Use in views
@Environment(Theme.self) private var theme
Text("Title").foregroundStyle(theme.labelColor)
```

Use semantic color names (primaryBackground, label, tint) — not raw Color values.

## Controls

### Toggle Styles

```swift
Toggle("WiFi", isOn: $wifiEnabled)
    .toggleStyle(.switch)  // Default

Toggle("Airplane", isOn: $airplaneMode)
    .toggleStyle(.button)  // Tappable button
```

### Stepper

```swift
Stepper("Quantity: \(quantity)", value: $quantity, in: 1...99)
```

### DatePicker

```swift
DatePicker("Date", selection: $date, displayedComponents: [.date])
    .datePickerStyle(.graphical)
```

### Slider

```swift
Slider(value: $volume, in: 0...100, step: 1) {
    Text("Volume")
} minimumValueLabel: {
    Image(systemName: "speaker")
} maximumValueLabel: {
    Image(systemName: "speaker.wave.3")
}
```

## Haptics

```swift
// Simple feedback
Button("Tap") {
    UIImpactFeedbackGenerator(style: .medium).impactOccurred()
    performAction()
}

// Notification feedback
UINotificationFeedbackGenerator().notificationOccurred(.success)

// Selection feedback
UISelectionFeedbackGenerator().selectionChanged()

// SensoryFeedback (iOS 17+)
Button("Tap") { }
    .sensoryFeedback(.impact(weight: .medium), trigger: tapCount)
```

## Loading States

```swift
enum LoadingState<Value> {
    case idle, loading, loaded(Value), failed(Error)
}

// Usage in view
switch state {
case .idle: Color.clear
case .loading: ProgressView()
case .loaded(let value): ContentView(value: value)
case .failed(let error):
    ContentUnavailableView {
        Label("Error", systemImage: "exclamationmark.triangle")
    } description: {
        Text(error.localizedDescription)
    } actions: {
        Button("Retry") { Task { await load() } }
    }
}
```

## Skeleton Loading

```swift
content
    .redacted(reason: isLoading ? .placeholder : [])
```
