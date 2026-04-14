# Greenfield Workflow

## State machine

```
      ┌──────────────┐
      │  empty dir   │
      └──────┬───────┘
             │
             │ /greenfield:init
             ▼
      ┌──────────────┐
      │   INIT       │──── .greenfield/context.md
      │              │──── .greenfield/assets/
      └──────┬───────┘
             │
             │ /greenfield:research
             ▼
      ┌──────────────┐
      │  RESEARCH    │──── 9 parallel agents
      │              │──── .greenfield/research/*.md (9 files)
      └──────┬───────┘
             │
             │ /greenfield:architect
             ▼
      ┌──────────────┐
      │  ARCHITECT   │──── system-design-architect reconciliation
      │              │──── .greenfield/plan/architecture.md
      │              │──── .greenfield/plan/adrs/
      │              │──── .greenfield/plan/c4/
      │              │──── .greenfield/plan/module-manifest.yaml
      └──────┬───────┘
             │
             │ /greenfield:review
             ▼
      ┌──────────────┐
      │   REVIEW     │──── security + qa adversarial pair
      │              │──── .greenfield/review/security-review.md
      │              │──── .greenfield/review/test-review.md
      └──────┬───────┘
             │
             ├── BLOCKED ──▶ revise plan, re-run architect
             │
             │ /greenfield:scaffold --template <name>
             ▼
      ┌──────────────┐
      │  SCAFFOLD    │──── hydrate templates/<name>/
      │              │──── writes source tree, infra, CI/CD, compliance
      │              │──── .greenfield/SCAFFOLD_COMPLETE
      └──────┬───────┘
             │
             │ user reviews, installs deps, git init, git commit
             ▼
      ┌──────────────┐
      │  BROWNFIELD  │──── hand off to Rival for features
      └──────────────┘
```

## Time budget

| Phase | Command | Agent count | Wall time |
|---|---|---|---|
| Init | /greenfield:init | 0 (interview only) | 5-10 min (human input) |
| Research | /greenfield:research | 9 parallel | 5-10 min |
| Architect | /greenfield:architect | 1 (system-design-architect reconcile) | 2-5 min |
| Review | /greenfield:review | 2 parallel (security + qa) | 2-5 min |
| Scaffold | /greenfield:scaffold --template <name> | 0 (template hydration) | 1-3 min |
| **Total** | | | **~20-35 min** |

Plus human review time between phases. Expect a real first-time run on a non-trivial project to be 1-2 hours end-to-end.

## Checkpoints

The user can exit and resume at any phase. Re-running `/greenfield:status` shows where you are. Re-running a phase command overwrites only that phase's outputs.

The only irreversible step is `/greenfield:scaffold` (it writes to the target directory). Everything before it lives in `.greenfield/` and is scoped.

## Failure modes

### Research agent produces empty/thin output
Cause: context.md is too vague. Fix: re-run `/greenfield:init` and push the user for more specifics in the weak section.

### Architect finds unresolvable contradictions
Cause: two specialists made contradictory assumptions about the stack or scale. Fix: architect surfaces the conflict in `open-questions.md`, user resolves, re-run architect.

### Review finds blockers
Cause: plan has unshippable issues (e.g., missing DSR endpoints, long-lived secrets). Fix: revise plan, re-run architect, re-run review. Do not scaffold with unresolved blockers.

### Scaffold target has conflicting files
Cause: target directory is not clean. Fix: scaffold aborts; user cleans and retries.

## Reentrancy

The entire workflow is reentrant — every skill is idempotent on its inputs. If the harness crashes mid-research, re-run `/greenfield:research` and only the missing agents will actually do work (they'll see existing files and skip). Same for architect and review.

The exception: scaffold. Once scaffolded, the project is brownfield and the workflow ends.
