# Updates Summary - API Key & Latest Tag Fixes

## ✅ Changes Made

### 1. ✅ API Key Authentication Added

**Files Modified:**
- `app.py` - Added API key authentication support
- `docs/ENVIRONMENT.md` - Documented API_KEY environment variable
- `docker-compose.yml` - Added API_KEY environment variable

**What Changed:**

#### app.py
```python
# New configuration variable
API_KEY = None  # Loaded from environment variable

# Load from environment at startup
API_KEY = os.environ.get("API_KEY", None)
if API_KEY:
    print(f"🔐 API Key authentication enabled")
else:
    print(f"⚠️  API Key authentication disabled")

# New authentication function
def check_api_key():
    """Check if API key is valid"""
    if not API_KEY:
        return True  # Authentication disabled

    auth_header = request.headers.get("Authorization", "")
    if auth_header.startswith("Bearer "):
        provided_key = auth_header[7:]
        return provided_key == API_KEY

    return False

# Applied to /v1/audio/transcriptions endpoint
@app.route("/v1/audio/transcriptions", methods=["POST"])
def transcribe_audio():
    if not check_api_key():
        return jsonify({
            "error": {
                "message": "Invalid API key",
                "type": "invalid_request_error",
                "code": "invalid_api_key"
            }
        }), 401
    # ... rest of function
```

**How to Use:**

**Without API Key (Open API - Default):**
```bash
# No API key needed
docker run -d -p 5092:5092 parakeet-tdt:cpu

# Client usage
curl -X POST http://localhost:5092/v1/audio/transcriptions \
  -F "file=@audio.mp3" \
  -F "model=parakeet-tdt-0.6b-v3"
```

**With API Key (Secured - Recommended for Production):**
```bash
# Set API key
docker run -d -p 5092:5092 \
  -e API_KEY="your-secret-api-key-here" \
  parakeet-tdt:cpu

# Client usage with OpenAI SDK
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:5092/v1",
    api_key="your-secret-api-key-here"  # Must match API_KEY env var
)

# Client usage with cURL
curl -X POST http://localhost:5092/v1/audio/transcriptions \
  -H "Authorization: Bearer your-secret-api-key-here" \
  -F "file=@audio.mp3" \
  -F "model=parakeet-tdt-0.6b-v3"
```

**Generate Secure API Key:**
```bash
# Linux/Mac
export API_KEY="sk-$(openssl rand -hex 32)"

# Or use any secure random string
export API_KEY="your-custom-secret-key"
```

---

### 2. ✅ OpenAI API Compatibility Confirmed

**Both CPU and GPU builds support the same OpenAI-compatible API:**

✅ **Endpoint**: `/v1/audio/transcriptions`
✅ **Compatible with**: OpenAI Python SDK, openai-compatible clients
✅ **Response formats**: `json`, `text`, `srt`, `vtt`, `verbose_json`

**The ONLY difference between CPU and GPU builds:**
- **CPU**: Uses `CPUExecutionProvider` (ONNX Runtime)
- **GPU**: Uses `CUDAExecutionProvider` or `TensorrtExecutionProvider` (ONNX Runtime)

**Same API, same endpoints, same functionality!**

**Example with OpenAI SDK:**
```python
from openai import OpenAI

# Works with both CPU and GPU builds
client = OpenAI(
    base_url="http://127.0.0.1:5092/v1",
    api_key="your-api-key-here"  # or "sk-no-key-required" if API_KEY not set
)

with open("audio.mp3", "rb") as audio_file:
    transcript = client.audio.transcriptions.create(
        model="parakeet-tdt-0.6b-v3",  # or "whisper-1" for compatibility
        file=audio_file,
        response_format="text"
    )

print(transcript)
```

---

### 3. ✅ Latest Tag Fixed in CD Pipeline

**File Modified:**
- `.github/workflows/docker-publish.yml`

**What Changed:**
```yaml
# BEFORE (latest only on branch push)
type=raw,value=latest,enable={{is_default_branch}}

# AFTER (latest on ALL builds including version tags)
type=raw,value=latest
```

**Result:**

**When you push to main:**
```
✅ latest-cpu
✅ latest-gpu
✅ main-cpu
✅ main-gpu
```

**When you push version tag (e.g., v3.1.0):**
```
✅ v3.1.0-cpu       (exact version)
✅ v3.1.0-gpu
✅ v3.1-cpu         (minor version)
✅ v3.1-gpu
✅ v3-cpu           (major version)
✅ v3-gpu
✅ latest-cpu       ⭐ NOW INCLUDED!
✅ latest-gpu       ⭐ NOW INCLUDED!
```

**This means `latest` ALWAYS points to the newest release! 🎉**

---

## 🎯 Complete Usage Examples

### Example 1: Development (No API Key)

```bash
# Start server
docker run -d -p 5092:5092 \
  --name parakeet \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# Test
curl -X POST http://localhost:5092/v1/audio/transcriptions \
  -F "file=@test.mp3" \
  -F "model=parakeet-tdt-0.6b-v3"
```

### Example 2: Production (With API Key)

