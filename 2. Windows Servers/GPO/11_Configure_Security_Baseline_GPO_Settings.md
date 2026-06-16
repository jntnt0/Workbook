11_Configure_Security_Baseline_GPO_Settings.md
# 11_Configure_Security_Baseline_GPO_Settings

# 11_Configure_Security_Baseline_GPO_Settings_Index
11_Configure_Security_Baseline_GPO_Settings.md
11_Configure_Security_Baseline_GPO_Settings
11_Configure_Security_Baseline_GPO_Settings_Source_Basis
11_Configure_Security_Baseline_GPO_Settings_Mental_Model
11_Configure_Security_Baseline_GPO_Settings_Planning_Table
11_Configure_Security_Baseline_GPO_Settings_Baseline_Settings_Map
11_Configure_Security_Baseline_GPO_Settings_Configuration_Checklist
11_Configure_Security_Baseline_GPO_Settings_Precheck_Skeleton
11_Configure_Security_Baseline_GPO_Settings_Create_And_Link_GPO_Skeleton
11_Configure_Security_Baseline_GPO_Settings_GPMC_Security_Settings_Skeleton
11_Configure_Security_Baseline_GPO_Settings_Admin_Template_Hardening_Skeleton
11_Configure_Security_Baseline_GPO_Settings_Client_RSOP_Validation_Skeleton
11_Configure_Security_Baseline_GPO_Settings_Report_And_Backup_Skeleton
11_Configure_Security_Baseline_GPO_Settings_Verification_Commands
11_Configure_Security_Baseline_GPO_Settings_Rollback
11_Configure_Security_Baseline_GPO_Settings_Failure_Checks
11_Configure_Security_Baseline_GPO_Settings_Related_Labs

# 11_Configure_Security_Baseline_GPO_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy security settings | Account policies, local policies, audit policy, user rights, and security options |
| Microsoft Learn | Group Policy Management Console | Editing baseline security settings in a domain GPO |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up the security baseline GPO |
| Microsoft Learn | Advanced audit policy configuration | Configuring audit categories and subcategories through Group Policy |
| Microsoft Learn | Windows security options | Configuring local security behavior through GPO |
| Microsoft Learn | Windows Defender Firewall with Advanced Security | Baseline firewall profile configuration through GPO |
| Microsoft Security Baselines | Security baseline design reference | Selecting sane hardening categories without inventing random policy |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, `secedit`, `auditpol`, and event logs | Proving security baseline settings apply on a test client |
| Windows Server operational practice | Pilot security hardening before production rollout | Preventing lockouts, broken admin access, or application outages |

# 11_Configure_Security_Baseline_GPO_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Security baseline GPO | GPO used to apply a controlled set of security settings to a target computer population |
| Computer security baseline | Computer-side hardening settings applied to workstation or server computer accounts |
| Account Policies | Password, account lockout, and Kerberos settings |
| Local Policies | Audit policy, user rights assignment, and security options |
| Security Options | Local security behavior settings such as UAC, logon, network security, and device behavior |
| Advanced Audit Policy | Detailed audit subcategories that control what Windows records in the Security log |
| User Rights Assignment | Controls who can log on locally, log on through RDP, shut down, back up files, debug programs, and perform other rights |
| Restricted Groups | Legacy GPO method for local group membership control |
| Local Users and Groups Preference | GPP method for managing local group membership |
| Windows Defender Firewall policy | Domain, Private, and Public firewall profile settings configured through GPO |
| Administrative Template hardening | ADMX-backed security settings under Computer Configuration |
| Security filtering | Controls which computers can apply the baseline |
| WMI filtering | Optional condition that can restrict the baseline by OS or hardware criteria |
| RSOP | Effective security policy result after all GPO precedence and targeting rules |
| secedit | Local tool for exporting effective security policy configuration |
| auditpol | Local tool for validating effective audit policy |
| First rule | Apply security baselines to a pilot OU before touching broad workstation or server OUs |
| Blunt rule | A security baseline can lock you out or break apps, so always back up, report, pilot, and document before expanding scope |

