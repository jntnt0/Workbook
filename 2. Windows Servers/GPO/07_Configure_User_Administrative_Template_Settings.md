07_Configure_User_Administrative_Template_Settings.md
# 07_Configure_User_Administrative_Template_Settings

# 07_Configure_User_Administrative_Template_Settings_Index
07_Configure_User_Administrative_Template_Settings.md
07_Configure_User_Administrative_Template_Settings
07_Configure_User_Administrative_Template_Settings_Source_Basis
07_Configure_User_Administrative_Template_Settings_Mental_Model
07_Configure_User_Administrative_Template_Settings_Planning_Table
07_Configure_User_Administrative_Template_Settings_Admin_Template_Settings_Map
07_Configure_User_Administrative_Template_Settings_Configuration_Checklist
07_Configure_User_Administrative_Template_Settings_Precheck_Skeleton
07_Configure_User_Administrative_Template_Settings_Create_And_Link_GPO_Skeleton
07_Configure_User_Administrative_Template_Settings_GPMC_Configuration_Skeleton
07_Configure_User_Administrative_Template_Settings_PowerShell_Registry_Policy_Skeleton
07_Configure_User_Administrative_Template_Settings_Client_RSOP_Validation_Skeleton
07_Configure_User_Administrative_Template_Settings_Report_And_Backup_Skeleton
07_Configure_User_Administrative_Template_Settings_Verification_Commands
07_Configure_User_Administrative_Template_Settings_Rollback
07_Configure_User_Administrative_Template_Settings_Failure_Checks
07_Configure_User_Administrative_Template_Settings_Related_Labs

# 07_Configure_User_Administrative_Template_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy Administrative Templates | ADMX-backed user policy settings |
| Microsoft Learn | Group Policy Management Console | Editing User Configuration Administrative Template settings |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up GPOs |
| Microsoft Learn | `Set-GPRegistryValue` | Configuring known user registry policy values |
| Microsoft Learn | `Remove-GPRegistryValue` | Removing configured user registry policy values during rollback |
| Microsoft Learn | `Get-GPOReport` | Verifying configured Administrative Template settings |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, and GroupPolicy event logs | Proving user Administrative Template settings apply |
| Windows Server operational practice | Separate user policy GPOs from computer policy GPOs | Keeping user experience control supportable and reversible |

# 07_Configure_User_Administrative_Template_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| User Administrative Templates | ADMX-backed policy settings under User Configuration |
| User Configuration | GPO half that applies to user accounts at logon and background refresh |
| HKCU policy | User policy values written under the current user's registry hive |
| User Registry.pol | GPO file that stores registry-based user policy settings in SYSVOL |
| Enabled | Policy state that writes the configured user policy value |
| Disabled | Policy state that writes the opposite policy value when the ADMX supports it |
| Not Configured | Policy state that removes the GPO's management of the setting |
| User OU scope | User account must be in or under the OU where the GPO is linked |
| Security filtering | User or group must have Read and Apply Group Policy permission |
| Loopback processing | Computer-side feature that can change user policy processing on special computers |
| RSOP | Effective user policy result after link order, inheritance, filtering, WMI, and processing |
| GPResult | Client-side command proving applied and denied user GPOs |
| GPMC path | GUI location where the ADMX-backed policy is configured |
| Registry path | Underlying managed registry key and value controlled by the policy |
| First rule | User settings follow the user object, not the computer object, unless loopback is configured |
| Blunt rule | If the test user is not in the linked OU scope, the user GPO will not apply no matter how perfect the setting is |

