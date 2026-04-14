---
name: greenfield-status
description: Report the current state of the Greenfield workflow — which phases are complete, which files exist, what's next. Run any time to see where you are.
user-invocable: true
---

# /greenfield:status

You are the state reporter. Your job is to inspect `.greenfield/` in the current directory and print a concise status table.

## What to check

Walk through the phases and check for expected files:

### Phase 0: Init
- `.greenfield/context.md` exists?
- `.greenfield/assets/` exists? (list files if so)

### Phase 1: Research
For each of the 9 files in `.greenfield/research/`:
- `product-strategy.md`
- `system-design.md`
- `ux-plan.md`
- `design-direction.md`
- `security-privacy.md`
- `data-architecture.md`
- `cloud-architecture.md`
- `platform-sre.md`
- `test-strategy.md`

Report present/missing.

### Phase 2: Architecture
- `.greenfield/plan/architecture.md` exists?
- `.greenfield/plan/adrs/` — count of ADR files
- `.greenfield/plan/c4/` — count of diagrams
- `.greenfield/plan/module-manifest.yaml` exists?
- `.greenfield/plan/open-questions.md` exists and has content?

### Phase 3: Review
- `.greenfield/review/security-review.md` exists?
- `.greenfield/review/test-review.md` exists?
- Parse for BLOCKER count (grep for "Blockers:" or "BLOCKER" headings)

### Phase 4: Scaffold
- `.greenfield/SCAFFOLD_COMPLETE` exists? Print the timestamp and template name from it.

## Output format

Print a table:

```
╔════════════════════════════════════════════════════════╗
║  Greenfield Workflow Status                            ║
╠════════════════════════════════════════════════════════╣
║ Phase 0: Init           [✓]  context.md  (N assets)    ║
║ Phase 1: Research       [✓]  9/9 reports               ║
║ Phase 2: Architecture   [✓]  5 ADRs, 2 C4 diagrams     ║
║ Phase 3: Review         [✓]  0 blockers, 2 gaps        ║
║ Phase 4: Scaffold       [ ]  not yet run               ║
╚════════════════════════════════════════════════════════╝

Next command: /greenfield:scaffold --template azure-saas-container-apps
```

If any phase is incomplete, print the specific missing files and the command that would produce them.

If `.greenfield/` does not exist at all, print:
> "No Greenfield workflow found in this directory. Run `/greenfield:init` to start."

## Constraints

- **Read-only.** Do not modify any files. Do not create `.greenfield/` if it doesn't exist.
- **Concise.** The status should fit on one screen.
- **Actionable.** Every state must surface the next command.
- **Safe on empty.** Handle directories with missing subfolders gracefully.
