# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Index
02_Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations.md
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Source_Basis
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Mental_Model
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Planning_Table
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Configuration_Checklist
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Portal_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Azure_CLI_Precheck_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_PowerShell_Precheck_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Resource_Group_Build_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Tag_Governance_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Lock_Management_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Move_Precheck_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Move_Execution_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Post_Move_Validation_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Evidence_Export_Skeleton
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Verification_Commands
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Rollback
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Failure_Checks
Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Related_Labs

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Resource Manager resource groups | Resource group lifecycle, metadata location, deployment scope, and resource organization |
| Microsoft Learn | Manage resource groups in the Azure portal | Creating, listing, opening, deleting, tagging, locking, and moving resource groups |
| Microsoft Learn | Azure Resource Manager tags | Tagging resource groups and resources for organization, cost, ownership, and lifecycle tracking |
| Microsoft Learn | Azure Resource Manager locks | Delete locks, read-only locks, inherited lock behavior, and critical resource protection |
| Microsoft Learn | Move Azure resources to a new resource group or subscription | Move prerequisites, same tenant requirement, resource provider registration, changed resource IDs, and move locks |
| Microsoft Learn | Azure resource move support matrix | Confirming whether resource types support move operations |
| Microsoft Learn | Azure RBAC role assignments | Required permissions for move, lock, tag, and resource group operations |
| Azure CLI | `az group`, `az tag`, `az lock`, `az resource move` | Repeatable resource group, tag, lock, and move operations |
| Azure PowerShell | `Az.Resources` | PowerShell-based resource group, tag, lock, and move validation |

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Resource group | Logical container for related Azure resources |
| Resource group location | Region where resource group metadata is stored, not necessarily where every contained resource runs |
| Resource lifecycle grouping | Best practice of placing resources with the same lifecycle in the same resource group |
| Tag | Key-value metadata used for ownership, cost tracking, environment, application, and lifecycle classification |
| Tag inheritance | Tags do not automatically inherit by default from resource group to child resources unless policy or automation applies them |
| Lock | Management control that blocks delete or write operations at a selected scope |
| Delete lock | Prevents deletion while still allowing normal management changes |
| Read-only lock | Prevents updates and deletion; can break normal control plane operations |
| Scope | Level where the operation applies: subscription, resource group, or resource |
| Resource move | Azure Resource Manager operation that changes resource group or subscription placement |
| Move lock | Temporary lock placed on source and target resource groups while resources are moving |
| Changed resource ID | Resource ID changes after a move because resource group or subscription path changes |
| Orphaned role assignment | Role assignment tied to old scope may not follow a moved resource and may need recreation |
| Provider registration | Destination subscription must be registered for resource providers used by moved resources |
| First rule | Validate locks, provider registration, tenant match, move support, and dependencies before moving anything |
| Blunt rule | Do not move production resources casually; resource IDs, dependencies, RBAC, policy, diagnostics, and automation may break |

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant ID | `aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee` | `<tenant-id>` |
| Source subscription name | `Azure Lab Subscription` | `<source-subscription-name>` |
| Source subscription ID | `11111111-2222-3333-4444-555555555555` | `<source-subscription-id>` |
| Destination subscription name | `Azure Platform Subscription` | `<destination-subscription-name>` |
| Destination subscription ID | `22222222-3333-4444-5555-666666666666` | `<destination-subscription-id>` |
| Resource group location | `eastus` | `<resource-group-location>` |
| Source resource group | `rg-lab-source-001` | `<source-resource-group>` |
| Destination resource group | `rg-lab-destination-001` | `<destination-resource-group>` |
| Resource to move | `vm-lab-001` | `<resource-name>` |
| Resource ID to move | `/subscriptions/<id>/resourceGroups/<rg>/providers/<provider>/<type>/<name>` | `<resource-id>` |
| Environment tag | `Lab` | `<environment-tag>` |
| Owner tag | `CloudOps` | `<owner-tag>` |
| Cost center tag | `IT-1001` | `<cost-center-tag>` |
| Application tag | `GovernanceLab` | `<application-tag>` |
| Lifecycle tag | `Training` | `<lifecycle-tag>` |
| Delete lock name | `lock-rg-delete-protection` | `<delete-lock-name>` |
| Read-only lock name | `lock-rg-readonly-protection` | `<readonly-lock-name>` |
| Move change window | `Lab window` | `<change-window>` |
| Evidence path | `C:\Azure-Governance\02-RG-Tags-Locks-Move` | `<evidence-path>` |
| Rollback target | Original resource group or subscription | `<rollback-target>` |

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Configuration_Checklist

