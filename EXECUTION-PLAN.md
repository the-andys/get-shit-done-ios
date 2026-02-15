# GSD iOS Adaptation — Execution Plan

> Read this file at the start of every session to know where to resume.

## Wave 0 — Setup

- [x] Create branch `feature/ios-adaptation`
- [x] Reset version to `0.1.0` in package.json
- [x] Create EXECUTION-PLAN.md (this file)
- [x] Update CLAUDE.md (language rules, status, execution plan reference)

## Wave 1 — Foundation (parallel, no dependencies)

- [x] Create `get-shit-done/references/ios-swift-guidelines.md` (CORNERSTONE)
- [x] Create `get-shit-done/references/ios-frameworks.md`
- [x] Create `get-shit-done/references/ios-testing.md`

## Wave 2 — Critical Agents (depends on Wave 1)

- [x] Adapt `agents/gsd-project-researcher.md` — iOS research mode
- [x] Adapt `agents/gsd-phase-researcher.md` — iOS implementation research
- [x] Adapt `agents/gsd-planner.md` — iOS examples, patterns, accessibility
- [x] Adapt `get-shit-done/templates/phase-prompt.md` — iOS plan template

## Wave 3 — Secondary Agents (depends on Wave 1)

- [x] Adapt `agents/gsd-executor.md` — iOS execution patterns
- [x] Adapt `agents/gsd-verifier.md` — iOS verification + accessibility
- [x] Adapt `agents/gsd-roadmapper.md` — iOS roadmap patterns
- [x] Adapt `agents/gsd-codebase-mapper.md` — iOS project analysis
- [x] Adapt `agents/gsd-debugger.md` — iOS debugging patterns

## Wave 4 — Templates, References and Workflows (depends on Waves 2-3)

- [ ] Adapt `get-shit-done/references/questioning.md` — iOS-specific questions
- [ ] Adapt `get-shit-done/references/tdd.md` — Swift Testing + XCTest
- [ ] Adapt `get-shit-done/references/verification-patterns.md` — iOS verification
- [ ] Adapt `get-shit-done/templates/codebase/stack.md` + other codebase templates
- [ ] Adapt `get-shit-done/workflows/verify-phase.md` — iOS verification workflow
- [ ] Adapt `get-shit-done/templates/verification-report.md` — iOS verification report examples
- [ ] Adapt `get-shit-done/workflows/plan-milestone-gaps.md` — iOS gap planning examples
- [ ] Adapt `get-shit-done/workflows/diagnose-issues.md` — iOS diagnosis examples

## Wave 5 — Complementary References and Workflows

- [ ] Create `get-shit-done/references/ios-app-lifecycle.md`
- [ ] Create `get-shit-done/references/ios-permissions.md`
- [ ] Adapt `get-shit-done/workflows/complete-milestone.md` — iOS milestone examples
- [ ] Adapt `get-shit-done/workflows/execute-plan.md` — iOS execution path examples
- [ ] Adapt `get-shit-done/workflows/add-todo.md` — iOS area references

## Version Tracking

| Wave | Version | Status |
|------|---------|--------|
| 0+1  | 0.1.0   | Complete |
| 2    | 0.2.0   | Complete |
| 3    | 0.3.0   | Complete |
| 4    | 0.4.0   | Pending |
| 5    | 0.5.0   | Pending |

## Verification

After all waves complete, run final validation:

```bash
grep -r "npm\|tsx\|jsx\|React\|webpack\|node_modules\|package\.json" agents/ get-shit-done/references/ get-shit-done/templates/phase-prompt.md
```

No web-specific references should remain in adapted files.
