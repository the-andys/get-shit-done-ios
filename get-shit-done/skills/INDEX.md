# Built-in iOS Skills

Lightweight discovery index. Agents read this file to identify which skills to load for the current task.

| Skill | Load When Task Involves |
|-------|------------------------|
| app-architecture | Project structure, naming, DI, MVVM, modularization, conventions, error handling, framework selection |
| swiftui | Views, UI, state, navigation, animations, layout, sheets, Liquid Glass, components |
| swift-concurrency | async/await, actors, Sendable, tasks, Swift 6, data races, @MainActor |
| swiftdata | @Model, persistence, predicates, CloudKit, migrations, class inheritance |
| swift-testing | Tests, mocking, TDD, XCTest migration, parameterized, test CLI commands |
| accessibility | VoiceOver, Dynamic Type, contrast, assistive tech, WCAG, Assistive Access |
| ux-writing | Interface text, alerts, errors, empty states, button labels, voice & tone |
| performance | Slow rendering, janky scrolling, Instruments, layout thrash, identity stability |
| networking | URLSession, API clients, REST, JSON decoding, WebView, error handling |
| mcp-tools | Xcode MCP, XcodeBuildMCP, simulator, build validation, debug on simulator |
| security-privacy | Keychain, permissions, Info.plist, CryptoKit, data protection (Phase 3) |
| app-lifecycle | Scene phases, background tasks, push notifications, deep links (Phase 3) |

## Load Process

1. Read this INDEX to identify 1-3 relevant skills for the current task
2. Read `SKILL.md` for each relevant skill (~5KB each — lightweight routers)
3. Follow SKILL.md routing to load specific `references/*.md` as needed
4. If skill has `workflows/`, follow the applicable workflow for the task type
5. Do NOT load all skills — load only those relevant to the current task

## Override Rules

Project-level skills (`.claude/skills/` or `.agents/skills/`) override built-in skills for the same domain. Check project directories first, then fall back to built-in.
