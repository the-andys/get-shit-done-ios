<overview>
Code-level performance smells in SwiftUI: invalidation storms, unstable identity, heavy body computation, expensive formatters, broad observable dependencies, and conditional view swapping. Most performance issues are visible in code without profiling. Related: remediation.md (fixes), instruments.md (profiling when code review isn't enough).
</overview>

## Invalidation Storms

**Symptom:** View recomputes constantly even when visible state hasn't changed.

**Causes:**
- Observable object with many properties — change to ANY property invalidates ALL subscribers
- State changes triggering cascading updates
- Hot paths (scroll handlers, timers) assigning state unconditionally

**Detection:**
```swift
var body: some View {
    let _ = Self._printChanges()  // Shows which property triggered recompute
    content
}
```

## Unstable Identity

**Symptom:** List items flicker, animations break, scroll position resets.

```swift
// WRONG — new identity every render
ForEach(items) { item in
    ItemRow(item: item).id(UUID())
}

// WRONG — unstable with id: \.self on mutable types
ForEach(items, id: \.self) { item in
    ItemRow(item: item)
}

// CORRECT — stable identifier from data
ForEach(items) { item in  // Uses item.id automatically
    ItemRow(item: item)
}
```

## Heavy Work in body

**Smells to catch:**

```swift
// WRONG — formatter created every recompute
var body: some View {
    let formatter = DateFormatter()  // 🔥 Created every time
    Text(formatter.string(from: date))
}

// WRONG — sorting in body
var body: some View {
    ForEach(items.sorted(by: { $0.date > $1.date })) { item in  // 🔥 Sorts every recompute
        ItemRow(item: item)
    }
}

// WRONG — filtering in body
var body: some View {
    let filtered = items.filter { $0.isActive }  // 🔥 Filters every recompute
    ForEach(filtered) { ... }
}
```

## Expensive Formatters

```swift
// CORRECT — cached at type level
private static let dateFormatter: DateFormatter = {
    let f = DateFormatter()
    f.dateStyle = .medium
    return f
}()

// CORRECT — FormatStyle (reused automatically)
Text(date, format: .dateTime.month().day())
```

## Broad Observable Dependencies

```swift
// WRONG — view reads entire store, invalidated by any change
struct ItemRow: View {
    @Environment(Store.self) private var store
    var body: some View {
        Text(store.items[index].name)  // Invalidated when ANY store property changes
    }
}

// BETTER — pass specific data
struct ItemRow: View {
    let name: String
    var body: some View { Text(name) }
}
```

### Per-Item @Observable

For lists where each item has mutable state:

```swift
@Observable class ItemViewModel {
    var isFavorite = false
}

struct ItemRow: View {
    let viewModel: ItemViewModel  // Only invalidated when THIS item changes
    var body: some View { /* ... */ }
}
```

## Top-Level Conditional View Swapping

```swift
// SMELL — entire view hierarchy replaced
var body: some View {
    if showDetail {
        DetailView()   // Different view tree
    } else {
        ListView()     // Different view tree
    }
}

// BETTER — stable base, local conditions
var body: some View {
    content
        .overlay { if showDetail { DetailOverlay() } }
}
```

## Hot Path Gate

```swift
// WRONG — assigns every scroll event
.onReceive(scrollPublisher) { value in
    scrollOffset = value  // Triggers recompute every frame
}

// CORRECT — gate by threshold
.onReceive(scrollPublisher) { value in
    if abs(value - scrollOffset) > 5 {
        scrollOffset = value
    }
}
```

## ~30-Line Rule

View `body` over ~30 lines is a code smell. Extract subviews as:
- Computed properties (simple, no state)
- Separate View structs (complex, has state — SwiftUI can skip body when inputs unchanged)

## Checklist

- [ ] No object creation in body
- [ ] Formatters cached (static or FormatStyle)
- [ ] No sorting/filtering in body
- [ ] Stable ForEach identity
- [ ] Specific data passed, not whole objects
- [ ] Hot paths gated by threshold
- [ ] Body under ~30 lines
