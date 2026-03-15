---
name: app-architecture
description: Project structure, naming conventions, MVVM, DI, error handling, framework selection, anti-patterns, deployment targets
---

<essential_principles>
## How This Skill Works

1. **MVVM with SwiftUI** is the default architecture. Views are declarative, ViewModels are `@Observable`, dependencies flow through `@Environment`.
2. **Composition over inheritance.** Build features from small, focused types. One ViewModel per feature/screen.
3. **Native first.** Always prefer Apple frameworks over third-party. The framework-selection reference has the full hierarchy.
4. **Conventions are enforced.** Naming, file structure, MARK patterns, and localization rules are mandatory — not suggestions.
5. **Deployment gating.** iOS 17 is the base. iOS 18+ and iOS 26+ APIs require availability checks.
</essential_principles>

<intake>
## What do you need?

1. Start a new project (folder structure, entry point, DI setup)
2. Review naming conventions or code structure
3. Choose the right framework for a task
4. Fix an anti-pattern or code smell
5. Handle errors properly
6. Understand deployment target gating
</intake>

<routing>
| Response | Reference |
|----------|-----------|
| 1, "new project", "structure", "MVVM", "DI", "modularization" | `references/code-structure.md` |
| 2, "naming", "conventions", "file names", "MARK", "casing" | `references/naming-conventions.md` |
| 3, "framework", "which library", "native vs third-party", "choose" | `references/framework-selection.md` |
| 4, "anti-pattern", "code smell", "force unwrap", "God ViewModel" | `references/anti-patterns.md` |
| 5, "error", "try catch", "guard", "throws", "alert" | `references/error-handling.md` |
| 6, "deployment", "iOS 18", "iOS 26", "availability", "#available" | `references/deployment-targets.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/naming-conventions.md | Naming types, properties, files, enums, argument labels |
| references/code-structure.md | File organization, MARK patterns, MVVM folders, DI, modularization, localization |
| references/error-handling.md | Typed errors, try-catch with async, guard, Result, SwiftUI alerts |
| references/framework-selection.md | Which framework to use per domain (UI, data, networking, concurrency, etc.) |
| references/anti-patterns.md | Force unwraps, Combine in new code, PreviewProvider, God ViewModels, hardcoded strings |
| references/deployment-targets.md | iOS 17 base, iOS 18/26 conditional APIs, availability patterns, xcconfig |
</reference_index>

<canonical_terminology>
## Terminology

- **ViewModel** (not: Controller, Presenter, Interactor)
- **@Observable** (not: ObservableObject for new iOS 17+ code)
- **@Environment** (not: Singleton, ServiceLocator)
- **NavigationStack** (not: NavigationView — deprecated)
- **#Preview** (not: PreviewProvider — deprecated)
- **String(localized:)** (not: NSLocalizedString, hardcoded strings)
- **SwiftData** (not: Core Data for new projects)
- **async/await** (not: Combine for simple async, DispatchQueue)
</canonical_terminology>
