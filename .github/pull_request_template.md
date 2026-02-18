## What

<!-- One sentence: what does this PR do? -->

## Why

<!-- One sentence: why is this change needed for iOS development? -->

## iOS or Upstream?

<!-- Is this change iOS-specific, or does it apply to all GSD platforms?
     If platform-agnostic, consider contributing to upstream GSD instead. -->

- [ ] This is iOS-specific (correct place)
- [ ] This is platform-agnostic → I'll open an upstream PR instead

## Testing

- [ ] `npm test` passes (83+ tests)
- [ ] Installed locally with `node bin/install.js --claude --local`
- [ ] Tested in Claude Code on a real iOS project

## iOS Quality Checklist

- [ ] No web framework references (React, Next.js, Prisma, etc.)
- [ ] Uses current iOS patterns (iOS 17+, @Observable, SwiftData, async/await)
- [ ] Paths use `Sources/` not `src/`
- [ ] No hardcoded user-facing strings (uses `String(localized:)`)
- [ ] Accessibility not removed or degraded

## GSD Style Checklist

- [ ] Follows GSD-iOS-STYLE.md (no enterprise patterns, no filler)
- [ ] Temporal language not used in implementation files
- [ ] CHANGELOG.md updated for user-facing changes
- [ ] No unnecessary dependencies added
- [ ] Commit messages follow `type(scope): description` format

## Breaking Changes

None
