---
name: itkdev-symfony
description: Symfony development assistance for ITK Dev projects. Use when working with Symfony codebases, scaffolding Symfony projects, configuring databases, running console commands, or managing Symfony configuration.
---

# Symfony Development Guide

You are assisting with Symfony development in ITK Dev Docker projects. This skill covers project scaffolding, database configuration, console commands, and coding standards.

## Project Detection

Recognize Symfony projects by:
- `config/bundles.php` file
- `symfony/*` packages in `composer.json`
- `public/index.php` entry point
- `.php-cs-fixer.dist.php` config file
- `docker-compose.yml` with ITK Dev Docker setup

## Docker and Taskfile

**IMPORTANT:** ITK Dev projects run in Docker containers. Never run PHP, Composer, or console commands directly on the host.

For Docker environment details (CLI commands, services, configuration), see the `itkdev-docker` skill.
For Taskfile automation (coding standards, site management), see the `itkdev-taskfile` skill.
For project templates and setup, see the `itkdev-docker-templates` skill.

**Quick reference:**

```bash
# Via docker compose
docker compose exec phpfpm bin/console cache:clear
docker compose exec phpfpm composer install

# Via Taskfile (preferred if available)
task                                         # List available tasks
task console -- cache:clear                  # Run console commands
```

## Project Scaffolding

To scaffold a Symfony project inside the Docker container:

```bash
docker compose exec phpfpm composer create-project symfony/skeleton:"<version>.*" /tmp/project && docker compose exec phpfpm cp -rT /tmp/project /app
```

### Restore Docker `.env`

The Symfony skeleton includes its own `.env` file, so `cp -rT` overwrites the Docker `.env`. After the copy, you MUST restore the Docker Compose variables (`COMPOSE_PROJECT_NAME`, `COMPOSE_DOMAIN`, `ITKDEV_TEMPLATE`) by prepending them back to the `.env` file. Because the `.env` is already overwritten at this point, `docker compose` will fail with `required variable COMPOSE_DOMAIN is missing a value` unless you pass the variables inline:

```bash
COMPOSE_DOMAIN=<name>.local.itkdev.dk COMPOSE_PROJECT_NAME=<name> docker compose exec phpfpm bash -c 'printf "COMPOSE_PROJECT_NAME=<name>\nCOMPOSE_DOMAIN=<name>.local.itkdev.dk\nITKDEV_TEMPLATE=<template>\n\n" > /tmp/env_header && cat /tmp/env_header /app/.env > /tmp/env_combined && mv /tmp/env_combined /app/.env'
```

Replace `<name>` and `<template>` with the actual project name and template used.

### Post-scaffold `composer install`

After scaffolding, run `composer install` to ensure the vendor directory and autoloader are fully initialized:

```bash
docker compose exec phpfpm composer install
```

This prevents extraction errors (e.g. `RecursiveDirectoryIterator` failures) that can occur when requiring new packages into a partially-initialized vendor directory from `cp -rT`.

## Database Configuration

When the project includes the MariaDB Docker service:

1. **Install Doctrine ORM:**
   ```bash
   docker compose exec phpfpm composer require symfony/orm-pack
   ```
   This should work cleanly as long as `composer install` was run first (see above).

2. **Set the DATABASE_URL:** The Doctrine recipe defaults to a PostgreSQL connection string. Replace it with the MariaDB container credentials from `docker-compose.yml`. The MariaDB service uses:
   - Host: `mariadb` (the Docker service name)
   - Port: `3306`
   - User: `db`
   - Password: `db`
   - Database: `db`

   Set the `DATABASE_URL` in `.env` to:
   ```
   DATABASE_URL="mysql://db:db@mariadb:3306/db?serverVersion=10.11.2-MariaDB&charset=utf8mb4"
   ```

3. **Verify the connection:**
   ```bash
   docker compose exec phpfpm bin/console doctrine:database:create --if-not-exists
   ```

## Console Commands

Always run console commands inside the container:

```bash
# Cache
docker compose exec phpfpm bin/console cache:clear

# Database
docker compose exec phpfpm bin/console doctrine:migrations:migrate
docker compose exec phpfpm bin/console doctrine:schema:validate

# Debug
docker compose exec phpfpm bin/console debug:router
docker compose exec phpfpm bin/console debug:container
```

## Coding Standards

Symfony projects use [PHP CS Fixer](https://github.com/PHP-CS-Fixer/PHP-CS-Fixer) (configured via `.php-cs-fixer.dist.php`):

```bash
# Fix code
docker compose run --rm phpfpm vendor/bin/php-cs-fixer fix

# Check code (dry-run)
docker compose run --rm phpfpm vendor/bin/php-cs-fixer fix --dry-run --diff
```

## ITK Dev Conventions

When working on ITK Dev Symfony projects:
- **Always use Docker**: Run all commands via `docker compose` or Taskfile tasks
- **Check Taskfile.yml first**: Use `task` to list available tasks before running raw commands
- Follow `itkdev-github-guidelines` for commits and PRs
- Check for project-specific `CLAUDE.md` guidelines
- Run `task ci` (or equivalent) before creating PRs to ensure code quality
