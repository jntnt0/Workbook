# Manage_Subscriptions_Management_Groups_And_Tenant_Association

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Index
01_Manage_Subscriptions_Management_Groups_And_Tenant_Association.md
Manage_Subscriptions_Management_Groups_And_Tenant_Association
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Source_Basis
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Mental_Model
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Planning_Table
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Configuration_Checklist
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Portal_Skeleton
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Azure_CLI_Precheck_Skeleton
Manage_Subscriptions_Management_Groups_And_Tenant_Association_PowerShell_Precheck_Skeleton
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Create_Management_Group_Skeleton
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Subscription_Move_Skeleton
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Inheritance_Validation_Skeleton
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Evidence_Export_Skeleton
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Verification_Commands
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Rollback
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Failure_Checks
Manage_Subscriptions_Management_Groups_And_Tenant_Association_Related_Labs

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure management groups overview | Management group hierarchy, tenant root group, inheritance, subscription organization |
| Microsoft Learn | Management group access | Azure RBAC at management group scope and inherited access |
| Microsoft Learn | Move management groups and subscriptions | Moving subscriptions under management groups |
| Microsoft Learn | Azure subscriptions and Microsoft Entra tenant association | Validating which tenant owns or trusts a subscription |
| Microsoft Learn | Azure RBAC scope model | Understanding scope inheritance across management groups, subscriptions, resource groups, and resources |
| Microsoft Learn | Azure Policy scope model | Understanding policy assignment inheritance from management groups |
| Azure CLI | `az account`, `az account management-group` | Tenant, subscription, and management group inventory and configuration |
| Azure PowerShell | `Az.Accounts`, `Az.Resources` | Tenant, subscription, management group, and role assignment validation |

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Microsoft Entra tenant | Identity boundary that owns users, groups, service principals, and subscription association |
| Azure subscription | Billing and resource boundary associated with one Microsoft Entra tenant |
| Management group | Governance scope above subscriptions used for inherited RBAC and policy |
| Tenant root group | Built-in top management group for the tenant hierarchy |
| Management group hierarchy | Parent and child structure that organizes management groups and subscriptions |
| Scope | Target level where RBAC, Policy, locks, and cost visibility may apply |
| Inheritance | RBAC and Policy assignments at higher scopes flow down to child scopes |
| Subscription association | Relationship between a subscription and the Microsoft Entra tenant it trusts |
| Subscription move | Placing a subscription under a different management group in the same tenant hierarchy |
| Root scope risk | Assignments at the tenant root group affect everything in the tenant |
| Governance baseline | Standard structure used before applying RBAC, Policy, budget, and resource organization |
| First rule | Confirm tenant and subscription context before changing management groups |
| Blunt rule | Do not assign broad Owner, Contributor, or Policy permissions at tenant root unless it is absolutely required |

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant display name | `Contoso Lab Tenant` | `<tenant-display-name>` |
| Tenant ID | `aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee` | `<tenant-id>` |
| Primary admin account | `admin@contoso.onmicrosoft.com` | `<admin-upn>` |
| Root management group ID | Same as tenant ID | `<root-management-group-id>` |
| Parent management group | `corp` | `<parent-management-group-id>` |
| Child management group | `corp-platform` | `<child-management-group-id>` |
| Child display name | `Corp Platform` | `<child-management-group-display-name>` |
| Subscription name | `Azure Lab Subscription` | `<subscription-name>` |
| Subscription ID | `11111111-2222-3333-4444-555555555555` | `<subscription-id>` |
| Subscription tenant ID | `aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee` | `<subscription-tenant-id>` |
| Target subscription parent | `corp-platform` | `<target-management-group-id>` |
| Governance owner group | `AZ-Governance-Admins` | `<governance-admin-group>` |
| Read-only visibility group | `AZ-Governance-Readers` | `<governance-reader-group>` |
| Evidence path | `C:\Azure-Governance\01-Subscriptions-MG` | `<evidence-path>` |
| Change window | `Lab` or approved production window | `<change-window>` |
| Rollback target | Previous management group or tenant root | `<rollback-parent-management-group-id>` |

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Configuration_Checklist

