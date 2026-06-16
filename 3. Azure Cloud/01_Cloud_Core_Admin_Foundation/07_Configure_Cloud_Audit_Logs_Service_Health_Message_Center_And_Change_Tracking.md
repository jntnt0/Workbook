# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Index

## Purpose

Configure a baseline for Microsoft cloud audit visibility, service health review, Message Center tracking, and change tracking.

This workbook covers:

- Microsoft Entra audit logs
- Microsoft Entra sign-in logs
- Microsoft 365 audit log readiness
- Microsoft 365 service health
- Microsoft 365 Message Center
- Azure Activity Log
- Azure resource diagnostic settings
- Log Analytics workspace planning
- Microsoft Graph reporting inventory
- Admin role change tracking
- User and group change tracking
- License change tracking
- Conditional Access change tracking
- Privileged access change tracking
- Emergency access account monitoring
- Service advisory tracking
- Planned change tracking
- Evidence export
- Review cadence
- Rollback and failure checks

## Outcome

By the end of this workbook, the admin should be able to:

- Identify which audit source records which type of cloud activity
- Review Entra audit and sign-in logs
- Review Microsoft 365 audit readiness
- Review Azure subscription Activity Log
- Review Microsoft 365 Service health
- Review Microsoft 365 Message Center
- Export baseline audit and service health evidence
- Track high-risk administrative changes
- Track role assignments and privileged changes
- Track user, group, guest, license, and SSPR changes
- Track Azure RBAC and Azure resource changes
- Document service advisories and planned changes
- Build a repeatable review cadence for later troubleshooting workbooks

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Source_Basis

| Source | Relevant Area | What It Supports |
|---|---|---|
| Microsoft Entra documentation | Audit logs, sign-in logs, role changes, Conditional Access results, authentication events | Identity audit and sign-in review |
| Microsoft 365 documentation fork | Service health, Message Center, Microsoft 365 admin center reports, audit controls | M365 health and change tracking |
| Microsoft Graph documentation | Reports, audit log queries, service communications, users, groups, roles, licenses | Graph-based audit and health inventory |
| Azure management documentation | Azure Activity Log, diagnostic settings, Log Analytics, Azure Monitor alerts | Azure resource-plane change tracking |
| Defender documentation | Security incidents, alerting, identity risk handoff, Defender portal evidence | Security monitoring handoff |
| Purview documentation | Audit, compliance search, DLP events, retention, eDiscovery, activity explorer | Compliance audit handoff |
| Existing Workbook 01 | Tenant, subscription, directory, and domain comparison | Supplies target tenant and subscription context |
| Existing Workbook 02 | Admin portals, PowerShell, Azure CLI, and Graph baseline | Supplies tooling required for audit and health review |
| Existing Workbook 03 | Custom domain and namespace baseline | Supplies domain and DNS change context |
| Existing Workbook 04 | Role plane comparison | Supplies role systems that must be monitored |
| Existing Workbook 05 | Cloud admin, break glass, and privileged access baseline | Supplies emergency access and privileged account monitoring targets |
| Existing Workbook 06 | Users, groups, guests, licenses, and SSPR baseline | Supplies identity objects and events to track |
| Existing Workbook structure | Source basis, mental model, planning, checklist, skeletons, verification, rollback, failure checks | Obsidian-ready workbook format |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Mental_Model

| Log / Tracking Source | Plane | Records | Common Use |
|---|---|---|---|
| Microsoft Entra audit logs | Identity plane | User, group, app, role, policy, directory changes | Who changed identity configuration |
| Microsoft Entra sign-in logs | Identity access plane | User sign-ins, app sign-ins, CA result, MFA result | Why sign-in succeeded or failed |
| Microsoft Entra provisioning logs | Identity automation plane | App provisioning, user/group provisioning events | Why app provisioning failed |
| Microsoft 365 audit log | Collaboration and workload plane | User and admin activity across M365 workloads | Workload activity investigations |
| Microsoft 365 Service health | Service availability plane | Incidents and advisories | Is Microsoft service degraded |
| Microsoft 365 Message Center | Planned change plane | Upcoming service changes and feature updates | What Microsoft is changing |
| Azure Activity Log | Azure resource plane | Subscription management events, RBAC, policy, resource changes | Who changed Azure resources |
| Azure diagnostic logs | Resource data plane | Service-specific logs and metrics | Deep resource troubleshooting |
| Azure Monitor alerts | Monitoring plane | Alert rules, action groups, fired alerts | Operational alerting |
| Defender portal | Security plane | Incidents, alerts, secure score, exposure | Security detection and response |
| Purview audit | Compliance plane | Compliance and workload audit events | Investigation and compliance evidence |
| Graph reports | API reporting plane | Usage, service communications, reports | Scripted reporting and exports |
| Change log | Local documentation plane | Planned/admin-approved changes | Human-readable operations timeline |
| Evidence folder | Local evidence plane | CSV, JSON, screenshots, notes | Repeatable audit trail |

## Core Rule

Logs answer different questions.

Use the right source for the question:

- Who changed a user, group, role, app, or policy? Use Entra audit logs.
- Why did sign-in fail? Use Entra sign-in logs.
- Who changed Azure resources or Azure RBAC? Use Azure Activity Log and Azure RBAC exports.
- Is Microsoft 365 degraded? Use Service health.
- What Microsoft change is coming? Use Message Center.
- What happened inside Exchange, SharePoint, Teams, Purview, or Defender? Use workload audit and service portals.
- What changed in the workbook project? Use the local change log and GitHub history.

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Log_Source_Map

| Question | Primary Source | Secondary Source | Notes |
|---|---|---|---|
| Who created a user? | Entra audit logs | Microsoft Graph audit query | Identity object event |
| Who deleted a user? | Entra audit logs | Deleted users inventory | Confirm soft-delete state |
| Who changed a user license? | Entra audit logs | User license details | Track direct and group-based licensing |
| Who added a guest? | Entra audit logs | Guest user inventory | Also review external collaboration settings |
| Who changed a group? | Entra audit logs | Group membership export | Track group ownership and membership |
| Who added a user to a privileged role? | Entra audit logs | Directory role member export | Critical security event |
| Who changed Conditional Access? | Entra audit logs | CA policy export | Critical lockout/security event |
| Why was a sign-in blocked? | Entra sign-in logs | Conditional Access policy result | Review CA, MFA, risk, device state |
| Did emergency account sign in? | Entra sign-in logs | Alert evidence | Must alert on every attempt |
| Who changed Azure RBAC? | Azure Activity Log | Get-AzRoleAssignment export | Track Owner and UAA changes |
| Who changed Azure resources? | Azure Activity Log | Resource diagnostic logs | Resource-plane tracking |
| Is Azure service impacted? | Azure Service Health | Azure status page | Subscription and region context matters |
| Is Microsoft 365 service impacted? | Microsoft 365 Service health | Message Center | Tenant-specific incidents and advisories |
| What changes are upcoming? | Microsoft 365 Message Center | Roadmap / admin center | Assign owner and due date |
| What did a workload admin do? | M365 audit / workload audit | Workload portal logs | Depends on workload and licensing |
| What changed in Defender? | Defender portal | Unified audit / security alerts | Security role required |
| What changed in Purview? | Purview audit | Purview role groups | Compliance permissions required |
| What changed in admin baseline workbooks? | Workbook change log | GitHub commits | Human process evidence |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Role_Requirements

