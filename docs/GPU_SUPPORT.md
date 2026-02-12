# GPU Support 🎮

Parakeet TDT supports multiple GPU backends with automatic detection for optimal performance.

## Supported GPU Backends

### 1. NVIDIA CUDA (Dockerfile.cuda-gpu) 🟢

**Best for:** NVIDIA GPUs (GeForce RTX, Tesla, Quadro)

- **Base Image:** `nvidia/cuda:12.6.2-cudnn-runtime-ubuntu22.04`
- **Runtime:** ONNX Runtime with CUDAExecutionProvider
- **Requirements:**
  - NVIDIA GPU with Compute Capability 6.0+ (Pascal or newer)
  - NVIDIA Container Toolkit installed
  - CUDA 12.6+ and cuDNN 9

**Supported Platforms:**
- ✅ Linux
- ✅ Windows with WSL2
- ❌ macOS (Docker Desktop doesn't support GPU passthrough)

**Docker Run:**
```bash
docker run -d -p 5092:5092 --gpus all \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cuda-gpu
```

**Docker Compose:**
```bash
docker compose up parakeet-cuda-gpu -d
```

### 2. AMD ROCm (Dockerfile.rocm-gpu) 🔴

**Best for:** AMD GPUs (Radeon RX, Instinct)

- **Base Image:** `rocm/pytorch:rocm6.2_ubuntu22.04_py3.10_pytorch_release_2.3.0` (10+ GB)
- **Runtime:** ONNX Runtime with ROCmExecutionProvider
- **Requirements:**
  - AMD GPU with ROCm support (RDNA 2+, CDNA)
  - ROCm drivers 6.0+ installed on host

**Supported Platforms:**
- ✅ Linux (native)
- ⚠️ Windows with WSL2 (experimental, requires ROCm on Linux)
- ❌ macOS

**⚠️ Build Locally (Not in Registry):**

Due to the large base image size (10+ GB), ROCm builds exceed GitHub Actions runner disk space and are not included in automated releases.

**Build locally:**
```bash
git clone https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai
cd parakeet-tdt-0.6b-v3-fastapi-openai
docker build -f Dockerfile.rocm-gpu -t parakeet-tdt:rocm-gpu .
```

**Run:**
```bash
docker run -d -p 5092:5092 \
  --device=/dev/kfd --device=/dev/dri \
  parakeet-tdt:rocm-gpu
```

**Docker Compose:**
```bash
docker compose up parakeet-rocm-gpu -d
```

**Common AMD GPU Configurations:**
```bash
# RX 6000/7000 series (RDNA 2/3) - may need GFX version override
docker run -d -p 5092:5092 \
  --device=/dev/kfd --device=/dev/dri \
  -e HSA_OVERRIDE_GFX_VERSION=11.0.0 \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-rocm-gpu
```

### 3. CPU Fallback (Dockerfile.cpu) 💻

**Best for:** Systems without GPU or macOS

- **Performance:** 10.6x real-time speedup with INT8 quantization
- **Optimization:** Multi-threaded execution, optimized for 8-core CPUs
- **Memory:** ~1.5 GB RAM

**Supported Platforms:**
- ✅ Linux (all architectures)
- ✅ Windows
- ✅ macOS (Intel & Apple Silicon via Rosetta)

**Docker Run:**
```bash
docker run -d -p 5092:5092 \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

**Docker Compose:**
```bash
docker compose up parakeet-cpu -d
```

## Auto-Detection Logic

The application automatically detects and uses the best available GPU provider:

```
Priority: ROCm → CUDA → CPU
```

**Detection Flow:**
1. Check for `ROCmExecutionProvider` (AMD GPU)
2. If not found, check for `CUDAExecutionProvider` (NVIDIA GPU)
3. If no GPU, fallback to `CPUExecutionProvider`

**Example Output:**
```
🔍 Available ONNX Runtime providers: ['CUDAExecutionProvider', 'CPUExecutionProvider']
🎮 NVIDIA GPU detected - using CUDA acceleration
🚀 Provider priority: CUDAExecutionProvider → CPUExecutionProvider
✅ Model loaded successfully with NVIDIA CUDA acceleration!
```

## Mac Users ⚠️

**Important:** Docker Desktop on macOS does **not** support GPU passthrough for any GPU type (neither Intel integrated graphics, AMD discrete, nor Apple Silicon GPUs).

### Why?

- Docker runs Linux containers in a lightweight VM
- macOS doesn't expose GPU devices to the VM
- Metal API (macOS graphics) can't be forwarded to Linux containers

### Recommendation for Mac Users

Use the **CPU version**, which provides excellent performance:
- ✅ 10.6x real-time speedup (same as GPU for this model)
- ✅ Works natively on Intel Macs
- ✅ Works on Apple Silicon via Rosetta 2
- ✅ No additional setup required

```bash
docker run -d -p 5092:5092 \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

## Troubleshooting

### NVIDIA GPU Issues

**Problem:** GPU not detected, fallback to CPU
```
💻 No GPU detected - using CPU
```

**Solutions:**
1. Install NVIDIA Container Toolkit:
   ```bash
   # Ubuntu/Debian
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
   distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
   curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
     sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   sudo nvidia-ctk runtime configure --runtime=docker
   sudo systemctl restart docker
   ```

2. Verify NVIDIA driver:
   ```bash
   nvidia-smi
   ```

3. Test GPU access:
   ```bash
   docker run --rm --gpus all nvidia/cuda:12.6.2-base-ubuntu22.04 nvidia-smi
   ```

### AMD GPU Issues

**Problem:** ROCm not detected

**Solutions:**
1. Install ROCm drivers on host (Linux):
   ```bash
   # Ubuntu 22.04
   wget https://repo.radeon.com/amdgpu-install/6.2.4/ubuntu/jammy/amdgpu-install_6.2.60204-1_all.deb
   sudo apt-get install ./amdgpu-install_6.2.60204-1_all.deb
   sudo amdgpu-install --usecase=rocm
   ```

2. Verify ROCm installation:
   ```bash
   rocm-smi
   ```

3. Check device permissions:
   ```bash
   ls -l /dev/kfd /dev/dri
   sudo usermod -a -G video,render $USER
   ```

### cuDNN Version Mismatch (CUDA)

**Problem:**
```
error while loading shared libraries: libcudnn.so.9: cannot open shared object file
```

**Solution:** Use the cuda-gpu image which includes cuDNN 9:
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cuda-gpu
```

## Performance Comparison

| Backend | Speed (Real-time) | Memory | Best Use Case |
|---------|-------------------|--------|---------------|
| NVIDIA CUDA | ~15-20x | ~2 GB VRAM | High throughput, batch processing |
| AMD ROCm | ~12-18x | ~2 GB VRAM | AMD GPU systems |
| CPU (INT8) | ~10.6x | ~1.5 GB RAM | Mac, no GPU, cost-effective |

## Windows WSL2 Setup

For Windows users wanting GPU acceleration:

### NVIDIA GPU (Tested ✅)

1. Install WSL2 with Ubuntu
2. Install NVIDIA drivers on Windows (latest Game Ready or Studio drivers)
3. Install NVIDIA Container Toolkit in WSL2:
   ```bash
   wsl -d ubuntu-docker
   # Follow NVIDIA Container Toolkit instructions above
   ```
4. Run Docker:
   ```bash
   docker run -d -p 5092:5092 --gpus all \
     ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cuda-gpu
   ```

### AMD GPU (Experimental ⚠️)

AMD GPU passthrough to WSL2 is experimental and may not work reliably. Recommend using CPU version on Windows with AMD GPUs.

## Building Locally

```bash
# NVIDIA CUDA GPU
docker build -f Dockerfile.cuda-gpu -t parakeet-tdt:cuda-gpu .

# AMD ROCm GPU
docker build -f Dockerfile.rocm-gpu -t parakeet-tdt:rocm-gpu .

# CPU
docker build -f Dockerfile.cpu -t parakeet-tdt:cpu .
```

## See Also

- [ENVIRONMENT.md](ENVIRONMENT.md) - Environment variable configuration
- [DEPLOYMENT.md](DEPLOYMENT.md) - Production deployment guide
- [Docker Compose](../docker-compose.yml) - Multi-variant setup
