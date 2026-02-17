# cicd_github

**Reusable GitHub Actions workflows for Python projects - DRY CI/CD at scale**

A centralized public repository of reusable GitHub Actions workflows that eliminates duplicate CI/CD configuration across multiple Python projects. Update once, benefit everywhere.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 🎯 Why This Repository?

**Problem:** Maintaining identical CI/CD workflows across multiple repositories is tedious and error-prone. Every improvement requires updating N repositories.

**Solution:** Reusable workflows in a single source of truth. Reference versioned workflows from any repository, update once, and all projects benefit.

**Benefits:**
- ✅ **DRY Principle**: Single source of truth for all CI/CD logic
- ✅ **Consistency**: Same quality gates across all projects
- ✅ **Versioning**: Pin workflows to specific versions for stability
- ✅ **Easy Updates**: Improve once, deploy everywhere
- ✅ **Flexibility**: Configurable via inputs to support different project structures

---

## 📋 Features

### CI Workflow (`reusable-ci.yml`)
Validates code quality on every pull request:
- ✅ Pre-commit hooks (ruff, trailing-whitespace, end-of-file-fixer, etc.)
- ✅ Linting with ruff (modern) or flake8 (legacy)
- ✅ Testing with pytest
- ✅ Configurable Python version, package manager (pip/uv), and package structure (src/app)

### CD Workflow (`reusable-cd.yml`)
Automates releases when PRs are merged to main:
- ✅ **Coverage Report**: Generates pytest coverage and commits badge to README
- ✅ **Semantic Versioning**: Analyzes commit messages to determine version bump
  - `fix:` → patch (0.0.X)
  - `feat:` → minor (0.X.0)
  - `BREAKING CHANGE:` → major (X.0.0)
- ✅ **Build & Publish**: Creates wheel and publishes to GitHub Releases
- ✅ **Automatic Tagging**: Tags releases in git for version pinning

---

## 🚀 Quick Start

### Step 1: Add Workflows to Your Repository

Copy the example workflows from [`examples/`](examples/) to your repository's `.github/workflows/` directory:

**For CI (runs on PRs):**
```yaml
# .github/workflows/python-app.yml
name: Python Application CI

on:
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  ci:
    uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@main
    with:
      python-version: '3.12'
      package-manager: 'pip'           # or 'uv'
      linter: 'ruff'                   # or 'flake8'
      package-structure: 'src'         # or 'app'
      run-precommit: true
      extra-install-args: '.[dev]'
```

**For CD (runs on merge):**
```yaml
# .github/workflows/semantic-release.yml
name: Release Pipeline

on:
  pull_request:
    branches: [ "main" ]
    types: [closed]

permissions:
  contents: write
  pull-requests: read

jobs:
  cd:
    uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-cd.yml@main
    with:
      python-version: '3.12'
      package-manager: 'pip'
      package-structure: 'src'
      package-name: 'my_package_name'  # REQUIRED
      extra-install-args: '.[dev]'
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Step 2: Configure Repository Secrets

The CD workflow requires a Personal Access Token (PAT) to push commits, tags, and create releases.

#### Create a Fine-Grained PAT

1. Go to [GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens](https://github.com/settings/tokens?type=beta)
2. Click **"Generate new token"**
3. Configure:
   - **Token name:** `RELEASE_TOKEN_YOUR_REPO_NAME`
   - **Expiration:** 90 days (recommended - set a reminder to rotate)
   - **Repository access:** "Only select repositories"
     - ✅ Select YOUR repository (the one using workflows)
   - **Permissions:**
     - **Contents:** Read and write (push commits, tags, releases)
     - **Metadata:** Read-only (automatically selected)
4. Click **"Generate token"** and **COPY IT IMMEDIATELY** (you won't see it again!)

> **Note:** Since this repository (`cicd_github`) is public, you don't need to grant your PAT access to it. The PAT only needs access to your own repository where you're using the workflows.

#### Add Token to Repository

1. Go to your repository → **Settings → Secrets and variables → Actions**
2. Click **"New repository secret"**
3. Configure:
   - **Name:** `RELEASE_TOKEN`
   - **Secret:** Paste your copied PAT
4. Click **"Add secret"**

### Step 3: Configure Your Project

#### Add Coverage Markers to README.md

Add these HTML comments to your `README.md` where you want the coverage badge to appear:

```markdown
# My Awesome Project

