# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Index

## Purpose

Configure a repeatable cloud admin tooling baseline for Azure, Microsoft 365, Microsoft Entra, Microsoft Graph, and workload portals.

This workbook establishes:

- Portal access baseline
- Browser/profile baseline
- Admin workstation baseline
- PowerShell baseline
- Azure PowerShell baseline
- Microsoft Graph PowerShell baseline
- Azure CLI baseline
- Microsoft 365 workload module baseline
- Context verification baseline
- Safe sign-in / sign-out procedures
- Export and evidence capture structure

## Outcome

By the end of this workbook, the admin workstation should be able to:

- Open the correct cloud admin portals
- Confirm current tenant and subscription context
- Authenticate to Azure PowerShell
- Authenticate to Azure CLI
- Authenticate to Microsoft Graph PowerShell
- Confirm Microsoft 365 tenant identity
- Confirm Azure subscription identity
- Run basic inventory commands safely
- Avoid running commands against the wrong tenant or subscription
- Save baseline output for future workbooks

## Workbook Sections

- Source_Basis
- Mental_Model
- Environment_Assumptions
- Planning_Table
- Admin_Portal_Map
- Required_Roles_And_Permissions
- Configuration_Checklist
- Admin_Workstation_Precheck_Skeleton
- PowerShell_Baseline_Skeleton
- Azure_PowerShell_Baseline_Skeleton
- Azure_CLI_Baseline_Skeleton
- Microsoft_Graph_PowerShell_Baseline_Skeleton
- Microsoft_365_Workload_Module_Skeleton
- Portal_Validation_Skeleton
- Verification_Commands
- Rollback
- Failure_Checks
- Documentation_Output_Template
- Related_Labs
- Completion_Criteria

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Source_Basis

| Source | Relevant Area | What It Supports |
|---|---|---|
| Microsoft 365 documentation fork | Microsoft 365 admin center, domains, users, roles, service health, Message Center | Microsoft 365 portal baseline |
| Microsoft Entra documentation | Entra admin center, users, groups, roles, authentication, Conditional Access, app registrations | Identity portal and Graph baseline |
| Azure management documentation | Azure portal, subscriptions, management groups, RBAC, resource groups, policy, activity log | Azure admin portal and Azure PowerShell baseline |
| Microsoft Graph documentation | Graph permissions, organization inventory, users, groups, subscribed SKUs, directory roles | Graph PowerShell baseline |
| Azure CLI documentation | az login, az account, tenant and subscription context | CLI baseline |
| Azure PowerShell documentation | Connect-AzAccount, Get-AzContext, Get-AzTenant, Get-AzSubscription | Azure PowerShell baseline |
| Existing Cloud Core Admin workbook 01 | Tenant, directory, subscription, and domain comparison | Context dependency for this workbook |
| Existing Workbook repository structure | Source basis, mental model, planning, checklist, skeletons, verification, rollback, failure checks | Obsidian-ready workbook format |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Mental_Model

| Layer | Tool / Portal | Main Purpose | Common Failure |
|---|---|---|---|
| Browser access | Admin portals | Interactive administration | Wrong browser profile or stale session |
| Identity plane | Entra admin center | Users, groups, roles, authentication, apps | Confusing tenant role with Azure RBAC |
| Azure resource plane | Azure portal | Subscriptions, resources, RBAC, policy, monitoring | Wrong directory or subscription selected |
| Microsoft 365 service plane | Microsoft 365 admin center | Users, licenses, domains, service health | Assuming M365 admin role grants Azure resource access |
| Security plane | Defender portal | Incidents, secure score, Defender workloads | Missing security role |
| Compliance plane | Purview portal | Audit, DLP, retention, eDiscovery | Missing compliance role |
| Mail plane | Exchange admin center | Mailboxes, mail flow, accepted domains, quarantine handoff | Missing Exchange role group |
| Collaboration plane | Teams admin center | Teams users, policies, meetings, apps | Missing Teams admin role |
| Content plane | SharePoint admin center | SharePoint sites, OneDrive, sharing, storage | Missing SharePoint admin role |
| Endpoint plane | Intune admin center | Device enrollment, compliance, apps, configuration | Missing Intune role |
| PowerShell identity | Microsoft Graph PowerShell | Directory and M365 API automation | Missing delegated Graph scopes |
| PowerShell Azure | Az PowerShell | Azure subscription and resource automation | Wrong Az context |
| CLI Azure | Azure CLI | Azure resource automation and scripting | Wrong az account context |
| Workload modules | ExchangeOnlineManagement, MicrosoftTeams, PnP.PowerShell | Workload-specific admin | Module mismatch or role mismatch |

## Core Rule

The portal or shell is not the authority by itself.

The real authority is:

- Signed-in account
- Tenant context
- Subscription context
- Role assignment
- Graph consent scope
- Workload role
- Conditional Access result

Always verify context before inventory, configuration, or troubleshooting.

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Environment_Assumptions

