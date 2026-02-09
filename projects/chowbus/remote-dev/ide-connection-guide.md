# Remote Development IDE Connection Guide

Connect your local IDE to the Kubernetes-based development workspace running in EKS (staging-02, menu-ai namespace).

## Prerequisites

### 1. kubectl Access

Ensure you have kubectl configured for the staging-02 cluster:

```bash
# Verify cluster access
kubectl config current-context
# Should show: arn:aws:eks:us-east-2:930998754881:cluster/eks-staging-02

# Test access to menu-ai namespace
kubectl get pods -n menu-ai
```

If not configured, contact the infrastructure team for EKS access.

### 2. SSH Key

You need an SSH key pair. Check if you have one:

```bash
ls ~/.ssh/id_ed25519.pub || ls ~/.ssh/id_rsa.pub
```

If not, generate one:

```bash
ssh-keygen -t ed25519 -C "your-email@chowbus.com"
```

### 3. Add Your SSH Key to the Workspace

Copy your public key to the remote workspace:

```bash
# Get your public key
cat ~/.ssh/id_ed25519.pub  # or ~/.ssh/id_rsa.pub

# Add it to the workspace (replace YOUR_PUBLIC_KEY with actual key)
kubectl exec -n menu-ai deployment/coder-workspace-manual -- bash -c "
mkdir -p /home/coder/.ssh
echo 'YOUR_PUBLIC_KEY' >> /home/coder/.ssh/authorized_keys
chmod 700 /home/coder/.ssh
chmod 600 /home/coder/.ssh/authorized_keys
chown -R coder:coder /home/coder/.ssh
"
```

Or use this one-liner:

```bash
kubectl exec -n menu-ai deployment/coder-workspace-manual -- bash -c "mkdir -p /home/coder/.ssh && cat >> /home/coder/.ssh/authorized_keys && chmod 700 /home/coder/.ssh && chmod 600 /home/coder/.ssh/authorized_keys && chown -R coder:coder /home/coder/.ssh" < ~/.ssh/id_ed25519.pub
```

### 4. Configure SSH

Add this to your `~/.ssh/config`:

```
Host coder-workspace
  ProxyCommand kubectl exec -i -n menu-ai deployment/coder-workspace-manual -- nc localhost 22
  User coder
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

### 5. Test SSH Connection

```bash
ssh coder-workspace "echo 'Connection successful!' && whoami"
```

Expected output:
```
Connection successful!
coder
```

---

## VS Code Remote-SSH

### Installation

1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. Install the **Remote - SSH** extension:
   - Open VS Code
   - Press `Cmd+Shift+X` (Mac) or `Ctrl+Shift+X` (Windows/Linux)
   - Search for "Remote - SSH"
   - Install the extension by Microsoft

### Connecting

1. Press `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux)
2. Type **"Remote-SSH: Connect to Host"**
3. Select **"coder-workspace"**
4. A new VS Code window opens connected to the remote workspace
5. Open a folder: **File → Open Folder → `/home/coder`**

### First-time Setup

On first connection, VS Code will:
- Install VS Code Server on the remote (~1-2 minutes)
- You may see "Setting up SSH Host" progress

### Tips

- Install extensions on the remote by clicking "Install in SSH: coder-workspace"
- Your extensions and settings sync separately for local and remote

---

## JetBrains IntelliJ IDEA (via Gateway)

### Option A: JetBrains Gateway (Recommended)

#### Installation

