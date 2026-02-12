# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- GitHub Actions CD pipeline for automated Docker builds
- Comprehensive documentation in `docs/` folder
  - DEPLOYMENT.md - Production deployment guide
  - ENVIRONMENT.md - Environment variables reference
  - DOCKER.md - Docker usage guide (moved from root)
  - README.md - Documentation index
- `.github/workflows/docker-publish.yml` - Automated CI/CD workflow
- Support for semantic versioning (v3.x.x format)
- Docker image labels and build metadata
- CHANGELOG.md for version tracking

### Changed
- Organized documentation into `docs/` folder
- Enhanced Dockerfiles with OCI metadata labels
- Updated README.md to reference new documentation structure

### Fixed
- None

---

## Release Template

Use this template when creating a new release:

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New features or capabilities

### Changed
- Changes to existing functionality

### Deprecated
- Features that will be removed in future versions

### Removed
- Features that have been removed

### Fixed
- Bug fixes

### Security
- Security-related changes

[X.Y.Z]: https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/compare/vX.Y.Z-1...vX.Y.Z
```

---

## Version History

_Versions will be added here as releases are created_

<!--
Example:
## [3.1.0] - 2024-02-12
### Added
- Docker compose support for GPU deployment
- Automatic silence-based audio chunking

[3.1.0]: https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/releases/tag/v3.1.0
-->
