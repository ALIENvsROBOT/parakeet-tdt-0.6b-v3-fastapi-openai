# GitHub Actions Workflows

This directory contains CI/CD workflows for automated testing, building, and publishing.

## Workflows

### ci.yml - Continuous Integration ✅

Automated testing and quality checks that run on every push and pull request.

**Triggers:**
- Push to `main` or `develop` branches
- Pull requests to `main`
- Manual dispatch

**What it does:**
1. **Run Tests**: Python syntax check, dependency verification
2. **Docker Build Test**: Verifies Docker images build successfully
3. **Security Scan**: Scans for vulnerabilities with Trivy
4. **Quality Gates**: Must pass before CD pipeline runs

**Jobs:**
- `test` - Python environment and syntax checks
- `docker-build-test` - Test Docker builds (CPU variant)
- `security-scan` - Vulnerability scanning
- `all-checks-passed` - Verification gate

---

### docker-publish.yml - Continuous Deployment 🚀

Automated Docker image build and publish workflow that creates both CPU and GPU variants.

**Only runs after CI passes!**

**Triggers:**
- Push to `main` branch → Creates `latest-cpu` and `latest-gpu` tags
- Version tags (e.g., `v3.1.0`) → Creates versioned tags
- Manual dispatch → Custom version builds

**What it does:**
1. Builds both CPU and GPU Docker images in parallel
2. Publishes to GitHub Container Registry (ghcr.io)
3. Creates semantic version tags
4. Generates build attestations
5. Creates GitHub Releases (for version tags)

---

## Usage

### Automatic Builds on Push

Every push to `main` automatically builds and publishes `latest` images:

```bash
git add .
git commit -m "feat: add new feature"
git push origin main
```

**Result:**
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu`

### Creating Versioned Releases

To create a new versioned release:

```bash
# 1. Tag the commit with semantic version
git tag v3.1.0

# 2. Push the tag
git push origin v3.1.0
```

**Result:**
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-cpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-gpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1-cpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1-gpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3-cpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3-gpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu`
- `ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu`
- GitHub Release with notes

### Manual Workflow Dispatch

For custom builds without tagging:

1. Go to **Actions** tab in GitHub
2. Select **Docker Build and Publish** workflow
3. Click **Run workflow**
4. Select branch: `main`
5. (Optional) Enter custom version: `v3.2.0`
6. Click **Run workflow**

---

## Semantic Versioning

We follow [Semantic Versioning 2.0.0](https://semver.org/):

**Format:** `vMAJOR.MINOR.PATCH`

- **MAJOR**: Incompatible API changes
- **MINOR**: Backward-compatible functionality additions
- **PATCH**: Backward-compatible bug fixes

**Examples:**
- `v3.0.0` - Major release with breaking changes
- `v3.1.0` - New features, backward compatible
- `v3.1.1` - Bug fixes only

### Version Tagging

Each version creates multiple tags for flexibility:

| Version Tag | Docker Tags Created |
|-------------|---------------------|
| `v3.1.2` | `v3.1.2-cpu`, `v3.1.2-gpu` (exact) |
| | `v3.1-cpu`, `v3.1-gpu` (minor) |
| | `v3-cpu`, `v3-gpu` (major) |
| | `latest-cpu`, `latest-gpu` (if default branch) |

**Why multiple tags?**
- **Exact** (`v3.1.2`): Pin to specific version
- **Minor** (`v3.1`): Auto-update patches, lock minor version
- **Major** (`v3`): Auto-update minor/patches, lock major version
- **Latest**: Always get newest stable release

---

## Image Naming Convention

All images are published to GitHub Container Registry:

```
ghcr.io/{owner}/{repo}:{version}-{variant}
```

**Examples:**
```
ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-cpu
ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-gpu
```

---

## Pulling Images

### Latest Stable Release

```bash
# CPU
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# GPU
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
```

### Specific Version

```bash
# CPU - Exact version
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-cpu

# GPU - Minor version (auto-updates patches)
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1-gpu
```

### Authentication

For private repositories, authenticate first:

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

**For public repositories**, no authentication needed.

---

## Workflow Configuration

### Environment Variables

Set in `.github/workflows/docker-publish.yml`:

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```

### Permissions Required

The workflow needs these permissions:

```yaml
permissions:
  contents: read    # Read repository contents
  packages: write   # Publish to GHCR
```

### Build Arguments

Passed to Docker build:

- `BUILD_DATE`: Timestamp of commit
- `VCS_REF`: Git commit SHA
- `VERSION`: Semantic version tag

---

## Monitoring Builds

### Check Build Status

1. Go to **Actions** tab in GitHub
2. View workflow runs
3. Click on a specific run for details

### View Published Images

1. Go to repository **Packages** section
2. Click on package name
3. View all tags and versions

---

## Troubleshooting

### Build Fails

**Check logs:**
1. Go to **Actions** → Failed workflow run
2. Expand failed job
3. Review error messages

**Common issues:**
- Invalid Dockerfile syntax
- Missing dependencies
- Registry authentication failure

### Image Not Available

**Verify publication:**
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

**Check:**
- Workflow completed successfully
- Image is public (or you're authenticated)
- Tag name is correct

### Permission Denied

For private repos:

```bash
# Create personal access token with packages:read scope
echo $PAT | docker login ghcr.io -u USERNAME --password-stdin
```

---

## Best Practices

### 1. Version Incrementing

```bash
# Current: v3.1.0
# Bug fix: v3.1.1
# New feature: v3.2.0
# Breaking change: v4.0.0
```

### 2. Tag Before Release

```bash
# Test locally first
docker build -f Dockerfile.cpu -t test:cpu .
docker run -p 5092:5092 test:cpu

# Then tag and push
git tag v3.1.0
git push origin v3.1.0
```

### 3. Changelog

Keep a CHANGELOG.md file:

```markdown
## [3.1.0] - 2024-02-12
### Added
- New feature X
- Improvement Y

### Fixed
- Bug Z
```

### 4. Pre-release Versions

For testing:

```bash
git tag v3.2.0-beta.1
git push origin v3.2.0-beta.1
```

Creates: `v3.2.0-beta.1-cpu` and `v3.2.0-beta.1-gpu`

---

## Advanced Configuration

### Multi-platform Builds

To support ARM64 (Apple Silicon, ARM servers):

```yaml
platforms: linux/amd64,linux/arm64
```

**Note**: Requires QEMU setup (already included in workflow).

### Build Caching

The workflow uses GitHub Actions cache:

```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

**Benefits:**
- Faster builds
- Reduced bandwidth
- Layer reuse

---

## Security

### Build Attestations

Workflow generates attestations:

```yaml
- name: Generate artifact attestation
  uses: actions/attest-build-provenance@v1
```

**Verify attestation:**
```bash
gh attestation verify ghcr.io/alienvsrobot/parakeet-tdt:v3.1.0-cpu \
  --owner alienvsrobot
```

### Secrets Management

**Never commit:**
- API keys
- Tokens
- Credentials

**Use GitHub Secrets instead:**
1. Settings → Secrets and variables → Actions
2. Add repository secret
3. Reference in workflow: `${{ secrets.SECRET_NAME }}`

---

## Support

For workflow issues:
- **GitHub Actions Docs**: https://docs.github.com/en/actions
- **Docker Build Action**: https://github.com/docker/build-push-action
- **Repository Issues**: https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/issues