1. Download [JetBrains Gateway](https://www.jetbrains.com/remote-development/gateway/)
   - Or install via JetBrains Toolbox → Gateway

#### Connecting

1. Open **JetBrains Gateway**
2. Under "Remote Development", click **"SSH"**
3. Click the **"+"** button (New Connection)
4. Configure connection:
   | Field | Value |
   |-------|-------|
   | Username | `coder` |
   | Host | `coder-workspace` |
   | Port | `22` |
   | Authentication | OpenSSH config and agent |
5. Click **"Check Connection and Continue"**
6. Select **"IntelliJ IDEA"** and choose version (Community or Ultimate)
7. Set **Project directory**: `/home/coder` or specific project path
8. Click **"Download and Start IDE"**

#### First-time Setup

On first connection, Gateway will:
- Download and install IntelliJ backend on remote (~2-5 minutes)
- Launch the thin client locally

### Option B: From IntelliJ IDEA directly

If you already have IntelliJ installed:

1. Open **IntelliJ IDEA**
2. Go to **File → Remote Development → SSH**
3. Click **"New Connection"**
4. Configure:
   - Host: `coder-workspace`
   - Username: `coder`
   - Authentication: OpenSSH config and agent
5. Click **"Check Connection and Continue"**
6. Select project directory: `/home/coder`
7. Click **"Download IDE and Connect"**

---

## Workspace Environment

The remote workspace includes:

| Tool | Version |
|------|---------|
| JDK | 21.0.9 (OpenJDK) |
| Maven | 3.6.3 |
| Node.js | 22.x |
| npm | included |
| yarn | installed globally |
| pnpm | installed globally |
| Claude Code CLI | 2.1.5 |
| kubectl | latest |
| AWS CLI | v2 |
| Git | latest |

### Verify Tools

```bash
ssh coder-workspace "java -version && mvn -version && node -v && claude --version"
```

---

## Troubleshooting

### SSH Connection Fails

**Error:** `kex_protocol_error` or connection timeout

**Solution:** SSH server may not be running. Start it:

```bash
kubectl exec -n menu-ai deployment/coder-workspace-manual -- bash -c "
sudo /usr/sbin/sshd || (sudo apt-get update && sudo apt-get install -y openssh-server && sudo mkdir -p /run/sshd && sudo ssh-keygen -A && sudo /usr/sbin/sshd)
"
```

### Permission Denied (publickey)

**Solution:** Your SSH key is not in the workspace. Re-add it:

```bash
kubectl exec -n menu-ai deployment/coder-workspace-manual -- bash -c "mkdir -p /home/coder/.ssh && cat >> /home/coder/.ssh/authorized_keys && chmod 700 /home/coder/.ssh && chmod 600 /home/coder/.ssh/authorized_keys && chown -R coder:coder /home/coder/.ssh" < ~/.ssh/id_ed25519.pub
```

### kubectl Command Not Found

**Solution:** Install kubectl:

```bash
# macOS
brew install kubectl

# Or download directly
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### VS Code: "Could not establish connection"

1. Check SSH works: `ssh coder-workspace "echo test"`
2. Check VS Code Remote-SSH logs: `Cmd+Shift+P` → "Remote-SSH: Show Log"
3. Try reloading: `Cmd+Shift+P` → "Developer: Reload Window"

### IntelliJ: IDE Backend Won't Start

**Possible cause:** Insufficient memory

**Solution:** Close other IDE connections (e.g., VS Code) or check pod resources:

```bash
kubectl exec -n menu-ai deployment/coder-workspace-manual -- free -h
```

### Pod Restarted - SSH Not Working

After pod restart, SSH server needs to be reinstalled:

```bash
kubectl exec -n menu-ai deployment/coder-workspace-manual -- bash -c "
sudo apt-get update && sudo apt-get install -y openssh-server netcat-openbsd
sudo mkdir -p /run/sshd
sudo ssh-keygen -A
sudo /usr/sbin/sshd
"
```

Then re-add your SSH key (see Prerequisites step 3).

---

## Quick Reference

### Connect via VS Code
```
Cmd+Shift+P → Remote-SSH: Connect to Host → coder-workspace
```

### Connect via IntelliJ
```
File → Remote Development → SSH → coder-workspace
```

### Manual SSH Access
```bash
ssh coder-workspace
```

### Check Workspace Status
```bash
kubectl get pod -n menu-ai -l app=coder-workspace-manual
```

### View Workspace Logs
```bash
kubectl logs -n menu-ai deployment/coder-workspace-manual --tail=50
```

---

## Support

For issues with:
- **EKS/kubectl access** → Infrastructure team
- **IDE connection** → Check this guide's troubleshooting section
- **Workspace tools/packages** → DevOps team
