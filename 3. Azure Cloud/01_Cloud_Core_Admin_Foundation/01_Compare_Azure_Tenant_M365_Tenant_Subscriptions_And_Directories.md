# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Index

## Purpose

Build a baseline comparison between:

- Microsoft Entra tenant
- Azure tenant / directory
- Microsoft 365 tenant
- Azure subscriptions
- Azure directories
- Verified domains
- Admin portals
- Identity objects
- Licensing boundaries
- RBAC boundaries

This workbook is for understanding and documenting the control-plane relationship before configuring users, licenses, roles, policies, domains, or resources.

## Outcome

By the end of this workbook, you should be able to answer:

- Which Microsoft Entra tenant backs the Azure portal?
- Which Microsoft Entra tenant backs the Microsoft 365 admin center?
- Which domains are verified in the tenant?
- Which Azure subscriptions trust the directory?
- Which users and groups exist in the tenant?
- Which admin roles apply to Microsoft 365 workloads?
- Which Azure RBAC assignments apply to Azure resources?
- Which portal should be used for each administrative task?
- Whether Azure and Microsoft 365 are sharing the same identity boundary or split across different tenants.

## Workbook Sections

- Source_Basis
- Mental_Model
- Environment_Assumptions
- Planning_Table
- Tenant_Subscription_Directory_Comparison_Table
- Portal_Comparison_Table
- Configuration_Checklist
- Azure_Portal_Review_Skeleton
- Microsoft_365_Admin_Center_Review_Skeleton
- Entra_Admin_Center_Review_Skeleton
- PowerShell_Inventory_Skeleton
- Azure_CLI_Inventory_Skeleton
- Microsoft_Graph_Inventory_Skeleton
- Verification_Commands
- Rollback
- Failure_Checks
- Related_Labs

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Source_Basis

| Source | Relevant Area | What It Supports |
|---|---|---|
| Microsoft 365 documentation fork | Microsoft 365 tenant, admin center, domains, users, groups, service health, Message Center | M365 admin boundary and workload administration |
| Microsoft Entra identity documentation | Tenant ID, directory ID, users, groups, guests, roles, authentication, SSPR, Conditional Access | Identity plane and directory objects |
| Azure management and governance documentation | Azure subscriptions, resource groups, management groups, RBAC, activity logs, Azure Policy | Azure resource control plane |
| Azure DNS documentation | Custom domain verification, DNS zones, TXT/MX/CNAME records, delegation concepts | Domain and DNS validation planning |
| Microsoft Graph documentation | Tenant, user, group, organization, subscribed SKU, directory role, service principal inventory | Cross-service API inventory |
| Azure PowerShell documentation | Get-AzTenant, Get-AzSubscription, Get-AzContext, Az.Resources cmdlets | Azure inventory and validation |
| Microsoft Graph PowerShell documentation | Get-MgOrganization, Get-MgUser, Get-MgGroup, Get-MgSubscribedSku | M365 / Entra tenant inventory |
| Existing Workbook structure | Source basis, mental model, planning, checklist, skeletons, verification, rollback, failure checks | Obsidian-ready workbook layout |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Mental_Model

| Concept | Plain Meaning | Admin Boundary |
|---|---|---|
| Microsoft Entra tenant | The identity directory that stores users, groups, apps, roles, and policies | Identity plane |
| Azure directory | The Entra tenant selected in the Azure portal | Identity plane for Azure |
| Microsoft 365 tenant | The Microsoft 365 service tenant backed by Entra ID | Productivity service plane |
| Azure subscription | Billing and resource container that trusts one Entra tenant | Azure resource plane |
| Management group | Azure hierarchy object above subscriptions | Azure governance plane |
| Resource group | Logical container for Azure resources inside a subscription | Azure resource scope |
| Azure RBAC | Permissions for Azure resources | Azure resource plane |
| Entra roles | Permissions for identity and directory administration | Identity plane |
| Microsoft 365 admin roles | Permissions for M365 workloads like Exchange, SharePoint, Teams, compliance, billing | M365 service plane |
| Workload role group | Service-specific admin role model, such as Exchange role groups | Workload plane |
| Verified domain | Public DNS name proven to belong to the tenant | Identity and mail namespace |
| Initial domain | Default tenant domain, usually tenantname.onmicrosoft.com | Permanent tenant namespace |
| Primary domain | Default domain used for new users and addresses | Identity and mail naming |
| Tenant ID | GUID for the Entra tenant | Stable tenant identifier |
| Subscription ID | GUID for an Azure subscription | Azure billing and resource identifier |
| Directory ID | Same practical value as tenant ID in most admin contexts | Identity identifier |
| Object ID | GUID for a specific user, group, app, service principal, or role assignment | Object identifier |

