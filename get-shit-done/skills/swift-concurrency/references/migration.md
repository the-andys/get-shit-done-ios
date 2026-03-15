<overview>
Swift 5→6 migration strategy: project settings, strict concurrency levels, six migration habits, step-by-step process, closure conversion, @preconcurrency, and tooling. Read when migrating to Swift 6 or enabling strict concurrency. Related: swift-6-2.md (6.2-specific changes), sendable.md (making types Sendable).
</overview>

## Six Migration Habits

1. **Don't panic** — iterate daily for 30 min. Large projects take weeks.
2. **Sendable by default** for new code. Design for concurrency upfront.
3. **Use Swift 6 for new projects/packages** to reduce debt.
4. **Resist refactoring** — focus solely on concurrency changes. Don't improve code structure at the same time.
5. **Minimal changes** — small PRs, one class/module at a time.
6. **Don't @MainActor everything** — consider if main-actor isolation is actually correct for each type.

## Project Settings Matrix

| Setting | Location | Impact |
|---------|----------|--------|
| Swift language mode | `SWIFT_VERSION` / swift-tools-version | Swift 6 enforces stricter defaults |
| Strict concurrency | `SWIFT_STRICT_CONCURRENCY` | Controls Sendable + isolation enforcement |
| Default actor isolation | `SWIFT_DEFAULT_ACTOR_ISOLATION` | Changes default isolation (Swift 6.2) |
| NonisolatedNonsendingByDefault | Upcoming feature flag | Changes nonisolated async execution |
| Approachable Concurrency | Build setting | Bundles multiple migration features |

## Step-by-Step Process

1. **Start with isolated piece** — minimal dependencies, low blast radius
2. **Update dependencies** to latest versions (they may already be Sendable-ready)
3. **Add async alternatives** for callback-based APIs (deprecate old ones)
4. **Consider default actor isolation** — `@MainActor` default for app targets (Swift 6.2+)
5. **Enable strict concurrency** — Minimal → Targeted → Complete
6. **Add Sendable conformances** even before compiler complains
7. **Enable Approachable Concurrency** (after feature-by-feature adoption)
8. **Enable upcoming features** individually
9. **Switch to Swift 6** language mode

## Strict Concurrency Levels

| Level | Behavior |
|-------|----------|
| Minimal | Only explicit Sendable checks |
| Targeted | Warns on closures and protocols |
| Complete | Full Sendable enforcement (same as Swift 6) |

```
// Build Settings
SWIFT_STRICT_CONCURRENCY = complete
```

## Closure to Async/Await

### Xcode Refactoring Options

1. **Add Async Wrapper** (safest first step) — adds async overload, keeps original
2. **Add Async Alternative** — rewrites alongside original
3. **Convert Function to Async** — replaces entirely

### Manual Conversion

```swift
// Before
func fetchUser(completion: @escaping (Result<User, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, response, error in
        if let error { completion(.failure(error)); return }
        guard let data else { completion(.failure(NetworkError.noData)); return }
        do {
            let user = try JSONDecoder().decode(User.self, from: data)
            completion(.success(user))
        } catch {
            completion(.failure(error))
        }
    }.resume()
}

// After
func fetchUser() async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}
```

## @preconcurrency

Suppress Sendable warnings from modules you don't control:

```swift
@preconcurrency import ThirdPartyModule
```

Use when: external dependency hasn't adopted Sendable yet. Remove when dependency updates.

## Migration Tooling (Swift 6.2+)

Semi-automatic migration for upcoming features:

```
Build Settings → Upcoming Feature → set to "Migrate"
```

Warnings appear with Apply buttons. Or via command line:

```bash
swift package migrate --to-feature ExistentialAny
```

## Common Migration Patterns

### Global Mutable State

```swift
// Before
static var shared = Manager()

// After
@MainActor static let shared = Manager()
```

### Non-Sendable Callback

```swift
// Before — warning: closure captures non-Sendable
Task { callback(result) }

// After — use continuation
try await withCheckedThrowingContinuation { cont in
    legacyAPI { result in cont.resume(with: result) }
}
```

### Protocol Conformance

```swift
// Before — warning: actor-isolated conformance
extension ViewModel: SomeProtocol { ... }

// After (Swift 6.2) — isolated conformance
extension ViewModel: @MainActor SomeProtocol { ... }
```

## Validation Loop

```
Build → Fix one warning → Rebuild → Repeat
```

Never batch fixes. One change at a time, rebuild after each.
