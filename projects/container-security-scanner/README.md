# Container Security Scanner

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![Trivy](https://img.shields.io/badge/Trivy-1904DA?style=flat&logo=aquasecurity&logoColor=white)

A startup I consulted for got hit with a cryptominer embedded in one of their container images. The image had been running in their production cluster for 3 weeks before their cloud bill spiked and someone noticed the GPU usage. This tool is my response — a lightweight CVE scanner that runs in CI and blocks images with critical vulnerabilities before they ever reach production.

## The Incident

The startup pulled a community base image from Docker Hub without verifying it. The image had been tampered with — a cryptomining binary was embedded, triggered on container start, and the container appeared healthy from the outside. Their cluster was mining cryptocurrency for 3 weeks. The attacker netted a few hundred dollars. The startup spent a week incident-responding, rotating all credentials that had ever touched that cluster, and filing insurance claims.

The fix was conceptually simple: scan every image in CI before it ships. The hard part was making scanning fast enough that engineers wouldn't bypass it and opinionated enough that it actually blocked bad images rather than just reporting on them.

## What It Does

```
Push to main →
  GitHub Actions triggers scanner →
  Build Docker image →
  scanner.py analyze IMAGE →
  Trivy CVE scan →
  Malware signature check →
  Base image provenance check →
  Secrets in layers check →
  SARIF report generated →
  If CRITICAL vulns: fail build, post PR comment →
  If MEDIUM/HIGH: warn, continue with annotation →
  SBOM attached to image manifest
```

## Features

### CVE Scanning
- Trivy integration for OS packages and language dependencies
- Configurable severity threshold (default: fail on CRITICAL, warn on HIGH)
- Ignore file support for accepted vulnerabilities with expiry dates
- Diff mode: only report new CVEs introduced in this PR

### Malware Detection
- Signature matching against known malicious binaries
- Entropy analysis for suspicious embedded executables
- Check for unexpected SUID/SGID binaries
- Cryptocurrency miner detection patterns

### Supply Chain Checks
- Base image must be from approved registry list
- Base image digest pinning verification (tag-only references fail)
- Layer history analysis for suspicious RUN commands
- Build reproducibility check

### Secret Detection in Image Layers
- Scans all image layers for credential patterns
- Detects: AWS keys, GitHub tokens, private keys, connection strings
- Exits non-zero if secrets found (cannot be ignored without explicit override)

## Usage

### In GitHub Actions

```yaml
# .github/workflows/security.yml
- name: Scan container image
  uses: ./
  with:
    image: ${{ steps.build.outputs.image }}
    severity-threshold: CRITICAL
    fail-on-severity: true
    sarif-output: results.sarif

- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: results.sarif
```

### Standalone

```bash
pip install -r requirements.txt

# Scan a local image
python scanner.py scan myapp:latest

# Scan with custom severity threshold
python scanner.py scan myapp:latest --fail-on HIGH

# Full report (JSON)
python scanner.py scan myapp:latest --output json > report.json

# CI mode (exits non-zero on failures, minimal output)
python scanner.py scan myapp:latest --ci
```

## Configuration

```yaml
# scanner-config.yml
approved_registries:
  - gcr.io/your-project
  - 123456789.dkr.ecr.us-east-1.amazonaws.com
  - ghcr.io/your-org

severity_threshold: CRITICAL  # UNKNOWN, LOW, MEDIUM, HIGH, CRITICAL

ignored_cves:
  - id: CVE-2024-12345
    reason: "Not exploitable in our runtime (no user input to this code path)"
    expires: "2025-06-01"

secret_scan:
  enabled: true
  patterns_file: .secret-patterns.yml

malware_scan:
  enabled: true
  entropy_threshold: 7.5
```

## Output Formats

**Terminal (default)** — Color-coded summary with vulnerability counts
**JSON** — Machine-readable full report with all findings
**SARIF** — GitHub Security tab compatible format
**HTML** — Human-readable report with CVE links and remediation steps

## Directory Structure

```
.
├── scanner.py                 # Main entry point
├── scanner/
│   ├── trivy.py              # Trivy wrapper and result parsing
│   ├── malware.py            # Malware detection
│   ├── secrets.py            # Secret scanning in layers
│   ├── provenance.py         # Supply chain checks
│   └── report.py             # Output formatters (JSON, SARIF, HTML)
├── action.yml                 # GitHub Action definition
├── patterns/
│   ├── malware-signatures.yml
│   └── secret-patterns.yml
└── tests/
    ├── test_scanner.py
    └── fixtures/              # Test images with known vulnerabilities
```

## Outcomes

- 23 critical CVEs blocked from reaching production in first 6 months
- Scan time: ~45 seconds average per image
- Zero cryptomining or malware incidents post-deployment
- SARIF integration gave security team full visibility without changing developer workflow
- Adopted as standard across 3 client engagements after initial build

## Requirements

- Python 3.10+
- Docker (for local scanning)
- Trivy (installed automatically by setup script)
- GitHub Actions (for CI integration)
