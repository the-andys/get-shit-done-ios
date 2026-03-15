---
name: swift-concurrency
description: async/await, actors, Sendable, tasks, Swift 6/6.2 migration, data races, @MainActor, structured concurrency
---

<essential_principles>
## How This Skill Works

1. **Identify the isolation boundary FIRST.** Before proposing any fix, determine: @MainActor? Custom actor? Nonisolated? This drives every decision.
2. **Structured concurrency by default.** `async let` and `TaskGroup` over unstructured `Task {}`. Use `Task.detached` only with documented justification.
3. **Sendable by default for new code.** Design types for concurrency upfront. Value types are your friend.
4. **Actors for shared mutable state.** Not `DispatchQueue`, not locks (unless `Mutex` on iOS 18+). Actors serialize access automatically.
5. **Swift 6.2 changes the defaults.** Nonisolated async functions now inherit caller's isolation. `@concurrent` replaces `Task.detached` for most offloading.
</essential_principles>

<intake>
## What do you need?

1. Diagnose a concurrency compiler error
2. Migrate to Swift 6 / Swift 6.2
3. Design async architecture (actors, tasks, data flow)
4. Quick fix a specific error
5. Understand Swift 6.2 changes (@concurrent, default isolation)
6. Write concurrent tests
7. Understand task lifecycle (cancellation, priorities, groups)
</intake>

<routing>
| Response | Reference / Workflow |
|----------|---------------------|
| 1, "error", "diagnose", "compiler", "data race" | `workflows/fix-concurrency-error.md` |
| 2, "migrate", "Swift 6", "strict concurrency" | `references/migration.md` |
| 3, "architecture", "actor", "design", "isolation" | `references/actors.md` |
| 4, "quick fix", "@MainActor", "Sendable" | `workflows/fix-concurrency-error.md` |
| 5, "Swift 6.2", "@concurrent", "default isolation" | `references/swift-6-2.md` |
| 6, "test", "async test", "flaky" | `references/testing.md` |
| 7, "Task", "cancel", "group", "priority", "async let" | `references/tasks.md` |
| "async", "await", "basics" | `references/async-await.md` |
| "Sendable", "sending", "region", "@unchecked" | `references/sendable.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/async-await.md | Basic patterns, URLSession async, structured concurrency, continuation bridging |
| references/actors.md | @MainActor, custom actors, reentrancy, Mutex, isolation boundaries, custom executors |
| references/sendable.md | Value/reference types, @unchecked, region-based isolation, sending keyword, @Sendable |
| references/tasks.md | Task lifecycle, cancellation, priorities, TaskGroup, async let, detached tasks |
| references/migration.md | Swift 5→6 strategy, strict concurrency settings, closure conversion, @preconcurrency |
| references/swift-6-2.md | Default actor isolation, @concurrent, isolated conformances, migration settings |
| references/testing.md | Async test patterns, actor testing, concurrency-aware mocking, @MainActor test isolation |
| workflows/fix-concurrency-error.md | Triage: read error → identify boundary → quick fix OR escalate → verify |
</reference_index>

<canonical_terminology>
## Terminology

- **@MainActor** (not: DispatchQueue.main)
- **actor** (not: DispatchQueue, NSLock for shared state)
- **async/await** (not: completion handlers, Combine for async)
- **structured concurrency** (not: fire-and-forget tasks)
- **@concurrent** (not: Task.detached for offloading in Swift 6.2)
- **Sendable** (not: thread-safe by convention)
- **isolation boundary** (not: thread boundary)
</canonical_terminology>
