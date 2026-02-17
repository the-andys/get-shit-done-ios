# Changelog

All notable changes to GSD iOS will be documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added
- Scope boundary and fix attempt limit rules in executor agent (upstream cherry-pick)
- Success Criteria from ROADMAP verification path in verifier agent (upstream cherry-pick)
- Write tool enforcement rule in executor and verifier agents (upstream cherry-pick)

### Changed
- `.cjs` rename completed in 41 content files (8 agents, 26 workflows, 5 references, 2 commands)

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
