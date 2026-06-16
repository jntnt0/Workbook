# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Index

## Purpose

Compare the major Microsoft cloud permission models so an admin can tell which role system controls which task.

This workbook covers:

- Azure RBAC roles
- Microsoft Entra directory roles
- Microsoft 365 admin roles
- Workload-specific roles and role groups
- Azure subscription scope
- Management group scope
- Resource group scope
- Resource scope
- Tenant-wide identity scope
- Microsoft 365 service scope
- Exchange Online role groups
- SharePoint admin roles
- Teams admin roles
- Intune roles
- Defender roles
- Purview roles
- Graph delegated scopes
- Application permissions
- Privileged Identity Management planning
- Role inventory and verification

## Outcome

By the end of this workbook, the admin should be able to:

- Explain the difference between Azure RBAC and Entra roles
- Explain why Global Administrator does not automatically mean Azure subscription Owner
- Explain why Azure subscription Owner does not automatically mean Global Administrator
- Identify which role model controls a portal or task
- Inventory Azure RBAC role assignments
- Inventory Entra directory roles
- Inventory Microsoft 365 admin roles
- Inventory selected workload admin roles
- Identify overprivileged assignments
- Identify missing role assignments
- Document role boundaries before configuring privileged access

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Source_Basis

| Source | Relevant Area | What It Supports |
|---|---|---|
| Microsoft Entra documentation | Directory roles, users, groups, role assignments, PIM, authentication, Conditional Access | Identity-plane role comparison |
| Azure management documentation | Azure RBAC, subscriptions, management groups, resource groups, role assignments, scope inheritance | Azure resource-plane role comparison |
| Microsoft 365 documentation fork | Microsoft 365 admin center roles, service roles, billing roles, domain roles, support roles | M365 admin role comparison |
| Exchange Online documentation | Exchange role groups, recipients, mail flow, mailbox permissions, accepted domains | Exchange workload role comparison |
| SharePoint Online documentation | SharePoint admin center, site admins, sharing, OneDrive, storage, service admin role | SharePoint workload role comparison |
| Teams documentation | Teams admin center, Teams Administrator, policies, meetings, calling, apps | Teams workload role comparison |
| Intune documentation | Intune RBAC, scope tags, device, compliance, app, endpoint management roles | Endpoint workload role comparison |
| Defender documentation | Security Reader, Security Administrator, Defender portal permissions, XDR operations | Security workload role comparison |
| Purview documentation | Compliance roles, role groups, audit, DLP, retention, eDiscovery | Compliance workload role comparison |
| Microsoft Graph documentation | Delegated scopes, application permissions, directory read/write permissions | API permission comparison |
| Existing Workbook 01 | Tenant, subscription, directory, and domain baseline | Supplies target tenant and subscription context |
| Existing Workbook 02 | Admin portals, PowerShell, Azure CLI, and Graph baseline | Supplies tooling required for inventory |
| Existing Workbook 03 | Custom domain and namespace baseline | Supplies domain role context |
| Existing Workbook structure | Source basis, mental model, planning, checklist, skeletons, verification, rollback, failure checks | Obsidian-ready workbook format |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Mental_Model

| Role Plane | Controls | Scope Model | Example Roles | Common Mistake |
|---|---|---|---|---|
| Azure RBAC | Azure resources | Management group, subscription, resource group, resource | Owner, Contributor, Reader, User Access Administrator | Thinking it grants Microsoft 365 tenant admin rights |
| Entra directory roles | Identity and directory objects | Tenant-wide or administrative unit scoped | Global Administrator, User Administrator, Groups Administrator, Privileged Role Administrator | Thinking it grants Azure resource access automatically |
| Microsoft 365 admin roles | Microsoft 365 tenant services | Tenant-wide service admin scopes | Global Reader, Exchange Administrator, SharePoint Administrator, Teams Administrator, Billing Administrator | Thinking all role visibility is the same across workload portals |
| Exchange Online role groups | Exchange recipients, mail flow, compliance handoff | Exchange role group scope | Organization Management, Recipient Management, View-Only Organization Management | Confusing M365 Exchange Administrator with every Exchange role group capability |
| SharePoint roles | SharePoint tenant and sites | Tenant admin and site admin scopes | SharePoint Administrator, Site Collection Administrator | Confusing SharePoint admin with site owner |
| Teams roles | Teams policy and service administration | Tenant service scope | Teams Administrator, Teams Communications Administrator | Confusing Teams owner with Teams Administrator |
| Intune RBAC | Endpoint and device management | Role, scope group, scope tag | Intune Administrator, Policy and Profile Manager, Help Desk Operator | Missing scope tag or group assignment |
| Defender roles | Security operations and Defender workloads | Role-based or unified RBAC depending on service | Security Reader, Security Administrator, Global Reader | Assuming portal access means action permission |
| Purview role groups | Compliance and data governance | Purview role group scope | Compliance Administrator, eDiscovery Manager, Audit Reader | Missing Purview-specific role group membership |
| Microsoft Graph delegated scopes | API actions as signed-in user | Consent plus signed-in user role | User.Read.All, Directory.Read.All, RoleManagement.Read.Directory | Having Graph scope but not having required user role |
| Microsoft Graph application permissions | App-only automation | Tenant-wide app consent | Directory.Read.All, User.ReadWrite.All | Granting app-only permissions too broadly |
| PIM eligible role | Just-in-time role assignment | Eligible, active, time-bound | Eligible Global Administrator, eligible Owner | Forgetting role must be activated before use |
| Administrative unit | Limited directory admin scope | Subset of users or groups | Scoped User Administrator | Expecting full tenant reach |

## Core Rule

There is no single Microsoft cloud permission model.

Always ask:

1. What object is being managed?
2. Which control plane owns that object?
3. What role system controls that plane?
4. What scope is the role assigned at?
5. Is the role active or only eligible?
6. Is the portal showing a tenant role, subscription role, workload role, or API permission?

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Role_Plane_Map

