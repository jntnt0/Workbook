18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers.md
# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Index
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers.md
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Source_Basis
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Mental_Model
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Planning_Table
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Mode_Map
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Configuration_Checklist
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Precheck_Skeleton
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Create_RDS_Kiosk_OU_And_GPO_Skeleton
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Enable_Loopback_GPMC_Skeleton
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Enable_Loopback_PowerShell_Skeleton
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Configure_Computer_Scoped_User_Settings_Skeleton
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Client_RSOP_Validation_Skeleton
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Report_And_Backup_Skeleton
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Verification_Commands
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Rollback
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Failure_Checks
18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Related_Labs

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy loopback processing | Configuring user policy based on the computer used to sign in |
| Microsoft Learn | Administrative Templates System Group Policy settings | Enabling loopback processing mode |
| Microsoft Learn | Group Policy Management Console | Configuring loopback through GPMC |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up GPOs |
| Microsoft Learn | `Set-GPRegistryValue` | Enabling loopback with known registry policy values |
| Microsoft Learn | `gpresult` and RSoP | Validating loopback Merge or Replace effective policy |
| Microsoft Learn | Remote Desktop Services policy settings | RDS session host user environment control |
| Windows Server operational practice | Use loopback only on dedicated shared, kiosk, lab, or RDS computer OUs | Preventing accidental user policy override across normal workstations |

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Loopback processing | Computer-side policy setting that changes how user policy is built when users sign in to specific computers |
| Normal user policy | User settings are based on the user account location in AD |
| Loopback user policy | User settings can be based on the computer account location in AD |
| Merge mode | User policy from the user OU applies first, then user settings linked to the computer OU apply after it |
| Replace mode | User policy from the user OU is ignored and user settings linked to the computer OU are used instead |
| RDS computer | Remote Desktop Session Host where many users sign in to the same server |
| Kiosk computer | Locked-down computer where user experience should be based on the device role |
| Shared computer | Multi-user endpoint where consistent user experience is more important than normal user OU policy |
| Computer-scoped user settings | User Configuration settings placed in GPOs linked to the computer OU when loopback is enabled |
| Loopback GPO | Computer-side GPO that enables loopback mode |
| User restriction GPO | GPO containing User Configuration settings intended for RDS, kiosk, or shared computers |
| Computer OU | OU containing the RDS, kiosk, lab, or shared computers |
| User OU | OU containing users who may sign in to many different computers |
| GPO precedence | Merge mode creates another layer of user policy precedence from the computer OU |
| First rule | Loopback belongs on computer OUs, not normal user OUs |
| Blunt rule | Replace mode can make users lose their normal user GPOs on loopback computers, so pilot before broad deployment |

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target computer role | RDS, Kiosk, Lab, Shared Workstation | `<computer-role>` |
| Loopback computer OU | `OU=RDS,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<loopback-computer-ou-dn>` |
| Test computer | `RDS01` or `KIOSK01` | `<test-computer>` |
| Test user | `tuser` | `<test-user>` |
| Test user OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<test-user-ou-dn>` |
| Loopback GPO | `CORP-RDS-Kiosk-Loopback-Processing` | `<loopback-gpo-name>` |
| User restrictions GPO | `CORP-RDS-Kiosk-User-Experience` | `<user-settings-gpo-name>` |
| Loopback mode | Merge or Replace | `<merge-or-replace>` |
| Security filtering group | `GG_GPO_RDS_Kiosk_Loopback_Apply` | `<filtering-group>` |
| Computer apply group | `GG_RDS_Kiosk_Computers` | `<computer-group>` |
| User settings apply group | `GG_RDS_Kiosk_Users` | `<user-group>` |
| Link enabled | Yes | `<yes-no>` |
| Enforced | No | `<yes-no>` |
| Block inheritance | No by default | `<yes-no>` |
| User setting examples | Hide Control Panel, remove Run, drive map, printer, shortcut | `<user-setting-list>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Disable loopback GPO link, set loopback to Not Configured, or restore backup | `<rollback-plan>` |

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Mode_Map
| Mode | Registry Value | Processing Behavior | Best Use | Risk |
|---|---:|---|---|---|
| Not Configured | Absent | Normal user policy based on user OU | Standard workstations | No loopback behavior |
| Merge | `1` | User OU policy applies, then computer OU user settings apply after it | Shared computers where normal user policy should still apply | Conflicting user settings may be overridden by computer OU user settings |
| Replace | `2` | User OU policy is ignored and computer OU user settings are used | Kiosk, RDS, lab, locked shared systems | Users may lose expected normal user policies on those computers |
| RDS design | Usually Merge or Replace | User environment follows RDS server OU | Session hosts | Wrong mode can over-restrict users |
| Kiosk design | Usually Replace | User environment follows kiosk computer OU | Locked-down terminals | Easy to lock out normal user experience |
| Shared lab design | Usually Merge | User keeps baseline policy plus lab-specific settings | Labs and training rooms | User settings may conflict |
| Normal workstation design | Not Configured | User settings follow user OU | Regular endpoints | Loopback should normally not be used |

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 6 | Confirm GPMC opens | Management Host | `gpmc.msc` | GPMC opens |
| 7 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 8 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 9 | Back up current GPO state | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 10 | Confirm loopback computer OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<loopback-computer-ou-dn>"` | OU returns |
| 11 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 12 | Confirm test computer is in loopback OU | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under loopback OU |
| 13 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled` | Test user returns |
| 14 | Create loopback GPO | Management Host | `New-GPO -Name "<loopback-gpo-name>" -Comment "Loopback processing for RDS, kiosk, and shared computers"` | Loopback GPO exists |
| 15 | Create user settings GPO | Management Host | `New-GPO -Name "<user-settings-gpo-name>" -Comment "User experience settings for loopback computers"` | User settings GPO exists |
| 16 | Link loopback GPO to computer OU | Management Host | `New-GPLink -Name "<loopback-gpo-name>" -Target "<loopback-computer-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 17 | Link user settings GPO to computer OU | Management Host | `New-GPLink -Name "<user-settings-gpo-name>" -Target "<loopback-computer-ou-dn>" -LinkEnabled Yes` | User settings GPO is linked to computer OU |
| 18 | Keep links unenforced by default | Management Host | `Set-GPLink -Name "<gpo-name>" -Target "<loopback-computer-ou-dn>" -Enforced No` | Links are not enforced |
| 19 | Confirm inheritance state | Management Host | `Get-GPInheritance -Target "<loopback-computer-ou-dn>"` | Both GPOs appear in link list |
| 20 | Configure loopback mode in GPMC | Management Host | `Computer Configuration > Policies > Administrative Templates > System > Group Policy > Configure user Group Policy loopback processing mode` | Loopback mode is configured |
| 21 | Configure loopback mode with PowerShell if preferred | Management Host | `Set-GPRegistryValue -Name "<loopback-gpo-name>" -Key "HKLM\Software\Policies\Microsoft\Windows\System" -ValueName "UserPolicyMode" -Type DWord -Value 1` | Merge mode is configured |
| 22 | Configure user restrictions in user settings GPO | Management Host | `User Configuration > Policies > Administrative Templates` | User settings are configured |
| 23 | Configure GPP user resources if needed | Management Host | `User Configuration > Preferences` | Drive maps, printers, shortcuts, or registry preferences are configured |
| 24 | Export loopback GPO report | Management Host | `Get-GPOReport -Name "<loopback-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<loopback-gpo-name>.html` | Report shows loopback mode |
| 25 | Export user settings GPO report | Management Host | `Get-GPOReport -Name "<user-settings-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<user-settings-gpo-name>.html` | Report shows user settings |
| 26 | Back up configured GPOs | Management Host | `Backup-GPO -Name "<loopback-gpo-name>" -Path C:\GPOPrep\Backup; Backup-GPO -Name "<user-settings-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backups exist |
| 27 | Sign in to loopback computer as test user | Test Computer | `whoami` | Test user session is active |
| 28 | Force policy refresh | Test Computer | `gpupdate /force` | Policy refresh completes |
| 29 | Reboot if computer-side loopback setting is new | Test Computer | `Restart-Computer` | Computer restarts |
| 30 | Sign back in as test user | Test Computer | `whoami` | New user session starts |
| 31 | Validate computer policy | Test Computer | `gpresult /scope computer /r` | Loopback GPO appears under Applied GPOs |
| 32 | Validate user policy | Test Computer | `gpresult /scope user /r` | User settings GPO linked to computer OU appears under Applied User GPOs |
| 33 | Export GPResult report | Test Computer | `gpresult /h C:\GPOPrep\Reports\gpresult-loopback.html` | HTML RSOP report exists |
| 34 | Validate loopback registry | Test Computer | `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\System"` | `UserPolicyMode` appears |
| 35 | Validate user setting behavior | Test Computer | `Check Control Panel, Run menu, mapped drives, printers, shortcuts, or registry values` | Expected user experience applies on loopback computer |
| 36 | Compare with non-loopback workstation | Standard Client | `gpresult /scope user /r` | Loopback user settings do not apply on normal workstation |
| 37 | Review Group Policy events | Test Computer | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Processing events are visible |
| 38 | Document result | Operator | `Record mode, target OU, linked GPOs, test computer, test user, reports, and final behavior` | Loopback configuration is documented |

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate OU, computer, user, and GPO readiness before enabling loopback.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$LoopbackComputerOU = "OU=RDS,OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "RDS01"
$TestUser = "tuser"

