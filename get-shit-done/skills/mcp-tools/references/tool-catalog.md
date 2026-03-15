<overview>
Complete catalog of MCP tools for iOS development. Three-layer architecture: Apple Xcode MCP → XcodeBuildMCP → CLI fallback. Includes validation hierarchy, tool-by-tool reference, and setup instructions. Read when you need to find the right tool for a task. Related: workflows/debug-on-simulator.md (step-by-step debugging).
</overview>

## Three-Layer Architecture

| Layer | Server | Tools | Source |
|-------|--------|-------|--------|
| **Apple Xcode MCP** | `xcode-tools` | ~20 tools | Ships with Xcode 26.3+ via `xcrun mcpbridge` |
| **XcodeBuildMCP** | `XcodeBuildMCP` | ~76 tools | Community (Sentry) — simulator, debugging, advanced build |
| **CLI Fallback** | none | unlimited | `xcodebuild`, `xcrun simctl`, `swift test` — always available |

Agents check tool availability at session start. If a tool call fails with "unknown tool", fall back to CLI.

## Validation Hierarchy

Use in order of increasing cost. Prefer the cheapest validation that catches the issue.

```
Edit file → XcodeRefreshCodeIssuesInFile → fix errors → repeat
All files clean → BuildProject once
UI change → build + run + screenshot
```

| Level | Tool | Cost | When |
|-------|------|------|------|
| 1. Per-file | `mcp__xcode__XcodeRefreshCodeIssuesInFile` | Fastest (~2s) | After every file edit |
| 2. Snippet | `mcp__xcode__ExecuteSnippet` | Medium | Test logic, API availability |
| 3. Full build | `mcp__xcode__BuildProject` | Slowest | After all files clean, or cross-file deps |
| 4. Preview | `mcp__xcode__RenderPreview` | Medium | SwiftUI layout verification |

## Apple Xcode MCP Tools

### File Operations

| Tool | Purpose |
|------|---------|
| `mcp__xcode__XcodeRead` | Read file contents from the project |
| `mcp__xcode__XcodeWrite` | Create or overwrite a file — Xcode adds to project automatically |
| `mcp__xcode__XcodeUpdate` | Apply targeted edits to existing file |
| `mcp__xcode__XcodeGlob` | Find files matching glob pattern |
| `mcp__xcode__XcodeGrep` | Search file contents with regex |
| `mcp__xcode__XcodeLS` | List directory contents |
| `mcp__xcode__XcodeMakeDir` | Create directories |
| `mcp__xcode__XcodeRM` | Remove files — Xcode removes from project automatically |
| `mcp__xcode__XcodeMV` | Move/rename — Xcode updates references |

**Why prefer MCP file ops:** `XcodeWrite` and `XcodeRM` handle all `pbxproj` modifications automatically — target membership, build phases, group hierarchy. Eliminates manual pbxproj editing, which is the primary source of project file corruption.

### Validation

| Tool | Purpose |
|------|---------|
| `mcp__xcode__XcodeRefreshCodeIssuesInFile` | Re-validate single file for compiler errors without building |
| `mcp__xcode__ExecuteSnippet` | Run Swift snippet in ephemeral context |
| `mcp__xcode__BuildProject` | Full project build |
| `mcp__xcode__RenderPreview` | Render SwiftUI `#Preview` as image |

### Documentation

| Tool | Purpose |
|------|---------|
| `mcp__xcode__DocumentationSearch` | Search Apple developer docs by keyword or symbol |

Use `DocumentationSearch` before implementing unfamiliar APIs. Returns current documentation which may differ from training data. **Mandatory** for: Liquid Glass, FoundationModels, any iOS 26+ API.

### Testing

| Tool | Purpose |
|------|---------|
| `mcp__xcode__RunAllTests` | Execute full test suite for active scheme |
| `mcp__xcode__RunSomeTests` | Run specific targets, suites, or individual tests |
| `mcp__xcode__GetTestList` | List available test targets and methods |

## XcodeBuildMCP Tools

### Simulator Management

| Tool | Purpose |
|------|---------|
| `mcp__XcodeBuildMCP__list_sims` | List simulators with state (booted/shutdown) |
| `mcp__XcodeBuildMCP__boot_sim` | Boot a specific simulator |
| `mcp__XcodeBuildMCP__screenshot` | Capture screenshot from running simulator |

### Build & Run

| Tool | Purpose |
|------|---------|
| `mcp__XcodeBuildMCP__build_run_sim` | Build and run on simulator |
| `mcp__XcodeBuildMCP__launch_app_sim` | Launch already-built app |
| `mcp__XcodeBuildMCP__session-set-defaults` | Set project, scheme, simulator for session |
| `mcp__XcodeBuildMCP__get_sim_app_path` | Find app path on simulator |
| `mcp__XcodeBuildMCP__get_app_bundle_id` | Get bundle ID from app path |

### UI Interaction

| Tool | Purpose |
|------|---------|
| `mcp__XcodeBuildMCP__describe_ui` | Describe current UI elements (call before tapping) |
| `mcp__XcodeBuildMCP__tap` | Tap element by id, label, or coordinates |
| `mcp__XcodeBuildMCP__type_text` | Type text in focused field |
| `mcp__XcodeBuildMCP__gesture` | Scrolls, swipes, edge gestures |

### Debugging

| Tool | Purpose |
|------|---------|
| `mcp__XcodeBuildMCP__debug_attach_sim` | Attach debugger to running app |
| `mcp__XcodeBuildMCP__debug_stack` | Capture stack traces |
| `mcp__XcodeBuildMCP__debug_variables` | Inspect variable values |

### Logs

| Tool | Purpose |
|------|---------|
| `mcp__XcodeBuildMCP__start_sim_log_cap` | Start capturing logs (pass bundle ID) |
| `mcp__XcodeBuildMCP__stop_sim_log_cap` | Stop and return captured logs |

### Testing (Extended)

| Tool | Purpose |
|------|---------|
| `mcp__XcodeBuildMCP__test_sim` | Run tests on specific simulator |
| `mcp__XcodeBuildMCP__test_device` | Run tests on physical device |

## CLI Fallback

When MCP tools are unavailable:

| MCP Tool | CLI Equivalent |
|----------|---------------|
| `BuildProject` | `xcodebuild build -scheme <Scheme> -destination 'platform=iOS Simulator,name=iPhone 16'` |
| `RunAllTests` | `xcodebuild test -scheme <Scheme> -destination 'platform=iOS Simulator,name=iPhone 16'` |
| `RunSomeTests` | `xcodebuild test -scheme <Scheme> -only-testing:<Target>/<Suite>/<Test>` |
| `XcodeRefreshCodeIssuesInFile` | `swiftc -typecheck <file>` (partial — no project context) |
| `ExecuteSnippet` | `swift -e '<code>'` or temp `.swift` file |
| `DocumentationSearch` | No equivalent — use training data |
| `RenderPreview` | No equivalent — build and run in simulator |
| `list_sims` | `xcrun simctl list devices` |
| `boot_sim` | `xcrun simctl boot <UDID>` |
| `screenshot` | `xcrun simctl io <UDID> screenshot <path>` |

For file operations without MCP, use standard `Read`, `Write`, `Edit`, `Glob`, `Grep` tools.

## Setup

### Apple Xcode MCP (Xcode 26.3+)

```bash
claude mcp add xcode-tools -- xcrun mcpbridge
```

### XcodeBuildMCP

```bash
claude mcp add XcodeBuildMCP -- npx -y xcodebuildmcp@latest
```

Add to `.claude/settings.json` (project) or `~/.claude/settings.json` (global). No API keys required.
