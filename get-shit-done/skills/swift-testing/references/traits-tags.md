<overview>
Test traits and tags: .enabled, .disabled, .timeLimit, .bug, custom tags, test plans, and withKnownIssue. Read when controlling test execution, tracking bugs, or organizing tests by concern. Related: fundamentals.md (basic structure), parameterized.md (.serialized trait).
</overview>

## Condition Traits

### .enabled / .disabled

```swift
@Test(.disabled("Server migration in progress"))
func syncWithCloud() { ... }

@Test(.enabled(if: ProcessInfo.processInfo.environment["CI"] != nil))
func ciOnlyTest() { ... }
```

### .timeLimit

```swift
@Test(.timeLimit(.minutes(1)))
func networkRequest() async { ... }
```

**Only `.minutes()` is available** — cannot use `.seconds()`. Shortest limit wins when combined with suite-level limit.

### .bug

Track bug references on tests:

```swift
@Test(.bug(id: 182))
func loginCrashFixed() { ... }

@Test(.bug("https://github.com/org/repo/issues/42", "Null pointer on empty input"))
func emptyInputHandled() { ... }
```

## Tags

Define custom tags for cross-cutting concerns:

```swift
extension Tag {
    @Tag static var networking: Self
    @Tag static var slow: Self
    @Tag static var smoke: Self
    @Tag static var edgeCase: Self
}

@Test(.tags(.networking))
func fetchUserProfile() async { ... }

@Suite(.tags(.slow))
struct PerformanceTests {
    @Test func heavyComputation() { ... }
}
```

Run tagged tests selectively in Xcode Test Plans or via `--filter`.

## withKnownIssue

Wraps code with known bugs. Test passes if the issue is recorded (expected failure):

```swift
@Test func brokenFeature() {
    withKnownIssue("Crashes on empty array — tracked in #182") {
        let result = processItems([])
        #expect(result.isEmpty)
    }
}
```

If the code STOPS failing, the test fails — alerting you that the bug was fixed.

### Intermittent Issues

```swift
withKnownIssue("Flaky on CI", isIntermittent: true) {
    // May or may not fail — both are acceptable
    try await flakyCITest()
}
```

## Test Scoping Traits (Swift 6.1+)

Custom traits that provide scoped setup/teardown:

```swift
struct DefaultPlayerTrait: TestTrait, TestScoping {
    func provideScope(for test: Test, testCase: Test.Case?, performing function: @Sendable () async throws -> Void) async throws {
        try await Player.$current.withValue(Player(name: "Test")) {
            try await function()
        }
    }
}

extension Trait where Self == DefaultPlayerTrait {
    static var defaultPlayer: Self { Self() }
}

@Test(.defaultPlayer)
func welcomeScreen() {
    #expect(Player.current.name == "Test")
}
```

## @available on Tests

Apply `@available` on **individual tests**, not suites:

```swift
@available(iOS 18, *)
@Test func meshGradientRenders() { ... }
```

## Trait Combinations

Traits compose naturally:

```swift
@Test(.tags(.networking), .timeLimit(.minutes(2)), .bug(id: 99))
func complexNetworkTest() async { ... }
```
