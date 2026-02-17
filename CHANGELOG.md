# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial reusable CI workflow (`reusable-ci.yml`)
- Initial reusable CD workflow (`reusable-cd.yml`)
- Example caller workflows for CI and CD
- Comprehensive documentation in README.md
- Self-validation workflows for cicd_github repository
- Example environment variable configuration
- Semantic release configuration

### Features
- Support for both `pip` and `uv` package managers
- Support for both `ruff` and `flake8` linters
- Support for both `src/` and `app/` package structures
- Configurable Python versions
- Optional pre-commit hook execution
- Automated coverage reporting
- Semantic version tagging
- Wheel building and GitHub Release publishing
- Flexible configuration via workflow inputs

## [0.1.0] - 2026-02-17

### Added
- Initial repository structure
- Project documentation

[Unreleased]: https://github.com/YOUR_USERNAME/cicd_github/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/YOUR_USERNAME/cicd_github/releases/tag/v0.1.0