# 07_Configure_User_Administrative_Template_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target user OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<target-user-ou-dn>` |
| Test user | `tuser` | `<test-user>` |
| Test workstation | `WIN11-01` | `<test-workstation>` |
| Target GPO | `CORP-User-Administrative-Templates` | `<target-gpo-name>` |
| Security filtering group | `GG_GPO_User_AdminTemplates_Apply` | `<filtering-group>` |
| Policy side | User Configuration | `User Configuration` |
| ADMX source | Local store or Central Store | `<admx-source>` |
| Central Store path | `\\corp.local\SYSVOL\corp.local\Policies\PolicyDefinitions` | `<central-store-path>` |
| Setting 1 | Prohibit access to Control Panel and PC settings | `<setting-1>` |
| Setting 2 | Remove Run menu from Start Menu | `<setting-2>` |
| Setting 3 | Prevent access to command prompt | `<setting-3>` |
| Link enabled | Yes | `<yes-no>` |
| Enforced | No | `<yes-no>` |
| Block inheritance | No | `<yes-no>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Remove registry policy values or disable GPO link | `<rollback-plan>` |

# 07_Configure_User_Administrative_Template_Settings_Admin_Template_Settings_Map
| Setting | GPMC Path | Registry Policy Path | Value | Example State | Validation |
|---|---|---|---|---|---|
| Prohibit access to Control Panel and PC settings | `User Configuration > Policies > Administrative Templates > Control Panel` | `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` | `NoControlPanel` DWORD `1` | Enabled | Control Panel and Settings access are blocked for test user |
| Remove Run menu from Start Menu | `User Configuration > Policies > Administrative Templates > Start Menu and Taskbar` | `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` | `NoRun` DWORD `1` | Enabled | Run option is unavailable for test user |
| Prevent access to the command prompt | `User Configuration > Policies > Administrative Templates > System` | `HKCU\Software\Policies\Microsoft\Windows\System` | `DisableCMD` DWORD `1` | Enabled | Command Prompt is blocked for test user |
| Prevent access to registry editing tools | `User Configuration > Policies > Administrative Templates > System` | `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System` | `DisableRegistryTools` DWORD `1` | Optional high impact | Registry Editor is blocked for test user |
| Hide specified Control Panel items | `User Configuration > Policies > Administrative Templates > Control Panel` | ADMX-managed registry policy values | Varies | Optional | GPO report and user behavior confirm result |
| Do not save settings at exit | `User Configuration > Policies > Administrative Templates > Desktop` | `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` | `NoSaveSettings` DWORD `1` | Optional | User shell setting behavior changes after refresh |

# 07_Configure_User_Administrative_Template_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target user OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-user-ou-dn>"` | Target user OU returns |
| 6 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled` | User object returns |
| 7 | Confirm test user is inside target user OU scope | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | DistinguishedName is under target user OU |
| 8 | Confirm test workstation is domain joined | Test Client | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Client is domain joined |
| 9 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Confirm GPMC opens | Management Host | `gpmc.msc` | GPMC opens |
| 11 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 12 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 13 | Back up existing GPOs before changes | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 14 | Create dedicated user Administrative Templates GPO if missing | Management Host | `New-GPO -Name "<target-gpo-name>" -Comment "User Administrative Template settings"` | GPO exists |
| 15 | Link GPO to target user OU | Management Host | `New-GPLink -Name "<target-gpo-name>" -Target "<target-user-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 16 | Keep link unenforced by default | Management Host | `Set-GPLink -Name "<target-gpo-name>" -Target "<target-user-ou-dn>" -Enforced No` | Link is not enforced |
| 17 | Confirm link state | Management Host | `Get-GPInheritance -Target "<target-user-ou-dn>"` | GPO appears in OU link list |
| 18 | Confirm GPO permissions | Management Host | `Get-GPPermission -Name "<target-gpo-name>" -All` | Read and Apply scope is known |
| 19 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<target-gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 20 | Preserve required read access | Management Host | `Set-GPPermission -Name "<target-gpo-name>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | Authenticated Users can read GPO |
| 21 | Open GPMC editor | Management Host | `GPMC > right-click GPO > Edit` | Group Policy Management Editor opens |
| 22 | Configure Control Panel restriction | Management Host | `User Configuration > Policies > Administrative Templates > Control Panel > Prohibit access to Control Panel and PC settings = Enabled` | Control Panel policy is configured |
| 23 | Configure Run menu restriction | Management Host | `User Configuration > Policies > Administrative Templates > Start Menu and Taskbar > Remove Run menu from Start Menu = Enabled` | Run menu policy is configured |
| 24 | Configure Command Prompt restriction | Management Host | `User Configuration > Policies > Administrative Templates > System > Prevent access to the command prompt = Enabled` | Command Prompt policy is configured |
| 25 | Optional PowerShell registry policy setting | Management Host | `Set-GPRegistryValue -Name "<target-gpo-name>" -Key "<policy-key>" -ValueName "<value-name>" -Type DWord -Value <value>` | User registry policy value is written |
| 26 | Export GPO report | Management Host | `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<target-gpo-name>.html` | Report shows configured user settings |
| 27 | Back up configured GPO | Management Host | `Backup-GPO -Name "<target-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 28 | Sign in as test user | Test Client | `whoami` | Test user session is active |
| 29 | Force user policy refresh | Test Client | `gpupdate /force` | User policy refresh completes |
| 30 | Validate applied user GPOs | Test Client | `gpresult /scope user /r` | Target GPO appears under Applied GPOs |
| 31 | Export user GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-user-admin-templates.html` | HTML RSOP report exists |
| 32 | Validate HKCU registry policy values | Test Client | `Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"` | Configured user policy values exist |
| 33 | Review Group Policy operational events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Recent processing events are visible |
| 34 | Document result | Operator | `Record GPO name, user OU, settings configured, reports, backup, and test user result` | User Administrative Template baseline is documented |

