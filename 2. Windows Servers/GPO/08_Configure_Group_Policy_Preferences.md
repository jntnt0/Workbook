08_Configure_Group_Policy_Preferences.md
# 08_Configure_Group_Policy_Preferences

# 08_Configure_Group_Policy_Preferences_Index
08_Configure_Group_Policy_Preferences.md
08_Configure_Group_Policy_Preferences
08_Configure_Group_Policy_Preferences_Source_Basis
08_Configure_Group_Policy_Preferences_Mental_Model
08_Configure_Group_Policy_Preferences_Planning_Table
08_Configure_Group_Policy_Preferences_Preference_Item_Map
08_Configure_Group_Policy_Preferences_Configuration_Checklist
08_Configure_Group_Policy_Preferences_Precheck_Skeleton
08_Configure_Group_Policy_Preferences_Create_And_Link_GPO_Skeleton
08_Configure_Group_Policy_Preferences_GPMC_User_Preferences_Skeleton
08_Configure_Group_Policy_Preferences_GPMC_Computer_Preferences_Skeleton
08_Configure_Group_Policy_Preferences_Item_Level_Targeting_Skeleton
08_Configure_Group_Policy_Preferences_Client_RSOP_Validation_Skeleton
08_Configure_Group_Policy_Preferences_Report_And_Backup_Skeleton
08_Configure_Group_Policy_Preferences_Verification_Commands
08_Configure_Group_Policy_Preferences_Rollback
08_Configure_Group_Policy_Preferences_Failure_Checks
08_Configure_Group_Policy_Preferences_Related_Labs

# 08_Configure_Group_Policy_Preferences_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy Preferences | Configuring preferences for users and computers |
| Microsoft Learn | Group Policy Management Console | Editing preference items under User Configuration and Computer Configuration |
| Microsoft Learn | Common tab and item-level targeting | Targeting preference items by group, OU, computer, IP range, operating system, and other conditions |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up the GPO that stores preferences |
| Microsoft Learn | `Get-GPOReport` | Exporting readable and searchable preference reports |
| Microsoft Learn | `Backup-GPO` | Backing up GPO preference state before and after changes |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, and GroupPolicy event logs | Proving preference items apply to the expected users and computers |
| Windows Server operational practice | Separate GPP from strict Administrative Template policy | Keeping flexible user environment configuration supportable |

# 08_Configure_Group_Policy_Preferences_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Group Policy Preferences | GPO-based configuration items that set user or computer environment values without always enforcing them like strict policy |
| Preference item | Individual configured object such as mapped drive, printer, registry value, file, folder, shortcut, or scheduled task |
| User Preferences | Preferences under `User Configuration > Preferences` that apply to user sessions |
| Computer Preferences | Preferences under `Computer Configuration > Preferences` that apply to computer accounts |
| Action Create | Creates the item if it does not exist |
| Action Replace | Deletes and recreates the item every refresh |
| Action Update | Updates the item if it exists and creates it if missing |
| Action Delete | Removes the item |
| Common tab | Preference item tab containing targeting and behavior options |
| Item-level targeting | Per-item logic that decides whether the preference item applies |
| Run in logged-on user's security context | User preference option that applies the item using the user's permissions |
| Remove this item when it is no longer applied | Option that removes the preference item when scope no longer matches |
| Apply once and do not reapply | Option that writes preference once and then stops enforcing it |
| Tattooing | Preference remains after GPO no longer applies unless removal is configured |
| Drive map preference | User preference commonly used to map network drives |
| Printer preference | User preference commonly used to connect shared printers |
| Registry preference | Preference used to create, update, or delete registry values |
| File preference | Preference used to copy, update, replace, or delete files |
| Folder preference | Preference used to create, update, replace, or delete folders |
| Shortcut preference | Preference used to create desktop, Start Menu, or shell shortcuts |
| Scheduled task preference | Preference used to create user or computer scheduled tasks |
| First rule | Use GPP for environment setup, not for security enforcement |
| Blunt rule | Never store passwords in legacy GPP preference items |

