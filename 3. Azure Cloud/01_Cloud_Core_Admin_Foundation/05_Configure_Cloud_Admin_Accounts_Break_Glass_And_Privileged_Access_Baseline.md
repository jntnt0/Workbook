# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Index

## Purpose

Configure a baseline for cloud admin identities, emergency access accounts, privileged access, and privileged role governance.

This workbook covers:

- Cloud admin account model
- Named admin accounts
- Emergency access / break glass accounts
- Privileged role assignment planning
- Privileged Identity Management planning
- Global Administrator reduction
- Role separation
- Azure RBAC privileged access
- Microsoft Entra privileged access
- Microsoft 365 privileged access
- Workload admin privilege
- Conditional Access exclusions for emergency access
- Authentication method planning
- Sign-in monitoring
- Alerting and audit checks
- Access review planning
- Password and credential handling
- Recovery and rollback procedures

## Outcome

By the end of this workbook, the admin should be able to:

- Identify all high privilege cloud roles
- Create a documented privileged account model
- Separate daily-use accounts from admin accounts
- Create or validate emergency access accounts
- Assign emergency access accounts only the roles needed for recovery
- Document Conditional Access exclusions for emergency access
- Configure monitoring for emergency access sign-in
- Identify permanent vs eligible role assignments
- Reduce unnecessary standing privileged access
- Prepare for PIM, access reviews, and privileged access governance
- Validate that cloud admin access can survive MFA, CA, federation, domain, and sync failures

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Source_Basis

| Source | Relevant Area | What It Supports |
|---|---|---|
| Microsoft Entra documentation | Users, roles, emergency access, Conditional Access, PIM, authentication methods, audit and sign-in logs | Identity privileged access baseline |
| Azure management documentation | Azure RBAC, subscription Owner, User Access Administrator, management group scope, activity logs | Azure privileged access baseline |
| Microsoft 365 documentation fork | Microsoft 365 admin roles, service health, Message Center, billing, licenses, admin center roles | M365 privileged access baseline |
| Microsoft Graph documentation | User inventory, role assignments, directory roles, role management, audit inventory | Graph-based validation |
| Defender documentation | Security roles, alerts, incidents, secure score, identity protection handoff | Security monitoring |
| Purview documentation | Audit, compliance roles, retention, eDiscovery, DLP, privileged audit requirements | Compliance monitoring |
| Existing Workbook 01 | Tenant, subscription, directory, and domain comparison | Supplies target tenant and subscription context |
| Existing Workbook 02 | Admin portals, PowerShell, Azure CLI, and Graph baseline | Supplies tooling required for privileged access inventory |
| Existing Workbook 03 | Custom domains and tenant namespaces | Supplies safe namespace design for admin and break glass accounts |
| Existing Workbook 04 | Azure RBAC, Entra roles, M365 roles, workload roles comparison | Supplies role plane decision model |
| Existing Workbook structure | Source basis, mental model, planning, checklist, skeletons, verification, rollback, failure checks | Obsidian-ready workbook format |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Mental_Model

| Concept | Plain Meaning | Admin Boundary |
|---|---|---|
| Daily-use account | Normal user account for email, Teams, browsing, and routine work | Low privilege user plane |
| Named admin account | Separate account used by a specific admin for privileged work | Admin identity plane |
| Emergency access account | Cloud-only account reserved for tenant recovery | Break glass plane |
| Break glass account | Same concept as emergency access account | Recovery plane |
| Standing privilege | Role is always active | Higher risk |
| Eligible privilege | Role must be activated before use | Lower risk when governed |
| Privileged Identity Management | Just-in-time activation, approval, duration, justification, and audit | Entra governance plane |
| Global Administrator | Highest broad tenant role | Critical identity privilege |
| Privileged Role Administrator | Can manage Entra role assignments | Critical delegation privilege |
| Azure Owner | Full Azure resource and RBAC control at assigned scope | Critical Azure privilege |
| User Access Administrator | Can assign Azure RBAC at assigned scope | Critical Azure delegation privilege |
| Conditional Access exclusion | Account, group, or role excluded from a policy | Lockout prevention and risk |
| Authentication strength | Required auth method combination for sign-in | Access control plane |
| Privileged access group | Group used to assign or activate privileged roles | Group-based privilege model |
| Admin unit | Scoped directory admin boundary | Delegated identity admin scope |
| Audit trail | Evidence of privileged assignment, activation, and sign-in | Governance plane |
| Access review | Periodic validation of who still needs access | Governance plane |

## Core Rule

Privileged access must be survivable, auditable, and least privilege.

The goal is not to make every admin a Global Administrator.

The goal is to make sure:

- Normal work uses normal accounts
- Admin work uses named admin accounts
- Emergency recovery uses dedicated emergency access accounts
- High privilege is time-bound when possible
- Emergency access bypasses lockout conditions but is heavily monitored
- Role assignment matches the actual admin task

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Account_Model

| Account Type | Example | Use | Privilege Model | Notes |
|---|---|---|---|---|
| Daily user account | jsmith@contoso.com | Email, Teams, web, normal work | No admin roles | Should not hold privileged roles |
| Named cloud admin account | adm-jsmith@contoso.com | Admin portals, PowerShell, Graph, CLI | Eligible roles preferred | Used by one human admin |
| Workload admin account | exo-jsmith@contoso.com or adm-jsmith@contoso.com | Exchange, SharePoint, Teams, Intune, Defender, Purview | Workload roles | Can be same named admin account depending on policy |
| Automation identity | spn-platform-deploy | Scripts, pipelines, app-only automation | Least privilege app permissions / Azure RBAC | Must not be shared with humans |
| Managed identity | mi-app-prod | Azure resource runtime identity | Azure RBAC scoped to resource need | No password or secret |
| Emergency access account 1 | breakglass01@tenant.onmicrosoft.com | Tenant recovery | Standing Global Administrator or approved emergency role set | Cloud-only, monitored, protected |
| Emergency access account 2 | breakglass02@tenant.onmicrosoft.com | Tenant recovery backup | Standing Global Administrator or approved emergency role set | Separate credential storage path |
| Vendor admin account | adm-vendor-name@contoso.com | Temporary vendor support | Time-bound, scoped access | Must have expiration and review |
| Service desk admin | adm-helpdesk01@contoso.com | Password reset, user support | Helpdesk/User Administrator scoped | Avoid broad roles |
| Security operator | adm-secops01@contoso.com | Defender incident response | Security Reader / Security Operator / Security Administrator | Separate from identity admin if possible |
| Compliance admin | adm-compliance01@contoso.com | Purview, eDiscovery, audit | Compliance roles | Strong audit requirement |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Privileged_Role_Model