| Task | Minimum Useful Role | Notes |
|---|---|---|
| View Entra audit logs | Reports Reader / Global Reader / Security Reader | Tenant role and licensing may affect visibility |
| View Entra sign-in logs | Reports Reader / Global Reader / Security Reader | Sign-in log retention depends on license |
| View role assignments | Global Reader / Privileged Role Administrator | RoleManagement.Read.Directory scope needed for Graph |
| View Conditional Access results | Security Reader / Conditional Access Administrator / Global Reader | Policy details may require CA permissions |
| View Microsoft 365 Service health | Service Support Administrator / Global Reader | M365 admin center role |
| View Microsoft 365 Message Center | Message Center Reader / Global Reader | Assign reader role for planned change review |
| View Azure Activity Log | Reader at subscription scope | Azure RBAC required |
| Configure diagnostic settings | Monitoring Contributor / Contributor / Owner | Depends on target resource scope |
| Create Log Analytics workspace | Contributor | Workspace creation requires Azure RBAC |
| Create Azure Monitor alerts | Monitoring Contributor / Contributor | Also requires action group access |
| View Defender incidents | Security Reader / Security Operator / Security Administrator | Defender permissions vary by workload |
| View Purview audit | Audit Reader / Compliance Administrator | Purview role group may be required |
| Export Graph reports | Reports Reader / Global Reader plus Graph scopes | Report.Read.All and relevant scopes |
| Review Exchange audit | Exchange Administrator / View-Only Organization Management | Workload-specific |
| Review SharePoint audit | SharePoint Administrator / Audit role | Workload and Purview dependent |
| Review Teams audit | Teams Administrator / Audit role | Workload and Purview dependent |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Planning_Table

| Item | Value To Record | Why It Matters |
|---|---|---|
| Tenant ID |  | Prevents wrong tenant review |
| Tenant display name |  | Evidence context |
| Primary domain |  | Identity namespace context |
| Subscription ID |  | Azure Activity Log scope |
| Log Analytics workspace name |  | Central log target |
| Log Analytics workspace resource group |  | Azure location and RBAC |
| Log Analytics region |  | Data residency and latency |
| Retention requirement |  | Audit investigation window |
| Service health owner |  | Owns Microsoft incidents |
| Message Center owner |  | Owns planned Microsoft changes |
| Identity audit owner |  | Owns Entra audit review |
| Security monitoring owner |  | Owns Defender/security alerts |
| Compliance owner |  | Owns Purview and audit |
| Azure monitoring owner |  | Owns Azure logs and alerts |
| Review cadence |  | Daily, weekly, monthly rhythm |
| Emergency account list |  | Must alert on sign-in |
| Critical role list |  | Must alert or review changes |
| Critical CA policies |  | Must track changes |
| Critical groups |  | Must monitor membership changes |
| License groups |  | Must monitor license assignment changes |
| Guest access groups |  | Must review external access |
| Export path |  | Evidence storage |
| Change log path |  | Human-readable change timeline |
| Alert destinations |  | Email, Teams, ITSM, SIEM |
| Time zone |  | Event correlation |
| Evidence retention |  | How long exports are retained |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Review_Cadence

| Review Area | Cadence | Owner | Evidence |
|---|---|---|---|
| Emergency account sign-ins | Daily / alert immediately | Identity / Security | Sign-in log export |
| Global Administrator changes | Daily / alert immediately | Identity | Directory audit export |
| Privileged Role Administrator changes | Daily / alert immediately | Identity | Directory audit export |
| Azure Owner / User Access Administrator changes | Daily / alert immediately | Azure platform owner | Activity Log and RBAC export |
| Conditional Access changes | Daily / alert immediately | Identity / Security | Audit log and policy export |
| Authentication methods policy changes | Weekly / alert for critical | Identity / Security | Audit log export |
| User creation/deletion | Weekly | Identity operations | Audit log and user export |
| Guest invitations | Weekly | Collaboration owner | Guest export and audit log |
| Group membership changes | Weekly | Identity operations | Group membership export |
| License assignment changes | Weekly | M365 admin | License export |
| SSPR registration/reset events | Weekly | Identity operations | Audit/sign-in export |
| M365 Service health | Daily | Service owner | Health issue log |
| Message Center | Weekly | Change owner | Message tracking table |
| Azure Activity Log | Weekly | Azure platform owner | Activity Log export |
| Diagnostic settings coverage | Monthly | Azure platform owner | Resource settings export |
| Workload audit coverage | Monthly | Workload owners | Exchange, SharePoint, Teams, Defender, Purview evidence |
| Workbook change log | Every change | Change owner | Change log table |
| Access reviews | Monthly / quarterly | Identity governance | Access review evidence |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Critical_Change_Table

