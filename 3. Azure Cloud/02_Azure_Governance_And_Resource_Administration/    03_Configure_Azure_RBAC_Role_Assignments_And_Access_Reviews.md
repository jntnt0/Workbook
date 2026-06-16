# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Index
03_Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews.md
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Source_Basis
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Mental_Model
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Planning_Table
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Configuration_Checklist
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Portal_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Azure_CLI_Precheck_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_PowerShell_Precheck_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Group_Based_RBAC_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_User_RBAC_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Managed_Identity_RBAC_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Inherited_Access_Review_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Privileged_Role_Audit_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Entra_Access_Review_Prep_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Evidence_Export_Skeleton
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Verification_Commands
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Rollback
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Failure_Checks
Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Related_Labs

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure RBAC overview | Scope-based access control for Azure resources |
| Microsoft Learn | Assign Azure roles using the Azure portal | IAM workflow, role selection, member selection, managed identity assignment, optional conditions |
| Microsoft Learn | Azure RBAC scope overview | Management group, subscription, resource group, and resource scope model |
| Microsoft Learn | Best practices for Azure RBAC | Least privilege, limiting subscription owners, privileged role reduction, assigning to groups |
| Microsoft Learn | Azure role assignments with Azure CLI | Repeatable RBAC role assignment and validation commands |
| Microsoft Learn | Azure role assignments with Azure PowerShell | PowerShell RBAC role assignment and validation commands |
| Microsoft Learn | Azure role assignment conditions | Conditional role assignments when supported |
| Microsoft Learn | Microsoft Entra access reviews overview | Reviewing continued access for groups, applications, Entra roles, and Azure resource roles |
| Microsoft Learn | Microsoft Entra Privileged Identity Management | Reviewing Azure resource role assignments and privileged access through PIM |
| Microsoft Graph | Identity governance access reviews | Access review lifecycle and reporting basis |
| Azure CLI | `az role assignment`, `az ad group`, `az ad user`, `az identity` | RBAC configuration and inventory |
| Azure PowerShell | `Get-AzRoleAssignment`, `New-AzRoleAssignment`, `Remove-AzRoleAssignment` | RBAC configuration and evidence capture |
| Microsoft Graph PowerShell | `Microsoft.Graph.Identity.Governance` | Access review prep and review visibility where licensed |

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Azure RBAC | Authorization system that grants Azure resource permissions to security principals |
| Security principal | User, group, service principal, or managed identity that receives access |
| Role definition | Permission set such as Reader, Contributor, Owner, or a job-specific built-in role |
| Role assignment | Binding of principal, role definition, and scope |
| Scope | Boundary where access applies: management group, subscription, resource group, or resource |
| Inheritance | Role assignments at parent scopes flow down to child scopes |
| Least privilege | Assign only the permissions required at the narrowest practical scope |
| Group-based access | Preferred method where users inherit Azure access through Entra security groups |
| Direct user assignment | Exception case, not the normal access model |
| Managed identity assignment | Grant Azure service identity access to another resource |
| Privileged administrator role | High-impact role such as Owner, Contributor, User Access Administrator, or RBAC Administrator |
| Access review | Governance process to confirm whether users should keep group, app, role, or resource access |
| PIM review | Review path for Microsoft Entra roles and Azure resource roles through Privileged Identity Management |
| Effective access | What the principal can actually do after inherited assignments are considered |
| Orphaned access | Assignment no longer needed, no longer justified, or left behind after move or lifecycle change |
| First rule | Always identify scope before assigning a role |
| Blunt rule | Assign roles to groups, not users, unless there is a documented exception |

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant ID | `aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee` | `<tenant-id>` |
| Subscription ID | `11111111-2222-3333-4444-555555555555` | `<subscription-id>` |
| Management group ID | `corp-platform` | `<management-group-id>` |
| Resource group | `rg-platform-identity-001` | `<resource-group-name>` |
| Resource ID | `/subscriptions/<id>/resourceGroups/<rg>/providers/<provider>/<type>/<name>` | `<resource-id>` |
| Assignment scope type | `ManagementGroup`, `Subscription`, `ResourceGroup`, `Resource` | `<scope-type>` |
| Assignment scope | `/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>` | `<scope>` |
| Principal type | `Group` | `<principal-type>` |
| Group display name | `AZ-RG-Platform-Contributors` | `<group-display-name>` |
| Group object ID | `bbbbbbbb-cccc-dddd-eeee-ffffffffffff` | `<group-object-id>` |
| User UPN | `user@contoso.com` | `<user-upn>` |
| User object ID | `cccccccc-dddd-eeee-ffff-111111111111` | `<user-object-id>` |
| Managed identity name | `mi-app-platform-001` | `<managed-identity-name>` |
| Managed identity object ID | `dddddddd-eeee-ffff-1111-222222222222` | `<managed-identity-object-id>` |
| Role definition | `Reader` | `<role-definition-name>` |
| Role definition ID | `acdd72a7-3385-48ef-bd42-f606fba81ae7` | `<role-definition-id>` |
| Assignment description | `Lab access for platform operators` | `<role-assignment-description>` |
| Review cadence | `Quarterly` | `<review-cadence>` |
| Review owner | `CloudOps Manager` | `<review-owner>` |
| Reviewer group | `AZ-Governance-Reviewers` | `<reviewer-group>` |
| Evidence path | `C:\Azure-Governance\03-RBAC-Access-Reviews` | `<evidence-path>` |
| Rollback action | Remove role assignment | `<rollback-action>` |

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Configuration_Checklist

