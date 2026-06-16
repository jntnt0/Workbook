05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Index

05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md
Configure_Azure_Site_Recovery_Failover_And_Test_Failover
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Source_Basis
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Mental_Model
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Planning_Table
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Configuration_Checklist
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_ASR_Prereq_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Replication_Enablement_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Replication_Health_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Test_Failover_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Test_Failover_Cleanup_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Production_Failover_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Commit_And_Reprotect_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Recovery_Plan_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_KQL_Query_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Evidence_Export_Skeleton
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Verification_Commands
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Rollback
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Failure_Checks
Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Related_Labs

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Set up disaster recovery for Azure VMs | Azure VM replication prerequisites, Recovery Services vault, VM replication, Mobility service extension |
| Microsoft Learn | Azure to Azure disaster recovery architecture | Cache storage account, target region replication, recovery point generation, target VM creation |
| Microsoft Learn | Run a disaster recovery drill for Azure VMs | Test failover prerequisites, recovery point selection, non-production network selection, validation, cleanup |
| Microsoft Learn | Fail over Azure VMs to a secondary region | Production failover, recovery point selection, shutdown option, commit, reprotect |
| Microsoft Learn | About recovery plans | App recovery grouping, startup sequencing, automation tasks, manual tasks, recovery plan test failover |
| Microsoft Learn | Site Recovery networking for Azure VMs | Test failover networking, target network mapping, NIC settings, subnet, NSG, public IP, load balancer |
| Microsoft Learn | Site Recovery monitoring | Replication health, job monitoring, events, logs, alerts, Recovery Services vault monitoring |
| Microsoft Learn | Azure Monitor Activity Log | ASR operation tracking and control plane evidence |
| Microsoft Learn | Recovery Services vault diagnostics | Routing ASR logs and jobs to Log Analytics for KQL validation |

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Azure Site Recovery | Disaster recovery service used to replicate protected machines and orchestrate failover |
| Recovery Services vault | Vault used to manage ASR replication, replicated items, recovery plans, jobs, and alerts |
| Source region | Azure region where the production VM currently runs |
| Target region | Azure region where the VM will be created during failover |
| Cache storage account | Source-region storage account used by ASR to stage replicated disk writes before sending them to target region |
| Mobility service extension | Component installed on replicated VMs so VM changes can be captured and sent to ASR |
| Replicated item | Protected VM or machine tracked by ASR inside the Recovery Services vault |
| Replication policy | Retention and app-consistent snapshot schedule used by ASR |
| Recovery point | Crash-consistent or app-consistent point used as the source for failover |
| Latest processed recovery point | Latest already-processed point, usually fastest RTO |
| Latest recovery point | Processes all replicated data before failover, usually best RPO |
| Latest app-consistent recovery point | Latest application-consistent point, useful for app-aware recovery |
| Custom recovery point | Specific recovery point selected for a single-VM failover without recovery plan |
| Test failover | Non-disruptive DR drill that creates test VM resources in target region while production continues running |
| Cleanup test failover | Required step that deletes test failover resources after validation |
| Planned failover | Controlled failover where source shutdown is attempted to minimize data loss |
| Unplanned failover | Disaster event failover when source region or workload is unavailable |
| Commit | Confirms the failover result and removes the option to switch recovery points for that failover |
| Reprotect | Reverses replication direction after failover so the new active VM replicates back toward the original region |
| Recovery plan | Ordered failover plan for one or more protected VMs, including sequencing, automation, and manual steps |
| First rule | Test failover before production failover |
| Blunt rule | Do not commit production failover until the target VM and application are validated |

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Subscription ID | `11111111-1111-1111-1111-111111111111` | `<subscription-id>` |
| Vault resource group | `rg-recovery-core-01` | `<vault-resource-group>` |
| Recovery Services vault | `rsv-core-01` | `<recovery-services-vault-name>` |
| Source region | `eastus` | `<source-region>` |
| Target region | `westus2` | `<target-region>` |
| Source VM resource group | `rg-compute-prod-01` | `<source-vm-resource-group>` |
| Source VM name | `vm-app-01` | `<source-vm-name>` |
| Source VM resource ID | `/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Compute/virtualMachines/<vm>` | `<source-vm-resource-id>` |
| Source VNet | `vnet-prod-eastus-01` | `<source-vnet-name>` |
| Target VNet | `vnet-dr-westus2-01` | `<target-vnet-name>` |
| Test failover VNet | `vnet-drtest-westus2-01` | `<test-failover-vnet-name>` |
| Test subnet | `snet-drtest-workload` | `<test-subnet-name>` |
| Production failover subnet | `snet-dr-workload` | `<target-subnet-name>` |
| Target resource group | `rg-compute-dr-01` | `<target-resource-group>` |
| Cache storage account | `stasrcasrcache01` | `<cache-storage-account>` |
| Replication policy | `asr-policy-24h-app4h` | `<replication-policy-name>` |
| Recovery point choice | `Latest processed` | `<Latest processed / Latest / Latest app-consistent / Custom>` |
| Recovery plan | `rp-three-tier-app-01` | `<recovery-plan-name>` |
| Action group | `ag-core-oncall-email` | `<action-group-name>` |
| Test failover owner | `Cloud Admin` | `<operator>` |
| Test failover evidence path | `C:\ASR-TestFailover` | `<test-evidence-path>` |
| Production failover evidence path | `C:\ASR-ProductionFailover` | `<production-evidence-path>` |
| RTO target | `60 minutes` | `<rto-target>` |
| RPO target | `15 minutes` | `<rpo-target>` |
| Validation method | `Boot VM, login, confirm service, test app URL` | `<validation-method>` |
| Next workbook | `06_Configure_Backup_Reports_Alerts_And_Recovery_Validation.md` | `<next-task>` |

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Configuration_Checklist

