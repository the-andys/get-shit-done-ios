<overview>
Instruments profiling for SwiftUI: SwiftUI template, Time Profiler, capture workflow, reading the timeline, and identifying hotspots. Use after code review when the cause isn't obvious from reading code. Related: code-review.md (code-level smells), remediation.md (fix patterns).
</overview>

## When to Profile

Profile when:
- Code review didn't reveal obvious smells
- Performance regression after a change
- Need before/after metrics for a fix
- User reports jank on specific device/OS

**Always profile on Release builds** — Debug builds have significant overhead that distorts results.

## SwiftUI Instruments Template

Xcode > Open Developer Tool > Instruments > SwiftUI template

Shows:
- **View Body** — how often each view's body is evaluated
- **View Properties** — which properties triggered the evaluation
- **Core Animation Commits** — frame rendering cost

## Time Profiler

For CPU-bound performance issues:

1. Select Time Profiler instrument
2. Record the problematic interaction
3. Look for:
   - Functions with high "Self Time" (the hotspot)
   - Deep call stacks during scroll or animation
   - Main thread work that should be background

## Capture Workflow

1. **Prepare:** Build in Release mode, connect target device
2. **Record:** Start Instruments, perform the exact interaction that's slow
3. **Stop:** End recording after the interaction completes
4. **Analyze:** Find the hotspots (see below)
5. **Fix:** Apply remediation from `remediation.md`
6. **Verify:** Profile again to confirm improvement

## Reading the Timeline

### View Body Track

| Pattern | Meaning |
|---------|---------|
| Many thin bars clustered | Invalidation storm — views recomputing frequently |
| Single thick bar | One expensive body evaluation |
| Bars during scroll | Scroll-driven invalidation |

### What to Look For

- **High body count** on a single view → check dependencies (what's triggering recompute?)
- **Body count matches scroll position** → view identity is unstable (see code-review.md)
- **Long body duration** → heavy computation in body (formatters, sorting, filtering)
- **Main thread blocked** → synchronous work (network, disk) on main thread

## Allocations Instrument

For memory-related performance:

- **Growth during scroll** → views not being recycled (use LazyVStack)
- **Many small allocations** → object creation in body
- **Retain cycles** → Xcode Memory Graph debugger is better for this

## Debugging Body Evaluations

In code (remove after debugging):

```swift
var body: some View {
    let _ = Self._printChanges()  // Console: "ProfileView: _isLoading changed."
    content
}
```

`Self._logChanges()` uses unified logging instead of print.

## Metrics Table Format

When reporting performance findings:

| Metric | Before | After | Target |
|--------|--------|-------|--------|
| Body evaluations (scroll 100 items) | 847 | 102 | <200 |
| Frame drops (60fps target) | 23 | 2 | 0 |
| Time in body (ms, p99) | 12ms | 2ms | <4ms |
| Memory growth (scroll) | +45MB | +3MB | <10MB |
