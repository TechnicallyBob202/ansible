# Windows Base Configuration Role

This role handles basic Windows Server configuration including networking, hostname, and system validation.

## Requirements

- Windows Server 2016 or higher
- WinRM enabled and accessible
- `ansible.windows` collection
- PowerShell 5.1 or higher on target systems

## Role Variables

### Required Variables (from inventory)
- `ansible_host` - Target IP address
- `inventory_hostname` - Target hostname

### Optional Variables (with defaults)
- `windows_admin_user` - Admin username (default: Administrator)
- `windows_admin_password` - Admin password (from env var)
- `windows_gateway` - Default gateway (default: 192.168.35.1)
- `windows_dns_servers` - List of DNS servers
- `windows_dns_suffix` - DNS suffix (default: lab)
- `windows_ip_prefix` - IP prefix length (default: 24)

### Variables from Proxmox role (if using DHCP discovery)
- `vmid` - Proxmox VM ID
- `proxmox_host` - Proxmox host IP

## Operations

Set `windows_operation` variable to control which tasks run:

- `set_static_ip` - Configure static IP only
- `set_dns` - Configure DNS servers only
- `set_hostname` - Set hostname only
- `reboot` - Reboot system
- `configure_network` - Full network config (IP + DNS + hostname + reboot)
- `validate` - Validate configuration

## Example Playbook
```yaml
- hosts: all
  roles:
    - role: windows
      vars:
        windows_operation: configure_network
```

## DHCP Discovery

If VMs boot with DHCP initially, use `discover_dhcp_ip.yml`:
```yaml
- include_role:
    name: windows
    tasks_from: discover_dhcp_ip.yml
```

## Validation

After configuration, validate settings:
```yaml
- hosts: all
  roles:
    - role: windows
      vars:
        windows_operation: validate
```

## Example Usage
```bash
# Configure all VMs
ansible-playbook -i inventory/adfr-lab.yml playbooks/02-configure-base-os.yml

# Validate configuration
ansible-playbook -i inventory/adfr-lab.yml playbooks/validate-network.yml
```

## Notes

- IP configuration will cause a brief network interruption
- Hostname changes require a reboot
- DNS validation assumes domain controllers are configured
- DHCP discovery requires SSH access to Proxmox hosts