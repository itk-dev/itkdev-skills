---
name: itkdev-docker
description: Docker development environment for ITK Dev projects. Use when working with Docker, running containers, debugging Docker issues, setting up local dev, using itkdev-docker-compose commands, Traefik reverse proxy, or server deployments.
---

# ITK Dev Docker Development Environment

You are assisting with Docker-based development in ITK Dev projects. This skill covers the `itkdev-docker-compose` CLI, Docker Compose architecture, service configuration, and server deployments.

**IMPORTANT:** ITK Dev projects run in Docker containers. Never run PHP, Composer, Drush, or other project commands directly on the host. Always use `itkdev-docker-compose` or Taskfile tasks.

## itkdev-docker-compose CLI Reference

The `itkdev-docker-compose` script wraps Docker Compose and provides project-specific commands. It auto-discovers `docker-compose.yml` by searching parent directories.

### Project Operations

```bash
itkdev-docker-compose url [service [port]]      # Print project URL
itkdev-docker-compose open [service [port]]      # Open project in browser
itkdev-docker-compose shell [service]            # Enter container shell
itkdev-docker-compose down                       # Stop and remove all containers
itkdev-docker-compose images:pull                # Pull latest Docker images
itkdev-docker-compose hosts:insert               # Add domain to /etc/hosts
itkdev-docker-compose version                    # Show script version
itkdev-docker-compose completions                # Generate shell completions
```

### PHP/Composer Commands (run inside phpfpm container)

```bash
itkdev-docker-compose composer install
itkdev-docker-compose composer require drupal/module_name
itkdev-docker-compose php script.php
itkdev-docker-compose vendor/bin/phpcs           # Any vendor/bin command
itkdev-docker-compose vendor/bin/phpunit
```

### Drupal-Specific

```bash
itkdev-docker-compose drush cr                   # Clear cache
itkdev-docker-compose drush cex -y               # Export config
itkdev-docker-compose drush cim -y               # Import config
itkdev-docker-compose drush updb -y              # Run database updates
```

Drush is auto-detected (composer vs container binary).

### Database Operations

```bash
itkdev-docker-compose sync:db                    # Sync database from remote
itkdev-docker-compose sync:files                 # Sync files from remote
itkdev-docker-compose sync                       # Sync both db and files
itkdev-docker-compose sql:cli [args]             # MySQL CLI
itkdev-docker-compose sql:connect                # Print MySQL connection string
itkdev-docker-compose sql:open                   # Open in database GUI
itkdev-docker-compose sql:port                   # Get exposed MySQL port
itkdev-docker-compose sql:log                    # Enable SQL query logging
```

### Xdebug

```bash
itkdev-docker-compose xdebug                     # Enable Xdebug, wait for Ctrl-C
```

Common `XDEBUG_MODE` values: `off` (default), `debug` (step debugging), `coverage`, `debug,coverage`.

### Mail

```bash
itkdev-docker-compose mail:url                   # Get Mailpit URL
itkdev-docker-compose mail:open                  # Open Mailpit in browser
```

### Traefik Reverse Proxy

```bash
itkdev-docker-compose traefik:start              # Start Traefik
itkdev-docker-compose traefik:stop               # Stop Traefik
itkdev-docker-compose traefik:url                # Get Traefik admin URL
itkdev-docker-compose traefik:open               # Open admin in browser
itkdev-docker-compose traefik:pull               # Pull latest Traefik images
itkdev-docker-compose traefik:logs               # View Traefik logs
```

Traefik provides reverse proxy routing with wildcard SSL for `*.local.itkdev.dk`. The script warns if Traefik is not running.

### Template Management

```bash
itkdev-docker-compose template:install [name] [--force] [--list]  # Install template
itkdev-docker-compose template:update [--force]                    # Update template
itkdev-docker-compose self:update                                  # Update CLI + devops repo
```

## Docker Compose Architecture

### File Layering

Projects use multiple Compose files that are chained automatically:

| File | Purpose |
|---|---|
| `docker-compose.yml` | Base configuration (services, networks, volumes) |
| `docker-compose.dev.yml` | Development overrides (ports, debug tools) |
| `docker-compose.server.yml` | Server/production overrides |
| `docker-compose.redirect.yml` | SSL redirect configuration |
| `docker-compose.override.yml` | Local developer overrides (gitignored) |

All versioned compose files contain `# itk-version: X.Y.Z` comments for tracking.

### Standard Services

