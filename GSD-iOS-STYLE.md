# GSD-iOS-STYLE.md

How GSD:iOS is written. Read this before contributing to agents, references, or templates.

## Base Principles (from upstream GSD)

These apply unchanged from upstream:

1. XML for semantic structure, Markdown for content
2. @-references are lazy loading signals
3. Commands delegate to workflows
4. Progressive disclosure hierarchy
5. Imperative, brief, technical — no filler, no sycophancy
6. Solo developer + Claude — no enterprise patterns
7. Context size as quality constraint — split aggressively
8. Temporal language banned in implementation docs — current state only
9. Plans ARE prompts — executable, not documents
10. Atomic commits — Git history as context source
11. AskUserQuestion for all exploration — always options
12. Checkpoints post-automation — automate first, verify after
13. Deviation rules are automatic — no permission for bugs/critical
14. Depth controls compression — derive from actual work
15. TDD gets dedicated plans — cycle too heavy to embed

Full detail at upstream GSD-STYLE.md (experimental branch).

## iOS Layer — What's Different

### Platform Identity

Every iOS-specific file teaches Claude it's building for:

- Language: Swift (not TypeScript, JavaScript, Python)
- UI Framework: SwiftUI (not React, Vue, HTML/CSS)
- Data: SwiftData, CoreData, CloudKit (not Prisma, SQL, MongoDB)
- Testing: XCTest, XCUITest (not Jest, Vitest, Cypress)
- Distribution: App Store / TestFlight (not Vercel, AWS, Heroku)
- IDE: Xcode (not VS Code, WebStorm)

### File Path Conventions

```
Sources/                  <- Swift source files (NOT src/)
Sources/Features/Auth/    <- feature modules
Sources/Shared/           <- shared utilities
Tests/                    <- XCTest files (NOT __tests__/)
Resources/                <- assets, localizations
```

### Swift Naming Conventions

- Types: `PascalCase` (UserProfileView, AuthService)
- Functions/properties: `camelCase` (fetchUser(), isLoading)
- Constants: `camelCase` (maxRetryCount)
- Files: match the primary type they contain (UserProfileView.swift)

### Current iOS Patterns (iOS 17+)

Use these — not deprecated equivalents:

| Use | Not |
|-----|-----|
| `@Observable` | `ObservableObject` + `@Published` |
| `SwiftData` | CoreData (unless existing project) |
| `async/await` | Completion handlers |
| `NavigationStack` | `NavigationView` |
| `.searchable()` | Custom search implementations |
| `#Preview` macro | `PreviewProvider` |

### Accessibility — Always Included

Not an afterthought. Every UI component includes:

- `.accessibilityLabel()` for non-obvious elements
- `.accessibilityHint()` for interactive elements
- Dynamic Type support (`@ScaledMetric`, avoid fixed sizes)
- VoiceOver navigation order when non-obvious

### Localization — Always Included

- `String(localized:)` for all user-facing strings
- No hardcoded string literals in UI
- `LocalizedStringResource` for string catalogs

## Agent File Conventions

iOS agents follow the same XML structure as upstream but with iOS context:

```xml
<task type="auto">
  <n>Task N: Create UserProfileView</n>
  <files>Sources/Features/Profile/UserProfileView.swift</files>
  <action>
    SwiftUI View using @Observable UserProfileViewModel.
    Display name, avatar, stats in List with Section headers.
    Add .accessibilityLabel() to avatar image.
    Support Dynamic Type — avoid fixed font sizes.
  </action>
  <verify>
    xcodebuild test -scheme AppName
      -destination 'platform=iOS Simulator,name=iPhone 16'
      -only-testing:AppNameTests/UserProfileTests
  </verify>
  <done>UserProfileView renders profile data, VoiceOver reads name and stats correctly</done>
</task>
```

## What to Never Write

In iOS-specific files, these references are banned:

- `npm install`, `yarn add`, `package.json` (dependency management)
- `src/`, `pages/`, `app/`, `components/` (web paths)
- React, Vue, Next.js, Remix, Vite (web frameworks)
- `useState`, `useEffect`, `props` (React hooks)
- Prisma, Supabase, Firebase web SDK (web backends)
- `localhost:3000`, API routes, Express (web servers)
- TypeScript, JavaScript patterns (wrong language)

## Temporal Language

Banned in implementation docs. Same as upstream.

DON'T: "We changed X to Y", "Previously", "No longer", "Instead of"

DO: Describe current state only.

Exception: CHANGELOG.md, MIGRATION.md, git commits.

## Commit Convention

Same as upstream format. iOS fork scopes:

| Scope | Use |
|-------|-----|
| `agents` | GSD agent files |
| `references` | iOS reference docs |
| `templates` | Plan/summary templates |
| `workflows` | GSD workflow files |
| `commands` | Slash command files |
| `installer` | bin/install.js |
| `tools` | gsd-tools.cjs |
| `sync` | Upstream sync work |
| `ci` | GitHub Actions |
| `branding` | Logo, README, assets |
