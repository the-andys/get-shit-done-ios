<overview>
Testing concurrent code: async test patterns with Swift Testing, actor testing, concurrency-aware mocking, @MainActor test isolation, handling flaky tests, and XCTest legacy patterns. Read when writing tests for async code or debugging flaky tests. Related: swift-testing skill (full testing reference), actors.md (isolation).
</overview>

## Swift Testing (Recommended)

```swift
import Testing

@Test
@MainActor
func searchReturnsResults() async {
    let searcher = ArticleSearcher()
    await searcher.search("swift")
    #expect(!searcher.results.isEmpty)
}
```

Mark test with `@MainActor` if the system under test requires it.

### Async Setup

```swift
@Suite("Profile Tests")
struct ProfileTests {
    let viewModel: ProfileViewModel

    init() async throws {
        viewModel = ProfileViewModel()
        await viewModel.load()
    }

    @Test func profileLoaded() {
        #expect(viewModel.profile != nil)
    }
}
```

## Awaiting Async Callbacks

### Continuation (Unstructured)

```swift
@Test func delegateNotified() async {
    let result = await withCheckedContinuation { continuation in
        service.fetchData { result in
            continuation.resume(returning: result)
        }
    }
    #expect(result.isSuccess)
}
```

### Confirmation (Structured)

```swift
@Test func notificationReceived() async {
    await confirmation { confirm in
        observer.onUpdate = { confirm() }
        await triggerUpdate()
    }
}
```

## Handling Flaky Tests

Problem: race conditions when checking state during concurrent execution.

### Serialized Execution

```swift
@Suite(.serialized)
struct OrderTests {
    // Tests run sequentially, not in parallel
}
```

### Main Serial Executor (swift-concurrency-extras)

```swift
import ConcurrencyExtras

@Test func stateUpdates() async {
    await withMainSerialExecutor {
        let vm = ViewModel()
        await vm.load()
        #expect(vm.items.count == 3)
    }
}
```

Forces deterministic ordering — eliminates timing-dependent failures.

## Testing Actors

```swift
@Test func cacheStoresValues() async {
    let cache = ImageCache()
    let image = UIImage()
    let url = URL(string: "https://example.com/photo.jpg")!

    await cache.store(image, for: url)
    let cached = await cache.image(for: url)
    #expect(cached != nil)
}
```

All actor method calls use `await` — tests naturally handle isolation.

## Testing with Mock Dependencies

```swift
protocol DataRepository: Sendable {
    func fetchItems() async throws -> [Item]
}

struct MockRepository: DataRepository {
    var items: [Item] = []
    var error: Error?

    func fetchItems() async throws -> [Item] {
        if let error { throw error }
        return items
    }
}

@Test @MainActor
func loadHandlesError() async {
    let vm = ItemViewModel(repository: MockRepository(error: NetworkError.noConnection))
    await vm.load()
    #expect(vm.errorMessage != nil)
}
```

## XCTest Patterns (Legacy)

```swift
func testAsyncLoad() async throws {
    let vm = ProfileViewModel()
    await vm.load()
    XCTAssertNotNil(vm.profile)
}
```

**Warning:** Use `await fulfillment(of:)`, NOT `wait(for:)` — `wait` causes deadlocks in async contexts.

```swift
func testNotification() async {
    let expectation = expectation(description: "received")
    observer.onUpdate = { expectation.fulfill() }
    await triggerUpdate()
    await fulfillment(of: [expectation], timeout: 2.0)
}
```

## Anti-Patterns

| Pattern | Problem | Fix |
|---------|---------|-----|
| `wait(for:)` in async test | Deadlock | Use `await fulfillment(of:)` |
| Missing `@MainActor` on test | Race condition with UI ViewModel | Add `@MainActor` |
| No cancellation test | Untested code path | Test that cancelled tasks clean up |
| Testing timing, not state | Flaky | Use serialized execution or confirmations |
