# Deployment Guide

Complete guide for deploying Parakeet TDT in production environments.

## Table of Contents
- [Quick Start](#quick-start)
- [Docker Deployment](#docker-deployment)
- [Production Deployment](#production-deployment)
- [CI/CD with GitHub Actions](#cicd-with-github-actions)
- [Performance Tuning](#performance-tuning)

---

## Quick Start

### Using Pre-built Images (Recommended)

Pull and run the latest release from GitHub Container Registry:

**CPU Version:**
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
docker run -d -p 5092:5092 \
  --name parakeet-cpu \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
```

**GPU Version:**
```bash
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
docker run -d -p 5092:5092 \
  --name parakeet-gpu \
  --gpus all \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
```

### Using Docker Compose

```bash
git clone https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai
cd parakeet-tdt-0.6b-v3-fastapi-openai

# CPU
docker compose up parakeet-cpu -d

# GPU (requires NVIDIA Container Toolkit)
docker compose up parakeet-gpu -d
```

---

## Docker Deployment

### Building from Source

**CPU Build:**
```bash
docker build -f Dockerfile.cpu -t parakeet-tdt:cpu .
docker run -d -p 5092:5092 -v parakeet-models:/app/models parakeet-tdt:cpu
```

**GPU Build:**
```bash
docker build -f Dockerfile.gpu -t parakeet-tdt:gpu .
docker run -d -p 5092:5092 --gpus all -v parakeet-models:/app/models parakeet-tdt:gpu
```

### Version Tags

Images are tagged with semantic versioning:

```bash
# Latest stable
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# Specific version
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-cpu

# Major.Minor version
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1-cpu
```

### Custom Port Mapping

```bash
docker run -d -p 8080:5092 \
  --name parakeet \
  parakeet-tdt:cpu
```

Access at: `http://localhost:8080`

---

## Production Deployment

### Using Docker Swarm

**1. Initialize Swarm:**
```bash
docker swarm init
```

**2. Create Stack File (`stack.yml`):**
```yaml
version: '3.8'

services:
  parakeet:
    image: ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
    ports:
      - "5092:5092"
    volumes:
      - parakeet-models:/app/models
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        max_attempts: 3
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:5092/health')"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  parakeet-models:
```

**3. Deploy Stack:**
```bash
docker stack deploy -c stack.yml parakeet
```

### Using Kubernetes

**1. Create Deployment (`k8s-deployment.yaml`):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: parakeet-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: parakeet
  template:
    metadata:
      labels:
        app: parakeet
    spec:
      containers:
      - name: parakeet
        image: ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
        ports:
        - containerPort: 5092
        volumeMounts:
        - name: models
          mountPath: /app/models
        livenessProbe:
          httpGet:
            path: /health
            port: 5092
          initialDelaySeconds: 120
          periodSeconds: 30
        resources:
          requests:
            memory: "2Gi"
            cpu: "2"
          limits:
            memory: "4Gi"
            cpu: "4"
      volumes:
      - name: models
        persistentVolumeClaim:
          claimName: parakeet-models-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: parakeet-service
spec:
  selector:
    app: parakeet
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5092
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: parakeet-models-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
```

**2. Apply Configuration:**
```bash
kubectl apply -f k8s-deployment.yaml
```

### GPU Deployment on Kubernetes

**Requires NVIDIA GPU Operator**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: parakeet-gpu-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: parakeet-gpu
  template:
    metadata:
      labels:
        app: parakeet-gpu
    spec:
      containers:
      - name: parakeet
        image: ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu
        ports:
        - containerPort: 5092
        resources:
          limits:
            nvidia.com/gpu: 1
        volumeMounts:
        - name: models
          mountPath: /app/models
      volumes:
      - name: models
        persistentVolumeClaim:
          claimName: parakeet-models-pvc
```

---

## CI/CD with GitHub Actions

This repository includes automated CI/CD via GitHub Actions.

### Automatic Builds

Builds are triggered on:
- **Push to main**: Creates `latest` tag
- **Version tags**: Creates semantic version tags (e.g., `v3.1.0`)
- **Manual dispatch**: Custom version via workflow UI

### Creating a New Release

**1. Tag a new version:**
```bash
git tag v3.1.0
git push origin v3.1.0
```

**2. GitHub Actions will automatically:**
- Build CPU and GPU images
- Push to GitHub Container Registry
- Create GitHub Release with notes
- Tag as: `v3.1.0-cpu`, `v3.1.0-gpu`, `latest-cpu`, `latest-gpu`

### Manual Workflow Dispatch

1. Go to: **Actions → Docker Build and Publish → Run workflow**
2. Select branch: `main`
3. Enter version (optional): `v3.2.0`
4. Click **Run workflow**

---

## Performance Tuning

### CPU Optimization

**For Hybrid CPUs (Intel 12th-14th Gen):**

Pin to P-cores using `taskset`:
```bash
# Find CPU topology
lscpu --extended

# Run on P-cores only (0-7 for typical 8P+8E config)
docker run -d -p 5092:5092 \
  --cpuset-cpus="0-7" \
  --name parakeet-cpu \
  parakeet-tdt:cpu
```

**Adjust Thread Count:**

Edit `app.py`:
```python
threads = 16  # Increase for more cores
sess_options.intra_op_num_threads = 8  # Match P-core count
```

### GPU Optimization

**Limit GPU Memory:**
```bash
docker run -d -p 5092:5092 \
  --gpus '"device=0"' \
  -e NVIDIA_VISIBLE_DEVICES=0 \
  parakeet-tdt:gpu
```

**Multi-GPU Setup:**
```bash
docker run -d -p 5092:5092 \
  --gpus all \
  -e NVIDIA_VISIBLE_DEVICES=0,1 \
  parakeet-tdt:gpu
```

### Memory Limits

**Set Docker memory limits:**
```bash
docker run -d -p 5092:5092 \
  --memory="4g" \
  --memory-swap="4g" \
  parakeet-tdt:cpu
```

### Load Balancing

**Using nginx:**

```nginx
upstream parakeet_backend {
    least_conn;
    server 127.0.0.1:5092;
    server 127.0.0.1:5093;
    server 127.0.0.1:5094;
}

server {
    listen 80;
    location / {
        proxy_pass http://parakeet_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        client_max_body_size 2000M;
    }
}
```

---

## Monitoring

### Health Checks

```bash
# Docker
curl http://localhost:5092/health

# Expected response
{"status":"healthy","model":"parakeet-tdt-0.6b-v3","speedup":"20.7x"}
```

### Metrics Endpoint

```bash
curl http://localhost:5092/metrics
```

**Response:**
```json
{
  "cpu_percent": 45.2,
  "ram_percent": 23.5,
  "ram_used_gb": 3.76,
  "ram_total_gb": 16.0
}
```

### Logging

**View Docker logs:**
```bash
# Real-time
docker logs -f parakeet-cpu

# Last 100 lines
docker logs --tail 100 parakeet-cpu
```

---

## Troubleshooting

### Container Won't Start

**Check logs:**
```bash
docker logs parakeet-cpu
```

**Common issues:**
- Model download timeout (first run takes ~60s)
- Port already in use (change `-p 5092:5092` to another port)
- Insufficient memory (requires ~2GB RAM for CPU)

### GPU Not Detected

**Verify NVIDIA Container Toolkit:**
```bash
docker run --rm --gpus all nvidia/cuda:12.1.1-base-ubuntu22.04 nvidia-smi
```

**Should show your GPU(s).**

### Out of Memory

**CPU:**
- Reduce `threads` in `app.py`
- Limit Docker memory: `--memory="2g"`

**GPU:**
- Use smaller batch size
- Limit GPU memory
- Requires ~4GB VRAM minimum

### Model Download Fails

**Manual download:**
```bash
docker run -it --rm \
  -v parakeet-models:/app/models \
  parakeet-tdt:cpu bash

# Inside container
python -c "import onnx_asr; onnx_asr.load_model('nemo-parakeet-tdt-0.6b-v3', quantization='int8')"
```

---

## Security Best Practices

1. **Run as non-root**: Images already use `parakeet` user
2. **Read-only filesystem**: Add `--read-only` flag
3. **Drop capabilities**: Add `--cap-drop=ALL`
4. **Network isolation**: Use Docker networks
5. **Secrets management**: Use Docker secrets for sensitive data

**Example secure deployment:**
```bash
docker run -d \
  --read-only \
  --cap-drop=ALL \
  --tmpfs /tmp \
  --tmpfs /app/temp_uploads \
  -p 5092:5092 \
  -v parakeet-models:/app/models:ro \
  parakeet-tdt:cpu
```

---

## Support

- **Issues**: https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/issues
- **Documentation**: https://github.com/ALIENvsROBOT/parakeet-tdt-0.6b-v3-fastapi-openai/tree/main/docs
- **Original Project**: https://github.com/groxaxo/parakeet-tdt-0.6b-v3-fastapi-openai
