<overview>
TDD is about design quality, not coverage metrics. The red-green-refactor cycle forces you to think about behavior before implementation, producing cleaner interfaces and more testable code.

**Principle:** If you can describe the behavior as `#expect(fn(input) == output)` before writing `fn`, TDD improves the result.

**iOS Testing Stack:**
- **Swift Testing** (`@Suite`, `@Test`, `#expect`) — Primary framework for unit tests
- **XCTest** (`XCTestCase`) — For UI tests (XCUITest) and iOS 16 compatibility
- See `@~/.claude/get-shit-done/references/ios-testing.md` for complete patterns

**Key insight:** TDD work is fundamentally heavier than standard tasks—it requires 2-3 execution cycles (RED → GREEN → REFACTOR), each with file reads, test runs, and potential debugging. TDD features get dedicated plans to ensure full context is available throughout the cycle.
</overview>

<when_to_use_tdd>
## When TDD Improves Quality

**TDD candidates (create a TDD plan):**
- ViewModels with defined inputs/outputs
- Services and repositories with request/response contracts
- Data transformations, Codable parsing, formatting
- Validation rules and constraints
- Algorithms with testable behavior
- State machines and workflows
- Utility functions and extensions with clear specifications

**Skip TDD (use standard plan with `type="auto"` tasks):**
- SwiftUI View body layout and styling
- Configuration and Info.plist changes
- Navigation setup and routing
- Simple @Model definitions with no business logic
- Exploratory prototyping

**Heuristic:** Can you write `#expect(fn(input) == output)` before writing `fn`?
→ Yes: Create a TDD plan
→ No: Use standard plan, add tests after if needed
</when_to_use_tdd>

<tdd_plan_structure>
## TDD Plan Structure

Each TDD plan implements **one feature** through the full RED-GREEN-REFACTOR cycle.

```markdown
---
phase: XX-name
plan: NN
type: tdd
---

<objective>
[What feature and why]
Purpose: [Design benefit of TDD for this feature]
Output: [Working, tested feature]
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@~/.claude/get-shit-done/references/ios-testing.md
@relevant/source/File.swift
</context>

<feature>
  <name>[Feature name]</name>
  <files>[source file, test file]</files>
  <behavior>
    [Expected behavior in testable terms]
    Cases: input → expected output
  </behavior>
  <implementation>[How to implement once tests pass]</implementation>
</feature>

<verification>
[Test command that proves feature works]
</verification>

<success_criteria>
- Failing test written and committed
- Implementation passes test
- Refactor complete (if needed)
- All 2-3 commits present
</success_criteria>

<output>
After completion, create SUMMARY.md with:
- RED: What test was written, why it failed
- GREEN: What implementation made it pass
- REFACTOR: What cleanup was done (if any)
- Commits: List of commits produced
</output>
```

**One feature per TDD plan.** If features are trivial enough to batch, they're trivial enough to skip TDD—use a standard plan and add tests after.
</tdd_plan_structure>

<execution_flow>
## Red-Green-Refactor Cycle

**RED - Write failing test:**
1. Create test file following project conventions
2. Write test describing expected behavior (from `<behavior>` element)
3. Run test - it MUST fail
4. If test passes: feature exists or test is wrong. Investigate.
5. Commit: `test({phase}-{plan}): add failing test for [feature]`

**GREEN - Implement to pass:**
1. Write minimal code to make test pass
2. No cleverness, no optimization - just make it work
3. Run test - it MUST pass
4. Commit: `feat({phase}-{plan}): implement [feature]`

**REFACTOR (if needed):**
1. Clean up implementation if obvious improvements exist
2. Run tests - MUST still pass
3. Only commit if changes made: `refactor({phase}-{plan}): clean up [feature]`

**Result:** Each TDD plan produces 2-3 atomic commits.
</execution_flow>

<test_quality>
## Good Tests vs Bad Tests

**Test behavior, not implementation:**
- Good: "returns formatted date string"
- Bad: "calls formatDate helper with correct params"
- Tests should survive refactors