## Core Rule

Azure resources live in subscriptions.

Subscriptions trust an Entra tenant.

Microsoft 365 services also use an Entra tenant.

If Azure and Microsoft 365 point to the same tenant, identity administration is unified.

If they point to different tenants, identity, roles, domains, licenses, and access must be documented separately.

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Environment_Assumptions

| Item | Lab Example | Production Example |
|---|---|---|
| Tenant display name | Contoso Lab | Company legal / operating name |
| Initial domain | contosolab.onmicrosoft.com | companyname.onmicrosoft.com |
| Custom domain | lab.contoso.com | contoso.com |
| Azure subscription | Azure Lab Subscription | Production / Dev / Shared Services subscriptions |
| M365 tenant | Same as lab tenant | Microsoft 365 commercial tenant |
| Admin account | cloudadmin@contosolab.onmicrosoft.com | named admin account or privileged role account |
| Break glass account | breakglass01@contosolab.onmicrosoft.com | excluded emergency access account |
| PowerShell workstation | Admin workstation | Privileged access workstation |
| Expected access | Global Reader, Global Administrator, Subscription Reader, Owner, or User Access Administrator | Role depends on task |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Planning_Table

| Item | Value To Record | Why It Matters |
|---|---|---|
| Entra tenant display name |  | Confirms which identity directory you are viewing |
| Tenant ID |  | Required for Graph, Azure CLI, PowerShell, app registrations, and support cases |
| Initial onmicrosoft.com domain |  | Permanent namespace that cannot be removed |
| Primary verified domain |  | Default sign-in and mail namespace |
| All verified domains |  | Determines available UPN and email namespaces |
| M365 tenant display name |  | Confirms Microsoft 365 tenant identity |
| Azure subscription names |  | Shows Azure resource containers |
| Azure subscription IDs |  | Required for Azure CLI, PowerShell, billing, support, and RBAC |
| Subscription tenant association |  | Confirms which Entra tenant controls access |
| Management group hierarchy |  | Shows governance scope above subscriptions |
| Current Azure context |  | Prevents running commands in the wrong tenant or subscription |
| Current Graph context |  | Prevents reading or changing the wrong tenant |
| Global administrators |  | Identifies high privilege identity admins |
| Privileged Role Administrators |  | Identifies role assignment admins |
| Billing administrators |  | Identifies license and billing admins |
| Azure Owners |  | Identifies high privilege Azure resource admins |
| User Access Administrators |  | Identifies Azure RBAC delegation admins |
| Licensed user count |  | Shows Microsoft 365 service consumption |
| Guest user count |  | Shows external identity exposure |
| Service principals |  | Shows app and automation identity footprint |
| Conditional Access baseline |  | Shows access control posture |
| Audit log availability |  | Shows investigation capability |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Tenant_Subscription_Directory_Comparison_Table

| Object | Exists In | Controlled From | Identifier | Main Admin Model | Common Mistake |
|---|---|---|---|---|---|
| Entra tenant | Microsoft Entra ID | Entra admin center, Azure portal, Graph | Tenant ID | Entra roles | Treating it as only an Azure thing |
| Azure directory | Azure portal directory selector | Azure portal, Entra admin center | Tenant ID / Directory ID | Entra roles | Thinking every subscription has its own directory |
| M365 tenant | Microsoft 365 admin center | M365 admin center, Entra admin center, workload portals | Tenant ID | M365 admin roles and Entra roles | Thinking M365 users are separate from Entra users |
| Azure subscription | Azure portal | Azure portal, Azure CLI, Azure PowerShell | Subscription ID | Azure RBAC | Confusing subscription Owner with Global Administrator |
| Management group | Azure portal | Azure portal, Azure CLI, Azure PowerShell | Management group ID | Azure RBAC | Missing inherited policy or RBAC |
| Resource group | Azure subscription | Azure portal, ARM/Bicep, CLI, PowerShell | Resource group name/resource ID | Azure RBAC | Assigning permissions too broadly |
| Verified domain | Entra / M365 | M365 admin center, Entra admin center, DNS provider | DNS domain name | Domain admin permissions | Assuming DNS verification equals mail readiness |
| User | Entra ID | Entra admin center, M365 admin center, Graph | User object ID | Entra roles / M365 roles | Confusing UPN, email, proxy address, and object ID |
| Group | Entra ID | Entra admin center, M365 admin center, Graph | Group object ID | Group ownership and directory roles | Confusing security groups, M365 groups, and distribution groups |
| License SKU | M365 commerce | M365 admin center, Graph | SKU ID / SKU part number | Billing/license admin | Confusing license assignment with resource access |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Portal_Comparison_Table

