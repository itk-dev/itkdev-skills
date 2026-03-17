---
name: itkdev-issue-workflow
description: "Autonomous GitHub issue workflow: develop, test, review, merge. Use this skill to work through GitHub issues with minimal user interaction - only pausing when user review/merge is required."
---

# GitHub Issue Workflow

You are an autonomous developer working through GitHub issues. Work with MINIMAL user interaction - only pause when user review/merge is required.

## PHASE 1: Issue Selection
1. Run `gh issue list --state open --limit 10` to show open issues
2. If user provided an issue number, use that. Otherwise, present the list and ask which issue to work on.
3. Run `gh issue view <number>` to get full details
4. Briefly summarize the issue and START WORKING IMMEDIATELY - do not ask for confirmation.

## PHASE 2: Development (AUTONOMOUS)
1. Switch to main branch and pull latest: `git checkout main && git pull`
2. Create feature branch: `git checkout -b feature/issue-<number>-<short-description>`
3. For non-trivial tasks, use EnterPlanMode to plan implementation
4. Implement the solution following CLAUDE.md guidelines
5. Update CHANGELOG.md with the changes
6. Run CI checks (see "Tool Detection Strategy" section below):
   
   **Step A - Detect available tools:**
   - Run `task` to list available Taskfile tasks (if Taskfile.yml exists)
   - Run `itkdev-docker-compose composer run --list` to see composer scripts
   
   **Step B - Apply coding standards fixes first (auto-fix before checking):**
   - Preferred: `task coding-standards:apply`
   - Fallback: `itkdev-docker-compose composer run phpcbf` (if script exists)
   - Fallback: `itkdev-docker-compose vendor/bin/phpcbf`
   
   **Step C - Run full CI checks:**
   - Preferred: `task ci` (runs all checks)
   - If `task ci` doesn't exist, run checks individually:
     - `task ci:coding-standards` or `itkdev-docker-compose vendor/bin/phpcs`
     - `task ci:phpunit` or `itkdev-docker-compose vendor/bin/phpunit`
     - `itkdev-docker-compose composer run phpstan` (if available)
   
   **Step D - Fix any remaining issues and repeat until all checks pass**

7. Commit changes with descriptive message referencing the issue
8. Push branch and create PR:
   ```bash
   git push -u origin <branch-name>
   gh pr create --title "Issue #<number>: <short description>" --body "$(cat <<'EOF'
   ## Summary
   <Brief description of what was implemented>
   
   ## Changes
   - <List key changes made>
   
   ## Testing
   - <How the changes were tested>
   
   Fixes #<issue-number>
   EOF
   )"
   ```
   Store the PR number for use in Phase 5.

## PHASE 3: Automated Testing (AUTONOMOUS)
Automatically test based on the type of change:

**For UI changes:**
- Use dev-browser skill to navigate to the affected page
- Test the specific functionality that was changed
- Take screenshots if helpful
- Document any issues found and fix them

**For API/backend changes:**
- If there's a UI component (like a download button), test it via dev-browser
- Verify the fix works as expected

**For PDF/report changes:**
- Navigate to a scan results page
- Click the PDF download button
- Verify no errors occur

**For form/validation changes:**
- Navigate to the relevant form
- Test with valid and invalid inputs

Fix any issues found during testing, commit, and push.

## PHASE 4: Automated Code Review (AUTONOMOUS)
1. Run Task tool with subagent_type='pr-review-toolkit:code-reviewer'
2. Run Task tool with subagent_type='pr-review-toolkit:silent-failure-hunter'
3. If HIGH priority issues are found, fix them automatically
4. For MEDIUM/LOW issues, use judgment - fix if straightforward, otherwise note them
5. Push any fixes made

## PHASE 5: User Review & Merge (WAIT FOR USER)
Present a summary to the user:
- What was implemented
- What was tested
- Code review results and any fixes made
- PR link

Then say: "PR is ready for your review and merge. I'll wait for the merge to complete."

Wait for merge using:
```bash
while true; do
  pr_state=$(gh pr view <PR_NUMBER> --json state -q '.state')
  if [ "$pr_state" = "MERGED" ]; then
    echo "PR merged!"
    break
  fi
  sleep 10
done
```

## PHASE 6: Next Issue (AFTER MERGE)
1. Switch to main and pull: `git checkout main && git pull`
2. Check for remaining open issues: `gh issue list --state open --limit 5`
3. If there are open issues, suggest the next one to tackle:
   "Merge complete! Here are the remaining open issues:
   [list issues]

   I suggest we tackle #XX next because [reason]. Should I start working on it?"
4. If user confirms, immediately start Phase 1 with that issue number.

## Tool Detection Strategy

Before running CI commands, detect what's available in the project:

### 1. Check for Taskfile
```bash
if [ -f "Taskfile.yml" ] || [ -f "Taskfile.yaml" ]; then
  task --list  # Shows available tasks
fi
```

### 2. Check composer.json scripts
```bash
itkdev-docker-compose composer run --list
```

### 3. Common Taskfile task names to look for
| Task | Purpose |
|------|---------|
| `task ci` | Full CI suite (coding standards + tests) |
| `task ci:coding-standards` | Coding standards check only |
| `task ci:phpunit` | PHPUnit tests only |
| `task coding-standards:apply` | Auto-fix coding standards issues |
| `task config:export` | Export Drupal configuration |
| `task config:import` | Import Drupal configuration |
| `task dev:setup` | Initial project setup |
| `task dev:reset` | Reset local environment |

### 4. Common composer scripts to look for
| Script | Purpose |
|--------|---------|
| `phpcs` | PHP CodeSniffer - check coding standards |
| `phpcbf` | PHP Code Beautifier - auto-fix standards |
| `phpunit` / `test` | Run PHPUnit tests |
| `phpstan` / `analyze` | Static analysis |
| `coding-standards` | Combined standards check |
| `coding-standards:check` | Check only (no fix) |
| `coding-standards:apply` | Apply/fix standards |

### 5. Direct itkdev-docker-compose fallbacks
When no task or composer script exists, run tools directly:
```bash
itkdev-docker-compose vendor/bin/phpcs      # Coding standards
itkdev-docker-compose vendor/bin/phpcbf     # Auto-fix standards
itkdev-docker-compose vendor/bin/phpunit    # PHPUnit tests
itkdev-docker-compose vendor/bin/phpstan    # Static analysis
```

## Important Rules
- Work AUTONOMOUSLY through phases 1-4 without asking for confirmation
- Only pause at Phase 5 for user review/merge
- Always follow CLAUDE.md guidelines
- Never commit directly to main
- Always run CI checks before creating PR (use tool detection strategy)
- Apply coding standards fixes BEFORE running checks (saves time)
- Fix issues found by automated testing and review without asking
- Be efficient - don't ask unnecessary questions
