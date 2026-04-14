# Greenfield Templates

Templates are the paved-road reference applications that `/greenfield:scaffold` hydrates with the plan.

## Structure of a template

```
templates/
└── <template-name>/
    ├── README.md              # what this template is, when to use it
    ├── .hydrate/
    │   └── README.md          # substitution variables this template uses
    ├── <source tree>          # the actual files that get copied to target
    ├── infra/
    ├── .github/workflows/
    ├── apps/
    ├── compliance/
    └── ...
```

Files outside `.hydrate/` are copied literally. Files inside `.hydrate/` are templates with `{{variable}}` substitution markers that get hydrated by the scaffold skill from `.greenfield/plan/*.md`.

## Available templates

| Name | Status | Description |
|---|---|---|
| `azure-saas-container-apps` | **stub** | Azure Container Apps SaaS with Bicep + AVM, GitHub Actions OIDC, Postgres Flexible Server, App Insights + OTel, Entra External ID, PIPEDA/Law 25 compliance scaffolding, shadcn/ui design system |

## Adding a template

For consulting engagements on non-Azure stacks, add a new template directory. Minimum requirements for a template to be production-ready:

- [ ] IaC (Bicep/Terraform/Pulumi) for all infra
- [ ] CI/CD (GitHub Actions or equivalent) with OIDC federation — no long-lived secrets
- [ ] OpenTelemetry instrumentation wired from commit zero
- [ ] Structured logging with correlation IDs
- [ ] Health endpoints (`/health`, `/ready`, `/metrics`)
- [ ] SBOM + SAST + container scanning in CI
- [ ] DSR endpoint stubs (`/me/data`, `/me/export`, etc.)
- [ ] Audit log retention ≥ 365 days
- [ ] Design system integration points (tailwind.config, theme tokens)
- [ ] README with bootstrap instructions
- [ ] `.hydrate/README.md` with substitution variable list

## v0.1.0 status

Templates are stubs. The plugin workflow (init → research → architect → review) is fully functional; the scaffold step exists but the azure-saas-container-apps template is a placeholder and must be built out before scaffold is useful. This is by design — iterate on the workflow first, then invest in the template once the research-to-plan loop is validated.