| Change Type | Source | Risk | Required Review |
|---|---|---|---|
| Global Administrator added | Entra audit logs | Critical | Immediate |
| Privileged Role Administrator added | Entra audit logs | Critical | Immediate |
| Conditional Access policy changed | Entra audit logs | Critical | Immediate |
| Emergency account sign-in | Entra sign-in logs | Critical | Immediate |
| Break glass CA exclusion changed | Entra audit logs | Critical | Immediate |
| Azure Owner added | Azure Activity Log | Critical | Immediate |
| User Access Administrator added | Azure Activity Log | Critical | Immediate |
| App registration credential added | Entra audit logs | High | Daily |
| Enterprise app consent granted | Entra audit logs | High | Daily |
| Authentication methods policy changed | Entra audit logs | High | Daily |
| Guest invited | Entra audit logs | Medium / High | Weekly |
| Guest added to access group | Entra audit logs | Medium / High | Weekly |
| License group changed | Entra audit logs | Medium | Weekly |
| SSPR policy changed | Entra audit logs | High | Daily |
| User deleted | Entra audit logs | Medium / High | Weekly or immediate for admin users |
| Group owner changed | Entra audit logs | Medium / High | Weekly |
| Mail flow rule changed | Exchange audit | High | Daily |
| External sharing changed | SharePoint audit / admin logs | High | Daily |
| Teams external access changed | Teams admin / audit | High | Daily |
| DLP policy changed | Purview audit | High | Daily |
| Defender security setting changed | Defender audit | High | Daily |
| Diagnostic setting removed | Azure Activity Log | High | Daily |
| Alert rule disabled | Azure Activity Log | High | Daily |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm workbook 01 tenant/subscription baseline | Admin workstation | Review tenant ID and subscription ID | Correct tenant and subscription known |
| 2 | Confirm workbook 02 tooling baseline | Admin workstation | Get-MgContext; Get-AzContext; az account show | Tools connected to correct tenant |
| 3 | Confirm workbook 04 role model | Admin workstation | Review role plane map | Audit roles and Azure RBAC requirements known |
| 4 | Confirm workbook 05 privileged targets | Admin workstation | Review emergency accounts and critical roles | High-risk accounts and roles known |
| 5 | Confirm workbook 06 identity targets | Admin workstation | Review users, groups, guests, licenses, SSPR | Identity objects to monitor known |
| 6 | Connect Microsoft Graph with audit/report scopes | Admin workstation | Connect-MgGraph -Scopes "AuditLog.Read.All","Directory.Read.All","Reports.Read.All","ServiceMessage.Read.All" | Graph session connected |
| 7 | Confirm organization context | Admin workstation | Get-MgOrganization | Correct tenant displayed |
| 8 | Export current Entra audit logs sample | Admin workstation | Get-MgAuditLogDirectoryAudit -Top 100 | Recent directory audit events exported |
| 9 | Export current Entra sign-in logs sample | Admin workstation | Get-MgAuditLogSignIn -Top 100 | Recent sign-in events exported |
| 10 | Export emergency account sign-ins | Admin workstation | Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'breakglass01@tenant.onmicrosoft.com'" | Emergency sign-in evidence exported |
| 11 | Export directory role members | Admin workstation | Get-MgDirectoryRoleMember | Privileged role baseline exported |
| 12 | Export users and groups | Admin workstation | Get-MgUser; Get-MgGroup | Identity inventory exported |
| 13 | Export license SKUs | Admin workstation | Get-MgSubscribedSku | License baseline exported |
| 14 | Review Microsoft 365 Service health portal | Admin workstation | https://admin.microsoft.com > Health > Service health | Active issues documented |
| 15 | Review Microsoft 365 Message Center portal | Admin workstation | https://admin.microsoft.com > Health > Message center | Planned changes documented |
| 16 | Export service communications through Graph if permitted | Admin workstation | Get-MgAdminServiceAnnouncementIssue / message command if available | Health and message data exported |
| 17 | Connect Azure PowerShell | Admin workstation | Connect-AzAccount -Tenant "<tenant-id>" | Azure session connected |
| 18 | Set target subscription | Admin workstation | Set-AzContext -SubscriptionId "<subscription-id>" | Correct subscription selected |
| 19 | Export Azure Activity Log sample | Admin workstation | Get-AzLog -StartTime (Get-Date).AddDays(-7) | Azure management events exported |
| 20 | Review Azure Service Health portal | Admin workstation | Azure portal > Service Health | Subscription/region issues documented |
| 21 | Review Azure Advisor if in scope | Admin workstation | Azure portal > Advisor | Operational recommendations documented |
| 22 | Review Azure Monitor alert rules | Admin workstation | Get-AzAlertRuleResource / portal | Existing alert coverage documented |
| 23 | Review Log Analytics workspace plan | Admin workstation | Get-AzOperationalInsightsWorkspace | Workspace presence documented |
| 24 | Create or validate Log Analytics workspace if approved | Admin workstation | New-AzOperationalInsightsWorkspace | Central workspace exists |
| 25 | Review diagnostic settings coverage | Admin workstation | Get-AzDiagnosticSetting | Resources with and without diagnostics documented |
| 26 | Configure diagnostic settings for selected resources if approved | Admin workstation | New-AzDiagnosticSetting | Resource logs sent to workspace |
| 27 | Define critical change alert requirements | Admin workstation | Planning table | Alert targets documented |
| 28 | Configure emergency account sign-in alert if supported | Portal / Monitor / Defender | Alert rule or detection rule | Sign-ins generate notification |
| 29 | Configure role change review process | Admin workstation | Change tracking table | Critical role changes reviewed |
| 30 | Configure Message Center owner workflow | M365 admin center | Assign owner / export message list | Planned changes tracked |
| 31 | Build local change log | Admin workstation | Documentation table | Manual admin changes recorded |
| 32 | Export final baseline | Admin workstation | Run export skeleton | Evidence saved |
| 33 | Save workbook output | Admin workstation | Commit note to workbook repo or Obsidian vault | Baseline ready for troubleshooting |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Precheck_Skeleton

```powershell
# ============================================================
# Cloud Audit, Health, Message Center, and Change Tracking Precheck
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$OutputRoot = ".\Cloud_Audit_Health_ChangeTracking_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

# -----------------------------
# Connect Microsoft Graph
# -----------------------------
Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "AuditLog.Read.All", `
    "Reports.Read.All", `
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
# Connect Azure
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

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Entra_Audit_Export_Skeleton

```powershell
# ============================================================
# Microsoft Entra Audit and Sign-In Export Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Cloud_Audit_Health_ChangeTracking_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "AuditLog.Read.All", `
    "Directory.Read.All", `
    "User.Read.All", `
    "Group.Read.All", `
    "RoleManagement.Read.Directory"

# -----------------------------
# Directory audit logs
# -----------------------------
try {
    $DirectoryAudit = Get-MgAuditLogDirectoryAudit -Top 250

    $DirectoryAudit |
        Select-Object `
            ActivityDateTime,
            ActivityDisplayName,
            Category,
            LoggedByService,
            OperationType,
            Result,
            ResultReason,
            Id |
        Sort-Object ActivityDateTime -Descending |
        Format-Table -AutoSize

    $DirectoryAudit |
        Select-Object `
            ActivityDateTime,
            ActivityDisplayName,
            Category,
            LoggedByService,
            OperationType,
            Result,
            ResultReason,
            Id |
        Sort-Object ActivityDateTime -Descending |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_DirectoryAudit_Recent.csv")
}
catch {
    Write-Warning "Could not retrieve Entra directory audit logs."
    Write-Warning $_.Exception.Message
}

# -----------------------------
# Sign-in logs
# -----------------------------
try {
    $SignIns = Get-MgAuditLogSignIn -Top 250

    $SignIns |
        Select-Object `
            CreatedDateTime,
            UserPrincipalName,
            AppDisplayName,
            IpAddress,
            ClientAppUsed,
            ConditionalAccessStatus,
            RiskState,
            RiskLevelAggregated,
            Status |
        Sort-Object CreatedDateTime -Descending |
        Format-Table -AutoSize

    $SignIns |
        Select-Object `
            CreatedDateTime,
            UserPrincipalName,
            AppDisplayName,
            IpAddress,
            ClientAppUsed,
            ConditionalAccessStatus,
            RiskState,
            RiskLevelAggregated,
            Status |
        Sort-Object CreatedDateTime -Descending |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_SignIns_Recent.csv")
}
catch {
    Write-Warning "Could not retrieve Entra sign-in logs."
    Write-Warning $_.Exception.Message
}

# -----------------------------
# Directory role membership baseline
# -----------------------------
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
            AppId             = $Member.AdditionalProperties.appId
        }
    }
}

$DirectoryRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_DirectoryRoleMembers_Baseline.csv")

# -----------------------------
# Critical role member baseline
# -----------------------------
$CriticalRoleNames = @(
    "Global Administrator",
    "Privileged Role Administrator",
    "Conditional Access Administrator",
    "Authentication Administrator",
    "Privileged Authentication Administrator",
    "Security Administrator",
    "Compliance Administrator",
    "Exchange Administrator",
    "SharePoint Administrator",
    "Teams Administrator",
    "Intune Administrator",
    "Application Administrator",
    "Cloud Application Administrator"
)

$CriticalRoleMembers = $DirectoryRoleMembers |
    Where-Object {$_.RoleDisplayName -in $CriticalRoleNames}

$CriticalRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Entra_CriticalRoleMembers_Baseline.csv")

Write-Host "Entra audit export complete. Output path: $OutputPath"
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Emergency_Access_Monitoring_Skeleton

```powershell
# ============================================================
# Emergency Access Account Monitoring Skeleton
# ============================================================

$TenantId = "<tenant-id>"

$EmergencyAccounts = @(
    "breakglass01@tenant.onmicrosoft.com",
    "breakglass02@tenant.onmicrosoft.com"
)

