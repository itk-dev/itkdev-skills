---
name: itkdev-obsidian-vault
description: Read, create, edit, search, move, and tag notes in a project's Obsidian vault using the normal file tools. Use whenever the user asks to capture, look up, update, organize, or search notes/documentation in their Obsidian vault, or says things like "add a note", "put this in Obsidian", "find my note about…", or "update the vault".
---

# Obsidian Vault

This skill operates on the project's Obsidian vault.

## Finding the vault path

There is no hardcoded vault location. Resolve it in this order:

1. Read the project's `CLAUDE.md` — a `## Obsidian vault` section (written by the
   `/itkdev-obsidian:init` command) records the absolute vault path and the home note.
2. Otherwise check for a `project-*-obsidian-vault` memory.
3. If neither exists, **ask the user** for the vault path — and suggest running
   `/itkdev-obsidian:init` to wire it up durably so future sessions find it.

An Obsidian vault is just a folder of Markdown (`.md`) and `.canvas` files, so
work with it using the **normal file tools** (Write, Read, Edit, Glob, Grep,
`mv`). Obsidian re-indexes files automatically when the app is focused, so direct
writes appear as normal notes. For the *content* syntax (wikilinks, callouts,
properties, embeds) follow `itkdev-obsidian-markdown`; for `.canvas` files follow
`itkdev-obsidian-canvas`.

> [!note] Why not an MCP?
> If an `obsidian` MCP server is configured, prefer direct file operations
> anyway — some Obsidian MCP servers have unreliable or hanging write operations.
> Direct file ops are equivalent for creating/reading/editing/searching notes.
> The only thing an MCP would add is auto-updating backlinks on move/rename;
> handle that manually (see below).

## Tool mapping

| Task | How |
|------|-----|
| Read a note | `Read` the file under the vault root |
| Find notes / full-text search | `Grep` (content) or `Glob` (by name) under the vault root |
| Create a note | `Write` to `<vault>/<path>/<Name>.md` (creates folders as needed) |
| Edit a note | `Read` then `Edit` (targeted change) |
| Create a folder | just `Write` a file into it — parents are created |
| Move / rename a note | `mv` via Bash, **then** fix backlinks (see below) |
| Add / remove / rename tags | `Edit` frontmatter, or `Grep` + `Edit` for vault-wide tag renames |

## Workflows

### Capture / create a note
1. Choose a clear path and filename; group related notes in folders (e.g.
   `Services/`, `Runbooks/`).
2. Draft the body using `itkdev-obsidian-markdown` conventions — frontmatter with
   `tags`/`aliases`, `[[wikilinks]]` to related notes, callouts for
   warnings/tips.
3. `Write` the file. If it may already exist, `Glob`/`Read` first so you don't
   clobber it.

### Look something up
1. `Grep` the vault for the user's terms (or `#tag`).
2. `Read` the best hit(s) for full context before answering.

### Update a note
1. `Read` the current content.
2. Make a minimal, targeted `Edit`; preserve existing frontmatter, links, and
   formatting.

### Reorganize (move / rename)
1. `mv` the file to its new path/name.
2. **Update backlinks yourself:** `Grep` the vault for `[[Old Name` and `Edit`
   each referencing note to point at the new name. (Nothing does this
   automatically without an MCP.)

### Link notes together
- When a note relates to existing ones, add `[[Wikilinks]]`. If unsure of exact
  titles, `Glob`/`Grep` first so links resolve to real notes.

## Conventions

- **Naming:** descriptive Title Case note names (e.g. `Docker Setup`). Use folders
  to group (`Services/`, `Infrastructure/`, `Runbooks/`).
- **Frontmatter:** include `tags` and a `created` date; add `aliases` for acronyms.
- **Connect, don't orphan:** link new notes to at least one related existing note,
  and to the vault's home/index note (recorded in `CLAUDE.md`).
- **Prefer wikilinks** for internal references.

## Safety

- **Destructive actions require confirmation.** Before deleting a note, or before
  an `Edit`/`mv` that would overwrite or significantly rewrite an existing note,
  read it first and confirm with the user. Report what exists rather than
  silently replacing it.
- Only touch files under the vault root; never edit `.obsidian/` config unless
  explicitly asked.
