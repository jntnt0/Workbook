20_Configure_LAPS_And_Local_Administrator_Password_Policy.md
# 20_Configure_LAPS_And_Local_Administrator_Password_Policy

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Index
20_Configure_LAPS_And_Local_Administrator_Password_Policy.md
20_Configure_LAPS_And_Local_Administrator_Password_Policy
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Source_Basis
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Mental_Model
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Planning_Table
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Control_Map
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Configuration_Checklist
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Precheck_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_AD_Schema_And_Permissions_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Create_And_Link_GPO_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Windows_LAPS_GPMC_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Windows_LAPS_PowerShell_GPO_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Client_Refresh_And_Password_Backup_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Read_Reset_And_Rotate_Password_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Report_And_Backup_Skeleton
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Verification_Commands
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Rollback
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Failure_Checks
20_Configure_LAPS_And_Local_Administrator_Password_Policy_Related_Labs

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Windows LAPS overview | Local administrator password rotation and backup model |
| Microsoft Learn | Windows LAPS Active Directory schema | Extending AD DS schema for Windows LAPS attributes |
| Microsoft Learn | Windows LAPS PowerShell cmdlets | `Update-LapsADSchema`, `Set-LapsADComputerSelfPermission`, `Get-LapsADPassword`, and reset workflows |
| Microsoft Learn | Windows LAPS Group Policy | Configuring Windows LAPS policy through Administrative Templates |
| Microsoft Learn | Group Policy Management Console | Linking LAPS policy to target computer OUs |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up GPOs |
| Microsoft Learn | Active Directory permissions | Delegating read and reset access to LAPS passwords |
| Windows Server operational practice | Separate LAPS password readers from GPO editors and domain admins | Reducing local admin credential exposure |

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Windows LAPS | Built-in Windows feature for rotating and backing up local administrator passwords |
| Legacy Microsoft LAPS | Older MSI-based LAPS product using legacy AD attributes and policy paths |
| Local administrator password | Password for a local admin account on each endpoint |
| Password uniqueness | Each managed computer has a different local admin password |
| Password rotation | Client changes the local admin password on schedule or on demand |
| Password backup | Client writes the managed password to AD DS or Entra ID depending on policy |
| AD DS backup | Password is stored in Active Directory attributes on the computer object |
| Schema extension | AD schema update that adds Windows LAPS attributes |
| Self permission | Permission allowing computer objects to write their own LAPS password attributes |
| Read permission | Delegated permission allowing approved staff to read LAPS passwords |
| Reset permission | Delegated permission allowing approved staff to force password expiration or reset |
| Encrypted password | Windows LAPS can store encrypted passwords in AD DS when configured |
| Password history | Optional encrypted password history when encryption is configured |
| Post-authentication action | Optional action after password is used, such as reset password, log off account, or reboot |
| Managed account | Built-in Administrator or named local admin account managed by LAPS |
| First rule | Configure schema, permissions, and pilot OU before linking LAPS policy broadly |
| Blunt rule | Never give broad LAPS read access to helpdesk groups without auditing and approval |

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-ou-dn>` |
| Target workstation OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-workstation-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| LAPS GPO | `CORP-Workstation-Windows-LAPS` | `<laps-gpo-name>` |
| Backup directory | Active Directory | `<ad-or-entra>` |
| Managed local admin account | Built-in Administrator or `LocalAdmin` | `<managed-account-name>` |
| Password length | `16` or higher | `<password-length>` |
| Password complexity | Large letters, small letters, numbers, special characters | `<password-complexity>` |
| Password age | `30` days | `<password-age-days>` |
| Password encryption | Enabled if supported and delegated correctly | `<yes-no>` |
| Password readers group | `GG_LAPS_Password_Readers` | `<reader-group>` |
| Password resetters group | `GG_LAPS_Password_Resetters` | `<resetter-group>` |
| LAPS admins group | `GG_LAPS_Admins` | `<laps-admin-group>` |
| Computer self permission scope | Pilot OU first | `<permission-scope>` |
| Post-authentication action | Reset password after delay | `<post-auth-action>` |
| Post-auth reset delay | `8` hours | `<post-auth-delay>` |
| Report path | `C:\GPOPrep\Reports\LAPS` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup\LAPS` | `<backup-path>` |
| Rollback method | Disable GPO link, remove policy values, restore GPO backup | `<rollback-plan>` |

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Control_Map
| Control | Tool / Location | Purpose | Validation |
|---|---|---|---|
| Windows LAPS schema | `Update-LapsADSchema` | Adds Windows LAPS AD attributes | Cmdlet completes and attributes exist |
| Computer self permission | `Set-LapsADComputerSelfPermission` | Allows computers to write LAPS password data to their AD object | Cmdlet completes for target OU |
| Read password permission | `Set-LapsADReadPasswordPermission` | Allows approved group to retrieve LAPS passwords | `Get-LapsADPassword` succeeds for approved group |
| Reset password permission | `Set-LapsADResetPasswordPermission` | Allows approved group to force password rotation | `Reset-LapsPassword` or expiration workflow succeeds |
| Extended rights audit | `Find-LapsADExtendedRights` | Finds principals with LAPS password read access | Only expected groups appear |
| LAPS GPO | `Computer Configuration > Policies > Administrative Templates > System > LAPS` | Configures Windows LAPS client behavior | GPO report and registry policy values |
| Backup directory | LAPS policy | Selects AD DS or Entra ID backup | Client writes password to selected directory |
| Password length | LAPS policy | Sets generated password length | GPO report and policy registry |
| Password age | LAPS policy | Controls rotation interval | AD password expiration timestamp |
| Password complexity | LAPS policy | Controls generated character set | GPO report and password behavior |
| Administrator account name | LAPS policy | Targets named local admin account if not built-in | Local account exists and is managed |
| Password encryption | LAPS policy | Encrypts AD-stored password data | Encrypted attributes and proper reader delegation |
| Post-auth action | LAPS policy | Rotates or controls account after password retrieval/use | Event logs and new expiration |
| Client validation | `gpupdate`, `Get-LapsADPassword`, Event Viewer | Proves policy applies and password is backed up | Password data is retrievable by approved admin |

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host / DC | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host / DC | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host / DC | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Import LAPS module | Management Host / DC | `Import-Module LAPS` | LAPS module imports |
| 5 | Confirm domain identity | Management Host / DC | `Get-ADDomain` | Domain object returns |
| 6 | Confirm SYSVOL access | Management Host / DC | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 7 | Confirm pilot OU exists | Management Host / DC | `Get-ADOrganizationalUnit -Identity "<pilot-ou-dn>"` | Pilot OU returns |
| 8 | Confirm test computer exists | Management Host / DC | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 9 | Confirm test computer is in pilot OU | Management Host / DC | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under pilot OU |
| 10 | Create report folder | Management Host / DC | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\LAPS` | Report path exists |
| 11 | Create backup folder | Management Host / DC | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup\LAPS` | Backup path exists |
| 12 | Back up current GPO state | Management Host / DC | `Backup-GPO -All -Path C:\GPOPrep\Backup\LAPS` | Pre-change backup exists |
| 13 | Confirm LAPS cmdlets are available | Management Host / DC | `Get-Command -Module LAPS` | LAPS cmdlets return |
| 14 | Extend AD schema for Windows LAPS | Schema Master DC | `Update-LapsADSchema` | Schema update completes |
| 15 | Delegate computer self permission | Management Host / DC | `Set-LapsADComputerSelfPermission -Identity "<pilot-ou-dn>"` | Computers can write LAPS attributes |
| 16 | Create LAPS reader group if missing | Management Host / DC | `New-ADGroup -Name "GG_LAPS_Password_Readers" -GroupScope Global -GroupCategory Security -Path "<groups-ou-dn>"` | Reader group exists |
| 17 | Create LAPS resetter group if missing | Management Host / DC | `New-ADGroup -Name "GG_LAPS_Password_Resetters" -GroupScope Global -GroupCategory Security -Path "<groups-ou-dn>"` | Resetter group exists |
| 18 | Delegate LAPS password read permission | Management Host / DC | `Set-LapsADReadPasswordPermission -Identity "<pilot-ou-dn>" -AllowedPrincipals "GG_LAPS_Password_Readers"` | Reader group can read LAPS password |
| 19 | Delegate LAPS reset permission | Management Host / DC | `Set-LapsADResetPasswordPermission -Identity "<pilot-ou-dn>" -AllowedPrincipals "GG_LAPS_Password_Resetters"` | Resetter group can reset LAPS password |
| 20 | Audit extended rights | Management Host / DC | `Find-LapsADExtendedRights -Identity "<pilot-ou-dn>"` | Only approved principals appear |
| 21 | Create LAPS GPO | Management Host / DC | `New-GPO -Name "<laps-gpo-name>" -Comment "Windows LAPS policy for pilot workstations"` | GPO exists |
| 22 | Link LAPS GPO to pilot OU | Management Host / DC | `New-GPLink -Name "<laps-gpo-name>" -Target "<pilot-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 23 | Keep LAPS GPO unenforced by default | Management Host / DC | `Set-GPLink -Name "<laps-gpo-name>" -Target "<pilot-ou-dn>" -Enforced No` | Link is not enforced |
| 24 | Configure Windows LAPS backup directory | Management Host / DC | `GPMC > Computer Configuration > Policies > Administrative Templates > System > LAPS > Configure password backup directory` | Backup to AD DS is configured |
| 25 | Configure password settings | Management Host / DC | `GPMC > System > LAPS > Password Settings` | Password length, complexity, and age are configured |
| 26 | Configure administrator account name if needed | Management Host / DC | `GPMC > System > LAPS > Name of administrator account to manage` | Custom admin account is configured if used |
| 27 | Configure password encryption if used | Management Host / DC | `GPMC > System > LAPS > Enable password encryption` | Encryption is configured if approved |
| 28 | Configure post-authentication action if used | Management Host / DC | `GPMC > System > LAPS > Post-authentication actions` | Post-authentication behavior is configured |
| 29 | Export LAPS GPO report | Management Host / DC | `Get-GPOReport -Name "<laps-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\LAPS\<laps-gpo-name>.html` | Report shows LAPS settings |
| 30 | Back up configured LAPS GPO | Management Host / DC | `Backup-GPO -Name "<laps-gpo-name>" -Path C:\GPOPrep\Backup\LAPS` | Post-change backup exists |
| 31 | Refresh policy on test client | Test Client | `gpupdate /force` | Computer policy refresh completes |
| 32 | Trigger LAPS processing if needed | Test Client | `Invoke-LapsPolicyProcessing` | LAPS client processing runs |
| 33 | Validate LAPS event log | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-LAPS/Operational" -MaxEvents 100` | LAPS events appear |
| 34 | Read backed-up password as approved reader | Management Host / DC | `Get-LapsADPassword -Identity "<test-computer>" -AsPlainText` | Password returns for authorized reader |
| 35 | Force password rotation test | Test Client | `Reset-LapsPassword` | Password expiration is reset locally |
| 36 | Confirm new password backup | Management Host / DC | `Get-LapsADPassword -Identity "<test-computer>" -AsPlainText` | Updated password or expiration data appears |
| 37 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\LAPS\gpresult-laps.html` | HTML RSOP report exists |
| 38 | Document result | Operator | `Record schema status, permissions, GPO, target OU, reader group, resetter group, password retrieval test, and event logs` | LAPS deployment is documented |

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring Windows LAPS.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"

$LapsGpoName = "CORP-Workstation-Windows-LAPS"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports\LAPS"
$BackupPath = "$BasePath\Backup\LAPS"

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

Test-Path "\\$DomainFqdn\NETLOGON" |
  Out-File "$ReportPath\netlogon-check.txt"

# Confirm OU and computer.
Get-ADOrganizationalUnit `
  -Identity $PilotOU |
  Out-File "$ReportPath\pilot-ou.txt"

Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,Enabled,DistinguishedName,MemberOf |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-computer-before-laps.txt"