| Step | Task | Device | Portal / Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Azure admin session | Admin Workstation / Cloud Shell | `https://portal.azure.com` | `Connect-AzAccount -Tenant "<tenant-id>"` | Admin is signed in to the correct tenant |
| 2 | Confirm active tenant | Cloud Shell | `az account tenant list -o table` | `Get-AzTenant` | Tenant ID and display name are known |
| 3 | Confirm active subscription context | Cloud Shell | `az account show -o table` | `Get-AzContext` | Current subscription context is visible |
| 4 | List available subscriptions | Cloud Shell | `az account list --all -o table` | `Get-AzSubscription` | Subscription inventory is visible |
| 5 | Confirm subscription tenant association | Cloud Shell | `az account show --subscription "<subscription-id>" --query tenantId -o tsv` | `(Get-AzSubscription -SubscriptionId "<subscription-id>").TenantId` | Subscription tenant ID matches expected tenant |
| 6 | Open Management Groups blade | Azure Portal | Search `Management groups` | N/A | Management group hierarchy is visible |
| 7 | Confirm tenant root group visibility | Azure Portal / Cloud Shell | `az account management-group list -o table` | `Get-AzManagementGroup` | Root and child management groups are visible |
| 8 | Inspect current hierarchy | Cloud Shell | `az account management-group show --name "<root-management-group-id>" --expand --recurse` | `Get-AzManagementGroup -GroupName "<root-management-group-id>" -Expand -Recurse` | Current management group tree is known |
| 9 | Confirm admin permissions | Azure Portal | Management group > Access control (IAM) | `Get-AzRoleAssignment -Scope "/providers/Microsoft.Management/managementGroups/<mg-id>"` | Admin has required rights |
| 10 | Create parent management group if needed | Cloud Shell | `az account management-group create --name "<parent-management-group-id>" --display-name "<display-name>"` | `New-AzManagementGroup -GroupName "<parent-management-group-id>" -DisplayName "<display-name>"` | Parent management group exists |
| 11 | Create child management group | Cloud Shell | `az account management-group create --name "<child-management-group-id>" --display-name "<display-name>" --parent "<parent-management-group-id>"` | `New-AzManagementGroup -GroupName "<child-management-group-id>" -DisplayName "<display-name>" -ParentId "<parent-management-group-id>"` | Child management group exists under intended parent |
| 12 | Validate child placement | Cloud Shell | `az account management-group show --name "<child-management-group-id>" --expand --recurse` | `Get-AzManagementGroup -GroupName "<child-management-group-id>" -Expand -Recurse` | Child group parent and children are visible |
| 13 | Move subscription to target management group | Cloud Shell | `az account management-group subscription add --name "<target-management-group-id>" --subscription "<subscription-id>"` | `New-AzManagementGroupSubscription -GroupName "<target-management-group-id>" -SubscriptionId "<subscription-id>"` | Subscription appears under target management group |
| 14 | Validate subscription placement | Cloud Shell | `az account management-group subscription show --name "<target-management-group-id>" --subscription "<subscription-id>"` | `Get-AzManagementGroupSubscription -GroupName "<target-management-group-id>"` | Subscription is listed under target management group |
| 15 | Validate inherited RBAC exposure | Cloud Shell | `az role assignment list --scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>" -o table` | `Get-AzRoleAssignment -Scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>"` | Management group role assignments are visible |
| 16 | Validate inherited policy exposure | Cloud Shell | `az policy assignment list --scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>" -o table` | `Get-AzPolicyAssignment -Scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>"` | Policy assignments at management group scope are visible |
| 17 | Confirm subscription still usable | Cloud Shell | `az account set --subscription "<subscription-id>"; az group list -o table` | `Set-AzContext -SubscriptionId "<subscription-id>"; Get-AzResourceGroup` | Subscription resource access still works |
| 18 | Capture hierarchy evidence | Admin Workstation | Run evidence export skeleton | Run evidence export skeleton | Evidence files are saved |
| 19 | Document governance state | Operator | Record tenant, subscriptions, management groups, scope, inherited RBAC, inherited policy | Same | Workbook record is complete |
| 20 | Stop before applying RBAC or Policy | Operator | Continue with task 03 or 04 only after hierarchy is validated | Same | Governance foundation is ready |

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Portal_Skeleton

