---
name: itkdev-issue-workflow
user-invocable: true
description: How the ITK Dev team works a GitHub issue from start to finish — understand, branch, implement, test, review, open a PR, and merge. Use when picking up a GitHub issue, asking how we solve issues, or wanting help working an issue in an ITK Dev project. Developer-driven; Claude assists at each step.
---

# Working a GitHub Issue (ITK Dev)

This is how the ITK Dev team takes a GitHub issue from "assigned" to "merged".
The **developer drives** — they make the decisions and set the pace. Claude
**assists** at each step: gathering context, drafting code, running checks, and
catching problems. Don't run the whole thing autonomously or merge on the
developer's behalf.

The Git/PR mechanics (branch naming, Conventional Commits, CHANGELOG, PR
description) live in the `itkdev-github-guidelines` skill — follow it rather than
restating the rules here.

## 1. Understand the issue

- `gh issue list --state open --limit 10` to see what's open, or
  `gh issue view <number>` for a specific one.
- Read the issue fully. Summarize it back to the developer and confirm the scope
  and acceptance criteria before writing code — surface anything ambiguous or
  underspecified now, not after a PR is open.
- For non-trivial work, plan the implementation first (use plan mode) and check
  the project's `CLAUDE.md` for project-specific conventions.

## 2. Branch and implement

- Start from an up-to-date `main`:
  `git checkout main && git pull`.
- Create a feature branch following `itkdev-github-guidelines`
  (`feature/issue-<number>-<short-description>`). Never commit to `main`.
- Implement the change. Keep commits focused and use Conventional Commits.
- Update `CHANGELOG.md` under `[Unreleased]` as part of the work.

## 3. Test before opening a PR

Use whichever of these fit the change — usually more than one:

- **Run the project's CI checks.** Most ITK Dev projects expose them through a
  Taskfile (`task ci`) or composer scripts, run inside Docker. See the
  `itkdev-taskfile` and `itkdev-docker` skills (in the
  `itkdev-scaffolding-and-templates` plugin) for the command patterns; the
  "Detecting the project's checks" section below is a quick fallback when those
  skills aren't installed. Apply coding-standards auto-fixes first, then run the
  full suite, and fix anything that fails before continuing.
- **Verify manually in the running local environment.** Bring the Docker site up
  and exercise the changed behaviour the way a user would — load the affected
  page, submit the form, trigger the flow — and confirm it does what the issue
  asks.
- **Add or extend automated tests.** When the fix is testable, write or update
  tests to cover it so the behaviour stays correct. Run them as part of the CI
  suite above.

## 4. Self-review

- Re-read your own diff critically: bugs, edge cases, security, leftover debug
  code, and whether it actually closes the issue.
- For a deeper pass, the `itkdev-code-review` agent (in the
  `itkdev-code-quality-and-review` plugin) reviews a branch/PR against ITK Dev
  standards and produces a structured report. Use it when available; otherwise
  review manually. Fix what you find before opening the PR.

## 5. Open the PR

- Push the branch and open a PR. Follow `itkdev-github-guidelines` for the PR
  description (Summary, the `Fixes #<number>` / `Closes #<number>` reference, and
  a Test Plan) and the pre-PR checklist (CI green, CHANGELOG updated, branch
  up to date with `main`).
- Hand the PR to the developer with a short summary of what changed, how it was
  tested, and what the self-review found.

## 6. Merge

Merging is the developer's call (and the team requires at least one approving
review). Don't merge automatically. Once it's merged, switch back to `main`,
pull, and you're ready for the next issue.

## Detecting the project's checks

A quick reference for finding what a project exposes, when the `itkdev-taskfile` /
`itkdev-docker` skills aren't available. All commands run inside the project's
Docker setup.

```bash
# Taskfile tasks (if Taskfile.yml / Taskfile.yaml exists)
task --list

# Composer scripts
itkdev-docker-compose composer run --list
```

Common Taskfile tasks:

| Task | Purpose |
|------|---------|
| `task ci` | Full CI suite (coding standards + tests) |
| `task ci:coding-standards` | Coding standards check only |
| `task ci:phpunit` | PHPUnit tests only |
| `task coding-standards:apply` | Auto-fix coding standards issues |
| `task config:export` | Export Drupal configuration |
| `task config:import` | Import Drupal configuration |
| `task dev:setup` | Initial project setup |
| `task dev:reset` | Reset local environment |

Direct fallbacks when no task or composer script exists:

```bash
itkdev-docker-compose vendor/bin/phpcbf     # Auto-fix standards (run first)
itkdev-docker-compose vendor/bin/phpcs      # Coding standards check
itkdev-docker-compose vendor/bin/phpunit    # PHPUnit tests
itkdev-docker-compose vendor/bin/phpstan    # Static analysis
```