# Confirm modules.
Get-Module -ListAvailable GroupPolicy |
  Out-File "$ReportPath\group-policy-module.txt"

Get-Module -ListAvailable LAPS |
  Out-File "$ReportPath\laps-module.txt"

Get-Command -Module LAPS -ErrorAction SilentlyContinue |
  Select-Object Name,CommandType |
  Out-File "$ReportPath\laps-cmdlets.txt"

# Capture current inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-laps.txt"

# Capture GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-laps.csv" -NoTypeInformation

# Back up current GPOs.
Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre Windows LAPS configuration backup"

Write-Host "Windows LAPS precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_AD_Schema_And_Permissions_Skeleton
```powershell
# Run on a domain controller or management host with required permissions.
# Purpose: extend AD schema and delegate Windows LAPS permissions for a pilot OU.

Import-Module ActiveDirectory
Import-Module LAPS

$Domain = Get-ADDomain
$DomainDN = $Domain.DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$GroupsOU = "OU=Groups,OU=Corp,$DomainDN"

$ReaderGroup = "GG_LAPS_Password_Readers"
$ResetterGroup = "GG_LAPS_Password_Resetters"

$ReportPath = "C:\GPOPrep\Reports\LAPS"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm target OU exists.
Get-ADOrganizationalUnit -Identity $PilotOU

# Confirm LAPS cmdlets.
Get-Command -Module LAPS |
  Select-Object Name |
  Out-File "$ReportPath\laps-cmdlets-before-schema.txt"

# Extend AD schema.
# Requires Schema Admins and schema master availability.
Update-LapsADSchema

# Create groups if missing.
if (-not (Get-ADGroup -Identity $ReaderGroup -ErrorAction SilentlyContinue)) {
    New-ADGroup `
      -Name $ReaderGroup `
      -SamAccountName $ReaderGroup `
      -GroupScope Global `
      -GroupCategory Security `
      -Path $GroupsOU `
      -Description "Can read Windows LAPS passwords for delegated scope"
}

if (-not (Get-ADGroup -Identity $ResetterGroup -ErrorAction SilentlyContinue)) {
    New-ADGroup `
      -Name $ResetterGroup `
      -SamAccountName $ResetterGroup `
      -GroupScope Global `
      -GroupCategory Security `
      -Path $GroupsOU `
      -Description "Can force Windows LAPS password reset for delegated scope"
}

# Delegate computer self permission for Windows LAPS attributes.
Set-LapsADComputerSelfPermission `
  -Identity $PilotOU

