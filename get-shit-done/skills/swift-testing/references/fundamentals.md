<overview>
Core Swift Testing patterns: @Test, @Suite, #expect, #require, suite structure, and test organization. Read when writing new tests or learning Swift Testing basics. Related: parameterized.md (arguments), traits-tags.md (traits), migration.md (from XCTest).
</overview>

## Test Structure

```swift
import Testing
@testable import MyApp

struct PlayerTests {
    let sut: Player

    init() {
        sut = Player(name: "Ada", score: 0)
    }

    @Test("Player name is set correctly")
    func nameIsCorrect() {
        #expect(sut.name == "Ada")
    }

    @Test func scoreStartsAtZero() {
        #expect(sut.score == 0)
    }
}
```

**Rules:**
- Use **struct** over class (unless subclassing needed)
- `init()` replaces `setUp()` — must accept zero parameters
- `init()` can be `async` and/or `throws`
- No `test` prefix required — `scoreStartsAtZero()` is valid
- Tests run in **random, parallel order** — each must be independent

## @Suite

`@Suite` is optional. Any type with `@Test` methods is auto-detected. Use it only to:
- Set a display name: `@Suite("Player Validation")`
- Attach traits: `@Suite(.tags(.model))`

```swift
@Suite("Authentication Tests")
struct AuthTests {
    @Test func loginSucceeds() { ... }
    @Test func loginFailsWithBadCredentials() { ... }
}
```

## #expect (Assertions)

```swift
#expect(value == 42)
#expect(array.isEmpty)
#expect(name.contains("Ada"))
#expect(result == .success)
```

**Important:** Do NOT use `!` for negation — use `== false` for clear failure messages:
```swift
// WRONG — failure message is cryptic
#expect(!isLoggedIn)

// CORRECT — clear failure message
#expect(isLoggedIn == false)
```

### User-Facing Messages

```swift
#expect(score > 0, "Score should be positive after playing a round")
```

## #require (Preconditions)

Stops the test on failure (requires `throws` on the test method). Use for preconditions at the start.

```swift
@Test func profileLoads() throws {
    let user = try #require(fetchUser())  // Unwraps or fails
    #expect(user.name == "Ada")
}
```

`#require` can unwrap optionals:
```swift
let value = try #require(optionalValue)  // Like XCTUnwrap
```

## Error Testing

```swift
// Expect specific error type
@Test func invalidInputThrows() {
    #expect(throws: ValidationError.self) {
        try validate(input: "")
    }
}

// Expect specific error value (Swift 6.1+)
@Test func specificErrorThrown() {
    let error = #expect(throws: GameError.self) {
        try playGame(at: 22)
    }
    #expect(error == .disallowedTime)
}

// Expect NO error
@Test func validInputDoesNotThrow() {
    #expect(throws: Never.self) {
        try validate(input: "valid")
    }
}
```

## Test Organization

- Mirror production code structure
- Group related tests in suites
- Use tags for cross-cutting concerns (`.networking`, `.slow`, `.smoke`)
- One behavior per test (multiple `#expect` lines OK if testing same behavior)

## What to Test vs What NOT to Test

| Test | Don't Test |
|------|-----------|
| ViewModels (logic, state) | SwiftUI View `body` |
| Services, Repositories | Apple framework internals |
| Data Models (encoding) | Simple property wrappers |
| Pure functions, algorithms | Static layout (use Previews) |
| State machines | Third-party library internals |
