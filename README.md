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
| `itkdev-symfony` | Symfony development assistance (scaffolding, database configuration, console commands, Symfony configuration) |
| `itkdev-validate-standards` | Project standards validation against ITK Dev conventions |

## Agents

| Agent | Description |
|-------|-------------|
| `itkdev-code-review` | Automated PR review against ITK Dev standards |
| `itkdev-create-project` | Create new Drupal/Symfony projects with ITK Dev Docker setup |
| `itkdev-issue-workflow` | Autonomous GitHub issue workflow (runs in isolated subagent context) |
