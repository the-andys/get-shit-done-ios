---
name: swiftui
description: SwiftUI views, state management, navigation, animations, layout, components, Liquid Glass, view composition, performance
---

<essential_principles>
## How This Skill Works

1. **Declarative mindset.** Describe what the UI should look like for a given state. Never force updates — change state and let SwiftUI react.
2. **Single source of truth.** Every piece of data has one authoritative location. Use the right property wrapper: @State for view-local, @Observable for shared, @Environment for app-wide.
3. **Composition over inheritance.** Build complex UIs from small, focused views. Extract subviews as separate structs — SwiftUI can skip their body when inputs don't change.
4. **Platform-adaptive.** Write once but respect platform idioms. Use native navigation, respect safe areas, test on target platforms.
5. **Verify visually.** Tests verify correctness, screenshots verify appearance. Build → test → preview → screenshot.
</essential_principles>

<intake>
## What do you need?

1. Build a new view
2. Review existing SwiftUI code
3. Add or fix animations (including Liquid Glass)
4. Improve performance
5. Add a component (TabView, Form, Grid, etc.)
6. Fix state management or navigation
7. Understand latest APIs (iOS 26+)
</intake>

<routing>
| Response | Reference / Workflow |
|----------|---------------------|
| 1, "build", "create", "new view" | `workflows/build-view.md` |
| 2, "review", "check", "audit" | `workflows/review-view.md` |
| 3, "animation", "transition", "Liquid Glass", "glassEffect" | `references/animations.md` |
| 4, "slow", "performance", "janky", "identity" | `references/performance.md` |
| 5, "component", "TabView", "Form", "Grid", "theming" | `references/components.md` |
| 6, "state", "@State", "@Observable", "@Binding" | `references/state-management.md` |
| 6, "navigation", "sheet", "NavigationStack", "deep link" | `references/navigation.md` |
| 7, "latest", "iOS 26", "new API", "deprecated" | `references/latest-apis.md` |
| "structure", "extract", "@ViewBuilder", "body size" | `references/view-composition.md` |
| "accessibility", "VoiceOver", "Dynamic Type" | `references/accessibility-in-views.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/state-management.md | Property wrapper decision, @Observable, @State, @Binding, @Environment, @Bindable |
| references/navigation.md | NavigationStack, sheets (enum-based), Inspector, deep links, TabView |
| references/view-composition.md | Extraction rules, @ViewBuilder, container views, MARK structure, body splitting |
| references/performance.md | Lazy stacks, identity stability, body purity, formatters, invalidation storms |
| references/animations.md | Implicit/explicit, transitions, phase/keyframe, Liquid Glass, matchedGeometryEffect |
| references/components.md | TabView, Form, Grid, ScrollView, theming, controls, haptics, loading states |
| references/accessibility-in-views.md | Labels, traits, Dynamic Type, @ScaledMetric, grouping, VoiceOver focus |
| references/latest-apis.md | Deprecated→modern transitions by iOS version, FoundationModels, Charts 3D, toolbars |
| workflows/build-view.md | Step-by-step: requirements → state model → hierarchy → compose → accessibility → verify |
| workflows/review-view.md | Checklist-driven review: state → body size → accessibility → performance → report |
</reference_index>

<canonical_terminology>
## Terminology

- **@Observable** (not: ObservableObject for new iOS 17+ code)
- **@State** (not: @StateObject with @Observable)
- **NavigationStack** (not: NavigationView — deprecated)
- **#Preview** (not: PreviewProvider — deprecated)
- **.animation(_:value:)** (not: .animation(_:) without value — deprecated)
- **modifier** (not: method when describing view modifiers)
- **body** (not: render/build when describing view body)
- **view** (not: widget, component, element)
</canonical_terminology>
