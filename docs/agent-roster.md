# Agent Roster & Handoff Protocol

## The 9 agents

Each agent produces exactly one artifact in `.greenfield/research/`. No agent reads its own output; the system-design-architect reads all 9 in the reconciliation phase.

| Agent | Input | Output | Invocation phase |
|---|---|---|---|
| product-strategist | context.md | research/product-strategy.md | research |
| system-design-architect | context.md + product-strategy.md | research/system-design.md | research |
| system-design-architect (reconcile) | all research/*.md | plan/* | architect |
| ux-researcher | context.md + product-strategy.md | research/ux-plan.md | research |
| design-director | context.md + assets/* | research/design-direction.md | research |
| security-privacy-architect | context.md + product-strategy.md | research/security-privacy.md | research |
| security-privacy-architect (review) | plan/* | review/security-review.md | review |
| data-architect | context.md + product-strategy.md | research/data-architecture.md | research |
| cloud-architect | context.md + system-design.md | research/cloud-architecture.md | research |
| platform-sre | context.md + system-design.md + cloud-architecture.md | research/platform-sre.md | research |
| qa-test-strategist | context.md + product-strategy.md | research/test-strategy.md | research |
| qa-test-strategist (review) | plan/* | review/test-review.md | review |

## Handoff protocol

Agents never talk to each other directly. All communication is via files in `.greenfield/`. Each agent:
1. Reads its declared inputs (above)
2. Writes its declared output (above)
3. Returns a short summary (<300 words) to the orchestrating skill

The system-design-architect plays the reconciliation role — it is the only agent that reads every other agent's output. In reconciliation mode, it produces the canonical plan in `.greenfield/plan/`, resolving any contradictions explicitly in ADRs.

The security-privacy-architect and qa-test-strategist are the only agents invoked in the review phase. They read `.greenfield/plan/` and produce adversarial critiques in `.greenfield/review/`.

## Why file-based, not message-based

- **Context isolation**: each agent has minimal context; no token burn on other agents' deliberations
- **Debuggability**: the user can inspect what each agent actually produced
- **Resumability**: partial workflows can be resumed by re-running specific agents
- **Git-commitable**: the plan is the repo's source of truth, not an in-memory artifact

## When to add/remove agents

**Add an agent** if:
- A new deliverable is needed that doesn't fit any existing agent's charter
- An existing agent is producing two distinct, independently-reviewable artifacts

**Do not add** if:
- The "role" is just a re-framing of an existing one (e.g., "Full-Stack Engineer" overlaps system-design-architect)
- The role only exists during implementation (push it to the Rival loop instead)

**Remove an agent** if:
- Its output is consistently empty or trivial for your project types
- Its output is always subsumed by another agent's output

The 9 is a default, not a mandate. For a tiny internal tool with no users, you might cut to 5. For a regulated healthcare product, you might add a compliance-specialist. Edit `.claude-plugin/plugin.json` and the `greenfield-research/` skill accordingly.
