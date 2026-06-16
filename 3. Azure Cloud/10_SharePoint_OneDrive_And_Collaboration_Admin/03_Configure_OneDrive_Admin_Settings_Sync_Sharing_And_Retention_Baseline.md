# 03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline

## Objective

Configure a Microsoft 365 OneDrive for Business administration baseline covering OneDrive tenant settings, sync behavior, sharing controls, storage limits, retention posture, and validation. This workbook focuses on tenant-level OneDrive administration using the SharePoint Admin Center, Microsoft 365 Admin Center, SharePoint Online PowerShell, and Purview where retention settings are required.

---

## Prerequisites

| Requirement | Details |
|------------|---------|
| Microsoft 365 Tenant | Active tenant with OneDrive for Business licensed |
| Role Assignment | SharePoint Administrator or Global Administrator |
| Optional Compliance Role | Compliance Administrator for retention validation |
| Previous Workbook | 01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md |
| Related Workbook | 02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md |
| SharePoint Admin Center | Accessible from Microsoft 365 Admin Center |
| PowerShell Module | Microsoft.Online.SharePoint.PowerShell |
| Optional Module | ExchangeOnlineManagement for Purview PowerShell |
| Admin URL | https://contoso-admin.sharepoint.com |
| Test User | user01@contoso.onmicrosoft.com |
| Client Device | Windows 10/11 device with OneDrive sync client |

---

## Lab Topology / Assumptions

| Component | Value |
|------------|---------|
| Tenant Name | Contoso |
| SharePoint Root URL | https://contoso.sharepoint.com |
| SharePoint Admin URL | https://contoso-admin.sharepoint.com |
| OneDrive URL Pattern | https://contoso-my.sharepoint.com/personal/user01_contoso_onmicrosoft_com |
| Admin Account | spadmin@contoso.onmicrosoft.com |
| Test User | user01@contoso.onmicrosoft.com |
| Test Client | WIN11-CLIENT01 |
| OneDrive Sync Client | Installed and current |
| Baseline Sharing Level | New and existing guests, or more restrictive |
| Baseline Storage Mode | Tenant default unless business requirement says otherwise |

---

# Phase 1 - Connect to SharePoint and OneDrive Administration

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Microsoft 365 Admin Center | Admin Workstation | https://admin.microsoft.com | N/A | Microsoft 365 Admin Center opens |
| 2 | Open SharePoint Admin Center | Admin Workstation | Admin centers → SharePoint | N/A | SharePoint Admin Center opens |
| 3 | Open OneDrive settings area | Admin Workstation | SharePoint Admin Center → Settings | N/A | OneDrive-related tenant settings available |
| 4 | Open PowerShell | Admin Workstation | Start PowerShell | N/A | PowerShell session opens |
| 5 | Install SharePoint Online module if missing | Admin Workstation | N/A | Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser | SPO module installed |
| 6 | Import SharePoint Online module | Admin Workstation | N/A | Import-Module Microsoft.Online.SharePoint.PowerShell | SPO module loaded |
| 7 | Connect to SharePoint Online | Admin Workstation | N/A | Connect-SPOService -Url https://contoso-admin.sharepoint.com | Admin connected |
| 8 | Verify connection | Admin Workstation | N/A | Get-SPOTenant | Tenant settings displayed |

---

# Phase 2 - Review Current OneDrive Tenant Settings

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Review tenant settings | Admin Workstation | N/A | Get-SPOTenant | Full tenant configuration displayed |
| 2 | Review OneDrive storage quota | Admin Workstation | N/A | Get-SPOTenant \| Select OneDriveStorageQuota | OneDrive quota displayed |
| 3 | Review deleted user retention | Admin Workstation | N/A | Get-SPOTenant \| Select OrphanedPersonalSitesRetentionPeriod | Retention period displayed |
| 4 | Review sharing capability | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability | Tenant sharing setting displayed |
| 5 | Review sync-related tenant settings | Admin Workstation | N/A | Get-SPOTenant \| Select *Sync* | Sync-related settings displayed |
| 6 | Export current baseline | Admin Workstation | N/A | Get-SPOTenant \| Out-File .\OneDrive-Tenant-Baseline-Before.txt | Baseline exported |

---