| Item | Lab Example | Production Example |
|---|---|---|
| Admin workstation | Windows 11 admin workstation | Privileged access workstation |
| Browser | Microsoft Edge | Microsoft Edge with separate admin profile |
| Admin profile | Cloud Admin Profile | Dedicated browser profile for admin work |
| Admin account | cloudadmin@contoso.onmicrosoft.com | named admin account |
| Break glass account | breakglass01@contoso.onmicrosoft.com | emergency access account |
| Target tenant | Contoso Lab | Production tenant |
| Target subscription | Azure Lab Subscription | Production / dev / shared services subscription |
| PowerShell version | PowerShell 7.x preferred | PowerShell 7.x preferred |
| Windows PowerShell | 5.1 fallback | Used only for legacy modules if required |
| Azure CLI | Current stable version | Current enterprise-approved version |
| Graph module | Microsoft.Graph | Microsoft.Graph selected profile |
| Azure module | Az | Az.Accounts plus required Az modules |
| M365 modules | ExchangeOnlineManagement, MicrosoftTeams, PnP.PowerShell | Installed only if admin scope requires them |
| Output path | .\Cloud_Admin_Tooling_Baseline | Secure evidence location |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Planning_Table

| Item | Value To Record | Why It Matters |
|---|---|---|
| Admin workstation hostname |  | Confirms where tooling was installed |
| Admin workstation OS |  | Confirms compatibility |
| PowerShell version |  | Determines module compatibility |
| Azure CLI version |  | Determines command support |
| Az module version |  | Determines Azure PowerShell behavior |
| Microsoft.Graph module version |  | Determines Graph cmdlet support |
| ExchangeOnlineManagement version |  | Determines Exchange Online command behavior |
| MicrosoftTeams version |  | Determines Teams cmdlet behavior |
| PnP.PowerShell version |  | Determines SharePoint/OneDrive automation behavior |
| Signed-in admin account |  | Confirms identity used |
| Tenant ID |  | Prevents wrong tenant actions |
| Subscription ID |  | Prevents wrong subscription actions |
| Default subscription |  | Prevents wrong Azure CLI/Az context |
| Graph scopes granted |  | Confirms Graph read/write capability |
| Admin roles |  | Confirms portal access |
| Conditional Access impact |  | Explains blocked sign-ins or MFA prompts |
| Browser profile used |  | Reduces session collision |
| Evidence output path |  | Stores validation output |
| Secret storage approach |  | Prevents credentials in scripts |
| Approved modules |  | Prevents unsupported tools |
| Update cadence |  | Keeps tooling current |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Admin_Portal_Map

| Portal | URL | Primary Use | Baseline Validation |
|---|---|---|---|
| Azure portal | https://portal.azure.com | Azure resources, subscriptions, RBAC, policy, monitoring | Confirm directory and subscription |
| Microsoft Entra admin center | https://entra.microsoft.com | Identity, users, groups, roles, authentication, apps | Confirm tenant ID and directory roles |
| Microsoft 365 admin center | https://admin.microsoft.com | Users, licenses, domains, service health, Message Center | Confirm tenant and license visibility |
| Exchange admin center | https://admin.exchange.microsoft.com | Mailboxes, groups, mail flow, accepted domains | Confirm Exchange admin access |
| SharePoint admin center | https://admin.microsoft.com/sharepoint | Sites, OneDrive, sharing, storage | Confirm SharePoint admin access |
| Teams admin center | https://admin.teams.microsoft.com | Teams policies, users, meetings, apps, calling | Confirm Teams admin access |
| Intune admin center | https://intune.microsoft.com | Device enrollment, compliance, apps, configuration | Confirm Intune admin access |
| Defender portal | https://security.microsoft.com | Incidents, secure score, Defender workloads | Confirm security role access |
| Purview portal | https://purview.microsoft.com | Audit, retention, DLP, eDiscovery, compliance | Confirm compliance role access |
| Microsoft 365 Apps admin center | https://config.office.com | M365 Apps servicing, inventory, policy | Confirm apps admin access |
| Power Platform admin center | https://admin.powerplatform.microsoft.com | Power Platform environments, DLP, capacity | Confirm Power Platform role access |
| Azure Cloud Shell | https://shell.azure.com | Browser-based Azure CLI and PowerShell | Confirm tenant/subscription context |
| Microsoft Graph Explorer | https://developer.microsoft.com/graph/graph-explorer | Manual Graph API testing | Confirm Graph permissions and tenant context |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Required_Roles_And_Permissions

