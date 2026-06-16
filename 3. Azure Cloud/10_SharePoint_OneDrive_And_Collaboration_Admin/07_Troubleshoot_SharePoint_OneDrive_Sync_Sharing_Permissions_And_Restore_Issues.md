# 07_Troubleshoot_SharePoint_OneDrive_Sync_Sharing_Permissions_And_Restore_Issues

## Objective

Troubleshoot common SharePoint Online and OneDrive issues involving sync failures, external sharing failures, broken permissions, access denied errors, deleted content recovery, version restore problems, site restore failures, and policy-related blocks. This workbook provides a structured operations workflow for isolating whether the issue is caused by identity, licensing, SharePoint site configuration, OneDrive sync client health, tenant policy, DLP/sensitivity policy, retention, or user error.

---

## Prerequisites

| Requirement | Details |
|------------|---------|
| Microsoft 365 Tenant | Active tenant with SharePoint Online and OneDrive licensed |
| Role Assignment | SharePoint Administrator or Global Administrator |
| Optional Role | Compliance Administrator for Purview, DLP, retention, and audit checks |
| Optional Role | Security Administrator or Identity Governance Administrator for guest/access review checks |
| Previous Workbook | 06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files.md |
| SharePoint Admin Center | Accessible from Microsoft 365 Admin Center |
| Microsoft Purview Portal | Accessible for DLP, audit, and retention checks |
| Entra Admin Center | Accessible for guest identity and Conditional Access checks |
| PowerShell Module | Microsoft.Online.SharePoint.PowerShell |
| Optional Modules | Microsoft.Graph, ExchangeOnlineManagement |
| Test Site | https://contoso.sharepoint.com/sites/FinanceTeam |
| Test User | user01@contoso.onmicrosoft.com |
| Test External User | guest.user@externaldomain.com |
| Test Client | WIN11-CLIENT01 |

---

## Lab Topology / Assumptions

| Component | Value |
|------------|---------|
| Tenant Name | Contoso |
| SharePoint Root URL | https://contoso.sharepoint.com |
| SharePoint Admin URL | https://contoso-admin.sharepoint.com |
| Test Site | Finance Team |
| Test Site URL | https://contoso.sharepoint.com/sites/FinanceTeam |
| Test Library | Documents |
| OneDrive URL Pattern | https://contoso-my.sharepoint.com/personal/user01_contoso_onmicrosoft_com |
| Internal Test User | user01@contoso.onmicrosoft.com |
| External Test User | guest.user@externaldomain.com |
| Test File | TroubleshootTest.docx |
| OneDrive Sync Client | Installed on WIN11-CLIENT01 |
| Baseline Sharing | New and existing guests or more restrictive |
| Baseline Link Type | Specific people |
| Baseline Restore Scope | File, folder, library, OneDrive, deleted site |

---

# Phase 1 - Collect the Trouble Ticket Facts

## Triage Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Identify affected workload | Admin Workstation | Ask whether issue is SharePoint site, OneDrive, Teams-backed files, or external sharing | N/A | Workload identified |
| 2 | Identify affected user | Admin Workstation | Record UPN and role | N/A | User identity confirmed |
| 3 | Identify affected URL | Admin Workstation | Capture exact file, folder, library, site, or OneDrive URL | N/A | Exact target recorded |
| 4 | Identify error message | Admin Workstation | Capture screenshot or exact wording | N/A | Error preserved |
| 5 | Identify scope | Admin Workstation | Single user, multiple users, single site, all sites, internal only, external only | N/A | Blast radius known |
| 6 | Identify time of failure | Admin Workstation | Record start time and recent changes | N/A | Timeline established |
| 7 | Identify recent admin changes | Admin Workstation | Check if sharing, DLP, labels, CA, retention, or site permissions changed | N/A | Change correlation identified |
| 8 | Record expected behavior | Admin Workstation | Ask what should happen | N/A | Success condition defined |

---