# Phase 3 - Configure Default OneDrive Storage Quota

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open OneDrive storage settings | SharePoint Admin Center | Settings → OneDrive storage limit | N/A | Storage limit settings displayed |
| 2 | Review current default quota | SharePoint Admin Center | OneDrive storage limit | N/A | Current default storage visible |
| 3 | Set default OneDrive storage quota | SharePoint Admin Center | Enter tenant standard quota | N/A | Default quota configured |
| 4 | Apply default quota with PowerShell if needed | Admin Workstation | N/A | Set-SPOTenant -OneDriveStorageQuota 1048576 | Default quota set to 1 TB |
| 5 | Verify quota setting | Admin Workstation | N/A | Get-SPOTenant \| Select OneDriveStorageQuota | Updated quota displayed |
| 6 | Document business exception process | Admin Workstation | N/A | N/A | Exception handling recorded |

---

# Phase 4 - Configure OneDrive Sync Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Sync settings | SharePoint Admin Center | Settings → Sync | N/A | Sync settings displayed |
| 2 | Confirm OneDrive sync is allowed | SharePoint Admin Center | Sync settings | N/A | Users can sync SharePoint and OneDrive files |
| 3 | Review unmanaged device posture | SharePoint Admin Center | Policies → Access control | N/A | Device access settings reviewed |
| 4 | Review domain restriction option | SharePoint Admin Center | Settings → Sync | N/A | Domain restriction setting visible |
| 5 | Restrict sync to domain-joined devices if required | SharePoint Admin Center | Enable sync restriction and enter domain GUIDs | N/A | Sync limited to approved domains |
| 6 | Leave unrestricted for basic lab baseline | SharePoint Admin Center | Do not configure domain restriction | N/A | Test client sync remains functional |
| 7 | Save sync settings | SharePoint Admin Center | Save | N/A | Sync policy saved |
| 8 | Export sync-related tenant values | Admin Workstation | N/A | Get-SPOTenant \| Select *Sync* \| Format-List | Sync baseline visible |

---

# Phase 5 - Validate OneDrive Sync Client Behavior

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Sign in to Windows client | WIN11-CLIENT01 | Sign in as user01@contoso.onmicrosoft.com | N/A | Test user signed in |
| 2 | Launch OneDrive client | WIN11-CLIENT01 | Start OneDrive | N/A | OneDrive setup opens |
| 3 | Sign in to OneDrive | WIN11-CLIENT01 | Enter user01 credentials | N/A | OneDrive signs in |
| 4 | Confirm folder location | WIN11-CLIENT01 | Accept default OneDrive path | N/A | Local OneDrive folder created |
| 5 | Create test file | WIN11-CLIENT01 | Create OneDriveSyncTest.txt | N/A | File appears locally |
| 6 | Verify cloud sync | Browser | Open OneDrive web portal | N/A | Test file appears online |
| 7 | Modify test file online | Browser | Edit or upload version | N/A | Change syncs to client |
| 8 | Confirm client sync health | WIN11-CLIENT01 | OneDrive tray icon | N/A | Sync client shows up to date |

---

# Phase 6 - Configure OneDrive Sharing Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Sharing policies | SharePoint Admin Center | Policies → Sharing | N/A | Tenant sharing policy displayed |
| 2 | Review SharePoint and OneDrive sharing sliders | SharePoint Admin Center | Sharing page | N/A | Current external sharing posture visible |
| 3 | Set OneDrive sharing level | SharePoint Admin Center | Choose New and existing guests, Existing guests only, or Only people in your organization | N/A | OneDrive sharing level configured |
| 4 | Confirm OneDrive does not exceed SharePoint sharing level | SharePoint Admin Center | Compare sliders | N/A | OneDrive sharing is equal to or more restrictive than SharePoint |
| 5 | Configure default link type | SharePoint Admin Center | File and folder links → Default link type | N/A | Default link type configured |
| 6 | Configure default link permission | SharePoint Admin Center | Default link permission | N/A | View or edit baseline selected |
| 7 | Configure link expiration if allowed | SharePoint Admin Center | Advanced settings for Anyone links | N/A | Anonymous link expiration configured if Anyone links are allowed |
| 8 | Save sharing settings | SharePoint Admin Center | Save | N/A | Sharing baseline saved |
| 9 | Verify sharing capability with PowerShell | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability, DefaultSharingLinkType, DefaultLinkPermission | Sharing values displayed |

---

