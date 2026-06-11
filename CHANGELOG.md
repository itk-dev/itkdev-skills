# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Dedicated security-review step in `itkdev-review-php` and
  `itkdev-review-javascript`, mirroring the one `itkdev-review-python` already
  had: ten language-specific categories, a per-finding format (severity,
  location, vulnerability type, description, proof of concept, remediation),
  and a closing prioritized remediation plan. Skill descriptions updated to
  advertise it.
- Drupal deep checks in `itkdev-review-php` (`\Drupal::` static calls in
  services, access checks on routes/entities, render-array escaping and `t()`
  placeholders, database API placeholders, deprecated Drupal 10/11 APIs,
  business logic in hooks), with a soft reference to the `itkdev-drupal` skill
  for deeper checks. Framework list reordered to Symfony, Drupal, Laravel,
  plain PHP to match how ITK Dev actually works.
- Python, YAML, and Shell rows in the `itkdev-review-comments` comment-syntax
  table.
- Security parity in `itkdev-review-python` itself: XSS (`mark_safe`, Jinja2
  autoescape), CSRF (`@csrf_exempt`), template injection (SSTI), and
  weak-randomness/crypto checks in the Critical checklist; HTTP-timeout and
  `assert`-for-validation deep checks; quick-reference rows for weak tokens,
  SSTI, and missing timeouts.

### Changed

- The `itkdev-code-review` agent now defines the explicit severity mapping from
  the review skills' four tiers to the report's three (Critical → Critical,
  High → Warning, Medium/Low → Suggestion), and only flags `error_log` /
  `console.warn` as debug code when they are clearly leftover debugging rather
  than deliberate logging.
- Tone Guidelines, severity headers, and the output-format template are now
  word-for-word identical across the three language review skills, so future
  drift shows up in diffs.

### Changed

- **BREAKING:** The single `itkdev-skills` plugin has been split into three
  category plugins, each an independent install target with its own version:
  - `itkdev-code-quality-and-review` (`0.1.1`) — `itkdev-review-php`,
    `itkdev-review-python`, `itkdev-review-javascript`, `itkdev-review-comments`,
    `itkdev-validate-standards`, and the `itkdev-code-review` agent.
  - `itkdev-scaffolding-and-templates` (`0.1.1`) — `itkdev-docker`,
    `itkdev-docker-templates`, `itkdev-gh-actions`, `itkdev-taskfile`,
    `itkdev-drupal`, `itkdev-symfony`, and the `itkdev-create-project` agent.
  - `itkdev-business-automation` (`0.2.0`) — `itkdev-issue-workflow`,
    `itkdev-github-guidelines`, `itkdev-adr`, and `itkdev-documentation`.

  The categories follow the skill *types* in Anthropic's *Lessons from building
  Claude Code: how we use skills*. The old single `itkdev-skills` plugin entry is
  removed from `itkdev-marketplace`; existing users must install the new plugins.
  Skills/agents moved into `plugins/<name>/` subtrees but are otherwise unchanged.
- Cross-plugin references are now *soft*: the `itkdev-code-review` agent no longer
  hard-declares `itkdev-github-guidelines` / `itkdev-drupal` (which live in other
  plugins) in its `skills:` frontmatter, and `itkdev-validate-standards` notes that
  its referenced Docker/Taskfile/CI skills ship in `itkdev-scaffolding-and-templates`.
  Both degrade gracefully when the other plugin is not installed. No git-tag-based
  plugin `dependencies` are used (consistent with the repo's no-tags release rule).
- `itkdev-documentation` rewritten as thin guidance: the four large templates moved
  to `references/` (progressive disclosure), duplicated Docker/Taskfile/detection
  tables replaced with pointers to the relevant skills.
- `.github/workflows/check-version.yml` now version-checks each plugin in
  `plugins/*/` independently — a plugin whose subtree changed must bump its own
  version; unchanged plugins are exempt.
- **`itkdev-business-automation` (`0.2.0`):** `itkdev-issue-workflow` skill reworked
  from an autonomous "phases 1–4" automation script into a developer-driven guide
  for how the team works a GitHub issue (Claude assists; the developer drives and
  decides when to merge). Removed references to the non-existent `dev-browser` skill
  and `pr-review-toolkit:*` subagents; testing guidance now covers project CI tasks,
  manual local verification, and writing/extending tests, and review points at the
  real `itkdev-code-review` agent. Git/PR mechanics now soft-reference
  `itkdev-github-guidelines` instead of restating them.
- **`itkdev-code-quality-and-review` (`0.1.1`):** `itkdev-validate-standards`
  frontmatter normalized (dropped the non-standard `author`/`version` fields) and the
  duplicated Problem/Trigger-Conditions preamble collapsed into a short intro, keeping
  the body under the 500-line guideline. No validation checks removed.
- **`itkdev-scaffolding-and-templates` (`0.1.1`):** `itkdev-symfony` now declares
  `user-invocable: true` to match every other skill. Frontmatter key order normalized
  on `itkdev-symfony` and `itkdev-review-comments` (`name → user-invocable →
  description`).

### Removed

- **`itkdev-business-automation` (`0.2.0`):** removed the autonomous
  `itkdev-issue-workflow` agent. The reworked, developer-driven
  `itkdev-issue-workflow` skill is now the single source for how the team solves
  issues.

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