| Role | Plane | Standing Assignment | Eligible Assignment | Emergency Access Use | Notes |
|---|---|---|---|---|---|
| Global Administrator | Entra / M365 | Avoid except emergency and minimal core admins | Preferred for named admins | Yes, usually required for tenant recovery | Critical role |
| Privileged Role Administrator | Entra | Avoid standing | Preferred for role admins | Optional depending recovery design | Can assign Entra roles |
| Conditional Access Administrator | Entra | Avoid standing | Preferred | Optional depending recovery design | Can lock out users if misused |
| Authentication Policy Administrator | Entra | Avoid standing | Preferred | Optional | Manages auth methods |
| Security Administrator | Security / Entra | Avoid standing unless SecOps requires | Preferred | Optional | Can manage security settings |
| Compliance Administrator | Purview / M365 | Avoid standing unless required | Preferred | Usually no | Compliance boundary |
| Exchange Administrator | Exchange / M365 | Workload dependent | Preferred | Usually no | Mail administration |
| SharePoint Administrator | SharePoint / M365 | Workload dependent | Preferred | Usually no | Sharing and sites |
| Teams Administrator | Teams / M365 | Workload dependent | Preferred | Usually no | Teams policies |
| Intune Administrator | Intune / M365 | Workload dependent | Preferred | Usually no | Endpoint control |
| Billing Administrator | M365 billing | Workload dependent | Preferred if available | Usually no | Financial control |
| License Administrator | M365 / Entra | Workload dependent | Preferred | Usually no | License assignment |
| Azure Owner | Azure RBAC | Avoid broad standing | Preferred | Optional at root/subscription depending design | Resource and RBAC control |
| User Access Administrator | Azure RBAC | Avoid standing | Preferred | Optional | Azure RBAC delegation |
| Contributor | Azure RBAC | Scoped as needed | Eligible for sensitive scopes | Usually no | Cannot assign RBAC |
| Reader / Global Reader | Azure / Entra / M365 | Acceptable for baseline visibility | Optional | No | Useful low-risk read access |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Emergency_Access_Design

| Design Item | Recommendation | Reason |
|---|---|---|
| Number of emergency accounts | At least two | Prevents single-account failure |
| Account source | Cloud-only | Avoids dependency on on-premises sync or federation |
| Domain suffix | Initial onmicrosoft.com domain preferred | Survives custom domain, DNS, and federation issues |
| Assignment | Direct role assignment, not only group if group/PIM dependency is risky | Reduces dependency chain during emergency |
| Role | Global Administrator or approved emergency recovery role set | Allows tenant recovery |
| Daily use | Never | Preserves account integrity |
| Mailbox | Avoid normal mailbox use | Reduces exposure |
| License | Only if required for monitoring or policy | Avoid unnecessary service footprint |
| MFA | Must be deliberate and survivable | Do not depend on a method likely to fail during lockout |
| Conditional Access | Exclude from policies that could block emergency access | Prevents tenant lockout |
| Password | Long, unique, randomly generated | High entropy |
| Password storage | Offline, sealed, access-controlled, tested | Do not store in workbook |
| Monitoring | Alert on every sign-in attempt | Emergency use should be rare |
| Review cadence | Regular scheduled test and access review | Confirms account still works |
| Ownership | Named executive/security/identity owner | Clear accountability |
| Documentation | Store location and recovery steps in secure runbook | Do not include secret values |

## Critical Note

Never store emergency access passwords, recovery codes, FIDO keys, TAP values, or secrets in this workbook.

Document only:

- Account name
- Owner
- Storage location reference
- Last test date
- Monitoring status
- Role assignment
- Conditional Access exclusion status

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Privileged_Access_Planning_Table

| Item | Value To Record | Why It Matters |
|---|---|---|
| Tenant ID |  | Prevents configuring wrong tenant |
| Initial domain |  | Best namespace for emergency accounts |
| Primary custom domain |  | Normal admin namespace |
| Admin naming standard |  | Keeps admin identities recognizable |
| Daily-use account rule |  | Prevents privilege on normal accounts |
| Named admin account list |  | Maps humans to admin identities |
| Emergency access account 1 |  | Primary recovery account |
| Emergency access account 2 |  | Secondary recovery account |
| Emergency account owners |  | Accountability |
| Emergency password storage location |  | Recovery process |
| Emergency MFA method |  | Survivability |
| Emergency CA exclusions |  | Lockout prevention |
| Global Administrators |  | Critical role inventory |
| Privileged Role Administrators |  | Critical delegation role inventory |
| Azure Owners |  | Critical Azure control |
| User Access Administrators |  | Critical Azure RBAC delegation |
| PIM enabled |  | Determines eligible vs standing privilege |
| PIM approval required |  | Governance and control |
| PIM max activation duration |  | Limits exposure |
| Justification required |  | Audit quality |
| Ticket number required |  | Change tracking |
| Access reviews enabled |  | Periodic privilege validation |
| Sign-in alert destination |  | Emergency use detection |
| Audit log retention |  | Investigation readiness |
| Break glass test cadence |  | Confirms recoverability |
| Last successful test date |  | Evidence |
| Known lockout dependencies |  | Reduces failure points |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Emergency_Account_Table

| Account | Object Type | Source | Domain | Role Assignment | CA Exclusion | MFA / Auth Method | Password Storage Reference | Owner | Last Tested | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| breakglass01@tenant.onmicrosoft.com | User | Cloud-only | onmicrosoft.com | Global Administrator | Yes | Approved emergency method | Secure vault reference only |  |  | Active / Needs Review |
| breakglass02@tenant.onmicrosoft.com | User | Cloud-only | onmicrosoft.com | Global Administrator | Yes | Approved emergency method | Secure vault reference only |  |  | Active / Needs Review |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Privileged_Admin_Account_Table

| Admin Human | Daily Account | Named Admin Account | Admin Role Type | Standing Roles | Eligible Roles | PIM Required | Notes |
|---|---|---|---|---|---|---|---|
|  | user@contoso.com | adm-user@contoso.com | Identity admin |  |  | Yes / No |  |
|  | user@contoso.com | adm-user@contoso.com | Azure admin |  |  | Yes / No |  |
|  | user@contoso.com | adm-user@contoso.com | M365 admin |  |  | Yes / No |  |
|  | user@contoso.com | adm-user@contoso.com | Security admin |  |  | Yes / No |  |
|  | user@contoso.com | adm-user@contoso.com | Compliance admin |  |  | Yes / No |  |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Conditional_Access_Emergency_Design

