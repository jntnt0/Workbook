12_Configure_Windows_Update_And_Defender_GPO_Settings.md
# 12_Configure_Windows_Update_And_Defender_GPO_Settings

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Index
12_Configure_Windows_Update_And_Defender_GPO_Settings.md
12_Configure_Windows_Update_And_Defender_GPO_Settings
12_Configure_Windows_Update_And_Defender_GPO_Settings_Source_Basis
12_Configure_Windows_Update_And_Defender_GPO_Settings_Mental_Model
12_Configure_Windows_Update_And_Defender_GPO_Settings_Planning_Table
12_Configure_Windows_Update_And_Defender_GPO_Settings_Settings_Map
12_Configure_Windows_Update_And_Defender_GPO_Settings_Configuration_Checklist
12_Configure_Windows_Update_And_Defender_GPO_Settings_Precheck_Skeleton
12_Configure_Windows_Update_And_Defender_GPO_Settings_Create_And_Link_GPO_Skeleton
12_Configure_Windows_Update_And_Defender_GPO_Settings_Windows_Update_GPMC_Skeleton
12_Configure_Windows_Update_And_Defender_GPO_Settings_Defender_GPMC_Skeleton
12_Configure_Windows_Update_And_Defender_GPO_Settings_PowerShell_Registry_Policy_Skeleton
12_Configure_Windows_Update_And_Defender_GPO_Settings_Client_RSOP_Validation_Skeleton
12_Configure_Windows_Update_And_Defender_GPO_Settings_Report_And_Backup_Skeleton
12_Configure_Windows_Update_And_Defender_GPO_Settings_Verification_Commands
12_Configure_Windows_Update_And_Defender_GPO_Settings_Rollback
12_Configure_Windows_Update_And_Defender_GPO_Settings_Failure_Checks
12_Configure_Windows_Update_And_Defender_GPO_Settings_Related_Labs

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Windows Update for Business Group Policy | Configuring update behavior through GPO |
| Microsoft Learn | Windows Update Administrative Templates | Configuring automatic updates, restart behavior, active hours, and update source behavior |
| Microsoft Learn | WSUS client Group Policy settings | Pointing clients to an internal update service when WSUS is used |
| Microsoft Learn | Microsoft Defender Antivirus Group Policy | Configuring Defender antivirus and protection settings through GPO |
| Microsoft Learn | Microsoft Defender Antivirus PowerShell module | Validating Defender state with `Get-MpComputerStatus` and `Get-MpPreference` |
| Microsoft Learn | Group Policy Management Console | Editing Windows Update and Defender Administrative Template settings |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up the GPO |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, registry policy paths, event logs | Proving update and Defender policies apply |
| Windows Server operational practice | Separate update policy from security baseline policy when troubleshooting | Avoiding confusion between update delivery, AV hardening, and general endpoint baseline settings |

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Windows Update policy | GPO settings controlling how Windows clients scan, download, install, and restart for updates |
| Windows Update for Business | Cloud-based Windows update policy model using deferral and quality or feature update controls |
| WSUS | Internal Windows Server Update Services update source |
| Update source | Location where clients scan for updates, either Microsoft Update or internal WSUS |
| Automatic Updates | Policy area controlling download and install behavior |
| Active Hours | Time range where automatic restarts should be avoided |
| Restart behavior | Policy area controlling restart warnings, deadlines, and user experience |
| Quality updates | Monthly security and reliability updates |
| Feature updates | Windows version upgrades |
| Driver updates | Optional device driver delivery through Windows Update |
| Microsoft Defender Antivirus | Built-in antimalware engine controlled through policy and Defender preferences |
| Real-time protection | Defender scanning behavior for active file and process activity |
| Cloud-delivered protection | Defender cloud intelligence feature |
| Sample submission | Defender behavior for sending file samples to Microsoft |
| Exclusions | Paths, processes, extensions, or IPs excluded from Defender scanning |
| Tamper Protection | Defender protection against unauthorized security setting changes, commonly managed through modern security tooling rather than classic GPO alone |
| Security Intelligence updates | Defender definition updates |
| Policy registry path | Managed registry location where GPO writes Windows Update or Defender settings |
| RSOP | Effective policy result after GPO processing |
| First rule | Do not mix WSUS and Windows Update for Business settings randomly |
| Blunt rule | A Windows Update GPO can stop patching or force reboots, so pilot before broad rollout |

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target computer OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-computer-ou-dn>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Update and Defender GPO | `CORP-Workstation-Update-Defender` | `<update-defender-gpo>` |
| Security filtering group | `GG_GPO_Workstation_Update_Defender_Apply` | `<filtering-group>` |
| Update source | Microsoft Update or WSUS | `<update-source>` |
| WSUS server | `http://WSUS01:8530` | `<wsus-url>` |
| Intranet statistics server | `http://WSUS01:8530` | `<wsus-status-url>` |
| Automatic update mode | Auto download and schedule install | `<auto-update-mode>` |
| Scheduled install day | Every day or specific day | `<install-day>` |
| Scheduled install time | `03:00` | `<install-time>` |
| Active hours start | `08:00` | `<active-hours-start>` |
| Active hours end | `17:00` | `<active-hours-end>` |
| Auto restart behavior | Avoid automatic restart with logged-on users | `<restart-decision>` |
| Quality update deferral | `0` to `7` days | `<quality-deferral-days>` |
| Feature update deferral | `30` to `90` days | `<feature-deferral-days>` |
| Driver updates | Include or exclude | `<driver-update-decision>` |
| Defender real-time protection | Enabled | `<defender-rtp>` |
| Defender cloud protection | Enabled | `<defender-cloud>` |
| Sample submission | Safe automatic sample submission | `<sample-submission>` |
| Defender exclusions | None unless required | `<exclusions>` |
| Security intelligence update source | Default or WSUS plus fallback | `<signature-update-source>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Disable link, restore GPO backup, or remove specific registry policy values | `<rollback-plan>` |

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Settings_Map
| Category | GPMC Path | Example Setting | Example Baseline | Validation |
|---|---|---|---|---|
| Automatic Updates | `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Update > Manage end user experience` | Configure Automatic Updates | Enabled, auto download and schedule install | Registry, GPO report, Windows Update UI |
| Restart behavior | `Windows Update > Manage end user experience` | No auto-restart with logged on users | Enabled | Registry and restart behavior |
| Active Hours | `Windows Update > Manage end user experience` | Turn off auto-restart for updates during active hours | Enabled | Registry and GPO report |
| WSUS source | `Windows Update > Manage updates offered from Windows Server Update Service` | Specify intranet Microsoft update service location | Enabled with WSUS URLs | Registry and update scan source |
| Microsoft Update source | `Windows Update > Manage updates offered from Windows Update` | Update source policy | Use when not using WSUS | GPO report and scan behavior |
| Quality update deferral | `Windows Update > Manage updates offered from Windows Update` | Select when Quality Updates are received | Deferral based on policy | Registry and GPO report |
| Feature update deferral | `Windows Update > Manage updates offered from Windows Update` | Select when Preview Builds and Feature Updates are received | Deferral based on policy | Registry and GPO report |
| Driver updates | `Windows Update > Manage updates offered from Windows Update` | Do not include drivers with Windows Updates | Based on design | Registry and update behavior |
| Defender Real-Time Protection | `Microsoft Defender Antivirus > Real-time Protection` | Turn off real-time protection | Disabled or Not Configured | `Get-MpComputerStatus` |
| Defender Cloud Protection | `Microsoft Defender Antivirus > MAPS` | Join Microsoft MAPS | Enabled based on org policy | `Get-MpPreference` |
| Defender Sample Submission | `Microsoft Defender Antivirus > MAPS` | Send file samples when further analysis is required | Safe automatic mode | `Get-MpPreference` |
| Defender Exclusions | `Microsoft Defender Antivirus > Exclusions` | Path, process, extension exclusions | Avoid unless documented | `Get-MpPreference` |
| Defender Scan | `Microsoft Defender Antivirus > Scan` | Schedule scan day and time | Based on endpoint plan | `Get-MpPreference` |
| Defender Signature Updates | `Microsoft Defender Antivirus > Security Intelligence Updates` | Define update source order | Default unless WSUS design requires change | `Get-MpComputerStatus` |

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target computer OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-computer-ou-dn>"` | Target computer OU returns |
| 6 | Confirm pilot OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-ou-dn>"` | Pilot OU returns |
| 7 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 8 | Confirm test computer is in pilot scope | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under pilot OU |
| 9 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Confirm GPMC opens | Management Host | `gpmc.msc` | GPMC opens |
| 11 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 12 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 13 | Back up current GPOs before changes | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 14 | Create Update and Defender GPO if missing | Management Host | `New-GPO -Name "<update-defender-gpo>" -Comment "Windows Update and Microsoft Defender policy for workstation pilot"` | GPO exists |
| 15 | Link GPO to pilot OU | Management Host | `New-GPLink -Name "<update-defender-gpo>" -Target "<pilot-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 16 | Keep GPO unenforced by default | Management Host | `Set-GPLink -Name "<update-defender-gpo>" -Target "<pilot-ou-dn>" -Enforced No` | Link is not enforced |
| 17 | Confirm link state | Management Host | `Get-GPInheritance -Target "<pilot-ou-dn>"` | GPO appears in pilot OU link list |
| 18 | Confirm GPO permissions | Management Host | `Get-GPPermission -Name "<update-defender-gpo>" -All` | Read and Apply permissions are known |
| 19 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<update-defender-gpo>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 20 | Preserve required read access | Management Host | `Set-GPPermission -Name "<update-defender-gpo>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | Authenticated Users can read GPO |
| 21 | Open GPO editor | Management Host | `GPMC > right-click GPO > Edit` | Group Policy Management Editor opens |
| 22 | Configure Automatic Updates | Management Host | `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Update > Manage end user experience > Configure Automatic Updates` | Automatic update behavior is configured |
| 23 | Configure restart behavior | Management Host | `Windows Update > Manage end user experience > No auto-restart with logged on users for scheduled automatic updates installations` | Restart behavior is configured |
| 24 | Configure Active Hours if required | Management Host | `Windows Update > Manage end user experience > Turn off auto-restart for updates during active hours` | Active hours behavior is configured |
| 25 | Configure WSUS source if used | Management Host | `Windows Update > Manage updates offered from Windows Server Update Service > Specify intranet Microsoft update service location` | WSUS URLs are configured |
| 26 | Configure Windows Update for Business deferrals if not using WSUS | Management Host | `Windows Update > Manage updates offered from Windows Update` | Deferral policy is configured |
| 27 | Configure driver update behavior | Management Host | `Windows Update > Manage updates offered from Windows Update > Do not include drivers with Windows Updates` | Driver update behavior is configured |
| 28 | Configure Defender real-time protection | Management Host | `Computer Configuration > Policies > Administrative Templates > Microsoft Defender Antivirus > Real-time Protection` | Real-time protection policy is configured |
| 29 | Configure Defender cloud protection | Management Host | `Microsoft Defender Antivirus > MAPS` | Cloud protection policy is configured |
| 30 | Configure Defender scan schedule if required | Management Host | `Microsoft Defender Antivirus > Scan` | Scheduled scan behavior is configured |
| 31 | Configure Defender security intelligence updates if required | Management Host | `Microsoft Defender Antivirus > Security Intelligence Updates` | Signature update behavior is configured |
| 32 | Avoid Defender exclusions unless documented | Management Host | `Microsoft Defender Antivirus > Exclusions` | Exclusions are absent or documented |
| 33 | Export GPO HTML report | Management Host | `Get-GPOReport -Name "<update-defender-gpo>" -ReportType Html -Path C:\GPOPrep\Reports\<update-defender-gpo>.html` | Report shows configured settings |
| 34 | Export GPO XML report | Management Host | `Get-GPOReport -Name "<update-defender-gpo>" -ReportType Xml -Path C:\GPOPrep\Reports\<update-defender-gpo>.xml` | XML report exists |
| 35 | Back up configured GPO | Management Host | `Backup-GPO -Name "<update-defender-gpo>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 36 | Force client policy refresh | Test Client | `gpupdate /force` | Computer policy refresh completes |
| 37 | Reboot test client if required | Test Client | `Restart-Computer` | Startup-applied policy is refreshed |
| 38 | Validate applied computer GPOs | Test Client | `gpresult /scope computer /r` | Update and Defender GPO appears under Applied GPOs |
| 39 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-update-defender.html` | HTML RSOP report exists |
| 40 | Validate Windows Update registry policy | Test Client | `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate" -ErrorAction SilentlyContinue` | Windows Update policy values appear |
| 41 | Validate Automatic Updates registry policy | Test Client | `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" -ErrorAction SilentlyContinue` | AU policy values appear |
| 42 | Validate Defender status | Test Client | `Get-MpComputerStatus` | Defender status returns |
| 43 | Validate Defender preferences | Test Client | `Get-MpPreference` | Defender policy preferences return |
| 44 | Validate update service state | Test Client | `Get-Service wuauserv,bits,WinDefend` | Services are present and expected state is visible |
| 45 | Review Windows Update events | Test Client | `Get-WinEvent -LogName System -MaxEvents 100 | Where-Object ProviderName -like "*WindowsUpdate*"` | Update activity is visible |
| 46 | Review Defender events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 100` | Defender events are visible |
| 47 | Review Group Policy events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Policy processing events are visible |
| 48 | Document result | Operator | `Record GPO, pilot OU, update source, restart behavior, Defender settings, reports, backup, and validation output` | Update and Defender policy deployment is documented |

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring Windows Update and Defender GPO settings.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"

$GpoName = "CORP-Workstation-Update-Defender"

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

# Confirm target and pilot OUs.
Get-ADOrganizationalUnit -Identity $TargetComputerOU
Get-ADOrganizationalUnit -Identity $PilotOU

# Confirm test computer exists and is in pilot scope.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-before-update-defender.txt"

# Capture current inheritance.
Get-GPInheritance `
  -Target $TargetComputerOU |
  Out-File "$ReportPath\target-computer-ou-inheritance-before-update-defender.txt"

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-update-defender.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-update-defender.csv" -NoTypeInformation

