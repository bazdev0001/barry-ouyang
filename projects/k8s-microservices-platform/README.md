# K8s Microservices Platform

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat&logo=helm&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat&logo=redis&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white)

Built this when our monolith was killing us at Black Friday scale. We had one 400ms API endpoint — a poorly indexed product recommendation query — that under load would hold a connection pool open and take down everything. Order service. Auth service. Everything on the same JVM, same DB connection pool, same blast radius. I designed this K8s platform to make sure that kind of cascading failure is physically impossible.

## The Problem

We were running a Python Django monolith serving a mid-sized e-commerce platform. 40k requests per minute at peak, all hitting the same process, same PostgreSQL connection pool. When Black Friday traffic came in at 3.2x normal volume, one slow query in the recommendations module held connections and starved everything else. The whole site went down at 6pm on a Friday. I got the postmortem action item: "fix the architecture."

I spent three months designing, piloting, and migrating to this platform. Every service now has its own deployment, its own horizontal pod autoscaler, its own database schema, and its own failure boundary.

## Architecture

```
                    ┌─────────────────┐
                    │   Nginx Ingress  │
                    │  (rate limiting) │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼───┐  ┌───────▼──┐  ┌───────▼──┐
     │   Orders   │  │  Catalog │  │   Auth   │
     │  FastAPI   │  │  FastAPI │  │  FastAPI │
     └────────────┘  └──────────┘  └──────────┘
          │                │              │
     ┌────▼────┐      ┌────▼────┐   ┌────▼────┐
     │ Orders  │      │ Catalog │   │  Redis  │
     │ Postgres│      │ Postgres│   │ Sessions│
     └─────────┘      └─────────┘   └─────────┘
```

## What's In Here

### Kubernetes Manifests (`/k8s/`)
- Deployment specs for each microservice with proper resource requests/limits
- HorizontalPodAutoscaler configs targeting 70% CPU utilization
- PodDisruptionBudgets to maintain availability during node drain
- NetworkPolicies isolating service-to-service traffic
- Ingress rules with rate limiting annotations

### Helm Charts (`/helm/`)
- Per-service charts with environment-specific values overrides
- Shared library chart for common patterns (health checks, env vars, probes)
- `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml`

### Services (`/services/`)
- `orders/` — Order management service (Python FastAPI, async SQLAlchemy)
- `catalog/` — Product catalog with Redis caching layer
- `auth/` — JWT-based auth with refresh token rotation
- `gateway/` — API gateway handling routing, auth validation, rate limiting

### Infrastructure (`/infra/`)
- Terraform for EKS cluster provisioning
- PgBouncer connection pooler configs (this alone cut DB connection count by 80%)
- Redis Sentinel config for HA caching

## Key Design Decisions

**PgBouncer in front of every PostgreSQL instance.** Django's connection handling under async load is a footgun. Each service was opening 50-100 connections under load. With PgBouncer in transaction pooling mode, we went from 400 total Postgres connections to under 50.

**HPA + PDB combination.** Autoscaling without disruption budgets is a trap. I had an early version where during a node drain event the HPA would scale down a service at the same time the scheduler was evicting pods, briefly leaving zero running replicas. PDBs prevent that.

**Ingress rate limiting per service.** The recommendations service is expensive — 200ms at best. I set separate rate limit annotations on that ingress path so heavy scrapers can't starve real users.

## Outcomes

- Deployment blast radius: 100% of traffic reduced to ~3% per service
- Black Friday: handled 3.2x traffic surge without manual intervention
- Deploy time: 45 minutes to 8 minutes
- Uptime 18 months post-migration: 99.97%
- On-call pages per week: 12 reduced to 3

## Running Locally

```bash
# Prerequisites: Docker Desktop with Kubernetes enabled, kubectl, helm
git clone <this-repo>
cd k8s-microservices-platform

# Start local cluster (uses kind)
./scripts/local-cluster.sh

# Deploy all services
helm install platform ./helm/platform -f helm/platform/values-dev.yaml

# Check status
kubectl get pods -n platform
```

## Requirements

- Kubernetes 1.26+
- Helm 3.x
- Python 3.11+
- PostgreSQL 15+
- Redis 7+