| Step | Task | Device | Portal / Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Azure admin session | Admin Workstation / Cloud Shell | `https://portal.azure.com` | `Connect-AzAccount -Tenant "<tenant-id>"` | Admin is signed in to correct tenant |
| 2 | Confirm active tenant | Cloud Shell | `az account tenant list -o table` | `Get-AzTenant` | Tenant is confirmed |
| 3 | Confirm active subscription | Cloud Shell | `az account show -o table` | `Get-AzContext` | Correct subscription is selected |
| 4 | Confirm target scope | Cloud Shell | `az group show --name "<resource-group-name>"` or `az resource show --ids "<resource-id>"` | `Get-AzResourceGroup`; `Get-AzResource` | Scope exists before assignment |
| 5 | Confirm current role assignments at scope | Cloud Shell | `az role assignment list --scope "<scope>" -o table` | `Get-AzRoleAssignment -Scope "<scope>"` | Current assignments are known |
| 6 | Confirm inherited assignments from parent scopes | Cloud Shell | `az role assignment list --scope "<scope>" --include-inherited -o table` | `Get-AzRoleAssignment -Scope "<scope>" -IncludeClassicAdministrators` | Effective access is known |
| 7 | Confirm role definition exists | Cloud Shell | `az role definition list --name "<role-definition-name>" -o table` | `Get-AzRoleDefinition -Name "<role-definition-name>"` | Role definition is valid |
| 8 | Confirm principal exists | Cloud Shell | `az ad group show --group "<group-display-name>"` or `az ad user show --id "<user-upn>"` | `Get-AzADGroup`; `Get-AzADUser` | Principal object ID is known |
| 9 | Prefer group-based assignment | Entra admin center / Cloud Shell | Create or select Entra security group | `Get-AzADGroup -DisplayName "<group-display-name>"` | RBAC assignment target is a group |
| 10 | Add members to access group | Entra admin center / Cloud Shell | `az ad group member add --group "<group-object-id>" --member-id "<user-object-id>"` | `Add-AzADGroupMember` | User receives access through group |
| 11 | Assign role to group at target scope | Cloud Shell | `az role assignment create --assignee-object-id "<group-object-id>" --assignee-principal-type Group --role "<role-definition-name>" --scope "<scope>"` | `New-AzRoleAssignment -ObjectId "<group-object-id>" -RoleDefinitionName "<role-definition-name>" -Scope "<scope>"` | Group receives role at scope |
| 12 | Validate assignment | Cloud Shell | `az role assignment list --assignee "<group-object-id>" --scope "<scope>" -o table` | `Get-AzRoleAssignment -ObjectId "<group-object-id>" -Scope "<scope>"` | Role assignment is visible |
| 13 | Test effective access with target user | User session | Open target resource or resource group | N/A | User has expected access only |
| 14 | Assign role to managed identity if required | Cloud Shell | `az role assignment create --assignee-object-id "<managed-identity-object-id>" --assignee-principal-type ServicePrincipal --role "<role-definition-name>" --scope "<scope>"` | `New-AzRoleAssignment -ObjectId "<managed-identity-object-id>" -RoleDefinitionName "<role-definition-name>" -Scope "<scope>"` | Managed identity receives required access |
| 15 | Avoid unnecessary privileged roles | Portal / Cloud Shell | Review Owner, Contributor, User Access Administrator, RBAC Administrator | `Get-AzRoleAssignment -Scope "<scope>" | Where-Object RoleDefinitionName -in @("Owner","Contributor","User Access Administrator")` | Privileged assignments are minimized |
| 16 | Capture all assignments at scope | Cloud Shell | `az role assignment list --scope "<scope>" --include-inherited -o json` | `Get-AzRoleAssignment -Scope "<scope>"` | Assignment evidence is saved |
| 17 | Prepare access review target | Entra admin center | Access reviews or PIM | Microsoft Graph PowerShell if licensed | Review target is identified |
| 18 | Create or document access review | Entra admin center | Identity Governance > Access Reviews | Graph PowerShell if available | Review exists or review plan is documented |
| 19 | Export review evidence | Admin Workstation | Access review results export | Graph PowerShell export if available | Review state is documented |
| 20 | Remediate stale access | Portal / Cloud Shell | Remove group membership or role assignment | `Remove-AzRoleAssignment` | Unneeded access is removed |
| 21 | Validate final access state | Cloud Shell | `az role assignment list --scope "<scope>" --include-inherited -o table` | `Get-AzRoleAssignment -Scope "<scope>"` | Final RBAC state is known |
| 22 | Export workbook evidence | Admin Workstation | Run evidence export skeleton | Run evidence export skeleton | Evidence files are saved |

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Portal_Skeleton

| Step | Portal Area | Action | Expected Result |
|---:|---|---|---|
| 1 | Azure Portal | Sign in as `<admin-upn>` | Correct tenant opens |
| 2 | Directories + subscriptions | Confirm selected directory | Portal is scoped to correct tenant |
| 3 | Target scope | Open Management group, Subscription, Resource group, or Resource | Assignment target is visible |
| 4 | Access control (IAM) | Select **Role assignments** | Current assignments are visible |
| 5 | Access control (IAM) | Select **Check access** | Existing user, group, or managed identity access can be reviewed |
| 6 | Access control (IAM) | Select **Add** > **Add role assignment** | Add role assignment wizard opens |
| 7 | Role tab | Select job function role or privileged administrator role | Role is selected |
| 8 | Role tab | Review role details before assignment | Permissions are understood |
| 9 | Members tab | Select **User, group, or service principal** | Principal selector opens |
| 10 | Members tab | Select Entra security group | Group is staged for assignment |
| 11 | Members tab | Add optional assignment description | Assignment reason is documented |
| 12 | Conditions tab | Add condition only when supported and required | Conditional access boundary is configured |
| 13 | Review + assign | Confirm scope, role, and principal | Assignment is created |
| 14 | Access control (IAM) | Refresh Role assignments tab | New assignment is visible |
| 15 | Access control (IAM) | Select **View inherited access** or include inherited assignments | Parent-scope access is reviewed |
| 16 | Microsoft Entra admin center | Open Identity Governance > Access reviews | Review capability is visible |
| 17 | Access reviews | Create review for group, app, Entra role, or Azure resource role through PIM | Review target is configured |
| 18 | Access reviews | Assign reviewers and recurrence | Review cadence is established |
| 19 | Access reviews | Complete or export review results | Review evidence is available |
| 20 | IAM / Entra groups | Remove stale assignments or group members | Access is remediated |

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Azure_CLI_Precheck_Skeleton

