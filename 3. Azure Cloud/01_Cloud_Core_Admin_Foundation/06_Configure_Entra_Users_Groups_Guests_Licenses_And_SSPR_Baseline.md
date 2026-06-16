# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Index

## Purpose

Configure a baseline for Microsoft Entra users, groups, guests, licenses, and self-service password reset.

This workbook covers:

- Entra user inventory
- Member user creation baseline
- Guest user invitation baseline
- Group inventory
- Security group baseline
- Microsoft 365 group baseline
- Dynamic group planning
- Administrative group planning
- License SKU inventory
- Direct license assignment planning
- Group-based licensing planning
- Usage location requirements
- Service plan enablement
- SSPR enablement planning
- Authentication method planning
- SSPR registration baseline
- SSPR test workflow
- Audit, validation, rollback, and failure checks

## Outcome

By the end of this workbook, the admin should be able to:

- Inventory Entra users, groups, guests, and license SKUs
- Create cloud-only test users
- Create baseline security groups
- Create baseline Microsoft 365 groups
- Invite guest users safely
- Assign licenses directly for lab use
- Plan group-based licensing for production use
- Validate usage location before license assignment
- Configure or validate SSPR scope
- Configure or validate authentication methods for SSPR
- Confirm users can register authentication methods
- Confirm users can reset their password through SSPR
- Document identity, group, guest, license, and SSPR baselines for later workbooks

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Source_Basis

| Source | Relevant Area | What It Supports |
|---|---|---|
| Microsoft Entra documentation | Users, groups, guests, authentication methods, SSPR, roles, audit logs, sign-in logs | Identity configuration baseline |
| Microsoft 365 documentation fork | Active users, licenses, groups, guest collaboration, admin center user management | M365 user and license workflow |
| Microsoft Graph documentation | Users, groups, invitations, subscribed SKUs, license assignment, authentication policy inventory | Graph PowerShell automation |
| Azure management documentation | Tenant context, role requirements, audit and monitoring handoff | Tenant context and governance |
| Existing Workbook 01 | Tenant, subscription, directory, and domain baseline | Supplies tenant ID, domains, and namespace context |
| Existing Workbook 02 | Admin portals, PowerShell, Azure CLI, and Graph baseline | Supplies tooling required for this workbook |
| Existing Workbook 03 | Custom domains and tenant namespaces | Supplies UPN domain and verified domain decisions |
| Existing Workbook 04 | Role plane comparison | Supplies User Administrator, Groups Administrator, License Administrator, and authentication role model |
| Existing Workbook 05 | Cloud admin, break glass, and privileged access baseline | Supplies safe admin account model |
| Existing Workbook structure | Source basis, mental model, planning, checklist, skeletons, verification, rollback, failure checks | Obsidian-ready workbook format |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Mental_Model

| Concept | Plain Meaning | Admin Boundary |
|---|---|---|
| Member user | Internal user object in the tenant | Identity plane |
| Guest user | External user invited into the tenant | External identity plane |
| Cloud-only user | User mastered directly in Entra ID | Cloud identity plane |
| Synced user | User mastered from on-premises AD through Entra Connect or cloud sync | Hybrid identity plane |
| User Principal Name | User sign-in name, usually user@domain.com | Identity namespace |
| Mail attribute | Email address value on user object | Mail identity attribute |
| Proxy address | Additional SMTP alias values | Exchange/mail attribute |
| Usage location | Country/region property required before license assignment | Licensing prerequisite |
| License SKU | Microsoft 365 product subscription | Licensing plane |
| Service plan | Individual service inside a license SKU | Workload entitlement |
| Direct licensing | License assigned directly to a user | Simple but harder to govern at scale |
| Group-based licensing | License assigned to a group and inherited by users | Scalable licensing model |
| Security group | Group used for access, licensing, policy, and assignment | Authorization plane |
| Microsoft 365 group | Collaboration group with mailbox, Teams/SharePoint backing depending use | Collaboration plane |
| Dynamic group | Group membership calculated from rules | Automated assignment plane |
| Assigned group | Group membership manually controlled | Explicit assignment plane |
| Administrative unit | Scoped directory boundary for delegated administration | Delegated identity admin scope |
| SSPR | Self-service password reset | Identity recovery plane |
| Combined registration | Unified registration for MFA and SSPR methods | Authentication method plane |
| Authentication method | Phone, Authenticator, FIDO2, TAP, email, security questions, etc. | Sign-in and recovery method |
| Authentication methods policy | Tenant policy controlling available authentication methods | Security policy plane |
| Registration campaign | Prompt users to register stronger authentication methods | Adoption and security plane |

## Core Rule

Users, groups, guests, licenses, and SSPR are identity-plane foundations.

Do not scale user creation, guest access, license assignment, or SSPR enforcement until:

- Tenant context is confirmed
- Verified domains are confirmed
- Admin roles are understood
- Break glass access is working
- Authentication methods are planned
- Licensing impact is understood
- Audit and rollback are documented

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Environment_Assumptions

| Item | Lab Example | Production Example |
|---|---|---|
| Tenant ID | 00000000-0000-0000-0000-000000000000 | Production tenant ID |
| Initial domain | contosolab.onmicrosoft.com | companyname.onmicrosoft.com |
| Primary verified domain | lab.contoso.com | contoso.com |
| Admin account | adm-cloud@contoso.com | named admin account |
| Break glass account | breakglass01@tenant.onmicrosoft.com | emergency access account |
| User admin role | User Administrator | Scoped User Administrator or Global Administrator only when needed |
| Group admin role | Groups Administrator | Groups Administrator or scoped owner model |
| License admin role | License Administrator | License Administrator / Billing admin split |
| Authentication role | Authentication Policy Administrator | Authentication Administrator / Authentication Policy Administrator |
| Test user prefix | labuser | standard naming convention |
| Test group prefix | SG-LAB | enterprise naming convention |
| Guest invite domain | partner.example.com | approved external partner domain |
| License model | Direct assignment for lab | Group-based licensing preferred |
| SSPR scope | Selected pilot group | Selected group, then all users after pilot |
| Output path | .\Entra_Users_Groups_Licenses_SSPR_Baseline | Secure evidence location |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Admin_Role_Requirements

| Task | Minimum Useful Role | Notes |
|---|---|---|
| View users | Directory Reader / Global Reader | Read-only inventory |
| Create users | User Administrator | Cloud-only users |
| Update user properties | User Administrator | Usage location, display name, account enabled |
| Delete users | User Administrator | Use soft-delete recovery awareness |
| Restore deleted users | User Administrator | Depends on deletion window |
| Invite guests | Guest Inviter / User Administrator | External collaboration settings may restrict |
| Create groups | Groups Administrator | Group owners can manage selected groups |
| Manage group membership | Groups Administrator / group owner | Scope depends on group ownership and settings |
| Manage dynamic groups | Groups Administrator | Requires suitable licensing |
| Assign licenses | License Administrator | Usage location required |
| View license SKUs | Global Reader / License Administrator | Billing role may be required for purchase details |
| Configure SSPR | Authentication Policy Administrator / Global Administrator | Portal feature permissions may vary |
| Manage authentication methods | Authentication Administrator / Authentication Policy Administrator | Security-sensitive |
| Review sign-in logs | Security Reader / Reports Reader / Global Reader | Licensing and retention may apply |
| Review audit logs | Reports Reader / Global Reader | Audit visibility depends on permissions |
| Manage Conditional Access for SSPR/MFA impact | Conditional Access Administrator | Separate from user/group/license roles |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_User_Model