| Task | Minimum Useful Role | Notes |
|---|---|---|
| View tenant overview | Global Reader / Directory Reader | Needed for read-only tenant inventory |
| View users and groups | Global Reader / Directory Reader | Graph scopes may still require consent |
| View domains | Global Reader / Domain Name Administrator | Domain actions require higher permissions |
| View licenses | Global Reader / License Administrator / Billing Reader | Billing data may require billing role |
| Manage users | User Administrator | Does not equal Azure subscription access |
| Manage groups | Groups Administrator | Group ownership can also matter |
| Manage roles | Privileged Role Administrator | Required for many role assignment tasks |
| Manage Conditional Access | Conditional Access Administrator | Security Reader may only view |
| View Azure subscriptions | Reader at subscription scope | Entra roles do not guarantee Azure resource visibility |
| Manage Azure RBAC | Owner or User Access Administrator | Subscription Owner is not Global Administrator by default |
| Manage Azure resources | Contributor / Owner | Contributor cannot assign RBAC |
| View service health | Service Support Administrator / Global Reader | Message Center visibility can vary |
| Manage Exchange Online | Exchange Administrator | Some tasks require Exchange role groups |
| Manage SharePoint / OneDrive | SharePoint Administrator | Tenant-wide sharing and site settings |
| Manage Teams | Teams Administrator | Teams policy and meeting/calling settings |
| Manage Intune | Intune Administrator | Device and app management |
| Manage Defender | Security Administrator | Security Reader for read-only |
| Manage Purview | Compliance Administrator / Compliance Data Administrator | Purview permissions may be portal-specific |
| Run Graph inventory | Delegated Graph scopes | Scopes must match cmdlet requirements |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Open dedicated admin browser profile | Admin workstation | Microsoft Edge profile: Cloud Admin | Admin session is isolated from personal/work browsing |
| 2 | Open Azure portal | Admin workstation | https://portal.azure.com | Azure portal loads |
| 3 | Confirm Azure directory | Admin workstation | Azure portal > Settings > Directories + subscriptions | Correct tenant is selected |
| 4 | Confirm Azure subscription | Admin workstation | Azure portal > Subscriptions | Target subscription is visible |
| 5 | Open Entra admin center | Admin workstation | https://entra.microsoft.com | Entra portal loads |
| 6 | Confirm tenant ID | Admin workstation | Entra admin center > Overview | Tenant ID matches workbook 01 |
| 7 | Open Microsoft 365 admin center | Admin workstation | https://admin.microsoft.com | M365 portal loads |
| 8 | Confirm M365 tenant | Admin workstation | Settings > Org settings > Organization profile | Tenant matches workbook 01 |
| 9 | Open workload portals | Admin workstation | Exchange, Teams, SharePoint, Intune, Defender, Purview URLs | Portals either load or access limitation is documented |
| 10 | Confirm PowerShell version | Admin workstation | $PSVersionTable | PowerShell version is documented |
| 11 | Configure PowerShell execution policy for current user if needed | Admin workstation | Set-ExecutionPolicy RemoteSigned -Scope CurrentUser | Current user can run local admin scripts |
| 12 | Confirm PSGallery trust | Admin workstation | Get-PSRepository | PSGallery status is documented |
| 13 | Install or update Az module | Admin workstation | Install-Module Az -Scope CurrentUser -AllowClobber | Az module is available |
| 14 | Install or update Microsoft Graph module | Admin workstation | Install-Module Microsoft.Graph -Scope CurrentUser | Microsoft.Graph module is available |
| 15 | Install or update Exchange Online module if needed | Admin workstation | Install-Module ExchangeOnlineManagement -Scope CurrentUser | Exchange module is available |
| 16 | Install or update Teams module if needed | Admin workstation | Install-Module MicrosoftTeams -Scope CurrentUser | Teams module is available |
| 17 | Install or update PnP PowerShell if needed | Admin workstation | Install-Module PnP.PowerShell -Scope CurrentUser | PnP module is available |
| 18 | Verify Azure CLI installed | Admin workstation | az version | Azure CLI version is shown |
| 19 | Update Azure CLI if approved | Admin workstation | winget upgrade Microsoft.AzureCLI | Azure CLI is current or update is documented |
| 20 | Connect Azure PowerShell | Admin workstation | Connect-AzAccount -Tenant "<tenant-id>" | Azure PowerShell authenticates to target tenant |
| 21 | Set Azure PowerShell subscription context | Admin workstation | Set-AzContext -SubscriptionId "<subscription-id>" | Correct subscription selected |
| 22 | Connect Azure CLI | Admin workstation | az login --tenant "<tenant-id>" | Azure CLI authenticates to target tenant |
| 23 | Set Azure CLI subscription context | Admin workstation | az account set --subscription "<subscription-id>" | Correct CLI subscription selected |
| 24 | Connect Microsoft Graph PowerShell | Admin workstation | Connect-MgGraph -TenantId "<tenant-id>" -Scopes "Organization.Read.All","Directory.Read.All","User.Read.All","Group.Read.All" | Graph PowerShell authenticates |
| 25 | Verify Graph context | Admin workstation | Get-MgContext | Correct tenant, account, and scopes are visible |
| 26 | Verify Azure PowerShell context | Admin workstation | Get-AzContext | Correct tenant and subscription are visible |
| 27 | Verify Azure CLI context | Admin workstation | az account show --output table | Correct tenant and subscription are visible |
| 28 | Export tooling baseline | Admin workstation | Run baseline export skeleton | Versions, contexts, portals, and access results are captured |
| 29 | Document access gaps | Admin workstation | Workbook notes | Missing roles, blocked portals, or missing modules are recorded |
| 30 | Save baseline | Admin workstation | Commit note to workbook repo or Obsidian vault | Tooling baseline is ready for later labs |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Admin_Workstation_Precheck_Skeleton

