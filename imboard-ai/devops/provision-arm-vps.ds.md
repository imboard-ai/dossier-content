---
name: 'provision-arm-vps'
description: 'Creates a Hetzner Cloud ARM server with Ubuntu 24.04, firewall, and SSH key. Designed as a throwaway bridge until Oracle Always Free A1 instance is claimed. Provider-agnostic inputs where possible.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Provision ARM VPS'
  dossier.version: '1.0.2'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Stable'
  dossier.last_updated: '2026-03-27'
  dossier.objective: 'Provision an ARM-based VPS on Hetzner Cloud (CAX series) via hcloud CLI, output SSH access details'
  dossier.category: '["devops","infrastructure"]'
  dossier.tags: '["hetzner","arm","vps","provisioning","cloud"]'
  dossier.tools_required: '[{"check_command":"hcloud version","install_url":"https://github.com/hetznercloud/cli","name":"hcloud"},{"check_command":"ssh-keygen -V","name":"ssh-keygen"}]'
  dossier.estimated_duration: '{"max_minutes":10,"min_minutes":3}'
  dossier.risk_level: 'high'
  dossier.risk_factors: '["modifies_cloud_resources","requires_credentials","incurs_cost"]'
  dossier.requires_approval: 'true'
  dossier.destructive_operations: '["Creates a billable Hetzner Cloud server (~€12.49/mo for CAX31)","Uploads SSH key to Hetzner Cloud account"]'
  dossier.content_scope: 'references-external'
  dossier.external_references: '[{"description":"Documentation link mentioned in the body: console.hetzner.cloud","required":false,"trust_level":"trusted","type":"documentation","url":"https://console.hetzner.cloud"}]'
  dossier.inputs: '{"image":{"default":"ubuntu-24.04","description":"OS image","type":"string"},"location":{"default":"fsn1","description":"Datacenter location (fsn1/nbg1/hel1)","type":"string"},"server_name":{"default":"imboard-dev","description":"Name for the server","type":"string"},"server_type":{"default":"cax31","description":"Hetzner server type (cax11/cax21/cax31/cax41)","type":"string"},"ssh_key_path":{"default":"~/.ssh/id_rsa.pub","description":"Path to SSH public key","type":"string"}}'
  dossier.outputs: '{"server_ip":{"description":"Public IPv4 address of the provisioned server","type":"string"},"ssh_command":{"description":"Full SSH command to connect","type":"string"}}'
  dossier.checksum: '{"algorithm":"sha256","hash":"554db5d895fdb548620b4206cc2b9fdad1da43bad2b554775e3ca9777d03b292"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"Cd7G+ZyPgusyrOy0GGY5SpCpX0Wg7glu9/tF/Z4vsdd/iAFPpVxsYdvK+OprL7rdO9dEY/zCX2Y87dYdWT2lDA==","signed_at":"2026-10-07T12:10:57.698Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
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
