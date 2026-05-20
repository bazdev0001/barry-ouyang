# Portfolio Overview — Barry Ouyang

A curated collection of DevOps, SRE, and platform engineering projects built across 8+ years in production infrastructure. These aren't toy examples — each one came out of a real operational problem that needed solving.

---

## 1. K8s Microservices Platform

**Link:** [./projects/k8s-microservices-platform/](./projects/k8s-microservices-platform/)

**Problem:** A Python monolith serving 40k req/min during peak traffic was a single point of failure. One slow database query during a Black Friday promotion cascaded into a full outage. The team had no way to isolate failure domains, and deployments required taking the whole app offline.

**Tech Stack:**
- Kubernetes 1.28 on AWS EKS
- Helm 3 for chart management
- Python FastAPI (each microservice)
- PostgreSQL with PgBouncer connection pooling
- Redis for caching and session state
- Nginx Ingress with rate limiting
- Horizontal Pod Autoscaler

**Key Outcomes:**
- Reduced deployment blast radius from 100% to per-service (avg 3% of traffic affected)
- Deployment time dropped from 45 minutes to 8 minutes
- 99.97% uptime over 18 months post-migration
- Black Friday peak: handled 3.2x normal traffic without manual intervention

**Year:** 2024

---

## 2. AWS EKS Terraform Module

**Link:** [./projects/aws-eks-terraform/](./projects/aws-eks-terraform/)

**Problem:** Every new AWS EKS cluster stood up at the company took 2-3 days of console clicking, missing IAM policies, and post-hoc drift fixes. There was no consistent baseline — every cluster was a snowflake. Disaster recovery was theoretical, not tested.

**Tech Stack:**
- Terraform 1.6+
- AWS EKS (managed node groups + Fargate profiles)
- AWS IAM (IRSA — IAM Roles for Service Accounts)
- VPC with public/private subnet topology
- ALB Ingress Controller
- Route53 + ACM for TLS
- AWS Secrets Manager integration

**Key Outcomes:**
- New cluster provisioning time: 3 days → 35 minutes
- 100% infrastructure state tracked in Terraform (zero console drift)
- Module reused across 6 production clusters
- DR test: full cluster rebuild completed in 42 minutes

**Year:** 2024

---

## 3. GCP Cloud Run Pipeline

**Link:** [./projects/gcp-cloud-run-pipeline/](./projects/gcp-cloud-run-pipeline/)

**Problem:** A Series A startup was burning $2,200/month on always-on EC2 instances for workloads that peaked for 4 hours a day. Their deploy process was SSH + rsync into a bastion host. No rollback story. Engineers were afraid to deploy on Fridays.

**Tech Stack:**
- Google Cloud Run (fully managed)
- Google Cloud Build for image builds
- GitHub Actions for pipeline orchestration
- Docker multi-stage builds
- Google Artifact Registry
- Python (Flask services)
- Cloud Armor for WAF

**Key Outcomes:**
- Infrastructure cost: $2,200/month → $180/month (92% reduction)
- Deploy time: 25 minutes → 4 minutes
- Zero-downtime deploys with traffic splitting and instant rollback
- Friday deploys became normal — 3.2 deploys/day average

**Year:** 2024

---

## 4. Azure AKS GitOps

**Link:** [./projects/azure-aks-gitops/](./projects/azure-aks-gitops/)

**Problem:** A 200-person enterprise client was Azure-locked and their platform team was manually applying Kubernetes manifests via kubectl. Releases required a 2-hour change window, a ticket, and a dedicated ops engineer present. Two deploys per week was considered "fast."

**Tech Stack:**
- Azure AKS (multiple clusters: dev, staging, prod)
- ArgoCD for GitOps reconciliation
- Helm with environment-specific values overrides
- Azure DevOps for PR pipelines
- Azure Container Registry
- Azure Key Vault + External Secrets Operator
- Renovate Bot for automated dependency updates

**Key Outcomes:**
- Release frequency: 2/week → 15/day
- Change window eliminated — all deploys via pull request
- Mean time to deploy: 2 hours → 6 minutes
- Incident response improved: rollback from 45 minutes to 90 seconds