```powershell
# ============================================================
# Admin Workstation Precheck
# ============================================================

$OutputRoot = ".\Cloud_Admin_Tooling_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

# Host information
$ComputerInfo = [PSCustomObject]@{
    ComputerName        = $env:COMPUTERNAME
    UserName            = $env:USERNAME
    UserDomain          = $env:USERDOMAIN
    OS                  = (Get-CimInstance Win32_OperatingSystem).Caption
    OSVersion           = (Get-CimInstance Win32_OperatingSystem).Version
    PowerShellVersion   = $PSVersionTable.PSVersion.ToString()
    PowerShellEdition   = $PSVersionTable.PSEdition
    CurrentDate         = Get-Date
}

$ComputerInfo | Format-List
$ComputerInfo | Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AdminWorkstation_Precheck.csv")

# Check execution policy
Get-ExecutionPolicy -List |
    Format-Table -AutoSize

Get-ExecutionPolicy -List |
    Out-File -FilePath (Join-Path $OutputPath "ExecutionPolicy.txt")

# Check PowerShell repositories
Get-PSRepository |
    Select-Object Name, SourceLocation, InstallationPolicy |
    Format-Table -AutoSize

Get-PSRepository |
    Select-Object Name, SourceLocation, InstallationPolicy |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "PSRepositories.csv")

# Check TLS setting for the current session
[Net.ServicePointManager]::SecurityProtocol |
    Out-File -FilePath (Join-Path $OutputPath "SecurityProtocol.txt")

Write-Host "Precheck output path: $OutputPath"
```

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_PowerShell_Baseline_Skeleton

```powershell
# ============================================================
# PowerShell Baseline
# ============================================================

# Optional: allow local scripts for current user
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# Optional: trust PSGallery after organization approval
# Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted

# Install baseline modules
$Modules = @(
    "Az",
    "Microsoft.Graph",
    "ExchangeOnlineManagement",
    "MicrosoftTeams"
)

foreach ($Module in $Modules) {
    Write-Host "Installing or updating module: $Module"
    Install-Module $Module -Scope CurrentUser -Force -AllowClobber
}

# Optional SharePoint / OneDrive automation module
# Install-Module PnP.PowerShell -Scope CurrentUser -Force -AllowClobber

# Inventory installed modules
$BaselineModules = @(
    "Az",
    "Az.Accounts",
    "Microsoft.Graph",
    "Microsoft.Graph.Authentication",
    "ExchangeOnlineManagement",
    "MicrosoftTeams",
    "PnP.PowerShell"
)

$InstalledModuleReport = foreach ($Module in $BaselineModules) {
    $Found = Get-Module -ListAvailable -Name $Module |
        Sort-Object Version -Descending |
        Select-Object -First 1

    [PSCustomObject]@{
        ModuleName = $Module
        Installed  = [bool]$Found
        Version    = if ($Found) { $Found.Version.ToString() } else { $null }
        Path       = if ($Found) { $Found.Path } else { $null }
    }
}

$InstalledModuleReport |
    Format-Table -AutoSize

$InstalledModuleReport |
    Export-Csv -NoTypeInformation -Path ".\PowerShell_Module_Baseline.csv"
```

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Azure_PowerShell_Baseline_Skeleton

```powershell
# ============================================================
# Azure PowerShell Baseline
# ============================================================

# Replace with values from workbook 01
$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"

# Connect to target tenant
Connect-AzAccount -Tenant $TenantId

# Set target subscription
Set-AzContext -SubscriptionId $SubscriptionId

# Confirm context
$AzContext = Get-AzContext

$AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Format-List

# Confirm tenant access
Get-AzTenant |
    Select-Object Id, Name, Domains |
    Format-Table -AutoSize

# Confirm subscription access
Get-AzSubscription |
    Select-Object Name, Id, TenantId, State |
    Sort-Object Name |
    Format-Table -AutoSize

# Confirm current subscription role assignments if permitted
try {
    Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
        Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
        Sort-Object RoleDefinitionName, DisplayName |
        Format-Table -AutoSize
}
catch {
    Write-Warning "Could not read subscription role assignments."
    Write-Warning $_.Exception.Message
}

# Export baseline
$AzBaseline = [PSCustomObject]@{
    AccountName       = $AzContext.Account.Id
    TenantId          = $AzContext.Tenant.Id
    SubscriptionName  = $AzContext.Subscription.Name
    SubscriptionId    = $AzContext.Subscription.Id
    Environment       = $AzContext.Environment.Name
    DateCollected     = Get-Date
}

$AzBaseline |
    Format-List

$AzBaseline |
    Export-Csv -NoTypeInformation -Path ".\AzurePowerShell_Context_Baseline.csv"
```

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Azure_CLI_Baseline_Skeleton

