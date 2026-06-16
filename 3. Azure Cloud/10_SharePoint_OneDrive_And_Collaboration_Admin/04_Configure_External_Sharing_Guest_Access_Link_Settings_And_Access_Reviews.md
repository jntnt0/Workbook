# 04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews

## Objective

Configure a SharePoint Online and OneDrive external collaboration baseline covering tenant-level external sharing, site-level sharing, guest access behavior, default link settings, anonymous link restrictions, domain allow/block controls, and access review handoff. This workbook establishes a controlled approach for sharing files and sites with external users without leaving broad anonymous access unmanaged.

---

## Prerequisites

| Requirement | Details |
|------------|---------|
| Microsoft 365 Tenant | Active tenant with SharePoint Online and OneDrive licensed |
| Role Assignment | SharePoint Administrator or Global Administrator |
| Entra Role | Global Administrator, Security Administrator, or Identity Governance Administrator for access reviews |
| Optional Compliance Role | Compliance Administrator for audit and DLP alignment |
| Previous Workbook | 03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md |
| SharePoint Admin Center | Accessible from Microsoft 365 Admin Center |
| Entra Admin Center | Accessible for guest and access review settings |
| PowerShell Module | Microsoft.Online.SharePoint.PowerShell |
| Graph Module | Microsoft.Graph, optional for validation |
| Test Internal User | user01@contoso.onmicrosoft.com |
| Test External User | guest.user@externaldomain.com |

---

## Lab Topology / Assumptions

| Component | Value |
|------------|---------|
| Tenant Name | Contoso |
| SharePoint Root URL | https://contoso.sharepoint.com |
| SharePoint Admin URL | https://contoso-admin.sharepoint.com |
| Test SharePoint Site | https://contoso.sharepoint.com/sites/FinanceTeam |
| Admin Account | spadmin@contoso.onmicrosoft.com |
| Internal Test User | user01@contoso.onmicrosoft.com |
| External Test User | guest.user@externaldomain.com |
| Default Sharing Baseline | New and existing guests |
| Anonymous Links | Disabled unless a business exception is approved |
| Default Link Type | Specific people |
| Default Link Permission | View |
| Guest Review Cadence | Quarterly |

---

# Phase 1 - Connect Administrative Tools

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Microsoft 365 Admin Center | Admin Workstation | https://admin.microsoft.com | N/A | Microsoft 365 Admin Center opens |
| 2 | Open SharePoint Admin Center | Admin Workstation | Admin centers → SharePoint | N/A | SharePoint Admin Center opens |
| 3 | Open Entra Admin Center | Admin Workstation | https://entra.microsoft.com | N/A | Entra Admin Center opens |
| 4 | Open PowerShell | Admin Workstation | Start PowerShell | N/A | PowerShell session opens |
| 5 | Install SharePoint Online module if missing | Admin Workstation | N/A | Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser | SPO module installed |
| 6 | Import SharePoint Online module | Admin Workstation | N/A | Import-Module Microsoft.Online.SharePoint.PowerShell | SPO module loaded |
| 7 | Connect to SharePoint Online | Admin Workstation | N/A | Connect-SPOService -Url https://contoso-admin.sharepoint.com | SPO admin session connected |
| 8 | Verify tenant connection | Admin Workstation | N/A | Get-SPOTenant | Tenant configuration displayed |

---

# Phase 2 - Review Current External Sharing Posture

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open sharing policy page | SharePoint Admin Center | Policies → Sharing | N/A | Tenant sharing page opens |
| 2 | Review SharePoint sharing level | SharePoint Admin Center | Sharing slider for SharePoint | N/A | Current SharePoint sharing level identified |
| 3 | Review OneDrive sharing level | SharePoint Admin Center | Sharing slider for OneDrive | N/A | Current OneDrive sharing level identified |
| 4 | Export current tenant sharing settings | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability, DefaultSharingLinkType, DefaultLinkPermission, RequireAnonymousLinksExpireInDays \| Format-List | Current sharing posture displayed |
| 5 | Review site-level sharing | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, SharingCapability | Site sharing level displayed |
| 6 | Export all site sharing states | Admin Workstation | N/A | Get-SPOSite -Limit All \| Select Url, Owner, Template, SharingCapability \| Export-Csv .\SPO-Site-Sharing-Before.csv -NoTypeInformation | Site sharing baseline exported |

---