$OutputRoot = ".\Cloud_Audit_Health_ChangeTracking_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "AuditLog.Read.All", `
    "Directory.Read.All", `
    "User.Read.All"

foreach ($EmergencyAccount in $EmergencyAccounts) {
    Write-Host "Checking sign-ins for emergency account: $EmergencyAccount"

    try {
        $Filter = "userPrincipalName eq '$EmergencyAccount'"

        $SignIns = Get-MgAuditLogSignIn -Filter $Filter -Top 50

        $SignIns |
            Select-Object `
                CreatedDateTime,
                UserPrincipalName,
                AppDisplayName,
                IpAddress,
                ClientAppUsed,
                ConditionalAccessStatus,
                RiskState,
                RiskLevelAggregated,
                Status |
            Sort-Object CreatedDateTime -Descending |
            Format-Table -AutoSize

        $FileName = "$($EmergencyAccount.Replace('@','_').Replace('.','_'))_EmergencyAccountSignIns.csv"

        $SignIns |
            Select-Object `
                CreatedDateTime,
                UserPrincipalName,
                AppDisplayName,
                IpAddress,
                ClientAppUsed,
                ConditionalAccessStatus,
                RiskState,
                RiskLevelAggregated,
                Status |
            Sort-Object CreatedDateTime -Descending |
            Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath $FileName)
    }
    catch {
        Write-Warning "Could not retrieve sign-ins for $EmergencyAccount"
        Write-Warning $_.Exception.Message
    }
}

# Expected operational rule:
# Any successful or failed emergency account sign-in should be treated as a high priority review item.
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Identity_Change_Query_Skeleton

```powershell
# ============================================================
# Identity Change Query Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Cloud_Audit_Health_ChangeTracking_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "AuditLog.Read.All", `
    "Directory.Read.All"

# Pull recent directory audit events
$Audit = Get-MgAuditLogDirectoryAudit -Top 500

# User changes
$UserChanges = $Audit |
    Where-Object {
        $_.ActivityDisplayName -like "*user*" -or
        $_.Category -eq "UserManagement"
    }

$UserChanges |
    Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, OperationType, Result, ResultReason |
    Sort-Object ActivityDateTime -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Identity_UserChanges.csv")

# Group changes
$GroupChanges = $Audit |
    Where-Object {
        $_.ActivityDisplayName -like "*group*" -or
        $_.Category -eq "GroupManagement"
    }

$GroupChanges |
    Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, OperationType, Result, ResultReason |
    Sort-Object ActivityDateTime -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Identity_GroupChanges.csv")

# Role changes
$RoleChanges = $Audit |
    Where-Object {
        $_.ActivityDisplayName -like "*role*" -or
        $_.Category -eq "RoleManagement"
    }

$RoleChanges |
    Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, OperationType, Result, ResultReason |
    Sort-Object ActivityDateTime -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Identity_RoleChanges.csv")

# Policy changes
$PolicyChanges = $Audit |
    Where-Object {
        $_.ActivityDisplayName -like "*policy*" -or
        $_.ActivityDisplayName -like "*Conditional Access*" -or
        $_.Category -eq "Policy"
    }

$PolicyChanges |
    Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, OperationType, Result, ResultReason |
    Sort-Object ActivityDateTime -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Identity_PolicyChanges.csv")

# Application changes
$AppChanges = $Audit |
    Where-Object {
        $_.ActivityDisplayName -like "*application*" -or
        $_.ActivityDisplayName -like "*service principal*" -or
        $_.Category -eq "ApplicationManagement"
    }

$AppChanges |
    Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, OperationType, Result, ResultReason |
    Sort-Object ActivityDateTime -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Identity_AppChanges.csv")

Write-Host "Identity change exports complete. Output path: $OutputPath"
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Azure_Activity_Log_Skeleton

```powershell
# ============================================================
# Azure Activity Log Export Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$OutputRoot = ".\Cloud_Audit_Health_ChangeTracking_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

$StartTime = (Get-Date).AddDays(-7)
$EndTime = Get-Date

# -----------------------------
# Export Azure Activity Log
# -----------------------------
$ActivityLog = Get-AzLog -StartTime $StartTime -EndTime $EndTime

$ActivityLog |
    Select-Object `
        EventTimestamp,
        ResourceGroupName,
        ResourceProviderName,
        ResourceId,
        OperationName,
        Status,
        Caller,
        Category,
        Level,
        SubscriptionId |
    Sort-Object EventTimestamp -Descending |
    Format-Table -AutoSize

$ActivityLog |
    Select-Object `
        EventTimestamp,
        ResourceGroupName,
        ResourceProviderName,
        ResourceId,
        OperationName,
        Status,
        Caller,
        Category,
        Level,
        SubscriptionId |
    Sort-Object EventTimestamp -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Azure_ActivityLog_Last7Days.csv")

# -----------------------------
# Export RBAC-related activity
# -----------------------------
$RbacActivity = $ActivityLog |
    Where-Object {
        $_.OperationName.Value -like "*roleAssignments*" -or
        $_.OperationName.Value -like "*Microsoft.Authorization*"
    }

$RbacActivity |
    Select-Object EventTimestamp, OperationName, Status, Caller, ResourceId, Category, Level |
    Sort-Object EventTimestamp -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Azure_ActivityLog_RBAC_Changes.csv")

# -----------------------------
# Export policy-related activity
# -----------------------------
$PolicyActivity = $ActivityLog |
    Where-Object {
        $_.OperationName.Value -like "*policy*" -or
        $_.ResourceProviderName.Value -like "*Microsoft.Authorization*"
    }

$PolicyActivity |
    Select-Object EventTimestamp, OperationName, Status, Caller, ResourceId, Category, Level |
    Sort-Object EventTimestamp -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Azure_ActivityLog_Policy_Changes.csv")

# -----------------------------
# Export diagnostic setting activity
# -----------------------------
$DiagnosticActivity = $ActivityLog |
    Where-Object {
        $_.OperationName.Value -like "*diagnosticSettings*"
    }

$DiagnosticActivity |
    Select-Object EventTimestamp, OperationName, Status, Caller, ResourceId, Category, Level |
    Sort-Object EventTimestamp -Descending |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Azure_ActivityLog_DiagnosticSetting_Changes.csv")

Write-Host "Azure Activity Log export complete. Output path: $OutputPath"
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Log_Analytics_Workspace_Skeleton

```powershell
# ============================================================
# Log Analytics Workspace Baseline Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ResourceGroupName = "rg-monitoring-prod"
$WorkspaceName = "law-cloud-audit-prod"
$Location = "eastus"
$RetentionInDays = 90

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# -----------------------------
# Create resource group if approved
# -----------------------------
if (-not (Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue)) {
    New-AzResourceGroup -Name $ResourceGroupName -Location $Location
}