| Step | Portal Area | Action | Expected Result |
|---:|---|---|---|
| 1 | Azure Portal | Sign in as `<admin-upn>` | Correct tenant session opens |
| 2 | Directories + subscriptions | Confirm selected directory | Portal is scoped to `<tenant-display-name>` |
| 3 | Subscriptions | Open target subscription | Subscription name, ID, and directory are visible |
| 4 | Management groups | Open management group hierarchy | Tenant root group and children are visible |
| 5 | Management groups | Create or select parent group | Parent group exists |
| 6 | Management groups | Create child group under parent | Child group exists in intended location |
| 7 | Management groups | Add subscription to target management group | Subscription appears under target group |
| 8 | Management group > Access control (IAM) | Review role assignments | Existing management group RBAC is known |
| 9 | Management group > Policy | Review policy assignments | Existing management group policies are known |
| 10 | Subscription > Overview | Confirm subscription remains accessible | Subscription opens and resources are visible |

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Azure_CLI_Precheck_Skeleton

```bash
# Run in Azure Cloud Shell or an admin workstation with Azure CLI.
# Purpose: confirm tenant, subscription, and management group baseline before making changes.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
ROOT_MG_ID="<root-management-group-id>"
TARGET_MG_ID="<target-management-group-id>"
EVIDENCE_PATH="$HOME/azure-governance/01-subscriptions-mg"

mkdir -p "$EVIDENCE_PATH"

# Sign in if running locally.
az login --tenant "$TENANT_ID"

# Confirm tenants available to the account.
az account tenant list \
  --output table | tee "$EVIDENCE_PATH/tenant-list.txt"

# Confirm all visible subscriptions.
az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state, IsDefault:isDefault}" \
  --output table | tee "$EVIDENCE_PATH/subscription-list.txt"

# Set target subscription.
az account set --subscription "$SUBSCRIPTION_ID"

# Confirm active context.
az account show \
  --query "{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  --output json | tee "$EVIDENCE_PATH/active-subscription-context.json"

# Confirm subscription tenant association.
az account show \
  --subscription "$SUBSCRIPTION_ID" \
  --query tenantId \
  --output tsv | tee "$EVIDENCE_PATH/subscription-tenant-id.txt"

# List management groups visible to this account.
az account management-group list \
  --output table | tee "$EVIDENCE_PATH/management-group-list.txt"

# Capture root management group hierarchy.
az account management-group show \
  --name "$ROOT_MG_ID" \
  --expand \
  --recurse \
  --output json | tee "$EVIDENCE_PATH/root-management-group-tree-before.json"

# Capture target management group if it already exists.
az account management-group show \
  --name "$TARGET_MG_ID" \
  --expand \
  --recurse \
  --output json | tee "$EVIDENCE_PATH/target-management-group-before.json"
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_PowerShell_Precheck_Skeleton

```powershell
# Run in elevated PowerShell or Azure Cloud Shell.
# Purpose: confirm tenant, subscription, and management group baseline before making changes.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$RootManagementGroupId = "<root-management-group-id>"
$TargetManagementGroupId = "<target-management-group-id>"
$EvidencePath = "C:\Azure-Governance\01-Subscriptions-MG"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Install modules if needed.
# Install-Module Az.Accounts -Scope CurrentUser
# Install-Module Az.Resources -Scope CurrentUser

Import-Module Az.Accounts
Import-Module Az.Resources

Connect-AzAccount -Tenant $TenantId

# Confirm tenant visibility.
Get-AzTenant |
  Select-Object Name,Id |
  Tee-Object "$EvidencePath\tenant-list.txt"

# Confirm subscription inventory.
Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Tee-Object "$EvidencePath\subscription-list.txt"

# Set target subscription context.
Set-AzContext -SubscriptionId $SubscriptionId

# Confirm active context.
Get-AzContext |
  Select-Object Name,Subscription,Tenant,Account |
  Tee-Object "$EvidencePath\active-context.txt"

# Confirm subscription tenant association.
(Get-AzSubscription -SubscriptionId $SubscriptionId).TenantId |
  Tee-Object "$EvidencePath\subscription-tenant-id.txt"

