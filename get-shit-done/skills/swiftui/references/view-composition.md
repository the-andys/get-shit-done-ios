<overview>
View extraction rules, @ViewBuilder patterns, container views, property ordering, body splitting, and the ~30-line rule. Read when structuring views, reducing body size, or improving code organization. Related: state-management.md (property wrapper placement), performance.md (composition for performance).
</overview>

## Property Ordering Convention

```swift
struct ProfileView: View {
    // 1. Environment
    @Environment(\.dismiss) private var dismiss
    @Environment(Theme.self) private var theme

    // 2. let/public properties
    let user: UserProfile

    // 3. @State
    @State private var isEditing = false

    // 4. Computed vars (non-view)
    var displayName: String { user.name ?? "Anonymous" }

    // 5. init (if custom)

    // 6. body
    var body: some View { /* ... */ }

    // 7. Computed view builders
    private var header: some View { /* ... */ }

    // 8. Helper functions
    private func save() async { /* ... */ }
}
```

## The ~30-Line Rule

If `body` exceeds ~30 lines, extract subviews.

```swift
// WRONG: 100+ line body
var body: some View {
    VStack {
        // ... 50 lines header ...
        // ... 30 lines content ...
        // ... 20 lines footer ...
    }
}

// CORRECT: Extracted
var body: some View {
    VStack {
        headerSection
        contentSection
        footerSection
    }
}
```

## Separate Structs > Computed Properties

For complex subviews, prefer separate structs over computed properties. SwiftUI can skip their body when inputs don't change.

```swift
// BETTER: Separate struct — body skipped when unchanged
struct ComplexSection: View {
    let items: [Item]
    var body: some View { /* expensive rendering */ }
}

// OK for simple extraction
private var simpleLabel: some View {
    Text("Hello").font(.headline)
}
```

## @ViewBuilder Functions

Use only for small, simple sections:

```swift
@ViewBuilder
private func row(for item: Item) -> some View {
    if item.isSpecial {
        SpecialRow(item: item)
    } else {
        StandardRow(item: item)
    }
}
```

## Container Pattern

```swift
struct CardContainer<Content: View>: View {
    @ViewBuilder let content: Content

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            content
        }
        .padding()
        .background(.regularMaterial)
        .clipShape(.rect(cornerRadius: 12))
    }
}

// Usage
CardContainer {
    Text("Title").font(.headline)
    Text("Description")
}
```

Use `@ViewBuilder let content: Content` — not closure-based. SwiftUI can compare and skip.

## Modifiers Over Conditionals

Prefer modifiers to maintain view identity (better for animations):

```swift
// GOOD: Same view, different modifier
Text("Hello")
    .opacity(isVisible ? 1 : 0)

// LESS GOOD: Different views (breaks identity)
if isVisible {
    Text("Hello")
}
```

## Top-Level Stability

Avoid root-level if/else that swaps entire view hierarchies:

```swift
// AVOID
var body: some View {
    if isLoggedIn {
        MainTabView()
    } else {
        LoginView()
    }
}

// BETTER: Stable base with internal routing
var body: some View {
    Group {
        switch appState {
        case .loggedIn: MainTabView()
        case .loggedOut: LoginView()
        }
    }
}
```

## Compositing Group for Clipping

Prevents antialiasing fringes at rounded corners:

```swift
Color.red
    .overlay(.white, in: shape)
    .compositingGroup()
    .clipShape(shape)
```

## ViewModifier for Reusable Styling

```swift
struct CardModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(.regularMaterial)
            .clipShape(.rect(cornerRadius: 12))
            .shadow(radius: 2)
    }
}

extension View {
    func cardStyle() -> some View { modifier(CardModifier()) }
}
```

## Custom ButtonStyle

```swift
struct ScaleButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed ? 0.95 : 1)
            .animation(.spring, value: configuration.isPressed)
    }
}
```

## ZStack vs overlay/background

| | ZStack | overlay/background |
|--|--------|-------------------|
| Purpose | Peer composition | Decoration |
| Size | Largest child | Matches parent |
| Use for | Stacked content | Badges, borders |
