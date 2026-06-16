06_Configure_Computer_Administrative_Template_Settings.md
# 06_Configure_Computer_Administrative_Template_Settings

# 06_Configure_Computer_Administrative_Template_Settings_Index
06_Configure_Computer_Administrative_Template_Settings.md
06_Configure_Computer_Administrative_Template_Settings
06_Configure_Computer_Administrative_Template_Settings_Source_Basis
06_Configure_Computer_Administrative_Template_Settings_Mental_Model
06_Configure_Computer_Administrative_Template_Settings_Planning_Table
06_Configure_Computer_Administrative_Template_Settings_Admin_Template_Settings_Map
06_Configure_Computer_Administrative_Template_Settings_Configuration_Checklist
06_Configure_Computer_Administrative_Template_Settings_Precheck_Skeleton
06_Configure_Computer_Administrative_Template_Settings_Create_And_Link_GPO_Skeleton
06_Configure_Computer_Administrative_Template_Settings_GPMC_Configuration_Skeleton
06_Configure_Computer_Administrative_Template_Settings_PowerShell_Registry_Policy_Skeleton
06_Configure_Computer_Administrative_Template_Settings_Client_RSOP_Validation_Skeleton
06_Configure_Computer_Administrative_Template_Settings_Report_And_Backup_Skeleton
06_Configure_Computer_Administrative_Template_Settings_Verification_Commands
06_Configure_Computer_Administrative_Template_Settings_Rollback
06_Configure_Computer_Administrative_Template_Settings_Failure_Checks
06_Configure_Computer_Administrative_Template_Settings_Related_Labs

# 06_Configure_Computer_Administrative_Template_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy Administrative Templates | ADMX-backed computer policy settings |
| Microsoft Learn | Group Policy Management Console | Editing Computer Configuration Administrative Template settings |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up GPOs |
| Microsoft Learn | `Set-GPRegistryValue` | Configuring registry-based policy values when the exact ADMX-backed registry path is known |
| Microsoft Learn | `Remove-GPRegistryValue` | Removing configured registry policy values during rollback |
| Microsoft Learn | `Get-GPOReport` | Verifying configured Administrative Template settings in reports |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, and GroupPolicy event logs | Proving computer Administrative Template settings apply |
| Windows Server operational practice | Separate computer baseline GPOs by purpose | Keeping workstation, server, security, update, and user policies supportable |

# 06_Configure_Computer_Administrative_Template_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Administrative Templates | ADMX-backed policy settings exposed in GPMC |
| ADMX | Policy definition file that describes available settings |
| ADML | Language file paired with ADMX definitions |
| Computer Configuration | GPO half applied to computer accounts |
| User Configuration | GPO half applied to user accounts |
| Registry-based policy | Policy stored under `HKLM\Software\Policies` or related managed policy paths |
| Registry.pol | File in SYSVOL where registry-based policy settings are stored |
| Enabled | Policy state that writes the configured registry policy value |
| Disabled | Policy state that writes the opposite policy value when the ADMX supports it |
| Not Configured | Policy state that removes the GPO's management of that setting |
| GPMC path | GUI path used to configure an ADMX policy |
| Registry path | Underlying managed registry key and value controlled by the policy |
| Central Store | Domain SYSVOL location for shared ADMX and ADML files |
| RSOP | Resultant Set of Policy showing effective Administrative Template settings |
| GPResult | Client-side proof of applied and denied GPOs |
| First rule | Configure computer Administrative Template settings in a dedicated computer GPO before mixing them into broad baselines |
| Blunt rule | Do not paste random registry values into a GPO unless you know the ADMX policy and expected client effect |