# 08_Configure_Group_Policy_Preferences_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target user OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<target-user-ou-dn>` |
| Target computer OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-computer-ou-dn>` |
| Test user | `tuser` | `<test-user>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| GPP GPO name | `CORP-User-Preferences` | `<gpp-gpo-name>` |
| Security filtering group | `GG_GPO_User_Preferences_Apply` | `<filtering-group>` |
| Drive map letter | `S:` | `<drive-letter>` |
| Drive map path | `\\FS1\Shared` | `<drive-path>` |
| Printer path | `\\PRINT1\HP-LaserJet-01` | `<printer-path>` |
| Desktop shortcut name | `Helpdesk Portal` | `<shortcut-name>` |
| Desktop shortcut target | `https://helpdesk.corp.local` | `<shortcut-target>` |
| Registry preference path | `HKCU\Software\Corp\Preferences` | `<registry-path>` |
| File copy source | `\\corp.local\NETLOGON\Scripts\sample.txt` | `<file-source>` |
| File copy destination | `%USERPROFILE%\Documents\sample.txt` | `<file-destination>` |
| Item-level targeting condition | User is member of `GG_GPO_User_Preferences_Apply` | `<ilt-condition>` |
| Remove when no longer applied | Yes for drive/printer/shortcut | `<yes-no>` |
| Apply once | No for baseline items | `<yes-no>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Delete preference items or disable GPO link | `<rollback-plan>` |

# 08_Configure_Group_Policy_Preferences_Preference_Item_Map
| Preference Type | Configuration Side | Typical Use | Example | Recommended Action | Validation |
|---|---|---|---|---|---|
| Drive Maps | User Configuration | Map shared folders | `S:` to `\\FS1\Shared` | Update | `Get-PSDrive`; File Explorer |
| Printers | User Configuration | Connect shared printers | `\\PRINT1\HP-LaserJet-01` | Update | `Get-Printer` |
| Files | User or Computer Configuration | Copy support files | `\\NETLOGON\sample.txt` to user Documents | Update or Replace | `Test-Path` |
| Folders | User or Computer Configuration | Create folder structure | `%USERPROFILE%\Corp` | Update | `Test-Path` |
| Registry | User or Computer Configuration | Set non-policy app values | `HKCU\Software\Corp\Preferences` | Update | `Get-ItemProperty` |
| Shortcuts | User Configuration | Create desktop or Start Menu shortcuts | Helpdesk Portal shortcut | Update | Desktop shortcut exists |
| Environment | User or Computer Configuration | Set environment variables | `CORP_ENV=Lab` | Update | `[Environment]::GetEnvironmentVariable()` |
| Scheduled Tasks | User or Computer Configuration | Run maintenance tasks | User logon helper task | Update | `Get-ScheduledTask` |
| Local Users and Groups | Computer Configuration | Local group membership management | Add support group to local admins | Use carefully | `Get-LocalGroupMember` |
| INI Files | User or Computer Configuration | Legacy application configuration | App INI update | Update | File content check |
| Data Sources | User or Computer Configuration | ODBC DSN configuration | App DSN | Update | ODBC console |
| Services | Computer Configuration | Service startup state | Set service startup | Use carefully | `Get-Service` |

# 08_Configure_Group_Policy_Preferences_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target user OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-user-ou-dn>"` | User OU returns |
| 6 | Confirm target computer OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-computer-ou-dn>"` | Computer OU returns |
| 7 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled` | Test user returns |
| 8 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 9 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Confirm GPMC opens | Management Host | `gpmc.msc` | GPMC opens |
| 11 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 12 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 13 | Back up existing GPO state | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 14 | Create GPP GPO if missing | Management Host | `New-GPO -Name "<gpp-gpo-name>" -Comment "Group Policy Preferences for user environment configuration"` | GPO exists |
| 15 | Link GPP GPO to user OU | Management Host | `New-GPLink -Name "<gpp-gpo-name>" -Target "<target-user-ou-dn>" -LinkEnabled Yes` | GPO is linked to user OU |
| 16 | Keep GPP GPO unenforced by default | Management Host | `Set-GPLink -Name "<gpp-gpo-name>" -Target "<target-user-ou-dn>" -Enforced No` | Link is not enforced |
| 17 | Confirm link state | Management Host | `Get-GPInheritance -Target "<target-user-ou-dn>"` | GPO appears in link list |
| 18 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<gpp-gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 19 | Preserve required read access | Management Host | `Set-GPPermission -Name "<gpp-gpo-name>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | Authenticated Users can read GPO |
| 20 | Open GPO editor | Management Host | `GPMC > right-click GPO > Edit` | Group Policy Management Editor opens |
| 21 | Create drive map preference | Management Host | `User Configuration > Preferences > Windows Settings > Drive Maps` | Drive map item exists |
| 22 | Configure drive map action | Management Host | `Action = Update; Location = \\FS1\Shared; Drive Letter = S:` | Drive map preference is configured |
| 23 | Configure drive map Common tab | Management Host | `Run in logged-on user's security context = enabled; Remove item when no longer applied = enabled` | Drive map behaves safely |
| 24 | Create printer preference | Management Host | `User Configuration > Preferences > Control Panel Settings > Printers` | Printer preference item exists |
| 25 | Configure shared printer | Management Host | `Action = Update; Shared printer = \\PRINT1\HP-LaserJet-01` | Printer preference is configured |
| 26 | Create shortcut preference | Management Host | `User Configuration > Preferences > Windows Settings > Shortcuts` | Shortcut preference item exists |
| 27 | Configure shortcut target | Management Host | `Name = Helpdesk Portal; Target = https://helpdesk.corp.local` | Shortcut preference is configured |
| 28 | Create registry preference | Management Host | `User Configuration > Preferences > Windows Settings > Registry` | Registry preference item exists |
| 29 | Configure registry preference | Management Host | `Action = Update; Hive = HKEY_CURRENT_USER; Key Path = Software\Corp\Preferences; Value = GPP_Applied` | Registry preference is configured |
| 30 | Configure item-level targeting | Management Host | `Common tab > Item-level targeting > Targeting` | Item applies only to intended group, user, computer, OU, or condition |
| 31 | Export GPO report | Management Host | `Get-GPOReport -Name "<gpp-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<gpp-gpo-name>.html` | Report shows preference items |
| 32 | Back up configured GPO | Management Host | `Backup-GPO -Name "<gpp-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 33 | Sign in as test user | Test Client | `whoami` | Test user session is active |
| 34 | Force policy refresh | Test Client | `gpupdate /force` | User and computer policy refresh completes |
| 35 | Validate applied user GPOs | Test Client | `gpresult /scope user /r` | GPP GPO appears under Applied GPOs |
| 36 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-gpp.html` | HTML RSOP report exists |
| 37 | Validate mapped drive | Test Client | `Get-PSDrive -Name S -ErrorAction SilentlyContinue` | Mapped drive exists |
| 38 | Validate shared printer | Test Client | `Get-Printer | Where-Object Name -like "*HP-LaserJet*"` | Printer connection appears |
| 39 | Validate shortcut | Test Client | `Test-Path "$env:USERPROFILE\Desktop\Helpdesk Portal.lnk"` | Shortcut exists |
| 40 | Validate registry preference | Test Client | `Get-ItemProperty -Path "HKCU:\Software\Corp\Preferences"` | Preference registry value exists |
| 41 | Review Group Policy events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 120` | Recent processing events are visible |
| 42 | Document result | Operator | `Record preference items, target scope, ILT rules, reports, backup, and client result` | GPP configuration is documented |

# 08_Configure_Group_Policy_Preferences_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring Group Policy Preferences.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"
$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"

$TestUser = "tuser"
$TestComputer = "WIN11-01"

$GpoName = "CORP-User-Preferences"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportPath,$BackupPath

# Confirm domain and tool readiness.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
Get-Module -ListAvailable GroupPolicy
Get-Command -Module GroupPolicy | Select-Object Name | Sort-Object Name

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm OU targets.
Get-ADOrganizationalUnit -Identity $TargetUserOU
Get-ADOrganizationalUnit -Identity $TargetComputerOU

# Confirm test principals.
Get-ADUser `
  -Identity $TestUser `
  -Properties DistinguishedName,Enabled |
  Select-Object SamAccountName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-user-before-gpp.txt"

Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-before-gpp.txt"

# Capture current inheritance.
Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\user-ou-inheritance-before-gpp.txt"

Get-GPInheritance `
  -Target $TargetComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance-before-gpp.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-gpp.csv" -NoTypeInformation

