# CD Pipeline Quick Start Guide

This guide will help you get started with the automated CD pipeline for building and publishing Docker images.

## 🚀 First-Time Setup

### 1. Enable GitHub Actions

Actions should be enabled by default. Verify:

1. Go to your repository on GitHub
2. Click **Settings** → **Actions** → **General**
3. Ensure "Allow all actions and reusable workflows" is selected

### 2. Configure Package Visibility

Make your packages public (optional):

1. Go to repository **Settings** → **Actions** → **General**
2. Scroll to **Workflow permissions**
3. Ensure "Read and write permissions" is checked
4. Save changes

### 3. Test the Workflow

Trigger a manual build:

1. Go to **Actions** tab
2. Select **Docker Build and Publish**
3. Click **Run workflow**
4. Select branch: `main`
5. Click **Run workflow** (leave version empty for latest)

**Expected result**: Two jobs run in parallel (CPU and GPU builds)

---

## 📦 Publishing Your First Release

### Step 1: Make Your Changes

```bash
# Edit code
vim app.py

# Test locally
docker build -f Dockerfile.cpu -t test:cpu .
docker run -p 5092:5092 test:cpu
```

### Step 2: Commit Changes

```bash
git add .
git commit -m "feat: add new transcription feature"
git push origin main
```

**Result**: Automatically builds `latest-cpu` and `latest-gpu` tags

### Step 3: Create a Version Tag

```bash
# Tag with semantic version
git tag v3.1.0

# Push tag to GitHub
git push origin v3.1.0
```

**Result**:
- Builds and publishes versioned images
- Creates GitHub Release with auto-generated notes
- Tags: `v3.1.0-cpu`, `v3.1.0-gpu`, `v3.1-cpu`, `v3.1-gpu`, `v3-cpu`, `v3-gpu`, `latest-cpu`, `latest-gpu`

---

## 🏷️ Version Numbering Guide

Follow [Semantic Versioning](https://semver.org/):

```
vMAJOR.MINOR.PATCH
```

**When to increment:**

| Change Type | Version | Example |
|-------------|---------|---------|
| Bug fix | `PATCH` | `v3.1.0` → `v3.1.1` |
| New feature (backward compatible) | `MINOR` | `v3.1.1` → `v3.2.0` |
| Breaking change | `MAJOR` | `v3.2.0` → `v4.0.0` |

### Examples

```bash
# Bug fix release
git tag v3.1.1
git push origin v3.1.1

# New feature release
git tag v3.2.0
git push origin v3.2.0

# Major release with breaking changes
git tag v4.0.0
git push origin v4.0.0
```

---

## 🐳 Using Published Images

### Pull Latest

```bash
# CPU
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# GPU
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
```

### Pull Specific Version

```bash
# Exact version
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-cpu

# Minor version (auto-updates patches)
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1-cpu

# Major version (auto-updates minor and patches)
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3-cpu
```

### Run

```bash
# CPU
docker run -d -p 5092:5092 \
  --name parakeet \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# GPU
docker run -d -p 5092:5092 \
  --name parakeet \
  --gpus all \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
```

---

## 🔍 Monitoring Builds

### Check Build Status

**GitHub UI:**
1. Go to **Actions** tab
2. View recent workflow runs
3. Green ✓ = success, Red ✗ = failure

**Badge (add to README):**
```markdown
[![Docker Build](https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/actions/workflows/docker-publish.yml)
```

### View Published Packages

1. Go to repository main page
2. Right sidebar → **Packages**
3. Click package name
4. View all tags and metadata

---

## 📝 Updating CHANGELOG

Before releasing, update `CHANGELOG.md`:

```markdown
## [3.1.0] - 2024-02-12

### Added
- New silence-based chunking feature
- Improved GPU support

### Fixed
- Audio processing bug with long files
```

**Commit the CHANGELOG:**
```bash
git add CHANGELOG.md
git commit -m "docs: update changelog for v3.1.0"
git push origin main

# Then tag
git tag v3.1.0
git push origin v3.1.0
```

---

## 🛠️ Common Workflows

### Hotfix Release

```bash
# Fix critical bug
vim app.py

# Commit and push
git add app.py
git commit -m "fix: critical security patch"
git push origin main

# Immediately tag patch version
git tag v3.1.1
git push origin v3.1.1
```

**Result**: New images available in ~5-10 minutes

### Pre-release Testing

```bash
# Tag as beta
git tag v3.2.0-beta.1
git push origin v3.2.0-beta.1
```

**Pull and test:**
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.2.0-beta.1-cpu
```

### Rollback

**Docker rollback** (if something breaks):
```bash
# Switch to previous version
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.0.0-cpu
docker run -d -p 5092:5092 ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.0.0-cpu
```

**Git rollback** (to fix and re-release):
```bash
# Delete bad tag locally and remotely
git tag -d v3.1.0
git push origin :refs/tags/v3.1.0

# Fix issue, then re-tag
git tag v3.1.0
git push origin v3.1.0
```

---

## 🚨 Troubleshooting

### Build Fails

**1. Check logs:**
- Go to Actions → Failed workflow → View logs

**2. Common issues:**
- **Dockerfile syntax error**: Fix Dockerfile, push fix
- **Missing dependency**: Update requirements.txt
- **Registry auth**: Check repo permissions

### Image Not Available

**Verify publication:**
```bash
# List tags
docker manifest inspect ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

**Check:**
- Workflow completed successfully
- Package is public (Settings → Packages)
- Correct tag name

### Workflow Doesn't Trigger

**Check:**
- `.github/workflows/docker-publish.yml` exists
- Actions enabled in repository settings
- Pushed to correct branch (`main`)
- Tag format is correct (`vX.Y.Z`)

---

## 📚 Additional Resources

- **Full Documentation**: [docs/](.)
- **Workflow Details**: [.github/workflows/README.md](../.github/workflows/README.md)
- **Deployment Guide**: [DEPLOYMENT.md](DEPLOYMENT.md)
- **Environment Variables**: [ENVIRONMENT.md](ENVIRONMENT.md)

---

## 🎯 Quick Reference

```bash
# Build locally
docker build -f Dockerfile.cpu -t test:cpu .

# Push to trigger latest build
git push origin main

# Create versioned release
git tag v3.1.0 && git push origin v3.1.0

# Pull published image
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# Run
docker run -d -p 5092:5092 -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

---

## ✅ Checklist for First Release

- [ ] Test builds locally
- [ ] Update CHANGELOG.md
- [ ] Commit all changes
- [ ] Push to main
- [ ] Verify Actions workflow succeeds
- [ ] Tag version (e.g., `v3.1.0`)
- [ ] Push tag
- [ ] Verify GitHub Release created
- [ ] Test pulling published image
- [ ] Update documentation if needed

**You're ready to go! 🎉**
