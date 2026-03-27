---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "setup-arm-dev-machine",
  "title": "Setup ARM Dev Machine",
  "version": "1.2.0",
  "protocol_version": "1.0",
  "status": "Stable",
  "last_updated": "2026-03-27",
  "objective": "Bootstrap a fresh ARM Ubuntu server into a fully configured imboard development machine with Node, pnpm, AWS CLI, secrets, and keep-alive cron",
  "description": "Connects to a freshly provisioned ARM Ubuntu server via SSH and installs the complete imboard development stack. Works on both Hetzner CAX and Oracle A1 instances. Includes Oracle-specific keep-alive cron to prevent reclamation.",
  "category": [
    "devops",
    "development"
  ],
  "tags": [
    "arm",
    "ubuntu",
    "dev-machine",
    "bootstrap",
    "node",
    "pnpm"
  ],
  "tools_required": [
    {
      "name": "ssh",
      "check_command": "ssh -V"
    },
    {
      "name": "aws",
      "check_command": "aws --version",
      "install_url": "https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html"
    }
  ],
  "inputs": {
    "server_ip": {
      "type": "string",
      "description": "Public IP address of the target server",
      "required": true
    },
    "ssh_user": {
      "type": "string",
      "description": "SSH username (root for Hetzner, ubuntu for Oracle)",
      "default": "root"
    },
    "provider": {
      "type": "string",
      "description": "Cloud provider: hetzner or oracle",
      "default": "hetzner",
      "enum": [
        "hetzner",
        "oracle"
      ]
    },
    "dev_username": {
      "type": "string",
      "description": "Non-root user to create for development (use same username as local machine for path consistency)",
      "default": "yuvaldim"
    },
    "telegram_bot_token": {
      "type": "string",
      "description": "Telegram bot token for notifications",
      "sensitive": true
    },
    "telegram_chat_id": {
      "type": "string",
      "description": "Telegram chat ID for notifications"
    }
  },
  "outputs": {
    "ssh_command": {
      "type": "string",
      "description": "SSH command to connect as dev user"
    },
    "node_version": {
      "type": "string",
      "description": "Installed Node.js version"
    }
  },
  "checksum": {
    "algorithm": "sha256",
    "hash": "e3f26d20b2d2f69e92c4c30b0b6b9630c867584e6241d07cd903fa85f5ec89e3"
  },
  "risk_level": "medium",
  "risk_factors": [
    "requires_credentials",
    "network_access",
    "executes_external_code"
  ],
  "requires_approval": true,
  "destructive_operations": [
    "Installs system packages on the remote server",
    "Creates user accounts and modifies sudoers",
    "Configures firewall rules",
    "Writes AWS credentials to the server"
  ],
  "estimated_duration": {
    "min_minutes": 10,
    "max_minutes": 25
  }
}
---

# Setup ARM Dev Machine

Bootstrap a fresh ARM Ubuntu server into a complete imboard development environment.

## Overview

This dossier transforms a bare ARM Ubuntu server (Hetzner CAX or Oracle A1) into a ready-to-use dev machine:

- **System**: build-essential, git, tmux, htop, jq, fail2ban
- **Runtime**: Node 22 (via nvm), pnpm, Python 3
- **Cloud tools**: AWS CLI v2 (aarch64), Claude Code
- **Security**: UFW firewall (SSH only), fail2ban, non-root dev user
- **Secrets**: AWS SSM Parameter Store integration via `sync-secrets.sh`
- **Keep-alive**: Cron job to prevent Oracle reclamation (Oracle provider only)

## Steps

### Step 1: System Packages

SSH into the server and install base packages.

```bash
ssh {{ssh_user}}@{{server_ip}} << 'REMOTE'
set -euo pipefail
export DEBIAN_FRONTEND=noninteractive

apt-get update -y && apt-get upgrade -y
apt-get install -y \
  curl wget git unzip build-essential \
  python3 python3-pip python3-venv \
  fail2ban ufw \
  htop tmux jq zsh \
  stress-ng
REMOTE
```

### Step 2: Firewall + Security

```bash
ssh {{ssh_user}}@{{server_ip}} << 'REMOTE'
set -euo pipefail

# UFW
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw --force enable

# fail2ban
systemctl enable fail2ban
systemctl start fail2ban
REMOTE
```

### Step 3: Create Dev User

```bash
ssh {{ssh_user}}@{{server_ip}} << 'REMOTE'
set -euo pipefail

# Create user with sudo, copy SSH keys
useradd -m -s /bin/zsh -G sudo {{dev_username}}
echo "{{dev_username}} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/{{dev_username}}

# Copy SSH authorized_keys from root/ubuntu
mkdir -p /home/{{dev_username}}/.ssh
cp ~/.ssh/authorized_keys /home/{{dev_username}}/.ssh/
chown -R {{dev_username}}:{{dev_username}} /home/{{dev_username}}/.ssh
chmod 700 /home/{{dev_username}}/.ssh
chmod 600 /home/{{dev_username}}/.ssh/authorized_keys
REMOTE
```

