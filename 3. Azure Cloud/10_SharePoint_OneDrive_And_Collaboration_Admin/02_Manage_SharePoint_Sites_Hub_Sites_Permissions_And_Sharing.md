# 02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing

## Objective

Create and manage SharePoint Online sites, hub sites, site permissions, site ownership, and sharing settings from the SharePoint Admin Center and SharePoint Online PowerShell. This workbook establishes the practical administration workflow for managing collaboration sites after the tenant baseline is configured.

---

## Prerequisites

| Requirement | Details |
|------------|---------|
| Microsoft 365 Tenant | Active tenant with SharePoint Online licensed |
| Role Assignment | SharePoint Administrator or Global Administrator |
| Previous Workbook | 01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md |
| SharePoint Admin Center | Accessible from Microsoft 365 Admin Center |
| PowerShell Module | Microsoft.Online.SharePoint.PowerShell |
| Admin URL | https://contoso-admin.sharepoint.com |
| Test Users | user01@contoso.onmicrosoft.com, user02@contoso.onmicrosoft.com |
| Test Group | SG-SPO-Finance-Members |

---

## Lab Topology / Assumptions

| Component | Value |
|------------|---------|
| Tenant Name | Contoso |
| SharePoint Root URL | https://contoso.sharepoint.com |
| SharePoint Admin URL | https://contoso-admin.sharepoint.com |
| Admin Account | spadmin@contoso.onmicrosoft.com |
| Communication Site | Finance Portal |
| Team Site | Finance Team |
| Hub Site | Contoso Finance Hub |
| Site Owner | user01@contoso.onmicrosoft.com |
| Site Member | user02@contoso.onmicrosoft.com |

---

# Phase 1 - Connect to SharePoint Administration

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open SharePoint Admin Center | Admin Workstation | Microsoft 365 Admin Center → Admin centers → SharePoint | N/A | SharePoint Admin Center opens |
| 2 | Open PowerShell as administrator | Admin Workstation | Start PowerShell | N/A | PowerShell session opens |
| 3 | Install SharePoint Online module if missing | Admin Workstation | N/A | Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser | SPO module installed |
| 4 | Import SharePoint Online module | Admin Workstation | N/A | Import-Module Microsoft.Online.SharePoint.PowerShell | Module loaded |
| 5 | Connect to SharePoint Online admin service | Admin Workstation | N/A | Connect-SPOService -Url https://contoso-admin.sharepoint.com | Admin session connected |
| 6 | Verify tenant connection | Admin Workstation | N/A | Get-SPOTenant | Tenant settings displayed |

---

# Phase 2 - Review Existing SharePoint Sites

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Active Sites | SharePoint Admin Center | Sites → Active sites | N/A | Site inventory appears |
| 2 | Review existing sites | SharePoint Admin Center | Active sites list | N/A | Existing sites identified |
| 3 | Filter by site type | SharePoint Admin Center | Filter → Site type | N/A | Team, Communication, and other site types visible |
| 4 | Review storage and activity columns | SharePoint Admin Center | Customize columns if needed | N/A | Storage and activity information visible |
| 5 | Export site inventory | SharePoint Admin Center | Export | N/A | CSV export downloaded |
| 6 | Review sites with PowerShell | Admin Workstation | N/A | Get-SPOSite -Limit All | All site collections listed |

---

# Phase 3 - Create a Communication Site

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Start site creation | SharePoint Admin Center | Sites → Active sites → Create | N/A | Create site wizard opens |
| 2 | Select Communication site | SharePoint Admin Center | Communication site | N/A | Communication site template selected |
| 3 | Enter site name | SharePoint Admin Center | Site name: Finance Portal | N/A | Site name accepted |
| 4 | Enter site address | SharePoint Admin Center | /sites/FinancePortal | N/A | Site URL generated |
| 5 | Assign owner | SharePoint Admin Center | Owner: user01@contoso.onmicrosoft.com | N/A | Owner assigned |
| 6 | Set language and time zone | SharePoint Admin Center | Select tenant standard | N/A | Regional settings configured |
| 7 | Finish creation | SharePoint Admin Center | Finish | N/A | Communication site created |
| 8 | Verify site with PowerShell | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinancePortal | Site details displayed |

---

# Phase 4 - Create a Team Site

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Start site creation | SharePoint Admin Center | Sites → Active sites → Create | N/A | Create site wizard opens |
| 2 | Select Team site | SharePoint Admin Center | Team site | N/A | Team site template selected |
| 3 | Enter site name | SharePoint Admin Center | Site name: Finance Team | N/A | Site name accepted |
| 4 | Enter site address | SharePoint Admin Center | /sites/FinanceTeam | N/A | Site URL generated |
| 5 | Assign owner | SharePoint Admin Center | Owner: user01@contoso.onmicrosoft.com | N/A | Owner assigned |
| 6 | Add member | SharePoint Admin Center | Member: user02@contoso.onmicrosoft.com | N/A | Member added |
| 7 | Finish creation | SharePoint Admin Center | Finish | N/A | Team site created |
| 8 | Verify site with PowerShell | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam | Site details displayed |