# 11_Configure_Security_Baseline_GPO_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target computer OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-computer-ou-dn>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Security baseline GPO | `CORP-Workstation-Security-Baseline` | `<security-baseline-gpo>` |
| Security filtering group | `GG_GPO_Workstation_Security_Baseline_Apply` | `<filtering-group>` |
| Admin exception group | `GG_Workstation_Local_Admins` | `<local-admin-group>` |
| RDP allowed group | `GG_Workstation_RDP_Users` | `<rdp-group>` |
| Baseline target | Workstations | `<workstations-or-servers>` |
| Link enabled | Yes | `<yes-no>` |
| Enforced | No | `<yes-no>` |
| WMI filter | None by default | `<wmi-filter-name-or-none>` |
| Password policy scope | Domain-level only unless local accounts are targeted | `<domain-or-local>` |
| Account lockout policy | Defined at domain level or local baseline | `<decision>` |
| Audit policy | Advanced audit policy | `<audit-policy-mode>` |
| Firewall profile stance | Enabled for Domain, Private, and Public | `<firewall-profile-decision>` |
| UAC stance | Enabled | `<uac-decision>` |
| SMB guest stance | Disabled | `<smb-guest-decision>` |
| Anonymous enumeration stance | Restricted | `<anonymous-decision>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Disable link, restore backup, or remove specific settings | `<rollback-plan>` |

# 11_Configure_Security_Baseline_GPO_Settings_Baseline_Settings_Map
| Category | GPMC Path | Example Setting | Example Baseline | Validation |
|---|---|---|---|---|
| Account Lockout | `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy` | Account lockout threshold | Define only when appropriate for scope | `secedit /export` or RSOP |
| Password Policy | `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy` | Minimum password length | Usually domain policy, not workstation OU GPO | Domain policy and RSOP |
| Kerberos Policy | `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Kerberos Policy` | Ticket lifetime settings | Usually domain policy only | Domain policy and RSOP |
| Audit Policy | `Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration` | Logon, Account Logon, Object Access, Policy Change | Success and Failure where required | `auditpol /get /category:*` |
| User Rights | `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment` | Allow log on through Remote Desktop Services | Admin or approved RDP group only | `secedit /export` |
| Security Options | `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options` | Do not allow anonymous enumeration of SAM accounts | Enabled | `secedit /export` |
| UAC | `Security Options` and Administrative Templates | Run all administrators in Admin Approval Mode | Enabled | Registry and RSOP |
| Firewall | `Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security` | Domain profile firewall state | On | `Get-NetFirewallProfile` |
| Defender AV | `Computer Configuration > Policies > Administrative Templates > Microsoft Defender Antivirus` | Turn off Microsoft Defender Antivirus | Disabled or Not Configured | GPO report and Defender status |
| Remote Assistance | `Computer Configuration > Policies > Administrative Templates > System > Remote Assistance` | Configure Solicited Remote Assistance | Disabled unless required | GPO report |
| Windows Installer | `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Installer` | Always install with elevated privileges | Disabled | Registry and GPO report |
| AutoRun | `Computer Configuration > Policies > Administrative Templates > Windows Components > AutoPlay Policies` | Turn off AutoPlay | Enabled | Registry and RSOP |
| Local Admin Control | `Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups` | Local Administrators membership | Controlled by approved group | `Get-LocalGroupMember Administrators` |
| Event Logs | `Computer Configuration > Policies > Windows Settings > Security Settings > Event Log` | Security log max size | Set to baseline value | `wevtutil gl Security` |

# 11_Configure_Security_Baseline_GPO_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target computer OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-computer-ou-dn>"` | Target computer OU returns |
| 6 | Confirm pilot OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-ou-dn>"` | Pilot OU returns |
| 7 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 8 | Confirm test computer is in pilot OU | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under pilot OU |
| 9 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 11 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 12 | Back up existing GPOs before changes | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 13 | Create security baseline GPO if missing | Management Host | `New-GPO -Name "<security-baseline-gpo>" -Comment "Workstation security baseline settings"` | GPO exists |
| 14 | Link security baseline GPO to pilot OU | Management Host | `New-GPLink -Name "<security-baseline-gpo>" -Target "<pilot-ou-dn>" -LinkEnabled Yes` | GPO is linked to pilot OU |
| 15 | Keep baseline unenforced by default | Management Host | `Set-GPLink -Name "<security-baseline-gpo>" -Target "<pilot-ou-dn>" -Enforced No` | Link is not enforced |
| 16 | Confirm link state | Management Host | `Get-GPInheritance -Target "<pilot-ou-dn>"` | Security baseline GPO appears in link list |
| 17 | Confirm GPO permissions | Management Host | `Get-GPPermission -Name "<security-baseline-gpo>" -All` | Read and Apply permissions are known |
| 18 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<security-baseline-gpo>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 19 | Preserve required read access | Management Host | `Set-GPPermission -Name "<security-baseline-gpo>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | Authenticated Users can read GPO |
| 20 | Open GPMC editor | Management Host | `gpmc.msc` | GPMC opens |
| 21 | Configure Account Lockout only if intentionally scoped | Management Host | `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy` | Account lockout behavior is configured or intentionally left to domain policy |
| 22 | Configure Advanced Audit Policy | Management Host | `Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration` | Audit subcategories are configured |
| 23 | Configure User Rights Assignment | Management Host | `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment` | Logon and privilege rights are configured |
| 24 | Configure Security Options | Management Host | `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options` | Local security behavior is configured |
| 25 | Configure Windows Defender Firewall baseline | Management Host | `Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security` | Firewall profiles are configured |
| 26 | Configure Event Log baseline if required | Management Host | `Computer Configuration > Policies > Windows Settings > Security Settings > Event Log` | Event log size and retention policy are configured |
| 27 | Configure Defender Administrative Template baseline | Management Host | `Computer Configuration > Policies > Administrative Templates > Microsoft Defender Antivirus` | Defender hardening settings are configured |
| 28 | Configure AutoPlay and AutoRun hardening | Management Host | `Computer Configuration > Policies > Administrative Templates > Windows Components > AutoPlay Policies` | AutoPlay baseline is configured |
| 29 | Configure Windows Installer hardening | Management Host | `Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Installer` | Unsafe installer elevation behavior is disabled |
| 30 | Configure local group membership if required | Management Host | `Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups` | Local Administrators membership is controlled |
| 31 | Export baseline GPO report | Management Host | `Get-GPOReport -Name "<security-baseline-gpo>" -ReportType Html -Path C:\GPOPrep\Reports\<security-baseline-gpo>.html` | Report shows configured settings |
| 32 | Export XML report for searching | Management Host | `Get-GPOReport -Name "<security-baseline-gpo>" -ReportType Xml -Path C:\GPOPrep\Reports\<security-baseline-gpo>.xml` | XML report exists |
| 33 | Back up configured GPO | Management Host | `Backup-GPO -Name "<security-baseline-gpo>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 34 | Force client policy refresh | Test Client | `gpupdate /force` | Computer policy refresh completes |
| 35 | Reboot test client if required | Test Client | `Restart-Computer` | Startup-applied settings refresh |
| 36 | Validate applied computer GPOs | Test Client | `gpresult /scope computer /r` | Security baseline GPO appears under Applied GPOs |
| 37 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-security-baseline.html` | HTML RSOP report exists |
| 38 | Validate audit policy | Test Client | `auditpol /get /category:*` | Expected audit subcategories are configured |
| 39 | Export effective security policy | Test Client | `secedit /export /cfg C:\GPOPrep\Reports\effective-security-policy.inf` | Effective security policy export exists |
| 40 | Validate firewall profiles | Test Client | `Get-NetFirewallProfile` | Firewall profiles show expected state |
| 41 | Validate local Administrators group | Test Client | `Get-LocalGroupMember -Group Administrators` | Expected local admin membership appears |
| 42 | Review Group Policy events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Recent processing events are visible |
| 43 | Review security log activity if audit enabled | Test Client | `Get-WinEvent -LogName Security -MaxEvents 50` | Security events are visible |
| 44 | Document baseline result | Operator | `Record GPO, pilot OU, settings categories, reports, backup, test computer, and validation output` | Security baseline deployment is documented |

# 11_Configure_Security_Baseline_GPO_Settings_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring a security baseline GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"

$GpoName = "CORP-Workstation-Security-Baseline"

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

# Confirm target OU and pilot OU.
Get-ADOrganizationalUnit -Identity $TargetComputerOU
Get-ADOrganizationalUnit -Identity $PilotOU

# Confirm test computer exists and is in pilot scope.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-before-security-baseline.txt"

# Capture current inheritance.
Get-GPInheritance `
  -Target $TargetComputerOU |
  Out-File "$ReportPath\target-computer-ou-inheritance-before-security-baseline.txt"

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-security-baseline.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-security-baseline.csv" -NoTypeInformation

