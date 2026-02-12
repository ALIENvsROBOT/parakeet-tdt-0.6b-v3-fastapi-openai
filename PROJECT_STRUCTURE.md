# Project Structure

## 📁 Complete Project Layout

```
parakeet-tdt-0.6b-v3-fastapi-openai/
│
├── 📂 .github/                    # GitHub configuration
│   └── 📂 workflows/
│       ├── docker-publish.yml     # ⭐ CD pipeline workflow
│       └── README.md              # Workflow documentation
│
├── 📂 docs/                       # ⭐ Documentation folder (NEW)
│   ├── README.md                  # Documentation index
│   ├── DOCKER.md                  # Docker deployment guide
│   ├── DEPLOYMENT.md              # Production deployment
│   ├── ENVIRONMENT.md             # Environment variables reference
│   └── CD_QUICKSTART.md           # CD pipeline quick start
│
├── 📂 templates/                  # Flask HTML templates
│   ├── index.html                 # Web UI
│   └── swagger.html               # API documentation UI
│
├── 📄 app.py                      # Main FastAPI application
├── 📄 benchmark.py                # Performance benchmarking script
├── 📄 inspect_model.py            # Model inspection utility
├── 📄 test_onnx_asr.py           # ONNX ASR tests
├── 📄 test_onnx_config.py        # ONNX configuration tests
│
├── 🐳 Dockerfile.cpu              # CPU Docker image (enhanced)
├── 🐳 Dockerfile.gpu              # GPU Docker image (enhanced)
├── 🐳 docker-compose.yml          # Docker Compose orchestration
├── 📄 .dockerignore               # Docker build exclusions
│
├── 📄 requirements.txt            # Python dependencies
├── 📄 .gitignore                  # Git exclusions
│
├── 📄 README.md                   # Main project README (updated)
├── 📄 CHANGELOG.md                # ⭐ Version history (NEW)
├── 📄 SETUP_SUMMARY.md            # ⭐ Setup summary (NEW)
├── 📄 PROJECT_STRUCTURE.md        # ⭐ This file (NEW)
│
├── 🖼️ parakeet.png                # Project logo
└── 📄 models                      # Model symlink/directory

⭐ = New or significantly enhanced
```

---

## 🎯 Key Components

### CI/CD Pipeline

**File**: `.github/workflows/docker-publish.yml`

**What it does:**
- ✅ Builds CPU and GPU Docker images in parallel
- ✅ Publishes to GitHub Container Registry
- ✅ Creates semantic version tags
- ✅ Auto-generates GitHub Releases
- ✅ Triggers on push, tags, or manual dispatch

**Triggers:**
```
Push to main     → latest-cpu, latest-gpu
Tag v3.1.0       → v3.1.0-cpu, v3.1.0-gpu, v3.1-cpu, v3.1-gpu, v3-cpu, v3-gpu
Manual dispatch  → Custom builds
```

---

### Documentation Structure

**Folder**: `docs/`

| File | Purpose | Audience |
|------|---------|----------|
| `README.md` | Documentation index | All users |
| `DOCKER.md` | Docker quick start | First-time users |
| `DEPLOYMENT.md` | Production deployment | DevOps/SRE |
| `ENVIRONMENT.md` | Configuration reference | Developers |
| `CD_QUICKSTART.md` | CD pipeline guide | Maintainers |

---

### Docker Images

**Files**: `Dockerfile.cpu`, `Dockerfile.gpu`

**Enhancements:**
- ✅ OCI metadata labels
- ✅ Build arguments (BUILD_DATE, VCS_REF, VERSION)
- ✅ Proper image versioning
- ✅ Health checks
- ✅ Security best practices (non-root user)

**Published to:**
```
ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:{version}-{variant}
```

---

## 🔄 Workflow Diagrams

### Development Workflow

```
┌─────────────┐
│ Code Change │
└──────┬──────┘
       │
       ├──> Local Testing
       │
       ├──> Commit & Push
       │
       ├──> GitHub Actions
       │    ├─> Build CPU Image
       │    └─> Build GPU Image
       │
       ├──> Publish to GHCR
       │    ├─> latest-cpu
       │    └─> latest-gpu
       │
       └──> Images Available
```

### Release Workflow

```
┌─────────────┐
│  Git Tag    │
│  v3.1.0     │
└──────┬──────┘
       │
       ├──> Trigger GitHub Actions
       │
       ├──> Build & Publish Images
       │    ├─> v3.1.0-cpu
       │    ├─> v3.1.0-gpu
       │    ├─> v3.1-cpu
       │    ├─> v3.1-gpu
       │    ├─> v3-cpu
       │    ├─> v3-gpu
       │    ├─> latest-cpu
       │    └─> latest-gpu
       │
       └──> Create GitHub Release
            ├─> Auto-generated notes
            ├─> Pull instructions
            └─> Quick start guide
```

