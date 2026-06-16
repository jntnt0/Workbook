10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md
# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Index
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Source_Basis
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Mental_Model
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Planning_Table
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Resource_Map
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Configuration_Checklist
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_File_Server_Precheck_Skeleton
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Create_And_Link_GPO_Skeleton
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Folder_Redirection_GPMC_Skeleton
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Drive_Map_GPP_Skeleton
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Printer_Deployment_GPP_Skeleton
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Client_RSOP_Validation_Skeleton
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Report_And_Backup_Skeleton
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Verification_Commands
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Rollback
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Failure_Checks
10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Related_Labs

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Folder Redirection Group Policy | Redirecting known user folders such as Documents, Desktop, and Pictures |
| Microsoft Learn | Group Policy Preferences Drive Maps | Mapping network drives through user-side GPP |
| Microsoft Learn | Group Policy Preferences Printers | Deploying shared printer connections through user-side GPP |
| Microsoft Learn | Group Policy Management Console | Configuring folder redirection and preference items |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up the GPO |
| Windows File Server | SMB shares and NTFS permissions | Hosting redirected folders and shared drive targets |
| Windows Print Server | Shared printer publishing and client connections | Providing printer deployment targets |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, and GroupPolicy event logs | Validating folder redirection, drive maps, and printer deployment |
| Windows Server operational practice | Pilot first, avoid broad rollout, configure rollback before redirecting data | Preventing user data loss and login disruption |

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Folder Redirection | User shell folders are redirected from the local profile to a network location |
| Known Folder | User folder such as Documents, Desktop, Pictures, Music, Videos, Favorites, or AppData |
| Root path | Base UNC path where redirected user folders are created |
| User data share | SMB share used to store redirected folders |
| Exclusive rights | Folder Redirection option that gives the user exclusive permissions to redirected folder content |
| Move contents | Option that moves existing local folder content to the new redirected path |
| Policy removal behavior | Determines whether redirected folders stay redirected or move back locally if policy stops applying |
| Drive map | GPP item that maps a drive letter to a UNC path |
| Printer deployment | GPP item that connects a shared printer for users or computers |
| Item-level targeting | Per-preference targeting logic for drive maps and printers |
| User Configuration | GPO half normally used for folder redirection, drive maps, and user printer connections |
| Computer Configuration | GPO half used for computer-targeted printers or machine-level preferences |
| Share permissions | SMB permissions controlling access at the network share layer |
| NTFS permissions | File system permissions controlling access to folders and files |
| Offline Files | Client caching feature that may interact with redirected folders |
| Tattooing | Some preference items can remain after GPO no longer applies unless removal is configured |
| First rule | Do not redirect folders until the share, NTFS permissions, backup plan, and rollback behavior are understood |
| Blunt rule | Folder Redirection moves user data, so treat it as higher risk than a normal drive map |

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| NetBIOS domain | `CORP` | `<netbios-domain>` |
| Target user OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<target-user-ou-dn>` |
| Test user | `tuser` | `<test-user>` |
| Test workstation | `WIN11-01` | `<test-workstation>` |
| File server | `FS1` | `<file-server>` |
| Print server | `PRINT1` | `<print-server>` |
| Folder redirection GPO | `CORP-User-Folder-Redirection-Resources` | `<resource-gpo-name>` |
| Folder redirection share | `\\FS1\UserData$` | `<user-data-share>` |
| Folder redirection local path | `D:\Shares\UserData` | `<user-data-local-path>` |
| Shared drive path | `\\FS1\Shared` | `<shared-drive-path>` |
| Drive letter | `S:` | `<drive-letter>` |
| Drive label | `Shared` | `<drive-label>` |
| Printer path | `\\PRINT1\HP-LaserJet-01` | `<printer-path>` |
| Printer default | No | `<yes-no>` |
| Redirected folder 1 | Documents | `<folder-1>` |
| Redirected folder 2 | Desktop | `<folder-2>` |
| Redirected folder 3 | Pictures | `<folder-3>` |
| Move existing contents | Yes for pilot users | `<yes-no>` |
| Grant exclusive rights | Usually No in labs, evaluate for production | `<yes-no>` |
| Removal behavior | Leave folder in new location | `<policy-removal-behavior>` |
| Item-level targeting group | `GG_GPO_User_Resources_Apply` | `<filtering-group>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Disable link, remove GPP items, or reverse folder redirection carefully | `<rollback-plan>` |

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Resource_Map
| Resource | GPO Area | Target | Example | Validation |
|---|---|---|---|---|
| Documents redirection | `User Configuration > Policies > Windows Settings > Folder Redirection` | User OU | `\\FS1\UserData$\%USERNAME%\Documents` | User shell folder path points to UNC |
| Desktop redirection | `User Configuration > Policies > Windows Settings > Folder Redirection` | User OU | `\\FS1\UserData$\%USERNAME%\Desktop` | Desktop path points to UNC |
| Pictures redirection | `User Configuration > Policies > Windows Settings > Folder Redirection` | User OU | `\\FS1\UserData$\%USERNAME%\Pictures` | Pictures path points to UNC |
| Shared drive map | `User Configuration > Preferences > Windows Settings > Drive Maps` | User session | `S:` to `\\FS1\Shared` | `Get-PSDrive -Name S` |
| Department drive map | `User Configuration > Preferences > Windows Settings > Drive Maps` | User group | `P:` to `\\FS1\Departments\IT` | `Get-PSDrive -Name P` |
| Shared printer | `User Configuration > Preferences > Control Panel Settings > Printers` | User session | `\\PRINT1\HP-LaserJet-01` | `Get-Printer` |
| Default printer | Printer preference option | User session | Set selected printer as default | `Get-CimInstance Win32_Printer | Where Default` |
| Shortcut to share | `User Configuration > Preferences > Windows Settings > Shortcuts` | User desktop | Shortcut to `\\FS1\Shared` | Shortcut exists |
| Registry marker | `User Configuration > Preferences > Windows Settings > Registry` | User session | `HKCU\Software\Corp\Resources` | `Get-ItemProperty` |

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target user OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-user-ou-dn>"` | User OU returns |
| 6 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled` | Test user returns |
| 7 | Confirm test user is in target OU | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | User DN is under target OU |
| 8 | Confirm test workstation is domain joined | Test Client | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Client is domain joined |
| 9 | Confirm file server is reachable | Management Host | `Test-Connection FS1 -Count 2` | File server responds |
| 10 | Confirm print server is reachable | Management Host | `Test-Connection PRINT1 -Count 2` | Print server responds |
| 11 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 12 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 13 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 14 | Back up existing GPOs before changes | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 15 | Create folder redirection root folder | File Server | `New-Item -ItemType Directory -Force -Path D:\Shares\UserData` | User data folder exists |
| 16 | Create shared drive folder | File Server | `New-Item -ItemType Directory -Force -Path D:\Shares\Shared` | Shared folder exists |
| 17 | Create SMB share for redirected folders | File Server | `New-SmbShare -Name "UserData$" -Path "D:\Shares\UserData" -FullAccess "CORP\Domain Admins","SYSTEM" -ChangeAccess "CORP\Domain Users"` | Hidden user data share exists |
| 18 | Create SMB share for shared drive | File Server | `New-SmbShare -Name "Shared" -Path "D:\Shares\Shared" -FullAccess "CORP\Domain Admins","SYSTEM" -ChangeAccess "CORP\Domain Users"` | Shared drive share exists |
| 19 | Configure NTFS permissions for user data root | File Server | `icacls D:\Shares\UserData` | Permission state is known |
| 20 | Configure NTFS permissions for shared drive | File Server | `icacls D:\Shares\Shared /grant "CORP\Domain Users:(OI)(CI)(M)"` | Domain users can modify shared drive content |
| 21 | Confirm UNC access to user data root | Management Host | `Test-Path "\\FS1\UserData$"` | Returns True |
| 22 | Confirm UNC access to shared drive | Management Host | `Test-Path "\\FS1\Shared"` | Returns True |
| 23 | Confirm printer share exists | Management Host | `Get-Printer -ComputerName PRINT1` | Shared printer appears |
| 24 | Create resource GPO if missing | Management Host | `New-GPO -Name "<resource-gpo-name>" -Comment "Folder redirection, drive maps, and printer deployment"` | GPO exists |
| 25 | Link resource GPO to user OU | Management Host | `New-GPLink -Name "<resource-gpo-name>" -Target "<target-user-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 26 | Keep link unenforced by default | Management Host | `Set-GPLink -Name "<resource-gpo-name>" -Target "<target-user-ou-dn>" -Enforced No` | Link is not enforced |
| 27 | Confirm link state | Management Host | `Get-GPInheritance -Target "<target-user-ou-dn>"` | Resource GPO appears in link list |
| 28 | Confirm GPO permissions | Management Host | `Get-GPPermission -Name "<resource-gpo-name>" -All` | Read and Apply permissions are known |
| 29 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<resource-gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 30 | Open GPMC editor | Management Host | `gpmc.msc` | GPMC opens |
| 31 | Configure Documents Folder Redirection | Management Host | `User Configuration > Policies > Windows Settings > Folder Redirection > Documents` | Documents redirection is configured |
| 32 | Configure Desktop Folder Redirection if required | Management Host | `User Configuration > Policies > Windows Settings > Folder Redirection > Desktop` | Desktop redirection is configured |
| 33 | Configure Pictures Folder Redirection if required | Management Host | `User Configuration > Policies > Windows Settings > Folder Redirection > Pictures` | Pictures redirection is configured |
| 34 | Configure drive map | Management Host | `User Configuration > Preferences > Windows Settings > Drive Maps` | `S:` maps to `\\FS1\Shared` |
| 35 | Configure printer deployment | Management Host | `User Configuration > Preferences > Control Panel Settings > Printers` | Shared printer is deployed |
| 36 | Configure item-level targeting if used | Management Host | `Common tab > Item-level targeting` | Items apply only to intended users |
| 37 | Export GPO report | Management Host | `Get-GPOReport -Name "<resource-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<resource-gpo-name>.html` | Report shows folder redirection and GPP items |
| 38 | Back up configured GPO | Management Host | `Backup-GPO -Name "<resource-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 39 | Sign in as test user | Test Client | `whoami` | Test user session is active |
| 40 | Force policy refresh | Test Client | `gpupdate /force` | User policy refresh completes |
| 41 | Sign out and sign back in | Test Client | Sign out, then log on again | Folder redirection fully applies |
| 42 | Validate applied user GPOs | Test Client | `gpresult /scope user /r` | Resource GPO appears under Applied GPOs |
| 43 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-user-resources.html` | HTML RSOP report exists |
| 44 | Validate redirected Documents path | Test Client | `[Environment]::GetFolderPath("MyDocuments")` | Documents path points to file server |
| 45 | Validate mapped drive | Test Client | `Get-PSDrive -Name S -ErrorAction SilentlyContinue` | `S:` drive exists |
| 46 | Validate printer connection | Test Client | `Get-Printer` | Deployed printer appears |
| 47 | Review Group Policy events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Recent processing events are visible |
| 48 | Document result | Operator | `Record GPO, target OU, shares, redirected folders, drives, printers, reports, backup, and client result` | User resource deployment is documented |

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_File_Server_Precheck_Skeleton
```powershell
# Run on the file server as an administrator.
# Purpose: create and validate shares for folder redirection and mapped drives.

$DomainNetBIOS = "CORP"

$UserDataRoot = "D:\Shares\UserData"
$SharedRoot = "D:\Shares\Shared"

$UserDataShareName = "UserData$"
$SharedShareName = "Shared"

# Create root folders.
New-Item -ItemType Directory -Force -Path $UserDataRoot,$SharedRoot

# Create user data share for folder redirection if missing.
if (-not (Get-SmbShare -Name $UserDataShareName -ErrorAction SilentlyContinue)) {
    New-SmbShare `
      -Name $UserDataShareName `
      -Path $UserDataRoot `
      -FullAccess "$DomainNetBIOS\Domain Admins","SYSTEM" `
      -ChangeAccess "$DomainNetBIOS\Domain Users"
}

# Create shared drive share if missing.
if (-not (Get-SmbShare -Name $SharedShareName -ErrorAction SilentlyContinue)) {
    New-SmbShare `
      -Name $SharedShareName `
      -Path $SharedRoot `
      -FullAccess "$DomainNetBIOS\Domain Admins","SYSTEM" `
      -ChangeAccess "$DomainNetBIOS\Domain Users"
}

# Baseline NTFS permissions for shared drive.
icacls $SharedRoot /inheritance:r
icacls $SharedRoot /grant "SYSTEM:(OI)(CI)(F)"
icacls $SharedRoot /grant "Administrators:(OI)(CI)(F)"
icacls $SharedRoot /grant "$DomainNetBIOS\Domain Admins:(OI)(CI)(F)"
icacls $SharedRoot /grant "$DomainNetBIOS\Domain Users:(OI)(CI)(M)"

# Baseline NTFS permissions for folder redirection root.
# Production permissions may differ. Validate against your organization's data ownership standard.
icacls $UserDataRoot /inheritance:r
icacls $UserDataRoot /grant "SYSTEM:(OI)(CI)(F)"
icacls $UserDataRoot /grant "Administrators:(OI)(CI)(F)"
icacls $UserDataRoot /grant "$DomainNetBIOS\Domain Admins:(OI)(CI)(F)"
icacls $UserDataRoot /grant "CREATOR OWNER:(OI)(CI)(IO)(F)"
icacls $UserDataRoot /grant "$DomainNetBIOS\Domain Users:(RX)"
icacls $UserDataRoot /grant "$DomainNetBIOS\Domain Users:(AD)"
icacls $UserDataRoot /grant "$DomainNetBIOS\Domain Users:(WD)"

# Confirm shares.
Get-SmbShare -Name $UserDataShareName,$SharedShareName |
  Select-Object Name,Path,Description

# Confirm share permissions.
Get-SmbShareAccess -Name $UserDataShareName
Get-SmbShareAccess -Name $SharedShareName

# Confirm NTFS permissions.
icacls $UserDataRoot
icacls $SharedRoot

# Optional seed file for drive map testing.
"Shared drive test file created $(Get-Date -Format o)" |
  Out-File "$SharedRoot\SharedDrive-Test.txt" -Encoding UTF8
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link the resource delivery GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-User-Folder-Redirection-Resources"
$GpoComment = "User resource delivery GPO for folder redirection, drive maps, and printer deployment."

# Confirm target OU exists.
Get-ADOrganizationalUnit -Identity $TargetUserOU

# Create the GPO if missing.
$ExistingGpo = Get-GPO `
  -Name $GpoName `
  -ErrorAction SilentlyContinue

if ($ExistingGpo) {
    Write-Host "GPO already exists: $GpoName"
}
else {
    New-GPO `
      -Name $GpoName `
      -Comment $GpoComment

    Write-Host "Created GPO: $GpoName"
}

# Link the GPO to the target user OU if not already linked.
$Inheritance = Get-GPInheritance -Target $TargetUserOU

$ExistingLink = $Inheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $GpoName }

if ($ExistingLink) {
    Write-Host "GPO link already exists on target user OU."
}
else {
    New-GPLink `
      -Name $GpoName `
      -Target $TargetUserOU `
      -LinkEnabled Yes

    Write-Host "Linked $GpoName to $TargetUserOU"
}

