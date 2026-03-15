<overview>
Parameterized testing: @Test(arguments:), combinatorics, custom argument types, and zip for pairwise testing. Read when you have repetitive test cases that differ only by input data. Related: fundamentals.md (basic @Test), traits-tags.md (traits on parameterized tests).
</overview>

## Basic Parameterized Test

```swift
@Test("Valid emails pass validation", arguments: [
    "user@example.com",
    "admin@domain.org",
    "test+tag@sub.domain.co",
])
func validEmails(email: String) {
    let result = EmailValidator.validate(email)
    #expect(result == .valid)
}
```

## Multiple Arguments (Cartesian Product)

Takes max 2 argument collections. Produces all combinations:

```swift
@Test("Discount applies correctly", arguments: [
    Plan.free, .pro, .enterprise
], [
    Duration.monthly, .annual
])
func discountCalculation(plan: Plan, duration: Duration) {
    let discount = calculateDiscount(plan: plan, duration: duration)
    #expect(discount >= 0)
}
```

This runs 3 × 2 = 6 test cases.

## Pairwise Arguments with zip

When you need matched pairs (not all combinations), use `zip`:

```swift
@Test("Input maps to expected output", arguments: zip(
    ["hello", "world", ""],
    [5, 5, 0]
))
func stringLength(input: String, expected: Int) {
    #expect(input.count == expected)
}
```

## Custom Argument Types

Arguments must conform to `Sendable`. For custom types, also conform to `CustomTestStringConvertible` for readable output:

```swift
struct TestCase: Sendable, CustomTestStringConvertible {
    let input: String
    let expected: Int
    var testDescription: String { "\(input) → \(expected)" }
}

@Test("Parse integers", arguments: [
    TestCase(input: "42", expected: 42),
    TestCase(input: "-1", expected: -1),
    TestCase(input: "0", expected: 0),
])
func parseInt(testCase: TestCase) {
    #expect(Int(testCase.input) == testCase.expected)
}
```

## Enum-Based Arguments

Enums with `CaseIterable` work naturally:

```swift
enum Theme: CaseIterable, Sendable {
    case light, dark, system
}

@Test("All themes render", arguments: Theme.allCases)
func themeRendering(theme: Theme) {
    let view = AppView(theme: theme)
    #expect(view.backgroundColor != nil)
}
```

## Serialized Parameterized Tests

**Critical:** `.serialized` trait ONLY works on parameterized tests. It has no effect on non-parameterized tests.

```swift
@Test(.serialized, arguments: [1, 2, 3])
func sequentialTest(value: Int) {
    // Runs value=1, then value=2, then value=3 (in order)
}
```

## When to Parameterize

| Scenario | Approach |
|----------|----------|
| Same logic, 3+ input variations | Parameterized test |
| Different assertions per input | Separate tests |
| Edge cases + happy path | Parameterized with edge cases included |
| All enum cases | `arguments: MyEnum.allCases` |