# Back up current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath
```

# 11_Configure_Security_Baseline_GPO_Settings_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link a dedicated security baseline GPO to a pilot OU.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Security-Baseline"
$GpoComment = "Pilot workstation security baseline GPO. Expand scope only after validation."

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

# 11_Configure_Security_Baseline_GPO_Settings_GPMC_Security_Settings_Skeleton
```powershell
# Native GUI workflow for configuring Windows Security Settings in a baseline GPO.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-Workstation-Security-Baseline
# 7. Select Edit.
#
# Security Settings path:
# 8. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#
# Account Policies:
# 9. Review:
#      Account Policies
#      > Password Policy
#      > Account Lockout Policy
#      > Kerberos Policy
# 10. In most AD environments:
#      Password and Kerberos policy are normally controlled at the domain level.
#      Avoid setting conflicting values in a workstation OU baseline unless there is a specific design reason.
#
# Advanced Audit Policy:
# 11. Go to:
#      Advanced Audit Policy Configuration
#      > Audit Policies
# 12. Configure baseline audit categories such as:
#      Account Logon
#      Account Management
#      Detailed Tracking
#      Logon/Logoff
#      Object Access
#      Policy Change
#      Privilege Use
#      System
#
# Local Policies:
# 13. Go to:
#      Local Policies
#      > User Rights Assignment
# 14. Configure only approved rights.
# 15. Example rights to review:
#      Allow log on locally
#      Allow log on through Remote Desktop Services
#      Deny log on locally
#      Deny log on through Remote Desktop Services
#      Shut down the system
#      Debug programs
#      Back up files and directories
#      Restore files and directories
#
# Security Options:
# 16. Go to:
#      Local Policies
#      > Security Options
# 17. Review and configure baseline options such as:
#      Accounts: Guest account status
#      Accounts: Limit local account use of blank passwords to console logon only
#      Interactive logon: Do not display last user name
#      Network access: Do not allow anonymous enumeration of SAM accounts
#      Network access: Do not allow anonymous enumeration of SAM accounts and shares
#      Network security: LAN Manager authentication level
#      User Account Control settings
#
# Windows Defender Firewall:
# 18. Go to:
#      Windows Defender Firewall with Advanced Security
# 19. Configure:
#      Domain Profile
#      Private Profile
#      Public Profile
# 20. Set firewall state and logging behavior according to baseline.
#
# Event Log:
# 21. Go to:
#      Event Log
# 22. Configure log size and retention if required.
#
# Close editor.
# Refresh GPMC.
# Export a GPO report.
```

# 11_Configure_Security_Baseline_GPO_Settings_Admin_Template_Hardening_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: configure known ADMX-backed baseline hardening registry policy values.
# Use only values that are approved and understood.

Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Security-Baseline"
$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm GPO exists.
Get-GPO -Name $GpoName

# Back up before registry-policy changes.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Baseline: turn off AutoPlay for all drives.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoDriveTypeAutoRun" `
  -Type DWord `
  -Value 255

# Baseline: turn off Microsoft consumer experiences.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\CloudContent" `
  -ValueName "DisableWindowsConsumerFeatures" `
  -Type DWord `
  -Value 1