$LoopbackGpoName = "CORP-RDS-Kiosk-Loopback-Processing"
$UserSettingsGpoName = "CORP-RDS-Kiosk-User-Experience"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports\Loopback-Processing"
$BackupPath = "$BasePath\Backup\Loopback-Processing"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm domain.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName,PDCEmulator |
  Out-File "$ReportPath\domain-info.txt"

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

# Confirm OU exists.
Get-ADOrganizationalUnit `
  -Identity $LoopbackComputerOU |
  Out-File "$ReportPath\loopback-computer-ou.txt"

# Confirm test computer and user.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,Enabled,DistinguishedName,MemberOf |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-computer-before-loopback.txt"

Get-ADUser `
  -Identity $TestUser `
  -Properties UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Select-Object SamAccountName,UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-user-before-loopback.txt"

# Capture current inheritance on loopback OU.
Get-GPInheritance `
  -Target $LoopbackComputerOU |
  Out-File "$ReportPath\loopback-ou-inheritance-before.txt"

# Capture GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-loopback.csv" -NoTypeInformation

# Backup current GPOs before loopback work.
Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre-loopback configuration backup"

Write-Host "Loopback precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Create_RDS_Kiosk_OU_And_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create or confirm the loopback computer OU and the two GPOs used for loopback design.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$ParentOU = "OU=Workstations,OU=Corp,$DomainDN"
$LoopbackOUName = "RDS"
$LoopbackComputerOU = "OU=$LoopbackOUName,$ParentOU"