| Step | Task | Device | PowerShell / Azure CLI / Portal | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Azure | Admin Workstation / Cloud Shell | `az login` | Azure session is authenticated |
| 2 | Select subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-id>"` | Correct subscription is active |
| 3 | Register Recovery Services provider | Admin Workstation / Cloud Shell | `az provider register --namespace Microsoft.RecoveryServices` | Provider is registered |
| 4 | Register Compute provider | Admin Workstation / Cloud Shell | `az provider register --namespace Microsoft.Compute` | Compute provider is registered |
| 5 | Register Network provider | Admin Workstation / Cloud Shell | `az provider register --namespace Microsoft.Network` | Network provider is registered |
| 6 | Confirm Recovery Services vault | Admin Workstation / Cloud Shell | `az backup vault show --resource-group "<vault-resource-group>" --name "<recovery-services-vault-name>" --output table` | Vault exists |
| 7 | Confirm source VM exists | Admin Workstation / Cloud Shell | `az vm show --resource-group "<source-vm-resource-group>" --name "<source-vm-name>" --output table` | Source VM exists |
| 8 | Confirm target region quota | Azure Portal | Subscriptions > Usage + quotas | Target region has enough VM family quota |
| 9 | Confirm source VM supportability | Azure Portal | VM > Disaster recovery > Review prerequisites | VM meets ASR requirements |
| 10 | Confirm source VM outbound access | Admin Workstation / Azure Portal | Check NSG, route table, firewall, proxy | VM can reach required ASR and Azure endpoints |
| 11 | Confirm target VNet exists | Admin Workstation / Cloud Shell | `az network vnet show --resource-group "<target-resource-group>" --name "<target-vnet-name>" --output table` | Target VNet exists |
| 12 | Confirm test failover VNet exists | Admin Workstation / Cloud Shell | `az network vnet show --resource-group "<target-resource-group>" --name "<test-failover-vnet-name>" --output table` | Non-production test network exists |
| 13 | Enable replication for VM | Azure Portal | Recovery Services vault > Site Recovery > Enable replication | VM appears as replicated item |
| 14 | Confirm replicated item health | Azure Portal | Vault > Replicated items > VM | VM shows protected and healthy |
| 15 | Confirm replication policy | Azure Portal | Vault > Site Recovery infrastructure > Replication policies | Policy is assigned |
| 16 | Confirm recovery points | Azure Portal | Replicated item > Latest recovery points | Recovery points are available |
| 17 | Create recovery plan if app has multiple tiers | Azure Portal | Vault > Recovery plans > Create | Recovery plan exists |
| 18 | Sequence recovery plan groups | Azure Portal | Recovery plan > Customize | Dependency order is documented |
| 19 | Add manual or automation tasks if needed | Azure Portal | Recovery plan > Customize | Required steps are attached |
| 20 | Run test failover | Azure Portal | Replicated item or recovery plan > Test Failover | Test failover job starts |
| 21 | Select recovery point | Azure Portal | Test Failover wizard | Planned recovery point is selected |
| 22 | Select non-production test VNet | Azure Portal | Test Failover wizard | Test VM will not collide with production |
| 23 | Monitor test failover job | Azure Portal | Vault > Site Recovery jobs | Test failover job completes |
| 24 | Validate test VM | Azure Portal / Admin Workstation | Boot, login, service check, app check | VM and workload validate in target test network |
| 25 | Record test failover evidence | Admin Workstation | Export screenshots, job logs, validation notes | Evidence is saved |
| 26 | Cleanup test failover | Azure Portal | Replicated item > Cleanup test failover | Test VM resources are removed |
| 27 | Confirm cleanup completion | Azure Portal | Vault > Jobs and target resource group | Cleanup job completes and test resources are gone |
| 28 | Prepare production failover runbook | Operator | Confirm outage trigger, approvals, DNS, firewall, app owners | Production failover is controlled |
| 29 | Run production failover only when approved | Azure Portal | Replicated item or recovery plan > Failover | Target VM is created in target region |
| 30 | Validate production failed-over VM | Admin Workstation / App owner | Login, health checks, app test, dependency checks | Workload is operational in target region |
| 31 | Commit failover | Azure Portal | Replicated item > Commit | Failover is finalized |
| 32 | Reprotect VM | Azure Portal | Replicated item > Re-Protect | Replication direction reverses |
| 33 | Confirm reprotect health | Azure Portal | Replicated item overview | VM is protected again |
| 34 | Export failover evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Jobs, resource state, and validation evidence are saved |
| 35 | Document RTO and RPO | Operator | Record start time, completion time, selected recovery point | Recovery performance is documented |

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_ASR_Prereq_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: validate high-level Azure prerequisites before configuring ASR.
# ASR replication and failover operations are primarily managed from the Recovery Services vault portal experience.

SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
SOURCE_VM_RG="<source-vm-resource-group>"
SOURCE_VM_NAME="<source-vm-name>"
TARGET_RG="<target-resource-group>"
TARGET_VNET="<target-vnet-name>"
TEST_VNET="<test-failover-vnet-name>"

az login

az account set \
  --subscription "$SUBSCRIPTION_ID"

az provider register \
  --namespace Microsoft.RecoveryServices

az provider register \
  --namespace Microsoft.Compute

az provider register \
  --namespace Microsoft.Network

az provider register \
  --namespace Microsoft.Storage

az provider show \
  --namespace Microsoft.RecoveryServices \
  --query registrationState \
  --output tsv

az backup vault show \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --output table

az vm show \
  --resource-group "$SOURCE_VM_RG" \
  --name "$SOURCE_VM_NAME" \
  --output table

az vm get-instance-view \
  --resource-group "$SOURCE_VM_RG" \
  --name "$SOURCE_VM_NAME" \
  --query "instanceView.statuses" \
  --output table

az network vnet show \
  --resource-group "$TARGET_RG" \
  --name "$TARGET_VNET" \
  --output table

az network vnet show \
  --resource-group "$TARGET_RG" \
  --name "$TEST_VNET" \
  --output table
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Replication_Enablement_Skeleton