| Policy Area | Emergency Access Treatment | Notes |
|---|---|---|
| Require MFA for all users | Exclude emergency access accounts only | Emergency auth method must still be protected |
| Require compliant device | Exclude emergency access accounts | Device compliance failure should not block recovery |
| Block legacy authentication | Usually include emergency accounts if modern auth works | Avoid allowing weak auth unless justified |
| Block risky sign-in | Review carefully | Risk policy can block recovery if false positive |
| Block by location | Exclude emergency accounts or allow recovery location | Location-based blocks can lock out recovery |
| Require approved client app | Exclude emergency accounts | Recovery may need browser access |
| Require app protection policy | Exclude emergency accounts | Recovery may need unmanaged endpoint |
| Session controls | Exclude or relax for emergency accounts | Avoid sign-in loop |
| Terms of use | Exclude emergency accounts | Avoid dependency during outage |
| Authentication strength | Use a survivable emergency method | Do not depend on single device or single person |
| Named locations | Document emergency access locations | Helps alerting and review |
| Report-only policy | Include for visibility when safe | Useful before enforcement |
| Alerting | Alert on all emergency account sign-ins | Required monitoring control |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm workbook 01 tenant baseline | Admin workstation | Review tenant ID and initial domain | Correct tenant and namespace are known |
| 2 | Confirm workbook 02 tooling baseline | Admin workstation | Get-AzContext; Get-MgContext; az account show | Tooling is connected to correct tenant |
| 3 | Confirm workbook 04 role plane baseline | Admin workstation | Review role plane map | Correct role systems are understood |
| 4 | Inventory current users | Admin workstation | Get-MgUser -All | User list exported |
| 5 | Inventory current Global Administrators | Admin workstation | Get-MgDirectoryRoleMember | Global Administrators documented |
| 6 | Inventory current Privileged Role Administrators | Admin workstation | Get-MgDirectoryRoleMember | PRA members documented |
| 7 | Inventory Azure Owners and User Access Administrators | Admin workstation | Get-AzRoleAssignment | Azure privileged access documented |
| 8 | Define admin account naming standard | Admin workstation | Workbook planning table | Standard such as adm-firstlast is documented |
| 9 | Identify daily accounts with admin roles | Admin workstation | Compare role inventory to account list | Normal accounts holding privilege are flagged |
| 10 | Create named admin accounts if needed | Entra admin center / Graph | New-MgUser | Separate admin accounts exist |
| 11 | Create emergency access account 1 if missing | Entra admin center / Graph | New-MgUser | Primary emergency account exists |
| 12 | Create emergency access account 2 if missing | Entra admin center / Graph | New-MgUser | Secondary emergency account exists |
| 13 | Confirm emergency accounts are cloud-only | Admin workstation | Get-MgUser | Accounts are not synced from on-premises |
| 14 | Assign emergency access roles | Entra admin center | Roles & admins > Global Administrator > Add assignment | Emergency accounts have approved recovery role |
| 15 | Confirm emergency accounts use stable namespace | Admin workstation | Get-MgUser | Accounts use onmicrosoft.com or approved stable namespace |
| 16 | Configure emergency account credentials | Secure offline process | Do not record secret in workbook | Credentials stored securely outside workbook |
| 17 | Configure emergency account authentication method | Entra admin center | Protection > Authentication methods | Survivable auth method documented |
| 18 | Create Conditional Access exclusion group if design uses group | Entra admin center / Graph | New-MgGroup | Exclusion group exists |
| 19 | Add emergency accounts to exclusion group if approved | Entra admin center / Graph | New-MgGroupMember | Emergency accounts excluded from lockout policies |
| 20 | Exclude emergency accounts from blocking CA policies | Entra admin center | Conditional Access policy users/exclusions | Recovery accounts are not blocked |
| 21 | Validate no unnecessary licenses assigned | Admin workstation | Get-MgUserLicenseDetail | Emergency accounts only have required licenses |
| 22 | Configure sign-in alerting | Entra / Defender / Azure Monitor | Alert rule or workbook notes | Emergency account sign-ins generate alert |
| 23 | Configure audit review process | Admin workstation | Workbook notes | Review cadence documented |
| 24 | Configure PIM eligible assignments for named admins | Entra admin center | PIM > Microsoft Entra roles | Named admins use eligible roles where possible |
| 25 | Configure PIM eligible Azure RBAC assignments | Azure portal | PIM > Azure resources | Azure privileged roles use eligibility where possible |
| 26 | Set PIM activation settings | Entra admin center | Approval, duration, justification, ticket | Activation governance is documented |
| 27 | Remove unnecessary standing admin roles only after approval | Admin workstation / portal | Role removal command or portal | Standing privilege reduced safely |
| 28 | Validate emergency account sign-in test | Controlled test | Browser private session | Emergency account can sign in |
| 29 | Validate emergency account monitoring | Security / audit portal | Check sign-in logs and alert | Sign-in event is visible |
| 30 | Export final privileged access baseline | Admin workstation | Run export skeleton | Evidence saved securely |
| 31 | Save final workbook | Admin workstation | Commit note to workbook repo or Obsidian vault | Baseline ready for later labs |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Precheck_Skeleton

```powershell
# ============================================================
# Privileged Access Baseline Precheck
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$OutputRoot = ".\Privileged_Access_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

# -----------------------------
# Connect Microsoft Graph
# -----------------------------
Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "User.Read.All", `
    "Group.Read.All", `
    "RoleManagement.Read.Directory", `
    "AuditLog.Read.All"

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
# Confirm domains
# -----------------------------
$Domains = Get-MgDomain -All

$Domains |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsInitial -Descending, IsDefault -Descending, Id |
    Format-Table -AutoSize

$Domains |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsInitial -Descending, IsDefault -Descending, Id |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Domains.csv")

# -----------------------------
# Connect Azure PowerShell
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

Write-Host "Precheck complete. Output path: $OutputPath"
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_User_And_Role_Inventory_Skeleton

```powershell
# ============================================================
# User and Privileged Role Inventory Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$OutputRoot = ".\Privileged_Access_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "User.Read.All", `
    "Group.Read.All", `
    "RoleManagement.Read.Directory", `
    "AuditLog.Read.All"

