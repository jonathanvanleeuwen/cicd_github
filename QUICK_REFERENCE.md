# Quick Reference - Common Configurations

Fast lookup for common cicd_github workflow configurations.

---

## 📋 Standard Configurations

### Standard Python Library (Recommended)
```yaml
# CI
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  extra-install-args: '.[dev]'
  # All steps run by default (pre-commit, lint, tests)

# CD
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-name: 'my_library'
  extra-install-args: '.[dev]'
  # All steps run by default (coverage, release, publish)
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### FastAPI Application
```yaml
# CI
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  extra-install-args: '.[dev]'

# CD
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-name: 'my_fastapi_app'
  extra-install-args: '.[dev]'
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Project with Custom Coverage Path
```yaml
# CI
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  extra-install-args: '.[dev]'
  skip-precommit: true  # Skip pre-commit checks

# CD
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-name: 'my_project'
  coverage-path: 'app/src/my_project'  # Custom path
  extra-install-args: '.[dev]'
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Project Skipping Specific Steps
```yaml
# CI
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  extra-install-args: '.[dev]'
  skip-lint: true  # Skip ruff linting

# CD
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-name: 'my_package'
  skip-coverage: true  # Skip coverage report
  skip-publish: true   # Skip wheel publishing
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Minimal Setup (No Dev Dependencies)
```yaml
# CI
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  extra-install-args: ''  # ← No [dev] extras
  skip-precommit: true    # No pre-commit setup

# CD
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-name: 'my_package'
  extra-install-args: ''  # ← No [dev] extras
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

---

## 🎯 Input Quick Reference

### CI Workflow Inputs

| Input | Values | Default | Notes |
|-------|--------|---------|-------|
| `python-version` | `'3.12'`, `'3.11'`, `'3.10'`, etc. | `'3.12'` | String, must be quoted |
| `extra-install-args` | `'.[dev]'`, `''`, etc. | `'.[dev]'` | uv install args |
| `skip-precommit` | `true` or `false` | `false` | Skip pre-commit checks |
| `skip-lint` | `true` or `false` | `false` | Skip ruff linting |
| `skip-tests` | `true` or `false` | `false` | Skip pytest tests |

**Note:** Workflows always use uv package manager and ruff linter.

### CD Workflow Inputs

| Input | Values | Default | Notes |
|-------|--------|---------|-------|
| `python-version` | `'3.12'`, `'3.11'`, etc. | `'3.12'` | String, must be quoted |
| `package-name` | `'my_package'` | **REQUIRED** | Your package dir name |
| `extra-install-args` | `'.[dev]'`, `''`, etc. | `'.[dev]'` | uv install args |
| `coverage-path` | Custom path or `''` | Auto (src/package-name) | Override coverage path |
| `skip-coverage` | `true` or `false` | `false` | Skip coverage report |
| `skip-release` | `true` or `false` | `false` | Skip semantic versioning |
| `skip-publish` | `true` or `false` | `false` | Skip wheel build/publish |

**Note:** Workflows always use uv package manager.

---

## 🔧 Common Customizations

### Using Different Python Versions

**Python 3.11:**
```yaml
with:
  python-version: '3.11'
```

**Python 3.13:**
```yaml
with:
  python-version: '3.13'
```

### Custom Coverage Path

**Default:** Auto-detected as `{package-structure}/{package-name}`

**Custom path:**
```yaml
with:
  package-name: 'my_package'
  coverage-path: 'custom/path/to/package'
```

### Skip Pre-commit Hooks

```yaml
with:
  skip-precommit: true
```

### No Dev Dependencies

```yaml
with:
  extra-install-args: ''
```

### Multiple Extra Dependencies

```yaml
with:
  extra-install-args: '.[dev,test,docs]'