# Back up current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath
```

# 08_Configure_Group_Policy_Preferences_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link a dedicated Group Policy Preferences GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-User-Preferences"
$GpoComment = "Group Policy Preferences for user environment configuration such as drives, printers, shortcuts, files, folders, and registry preferences."

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

# Optional: set desired link order.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetUserOU `
  -Order 1

# Confirm final link state.
Get-GPInheritance `
  -Target $TargetUserOU |
  Format-List
```

# 08_Configure_Group_Policy_Preferences_GPMC_User_Preferences_Skeleton
```powershell
# Native GUI workflow for configuring user Group Policy Preferences.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-User-Preferences
# 7. Select Edit.
# 8. Go to:
#      User Configuration
#      > Preferences
#
# Drive map preference:
# 9. Go to:
#      Windows Settings
#      > Drive Maps
# 10. Right-click Drive Maps.
# 11. Select:
#      New > Mapped Drive
# 12. General tab:
#      Action: Update
#      Location: \\FS1\Shared
#      Reconnect: Enabled
#      Label as: Shared
#      Drive Letter: Use S:
# 13. Common tab:
#      Run in logged-on user's security context: Enabled
#      Remove this item when it is no longer applied: Enabled
#      Item-level targeting: Enabled if required
#
# Printer preference:
# 14. Go to:
#      Control Panel Settings
#      > Printers
# 15. Right-click Printers.
# 16. Select:
#      New > Shared Printer
# 17. General tab:
#      Action: Update
#      Share path: \\PRINT1\HP-LaserJet-01
#      Set this printer as the default printer: Optional
# 18. Common tab:
#      Run in logged-on user's security context: Enabled
#      Remove this item when it is no longer applied: Enabled
#      Item-level targeting: Enabled if required
#
# Shortcut preference:
# 19. Go to:
#      Windows Settings
#      > Shortcuts
# 20. Right-click Shortcuts.
# 21. Select:
#      New > Shortcut
# 22. General tab:
#      Action: Update
#      Name: Helpdesk Portal
#      Target type: URL
#      Location: Desktop
#      Target URL: https://helpdesk.corp.local
# 23. Common tab:
#      Remove this item when it is no longer applied: Enabled
#
# Registry preference:
# 24. Go to:
#      Windows Settings
#      > Registry
# 25. Right-click Registry.
# 26. Select:
#      New > Registry Item
# 27. General tab:
#      Action: Update
#      Hive: HKEY_CURRENT_USER
#      Key Path: Software\Corp\Preferences
#      Value name: GPP_Applied
#      Value type: REG_SZ
#      Value data: True
#
# Close the editor.
# Refresh GPMC.
# Export a GPO report.
```