# Capture management group visibility.
Get-AzManagementGroup |
  Tee-Object "$EvidencePath\management-group-list.txt"

# Capture root hierarchy if visible.
Get-AzManagementGroup -GroupName $RootManagementGroupId -Expand -Recurse |
  Tee-Object "$EvidencePath\root-management-group-tree-before.txt"

# Capture target management group if it exists.
Get-AzManagementGroup -GroupName $TargetManagementGroupId -Expand -Recurse -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-management-group-before.txt"
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Create_Management_Group_Skeleton

```powershell
# Run after precheck confirms tenant and permissions.
# Purpose: create a management group hierarchy for governance.

$TenantId = "<tenant-id>"
$ParentManagementGroupId = "<parent-management-group-id>"
$ParentDisplayName = "Corp"
$ChildManagementGroupId = "<child-management-group-id>"
$ChildDisplayName = "Corp Platform"
$EvidencePath = "C:\Azure-Governance\01-Subscriptions-MG"

Connect-AzAccount -Tenant $TenantId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Create parent management group if missing.
$ParentMg = Get-AzManagementGroup -GroupName $ParentManagementGroupId -ErrorAction SilentlyContinue

if ($ParentMg) {
  Write-Output "Parent management group already exists: $ParentManagementGroupId"
}
else {
  New-AzManagementGroup `
    -GroupName $ParentManagementGroupId `
    -DisplayName $ParentDisplayName
}

# Create child management group under the parent if missing.
$ChildMg = Get-AzManagementGroup -GroupName $ChildManagementGroupId -ErrorAction SilentlyContinue

if ($ChildMg) {
  Write-Output "Child management group already exists: $ChildManagementGroupId"
}
else {
  New-AzManagementGroup `
    -GroupName $ChildManagementGroupId `
    -DisplayName $ChildDisplayName `
    -ParentId $ParentManagementGroupId
}

# Validate final hierarchy.
Get-AzManagementGroup -GroupName $ParentManagementGroupId -Expand -Recurse |
  Tee-Object "$EvidencePath\parent-management-group-after-create.txt"

Get-AzManagementGroup -GroupName $ChildManagementGroupId -Expand -Recurse |
  Tee-Object "$EvidencePath\child-management-group-after-create.txt"
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Azure_CLI_Create_Management_Group_Skeleton

```bash
# Run in Azure Cloud Shell or an admin workstation with Azure CLI.

TENANT_ID="<tenant-id>"
PARENT_MG_ID="<parent-management-group-id>"
PARENT_DISPLAY_NAME="Corp"
CHILD_MG_ID="<child-management-group-id>"
CHILD_DISPLAY_NAME="Corp Platform"
EVIDENCE_PATH="$HOME/azure-governance/01-subscriptions-mg"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"

# Create parent management group.
az account management-group create \
  --name "$PARENT_MG_ID" \
  --display-name "$PARENT_DISPLAY_NAME"

# Create child management group.
az account management-group create \
  --name "$CHILD_MG_ID" \
  --display-name "$CHILD_DISPLAY_NAME" \
  --parent "$PARENT_MG_ID"

# Validate final hierarchy.
az account management-group show \
  --name "$PARENT_MG_ID" \
  --expand \
  --recurse \
  --output json | tee "$EVIDENCE_PATH/parent-management-group-after-create.json"

az account management-group show \
  --name "$CHILD_MG_ID" \
  --expand \
  --recurse \
  --output json | tee "$EVIDENCE_PATH/child-management-group-after-create.json"
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Subscription_Move_Skeleton

```powershell
# Run after target management group exists.
# Purpose: place a subscription under the target management group.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$TargetManagementGroupId = "<target-management-group-id>"
$EvidencePath = "C:\Azure-Governance\01-Subscriptions-MG"

Connect-AzAccount -Tenant $TenantId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Confirm subscription tenant association before move.
$Subscription = Get-AzSubscription -SubscriptionId $SubscriptionId

$Subscription |
  Select-Object Name,Id,TenantId,State |
  Tee-Object "$EvidencePath\subscription-before-move.txt"

# Confirm target management group exists.
Get-AzManagementGroup -GroupName $TargetManagementGroupId |
  Tee-Object "$EvidencePath\target-management-group-before-move.txt"

