---
name: itkdev-drupal
description: Drupal development assistance for ITK Dev projects. Use when working with Drupal 10/11 codebases, creating modules/themes, running drush commands, auditing code quality, or managing configuration. Activates for projects with *.info.yml files or web/modules structure.
---

# Drupal Development Guide

You are assisting with Drupal 10/11 development. This skill covers code auditing, module/theme development, drush commands, and configuration management.

## Project Detection

Recognize Drupal projects by:
- `*.info.yml` files in modules/themes
- `web/modules` or `web/themes` directory structure
- `composer.json` with `drupal/core` dependency
- `docker-compose.yml` with ITK Dev Docker setup
- `Taskfile.yml` with defined tasks

## Docker and Taskfile

**IMPORTANT:** ITK Dev projects run in Docker containers. Never run PHP, Composer, Drush, or other project commands directly on the host.

For Docker environment details (CLI commands, services, configuration), see the `itkdev-docker` skill.
For Taskfile automation (coding standards, site management, asset building), see the `itkdev-taskfile` skill.
For project templates and setup, see the `itkdev-docker-templates` skill.

These skills are loaded automatically when relevant. Below is a quick reference for the most common Drupal-specific commands:

**Quick reference:**

```bash
# Via itkdev-docker-compose
itkdev-docker-compose drush cr              # Clear cache
itkdev-docker-compose drush cex -y          # Export config
itkdev-docker-compose drush cim -y          # Import config
itkdev-docker-compose composer install      # Install dependencies

# Via Taskfile (preferred if available)
task                                         # List available tasks
task site-update                             # Update site after code changes
```

## Code Audit

When reviewing Drupal code, check for:

### Drupal Coding Standards
- Follow [Drupal.org coding standards](https://www.drupal.org/docs/develop/standards)
- Use proper namespacing: `Drupal\module_name\...`
- Service-based architecture over procedural code
- Dependency injection in classes

### Security Review
- **SQL Injection**: Use Database API with placeholders, never concatenate user input
- **XSS Prevention**: Use `#markup` with `Xss::filter()`, `#plain_text`, or Twig auto-escaping
- **CSRF Protection**: Use Form API with tokens for state-changing operations
- **Access Control**: Implement proper permission checks
- **Input Sanitization**: Use `\Drupal\Component\Utility\Html::escape()`

### Debug Code Detection
Flag and remove before commit:
- `var_dump()`, `print_r()`, `dd()`
- `dpm()`, `dsm()`, `kint()`
- `\Drupal::logger()` with debug level in production code

### Deprecated API Detection

**Drupal 10 Deprecations** (removed in 11):
- `drupal_set_message()` → use `\Drupal::messenger()->addMessage()`
- `file_unmanaged_*` functions → use file system service
- `entity_load()` → use entity type manager
- `db_query()` → use Database service

**Drupal 11 Changes**:
- Symfony 7 compatibility required
- PHP 8.3+ required
- Check `*.info.yml` for `core_version_requirement`

### Code Quality
- SOLID principles in custom code
- Single responsibility for services
- Proper use of hooks vs. event subscribers
- Render arrays over direct HTML
- Configuration over hardcoded values

## Module Development

### Creating a New Module

Basic structure:
```
modules/custom/module_name/
├── module_name.info.yml
├── module_name.module
├── module_name.services.yml
├── module_name.routing.yml
├── module_name.permissions.yml
├── src/
│   ├── Controller/
│   ├── Form/
│   ├── Plugin/
│   └── Service/
├── config/
│   ├── install/
│   └── schema/
└── templates/
```

### info.yml Template
```yaml
name: 'Module Name'
type: module
description: 'Module description'
package: Custom
core_version_requirement: ^10 || ^11
dependencies:
  - drupal:node
```

### Service Definition
```yaml
services:
  module_name.example_service:
    class: Drupal\module_name\Service\ExampleService
    arguments: ['@entity_type.manager', '@current_user']
```

## Theme Development

### Creating a New Theme

Basic structure:
```
themes/custom/theme_name/
├── theme_name.info.yml
├── theme_name.libraries.yml
├── theme_name.theme
├── css/
├── js/
├── images/
└── templates/
```

### Theme info.yml Template
```yaml
name: 'Theme Name'
type: theme
description: 'Theme description'
package: Custom
core_version_requirement: ^10 || ^11
base theme: stable9
libraries:
  - theme_name/global-styling
```

## Drush Commands

**Always run drush via itkdev-docker-compose** (or use Taskfile tasks if available):

```bash
# Cache
itkdev-docker-compose drush cr                    # Clear all caches
itkdev-docker-compose drush cc render             # Clear render cache only

# Configuration
itkdev-docker-compose drush cex -y                # Export configuration
itkdev-docker-compose drush cim -y                # Import configuration
itkdev-docker-compose drush config:get system.site  # Get specific config

# Database
itkdev-docker-compose drush sql:dump > backup.sql # Database backup
itkdev-docker-compose sql:cli                     # Database CLI (direct command)

# Updates
itkdev-docker-compose drush updb -y               # Run database updates
itkdev-docker-compose drush entity:updates        # Apply entity schema updates

# Development
itkdev-docker-compose drush en module_name -y     # Enable module
itkdev-docker-compose drush pmu module_name -y    # Uninstall module

# Generate (requires drush generators)
itkdev-docker-compose drush generate module       # Generate module scaffold
itkdev-docker-compose drush generate controller   # Generate controller
```

## Configuration Management

### Workflow
1. Make changes in development environment
2. Export: `task config:export` or `itkdev-docker-compose drush cex -y`
3. Review changes in `config/sync/`
4. Commit configuration files
5. On deployment: `task config:import` or `itkdev-docker-compose drush cim -y`

### Config Split
For environment-specific configuration:
- `config/sync/` - Shared configuration
- `config/dev/` - Development overrides
- `config/prod/` - Production overrides

### Schema Definition
Always define schema for custom configuration in `config/schema/module_name.schema.yml`:
```yaml
module_name.settings:
  type: config_object
  label: 'Module settings'
  mapping:
    enabled:
      type: boolean
      label: 'Enabled'
    api_key:
      type: string
      label: 'API Key'
```

## ITK Dev Conventions

When working on ITK Dev Drupal projects:
- **Always use Docker**: Run all commands via `itkdev-docker-compose` or Taskfile tasks
- **Check Taskfile.yml first**: Use `task` to list available tasks before running raw commands
- Follow `itkdev-github-guidelines` for commits and PRs
- Check for project-specific `CLAUDE.md` guidelines
- Configuration should be exportable and version controlled
- Custom modules go in `web/modules/custom/`
- Custom themes go in `web/themes/custom/`
- Run `task ci` (or equivalent) before creating PRs to ensure code quality