# 06_Configure_Computer_Administrative_Template_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-ou-dn>` |
| Target computer | `WIN11-01` | `<test-computer>` |
| Target GPO | `CORP-Workstation-Administrative-Templates` | `<target-gpo-name>` |
| Parent baseline GPO | `CORP-Workstation-Baseline` | `<baseline-gpo-name>` |
| Security filtering group | `GG_GPO_Workstation_Baseline_Apply` | `<filtering-group>` |
| Policy side | Computer Configuration | `Computer Configuration` |
| ADMX source | Local store or Central Store | `<admx-source>` |
| Central Store path | `\\corp.local\SYSVOL\corp.local\Policies\PolicyDefinitions` | `<central-store-path>` |
| Setting 1 | Always wait for network at computer startup and logon | `<setting-1>` |
| Setting 2 | Turn off Microsoft consumer experiences | `<setting-2>` |
| Setting 3 | Turn off AutoPlay | `<setting-3>` |
| Link enabled | Yes | `<yes-no>` |
| Enforced | No | `<yes-no>` |
| Block inheritance | No | `<yes-no>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Remove registry policy values or restore GPO backup | `<rollback-plan>` |

# 06_Configure_Computer_Administrative_Template_Settings_Admin_Template_Settings_Map
| Setting | GPMC Path | Registry Policy Path | Value | Example State | Validation |
|---|---|---|---|---|---|
| Always wait for network at computer startup and logon | `Computer Configuration > Policies > Administrative Templates > System > Logon` | `HKLM\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon` | `SyncForegroundPolicy` DWORD `1` | Enabled | Registry value exists and GPO appears in `gpresult` |
| Turn off Microsoft consumer experiences | `Computer Configuration > Policies > Administrative Templates > Windows Components > Cloud Content` | `HKLM\Software\Policies\Microsoft\Windows\CloudContent` | `DisableWindowsConsumerFeatures` DWORD `1` | Enabled | Registry value exists and GPO report shows setting |
| Turn off AutoPlay | `Computer Configuration > Policies > Administrative Templates > Windows Components > AutoPlay Policies` | `HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` | `NoDriveTypeAutoRun` DWORD `255` | Enabled for all drives | Registry value exists and policy appears in RSOP |
| Do not process the run once list | `Computer Configuration > Policies > Administrative Templates > System > Logon` | `HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` | `DisableLocalMachineRunOnce` DWORD `1` | Optional hardening | Registry value exists if configured |
| Turn off Windows Installer | `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Installer` | `HKLM\Software\Policies\Microsoft\Windows\Installer` | `DisableMSI` DWORD `1` or `2` | Optional, high impact | Only configure with change approval |
| Group Policy refresh interval for computers | `Computer Configuration > Policies > Administrative Templates > System > Group Policy` | ADMX-managed policy values under Group Policy paths | Varies | Optional lab setting | Confirm in GPO report and event logs |

# 06_Configure_Computer_Administrative_Template_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-ou-dn>"` | Target OU returns |
| 6 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer object returns |
| 7 | Confirm test computer is inside target OU scope | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | DistinguishedName is under target OU |
| 8 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 9 | Confirm GPMC opens | Management Host | `gpmc.msc` | GPMC opens |
| 10 | Confirm current GPO inventory | Management Host | `Get-GPO -All` | Existing GPOs are listed |
| 11 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 12 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 13 | Back up existing GPOs before changes | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 14 | Create dedicated computer Administrative Templates GPO if missing | Management Host | `New-GPO -Name "<target-gpo-name>" -Comment "Computer Administrative Template settings for workstations"` | GPO exists |
| 15 | Link GPO to workstation OU | Management Host | `New-GPLink -Name "<target-gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 16 | Keep link unenforced by default | Management Host | `Set-GPLink -Name "<target-gpo-name>" -Target "<target-ou-dn>" -Enforced No` | Link is not enforced |
| 17 | Confirm link state | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | GPO appears in OU link list |
| 18 | Confirm GPO permissions | Management Host | `Get-GPPermission -Name "<target-gpo-name>" -All` | Read and Apply scope is known |
| 19 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<target-gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 20 | Open GPMC editor | Management Host | `GPMC > right-click GPO > Edit` | Group Policy Management Editor opens |
| 21 | Configure Always wait for network | Management Host | `Computer Configuration > Policies > Administrative Templates > System > Logon > Always wait for the network at computer startup and logon = Enabled` | Computer startup policy is configured |
| 22 | Configure consumer experiences policy | Management Host | `Computer Configuration > Policies > Administrative Templates > Windows Components > Cloud Content > Turn off Microsoft consumer experiences = Enabled` | Consumer experiences policy is configured |
| 23 | Configure AutoPlay policy | Management Host | `Computer Configuration > Policies > Administrative Templates > Windows Components > AutoPlay Policies > Turn off AutoPlay = Enabled` | AutoPlay policy is configured |
| 24 | Optional PowerShell registry policy setting | Management Host | `Set-GPRegistryValue -Name "<target-gpo-name>" -Key "<policy-key>" -ValueName "<value-name>" -Type DWord -Value <value>` | ADMX-backed registry policy value is written |
| 25 | Export GPO report | Management Host | `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<target-gpo-name>.html` | Report shows configured settings |
| 26 | Back up configured GPO | Management Host | `Backup-GPO -Name "<target-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 27 | Force policy refresh | Test Client | `gpupdate /force` | Computer policy refresh completes |
| 28 | Validate applied computer GPOs | Test Client | `gpresult /scope computer /r` | Target GPO appears under Applied GPOs |
| 29 | Export client GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-admin-templates.html` | HTML RSOP report exists |
| 30 | Validate registry policy values | Test Client | `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon"` | Configured policy value exists |
| 31 | Review Group Policy operational events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Recent processing events are visible |
| 32 | Document result | Operator | `Record GPO name, target OU, settings configured, reports, backup, and test client result` | Administrative Template baseline is documented |