| Step | Task | Device | Portal / Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Azure admin session | Admin Workstation / Cloud Shell | `https://portal.azure.com` | `Connect-AzAccount -Tenant "<tenant-id>"` | Admin is signed in to correct tenant |
| 2 | Confirm active subscription | Cloud Shell | `az account show -o table` | `Get-AzContext` | Correct subscription context is visible |
| 3 | List subscriptions | Cloud Shell | `az account list --all -o table` | `Get-AzSubscription` | Source and destination subscriptions are visible |
| 4 | Confirm same tenant for move | Cloud Shell | `az account show --subscription "<source-subscription-id>" --query tenantId -o tsv` | `(Get-AzSubscription -SubscriptionId "<source-subscription-id>").TenantId` | Source tenant ID is known |
| 5 | Confirm destination tenant | Cloud Shell | `az account show --subscription "<destination-subscription-id>" --query tenantId -o tsv` | `(Get-AzSubscription -SubscriptionId "<destination-subscription-id>").TenantId` | Destination tenant ID matches source if cross-subscription move is planned |
| 6 | Create source resource group if needed | Cloud Shell | `az group create --name "<source-resource-group>" --location "<location>"` | `New-AzResourceGroup -Name "<source-resource-group>" -Location "<location>"` | Source resource group exists |
| 7 | Create destination resource group if needed | Cloud Shell | `az group create --name "<destination-resource-group>" --location "<location>"` | `New-AzResourceGroup -Name "<destination-resource-group>" -Location "<location>"` | Destination resource group exists |
| 8 | Apply baseline tags to source RG | Cloud Shell | `az group update --name "<source-resource-group>" --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training` | `Set-AzResourceGroup -Name "<source-resource-group>" -Tag @{Environment="Lab";Owner="CloudOps";CostCenter="IT-1001";Application="GovernanceLab";Lifecycle="Training"}` | Source RG has standard tags |
| 9 | Apply baseline tags to destination RG | Cloud Shell | `az group update --name "<destination-resource-group>" --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training` | `Set-AzResourceGroup -Name "<destination-resource-group>" -Tag @{Environment="Lab";Owner="CloudOps";CostCenter="IT-1001";Application="GovernanceLab";Lifecycle="Training"}` | Destination RG has standard tags |
| 10 | List resources in source RG | Cloud Shell | `az resource list --resource-group "<source-resource-group>" -o table` | `Get-AzResource -ResourceGroupName "<source-resource-group>"` | Source resources are visible |
| 11 | List resources in destination RG | Cloud Shell | `az resource list --resource-group "<destination-resource-group>" -o table` | `Get-AzResource -ResourceGroupName "<destination-resource-group>"` | Destination state is known |
| 12 | Apply tag to individual resource if needed | Cloud Shell | `az resource tag --ids "<resource-id>" --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training` | `Update-AzTag -ResourceId "<resource-id>" -Tag @{Environment="Lab";Owner="CloudOps";CostCenter="IT-1001"} -Operation Merge` | Resource tag state is standardized |
| 13 | Create delete lock on critical RG | Cloud Shell | `az lock create --name "<delete-lock-name>" --lock-type CanNotDelete --resource-group "<source-resource-group>"` | `New-AzResourceLock -LockName "<delete-lock-name>" -LockLevel CanNotDelete -ResourceGroupName "<source-resource-group>" -Force` | Delete lock exists |
| 14 | Validate locks | Cloud Shell | `az lock list --resource-group "<source-resource-group>" -o table` | `Get-AzResourceLock -ResourceGroupName "<source-resource-group>"` | Lock state is visible |
| 15 | Remove locks before move if required | Cloud Shell | `az lock delete --name "<delete-lock-name>" --resource-group "<source-resource-group>"` | `Remove-AzResourceLock -LockName "<delete-lock-name>" -ResourceGroupName "<source-resource-group>" -Force` | Move blockers are removed |
| 16 | Confirm move support | Admin Workstation | Check resource provider move support matrix | Record resource provider and type | Resource type supports move |
| 17 | Confirm destination provider registration | Cloud Shell | `az provider list --query "[].{Provider:namespace,State:registrationState}" -o table` | `Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace,RegistrationState` | Required providers are registered |
| 18 | Register missing provider if needed | Cloud Shell | `az provider register --namespace "<provider-namespace>"` | `Register-AzResourceProvider -ProviderNamespace "<provider-namespace>"` | Provider registration starts |
| 19 | Confirm source and destination locks clear | Cloud Shell | `az lock list --resource-group "<source-resource-group>" -o table`; `az lock list --resource-group "<destination-resource-group>" -o table` | `Get-AzResourceLock -ResourceGroupName "<source-resource-group>"`; `Get-AzResourceLock -ResourceGroupName "<destination-resource-group>"` | Move-blocking locks are not present |
| 20 | Capture resource IDs before move | Cloud Shell | `az resource list --resource-group "<source-resource-group>" --query "[].id" -o tsv` | `Get-AzResource -ResourceGroupName "<source-resource-group>" | Select-Object ResourceId` | Pre-move IDs are recorded |
| 21 | Move resource to destination RG | Cloud Shell | `az resource move --destination-group "<destination-resource-group>" --ids "<resource-id>"` | `Move-AzResource -DestinationResourceGroupName "<destination-resource-group>" -ResourceId "<resource-id>" -Force` | Resource moves to destination RG |
| 22 | Validate destination resource list | Cloud Shell | `az resource list --resource-group "<destination-resource-group>" -o table` | `Get-AzResource -ResourceGroupName "<destination-resource-group>"` | Moved resource appears in destination RG |
| 23 | Validate source resource list | Cloud Shell | `az resource list --resource-group "<source-resource-group>" -o table` | `Get-AzResource -ResourceGroupName "<source-resource-group>"` | Moved resource no longer appears in source RG |
| 24 | Validate changed resource ID | Cloud Shell | `az resource show --ids "<new-resource-id>"` | `Get-AzResource -ResourceId "<new-resource-id>"` | Resource ID reflects new RG or subscription |
| 25 | Reapply tags if needed | Cloud Shell | `az resource tag --ids "<new-resource-id>" --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training` | `Update-AzTag -ResourceId "<new-resource-id>" -Tag @{Environment="Lab";Owner="CloudOps"} -Operation Merge` | Tag posture is correct after move |
| 26 | Recreate lock if needed | Cloud Shell | `az lock create --name "<delete-lock-name>" --lock-type CanNotDelete --resource-group "<destination-resource-group>"` | `New-AzResourceLock -LockName "<delete-lock-name>" -LockLevel CanNotDelete -ResourceGroupName "<destination-resource-group>" -Force` | Destination RG is protected |
| 27 | Validate role assignments after move | Cloud Shell | `az role assignment list --scope "<new-resource-id>" -o table` | `Get-AzRoleAssignment -Scope "<new-resource-id>"` | Access state is known |
| 28 | Validate policy compliance after move | Cloud Shell | `az policy state list --resource "<new-resource-id>" -o table` | `Get-AzPolicyState -ResourceId "<new-resource-id>"` | Policy impact is visible |
| 29 | Export evidence | Admin Workstation | Run evidence export skeleton | Run evidence export skeleton | Evidence files are saved |
| 30 | Document final state | Operator | Record source RG, destination RG, tags, locks, moved resources, new IDs, and validation results | Same | Workbook record is complete |

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Portal_Skeleton