# -----------------------------
# Create Log Analytics workspace if approved
# -----------------------------
$Workspace = Get-AzOperationalInsightsWorkspace `
    -ResourceGroupName $ResourceGroupName `
    -Name $WorkspaceName `
    -ErrorAction SilentlyContinue

if (-not $Workspace) {
    $Workspace = New-AzOperationalInsightsWorkspace `
        -ResourceGroupName $ResourceGroupName `
        -Name $WorkspaceName `
        -Location $Location `
        -RetentionInDays $RetentionInDays
}

# -----------------------------
# Validate workspace
# -----------------------------
Get-AzOperationalInsightsWorkspace `
    -ResourceGroupName $ResourceGroupName `
    -Name $WorkspaceName |
    Select-Object Name, ResourceGroupName, Location, CustomerId, RetentionInDays |
    Format-List

# -----------------------------
# Export workspace baseline
# -----------------------------
Get-AzOperationalInsightsWorkspace `
    -ResourceGroupName $ResourceGroupName `
    -Name $WorkspaceName |
    Select-Object Name, ResourceGroupName, Location, CustomerId, RetentionInDays |
    Export-Csv -NoTypeInformation -Path ".\LogAnalyticsWorkspace_Baseline.csv"
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Diagnostic_Settings_Inventory_Skeleton

```powershell
# ============================================================
# Azure Diagnostic Settings Inventory Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$OutputRoot = ".\Cloud_Audit_Health_ChangeTracking_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# Get resources
$Resources = Get-AzResource

$DiagnosticInventory = foreach ($Resource in $Resources) {
    try {
        $Settings = Get-AzDiagnosticSetting -ResourceId $Resource.ResourceId -ErrorAction Stop

        if ($Settings) {
            foreach ($Setting in $Settings) {
                [PSCustomObject]@{
                    ResourceName       = $Resource.Name
                    ResourceType       = $Resource.ResourceType
                    ResourceGroupName  = $Resource.ResourceGroupName
                    ResourceId         = $Resource.ResourceId
                    DiagnosticName     = $Setting.Name
                    WorkspaceId        = $Setting.WorkspaceId
                    StorageAccountId   = $Setting.StorageAccountId
                    EventHubName       = $Setting.EventHubName
                    LogsEnabled        = ($Setting.Logs | Where-Object {$_.Enabled -eq $true}).Count
                    MetricsEnabled     = ($Setting.Metrics | Where-Object {$_.Enabled -eq $true}).Count
                }
            }
        }
        else {
            [PSCustomObject]@{
                ResourceName       = $Resource.Name
                ResourceType       = $Resource.ResourceType
                ResourceGroupName  = $Resource.ResourceGroupName
                ResourceId         = $Resource.ResourceId
                DiagnosticName     = $null
                WorkspaceId        = $null
                StorageAccountId   = $null
                EventHubName       = $null
                LogsEnabled        = 0
                MetricsEnabled     = 0
            }
        }
    }
    catch {
        [PSCustomObject]@{
            ResourceName       = $Resource.Name
            ResourceType       = $Resource.ResourceType
            ResourceGroupName  = $Resource.ResourceGroupName
            ResourceId         = $Resource.ResourceId
            DiagnosticName     = "ERROR_OR_UNSUPPORTED"
            WorkspaceId        = $null
            StorageAccountId   = $null
            EventHubName       = $null
            LogsEnabled        = $null
            MetricsEnabled     = $null
        }
    }
}

$DiagnosticInventory |
    Sort-Object ResourceType, ResourceGroupName, ResourceName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Azure_DiagnosticSettings_Inventory.csv")

# Resources without obvious diagnostic settings
$ResourcesWithoutDiagnostics = $DiagnosticInventory |
    Where-Object {$_.DiagnosticName -eq $null -or $_.DiagnosticName -eq "ERROR_OR_UNSUPPORTED"}

$ResourcesWithoutDiagnostics |
    Sort-Object ResourceType, ResourceGroupName, ResourceName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Azure_Resources_Without_DiagnosticSettings.csv")

Write-Host "Diagnostic settings inventory complete. Output path: $OutputPath"
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Create_Diagnostic_Setting_Skeleton

```powershell
# ============================================================
# Create Diagnostic Setting Skeleton
# Use only after confirming the resource supports diagnostic settings.
# Categories vary by resource type.
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$WorkspaceResourceGroupName = "rg-monitoring-prod"
$WorkspaceName = "law-cloud-audit-prod"
$TargetResourceId = "<resource-id>"
$DiagnosticSettingName = "diag-to-law-cloud-audit"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

$Workspace = Get-AzOperationalInsightsWorkspace `
    -ResourceGroupName $WorkspaceResourceGroupName `
    -Name $WorkspaceName

# List supported diagnostic categories
Get-AzDiagnosticSettingCategory -ResourceId $TargetResourceId |
    Select-Object Name, CategoryType |
    Format-Table -AutoSize

# Example:
# Configure after selecting the categories supported by the resource.
# Replace category names with valid categories from the command above.

$Logs = @(
    New-AzDiagnosticSettingLogSettingsObject -Enabled $true -Category "<log-category-name>"
)

$Metrics = @(
    New-AzDiagnosticSettingMetricSettingsObject -Enabled $true -Category "AllMetrics"
)

New-AzDiagnosticSetting `
    -Name $DiagnosticSettingName `
    -ResourceId $TargetResourceId `
    -WorkspaceId $Workspace.ResourceId `
    -Log $Logs `
    -Metric $Metrics

# Verify
Get-AzDiagnosticSetting -ResourceId $TargetResourceId |
    Select-Object Name, WorkspaceId, StorageAccountId, EventHubName |
    Format-List
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Service_Health_Portal_Skeleton

```text
# ============================================================
# Microsoft 365 Service Health Portal Review
# ============================================================

Portal:
https://admin.microsoft.com

Navigation:
Health > Service health

Review:
1. Active issues
2. Issues in your environment
3. Advisory history
4. Incident start time
5. Impacted services
6. Impacted users or regions if shown
7. Current status
8. Last update
9. Root cause if available
10. Next update time
11. Admin actions required
12. Workload owner
13. Internal incident ticket if created
14. User communication requirement

Documentation table:
| Issue ID | Service | Status | Impact | Start Time | Last Update | Owner | Internal Ticket | Notes |
|---|---|---|---|---|---|---|---|---|
|  |  | Active / Restored / Advisory |  |  |  |  |  |  |

Expected result:
Active Microsoft 365 service health issues are documented and assigned to an owner.
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Message_Center_Portal_Skeleton

```text
# ============================================================
# Microsoft 365 Message Center Review
# ============================================================

Portal:
https://admin.microsoft.com

Navigation:
Health > Message center

Review:
1. New messages
2. Major updates
3. Feature updates
4. Retirement notices
5. Admin action required messages
6. Service impacted
7. Message ID
8. Published date
9. Updated date
10. Action by date
11. Targeted release impact
12. User impact
13. Admin owner
14. Internal change ticket
15. Communication needed
16. Testing needed
17. Rollout status

Documentation table:
| Message ID | Title | Service | Category | Published | Action By | Owner | Status | Internal Ticket | Notes |
|---|---|---|---|---|---|---|---|---|---|
|  |  | Exchange / SharePoint / Teams / Entra / Intune / Defender / Purview | Plan / Prevent / Stay informed |  |  |  | New / Reviewing / Actioned / Closed |  |  |

Expected result:
Message Center items are tracked as planned changes rather than surprise changes.
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Graph_Service_Communications_Skeleton