# Phase 2 - Connect Administrative Tools

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open SharePoint Admin Center | Admin Workstation | Microsoft 365 Admin Center → Admin centers → SharePoint | N/A | SharePoint Admin Center opens |
| 2 | Open Entra Admin Center | Admin Workstation | https://entra.microsoft.com | N/A | Entra Admin Center opens |
| 3 | Open Microsoft Purview portal | Admin Workstation | https://purview.microsoft.com | N/A | Purview opens |
| 4 | Open PowerShell | Admin Workstation | Start PowerShell | N/A | PowerShell opens |
| 5 | Install SPO module if missing | Admin Workstation | N/A | Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser | SPO module installed |
| 6 | Import SPO module | Admin Workstation | N/A | Import-Module Microsoft.Online.SharePoint.PowerShell | Module loaded |
| 7 | Connect to SPO service | Admin Workstation | N/A | Connect-SPOService -Url https://contoso-admin.sharepoint.com | SPO connection established |
| 8 | Verify tenant access | Admin Workstation | N/A | Get-SPOTenant | Tenant settings returned |

---

# Phase 3 - Verify User Licensing, Identity, and Account State

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open affected user in Microsoft 365 Admin Center | Admin Workstation | Users → Active users → user01 | N/A | User profile opens |
| 2 | Verify license assignment | Admin Workstation | Licenses and apps | N/A | SharePoint/OneDrive service plan assigned |
| 3 | Verify sign-in allowed | Admin Workstation | Account tab | N/A | Sign-in is not blocked |
| 4 | Verify user type | Entra Admin Center | Identity → Users → user01 | N/A | Member or Guest status confirmed |
| 5 | Verify group membership | Entra Admin Center | User → Groups | N/A | Required group membership present |
| 6 | Connect Graph if needed | Admin Workstation | N/A | Connect-MgGraph -Scopes "User.Read.All","Group.Read.All","Directory.Read.All" | Graph connected |
| 7 | Check user object with Graph | Admin Workstation | N/A | Get-MgUser -UserId user01@contoso.onmicrosoft.com \| Select DisplayName, UserPrincipalName, AccountEnabled, UserType | User object healthy |
| 8 | Export user group membership if needed | Admin Workstation | N/A | Get-MgUserMemberOf -UserId user01@contoso.onmicrosoft.com | Membership returned |

---

# Phase 4 - Troubleshoot SharePoint Site Access Denied

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Confirm site exists | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam | Site details returned |
| 2 | Check site lock state | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, LockState | LockState reviewed |
| 3 | Unlock site if incorrectly locked | Admin Workstation | N/A | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -LockState Unlock | Site unlocked |
| 4 | Open site permissions | Browser | Site → Settings → Site permissions | N/A | Permissions page opens |
| 5 | Check user permissions | Browser | Site permissions → Check permissions | N/A | User effective permissions displayed |
| 6 | Check site admins | Admin Workstation | N/A | Get-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -Limit All \| Where-Object {$_.IsSiteAdmin -eq $true} | Site admins listed |
| 7 | Add temporary site collection admin for troubleshooting | Admin Workstation | N/A | Set-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -LoginName spadmin@contoso.onmicrosoft.com -IsSiteCollectionAdmin $true | Admin can inspect site |
| 8 | Validate access as affected user | Client Workstation | Open site URL in browser | N/A | User access succeeds or fails with known cause |

---

# Phase 5 - Troubleshoot Broken Permissions on File, Folder, or Library

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open affected library | Browser | Finance Team → Documents | N/A | Library opens |
| 2 | Locate affected item | Browser | Browse to file or folder | N/A | Item located |
| 3 | Open Manage access | Browser | File/folder → Details → Manage access | N/A | Direct links and permissions visible |
| 4 | Identify unique permissions | Browser | Advanced permission settings if needed | N/A | Inheritance status identified |
| 5 | Check if user has direct access | Browser | Manage access → Direct access | N/A | User or group permission visible |
| 6 | Check sharing links | Browser | Manage access → Links | N/A | Existing sharing links visible |
| 7 | Remove stale or risky links | Browser | Manage access → Remove link | N/A | Bad link removed |
| 8 | Regrant access through group | Browser | Add user to proper SharePoint group or M365 group | N/A | Access restored through standard group |
| 9 | Validate affected item access | Client Workstation | Open exact file/folder URL | N/A | Access succeeds |

---