```text
Azure Portal path:

1. Open Azure portal.
2. Go to Recovery Services vaults.
3. Open <recovery-services-vault-name>.
4. Select Site Recovery.
5. Under Azure virtual machines, select Enable replication.
6. Source tab:
   - Source location: <source-region>
   - Subscription: <subscription-id>
   - Resource group: <source-vm-resource-group>
   - Deployment model: Resource Manager
   - Disaster recovery between availability zones: <Yes / No>
7. Virtual machines tab:
   - Select <source-vm-name>
   - Select up to 10 VMs if this is a simple batch
8. Replication settings tab:
   - Target location: <target-region>
   - Target subscription: <subscription-id>
   - Target resource group: <target-resource-group>
   - Target virtual network: <target-vnet-name>
   - Target subnet: <target-subnet-name>
   - Cache storage account: <cache-storage-account>
   - Target availability options: <availability set / zone / no infrastructure redundancy>
   - Disk replication setting: confirm all required disks
   - Churn option: Normal unless high-change workload requires high churn support
9. Manage tab:
   - Replication policy: <replication-policy-name>
   - Replication group: <replication-group-name or none>
   - Extension settings: confirm automation account or extension settings
10. Review tab:
   - Confirm all settings.
   - Select Enable replication.
11. Go to Vault > Replicated items.
12. Confirm <source-vm-name> appears as protected.
13. Wait until initial replication completes before test failover.
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Replication_Health_Skeleton

```text
Azure Portal path:

1. Open Recovery Services vault.
2. Select Replicated items.
3. Select <source-vm-name>.
4. Confirm these fields:
   - Replication health: Healthy
   - Failover health: Healthy
   - Agent status: Healthy
   - Last recovery point: recent enough for RPO
   - Latest app-consistent recovery point: present if required
   - Target region: <target-region>
   - Target resource group: <target-resource-group>
   - Target VNet: <target-vnet-name>
   - Disks: all required disks included
5. Open Properties.
6. Record:
   - Policy name
   - Target compute settings
   - Target network settings
   - Extension settings
   - Recovery point retention
   - App-consistent snapshot frequency
7. Open Jobs.
8. Confirm no active failed ASR jobs are blocking failover.
```

```bash
# CLI evidence support.
# Purpose: capture control plane activity related to ASR operations.
# This does not replace the portal health view.

SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
SOURCE_VM_ID="<source-vm-resource-id>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az resource show \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --resource-type "Microsoft.RecoveryServices/vaults" \
  --output jsonc

az monitor activity-log list \
  --resource-id "$SOURCE_VM_ID" \
  --offset 7d \
  --output jsonc

az monitor activity-log list \
  --resource-group "$VAULT_RG" \
  --offset 7d \
  --output jsonc
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Test_Failover_Skeleton

```text
Azure Portal path for single replicated VM:

1. Open Recovery Services vault.
2. Select Replicated items.
3. Select <source-vm-name>.
4. Confirm the VM is protected and healthy.
5. Select Test Failover.
6. Choose recovery point:
   - Latest processed for fastest RTO
   - Latest for lowest RPO
   - Latest app-consistent for app-aware recovery
   - Custom for a specific single-VM recovery point
7. Select Azure virtual network:
   - Use <test-failover-vnet-name>
   - Do not use production VNet for test failover unless the test plan explicitly requires it
8. Select OK.
9. Monitor:
   - Notifications
   - Recovery Services vault > Site Recovery jobs
10. After job completion, open Virtual Machines.
11. Find the test failover VM in the target region.
12. Confirm:
   - VM is running
   - VM size is expected or accepted alternative SKU
   - VM is attached to <test-failover-vnet-name>
   - NIC, NSG, and subnet are correct
   - No production IP collision exists
13. Run workload checks:
   - Boot/login check
   - Disk check
   - Service check
   - Application dependency check
   - Web/API/database health check if applicable
14. Record:
   - Test failover start time
   - Test failover complete time
   - Recovery point selected
   - VM boot state
   - Application validation result
   - Issues found
   - Cleanup status
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Test_Failover_Cleanup_Skeleton

```text
Azure Portal path:

1. Open Recovery Services vault.
2. Select Replicated items.
3. Select <source-vm-name>.
4. Confirm test failover validation is complete.
5. Select Cleanup test failover.
6. In Notes, enter:
   - Test result
   - Recovery point used
   - VM boot result
   - App validation result
   - Network validation result
   - Issues found
   - Whether retest is required
