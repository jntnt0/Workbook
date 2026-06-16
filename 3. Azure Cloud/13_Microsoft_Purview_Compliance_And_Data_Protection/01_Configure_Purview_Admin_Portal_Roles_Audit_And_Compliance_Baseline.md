# 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline

## Objective

Configure the baseline Microsoft Purview administration environment for Microsoft 365 compliance operations.

This workbook covers:

- Microsoft Purview portal access
- Compliance role group review
- Least-privilege admin assignment
- Audit search readiness
- Unified audit log validation
- Compliance baseline documentation
- Service health and message center handoff
- PowerShell validation
- Troubleshooting and rollback

## Lab Context

This workbook establishes the first operational baseline for the Microsoft Purview workbook set.

Do this before configuring:

- Sensitive information types
- Trainable classifiers
- Sensitivity labels
- Retention labels
- Retention policies
- DLP policies
- Endpoint DLP
- Content Explorer
- Activity Explorer
- DLP alerts
- Compliance alert investigations

The goal is to confirm that the tenant has the correct admin access, role structure, audit visibility, and baseline validation before any compliance enforcement policies are created.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft 365 tenant | Required |
| Microsoft Purview portal access | Required |
| Dedicated admin account | Recommended |
| Global Administrator | Required only for initial role delegation |
| Compliance Administrator | Required for most Purview administration |
| Audit Reader or Audit Manager | Required for audit investigation workflows |
| Exchange Online PowerShell module | Required for audit log validation |
| Microsoft Graph PowerShell SDK | Optional but recommended |
| Test user account | Recommended |
| Break-glass account | Required for emergency access model |
| MFA for admin accounts | Required |
| Modern browser | Microsoft Edge or Chrome recommended |

## Topology / Scope

| Component | Purpose |
|---|---|
| Microsoft Purview portal | Main compliance administration portal |
| Microsoft 365 admin center | Licensing, service health, and message center validation |
| Microsoft Entra admin center | Identity, users, groups, roles, and break-glass review |
| Exchange Online PowerShell | Unified audit log and role group validation |
| Microsoft Graph PowerShell | Optional tenant and identity validation |
| Audit solution | Central audit search and investigation area |
| Compliance role groups | Purview permission delegation model |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| Purview admin account | adm-purview-admin01 |
| Compliance admin group | GRP-Purview-Compliance-Admins |
| Audit reader group | GRP-Purview-Audit-Readers |
| Test user | user-purview-test01 |
| Evidence folder | Purview/Admin-Baseline |
| Role export | Purview_RoleGroups_YYYYMMDD.csv |
| Audit export | Purview_Audit_Baseline_YYYYMMDD.csv |

## Admin Access Model