# Phase 6 - Troubleshoot External Sharing Failures

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Verify tenant sharing setting | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability, DefaultSharingLinkType, DefaultLinkPermission | Tenant sharing posture displayed |
| 2 | Verify site sharing setting | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Select Url, SharingCapability | Site sharing posture displayed |
| 3 | Confirm site does not exceed tenant policy | Admin Workstation | Compare tenant and site values | N/A | Site policy is equal or more restrictive |
| 4 | Check domain restrictions | Admin Workstation | N/A | Get-SPOTenant \| Select SharingDomainRestrictionMode, SharingAllowedDomainList, SharingBlockedDomainList | Domain restrictions displayed |
| 5 | Verify external guest account | Entra Admin Center | Users → All users → Filter Guest | N/A | Guest object exists if invitation redeemed |
| 6 | Check guest account enabled state | Entra Admin Center | Guest user profile | N/A | Guest account enabled |
| 7 | Reissue sharing invitation | Browser | File → Share → Specific people → guest address | N/A | New invitation sent |
| 8 | Test with private browser session | External Browser Session | Open invitation link | N/A | Guest prompt appears |
| 9 | Confirm exact identity match | External Browser Session | Sign in with invited guest address | N/A | Guest identity matches invitation |
| 10 | Validate result | External Browser Session | Open shared file | N/A | Guest can access or policy block is confirmed |

---

# Phase 7 - Troubleshoot Link Settings and Anonymous Link Problems

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Check default link type | Admin Workstation | N/A | Get-SPOTenant \| Select DefaultSharingLinkType, DefaultLinkPermission | Link defaults displayed |
| 2 | Check anonymous link expiration | Admin Workstation | N/A | Get-SPOTenant \| Select RequireAnonymousLinksExpireInDays | Expiration setting displayed |
| 3 | Confirm Anyone links are allowed only if approved | SharePoint Admin Center | Policies → Sharing | N/A | Anyone links enabled or disabled |
| 4 | Open Manage access on affected file | Browser | File → Manage access | N/A | Links visible |
| 5 | Check whether link expired | Browser | Manage access → Links | N/A | Expired link identified |
| 6 | Create new Specific people link | Browser | Share → Specific people | N/A | Controlled link created |
| 7 | Avoid broad Anyone link unless business-approved | Browser | Do not choose Anyone | N/A | Anonymous exposure avoided |
| 8 | Validate link access | Browser or External Session | Open new link | N/A | Link works for intended recipient only |

---

# Phase 8 - Troubleshoot OneDrive Sync Client Issues

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Check OneDrive tray status | WIN11-CLIENT01 | Select OneDrive cloud icon | N/A | Sync status visible |
| 2 | Confirm user is signed in | WIN11-CLIENT01 | OneDrive settings → Account | N/A | Correct account connected |
| 3 | Confirm sync is not paused | WIN11-CLIENT01 | OneDrive menu | N/A | Sync active |
| 4 | Confirm storage is available | WIN11-CLIENT01 | File Explorer drive properties | N/A | Local disk has space |
| 5 | Confirm file path length and names | WIN11-CLIENT01 | Review affected path | N/A | No unsupported names or extreme paths |
| 6 | Check file locks | WIN11-CLIENT01 | Close Office apps using file | N/A | Locked file released |
| 7 | Check OneDrive client version | WIN11-CLIENT01 | OneDrive settings → About | N/A | Client version visible |
| 8 | Restart OneDrive client | WIN11-CLIENT01 | Run dialog | %localappdata%\Microsoft\OneDrive\OneDrive.exe /shutdown | OneDrive exits |
| 9 | Start OneDrive client | WIN11-CLIENT01 | Run dialog | %localappdata%\Microsoft\OneDrive\OneDrive.exe | OneDrive restarts |
| 10 | Validate sync | WIN11-CLIENT01 | Create small test file | N/A | File syncs successfully |

---

# Phase 9 - Reset OneDrive Sync Client

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Confirm reset is needed | WIN11-CLIENT01 | Verify normal restart did not fix issue | N/A | Reset justified |
| 2 | Run OneDrive reset command | WIN11-CLIENT01 | Run dialog | %localappdata%\Microsoft\OneDrive\onedrive.exe /reset | OneDrive reset starts |
| 3 | Wait for client to disappear and return | WIN11-CLIENT01 | Observe tray icon | N/A | OneDrive restarts |
| 4 | Start manually if needed | WIN11-CLIENT01 | Run dialog | %localappdata%\Microsoft\OneDrive\OneDrive.exe | OneDrive starts |
| 5 | Confirm account remains connected | WIN11-CLIENT01 | OneDrive settings → Account | N/A | Account visible |
| 6 | Reconnect library if required | Browser | SharePoint library → Sync | N/A | Library sync relationship recreated |
| 7 | Validate sync health | WIN11-CLIENT01 | OneDrive tray icon | N/A | Status shows up to date |
| 8 | Document result | Admin Workstation | N/A | N/A | Ticket updated |

---