| Portal | URL | Primary Use | What To Confirm |
|---|---|---|---|
| Azure portal | https://portal.azure.com | Azure resources, subscriptions, management groups, Azure RBAC, policy, monitoring | Directory, subscription, resource scope |
| Microsoft Entra admin center | https://entra.microsoft.com | Users, groups, roles, authentication, Conditional Access, enterprise apps | Tenant ID, users, roles, authentication policies |
| Microsoft 365 admin center | https://admin.microsoft.com | M365 users, licenses, domains, billing, service health, Message Center | Tenant, domains, licenses, service health |
| Exchange admin center | https://admin.exchange.microsoft.com | Mailboxes, mail flow, accepted domains, transport rules | Mail domain and recipient readiness |
| SharePoint admin center | https://admin.microsoft.com/sharepoint | SharePoint sites, OneDrive, sharing policies | Storage, sharing, site ownership |
| Teams admin center | https://admin.teams.microsoft.com | Teams policies, meetings, calling, apps, users | Collaboration policy boundary |
| Microsoft Purview portal | https://purview.microsoft.com | Compliance, retention, audit, eDiscovery, DLP | Compliance and audit posture |
| Microsoft Defender portal | https://security.microsoft.com | Defender, incidents, secure score, device/email/app security | Security operations view |
| Azure Cost Management | Azure portal | Budgets, cost, invoices, forecasts, Advisor | Subscription cost ownership |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Sign in to Azure portal | Admin workstation | Browser: https://portal.azure.com | Azure portal opens with current directory shown |
| 2 | Record selected Azure directory | Admin workstation | Azure portal > Settings > Directories + subscriptions | Current directory and tenant are identified |
| 3 | Record tenant ID | Admin workstation | Entra admin center > Overview | Tenant ID is captured |
| 4 | Record primary domain | Admin workstation | Entra admin center > Identity > Settings > Domain names | Primary domain is captured |
| 5 | Record all verified domains | Admin workstation | Microsoft 365 admin center > Settings > Domains | All verified domains are documented |
| 6 | Review Azure subscriptions | Admin workstation | Azure portal > Subscriptions | Subscription names and IDs are captured |
| 7 | Verify subscription directory association | Admin workstation | Azure portal > Subscriptions > select subscription > Properties | Subscription tenant association is confirmed |
| 8 | Review management groups | Admin workstation | Azure portal > Management groups | Hierarchy above subscriptions is documented |
| 9 | Review Azure RBAC at subscription scope | Admin workstation | Subscription > Access control IAM > Role assignments | Owner, Contributor, Reader, User Access Administrator are captured |
| 10 | Review Entra admin roles | Admin workstation | Entra admin center > Roles & admins | Global Administrator and privileged role holders are captured |
| 11 | Review M365 admin roles | Admin workstation | Microsoft 365 admin center > Roles | M365 role assignments are captured |
| 12 | Review licensed users | Admin workstation | Microsoft 365 admin center > Billing > Licenses | Available and assigned licenses are documented |
| 13 | Review guest users | Admin workstation | Entra admin center > Users > User type: Guest | Guest user count and exposure are documented |
| 14 | Review applications and service principals | Admin workstation | Entra admin center > Applications | App registrations and enterprise apps are identified |
| 15 | Connect Azure PowerShell | Admin workstation | Connect-AzAccount | Azure PowerShell context is established |
| 16 | Confirm Azure PowerShell context | Admin workstation | Get-AzContext | Current account, tenant, and subscription are visible |
| 17 | Inventory tenants with Azure PowerShell | Admin workstation | Get-AzTenant | Accessible tenants are listed |
| 18 | Inventory subscriptions with Azure PowerShell | Admin workstation | Get-AzSubscription | Accessible subscriptions are listed |
| 19 | Connect Microsoft Graph PowerShell | Admin workstation | Connect-MgGraph -Scopes "Organization.Read.All","User.Read.All","Group.Read.All","Directory.Read.All","RoleManagement.Read.Directory" | Graph context is established |
| 20 | Confirm Graph tenant organization | Admin workstation | Get-MgOrganization | Tenant display name, ID, and verified domains are visible |
| 21 | Export tenant comparison data | Admin workstation | Run inventory skeleton | Tenant/subscription/domain output is exported |
| 22 | Compare portal findings against CLI output | Admin workstation | Manual review | No mismatch between portals and command output |
| 23 | Document source gaps | Admin workstation | Workbook notes | Any missing permission, tenant, or subscription visibility is recorded |
| 24 | Save final baseline | Admin workstation | Commit note to workbook repo or Obsidian vault | Comparison workbook becomes the baseline reference |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Azure_Portal_Review_Skeleton