<!-- Pytest Coverage Comment:Begin -->
<!-- Pytest Coverage Comment:End -->

## Installation
...
```

The CD workflow will automatically inject the coverage badge between these markers.

#### Configure pyproject.toml for Semantic Release

Add this configuration to your `pyproject.toml`:

```toml
[tool.semantic_release]
version_toml = ["pyproject.toml:project.version"]
version_variables = ["src/my_package/__init__.py:__version__"]
branch = "main"
upload_to_vcs_release = true
upload_to_pypi = false
build_command = "pip install build && python -m build"
```

> **Note:** Replace `my_package` with your actual package name!

#### Add Version to Your Package

Ensure your package has a `__version__` variable:

```python
# src/my_package/__init__.py
__version__ = "0.1.0"
```

And in `pyproject.toml`:

```toml
[project]
name = "my_package"
version = "0.1.0"  # semantic-release will auto-update this
```

### Step 4: Protect Your Main Branch

Configure branch protection to require PR reviews and status checks:

1. Go to your repository → **Settings → Rules → Rulesets**
2. Click **"New ruleset"** → **"New branch ruleset"**
3. Configure:
   - **Ruleset name:** `Protect main`
   - **Enforcement status:** Active
   - **Target branches:** Include pattern `main`
   - **Rules:**
     - ✅ Restrict deletions
     - ✅ Require pull request before merging (1+ approvals)
     - ✅ Require status checks to pass
       - Add checks: `Run Pre-commit Checks`, `Run Tests and Lint`
     - ✅ Block force pushes
4. Click **"Create"**

### Step 5: Start Using It!

**Create a feature branch and make changes:**
```bash
git checkout -b feature/my-new-feature
# Make your changes
git add .
git commit -m "feat: add amazing new feature"
git push origin feature/my-new-feature
```

**Create a Pull Request:**
- The CI workflow will automatically run pre-commit checks, linting, and tests
- Get your PR reviewed and approved
- Merge to main

**Automatic Release:**
- Upon merge, the CD workflow automatically:
  1. Generates and commits coverage report to README
  2. Analyzes commit messages and creates a new semantic version tag
  3. Builds a wheel package
  4. Publishes the wheel to GitHub Releases

---

## 📚 Configuration Reference

### CI Workflow Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `python-version` | string | `'3.12'` | Python version to use |
| `package-manager` | string | `'pip'` | Package manager: `'pip'` or `'uv'` |
| `linter` | string | `'ruff'` | Linter: `'ruff'` or `'flake8'` |
| `package-structure` | string | `'src'` | Package structure: `'src'` or `'app'` |
| `run-precommit` | boolean | `true` | Whether to run pre-commit hooks |
| `extra-install-args` | string | `'.[dev]'` | Extra install args (e.g., `'.[dev]'` or `''`) |

### CD Workflow Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `python-version` | string | `'3.12'` | Python version to use |
| `package-manager` | string | `'pip'` | Package manager: `'pip'` or `'uv'` |
| `package-structure` | string | `'src'` | Package structure: `'src'` or `'app'` |
| `package-name` | string | **REQUIRED** | Name of package (for coverage and wheel) |
| `extra-install-args` | string | `'.[dev]'` | Extra install args (e.g., `'.[dev]'` or `''`) |
| `coverage-path` | string | `''` | Override coverage path (default: auto-detected) |

### CD Workflow Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `RELEASE_TOKEN` | **Yes** | PAT with repo/contents/workflow permissions |

---

## 🎨 Usage Examples

### Example 1: Modern Python Library

```yaml
# .github/workflows/python-app.yml
jobs:
  ci:
    uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
    with:
      python-version: '3.12'
      package-manager: 'pip'
      linter: 'ruff'
      package-structure: 'src'
      run-precommit: true
      extra-install-args: '.[dev]'

