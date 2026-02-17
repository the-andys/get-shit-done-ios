# GSD iOS Fork — Project State

## Project Reference

**Core Value:** Platform port of the GSD meta-prompting system for iOS native development with Swift/SwiftUI/SwiftData/MVVM, preserving upstream philosophy and quality.

**Current Focus:** Spec 004 — README + USER-GUIDE adapted for iOS

**Fork Version:** 0.4.0
**Upstream:** 1.20.0 (fully synced — infra + content)

---

## Current Position

**Phase:** Post-port maintenance & evolution
**Active Spec:** 004 — README + USER-GUIDE adapted for iOS
**Status:** Spec 003 complete, ready for Spec 004

---

## Port (Complete)

Systematic platform port from web to iOS, waves 0-6.
Version: 0.1.0 | Base: upstream 1.18.0 | See `.dev/port/` for full history.

| Wave | Summary |
|------|---------|
| 0 | Setup: fork, branch, EXECUTION-PLAN, version reset |
| 1 | Foundation: ios-swift-guidelines, ios-frameworks, ios-testing |
| 2 | Critical agents: project-researcher, phase-researcher, planner, phase-prompt |
| 3 | Secondary agents: executor, verifier, roadmapper, codebase-mapper, debugger |
| 4 | Templates, references, workflows |
| 5 | Complementary: ios-app-lifecycle, ios-permissions, remaining workflows |
| 6 | Missed files: plan-checker, integration-checker, checkpoints |

---

## Active Specs

| # | Type | Description | Status |
|---|------|-------------|--------|
| 001 | changed | Upstream infra sync 1.18.0 → 1.20.0 | **done** |
| 002 | changed | Content .cjs rename (41 files) + cherry-picks (executor, verifier) | **done** |
| 003 | changed | Layer 2 content sync — 19 commits, ~35 files, ~650 lines (upstream 1.18.0→1.20.0) | **done** |

## Planned Specs

| # | Type | Description | Depends On |
|---|------|-------------|------------|
| 004 | changed | README + USER-GUIDE adapted for iOS | 003 |
| 005 | added | Auto-advance pipeline evaluation | 004 |

---

## Key Decisions

| # | Decision | Rationale | Phase |
|---|----------|-----------|-------|
| D01 | Condensed Apple docs + dynamic research | Respects GSD context engineering philosophy | Port W2 |
| D02 | Hard fork, 100% iOS only | Simpler than dual-target, more focused | Port W0 |
| D03 | iOS 17+ default, configurable | Stable modern base, conditional for 18+/26+ | Port W2 |
| D04 | English for GitHub, PT-BR for chat | International audience + local workflow | Port W0 |
| D05 | .cjs rename: infra now, content in Layer 2 | Zero risk separation of concerns | Spec 001 |
| D06 | Brave Search = valuable for iOS | Web search for Apple docs, not web-dev feature | Spec 001 |
| D07 | cleanup.md + health.md = generic infra | Promoted from content to infra after review | Spec 001 |

See `.dev/port/SUMMARY.md` for full context on port decisions.

---

## Active TODOs

### Spec 001 (done)
- [x] Execute infra sync (9 tasks: rename, replace, copy, merge)
- [x] Run final audit (10 verification checks — all PASS)
- [x] Create SUMMARY.md

### Spec 002 (done)
- [x] .cjs rename in 41 content files (8 agents, 26 workflows, 5 references, 2 commands)
- [x] Cherry-pick executor: scope boundary + fix attempt limit + Write tool rule
- [x] Cherry-pick verifier: success criteria from ROADMAP + Write tool rule
- [x] Brave Search analysis — already present and iOS-adapted, only needs .cjs rename
- [x] Version bumped to 0.3.0
- [x] CHANGELOG updated
- [x] Create SUMMARY.md

### Spec 003 (done)
- [x] Wave 1: {phase} → {phase_num} in 13 files (templates, workflows, commands, agents)
- [x] Wave 2: Tmpfile pattern, CLI tools, 12-char headers, context/plans warnings, STATE.md update
- [x] Wave 3: Verifier Write tool, planner Write instruction, roadmapper structure
- [x] Wave 4: Model inherit + per-agent overrides, planning-config + questioning updates
- [x] Wave 5: Parent artifacts, complete-milestone, quick --full, settings defaults, transition routing, minor updates, new-project auto mode
- [x] Wave 6: Version bump to 0.4.0, audit (82 tests pass, 0 failures)
- [x] CHANGELOG updated, SUMMARY created

### Future
- [ ] README + USER-GUIDE adapted for iOS audience (Spec 004)
- [ ] End-to-end testing of full GSD workflow on iOS project
- [ ] npm publish as `get-shit-done-ios-cc`

### Blocked
None.

---

## Session Continuity

**Last Session:** 2026-02-17

**Stopped At:** Spec 003 complete. Branch `sync/upstream-1.20.0-layer2` ready for merge.

**Resume Context:**
- Specs 001+002+003 done (v0.4.0)
- Upstream 1.20.0 100% synced (infra + content)
- 19 commits on branch, 82 tests pass
- Auto-advance pipeline deferred to Spec 005 (8 files with --auto routing)
- All iOS adaptations preserved (verified: roadmapper 13 refs, verify-phase 8 refs)

**What to Resume:**
1. Merge `sync/upstream-1.20.0-layer2` → `main` (pending user approval)
2. Plan Spec 004: README + USER-GUIDE adaptation for iOS
3. Plan Spec 005: Auto-advance pipeline evaluation

**Blockers:**
None.

---

*State initialized: 2026-02-16*
*Last updated: 2026-02-17*