# 06_Configure_Computer_Administrative_Template_Settings_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring computer Administrative Template settings.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"

$GpoName = "CORP-Workstation-Administrative-Templates"

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

# Confirm target OU and test computer.
Get-ADOrganizationalUnit -Identity $TargetOU

Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName

# Capture current inheritance.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\inheritance-before-admin-template-settings.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-admin-template-settings.csv" -NoTypeInformation

# Back up current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath
```

# 06_Configure_Computer_Administrative_Template_Settings_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link a dedicated computer Administrative Templates GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Administrative-Templates"
$GpoComment = "Computer-side Administrative Template settings for workstation baseline policy."

# Confirm target OU exists.
Get-ADOrganizationalUnit -Identity $TargetOU

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

# Link the GPO to the target OU if not already linked.
$Inheritance = Get-GPInheritance -Target $TargetOU
$ExistingLink = $Inheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $GpoName }

if ($ExistingLink) {
    Write-Host "GPO link already exists on target OU."
}
else {
    New-GPLink `
      -Name $GpoName `
      -Target $TargetOU `
      -LinkEnabled Yes

    Write-Host "Linked $GpoName to $TargetOU"
}

# Safe default: enabled, not enforced.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetOU `
  -LinkEnabled Yes `
  -Enforced No

# Optional: place at desired link order.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetOU `
  -Order 1

# Confirm final link state.
Get-GPInheritance `
  -Target $TargetOU |
  Format-List
```

# 06_Configure_Computer_Administrative_Template_Settings_GPMC_Configuration_Skeleton
```powershell
# Native GUI workflow for configuring Administrative Template settings.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-Workstation-Administrative-Templates
# 7. Select Edit.
# 8. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#
# Configure example setting 1:
# 9. Go to:
#      System
#      > Logon
# 10. Open:
#      Always wait for the network at computer startup and logon
# 11. Set:
#      Enabled
#
# Configure example setting 2:
# 12. Go to:
#      Windows Components
#      > Cloud Content
# 13. Open:
#      Turn off Microsoft consumer experiences
# 14. Set:
#      Enabled
#
# Configure example setting 3:
# 15. Go to:
#      Windows Components
#      > AutoPlay Policies
# 16. Open:
#      Turn off AutoPlay
# 17. Set:
#      Enabled
# 18. Select:
#      All drives
#
# Close the editor.
# Refresh GPMC.
# Export a GPO report after configuration.
```

# 06_Configure_Computer_Administrative_Template_Settings_PowerShell_Registry_Policy_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: configure known ADMX-backed computer Administrative Template registry policy values.
# Use only when the policy registry key and value are known and verified.

Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Administrative-Templates"
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
# Always wait for the network at computer startup and logon.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -ValueName "SyncForegroundPolicy" `
  -Type DWord `
  -Value 1

# Setting 2:
# Turn off Microsoft consumer experiences.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\CloudContent" `
  -ValueName "DisableWindowsConsumerFeatures" `
  -Type DWord `
  -Value 1