# Baseline: always wait for network at computer startup and logon.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -ValueName "SyncForegroundPolicy" `
  -Type DWord `
  -Value 1

# Baseline: disable Always install with elevated privileges for machine side.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\Installer" `
  -ValueName "AlwaysInstallElevated" `
  -Type DWord `
  -Value 0

# Optional: configure local group membership through GPP or Restricted Groups in GPMC.
# Use GPMC for local group membership because accidental overwrite can lock out support access.

# Export report after registry-policy configuration.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-admin-template-hardening.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-admin-template-hardening.xml"

# Back up after configuration.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath
```

# 11_Configure_Security_Baseline_GPO_Settings_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: prove the security baseline applies and inspect effective security state.

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

# Reboot if security settings require startup processing.
# Restart-Computer

# Show applied computer-side GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-security-baseline.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-security-baseline.html"

# Optional RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-security-baseline.html" `
  -ErrorAction SilentlyContinue

# Export effective security policy.
secedit /export /cfg "$ReportPath\effective-security-policy.inf"

# Validate audit policy.
auditpol /get /category:* |
  Out-File "$ReportPath\auditpol-effective.txt"

# Validate firewall profiles.
Get-NetFirewallProfile |
  Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction,LogAllowed,LogBlocked,LogFileName |
  Tee-Object "$ReportPath\firewall-profiles.txt"

