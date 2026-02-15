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

- [x] Adapt `get-shit-done/references/questioning.md` — iOS-specific questions (clean, no changes needed)
- [x] Adapt `get-shit-done/references/tdd.md` — Swift Testing + XCTest (clean, no changes needed)
- [x] Adapt `get-shit-done/references/verification-patterns.md` — iOS verification
- [x] Adapt `get-shit-done/templates/codebase/stack.md` + other codebase templates
- [x] Adapt `get-shit-done/workflows/verify-phase.md` — iOS verification workflow
- [x] Adapt `get-shit-done/templates/verification-report.md` — iOS verification report examples
- [x] Adapt `get-shit-done/workflows/plan-milestone-gaps.md` — iOS gap planning examples
- [x] Adapt `get-shit-done/workflows/diagnose-issues.md` — iOS diagnosis examples

## Wave 5 — Complementary References and Workflows

- [x] Create `get-shit-done/references/ios-app-lifecycle.md`
- [x] Create `get-shit-done/references/ios-permissions.md`
- [x] Adapt `get-shit-done/workflows/complete-milestone.md` — iOS milestone examples
- [x] Adapt `get-shit-done/workflows/execute-plan.md` — iOS execution path examples
- [x] Adapt `get-shit-done/workflows/add-todo.md` — iOS area references

## Wave 6 — Missed Files (originally classified as 🟢 OK)

> **Discovery:** The original adaptation plan (02-gsd-ios-edition-plan.md) mapped these 3 files
> but classified them as "generic/universal" — assuming their logic was platform-agnostic.
> Post-Wave 5 audit revealed all 3 contain hardcoded web examples (tsx, npm, React, Prisma,
> fetch/axios, Next.js routing, Vercel, etc.). The structure IS generic, but the examples
> that teach agents what to look for are 100% web.
>
> **Lesson learned:** Classification of "OK/generic" should require a confirmatory grep
> before being accepted. Generic logic ≠ generic content.

- [x] Adapt `agents/gsd-plan-checker.md` — Replace web examples (Chat.tsx, LoginForm.tsx, Prisma, fetch) with iOS (MVVM, SwiftData, @Observable)
- [x] Adapt `agents/gsd-integration-checker.md` — Replace web grep patterns and flow examples with iOS (Services, ViewModels, Views)
- [x] Adapt `get-shit-done/references/checkpoints.md` — Replace npm/Next.js/Vercel examples with xcodebuild/Simulator/TestFlight/Fastlane

## Version Tracking

| Wave | Version | Status |
|------|---------|--------|
| 0+1  | 0.1.0   | Complete |
| 2    | 0.2.0   | Complete |
| 3    | 0.3.0   | Complete |
| 4    | 0.4.0   | Complete |
| 5    | 0.5.0   | Complete |
| 6    | 0.6.0   | Complete |

## Verification

After all waves complete, run final validation:

```bash
grep -r "npm\|tsx\|jsx\|React\|webpack\|node_modules\|package\.json" agents/ get-shit-done/references/ get-shit-done/templates/phase-prompt.md
```

No web-specific references should remain in adapted files.