| Admin Task | Correct Role Plane | Typical Minimum Role | Portal / Tool |
|---|---|---|---|
| View Azure resources | Azure RBAC | Reader | Azure portal, Az PowerShell, Azure CLI |
| Create Azure resources | Azure RBAC | Contributor | Azure portal, Az PowerShell, Azure CLI |
| Assign Azure RBAC | Azure RBAC | Owner or User Access Administrator | Azure portal, Az PowerShell, Azure CLI |
| Manage management groups | Azure RBAC | Management Group Contributor / Owner | Azure portal |
| Manage Azure Policy | Azure RBAC | Resource Policy Contributor or Owner | Azure portal |
| Manage users | Entra roles | User Administrator | Entra admin center, Graph |
| Manage groups | Entra roles | Groups Administrator | Entra admin center, Graph |
| Manage directory roles | Entra roles | Privileged Role Administrator | Entra admin center, Graph |
| Manage Conditional Access | Entra roles | Conditional Access Administrator | Entra admin center |
| Manage authentication methods | Entra roles | Authentication Policy Administrator | Entra admin center |
| Manage app registrations | Entra roles | Application Administrator / Cloud Application Administrator | Entra admin center, Graph |
| Manage enterprise apps | Entra roles | Cloud Application Administrator | Entra admin center |
| Manage M365 licenses | M365 / Entra roles | License Administrator | Microsoft 365 admin center |
| Manage billing | Microsoft 365 admin roles | Billing Administrator | Microsoft 365 admin center |
| View service health | Microsoft 365 admin roles | Service Support Administrator or Global Reader | Microsoft 365 admin center |
| Manage Microsoft 365 domains | Microsoft 365 / Entra roles | Domain Name Administrator or Global Administrator | M365 admin center, Entra |
| Manage Exchange mailboxes | Exchange workload role | Exchange Administrator / Recipient Management | Exchange admin center |
| Manage Exchange mail flow | Exchange workload role | Exchange Administrator / Organization Management | Exchange admin center |
| Manage SharePoint tenant | SharePoint workload role | SharePoint Administrator | SharePoint admin center |
| Manage Teams policies | Teams workload role | Teams Administrator | Teams admin center |
| Manage Intune devices | Intune workload role | Intune Administrator or scoped Intune RBAC role | Intune admin center |
| Manage Defender incidents | Security role | Security Operator / Security Administrator | Defender portal |
| Manage Purview retention | Compliance role group | Compliance Administrator / Records Management | Purview portal |
| Manage eDiscovery | Compliance role group | eDiscovery Manager | Purview portal |
| Run Graph user inventory | Graph delegated scope plus directory read role | Directory.Read.All plus suitable user role | Microsoft Graph PowerShell |
| Run app-only automation | Graph app permission plus admin consent | Application permission and consent | App registration / Graph |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Scope_Model_Table

| Scope Type | Applies To | Example Scope | Inheritance | Notes |
|---|---|---|---|---|
| Tenant | Entra roles, M365 roles, many Graph permissions | Entire Entra tenant | Usually tenant-wide unless scoped by admin unit or service model | Highest identity scope |
| Administrative unit | Some Entra roles | Users/groups in an AU | Limited to AU members | Useful for delegated user/group admin |
| Management group | Azure RBAC | /providers/Microsoft.Management/managementGroups/prod | Inherits to child management groups and subscriptions | Azure-only governance scope |
| Subscription | Azure RBAC | /subscriptions/<subscription-id> | Inherits to resource groups and resources | Billing and resource container |
| Resource group | Azure RBAC | /subscriptions/<id>/resourceGroups/rg-app | Inherits to resources in RG | Common least privilege scope |
| Resource | Azure RBAC | /subscriptions/<id>/resourceGroups/<rg>/providers/... | No broad inheritance below resource except child resources | Most specific Azure RBAC scope |
| Exchange role group | Exchange workload | Organization Management | Exchange workload only | Workload-specific permissions |
| SharePoint site | SharePoint workload | Site collection admin | Site and child content | Different from SharePoint tenant admin |
| Teams team | Teams object | Team owner | That team only | Not the same as Teams Administrator |
| Intune scope tag | Intune workload | Devices with tag | Depends on role and assignment | Restricts Intune object visibility |
| Purview role group | Compliance workload | eDiscovery Manager | Compliance feature scope | Portal/service-specific |
| Graph delegated scope | Microsoft Graph | User.Read.All | API permission plus signed-in user authority | Both scope and user role matter |
| Graph application permission | Microsoft Graph | Directory.Read.All app-only | App permission tenant-wide unless restricted by resource model | High-risk if over granted |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Privileged_Role_Risk_Table

| Role | Plane | Risk Level | Why It Matters | Review Action |
|---|---|---|---|---|
| Global Administrator | Entra / M365 | Critical | Full tenant control across most identity and M365 services | Keep extremely limited, use PIM |
| Privileged Role Administrator | Entra | Critical | Can manage role assignments | Review frequently |
| Owner | Azure RBAC | Critical | Full control of Azure resources and RBAC at assigned scope | Review scope and inheritance |
| User Access Administrator | Azure RBAC | Critical | Can assign Azure RBAC without owning resources | Review scope and inheritance |
| Application Administrator | Entra | High | Can manage app registrations and credentials | Review app ownership |
| Cloud Application Administrator | Entra | High | Can manage enterprise apps and app access | Review app impact |
| Conditional Access Administrator | Entra | High | Can lock out or weaken access policy | Require change control |
| Security Administrator | Security | High | Can manage security settings and incidents | Review operational need |
| Compliance Administrator | Purview / M365 | High | Can manage compliance features and data access | Review compliance scope |
| Exchange Administrator | Exchange / M365 | High | Can manage mailboxes, mail flow, and recipient settings | Review mailbox and transport impact |
| SharePoint Administrator | SharePoint / M365 | High | Can manage tenant sharing and site settings | Review external sharing exposure |
| Teams Administrator | Teams / M365 | Medium / High | Can manage communication and collaboration policies | Review policy impact |
| Intune Administrator | Intune / M365 | High | Can manage device compliance, apps, and endpoint posture | Review device control impact |
| Billing Administrator | M365 billing | Medium / High | Can manage purchases, subscriptions, and billing data | Review financial control |
| License Administrator | M365 / Entra | Medium | Can assign or remove service access | Review license changes |
| User Administrator | Entra | Medium / High | Can manage users and passwords | Review password reset scope |
| Groups Administrator | Entra | Medium | Can manage groups and membership | Review privileged group membership |
| Directory Reader | Entra | Low / Medium | Read directory objects | Review if broad visibility matters |
| Global Reader | Entra / M365 | Low / Medium | Broad read-only visibility | Good baseline read role |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Planning_Table