# Add subscription to target management group.
New-AzManagementGroupSubscription `
  -GroupName $TargetManagementGroupId `
  -SubscriptionId $SubscriptionId

# Validate subscription placement.
Get-AzManagementGroupSubscription -GroupName $TargetManagementGroupId |
  Tee-Object "$EvidencePath\target-management-group-subscriptions-after-move.txt"

Get-AzManagementGroup -GroupName $TargetManagementGroupId -Expand -Recurse |
  Tee-Object "$EvidencePath\target-management-group-tree-after-move.txt"
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Azure_CLI_Subscription_Move_Skeleton

```bash
# Run after target management group exists.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
TARGET_MG_ID="<target-management-group-id>"
EVIDENCE_PATH="$HOME/azure-governance/01-subscriptions-mg"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"

# Confirm subscription tenant association.
az account show \
  --subscription "$SUBSCRIPTION_ID" \
  --query "{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  --output json | tee "$EVIDENCE_PATH/subscription-before-move.json"

# Confirm target management group exists.
az account management-group show \
  --name "$TARGET_MG_ID" \
  --output json | tee "$EVIDENCE_PATH/target-management-group-before-move.json"

# Move subscription under target management group.
az account management-group subscription add \
  --name "$TARGET_MG_ID" \
  --subscription "$SUBSCRIPTION_ID"

# Validate subscription placement.
az account management-group subscription show \
  --name "$TARGET_MG_ID" \
  --subscription "$SUBSCRIPTION_ID" \
  --output json | tee "$EVIDENCE_PATH/subscription-placement-after-move.json"

az account management-group show \
  --name "$TARGET_MG_ID" \
  --expand \
  --recurse \
  --output json | tee "$EVIDENCE_PATH/target-management-group-tree-after-move.json"
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Inheritance_Validation_Skeleton

```powershell
# Purpose: validate governance inheritance points after subscription placement.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$TargetManagementGroupId = "<target-management-group-id>"
$ManagementGroupScope = "/providers/Microsoft.Management/managementGroups/$TargetManagementGroupId"
$EvidencePath = "C:\Azure-Governance\01-Subscriptions-MG"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Validate management group hierarchy.
Get-AzManagementGroup -GroupName $TargetManagementGroupId -Expand -Recurse |
  Tee-Object "$EvidencePath\mg-hierarchy-validation.txt"

# Validate subscription context still works.
Get-AzContext |
  Tee-Object "$EvidencePath\active-context-after-move.txt"

Get-AzResourceGroup |
  Select-Object ResourceGroupName,Location,ProvisioningState |
  Tee-Object "$EvidencePath\resource-groups-after-move.txt"

# Validate RBAC assignments at management group scope.
Get-AzRoleAssignment -Scope $ManagementGroupScope |
  Select-Object DisplayName,SignInName,RoleDefinitionName,Scope,ObjectType |
  Tee-Object "$EvidencePath\mg-rbac-assignments.txt"

# Validate Policy assignments at management group scope.
Get-AzPolicyAssignment -Scope $ManagementGroupScope |
  Select-Object Name,DisplayName,Scope,PolicyDefinitionId |
  Tee-Object "$EvidencePath\mg-policy-assignments.txt"

# Validate subscription-level role assignments separately.
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
  Select-Object DisplayName,SignInName,RoleDefinitionName,Scope,ObjectType |
  Tee-Object "$EvidencePath\subscription-rbac-assignments.txt"

# Validate subscription-level policy assignments separately.
Get-AzPolicyAssignment -Scope "/subscriptions/$SubscriptionId" |
  Select-Object Name,DisplayName,Scope,PolicyDefinitionId |
  Tee-Object "$EvidencePath\subscription-policy-assignments.txt"
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Evidence_Export_Skeleton

