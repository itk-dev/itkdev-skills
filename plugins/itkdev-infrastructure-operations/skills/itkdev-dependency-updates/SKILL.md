---
name: itkdev-dependency-updates
description: Update the dependencies of an ITK Dev project (Symfony or Drupal). Two scenarios - a fast CVE/advisory update ("skal gå stærkt") of only the affected packages, or the quarterly maintenance update (kvartalsvis opdatering, cybersikkerhedsaftale) of templates, images, PHP and all dependencies. Use when composer audit, npm audit or importmap:audit reports an advisory, when a CVE or security release is published, or when the user asks for the quarterly update, a cybersikkerhedsaftale update, or a dependency, template or PHP version update.
---

# Dependency updates

This skill is the ITK Dev update procedure. The source is the page
[Opdateringer](https://github.com/itk-dev/documentation/blob/main/docs/teamwork/how_we_work/Opdateringer.md)
in the documentation repo. The Composer rules (direct dependencies, security advisories, abandoned packages and
the cooldown) are in the section "Composer" of the page
[PHP](https://github.com/itk-dev/documentation/blob/main/docs/web/php/php.md#composer). If the pages and this
skill do not agree, the pages are correct.

## Step 1: Choose the scenario

Ask the user which scenario applies, if it is not clear from the request.

| | CVE/advisory ("skal gå stærkt") | Quarterly (cybersikkerhedsaftale) |
| --- | --- | --- |
| Reason | An advisory or CVE for a dependency | The fixed quarterly update |
| Packages | Only the packages with an advisory | All packages |
| Version level | Up to the latest patch | Up to the latest minor |
| Major versions | No | Indirect dependencies and dev tools. Other majors need an agreement with the customer |
| Docker images | Only if the advisory is for the image | Yes, all images |
| Customer approval | No | Yes |

## Rules for both scenarios

- Run PHP, Composer, console and Drush commands in the PHP container (for example
  `docker compose exec phpfpm composer ...` or the project's `task` wrappers). Do not run them on the host.
- Make one pull request for each topic. Add a changelog entry in the format that the project uses.
- Do not fix application bugs and do not add features. The only code changes are the changes that the update needs.
- Commit generated files that the update changes, for example `config/reference.php` in Symfony projects and
  the scaffold files (`web/.htaccess`, `web/index.php`, `web/robots.txt`) in Drupal projects.
- Read the list of changed packages before you commit. Look for downgrades.
- The page tells the developer to use their judgement or to ask the project manager (PL). When this skill says
  "ask the user", that is the decision it refers to. Do not make that decision for the user.

### Composer cooldown

Composer (2.11 and later) holds back new releases for a cooldown period. Our PHP images set it to 7 days
(`COMPOSER_POLICY_COOLDOWN_PERIOD="7 days"`). The cooldown is intentional: a compromised release can be found
before we install it.

- In a CVE update, the fixed version is usually new. Use `--no-cooldown` for the affected packages only.
- Do not use `--no-cooldown` together with `--with-all-dependencies`. Then all packages skip the cooldown. The
  only exception is Drupal core (see CVE step 4).
- Do not change the cooldown in project files. To change it for one command, use
  `docker compose exec -e 'COMPOSER_POLICY_COOLDOWN_PERIOD=24 hours' phpfpm composer ...`.

### Composer audit

After an update, `composer audit` must report no advisories. This applies to both scenarios.

- Update the package to a fixed version. This is the default.
- If an update is not possible, ignore the advisory in `composer.json` with a valid reason. Tell the user the
  reason, and get their approval before you add it:

  ```json
  {
      "config": {
          "policy": {
              "advisories": {
                  "ignore-id": {
                      "CVE-2026-12345": "Only affects the X component, which we don't use. Reviewed 2026-11-18."
                  }
              }
          }
      }
  }
  ```

- Always ignore by ID (`policy.advisories.ignore-id`), not by package name (`policy.advisories.ignore`). Then a
  new advisory on the same package still fails the audit.
- `config.audit` is deprecated. Composer reads the old settings (`audit.ignore`, `audit.ignore-severity` and
  `audit.block-insecure`) only when `config.policy.advisories` is absent. If the project has old `audit`
  settings, move all of them to `policy` in the same change.
- `composer audit` also fails on abandoned packages. Remove or replace the package if that is possible. If it is
  not possible, ignore the package with a reason:

  ```json
  {
      "config": {
          "policy": {
              "abandoned": {
                  "ignore": {
                      "vendor/package": "No replacement exists. We use only the X component. Reviewed 2026-11-18."
                  }
              }
          }
      }
  }
  ```

### Direct dependencies

If the application uses a package directly (classes, functions or configuration), the package must be in
`require` or `require-dev` in `composer.json`. It is not sufficient that the package is installed as an indirect
dependency. In Drupal projects, the rule also applies to all modules and themes in
`config/sync/core.extension.yml`. The rule applies to all PHP code, not only to updates.

## CVE/advisory update

Security updates are mandatory for all projects, also for projects without a "cybersikkerhedsaftale". A project
without an agreement can be on an old minor or patch version that gets no fixes. Then the fix needs a minor or
major update, and that update can need a new PHP version or changes to application code. If the update is not
trivial, stop and tell the user. The user must tell the project manager (PL).

**Included:**

- Only the packages that have an advisory.
- Up to the latest patch version that closes the advisory.
- Dependencies that the update needs to install, but as few as possible.

**Not included:**

- Other packages, also when they have new versions.
- Minor and major updates, unless that is the only way to a fixed version.
- Docker image updates, unless the advisory is for the image.

### Steps

1. Find the affected packages: `composer audit` (and `npm audit`). In Symfony projects with AssetMapper, also
   run `bin/console importmap:audit`. In Drupal projects, `composer audit` also reports advisories for Drupal
   core and modules. Do not use `drush pm:security`. Drush removed it in favour of `composer audit`.
2. Find all projects that use the affected packages, and update all of them. Remember projects that belong
   together, for example an application and its API.
3. Make a branch: `hotfix/x.y.z` from `main`. If a release from `develop` is near, you can make the update
   against `develop`.
4. Update only the affected packages, without the cooldown. Do not use `--with-all-dependencies`. Use a
   wildcard for packages that are released together, for example Symfony:

   ```sh
   composer update 'symfony/*' --no-cooldown
   ```

   Drupal core is the exception, because `drupal/core-recommended` locks the dependencies of core:

   ```sh
   composer update 'drupal/core-*' --with-all-dependencies --no-cooldown
   ```

   This command skips the cooldown for all dependencies of core. Packages without a fixed version (for example
   from `drupal/core-dev`) can get new versions. Read the list of changes. If such a package was released in
   the cooldown period, ask the user if it is necessary for the fix. Then run `drush updatedb` and
   `drush config:export` (see [Drupal](#drupal)).

5. Run `composer audit` again. It must report no advisories. If a package cannot be updated, ignore the
   advisory by ID with a reason (see [Composer audit](#composer-audit)).
6. Commit `composer.lock` and the generated files that changed.
7. Run the tests, and make sure that CI passes.
8. Add a changelog entry and make a pull request.
9. Make a release: tag on `main`, then merge `main` back into `develop`.

A fixed version that you installed with `--no-cooldown` stays in the cooldown for 7 days. A quarterly update in
that period needs a shorter cooldown (see quarterly step 4).

## Quarterly update

The quarterly update is maintenance only: old templates, PHP versions, images and dependencies.

**Included:**

- All direct dependencies (Composer, npm and others) up to the latest minor or patch in their current major
  version.
- Indirect dependencies (dependencies of dependencies) can get major updates. They follow the constraints of
  the packages that require them.
- Dev tools (`require-dev`, for example PHPStan, Rector and PHPUnit) can get major updates. A trivial update is
  included if there is time after the mandatory tasks. Ask the user.
- All Docker images.
- Code changes that the update needs.
- Simple deprecations (see [Deprecations](#deprecations)).

**Not included:**

- Fixes and changes in the application.
- New features.
- Major versions of frameworks and of direct dependencies (dev tools excepted). These need an agreement with
  the customer.

### Step 1: Update the docker compose files from the templates

```sh
itkdev-docker-compose self:update
itkdev-docker-compose template:update
```

Do this step first, because it can change the PHP version. The PHP version controls which package versions
Composer can install.

- `template:update` uses `ITKDEV_TEMPLATE` from `.env` (for example `symfony-7` or `drupal-11`). If an old
  project does not have it, add it to `.env`. Make sure that the template agrees with the major version of the
  project.
- `template:update` does not merge. It copies the template files over the project files (`docker-compose*.yml`,
  `.docker/`, `.github/workflows/` and configuration files such as `.php-cs-fixer.dist.php` or
  `.phpcs.xml.dist`). Read the full diff.
- Restore project specific changes (`git checkout -- <file>` or `git add -p`). Move them to files that the
  template does not overwrite: `docker-compose.override.yml`, `docker-compose.server.override.yml`, or
  `.docker/templates/server.d/*.conf.template` for nginx.
- Keep the improvements from the template.
- Delete new files that the project does not use (for example `javascript.yaml` and `styles.yaml` workflows in a
  project without `assets/`).
- Delete old project files that the template now replaces (for example one combined `pr.yaml` workflow, or
  `phpcs.xml.dist` when the template has `.phpcs.xml.dist`).

The template always has the PHP version that we recommend. Accept the PHP upgrade if the dependencies of the
application can run on the new version. Make the PHP upgrade and the Rector changes in a separate pull request.
If the PHP version changes:

- Update other references to the PHP version that the template does not control (for example
  `itkdev/supervisor-php8.4` in `docker-compose.server.override.yml` and cache keys in GitHub workflows).
- Update `require.php` in `composer.json`, then run `composer update --lock`.
- Pull and start the new images (`docker compose pull` and `docker compose up -d`), then run the tests and the
  code analysis.
- Run Rector (it now uses the rules for the new PHP version), and apply the changes.
- Run all linters again. The template can change their configuration (for example `.prettierrc.yaml`).

### Step 2: Update the other images

Update images that the templates do not control (for example in `docker-compose.override.yml` and
`docker-compose.server.yml`) to the latest minor or patch in the same major version.

- Images without a version (for example `mariadb:latest`, `memcached:alpine`) get the latest release with
  `docker compose pull`. They can change major version, and nobody chose it. Tell the user if that occurs.
- Images with only a major version in the tag (for example `rabbitmq:4`, `node:22`) get the latest minor or
  patch with `docker compose pull`.
- Images that are only used in development (for example `node` with `profiles: [dev]`) can go to the latest LTS
  major version.
- If an image must match a version in a different place (for example a shared Elasticsearch cluster), update it
  in all repositories at the same time.
- If the database version changes, also update `serverVersion` in `DATABASE_URL` (for example
  `serverVersion=10.11.19-MariaDB`). Doctrine uses the value to select SQL and types.

Pull and start the new images, then run the tests.

### Step 3: Check the direct dependencies

Do this before you update the dependencies (see [Direct dependencies](#direct-dependencies)). If a package is
missing, an indirect package can get a major version during the update, and nobody chose it. Use
[`pb30/phpstan-composer-analysis`](https://github.com/pb30/phpstan-composer-analysis) to find missing packages.
Add each missing package with its installed version, so that `composer.lock` does not change:

```sh
composer require --no-update vendor/package:^<installed version>
composer update --lock
```

### Step 4: Update the dependencies

```sh
composer update --with-all-dependencies
```

- The versions from the last 7 days are not in the update, because of the cooldown. This is intended.
- If a package was updated with `--no-cooldown` less than 7 days ago, this command fails with "could not be
  resolved". The fixed version is still in the cooldown, and the older versions have an advisory. Use a shorter
  cooldown for this command (see [Composer cooldown](#composer-cooldown)).
- If `composer audit` fails on a package with a CVE, and the fixed version is still in the cooldown, update
  only that package with `--no-cooldown`.
- In Drupal projects, also update the database and the configuration (see [Drupal](#drupal)).
- Update large packages with many changes (for example API Platform) in a separate pull request.
- If the project has a committed API specification (for example OpenAPI from API Platform), export it again
  after the update and read the diff. A minor version can change the API for the clients. Change the code so
  that the API does not change.
- Run the tests after the update, also for minor and patch versions. A minor version can change behaviour
  without an error (for example how a library reads a wrong `@var` type). Only a test finds such a change.

Also update the JavaScript dependencies to the latest minor or patch:

- npm: `npm update` follows the constraints in `package.json`. Use `npm outdated` to see what remains.
- Symfony AssetMapper: see [Symfony](#symfony).

### Step 5: Update Rector and PHPStan

Do this in a separate pull request, because new rules change code and give new errors.

- Read the Rector changes before you apply them. Remove rules from `withSkip()` that Rector reports as unknown.
- Generate the PHPStan baseline again (`--generate-baseline`). It can only lose lines. Fix new errors, or ignore
  them with a comment (see [Deprecations](#deprecations)). Do not add them to the baseline.

### Step 6: Update indirect major versions

Update indirect packages that have a new major version (for example a package that is only in `composer.json`
to control the version), one package for each pull request. Do a `--dry-run` first, to see which other packages
must change.

### Step 7: Do the framework steps

Do the steps that apply only to the framework of the project: [Symfony](#symfony) or [Drupal](#drupal).

### Step 8: Check and test before the release

- Run `composer outdated --direct --minor-only` and, in Symfony projects, `composer recipes --outdated`. Only
  versions that are still in the cooldown can remain.
- Run `composer audit`. It must report no advisories and no abandoned packages (see
  [Composer audit](#composer-audit)).
- Drupal: run `drush config:status`. The configuration must have no changes that are not exported.
- Run PHPStan with `phpstan/phpstan-deprecation-rules` (see [Deprecations](#deprecations)).
- Run the full test suite with coverage, as in CI (for example `task test`). Some warnings only occur with
  coverage, for example `#[CoversClass]` on a class outside `src/`.

### Step 9: Make the release

1. Make `release/x.y.0` from `develop`.
2. Move the changelog entries from `[Unreleased]` to `## [x.y.0] - <date>`, and update the links at the end of the
   changelog.
3. Make a pull request against `main`. If the project has a staging pipeline, the release branch goes to staging.
   Test it there.
4. Merge the pull request, and tag `x.y.0` on `main`. The tag deploys to production.
5. Merge `main` back into `develop`.
6. Do the steps that must occur after the deploy (for example a re-index or a manual correction of data). Write
   them in the release pull request.

## Symfony

### Flex recipes

```sh
composer recipes --outdated
composer recipes:update vendor/package
```

- Update one recipe at a time, and commit each recipe. `recipes:update` needs a clean git index.
- Keep project specific configuration (for example `security.yaml` and `messenger.yaml`) and files from the
  ITK Dev templates (for example `.php-cs-fixer.dist.php` and `.editorconfig`).
- If a recipe removes a setting, make sure that the setting is now the default. Else the behaviour changes (for
  example `translator.fallbacks`).
- Make sure that the routes and the services did not change (`debug:router`, `lint:container`).
- The PHPUnit recipe enables `failOnDeprecation` for deprecations from the project's own code. Fix them first, in
  a separate pull request that is merged before the recipes.

### AssetMapper

Projects with `importmap.php` have their JavaScript packages in the importmap, not in npm. The importmap has no
version constraints, and `importmap:update` goes to the latest version, also a new major. Find the new
versions, and update each package to a specified version:

```sh
bin/console importmap:outdated
bin/console importmap:require vendor-package@1.2.3
```

### Doctrine

A new Doctrine version can change how types and columns map to the database. After the update, the schema must
still agree with the migrations:

```sh
bin/console doctrine:schema:validate
bin/console doctrine:migrations:diff
```

`doctrine:migrations:diff` must not make a new migration. If it does, find the cause before the update is
merged.

## Drupal

Drush is the standard in all our Drupal projects.

### Drupal core

`drupal/core-recommended` locks the dependencies of core to fixed versions. Update all core packages together,
with their dependencies:

```sh
composer update 'drupal/core-*' --with-all-dependencies
```

### Database and configuration

After each update of core or of modules:

1. Run the database updates: `drush updatedb --yes`.
2. Clear the cache: `drush cache:rebuild`.
3. Export the configuration: `drush config:export --yes`. Updates can change the configuration.
4. Read the changes in `config/sync`, and commit them together with `composer.lock`.

Do not run `drush deploy` (or, for example, `task site-update`) locally before the configuration is exported.
`config:import` overwrites the changes that the database updates made to the configuration.

### Patches

Projects with `cweagans/composer-patches` have patches in `composer.json` (`extra.patches`). A patch can fail
after an update, or the new version can include it. Read the issue for each patch that fails. Remove patches
that the new version includes.

### Deprecations in Drupal

- [Upgrade Status](https://www.drupal.org/project/upgrade_status) finds deprecations in custom modules, and
  shows if contrib modules are ready for the next major version (`drush upgrade_status:analyze --all`).
- `mglaman/phpstan-drupal` prepares PHPStan for Drupal.
- Rector uses the rules from [`palantirnet/drupal-rector`](https://github.com/palantirnet/drupal-rector).

### Tests

Many Drupal projects have few tests. The `site.yaml` workflow from the template installs the site from the
configuration (`drush site:install --existing-config`). This must also work after the update. Ask the user to
test the most important pages in the browser.

## Deprecations

Major updates are not part of the quarterly update, so the removal of deprecations is not part of it either.
But it depends on the quantity of work:

- **Included:** deprecations that Rector fixes. Let Rector do it.
- **Included:** trivial fixes, for example a one-line configuration change or a new namespace in a `use`
  statement.
- **Not included:** deprecations that need changes to application code. Write them down for the next major
  upgrade.

"Included" is not mandatory. It depends on the time that remains after the mandatory tasks. Ask the user.

The tests only find deprecations that occur when the code runs (PHPUnit `failOnDeprecation`). A class or method
that only has `@deprecated` in a docblock does not trigger a deprecation, so the tests and Rector do not find
it. Example: `Symfony\Component\Serializer\Annotation\Groups` (deprecated in Symfony 7.4, use
`Attribute\Groups`).

To find them, use [`phpstan/phpstan-deprecation-rules`](https://github.com/phpstan/phpstan-deprecation-rules):

```sh
composer require --dev phpstan/phpstan-deprecation-rules
```

Fix the deprecations that are included. Do not put the other deprecations in the PHPStan baseline. The baseline
is only for the errors that were there when PHPStan came into the project, and it can only become smaller.
Ignore new errors under `ignoreErrors` in `phpstan.dist.neon`, with a comment that gives the reason:

```neon
parameters:
    ignoreErrors:
        # Order::Ascending is deprecated in favour of \SortDirection, which needs PHP 8.6
        - message: '#Order::Ascending#'
```

## Rector

We recommend [Rector](https://getrector.com/) as a dev dependency in PHP projects
(`composer require --dev rector/rector`). Configure it in `rector.php` so that the rules follow the versions of
the project:

- `withPhpSets()` uses the PHP version from `composer.json` (`require.php`).
- `withComposerBased(...)` uses the installed versions of, for example, Symfony, Doctrine and PHPUnit.

Run `vendor/bin/rector process --dry-run` in CI. Read the changes before you commit them, and use `withSkip()`
for rules that are not applicable to the project.

## PHP version

Projects with a "cybersikkerhedsaftale" must always be on a PHP version with
["Active Support"](https://www.php.net/supported-versions.php). The templates only set the version of the PHP
images. Other services (for example PostgreSQL, RabbitMQ and Elasticsearch) are in the project's own docker
compose files, and step 2 of the quarterly update covers them.

## Things to never do

- Do not update frameworks or direct dependencies to a new major version in the quarterly update. This needs an
  agreement with the customer. Dev tools are the exception.
- Do not use `--no-cooldown` for more packages than the advisory affects.
- Do not ignore an advisory by package name, and do not ignore an advisory or an abandoned package without a
  reason.
- Do not add application fixes or features to an update pull request. Make a separate pull request.
- Do not make the PHPStan baseline larger to make an update pass.
- Do not run `drush deploy` locally before the configuration is exported.
