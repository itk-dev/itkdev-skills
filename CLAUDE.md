# itkdev-skills — project conventions

This repository is a **Claude Code plugin** distributed through the
`itkdev-marketplace` (defined in `itk-dev/itkdev-claude-plugins`). The marketplace
tracks this repo directly — it pins no tag, branch, or commit.

## Versioning and releasing

### The one rule

**One version number maps to exactly one content state. Never reuse a version
number for different content.**

Claude Code caches an installed plugin under a directory named after the
`version` field in `.claude-plugin/plugin.json`:

```
~/.claude/plugins/cache/itkdev-marketplace/itkdev-skills/<version>/
```

The cache is keyed **only** on that version string. When updating, Claude Code
checks "is `<version>` already cached?" — if yes, it does **not** re-download.
Git tags, GitHub releases, and commit SHAs are not consulted. So if two different
content states ever ship under the same version, whoever fetched the first state
is stuck with it forever — the second never propagates.

This exact bug happened with `0.6.0`: the version was set on two separate branches
with different content, and caches populated from the earlier state never received
the four `itkdev-review-*` skills. It was resolved by releasing `0.7.0`.

### Release workflow

Do **not** use the global `/release` skill here — it creates Git tags and GitHub
releases, which this repo does not use (they give the plugin cache nothing).

To release:

1. Branch off `main` (`feature/...`).
2. Bump `version` in `.claude-plugin/plugin.json` to a **new, strictly greater**
   value, in the same change that ships the content.
3. Move the `[Unreleased]` notes in `CHANGELOG.md` under a new
   `## [X.Y.Z] - YYYY-MM-DD` header.
4. Open a PR. CI (`.github/workflows/check-version.yml`) fails the PR if the
   version is not greater than `main`'s.
5. Merge. The new version string is what triggers every machine to re-fetch.

No Git tag and no `gh release` are needed — bumping the version and merging to
`main` is the entire release.

### Recovering a poisoned version

If a version number was already published with the wrong content, you cannot fix
it by re-publishing the same number — the stale cache directory still exists on
every machine. Bump to a new version instead. (Individual machines can clear a
stale entry by deleting its cache directory, but that does not help teammates.)

## General conventions

Follow the global conventions in `~/.claude/CLAUDE.md` (English everywhere,
Conventional Commits, `Co-authored-by` trailer, feature branches off `main`,
Keep a Changelog). The release process above **overrides** the global
changelog/release guidance for this repo specifically.
