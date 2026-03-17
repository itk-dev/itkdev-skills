---
name: itkdev-validate-standards
description: |
  Validate project against itk-dev Docker and development conventions from
  itk-dev/devops_itkdev-docker. Use when: (1) auditing Docker setup, (2)
  "check itk-dev standards", (3) reviewing PR for convention compliance,
  (4) setting up a new itk-dev project, (5) upgrading Docker configuration.
  Walks through 9 validation areas: template comparison, docker-compose,
  server compose, environment, Taskfile, framework-specific config, Composer,
  GitHub Actions, and miscellaneous files.
author: Claude Code
version: 1.0.0
---

# Validate itk-dev Standards

## Problem

itk-dev projects must follow conventions from the
`itk-dev/devops_itkdev-docker` template repository. Manually checking
compliance across Docker Compose files, Taskfile, GitHub Actions, PHP
tooling, and environment configuration is error-prone. This skill
provides a structured checklist to audit any itk-dev project.

## Context / Trigger Conditions

Use this skill when:
- Auditing a project for itk-dev convention compliance
- Reviewing a PR that changes Docker, Taskfile, or CI configuration
- Setting up a new itk-dev project
- Upgrading from older itk-dev Docker conventions
- User asks to "check itk-dev standards" or "validate Docker setup"

## Solution

### Step 0 — Automated Comparison

Before manual checks, attempt automated comparison against the canonical
template. See the `itkdev-docker` and `itkdev-docker-templates` skills
for full procedural details.

**Procedure:**

1. **Detect project type:** Read `.env` for `ITKDEV_TEMPLATE` value and
   `docker-compose.yml` for `# itk-version:` comment and PHP image version.
   See the `itkdev-docker` skill (Project Detection section).

2. **Compare against template:** Fetch template files from GitHub and compare
   `# itk-version:` comments in local files against the template versions.
   See the `itkdev-docker-templates` skill (Compare Project Against Template section).

   ```bash
   # List template files
   gh api repos/itk-dev/devops_itkdev-docker/contents/templates/{template} --jq '.[].name'

   # Fetch a specific template file for comparison
   gh api repos/itk-dev/devops_itkdev-docker/contents/templates/{template}/{file} --jq '.content' | base64 -d
   ```

3. **List available templates (if detection fails):**
   ```bash
   gh api repos/itk-dev/devops_itkdev-docker/contents/templates --jq '.[].name'
   ```

**Fallback:** If the project type cannot be detected, proceed with the
manual checklist below.

#### Project Type Classification

Use the detected template name to classify the project:

| Detected Template | Project Type | Steps with Conditional Checks |
|-------------------|-------------|-------------------------------|
| `symfony-*`       | Symfony     | Steps 4, 5, 7 include Symfony-specific checks |
| `drupal-*`        | Drupal      | Steps 4, 5, 7 include Drupal-specific checks |
| Other / unknown   | Generic     | Steps 4, 5, 7 use minimal universal checks only |

#### Project Exceptions

Deviations from these standards that are documented in the project's `CLAUDE.md`
or memory files should be treated as acknowledged exceptions, not failures.
When a check fails but the deviation is documented, report it as
**"Exception (documented)"** in the summary rather than a failure.

---

### Step 1 — Docker Compose (`docker-compose.yml`)

Validate the development Docker Compose file. See the `itkdev-docker` skill
for full architecture details.

#### Networks

- [ ] `frontend` network declared as `external: true`
- [ ] `app` network declared with `driver: bridge`

#### Images

- [ ] `phpfpm` uses `itkdev/php8.4-fpm:latest` (or appropriate PHP version)
- [ ] `mariadb` uses `itkdev/mariadb:latest`
- [ ] `nginx` uses `nginxinc/nginx-unprivileged:alpine`
- [ ] If `worker` service is present: uses `itkdev/supervisor-php8.4:alpine`
  (matching PHP version)

#### Service Names

Core services (required):
- [ ] `phpfpm`, `mariadb`, `nginx`

Optional services (check if present):
- [ ] `worker` — only required for projects using background job processing
- [ ] Other project-specific services

#### Healthchecks

- [ ] Every service has a `healthcheck` block
- [ ] Each healthcheck has `start_period`, `interval`, `timeout`, `retries`
- [ ] `phpfpm` healthcheck tests TCP port 9000
- [ ] `mariadb` healthcheck uses `healthcheck.sh --connect --innodb_initialized`
- [ ] `nginx` depends_on phpfpm with `condition: service_healthy`
- [ ] If `worker` service is present: depends_on mariadb and phpfpm with
  `condition: service_healthy`

