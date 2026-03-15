<overview>
Task lifecycle, cancellation, priorities, TaskGroup, async let, detached tasks, and advanced patterns like timeouts. Read when managing concurrent operations, understanding cancellation, or choosing between async let and TaskGroup. Related: async-await.md (basics), actors.md (isolation).
</overview>

## Task Fundamentals

Tasks bridge sync and async contexts. They start executing immediately.

```swift
let task = Task {
    try await fetchData()
}
// Later
let result = try await task.value
task.cancel()
```

## Cancellation

Cancellation is cooperative — tasks must check and respond.

```swift
func processItems(_ items: [Item]) async throws {
    for item in items {
        try Task.checkCancellation()  // Throws CancellationError
        await process(item)
    }
}

// Or check boolean for custom handling
func search(query: String) async -> [Result] {
    guard !Task.isCancelled else { return [] }
    // ... expensive work
}
```

**Check at:** Before expensive work, after network calls, before processing results.

**Parent cancellation:** Canceling parent automatically notifies children, but children must still check.

## SwiftUI Integration

```swift
.task {
    await loadData()  // Auto-cancels on view disappear
}

.task(id: selectedCategory) {
    await loadItems(for: selectedCategory)  // Restarts on change
}
```

## async let vs TaskGroup

| Feature | async let | TaskGroup |
|---------|-----------|-----------|
| Task count | Fixed at compile time | Dynamic at runtime |
| Best for | 2-5 known tasks | Loop-based parallel work |
| Cancellation | Auto on scope exit | Manual `cancelAll()` |
| Error handling | Propagates on await | Check in iteration |

### async let

```swift
async let users = fetchUsers()
async let posts = fetchPosts()
let (u, p) = try await (users, posts)
```

### TaskGroup

```swift
let results = try await withThrowingTaskGroup(of: Image.self) { group in
    for url in imageURLs {
        group.addTask { try await downloadImage(url) }
    }

    var images: [Image] = []
    for try await image in group {
        images.append(image)
    }
    return images
}
```

### Discarding Task Group (Fire-and-Forget)

```swift
await withDiscardingTaskGroup { group in
    for item in items {
        group.addTask { await sendAnalytics(item) }
    }
}  // Automatically waits for all, discards results
```

## Task Priorities

| Priority | Use Case |
|----------|----------|
| `.userInitiated` / `.high` | Immediate user feedback |
| `.medium` | Default for detached tasks |
| `.utility` | Longer-running, non-urgent |
| `.low` / `.background` | Background processing |

Structured tasks inherit parent priority. System auto-elevates to prevent priority inversion.

## Detached Tasks (Use as Last Resort)

Don't inherit priority, task-local values, or cancellation state:

```swift
Task.detached(priority: .background) {
    await cleanupOldData()
}
```

**When:** Independent background work, no parent connection needed.
**Prefer:** `@concurrent` in Swift 6.2 for offloading.

## Timeout Pattern

```swift
func fetchWithTimeout<T>(_ operation: @escaping () async throws -> T,
                          timeout: Duration) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        group.addTask { try await operation() }
        group.addTask {
            try await Task.sleep(for: timeout)
            throw TimeoutError()
        }

        let result = try await group.next()!
        group.cancelAll()
        return result
    }
}
```

## Task.sleep vs Task.yield

| | `Task.sleep(for:)` | `Task.yield()` |
|--|---------------------|----------------|
| Duration | Fixed | Instant |
| Purpose | Wait/delay | Let other tasks run |
| Cancellation | Throws CancellationError | Does not throw |

## Anti-Patterns

| Pattern | Problem | Fix |
|---------|---------|-----|
| `Task.detached` for everything | Loses context, priority, cancellation | Use structured concurrency or `@concurrent` |
| Not storing Task reference | Can't cancel | Store in `@State` or property |
| `DispatchQueue.main.async` | GCD mixing | Use `@MainActor` |
| Ignoring cancellation | Wasted CPU | Check `Task.isCancelled` regularly |