| User Type | Source | Example UPN | Use | Notes |
|---|---|---|---|---|
| Cloud-only member | Entra ID | labuser01@contoso.com | Lab, cloud-native users, emergency admin support | Managed directly in cloud |
| Synced member | On-premises AD | user01@contoso.com | Hybrid workforce | Mastered on-premises |
| Guest user | External tenant/email | external_user#EXT#@tenant.onmicrosoft.com | B2B collaboration | External identity |
| Admin user | Entra ID | adm-user01@contoso.com | Privileged administration | Separate from daily account |
| Emergency user | Entra ID | breakglass01@tenant.onmicrosoft.com | Tenant recovery | Cloud-only and monitored |
| Shared user account | Avoid | shared@contoso.com | Avoid | Poor auditability |
| Service account | Avoid for cloud admin | svc-app@contoso.com | Legacy service needs | Prefer managed identity or app registration |
| Automation app identity | App registration / service principal | appId | Automation | Govern with least privilege |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Group_Model

| Group Type | Example | Main Use | Notes |
|---|---|---|---|
| Security group | SG-LIC-M365-E3 | Licensing, access, Conditional Access scoping | Preferred for assignments |
| Microsoft 365 group | M365-Project-CloudOps | Collaboration, mailbox, SharePoint, Teams backing | Not always best for security control |
| Mail-enabled security group | MESG-Alerts-Admins | Mail plus access/security in some services | Often Exchange-managed |
| Distribution group | DL-CloudOps | Email distribution | Not for Azure/Entra security access |
| Dynamic user group | SG-DYN-Dept-IT | Automated membership | Requires careful rule testing |
| Dynamic device group | SG-DYN-Devices-Windows | Intune targeting | Device attributes matter |
| Privileged access group | PAG-Entra-Role-Admins | PIM group-based role assignment | High risk, must be governed |
| Conditional Access group | CA-Require-MFA-Pilot | CA policy scoping | Avoid nested confusion |
| License assignment group | SG-LIC-M365-BusinessPremium | Group-based licensing | Track service plan changes |
| Guest access group | SG-GUEST-Partner-ProjectA | External access scoping | Review regularly |
| Break glass exclusion group | CA-EXCLUDE-Emergency-Access | Emergency CA exclusion | Extremely limited membership |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_License_Model

| License Concept | Operational Meaning | Admin Action |
|---|---|---|
| Subscribed SKU | Product purchased by tenant | Inventory with Graph or M365 admin center |
| SKU part number | Human-readable product code | Document exact SKU |
| SKU ID | GUID used in Graph license assignment | Required for automation |
| Consumed units | Number of assigned licenses | Track capacity |
| Enabled units | Available purchased units | Track availability |
| Service plan | Feature inside license | Disable only with planning |
| Usage location | Required user property for license assignment | Set before assigning license |
| Direct assignment | License applied directly to user | Useful for testing, harder at scale |
| Group-based assignment | License applied through group membership | Preferred for scalable governance |
| License conflict | User has incompatible assignment or disabled plan conflict | Troubleshoot assignment errors |
| Unlicensed user | User without product license | May still exist for admin/guest/service scenarios |
| Guest licensing | Depends on service and collaboration model | Review workload requirements |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_SSPR_Model

| SSPR Area | Recommendation | Notes |
|---|---|---|
| Scope | Start with selected pilot group | Avoid immediate tenant-wide enforcement |
| Pilot group | SG-SSPR-Pilot | Include IT and test users first |
| Production group | SG-SSPR-Enabled | Broader rollout group |
| Admin SSPR | Treat separately from standard users | Admin reset requirements are stricter |
| Registration | Use combined security info registration | Aligns MFA and SSPR method registration |
| Required methods | At least two where practical | Reduces single method failure |
| Authentication methods | Microsoft Authenticator, phone, alternate email, FIDO2/TAP depending policy | Align with security standard |
| Security questions | Avoid or restrict where possible | Weaker method |
| Notification on reset | Enable user notification | Helps detect abuse |
| Admin notification | Enable admin notification where available | Helps monitor risky resets |
| Helpdesk integration | Document support workflow | Users need fallback process |
| Conditional Access impact | Ensure registration page access works | CA can block registration |
| Break glass impact | Do not rely on SSPR for emergency accounts | Emergency accounts need separate recovery |
| Audit | Review password reset and registration events | Required for validation |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Planning_Table

| Item | Value To Record | Why It Matters |
|---|---|---|
| Tenant ID |  | Prevents wrong tenant changes |
| Initial domain |  | Fallback namespace |
| Primary verified domain |  | User UPN namespace |
| Admin account used |  | Audit trail |
| User naming standard |  | Keeps user inventory consistent |
| Admin account naming standard |  | Separates admin identities |
| Guest naming behavior |  | Identifies external users |
| Group naming standard |  | Keeps groups readable |
| License group naming standard |  | Supports group-based licensing |
| SSPR pilot group |  | Controls SSPR rollout |
| MFA/SSPR method policy |  | Determines allowed recovery methods |
| Usage location default |  | Required for licensing |
| Target license SKU |  | Defines service access |
| Disabled service plans |  | Prevents accidental service enablement |
| Guest invitation policy |  | Controls external collaboration |
| Guest invite approver |  | External access governance |
| Access review cadence |  | Guest/group cleanup |
| Audit retention |  | Investigation readiness |
| Output path |  | Evidence storage |
| Rollback owner |  | Recovery accountability |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Baseline_Groups

