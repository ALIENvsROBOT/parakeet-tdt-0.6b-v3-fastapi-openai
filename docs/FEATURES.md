# Features

Complete feature list for Parakeet TDT 0.6B v3 FastAPI OpenAI-Compatible Speech Recognition Service.

---

## 🎯 Core Features

### 1. Speech Recognition
- ✅ NVIDIA Parakeet TDT 0.6B v3 model
- ✅ ONNX Runtime with INT8 quantization
- ✅ 25 European languages support
- ✅ Automatic language detection (no language parameter needed)
- ✅ High-speed transcription (10.6x - 29.7x real-time)
- ✅ Automatic punctuation and capitalization

### 2. Audio Processing
- ✅ Automatic audio conversion to 16kHz mono WAV
- ✅ Supports multiple input formats (MP3, WAV, FLAC, M4A, OGG, etc.)
- ✅ Intelligent silence-based chunking for long audio
- ✅ Time-based chunking fallback
- ✅ Maximum 2GB file size support
- ✅ FFmpeg integration for audio preprocessing

### 3. Intelligent Chunking System
- ✅ Silence detection using FFmpeg silencedetect filter
- ✅ Optimal split point calculation (avoids mid-word cuts)
- ✅ Configurable chunk duration (default 90 seconds)
- ✅ Search window for silence points (±30 seconds)
- ✅ Minimum split gap protection (5 seconds)
- ✅ Timeout protection for silence detection (300s)

---

## 🌐 API Features

### 4. OpenAI-Compatible API
- ✅ Drop-in replacement for OpenAI Whisper API
- ✅ Standard `/v1/audio/transcriptions` endpoint
- ✅ OpenAI Python SDK compatible
- ✅ Bearer token authentication support (optional)
- ✅ API key validation via `API_KEY` environment variable

### 5. Response Formats
- ✅ `json` - Standard JSON with text
- ✅ `text` - Plain text output
- ✅ `srt` - SubRip subtitle format
- ✅ `vtt` - WebVTT subtitle format
- ✅ `verbose_json` - Detailed JSON with segments
- ✅ `parakeet_srt_words` - SRT + word timestamps (legacy)

### 6. Timestamp Support
- ✅ Word-level timestamps
- ✅ Segment-level timestamps
- ✅ Character-level timestamps
- ✅ Cumulative time offset for chunked audio
- ✅ Accurate timestamp alignment across chunks

---

## 🖥️ Web Interface

### 7. Built-in Web UI
- ✅ Drag-and-drop file upload
- ✅ Real-time transcription progress
- ✅ Clean, responsive design
- ✅ Accessible at `http://localhost:5092`
- ✅ Parakeet logo and branding
- ✅ Format selection (Text, JSON, SRT, VTT)

### 8. API Documentation
- ✅ Swagger/OpenAPI UI at `/docs`
- ✅ OpenAPI JSON spec at `/openapi.json`
- ✅ Interactive API testing

---

## 📊 Monitoring & Health

### 9. Health Check Endpoints
- ✅ `/health` - Service health status
- ✅ `/status` - System status with CPU/memory/disk info
- ✅ `/metrics` - Prometheus-compatible metrics
- ✅ `/progress/<job_id>` - Real-time job progress tracking

### 10. Progress Tracking
- ✅ Job ID generation for each request
- ✅ Real-time progress updates
- ✅ Current chunk tracking
- ✅ Progress percentage
- ✅ Partial text streaming
- ✅ Status tracking (processing/complete/error)

### 11. System Metrics
- ✅ CPU usage monitoring
- ✅ Memory usage tracking
- ✅ Disk space monitoring
- ✅ Model info (name, quantization, provider)
- ✅ Uptime tracking

---

## 🐳 Docker & Deployment

### 12. Multi-Platform Docker Images
- ✅ **Dockerfile.cpu** - Universal CPU support
- ✅ **Dockerfile.cuda-gpu** - NVIDIA CUDA GPU (with cuDNN 9)
- ✅ **Dockerfile.rocm-gpu** - AMD ROCm GPU (build locally)

### 13. Docker Compose Services
- ✅ `parakeet-cpu` - CPU-only deployment
- ✅ `parakeet-cuda-gpu` - NVIDIA GPU deployment
- ✅ `parakeet-rocm-gpu` - AMD GPU deployment
- ✅ Persistent model volume (`parakeet-models`)
- ✅ Health checks for all services
- ✅ Auto-restart policies

### 14. GPU Support
- ✅ Automatic GPU provider detection
- ✅ Priority: ROCm → CUDA → CPU
- ✅ CUDA 12.6.2 with cuDNN 9 support
- ✅ ROCm 6.2 support
- ✅ Multi-GPU optimization
- ✅ Fallback to CPU if GPU unavailable

