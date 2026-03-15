<overview>
Type safety across concurrency boundaries: Sendable conformance for value and reference types, @unchecked Sendable risks, region-based isolation, sending keyword, closure captures, and global variable safety. Read when dealing with Sendable errors or designing types for concurrent use. Related: actors.md (isolation domains), migration.md (making code Sendable).
</overview>

## What Sendable Means

`Sendable` indicates a type is safe to share across isolation domains. The compiler verifies thread-safety at compile time.

**Three isolation domains:**
- Nonisolated (default)
- Actor-isolated (specific actor instance)
- Global actor-isolated (@MainActor, custom global actors)

## Value Types (Usually Automatic)

Non-public structs/enums with all-Sendable members are implicitly Sendable:

```swift
struct UserProfile {  // Implicitly Sendable
    let name: String
    let age: Int
}

enum Status: Sendable {  // Explicit for public types
    case active, inactive
}
```

Public types require explicit conformance — compiler can't verify internal details across modules.

## Reference Types (Strict Rules)

Classes must be `final` with only immutable, Sendable properties:

```swift
final class Configuration: Sendable {
    let apiURL: URL      // Immutable ✓
    let timeout: Double  // Immutable ✓
}
```

**Requirements for Sendable class:**
- `final` (no inheritance that could add mutable state)
- All stored properties are `let` (immutable)
- All stored properties are Sendable
- No superclass (except NSObject)

**Alternative:** Actor isolation makes classes Sendable automatically.

## @unchecked Sendable (Last Resort)

For types with manual thread-safety the compiler can't verify:

```swift
final class ThreadSafeCache: @unchecked Sendable {
    private let lock = NSLock()
    private var storage: [String: Data] = [:]

    func get(_ key: String) -> Data? {
        lock.withLock { storage[key] }
    }
}
```

**Risks:**
- No compile-time safety — data races won't be caught
- Must ensure ALL access paths use the lock
- Document the safety invariant

**Prefer:** Actor instead. Use @unchecked only for legacy code + file migration ticket.

## Region-Based Isolation

Non-Sendable types allowed in same scope if no mutation after transfer:

```swift
let article = Article()  // Non-Sendable
Task { print(article.title) }  // ✅ Same region, no further access
// print(article.title)  // ❌ Would be error — accessed after transfer
```

## The `sending` Keyword

Enforces ownership transfer for non-Sendable parameters:

```swift
func process(_ item: sending Article) async {
    // Caller can't use item after passing it
}
```

## @Sendable Closures

Closures crossing isolation boundaries must be `@Sendable`:

```swift
Task { @Sendable in
    // Only capture Sendable values
}
```

Capture rules:
- ✅ Value types (copied)
- ✅ Sendable reference types
- ❌ Non-Sendable classes (use actor or sending)
- ❌ Mutable local variables (copy first to let)

## Global Variables

Problem: accessible from any context — unsafe.

| Solution | When |
|----------|------|
| `@MainActor static let shared` | UI-related singletons |
| `final class: Sendable` (immutable) | Configuration |
| `nonisolated(unsafe) static var` | Last resort — you guarantee safety |

```swift
@MainActor
final class AppState {
    static let shared = AppState()  // Protected by @MainActor
    var currentUser: User?
}
```

## Data Races vs Race Conditions

| | Data Race | Race Condition |
|--|-----------|----------------|
| **What** | Multiple threads access shared mutable state | Timing-dependent behavior |
| **Swift prevents?** | Yes (Sendable + actors) | No (requires sequencing) |
| **Fix** | Sendable types, actors | Sequential async/await, state machines |