7. Select Testing is complete.
8. Confirm cleanup.
9. Monitor cleanup job.
10. Open target test resource group.
11. Confirm temporary test failover VM resources are deleted:
    - VM
    - NIC
    - Disks
    - Public IP if created
    - Temporary ASR resources
12. Return to replicated item.
13. Confirm replication is still healthy.
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Production_Failover_Skeleton

```text
Production failover path:

1. Confirm production failover approval.
2. Confirm business outage trigger:
   - Source region outage
   - App outage
   - Planned DR exercise
   - Migration event
3. Confirm test failover was completed successfully.
4. Confirm target region capacity and quota.
5. Confirm DNS cutover plan.
6. Confirm firewall and NSG rules in target region.
7. Confirm app owner and database owner are available.
8. Confirm backup and restore posture before failover.
9. Open Recovery Services vault.
10. Select Replicated items or Recovery plans.
11. Select <source-vm-name> or <recovery-plan-name>.
12. Select Failover.
13. Choose recovery point:
    - Latest processed for lower RTO
    - Latest for lower RPO
    - Latest app-consistent for app-aware recovery
    - Custom only for single VM without recovery plan
14. Select Shut down machine before beginning failover when source VM is reachable and controlled failover is desired.
15. Select OK.
16. Monitor failover in:
    - Azure notifications
    - Vault > Site Recovery jobs
    - Target region Virtual Machines
17. After failover, confirm target VM exists.
18. Confirm VM is running.
19. Confirm target VM size is expected or accepted alternative.
20. Confirm NIC, NSG, route table, and subnet placement.
21. Run workload validation.
22. Update DNS, traffic manager, load balancer, or app gateway path according to runbook.
23. Confirm users or test clients can reach application.
24. Do not commit until validation passes.
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Commit_And_Reprotect_Skeleton

```text
Commit path:

1. Open Recovery Services vault.
2. Select Replicated items.
3. Select failed-over VM.
4. Confirm target VM validation passed.
5. Confirm no need exists to change recovery point.
6. Select Commit.
7. Confirm commit.
8. Record commit time.
9. Monitor commit job.
10. Confirm VM status is Failover committed.

Important:
- Commit finalizes the failover.
- After commit, previous recovery points for that failover path are removed.
- Do not commit if the target VM or application has not been validated.

Reprotect path:

1. Confirm failover is committed.
2. Confirm original primary region is reachable if reprotecting back toward the primary region.
3. Confirm permissions to create VMs, disks, NICs, and storage resources in the primary region.
4. Open the replicated item.
5. Select Re-Protect.
6. Confirm replication direction:
   - Current active region: <target-region>
   - Reprotect target region: <source-region>
7. Review resources that ASR will create.
8. Select OK.
9. Monitor reprotect job.
10. Confirm replication health returns to Healthy.
11. Record:
    - Reprotect start time
    - Reprotect completion time
    - New replication direction
    - Any errors or warnings
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Recovery_Plan_Skeleton

```text
Recovery plan design:

Example three-tier app:

Group 1:
- SQL backend VM
- Domain controller or dependency VM if required by app plan

Group 2:
- Middleware VM
- API VM

Group 3:
- Web frontend VM
- Worker VM

Manual actions:
- Confirm database starts
- Confirm middleware service starts
- Confirm DNS cutover
- Confirm load balancer backend pool
- Confirm smoke test

Automation actions:
- Azure Automation runbook to update NSG
- Azure Automation runbook to attach public IP if required
- Azure Automation runbook to restart service
- Script to update app config if required

Azure Portal path:

1. Open Recovery Services vault.
2. Select Recovery plans.
3. Select Create recovery plan.
4. Name: <recovery-plan-name>.
5. Source: <source-region>.
6. Target: <target-region>.
7. Select replicated VMs.
8. Create plan.
9. Open the recovery plan.
10. Select Customize.
11. Add groups based on app dependency order.
12. Move VMs into correct startup groups.
13. Add manual actions where operator decisions are required.
14. Add automation tasks where repeatable steps can be scripted.
15. Save recovery plan.
16. Run test failover on the recovery plan.
17. Cleanup test failover.
18. Update recovery plan based on lessons learned.
19. Retest until recovery is repeatable.
20. Schedule quarterly test failover review.
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_KQL_Query_Skeleton

```kusto
// Purpose: query ASR and Recovery Services activity after diagnostic settings route vault logs to Log Analytics.
// Run in the Log Analytics workspace used by the recovery monitoring baseline.