---

# Phase 5 - Register a Hub Site

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Select Communication Site | SharePoint Admin Center | Active sites → Finance Portal | N/A | Finance Portal selected |
| 2 | Open Hub menu | SharePoint Admin Center | Hub → Register as hub site | N/A | Hub registration panel opens |
| 3 | Enter hub name | SharePoint Admin Center | Hub name: Contoso Finance Hub | N/A | Hub name entered |
| 4 | Configure people who can associate sites | SharePoint Admin Center | Select allowed users or groups | N/A | Association permissions configured |
| 5 | Save hub registration | SharePoint Admin Center | Save | N/A | Hub site registered |
| 6 | Verify hub site | Admin Workstation | N/A | Get-SPOHubSite | Hub site listed |

---

# Phase 6 - Associate a Site to the Hub

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Select Team Site | SharePoint Admin Center | Active sites → Finance Team | N/A | Team site selected |
| 2 | Open Hub association | SharePoint Admin Center | Hub → Associate with a hub | N/A | Hub association panel opens |
| 3 | Select hub site | SharePoint Admin Center | Contoso Finance Hub | N/A | Hub selected |
| 4 | Save association | SharePoint Admin Center | Save | N/A | Site associated to hub |
| 5 | Verify association | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, HubSiteId | HubSiteId populated |
| 6 | Browse associated site | Browser | Open Finance Team site | N/A | Hub navigation appears |

---

# Phase 7 - Manage Site Owners and Admins

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open site permissions | SharePoint Admin Center | Active sites → Finance Team → Membership | N/A | Membership panel opens |
| 2 | Review owners | SharePoint Admin Center | Owners tab | N/A | Site owners displayed |
| 3 | Add secondary owner | SharePoint Admin Center | Add owner: user02@contoso.onmicrosoft.com | N/A | Secondary owner added |
| 4 | Add site collection admin with PowerShell | Admin Workstation | N/A | Set-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -LoginName user02@contoso.onmicrosoft.com -IsSiteCollectionAdmin $true | User becomes site collection admin |
| 5 | Verify site collection admins | Admin Workstation | N/A | Get-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -Limit All \| Where-Object {$_.IsSiteAdmin -eq $true} | Site admins listed |
| 6 | Document site ownership | Admin Workstation | N/A | N/A | Ownership baseline recorded |

---

# Phase 8 - Manage Site Members and Visitors

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open site membership | SharePoint Admin Center | Active sites → Finance Team → Membership | N/A | Membership panel opens |
| 2 | Review members | SharePoint Admin Center | Members tab | N/A | Members listed |
| 3 | Add member | SharePoint Admin Center | Add user02@contoso.onmicrosoft.com | N/A | Member added |
| 4 | Add security group as member | SharePoint Admin Center | Add SG-SPO-Finance-Members | N/A | Group added |
| 5 | Review visitors | SharePoint Admin Center | Visitors tab | N/A | Visitors listed |
| 6 | Add visitor if required | SharePoint Admin Center | Add read-only user or group | N/A | Visitor added |
| 7 | Validate user permissions | Browser | Open site as test user | N/A | User access matches role |

---

# Phase 9 - Configure Site Sharing Settings

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open site sharing settings | SharePoint Admin Center | Active sites → Finance Team → Policies → Sharing | N/A | Site sharing settings displayed |
| 2 | Review tenant-level limit | SharePoint Admin Center | Sharing policy page | N/A | Site cannot exceed tenant sharing limit |
| 3 | Set site sharing level | SharePoint Admin Center | Select appropriate sharing setting | N/A | Sharing level configured |
| 4 | Restrict external sharing if required | SharePoint Admin Center | Set to New and existing guests or Existing guests only | N/A | External sharing restricted |
| 5 | Save site sharing settings | SharePoint Admin Center | Save | N/A | Site sharing policy applied |
| 6 | Verify with PowerShell | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, SharingCapability | Site sharing setting displayed |

---

# Phase 10 - Configure Hub Site Permissions and Association Control

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open hub site settings | SharePoint Admin Center | Active sites → Finance Portal → Hub | N/A | Hub settings displayed |
| 2 | Review association permissions | SharePoint Admin Center | People who can associate sites | N/A | Allowed users/groups visible |
| 3 | Restrict hub association | SharePoint Admin Center | Add approved admins or group | N/A | Only approved users can associate sites |
| 4 | Review hub navigation | Browser | Open Finance Portal | N/A | Hub navigation available |
| 5 | Update hub navigation | Browser | Site settings → Navigation | N/A | Hub navigation updated |
| 6 | Verify hub association | Admin Workstation | N/A | Get-SPOHubSite | Hub configuration displayed |

