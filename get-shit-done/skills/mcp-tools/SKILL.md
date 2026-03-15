---
name: mcp-tools
description: Xcode MCP tools, XcodeBuildMCP, simulator management, build validation, debug on simulator
---

<essential_principles>
## How This Skill Works

1. **Cheapest validation first.** Per-file check → snippet → full build. Never start with a full build when a file-level check can catch the error.
2. **Prefer MCP over CLI.** When MCP tools are available, use them. They manage pbxproj, provide faster feedback, and avoid manual project manipulation.
3. **Always verify visually for UI.** Tests verify correctness, screenshots verify appearance. For UI changes, build → run → screenshot.
4. **Strategy: recommend, don't require.** MCP tools are preferred but never mandatory. CLI fallback always works.
5. **Three-layer architecture.** Apple Xcode MCP → XcodeBuildMCP → CLI fallback. Check availability at session start.
</essential_principles>

<intake>
## What do you need?

1. Validate code changes (per-file, snippet, or full build)
2. Build and run on simulator
3. Run tests (all or specific)
4. Debug on simulator (UI inspection, logs, screenshots)
5. Set up MCP servers
6. Find the right tool for a task
</intake>

<routing>
| Response | Reference / Workflow |
|----------|---------------------|
| 1, "validate", "check", "build", "compile" | `references/tool-catalog.md` (validation hierarchy) |
| 2, "run", "simulator", "launch" | `workflows/debug-on-simulator.md` |
| 3, "test", "tests", "run tests" | `references/tool-catalog.md` (testing tools) |
| 4, "debug", "inspect", "screenshot", "logs" | `workflows/debug-on-simulator.md` |
| 5, "setup", "configure", "MCP" | `references/tool-catalog.md` (setup section) |
| 6, "which tool", "find tool" | `references/tool-catalog.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/tool-catalog.md | Full catalog of MCP tools, validation hierarchy, CLI fallback table, setup |
| workflows/debug-on-simulator.md | Step-by-step: build, run, interact, inspect UI, capture logs on simulator |
</reference_index>

<canonical_terminology>
## Terminology

- **Xcode MCP** (not: xcode-tools — that's the server name)
- **XcodeBuildMCP** (not: build MCP, sentry MCP)
- **validation hierarchy** (not: test pyramid — different concept)
- **per-file check** (not: lint — it's a compiler check)
- **CLI fallback** (not: manual mode)
</canonical_terminology>