| Item | Value To Record | Why It Matters |
|---|---|---|
| Tenant ID |  | Role inventory must target the correct tenant |
| Subscription ID |  | Azure RBAC inventory must target the correct subscription |
| Management group ID |  | Detects inherited Azure RBAC |
| Admin account |  | Identifies who ran inventory |
| Current Azure context |  | Prevents wrong subscription analysis |
| Current Graph context |  | Prevents wrong tenant analysis |
| Global Administrators |  | Critical tenant role |
| Privileged Role Administrators |  | Can grant directory roles |
| Azure subscription Owners |  | Can fully manage subscription and RBAC |
| User Access Administrators |  | Can delegate Azure resource access |
| Conditional Access Administrators |  | Can alter access controls |
| Exchange Administrators |  | Can manage mail and mail flow |
| SharePoint Administrators |  | Can manage sites and sharing |
| Teams Administrators |  | Can manage Teams policies |
| Intune Administrators |  | Can manage endpoint posture |
| Security Administrators |  | Can manage Defender/security configuration |
| Compliance Administrators |  | Can manage Purview/compliance configuration |
| Billing Administrators |  | Can manage billing and subscriptions |
| License Administrators |  | Can assign service access |
| PIM enabled | Yes / No | Determines active vs eligible privilege |
| Emergency access accounts |  | Must be excluded from risky role changes |
| Role review cadence |  | Keeps admin sprawl controlled |
| Evidence output path |  | Stores role inventory exports |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm workbook 01 tenant/subscription baseline | Admin workstation | Review workbook 01 output | Tenant ID and subscription ID are known |
| 2 | Confirm workbook 02 tooling baseline | Admin workstation | Review workbook 02 output | Az, Azure CLI, Graph, and portals are ready |
| 3 | Confirm current Azure PowerShell context | Admin workstation | Get-AzContext | Correct tenant and subscription are selected |
| 4 | Confirm current Graph context | Admin workstation | Get-MgContext | Correct tenant and scopes are selected |
| 5 | Inventory Azure subscriptions | Admin workstation | Get-AzSubscription | Available subscriptions are listed |
| 6 | Set target subscription | Admin workstation | Set-AzContext -SubscriptionId "<subscription-id>" | Correct subscription selected |
| 7 | Inventory Azure RBAC at subscription scope | Admin workstation | Get-AzRoleAssignment -Scope "/subscriptions/<subscription-id>" | Azure role assignments exported |
| 8 | Inventory Azure RBAC inherited assignments | Admin workstation | Get-AzRoleAssignment -Scope "/subscriptions/<subscription-id>" -IncludeClassicAdministrators | Inherited/classic assignments reviewed if visible |
| 9 | Review Azure Owners | Admin workstation | Filter RoleDefinitionName -eq Owner | Owners are documented |
| 10 | Review Azure User Access Administrators | Admin workstation | Filter RoleDefinitionName -eq User Access Administrator | RBAC delegators are documented |
| 11 | Review Azure Contributors | Admin workstation | Filter RoleDefinitionName -eq Contributor | Resource admins are documented |
| 12 | Review management group RBAC if visible | Admin workstation | Get-AzManagementGroup; Get-AzRoleAssignment | Parent-scope permissions are documented |
| 13 | Inventory Entra directory roles | Admin workstation | Get-MgDirectoryRole -All | Active directory roles are listed |
| 14 | Inventory Entra role members | Admin workstation | Get-MgDirectoryRoleMember | Role membership is exported |
| 15 | Review Global Administrators | Admin workstation | Directory role member export | Global Administrators are documented |
| 16 | Review Privileged Role Administrators | Admin workstation | Directory role member export | Privileged Role Administrators are documented |
| 17 | Review Conditional Access Administrators | Admin workstation | Directory role member export | CA administrators are documented |
| 18 | Inventory role eligibility if PIM data is available | Admin workstation | Role management / PIM commands or portal | Eligible vs active status is documented |
| 19 | Inventory Microsoft 365 admin roles | Admin workstation | Microsoft 365 admin center > Roles | M365 role assignments are documented |
| 20 | Inventory Exchange role groups | Admin workstation | Get-RoleGroup; Get-RoleGroupMember | Exchange workload roles are documented |
| 21 | Inventory SharePoint admin access | Admin workstation | M365 admin center / SharePoint admin center / Graph | SharePoint admins are documented |
| 22 | Inventory Teams admin roles | Admin workstation | Teams admin center / Graph roles | Teams admins are documented |
| 23 | Inventory Intune role assignments | Admin workstation | Intune admin center / Graph if available | Intune role and scope tag assignments are documented |
| 24 | Inventory Defender roles | Admin workstation | Defender portal / Entra roles | Security roles are documented |
| 25 | Inventory Purview role groups | Admin workstation | Purview portal | Compliance roles are documented |
| 26 | Compare role plane to task map | Admin workstation | Workbook Role Plane Map | Correct permission model is identified per admin task |
| 27 | Identify overprivileged assignments | Admin workstation | Compare exports | Excessive Global Admin, Owner, UAA, PRA assignments flagged |
| 28 | Identify missing role assignments | Admin workstation | Compare task needs to role assignments | Missing roles are documented |
| 29 | Document PIM recommendations | Admin workstation | Workbook notes | Standing privilege reduction plan is documented |
| 30 | Export final role baseline | Admin workstation | Run role inventory skeleton | Role baseline evidence is saved |
| 31 | Save final workbook | Admin workstation | Commit note to workbook repo or Obsidian vault | Role comparison baseline is ready for later labs |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Precheck_Skeleton

```powershell
# ============================================================
# Role Plane Comparison Precheck Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$OutputRoot = ".\Role_Plane_Comparison_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

# -----------------------------
# Confirm Azure PowerShell context
# -----------------------------
Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

$AzContext = Get-AzContext

$AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Format-List

$AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzContext.csv")

# -----------------------------
# Confirm Graph context
# -----------------------------
Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "RoleManagement.Read.Directory", `
    "User.Read.All", `
    "Group.Read.All"

$MgContext = Get-MgContext

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgContext.csv")

# -----------------------------
# Confirm organization
# -----------------------------
$Organization = Get-MgOrganization

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Organization.csv")

# -----------------------------
# Confirm subscription
# -----------------------------
Get-AzSubscription -SubscriptionId $SubscriptionId |
    Select-Object Name, Id, TenantId, State |
    Format-List

Get-AzSubscription -SubscriptionId $SubscriptionId |
    Select-Object Name, Id, TenantId, State |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "TargetSubscription.csv")

Write-Host "Precheck complete. Output path: $OutputPath"
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Azure_RBAC_Inventory_Skeleton

```powershell
# ============================================================
# Azure RBAC Inventory Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$OutputRoot = ".\Role_Plane_Comparison_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# -----------------------------
# Subscription role assignments
# -----------------------------
$SubscriptionScope = "/subscriptions/$SubscriptionId"

