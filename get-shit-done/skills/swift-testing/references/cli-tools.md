<overview>
Test execution from the command line: xcodebuild test, swift test, destinations, filtering, output formatting, and CI patterns. MCP test tools are in the mcp-tools skill — see `skills/mcp-tools/references/tool-catalog.md` for RunAllTests, RunSomeTests, GetTestList. Related: migration.md (running both frameworks).
</overview>

## xcodebuild test (Xcode Projects)

### Run All Tests

```bash
xcodebuild test \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 16' \
    2>&1 | xcpretty
```

### Run Specific Test Target

```bash
xcodebuild test \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 16' \
    -only-testing:MyAppTests
```

### Run Specific Test Suite

```bash
xcodebuild test \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 16' \
    -only-testing:MyAppTests/UserProfileTests
```

### Run Specific Test Method

```bash
xcodebuild test \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 16' \
    -only-testing:MyAppTests/UserProfileTests/validEmail
```

### Skip Specific Tests

```bash
xcodebuild test \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 16' \
    -skip-testing:MyAppTests/SlowTests
```

## swift test (SPM Packages)

### Run All Tests

```bash
swift test
```

### Run With Filter

```bash
swift test --filter UserProfileTests
swift test --filter "validEmail"
```

### Parallel Testing

```bash
swift test --parallel
```

## Destinations

| Destination | Value |
|-------------|-------|
| iPhone 16 Simulator | `'platform=iOS Simulator,name=iPhone 16'` |
| iPhone 16 Pro Max | `'platform=iOS Simulator,name=iPhone 16 Pro Max'` |
| iPad (any) | `'platform=iOS Simulator,name=iPad (10th generation)'` |
| macOS | `'platform=macOS'` |
| Any simulator | `'platform=iOS Simulator,name=Any iOS Simulator Device'` |

List available:
```bash
xcodebuild -showdestinations -scheme MyApp 2>/dev/null | grep "iOS Simulator"
```

## Output Formatting

### xcpretty (Recommended)

```bash
xcodebuild test ... 2>&1 | xcpretty
```

Install: `gem install xcpretty`

### JUnit Output (for CI)

```bash
xcodebuild test ... 2>&1 | xcpretty --report junit --output report.xml
```

### Raw xcodebuild Output

Without xcpretty, filter for results:
```bash
xcodebuild test ... 2>&1 | grep -E "Test (Suite|Case|Passed|Failed)"
```

## CI Patterns

### GitHub Actions

```yaml
- name: Run tests
  run: |
    xcodebuild test \
      -scheme MyApp \
      -destination 'platform=iOS Simulator,name=iPhone 16' \
      -resultBundlePath TestResults.xcresult \
      2>&1 | xcpretty
```

### Xcode Cloud

Tests run automatically when the scheme includes test targets. Configure in Xcode Cloud workflow settings.

### Fastlane

```ruby
lane :test do
  scan(
    scheme: "MyApp",
    device: "iPhone 16",
    clean: true
  )
end
```

## Common Flags

| Flag | Purpose |
|------|---------|
| `-resultBundlePath` | Save test results for later inspection |
| `-enableCodeCoverage YES` | Generate coverage data |
| `-parallel-testing-enabled YES` | Enable parallel test execution |
| `-test-iterations 3` | Run tests multiple times (detect flaky) |
| `-retry-tests-on-failure` | Retry failed tests |

## MCP Test Tools (Cross-Reference)

When MCP servers are available, prefer them over CLI:
- `mcp__xcode__RunAllTests` — full test suite
- `mcp__xcode__RunSomeTests` — specific targets/suites/methods
- `mcp__xcode__GetTestList` — list available tests

See `skills/mcp-tools/references/tool-catalog.md` for the full catalog.
