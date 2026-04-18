# Episode 3: Development Tools Installation — Docker, Python, Node.js, C/C++, VoIP Utilities

> 🎯 **Goal**: Install and verify all core development tools required for the Zero to Hero course — with theory explaining WHY each tool matters for production engineering.

📺 **Watch on YouTube**: [Episode 3: Development Tools Setup](https://youtu.be/48xKbMgweq4)

---

## 📋 Table of Contents

- [Why These Tools? The Professional Engineer's Toolkit](#why-these-tools-the-professional-engineers-toolkit)
- [Prerequisites](#prerequisites)
- [Step-by-Step Installation](#step-by-step-installation)
- [Verification Checklist](#verification-checklist)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)
- [Video Timestamps](#video-timestamps)
- [Resources & Further Reading](#resources--further-reading)
- [What's Next: Episode 4](#whats-next-episode-4)

---

## Why These Tools? The Professional Engineer's Toolkit

Before typing any commands, understand **why** we install these specific tools. This isn't a random list — it's the exact stack used by Fortune 500 engineers, telecom providers, and top freelance developers.

### The Kitchen Analogy 🍳

Think of your development environment like a professional kitchen:

| Tool | Kitchen Equivalent | Why It Matters |
|------|-------------------|----------------|
| **Ubuntu (WSL2)** | The kitchen itself | Clean, organized, standardized environment for consistent results |
| **Python** | Versatile chef's knife | Quick automation, AI/ML work, scripting, data processing |
| **Node.js** | High-speed blender | Modern full-stack JavaScript, real-time APIs, frontend/backend unity |
| **C/C++ (GCC/G++)** | Precision surgical tools | Systems programming, performance-critical code, VoIP library compilation |
| **Docker** | Meal-prep containers | Package apps with dependencies — runs identically anywhere (your laptop, cloud, client server) |
| **FFmpeg/sox** | Specialty spices & audio gear | Voice processing, call recording, speech-to-text, media manipulation for VoIP/AI |
| **PostgreSQL/Redis clients** | Quality ingredients | Database interaction, testing, debugging data-driven applications |

> 💡 **Key Insight**: Without these tools, you're trying to cook a five-course meal with a butter knife. With them? You're a professional chef ready for production.

### Industry Demand Context

| Skill | Global Demand | Typical Salary Range |
|-------|--------------|---------------------|
| Docker + DevOps | 250,000+ roles | $90k – $160k |
| Python + AI/ML | +40% YoY growth | $120k – $200k+ |
| C/C++ Systems Programming | Critical infrastructure roles | $100k – $180k |
| VoIP Engineering (Asterisk/Kamailio) | Niche but high-value | $85k – $150k+ |

> ⚠️ **Disclaimer**: Salary figures reflect 2024–2025 market averages (BLS, LinkedIn, Upwork). Actual earnings depend on skill, experience, location, and client acquisition. This course teaches engineering skills — not a get-rich-quick scheme.

---

## Prerequisites

Before starting Episode 3, ensure you have:

- ✅ Ubuntu 24.04 running in WSL2 (from [Episode 1](https://youtu.be/Ke6eLofGDp0))
- ✅ Git configured and connected to GitHub via SSH (from [Episode 2](https://youtu.be/hBRxGIyZDnc))
- ✅ ~5GB free disk space for tool installation
- ✅ Active internet connection for package downloads
- ✅ Administrator/sudo access in your WSL terminal

---

## Step-by-Step Installation

### Step 1: Update System (Critical for Security & Reproducibility)

```bash
sudo apt update && sudo apt upgrade -y
```

**Why this matters**:  
- Ensures you have the latest security patches  
- Provides a clean, reproducible baseline for all learners  
- Prevents dependency conflicts during later installations  

> 💡 Pro Tip: Always run this before installing new tools — it's the #1 cause of "it works on my machine" failures.

---

### Step 2: Install Essential Development Tools

```bash
sudo apt install -y \
  build-essential \
  git \
  curl \
  wget \
  unzip \
  vim \
  htop \
  net-tools \
  dnsutils \
  software-properties-common \
  gnupg \
  lsb-release
```

**Key Packages Explained**:

| Package | Purpose | Course Relevance |
|---------|---------|-----------------|
| `build-essential` | GCC, Make, compilers for C/C++ | Required for compiling VoIP libraries (Asterisk, Kamailio) |
| `curl` / `wget` | Download files from the web | Installing NodeSource, Python packages, Docker scripts |
| `htop` | Interactive process viewer | Monitoring system resources during VoIP/AI workloads |
| `net-tools` / `dnsutils` | Network debugging (`ifconfig`, `dig`, `nslookup`) | Critical for SIP/RTP troubleshooting in VoIP projects |
| `software-properties-common` | Manage PPAs (Personal Package Archives) | Required to add NodeSource repository for latest Node.js |

---

### Step 3: Install Language Runtimes

#### Python 3 (AI/ML, Automation, Scripting)

```bash
sudo apt install -y python3 python3-pip python3-venv python3-dev
```

**Why each component matters**:
- `python3`: Core interpreter for AI/ML, automation, backend logic
- `python3-pip`: Package manager for installing libraries (`pip install torch`)
- `python3-venv`: Create isolated environments per project (`python3 -m venv myproject`)
- `python3-dev`: Header files for compiling Python C extensions (required for `numpy`, `pandas`, `pytorch`)

> 💡 Best Practice: Always use virtual environments (`venv`) to avoid dependency conflicts between projects.

#### Node.js 20 LTS (Full-Stack JavaScript)

```bash
# Download and run NodeSource setup script (for latest LTS, not outdated Ubuntu default)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs
```

**Why NodeSource?**  
Ubuntu's default Node.js package is often outdated (v18 or older). NodeSource provides the **latest LTS (v20)** with security patches and modern features required for:
- React/Next.js frontend development
- FastAPI/Express backend APIs
- Real-time features with WebSockets

#### Verify C/C++ Compilers

```bash
gcc --version
g++ --version
```

These should already be installed via `build-essential`, but verification ensures:
- Compilers are correctly linked in your PATH
- You can compile C/C++ code for systems programming modules
- VoIP libraries (like `libsrtp`, `libosip`) will build correctly later

---

### Step 4: Install Docker + Docker Compose

```bash
# Install Docker Engine
sudo apt install -y docker.io

# Install Docker Compose (v1 for compatibility with course projects)
sudo apt install -y docker-compose

# Add your user to the docker group (avoid sudo for docker commands)
sudo usermod -aG docker $USER

# Apply group change immediately in current session
newgrp docker
```

**Why Docker?**  
Docker containerization ensures your applications run identically across environments:
- Your WSL2 Ubuntu laptop
- A cloud VM (AWS, Azure, DigitalOcean)
- A client's production server

This eliminates "it works on my machine" issues — critical for professional deployments.

**Why `newgrp docker`?**  
Group changes don't take effect until you log out and back in. `newgrp docker` applies the change immediately in your current terminal session.

---

### Step 5: Verify Docker Installation

```bash
# Check CLI versions
docker --version
docker-compose --version

# Test with official hello-world container
docker run hello-world
```

**Expected Output**:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image...
```

> ✅ If you see this, your container runtime is production-ready.

---

### Step 6: Install VoIP and AI Utilities

```bash
sudo apt install -y \
  ffmpeg \
  sox \
  libssl-dev \
  libsqlite3-dev \
  libpq-dev \
  redis-tools \
  postgresql-client
```

**Why These Specific Packages?**

| Package | Use Case in This Course |
|---------|------------------------|
| `ffmpeg` | Audio/video processing for call recording, WebRTC media, speech-to-text preprocessing |
| `sox` | Command-line audio manipulation (format conversion, filtering, effects) for VoIP testing |
| `libssl-dev` | OpenSSL development headers — required for secure TLS/SRTP connections in C/C++/Python |
| `libsqlite3-dev` | SQLite development library — for lightweight local databases in early projects |
| `libpq-dev` | PostgreSQL development library — for production-grade relational databases |
| `redis-tools` | Redis CLI tools — for caching, session management, real-time pub/sub in full-stack apps |
| `postgresql-client` | `psql` command-line client — for database administration and debugging |

> 💡 These aren't "nice-to-have" extras — they're **required dependencies** for the VoIP and AI projects we'll build in Episodes 5–10.

---

## Verification Checklist

Run this comprehensive checklist to confirm everything is working:

```bash
echo "🔍 Development Tools Verification"
echo "=================================="

# Python
python3 --version && echo "✅ Python 3 installed" || echo "❌ Python 3 missing"

# Node.js
node --version && echo "✅ Node.js installed" || echo "❌ Node.js missing"
npm --version && echo "✅ npm installed" || echo "❌ npm missing"

# C/C++ Compilers
gcc --version | head -n1 && echo "✅ GCC installed" || echo "❌ GCC missing"
g++ --version | head -n1 && echo "✅ G++ installed" || echo "❌ G++ missing"

# Docker
docker --version && echo "✅ Docker CLI installed" || echo "❌ Docker CLI missing"
docker-compose --version && echo "✅ Docker Compose installed" || echo "❌ Docker Compose missing"
docker info >/dev/null 2>&1 && echo "✅ Docker daemon running" || echo "❌ Docker daemon not running"

# Docker Group Permission
groups $USER | grep -q docker && echo "✅ User in docker group" || echo "❌ User NOT in docker group"

# VoIP/AI Utilities
ffmpeg -version | head -n1 && echo "✅ FFmpeg installed" || echo "❌ FFmpeg missing"
sox --version | head -n1 && echo "✅ sox installed" || echo "❌ sox missing"

# Database Clients
which psql && echo "✅ PostgreSQL client installed" || echo "❌ PostgreSQL client missing"
which redis-cli && echo "✅ Redis client installed" || echo "❌ Redis client missing"

echo ""
echo "🎉 All tools ready for course projects!"
```

**Expected Output Summary**:
```
🔍 Development Tools Verification
==================================
Python 3.12.3
✅ Python 3 installed
v20.11.0
✅ Node.js installed
10.2.4
✅ npm installed
gcc (Ubuntu 13.2.0-23ubuntu4) 13.2.0
✅ GCC installed
g++ (Ubuntu 13.2.0-23ubuntu4) 13.2.0
✅ G++ installed
Docker version 24.0.7, build afdd53b
✅ Docker CLI installed
docker-compose version 1.29.2, build unknown
✅ Docker Compose installed
✅ Docker daemon running
zero2hero : zero2hero adm cdrom sudo dip plugdev users docker
✅ User in docker group
ffmpeg version 6.1.1-ubuntu3
✅ FFmpeg installed
sox: SoX v14.4.2
✅ sox installed
/usr/bin/psql
✅ PostgreSQL client installed
/usr/bin/redis-cli
✅ Redis client installed

🎉 All tools ready for course projects!
```

---

## Troubleshooting Common Issues

| Issue | Likely Cause | Solution |
|-------|-------------|----------|
| `docker: permission denied` | User not in docker group OR group change not applied | Run `newgrp docker` or log out/in; verify with `groups $USER` |
| `node: command not found` | NodeSource script failed or Node.js not installed | Re-run NodeSource setup: `curl -fsSL https://deb.nodesource.com/setup_20.x \| sudo -E bash -` then `sudo apt install -y nodejs` |
| `python3: command not found` | Ubuntu minimal install missing Python | Install explicitly: `sudo apt install -y python3 python3-pip` |
| `gcc: command not found` | `build-essential` not installed | Reinstall: `sudo apt install -y build-essential` |
| Docker container fails to pull | Network issue or Docker daemon not running | Check internet; restart Docker: `sudo systemctl restart docker` |
| `newgrp: command not found` | Missing coreutils package | Install: `sudo apt install -y coreutils` |
| `ffmpeg: command not found` | Universe repository not enabled | Enable and install: `sudo add-apt-repository universe && sudo apt install -y ffmpeg` |

### Making Docker Group Persistent Across Reboots

If Docker permissions reset after reboot, add this to `~/.bashrc`:

```bash
# Auto-apply docker group on session start
if ! groups $USER | grep -q docker; then
  sudo usermod -aG docker $USER 2>/dev/null
fi
```

Then reload:
```bash
source ~/.bashrc
```

---

## Video Timestamps

| Time | Topic | Key Takeaway |
|------|-------|-------------|
| 0:00 | Introduction + Why These Tools Matter | Professional engineers use standardized toolchains — not random tutorials |
| 1:00 | Kitchen Analogy: Tools = Chef's Equipment | Memorable framework for understanding tool purposes |
| 3:30 | Step 1: System Update + Essential Tools | Security and reproducibility start with `apt update && upgrade` |
| 6:00 | Step 2: Python + Node.js + C/C++ | Language runtimes enable AI, web, and systems programming |
| 9:00 | Step 3: Docker + Docker Compose | Containerization eliminates environment inconsistencies |
| 12:00 | Step 4: VoIP/AI Utilities (FFmpeg, sox, databases) | Specialized tools enable voice processing and data-driven apps |
| 14:00 | Verification Checklist + Summary | Confirm everything works before moving to coding |
| 15:30 | Next Steps + Call to Action | Join Discord, star repo, prepare for Episode 4 |

---

## Resources & Further Reading

### Official Documentation
- [Docker Installation Guide (Ubuntu)](https://docs.docker.com/engine/install/ubuntu/)
- [NodeSource Distributions](https://github.com/nodesource/distributions)
- [Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [SoX Manual](https://sox.sourceforge.net/sox.html)

### Course-Specific References
- [Zero to Hero GitHub Repo](https://github.com/S-V-J/zero2hero)
- [Discord Community](https://discord.gg/rE4gXC36) — Get help, share progress
- [YouTube Playlist](https://www.youtube.com/@heroStackAcademy) — Watch all episodes

### Security Best Practices
- Never commit `.env` files or secrets to Git
- Use `python3 -m venv` for project isolation
- Run `docker scan` on images before deployment
- Keep system packages updated: `sudo apt update && sudo apt upgrade -y`

---

## What's Next: Episode 4

**Episode 4: Linux CLI Mastery + Bash Automation**  
→ We'll dive deep into:
- Essential Linux commands (`find`, `grep`, `awk`, `sed`)
- Shell scripting best practices
- Text processing pipelines for log analysis
- Automation workflows for DevOps tasks
- Integrating Bash with Python/C for hybrid solutions

> 💬 **Stuck?** Join Discord: [discord.gg/rE4gXC36](https://discord.gg/rE4gXC36)  
> ⭐ **Found this helpful?** Star the repo: [github.com/S-V-J/zero2hero](https://github.com/S-V-J/zero2hero)

---

## 📝 Quick Reference Commands

```bash
# System Management
sudo apt update && sudo apt upgrade -y    # Update system packages
sudo apt install -y <package>             # Install a package
sudo apt remove --purge <package>         # Remove package + config files

# Docker Workflow
docker run hello-world                    # Test Docker installation
docker ps                                 # List running containers
docker-compose up -d                      # Start services in background
docker-compose down                       # Stop and remove services

# Language Verification
python3 --version                         # Check Python version
node --version                            # Check Node.js version
gcc --version                             # Check GCC compiler

# User/Group Management
groups $USER                              # Check user's groups
sudo usermod -aG docker $USER             # Add user to docker group
newgrp docker                             # Apply group change immediately

# VoIP/AI Utility Checks
ffmpeg -i input.mp3 output.wav            # Convert audio format (test FFmpeg)
sox input.wav -n stat                     # Analyze audio file (test sox)
psql --version                            # Check PostgreSQL client
redis-cli ping                            # Test Redis connection (requires server)
```

---

> 🚀 **Pro Tip**: Bookmark this page. You'll reference these commands throughout the course — and in your professional career.

*Last Updated: $(date 2026-04-18) • Course Status: In Progress*
```