//
// 1. Recovery Services vault control plane operations.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Caller, CorrelationId
| order by TimeGenerated desc

//
// 2. ASR operation hunting.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| where OperationNameValue has_any ("replication", "failover", "test", "commit", "reprotect", "recovery plan", "protected item")
| summarize Count=count(), LastSeen=max(TimeGenerated) by OperationNameValue, ActivityStatusValue, Caller
| order by LastSeen desc

//
// 3. Failed Recovery Services operations.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| where ActivityStatusValue has_any ("Failed", "Failure")
| project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Caller, Properties, CorrelationId
| order by TimeGenerated desc

//
// 4. Recovery Services diagnostic logs if routed to AzureDiagnostics.
//
AzureDiagnostics
| where TimeGenerated > ago(30d)
| where ResourceProvider =~ "MICROSOFT.RECOVERYSERVICES"
| summarize Count=count(), LastRecord=max(TimeGenerated) by Resource, Category, OperationName, ResultType
| order by LastRecord desc

//
// 5. ASR job or event failure hunting if diagnostic logs are present.
//
AzureDiagnostics
| where TimeGenerated > ago(30d)
| where ResourceProvider =~ "MICROSOFT.RECOVERYSERVICES"
| where ResultType has_any ("Failed", "Failure", "Error") or Level in ("Error", "Critical")
| project TimeGenerated, Resource, Category, OperationName, ResultType, ResultDescription
| order by TimeGenerated desc

//
// 6. Target VM creation evidence after failover or test failover.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.COMPUTE"
| where OperationNameValue has_any ("virtualMachines/write", "disks/write")
| project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Caller
| order by TimeGenerated desc

//
// 7. Network resource creation evidence after failover or test failover.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.NETWORK"
| where OperationNameValue has_any ("networkInterfaces/write", "publicIPAddresses/write", "networkSecurityGroups/write")
| project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Caller
| order by TimeGenerated desc
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Evidence_Export_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: export ASR evidence before and after test failover or production failover.

EVIDENCE_PATH="./ASR-Failover-Evidence"
SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
SOURCE_VM_RG="<source-vm-resource-group>"
SOURCE_VM_NAME="<source-vm-name>"
TARGET_RG="<target-resource-group>"
TARGET_VNET="<target-vnet-name>"
TEST_VNET="<test-failover-vnet-name>"

mkdir -p "$EVIDENCE_PATH"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az account show \
  --output jsonc \
  > "$EVIDENCE_PATH/account-context.jsonc"

az backup vault show \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/recovery-services-vault.jsonc"

az vm show \
  --resource-group "$SOURCE_VM_RG" \
  --name "$SOURCE_VM_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/source-vm.jsonc"

az vm get-instance-view \
  --resource-group "$SOURCE_VM_RG" \
  --name "$SOURCE_VM_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/source-vm-instance-view.jsonc"

az network vnet show \
  --resource-group "$TARGET_RG" \
  --name "$TARGET_VNET" \
  --output jsonc \
  > "$EVIDENCE_PATH/target-vnet.jsonc"

az network vnet show \
  --resource-group "$TARGET_RG" \
  --name "$TEST_VNET" \
  --output jsonc \
  > "$EVIDENCE_PATH/test-failover-vnet.jsonc"

az resource list \
  --resource-group "$TARGET_RG" \
  --output jsonc \
  > "$EVIDENCE_PATH/target-resource-group-resources.jsonc"

az monitor activity-log list \
  --resource-group "$VAULT_RG" \
  --offset 30d \
  --output jsonc \
  > "$EVIDENCE_PATH/vault-activity-log-30d.jsonc"