#### Volumes

- [ ] Application volume mounted as `.:/app` (no `:Z` suffix)
- [ ] Nginx templates from `.docker/templates` (dot-prefix directory)

#### Traefik Labels

- [ ] `traefik.enable=true`
- [ ] `traefik.docker.network=frontend`
- [ ] Router rule uses `${COMPOSE_PROJECT_NAME}` and `${COMPOSE_DOMAIN}`

#### Dev Profile Services

- [ ] `node`, `markdownlint`, `prettier` services have `profiles: [dev]`

> **Note:** Dev profile services are typically defined in
> `docker-compose.override.yml` for local customization.
> If they only appear in the override file, the check passes automatically.

#### Remediation

| Check | Fix |
|-------|-----|
| Wrong image | Replace with `itkdev/php8.4-fpm:latest` etc. |
| Missing healthcheck | Add healthcheck block with all 4 timing fields |
| Volume uses `:Z` | Remove `:Z` suffix — not needed for standard Docker |
| Docker dir not dot-prefixed | Rename `docker/` to `.docker/` |
| Missing Traefik labels | Add labels using `${COMPOSE_PROJECT_NAME}` variables |

---

### Step 2 — Docker Compose Server (`docker-compose.server.yml`)

Validate the production/server Docker Compose override.

- [ ] File exists at project root
- [ ] All services have `restart: unless-stopped`
- [ ] Required variables use `:?` syntax (e.g., `${COMPOSE_PROJECT_NAME:?}`,
  `${COMPOSE_SERVER_DOMAIN:?}`)
- [ ] Traefik labels include HTTPS redirect middleware:
  - Router with `-http` suffix on `entrypoints=web`
  - `redirect-to-https` middleware
  - Router on `entrypoints=websecure`
- [ ] No dev-profile services (`node`, `markdownlint`, `prettier`)
- [ ] `mariadb` service is absent (server uses external/managed database,
  or is present if self-hosted DB is intended)
- [ ] Nginx volumes include `.docker/nginx.conf` mount

#### Remediation

| Check | Fix |
|-------|-----|
| Missing `restart` | Add `restart: unless-stopped` to every service |
| Variables without `:?` | Change `${VAR}` to `${VAR:?}` for required vars |
| No HTTPS redirect | Add `-http` router + `redirect-to-https` middleware labels |

---

### Step 3 — Environment (`.env`)

Validate environment file conventions.

- [ ] `.env` is committed to version control
- [ ] `.env.local` is in `.gitignore`
- [ ] First 3 lines are (in order):
  1. `COMPOSE_PROJECT_NAME=<project-name>`
  2. `COMPOSE_DOMAIN=<project>.local.itkdev.dk`
  3. `COMPOSE_SERVER_DOMAIN=<project>.<domain>`
- [ ] `COMPOSE_DOMAIN` matches `*.local.itkdev.dk` pattern
- [ ] `DATABASE_URL` uses `mariadb` as database hostname
- [ ] No secrets or real API keys in `.env` (only placeholders)

#### Remediation

| Check | Fix |
|-------|-----|
| `.env` not committed | Remove from `.gitignore`, commit with safe defaults |
| Wrong `COMPOSE_DOMAIN` | Set to `<project>.local.itkdev.dk` |
| Real secrets in `.env` | Move to `.env.local`, replace with placeholders |

---

### Step 4 — Taskfile.yml

Validate task runner configuration. See the `itkdev-taskfile` skill for
standard patterns and task definitions.

#### Header

- [ ] `version: "3"`
- [ ] `dotenv: [".env.local", ".env"]` (`.env.local` first for override priority)

#### Standard Variables

- [ ] `DOCKER_COMPOSE` defined (supports override via `CONTAINER_COMPOSE`)
- [ ] `PHP` defined as `{{.DOCKER_COMPOSE}} exec phpfpm`
- [ ] `COMPOSER` defined as `{{.PHP}} composer`

Conditional variables:
- [ ] If Symfony: `CONSOLE` defined as `{{.PHP}} bin/console`
- [ ] If Drupal: `DRUSH` defined as `{{.PHP}} vendor/bin/drush`
- [ ] If project uses Node: `NODE` defined as `{{.DOCKER_COMPOSE}} run --rm node`

#### Required Tasks (all project types)

- [ ] `compose`/`up` — start containers
- [ ] `down` — stop containers
- [ ] A dependency install task (e.g., `install`, `composer:install`)

#### Framework-Specific Tasks

- [ ] If Symfony: `lint:twig`, `lint:twig:fix`, `analyze`
- [ ] If Drupal: `drush`, config import/export tasks

