# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.7.0] - 2026-06-04

### Fixed

- Force a clean plugin cache refresh for all team members. Version `0.6.0` was assigned to two
  different content states (an early `0.5.0 → 0.6.0` bump on one branch in April, and the actual
  review-skills release on another branch in June), so any cache populated from the April state
  kept serving 11 skills and never picked up the 4 `itkdev-review-*` skills. Because Claude Code
  keys the plugin cache on the `plugin.json` `version` string, the only reliable fix is a brand new
  version number — this release carries no content changes beyond the safeguards below.

### Added

- `.github/workflows/check-version.yml` — CI guard that fails any PR to `main` whose
  `.claude-plugin/plugin.json` version is not strictly greater than the version currently on `main`,
  preventing a version number from ever being reused for different content again.
- Project `CLAUDE.md` documenting the cache-safe release workflow and the rule that one version
  number maps to exactly one content state.

### Changed

- Releasing this plugin no longer creates Git tags or GitHub releases. The plugin cache keys solely
  on the `plugin.json` `version` field, so bumping that field and merging to `main` is the entire
  release. The `v0.6.0` tag and GitHub release have been removed.

## [0.6.0] - 2026-06-02

### Added

- `itkdev-symfony` skill for Symfony development assistance (scaffolding, database configuration,
  console commands, Symfony configuration)
- `itkdev-create-project` agent for creating new Drupal/Symfony projects with ITK Dev Docker setup
- Three per-language code review skills: `itkdev-review-php`, `itkdev-review-python` (with a
  dedicated security review), and `itkdev-review-javascript`
- `itkdev-review-comments` skill for reviewing and improving inline comments and docblocks
  (only touches comments, never code)

### Changed

- `itkdev-code-review` agent now delegates language-specific code-quality checks to the new
  `itkdev-review-*` skills based on the file types in the diff
- `itkdev-code-review` agent findings now include concrete suggested fixes, and the agent can hand
  over its review (post as a GitHub PR review, add inline line comments, or save to a file) via a
  confirmation-gated next-action menu — analysis stays read-only and nothing is published without
  explicit user confirmation; the agent never auto-approves a PR
- All skills are now individually invocable (`user-invocable: true`), so any single skill can be
  run on its own

## [0.5.0] - 2026-03-17

### Added

- Initial release as standalone plugin (extracted from [itkdev-claude-plugins](https://github.com/itk-dev/itkdev-claude-plugins))
- 10 skills: itkdev-docker, itkdev-docker-templates, itkdev-gh-actions, itkdev-taskfile, itkdev-adr, itkdev-documentation, itkdev-drupal, itkdev-github-guidelines, itkdev-issue-workflow, itkdev-validate-standards
- 2 agents: itkdev-code-review, itkdev-issue-workflow

[Unreleased]: https://github.com/itk-dev/itkdev-skills/commits/main
[0.7.0]: https://github.com/itk-dev/itkdev-skills/commits/main