```bash
# Run in Azure Cloud Shell or an admin workstation with Azure CLI.
# Purpose: capture tenant, subscription, scope, role, and principal state before RBAC changes.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
RESOURCE_GROUP="<resource-group-name>"
RESOURCE_ID="<resource-id>"
SCOPE="<scope>"
ROLE_NAME="<role-definition-name>"
GROUP_DISPLAY_NAME="<group-display-name>"
USER_UPN="<user-upn>"
EVIDENCE_PATH="$HOME/azure-governance/03-rbac-access-reviews"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"

# Confirm visible tenants.
az account tenant list \
  -o table | tee "$EVIDENCE_PATH/tenant-list.txt"

# Confirm subscription inventory.
az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o table | tee "$EVIDENCE_PATH/subscription-list.txt"

# Set target subscription.
az account set --subscription "$SUBSCRIPTION_ID"

# Confirm active context.
az account show \
  --query "{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o json | tee "$EVIDENCE_PATH/active-context.json"

# Confirm resource group if scope is RG-based.
az group show \
  --name "$RESOURCE_GROUP" \
  -o json | tee "$EVIDENCE_PATH/resource-group-scope.json"

# Confirm resource if scope is resource-based.
az resource show \
  --ids "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/resource-scope.json"

# Confirm role definition.
az role definition list \
  --name "$ROLE_NAME" \
  --query "[].{Name:roleName, Id:name, Type:type, Description:description}" \
  -o json | tee "$EVIDENCE_PATH/role-definition.json"

# Confirm group.
az ad group show \
  --group "$GROUP_DISPLAY_NAME" \
  --query "{DisplayName:displayName, ObjectId:id, Mail:mail, SecurityEnabled:securityEnabled}" \
  -o json | tee "$EVIDENCE_PATH/group-principal.json"

# Confirm user.
az ad user show \
  --id "$USER_UPN" \
  --query "{DisplayName:displayName, UserPrincipalName:userPrincipalName, ObjectId:id, AccountEnabled:accountEnabled}" \
  -o json | tee "$EVIDENCE_PATH/user-principal.json"

# Capture current assignments at exact scope.
az role assignment list \
  --scope "$SCOPE" \
  --query "[].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope}" \
  -o json | tee "$EVIDENCE_PATH/rbac-current-exact-scope.json"

# Capture current assignments including inherited.
az role assignment list \
  --scope "$SCOPE" \
  --include-inherited \
  --query "[].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope}" \
  -o json | tee "$EVIDENCE_PATH/rbac-current-include-inherited.json"
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_PowerShell_Precheck_Skeleton

```powershell
# Run in elevated PowerShell or Azure Cloud Shell.
# Purpose: capture tenant, subscription, scope, role, and principal state before RBAC changes.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$Scope = "<scope>"
$RoleDefinitionName = "<role-definition-name>"
$GroupDisplayName = "<group-display-name>"
$UserPrincipalName = "<user-upn>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Import-Module Az.Accounts
Import-Module Az.Resources

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# Confirm tenant and subscription.
Get-AzTenant |
  Select-Object Name,Id |
  Tee-Object "$EvidencePath\tenant-list.txt"

Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Tee-Object "$EvidencePath\subscription-list.txt"

Get-AzContext |
  Select-Object Name,Subscription,Tenant,Account |
  Tee-Object "$EvidencePath\active-context.txt"

# Confirm scope.
Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resource-group-scope.txt"

Get-AzResource -ResourceId $ResourceId -ErrorAction SilentlyContinue |
  Select-Object Name,ResourceType,Location,ResourceId |
  Tee-Object "$EvidencePath\resource-scope.txt"

# Confirm role definition.
Get-AzRoleDefinition -Name $RoleDefinitionName |
  Select-Object Name,Id,Description,Actions,NotActions,DataActions,NotDataActions |
  Tee-Object "$EvidencePath\role-definition.txt"

# Confirm group principal.
Get-AzADGroup -DisplayName $GroupDisplayName |
  Select-Object DisplayName,Id,Mail,SecurityEnabled |
  Tee-Object "$EvidencePath\group-principal.txt"

# Confirm user principal.
Get-AzADUser -UserPrincipalName $UserPrincipalName |
  Select-Object DisplayName,UserPrincipalName,Id,AccountEnabled |
  Tee-Object "$EvidencePath\user-principal.txt"

# Capture current assignments at scope.
Get-AzRoleAssignment -Scope $Scope |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Tee-Object "$EvidencePath\rbac-current-exact-scope.txt"

# Capture privileged assignments at scope.
Get-AzRoleAssignment -Scope $Scope |
  Where-Object {
    $_.RoleDefinitionName -in @(
      "Owner",
      "Contributor",
      "User Access Administrator",
      "Role Based Access Control Administrator"
    )
  } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Tee-Object "$EvidencePath\privileged-rbac-current-scope.txt"
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Group_Based_RBAC_Skeleton

