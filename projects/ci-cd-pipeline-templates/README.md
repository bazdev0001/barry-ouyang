# CI/CD Pipeline Templates

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat&logo=jenkins&logoColor=white)

I've set up CI/CD pipelines at 12+ companies over 8 years. The same anti-patterns keep appearing: no caching so builds take 40 minutes, secrets hardcoded in YAML files, no semantic versioning, test parallelism as an afterthought, notification spam that everyone ignores. These templates are my collected wisdom — opinionated defaults with documented escape hatches.

## Why Templates?

In 2023, I did a consulting engagement where I audited the CI/CD setup for a 50-engineer company. Their GitHub Actions pipelines were:
- 47 minutes average per build (Docker layer cache not used, pip install from scratch every time)
- Secrets in YAML committed to the repo (I found 3 distinct API keys)
- No parallelism — linting, testing, and building ran sequentially
- Zero version pinning on actions (using `@main` in production pipelines)

I rebuilt their main pipeline using patterns I'd developed over the years. Build time went to 9 minutes. In the first month, they had zero failed production deploys.

These templates are what I use as the starting point everywhere now.

## Templates

### GitHub Actions (`/github-actions/`)

**`python-ci.yml`** — Python project CI
- Dependency caching (pip + pre-commit hooks)
- Matrix testing across Python versions
- Parallel: lint + typecheck + tests in separate jobs
- Coverage reporting to PR comment
- Build time: typically 4-8 minutes for a medium Python project

**`docker-build-push.yml`** — Docker image build and push
- Multi-stage build with BuildKit
- Layer caching via GitHub Actions cache or ECR cache-from
- Trivy vulnerability scan before push
- SBOM generation
- Multi-platform builds (linux/amd64, linux/arm64)

**`semantic-release.yml`** — Versioning and changelog
- Conventional commits enforcement (commitlint)
- Automated semver bump based on commit types
- CHANGELOG.md generation
- GitHub release creation with release notes

**`deploy-k8s.yml`** — Kubernetes deployment
- Helm diff on PR (shows exactly what will change)
- Deploy to staging on merge, production via approval gate
- Smoke tests after each environment
- Automated rollback on failed health checks

**`security-scan.yml`** — Security pipeline
- SAST via Semgrep
- Dependency audit (pip-audit, safety)
- Secret scanning (detect-secrets)
- Container scanning (Trivy)
- SARIF output to GitHub Security tab

### Jenkins (`/jenkins/`)

**`Jenkinsfile.python`** — Python project (for teams not yet on GitHub Actions)
- Shared library compatible
- Parallel stages for lint/test/build
- JUnit test result publishing
- Blue Ocean compatible

### Shared Utilities (`/scripts/`)

**`notify.sh`** — Slack/PagerDuty notifications
- Only pings on failure (no success spam by default)
- Includes commit author, PR link, and failed step
- Escalates to PagerDuty if deploy pipeline fails

**`semver-bump.sh`** — Version bumping
- Reads conventional commit messages since last tag
- Outputs the appropriate semver increment
- Used by the semantic-release workflow

**`health-check.sh`** — Post-deploy validation
- Configurable endpoint checks
- Retries with exponential backoff
- Returns non-zero on failure (triggers rollback in CI)

## Directory Structure

```
.
├── github-actions/
│   ├── python-ci.yml
│   ├── docker-build-push.yml
│   ├── semantic-release.yml
│   ├── deploy-k8s.yml
│   └── security-scan.yml
├── jenkins/
│   ├── Jenkinsfile.python
│   └── shared-library/
├── scripts/
│   ├── notify.sh
│   ├── semver-bump.sh
│   └── health-check.sh
├── docs/
│   ├── customization-guide.md
│   └── anti-patterns.md           # The traps I've seen teams fall into
└── examples/
    ├── fastapi-project/
    └── react-app/
```

## How to Use

```bash
# Copy the template that fits your project
cp github-actions/python-ci.yml YOUR_PROJECT/.github/workflows/ci.yml

# Edit the required variables at the top
# Each template has a "REQUIRED CONFIGURATION" section

# For the full recommended stack
./scripts/setup.sh YOUR_PROJECT_PATH
```

## Key Opinions Baked In

**Pin your action versions to SHA, not tags.** `actions/checkout@v4` can change without warning. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683` won't. The templates use SHA pins.

**Separate jobs for lint, test, build.** If lint fails, you want to know fast without waiting for tests. Parallel jobs give you faster feedback and lower per-commit cost.

**No secrets in environment variables at the workflow level.** All secrets flow through OIDC (GitHub's identity federation) or are pulled from Secrets Manager at runtime. Never `${{ secrets.MY_SECRET }}` inline in commands.

**Build once, deploy many.** The Docker image is built and tagged in CI, then the same artifact is promoted through environments. No rebuilding on each environment.

## Outcomes

- Average pipeline time at adopting teams: 40 minutes reduced to 8 minutes
- Secrets incidents: zero since adoption (previously 1-2/year at typical shops)
- Template adopted by 4 teams at one company, estimated 800 engineer-hours/year saved
- Failed production deploys: 1-2/month reduced to zero over 14 months

## Requirements

- GitHub repository
- For Kubernetes templates: kubectl configured, Helm 3.x
- For Docker templates: Docker Hub or private registry
- Python 3.10+ (for Python templates)