```powershell
# ============================================================
# Microsoft 365 Service Communications Graph Skeleton
# Cmdlet availability can vary by installed Microsoft.Graph module profile/version.
# If cmdlets are unavailable, use Graph REST or the admin portal.
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Cloud_Audit_Health_ChangeTracking_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "ServiceMessage.Read.All", `
    "Reports.Read.All", `
    "Organization.Read.All"

# -----------------------------
# Option 1: Microsoft Graph PowerShell service announcement cmdlets
# -----------------------------
try {
    $Issues = Get-MgAdminServiceAnnouncementIssue -All

    $Issues |
        Select-Object Id, Title, Service, Status, Classification, StartDateTime, LastModifiedDateTime, EndDateTime |
        Sort-Object LastModifiedDateTime -Descending |
        Format-Table -AutoSize

    $Issues |
        Select-Object Id, Title, Service, Status, Classification, StartDateTime, LastModifiedDateTime, EndDateTime |
        Sort-Object LastModifiedDateTime -Descending |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "M365_ServiceHealth_Issues.csv")
}
catch {
    Write-Warning "Could not retrieve service announcement issues with Graph PowerShell cmdlet."
    Write-Warning $_.Exception.Message
}

try {
    $Messages = Get-MgAdminServiceAnnouncementMessage -All

    $Messages |
        Select-Object Id, Title, Services, Category, Severity, StartDateTime, LastModifiedDateTime, IsMajorChange |
        Sort-Object LastModifiedDateTime -Descending |
        Format-Table -AutoSize

    $Messages |
        Select-Object Id, Title, Services, Category, Severity, StartDateTime, LastModifiedDateTime, IsMajorChange |
        Sort-Object LastModifiedDateTime -Descending |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "M365_MessageCenter_Messages.csv")
}
catch {
    Write-Warning "Could not retrieve service announcement messages with Graph PowerShell cmdlet."
    Write-Warning $_.Exception.Message
}

# -----------------------------
# Option 2: REST fallback
# -----------------------------
try {
    Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/issues" |
        ConvertTo-Json -Depth 10 |
        Out-File -FilePath (Join-Path $OutputPath "M365_ServiceHealth_Issues_Rest.json")
}
catch {
    Write-Warning "Could not retrieve service health issues with Graph REST."
    Write-Warning $_.Exception.Message
}

try {
    Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/messages" |
        ConvertTo-Json -Depth 10 |
        Out-File -FilePath (Join-Path $OutputPath "M365_MessageCenter_Messages_Rest.json")
}
catch {
    Write-Warning "Could not retrieve Message Center messages with Graph REST."
    Write-Warning $_.Exception.Message
}
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Azure_Service_Health_Portal_Skeleton

```text
# ============================================================
# Azure Service Health Portal Review
# ============================================================

Portal:
https://portal.azure.com

Navigation:
Service Health

Review:
1. Service issues
2. Planned maintenance
3. Health advisories
4. Security advisories
5. Resource health
6. Subscription affected
7. Region affected
8. Service affected
9. Impact start time
10. Current status
11. Last update
12. Recommended action
13. Alert rules configured
14. Internal incident ticket if created

Documentation table:
| Tracking ID | Service | Region | Type | Status | Start Time | Owner | Internal Ticket | Notes |
|---|---|---|---|---|---|---|---|---|
|  |  |  | Service Issue / Planned Maintenance / Advisory / Security | Active / Resolved |  |  |  |  |

Expected result:
Azure health issues are tracked against subscription, region, and service ownership.
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Change_Log_Template

| Change ID | Date / Time | Change Type | Plane | Object | Actor | Approval / Ticket | Expected Result | Actual Result | Rollback Needed | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| CHG-0001 |  | User created | Entra |  |  |  |  |  | No |  |
| CHG-0002 |  | Group membership changed | Entra |  |  |  |  |  | No |  |
| CHG-0003 |  | License assigned | M365 / Entra |  |  |  |  |  | No |  |
| CHG-0004 |  | Conditional Access changed | Entra |  |  |  |  |  | Yes / No |  |
| CHG-0005 |  | Role assignment changed | Entra / Azure RBAC |  |  |  |  |  | Yes / No |  |
| CHG-0006 |  | Azure resource changed | Azure |  |  |  |  |  | Yes / No |  |
| CHG-0007 |  | Diagnostic setting changed | Azure Monitor |  |  |  |  |  | Yes / No |  |
| CHG-0008 |  | Message Center item reviewed | M365 |  |  |  |  |  | No |  |
| CHG-0009 |  | Service health incident reviewed | M365 / Azure |  |  |  |  |  | No |  |
| CHG-0010 |  | Emergency access test | Entra |  |  |  |  |  | No |  |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_KQL_Starter_Queries

```kusto
// ============================================================
// KQL Starter Queries for Log Analytics
// Use only where Entra, Azure, or resource logs are connected.
// Table names vary based on diagnostic configuration.
// ============================================================

// Azure Activity changes in the last 7 days
AzureActivity
| where TimeGenerated > ago(7d)
| project TimeGenerated, OperationNameValue, ActivityStatusValue, Caller, ResourceGroup, ResourceProviderValue, ResourceId
| order by TimeGenerated desc

// Azure RBAC role assignment changes
AzureActivity
| where TimeGenerated > ago(30d)
| where OperationNameValue has "roleAssignments"
| project TimeGenerated, OperationNameValue, ActivityStatusValue, Caller, ResourceId, CorrelationId
| order by TimeGenerated desc

// Azure diagnostic setting changes
AzureActivity
| where TimeGenerated > ago(30d)
| where OperationNameValue has "diagnosticSettings"
| project TimeGenerated, OperationNameValue, ActivityStatusValue, Caller, ResourceId
| order by TimeGenerated desc

// Failed Azure operations
AzureActivity
| where TimeGenerated > ago(7d)
| where ActivityStatusValue !in ("Success", "Succeeded")
| project TimeGenerated, OperationNameValue, ActivityStatusValue, Caller, ResourceGroup, ResourceId, Properties
| order by TimeGenerated desc

// Activity by caller
AzureActivity
| where TimeGenerated > ago(7d)
| summarize Count=count() by Caller
| order by Count desc
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Azure_CLI_Skeleton

```bash
# ============================================================
# Azure CLI Audit and Health Baseline Skeleton
# ============================================================

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

mkdir -p Cloud_Audit_Health_ChangeTracking_Baseline

# Confirm context
az account show --output table

az account show \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/az_account_show.json

# Azure Activity Log, last 7 days
az monitor activity-log list \
  --offset 7d \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/azure_activity_log_last7days.json

# Azure Activity Log, role assignment changes
az monitor activity-log list \
  --offset 30d \
  --query "[?contains(operationName.value, 'roleAssignments')]" \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/azure_rbac_activity_last30days.json

# Azure Activity Log, diagnostic setting changes
az monitor activity-log list \
  --offset 30d \
  --query "[?contains(operationName.value, 'diagnosticSettings')]" \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/azure_diagnostic_activity_last30days.json

# Role assignments baseline
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/azure_role_assignments_subscription.json

# Graph organization
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/graph_organization.json

# Graph service health issues if permission is available
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/issues" \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/m365_service_health_issues.json

# Graph Message Center messages if permission is available
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/messages" \
  --output json > Cloud_Audit_Health_ChangeTracking_Baseline/m365_message_center_messages.json
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Verification_Commands