| Account / Group | Purpose | Recommended Access |
|---|---|---|
| Daily user account | Normal productivity | No admin roles |
| Named Purview admin | Routine compliance administration | Compliance Administrator or workload-specific role |
| Audit investigator | Search and review audit data | Audit Reader or Audit Manager |
| Records admin | Retention and records configuration | Records Management |
| DLP admin | Data loss prevention configuration | DLP Compliance Management |
| Information protection admin | Labels and classification administration | Information Protection Admin |
| Break-glass account | Emergency access | Global Administrator, monitored, not used daily |
| Security group | Role assignment container | Preferred over individual assignments |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Sign in to Microsoft 365 admin center | Admin workstation | https://admin.microsoft.com | N/A | Microsoft 365 admin center opens |
| 2 | Confirm the signed-in account | Admin workstation | Profile menu | N/A | Correct admin account is signed in |
| 3 | Review tenant licensing | Admin workstation | Billing > Licenses | N/A | Required Microsoft 365 and compliance licenses are visible |
| 4 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Microsoft Purview portal opens |
| 5 | Confirm Purview portal access | Admin workstation | Purview home page | N/A | No access denied error appears |
| 6 | Review left navigation | Admin workstation | Purview portal navigation | N/A | Solutions, Settings, and role areas are visible |
| 7 | Open Settings | Admin workstation | Settings | N/A | Tenant Purview settings are visible |
| 8 | Open Roles and scopes | Admin workstation | Settings > Roles and scopes | N/A | Role groups page opens |
| 9 | Review built-in role groups | Admin workstation | Roles and scopes > Role groups | N/A | Compliance role groups are listed |
| 10 | Search for Compliance Administrator | Admin workstation | Search role groups | N/A | Compliance Administrator role group is found |
| 11 | Open Compliance Administrator role group | Admin workstation | Select role group | N/A | Role group details open |
| 12 | Review assigned members | Admin workstation | Members tab | N/A | Current members are displayed |
| 13 | Add dedicated compliance admin group | Admin workstation | Edit members | N/A | GRP-Purview-Compliance-Admins is added |
| 14 | Search for Audit Reader | Admin workstation | Search role groups | N/A | Audit Reader role group is found |
| 15 | Review Audit Reader membership | Admin workstation | Open role group | N/A | Members are displayed |
| 16 | Add audit investigator group if required | Admin workstation | Edit members | N/A | Audit reader group is added |
| 17 | Search for Audit Manager | Admin workstation | Search role groups | N/A | Audit Manager role group is found |
| 18 | Review Audit Manager membership | Admin workstation | Open role group | N/A | Members are displayed |
| 19 | Add audit manager group if required | Admin workstation | Edit members | N/A | Audit manager group is added |
| 20 | Search for Information Protection Admin | Admin workstation | Search role groups | N/A | Information protection role group is found |
| 21 | Review Information Protection Admin membership | Admin workstation | Open role group | N/A | Members are displayed |
| 22 | Search for DLP role groups | Admin workstation | Search: DLP | N/A | DLP-related role groups are visible |
| 23 | Search for Records Management role group | Admin workstation | Search: Records | N/A | Records Management role group is visible |
| 24 | Open Audit solution | Admin workstation | Solutions > Audit | N/A | Audit page opens |
| 25 | Run basic audit search | Admin workstation | Audit > Search | N/A | Audit search starts successfully |
| 26 | Set audit search time range | Admin workstation | Last 24 hours | N/A | Time range is accepted |
| 27 | Review audit result behavior | Admin workstation | Search results | N/A | Results return or no results message appears without permission error |
| 28 | Open Microsoft 365 service health | Admin workstation | Admin center > Health > Service health | N/A | Service health page opens |
| 29 | Review Purview or compliance advisories | Admin workstation | Service health search/filter | N/A | Relevant advisories can be reviewed |
| 30 | Open Message center | Admin workstation | Admin center > Health > Message center | N/A | Message center opens |
| 31 | Search for Purview messages | Admin workstation | Search: Purview, compliance, audit, DLP, sensitivity | N/A | Upcoming changes can be reviewed |
| 32 | Review Entra admin roles | Admin workstation | https://entra.microsoft.com > Roles and administrators | N/A | Admin role assignments are visible |
| 33 | Review break-glass accounts | Admin workstation | Entra ID > Users | N/A | Break-glass accounts are identified |
| 34 | Confirm MFA for named admins | Admin workstation | Entra ID > Users / Authentication methods | N/A | Admin MFA readiness is confirmed |
| 35 | Document baseline evidence | Admin workstation | Evidence folder | N/A | Baseline notes are recorded |

## PowerShell / CLI Validation

## Install Required Modules

Run from an elevated or normal PowerShell session depending on workstation policy.

    Install-Module ExchangeOnlineManagement -Scope CurrentUser
    Install-Module Microsoft.Graph -Scope CurrentUser

Expected result:

    Required PowerShell modules install successfully or already exist.

## Connect to Exchange Online

    Connect-ExchangeOnline

Expected result:

    Exchange Online PowerShell connects successfully.

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","RoleManagement.Read.Directory","User.Read.All","Group.Read.All"

Expected result:

    Microsoft Graph connects successfully with the requested scopes.

## Validate Current Graph Context

    Get-MgContext

Expected result:

    Tenant ID, account, and granted scopes are displayed.

## Validate Organization Audit Setting

    Get-OrganizationConfig | Select-Object DisplayName,AuditDisabled

Expected result:

    AuditDisabled should be False.

Interpretation:

| AuditDisabled Value | Meaning |
|---|---|
| False | Organization auditing is enabled |
| True | Organization auditing is disabled and must be investigated |
| Blank / unavailable | Validate audit status through Purview portal and audit search behavior |

