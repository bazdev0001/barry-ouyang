# Barry Ouyang

**Senior DevOps Engineer | SRE | Cloud Infrastructure | Open Source**

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Go](https://img.shields.io/badge/Go-00ADD8?style=flat&logo=go&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)
![GCP](https://img.shields.io/badge/GCP-4285F4?style=flat&logo=google-cloud&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat&logo=microsoft-azure&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat&logo=argo&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)

---

I started out as a backend Python developer — building Django APIs, writing async workers, fighting with database migrations. I was decent at it. But around year three, the company I was at started hitting real scale problems: our monolith was crumbling under Black Friday traffic, deploys were terrifying, and nobody had visibility into what was actually happening in production. I ended up being the person who said "I'll figure out the infrastructure." That was 2017. I never looked back.

My cloud home is AWS — I've run everything from scrappy startups on $500/month EC2 budgets to regulated financial services workloads with multi-region failover and compliance requirements. I work in GCP and Azure regularly too; the mental model transfers, the footguns are just in different places. Kubernetes has been my daily driver since 2019. I've built K8s clusters on all three major clouds, migrated workloads onto it, and spent enough late nights debugging networking issues that I now have strong opinions about CNI plugins and strong feelings about the people who thought flannel was a good default.

My philosophy is that infrastructure is a product. The people who consume it are developers, and developer experience matters as much as uptime. I care about observability first — if you can't see it, you can't fix it. I care about automation that actually reduces toil rather than just moving it around. I care about zero-downtime deployments not as a nice-to-have but as a baseline expectation. When I join a team, I'm asking: how long does a deploy take, how do you know if it went wrong, and what's the blast radius if it does?

Right now I'm focused on platform engineering — building internal developer platforms that abstract away infrastructure complexity without hiding it when developers need to reach in. GitOps is central to everything I'm shipping: declarative desired state, automated reconciliation, pull-based deployments. FinOps is increasingly part of the picture too — cloud costs have a way of ballooning when nobody's watching the bill, and I've found that a well-instrumented cost dashboard pays for itself within the first month.

---

## Projects

| Project | Description |
|---|---|
| [k8s-microservices-platform](./projects/k8s-microservices-platform/) | Production-grade K8s platform that saved us from Black Friday monolith meltdowns |
| [aws-eks-terraform](./projects/aws-eks-terraform/) | Opinionated Terraform module for production EKS — everything I wish existed when I started |
| [gcp-cloud-run-pipeline](./projects/gcp-cloud-run-pipeline/) | Cloud Run CI/CD pipeline that cut our infrastructure costs by 90% |
| [azure-aks-gitops](./projects/azure-aks-gitops/) | GitOps setup on Azure AKS that took a team from 2 deploys/week to 15 deploys/day |
| [python-observability-stack](./projects/python-observability-stack/) | Full-stack observability (metrics, logs, traces) so 3am pages have actual context |
| [ci-cd-pipeline-templates](./projects/ci-cd-pipeline-templates/) | Collected CI/CD wisdom from 12+ companies, packaged as reusable templates |
| [zero-downtime-deploy](./projects/zero-downtime-deploy/) | Blue-green deployment toolkit with automated health checks and instant rollback |
| [container-security-scanner](./projects/container-security-scanner/) | Lightweight CVE scanner for container images — runs in CI, blocks bad images before prod |

---

## Skills

**Cloud Platforms**
AWS (EKS, ECS, Lambda, RDS, S3, Route53, ALB, IAM), GCP (GKE, Cloud Run, Cloud Build, BigQuery), Azure (AKS, Azure DevOps, ACR, Azure Functions)

**Container & Orchestration**
Kubernetes, Helm, Docker, containerd, Kustomize, ArgoCD, Flux

**Infrastructure as Code**
Terraform, Pulumi, AWS CDK, Ansible, CloudFormation

**CI/CD**
GitHub Actions, Jenkins, GitLab CI, CircleCI, ArgoCD, Tekton

**Observability**
Prometheus, Grafana, Loki, Jaeger, OpenTelemetry, Datadog, PagerDuty, ELK Stack

**Languages**
Python (primary), Go (secondary), Bash, TypeScript

**Databases**
PostgreSQL, MySQL, Redis, DynamoDB, MongoDB

---

## Contact

- GitHub: [github.com/bazdev0001](https://github.com/bazdev0001)
- Email: barry@barry-ouyang.dev
