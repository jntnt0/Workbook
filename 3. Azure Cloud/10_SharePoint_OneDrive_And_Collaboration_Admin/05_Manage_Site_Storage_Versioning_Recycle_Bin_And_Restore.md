# 05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restores

## Objective

Manage SharePoint Online site storage, document library versioning, recycle bin recovery, deleted site restoration, and file/folder restore workflows. This workbook establishes operational recovery steps for common SharePoint and OneDrive data protection scenarios, including accidental file deletion, version rollback, site storage pressure, and site collection restore.

---

## Prerequisites

| Requirement | Details |
|------------|---------|
| Microsoft 365 Tenant | Active tenant with SharePoint Online licensed |
| Role Assignment | SharePoint Administrator or Global Administrator |
| Site-Level Role | Site Owner or Site Collection Administrator |
| Previous Workbook | 02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md |
| Related Workbook | 03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md |
| SharePoint Admin Center | Accessible from Microsoft 365 Admin Center |
| PowerShell Module | Microsoft.Online.SharePoint.PowerShell |
| Test Site | https://contoso.sharepoint.com/sites/FinanceTeam |
| Test Library | Documents |
| Test User | user01@contoso.onmicrosoft.com |

---

## Lab Topology / Assumptions

| Component | Value |
|------------|---------|
| Tenant Name | Contoso |
| SharePoint Root URL | https://contoso.sharepoint.com |
| SharePoint Admin URL | https://contoso-admin.sharepoint.com |
| Test Site | Finance Team |
| Test Site URL | https://contoso.sharepoint.com/sites/FinanceTeam |
| Document Library | Documents |
| Test File | VersionRestoreTest.docx |
| Test Folder | RestoreTestFolder |
| Storage Mode | Automatic tenant storage management unless otherwise stated |
| Site Storage Warning Level | Configured only if manual quota is used |
| Recycle Bin Scope | First-stage and second-stage recycle bins |
| Restore Scope | File, folder, library, OneDrive, and deleted site scenarios |

---

# Phase 1 - Connect Administrative Tools

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Microsoft 365 Admin Center | Admin Workstation | https://admin.microsoft.com | N/A | Microsoft 365 Admin Center opens |
| 2 | Open SharePoint Admin Center | Admin Workstation | Admin centers → SharePoint | N/A | SharePoint Admin Center opens |
| 3 | Open Active Sites | Admin Workstation | SharePoint Admin Center → Sites → Active sites | N/A | Site inventory is visible |
| 4 | Open PowerShell | Admin Workstation | Start PowerShell | N/A | PowerShell session opens |
| 5 | Install SharePoint Online module if missing | Admin Workstation | N/A | Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser | SPO module installed |
| 6 | Import SharePoint Online module | Admin Workstation | N/A | Import-Module Microsoft.Online.SharePoint.PowerShell | SPO module loaded |
| 7 | Connect to SharePoint Online | Admin Workstation | N/A | Connect-SPOService -Url https://contoso-admin.sharepoint.com | SPO admin session connected |
| 8 | Verify tenant connection | Admin Workstation | N/A | Get-SPOTenant | Tenant configuration displayed |

---

# Phase 2 - Review Tenant and Site Storage Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open SharePoint storage settings | SharePoint Admin Center | Settings → Site storage limits | N/A | Storage configuration page opens |
| 2 | Review storage management mode | SharePoint Admin Center | Site storage limits | N/A | Automatic or manual mode identified |
| 3 | Review tenant storage pool | SharePoint Admin Center | Active sites storage columns | N/A | Tenant storage usage visible |
| 4 | Review target site storage | SharePoint Admin Center | Active sites → Finance Team | N/A | Site storage usage visible |
| 5 | Review site storage with PowerShell | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, StorageUsageCurrent, StorageQuota, StorageQuotaWarningLevel | Site storage details displayed |
| 6 | Export site storage inventory | Admin Workstation | N/A | Get-SPOSite -Limit All \| Select Url, Owner, StorageUsageCurrent, StorageQuota, StorageQuotaWarningLevel \| Export-Csv .\SPO-SiteStorage-Baseline.csv -NoTypeInformation | Storage baseline exported |

---