| Step | Portal Area | Action | Expected Result |
|---:|---|---|---|
| 1 | Azure Portal | Sign in as `<admin-upn>` | Correct tenant session opens |
| 2 | Directories + subscriptions | Confirm selected directory | Portal is scoped to correct tenant |
| 3 | Subscriptions | Select source subscription | Source subscription opens |
| 4 | Resource groups | Select or create `<source-resource-group>` | Source RG exists |
| 5 | Resource groups | Select or create `<destination-resource-group>` | Destination RG exists |
| 6 | Source resource group > Tags | Add standard tags | RG tags are visible |
| 7 | Destination resource group > Tags | Add standard tags | Destination RG tags are visible |
| 8 | Source resource group > Locks | Add delete lock if protection is needed | Delete lock exists |
| 9 | Source resource group > Resources | Review resources planned for move | Resource list is known |
| 10 | Resource > Move | Select **Move to another resource group** or **Move to another subscription** | Move wizard opens |
| 11 | Move wizard | Select destination resource group or subscription | Destination is selected |
| 12 | Move wizard | Validate move | Portal confirms whether resources can move |
| 13 | Move wizard | Acknowledge changed resource IDs | Operator confirms dependency impact |
| 14 | Move wizard | Start move | Resource move begins |
| 15 | Destination resource group | Confirm moved resource appears | Resource exists under destination RG |
| 16 | Source resource group | Confirm moved resource no longer appears | Source RG no longer contains moved resource |
| 17 | Destination resource group > Tags | Confirm tags after move | Tags are correct |
| 18 | Destination resource group > Locks | Add or recreate lock if needed | Destination RG protection is restored |
| 19 | Activity log | Review move events | Move operation evidence is visible |
| 20 | Resource overview | Confirm application or service still works | Resource is operational after move |

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Azure_CLI_Precheck_Skeleton

```bash
# Run in Azure Cloud Shell or an admin workstation with Azure CLI.
# Purpose: capture tenant, subscription, resource group, tag, lock, and move baseline.

TENANT_ID="<tenant-id>"
SOURCE_SUBSCRIPTION_ID="<source-subscription-id>"
DESTINATION_SUBSCRIPTION_ID="<destination-subscription-id>"
SOURCE_RG="<source-resource-group>"
DESTINATION_RG="<destination-resource-group>"
LOCATION="<resource-group-location>"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"

# Confirm subscriptions.
az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o table | tee "$EVIDENCE_PATH/subscription-list.txt"

# Confirm source subscription context.
az account set --subscription "$SOURCE_SUBSCRIPTION_ID"

az account show \
  --query "{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o json | tee "$EVIDENCE_PATH/source-context.json"

# Confirm destination subscription tenant when cross-subscription move is planned.
az account show \
  --subscription "$DESTINATION_SUBSCRIPTION_ID" \
  --query "{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o json | tee "$EVIDENCE_PATH/destination-context.json"

# Capture source resource groups.
az group list \
  --query "[].{Name:name, Location:location, ProvisioningState:properties.provisioningState, Tags:tags}" \
  -o json | tee "$EVIDENCE_PATH/resource-groups-before.json"

# Capture source RG if it exists.
az group show \
  --name "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/source-rg-before.json"

# Capture destination RG if it exists.
az group show \
  --name "$DESTINATION_RG" \
  -o json | tee "$EVIDENCE_PATH/destination-rg-before.json"

# Capture current source RG resources.
az resource list \
  --resource-group "$SOURCE_RG" \
  --query "[].{Name:name, Type:type, Location:location, Id:id, Tags:tags}" \
  -o json | tee "$EVIDENCE_PATH/source-rg-resources-before.json"

# Capture locks.
az lock list \
  --resource-group "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/source-rg-locks-before.json"

az lock list \
  --resource-group "$DESTINATION_RG" \
  -o json | tee "$EVIDENCE_PATH/destination-rg-locks-before.json"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_PowerShell_Precheck_Skeleton

```powershell
# Run in elevated PowerShell or Azure Cloud Shell.
# Purpose: capture tenant, subscription, resource group, tag, lock, and move baseline.

$TenantId = "<tenant-id>"
$SourceSubscriptionId = "<source-subscription-id>"
$DestinationSubscriptionId = "<destination-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$Location = "<resource-group-location>"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Import-Module Az.Accounts
Import-Module Az.Resources

Connect-AzAccount -Tenant $TenantId

# Confirm visible subscriptions.
Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Tee-Object "$EvidencePath\subscription-list.txt"

# Confirm source context.
Set-AzContext -SubscriptionId $SourceSubscriptionId

Get-AzContext |
  Select-Object Name,Subscription,Tenant,Account |
  Tee-Object "$EvidencePath\source-context.txt"

# Confirm source and destination tenant IDs.
(Get-AzSubscription -SubscriptionId $SourceSubscriptionId).TenantId |
  Tee-Object "$EvidencePath\source-subscription-tenant-id.txt"

(Get-AzSubscription -SubscriptionId $DestinationSubscriptionId).TenantId |
  Tee-Object "$EvidencePath\destination-subscription-tenant-id.txt"

# Capture resource group state.
Get-AzResourceGroup |
  Select-Object ResourceGroupName,Location,ProvisioningState,Tags |
  Tee-Object "$EvidencePath\resource-groups-before.txt"

Get-AzResourceGroup -Name $SourceResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\source-rg-before.txt"

Get-AzResourceGroup -Name $DestinationResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\destination-rg-before.txt"

# Capture resource inventory.
Get-AzResource -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  Tee-Object "$EvidencePath\source-rg-resources-before.txt"

# Capture locks.
Get-AzResourceLock -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\source-rg-locks-before.txt"

Get-AzResourceLock -ResourceGroupName $DestinationResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\destination-rg-locks-before.txt"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Resource_Group_Build_Skeleton

```powershell
# Purpose: create source and destination resource groups with baseline tags.

$TenantId = "<tenant-id>"
$SubscriptionId = "<source-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$Location = "<resource-group-location>"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

$Tags = @{
  Environment = "Lab"
  Owner       = "CloudOps"
  CostCenter  = "IT-1001"
  Application = "GovernanceLab"
  Lifecycle   = "Training"
}

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Create source RG if missing.
$SourceRgObject = Get-AzResourceGroup -Name $SourceResourceGroup -ErrorAction SilentlyContinue

if ($SourceRgObject) {
  Write-Output "Source resource group already exists: $SourceResourceGroup"
}
else {
  New-AzResourceGroup `
    -Name $SourceResourceGroup `
    -Location $Location `
    -Tag $Tags
}

# Create destination RG if missing.
$DestinationRgObject = Get-AzResourceGroup -Name $DestinationResourceGroup -ErrorAction SilentlyContinue