# -----------------------------
# User inventory
# -----------------------------
$Users = Get-MgUser -All -Property `
    Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,CreatedDateTime,OnPremisesSyncEnabled,AssignedLicenses

$Users |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, OnPremisesSyncEnabled, CreatedDateTime, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Users.csv")

# -----------------------------
# Potential admin account pattern inventory
# Adjust naming patterns to match organization standard
# -----------------------------
$PotentialAdminAccounts = $Users |
    Where-Object {
        $_.UserPrincipalName -like "adm-*" -or
        $_.UserPrincipalName -like "admin-*" -or
        $_.UserPrincipalName -like "breakglass*" -or
        $_.UserPrincipalName -like "emergency*"
    }

$PotentialAdminAccounts |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, OnPremisesSyncEnabled, Id |
    Sort-Object UserPrincipalName |
    Format-Table -AutoSize

$PotentialAdminAccounts |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, OnPremisesSyncEnabled, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Potential_Admin_And_Emergency_Accounts.csv")

# -----------------------------
# Active directory roles
# -----------------------------
$DirectoryRoles = Get-MgDirectoryRole -All

$DirectoryRoles |
    Select-Object DisplayName, Id, Description |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "DirectoryRoles.csv")

# -----------------------------
# Directory role members
# -----------------------------
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
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "DirectoryRoleMembers.csv")

# -----------------------------
# Critical role members
# -----------------------------
$CriticalRoles = @(
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
    "Cloud Application Administrator",
    "User Administrator",
    "Groups Administrator",
    "License Administrator",
    "Billing Administrator"
)

$CriticalRoleMembers = $DirectoryRoleMembers |
    Where-Object {$_.RoleDisplayName -in $CriticalRoles}

$CriticalRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Format-Table -AutoSize

$CriticalRoleMembers |
    Sort-Object RoleDisplayName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "CriticalDirectoryRoleMembers.csv")

# -----------------------------
# Daily accounts with privileged roles
# Adjust pattern logic to fit the account naming standard
# -----------------------------
$PrivilegedUserUpns = $CriticalRoleMembers |
    Where-Object {$_.UserPrincipalName} |
    Select-Object -ExpandProperty UserPrincipalName -Unique

$PotentialDailyAccountsWithPrivilege = $PrivilegedUserUpns |
    Where-Object {
        $_ -notlike "adm-*" -and
        $_ -notlike "admin-*" -and
        $_ -notlike "breakglass*" -and
        $_ -notlike "emergency*"
    }

$PotentialDailyAccountsWithPrivilege |
    Out-File -FilePath (Join-Path $OutputPath "Potential_Daily_Accounts_With_Privilege.txt")

# -----------------------------
# Azure RBAC privileged assignments
# -----------------------------
Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

$SubscriptionScope = "/subscriptions/$SubscriptionId"

$AzureRoleAssignments = Get-AzRoleAssignment -Scope $SubscriptionScope

$AzureRoleAssignments |
    Select-Object DisplayName, SignInName, ObjectType, ObjectId, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzureRBAC_SubscriptionAssignments.csv")

$CriticalAzureRoles = @(
    "Owner",
    "User Access Administrator",
    "Contributor"
)

$CriticalAzureAssignments = $AzureRoleAssignments |
    Where-Object {$_.RoleDefinitionName -in $CriticalAzureRoles}

$CriticalAzureAssignments |
    Select-Object DisplayName, SignInName, ObjectType, ObjectId, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Format-Table -AutoSize

$CriticalAzureAssignments |
    Select-Object DisplayName, SignInName, ObjectType, ObjectId, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "AzureRBAC_CriticalAssignments.csv")

# -----------------------------
# Summary
# -----------------------------
$Summary = [PSCustomObject]@{
    TenantId                         = $TenantId
    SubscriptionId                   = $SubscriptionId
    TotalUsers                       = $Users.Count
    PotentialAdminAccounts           = $PotentialAdminAccounts.Count
    ActiveDirectoryRoles             = $DirectoryRoles.Count
    CriticalDirectoryRoleAssignments = $CriticalRoleMembers.Count
    GlobalAdministratorAssignments   = ($CriticalRoleMembers | Where-Object {$_.RoleDisplayName -eq "Global Administrator"}).Count
    PrivilegedRoleAdminAssignments   = ($CriticalRoleMembers | Where-Object {$_.RoleDisplayName -eq "Privileged Role Administrator"}).Count
    AzureOwnerAssignments            = ($CriticalAzureAssignments | Where-Object {$_.RoleDefinitionName -eq "Owner"}).Count
    AzureUserAccessAdminAssignments  = ($CriticalAzureAssignments | Where-Object {$_.RoleDefinitionName -eq "User Access Administrator"}).Count
    DateCollected                    = Get-Date
}

$Summary |
    Format-List

$Summary |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "PrivilegedAccessSummary.csv")

Write-Host "Inventory complete. Output path: $OutputPath"
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Create_Named_Admin_Account_Skeleton

```powershell
# ============================================================
# Create Named Admin Account Skeleton
# Review and customize before use
# Do not store passwords in this workbook
# ============================================================

$TenantId = "<tenant-id>"
$AdminUpn = "adm-jsmith@contoso.com"
$AdminDisplayName = "ADM John Smith"
$MailNickname = "adm-jsmith"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "User.ReadWrite.All", `
    "Directory.ReadWrite.All"

# Generate password outside this workbook and store it in an approved secure location.
# This placeholder intentionally does not include a real password.
$PasswordProfile = @{
    forceChangePasswordNextSignIn = $true
    password = "<temporary-password-from-secure-process>"
}

# Create named admin account
New-MgUser `
    -AccountEnabled:$true `
    -DisplayName $AdminDisplayName `
    -MailNickname $MailNickname `
    -UserPrincipalName $AdminUpn `
    -PasswordProfile $PasswordProfile

# Verify account
Get-MgUser -UserId $AdminUpn -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,OnPremisesSyncEnabled |
    Select-Object Id, DisplayName, UserPrincipalName, UserType, AccountEnabled, OnPremisesSyncEnabled |
    Format-List

# Important:
# Assign roles separately through PIM or approved role assignment process.
# Do not assign Global Administrator by default.
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Create_Emergency_Access_Account_Skeleton

```powershell
# ============================================================
# Create Emergency Access Account Skeleton
# Review and customize before use
# Do not store passwords in this workbook
# ============================================================

$TenantId = "<tenant-id>"
$EmergencyUpn = "breakglass01@tenant.onmicrosoft.com"
$EmergencyDisplayName = "Emergency Access 01"
$MailNickname = "breakglass01"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "User.ReadWrite.All", `
    "Directory.ReadWrite.All", `
    "RoleManagement.ReadWrite.Directory"

# Generate password outside this workbook and store it in an approved secure offline location.
# This placeholder intentionally does not include a real password.
$PasswordProfile = @{
    forceChangePasswordNextSignIn = $false
    password = "<long-random-password-from-secure-process>"
}

# Create emergency account
New-MgUser `
    -AccountEnabled:$true `
    -DisplayName $EmergencyDisplayName `
    -MailNickname $MailNickname `
    -UserPrincipalName $EmergencyUpn `
    -PasswordProfile $PasswordProfile

# Verify account
Get-MgUser -UserId $EmergencyUpn -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,OnPremisesSyncEnabled |
    Select-Object Id, DisplayName, UserPrincipalName, UserType, AccountEnabled, OnPremisesSyncEnabled |
    Format-List

# Important:
# Assign emergency role separately after approval.
# Configure Conditional Access exclusion separately.
# Configure monitoring separately.
# Test sign-in in a controlled window.
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Assign_Entra_Role_Skeleton