$TestComputer = "RDS01"

$LoopbackGpoName = "CORP-RDS-Kiosk-Loopback-Processing"
$UserSettingsGpoName = "CORP-RDS-Kiosk-User-Experience"

# Create loopback computer OU if missing.
if (-not (Get-ADOrganizationalUnit -LDAPFilter "(distinguishedName=$LoopbackComputerOU)" -ErrorAction SilentlyContinue)) {
    New-ADOrganizationalUnit `
      -Name $LoopbackOUName `
      -Path $ParentOU `
      -ProtectedFromAccidentalDeletion $true
}

# Optional: move test computer into loopback OU.
# Move-ADObject `
#   -Identity (Get-ADComputer -Identity $TestComputer).DistinguishedName `
#   -TargetPath $LoopbackComputerOU

# Create loopback GPO if missing.
if (-not (Get-GPO -Name $LoopbackGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $LoopbackGpoName `
      -Comment "Enables Group Policy loopback processing for RDS, kiosk, and shared computers."
}

# Create user settings GPO if missing.
if (-not (Get-GPO -Name $UserSettingsGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $UserSettingsGpoName `
      -Comment "User Configuration settings that apply based on loopback computer OU."
}

# Link loopback GPO to loopback computer OU.
$Inheritance = Get-GPInheritance -Target $LoopbackComputerOU

if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $LoopbackGpoName)) {
    New-GPLink `
      -Name $LoopbackGpoName `
      -Target $LoopbackComputerOU `
      -LinkEnabled Yes
}

# Link user settings GPO to loopback computer OU.
$Inheritance = Get-GPInheritance -Target $LoopbackComputerOU

if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $UserSettingsGpoName)) {
    New-GPLink `
      -Name $UserSettingsGpoName `
      -Target $LoopbackComputerOU `
      -LinkEnabled Yes
}

