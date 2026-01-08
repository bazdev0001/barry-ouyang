# Azure AKS GitOps

![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat&logo=microsoft-azure&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat&logo=argo&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat&logo=helm&logoColor=white)

Enterprise client, Azure-locked, teams deploying manually via kubectl. Nobody could tell you what was actually running in production without SSHing into a cluster. GitOps transformed their release velocity from 2 deploys/week to 15 deploys/day — and for the first time, the cluster state was auditable from a git log.

## The Situation

200-person company, about 30 engineers. Azure-locked due to an enterprise Microsoft agreement. Their platform team was applying Kubernetes manifests manually via kubectl with no audit trail. Releases required:

1. Engineer SSHs into bastion
2. Edits YAML inline or copies from local machine
3. `kubectl apply -f` hoping nothing breaks
4. 2-hour change window approved by change advisory board
5. Operations engineer on Zoom call for the whole window

Two deploys per week was their target velocity. The business wanted daily deployments. The ops team said "impossible." I said there's a better way.

## Solution: GitOps with ArgoCD

The core insight: if the desired state lives in git and ArgoCD continuously reconciles the actual state to match it, then "deploying" is just opening a pull request. The change window goes away. The audit trail is the git log.

## Architecture

```
Developer PR → Azure DevOps Review → Merge to main →
  ArgoCD detects diff →
  Applies to dev cluster →
  Automated smoke tests →
  Promotion PR to staging values →
  Staging deploy →
  Manual approval for prod →
  Production deploy (blue-green)
```

## Repository Structure

```
.
├── apps/                          # ArgoCD Application manifests
│   ├── dev/
│   ├── staging/
│   └── prod/
├── charts/                        # Internal Helm charts
│   ├── api-service/
│   ├── worker-service/
│   └── _library/                  # Shared helpers
├── clusters/                      # Cluster-level config (ArgoCD, cert-manager)
│   ├── dev/
│   ├── staging/
│   └── prod/
├── infrastructure/                # AKS + supporting Azure resources
│   ├── terraform/
│   └── pipelines/
└── scripts/                       # Operational tooling
```

## Key Components

### ArgoCD Setup (`/clusters/`)
- ArgoCD installed via Helm, self-managed (applies its own config)
- App of Apps pattern — one root Application manages all others
- SSO via Azure AD (existing corporate accounts, no new credentials)
- RBAC: read-only for most engineers, deploy access for leads, admin restricted to platform team

### Helm Charts with Environment Overrides (`/charts/`)
- Base values in `values.yaml` — shared defaults
- `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml` for overrides
- Sealed Secrets for in-repo secret management (encrypted, safe to commit)
- External Secrets Operator integration with Azure Key Vault

### Azure Infrastructure (`/infrastructure/terraform/`)
- AKS clusters (3 environments) with Azure CNI
- Azure Container Registry with geo-replication to EU and APAC
- Azure Key Vault per environment
- Azure Monitor integration + Log Analytics workspace
- Managed identities (workload identity, no service principal secrets)

### Azure DevOps Pipelines (`/infrastructure/pipelines/`)
- PR validation pipeline: linting, Helm template rendering, Conftest policy checks
- Docker build pipeline: multi-stage build, push to ACR, Trivy scan
- Promotion pipeline: automated PR creation for staging and prod promotions

## Key Design Decisions

**Sealed Secrets for in-repo secret management.** I evaluated Azure Key Vault CSI, External Secrets Operator, and Sealed Secrets. For this team, Sealed Secrets won because it requires no additional cluster dependencies and secrets live in the same PR as the code that uses them.

**App of Apps pattern.** Single root ArgoCD Application that manages all other Applications. Onboarding a new service is one file in `apps/dev/`. No cluster access required.

**Renovate Bot for dependency updates.** Automated PRs for Helm chart version updates, base image bumps, and Kubernetes API deprecations. The platform team reviews and merges; engineers don't have to track upstream.

## Outcomes

- Release frequency: 2/week increased to 15/day
- Change window eliminated — all deploys via pull request
- Mean time to deploy: 2 hours reduced to 6 minutes
- Rollback time: 45 minutes reduced to 90 seconds (revert + auto-sync)
- Cluster drift incidents: 3/month reduced to zero (continuous reconciliation)

## Getting Started

```bash
# Prerequisites: az CLI, kubectl, helm, argocd CLI

# 1. Create AKS clusters
cd infrastructure/terraform
terraform init && terraform apply

# 2. Install ArgoCD
kubectl apply -k clusters/dev/argocd

# 3. Bootstrap App of Apps
argocd app create root \
  --repo https://dev.azure.com/YOUR_ORG/YOUR_REPO \
  --path apps/dev \
  --dest-server https://kubernetes.default.svc

# 4. Watch ArgoCD sync everything
argocd app sync root
```

## Requirements

- Azure subscription
- AKS 1.27+
- ArgoCD 2.9+
- Helm 3.x
- Azure DevOps (or GitHub Actions — pipelines are portable)
# ArgoCD upgrade notes 2.10
