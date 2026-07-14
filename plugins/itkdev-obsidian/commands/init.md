---
description: Bootstrap a project to use an Obsidian vault — create/adopt a vault, seed a home note, and wire the project to it via CLAUDE.md and a project memory
argument-hint: [vault-path]
---

Arguments: `$ARGUMENTS`
If a path is given, propose it as the vault location (still confirm it in step 1);
otherwise ask the user where the vault should live.

One-time setup that connects a code project to an Obsidian vault so future
sessions know to keep notes/docs there. A vault is just a folder of Markdown, so
everything is done with the normal file tools (Write/Read/Edit/Grep) — no MCP
required (see the `itkdev-obsidian-vault` skill for why).

Run the steps in order. Confirm with the user at the marked points.

## 1. Determine the vault path (ask)

There is no fixed location convention — **ask the user where the vault should
live** (e.g. `~/source/obsidian/<project>`, or a `./docs` folder in the repo).
Then establish whether you're:
- **Creating a new vault** — the folder doesn't exist yet, or exists but is empty.
- **Adopting an existing vault** — it already has notes and/or a `.obsidian/`
  folder. In that case do **not** overwrite anything; only add.

Confirm the resolved absolute path with the user before writing anything.

## 2. Create / adopt the vault + seed an index note

- If new, create the vault folder. (Obsidian generates `.obsidian/` itself on
  first open; you don't need to.)
- Create a home/index note named after the project, e.g. `<Project> Notes.md`,
  using `itkdev-obsidian-markdown` conventions:

```markdown
---
title: <Project> Notes
tags: [<project-slug>, index]
created: <today's date, YYYY-MM-DD>
---

# <Project> Notes

Home note for the [[<Project>]] vault. Companion to the code repo at `<repo path>`.

## Contents
_(link notes here as they're created)_
```

- If adopting an existing vault, skip creating an index if one already exists;
  just note the existing home note.

## 3. Wire the project to the vault (durable)

**a. CLAUDE.md** — add (or create) an `## Obsidian vault` section in the
project's `CLAUDE.md` so every session picks it up:

```markdown
## Obsidian vault

Documentation and notes for this project live in an Obsidian vault at:
`<absolute vault path>`

Work the vault with the normal file tools (Write/Read/Edit/Grep) directly against
that path. If an Obsidian MCP is configured, still prefer direct file operations
(some MCP servers have unreliable/hanging writes). Follow the
`itkdev-obsidian-vault`, `itkdev-obsidian-markdown`, and `itkdev-obsidian-canvas`
skills for conventions. The home note is `<index note name>`.
```

**b. Project memory** — save a `project`-type memory recording the link, and add
its one-line pointer to `MEMORY.md`:

```markdown
---
name: project-<slug>-obsidian-vault
description: this project's notes live in an Obsidian vault at <path>
metadata:
  type: project
---

The <Project> repo keeps its documentation/notes in an Obsidian vault at
`<absolute path>`. Home note: `<index note name>`. Work the vault with direct
file tools (Write/Read/Edit/Grep). See the `itkdev-obsidian-vault` skill.
```

## 4. Confirm

Report back: the vault path, the index note created, and that CLAUDE.md + memory
now record the link. The `itkdev-obsidian` skills (`itkdev-obsidian-vault`,
`itkdev-obsidian-markdown`, `itkdev-obsidian-canvas`) are installed with this
plugin, so nothing else needs copying in. Suggest the user open the vault in
Obsidian once to let it initialize `.obsidian/`.

## Notes
- Vaults typically live **outside** the code repo. If the user puts it inside the
  repo, ask whether it should be tracked or added to `.gitignore`.
- Never commit anything unless the user asks.
