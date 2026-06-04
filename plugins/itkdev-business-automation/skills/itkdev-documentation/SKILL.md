---
name: itkdev-documentation
user-invocable: true
description: Technical documentation and README generation for ITK Dev projects. Use when asked to create, update, or improve documentation, README files, deployment guides, architecture docs, or API documentation. Follows ITK Dev documentation standards with clear structure and procedural content.
---

# Documentation Generation Guide

Assist with technical documentation for ITK Dev projects — READMEs, deployment
guides, architecture docs, and API references. This skill gives you the ITK Dev
*house style* and points to ready-made templates; it does not prescribe a rigid
fill-in-the-blanks procedure. Adapt the structure to the project at hand.

## When to use

The user asks to create or update a README, write technical documentation,
generate a deployment/installation guide, or document architecture or API
endpoints.

## House style

Follow the [AarhusAI technical documentation style](https://github.com/AarhusAI/documentation/tree/main/technical).

- **Headings** are hierarchical: H1 for the document title, H2 for major
  sections, H3 for subsections. Lead with an overview, then prerequisites, then
  procedures, then reference/troubleshooting.
- **Placeholders** use angle brackets for values the reader must replace:
  `<DOMAIN_NAME>`, `<API_KEY>`, `<DATABASE_PASSWORD>`.
- **Code blocks** are fenced with a language identifier (` ```bash `, ` ```json `).
- **Callouts** use bold labels: `**NOTE:**` for important information,
  `**WARNING:**` for potentially destructive operations.
- **Procedures** are numbered when steps are sequential; use bullets otherwise.
- **Cross-reference** related docs and external resources with links.

## Templates

Use the template that matches the document, then tailor it — drop sections that
do not apply rather than leaving empty placeholders:

| Document | Template |
|----------|----------|
| README | [`references/readme-template.md`](references/readme-template.md) |
| Deployment guide | [`references/deployment-guide.md`](references/deployment-guide.md) |
| Architecture overview | [`references/architecture.md`](references/architecture.md) |
| API reference | [`references/api-reference.md`](references/api-reference.md) |

## Project-specific content

Detect the stack (Drupal, Symfony, Node.js, Python, Docker) and tailor wording
and commands accordingly — but **do not restate command tables that already live
in dedicated skills.** Link to them instead:

- **Docker / `itkdev-docker-compose` CLI commands** → the `itkdev-docker` skill
  (in the `itkdev-scaffolding-and-templates` plugin).
- **Taskfile tasks** → the `itkdev-taskfile` skill (same plugin). If a
  `Taskfile.yml` exists, document available tasks by pointing readers to
  `task --list` rather than duplicating the table.
- **Framework setup** → the `itkdev-drupal` / `itkdev-symfony` skills.

If those plugins are not installed, write the commands inline from the project's
own `Taskfile.yml`, `composer.json`, and `compose.*.yml`.

## Quality checklist

Before finalizing, verify:

- [ ] All placeholders use angle-bracket notation (`<VALUE>`)
- [ ] Code blocks have language identifiers
- [ ] Sequential operations are numbered
- [ ] Prerequisites appear before instructions
- [ ] Commands are copy-paste ready
- [ ] Links to related/external resources are included where helpful
- [ ] No sensitive information (passwords, keys) in examples
- [ ] Documentation matches the actual project state
