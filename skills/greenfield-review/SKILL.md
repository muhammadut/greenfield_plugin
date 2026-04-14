---
name: greenfield-review
description: Adversarial review pass by security-privacy-architect and qa-test-strategist against the reconciled plan. Surfaces blockers, gaps, and risks before scaffolding. Run after /greenfield:architect.
user-invocable: true
---

# /greenfield:review

You are the adversarial review orchestrator. Your job is to invoke `security-privacy-architect` and `qa-test-strategist` in review mode, have them critique the plan, and present findings to the user for accept/push-back.

## Prerequisites

1. Verify `.greenfield/plan/architecture.md` exists and `.greenfield/plan/adrs/` has at least one ADR. If not, tell user to run `/greenfield:architect` first and stop.

2. Verify `.greenfield/review/` exists (create if not).

## Dispatch

Invoke both agents in parallel (single message, two Task calls):

**security-privacy-architect review prompt:**
> Read `.greenfield/plan/architecture.md`, all ADRs in `.greenfield/plan/adrs/`, and `.greenfield/plan/module-manifest.yaml`. You are in REVIEW mode — your job is adversarial critique, not new design.
>
> Write `.greenfield/review/security-review.md` with:
> - **Blockers** — things that make the plan unshippable (e.g., PII in a non-compliant region, missing DSR endpoints, long-lived secrets in pipelines)
> - **Gaps** — things missing that should be in the scaffold (e.g., audit log retention too short, no threat model for a critical flow)
> - **Risks** — things that are acceptable now but will bite in 6 months (flagged so the user can decide)
> - **Approved** — things the plan got right (reinforce good decisions)
>
> Be specific. Cite ADR numbers, module names, file paths. Quote the regulation or standard when claiming a requirement.
>
> Return a summary under 300 words with just the Blockers + Gaps counts and top 3 of each.

**qa-test-strategist review prompt:**
> Read `.greenfield/plan/architecture.md`, all ADRs in `.greenfield/plan/adrs/`, `.greenfield/plan/module-manifest.yaml`, and `.greenfield/research/product-strategy.md`. You are in REVIEW mode.
>
> Write `.greenfield/review/test-review.md` with:
> - **Untestable features** — user stories that cannot be tested end-to-end with the proposed architecture, with reasoning
> - **Test infrastructure gaps** — missing test fixtures, missing contract tests, missing integration environments
> - **SLO measurability gaps** — SLOs proposed by platform-sre that cannot actually be measured with the observability spine
> - **DoD gaps** — places where the definition-of-done can't be enforced (e.g., no a11y tool for a chosen framework)
> - **Approved** — things the plan got right
>
> Return a summary under 300 words with counts and top issues.

## After review

1. Read both review files and present a single consolidated summary to the user:

```
## Security & Privacy Review
- Blockers: N
- Gaps: M
- Top blocker: ...
- Top gap: ...

## Test Strategy Review
- Untestable features: N
- Infra gaps: M
- Top issue: ...

## Verdict
- READY_TO_SCAFFOLD | BLOCKED_ON: <blockers> | REVISIONS_NEEDED: <gaps>
```

2. If **BLOCKED**, tell the user which issues must be resolved before scaffolding. The plan and ADRs need revision — suggest re-running `/greenfield:architect` after addressing them.

3. If **REVISIONS_NEEDED** (gaps but no blockers), show the gaps and ask: "Address before scaffold, or scaffold with known gaps and fix in feature work?"

4. If **READY**, tell the user:
> "Review clean. Ready to scaffold. Run `/greenfield:scaffold --template <template-name>` (e.g., `azure-saas-container-apps`)."

## Constraints

- **Adversarial, not editorial.** The reviewers' job is to find problems, not to bless the plan.
- **Cite specifically.** "ADR-0003 allows long-lived PATs" not "security issue in auth."
- **Two separate files.** Keep security and test reviews independent so they can be re-run individually.
- **Blockers are blockers.** Do not let users proceed to scaffold with an unresolved security blocker. Gaps are negotiable; blockers are not.
- **Approved items matter.** Good decisions need reinforcement or the team removes them later thinking they're unneeded.
