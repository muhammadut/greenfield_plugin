# Greenfield Plugin — Design Document

> v0.1.0 — synthesized from 4 parallel research streams in `../../greenfield-research/`

## Problem

AI coding assistants parachute users into `app.py` on an empty directory. The planning phase — the part where architectural decisions that cost 10-30x more to undo get locked in — is either skipped or buried inside the same agent that writes code. Enterprise greenfield projects need more discipline than that.

Research from ThoughtWorks Tech Radar Vol 33 ("Anchoring coding agents to a reference application") confirms that the most effective pattern is to produce a high-quality reference application first, then use it as the paved road every future coding agent is anchored against.

## Thesis

Split greenfield work into two plugins with a clean boundary at the first commit:

- **Greenfield (this plugin)** — planning phase. 9 specialist agents research, architect, and scaffold a production-grade reference app. Runs once per project.
- **Rival (separate)** — implementation loop. Plan / execute / verify per feature. Runs per feature after Greenfield is done.

The scaffold Greenfield produces is the reference application for Rival to anchor against.

## The 9-agent roster

| # | Agent | Deliverable | Reasoning |
|---|---|---|---|
| 1 | product-strategist | Press release, MVP scope, user stories | Amazon Working Backwards discipline; blocks architecture until WHAT is clear |
| 2 | system-design-architect | C4, ADRs, API contracts, module manifest | Load-bearing role; also runs reconciliation phase |
| 3 | ux-researcher | User journeys, IA, accessibility plan | How users move — separate from visual taste |
| 4 | design-director | Design system pick, tokens, component install list | NEW role, reads mood board (screenshots/URLs/Nano Banana mockups) |
| 5 | security-privacy-architect | Threat model, data classification, compliance baseline | Highest shift-left leverage; PIPEDA/Law 25/GDPR baseline from day 1 |
| 6 | data-architect | Logical model, event taxonomy, PII map | Schemas/events are irreversible tech debt |
| 7 | cloud-architect | Service list, IaC pattern, cost estimate | Azure by default (per first user), swappable for AWS/GCP |
| 8 | platform-sre | CI/CD, observability spine, SLOs | NOT backend engineer — owns deploys, observability, dev env |
| 9 | qa-test-strategist | Test strategy, DoD, CI gates | Epistemic separation of test design from builders (AgentCoder 91.5% HumanEval proof) |

## Why 9 agents, not 6 or 12

Research stream 2 (multi-agent systems benchmarks, AgentCoder/MAST 2025) argues for **minimalism in code-generation loops** — extra agents hurt quality because handoff friction dominates. Stream 4 (human team composition) argues for **breadth in planning phase** because architectural decisions compound. Both are right *about different phases*.

The resolution: planning earns breadth, implementation earns minimalism. The 9 agents live only in the planning phase. Implementation hands off to Rival, which has 3-4 agents. Total system is not 9 agents churning on every feature — it's 9 agents once, then 3-4 per feature.

## Workflow state machine

```
empty dir
  │
  ▼
/greenfield:init       → .greenfield/context.md, .greenfield/assets/
  │
  ▼
/greenfield:research   → 9 parallel agents → .greenfield/research/*.md
  │
  ▼
/greenfield:architect  → system-design-architect in reconcile mode → .greenfield/plan/
  │
  ▼
/greenfield:review     → security + qa adversarial pair → .greenfield/review/
  │
  ├─── BLOCKED ──▶ revise and re-run architect
  │
  ▼
/greenfield:scaffold   → hydrate templates/<name>/ → project files
  │
  ▼
git commit (hand off to Rival)
```

## Key decisions locked in research

- **Bicep + AVM** over Terraform for Azure-only projects (CAF Terraform module archived August 2026)
- **Entra External ID** over Auth0/Clerk for Canadian SaaS (B2C stopped accepting new customers May 1, 2025; Auth0/Clerk trigger Law 25 TIA requirements)
- **Azure Container Apps** over App Service as the default compute
- **OpenFeature** over vendor-specific feature flag SDKs
- **SLSA L2 minimum**: SBOM (CycloneDX) + Sigstore cosign + OIDC federation
- **365+ day Log Analytics retention** — the 30-day default is a PIPEDA audit trap
- **Templates in Structurizr DSL, not PNG** — every plan artifact is git-committable text

## References

Full research reports: `../../greenfield-research/`
- `01-enterprise-greenfield-practices.md`
- `02-multi-agent-systems.md`
- `03-azure-patterns.md`
- `04-team-composition.md`

Key external references:
- ThoughtWorks Technology Radar Vol 33 — "Anchoring coding agents to a reference application"
- AgentCoder (91.5% HumanEval with 3 agents vs MetaGPT's ~85% with 8+)
- MAST paper, NeurIPS 2025 — multi-agent compute accounting
- Azure Well-Architected Framework (5 pillars)
- PIPEDA fair information principles + Quebec Law 25 §27
- Google SRE workbook (SLOs, error budgets)

## Open questions (for v0.2)

- Should the scaffold skill run `npm install` / `pip install` automatically? Currently no — user reviews first.
- How do we handle template version drift — `cruft`-style template updates?
- Should there be a `/greenfield:feature "..."` command for capturing features after init?
- How do we A/B test the 9-agent roster against a single-agent baseline per the MAST findings?