# 08_Configure_Group_Policy_Preferences_GPMC_Computer_Preferences_Skeleton
```powershell
# Native GUI workflow for configuring computer Group Policy Preferences.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click the target GPO.
# 7. Select Edit.
# 8. Go to:
#      Computer Configuration
#      > Preferences
#
# Folder preference:
# 9. Go to:
#      Windows Settings
#      > Folders
# 10. Right-click Folders.
# 11. Select:
#      New > Folder
# 12. General tab:
#      Action: Update
#      Path: C:\Corp
#
# File preference:
# 13. Go to:
#      Windows Settings
#      > Files
# 14. Right-click Files.
# 15. Select:
#      New > File
# 16. General tab:
#      Action: Update
#      Source file: \\corp.local\NETLOGON\Scripts\sample.txt
#      Destination file: C:\Corp\sample.txt
#
# Registry preference:
# 17. Go to:
#      Windows Settings
#      > Registry
# 18. Right-click Registry.
# 19. Select:
#      New > Registry Item
# 20. General tab:
#      Action: Update
#      Hive: HKEY_LOCAL_MACHINE
#      Key Path: Software\Corp\Preferences
#      Value name: ComputerGPP_Applied
#      Value type: REG_SZ
#      Value data: True
#
# Scheduled task preference:
# 21. Go to:
#      Control Panel Settings
#      > Scheduled Tasks
# 22. Right-click Scheduled Tasks.
# 23. Select:
#      New > Scheduled Task
# 24. Configure task only with a safe lab action.
# 25. Avoid storing passwords in preference items.
#
# Close the editor.
# Refresh GPMC.
# Export a GPO report.
```