# Safe defaults.
Set-GPLink `
  -Name $LoopbackGpoName `
  -Target $LoopbackComputerOU `
  -LinkEnabled Yes `
  -Enforced No

Set-GPLink `
  -Name $UserSettingsGpoName `
  -Target $LoopbackComputerOU `
  -LinkEnabled Yes `
  -Enforced No

# Confirm.
Get-ADOrganizationalUnit -Identity $LoopbackComputerOU
Get-GPInheritance -Target $LoopbackComputerOU
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Enable_Loopback_GPMC_Skeleton
```powershell
# Native GUI workflow for enabling loopback processing.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-RDS-Kiosk-Loopback-Processing
# 7. Select Edit.
# 8. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > System
#      > Group Policy
# 9. Open:
#      Configure user Group Policy loopback processing mode
# 10. Set:
#      Enabled
# 11. Select mode:
#      Merge
#      or
#      Replace
#
# Mode choice:
# - Merge:
#     User's normal user GPOs apply first.
#     User Configuration settings linked to the computer OU apply after.
# - Replace:
#     User's normal user GPOs are ignored on this computer.
#     User Configuration settings linked to the computer OU become the user policy set.
#
# Recommendation:
# - Use Merge for shared lab computers where normal user baseline still matters.
# - Use Replace for kiosk or tightly controlled RDS systems only after pilot testing.
#
# Close editor.
# Refresh GPMC.
# Export the GPO report.
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Enable_Loopback_PowerShell_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: configure loopback processing using the known registry policy value.

Import-Module GroupPolicy

$LoopbackGpoName = "CORP-RDS-Kiosk-Loopback-Processing"

$ReportPath = "C:\GPOPrep\Reports\Loopback-Processing"
$BackupPath = "C:\GPOPrep\Backup\Loopback-Processing"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm GPO exists.
Get-GPO -Name $LoopbackGpoName

# Back up before change.
Backup-GPO `
  -Name $LoopbackGpoName `
  -Path $BackupPath `
  -Comment "Before enabling loopback processing"

# Loopback registry policy:
# Key:   HKLM\Software\Policies\Microsoft\Windows\System
# Value: UserPolicyMode
# Data:  1 = Merge, 2 = Replace

$LoopbackMode = "Merge"

switch ($LoopbackMode) {
    "Merge"   { $LoopbackValue = 1 }
    "Replace" { $LoopbackValue = 2 }
    default   { throw "LoopbackMode must be Merge or Replace." }
}

Set-GPRegistryValue `
  -Name $LoopbackGpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\System" `
  -ValueName "UserPolicyMode" `
  -Type DWord `
  -Value $LoopbackValue

# Export report.
Get-GPOReport `
  -Name $LoopbackGpoName `
  -ReportType Html `
  -Path "$ReportPath\$LoopbackGpoName-loopback-$LoopbackMode.html"

Get-GPOReport `
  -Name $LoopbackGpoName `
  -ReportType Xml `
  -Path "$ReportPath\$LoopbackGpoName-loopback-$LoopbackMode.xml"