# Validate local Administrators membership.
Get-LocalGroupMember `
  -Group Administrators `
  -ErrorAction SilentlyContinue |
  Select-Object Name,ObjectClass,PrincipalSource |
  Out-File "$ReportPath\local-administrators-members.txt"

# Validate selected registry policy values.
Get-ItemProperty `
  -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ErrorAction SilentlyContinue |
  Select-Object NoDriveTypeAutoRun |
  Out-File "$ReportPath\policy-autoplay.txt"

Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\CloudContent" `
  -ErrorAction SilentlyContinue |
  Select-Object DisableWindowsConsumerFeatures |
  Out-File "$ReportPath\policy-cloudcontent.txt"

Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -ErrorAction SilentlyContinue |
  Select-Object SyncForegroundPolicy |
  Out-File "$ReportPath\policy-winlogon.txt"

Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\Installer" `
  -ErrorAction SilentlyContinue |
  Select-Object AlwaysInstallElevated |
  Out-File "$ReportPath\policy-installer.txt"

# Review Group Policy processing events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 150 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-security-baseline.txt"

# Review recent Security log entries.
Get-WinEvent `
  -LogName Security `
  -MaxEvents 50 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,Message |
  Out-File "$ReportPath\recent-security-events.txt"
```

# 11_Configure_Security_Baseline_GPO_Settings_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final security baseline GPO report, inheritance, permissions, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$GpoName = "CORP-Workstation-Security-Baseline"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-security-baseline.txt"

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

# Search XML report for security baseline content.
Select-String `
  -Path "$ReportPath\$GpoName-final.xml" `
  -Pattern "Security","Audit","Firewall","UserRights","Privilege","NoDriveTypeAutoRun","DisableWindowsConsumerFeatures","AlwaysInstallElevated","SyncForegroundPolicy" |
  Out-File "$ReportPath\$GpoName-security-baseline-report-search.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-security-baseline.csv" -NoTypeInformation

Write-Host "Security baseline GPO report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 11_Configure_Security_Baseline_GPO_Settings_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<security-baseline-gpo>"` | Confirms security baseline GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-ou-dn>"` | Confirms GPO is linked to pilot OU | GPO appears in link list |
| `Get-GPPermission -Name "<security-baseline-gpo>" -All` | Confirms target computers can read and apply GPO | Expected permissions appear |
| `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Confirms test computer is in pilot scope | Computer DN is under pilot OU |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms SYSVOL is reachable | Returns True |
| `Get-GPOReport -Name "<security-baseline-gpo>" -ReportType Html -Path "<report-path>\<security-baseline-gpo>.html"` | Exports readable baseline report | HTML report exists |
| `Get-GPOReport -Name "<security-baseline-gpo>" -ReportType Xml -Path "<report-path>\<security-baseline-gpo>.xml"` | Exports searchable baseline report | XML report exists |
| `Backup-GPO -Name "<security-baseline-gpo>" -Path "<backup-path>"` | Backs up configured baseline GPO | Backup completes |
| `gpupdate /force` | Forces client policy refresh | Computer policy update completes |
| `gpresult /scope computer /r` | Shows applied computer GPOs | Security baseline GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-security-baseline.html` | Exports full RSOP report | HTML report exists |
| `secedit /export /cfg C:\GPOPrep\Reports\effective-security-policy.inf` | Exports effective security policy | INF export exists |
| `auditpol /get /category:*` | Shows effective audit policy | Expected audit subcategories appear |
| `Get-NetFirewallProfile` | Shows firewall profile state | Expected profile settings appear |
| `Get-LocalGroupMember -Group Administrators` | Validates local administrator membership | Expected groups/users appear |
| `Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"` | Validates AutoPlay hardening | `NoDriveTypeAutoRun` appears |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\CloudContent"` | Validates Cloud Content hardening | `DisableWindowsConsumerFeatures` appears |
| `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\Installer"` | Validates installer hardening | `AlwaysInstallElevated` is absent or `0` |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews GPO processing events | Recent successful processing events appear |
| `Get-WinEvent -LogName Security -MaxEvents 50` | Reviews Security log activity | Security events are visible if auditing is enabled |

