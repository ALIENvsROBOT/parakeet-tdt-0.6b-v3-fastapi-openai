# Environment Variables Reference

This document lists all environment variables used in the Parakeet TDT project.

## Table of Contents
- [Application Configuration](#application-configuration)
- [Model & Cache Configuration](#model--cache-configuration)
- [Docker & Runtime Configuration](#docker--runtime-configuration)
- [GPU Configuration (GPU Build Only)](#gpu-configuration-gpu-build-only)
- [Audio Processing Configuration](#audio-processing-configuration)

---

## Application Configuration

### `host`
- **Default**: `"0.0.0.0"`
- **Description**: Host address for the FastAPI server to bind to
- **Usage**: Defined in `app.py`
- **Example**: `0.0.0.0` allows external connections

### `port`
- **Default**: `5092`
- **Description**: Port number for the FastAPI server
- **Usage**: Defined in `app.py`
- **Example**: Access API at `http://localhost:5092`

### `threads`
- **Default**: `8`
- **Description**: Number of threads for Waitress WSGI server
- **Usage**: Optimized for 8 P-cores on hybrid CPUs
- **Example**: Adjust based on your CPU core count

### `API_KEY` 🔐
- **Default**: `None` (authentication disabled)
- **Description**: API key for authentication (optional but recommended for production)
- **Usage**: Set to enable bearer token authentication for `/v1/audio/transcriptions` endpoint
- **Format**: Any string (recommend using secure random string)
- **Example**:
  ```bash
  export API_KEY="your-secret-api-key-here"
  # or
  export API_KEY="sk-$(openssl rand -hex 32)"
  ```
- **Client usage**:
  ```python
  from openai import OpenAI

  client = OpenAI(
      base_url="http://127.0.0.1:5092/v1",
      api_key="your-secret-api-key-here"  # Must match API_KEY env var
  )
  ```
- **cURL example**:
  ```bash
  curl -X POST http://localhost:5092/v1/audio/transcriptions \
    -H "Authorization: Bearer your-secret-api-key-here" \
    -F "file=@audio.mp3" \
    -F "model=parakeet-tdt-0.6b-v3"
  ```
- **Security Note**:
  - ⚠️ If not set, API is **completely open** (no authentication)
  - ✅ **Highly recommended** for production deployments
  - 🔒 Use HTTPS in production to protect API key in transit

---

## Model & Cache Configuration

### `HF_HOME`
- **Default**: `{ROOT_DIR}/models` or `/app/models` (Docker)
- **Description**: HuggingFace model cache directory
- **Usage**: Set in `app.py` and Dockerfiles
- **Docker Volume**: Persisted in `parakeet-models` volume
- **Example**:
  ```bash
  export HF_HOME=/path/to/models
  ```

### `HF_HUB_CACHE`
- **Default**: `{ROOT_DIR}/models` or `/app/models` (Docker)
- **Description**: HuggingFace Hub cache directory
- **Usage**: Set in `app.py` and Dockerfiles
- **Example**:
  ```bash
  export HF_HUB_CACHE=/path/to/models
  ```

### `HF_HUB_DISABLE_SYMLINKS_WARNING`
- **Default**: `"true"`
- **Description**: Disables symlink warnings from HuggingFace Hub
- **Usage**: Set in `app.py` and Dockerfiles
- **Why**: Prevents warning spam during model loading

---

## Docker & Runtime Configuration

### `DEBIAN_FRONTEND`
- **Default**: `noninteractive`
- **Description**: Prevents interactive prompts during Docker build
- **Usage**: Set in Dockerfiles during `apt-get` operations
- **Context**: Build-time only

### `PYTHONUNBUFFERED`
- **Default**: `1`
- **Description**: Forces Python stdout/stderr to be unbuffered
- **Usage**: Set in Dockerfiles
- **Why**: Ensures real-time log output in Docker containers
- **Example**:
  ```bash
  export PYTHONUNBUFFERED=1
  ```

### `PATH` (Windows Only)
- **Default**: Extends system PATH with FFmpeg location
- **Description**: Adds FFmpeg to system PATH on Windows
- **Usage**: Set in `app.py` for Windows (`sys.platform == "win32"`)
- **Example**:
  ```python
  os.environ["PATH"] = ROOT_DIR + f";{ROOT_DIR}/ffmpeg;" + os.environ["PATH"]
  ```

---

## GPU Configuration (GPU Build Only)

These variables are only used in the GPU Docker build (`Dockerfile.gpu`).

### `NVIDIA_VISIBLE_DEVICES`
- **Default**: `all`
- **Description**: Controls which GPUs are visible to the container
- **Usage**: Set in `Dockerfile.gpu`
- **Example**:
  ```bash
  # Use specific GPU
  export NVIDIA_VISIBLE_DEVICES=0

  # Use multiple GPUs
  export NVIDIA_VISIBLE_DEVICES=0,1
  ```

### `NVIDIA_DRIVER_CAPABILITIES`
- **Default**: `compute,utility`
- **Description**: Specifies which driver capabilities to enable
- **Usage**: Set in `Dockerfile.gpu`
- **Options**: `compute`, `utility`, `graphics`, `video`, `display`
- **Why**: `compute` for CUDA, `utility` for nvidia-smi

---

## Audio Processing Configuration

These are application-level constants defined in `app.py` that control audio processing behavior.

### `CHUNK_MINUTE`
- **Default**: `1.5`
- **Description**: Target chunk duration in minutes (90 seconds)
- **Usage**: Used to split long audio files
- **Why**: Optimal balance between accuracy and processing speed

### `SILENCE_THRESHOLD`
- **Default**: `"-40dB"`
- **Description**: Silence detection threshold for intelligent chunking
- **Usage**: Used by FFmpeg's silencedetect filter
- **Tuning**: Lower values (e.g., `-50dB`) = more sensitive to quiet sounds

### `SILENCE_MIN_DURATION`
- **Default**: `0.5`
- **Description**: Minimum silence duration in seconds to detect
- **Usage**: Prevents splitting on brief pauses
- **Example**: 0.5s = splits only on silences longer than 500ms

### `SILENCE_SEARCH_WINDOW`
- **Default**: `30.0`
- **Description**: Search window in seconds around target split point
- **Usage**: Used to find optimal silence-based split points
- **Why**: Balances chunk uniformity with natural speech boundaries

### `SILENCE_DETECT_TIMEOUT`
- **Default**: `300`
- **Description**: Timeout for silence detection in seconds
- **Usage**: Prevents hanging on very long audio files
- **Example**: 300s = 5 minute timeout

### `MIN_SPLIT_GAP`
- **Default**: `5.0`
- **Description**: Minimum gap between split points in seconds
- **Usage**: Prevents creating too-short chunks
- **Why**: Ensures each chunk has enough content for accurate transcription

---

## ONNX Runtime Configuration

These are programmatically configured in `app.py` via `onnxruntime.SessionOptions`.

### Session Options (CPU)
- **`intra_op_num_threads`**: `8` - Threads for parallelizing ops within a node
- **`inter_op_num_threads`**: `1` - Threads for parallelizing across nodes
- **`execution_mode`**: `ORT_SEQUENTIAL` - Sequential execution
- **`graph_optimization_level`**: `ORT_ENABLE_ALL` - Enable all optimizations

### Execution Providers
- **CPU Build**: `["CPUExecutionProvider"]`
- **GPU Build**: Priority order:
  1. `TensorrtExecutionProvider` (if available)
  2. `CUDAExecutionProvider` (if available)
  3. `CPUExecutionProvider` (fallback)

---

## Docker Compose Environment

When using `docker-compose.yml`, these can be overridden:

```yaml
environment:
  - HF_HOME=/app/models
  - HF_HUB_CACHE=/app/models
  - PYTHONUNBUFFERED=1
```

---

## Quick Reference Table

| Variable | Default | Scope | Modifiable |
|----------|---------|-------|------------|
| `host` | `0.0.0.0` | App | ✅ Edit `app.py` |
| `port` | `5092` | App | ✅ Edit `app.py` |
| `threads` | `8` | App | ✅ Edit `app.py` |
| `API_KEY` 🔐 | `None` | Runtime | ✅ ENV (recommended) |
| `HF_HOME` | `/app/models` | Runtime | ✅ ENV or Docker volume |
| `HF_HUB_CACHE` | `/app/models` | Runtime | ✅ ENV or Docker volume |
| `PYTHONUNBUFFERED` | `1` | Runtime | ✅ ENV |
| `CHUNK_MINUTE` | `1.5` | App | ✅ Edit `app.py` |
| `SILENCE_THRESHOLD` | `-40dB` | App | ✅ Edit `app.py` |
| `NVIDIA_VISIBLE_DEVICES` | `all` | Docker GPU | ✅ ENV or docker run |
| `intra_op_num_threads` | `8` | ONNX | ⚙️ Programmatic |

---

## Examples

### Running with Custom Configuration

**Docker (CPU) with API Key**:
```bash
docker run -d \
  -p 5092:5092 \
  -e API_KEY="your-secret-api-key-here" \
  -e HF_HOME=/custom/models \
  -e PYTHONUNBUFFERED=1 \
  -v my-models:/custom/models \
  parakeet-tdt:cpu
```

**Docker (GPU) with API Key**:
```bash
docker run -d \
  -p 5092:5092 \
  --gpus all \
  -e API_KEY="your-secret-api-key-here" \
  -e NVIDIA_VISIBLE_DEVICES=0 \
  -e HF_HOME=/app/models \
  -v my-models:/app/models \
  parakeet-tdt:gpu
```

**Local Python with API Key**:
```bash
export API_KEY="your-secret-api-key-here"
export HF_HOME=/path/to/models
export HF_HUB_CACHE=/path/to/models
python app.py
```

---

## Notes

- **Model Download**: On first run, models (~600MB) are downloaded to `HF_HOME`
- **Persistent Storage**: Use Docker volumes to persist models across container restarts
- **Performance**: Adjust `threads` and `intra_op_num_threads` based on your CPU
- **GPU Memory**: GPU build requires ~4GB VRAM for optimal performance
- **CPU Optimization**: For hybrid CPUs (Intel 12th-14th gen), pin to P-cores for best performance
