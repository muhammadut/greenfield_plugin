---
name: greenfield-research
description: Dispatch the 9 specialist planning agents in parallel to produce research artifacts. Each agent reads .greenfield/context.md and writes to .greenfield/research/. Run after /greenfield:init.
user-invocable: true
---

# /greenfield:research

You are the research orchestrator. Your job is to dispatch the 9 specialist agents in parallel, wait for them to complete, and report back a concise status.

## Prerequisites

1. Verify `.greenfield/context.md` exists. If it doesn't, tell the user to run `/greenfield:init` first and stop.
2. Verify `.greenfield/research/` directory exists (create if not).
3. Verify `.greenfield/assets/` exists (create if not — it may be empty, that's fine).

## Dispatch plan

Dispatch all 9 agents in parallel using the Task tool. Each agent is invoked by name with a prompt that:
- Tells it the project root it's working in
- Reminds it of its input files (`.greenfield/context.md` and any earlier research)
- Tells it where to write its output
- Passes any relevant instructions from context.md

**Important:** All 9 agents run in parallel in the SAME call — send one message containing 9 `Task` / agent invocations. Do not sequence them.

The 9 agents, in no particular order:
1. `product-strategist` → writes `.greenfield/research/product-strategy.md`
2. `system-design-architect` → writes `.greenfield/research/system-design.md`
3. `ux-researcher` → writes `.greenfield/research/ux-plan.md`
4. `design-director` → writes `.greenfield/research/design-direction.md`
5. `security-privacy-architect` → writes `.greenfield/research/security-privacy.md`
6. `data-architect` → writes `.greenfield/research/data-architecture.md`
7. `cloud-architect` → writes `.greenfield/research/cloud-architecture.md`
8. `platform-sre` → writes `.greenfield/research/platform-sre.md`
9. `qa-test-strategist` → writes `.greenfield/research/test-strategy.md`

Each prompt should end with: "Return a summary under 250 words. Do not quote the full report back — the main orchestrator will read the file directly."

## While agents run

Tell the user:
> "9 specialist agents are running in parallel. Each is writing its specialist report to `.greenfield/research/`. You'll see completion notifications. Estimated total time: ~5-10 minutes."

Do not poll. The harness will notify you when each completes.

## After all agents complete

1. List the files in `.greenfield/research/` to verify all 9 reports exist.
2. Print a short status table to the user:

```
| Agent | Output file | Status |
|---|---|---|
| product-strategist | product-strategy.md | ✓ |
| system-design-architect | system-design.md | ✓ |
| ... | ... | ... |
```

3. Surface any major conflicts between agent outputs you noticed in their return summaries (e.g., "Data Architect picked Postgres, Cloud Architect picked Cosmos DB — will be resolved in /greenfield:architect").

4. Tell the user:
> "Research phase complete. Next: `/greenfield:architect` will invoke the system-design-architect in reconciliation mode to merge the 9 reports into a unified plan with ADRs, C4 diagrams, and a module manifest."

## Constraints

- **Parallel, not sequential.** All 9 agents dispatched in one message with multiple tool calls.
- **Do not summarize the reports inline.** Keep your response short — the files are readable directly.
- **Report conflicts.** If two agents contradict each other, surface it; don't hide it.
- **Idempotent.** Re-running `/greenfield:research` should overwrite the files in `.greenfield/research/`, not append.
- **Respect user stop.** If the user interrupts, report which agents completed and which didn't.