# 07_Configure_User_Administrative_Template_Settings_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring user Administrative Template settings.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"
$TestUser = "tuser"

$GpoName = "CORP-User-Administrative-Templates"

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

# Optional Central Store check.
$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"
Test-Path $CentralStorePath

# Confirm target user OU and test user.
Get-ADOrganizationalUnit -Identity $TargetUserOU

Get-ADUser `
  -Identity $TestUser `
  -Properties DistinguishedName,Enabled |
  Select-Object SamAccountName,Enabled,DistinguishedName

# Capture current inheritance.
Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\inheritance-before-user-admin-template-settings.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-user-admin-template-settings.csv" -NoTypeInformation

# Back up current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath
```

# 07_Configure_User_Administrative_Template_Settings_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link a dedicated user Administrative Templates GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-User-Administrative-Templates"
$GpoComment = "User-side Administrative Template settings for controlled user experience policy."

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

# 07_Configure_User_Administrative_Template_Settings_GPMC_Configuration_Skeleton
```powershell
# Native GUI workflow for configuring User Administrative Template settings.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-User-Administrative-Templates
# 7. Select Edit.
# 8. Go to:
#      User Configuration
#      > Policies
#      > Administrative Templates
#
# Configure example setting 1:
# 9. Go to:
#      Control Panel
# 10. Open:
#      Prohibit access to Control Panel and PC settings
# 11. Set:
#      Enabled
#
# Configure example setting 2:
# 12. Go to:
#      Start Menu and Taskbar
# 13. Open:
#      Remove Run menu from Start Menu
# 14. Set:
#      Enabled
#
# Configure example setting 3:
# 15. Go to:
#      System
# 16. Open:
#      Prevent access to the command prompt
# 17. Set:
#      Enabled
#
# Optional high impact setting:
# 18. Go to:
#      System
# 19. Open:
#      Prevent access to registry editing tools
# 20. Set:
#      Enabled
#
# Close the editor.
# Refresh GPMC.
# Export a GPO report after configuration.
```

# 07_Configure_User_Administrative_Template_Settings_PowerShell_Registry_Policy_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: configure known ADMX-backed user Administrative Template registry policy values.
# Use only when the policy registry key and value are known and verified.

Import-Module GroupPolicy

$GpoName = "CORP-User-Administrative-Templates"
$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm GPO exists.
Get-GPO -Name $GpoName

# Back up before registry-policy changes.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Setting 1:
# Prohibit access to Control Panel and PC settings.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoControlPanel" `
  -Type DWord `
  -Value 1

# Setting 2:
# Remove Run menu from Start Menu.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoRun" `
  -Type DWord `
  -Value 1

# Setting 3:
# Prevent access to the command prompt.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKCU\Software\Policies\Microsoft\Windows\System" `
  -ValueName "DisableCMD" `
  -Type DWord `
  -Value 1

# Optional high impact setting:
# Prevent access to registry editing tools.
# Use carefully because it blocks the user from opening Registry Editor.
# Set-GPRegistryValue `
#   -Name $GpoName `
#   -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
#   -ValueName "DisableRegistryTools" `
#   -Type DWord `
#   -Value 1

# Export reports after registry-policy configuration.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-user-admin-template-settings.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-user-admin-template-settings.xml"