# 08_Configure_Group_Policy_Preferences_Item_Level_Targeting_Skeleton
```powershell
# Native GUI workflow for item-level targeting.
# Run on the management host inside a preference item.

gpmc.msc

# Item-level targeting workflow:
# 1. Open the target GPO in Group Policy Management Editor.
# 2. Open the preference item.
# 3. Select the Common tab.
# 4. Check:
#      Item-level targeting
# 5. Click:
#      Targeting
# 6. Add a condition.
#
# Common targeting examples:
#
# Security Group targeting:
# 7. New Item > Security Group
# 8. Group:
#      GG_GPO_User_Preferences_Apply
# 9. User in group: Enabled for user preferences.
# 10. Computer in group: Enabled for computer preferences.
#
# OU targeting:
# 11. New Item > Organizational Unit
# 12. Select the target OU DN.
#
# Computer Name targeting:
# 13. New Item > Computer Name
# 14. NetBIOS computer name:
#      WIN11-01
#
# Operating System targeting:
# 15. New Item > Operating System
# 16. Product:
#      Windows 11
#
# IP Address Range targeting:
# 17. New Item > IP Address Range
# 18. Range:
#      10.10.10.1 through 10.10.10.254
#
# Notes:
# - Keep targeting simple.
# - Prefer one or two conditions per item.
# - If targeting becomes complicated, create a clearer OU or security group design.
# - Test with one positive and one negative client or user.
```

# 08_Configure_Group_Policy_Preferences_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client while signed in as the test user.
# Purpose: prove Group Policy Preferences applied correctly.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm user and client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Force policy refresh.
gpupdate /force

# Show applied user GPOs.
gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-gpp.txt"

# Show applied computer GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-gpp.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-gpp.html"

# Optional PowerShell RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-gpp.html" `
  -ErrorAction SilentlyContinue

# Validate mapped drive.
Get-PSDrive `
  -Name S `
  -ErrorAction SilentlyContinue |
  Format-List

# Validate printer connection.
Get-Printer |
  Where-Object {
    $_.Name -like "*HP*" -or
    $_.Name -like "*LaserJet*" -or
    $_.ShareName -like "*HP*"
  } |
  Format-Table Name,ShareName,PortName,DriverName -AutoSize

# Validate desktop shortcut.
Test-Path "$env:USERPROFILE\Desktop\Helpdesk Portal.lnk"

# Validate user registry preference.
Get-ItemProperty `
  -Path "HKCU:\Software\Corp\Preferences" `
  -ErrorAction SilentlyContinue

# Validate computer registry preference if configured.
Get-ItemProperty `
  -Path "HKLM:\Software\Corp\Preferences" `
  -ErrorAction SilentlyContinue

# Validate folder or file preference if configured.
Test-Path "C:\Corp"
Test-Path "C:\Corp\sample.txt"