# Phase 3 - Configure Site Storage Quota

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Determine storage model | Admin Workstation | Review business standard | N/A | Automatic or manual quota model selected |
| 2 | Open site details | SharePoint Admin Center | Active sites → Finance Team | N/A | Site details panel opens |
| 3 | Open storage settings | SharePoint Admin Center | Storage tab or Edit storage limit | N/A | Site quota settings visible |
| 4 | Set site quota if manual storage is used | SharePoint Admin Center | Enter quota and warning level | N/A | Site quota configured |
| 5 | Set site quota with PowerShell if required | Admin Workstation | N/A | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -StorageQuota 102400 -StorageQuotaWarningLevel 92160 | Site quota set to 100 GB with 90 GB warning |
| 6 | Verify quota | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, StorageUsageCurrent, StorageQuota, StorageQuotaWarningLevel | Updated quota displayed |
| 7 | Document quota exception | Admin Workstation | N/A | N/A | Storage exception recorded |

---

# Phase 4 - Configure Document Library Versioning

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open test site | Browser | https://contoso.sharepoint.com/sites/FinanceTeam | N/A | Site opens |
| 2 | Open Documents library | Browser | Documents | N/A | Library opens |
| 3 | Open library settings | Browser | Settings gear → Library settings | N/A | Library settings page opens |
| 4 | Open versioning settings | Browser | Versioning settings | N/A | Versioning settings page opens |
| 5 | Enable major versioning | Browser | Create major versions | N/A | Major versions enabled |
| 6 | Configure version limit | Browser | Keep the following number of major versions: 500 | N/A | Version history limit configured |
| 7 | Configure draft/version behavior if required | Browser | Draft item security and approval settings | N/A | Draft behavior configured |
| 8 | Save library versioning settings | Browser | OK | N/A | Versioning baseline applied |

---

# Phase 5 - Validate File Version History and Restore

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Upload test file | Browser | Documents → Upload → VersionRestoreTest.docx | N/A | Test file uploaded |
| 2 | Edit test file first time | Browser | Open file → Edit content → Save | N/A | First version created |
| 3 | Edit test file second time | Browser | Modify file again → Save | N/A | Additional version created |
| 4 | Open version history | Browser | File menu → Version history | N/A | Version history displayed |
| 5 | Review previous version | Browser | Select older version | N/A | Previous version opens |
| 6 | Restore previous version | Browser | Version history → Restore | N/A | Older version restored as current |
| 7 | Validate restored content | Browser | Open file | N/A | Restored content visible |
| 8 | Confirm version history retained | Browser | Version history | N/A | Prior versions still available |

---

# Phase 6 - Manage First-Stage Recycle Bin Restore

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Create test folder | Browser | Documents → New → Folder → RestoreTestFolder | N/A | Folder created |
| 2 | Upload test file to folder | Browser | Upload file into RestoreTestFolder | N/A | File uploaded |
| 3 | Delete test file | Browser | Select file → Delete | N/A | File deleted |
| 4 | Open site recycle bin | Browser | Site contents → Recycle bin | N/A | First-stage recycle bin opens |
| 5 | Locate deleted file | Browser | Search or browse recycle bin | N/A | Deleted file visible |
| 6 | Restore deleted file | Browser | Select file → Restore | N/A | File restored |
| 7 | Validate restored file location | Browser | Documents → RestoreTestFolder | N/A | File restored to original folder |
| 8 | Delete test folder | Browser | Select RestoreTestFolder → Delete | N/A | Folder moved to recycle bin |
| 9 | Restore deleted folder | Browser | Recycle bin → Select folder → Restore | N/A | Folder and contents restored |

---

# Phase 7 - Manage Second-Stage Recycle Bin Restore

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Delete test file again | Browser | Documents → Delete VersionRestoreTest.docx | N/A | File enters first-stage recycle bin |
| 2 | Open first-stage recycle bin | Browser | Site contents → Recycle bin | N/A | File visible |
| 3 | Delete from first-stage recycle bin | Browser | Select file → Delete | N/A | File moved to second-stage recycle bin |
| 4 | Open second-stage recycle bin | Browser | Recycle bin → Second-stage recycle bin | N/A | Second-stage recycle bin opens |
| 5 | Locate deleted file | Browser | Search or browse | N/A | File visible in second-stage recycle bin |
| 6 | Restore file from second-stage recycle bin | Browser | Select file → Restore | N/A | File restored |
| 7 | Validate restored file | Browser | Documents library | N/A | File appears in original location |
| 8 | Document restore result | Admin Workstation | N/A | N/A | Restore evidence recorded |