# Back up after configuration.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath
```

# 07_Configure_User_Administrative_Template_Settings_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client while signed in as the test user.
# Purpose: prove user-side Administrative Template settings apply.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm user and client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Force user and computer policy refresh.
gpupdate /force

# Show applied user-side GPOs.
gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-admin-template.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-user-admin-template-settings.html"

# Optional PowerShell RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-user-admin-template-settings.html" `
  -ErrorAction SilentlyContinue

# Validate registry policy setting 1:
# Prohibit Control Panel and PC settings.
Get-ItemProperty `
  -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ErrorAction SilentlyContinue |
  Select-Object NoControlPanel,NoRun

# Validate registry policy setting 2:
# Prevent access to command prompt.
Get-ItemProperty `
  -Path "HKCU:\Software\Policies\Microsoft\Windows\System" `
  -ErrorAction SilentlyContinue |
  Select-Object DisableCMD

# Optional validation:
# Prevent registry editing tools.
Get-ItemProperty `
  -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ErrorAction SilentlyContinue |
  Select-Object DisableRegistryTools

# Review Group Policy processing events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 100 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-user-admin-template-settings.txt"

# Practical behavior checks:
# 1. Try opening Control Panel or Settings.
# 2. Try opening Run with Windows key + R.
# 3. Try opening cmd.exe.
# Expected: configured restrictions should apply to the test user.
```

# 07_Configure_User_Administrative_Template_Settings_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final GPO report, inheritance, permission, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"
$GpoName = "CORP-User-Administrative-Templates"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\inheritance-after-user-admin-template-settings.txt"

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

# Search report for configured settings.
Select-String `
  -Path "$ReportPath\$GpoName-final.xml" `
  -Pattern "NoControlPanel","NoRun","DisableCMD","DisableRegistryTools","Administrative" |
  Out-File "$ReportPath\$GpoName-user-admin-template-report-search.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-user-admin-template-settings.csv" -NoTypeInformation

Write-Host "User Administrative Template settings report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 07_Configure_User_Administrative_Template_Settings_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<target-gpo-name>"` | Confirms target GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<target-user-ou-dn>"` | Confirms GPO is linked to target user OU | GPO appears in link list |
| `Get-GPPermission -Name "<target-gpo-name>" -All` | Confirms user or group can read and apply GPO | Expected permissions appear |
| `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | Confirms user is in target OU scope | User DN is under target user OU |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms SYSVOL is reachable | Returns True |
| `gpmc.msc` | Opens GPMC for User Administrative Template editing | Console opens |
| `Set-GPRegistryValue -Name "<target-gpo-name>" -Key "<key>" -ValueName "<name>" -Type DWord -Value <value>` | Configures known user registry-based policy value | GPO report shows setting |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path "<report-path>\<target-gpo-name>.html"` | Exports readable GPO report | HTML report exists |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Xml -Path "<report-path>\<target-gpo-name>.xml"` | Exports searchable GPO report | XML report exists |
| `Select-String -Path "<report-path>\<target-gpo-name>.xml" -Pattern "NoControlPanel","NoRun","DisableCMD"` | Searches report for configured settings | Configured values appear |
| `Backup-GPO -Name "<target-gpo-name>" -Path "<backup-path>"` | Backs up configured GPO | Backup completes |
| `whoami` | Confirms test user session | Expected domain user appears |
| `gpupdate /force` | Forces user policy refresh | User policy update completes |
| `gpresult /scope user /r` | Shows applied user GPOs | Target GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-user-admin-template-settings.html` | Exports full client RSOP report | HTML report exists |
| `Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"` | Validates Control Panel and Run restrictions | `NoControlPanel` and `NoRun` appear |
| `Get-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\System"` | Validates command prompt restriction | `DisableCMD` appears |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Reviews client processing events | Recent policy processing events appear |

# 07_Configure_User_Administrative_Template_Settings_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: remove configured user Administrative Template registry policy values or disable the GPO link.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-User-Administrative-Templates"

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
# Remove specific configured user registry policy values.

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoControlPanel" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoRun" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKCU\Software\Policies\Microsoft\Windows\System" `
  -ValueName "DisableCMD" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ValueName "DisableRegistryTools" `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Disable the GPO link while preserving the GPO for review.
# Set-GPLink `
#   -Name $GpoName `
#   -Target $TargetUserOU `
#   -LinkEnabled No

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
  Out-File "$ReportPath\inheritance-after-user-admin-template-rollback.txt"

