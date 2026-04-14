---
name: greenfield-scaffold
description: Generate the reference application from a template, hydrated with the reconciled plan. Produces first commit with IaC, CI/CD, observability, design tokens, compliance scaffolding baked in. Run after /greenfield:review passes.
user-invocable: true
---

# /greenfield:scaffold

You are the scaffold orchestrator. Your job is to take the reconciled plan in `.greenfield/plan/` and hydrate a template from `templates/<template-name>/` into the target project directory, producing a committable reference application.

## Prerequisites

1. Verify `.greenfield/plan/architecture.md` and `.greenfield/plan/module-manifest.yaml` exist. If not, tell user to run `/greenfield:architect` (and ideally `/greenfield:review`) first.

2. Verify the user specified a template. Default: `azure-saas-container-apps`. Syntax: `/greenfield:scaffold --template <name>`. If no template flag, use the default and tell the user which template you're using.

3. Verify the template directory exists at the plugin's `templates/<name>/` path. If not, list available templates from `templates/` and ask the user to pick.

4. Verify `.greenfield/review/` exists and check for blockers. If the security review has unresolved blockers, REFUSE to scaffold and tell the user to address them first. Gaps are OK but warn about them.

## Scaffold protocol

The scaffold is a file-tree copy + hydration operation:

### Step 1: Read the plan
Load these into working memory:
- `.greenfield/plan/module-manifest.yaml` — the list of modules/services
- `.greenfield/research/design-direction.md` — design tokens for Tailwind config
- `.greenfield/research/data-architecture.md` — schemas and events for migrations
- `.greenfield/research/cloud-architecture.md` — Azure service names and regions
- `.greenfield/research/security-privacy.md` — compliance checkpoints

### Step 2: Copy template to target
Copy every file from `templates/<name>/` to the current working directory, EXCEPT any files in a `.hydrate/` subdirectory (those are templates, not literals).

### Step 3: Hydrate templates
For each `.hydrate/` template file, substitute plan values:
- `{{project_name}}` ← from context.md "What we're building"
- `{{tech_stack}}` ← from locked tech stack in architecture.md
- `{{region_primary}}`, `{{region_dr}}` ← from cloud-architecture.md
- `{{design_tokens}}` ← from design-direction.md (inline into tailwind.config.ts)
- `{{module_list}}` ← from module-manifest.yaml
- (etc — each template defines its own substitution map in `.hydrate/README.md`)

### Step 4: Wire design tokens
Specifically: extract color + typography + spacing tokens from `design-direction.md` and write them into the scaffold's `tailwind.config.ts` and `app/globals.css`. Install the component library list from the design-direction.md install command.

### Step 5: Wire IaC parameters
Extract region, SKU tier, and naming from `cloud-architecture.md` and write them into the scaffold's `infra/envs/dev.bicepparam` and `infra/envs/prod.bicepparam`.

### Step 6: Wire compliance scaffolding
Ensure the scaffold includes (add if the template doesn't have them by default):
- `compliance/tia-template.md` — Transfer Impact Assessment template for Law 25
- `compliance/pia-template.md` — Privacy Impact Assessment template for PIPEDA
- `compliance/consent-log-schema.sql` — immutable consent log table
- `compliance/dsr-endpoints/` — stubs for `/me/data`, `/me/export`, `/me/data` (DELETE), `/me/consent`
- Log retention config set to 365+ days in Log Analytics module

### Step 7: Install dependencies
Do NOT run `npm install` / `pip install` automatically. Print the exact commands the user should run after they review the scaffold.

### Step 8: Initial commit readiness
Create a `.greenfield/SCAFFOLD_COMPLETE` marker file with timestamp and template name.

### Step 9: Report to user

Print:
```
Scaffold complete. Files created: N.

Directory layout:
<tree of top-level dirs>

Next steps (do these in order):
1. Review the scaffold with `git status` and `git diff`
2. Install dependencies: <exact commands>
3. Initial commit: `git init && git add . && git commit -m "chore: greenfield scaffold"`
4. Bootstrap infra: cd infra && az deployment sub create ...
5. Wire OIDC federation: scripts/bootstrap-oidc.sh
6. First deploy: push to main

Hand off to feature work: this project is now brownfield. Use your feature-level workflow (e.g., /rival:plan) for everything after this.
```

## Template structure expected

Each template directory at `templates/<name>/` should contain:
- `.hydrate/README.md` — substitution map and hydration notes
- Source tree with `{{...}}` substitution markers in file contents
- A `README.md` at the template root describing the template

## Constraints

- **No scaffold if blockers exist.** Refuse to proceed if `.greenfield/review/security-review.md` has unresolved BLOCKER entries.
- **Do not run package installs** automatically — print commands for the user.
- **Do not initialize git** automatically — let the user review first.
- **Idempotent on clean target.** If the target directory has conflicting files, abort and tell the user to resolve (do not overwrite).
- **Every scaffold file must be committable.** No secrets, no absolute paths, no user-specific tokens embedded.
- **Hand off cleanly.** The last message should tell the user the project is now brownfield and to use their feature workflow.
