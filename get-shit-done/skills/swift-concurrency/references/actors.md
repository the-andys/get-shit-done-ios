<overview>
Actors for thread-safe shared state: @MainActor, custom actors, reentrancy traps, Mutex (iOS 18+), isolation boundaries, isolated parameters, and custom executors. Read when protecting mutable state or designing isolation architecture. Related: sendable.md (type safety), swift-6-2.md (isolated conformances).
</overview>

## Actor Basics

Actors protect mutable state with serialized access. Compiler enforces isolation — can't mutate from outside without `await`.

```swift
actor ImageCache {
    private var cache: [URL: UIImage] = [:]

    func image(for url: URL) -> UIImage? { cache[url] }
    func store(_ image: UIImage, for url: URL) { cache[url] = image }
}

// Usage — requires await from outside
let cached = await imageCache.image(for: url)
```

**Rules:**
- Actors cannot inherit (except NSObject for ObjC interop)
- Actors are implicitly Sendable
- Methods are isolated by default
- `nonisolated` opts out for immutable data access

## @MainActor

Ensures execution on main thread. Use for all UI-bound code.

```swift
@MainActor
@Observable
class ProfileViewModel {
    var profile: UserProfile?
    var isLoading = false

    func load() async {
        isLoading = true
        defer { isLoading = false }
        profile = try? await api.fetchProfile()
    }
}
```

**When to use @MainActor:**
- ViewModels that update UI state
- Any code that reads/writes UI properties
- Notification handlers that update UI

**When NOT to blindly apply @MainActor:**
- Pure computation (CPU-bound work)
- Network calls (they already use background threads)
- File I/O, image processing

## Reentrancy (Critical Trap)

During `await`, the actor is unlocked — other tasks can modify state.

```swift
actor BankAccount {
    var balance: Decimal = 100

    func withdraw(_ amount: Decimal) async throws {
        guard balance >= amount else { throw InsufficientFunds() }
        // ⚠️ Another task can modify balance here during await
        await logTransaction(amount)
        balance -= amount  // balance may have changed!
    }
}
```

**Solution:** Complete all state changes before suspension points:

```swift
func withdraw(_ amount: Decimal) async throws {
    guard balance >= amount else { throw InsufficientFunds() }
    balance -= amount  // Modify state BEFORE await
    await logTransaction(amount)
}
```

## Isolated Parameters

Reduce suspension points by passing isolated actor instances:

```swift
static func transfer(from source: isolated BankAccount, to dest: isolated BankAccount, amount: Decimal) {
    source.balance -= amount
    dest.balance += amount  // No await needed — both are isolated parameters
}
```

## Nonisolated

Opt out of actor isolation for immutable data:

```swift
actor UserManager {
    let id: UUID  // Immutable — safe from any context
    nonisolated var identifier: String { id.uuidString }
}
```

## Mutex (iOS 18+)

Synchronous locking without async overhead:

```swift
import Synchronization

let counter = Mutex<Int>(0)
counter.withLock { $0 += 1 }
```

**Use Mutex when:** Need synchronous access, low contention, legacy non-async APIs.
**Use Actor when:** High contention, complex state, need async methods.

## Custom Global Actors

```swift
@globalActor
actor DatabaseActor {
    static let shared = DatabaseActor()
    private init() { }
}

@DatabaseActor
class DatabaseManager {
    func query(_ sql: String) async throws -> [Row] { ... }
}
```

## Isolated Deinit (Swift 6.2+)

```swift
actor FileDownloader {
    var downloadTask: Task<Void, Error>?

    isolated deinit {
        downloadTask?.cancel()  // Safe: runs on actor's executor
    }
}
```

## #isolation Macro

Inherit caller's isolation for generic code:

```swift
func process(delegate: NonSendableDelegate, isolation: isolated (any Actor)? = #isolation) {
    Task { _ = isolation; delegate.doWork() }
}
```

## Custom Executors

Advanced — control how actor schedules work:

```swift
actor LegacyBridge {
    nonisolated var unownedExecutor: UnownedSerialExecutor {
        legacyQueue.asUnownedSerialExecutor()
    }
}
```

Use only when bridging to legacy DispatchQueue-based code or specific thread requirements.