if ($DestinationRgObject) {
  Write-Output "Destination resource group already exists: $DestinationResourceGroup"
}
else {
  New-AzResourceGroup `
    -Name $DestinationResourceGroup `
    -Location $Location `
    -Tag $Tags
}

# Confirm resource groups.
Get-AzResourceGroup -Name $SourceResourceGroup |
  Tee-Object "$EvidencePath\source-rg-after-create.txt"

Get-AzResourceGroup -Name $DestinationResourceGroup |
  Tee-Object "$EvidencePath\destination-rg-after-create.txt"
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<source-subscription-id>"
SOURCE_RG="<source-resource-group>"
DESTINATION_RG="<destination-resource-group>"
LOCATION="<resource-group-location>"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

az group create \
  --name "$SOURCE_RG" \
  --location "$LOCATION" \
  --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training

az group create \
  --name "$DESTINATION_RG" \
  --location "$LOCATION" \
  --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training

az group show --name "$SOURCE_RG" -o json | tee "$EVIDENCE_PATH/source-rg-after-create.json"
az group show --name "$DESTINATION_RG" -o json | tee "$EVIDENCE_PATH/destination-rg-after-create.json"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Tag_Governance_Skeleton

```powershell
# Purpose: apply standard tags to resource groups and selected resources.
# Warning: Set-AzResourceGroup -Tag replaces the tag set on the resource group.

$TenantId = "<tenant-id>"
$SubscriptionId = "<source-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$ResourceId = "<resource-id>"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

$StandardTags = @{
  Environment = "Lab"
  Owner       = "CloudOps"
  CostCenter  = "IT-1001"
  Application = "GovernanceLab"
  Lifecycle   = "Training"
}

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Capture current RG tags.
Get-AzResourceGroup -Name $SourceResourceGroup |
  Select-Object ResourceGroupName,Tags |
  Tee-Object "$EvidencePath\source-rg-tags-before.txt"

Get-AzResourceGroup -Name $DestinationResourceGroup |
  Select-Object ResourceGroupName,Tags |
  Tee-Object "$EvidencePath\destination-rg-tags-before.txt"

# Apply standard RG tags.
Set-AzResourceGroup `
  -Name $SourceResourceGroup `
  -Tag $StandardTags

Set-AzResourceGroup `
  -Name $DestinationResourceGroup `
  -Tag $StandardTags

# Merge tags onto an individual resource.
Update-AzTag `
  -ResourceId $ResourceId `
  -Tag $StandardTags `
  -Operation Merge

# Validate tag state.
Get-AzResourceGroup -Name $SourceResourceGroup |
  Select-Object ResourceGroupName,Tags |
  Tee-Object "$EvidencePath\source-rg-tags-after.txt"

Get-AzResourceGroup -Name $DestinationResourceGroup |
  Select-Object ResourceGroupName,Tags |
  Tee-Object "$EvidencePath\destination-rg-tags-after.txt"

Get-AzResource -ResourceId $ResourceId |
  Select-Object Name,ResourceType,ResourceId,Tags |
  Tee-Object "$EvidencePath\resource-tags-after.txt"
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<source-subscription-id>"
SOURCE_RG="<source-resource-group>"
DESTINATION_RG="<destination-resource-group>"
RESOURCE_ID="<resource-id>"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

# Capture current tags.
az group show --name "$SOURCE_RG" --query "{Name:name,Tags:tags}" -o json | tee "$EVIDENCE_PATH/source-rg-tags-before.json"
az group show --name "$DESTINATION_RG" --query "{Name:name,Tags:tags}" -o json | tee "$EVIDENCE_PATH/destination-rg-tags-before.json"

# Apply standard tags to resource groups.
az group update \
  --name "$SOURCE_RG" \
  --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training

az group update \
  --name "$DESTINATION_RG" \
  --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training

# Apply standard tags to selected resource.
az resource tag \
  --ids "$RESOURCE_ID" \
  --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training

# Validate tags.
az group show --name "$SOURCE_RG" --query "{Name:name,Tags:tags}" -o json | tee "$EVIDENCE_PATH/source-rg-tags-after.json"
az group show --name "$DESTINATION_RG" --query "{Name:name,Tags:tags}" -o json | tee "$EVIDENCE_PATH/destination-rg-tags-after.json"
az resource show --ids "$RESOURCE_ID" --query "{Name:name,Type:type,Id:id,Tags:tags}" -o json | tee "$EVIDENCE_PATH/resource-tags-after.json"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Lock_Management_Skeleton

```powershell
# Purpose: create, validate, and remove management locks.
# Use CanNotDelete for normal protection. Use ReadOnly only when you understand the operational impact.

$TenantId = "<tenant-id>"
$SubscriptionId = "<source-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$DeleteLockName = "lock-rg-delete-protection"
$ReadOnlyLockName = "lock-rg-readonly-protection"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Capture current locks.
Get-AzResourceLock -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\source-rg-locks-before.txt"

# Create delete lock.
New-AzResourceLock `
  -LockName $DeleteLockName `
  -LockLevel CanNotDelete `
  -ResourceGroupName $SourceResourceGroup `
  -LockNotes "Prevents accidental deletion of resource group resources." `
  -Force

# Optional: create read-only lock only when required.
# New-AzResourceLock `
#   -LockName $ReadOnlyLockName `
#   -LockLevel ReadOnly `
#   -ResourceGroupName $SourceResourceGroup `
#   -LockNotes "Prevents writes and deletes. Use carefully." `
#   -Force

# Validate locks.
Get-AzResourceLock -ResourceGroupName $SourceResourceGroup |
  Tee-Object "$EvidencePath\source-rg-locks-after-create.txt"

# Remove lock before move if it blocks the operation.
Remove-AzResourceLock `
  -LockName $DeleteLockName `
  -ResourceGroupName $SourceResourceGroup `
  -Force

# Validate removal.
Get-AzResourceLock -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\source-rg-locks-after-remove.txt"
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<source-subscription-id>"
SOURCE_RG="<source-resource-group>"
DELETE_LOCK_NAME="lock-rg-delete-protection"
READONLY_LOCK_NAME="lock-rg-readonly-protection"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

# Capture current locks.
az lock list \
  --resource-group "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/source-rg-locks-before.json"

# Create delete lock.
az lock create \
  --name "$DELETE_LOCK_NAME" \
  --lock-type CanNotDelete \
  --resource-group "$SOURCE_RG" \
  --notes "Prevents accidental deletion of resource group resources."

# Optional: create read-only lock only when required.
# az lock create \
#   --name "$READONLY_LOCK_NAME" \
#   --lock-type ReadOnly \
#   --resource-group "$SOURCE_RG" \
#   --notes "Prevents writes and deletes. Use carefully."

# Validate locks.
az lock list \
  --resource-group "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/source-rg-locks-after-create.json"

# Remove lock before move if it blocks the operation.
az lock delete \
  --name "$DELETE_LOCK_NAME" \
  --resource-group "$SOURCE_RG"

# Validate removal.
az lock list \
  --resource-group "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/source-rg-locks-after-remove.json"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Move_Precheck_Skeleton

```powershell
# Purpose: validate move prerequisites before moving resources.
# This does not prove every provider-specific dependency is safe. It builds the required evidence first.

$TenantId = "<tenant-id>"
$SourceSubscriptionId = "<source-subscription-id>"
$DestinationSubscriptionId = "<destination-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$ResourceId = "<resource-id>"
$ProviderNamespace = "<provider-namespace>"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SourceSubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\move-precheck-transcript.txt"

# Confirm tenant match.
$SourceTenantId = (Get-AzSubscription -SubscriptionId $SourceSubscriptionId).TenantId
$DestinationTenantId = (Get-AzSubscription -SubscriptionId $DestinationSubscriptionId).TenantId

[PSCustomObject]@{
  SourceSubscriptionId      = $SourceSubscriptionId
  SourceTenantId            = $SourceTenantId
  DestinationSubscriptionId = $DestinationSubscriptionId
  DestinationTenantId       = $DestinationTenantId
  SameTenant                = ($SourceTenantId -eq $DestinationTenantId)
} | Tee-Object "$EvidencePath\tenant-match-check.txt"

if ($SourceTenantId -ne $DestinationTenantId) {
  throw "Source and destination subscriptions are not associated with the same tenant. Stop move."
}

# Confirm source and destination RGs exist.
Get-AzResourceGroup -Name $SourceResourceGroup |
  Tee-Object "$EvidencePath\source-rg-precheck.txt"

Set-AzContext -SubscriptionId $DestinationSubscriptionId

Get-AzResourceGroup -Name $DestinationResourceGroup |
  Tee-Object "$EvidencePath\destination-rg-precheck.txt"

# Confirm provider registration in destination subscription.
Get-AzResourceProvider -ProviderNamespace $ProviderNamespace |
  Select-Object ProviderNamespace,RegistrationState,ResourceTypes |
  Tee-Object "$EvidencePath\destination-provider-registration.txt"

# Return to source subscription.
Set-AzContext -SubscriptionId $SourceSubscriptionId

# Capture resource before move.
Get-AzResource -ResourceId $ResourceId |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  Tee-Object "$EvidencePath\resource-before-move.txt"

# Capture locks that may block move.
Get-AzResourceLock -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\source-rg-locks-move-precheck.txt"

Set-AzContext -SubscriptionId $DestinationSubscriptionId

Get-AzResourceLock -ResourceGroupName $DestinationResourceGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\destination-rg-locks-move-precheck.txt"

Set-AzContext -SubscriptionId $SourceSubscriptionId

# Capture existing role assignments on old resource scope.
Get-AzRoleAssignment -Scope $ResourceId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,RoleDefinitionName,Scope,ObjectType |
  Tee-Object "$EvidencePath\resource-rbac-before-move.txt"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SOURCE_SUBSCRIPTION_ID="<source-subscription-id>"
DESTINATION_SUBSCRIPTION_ID="<destination-subscription-id>"
SOURCE_RG="<source-resource-group>"
DESTINATION_RG="<destination-resource-group>"
RESOURCE_ID="<resource-id>"
PROVIDER_NAMESPACE="<provider-namespace>"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"

SOURCE_TENANT_ID=$(az account show --subscription "$SOURCE_SUBSCRIPTION_ID" --query tenantId -o tsv)
DESTINATION_TENANT_ID=$(az account show --subscription "$DESTINATION_SUBSCRIPTION_ID" --query tenantId -o tsv)

echo "Source subscription tenant: $SOURCE_TENANT_ID" | tee "$EVIDENCE_PATH/tenant-match-check.txt"
echo "Destination subscription tenant: $DESTINATION_TENANT_ID" | tee -a "$EVIDENCE_PATH/tenant-match-check.txt"

if [ "$SOURCE_TENANT_ID" != "$DESTINATION_TENANT_ID" ]; then
  echo "Source and destination subscriptions are not associated with the same tenant. Stop move."
  exit 1
fi

# Confirm source RG.
az account set --subscription "$SOURCE_SUBSCRIPTION_ID"

az group show \
  --name "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/source-rg-precheck.json"

az resource show \
  --ids "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/resource-before-move.json"

az lock list \
  --resource-group "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/source-rg-locks-move-precheck.json"

az role assignment list \
  --scope "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/resource-rbac-before-move.json"

# Confirm destination RG and provider registration.
az account set --subscription "$DESTINATION_SUBSCRIPTION_ID"

az group show \
  --name "$DESTINATION_RG" \
  -o json | tee "$EVIDENCE_PATH/destination-rg-precheck.json"

az provider show \
  --namespace "$PROVIDER_NAMESPACE" \
  --query "{Provider:namespace,State:registrationState}" \
  -o json | tee "$EVIDENCE_PATH/destination-provider-registration.json"

az lock list \
  --resource-group "$DESTINATION_RG" \
  -o json | tee "$EVIDENCE_PATH/destination-rg-locks-move-precheck.json"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Move_Execution_Skeleton

```powershell
# Purpose: move a resource to another resource group in the same subscription.
# For cross-subscription moves, validate tenant match and provider registration first.

$TenantId = "<tenant-id>"
$SourceSubscriptionId = "<source-subscription-id>"
$DestinationSubscriptionId = "<destination-subscription-id>"
$DestinationResourceGroup = "<destination-resource-group>"
$ResourceId = "<resource-id>"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SourceSubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\move-execution-transcript.txt"

# Capture old resource ID.
$OldResource = Get-AzResource -ResourceId $ResourceId

$OldResource |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  Tee-Object "$EvidencePath\resource-before-move-execution.txt"

# Same-subscription move.
if ($SourceSubscriptionId -eq $DestinationSubscriptionId) {
  Move-AzResource `
    -DestinationResourceGroupName $DestinationResourceGroup `
    -ResourceId $ResourceId `
    -Force
}
else {
  # Cross-subscription move.
  Move-AzResource `
    -DestinationSubscriptionId $DestinationSubscriptionId `
    -DestinationResourceGroupName $DestinationResourceGroup `
    -ResourceId $ResourceId `
    -Force
}