```powershell
# ============================================================
# Verification Commands
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"

# -----------------------------
# Graph context
# -----------------------------
Get-MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

# -----------------------------
# Organization
# -----------------------------
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

# -----------------------------
# Entra audit logs
# -----------------------------
Get-MgAuditLogDirectoryAudit -Top 25 |
    Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, OperationType, Result |
    Format-Table -AutoSize

# -----------------------------
# Entra sign-in logs
# -----------------------------
Get-MgAuditLogSignIn -Top 25 |
    Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, IpAddress, ClientAppUsed, ConditionalAccessStatus, Status |
    Format-Table -AutoSize

# -----------------------------
# Directory role members
# -----------------------------
Get-MgDirectoryRole -All |
    Select-Object DisplayName, Id |
    Sort-Object DisplayName |
    Format-Table -AutoSize

# -----------------------------
# Azure context
# -----------------------------
Get-AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Format-List

# -----------------------------
# Azure Activity Log
# -----------------------------
Get-AzLog -StartTime (Get-Date).AddDays(-7) |
    Select-Object EventTimestamp, OperationName, Status, Caller, ResourceGroupName, ResourceId |
    Sort-Object EventTimestamp -Descending |
    Select-Object -First 25 |
    Format-Table -AutoSize

# -----------------------------
# Azure RBAC baseline
# -----------------------------
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Format-Table -AutoSize

# -----------------------------
# Log Analytics workspaces
# -----------------------------
Get-AzOperationalInsightsWorkspace |
    Select-Object Name, ResourceGroupName, Location, CustomerId, RetentionInDays |
    Format-Table -AutoSize

# -----------------------------
# Microsoft 365 service communications through Graph REST
# -----------------------------
Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/issues"

Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/messages"
```

```bash
# ============================================================
# Azure CLI Verification Commands
# ============================================================

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"

az account show --output table

az monitor activity-log list \
  --offset 7d \
  --output table

az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --output table

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/issues" \
  --output json

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/messages" \
  --output json
```

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm this workbook was mostly read-only | Admin workstation | Review command history | No unwanted changes made |
| 2 | Disconnect Graph session | Admin workstation | Disconnect-MgGraph | Graph session closed |
| 3 | Disconnect Azure PowerShell | Admin workstation | Disconnect-AzAccount | Azure session closed |
| 4 | Logout Azure CLI | Admin workstation | az logout | CLI session closed |
| 5 | Remove exported evidence if sensitive | Admin workstation | Remove-Item ".\Cloud_Audit_Health_ChangeTracking_Baseline" -Recurse -Force | Local exports removed |
| 6 | Remove test diagnostic setting if created by mistake | Admin workstation | Remove-AzDiagnosticSetting | Diagnostic setting removed |
| 7 | Remove test Log Analytics workspace only if unused | Admin workstation | Remove-AzOperationalInsightsWorkspace | Workspace removed after approval |
| 8 | Remove test alert rule if created by mistake | Admin workstation | Remove-AzMetricAlertRuleV2 or portal | Alert rule removed |
| 9 | Restore previous diagnostic setting if overwritten | Admin workstation | New-AzDiagnosticSetting using previous config | Logging destination restored |
| 10 | Restore previous retention setting if changed | Admin workstation | Set workspace retention | Retention restored |
| 11 | Restore Message Center owner tracking if changed | M365 admin center | Reassign owner/status | Planned change workflow restored |
| 12 | Document rollback action | Admin workstation | Change log table | Audit trail preserved |

## Rollback Notes

Do not delete production Log Analytics workspaces without validating:

- Connected resources
- Alert rules
- Automation dependencies
- Sentinel or SIEM dependencies
- Retention requirements
- Compliance requirements
- Cost management requirements

Do not remove diagnostic settings unless you know:

- Where logs are currently going
- Whether security monitoring depends on them
- Whether compliance retention depends on them
- Whether troubleshooting dashboards depend on them
- Whether alert rules depend on them

This workbook should not disable audit logging, emergency monitoring, or service health review.

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Failure_Checks

| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| Get-MgAuditLogDirectoryAudit fails | Missing AuditLog.Read.All or role permission | Admin workstation | Get-MgContext | Reconnect with required scope and correct admin role |
| Get-MgAuditLogSignIn fails | Missing AuditLog.Read.All or sign-in log permission | Admin workstation | Get-MgContext | Use Reports Reader, Security Reader, or Global Reader as approved |
| Sign-in logs are empty | No events in retention window or wrong tenant | Admin workstation | Get-MgContext | Confirm tenant and retention |
| Emergency account sign-ins not found | No sign-ins, wrong UPN, or wrong tenant | Admin workstation | Get-MgAuditLogSignIn -Filter | Confirm emergency account UPN and tenant |
| Audit log lacks detail | Retention/licensing/permission limitation | Admin workstation | Portal review | Use proper license and role |
| Directory audit query is too broad | Top result limit too small or filter missing | Admin workstation | Use filters and date windows | Query by category, activity, or object |
| Service health portal unavailable | Missing Service Support Administrator or Global Reader | Admin workstation | M365 admin center role review | Assign appropriate reader role |
| Message Center unavailable | Missing Message Center Reader or Global Reader | Admin workstation | M365 admin center role review | Assign Message Center Reader |
| Graph service announcement query fails | Missing ServiceMessage.Read.All | Admin workstation | Get-MgContext | Reconnect with ServiceMessage.Read.All and consent |
| Azure Activity Log fails | Missing Azure Reader at subscription scope | Admin workstation | Get-AzContext; Get-AzRoleAssignment | Assign Reader at target scope if approved |
| Azure Activity Log shows wrong subscription | Wrong Azure context | Admin workstation | Get-AzContext | Set-AzContext to correct subscription |
| No Azure Activity Log events returned | Time window too small or wrong subscription | Admin workstation | Get-AzLog -StartTime | Expand time window and verify subscription |
| Diagnostic settings command fails | Resource does not support diagnostic settings or missing permission | Admin workstation | Get-AzDiagnosticSettingCategory | Confirm supported categories and role |
| Diagnostic category invalid | Category name differs by resource type | Admin workstation | Get-AzDiagnosticSettingCategory | Use valid category names |
| Log Analytics workspace creation fails | Missing Contributor, invalid region, naming conflict | Admin workstation | Get-AzContext | Fix role, region, or name |
| Diagnostic setting points to wrong workspace | Wrong workspace resource ID | Admin workstation | Get-AzDiagnosticSetting | Update diagnostic setting to correct workspace |
| Alert does not fire | Alert rule missing, wrong scope, wrong query, wrong action group | Azure portal | Alert rule review | Fix scope/query/action group |
| Message Center items not assigned | No review workflow | M365 admin center | Message Center tracking table | Assign owner and status |
| Service health issue missed | No daily review or alert | M365 admin center / Azure Service Health | Service health review | Configure review cadence and alerts |
| Audit export contains sensitive data | Evidence stored insecurely | Admin workstation | Review output path | Move to secure location or delete |
| Portal and Graph disagree | Wrong tenant, stale portal session, or permission difference | Admin workstation | Get-MgContext and portal tenant selector | Align tenant and account |
| Timestamps are confusing | Time zone mismatch | Admin workstation | Record time zone | Normalize to UTC plus local time |
| Change log not trusted | Manual changes not recorded consistently | Admin workstation | Change log table | Make change log mandatory for admin work |
| Purview audit unavailable | Missing Purview role or audit not available | Purview portal | Role group review | Assign audit role and validate licensing |
| Defender alerts unavailable | Missing security role or Defender license | Defender portal | Role/license review | Assign Security Reader/Admin or validate license |
| M365 audit search unavailable | Missing audit role or service limitation | Purview portal | Audit role review | Assign Audit Reader or Compliance role |
| Activity log retention too short | Default retention not enough for investigation | Azure portal | Diagnostic settings to workspace | Export to Log Analytics or storage |
| Costs unexpectedly increase | Diagnostic settings send high-volume logs | Azure portal | Workspace usage/cost review | Tune categories, retention, and commitment |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Documentation_OUTPUT_TEMPLATE

