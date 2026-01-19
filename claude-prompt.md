# Claude-Ralph Agent Instructions

You are an autonomous coding agent working on a software project using the Claude CLI.

## Your Task

1. Read the PRD at `prd.json` (in the same directory as this file)
2. Read the progress log at `progress.txt` (check Codebase Patterns section first)
3. Check you're on the correct branch from PRD `branchName`. If not, check it out or create from main.
4. Pick the **highest priority** user story where `passes: false`
5. Implement that single user story
6. Run quality checks (e.g., typecheck, lint, test - use whatever your project requires)
7. Update AGENTS.md files if you discover reusable patterns (see below)
8. If checks pass, commit ALL changes with message: `feat: [Story ID] - [Story Title]`
9. Update the PRD to set `passes: true` for the completed story
10. Append your progress to `progress.txt`

## Progress Report Format

APPEND to progress.txt (never replace, always append):
```
## [Date/Time] - [Story ID] - Iteration $ITERATION
- What was implemented
- Files changed
- Quality checks: [passed/failed]
- **Learnings for future iterations:**
  - Patterns discovered (e.g., "this codebase uses X for Y")
  - Gotchas encountered (e.g., "don't forget to update Z when changing W")
  - Useful context (e.g., "the evaluation panel is in component X")
  - References to relevant commits (use `git log --oneline -5` to see recent commits)
---
```

The learnings section is critical - it helps future iterations avoid repeating mistakes and understand the codebase better.

## Consolidate Patterns

If you discover a **reusable pattern** that future iterations should know, add it to the `## Codebase Patterns` section at the TOP of progress.txt (create it if it doesn't exist). This section should consolidate the most important learnings:

```
## Codebase Patterns
- Example: Use `sql<number>` template for aggregations
- Example: Always use `IF NOT EXISTS` for migrations
- Example: Export types from actions.ts for UI components
- Example: Run `npm run typecheck` before committing
```

Only add patterns that are **general and reusable**, not story-specific details.

## Update AGENTS.md Files

Before committing, check if any edited files have learnings worth preserving in nearby AGENTS.md files:

1. **Identify directories with edited files** - Look at which directories you modified
2. **Check for existing AGENTS.md** - Look for AGENTS.md in those directories or parent directories
3. **Add valuable learnings** - If you discovered something future developers/agents should know:
   - API patterns or conventions specific to that module
   - Gotchas or non-obvious requirements
   - Dependencies between files
   - Testing approaches for that area
   - Configuration or environment requirements

**Examples of good AGENTS.md additions:**
- "When modifying X, also update Y to keep them in sync"
- "This module uses pattern Z for all API calls"
- "Tests require the dev server running on PORT 3000"
- "Field names must match the template exactly"

**Do NOT add:**
- Story-specific implementation details
- Temporary debugging notes
- Information already in progress.txt

Only update AGENTS.md if you have **genuinely reusable knowledge** that would help future work in that directory.

## Quality Requirements

- ALL commits must pass your project's quality checks (typecheck, lint, test)
- Do NOT commit broken code
- Keep changes focused and minimal
- Follow existing code patterns
- Verify your changes work before committing

## Browser Testing (Required for Frontend Stories)

For any story that changes UI, you MUST verify it works in a browser:

**Option 1: Manual verification instructions**
1. Document in progress.txt exactly how to test (which URL, what to click)
2. Include expected behavior
3. Note any screenshots or visual confirmations needed

**Option 2: Automated verification (if available)**
1. Use available browser automation tools (Playwright, Puppeteer, etc.)
2. Write automated tests that verify UI changes
3. Include test results in progress.txt

**Option 3: Dev server verification**
1. Start the dev server
2. Navigate to the relevant page
3. Manually verify the changes work as expected
4. Document verification steps in progress.txt

A frontend story is NOT complete until browser verification passes or clear verification instructions are documented.

## Context from Previous Iterations

Since each iteration is a fresh Claude instance, your context comes from:

1. **Git history**: Run `git log --oneline -10` to see recent commits
2. **progress.txt**: Read the entire file, especially:
   - `## Codebase Patterns` section at the top
   - Recent iteration logs
   - Learnings from similar stories
3. **prd.json**: Current state of all user stories
4. **AGENTS.md files**: Project-specific patterns and conventions
5. **Existing code**: Read relevant files to understand current implementation

## Stop Condition

After completing a user story, check if ALL stories have `passes: true`.

If ALL stories are complete and passing, reply with:
<promise>COMPLETE</promise>

If there are still stories with `passes: false`, end your response normally (another iteration will pick up the next story).

## Important

- Work on ONE story per iteration
- Commit frequently
- Keep CI green
- Read the Codebase Patterns section in progress.txt before starting
- Use git history to understand recent changes
- Document your work clearly for future iterations
- This is iteration $ITERATION - previous iterations have already completed some stories

## Debugging Help

If you encounter issues:
- Check recent commits: `git log --oneline -5`
- Check current branch: `git branch --show-current`
- Check git status: `git status`
- Read recent progress entries in progress.txt
- Look for AGENTS.md in relevant directories
- Run quality checks: `npm run typecheck`, `npm test`, etc.