---

## 🔧 CI/CD & Automation

### 15. GitHub Actions CI
- ✅ Python syntax checking
- ✅ Code style validation (flake8)
- ✅ Dependency verification
- ✅ Docker build testing
- ✅ Security scanning (Trivy)
- ✅ Multi-Python version support (3.10)

### 16. GitHub Actions CD
- ✅ Automatic Docker image building on tags
- ✅ GitHub Container Registry publishing
- ✅ Multi-variant builds (CPU, CUDA-GPU)
- ✅ Artifact attestation
- ✅ Automatic GitHub releases
- ✅ Semantic versioning support

### 17. OCI Metadata Labels
- ✅ Build date tracking
- ✅ VCS revision tracking
- ✅ Version labeling
- ✅ Variant identification
- ✅ Source repository linking

---

## 🔐 Security Features

### 18. Security
- ✅ Optional API key authentication
- ✅ Bearer token support
- ✅ Non-root container user
- ✅ Secure file handling
- ✅ Automatic cleanup of temporary files
- ✅ Input validation and sanitization
- ✅ Trivy security scanning in CI

---

## 📚 Documentation

### 19. Comprehensive Docs
- ✅ `README.md` - Main documentation
- ✅ `docs/DOCKER.md` - Docker deployment guide
- ✅ `docs/DEPLOYMENT.md` - Production deployment guide
- ✅ `docs/ENVIRONMENT.md` - Environment variables reference
- ✅ `docs/GPU_SUPPORT.md` - Multi-GPU setup guide
- ✅ `docs/CD_QUICKSTART.md` - CI/CD quickstart
- ✅ `docs/FEATURES.md` - Complete feature list (this file)
- ✅ `CHANGELOG.md` - Version history
- ✅ `PROJECT_STRUCTURE.md` - Repository structure

### 20. Benchmarking
- ✅ `benchmark.py` - Performance benchmarking script
- ✅ Speed measurement (real-time factor)
- ✅ Resource usage tracking
- ✅ Multiple audio file testing
- ✅ JSON report generation
- ✅ Statistical analysis

---

## 🔌 Integration Features

### 21. Open WebUI Integration
- ✅ Out-of-the-box compatibility
- ✅ Drop-in Whisper API replacement
- ✅ Voice interaction support
- ✅ Local processing (privacy-focused)
- ✅ Setup documentation included

### 22. Client SDK Support
- ✅ OpenAI Python SDK compatible
- ✅ Standard HTTP REST API
- ✅ cURL examples provided
- ✅ Multiple language support (any HTTP client)

---

## ⚙️ Configuration & Optimization

### 23. Performance Tuning
- ✅ Configurable CPU threading (8 P-cores optimized)
- ✅ GPU-specific optimizations
- ✅ Execution mode configuration
- ✅ Graph optimization levels
- ✅ Batch processing support
- ✅ Memory-efficient processing

### 24. Environment Variables
- ✅ `API_KEY` - Optional authentication
- ✅ `HF_HOME` - Model cache location
- ✅ `HF_HUB_CACHE` - HuggingFace cache
- ✅ `PYTHONUNBUFFERED` - Python output buffering
- ✅ `NVIDIA_VISIBLE_DEVICES` - GPU selection
- ✅ `HSA_OVERRIDE_GFX_VERSION` - ROCm GPU config

### 25. Customization
- ✅ Configurable chunk duration (default 1.5 minutes)
- ✅ Adjustable silence threshold (-40dB)
- ✅ Configurable silence detection parameters
- ✅ Custom port (default 5092)
- ✅ Thread count customization (default 8)

---

## 🧪 Testing & Validation

### 26. Test Scripts
- ✅ `test_onnx_asr.py` - ONNX ASR testing
- ✅ `test_onnx_config.py` - Configuration testing
- ✅ `inspect_model.py` - Model inspection tool
- ✅ Health check validation

---

## 📦 Additional Utilities

### 27. Helper Functions
- ✅ Audio duration detection
- ✅ Silence point detection
- ✅ Optimal split point calculation
- ✅ SRT time formatting
- ✅ VTT formatting
- ✅ Text cleaning (SentencePiece artifacts)
- ✅ Token cleaning and normalization

### 28. Error Handling
- ✅ Comprehensive error messages
- ✅ OpenAI-compatible error format
- ✅ Graceful fallbacks
- ✅ Timeout protection
- ✅ Resource cleanup on errors
- ✅ Detailed logging

---

## 🌍 Language Support

### 29. 25 Supported Languages

The model automatically detects and transcribes speech in any of these languages:

