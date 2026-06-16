04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Index

04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Source_Basis
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Mental_Model
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Planning_Table
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Configuration_Checklist
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Recovery_Services_Vault_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Backup_Vault_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_VM_Backup_Policy_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Enable_VM_Backup_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_On_Demand_Backup_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Recovery_Point_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Restore_Disks_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_File_Recovery_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Restore_Validation_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_KQL_Query_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Evidence_Export_Skeleton
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Verification_Commands
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Rollback
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Failure_Checks
Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Related_Labs

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | What is Azure Backup | Azure Backup purpose, protected workload types, security, monitoring, retention, storage redundancy |
| Microsoft Learn | Recovery Services vault overview | Recovery Services vault model, backup data, recovery points, backup policies, RBAC, soft delete, cross-region restore |
| Microsoft Learn | Backup vault overview | Backup vault model for newer Azure Data Protection workloads, RBAC, soft delete, storage settings |
| Microsoft Learn | Back up an Azure VM from VM settings | Azure VM backup workflow, VM agent, backup extension, policy subtype, recovery point behavior |
| Microsoft Learn | Restore Azure VM data in Azure portal | Create new VM, restore disk, replace existing, cross-region restore options |
| Microsoft Learn | Azure CLI az backup vault | Recovery Services vault creation, storage redundancy, immutability, public network access, job failure alerts |
| Microsoft Learn | Azure CLI az backup policy | Default VM policy, custom backup policy creation, policy listing, policy association |
| Microsoft Learn | Azure CLI az backup protection | Enable protection, check VM protection, backup now, disable protection, resume protection |
| Microsoft Learn | Azure CLI az backup restore | File recovery, restore Azure Files, restore Azure workloads, restore VM disks |
| Microsoft Learn | Azure CLI az dataprotection backup-vault | Backup vault creation for newer Data Protection workloads |

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Azure Backup | Azure recovery service used to protect supported Azure and hybrid workloads |
| Recovery Services vault | Classic Azure Backup vault used for Azure VM backup, Azure Files backup, MARS, MABS, DPM, SQL in Azure VM, SAP HANA in Azure VM, and Site Recovery scenarios |
| Backup vault | Azure Data Protection vault used for newer workloads such as Azure Blob vaulted backup, PostgreSQL, disks, AKS, and future workload types |
| Backup policy | Schedule and retention definition that controls when backups run and how long recovery points are retained |
| Policy subtype | Azure VM backup policy type such as Standard or Enhanced |
| Recovery point | Restorable backup instance created by scheduled or on-demand backup |
| Vault tier | Recovery point data stored in the vault for longer retention and isolated recovery |
| Snapshot tier | Faster short-term restore point behavior for supported Azure VM backups |
| Soft delete | Protection feature that retains deleted backup data for a recovery window after protection or backup data deletion |
| Enhanced soft delete | Stronger soft delete mode that can be made always-on to resist disabling |
| Immutability | Vault setting that helps prevent backup data from being modified or deleted during retention |
| Cross Region Restore | Restore option that uses replicated backup data in the paired secondary region when configured |
| Cross Subscription Restore | Restore option allowing restore into another subscription when allowed by vault settings and permissions |
| Backup extension | VM extension installed by Azure Backup through the VM agent to protect Azure VMs |
| On-demand backup | Manual backup job started outside the normal policy schedule |
| Restore disks | Restore mode that creates disks from a recovery point so a VM can be rebuilt or data can be recovered |
| File recovery | Restore method that mounts a recovery point and allows individual files to be copied out |
| First rule | A backup is not proven until restore is tested |
| Blunt rule | Creating a vault is not backup. Enabling protection and validating recovery points is backup |

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Subscription ID | `11111111-1111-1111-1111-111111111111` | `<subscription-id>` |
| Location | `eastus` | `<location>` |
| Vault resource group | `rg-recovery-core-01` | `<vault-resource-group>` |
| Recovery Services vault name | `rsv-core-01` | `<recovery-services-vault-name>` |
| Backup vault name | `bv-core-01` | `<backup-vault-name>` |
| Vault redundancy | `GeoRedundant` | `<LocallyRedundant / ZoneRedundant / GeoRedundant>` |
| Cross Region Restore | `Enabled` | `<Enabled / Disabled>` |
| Cross Subscription Restore | `Enable` | `<Enable / Disable / PermanentlyDisable>` |
| Immutability state | `Unlocked` | `<Disabled / Unlocked / Locked>` |
| Soft delete state | `On` | `<On / AlwaysOn>` |
| Job failure alerts | `Enable` | `<Enable / Disable>` |
| Target VM resource group | `rg-compute-01` | `<vm-resource-group>` |
| Target VM name | `vm-app-01` | `<vm-name>` |
| Target VM resource ID | `/subscriptions/<id>/resourceGroups/rg-compute-01/providers/Microsoft.Compute/virtualMachines/vm-app-01` | `<vm-resource-id>` |
| Backup policy name | `bp-vm-daily-30d` | `<backup-policy-name>` |
| Backup policy subtype | `Standard` | `<Standard / Enhanced>` |
| Daily backup time | `23:00` | `<backup-time>` |
| Retention days | `30` | `<retention-days>` |
| On-demand backup retain until | `31-12-2026` | `<retain-until-dd-mm-yyyy>` |
| Restore test resource group | `rg-restore-test-01` | `<restore-test-resource-group>` |
| Restore staging storage account | `strestoretest01` | `<restore-staging-storage-account>` |
| Restore target VM name | `vm-app-01-restoretest` | `<restore-test-vm-name>` |
| Log Analytics workspace | `law-monitor-core-01` | `<workspace-name>` |
| Evidence path | `C:\AzureBackup-Restore` | `<evidence-path>` |
| Next workbook | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md` | `<next-task>` |

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Configuration_Checklist

| Step | Task | Device | PowerShell / Azure CLI / KQL | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Azure | Admin Workstation / Cloud Shell | `az login` | Azure CLI session is authenticated |
| 2 | Select subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-id>"` | Correct subscription is active |
| 3 | Register Recovery Services provider | Admin Workstation / Cloud Shell | `az provider register --namespace Microsoft.RecoveryServices` | Recovery Services provider is registered |
| 4 | Register Data Protection provider | Admin Workstation / Cloud Shell | `az provider register --namespace Microsoft.DataProtection` | Data Protection provider is registered |
| 5 | Register Compute provider | Admin Workstation / Cloud Shell | `az provider register --namespace Microsoft.Compute` | Compute provider is registered |
| 6 | Create vault resource group | Admin Workstation / Cloud Shell | `az group create --name "<vault-resource-group>" --location "<location>"` | Vault resource group exists |
| 7 | Create Recovery Services vault | Admin Workstation / Cloud Shell | Run Recovery Services vault skeleton | Recovery Services vault exists |
| 8 | Configure vault redundancy | Admin Workstation / Cloud Shell | `az backup vault update --name "<recovery-services-vault-name>" --resource-group "<vault-resource-group>" --backup-storage-redundancy "<vault-redundancy>"` | Vault storage redundancy is configured |
| 9 | Configure job failure alerts | Admin Workstation / Cloud Shell | `az backup vault update --name "<recovery-services-vault-name>" --resource-group "<vault-resource-group>" --job-failure-alerts Enable` | Built-in job failure alerts are enabled |
| 10 | Configure soft delete and immutability stance | Azure Portal / CLI | Review vault Properties and Security settings | Backup deletion protection posture is documented |
| 11 | Create Backup vault for Data Protection workloads | Admin Workstation / Cloud Shell | Run Backup vault skeleton | Backup vault exists for newer workload support |
| 12 | Confirm target VM exists | Admin Workstation / Cloud Shell | `az vm show --resource-group "<vm-resource-group>" --name "<vm-name>" --output table` | VM exists |
| 13 | Confirm VM agent status | Admin Workstation / Cloud Shell | `az vm get-instance-view --resource-group "<vm-resource-group>" --name "<vm-name>" --query "instanceView.vmAgent"` | VM agent status is visible |
| 14 | Check existing VM backup protection | Admin Workstation / Cloud Shell | `az backup protection check-vm --vm "<vm-resource-id>"` | Existing protection state is known |
| 15 | Get default VM policy | Admin Workstation / Cloud Shell | `az backup policy get-default-for-vm --resource-group "<vault-resource-group>" --vault-name "<recovery-services-vault-name>"` | Default VM policy JSON is returned |
| 16 | Create or confirm backup policy | Admin Workstation / Cloud Shell | Run VM backup policy skeleton | VM backup policy exists |
| 17 | Enable backup protection for VM | Admin Workstation / Cloud Shell | Run Enable VM backup skeleton | VM is protected by selected policy |
| 18 | Confirm protected items | Admin Workstation / Cloud Shell | `az backup item list --resource-group "<vault-resource-group>" --vault-name "<recovery-services-vault-name>" --backup-management-type AzureIaasVM --workload-type VM --output table` | Protected VM item appears |
| 19 | Trigger on-demand backup | Admin Workstation / Cloud Shell | Run on-demand backup skeleton | Backup job starts |
| 20 | Monitor backup jobs | Admin Workstation / Cloud Shell | `az backup job list --resource-group "<vault-resource-group>" --vault-name "<recovery-services-vault-name>" --output table` | Backup job status is visible |
| 21 | List recovery points | Admin Workstation / Cloud Shell | Run recovery point skeleton | Recovery points are listed |
| 22 | Open vault Backup items view | Azure Portal | Recovery Services vault > Backup items | Protected items and latest recovery point are visible |
| 23 | Open vault Backup jobs view | Azure Portal | Recovery Services vault > Backup jobs | Completed and failed jobs are visible |
| 24 | Open vault Alerts view | Azure Portal | Recovery Services vault > Monitoring > Alerts | Backup alert posture is visible |
| 25 | Run restore test to disk | Admin Workstation / Cloud Shell | Run restore disks skeleton | Restore job starts and disks are restored to test resource group |
| 26 | Validate restored disk or rebuilt VM | Admin Workstation / Azure Portal | Review restored disks or create test VM | Restore output is usable |
| 27 | Run file recovery test if needed | Azure Portal / Admin Workstation | Run file recovery skeleton | Individual file restore method is validated |
| 28 | Query backup telemetry | Log Analytics | Run KQL query skeleton | Backup jobs and alerts are queryable if diagnostics are routed |
| 29 | Export evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Vault, policy, item, job, recovery point, and restore evidence is saved |
| 30 | Document restore readiness | Operator | Record protected item, recovery point, restore method, RTO notes, and validation result | Backup and restore baseline is documented |

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Recovery_Services_Vault_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: create a Recovery Services vault and configure backup security posture.

