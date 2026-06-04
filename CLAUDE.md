# itkdev-skills — project conventions

This repository hosts **three Claude Code plugins**, each in its own
`plugins/<name>/` subtree, distributed through the `itkdev-marketplace` (defined
in `itk-dev/itkdev-claude-plugins`):

- `plugins/itkdev-code-quality-and-review`
- `plugins/itkdev-scaffolding-and-templates`
- `plugins/itkdev-business-automation`

The marketplace points each plugin at its subdirectory with a `git-subdir`
source; it pins no tag, branch, or commit. The categories follow the skill
*types* in Anthropic's *Lessons from building Claude Code: how we use skills*.

## Versioning and releasing

### The one rule

**One version number maps to exactly one content state. Never reuse a version
number for different content.** This applies **per plugin** — each plugin
versions independently.

Claude Code caches each installed plugin under a directory named after the
`version` field in *that plugin's* `.claude-plugin/plugin.json`:

```
~/.claude/plugins/cache/itkdev-marketplace/<plugin-name>/<version>/
```

The cache is keyed **only** on the plugin name + version string (the
`git-subdir` `path` does not change the key). When updating, Claude Code checks
"is `<plugin-name>/<version>` already cached?" — if yes, it does **not**
re-download. Git tags, GitHub releases, and commit SHAs are not consulted. So if
two different content states ever ship under the same plugin version, whoever
fetched the first state is stuck with it forever — the second never propagates.

This exact bug happened with the old single plugin's `0.6.0`: the version was set
on two separate branches with different content, and caches populated from the
earlier state never received the four `itkdev-review-*` skills. It was resolved by
releasing `0.7.0`, and the plugin was later split into the three plugins above
(each starting fresh at `0.1.0`).

### Release workflow

Do **not** use the global `/release` skill here — it creates Git tags and GitHub
releases, which this repo does not use (they give the plugin cache nothing).

To release a change to one or more plugins:

1. Branch off `main` (`feature/...`).
2. For **each plugin you changed**, bump `version` in its
   `plugins/<name>/.claude-plugin/plugin.json` to a **new, strictly greater**
   value, in the same change that ships the content. Plugins you did not touch
   keep their version.
3. Move the `[Unreleased]` notes in `CHANGELOG.md` under a new
   `## [...] - YYYY-MM-DD` header (one repo-level changelog covers all plugins;
   note which plugin each change belongs to).
4. Open a PR. CI (`.github/workflows/check-version.yml`) checks each plugin
   independently and fails the PR if a **changed** plugin's version is not
   strictly greater than `main`'s (unchanged plugins are exempt).
5. Merge. Each bumped plugin's new version string is what triggers every machine
   to re-fetch that plugin.

No Git tag and no `gh release` are needed — bumping the version(s) and merging to
`main` is the entire release. In particular, do **not** adopt plugin
`dependencies`: that mechanism resolves versions via `{plugin}--v{version}` git
tags, which this repo deliberately avoids. Cross-plugin coupling is handled with
*soft references* (graceful degradation) instead.

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