# Back up current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link a dedicated Windows Update and Defender GPO to a pilot OU.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Update-Defender"
$GpoComment = "Pilot Windows Update and Microsoft Defender GPO. Expand scope only after validation."

# Confirm pilot OU exists.
Get-ADOrganizationalUnit -Identity $PilotOU

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

# Link the GPO to pilot OU if not already linked.
$Inheritance = Get-GPInheritance -Target $PilotOU

$ExistingLink = $Inheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $GpoName }

if ($ExistingLink) {
    Write-Host "GPO link already exists on pilot OU."
}
else {
    New-GPLink `
      -Name $GpoName `
      -Target $PilotOU `
      -LinkEnabled Yes

    Write-Host "Linked $GpoName to $PilotOU"
}

# Safe default: enabled, not enforced.
Set-GPLink `
  -Name $GpoName `
  -Target $PilotOU `
  -LinkEnabled Yes `
  -Enforced No

# Optional link order.
Set-GPLink `
  -Name $GpoName `
  -Target $PilotOU `
  -Order 1

# Confirm final link state.
Get-GPInheritance `
  -Target $PilotOU |
  Format-List
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Windows_Update_GPMC_Skeleton
```powershell
# Native GUI workflow for Windows Update policy configuration.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-Workstation-Update-Defender
# 7. Select Edit.
#
# Windows Update policy path:
# 8. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > Windows Update
#
# Automatic Updates:
# 9. Go to:
#      Manage end user experience
# 10. Open:
#      Configure Automatic Updates
# 11. Set:
#      Enabled
# 12. Example option:
#      4 - Auto download and schedule the install
# 13. Example schedule:
#      Every day at 03:00
#
# Restart behavior:
# 14. Open:
#      No auto-restart with logged on users for scheduled automatic updates installations
# 15. Set:
#      Enabled
#
# Active Hours:
# 16. Open:
#      Turn off auto-restart for updates during active hours
# 17. Set:
#      Enabled
# 18. Example:
#      Start: 8
#      End: 17
#
# WSUS source path:
# 19. Go to:
#      Manage updates offered from Windows Server Update Service
# 20. Open:
#      Specify intranet Microsoft update service location
# 21. Set:
#      Enabled
# 22. Set both URLs:
#      http://WSUS01:8530
#      http://WSUS01:8530
#
# Windows Update for Business path:
# 23. Go to:
#      Manage updates offered from Windows Update
# 24. Configure deferrals only if using Windows Update for Business design.
#
# Driver updates:
# 25. Open:
#      Do not include drivers with Windows Updates
# 26. Set based on design.
#
# Notes:
# - Do not configure WSUS and cloud deferral policies randomly in the same baseline.
# - Pick one update control model for the lab.
# - Export a GPO report after configuration.
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Defender_GPMC_Skeleton
```powershell
# Native GUI workflow for Microsoft Defender Antivirus policy configuration.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-Workstation-Update-Defender
# 3. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Microsoft Defender Antivirus
#
# Core Defender policy:
# 4. Open:
#      Turn off Microsoft Defender Antivirus
# 5. Set:
#      Disabled
#    or leave Not Configured if another management plane owns Defender.
#
# Real-time Protection:
# 6. Go to:
#      Microsoft Defender Antivirus
#      > Real-time Protection
# 7. Review and configure:
#      Turn off real-time protection
#      Turn on behavior monitoring
#      Scan all downloaded files and attachments
#      Monitor file and program activity on your computer
# 8. Baseline stance:
#      Do not turn off real-time protection.
#
# MAPS and cloud protection:
# 9. Go to:
#      Microsoft Defender Antivirus
#      > MAPS
# 10. Configure:
#      Join Microsoft MAPS
#      Send file samples when further analysis is required
#
# Scan:
# 11. Go to:
#      Microsoft Defender Antivirus
#      > Scan
# 12. Configure scheduled scan behavior only if required.
#
# Security Intelligence Updates:
# 13. Go to:
#      Microsoft Defender Antivirus
#      > Security Intelligence Updates
# 14. Configure update source behavior only if required.
#
# Exclusions:
# 15. Go to:
#      Microsoft Defender Antivirus
#      > Exclusions
# 16. Avoid exclusions unless documented and approved.
# 17. Record every exclusion with application owner and reason.
#
# Notes:
# - Do not create broad exclusions such as entire drives unless there is a controlled reason.
# - Validate Defender state on the client with Get-MpComputerStatus and Get-MpPreference.
# - Export a GPO report after configuration.
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_PowerShell_Registry_Policy_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: configure known Windows Update and Defender registry-based policy values.
# Use only values that match the intended update and Defender design.

Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Update-Defender"

$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm GPO exists.
Get-GPO -Name $GpoName

# Back up before registry-policy changes.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Windows Update:
# Configure Automatic Updates.
# AUOptions:
# 2 = Notify before download
# 3 = Auto download and notify for install
# 4 = Auto download and schedule install
# 5 = Allow local admin to choose setting
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "NoAutoUpdate" `
  -Type DWord `
  -Value 0

Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "AUOptions" `
  -Type DWord `
  -Value 4

Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "ScheduledInstallDay" `
  -Type DWord `
  -Value 0

Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "ScheduledInstallTime" `
  -Type DWord `
  -Value 3

Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "NoAutoRebootWithLoggedOnUsers" `
  -Type DWord `
  -Value 1

# WSUS example:
# Enable only if this lab uses WSUS.
$UseWSUS = $false
$WUServer = "http://WSUS01:8530"

if ($UseWSUS) {
    Set-GPRegistryValue `
      -Name $GpoName `
      -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate" `
      -ValueName "WUServer" `
      -Type String `
      -Value $WUServer

    Set-GPRegistryValue `
      -Name $GpoName `
      -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate" `
      -ValueName "WUStatusServer" `
      -Type String `
      -Value $WUServer

    Set-GPRegistryValue `
      -Name $GpoName `
      -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
      -ValueName "UseWUServer" `
      -Type DWord `
      -Value 1
}