## Validate Unified Audit Log Search

    Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-1) -EndDate (Get-Date) -ResultSize 10

Expected result:

    Command completes successfully.

A successful command may return records, or it may return no records if there was no matching activity. The important result is that it does not fail with a permissions or audit-disabled error.

## Search for Recent Sign-In or User Activity

    Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-1) -EndDate (Get-Date) -Operations UserLoggedIn -ResultSize 10

Expected result:

    Search completes successfully.

## Search for Recent Exchange Admin Activity

    Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -RecordType ExchangeAdmin -ResultSize 50

Expected result:

    Search completes successfully and returns matching activity if available.

## Review Compliance-Related Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Audit|eDiscovery|Records|DLP|Information|Insider|Communication"
    } | Select-Object Name

Expected result:

    Compliance-related role groups are listed.

## Review Compliance Management Membership

    Get-RoleGroupMember "Compliance Management"

Expected result:

    Members of the Compliance Management role group are displayed.

## Review Organization Management Membership

    Get-RoleGroupMember "Organization Management"

Expected result:

    Organization Management members are displayed.

Use this to identify broad admin exposure. Do not use Organization Management as the default model for Purview administration.

## Review Audit Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Audit"
    } | Select-Object Name

Expected result:

    Audit-related role groups are listed.

## Export Compliance Role Group Membership

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_RoleGroup_Membership_$Date.csv"

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Audit|eDiscovery|Records|DLP|Information|Insider|Communication"
    } | ForEach-Object {
        $RoleGroup = $_.Name
        Get-RoleGroupMember $RoleGroup -ErrorAction SilentlyContinue | Select-Object `
            @{Name="RoleGroup";Expression={$RoleGroup}},
            Name,
            PrimarySmtpAddress,
            RecipientType
    } | Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV file is created with Purview and compliance role group membership evidence.

## Export Recent Audit Baseline Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $StartDate = (Get-Date).AddDays(-1)
    $EndDate = Get-Date
    $OutputPath = ".\Purview_Audit_Baseline_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate $StartDate `
        -EndDate $EndDate `
        -ResultSize 100 |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV file is created with recent audit search results or an empty export if no matching records were returned.

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

Expected result:

    PowerShell sessions disconnect cleanly.

## Baseline Configuration Checklist

| Area | Required State | Status |
|---|---|---|
| Purview portal access | Confirmed | Not Started |
| Microsoft 365 admin center access | Confirmed | Not Started |
| Licensing reviewed | Confirmed | Not Started |
| Dedicated Purview admin account | Confirmed | Not Started |
| Compliance Administrator assignment | Confirmed | Not Started |
| Audit Reader or Audit Manager assignment | Confirmed | Not Started |
| Information Protection role reviewed | Confirmed | Not Started |
| DLP role reviewed | Confirmed | Not Started |
| Records Management role reviewed | Confirmed | Not Started |
| Break-glass account documented | Confirmed | Not Started |
| MFA for admins reviewed | Confirmed | Not Started |
| Unified audit log search tested | Confirmed | Not Started |
| Audit baseline exported | Recommended | Not Started |
| Role group membership exported | Recommended | Not Started |
| Service health location documented | Confirmed | Not Started |
| Message center location documented | Confirmed | Not Started |

## Recommended Role Group Mapping

| Administrative Function | Recommended Role / Role Group | Notes |
|---|---|---|
| Broad compliance administration | Compliance Administrator | Use sparingly |
| General compliance operations | Compliance Management | Use for compliance admins who manage multiple Purview areas |
| Audit investigation | Audit Reader | Read audit logs without broader admin control |
| Audit management | Audit Manager | Higher audit capability than reader |
| Information protection | Information Protection Admin | Sensitivity labels and classification administration |
| DLP administration | DLP Compliance Management | DLP policies, alerts, and related configuration |
| Records management | Records Management | Retention labels, records, and lifecycle configuration |
| eDiscovery investigation | eDiscovery Manager | Legal and investigation workflows |
| Communication compliance | Communication Compliance | Communication compliance administration |
| Insider risk management | Insider Risk Management | Insider risk workflows |
| Global tenant administration | Global Administrator | Initial delegation and emergency use only |

## Validation Tests

## Test 1: Purview Portal Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Browse to https://purview.microsoft.com | Purview portal opens |
| 2 | Select Settings | Settings page opens |
| 3 | Select Roles and scopes | Role group page opens |
| 4 | Select Solutions | Compliance solutions are visible |

## Test 2: Role Group Visibility

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Roles and scopes | Role groups appear |
| 2 | Search Compliance Administrator | Role group appears |
| 3 | Search Audit Reader | Role group appears |
| 4 | Search DLP | DLP role groups appear |
| 5 | Search Records | Records role group appears |

## Test 3: Role Assignment

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Compliance Administrator role group | Details open |
| 2 | Review members | Members are visible |
| 3 | Add admin security group | Group is accepted |
| 4 | Save changes | Membership update succeeds |
| 5 | Reopen role group | Group remains assigned |

## Test 4: Audit Search

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Audit solution | Audit page opens |
| 2 | Set date range to last 24 hours | Date range is accepted |
| 3 | Start search | Search starts |
| 4 | Review results | Results or no results message appears |
| 5 | Confirm no permission error | Audit access is valid |

## Test 5: PowerShell Audit Validation

| Step | Command | Expected Result |
|---|---|---|
| 1 | Connect-ExchangeOnline | Connection succeeds |
| 2 | Get-OrganizationConfig | Organization settings return |
| 3 | Search-UnifiedAuditLog | Search completes |
| 4 | Export-Csv | Evidence export is created |
| 5 | Disconnect-ExchangeOnline | Session closes |

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Purview portal access denied | Account lacks Purview/compliance role | Assign Compliance Administrator or required role group |
| Roles and scopes missing | Insufficient permissions | Use Global Administrator for initial delegation or add proper compliance role |
| Audit page missing | Missing audit role | Assign Audit Reader or Audit Manager |
| Audit search fails with permissions error | Account lacks audit permissions | Add account or group to audit role group |
| Audit search returns no results | No matching activity or ingestion delay | Expand date range and retry later |
| AuditDisabled shows True | Organization auditing disabled | Investigate and enable auditing through supported tenant process |
| PowerShell module missing | ExchangeOnlineManagement not installed | Install module |
| Connect-ExchangeOnline fails | MFA, network, or account issue | Validate admin account, MFA, and conditional access |
| Role membership change not effective | Propagation delay | Wait and retest |
| Too many users have compliance access | Role sprawl | Replace individual users with scoped security groups |
| Break-glass account used for routine admin | Poor operational practice | Remove from daily workflows and monitor sign-ins |
| Content or activity areas unavailable | Licensing or role limitation | Confirm licensing and role group membership |
| Message center not visible | Missing admin center role | Assign appropriate M365 admin role |

## Common Commands

## Connect to Exchange Online

    Connect-ExchangeOnline

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","RoleManagement.Read.Directory","User.Read.All","Group.Read.All"

## Check Organization Audit Setting

    Get-OrganizationConfig | Select-Object DisplayName,AuditDisabled

## Search Recent Unified Audit Log Events

    Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-1) -EndDate (Get-Date) -ResultSize 25

## Search Exchange Admin Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -RecordType ExchangeAdmin `
        -ResultSize 50

## List Compliance-Related Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Audit|DLP|eDiscovery|Records|Information|Insider|Communication"
    } | Select-Object Name

