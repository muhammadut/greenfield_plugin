# Greenfield Plugin

**Enterprise greenfield scaffolding for Claude Code.** Take an empty directory to a production-grade reference application in one session, using 9 specialist AI agents that research, architect, review, and scaffold your project before any code is written.

> **Status:** v0.1.0 — workflow is complete and runnable end-to-end. The first template (`azure-saas-container-apps`) is a stub and must be built out before `/greenfield:scaffold` produces a usable project.

---

## Table of Contents

1. [What is Greenfield?](#what-is-greenfield)
2. [Why it exists](#why-it-exists)
3. [Core principle: scaffold as anchor](#core-principle-scaffold-as-anchor)
4. [Installation](#installation)
5. [Quick start](#quick-start)
6. [The workflow in detail](#the-workflow-in-detail)
7. [The 9 agents — full reference](#the-9-agents--full-reference)
8. [The 6 commands — full reference](#the-6-commands--full-reference)
9. [State directory layout](#state-directory-layout)
10. [Templates](#templates)
11. [Architecture decisions baked in](#architecture-decisions-baked-in)
12. [How Greenfield differs from other AI coding tools](#how-greenfield-differs-from-other-ai-coding-tools)
13. [Troubleshooting](#troubleshooting)
14. [FAQ](#faq)
15. [Roadmap](#roadmap)
16. [Contributing & extending](#contributing--extending)
17. [License](#license)

---

## What is Greenfield?

Greenfield is a Claude Code plugin for the **planning phase** of new software projects. You run it once per new project. It does the work that almost every AI coding tool skips:

1. **Interviews you** about what you're building (`/greenfield:init`)
2. **Captures your visual mood board** — screenshots, URLs, descriptions
3. **Dispatches 9 specialist AI agents in parallel** to research the project from every angle (product, architecture, UX, design system, security, data, cloud, platform/SRE, testing)
4. **Reconciles** their outputs into a unified architectural plan with ADRs and C4 diagrams
5. **Adversarially reviews** the plan with security and QA specialists
6. **Scaffolds** a production-grade reference application with IaC, CI/CD, observability, compliance baseline, and design tokens already wired in

After the first commit, Greenfield's job is done. The project is now brownfield, and you switch to your normal feature workflow (e.g., the brownfield plugin or any feature-level loop).

**The 9 agents in one line each:**

| Agent | What it produces |
|---|---|
| **product-strategist** | Press release, problem statement, MVP scope, user stories |
| **system-design-architect** | C4 diagrams, ADRs, API contracts, module manifest |
| **ux-researcher** | User journeys, information architecture, accessibility plan |
| **design-director** | Design system pick + concrete tokens (reads your mood board) |
| **security-privacy-architect** | Threat model, PIPEDA/Law 25/GDPR baseline, DSR endpoint specs |
| **data-architect** | Logical data model, event taxonomy, PII map |
| **cloud-architect** | Cloud services list, IaC pattern, cost estimate |
| **platform-sre** | CI/CD blueprint, observability spine, SLOs |
| **qa-test-strategist** | Test strategy, definition-of-done, CI gates |

---

## Why it exists

Most AI coding assistants drop you into an empty `app.py` and start writing code. The expensive part — architectural decisions that cost 10-30x more to undo than code changes — gets either skipped entirely or buried inside the same agent that's writing the code. Enterprise greenfield projects deserve more discipline than that.

**Three research findings drove this plugin's design:**

1. **ThoughtWorks Tech Radar Vol 33** introduced the technique *"Anchoring coding agents to a reference application"* — point AI agents at a known-good reference repo so their output stays inside the paved road. Greenfield's entire reason to exist is to produce that reference application before anyone writes a feature.

2. **AgentCoder (91.5% HumanEval, half the tokens of MetaGPT)** showed that minimal multi-agent designs beat sprawling ones for code generation, but the win came from **epistemic separation of test design from code generation**, not from adding more roles. So Greenfield uses minimal agents *during implementation* (handoff to Rival) but earns more agents *during planning* because architectural decisions compound.

3. **Anthropic's own multi-agent research post** explicitly warned: *"Teams have invested months building elaborate multi-agent architectures only to discover that improved prompting on a single agent achieved equivalent results."* Greenfield takes that warning seriously — the 9 agents only run *once per project*, not per feature. Per-feature work goes to the lean Rival loop.

---

## Core principle: scaffold as anchor

The output of `/greenfield:scaffold` is not a finished application. It's a **reference application** — a paved road every future coding agent on the project anchors against. The reference app has:

- The right tech stack (locked by ADRs, justified by research)
- Working IaC (Bicep + AVM for Azure, swappable for other clouds)
- CI/CD wired with OIDC federation (no long-lived secrets, ever)
- OpenTelemetry instrumentation from commit zero
- Health endpoints (`/health`, `/ready`, `/metrics`)
- Structured logging with correlation IDs
- The compliance baseline you actually need (PIPEDA, Law 25, GDPR, SOC 2 — whichever apply)
- Audit log retention set to 365+ days (the default 30 is a PIPEDA trap)
- DSR endpoint stubs (`/me/data`, `/me/export`, etc.)
- Design system (shadcn/Radix/Material/Cupertino — picked by the design-director from your mood board) with concrete tokens in `tailwind.config.ts`
- Component library install list ready to run
- Test scaffolding with the right pyramid shape for your stack
- Runbooks for deploy, rollback, incident, oncall
- ADRs explaining every choice — anchored to the research that produced them

When you (or a feature-level coding agent) build a feature on top of this, the paved road keeps you inside best practices automatically. You don't have to re-derive observability or compliance for every new endpoint — they're already wired.

---

## Installation

Greenfield is a Claude Code plugin distributed via a marketplace.

### Option 1: Add directly from GitHub

```bash
# Inside Claude Code, add the marketplace
/plugin marketplace add muhammadut/greenfield_plugin

# Install the plugin
/plugin install greenfield@greenfield-plugin
```

### Option 2: Local install for development

```bash
# Clone the repo
git clone https://github.com/muhammadut/greenfield_plugin.git
cd greenfield_plugin

# Add the local marketplace
# (in Claude Code)
/plugin marketplace add /absolute/path/to/greenfield_plugin
/plugin install greenfield@greenfield-plugin
```

### Verify installation

```bash
/plugin list
# greenfield should appear with version 0.1.0
```

You should now see 6 slash commands:
- `/greenfield:init`
- `/greenfield:research`
- `/greenfield:architect`
- `/greenfield:review`
- `/greenfield:scaffold`
- `/greenfield:status`

And 9 agents addressable by name from the Task tool.

---

## Quick start

A complete walkthrough of taking an empty directory to a planned greenfield project, in ~30 minutes.

```bash
# 1. Make an empty directory
mkdir my-new-project && cd my-new-project

# 2. Open Claude Code in this directory
claude
```

Then in Claude Code:

```
/greenfield:init
```

Claude will interview you. Answer one question at a time:

1. What are you building? (1-2 sentences)
2. Who is the user?
3. What's the core problem and what does "done" look like in 30 days?
4. Stack preference (or "help me decide")
5. **Visual direction** — paste URLs, drop screenshots into `.greenfield/assets/`, or describe the vibe
6. Compliance requirements (PIPEDA, Law 25, GDPR, SOC 2, none)
7. Timeline + team size
8. Top 5-10 MVP features in plain English

Your answers are written to `.greenfield/context.md`.

```
/greenfield:research
```

9 specialist agents run in parallel, each writing their report to `.greenfield/research/`. Takes ~5-10 minutes. You'll see completion notifications.

```
/greenfield:architect
```

The system-design-architect reads all 9 reports and reconciles them into:
- `.greenfield/plan/architecture.md` — unified plan
- `.greenfield/plan/adrs/` — committable ADRs
- `.greenfield/plan/c4/` — C4 diagrams in Structurizr DSL
- `.greenfield/plan/module-manifest.yaml` — module list for scaffolding
- `.greenfield/plan/open-questions.md` — anything that needs your decision

Resolve any open questions, then:

```
/greenfield:review
```

Security and QA specialists do an adversarial review. Output:
- `.greenfield/review/security-review.md`
- `.greenfield/review/test-review.md`

If there are blockers, you'll be asked to resolve them and re-run `/greenfield:architect`. If only gaps, you can decide to fix or scaffold-and-fix-later.

```
/greenfield:scaffold --template azure-saas-container-apps
```

Hydrates the template with your plan. Writes the source tree, infra, CI/CD, compliance scaffolding, and design tokens to your project directory.

```bash
# Review what was created
git status
git diff

# Install dependencies (commands printed by scaffold)
pnpm install
pnpm exec playwright install

# First commit
git init && git add . && git commit -m "chore: greenfield scaffold v0.1"

# Bootstrap infra (one-time)
cd infra && az deployment sub create -l canadacentral -f main.bicep -p envs/dev.bicepparam
```

Project is now brownfield. Hand off to your feature workflow.

```
/greenfield:status
```

Run any time to see where you are in the workflow.

---

## The workflow in detail

```
                    ┌──────────────┐
                    │  empty dir   │
                    └──────┬───────┘
                           │
                           │ /greenfield:init
                           ▼
                    ┌──────────────┐
                    │   INIT       │──── .greenfield/context.md
                    │              │──── .greenfield/assets/ (mood board)
                    └──────┬───────┘
                           │
                           │ /greenfield:research
                           ▼
                    ┌──────────────┐
                    │  RESEARCH    │──── 9 parallel agents
                    │              │──── .greenfield/research/*.md
                    └──────┬───────┘
                           │
                           │ /greenfield:architect
                           ▼
                    ┌──────────────┐
                    │  ARCHITECT   │──── reconciliation by system-design-architect
                    │              │──── .greenfield/plan/architecture.md
                    │              │──── .greenfield/plan/adrs/
                    │              │──── .greenfield/plan/c4/
                    │              │──── .greenfield/plan/module-manifest.yaml
                    └──────┬───────┘
                           │
                           │ /greenfield:review
                           ▼
                    ┌──────────────┐
                    │   REVIEW     │──── adversarial security + qa
                    │              │──── .greenfield/review/security-review.md
                    │              │──── .greenfield/review/test-review.md
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
           BLOCKED      GAPS OK      CLEAN
              │            │            │
              ▼            ▼            ▼
         revise plan,  decide:     /greenfield:scaffold
         re-run        fix or       --template <name>
         architect     scaffold          │
                       anyway            ▼
                                  ┌──────────────┐
                                  │  SCAFFOLD    │──── hydrate template
                                  │              │──── source tree
                                  │              │──── infra/ + .github/
                                  │              │──── compliance/
                                  │              │──── .greenfield/SCAFFOLD_COMPLETE
                                  └──────┬───────┘
                                         │
                                         │ user reviews, commits
                                         ▼
                                  ┌──────────────┐
                                  │  BROWNFIELD  │──── hand off to Rival or
                                  │              │     your feature workflow
                                  └──────────────┘
```

### Time budget

| Phase | Command | Agents | Wall time |
|---|---|---|---|
| Init | `/greenfield:init` | 0 (interview only) | 5-10 min (your input time) |
| Research | `/greenfield:research` | 9 parallel | 5-10 min |
| Architect | `/greenfield:architect` | 1 (reconciliation) | 2-5 min |
| Review | `/greenfield:review` | 2 parallel | 2-5 min |
| Scaffold | `/greenfield:scaffold` | 0 (template hydration) | 1-3 min |
| **Total** | | | **~20-35 min compute + your think time** |

A real first run on a non-trivial project will be 1-2 hours wall-clock end-to-end including your own review time between phases.

### Reentrancy

Every phase is idempotent on its inputs. If you crash mid-research, re-running `/greenfield:research` will only re-run the agents whose outputs are missing. Same for architect and review. The only irreversible step is scaffold — and even that aborts safely if the target directory has conflicting files.

You can exit and resume at any phase. `/greenfield:status` always shows where you are.

---

## The 9 agents — full reference

Each agent is a specialist subagent with its own system prompt, tool list, and output contract. Agents never talk to each other directly — all communication is via files in `.greenfield/`.

### 1. product-strategist

| | |
|---|---|
| **Purpose** | Pin down WHAT is being built and WHY before anyone argues about HOW |
| **Inputs** | `.greenfield/context.md`, `.greenfield/assets/` |
| **Output** | `.greenfield/research/product-strategy.md` |
| **Phase** | research |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces these sections:
1. **Press Release** (Amazon "Working Backwards" format) — fictional launch-day press release
2. **Problem Statement** — who is in pain, what they try to do, cost of pain, alternatives
3. **MVP Scope** — 5-10 in-scope, 5-10 out-of-scope, success metric
4. **User Stories** — 8-15 stories with acceptance criteria, prioritized P0/P1/P2
5. **Non-Goals and Anti-Features** — what this product will never have
6. **Open Questions for Downstream Agents**

**Constraint discipline:** No architecture in the output. No feature bloat. Concrete personas, not "small business owner."

### 2. system-design-architect

| | |
|---|---|
| **Purpose** | Architectural shape — services, contracts, data flow, decisions; also reconciles all 9 in the architect phase |
| **Inputs (research)** | `.greenfield/context.md`, `.greenfield/research/product-strategy.md` |
| **Inputs (reconcile)** | All files in `.greenfield/research/` |
| **Output (research)** | `.greenfield/research/system-design.md` |
| **Output (reconcile)** | `.greenfield/plan/architecture.md`, `adrs/*.md`, `c4/*.dsl`, `module-manifest.yaml`, `open-questions.md` |
| **Phase** | research + architect (load-bearing) |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces:
1. **C4 Level 1** (System Context) — Structurizr DSL or ASCII
2. **C4 Level 2** (Container Diagram) — services, responsibilities, tech (TENTATIVE)
3. **Domain Model** — DDD light, aggregates, entities
4. **API Contract Sketch** — top 10-20 endpoints
5. **ADRs** — Nygard format, 5-10 records
6. **Module/Service Manifest** — YAML for the scaffold phase
7. **Risks & Unknowns**

**Reconciliation protocol:** When specialists disagree, surface the trade-off explicitly in the relevant ADR and pick with reasoning. Never silently resolve.

### 3. ux-researcher

| | |
|---|---|
| **Purpose** | How users move through the product — flows, IA, accessibility |
| **Inputs** | `.greenfield/context.md`, `.greenfield/research/product-strategy.md` |
| **Output** | `.greenfield/research/ux-plan.md` |
| **Phase** | research |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces:
1. **Primary User Journey** — golden path with screens, intents, success/failure states
2. **Secondary Journeys** — support, admin, offboarding, recovery
3. **Information Architecture** — sitemap with auth-gated marking
4. **Core Screens (skeleton)** — purpose, content blocks, primary action, all states
5. **Accessibility Plan** — WCAG 2.2 AA minimum, axe-core in CI
6. **i18n / l10n** — flagged if context.md mentions bilingual
7. **Empty / Error / Loading States** — explicit enumeration
8. **Analytics Hooks** — events to instrument

**Constraint discipline:** No visuals (Design Director owns those). States are as important as screens. Concrete error messages, not "show error."

### 4. design-director

| | |
|---|---|
| **Purpose** | Translate mood board into concrete design system + tokens |
| **Inputs** | `.greenfield/context.md`, `.greenfield/assets/` (screenshots/mockups/Nano Banana exports), referenced URLs via WebFetch |
| **Output** | `.greenfield/research/design-direction.md` |
| **Phase** | research |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces:
1. **Visual Language Summary** — one paragraph distillation
2. **Design System Recommendation** — picks one of: shadcn/ui + Radix, Mantine, Material (MUI), Chakra UI v3, Cupertino, Custom Tailwind + Headless UI
3. **Design Tokens (concrete)** — colors with hex, typography scale, spacing, radii, shadows, motion durations + easing
4. **Component Library Install List** — exact `pnpm add` commands and `tailwind.config.ts` stub
5. **Reference Inspirations** — what to take from each source, what to leave
6. **Anti-Patterns to Avoid** — based on what the references *don't* do
7. **Accessibility Floor** — WCAG AA contrast pairs
8. **Open Questions** — dark mode? logo? icon style?

**Constraint discipline:** Always hex values, never "calming blue." Always WCAG AA. Cite references specifically. Every recommendation maps to an installable package or Tailwind config line.

### 5. security-privacy-architect

| | |
|---|---|
| **Purpose** | Threat model, data classification, compliance baseline; also runs in review phase |
| **Inputs (research)** | `.greenfield/context.md`, `product-strategy.md`, optional `system-design.md` |
| **Inputs (review)** | `.greenfield/plan/*` |
| **Output (research)** | `.greenfield/research/security-privacy.md` |
| **Output (review)** | `.greenfield/review/security-review.md` |
| **Phase** | research + review |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces (research phase):
1. **Data Classification** — PII / Sensitive PII / Business confidential / Public, with retention, residency, encryption
2. **Threat Model (STRIDE lite)** — Spoofing/Tampering/Repudiation/Info Disclosure/DoS/Elevation
3. **Authentication & Authorization** — provider rec, RBAC/ABAC/ReBAC, session, MFA
4. **Compliance Baseline** — PIPEDA, Quebec Law 25, GDPR, SOC 2, HIPAA (whichever apply)
5. **Secrets & Key Management** — Key Vault, managed identity, rotation, envelope encryption
6. **Supply Chain Security** — SBOM (CycloneDX), SLSA L2+, dep scanning, SAST, container signing
7. **Day-1 DSR Endpoints** — exact endpoint specs that MUST be in the scaffold
8. **Risk Register** — top 10 H/M/L

**Constraint discipline:** Concrete controls, not checklists. Cite regulatory text. Day-1 DSR endpoints non-negotiable. Flag project-killing requirements.

### 6. data-architect

| | |
|---|---|
| **Purpose** | Logical data model, event taxonomy, PII map, database choice |
| **Inputs** | `.greenfield/context.md`, `product-strategy.md`, optional `system-design.md`, optional `security-privacy.md` |
| **Output** | `.greenfield/research/data-architecture.md` |
| **Phase** | research |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces:
1. **Logical Data Model** — entities, attributes, relationships, PII flags
2. **Physical Schema Recommendations** — store choice per entity, partition keys, indexes
3. **Database Technology Choice** — Postgres / MySQL / Cosmos / DynamoDB / SQLite + Litestream
4. **Event Taxonomy** — CloudEvents-spec events with versioning (.v1)
5. **Event Bus / Messaging** — broker choice, semantics, DLQ, ordering
6. **PII Map (detailed)** — every PII field → entity → store → retention → erasure strategy
7. **Analytics & Warehouse** — yes/no for v1, where the seam goes
8. **Data Migration Plan** — n/a for pure greenfield
9. **Seed Data & Fixtures**
10. **Open Questions**

**Constraint discipline:** Logical before physical. Events versioned from day 1. PII map is exhaustive. Cost-aware (Cosmos is usually wrong for small SaaS).

### 7. cloud-architect

| | |
|---|---|
| **Purpose** | Cloud services, IaC pattern, landing zone, cost estimate |
| **Inputs** | `.greenfield/context.md`, `product-strategy.md`, optional `system-design.md`, `data-architecture.md`, `security-privacy.md` |
| **Output** | `.greenfield/research/cloud-architecture.md` |
| **Phase** | research |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces:
1. **Cloud Provider Decision** — Azure default for v1, swap as needed
2. **Region Strategy** — primary, DR, residency
3. **Reference Architecture** — service-by-service table with tier and purpose
4. **Monthly Cost Estimate** — line items + total
5. **Landing Zone Decision** — full ALZ vs lightweight
6. **IaC Recommendation** — Bicep + AVM by default, Terraform/Pulumi as alternatives
7. **Networking Topology** — public vs private endpoints, VNET, NAT, DDoS
8. **Well-Architected Checklist** — 3 high-leverage checks per pillar
9. **Auth Recommendation** — Entra External ID for Canadian SaaS, alternatives explained
10. **Open Questions**

**Constraint discipline:** No AWS/GCP if user picked Azure. Use AVM, not legacy CAF. Entra External ID, not B2C. No multi-region for MVPs.

### 8. platform-sre

| | |
|---|---|
| **Purpose** | CI/CD, observability spine, SLOs, deploy topology, dev environment |
| **Inputs** | `.greenfield/context.md`, `product-strategy.md`, `system-design.md`, `cloud-architecture.md`, `security-privacy.md` |
| **Output** | `.greenfield/research/platform-sre.md` |
| **Phase** | research |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces:
1. **Development Environment** — devcontainer, one-command boot
2. **CI/CD Pipeline** — lint, typecheck, test, build, security scan, sign, deploy, rollback
3. **Observability Spine** — OTel SDK, structured logs, USE+RED metrics, health endpoints, 365+ day Log Analytics
4. **SLOs** — 3-5 with error budgets and burn rate alerts
5. **Deployment Topology** — environments, parity, strategy, config, feature flags (OpenFeature)
6. **Runbooks** — deploy, rollback, incident, oncall skeletons
7. **Backup & DR** — RPO/RTO, region failover plan
8. **Secrets & Credentials Lifecycle** — OIDC federation, no long-lived secrets
9. **Cost Observability** — budget alerts, tagged resources
10. **Open Questions**

**Constraint discipline:** Day-1 observability non-negotiable. No long-lived secrets. 365+ day log retention. Feature flags via OpenFeature. SLOs in repo, not wiki. One command to dev.

### 9. qa-test-strategist

| | |
|---|---|
| **Purpose** | Test strategy, definition-of-done, CI gates; also runs in review phase |
| **Inputs (research)** | `.greenfield/context.md`, `product-strategy.md`, optional `system-design.md`, `data-architecture.md` |
| **Inputs (review)** | `.greenfield/plan/*` and `product-strategy.md` |
| **Output (research)** | `.greenfield/research/test-strategy.md` |
| **Output (review)** | `.greenfield/review/test-review.md` |
| **Phase** | research + review |
| **Tools** | Read, Write, Grep, Glob, WebFetch, WebSearch |

Produces (research phase):
1. **Definition of Done** — universal DoD checklist
2. **Test Pyramid Shape** — 70/20/10 with stack-specific guidance
3. **Test Frameworks per Stack** — pytest, vitest, playwright, etc.
4. **Test Data Strategy** — fixtures, factories, no PII, testcontainers
5. **Acceptance Criteria Template** — Given/When/Then with examples
6. **CI Gates** — what blocks vs warns
7. **Production Verification** — smoke checks, rollback triggers
8. **Flake Policy** — quarantine then delete
9. **Review Phase Responsibilities** — questions to ask in review
10. **Open Questions**

**Constraint discipline:** Test plan in repo, not wiki. No mocks of own code. Real DBs via testcontainers. Coverage as health metric, not goal. Quarantine flaky, don't delete.

---

## The 6 commands — full reference

### `/greenfield:init`

**Purpose:** Interview you and capture project vision into `.greenfield/context.md`.

**Prerequisites:** None. Works in empty directories.

**Behavior:**
1. Checks if `.greenfield/context.md` exists. If yes, asks: resume / edit / start over.
2. Creates `.greenfield/{context.md, assets/, research/, plan/}` directory tree.
3. Creates `.greenfield/assets/README.md` with mood board drop instructions.
4. Asks 8 questions ONE AT A TIME, waiting for each answer.
5. Writes `.greenfield/context.md` with structured headers that downstream agents parse.
6. Confirms with you before exiting and tells you the next command.

**Outputs:**
- `.greenfield/context.md`
- `.greenfield/assets/README.md`
- Empty `.greenfield/{research,plan}/` directories

**Constraints:** One question at a time. Saves incrementally. No code. Visual references first-class.

### `/greenfield:research`

**Purpose:** Dispatch all 9 specialist agents in parallel.

**Prerequisites:** `.greenfield/context.md` exists.

**Behavior:**
1. Verifies prerequisites.
2. Dispatches all 9 agents in a SINGLE message (parallel) using the Task tool.
3. Each agent reads `.greenfield/context.md` and writes its specialist file.
4. Waits for completion notifications (no polling).
5. Verifies all 9 output files exist.
6. Surfaces any conflicts between agent outputs from their return summaries.
7. Tells you the next command.

**Outputs:**
- `.greenfield/research/product-strategy.md`
- `.greenfield/research/system-design.md`
- `.greenfield/research/ux-plan.md`
- `.greenfield/research/design-direction.md`
- `.greenfield/research/security-privacy.md`
- `.greenfield/research/data-architecture.md`
- `.greenfield/research/cloud-architecture.md`
- `.greenfield/research/platform-sre.md`
- `.greenfield/research/test-strategy.md`

**Constraints:** Parallel dispatch (one message, 9 tool calls). Idempotent. Reports conflicts.

### `/greenfield:architect`

**Purpose:** Reconcile the 9 research reports into a unified plan with ADRs.

**Prerequisites:** All 9 files in `.greenfield/research/` exist.

**Behavior:**
1. Verifies all 9 research files exist.
2. Dispatches `system-design-architect` in **reconciliation mode**.
3. Architect reads every file in `.greenfield/research/` and merges them.
4. Writes the canonical plan files.
5. Surfaces any contradictions explicitly in ADRs.
6. Lists open questions that block scaffolding.

**Outputs:**
- `.greenfield/plan/architecture.md` — unified plan
- `.greenfield/plan/adrs/ADR-0001-*.md`, `ADR-0002-*.md`, ... — one file per decision, Nygard format
- `.greenfield/plan/c4/context.dsl` — C4 L1 in Structurizr DSL
- `.greenfield/plan/c4/containers.dsl` — C4 L2
- `.greenfield/plan/module-manifest.yaml` — module list for scaffolding
- `.greenfield/plan/open-questions.md` — needs human decision

**Constraints:** Doesn't write the plan itself — dispatches the agent. Surfaces conflicts. Open questions block scaffold.

### `/greenfield:review`

**Purpose:** Adversarial review by security and QA specialists.

**Prerequisites:** `.greenfield/plan/architecture.md` and `.greenfield/plan/adrs/` exist.

**Behavior:**
1. Verifies prerequisites.
2. Dispatches `security-privacy-architect` AND `qa-test-strategist` in parallel (one message, two tool calls), both in review mode.
3. Each reads `.greenfield/plan/*` and writes a review file.
4. Reads both review files and presents a consolidated summary.
5. Verdict: `READY_TO_SCAFFOLD` / `BLOCKED_ON: <blockers>` / `REVISIONS_NEEDED: <gaps>`.
6. Tells you next steps depending on verdict.

**Outputs:**
- `.greenfield/review/security-review.md`
- `.greenfield/review/test-review.md`

**Constraints:** Adversarial, not editorial. Cites specifically. Blockers block scaffold. Gaps are negotiable.

### `/greenfield:scaffold --template <name>`

**Purpose:** Generate the reference application from a template, hydrated with the plan.

**Arguments:**
- `--template <name>` (optional) — template directory name. Default: `azure-saas-container-apps`.

**Prerequisites:**
- `.greenfield/plan/architecture.md` and `.greenfield/plan/module-manifest.yaml` exist.
- Template directory exists at `templates/<name>/`.
- No unresolved BLOCKERS in `.greenfield/review/security-review.md`. (Gaps OK with warning.)
- Target directory has no conflicting files.

**Behavior:**
1. Verifies prerequisites; refuses to proceed on blockers.
2. Reads plan files into working memory.
3. Copies template tree to current working directory (literal files outside `.hydrate/`).
4. Hydrates `.hydrate/` template files with plan values (`{{project_name}}`, `{{tech_stack}}`, `{{region_primary}}`, `{{design_tokens}}`, `{{module_list}}`, etc).
5. Wires design tokens from `design-direction.md` into `tailwind.config.ts` and `app/globals.css`.
6. Wires IaC parameters from `cloud-architecture.md` into Bicep param files.
7. Wires compliance scaffolding (TIA template, PIA template, consent log, DSR endpoints, 365+ day log retention).
8. Writes `.greenfield/SCAFFOLD_COMPLETE` marker.
9. Prints exact dependency-install commands and next steps for the user.

**Outputs:**
- Source tree, infra/, .github/workflows/, apps/, compliance/, docs/, .devcontainer/ in target directory
- `.greenfield/SCAFFOLD_COMPLETE` marker

**Constraints:** No scaffold if blockers. Doesn't run package installs. Doesn't init git. Idempotent on clean target. Hands off cleanly to brownfield workflow.

### `/greenfield:status`

**Purpose:** Report current state of the workflow.

**Prerequisites:** None.

**Behavior:**
1. Walks through phases 0-4 and checks for expected files.
2. Counts ADRs, C4 diagrams, blockers, gaps.
3. Prints a compact status table.
4. Surfaces the next command.

**Outputs:** None (read-only).

**Constraints:** Read-only. Concise. Always actionable. Safe on empty directories.

---

## State directory layout

Greenfield writes everything to a `.greenfield/` directory inside your target project. Plugin code is separate; only state lives in your project.

```
.greenfield/
├── context.md                 # init output — project vision
├── assets/                    # mood board (you drop files here)
│   ├── README.md
│   ├── inspiration-1.png      # screenshots
│   ├── linear-sidebar.jpg
│   └── mockup-from-nano.png   # Nano Banana exports
├── research/                  # research output (9 files)
│   ├── product-strategy.md
│   ├── system-design.md
│   ├── ux-plan.md
│   ├── design-direction.md
│   ├── security-privacy.md
│   ├── data-architecture.md
│   ├── cloud-architecture.md
│   ├── platform-sre.md
│   └── test-strategy.md
├── plan/                      # architect output
│   ├── architecture.md
│   ├── adrs/
│   │   ├── ADR-0001-cloud-provider.md
│   │   ├── ADR-0002-database.md
│   │   ├── ADR-0003-auth.md
│   │   └── ...
│   ├── c4/
│   │   ├── context.dsl        # Structurizr DSL
│   │   └── containers.dsl
│   ├── module-manifest.yaml
│   └── open-questions.md
├── review/                    # review output
│   ├── security-review.md
│   └── test-review.md
└── SCAFFOLD_COMPLETE          # marker file (after scaffold runs)
```

**Why files, not in-memory state:**
- Context isolation — agents have minimal context; no token burn on each other's deliberations
- Debuggability — you can inspect what each agent actually produced
- Resumability — partial workflows resumable by re-running specific commands
- Git-commitable — the plan is the repo's source of truth

**Should `.greenfield/` be committed?**
Yes — the research, plan, ADRs, and reviews are valuable history. Add it to your repo. The only thing you might `.gitignore` is large binary mood board assets (commit small ones, ignore large screenshots).

---

## Templates

Templates are the paved-road reference applications that `/greenfield:scaffold` hydrates with the plan.

### Template directory structure

```
templates/
└── <template-name>/
    ├── README.md              # what this template is, when to use it
    ├── .hydrate/
    │   └── README.md          # substitution variables this template uses
    ├── infra/                 # IaC
    ├── .github/workflows/     # CI/CD
    ├── apps/                  # source tree
    ├── compliance/            # TIA, PIA, DSR, consent log
    ├── docs/runbooks/         # deploy, rollback, incident, oncall
    ├── .devcontainer/         # dev environment
    └── ...
```

Files outside `.hydrate/` are copied literally. Files inside `.hydrate/` use `{{variable}}` substitution markers, hydrated by the scaffold skill.

### Hydration variables (standard)

- `{{project_name}}` — from context.md
- `{{tech_stack}}` — from architecture.md
- `{{region_primary}}`, `{{region_dr}}` — from cloud-architecture.md
- `{{design_tokens}}` — from design-direction.md (inlined into tailwind.config.ts)
- `{{component_library}}` — from design-direction.md install list
- `{{module_list}}` — from module-manifest.yaml
- `{{compliance_list}}` — from context.md and security-privacy.md

Each template can define additional variables in its own `.hydrate/README.md`.

### Available templates (v0.1.0)

| Name | Status | Description |
|---|---|---|
| `azure-saas-container-apps` | **stub** | Azure Container Apps SaaS with Bicep + AVM, GitHub Actions OIDC, Postgres Flexible Server, App Insights + OTel, Entra External ID, PIPEDA/Law 25 compliance scaffolding, shadcn/ui design system |

The template is a stub by design — the workflow needs to be validated on a real project before investing in the template. Do `/greenfield:init`, `/greenfield:research`, `/greenfield:architect`, `/greenfield:review` end-to-end first; then build the template based on what the actual outputs need.

### Adding a new template

Minimum requirements for a template to be production-ready:
- [ ] IaC for all infra
- [ ] CI/CD with OIDC federation — no long-lived secrets
- [ ] OpenTelemetry instrumentation from commit zero
- [ ] Structured logging with correlation IDs
- [ ] Health endpoints (`/health`, `/ready`, `/metrics`)
- [ ] SBOM + SAST + container scanning in CI
- [ ] DSR endpoint stubs (`/me/data`, `/me/export`, etc.)
- [ ] Audit log retention ≥ 365 days
- [ ] Design system integration points (`tailwind.config.ts`, theme tokens)
- [ ] README with bootstrap instructions
- [ ] `.hydrate/README.md` with substitution variable list

---

## Architecture decisions baked in

These decisions are encoded in the agent prompts. Override them with explicit guidance in your `context.md` if needed.

| Decision | Default | Reason |
|---|---|---|
| **Cloud (Azure projects)** | Microsoft Azure | First user is Canadian, compliance-driven |
| **IaC (Azure)** | Bicep + Azure Verified Modules | CAF Terraform module being archived August 2026 |
| **Compute (Azure)** | Azure Container Apps | Cheaper and simpler than App Service / AKS for new SaaS |
| **Customer auth (Azure)** | Microsoft Entra External ID | Free <50k MAU; Azure AD B2C stopped accepting new customers May 1, 2025 |
| **Workforce auth** | Microsoft Entra ID | Native to Azure |
| **Database (default)** | Azure Database for PostgreSQL Flexible Server | Boring, proven, ACID, rich ecosystem |
| **CI/CD** | GitHub Actions with OIDC federation | No long-lived secrets ever |
| **Container signing** | Sigstore cosign | SLSA L2 baseline |
| **SBOM format** | CycloneDX or SPDX | Industry standards |
| **Observability** | OpenTelemetry → App Insights | Vendor-neutral, future-proof |
| **Log retention** | 365+ days minimum | Default 30 is a PIPEDA audit trap |
| **Feature flags** | OpenFeature | Vendor-neutral; swap providers without code change |
| **Test framework strategy** | Real DBs via testcontainers, no mocks of own code | AgentCoder-style epistemic separation |
| **Coverage gate** | 80% diff coverage, not 80% project | Health metric, not goal |
| **Diagrams** | Structurizr DSL or ASCII (text) | Git-committable, reviewable |
| **ADR format** | Nygard | Status / Context / Decision / Consequences |
| **Compliance defaults** | PIPEDA + Quebec Law 25 baseline | Canadian-first; add GDPR/SOC 2/HIPAA per context |

---

## How Greenfield differs from other AI coding tools

| Tool | What it does | What Greenfield does differently |
|---|---|---|
| **Cursor / Copilot / Cline** | Inline code suggestions and chat-driven edits | Greenfield is plan-first. It runs *before* you write any code. Cursor takes over after `/greenfield:scaffold`. |
| **Devin / Cognition** | Autonomous SWE for tasks on existing repos | Greenfield is greenfield-only — it produces the reference app Devin would otherwise have to invent. |
| **AutoGen / CrewAI / MetaGPT** | Multi-agent frameworks for code generation | Greenfield uses multi-agent only in planning, where breadth pays off. Implementation handoff goes to a minimal loop (Rival). MetaGPT-style "software company" roles are explicitly rejected based on AgentCoder/MAST evidence. |
| **Yeoman / cookiecutter / create-react-app** | Static project scaffolders | Greenfield is *dynamic* scaffolding — the template is hydrated with research-driven decisions, not static defaults. Compliance, observability, design system are all driven by your project's actual needs. |
| **Backstage software templates** | Internal developer platform templates at scale | Closer cousin. Greenfield can be thought of as Backstage Software Templates with an AI research front-end. The scaffold output could literally feed Backstage. |
| **Rival (sibling plugin)** | Per-feature plan/execute/verify loop | Rival operates *after* Greenfield. Greenfield produces the reference app; Rival anchors all feature work against it. |

---

## Troubleshooting

### "Missing research files" when running `/greenfield:architect`

Cause: One or more agents in `/greenfield:research` failed or you didn't run it.
Fix: Run `/greenfield:status` to see which files are missing. Re-run `/greenfield:research` (it's idempotent — only missing files get re-generated... or run with `--force` to redo all).

### Agent output is empty or thin

Cause: Your `context.md` is too vague for the agent to work with.
Fix: Re-run `/greenfield:init` and push for more specifics in the weak section. Don't say "small business" — say "plumber in Ontario running a 2-person shop with a flip phone."

### Architect finds unresolvable contradictions

Cause: Two specialists made contradictory assumptions about the stack or scale.
Fix: Architect surfaces the conflict in `.greenfield/plan/open-questions.md`. Resolve each open question, then re-run `/greenfield:architect`.

### Review finds blockers

Cause: Plan has unshippable issues — e.g., missing DSR endpoints, long-lived secrets in the deploy plan, PII in a non-compliant region.
Fix: The review file lists exact blockers with citations. Revise the plan (often by editing ADRs or re-running architect with updated context), then re-run `/greenfield:review`. Do not scaffold with unresolved blockers.

### Scaffold target has conflicting files

Cause: Target directory is not clean.
Fix: Scaffold aborts safely. Clean the conflicting files (or move to a fresh directory) and retry.

### "Template directory not found"

Cause: You specified a `--template <name>` that doesn't exist.
Fix: List available templates with `ls templates/` in the plugin directory, or run `/greenfield:scaffold` without `--template` to use the default.

### `azure-saas-container-apps` template is empty

Expected for v0.1.0. The template is a stub. Either build it out, or use the workflow phases (init/research/architect/review) without scaffolding to get the planning artifacts as a standalone deliverable.

### Plugin doesn't show up after install

Cause: Marketplace not added or plugin not enabled.
Fix:
```
/plugin marketplace list
/plugin list
/plugin install greenfield@greenfield-plugin
```

---

## FAQ

**Q: Is Greenfield only for Azure?**
A: No. Azure is the default in agent prompts because the first user is on Azure, but the workflow is cloud-agnostic. Override in `context.md` ("we're on AWS") and the cloud-architect agent will produce AWS recommendations. Templates are also swappable — write your own `templates/aws-eks-saas/` for an AWS Kubernetes target.

**Q: Can I run individual agents without the full workflow?**
A: Yes. Each agent is addressable by name from the Task tool. You can dispatch just `design-director` against an existing `context.md` to redo your design tokens, for example.

**Q: How do I update the agent prompts to match my consulting client's preferences?**
A: Fork the plugin, edit the `agents/*.md` files, and reinstall. Each agent file has a clear constraint section near the bottom that's easy to modify.

**Q: Does Greenfield write code?**
A: Greenfield writes scaffold code (template hydration), but does NOT write feature code. Feature code is the job of your post-scaffold workflow (Rival or any feature-level loop). This separation is intentional — research shows multi-agent code generation underperforms minimal designs.

**Q: How does this work with the Rival plugin?**
A: Greenfield runs once. Rival runs per feature. The scaffold produced by Greenfield is the reference app Rival anchors against. Together they form a complete project lifecycle: planning + scaffolding (Greenfield) → feature loop (Rival).

**Q: Why 9 agents, not 6 or 12?**
A: Research stream 2 (multi-agent benchmarks like AgentCoder, MAST 2025) argues for minimalism. Research stream 4 (human team composition) argues for breadth. The resolution: planning earns breadth (architectural decisions compound 10-30x), implementation earns minimalism (token burn dominates). The 9 only run *once per project*, not per feature.

**Q: Can I add my own agent to the roster?**
A: Yes. Add an `agents/<name>.md` file with the standard frontmatter (name, description, tools), update `skills/greenfield-research/SKILL.md` to dispatch it, and update `docs/agent-roster.md`. The plugin is designed for editing.

**Q: What if I don't want all 9 agents?**
A: Edit `skills/greenfield-research/SKILL.md` to remove the agents you don't need, and edit `skills/greenfield-architect/SKILL.md` so the reconciler doesn't expect their files. For a tiny internal tool, you might cut to 5 (drop product-strategist, ux-researcher, design-director if there's no UI).

**Q: Does Greenfield work for non-Azure projects?**
A: Yes — see "Is Greenfield only for Azure?" above. The architecture decisions table lists Azure as defaults but every default is overridable.

**Q: Can I use Greenfield on an existing brownfield project?**
A: Not really. Greenfield assumes an empty (or near-empty) target directory. For brownfield work, use Rival or another feature-level workflow.

**Q: What's the minimum tech stack I need installed?**
A: Just Claude Code. The plugin itself is pure markdown — no runtime dependencies. The scaffold step will print the dependencies you need to install for the actual reference app (e.g., `pnpm`, `az`, `bicep`).

**Q: Where do I find the research that drove the design?**
A: Inside the parent project (`elocal_clone/greenfield-research/`) there are 4 reports with full citations. Also see `docs/design.md` for the synthesis.

---

## Roadmap

### v0.2.0
- Build out `azure-saas-container-apps` template (Bicep + AVM, GitHub Actions OIDC, Container Apps + Postgres + Entra External ID, OTel, compliance scaffolding)
- Add `/greenfield:feature "..."` for capturing post-init feature requests
- Cruft-style template version drift management

### v0.3.0
- Second template: `aws-ecs-fargate-saas` (AWS equivalent for consulting clients)
- Third template: `vercel-nextjs-prisma-saas` (lightweight startup option)
- A/B test infrastructure: 9-agent vs 1-agent baseline per MAST findings

### v0.4.0
- IDE-aware mood board ingestion (Figma plugin, Tldraw integration)
- Cost estimation refinement using real billing APIs
- Compliance presets: HIPAA, FedRAMP, SOC 2 Type II

### v1.0.0
- All templates production-ready
- Compliance presets cover the top 8 regimes
- Stable agent roster
- Documentation per agent that reflects 6 months of real project use

---

## Contributing & extending

### Add an agent

1. Create `agents/<your-agent>.md` with frontmatter:
   ```yaml
   ---
   name: your-agent
   description: Use when ...
   tools: Read, Write, Grep, Glob, WebFetch, WebSearch
   ---
   ```
2. Write the system prompt with: Inputs / Outputs / Expertise / Constraints sections.
3. Update `skills/greenfield-research/SKILL.md` to add a Task dispatch for it.
4. Update `docs/agent-roster.md`.
5. Test with a real project.

### Add a template

1. Create `templates/<your-template>/` with the structure shown in [Templates](#templates).
2. Identify what variables your template needs and document in `.hydrate/README.md`.
3. Make sure every minimum-requirement checkbox is covered.
4. Add it to the table in this README.
5. Test by running the full workflow and scaffolding into a throwaway directory.

### Modify agent constraints

Each agent has a `## Constraints` section near the bottom. Edit freely — these are the highest-leverage place to encode your consulting-firm preferences.

### Issues & PRs

Repo: https://github.com/muhammadut/greenfield_plugin

---

## License

MIT
