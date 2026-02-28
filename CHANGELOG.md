# Changelog

All notable changes to GSD iOS will be documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased] — Task 016 (accumulating for v0.11.0)

### Added
- Xcode MCP tools reference (`ios-mcp-tools.md`) — three-layer architecture: Apple xcode-tools (~20 tools), XcodeBuildMCP (~76 tools), CLI fallback
- Build & Test Tooling section in `ios-swift-guidelines.md` — MCP-preferred with CLI fallback for every operation
- MCP Testing Tools section in `ios-testing.md` — Apple vs XcodeBuildMCP decision table
- Xcode MCP File Operations subsection in `ios-frameworks.md` — pbxproj automatic management via XcodeWrite/XcodeUpdate
- iOS scaffolding step in `new-project.md` — generates `.gitignore` and `CLAUDE.md` when iOS stack detected (GAP-12)
- Context prompt in `discuss-phase.md` — captures rationale per decision, skippable (GAP-05)

### Fixed
- Git status warning in `verify_phase_goal` — warns on untracked `.planning/` files, informational only (GAP-09)
- Summary fidelity instruction in `checkpoint_handling` — SUMMARY.md must reflect actual execution data, not predicted values (GAP-08)
- Test co-location constraint in `gsd-roadmapper.md` — tests must be in same phase as feature, never in separate Testing phase (GAP-10)

### Task 017 — Upstream sync v1.21.0 to v1.22.0 (accumulating for v0.11.0)

#### Added
- Code-aware discuss phase with codebase scouting — scout step identifies reusable Views/ViewModels/Services before gray area analysis; iOS-adapted examples (Swift, SwiftData, MVVM, NavigationStack)
- code_context section in CONTEXT.md template — captures reusable assets, established patterns, and integration points
- Analysis paralysis guard in gsd-executor — stops infinite read loops after 5+ consecutive reads without action
- Exhaustive cross-check in gsd-plan-checker — verifies PROJECT.md requirements not silently dropped from phases
- Task-level TDD guidance in gsd-planner — tdd="true" + behavior block pattern (iOS-adapted: swift test --filter)

#### Fixed
- $HOME instead of ~ in all agent/workflow paths — prevents subagent MODULE_NOT_FOUND on path rewrites (affects external volume users)
- Dual .claude/skills/ + .agents/skills/ discovery — project skills no longer silently ignored in Claude Code
- nyquist_validation missing from loadConfig() return — plan-phase now correctly detects nyquist validation config
- model_overrides missing from loadConfig() return — configured GSD profiles now correctly applied
- stateExtractField regex metacharacter escape — field names with special chars no longer cause regex errors
- phase-plan-index field lookups — underscore/hyphen normalization, XML tag extraction for objective and task_count
- getMilestoneInfo() wrong version after milestone completion — strips details blocks before matching
- Milestone stats/archive scoped to current milestone phases only — prevents inflated stats from prior milestones
- MILESTONES.md entries inserted in reverse chronological order (newest first)
- Resolve-model variable name alignment — DEBUGGER_MODEL/CHECKER_MODEL to lowercase to match Task() placeholders
- Statusline migration regex narrowed to hooks/statusline.js path only — prevents clobbering third-party statusline scripts
- Update workflow clears both cache paths — prevents stale update indicators regardless of install type

## [0.10.0] - 2026-02-27

Upstream sync: v1.20.6 → v1.21.0

### Added
- YAML frontmatter sync for STATE.md — machine-readable status tracking via state-json and state-sync commands
- PRD express path for plan-phase (`--prd <file>` flag) — bypasses discuss-phase with locked decisions from PRD
- `/gsd:add-tests` workflow — standardized post-phase test generation (iOS-adapted: Swift Testing, XCTest, XCUITest)
- Human verification gate before resolving debug sessions — new `awaiting_human_verify` status and checkpoint
- `<project_context>` block in gsd-verifier — discovers CLAUDE.md and project skills before verification
- Interface Context for Executors section in planner — provides protocol/type contracts to executor agents
- Interface-First Task Ordering guidance in planner
- Living retrospective in complete-milestone workflow — RETROSPECTIVE.md updated per milestone with trends
- RETROSPECTIVE.md reading in planner for cross-milestone learning
- `phase_req_ids` propagated from init JSON in execute-phase and plan-phase workflows (removes fragile shell extraction)

### Changed
- Nyquist compression across agents: gsd-phase-researcher, gsd-plan-checker, gsd-planner — reduced verbose text while preserving iOS markers (XCUITest, swift test, .xctestplan references)
- Nyquist compression in VALIDATION.md template
- Update workflow version detection now validates install integrity with marker file checks and version regex
- Update workflow npx commands now use `npx -y get-shit-done-ios@latest` for reliable execution
- Installer onboarding: `/gsd:help` → `/gsd:new-project`, improved launch instructions, updated Discord link
- `writeManifest()` and `reportLocalPatches()` now accept runtime parameter and execute before hooks config
- Decision/blocker inputs use file-based text for shell-safe handling (preserves `$`, `*` in STATE.md)