# Setting 3:
# Turn off AutoPlay for all drives.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoDriveTypeAutoRun" `
  -Type DWord `
  -Value 255

# Optional high impact example:
# Turn off Windows Installer.
# Do not enable unless approved because it can interfere with software deployment.
# Set-GPRegistryValue `
#   -Name $GpoName `
#   -Key "HKLM\Software\Policies\Microsoft\Windows\Installer" `
#   -ValueName "DisableMSI" `
#   -Type DWord `
#   -Value 1

# Export report after registry-policy configuration.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-admin-template-settings.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-admin-template-settings.xml"

# Back up after configuration.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath
```

# 06_Configure_Computer_Administrative_Template_Settings_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: prove computer-side Administrative Template settings apply.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Force computer policy refresh.
gpupdate /force

# Show applied computer-side GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-admin-template.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-admin-template-settings.html"

# Optional PowerShell RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-admin-template-settings.html" `
  -ErrorAction SilentlyContinue

# Validate registry policy setting 1:
# Always wait for network.
Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -ErrorAction SilentlyContinue |
  Select-Object SyncForegroundPolicy

# Validate registry policy setting 2:
# Turn off Microsoft consumer experiences.
Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\CloudContent" `
  -ErrorAction SilentlyContinue |
  Select-Object DisableWindowsConsumerFeatures

# Validate registry policy setting 3:
# Turn off AutoPlay.
Get-ItemProperty `
  -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ErrorAction SilentlyContinue |
  Select-Object NoDriveTypeAutoRun

# Review Group Policy processing events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 100 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-admin-template-settings.txt"
```

# 06_Configure_Computer_Administrative_Template_Settings_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final GPO report, inheritance, permission, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"
$GpoName = "CORP-Workstation-Administrative-Templates"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\inheritance-after-admin-template-settings.txt"

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
  -Pattern "SyncForegroundPolicy","DisableWindowsConsumerFeatures","NoDriveTypeAutoRun","Administrative" |
  Out-File "$ReportPath\$GpoName-admin-template-report-search.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-admin-template-settings.csv" -NoTypeInformation

Write-Host "Computer Administrative Template settings report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 06_Configure_Computer_Administrative_Template_Settings_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<target-gpo-name>"` | Confirms target GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<target-ou-dn>"` | Confirms GPO is linked to target OU | GPO appears in link list |
| `Get-GPPermission -Name "<target-gpo-name>" -All` | Confirms target can read and apply GPO | Expected permissions appear |
| `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Confirms computer is in target OU scope | Computer DN is under target OU |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms SYSVOL is reachable | Returns True |
| `gpmc.msc` | Opens GPMC for Administrative Template editing | Console opens |
| `Set-GPRegistryValue -Name "<target-gpo-name>" -Key "<key>" -ValueName "<name>" -Type DWord -Value <value>` | Configures known registry-based policy value | GPO report shows setting |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path "<report-path>\<target-gpo-name>.html"` | Exports readable GPO report | HTML report exists |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Xml -Path "<report-path>\<target-gpo-name>.xml"` | Exports searchable GPO report | XML report exists |
| `Select-String -Path "<report-path>\<target-gpo-name>.xml" -Pattern "SyncForegroundPolicy","DisableWindowsConsumerFeatures","NoDriveTypeAutoRun"` | Searches report for configured settings | Configured values appear |
| `Backup-GPO -Name "<target-gpo-name>" -Path "<backup-path>"` | Backs up configured GPO | Backup completes |
| `gpupdate /force` | Forces client policy refresh | Computer policy update completes |
| `gpresult /scope computer /r` | Shows applied computer GPOs | Target GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-admin-template-settings.html` | Exports full client RSOP report | HTML report exists |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon"` | Validates network wait policy value | `SyncForegroundPolicy` appears |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\CloudContent"` | Validates consumer experiences policy value | `DisableWindowsConsumerFeatures` appears |
| `Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"` | Validates AutoPlay policy value | `NoDriveTypeAutoRun` appears |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Reviews client processing events | Recent policy processing events appear |

