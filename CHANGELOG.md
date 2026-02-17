# Changelog

All notable changes to GSD iOS will be documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