---

# Phase 8 - Restore Deleted SharePoint Site

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Create lab restore site if needed | SharePoint Admin Center | Active sites → Create | N/A | Lab site created |
| 2 | Delete lab restore site | SharePoint Admin Center | Active sites → Select site → Delete | N/A | Site moved to deleted sites |
| 3 | Open deleted sites | SharePoint Admin Center | Sites → Deleted sites | N/A | Deleted sites list opens |
| 4 | Select deleted lab site | SharePoint Admin Center | Select site | N/A | Deleted site selected |
| 5 | Restore deleted site | SharePoint Admin Center | Restore | N/A | Site restore starts |
| 6 | Verify site returns to active sites | SharePoint Admin Center | Sites → Active sites | N/A | Site appears as active |
| 7 | Restore deleted site with PowerShell if required | Admin Workstation | N/A | Restore-SPODeletedSite -Identity https://contoso.sharepoint.com/sites/LabRestoreSite | Deleted site restored |
| 8 | Verify restored site with PowerShell | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/LabRestoreSite | Restored site displayed |

---

# Phase 9 - Restore OneDrive Files and Folders

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Sign in as test user | Browser | Open Microsoft 365 as user01 | N/A | Test user signed in |
| 2 | Open OneDrive | Browser | App launcher → OneDrive | N/A | OneDrive opens |
| 3 | Upload OneDrive test file | Browser | Upload OneDriveRestoreTest.docx | N/A | File uploaded |
| 4 | Delete OneDrive test file | Browser | Select file → Delete | N/A | File deleted |
| 5 | Open OneDrive recycle bin | Browser | OneDrive → Recycle bin | N/A | Deleted file visible |
| 6 | Restore OneDrive file | Browser | Select file → Restore | N/A | File restored |
| 7 | Validate restored OneDrive file | Browser | My files | N/A | File visible again |
| 8 | Open Restore your OneDrive if broad rollback is needed | Browser | OneDrive settings → Restore your OneDrive | N/A | Point-in-time OneDrive restore page opens |
| 9 | Choose restore point if required | Browser | Select date/time or activity slider | N/A | OneDrive restore scope selected |
| 10 | Start OneDrive restore only for lab/test data | Browser | Restore | N/A | OneDrive reverted to selected point |

---

# Phase 10 - Restore Files from Library Activity or Restore This Library

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Documents library | Browser | Finance Team → Documents | N/A | Library opens |
| 2 | Open library settings menu | Browser | Settings gear or library menu | N/A | Library options displayed |
| 3 | Open Restore this library if available | Browser | Restore this library | N/A | Library restore page opens |
| 4 | Review file activity | Browser | Activity timeline | N/A | Recent file activity visible |
| 5 | Select restore point | Browser | Choose date/time before test deletion/change | N/A | Restore point selected |
| 6 | Review affected changes | Browser | Inspect restore preview | N/A | Restore impact understood |
| 7 | Run restore only if lab-safe | Browser | Restore | N/A | Library restored to selected point |
| 8 | Validate restored library state | Browser | Documents library | N/A | Deleted or changed content recovered |

---

# Phase 11 - Review Retention and Preservation Considerations

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Microsoft Purview portal | Admin Workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Review retention policies | Purview Portal | Data lifecycle management → Microsoft 365 → Retention policies | N/A | Existing retention policies listed |
| 3 | Identify SharePoint retention policies | Purview Portal | Review policy locations | N/A | SharePoint sites policy coverage identified |
| 4 | Identify OneDrive retention policies | Purview Portal | Review policy locations | N/A | OneDrive policy coverage identified |
| 5 | Review preservation behavior | Purview Portal | Policy details | N/A | Retention impact understood |
| 6 | Document restore impact | Admin Workstation | N/A | N/A | Admin notes updated |

---

