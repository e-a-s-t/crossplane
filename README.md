# Crossplane

GitOps-managed installation of [Crossplane](https://www.crossplane.io/) for the homelab platform cluster.

Crossplane runs on the dedicated platform Kubernetes cluster and is used as a control plane for managing infrastructure and services outside the cluster, initially focusing on the Proxmox homelab environment.

> [!IMPORTANT]
> This repository is intended for **personal homelab use only**.
>
> The configuration is not designed, reviewed, or hardened for production environments.

## Overview

Crossplane is installed and managed by Argo CD using the official Crossplane Helm chart.

```text
GitHub
   │
   ▼
Argo CD
   │
   ▼
Crossplane
   │
   ├── Providers
   ├── Functions
   └── Platform APIs
           │
           ▼
      Infrastructure
           │
           └── Proxmox
```

The Kubernetes cluster running Crossplane acts as a dedicated **platform/control-plane cluster**.

The intention is to keep workloads off this cluster and instead use it to manage infrastructure and other Kubernetes clusters.

## Repository Structure

```text
.
├── argocd
│   └── application.yaml
├── LICENSE
└── README.md
```

### `argocd/`

Contains the Argo CD `Application` used to install and manage Crossplane.

The application deploys the official Crossplane Helm chart from:

```text
https://charts.crossplane.io/stable
```

Crossplane is installed into:

```text
crossplane-system
```

## Installation

The repository itself is consumed by the existing Argo CD installation.

Apply the Argo CD application:

```bash
kubectl apply -f argocd/application.yaml
```

Argo CD will then install Crossplane and continuously reconcile the installation.

## Verify Installation

Check the Argo CD application:

```bash
kubectl -n argocd get application crossplane
```

Expected state:

```text
NAME         SYNC STATUS   HEALTH STATUS
crossplane   Synced        Healthy
```

Check the Crossplane pods:

```bash
kubectl -n crossplane-system get pods
```

Crossplane should also be visible through the Crossplane CLI:

```bash
crossplane beta trace
```

or directly through Kubernetes:

```bash
kubectl get deployments -n crossplane-system
```

## GitOps

Crossplane is managed entirely through GitOps.

Changes to the Crossplane installation should be made in this repository and reconciled by Argo CD rather than manually modifying resources in the cluster.

```text
Change
  │
  ▼
Git commit
  │
  ▼
GitHub
  │
  ▼
Argo CD
  │
  ▼
Cluster
```

Argo CD is configured with:

* Automatic synchronization
* Automatic pruning
* Self-healing
* Automatic creation of the `crossplane-system` namespace

## Secrets

Secrets should **not** be committed to this repository.

The cluster uses External Secrets Operator together with 1Password for secret management.

Provider credentials should therefore be retrieved through External Secrets and exposed to Crossplane as Kubernetes Secrets where required.

```text
1Password
    │
    ▼
External Secrets Operator
    │
    ▼
Kubernetes Secret
    │
    ▼
Crossplane ProviderConfig
```

## Providers

Providers will be added as the platform evolves.

The initial goal is to allow Crossplane to manage the Proxmox homelab environment.

Future repository structure may therefore include:

```text
.
├── argocd/
├── providers/
│   └── proxmox/
├── functions/
├── LICENSE
└── README.md
```

Provider installation and configuration should also be managed through GitOps.

## Platform APIs

Crossplane platform APIs, XRDs, Compositions, and Functions may be developed separately from this repository.

This repository primarily owns the **Crossplane runtime and its integration with the homelab platform cluster**.

This keeps the responsibilities separated:

```text
crossplane
    │
    ├── Crossplane installation
    ├── Providers
    ├── Provider configuration
    └── Functions

Platform API repositories
    │
    ├── XRDs
    ├── Compositions
    └── Platform APIs
```

## Goal

The long-term architecture is:

```text
                       Git
                        │
                        ▼
                     Argo CD
                        │
                        ▼
                Platform Cluster
                        │
                   Crossplane
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
          Proxmox             Other APIs
             │
             ▼
     Kubernetes Clusters
             │
             ▼
        Applications
```

The platform cluster remains small and dedicated to managing the rest of the homelab infrastructure.

## License

See [LICENSE](LICENSE).
