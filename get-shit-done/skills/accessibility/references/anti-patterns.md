<overview>
Common accessibility mistakes with quick fixes, plus Assistive Access (iOS 17+) implementation. Read when fixing accessibility issues or implementing cognitive accessibility. Related: voiceover.md (correct patterns), enforcement.md (mandatory checklist).
</overview>

## Common Mistakes & Fixes

### Trait Names in Labels

```swift
// WRONG — VoiceOver reads "Close button, button"
.accessibilityLabel("Close button")

// CORRECT
.accessibilityLabel("Close")
```

### Hidden Interactive Elements

```swift
// WRONG — interactive element hidden from assistive tech
Button("Delete") { }
    .accessibilityHidden(true)

// CORRECT — only hide decorative elements
Image("divider")
    .accessibilityHidden(true)
```

### Fixed Font Sizes

```swift
// WRONG — does not scale
Text("Title").font(.system(size: 24))

// CORRECT — scales with Dynamic Type
Text("Title").font(.title)
```

### onTapGesture for Interactive Elements

```swift
// WRONG — invisible to VoiceOver and Voice Control
Text("Option A")
    .onTapGesture { select() }

// CORRECT — semantic control
Button("Option A") { select() }
```

### Ephemeral Messages (Toasts)

```swift
// WRONG — toast disappears before VoiceOver reads it
showToast("Saved!")

// CORRECT — announce to VoiceOver
AccessibilityNotification.Announcement("Saved!").post()
// AND use a persistent banner or inline message
```

### Many Swipes Per Cell

```swift
// WRONG — every element in a row is a separate VoiceOver stop
HStack {
    AsyncImage(url: imageURL)
    VStack { Text(title); Text(subtitle) }
}

// CORRECT — group into single element
HStack {
    AsyncImage(url: imageURL)
    VStack { Text(title); Text(subtitle) }
}
.accessibilityElement(children: .combine)
```

### Hardcoded Colors Without Dark Mode

```swift
// WRONG
Text("Error").foregroundColor(Color(red: 0.8, green: 0, blue: 0))

// CORRECT — semantic colors adapt to color scheme + high contrast
Text("Error").foregroundStyle(.red)
```

## Summary Table

| Anti-Pattern | Fix |
|---|---|
| Trait names in labels | Say "Close", not "Close button" |
| `.accessibilityHidden(true)` on interactive | Only on decorative elements |
| Fixed font sizes | Use text styles (`.body`, `.title`) |
| `onTapGesture` for buttons | Use `Button` |
| Ephemeral toasts | Announce + persistent message |
| Many swipes per cell | `.accessibilityElement(children: .combine)` |
| Hardcoded colors | Semantic colors (`.primary`, `.label`) |
| Hints everywhere | Only when action isn't obvious |
| Scaling toolbar icons | Use Large Content Viewer instead |

## Assistive Access (iOS 17+)

Specialized mode for cognitive disabilities. Streamlined interface with large controls and reduced complexity.

### Info.plist

```xml
<key>UISupportsAssistiveAccess</key>
<true/>

<!-- Optional: full screen display -->
<key>UISupportsFullScreenInAssistiveAccess</key>
<true/>
```

### AssistiveAccess Scene (SwiftUI)

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }

        AssistiveAccess {
            AssistiveAccessContentView()
        }
    }
}
```

### Runtime Detection

```swift
@Environment(\.accessibilityAssistiveAccessEnabled) var assistiveAccessEnabled

if assistiveAccessEnabled {
    SimplifiedView()
} else {
    FullView()
}
```

### Navigation Icons

```swift
NavigationStack {
    MyView()
        .navigationTitle("My Feature")
        .assistiveAccessNavigationIcon(systemImage: "star.fill")
}
```

### Design Principles

1. **Distill to core functionality** — remove distractions, secondary features
2. **Clear, prominent controls** — large buttons, ample spacing
3. **Multiple representations** — text + icons + visual alternatives
4. **Intuitive navigation** — step-by-step pathways, clear back buttons
5. **Safe interactions** — no irreversible actions without confirmation

### Testing

```swift
#Preview(traits: .assistiveAccess) {
    AssistiveAccessContentView()
}
```

Enable in Settings > Accessibility > Assistive Access. Verify app appears in "Optimized Apps" list.
