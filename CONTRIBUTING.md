# Contributing to GSD:iOS

iOS-native development layer for GSD. If it's not iOS-specific, it belongs upstream.

## What This Fork Is

GSD:iOS is a platform port of [GSD](https://github.com/gsd-build/get-shit-done) for native iOS development with Swift, SwiftUI, and MVVM. The upstream GSD handles orchestration, tools, and CI architecture. This fork handles the iOS-specific layer on top: agents tuned for Swift/SwiftUI, iOS reference docs, Xcode-aware templates, and Apple framework guidance.

## Where to Contribute

| Contribute here (GSD:iOS) | Contribute upstream (GSD) |
|---------------------------|---------------------------|
| Swift/SwiftUI agent improvements | New workflow orchestration |
| iOS reference docs (frameworks, testing) | Tool improvements (gsd-tools.cjs) |
| Xcode/Simulator specific fixes | New commands (non-iOS) |
| iOS template improvements | Platform-agnostic bug fixes |
| VoiceOver/accessibility guidance | Installer improvements |

Not sure? Open an issue first describing what you want to fix. We'll route it.

## Branch Strategy

Fork the repo, create a branch, open a PR.

Branch naming (aligned with upstream convention):

- `feat/description` — New iOS capability
- `fix/description` — Bug fix
- `docs/description` — Documentation only
- `sync/description` — Upstream sync work (maintainers only)
- `ci/description` — CI/CD changes (maintainers only)

## Commits

Conventional commits. Same types as upstream: `feat`, `fix`, `docs`, `chore`, `refactor`, `revert`.

Scopes for iOS fork: `agents`, `references`, `templates`, `workflows`, `commands`, `installer`, `tools`, `sync`, `ci`, `branding`.

Format: `type(scope): description`

## What Makes a Good iOS Contribution

- Improves Swift/SwiftUI/MVVM context for Claude
- Follows Apple's current patterns (SwiftUI, @Observable, SwiftData, async/await)
- Doesn't introduce web-framework references (React, Next.js, Prisma, etc.)
- Has a clear iOS use case — not theoretical

## What We Don't Accept

- Changes to platform-agnostic orchestration (belongs upstream)
- Breaking the upstream sync compatibility
- Web framework references in iOS-specific files
- New npm dependencies without discussion

## Pull Request Process

1. Fork the repo, create a branch, make your changes
2. Fill out the PR template
3. One maintainer review required
4. CI must pass (tests)

## Setting Up Locally

```bash
git clone https://github.com/the-andys/get-shit-done-ios
cd get-shit-done-ios
npm install
npm test                          # 83 tests must pass
node bin/install.js --claude --local   # install in current project
```

---

Questions? Open an issue. The complexity is in the system, not your workflow.