---

## 📦 Image Variants

### CPU Image (`Dockerfile.cpu`)

**Base**: `python:3.10-slim`

**Use case:**
- ✅ Development
- ✅ Cost-effective deployment
- ✅ High-speed inference on modern CPUs (29.7x real-time)

**Size**: ~1.2GB

**Pull:**
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

### GPU Image (`Dockerfile.gpu`)

**Base**: `nvidia/cuda:12.1.1-cudnn8-runtime-ubuntu22.04`

**Use case:**
- ✅ GPU-accelerated inference
- ✅ Batch processing
- ✅ High-throughput scenarios

**Size**: ~4.5GB

**Requirements**: NVIDIA GPU + Container Toolkit

**Pull:**
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
```

---

## 🚀 Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai
cd parakeet-tdt-0.6b-v3-fastapi-openai
```

### 2. Choose Deployment Method

**Option A: Use Pre-built Images** (Recommended)
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
docker run -d -p 5092:5092 -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

**Option B: Docker Compose**
```bash
docker compose up parakeet-cpu -d
```

**Option C: Build from Source**
```bash
docker build -f Dockerfile.cpu -t parakeet:cpu .
docker run -d -p 5092:5092 -v parakeet-models:/app/models parakeet:cpu
```

### 3. Test

```bash
curl http://localhost:5092/health
```

**Expected response:**
```json
{"status":"healthy","model":"parakeet-tdt-0.6b-v3","speedup":"20.7x"}
```

---

## 📊 File Changes Summary

### New Files (19)

```
✅ .github/workflows/docker-publish.yml
✅ .github/workflows/README.md
✅ docs/README.md
✅ docs/DOCKER.md (moved from root)
✅ docs/DEPLOYMENT.md
✅ docs/ENVIRONMENT.md
✅ docs/CD_QUICKSTART.md
✅ CHANGELOG.md
✅ SETUP_SUMMARY.md
✅ PROJECT_STRUCTURE.md
```

### Modified Files (3)

```
📝 README.md                # Updated docs reference
📝 Dockerfile.cpu          # Added OCI metadata labels
📝 Dockerfile.gpu          # Added OCI metadata labels
```

---

## 🔧 Environment Variables

**Quick Reference** (see [docs/ENVIRONMENT.md](docs/ENVIRONMENT.md) for complete list):

| Variable | Default | Description |
|----------|---------|-------------|
| `HF_HOME` | `/app/models` | Model cache directory |
| `HF_HUB_CACHE` | `/app/models` | HuggingFace Hub cache |
| `PYTHONUNBUFFERED` | `1` | Unbuffered Python output |
| `NVIDIA_VISIBLE_DEVICES` | `all` | GPU visibility (GPU only) |

---

## 📚 Documentation Map

**Need to...** → **Read this:**

| Task | Documentation |
|------|---------------|
| First-time setup | [README.md](README.md) |
| Docker deployment | [docs/DOCKER.md](docs/DOCKER.md) |
| Production deployment | [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) |
| Configure environment | [docs/ENVIRONMENT.md](docs/ENVIRONMENT.md) |
| Set up CD pipeline | [docs/CD_QUICKSTART.md](docs/CD_QUICKSTART.md) |
| Manage versions | [CHANGELOG.md](CHANGELOG.md) |
| Understand changes | [SETUP_SUMMARY.md](SETUP_SUMMARY.md) |

---

## 🎯 Next Steps

### For End Users

1. Pull pre-built image
2. Run with Docker
3. Access at `http://localhost:5092`

### For Developers

1. Clone repository
2. Make changes
3. Test locally
4. Push to trigger CD

### For Maintainers

1. Review [docs/CD_QUICKSTART.md](docs/CD_QUICKSTART.md)
2. Set up GitHub Actions
3. Create first release: `git tag v3.1.0 && git push origin v3.1.0`
4. Monitor builds in Actions tab

---

## 🆘 Support

- **Documentation**: [docs/](docs/)
- **Issues**: [GitHub Issues](https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/issues)
- **Upstream**: [groxaxo/parakeet-tdt](https://github.com/groxaxo/parakeet-tdt-0.6b-v3-fastapi-openai)

---

## ✨ Features

✅ Automated CI/CD pipeline
✅ CPU and GPU Docker images
✅ Semantic versioning
✅ Comprehensive documentation
✅ Production-ready deployment
✅ OpenAI-compatible API
✅ 25 language support
✅ Real-time transcription
✅ Web interface included

**Ready for production deployment!** 🚀