# Defender:
# Do not turn off Defender Antivirus.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender" `
  -ValueName "DisableAntiSpyware" `
  -Type DWord `
  -Value 0

# Real-time protection baseline:
# Do not disable real-time monitoring.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" `
  -ValueName "DisableRealtimeMonitoring" `
  -Type DWord `
  -Value 0

# Scan downloaded files and attachments.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" `
  -ValueName "DisableIOAVProtection" `
  -Type DWord `
  -Value 0

# Enable behavior monitoring.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" `
  -ValueName "DisableBehaviorMonitoring" `
  -Type DWord `
  -Value 0

# Export reports.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-update-defender.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-update-defender.xml"

# Back up after configuration.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: prove Windows Update and Defender GPO settings apply.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Force policy refresh.
gpupdate /force

# Reboot if required for policy behavior.
# Restart-Computer

# Show applied computer-side GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-update-defender.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-update-defender.html"

# Optional RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-update-defender.html" `
  -ErrorAction SilentlyContinue

# Validate Windows Update policy registry paths.
Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate" `
  -ErrorAction SilentlyContinue |
  Out-File "$ReportPath\windowsupdate-policy-root.txt"

Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ErrorAction SilentlyContinue |
  Out-File "$ReportPath\windowsupdate-au-policy.txt"