# Phase 3 - Configure Tenant External Sharing Level

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open tenant sharing settings | SharePoint Admin Center | Policies → Sharing | N/A | Sharing configuration visible |
| 2 | Set SharePoint external sharing level | SharePoint Admin Center | Select New and existing guests | N/A | SharePoint allows authenticated external guests |
| 3 | Set OneDrive external sharing level | SharePoint Admin Center | Select Same or more restrictive than SharePoint | N/A | OneDrive sharing level configured |
| 4 | Avoid broad Anyone links for baseline | SharePoint Admin Center | Do not select Anyone unless required | N/A | Anonymous access avoided |
| 5 | Save sharing policy | SharePoint Admin Center | Save | N/A | Tenant sharing baseline applied |
| 6 | Apply with PowerShell if required | Admin Workstation | N/A | Set-SPOTenant -SharingCapability ExternalUserSharingOnly | Tenant sharing set to new and existing guests |
| 7 | Verify tenant sharing level | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability | SharingCapability shows ExternalUserSharingOnly |

---

# Phase 4 - Configure Default Link Settings

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open file and folder links settings | SharePoint Admin Center | Policies → Sharing → File and folder links | N/A | Link settings displayed |
| 2 | Configure default sharing link type | SharePoint Admin Center | Select Specific people | N/A | Default links require named recipients |
| 3 | Configure default link permission | SharePoint Admin Center | Select View | N/A | Default links are view-only |
| 4 | Disable default edit links | SharePoint Admin Center | Avoid Edit as default | N/A | Users must intentionally select edit |
| 5 | Save settings | SharePoint Admin Center | Save | N/A | Link baseline applied |
| 6 | Verify default link type | Admin Workstation | N/A | Get-SPOTenant \| Select DefaultSharingLinkType, DefaultLinkPermission | Specific people and View displayed |

---

# Phase 5 - Configure Anonymous Link Restrictions If Anyone Links Are Allowed

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Determine if Anyone links are required | Admin Workstation | Review business requirement | N/A | Requirement confirmed or rejected |
| 2 | Open Anyone link settings | SharePoint Admin Center | Policies → Sharing → Choose expiration and permissions options for Anyone links | N/A | Anonymous link controls displayed |
| 3 | Configure anonymous link expiration | SharePoint Admin Center | Set expiration days, example 30 | N/A | Anyone links expire automatically |
| 4 | Configure anonymous link permission | SharePoint Admin Center | View-only unless exception approved | N/A | Anonymous edit links avoided |
| 5 | Apply expiration with PowerShell if required | Admin Workstation | N/A | Set-SPOTenant -RequireAnonymousLinksExpireInDays 30 | Anonymous links expire after 30 days |
| 6 | Verify anonymous link expiration | Admin Workstation | N/A | Get-SPOTenant \| Select RequireAnonymousLinksExpireInDays | Expiration value displayed |
| 7 | Document exception | Admin Workstation | N/A | N/A | Business approval recorded |

---

# Phase 6 - Configure Domain Allow or Block List

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Identify external partner domains | Admin Workstation | Review approved vendor/customer domains | N/A | Domain list prepared |
| 2 | Open domain restriction settings | SharePoint Admin Center | Policies → Sharing → More external sharing settings | N/A | Domain controls visible |
| 3 | Choose allow or block strategy | SharePoint Admin Center | Limit external sharing by domain | N/A | Domain control mode selected |
| 4 | Configure allowed domains if strict baseline | SharePoint Admin Center | Add partnerdomain.com | N/A | Only approved domains allowed |
| 5 | Configure blocked domains if broad baseline | SharePoint Admin Center | Add blocked domains | N/A | Specific risky domains blocked |
| 6 | Save domain settings | SharePoint Admin Center | Save | N/A | Domain restrictions applied |
| 7 | Verify policy with PowerShell | Admin Workstation | N/A | Get-SPOTenant \| Select SharingDomainRestrictionMode, SharingAllowedDomainList, SharingBlockedDomainList | Domain policy displayed |

---

# Phase 7 - Configure Site-Level Sharing Policy

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Active Sites | SharePoint Admin Center | Sites → Active sites | N/A | Site inventory displayed |
| 2 | Select test site | SharePoint Admin Center | Select Finance Team | N/A | Site details panel opens |
| 3 | Open site sharing policy | SharePoint Admin Center | Policies → Sharing | N/A | Site sharing policy displayed |
| 4 | Set site sharing lower than or equal to tenant level | SharePoint Admin Center | Select New and existing guests or Existing guests only | N/A | Site does not exceed tenant policy |
| 5 | Save site sharing policy | SharePoint Admin Center | Save | N/A | Site sharing baseline applied |
| 6 | Apply with PowerShell if required | Admin Workstation | N/A | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -SharingCapability ExternalUserSharingOnly | Site sharing configured |
| 7 | Verify site sharing | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, SharingCapability | Expected sharing setting displayed |