# Back up after change.
Backup-GPO `
  -Name $LoopbackGpoName `
  -Path $BackupPath `
  -Comment "After enabling loopback processing mode $LoopbackMode"

Write-Host "Loopback processing configured."
Write-Host "Mode: $LoopbackMode"
Write-Host "Value: $LoopbackValue"
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Configure_Computer_Scoped_User_Settings_Skeleton
```powershell
# Native GUI workflow for configuring User Configuration settings that apply through loopback.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-RDS-Kiosk-User-Experience
# 3. Configure user-side settings in this GPO.
# 4. Confirm this GPO is linked to the computer OU where loopback is enabled:
#      OU=RDS,OU=Workstations,OU=Corp,DC=corp,DC=local
#
# Example User Administrative Template settings:
# 5. Go to:
#      User Configuration
#      > Policies
#      > Administrative Templates
#
# Example setting 1:
# 6. Control Panel
#      > Prohibit access to Control Panel and PC settings
#      Enabled
#
# Example setting 2:
# 7. Start Menu and Taskbar
#      > Remove Run menu from Start Menu
#      Enabled
#
# Example setting 3:
# 8. System
#      > Prevent access to the command prompt
#      Enabled
#
# Example Group Policy Preferences:
# 9. Go to:
#      User Configuration
#      > Preferences
#
# Example drive map:
# 10. Windows Settings
#      > Drive Maps
#      New > Mapped Drive
#      Action: Update
#      Location: \\FS1\RDSShared
#      Drive Letter: R:
#
# Example shortcut:
# 11. Windows Settings
#      > Shortcuts
#      New > Shortcut
#      Action: Update
#      Name: RDS Helpdesk Portal
#      Target type: URL
#      Location: Desktop
#      Target URL: https://helpdesk.corp.local
#
# Example printer:
# 12. Control Panel Settings
#      > Printers
#      New > Shared Printer
#      Action: Update
#      Share path: \\PRINT1\RDS-Printer
#
# Notes:
# - These are User Configuration settings.
# - They apply because the GPO is linked to the computer OU and loopback is enabled.
# - Validate with gpresult /scope user /r while signed in to the loopback computer.
```

```powershell
# Optional PowerShell example for user registry policy settings inside the loopback user settings GPO.
# Run on management host.

Import-Module GroupPolicy

$UserSettingsGpoName = "CORP-RDS-Kiosk-User-Experience"

# Example: Prohibit access to Control Panel and PC settings.
Set-GPRegistryValue `
  -Name $UserSettingsGpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoControlPanel" `
  -Type DWord `
  -Value 1

# Example: Remove Run menu from Start Menu.
Set-GPRegistryValue `
  -Name $UserSettingsGpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoRun" `
  -Type DWord `
  -Value 1

# Example: Prevent access to command prompt.
Set-GPRegistryValue `
  -Name $UserSettingsGpoName `
  -Key "HKCU\Software\Policies\Microsoft\Windows\System" `
  -ValueName "DisableCMD" `
  -Type DWord `
  -Value 1

# Export report.
Get-GPOReport `
  -Name $UserSettingsGpoName `
  -ReportType Html `
  -Path "C:\GPOPrep\Reports\Loopback-Processing\$UserSettingsGpoName-user-settings.html"
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Client_RSOP_Validation_Skeleton
```powershell
# Run on the RDS, kiosk, or shared computer while signed in as the test user.
# Purpose: prove loopback mode applies user settings from the computer OU.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\Loopback-Processing-Client"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm identity and domain state.
hostname |
  Out-File "$ReportPath\hostname.txt"

whoami /all |
  Out-File "$ReportPath\whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$ReportPath\computer-info.txt"

# Confirm DC locator and SYSVOL access.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

# Force policy refresh.
gpupdate /force |
  Tee-Object "$ReportPath\gpupdate-force.txt"

# Reboot if loopback was newly configured.
# Restart-Computer

# After reboot, sign back in as the test user and continue.

# Confirm computer-side applied GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-r.txt"

# Confirm user-side applied GPOs.
gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-r.txt"

# Export full reports.
gpresult /h "$ReportPath\gpresult-loopback-full.html"
gpresult /x "$ReportPath\gpresult-loopback-full.xml"
gpresult /z > "$ReportPath\gpresult-loopback-verbose.txt"

# Validate loopback registry policy.
Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\System" `
  -ErrorAction SilentlyContinue |
  Select-Object UserPolicyMode |
  Out-File "$ReportPath\loopback-registry-policy.txt"

