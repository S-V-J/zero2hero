# Episode 5: Git Like a Pro — Pre-commit Hooks + .gitignore

> 🎯 **Goal**: Configure professional Git workflows with pre-commit hooks for auto-formatting, linting, and a comprehensive .gitignore to keep your repository clean and secure.

📺 **Watch on YouTube**: [Episode 5: Git Like a Pro](https://youtu.be/m_BMYqglbvQ)

---

## 📋 Table of Contents

- [Why Pre-commit Hooks?](#why-pre-commit-hooks)
- [Python Virtual Environments: The Kitchen Analogy](#python-virtual-environments-the-kitchen-analogy)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Video Timestamps](#video-timestamps)
- [Resources](#resources)
- [What's Next](#whats-next)

---

## Why Pre-commit Hooks?

Think of your Git repository like a **professional publishing house**:

| Component | Analogy | Purpose |
|-----------|---------|---------|
| **Your Code** | The manuscript | Your ideas, logic, and functionality |
| **Git** | The printer | Captures and versions your work |
| **Pre-commit Hooks** | The editorial team | Checks spelling, formatting, and style before publishing |
| **.gitignore** | The security guard | Keeps secrets, build files, and junk out of the published book |

**Without hooks**: You might publish typos, messy formatting, or accidental secrets.
**With hooks**: Every commit is clean, consistent, and production-ready.

### What Our Hooks Do

| Hook | Purpose | Example |
|------|---------|---------|
| `trailing-whitespace` | Removes extra spaces at end of lines | `"hello  "` → `"hello"` |
| `end-of-file-fixer` | Ensures files end with single newline | Adds `\n` if missing |
| `check-yaml` | Validates YAML syntax before commit | Catches indentation errors early |
| `black` (Python) | Auto-formats Python code to consistent style | `def hello( name ):` → `def hello(name):` |
| `flake8` (Python) | Catches common Python errors and style issues | `F401: module imported but unused` |
| `eslint` (JS/TS) | Linting for JavaScript and TypeScript | `no-unused-vars`, `eqeqeq` rules |

> 💡 This isn't optional polish — it's how professional teams ship reliable software.

---

## Python Virtual Environments: The Kitchen Analogy

> 🎬 *This section was added in the video to explain why we use virtual environments.*

Think of your **system Python** like a **shared kitchen in an apartment building**:

- Everyone uses it — your OS, system tools, other apps
- If you start installing random packages globally, you might break something for everyone else
- That's what **PEP 668** prevents

A **virtual environment** is like **your own private kitchen**:

- Inside it, you can install any packages you want — specific versions, experimental tools, project dependencies
- Without affecting anyone else

### Benefits of Virtual Environments

✅ No more "it works on my machine" problems
✅ Easy to share exact dependencies with teammates via `requirements.txt`
✅ Safe to experiment without breaking your system
✅ Required for professional Python development

---

## Prerequisites

Before starting Episode 5, ensure you have:

- ✅ Ubuntu 24.04 running in WSL2 (from [Episode 1](https://youtu.be/Ke6eLofGDp0))
- ✅ Git configured and connected to GitHub via SSH (from [Episode 2](https://youtu.be/hBRxGIyZDnc))
- ✅ Python 3 installed with pip (from [Episode 3](https://youtu.be/48xKbMgweq4))
- ✅ Project folder `~/zero2hero` with Git initialized
- ✅ Internet connection for package downloads

---

## Step-by-Step Setup

### Step 1: Create a Python Virtual Environment

```bash
# Navigate to your project folder
cd ~/zero2hero

# Create a virtual environment named 'venv'
python3 -m venv venv

# Verify it was created
ls -la venv/
# Expected: bin/, lib/, pyvenv.cfg
```

### Step 2: Activate the Virtual Environment

```bash
# Activate the environment
source venv/bin/activate

# Notice your prompt now shows (venv)
# Verify which Python/pip you're using
which python
which pip
# Both should point to ~/zero2hero/venv/bin/
```

### Step 3: Upgrade pip Inside venv

```bash
# Always upgrade pip in new environments
pip install --upgrade pip
```

### Step 4: Install pre-commit Framework

```bash
# Install pre-commit inside the virtual environment
pip install pre-commit

# Verify installation
pre-commit --version
# Expected: pre-commit 3.6.0 or similar
```

### Step 5: Create .pre-commit-config.yaml

In your project root (`~/zero2hero`), create `.pre-commit-config.yaml`:

```bash
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3

  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8

  - repo: https://github.com/pre-commit/mirrors-eslint
    rev: v8.56.0
    hooks:
      - id: eslint
        files: \.(js|ts|jsx|tsx)$
        types: [file]
        additional_dependencies:
          - eslint@8.56.0
EOF
```

**Configuration Breakdown**:
| Section | Purpose |
|---------|---------|
| `pre-commit-hooks` | Basic hygiene: trailing whitespace, EOF fixer, YAML validation, large file blocking |
| `black` | Auto-format Python code to consistent style |
| `flake8` | Catch Python errors and style issues |
| `eslint` | Lint JavaScript/TypeScript files |

### Step 6: Install the Git Hook

```bash
# Tell Git to use pre-commit
pre-commit install

# Verify the hook was installed
ls -la .git/hooks/pre-commit
# Expected: -rwxr-xr-x permissions (executable)
```

### Step 7: Create .gitignore for Multi-Language Projects

Create or edit `.gitignore` in your project root:

```bash
cat > .gitignore << 'EOF'
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
venv/
.venv/
env/
.env
pip-log.txt
pip-delete-this-directory.txt

# Node.js
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
.npm/
.yarn/
.pnpm-store/

# C/C++
*.o
*.so
*.a
*.dll
*.exe
*.out
*.app
*.dSYM/
*.sublime-project
*.sublime-workspace
build/
CMakeFiles/
cmake_install.cmake
Makefile
*.gcno
*.gcda

# Docker
.docker/
*.log

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Secrets (NEVER commit these)
*.pem
*.key
*.env.local
.env*.local
secrets.json
credentials.json
EOF
```

> ⚠️ **Critical**: The "Secrets" section at the bottom prevents accidental commits of API keys, certificates, or passwords. If you accidentally commit secrets, rotate them immediately.

### Step 8: Test the Workflow

```bash
# Create a messy Python file to test hooks
echo 'def hello(  name  ):
    print( "Hello, "+name )
    return name' > test_hook.py

# Try to commit the messy file
git add test_hook.py
git commit -m "test: messy python file for hook demo"
```

**Expected Output**:
```
[INFO] Initializing environment for https://github.com/psf/black...
[INFO] Installing environment for https://github.com/psf/black...
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
black....................................................................Passed
flake8...................................................................Passed
[main abc1234] test: messy python
 1 file changed, 3 insertions(+)
 create mode 100644 test_hook.py
```

**Verify the file was auto-formatted**:
```bash
cat test_hook.py
```

**Expected Output**:
```python
def hello(name):
    print("Hello, " + name)
    return name
```

### Step 9: Run Hooks Manually (Optional)

```bash
# Run hooks on all files (great for final check before push)
pre-commit run --all-files

# Run hooks on a single file (useful for debugging)
pre-commit run --files test_hook.py
```

### Step 10: Clean Up Test File

```bash
# Reset and remove test file
git reset HEAD test_hook.py
rm test_hook.py
```

---

## Verification

Run this checklist to confirm everything is working:

```bash
echo "🔍 Pre-commit + .gitignore Verification"
echo "======================================="

# Verify pre-commit is installed
pre-commit --version && echo "✅ pre-commit installed"

# Verify config file exists
test -f .pre-commit-config.yaml && echo "✅ .pre-commit-config.yaml exists"

# Verify Git hook is installed and executable
test -x .git/hooks/pre-commit && echo "✅ Git pre-commit hook is executable"

# Verify .gitignore exists and has expected patterns
grep -q "venv/" .gitignore && echo "✅ .gitignore contains Python patterns"
grep -q "node_modules/" .gitignore && echo "✅ .gitignore contains Node.js patterns"
grep -q "*.pem" .gitignore && echo "✅ .gitignore contains secrets patterns"

# Test auto-format with a quick Python snippet
echo 'x=1+2' > test_format.py
git add test_format.py
git commit -m "test: format check" 2>&1 | grep -q "black" && echo "✅ Black formatter triggered"
rm test_format.py
git reset HEAD test_format.py 2>/dev/null

echo ""
echo "🎉 Professional Git workflow ready!"
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `pip: command not found` | Install pip: `sudo apt install -y python3-pip` |
| `pre-commit: command not found` after install | Ensure venv is activated: `source venv/bin/activate` |
| `git commit` hangs on first run | Pre-commit is downloading hook environments — wait 2-5 minutes (normal) |
| ESLint hook fails to install | Ensure Node.js is installed: `node --version`; if missing, install from Episode 3 |
| `.gitignore` not working | Verify file is in project root (not subfolder); check for typos in patterns |
| Hooks not running on commit | Verify `.git/hooks/pre-commit` exists and is executable: `chmod +x .git/hooks/pre-commit` |
| Virtual environment not activating | Ensure you're in the project directory: `cd ~/zero2hero` then `source venv/bin/activate` |

### Making Virtual Environment Activation Persistent

To auto-activate your venv when you open a terminal in the project:

```bash
# Add to ~/.bashrc
echo '
# Auto-activate zero2hero venv if in project directory
if [ -f "~/zero2hero/venv/bin/activate" ] && [ "$(basename "$PWD")" = "zero2hero" ]; then
    source ~/zero2hero/venv/bin/activate
fi
' >> ~/.bashrc

# Reload configuration
source ~/.bashrc
```
---

## Resources

### Official Documentation
- [pre-commit Framework](https://pre-commit.com/)
- [Black Python Formatter](https://black.readthedocs.io/)
- [Flake8 Linter](https://flake8.pycqa.org/)
- [ESLint](https://eslint.org/)
- [Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)
- [PEP 668: Marking base Python as externally managed](https://peps.python.org/pep-0668/)

### Course-Specific References
- [Zero to Hero GitHub Repo](https://github.com/S-V-J/zero2hero)
- [Discord Community](https://discord.gg/rE4gXC36) — Get help, share progress
- [YouTube Playlist](https://www.youtube.com/@heroStackAcademy) — Watch all episodes

### Security Best Practices
- Never commit `.env`, `*.pem`, `*.key`, or `credentials.json` files
- Use `python3 -m venv` for project isolation
- Run `pre-commit run --all-files` before major pushes
- Rotate secrets immediately if accidentally committed

---

## What's Next?

**Episode 6: Linux CLI Mastery + Bash Automation**
→ We'll dive deep into:
- Essential Linux commands (`find`, `grep`, `awk`, `sed`)
- Shell scripting best practices
- Text processing pipelines for log analysis
- Automation workflows for DevOps tasks
- Integrating Bash with Python/C for hybrid solutions

> 💬 **Stuck?** Join Discord: [discord.gg/rE4gXC36](https://discord.gg/rE4gXC36)
> ⭐ **Found this helpful?** Star the repo: [github.com/S-V-J/zero2hero](https://github.com/S-V-J/zero2hero)
> 💙 **Support This Course**: [github.com/sponsors/S-V-J](https://github.com/sponsors/S-V-J)

---

## 📝 Quick Reference Commands

```bash
# Virtual Environment Management
python3 -m venv venv              # Create virtual environment
source venv/bin/activate          # Activate environment
deactivate                        # Exit virtual environment
pip install --upgrade pip         # Upgrade pip inside venv

# Pre-commit Workflow
pip install pre-commit            # Install framework
pre-commit install                # Install Git hook
pre-commit run --all-files        # Run on all files
pre-commit run --files file.py    # Run on specific file

# Git + .gitignore
git add .                         # Stage all changes
git commit -m "message"           # Commit with message (hooks run automatically)
git status                        # Check repository status

# Verification Commands
which python                      # Verify Python path (should be in venv)
pre-commit --version              # Check pre-commit version
ls -la .git/hooks/pre-commit      # Verify hook is executable
```

---

> 🚀 **Pro Tip**: Bookmark this page. You'll reference these commands throughout the course — and in your professional career.

*Last Updated: $(date +%Y-%m-%d) • Course Status: In Progress*

---

## 📬 Connect With Me

| Platform | Link |
|----------|------|
| ✉️ Email | [stjl093@gmail.com](mailto:stjl093@gmail.com) |
| 💼 LinkedIn | [linkedin.com/in/sid-093](https://www.linkedin.com/in/sid-093) |
| 📦 GitHub | [github.com/S-V-J](https://github.com/S-V-J) |
| 🎥 YouTube | [@heroStackAcademy](https://www.youtube.com/@heroStackAcademy) |
| 💙 Sponsor | [github.com/sponsors/S-V-J](https://github.com/sponsors/S-V-J) |
| 💬 Discord | [discord.gg/rE4gXC36](https://discord.gg/rE4gXC36) |

---

> I'm **Siddhant Kumar** — Full Stack + VoIP + AI Engineer, working professionally since 2021. This course is my passion project to help developers like you master enterprise-grade skills through hands-on learning.

*Built with ❤️ for the open-source community • One commit at a time 🚀*
```