```powershell
# Purpose: assign Azure RBAC to an Entra security group at the target scope.
# Preferred model: group receives role, users receive group membership.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$Scope = "<scope>"
$RoleDefinitionName = "<role-definition-name>"
$GroupDisplayName = "<group-display-name>"
$UserPrincipalName = "<user-upn>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\group-based-rbac-transcript.txt"

# Resolve group and user.
$Group = Get-AzADGroup -DisplayName $GroupDisplayName
$User = Get-AzADUser -UserPrincipalName $UserPrincipalName

$Group |
  Select-Object DisplayName,Id,Mail,SecurityEnabled |
  Tee-Object "$EvidencePath\resolved-group.txt"

$User |
  Select-Object DisplayName,UserPrincipalName,Id,AccountEnabled |
  Tee-Object "$EvidencePath\resolved-user.txt"

# Optional: add user to group if membership is part of this lab.
# Use Entra admin center if Add-AzADGroupMember is unavailable in your module version.
Add-AzADGroupMember `
  -TargetGroupObjectId $Group.Id `
  -MemberObjectId $User.Id

# Confirm membership.
Get-AzADGroupMember -GroupObjectId $Group.Id |
  Select-Object DisplayName,UserPrincipalName,Id,ObjectType |
  Tee-Object "$EvidencePath\group-members-after-add.txt"

# Create RBAC assignment.
New-AzRoleAssignment `
  -ObjectId $Group.Id `
  -RoleDefinitionName $RoleDefinitionName `
  -Scope $Scope

# Validate assignment.
Get-AzRoleAssignment `
  -ObjectId $Group.Id `
  -Scope $Scope |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Tee-Object "$EvidencePath\group-rbac-assignment-after-create.txt"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
SCOPE="<scope>"
ROLE_NAME="<role-definition-name>"
GROUP_DISPLAY_NAME="<group-display-name>"
USER_UPN="<user-upn>"
EVIDENCE_PATH="$HOME/azure-governance/03-rbac-access-reviews"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

GROUP_OBJECT_ID=$(az ad group show --group "$GROUP_DISPLAY_NAME" --query id -o tsv)
USER_OBJECT_ID=$(az ad user show --id "$USER_UPN" --query id -o tsv)

echo "$GROUP_OBJECT_ID" | tee "$EVIDENCE_PATH/group-object-id.txt"
echo "$USER_OBJECT_ID" | tee "$EVIDENCE_PATH/user-object-id.txt"

# Add user to group.
az ad group member add \
  --group "$GROUP_OBJECT_ID" \
  --member-id "$USER_OBJECT_ID"

# Confirm membership.
az ad group member list \
  --group "$GROUP_OBJECT_ID" \
  --query "[].{DisplayName:displayName, UserPrincipalName:userPrincipalName, ObjectId:id}" \
  -o json | tee "$EVIDENCE_PATH/group-members-after-add.json"

# Create RBAC assignment.
az role assignment create \
  --assignee-object-id "$GROUP_OBJECT_ID" \
  --assignee-principal-type Group \
  --role "$ROLE_NAME" \
  --scope "$SCOPE"

# Validate assignment.
az role assignment list \
  --assignee "$GROUP_OBJECT_ID" \
  --scope "$SCOPE" \
  --query "[].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope}" \
  -o json | tee "$EVIDENCE_PATH/group-rbac-assignment-after-create.json"
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_User_RBAC_Skeleton

```powershell
# Purpose: assign Azure RBAC directly to a user only when there is a documented exception.
# Normal model should be group-based assignment.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$Scope = "<scope>"
$RoleDefinitionName = "<role-definition-name>"
$UserPrincipalName = "<user-upn>"
$ExceptionReason = "Temporary lab exception. Convert to group-based assignment."
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\direct-user-rbac-transcript.txt"

$User = Get-AzADUser -UserPrincipalName $UserPrincipalName

$User |
  Select-Object DisplayName,UserPrincipalName,Id,AccountEnabled |
  Tee-Object "$EvidencePath\direct-user-principal.txt"

# Record exception reason.
[PSCustomObject]@{
  UserPrincipalName = $UserPrincipalName
  UserObjectId      = $User.Id
  RoleDefinition    = $RoleDefinitionName
  Scope             = $Scope
  ExceptionReason   = $ExceptionReason
  ReviewRequired    = $true
} | Tee-Object "$EvidencePath\direct-user-assignment-exception.txt"

# Create direct user assignment.
New-AzRoleAssignment `
  -ObjectId $User.Id `
  -RoleDefinitionName $RoleDefinitionName `
  -Scope $Scope

# Validate assignment.
Get-AzRoleAssignment `
  -ObjectId $User.Id `
  -Scope $Scope |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Tee-Object "$EvidencePath\direct-user-rbac-after-create.txt"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
SCOPE="<scope>"
ROLE_NAME="<role-definition-name>"
USER_UPN="<user-upn>"
EVIDENCE_PATH="$HOME/azure-governance/03-rbac-access-reviews"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

USER_OBJECT_ID=$(az ad user show --id "$USER_UPN" --query id -o tsv)

echo "$USER_OBJECT_ID" | tee "$EVIDENCE_PATH/direct-user-object-id.txt"

cat > "$EVIDENCE_PATH/direct-user-assignment-exception.txt" <<EOF
UserPrincipalName=$USER_UPN
UserObjectId=$USER_OBJECT_ID
RoleDefinition=$ROLE_NAME
Scope=$SCOPE
ExceptionReason=Temporary lab exception. Convert to group-based assignment.
ReviewRequired=true
EOF

az role assignment create \
  --assignee-object-id "$USER_OBJECT_ID" \
  --assignee-principal-type User \
  --role "$ROLE_NAME" \
  --scope "$SCOPE"

az role assignment list \
  --assignee "$USER_OBJECT_ID" \
  --scope "$SCOPE" \
  -o json | tee "$EVIDENCE_PATH/direct-user-rbac-after-create.json"
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Managed_Identity_RBAC_Skeleton