| Group Name | Type | Membership | Purpose |
|---|---|---|---|
| SG-SSPR-Pilot | Security | Assigned | Pilot SSPR enablement |
| SG-SSPR-Enabled | Security | Assigned or dynamic after pilot | Production SSPR scope |
| SG-LIC-M365-Baseline | Security | Assigned | Group-based licensing baseline |
| SG-LIC-M365-AdminTools | Security | Assigned | Admin tool licensing if needed |
| SG-CA-MFA-Pilot | Security | Assigned | Conditional Access MFA pilot |
| SG-CA-Exclude-Emergency-Access | Security | Assigned | Emergency access CA exclusion |
| SG-GUEST-ProjectA | Security | Assigned | Guest access scoping |
| SG-APP-CloudOps-Readers | Security | Assigned | Application/resource read access |
| SG-APP-CloudOps-Contributors | Security | Assigned | Application/resource contributor access |
| M365-CloudOps-Collab | Microsoft 365 group | Assigned | Collaboration workspace |
| M365-SecurityOps-Collab | Microsoft 365 group | Assigned | Security team collaboration |
| PAG-Entra-Role-Admins | Security / privileged access group | Assigned/PIM governed | Optional privileged role group |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm workbook 01 tenant and domain baseline | Admin workstation | Review tenant ID and verified domains | Correct tenant and UPN namespace are known |
| 2 | Confirm workbook 02 tooling baseline | Admin workstation | Get-MgContext; Get-AzContext | Graph and Azure tooling are ready |
| 3 | Confirm workbook 04 role requirements | Admin workstation | Review role map | User, group, license, and auth admin roles understood |
| 4 | Confirm workbook 05 admin and break glass model | Admin workstation | Review break glass baseline | Emergency access is protected before changes |
| 5 | Connect Microsoft Graph | Admin workstation | Connect-MgGraph -TenantId "<tenant-id>" -Scopes "Directory.ReadWrite.All","User.ReadWrite.All","Group.ReadWrite.All","Organization.Read.All" | Graph session connected |
| 6 | Confirm organization | Admin workstation | Get-MgOrganization | Correct tenant shown |
| 7 | Inventory existing users | Admin workstation | Get-MgUser -All | Existing users exported |
| 8 | Inventory guest users | Admin workstation | Get-MgUser -Filter "userType eq 'Guest'" | Existing guests exported |
| 9 | Inventory existing groups | Admin workstation | Get-MgGroup -All | Existing groups exported |
| 10 | Inventory license SKUs | Admin workstation | Get-MgSubscribedSku -All | License capacity documented |
| 11 | Confirm verified UPN domains | Admin workstation | Get-MgDomain -All | Verified domain list documented |
| 12 | Create baseline test users | Admin workstation | New-MgUser | Cloud-only test users created |
| 13 | Set usage location for test users | Admin workstation | Update-MgUser -UsageLocation "US" | Users become license-ready |
| 14 | Create baseline security groups | Admin workstation | New-MgGroup -SecurityEnabled:$true | Security groups created |
| 15 | Create baseline Microsoft 365 group if needed | Admin workstation | New-MgGroup -GroupTypes "Unified" | M365 group created |
| 16 | Add users to baseline groups | Admin workstation | New-MgGroupMemberByRef | Membership assigned |
| 17 | Invite guest test user if approved | Admin workstation | New-MgInvitation | Guest invitation sent or staged |
| 18 | Add guest to scoped guest group | Admin workstation | New-MgGroupMemberByRef | Guest access scoped through group |
| 19 | Assign test license directly only for lab | Admin workstation | Set-MgUserLicense | Test user receives license |
| 20 | Plan group-based licensing | M365 admin center / Entra admin center | Groups > Licenses | License group design documented |
| 21 | Assign license to group if approved | Entra admin center | Group > Licenses > Assignments | Group-based license assignment configured |
| 22 | Validate license assignment result | Admin workstation | Get-MgUserLicenseDetail | User license and service plans shown |
| 23 | Create SSPR pilot group | Admin workstation | New-MgGroup | Pilot group exists |
| 24 | Add test users to SSPR pilot group | Admin workstation | New-MgGroupMemberByRef | Pilot users scoped for SSPR |
| 25 | Configure SSPR scope | Entra admin center | Protection > Password reset > Properties | SSPR enabled for selected group |
| 26 | Configure SSPR authentication methods | Entra admin center | Protection > Password reset > Authentication methods | Required methods documented |
| 27 | Configure SSPR registration settings | Entra admin center | Protection > Password reset > Registration | Registration policy documented |
| 28 | Configure SSPR notifications | Entra admin center | Protection > Password reset > Notifications | User/admin notifications documented |
| 29 | Validate security info registration | Test user browser | https://aka.ms/mysecurityinfo | User can register method |
| 30 | Validate SSPR reset test | Test user browser | https://passwordreset.microsoftonline.com | Test user can reset password |
| 31 | Review SSPR audit/sign-in evidence | Admin workstation | Audit/sign-in log review | Registration/reset events visible |
| 32 | Export final baseline | Admin workstation | Run export skeleton | Evidence saved |
| 33 | Save workbook output | Admin workstation | Commit note to workbook repo or Obsidian vault | Baseline ready for later labs |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Precheck_Skeleton

```powershell
# ============================================================
# Entra Users, Groups, Guests, Licenses, and SSPR Precheck
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Entra_Users_Groups_Licenses_SSPR_Baseline"
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

# Confirm Graph context
$MgContext = Get-MgContext

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgContext.csv")

# Confirm organization
$Organization = Get-MgOrganization

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Organization.csv")

# Confirm domains
$Domains = Get-MgDomain -All

$Domains |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsDefault -Descending, Id |
    Format-Table -AutoSize

$Domains |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsDefault -Descending, Id |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Domains.csv")

# Confirm subscribed SKUs
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
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "SubscribedSkus.csv")

Write-Host "Precheck complete. Output path: $OutputPath"
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Inventory_Skeleton

```powershell
# ============================================================
# Entra Identity Inventory Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Entra_Users_Groups_Licenses_SSPR_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "User.Read.All", `
    "Group.Read.All", `
    "AuditLog.Read.All"

# -----------------------------
# Users
# -----------------------------
$Users = Get-MgUser -All -Property `
    Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled,CreatedDateTime,UsageLocation,OnPremisesSyncEnabled,AssignedLicenses

$Users |
    Select-Object DisplayName, UserPrincipalName, Mail, UserType, AccountEnabled, UsageLocation, OnPremisesSyncEnabled, CreatedDateTime, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Users.csv")

# -----------------------------
# Guest users
# -----------------------------
$Guests = $Users | Where-Object {$_.UserType -eq "Guest"}

$Guests |
    Select-Object DisplayName, UserPrincipalName, Mail, AccountEnabled, CreatedDateTime, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Guests.csv")

# -----------------------------
# Cloud-only members
# -----------------------------
$CloudOnlyMembers = $Users |
    Where-Object {$_.UserType -eq "Member" -and $_.OnPremisesSyncEnabled -ne $true}

$CloudOnlyMembers |
    Select-Object DisplayName, UserPrincipalName, Mail, AccountEnabled, UsageLocation, CreatedDateTime, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "CloudOnlyMembers.csv")

# -----------------------------
# Synced members
# -----------------------------
$SyncedMembers = $Users |
    Where-Object {$_.UserType -eq "Member" -and $_.OnPremisesSyncEnabled -eq $true}

$SyncedMembers |
    Select-Object DisplayName, UserPrincipalName, Mail, AccountEnabled, UsageLocation, CreatedDateTime, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "SyncedMembers.csv")

# -----------------------------
# Groups
# -----------------------------
$Groups = Get-MgGroup -All -Property `
    Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes,MembershipRule,MembershipRuleProcessingState,CreatedDateTime

$Groups |
    Select-Object DisplayName, Mail, MailEnabled, SecurityEnabled, GroupTypes, MembershipRule, MembershipRuleProcessingState, CreatedDateTime, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Groups.csv")

# -----------------------------
# Security groups
# -----------------------------
$SecurityGroups = $Groups |
    Where-Object {$_.SecurityEnabled -eq $true -and ($_.GroupTypes -notcontains "Unified")}

$SecurityGroups |
    Select-Object DisplayName, Mail, SecurityEnabled, GroupTypes, MembershipRule, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "SecurityGroups.csv")

# -----------------------------
# Microsoft 365 groups
# -----------------------------
$M365Groups = $Groups |
    Where-Object {$_.GroupTypes -contains "Unified"}

$M365Groups |
    Select-Object DisplayName, Mail, MailEnabled, SecurityEnabled, GroupTypes, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "M365Groups.csv")

# -----------------------------
# Dynamic groups
# -----------------------------
$DynamicGroups = $Groups |
    Where-Object {$_.GroupTypes -contains "DynamicMembership"}

$DynamicGroups |
    Select-Object DisplayName, MailEnabled, SecurityEnabled, GroupTypes, MembershipRule, MembershipRuleProcessingState, Id |
    Sort-Object DisplayName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "DynamicGroups.csv")

# -----------------------------
# Subscribed SKUs
# -----------------------------
$SubscribedSkus = Get-MgSubscribedSku -All

$SubscribedSkus |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="SuspendedUnits";Expression={$_.PrepaidUnits.Suspended}},
        @{Name="WarningUnits";Expression={$_.PrepaidUnits.Warning}} |
    Sort-Object SkuPartNumber |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "SubscribedSkus.csv")

# -----------------------------
# Summary
# -----------------------------
$Summary = [PSCustomObject]@{
    TenantId             = $TenantId
    TotalUsers           = $Users.Count
    MemberUsers          = ($Users | Where-Object {$_.UserType -eq "Member"}).Count
    GuestUsers           = $Guests.Count
    CloudOnlyMembers     = $CloudOnlyMembers.Count
    SyncedMembers        = $SyncedMembers.Count
    TotalGroups          = $Groups.Count
    SecurityGroups       = $SecurityGroups.Count
    M365Groups           = $M365Groups.Count
    DynamicGroups        = $DynamicGroups.Count
    SubscribedSkuCount   = $SubscribedSkus.Count
    DateCollected        = Get-Date
}

$Summary |
    Format-List

$Summary |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "IdentityBaselineSummary.csv")

Write-Host "Inventory complete. Output path: $OutputPath"
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Create_Cloud_Users_Skeleton

```powershell
# ============================================================
# Create Cloud-Only Test Users Skeleton
# Do not store real passwords in this workbook.
# ============================================================

