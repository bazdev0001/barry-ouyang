# Zero-Downtime Deploy Toolkit

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat&logo=nginx&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)

Our CTO demanded zero-downtime deploys after a Saturday night maintenance window went 3 hours overdue and cost an estimated $80k in lost transactions. "No more maintenance windows. Ever." I took that seriously. This is what I built. Blue-green with automated health checks and one-command rollback. We ran it in production for 2 years without a user-visible outage during deployment.

## The Incident That Started This

Saturday, 11pm. Planned maintenance window: 30 minutes. Actual: 3 hours and 12 minutes.

The database schema migration took longer than expected. Then the rollback of the migration failed. Then we had half the fleet on the new code and half on the old. The on-call engineer was sweating. Users were getting errors. The CTO was awake.

Monday morning postmortem had one action item assigned to me: "Make this impossible."

## How It Works

Blue-green deployment keeps two identical environments — blue (currently live) and green (being updated). Traffic flows entirely to blue. We deploy to green, validate it, then switch traffic. Rollback is switching traffic back to blue — instant, no code change required.

```
Current State:
  Load Balancer → Blue (v1.4.1) [100% traffic]
  Green (v1.4.1) [0% traffic, idle]

Deploy v1.4.2:
  Step 1: Deploy v1.4.2 to Green
  Step 2: Run automated health checks on Green
  Step 3: Switch 10% of traffic to Green
  Step 4: Monitor error rate for 2 minutes
  Step 5: Switch 50% → 100% traffic to Green
  Step 6: Blue becomes idle (ready for next deploy)

Rollback (if needed at any step):
  ./deploy.sh rollback  # Switches all traffic back to Blue
  # Takes 30 seconds
```

## What's In Here

### Deployment Orchestrator (`/deploy/`)

**`deploy.py`** — Main orchestration script
- `deploy.py promote VERSION` — Deploy new version to inactive environment
- `deploy.py switch [PERCENT]` — Switch traffic percentage
- `deploy.py rollback` — Instant rollback to previous environment
- `deploy.py status` — Show current state of both environments

**`health.py`** — Health check framework
- Configurable check: HTTP endpoints, TCP ports, Prometheus queries
- Retry logic with exponential backoff
- Threshold-based pass/fail (e.g., error rate must stay below 0.1%)
- Timeout per check and per overall validation suite

### Kubernetes Resources (`/k8s/`)
- Two Deployment manifests (blue and green) with shared labels
- Service pointing at active environment (label selector based)
- Ingress with traffic weight annotations (for gradual rollout)
- ConfigMap for environment metadata (which color is active, last deploy timestamp)

### Nginx Configuration (`/nginx/`)
- Upstream definitions for both environments
- Weight-based traffic splitting
- Health check passive monitoring
- Access log format that includes which upstream handled each request

### Monitoring (`/monitoring/`)
- Prometheus rules for deployment health
- Grafana dashboard showing traffic split, error rates per environment, latency comparison
- Alertmanager rule: alert if error rate differs by more than 0.5% between environments

## Usage

```bash
# Deploy new version
./deploy.py promote v1.4.2

# Watch automated health checks run
# ... checking HTTP /health on green: OK
# ... checking error rate on green (threshold: 0.1%): 0.02% OK
# ... checking p99 latency on green (threshold: 200ms): 145ms OK

# Switch traffic (gradual)
./deploy.py switch 10
# Wait and observe...
./deploy.py switch 50
./deploy.py switch 100

# Or let it do the whole thing automatically
./deploy.py promote v1.4.2 --auto-promote

# If something's wrong, rollback in 30 seconds
./deploy.py rollback
```

## Health Check Configuration

```yaml
# health-checks.yml
checks:
  - name: api_health_endpoint
    type: http
    url: "http://GREEN_HOST/health"
    expected_status: 200
    timeout: 5s

  - name: error_rate
    type: prometheus
    query: 'rate(http_requests_total{status=~"5..",env="green"}[2m]) / rate(http_requests_total{env="green"}[2m])'
    threshold: 0.001  # 0.1%
    comparison: less_than

  - name: latency_p99
    type: prometheus
    query: 'histogram_quantile(0.99, http_request_duration_seconds_bucket{env="green"})'
    threshold: 0.2  # 200ms
    comparison: less_than
```

## Outcomes

- Zero maintenance windows in 2+ years of production use
- 47 production deployments without user-visible downtime
- Average deploy time: 12 minutes (including validation and gradual rollout)
- Rollback time: 30 seconds (manual) or 90 seconds (automated on health check failure)
- Engineer confidence in Friday deploys went from 2/10 to 9/10 (team survey)

## Requirements

- Kubernetes 1.25+
- Python 3.10+
- Prometheus (for health checks against metrics)
- Nginx Ingress Controller
# Traffic splitting improvements