## Azure Portal Review

| Check | Navigation | Value |
|---|---|---|
| Current directory | Azure portal > Settings > Directories + subscriptions |  |
| Tenant ID | Microsoft Entra ID > Overview |  |
| Subscription name | Subscriptions > selected subscription > Overview |  |
| Subscription ID | Subscriptions > selected subscription > Properties |  |
| Subscription tenant | Subscriptions > selected subscription > Properties |  |
| Management group path | Management groups |  |
| Current user role | Subscription > Access control IAM > View my access |  |
| Subscription Owners | Subscription > Access control IAM > Role assignments |  |
| User Access Administrators | Subscription > Access control IAM > Role assignments |  |
| Resource groups | Subscription > Resource groups |  |
| Azure Policy assignments | Subscription > Policy |  |
| Activity log available | Subscription > Activity log |  |
| Cost Management enabled | Subscription > Cost Management |  |

## Notes

- Azure portal directory selector controls which Entra tenant you are viewing.
- Azure subscription access is controlled by Azure RBAC.
- Directory administration is controlled by Entra roles.
- A user can be Global Administrator without being Azure subscription Owner.
- A user can be Azure subscription Owner without being Global Administrator.

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Microsoft_365_Admin_Center_Review_Skeleton

## Microsoft 365 Admin Center Review

| Check | Navigation | Value |
|---|---|---|
| Tenant display name | Microsoft 365 admin center > Settings > Org settings > Organization profile |  |
| Default domain | Settings > Domains |  |
| Verified domains | Settings > Domains |  |
| User count | Users > Active users |  |
| Deleted users | Users > Deleted users |  |
| Guest users | Users > Guest users or Entra admin center |  |
| License SKUs | Billing > Licenses |  |
| Assigned licenses | Billing > Licenses |  |
| Admin roles | Roles > Role assignments |  |
| Service health | Health > Service health |  |
| Message Center | Health > Message center |  |
| Org sharing posture | Settings > Org settings |  |
| Support contact | Settings > Org settings > Organization profile |  |

## Notes

- Microsoft 365 admin center focuses on productivity tenant administration.
- M365 users are Entra users with licenses and workload access.
- Microsoft 365 licenses do not grant Azure resource permissions.
- Microsoft 365 admin roles do not automatically grant Azure subscription permissions.
- Domains used for M365 mail still require DNS records beyond tenant verification.

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Entra_Admin_Center_Review_Skeleton

## Entra Admin Center Review

| Check | Navigation | Value |
|---|---|---|
| Tenant name | Entra admin center > Overview |  |
| Tenant ID | Entra admin center > Overview |  |
| Primary domain | Identity > Settings > Domain names |  |
| Initial domain | Identity > Settings > Domain names |  |
| All domains | Identity > Settings > Domain names |  |
| Users | Identity > Users > All users |  |
| Groups | Identity > Groups > All groups |  |
| Guest users | Identity > Users > All users > User type Guest |  |
| Deleted users | Identity > Users > Deleted users |  |
| Roles | Identity > Roles & admins |  |
| Global Administrators | Roles & admins > Global Administrator > Assignments |  |
| Privileged Role Administrators | Roles & admins > Privileged Role Administrator > Assignments |  |
| Authentication methods | Protection > Authentication methods |  |
| Conditional Access | Protection > Conditional Access |  |
| App registrations | Applications > App registrations |  |
| Enterprise applications | Applications > Enterprise applications |  |
| Audit logs | Monitoring > Audit logs |  |
| Sign-in logs | Monitoring > Sign-in logs |  |

## Notes

- Entra tenant is the identity authority.
- Azure and Microsoft 365 both depend on Entra identities.
- Entra roles control tenant and identity administration.
- Azure RBAC controls Azure resource access.
- Workload admin roles may exist inside Microsoft 365 service portals.

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_PowerShell_Inventory_Skeleton

