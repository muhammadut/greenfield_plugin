---
name: greenfield-architect
description: Reconcile the 9 research reports into a unified architectural plan with ADRs, C4 diagrams, and a committable module manifest. Invokes system-design-architect in reconciliation mode. Run after /greenfield:research.
user-invocable: true
---

# /greenfield:architect

You are the architecture reconciliation orchestrator. Your job is to invoke the `system-design-architect` agent in reconciliation mode to merge the 9 specialist research reports into a unified, committable plan.

## Prerequisites

1. Verify all 9 files exist in `.greenfield/research/`:
   - `product-strategy.md`
   - `system-design.md`
   - `ux-plan.md`
   - `design-direction.md`
   - `security-privacy.md`
   - `data-architecture.md`
   - `cloud-architecture.md`
   - `platform-sre.md`
   - `test-strategy.md`

   If any are missing, tell the user: "Missing research files: [list]. Run `/greenfield:research` first (or re-run for the missing ones)." and stop.

2. Verify `.greenfield/plan/` exists (create if not).

## Invocation

Dispatch the `system-design-architect` agent with this prompt:

> You are invoked in RECONCILIATION mode. Read every file in `.greenfield/research/`. Merge them into a unified architectural plan.
>
> Produce these outputs in `.greenfield/plan/`:
> - `architecture.md` — the unified plan. Covers: system purpose, stakeholders, architectural drivers, tech stack (locked), C4 L1 and L2 summary, data model summary, security/compliance summary, operational plan, risk register.
> - `adrs/ADR-0001-*.md`, `ADR-0002-*.md`, ... — one file per decision, numbered, committable, Nygard format (Status / Context / Decision / Consequences).
> - `c4/context.dsl` and `c4/containers.dsl` — Structurizr DSL for C4 L1 and L2 diagrams.
> - `module-manifest.yaml` — the machine-readable list of modules/services to be scaffolded (used by /greenfield:scaffold).
> - `open-questions.md` — unresolved items requiring human decision before scaffolding.
>
> Resolution protocol: when two specialists contradict each other, do not silently pick one. Surface the trade-off explicitly in the relevant ADR and pick the option with the stronger case, documenting the rejected option.
>
> Return a summary under 400 words listing: (a) the top 10 ADRs by name, (b) the final tech stack, (c) any unresolved open questions that block scaffold. Do not quote the full architecture document back.

## After reconciliation

1. List files in `.greenfield/plan/` to verify outputs.
2. Show a short status report to the user: tech stack locked, ADR count, any open questions.
3. If open questions exist, present them and ask the user to resolve each before proceeding.
4. Tell the user:
> "Architecture reconciled. Next: `/greenfield:review` runs an adversarial security + QA pass before scaffolding. Or if you're ready: `/greenfield:scaffold --template <name>` to emit the reference app."

## Constraints

- **Do not write the plan yourself.** Dispatch the system-design-architect agent for it — it has the context and expertise.
- **Surface conflicts explicitly.** The reconciliation is valuable precisely because it forces choices; don't bury them.
- **Open questions block scaffold.** If ADRs depend on open questions, the scaffold should not run until they're resolved.
- **Commit-friendly outputs.** Every file in `.greenfield/plan/` should be git-committable markdown or YAML or DSL. No binary, no diagrams-as-images.
