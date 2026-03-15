---
name: performance
description: SwiftUI performance — code review smells, Instruments profiling, remediation patterns, identity stability, body purity
---

<essential_principles>
## How This Skill Works

1. **Measure before optimizing.** Profile with Instruments first. Don't guess where the bottleneck is.
2. **Cheapest fix first.** Narrow state scope and stabilize identity before reaching for `equatable()` or custom layouts.
3. **Body purity is non-negotiable.** `body` is a pure function of state. No object creation, no heavy computation, no side effects.
4. **~30-line rule.** View bodies over ~30 lines are a performance smell — extract subviews.
5. **Code review catches most issues.** Most SwiftUI performance problems are visible in the code without profiling.
</essential_principles>

<intake>
## What do you need?

1. Diagnose slow UI or janky scrolling
2. Review code for performance smells
3. Profile with Instruments
4. Optimize images or large lists
5. Fix invalidation storms
</intake>

<routing>
| Response | Reference |
|----------|-----------|
| 1, "slow", "janky", "laggy", "scroll" | `references/code-review.md` then `references/instruments.md` |
| 2, "review", "smell", "check" | `references/code-review.md` |
| 3, "Instruments", "profile", "Time Profiler" | `references/instruments.md` |
| 4, "image", "list", "lazy", "downsample" | `references/remediation.md` |
| 5, "invalidation", "recompute", "identity" | `references/remediation.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/code-review.md | Code smells: invalidation storms, unstable identity, heavy body, formatters, broad dependencies |
| references/instruments.md | Instruments profiling: SwiftUI template, Time Profiler, capture workflow, reading timeline |
| references/remediation.md | Fix patterns: narrow state, stabilize identity, precompute, downsample, LazyVStack, InlineArray |
</reference_index>

<canonical_terminology>
## Terminology

- **invalidation** (not: re-render — SwiftUI invalidates, then re-evaluates body)
- **identity stability** (not: key stability — SwiftUI uses identity, not keys)
- **body purity** (not: render purity — body is the term)
- **Instruments** (not: profiler generically)
</canonical_terminology>
