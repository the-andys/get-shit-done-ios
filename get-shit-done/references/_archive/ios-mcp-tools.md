# Xcode MCP Tools Reference

Reference for Xcode MCP (Model Context Protocol) tools available to agents during iOS development. When MCP tools are available, agents should prefer them over CLI commands. When unavailable, CLI fallback continues to work.

Strategy: **recommend, don't require** — MCP tools are preferred but never mandatory.

## Three-Layer Architecture

| Layer | Server | Tools | Source |
|-------|--------|-------|--------|
| **Apple Xcode MCP** | `xcode-tools` | ~20 tools | Ships with Xcode 26.3+ via `xcrun mcpbridge` |
| **XcodeBuildMCP** | `XcodeBuildMCP` | ~76 tools | Community (Sentry) — simulator, debugging, advanced build |
| **CLI Fallback** | none | unlimited | `xcodebuild`, `xcrun simctl`, `swift test` — always available |

Agents check tool availability at session start. If a tool call fails with "unknown tool", fall back to CLI.

---

## Apple Xcode MCP Tools (`xcode-tools`)

### Validation Hierarchy

Use these tools in order of increasing cost. Prefer the cheapest validation that catches the issue.

1. **`XcodeRefreshCodeIssuesInFile`** — Fastest. Re-validates a single file for compiler errors and warnings without building. Use after every file edit.
2. **`ExecuteSnippet`** — Medium. Runs a Swift code snippet in an ephemeral context. Use to validate logic, test a function signature, or confirm API availability.
3. **`BuildProject`** — Slowest. Full project build. Use only after all file-level validations pass, or when you need to verify cross-file dependencies.

```
Preferred flow:
  Edit file → XcodeRefreshCodeIssuesInFile → fix errors → repeat
  All files clean → BuildProject once
```

### File Operations

These tools let Xcode manage project structure automatically. When you use `XcodeWrite` or `XcodeUpdate` to create or modify files, Xcode handles `pbxproj` updates — no manual project file manipulation needed.

| Tool | Purpose |
|------|---------|
| **`XcodeRead`** | Read file contents from the project |
| **`XcodeWrite`** | Create or overwrite a file — Xcode adds it to the project automatically |
| **`XcodeUpdate`** | Apply targeted edits to an existing file |
| **`XcodeGlob`** | Find files matching a glob pattern within the project |
| **`XcodeGrep`** | Search file contents with regex |
| **`XcodeLS`** | List directory contents |
| **`XcodeMakeDir`** | Create directories |
| **`XcodeRM`** | Remove files — Xcode removes them from the project automatically |
| **`XcodeMV`** | Move or rename files — Xcode updates references automatically |

**Why this matters:** Direct `pbxproj` manipulation is fragile and the source of recurring merge conflicts and build failures. When MCP file operations are available, they eliminate this entire class of problems.

### Documentation

| Tool | Purpose |
|------|---------|
| **`DocumentationSearch`** | Search Apple developer documentation by keyword or symbol name. Returns framework docs, API references, and usage guidance. |

Use `DocumentationSearch` before implementing unfamiliar APIs. It returns current documentation, which may differ from training data.

### Testing

| Tool | Purpose |
|------|---------|
| **`RunAllTests`** | Execute the full test suite for the active scheme |
| **`RunSomeTests`** | Run specific test targets, suites, or individual test methods |
| **`GetTestList`** | List available test targets and test methods without running them |

### Preview Validation

| Tool | Purpose |
|------|---------|
| **`RenderPreview`** | Render a SwiftUI `#Preview` and return an image. Use to verify layout, styling, and content without launching a simulator. |

Use `RenderPreview` after modifying SwiftUI views. Catches layout issues earlier than a full build-and-run cycle.

---

## XcodeBuildMCP Tools

Extended tools from the community XcodeBuildMCP server. Covers simulator management, debugging, and advanced build operations not available in Apple's `xcode-tools`.

### Simulator Management

| Tool | Purpose |
|------|---------|
| **`list_sims`** | List available simulators with their state (booted, shutdown) |
| **`boot_sim`** | Boot a specific simulator by name or UDID |
| **`screenshot`** | Capture a screenshot from a running simulator |

### Testing (Extended)

| Tool | Purpose |
|------|---------|
| **`test_sim`** | Run tests on a specific simulator with detailed output |
| **`test_device`** | Run tests on a connected physical device |

### Debugging

| Tool | Purpose |
|------|---------|
| **`debug_attach_sim`** | Attach debugger to a running app in the simulator |
| **`debug_stack`** | Capture stack traces from the attached process |
| **`debug_variables`** | Inspect variable values in the current debug context |

---

## CLI Fallback

When MCP tools are not available, all operations fall back to standard CLI commands.

| MCP Tool | CLI Equivalent |
|----------|---------------|
| `BuildProject` | `xcodebuild build -scheme <Scheme> -destination 'platform=iOS Simulator,name=iPhone 16'` |
| `RunAllTests` | `xcodebuild test -scheme <Scheme> -destination 'platform=iOS Simulator,name=iPhone 16'` |
| `RunSomeTests` | `xcodebuild test -scheme <Scheme> -only-testing:<Target>/<Suite>/<Test>` |
| `XcodeRefreshCodeIssuesInFile` | `swiftc -typecheck <file>` (partial — no project context) |
| `ExecuteSnippet` | `swift -e '<code>'` or a temp `.swift` file |
| `DocumentationSearch` | No direct equivalent — use training data knowledge |
| `RenderPreview` | No direct equivalent — build and run in simulator |
| `list_sims` | `xcrun simctl list devices` |
| `boot_sim` | `xcrun simctl boot <UDID>` |
| `screenshot` | `xcrun simctl io <UDID> screenshot <path>` |

For file operations without MCP, use standard `Read`, `Write`, `Edit`, `Glob`, `Grep` tools. Note that creating new Swift files without MCP requires manual addition to the Xcode project — use `xcodebuild` or open Xcode to add files to targets.

---

## Setup Reference

### Apple Xcode MCP (requires Xcode 26.3+)

```bash
claude mcp add xcode-tools -- xcrun mcpbridge
```

### XcodeBuildMCP

```bash
claude mcp add XcodeBuildMCP -- npx -y xcodebuildmcp@latest
```

Both servers are compatible with Claude Code and Codex. Add them to `.claude/settings.json` for project-level configuration, or `~/.claude/settings.json` for global.

### Verifying Setup

After adding MCP servers, verify tools are available:

```bash
# In a Claude Code session, the agent can check by calling a simple tool
# If the tool returns a result, the server is configured correctly
# If it fails with "unknown tool", the server is not available
```

No additional configuration or API keys required for either server.