---

# Phase 11 - Manage Site Lock State

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Review current lock state | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, LockState | Lock state displayed |
| 2 | Set site read-only for test | Admin Workstation | N/A | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -LockState ReadOnly | Site becomes read-only |
| 3 | Validate read-only behavior | Browser | Open Finance Team as member | N/A | Upload/edit is blocked |
| 4 | Restore site unlock state | Admin Workstation | N/A | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -LockState Unlock | Site unlocked |
| 5 | Verify restored state | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, LockState | LockState shows Unlock |

---

# Phase 12 - Export Site and Permission Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Export all SharePoint sites | Admin Workstation | N/A | Get-SPOSite -Limit All \| Select Url, Title, Owner, Template, StorageUsageCurrent, SharingCapability \| Export-Csv .\SPO-Sites-Baseline.csv -NoTypeInformation | Site baseline exported |
| 2 | Export hub sites | Admin Workstation | N/A | Get-SPOHubSite \| Export-Csv .\SPO-HubSites-Baseline.csv -NoTypeInformation | Hub baseline exported |
| 3 | Export site admins | Admin Workstation | N/A | Get-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -Limit All \| Where-Object {$_.IsSiteAdmin -eq $true} \| Export-Csv .\SPO-FinanceTeam-Admins.csv -NoTypeInformation | Site admins exported |
| 4 | Confirm files exist | Admin Workstation | N/A | Get-ChildItem .\SPO-*.csv | Exported files listed |
| 5 | Store exported baseline | Admin Workstation | N/A | N/A | Baseline preserved |

---

# Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Verify Communication Site exists | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinancePortal | Site exists |
| 2 | Verify Team Site exists | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam | Site exists |
| 3 | Verify hub registration | Admin Workstation | N/A | Get-SPOHubSite | Finance hub listed |
| 4 | Verify hub association | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select HubSiteId | HubSiteId populated |
| 5 | Verify owners | Admin Workstation | N/A | Get-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -Limit All \| Where-Object {$_.IsSiteAdmin -eq $true} | Owners/admins listed |
| 6 | Verify sharing setting | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select SharingCapability | Sharing setting displayed |
| 7 | Verify site access as member | Client Workstation | Open Finance Team URL | N/A | Member can access site |
| 8 | Verify site access as nonmember | Client Workstation | Open Finance Team URL | N/A | Nonmember denied unless sharing allows access |
| 9 | Verify export files | Admin Workstation | N/A | Test-Path .\SPO-Sites-Baseline.csv | Export file exists |

---

# Troubleshooting

| Issue | Cause | Resolution |
|------|-------|------------|
| Site creation option missing | Admin role missing or site creation disabled | Verify SharePoint Administrator role and tenant settings |
| Cannot connect to SPOService | Wrong admin URL or module issue | Use https://tenant-admin.sharepoint.com and reinstall module |
| Hub registration fails | Site already associated or unsupported site type | Use a valid communication site or unassociate first |
| Site cannot associate to hub | Association permissions restricted | Add admin or group to hub association permissions |
| User cannot access site | Missing membership or broken permission inheritance | Add user/group to Members or Owners |
| External sharing unavailable | Tenant sharing level blocks site sharing | Raise tenant sharing level before site-level change |
| Sharing setting will not save | Site setting exceeds tenant-level sharing policy | Configure tenant policy first |
| Site locked unexpectedly | LockState set to ReadOnly or NoAccess | Run Set-SPOSite -LockState Unlock |
| PowerShell command fails with access denied | Admin account lacks SharePoint role | Assign SharePoint Administrator role and reconnect |

---

# Rollback / Cleanup

| Step | Action | PowerShell |
|------|--------|------------|
| 1 | Remove test member | Remove user/group from site membership in SharePoint Admin Center |
| 2 | Remove secondary site collection admin | Set-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -LoginName user02@contoso.onmicrosoft.com -IsSiteCollectionAdmin $false |
| 3 | Unassociate Team Site from hub | Remove-SPOHubSiteAssociation -Site https://contoso.sharepoint.com/sites/FinanceTeam |
| 4 | Unregister hub site if lab-only | Unregister-SPOHubSite -Identity https://contoso.sharepoint.com/sites/FinancePortal |
| 5 | Delete lab Team Site if required | Remove-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam |
| 6 | Delete lab Communication Site if required | Remove-SPOSite -Identity https://contoso.sharepoint.com/sites/FinancePortal |
| 7 | Disconnect SharePoint Online session | Disconnect-SPOService |

---

# Related Workbooks

```text
01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md
03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md
04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews.md
05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restore.md
06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files.md
07_Troubleshoot_SharePoint_OneDrive_Sync_Sharing_Permissions_And_Restore_Issues.md
```