# GSD:iOS Maintainer Guide

Release workflows and maintenance reference.

## Fork Architecture

Two layers:

- **Upstream layer** — GSD orchestration, tools, CI. Flows in via upstream sync (automated monitor + manual spec execution).
- **iOS layer** — Swift/SwiftUI/MVVM agents, references, templates. Maintained here.

Maintainer responsibility: keep the iOS layer healthy, sync upstream when it advances.

## Upstream Sync Workflow

The automated monitor (`.github/workflows/upstream-sync-monitor.yml`) runs daily. When upstream advances, it creates a GitHub issue with AI-classified commits (KEEP/ADAPT/SKIP).

Manual sync process:

1. Review the GitHub issue created by the monitor
2. Create spec in `.dev/analysis/NNN-upstream-sync-X.X.X/PROMPT.md`
3. Let Claude Code generate `REPORT.md`
4. Create `PLAN.md` in `.dev/tasks/NNN-upstream-sync-X.X.X/`
5. Execute on branch `sync/upstream-X.X.X`
6. Merge to main, close GitHub issue
7. Update `.dev/UPSTREAM_VERSION`

## Release Workflow

Standard release:

```bash
# 1. Create release branch
git checkout -b release/vX.X.X

# 2. Bump version
npm version patch    # or minor, or major

# 3. Update CHANGELOG.md with the release section

# 4. Commit
git add package.json CHANGELOG.md
git commit -m "chore: release vX.X.X"

# 5. Push branch and open PR into main
git push origin release/vX.X.X

# 6. Merge PR into main (--no-ff)
# GitHub Actions (publish.yml) publishes to npm automatically on merge
```

Version cadence:

- **PATCH (0.6.x):** Bug fixes, small iOS improvements. Batch weekly or immediately if critical.
- **MINOR (0.x.0):** New iOS capabilities, significant agent improvements.
- **MAJOR (x.0.0):** Breaking changes. Rare.

Promote to 1.0.0 when: end-to-end tested on real iOS project, stable API, upstream sync proven reliable.

Pre-release for risky changes:

```bash
npm version prerelease --preid=alpha
git push origin main --tags
# Does NOT trigger npm publish — users opt in explicitly
```

## Reviewing iOS Contributions

Checklist:

- [ ] Change is iOS-specific (not platform-agnostic)
- [ ] No web framework references introduced
- [ ] Swift/SwiftUI patterns are current (iOS 17+, @Observable, etc.)
- [ ] Tests still pass (83+)
- [ ] CHANGELOG.md updated for user-facing changes
- [ ] Commit messages follow conventional format

## npm Publish Setup (OIDC)

No NPM_TOKEN secret needed. Uses Trusted Publisher (OIDC).

Already configured. For reference, the one-time setup was:

1. Enable 2FA on npm account (`the-andys`)
2. Publish package manually once to create it on npm registry
3. npmjs.com > `get-shit-done-ios` > Settings > Trusted Publishers > GitHub Actions
4. Fill: Owner=`the-andys`, Repository=`get-shit-done-ios`, Workflow=`publish.yml`, Environment=(blank)

The `publish.yml` workflow handles all subsequent publishes automatically on GitHub Release.

## Recovery Procedures

Broken npm release (within 72h):

```bash
npm unpublish get-shit-done-ios@X.X.X
```

After 72h: publish fix as new patch version.

Wrong tag:

```bash
git tag -d vX.X.X
git push origin :refs/tags/vX.X.X
git tag -a vX.X.X -m "Release vX.X.X"
git push origin vX.X.X
```

## `.dev/UPSTREAM_VERSION`

Single-line file containing the last synced upstream version (e.g., `1.20.4`). Update this file as part of every upstream sync spec, alongside STATUS.md. The upstream monitor reads this file to detect new upstream commits.
