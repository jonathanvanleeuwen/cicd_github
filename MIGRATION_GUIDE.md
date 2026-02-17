# Migration Guide - Moving Existing Repositories to cicd_github Workflows

This guide helps you migrate existing repositories from local workflows to the centralized reusable workflows in cicd_github.

## Overview

**Before:** Each repository has its own `.github/workflows/` files with duplicated logic.  
**After:** Each repository calls reusable workflows from cicd_github with configuration inputs.

**Benefits:**
- ✅ Future updates apply to all repos instantly
- ✅ Consistent CI/CD across all projects
- ✅ Less maintenance overhead
- ✅ Version pinning for stability

---

## Pre-Migration Checklist

- [ ] Backup existing workflows (commit to a branch named `backup/pre-cicd-migration`)
- [ ] Review current workflow configuration and identify inputs needed
- [ ] Ensure you have a PAT that includes the cicd_github repository access
- [ ] Test migration on a non-critical repository first

---

## Migration Steps

### Step 1: Backup Current Workflows

```bash
git checkout -b backup/pre-cicd-migration
git push origin backup/pre-cicd-migration
git checkout main
```

### Step 2: Identify Your Current Configuration

Review your existing workflows and note:

**From `python-app.yml` (CI workflow):**
```yaml
# What Python version?
python-version: "3.12"  # or "3.11", etc.

# What commands for installation?
pip install -e ".[dev]"  # → package-manager: 'pip', extra-install-args: '.[dev]'
uv pip install --system -e ".[dev]"  # package-manager: 'uv', extra-install-args: '.[dev]'

# What linter?
ruff check .  # → linter: 'ruff'
flake8 .      # → linter: 'flake8'

# Package structure?
src/my_package/  # → package-structure: 'src'
app/           # → package-structure: 'app'

# Pre-commit?
pre-commit run --all-files  # → run-precommit: true
# (absent)                   # → run-precommit: false
```

**From `semantic-release.yml` (CD workflow):**
```yaml
# What is your package name?
--cov=src/my_package  # → package-name: 'my_package'
--cov=app/src/legacy  # → package-name: 'legacy', coverage-path: 'app/src/legacy'

# What secret name for PAT?
secrets.RELEASE_TOKEN  # ✅ Already correct
secrets.SEM_RELEASE    # → Need to add RELEASE_TOKEN secret (or update cicd_github caller)
```

### Step 3: Update or Add PAT Secret

#### Option A: Your Repository Uses `RELEASE_TOKEN` (✅ No Change Needed)

If your existing workflow already uses `secrets.RELEASE_TOKEN`, no changes needed.

#### Option B: Your Repository Uses Different Secret Name (e.g., `SEM_RELEASE`)

**Recommended:** Add `RELEASE_TOKEN` as an additional secret pointing to same PAT:

1. Go to repository → Settings → Secrets and variables → Actions
2. Note the value of your existing secret (you may need to regenerate it)
3. Add new secret: `RELEASE_TOKEN` with same value
4. Keep old secret for backwards compatibility during migration

**Alternative:** Modify the caller workflow to use your existing secret name (less recommended).

#### Option C: No PAT Secret Yet

Follow [SETUP_CHECKLIST.md](SETUP_CHECKLIST.md) Step 1-2 to create and add the PAT.

#### Option D: PAT Doesn't Include cicd_github Repository

If your PAT exists but doesn't have access to cicd_github (private repo):

