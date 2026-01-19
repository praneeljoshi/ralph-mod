# Ralph

![Ralph](ralph.webp)

Ralph is an autonomous AI agent loop that runs AI coding assistants repeatedly until all PRD items are complete. Each iteration is a fresh AI instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

**Supported AI assistants:**
- [Amp CLI](https://ampcode.com) - Full integration with skills and browser automation
- [Claude CLI](https://docs.anthropic.com/en/docs/claude-cli) - Native Claude Code support

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/).

[Read my in-depth article on how I use Ralph](https://x.com/ryancarson/status/2008548371712135632)

## Prerequisites

### For Amp CLI
- [Amp CLI](https://ampcode.com) installed and authenticated
- `jq` installed (`brew install jq` on macOS)
- A git repository for your project

### For Claude CLI
- [Claude CLI](https://docs.anthropic.com/en/docs/claude-cli) installed and authenticated
- `jq` installed (`brew install jq` on macOS)
- A git repository for your project

## Setup

Choose either Amp CLI or Claude CLI based on your preference.

### For Amp CLI

#### Option 1: Copy to your project

Copy the ralph files into your project:

```bash
# From your project root
mkdir -p scripts/ralph
cp /path/to/ralph/ralph.sh scripts/ralph/
cp /path/to/ralph/prompt.md scripts/ralph/
chmod +x scripts/ralph/ralph.sh
```

#### Option 2: Install skills globally

Copy the skills to your Amp config for use across all projects:

```bash
cp -r skills/prd ~/.config/amp/skills/
cp -r skills/ralph ~/.config/amp/skills/
```

#### Configure Amp auto-handoff (recommended)

Add to `~/.config/amp/settings.json`:

```json
{
  "amp.experimental.autoHandoff": { "context": 90 }
}
```

This enables automatic handoff when context fills up, allowing Ralph to handle large stories that exceed a single context window.

### For Claude CLI

Copy the Claude-Ralph files into your project:

```bash
# From your project root
mkdir -p scripts/ralph
cp /path/to/ralph/claude-ralph.sh scripts/ralph/
cp /path/to/ralph/claude-prompt.md scripts/ralph/
chmod +x scripts/ralph/claude-ralph.sh
```

## Workflow

### For Amp CLI

#### 1. Create a PRD

Use the PRD skill to generate a detailed requirements document:

```
Load the prd skill and create a PRD for [your feature description]
```

Answer the clarifying questions. The skill saves output to `tasks/prd-[feature-name].md`.

#### 2. Convert PRD to Ralph format

Use the Ralph skill to convert the markdown PRD to JSON:

```
Load the ralph skill and convert tasks/prd-[feature-name].md to prd.json
```

This creates `prd.json` with user stories structured for autonomous execution.

#### 3. Run Ralph

```bash
./scripts/ralph/ralph.sh [max_iterations]
```

Default is 10 iterations.

Ralph will:
1. Create a feature branch (from PRD `branchName`)
2. Pick the highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks (typecheck, tests)
5. Commit if checks pass
6. Update `prd.json` to mark story as `passes: true`
7. Append learnings to `progress.txt`
8. Repeat until all stories pass or max iterations reached

### For Claude CLI (Using prd.json directly)

If you already have a `prd.json` file, you can use Claude-Ralph directly:

#### 1. Create or obtain prd.json

Create a `prd.json` file in your project root following this structure:

```json
{
  "project": "YourProject",
  "branchName": "claude/feature-name",
  "description": "Feature description",
  "userStories": [
    {
      "id": "US-001",
      "title": "Story title",
      "description": "As a [user], I want [feature] so that [benefit]",
      "acceptanceCriteria": [
        "Specific verifiable criterion",
        "Tests pass",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

**Key requirements for prd.json:**
- Each story must be completable in ONE iteration (small scope)
- Stories ordered by `priority` (1 = highest)
- All stories start with `"passes": false`
- Acceptance criteria are specific and verifiable (not vague)
- Dependencies come first (e.g., schema → backend → UI)
- Use `claude/` or `ralph/` prefix for branch names

See `prd.json.example` for a complete reference.

#### 2. Review your prd.json

Before running, verify:
- [ ] Each story is small enough for one context window
- [ ] Priority ordering is correct
- [ ] Dependencies are ordered properly
- [ ] Acceptance criteria are verifiable
- [ ] Branch name uses `claude/` or `ralph/` prefix

#### 3. Run Claude-Ralph

```bash
./scripts/ralph/claude-ralph.sh [max_iterations]
```

Default is 10 iterations.

Claude-Ralph will:
1. Create/checkout the feature branch (from PRD `branchName`)
2. Pick the highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks (typecheck, tests)
5. Commit if checks pass (message: `feat: [Story ID] - [Story Title]`)
6. Update `prd.json` to mark story as `passes: true`
7. Append learnings to `progress.txt`
8. Repeat until all stories pass or max iterations reached

#### 4. Monitor Progress

```bash
# Check which stories are complete
cat prd.json | jq '.userStories[] | {id, title, passes}'

# View learnings from iterations
cat progress.txt

# Check git history
git log --oneline -10

# Check current branch
git branch --show-current
```

## Key Files

| File | Purpose |
|------|---------|
| `ralph.sh` | The bash loop that spawns fresh Amp instances |
| `prompt.md` | Instructions given to each Amp instance |
| `claude-ralph.sh` | The bash loop for Claude CLI |
| `claude-prompt.md` | Instructions given to each Claude instance |
| `prd.json` | User stories with `passes` status (the task list) |
| `prd.json.example` | Example PRD format for reference |
| `progress.txt` | Append-only learnings for future iterations |
| `skills/prd/` | Skill for generating PRDs (Amp only) |
| `skills/ralph/` | Skill for converting PRDs to JSON (Amp only) |
| `flowchart/` | Interactive visualization of how Ralph works |

## Flowchart

[![Ralph Flowchart](ralph-flowchart.png)](https://snarktank.github.io/ralph/)

**[View Interactive Flowchart](https://snarktank.github.io/ralph/)** - Click through to see each step with animations.

The `flowchart/` directory contains the source code. To run locally:

```bash
cd flowchart
npm install
npm run dev
```

## Critical Concepts

### Each Iteration = Fresh Context

Each iteration spawns a **new Amp instance** with clean context. The only memory between iterations is:
- Git history (commits from previous iterations)
- `progress.txt` (learnings and context)
- `prd.json` (which stories are done)

### Small Tasks

Each PRD item should be small enough to complete in one context window. If a task is too big, the LLM runs out of context before finishing and produces poor code.

Right-sized stories:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

Too big (split these):
- "Build the entire dashboard"
- "Add authentication"
- "Refactor the API"

### AGENTS.md Updates Are Critical

After each iteration, Ralph updates the relevant `AGENTS.md` files with learnings. This is key because Amp automatically reads these files, so future iterations (and future human developers) benefit from discovered patterns, gotchas, and conventions.

Examples of what to add to AGENTS.md:
- Patterns discovered ("this codebase uses X for Y")
- Gotchas ("do not forget to update Z when changing W")
- Useful context ("the settings panel is in component X")

### Feedback Loops

Ralph only works if there are feedback loops:
- Typecheck catches type errors
- Tests verify behavior
- CI must stay green (broken code compounds across iterations)

### Browser Verification for UI Stories

**For Amp CLI:**
Frontend stories must include "Verify in browser using dev-browser skill" in acceptance criteria. Ralph will use the dev-browser skill to navigate to the page, interact with the UI, and confirm changes work.

**For Claude CLI:**
Frontend stories should include browser verification in acceptance criteria. Claude-Ralph will document verification steps in progress.txt. You can use manual verification, browser automation tools (Playwright, Puppeteer), or run the dev server and visually confirm changes.

### Stop Condition

When all stories have `passes: true`, Ralph outputs `<promise>COMPLETE</promise>` and the loop exits.

## Debugging

Check current state:

```bash
# See which stories are done
cat prd.json | jq '.userStories[] | {id, title, passes}'

# See learnings from previous iterations
cat progress.txt

# Check git history
git log --oneline -10
```

## Customizing prompt.md

Edit `prompt.md` to customize Ralph's behavior for your project:
- Add project-specific quality check commands
- Include codebase conventions
- Add common gotchas for your stack

## Archiving

Ralph automatically archives previous runs when you start a new feature (different `branchName`). Archives are saved to `archive/YYYY-MM-DD-feature-name/`.

## References

- [Geoffrey Huntley's Ralph article](https://ghuntley.com/ralph/)
- [Amp documentation](https://ampcode.com/manual)
- [Claude CLI documentation](https://docs.anthropic.com/en/docs/claude-cli)
