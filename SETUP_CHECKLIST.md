# New Repository Setup Checklist

Use this checklist when setting up a new repository to use the cicd_github workflows.

## Prerequisites

- [ ] Repository created on GitHub
- [ ] Local repository initialized and connected to GitHub
- [ ] Project has `pyproject.toml` with `[project]` section
- [ ] Project has test suite (pytest)

## Step 1: Create Personal Access Token (PAT)

- [ ] Go to [GitHub Settings → Personal access tokens → Fine-grained tokens](https://github.com/settings/tokens?type=beta)
- [ ] Click "Generate new token"
- [ ] Configure token:
  - [ ] Name: `RELEASE_TOKEN_YOUR_REPO_NAME`
  - [ ] Expiration: 90 days (set reminder to rotate)
  - [ ] Repository access: Select your repository + cicd_github
  - [ ] Permissions:
    - [ ] Contents: Read and write
    - [ ] Workflows: Read
    - [ ] Metadata: Read-only (automatic)
- [ ] Generate and copy token immediately
- [ ] Store token securely (password manager)

## Step 2: Add Token to Repository Secrets

- [ ] Go to your repository → Settings → Secrets and variables → Actions
- [ ] Click "New repository secret"
- [ ] Name: `RELEASE_TOKEN`
- [ ] Paste token value
- [ ] Add secret

## Step 3: Add Workflows to Repository

### CI Workflow

- [ ] Create file: `.github/workflows/python-app.yml`
- [ ] Copy content from [examples/caller-ci.yml](examples/caller-ci.yml)
- [ ] Replace `YOUR_GITHUB_USERNAME` with your username
- [ ] Configure inputs:
  - [ ] `python-version`: Your Python version (e.g., `'3.12'`)
  - [ ] `package-manager`: `'pip'` or `'uv'`
  - [ ] `linter`: `'ruff'` or `'flake8'`
  - [ ] `package-structure`: `'src'` or `'app'`
  - [ ] `run-precommit`: `true` or `false`
  - [ ] `extra-install-args`: `'.[dev]'` or `''`
- [ ] Commit and push

### CD Workflow

- [ ] Create file: `.github/workflows/semantic-release.yml`
- [ ] Copy content from [examples/caller-cd.yml](examples/caller-cd.yml)
- [ ] Replace `YOUR_GITHUB_USERNAME` with your username
- [ ] Replace `YOUR_PACKAGE_NAME` with your package name
- [ ] Configure inputs (same as CI workflow, plus):
  - [ ] `package-name`: Your package directory name (REQUIRED)
  - [ ] `coverage-path`: Custom path if needed (optional)
- [ ] Commit and push

## Step 4: Configure Project for Semantic Release

### Add pyproject.toml Configuration

Add this to `pyproject.toml`:

```toml
[tool.semantic_release]
version_toml = ["pyproject.toml:project.version"]
version_variables = ["src/YOUR_PACKAGE/__init__.py:__version__"]
branch = "main"
upload_to_vcs_release = true
upload_to_pypi = false
build_command = "pip install build && python -m build"
```

- [ ] Add `[tool.semantic_release]` section
- [ ] Replace `YOUR_PACKAGE` with your package name
- [ ] Verify `version_toml` path is correct
- [ ] Verify `version_variables` path is correct

### Add Version to Package

- [ ] Add `__version__ = "0.1.0"` to `src/YOUR_PACKAGE/__init__.py`
- [ ] Add `version = "0.1.0"` to `pyproject.toml` under `[project]`
- [ ] Ensure versions match

### Add Coverage Markers to README

Add these HTML comments to `README.md`:

```markdown
<!-- Pytest Coverage Comment:Begin -->
<!-- Pytest Coverage Comment:End -->
```

- [ ] Add coverage markers to README.md
- [ ] Choose a good location (typically near top, after title)

## Step 5: Configure Branch Protection

- [ ] Go to repository → Settings → Rules → Rulesets
- [ ] Click "New ruleset" → "New branch ruleset"
- [ ] Configure:
  - [ ] Name: `Protect main`
  - [ ] Enforcement: Active
  - [ ] Target branches: Include pattern `main`
  - [ ] Rules:
    - [ ] Restrict deletions
    - [ ] Require pull request before merging
      - [ ] Required approvals: 1+
      - [ ] Dismiss stale reviews
      - [ ] Require conversation resolution
    - [ ] Require status checks to pass
      - [ ] Require branches up to date
      - [ ] Add checks: `Run Pre-commit Checks`, `Run Tests and Lint`
    - [ ] Block force pushes
- [ ] Create ruleset

## Step 6: Test the Setup

### Test CI Workflow

- [ ] Create a feature branch: `git checkout -b test/ci-setup`
- [ ] Make a small change (e.g., add comment to README)
- [ ] Commit: `git commit -m "chore: test CI workflow"`
- [ ] Push: `git push origin test/ci-setup`
- [ ] Create pull request on GitHub
- [ ] Verify CI workflow runs automatically
- [ ] Check that all jobs pass (pre-commit, tests, lint)
- [ ] Review workflow logs for any warnings

### Test CD Workflow

- [ ] Update commit message to: `fix: test CD workflow`
- [ ] Get PR approved
- [ ] Merge PR to main
- [ ] Verify CD workflow runs automatically
- [ ] Check that all jobs pass:
  - [ ] Coverage report generated
  - [ ] README updated with coverage badge
  - [ ] Version tag created (e.g., v0.1.1)
  - [ ] Wheel built
  - [ ] GitHub Release created with wheel asset
- [ ] Verify new version appears in GitHub Releases
- [ ] Verify README shows coverage badge

## Step 7: Document Usage

- [ ] Add CI/CD badges to README (optional)
- [ ] Document commit message conventions in README or CONTRIBUTING.md
- [ ] Add semantic versioning notes
- [ ] Document any project-specific workflow configuration

## Troubleshooting

If something doesn't work:

1. **Check workflow logs** in GitHub Actions tab
2. **Verify secret exists** and has correct name (`RELEASE_TOKEN`)
3. **Check PAT permissions** include required repos and permissions
4. **Verify workflow file paths** are exactly:
   - `.github/workflows/python-app.yml`
   - `.github/workflows/semantic-release.yml`
5. **Check commit message format** follows semantic conventions
6. **Review** [README Troubleshooting section](README.md#-troubleshooting)

## Optional Enhancements

- [ ] Add pre-commit hooks locally (`pip install pre-commit && pre-commit install`)
- [ ] Add `.pre-commit-config.yaml` for automated code quality
- [ ] Add CODEOWNERS file for workflow protection
- [ ] Add CONTRIBUTING.md with development guidelines
- [ ] Configure Renovate/Dependabot for dependency updates
- [ ] Add issue/PR templates

## Checklist Complete! 🎉

Your repository is now set up with automated CI/CD. Every PR will be tested, and every merge will create a semantic release.

**Remember:**
- Use semantic commit messages (`fix:`, `feat:`, `BREAKING CHANGE:`)
- Always work in feature branches
- Get PR reviews before merging
- Monitor workflow runs for failures
- Rotate PAT every 90 days

---

Need help? Check the [main README](README.md) or open an issue!
