---
description: Re-check each Obsidian note against the current repo and update stale facts
argument-hint: [note-name | all] [dry-run]
---

Arguments: `$ARGUMENTS`
Parse them: if a **note name** (or partial title) is given, sync only that note;
otherwise sync **all** notes. If the word `dry-run` appears, report proposed
changes without writing.

Keep the project's Obsidian vault in sync with the current state of this repo
(and any related repos the notes document). Find the vault path via the
`itkdev-obsidian-vault` skill (the `## Obsidian vault` section in `CLAUDE.md`, a
project memory, or by asking the user), and follow every convention there
(house style, frontmatter, linking, the direct-file-writes and no-commit rules).

## Procedure

1. **Enumerate** the target note(s) in the vault.
2. For each note, **re-verify its claims against the source** — the file(s) it
   documents in this repo (or a related repo). Use `git log`/`git diff` since the
   note's `created`/`updated` date to find what moved: renamed/added/removed
   services, changed env keys, config edits, patch changes, version bumps.
3. **Update only what drifted.** Fix stale facts; add newly-material behaviour;
   remove what no longer exists. Do **not** rewrite accurate prose for its own
   sake, and do not soften existing `(inferred)` markers or contradiction
   callouts unless the source now resolves them.
4. When something genuinely new lacks a note, note the gap in your report (or
   create it under the fitting folder if clearly in scope) — don't cram
   unrelated material into an existing note.
5. Set `updated:` to today's date on any note you changed. Keep links
   bidirectional and the vault's index/home note current.

## Rules
- Write directly to the vault path. If an Obsidian MCP is configured, prefer
  direct file operations anyway (some MCP servers have unreliable writes).
- Do **not** commit anything.
- End with a per-note summary: `unchanged` / the specific facts you corrected /
  gaps found. Be precise about what changed and why (cite the source).
