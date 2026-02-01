# Flux GitOps Repository

This repository defines a Flux CD–managed GitOps setup for Kubernetes with clear separation between development and production. All changes are declarative, reproducible, and reviewed as code.

## Goals

- Predictable, safe GitOps deployments for dev and prod clusters
- Strict separation of environment concerns via overlays
- No plaintext secrets in Git; secrets come from HashiCorp Vault
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

## Secret Management (Vault)

Kubernetes consumes secrets via External Secrets Operator or the Vault CSI driver. Vault paths are environment-scoped:

```text
kv/
└── kubernetes/
    ├── production/
    │   └── <app-name>/
    │       ├── username
    │       └── password
    └── development/
        └── <app-name>/
            ├── username
            └── password
```
