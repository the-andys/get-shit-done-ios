<overview>
XCTest→Swift Testing migration: assertion mapping, incremental migration strategy, coexistence rules. Read when converting existing XCTest tests to Swift Testing. Related: fundamentals.md (Swift Testing patterns), cli-tools.md (running both frameworks).
</overview>

## Assertion Mapping

| XCTest | Swift Testing |
|--------|--------------|
| `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| `XCTAssertNotEqual(a, b)` | `#expect(a != b)` |
| `XCTAssertTrue(x)` | `#expect(x)` or `#expect(x == true)` |
| `XCTAssertFalse(x)` | `#expect(x == false)` |
| `XCTAssertNil(x)` | `#expect(x == nil)` |
| `XCTAssertNotNil(x)` | `#expect(x != nil)` |
| `XCTAssertGreaterThan(a, b)` | `#expect(a > b)` |
| `XCTAssertLessThan(a, b)` | `#expect(a < b)` |
| `XCTUnwrap(optional)` | `try #require(optional)` |
| `XCTAssertThrowsError` | `#expect(throws: ErrorType.self) { }` |
| `XCTAssertIdentical(a, b)` | `#expect(a === b)` |
| `XCTFail("msg")` | `Issue.record("msg")` |

## Float Tolerance

Swift Testing has NO built-in float tolerance. Use Apple's **Swift Numerics**:

```swift
import Numerics

#expect(celsius.isApproximatelyEqual(to: 0, absoluteTolerance: 0.000001))
```

**Do NOT add Swift Numerics without user permission.** If not available, use manual comparison:

```swift
#expect(abs(actual - expected) < 0.001)
```

## Structure Mapping

| XCTest | Swift Testing |
|--------|--------------|
| `class MyTests: XCTestCase` | `struct MyTests` |
| `override func setUp()` | `init()` |
| `override func tearDown()` | `deinit` or test scope trait |
| `func testSomething()` | `@Test func something()` |
| `setUpWithError()` | `init() throws` |
| `XCTestExpectation` | `confirmation()` |
| `wait(for:)` | `await confirmation()` |

## Incremental Migration Strategy

1. **Don't convert everything at once.** Migrate one test class at a time.
2. **Start with simplest tests** — pure logic, no async, no mocks.
3. **Both frameworks coexist** in the same target. Swift Testing and XCTest can live side by side.
4. **Keep UI tests in XCTest** — Swift Testing cannot do UI tests.
5. **Don't mix frameworks in one file** — each file should be all Swift Testing or all XCTest.

## Conversion Steps (Per Test Class)

1. Remove `import XCTest`, add `import Testing`
2. Change `class MyTests: XCTestCase` to `struct MyTests`
3. Replace `setUp()` with `init()`
4. Remove `test` prefix from method names
5. Add `@Test` to each test method
6. Replace `XCTAssert*` with `#expect` / `#require`
7. Replace `XCTestExpectation` with `confirmation()`
8. Add traits where applicable (`.tags()`, `.timeLimit()`)
9. Look for parameterized test opportunities
10. Run and verify

## Coexistence Rules

- Both frameworks can be in the same test target
- Swift Testing tests run in parallel; XCTest tests run serially by default
- `@Test` methods are invisible to XCTest; `XCTestCase` subclasses are invisible to Swift Testing
- Shared helpers can be used by both (just regular Swift functions)

## What Stays in XCTest

| Use Case | Why |
|----------|-----|
| UI tests (XCUITest) | No Swift Testing equivalent |
| Performance tests (`measure {}`) | Not available in Swift Testing |
| Tests requiring serial execution | `.serialized` only works on parameterized tests |
| iOS 16 compatibility | Swift Testing requires iOS 17+ |
