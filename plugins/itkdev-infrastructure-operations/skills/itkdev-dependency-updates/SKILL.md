---
name: itkdev-dependency-updates
description: Update the dependencies of an ITK Dev project. Two scenarios - a fast CVE/advisory update ("skal gå stærkt") of only the affected packages, or the quarterly maintenance update (kvartalsvis opdatering, cybersikkerhedsaftale) of templates, images, PHP and all dependencies. Use when composer audit or npm audit reports an advisory, when a CVE or security release is published, or when the user asks for the quarterly update, a cybersikkerhedsaftale update, or a dependency, template or PHP version update.
---

# Dependency updates

This skill is the ITK Dev update procedure. The source is the page
[Opdateringer](https://github.com/itk-dev/documentation/blob/main/docs/teamwork/how_we_work/Opdateringer.md)
in the documentation repo. If the page and this skill do not agree, the page is correct.

## Step 1: Choose the scenario

Ask the user which scenario applies, if it is not clear from the request.

| | CVE/advisory ("skal gå stærkt") | Quarterly (cybersikkerhedsaftale) |
| --- | --- | --- |
| Reason | An advisory or CVE for a dependency | The fixed quarterly update |
| Packages | Only the packages with an advisory | All packages |
| Version level | Up to the latest patch | Up to the latest minor |
| Major versions | No | Only transitive dependencies. Direct majors need an agreement with the customer |
| Docker images | Only if the advisory is for the image | Yes, all images |
| Customer approval | No | Yes |

## Rules for both scenarios

- Run PHP, Composer and console commands in the PHP container (for example
  `docker compose exec phpfpm composer ...` or the project's `task` wrappers). Do not run them on the host.
- Make one pull request for each topic. Add a changelog entry in the format that the project uses.
- Do not fix application bugs and do not add features. The only code changes are the changes that the update needs.
- Commit generated files that the update changes, for example `config/reference.php` in Symfony projects.
- Read the list of changed packages before you commit. Look for downgrades.

### Composer cooldown

Composer (2.11 and later) holds back new releases for a cooldown period. Our PHP images set it to 7 days
(`COMPOSER_POLICY_COOLDOWN_PERIOD="7 days"`). The cooldown is intentional: a compromised release can be found
before we install it.

- In a CVE update, the fixed version is usually new. Use `--no-cooldown` for the affected packages only.
- Do not use `--no-cooldown` together with `--with-all-dependencies`. Then all packages skip the cooldown.
- Do not change the cooldown in project files. To change it for one command, use
  `docker compose exec -e 'COMPOSER_POLICY_COOLDOWN_PERIOD=24 hours' phpfpm composer ...`.

## CVE/advisory update

1. Find the affected packages: `composer audit` (and `npm audit`).
2. Find all projects that use the affected packages, and update all of them. Remember projects that belong
   together, for example an application and its API.
3. Make a branch: `hotfix/x.y.z` from `main`. If a release from `develop` is near, you can make the update
   against `develop`.
4. Update only the affected packages, without the cooldown. Use a wildcard for packages that are released
   together, for example Symfony:

   ```sh
   composer update 'symfony/*' --no-cooldown
   ```

5. Run `composer audit` again. It must report no advisories.
6. Commit `composer.lock` and the generated files that changed.
7. Run the tests, and make sure that CI passes.
8. Add a changelog entry and make a pull request.
9. Make a release: tag on `main`, then merge `main` back into `develop`.

A fixed version that you installed with `--no-cooldown` stays in the cooldown for 7 days. A quarterly update in
that period needs a shorter cooldown (see quarterly step 4).

## Quarterly update

The quarterly update is maintenance only: old templates, PHP versions, images and dependencies.

### Step 1: Update the docker compose files from the templates

```sh
itkdev-docker-compose self:update
itkdev-docker-compose template:update
```

Do this step first, because it can change the PHP version. The PHP version controls which package versions
Composer can install.

- `template:update` does not merge. It copies the template files over the project files (`docker-compose*.yml`,
  `.docker/`, `.github/workflows/` and configuration files). Read the full diff.
- Restore project specific changes. Move them to files that the template does not overwrite:
  `docker-compose.override.yml`, `docker-compose.server.override.yml`, or
  `.docker/templates/server.d/*.conf.template` for nginx.
- Keep the improvements from the template. Delete new files that the project does not use.
- If the PHP version changes:
  - Update other references to the PHP version (for example the supervisor image and CI cache keys).
  - Update `require.php` in `composer.json`, then run `composer update --lock`.
  - Pull and start the new images, then run the tests and the code analysis.
  - Run Rector (it uses the rules for the new PHP version) and the linters.

### Step 2: Update the other images

Update images that the templates do not control (for example in `docker-compose.override.yml` and
`docker-compose.server.yml`) to the latest minor or patch in the same major version.

- Images that are only used in development (for example `node` with `profiles: [dev]`) can go to the latest LTS
  major version.
- If an image must match a version in a different place (for example a shared Elasticsearch cluster), update it
  in all repositories at the same time.

### Step 3: Check the direct dependencies

All packages that the code uses directly must be in `require` or `require-dev` in `composer.json`. Else a
transitive package can get a major version during the update, and nobody chose it. Use
[`pb30/phpstan-composer-analysis`](https://github.com/pb30/phpstan-composer-analysis) to find them. Add each
missing package with its installed version, so that `composer.lock` does not change:

```sh
composer require --no-update vendor/package:^<installed version>
composer update --lock
```

This also applies to Symfony components (for example `symfony/routing`).

### Step 4: Update the dependencies

```sh
composer update --with-all-dependencies
```

- If a package was updated with `--no-cooldown` less than 7 days ago, this command fails with "could not be
  resolved". The fixed version is still in the cooldown, and the older versions have an advisory. Use a shorter
  cooldown for this command (see [Composer cooldown](#composer-cooldown)).
- Run the tests after the update, also for minor and patch versions. A minor version can change behaviour
  without an error (for example how a library reads a wrong `@var` type). Only a test finds such a change.

### Step 5: Update Rector and PHPStan

Do this in a separate pull request, because new rules change code.

- Read the Rector changes before you apply them. Remove rules from `withSkip()` that Rector reports as unknown.
- Generate the PHPStan baseline again. It can only lose lines. Fix new findings, do not add them to the baseline.

### Step 6: Update transitive major versions

Update transitive packages that have a new major version, one package for each pull request. Do a `--dry-run`
first, to see which other packages must change.

### Step 7: Update the Symfony Flex recipes

```sh
composer recipes --outdated
composer recipes:update vendor/package
```

- Update one recipe at a time, and commit each recipe. `recipes:update` needs a clean git index.
- Keep project specific configuration (for example `security.yaml` and `messenger.yaml`) and files from the
  ITK Dev templates (for example `.php-cs-fixer.dist.php` and `.editorconfig`).
- If a recipe removes a setting, make sure that the setting is now the default. Else the behaviour changes.
- Make sure that the routes and the services did not change (`debug:router`, `lint:container`).
- The PHPUnit recipe enables `failOnDeprecation` for deprecations from the project's own code. Fix them first, in
  a separate pull request that is merged before the recipes.

### Step 8: Check and test before the release

- Run `composer audit`, `composer outdated --direct --minor-only` and `composer recipes --outdated`. Only versions
  that are still in the cooldown can remain.
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
6. Do the steps that must occur after the deploy (for example a re-index). Write them in the release pull request.

## Deprecations

Remove a deprecation in the quarterly update only if it is easy:

- **Included:** deprecations that Rector fixes. Let Rector do it.
- **Included:** easy fixes, for example a one-line configuration change or a new namespace in a `use` statement.
- **Not included:** deprecations that need changes to application code. Write them down for the next major upgrade.

The tests only find deprecations that occur when the code runs. A class or method that only has `@deprecated` in a
docblock does not trigger a deprecation, so the tests and Rector do not find it. Example:
`Symfony\Component\Serializer\Annotation\Groups` (deprecated in Symfony 7.4, use `Attribute\Groups`).

To find them, use [`phpstan/phpstan-deprecation-rules`](https://github.com/phpstan/phpstan-deprecation-rules):

```sh
composer require --dev phpstan/phpstan-deprecation-rules
```

Fix the deprecations that are included. Put the other deprecations in the PHPStan baseline, so that CI finds new
deprecations.

## Things to never do

- Do not update direct dependencies to a new major version in the quarterly update. This needs an agreement with
  the customer.
- Do not use `--no-cooldown` for more packages than the advisory affects.
- Do not add application fixes or features to an update pull request. Make a separate pull request.
- Do not make the PHPStan baseline larger to make an update pass.