---

# Phase 8 - Test External Guest Sharing

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Sign in as internal user | Client Workstation | Open Microsoft 365 as user01 | N/A | Internal user signed in |
| 2 | Open test SharePoint site | Browser | Open Finance Team site | N/A | Site opens |
| 3 | Upload test file | Browser | Upload ExternalSharingTest.docx | N/A | Test file uploaded |
| 4 | Share file with external guest | Browser | Share → Specific people → guest.user@externaldomain.com | N/A | Sharing invitation sent |
| 5 | Accept guest invitation | External Browser Session | Open invitation email | N/A | Guest redemption starts |
| 6 | Authenticate as guest | External Browser Session | Sign in with guest account | N/A | Guest accesses shared file |
| 7 | Validate guest permission | External Browser Session | Attempt view/edit based on permission | N/A | Access matches configured link permission |
| 8 | Verify guest object in Entra | Entra Admin Center | Identity → Users → All users → filter Guest | N/A | Guest user object exists |

---

# Phase 9 - Review Guest Users and External Access

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Entra users | Entra Admin Center | Identity → Users → All users | N/A | User list displayed |
| 2 | Filter guest users | Entra Admin Center | User type → Guest | N/A | Guest users listed |
| 3 | Review guest profile | Entra Admin Center | Select guest.user@externaldomain.com | N/A | Guest details displayed |
| 4 | Review guest sign-in activity | Entra Admin Center | Sign-in logs | N/A | Guest activity visible if logs available |
| 5 | Connect to Microsoft Graph if using PowerShell | Admin Workstation | N/A | Connect-MgGraph -Scopes "User.Read.All","AuditLog.Read.All" | Graph session connected |
| 6 | List guest users with Graph | Admin Workstation | N/A | Get-MgUser -Filter "userType eq 'Guest'" -All \| Select DisplayName, UserPrincipalName, UserType | Guest users listed |
| 7 | Export guest users | Admin Workstation | N/A | Get-MgUser -Filter "userType eq 'Guest'" -All \| Select DisplayName, UserPrincipalName, UserType, AccountEnabled \| Export-Csv .\Entra-GuestUsers.csv -NoTypeInformation | Guest list exported |

---

# Phase 10 - Configure Guest Access Review Handoff

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Identity Governance | Entra Admin Center | Identity governance → Access reviews | N/A | Access reviews page opens |
| 2 | Start new access review | Entra Admin Center | New access review | N/A | Access review wizard opens |
| 3 | Select review target | Entra Admin Center | Teams + Groups or Users | N/A | Review scope selected |
| 4 | Scope to guest users | Entra Admin Center | Select guest users only where available | N/A | Guest review scope configured |
| 5 | Select reviewers | Entra Admin Center | Group owners or selected reviewers | N/A | Reviewers assigned |
| 6 | Configure recurrence | Entra Admin Center | Quarterly | N/A | Review cadence configured |
| 7 | Configure action on denial or no response | Entra Admin Center | Remove access or require manual decision | N/A | Remediation behavior configured |
| 8 | Create access review | Entra Admin Center | Create | N/A | Guest access review created |
| 9 | Document review owner | Admin Workstation | N/A | N/A | Operational owner recorded |

---

# Phase 11 - Review External Sharing Audit and Reporting

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Microsoft Purview portal | Admin Workstation | https://purview.microsoft.com | N/A | Purview opens |
| 2 | Open audit search | Purview Portal | Audit | N/A | Audit search available |
| 3 | Search sharing events | Purview Portal | Activities related to sharing and access requests | N/A | Sharing events returned |
| 4 | Filter by test user | Purview Portal | user01@contoso.onmicrosoft.com | N/A | Test sharing events visible |
| 5 | Filter by date range | Purview Portal | Current lab date | N/A | Relevant events shown |
| 6 | Export audit results if required | Purview Portal | Export | N/A | Audit record preserved |
| 7 | Document evidence location | Admin Workstation | N/A | N/A | Audit evidence recorded |

---