## Get Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

## Get Audit Reader Members

    Get-RoleGroupMember "Audit Reader"

## Get Audit Manager Members

    Get-RoleGroupMember "Audit Manager"

## Export Role Groups

    Get-RoleGroup | Select-Object Name,ManagedBy,Roles | Export-Csv ".\Purview_RoleGroups.csv" -NoTypeInformation

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Added user to Compliance Administrator | Remove user from role group |
| Added group to Compliance Administrator | Remove group from role group |
| Added audit reader group | Remove group from Audit Reader |
| Added audit manager group | Remove group from Audit Manager |
| Exported audit data | Store securely or delete lab-only export |
| Installed PowerShell modules | Leave installed or uninstall if required |
| Created evidence files | Archive or delete based on lab policy |
| Granted broad admin access | Replace with least-privilege role group |
| Used Global Administrator for setup | Remove daily dependency on Global Administrator |

## Remove User From Compliance Management

    Remove-RoleGroupMember `
        -Identity "Compliance Management" `
        -Member "adm-purview-admin01@contoso.com" `
        -Confirm:$false

Expected result:

    User is removed from the Compliance Management role group.

## Remove User From Audit Reader

    Remove-RoleGroupMember `
        -Identity "Audit Reader" `
        -Member "adm-purview-audit01@contoso.com" `
        -Confirm:$false