$SubscriptionRoleAssignments = Get-AzRoleAssignment -Scope $SubscriptionScope

$SubscriptionRoleAssignments |
    Select-Object DisplayName, SignInName, ObjectType, ObjectId, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Format-Table -AutoSize

$SubscriptionRoleAssignments |
    Select-Object DisplayName, SignInName, ObjectType, ObjectId, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzureRBAC_SubscriptionScope.csv")

# -----------------------------
# Critical Azure RBAC role review
# -----------------------------
$CriticalAzureRoles = @(
    "Owner",
    "User Access Administrator",
    "Contributor"
)

$CriticalAzureRoleAssignments = $SubscriptionRoleAssignments |
    Where-Object {$_.RoleDefinitionName -in $CriticalAzureRoles}

$CriticalAzureRoleAssignments |
    Select-Object DisplayName, SignInName, ObjectType, ObjectId, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Format-Table -AutoSize

$CriticalAzureRoleAssignments |
    Select-Object DisplayName, SignInName, ObjectType, ObjectId, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzureRBAC_CriticalRoles.csv")

# -----------------------------
# Resource group role assignments
# -----------------------------
$ResourceGroups = Get-AzResourceGroup

$RgRoleAssignments = foreach ($Rg in $ResourceGroups) {
    $Scope = "/subscriptions/$SubscriptionId/resourceGroups/$($Rg.ResourceGroupName)"
    Get-AzRoleAssignment -Scope $Scope -ErrorAction SilentlyContinue |
        Select-Object `
            @{Name="ResourceGroupName";Expression={$Rg.ResourceGroupName}},
            DisplayName,
            SignInName,
            ObjectType,
            ObjectId,
            RoleDefinitionName,
            Scope
}

$RgRoleAssignments |
    Sort-Object ResourceGroupName, RoleDefinitionName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzureRBAC_ResourceGroupScopes.csv")

# -----------------------------
# Management group visibility
# -----------------------------
try {
    $ManagementGroups = Get-AzManagementGroup

    $ManagementGroups |
        Select-Object Name, DisplayName, Id, TenantId |
        Format-Table -AutoSize

    $ManagementGroups |
        Select-Object Name, DisplayName, Id, TenantId |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Azure_ManagementGroups.csv")

    $ManagementGroupRoleAssignments = foreach ($Mg in $ManagementGroups) {
        Get-AzRoleAssignment -Scope $Mg.Id -ErrorAction SilentlyContinue |
            Select-Object `
                @{Name="ManagementGroupName";Expression={$Mg.Name}},
                DisplayName,
                SignInName,
                ObjectType,
                ObjectId,
                RoleDefinitionName,
                Scope
    }

    $ManagementGroupRoleAssignments |
        Sort-Object ManagementGroupName, RoleDefinitionName, DisplayName |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzureRBAC_ManagementGroupScopes.csv")
}
catch {
    Write-Warning "Management group role assignments could not be inventoried."
    $_.Exception.Message | Out-File -FilePath (Join-Path $OutputPath "Azure_ManagementGroup_Error.txt")
}

# -----------------------------
# Summary
# -----------------------------
$AzureRbacSummary = [PSCustomObject]@{
    SubscriptionId                     = $SubscriptionId
    TotalSubscriptionAssignments        = $SubscriptionRoleAssignments.Count
    OwnerAssignments                    = ($SubscriptionRoleAssignments | Where-Object {$_.RoleDefinitionName -eq "Owner"}).Count
    UserAccessAdministratorAssignments = ($SubscriptionRoleAssignments | Where-Object {$_.RoleDefinitionName -eq "User Access Administrator"}).Count
    ContributorAssignments              = ($SubscriptionRoleAssignments | Where-Object {$_.RoleDefinitionName -eq "Contributor"}).Count
    ReaderAssignments                   = ($SubscriptionRoleAssignments | Where-Object {$_.RoleDefinitionName -eq "Reader"}).Count
    DateCollected                       = Get-Date
}

$AzureRbacSummary |
    Format-List

$AzureRbacSummary |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzureRBAC_Summary.csv")
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Entra_Role_Inventory_Skeleton

