# Episode 10: Write Dockerfile from Scratch — Zero to Hero
> 🎯 **Goal**: Master writing Dockerfiles from zero — understanding every instruction, best practice, and optimization pattern for production-ready container images.

📺 **Watch on YouTube**: [Episode 10: Write Dockerfile](https://youtu.be/ivYMOuJhT5o)

---

## 📋 Table of Contents
- [Why Write Dockerfiles?](#why-write-dockerfiles)
- [Prerequisites](#prerequisites)
- [Dockerfile Instructions Deep Dive](#dockerfile-instructions-deep-dive)
- [Step-by-Step Writing Guide](#step-by-step-writing-guide)
- [Advanced Patterns](#advanced-patterns)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Video Timestamps](#video-timestamps)
- [Resources](#resources)
- [What's Next](#whats-next)

---

## Why Write Dockerfiles?

| Concept | Purpose | Why It Matters |
|---------|---------|---------------|
| **Base Image Selection** | Foundation for your container | Impacts size, security, and compatibility |
| **Layer Caching** | Optimizes build speed | Reduces rebuild time from minutes to seconds |
| **WORKDIR** | Sets execution context | Prevents path-related errors and improves clarity |
| **COPY vs ADD** | File transfer strategies | COPY is explicit; ADD has extra features (use wisely) |
| **RUN with Chaining** | Efficient layer creation | Reduces image size and build time |
| **EXPOSE** | Documents intended ports | Helps orchestration tools and documentation |
| **CMD vs ENTRYPOINT** | Container startup behavior | Determines how your app runs in production |
| **.dockerignore** | Excludes unnecessary files | Smaller images, faster builds, better security |

> 💡 **Chef's Recipe Analogy**: Writing a Dockerfile is like creating a recipe for a dish. The base image is your kitchen setup. Each instruction is a step in the cooking process. Layer caching is like prepping ingredients ahead of time. When you write it yourself, you control every ingredient, every technique, and every outcome.

---

## Prerequisites

Before starting Episode 10, ensure you have:
- ✅ Docker and Docker Compose installed (from Episode 3)
- ✅ Basic understanding of Linux commands and file structure
- ✅ A working `app.py` and `requirements.txt` (from Episode 9)
- ✅ VS Code with Docker extension (optional but helpful)

---

## Dockerfile Instructions Deep Dive

### Core Instructions (Every Dockerfile Needs These)

| Instruction | Purpose | Example | Best Practice |
|-------------|---------|---------|--------------|
| `FROM` | Sets the base image | `FROM python:3.12-slim` | Use specific tags, prefer `-slim` or `-alpine` for size |
| `WORKDIR` | Sets working directory | `WORKDIR /app` | Use absolute paths; creates directory if missing |
| `COPY` | Copies files from host to image | `COPY requirements.txt .` | Copy dependency files first for better caching |
| `RUN` | Executes commands during build | `RUN pip install -r requirements.txt` | Chain commands with `&&` to reduce layers |
| `EXPOSE` | Documents listening ports | `EXPOSE 8000` | Does not publish port; use `-p` in docker run |
| `CMD` | Default command when container starts | `CMD ["python", "app.py"]` | Use exec form `["cmd", "arg"]` for signal handling |

### Advanced Instructions (For Production)

| Instruction | Purpose | Example | When to Use |
|-------------|---------|---------|------------|
| `ENV` | Sets environment variables | `ENV PYTHONUNBUFFERED=1` | For app configuration, not secrets |
| `ARG` | Build-time variables | `ARG VERSION=1.0` | For build metadata, not runtime config |
| `USER` | Sets runtime user | `USER nobody` | For security; avoid running as root |
| `HEALTHCHECK` | Defines health probe | `HEALTHCHECK CMD curl -f http://localhost:8000/ || exit 1` | For orchestration and monitoring |
| `LABEL` | Adds metadata | `LABEL maintainer="you@example.com"` | For documentation and tooling |

### COPY vs ADD: Know the Difference

```dockerfile
# ✅ COPY: Explicit, predictable, preferred for most cases
COPY requirements.txt .
COPY . .

# ⚠️ ADD: Has extra features (URL download, tar extraction) — use intentionally
ADD https://example.com/file.tar.gz /tmp/  # Downloads and extracts
ADD config.tar.gz /etc/config/              # Auto-extracts tar files
```

> 💡 Rule of thumb: Use `COPY` unless you specifically need `ADD`'s extra features.

---

## Step-by-Step Writing Guide

### Step 1: Start with a Minimal Base Image
```dockerfile
FROM python:3.12-slim
```
**Why `python:3.12-slim`?**
- `3.12`: Specific Python version for reproducibility
- `-slim`: Debian-based but without unnecessary packages (~50% smaller than default)
- Avoid `latest` — it changes over time and breaks reproducibility

### Step 2: Set the Working Directory
```dockerfile
WORKDIR /app
```
**Why `/app`?**
- Convention for application code in containers
- Creates the directory if it doesn't exist
- All subsequent relative paths are resolved from here

### Step 3: Copy Dependencies First (Layer Caching Optimization)
```dockerfile
COPY requirements.txt .
```
**Why copy dependencies before code?**
- Docker caches layers based on instruction + file content
- If `requirements.txt` doesn't change, Docker reuses the cached `pip install` layer
- Only when dependencies change does it re-run the install — saving significant build time

### Step 4: Install Dependencies with Caching Flags
```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```
**Why `--no-cache-dir`?**
- Prevents pip from storing downloaded packages in the image
- Reduces final image size by 100-300MB
- Safe because dependencies are already installed

### Step 5: Copy Application Code
```dockerfile
COPY . .
```
**Why after dependencies?**
- Code changes frequently; dependencies change rarely
- This order maximizes cache hits during development
- Ensure `.dockerignore` excludes `__pycache__`, `.git`, `venv/`, etc.

### Step 6: Document the Exposed Port
```dockerfile
EXPOSE 8000
```
**What EXPOSE does (and doesn't do):**
- ✅ Documents the port the app listens on
- ✅ Helps `docker run -P` auto-publish ports
- ❌ Does NOT publish the port — use `-p 8000:8000` for that
- ❌ Does NOT make the port accessible from outside Docker

### Step 7: Define the Startup Command
```dockerfile
CMD ["python", "app.py"]
```
**Why exec form `["cmd", "arg"]`?**
- Proper signal handling (Ctrl+C, SIGTERM)
- Python runs as PID 1, receives shutdown signals correctly
- Shell form `CMD python app.py` spawns a subshell — signals may not reach your app

### Step 8: (Optional) Add .dockerignore
Create a `.dockerignore` file in the same directory:
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
.venv/
env/
.env

# Git
.git/
.gitignore

# IDE
.vscode/
.idea/
*.swp
*.swo

# Docker
Dockerfile
docker-compose*.yml
!.dockerignore

# OS
.DS_Store
Thumbs.db
```
**Why .dockerignore matters:**
- Excludes unnecessary files from the build context
- Speeds up builds (less data to send to Docker daemon)
- Prevents accidental inclusion of secrets or local configs

---

## Advanced Patterns

### Pattern 1: Multi-Stage Builds (For Compiled Languages)
```dockerfile
# Build stage
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Runtime stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```
**Why multi-stage?**
- Final image contains only the compiled binary, not build tools
- Reduces image size from 800MB+ to <20MB for Go apps
- Same pattern works for Rust, C++, Java, etc.

### Pattern 2: Non-Root User for Security
```dockerfile
FROM python:3.12-slim
WORKDIR /app

# Create non-root user
RUN useradd -m -u 1000 appuser

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appuser . .

USER appuser
EXPOSE 8000
CMD ["python", "app.py"]
```
**Why run as non-root?**
- Limits damage if container is compromised
- Required by many Kubernetes security policies
- Best practice for production deployments

### Pattern 3: Health Checks for Orchestration
```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/')" || exit 1
```
**Why HEALTHCHECK?**
- Docker and orchestrators (Kubernetes, ECS) use this to detect unhealthy containers
- Enables auto-restart of failed services
- Provides visibility into application health

---

## Verification

After writing your Dockerfile, verify it works correctly:

```bash
# 1. Check Dockerfile syntax (optional)
docker build --no-cache -t test-app .

# 2. Build the image (with cache for normal use)
docker build -t zero2hero-app .

# 3. Run the container with port mapping
docker run -p 8000:8000 --name test-container zero2hero-app

# 4. Test the endpoint
curl http://localhost:8000

# Expected output:
{
  "database": "Database connected successfully: 2024-04-14 12:34:56.789012+00:00",
  "message": "Hello from Docker Compose!",
  "redis": "Redis connected: world",
  "timestamp": 1713098096.789
}

# 5. Inspect the image layers (educational)
docker history zero2hero-app --no-trunc

# 6. Check image size
docker images zero2hero-app --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Expected Layer Structure (Educational)
```bash
$ docker history zero2hero-app --no-trunc | head -10
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
abc123def456        2 minutes ago       /bin/sh -c #(nop) CMD ["python" "app.py"]      0B                  built-in dockerfile command
def456ghi789        2 minutes ago       /bin/sh -c #(nop) EXPOSE 8000                   0B                  built-in dockerfile command
ghi789jkl012        3 minutes ago       /bin/sh -c #(nop) COPY dir:... in /app         15.2MB              Copy application code
jkl012mno345        5 minutes ago       /bin/sh -c pip install --no-cache-dir -r r...  45.8MB              Install dependencies
mno345pqr678        6 minutes ago       /bin/sh -c #(nop) COPY file:... in /app        1.2KB               Copy requirements.txt
pqr678stu901        6 minutes ago       /bin/sh -c #(nop) WORKDIR /app                  0B                  built-in dockerfile command
stu901vwx234        6 minutes ago       /bin/sh -c #(nop) FROM python:3.12-slim        125MB               Base image
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `failed to solve: failed to compute cache key` | Clear Docker build cache: `docker builder prune -f` |
| `pip install` takes too long on every build | Ensure `requirements.txt` is copied before app code for layer caching |
| Image is too large (>500MB) | Use `-slim` or `-alpine` base; add `--no-cache-dir` to pip; use `.dockerignore` |
| App doesn't receive SIGTERM on shutdown | Use exec form `CMD ["python", "app.py"]` not shell form |
| Permission denied writing files in container | Add `USER` instruction or fix file permissions with `--chown` in COPY |
| Environment variables not available at runtime | Use `ENV` for build-time, pass via `docker run -e` or docker-compose for runtime |
| Build context is slow (>30 seconds) | Add `.dockerignore` to exclude `.git`, `node_modules`, `__pycache__`, etc. |

### Quick Debug Commands
```bash
# View build output with more detail
docker build --progress=plain -t debug-app .

# Run container with interactive shell for debugging
docker run -it --rm zero2hero-app bash

# Test environment variables inside container
docker run --rm -e TEST_VAR=hello zero2hero-app python -c "import os; print(os.getenv('TEST_VAR'))"

# Check which files are in the image
docker run --rm zero2hero-app ls -la /app

# Compare image sizes between builds
docker images --filter=reference='zero2hero-app*' --format "table {{.ID}}\t{{.Size}}\t{{.CreatedAt}}"
```

---

## Video Timestamps

| Time | Topic | Key Takeaway |
|------|-------|-------------|
| 0:00 | Introduction + Why Write Dockerfiles Matters | Understanding composition > copying recipes |
| 1:00 | Chef's Recipe Analogy: Dockerfile Structure | Every instruction has a purpose in the build |
| 2:30 | Step 1: FROM — Choosing the Right Base Image | Specific tags + slim variants = reproducibility + size |
| 4:00 | Step 2: WORKDIR — Setting the Execution Context | Absolute paths prevent path-related errors |
| 5:15 | Step 3: COPY Dependencies First — Layer Caching | Order matters for build performance |
| 6:45 | Step 4: RUN with --no-cache-dir — Optimizing Size | Reduce image size by 100-300MB |
| 8:00 | Step 5: COPY Application Code — After Dependencies | Maximize cache hits during development |
| 9:15 | Step 6: EXPOSE — Documenting Ports | Helps orchestration, doesn't publish ports |
| 10:00 | Step 7: CMD in Exec Form — Proper Signal Handling | Ensures clean shutdowns in production |
| 11:00 | Bonus: .dockerignore — Exclude Unnecessary Files | Faster builds, smaller images, better security |
| 12:30 | Advanced: Multi-Stage Builds, Non-Root Users, HEALTHCHECK | Production-ready patterns for real-world use |
| 14:00 | Verification: Build, Run, Test, Inspect | Confirm your Dockerfile works as intended |
| 15:30 | Summary: What We've Accomplished | Dockerfile mastery from zero to hero |
| 16:30 | Sponsor Request + Thank You + Connect | Support the course, join the community |

---

## Resources

### Official Documentation
- [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Docker Scan (Security)](https://docs.docker.com/engine/scan/)

### Base Image Recommendations
| Language | Recommended Base | Size (approx) | Use Case |
|----------|-----------------|---------------|----------|
| Python | `python:3.12-slim` | 125MB | General Python apps |
| Python (minimal) | `python:3.12-alpine` | 50MB | Size-critical deployments |
| Node.js | `node:20-slim` | 180MB | General Node.js apps |
| Node.js (minimal) | `node:20-alpine` | 70MB | Size-critical deployments |
| Go | `golang:1.22-alpine` (build) + `alpine:latest` (runtime) | <20MB final | Compiled Go binaries |
| C/C++ | `gcc:13` (build) + `debian:bookworm-slim` (runtime) | ~100MB final | Compiled C/C++ apps |

### Course-Specific References
- [Zero to Hero GitHub Repo](https://github.com/S-V-J/zero2hero)
- [Discord Community](https://discord.gg/rE4gXC36) — Get help, share progress
- [YouTube Playlist](https://www.youtube.com/@heroStackAcademy) — Watch all episodes

### Best Practices Checklist
- [ ] Use specific base image tags (not `latest`)
- [ ] Prefer `-slim` or `-alpine` variants for smaller images
- [ ] Copy dependency files before application code (layer caching)
- [ ] Use `--no-cache-dir` with pip/npm to reduce image size
- [ ] Use exec form `["cmd", "arg"]` for CMD/ENTRYPOINT
- [ ] Add `.dockerignore` to exclude unnecessary files
- [ ] Run as non-root user in production (`USER` instruction)
- [ ] Add `HEALTHCHECK` for orchestration readiness
- [ ] Scan images for vulnerabilities (`docker scan` or `trivy`)

---

## What's Next?

**Episode 11**: Linux CLI Mastery + Bash Automation
→ We'll dive deep into:
- Essential Linux commands (`find`, `grep`, `awk`, `sed`, `jq`)
- Shell scripting best practices (functions, error handling, logging)
- Text processing pipelines for log analysis and data transformation
- Automation workflows for DevOps tasks (backup, deploy, monitor)
- Integrating Bash with Python/C for hybrid solutions

> 💬 **Stuck?** Join Discord: [discord.gg/rE4gXC36](https://discord.gg/rE4gXC36)
> ⭐ **Found this helpful?** Star the repo: [github.com/S-V-J/zero2hero](https://github.com/S-V-J/zero2hero)
> 💙 **Support This Course**: [github.com/sponsors/S-V-J](https://github.com/sponsors/S-V-J)

---

## 📝 Quick Reference Commands

```bash
# Dockerfile Development
docker build -t my-app .                    # Build image from Dockerfile
docker build --no-cache -t my-app .         # Build without using cache
docker build --progress=plain -t my-app .   # Verbose build output

# Image Inspection
docker images my-app --format "table {{.ID}}\t{{.Size}}\t{{.CreatedAt}}"
docker history my-app --no-trunc | head -10
docker inspect my-app | jq '.[0].Config'    # Requires jq installed

# Container Testing
docker run -p 8000:8000 --rm my-app         # Run with port mapping
docker run -it --rm my-app bash             # Interactive shell for debugging
docker run --rm -e VAR=value my-app         # Pass environment variable

# Cleanup
docker image prune -f                       # Remove dangling images
docker builder prune -f                     # Clear build cache
docker system df                            # Show disk usage by Docker

# Security Scanning
docker scan my-app                          # Built-in vulnerability scan (requires Docker Desktop)
trivy image my-app                          # Alternative: Trivy scanner (install separately)
```

---

> 🚀 **Pro Tip**: Bookmark this page. You'll reference Dockerfile patterns throughout the course — and in your professional career.

*Last Updated: $(2026-07-04) • Course Status: In Progress*
```
