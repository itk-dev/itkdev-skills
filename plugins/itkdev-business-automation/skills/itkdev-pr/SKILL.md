---
name: pr-description
description: Generate a pull request description respecting the repository's PR template if one exists.
---

# PR Description Generator

Generate a high-quality pull request description based on the actual changes in the branch, formatted
according to the repository's PR template when one exists. The final deliverable is a ready-to-use
`gh pr create` command that the user runs themselves — never create the PR on the user's behalf.

## Workflow

### Step 1: Determine the base branch

Identify which branch the PR will target. In order of preference:

1. If the user named a base branch, use it.
2. Ask git for the default branch of the remote:
   ```bash
   git remote show origin 2>/dev/null | sed -n 's/.*HEAD branch: //p'
   ```
3. Fall back to checking which of `main`, `master`, or `develop` exists:
   ```bash
   git branch -r | grep -E 'origin/(main|master|develop)$'
   ```

If the current branch IS the base branch, stop and tell the user — there is nothing to describe.

### Step 2: Collect the changes

Base the description on the diff, not on assumptions. Use the merge-base so only the branch's own 
changes are included:

```bash
BASE=<base-branch>
git fetch origin $BASE --quiet 2>/dev/null
MERGE_BASE=$(git merge-base origin/$BASE HEAD 2>/dev/null || git merge-base $BASE HEAD)
git diff --stat $MERGE_BASE..HEAD          # overview of what changed
git log --oneline $MERGE_BASE..HEAD        # commits, for context only
git diff $MERGE_BASE..HEAD                 # the actual changes
```

For large diffs (more than ~2000 lines), read the `--stat` output first, then inspect the diff 
file-by-file (`git diff $MERGE_BASE..HEAD -- <path>`), prioritizing source files over lockfiles,
generated code, and snapshots. Never describe a change you have not actually seen in the diff. If
the diff is empty, tell the user and stop.

### Step 3: Find the PR template

GitHub looks for templates in these locations (case-insensitive filename, `.md` or no extension).

Check all of them:

```bash
for f in .github/PULL_REQUEST_TEMPLATE.md .github/pull_request_template.md \
         PULL_REQUEST_TEMPLATE.md pull_request_template.md \
         docs/PULL_REQUEST_TEMPLATE.md docs/pull_request_template.md; do
  [ -f "$f" ] && echo "FOUND: $f"
done
ls .github/PULL_REQUEST_TEMPLATE/ 2>/dev/null   # multiple-template directory
```

- **One template found** → it is mandatory. Use it.
- **Multiple templates** (a `PULL_REQUEST_TEMPLATE/` directory) → list them and ask the user which one applies, unless one obviously matches the change type (e.g. `bugfix.md` for a fix).
- **No template** → use the default structure in Step 4.

### Step 4: Write the description

**With a template:** Preserve the template's structure exactly — every heading, checklist, and HTML comment marker stays in its original order. Fill each section with content derived from the diff:

- Replace placeholder text and fill in blank sections.
- Keep HTML comments (`<!-- ... -->`) only if they are instructions meant to remain; remove ones that say "delete this" or are pure placeholders that have been satisfied.
- Checklists: check items (`- [x]`) only when the diff proves they are true (e.g. check "Tests added" only if test files changed). Leave the rest unchecked — never check boxes optimistically.
- If a section does not apply (e.g. "Screenshots" for a backend change), write "N/A" or a one-line explanation rather than deleting the section.

**Without a template**, use this structure:

```markdown
## Summary
<1-3 sentences: what this PR does and why>

## Changes
<short bullet list of the meaningful changes, grouped logically — not a file-by-file recap>

## Testing
<how the change was verified, based on test files in the diff; if no tests changed, say so honestly>
```

**Writing rules (either case):**
- Lead with the *why*, then the *what*. Reviewers can read the diff; the description's job is intent and context.
- Be specific: "Add retry with exponential backoff to the S3 uploader" beats "Improve reliability".
- Mention breaking changes, migrations, or config changes prominently.
- Don't pad. A small fix deserves a short description.
- Title: imperative mood, under ~70 characters, following the repo's existing convention if visible in `git log` 
  (e.g. conventional commits like `fix:`/`feat:`).

### Step 5: Deliver as a gh command — do not run it

Write the description body to a file, then present the command, the user reviews and runs it.

```bash
cat > /tmp/pr-body.md << 'PRBODY'
<the description>
PRBODY
```

Then show the user:

```bash
gh pr create --base <base-branch> --title "<title>" --body-file /tmp/pr-body.md
```

Also show the full description text itself so the user can review it before running anything. If `--draft`
seems appropriate (WIP markers, failing checks mentioned), suggest it but let the user decide.

## Things to never do

- Never invent content (test results, issue numbers, reviewers) not evidenced by the diff or stated by the 
  user. If the template asks for an issue link and none is known, leave the placeholder and flag it.
- Never drop or reorder template sections.