```powershell
# ============================================================
# Assign Entra Directory Role Skeleton
# Prefer PIM for named admins when available.
# Use direct assignment only when approved.
# ============================================================

$TenantId = "<tenant-id>"
$UserUpn = "breakglass01@tenant.onmicrosoft.com"
$RoleName = "Global Administrator"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Directory.Read.All", `
    "RoleManagement.ReadWrite.Directory", `
    "User.Read.All"

# Get user
$User = Get-MgUser -UserId $UserUpn

# Get active directory role
$Role = Get-MgDirectoryRole -All | Where-Object {$_.DisplayName -eq $RoleName}

# If role is not activated in directory roles, activate from template
if (-not $Role) {
    $Template = Get-MgDirectoryRoleTemplate -All | Where-Object {$_.DisplayName -eq $RoleName}
    Enable-MgDirectoryRole -DirectoryRoleTemplateId $Template.Id
    Start-Sleep -Seconds 10
    $Role = Get-MgDirectoryRole -All | Where-Object {$_.DisplayName -eq $RoleName}
}

# Assign role
# Use only after approval
New-MgDirectoryRoleMemberByRef `
    -DirectoryRoleId $Role.Id `
    -BodyParameter @{
        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
    }

# Verify role membership
Get-MgDirectoryRoleMember -DirectoryRoleId $Role.Id -All |
    Select-Object Id, AdditionalProperties |
    Format-List
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Remove_Entra_Role_Skeleton

```powershell
# ============================================================
# Remove Entra Directory Role Skeleton
# Use only after confirming role removal will not cause lockout.
# ============================================================

$TenantId = "<tenant-id>"
$UserUpn = "adm-user@contoso.com"
$RoleName = "Global Administrator"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Directory.Read.All", `
    "RoleManagement.ReadWrite.Directory", `
    "User.Read.All"

$User = Get-MgUser -UserId $UserUpn
$Role = Get-MgDirectoryRole -All | Where-Object {$_.DisplayName -eq $RoleName}

# Confirm current role members
Get-MgDirectoryRoleMember -DirectoryRoleId $Role.Id -All |
    Select-Object Id, AdditionalProperties |
    Format-List

# Remove role assignment by reference
# Use only after approval and emergency access validation
Remove-MgDirectoryRoleMemberByRef `
    -DirectoryRoleId $Role.Id `
    -DirectoryObjectId $User.Id

# Verify removal
Get-MgDirectoryRoleMember -DirectoryRoleId $Role.Id -All |
    Select-Object Id, AdditionalProperties |
    Format-List
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Azure_RBAC_Privileged_Assignment_Skeleton

```powershell
# ============================================================
# Azure RBAC Privileged Assignment Skeleton
# Prefer PIM for Azure resources where available.
# Use permanent assignment only when approved.
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$PrincipalUpn = "adm-jsmith@contoso.com"
$RoleDefinitionName = "Owner"
$Scope = "/subscriptions/$SubscriptionId"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# Confirm target principal
$Principal = Get-AzADUser -UserPrincipalName $PrincipalUpn

$Principal |
    Select-Object DisplayName, UserPrincipalName, Id |
    Format-List

# Assign Azure RBAC role
# Use only after approval
New-AzRoleAssignment `
    -ObjectId $Principal.Id `
    -RoleDefinitionName $RoleDefinitionName `
    -Scope $Scope

# Verify
Get-AzRoleAssignment -ObjectId $Principal.Id -Scope $Scope |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Format-Table -AutoSize
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Remove_Azure_RBAC_Privileged_Assignment_Skeleton

```powershell
# ============================================================
# Remove Azure RBAC Privileged Assignment Skeleton
# Use only after confirming removal will not cause access loss.
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$PrincipalUpn = "adm-jsmith@contoso.com"
$RoleDefinitionName = "Owner"
$Scope = "/subscriptions/$SubscriptionId"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

$Principal = Get-AzADUser -UserPrincipalName $PrincipalUpn

# Verify current assignment
Get-AzRoleAssignment -ObjectId $Principal.Id -Scope $Scope |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Format-Table -AutoSize

# Remove assignment
# Use only after approval and emergency access validation
Remove-AzRoleAssignment `
    -ObjectId $Principal.Id `
    -RoleDefinitionName $RoleDefinitionName `
    -Scope $Scope

# Verify removal
Get-AzRoleAssignment -ObjectId $Principal.Id -Scope $Scope |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Format-Table -AutoSize
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Conditional_Access_Exclusion_Group_Skeleton

```powershell
# ============================================================
# Conditional Access Emergency Exclusion Group Skeleton
# Group creation only.
# Policy exclusion must be reviewed in Entra portal or CA policy automation.
# ============================================================

$TenantId = "<tenant-id>"
$GroupDisplayName = "CA-EXCLUDE-Emergency-Access"
$GroupMailNickname = "ca-exclude-emergency-access"
$EmergencyAccounts = @(
    "breakglass01@tenant.onmicrosoft.com",
    "breakglass02@tenant.onmicrosoft.com"
)

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Group.ReadWrite.All", `
    "User.Read.All", `
    "Directory.ReadWrite.All"

# Create security group
$Group = New-MgGroup `
    -DisplayName $GroupDisplayName `
    -MailEnabled:$false `
    -MailNickname $GroupMailNickname `
    -SecurityEnabled:$true

# Add emergency accounts to group
foreach ($Account in $EmergencyAccounts) {
    $User = Get-MgUser -UserId $Account

    New-MgGroupMemberByRef `
        -GroupId $Group.Id `
        -BodyParameter @{
            "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
        }
}

# Verify members
Get-MgGroupMember -GroupId $Group.Id -All |
    Select-Object Id, AdditionalProperties |
    Format-List

# Important:
# Add this group as an exclusion only to approved Conditional Access policies.
# Keep membership extremely limited.
# Alert on membership changes.
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_PIM_Portal_Skeleton

```text
# ============================================================
# Privileged Identity Management Portal Skeleton
# ============================================================

Portal:
https://entra.microsoft.com

Navigation:
Identity governance > Privileged Identity Management

Microsoft Entra roles:
1. Open Privileged Identity Management.
2. Select Microsoft Entra roles.
3. Review Assignments.
4. Review Active assignments.
5. Review Eligible assignments.
6. Review Expired assignments.
7. Select Settings.
8. Review role settings for:
   - Global Administrator
   - Privileged Role Administrator
   - Conditional Access Administrator
   - Security Administrator
   - Compliance Administrator
   - Exchange Administrator
   - SharePoint Administrator
   - Teams Administrator
   - Intune Administrator
9. Configure activation duration.
10. Require justification.
11. Require ticket information if change process supports it.
12. Require approval for critical roles where practical.
13. Require MFA or authentication strength for activation.
14. Configure notification recipients.
15. Save settings.
16. Assign named admin accounts as eligible where possible.
17. Keep emergency access accounts direct and survivable if design requires.

