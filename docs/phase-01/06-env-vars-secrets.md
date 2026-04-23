# Episode 6: Environment Variables 101 — .env + bashrc + Secrets

> **Goal**: Master secure configuration management by learning to create `.env` files, load secrets with `python-dotenv`, export variables in `.bashrc`, and never accidentally commit credentials to Git.

📺 **Watch on YouTube**: [Episode 6: Environment Variables 101](https://youtu.be/8JlwntJzSW4)

---

## 📋 Table of Contents

- [Why Environment Variables Matter](#why-environment-variables-matter)
- [The Restaurant Analogy: Code vs. Configuration](#the-restaurant-analogy-code-vs-configuration)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Verification Checklist](#verification-checklist)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)
- [Security Best Practices](#security-best-practices)
- [Video Timestamps](#video-timestamps)
- [Resources & Further Reading](#resources--further-reading)
- [What's Next: Episode 7](#whats-next-episode-7)

---

## Why Environment Variables Matter

Environment variables are a fundamental concept in professional software development. They allow you to:

| Benefit | Explanation |
|---------|-------------|
| 🔐 **Separate Secrets from Code** | Keep API keys, database passwords, and tokens out of your public repository |
| 🔄 **Support Multiple Environments** | Use different values for development, staging, and production without changing code |
| 🤝 **Enable Team Collaboration** | Share `.env.example` templates so teammates know what configuration they need |
| 🛡️ **Prevent Accidental Leaks** | `.gitignore` rules ensure secrets never get committed, even by mistake |
| ⚙️ **Simplify Deployment** | Configure applications via environment rather than hardcoding values |

> 💡 **Key Insight**: Hardcoding secrets in your source code is one of the most common security mistakes. Environment variables are the professional solution.

---

## The Restaurant Analogy: Code vs. Configuration

Think of your application like a professional restaurant:

| Component | Restaurant Equivalent | Why It Matters |
|-----------|----------------------|----------------|
| **Your Code** | The recipe — public, shareable, version-controlled | Anyone can read the recipe; it's your intellectual property |
| **Environment Variables** | The secret ingredients — API keys, passwords, config | Kept private in a locked drawer, only accessible to the chef |
| **`.env` File** | The locked drawer — stores secrets separately from the recipe | Never published; only used during preparation |
| **`.gitignore`** | The security guard — ensures secrets never leave the kitchen | Prevents accidental publication of sensitive information |
| **`.env.example`** | The ingredient list on the menu — shows what's needed, not the amounts | Safe to share; helps others know what they need to provide |

> 🍽️ **Punchline**: Without environment variables, you'd publish your secret sauce on the menu. With them, your recipe is public, but your secrets stay secure.

---

## Prerequisites

Before starting Episode 6, ensure you have:

- ✅ Ubuntu 24.04 running in WSL2 (from [Episode 1](https://youtu.be/Ke6eLofGDp0))
- ✅ Git configured and connected to GitHub via SSH (from [Episode 2](https://youtu.be/hBRxGIyZDnc))
- ✅ Python 3 and pip installed (from [Episode 3](https://youtu.be/48xKbMgweq4))
- ✅ Project folder `~/zero2hero` with `.gitignore` from Episode 5
- ✅ ~50MB free disk space for python-dotenv installation
- ✅ Internet connection for pip package downloads

---

## Step-by-Step Setup

### Step 1: Create a `.env` File

```bash
# Navigate to project root
cd ~/zero2hero

# Create the .env file
touch .env

# Open in VS Code
code .env
```

Add example variables to `.env` (use placeholder values for the video):

```env
# Database credentials
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# API keys (NEVER use real keys in examples)
OPENAI_API_KEY=sk-example123456789
ASTERISK_API_TOKEN=ast-example-token

# App configuration
DEBUG=true
PORT=8000
LOG_LEVEL=info
```

**Format Rules**:
- `KEY=value` with no spaces around the equals sign
- Lines starting with `#` are comments
- Values with spaces should be quoted: `MY_VAR="value with spaces"`
- Empty values are allowed: `OPTIONAL_VAR=`

---

### Step 2: Update `.gitignore` to Exclude `.env` Files

Open your `.gitignore` file (created in Episode 5) and add these lines at the top:

```gitignore
# ===========================================
# Environment Variables (NEVER COMMIT)
# ===========================================
.env
.env.local
.env*.local
*.env
.env.example.local
!.env.example

# Python environment files
.venv/
venv/
env/
```

**Why the `!.env.example` exception?**
- `.env.example` is SAFE to commit — it documents required variables without exposing secrets
- The `!` prefix in `.gitignore` means "do not ignore this, even if a previous rule matches"

Verify `.env` is ignored:

```bash
git status
```

**Expected Output**:
```
On branch main
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.env

nothing added to commit but untracked files present
```

> ✅ `.env` appears as "untracked" but is NOT staged — perfect.

---

### Step 3: Install and Use `python-dotenv`

#### Install the Library

```bash
pip3 install python-dotenv
```

#### Create a Test Script

```bash
code test_env.py
```

Add this code to `test_env.py`:

```python
#!/usr/bin/env python3
"""
Test script to demonstrate loading environment variables with python-dotenv.
This script shows how to safely access secrets without hardcoding them.
"""

from dotenv import load_dotenv
import os

# Load variables from .env file into os.environ
# This must be called BEFORE accessing variables with os.getenv()
load_dotenv()

# Access variables safely with default fallbacks
database_url = os.getenv('DATABASE_URL')
api_key = os.getenv('OPENAI_API_KEY')
debug_mode = os.getenv('DEBUG', 'false').lower() == 'true'
port = int(os.getenv('PORT', 8000))

# Print results (mask sensitive values in real apps)
print("=== Environment Variables Loaded ===")
print(f"Database URL: {database_url}")
print(f"API Key (masked): {api_key[:10]}..." if api_key else "No API key set")
print(f"Debug Mode: {debug_mode}")
print(f"Port: {port}")
print("====================================")

# Example: Use variables in application logic
if debug_mode:
    print("⚠️  Running in DEBUG mode — not for production!")
```

#### Run the Test Script

```bash
python3 test_env.py
```

**Expected Output**:
```
=== Environment Variables Loaded ===
Database URL: postgresql://user:password@localhost:5432/mydb
API Key (masked): sk-example...
Debug Mode: True
Port: 8000
====================================
⚠️  Running in DEBUG mode — not for production!
```

> ✅ Variables loaded successfully without hardcoding in the script.

---

### Step 4: Export Variables in `.bashrc` for Terminal Use

Sometimes you need variables available in your terminal session — for CLI tools, scripts, or interactive debugging.

#### Edit `.bashrc`

```bash
code ~/.bashrc
```

Scroll to the bottom and add:

```bash
# ===========================================
# Project Environment Variables (Zero to Hero)
# ===========================================

# Export simple variables for terminal use
export ZERO2HERO_DEBUG="true"
export ZERO2HERO_PROJECT_ROOT="$HOME/zero2hero"

# Auto-load .env file if it exists in project root
# Uses 'set -a' to automatically export all sourced variables
if [ -f "$HOME/zero2hero/.env" ]; then
    set -a  # Automatically export all variables
    source "$HOME/zero2hero/.env"
    set +a  # Stop auto-exporting
fi

# Optional: Print confirmation (comment out for production)
# echo "✓ Zero to Hero environment loaded"
```

#### Reload Your Shell

```bash
source ~/.bashrc
```

#### Test the Variables

```bash
# Test simple export
echo $ZERO2HERO_DEBUG
# Expected: true

# Test loaded from .env
echo $DATABASE_URL
# Expected: postgresql://user:password@localhost:5432/mydb
```

> ✅ Variables are now available in every new terminal session.

---

### Step 5: Create `.env.example` for Team Documentation

Create a safe-to-commit template that documents required variables:

```bash
code .env.example
```

Add the same keys but with placeholder values:

```env
# ===========================================
# Environment Configuration Template
# Copy this file to .env and fill in your values
# NEVER commit .env — only .env.example
# ===========================================

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# External APIs (replace with your real keys)
OPENAI_API_KEY=sk-your-key-here
ASTERISK_API_TOKEN=ast-your-token-here

# Application Settings
DEBUG=true
PORT=8000
LOG_LEVEL=info

# Optional Features (uncomment to enable)
# ENABLE_ANALYTICS=true
# RATE_LIMIT_REQUESTS=100
```

#### Commit `.env.example` (NOT `.env`)

```bash
git add .env.example
git commit -m "docs: add .env.example template for environment variables"
git push origin main
```

#### Verify Security: Try to Add `.env`

```bash
git add .env
git status
```

**Expected Output**:
```
On branch main
nothing to commit, working tree clean
```

> ✅ `.env` is ignored and cannot be accidentally committed.

---

### Step 6: Test the Complete Workflow

Create a more realistic test that mimics production usage:

```bash
code app_config.py
```

```python
#!/usr/bin/env python3
"""
Production-style configuration loader.
Demonstrates best practices for environment variable usage.
"""

from dotenv import load_dotenv
import os
import sys

def load_config():
    """Load and validate required environment variables."""
    load_dotenv()

    required_vars = ['DATABASE_URL', 'PORT']
    missing = [var for var in required_vars if not os.getenv(var)]

    if missing:
        print(f"❌ Missing required environment variables: {missing}", file=sys.stderr)
        print("💡 Copy .env.example to .env and fill in values", file=sys.stderr)
        return False

    return True

def main():
    if not load_config():
        sys.exit(1)

    # Application logic would go here
    print("✅ Configuration loaded successfully")
    print(f"🗄️  Database: {os.getenv('DATABASE_URL')[:30]}...")
    print(f"🔌 Port: {os.getenv('PORT')}")

if __name__ == "__main__":
    main()
```

Run the test:

```bash
python3 app_config.py
```

**Expected Output**:
```
✅ Configuration loaded successfully
🗄️  Database: postgresql://user:password@loc...
🔌 Port: 8000
```

---

## Verification Checklist

Run this comprehensive checklist to confirm everything is working:

```bash
echo "🔍 Environment Variables Verification"
echo "======================================"

# 1. Check .env exists and has content
[ -f .env ] && echo "✅ .env file exists" || echo "❌ .env file missing"
[ -s .env ] && echo "✅ .env has content" || echo "❌ .env is empty"

# 2. Verify .gitignore excludes .env
grep -q "^\.env$" .gitignore && echo "✅ .gitignore excludes .env" || echo "❌ .gitignore misconfigured"

# 3. Test python-dotenv installation
python3 -c "from dotenv import load_dotenv" 2>/dev/null && echo "✅ python-dotenv installed" || echo "❌ python-dotenv missing"

# 4. Test variable loading in Python
python3 -c "from dotenv import load_dotenv; load_dotenv(); print(os.getenv('DATABASE_URL')[:20])" 2>/dev/null && echo "✅ Variables load in Python" || echo "❌ Python loading failed"

# 5. Test .bashrc exports
source ~/.bashrc 2>/dev/null
[ -n "$ZERO2HERO_DEBUG" ] && echo "✅ .bashrc exports working" || echo "❌ .bashrc exports failed"

# 6. Verify .env.example exists and is commitable
[ -f .env.example ] && echo "✅ .env.example exists" || echo "❌ .env.example missing"
git ls-files .env.example | grep -q ".env.example" && echo "✅ .env.example is tracked" || echo "❌ .env.example not tracked"

# 7. Security check: ensure .env is NOT tracked
git ls-files .env | grep -q ".env" && echo "❌ SECURITY: .env is tracked!" || echo "✅ .env is NOT tracked (secure)"

echo ""
echo "🎉 Environment variables configured securely!"
```

---

## Troubleshooting Common Issues

| Issue | Likely Cause | Solution |
|-------|-------------|----------|
| `ModuleNotFoundError: No module named 'dotenv'` | python-dotenv not installed or not in PATH | Reinstall: `pip3 install --user python-dotenv`; ensure `~/.local/bin` is in PATH |
| Variables not loading in Python | `load_dotenv()` called after `os.getenv()`, or `.env` not in project root | Call `load_dotenv()` first; verify `.env` is in same directory as script or use `load_dotenv('.env')` |
| `.bashrc` changes not taking effect | Syntax error in `.bashrc`, or shell not reloaded | Run `source ~/.bashrc`; check for errors with `bash -n ~/.bashrc` |
| `git add .env` doesn't ignore file | `.gitignore` not in project root, or rule has typo | Ensure `.gitignore` is in `~/zero2hero`; check for trailing spaces in patterns |
| `set -a` command not found | Using `sh` or `zsh` instead of `bash` | Ensure you're using bash: `echo $SHELL`; change if needed |
| Variables show as empty in Python | `.env` file has syntax errors (spaces around `=`) | Use `KEY=value` format; no spaces; quote values with spaces |

### Making `.bashrc` Changes Persistent

If environment variables reset after reboot:

1. Verify `.bashrc` is sourced by your shell:
   ```bash
   grep "source.*bashrc" ~/.bash_profile ~/.profile 2>/dev/null || echo "Add: source ~/.bashrc"
   ```

2. Add this to `~/.bash_profile` if it doesn't exist:
   ```bash
   if [ -f ~/.bashrc ]; then
       source ~/.bashrc
   fi
   ```

---

## Security Best Practices

Follow these habits to keep your projects secure:

### ✅ DO:
1. **Never commit `.env` files** — always add them to `.gitignore`
2. **Use `.env.example`** to document required variables for teammates
3. **Rotate secrets immediately** if accidentally committed — use GitHub's "Revoke" button for exposed tokens
4. **Use different values for development vs production** — never reuse dev credentials in prod
5. **Validate required variables** at application startup with clear error messages
6. **Mask sensitive values** in logs and debug output

### ❌ DON'T:
1. **Hardcode secrets** in source code, even in comments
2. **Commit `.env` files** "just this once" — it only takes one mistake
3. **Use the same API key** across multiple projects or environments
4. **Log full environment variables** — mask or hash sensitive values
5. **Store secrets in client-side code** — environment variables are for server-side only

### 🔐 Advanced: Secret Managers for Production

For production deployments, consider dedicated secret management:

| Tool | Best For | Learning Resource |
|------|----------|------------------|
| **HashiCorp Vault** | Enterprise secrets management | [vaultproject.io](https://www.vaultproject.io) |
| **AWS Secrets Manager** | AWS-hosted applications | [AWS Docs](https://docs.aws.amazon.com/secretsmanager) |
| **Doppler** | Developer-friendly secret sync | [doppler.com](https://www.doppler.com) |
| **GitHub Secrets** | CI/CD pipeline secrets | [GitHub Docs](https://docs.github.com/en/actions/security-guides/encrypted-secrets) |

> 💡 This course teaches `.env` files because they're universal and beginner-friendly. Secret managers are covered in later episodes on deployment.

---

## Resources & Further Reading

### Official Documentation
- [python-dotenv Documentation](https://saurabh-kumar.com/python-dotenv/)
- [Git Ignore Patterns](https://git-scm.com/docs/gitignore)
- [Bash Environment Variables Guide](https://www.gnu.org/software/bash/manual/html_node/Environment.html)
- [Twelve-Factor App: Config](https://12factor.net/config) — Industry standard for config management

### Security References
- [OWASP: Credential Management](https://owasp.org/www-project-cheat-sheets/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html)
- [GitHub: Revoking Compromised Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
- [NIST: Digital Identity Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)

### Course-Specific Resources
- [Zero to Hero GitHub Repo](https://github.com/S-V-J/zero2hero)
- [Discord Community](https://discord.gg/rE4gXC36) — Get help, share progress
- [YouTube Playlist](https://www.youtube.com/@heroStackAcademy) — Watch all episodes

---

> 💬 **Stuck?** Join Discord: [discord.gg/rE4gXC36](https://discord.gg/rE4gXC36)
> ⭐ **Found this helpful?** Star the repo: [github.com/S-V-J/zero2hero](https://github.com/S-V-J/zero2hero)
> 💙 **Support This Course**: [github.com/sponsors/S-V-J](https://github.com/sponsors/S-V-J)

---

## 📝 Quick Reference Commands

```bash
# Environment Variable Management
touch .env                          # Create .env file
code .env                           # Edit in VS Code
source ~/.bashrc                    # Reload shell configuration

# Python dotenv Usage
pip3 install python-dotenv          # Install library
python3 -c "from dotenv import load_dotenv; load_dotenv(); print(os.getenv('VAR'))"  # Test loading

# Git Security Checks
git status                          # Verify .env is untracked
git ls-files .env                   # Confirm .env is NOT committed
git add .env.example                # Safe to commit template

# Bash Export Patterns
export MY_VAR="value"               # Simple export
set -a; source .env; set +a         # Auto-export all from file
echo $MY_VAR                        # Verify variable is set

# Security Verification
grep -q "^\.env$" .gitignore && echo "Secure" || echo "Fix .gitignore"
python3 -c "import os; print('OK' if os.getenv('SECRET') else 'Missing')"
```

---

> 🚀 **Pro Tip**: Bookmark this page. You'll reference these patterns throughout the course — and in your professional career.

*Last Updated: 23/04/2026 • Course Status: In Progress • MIT Licensed Code*
```
