# Parakeet TDT Documentation

Welcome to the Parakeet TDT documentation. This directory contains comprehensive guides for deploying, configuring, and using the Parakeet TDT speech recognition service.

## 📚 Documentation Index

### Getting Started
- **[Main README](../README.md)** - Project overview, features, and quick start
- **[Docker Guide](DOCKER.md)** - Docker deployment options and troubleshooting

### Deployment & Operations
- **[Deployment Guide](DEPLOYMENT.md)** - Production deployment with Docker, Kubernetes, and CI/CD
- **[Environment Variables](ENVIRONMENT.md)** - Complete reference for all configuration options

### Additional Resources
- **[GitHub Repository](https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai)** - Source code and issues
- **[Original Project](https://github.com/groxaxo/parakeet-tdt-0.6b-v3-fastapi-openai)** - Upstream repository

---

## Quick Links

### Common Tasks

| Task | Documentation |
|------|---------------|
| First-time setup | [Main README → Installation](../README.md#installation) |
| Docker deployment | [Docker Guide](DOCKER.md) |
| Production deployment | [Deployment Guide → Production](DEPLOYMENT.md#production-deployment) |
| Environment configuration | [Environment Variables](ENVIRONMENT.md) |
| CI/CD setup | [Deployment Guide → CI/CD](DEPLOYMENT.md#cicd-with-github-actions) |
| Troubleshooting | [Docker Guide → Troubleshooting](DOCKER.md#troubleshooting) |
| Performance tuning | [Deployment Guide → Performance](DEPLOYMENT.md#performance-tuning) |

---

## Document Summaries

### [DOCKER.md](DOCKER.md)
Quick reference for Docker deployments:
- CPU and GPU container builds
- Docker Compose usage
- Volume management
- Health checks
- Basic troubleshooting

**Best for**: Quick Docker deployments and testing

---

### [DEPLOYMENT.md](DEPLOYMENT.md)
Comprehensive production deployment guide:
- Pre-built image usage
- Docker Swarm orchestration
- Kubernetes deployments
- CI/CD with GitHub Actions
- Performance optimization
- Monitoring and security

**Best for**: Production environments and advanced deployments

---

### [ENVIRONMENT.md](ENVIRONMENT.md)
Complete environment variable reference:
- Application configuration (host, port, threads)
- Model & cache paths
- Docker runtime settings
- GPU configuration
- Audio processing parameters
- ONNX Runtime tuning

**Best for**: Configuration and customization

---

## API Documentation

### Interactive API Docs

Once the server is running, access interactive API documentation:

- **Swagger UI**: http://localhost:5092/docs
- **OpenAPI Spec**: http://localhost:5092/openapi.json

### Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Web UI for testing |
| `/health` | GET | Health check endpoint |
| `/metrics` | GET | Real-time CPU/RAM metrics |
| `/v1/audio/transcriptions` | POST | OpenAI-compatible transcription API |

### Example Usage

**Python (OpenAI SDK):**
```python
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:5092/v1",
    api_key="sk-no-key-required"
)

with open("audio.mp3", "rb") as audio_file:
    transcript = client.audio.transcriptions.create(
        model="parakeet-tdt-0.6b-v3",
        file=audio_file,
        response_format="text"
    )

print(transcript)
```

**cURL:**
```bash
curl -X POST http://localhost:5092/v1/audio/transcriptions \
  -F "file=@audio.mp3" \
  -F "model=parakeet-tdt-0.6b-v3" \
  -F "response_format=text"
```

---

## Architecture Overview

### Components

```
┌─────────────────────────────────────────────┐
│           FastAPI Application               │
│  (app.py - Waitress WSGI Server)           │
└─────────────┬───────────────────────────────┘
              │
              ├──> ONNX Runtime (CPU/GPU)
              │    └──> Parakeet TDT 0.6B v3 Model
              │
              ├──> FFmpeg (Audio Processing)
              │    └──> Intelligent Chunking
              │
              └──> HuggingFace Hub (Model Cache)
```

### Processing Flow

1. **Audio Upload** → Client sends audio file via OpenAI-compatible API
2. **Format Conversion** → FFmpeg converts to 16kHz mono WAV
3. **Intelligent Chunking** → Long files split using silence detection
4. **Transcription** → ONNX Runtime processes with Parakeet TDT model
5. **Response** → Returns text/JSON/SRT/VTT format

---

## Performance Benchmarks

### CPU Performance (Intel i7-12700KF)

| Metric | Value |
|--------|-------|
| **Average Speedup** | **29.7x** |
| **Real Time Factor** | **0.033** |
| **Max Speedup** | **~30x** |

### Comparison

| Implementation | Hardware | Model | Speedup |
|----------------|----------|-------|---------|
| **Parakeet TDT** (This) | **CPU** (i7-12700KF) | **TDT 0.6B v3** | **~29.7x** |
| faster-whisper | GPU (RTX 3070 Ti) | Large-v2 | 13.2x |
| faster-whisper | CPU (i7-12700K) | Small | 7.6x |

---

## Supported Features

### ✅ Supported

- **25 Languages** with automatic detection
- **Multiple Response Formats**: JSON, text, SRT, VTT, verbose_json
- **OpenAI API Compatibility**: Drop-in replacement
- **Intelligent Chunking**: Silence-based splitting for long audio
- **CPU & GPU Support**: Optimized for both
- **Docker Deployment**: Pre-built images with CI/CD
- **Web Interface**: Built-in drag-and-drop UI
- **Health Monitoring**: `/health` and `/metrics` endpoints

### 🌍 Supported Languages

English, Spanish, French, Russian, German, Italian, Polish, Ukrainian, Romanian, Dutch, Hungarian, Greek, Swedish, Czech, Bulgarian, Portuguese, Slovak, Croatian, Danish, Finnish, Lithuanian, Slovenian, Latvian, Estonian, Maltese

---

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

**Note**: This is a fork of the original [groxaxo/parakeet-tdt-0.6b-v3-fastapi-openai](https://github.com/groxaxo/parakeet-tdt-0.6b-v3-fastapi-openai) project.

---

## License

MIT License - See [LICENSE](../LICENSE) file for details

---

## Acknowledgments

- **[NVIDIA](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3)** - Parakeet TDT model
- **[Shadowfita](https://github.com/Shadowfita/parakeet-tdt-0.6b-v2-fastapi)** - Original FastAPI implementation
- **[groxaxo](https://github.com/groxaxo)** - ONNX optimization and multilingual support

---

## Support & Community

- **Report Issues**: [GitHub Issues](https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/issues)
- **Discussions**: [GitHub Discussions](https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/discussions)
- **Upstream Project**: [groxaxo/parakeet-tdt](https://github.com/groxaxo/parakeet-tdt-0.6b-v3-fastapi-openai)