1. Go to [GitHub Settings → Personal access tokens](https://github.com/settings/tokens)
2. Find your existing PAT (e.g., `RELEASE_TOKEN_YOUR_REPO`)
3. Click "Regenerate token"
4. Update "Repository access" to include cicd_github
5. Copy new token value
6. Update secret in your repository with new value

### Step 4: Replace CI Workflow

**Delete old file:**
```bash
git rm .github/workflows/python-app.yml
```

**Create new file:** `.github/workflows/python-app.yml`

```yaml
name: Python Application CI

on:
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  ci:
    uses: YOUR_GITHUB_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@main
    with:
      python-version: '3.12'           # ← Your Python version
      package-manager: 'pip'            # ← 'pip' or 'uv'
      linter: 'ruff'                    # ← 'ruff' or 'flake8'
      package-structure: 'src'          # ← 'src' or 'app'
      run-precommit: true               # ← true or false
      extra-install-args: '.[dev]'      # ← Your install args
```

**Commit:**
```bash
git add .github/workflows/python-app.yml
git commit -m "ci: migrate to reusable CI workflow from cicd_github"
```

### Step 5: Replace CD Workflow

**Delete old file:**
```bash
git rm .github/workflows/semantic-release.yml
```

**Create new file:** `.github/workflows/semantic-release.yml`

```yaml
name: Release Pipeline - Coverage, Versioning & Publishing

on:
  pull_request:
    branches:
      - main
    types: [closed]

permissions:
  contents: write
  pull-requests: read

jobs:
  cd:
    uses: YOUR_GITHUB_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@main
    with:
      python-version: '3.12'           # ← Your Python version
      package-manager: 'pip'            # ← 'pip' or 'uv'
      package-structure: 'src'          # ← 'src' or 'app'
      package-name: 'YOUR_PACKAGE_NAME' # ← REQUIRED: Your package name
      extra-install-args: '.[dev]'      # ← Your install args
      # coverage-path: 'custom/path'   # ← Uncomment if needed
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

**Commit:**
```bash
git add .github/workflows/semantic-release.yml
git commit -m "ci: migrate to reusable CD workflow from cicd_github"
```

### Step 6: Update Branch Protection Rules

If status check names changed, update branch protection:

1. Go to repository → Settings → Rules → Rulesets
2. Click your "Protect main" ruleset
3. Under "Require status checks to pass":
   - Remove old check names if they're different
   - Add new check names:
     - `Run Pre-commit Checks` (if `run-precommit: true`)
     - `Run Tests and Lint`
4. Save changes

### Step 7: Test Migration

**Create test PR:**
```bash
git checkout -b test/cicd-migration
echo "# Test migration" >> README.md
git add README.md
git commit -m "chore: test migrated CI/CD workflows"
git push origin test/cicd-migration
```

**Verify:**
1. Create PR on GitHub
2. Check CI workflow runs (should see new workflow name)
3. Verify all checks pass
4. Merge PR
5. Verify CD workflow runs
6. Check that coverage, version tag, and release are created

### Step 8: Pin to Version (Recommended for Production)

Once migration is validated, pin workflows to a specific version:

```yaml
# Instead of @main
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@main

# Use version tag
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
```

This prevents unexpected breaking changes from cicd_github updates.

---

## Migration Mapping Table

Quick reference for translating old workflow syntax to new inputs:

| Old Workflow Feature | New Input Configuration |
|---------------------|------------------------|
| `pip install -e ".[dev]"` | `package-manager: 'pip'`<br>`extra-install-args: '.[dev]'` |
| `uv pip install --system -e ".[dev]"` | `package-manager: 'uv'`<br>`extra-install-args: '.[dev]'` |
| `ruff check .` | `linter: 'ruff'` |
| `flake8 . --count...` | `linter: 'flake8'` |
| `pre-commit run --all-files` | `run-precommit: true` |
| `src/my_package/` structure | `package-structure: 'src'`<br>`package-name: 'my_package'` |
| `app/` structure | `package-structure: 'app'`<br>`package-name: 'your_package'` |
| `--cov=src/my_package` | `package-name: 'my_package'` (auto-detected) |
| `--cov=custom/path` | `coverage-path: 'custom/path'` |
| `secrets.SEM_RELEASE` | Add `secrets.RELEASE_TOKEN` pointing to same PAT |

---

## Troubleshooting Migration

### Workflow Can't Find Reusable Workflow

**Error:** `workflow was not found`

**Solution:**
- Check `YOUR_GITHUB_USERNAME` is correct
- Verify PAT includes cicd_github repository access
- Verify PAT has "Workflows: Read" permission
- Check workflow path is exactly: `.github/workflows/reusable-ci.yml`

### Status Checks Don't Appear

**Error:** Branch protection looking for old check names

**Solution:**
- Update branch protection ruleset with new check names
- Check names are visible in the Actions tab of a recent PR

### CD Workflow Fails to Push

**Error:** `protected branch` or `permission denied`

**Solution:**
- Verify `RELEASE_TOKEN` secret exists and is correct
- Ensure PAT has "Contents: Read and write" permission
- Check PAT hasn't expired (90-day expiration)

### Coverage Path Wrong

**Error:** Coverage report shows 0% or wrong modules

**Solution:**
- Check `package-name` matches your package directory name exactly
- For custom paths, explicitly set `coverage-path` input
- Verify path with: `find . -name "__init__.py" -path "*/my_package/*"`

### Different Python Version Needed

Just update the input:
```yaml
with:
  python-version: '3.11'  # or '3.10', '3.13', etc.
```

---

## Rollback Procedure

If migration causes issues:

```bash
# Restore backup
git checkout backup/pre-cicd-migration .github/workflows/

# Commit restoration
git add .github/workflows/
git commit -m "revert: rollback to pre-migration workflows"
git push origin main
```

Review issues, fix configuration, and try migration again.

---

## Post-Migration Cleanup

After successful migration (wait a few days to be sure):

```bash
# Delete backup branch
git push origin --delete backup/pre-cicd-migration
git branch -d backup/pre-cicd-migration

# Optional: Remove old secret if renamed
# (Only if you added RELEASE_TOKEN and no longer need old name)
# Go to Settings → Secrets and delete old secret
```

---

## Migration Checklist

Use this checklist for each repository:

- [ ] Backup current workflows to branch
- [ ] Identify current configuration (Python version, package manager, etc.)
- [ ] Ensure PAT has access to cicd_github repository
- [ ] Add/update RELEASE_TOKEN secret if needed
- [ ] Replace CI workflow (python-app.yml)
- [ ] Replace CD workflow (semantic-release.yml)
- [ ] Update branch protection rules
- [ ] Test with a PR
- [ ] Verify CD workflow on merge
- [ ] Pin to version tag (optional but recommended)
- [ ] Delete backup branch after validation period

---

## Migrating Multiple Repositories

For bulk migration across many repositories:

1. **Start with one test repository** - validate the process
2. **Document repo-specific configurations** - if variations exist
3. **Migrate repositories in batches** - 3-5 at a time
4. **Monitor first few workflows** - catch issues early
5. **Update documentation** - note any special cases discovered

**Estimation:** ~15-30 minutes per repository (first time), ~5-10 minutes after learning the process.

---

## Getting Help

- Check [README.md Troubleshooting](README.md#-troubleshooting)
- Review [SETUP_CHECKLIST.md](SETUP_CHECKLIST.md) for detailed steps
- Open an issue in cicd_github repository
- Reference workflow logs when asking for help

---

**Remember:** Migration is one-time effort with long-term benefits! 🚀
