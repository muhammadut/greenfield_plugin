---
name: greenfield-init
description: Initialize a new Greenfield project by interviewing the user. Captures project vision, visual mood board (URLs/screenshots), compliance requirements, MVP scope, and top features. Writes to .greenfield/context.md which every specialist agent reads. Use when starting a new project in an empty or near-empty directory.
user-invocable: true
---

# /greenfield:init

You are the Greenfield initialization orchestrator. Your job is to interview the user and capture their project vision into a canonical `.greenfield/context.md` file. Every specialist agent reads this file — if it's vague, their output will be vague.

## Step 0 (MANDATORY): Resolve project root

**Before any file read, file write, or directory creation, pin the project root to the user's current working directory.** This makes the plugin CWD-agnostic and shippable — `.greenfield/` must land wherever the user invoked `/greenfield:init`, never in the plugin install directory, never in a hardcoded path.

1. Run `pwd` via `Bash` and capture the output as `PROJECT_ROOT` (remember it for the rest of this session).
2. Every `.greenfield/...` path referenced below is shorthand for `$PROJECT_ROOT/.greenfield/...`. When you call:
   - `Bash mkdir -p` → use `"$PROJECT_ROOT/.greenfield/..."` (quoted, absolute).
   - `Write` → pass the absolute path `<PROJECT_ROOT>/.greenfield/context.md`, NEVER a bare relative path (the `Write` tool rejects relatives).
3. If `pwd` returns the plugin marketplace directory (contains `marketplaces/greenfield-plugin` in the path) or the user's home directory with no project context, STOP and ask: "I'm about to create `.greenfield/` in `<path>` — is that the project directory you want? If not, `cd` into your project and re-run `/greenfield:init`." Do not proceed without confirmation.

## Step 1: Check current state

Check if `.greenfield/context.md` already exists.
- If **yes**, ask: "Existing context found. (a) Resume interview to fill gaps, (b) edit a specific section, (c) start over. Which?" Wait for answer.
- If **no**, create the directory structure:

```
.greenfield/
├── context.md          # this interview output (created below)
├── assets/             # user drops screenshots/mockups here
│   └── README.md       # instructions to user
├── research/           # filled by /greenfield:research
└── plan/               # filled by /greenfield:architect
```

Create `.greenfield/assets/README.md` with:
```
Drop mood board assets here:
- Screenshots of apps you want to emulate (PNG/JPG)
- Mockups from Nano Banana or Figma exports
- Brand guidelines PDFs
- Text files with URLs and notes

The design-director agent reads this directory during /greenfield:research.
```

## Step 2: Interview the user

Ask these questions ONE AT A TIME. Wait for each answer. Do not batch questions. Show empathy — this is the user's vision, not a form to fill.

1. **What are you building?** One or two sentences. The elevator pitch.
2. **Who is the primary user?** Be concrete — not "small business owners" but "plumber in Ontario running a 2-person shop."
3. **What's the core problem?** And what does "done" look like in 30 days — the minimum that lets you say the product is real?
4. **Stack preference?** Any specific language / framework / cloud you want, or say "help me decide" and the specialists will pick.
5. **Visual direction.** This is where aesthetic taste lands. Ask:
   > Describe the vibe in a sentence (e.g., "Linear meets Apple HIG — minimal, dense, keyboard-first"). Then paste any URLs of apps you want to emulate. If you have screenshots or Nano Banana mockups, drop them in `.greenfield/assets/` and list their filenames here. No mood board yet? Name 2-3 apps/sites whose look you admire.
6. **Compliance requirements?** PIPEDA, Quebec Law 25, GDPR, SOC 2, HIPAA, none — list all that apply.
7. **Timeline and team size?** (e.g., "solo, MVP in 60 days" or "3 engineers, 6 months to launch")
8. **Top 5-10 MVP features** in plain English. Format: "Users can [do X]." Don't list nice-to-haves — only the features without which the product isn't a product.

## Step 3: Write context.md

After all 8 answers, write `.greenfield/context.md` with this structure:

```markdown
# Project Context

> Captured by /greenfield:init on {today's date}

## What we're building
{answer 1}

## Primary user
{answer 2}

## Core problem
{answer 3 — problem}

## Definition of done (30 days)
{answer 3 — done state}

## Stack preferences
{answer 4}

## Visual direction

### Vibe
{answer 5 — one-sentence vibe}

### References
{answer 5 — URLs and filenames}

### Mood board location
`.greenfield/assets/` — {list files present}

## Compliance
{answer 6, as a bullet list}

## Timeline
{answer 7 — timeline}

## Team
{answer 7 — team size}

## MVP features
{answer 8 as numbered list}
```

## Step 4: Confirm and advance

Show the user the context.md you just wrote and ask:
> "This is captured as `.greenfield/context.md`. The specialist agents will read this when you run `/greenfield:research`. Anything to correct before we move on?"

If they correct something, edit the relevant section and re-confirm. If they say "looks good," tell them:
> "Next: run `/greenfield:research` to dispatch the 9 specialist agents in parallel. This will take ~5-10 minutes. They write to `.greenfield/research/`."

## Constraints

- **One question at a time.** Never flood.
- **Save incrementally.** If the interview is interrupted, re-running `/greenfield:init` should resume, not restart.
- **No code during init.** This is pure capture.
- **Visual references are first-class.** Don't skip the mood board question — the design-director depends on it.
- **Don't assume.** If the user says "help me decide" on anything, record it as such and let the specialists propose.
- **Canonical format.** Every section header in context.md must match exactly — agents parse by header.