```powershell
# Purpose: assign Azure RBAC to a managed identity for workload access.
# Example: managed identity gets Reader or Storage Blob Data Reader at a resource scope.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagedIdentityResourceGroup = "<managed-identity-resource-group>"
$ManagedIdentityName = "<managed-identity-name>"
$Scope = "<scope>"
$RoleDefinitionName = "<role-definition-name>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\managed-identity-rbac-transcript.txt"

# Resolve user-assigned managed identity.
$Identity = Get-AzUserAssignedIdentity `
  -ResourceGroupName $ManagedIdentityResourceGroup `
  -Name $ManagedIdentityName

$Identity |
  Select-Object Name,ResourceGroupName,PrincipalId,ClientId,Id |
  Tee-Object "$EvidencePath\managed-identity-resolved.txt"

# Assign role to managed identity principal ID.
New-AzRoleAssignment `
  -ObjectId $Identity.PrincipalId `
  -RoleDefinitionName $RoleDefinitionName `
  -Scope $Scope

# Validate assignment.
Get-AzRoleAssignment `
  -ObjectId $Identity.PrincipalId `
  -Scope $Scope |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Tee-Object "$EvidencePath\managed-identity-rbac-after-create.txt"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
MANAGED_IDENTITY_RG="<managed-identity-resource-group>"
MANAGED_IDENTITY_NAME="<managed-identity-name>"
SCOPE="<scope>"
ROLE_NAME="<role-definition-name>"
EVIDENCE_PATH="$HOME/azure-governance/03-rbac-access-reviews"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

MI_PRINCIPAL_ID=$(az identity show \
  --resource-group "$MANAGED_IDENTITY_RG" \
  --name "$MANAGED_IDENTITY_NAME" \
  --query principalId \
  -o tsv)

echo "$MI_PRINCIPAL_ID" | tee "$EVIDENCE_PATH/managed-identity-principal-id.txt"

az identity show \
  --resource-group "$MANAGED_IDENTITY_RG" \
  --name "$MANAGED_IDENTITY_NAME" \
  -o json | tee "$EVIDENCE_PATH/managed-identity-resolved.json"

az role assignment create \
  --assignee-object-id "$MI_PRINCIPAL_ID" \
  --assignee-principal-type ServicePrincipal \
  --role "$ROLE_NAME" \
  --scope "$SCOPE"

az role assignment list \
  --assignee "$MI_PRINCIPAL_ID" \
  --scope "$SCOPE" \
  -o json | tee "$EVIDENCE_PATH/managed-identity-rbac-after-create.json"
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Inherited_Access_Review_Skeleton

```powershell
# Purpose: review exact and inherited Azure RBAC at a target scope.
# This is the operational access review prep before formal Entra/PIM review.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$Scope = "<scope>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\inherited-access-review-transcript.txt"

# Exact scope assignments.
$ExactAssignments = Get-AzRoleAssignment -Scope $Scope

$ExactAssignments |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-exact-scope.csv" -NoTypeInformation

# All assignments visible from subscription for analysis.
$SubscriptionScope = "/subscriptions/$SubscriptionId"

$SubscriptionAssignments = Get-AzRoleAssignment -Scope $SubscriptionScope

$SubscriptionAssignments |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-subscription-scope.csv" -NoTypeInformation

# Privileged assignments at or above target subscription.
$SubscriptionAssignments |
  Where-Object {
    $_.RoleDefinitionName -in @(
      "Owner",
      "Contributor",
      "User Access Administrator",
      "Role Based Access Control Administrator"
    )
  } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-privileged-subscription-scope.csv" -NoTypeInformation

# Direct user assignments that should be reviewed.
$SubscriptionAssignments |
  Where-Object {
    $_.ObjectType -eq "User"
  } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-direct-user-assignments-review.csv" -NoTypeInformation

# Group assignments that should map to formal access reviews.
$SubscriptionAssignments |
  Where-Object {
    $_.ObjectType -eq "Group"
  } |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-group-assignments-review.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<subscription-id>"
SCOPE="<scope>"
EVIDENCE_PATH="$HOME/azure-governance/03-rbac-access-reviews"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

# Exact scope assignments.
az role assignment list \
  --scope "$SCOPE" \
  --query "[].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope, PrincipalId:principalId}" \
  -o json | tee "$EVIDENCE_PATH/rbac-exact-scope.json"

# Include inherited assignments.
az role assignment list \
  --scope "$SCOPE" \
  --include-inherited \
  --query "[].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope, PrincipalId:principalId}" \
  -o json | tee "$EVIDENCE_PATH/rbac-include-inherited.json"

# Privileged assignments.
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --query "[?roleDefinitionName=='Owner' || roleDefinitionName=='Contributor' || roleDefinitionName=='User Access Administrator' || roleDefinitionName=='Role Based Access Control Administrator'].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope, PrincipalId:principalId}" \
  -o json | tee "$EVIDENCE_PATH/rbac-privileged-subscription-scope.json"

# Direct user assignment review list.
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --query "[?principalType=='User'].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope, PrincipalId:principalId}" \
  -o json | tee "$EVIDENCE_PATH/rbac-direct-user-assignments-review.json"

# Group assignment review list.
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --query "[?principalType=='Group'].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope, PrincipalId:principalId}" \
  -o json | tee "$EVIDENCE_PATH/rbac-group-assignments-review.json"
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Privileged_Role_Audit_Skeleton

```powershell
# Purpose: identify privileged Azure RBAC assignments that need cleanup, PIM, or recurring review.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

$PrivilegedRoles = @(
  "Owner",
  "Contributor",
  "User Access Administrator",
  "Role Based Access Control Administrator"
)

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\privileged-role-audit-transcript.txt"

$Assignments = Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId"

$PrivilegedAssignments = $Assignments |
  Where-Object { $_.RoleDefinitionName -in $PrivilegedRoles } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId

$PrivilegedAssignments |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\privileged-role-assignments.txt"

$PrivilegedAssignments |
  Export-Csv "$EvidencePath\privileged-role-assignments.csv" -NoTypeInformation

# Count subscription Owners.
$OwnerAssignments = $PrivilegedAssignments |
  Where-Object { $_.RoleDefinitionName -eq "Owner" }

[PSCustomObject]@{
  SubscriptionId = $SubscriptionId
  OwnerCount     = ($OwnerAssignments | Measure-Object).Count
  ReviewRequired = (($OwnerAssignments | Measure-Object).Count -gt 3)
} | Tee-Object "$EvidencePath\subscription-owner-count-check.txt"

# Direct privileged user assignments.
$PrivilegedAssignments |
  Where-Object { $_.ObjectType -eq "User" } |
  Export-Csv "$EvidencePath\direct-privileged-user-assignments.csv" -NoTypeInformation

# Group-based privileged assignments.
$PrivilegedAssignments |
  Where-Object { $_.ObjectType -eq "Group" } |
  Export-Csv "$EvidencePath\group-based-privileged-assignments.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Entra_Access_Review_Prep_Skeleton

```powershell
# Purpose: prepare access review evidence for Entra security groups that carry Azure RBAC.
# Formal review creation may require Microsoft Entra ID Governance or PIM licensing.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$Scope = "<scope>"
$GroupDisplayName = "<group-display-name>"
$ReviewName = "Quarterly Azure RBAC Review - <group-display-name>"
$ReviewCadence = "Quarterly"
$Reviewer = "<reviewer-upn>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\entra-access-review-prep-transcript.txt"

# Resolve group.
$Group = Get-AzADGroup -DisplayName $GroupDisplayName

$Group |
  Select-Object DisplayName,Id,Mail,SecurityEnabled |
  Tee-Object "$EvidencePath\access-review-target-group.txt"

# Export group membership for review.
Get-AzADGroupMember -GroupObjectId $Group.Id |
  Select-Object DisplayName,UserPrincipalName,Id,ObjectType |
  Export-Csv "$EvidencePath\access-review-target-group-members.csv" -NoTypeInformation

# Export Azure RBAC assignments for this group.
Get-AzRoleAssignment -ObjectId $Group.Id |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\access-review-target-group-rbac-assignments.csv" -NoTypeInformation

# Create review planning record.
[PSCustomObject]@{
  ReviewName       = $ReviewName
  ReviewCadence    = $ReviewCadence
  ReviewTarget     = $GroupDisplayName
  TargetGroupId    = $Group.Id
  Reviewer         = $Reviewer
  ReviewType       = "Group membership review for Azure RBAC access"
  ReviewLocation   = "Microsoft Entra admin center > Identity Governance > Access reviews"
  RemediationPath  = "Remove stale group members or remove Azure RBAC role assignment"
  EvidencePath     = $EvidencePath
} | Export-Csv "$EvidencePath\access-review-plan.csv" -NoTypeInformation

Stop-Transcript
```

```powershell
# Optional Microsoft Graph PowerShell prep.
# This section is intentionally a prep skeleton because access review creation depends on tenant licensing,
# Graph permissions, and whether the target is group, app, Entra role, or Azure resource role through PIM.

$TenantId = "<tenant-id>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

# Install-Module Microsoft.Graph -Scope CurrentUser
# Install-Module Microsoft.Graph.Identity.Governance -Scope CurrentUser

Import-Module Microsoft.Graph.Authentication
Import-Module Microsoft.Graph.Identity.Governance

Connect-MgGraph `
  -TenantId $TenantId `
  -Scopes "AccessReview.ReadWrite.All","Group.Read.All","User.Read.All","RoleManagement.Read.Directory"

# Confirm context.
Get-MgContext |
  Out-File "$EvidencePath\graph-context.txt"

# List access review definitions visible to the account.
Get-MgIdentityGovernanceAccessReviewDefinition -All |
  Select-Object Id,DisplayName,Status,CreatedDateTime |
  Export-Csv "$EvidencePath\access-review-definitions-visible.csv" -NoTypeInformation

# Placeholder for formal review creation.
# Build formal review only after confirming:
# 1. License requirement
# 2. Graph permissions
# 3. Review target type
# 4. Reviewer identity
# 5. Recurrence and auto-apply behavior
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Evidence_Export_Skeleton

```powershell
# Purpose: export final RBAC and access review evidence.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagementGroupId = "<management-group-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$Scope = "<scope>"
$GroupDisplayName = "<group-display-name>"
$EvidencePath = "C:\Azure-Governance\03-RBAC-Access-Reviews"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\03-rbac-access-review-final-evidence-transcript.txt"

# Tenant and subscription evidence.
Get-AzTenant |
  Select-Object Name,Id |
  Export-Csv "$EvidencePath\tenants.csv" -NoTypeInformation

Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Export-Csv "$EvidencePath\subscriptions.csv" -NoTypeInformation

Get-AzContext |
  Format-List |
  Out-File "$EvidencePath\active-context.txt"

# Scope evidence.
Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\resource-group-scope.json"

Get-AzResource -ResourceId $ResourceId -ErrorAction SilentlyContinue |
  Select-Object Name,ResourceType,Location,ResourceId,Tags |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\resource-scope.json"

# RBAC evidence at management group, subscription, resource group, and resource.
if ($ManagementGroupId -ne "<management-group-id>") {
  $ManagementGroupScope = "/providers/Microsoft.Management/managementGroups/$ManagementGroupId"

  Get-AzRoleAssignment -Scope $ManagementGroupScope |
    Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
    Export-Csv "$EvidencePath\rbac-management-group-scope.csv" -NoTypeInformation
}

Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-subscription-scope.csv" -NoTypeInformation

Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId/resourceGroups/$ResourceGroupName" -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-resource-group-scope.csv" -NoTypeInformation

