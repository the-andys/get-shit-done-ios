<overview>
SwiftUI performance patterns: lazy stacks, identity stability, body purity, formatter caching, invalidation storms, and debugging tools. Read when diagnosing slow rendering or optimizing list performance. Related: view-composition.md (extraction for performance), state-management.md (dependency minimization).
</overview>

## Core Principle: Minimize Dependencies

SwiftUI recomputes `body` when any dependency changes. Fewer dependencies = fewer recomputations.

### Pass Only What's Needed

```swift
// AVOID: Passing large object
struct ItemRow: View {
    let store: Store  // Reads every property change

    var body: some View { Text(store.items[index].name) }
}

// BETTER: Pass specific data
struct ItemRow: View {
    let name: String  // Only recomputes if name changes

    var body: some View { Text(name) }
}
```

### Per-Item @Observable for Lists

```swift
@Observable
class ItemViewModel {
    var isFavorite: Bool = false
}

struct ItemRow: View {
    let viewModel: ItemViewModel  // Granular dependency

    var body: some View {
        Button(viewModel.isFavorite ? "★" : "☆") {
            viewModel.isFavorite.toggle()
        }
    }
}
```

## Check Before Assigning (Hot Paths)

SwiftUI doesn't compare before assigning. Gate hot paths:

```swift
// Scroll handlers, timers, publishers
.onReceive(publisher) { value in
    if currentValue != value {
        currentValue = value  // Only assign if changed
    }
}
```

## Lazy Loading

```swift
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemRow(item: item)  // Created on demand
        }
    }
}
```

Use `LazyVStack` / `LazyHStack` for long lists. Regular `VStack` creates all views upfront.

iOS 26+: Nested lazy stacks defer loading automatically.

## Identity Stability

Unstable identity causes unnecessary view recreation:

```swift
// AVOID: Unstable ID (changes every render)
ForEach(items) { item in
    ItemRow(item: item)
        .id(UUID())  // New identity every time!
}

// CORRECT: Stable ID from data
ForEach(items) { item in
    ItemRow(item: item)  // Uses item.id automatically
}
```

## Body Purity

`body` should be a pure function of state. Never:
- Create objects in body (formatters, date formatters, timers)
- Perform heavy computation
- Make network calls
- Modify state (triggers infinite loops)

```swift
// AVOID: Object creation in body
var body: some View {
    let formatter = DateFormatter()  // Created every recompute!
    Text(formatter.string(from: date))
}

// CORRECT: Cache outside body
private static let dateFormatter: DateFormatter = {
    let f = DateFormatter()
    f.dateStyle = .medium
    return f
}()
```

## Debugging Updates

```swift
var body: some View {
    let _ = Self._printChanges()  // Console: which property changed
    // or
    let _ = Self._logChanges()    // Unified logging
    content
}
```

Remove after debugging — these have performance cost.

## Equatable Views

For custom equality when SwiftUI's default diff is too broad:

```swift
struct ExpensiveView: View, Equatable {
    let title: String
    let metadata: LargeObject

    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.title == rhs.title  // Only recompute if title changes
    }

    var body: some View { /* expensive rendering */ }
}
```

## Off-Main-Thread Closures

These run off the main thread — don't capture @MainActor state:

```swift
// Shape.path(), visualEffect, Layout, onGeometryChange
.visualEffect { [pulse] content, geometry in
    content.blur(radius: pulse ? 5 : 0)  // Capture value, not binding
}
```

## Performance Checklist

- [ ] Body under ~30 lines (extracted subviews)
- [ ] No object creation in body
- [ ] Large lists use LazyVStack/LazyHStack
- [ ] ForEach items have stable identity
- [ ] Pass specific values, not entire objects
- [ ] Hot paths check before assigning
- [ ] Formatters cached as static properties