# 11_Configure_Security_Baseline_GPO_Settings_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely rollback security baseline changes.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Security-Baseline"

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
# Disable the link while preserving the GPO for review.
Set-GPLink `
  -Name $GpoName `
  -Target $PilotOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Remove known registry-based Administrative Template hardening values.

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoDriveTypeAutoRun" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\CloudContent" `
  -ValueName "DisableWindowsConsumerFeatures" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -ValueName "SyncForegroundPolicy" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\Installer" `
  -ValueName "AlwaysInstallElevated" `
  -ErrorAction SilentlyContinue

# Rollback option 3:
# Use GPMC to remove Security Settings, User Rights Assignment, Firewall, and Audit Policy changes.
gpmc.msc

# GUI rollback workflow:
# 1. Open the baseline GPO in Group Policy Management Editor.
# 2. Review Security Settings categories.
# 3. Set risky settings back to Not Configured where appropriate.
# 4. Review User Rights Assignment carefully before removing entries.
# 5. Review firewall policy before disabling or relaxing profiles.
# 6. Export a fresh GPO report.
# 7. Run gpupdate /force on the test client.
# 8. Reboot the test client if required.
# 9. Validate with gpresult, secedit, auditpol, and firewall checks.

# Rollback option 4:
# Restore from a known GPO backup if the backup set and target are confirmed.
# Restore-GPO `
#   -Name $GpoName `
#   -Path $BackupPath

# Rollback option 5:
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
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-security-baseline-rollback.txt"