# Phase 10 - Troubleshoot OneDrive Provisioning and Personal Site Issues

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Confirm user license | Microsoft 365 Admin Center | User → Licenses and apps | N/A | OneDrive service plan assigned |
| 2 | Ask user to open OneDrive web | Browser | https://www.microsoft365.com/launch/onedrive | N/A | OneDrive provisions if not already |
| 3 | Search personal sites | Admin Workstation | N/A | Get-SPOSite -IncludePersonalSite $true -Limit All -Template SPSPERS \| Where-Object {$_.Owner -like "*user01*"} | User OneDrive site found if provisioned |
| 4 | Check personal site lock state | Admin Workstation | N/A | Get-SPOSite -Identity <OneDriveUrl> \| Select Url, LockState, Owner | OneDrive status displayed |
| 5 | Unlock personal site if incorrectly locked | Admin Workstation | N/A | Set-SPOSite -Identity <OneDriveUrl> -LockState Unlock | OneDrive unlocked |
| 6 | Check storage usage | Admin Workstation | N/A | Get-SPOSite -Identity <OneDriveUrl> \| Select StorageUsageCurrent, StorageQuota | Storage reviewed |
| 7 | Validate user OneDrive access | Browser | Open OneDrive as user01 | N/A | OneDrive opens |

---

# Phase 11 - Troubleshoot DLP, Sensitivity Label, and Compliance Blocks

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Identify policy symptom | Admin Workstation | Check whether user sees DLP tip, encryption error, or sharing block | N/A | Policy symptom identified |
| 2 | Open Purview DLP policies | Purview Portal | Data Loss Prevention → Policies | N/A | DLP policies displayed |
| 3 | Review policy scope | Purview Portal | Open relevant policy → Locations | N/A | SharePoint/OneDrive scope confirmed |
| 4 | Review DLP rule actions | Purview Portal | Policy → Rules → Actions | N/A | Block/warn behavior identified |
| 5 | Open sensitivity labels | Purview Portal | Information protection → Sensitivity labels | N/A | Label configuration displayed |
| 6 | Check label encryption permissions | Purview Portal | Label → Encryption | N/A | Rights assignment reviewed |
| 7 | Connect Purview PowerShell if needed | Admin Workstation | N/A | Connect-IPPSSession | Purview connected |
| 8 | List DLP policies | Admin Workstation | N/A | Get-DlpCompliancePolicy | DLP policies returned |
| 9 | List labels | Admin Workstation | N/A | Get-Label | Labels returned |
| 10 | Validate by changing test file or policy mode only in lab | Purview Portal | Use test mode or pilot scope | N/A | Root cause confirmed without broad impact |

---

# Phase 12 - Troubleshoot Deleted File or Folder Restore

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Confirm deleted item name and original location | Admin Workstation | Ask user for exact file/folder path | N/A | Item details known |
| 2 | Check first-stage recycle bin | Browser | Site contents → Recycle bin | N/A | Deleted item found or not found |
| 3 | Restore from first-stage recycle bin | Browser | Select item → Restore | N/A | Item restored |
| 4 | Check second-stage recycle bin | Browser | Recycle bin → Second-stage recycle bin | N/A | Item found if removed from first-stage |
| 5 | Restore from second-stage recycle bin | Browser | Select item → Restore | N/A | Item restored |
| 6 | Validate restored location | Browser | Original library/folder | N/A | Item appears in original location |
| 7 | Check version history if item exists but content changed | Browser | File → Version history | N/A | Previous versions visible |
| 8 | Restore previous version if needed | Browser | Version history → Restore | N/A | Older version restored |

---

# Phase 13 - Troubleshoot OneDrive Restore Problems

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Confirm OneDrive URL | Admin Workstation | Capture affected OneDrive URL | N/A | Correct personal site identified |
| 2 | Check OneDrive recycle bin | Browser | OneDrive → Recycle bin | N/A | Deleted file visible if recoverable |
| 3 | Restore deleted file | Browser | Select file → Restore | N/A | File restored |
| 4 | Check OneDrive second-stage recycle bin | Browser | Recycle bin → Second-stage recycle bin | N/A | Item found if removed from first-stage |
| 5 | Use Restore your OneDrive for broad damage | Browser | OneDrive settings → Restore your OneDrive | N/A | Point-in-time restore page opens |
| 6 | Select restore point | Browser | Choose time before mass delete/change | N/A | Restore point selected |
| 7 | Review restore impact | Browser | Activity timeline | N/A | Impact understood |
| 8 | Run restore only with user/admin approval | Browser | Restore | N/A | OneDrive restored |
| 9 | Validate restored data | Browser | OneDrive → My files | N/A | Files restored |

