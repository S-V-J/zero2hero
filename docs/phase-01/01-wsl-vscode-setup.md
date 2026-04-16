# Episode 1: WSL2 + Ubuntu 24.04 + VS Code Setup

> 🎯 **Goal**: Get a clean Ubuntu 24.04 system ready for development — no extra steps.

📺 **Watch on YouTube**: [Episode 1: WSL Setup](https://youtu.be/Ke6eLofGDp0)

---

## 📋 Table of Contents

- [Why WSL2?](#why-wsl2)
- [Alternatives (If You Prefer)](#alternatives-if-you-prefer)
- [Prerequisites](#prerequisites)
- [Step-by-Step Setup](#step-by-step-setup)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [What's Next](#whats-next)

---

## Why WSL2?

We use **WSL2 (Windows Subsystem for Linux)** because it provides:

| Benefit | Explanation |
|---------|-------------|
| ⚡ Native Performance | Runs a real Linux kernel with near-native speed — no emulation overhead |
| 🔗 Seamless Integration | Access Windows files from Linux and vice versa. Use VS Code on Windows to edit Linux code |
| 💾 Lightweight | Uses less RAM and disk than full VMs. Starts in seconds |
| 🛠️ Official Microsoft Support | Built into Windows 10/11. Regular updates. Enterprise-ready |

For this course, WSL2 means: faster setup, less frustration, and a real Linux environment to build in.

---

## Alternatives (If You Prefer)

WSL2 is our default, but you can use:

| Option | Best For | Notes |
|--------|----------|-------|
| 🖥️ VMware / VirtualBox | Full VM control, offline work | Heavier on resources; requires manual ISO setup |
| ☁️ AWS / Azure / GCP | Cloud deployment practice, team collaboration | Costs money; requires cloud account setup |
| 🍎 macOS with Docker/Multipass | Mac users | Works well — adjust commands for your system |

💡 *Want to learn more about these? Search YouTube for "WSL2 vs VMware" or "Ubuntu on AWS for beginners".*

---

## Prerequisites

- Windows 10 version 2004 or higher (Build 19041 or higher) or Windows 11
- Virtualization enabled in BIOS/UEFI
- ~20GB free disk space
- Administrator access to enable WSL feature

---

## Step-by-Step Setup

### 1. Open PowerShell as Administrator

- Press `Win + X` → Select "Windows Terminal (Admin)" or "PowerShell (Admin)"
- Confirm the User Account Control prompt

### 2. Install Ubuntu 24.04 via WSL2

```powershell
wsl --install -d Ubuntu-24.04 --name ZeroToHero-Course --web-download
```

**Command Breakdown**:
| Flag | Purpose |
|------|---------|
| `--install` | Installs WSL and the specified distribution |
| `-d Ubuntu-24.04` | Specifies Ubuntu 24.04 LTS as the distribution |
| `--name ZeroToHero-Course` | Names the WSL instance for easy identification |
| `--web-download` | Uses Microsoft's CDN for faster, more reliable download |

> 💡 If you get an error about virtualization, enable it in BIOS/UEFI and reboot.

### 3. Create Your Linux User

When the installation completes, a terminal window will open asking for:

```
Enter new UNIX username: zero2hero
New password: ********
Retype new password: ********
```

> ⚠️ **Important**: When typing your password, **nothing will appear on screen** — no asterisks, no dots. This is normal Linux behavior. Just type and press Enter.

### 4. Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### 5. Navigate to Project Folder

```bash
cd ~
cd /home
ls
cd zero2hero
```

> 📁 If the `zero2hero` folder doesn't exist yet, that's okay — we'll create it in the next episode.

### 6. Launch VS Code

```bash
code .
```

> 🔗 This requires the [Remote - WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl), which installs automatically on first run.

### 7. Verify Your Location

In the VS Code integrated terminal:

```bash
pwd
# Output should be: /home/zero2hero
```

### 8. Test Reopening VS Code

- Close VS Code
- Reopen it from the Welcome page or Start menu
- Confirm it reconnects to WSL automatically

---

## Verification

Run these commands to confirm everything is working:

```bash
# Check WSL version
wsl --list --verbose

# Check Ubuntu version inside WSL
lsb_release -a

# Check VS Code connection
code --version
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `wsl : The term 'wsl' is not recognized` | Enable WSL feature: `wsl --install` (requires reboot) |
| Virtualization not enabled | Reboot → Enter BIOS/UEFI → Enable Intel VT-x or AMD-V |
| VS Code doesn't open | Install VS Code: [code.visualstudio.com](https://code.visualstudio.com) |
| `code: command not found` | In VS Code: `Ctrl+Shift+P` → "Shell Command: Install 'code' in PATH" |
| Permission denied on folders | Run `sudo chown -R $USER:$USER /home/zero2hero` |
| Slow file access between Windows/WSL | Keep project files inside WSL filesystem (`/home/...`), not `/mnt/c/...` |

---

## 🔗 Resources

- [Official WSL Documentation](https://learn.microsoft.com/en-us/windows/wsl/)
- [VS Code Remote - WSL](https://code.visualstudio.com/docs/remote/wsl)
- [Ubuntu 24.04 Release Notes](https://discourse.ubuntu.com/t/noble-numbat-release-notes/39890)
- [Windows Terminal Guide](https://learn.microsoft.com/en-us/windows/terminal/)

---
## 📝 Quick Reference Commands

```bash
# WSL Management
wsl --list --verbose          # List installed distributions
wsl --terminate ZeroToHero-Course  # Stop the WSL instance
wsl --shutdown                # Shut down all WSL instances

# Inside Ubuntu
cd ~                          # Go to home directory
ls -la                        # List all files (including hidden)
pwd                           # Print working directory
code .                        # Open current folder in VS Code

# VS Code Tips
Ctrl+`                        # Toggle integrated terminal
Ctrl+Shift+P                  # Open command palette
Ctrl+Shift+E                  # Open file explorer
```
---