# Phase 7 - Configure OneDrive Deleted User Retention Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Review deleted user OneDrive retention | Admin Workstation | N/A | Get-SPOTenant \| Select OrphanedPersonalSitesRetentionPeriod | Current value displayed |
| 2 | Set deleted user OneDrive retention | Admin Workstation | N/A | Set-SPOTenant -OrphanedPersonalSitesRetentionPeriod 365 | Deleted user's OneDrive retained for 365 days |
| 3 | Verify retention value | Admin Workstation | N/A | Get-SPOTenant \| Select OrphanedPersonalSitesRetentionPeriod | Updated value displayed |
| 4 | Document recovery owner process | Admin Workstation | N/A | N/A | Recovery process documented |
| 5 | Review Microsoft 365 user deletion workflow | Microsoft 365 Admin Center | Users → Deleted users | N/A | Deleted user restore location known |
| 6 | Confirm escalation path for deleted OneDrive recovery | Admin Workstation | N/A | N/A | Admin procedure documented |

---

# Phase 8 - Review OneDrive Retention Policy Coverage in Purview

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Microsoft Purview portal | Admin Workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Open retention policies | Admin Workstation | Data lifecycle management → Microsoft 365 → Retention policies | N/A | Retention policy list displayed |
| 3 | Review existing OneDrive retention policies | Admin Workstation | Filter or inspect policy locations | N/A | Policies covering OneDrive identified |
| 4 | Create baseline retention policy if required | Admin Workstation | Create retention policy | N/A | Policy wizard opens |
| 5 | Select OneDrive accounts location | Admin Workstation | Locations → OneDrive accounts | N/A | OneDrive selected |
| 6 | Configure retain/delete behavior | Admin Workstation | Set business retention period | N/A | Retention behavior configured |
| 7 | Complete policy creation | Admin Workstation | Submit | N/A | Retention policy created |
| 8 | Document policy name and scope | Admin Workstation | N/A | N/A | Retention baseline recorded |

---

# Phase 9 - Optional Purview PowerShell Validation

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Install Exchange Online module if needed | Admin Workstation | N/A | Install-Module ExchangeOnlineManagement -Scope CurrentUser | Module installed |
| 2 | Import module | Admin Workstation | N/A | Import-Module ExchangeOnlineManagement | Module loaded |
| 3 | Connect to Purview PowerShell | Admin Workstation | N/A | Connect-IPPSSession | Compliance PowerShell connected |
| 4 | Review retention compliance policies | Admin Workstation | N/A | Get-RetentionCompliancePolicy | Retention policies listed |
| 5 | Review retention rules | Admin Workstation | N/A | Get-RetentionComplianceRule | Retention rules listed |
| 6 | Confirm OneDrive retention coverage | Admin Workstation | N/A | Get-RetentionCompliancePolicy \| Format-Table Name, Workload, Enabled | OneDrive-related policy visible |
| 7 | Disconnect Exchange Online session | Admin Workstation | N/A | Disconnect-ExchangeOnline -Confirm:$false | Session disconnected |

---

# Phase 10 - Configure OneDrive Access Control Review Points

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open access control settings | SharePoint Admin Center | Policies → Access control | N/A | Access control page opens |
| 2 | Review unmanaged device policy | SharePoint Admin Center | Unmanaged devices | N/A | Current policy visible |
| 3 | Review idle session sign-out | SharePoint Admin Center | Idle session sign-out | N/A | Session policy visible |
| 4 | Review network location policy | SharePoint Admin Center | Network location | N/A | Location policy visible |
| 5 | Confirm Conditional Access dependency | Entra Admin Center | Protection → Conditional Access | N/A | CA integration understood |
| 6 | Document chosen access control baseline | Admin Workstation | N/A | N/A | Access posture recorded |

---

# Phase 11 - Export OneDrive Administration Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Export tenant configuration | Admin Workstation | N/A | Get-SPOTenant \| Out-File .\OneDrive-Tenant-Baseline-After.txt | Tenant baseline exported |
| 2 | Export OneDrive quota and retention values | Admin Workstation | N/A | Get-SPOTenant \| Select OneDriveStorageQuota, OrphanedPersonalSitesRetentionPeriod, SharingCapability \| Export-Csv .\OneDrive-Core-Baseline.csv -NoTypeInformation | Core settings exported |
| 3 | Export personal sites if provisioned | Admin Workstation | N/A | Get-SPOSite -IncludePersonalSite $true -Limit All -Template SPSPERS \| Select Url, Owner, StorageUsageCurrent, LockState \| Export-Csv .\OneDrive-PersonalSites.csv -NoTypeInformation | Personal sites exported |
| 4 | Confirm export files exist | Admin Workstation | N/A | Get-ChildItem .\OneDrive-* | Export files listed |
| 5 | Store baseline artifacts | Admin Workstation | N/A | N/A | Baseline preserved |

