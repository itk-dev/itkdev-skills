---
name: itkdev-taskfile
description: Taskfile development workflows for ITK Dev projects. Use when working with Taskfile.yml, running task commands, setting up task automation, coding standards, site management, asset building, or asking about available tasks.
---

# ITK Dev Taskfile Workflows

You are assisting with Taskfile-based automation in ITK Dev projects. This skill covers Taskfile.yml structure, standard tasks, and development workflows.

**Convention:** Always check for and use Taskfile tasks before running raw `itkdev-docker-compose` commands. Tasks chain multiple commands and handle edge cases.

## Listing Available Tasks

```bash
task                    # List all available tasks with descriptions
task --list-all         # List all tasks including those without descriptions
```

## Taskfile.yml Structure

ITK Dev Taskfiles follow a standard structure with variables and task composition:

```yaml
version: '3'

vars:
  DOCKER: itkdev-docker-compose
  COMPOSER: "{{.DOCKER}} composer"
  DRUSH: "{{.DOCKER}} drush"
  PHP: "{{.DOCKER}} php"
  NPM: docker compose run --rm node npm

tasks:
  # Tasks defined here...
```

### Variable Hierarchy

- `vars:` at root level define global defaults
- Tasks can override with local `vars:`
- Dynamic variables use `sh:` for runtime evaluation
- CLI arguments passed via `-- <args>` syntax (e.g., `task drush -- cr`)

## Core Task Patterns

### Docker Compose Wrapper

```yaml
compose:
  desc: Run docker compose command
  cmds:
    - docker compose {{.CLI_ARGS}}
```

Usage: `task compose -- up -d`

### Compose Up

```yaml
compose-up:
  desc: Start Docker containers
  cmds:
    - docker compose up -d
```

### Site Install (Drupal)

```yaml
site-install:
  desc: Install site from scratch
  cmds:
    - "{{.COMPOSER}} install"
    - "{{.DRUSH}} site:install --existing-config --yes"
    - "{{.DRUSH}} cr"
```

### Site Update (Drupal)

```yaml
site-update:
  desc: Update site after code changes
  cmds:
    - "{{.COMPOSER}} install"
    - "{{.DRUSH}} updb --yes"
    - "{{.DRUSH}} cim --yes"
    - "{{.DRUSH}} cr"
```

## Composer and Drush Wrappers

```yaml
composer:
  desc: Run composer command
  cmds:
    - "{{.COMPOSER}} {{.CLI_ARGS}}"

drush:
  desc: Run drush command
  cmds:
    - "{{.DRUSH}} {{.CLI_ARGS}}"
```

Usage: `task composer -- require drupal/admin_toolbar` or `task drush -- cr`

## Asset Building

```yaml
npm-install:
  desc: Install Node.js dependencies
  cmds:
    - "{{.NPM}} install"

npm-build:
  desc: Build frontend assets
  cmds:
    - "{{.NPM}} run build"

npm-watch:
  desc: Watch for frontend asset changes
  cmds:
    - "{{.NPM}} run watch"
```

## Coding Standards Tasks

ITK Dev projects define tasks for checking and auto-fixing coding standards. The naming pattern is `coding-standards-{type}-{check|apply}`.

**Available types:** `php`, `javascript`, `markdown`, `styles` (CSS/SCSS), `twig`, `yaml`

Check tasks run linters in read-only mode. Apply tasks auto-fix violations.

```bash
# Check all standards
task coding-standards-php-check
task coding-standards-javascript-check
task coding-standards-twig-check
task coding-standards-markdown-check
task coding-standards-styles-check
task coding-standards-yaml-check

# Auto-fix violations
task coding-standards-php-apply
task coding-standards-javascript-apply
task coding-standards-twig-apply
task coding-standards-markdown-apply
task coding-standards-styles-apply
task coding-standards-yaml-apply
```

**Tools by framework:**
- PHP: `phpcs`/`phpcbf` (Drupal) or `php-cs-fixer` (Symfony)
- JavaScript/Styles/YAML: `prettier`
- Markdown: `markdownlint`
- Twig: `twig-cs-fixer`

## Other Common Tasks

| Task | Description |
|---|---|
| `code-analysis` | Run PHPStan static analysis |
| `database-dump` | Dump database to file |
| `dev-settings-twig-debug` | Enable Twig debug mode |
| `translations-import` / `translations-export` | Import/export translations |
| `images-pull` | Pull latest Docker images |
| `fixtures-load` | Load fixture content |

## Task Patterns

Tasks support these Taskfile features:

- **CLI arguments**: `task drush -- cr` (pass `-- <args>` to the underlying command)
- **Prompts**: `prompt:` key for interactive confirmation on dangerous tasks
- **Silent mode**: `silent: true` suppresses command echo
- **Dynamic variables**: `sh:` for runtime evaluation (e.g., current git branch)
- **Composition**: `deps:` for parallel prerequisites, `cmds:` with `task:` for sequential subtasks

## Common Workflows

### New Developer Setup

```bash
task compose-up          # Start containers
task site-install        # Install site (or site-update for existing)
task npm-install         # Install frontend dependencies
task npm-build           # Build assets
```

### Daily Development

```bash
task compose-up          # Ensure containers running
task site-update         # Pull latest config/database changes
task npm-watch           # Watch for asset changes (if applicable)
```

### Before Committing

```bash
task coding-standards-php-check
task coding-standards-javascript-check
task coding-standards-twig-check
task coding-standards-markdown-check
task code-analysis       # PHPStan (if configured)
```

Or if a combined CI task exists:

```bash
task ci                  # Run all checks
```