Write-Host "Rollback action complete."
Write-Host "On test client: run gpupdate /force, reboot if required, and validate with gpresult, secedit, auditpol, and firewall checks."
```

# 11_Configure_Security_Baseline_GPO_Settings_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Security baseline GPO does not apply | Computer object is not under linked pilot OU | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Move computer to pilot OU or link GPO to correct OU |
| GPO appears denied | Security filtering or WMI filter blocks computer | `gpresult /scope computer /r`; `Get-GPPermission` | Grant Apply permission or fix WMI filter |
| Security settings do not appear in RSOP | GPO not applied, wrong policy side, or setting not configured | `gpresult /h`; GPO report | Confirm Computer Configuration settings and scope |
| Account policy settings do not behave as expected | Domain account policy precedence misunderstood | Domain policy report and RSOP | Configure domain account policy intentionally, not randomly in workstation OU |
| User rights setting locks out admin access | Overly restrictive User Rights Assignment | `secedit /export`; local sign-in test | Disable link or restore backup immediately |
| RDP stops working | RDP user right or firewall rule changed | `gpresult /r`; `Get-NetFirewallProfile`; RDP event logs | Restore allowed RDP group and firewall rules |
| Firewall blocks management tools | Firewall profile or inbound rules too restrictive | `Get-NetFirewallProfile`; `Get-NetFirewallRule` | Add required management rules or disable pilot link |
| Local admin membership overwritten | Restricted Groups or Local Users and Groups preference configured too aggressively | `Get-LocalGroupMember Administrators` | Restore local admin access and correct membership model |
| Audit policy does not match | Advanced audit policy conflict or local policy override | `auditpol /get /category:*`; `gpresult /h` | Confirm advanced audit policy and precedence |
| Security log fills quickly | Too much auditing enabled or log size too small | `wevtutil gl Security`; event volume | Tune audit categories and increase Security log size |
| UAC behavior breaks admin workflow | UAC security option changed | `secedit /export`; GPO report | Revert UAC setting or pilot with admin process documented |
| App installation breaks | Windows Installer hardening too restrictive | Installer logs and GPO report | Adjust installer policy or deploy software through approved method |
| AutoPlay setting missing | Wrong registry path or GPO not applied | `Get-ItemProperty`; `gpresult /r` | Correct setting or scope |
| GPO report does not show setting | Wrong GPO edited or stale report | `Get-GPOReport -Name "<security-baseline-gpo>"` | Edit correct GPO and export fresh report |
| GPO applies to too many computers | Link target too broad or filtering too open | `Get-GPInheritance`; `Get-GPPermission` | Link to pilot OU first and narrow filtering |
| GPO differs between domain controllers | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before expanding scope |
| Client still has old security settings after rollback | Policy refresh or reboot required | `gpupdate /force`; `Restart-Computer` | Refresh policy and reboot |
| Security setting remains after GPO removal | Local setting persists or another GPO configures it | `gpresult /h`; `secedit /export` | Find winning GPO and set desired state |
| Baseline breaks application | Hardening conflict with app requirement | Event logs, app logs, GPO report | Disable link, isolate setting, then create exception or app-specific policy |
| No clear proof of applied baseline | Validation artifacts not collected | `gpresult`, `secedit`, `auditpol`, firewall output | Collect reports before moving beyond pilot |

# 11_Configure_Security_Baseline_GPO_Settings_Related_Labs
| Related Lab                                                            | Relationship                                                                    |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `00_Group_Policy_Index.md`                                             | Defines suite order and dependency path                                         |
| `01_Install_Group_Policy_Management_Tools.md`                          | Provides GPMC and GroupPolicy PowerShell module                                 |
| `02_Create_And_Link_Baseline_GPO.md`                                   | Establishes GPO creation and linking workflow                                   |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md`  | Controls precedence before security hardening                                   |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md`                | Controls which computers can apply the baseline                                 |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md`                      | Adds optional OS or hardware targeting before baseline application              |
| `06_Configure_Computer_Administrative_Template_Settings.md`            | Computer Administrative Template foundation for baseline hardening              |
| `08_Configure_Group_Policy_Preferences.md`                             | Supports local group management and other preference-based security items       |
| `09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md`            | Script alternative for hardening that cannot be done cleanly through policy     |
| `10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md` | User resource workbook that should be tested separately from security hardening |
| `Create_Baseline_OU_Structure.md`                                      | Provides pilot and workstation OU targets                                       |
| `Create_Baseline_Groups_And_Test_Users.md`                             | Provides test computer, groups, and filtering principals                        |
| `Join_Windows_Client_To_Domain.md`                                     | Provides domain-joined test client for security baseline validation             |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md`                  | Confirms SYSVOL and DC locator before troubleshooting baseline application      |
| `12_Configure_Windows_Defender_Firewall_With_GPO.md`                   | Next workbook for deeper firewall-specific baseline configuration               |