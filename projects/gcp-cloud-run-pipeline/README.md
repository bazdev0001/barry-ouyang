# GCP Cloud Run Pipeline

![GCP](https://img.shields.io/badge/GCP-4285F4?style=flat&logo=google-cloud&logoColor=white)
![Cloud Run](https://img.shields.io/badge/Cloud_Run-4285F4?style=flat&logo=google-cloud&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)

Built for a cost-obsessed startup. Cloud Run went from "interesting experiment" to our primary deployment target. $200/month for what used to cost $2,000 in EC2 — and we get better reliability, zero maintenance, and deploys that don't require a dedicated ops engineer to babysit.

## The Story

I joined a Series A startup that was spending $2,200/month on always-on EC2 instances for Python Flask services that genuinely only received traffic for about 4 hours a day. They were paying 24/7 for 4 hours of use. Their deploy process was SSH + rsync into a bastion host — no version control for infrastructure, no rollback, no audit trail. Engineers were scared to deploy on Fridays, and honestly they were right to be scared.

I proposed Cloud Run as an experiment for one of their less critical services. The first month we saved $400. By month three, we'd migrated everything and the bill was $180.

## Architecture

```
Developer pushes → GitHub Actions triggers →
  Docker multi-stage build →
  Push to Google Artifact Registry →
  Deploy to Cloud Run (canary) →
  Smoke tests →
  Traffic split 10% → 50% → 100% →
  Old revision retired
```

## What's In Here

### GitHub Actions Workflows (`.github/workflows/`)
- `ci.yml` — Lint, test, build on every PR
- `deploy-staging.yml` — Auto-deploy to staging on merge to `main`
- `deploy-production.yml` — Production deploy with approval gate and traffic splitting
- `rollback.yml` — One-command rollback to any previous revision

### Cloud Run Configuration (`/cloudrun/`)
- Service YAML specs with min/max instances configured
- Traffic splitting configuration for gradual rollouts
- Cloud Armor WAF rules (rate limiting, geo-blocking for our threat model)
- VPC Connector config for private database access

### Docker (`/docker/`)
- Multi-stage Dockerfiles for each service (build stage separate from runtime)
- `.dockerignore` files that actually work (no more accidentally shipping virtualenvs)
- Base image pinned to digest, not just tag

### Infrastructure (`/terraform/`)
- Cloud Run service definitions
- Artifact Registry repository
- IAM bindings for least-privilege service accounts
- Cloud Armor security policy
- Secret Manager integration

### Python Services (`/services/`)
- Flask API services instrumented with Cloud Trace
- Structured JSON logging for Cloud Logging
- Health check endpoints Cloud Run expects

## Key Decisions

**Traffic splitting for all production deploys.** Every deploy starts at 0% traffic, runs smoke tests, then steps up: 10% → 50% → 100%. The whole promotion takes about 90 seconds if tests pass. If any step fails, we stay on the old revision.

**Artifact Registry over Container Registry.** Container Registry is deprecated anyway, but more importantly Artifact Registry gives us per-region repositories, vulnerability scanning on push, and cleanup policies.

**Multi-stage Docker builds.** Our Python services were shipping 2.4GB images with dev dependencies included. Multi-stage builds got us to 180MB. Cold start time dropped from 12 seconds to 3 seconds.

**Workload Identity over service account keys.** Service account key files are a security liability — they don't expire, they get committed to repos, they're copied to developer laptops. Workload Identity Federation gives GitHub Actions access to GCP without any key file.

## Outcomes

- Infrastructure cost: $2,200/month reduced to $180/month (92% reduction)
- Deploy time: 25 minutes reduced to 4 minutes
- Zero-downtime deploys via traffic splitting
- Cold start time: 12 seconds reduced to 3 seconds
- Friday deploys became normal — 3.2 deploys/day average

## Quick Start

```bash
# Prerequisites: gcloud CLI authenticated, gh CLI

# 1. Set up GCP project
gcloud config set project YOUR_PROJECT_ID

# 2. Enable required APIs
./scripts/enable-apis.sh

# 3. Create infrastructure
cd terraform && terraform init && terraform apply

# 4. Configure GitHub secrets (see docs/setup.md)
./scripts/configure-github-secrets.sh

# 5. Push to main to trigger first deploy
git push origin main
```

## Requirements

- Google Cloud project with billing enabled
- APIs: Cloud Run, Artifact Registry, Secret Manager, Cloud Build
- GitHub repository (for Actions)
- Python 3.11+