```powershell
# ============================================================
# Microsoft Entra Directory Role Inventory Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Role_Plane_Comparison_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "RoleManagement.Read.Directory", `
    "User.Read.All", `
    "Group.Read.All"

# -----------------------------
# Active directory roles
# -----------------------------
$DirectoryRoles = Get-MgDirectoryRole -All

$DirectoryRoles |
    Select-Object DisplayName, Id, Description |
    Sort-Object DisplayName |
    Format-Table -AutoSize

$DirectoryRoles |
    Select-Object DisplayName, Id, Description |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_ActiveDirectoryRoles.csv")

# -----------------------------
# Directory role members
# -----------------------------
$DirectoryRoleMembers = foreach ($Role in $DirectoryRoles) {
    $Members = Get-MgDirectoryRoleMember -DirectoryRoleId $Role.Id -All -ErrorAction SilentlyContinue

    foreach ($Member in $Members) {
        [PSCustomObject]@{
            RoleDisplayName = $Role.DisplayName
            RoleId          = $Role.Id
            MemberId        = $Member.Id
            ODataType       = $Member.AdditionalProperties.'@odata.type'
            DisplayName     = $Member.AdditionalProperties.displayName
            UserPrincipalName = $Member.AdditionalProperties.userPrincipalName
            Mail            = $Member.AdditionalProperties.mail
            AppId           = $Member.AdditionalProperties.appId
        }
    }
}

$DirectoryRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Format-Table -AutoSize

$DirectoryRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_DirectoryRoleMembers.csv")

# -----------------------------
# Critical Entra role review
# -----------------------------
$CriticalEntraRoles = @(
    "Global Administrator",
    "Privileged Role Administrator",
    "Conditional Access Administrator",
    "Security Administrator",
    "Compliance Administrator",
    "Application Administrator",
    "Cloud Application Administrator",
    "User Administrator",
    "Groups Administrator",
    "Authentication Administrator",
    "Privileged Authentication Administrator"
)

$CriticalEntraRoleMembers = $DirectoryRoleMembers |
    Where-Object {$_.RoleDisplayName -in $CriticalEntraRoles}

$CriticalEntraRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Format-Table -AutoSize

$CriticalEntraRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_CriticalRoleMembers.csv")

# -----------------------------
# Role templates
# -----------------------------
$RoleTemplates = Get-MgDirectoryRoleTemplate -All

$RoleTemplates |
    Select-Object DisplayName, Id, Description |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_DirectoryRoleTemplates.csv")

# -----------------------------
# Summary
# -----------------------------
$EntraRoleSummary = [PSCustomObject]@{
    TenantId                       = $TenantId
    ActiveDirectoryRoleCount        = $DirectoryRoles.Count
    DirectoryRoleMemberCount        = $DirectoryRoleMembers.Count
    GlobalAdministratorCount        = ($DirectoryRoleMembers | Where-Object {$_.RoleDisplayName -eq "Global Administrator"}).Count
    PrivilegedRoleAdministratorCount = ($DirectoryRoleMembers | Where-Object {$_.RoleDisplayName -eq "Privileged Role Administrator"}).Count
    ConditionalAccessAdminCount     = ($DirectoryRoleMembers | Where-Object {$_.RoleDisplayName -eq "Conditional Access Administrator"}).Count
    SecurityAdministratorCount      = ($DirectoryRoleMembers | Where-Object {$_.RoleDisplayName -eq "Security Administrator"}).Count
    DateCollected                   = Get-Date
}

$EntraRoleSummary |
    Format-List

$EntraRoleSummary |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_Role_Summary.csv")
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_M365_Admin_Role_Inventory_Skeleton

```powershell
# ============================================================
# Microsoft 365 Admin Role Inventory Skeleton
# Uses Entra directory roles because M365 admin roles are backed by directory role assignments
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Role_Plane_Comparison_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "RoleManagement.Read.Directory", `
    "User.Read.All", `
    "Group.Read.All"

$DirectoryRoles = Get-MgDirectoryRole -All

$DirectoryRoleMembers = foreach ($Role in $DirectoryRoles) {
    $Members = Get-MgDirectoryRoleMember -DirectoryRoleId $Role.Id -All -ErrorAction SilentlyContinue

    foreach ($Member in $Members) {
        [PSCustomObject]@{
            RoleDisplayName   = $Role.DisplayName
            RoleId            = $Role.Id
            MemberId          = $Member.Id
            ODataType         = $Member.AdditionalProperties.'@odata.type'
            DisplayName       = $Member.AdditionalProperties.displayName
            UserPrincipalName = $Member.AdditionalProperties.userPrincipalName
            Mail              = $Member.AdditionalProperties.mail
        }
    }
}

$M365AdminRolesOfInterest = @(
    "Global Administrator",
    "Global Reader",
    "Exchange Administrator",
    "SharePoint Administrator",
    "Teams Administrator",
    "Teams Communications Administrator",
    "Teams Communications Support Engineer",
    "Teams Communications Support Specialist",
    "Intune Administrator",
    "Security Administrator",
    "Security Reader",
    "Compliance Administrator",
    "Compliance Data Administrator",
    "Billing Administrator",
    "License Administrator",
    "Service Support Administrator",
    "Helpdesk Administrator",
    "User Administrator",
    "Groups Administrator",
    "Reports Reader",
    "Message Center Reader",
    "Power Platform Administrator",
    "Dynamics 365 Administrator",
    "Domain Name Administrator"
)

$M365RoleMembers = $DirectoryRoleMembers |
    Where-Object {$_.RoleDisplayName -in $M365AdminRolesOfInterest}

$M365RoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Format-Table -AutoSize

$M365RoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "M365_AdminRoleMembers.csv")

$M365RoleSummary = $M365RoleMembers |
    Group-Object RoleDisplayName |
    Select-Object Name, Count |
    Sort-Object Name

$M365RoleSummary |
    Format-Table -AutoSize

$M365RoleSummary |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "M365_AdminRoleSummary.csv")
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Exchange_Role_Group_Inventory_Skeleton

```powershell
# ============================================================
# Exchange Online Role Group Inventory Skeleton
# ============================================================

$AdminUpn = "<admin-upn>"
$OutputRoot = ".\Role_Plane_Comparison_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-ExchangeOnline -UserPrincipalName $AdminUpn

# -----------------------------
# Exchange role groups
# -----------------------------
$RoleGroups = Get-RoleGroup

$RoleGroups |
    Select-Object Name, ManagedBy, RoleGroupType, Description |
    Sort-Object Name |
    Format-Table -AutoSize

$RoleGroups |
    Select-Object Name, ManagedBy, RoleGroupType, Description |
    Sort-Object Name |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Exchange_RoleGroups.csv")

# -----------------------------
# Exchange role group members
# -----------------------------
$ExchangeRoleGroupMembers = foreach ($RoleGroup in $RoleGroups) {
    $Members = Get-RoleGroupMember -Identity $RoleGroup.Name -ErrorAction SilentlyContinue

    foreach ($Member in $Members) {
        [PSCustomObject]@{
            RoleGroupName      = $RoleGroup.Name
            MemberName         = $Member.Name
            RecipientType      = $Member.RecipientType
            PrimarySmtpAddress = $Member.PrimarySmtpAddress
            DistinguishedName  = $Member.DistinguishedName
        }
    }
}

$ExchangeRoleGroupMembers |
    Sort-Object RoleGroupName, MemberName |
    Format-Table -AutoSize

$ExchangeRoleGroupMembers |
    Sort-Object RoleGroupName, MemberName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Exchange_RoleGroupMembers.csv")

# -----------------------------
# Critical Exchange role groups
# -----------------------------
$CriticalExchangeRoleGroups = @(
    "Organization Management",
    "Recipient Management",
    "View-Only Organization Management",
    "Compliance Management",
    "Discovery Management",
    "Hygiene Management",
    "Help Desk"
)

$CriticalExchangeRoleGroupMembers = $ExchangeRoleGroupMembers |
    Where-Object {$_.RoleGroupName -in $CriticalExchangeRoleGroups}

$CriticalExchangeRoleGroupMembers |
    Sort-Object RoleGroupName, MemberName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Exchange_CriticalRoleGroupMembers.csv")

Disconnect-ExchangeOnline -Confirm:$false
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Intune_Role_Review_Skeleton

```text
# ============================================================
# Intune Role Review Skeleton
# ============================================================