# .github/workflows/semantic-release.yml
jobs:
  cd:
    uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
    with:
      package-name: 'my_library'
      python-version: '3.12'
      package-manager: 'pip'
      package-structure: 'src'
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Example 2: FastAPI Application with UV

```yaml
# .github/workflows/python-app.yml
jobs:
  ci:
    uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@main
    with:
      python-version: '3.12'
      package-manager: 'uv'
      linter: 'ruff'
      package-structure: 'src'
      run-precommit: true
      extra-install-args: '.[dev]'

# .github/workflows/semantic-release.yml
jobs:
  cd:
    uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@main
    with:
      package-name: 'my_fastapi_app'
      python-version: '3.12'
      package-manager: 'uv'
      package-structure: 'src'
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Example 3: Legacy Project (App Structure, Flake8)

```yaml
# .github/workflows/python-app.yml
jobs:
  ci:
    uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@main
    with:
      python-version: '3.12'
      package-manager: 'pip'
      linter: 'flake8'
      package-structure: 'app'
      run-precommit: false  # No pre-commit setup yet
      extra-install-args: ''

# .github/workflows/semantic-release.yml
jobs:
  cd:
    uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@main
    with:
      package-name: 'legacy_project'
      python-version: '3.12'
      package-structure: 'app'
      coverage-path: 'app/src/legacy_project'  # Custom coverage path
    secrets:
      RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Example 4: Cookiecutter Template

For cookiecutter templates, use Jinja2 template variables:

```yaml
# {{cookiecutter.project_name}}/.github/workflows/semantic-release.yml
jobs:
  cd:
    uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@main
    with:
      package-name: '{{cookiecutter.project_name}}'
      python-version: '3.12'
      package-manager: 'pip'
      package-structure: 'src'
    secrets:
      RELEASE_TOKEN: {% raw %}${{ secrets.RELEASE_TOKEN }}{% endraw %}
```

---

## 🔒 Security & Best Practices

### Token Security

- **Never commit tokens to git** - always use repository secrets
- **Use fine-grained PATs** instead of classic tokens (better security controls)
- **Rotate tokens every 90 days** - set calendar reminders
- **Limit repository access** - only grant access to required repositories
- **Review audit logs** - periodically check token usage in GitHub audit logs

### Branch Protection

Always protect your `main` branch to:
- Require PR reviews before merging
- Require status checks to pass (CI workflow)
- Block direct pushes (force PR workflow)
- Prevent force pushes and branch deletion

The PAT allows the workflow to bypass these rules while humans must use PRs.

### Workflow Versioning

**Recommendation:** Pin workflows to specific versions for production repositories:

```yaml
# ✅ GOOD - Pinned to version tag (stable)
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.2.0

# ⚠️ CAUTION - Uses main branch (auto-updates, may break)
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@main

# ✅ ALSO GOOD - Uses commit SHA (immutable)
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@a1b2c3d
```

For development/testing, `@main` is fine. For production, use versioned releases.

---

## 📦 Semantic Release Commit Conventions

The CD workflow uses commit messages to determine version bumps. Follow these conventions:

| Commit Type | Version Bump | Example |
|-------------|--------------|---------|
| `fix: ...` | Patch (0.0.X) | `fix: resolve authentication bug` |
| `feat: ...` | Minor (0.X.0) | `feat: add user profile endpoint` |
| `feat: ...\n\nBREAKING CHANGE: ...` | Major (X.0.0) | `feat: redesign API\n\nBREAKING CHANGE: removed v1 endpoints` |
| `docs: ...`, `chore: ...`, etc. | No release | `docs: update README examples` |

### Examples

**Patch Release (Bug Fix):**
```bash
git commit -m "fix: correct validation logic in user registration"
# Version: 1.2.3 → 1.2.4
```

