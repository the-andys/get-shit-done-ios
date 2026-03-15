<overview>
Step-by-step workflow for building, running, and debugging an iOS app on a simulator using MCP tools. Covers the full cycle: discover simulator → set defaults → build → run → interact → inspect UI → capture logs. Related: references/tool-catalog.md (full tool catalog).
</overview>

<required_reading>
Read `references/tool-catalog.md` first to understand available tools and their purposes.
</required_reading>

<process>
## Workflow: Debug on Simulator

### Step 1 — Discover the booted simulator

```
Call: mcp__XcodeBuildMCP__list_sims
```

Select the simulator with state `Booted`. If none are booted, ask the user which simulator to boot, then:

```
Call: mcp__XcodeBuildMCP__boot_sim
  simulatorName: "iPhone 16 Pro"
```

### Step 2 — Set session defaults

```
Call: mcp__XcodeBuildMCP__session-set-defaults
  projectPath: "<path to .xcodeproj>"   # or workspacePath for .xcworkspace
  scheme: "<AppScheme>"
  simulatorId: "<UDID from step 1>"
  configuration: "Debug"
```

### Step 3 — Build and run

```
Call: mcp__XcodeBuildMCP__build_run_sim
```

If the app is already built and only a launch is needed:

```
Call: mcp__XcodeBuildMCP__launch_app_sim
```

If bundle ID is unknown:
1. `mcp__XcodeBuildMCP__get_sim_app_path`
2. `mcp__XcodeBuildMCP__get_app_bundle_id`

### Step 4 — Interact with the UI

Always describe the UI before interacting:

```
Call: mcp__XcodeBuildMCP__describe_ui
```

Then tap, type, or gesture:

```
Call: mcp__XcodeBuildMCP__tap
  label: "Add Item"      # prefer label or id over coordinates

Call: mcp__XcodeBuildMCP__type_text
  text: "New task name"

Call: mcp__XcodeBuildMCP__gesture
  gesture: "scroll_down"
```

### Step 5 — Visual verification

```
Call: mcp__XcodeBuildMCP__screenshot
```

Compare against expected layout. Report findings to user.

### Step 6 — Capture logs

Start log capture before reproducing the issue:

```
Call: mcp__XcodeBuildMCP__start_sim_log_cap
  bundleId: "com.example.myapp"
```

Reproduce the issue, then stop and analyze:

```
Call: mcp__XcodeBuildMCP__stop_sim_log_cap
```

Summarize important lines: errors, warnings, unexpected state changes.

### Step 7 — Debug (if needed)

For deeper investigation:

```
Call: mcp__XcodeBuildMCP__debug_attach_sim
Call: mcp__XcodeBuildMCP__debug_stack
Call: mcp__XcodeBuildMCP__debug_variables
```
</process>

<anti_patterns>
## Common Mistakes

- **Building without per-file check first** — Use `XcodeRefreshCodeIssuesInFile` after edits, `BuildProject` only when needed
- **Tapping without describe_ui** — Always describe UI state before interaction to avoid hitting wrong elements
- **Using coordinates instead of labels** — Prefer `id` or `label` for tap targets; coordinates break across device sizes
- **Forgetting to set session defaults** — Must set project/scheme/simulator before build_run_sim
- **Not capturing logs before reproducing** — Start log capture BEFORE the action that triggers the bug
</anti_patterns>

<success_criteria>
## Done When

- App builds and launches on simulator without errors
- Target behavior is reproduced or verified via UI interaction
- Screenshots confirm visual correctness
- Relevant logs captured and analyzed
- Findings reported to user with clear next steps
</success_criteria>