az monitor activity-log list \
  --resource-group "$TARGET_RG" \
  --offset 30d \
  --output jsonc \
  > "$EVIDENCE_PATH/target-rg-activity-log-30d.jsonc"

echo "Evidence exported to $EVIDENCE_PATH"
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Verification_Commands

```bash
# Provider verification.

az provider show \
  --namespace Microsoft.RecoveryServices \
  --query registrationState \
  --output tsv

az provider show \
  --namespace Microsoft.Compute \
  --query registrationState \
  --output tsv

az provider show \
  --namespace Microsoft.Network \
  --query registrationState \
  --output tsv

# Vault verification.

az backup vault show \
  --resource-group "<vault-resource-group>" \
  --name "<recovery-services-vault-name>" \
  --output table

# Source VM verification.

az vm show \
  --resource-group "<source-vm-resource-group>" \
  --name "<source-vm-name>" \
  --output table

az vm get-instance-view \
  --resource-group "<source-vm-resource-group>" \
  --name "<source-vm-name>" \
  --query "instanceView.statuses" \
  --output table

# Target network verification.

az network vnet show \
  --resource-group "<target-resource-group>" \
  --name "<target-vnet-name>" \
  --output table

az network vnet show \
  --resource-group "<target-resource-group>" \
  --name "<test-failover-vnet-name>" \
  --output table

# Target region resource verification after test failover or production failover.

az resource list \
  --resource-group "<target-resource-group>" \
  --output table

az vm list \
  --resource-group "<target-resource-group>" \
  --output table

az network nic list \
  --resource-group "<target-resource-group>" \
  --output table

az disk list \
  --resource-group "<target-resource-group>" \
  --output table

# Activity Log verification.

az monitor activity-log list \
  --resource-group "<vault-resource-group>" \
  --offset 7d \
  --output table

az monitor activity-log list \
  --resource-group "<target-resource-group>" \
  --offset 7d \
  --output table
```

```kusto
// Log Analytics verification.

AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| summarize Count=count(), LastSeen=max(TimeGenerated) by OperationNameValue, ActivityStatusValue, Caller
| order by LastSeen desc

AzureActivity
| where TimeGenerated > ago(30d)
| where OperationNameValue has_any ("failover", "replication", "reprotect", "protected item", "recovery plan")
| project TimeGenerated, ResourceGroup, ResourceProviderValue, Resource, OperationNameValue, ActivityStatusValue, Caller
| order by TimeGenerated desc

AzureDiagnostics
| where TimeGenerated > ago(30d)
| where ResourceProvider =~ "MICROSOFT.RECOVERYSERVICES"
| summarize Count=count(), LastSeen=max(TimeGenerated) by Category, OperationName, ResultType
| order by LastSeen desc
```

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| Replication enabled for wrong VM | Use vault Replicated items to disable replication for that VM after confirming no DR dependency | Removes protection and recovery points for that VM |
| Wrong target network selected | Update replicated item compute and network settings before failover | Wrong target network can break app recovery |
| Wrong test failover network selected | Cleanup test failover, correct network selection, rerun test failover | Test resources may be unreachable or collide with production |
| Test failover created temporary VM resources | Use Cleanup test failover from replicated item | Leaving resources creates cost and confusion |
| Recovery plan order wrong | Edit recovery plan groups and retest | Wrong startup order can break app recovery |
| Manual recovery plan task wrong | Edit or remove task and retest | Operators may follow bad instructions during outage |
| Automation task wrong | Disable automation step or correct runbook | Bad automation can misconfigure target resources |
| Production failover started accidentally | Do not commit until validated; check whether change recovery point or failback path is available | Production risk depends on job state |
| Production failover validated and committed | Reprotect and plan failback | Commit removes ability to change recovery point for that failover |
| Target VM created with wrong size | Resize failed-over VM after validation if supported | Performance or cost impact |
| Target VM created with wrong NIC or subnet | Correct NIC, subnet, NSG, and route table after failover or update ASR settings before retest | Network outage risk |
| DNS cutover wrong | Revert DNS record or traffic manager endpoint according to DNS TTL plan | Client connectivity impact |
| Reprotect started with wrong settings | Review job state, correct replication settings where possible, or stop and reconfigure under Microsoft guidance | Can complicate failback |
| Evidence folder created locally | Delete evidence folder | Removes local evidence only |

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Failure_Checks

| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Enable replication fails | VM supportability, provider, RBAC, region, quota, or network issue | Check VM Disaster recovery prerequisites page | Recreating vault immediately |
| VM never reaches protected state | Initial replication, Mobility service, cache storage, or outbound connectivity | Replicated item health and ASR jobs | Running failover |
| Replication health unhealthy | Agent, disk, cache account, service endpoint, or high churn issue | Replicated item health details | Disabling replication |
| No recovery points available | Initial replication incomplete or replication broken | Recovery point list and ASR jobs | Starting failover |
| Latest app-consistent point missing | App-consistent snapshots disabled or failed | Replication policy and application consistency settings | Assuming crash-consistent points are unusable |
| Test failover fails | Target quota, network, SKU, disk, or ASR job error | Vault > Site Recovery jobs error details | Running production failover |
| Test VM boots but cannot connect | Test VNet, NSG, route table, DNS, public IP, or Bastion issue | NIC effective routes and NSG rules | Cleaning up before evidence |
| Test VM connects but app fails | Dependency order, database, identity, firewall, config, or service startup | App logs and dependency checks | Blaming ASR replication only |
| Cleanup test failover fails | Locked resource, permission issue, or pending job | Cleanup job details and target resource group | Manually deleting random resources first |
| Recovery plan starts VMs in wrong order | Groups configured incorrectly | Recovery plan customization | Editing individual VM settings only |
| Recovery plan automation fails | Runbook identity, permissions, script bug, or parameter issue | Automation job output | Rerunning full failover blindly |
| Production failover option unavailable | Replication unhealthy or no recovery points | Replicated item overview | Committing anything |
| Planned shutdown fails | Source VM unreachable or guest issue | Job details and VM state | Assuming failover failed |
| Failover completes but target VM missing | Wrong target resource group or portal filter | Target RG resources and ASR job details | Starting another failover |
| Commit unavailable | Failover not complete or job still running | ASR job status | Reprotecting first |
| Commit done too early | Target app not validated | App validation notes and job history | Changing recovery point after commit |
| Reprotect fails | Target to source access, quota, permissions, disks, or network issue | Reprotect job error details | Running failback |
| Activity logs missing ASR detail | Diagnostic settings not routed or wrong query scope | Vault diagnostics and AzureActivity scope | Recreating alerts |
| ASR alerts not firing | Alert rule, action group, or vault monitoring issue | Alert rules and action group test | Turning off ASR |
| RTO evidence missing | No timestamps captured | Export ASR jobs and operator notes | Guessing recovery time later |

# Configure_Azure_Site_Recovery_Failover_And_Test_Failover_Related_Labs

| Lab                                 | Related Workbook                                                                   | Skill Proven                  |
| ----------------------------------- | ---------------------------------------------------------------------------------- | ----------------------------- |
| Enable Azure VM replication         | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | ASR protection setup          |
| Confirm replicated item health      | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Replication health validation |
| Build test failover network         | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Safe DR drill networking      |
| Run test failover                   | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Non-disruptive DR validation  |
| Cleanup test failover               | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Drill cleanup discipline      |
| Build recovery plan                 | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Ordered app recovery          |
| Add manual recovery plan action     | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Operator runbook integration  |
| Add automation recovery plan task   | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Recovery automation           |
| Run production failover simulation  | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Controlled DR execution       |
| Commit and reprotect                | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Post-failover protection      |
| Configure backup and restore        | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Backup recovery foundation    |
| Configure backup reports and alerts | `06_Configure_Backup_Reports_Alerts_And_Recovery_Validation.md`                    | Recovery monitoring           |
| Troubleshoot ASR failover           | `07_Troubleshoot_Azure_Monitor_Backup_Restore_ASR_And_Alerting_Issues.md`          | DR troubleshooting            |