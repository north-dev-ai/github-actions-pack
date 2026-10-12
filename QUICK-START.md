# Quick Start Guide

Get up and running in 5 minutes.

## Step 1: Choose Your Workflows

Pick the workflows that match your project:

**For Node.js projects:**
- ✅ `nodejs-ci-cd.yml` - Testing, building, deploying
- ✅ `security-scan.yml` - Vulnerability scanning
- ✅ `pr-checks.yml` - PR quality enforcement

**For Python projects:**
- ✅ `python-ci.yml` - Testing and PyPI publishing
- ✅ `security-scan.yml` - Vulnerability scanning
- ✅ `pr-checks.yml` - PR quality enforcement

**For Docker projects:**
- ✅ `docker-build.yml` - Build and push images
- ✅ `security-scan.yml` - Container scanning

**For monorepos:**
- ✅ `monorepo-ci.yml` - Smart selective testing
- ✅ `security-scan.yml` - Security for all packages

**For any project:**
- ✅ `auto-release.yml` - Automated versioning
- ✅ `cleanup-tasks.yml` - Keep repo clean

## Step 2: Copy Files

```bash
# Create workflows directory if it doesn't exist
mkdir -p .github/workflows

# Copy the workflows you need
cp github-actions-pack/.github/workflows/nodejs-ci-cd.yml .github/workflows/
cp github-actions-pack/.github/workflows/security-scan.yml .github/workflows/
cp github-actions-pack/.github/workflows/pr-checks.yml .github/workflows/
```

## Step 3: Configure Secrets

Add secrets in your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**

**Required secrets per workflow:**

| Workflow | Secret Name | Where to Get It | Required? |
|----------|-------------|-----------------|-----------|
| nodejs-ci-cd.yml | `CODECOV_TOKEN` | codecov.io | Optional |
| nodejs-ci-cd.yml | `DEPLOY_TOKEN` | Your hosting provider | If deploying |
| python-ci.yml | `PYPI_TOKEN` | pypi.org account | If publishing |
| auto-release.yml | `NPM_TOKEN` | npmjs.com | If publishing |

**Note:** `GITHUB_TOKEN` is automatic - you don't need to add it!

## Step 4: Adjust for Your Project (Optional)

Most workflows work out-of-the-box, but you might want to:

**Change branch names:**
```yaml
# If you use "master" instead of "main"
on:
  push:
    branches: [ master, develop ]
```

**Change Node/Python versions:**
```yaml
strategy:
  matrix:
    node-version: [20.x, 22.x]  # Remove 18.x if you don't need it
```

**Adjust monorepo paths:**
```yaml
filters: |
  frontend:
    - 'apps/web/**'     # Change to your actual paths
  backend:
    - 'apps/api/**'     # Change to your actual paths
```

## Step 5: Commit and Push

```bash
git add .github/workflows/
git commit -m "Add CI/CD workflows"
git push
```

**That's it!** Go to the **Actions** tab in your repo to see workflows running.

## Verification Checklist

After your first push:

- [ ] Actions tab shows workflow runs
- [ ] Tests are passing (green checkmarks)
- [ ] PR checks appear on pull requests
- [ ] Security tab shows scanning results
- [ ] No error messages in workflow logs

## Troubleshooting

**Workflow didn't trigger?**
- Check the `on:` section matches your branch name
- Ensure you pushed to the correct branch

**Tests failing?**
- Verify required npm scripts exist (`lint`, `test`, `build`)
- Check that dependencies install correctly
- Look at the detailed logs in the Actions tab

**Missing secrets error?**
- Add the secret in Settings → Secrets
- Redeploy by pushing a new commit

**Syntax error?**
- Workflow files must be valid YAML
- Check indentation (use spaces, not tabs)
- Use a YAML validator if needed

## Next Steps

- Read `README.md` for detailed feature descriptions
- Check `USAGE.md` for advanced customization
- Monitor the Actions tab for workflow results
- Refine workflows based on your project's needs

## Common Patterns

**Start small, add more later:**
```bash
# Week 1: Just CI
cp nodejs-ci-cd.yml .github/workflows/

# Week 2: Add security
cp security-scan.yml .github/workflows/

# Week 3: Add automation
cp auto-release.yml .github/workflows/
cp pr-checks.yml .github/workflows/
```

**Full production setup:**
```bash
# Everything you need for a serious project
cp nodejs-ci-cd.yml .github/workflows/
cp security-scan.yml .github/workflows/
cp pr-checks.yml .github/workflows/
cp auto-release.yml .github/workflows/
cp cleanup-tasks.yml .github/workflows/
```

---

**Questions?** All workflows are well-commented. Read the workflow file itself for inline documentation.
