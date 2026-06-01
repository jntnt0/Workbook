### Practical Mapping to Your Pillars

| Your Pillar             | Repo                                                          |
| ----------------------- | ------------------------------------------------------------- |
| Landing Zones           | `Azure/ALZ-Bicep` or `terraform-azurerm-caf-enterprise-scale` |
| Networking              | `ALZ-Bicep` hub-spoke modules + AVM networking modules        |
| Identity (AD DS)        | `ALZ-Bicep` identity subscription + Azure Arc                 |
| Security & Governance   | `ALZ-Bicep` policy assignments + AVM Key Vault                |
| Compute / Storage / DBs | `Azure/bicep-registry-modules` (AVM)                          |
| Monitoring              | AVM `operational-insights` modules                            |
| Backup & DR             | AVM `recovery-services` modules                               |
| Cost Management         | Azure Migrate assessments + Hybrid Benefit tooling            |
| Migration Execution     | `Azure/migration` workload templates                          |
| Operational Handoff     | `Azure/DevOps-Landing-Zone` pipelines                         |