# Validate Windows Update related services.
Get-Service `
  -Name wuauserv,bits `
  -ErrorAction SilentlyContinue |
  Select-Object Name,Status,StartType |
  Tee-Object "$ReportPath\windows-update-services.txt"

# Trigger an update detection cycle.
# This does not guarantee immediate install. It starts client-side scan behavior.
UsoClient StartScan

# Validate Defender services.
Get-Service `
  -Name WinDefend,SecurityHealthService `
  -ErrorAction SilentlyContinue |
  Select-Object Name,Status,StartType |
  Tee-Object "$ReportPath\defender-services.txt"

# Validate Defender status.
Get-MpComputerStatus |
  Out-File "$ReportPath\defender-computer-status.txt"

# Validate Defender preferences.
Get-MpPreference |
  Out-File "$ReportPath\defender-preferences.txt"

# Validate selected Defender registry paths.
Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows Defender" `
  -ErrorAction SilentlyContinue |
  Out-File "$ReportPath\defender-policy-root.txt"

Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" `
  -ErrorAction SilentlyContinue |
  Out-File "$ReportPath\defender-realtime-policy.txt"

# Review Windows Update related events from System log.
Get-WinEvent `
  -LogName System `
  -MaxEvents 200 |
  Where-Object {
    $_.ProviderName -like "*WindowsUpdate*" -or
    $_.ProviderName -like "*Update*"
  } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Out-File "$ReportPath\windows-update-events.txt"

