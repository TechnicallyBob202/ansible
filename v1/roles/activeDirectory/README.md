# Active Directory Role

This role manages Active Directory forest and domain deployment, including forest roots, replica DCs, and child domains.

## Requirements

- Windows Server 2016 or higher
- `microsoft.ad` collection
- `ansible.windows` collection
- PowerShell 5.1 or higher
- Sufficient privileges (local Administrator)

## Role Variables

### Required Variables (from inventory)
- `domain_name` - Fully qualified domain name (e.g., arbor.lab)
- `netbios_name` - NetBIOS domain name (e.g., ARBOR)
- `parent_domain` - Parent domain (only for child domains)

### Optional Variables (with defaults)
- `ad_safe_mode_password` - DSRM password (from env var)
- `ad_database_path` - AD database location (default: C:\Windows\NTDS)
- `ad_sysvol_path` - SYSVOL location (default: C:\Windows\SYSVOL)
- `ad_install_dns` - Install DNS service (default: yes)
- `ad_promotion_timeout` - Timeout for DC promotion (default: 1200s)

## Operations

Set `ad_operation` variable to control which tasks run:

- `check_domain_status` - Check if domain exists and get info
- `install_ad_features` - Install AD-DS features and tools
- `promote_forest_root` - Promote to forest root PDC
- `promote_replica_dc` - Promote to replica DC in existing domain
- `promote_child_domain` - Create and promote child domain
- `wait_for_dc_ready` - Wait for DC services to be operational
- `verify_replication` - Check AD replication health
- `create_service_accounts` - Create ADFR/DSP service accounts
- `check_fsmo_roles` - Display FSMO role placement

## Deployment Order

### 1. Forest Root PDCs (parallel)
```yaml
- hosts: arbor_forest_root_pdc:alpine_forest_root_pdc
  serial: 1
  roles:
    - role: activeDirectory
      vars:
        ad_operation: promote_forest_root
```

### 2. Forest Root Replicas
```yaml
- hosts: arbor_forest_root_replica:alpine_forest_root_replica
  roles:
    - role: activeDirectory
      vars:
        ad_operation: promote_replica_dc
```

### 3. Child Domain PDCs
```yaml
- hosts: mauna_child_pdc:rainier_child_pdc
  serial: 1
  roles:
    - role: activeDirectory
      vars:
        ad_operation: promote_child_domain
```

### 4. Child Domain Replicas
```yaml
- hosts: mauna_child_replica:rainier_child_replica
  roles:
    - role: activeDirectory
      vars:
        ad_operation: promote_replica_dc
```

## Example Playbook
```yaml
---
- name: Build AD Forests
  hosts: domain_controllers
  gather_facts: no

  tasks:
    - name: Install AD features
      include_role:
        name: activeDirectory
        tasks_from: install_ad_features.yml

    - name: Check domain status
      include_role:
        name: activeDirectory
        tasks_from: check_domain_status.yml

    - name: Promote based on role
      include_role:
        name: activeDirectory
      vars:
        ad_operation: "{{ dc_operation }}"
```

## Service Accounts

The role creates service accounts with:
- Domain Admin privileges
- Replicating Directory Changes permissions
- Password never expires
- Account cannot be disabled

Accounts created:
- `svc-adfr` - For ADFR agent
- `svc-dsp` - For DSP monitoring

## FSMO Roles

Default placement:
- **Forest Roles** (Schema, Domain Naming): First DC in forest root
- **Domain Roles** (PDC, RID, Infrastructure): First DC in each domain

## Validation

After deployment, validate with:
```bash
ansible-playbook -i inventory/adfr-lab.yml playbooks/verify-ad-health.yml
```

## Notes

- All promotions trigger automatic reboots
- Child domains require parent domain Administrator credentials
- Replication checks may take several minutes
- Domain SIDs are saved to `/tmp/domain-sid-*.txt`
- FSMO roles are saved to `/tmp/fsmo-roles-*.txt`