```powershell
# Purpose: export final state evidence after task completion.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$RootManagementGroupId = "<root-management-group-id>"
$TargetManagementGroupId = "<target-management-group-id>"
$EvidencePath = "C:\Azure-Governance\01-Subscriptions-MG"
$ManagementGroupScope = "/providers/Microsoft.Management/managementGroups/$TargetManagementGroupId"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\01-management-group-subscription-build-transcript.txt"

# Tenant and subscription evidence.
Get-AzTenant |
  Select-Object Name,Id |
  Export-Csv "$EvidencePath\tenants.csv" -NoTypeInformation

Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Export-Csv "$EvidencePath\subscriptions.csv" -NoTypeInformation

Get-AzContext |
  Select-Object Name,Subscription,Tenant,Account |
  Format-List |
  Out-File "$EvidencePath\active-context.txt"

# Management group hierarchy evidence.
Get-AzManagementGroup -GroupName $RootManagementGroupId -Expand -Recurse |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\root-management-group-tree.json"

Get-AzManagementGroup -GroupName $TargetManagementGroupId -Expand -Recurse |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\target-management-group-tree.json"

Get-AzManagementGroupSubscription -GroupName $TargetManagementGroupId |
  Export-Csv "$EvidencePath\target-management-group-subscriptions.csv" -NoTypeInformation

# Governance inheritance evidence.
Get-AzRoleAssignment -Scope $ManagementGroupScope |
  Select-Object DisplayName,SignInName,RoleDefinitionName,Scope,ObjectType,ObjectId |
  Export-Csv "$EvidencePath\management-group-rbac.csv" -NoTypeInformation

Get-AzPolicyAssignment -Scope $ManagementGroupScope |
  Select-Object Name,DisplayName,Scope,PolicyDefinitionId |
  Export-Csv "$EvidencePath\management-group-policy-assignments.csv" -NoTypeInformation

Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
  Select-Object DisplayName,SignInName,RoleDefinitionName,Scope,ObjectType,ObjectId |
  Export-Csv "$EvidencePath\subscription-rbac.csv" -NoTypeInformation

Get-AzPolicyAssignment -Scope "/subscriptions/$SubscriptionId" |
  Select-Object Name,DisplayName,Scope,PolicyDefinitionId |
  Export-Csv "$EvidencePath\subscription-policy-assignments.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Verification_Commands

```powershell
# Tenant visibility.
Get-AzTenant

# Subscription inventory.
Get-AzSubscription | Select-Object Name,Id,TenantId,State

# Active context.
Get-AzContext

# Management group inventory.
Get-AzManagementGroup

# Root management group tree.
Get-AzManagementGroup -GroupName "<root-management-group-id>" -Expand -Recurse

# Target management group tree.
Get-AzManagementGroup -GroupName "<target-management-group-id>" -Expand -Recurse

# Subscriptions under target management group.
Get-AzManagementGroupSubscription -GroupName "<target-management-group-id>"

# Management group RBAC.
Get-AzRoleAssignment -Scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>"

# Management group Policy.
Get-AzPolicyAssignment -Scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>"

# Subscription still accessible.
Set-AzContext -SubscriptionId "<subscription-id>"
Get-AzResourceGroup
```

```bash
# Azure CLI equivalents.

az account tenant list -o table

az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o table

az account show -o table

az account management-group list -o table

az account management-group show \
  --name "<root-management-group-id>" \
  --expand \
  --recurse \
  -o json

az account management-group show \
  --name "<target-management-group-id>" \
  --expand \
  --recurse \
  -o json

az account management-group subscription show \
  --name "<target-management-group-id>" \
  --subscription "<subscription-id>" \
  -o json

az role assignment list \
  --scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>" \
  -o table

az policy assignment list \
  --scope "/providers/Microsoft.Management/managementGroups/<target-management-group-id>" \
  -o table