# Review Defender operational events.
Get-WinEvent `
  -LogName "Microsoft-Windows-Windows Defender/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\defender-operational-events.txt"

# Review Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 150 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-update-defender.txt"
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final Update and Defender GPO report, inheritance, permissions, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$GpoName = "CORP-Workstation-Update-Defender"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-update-defender.txt"

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

# Search XML report for Update and Defender content.
Select-String `
  -Path "$ReportPath\$GpoName-final.xml" `
  -Pattern "WindowsUpdate","Windows Update","AUOptions","WUServer","Defender","Real-Time","DisableRealtimeMonitoring","DisableBehaviorMonitoring","DisableIOAVProtection" |
  Out-File "$ReportPath\$GpoName-update-defender-report-search.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-update-defender.csv" -NoTypeInformation

Write-Host "Windows Update and Defender GPO report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<update-defender-gpo>"` | Confirms Update and Defender GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-ou-dn>"` | Confirms GPO is linked to pilot OU | GPO appears in link list |
| `Get-GPPermission -Name "<update-defender-gpo>" -All` | Confirms target computers can read and apply GPO | Expected permissions appear |
| `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Confirms test computer is in pilot scope | Computer DN is under pilot OU |
| `Get-GPOReport -Name "<update-defender-gpo>" -ReportType Html -Path "<report-path>\<update-defender-gpo>.html"` | Exports readable GPO report | HTML report exists |
| `Get-GPOReport -Name "<update-defender-gpo>" -ReportType Xml -Path "<report-path>\<update-defender-gpo>.xml"` | Exports searchable GPO report | XML report exists |
| `Backup-GPO -Name "<update-defender-gpo>" -Path "<backup-path>"` | Backs up configured GPO | Backup completes |
| `gpupdate /force` | Forces computer policy refresh | Policy update completes |
| `gpresult /scope computer /r` | Shows applied computer GPOs | Update and Defender GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-update-defender.html` | Exports full RSOP report | HTML report exists |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate"` | Validates Windows Update policy root | Configured values appear |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU"` | Validates Automatic Updates policy | AU values appear |
| `Get-Service wuauserv,bits` | Checks update-related services | Services are visible and expected state is known |
| `UsoClient StartScan` | Starts update scan behavior | Command returns without visible failure |
| `Get-Service WinDefend,SecurityHealthService` | Checks Defender services | Services are visible |
| `Get-MpComputerStatus` | Shows Defender health and protection state | Defender status object returns |
| `Get-MpPreference` | Shows Defender preferences | Preference object returns |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows Defender"` | Validates Defender policy root | Configured values appear |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows Defender\Real-Time Protection"` | Validates Defender real-time policy | Real-time policy values appear |
| `Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 100` | Reviews Defender events | Defender events appear |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews Group Policy events | Recent policy processing events appear |

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely rollback Windows Update and Defender GPO settings.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Update-Defender"

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
  -Target $PilotOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Remove known Windows Update registry policy values.

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "NoAutoUpdate" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "AUOptions" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "ScheduledInstallDay" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "ScheduledInstallTime" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "NoAutoRebootWithLoggedOnUsers" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate" `
  -ValueName "WUServer" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate" `
  -ValueName "WUStatusServer" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU" `
  -ValueName "UseWUServer" `
  -ErrorAction SilentlyContinue

# Rollback option 3:
# Remove known Defender registry policy values.

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender" `
  -ValueName "DisableAntiSpyware" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" `
  -ValueName "DisableRealtimeMonitoring" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" `
  -ValueName "DisableIOAVProtection" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" `
  -ValueName "DisableBehaviorMonitoring" `
  -ErrorAction SilentlyContinue

# Rollback option 4:
# Use GPMC to set Update and Defender settings back to Not Configured where appropriate.
gpmc.msc

# GUI rollback workflow:
# 1. Open the GPO in Group Policy Management Editor.
# 2. Review Windows Update Administrative Template settings.
# 3. Set test settings back to Not Configured or approved baseline state.
# 4. Review Microsoft Defender Antivirus settings.
# 5. Remove unnecessary exclusions.
# 6. Export a fresh GPO report.
# 7. Run gpupdate /force on the test client.
# 8. Reboot if required.
# 9. Validate with gpresult, registry, Get-MpComputerStatus, and Get-MpPreference.

# Rollback option 5:
# Restore from known GPO backup if the backup path and target are confirmed.
# Restore-GPO `
#   -Name $GpoName `
#   -Path $BackupPath

