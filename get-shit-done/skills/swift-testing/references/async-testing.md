<overview>
Async test patterns: confirmation(), actor isolation, callback bridging, time limits, and networking mocks. Read when testing async code, debugging flaky tests, or dealing with concurrency in tests. Related: fundamentals.md (basic assertions), mocking.md (protocol injection).
</overview>

## Async Tests

```swift
@Test func dataLoads() async throws {
    let service = DataService(client: MockClient())
    let items = try await service.fetchAll()
    #expect(items.count > 0)
}
```

Mark tests `async` when the code under test is async.

## confirmation() Pattern

Replaces XCTestExpectation. Use when testing that a callback or event fires:

```swift
@Test func workerCompletesThreeTimes() async {
    let worker = Worker()
    await confirmation(expectedCount: 3) { confirm in
        for _ in 0..<3 {
            await worker.run()
            confirm()
        }
    }
}
```

### Critical: Callbacks Don't Work

`confirmation()` does NOT wait for completion handlers:

```swift
// ❌ WRONG — confirmation exits before callback fires
await confirmation { confirm in
    service.fetch { result in
        confirm()  // Too late — confirmation already returned
    }
}

// ✓ CORRECT — make code async and await it
await confirmation { confirm in
    let result = await service.fetch()
    confirm()
}

// ✓ CORRECT — wrap callback in continuation
await confirmation { confirm in
    await withCheckedContinuation { continuation in
        service.fetch { _ in
            confirm()
            continuation.resume()
        }
    }
}
```

### Range-Based Confirmations (Swift 6.1+)

```swift
await confirmation(expectedCount: 5...10) { confirm in
    // Between 5 and 10 calls expected
}

await confirmation(expectedCount: 5...) { confirm in
    // At least 5 calls
}

await confirmation(expectedCount: 0) { confirm in
    // Must NEVER be called
}
```

## Actor Isolation in Tests

```swift
@Test @MainActor
func viewModelUpdatesUI() async {
    let vm = ProfileViewModel()
    await vm.load()
    #expect(vm.profile != nil)
}
```

Mark test with `@MainActor` when the system under test requires it. Mark whole suite if all tests need it.

## Testing Pre-Concurrency Code

Wrap callback-based APIs with `withCheckedContinuation`:

```swift
@Test func legacyAPIReturnsData() async {
    await withCheckedContinuation { continuation in
        legacyService.fetch { result in
            #expect(result.isSuccess)
            continuation.resume()
        }
    }
}
```

## Networking Mocks

Zero live networking in unit tests. Define protocol, inject mock:

```swift
protocol NetworkClient: Sendable {
    func data(for request: URLRequest) async throws -> (Data, URLResponse)
}

struct MockNetworkClient: NetworkClient {
    var responseData: Data = Data()
    var statusCode: Int = 200

    func data(for request: URLRequest) async throws -> (Data, URLResponse) {
        let response = HTTPURLResponse(url: request.url!, statusCode: statusCode, httpVersion: nil, headerFields: nil)!
        return (responseData, response)
    }
}
```

## .serialized Caveat

**`.serialized` ONLY works on parameterized tests.** It has NO effect on non-parameterized tests.

```swift
// ✓ This works — parameterized tests run in order
@Test(.serialized, arguments: [1, 2, 3])
func sequentialWork(value: Int) async { ... }

// ❌ This does NOTHING — non-parameterized test
@Suite(.serialized)  // Has no effect!
struct MyTests {
    @Test func a() { ... }
    @Test func b() { ... }
}
```

For serialized non-parameterized tests, use `swift-concurrency-extras` with `withMainSerialExecutor`.

## Time Limits

```swift
@Test(.timeLimit(.minutes(1)))
func networkTest() async throws {
    let result = try await client.fetch()
    #expect(result.count > 0)
}
```

Only `.minutes()` is available. Shorter of suite + test limit wins.

## Anti-Patterns

| Pattern | Problem | Fix |
|---------|---------|-----|
| `wait(for:)` in async test | Deadlock | Use `await fulfillment(of:)` (XCTest) or `confirmation()` (Swift Testing) |
| Live network calls in tests | Flaky, slow | Inject mock client |
| Missing `@MainActor` | Race condition | Add to test or suite |
| `.serialized` on suite | Does nothing | Use parameterized or main serial executor |