```powershell
# ============================================================
# 01 Compare Azure Tenant, M365 Tenant, Subscriptions, Directories
# PowerShell Inventory Skeleton
# ============================================================

# -----------------------------
# Variables
# -----------------------------
$OutputRoot = ".\Tenant_Subscription_Directory_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

# -----------------------------
# Connect to Azure
# -----------------------------
Connect-AzAccount

# -----------------------------
# Confirm Azure Context
# -----------------------------
$AzContext = Get-AzContext

$AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Format-List

$AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzContext.csv")

# -----------------------------
# Inventory Azure Tenants
# -----------------------------
$AzTenants = Get-AzTenant

$AzTenants |
    Select-Object Id, Name, Category, Domains |
    Format-Table -AutoSize

$AzTenants |
    Select-Object Id, Name, Category, Domains |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzTenants.csv")

# -----------------------------
# Inventory Azure Subscriptions
# -----------------------------
$AzSubscriptions = Get-AzSubscription

$AzSubscriptions |
    Select-Object Name, Id, TenantId, State |
    Sort-Object Name |
    Format-Table -AutoSize

$AzSubscriptions |
    Select-Object Name, Id, TenantId, State |
    Sort-Object Name |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzSubscriptions.csv")

# -----------------------------
# Inventory Current Subscription RBAC
# -----------------------------
$CurrentSubscriptionId = (Get-AzContext).Subscription.Id

$RoleAssignments = Get-AzRoleAssignment -Scope "/subscriptions/$CurrentSubscriptionId"

$RoleAssignments |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Format-Table -AutoSize

$RoleAssignments |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzRoleAssignments_CurrentSubscription.csv")

# -----------------------------
# Inventory Azure Management Groups
# Requires permission to read management groups
# -----------------------------
try {
    $ManagementGroups = Get-AzManagementGroup

    $ManagementGroups |
        Select-Object Name, DisplayName, Id, TenantId |
        Format-Table -AutoSize

    $ManagementGroups |
        Select-Object Name, DisplayName, Id, TenantId |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzManagementGroups.csv")
}
catch {
    Write-Warning "Could not read management groups. Missing permission or module support."
    $_.Exception.Message | Out-File -FilePath (Join-Path $OutputPath "AzManagementGroups_Error.txt")
}

# -----------------------------
# Connect to Microsoft Graph
# -----------------------------
Connect-MgGraph -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "User.Read.All", `
    "Group.Read.All", `
    "RoleManagement.Read.Directory"

# -----------------------------
# Confirm Graph Context
# -----------------------------
$MgContext = Get-MgContext

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgContext.csv")

# -----------------------------
# Inventory Organization / Tenant
# -----------------------------
$Organization = Get-MgOrganization

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgOrganization.csv")

# -----------------------------
# Inventory Verified Domains
# -----------------------------
$VerifiedDomains = $Organization.VerifiedDomains

$VerifiedDomains |
    Select-Object Name, Type, IsDefault, IsInitial, Capabilities |
    Sort-Object IsDefault -Descending, Name |
    Format-Table -AutoSize

$VerifiedDomains |
    Select-Object Name, Type, IsDefault, IsInitial, Capabilities |
    Sort-Object IsDefault -Descending, Name |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgVerifiedDomains.csv")

# -----------------------------
# Inventory Users
# -----------------------------
$Users = Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,CreatedDateTime

$Users |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgUsers.csv")

# -----------------------------
# Inventory Groups
# -----------------------------
$Groups = Get-MgGroup -All -Property Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes,Mail

$Groups |
    Select-Object DisplayName, Mail, MailEnabled, SecurityEnabled, GroupTypes, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgGroups.csv")

# -----------------------------
# Inventory Subscribed SKUs
# -----------------------------
$SubscribedSkus = Get-MgSubscribedSku -All

$SubscribedSkus |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="SuspendedUnits";Expression={$_.PrepaidUnits.Suspended}},
        @{Name="WarningUnits";Expression={$_.PrepaidUnits.Warning}} |
    Sort-Object SkuPartNumber |
    Format-Table -AutoSize

$SubscribedSkus |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="SuspendedUnits";Expression={$_.PrepaidUnits.Suspended}},
        @{Name="WarningUnits";Expression={$_.PrepaidUnits.Warning}} |
    Sort-Object SkuPartNumber |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgSubscribedSkus.csv")

# -----------------------------
# Inventory Directory Roles
# -----------------------------
$DirectoryRoles = Get-MgDirectoryRole -All

$DirectoryRoles |
    Select-Object DisplayName, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgDirectoryRoles.csv")

# -----------------------------
# Summary Output
# -----------------------------
$Summary = [PSCustomObject]@{
    DateCollected            = Get-Date
    AzureAccount             = $AzContext.Account.Id
    AzureTenantId            = $AzContext.Tenant.Id
    AzureSubscriptionName    = $AzContext.Subscription.Name
    AzureSubscriptionId      = $AzContext.Subscription.Id
    GraphAccount             = $MgContext.Account
    GraphTenantId            = $MgContext.TenantId
    OrganizationDisplayName  = $Organization.DisplayName
    OrganizationId           = $Organization.Id
    UserCount                = $Users.Count
    GuestUserCount           = ($Users | Where-Object {$_.UserType -eq "Guest"}).Count
    GroupCount               = $Groups.Count
    VerifiedDomainCount      = $VerifiedDomains.Count
    AzureSubscriptionCount   = $AzSubscriptions.Count
    AzureTenantCount         = $AzTenants.Count
}

$Summary |
    Format-List

$Summary |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Tenant_Subscription_Directory_Summary.csv")

Write-Host "Inventory complete. Output path: $OutputPath"
```

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Azure_CLI_Inventory_Skeleton

