---
description: Analyse a folder and capture the findings as notes in the project's Obsidian vault
argument-hint: <folder> [depth: quick|deep]
---

Arguments: `$ARGUMENTS`
Parse them: the **folder to analyse** is the first token (a path); the **depth**
is `deep` or `quick` if either word appears (default `quick`). Ignore order
otherwise.

Analyse that folder and capture the result in the project's Obsidian vault. Find
the vault path via the `itkdev-obsidian-vault` skill (the `## Obsidian vault`
section in `CLAUDE.md`, a project memory, or by asking the user).

## Analyse
Cover: its purpose, architecture, entry point(s), key modules/routes/pipeline
stages, config/env surface, external dependencies, and how it fits the wider
project. Start from the folder's own `README.md` / `CLAUDE.md` / `FINDINGS.md` /
`docs/` if present, then confirm against the source.

Depth:
- **quick** — architecture-map level: read the docs + top-level files + entry point.
- **deep** — also read into the source; fan out read-only `Explore` agents over
  sub-areas and synthesise.

## Capture in the vault
Write to the vault following the `itkdev-obsidian-vault`,
`itkdev-obsidian-markdown`, and `itkdev-obsidian-canvas` skills:
- If a note for this component already exists (search the vault first), **expand
  it** rather than creating a duplicate. Otherwise create one under the most
  fitting folder (e.g. `Services/<Name>.md`).
- Include YAML frontmatter (`title`, `tags`, `created`), use callouts for
  gotchas/warnings, and add a Mermaid or `.canvas` diagram when it aids understanding.
- Link the note both ways: `[[wikilinks]]` to related notes, and add it to the
  vault's index/home note (recorded in `CLAUDE.md`) if not already listed.

## Rules
- Use **direct file writes** to the vault path. If an Obsidian MCP is configured,
  prefer direct file operations anyway (some MCP servers have unreliable writes).
- Do **not** commit anything.
- End by reporting the vault path(s) written and a one-line summary of the component.
