# iOS Conventions — Mandatory Enforcement Rules

Slim reference loaded every session. Contains ONLY rules that must be enforced regardless of task domain. For in-depth guidance on any topic, agents load built-in skills via `~/.claude/get-shit-done/skills/INDEX.md`.

## Validation Checklist

Run before declaring any task complete. Any unchecked item is a blocker.

**Code Quality**
- [ ] No compiler warnings
- [ ] No force unwraps without justification comment
- [ ] No `DispatchQueue.main.async` — use `@MainActor`
- [ ] No `PreviewProvider` — use `#Preview` macro
- [ ] No `NavigationView` — use `NavigationStack`
- [ ] No Combine for simple async — use async/await
- [ ] All `@State` properties are `private`
- [ ] View `body` under ~30 lines (extract subviews if longer)

**Accessibility**
- [ ] All interactive elements have `.accessibilityLabel`
- [ ] Decorative images use `.accessibilityHidden(true)`
- [ ] Text uses semantic font styles or `relativeTo:` for Dynamic Type
- [ ] Animations respect `accessibilityReduceMotion`
- [ ] Tap targets at least 44x44pt

**Architecture**
- [ ] One ViewModel per feature/screen (no God ViewModels)
- [ ] ViewModels use `@Observable` (not `ObservableObject`)
- [ ] Dependencies injected via `@Environment` (not singletons)
- [ ] Error types conform to `LocalizedError`

**Testing**
- [ ] New business logic has Swift Testing tests (`@Test`, `#expect`)
- [ ] Tests run and pass

**Localization**
- [ ] No hardcoded user-facing strings — use `String(localized:)` or `Localizable.xcstrings`

**iOS-Specific**
- [ ] iOS 17+ base, iOS 18+/26+ behind `#available` checks
- [ ] Native frameworks over third-party
- [ ] `#Preview` provided for every new View

## Anti-Patterns Summary

| Anti-Pattern | Correct Alternative |
|---|---|
| Force unwrap on runtime values | `guard let` / `if let` / throw |
| Combine for simple async | async/await |
| `PreviewProvider` protocol | `#Preview` macro |
| `DispatchQueue.main.async` | `@MainActor` |
| View body 100+ lines | Extract subviews (~30 line rule) |
| God ViewModel (all features) | Focused ViewModel per feature |
| Hardcoded user-facing strings | `String(localized:)` |
| `ObservableObject` + `@Published` | `@Observable` (iOS 17+) |
| `NavigationView` | `NavigationStack` |
| `onChange(of:perform:)` one-param | `onChange(of:) { old, new in }` |
| Singletons for DI | `@Environment` injection |

## Localization Enforcement

All user-facing text MUST use `String(localized:)`. No exceptions.

```swift
Text(String(localized: "welcome_title"))
Button(String(localized: "delete_action")) { deleteItem() }
```

Store translations in `Localizable.xcstrings`. Accessibility identifiers (`.accessibilityIdentifier`) are NOT user-facing and remain as plain strings.

## Preview Macro Enforcement

Always use `#Preview`. Never use the deprecated `PreviewProvider` protocol.

```swift
#Preview { ProfileView(user: .preview) }
#Preview("Editing Mode") { ProfileView(user: .preview, isEditing: true) }
```

## Accessibility Minimum

Every interactive element (button, toggle, slider, link) MUST have `.accessibilityLabel`. Decorative images MUST use `.accessibilityHidden(true)`. Text MUST use semantic font styles for Dynamic Type.