#### Optional / Recommended Tasks

Note as suggestions, not failures:
`setup`, `restart`, `ci`, `lint`, `lint:php`, `lint:php:fix`,
`lint:composer`, `lint:markdown`, `lint:styles`, `test`

#### Remediation

| Check | Fix |
|-------|-----|
| Missing dotenv | Add `dotenv: [".env.local", ".env"]` |
| Wrong var names | Rename to match standard: `DOCKER_COMPOSE`, `PHP`, etc. |
| Missing required task | Add task following the patterns in the `itkdev-taskfile` skill |

---

### Step 5 — Framework-Specific Config

Validate PHP tooling configuration. Checks are conditional on project type.
For project types other than Symfony or Drupal, skip this step and report
as "N/A" in the summary.

#### For Symfony Projects

**PHP CS Fixer (`.php-cs-fixer.dist.php`)**

- [ ] File exists at project root
- [ ] Rules include `@Symfony` and `@PHP84Migration`
  (PHP migration rule matches project PHP version)
- [ ] Finder uses `ignoreVCSIgnored(true)`
- [ ] Header comment references `itk-dev/devops_itkdev-docker` origin

**PHPStan (`phpstan.dist.neon`)**

- [ ] File exists at project root
- [ ] Level is 6 (or higher)
- [ ] `paths` includes `src`
- [ ] `excludePaths` includes `src/Kernel.php`

**PHP Code Conventions**

- [ ] All PHP files have `declare(strict_types=1)`
- [ ] Classes use `final class` pattern (where appropriate)

#### For Drupal Projects

**PHP CS Fixer (`.php-cs-fixer.dist.php`)**

- [ ] File exists at project root
- [ ] Finder targets `web/modules/custom` and `web/themes/custom`
- [ ] Finder uses `ignoreVCSIgnored(true)`

**PHPStan (`phpstan.dist.neon`)**

- [ ] File exists at project root
- [ ] Level is 6 (or higher)
- [ ] `paths` includes `web/modules/custom` and/or `web/themes/custom`

#### Remediation

| Check | Fix |
|-------|-----|
| Missing `.php-cs-fixer.dist.php` | Copy from template, adjust finder for project type |
| Wrong PHPStan level | Set `level: 6` in `phpstan.dist.neon` |
| Wrong PHPStan paths | Adjust `paths` to match project type (`src` for Symfony, `web/modules/custom` for Drupal) |

---

### Step 6 — Composer

Validate Composer configuration.

- [ ] `ergebnis/composer-normalize` is a dev dependency
- [ ] `friendsofphp/php-cs-fixer` is a dev dependency
- [ ] `phpstan/phpstan` is a dev dependency
- [ ] `ergebnis/composer-normalize` is in `allow-plugins`
- [ ] `composer.json` is normalized (run `composer normalize --dry-run`)

#### Remediation

| Check | Fix |
|-------|-----|
| Missing dev dep | `composer require --dev <package>` |
| Not in allow-plugins | Add to `config.allow-plugins` in `composer.json` |
| Not normalized | Run `composer normalize` |

---

### Step 7 — GitHub Actions

Validate CI workflow files. See the `itkdev-gh-actions` skill for available
workflow templates and naming conventions.

#### Expected Workflow Files

Universal (all project types):
- [ ] `.github/workflows/changelog.yaml`
- [ ] `.github/workflows/composer.yaml`
- [ ] `.github/workflows/markdown.yaml`
- [ ] `.github/workflows/php.yaml`
- [ ] `.github/workflows/yaml.yaml`

Symfony additionally:
- [ ] `.github/workflows/twig.yaml`

Projects with JS/CSS assets:
- [ ] `.github/workflows/javascript.yaml`
- [ ] `.github/workflows/styles.yaml`

#### Workflow Content Standards

- [ ] Each file starts with header comment:
  `# Do not edit this file!` (referencing `itk-dev/devops_itkdev-docker`)
- [ ] `env` section includes `COMPOSE_USER: runner`
- [ ] `composer.yaml` additionally has `COMPOSE_DOMAIN: test.itkdev.dk`
- [ ] Steps use `actions/checkout@v5`
- [ ] Steps include `docker network create frontend`
- [ ] Triggers: `pull_request` + `push` to `main` and `develop`
  (exception: `changelog.yaml` triggers only on `pull_request`)

#### Remediation

| Check | Fix |
|-------|-----|
| Missing workflow | Fetch from template repo (see `itkdev-gh-actions` skill) |
| Outdated checkout | Update to `actions/checkout@v5` |
| Missing `COMPOSE_USER` | Add `env: COMPOSE_USER: runner` at workflow level |
| Missing network create | Add step: `docker network create frontend` |