# Safe default: enabled, not enforced.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetUserOU `
  -LinkEnabled Yes `
  -Enforced No

# Optional link order.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetUserOU `
  -Order 1

# Confirm final link state.
Get-GPInheritance `
  -Target $TargetUserOU |
  Format-List
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Folder_Redirection_GPMC_Skeleton
```powershell
# Native GUI workflow for Folder Redirection.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-User-Folder-Redirection-Resources
# 7. Select Edit.
# 8. Go to:
#      User Configuration
#      > Policies
#      > Windows Settings
#      > Folder Redirection
#
# Documents redirection:
# 9. Right-click Documents.
# 10. Select Properties.
# 11. Setting:
#      Basic - Redirect everyone's folder to the same location
# 12. Target folder location:
#      Create a folder for each user under the root path
# 13. Root Path:
#      \\FS1\UserData$
# 14. Settings tab:
#      Grant the user exclusive rights to Documents: choose based on design
#      Move the contents of Documents to the new location: Enabled for pilot
#      Also apply redirection policy to older Windows operating systems: optional
# 15. Policy Removal:
#      Leave the folder in the new location when policy is removed
#
# Desktop redirection:
# 16. Repeat for Desktop only if approved.
# 17. Root Path:
#      \\FS1\UserData$
#
# Pictures redirection:
# 18. Repeat for Pictures only if approved.
# 19. Root Path:
#      \\FS1\UserData$
#
# Notes:
# - Start with Documents in a pilot.
# - Desktop redirection can impact logon performance if users store large files on Desktop.
# - Do not redirect AppData unless there is a strong reason and a tested design.
# - Confirm backup coverage before redirecting real user data.
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Drive_Map_GPP_Skeleton
```powershell
# Native GUI workflow for mapped drives through Group Policy Preferences.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-User-Folder-Redirection-Resources
# 3. Go to:
#      User Configuration
#      > Preferences
#      > Windows Settings
#      > Drive Maps
# 4. Right-click Drive Maps.
# 5. Select:
#      New > Mapped Drive
#
# General tab:
# 6. Action:
#      Update
# 7. Location:
#      \\FS1\Shared
# 8. Reconnect:
#      Enabled
# 9. Label as:
#      Shared
# 10. Drive Letter:
#      Use S:
#
# Common tab:
# 11. Run in logged-on user's security context:
#      Enabled
# 12. Remove this item when it is no longer applied:
#      Enabled
# 13. Item-level targeting:
#      Enabled if using a group, OU, computer, or IP condition
#
# Optional item-level targeting:
# 14. Click Targeting.
# 15. Add:
#      Security Group
# 16. Group:
#      GG_GPO_User_Resources_Apply
# 17. User in group:
#      Enabled
#
# Close editor.
# Refresh GPMC.
# Export a GPO report.
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Printer_Deployment_GPP_Skeleton
```powershell
# Native GUI workflow for shared printer deployment through Group Policy Preferences.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-User-Folder-Redirection-Resources
# 3. Go to:
#      User Configuration
#      > Preferences
#      > Control Panel Settings
#      > Printers
# 4. Right-click Printers.
# 5. Select:
#      New > Shared Printer
#
# General tab:
# 6. Action:
#      Update
# 7. Share path:
#      \\PRINT1\HP-LaserJet-01
# 8. Set this printer as the default printer:
#      Optional
# 9. Only if local printer is not present:
#      Optional
#
# Common tab:
# 10. Run in logged-on user's security context:
#      Enabled
# 11. Remove this item when it is no longer applied:
#      Enabled
# 12. Item-level targeting:
#      Enabled if deploying by group, location, computer, OU, or IP range
#
# Optional item-level targeting:
# 13. Click Targeting.
# 14. Add:
#      Security Group
# 15. Group:
#      GG_GPO_User_Resources_Apply
# 16. User in group:
#      Enabled
#
# Notes:
# - Printer driver installation behavior may depend on current Windows point-and-print restrictions.
# - Test with a standard user account, not only with an admin account.
# - Keep printer deployment separate from folder redirection when troubleshooting.
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client while signed in as the test user.
# Purpose: prove folder redirection, drive maps, and printer deployment work.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm user and client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Confirm direct access to resource paths.
Test-Path "\\FS1\UserData$"
Test-Path "\\FS1\Shared"
Test-Path "\\PRINT1\HP-LaserJet-01"