```bash
# ============================================================
# Azure CLI Baseline
# ============================================================

# Replace with values from workbook 01
TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"

# Check Azure CLI version
az version

# Sign in to target tenant
az login --tenant "$TENANT_ID"

# List available subscriptions
az account list --output table

# Set target subscription
az account set --subscription "$SUBSCRIPTION_ID"

# Confirm active context
az account show --output table

# Confirm active context as JSON
az account show \
  --query "{name:name,id:id,tenantId:tenantId,user:user.name,isDefault:isDefault,state:state}" \
  --output json

# List tenants visible to account
az account tenant list --output table

# List management groups if permitted
az account management-group list --output table

# List role assignments at subscription scope if permitted
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --output table

# Test Microsoft Graph call through az rest
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json

# Save outputs
mkdir -p Cloud_Admin_Tooling_Baseline

az version > Cloud_Admin_Tooling_Baseline/az_version.json

az account show \
  --query "{name:name,id:id,tenantId:tenantId,user:user.name,isDefault:isDefault,state:state}" \
  --output json > Cloud_Admin_Tooling_Baseline/az_account_show.json

az account list \
  --output json > Cloud_Admin_Tooling_Baseline/az_account_list.json

az account tenant list \
  --output json > Cloud_Admin_Tooling_Baseline/az_tenant_list.json
```

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Microsoft_Graph_PowerShell_Baseline_Skeleton

```powershell
# ============================================================
# Microsoft Graph PowerShell Baseline
# ============================================================

# Replace with tenant ID from workbook 01
$TenantId = "<tenant-id>"

# Baseline read scopes
$Scopes = @(
    "Organization.Read.All",
    "Directory.Read.All",
    "User.Read.All",
    "Group.Read.All",
    "RoleManagement.Read.Directory"
)

# Connect to Microsoft Graph
Connect-MgGraph -TenantId $TenantId -Scopes $Scopes

# Confirm Graph context
$MgContext = Get-MgContext

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

# Confirm organization
$Organization = Get-MgOrganization

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

# Confirm verified domains
$Organization.VerifiedDomains |
    Select-Object Name, Type, IsDefault, IsInitial, Capabilities |
    Sort-Object IsDefault -Descending, Name |
    Format-Table -AutoSize

# Confirm user inventory read access
$Users = Get-MgUser -Top 10 -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled

$Users |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, Id |
    Format-Table -AutoSize

# Confirm group inventory read access
$Groups = Get-MgGroup -Top 10 -Property Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes

$Groups |
    Select-Object DisplayName, MailEnabled, SecurityEnabled, GroupTypes, Id |
    Format-Table -AutoSize

# Confirm license inventory read access
try {
    Get-MgSubscribedSku -All |
        Select-Object SkuPartNumber, SkuId, ConsumedUnits,
            @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}} |
        Format-Table -AutoSize
}
catch {
    Write-Warning "Could not read subscribed SKUs."
    Write-Warning $_.Exception.Message
}

# Export Graph baseline
$GraphBaseline = [PSCustomObject]@{
    AccountName      = $MgContext.Account
    TenantId         = $MgContext.TenantId
    ClientId         = $MgContext.ClientId
    OrganizationName = $Organization.DisplayName
    OrganizationId   = $Organization.Id
    ScopeCount       = $MgContext.Scopes.Count
    Scopes           = ($MgContext.Scopes -join ";")
    DateCollected    = Get-Date
}

$GraphBaseline |
    Format-List

$GraphBaseline |
    Export-Csv -NoTypeInformation -Path ".\MicrosoftGraph_Context_Baseline.csv"
```

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Microsoft_365_Workload_Module_Skeleton

```powershell
# ============================================================
# Microsoft 365 Workload Module Baseline
# ============================================================

# Replace with admin UPN
$AdminUpn = "<admin-upn>"

# -----------------------------
# Exchange Online
# -----------------------------
try {
    Connect-ExchangeOnline -UserPrincipalName $AdminUpn

    Get-OrganizationConfig |
        Select-Object Name, Guid, DisplayName |
        Format-List

    Get-EXOMailbox -ResultSize 5 |
        Select-Object DisplayName, UserPrincipalName, RecipientTypeDetails |
        Format-Table -AutoSize

    Disconnect-ExchangeOnline -Confirm:$false
}
catch {
    Write-Warning "Exchange Online connection or validation failed."
    Write-Warning $_.Exception.Message
}

# -----------------------------
# Microsoft Teams
# -----------------------------
try {
    Connect-MicrosoftTeams

    Get-CsTenant |
        Select-Object DisplayName, TenantId |
        Format-List

    Get-Team -NumberOfThreads 1 |
        Select-Object DisplayName, GroupId, Visibility |
        Format-Table -AutoSize

    Disconnect-MicrosoftTeams
}
catch {
    Write-Warning "Microsoft Teams connection or validation failed."
    Write-Warning $_.Exception.Message
}

# -----------------------------
# SharePoint Online via PnP PowerShell
# Requires tenant admin URL
# -----------------------------
try {
    $TenantAdminUrl = "https://<tenant-admin-name>-admin.sharepoint.com"

    Connect-PnPOnline -Url $TenantAdminUrl -Interactive

    Get-PnPTenant |
        Select-Object SharingCapability, OneDriveStorageQuota, DefaultSharingLinkType |
        Format-List

    Disconnect-PnPOnline
}
catch {
    Write-Warning "SharePoint / PnP connection or validation failed."
    Write-Warning $_.Exception.Message
}
```

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Portal_Validation_Skeleton