---

### Step 8 — Miscellaneous

Validate supporting files.

- [ ] `CHANGELOG.md` exists at project root
- [ ] `CHANGELOG.md` follows [Keep a Changelog](https://keepachangelog.com/) format
- [ ] `.prettierrc.yaml` exists (or equivalent Prettier config)
- [ ] Markdown lint config exists (`.markdownlint.jsonc`, `.markdownlint.yml`,
  or similar)
- [ ] `.docker/` directory uses dot-prefix (not `docker/`)

#### Remediation

| Check | Fix |
|-------|-----|
| No CHANGELOG.md | Create with Keep a Changelog template |
| No Prettier config | Add `.prettierrc.yaml` from template |
| No markdown lint config | Add config file from template |
| `docker/` not dot-prefixed | Rename to `.docker/` and update all references |

---

## Verification

After completing all checks, produce a summary report in this format:

```
## itk-dev Standards Validation Report

Project: <project name>
Template: <detected template or "manual">
Date: <current date>

### Results

| Area                     | Status | Issues |
|--------------------------|--------|--------|
| Step 0: Template Compare |   /    |   -    |
| Step 1: docker-compose   |   /    |   0    |
| Step 2: Server compose   |   /    |   0    |
| Step 3: Environment      |   /    |   0    |
| Step 4: Taskfile          |   /    |   0    |
| Step 5: Framework Config  |   /    |   0    |
| Step 6: Composer          |   /    |   0    |
| Step 7: GitHub Actions    |   /    |   0    |
| Step 8: Miscellaneous     |   /    |   0    |

### Issues Found

(List any failing checks with remediation steps)

### Documented Exceptions

(List any checks that failed but have documented justification in CLAUDE.md
or project memory files — these are acknowledged deviations, not failures)

### Summary

X/Y checks passed, Z documented exceptions. [COMPLIANT | NEEDS FIXES]
```

## Common Issues

### `docker/` directory instead of `.docker/`

**Cause:** Older itk-dev convention or custom setup.

**Fix:** Rename directory and update all references in `docker-compose.yml`,
`docker-compose.server.yml`, and any Dockerfiles.

### Missing healthchecks on services

**Cause:** Services added without following the template pattern.

**Fix:** Add healthcheck blocks. Use existing services as reference — each
needs `test`, `start_period`, `interval`, `timeout`, and `retries`.

### Taskfile uses `docker compose` directly instead of variables

**Cause:** Taskfile not updated for Podman compatibility.

**Fix:** Replace hardcoded `docker compose` with `{{.DOCKER_COMPOSE}}` and
add the standard vars block.

### GitHub Action workflows edited locally

**Cause:** Fixes applied directly instead of upstream in the template repo.

**Fix:** Revert local changes. If the fix is needed, submit a PR to
`itk-dev/devops_itkdev-docker` and re-copy the workflow file.

### `.env` contains real secrets

**Cause:** API keys or passwords committed by mistake.

**Fix:** Move secrets to `.env.local` (gitignored). Replace values in `.env`
with empty strings or placeholder text. Rotate any exposed credentials.

## Example

**Scenario:** Validate this project after PR #80 alignment

```
## itk-dev Standards Validation Report

Project: accessibility-checker
Template: symfony-7 (auto-detected)
Date: 2026-02-26

### Results

| Area                     | Status | Issues |
|--------------------------|--------|--------|
| Step 0: Template Compare | PASS   |   0    |
| Step 1: docker-compose   | PASS   |   0    |
| Step 2: Server compose   | PASS   |   0    |
| Step 3: Environment      | PASS   |   0    |
| Step 4: Taskfile          | PASS   |   0    |
| Step 5: Framework Config  | PASS   |   0    |
| Step 6: Composer          | PASS   |   0    |
| Step 7: GitHub Actions    | PASS   |   0    |
| Step 8: Miscellaneous     | PASS   |   0    |

### Summary

All checks passed. COMPLIANT
```

## References

- Template repository: <https://github.com/itk-dev/devops_itkdev-docker>
- Related skills:
  - `itkdev-docker` — CLI reference, Compose architecture, project detection
  - `itkdev-docker-templates` — template management, comparison procedures
  - `itkdev-taskfile` — task patterns, coding standards, workflows
  - `itkdev-gh-actions` — workflow templates, naming conventions
- Project CLAUDE.md: conventions and architecture overview
- Taskfile.dev documentation: <https://taskfile.dev>