Portal:
https://intune.microsoft.com

Navigation:
Tenant administration > Roles

Review:
1. Built-in roles
2. Custom roles
3. Assignments
4. Scope groups
5. Scope tags
6. Members
7. Admin units if used
8. Device visibility limitations
9. App assignment limitations
10. Policy assignment limitations

Roles to review:
- Intune Administrator
- Policy and Profile Manager
- Application Manager
- Help Desk Operator
- Endpoint Security Manager
- Read Only Operator
- School Administrator if education tenant
- Custom roles

Documentation table:
| Role | Assignment Name | Members | Scope Groups | Scope Tags | Purpose | Risk |
|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |

Expected result:
Intune permissions are documented separately from Entra tenant roles and Azure RBAC.
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Defender_And_Purview_Role_Review_Skeleton

```text
# ============================================================
# Defender and Purview Role Review Skeleton
# ============================================================

Defender portal:
https://security.microsoft.com

Purview portal:
https://purview.microsoft.com

Defender areas to review:
1. Settings > Permissions
2. Microsoft Defender XDR role access
3. Incidents and alerts access
4. Hunting access
5. Device action permissions
6. Email and collaboration permissions if Defender for Office 365 is in scope
7. Unified RBAC status if enabled

Purview areas to review:
1. Settings > Roles and scopes
2. Role groups
3. Audit roles
4. eDiscovery roles
5. Compliance Administrator
6. Compliance Data Administrator
7. Data Loss Prevention roles
8. Records Management roles
9. Insider Risk roles if enabled
10. Communication Compliance roles if enabled

Documentation table:
| Portal | Role Group / Role | Members | Scope | Purpose | Risk |
|---|---|---|---|---|---|
| Defender |  |  |  |  |  |
| Purview |  |  |  |  |  |

Expected result:
Security and compliance roles are documented separately from Azure RBAC and general M365 admin roles.
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Graph_Permission_Comparison_Skeleton

```powershell
# ============================================================
# Microsoft Graph Permission Comparison Skeleton
# ============================================================

$TenantId = "<tenant-id>"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "Application.Read.All", `
    "AppRoleAssignment.ReadWrite.All"

# -----------------------------
# Current Graph delegated context
# -----------------------------
Get-MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

# -----------------------------
# App registrations
# -----------------------------
$Applications = Get-MgApplication -All -Property Id,AppId,DisplayName,RequiredResourceAccess

$Applications |
    Select-Object DisplayName, AppId, Id |
    Sort-Object DisplayName |
    Format-Table -AutoSize

$Applications |
    Select-Object DisplayName, AppId, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path ".\Graph_AppRegistrations.csv"

# -----------------------------
# Service principals
# -----------------------------
$ServicePrincipals = Get-MgServicePrincipal -All -Property Id,AppId,DisplayName,ServicePrincipalType,AccountEnabled

$ServicePrincipals |
    Select-Object DisplayName, AppId, ServicePrincipalType, AccountEnabled, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path ".\Graph_ServicePrincipals.csv"

# -----------------------------
# Note
# -----------------------------
# Graph permission analysis requires reviewing delegated scopes and application permissions.
# A user can have a Graph delegated scope but still fail if the signed-in user lacks the necessary directory role.
# An application permission can run without a signed-in user and should be treated as high privilege.
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Azure_CLI_RBAC_Skeleton

```bash
# ============================================================
# Azure CLI RBAC Inventory Skeleton
# ============================================================

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

# Confirm current context
az account show --output table

# List all role assignments at subscription scope
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --output table

# Export role assignments
mkdir -p Role_Plane_Comparison_Baseline

az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --output json > Role_Plane_Comparison_Baseline/azure_rbac_subscription_assignments.json

# List role definitions
az role definition list \
  --output json > Role_Plane_Comparison_Baseline/azure_rbac_role_definitions.json

# Filter Owners
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --query "[?roleDefinitionName=='Owner'].{principalName:principalName,principalType:principalType,role:roleDefinitionName,scope:scope}" \
  --output table

# Filter User Access Administrators
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --query "[?roleDefinitionName=='User Access Administrator'].{principalName:principalName,principalType:principalType,role:roleDefinitionName,scope:scope}" \
  --output table

# Filter Contributors
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --query "[?roleDefinitionName=='Contributor'].{principalName:principalName,principalType:principalType,role:roleDefinitionName,scope:scope}" \
  --output table
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Verification_Commands

```powershell
# ============================================================
# Verification Commands
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"

# Azure context
Get-AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Format-List

# Azure subscription
Get-AzSubscription -SubscriptionId $SubscriptionId |
    Select-Object Name, Id, TenantId, State |
    Format-List

# Azure RBAC at subscription scope
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Format-Table -AutoSize

# Azure Owners
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
    Where-Object {$_.RoleDefinitionName -eq "Owner"} |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Format-Table -AutoSize

# Azure User Access Administrators
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
    Where-Object {$_.RoleDefinitionName -eq "User Access Administrator"} |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Format-Table -AutoSize

# Graph context
Get-MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

# Active directory roles
Get-MgDirectoryRole -All |
    Select-Object DisplayName, Id |
    Sort-Object DisplayName |
    Format-Table -AutoSize

# Directory role members for critical roles
$CriticalRoles = @(
    "Global Administrator",
    "Privileged Role Administrator",
    "Conditional Access Administrator",
    "Security Administrator",
    "Compliance Administrator",
    "Exchange Administrator",
    "SharePoint Administrator",
    "Teams Administrator",
    "Intune Administrator"
)

$Roles = Get-MgDirectoryRole -All | Where-Object {$_.DisplayName -in $CriticalRoles}

foreach ($Role in $Roles) {
    Write-Host "Role: $($Role.DisplayName)"
    Get-MgDirectoryRoleMember -DirectoryRoleId $Role.Id -All |
        Select-Object Id, AdditionalProperties |
        Format-List
}

# Organization confirmation
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType |
    Format-List
```