# Force policy refresh.
gpupdate /force

# Sign out and sign back in after first Folder Redirection configuration.
# Folder Redirection often completes cleanly after a new logon session.

# Show applied user GPOs.
gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-resources.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-folder-drive-printer.html"

# Optional RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-folder-drive-printer.html" `
  -ErrorAction SilentlyContinue

# Validate redirected known folders.
[PSCustomObject]@{
    Desktop   = [Environment]::GetFolderPath("Desktop")
    Documents = [Environment]::GetFolderPath("MyDocuments")
    Pictures  = [Environment]::GetFolderPath("MyPictures")
} | Tee-Object "$ReportPath\known-folder-paths.txt"

# Validate mapped drive.
Get-PSDrive `
  -Name S `
  -ErrorAction SilentlyContinue |
  Format-List

# Validate drive map content.
Test-Path "S:\"
Test-Path "S:\SharedDrive-Test.txt"

# Validate printer connection.
Get-Printer |
  Select-Object Name,ShareName,PortName,DriverName,Type |
  Sort-Object Name |
  Tee-Object "$ReportPath\printers-after-gpo.txt"

# Validate default printer if configured.
Get-CimInstance Win32_Printer |
  Where-Object Default -eq $true |
  Select-Object Name,Default,Network,ShareName