---

# Phase 14 - Troubleshoot Deleted SharePoint Site Restore

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Check Active sites | SharePoint Admin Center | Sites → Active sites | N/A | Site confirmed missing or present |
| 2 | Check Deleted sites | SharePoint Admin Center | Sites → Deleted sites | N/A | Deleted site list opens |
| 3 | Search deleted site by URL | SharePoint Admin Center | Filter/search deleted sites | N/A | Deleted site found |
| 4 | List deleted sites with PowerShell | Admin Workstation | N/A | Get-SPODeletedSite \| Select Url, DeletionTime, DaysRemaining, Status | Deleted sites displayed |
| 5 | Restore deleted site | SharePoint Admin Center | Select site → Restore | N/A | Site restore initiated |
| 6 | Restore with PowerShell if needed | Admin Workstation | N/A | Restore-SPODeletedSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam | Site restored |
| 7 | Resolve URL conflict if restore fails | SharePoint Admin Center | Rename or delete conflicting active site | N/A | Conflict removed |
| 8 | Verify restored site | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam | Site returned |

---

# Phase 15 - Troubleshoot Library Restore and Versioning Issues

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Confirm versioning status | Browser | Library settings → Versioning settings | N/A | Versioning configuration visible |
| 2 | Confirm item version history | Browser | File → Version history | N/A | Versions visible |
| 3 | If versions missing, verify file was edited after versioning enabled | Browser | Version history | N/A | Root cause identified |
| 4 | Open Restore this library if broad issue | Browser | Library settings/menu → Restore this library | N/A | Library restore page opens |
| 5 | Review activity timeline | Browser | Restore this library page | N/A | Change timeline visible |
| 6 | Select safe restore point | Browser | Choose time before issue | N/A | Restore point selected |
| 7 | Restore only after owner approval | Browser | Restore | N/A | Library restored |
| 8 | Validate restored library state | Browser | Documents library | N/A | Content restored |

---

# Phase 16 - Check Service Health and Recent Changes

## Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Service health | Microsoft 365 Admin Center | Health → Service health | N/A | Service health page opens |
| 2 | Check SharePoint Online advisories | Microsoft 365 Admin Center | Service health → SharePoint Online | N/A | Known incidents identified or ruled out |
| 3 | Check OneDrive advisories | Microsoft 365 Admin Center | Service health → OneDrive for Business | N/A | Known incidents identified or ruled out |
| 4 | Open Message Center | Microsoft 365 Admin Center | Health → Message center | N/A | Recent changes visible |
| 5 | Review tenant admin audit | Purview Portal | Audit | N/A | Relevant admin actions found |
| 6 | Search affected user activity | Purview Portal | Audit → user01 | N/A | User file/share events identified |
| 7 | Document incident correlation | Admin Workstation | N/A | N/A | Service or change correlation recorded |

---

# Phase 17 - Export Diagnostic Evidence

## Diagnostic Export Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Export affected site details | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam \| Format-List * \| Out-File .\SPO-Troubleshoot-SiteDetails.txt | Site details exported |
| 2 | Export affected site users | Admin Workstation | N/A | Get-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -Limit All \| Export-Csv .\SPO-Troubleshoot-SiteUsers.csv -NoTypeInformation | Site users exported |
| 3 | Export tenant sharing settings | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability, DefaultSharingLinkType, DefaultLinkPermission, SharingDomainRestrictionMode, SharingAllowedDomainList, SharingBlockedDomainList \| Export-Csv .\SPO-Troubleshoot-Sharing.csv -NoTypeInformation | Sharing evidence exported |
| 4 | Export deleted sites | Admin Workstation | N/A | Get-SPODeletedSite \| Select Url, DeletionTime, DaysRemaining, Status \| Export-Csv .\SPO-Troubleshoot-DeletedSites.csv -NoTypeInformation | Deleted site evidence exported |
| 5 | Export DLP policies if connected | Admin Workstation | N/A | Get-DlpCompliancePolicy \| Export-Csv .\Purview-Troubleshoot-DLPPolicies.csv -NoTypeInformation | DLP policy export created |
| 6 | Export labels if connected | Admin Workstation | N/A | Get-Label \| Export-Csv .\Purview-Troubleshoot-Labels.csv -NoTypeInformation | Label export created |
| 7 | Confirm evidence files | Admin Workstation | N/A | Get-ChildItem .\*Troubleshoot* | Evidence files listed |