```bash
# ============================================================
# Azure CLI Verification Commands
# ============================================================

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"

az account show --output table

az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --output table

az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --query "[?roleDefinitionName=='Owner']" \
  --output table

az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --query "[?roleDefinitionName=='User Access Administrator']" \
  --output table

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/directoryRoles" \
  --output json
```

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm workbook was inventory-only | Admin workstation | Review command history | No role changes were made |
| 2 | Disconnect Azure PowerShell | Admin workstation | Disconnect-AzAccount | Azure session is cleared |
| 3 | Clear Azure context if wrong tenant was used | Admin workstation | Clear-AzContext -Force | Cached context removed |
| 4 | Disconnect Microsoft Graph | Admin workstation | Disconnect-MgGraph | Graph session is closed |
| 5 | Logout Azure CLI | Admin workstation | az logout | Azure CLI session is closed |
| 6 | Disconnect Exchange Online if used | Admin workstation | Disconnect-ExchangeOnline -Confirm:$false | Exchange session is closed |
| 7 | Remove exported evidence if sensitive | Admin workstation | Remove-Item ".\Role_Plane_Comparison_Baseline" -Recurse -Force | Local exports removed |
| 8 | If a role was accidentally assigned in Azure RBAC | Admin workstation | Remove-AzRoleAssignment | Role assignment removed only after validation |
| 9 | If an Entra role was accidentally assigned | Admin workstation | Remove-MgDirectoryRoleMemberByRef or portal removal | Directory role assignment removed only after validation |
| 10 | If a workload role was accidentally assigned | Admin workstation | Workload portal or workload cmdlet | Workload role assignment reverted |
| 11 | Document accidental changes if any | Admin workstation | Workbook notes | Audit trail is preserved |

## Rollback Notes

This workbook is intended to compare and inventory role systems.

Do not remove role assignments just because they appear privileged.

Role cleanup requires a separate change record and approval because removing the wrong assignment can break:

- Production administration
- Emergency access
- Billing administration
- Security operations
- Compliance operations
- Mail flow administration
- Endpoint management
- Automation identity access
- CI/CD deployment access
- Break glass access

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Failure_Checks

| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| User is Global Administrator but cannot see Azure subscription | Entra role does not grant Azure RBAC | Admin workstation | Get-AzSubscription | Assign Reader/Owner/Contributor at Azure scope as approved |
| User is Azure Owner but cannot manage users | Azure RBAC does not grant Entra directory role | Admin workstation | Get-MgUser | Assign suitable Entra role as approved |
| User can view Azure resources but cannot assign roles | Contributor lacks RBAC delegation rights | Admin workstation | Get-AzRoleAssignment | Use Owner or User Access Administrator if approved |
| User can manage users but not licenses | User Administrator does not always grant license management | Admin workstation | M365 admin center role review | Assign License Administrator if approved |
| User can view M365 admin center but not Exchange admin center | Missing Exchange role | Admin workstation | Exchange role group review | Assign Exchange Administrator or required role group |
| Exchange PowerShell connects but cmdlet fails | Missing Exchange role group permission | Admin workstation | Get-RoleGroupMember | Add required role group membership if approved |
| SharePoint admin center blocked | Missing SharePoint Administrator role | Admin workstation | Entra role inventory | Assign SharePoint Administrator if approved |
| Teams admin center blocked | Missing Teams Administrator role | Admin workstation | Entra role inventory | Assign Teams role if approved |
| Intune portal opens but devices hidden | Intune scope tags or RBAC scope groups limit visibility | Admin workstation | Intune portal role review | Review Intune role assignment scope |
| Defender portal limited | Missing Security Reader/Admin or unified RBAC assignment | Admin workstation | Defender portal permissions | Assign correct security role if approved |
| Purview portal limited | Missing Purview role group | Admin workstation | Purview role group review | Assign correct Purview role group if approved |
| Get-AzRoleAssignment fails | Missing Azure RBAC read permission | Admin workstation | Get-AzContext | Request Reader at scope or use account with permission |
| Get-MgDirectoryRole fails | Missing Graph scope or directory read permission | Admin workstation | Get-MgContext | Reconnect with RoleManagement.Read.Directory and Directory.Read.All |
| Directory role member output lacks display names | Graph returns directoryObject with additional properties only | Admin workstation | Use AdditionalProperties fields | Expand or query object by ID |
| PIM eligible roles do not show in active directory roles | Eligible role is not active | Admin workstation | Entra PIM portal | Review eligible assignments separately |
| Role appears inherited unexpectedly | Assignment inherited from parent Azure scope | Admin workstation | Check Scope field | Trace management group or subscription inheritance |
| Admin removed role and lost access | Role cleanup done without break glass/change plan | Admin workstation | Break glass account / another admin | Restore role using approved emergency process |
| Graph scope granted but command still fails | Delegated Graph scope is not enough without user role | Admin workstation | Get-MgContext and role inventory | Assign required directory role or use correct account |
| App-only automation has too much access | Broad application permission granted | Admin workstation | App registration API permissions | Reduce app permissions and use least privilege |
| Role assignment belongs to group but user cannot act | Group membership not active, dynamic group delay, PIM group not activated | Admin workstation | Check group membership and PIM | Activate group or wait for membership evaluation |
| Portal and PowerShell disagree | Wrong tenant, stale session, or different account | Admin workstation | Get-AzContext; Get-MgContext; az account show | Align account, tenant, and subscription |
| Role assignment cannot be removed | Assignment inherited or managed by group/PIM | Admin workstation | Check scope and assignment source | Remove at source scope or group/PIM assignment |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Documentation_Output_TEMPLATE

## Role Baseline Summary

| Field | Value |
|---|---|
| Tenant display name |  |
| Tenant ID |  |
| Subscription name |  |
| Subscription ID |  |
| Management group reviewed | Yes / No |
| PIM reviewed | Yes / No |
| Date collected |  |
| Admin account used |  |
| Output path |  |

## Azure RBAC Baseline

| Principal | Principal Type | Role | Scope | Inherited | Risk | Notes |
|---|---|---|---|---|---|---|
|  |  | Owner |  | Yes / No | Critical |  |
|  |  | User Access Administrator |  | Yes / No | Critical |  |
|  |  | Contributor |  | Yes / No | High |  |
|  |  | Reader |  | Yes / No | Low |  |