## Portal Validation Table

| Portal | URL | Sign-In Account | Tenant Confirmed | Access Result | Notes |
|---|---|---|---|---|---|
| Azure portal | https://portal.azure.com |  |  | Success / Blocked / Limited |  |
| Entra admin center | https://entra.microsoft.com |  |  | Success / Blocked / Limited |  |
| Microsoft 365 admin center | https://admin.microsoft.com |  |  | Success / Blocked / Limited |  |
| Exchange admin center | https://admin.exchange.microsoft.com |  |  | Success / Blocked / Limited |  |
| SharePoint admin center | https://admin.microsoft.com/sharepoint |  |  | Success / Blocked / Limited |  |
| Teams admin center | https://admin.teams.microsoft.com |  |  | Success / Blocked / Limited |  |
| Intune admin center | https://intune.microsoft.com |  |  | Success / Blocked / Limited |  |
| Defender portal | https://security.microsoft.com |  |  | Success / Blocked / Limited |  |
| Purview portal | https://purview.microsoft.com |  |  | Success / Blocked / Limited |  |
| Power Platform admin center | https://admin.powerplatform.microsoft.com |  |  | Success / Blocked / Limited |  |
| Azure Cloud Shell | https://shell.azure.com |  |  | Success / Blocked / Limited |  |
| Graph Explorer | https://developer.microsoft.com/graph/graph-explorer |  |  | Success / Blocked / Limited |  |

## Validation Notes

- Success means the portal loads and expected blade access is visible.
- Blocked means sign-in, Conditional Access, license, or role prevented access.
- Limited means portal loads but required admin blades are unavailable.
- Record whether the issue is identity, role, license, Conditional Access, or workload-specific permission.

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Verification_Commands

```powershell
# ============================================================
# Verification Commands
# ============================================================

# PowerShell version
$PSVersionTable

# Installed modules
Get-Module -ListAvailable Az,Az.Accounts,Microsoft.Graph,Microsoft.Graph.Authentication,ExchangeOnlineManagement,MicrosoftTeams,PnP.PowerShell |
    Select-Object Name, Version, Path |
    Sort-Object Name, Version |
    Format-Table -AutoSize

# Azure PowerShell context
Get-AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Format-List

# Azure tenants
Get-AzTenant |
    Select-Object Id, Name, Domains |
    Format-Table -AutoSize

# Azure subscriptions
Get-AzSubscription |
    Select-Object Name, Id, TenantId, State |
    Sort-Object Name |
    Format-Table -AutoSize

# Microsoft Graph context
Get-MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

# Microsoft 365 / Entra organization
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

# Verified domains
(Get-MgOrganization).VerifiedDomains |
    Select-Object Name, Type, IsDefault, IsInitial, Capabilities |
    Sort-Object IsDefault -Descending, Name |
    Format-Table -AutoSize

# Sample users
Get-MgUser -Top 5 -Property DisplayName,UserPrincipalName,UserType,AccountEnabled |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled |
    Format-Table -AutoSize

# Sample groups
Get-MgGroup -Top 5 -Property DisplayName,MailEnabled,SecurityEnabled,GroupTypes |
    Select-Object DisplayName, MailEnabled, SecurityEnabled, GroupTypes |
    Format-Table -AutoSize

# License SKUs
Get-MgSubscribedSku -All |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}} |
    Format-Table -AutoSize
```

```bash
# ============================================================
# Azure CLI Verification Commands
# ============================================================

# Azure CLI version
az version

# Active Azure CLI account
az account show --output table

# List Azure CLI subscriptions
az account list --output table

# List tenants
az account tenant list --output table

# Test Microsoft Graph via Azure CLI
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json
```

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Disconnect Azure PowerShell account | Admin workstation | Disconnect-AzAccount | Azure PowerShell session is cleared |
| 2 | Clear Azure PowerShell context if needed | Admin workstation | Clear-AzContext -Force | Cached Az context is removed |
| 3 | Disconnect Microsoft Graph | Admin workstation | Disconnect-MgGraph | Graph session is cleared |
| 4 | Logout Azure CLI | Admin workstation | az logout | Azure CLI token cache is cleared |
| 5 | Clear Azure CLI account data if needed | Admin workstation | az account clear | Azure CLI account context is cleared |
| 6 | Disconnect Exchange Online | Admin workstation | Disconnect-ExchangeOnline -Confirm:$false | Exchange Online session is closed |
| 7 | Disconnect Teams | Admin workstation | Disconnect-MicrosoftTeams | Teams session is closed |
| 8 | Disconnect PnP PowerShell | Admin workstation | Disconnect-PnPOnline | SharePoint PnP session is closed |
| 9 | Remove baseline output if sensitive | Admin workstation | Remove-Item ".\Cloud_Admin_Tooling_Baseline" -Recurse -Force | Local evidence files are removed |
| 10 | Uninstall module if wrong module installed | Admin workstation | Uninstall-Module <ModuleName> -AllVersions | Incorrect module is removed |
| 11 | Reset PSGallery trust if required | Admin workstation | Set-PSRepository -Name "PSGallery" -InstallationPolicy Untrusted | Repository trust reverted |
| 12 | Close admin browser profile | Admin workstation | Browser sign-out | Portal sessions are closed |