# 06_Configure_Computer_Administrative_Template_Settings_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: remove configured computer Administrative Template registry policy values or disable the GPO link.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Administrative-Templates"

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
# Remove specific configured registry policy values.

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -ValueName "SyncForegroundPolicy" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\CloudContent" `
  -ValueName "DisableWindowsConsumerFeatures" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoDriveTypeAutoRun" `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Disable the GPO link while preserving the GPO for review.
# Set-GPLink `
#   -Name $GpoName `
#   -Target $TargetOU `
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
  -Target $TargetOU |
  Out-File "$ReportPath\inheritance-after-admin-template-rollback.txt"

Write-Host "Rollback complete. Run gpupdate /force on the test client and validate registry values are removed."
```

# 06_Configure_Computer_Administrative_Template_Settings_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Administrative Templates missing in GPMC | GPMC issue, ADMX issue, or console opened wrong GPO | `gpmc.msc`; check target GPO editor | Reopen GPMC, confirm GPO, verify ADMX files |
| New ADMX settings missing | Central Store outdated or local ADMX mismatch | `Test-Path "\\<domain>\SYSVOL\<domain>\Policies\PolicyDefinitions"` | Update Central Store in later ADMX workbook |
| `Set-GPRegistryValue` fails | Wrong key format, GPO missing, or permission issue | `Get-GPO -Name "<target-gpo-name>"`; `whoami /groups` | Correct GPO name, key path, or permissions |
| GPO report does not show setting | Setting not configured, wrong GPO, or report stale | `Get-GPOReport -Name "<target-gpo-name>"` | Reconfigure setting and export fresh report |
| Client does not apply GPO | Wrong OU, disabled link, filtering, WMI, or replication | `gpresult /scope computer /r`; `Get-GPInheritance` | Fix scope, link, filtering, WMI, or replication |
| Registry policy value missing on client | Computer policy did not process or wrong registry path | `gpresult /scope computer /r`; `Get-ItemProperty` | Confirm setting path and rerun `gpupdate /force` |
| User policy applies but computer policy does not | GPO has only user settings or computer is out of scope | `gpresult /scope computer /r`; GPO report | Configure Computer Configuration and move computer into scope |
| Setting appears under denied GPOs | Security filtering or WMI filter blocks client | `gpresult /r`; `Get-GPPermission`; WMI filter check | Fix filtering or remove WMI filter |
| Policy applies to too many computers | GPO linked too high or filtering too broad | `Get-GPInheritance`; `Get-GPPermission` | Link lower or use security filtering |
| Client needs reboot for behavior change | Some computer settings require restart | `gpupdate /force`; event logs | Reboot test client |
| Multiple GPOs configure same setting | GPO precedence conflict | `gpresult /h`; `Get-GPInheritance` | Fix link order or consolidate setting |
| GPO link is enforced accidentally | Enforced flag set | `Get-GPInheritance -Target "<target-ou-dn>"` | `Set-GPLink -Enforced No` |
| Inheritance is blocked unexpectedly | OU blocks parent GPOs | `Get-GPInheritance -Target "<target-ou-dn>"` | `Set-GPInheritance -IsBlocked No` |
| SYSVOL unavailable | DNS, SMB, or DC issue | `Test-Path "\\<domain>\SYSVOL"`; `dcdiag /test:sysvolcheck` | Fix DNS, SMB, or SYSVOL |
| GPO state differs between DCs | AD or SYSVOL replication problem | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before continuing |
| Removing policy value does not remove effect immediately | Client has not refreshed or tattooed non-policy value exists | `gpupdate /force`; check registry | Refresh policy, reboot, and verify policy path |
| High impact policy breaks workflow | Setting was not piloted | GPO report and affected client behavior | Disable link or remove setting, then pilot before redeploying |

# 06_Configure_Computer_Administrative_Template_Settings_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes basic GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before adding Administrative Template settings |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which computers can apply this GPO |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Adds optional WMI targeting before this setting GPO applies |
| `Create_Baseline_OU_Structure.md` | Provides workstation OU target for computer policy |
| `Join_Windows_Client_To_Domain.md` | Provides domain-joined test client for `gpupdate` and `gpresult` |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms SYSVOL and DC locator before troubleshooting GPO settings |
| `07_Configure_User_Administrative_Template_Settings.md` | Next workbook for user-side Administrative Template policy |