---

# Phase 18 - Remediation Decision Table

| Symptom | Likely Cause | Corrective Action |
|--------|--------------|------------------|
| User gets access denied to whole site | User not in group, site locked, or CA block | Check permissions, unlock site, review Conditional Access |
| User can open site but not file | Unique file/folder permissions or broken inheritance | Manage access, remove bad links, restore inheritance if appropriate |
| External user cannot open shared file | Guest identity mismatch, sharing disabled, domain blocked | Verify exact guest identity, tenant/site sharing, and domain restrictions |
| Sharing option missing | Tenant or site sharing disabled | Review Get-SPOTenant and Get-SPOSite SharingCapability |
| Anyone link unavailable | Tenant does not allow anonymous sharing | Use Specific people links or change policy with approval |
| OneDrive stuck syncing | Client paused, path issue, file lock, bad cache | Restart client, fix path/name, close locked file, reset OneDrive |
| OneDrive web works but client fails | Local client issue | Reset OneDrive sync client |
| OneDrive not provisioned | User never launched OneDrive or lacks license | Assign license and have user open OneDrive web |
| File missing | Deleted, moved, or permission removed | Check recycle bin, search library, check audit, check permissions |
| Version restore unavailable | Versioning disabled or no older versions | Enable versioning for future and restore from recycle/backup if possible |
| Deleted site will not restore | URL conflict or retention expired | Remove conflict or escalate if retention expired |
| Sharing blocked by policy tip | DLP policy matched sensitive content | Review DLP match and change content, policy, or exception |
| Labeled file cannot open | Sensitivity label encryption denies user | Update label permissions or grant appropriate access |
| Restore your OneDrive unavailable | Permission or scope issue | Use user account or admin-assisted restore workflow |

---

# Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Verify original issue is reproduced | Client Workstation | Repeat failing action | N/A | Failure confirmed before change |
| 2 | Apply one corrective action | Admin Workstation | Based on root cause | N/A | Single change made |
| 3 | Retest with affected user | Client Workstation | Repeat original action | N/A | Issue resolved or next cause identified |
| 4 | Verify site health | Admin Workstation | N/A | Get-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam | Site returns healthy details |
| 5 | Verify user access | Browser | Open affected URL as user | N/A | User can access expected resource |
| 6 | Verify external access if applicable | External Browser Session | Open shared link | N/A | Guest access matches policy |
| 7 | Verify OneDrive sync if applicable | WIN11-CLIENT01 | OneDrive tray icon and test file | N/A | Sync shows up to date |
| 8 | Verify restored data if applicable | Browser | Check original library/OneDrive location | N/A | Restored item present |
| 9 | Verify compliance policy behavior if applicable | Purview Portal | DLP alerts or policy tips | N/A | Policy behavior understood |
| 10 | Document final cause and fix | Ticket System | Update ticket | N/A | Ticket contains cause, fix, and evidence |

---

# Rollback / Cleanup

| Step | Action | PowerShell |
|------|--------|------------|
| 1 | Remove temporary site collection admin | Set-SPOUser -Site https://contoso.sharepoint.com/sites/FinanceTeam -LoginName spadmin@contoso.onmicrosoft.com -IsSiteCollectionAdmin $false |
| 2 | Remove temporary guest access | Remove from Manage access or Entra guest object if lab-only |
| 3 | Restore original sharing settings if changed | Set-SPOSite -Identity https://contoso.sharepoint.com/sites/FinanceTeam -SharingCapability <PreviousValue> |
| 4 | Restore original tenant sharing setting if changed | Set-SPOTenant -SharingCapability <PreviousValue> |
| 5 | Remove test files | Delete TroubleshootTest.docx and sync test files |
| 6 | Remove temporary DLP or label test changes | Revert policy to previous mode or scope |
| 7 | Remove diagnostic exports if required | Remove-Item .\*Troubleshoot* |
| 8 | Disconnect SPO session | Disconnect-SPOService |
| 9 | Disconnect Graph session if used | Disconnect-MgGraph |
| 10 | Disconnect Purview session if used | Disconnect-ExchangeOnline -Confirm:$false |

---

# Related Workbooks

```text
01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md
02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md
03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md
04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews.md
05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restores.md
06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files.md
```