# Phase 12 - Export External Sharing Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Export tenant sharing baseline | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability, DefaultSharingLinkType, DefaultLinkPermission, RequireAnonymousLinksExpireInDays, SharingDomainRestrictionMode, SharingAllowedDomainList, SharingBlockedDomainList \| Export-Csv .\SPO-Tenant-ExternalSharing-Baseline.csv -NoTypeInformation | Tenant sharing baseline exported |
| 2 | Export site sharing baseline | Admin Workstation | N/A | Get-SPOSite -Limit All \| Select Url, Owner, Template, SharingCapability \| Export-Csv .\SPO-Site-ExternalSharing-Baseline.csv -NoTypeInformation | Site sharing baseline exported |
| 3 | Export guest users | Admin Workstation | N/A | Get-MgUser -Filter "userType eq 'Guest'" -All \| Select DisplayName, UserPrincipalName, AccountEnabled, UserType \| Export-Csv .\Entra-GuestUsers-Baseline.csv -NoTypeInformation | Guest user baseline exported |
| 4 | Confirm exported files | Admin Workstation | N/A | Get-ChildItem .\*Sharing*Baseline*.csv, .\Entra-GuestUsers*.csv | Export files listed |
| 5 | Store baseline evidence | Admin Workstation | N/A | N/A | Baseline archived |

---

# Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Verify tenant sharing level | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability | Expected sharing level displayed |
| 2 | Verify default link type | Admin Workstation | N/A | Get-SPOTenant \| Select DefaultSharingLinkType | Specific people displayed |
| 3 | Verify default link permission | Admin Workstation | N/A | Get-SPOTenant \| Select DefaultLinkPermission | View displayed |
| 4 | Verify anonymous expiration | Admin Workstation | N/A | Get-SPOTenant \| Select RequireAnonymousLinksExpireInDays | Expected expiration displayed |
| 5 | Verify domain restrictions | Admin Workstation | N/A | Get-SPOTenant \| Select SharingDomainRestrictionMode, SharingAllowedDomainList, SharingBlockedDomainList | Domain policy displayed |
| 6 | Verify site sharing policy | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select SharingCapability | Site sharing displayed |
| 7 | Verify guest object creation | Entra Admin Center | Users → All users → Guest filter | N/A | Guest object visible |
| 8 | Verify guest can access shared file | External Browser Session | Open shared link | N/A | Guest access succeeds |
| 9 | Verify non-invited external user blocked | External Browser Session | Try link with different external account | N/A | Access denied |
| 10 | Verify access review exists | Entra Admin Center | Identity governance → Access reviews | N/A | Review visible |
| 11 | Verify export files exist | Admin Workstation | N/A | Test-Path .\SPO-Tenant-ExternalSharing-Baseline.csv | Export file exists |

---

# Troubleshooting

| Issue | Cause | Resolution |
|------|-------|------------|
| External user cannot access shared file | Sharing disabled at tenant or site level | Verify tenant and site SharingCapability |
| Site sharing option is grayed out | Site cannot exceed tenant sharing setting | Raise tenant setting first or keep site more restrictive |
| Anyone links unavailable | Tenant set to authenticated guest sharing only | Use Specific people links or change policy with approval |
| External domain blocked | Domain restriction policy blocks the domain | Remove from block list or add to allowed domain list |
| Guest invitation not received | Mail filtering or wrong address | Check junk mail, resend invite, validate address |
| Guest signs in but sees access denied | Wrong guest identity or missing permission | Reshare to the exact external identity |
| Default link settings not visible immediately | Portal propagation delay | Wait and verify with Get-SPOTenant |
| Access review creation unavailable | Missing Identity Governance licensing or role | Assign correct license and Identity Governance role |
| Graph command fails | Missing module or consent | Install Microsoft.Graph and connect with required scopes |
| Audit events missing | Audit delay or insufficient permissions | Wait for audit ingestion and verify Purview role |

---

# Rollback / Cleanup

| Step | Action | PowerShell |
|------|--------|------------|
| 1 | Remove test sharing link | Remove sharing from file through Manage access |
| 2 | Remove test external guest access from file | Manage access → Remove guest |
| 3 | Remove guest user if lab-only | Remove-MgUser -UserId <GuestUserId> |
| 4 | Restore previous tenant sharing level | Set-SPOTenant -SharingCapability <PreviousValue> |
| 5 | Restore previous site sharing level | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -SharingCapability <PreviousValue> |
| 6 | Remove anonymous link expiration exception | Set-SPOTenant -RequireAnonymousLinksExpireInDays <PreviousValue> |
| 7 | Remove domain allow/block test entries | SharePoint Admin Center → Policies → Sharing |
| 8 | Delete test access review if lab-only | Entra Admin Center → Identity Governance → Access reviews |
| 9 | Remove exported baseline files if required | Remove-Item .\SPO-*ExternalSharing*.csv, .\Entra-GuestUsers*.csv |
| 10 | Disconnect SPO session | Disconnect-SPOService |
| 11 | Disconnect Graph session | Disconnect-MgGraph |

---

# Related Workbooks

```text
01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md
02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md
03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md
05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restore.md
06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files.md
07_Troubleshoot_SharePoint_OneDrive_Sync_Sharing_Permissions_And_Restore_Issues.md
```