### Fixed
- Multi-level decimal phase handling (e.g., 72.1.1) with proper regex escaping
- Progress bar percent clamping to prevent RangeError crash
- requirements-completed frontmatter mapping in summary-extract
- STATE.md decision corruption with dollar sign handling
- `--cwd` override support in state-snapshot command
- Discord invite URL updated to vanity URL (discord.gg/gsd)
- Stale version references in package-lock.json (0.7.0 → 0.10.0) and terminal.svg (v0.7.0 → v0.10.0)
- Package name mismatch in package-lock.json

## [0.9.0] - 2026-02-26

Upstream sync: v1.20.5 → v1.20.6

### Added
- Nyquist validation layer — researcher, planner, plan-checker, settings, VALIDATION.md template
- Context window monitor hook with agent-side WARNING/CRITICAL alerts
- Option highlighting and gray area looping in discuss-phase
- Universal phase number parsing with letter-suffix support (comparePhaseNum)

### Changed
- gsd-tools.cjs refactored into 11 domain modules under bin/lib/
- Test files split into per-module files under tests/

### Fixed
- Auto-advance chain — Skills don't resolve inside Task subagents (discuss-phase Task spawn)

## [0.8.0] - 2026-02-20

### Changed (upstream sync v1.20.4 → v1.20.5)
- Context-proxy orchestration: agents now consume `<files_to_read>` path blocks
  instead of receiving inline content — reduces context bloat
- All commands delegate file resolution to workflow layer (removed inline `@.planning/` refs)
- Workflows migrated from `--include` flags + `*_content` vars to `*_path` references
- Agents (executor, planner, phase-researcher, plan-checker) now discover and read
  project-level `CLAUDE.md` and `.agents/skills/` at spawn time
- `map-codebase` workflow now uses inline `Task()` syntax for agent spawning

### Fixed (upstream sync v1.20.4 → v1.20.5)
- `gsd:health --repair` now creates timestamped backup (`STATE.md.bak-*`) before
  overwriting STATE.md — prevents accidental context loss

### Skipped (upstream-only)
- Codex CLI features and their reverts (`87c3873`, `db1d003`, `e820263`, `d55998b`) —
  net zero change to codebase, Codex-specific only

## [0.7.1] - 2026-02-19

### Fixed
- Installer logo now correctly displays full GSD:iOS block-letter branding
- Update check now queries `get-shit-done-ios` on npm instead of upstream package
- Help documentation no longer references upstream `get-shit-done-cc` package name
- Local installs now use absolute paths, fixing resolution on macOS external volumes

## [0.7.0] - 2026-02-18

### Fixed
- Installer banner no longer references OpenCode or Gemini — correctly attributed to Claude Code by TÂCHES, iOS adaptation by the-andys
- `publish.yml` version check now properly skips the publish step when a version already exists on npm (was using `exit 0` which only exited the shell script, not the job step)
- `engines.node` in `package.json` aligned to `>=18.0.0` (was `>=16.7.0`, conflicting with README and CI)

### Changed
- `MAINTAINERS.md` OIDC setup section updated to reflect actual setup process and correct package name
- Release workflow updated: from v0.7.0 onwards, all releases use a `release/vX.X.X` branch and PR into main

### Removed
- `commands/gsd/new-project.md.bak` removed from repository and npm tarball (29KB of stale content)
- `assets/terminal.svg` updated to match correct attribution and remove stale runtime references

## [0.6.0] - 2026-02-17

### Added
- Auto-advance pipeline mode (`--auto` flag or `workflow.auto_advance` config) for unattended execution
- Requirements tracking chain: `requirements` field in PLAN.md frontmatter, `requirements-completed` in SUMMARY.md
- `gsd-tools.cjs requirements mark-complete` CLI command with bracket-format support
- 3-source cross-reference for requirements verification (PLAN frontmatter × REQUIREMENTS.md × VERIFICATION.md)
- Requirements Integration Map in integration checker output
- Global defaults support (`~/.gsd/defaults.json`) in new-project workflow
- ROADMAP.md and REQUIREMENTS.md auto-update in executor post-plan state management

### Changed
- Verifier Step 6 now uses rigorous 3-part requirements coverage check (6a, 6b, 6c)
- Plan-checker Dimension 1 now verifies requirement IDs across plan frontmatter fields
- Milestone audit uses 3-source cross-reference system (5a-5e) with FAIL gates
- Requirements bracket format (`[REQ-01, REQ-02]`) now stripped automatically by parser