Azure resources:
1. Open Privileged Identity Management.
2. Select Azure resources.
3. Select target subscription or management group.
4. Review active and eligible assignments.
5. Review Owner and User Access Administrator assignments.
6. Configure role settings.
7. Assign named admin accounts as eligible where possible.
8. Avoid broad standing Owner assignments.

Expected result:
Named admins use eligible roles when possible.
Standing high privilege is reduced.
Emergency access accounts remain recoverable and monitored.
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Emergency_Access_Test_Skeleton

```text
# ============================================================
# Emergency Access Account Test Skeleton
# ============================================================

Testing window:
Scheduled and approved

Participants:
- Identity admin owner
- Security monitoring owner
- Second approver or observer

Pre-test checks:
1. Confirm emergency account exists.
2. Confirm account is enabled.
3. Confirm account is cloud-only.
4. Confirm account has approved emergency role.
5. Confirm account is excluded from blocking CA policies.
6. Confirm password / auth method is available through secure process.
7. Confirm monitoring owner is watching sign-in logs.
8. Confirm test will not change production settings.

Test steps:
1. Open private browser session.
2. Go to https://entra.microsoft.com.
3. Sign in as emergency account.
4. Complete approved auth method.
5. Confirm portal access.
6. Confirm tenant ID.
7. Confirm role visibility.
8. Do not modify tenant configuration unless part of approved test.
9. Sign out.
10. Close private browser session.
11. Confirm sign-in log appears.
12. Confirm alert triggered.
13. Record test date, tester, and result.

Post-test:
1. Verify no configuration was changed.
2. Verify alert was received.
3. Verify sign-in logs show expected location and device details.
4. Update emergency access account table.
5. Store test evidence securely.

Expected result:
Emergency account can sign in and is detected by monitoring.
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Sign_In_And_Audit_Review_Skeleton

```powershell
# ============================================================
# Sign-In and Audit Review Skeleton
# Requires appropriate Entra ID licensing and permissions for sign-in logs.
# ============================================================

$TenantId = "<tenant-id>"
$EmergencyAccounts = @(
    "breakglass01@tenant.onmicrosoft.com",
    "breakglass02@tenant.onmicrosoft.com"
)

$OutputRoot = ".\Privileged_Access_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "AuditLog.Read.All", `
    "Directory.Read.All", `
    "User.Read.All"

# -----------------------------
# Review emergency account sign-ins
# -----------------------------
foreach ($Account in $EmergencyAccounts) {
    Write-Host "Reviewing sign-ins for $Account"

    $Filter = "userPrincipalName eq '$Account'"

    try {
        $SignIns = Get-MgAuditLogSignIn -Filter $Filter -Top 25

        $SignIns |
            Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, IpAddress, ClientAppUsed, ConditionalAccessStatus, Status |
            Format-Table -AutoSize

        $SignIns |
            Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, IpAddress, ClientAppUsed, ConditionalAccessStatus, Status |
            Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "$($Account.Replace('@','_').Replace('.','_'))_SignIns.csv")
    }
    catch {
        Write-Warning "Could not retrieve sign-ins for $Account"
        Write-Warning $_.Exception.Message
    }
}

# -----------------------------
# Review directory audit entries related to role assignments
# -----------------------------
try {
    $RoleAudit = Get-MgAuditLogDirectoryAudit -Top 50

    $RoleAudit |
        Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, Result |
        Format-Table -AutoSize

    $RoleAudit |
        Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, Result |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "DirectoryAudit_Recent.csv")
}
catch {
    Write-Warning "Could not retrieve directory audit logs."
    Write-Warning $_.Exception.Message
}
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Azure_CLI_Inventory_Skeleton

```bash
# ============================================================
# Azure CLI Privileged Access Inventory Skeleton
# ============================================================

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

mkdir -p Privileged_Access_Baseline

# Confirm context
az account show --output table

az account show \
  --output json > Privileged_Access_Baseline/az_account_show.json

# Azure RBAC critical assignments
az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --query "[?roleDefinitionName=='Owner' || roleDefinitionName=='User Access Administrator' || roleDefinitionName=='Contributor']" \
  --output table

az role assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  --include-inherited \
  --query "[?roleDefinitionName=='Owner' || roleDefinitionName=='User Access Administrator' || roleDefinitionName=='Contributor']" \
  --output json > Privileged_Access_Baseline/azure_critical_rbac_assignments.json

# Microsoft Graph organization check
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json > Privileged_Access_Baseline/graph_organization.json

# Microsoft Graph users with admin naming patterns
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/users?\$select=id,displayName,userPrincipalName,accountEnabled,userType,onPremisesSyncEnabled" \
  --output json > Privileged_Access_Baseline/graph_users_sample.json

# Directory roles
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/directoryRoles" \
  --output json > Privileged_Access_Baseline/graph_directory_roles.json
```

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Verification_Commands

```powershell
# ============================================================
# Verification Commands
# ============================================================

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$EmergencyAccounts = @(
    "breakglass01@tenant.onmicrosoft.com",
    "breakglass02@tenant.onmicrosoft.com"
)

# -----------------------------
# Verify Graph context
# -----------------------------
Get-MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

# -----------------------------
# Verify tenant
# -----------------------------
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

# -----------------------------
# Verify emergency accounts
# -----------------------------
foreach ($Account in $EmergencyAccounts) {
    Get-MgUser -UserId $Account -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,OnPremisesSyncEnabled |
        Select-Object Id, DisplayName, UserPrincipalName, UserType, AccountEnabled, OnPremisesSyncEnabled |
        Format-List
}

# -----------------------------
# Verify directory roles
# -----------------------------
Get-MgDirectoryRole -All |
    Select-Object DisplayName, Id |
    Sort-Object DisplayName |
    Format-Table -AutoSize

# -----------------------------
# Verify Global Administrator membership
# -----------------------------
$GlobalAdminRole = Get-MgDirectoryRole -All | Where-Object {$_.DisplayName -eq "Global Administrator"}

Get-MgDirectoryRoleMember -DirectoryRoleId $GlobalAdminRole.Id -All |
    Select-Object Id, AdditionalProperties |
    Format-List

# -----------------------------
# Verify Privileged Role Administrator membership
# -----------------------------
$PraRole = Get-MgDirectoryRole -All | Where-Object {$_.DisplayName -eq "Privileged Role Administrator"}

if ($PraRole) {
    Get-MgDirectoryRoleMember -DirectoryRoleId $PraRole.Id -All |
        Select-Object Id, AdditionalProperties |
        Format-List
}

# -----------------------------
# Verify Azure context
# -----------------------------
Get-AzContext |
    Select-Object Account, Tenant, Subscription, Environment |
    Format-List

