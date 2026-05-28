Fair point. Here's the full list:

**Deploy immediately:**

- 2x Domain Controllers
- 1x Entra Connect server
- Optional 1x Issuing CA
- 1x Azure Firewall
- 1x VPN Gateway or ExpressRoute Gateway
- 1x Azure Private DNS Resolver
- 1x Azure Bastion
- 1x Log Analytics Workspace (configure DCRs and solutions at deployment, not later)
- 1x Recovery Services Vault
- 1x Azure Migrate Project
- 1x Windows test VM
- 1x Linux test VM
- 1x Source-side Azure Migrate appliance

**Do not deploy yet (workload-specific targets, not foundational):**

- 2x SQL servers (primary + witness/AG node)
- 2x File servers (source identity cutover target + Storage Migration Service orchestrator)
- 2x IIS/App servers (one Windows, one to validate multi-site/binding scenarios)
- 2x AVD/RDS hosts (session host pool minimum for meaningful testing)
- 1x Print server
- 1x WSUS
- 1x Jump box (Bastion covers this)
- 1x Monitoring VM (Log Analytics + DCRs covers this)