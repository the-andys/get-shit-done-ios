<overview>
Swift 6.2 "Approachable Concurrency" — the biggest behavior change since Swift Concurrency was introduced. Default actor isolation, @concurrent attribute, isolated conformances, and migration settings. Based on Apple's Swift-Concurrency-Updates documentation (AUTHORITATIVE). Read when targeting iOS 26+ or preparing for Swift 6.2. Related: migration.md (full migration strategy), actors.md (isolation details).
</overview>

## Philosophy Change

**Swift 6.2 principle:** Stay single-threaded by default until you choose to introduce concurrency. Natural code is data-race-free by default.

## Three Key Changes

### 1. Async Functions Inherit Caller's Isolation

**Old behavior (Swift 6.0/6.1):** Nonisolated async functions always offload to background thread.

**New behavior (Swift 6.2):** Nonisolated async functions inherit caller's isolation — if called from `@MainActor`, they stay on `@MainActor`.

```swift
// In Swift 6.2, this stays on @MainActor when called from @MainActor
func processData() async -> ProcessedData {
    // Runs on caller's isolation domain
    return transform(rawData)
}
```

**Benefits:**
- Eliminates data races — values never accidentally leave an actor
- Client code doesn't worry about non-Sendable state
- Most app code "just works" without explicit isolation

**Enable:** `.enableUpcomingFeature("NonisolatedNonsendingByDefault")`

### 2. Isolated Conformances

Protocol conformance on `@MainActor` types is now safe when marked:

```swift
extension StickerModel: @MainActor Exportable {
    func export() {
        photoProcessor.exportAsPNG()  // Safe: @MainActor conformance
    }
}
```

Compiler ensures the conformance is only used on the main actor.

**When useful:** Observable ViewModels conforming to protocols that require method implementations.

### 3. Default Actor Isolation

Set `@MainActor` as default isolation for an entire target/module:

```
// Build Settings
SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor
```

Or in Package.swift:
```swift
.swiftSettings([.defaultIsolation(MainActor.self)])
```

**Why:** Most app code runs on main thread. This reduces false warnings and avoids "concurrency rabbit holes."

**When to use:** App targets (not library targets).

## @concurrent — Explicit Offloading

When you specifically want background execution, use `@concurrent`:

```swift
nonisolated struct PhotoProcessor {
    @concurrent
    func process(data: Data) async -> ProcessedPhoto? {
        // Guaranteed to run on concurrent thread pool
        let processed = applyFilters(data)
        return processed
    }
}
```

**Requirements for @concurrent:**
- `nonisolated` context
- `@concurrent` attribute
- `async` keyword
- `await` at call sites

**Replaces:** Most uses of `Task.detached` for CPU-bound work.

## Protected Global State

```swift
@MainActor
final class StickerLibrary {
    static let shared: StickerLibrary = .init()
    var stickers: [Sticker] = []
}
```

`@MainActor` on the class + `static let` protects from concurrent access.

## Migration Path to Swift 6.2

1. **Start with default actor isolation** — set @MainActor as project default
2. **Enable Approachable Concurrency** — bundles nonisolated async inheritance + other features
3. **Mark CPU-bound work as @concurrent** — explicit background offloading
4. **Use isolated conformances** — clean up protocol conformance warnings
5. **Remove unnecessary @MainActor annotations** — default handles most cases

## Where Swift 6.2 Differs from 6.0/6.1

| Aspect | Swift 6.0/6.1 | Swift 6.2 |
|--------|--------------|-----------|
| Nonisolated async | Offloads to background | Inherits caller's isolation |
| Protocol conformance on actors | Warning/error | `@MainActor Conformance` is safe |
| Default isolation | Nonisolated | Can set @MainActor per target |
| Offloading to background | `Task.detached` | `@concurrent` |
| Global mutable state | Manual isolation | Default to @MainActor |

## Compatibility

- Swift 6.2 changes are opt-in via build settings and upcoming features
- Code compiled under Swift 6.0 still works
- Enable features incrementally — don't flip everything at once
- iOS 26+ projects benefit most from full adoption