### Step 4: Node.js + pnpm (as dev user)

```bash
ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
set -euo pipefail

# oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# nvm + Node 22
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"
nvm install 22
nvm alias default 22

# pnpm
npm install -g pnpm

echo "Node: $(node --version)"
echo "pnpm: $(pnpm --version)"
REMOTE
```

### Step 5: AWS CLI v2 (aarch64)

```bash
ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
set -euo pipefail

curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o /tmp/awscli.zip
unzip -q /tmp/awscli.zip -d /tmp/aws-install
sudo /tmp/aws-install/aws/install
rm -rf /tmp/awscli.zip /tmp/aws-install

echo "AWS CLI: $(aws --version)"
REMOTE
```

### Step 6: AWS Credentials for SSM

Configure the `oracle-dev-reader` IAM user credentials (created by Terraform in issue #274).

```bash
ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
mkdir -p ~/.aws
cat > ~/.aws/config << 'AWSCONF'
[default]
region = us-east-1
output = json
AWSCONF

cat > ~/.aws/credentials << 'AWSCREDS'
[default]
aws_access_key_id = PLACEHOLDER
aws_secret_access_key = PLACEHOLDER
AWSCREDS
chmod 600 ~/.aws/credentials
REMOTE
```

> **Manual step**: Replace PLACEHOLDER values with the `oracle-dev-reader` IAM credentials.
> Run `aws ssm get-parameters-by-path --path /imboard/development --with-decryption` to verify access.

### Step 7: Clone Repo + Sync Secrets

Mirror the local dev directory structure (`~/projects/imboard/imboard-monorepo/main/`) so paths are copy-pasteable between machines.

```bash
ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
set -euo pipefail
export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh"

# Mirror local directory structure: ~/projects/imboard/imboard-monorepo
mkdir -p ~/projects/imboard
git clone https://github.com/imboard-ai/imboard-monorepo.git ~/projects/imboard/imboard-monorepo
cd ~/projects/imboard/imboard-monorepo/main

# Install dependencies
pnpm install

# Sync secrets from SSM
bash scripts/sync-secrets.sh imboard/development packages/backend/.env.development
REMOTE
```

### Step 8: Install Dev Tools (Claude Code, gh, Fly, Wrangler, Dossier CLI)

```bash
ssh {{ssh_user}}@{{server_ip}} << 'REMOTE'
set -euo pipefail

# gh CLI
(type -p wget >/dev/null || (sudo apt update && sudo apt-get install wget -y))
sudo mkdir -p -m 755 /etc/apt/keyrings
out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update -qq && sudo apt install gh -y -qq
REMOTE

ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
set -euo pipefail
export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh"

# Claude Code
npm install -g @anthropic-ai/claude-code

# Fly.io CLI
curl -sL https://fly.io/install.sh | sh

# Cloudflare Wrangler
npm install -g wrangler

# Dossier CLI (from ai-dossier monorepo)
cd /tmp && git clone https://github.com/imboard-ai/ai-dossier.git && cd ai-dossier
npm install --workspaces && npm run build --workspace=packages/core && npm run build --workspace=cli
npm install -g .
cd ~ && rm -rf /tmp/ai-dossier

echo "Claude Code: $(claude --version)"
echo "gh: $(gh --version | head -1)"
echo "Fly: $($HOME/.fly/bin/fly version)"
echo "Wrangler: $(wrangler --version)"
REMOTE
```

### Step 9: Configure Shell (SSM-sourced .zshrc)

Pull secrets from SSM and write a portable `.zshrc`. This avoids hardcoding tokens.

```bash
# Run from local machine (has full AWS access)
CF_KEY=$(aws ssm get-parameter --name /imboard/development/cloudflare-api-key --with-decryption --query Parameter.Value --output text)
CF_EMAIL=$(aws ssm get-parameter --name /imboard/development/cloudflare-email --query Parameter.Value --output text)
HANEST_BOT=$(aws ssm get-parameter --name /imboard/development/hanest-telegram-bot-token --with-decryption --query Parameter.Value --output text)
HANEST_CHAT=$(aws ssm get-parameter --name /imboard/development/hanest-telegram-chat-id --query Parameter.Value --output text)
VERCEL=$(aws ssm get-parameter --name /imboard/development/vercel-token --with-decryption --query Parameter.Value --output text)
GH_MCP_PAT=$(aws ssm get-parameter --name /imboard/development/github-mcp-pat --with-decryption --query Parameter.Value --output text)

cat > /tmp/remote-zshrc << ZSHRC
export ZSH="\$HOME/.oh-my-zsh"
ZSH_THEME="agnoster"
plugins=(git)
source \$ZSH/oh-my-zsh.sh

# nvm
export NVM_DIR="\$HOME/.nvm"
[ -s "\$NVM_DIR/nvm.sh" ] && \. "\$NVM_DIR/nvm.sh"
[ -s "\$NVM_DIR/bash_completion" ] && \. "\$NVM_DIR/bash_completion"

# PATH
export PATH="./node_modules/.bin:\$HOME/.local/bin:\$PATH"
export FLYCTL_INSTALL="\$HOME/.fly"
export PATH="\$FLYCTL_INSTALL/bin:\$PATH"

# Cloudflare
export CLOUDFLARE_API_KEY="${CF_KEY}"
export CLOUDFLARE_EMAIL="${CF_EMAIL}"

# Telegram
export HANEST_TELEGRAM_BOT_TOKEN="${HANEST_BOT}"
export HANEST_TELEGRAM_CHAT_ID="${HANEST_CHAT}"

# Vercel
export VERCEL_TOKEN="${VERCEL}"

# GitHub MCP
export GITHUB_MCP_PAT="${GH_MCP_PAT}"

# Claude Code
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Start in projects
cd ~/projects/imboard 2>/dev/null

# Ensure cron is running
CRON_RUNNING=\$(service cron status 2>/dev/null | grep -q "is running" && echo "running")
if [ -z "\$CRON_RUNNING" ]; then
    sudo service cron start > /dev/null 2>&1
fi
ZSHRC

scp /tmp/remote-zshrc {{dev_username}}@{{server_ip}}:~/.zshrc
rm /tmp/remote-zshrc
```

### Step 10: Authenticate CLIs (interactive)

SSH in and authenticate each CLI. These require browser interaction.

```bash
ssh {{dev_username}}@{{server_ip}}

# Claude Code — just run `claude`, it will prompt for auth
claude

# GitHub CLI
gh auth login
# → GitHub.com → HTTPS → Paste authentication token

# Fly.io (prints URL to open in browser)
fly auth login

# Cloudflare — already authenticated via CLOUDFLARE_API_KEY env var
wrangler whoami
```

### Step 12: Keep-Alive Cron (Oracle only)

Skip this step if `{{provider}}` is `hetzner`.

Oracle reclaims Always Free instances if CPU, network, AND memory ALL stay below 20% for 7 days.

```bash
ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
cat > ~/keep-alive.sh << 'KEEPALIVE'
#!/bin/bash
# 60s CPU burst every 5 min = ~20% avg duty cycle
stress-ng --cpu $(nproc) --cpu-load 20 -t 60s >/dev/null 2>&1
curl -s -o /dev/null https://ifconfig.me
KEEPALIVE
chmod +x ~/keep-alive.sh

(crontab -l 2>/dev/null; echo "*/5 * * * * ~/keep-alive.sh >> ~/keep-alive.log 2>&1") | crontab -
echo "Keep-alive cron installed"
REMOTE
```

### Step 13: Notify

```bash
PUBLIC_IP={{server_ip}}
curl -s "https://api.telegram.org/bot{{telegram_bot_token}}/sendMessage" \
  -d chat_id="{{telegram_chat_id}}" \
  -d parse_mode="HTML" \
  -d text="<b>Dev Machine Ready</b>
IP: <code>${PUBLIC_IP}</code>
SSH: <code>ssh {{dev_username}}@${PUBLIC_IP}</code>
Provider: {{provider}}
Node: $(ssh {{dev_username}}@${PUBLIC_IP} 'source ~/.nvm/nvm.sh && node --version')
Keep-alive: $(if [ '{{provider}}' = 'oracle' ]; then echo 'active'; else echo 'n/a (hetzner)'; fi)"
```

## Transfer: Hetzner → Oracle

When the Oracle A1 instance is claimed:

1. Run this dossier on the Oracle instance with `provider=oracle`, `ssh_user=ubuntu`
2. Copy any local state (git branches, .env overrides) from Hetzner
3. Verify the Oracle machine works
4. Tear down Hetzner (use the teardown section in `provision-arm-vps` dossier)

## Verification

```bash
ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
echo "=== Verification ==="
source ~/.nvm/nvm.sh
echo "Node: $(node --version)"
echo "pnpm: $(pnpm --version)"
echo "AWS CLI: $(aws --version 2>&1)"
echo "Git: $(git --version)"
echo "Arch: $(uname -m)"
echo "RAM: $(free -h | awk '/Mem:/{print $2}')"
echo "Disk: $(df -h / | awk 'NR==2{print $2}')"
echo "UFW: $(sudo ufw status | head -1)"
aws ssm get-parameters-by-path --path /imboard/development --max-results 1 --query "Parameters[0].Name" --output text 2>/dev/null && echo "SSM: OK" || echo "SSM: needs credentials"
REMOTE
```