# Review Group Policy preference processing events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 120 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-gpp.txt"
```

# 08_Configure_Group_Policy_Preferences_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final GPO report, inheritance, permission, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"
$GpoName = "CORP-User-Preferences"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\inheritance-after-gpp.txt"

# Capture final permissions.
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

# Search XML report for preference content.
Select-String `
  -Path "$ReportPath\$GpoName-final.xml" `
  -Pattern "Drive","Printer","Shortcut","Registry","Files","Folders","ScheduledTasks","GPP_Applied","Helpdesk Portal" |
  Out-File "$ReportPath\$GpoName-gpp-report-search.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-gpp.csv" -NoTypeInformation

Write-Host "Group Policy Preferences report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 08_Configure_Group_Policy_Preferences_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<gpp-gpo-name>"` | Confirms GPP GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<target-user-ou-dn>"` | Confirms GPO is linked to user OU | GPO appears in link list |
| `Get-GPPermission -Name "<gpp-gpo-name>" -All` | Confirms user or group can read and apply GPO | Expected permissions appear |
| `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | Confirms user is in target OU scope | User DN is under target user OU |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms SYSVOL is reachable | Returns True |
| `gpmc.msc` | Opens GPMC for GPP editing | Console opens |
| `Get-GPOReport -Name "<gpp-gpo-name>" -ReportType Html -Path "<report-path>\<gpp-gpo-name>.html"` | Exports readable GPO report | HTML report exists |
| `Get-GPOReport -Name "<gpp-gpo-name>" -ReportType Xml -Path "<report-path>\<gpp-gpo-name>.xml"` | Exports searchable GPO report | XML report exists |
| `Select-String -Path "<report-path>\<gpp-gpo-name>.xml" -Pattern "Drive","Printer","Shortcut","Registry"` | Searches report for preference items | Preference item entries appear |
| `Backup-GPO -Name "<gpp-gpo-name>" -Path "<backup-path>"` | Backs up configured GPO | Backup completes |
| `whoami` | Confirms test user session | Expected domain user appears |
| `gpupdate /force` | Forces user and computer policy refresh | Policy update completes |
| `gpresult /scope user /r` | Shows applied user GPOs | GPP GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-gpp.html` | Exports full client RSOP report | HTML report exists |
| `Get-PSDrive -Name S` | Validates mapped drive | `S:` drive exists |
| `Get-Printer` | Validates printer preference | Shared printer appears |
| `Test-Path "$env:USERPROFILE\Desktop\Helpdesk Portal.lnk"` | Validates shortcut preference | Shortcut exists |
| `Get-ItemProperty -Path "HKCU:\Software\Corp\Preferences"` | Validates user registry preference | GPP value exists |
| `Test-Path "C:\Corp"` | Validates computer folder preference | Folder exists |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 120` | Reviews client processing events | Recent policy processing events appear |

# 08_Configure_Group_Policy_Preferences_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: rollback Group Policy Preferences by disabling link, deleting preference items in GPMC, or removing the GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-User-Preferences"

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
Set-GPLink `
  -Name $GpoName `
  -Target $TargetUserOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Use GPMC to delete individual preference items.
# This is the safest way to remove specific drive map, printer, shortcut, file, folder,
# registry, or scheduled task preference items without deleting the whole GPO.

gpmc.msc

# GUI rollback workflow:
# 1. Open the target GPO in Group Policy Management Editor.
# 2. Go to User Configuration or Computer Configuration > Preferences.
# 3. Open the preference category.
# 4. Delete or disable the specific preference item.
# 5. For drive maps, printers, and shortcuts, confirm whether:
#      Remove this item when it is no longer applied
#    was enabled before rollback.
# 6. Export a fresh GPO report.
# 7. Run gpupdate /force on the client.
# 8. Validate the item is removed.

# Rollback option 3:
# Remove the GPO entirely if it was created only for this lab.
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
  Out-File "$ReportPath\inheritance-after-gpp-rollback.txt"

Write-Host "Rollback action complete. Run gpupdate /force on the test client and validate preference items."
```

