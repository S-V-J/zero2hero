# Episode 9: Write app.py — Simple Python Flask App from Scratch

> 🎯 **Goal**: Write a professional Flask application from zero that connects to PostgreSQL and Redis with proper error handling, environment variables, and health checks.

📺 **Watch on YouTube**: [Episode 9: Write app.py](https://youtu.be/k39Vk6n6ZRk)

---

## 📋 Table of Contents

- [Why Write app.py?](#why-write-apppy)
- [Prerequisites](#prerequisites)
- [Step-by-Step Writing Guide](#step-by-step-writing-guide)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Video Timestamps](#video-timestamps)
- [Resources](#resources)
- [What's Next](#whats-next)

---

## Why Write app.py?

| Concept | Purpose | Why It Matters |
|---------|---------|---------------|
| **Flask Framework** | Lightweight web framework for Python | Industry standard for building APIs and web apps quickly |
| **Environment Variables** | Configuration via `os.getenv()` | Keeps secrets out of code; enables dev/prod environment switching |
| **Database Connection Function** | Encapsulated PostgreSQL logic | Clean, reusable, testable code pattern |
| **Redis Connection Function** | Encapsulated cache logic | Same benefits as DB function; enables caching strategies |
| **Try-Except Error Handling** | Graceful failure handling | Prevents crashes when services are down; improves user experience |
| **JSON Response with jsonify** | Structured API responses | Standard format for frontend consumption and API consumers |
| **Main Guard (`if __name__`)** | Proper execution control | Enables both development and production deployment patterns |

> 💡 **Detective Novel Analogy**: Your `app.py` is like a mystery novel. The main route (`/`) is your opening scene. The database is your evidence locker. Redis is your informant network. Error handling is your backup plan. When you write it yourself, you become the author — in complete control of the plot.

---

## Prerequisites

Before starting Episode 9, ensure you have:

- ✅ Python 3.11+ installed (from Episode 3)
- ✅ Flask, psycopg2-binary, and redis packages in `requirements.txt`
- ✅ Docker Compose file with `app`, `db`, and `cache` services (from Episode 8)
- ✅ VS Code with Python extension installed
- ✅ Basic understanding of Python functions and error handling

---

## Step-by-Step Writing Guide

### Step 1: Start with Imports

```python
import os
import time
from flask import Flask, jsonify
import psycopg2
import redis
```

**Why these imports?**
| Import | Purpose |
|--------|---------|
| `os` | Access environment variables for configuration |
| `time` | Add timestamps to API responses |
| `Flask, jsonify` | Web framework and JSON response helper |
| `psycopg2` | PostgreSQL database adapter |
| `redis` | Redis cache client |

> 💡 Pro Tip: Always put imports at the top of your file — it's a Python convention that makes code easier to read and maintain.

### Step 2: Create Flask App Instance

```python
app = Flask(__name__)
```

**Why `__name__`?**
- Tells Flask where to look for templates and static files
- Standard practice for all Flask applications
- Enables proper module loading in larger projects

### Step 3: Configure Environment Variables

```python
# Get environment variables
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:password@db:5432/mydb")
REDIS_URL = os.getenv("REDIS_URL", "redis://cache:6379/0")
```

**Why use `os.getenv()` with defaults?**
- Allows configuration via environment (secure for production)
- Provides safe defaults for local development
- Prevents crashes if variables are missing

> ⚠️ Security Note: Never hardcode real credentials in your code. Use environment variables or secret managers in production.

### Step 4: Create Database Connection Function

```python
# Database connection
def get_db_connection():
    conn = psycopg2.connect(DATABASE_URL)
    return conn
```

**Why encapsulate connection logic?**
- Single place to update connection parameters
- Easier to add connection pooling later
- Cleaner route functions (separation of concerns)

### Step 5: Create Redis Connection Function

```python
# Redis connection
def get_redis_connection():
    return redis.from_url(REDIS_URL)
```

**Why `from_url()`?**
- Redis-py provides this convenient method
- Handles connection parameters parsing automatically
- Consistent with database connection pattern

### Step 6: Create Main Route with Error Handling

```python
@app.route("/")
def hello():
    # Test database
    try:
        conn = get_db_connection()
        cur = conn.cursor()
        cur.execute("SELECT NOW()")
        db_time = cur.fetchone()[0]
        cur.close()
        conn.close()
        db_status = f"Database connected successfully: {db_time}"
    except Exception as e:
        db_status = f"Database connection failed: {e}"

    # Test Redis
    try:
        r = get_redis_connection()
        r.set("hello", "world")
        redis_value = r.get("hello").decode()
        redis_status = f"Redis connected: {redis_value}"
    except Exception as e:
        redis_status = f"Redis error: {e}"
```

**Why try-except blocks?**
| Benefit | Explanation |
|---------|-------------|
| **Prevents Crashes** | App continues running even if DB/Redis is down |
| **User Feedback** | Returns helpful error messages instead of 500 errors |
| **Debugging** | Error messages help identify issues quickly |
| **Production Ready** | Graceful degradation is essential for reliability |

> 💡 Best Practice: In production, log errors to a monitoring system instead of (or in addition to) returning them to users.

### Step 7: Return JSON Response

```python
    return jsonify(
        {
            "message": "Hello from Docker Compose!",
            "database": db_status,
            "redis": redis_status,
            "timestamp": time.time(),
        }
    )
```

**Why this response structure?**
| Field | Purpose |
|-------|---------|
| `message` | Human-readable status for quick checks |
| `database` | Detailed DB connection status for debugging |
| `redis` | Detailed Redis connection status for debugging |
| `timestamp` | Helps verify response freshness and caching behavior |

### Step 8: Add Main Guard

```python
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

**Why the main guard?**
- Allows the file to be imported as a module without running the server
- Enables both development (`python app.py`) and production (Gunicorn/uWSGI) deployment
- `host="0.0.0.0"` makes the app accessible from outside the container
- `port=8000` matches the Docker Compose port mapping

---

## Verification

After writing `app.py`, verify it works correctly:

```bash
# 1. Check syntax (optional but recommended)
python3 -m py_compile app.py

# 2. Build and start services
docker-compose up --build

# 3. Wait for app to be ready (watch logs for "Running on http://0.0.0.0:8000")

# 4. Test the endpoint
curl http://localhost:8000

# Expected output:
{
  "database": "Database connected successfully: 2024-04-14 12:34:56.789012+00:00",
  "message": "Hello from Docker Compose!",
  "redis": "Redis connected: world",
  "timestamp": 1713098096.789
}

# 5. Test error handling (optional): Stop the database container
docker-compose stop db
curl http://localhost:8000
# Should return error message instead of crashing:
# {"database": "Database connection failed: ...", ...}
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: No module named 'flask'` | Ensure `requirements.txt` includes `flask==2.3.3` and Dockerfile installs dependencies |
| `psycopg2.OperationalError: could not connect to server` | Ensure PostgreSQL service is running in Docker Compose and the connection URL uses hostname `db` |
| `redis.exceptions.ConnectionError` | Ensure Redis service is running and the connection URL uses hostname `cache` |
| `IndentationError: unexpected indent` | Check Python indentation — use 4 spaces consistently, not tabs |
| `SyntaxError: invalid syntax` | Check for missing colons, parentheses, or quotes; use a linter like `flake8` |
| App starts but returns 500 errors | Check Docker logs: `docker-compose logs app` for detailed error messages |
| Environment variables not loading | Ensure `docker-compose.yml` passes environment variables to the `app` service |

### Quick Debug Commands

```bash
# View app logs in real-time
docker-compose logs -f app

# Execute commands inside the app container
docker-compose exec app python3 -c "import os; print(os.getenv('DATABASE_URL'))"

# Test database connection directly from container
docker-compose exec app python3 -c "import psycopg2; conn = psycopg2.connect('postgresql://user:password@db:5432/mydb'); print('DB OK'); conn.close()"

# Test Redis connection directly from container
docker-compose exec app python3 -c "import redis; r = redis.from_url('redis://cache:6379/0'); print('Redis OK:', r.ping())"
```

---

## Resources

### Official Documentation
- [Flask Quickstart](https://flask.palletsprojects.com/en/2.3.x/quickstart/)
- [psycopg2 Documentation](https://www.psycopg.org/docs/)
- [redis-py Documentation](https://redis.readthedocs.io/en/stable/)
- [Python os Module](https://docs.python.org/3/library/os.html)

### Course-Specific References
- [Zero to Hero GitHub Repo](https://github.com/S-V-J/zero2hero)
- [Discord Community](https://discord.gg/rE4gXC36) — Get help, share progress
- [YouTube Playlist](https://www.youtube.com/@heroStackAcademy) — Watch all episodes

### Best Practices
- Use `python-dotenv` for local development environment management
- Add health check endpoints (`/health`, `/ready`) for production deployments
- Implement connection pooling for high-traffic applications
- Log errors to a monitoring system (Sentry, Datadog, etc.) in production

---

## 📝 Quick Reference Commands

```bash
# Python Development
python3 -m py_compile app.py          # Check syntax without running
python3 app.py                        # Run app locally (dev only)
pip3 install -r requirements.txt      # Install dependencies

# Docker Compose Workflow
docker-compose up --build             # Build and start all services
docker-compose logs -f app            # View app logs in real-time
docker-compose exec app bash          # Open shell inside app container
docker-compose down                   # Stop and remove containers

# Testing Endpoints
curl http://localhost:8000            # Test main health endpoint
curl -v http://localhost:8000         # Verbose output for debugging

# Database Testing (from host)
docker-compose exec db psql -U user -d mydb -c "SELECT NOW();"

# Redis Testing (from host)
docker-compose exec cache redis-cli ping
```

---

> 🚀 **Pro Tip**: Bookmark this page. You'll reference this app structure throughout the course — and in your professional career.

*Last Updated: $(18/05/2026) • Course Status: In Progress*
```
