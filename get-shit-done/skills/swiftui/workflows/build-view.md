<overview>
Step-by-step workflow for building a SwiftUI view from requirements to verified output. Adapted from GSD-2 build-new-app workflow for building individual views. Related: workflows/review-view.md (review checklist after building).
</overview>

<required_reading>
Depending on the view's domain, read relevant references first:
- State choices: `references/state-management.md`
- Navigation: `references/navigation.md`
- Components: `references/components.md`
</required_reading>

<process>
## Workflow: Build a SwiftUI View

### Step 1 — Understand Requirements

- What data does this view display?
- What actions can the user take?
- Where does data come from (API, SwiftData, passed as prop)?
- What navigation does this view need (push, sheet, alert)?
- Any specific accessibility requirements?

### Step 2 — Design the State Model

Choose property wrappers:
- View-owned value → `@State private var`
- View-owned @Observable → `@State private var viewModel = VM()`
- Parent-passed, child modifies → `@Binding var`
- Parent-passed, read-only → `let`
- Environment → `@Environment(\.key)` or `@Environment(Type.self)`

### Step 3 — Structure the View Hierarchy

Sketch the component tree:
```
MyView
├── NavigationStack
│   ├── headerSection (computed view)
│   ├── contentSection (computed view or child View struct)
│   ├── footerSection
│   └── .toolbar { toolbarContent }
├── .sheet(item: $selectedItem) { ... }
└── .task { await loadData() }
```

### Step 4 — Compose the UI

Write the view following property ordering:
1. Environment
2. Properties (let, @Binding)
3. @State
4. body
5. Subviews (extracted computed properties)
6. Actions

Keep `body` under ~30 lines. Extract subviews as computed properties or separate structs.

### Step 5 — Add Accessibility

For every interactive element:
- [ ] `.accessibilityLabel` on icon-only buttons
- [ ] `.accessibilityHidden(true)` on decorative images
- [ ] Semantic font styles for Dynamic Type
- [ ] Group related elements with `.accessibilityElement(children:)`
- [ ] Minimum 44x44pt tap targets

### Step 6 — Add Localization

All user-facing text uses `String(localized:)`:
```swift
Text(String(localized: "profile_title"))
Button(String(localized: "save_action")) { save() }
```

### Step 7 — Add Preview

```swift
#Preview {
    MyView(item: .preview)
        .environment(Store.preview)
}

#Preview("Empty State") {
    MyView(item: nil)
}
```

### Step 8 — Verify

1. **Build** — No compiler errors
2. **Preview** — Renders correctly in light/dark mode
3. **Accessibility** — VoiceOver labels present on all interactive elements
4. **Test** — Write test if view has complex logic (via ViewModel)
</process>

<success_criteria>
## Done When

- View builds without warnings
- Preview renders correctly
- Accessibility labels on all interactive elements
- User-facing strings localized
- Body under ~30 lines
- Tests pass (if applicable)
</success_criteria>