# Review Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 150 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-folder-drive-printer.txt"
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final GPO report, inheritance, permissions, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"
$GpoName = "CORP-User-Folder-Redirection-Resources"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\user-ou-inheritance-after-folder-drive-printer.txt"

# Capture final GPO permissions.
Get-GPPermission `
  -Name $GpoName `
  -All |
  Out-File "$ReportPath\$GpoName-permissions-after.txt"

# Export final reports.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-final.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-final.xml"

# Search XML report for resource configuration.
Select-String `
  -Path "$ReportPath\$GpoName-final.xml" `
  -Pattern "Folder Redirection","Drive","Printer","Documents","Desktop","Pictures","FS1","PRINT1","UserData","Shared" |
  Out-File "$ReportPath\$GpoName-resource-report-search.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-folder-drive-printer.csv" -NoTypeInformation

Write-Host "Folder redirection, drive map, and printer deployment report complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<resource-gpo-name>"` | Confirms resource GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<target-user-ou-dn>"` | Confirms resource GPO is linked to user OU | GPO appears in link list |
| `Get-GPPermission -Name "<resource-gpo-name>" -All` | Confirms test user can read and apply GPO | Expected permissions appear |
| `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | Confirms user is in target OU scope | User DN is under target user OU |
| `Test-Path "\\FS1\UserData$"` | Confirms redirected folder root is reachable | Returns True |
| `Test-Path "\\FS1\Shared"` | Confirms mapped drive target is reachable | Returns True |
| `Get-SmbShare -CimSession FS1` | Lists file server shares remotely | UserData and Shared shares appear |
| `Get-SmbShareAccess -Name UserData$` | Confirms share permissions | Intended access entries appear |
| `icacls D:\Shares\UserData` | Confirms NTFS permissions on file server | Expected ACL entries appear |
| `Get-Printer -ComputerName PRINT1` | Lists print server printers | Target printer appears |
| `Get-GPOReport -Name "<resource-gpo-name>" -ReportType Html -Path "<report-path>\<resource-gpo-name>.html"` | Exports readable GPO report | HTML report exists |
| `Get-GPOReport -Name "<resource-gpo-name>" -ReportType Xml -Path "<report-path>\<resource-gpo-name>.xml"` | Exports searchable GPO report | XML report exists |
| `gpupdate /force` | Forces user policy refresh | Policy update completes |
| `gpresult /scope user /r` | Shows applied user GPOs | Resource GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-folder-drive-printer.html` | Exports full RSOP report | HTML report exists |
| `[Environment]::GetFolderPath("MyDocuments")` | Validates redirected Documents path | Path points to file server |
| `[Environment]::GetFolderPath("Desktop")` | Validates redirected Desktop path if configured | Path points to file server |
| `Get-PSDrive -Name S` | Validates mapped drive | `S:` drive exists |
| `Test-Path "S:\"` | Validates mapped drive access | Returns True |
| `Get-Printer` | Validates deployed printer | Target printer appears |
| `Get-CimInstance Win32_Printer | Where-Object Default -eq $true` | Validates default printer if configured | Expected default printer appears |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews policy processing events | Recent events show processing status |

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: rollback resource delivery policy safely.
# Folder Redirection rollback must be planned because user data may already exist on the file server.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-User-Folder-Redirection-Resources"

