# Quick Reference - Common Configurations

Fast lookup for common cicd_github workflow configurations.

---

## 📋 Standard Configurations

### Modern Python Library (Recommended)
```yaml
# CI
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  linter: 'ruff'
  package-structure: 'src'
  run-precommit: true
  extra-install-args: '.[dev]'

# CD
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  package-structure: 'src'
  package-name: 'my_library'
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### FastAPI Application
```yaml
# CI
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  linter: 'ruff'
  package-structure: 'src'
  run-precommit: true
  extra-install-args: '.[dev]'

# CD
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  package-structure: 'src'
  package-name: 'my_fastapi_app'
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Legacy Project (App Structure + Flake8)
```yaml
# CI
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  linter: 'flake8'
  package-structure: 'app'
  run-precommit: false
  extra-install-args: ''

# CD
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  package-structure: 'app'
  package-name: 'legacy_project'
  coverage-path: 'app/src/legacy_project'
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Project Using UV Package Manager
```yaml
# CI
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'uv'  # ← Changed to uv
  linter: 'ruff'
  package-structure: 'src'
  run-precommit: true
  extra-install-args: '.[dev]'

# CD
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'uv'  # ← Changed to uv
  package-structure: 'src'
  package-name: 'my_package'
secrets:
  RELEASE_TOKEN: ${{ secrets.RELEASE_TOKEN }}
```

### Minimal Setup (No Dev Dependencies)
```yaml
# CI
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  linter: 'ruff'
  package-structure: 'src'
  run-precommit: false
  extra-install-args: ''  # ← No [dev] extras

# CD
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-cd.yml@v1.0.0
with:
  python-version: '3.12'
  package-manager: 'pip'
  package-structure: 'src'
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
| `package-manager` | `'pip'` or `'uv'` | `'pip'` | Package installer |
| `linter` | `'ruff'` or `'flake8'` | `'ruff'` | Code quality tool |
| `package-structure` | `'src'` or `'app'` | `'src'` | Project layout |
| `run-precommit` | `true` or `false` | `true` | Run pre-commit hooks |
| `extra-install-args` | `'.[dev]'`, `''`, etc. | `'.[dev]'` | pip/uv install args |

### CD Workflow Inputs

| Input | Values | Default | Notes |
|-------|--------|---------|-------|
| `python-version` | `'3.12'`, `'3.11'`, etc. | `'3.12'` | String, must be quoted |
| `package-manager` | `'pip'` or `'uv'` | `'pip'` | Package installer |
| `package-structure` | `'src'` or `'app'` | `'src'` | Project layout |
| `package-name` | `'my_package'` | **REQUIRED** | Your package dir name |
| `extra-install-args` | `'.[dev]'`, `''`, etc. | `'.[dev]'` | pip/uv install args |
| `coverage-path` | Custom path or `''` | Auto-detected | Override coverage path |

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

### No Pre-commit Hooks

```yaml
with:
  run-precommit: false
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
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@main
```
✅ Always latest features  
⚠️ May introduce breaking changes

### Pinned Version (Recommended)
```yaml
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.2.0
```
✅ Stable, predictable  
✅ Control when to update  
⚠️ Miss out on new features until manually updated

### Commit SHA (Maximum Stability)
```yaml
uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@a1b2c3d4e5f6
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
| Coverage shows 0% | Verify `package-name` matches directory name exactly |
| No version tag created | Use semantic commit messages (`fix:`, `feat:`) |
| Pre-commit fails | Run `pre-commit run --all-files` locally and fix issues |
| Wrong Python version | Update `python-version` input |

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

**Quick copy:** `YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@v1.0.0`

**Remember to replace:** `YOUR_USERNAME`, `YOUR_PACKAGE_NAME`, `v1.0.0`