az account set --subscription "<subscription-id>"
az group list -o table
```

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm current subscription placement | Cloud Shell | `az account management-group subscription show --name "<current-mg-id>" --subscription "<subscription-id>"` | Current placement is known |
| 2 | Confirm rollback target exists | Cloud Shell | `az account management-group show --name "<rollback-parent-management-group-id>"` | Rollback management group exists |
| 3 | Move subscription back to prior management group | Cloud Shell | `az account management-group subscription add --name "<rollback-parent-management-group-id>" --subscription "<subscription-id>"` | Subscription moves back to rollback target |
| 4 | Validate rollback placement | Cloud Shell | `az account management-group subscription show --name "<rollback-parent-management-group-id>" --subscription "<subscription-id>"` | Subscription appears under rollback target |
| 5 | Remove empty test child management group if needed | Cloud Shell | `az account management-group delete --name "<child-management-group-id>"` | Empty test group is removed |
| 6 | Validate hierarchy after rollback | Cloud Shell | `az account management-group show --name "<root-management-group-id>" --expand --recurse` | Hierarchy is back to expected state |
| 7 | Recheck RBAC and Policy inheritance | Cloud Shell / PowerShell | Run inheritance validation skeleton | Unexpected inherited access or policy is removed |
| 8 | Document rollback | Operator | Record original state, rollback command, validation result | Rollback evidence is complete |

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Management groups blade is empty or limited | Account lacks management group visibility or rights | Azure Portal / Cloud Shell | `az account management-group list -o table` | Use an account with management group Reader, Contributor, Owner, or appropriate delegated access |
| Tenant root group visible but not manageable | No rights at root management group | Azure Portal | Management groups > Tenant root group > IAM | Assign least privilege at the required lower scope instead of root when possible |
| Subscription does not appear in inventory | Wrong directory selected or no subscription permission | Portal / Cloud Shell | `az account list --all -o table` | Switch directory or grant Reader at subscription scope |
| Subscription tenant ID does not match expected tenant | Subscription is associated with another Microsoft Entra tenant | Cloud Shell | `az account show --subscription "<subscription-id>" --query tenantId -o tsv` | Use correct tenant or complete subscription directory association before management group placement |
| Cannot add subscription to management group | Missing management group or subscription permission | Cloud Shell | `az account management-group subscription add --name "<mg-id>" --subscription "<subscription-id>"` | Grant required management group access and subscription access |
| Cannot create management group | Insufficient permissions or name conflict | Cloud Shell | `az account management-group create --name "<mg-id>"` | Use unique group ID and proper management group Contributor or Owner rights |
| Wrong subscription context used | Active CLI or PowerShell context points to another subscription | Cloud Shell | `az account show`; `Get-AzContext` | Run `az account set --subscription "<subscription-id>"` or `Set-AzContext -SubscriptionId "<subscription-id>"` |
| Inherited RBAC appears unexpectedly | Role assignment exists at parent management group or root | Cloud Shell / PowerShell | `az role assignment list --scope "/providers/Microsoft.Management/managementGroups/<parent-mg-id>" -o table` | Review parent scope assignments and move subscription only after risk is understood |
| Inherited Policy appears unexpectedly | Policy assignment exists at parent management group or root | Cloud Shell / PowerShell | `az policy assignment list --scope "/providers/Microsoft.Management/managementGroups/<parent-mg-id>" -o table` | Exempt, move, or adjust assignment scope only through change control |
| Resource operations begin failing after move | Inherited deny policy, policy effect, or RBAC change | Portal / Cloud Shell | Check Activity Log, Policy compliance, IAM | Identify inherited assignment and resolve scope or assignment issue |
| Delete management group fails | Management group still contains subscriptions or child groups | Cloud Shell | `az account management-group show --name "<mg-id>" --expand --recurse` | Move children out before deleting |
| PowerShell cmdlet not found | Az.Resources module missing or stale | Admin Workstation | `Get-Module Az.Resources -ListAvailable` | Install or update Az modules |
| CLI command not found | Azure CLI outdated | Admin Workstation | `az version` | Update Azure CLI or use Azure Cloud Shell |

# Manage_Subscriptions_Management_Groups_And_Tenant_Association_Related_Labs

| Lab                                                                        | Relationship                                                                                 |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| 02_Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations.md       | Builds resource group governance under validated subscription hierarchy                      |
| 03_Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews.md             | Applies access control after management group scope is established                           |
| 04_Configure_Azure_Policy_Initiatives_Assignments_And_Remediation.md       | Applies policy and initiative assignments to management groups and subscriptions             |
| 05_Configure_Cost_Management_Budgets_Alerts_And_Advisor_Recommendations.md | Uses subscription and management group scope for cost visibility and budgets                 |
| 06_Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues.md      | Troubleshoots failures caused by hierarchy, scope, RBAC, policy, locks, and cost permissions |