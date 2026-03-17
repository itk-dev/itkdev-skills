---
name: itkdev-gh-actions
description: GitHub Actions workflow templates for ITK Dev projects. Use when setting up or updating GitHub Actions workflows, CI/CD pipelines, asking about available workflow templates, or configuring coding standards checks in CI.
---

# ITK Dev GitHub Actions Workflow Templates

You are assisting with GitHub Actions CI/CD for ITK Dev projects. Workflow templates come from the `devops_itkdev-docker` repository and are installed via `itkdev-docker-compose template:install`.

## Workflow Naming Convention

Workflows are named by **concern**, not by tool:

- `php.yaml` (not `phpcs.yaml` or `php-cs-fixer.yaml`)
- `markdown.yaml` (not `markdownlint.yaml`)
- `styles.yaml` (not `prettier-css.yaml`)

## Available Workflow Templates

### General Workflows (all projects)

| Workflow | Purpose | Tool |
|---|---|---|
| `changelog.yaml` | Validates CHANGELOG.md is updated | Custom check |
| `composer.yaml` | Validates composer.json, checks normalization | Composer |
| `markdown.yaml` | Lints Markdown files | markdownlint-cli |
| `twig.yaml` | Validates Twig templates | twig-cs-fixer |
| `yaml.yaml` | Validates YAML files | Prettier |

### Drupal Workflows

| Workflow | Purpose | Tool |
|---|---|---|
| `drupal/php.yaml` | PHP coding standards | phpcs (drupal/coder) |
| `drupal/javascript.yaml` | JavaScript formatting | Prettier |
| `drupal/styles.yaml` | CSS/SCSS formatting | Prettier |
| `drupal/site.yaml` | Tests site install/update from config | Drush |

### Drupal Module Workflows

| Workflow | Purpose | Tool |
|---|---|---|
| `drupal-module/php.yaml` | PHP coding standards for modules | phpcs (drupal/coder) |
| `drupal-module/javascript.yaml` | JavaScript formatting | Prettier |
| `drupal-module/styles.yaml` | CSS/SCSS formatting | Prettier |

### Symfony Workflows

| Workflow | Purpose | Tool |
|---|---|---|
| `symfony/php.yaml` | PHP coding standards | php-cs-fixer |
| `symfony/javascript.yaml` | JavaScript formatting | Prettier |
| `symfony/styles.yaml` | CSS/SCSS formatting | Prettier |

## Workflow Assumptions

All workflows assume:

- A **phpfpm** service is defined in `docker-compose.yml`
- `COMPOSE_USER` environment variable is set (defaults to `deploy`)
- The **frontend** Docker network exists (created by Traefik)
- Required config files are present (`.phpcs.xml.dist`, `.markdownlint.jsonc`, etc.)

## Required Configuration Files

| Workflow | Config File | Framework |
|---|---|---|
| PHP (Drupal) | `.phpcs.xml.dist` | Drupal |
| PHP (Symfony) | `.php-cs-fixer.dist.php` | Symfony |
| Twig | `.twig-cs-fixer.dist.php` | Both |
| Markdown | `.markdownlint.jsonc` | Both |
| YAML (Symfony) | `.prettierrc.yaml` | Symfony |

These config files are installed automatically with `itkdev-docker-compose template:install`.

## How Workflows Are Installed

Templates include `.github/workflows/` directory with pre-configured workflows:

```bash
# Install template (includes workflows)
itkdev-docker-compose template:install drupal-11

# Files created in .github/workflows/:
# - changelog.yaml
# - composer.yaml
# - markdown.yaml
# - twig.yaml
# - yaml.yaml
# - drupal/php.yaml
# - drupal/javascript.yaml
# - drupal/styles.yaml
# - drupal/site.yaml
```

## Updating Workflows

When new workflow versions are released in the `devops_itkdev-docker` repo:

```bash
# Update all template files including workflows
itkdev-docker-compose template:update

# Force update (overwrites local changes)
itkdev-docker-compose template:update --force
```

To manually check for updates, compare local workflow files against the template:

```bash
# Fetch latest workflow from template
curl -sL "https://raw.githubusercontent.com/itk-dev/devops_itkdev-docker/develop/templates/{template}/.github/workflows/{workflow}"
```

## Workflow Documentation Headers

Each workflow template includes a documentation block:

```yaml
### Workflow Title
###
### Description of what this workflow checks
###
### Required config files and dependencies
```

## Common CI Pipeline

A typical Drupal 11 project has these workflows:

```
.github/workflows/
├── changelog.yaml           # PR must update CHANGELOG.md
├── composer.yaml            # composer.json valid and normalized
├── markdown.yaml            # Markdown passes linting
├── twig.yaml                # Twig templates pass linting
├── yaml.yaml                # YAML files properly formatted
└── drupal/
    ├── php.yaml             # PHP passes phpcs with Drupal standards
    ├── javascript.yaml      # JS/TS files properly formatted
    ├── styles.yaml          # CSS/SCSS files properly formatted
    └── site.yaml            # Drupal installs and updates cleanly
```

## Procedural: List Available Workflow Templates

To list workflow templates for a specific project type:

```bash
# General workflows
gh api repos/itk-dev/devops_itkdev-docker/contents/github/workflows --jq '.[].name'

# Framework-specific workflows
gh api repos/itk-dev/devops_itkdev-docker/contents/github/workflows/drupal --jq '.[].name'
gh api repos/itk-dev/devops_itkdev-docker/contents/github/workflows/drupal-module --jq '.[].name'
gh api repos/itk-dev/devops_itkdev-docker/contents/github/workflows/symfony --jq '.[].name'
```

## Procedural: Get Workflow Content

```bash
curl -sL "https://raw.githubusercontent.com/itk-dev/devops_itkdev-docker/develop/github/workflows/{workflow}"
```

Or for framework-specific:

```bash
curl -sL "https://raw.githubusercontent.com/itk-dev/devops_itkdev-docker/develop/github/workflows/drupal/{workflow}"
```
