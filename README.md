# K3s Deploy with Ansible

## Overview

This guide describes how to deploy K3s using Ansible playbooks through configd.

## Configuration Files

### 1. ansible-vars-config/vars.yml

Define the `host_type` variable to specify whether the node is RC (server) or Drone (agent).

**For RC (Server) deployment:**
```yaml
host_type: rc
k3s_token: pegasus_k3s_server_token
```

**For Drone (Agent) deployment:**
```yaml
host_type: drone
controller_addr: <RC_IP_ADDRESS>  # Specify the RC server IP address
k3s_token: pegasus_k3s_server_token
```

**Note:** When `host_type` is set to `drone`, you must specify the RC server's IP address in the `controller_addr` field.

### 2. configd-config/k3s_cluster.yml

This file is used by configd to execute the K3s deployment.

```yaml
apiVersion: configd/v1
kind: TaskList
metadata:
  name: k3s-installation

cmd: >-
  ansible-playbook k3s_playbook.yml
  -i localhost,
  -e "@/mnt/ansible-vars-config/vars.yml"
```

## Deployment Steps

1. **Prepare the configuration file**
   - Edit `ansible-vars-config/vars.yml`
   - Set `host_type` to `rc` or `drone`
   - If deploying as Drone, specify the RC server IP address

2. **Deploy using configd**
   - Place the configuration file in the configd config directory
   - Configd will execute the deployment automatically

3. **Deployment Order**
   - Deploy RC (server) nodes first
   - Wait for RC nodes to be ready
   - Deploy Drone (agent) nodes

## Variables Reference

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| host_type | Node type: `rc` (server) or `drone` (agent) | Yes | `rc` |
| controller_addr | RC server IP address (required for drone mode) | Only for drone | `10.96.157.15` |
| k3s_token | K3s cluster token | Yes | `pegasus_k3s_server_token` |

## Example Configurations

### Example 1: Deploy RC Server
```yaml
# ansible-vars-config/vars.yml
host_type: rc
k3s_token: pegasus_k3s_server_token
```

### Example 2: Deploy Drone Agent
```yaml
# ansible-vars-config/vars.yml
host_type: drone
controller_addr: 192.168.1.100
k3s_token: pegasus_k3s_server_token
```

## Verification

After deployment, verify the installation:

**On RC (Server) node:**
```bash
sudo k3s kubectl get nodes
```

**Check K3s service status:**
```bash
# For RC (Server)
sudo systemctl status k3s

# For Drone (Agent)
sudo systemctl status k3s-agent
```

## Troubleshooting

- Ensure the RC server is fully started before deploying Drone agents
- Verify network connectivity between Drone and RC nodes (port 6443)
- Check firewall settings to allow K3s communication ports (6443, 8472)
