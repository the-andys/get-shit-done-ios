<overview>
Triage workflow for diagnosing and fixing Swift concurrency compiler errors. Covers the full cycle: read error → identify isolation boundary → determine quick fix vs escalation → apply fix → verify. Related: All swift-concurrency references for deep dives on specific topics.
</overview>

<required_reading>
Read the SKILL.md for the reference_index — you may need to load a specific reference based on what the triage reveals.
</required_reading>

<process>
## Workflow: Fix Concurrency Error

### Step 1 — Capture the Error

Read the exact compiler diagnostic. Key phrases to look for:

| Error Contains | Likely Domain |
|----------------|--------------|
| "Main actor-isolated" | @MainActor boundary |
| "Sendable" / "non-Sendable" | Type safety |
| "actor-isolated" / "cannot be used from" | Actor isolation |
| "data race" / "concurrent access" | Shared mutable state |
| "task-isolated" | Task context |

### Step 2 — Identify Isolation Boundary

Before proposing ANY fix, determine:
- What is the current isolation? (@MainActor, custom actor, nonisolated?)
- What isolation does the target require?
- What Swift version / strict concurrency level?

### Step 3 — Quick Fix or Escalate?

**Quick Fix** (localized, < 5 min):
- Error is in a single file
- Fix doesn't change public API
- No architectural implications

**Escalate** (needs design):
- Error involves public API signatures
- Fix requires moving code between isolation domains
- Multiple files affected
- Requires @preconcurrency or @unchecked Sendable

### Step 4 — Apply Common Quick Fixes

| Error | Quick Fix |
|-------|-----------|
| "Main actor-isolated...cannot be used from nonisolated" | Make caller `@MainActor` or use `await MainActor.run {}` |
| "Actor-isolated type does not conform to protocol" | Add isolated conformance: `extension Foo: @MainActor Protocol` (Swift 6.2) |
| "Sending non-Sendable type...data races" | Confine access inside actor OR convert to value type |
| "Capture of non-Sendable in @Sendable closure" | Copy to local `let` before capture, or make type Sendable |
| "Static property is not concurrency-safe" | Add `@MainActor` to containing type or make property `nonisolated(unsafe)` |
| "Call to nonisolated async from @MainActor" | Mark function `@MainActor` or accept the hop |

### Step 5 — Verify

```
Build → Check error resolved → No new errors introduced → Rebuild full project
```

**If new errors appear:** You've shifted the boundary. Step back, reassess isolation domains.

### Step 6 — Document

For @preconcurrency, @unchecked Sendable, or nonisolated(unsafe):
- Add a comment explaining the safety invariant
- File a migration ticket if temporary

</process>

<anti_patterns>
## Common Mistakes

- **Blindly adding @MainActor everywhere** — adds overhead, may not be correct
- **Using @unchecked Sendable to silence warnings** — hides real data races
- **Fixing symptoms not causes** — if the same error keeps moving, redesign isolation
- **Batching fixes** — one change at a time, rebuild after each
- **Ignoring Swift version** — Swift 6.2 fixes many errors that require workarounds in 6.0
</anti_patterns>

<success_criteria>
## Done When

- Original error is resolved
- No new concurrency warnings introduced
- Build succeeds with current strict concurrency level
- Any escape hatches (@unchecked, @preconcurrency) are documented with safety invariant
</success_criteria>