# Delegate password read permission.
Set-LapsADReadPasswordPermission `
  -Identity $PilotOU `
  -AllowedPrincipals $ReaderGroup

# Delegate password reset permission.
Set-LapsADResetPasswordPermission `
  -Identity $PilotOU `
  -AllowedPrincipals $ResetterGroup

# Audit who has extended rights.
Find-LapsADExtendedRights `
  -Identity $PilotOU |
  Out-File "$ReportPath\laps-extended-rights-after-delegation.txt"

# Record groups.
Get-ADGroup `
  -Identity $ReaderGroup `
  -Properties Description,DistinguishedName |
  Select-Object Name,SamAccountName,Description,DistinguishedName |
  Out-File "$ReportPath\laps-reader-group.txt"

Get-ADGroup `
  -Identity $ResetterGroup `
  -Properties Description,DistinguishedName |
  Select-Object Name,SamAccountName,Description,DistinguishedName |
  Out-File "$ReportPath\laps-resetter-group.txt"

Write-Host "Windows LAPS schema and permissions workflow complete."
Write-Host "Pilot OU: $PilotOU"
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link the Windows LAPS GPO to the pilot OU.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$LapsGpoName = "CORP-Workstation-Windows-LAPS"

# Confirm pilot OU.
Get-ADOrganizationalUnit -Identity $PilotOU

# Create GPO if missing.
if (-not (Get-GPO -Name $LapsGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $LapsGpoName `
      -Comment "Windows LAPS policy for pilot workstation local administrator password rotation"
}

