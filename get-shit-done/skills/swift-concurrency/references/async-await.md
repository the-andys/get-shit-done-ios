<overview>
Foundation of Swift concurrency: async/await patterns, URLSession async APIs, structured concurrency with async let, and continuation bridging for legacy code. Read when writing new async code or converting callbacks to async. Related: tasks.md (TaskGroup, cancellation), actors.md (isolation).
</overview>

## Basic Patterns

```swift
// Declaration
func fetchData() async throws -> Data { ... }

// Calling
let data = try await fetchData()

// Execution is top-to-bottom — code after await only runs when awaited function returns
```

### Sequential Execution

```swift
let user = try await fetchUser(id: userID)
let posts = try await fetchPosts(for: user)  // Waits for user first
let profile = Profile(user: user, posts: posts)
```

### Parallel Execution with async let

```swift
func loadDashboard() async throws -> Dashboard {
    async let profile = fetchProfile()
    async let tasks = fetchTasks()
    async let notifications = fetchNotifications()

    return try await Dashboard(
        profile: profile,
        tasks: tasks,
        notifications: notifications
    )
}
```

`async let` starts immediately, runs concurrently. Automatically cancelled on scope exit if not awaited.

## URLSession Async Patterns

```swift
// Data
let (data, response) = try await URLSession.shared.data(for: request)

// Bytes (streaming)
let (bytes, response) = try await URLSession.shared.bytes(for: URLRequest(url: url))
for try await byte in bytes { /* process */ }

// Upload
let (data, response) = try await URLSession.shared.upload(for: request, from: bodyData)
```

## .task Modifier in SwiftUI

```swift
.task {
    await loadData()  // Auto-cancels on view disappear
}

.task(id: selectedCategory) {
    await loadItems(for: selectedCategory)  // Restarts when id changes
}
```

Do NOT wrap in `Task {}` inside `.task` — `.task` already creates a Task.

## Continuation Bridging

Convert callback-based APIs to async/await:

```swift
func fetchLegacy() async throws -> User {
    try await withCheckedThrowingContinuation { continuation in
        legacyFetch { result in
            continuation.resume(with: result)
        }
    }
}
```

**Rules:**
- Resume exactly ONCE — double resume is undefined behavior
- Use `withCheckedThrowingContinuation` (debug checks) over `withUnsafeContinuation`
- Handle all code paths in the callback

## Typed Throws (Swift 6)

```swift
func fetchData() async throws(NetworkError) -> Data {
    // Caller knows exactly which errors to handle
}
```

## Anti-Patterns

| Pattern | Problem | Fix |
|---------|---------|-----|
| `Task { }` inside `.task` | Unnecessary nesting | Use `.task` directly |
| Forgetting `try Task.checkCancellation()` | Wasted work | Check at natural breakpoints |
| `await` in a loop without cancellation check | Long-running uninterruptible work | Add `try Task.checkCancellation()` |
| Ignoring `CancellationError` in catch | Shows error UI for cancelled tasks | Catch separately, no-op |