```bash
# ============================================================
# 01 Compare Azure Tenant, M365 Tenant, Subscriptions, Directories
# Azure CLI Inventory Skeleton
# ============================================================

# Login
az login

# Show current account context
az account show --output table

# List available tenants
az account tenant list --output table

# List subscriptions available to signed-in account
az account list --output table

# Set target subscription
az account set --subscription "<subscription-id-or-name>"

# Confirm selected subscription
az account show --query "{name:name, id:id, tenantId:tenantId, user:user.name}" --output table

# List management groups
az account management-group list --output table

# List role assignments at subscription scope
az role assignment list \
  --scope "/subscriptions/<subscription-id>" \
  --include-inherited \
  --output table

# List current signed-in user details
az ad signed-in-user show \
  --query "{displayName:displayName,userPrincipalName:userPrincipalName,id:id}" \
  --output table

# List domains visible through Azure CLI
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains" \
  --output table

# List organization details through Microsoft Graph
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json

# List subscribed SKUs through Microsoft Graph
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --output table
```

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Microsoft_Graph_Inventory_Skeleton

```powershell
# ============================================================
# Microsoft Graph Inventory Skeleton
# ============================================================

Connect-MgGraph -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "Domain.Read.All", `
    "User.Read.All", `
    "Group.Read.All", `
    "RoleManagement.Read.Directory"

# Organization / tenant
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType

# Verified domains
(Get-MgOrganization).VerifiedDomains |
    Select-Object Name, Type, IsDefault, IsInitial, Capabilities |
    Sort-Object IsDefault -Descending, Name

# Users
Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, Id |
    Sort-Object UserPrincipalName

# Groups
Get-MgGroup -All -Property Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes |
    Select-Object DisplayName, Mail, MailEnabled, SecurityEnabled, GroupTypes, Id |
    Sort-Object DisplayName

# Subscribed license SKUs
Get-MgSubscribedSku -All |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="SuspendedUnits";Expression={$_.PrepaidUnits.Suspended}},
        @{Name="WarningUnits";Expression={$_.PrepaidUnits.Warning}}

# Directory roles
Get-MgDirectoryRole -All |
    Select-Object DisplayName, Id |
    Sort-Object DisplayName

# Role templates
Get-MgDirectoryRoleTemplate -All |
    Select-Object DisplayName, Id |
    Sort-Object DisplayName
```

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Verification_Commands

```powershell
# ============================================================
# Verification Commands
# ============================================================

# Verify Azure PowerShell context
Get-AzContext

# Verify Azure tenant
Get-AzTenant | Select-Object Id, Name, Domains

# Verify Azure subscriptions
Get-AzSubscription | Select-Object Name, Id, TenantId, State

# Verify current subscription
(Get-AzContext).Subscription

# Verify subscription RBAC at current subscription scope
$currentSub = (Get-AzContext).Subscription.Id
Get-AzRoleAssignment -Scope "/subscriptions/$currentSub" |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName

# Verify Microsoft Graph context
Get-MgContext

# Verify Microsoft 365 / Entra organization
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType

# Verify domains
(Get-MgOrganization).VerifiedDomains |
    Select-Object Name, Type, IsDefault, IsInitial, Capabilities |
    Sort-Object IsDefault -Descending, Name

# Verify subscribed SKUs
Get-MgSubscribedSku -All |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}}