| Language | Code | Language | Code |
|----------|------|----------|------|
| English | en | Polish | pl |
| Spanish | es | Ukrainian | uk |
| French | fr | Romanian | ro |
| Russian | ru | Dutch | nl |
| German | de | Hungarian | hu |
| Italian | it | Greek | el |
| Swedish | sv | Lithuanian | lt |
| Czech | cs | Slovenian | sl |
| Bulgarian | bg | Latvian | lv |
| Portuguese | pt | Estonian | et |
| Slovak | sk | Maltese | mt |
| Croatian | hr | | |
| Danish | da | | |
| Finnish | fi | | |

---

## 📈 Performance Metrics

### 30. Benchmarked Performance

| Hardware | Speedup | Real-time Factor | Notes |
|----------|---------|------------------|-------|
| Intel i7-12700KF (CPU) | ~29.7x | 0.033 | 8 P-cores + INT8 |
| Intel i7-4790 (CPU) | ~17.0x | 0.059 | Older generation |
| NVIDIA CUDA GPU | ~15-20x* | ~0.05-0.066* | Estimated |
| AMD ROCm GPU | ~12-18x* | ~0.055-0.083* | Estimated |

*GPU performance varies by model and VRAM

**Key Metrics:**
- ✅ Memory: ~1.5 GB RAM (CPU) / ~2 GB VRAM (GPU)
- ✅ Beats Whisper Large v3 on CPU performance
- ✅ Lower latency than cloud APIs
- ✅ Consistent performance across all 25 languages

---

## 🏆 Performance Comparison

### vs. faster-whisper (from benchmarks)

| Implementation | Hardware | Model | Precision | Speedup |
|----------------|----------|-------|-----------|---------|
| **Parakeet TDT** (Ours) | **CPU** (i7-12700KF) | **TDT 0.6B v3** | **int8** | **~29.7x** |
| **Parakeet TDT** (Ours) | **CPU** (i7-4790) | **TDT 0.6B v3** | **int8** | **~17.0x** |
| faster-whisper | GPU (RTX 3070 Ti) | Large-v2 | int8 | 13.2x |
| faster-whisper | GPU (RTX 3070 Ti) | Large-v2 | fp16 | 12.4x |
| faster-whisper | CPU (i7-12700K) | Small | int8 | 7.6x |
| faster-whisper | CPU (i7-12700K) | Small | fp32 | 4.9x |

**Key Advantages:**
- ✅ Faster than GPU Whisper on modern CPUs
- ✅ Multilingual support (25 languages vs Whisper's 99 with slower performance)
- ✅ Smaller model size (600M parameters)
- ✅ Lower memory footprint
- ✅ Better CPU optimization

---

## 📋 Feature Categories Summary

| Category | Feature Count | Status |
|----------|---------------|--------|
| Core Speech Recognition | 3 | ✅ Complete |
| API & Integration | 4 | ✅ Complete |
| Web Interface | 2 | ✅ Complete |
| Monitoring & Health | 3 | ✅ Complete |
| Docker & Deployment | 3 | ✅ Complete |
| CI/CD & Automation | 3 | ✅ Complete |
| Security | 1 | ✅ Complete |
| Documentation | 2 | ✅ Complete |
| Integration Features | 2 | ✅ Complete |
| Configuration | 3 | ✅ Complete |
| Testing | 1 | ✅ Complete |
| Utilities | 2 | ✅ Complete |
| Language Support | 1 | ✅ 25 languages |
| Performance | 1 | ✅ Benchmarked |

**Total: 30 Major Feature Categories**

---

## 🚀 Quick Feature Overview

**What makes this project special:**

1. **Speed** - Up to 29.7x faster than real-time on modern CPUs
2. **Multilingual** - 25 European languages with automatic detection
3. **Compatible** - Drop-in OpenAI Whisper API replacement
4. **Flexible** - CPU, NVIDIA GPU, or AMD GPU support
5. **Private** - All processing happens locally
6. **Production-Ready** - Full CI/CD, monitoring, and documentation
7. **Easy to Use** - Docker deployment, web UI, and simple API
8. **Well-Documented** - Comprehensive guides for all use cases

---

## 📖 Related Documentation

- [Main README](../README.md) - Getting started guide
- [Docker Guide](DOCKER.md) - Container deployment
- [GPU Support](GPU_SUPPORT.md) - Multi-GPU setup
- [Environment Variables](ENVIRONMENT.md) - Configuration reference
- [Deployment Guide](DEPLOYMENT.md) - Production deployment
- [CD Quickstart](CD_QUICKSTART.md) - CI/CD setup

---

**Last Updated:** v3.0.5 (2026-02-12)