# Validate example user settings.
Get-ItemProperty `
  -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ErrorAction SilentlyContinue |
  Select-Object NoControlPanel,NoRun |
  Out-File "$ReportPath\user-explorer-policy-values.txt"

Get-ItemProperty `
  -Path "HKCU:\Software\Policies\Microsoft\Windows\System" `
  -ErrorAction SilentlyContinue |
  Select-Object DisableCMD |
  Out-File "$ReportPath\user-system-policy-values.txt"

# Search gpresult for loopback-related GPO names.
Select-String `
  -Path "$ReportPath\gpresult-loopback-verbose.txt" `
  -Pattern "Loopback","CORP-RDS-Kiosk-Loopback-Processing","CORP-RDS-Kiosk-User-Experience","Applied Group Policy Objects","Denied GPOs" |
  Out-File "$ReportPath\gpresult-loopback-search.txt"

# Review Group Policy operational log.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 200 |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-events.txt"

Write-Host "Loopback client validation complete."
Write-Host "Report path: $ReportPath"
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final loopback GPO reports, inheritance, permissions, and backups.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$LoopbackComputerOU = "OU=RDS,OU=Workstations,OU=Corp,$DomainDN"

$LoopbackGpoName = "CORP-RDS-Kiosk-Loopback-Processing"
$UserSettingsGpoName = "CORP-RDS-Kiosk-User-Experience"

$ReportPath = "C:\GPOPrep\Reports\Loopback-Processing"
$BackupPath = "C:\GPOPrep\Backup\Loopback-Processing"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture inheritance.
Get-GPInheritance `
  -Target $LoopbackComputerOU |
  Out-File "$ReportPath\loopback-computer-ou-inheritance-final.txt"

