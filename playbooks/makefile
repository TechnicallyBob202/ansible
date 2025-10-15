.PHONY: help deploy cleanup start stop validate rebuild-ad

# Environment variables
export ANSIBLE_CONFIG := ansible.cfg
export ANSIBLE_INVENTORY := inventory/adfr-lab.yml

help:
	@echo "ADFR/DSP Lab Management"
	@echo ""
	@echo "Available targets:"
	@echo "  deploy       - Full lab deployment (VMs + Network + AD)"
	@echo "  cleanup      - Destroy all VMs"
	@echo "  start        - Start all VMs"
	@echo "  stop         - Stop all VMs"
	@echo "  validate     - Verify lab configuration"
	@echo "  rebuild-ad   - Rebuild AD only (VMs must exist)"
	@echo ""
	@echo "Individual steps:"
	@echo "  deploy-vms   - Deploy VMs only"
	@echo "  config-net   - Configure networking only"
	@echo "  build-ad     - Build AD forests only"
	@echo ""

deploy:
	ansible-playbook playbooks/all-in-one-deploy.yml

cleanup:
	ansible-playbook playbooks/00-cleanup-lab.yml

start:
	ansible-playbook playbooks/start-all-vms.yml

stop:
	ansible-playbook playbooks/stop-all-vms.yml

validate:
	ansible-playbook playbooks/04-verify-lab.yml

rebuild-ad:
	ansible-playbook playbooks/rebuild-ad-only.yml

deploy-vms:
	ansible-playbook playbooks/01-deploy-infrastructure.yml

config-net:
	ansible-playbook playbooks/02-configure-base-os.yml

build-ad:
	ansible-playbook playbooks/03-build-ad-forests.yml