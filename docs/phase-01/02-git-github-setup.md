# Episode 2: Git + SSH + GitHub Repository Setup

> 🎯 **Goal**: Configure professional version control with SSH authentication and make your first commit.

📺 **Watch on YouTube**: [Episode 2: Git Setup](https://youtu.be/hBRxGIyZDnc)

---

## 📋 Table of Contents

- [What is Git and Why Use It?](#what-is-git-and-why-use-it)
- [Why SSH Authentication?](#why-ssh-authentication)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [What's Next](#whats-next)

---

## What is Git and Why Use It?

**Git** is a distributed version control system that tracks changes in your code over time. It allows you to:

| Benefit | Explanation |
|---------|-------------|
| 🔁 Revert Safely | Go back to any previous version if something breaks |
| 👥 Collaborate | Work with others without overwriting each other's work |
| 🌿 Experiment | Create branches to test ideas without affecting main code |
| 📊 Build Portfolio | Your GitHub profile becomes your resume for employers |

### Why We Use Git from Day One

1. **Backup & Recovery** — Your code lives both locally and in the cloud
2. **Professional Workflow** — Every company uses Git; learn it now
3. **Collaboration** — Share progress with the Discord community
4. **Portfolio Building** — Employers review your commit history and code quality

> 💡 Git is not optional. It's essential. And we're mastering it today.

---

## Why SSH Authentication?

SSH (Secure Shell) keys provide **passwordless, secure authentication** between your computer and GitHub.

| Method | Pros | Cons |
|--------|------|------|
| 🔑 SSH Keys | Secure, no password prompts, works in scripts | One-time setup required |
| 🔐 HTTPS + Token | Simple initial setup | Requires token management, less secure for automation |

For professional development workflows, **SSH is the industry standard**.

---

## Prerequisites

- Ubuntu 24.04 running in WSL2 (from Episode 1)
- VS Code with Remote-WSL extension
- GitHub account: [github.com](https://github.com)
- Basic terminal familiarity

---

## Step-by-Step Setup

### Step 1: Check Existing SSH Keys

```bash
ls -la ~/.ssh
```

If the `.ssh` folder doesn't exist, create it:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

### Step 2: Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "stjl093@gmail.com" -f ~/.ssh/zero2hero-github
```

**Parameters Explained**:
| Flag | Purpose |
|------|---------|
| `-t ed25519` | Encryption type (modern, secure, fast) |
| `-C "stjl093@gmail.com"` | Adds your email as a label (use your own) |
| `-f ~/.ssh/zero2hero-github` | Output filename (keeps it organized) |

When prompted for a passphrase:
- Press Enter twice to skip (simpler for learning)
- Or enter a secure passphrase for extra protection

### Step 3: Start SSH Agent

```bash
# Start the agent in current session
eval "$(ssh-agent -s)"

# Add your private key to the agent
ssh-add ~/.ssh/zero2hero-github

# Verify the key is loaded
ssh-add -l
```

**Expected Output**:
```
Agent pid 12345
256 SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxx stjl093@gmail.com (ED25519)
```

### Step 4: Copy Your Public Key

```bash
cat ~/.ssh/zero2hero-github.pub
```

This displays a string starting with `ssh-ed25519`. **Select and copy the entire output** — from `ssh-ed25519` to the end.

### Step 5: Add Key to GitHub

1. Go to [GitHub.com](https://github.com) and log in
2. Click your profile picture → **Settings**
3. In the left sidebar: **SSH and GPG keys**
4. Click the green **New SSH key** button
5. Fill in the form:
   - **Title**: `ZeroToHero-Course - WSL2 Ubuntu`
   - **Key**: Paste your public key (from Step 4)
   - **Type**: Select **Authentication** (not Signing)
6. Click **Add SSH key**
7. Enter your GitHub password if prompted

### Step 6: Test SSH Connection

```bash
ssh -T git@github.com
```

**First-time connection**:
```
The authenticity of host 'github.com (20.207.73.82)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

Type `yes` and press Enter.

**Success message**:
```
Hi S-V-J! You've successfully authenticated, but GitHub does not provide shell access.
```

> ✅ This confirms GitHub recognizes your key. The "no shell access" message is normal.

### Step 7: Create GitHub Repository

1. Go to [GitHub.com](https://github.com)
2. Click the **+** icon (top-right) → **New repository**
3. Fill in:
   - **Repository name**: `zero2hero`
   - **Visibility**: **Public** (this is an open-source course)
   - **Initialize this repository with**: Uncheck all boxes (we have local files)
4. Click **Create repository**

### Step 8: Clone or Initialize Repository

**Option A: Fresh Clone (Recommended for new setups)**
```bash
cd ~
mkdir zero2hero
cd zero2hero
git clone git@github.com:S-V-J/zero2hero.git .
```

**Option B: Initialize Existing Folder**
```bash
cd /your/existing/folder
git init
git remote add origin git@github.com:S-V-J/zero2hero.git
git branch -M main
```

### Step 9: Verify Git Connection

```bash
# Check remote URL
git remote -v
# Expected: origin  git@github.com:S-V-J/zero2hero.git (fetch)
#           origin  git@github.com:S-V-J/zero2hero.git (push)

# Check branches
git branch -a
# Expected: * main
#           remotes/origin/main

# List files
ls -la
# Expected: .git, LICENSE, README.md, etc.
```

### Step 10: Configure Git User

```bash
# Set your name (appears on every commit)
git config --global user.name "Siddhant Kumar"

# Set your email (linked to GitHub account)
git config --global user.email "stjl093@gmail.com"

# Verify configuration
git config --global --list
```

### Step 11: Make Your First Commit

```bash
# Create a welcome file
echo "# 👋 Welcome to Zero to Hero Course" > WELCOME.md
echo "Environment: WSL2 Ubuntu 24.04 + VS Code + SSH Auth" >> WELCOME.md
echo "Date: $(date)" >> WELCOME.md

# Stage the file
git add WELCOME.md

# Commit with descriptive message
git commit -m "chore: add welcome file from $(whoami)@$(hostname)"

# Push to GitHub
git push origin main
```

### Step 12: Verify on GitHub

1. Open your browser to: [https://github.com/S-V-J/zero2hero](https://github.com/S-V-J/zero2hero)
2. Refresh the page
3. Confirm you see:
   - `WELCOME.md` in the file list
   - Your commit message in the history
   - Your username as the committer

---

## Verification

Run these commands to confirm everything is working:

```bash
# Test SSH authentication
ssh -T git@github.com

# Check Git configuration
git config --global --list

# Verify remote connection
git remote -v

# Check commit history
git log --oneline -5
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `Permission denied (publickey)` | Re-check SSH key was added to GitHub correctly; ensure `ssh-add` was run |
| `fatal: remote origin already exists` | Run `git remote remove origin` first, then `git remote add origin ...` |
| `fatal: not a git repository` | Run `git init` in your project folder first |
| `fatal: refusing to merge unrelated histories` | Use `git pull origin main --allow-unrelated-histories` |
| SSH key not loading after reboot | Add `eval "$(ssh-agent -s)"` and `ssh-add ~/.ssh/zero2hero-github` to `~/.bashrc` |
| GitHub shows "Unverified" email | Verify your email in GitHub Settings → Emails |

### Making SSH Persistent Across Sessions

To avoid re-adding your key after reboot:

```bash
# Edit your bash configuration
nano ~/.bashrc

# Add these lines at the end:
if [ -z "$SSH_AUTH_SOCK" ]; then
  eval "$(ssh-agent -s)"
fi
ssh-add ~/.ssh/zero2hero-github 2>/dev/null

# Save and exit (Ctrl+X, Y, Enter)
# Reload configuration
source ~/.bashrc
```
---

## 🔗 Resources

- [Official Git Documentation](https://git-scm.com/doc)
- [GitHub SSH Guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [SSH Key Types Explained](https://www.ssh.com/academy/ssh/keygen)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)

---

## 📝 Quick Reference Commands

```bash
# SSH Key Management
ssh-keygen -t ed25519 -C "email@example.com" -f ~/.ssh/keyname
ssh-add ~/.ssh/keyname
ssh-add -l                      # List loaded keys
ssh-add -d ~/.ssh/keyname       # Remove a key

# Git Basics
git init                        # Initialize new repository
git clone <url>                 # Clone existing repository
git status                      # Check repository status
git add <file>                  # Stage changes
git commit -m "message"         # Commit changes
git push origin main            # Push to remote
git pull origin main            # Pull from remote

# Git Configuration
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git config --global --list      # View all config

# Troubleshooting Git
git remote -v                   # Check remote URLs
git branch -a                   # List all branches
git log --oneline -10           # View recent commits
```
---