```bash
# Generate secure API key
API_KEY="sk-$(openssl rand -hex 32)"

# Start server with API key
docker run -d -p 5092:5092 \
  --name parakeet \
  -e API_KEY="$API_KEY" \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# Test with authentication
curl -X POST http://localhost:5092/v1/audio/transcriptions \
  -H "Authorization: Bearer $API_KEY" \
  -F "file=@test.mp3" \
  -F "model=parakeet-tdt-0.6b-v3"
```

### Example 3: GPU with API Key

```bash
# Start GPU server with authentication
docker run -d -p 5092:5092 \
  --name parakeet-gpu \
  --gpus all \
  -e API_KEY="your-secret-key" \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu

# Python client
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:5092/v1",
    api_key="your-secret-key"
)

# Works exactly like OpenAI's API!
transcript = client.audio.transcriptions.create(
    model="whisper-1",  # compatible name
    file=open("audio.mp3", "rb"),
    response_format="json"
)

print(transcript.text)
```

### Example 4: Docker Compose with API Key

**Create `.env` file:**
```bash
API_KEY=your-secret-api-key-here
```

**Update docker-compose.yml:**
```yaml
services:
  parakeet-cpu:
    image: ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu
    ports:
      - "5092:5092"
    volumes:
      - parakeet-models:/app/models
    environment:
      - API_KEY=${API_KEY}
```

**Run:**
```bash
docker compose up -d
```

---

## 🔐 Security Best Practices

### ✅ DO:
- ✅ Set `API_KEY` environment variable in production
- ✅ Use strong, random API keys: `openssl rand -hex 32`
- ✅ Use HTTPS/TLS in production (reverse proxy with nginx/caddy)
- ✅ Store API keys in environment variables, never in code
- ✅ Use Docker secrets for API keys in Swarm/Kubernetes
- ✅ Rotate API keys periodically

### ❌ DON'T:
- ❌ Use API without authentication in production
- ❌ Commit API keys to git
- ❌ Use weak/guessable API keys
- ❌ Share API keys in plaintext
- ❌ Use HTTP in production (use HTTPS)

---

## 📊 Version Tagging Strategy

### Semantic Versioning

```
vMAJOR.MINOR.PATCH
```

**Examples:**

| Git Tag | Docker Tags Created |
|---------|---------------------|
| `v3.1.0` | `v3.1.0-cpu`, `v3.1.0-gpu`, `v3.1-cpu`, `v3.1-gpu`, `v3-cpu`, `v3-gpu`, `latest-cpu`, `latest-gpu` |
| `v3.1.1` | `v3.1.1-cpu`, `v3.1.1-gpu`, `v3.1-cpu`, `v3.1-gpu`, `v3-cpu`, `v3-gpu`, `latest-cpu`, `latest-gpu` |
| `v3.2.0` | `v3.2.0-cpu`, `v3.2.0-gpu`, `v3.2-cpu`, `v3.2-gpu`, `v3-cpu`, `v3-gpu`, `latest-cpu`, `latest-gpu` |
| `v4.0.0` | `v4.0.0-cpu`, `v4.0.0-gpu`, `v4.0-cpu`, `v4.0-gpu`, `v4-cpu`, `v4-gpu`, `latest-cpu`, `latest-gpu` |

**What users should pull:**

| User Type | Recommended Tag | Why |
|-----------|----------------|-----|
| Production (stable) | `v3.1.0-cpu` | Exact version, no surprises |
| Production (auto-patch) | `v3.1-cpu` | Auto-updates patches (3.1.1, 3.1.2...) |
| Development | `latest-cpu` | Always get newest features |
| Testing | `v3.2.0-beta.1-cpu` | Pre-release versions |

---

## 🚀 Next Steps

### 1. Commit Changes

```bash
git add .
git commit -m "feat: add API key authentication and fix latest tag

- Add API_KEY environment variable for optional authentication
- Update documentation with API key usage
- Fix CD pipeline to always tag latest on version releases
- Update docker-compose with API_KEY support"

git push origin main
```

### 2. Create First Secure Release

```bash
# Tag version
git tag v3.1.0
git push origin v3.1.0

# Wait for GitHub Actions to build and publish
# Images will be available as:
# - ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-cpu
# - ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:v3.1.0-gpu
# - ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu ⭐
# - ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-gpu ⭐
```

### 3. Deploy with API Key

```bash
# Pull latest
docker pull ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# Run with API key
docker run -d -p 5092:5092 \
  -e API_KEY="$(openssl rand -hex 32)" \
  -v parakeet-models:/app/models \
  ghcr.io/alienvsrobot/parakeet-tdt-0.6b-v3-fastapi-openai:latest-cpu

# Save the API key for your clients!
```

---

## 📝 Files Modified

### Core Application
- ✅ `app.py` - API key authentication logic

### Configuration
- ✅ `docker-compose.yml` - API_KEY environment variable

### Documentation
- ✅ `docs/ENVIRONMENT.md` - API_KEY documentation

### CI/CD
- ✅ `.github/workflows/docker-publish.yml` - Latest tag fix

### New Documentation
- ✅ `UPDATES_SUMMARY.md` - This file

---

## ✨ Summary

✅ **API Key Authentication**: Optional but recommended for production
✅ **OpenAI Compatibility**: Both CPU and GPU builds support OpenAI API
✅ **Latest Tag**: Now always points to newest release
✅ **Security**: Production-ready with authentication support
✅ **Backward Compatible**: Works without API key for development

**All requests addressed! Ready to deploy! 🚀**
