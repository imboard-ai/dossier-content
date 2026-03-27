---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "setup-arm-dev-machine",
  "title": "Setup ARM Dev Machine",
  "version": "1.0.0",
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
      "description": "Non-root user to create for development",
      "default": "dev"
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
    "hash": "f33e26e48380011824e83df5efc7bf629b7e4e27a285114776aeacac1c902945"
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

```bash
ssh {{dev_username}}@{{server_ip}} << 'REMOTE'
set -euo pipefail
export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh"

# Clone
git clone https://github.com/imboard-ai/imboard-monorepo.git ~/imboard
cd ~/imboard

# Install dependencies
pnpm install

# Sync secrets from SSM
bash scripts/sync-secrets.sh imboard/development packages/backend/.env.development
REMOTE
```

### Step 8: Keep-Alive Cron (Oracle only)

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

### Step 9: Notify

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