$TenantId = "<tenant-id>"
$DomainName = "contoso.com"
$UsageLocation = "US"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "User.ReadWrite.All", `
    "Directory.ReadWrite.All"

# Replace temporary password through an approved secure process.
$TemporaryPassword = "<temporary-password-from-secure-process>"

$UsersToCreate = @(
    @{
        DisplayName = "Lab User 01"
        MailNickname = "labuser01"
        UserPrincipalName = "labuser01@$DomainName"
    },
    @{
        DisplayName = "Lab User 02"
        MailNickname = "labuser02"
        UserPrincipalName = "labuser02@$DomainName"
    },
    @{
        DisplayName = "Lab User 03"
        MailNickname = "labuser03"
        UserPrincipalName = "labuser03@$DomainName"
    }
)

foreach ($User in $UsersToCreate) {
    $PasswordProfile = @{
        forceChangePasswordNextSignIn = $true
        password = $TemporaryPassword
    }

    New-MgUser `
        -AccountEnabled:$true `
        -DisplayName $User.DisplayName `
        -MailNickname $User.MailNickname `
        -UserPrincipalName $User.UserPrincipalName `
        -PasswordProfile $PasswordProfile

    Update-MgUser `
        -UserId $User.UserPrincipalName `
        -UsageLocation $UsageLocation
}

# Verify
foreach ($User in $UsersToCreate) {
    Get-MgUser -UserId $User.UserPrincipalName -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,UsageLocation |
        Select-Object Id, DisplayName, UserPrincipalName, UserType, AccountEnabled, UsageLocation |
        Format-List
}
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Create_Groups_Skeleton

```powershell
# ============================================================
# Create Baseline Groups Skeleton
# ============================================================

$TenantId = "<tenant-id>"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Group.ReadWrite.All", `
    "Directory.ReadWrite.All"

$SecurityGroups = @(
    @{
        DisplayName = "SG-SSPR-Pilot"
        MailNickname = "sg-sspr-pilot"
        Description = "Pilot group for self-service password reset enablement"
    },
    @{
        DisplayName = "SG-SSPR-Enabled"
        MailNickname = "sg-sspr-enabled"
        Description = "Production group for self-service password reset enablement"
    },
    @{
        DisplayName = "SG-LIC-M365-Baseline"
        MailNickname = "sg-lic-m365-baseline"
        Description = "Baseline group for Microsoft 365 license assignment"
    },
    @{
        DisplayName = "SG-CA-MFA-Pilot"
        MailNickname = "sg-ca-mfa-pilot"
        Description = "Pilot group for Conditional Access MFA policy"
    },
    @{
        DisplayName = "SG-GUEST-ProjectA"
        MailNickname = "sg-guest-projecta"
        Description = "Scoped guest access group for ProjectA"
    }
)