### Skipped (upstream-only)
- Gemini CLI shell variable escaping (`e449c5a`) — iOS fork does not target Gemini CLI

## [0.5.0] - 2026-02-17

### Changed
- README.md fully adapted for iOS fork identity (title, badges, attribution, install, examples, troubleshooting)
- SECURITY.md dual-contact model — GSD core issues and iOS fork issues with separate contacts
- USER-GUIDE.md quick fix example updated from mobile Safari to iOS 17 VoiceOver scenario
- Security deny list expanded with `.p12`, `.mobileprovision`, `Provisioning/`, `Certificates/` patterns

### Removed
- Upstream-only README sections: Community Ports, Star History, Discord/Twitter badges, $GSD token badge
- npx install commands replaced with git clone workflow
- OpenCode/Gemini CLI references from README

## [0.4.0] - 2026-02-17

### Added
- `--full` flag for `/gsd:quick` — enables plan-checking (max 2 iterations) and post-execution verification
- Parent artifact closure in `execute-phase` — resolves UAT gaps and debug sessions for decimal phases
- Phase completion via CLI (`gsd-tools.cjs phase complete`) replacing manual ROADMAP marking
- CONTEXT.md existence check in transition routing — suggests discuss-phase when no context exists
- Warning when planning without user context (plan-phase.md)
- Warning when discussing a phase that already has plans (discuss-phase.md)
- STATE.md update step after discuss-phase sessions
- Success Criteria from ROADMAP as Option B in verify-phase
- Per-Agent model overrides (`model_overrides` in config.json)
- Model `inherit` resolution for opus-tier agents (avoids version conflicts)
- Save-as-defaults option in settings workflow (`~/.gsd/defaults.json`)
- Phase archival option in complete-milestone workflow
- `/gsd:cleanup` command entry in help workflow
- `milestones/` directory in planning tree structure
- 12-char header limit enforcement for AskUserQuestion
- "Other" free text intent handling in discuss-phase
- `commit_docs` conditional reset in branch merge scenarios
- Tmpfile `@file:` pattern for large JSON payloads (progress, plan-phase, execute-plan)
- Write tool instruction to planner agent
- Detailed ROADMAP.md structure guidance to roadmapper agent (dual representation)
- User tip for modifying options via "Other" in questioning reference
- Task tool to discuss-phase allowed-tools

### Changed
- `{phase}` → `{phase_num}` in all filename references (13 files: templates, workflows, commands, agents)
- `.cjs` rename completed in 41 content files (8 agents, 26 workflows, 5 references, 2 commands)
- Model resolution steps updated (check overrides before profile table)
- `execute-plan.md` uses CLI tools for phase listing and progress updates
- `audit-milestone.md` uses `find-phase` CLI approach
- `new-project.md` auto mode accepts pasted text (not just @file: references)
- Shorter description for plan-phase command
- Path templating comments in update and reapply-patches workflows

## [0.2.0] - 2026-02-16

### Added
- `/gsd:cleanup` command — archives phase directories from completed milestones
- `/gsd:health` command — validates `.planning/` directory integrity with `--repair` flag
- `docs/USER-GUIDE.md` — comprehensive usage guide

### Changed
- Synced all infrastructure with upstream GSD 1.20.0
- Renamed `gsd-tools.js` → `gsd-tools.cjs` to prevent ESM module resolution conflicts
- Updated `install.js` with CommonJS package.json injection and OpenCode improvements
- Added `workflow.auto_advance` setting to config template (default: false)
- Synced README and CHANGELOG with upstream

### Fixed
- ESM inheritance conflicts when user project uses `"type": "module"`
- Settings file corruption during config.json writes
- JSON truncation with large payloads (now uses temp files)
- Test script path in package.json (upstream bug: referenced `.test.js` instead of `.test.cjs`)

## [0.1.0] - 2026-02-16

### Added
- **Platform port of GSD from web to iOS native development**
- 3 new iOS reference documents: `ios-swift-guidelines.md`, `ios-frameworks.md`, `ios-testing.md`
- 2 new iOS-specific references: `ios-app-lifecycle.md`, `ios-permissions.md`

### Changed
- All 10 agent prompts adapted for Swift/SwiftUI/SwiftData/MVVM patterns
- All workflow examples replaced with iOS equivalents (xcodebuild, Simulator, TestFlight)
- All templates adapted with iOS file paths, Swift Testing, and MVVM structure
- All reference docs updated with iOS debugging, verification, and checkpoint patterns
- Package identity: `get-shit-done-ios-cc`

### Removed
- All web-specific patterns: React, Next.js, Prisma, npm run, .tsx/.jsx references

---

*Based on upstream [Get Shit Done](https://github.com/gsd-build/get-shit-done) v1.18.0*