## Entra Directory Role Baseline

| Principal | Principal Type | Role | Scope | Active / Eligible | Risk | Notes |
|---|---|---|---|---|---|---|
|  |  | Global Administrator | Tenant |  | Critical |  |
|  |  | Privileged Role Administrator | Tenant |  | Critical |  |
|  |  | Conditional Access Administrator | Tenant |  | High |  |
|  |  | User Administrator | Tenant / AU |  | Medium / High |  |
|  |  | Groups Administrator | Tenant / AU |  | Medium |  |

## Microsoft 365 Admin Role Baseline

| Principal | Role | Service Area | Scope | Risk | Notes |
|---|---|---|---|---|---|
|  | Exchange Administrator | Exchange Online | Tenant | High |  |
|  | SharePoint Administrator | SharePoint / OneDrive | Tenant | High |  |
|  | Teams Administrator | Teams | Tenant | Medium / High |  |
|  | Intune Administrator | Endpoint Management | Tenant | High |  |
|  | Security Administrator | Defender / Security | Tenant | High |  |
|  | Compliance Administrator | Purview / Compliance | Tenant | High |  |
|  | Billing Administrator | Billing | Tenant | Medium / High |  |
|  | License Administrator | Licensing | Tenant | Medium |  |

## Workload Role Baseline

| Workload | Role / Role Group | Members | Scope | Notes |
|---|---|---|---|---|
| Exchange Online | Organization Management |  | Exchange org |  |
| Exchange Online | Recipient Management |  | Exchange org |  |
| SharePoint | SharePoint Administrator |  | Tenant |  |
| SharePoint | Site Collection Administrator |  | Site |  |
| Teams | Teams Administrator |  | Tenant |  |
| Intune | Intune role assignment |  | Scope tags/groups |  |
| Defender | Security role / unified RBAC |  | Defender scope |  |
| Purview | Compliance role group |  | Compliance scope |  |

## Role Plane Decision Table

| Task | Correct Role Plane | Current Admin Has Access | Missing Role | Notes |
|---|---|---|---|---|
| Assign Azure RBAC | Azure RBAC | Yes / No | Owner or User Access Administrator |  |
| Manage users | Entra roles | Yes / No | User Administrator |  |
| Manage groups | Entra roles | Yes / No | Groups Administrator |  |
| Manage Conditional Access | Entra roles | Yes / No | Conditional Access Administrator |  |
| Manage Exchange mail flow | Exchange workload roles | Yes / No | Exchange Administrator / role group |  |
| Manage SharePoint sharing | SharePoint workload role | Yes / No | SharePoint Administrator |  |
| Manage Teams policies | Teams workload role | Yes / No | Teams Administrator |  |
| Manage Intune compliance | Intune role | Yes / No | Intune Administrator / scoped role |  |
| Manage Defender incidents | Security role | Yes / No | Security Operator / Security Administrator |  |
| Manage Purview DLP | Compliance role | Yes / No | Compliance Administrator / DLP role |  |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories | Supplies tenant, directory, subscription, and namespace context |
| 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline | Supplies tooling and authenticated contexts for role inventory |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces | Shows domain role requirements and tenant namespace control |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline | Uses this role comparison to decide privileged admin model |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline | Depends on knowing User Administrator, Groups Administrator, License Administrator, and authentication roles |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking | Depends on knowing audit, service health, and monitoring role requirements |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Uses role plane comparison to troubleshoot blocked portals and commands |
| Azure Governance RBAC Workbook | Deep dive for management groups, subscriptions, policy, locks, and Azure RBAC |
| Microsoft 365 Tenant Administration Workbook | Deep dive for Microsoft 365 tenant admin roles |
| Exchange Online Administration Workbook | Deep dive for Exchange role groups and RBAC |
| SharePoint OneDrive Workbook | Deep dive for SharePoint admin and site permissions |
| Teams Administration Workbook | Deep dive for Teams admin roles and Teams ownership |
| Intune Workbook | Deep dive for Intune RBAC, scope groups, and scope tags |
| Defender XDR Workbook | Deep dive for security roles and Defender portal access |
| Purview Workbook | Deep dive for compliance role groups and Purview permissions |
| Entra Enterprise Apps Workbook | Deep dive for app admin roles, app registrations, enterprise apps, and Graph consent |

---

# 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles_Completion_Criteria

| Requirement | Complete |
|---|---|
| Target tenant confirmed |  |
| Target subscription confirmed |  |
| Azure PowerShell context verified |  |
| Microsoft Graph context verified |  |
| Azure CLI context verified if used |  |
| Azure RBAC subscription assignments exported |  |
| Azure Owner assignments reviewed |  |
| Azure User Access Administrator assignments reviewed |  |
| Azure Contributor assignments reviewed |  |
| Management group RBAC reviewed if visible |  |
| Entra directory roles exported |  |
| Entra directory role members exported |  |
| Global Administrators reviewed |  |
| Privileged Role Administrators reviewed |  |
| Conditional Access Administrators reviewed |  |
| M365 admin roles reviewed |  |
| Exchange role groups reviewed if in scope |  |
| SharePoint admin roles reviewed if in scope |  |
| Teams admin roles reviewed if in scope |  |
| Intune roles and scope tags reviewed if in scope |  |
| Defender roles reviewed if in scope |  |
| Purview role groups reviewed if in scope |  |
| Graph delegated scope concept documented |  |
| Graph application permission concept documented |  |
| PIM eligible vs active status documented if available |  |
| Overprivileged assignments flagged |  |
| Missing role assignments flagged |  |
| Role plane decision table completed |  |
| Evidence output stored securely |  |
| No role changes made without approval |  |

## Final Expected State

The admin can clearly state:

- Azure RBAC controls Azure resources.
- Entra roles control identity and directory administration.
- Microsoft 365 admin roles control Microsoft 365 service administration.
- Workload roles and role groups control specific services like Exchange, SharePoint, Teams, Intune, Defender, and Purview.
- Graph permissions control API access but do not replace user role requirements.
- Global Administrator and Azure subscription Owner are not the same role.
- User Access Administrator and Privileged Role Administrator are both high-risk delegation roles, but they operate in different planes.
- The organization has a documented baseline of who holds critical roles and where those roles apply.