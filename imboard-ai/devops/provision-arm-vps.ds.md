---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "provision-arm-vps",
  "title": "Provision ARM VPS",
  "version": "1.0.0",
  "protocol_version": "1.0",
  "status": "Stable",
  "last_updated": "2026-03-27",
  "objective": "Provision an ARM-based VPS on Hetzner Cloud (CAX series) via hcloud CLI, output SSH access details",
  "description": "Creates a Hetzner Cloud ARM server with Ubuntu 24.04, firewall, and SSH key. Designed as a throwaway bridge until Oracle Always Free A1 instance is claimed. Provider-agnostic inputs where possible.",
  "category": [
    "devops",
    "infrastructure"
  ],
  "tags": [
    "hetzner",
    "arm",
    "vps",
    "provisioning",
    "cloud"
  ],
  "tools_required": [
    {
      "name": "hcloud",
      "check_command": "hcloud version",
      "install_url": "https://github.com/hetznercloud/cli"
    },
    {
      "name": "ssh-keygen",
      "check_command": "ssh-keygen -V"
    }
  ],
  "inputs": {
    "server_name": {
      "type": "string",
      "description": "Name for the server",
      "default": "imboard-dev"
    },
    "server_type": {
      "type": "string",
      "description": "Hetzner server type (cax11/cax21/cax31/cax41)",
      "default": "cax31"
    },
    "location": {
      "type": "string",
      "description": "Datacenter location (fsn1/nbg1/hel1)",
      "default": "fsn1"
    },
    "ssh_key_path": {
      "type": "string",
      "description": "Path to SSH public key",
      "default": "~/.ssh/id_rsa.pub"
    },
    "image": {
      "type": "string",
      "description": "OS image",
      "default": "ubuntu-24.04"
    }
  },
  "outputs": {
    "server_ip": {
      "type": "string",
      "description": "Public IPv4 address of the provisioned server"
    },
    "ssh_command": {
      "type": "string",
      "description": "Full SSH command to connect"
    }
  },
  "checksum": {
    "algorithm": "sha256",
    "hash": "554db5d895fdb548620b4206cc2b9fdad1da43bad2b554775e3ca9777d03b292"
  },
  "risk_level": "high",
  "risk_factors": [
    "modifies_cloud_resources",
    "requires_credentials",
    "incurs_cost"
  ],
  "requires_approval": true,
  "destructive_operations": [
    "Creates a billable Hetzner Cloud server (~€12.49/mo for CAX31)",
    "Uploads SSH key to Hetzner Cloud account"
  ],
  "estimated_duration": {
    "min_minutes": 3,
    "max_minutes": 10
  }
}
---

# Provision ARM VPS

Provision an ARM-based development server on Hetzner Cloud.

## Prerequisites

### 1. Hetzner Cloud Account + API Token

- Sign up at https://console.hetzner.cloud
- Create a project (e.g., "imboard-dev")
- Go to Security → API Tokens → Generate API Token (Read & Write)
- Save the token securely

### 2. Install hcloud CLI

```bash
# Linux (amd64)
curl -sSL https://github.com/hetznercloud/cli/releases/latest/download/hcloud-linux-amd64.tar.gz | sudo tar -C /usr/local/bin -xz hcloud

# Verify
hcloud version
```

### 3. Authenticate

```bash
hcloud context create imboard-dev
# Paste API token when prompted
```

## Steps

### Step 1: Upload SSH Key

```bash
hcloud ssh-key create \
  --name "dev-key" \
  --public-key-from-file {{ssh_key_path}}
```

### Step 2: Create Firewall

```bash
hcloud firewall create --name "dev-firewall" \
  --rules-file /dev/stdin <<'EOF'
[
  {"description": "SSH", "direction": "in", "protocol": "tcp", "port": "22", "source_ips": ["0.0.0.0/0", "::/0"]},
  {"description": "ICMP", "direction": "in", "protocol": "icmp", "source_ips": ["0.0.0.0/0", "::/0"]}
]
EOF
```

### Step 3: Create Server

```bash
hcloud server create \
  --name {{server_name}} \
  --type {{server_type}} \
  --location {{location}} \
  --image {{image}} \
  --ssh-key "dev-key" \
  --firewall "dev-firewall"
```

### Step 4: Extract IP and Verify SSH

```bash
SERVER_IP=$(hcloud server ip {{server_name}})
echo "Server IP: $SERVER_IP"
echo "SSH: ssh root@$SERVER_IP"

# Wait for SSH to be ready
sleep 10
ssh -o StrictHostKeyChecking=accept-new root@$SERVER_IP "uname -a && echo 'SSH OK'"
```

## Outputs

- `server_ip`: IPv4 address from `hcloud server ip`
- `ssh_command`: `ssh root@<server_ip>`

## Teardown

When Oracle A1 instance is ready:

```bash
hcloud server delete {{server_name}}
hcloud firewall delete dev-firewall
hcloud ssh-key delete dev-key
```