# Capture permissions.
foreach ($GpoName in @($LoopbackGpoName,$UserSettingsGpoName)) {
    Get-GPPermission `
      -Name $GpoName `
      -All |
      Out-File "$ReportPath\$GpoName-permissions-final.txt"

    Get-GPOReport `
      -Name $GpoName `
      -ReportType Html `
      -Path "$ReportPath\$GpoName-final.html"

    Get-GPOReport `
      -Name $GpoName `
      -ReportType Xml `
      -Path "$ReportPath\$GpoName-final.xml"

    Backup-GPO `
      -Name $GpoName `
      -Path $BackupPath `
      -Comment "Final backup after loopback processing configuration"
}

# Search reports for loopback and user settings.
Select-String `
  -Path "$ReportPath\*.xml" `
  -Pattern "UserPolicyMode","Loopback","NoControlPanel","NoRun","DisableCMD","Drive","Printer","Shortcut" |
  Out-File "$ReportPath\loopback-report-search.txt"

# Replication health check after changes.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-loopback.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-loopback.txt"

Write-Host "Loopback reports and backups complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<loopback-gpo-name>"` | Confirms loopback GPO exists | GPO object returns |
| `Get-GPO -Name "<user-settings-gpo-name>"` | Confirms loopback user settings GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<loopback-computer-ou-dn>"` | Confirms both GPOs are linked to computer OU | Both GPOs appear |
| `Get-GPPermission -Name "<loopback-gpo-name>" -All` | Confirms computer can apply loopback GPO | Expected permissions appear |
| `Get-GPPermission -Name "<user-settings-gpo-name>" -All` | Confirms user settings GPO can apply | Expected permissions appear |
| `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Confirms computer is in loopback OU | DN is under loopback computer OU |
| `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | Confirms test user exists and normal user OU is known | User DN returns |
| `Get-GPOReport -Name "<loopback-gpo-name>" -ReportType Html -Path "<path>"` | Confirms loopback setting report | Report exists |
| `Get-GPOReport -Name "<user-settings-gpo-name>" -ReportType Html -Path "<path>"` | Confirms user setting report | Report exists |
| `gpupdate /force` | Forces policy refresh | Policy refresh completes |
| `Restart-Computer` | Refreshes computer-side loopback mode after first configuration | Computer restarts |
| `gpresult /scope computer /r` | Shows loopback GPO under applied computer GPOs | Loopback GPO appears |
| `gpresult /scope user /r` | Shows user settings from computer OU | User settings GPO appears |
| `gpresult /h C:\GPOPrep\Reports\gpresult-loopback.html` | Exports full effective policy report | HTML report exists |
| `Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\System"` | Validates loopback registry policy | `UserPolicyMode = 1` or `2` |
| `Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"` | Validates example user restrictions | `NoControlPanel` or `NoRun` appears |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews Group Policy processing | Recent events are visible |
| `repadmin /replsummary` | Confirms AD replication health | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Confirms SYSVOL and DC health | Tests pass |

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely remove or disable loopback processing.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$LoopbackComputerOU = "OU=RDS,OU=Workstations,OU=Corp,$DomainDN"

$LoopbackGpoName = "CORP-RDS-Kiosk-Loopback-Processing"
$UserSettingsGpoName = "CORP-RDS-Kiosk-User-Experience"

$ReportPath = "C:\GPOPrep\Reports\Loopback-Processing-Rollback"
$BackupPath = "C:\GPOPrep\Backup\Loopback-Processing-Rollback"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture state before rollback.
foreach ($GpoName in @($LoopbackGpoName,$UserSettingsGpoName)) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-before-rollback.html"

        Backup-GPO `
          -Name $GpoName `
          -Path $BackupPath `
          -Comment "Before loopback rollback"
    }
}

Get-GPInheritance `
  -Target $LoopbackComputerOU |
  Out-File "$ReportPath\loopback-ou-inheritance-before-rollback.txt"

# Rollback option 1:
# Disable loopback GPO link, preserving the GPO.
Set-GPLink `
  -Name $LoopbackGpoName `
  -Target $LoopbackComputerOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Disable user settings GPO link, preserving the GPO.
Set-GPLink `
  -Name $UserSettingsGpoName `
  -Target $LoopbackComputerOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 3:
# Remove only the loopback registry policy value from the loopback GPO.
Remove-GPRegistryValue `
  -Name $LoopbackGpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\System" `
  -ValueName "UserPolicyMode" `
  -ErrorAction SilentlyContinue

# Rollback option 4:
# Remove example user restrictions from user settings GPO.
Remove-GPRegistryValue `
  -Name $UserSettingsGpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoControlPanel" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $UserSettingsGpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoRun" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $UserSettingsGpoName `
  -Key "HKCU\Software\Policies\Microsoft\Windows\System" `
  -ValueName "DisableCMD" `
  -ErrorAction SilentlyContinue

# Rollback option 5:
# Remove lab GPOs only if they were created for disposable testing.
# Remove-GPO -Name $LoopbackGpoName
# Remove-GPO -Name $UserSettingsGpoName

# Capture after rollback.
Get-GPInheritance `
  -Target $LoopbackComputerOU |
  Out-File "$ReportPath\loopback-ou-inheritance-after-rollback.txt"

foreach ($GpoName in @($LoopbackGpoName,$UserSettingsGpoName)) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-after-rollback.html"
    }
}