# Link GPO if missing.
$Inheritance = Get-GPInheritance -Target $PilotOU

if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $LapsGpoName)) {
    New-GPLink `
      -Name $LapsGpoName `
      -Target $PilotOU `
      -LinkEnabled Yes
}

# Safe default: enabled, not enforced.
Set-GPLink `
  -Name $LapsGpoName `
  -Target $PilotOU `
  -LinkEnabled Yes `
  -Enforced No

# Confirm link state.
Get-GPInheritance `
  -Target $PilotOU |
  Format-List
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Windows_LAPS_GPMC_Skeleton
```powershell
# Native GUI workflow for configuring Windows LAPS through GPMC.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-Workstation-Windows-LAPS
# 7. Select Edit.
# 8. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > System
#      > LAPS
#
# Backup directory:
# 9. Open:
#      Configure password backup directory
# 10. Set:
#      Enabled
# 11. Select:
#      Active Directory
#
# Password settings:
# 12. Open:
#      Password Settings
# 13. Set:
#      Enabled
# 14. Example:
#      Password complexity: Large letters + small letters + numbers + special characters
#      Password length: 16 or higher
#      Password age: 30 days
#
# Managed account name:
# 15. If using the built-in Administrator account:
#      Leave "Name of administrator account to manage" Not Configured.
# 16. If using a custom local admin account:
#      Open "Name of administrator account to manage"
#      Set to: LocalAdmin
#
# Password encryption:
# 17. If AD DS encryption is approved and supported:
#      Enable password encryption
#      Configure authorized decryptors according to delegation design.
# 18. If not piloting encryption:
#      Leave encryption settings Not Configured until delegation is tested.
#
# Post-authentication actions:
# 19. Optional:
#      Configure post-authentication actions
# 20. Example:
#      Reset password after delay
#      Delay: 8 hours
#
# Notes:
# - Configure only Windows LAPS settings under System > LAPS.
# - Do not mix legacy LAPS policy paths unless intentionally migrating.
# - Start with the pilot OU only.
# - Export a GPO report after configuration.
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Windows_LAPS_PowerShell_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: configure common Windows LAPS policy registry values through a GPO.
# Use GPMC when possible. This section is for repeatable lab configuration.

Import-Module GroupPolicy

$LapsGpoName = "CORP-Workstation-Windows-LAPS"

$ReportPath = "C:\GPOPrep\Reports\LAPS"
$BackupPath = "C:\GPOPrep\Backup\LAPS"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm GPO exists.
Get-GPO -Name $LapsGpoName

# Back up before policy changes.
Backup-GPO `
  -Name $LapsGpoName `
  -Path $BackupPath `
  -Comment "Before Windows LAPS policy registry configuration"

# Windows LAPS policy root.
$LapsPolicyKey = "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\LAPS"

# Configure password backup directory.
# Common lab value:
# 2 = backup password to Active Directory.
Set-GPRegistryValue `
  -Name $LapsGpoName `
  -Key $LapsPolicyKey `
  -ValueName "BackupDirectory" `
  -Type DWord `
  -Value 2

# Configure password age in days.
Set-GPRegistryValue `
  -Name $LapsGpoName `
  -Key $LapsPolicyKey `
  -ValueName "PasswordAgeDays" `
  -Type DWord `
  -Value 30

# Configure password length.
Set-GPRegistryValue `
  -Name $LapsGpoName `
  -Key $LapsPolicyKey `
  -ValueName "PasswordLength" `
  -Type DWord `
  -Value 16

# Configure password complexity.
# Use GPMC to select the exact complexity option for production.
# This lab value represents a strong mixed-character policy.
Set-GPRegistryValue `
  -Name $LapsGpoName `
  -Key $LapsPolicyKey `
  -ValueName "PasswordComplexity" `
  -Type DWord `
  -Value 4

# Optional:
# Manage a custom local admin account.
# Leave absent if using the built-in Administrator account.
# Set-GPRegistryValue `
#   -Name $LapsGpoName `
#   -Key $LapsPolicyKey `
#   -ValueName "AdministratorAccountName" `
#   -Type String `
#   -Value "LocalAdmin"

# Optional:
# Enable password encryption only after permission model is tested.
# Set-GPRegistryValue `
#   -Name $LapsGpoName `
#   -Key $LapsPolicyKey `
#   -ValueName "ADPasswordEncryptionEnabled" `
#   -Type DWord `
#   -Value 1

# Optional:
# Configure post-authentication behavior after pilot testing.
# Set-GPRegistryValue `
#   -Name $LapsGpoName `
#   -Key $LapsPolicyKey `
#   -ValueName "PostAuthenticationResetDelay" `
#   -Type DWord `
#   -Value 8

# Export reports.
Get-GPOReport `
  -Name $LapsGpoName `
  -ReportType Html `
  -Path "$ReportPath\$LapsGpoName-final.html"

Get-GPOReport `
  -Name $LapsGpoName `
  -ReportType Xml `
  -Path "$ReportPath\$LapsGpoName-final.xml"

# Back up after configuration.
Backup-GPO `
  -Name $LapsGpoName `
  -Path $BackupPath `
  -Comment "After Windows LAPS policy registry configuration"

Write-Host "Windows LAPS GPO policy configuration complete."
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Client_Refresh_And_Password_Backup_Skeleton
```powershell
# Run on the pilot client.
# Purpose: force policy refresh, trigger Windows LAPS processing, and collect client evidence.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\LAPS"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm identity.
hostname |
  Out-File "$ReportPath\client-hostname.txt"

whoami /all |
  Out-File "$ReportPath\client-whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$ReportPath\client-computer-info.txt"

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\client-nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\client-sysvol-check.txt"

# Force Group Policy refresh.
gpupdate /force |
  Tee-Object "$ReportPath\client-gpupdate-force.txt"

# Trigger Windows LAPS processing.
Invoke-LapsPolicyProcessing |
  Out-File "$ReportPath\invoke-laps-policy-processing.txt" `
  -ErrorAction SilentlyContinue

# Export GPResult.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\client-gpresult-computer-r.txt"

gpresult /h "$ReportPath\client-gpresult-laps.html"
gpresult /x "$ReportPath\client-gpresult-laps.xml"

# Validate Windows LAPS policy registry.
Get-ItemProperty `
  -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\LAPS" `
  -ErrorAction SilentlyContinue |
  Out-File "$ReportPath\client-laps-policy-registry.txt"

# Validate local admin account state.
Get-LocalUser |
  Select-Object Name,Enabled,LastLogon,PasswordLastSet,PasswordRequired |
  Out-File "$ReportPath\client-local-users.txt"

# Review Windows LAPS event log.
Get-WinEvent `
  -LogName "Microsoft-Windows-LAPS/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\client-laps-operational-events.txt"

# Review Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\client-group-policy-operational-events.txt"

Write-Host "Client LAPS processing collection complete."
Write-Host "Report path: $ReportPath"
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Read_Reset_And_Rotate_Password_Skeleton
```powershell
# Run on a management host or domain controller as an approved delegated operator.
# Purpose: read a Windows LAPS password, force rotation, and verify new backup state.

Import-Module ActiveDirectory
Import-Module LAPS

$ComputerName = "WIN11-01"
$ReportPath = "C:\GPOPrep\Reports\LAPS"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm computer object.
Get-ADComputer `
  -Identity $ComputerName `
  -Properties DistinguishedName |
  Select-Object Name,DistinguishedName |
  Out-File "$ReportPath\$ComputerName-ad-object.txt"

# Read LAPS password metadata.
Get-LapsADPassword `
  -Identity $ComputerName |
  Out-File "$ReportPath\$ComputerName-laps-password-metadata.txt"

# Read password as plaintext.
# Requires delegated read permission.
Get-LapsADPassword `
  -Identity $ComputerName `
  -AsPlainText |
  Out-File "$ReportPath\$ComputerName-laps-password-plaintext.txt"

# Force local password reset from the client side:
# Run this on the target computer as admin if testing local immediate reset.
# Reset-LapsPassword

# Force expiration from AD side:
# This can cause client to rotate at next processing interval depending on client behavior and policy.
# Set-LapsADPasswordExpirationTime `
#   -Identity $ComputerName `
#   -WhenEffective ([DateTime]::Now)

# Trigger client processing after expiration:
# On client:
# Invoke-LapsPolicyProcessing

# Read updated metadata after processing.
Get-LapsADPassword `
  -Identity $ComputerName |
  Out-File "$ReportPath\$ComputerName-laps-password-metadata-after-reset.txt"

Write-Host "LAPS read and reset validation workflow complete."
Write-Host "Securely delete plaintext report output if it was created for testing."
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final LAPS GPO report, OU inheritance, delegated permissions, and backup.

Import-Module ActiveDirectory
Import-Module GroupPolicy
Import-Module LAPS

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$LapsGpoName = "CORP-Workstation-Windows-LAPS"

$ReportPath = "C:\GPOPrep\Reports\LAPS"
$BackupPath = "C:\GPOPrep\Backup\LAPS"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture OU inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-final-laps.txt"

# Capture LAPS GPO permissions.
Get-GPPermission `
  -Name $LapsGpoName `
  -All |
  Out-File "$ReportPath\$LapsGpoName-gpo-permissions.txt"

# Capture LAPS delegated extended rights.
Find-LapsADExtendedRights `
  -Identity $PilotOU |
  Out-File "$ReportPath\laps-extended-rights-final.txt"

# Export final GPO reports.
Get-GPOReport `
  -Name $LapsGpoName `
  -ReportType Html `
  -Path "$ReportPath\$LapsGpoName-final.html"

Get-GPOReport `
  -Name $LapsGpoName `
  -ReportType Xml `
  -Path "$ReportPath\$LapsGpoName-final.xml"

# Search LAPS GPO report.
Select-String `
  -Path "$ReportPath\$LapsGpoName-final.xml" `
  -Pattern "LAPS","BackupDirectory","PasswordAgeDays","PasswordLength","PasswordComplexity","AdministratorAccountName","PostAuthentication","ADPasswordEncryption" |
  Out-File "$ReportPath\$LapsGpoName-report-search.txt"

# Back up final GPO.
Backup-GPO `
  -Name $LapsGpoName `
  -Path $BackupPath `
  -Comment "Final Windows LAPS GPO backup"

# Replication checks.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-laps.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-laps.txt"

Write-Host "Windows LAPS reports and backups complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-Command -Module LAPS` | Confirms Windows LAPS PowerShell module is available | LAPS cmdlets return |
| `Update-LapsADSchema` | Extends AD schema for Windows LAPS | Completes without error |
| `Set-LapsADComputerSelfPermission -Identity "<pilot-ou-dn>"` | Delegates computer self-write permission | Completes without error |
| `Set-LapsADReadPasswordPermission -Identity "<pilot-ou-dn>" -AllowedPrincipals "<reader-group>"` | Delegates password read permission | Completes without error |
| `Set-LapsADResetPasswordPermission -Identity "<pilot-ou-dn>" -AllowedPrincipals "<resetter-group>"` | Delegates password reset permission | Completes without error |
| `Find-LapsADExtendedRights -Identity "<pilot-ou-dn>"` | Audits LAPS extended rights | Only approved groups appear |
| `Get-GPO -Name "<laps-gpo-name>"` | Confirms LAPS GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-ou-dn>"` | Confirms LAPS GPO link | LAPS GPO appears in link list |
| `Get-GPOReport -Name "<laps-gpo-name>" -ReportType Html -Path "<path>"` | Exports readable LAPS policy report | HTML report exists |
| `Backup-GPO -Name "<laps-gpo-name>" -Path "<backup-path>"` | Backs up LAPS GPO | Backup completes |
| `gpupdate /force` | Forces client policy refresh | Policy refresh completes |
| `Invoke-LapsPolicyProcessing` | Triggers LAPS client processing | Cmdlet completes or logs actionable event |
| `gpresult /scope computer /r` | Confirms LAPS GPO applied to computer | LAPS GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\LAPS\gpresult-laps.html` | Exports client RSOP report | HTML report exists |
| `Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\LAPS"` | Validates local policy registry | LAPS policy values appear |
| `Get-WinEvent -LogName "Microsoft-Windows-LAPS/Operational" -MaxEvents 100` | Reviews LAPS client events | Password backup or processing events appear |
| `Get-LapsADPassword -Identity "<test-computer>"` | Reads LAPS password metadata | Password metadata returns |
| `Get-LapsADPassword -Identity "<test-computer>" -AsPlainText` | Reads password for approved reader | Password returns only for authorized account |
| `Reset-LapsPassword` | Forces local password reset on client | Password reset requested |
| `Set-LapsADPasswordExpirationTime -Identity "<test-computer>" -WhenEffective (Get-Date)` | Forces expiration from AD side | Expiration timestamp updates |
| `repadmin /replsummary` | Checks AD replication after schema and attribute changes | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks DC and SYSVOL health | Tests pass |

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely roll back Windows LAPS GPO deployment from the pilot scope.
# Schema extensions are normally not rolled back. Roll back policy links and permissions instead.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$LapsGpoName = "CORP-Workstation-Windows-LAPS"

$ReportPath = "C:\GPOPrep\Reports\LAPS-Rollback"
$BackupPath = "C:\GPOPrep\Backup\LAPS-Rollback"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture current state before rollback.
if (Get-GPO -Name $LapsGpoName -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $LapsGpoName `
      -ReportType Html `
      -Path "$ReportPath\$LapsGpoName-before-rollback.html"

    Backup-GPO `
      -Name $LapsGpoName `
      -Path $BackupPath `
      -Comment "Before Windows LAPS rollback"
}

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-laps-rollback.txt"

# Rollback option 1:
# Disable the LAPS GPO link while preserving the GPO.
Set-GPLink `
  -Name $LapsGpoName `
  -Target $PilotOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Remove known Windows LAPS policy registry values from the GPO.
$LapsPolicyKey = "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\LAPS"

$ValueNames = @(
  "BackupDirectory",
  "PasswordAgeDays",
  "PasswordLength",
  "PasswordComplexity",
  "AdministratorAccountName",
  "ADPasswordEncryptionEnabled",
  "ADPasswordEncryptionPrincipal",
  "ADEncryptedPasswordHistorySize",
  "PostAuthenticationActions",
  "PostAuthenticationResetDelay"
)

foreach ($ValueName in $ValueNames) {
    Remove-GPRegistryValue `
      -Name $LapsGpoName `
      -Key $LapsPolicyKey `
      -ValueName $ValueName `
      -ErrorAction SilentlyContinue
}

# Rollback option 3:
# Remove disposable lab GPO only if this was created only for testing.
# Remove-GPO -Name $LapsGpoName

# Capture state after rollback.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-laps-rollback.txt"

if (Get-GPO -Name $LapsGpoName -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $LapsGpoName `
      -ReportType Html `
      -Path "$ReportPath\$LapsGpoName-after-rollback.html"
}

Write-Host "Windows LAPS rollback workflow complete."
Write-Host "On pilot client: run gpupdate /force and validate LAPS policy no longer applies."
```

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| LAPS cmdlets not found | LAPS module unavailable on management host | `Get-Command -Module LAPS` | Use supported Windows management host or install required management components |
| `Update-LapsADSchema` fails | Insufficient permissions or schema master unavailable | Check Schema Admins membership and FSMO role | Use authorized account on reachable schema master |
| Computer cannot write password | Missing self permission on OU | `Set-LapsADComputerSelfPermission`; LAPS event log | Reapply self permission to correct computer OU |
| Password not backed up to AD | GPO not applied, backup directory not configured, or schema/permission issue | `gpresult /r`; LAPS event log; `Get-LapsADPassword` | Fix GPO scope, schema, or OU permissions |
| LAPS GPO does not apply | Computer outside pilot OU or filtering denies it | `gpresult /scope computer /r`; `Get-GPInheritance` | Move computer or fix link/filtering |
| Password read denied | Operator not in delegated reader group | `Find-LapsADExtendedRights`; group membership check | Add approved user to reader group and refresh token |
| Too many users can read passwords | Over-delegated extended rights | `Find-LapsADExtendedRights -Identity "<ou>"` | Remove excessive delegation and audit access |
| Password reset denied | Operator lacks reset permission | Delegation check | Add approved user to resetter group |
| Managed account not found | Custom local admin account name is wrong or account missing | `Get-LocalUser`; GPO report | Create account or correct `AdministratorAccountName` |
| Built-in Administrator not managed | Policy set to custom account or built-in disabled behavior misunderstood | GPO report and local users | Decide built-in vs custom account and configure correctly |
| Password complexity not accepted | Invalid complexity value or policy conflict | LAPS event log and GPO report | Configure supported complexity through GPMC |
| Encryption fails | Encryption enabled without proper directory or decryptor configuration | LAPS event log and delegation check | Configure decryptors or disable encryption during pilot |
| `Get-LapsADPassword` shows no password | Client has not processed policy yet | `Invoke-LapsPolicyProcessing`; event log | Refresh policy and trigger LAPS processing |
| Password does not rotate | Password age not expired or reset not triggered | `Get-LapsADPassword`; expiration timestamp | Use `Reset-LapsPassword` or expiration cmdlet |
| Client event log missing | Windows LAPS not processing or log path unavailable | `Get-WinEvent -ListLog "*LAPS*"` | Confirm OS support and policy application |
| Legacy LAPS conflict | Legacy policy and Windows LAPS policy both configured | GPO reports and registry paths | Migrate cleanly and avoid overlapping policy paths |
| GPO report missing LAPS settings | Wrong GPO edited or ADMX issue | `Get-GPOReport`; GPMC path | Edit correct GPO and confirm ADMX Central Store |
| Central Store missing LAPS settings | ADMX files outdated or missing | GPMC Administrative Templates | Update ADMX Central Store |
| Computer group filtering not reflected | Computer token stale | `gpresult /scope computer /r` | Reboot computer |
| User group read permission not reflected | User token stale | `whoami /groups` | Sign out and sign back in |
| AD replication delay | Password written to one DC but read from another before replication | `repadmin /replsummary`; specify DC if needed | Wait for replication or fix AD replication |
| SYSVOL replication issue | GPO policy files inconsistent across DCs | `dcdiag /test:sysvolcheck`; GPO report by DC | Fix SYSVOL/DFSR replication |
| Rollback does not remove policy | Another GPO configures Windows LAPS | `gpresult /h` | Find winning GPO and remove setting there |

# 20_Configure_LAPS_And_Local_Administrator_Password_Policy_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before LAPS rollout |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which computers apply the LAPS GPO and who can edit it |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Provides computer ADMX policy foundation |
| `11_Configure_Security_Baseline_GPO_Settings.md` | Security baseline companion workbook |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md` | Related local admin access control workbook |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Provides GPO backup and restore path before LAPS deployment |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Validates LAPS GPO application on clients |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Diagnoses GPO processing, AD replication, and SYSVOL issues |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md` | Ensures LAPS ADMX settings are visible in GPMC |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms domain infrastructure before deploying LAPS |
| `21_Configure_Advanced_Audit_Policy_And_Event_Log_Settings.md` | Next workbook for auditing password access, admin activity, and endpoint events |