## Rollback Notes

This workbook installs or validates admin tooling.

Normal rollback is only needed when:

- You authenticated with the wrong account
- You consented to Graph scopes in the wrong tenant
- You installed modules on the wrong workstation
- You cached CLI or PowerShell context for the wrong tenant
- You exported sensitive tenant data to an unsafe location
- You installed an unsupported module version

Do not delete tenant, subscription, user, group, role, or domain objects as rollback for this workbook. This workbook should not modify tenant configuration.

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Failure_Checks

| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| Azure portal opens wrong tenant | Browser session or directory selector points to another tenant | Admin workstation | Check portal directory selector | Switch directory and use dedicated admin profile |
| Azure portal shows no subscriptions | User lacks Azure RBAC or wrong tenant selected | Admin workstation | Get-AzSubscription; az account list | Switch tenant or request Reader role |
| Entra admin center blocks role blades | Missing Entra role | Admin workstation | Get-MgContext | Request Global Reader, Directory Reader, or relevant admin role |
| Microsoft 365 admin center shows limited blades | Missing M365 admin role | Admin workstation | Portal role review | Assign required M365 admin role |
| PowerShell cannot install module | Execution policy, TLS, proxy, or PSGallery issue | Admin workstation | Get-PSRepository; Get-ExecutionPolicy -List | Fix repository, proxy, or execution policy |
| Install-Module says command not found | PowerShellGet unavailable or broken | Admin workstation | Get-Module PowerShellGet -ListAvailable | Update PowerShellGet or use approved package method |
| Az module command not found | Az module not installed or session not refreshed | Admin workstation | Get-Module -ListAvailable Az.Accounts | Install module and restart PowerShell |
| Connect-AzAccount fails | Conditional Access, MFA, tenant restriction, or stale token | Admin workstation | Connect-AzAccount -Tenant "<tenant-id>" | Use correct account, meet CA requirements, clear context |
| Get-AzContext shows wrong subscription | Context set to another subscription | Admin workstation | Set-AzContext -SubscriptionId "<subscription-id>" | Set explicit subscription |
| Azure CLI az not recognized | Azure CLI not installed or PATH not refreshed | Admin workstation | az version | Install Azure CLI and reopen shell |
| az login succeeds but wrong subscription active | Default CLI subscription differs from target | Admin workstation | az account set --subscription "<subscription-id>" | Set explicit subscription |
| az rest Graph call fails | Graph permission issue or token audience issue | Admin workstation | az account get-access-token --resource-type ms-graph | Reauthenticate and confirm tenant |
| Connect-MgGraph prompts repeatedly | Browser auth issue or Conditional Access issue | Admin workstation | Connect-MgGraph -TenantId "<tenant-id>" | Use device code or correct browser profile if approved |
| Get-MgOrganization fails | Missing Graph scope or tenant consent | Admin workstation | Get-MgContext | Reconnect with Organization.Read.All |
| Get-MgUser fails | Missing User.Read.All or Directory.Read.All | Admin workstation | Connect-MgGraph -Scopes User.Read.All,Directory.Read.All | Reconnect with required scopes |
| Get-MgSubscribedSku fails | Missing organization/license read permissions | Admin workstation | Connect-MgGraph -Scopes Organization.Read.All,Directory.Read.All | Reconnect with required scopes |
| Exchange Online connection fails | Missing module or Exchange role | Admin workstation | Get-Module ExchangeOnlineManagement -ListAvailable | Install module or assign Exchange role |
| Teams connection fails | Missing module or Teams role | Admin workstation | Get-Module MicrosoftTeams -ListAvailable | Install module or assign Teams role |
| PnP connection fails | Wrong tenant admin URL or module not installed | Admin workstation | Get-Module PnP.PowerShell -ListAvailable | Correct URL or install module |
| Portal works but PowerShell fails | Different account, tenant, or CA path | Admin workstation | Get-AzContext; Get-MgContext | Align account and tenant |
| PowerShell works but portal fails | Browser profile stale session or blocked cookies | Admin workstation | Browser sign-out | Clear profile session or use dedicated admin profile |
| Commands run against wrong tenant | Tenant ID not explicitly specified | Admin workstation | Get-AzContext; Get-MgContext; az account show | Always use explicit tenant and subscription IDs |
| Output files contain sensitive data | Evidence exported to unsafe path | Admin workstation | Review output folder | Move to secure location or delete |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Documentation_Output_Template

