---
name: itkdev-github-guidelines
description: GitHub workflow guidelines for itk-dev team. Use when working with Git, creating branches, making commits, opening pull requests, or any GitHub-related development tasks. Ensures compliance with team conventions for branching, commits, changelogs, and PR workflows.
---

# itk-dev GitHub Workflow

## Core Rules

1. **Never commit directly to main** - All changes go through pull requests
2. **Always create a feature branch** for every PR
3. **Every PR must include** an updated CHANGELOG and reference related GitHub issues
4. **Close related issues** upon PR merge using "Closes #XX" syntax

## Branch Naming

Format: `feature/issue-{number}-{short-description}`

Examples:
- `feature/issue-123-add-user-authentication`
- `feature/issue-45-fix-pagination-bug`
- `feature/issue-78-update-api-endpoints`

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`

Examples:
- `feat: add language switcher component`
- `fix: resolve pagination offset error`
- `docs: update API documentation`
- `refactor: extract validation logic to service`

## Changelog

Follow [Keep a Changelog](https://keepachangelog.com/) format. Update `CHANGELOG.md` with every PR under the `[Unreleased]` section.

Categories: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`

## Pull Request Requirements

### Before Opening PR
- [ ] All GitHub Actions checks pass
- [ ] CHANGELOG.md updated under `[Unreleased]`
- [ ] Branch is up-to-date with main

### PR Description Template

```markdown
## Summary
Brief description of what this PR does and which issue it addresses (#XX).

## Features Added
* Feature 1: Description
* Feature 2: Description

## Files Changed
* `path/to/file.ext` - What changed
* `path/to/new-file.ext` (new) - Purpose

## Test Plan
* Step-by-step verification instructions
* Expected outcomes for each test

Closes #XX
```

### Review Requirements
- Minimum **one approving review** required before merge
- All GitHub Actions checks must pass