Write-Host "Rollback complete. Sign in as the test user, run gpupdate /force, and validate HKCU policy values are removed."
```

# 07_Configure_User_Administrative_Template_Settings_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| User GPO does not apply | User object is not under linked OU | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | Move user to target OU or link GPO to correct user OU |
| Computer GPO applies but user GPO does not | User and computer are in different OU paths | `gpresult /scope user /r`; `gpresult /scope computer /r` | Link user GPO to user OU or configure loopback intentionally in a later task |
| Administrative Templates missing in GPMC | ADMX issue, GPMC issue, or wrong GPO editor path | `gpmc.msc`; check User Configuration path | Reopen GPMC and verify ADMX source |
| New ADMX settings missing | Central Store outdated or local ADMX mismatch | `Test-Path "\\<domain>\SYSVOL\<domain>\Policies\PolicyDefinitions"` | Update Central Store in later ADMX workbook |
| `Set-GPRegistryValue` fails | Wrong key format, GPO missing, or permission issue | `Get-GPO -Name "<target-gpo-name>"`; `whoami /groups` | Correct GPO name, registry path, or permissions |
| GPO report does not show setting | Setting not configured, wrong GPO, or stale report | `Get-GPOReport -Name "<target-gpo-name>"` | Reconfigure setting and export fresh report |
| `gpresult /scope user /r` does not show target GPO | Link, filtering, WMI, disabled GPO side, or wrong user OU | `Get-GPInheritance`; `Get-GPPermission`; `Get-GPOReport` | Fix scope, filtering, WMI, or GPO status |
| GPO appears under denied GPOs | Security filtering or WMI filter blocks test user | `gpresult /scope user /r` | Add test user or group with `GpoApply` |
| HKCU policy value missing | User policy did not process or wrong registry path | `gpresult /scope user /r`; `Get-ItemProperty` | Confirm user-side setting and rerun `gpupdate /force` |
| Setting does not take effect immediately | User setting requires sign out, sign in, or Explorer restart | `gpupdate /force`; logoff/logon | Sign out and back in as test user |
| User restrictions apply to too many users | GPO linked too high or filtering too broad | `Get-GPInheritance`; `Get-GPPermission` | Link lower or narrow security filtering |
| User restrictions do not apply on shared workstation | User object receives policy but loopback expectations are wrong | `gpresult /r` | Configure loopback in later workbook if shared computer behavior is required |
| Command Prompt remains accessible | Policy path, value, or user refresh issue | `Get-ItemProperty HKCU:\Software\Policies\Microsoft\Windows\System` | Confirm `DisableCMD`, refresh user policy, sign out and back in |
| Control Panel remains accessible | Policy value missing or conflicting GPO | `gpresult /h`; check `NoControlPanel` | Fix conflicting GPO or setting value |
| Run dialog still works | Policy not applied or Explorer needs refresh | `Get-ItemProperty HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` | Refresh policy, restart Explorer, or sign out and back in |
| SYSVOL unavailable | DNS, SMB, or DC issue | `Test-Path "\\<domain>\SYSVOL"`; `dcdiag /test:sysvolcheck` | Fix DNS, SMB, or SYSVOL |
| GPO differs between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before continuing |
| Removing policy value does not remove behavior immediately | Client has not refreshed or user session is stale | `gpupdate /force`; sign out/sign in | Refresh policy and start a new user session |
| User lockout risk from restrictive settings | High impact setting applied too broadly | `Get-GPInheritance`; `Get-GPPermission` | Disable link, remove setting, or narrow filtering |

# 07_Configure_User_Administrative_Template_Settings_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes basic GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before adding user settings |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which users can apply this GPO |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Adds optional WMI targeting before setting application |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Computer-side companion workbook |
| `Create_Baseline_OU_Structure.md` | Provides user OU target for user policy |
| `Create_Baseline_Groups_And_Test_Users.md` | Provides test user and filtering groups |
| `Join_Windows_Client_To_Domain.md` | Provides domain-joined test client for user logon and `gpresult` |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms SYSVOL and DC locator before troubleshooting user policy |
| `08_Configure_Security_Options_User_Rights_Assignment_And_Audit_Policy.md` | Next workbook for deeper security settings |