## Admin Workstation Baseline

| Field | Value |
|---|---|
| Computer name |  |
| OS |  |
| PowerShell version |  |
| Azure CLI version |  |
| Admin browser profile |  |
| Output folder |  |
| Date collected |  |

## Tenant Context Baseline

| Field | Value |
|---|---|
| Admin account |  |
| Tenant display name |  |
| Tenant ID |  |
| Initial domain |  |
| Primary domain |  |
| Verified domain count |  |

## Azure Context Baseline

| Field | Value |
|---|---|
| Azure account |  |
| Azure tenant ID |  |
| Subscription name |  |
| Subscription ID |  |
| Azure environment |  |
| Management group visible | Yes / No |
| Subscription RBAC visible | Yes / No |

## Microsoft Graph Context Baseline

| Field | Value |
|---|---|
| Graph account |  |
| Graph tenant ID |  |
| Graph client ID |  |
| Granted scopes |  |
| Organization read test | Success / Failed |
| User read test | Success / Failed |
| Group read test | Success / Failed |
| License read test | Success / Failed |

## Module Baseline

| Module | Installed | Version | Notes |
|---|---|---|---|
| Az |  |  |  |
| Az.Accounts |  |  |  |
| Microsoft.Graph |  |  |  |
| Microsoft.Graph.Authentication |  |  |  |
| ExchangeOnlineManagement |  |  |  |
| MicrosoftTeams |  |  |  |
| PnP.PowerShell |  |  |  |

## Portal Baseline

| Portal | Access Result | Notes |
|---|---|---|
| Azure portal |  |  |
| Entra admin center |  |  |
| Microsoft 365 admin center |  |  |
| Exchange admin center |  |  |
| SharePoint admin center |  |  |
| Teams admin center |  |  |
| Intune admin center |  |  |
| Defender portal |  |  |
| Purview portal |  |  |
| Power Platform admin center |  |  |
| Azure Cloud Shell |  |  |
| Graph Explorer |  |  |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories | Supplies tenant ID, subscription ID, domain, and role baseline |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces | Uses admin portals and modules configured here |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles | Uses portal and command access to compare role planes |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline | Uses admin tools to configure privileged access |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline | Uses Graph and portal access to inventory and configure identities |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking | Uses portal, Graph, Az, and CLI access for audit validation |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Uses this baseline to detect wrong tenant, wrong account, wrong subscription, stale token, and missing role problems |
| Azure Governance RBAC Workbook | Extends Azure PowerShell and Azure CLI baseline |
| Microsoft 365 Tenant Administration Workbook | Extends M365 admin center and Graph baseline |
| Exchange Online Administration Workbook | Extends ExchangeOnlineManagement baseline |
| Teams Administration Workbook | Extends MicrosoftTeams baseline |
| SharePoint OneDrive Workbook | Extends SharePoint admin center and PnP.PowerShell baseline |
| Defender XDR Workbook | Extends security portal baseline |
| Purview Workbook | Extends compliance portal baseline |
| Intune Workbook | Extends endpoint admin center baseline |

---

# 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline_Completion_Criteria

| Requirement | Complete |
|---|---|
| Dedicated admin browser profile created or identified |  |
| Azure portal access validated |  |
| Entra admin center access validated |  |
| Microsoft 365 admin center access validated |  |
| Workload portal access reviewed |  |
| PowerShell version documented |  |
| Execution policy documented |  |
| PSGallery state documented |  |
| Az module installed and validated |  |
| Microsoft.Graph module installed and validated |  |
| Azure CLI installed and validated |  |
| Azure PowerShell authenticated to explicit tenant |  |
| Azure PowerShell subscription context set |  |
| Azure CLI authenticated to explicit tenant |  |
| Azure CLI subscription context set |  |
| Microsoft Graph PowerShell authenticated to explicit tenant |  |
| Graph scopes documented |  |
| Organization read test completed |  |
| User read test completed |  |
| Group read test completed |  |
| License read test completed |  |
| Optional Exchange module tested |  |
| Optional Teams module tested |  |
| Optional PnP module tested |  |
| Output files secured or removed |  |
| Access gaps documented |  |

## Final Expected State

The admin can clearly state:

- This is the browser profile used for cloud administration.
- This is the tenant ID used for portal, PowerShell, CLI, and Graph sessions.
- This is the Azure subscription ID used for Azure PowerShell and Azure CLI.
- This is the signed-in admin account.
- These are the installed admin modules and versions.
- These are the Graph scopes granted for baseline inventory.
- These portals work.
- These portals are blocked or limited.
- These modules work.
- These modules are missing or blocked.
- The next workbook can safely assume a validated tooling baseline.