# Capture after rollback.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-after-rollback.html" `
  -ErrorAction SilentlyContinue

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-update-defender-rollback.txt"

Write-Host "Rollback action complete."
Write-Host "On test client: run gpupdate /force, reboot if required, and validate Windows Update plus Defender state."
```

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Update and Defender GPO does not apply | Computer object is not under linked pilot OU | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Move computer to pilot OU or link GPO to correct OU |
| GPO appears denied | Security filtering or WMI filter blocks computer | `gpresult /scope computer /r`; `Get-GPPermission` | Grant Apply permission or fix WMI filter |
| Windows Update settings missing | Wrong GPO, wrong path, stale report, or policy not refreshed | `gpresult /h`; registry check | Confirm GPO scope and run `gpupdate /force` |
| Client scans wrong update source | WSUS and Windows Update for Business settings conflict | WindowsUpdate registry path and GPO report | Pick one update control model and remove conflicting settings |
| WSUS client cannot scan | WSUS URL wrong, server unreachable, firewall, or SSL mismatch | `Test-NetConnection WSUS01 -Port 8530`; registry check | Fix WSUS URL, service, firewall, or client source |
| Updates do not install on schedule | Schedule policy not configured, active hours, restart behavior, or client asleep | AU registry path and event logs | Correct schedule and restart policies |
| Client restarts unexpectedly | Restart policy too aggressive | GPO report and Windows Update events | Adjust active hours, deadlines, and no auto-restart policy |
| Defender cmdlets unavailable | Defender feature missing, OS edition difference, or third-party AV state | `Get-Service WinDefend`; `Get-Command Get-MpComputerStatus` | Confirm Defender availability and security product ownership |
| Defender real-time protection off | Policy disables it, tamper/security tooling conflict, or third-party AV | `Get-MpComputerStatus`; Defender registry path | Remove disabling policy and confirm management plane |
| Defender settings do not change | Managed by another product or modern security tooling | `Get-MpPreference`; Security Center status | Identify policy authority and avoid conflicting controls |
| Defender exclusions appear unexpectedly | Existing GPO, local policy, or another management plane | `Get-MpPreference | Select Exclusion*` | Remove unapproved exclusions from winning policy |
| Defender cloud protection not enabled | MAPS policy missing or privacy policy prevents it | `Get-MpPreference`; GPO report | Configure MAPS or document privacy decision |
| Security intelligence updates fail | Network, proxy, WSUS, or source order issue | Defender events and `Get-MpComputerStatus` | Fix update source and network path |
| GPO report does not show settings | Wrong GPO edited or stale report | `Get-GPOReport -Name "<update-defender-gpo>"` | Edit correct GPO and export fresh report |
| Registry policy exists but UI does not show expected state | UI delay, conflicting policy, or modern settings page limitation | `gpresult /h`; registry; event logs | Trust RSOP and registry, then test actual behavior |
| GPO differs between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before expanding scope |
| Windows Update service disabled | Local service baseline or other GPO disabled service | `Get-Service wuauserv,bits` | Correct service policy or baseline conflict |
| BITS disabled | Service policy or security baseline conflict | `Get-Service bits` | Enable required service behavior |
| Rollback does not change client immediately | Client needs refresh, reboot, or another GPO still wins | `gpupdate /force`; `gpresult /h` | Refresh, reboot, and identify winning GPO |
| Patch compliance not proven by this task alone | This task configures policy, not full update compliance reporting | Windows Update events and management platform reports | Add WSUS or reporting workflow in a monitoring workbook |

# 12_Configure_Windows_Update_And_Defender_GPO_Settings_Related_Labs
| Related Lab                                                           | Relationship                                                         |
| --------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `00_Group_Policy_Index.md`                                            | Defines suite order and dependency path                              |
| `01_Install_Group_Policy_Management_Tools.md`                         | Provides GPMC and GroupPolicy PowerShell module                      |
| `02_Create_And_Link_Baseline_GPO.md`                                  | Establishes GPO creation and linking workflow                        |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before update and Defender policy rollout        |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md`               | Controls which computers can apply the Update and Defender GPO       |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md`                     | Adds optional OS targeting before policy application                 |
| `06_Configure_Computer_Administrative_Template_Settings.md`           | Provides ADMX-backed computer policy foundation                      |
| `11_Configure_Security_Baseline_GPO_Settings.md`                      | Security baseline companion workbook                                 |
| `Create_Baseline_OU_Structure.md`                                     | Provides pilot and workstation OU targets                            |
| `Create_Baseline_Groups_And_Test_Users.md`                            | Provides test computer and filtering group                           |
| `Join_Windows_Client_To_Domain.md`                                    | Provides domain-joined test client for validation                    |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md`                 | Confirms SYSVOL and DC locator before troubleshooting GPO processing |
| `13_Configure_AppLocker_And_Software_Restriction_Policies.md`         | Next workbook for application control policy                         |