# azure-saas-container-apps (stub)

> **Status: v0.1.0 — stub placeholder. Must be built out before `/greenfield:scaffold` can use it.**

## Intended scope

A production-grade Azure Container Apps SaaS reference application with:

- **Compute**: Azure Container Apps (web, api, worker)
- **Infra**: Bicep + Azure Verified Modules (AVM)
- **CI/CD**: GitHub Actions with OIDC federation, SLSA L2 provenance, SBOM + cosign
- **Database**: Azure Database for PostgreSQL Flexible Server, HA, GRS backups
- **Cache**: Azure Cache for Redis
- **Messaging**: Azure Service Bus Standard
- **Auth**: Microsoft Entra External ID (customers) + Entra ID (workforce)
- **Observability**: App Insights workspace-based + Log Analytics (365+ day retention), OpenTelemetry SDK wired in every service, W3C trace context propagation
- **Secrets**: Key Vault with managed identity, no long-lived secrets
- **Frontend**: Next.js 15 App Router + Tailwind + shadcn/ui (design tokens hydrated from plan)
- **Backend**: FastAPI or Node (decided during scaffold from plan)
- **Compliance**: PIPEDA TIA + PIA templates, Law 25 consent log schema, DSR endpoints (`/me/data`, `/me/export`, `/me/data` DELETE, `/me/consent`)

## What needs to be built

- [ ] `infra/` — Bicep with AVM modules
- [ ] `infra/envs/dev.bicepparam`, `prod.bicepparam`
- [ ] `.github/workflows/ci.yaml` with lint, test, SAST, SBOM, build, sign, deploy
- [ ] `.github/workflows/cd.yaml` with OIDC to Azure
- [ ] `apps/api/` — FastAPI skeleton with OTel + structured logs + health endpoints + DSR endpoint stubs
- [ ] `apps/web/` — Next.js 15 skeleton with tailwind.config.ts token slots
- [ ] `compliance/` — TIA, PIA, consent log schema, DSR endpoint stubs
- [ ] `docs/runbooks/` — deploy, rollback, incident, oncall
- [ ] `.hydrate/README.md` — list of `{{variable}}` substitution points
- [ ] `.devcontainer/` — one-command dev env
- [ ] `README.md` — how to boot the template after scaffold

## Priority

Build this after the research-to-plan workflow is validated on a throwaway project. Premature template-building locks in assumptions that the agents haven't validated yet.