# Phase 12 - Export Recovery and Storage Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Export site storage baseline | Admin Workstation | N/A | Get-SPOSite -Limit All \| Select Url, Owner, Template, StorageUsageCurrent, StorageQuota, StorageQuotaWarningLevel, LockState \| Export-Csv .\SPO-SiteStorage-After.csv -NoTypeInformation | Storage baseline exported |
| 2 | Export deleted sites | Admin Workstation | N/A | Get-SPODeletedSite \| Select Url, DeletionTime, DaysRemaining, Status \| Export-Csv .\SPO-DeletedSites.csv -NoTypeInformation | Deleted sites exported |
| 3 | Export target site details | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Format-List * \| Out-File .\SPO-FinanceTeam-SiteDetails.txt | Site details exported |
| 4 | Confirm export files | Admin Workstation | N/A | Get-ChildItem .\SPO-* | Export files listed |
| 5 | Store baseline evidence | Admin Workstation | N/A | N/A | Recovery evidence preserved |

---

# Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Verify SPO connection | Admin Workstation | N/A | Get-SPOTenant | Tenant output returned |
| 2 | Verify site storage values | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select StorageUsageCurrent, StorageQuota, StorageQuotaWarningLevel | Storage values displayed |
| 3 | Verify version history exists | Browser | Documents → File → Version history | N/A | Multiple versions visible |
| 4 | Verify version restore | Browser | Open restored file | N/A | Restored content visible |
| 5 | Verify first-stage recycle bin restore | Browser | Documents library | N/A | Deleted item restored |
| 6 | Verify second-stage recycle bin restore | Browser | Documents library | N/A | Second-stage item restored |
| 7 | Verify deleted site restore | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/LabRestoreSite | Restored site returned |
| 8 | Verify OneDrive file restore | Browser | OneDrive → My files | N/A | Restored file visible |
| 9 | Verify deleted sites report | Admin Workstation | N/A | Get-SPODeletedSite | Deleted site list returned |
| 10 | Verify export files | Admin Workstation | N/A | Test-Path .\SPO-SiteStorage-After.csv | Export file exists |

---

# Troubleshooting

| Issue | Cause | Resolution |
|------|-------|------------|
| Site quota settings unavailable | Tenant uses automatic storage management | Switch to manual storage only if business requires it |
| Set-SPOSite quota fails | Invalid quota value or insufficient role | Use MB values and verify SharePoint Administrator role |
| Version history missing | Library versioning disabled before file edits | Enable versioning and create multiple versions |
| Restore option unavailable | User lacks permission or file is not in recycle bin | Use Site Owner or Site Collection Admin account |
| File not in first-stage recycle bin | Item was deleted from first-stage bin | Check second-stage recycle bin |
| File not in second-stage recycle bin | Retention period expired or item permanently deleted | Check retention policy, backups, or support options |
| Deleted site not visible | Site deletion not completed or retention expired | Wait for deletion processing or verify URL |
| Restore-SPODeletedSite fails | Site URL conflict or site not in deleted state | Remove conflicting active site or verify deleted site URL |
| OneDrive restore unavailable | User lacks access or OneDrive not provisioned | Confirm license and OneDrive provisioning |
| Library restore could affect many files | Restore operation is broad | Use version history or recycle bin for single-item recovery when possible |
| Storage usage does not update immediately | Reporting delay | Wait and recheck later with Get-SPOSite |

---

# Rollback / Cleanup

| Step | Action | PowerShell |
|------|--------|------------|
| 1 | Restore original site quota | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -StorageQuota <PreviousQuotaMB> -StorageQuotaWarningLevel <PreviousWarningMB> |
| 2 | Remove lab test files | Delete VersionRestoreTest.docx and OneDriveRestoreTest.docx |
| 3 | Remove lab test folder | Delete RestoreTestFolder |
| 4 | Delete lab restore site if created | Remove-SPOSite -Identity https://contoso.sharepoint.com/sites/LabRestoreSite |
| 5 | Permanently delete lab restore site if required | Remove-SPODeletedSite -Identity https://contoso.sharepoint.com/sites/LabRestoreSite |
| 6 | Remove exported baseline files if required | Remove-Item .\SPO-SiteStorage-*.csv, .\SPO-DeletedSites.csv, .\SPO-FinanceTeam-SiteDetails.txt |
| 7 | Disconnect SharePoint Online session | Disconnect-SPOService |

---

# Related Workbooks

```text
01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md
02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md
03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md
04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews.md
06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files.md
07_Troubleshoot_SharePoint_OneDrive_Sync_Sharing_Permissions_And_Restore_Issues.md
```