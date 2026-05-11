# Episode 8: Write docker-compose.yml from Scratch — Zero to Hero

> 🎯 **Goal**: Learn to write a `docker-compose.yml` file from scratch with proper YAML indentation, service configuration, and networking — understanding every line, not just copying.

📺 **Watch on YouTube**: [Episode 8: Write docker-compose.yml](https://youtu.be/X9MXNxcwy7E)

---

## 📋 Table of Contents

- [Why Write docker-compose.yml from Scratch?](#why-write-docker-composeyml-from-scratch)
- [Prerequisites](#prerequisites)
- [YAML Fundamentals: Indentation Rules](#yaml-fundamentals-indentation-rules)
- [Step-by-Step: Writing docker-compose.yml](#step-by-step-writing-docker-composeyml)
- [Testing Your Configuration](#testing-your-configuration)
- [Common Pitfalls & Troubleshooting](#common-pitfalls--troubleshooting)
- [Video Timestamps](#video-timestamps)
- [Resources](#resources)
- [What's Next](#whats-next)

---

## Why Write docker-compose.yml from Scratch?

In Episode 7, we ran a Docker Compose application using a pre-written file. But copying code doesn't teach you **why** it works.

Writing your own `docker-compose.yml` from scratch gives you:

| Benefit | Explanation |
|---------|-------------|
| 🔍 **Deep Understanding** | You'll know what every key does, not just that "it works" |
| 🛠️ **Debugging Skills** | When something breaks, you'll know where to look |
| 🎨 **Flexibility** | You can adapt the file for any project, not just the course examples |
| 💼 **Professional Value** | Employers want engineers who can write infrastructure code, not just run it |

> 💡 **Musical Analogy**: Pre-written docker-compose.yml files are like sheet music someone else composed — you can play it, but you don't understand why it sounds good. Writing your own is like composing your own symphony — you control every instrument, every cue, every note.

---

## Prerequisites

Before starting this episode, ensure you have:

- ✅ Completed Episodes 1–7 (WSL2, Git, Docker, VS Code setup)
- ✅ Docker and Docker Compose installed and running
- ✅ Existing project files from Episode 7 in `~/zero2hero/episode-7-docker-compose`:
  - `app.py` (Flask application)
  - `Dockerfile` (Python app build instructions)
  - `requirements.txt` (Python dependencies)
- ✅ VS Code with YAML language support (install "YAML" extension by Red Hat if needed)

---

## YAML Fundamentals: Indentation Rules

YAML is **space-sensitive** — not tab-sensitive. This is the #1 cause of errors.

### Indentation Guide

```yaml
version: '3.8'           # Root level — NO indent (0 spaces)
services:                # Root level — NO indent (0 spaces)
  app:                   # Service level — 2 spaces
    build: .             # App config — 4 spaces
    ports:               # App config — 4 spaces
      - "8000:8000"      # Ports list — 6 spaces
  db:                    # Service level — 2 spaces (same as app:)
    image: postgres:15   # DB config — 4 spaces
```

### Key Rules

| Rule | Example | Why It Matters |
|------|---------|---------------|
| Use **2 spaces per indentation level** | `services:` → `  app:` | YAML parsers expect consistent spacing |
| **Never use tabs** | ❌ `	app:` (tab character) | Tabs cause "mapping values not allowed" errors |
| **List items use `-`** | `ports:` → `  - "8000:8000"` | Dashes indicate array/list elements |
| **Key-value pairs use `:`** | `build: .` | Colons separate keys from values |
| **String values with special chars need quotes** | `ports: - "8000:8000"` | Prevents YAML parsing errors |

> ⚠️ **Pro Tip**: In VS Code, enable "Render Whitespace" (`Ctrl+Shift+P` → "Toggle Render Whitespace") to see spaces vs tabs visually.

---

## Step-by-Step: Writing docker-compose.yml

### Step 1: Start with Version Declaration

```yaml
version: '3.8'
```

- **Root level** — no indentation
- `'3.8'` is the latest stable Compose file version as of 2025
- Press **Enter** after this line

### Step 2: Define the Services Section

```yaml
version: '3.8'
services:
```

- `services:` is a **root-level key** — no indentation
- Press **Enter**, then **Tab once** (2 spaces) to indent for the first service

### Step 3: Configure the App Service

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://cache:6379/0
    depends_on:
      - db
      - cache
    volumes:
      - .:/app
    command: python app.py
```

**Indentation breakdown**:
- `app:` — 2 spaces (service name)
- `build:`, `ports:`, etc. — 4 spaces (app service config)
- `- "8000:8000"` — 6 spaces (list item under ports)

**Key explanations**:
| Key | Purpose |
|-----|---------|
| `build: .` | Build image from Dockerfile in current directory |
| `ports:` | Map host port 8000 to container port 8000 |
| `environment:` | Pass configuration variables to the app |
| `depends_on:` | Start db and cache before app (doesn't wait for readiness) |
| `volumes:` | Mount current directory to `/app` for live code reload |
| `command:` | Override default CMD in Dockerfile |

### Step 4: Configure the Database Service

Go back to **service level** (2 spaces indentation):

```yaml
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
```

**Key notes**:
- `image:` uses a pre-built PostgreSQL image (no build needed)
- Environment variables use **key: value** format (no dashes) because they're a mapping, not a list
- `volumes:` uses a **named volume** `postgres_data` for data persistence

### Step 5: Configure the Cache Service

Still at service level (2 spaces):

```yaml
  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
```

- Simple Redis cache with named volume for persistence

### Step 6: Define Named Volumes at Root Level

Go back to **root level** (no indentation):

```yaml
volumes:
  postgres_data:
  redis_data:
```

- These declarations create the named volumes referenced in services
- Without this, Docker creates anonymous volumes that are harder to manage

### Final Complete File

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://cache:6379/0
    depends_on:
      - db
      - cache
    volumes:
      - .:/app
    command: python app.py

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

## Testing Your Configuration

### Step 1: Validate Syntax

Before running, validate your YAML:

```bash
docker-compose config
```

✅ **Expected**: Clean YAML output with no errors
❌ **If error**: Check indentation — use VS Code's YAML extension for real-time validation

### Step 2: Build and Start Services

```bash
docker-compose up --build
```

**Watch for these success indicators**:
```
Creating network "episode-7-docker-compose_default" with the default driver
Creating volume "episode-7-docker-compose_postgres_data" with default driver
Creating volume "episode-7-docker-compose_redis_data" with default driver
...
app-1  | * Running on http://0.0.0.0:8000
```

### Step 3: Test the Application

In a new terminal tab:

```bash
curl http://localhost:8000
```

✅ **Expected JSON response**:
```json
{
  "database": "✅ Database connected: 2024-04-14 12:34:56.789012+00:00",
  "message": "Hello from Docker Compose!",
  "redis": "✅ Redis connected: world",
  "timestamp": 1713098096.789
}
```

### Step 4: Test Direct Service Access (Optional)

```bash
# Test PostgreSQL directly
docker-compose exec db psql -U user -d mydb -c "SELECT NOW();"

# Test Redis directly
docker-compose exec cache redis-cli ping
# Expected: PONG
```

### Step 5: Stop and Clean Up

```bash
# Stop services (Ctrl+C in the up terminal)
docker-compose down

# To remove volumes too (⚠️ deletes data):
# docker-compose down -v
```

---

## Common Pitfalls & Troubleshooting

| Error | Likely Cause | Solution |
|-------|-------------|----------|
| `yaml.scanner.ScannerError: mapping values are not allowed here` | Mixed tabs/spaces or inconsistent indentation | Use 2 spaces per level; enable "Render Whitespace" in VS Code |
| `ERROR: The Compose file is invalid` | Missing colon, wrong indentation, or typo in key names | Run `docker-compose config` to see specific error location |
| `Cannot create container for service app: invalid volume specification` | Volume path typo or missing directory | Ensure paths are correct; use absolute paths if needed |
| `Connection refused` when testing app | Service not ready or port not mapped | Wait for "Running on http://0.0.0.0:8000" log; verify `ports:` mapping |
| `depends_on` not waiting for DB readiness | `depends_on` only controls startup order, not health | Add health checks (advanced) or retry logic in app code |
| Named volumes not created | Forgot `volumes:` section at root level | Add `volumes:` with volume names at root level |

### YAML Validation Tips

1. **Install VS Code YAML extension** (by Red Hat) for real-time error highlighting
2. **Use `docker-compose config`** to validate before running
3. **Copy-paste the final file above** if stuck — then modify incrementally

---

## Resources

### Official Documentation
- [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [YAML Specification](https://yaml.org/spec/)
- [VS Code YAML Extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)

### Course Resources
- [GitHub Repository](https://github.com/S-V-J/zero2hero)
- [Discord Community](https://discord.gg/rE4gXC36) — Get help with YAML indentation
- [Previous Episode 7](https://youtu.be/Fiu3d6L9Nfo) — Running docker-compose (for comparison)

### YAML Quick Reference
```yaml
# Root level (0 spaces)
version: '3.8'
services:
volumes:

# Service level (2 spaces)
  app:
  db:

# Service config (4 spaces)
    build: .
    image: postgres:15

# List items (6 spaces)
    ports:
      - "8000:8000"
    environment:
      - KEY=value

# Mapping values (4 spaces, no dash)
    environment:
      POSTGRES_USER: user
```

---

> 🚀 **Pro Tip**: Bookmark this page. You'll reference this YAML structure for every Docker Compose project you build — from course projects to production deployments.

*Last Updated: $(11/05/2026) • Course Status: In Progress*
```
