# Semperis ADFR/DSP Lab Automation

Automated deployment of a complete Semperis Active Directory Forest Recovery (ADFR) and Directory Services Protector (DSP) lab environment using Ansible.

## Overview

This project automates the deployment of:
- 15 Virtual Machines across 2 Proxmox hosts
- 2 Active Directory Forests (arbor.lab, alpine.lab)
- 4 Domains (1 root + 3 child domains)
- 8 Domain Controllers
- Infrastructure services (ADFR, DSP, SQL, Distribution Points)
- Recovery environment VMs

## Architecture
```
arbor.lab Forest (Single Domain)
├── KOA-DC (PDC)
└── OAK-DC (Replica)

alpine.lab Forest (Multi-Domain)
├── alpine.lab (Root)
│   ├── GRANITE-DC (PDC)
│   └── BASALT-DC (Replica)
├── mauna.alpine.lab (Child)
│   ├── SLATE-DC (PDC)
│   └── SHALE-DC (Replica)
└── rainier.alpine.lab (Child)
    ├── MARBLE-DC (PDC)
    └── QUARTZ-DC (Replica)

Infrastructure
├── ATLAS (ADFR Management)
├── SUMMIT (DSP Management)
├── PEAK (SQL Server)
├── RIDGE (Primary DP)
└── CREST (Secondary DP)

Recovery VMs
├── MESA
└── BLUFF
```

## Prerequisites

### Control Node Requirements
- Linux/WSL/macOS
- Ansible 2.15+
- Python 3.8+
- SSH access to Proxmox hosts

### Proxmox Requirements
- Proxmox VE 7.x or 8.x
- 2 Proxmox hosts (semperis-nuc-1, semperis-nuc-2)
- Windows Server 2022 templates (VM IDs 9001, 9002)
- Network configuration:
  - VLAN 35 (192.168.35.0/24) for lab VMs
  - VLAN 33 (192.168.33.0/24) for Proxmox management

### Required Collections
```bash
ansible-galaxy collection install -r requirements.yml
```

## Quick Start

### 1. Configure Environment Variables
```bash
export WINDOWS_PASSWORD="YourWindowsPassword"
export proxmox-root-password="YourProxmoxPassword"
```

### 2. Update Inventory

Edit `inventory/adfr-lab.yml` to match your environment:
- Proxmox host IPs
- VM IDs
- Network configuration

### 3. Deploy Lab
```bash
# Full deployment (recommended for first run)
make deploy

# Or individual steps
make deploy-vms      # Deploy VMs only
make config-net      # Configure networking
make build-ad        # Build AD forests
make validate        # Verify configuration
```

## Usage

### Management Commands
```bash
make help           # Show all available commands
make deploy         # Full lab deployment
make cleanup        # Destroy all VMs
make start          # Start all VMs
make stop           # Stop all VMs
make validate       # Verify lab configuration
make rebuild-ad     # Rebuild AD only (VMs must exist)
```

### Manual Playbook Execution
```bash
# Full deployment
ansible-playbook playbooks/all-in-one-deploy.yml

# Individual playbooks
ansible-playbook playbooks/01-deploy-infrastructure.yml
ansible-playbook playbooks/02-configure-base-os.yml
ansible-playbook playbooks/03-build-ad-forests.yml
ansible-playbook playbooks/04-verify-lab.yml
```

## Project Structure
```
.
├── ansible.cfg                 # Ansible configuration
├── requirements.yml            # Galaxy dependencies
├── Makefile                    # Make targets for easy execution
│
├── inventory/
│   ├── adfr-lab.yml           # Main inventory (YAML)
│   └── group_vars/            # Group variables
│
├── roles/
│   ├── proxmox/               # Proxmox VM management
│   │   ├── tasks/
│   │   └── defaults/
│   ├── windows/               # Windows base configuration
│   │   ├── tasks/
│   │   └── defaults/
│   └── activeDirectory/       # AD deployment
│       ├── tasks/
│       └── defaults/
│
├── playbooks/
│   ├── 00-cleanup-lab.yml
│   ├── 01-deploy-infrastructure.yml
│   ├── 02-configure-base-os.yml
│   ├── 03-build-ad-forests.yml
│   ├── 04-verify-lab.yml
│   └── all-in-one-deploy.yml
│
├── templates/
│   ├── lab-summary.md.j2
│   └── ansible-inventory.j2
│
└── files/
    ├── domain-sids.txt        # Generated domain SIDs
    ├── fsmo-roles.txt         # Generated FSMO information
    └── adfr-lab-mremoteng.xml # Remote desktop connections
```

## Roles

### proxmox
Manages VM lifecycle on Proxmox hosts.

**Operations:**
- `clone` - Clone from template
- `configure_network` - Set VLAN tags
- `configure_hardware` - Set CPU/RAM/disk
- `start` - Start VM
- `stop` - Stop VM
- `destroy` - Delete VM
- `deploy` - Full deployment

### windows
Handles Windows Server base configuration.

**Operations:**
- `set_static_ip` - Configure static IP
- `set_dns` - Configure DNS servers
- `set_hostname` - Set hostname
- `reboot` - Reboot and wait
- `configure_network` - Full network config
- `validate` - Validate configuration

### activeDirectory
Deploys and manages Active Directory.

**Operations:**
- `check_domain_status` - Check domain state
- `install_ad_features` - Install AD-DS
- `promote_forest_root` - Create new forest
- `promote_replica_dc` - Add replica DC
- `promote_child_domain` - Create child domain
- `wait_for_dc_ready` - Wait for services
- `verify_replication` - Check replication
- `create_service_accounts` - Create service accounts
- `check_fsmo_roles` - Display FSMO placement

## Deployment Timeline

| Phase | Duration | Tasks |
|-------|----------|-------|
| VM Deployment | 10-15 min | Clone and configure VMs |
| Network Configuration | 5-10 min | Set IPs, DNS, hostnames |
| AD Forest Deployment | 40-50 min | Promote all DCs |
| Validation | 5 min | Verify configuration |
| **Total** | **60-90 min** | **Full lab deployment** |

## Troubleshooting

### VMs not responding
```bash
# Check VM status
make start

# Verify connectivity
ansible all -m win_ping
```

### AD promotion failures
```bash
# Check logs
ansible-playbook playbooks/04-verify-lab.yml

# Rebuild AD only
make rebuild-ad
```

### Network issues
```bash
# Reconfigure network
ansible-playbook playbooks/02-configure-base-os.yml --tags validate
```

## Generated Files

After deployment, the following files are generated in `files/`:

- `domain-sids.txt` - Domain Security Identifiers
- `fsmo-roles.txt` - FSMO role placement
- `lab-summary.md` - Complete lab documentation
- `deployment-timestamp.txt` - Deployment metadata

## Next Steps

After successful deployment:

1. Review generated documentation in `files/`
2. Import mRemoteNG configuration
3. Deploy Semperis ADFR/DSP software
4. Configure backup schedules
5. Run test recovery scenarios

## Support

- Semperis Documentation: https://docs.semperis.com
- Semperis Support: https://support.semperis.com

## License

Internal use for Semperis Federal SA demonstrations and training.

---

**Last Updated:** 2025-10-15
**Maintained By:** Semperis Federal SA Team