Stop-Transcript
```

```bash
# Azure CLI equivalent.
# Same-subscription move.

SOURCE_SUBSCRIPTION_ID="<source-subscription-id>"
DESTINATION_RG="<destination-resource-group>"
RESOURCE_ID="<resource-id>"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SOURCE_SUBSCRIPTION_ID"

az resource show \
  --ids "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/resource-before-move-execution.json"

az resource move \
  --destination-group "$DESTINATION_RG" \
  --ids "$RESOURCE_ID"
```

```bash
# Azure CLI equivalent.
# Cross-subscription move when source and destination subscriptions are in the same tenant.

SOURCE_SUBSCRIPTION_ID="<source-subscription-id>"
DESTINATION_SUBSCRIPTION_ID="<destination-subscription-id>"
DESTINATION_RG="<destination-resource-group>"
RESOURCE_ID="<resource-id>"
DESTINATION_RG_ID="/subscriptions/$DESTINATION_SUBSCRIPTION_ID/resourceGroups/$DESTINATION_RG"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SOURCE_SUBSCRIPTION_ID"

az resource show \
  --ids "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/resource-before-cross-subscription-move.json"

az resource move \
  --destination-group "$DESTINATION_RG_ID" \
  --ids "$RESOURCE_ID"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Post_Move_Validation_Skeleton