$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture state before rollback.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-before-rollback.html"

Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Rollback option 1:
# Disable the GPO link while preserving the GPO for review.
# Use carefully with Folder Redirection because removal behavior depends on the Folder Redirection settings.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetUserOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Use GPMC to remove drive maps and printer preferences first.
gpmc.msc

# GUI rollback workflow for drive maps and printers:
# 1. Edit the GPO.
# 2. Go to User Configuration > Preferences.
# 3. Remove or disable Drive Maps preference items.
# 4. Remove or disable Printers preference items.
# 5. Confirm "Remove this item when it is no longer applied" was enabled before expecting cleanup.
# 6. Export a fresh report.
# 7. Run gpupdate /force as test user.
#
# GUI rollback workflow for Folder Redirection:
# 8. Go to User Configuration > Policies > Windows Settings > Folder Redirection.
# 9. Review the Settings tab removal behavior.
# 10. Decide whether to leave data in the redirected location or redirect back to local profile.
# 11. Pilot rollback with one test user before broad rollout.
# 12. Confirm data is present and accessible before deleting any server-side folders.

# Rollback option 3:
# Remove the whole GPO only in a disposable lab.
# Remove-GPO `
#   -Name $GpoName

# Capture after rollback.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-after-rollback.html" `
  -ErrorAction SilentlyContinue

Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\user-ou-inheritance-after-resource-rollback.txt"