foreach ($Group in $SecurityGroups) {
    New-MgGroup `
        -DisplayName $Group.DisplayName `
        -Description $Group.Description `
        -MailEnabled:$false `
        -MailNickname $Group.MailNickname `
        -SecurityEnabled:$true
}

# Optional Microsoft 365 collaboration group
New-MgGroup `
    -DisplayName "M365-CloudOps-Collab" `
    -Description "Microsoft 365 collaboration group for CloudOps" `
    -MailEnabled:$true `
    -MailNickname "m365-cloudops-collab" `
    -SecurityEnabled:$false `
    -GroupTypes @("Unified")

# Verify
Get-MgGroup -All -Property Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes |
    Where-Object {
        $_.DisplayName -like "SG-SSPR-*" -or
        $_.DisplayName -like "SG-LIC-*" -or
        $_.DisplayName -like "SG-CA-*" -or
        $_.DisplayName -like "SG-GUEST-*" -or
        $_.DisplayName -like "M365-CloudOps-*"
    } |
    Select-Object DisplayName, Mail, MailEnabled, SecurityEnabled, GroupTypes, Id |
    Sort-Object DisplayName |
    Format-Table -AutoSize
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Add_Group_Members_Skeleton

```powershell
# ============================================================
# Add Users to Groups Skeleton
# ============================================================

$TenantId = "<tenant-id>"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Group.ReadWrite.All", `
    "User.Read.All", `
    "Directory.ReadWrite.All"

$MembershipPlan = @(
    @{
        GroupName = "SG-SSPR-Pilot"
        Members = @(
            "labuser01@contoso.com",
            "labuser02@contoso.com"
        )
    },
    @{
        GroupName = "SG-LIC-M365-Baseline"
        Members = @(
            "labuser01@contoso.com",
            "labuser02@contoso.com"
        )
    },
    @{
        GroupName = "SG-CA-MFA-Pilot"
        Members = @(
            "labuser01@contoso.com"
        )
    }
)

foreach ($Plan in $MembershipPlan) {
    $Group = Get-MgGroup -Filter "displayName eq '$($Plan.GroupName)'"

    foreach ($MemberUpn in $Plan.Members) {
        $User = Get-MgUser -UserId $MemberUpn

        New-MgGroupMemberByRef `
            -GroupId $Group.Id `
            -BodyParameter @{
                "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
            }
    }
}

# Verify group members
foreach ($Plan in $MembershipPlan) {
    $Group = Get-MgGroup -Filter "displayName eq '$($Plan.GroupName)'"

    Write-Host "Members of $($Plan.GroupName)"
    Get-MgGroupMember -GroupId $Group.Id -All |
        Select-Object Id, AdditionalProperties |
        Format-List
}
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Dynamic_Group_Planning_Skeleton

```text
# ============================================================
# Dynamic Group Planning Skeleton
# ============================================================

Portal:
https://entra.microsoft.com

Navigation:
Identity > Groups > All groups > New group

Dynamic user group example:
Group type:
Security

Group name:
SG-DYN-Dept-IT

Membership type:
Dynamic User

Rule example:
(user.department -eq "IT")

Dynamic device group example:
Group type:
Security

Group name:
SG-DYN-Devices-Windows

Membership type:
Dynamic Device

Rule example:
(device.deviceOSType -contains "Windows")

Planning notes:
1. Validate source attributes before relying on dynamic groups.
2. Do not use dynamic groups for emergency access.
3. Do not use dynamic groups for highly privileged access without strong governance.
4. Test dynamic membership with pilot users/devices.
5. Document rule logic and expected members.
6. Monitor membership processing state.
7. Use assigned groups when exact control is required.

Documentation table:
| Group | Membership Rule | Expected Members | Actual Members | Used For | Risk |
|---|---|---|---|---|---|
| SG-DYN-Dept-IT | user.department -eq "IT" |  |  | Licensing / CA / App access | Medium |
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Guest_Invitation_Skeleton

```powershell
# ============================================================
# Guest Invitation Skeleton
# Use only for approved external collaboration.
# ============================================================

$TenantId = "<tenant-id>"
$GuestEmail = "external.user@partner.example.com"
$RedirectUrl = "https://myapplications.microsoft.com"
$GuestGroupName = "SG-GUEST-ProjectA"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "User.Invite.All", `
    "User.ReadWrite.All", `
    "Group.ReadWrite.All", `
    "Directory.ReadWrite.All"

# Invite guest
$Invitation = New-MgInvitation `
    -InvitedUserEmailAddress $GuestEmail `
    -InviteRedirectUrl $RedirectUrl `
    -SendInvitationMessage:$true

$Invitation |
    Format-List

# Add guest to scoped group after invitation object exists
$GuestUserId = $Invitation.InvitedUser.Id
$GuestGroup = Get-MgGroup -Filter "displayName eq '$GuestGroupName'"

New-MgGroupMemberByRef `
    -GroupId $GuestGroup.Id `
    -BodyParameter @{
        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$GuestUserId"
    }

# Verify guest
Get-MgUser -UserId $GuestUserId -Property Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled |
    Select-Object Id, DisplayName, UserPrincipalName, Mail, UserType, AccountEnabled |
    Format-List

# Verify group membership
Get-MgGroupMember -GroupId $GuestGroup.Id -All |
    Select-Object Id, AdditionalProperties |
    Format-List
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Guest_Access_Portal_Skeleton

```text
# ============================================================
# Guest Access Portal Review Skeleton
# ============================================================

Portal:
https://entra.microsoft.com

Navigation:
Identity > External Identities > External collaboration settings

Review:
1. Guest invite settings
2. Who can invite guests
3. Guest user access restrictions
4. Collaboration restrictions
5. Allowed domains
6. Blocked domains
7. Guest self-service sign-up if used
8. Cross-tenant access settings
9. Default user permissions
10. Guest lifecycle and access review process

Recommended documentation:
| Setting | Current Value | Desired Value | Notes |
|---|---|---|---|
| Guest invite permissions |  |  |  |
| Guest user access |  |  |  |
| Allow/block domains |  |  |  |
| Cross-tenant access |  |  |  |
| Guest access reviews |  |  |  |

Expected result:
Guest access is scoped, intentional, and reviewable.
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_License_Inventory_Skeleton

```powershell
# ============================================================
# License Inventory Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$OutputRoot = ".\Entra_Users_Groups_Licenses_SSPR_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "User.Read.All"

# Subscribed SKUs
$Skus = Get-MgSubscribedSku -All

$Skus |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="SuspendedUnits";Expression={$_.PrepaidUnits.Suspended}},
        @{Name="WarningUnits";Expression={$_.PrepaidUnits.Warning}},
        @{Name="ServicePlanCount";Expression={$_.ServicePlans.Count}} |
    Sort-Object SkuPartNumber |
    Format-Table -AutoSize

$Skus |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="SuspendedUnits";Expression={$_.PrepaidUnits.Suspended}},
        @{Name="WarningUnits";Expression={$_.PrepaidUnits.Warning}},
        @{Name="ServicePlanCount";Expression={$_.ServicePlans.Count}} |
    Sort-Object SkuPartNumber |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "License_Skus.csv")

# Service plans per SKU
$ServicePlans = foreach ($Sku in $Skus) {
    foreach ($Plan in $Sku.ServicePlans) {
        [PSCustomObject]@{
            SkuPartNumber       = $Sku.SkuPartNumber
            SkuId               = $Sku.SkuId
            ServicePlanName     = $Plan.ServicePlanName
            ServicePlanId       = $Plan.ServicePlanId
            ProvisioningStatus  = $Plan.ProvisioningStatus
            AppliesTo           = $Plan.AppliesTo
        }
    }
}

$ServicePlans |
    Sort-Object SkuPartNumber, ServicePlanName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "License_ServicePlans.csv")

# Users missing usage location but likely license candidates
$Users = Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,UsageLocation,AssignedLicenses

$UsersMissingUsageLocation = $Users |
    Where-Object {$_.UserType -eq "Member" -and $_.AccountEnabled -eq $true -and [string]::IsNullOrWhiteSpace($_.UsageLocation)}

$UsersMissingUsageLocation |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, UsageLocation, Id |
    Sort-Object UserPrincipalName |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Users_MissingUsageLocation.csv")

Write-Host "License inventory complete. Output path: $OutputPath"
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Direct_License_Assignment_Skeleton

```powershell
# ============================================================
# Direct License Assignment Skeleton
# Use direct assignment for lab/testing only unless approved.
# Group-based licensing is preferred for production scale.
# ============================================================

$TenantId = "<tenant-id>"
$UserUpn = "labuser01@contoso.com"
$UsageLocation = "US"
$SkuPartNumber = "<sku-part-number>"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "User.ReadWrite.All", `
    "Organization.Read.All", `
    "Directory.ReadWrite.All"

# Confirm user
Get-MgUser -UserId $UserUpn -Property Id,DisplayName,UserPrincipalName,UsageLocation,AssignedLicenses |
    Select-Object Id, DisplayName, UserPrincipalName, UsageLocation |
    Format-List

# Set usage location
Update-MgUser -UserId $UserUpn -UsageLocation $UsageLocation

# Get SKU
$Sku = Get-MgSubscribedSku -All | Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

$Sku |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}} |
    Format-List

# Assign license with all service plans enabled
$AddLicenses = @(
    @{
        SkuId = $Sku.SkuId
        DisabledPlans = @()
    }
)

Set-MgUserLicense `
    -UserId $UserUpn `
    -AddLicenses $AddLicenses `
    -RemoveLicenses @()

# Verify license details
Get-MgUserLicenseDetail -UserId $UserUpn |
    Select-Object SkuPartNumber, SkuId |
    Format-Table -AutoSize
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Remove_Direct_License_Skeleton

```powershell
# ============================================================
# Remove Direct License Assignment Skeleton
# Use only after confirming user does not need service access.
# ============================================================

$TenantId = "<tenant-id>"
$UserUpn = "labuser01@contoso.com"
$SkuPartNumber = "<sku-part-number>"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "User.ReadWrite.All", `
    "Organization.Read.All"

$Sku = Get-MgSubscribedSku -All | Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

# Verify current license
Get-MgUserLicenseDetail -UserId $UserUpn |
    Select-Object SkuPartNumber, SkuId |
    Format-Table -AutoSize

# Remove license
Set-MgUserLicense `
    -UserId $UserUpn `
    -AddLicenses @() `
    -RemoveLicenses @($Sku.SkuId)

# Verify removal
Get-MgUserLicenseDetail -UserId $UserUpn |
    Select-Object SkuPartNumber, SkuId |
    Format-Table -AutoSize
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Group_Based_Licensing_Portal_Skeleton

```text
# ============================================================
# Group-Based Licensing Portal Skeleton
# ============================================================

Portal:
https://entra.microsoft.com

Navigation:
Identity > Billing > Licenses
or
Identity > Groups > All groups > select group > Licenses

Recommended group:
SG-LIC-M365-Baseline

Steps:
1. Confirm the target license SKU and available units.
2. Confirm users in the license group have Usage location set.
3. Open the target license assignment group.
4. Select Licenses.
5. Select Assignments.
6. Select the target product SKU.
7. Review service plans.
8. Disable only service plans intentionally excluded by design.
9. Assign the license.
10. Wait for processing.
11. Review assignment errors.
12. Validate user license details.
13. Export group membership and license result.

Expected result:
Users inherit licenses through the group and show no assignment errors.

Notes:
- Usage location is required before license assignment.
- Group-based licensing is preferred for production scale.
- Do not mix direct and group licensing without documentation.
- Review disabled service plans before assigning the group broadly.
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_SSPR_Portal_Skeleton

```text
# ============================================================
# SSPR Portal Configuration Skeleton
# ============================================================