Expected result:

    User is removed from the Audit Reader role group.

## Validate Removal

    Get-RoleGroupMember "Compliance Management"
    Get-RoleGroupMember "Audit Reader"

Expected result:

    Removed users or groups are no longer listed.

## Security Notes

- Do not use Global Administrator for routine Purview administration.
- Assign Purview access through role groups where possible.
- Use security groups for repeatable access assignment.
- Require MFA for all admin accounts.
- Monitor break-glass account sign-ins.
- Keep break-glass accounts out of routine workflows.
- Audit exports may contain sensitive user, admin, mailbox, SharePoint, OneDrive, Teams, and compliance activity.
- Store exported evidence in a restricted location.
- Review compliance roles regularly.
- Avoid assigning broad eDiscovery or Content Explorer access without a real investigation need.
- Document every privileged role assignment.

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| Purview portal access screenshot | Browser screenshot | Evidence folder |
| Role group membership export | PowerShell Export-Csv | Purview_RoleGroup_Membership_YYYYMMDD.csv |
| Audit search export | PowerShell Export-Csv | Purview_Audit_Baseline_YYYYMMDD.csv |
| Licensing screenshot | Microsoft 365 admin center | Evidence folder |
| Service health screenshot | Microsoft 365 admin center | Evidence folder |
| Message center screenshot | Microsoft 365 admin center | Evidence folder |
| Break-glass account review | Admin notes | Workbook notes |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Microsoft 365 admin center opens |  |
| Microsoft Purview portal opens |  |
| Purview Settings page is visible |  |
| Roles and scopes page is visible |  |
| Compliance Administrator role group reviewed |  |
| Audit Reader role group reviewed |  |
| Audit Manager role group reviewed |  |
| DLP role groups reviewed |  |
| Records Management role group reviewed |  |
| Information Protection Admin role group reviewed |  |
| Dedicated admin group assigned |  |
| Break-glass account documented |  |
| MFA reviewed for admin accounts |  |
| Unified audit search works in portal |  |
| Unified audit search works in PowerShell |  |
| Role group membership exported |  |
| Audit baseline exported |  |
| Service health reviewed |  |
| Message center reviewed |  |
| Least privilege reviewed |  |

## Exam / Interview Notes

- Microsoft Purview is the main Microsoft 365 compliance administration platform.
- Purview administration should be delegated through role groups.
- Global Administrator should not be used for daily compliance administration.
- Compliance Administrator is broad and should be assigned carefully.
- Audit Reader allows audit visibility without full compliance administration.
- Audit Manager has broader audit management capability than Audit Reader.
- Unified audit log is central for Microsoft 365 compliance investigations.
- Audit records can have ingestion delay.
- A search returning no records is different from a search failing due to permissions.
- Role assignment changes may require propagation time.
- Purview workloads often include audit, DLP, sensitivity labels, retention, eDiscovery, records management, insider risk, communication compliance, and information protection.
- Service health and message center should be checked before assuming a tenant configuration issue.
- Least privilege matters because Purview can expose sensitive user, file, mailbox, Teams, SharePoint, OneDrive, and admin activity.
- A good Purview baseline validates portal access, role assignment, audit search, exports, and operational evidence before enforcement policies are configured.

## Related Workbooks

| Workbook | Relationship |
|---|---|
| 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md | Builds classification foundation after portal and role baseline |
| 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md | Uses Purview roles and information protection admin access |
| 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md | Uses compliance and records role baseline |
| 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md | Uses DLP role baseline |
| 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md | Extends DLP operations to endpoints |
| 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md | Uses audit and monitoring baseline |
| 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts.md | Uses baseline validation and troubleshooting workflow |