Write-Host "Loopback rollback complete."
Write-Host "On the test computer: run gpupdate /force, reboot, sign in again, and validate gpresult."
```

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Loopback does not apply | Computer is not in loopback OU | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Move computer to loopback OU or link GPO to correct OU |
| Loopback GPO missing from computer GPResult | GPO not linked, disabled, filtered, or WMI denied | `gpresult /scope computer /r`; `Get-GPInheritance` | Fix link, filtering, or WMI |
| User settings GPO missing from user GPResult | User settings GPO is not linked to computer OU or loopback not active | `gpresult /scope user /r`; `Get-GPInheritance` | Link user settings GPO to computer OU and enable loopback |
| User settings still follow normal user OU | Loopback not configured or client not rebooted | Registry check for `UserPolicyMode` | Enable loopback and reboot |
| Normal user GPOs disappear unexpectedly | Replace mode was selected | `gpresult /scope user /r` | Use Merge mode or redesign scope |
| Loopback settings override too much | Merge mode precedence or Replace mode too aggressive | `gpresult /h` | Move settings, adjust link order, or change mode |
| User restriction applies on normal workstation | User settings GPO linked to user OU instead of loopback computer OU | `Get-GPInheritance` for user OU and computer OU | Remove wrong link and keep link on loopback computer OU |
| User restriction does not apply on RDS server | RDS server not in loopback OU or user GPO not linked to computer OU | AD computer DN and inheritance | Move server or link GPO properly |
| GPO denied by security filtering | Computer or user lacks Apply permission | `Get-GPPermission`; `gpresult /r` | Add correct computer or user group with `GpoApply` |
| GPO denied by WMI | WMI filter does not match RDS, kiosk, or shared computer | Test WMI query locally | Fix or remove WMI filter |
| Group membership change not reflected | User or computer token stale | `whoami /groups`; `gpresult /scope computer /r` | Sign out user or reboot computer |
| User settings apply only after second logon | Loopback or user shell setting requires new session | `gpupdate /force`; sign out/in | Reboot and sign in again |
| GPP item missing under loopback | Item-level targeting fails | GPO report and Operational log | Fix targeting condition |
| Drive map or printer from loopback missing | User settings GPO applies but resource path or permissions fail | `Test-Path`, `Get-Printer`, event logs | Fix share, printer, or user permissions |
| RDS users lose expected resources | Replace mode removed normal user OU policy | `gpresult /scope user /r` | Use Merge or add required user settings to loopback computer OU |
| Kiosk not locked down enough | Merge mode keeps normal user policy and lacks kiosk restrictions | `gpresult /h` | Use Replace or add stronger loopback user settings |
| GPMC report shows loopback but client does not | Client has not refreshed, wrong DC, or replication delay | `gpupdate`, `nltest`, `repadmin` | Refresh, reboot, and fix replication if needed |
| `UserPolicyMode` missing | Wrong GPO, wrong key, or GPO did not apply | Registry check and GPO report | Correct loopback GPO and scope |
| Loopback changes differ between clients | Different OU placement, DC, replication, or local state | Compare `gpresult /h` reports | Normalize placement and fix replication |
| SYSVOL unavailable | DNS, SMB, DC, or SYSVOL issue | `Test-Path "\\<domain>\SYSVOL"` | Fix SYSVOL and DC locator |
| GPO differs between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before retesting |
| Rollback does not remove loopback behavior | Client needs reboot or another GPO still sets loopback | `gpresult /h`; registry check | Disable winning GPO, refresh, reboot |
| Locked user experience after rollback | Tattooed preferences or user setting still configured elsewhere | `gpresult /h`; registry checks | Remove winning user GPO setting and refresh user session |

# 18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers_Related_Labs
| Related Lab                                                               | Relationship                                                                         |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `00_Group_Policy_Index.md`                                                | Defines suite order and dependency path                                              |
| `01_Install_Group_Policy_Management_Tools.md`                             | Provides GPMC and GroupPolicy PowerShell module                                      |
| `02_Create_And_Link_Baseline_GPO.md`                                      | Establishes baseline GPO creation and linking workflow                               |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md`     | Explains precedence behavior that affects Merge mode                                 |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md`                   | Controls which computers and users can apply loopback GPOs                           |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md`                         | Adds optional OS or role targeting for loopback systems                              |
| `06_Configure_Computer_Administrative_Template_Settings.md`               | Provides computer-side ADMX policy foundation used to enable loopback                |
| `07_Configure_User_Administrative_Template_Settings.md`                   | Provides user-side policy settings used through loopback                             |
| `08_Configure_Group_Policy_Preferences.md`                                | Provides GPP user resources often applied through loopback                           |
| `10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md`    | User resource workbook that can behave differently under loopback                    |
| `14_Backup_Restore_Import_And_Report_GPOs.md`                             | Provides backup and report workflow before loopback rollout                          |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Primary validation method for loopback behavior                                      |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md`              | Diagnoses loopback failures from scope, filtering, replication, or client processing |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md`               | Ensures loopback Administrative Template setting is available in GPMC                |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md`                     | Confirms domain infrastructure required for loopback policy processing               |
| `19_Configure_Starter_GPOs_And_Baseline_Templates.md`                     | Next workbook for reusable baseline GPO templates                                    |