Portal:
https://entra.microsoft.com

Navigation:
Protection > Password reset

Properties:
1. Open Protection > Password reset.
2. Select Properties.
3. Choose one:
   - None
   - Selected
   - All
4. For pilot, choose Selected.
5. Select group:
   SG-SSPR-Pilot
6. Save.

Authentication methods:
1. Open Authentication methods.
2. Set number of methods required to reset.
3. Choose approved methods.
4. Recommended for pilot:
   - Microsoft Authenticator app notification or code if enabled in auth methods policy
   - Mobile phone if approved
   - Alternate email if approved
5. Avoid security questions for high-security environments.
6. Save.

Registration:
1. Open Registration.
2. Require users to register when signing in:
   Yes for rollout
3. Number of days before users are asked to reconfirm authentication information:
   Document value
4. Save.

Notifications:
1. Notify users on password resets:
   Yes
2. Notify all admins when other admins reset their passwords:
   Yes where available
3. Save.

Customization:
1. Add helpdesk email or URL if approved.
2. Save.

Expected result:
SSPR is enabled for the pilot group and users can register recovery methods.
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Authentication_Methods_Portal_Skeleton

```text
# ============================================================
# Authentication Methods Policy Portal Skeleton
# ============================================================

Portal:
https://entra.microsoft.com

Navigation:
Protection > Authentication methods

Review:
1. Policies
2. Microsoft Authenticator
3. FIDO2 security key
4. Temporary Access Pass
5. SMS
6. Voice call
7. Email OTP
8. Certificate-based authentication if used
9. Registration campaign
10. System-preferred MFA if enabled
11. Migration from legacy MFA/SSPR policies if applicable

Recommended documentation:
| Method | Enabled | Target | Used For | Notes |
|---|---|---|---|---|
| Microsoft Authenticator | Yes / No |  | MFA / SSPR |  |
| FIDO2 security key | Yes / No |  | Phishing-resistant MFA |  |
| Temporary Access Pass | Yes / No |  | Bootstrap / recovery |  |
| SMS | Yes / No |  | MFA / SSPR |  |
| Voice call | Yes / No |  | MFA / SSPR |  |
| Email OTP | Yes / No |  | Guest / SSPR depending config |  |
| Security questions | Yes / No |  | SSPR only |  |

Expected result:
Authentication methods are documented before SSPR enforcement.
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_SSPR_Test_Skeleton

```text
# ============================================================
# SSPR Test Skeleton
# ============================================================

Test user:
labuser01@contoso.com

Pre-test:
1. Confirm user account is enabled.
2. Confirm user is in SG-SSPR-Pilot.
3. Confirm SSPR is enabled for selected group.
4. Confirm user has registered required authentication methods.
5. Confirm Conditional Access does not block security info registration.
6. Confirm helpdesk or admin owner is available.

Security info registration:
1. Open private browser session.
2. Go to https://aka.ms/mysecurityinfo.
3. Sign in as test user.
4. Register required authentication methods.
5. Confirm methods show as registered.
6. Sign out.

Password reset:
1. Open private browser session.
2. Go to https://passwordreset.microsoftonline.com.
3. Enter test user UPN.
4. Complete verification steps.
5. Set a new password.
6. Sign in with new password.
7. Confirm reset event appears in audit/sign-in logs.
8. Sign out.

Expected result:
User can register methods and complete password reset without admin intervention.

Do not test with emergency access accounts.
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Audit_And_Sign_In_Review_Skeleton

```powershell
# ============================================================
# Audit and Sign-In Review Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$TestUsers = @(
    "labuser01@contoso.com",
    "labuser02@contoso.com"
)

$OutputRoot = ".\Entra_Users_Groups_Licenses_SSPR_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

Connect-MgGraph -TenantId $TenantId -Scopes `
    "AuditLog.Read.All", `
    "Directory.Read.All", `
    "User.Read.All"

# Recent directory audit events
try {
    $DirectoryAudit = Get-MgAuditLogDirectoryAudit -Top 100

    $DirectoryAudit |
        Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, Result |
        Format-Table -AutoSize

    $DirectoryAudit |
        Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, Result |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "DirectoryAudit_Recent.csv")
}
catch {
    Write-Warning "Could not retrieve directory audit events."
    Write-Warning $_.Exception.Message
}

# Sign-ins for test users
foreach ($UserUpn in $TestUsers) {
    try {
        $Filter = "userPrincipalName eq '$UserUpn'"

        $SignIns = Get-MgAuditLogSignIn -Filter $Filter -Top 25

        $SignIns |
            Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, IpAddress, ClientAppUsed, ConditionalAccessStatus, Status |
            Format-Table -AutoSize

        $SignIns |
            Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, IpAddress, ClientAppUsed, ConditionalAccessStatus, Status |
            Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "$($UserUpn.Replace('@','_').Replace('.','_'))_SignIns.csv")
    }
    catch {
        Write-Warning "Could not retrieve sign-ins for $UserUpn"
        Write-Warning $_.Exception.Message
    }
}
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Azure_CLI_Inventory_Skeleton

```bash
# ============================================================
# Azure CLI and Graph REST Identity Inventory Skeleton
# ============================================================

TENANT_ID="<tenant-id>"

az login --tenant "$TENANT_ID"

mkdir -p Entra_Users_Groups_Licenses_SSPR_Baseline

# Confirm account
az account show --output table

# Organization
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json > Entra_Users_Groups_Licenses_SSPR_Baseline/organization.json

# Domains
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains" \
  --output json > Entra_Users_Groups_Licenses_SSPR_Baseline/domains.json

# Users
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/users?\$select=id,displayName,userPrincipalName,mail,userType,accountEnabled,usageLocation,onPremisesSyncEnabled" \
  --output json > Entra_Users_Groups_Licenses_SSPR_Baseline/users_sample.json

# Groups
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/groups?\$select=id,displayName,mail,mailEnabled,securityEnabled,groupTypes" \
  --output json > Entra_Users_Groups_Licenses_SSPR_Baseline/groups_sample.json

# License SKUs
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --output json > Entra_Users_Groups_Licenses_SSPR_Baseline/subscribedSkus.json

# Guest sample
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/users?\$filter=userType eq 'Guest'&\$select=id,displayName,userPrincipalName,mail,userType,accountEnabled" \
  --output json > Entra_Users_Groups_Licenses_SSPR_Baseline/guests_sample.json
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Verification_Commands

```powershell
# ============================================================
# Verification Commands
# ============================================================

$TenantId = "<tenant-id>"

# Confirm Graph context
Get-MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

# Confirm organization
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

# Confirm domains
Get-MgDomain -All |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsDefault -Descending, Id |
    Format-Table -AutoSize

# Confirm users
Get-MgUser -Top 10 -Property DisplayName,UserPrincipalName,UserType,AccountEnabled,UsageLocation,OnPremisesSyncEnabled |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, UsageLocation, OnPremisesSyncEnabled |
    Format-Table -AutoSize

# Confirm guests
Get-MgUser -Filter "userType eq 'Guest'" -All -Property DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled |
    Select-Object DisplayName, UserPrincipalName, Mail, UserType, AccountEnabled |
    Sort-Object UserPrincipalName |
    Format-Table -AutoSize

# Confirm groups
Get-MgGroup -All -Property DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes |
    Select-Object DisplayName, Mail, MailEnabled, SecurityEnabled, GroupTypes |
    Sort-Object DisplayName |
    Format-Table -AutoSize