**Minor Release (New Feature):**
```bash
git commit -m "feat: add export to CSV functionality"
# Version: 1.2.4 → 1.3.0
```

**Major Release (Breaking Change):**
```bash
git commit -m "feat: redesign authentication system

BREAKING CHANGE: OAuth2 replaces API keys. Existing API keys will no longer work."
# Version: 1.3.0 → 2.0.0
```

**No Release:**
```bash
git commit -m "docs: fix typo in installation guide"
# No version change
```

---

## 🔧 Troubleshooting

### CI Workflow Fails with "Permission Denied"

**Problem:** Workflow can't access reusable workflow file.

**Solution:**
- Check the workflow path is correct: `jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@main`
- Ensure you're referencing the correct branch or tag (e.g., `@main` or `@v1.0.0`)
- Verify your repository has internet access (for self-hosted runners)

### CD Workflow Can't Push to Main

**Problem:** Workflow fails with "protected branch" error.

**Solution:**
- Verify `RELEASE_TOKEN` secret exists and is correct
- Ensure the PAT has `Contents: Read and write` permission
- Check branch protection rules allow the PAT to push

### Coverage Badge Not Updating

**Problem:** Coverage report workflow succeeds but README doesn't update.

**Solution:**
- Verify README contains the coverage markers:
  ```markdown
  <!-- Pytest Coverage Comment:Begin -->
  <!-- Pytest Coverage Comment:End -->
  ```
- Check the workflow logs for sed command errors
- Ensure the PAT has write access to the repository

### Semantic Release Creates No Version Tag

**Problem:** Workflow runs but no version tag is created.

**Solution:**
- Check commit messages follow semantic release conventions (`fix:`, `feat:`, etc.)
- Verify `pyproject.toml` has `[tool.semantic_release]` configuration
- Check workflow logs for semantic-release output
- Ensure commits since last release include versioned types (not just `docs:` or `chore:`)

### Wheel Build Fails

**Problem:** Wheel build step fails with module not found.

**Solution:**
- Verify `pyproject.toml` has correct `[project]` configuration
- Check `package-name` input matches actual package directory name
- Ensure `__init__.py` exists in package directory
- Verify `__version__` variable is defined in `__init__.py`

### Pre-commit Checks Fail

**Problem:** Pre-commit job fails with hook errors.

**Solution:**
- Run `pre-commit run --all-files` locally to see errors
- Fix issues reported by hooks (trailing whitespace, ruff format, etc.)
- Add `.pre-commit-config.yaml` to your repository if missing
- Consider disabling pre-commit in workflow: `run-precommit: false`

---

## 🛠️ Development & Contributing

### Testing Workflow Changes

To test changes to reusable workflows:

1. Create a test repository
2. Reference your development branch:
   ```yaml
   uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@your-dev-branch
   ```
3. Open a PR in the test repository
4. Verify workflows execute correctly
5. Iterate until working

### Making cicd_github Self-Referencing

Once workflows are stable, `cicd_github` itself should use its own workflows:

```yaml
# cicd_github/.github/workflows/ci.yml
jobs:
  ci:
    uses: ./.github/workflows/reusable-ci.yml  # Local reference
    with:
      package-name: 'cicd_github'
      # ... other inputs
```

This ensures the workflows are battle-tested on themselves!

---

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

This project consolidates CI/CD patterns used across:
- `fastapi_template` - FastAPI application template
- `lib_template` - Python library template
- `queue_worker_template` - Queue worker application template
- `document_search`, `document_agent`, `document_frontend` - Production applications

Built with ❤️ to eliminate repetitive workflow maintenance across multiple repositories.

---

## 📞 Support

- **Issues:** [GitHub Issues](https://github.com/YOUR_USERNAME/cicd_github/issues)
- **Discussions:** [GitHub Discussions](https://github.com/YOUR_USERNAME/cicd_github/discussions)

---

**Remember:** Update once, benefit everywhere! 🚀
