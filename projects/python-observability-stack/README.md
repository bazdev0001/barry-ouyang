# Python Observability Stack

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?style=flat&logo=opentelemetry&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)

The incident that pushed me to build this: 3am page, production is "slow," no metrics, no traces, no structured logs. Just a graph showing error rate going up and user reports in Slack saying "it's broken." Root cause took 4 hours to identify — a memory leak in a third-party library triggered by a specific request pattern. If I'd had distributed traces, it would've been 10 minutes. Never again.

## The Incident

January, 3:47am. PagerDuty fires. I'm on-call. I SSH in and have:
- No metrics dashboard (they'd been added to the backlog for months)
- Application logs that are a mix of `print()` statements and unstructured exception tracebacks
- No way to follow a single user's request through multiple services

We found the bug eventually. A senior engineer spent most of the night adding debug logging to production. We had to redeploy 4 times during the outage window just to get enough visibility to diagnose the problem.

The next morning I wrote the ticket: "Implement observability stack." I got it prioritized that same day.

## What This Stack Gives You

Three pillars, actually working together:

**Metrics (Prometheus + Grafana)**
- Automatically collected Python process metrics (CPU, memory, GC, thread count)
- Custom business metrics via the `prometheus_client` library
- Pre-built dashboards for FastAPI, Flask, Celery, SQLAlchemy
- Alerting rules with runbook links embedded in the alert annotations

**Logs (Loki + Promtail)**
- Structured JSON logging for all Python services
- Log correlation with trace IDs so you can jump from a trace to the associated logs
- Log-based alerting (error rate from log lines, not just HTTP status codes)
- 30-day retention with compaction

**Traces (Jaeger + OpenTelemetry)**
- Automatic instrumentation for FastAPI, requests, SQLAlchemy, Redis, Celery
- W3C trace context propagation between services
- Trace-to-log correlation
- Sampling at 10% normal load, 100% on errors

## Directory Structure

```
.
├── docker-compose.yml             # Local dev stack
├── docker-compose.prod.yml        # Production with resource limits
├── prometheus/
│   ├── prometheus.yml
│   ├── alert.rules.yml            # Alerting rules (28 rules covering common failure modes)
│   └── recording.rules.yml
├── grafana/
│   ├── provisioning/
│   │   ├── dashboards/
│   │   └── datasources/
│   └── dashboards/
│       ├── python-services.json
│       ├── infrastructure.json
│       └── slo-overview.json
├── loki/
│   └── loki-config.yml
├── jaeger/
│   └── jaeger-config.yml
├── python-sdk/                    # Drop-in observability SDK for Python services
│   ├── observability/
│   │   ├── __init__.py
│   │   ├── metrics.py             # Prometheus metrics helpers
│   │   ├── logging.py             # Structured logging setup
│   │   └── tracing.py             # OTel tracer setup
│   └── middleware/
│       ├── fastapi.py             # FastAPI middleware (metrics + trace injection)
│       └── flask.py               # Flask middleware
└── examples/
    ├── fastapi-service/
    └── flask-service/
```

## The Python SDK

I got tired of copy-pasting the same observability setup into every service. The `python-sdk/` directory is a small library that handles all three pillars with a single import:

```python
from observability import configure_observability

app = FastAPI()
configure_observability(
    app=app,
    service_name="orders-api",
    service_version="1.4.2",
    prometheus_port=9090,
    jaeger_endpoint="http://jaeger:4317",
    loki_endpoint="http://loki:3100",
)
```

That's it. Your service now has metrics on `/metrics`, structured logs going to Loki, and every request traced through Jaeger with the trace ID attached to every log line.

## Running Locally

```bash
# Start the full stack
docker compose up -d

# Check everything's healthy
docker compose ps

# Access dashboards
# Grafana:    http://localhost:3000 (admin/admin)
# Prometheus: http://localhost:9090
# Jaeger:     http://localhost:16686
# Loki:       (queried via Grafana)

# Run example services to generate data
cd examples/fastapi-service
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

## Production Deployment

```bash
# Uses separate compose file with resource limits + external volume mounts
docker compose -f docker-compose.prod.yml up -d

# Or deploy to Kubernetes
helm install observability ./helm -f helm/values-prod.yaml
```

## Outcomes

- MTTD (mean time to detect): 18 minutes reduced to 2 minutes
- MTTR (mean time to resolve): 4 hours reduced to 35 minutes
- All Python services instrumented within 2 weeks of adoption
- On-call burden reduced by 60% — fewer ambiguous alerts, more actionable pages
- Zero "flying blind" incidents in 14 months post-deployment

## Requirements

- Docker and Docker Compose (local)
- Python 3.10+ (for SDK)
- 4GB RAM minimum for local stack (Grafana + Loki are hungry)
# OpenTelemetry SDK upgrade to 1.30