# Confirm baseline groups
$BaselineGroupNames = @(
    "SG-SSPR-Pilot",
    "SG-SSPR-Enabled",
    "SG-LIC-M365-Baseline",
    "SG-CA-MFA-Pilot",
    "SG-GUEST-ProjectA"
)

foreach ($GroupName in $BaselineGroupNames) {
    Get-MgGroup -Filter "displayName eq '$GroupName'" -Property Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes |
        Select-Object Id, DisplayName, MailEnabled, SecurityEnabled, GroupTypes |
        Format-List
}

# Confirm license SKUs
Get-MgSubscribedSku -All |
    Select-Object SkuPartNumber, SkuId, ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="AvailableUnits";Expression={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}} |
    Sort-Object SkuPartNumber |
    Format-Table -AutoSize

# Confirm user license details
Get-MgUserLicenseDetail -UserId "labuser01@contoso.com" |
    Select-Object SkuPartNumber, SkuId |
    Format-Table -AutoSize

# Confirm SSPR pilot group membership
$SsprGroup = Get-MgGroup -Filter "displayName eq 'SG-SSPR-Pilot'"
Get-MgGroupMember -GroupId $SsprGroup.Id -All |
    Select-Object Id, AdditionalProperties |
    Format-List

# Recent audit events
Get-MgAuditLogDirectoryAudit -Top 25 |
    Select-Object ActivityDateTime, ActivityDisplayName, Category, LoggedByService, Result |
    Format-Table -AutoSize
```

```bash
# ============================================================
# Azure CLI Verification Commands
# ============================================================

TENANT_ID="<tenant-id>"

az account show --output table

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains" \
  --output table

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/subscribedSkus" \
  --output table

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/users?\$top=10&\$select=id,displayName,userPrincipalName,userType,accountEnabled,usageLocation" \
  --output table

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/groups?\$top=10&\$select=id,displayName,mailEnabled,securityEnabled,groupTypes" \
  --output table