```powershell
# Purpose: validate resource placement, new resource ID, tags, locks, RBAC, and policy after move.

$TenantId = "<tenant-id>"
$DestinationSubscriptionId = "<destination-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$MovedResourceName = "<resource-name>"
$MovedResourceType = "<resource-type>"
$DeleteLockName = "lock-rg-delete-protection"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

$StandardTags = @{
  Environment = "Lab"
  Owner       = "CloudOps"
  CostCenter  = "IT-1001"
  Application = "GovernanceLab"
  Lifecycle   = "Training"
}

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $DestinationSubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\post-move-validation-transcript.txt"

# Validate resource no longer appears in source RG if source is in current subscription.
Get-AzResource -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Select-Object Name,ResourceType,Location,ResourceId |
  Tee-Object "$EvidencePath\source-rg-resources-after-move.txt"

# Validate resource appears in destination RG.
Get-AzResource -ResourceGroupName $DestinationResourceGroup |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  Tee-Object "$EvidencePath\destination-rg-resources-after-move.txt"

# Find moved resource.
$MovedResource = Get-AzResource `
  -ResourceGroupName $DestinationResourceGroup `
  -Name $MovedResourceName `
  -ResourceType $MovedResourceType

$MovedResource |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  Tee-Object "$EvidencePath\moved-resource-after-move.txt"

# Reapply or merge tags if needed.
Update-AzTag `
  -ResourceId $MovedResource.ResourceId `
  -Tag $StandardTags `
  -Operation Merge

# Recreate destination delete lock if needed.
$ExistingLock = Get-AzResourceLock `
  -ResourceGroupName $DestinationResourceGroup `
  -LockName $DeleteLockName `
  -ErrorAction SilentlyContinue

