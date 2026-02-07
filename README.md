# Flux GitOps Repository

This repository defines a Flux CD–managed GitOps setup for Kubernetes with clear separation between development and production. All changes are declarative, reproducible, and reviewed as code.

## Goals

- Predictable, safe GitOps deployments for dev and prod clusters
- Strict separation of environment concerns via overlays
- No plaintext secrets in Git; secrets are synced from Infisical
- All deployable components managed via Flux HelmRelease resources

## Repository Layout

```text
.
├── apps
│   ├── base
│   ├── development
│   └── production
├── clusters
│   ├── development
│   │   ├── flux-system
│   │   ├── infrastructure.yaml
│   │   └── kustomization.yaml
│   └── production
│       ├── flux-system
│       ├── infrastructure.yaml
│       └── kustomization.yaml
├── infrastructure
│   ├── base
│   │   ├── core
│   │   ├── namespaces.yaml
│   │   └── sources
│   ├── development
│   └── production
```

### Structure Principles

- `clusters/*` only wires Flux to a cluster and its top-level Kustomizations
- `infrastructure/base` contains reusable, environment-agnostic components
- `infrastructure/{development,production}` only override or extend `base`
- `apps/base` contains shared app manifests
- `apps/{development,production}` only override or extend `apps/base`
- No HelmRelease, Secret, or Namespace is defined directly under `clusters/`

## Flux Usage

All deployable components are managed via `HelmRelease` resources and sourced from `HelmRepository` definitions in `infrastructure/base/sources`. Rendered manifests are not committed.

Kustomizations are scoped, explicit, and may use `dependsOn` to ensure correct ordering.

## Secret Management (Infisical)

Secrets are synced into Kubernetes using the Infisical Secrets Operator (`InfisicalSecret` resources).

Environment separation is enforced via `envSlug` (`dev` vs `prod`) and by keeping development and production clusters fully isolated.

### Required Infisical Secrets

The following secrets must exist in Infisical for both `dev` and `prod` (unless explicitly noted). Paths are the `secretsPath` configured in this repo.

- `/Cert-Manager`
- `CLOUDFLARE_API_TOKEN` (used by cert-manager Cloudflare DNS-01 solver)
- `/Traefik`
- `CLOUDFLARE_API_TOKEN` (synced for Traefik; keep separate from cert-manager)
- `/Proxmox-CSI`
- `PROXMOX_API_TOKEN_ID`
- `PROXMOX_API_TOKEN_SECRET`
- `/Authentik`
- `AUTHENTIK_SECRET_KEY`
- `AUTHENTIK_EMAIL_USERNAME`
- `AUTHENTIK_EMAIL_PASSWORD`

## Flux Bootstrap

This repository follows Flux bootstrap conventions with separate paths per cluster.

```bash
export GITHUB_TOKEN="..."

flux bootstrap github --token-auth --owner=adrianzech --repository=flux --branch=main --path=clusters/development
flux bootstrap github --token-auth --owner=adrianzech --repository=flux --branch=main --path=clusters/production
```