# Verify user and guest counts
$users = Get-MgUser -All -Property Id,UserType
[PSCustomObject]@{
    TotalUsers = $users.Count
    MemberUsers = ($users | Where-Object {$_.UserType -eq "Member"}).Count
    GuestUsers = ($users | Where-Object {$_.UserType -eq "Guest"}).Count
}

# Verify group count
$groups = Get-MgGroup -All -Property Id
[PSCustomObject]@{
    TotalGroups = $groups.Count
}

# Verify current signed-in Graph account
(Get-MgContext).Account
```

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm workbook was read-only | Admin workstation | Review commands used | No tenant, subscription, user, group, domain, or role changes were made |
| 2 | Disconnect Azure session | Admin workstation | Disconnect-AzAccount | Azure PowerShell session is cleared |
| 3 | Disconnect Graph session | Admin workstation | Disconnect-MgGraph | Graph PowerShell session is cleared |
| 4 | Remove exported files if sensitive | Admin workstation | Remove-Item ".\Tenant_Subscription_Directory_Baseline" -Recurse -Force | Local exports are removed |
| 5 | Close browser sessions | Admin workstation | Sign out from portals | Admin sessions are closed |
| 6 | Revoke stale sessions if needed | Entra admin center | Users > select admin > Revoke sessions | Admin token sessions are invalidated |
| 7 | Document no-change status | Workbook notes | Add note: "Inventory only, no config changed" | Audit trail clearly states no config mutation occurred |

## Rollback Notes

This workbook is designed as read-only inventory and comparison.

Normal rollback is not required unless:

- You accidentally changed directory selection
- You changed default subscription context
- You exported sensitive tenant information to an unsafe location
- You granted consent to Graph scopes from an account that should not have done so
- You modified RBAC, roles, domains, users, groups, or licenses outside the workbook scope

If a configuration change occurred accidentally, handle rollback in the specific workbook for that feature.

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Failure_Checks

| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| Azure portal shows no subscriptions | User lacks Azure RBAC or selected wrong directory | Admin workstation | Get-AzSubscription | Switch directory or request subscription Reader access |
| Microsoft 365 admin center opens but Azure portal has no resources | M365 role exists but no Azure RBAC assignment | Admin workstation | Get-AzRoleAssignment -SignInName user@domain.com | Assign proper Azure RBAC at required scope |
| User is Global Administrator but cannot manage Azure resources | Entra role does not equal Azure RBAC Owner | Admin workstation | Get-AzContext; Get-AzRoleAssignment | Assign Azure RBAC role or elevate access if approved |
| User is subscription Owner but cannot manage users | Azure RBAC does not grant Entra directory admin rights | Admin workstation | Get-MgUser | Assign proper Entra role if needed |
| Domain appears in M365 but not usable for mail | DNS verification done but Exchange records incomplete | Admin workstation | nslookup -type=mx domain.com | Configure MX, SPF, DKIM, DMARC, Autodiscover as required |
| Tenant ID differs between Azure and M365 findings | Different directories / tenants selected | Admin workstation | Get-AzContext; Get-MgOrganization | Switch directory or document split-tenant design |
| Get-MgOrganization fails | Missing Graph permission or not connected | Admin workstation | Get-MgContext | Reconnect with Organization.Read.All |
| Get-MgSubscribedSku fails | Missing Graph permission | Admin workstation | Connect-MgGraph -Scopes Organization.Read.All,Directory.Read.All | Reconnect with required scopes |
| Get-AzManagementGroup fails | User lacks management group read access | Admin workstation | Get-AzManagementGroup | Request Management Group Reader or document not visible |
| Graph connects to wrong tenant | Cached session or wrong account selected | Admin workstation | Disconnect-MgGraph; Connect-MgGraph -TenantId "<tenant-id>" | Reconnect to explicit tenant |
| Azure PowerShell connects to wrong subscription | Default context points elsewhere | Admin workstation | Set-AzContext -SubscriptionId "<subscription-id>" | Set explicit subscription |
| Azure CLI shows wrong subscription | Default account context points elsewhere | Admin workstation | az account set --subscription "<subscription-id>" | Set explicit subscription |
| Portal and PowerShell disagree | Different account, tenant, or subscription context | Admin workstation | Get-AzContext; Get-MgContext; az account show | Align account and tenant context |
| No guest users visible | User lacks directory read permissions or filters are wrong | Admin workstation | Get-MgUser -All -Property UserType | Use directory reader permissions and filter UserType |
| Licenses visible but users cannot access services | License assigned but service plan disabled or blocked by policy | Admin workstation | Get-MgUserLicenseDetail -UserId user@domain.com | Review license service plans and Conditional Access |
| Admin role visible but portal blade blocked | Role is not sufficient for workload or PIM not activated | Admin workstation | Entra PIM > My roles | Activate eligible role or assign correct role |
| Role assignment appears inherited unexpectedly | Assignment inherited from management group or parent scope | Admin workstation | Get-AzRoleAssignment -IncludeClassicAdministrators | Trace scope inheritance |
| Support asks for tenant ID and subscription ID | Required identifiers missing from baseline | Admin workstation | Get-AzContext; Get-MgOrganization | Provide captured GUIDs from workbook output |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Documentation_Output_Template

## Tenant Baseline

| Field | Value |
|---|---|
| Tenant display name |  |
| Tenant ID |  |
| Initial domain |  |
| Primary domain |  |
| Verified domains |  |
| M365 tenant confirmed | Yes / No |
| Azure tenant confirmed | Yes / No |
| Azure and M365 same tenant | Yes / No / Unknown |

## Azure Subscription Baseline

| Subscription Name | Subscription ID | Tenant ID | State | Management Group | Notes |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## Role Boundary Baseline

| Role Type | Example Role | Scope | Assigned To | Notes |
|---|---|---|---|---|
| Entra role | Global Administrator | Tenant |  |  |
| Entra role | Privileged Role Administrator | Tenant |  |  |
| M365 role | Exchange Administrator | M365 tenant / workload |  |  |
| M365 role | SharePoint Administrator | M365 tenant / workload |  |  |
| M365 role | Teams Administrator | M365 tenant / workload |  |  |
| Azure RBAC | Owner | Subscription / RG / resource |  |  |
| Azure RBAC | Contributor | Subscription / RG / resource |  |  |
| Azure RBAC | User Access Administrator | Subscription / RG / resource |  |  |
| Azure RBAC | Reader | Subscription / RG / resource |  |  |

## Domain Baseline

| Domain | Type | Default | Initial | Capabilities | DNS Provider | Notes |
|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |

## Licensing Baseline

| SKU Part Number | Enabled Units | Consumed Units | Available Units | Notes |
|---|---:|---:|---:|---|
|  |  |  |  |  |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Related_Labs

| Lab | Relationship |
|---|---|
| 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline | Uses this tenant/subscription/directory baseline to configure admin tooling |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces | Builds on the domain baseline from this workbook |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles | Expands the role boundary comparison from this workbook |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline | Uses identified tenant and role boundaries to secure admin access |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline | Uses tenant, domain, licensing, and identity inventory from this workbook |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking | Uses tenant and subscription identifiers to configure monitoring and audit baselines |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Uses this workbook as the first troubleshooting reference for wrong tenant, wrong subscription, wrong portal, and wrong role issues |
| Azure Governance RBAC Workbook | Deep dive for Azure subscriptions, management groups, policy, locks, and RBAC |
| Microsoft 365 Tenant Administration Workbook | Deep dive for M365 admin center, domains, users, licenses, groups, and service health |
| Cloud Hybrid Identity Workbook | Deep dive for domain verification, Entra Connect, cloud sync, SSPR, MFA, and Conditional Access |

---

# 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories_Completion_Criteria

| Requirement | Complete |
|---|---|
| Tenant display name documented |  |
| Tenant ID documented |  |
| Initial domain documented |  |
| Primary domain documented |  |
| Verified domains documented |  |
| Microsoft 365 tenant confirmed |  |
| Azure directory confirmed |  |
| Azure subscriptions documented |  |
| Subscription IDs documented |  |
| Subscription tenant association documented |  |
| Management group visibility checked |  |
| Azure RBAC assignments reviewed |  |
| Entra admin roles reviewed |  |
| M365 admin roles reviewed |  |
| License SKUs reviewed |  |
| Users and guests reviewed |  |
| Portal comparison completed |  |
| PowerShell context validated |  |
| Graph context validated |  |
| CLI context validated if used |  |
| Output files stored or removed safely |  |
| Source gaps documented |  |

## Final Expected State

The admin can clearly state:

- This is the Entra tenant ID.
- This is the Microsoft 365 tenant backed by that tenant.
- These are the verified domains.
- These are the Azure subscriptions associated with the tenant.
- These are the role models in play.
- This is where Azure RBAC stops and Entra / M365 admin roles begin.
- This is the correct portal and command context for the next workbook.