Write-Host "Rollback action complete."
Write-Host "On test client: run gpupdate /force, sign out and sign back in, then validate folders, drives, and printers."
```

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Folder Redirection does not apply | User object not in linked OU | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | Move user to target OU or link GPO to correct user OU |
| Folder Redirection applies slowly | Large local folder content, slow network, Desktop redirection, or Offline Files interaction | GroupPolicy Operational log; file copy timing | Pilot smaller scope, redirect Documents first, avoid Desktop until tested |
| Documents path remains local | GPO not applied, folder redirection not configured, or user has not logged on again | `gpresult /scope user /r`; `[Environment]::GetFolderPath("MyDocuments")` | Fix GPO scope and sign out/sign in |
| User receives access denied during redirection | NTFS or share permissions wrong | `Test-Path \\FS1\UserData$`; file server ACL check | Fix SMB and NTFS permissions |
| User folder is not created on server | User lacks create folder rights on root path | `icacls D:\Shares\UserData` | Correct root folder permissions |
| Admin cannot access redirected user folders | Exclusive rights enabled or NTFS ownership design | File server ACL check | Decide whether exclusive rights are required and document admin access model |
| Drive map does not appear | Share unavailable, permissions issue, item-level targeting failed, or user context option wrong | `Test-Path \\FS1\Shared`; `gpresult /r` | Fix path, permissions, Common tab, or targeting |
| Drive map persists after GPO no longer applies | Remove item option was not enabled or item tattooed | GPO drive map Common tab | Enable removal option or manually remove drive |
| Printer does not deploy | Print server unreachable, driver restriction, permissions issue, or targeting failed | `Get-Printer -ComputerName PRINT1`; `gpresult /r` | Fix print server, driver policy, permissions, or targeting |
| Printer prompts for admin credentials | Point-and-print driver restrictions or driver install policy | Client event logs and printer policy settings | Pre-stage driver or configure approved print deployment policy |
| Default printer does not change | Windows default printer management or GPP option not set | `Get-CimInstance Win32_Printer | Where Default` | Configure default printer intentionally and test user behavior |
| GPO appears denied | Security filtering or WMI filter blocks user | `gpresult /scope user /r`; `Get-GPPermission` | Grant Apply permission or correct WMI filter |
| GPO does not show in gpresult | Wrong OU, disabled link, or replication delay | `Get-GPInheritance -Target "<target-user-ou-dn>"` | Fix link, OU, or replication |
| Folder redirection affects too many users | Link target too broad or filtering too broad | `Get-GPInheritance`; `Get-GPPermission` | Link lower or narrow security filtering |
| Network share works for admin but not test user | Permissions tested with wrong account | Sign in as test user and run `Test-Path` | Fix share and NTFS permissions for user |
| GPO report does not show folder redirection | Wrong GPO edited or stale report | `Get-GPOReport -Name "<resource-gpo-name>"` | Edit correct GPO and export fresh report |
| GPO report does not show drive/printer items | Preference item saved in wrong GPO or wrong side | GPMC editor path and report | Move item to User Configuration Preferences |
| Client has old resource state after rollback | User session stale or preference cleanup not configured | `gpupdate /force`; sign out/sign in | Refresh policy and manually clean tattooed items |
| SYSVOL unavailable | DNS, SMB, or DC issue | `Test-Path "\\<domain>\SYSVOL"`; `dcdiag /test:sysvolcheck` | Fix DNS, SMB, or SYSVOL |
| Resource behavior differs by client | Replication, targeting, cached credentials, driver, or permissions difference | `repadmin /replsummary`; `gpresult /h` | Fix replication, targeting, driver state, or permissions |
| User data missing after rollback | Folder Redirection removal behavior misunderstood | Server folder check and backup | Restore from backup and pilot rollback before broad changes |

# 10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before user resource deployment |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which users can apply the resource GPO |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Adds optional targeting before resource deployment |
| `07_Configure_User_Administrative_Template_Settings.md` | User-side policy companion workbook |
| `08_Configure_Group_Policy_Preferences.md` | Preference item foundation used for drive maps and printers |
| `09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md` | Script alternative when GPP is not sufficient |
| `Create_Baseline_OU_Structure.md` | Provides user OU scope for Folder Redirection and GPP links |
| `Create_Baseline_Groups_And_Test_Users.md` | Provides test user and filtering group |
| `Join_Windows_Client_To_Domain.md` | Provides domain-joined test client for validation |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms SYSVOL and DC locator before troubleshooting user resources |
| `11_Configure_Software_Installation_And_App_Deployment_With_GPO.md` | Next workbook for software deployment through GPO |