| Service | Image | Purpose |
|---|---|---|
| **phpfpm** | `itkdev/php8.x-fpm` | PHP-FPM application server |
| **nginx** | `nginxinc/nginx-unprivileged:alpine` | Web server |
| **mariadb** | `itkdev/mariadb` | Database with healthcheck |
| **memcached** | `memcached:alpine` | Cache (Drupal projects) |
| **mail** | `axllent/mailpit` | Mail catcher for development |
| **markdownlint** | Dev profile | Markdown linting |
| **prettier** | Dev profile | Code formatting |

### Networks

- **frontend**: External network shared with Traefik for routing
- **app**: Internal bridge network for inter-service communication

### Key Environment Variables

**phpfpm service:**
- `PHP_XDEBUG_MODE` - Xdebug mode (off/debug/coverage)
- `PHP_MAX_EXECUTION_TIME` - PHP execution timeout
- `PHP_MEMORY_LIMIT` - PHP memory limit
- `PHP_SENDMAIL_PATH` - msmtp mail integration
- `DOCKER_HOST_DOMAIN` - Host machine domain for Xdebug
- `PHP_IDE_CONFIG` - IDE server name for debugging
- `DRUSH_OPTIONS_URI` - Drush base URL (Drupal)
- `COMPOSE_USER` - Container user (defaults to `deploy`)

**nginx service:**
- `NGINX_FPM_SERVICE` - PHP-FPM service name
- `NGINX_WEB_ROOT` - Document root (`/app/web` for Drupal, `/app/public` for Symfony)
- `NGINX_PORT` - Listen port
- `NGINX_MAX_BODY_SIZE` - Upload size limit

**mariadb service:**
- `MYSQL_ROOT_PASSWORD`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE`

### Traefik Labels

Nginx service uses Traefik labels for routing:
- Host routing: `` Host(`${COMPOSE_DOMAIN}`) ``
- Mailpit routing: `` Host(`mail-${COMPOSE_DOMAIN}`) ``
- Optional HTTPS redirect and metrics auth (`ITKMetricsAuth@file`)

### Nginx Configuration

Projects include `.docker/nginx.conf` (main config) and `.docker/templates/default.conf.template` (vhost template) for nginx.

## .env Configuration

```bash
COMPOSE_PROJECT_NAME=myproject            # Required: Docker project name
COMPOSE_DOMAIN=myproject.docker.localhost # Optional: defaults to {project}.docker.localhost
COMPOSE_USER=deploy                       # Container user
ITKDEV_TEMPLATE=drupal-11                 # Template name (auto-set on install)

# Remote sync
REMOTE_HOST=user@server.example.com
REMOTE_DB_DUMP_CMD="drush sql:dump"
REMOTE_PATH=/data/www/project/htdocs
LOCAL_PATH=./web
REMOTE_EXCLUDE="css\njs\nstyles"
SYNC_DB_POST_SCRIPT=./scripts/post-sync.sh
```

The CLI sources both `.env` and `.env.local` (for local overrides).

## Framework Differences

| Feature | Drupal | Symfony |
|---|---|---|
| Web root | `/app/web` | `/app/public` |
| Memcached | Yes | No |
| Drush | Yes | No |
| Prettier config | No | Yes (YAML) |

## Server Deployment

Production uses `docker-compose.server.yml` with the `itkdev-docker-compose-server` CLI variant. Server compose files configure production-appropriate settings (no dev tools, proper resource limits).

## Project Detection (Procedural)

When you need to detect an ITK Dev Docker project, read these files:

1. **`docker-compose.yml`**: Look for `# itk-version:` comment and PHP image version (e.g., `itkdev/php8.4-fpm`)
2. **`.env`**: Read `ITKDEV_TEMPLATE`, `COMPOSE_PROJECT_NAME`, `COMPOSE_DOMAIN`
3. **Framework detection**:
   - Drupal: `web/modules` directory, `*.info.yml` files, `drupal/core` in `composer.json`
   - Symfony: `config/bundles.php`, `symfony/*` packages in `composer.json`
4. **Services**: Parse `docker-compose.yml` for service names under `services:` key
5. **Extras**: Check for `Taskfile.yml`, `.github/workflows/` directory

## Template Comparison

For template comparison procedures, see the **"Procedural: Compare Project Against Template"** section in the `itkdev-docker-templates` skill.

## Troubleshooting

- **Traefik not running**: Run `itkdev-docker-compose traefik:start`
- **Port conflicts**: Check for other services on ports 80/443, or use `docker ps`
- **Container won't start**: Check `docker compose logs <service>` for errors
- **Database connection refused**: Ensure mariadb healthcheck passes before phpfpm starts
- **Xdebug not connecting**: Verify `DOCKER_HOST_DOMAIN` points to host (`host.docker.internal` on macOS)
- **Wrong path mappings**: IDE path mappings must match Docker volume mounts (typically `/app`)
- **Performance issues**: Set `XDEBUG_MODE=off` when not debugging