**One concept per test:**
- Good: Separate tests for valid input, empty input, malformed input
- Bad: Single test checking all edge cases with multiple assertions

**Descriptive names:**
- Good: "should reject empty email", "returns null for invalid ID"
- Bad: "test1", "handles error", "works correctly"

**No implementation details:**
- Good: Test public API, observable behavior
- Bad: Mock internals, test private methods, assert on internal state
</test_quality>

<framework_setup>
## Test Framework Setup (iOS)

Swift Testing and XCTest are built into Xcode — no installation needed.

**1. Detect project type:**
```bash
# Xcode project
if [ -d *.xcodeproj ] || [ -d *.xcworkspace ]; then echo "xcode"; fi

# Swift Package
if [ -f Package.swift ]; then echo "spm"; fi
```

**2. Framework availability:**
| Project | Framework | Setup |
|---------|-----------|-------|
| Xcode project | Swift Testing | Built-in (Xcode 16+) |
| Xcode project | XCTest | Built-in |
| SPM package | Swift Testing | Add to `testTarget` in Package.swift |
| SPM package | XCTest | Built-in |

**3. Verify setup:**
```bash
# SPM package
swift test

# Xcode project
xcodebuild test -scheme AppName -destination 'platform=iOS Simulator,name=iPhone 16'
```

**4. Create first test file:**
Follow iOS conventions for test location:
- `Tests/FeatureNameTests/` for SPM packages
- `AppNameTests/` group in Xcode project
- Name: `FeatureNameTests.swift`

**5. Test file structure (Swift Testing):**
```swift
import Testing
@testable import AppName

@Suite("Feature Name")
struct FeatureNameTests {
    @Test("should do expected behavior")
    func testBehavior() async throws {
        // Arrange
        let sut = FeatureName()

        // Act
        let result = sut.process(input)

        // Assert
        #expect(result == expected)
    }
}
```

Framework setup is trivial on iOS — Swift Testing is built-in since Xcode 16.
</framework_setup>

<error_handling>
## Error Handling

**Test doesn't fail in RED phase:**
- Feature may already exist - investigate
- Test may be wrong (not testing what you think)
- Fix before proceeding

**Test doesn't pass in GREEN phase:**
- Debug implementation
- Don't skip to refactor
- Keep iterating until green

**Tests fail in REFACTOR phase:**
- Undo refactor
- Commit was premature
- Refactor in smaller steps

**Unrelated tests break:**
- Stop and investigate
- May indicate coupling issue
- Fix before proceeding
</error_handling>

<commit_pattern>
## Commit Pattern for TDD Plans

TDD plans produce 2-3 atomic commits (one per phase):

```
test(08-02): add failing test for email validation

- Tests valid email formats accepted
- Tests invalid formats rejected
- Tests empty input handling

feat(08-02): implement email validation

- Regex pattern matches RFC 5322
- Returns boolean for validity
- Handles edge cases (empty, null)

refactor(08-02): extract regex to constant (optional)

- Moved pattern to EMAIL_REGEX constant
- No behavior changes
- Tests still pass
```

**Comparison with standard plans:**
- Standard plans: 1 commit per task, 2-4 commits per plan
- TDD plans: 2-3 commits for single feature

Both follow same format: `{type}({phase}-{plan}): {description}`

**Benefits:**
- Each commit independently revertable
- Git bisect works at commit level
- Clear history showing TDD discipline
- Consistent with overall commit strategy
</commit_pattern>

<context_budget>
## Context Budget

TDD plans target **~40% context usage** (lower than standard plans' ~50%).

Why lower:
- RED phase: write test, run test, potentially debug why it didn't fail
- GREEN phase: implement, run test, potentially iterate on failures
- REFACTOR phase: modify code, run tests, verify no regressions

Each phase involves reading files, running commands, analyzing output. The back-and-forth is inherently heavier than linear task execution.

Single feature focus ensures full quality throughout the cycle.
</context_budget>