# 08_Configure_Group_Policy_Preferences_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| GPP GPO does not apply | User object is not under linked OU | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | Move user to target OU or link GPO to correct user OU |
| Drive map does not appear | Path unavailable, user lacks share permissions, item-level targeting failed, or user context option missing | `Test-Path \\FS1\Shared`; `gpresult /r` | Fix share/NTFS permissions, targeting, or Common tab settings |
| Drive map appears for wrong users | GPO linked too broadly or targeting too broad | `Get-GPInheritance`; GPO report | Narrow link scope, security filtering, or item-level targeting |
| Printer does not connect | Print server unreachable, driver issue, permissions issue, or targeting failed | `Test-Connection PRINT1`; `Get-Printer` | Fix print server access, driver, permissions, or targeting |
| Shortcut does not appear | Wrong shortcut location, targeting failed, or item did not process | `Test-Path "$env:USERPROFILE\Desktop\<shortcut>.lnk"` | Correct shortcut location and Common tab options |
| Registry preference missing | Wrong hive, wrong side, targeting failed, or user did not refresh | `Get-ItemProperty <path>`; `gpresult /r` | Confirm User vs Computer side and rerun `gpupdate /force` |
| File preference fails | Source path unavailable or permissions issue | `Test-Path <source>`; `Test-Path <destination>` | Fix source path and permissions |
| Folder preference fails | Destination permission issue | `Test-Path <folder>` | Fix NTFS permissions or use computer-side context |
| Preference item stays after GPO no longer applies | Tattooing or Remove item option not enabled | GPO item Common tab | Enable removal option before removing scope or manually delete item |
| Preference applies repeatedly and overwrites user choice | Action is Replace or Apply once not enabled | GPO report | Use Update or Apply once if appropriate |
| User context item fails | Preference needs user permission but runs under wrong context | GPO item Common tab | Enable Run in logged-on user's security context |
| Computer preference fails on user path | Computer-side preference cannot resolve user-specific path as expected | GPO report; event logs | Move item to User Configuration or use correct variables |
| Item-level targeting does not match | Wrong group, stale Kerberos token, wrong condition type | `whoami /groups`; `gpresult /r` | Correct ILT rule and refresh token or log off/on |
| New group membership not reflected | User token stale | `whoami /groups`; `klist` | Sign out and sign back in |
| Computer group membership not reflected | Computer token stale | `gpresult /scope computer /r` | Reboot computer |
| GPO report does not show preference item | Wrong GPO edited or stale report | `Get-GPOReport -Name "<gpp-gpo-name>"` | Edit correct GPO and export fresh report |
| GPP item processing error appears in event log | Bad path, permissions, targeting, or item configuration | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational"` | Fix item configuration based on event detail |
| SYSVOL unavailable | DNS, SMB, or DC issue | `Test-Path "\\<domain>\SYSVOL"`; `dcdiag /test:sysvolcheck` | Fix DNS, SMB, or SYSVOL |
| Preference differs between clients | Replication, targeting, permissions, or client refresh difference | `repadmin /replsummary`; `gpresult /h` | Fix replication or targeting and refresh policy |
| Legacy password preference is present | Unsafe deprecated GPP password storage pattern | Search SYSVOL for `cpassword` | Remove legacy password preference and rotate exposed credentials |

# 08_Configure_Group_Policy_Preferences_Related_Labs
| Related Lab                                                           | Relationship                                                                 |
| --------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `00_Group_Policy_Index.md`                                            | Defines suite order and dependency path                                      |
| `01_Install_Group_Policy_Management_Tools.md`                         | Provides GPMC and GroupPolicy PowerShell module                              |
| `02_Create_And_Link_Baseline_GPO.md`                                  | Establishes GPO creation and linking workflow                                |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before adding preferences                                |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md`               | Controls which users or computers can apply this GPO                         |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md`                     | Adds optional WMI targeting before preference application                    |
| `06_Configure_Computer_Administrative_Template_Settings.md`           | Computer-side policy companion workbook                                      |
| `07_Configure_User_Administrative_Template_Settings.md`               | User-side policy companion workbook                                          |
| `Create_Baseline_OU_Structure.md`                                     | Provides OU scope for preference GPO links                                   |
| `Create_Baseline_Groups_And_Test_Users.md`                            | Provides test users, groups, and filtering principals                        |
| `Join_Windows_Client_To_Domain.md`                                    | Provides domain-joined test client for user logon and validation             |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md`                 | Confirms SYSVOL and DC locator before troubleshooting GPP                    |
| `09_Configure_Scripts_Startup_Shutdown_Logon_And_Logoff.md`           | Next workbook for script-based configuration outside normal preference items |