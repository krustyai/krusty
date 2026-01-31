# Krusty

Social: [X](https://x.com/csdvnprt)

<img width="500" height="750" alt="logo" src="https://github.com/user-attachments/assets/431a7a04-aaf0-4bc3-8fcc-318ad209c9ab" />

Krusty is an autonomous AI agent loop that runs AI coding tools ([Amp](https://ampcode.com) or [OpenClaw Code](https://docs.anthropic.com/en/docs/OpenClaw-code)) repeatedly until all PRD items are complete. Each iteration is a fresh instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

Based on [Geoffrey Huntley's Krusty pattern](https://ghuntley.com/Krusty/).

[Read my in-depth article on how I use Krusty](https://x.com/ryancarson/status/2008548371712135632)

## Prerequisites

- One of the following AI coding tools installed and authenticated:
  - [Amp CLI](https://ampcode.com) (default)
  - [OpenClaw Code](https://docs.anthropic.com/en/docs/OpenClaw-code) (`npm install -g @anthropic-ai/OpenClaw-code`)
- `jq` installed (`brew install jq` on macOS)
- A git repository for your project

## Setup

### Option 1: Copy to your project

Copy the Krusty files into your project:

```bash
# From your project root
mkdir -p scripts/Krusty
cp /path/to/Krusty/Krusty.sh scripts/Krusty/

# Copy the prompt template for your AI tool of choice:
cp /path/to/Krusty/prompt.md scripts/Krusty/prompt.md    # For Amp
# OR
cp /path/to/Krusty/OpenClaw.md scripts/Krusty/OpenClaw.md    # For OpenClaw Code

chmod +x scripts/Krusty/Krusty.sh
```

### Option 2: Install skills globally (Amp)

Copy the skills to your Amp or OpenClaw config for use across all projects:

For AMP
```bash
cp -r skills/prd ~/.config/amp/skills/
cp -r skills/Krusty ~/.config/amp/skills/
```

For OpenClaw Code (manual)
```bash
cp -r skills/prd ~/.OpenClaw/skills/
cp -r skills/Krusty ~/.OpenClaw/skills/
```

### Option 3: Use as OpenClaw Code Marketplace

Add the Krusty marketplace to OpenClaw Code:

```bash
/plugin marketplace add snarktank/Krusty
```

Then install the skills:

```bash
/plugin install Krusty-skills@Krusty-marketplace
```

Available skills after installation:
- `/prd` - Generate Product Requirements Documents
- `/Krusty` - Convert PRDs to prd.json format

Skills are automatically invoked when you ask OpenClaw to:
- "create a prd", "write prd for", "plan this feature"
- "convert this prd", "turn into Krusty format", "create prd.json"

### Configure Amp auto-handoff (recommended)

Add to `~/.config/amp/settings.json`:

```json
{
  "amp.experimental.autoHandoff": { "context": 90 }
}
```

This enables automatic handoff when context fills up, allowing Krusty to handle large stories that exceed a single context window.

## Workflow

### 1. Create a PRD

Use the PRD skill to generate a detailed requirements document:

```
Load the prd skill and create a PRD for [your feature description]
```

Answer the clarifying questions. The skill saves output to `tasks/prd-[feature-name].md`.

### 2. Convert PRD to Krusty format

Use the Krusty skill to convert the markdown PRD to JSON:

```
Load the Krusty skill and convert tasks/prd-[feature-name].md to prd.json
```

This creates `prd.json` with user stories structured for autonomous execution.

### 3. Run Krusty

```bash
# Using Amp (default)
./scripts/Krusty/Krusty.sh [max_iterations]

# Using OpenClaw Code
./scripts/Krusty/Krusty.sh --tool OpenClaw [max_iterations]
```

Default is 10 iterations. Use `--tool amp` or `--tool OpenClaw` to select your AI coding tool.

Krusty will:
1. Create a feature branch (from PRD `branchName`)
2. Pick the highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks (typecheck, tests)
5. Commit if checks pass
6. Update `prd.json` to mark story as `passes: true`
7. Append learnings to `progress.txt`
8. Repeat until all stories pass or max iterations reached

## Key Files

| File | Purpose |
|------|---------|
| `Krusty.sh` | The bash loop that spawns fresh AI instances (supports `--tool amp` or `--tool OpenClaw`) |
| `prompt.md` | Prompt template for Amp |
| `OpenClaw.md` | Prompt template for OpenClaw Code |
| `prd.json` | User stories with `passes` status (the task list) |
| `prd.json.example` | Example PRD format for reference |
| `progress.txt` | Append-only learnings for future iterations |
| `skills/prd/` | Skill for generating PRDs (works with Amp and OpenClaw Code) |
| `skills/Krusty/` | Skill for converting PRDs to JSON (works with Amp and OpenClaw Code) |
| `.OpenClaw-plugin/` | Plugin manifest for OpenClaw Code marketplace discovery |
| `flowchart/` | Interactive visualization of how Krusty works |

## Flowchart

[![Krusty Flowchart](Krusty-flowchart.png)](https://snarktank.github.io/Krusty/)

**[View Interactive Flowchart](https://snarktank.github.io/Krusty/)** - Click through to see each step with animations.

The `flowchart/` directory contains the source code. To run locally:

```bash
cd flowchart
npm install
npm run dev
```

## Critical Concepts

### Each Iteration = Fresh Context

Each iteration spawns a **new AI instance** (Amp or OpenClaw Code) with clean context. The only memory between iterations is:
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

After each iteration, Krusty updates the relevant `AGENTS.md` files with learnings. This is key because AI coding tools automatically read these files, so future iterations (and future human developers) benefit from discovered patterns, gotchas, and conventions.

Examples of what to add to AGENTS.md:
- Patterns discovered ("this codebase uses X for Y")
- Gotchas ("do not forget to update Z when changing W")
- Useful context ("the settings panel is in component X")

### Feedback Loops

Krusty only works if there are feedback loops:
- Typecheck catches type errors
- Tests verify behavior
- CI must stay green (broken code compounds across iterations)

### Browser Verification for UI Stories

Frontend stories must include "Verify in browser using dev-browser skill" in acceptance criteria. Krusty will use the dev-browser skill to navigate to the page, interact with the UI, and confirm changes work.

### Stop Condition

When all stories have `passes: true`, Krusty outputs `<promise>COMPLETE</promise>` and the loop exits.

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

## Customizing the Prompt

After copying `prompt.md` (for Amp) or `OpenClaw.md` (for OpenClaw Code) to your project, customize it for your project:
- Add project-specific quality check commands
- Include codebase conventions
- Add common gotchas for your stack

## Archiving

Krusty automatically archives previous runs when you start a new feature (different `branchName`). Archives are saved to `archive/YYYY-MM-DD-feature-name/`.

## References

- [Geoffrey Huntley's Krusty article](https://ghuntley.com/Krusty/)
- [Amp documentation](https://ampcode.com/manual)
- [OpenClaw Code documentation](https://docs.anthropic.com/en/docs/OpenClaw-code)