**Year:** 2025

---

## 5. Python Observability Stack

**Link:** [./projects/python-observability-stack/](./projects/python-observability-stack/)

**Problem:** Got paged at 3am for a production incident with no metrics, no distributed traces, no structured logs — just anecdotes from users saying "it's slow." Root cause took 4 hours to identify. I built this stack so that never happens again.

**Tech Stack:**
- Prometheus + Alertmanager (metrics + alerting)
- Grafana (dashboards + on-call runbooks)
- Loki + Promtail (log aggregation)
- Jaeger (distributed tracing)
- OpenTelemetry Python SDK (instrumentation)
- Docker Compose (local development parity)
- PagerDuty integration

**Key Outcomes:**
- Mean time to detect (MTTD): 18 minutes → 2 minutes
- Mean time to resolve (MTTR): 4 hours → 35 minutes
- 100% of Python services instrumented with traces and structured logs
- Oncall burden reduced by 60% — fewer ambiguous alerts, more actionable pages

**Year:** 2025

---

## 6. CI/CD Pipeline Templates

**Link:** [./projects/ci-cd-pipeline-templates/](./projects/ci-cd-pipeline-templates/)

**Problem:** I've set up CI/CD pipelines at 12+ companies across 8 years. The same anti-patterns keep appearing: no caching, no parallelism, test suites that take 40+ minutes, no semantic versioning, secrets hardcoded in YAML. These templates encode what I've learned the hard way.

**Tech Stack:**
- GitHub Actions (primary)
- Jenkins (legacy enterprise support)
- Docker layer caching
- Semantic versioning with conventional commits
- Trivy security scanning in pipeline
- Slack/PagerDuty notifications
- Python build tooling

**Key Outcomes:**
- Pipeline runtime improvement: average 40-minute builds → 8 minutes across adopting teams
- Secrets incidents eliminated (all credentials via GitHub OIDC or Secrets Manager)
- Template adopted across 4 teams at one company, saved estimated 800 engineer-hours/year
- Zero failed prod deploys in 14 months of use (versus 1-2/month prior)

**Year:** 2025

---

## 7. Zero-Downtime Deploy Toolkit

**Link:** [./projects/zero-downtime-deploy/](./projects/zero-downtime-deploy/)

**Problem:** Our CTO issued a hard requirement after a Saturday night maintenance window went 3 hours overdue and cost an estimated $80k in lost transactions. "No more maintenance windows. Ever." This toolkit is what I built to make that demand technically achievable.

**Tech Stack:**
- Kubernetes (blue-green deployment controller)
- Python (orchestration scripts and health check framework)
- Bash (operational tooling)
- Nginx (traffic switching)
- Prometheus (deployment health metrics)
- Automated rollback on health check failure

**Key Outcomes:**
- Zero maintenance windows in 2+ years of production use
- Average deploy time: 12 minutes with automated validation
- Rollback time: 90 seconds (automated) or 30 seconds (manual trigger)
- Successfully handled 47 production deployments without user-visible downtime

**Year:** 2025

---

## 8. Container Security Scanner

**Link:** [./projects/container-security-scanner/](./projects/container-security-scanner/)

**Problem:** A startup I was consulting for discovered a cryptomining binary embedded in a third-party base image that had been running in their production cluster for 3 weeks. Nobody had scanned their images before pushing to production. This tool is my response — a lightweight, opinionated scanner that blocks vulnerable images before they ship.

**Tech Stack:**
- Python (scanner orchestration and reporting)
- Trivy (CVE database and image scanning)
- Docker (local image inspection)
- GitHub Actions (CI integration)
- JSON/HTML reporting
- Slack/webhook notifications
- SARIF output for GitHub Security tab

**Key Outcomes:**
- Blocked 23 critical CVEs from reaching production in first 6 months
- Scan time per image: ~45 seconds average
- Zero cryptomining or malware incidents post-deployment
- SARIF integration gave security team visibility without changing developer workflow

**Year:** 2026
# Portfolio refresh — updated outcomes and metrics
