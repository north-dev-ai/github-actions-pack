# Detailed Usage Guide

## Table of Contents
1. [Node.js CI/CD](#nodejs-cicd)
2. [Python CI](#python-ci)
3. [Docker Build & Push](#docker-build--push)
4. [Security Scanning](#security-scanning)
5. [Automated Release](#automated-release)
6. [PR Quality Checks](#pr-quality-checks)
7. [Monorepo CI](#monorepo-ci)
8. [Scheduled Cleanup](#scheduled-cleanup)

---

## Node.js CI/CD

**File:** `nodejs-ci-cd.yml`

### What It Does
- Runs on every push to `main` and `develop`
- Runs on all pull requests
- Tests on Node.js 18.x, 20.x, and 22.x
- Executes: install → lint → test → build
- Uploads code coverage (optional)
- Deploys to production on main branch pushes

### Prerequisites
Your project must have these npm scripts:
```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest",
    "build": "tsc",
    "deploy": "your-deploy-command"
  }
}
```

### Secrets Needed
- `CODECOV_TOKEN` (optional) - Get from codecov.io
- `DEPLOY_TOKEN` - Your deployment platform's token

### Customization
```yaml
# Change Node versions
strategy:
  matrix:
    node-version: [18.x, 20.x, 22.x]  # Edit here

# Change branches
on:
  push:
    branches: [ main, develop ]  # Edit here

# Remove deployment step if not needed
# Delete the entire "deploy:" job
```

---

## Python CI

**File:** `python-ci.yml`

### What It Does
- Tests on Python 3.9, 3.10, 3.11, 3.12
- Lints with ruff (fast modern linter)
- Type-checks with mypy
- Tests with pytest
- Publishes to PyPI when you push a version tag

### Prerequisites
Your project structure:
```
project/
├── requirements.txt
├── requirements-dev.txt
└── pyproject.toml or setup.py
```

### Secrets Needed
- `CODECOV_TOKEN` (optional)
- `PYPI_TOKEN` - Get from pypi.org

### Publishing to PyPI
```bash
# Create and push a version tag
git tag v1.0.0
git push origin v1.0.0

# Workflow automatically builds and publishes
```

### Customization
```yaml
# Change Python versions
strategy:
  matrix:
    python-version: ['3.9', '3.10', '3.11', '3.12']

# Skip type checking if you don't use it
# Remove the "Type check with mypy" step
```

---

## Docker Build & Push

**File:** `docker-build.yml`

### What It Does
- Builds Docker images on every push
- Tags images intelligently (branch, PR, semver, SHA)
- Pushes to GitHub Container Registry (ghcr.io)
- Scans for vulnerabilities with Trivy
- Uploads security results to GitHub Security tab

### Prerequisites
- Dockerfile in your repo root
- No secrets needed (uses GITHUB_TOKEN automatically)

### Image Tagging
Your images get multiple tags automatically:
- `main` - for main branch
- `pr-123` - for pull requests
- `v1.2.3`, `v1.2`, `v1` - for version tags
- `main-abc1234` - branch + commit SHA

### Accessing Your Images
```bash
# Pull your image
docker pull ghcr.io/your-username/your-repo:main

# In Kubernetes, Docker Compose, etc.
image: ghcr.io/your-username/your-repo:v1.2.3
```

### Customization
```yaml
# Change registry (to Docker Hub, AWS ECR, etc.)
env:
  REGISTRY: ghcr.io  # Change here

# Add build arguments
- name: Build and push
  uses: docker/build-push-action@v5
  with:
    build-args: |
      NODE_ENV=production
      API_URL=${{ secrets.API_URL }}
```

---

## Security Scanning

**File:** `security-scan.yml`

### What It Does
- **Dependency scanning**: npm audit and pip-audit
- **Secret scanning**: TruffleHog checks for leaked credentials
- **Code scanning**: CodeQL static analysis
- Runs on every push/PR + weekly schedule
- Results appear in GitHub Security tab

### Prerequisites
- None! Works out of the box
- Adapts to your project (detects package.json, requirements.txt)

### What It Finds
- Known vulnerabilities in dependencies
- Hardcoded secrets, API keys, passwords
- Security issues in code (SQL injection, XSS, etc.)

### Viewing Results
1. Go to your repo's **Security** tab
2. Check **Dependabot alerts** for dependencies
3. Check **Code scanning alerts** for code issues
4. Check **Secret scanning alerts** for leaked credentials

### Customization
```yaml
# Change schedule (default: weekly Monday 6 AM)
schedule:
  - cron: '0 6 * * 1'  # Use crontab.guru to change

# Add more CodeQL languages
strategy:
  matrix:
    language: [ 'javascript', 'python', 'go', 'java' ]
```

---

## Automated Release

**File:** `auto-release.yml`

### What It Does
- Analyzes your commit messages
- Automatically determines next version (major.minor.patch)
- Creates GitHub releases with changelogs
- Optionally publishes to NPM

### Prerequisites
Use conventional commits:
```bash
feat: add new feature      # → minor version bump
fix: fix bug              # → patch version bump
feat!: breaking change    # → major version bump
docs: update README       # → no version bump
```

### Setup
1. Copy workflow to `.github/workflows/`
2. Create `.releaserc.json` in repo root:
```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/github",
    "@semantic-release/git"
  ]
}
```

### Secrets Needed
- `GITHUB_TOKEN` (auto-provided)
- `NPM_TOKEN` (only if publishing to NPM)

---

## PR Quality Checks

**File:** `pr-checks.yml`

### What It Does
- Enforces semantic PR titles
- Adds size labels (XS, S, M, L, XL)
- Warns about large PRs (>1000 lines)
- Reminds to link issues
- Validates description length

### PR Title Format
```
feat: add user authentication
fix: resolve memory leak in parser
docs: update API documentation
chore: bump dependencies
```

### Size Labels
- XS: <100 lines
- S: 100-300 lines
- M: 300-500 lines
- L: 500-1000 lines
- XL: >1000 lines (triggers warning)

### Linking Issues
Include in PR description:
```
Fixes #123
Closes #456
Resolves #789
```

---

## Monorepo CI

**File:** `monorepo-ci.yml`

### What It Does
- Detects which packages changed
- Only tests affected packages (saves time!)
- Handles shared dependencies correctly
- Runs integration tests after package tests

### Prerequisites
Monorepo structure:
```
repo/
├── packages/
│   ├── frontend/
│   ├── backend/
│   └── shared/
└── package.json (with workspaces)
```

### Customization
**Critical:** Adjust paths to match your structure:
```yaml
filters: |
  frontend:
    - 'packages/frontend/**'
    - 'packages/shared/**'  # Also test frontend if shared changes
  backend:
    - 'packages/backend/**'
    - 'packages/shared/**'
```

For a different structure:
```yaml
filters: |
  api:
    - 'apps/api/**'
    - 'libs/**'
  web:
    - 'apps/web/**'
    - 'libs/**'
```

---

## Scheduled Cleanup

**File:** `cleanup-tasks.yml`

### What It Does
- Closes stale issues (60 days inactive)
- Closes stale PRs (30 days inactive)
- Deletes old artifacts (>30 days)
- Deletes old workflow runs (>90 days)
- Runs weekly + manual trigger

### Configuration
```yaml
# Stale issue settings
days-before-stale: 60        # When to mark stale
days-before-close: 7         # After stale, when to close
stale-issue-label: 'stale'   # Label to apply

# Exempt labels (never mark as stale)
exempt-issue-labels: 'pinned,security,roadmap'
```

### Manual Trigger
1. Go to **Actions** tab
2. Select "Scheduled Cleanup"
3. Click **Run workflow**

---

## Tips & Tricks

### Combining Workflows
Most projects want:
- `nodejs-ci-cd.yml` OR `python-ci.yml` (language-specific)
- `security-scan.yml` (always recommended)
- `pr-checks.yml` (improves code review quality)
- `cleanup-tasks.yml` (keeps repo clean)

### Debugging Workflows
- Check the **Actions** tab for detailed logs
- Common issues:
  - Missing secrets → Add in Settings > Secrets
  - Wrong branch names → Check workflow `on:` section
  - Missing npm scripts → Add to package.json

### Performance Tips
- Dependency caching is already enabled (saves time)
- Monorepo workflow only tests changed packages
- Use matrix testing for parallel execution

### Security Best Practices
- Always use pinned action versions (`@v4`, not `@main`)
- Minimize token permissions (`permissions:` blocks)
- Never commit secrets to the repo
- Use environment protection rules for production deploys

---

## Getting Help

Each workflow is self-contained and well-commented. If you need to customize something:

1. Read the comments in the workflow file
2. Check [GitHub Actions documentation](https://docs.github.com/en/actions)
3. Use [actionlint](https://github.com/rhysd/actionlint) to validate syntax

These templates give you a 90% solution - the last 10% is project-specific and much easier to customize from a working base.
