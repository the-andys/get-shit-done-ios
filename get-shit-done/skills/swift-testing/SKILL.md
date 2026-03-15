---
name: swift-testing
description: Swift Testing framework — @Test, #expect, parameterized tests, mocking, ViewModel testing, XCTest migration, test CLI
---

<essential_principles>
## How This Skill Works

1. **Swift Testing is the default** for all new unit/integration tests. XCTest is only for UI tests (XCUITest) and performance tests.
2. **Struct suites, not classes.** Use struct test suites with `init()` for setup. No `setUp()`/`tearDown()`.
3. **#expect for assertions, #require for preconditions.** `#require` stops the test on failure; `#expect` continues.
4. **Parameterized tests are powerful.** Convert repetitive test cases into `@Test(arguments:)`.
5. **Test ViewModels, not Views.** Never test SwiftUI `body` directly. Test the `@Observable` ViewModel instead.
6. **Zero live networking.** Inject protocol-based mocks for all external dependencies.
7. **Tests run in parallel and random order.** Each test must be fully independent.
8. **60-second triage first.** Before writing tests, identify: target code, isolation boundary, test framework (Swift Testing vs XCUITest).
</essential_principles>

<intake>
## What do you need?

1. Write new tests for a feature
2. Migrate existing XCTest tests to Swift Testing
3. Debug a flaky or failing test
4. Run tests (CLI or MCP)
5. Mock dependencies for testing
6. Test a ViewModel with @Observable
7. Write parameterized or async tests
</intake>

<routing>
| Response | Reference / Workflow |
|----------|---------------------|
| 1, "write", "new test", "TDD" | `workflows/write-tests.md` |
| 2, "migrate", "XCTest", "convert" | `references/migration.md` |
| 3, "flaky", "failing", "debug test" | `references/async-testing.md` |
| 4, "run", "CLI", "xcodebuild", "MCP" | `references/cli-tools.md` |
| 5, "mock", "inject", "protocol", "dependency" | `references/mocking.md` |
| 6, "ViewModel", "@Observable", "state" | `references/viewmodel-testing.md` |
| 7, "parameterized", "arguments", "async", "confirmation" | `references/parameterized.md` or `references/async-testing.md` |
| "@Test", "#expect", "@Suite", "basics" | `references/fundamentals.md` |
| "tag", "trait", "disable", "timeLimit" | `references/traits-tags.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/fundamentals.md | @Test, @Suite, #expect, #require, suite structure, zero-arg init |
| references/parameterized.md | @Test(arguments:), combinatorics, custom argument types, zip |
| references/traits-tags.md | .enabled, .disabled, .timeLimit, .bug, tags, withKnownIssue |
| references/async-testing.md | Async patterns, confirmation(), actor isolation, time limits, networking mocks |
| references/mocking.md | Protocol+mock pattern, DI, mock network/cache/UserDefaults |
| references/viewmodel-testing.md | @Observable ViewModel testing, protocol-injected repos, optimistic updates |
| references/migration.md | XCTest→Swift Testing mapping, incremental migration, coexistence |
| references/cli-tools.md | xcodebuild test, swift test, destinations, CI patterns |
| workflows/write-tests.md | Step-by-step: identify target → choose framework → write → run → verify |
</reference_index>

<canonical_terminology>
## Terminology

- **@Test** (not: `func test...` with XCTest prefix)
- **#expect** (not: XCTAssertEqual, XCTAssertTrue)
- **#require** (not: XCTUnwrap)
- **@Suite** (not: XCTestCase)
- **confirmation()** (not: XCTestExpectation)
- **Swift Testing** (not: XCTest for unit tests)
- **XCUITest** (not: Swift Testing for UI tests — Swift Testing cannot do UI tests)
</canonical_terminology>
