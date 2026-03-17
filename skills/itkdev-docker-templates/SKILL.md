---
name: itkdev-docker-templates
description: Project template conventions for ITK Dev Docker projects. Use when setting up new projects, installing or updating Docker templates, comparing project config against templates, scaffolding project structure, or asking about available templates.
---

# ITK Dev Docker Project Templates

You are assisting with ITK Dev Docker project templates from the `devops_itkdev-docker` repository. This skill covers template selection, installation, comparison, and project scaffolding.

## Available Templates

| Template | Framework | PHP Image | Web Root | Memcached | Drush |
|---|---|---|---|---|---|
| `drupal-8` | Drupal 8 | itkdev/php8.1-fpm | /app/web | Yes | Yes |
| `drupal-9` | Drupal 9 | itkdev/php8.1-fpm | /app/web | Yes | Yes |
| `drupal-10` | Drupal 10 | itkdev/php8.3-fpm | /app/web | Yes | Yes |
| `drupal-11` | Drupal 11 | itkdev/php8.4-fpm | /app/web | Yes | Yes |
| `drupal-module` | Drupal module | itkdev/php8.3-fpm | /app/web | Yes | Yes |
| `symfony-6` | Symfony 6 | itkdev/php8.1-fpm | /app/public | No | No |
| `symfony-7` | Symfony 7 | itkdev/php8.3-fpm | /app/public | No | No |
| `symfony-8` | Symfony 8 | itkdev/php8.4-fpm | /app/public | No | No |

Base templates (`drupal`, `symfony`) contain shared configuration that version-specific templates extend via symlinks.

## Template Selection Guide

- **New Drupal site**: Use `drupal-11` (or the version matching your Drupal core)
- **Drupal contrib module**: Use `drupal-module`
- **New Symfony app**: Use `symfony-8` (or the version matching your Symfony framework)
- **Existing project**: Match the template to the framework and version in `composer.json`

## Template File Structure

Each template installs these files into the project:

```
project/
├── docker-compose.yml              # Base Docker Compose config
├── docker-compose.server.yml       # Server/production overrides
├── docker-compose.dev.yml          # Development overrides (symlinked)
├── docker-compose.redirect.yml     # SSL redirect config (symlinked)
├── .docker/
│   ├── nginx.conf                  # Main Nginx configuration
│   └── templates/
│       └── default.conf.template   # Nginx vhost template
├── .env                            # Environment variables (template)
├── .github/
│   └── workflows/                  # CI/CD workflow templates
├── .markdownlint.jsonc             # Markdown linting rules
├── .php-cs-fixer.dist.php          # PHP CS Fixer config (Symfony)
├── .phpcs.xml.dist                 # PHP CodeSniffer config (Drupal)
├── .twig-cs-fixer.dist.php         # Twig CS Fixer config
├── .prettierrc.yaml                # Prettier config (Symfony)
└── Taskfile.yml                    # Task automation
```

Not all files are present in every template. Drupal uses `.phpcs.xml.dist`, Symfony uses `.php-cs-fixer.dist.php`.

## Installing a Template

```bash
# List available templates
itkdev-docker-compose template:install --list

# Install a specific template
itkdev-docker-compose template:install drupal-11

# Force reinstall (overwrites existing files)
itkdev-docker-compose template:install drupal-11 --force

# Update an existing template
itkdev-docker-compose template:update
itkdev-docker-compose template:update --force
```

After installation, the `.env` file will contain `ITKDEV_TEMPLATE=<template-name>`.

## Setup Workflows

### New Drupal 11 Project

1. Create project: `composer create-project drupal/recommended-project my-project`
2. `cd my-project`
3. Install template: `itkdev-docker-compose template:install drupal-11`
4. Edit `.env`: set `COMPOSE_PROJECT_NAME` and `COMPOSE_DOMAIN`
5. Start Traefik: `itkdev-docker-compose traefik:start`
6. Start containers: `docker compose up -d`
7. Install Drupal: `itkdev-docker-compose drush site:install --yes`
8. Open site: `itkdev-docker-compose open`

### New Symfony 8 Project

1. Create project: `composer create-project symfony/skeleton my-project`
2. `cd my-project`
3. Install template: `itkdev-docker-compose template:install symfony-8`
4. Edit `.env`: set `COMPOSE_PROJECT_NAME` and `COMPOSE_DOMAIN`
5. Start Traefik: `itkdev-docker-compose traefik:start`
6. Start containers: `docker compose up -d`
7. Open site: `itkdev-docker-compose open`

### Existing Project

1. Identify framework and version from `composer.json`
2. Install matching template: `itkdev-docker-compose template:install <template>`
3. Review and merge generated files with existing configuration
4. Adjust `.env` variables as needed

## Procedural: List Templates

When the user asks to list available templates, run:

```bash
gh api repos/itk-dev/devops_itkdev-docker/contents/templates --jq '.[].name'
```

This returns directory names from the templates folder.

## Procedural: Get Template Files

To list files in a specific template:

```bash
gh api repos/itk-dev/devops_itkdev-docker/contents/templates/{template} --jq '.[].name'
```

For nested directories (like `.docker/` or `.github/`), follow up with:

```bash
gh api repos/itk-dev/devops_itkdev-docker/contents/templates/{template}/{subdir} --jq '.[].name'
```

## Procedural: Get Template File Content

To read a specific template file:

```bash
curl -sL "https://raw.githubusercontent.com/itk-dev/devops_itkdev-docker/develop/templates/{template}/{file}"
```

Or via GitHub CLI:

```bash
gh api repos/itk-dev/devops_itkdev-docker/contents/templates/{template}/{file} --jq '.content' | base64 -d
```

## Procedural: Compare Project Against Template

When the user asks to compare their project against a template:

1. **Detect template**: Read `.env` for `ITKDEV_TEMPLATE` value
2. **List template files**: Fetch file list from GitHub (see above)
3. **For each key file**, compare local vs template:

   Key files to check:
   - `docker-compose.yml`
   - `docker-compose.server.yml`
   - `.docker/nginx.conf`
   - `.docker/templates/default.conf.template`
   - `Taskfile.yml`
   - `.github/workflows/*`
   - Config files (`.phpcs.xml.dist`, `.markdownlint.jsonc`, etc.)

4. **Version comparison**: Look for `# itk-version: X.Y.Z` comments in compose files. Compare local version against template version.

5. **Report**:
   - **Missing files**: Template files not present in project
   - **Outdated files**: Local `itk-version` is older than template
   - **Matching files**: Local version matches template
   - **Recommendations**: Suggest updating outdated files, adding missing files
