<overview>
Checklist-driven review workflow for existing SwiftUI views. Covers state management, body size, accessibility, performance, and modern API usage. Related: workflows/build-view.md (building views), all references for deep dives on specific findings.
</overview>

<process>
## Workflow: Review SwiftUI View

### Step 1 — Read the View

Read the entire file. Note:
- Total line count of `body`
- Property wrappers used
- Navigation patterns
- Modifiers applied
- Preview presence

### Step 2 — Check State Management

| Check | Pass? |
|-------|-------|
| `@State` is private | |
| No `@StateObject` with `@Observable` | |
| No `@Published` with `@Observable` | |
| No `@EnvironmentObject` (use `@Environment(Type.self)`) | |
| No `@ObservedObject` for new code (use `@Bindable` or `let`) | |
| `@Binding` only where child needs to modify parent state | |
| `onChange` uses two-parameter form | |

### Step 3 — Check Body Size

| Check | Pass? |
|-------|-------|
| `body` under ~30 lines | |
| Subviews extracted as computed properties or separate structs | |
| No object creation in body (formatters, dates) | |
| No heavy computation in body | |

### Step 4 — Check Accessibility

| Check | Pass? |
|-------|-------|
| All buttons/controls have `.accessibilityLabel` | |
| Decorative images use `.accessibilityHidden(true)` | |
| Text uses semantic font styles or `relativeTo:` | |
| Animations respect `accessibilityReduceMotion` | |
| Tap targets at least 44x44pt | |
| Related elements grouped appropriately | |

### Step 5 — Check Performance

| Check | Pass? |
|-------|-------|
| ForEach items have stable identity | |
| Large lists use LazyVStack/LazyHStack | |
| Specific data passed (not whole objects) | |
| Formatters cached as static properties | |

### Step 6 — Check Modern API Usage

| Check | Pass? |
|-------|-------|
| No `NavigationView` (use `NavigationStack`) | |
| No `PreviewProvider` (use `#Preview`) | |
| No `.animation(_:)` without value | |
| No `.foregroundColor()` (use `.foregroundStyle()`) | |
| No `DispatchQueue.main.async` (use `@MainActor`) | |
| No Combine for simple async (use async/await) | |

### Step 7 — Check Localization

| Check | Pass? |
|-------|-------|
| No hardcoded user-facing strings | |
| `String(localized:)` used for all Text, Button labels, etc. | |
| Accessibility identifiers remain plain strings | |

### Step 8 — Report Findings

For each issue found:
1. State the file and line
2. Name the rule being violated
3. Show brief before/after fix
4. Categorize severity: blocker / warning / suggestion

End with prioritized summary of most impactful changes.
</process>

<success_criteria>
## Review Complete When

- All 7 check categories evaluated
- Each finding has file, line, rule, and fix
- Findings prioritized by impact
- No false positives (only report genuine issues)
</success_criteria>