```

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm emergency access is still valid | Admin workstation | Review workbook 05 | Recovery access protected |
| 2 | Remove test user license assignment if needed | Admin workstation | Set-MgUserLicense -RemoveLicenses | License freed |
| 3 | Remove test users from license group if needed | Admin workstation | Remove-MgGroupMemberByRef | Group-based license removed after processing |
| 4 | Remove test users from SSPR pilot group if needed | Admin workstation | Remove-MgGroupMemberByRef | User no longer scoped for SSPR pilot |
| 5 | Disable test user before deletion if investigation needed | Admin workstation | Update-MgUser -AccountEnabled:$false | User blocked from sign-in |
| 6 | Delete test cloud-only user if safe | Admin workstation | Remove-MgUser -UserId user@domain.com | User soft-deleted |
| 7 | Restore deleted user if needed | Admin workstation | Restore-MgDirectoryDeletedItem | Deleted user restored |
| 8 | Remove guest from scoped group | Admin workstation | Remove-MgGroupMemberByRef | Guest loses scoped access |
| 9 | Delete guest if approved | Admin workstation | Remove-MgUser -UserId <guest-id> | Guest soft-deleted |
| 10 | Remove test groups if unused | Admin workstation | Remove-MgGroup -GroupId <group-id> | Group deleted |
| 11 | Revert SSPR scope if pilot failed | Entra admin center | Password reset > Properties | SSPR set back to previous scope |
| 12 | Revert SSPR method settings if changed | Entra admin center | Password reset > Authentication methods | Previous method configuration restored |
| 13 | Revert auth method policy if changed | Entra admin center | Protection > Authentication methods | Previous auth method state restored |
| 14 | Disconnect Graph session | Admin workstation | Disconnect-MgGraph | Session closed |
| 15 | Remove sensitive exports if needed | Admin workstation | Remove-Item ".\Entra_Users_Groups_Licenses_SSPR_Baseline" -Recurse -Force | Local evidence removed |
| 16 | Document rollback | Admin workstation | Workbook notes | Audit trail preserved |

## Rollback Notes

Do not delete production users, production groups, production guests, or production license groups as rollback for a lab test.

For production objects:

- Disable before delete when investigation is needed
- Remove licenses before deleting if license recovery is required
- Remove group membership before deleting group if access impact must be staged
- Do not remove SSPR for all users without a helpdesk fallback
- Do not remove authentication methods without checking Conditional Access impact
- Do not delete synced users in cloud if they are mastered on-premises
- Do not delete emergency access accounts

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Failure_Checks

| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| New user creation fails | Missing User Administrator or User.ReadWrite.All permission | Admin workstation | Get-MgContext | Use correct admin role and Graph scope |
| User UPN domain rejected | Domain not verified | Admin workstation | Get-MgDomain -All | Verify custom domain first |
| User cannot sign in | Account disabled, password issue, CA block, license issue, or wrong UPN | Admin workstation | Get-MgUser; sign-in logs | Enable account, reset password, review CA |
| Usage location update fails | Missing permission or invalid location code | Admin workstation | Update-MgUser -UsageLocation "US" | Use valid country code and permissions |
| License assignment fails | Usage location missing | Admin workstation | Get-MgUser -Property UsageLocation | Set usage location first |
| License assignment fails | No available license units | Admin workstation | Get-MgSubscribedSku | Free license, buy more, or assign different SKU |
| License assignment fails | Conflicting service plan | Admin workstation | Get-MgUserLicenseDetail | Review direct/group license conflicts |
| User has license but app unavailable | Service plan disabled or provisioning pending | Admin workstation | Get-MgUserLicenseDetail | Enable needed service plan or wait for provisioning |
| Group creation fails | Missing Groups Administrator or Graph scope | Admin workstation | Get-MgContext | Use Group.ReadWrite.All and correct admin role |
| Group mail nickname conflict | MailNickname already used | Admin workstation | Get-MgGroup -Filter | Choose unique mailNickname |
| Microsoft 365 group creation fails | Missing permission or naming policy conflict | Admin workstation | New-MgGroup | Use approved name and role |
| Dynamic group not processing | Rule invalid or license requirement missing | Entra portal | Group membership rule validation | Fix rule and check licensing |
| Group-based license not applying | User missing usage location or group processing delay | Admin workstation | Get-MgUser; portal group license errors | Set usage location and review assignment errors |
| Guest invite fails | External collaboration settings block invite | Entra portal | External Identities settings | Adjust policy or use approved inviter |
| Guest cannot access resource | Guest invited but not assigned to app/group/resource | Admin workstation | Group membership review | Add guest to scoped access group |
| Guest appears as pending | Invitation not redeemed | Entra portal | Guest user state review | Resend invitation or verify external mailbox |
| SSPR option not available to user | User not in enabled SSPR group | Admin workstation | Group membership check | Add user to SSPR pilot group |
| SSPR registration blocked | Conditional Access blocks security info registration | Entra portal | Sign-in logs | Adjust CA policy for registration flow |
| SSPR reset fails | Not enough registered methods | User browser | https://aka.ms/mysecurityinfo | Register required methods |
| SSPR reset fails for admin | Admin SSPR requirements differ from standard users | Entra portal | Role membership check | Configure admin recovery method and process |
| Security questions unavailable | Policy disabled or not supported for target user | Entra portal | Password reset methods | Use stronger approved methods |
| User cannot change password after reset | Password policy, sync, or writeback issue | Admin workstation / sync server | Check user source | For synced users, validate password writeback |
| Synced user property change reverts | User mastered on-premises | Admin workstation | Get-MgUser OnPremisesSyncEnabled | Change attribute on-premises and sync |
| Deleted user still appears | Soft-delete retention | Admin workstation | Get-MgDirectoryDeletedItem | Restore or permanently delete only after approval |
| Duplicate users | Guest/member or sync conflict | Admin workstation | Search UPN/mail/object IDs | Resolve source of authority and duplicates |
| Graph command returns insufficient privileges | Missing delegated scope or user role | Admin workstation | Get-MgContext | Reconnect with scope and proper role |
| Portal and Graph disagree | Wrong tenant or stale portal session | Admin workstation | Get-MgContext; portal tenant selector | Align tenant and browser session |
| License group includes guests accidentally | Group membership too broad | Admin workstation | Get-MgGroupMember | Remove guests or separate group |
| SSPR enabled for emergency accounts | Emergency accounts included in broad group | Admin workstation | Group membership review | Exclude emergency accounts from normal SSPR groups |
| Auth method policy breaks legacy users | Methods restricted before registration | Entra portal | Registration and sign-in logs | Pilot first, then staged rollout |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Documentation_OUTPUT_TEMPLATE

## Tenant Identity Baseline

| Field | Value |
|---|---|
| Tenant display name |  |
| Tenant ID |  |
| Initial domain |  |
| Primary verified domain |  |
| Admin account used |  |
| Date collected |  |
| Evidence path |  |

## User Baseline

| Category | Count | Notes |
|---|---:|---|
| Total users |  |  |
| Member users |  |  |
| Guest users |  |  |
| Cloud-only members |  |  |
| Synced members |  |  |
| Disabled users |  |  |
| Users missing usage location |  |  |
| Licensed users |  |  |
| Unlicensed users |  |  |

## Test User Baseline

| Display Name | UPN | User Type | Source | Usage Location | Licensed | SSPR Pilot | Notes |
|---|---|---|---|---|---|---|---|
| Lab User 01 | labuser01@contoso.com | Member | Cloud-only | US | Yes / No | Yes / No |  |
| Lab User 02 | labuser02@contoso.com | Member | Cloud-only | US | Yes / No | Yes / No |  |
| Lab User 03 | labuser03@contoso.com | Member | Cloud-only | US | Yes / No | Yes / No |  |

## Group Baseline

| Group Name | Type | Membership Type | Purpose | Owner | Notes |
|---|---|---|---|---|---|
| SG-SSPR-Pilot | Security | Assigned | SSPR pilot |  |  |
| SG-SSPR-Enabled | Security | Assigned / Dynamic | SSPR production |  |  |
| SG-LIC-M365-Baseline | Security | Assigned | License assignment |  |  |
| SG-CA-MFA-Pilot | Security | Assigned | CA pilot |  |  |
| SG-GUEST-ProjectA | Security | Assigned | Guest scoping |  |  |
| M365-CloudOps-Collab | Microsoft 365 | Assigned | Collaboration |  |  |

## Guest Baseline

| Guest | Email | Invited By | Group Scope | Redemption Status | Review Date | Notes |
|---|---|---|---|---|---|---|
|  |  |  | SG-GUEST-ProjectA | Pending / Accepted |  |  |

## License Baseline

| SKU Part Number | SKU ID | Enabled Units | Consumed Units | Available Units | Assignment Model | Notes |
|---|---|---:|---:|---:|---|---|
|  |  |  |  |  | Direct / Group |  |

## Group-Based License Baseline

| License Group | SKU | Disabled Service Plans | Members | Assignment Errors | Notes |
|---|---|---|---|---|---|
| SG-LIC-M365-Baseline |  |  |  | Yes / No |  |

## SSPR Baseline

| Setting | Value | Notes |
|---|---|---|
| SSPR scope | None / Selected / All |  |
| SSPR pilot group | SG-SSPR-Pilot |  |
| Methods required to reset |  |  |
| Allowed methods |  |  |
| Require registration on sign-in | Yes / No |  |
| Reconfirm methods interval |  |  |
| Notify users on reset | Yes / No |  |
| Notify admins on admin reset | Yes / No |  |
| Helpdesk link/email |  |  |
| Test user |  |  |
| Test result | Pass / Fail |  |
| Last test date |  |  |

## Authentication Methods Baseline

| Method | Enabled | Target Users / Groups | Used For | Notes |
|---|---|---|---|---|
| Microsoft Authenticator | Yes / No |  | MFA / SSPR |  |
| FIDO2 security key | Yes / No |  | MFA |  |
| Temporary Access Pass | Yes / No |  | Bootstrap / recovery |  |
| SMS | Yes / No |  | MFA / SSPR |  |
| Voice call | Yes / No |  | MFA / SSPR |  |
| Email OTP | Yes / No |  | Guest / SSPR |  |
| Security questions | Yes / No |  | SSPR |  |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories | Supplies tenant, directory, subscription, and domain baseline |
| 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline | Supplies Graph and portal tooling |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces | Supplies verified domain and UPN namespace |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles | Supplies role model for user, group, license, guest, and SSPR administration |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline | Supplies safe admin and emergency access model |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking | Monitors user, group, guest, license, and SSPR changes |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Uses this identity baseline for sign-in and user access troubleshooting |
| M365 Tenant Administration Workbook | Deep dive for users, contacts, guests, groups, licensing, roles, and service health |
| Exchange Online Administration Workbook | Uses users, groups, license, and domain baselines for mailbox work |
| Teams Administration Workbook | Uses users, groups, guests, and licenses for collaboration admin |
| SharePoint OneDrive Workbook | Uses groups, guests, and licenses for site and sharing work |
| Intune Workbook | Uses users, groups, licenses, and auth methods for device enrollment |
| Defender XDR Workbook | Uses users, groups, roles, and licensing for security operations |
| Purview Workbook | Uses users, groups, guests, and license state for compliance workloads |
| Hybrid Identity Workbook | Extends cloud-only vs synced user source, SSPR, MFA, and password writeback |

---

# 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline_Completion_Criteria

| Requirement | Complete |
|---|---|
| Tenant ID confirmed |  |
| Verified domain confirmed |  |
| Graph context confirmed |  |
| Required admin roles confirmed |  |
| Existing users inventoried |  |
| Guest users inventoried |  |
| Groups inventoried |  |
| License SKUs inventoried |  |
| Users missing usage location identified |  |
| Cloud-only test users created if required |  |
| Usage location set for test users |  |
| Baseline security groups created |  |
| Baseline Microsoft 365 group created if required |  |
| Test users added to baseline groups |  |
| Guest invitation settings reviewed |  |
| Guest test invitation completed if approved |  |
| Guest scoped to access group |  |
| License assignment model documented |  |
| Direct test license assigned if required |  |
| Group-based licensing plan documented |  |
| Group-based license assignment configured if approved |  |
| License assignment errors reviewed |  |
| SSPR pilot group created |  |
| SSPR scope configured for pilot |  |
| SSPR authentication methods documented |  |
| SSPR registration settings documented |  |
| SSPR notifications documented |  |
| Test user security info registration completed |  |
| Test user password reset completed |  |
| Audit and sign-in evidence reviewed |  |
| Rollback notes completed |  |
| Evidence stored securely |  |

## Final Expected State

The admin can clearly state:

- These are the users in the tenant.
- These are the guest users in the tenant.
- These are the baseline groups and what they are used for.
- These are the available license SKUs.
- This is the license assignment model.
- These users have usage location configured.
- These users are licensed directly or through groups.
- These users are in the SSPR pilot group.
- SSPR is enabled for the planned scope.
- Authentication methods for SSPR are documented.
- A test user successfully registered security info.
- A test user successfully completed SSPR.
- Guest access is scoped and reviewable.
- Emergency access accounts are not broken by user, group, license, or SSPR changes.