## Audit and Health Baseline Summary

| Field | Value |
|---|---|
| Tenant display name |  |
| Tenant ID |  |
| Primary domain |  |
| Subscription name |  |
| Subscription ID |  |
| Admin account used |  |
| Evidence path |  |
| Review time zone |  |
| Date collected |  |

## Entra Audit Baseline

| Item | Status | Notes |
|---|---|---|
| Directory audit logs accessible | Yes / No |  |
| Sign-in logs accessible | Yes / No |  |
| Emergency account sign-ins reviewed | Yes / No |  |
| Directory role members exported | Yes / No |  |
| Critical role members exported | Yes / No |  |
| Conditional Access changes reviewed | Yes / No |  |
| User/group/license changes reviewed | Yes / No |  |
| Audit retention understood | Yes / No |  |

## Azure Monitoring Baseline

| Item | Status | Notes |
|---|---|---|
| Azure Activity Log accessible | Yes / No |  |
| RBAC changes exported | Yes / No |  |
| Policy changes exported | Yes / No |  |
| Diagnostic settings inventoried | Yes / No |  |
| Log Analytics workspace exists | Yes / No |  |
| Workspace name |  |  |
| Workspace resource group |  |  |
| Workspace region |  |  |
| Workspace retention |  |  |
| Azure Service Health reviewed | Yes / No |  |
| Azure health alerts configured | Yes / No |  |

## Microsoft 365 Health Baseline

| Item | Status | Notes |
|---|---|---|
| Service health portal accessible | Yes / No |  |
| Active service issues reviewed | Yes / No |  |
| Message Center accessible | Yes / No |  |
| Message Center items reviewed | Yes / No |  |
| Message Center owner assigned | Yes / No |  |
| Service communications Graph export works | Yes / No |  |
| Service health review cadence documented | Yes / No |  |
| Planned change workflow documented | Yes / No |  |

## Critical Monitoring Targets

| Target | Source | Alert / Review | Owner | Notes |
|---|---|---|---|---|
| Emergency access account sign-in | Entra sign-in logs | Immediate alert |  |  |
| Global Administrator assignment | Entra audit logs | Immediate alert/review |  |  |
| Privileged Role Administrator assignment | Entra audit logs | Immediate alert/review |  |  |
| Conditional Access policy change | Entra audit logs | Immediate alert/review |  |  |
| Authentication method policy change | Entra audit logs | Daily review |  |  |
| Azure Owner assignment | Azure Activity Log | Immediate alert/review |  |  |
| Azure User Access Administrator assignment | Azure Activity Log | Immediate alert/review |  |  |
| Diagnostic setting removed | Azure Activity Log | Daily review |  |  |
| Message Center major change | M365 Message Center | Weekly review |  |  |
| M365 active incident | Service health | Daily review |  |  |

## Service Health Tracking Table

| Issue ID | Platform | Service | Status | Impact | Start Time | Last Update | Owner | Internal Ticket | Notes |
|---|---|---|---|---|---|---|---|---|---|
|  | M365 / Azure |  | Active / Advisory / Resolved |  |  |  |  |  |  |

## Message Center Tracking Table

| Message ID | Title | Service | Category | Published | Action By | Owner | Status | Internal Ticket | Notes |
|---|---|---|---|---|---|---|---|---|---|
|  |  |  | Plan / Prevent / Stay informed |  |  |  | New / Reviewing / Actioned / Closed |  |  |

## Local Change Log

| Change ID | Date / Time | Change Type | Plane | Object | Actor | Approval / Ticket | Expected Result | Actual Result | Rollback Needed | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
|  |  |  | Entra / Azure / M365 / Defender / Purview |  |  |  |  |  | Yes / No |  |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories | Supplies tenant, directory, and subscription context |
| 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline | Supplies Graph, Az PowerShell, Azure CLI, and portal tooling |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces | Domain verification and DNS changes should be tracked |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles | Role changes and role assignments must be monitored |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline | Emergency access and privileged changes must be monitored |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline | User, group, guest, license, and SSPR changes must be audited |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Uses logs and service health to troubleshoot access and portal problems |
| Azure Governance RBAC Workbook | Uses Azure Activity Log, role assignment exports, policy events, and management group tracking |
| Azure Monitor Backup Recovery Workbook | Deep dive for Log Analytics, KQL, alerts, action groups, workbooks, and resource health |
| M365 Tenant Administration Workbook | Deep dive for Service health, Message Center, users, groups, licensing, and admin roles |
| Exchange Online Administration Workbook | Uses workload audit, mail flow changes, admin role changes, and service health |
| SharePoint OneDrive Workbook | Uses sharing changes, site changes, service health, and audit |
| Teams Administration Workbook | Uses Teams policy changes, guest/external access changes, and service health |
| Defender XDR Workbook | Uses incidents, alerts, advanced hunting, and security role monitoring |
| Purview Workbook | Uses audit, DLP, retention, eDiscovery, and compliance change tracking |
| Intune Workbook | Uses device compliance, configuration policy changes, enrollment issues, and service health |
| Hybrid Identity Workbook | Uses sign-in logs, sync errors, authentication events, and Conditional Access results |

---

# 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking_Completion_Criteria

| Requirement | Complete |
|---|---|
| Tenant ID confirmed |  |
| Subscription ID confirmed |  |
| Graph audit/report scopes validated |  |
| Entra audit logs accessible |  |
| Entra sign-in logs accessible |  |
| Emergency access account sign-ins reviewed |  |
| Directory role members exported |  |
| Critical role members exported |  |
| User/group/license audit sample exported |  |
| Conditional Access change review process documented |  |
| Microsoft 365 Service health reviewed |  |
| Microsoft 365 Message Center reviewed |  |
| Message Center tracking table created |  |
| Service health tracking table created |  |
| Service communications Graph export tested if permitted |  |
| Azure Activity Log exported |  |
| Azure RBAC changes exported |  |
| Azure diagnostic setting changes exported |  |
| Azure Service Health reviewed |  |
| Log Analytics workspace documented |  |
| Diagnostic settings inventory exported |  |
| Critical change table completed |  |
| Emergency sign-in alert requirement documented |  |
| Role change alert requirement documented |  |
| Message Center owner assigned |  |
| Review cadence documented |  |
| Local change log created |  |
| Evidence stored securely |  |
| Rollback notes completed |  |

## Final Expected State

The admin can clearly state:

- These are the audit sources for identity, Azure, Microsoft 365, Defender, and Purview.
- Entra audit logs are accessible.
- Entra sign-in logs are accessible.
- Emergency account sign-ins are reviewed and should alert.
- Critical role changes are reviewed.
- Conditional Access changes are reviewed.
- Azure Activity Log is accessible for the target subscription.
- Azure RBAC changes can be tracked.
- Diagnostic settings coverage is documented.
- Microsoft 365 Service health is reviewed on a defined cadence.
- Microsoft 365 Message Center is reviewed on a defined cadence.
- Planned Microsoft changes have owners and statuses.
- Admin changes are recorded in a local change log.
- Evidence exports are stored securely.
- Later troubleshooting workbooks can rely on known audit, health, and change tracking locations.