SUBSCRIPTION_ID="<subscription-id>"
LOCATION="<location>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
VAULT_REDUNDANCY="GeoRedundant"
CROSS_REGION_RESTORE="Enabled"
CROSS_SUBSCRIPTION_RESTORE="Enable"
IMMUTABILITY_STATE="Unlocked"
JOB_FAILURE_ALERTS="Enable"

az login

az account set \
  --subscription "$SUBSCRIPTION_ID"

az provider register \
  --namespace Microsoft.RecoveryServices

az group create \
  --name "$VAULT_RG" \
  --location "$LOCATION"

az backup vault create \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --location "$LOCATION"

az backup vault update \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --backup-storage-redundancy "$VAULT_REDUNDANCY" \
  --cross-region-restore-flag "$CROSS_REGION_RESTORE" \
  --cross-subscription-restore-state "$CROSS_SUBSCRIPTION_RESTORE" \
  --immutability-state "$IMMUTABILITY_STATE" \
  --job-failure-alerts "$JOB_FAILURE_ALERTS"

az backup vault show \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --output jsonc

az backup vault backup-properties show \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --output jsonc
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Backup_Vault_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: create a Backup vault for newer Azure Data Protection workloads.
# This does not replace the Recovery Services vault for Azure VM backup in this workbook.

