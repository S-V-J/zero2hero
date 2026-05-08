# Episode 7: Docker Compose — Run App + PostgreSQL + Redis in 1 Command

> 🎯 **Goal**: Master multi-container orchestration by writing a `docker-compose.yml` that runs a Flask app with PostgreSQL and Redis — all started with a single command.

📺 **Watch on YouTube**: [Episode 7: Docker Compose Setup](https://youtu.be/Fiu3d6L9Nfo)

---

## 📋 Table of Contents

- [Why Docker Compose? The Restaurant Kitchen Analogy](#why-docker-compose-the-restaurant-kitchen-analogy)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Verification Checklist](#verification-checklist)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)
- [Video Timestamps](#video-timestamps)
- [Resources & Further Reading](#resources--further-reading)
- [What's Next: Episode 8](#whats-next-episode-8)

---

## Why Docker Compose? The Restaurant Kitchen Analogy

Before we type any commands, let me explain why Docker Compose matters — with a fun analogy.

### 🍽️ Your Application = A Professional Restaurant Kitchen

| Component | Kitchen Equivalent | Why It Matters |
|-----------|-------------------|----------------|
| **Your App Code** | The chef 👨‍🍳 | Prepares the dishes (business logic) |
| **PostgreSQL** | The pantry 🗄️ | Stores all ingredients (persistent data) |
| **Redis** | The expediter ⚡ | Caches frequently used items for quick access |
| **docker-compose.yml** | The recipe 📋 | Defines all services and how they work together |

**Without Docker Compose**: You hire the chef, stock the pantry, and train the expediter separately — hoping they all show up at the same time and can talk to each other.

**With Docker Compose**: You write one recipe (`docker-compose.yml`) that says:
> "I need one chef, one pantry, and one expediter. Start them all together, and make sure they can communicate."

Then you run one command: `docker-compose up`

And just like that — your kitchen is open for business. 🚀

This isn't just convenient — it's how professional teams ensure consistency across development, testing, and production.

---

## Prerequisites

Before starting Episode 7, ensure you have:

- ✅ Ubuntu 24.04 running in WSL2 (from [Episode 1](https://youtu.be/Ke6eLofGDp0))
- ✅ Docker and Docker Compose installed (from [Episode 3](https://youtu.be/48xKbMgweq4))
- ✅ VS Code with Remote-WSL extension (from [Episode 4](https://youtu.be/REPLACE_WITH_LINK))
- ✅ Git configured and repository cloned (from [Episode 2](https://youtu.be/hBRxGIyZDnc))
- ✅ ~2GB free disk space for Docker images
- ✅ Internet connection for pulling base images

---

## Step-by-Step Setup

### Step 1: Create Project Structure

```bash
# Navigate to your course root
cd ~/zero2hero

# Create a dedicated folder for this episode
mkdir episode-7-docker-compose
cd episode-7-docker-compose

# Create required files
touch app.py docker-compose.yml requirements.txt Dockerfile

# Open all files in VS Code
code app.py docker-compose.yml requirements.txt Dockerfile
```

> 💡 **Pro Tip**: Keeping each episode in its own folder makes it easy to experiment without affecting other projects.

---

### Step 2: Write docker-compose.yml

In `docker-compose.yml`, add:

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

**Key Concepts Explained**:

| Section | Purpose |
|---------|---------|
| `services:` | Defines the containers to run (app, db, cache) |
| `build: .` | Tells Docker to build the `app` service from the local `Dockerfile` |
| `ports:` | Maps container port 8000 to host port 8000 |
| `environment:` | Passes configuration variables to the app |
| `depends_on:` | Ensures `db` and `cache` start before `app` |
| `volumes:` | Mounts local code for live development + persistent data storage |
| `image:` | Uses official lightweight Alpine-based images for PostgreSQL and Redis |

> ⚠️ **Critical**: Note the indentation! YAML is whitespace-sensitive. Services are indented 2 spaces under `services:`, config keys are indented 4 spaces under their service.

---

### Step 3: Create Python App (app.py)

In `app.py`, add:

```python
import os
import time
from flask import Flask, jsonify
import psycopg2
import redis

app = Flask(__name__)

# Get environment variables
DATABASE_URL = os.getenv('DATABASE_URL', 'postgresql://user:password@localhost:5432/mydb')
REDIS_URL = os.getenv('REDIS_URL', 'redis://localhost:6379/0')

# Database connection
def get_db_connection():
    conn = psycopg2.connect(DATABASE_URL)
    return conn

# Redis connection
def get_redis_connection():
    return redis.from_url(REDIS_URL)

@app.route('/')
def hello():
    # Test database
    try:
        conn = get_db_connection()
        cur = conn.cursor()
        cur.execute('SELECT NOW()')
        db_time = cur.fetchone()[0]
        cur.close()
        conn.close()
        db_status = f'✅ Database connected: {db_time}'
    except Exception as e:
        db_status = f'❌ Database error: {e}'

    # Test Redis
    try:
        r = get_redis_connection()
        r.set('hello', 'world')
        redis_value = r.get('hello').decode()
        redis_status = f'✅ Redis connected: {redis_value}'
    except Exception as e:
        redis_status = f'❌ Redis error: {e}'

    return jsonify({
        'message': 'Hello from Docker Compose!',
        'database': db_status,
        'redis': redis_status,
        'timestamp': time.time()
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

**What This App Does**:
- Connects to PostgreSQL and Redis using environment variables
- Has a health check endpoint (`/`) that tests both connections
- Returns JSON with connection status and timestamp
- Runs on `0.0.0.0:8000` to be accessible from outside the container

---

### Step 4: Create requirements.txt

Instead of manually writing version numbers, let `pip` generate them:

```bash
# Install core dependencies (without versions for flexibility)
pip install flask psycopg2-binary redis

# Generate exact versions for reproducibility
pip freeze > requirements.txt

# Verify the file
cat requirements.txt
```

**Expected Output**:
```
blinker==1.9.0
click==8.3.3
Flask==3.1.3
itsdangerous==2.2.0
Jinja2==3.1.6
psycopg2-binary==2.9.12
redis==7.4.0
Werkzeug==3.1.8
...
```

> 💡 **Pro Tip**: Using `pip freeze` ensures your team and future you get the exact same dependency versions — no "it works on my machine" surprises.

---

### Step 5: Create Dockerfile

In `Dockerfile`, add:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

**Build Steps Explained**:

| Instruction | Purpose |
|-------------|---------|
| `FROM python:3.12-slim` | Start with lightweight Python 3.12 base image |
| `WORKDIR /app` | Set working directory inside container |
| `COPY requirements.txt .` | Copy dependency file first (for layer caching) |
| `RUN pip install...` | Install dependencies in isolated layer |
| `COPY . .` | Copy application code |
| `EXPOSE 8000` | Document that the app listens on port 8000 |
| `CMD ["python", "app.py"]` | Define startup command (exec form preferred) |

> ⚠️ **Why copy requirements.txt first?** Docker caches layers. If your code changes but dependencies don't, Docker reuses the cached `pip install` layer — making rebuilds much faster.

---

### Step 6: Build and Start Services

```bash
# Build and start all services
docker-compose up --build
```

**What Happens**:
1. Docker creates a private network for service communication
2. Creates named volumes for PostgreSQL and Redis data persistence
3. Pulls `postgres:15-alpine` and `redis:7-alpine` images (if not cached)
4. Builds your `app` image from the `Dockerfile`
5. Starts containers in order: `db` → `cache` → `app` (thanks to `depends_on`)

**Watch for This Output**:
```
app-1  | * Running on http://0.0.0.0:8000
app-1  | * Running on http://127.0.0.1:8000
```

> 💡 **Pro Tip**: Press `Ctrl+C` to stop the services. Run `docker-compose down` to clean up containers (volumes persist by default).

---

### Step 7: Test the Application

In a new terminal tab:

```bash
# Test the health endpoint
curl http://localhost:8000
```

**Expected JSON Response**:
```json
{
  "message": "Hello from Docker Compose!",
  "database": "✅ Database connected: 2026-04-26 09:26:11.592+00:00",
  "redis": "✅ Redis connected: world",
  "timestamp": 1713098096.789
}
```

**Optional: Test Services Directly**

```bash
# Test PostgreSQL directly
docker-compose exec db psql -U user -d mydb -c "SELECT NOW();"
# Expected: Current timestamp

# Test Redis directly
docker-compose exec cache redis-cli ping
# Expected: PONG
```

---

### Step 8: Stop and Clean Up

```bash
# Stop services (Ctrl+C in the terminal running docker-compose up)

# Clean up containers and network (volumes persist)
docker-compose down

# To remove volumes too (⚠️ deletes all data):
# docker-compose down -v
```

> ⚠️ **Warning**: `docker-compose down -v` deletes your database and cache data. Only use this when you want a fresh start.

---

## Verification Checklist

Run these commands to confirm everything is working:

```bash
# 1. Check running containers
docker-compose ps
# Expected: app, db, cache all showing "Up"

# 2. Check service logs
docker-compose logs app
# Expected: Flask startup message

# 3. Test HTTP endpoint
curl -s http://localhost:8000 | jq
# Expected: JSON with database and redis status

# 4. Verify volume persistence
docker volume ls | grep episode-7
# Expected: postgres_data and redis_data volumes listed

# 5. Test direct service access
docker-compose exec db psql -U user -d mydb -c "\dt"
# Expected: List of tables (empty for new DB)
```

---

## Troubleshooting Common Issues

| Issue | Likely Cause | Solution |
|-------|-------------|----------|
| `ERROR: service 'volumes' must be a mapping not an array` | `volumes:` indented at service level instead of under `db:` | Indent `volumes:` 4 spaces under `db:`, not 2 spaces at service level |
| `Cannot connect to the Docker daemon` | Docker not running or user not in docker group | `sudo systemctl start docker` + `sudo usermod -aG docker $USER` + `newgrp docker` |
| `ports are not available` | Port 8000 already in use | `sudo lsof -i :8000` to find process, kill it, or change port in docker-compose.yml |
| `psycopg2.OperationalError: could not connect to server` | PostgreSQL not ready when app starts | Add retry logic in app or use health checks in docker-compose.yml |
| `redis.exceptions.ConnectionError` | Wrong Redis URL or service not ready | Ensure `REDIS_URL=redis://cache:6379/0` and `depends_on` is set |
| `ModuleNotFoundError: No module named 'flask'` | requirements.txt not copied or pip install failed | Verify `COPY requirements.txt .` and `RUN pip install...` in Dockerfile |
| YAML indentation errors | Mixed tabs/spaces or wrong indentation level | Use spaces only (VS Code: "Insert Spaces" enabled); 2 spaces per indentation level |

### Fixing YAML Indentation (Common Mistake)

```yaml
# ❌ WRONG: volumes at service level (2 spaces)
services:
  db:
    image: postgres:15-alpine
  volumes:  # ← This is interpreted as a new service!
    - postgres_data:/var/lib/postgresql/data

# ✅ CORRECT: volumes under db service (4 spaces)
services:
  db:
    image: postgres:15-alpine
    volumes:  # ← Now correctly a child of db
      - postgres_data:/var/lib/postgresql/data
```
---

## Resources & Further Reading

### Official Documentation
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- [Redis Docker Image](https://hub.docker.com/_/redis)
- [Flask Quickstart](https://flask.palletsprojects.com/en/3.0.x/quickstart/)

### Course-Specific Resources
- [Zero to Hero GitHub Repo](https://github.com/S-V-J/zero2hero)
- [Discord Community](https://discord.gg/rE4gXC36) — Get help, share progress
- [YouTube Playlist](https://www.youtube.com/@heroStackAcademy) — Watch all episodes

### Security Best Practices
- Never hardcode secrets in docker-compose.yml — use `.env` files + `.gitignore`
- Use `--no-cache-dir` in pip install to reduce image size
- Add health checks in production: `healthcheck:` in docker-compose.yml
- Use non-root users in Dockerfile for production deployments

---

## 📝 Quick Reference Commands

```bash
# Docker Compose Basics
docker-compose up --build          # Build and start all services
docker-compose up -d               # Start in detached mode (background)
docker-compose ps                  # List running containers
docker-compose logs [service]      # View logs for a service
docker-compose exec [service] [cmd] # Run command inside a container
docker-compose down                # Stop and remove containers/network
docker-compose down -v             # Also remove volumes (⚠️ deletes data)

# Testing & Debugging
curl http://localhost:8000                     # Test app health endpoint
docker-compose exec db psql -U user -d mydb    # Access PostgreSQL CLI
docker-compose exec cache redis-cli            # Access Redis CLI
docker-compose logs -f app                     # Follow app logs in real-time

# Dependency Management
pip install flask psycopg2-binary redis        # Install core packages
pip freeze > requirements.txt                  # Generate exact versions
pip install -r requirements.txt                # Install from file

# YAML Best Practices
# - Use 2 spaces per indentation level (never tabs)
# - Keep services at 2 spaces, config keys at 4 spaces, list items at 6 spaces
# - Validate with: docker-compose config
```

---

> 🚀 **Pro Tip**: Bookmark this page. You'll reference these commands throughout the course — and in your professional career.

*Last Updated: 2026-04-26 • Course Status: In Progress*

---

## ⚠️ LEGAL DISCLAIMER

> **Educational Purpose Only**: Content is for educational purposes only. Not professional advice, employment guarantee, or financial recommendation.
>
> **Earnings Disclaimer**: Income figures reflect 2024-2025 market averages from BLS, LinkedIn, Upwork. Actual earnings depend on skill, effort, location, and market conditions. This course teaches engineering skills — not a get-rich-quick scheme. Results vary.
>
> **Use at Your Own Risk**: Code and commands are provided "as is" without warranty. Test in safe environments before production. You are responsible for your systems and data.
>
> **Third-Party Tools**: Course uses open-source tools (Linux, Asterisk, Kamailio, Docker). We are not affiliated with these projects. Review official documentation and licenses.
>
> **Security & Ethics**: Techniques are for defensive security and ethical development only. Never access systems without permission. Unauthorized access is illegal.
>
> **© Copyright & License**:
> - **Code License**: All source code is MIT licensed. Free to use, modify, distribute — even commercially — with attribution.
> - **Video Content**: Tutorials and explanations are © 2025 Siddhant Kumar / heroStackAcademy. All rights reserved. Share links, but do not re-upload or redistribute video content without permission.
> - **Attribution**: If you reuse code or concepts, please credit: "Based on Zero to Hero: Full Stack + VoIP + AI Engineer by Siddhant Kumar (https://github.com/S-V-J/zero2hero)"
```
