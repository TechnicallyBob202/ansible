# Proxmox VM Management Role

This role manages Proxmox VMs through the Proxmox API.

## Requirements

- `community.general` collection
- Proxmox VE 6.x or higher
- Valid Proxmox API credentials

## Role Variables

### Required Variables (from inventory)
- `vmid` - Proxmox VM ID
- `proxmox_host` - Proxmox API host IP
- `proxmox_node` - Proxmox node name
- `template_id` - Template VM ID to clone from
- `vm_cores` - Number of CPU cores
- `vm_memory` - RAM in MB
- `vm_disk` - Disk size (e.g., "80G")

### Optional Variables (with defaults)
- `proxmox_user` - API user (default: root@pam)
- `proxmox_password` - API password (from env var)
- `proxmox_pool` - VM pool name (default: ADFR-DSP)
- `proxmox_network_vlan` - VLAN tag (default: 35)
- `proxmox_vm_tags` - VM tags (default: adfr-dsp)

## Operations

Set `proxmox_operation` variable to control which tasks run:

- `clone` - Clone VM from template only
- `configure_network` - Configure network only
- `configure_hardware` - Configure CPU/RAM/disk only
- `start` - Start VM only
- `stop` - Stop VM only
- `destroy` - Delete VM completely
- `deploy` - Full deployment (clone + configure + start)

## Example Playbook
```yaml
- hosts: all
  roles:
    - role: proxmox
      vars:
        proxmox_operation: deploy
```

## Example Usage
```bash
# Deploy all VMs
ansible-playbook -i inventory/adfr-lab.yml playbooks/01-deploy-infrastructure.yml

# Stop all VMs
ansible-playbook -i inventory/adfr-lab.yml playbooks/stop-all-vms.yml

# Destroy all VMs
ansible-playbook -i inventory/adfr-lab.yml playbooks/00-cleanup-lab.yml
```