---

# Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Verify SPO connection | Admin Workstation | N/A | Get-SPOTenant | Tenant output returned |
| 2 | Verify OneDrive storage quota | Admin Workstation | N/A | Get-SPOTenant \| Select OneDriveStorageQuota | Expected quota shown |
| 3 | Verify deleted user retention | Admin Workstation | N/A | Get-SPOTenant \| Select OrphanedPersonalSitesRetentionPeriod | Expected retention shown |
| 4 | Verify tenant sharing posture | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability | Expected sharing posture shown |
| 5 | Verify OneDrive web access | Browser | Open https://www.microsoft365.com/launch/onedrive | N/A | User OneDrive opens |
| 6 | Verify sync client sign-in | WIN11-CLIENT01 | Open OneDrive client | N/A | Client signed in |
| 7 | Verify local-to-cloud sync | WIN11-CLIENT01 | Create test file | N/A | File appears online |
| 8 | Verify cloud-to-local sync | Browser/WIN11-CLIENT01 | Modify file online | N/A | Change appears locally |
| 9 | Verify retention policy presence | Purview Portal | Data lifecycle management → Retention policies | N/A | OneDrive policy visible |
| 10 | Verify export files | Admin Workstation | N/A | Test-Path .\OneDrive-Core-Baseline.csv | Export file exists |

---

# Troubleshooting

| Issue | Cause | Resolution |
|------|-------|------------|
| OneDrive settings missing | Admin lacks role or SharePoint Admin Center not fully loaded | Assign SharePoint Administrator role and reload portal |
| Connect-SPOService fails | Incorrect tenant admin URL | Use https://tenant-admin.sharepoint.com |
| OneDrive sync client will not sign in | License, identity, or client issue | Verify user license, password, MFA, and client version |
| Files do not sync | Sync paused, blocked file type, path issue, or policy restriction | Resume sync, shorten path, check file name, review sync restrictions |
| User cannot share OneDrive file externally | Tenant or OneDrive sharing level blocks external sharing | Review SharePoint Admin Center → Policies → Sharing |
| OneDrive sharing slider cannot be set higher | OneDrive cannot be less restrictive than SharePoint tenant sharing | Raise SharePoint sharing first or keep OneDrive more restrictive |
| Deleted user OneDrive unavailable | Retention period expired or personal site not provisioned | Restore user if possible or recover from backup/retention if configured |
| Retention policy not visible | Missing compliance permissions | Assign Compliance Administrator or Data Lifecycle Management role |
| Personal sites export returns no users | Users have not opened OneDrive yet | Have users access OneDrive to provision personal sites |
| Sync restriction blocks test device | Domain restriction configured incorrectly | Remove or correct allowed domain GUIDs |

---

# Rollback / Cleanup

| Step | Action | PowerShell |
|------|--------|------------|
| 1 | Restore previous OneDrive storage quota | Set-SPOTenant -OneDriveStorageQuota <PreviousQuotaInMB> |
| 2 | Restore previous deleted user retention period | Set-SPOTenant -OrphanedPersonalSitesRetentionPeriod <PreviousDays> |
| 3 | Restore previous sharing settings | Use SharePoint Admin Center → Policies → Sharing |
| 4 | Remove lab retention policy if created only for testing | Remove from Purview portal after confirming it is not production scoped |
| 5 | Remove OneDrive sync test file | Delete OneDriveSyncTest.txt from client or web |
| 6 | Remove exported baseline files if required | Remove-Item .\OneDrive-* |
| 7 | Disconnect SharePoint Online session | Disconnect-SPOService |
| 8 | Disconnect Purview session if connected | Disconnect-ExchangeOnline -Confirm:$false |

---

# Related Workbooks

```text
01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md
02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md
04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews.md
05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restore.md
06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files.md
07_Troubleshoot_SharePoint_OneDrive_Sync_Sharing_Permissions_And_Restore_Issues.md
```