SUBSCRIPTION_ID="<subscription-id>"
LOCATION="<location>"
VAULT_RG="<vault-resource-group>"
BACKUP_VAULT_NAME="<backup-vault-name>"
BACKUP_VAULT_REDUNDANCY="LocallyRedundant"
SOFT_DELETE_STATE="On"
SOFT_DELETE_RETENTION_DAYS="14"
JOB_FAILURE_ALERTS="Enabled"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az provider register \
  --namespace Microsoft.DataProtection

az extension add \
  --name dataprotection \
  --upgrade

az dataprotection backup-vault create \
  --resource-group "$VAULT_RG" \
  --vault-name "$BACKUP_VAULT_NAME" \
  --location "$LOCATION" \
  --storage-setting "[{type:'$BACKUP_VAULT_REDUNDANCY',datastore-type:'VaultStore'}]" \
  --soft-delete-state "$SOFT_DELETE_STATE" \
  --retention-duration-in-days "$SOFT_DELETE_RETENTION_DAYS" \
  --azure-monitor-alerts-for-job-failures "$JOB_FAILURE_ALERTS"

az dataprotection backup-vault show \
  --resource-group "$VAULT_RG" \
  --vault-name "$BACKUP_VAULT_NAME" \
  --output jsonc
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_VM_Backup_Policy_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: create a custom Azure VM backup policy from the default VM policy template.
# For simple labs, the default policy can be used directly. For portfolio evidence, export and save the policy JSON.

SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
POLICY_NAME="<backup-policy-name>"
POLICY_FILE="./vm-backup-policy.json"

az account set \
  --subscription "$SUBSCRIPTION_ID"

# Export the default policy.
az backup policy get-default-for-vm \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --output json \
  > "$POLICY_FILE"

# Edit this file before creation if you need custom schedule or retention.
# Common fields to review:
# - name
# - schedulePolicy.scheduleRunTimes
# - retentionPolicy.dailySchedule.retentionDuration.count
# - policyType or policySubType when present

echo "Policy file exported to $POLICY_FILE"
echo "Edit the policy JSON, then create the custom policy."

# Create the policy from JSON.
az backup policy create \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --name "$POLICY_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --policy @"$POLICY_FILE"

az backup policy show \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --name "$POLICY_NAME" \
  --output jsonc

az backup policy list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --output table
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Enable_VM_Backup_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: enable Recovery Services vault backup protection for an Azure VM.

SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
VM_RG="<vm-resource-group>"
VM_NAME="<vm-name>"
POLICY_NAME="<backup-policy-name>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

VM_ID=$(az vm show \
  --resource-group "$VM_RG" \
  --name "$VM_NAME" \
  --query id \
  --output tsv)

echo "VM ID: $VM_ID"

# Check current protection state.
az backup protection check-vm \
  --vm "$VM_ID" \
  --output jsonc

# Enable backup protection.
az backup protection enable-for-vm \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --policy-name "$POLICY_NAME" \
  --vm "$VM_ID"

# Confirm protected item appears.
az backup item list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --output table
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_On_Demand_Backup_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: trigger an on-demand backup and monitor the backup job.
# Retain-until uses day-month-year format.

SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
VM_NAME="<vm-name>"
RETAIN_UNTIL="<retain-until-dd-mm-yyyy>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