```

---

## 📌 Version Pinning

### Latest (Auto-Update)
```yaml
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@main
```
✅ Always latest features  
⚠️ May introduce breaking changes

### Pinned Version (Recommended)
```yaml
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@v1.2.0
```
✅ Stable, predictable  
✅ Control when to update  
⚠️ Miss out on new features until manually updated

### Commit SHA (Maximum Stability)
```yaml
uses: jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@a1b2c3d4e5f6
```
✅ Immutable, never changes  
⚠️ Hardest to update

---

## 🔐 Required Secrets

### RELEASE_TOKEN

**Required for:** CD workflow (semantic release)  
**Type:** Personal Access Token (PAT)  
**Permissions needed:**
- Contents: Read and write
- Metadata: Read-only (automatic)

**Repository access:**
- Your repository (the one using the workflows)

> **Note:** Since `cicd_github` is public, the PAT doesn't need access to it.

**Setup:**
1. [Create PAT](https://github.com/settings/tokens?type=beta)
2. Repository → Settings → Secrets → Actions
3. Add secret: `RELEASE_TOKEN`

---

## 📝 pyproject.toml Requirements

### Minimal Configuration

```toml
[project]
name = "my_package"
version = "0.1.0"
# ... other project metadata

[tool.semantic_release]
version_toml = ["pyproject.toml:project.version"]
version_variables = ["src/my_package/__init__.py:__version__"]
branch = "main"
upload_to_vcs_release = true
upload_to_pypi = false
build_command = "pip install build && python -m build"
```

**Don't forget:**
- Add `__version__ = "0.1.0"` to `src/my_package/__init__.py`
- Match version number in both places

---

## 📊 Coverage Badge Setup

Add to README.md:

```markdown
# My Project

<!-- Pytest Coverage Comment:Begin -->
<!-- Pytest Coverage Comment:End -->

## Installation
...
```

The CD workflow automatically injects coverage badge between these markers.

---

## 🎨 Cookiecutter Templates

For cookiecutter templates, use Jinja2 variables:

```yaml
with:
  package-name: '{{cookiecutter.project_name}}'
secrets:
  RELEASE_TOKEN: {% raw %}${{ secrets.RELEASE_TOKEN }}{% endraw %}
```

---

## 🔄 Semantic Release Commit Types

| Commit Prefix | Version Bump | Example |
|---------------|--------------|---------|
| `fix:` | Patch (0.0.X) | `fix: resolve auth bug` |
| `feat:` | Minor (0.X.0) | `feat: add CSV export` |
| `feat:` + `BREAKING CHANGE:` | Major (X.0.0) | See below |
| `docs:`, `chore:`, etc. | None | `docs: fix typo` |

**Breaking change example:**
```
feat: redesign API

BREAKING CHANGE: removed v1 endpoints
```

---

## 🆘 Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| Can't find workflow | Verify workflow path: `jonathanvanleeuwen/cicd_github/.github/workflows/reusable-*.yml@main` |
| Can't push to main | Check PAT has Contents: Read and write permission |
| Coverage shows 0% | Verify `package-name` matches directory name exactly, or set `coverage-path` |
| No version tag created | Use semantic commit messages (`fix:`, `feat:`) |
| Pre-commit fails | Run `pre-commit run --all-files` locally and fix issues, or set `skip-precommit: true` |
| Wrong Python version | Update `python-version` input |
| Want to skip a step | Use `skip-precommit`, `skip-lint`, `skip-tests`, `skip-coverage`, `skip-release`, or `skip-publish` |

---

## 📚 Full Documentation

- [Complete Setup Guide](SETUP_CHECKLIST.md)
- [Migration from Local Workflows](MIGRATION_GUIDE.md)
- [Detailed README](README.md)
- [Contributing Guidelines](CONTRIBUTING.md)

---

## 💡 Pro Tips

1. **Pin to versions** for production repos
2. **Use @main** for development/testing
3. **Keep PAT expiration calendar reminders**
4. **Test on non-critical repo first**
5. **Update workflows in batches** (3-5 repos at a time)
6. **Document repo-specific configurations** if variations exist

---

**Quick copy:** `jonathanvanleeuwen/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0`

**Remember to replace:** `YOUR_PACKAGE_NAME`, `v1.0.0` (version is optional)
