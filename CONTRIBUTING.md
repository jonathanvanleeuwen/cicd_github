# Contributing to cicd_github

Thank you for your interest in contributing to cicd_github! This document provides guidelines for contributing to the project.

## Code of Conduct

Be respectful and constructive. We're all here to improve CI/CD workflows for Python projects.

## How to Contribute

### Reporting Issues

If you find a bug or have a feature request:

1. **Check existing issues** first to avoid duplicates
2. **Create a new issue** with:
   - Clear title and description
   - Steps to reproduce (for bugs)
   - Expected vs actual behavior
   - Workflow configuration that triggers the issue
   - Relevant log output

### Submitting Changes

1. **Fork the repository**
2. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes** following the guidelines below
4. **Test your changes** (see Testing section)
5. **Commit with semantic messages**:
   - `feat: add support for Python 3.13`
   - `fix: correct coverage path detection`
   - `docs: update troubleshooting guide`
6. **Push to your fork** and **create a Pull Request**

## Development Guidelines

### Workflow Design Principles

Follow these principles when modifying workflows:

#### DRY (Don't Repeat Yourself)
- Extract common steps into reusable patterns
- Use inputs for configuration, not hardcoded values
- Avoid duplicating logic across workflows

#### CLEAN Code
- **Clear**: Workflow intent is obvious from reading it
- **Logical**: Each job/step has a single purpose
- **Easy to understand**: Junior developers should be able to use it
- **Accessible**: Avoid complex bash scripts; prefer clear commands
- **Necessary**: Every step serves a purpose; no dead code

#### Flexibility
- Support common variations via inputs (package managers, linters, etc.)
- Provide sensible defaults
- Allow overrides when needed

#### Backwards Compatibility
- Don't break existing callers without major version bump
- Add new inputs as optional with defaults
- Deprecate features gradually with warnings

### Testing Changes

Since these are reusable workflows, testing requires a test repository:

1. **Create a test repository** (or use an existing one)
2. **Reference your development branch**:
   ```yaml
   uses: YOUR_USERNAME/cicd_github/.github/workflows/reusable-ci.yml@your-feature-branch
   ```
3. **Open a PR in the test repository** to trigger the workflow
4. **Verify the workflow executes correctly**
5. **Test various configurations** (different inputs, edge cases)
6. **Check logs** for any errors or warnings

### Workflow Testing Checklist

Test your changes with:

- ✅ Default configuration (all inputs use defaults)
- ✅ `pip` package manager
- ✅ `uv` package manager
- ✅ `ruff` linter
- ✅ `flake8` linter
- ✅ `src/` package structure
- ✅ `app/` package structure
- ✅ With pre-commit enabled
- ✅ With pre-commit disabled
- ✅ With extra install args
- ✅ Without extra install args
- ✅ Python 3.12
- ✅ Python 3.11 (if supporting older versions)

### Documentation

#### Update README.md

When adding features:
- Add to **Configuration Reference** table
- Add to **Usage Examples** section
- Update **Troubleshooting** if needed
- Add to **Changelog** (Features section)

When fixing bugs:
- Update **Troubleshooting** section
- Add to **Changelog** (Fixed section)

#### Update Examples

Keep examples synced with latest features:
- Update `examples/caller-ci.yml` with new inputs
- Update `examples/caller-cd.yml` with new inputs
- Add comments explaining new options

#### Comments in Workflows

Use comments to explain:
- **WHY** a step exists (not HOW it works)
- Non-obvious configuration choices
- Workarounds for GitHub Actions limitations
- Complex bash scripts (or refactor to be simpler)

### Commit Messages

Follow [Semantic Commit Messages](https://www.conventionalcommits.org/):

```
<type>: <description>

[optional body]

[optional footer]
```

**Types:**
- `feat:` - New feature (minor version bump)
- `fix:` - Bug fix (patch version bump)
- `docs:` - Documentation only
- `chore:` - Maintenance (no version bump)
- `refactor:` - Code restructuring (no version bump)
- `test:` - Add/update tests
- `ci:` - CI/CD changes

**Examples:**
```
feat: add support for Python 3.13

feat: add configurable test coverage threshold
- New input: minimum-coverage
- Workflow fails if coverage below threshold
- Defaults to 0 (no minimum)

fix: correct coverage path detection for app/ structure

docs: add troubleshooting section for PAT permissions

chore: update actions/checkout to v5
```

## Release Process

Releases are automated via semantic-release:

1. **Merge PR to main** (with semantic commit messages)
2. **Workflow automatically**:
   - Analyzes commit messages
   - Bumps version (major.minor.patch)
   - Creates git tag
   - Creates GitHub Release
   - Attaches workflow files to release
3. **Users can pin to version**: `@v1.2.3`

### Version Bumps

| Commits | Version Change | Example |
|---------|----------------|---------|
| Only `fix:` | Patch | 1.2.3 → 1.2.4 |
| Any `feat:` | Minor | 1.2.4 → 1.3.0 |
| `BREAKING CHANGE:` | Major | 1.3.0 → 2.0.0 |
| Only `docs:`, `chore:` | None | 1.3.0 → 1.3.0 |

## Questions?

- Open an issue for questions
- Tag maintainers in discussions
- Check existing issues and PRs

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Remember:** These workflows are used by multiple projects. Changes have downstream impact. Test thoroughly! 🚀
