# itkdev-skills

ITK Dev team conventions, workflows, and coding standards for Claude Code.

This repository hosts **three** Claude Code plugins, distributed through the
`itkdev-marketplace`. Each plugin is an independent install target, so a team can
install only the categories it needs. The categories follow the skill *types*
described in Anthropic's
[Lessons from building Claude Code: how we use skills](https://claude.com/blog/lessons-from-building-claude-code-how-we-use-skills).

> **Migrating from the old `itkdev-skills` plugin?** It has been split into the
> three plugins below and is no longer published as a single plugin. Install the
> ones you need (see each section's install command).

## `itkdev-code-quality-and-review`

*Code quality and review* — the PR review agent and per-language review skills.

```bash
/plugin install itkdev-code-quality-and-review@itkdev-marketplace
```

| Skill | Description |
|-------|-------------|
| `itkdev-review-php` | PHP code review checklist (Laravel, Symfony, plain PHP) — security, correctness, performance, PSR-12 |
| `itkdev-review-python` | Python code review checklist (Django, Flask, FastAPI, scripts) — includes a dedicated security review |
| `itkdev-review-javascript` | JavaScript/TypeScript code review checklist — security, async error handling, type safety, style |
| `itkdev-review-comments` | Review and improve inline comments and docblocks (explains "why" not "what") — only touches comments, never code |
| `itkdev-validate-standards` | Project standards validation against ITK Dev conventions |

| Agent | Description |
|-------|-------------|
| `itkdev-code-review` | Automated PR review against ITK Dev standards |

> **Review skills vs. the review agent:** the `itkdev-review-*` skills are language-specific
> checklists you can invoke inline on a file or snippet. The `itkdev-code-review` **agent** is the
> PR-review orchestrator — it gathers PR data, runs process-compliance checks, and delegates
> language-specific code-quality checks to the matching `itkdev-review-*` skill. For the fullest
> Drupal and GitHub-guideline coverage it also draws on skills in the
> `itkdev-scaffolding-and-templates` and `itkdev-business-automation` plugins when those are
> installed, but it degrades gracefully without them.

## `itkdev-scaffolding-and-templates`

*Code scaffolding and templates* — Docker, project templates, CI, and framework setup.

```bash
/plugin install itkdev-scaffolding-and-templates@itkdev-marketplace
```

| Skill | Description |
|-------|-------------|
| `itkdev-docker` | Docker development environment (CLI reference, Compose architecture, services, Traefik, server deployments, project detection, template comparison) |
| `itkdev-docker-templates` | Project template conventions (available templates, installation, setup workflows, procedural template operations) |
| `itkdev-gh-actions` | GitHub Actions workflow templates (general, Drupal, Symfony workflows, configuration files) |
| `itkdev-taskfile` | Taskfile development workflows (task patterns, coding standards, site management, asset building) |
| `itkdev-drupal` | Drupal 10/11 development assistance (code auditing, module/theme development, configuration management) |
| `itkdev-symfony` | Symfony development assistance (scaffolding, database configuration, console commands, Symfony configuration) |

| Agent | Description |
|-------|-------------|
| `itkdev-create-project` | Create new Drupal/Symfony projects with ITK Dev Docker setup |

## `itkdev-business-automation`

*Business process and team automation* — issue workflow, GitHub guidelines, ADRs, and documentation.

```bash
/plugin install itkdev-business-automation@itkdev-marketplace
```

| Skill | Description |
|-------|-------------|
| `itkdev-issue-workflow` | How the team works a GitHub issue end to end (developer-driven; Claude assists) |
| `itkdev-github-guidelines` | GitHub workflow guidelines (branch naming, commits, changelogs, PRs) |
| `itkdev-adr` | Architecture Decision Record management |
| `itkdev-documentation` | Technical documentation and README generation following ITK Dev standards |
| `itkdev-pr` | Generate a pull request description from the branch diff, respecting the repository's PR template if one exists |

## `itkdev-obsidian`

*Obsidian knowledge base* — bootstrap a project vault and keep notes, docs, and diagrams in it with the normal file tools.

```bash
/plugin install itkdev-obsidian@itkdev-marketplace
```

| Skill | Description |
|-------|-------------|
| `itkdev-obsidian-vault` | Read, create, edit, search, move, and tag notes in the project's vault using the normal file tools (vault path resolved from `CLAUDE.md`) |
| `itkdev-obsidian-markdown` | Author Obsidian Flavored Markdown — wikilinks, embeds, callouts, properties, tags, Mermaid, LaTeX, footnotes, block references |
| `itkdev-obsidian-canvas` | Create and edit JSON Canvas (`.canvas`) files — nodes (text, file, link, group) and edges for Obsidian Canvas diagrams |

| Command | Description |
|---------|-------------|
| `/itkdev-obsidian:init` | Bootstrap a project to use an Obsidian vault (create/adopt a vault, seed a home note, wire it into `CLAUDE.md` + a project memory) |
| `/itkdev-obsidian:analyse` | Analyse a folder and capture the findings as notes in the vault (`quick`/`deep` depth) |
| `/itkdev-obsidian:sync` | Re-check each vault note against the current repo and update stale facts |
