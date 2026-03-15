<overview>
Step-by-step workflow for writing tests: identify target → choose framework → write tests → run → verify → check coverage. Combines community skill triage with fork ViewModel testing patterns. Related: All swift-testing references for deep dives.
</overview>

<required_reading>
Depending on what you're testing:
- ViewModel logic: `references/mocking.md` + `references/viewmodel-testing.md`
- Async code: `references/async-testing.md`
- Migrating XCTest: `references/migration.md`
</required_reading>

<process>
## Workflow: Write Tests

### Step 1 — 60-Second Triage

Before writing any test, answer:

| Question | Answer |
|----------|--------|
| What am I testing? | ViewModel / Service / Model / Utility |
| What framework? | Swift Testing (default) / XCUITest (UI only) |
| Is it async? | Yes → `async throws` + `@MainActor` if needed |
| External dependencies? | Yes → Protocol + Mock |
| Multiple input variations? | Yes → Parameterized test |

### Step 2 — Choose Framework

```
Is this a UI interaction test?
├── YES → XCUITest (XCTest)
└── NO → Swift Testing
         ├── Performance measurement? → XCTest measure {}
         └── Everything else → Swift Testing @Test
```

### Step 3 — Set Up Test Structure

```swift
import Testing
@testable import MyApp

struct FeatureTests {
    let mockRepo: MockRepository
    let sut: FeatureViewModel

    init() {
        mockRepo = MockRepository()
        sut = FeatureViewModel(repository: mockRepo)
    }
}
```

- Struct over class
- `init()` for setup
- Mock dependencies injected

### Step 4 — Write Tests (Priority Order)

1. **Happy path** — normal operation succeeds
2. **Error path** — dependency throws → graceful handling
3. **Edge cases** — empty input, nil values, boundary conditions
4. **State transitions** — loading → loaded, optimistic update → revert

```swift
@Test @MainActor
func happyPath() async {
    mockRepo.stubbedItems = [.preview]
    await sut.load()
    #expect(sut.items.count == 1)
}

@Test @MainActor
func errorShowsMessage() async {
    mockRepo.shouldThrow = true
    await sut.load()
    #expect(sut.errorMessage != nil)
    #expect(sut.items.isEmpty)
}

@Test @MainActor
func emptyStateHandled() async {
    mockRepo.stubbedItems = []
    await sut.load()
    #expect(sut.items.isEmpty)
    #expect(sut.errorMessage == nil)
}
```

### Step 5 — Look for Parameterization

If 3+ tests differ only by input data → convert:

```swift
@Test("Invalid emails rejected", arguments: [
    "", "@", "missing-at.com", "no-domain@",
])
func invalidEmails(email: String) {
    #expect(EmailValidator.validate(email) == .invalid)
}
```

### Step 6 — Run Tests

```bash
# Via CLI
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16' 2>&1 | xcpretty

# Via MCP (if available)
# mcp__xcode__RunSomeTests with target/suite
```

### Step 7 — Verify

- [ ] All tests pass
- [ ] No flaky tests (run twice to confirm)
- [ ] Happy + error + edge cases covered
- [ ] Mock tracks calls (verify side effects)
- [ ] `@MainActor` on tests that test UI-bound ViewModels

### Step 8 — Coverage Check

```bash
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16' -enableCodeCoverage YES
```

Focus on:
- Business logic coverage (ViewModels, Services)
- Error handling paths
- Don't chase 100% — test what matters

</process>

<anti_patterns>
## Common Mistakes

- **Testing View body** — test the ViewModel instead
- **Live network calls** — inject protocol mocks
- **Shared test state** — each test must be independent
- **Missing @MainActor** — causes race conditions with @Observable
- **No error path tests** — always test what happens when things fail
- **Giant test methods** — one behavior per test, use parameterized for variations
</anti_patterns>

<success_criteria>
## Done When

- All new tests pass
- Happy path + error path + edge cases covered
- No flaky tests
- Mocks verify side effects (calls tracked)
- Existing tests still pass
</success_criteria>