# -----------------------------
# Verify Azure privileged assignments
# -----------------------------
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
    Where-Object {$_.RoleDefinitionName -in @("Owner","User Access Administrator","Contributor")} |
    Select-Object DisplayName, SignInName, ObjectType, RoleDefinitionName, Scope |
    Sort-Object RoleDefinitionName, DisplayName |
    Format-Table -AutoSize

# -----------------------------
# Verify emergency sign-ins if logs are available
# -----------------------------
foreach ($Account in $EmergencyAccounts) {
    try {
        Get-MgAuditLogSignIn -Filter "userPrincipalName eq '$Account'" -Top 10 |
            Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, IpAddress, ConditionalAccessStatus, Status |
            Format-Table -AutoSize
    }
    catch {
        Write-Warning "Could not read sign-in logs for $Account"
    }
}
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
  --query "[?roleDefinitionName=='Owner' || roleDefinitionName=='User Access Administrator' || roleDefinitionName=='Contributor']" \
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

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm emergency accounts still work before rollback | Admin workstation | Controlled sign-in test | Recovery access is available |
| 2 | Do not delete emergency accounts during normal rollback | Admin workstation | N/A | Tenant recovery remains protected |
| 3 | Remove accidental named admin role assignment | Admin workstation | Remove-MgDirectoryRoleMemberByRef | Unapproved role assignment removed |
| 4 | Remove accidental Azure RBAC assignment | Admin workstation | Remove-AzRoleAssignment | Unapproved Azure RBAC assignment removed |
| 5 | Remove accidental CA exclusion membership | Admin workstation | Remove-MgGroupMemberByRef | Unapproved exclusion removed |
| 6 | Restore CA policy assignment if changed incorrectly | Entra admin center | Conditional Access policy edit | Policy returns to approved state |
| 7 | Disable mistakenly created admin account only after access review | Admin workstation | Update-MgUser -AccountEnabled:$false | Account disabled but not deleted |
| 8 | Remove mistakenly created admin account only after approval | Admin workstation | Remove-MgUser | Account deleted only if safe |
| 9 | Clear Azure PowerShell context | Admin workstation | Clear-AzContext -Force | Cached context removed |
| 10 | Disconnect Graph session | Admin workstation | Disconnect-MgGraph | Graph session closed |
| 11 | Logout Azure CLI | Admin workstation | az logout | CLI session closed |
| 12 | Remove sensitive exports if needed | Admin workstation | Remove-Item ".\Privileged_Access_Baseline" -Recurse -Force | Local evidence removed |
| 13 | Document rollback actions | Admin workstation | Workbook notes | Audit trail preserved |

## Rollback Notes

Never remove the last working Global Administrator.

Never remove emergency access accounts until replacement emergency access accounts are created, tested, monitored, and approved.

Never remove Conditional Access exclusions from emergency accounts until emergency sign-in is tested through the replacement access path.

Never reduce privilege until you know:

- Who owns tenant recovery
- Which account can assign roles
- Which account can modify Conditional Access
- Which account can access Azure subscriptions
- Which account can restore admin access
- Which monitoring confirms emergency use

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Failure_Checks

| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| Emergency account cannot sign in | Password wrong, account disabled, CA block, risk policy block, MFA method unavailable | Admin workstation | Get-MgUser; sign-in logs | Use second emergency account, fix policy or credential storage |
| Emergency account blocked by Conditional Access | Account or exclusion group not excluded from blocking policy | Entra portal | Conditional Access policy review | Add approved exclusion and monitor |
| Emergency account requires unavailable MFA | Auth method depends on lost device, phone, or person | Entra portal | Authentication methods review | Configure survivable emergency auth method |
| Emergency account uses custom domain and DNS is broken | Domain/DNS/federation dependency | Admin workstation | UPN review | Use onmicrosoft.com emergency namespace |
| Emergency account synced from AD | Account depends on on-premises directory sync | Admin workstation | Get-MgUser OnPremisesSyncEnabled | Create cloud-only replacement account |
| Named admin account has no access | Role not assigned, PIM not activated, wrong account | Admin workstation | Get-MgContext; role inventory | Activate eligible role or assign correct role |
| Daily account has Global Administrator | Privilege assigned to normal user | Admin workstation | Directory role inventory | Move privilege to named admin account after approval |
| Too many Global Administrators | Standing privilege sprawl | Admin workstation | Critical role export | Convert to eligible roles and reduce standing assignments |
| Cannot assign Entra roles | Missing Privileged Role Administrator or Global Administrator | Admin workstation | Get-MgDirectoryRoleMember | Use approved role admin account |
| Cannot assign Azure RBAC | Missing Owner or User Access Administrator | Admin workstation | Get-AzRoleAssignment | Use approved Azure RBAC delegator |
| User is Global Administrator but cannot manage Azure subscription | Entra role does not grant Azure RBAC | Admin workstation | Get-AzSubscription | Assign Azure RBAC if approved |
| User is Azure Owner but cannot manage Entra roles | Azure RBAC does not grant Entra role management | Admin workstation | Get-MgDirectoryRole | Assign Entra role if approved |
| PIM role does not appear | Role is not eligible, wrong tenant, or PIM not enabled/licensed | Admin workstation | Entra PIM portal | Review PIM settings and role eligibility |
| PIM activation fails | Missing MFA, approval, justification, ticket, or policy requirement | Admin workstation | PIM activation page | Meet activation requirements |
| Emergency sign-in alert not triggered | Alert rule missing or wrong account filter | Security portal / Monitor | Sign-in log query | Create or fix alert |
| Sign-in logs unavailable | Licensing, retention, or permission issue | Admin workstation | Get-MgAuditLogSignIn | Use correct role/license or portal |
| Break glass account has license and mailbox | Account treated like normal user | Admin workstation | Get-MgUserLicenseDetail | Remove unnecessary license if approved |
| Password stored in insecure notes | Bad secret handling | Admin process | N/A | Move secret to approved secure storage and rotate |
| CA exclusion group has extra members | Exclusion group overused | Admin workstation | Get-MgGroupMember | Remove unapproved members |
| Privileged group has dynamic membership | Uncontrolled privilege assignment | Entra portal | Group membership rules | Use assigned membership for privileged groups |
| Automation identity has broad Graph permissions | App-only permission sprawl | Admin workstation | App registration permissions review | Reduce permissions and require admin consent review |
| Admin account disabled by lifecycle process | Admin account not excluded from HR/user lifecycle automation | Admin workstation | Audit logs | Restore account and update lifecycle controls |
| Vendor admin still active | Temporary access not expired | Entra portal | User/account review | Disable or remove vendor access |
| Access review removes needed admin | Review process lacks emergency/control validation | Entra portal | Access review history | Restore role and update review guidance |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Documentation_Output_TEMPLATE

## Privileged Access Baseline Summary