Get-AzRoleAssignment -Scope $ResourceId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-resource-scope.csv" -NoTypeInformation

# Privileged assignment evidence.
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
  Where-Object {
    $_.RoleDefinitionName -in @(
      "Owner",
      "Contributor",
      "User Access Administrator",
      "Role Based Access Control Administrator"
    )
  } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-privileged-subscription-scope.csv" -NoTypeInformation

# Direct user assignment evidence.
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
  Where-Object { $_.ObjectType -eq "User" } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-direct-user-assignments.csv" -NoTypeInformation

# Group assignment and membership evidence.
$Group = Get-AzADGroup -DisplayName $GroupDisplayName -ErrorAction SilentlyContinue

if ($Group) {
  $Group |
    Select-Object DisplayName,Id,Mail,SecurityEnabled |
    Export-Csv "$EvidencePath\review-group-object.csv" -NoTypeInformation

  Get-AzADGroupMember -GroupObjectId $Group.Id |
    Select-Object DisplayName,UserPrincipalName,Id,ObjectType |
    Export-Csv "$EvidencePath\review-group-members.csv" -NoTypeInformation

  Get-AzRoleAssignment -ObjectId $Group.Id |
    Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
    Export-Csv "$EvidencePath\review-group-rbac-assignments.csv" -NoTypeInformation
}