if ($ExistingLock) {
  Write-Output "Destination delete lock already exists."
}
else {
  New-AzResourceLock `
    -LockName $DeleteLockName `
    -LockLevel CanNotDelete `
    -ResourceGroupName $DestinationResourceGroup `
    -LockNotes "Protects destination resource group after move." `
    -Force
}

# Validate final tag and lock state.
Get-AzResource -ResourceId $MovedResource.ResourceId |
  Select-Object Name,ResourceType,ResourceId,Tags |
  Tee-Object "$EvidencePath\moved-resource-tags-after-remediation.txt"

Get-AzResourceLock -ResourceGroupName $DestinationResourceGroup |
  Tee-Object "$EvidencePath\destination-rg-locks-after-move.txt"

# Validate RBAC and policy state.
Get-AzRoleAssignment -Scope $MovedResource.ResourceId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,RoleDefinitionName,Scope,ObjectType |
  Tee-Object "$EvidencePath\moved-resource-rbac-after-move.txt"

Get-AzPolicyState -ResourceId $MovedResource.ResourceId -ErrorAction SilentlyContinue |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
  Tee-Object "$EvidencePath\moved-resource-policy-state-after-move.txt"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

DESTINATION_SUBSCRIPTION_ID="<destination-subscription-id>"
SOURCE_RG="<source-resource-group>"
DESTINATION_RG="<destination-resource-group>"
MOVED_RESOURCE_NAME="<resource-name>"
MOVED_RESOURCE_TYPE="<resource-type>"
DELETE_LOCK_NAME="lock-rg-delete-protection"
EVIDENCE_PATH="$HOME/azure-governance/02-rg-tags-locks-move"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$DESTINATION_SUBSCRIPTION_ID"

# Validate destination resource list.
az resource list \
  --resource-group "$DESTINATION_RG" \
  --query "[].{Name:name,Type:type,Location:location,Id:id,Tags:tags}" \
  -o json | tee "$EVIDENCE_PATH/destination-rg-resources-after-move.json"

# Get moved resource.
NEW_RESOURCE_ID=$(az resource list \
  --resource-group "$DESTINATION_RG" \
  --name "$MOVED_RESOURCE_NAME" \
  --resource-type "$MOVED_RESOURCE_TYPE" \
  --query "[0].id" \
  -o tsv)

echo "$NEW_RESOURCE_ID" | tee "$EVIDENCE_PATH/new-resource-id.txt"

az resource show \
  --ids "$NEW_RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/moved-resource-after-move.json"

# Reapply tags if needed.
az resource tag \
  --ids "$NEW_RESOURCE_ID" \
  --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001 Application=GovernanceLab Lifecycle=Training

# Recreate delete lock at destination RG if needed.
az lock create \
  --name "$DELETE_LOCK_NAME" \
  --lock-type CanNotDelete \
  --resource-group "$DESTINATION_RG" \
  --notes "Protects destination resource group after move."

# Validate final state.
az resource show \
  --ids "$NEW_RESOURCE_ID" \
  --query "{Name:name,Type:type,Id:id,Tags:tags}" \
  -o json | tee "$EVIDENCE_PATH/moved-resource-tags-after-remediation.json"

az lock list \
  --resource-group "$DESTINATION_RG" \
  -o json | tee "$EVIDENCE_PATH/destination-rg-locks-after-move.json"

az role assignment list \
  --scope "$NEW_RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/moved-resource-rbac-after-move.json"

az policy state list \
  --resource "$NEW_RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/moved-resource-policy-state-after-move.json"
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Evidence_Export_Skeleton

```powershell
# Purpose: export final evidence for resource groups, tags, locks, resource IDs, RBAC, policy, and activity log.

$TenantId = "<tenant-id>"
$SourceSubscriptionId = "<source-subscription-id>"
$DestinationSubscriptionId = "<destination-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$MovedResourceName = "<resource-name>"
$MovedResourceType = "<resource-type>"
$EvidencePath = "C:\Azure-Governance\02-RG-Tags-Locks-Move"

Connect-AzAccount -Tenant $TenantId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\02-rg-tags-locks-move-final-evidence-transcript.txt"

# Source subscription evidence.
Set-AzContext -SubscriptionId $SourceSubscriptionId

Get-AzContext |
  Format-List |
  Out-File "$EvidencePath\source-context.txt"

Get-AzResourceGroup -Name $SourceResourceGroup -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\source-rg-final.json"

Get-AzResource -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  Export-Csv "$EvidencePath\source-rg-resources-final.csv" -NoTypeInformation

Get-AzResourceLock -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\source-rg-locks-final.csv" -NoTypeInformation

# Destination subscription evidence.
Set-AzContext -SubscriptionId $DestinationSubscriptionId

Get-AzContext |
  Format-List |
  Out-File "$EvidencePath\destination-context.txt"

Get-AzResourceGroup -Name $DestinationResourceGroup |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\destination-rg-final.json"

Get-AzResource -ResourceGroupName $DestinationResourceGroup |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  Export-Csv "$EvidencePath\destination-rg-resources-final.csv" -NoTypeInformation

Get-AzResourceLock -ResourceGroupName $DestinationResourceGroup -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\destination-rg-locks-final.csv" -NoTypeInformation

# Moved resource evidence.
$MovedResource = Get-AzResource `
  -ResourceGroupName $DestinationResourceGroup `
  -Name $MovedResourceName `
  -ResourceType $MovedResourceType `
  -ErrorAction SilentlyContinue

if ($MovedResource) {
  $MovedResource |
    Select-Object Name,ResourceType,Location,ResourceId,Tags |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\moved-resource-final.json"

  Get-AzRoleAssignment -Scope $MovedResource.ResourceId -ErrorAction SilentlyContinue |
    Select-Object DisplayName,SignInName,RoleDefinitionName,Scope,ObjectType,ObjectId |
    Export-Csv "$EvidencePath\moved-resource-rbac-final.csv" -NoTypeInformation

  Get-AzPolicyState -ResourceId $MovedResource.ResourceId -ErrorAction SilentlyContinue |
    Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
    Export-Csv "$EvidencePath\moved-resource-policy-state-final.csv" -NoTypeInformation
}

# Activity log evidence for recent resource group and move activity.
Get-AzActivityLog -StartTime (Get-Date).AddDays(-1) |
  Where-Object {
    $_.ResourceGroupName -eq $SourceResourceGroup -or
    $_.ResourceGroupName -eq $DestinationResourceGroup
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,Status,Caller,CorrelationId |
  Export-Csv "$EvidencePath\activity-log-final.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Verification_Commands

```powershell
# Confirm context.
Get-AzContext

# Confirm subscriptions.
Get-AzSubscription | Select-Object Name,Id,TenantId,State

# Confirm resource groups.
Get-AzResourceGroup | Select-Object ResourceGroupName,Location,ProvisioningState,Tags

# Confirm a specific source RG.
Get-AzResourceGroup -Name "<source-resource-group>"

# Confirm a specific destination RG.
Get-AzResourceGroup -Name "<destination-resource-group>"

# Confirm resources in source RG.
Get-AzResource -ResourceGroupName "<source-resource-group>" |
  Select-Object Name,ResourceType,Location,ResourceId,Tags

# Confirm resources in destination RG.
Get-AzResource -ResourceGroupName "<destination-resource-group>" |
  Select-Object Name,ResourceType,Location,ResourceId,Tags

# Confirm tags on a resource group.
(Get-AzResourceGroup -Name "<resource-group-name>").Tags

# Confirm tags on a resource.
(Get-AzResource -ResourceId "<resource-id>").Tags

# Confirm locks at source RG.
Get-AzResourceLock -ResourceGroupName "<source-resource-group>"

# Confirm locks at destination RG.
Get-AzResourceLock -ResourceGroupName "<destination-resource-group>"

# Confirm provider registration.
Get-AzResourceProvider -ProviderNamespace "<provider-namespace>" |
  Select-Object ProviderNamespace,RegistrationState

# Confirm resource exists at new ID.
Get-AzResource -ResourceId "<new-resource-id>"

# Confirm RBAC at resource scope.
Get-AzRoleAssignment -Scope "<new-resource-id>"

# Confirm policy state.
Get-AzPolicyState -ResourceId "<new-resource-id>"
```

```bash
# Azure CLI equivalents.

az account show -o table

az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o table

az group list \
  --query "[].{Name:name, Location:location, State:properties.provisioningState, Tags:tags}" \
  -o table

az group show --name "<source-resource-group>" -o json

az group show --name "<destination-resource-group>" -o json

az resource list \
  --resource-group "<source-resource-group>" \
  --query "[].{Name:name, Type:type, Location:location, Id:id}" \
  -o table

az resource list \
  --resource-group "<destination-resource-group>" \
  --query "[].{Name:name, Type:type, Location:location, Id:id}" \
  -o table

az group show \
  --name "<resource-group-name>" \
  --query tags \
  -o json

az resource show \
  --ids "<resource-id>" \
  --query tags \
  -o json

az lock list \
  --resource-group "<source-resource-group>" \
  -o table

az lock list \
  --resource-group "<destination-resource-group>" \
  -o table

az provider show \
  --namespace "<provider-namespace>" \
  --query "{Provider:namespace,State:registrationState}" \
  -o table

az resource show \
  --ids "<new-resource-id>" \
  -o json

az role assignment list \
  --scope "<new-resource-id>" \
  -o table

az policy state list \
  --resource "<new-resource-id>" \
  -o table
```

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm current destination resource ID | Cloud Shell | `az resource show --ids "<new-resource-id>"` | Current resource exists |
| 2 | Remove destination lock if it blocks rollback | Cloud Shell | `az lock delete --name "<delete-lock-name>" --resource-group "<destination-resource-group>"` | Destination RG lock is removed |
| 3 | Confirm source RG still exists | Cloud Shell | `az group show --name "<source-resource-group>"` | Source RG is available |
| 4 | Confirm source provider registration if cross-subscription rollback | Cloud Shell | `az provider show --namespace "<provider-namespace>"` | Provider is registered |
| 5 | Move resource back to original RG | Cloud Shell | `az resource move --destination-group "<source-resource-group>" --ids "<new-resource-id>"` | Resource returns to source RG |
| 6 | Validate original placement | Cloud Shell | `az resource list --resource-group "<source-resource-group>" -o table` | Resource appears in source RG |
| 7 | Validate destination cleanup | Cloud Shell | `az resource list --resource-group "<destination-resource-group>" -o table` | Destination no longer contains rolled-back resource |
| 8 | Reapply original tags if needed | Cloud Shell | `az resource tag --ids "<rollback-resource-id>" --tags Environment=Lab Owner=CloudOps CostCenter=IT-1001` | Resource tags are restored |
| 9 | Recreate original lock if needed | Cloud Shell | `az lock create --name "<delete-lock-name>" --lock-type CanNotDelete --resource-group "<source-resource-group>"` | Source RG is protected again |
| 10 | Recreate RBAC if orphaned | Cloud Shell / PowerShell | Use task 03 RBAC workflow | Required access is restored |
| 11 | Validate application dependency | Portal / Cloud Shell | Application-specific health check | Service works after rollback |
| 12 | Export rollback evidence | Admin Workstation | Run evidence export skeleton | Rollback record is complete |

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Cannot create resource group | Wrong subscription context or missing permission | Cloud Shell | `az account show`; `az role assignment list --assignee <user>` | Set correct subscription and grant Resource Group Contributor or appropriate role |
| Resource group created in wrong region | Incorrect location selected for RG metadata | Cloud Shell | `az group show --name "<rg-name>" --query location -o tsv` | Recreate RG in intended metadata location if required |
| Tags disappear after update | Tag operation replaced existing tag set | Cloud Shell / PowerShell | `az group show --name "<rg-name>" --query tags` | Use merge-style tagging where supported or capture tags before replacement |
| Resource tags do not match RG tags | Tags do not automatically inherit by default | Portal / Cloud Shell | `az resource show --ids "<resource-id>" --query tags` | Apply tags directly or use Azure Policy inheritance/remediation later |
| Cannot delete resource group | Delete lock exists at RG, resource, subscription, or parent scope | Cloud Shell | `az lock list --resource-group "<rg-name>" -o table` | Remove lock only through approved change control |
| Cannot update resource | Read-only lock exists | Cloud Shell | `az lock list --resource-group "<rg-name>" -o table` | Remove or relocate read-only lock if operation is legitimate |
| Move fails immediately | Resource type does not support move | Admin Workstation | Check move support matrix for provider and type | Rebuild or redeploy resource instead of moving |
| Move fails across subscriptions | Source and destination subscriptions are in different tenants | Cloud Shell | `az account show --subscription "<sub-id>" --query tenantId -o tsv` | Associate subscriptions with same tenant or do not perform move |
| Move fails because subscription not registered | Destination subscription missing resource provider registration | Cloud Shell | `az provider show --namespace "<provider-namespace>"` | Register provider in destination subscription |
| Move blocked by lock | Read-only lock on source, destination, resource group, subscription, or resource | Cloud Shell | `az lock list --resource-group "<rg-name>" -o table` | Remove blocking lock temporarily, then restore after move |
| Move appears stuck | Azure temporarily locks source and target RGs during move | Portal / Cloud Shell | Check Activity Log and resource group operations | Wait for move operation to complete or fail before retry |
| Scripts break after move | Resource ID changed | Admin Workstation | Compare old and new resource IDs | Update scripts, dashboards, alerts, automation, and references |
| Access breaks after move | Role assignment was scoped to old resource ID or old RG | Cloud Shell | `az role assignment list --scope "<new-resource-id>"` | Recreate RBAC at correct scope |
| Policy behavior changes after move | Destination RG or subscription has different inherited policy | Cloud Shell | `az policy state list --resource "<new-resource-id>"` | Review policy assignments at destination scopes |
| Diagnostic settings break | Destination context changed dependencies or resource ID references | Portal / Cloud Shell | Check resource Diagnostic settings and Activity Log | Recreate diagnostic settings if required |
| Dependent resource cannot find moved resource | Dependency still references old resource ID or RG path | Portal / Cloud Shell | Review resource configuration and Activity Log | Update dependent resource configuration |
| Move to destination RG fails due to quota | Destination subscription lacks quota | Portal / Cloud Shell | Check subscription usage and quotas | Request quota increase or choose another subscription |
| CLI command succeeds but portal does not show resource yet | Portal cache or delayed ARM visibility | Portal / Cloud Shell | `az resource show --ids "<new-resource-id>"` | Refresh portal and verify through CLI |
| PowerShell cmdlet not found | Az.Resources module missing or old | Admin Workstation | `Get-Module Az.Resources -ListAvailable` | Install or update Az module |
| Azure CLI syntax fails | CLI version is stale | Admin Workstation | `az version` | Update Azure CLI or use Azure Cloud Shell |

# Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Manage_Subscriptions_Management_Groups_And_Tenant_Association.md | Establishes subscription and management group scope before resource group governance |
| 03_Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews.md | Recreates or validates access after resource moves |
| 04_Configure_Azure_Policy_Initiatives_Assignments_And_Remediation.md | Applies tag policy, allowed location policy, and remediation after resource group baseline |
| 05_Configure_Cost_Management_Budgets_Alerts_And_Advisor_Recommendations.md | Uses tags and resource group structure for cost accountability |
| 06_Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues.md | Troubleshoots failed moves, blocked deletes, inherited policy, RBAC, and lock conflicts |