| Field | Value |
|---|---|
| Tenant display name |  |
| Tenant ID |  |
| Initial domain |  |
| Primary domain |  |
| Subscription name |  |
| Subscription ID |  |
| Admin account used for setup |  |
| PIM enabled | Yes / No |
| Access reviews enabled | Yes / No |
| Date collected |  |
| Evidence path |  |

## Emergency Access Account Baseline

| Field | Emergency Account 1 | Emergency Account 2 |
|---|---|---|
| UPN |  |  |
| Object ID |  |  |
| Cloud-only | Yes / No | Yes / No |
| Account enabled | Yes / No | Yes / No |
| Domain suffix |  |  |
| Role assignment |  |  |
| CA exclusion status |  |  |
| Auth method documented | Yes / No | Yes / No |
| Secret storage reference | Reference only | Reference only |
| Owner |  |  |
| Last test date |  |  |
| Monitoring alert configured | Yes / No | Yes / No |
| Last sign-in reviewed |  |  |

## Named Admin Account Baseline

| Human Owner | Daily Account | Admin Account | Account Enabled | Roles | PIM Eligible | Standing Privilege | Notes |
|---|---|---|---|---|---|---|---|
|  |  | adm- | Yes / No |  | Yes / No | Yes / No |  |

## Critical Entra Role Baseline

| Role | Members | Standing | Eligible | Risk | Notes |
|---|---|---|---|---|---|
| Global Administrator |  |  |  | Critical |  |
| Privileged Role Administrator |  |  |  | Critical |  |
| Conditional Access Administrator |  |  |  | High |  |
| Authentication Administrator |  |  |  | High |  |
| Privileged Authentication Administrator |  |  |  | Critical |  |
| Security Administrator |  |  |  | High |  |
| Compliance Administrator |  |  |  | High |  |
| Application Administrator |  |  |  | High |  |
| Cloud Application Administrator |  |  |  | High |  |
| User Administrator |  |  |  | Medium / High |  |
| Groups Administrator |  |  |  | Medium |  |

## Critical Azure RBAC Baseline

| Scope | Role | Principal | Principal Type | Standing / Eligible | Risk | Notes |
|---|---|---|---|---|---|---|
| Subscription | Owner |  |  |  | Critical |  |
| Subscription | User Access Administrator |  |  |  | Critical |  |
| Subscription | Contributor |  |  |  | High |  |
| Management group | Owner |  |  |  | Critical |  |
| Management group | User Access Administrator |  |  |  | Critical |  |

## Conditional Access Emergency Exclusion Baseline

| Policy | Emergency Account Excluded | Exclusion Method | Notes |
|---|---|---|---|
| Require MFA for admins | Yes / No | Direct / Group |  |
| Require compliant device | Yes / No | Direct / Group |  |
| Block risky sign-ins | Yes / No / Review | Direct / Group |  |
| Block by location | Yes / No / Review | Direct / Group |  |
| Require approved app | Yes / No | Direct / Group |  |
| Require terms of use | Yes / No | Direct / Group |  |

## Monitoring Baseline

| Event | Alert Configured | Destination | Test Result | Notes |
|---|---|---|---|---|
| Emergency account successful sign-in | Yes / No |  |  |  |
| Emergency account failed sign-in | Yes / No |  |  |  |
| Emergency account role change | Yes / No |  |  |  |
| CA exclusion group membership change | Yes / No |  |  |  |
| Global Administrator assignment | Yes / No |  |  |  |
| Privileged Role Administrator assignment | Yes / No |  |  |  |
| Azure Owner assignment | Yes / No |  |  |  |
| User Access Administrator assignment | Yes / No |  |  |  |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories | Supplies tenant, subscription, and directory baseline |
| 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline | Supplies portal and command tooling |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces | Supplies stable namespace decisions for admin and emergency accounts |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles | Supplies role plane comparison needed for privilege decisions |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline | Uses named admin account model and authentication controls |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking | Monitors privileged access and emergency account use |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Uses emergency access and role model for lockout troubleshooting |
| Azure Governance RBAC Workbook | Deep dive for Azure Owner, User Access Administrator, management group, and subscription role governance |
| Microsoft 365 Tenant Administration Workbook | Deep dive for M365 admin roles, licensing, service health, and admin center |
| Entra Enterprise Apps Workbook | Deep dive for app admin roles, app consent, service principals, and Graph permissions |
| Defender XDR Workbook | Deep dive for security role monitoring and incident response |
| Purview Workbook | Deep dive for compliance role groups, audit, eDiscovery, and DLP |
| Intune Workbook | Deep dive for endpoint admin roles, scope tags, and device management |
| Hybrid Identity Workbook | Deep dive for synced admin risks, federation failure, SSPR, MFA, and Conditional Access |

---

# 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline_Completion_Criteria

| Requirement | Complete |
|---|---|
| Tenant ID confirmed |  |
| Initial onmicrosoft.com domain confirmed |  |
| Admin account model documented |  |
| Daily-use accounts separated from admin accounts |  |
| Named admin account naming standard documented |  |
| Named admin accounts inventoried |  |
| Emergency access account 1 exists |  |
| Emergency access account 2 exists |  |
| Emergency accounts are cloud-only |  |
| Emergency accounts use stable namespace |  |
| Emergency accounts are enabled |  |
| Emergency accounts have approved recovery roles |  |
| Emergency account credentials stored outside workbook |  |
| Emergency auth methods documented |  |
| Emergency accounts excluded from lockout CA policies |  |
| Emergency account sign-in monitoring configured |  |
| Emergency account sign-in test completed |  |
| Global Administrators inventoried |  |
| Privileged Role Administrators inventoried |  |
| Conditional Access Administrators inventoried |  |
| Azure Owners inventoried |  |
| User Access Administrators inventoried |  |
| Daily accounts with privileged roles flagged |  |
| Standing privilege reviewed |  |
| PIM eligibility reviewed |  |
| PIM activation settings documented |  |
| Access review plan documented |  |
| Vendor privileged accounts reviewed |  |
| Automation privileged identities reviewed |  |
| Conditional Access exclusion group membership reviewed |  |
| Break glass test cadence documented |  |
| Evidence output stored securely |  |
| No secret values stored in workbook |  |

## Final Expected State

The admin can clearly state:

- These are the named admin accounts.
- These are the emergency access accounts.
- Emergency access accounts are cloud-only and recoverable.
- Emergency access accounts are excluded from policies that could block recovery.
- Emergency access accounts are monitored.
- Daily-use accounts do not hold unnecessary privileged roles.
- Global Administrator assignments are limited and documented.
- Privileged Role Administrator assignments are limited and documented.
- Azure Owner and User Access Administrator assignments are limited and documented.
- Named admins use PIM eligible roles where possible.
- Standing privilege is justified or flagged for reduction.
- Secret material is not stored in the workbook.
- The tenant has a documented privileged access recovery path.