# Activity log evidence for RBAC assignment changes.
Get-AzActivityLog -StartTime (Get-Date).AddDays(-7) |
  Where-Object {
    $_.OperationName.Value -like "*roleAssignments*" -or
    $_.OperationName.Value -like "*authorization*"
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,Status,Caller,CorrelationId |
  Export-Csv "$EvidencePath\rbac-activity-log-last-7-days.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Verification_Commands

```powershell
# Confirm context.
Get-AzContext

# Confirm tenant.
Get-AzTenant

# Confirm subscription.
Get-AzSubscription | Select-Object Name,Id,TenantId,State

# Confirm target resource group scope.
Get-AzResourceGroup -Name "<resource-group-name>"

# Confirm target resource scope.
Get-AzResource -ResourceId "<resource-id>"

# Confirm role definition.
Get-AzRoleDefinition -Name "<role-definition-name>"

# Confirm group.
Get-AzADGroup -DisplayName "<group-display-name>"

# Confirm user.
Get-AzADUser -UserPrincipalName "<user-upn>"

# Confirm group membership.
Get-AzADGroupMember -GroupObjectId "<group-object-id>"

# Confirm exact role assignments at scope.
Get-AzRoleAssignment -Scope "<scope>"

# Confirm role assignments for group.
Get-AzRoleAssignment -ObjectId "<group-object-id>"

# Confirm role assignments for user.
Get-AzRoleAssignment -ObjectId "<user-object-id>"

# Confirm role assignments for managed identity.
Get-AzRoleAssignment -ObjectId "<managed-identity-object-id>"

# Confirm privileged assignments at subscription scope.
Get-AzRoleAssignment -Scope "/subscriptions/<subscription-id>" |
  Where-Object {
    $_.RoleDefinitionName -in @(
      "Owner",
      "Contributor",
      "User Access Administrator",
      "Role Based Access Control Administrator"
    )
  }

# Remove assignment validation after rollback.
Get-AzRoleAssignment -ObjectId "<principal-object-id>" -Scope "<scope>"
```

```bash
# Azure CLI equivalents.

az account show -o table

az account tenant list -o table

az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o table

az group show --name "<resource-group-name>" -o table

az resource show --ids "<resource-id>" -o json

az role definition list \
  --name "<role-definition-name>" \
  -o table

az ad group show \
  --group "<group-display-name>" \
  -o json

az ad user show \
  --id "<user-upn>" \
  -o json

az ad group member list \
  --group "<group-object-id>" \
  -o table

az role assignment list \
  --scope "<scope>" \
  -o table

az role assignment list \
  --scope "<scope>" \
  --include-inherited \
  -o table

az role assignment list \
  --assignee "<group-object-id>" \
  -o table

az role assignment list \
  --assignee "<user-object-id>" \
  -o table

az role assignment list \
  --assignee "<managed-identity-object-id>" \
  -o table

az role assignment list \
  --scope "/subscriptions/<subscription-id>" \
  --query "[?roleDefinitionName=='Owner' || roleDefinitionName=='Contributor' || roleDefinitionName=='User Access Administrator' || roleDefinitionName=='Role Based Access Control Administrator'].{PrincipalName:principalName, PrincipalType:principalType, Role:roleDefinitionName, Scope:scope}" \
  -o table
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm role assignment to remove | Cloud Shell | `az role assignment list --assignee "<principal-object-id>" --scope "<scope>" -o table` | Assignment is visible before removal |
| 2 | Remove group-based role assignment | Cloud Shell | `az role assignment delete --assignee "<group-object-id>" --role "<role-definition-name>" --scope "<scope>"` | Group no longer has role at scope |
| 3 | Remove direct user role assignment | Cloud Shell | `az role assignment delete --assignee "<user-object-id>" --role "<role-definition-name>" --scope "<scope>"` | User no longer has direct role at scope |
| 4 | Remove managed identity assignment | Cloud Shell | `az role assignment delete --assignee "<managed-identity-object-id>" --role "<role-definition-name>" --scope "<scope>"` | Managed identity role is removed |
| 5 | Remove user from access group if needed | Cloud Shell | `az ad group member remove --group "<group-object-id>" --member-id "<user-object-id>"` | User no longer receives access through group |
| 6 | Validate exact scope assignments | Cloud Shell | `az role assignment list --scope "<scope>" -o table` | Removed assignment is gone |
| 7 | Validate inherited access | Cloud Shell | `az role assignment list --scope "<scope>" --include-inherited -o table` | No unexpected inherited access remains |
| 8 | Cancel or close access review if it was created for lab only | Entra admin center | Identity Governance > Access reviews | Lab review is no longer active |
| 9 | Export rollback evidence | Admin Workstation | Run evidence export skeleton | Rollback evidence is saved |
| 10 | Document rollback reason | Operator | Record assignment removed, principal, scope, role, and validation result | Workbook rollback record is complete |

```powershell
# PowerShell rollback examples.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$Scope = "<scope>"
$RoleDefinitionName = "<role-definition-name>"
$GroupObjectId = "<group-object-id>"
$UserObjectId = "<user-object-id>"
$ManagedIdentityObjectId = "<managed-identity-object-id>"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# Remove group assignment.
Remove-AzRoleAssignment `
  -ObjectId $GroupObjectId `
  -RoleDefinitionName $RoleDefinitionName `
  -Scope $Scope

# Remove direct user assignment.
Remove-AzRoleAssignment `
  -ObjectId $UserObjectId `
  -RoleDefinitionName $RoleDefinitionName `
  -Scope $Scope

# Remove managed identity assignment.
Remove-AzRoleAssignment `
  -ObjectId $ManagedIdentityObjectId `
  -RoleDefinitionName $RoleDefinitionName `
  -Scope $Scope

# Validate removed assignment.
Get-AzRoleAssignment -Scope $Scope |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId
```

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Add role assignment button is disabled | Admin lacks `Microsoft.Authorization/roleAssignments/write` at scope | Azure Portal / Cloud Shell | `az role assignment list --assignee "<admin-object-id>" --include-inherited -o table` | Use Owner, User Access Administrator, or RBAC Administrator at required scope |
| Cannot see target subscription | Wrong directory selected or no subscription permission | Portal / Cloud Shell | `az account list --all -o table` | Switch directory or grant Reader at subscription scope |
| Role assignment appears successful but user cannot access resource | Token cache or propagation delay | User workstation | Sign out and sign in; wait briefly; check `az account clear` | Refresh token and validate after propagation |
| User has more access than expected | Inherited assignment from management group or subscription | Cloud Shell | `az role assignment list --scope "<scope>" --include-inherited -o table` | Remove or narrow parent assignment |
| User has direct access instead of group-based access | Assignment was made to user object | Cloud Shell | `az role assignment list --scope "<scope>" --query "[?principalType=='User']"` | Replace direct assignment with group assignment |
| Too many subscription Owners | Excess privileged assignments | Cloud Shell | Query Owner assignments at subscription scope | Reduce Owner count and use narrower roles |
| Contributor assigned where Reader is enough | Poor least-privilege selection | Portal / Cloud Shell | Review role definition permissions | Replace role with Reader or job-specific role |
| Assignment to group fails | Wrong group identifier or group not security-enabled | Cloud Shell | `az ad group show --group "<group-display-name>"` | Use correct security group object ID |
| Assignment to managed identity fails | Used client ID instead of principal/object ID | Cloud Shell | `az identity show --name "<mi-name>" --resource-group "<rg>" --query principalId -o tsv` | Use principal ID as assignee object ID |
| Role name fails in automation | Role name changed or preview role renamed | Cloud Shell | `az role definition list --name "<role-name>"` | Use role definition ID in automation |
| Scope string is invalid | Incorrect resource ID or management group scope format | Cloud Shell | `az resource show --ids "<resource-id>"` | Copy scope directly from Azure resource ID |
| Access review option missing | Licensing or permissions missing | Entra admin center | Check Identity Governance licensing and admin roles | Use licensed tenant and proper Identity Governance permissions |
| Azure resource role review not visible in Access reviews | Azure resource role reviews are handled through PIM path | Entra admin center | Identity Governance > Privileged Identity Management | Use PIM review for Azure resource roles |
| Reviewers cannot complete review | Reviewer not assigned or notification issue | Entra admin center | Open review details | Add correct reviewer or resend notification |
| Stale access remains after review | Review results were not auto-applied or manually remediated | Entra admin center / Cloud Shell | Compare review results with group membership and role assignments | Remove denied users from group or remove role assignment |
| Role assignment removal fails | Lock is not the issue; insufficient authorization is | Cloud Shell | `az role assignment delete ...` | Use account with role assignment delete permission |
| Access still exists after removing assignment | Access comes from another group or parent scope | Cloud Shell | `az role assignment list --assignee "<object-id>" --include-inherited -o table` | Remove additional assignment or group membership |
| PowerShell does not find Entra group | Module syntax/version issue or display name mismatch | PowerShell | `Get-AzADGroup -SearchString "<name>"` | Use object ID or update Az.Resources module |
| Graph access review commands fail | Missing Graph permissions or admin consent | PowerShell | `Get-MgContext` | Consent required Graph scopes or use portal |
| Azure CLI cannot query directory objects | Directory read permission or CLI auth issue | Cloud Shell | `az ad signed-in-user show` | Reauthenticate or use account with directory read permissions |

# Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Manage_Subscriptions_Management_Groups_And_Tenant_Association.md | Establishes management group and subscription hierarchy where inherited RBAC can be applied |
| 02_Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations.md | Defines resource group and resource scopes that RBAC assignments target |
| 04_Configure_Azure_Policy_Initiatives_Assignments_And_Remediation.md | Separates authorization control from resource compliance control |
| 05_Configure_Cost_Management_Budgets_Alerts_And_Advisor_Recommendations.md | Requires Cost Management Reader or Contributor roles for cost visibility and budget management |
| 06_Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues.md | Troubleshoots authorization failures, inherited assignments, policy conflicts, and stale access |