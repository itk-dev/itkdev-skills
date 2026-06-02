# itkdev-skills

ITK Dev team conventions, workflows, and coding standards for Claude Code.

## Install

```bash
claude plugin add itk-dev/itkdev-skills
```

## Skills

| Skill | Description |
|-------|-------------|
| `itkdev-docker` | Docker development environment (CLI reference, Compose architecture, services, Traefik, server deployments, project detection, template comparison) |
| `itkdev-docker-templates` | Project template conventions (available templates, installation, setup workflows, procedural template operations) |
| `itkdev-gh-actions` | GitHub Actions workflow templates (general, Drupal, Symfony workflows, configuration files) |
| `itkdev-taskfile` | Taskfile development workflows (task patterns, coding standards, site management, asset building) |
| `itkdev-adr` | Architecture Decision Record management |
| `itkdev-documentation` | Technical documentation and README generation following ITK Dev standards |
| `itkdev-drupal` | Drupal 10/11 development assistance (code auditing, module/theme development, configuration management) |
| `itkdev-github-guidelines` | GitHub workflow guidelines (branch naming, commits, changelogs, PRs) |
| `itkdev-issue-workflow` | Autonomous GitHub issue workflow |
| `itkdev-validate-standards` | Project standards validation against ITK Dev conventions |
| `itkdev-review-php` | PHP code review checklist (Laravel, Symfony, plain PHP) — security, correctness, performance, PSR-12 |
| `itkdev-review-python` | Python code review checklist (Django, Flask, FastAPI, scripts) — includes a dedicated security review |
| `itkdev-review-javascript` | JavaScript/TypeScript code review checklist — security, async error handling, type safety, style |
| `itkdev-review-comments` | Review and improve inline comments and docblocks (explains "why" not "what") — only touches comments, never code |

## Agents

| Agent | Description |
|-------|-------------|
| `itkdev-code-review` | Automated PR review against ITK Dev standards |
| `itkdev-issue-workflow` | Autonomous GitHub issue workflow (runs in isolated subagent context) |

> **Review skills vs. the review agent:** the `itkdev-review-*` skills are language-specific
> checklists you can invoke inline on a file or snippet. The `itkdev-code-review` **agent** is the
> PR-review orchestrator — it gathers PR data, runs process-compliance checks, and delegates
> language-specific code-quality checks to the matching `itkdev-review-*` skill.