CONTAINER_NAME=$(az backup container list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

ITEM_NAME=$(az backup item list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

echo "Container: $CONTAINER_NAME"
echo "Item: $ITEM_NAME"

az backup protection backup-now \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --container-name "$CONTAINER_NAME" \
  --item-name "$ITEM_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --retain-until "$RETAIN_UNTIL"

az backup job list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --output table
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Recovery_Point_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: list recovery points for a protected Azure VM.

SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
VM_NAME="<vm-name>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

CONTAINER_NAME=$(az backup container list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

ITEM_NAME=$(az backup item list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

az backup recoverypoint list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --container-name "$CONTAINER_NAME" \
  --item-name "$ITEM_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --output table

RP_NAME=$(az backup recoverypoint list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --container-name "$CONTAINER_NAME" \
  --item-name "$ITEM_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --query "[0].name" \
  --output tsv)

echo "Selected recovery point: $RP_NAME"
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Restore_Disks_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: restore disks from a VM recovery point into a restore test resource group.
# This validates recovery without overwriting production.

SUBSCRIPTION_ID="<subscription-id>"
LOCATION="<location>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
VM_NAME="<vm-name>"
RESTORE_RG="<restore-test-resource-group>"
STAGING_STORAGE_RG="<restore-test-resource-group>"
STAGING_STORAGE_ACCOUNT="<restore-staging-storage-account>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az group create \
  --name "$RESTORE_RG" \
  --location "$LOCATION"

# Create staging storage account if needed.
az storage account create \
  --resource-group "$STAGING_STORAGE_RG" \
  --name "$STAGING_STORAGE_ACCOUNT" \
  --location "$LOCATION" \
  --sku Standard_LRS \
  --kind StorageV2

CONTAINER_NAME=$(az backup container list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

ITEM_NAME=$(az backup item list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

RP_NAME=$(az backup recoverypoint list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --container-name "$CONTAINER_NAME" \
  --item-name "$ITEM_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --query "[0].name" \
  --output tsv)

echo "Restoring recovery point $RP_NAME"

az backup restore restore-disks \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --container-name "$CONTAINER_NAME" \
  --item-name "$ITEM_NAME" \
  --rp-name "$RP_NAME" \
  --storage-account "$STAGING_STORAGE_ACCOUNT" \
  --storage-account-resource-group "$STAGING_STORAGE_RG" \
  --target-resource-group "$RESTORE_RG" \
  --restore-mode AlternateLocation

az backup job list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --output table

az disk list \
  --resource-group "$RESTORE_RG" \
  --output table
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_File_Recovery_Skeleton

```bash
# Run from Azure portal and target VM or recovery workstation.
# Purpose: validate individual file recovery from a VM recovery point.
# File recovery usually downloads a script from the vault and mounts the recovery point to copy files.

# Portal path:
# 1. Open Azure portal.
# 2. Go to Recovery Services vault.
# 3. Select Backup items.
# 4. Select Azure Virtual Machine.
# 5. Open the protected VM.
# 6. Select File Recovery.
# 7. Choose a recovery point.
# 8. Download the generated script.
# 9. Run the script on a compatible machine with required permissions.
# 10. Browse mounted volumes from the recovery point.
# 11. Copy a test file to a safe restore location.
# 12. Return to the portal and unmount disks after recovery.
# 13. Document:
#     - Recovery point time
#     - File path restored
#     - Destination path
#     - Validation result
#     - Operator
#     - Date and time
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Restore_Validation_Skeleton

```bash
# Run after restore disks completes.
# Purpose: validate that restore output exists and can be used.

SUBSCRIPTION_ID="<subscription-id>"
RESTORE_RG="<restore-test-resource-group>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az disk list \
  --resource-group "$RESTORE_RG" \
  --output table

az resource list \
  --resource-group "$RESTORE_RG" \
  --output table

# Portal validation path:
# 1. Open the restore test resource group.
# 2. Confirm restored disks exist.
# 3. Confirm disk sizes and names match expected VM layout.
# 4. If a restore template was generated, download and inspect the template.
# 5. Create a test VM only if the restore test plan requires boot validation.
# 6. Keep the test VM isolated from production networks.
# 7. Confirm OS boots, application files exist, and data integrity checks pass.
# 8. Record RTO evidence:
#    - Restore start time
#    - Restore completion time
#    - Validation time
#    - Restored resource names
#    - Recovery point timestamp
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_KQL_Query_Skeleton

```kusto
// Purpose: query backup telemetry after diagnostic settings route Recovery Services vault logs to Log Analytics.
// Run inside the Log Analytics workspace used for recovery monitoring.

//
// 1. Recovery Services vault diagnostic records if routed to AzureDiagnostics.
//
AzureDiagnostics
| where TimeGenerated > ago(7d)
| where ResourceProvider =~ "MICROSOFT.RECOVERYSERVICES"
| summarize Count=count(), LastRecord=max(TimeGenerated) by Resource, Category, OperationName
| order by LastRecord desc

//
// 2. Backup job failures if the vault emits them to Log Analytics.
//
AzureDiagnostics
| where TimeGenerated > ago(30d)
| where ResourceProvider =~ "MICROSOFT.RECOVERYSERVICES"
| where Level in ("Error", "Critical") or ResultType has_any ("Failed", "Failure", "Error")
| project TimeGenerated, Resource, Category, OperationName, Level, ResultType, ResultDescription
| order by TimeGenerated desc

//
// 3. Activity Log operations against Recovery Services vaults.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Caller, CorrelationId
| order by TimeGenerated desc

//
// 4. Backup protection changes.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| where OperationNameValue has_any ("backup", "protection", "vault", "restore", "delete")
| summarize Count=count(), LastSeen=max(TimeGenerated) by OperationNameValue, ActivityStatusValue, Caller
| order by LastSeen desc

//
// 5. Vault delete or destructive action watch.
//
AzureActivity
| where TimeGenerated > ago(90d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| where OperationNameValue has_any ("delete", "Disable", "Stop", "Purge", "Update")
| project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Caller
| order by TimeGenerated desc
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Evidence_Export_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: export evidence for vault, policy, backup protection, recovery points, jobs, and restore output.

EVIDENCE_PATH="./AzureBackup-Restore"
SUBSCRIPTION_ID="<subscription-id>"
VAULT_RG="<vault-resource-group>"
RSV_NAME="<recovery-services-vault-name>"
BACKUP_VAULT_NAME="<backup-vault-name>"
VM_NAME="<vm-name>"
POLICY_NAME="<backup-policy-name>"
RESTORE_RG="<restore-test-resource-group>"

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

az backup vault backup-properties show \
  --resource-group "$VAULT_RG" \
  --name "$RSV_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/recovery-services-vault-backup-properties.jsonc"

az dataprotection backup-vault show \
  --resource-group "$VAULT_RG" \
  --vault-name "$BACKUP_VAULT_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/backup-vault.jsonc"

az backup policy show \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --name "$POLICY_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/vm-backup-policy.jsonc"

az backup item list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --output jsonc \
  > "$EVIDENCE_PATH/protected-items.jsonc"

az backup job list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/backup-jobs.jsonc"

CONTAINER_NAME=$(az backup container list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

ITEM_NAME=$(az backup item list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --query "[?properties.friendlyName=='$VM_NAME'].name | [0]" \
  --output tsv)

az backup recoverypoint list \
  --resource-group "$VAULT_RG" \
  --vault-name "$RSV_NAME" \
  --container-name "$CONTAINER_NAME" \
  --item-name "$ITEM_NAME" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --output jsonc \
  > "$EVIDENCE_PATH/recovery-points.jsonc"

az resource list \
  --resource-group "$RESTORE_RG" \
  --output jsonc \
  > "$EVIDENCE_PATH/restore-test-resource-group-resources.jsonc"

az disk list \
  --resource-group "$RESTORE_RG" \
  --output jsonc \
  > "$EVIDENCE_PATH/restored-disks.jsonc"

echo "Evidence exported to $EVIDENCE_PATH"
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Verification_Commands

```bash
# Provider verification.

az provider show \
  --namespace Microsoft.RecoveryServices \
  --query registrationState \
  --output tsv

az provider show \
  --namespace Microsoft.DataProtection \
  --query registrationState \
  --output tsv

# Recovery Services vault verification.

az backup vault list \
  --resource-group "<vault-resource-group>" \
  --output table

az backup vault show \
  --resource-group "<vault-resource-group>" \
  --name "<recovery-services-vault-name>" \
  --output table

az backup vault backup-properties show \
  --resource-group "<vault-resource-group>" \
  --name "<recovery-services-vault-name>" \
  --output jsonc

# Backup vault verification.

az dataprotection backup-vault list \
  --resource-group "<vault-resource-group>" \
  --output table

az dataprotection backup-vault show \
  --resource-group "<vault-resource-group>" \
  --vault-name "<backup-vault-name>" \
  --output jsonc

# Policy verification.

az backup policy list \
  --resource-group "<vault-resource-group>" \
  --vault-name "<recovery-services-vault-name>" \
  --output table

az backup policy show \
  --resource-group "<vault-resource-group>" \
  --vault-name "<recovery-services-vault-name>" \
  --name "<backup-policy-name>" \
  --output jsonc

# VM protection verification.

az backup protection check-vm \
  --vm "<vm-resource-id>" \
  --output jsonc

az backup item list \
  --resource-group "<vault-resource-group>" \
  --vault-name "<recovery-services-vault-name>" \
  --backup-management-type AzureIaasVM \
  --workload-type VM \
  --output table

# Job verification.

az backup job list \
  --resource-group "<vault-resource-group>" \
  --vault-name "<recovery-services-vault-name>" \
  --output table

# Restore output verification.

az disk list \
  --resource-group "<restore-test-resource-group>" \
  --output table

az resource list \
  --resource-group "<restore-test-resource-group>" \
  --output table
```

```kusto
// Log Analytics verification, if vault diagnostic settings are routed.

AzureActivity
| where TimeGenerated > ago(30d)
| where ResourceProviderValue =~ "MICROSOFT.RECOVERYSERVICES"
| summarize Count=count(), LastSeen=max(TimeGenerated) by OperationNameValue, ActivityStatusValue
| order by LastSeen desc

AzureDiagnostics
| where TimeGenerated > ago(30d)
| where ResourceProvider =~ "MICROSOFT.RECOVERYSERVICES"
| summarize Count=count(), LastSeen=max(TimeGenerated) by Category, OperationName, ResultType
| order by LastSeen desc
```

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| Recovery Services vault created in wrong region | Delete vault only after removing protected items, jobs, and soft-deleted items | Vault deletion can be blocked by protected data |
| Backup vault created in wrong region | `az dataprotection backup-vault delete --resource-group "<vault-resource-group>" --vault-name "<backup-vault-name>" --yes` | Deletes Backup vault if no protected data blocks deletion |
| Vault redundancy set incorrectly | Change redundancy only before protection when supported | Existing backup storage behavior may not be reversible |
| Cross Region Restore enabled by mistake | Review whether setting can be disabled for selected vault type | Some settings may be one-way depending on vault type |
| Immutability set to Locked | Cannot reverse once locked | Treat as permanent security posture |
| Wrong backup policy created | `az backup policy delete --resource-group "<vault-resource-group>" --vault-name "<recovery-services-vault-name>" --name "<backup-policy-name>"` | Only works if no items are associated |
| VM protected with wrong policy | Reconfigure or resume protection with correct policy | Backup schedule and retention may change |
| VM protected in wrong vault | Stop protection with retain data, then reconfigure to correct vault if required | Can create operational confusion and restore delays |
| On-demand backup created | No rollback needed | Backup point remains according to retention |
| Restore disks created in test RG | Delete restored disks and test resources | Deletes restore test artifacts only |
| Test VM created from restored disks | Delete test VM, NICs, public IPs, and disks after validation | Prevents duplicate workload or cost |
| File recovery script downloaded | Delete script after use and unmount recovery point | Avoids stale recovery access |
| Evidence exported locally | Delete evidence folder | Removes local evidence only |

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Failure_Checks

| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Vault creation fails | Provider, RBAC, policy, or region issue | Check provider registration and Azure Policy deny assignments | Rebuilding the subscription |
| Backup vault command not found | Missing dataprotection CLI extension | `az extension add --name dataprotection --upgrade` | Switching to unrelated commands |
| VM backup enable fails | VM agent missing, unsupported VM, RBAC, or vault region mismatch | Check VM agent and backup support | Deleting the VM |
| `check-vm` returns existing vault | VM already protected | Confirm vault details and owner | Enabling duplicate protection |
| Backup policy create fails | Invalid edited policy JSON | Validate JSON and start from default policy again | Editing vault properties |
| Policy delete fails | Protected items are associated | List associated items | Force deleting vault |
| On-demand backup fails | Protected item not found or wrong container/item name | List containers and items | Recreating policy |
| Backup job stays in progress | Large VM or snapshot delay | Check backup job details | Canceling early |
| Backup job fails after snapshot | VM extension or guest state issue | Review job error and VM extension health | Recreating vault |
| Recovery points missing | Backup has not completed or wrong item selected | Check backup jobs and item name | Running restore blindly |
| Restore disks fails | Staging storage, target RG, RBAC, region, or unsupported option | Check restore job details and staging storage account | Deleting recovery points |
| Restore to existing VM unavailable | VM deleted or unsupported configuration | Use restore disks or create new VM option | Recreating source VM first |
| File recovery script fails | OS compatibility, permissions, networking, or antivirus | Run script as admin and confirm requirements | Re-running backup |
| Restored VM boots with network conflict | Test VM connected to production network | Isolate restore test subnet | Replacing production VM |
| Cross Region Restore unavailable | Vault not configured for GRS and CRR | Check vault redundancy and CRR setting | Assuming backup data is lost |
| Soft delete blocks cleanup | Deleted backup item retained by soft delete | Review soft-deleted containers/items | Force deleting resource group |
| Vault deletion blocked | Protected items, soft-deleted data, private endpoints, or locks | Remove protection and clean soft-deleted items | Deleting resource group first |
| Backup telemetry missing in Log Analytics | Diagnostic settings not configured | Check vault diagnostic settings | Recreating backup job |
| Alerts not firing for backup failures | Built-in alerts or action rules not configured | Check vault monitoring and action groups | Turning off Backup |
| Restore test lacks evidence | No exported job and recovery point data | Run evidence export skeleton | Assuming restore readiness is proven |

# Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore_Related_Labs

| Lab                                 | Related Workbook                                                                   | Skill Proven                            |
| ----------------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------- |
| Create Recovery Services vault      | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Classic Azure Backup vault creation     |
| Configure vault redundancy          | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Backup storage resiliency               |
| Create Backup vault                 | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Newer Azure Data Protection vault model |
| Export default VM backup policy     | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Policy inspection                       |
| Create VM backup policy             | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Schedule and retention control          |
| Enable Azure VM backup              | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Backup protection                       |
| Trigger on-demand backup            | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Manual recovery point creation          |
| List recovery points                | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Restore point validation                |
| Restore VM disks                    | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Restore testing                         |
| Run file recovery test              | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | Granular restore                        |
| Configure ASR failover              | `05_Configure_Azure_Site_Recovery_Failover_And_Test_Failover.md`                   | Disaster recovery                       |
| Configure backup reports and alerts | `06_Configure_Backup_Reports_Alerts_And_Recovery_Validation.md`                    | Backup operations monitoring            |
| Troubleshoot backup and restore     | `07_Troubleshoot_Azure_Monitor_Backup_Restore_ASR_And_Alerting_Issues.md`          | Recovery troubleshooting                |