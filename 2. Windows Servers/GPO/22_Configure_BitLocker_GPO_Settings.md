22_Configure_BitLocker_GPO_Settings.md
# 22_Configure_BitLocker_GPO_Settings

# 22_Configure_BitLocker_GPO_Settings_Index
22_Configure_BitLocker_GPO_Settings.md
22_Configure_BitLocker_GPO_Settings
22_Configure_BitLocker_GPO_Settings_Source_Basis
22_Configure_BitLocker_GPO_Settings_Mental_Model
22_Configure_BitLocker_GPO_Settings_Planning_Table
22_Configure_BitLocker_GPO_Settings_Control_Map
22_Configure_BitLocker_GPO_Settings_Configuration_Checklist
22_Configure_BitLocker_GPO_Settings_Precheck_Skeleton
22_Configure_BitLocker_GPO_Settings_Create_And_Link_GPO_Skeleton
22_Configure_BitLocker_GPO_Settings_OS_Drive_GPMC_Skeleton
22_Configure_BitLocker_GPO_Settings_Fixed_And_Removable_Drive_GPMC_Skeleton
22_Configure_BitLocker_GPO_Settings_Recovery_Backup_And_ADDS_Skeleton
22_Configure_BitLocker_GPO_Settings_Client_Enablement_And_Validation_Skeleton
22_Configure_BitLocker_GPO_Settings_Recovery_Test_Skeleton
22_Configure_BitLocker_GPO_Settings_Report_And_Backup_Skeleton
22_Configure_BitLocker_GPO_Settings_Verification_Commands
22_Configure_BitLocker_GPO_Settings_Rollback
22_Configure_BitLocker_GPO_Settings_Failure_Checks
22_Configure_BitLocker_GPO_Settings_Related_Labs

# 22_Configure_BitLocker_GPO_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | BitLocker Drive Encryption | OS, fixed, and removable drive encryption policy |
| Microsoft Learn | BitLocker Group Policy settings | Configuring BitLocker through Administrative Templates |
| Microsoft Learn | BitLocker recovery information in AD DS | Backing up recovery passwords and key packages to Active Directory |
| Microsoft Learn | BitLocker PowerShell cmdlets | `Get-BitLockerVolume`, `Enable-BitLocker`, `Backup-BitLockerKeyProtector`, and protector management |
| Microsoft Learn | `manage-bde` | Command-line validation, recovery backup, protector, and status operations |
| Microsoft Learn | TPM management | Checking TPM readiness for OS drive encryption |
| Microsoft Learn | Group Policy Management Console | Editing BitLocker GPO settings |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up BitLocker GPOs |
| Windows Server operational practice | Require AD DS recovery backup before encryption | Preventing unrecoverable encrypted endpoints |

# 22_Configure_BitLocker_GPO_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BitLocker | Windows full-volume encryption for OS, fixed data, and removable drives |
| TPM | Hardware security module used to protect OS drive unlock secrets |
| TPM protector | BitLocker protector that unlocks OS drive using TPM state |
| TPM + PIN | Stronger startup protector requiring user PIN plus TPM |
| Recovery password | 48-digit recovery key used when normal unlock fails |
| Recovery key package | Additional recovery material useful for advanced recovery scenarios |
| AD DS recovery backup | Storing BitLocker recovery data on the computer object in Active Directory |
| Key protector | Method used to unlock a BitLocker-protected volume |
| OS drive | Usually `C:` and contains Windows |
| Fixed data drive | Internal non-OS data volume |
| Removable data drive | USB or removable storage protected by BitLocker To Go |
| Used space only encryption | Encrypts used disk space, faster for new or clean devices |
| Full drive encryption | Encrypts entire volume, common for reused devices |
| Encryption method | Algorithm and strength such as XTS-AES 128 or XTS-AES 256 |
| Hardware encryption | Drive-based encryption; many baselines prefer software encryption for control and consistency |
| Recovery validation | Proving the recovery password exists in AD before treating BitLocker as complete |
| First rule | Do not enable BitLocker broadly until AD DS recovery backup has been proven on a pilot device |
| Blunt rule | BitLocker without recoverable keys can create unrecoverable endpoints |

# 22_Configure_BitLocker_GPO_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-ou-dn>` |
| Target workstation OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| BitLocker GPO | `CORP-Workstation-BitLocker-Policy` | `<bitlocker-gpo-name>` |
| Security filtering group | `GG_GPO_BitLocker_Apply` | `<filtering-group>` |
| Recovery reader group | `GG_BitLocker_Recovery_Readers` | `<recovery-reader-group>` |
| OS drive | `C:` | `<os-drive>` |
| Data drive | `D:` | `<data-drive>` |
| Removable drive policy | Deny write access unless encrypted | `<removable-policy>` |
| OS drive encryption method | `XTS-AES 256` or `XTS-AES 128` | `<os-encryption-method>` |
| Fixed drive encryption method | `XTS-AES 256` or `XTS-AES 128` | `<fixed-encryption-method>` |
| Removable drive encryption method | `AES-CBC 256` or approved baseline | `<removable-encryption-method>` |
| Encryption type | Used space only for new devices, full for reused devices | `<used-space-or-full>` |
| Startup protector | TPM only or TPM + PIN | `<startup-protector>` |
| Allow enhanced PIN | Yes if TPM + PIN design | `<yes-no>` |
| Recovery backup | AD DS | `<recovery-backup-location>` |
| Require recovery backup before encryption | Yes | `<yes-no>` |
| Save recovery password | Yes | `<yes-no>` |
| Save key package | Yes for OS/fixed drives if required | `<yes-no>` |
| Report path | `C:\GPOPrep\Reports\BitLocker` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup\BitLocker` | `<backup-path>` |
| Rollback method | Disable GPO link, suspend protectors, decrypt only if intentionally removing encryption | `<rollback-plan>` |

# 22_Configure_BitLocker_GPO_Settings_Control_Map
| Control | GPO Location | Purpose | Validation |
|---|---|---|---|
| Encryption method | `BitLocker Drive Encryption > Choose drive encryption method and cipher strength` | Sets OS, fixed, and removable encryption algorithms | GPO report and `manage-bde -status` |
| OS recovery | `Operating System Drives > Choose how BitLocker-protected operating system drives can be recovered` | Defines AD DS recovery backup and recovery options | Recovery object appears in AD |
| OS startup auth | `Operating System Drives > Require additional authentication at startup` | Defines TPM, TPM+PIN, startup key options | `manage-bde -protectors -get C:` |
| Enhanced PINs | `Operating System Drives > Allow enhanced PINs for startup` | Allows letters and symbols in startup PIN | GPO report |
| Require AD backup | OS recovery policy | Prevents BitLocker enablement until recovery data is backed up | Client event and AD object check |
| Fixed drive recovery | `Fixed Data Drives > Choose how BitLocker-protected fixed drives can be recovered` | Defines recovery behavior for internal data volumes | Recovery object appears in AD |
| Removable drive control | `Removable Data Drives` | Controls BitLocker To Go behavior | USB behavior and GPO report |
| Deny write to unencrypted removable drives | `Removable Data Drives > Deny write access to removable drives not protected by BitLocker` | Blocks unencrypted USB writes | USB write test |
| AD DS recovery object | Computer object child `msFVE-RecoveryInformation` | Stores recovery password metadata | `Get-ADObject` search |
| Client status | `Get-BitLockerVolume` | Shows encryption, protection, and protector state | Volume shows protection on |
| Recovery backup command | `Backup-BitLockerKeyProtector` or `manage-bde -protectors -adbackup` | Manually backs up protector to AD | AD recovery object appears |
| Recovery event logs | BitLocker API and Management log | Shows BitLocker policy and recovery activity | Event log contains success or error |

# 22_Configure_BitLocker_GPO_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 6 | Confirm pilot OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-ou-dn>"` | Pilot OU returns |
| 7 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 8 | Confirm test computer is in pilot OU | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under pilot OU |
| 9 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\BitLocker` | Report path exists |
| 10 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup\BitLocker` | Backup path exists |
| 11 | Back up current GPOs | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup\BitLocker` | Pre-change backup exists |
| 12 | Check TPM state | Test Client | `Get-Tpm` | TPM is present, ready, and enabled |
| 13 | Check Secure Boot state | Test Client | `Confirm-SecureBootUEFI` | Secure Boot state is known |
| 14 | Check BitLocker state | Test Client | `Get-BitLockerVolume` | Current encryption state is known |
| 15 | Check Windows recovery state | Test Client | `reagentc /info` | Recovery environment state is known |
| 16 | Create BitLocker GPO | Management Host | `New-GPO -Name "<bitlocker-gpo-name>" -Comment "BitLocker policy for pilot workstations"` | GPO exists |
| 17 | Link BitLocker GPO to pilot OU | Management Host | `New-GPLink -Name "<bitlocker-gpo-name>" -Target "<pilot-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 18 | Keep BitLocker GPO unenforced by default | Management Host | `Set-GPLink -Name "<bitlocker-gpo-name>" -Target "<pilot-ou-dn>" -Enforced No` | Link is not enforced |
| 19 | Configure encryption method | Management Host | `GPMC > BitLocker Drive Encryption > Choose drive encryption method and cipher strength` | Encryption methods are configured |
| 20 | Configure OS drive recovery | Management Host | `GPMC > BitLocker Drive Encryption > Operating System Drives > Choose how BitLocker-protected operating system drives can be recovered` | AD DS recovery backup is required |
| 21 | Configure startup authentication | Management Host | `GPMC > Operating System Drives > Require additional authentication at startup` | TPM or TPM + PIN behavior is configured |
| 22 | Configure enhanced PINs if used | Management Host | `GPMC > Operating System Drives > Allow enhanced PINs for startup` | Enhanced PIN stance is configured |
| 23 | Configure fixed drive recovery | Management Host | `GPMC > Fixed Data Drives > Choose how BitLocker-protected fixed drives can be recovered` | Fixed drive recovery backup is configured |
| 24 | Configure removable drive policy | Management Host | `GPMC > Removable Data Drives` | BitLocker To Go behavior is configured |
| 25 | Export BitLocker GPO report | Management Host | `Get-GPOReport -Name "<bitlocker-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\BitLocker\<bitlocker-gpo-name>.html` | Report shows BitLocker settings |
| 26 | Back up BitLocker GPO | Management Host | `Backup-GPO -Name "<bitlocker-gpo-name>" -Path C:\GPOPrep\Backup\BitLocker` | Post-change backup exists |
| 27 | Refresh policy on client | Test Client | `gpupdate /force` | Computer policy refresh completes |
| 28 | Reboot client if needed | Test Client | `Restart-Computer` | Computer restarts |
| 29 | Validate BitLocker GPO application | Test Client | `gpresult /scope computer /r` | BitLocker GPO appears under Applied GPOs |
| 30 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\BitLocker\gpresult-bitlocker.html` | HTML RSOP report exists |
| 31 | Enable BitLocker on OS drive if not already encrypted | Test Client | `Enable-BitLocker -MountPoint C: -EncryptionMethod XtsAes256 -UsedSpaceOnly -TpmProtector` | Encryption begins |
| 32 | Add recovery password protector if needed | Test Client | `Add-BitLockerKeyProtector -MountPoint C: -RecoveryPasswordProtector` | Recovery protector exists |
| 33 | Back up recovery protector to AD | Test Client | `Backup-BitLockerKeyProtector -MountPoint C: -KeyProtectorId "<protector-id>"` | Recovery protector is backed up |
| 34 | Validate BitLocker status | Test Client | `Get-BitLockerVolume -MountPoint C:` | Protection and encryption state are visible |
| 35 | Validate AD recovery object | Management Host | `Get-ADObject -SearchBase "<computer-dn>" -LDAPFilter "(objectClass=msFVE-RecoveryInformation)" -Properties msFVE-RecoveryPassword` | Recovery object exists |
| 36 | Validate recovery key retrieval | Management Host | `Get-ADObject -SearchBase "<computer-dn>" -LDAPFilter "(objectClass=msFVE-RecoveryInformation)" -Properties msFVE-RecoveryPassword,whenCreated` | Recovery password metadata returns |
| 37 | Review BitLocker client events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-BitLocker/BitLocker Management" -MaxEvents 100` | BitLocker events are visible |
| 38 | Document result | Operator | `Record GPO, target OU, encryption method, protector type, AD recovery object, recovery password retrieval test, reports, and backup` | BitLocker GPO rollout is documented |

# 22_Configure_BitLocker_GPO_Settings_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate domain, OU, test computer, and GPO readiness before configuring BitLocker policy.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"

$BitLockerGpoName = "CORP-Workstation-BitLocker-Policy"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports\BitLocker"
$BackupPath = "$BasePath\Backup\BitLocker"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm domain identity.
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

# Confirm pilot OU and test computer.
Get-ADOrganizationalUnit `
  -Identity $PilotOU |
  Out-File "$ReportPath\pilot-ou.txt"

Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,Enabled,DistinguishedName,MemberOf |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-computer-before-bitlocker.txt"

# Capture current inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-bitlocker.txt"

# Capture GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-bitlocker.csv" -NoTypeInformation

# Back up current GPOs.
Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre BitLocker GPO configuration backup"

Write-Host "BitLocker GPO precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 22_Configure_BitLocker_GPO_Settings_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link the BitLocker GPO to the pilot OU.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$BitLockerGpoName = "CORP-Workstation-BitLocker-Policy"

# Confirm pilot OU.
Get-ADOrganizationalUnit -Identity $PilotOU

# Create GPO if missing.
if (-not (Get-GPO -Name $BitLockerGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $BitLockerGpoName `
      -Comment "BitLocker policy for pilot workstation OS, fixed, and removable drives"
}

# Link GPO if missing.
$Inheritance = Get-GPInheritance -Target $PilotOU

if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $BitLockerGpoName)) {
    New-GPLink `
      -Name $BitLockerGpoName `
      -Target $PilotOU `
      -LinkEnabled Yes
}

# Safe default: enabled, not enforced.
Set-GPLink `
  -Name $BitLockerGpoName `
  -Target $PilotOU `
  -LinkEnabled Yes `
  -Enforced No

# Optional link order.
Set-GPLink `
  -Name $BitLockerGpoName `
  -Target $PilotOU `
  -Order 1

# Confirm.
Get-GPInheritance `
  -Target $PilotOU |
  Format-List
```

# 22_Configure_BitLocker_GPO_Settings_OS_Drive_GPMC_Skeleton
```powershell
# Native GUI workflow for BitLocker operating system drive policy.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-Workstation-BitLocker-Policy
# 7. Select Edit.
# 8. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > BitLocker Drive Encryption
#
# Encryption method:
# 9. Open:
#      Choose drive encryption method and cipher strength
# 10. Set:
#      Enabled
# 11. Example:
#      Operating system drives: XTS-AES 256-bit
#      Fixed data drives: XTS-AES 256-bit
#      Removable data drives: AES-CBC 256-bit or approved baseline
#
# Operating system drive recovery:
# 12. Go to:
#      Operating System Drives
# 13. Open:
#      Choose how BitLocker-protected operating system drives can be recovered
# 14. Set:
#      Enabled
# 15. Enable:
#      Save BitLocker recovery information to AD DS for operating system drives
# 16. Select:
#      Store recovery passwords and key packages
# 17. Enable:
#      Do not enable BitLocker until recovery information is stored to AD DS for operating system drives
#
# Startup authentication:
# 18. Open:
#      Require additional authentication at startup
# 19. Set:
#      Enabled
# 20. Example TPM-only baseline:
#      Allow TPM
#      Do not allow startup PIN without TPM
#      Do not allow startup key without TPM
#      Do not allow startup key and PIN with TPM
# 21. Example TPM + PIN baseline:
#      Allow TPM
#      Require startup PIN with TPM
#
# Enhanced PIN:
# 22. If using TPM + PIN:
#      Open Allow enhanced PINs for startup
#      Set Enabled if approved
#
# Hardware encryption:
# 23. Review:
#      Configure use of hardware-based encryption for operating system drives
# 24. Many baselines disable hardware-based encryption to prefer software encryption consistency.
#
# Notes:
# - Recovery backup to AD DS should be proven before broad enablement.
# - Start TPM-only unless the environment specifically requires startup PINs.
```

# 22_Configure_BitLocker_GPO_Settings_Fixed_And_Removable_Drive_GPMC_Skeleton
```powershell
# Native GUI workflow for fixed and removable BitLocker policy.
# Run on the management host.

gpmc.msc

# Fixed data drive recovery:
# 1. Edit:
#      CORP-Workstation-BitLocker-Policy
# 2. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > BitLocker Drive Encryption
#      > Fixed Data Drives
# 3. Open:
#      Choose how BitLocker-protected fixed drives can be recovered
# 4. Set:
#      Enabled
# 5. Enable:
#      Save BitLocker recovery information to AD DS for fixed data drives
# 6. Select:
#      Store recovery passwords and key packages
# 7. Enable:
#      Do not enable BitLocker until recovery information is stored to AD DS for fixed data drives
#
# Fixed data drive write protection:
# 8. Open:
#      Deny write access to fixed drives not protected by BitLocker
# 9. Use carefully.
# 10. Pilot only before broad rollout.
#
# Removable data drives:
# 11. Go to:
#      Removable Data Drives
# 12. Open:
#      Choose how BitLocker-protected removable drives can be recovered
# 13. Configure recovery backup behavior if removable drive recovery is in scope.
#
# Removable drive write protection:
# 14. Open:
#      Deny write access to removable drives not protected by BitLocker
# 15. Set based on organizational policy.
# 16. Optional:
#      Deny write access to devices configured in another organization
#
# Notes:
# - Removable-drive policy affects USB workflows.
# - Test with a disposable USB device before production rollout.
# - Fixed-drive write deny can break data volumes if not staged carefully.
```

# 22_Configure_BitLocker_GPO_Settings_Recovery_Backup_And_ADDS_Skeleton
```powershell
# Run on a management host with AD tools.
# Purpose: validate AD DS recovery object storage and retrieval.

Import-Module ActiveDirectory

$TestComputer = "WIN11-01"
$ReportPath = "C:\GPOPrep\Reports\BitLocker"

New-Item -ItemType Directory -Force -Path $ReportPath

# Get computer object DN.
$Computer = Get-ADComputer `
  -Identity $TestComputer `
  -Properties DistinguishedName

$ComputerDN = $Computer.DistinguishedName

$Computer |
  Select-Object Name,DistinguishedName |
  Out-File "$ReportPath\$TestComputer-ad-object.txt"

# Search for BitLocker recovery objects under the computer object.
Get-ADObject `
  -SearchBase $ComputerDN `
  -LDAPFilter "(objectClass=msFVE-RecoveryInformation)" `
  -Properties msFVE-RecoveryPassword,msFVE-RecoveryGuid,whenCreated |
  Select-Object Name,whenCreated,msFVE-RecoveryGuid,msFVE-RecoveryPassword |
  Out-File "$ReportPath\$TestComputer-bitlocker-recovery-objects.txt"

# Count recovery objects.
$RecoveryObjects = Get-ADObject `
  -SearchBase $ComputerDN `
  -LDAPFilter "(objectClass=msFVE-RecoveryInformation)" `
  -Properties msFVE-RecoveryPassword,whenCreated

[PSCustomObject]@{
    Computer = $TestComputer
    ComputerDN = $ComputerDN
    RecoveryObjectCount = ($RecoveryObjects | Measure-Object).Count
} | Export-Csv "$ReportPath\$TestComputer-bitlocker-recovery-summary.csv" -NoTypeInformation

Write-Host "AD DS BitLocker recovery object validation complete."
Write-Host "Computer DN: $ComputerDN"
```

# 22_Configure_BitLocker_GPO_Settings_Client_Enablement_And_Validation_Skeleton
```powershell
# Run on the pilot Windows client.
# Purpose: validate hardware readiness, apply policy, enable BitLocker if needed, and back up recovery key.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\BitLocker"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm identity and domain state.
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

# Hardware and boot readiness.
Get-Tpm |
  Out-File "$ReportPath\client-tpm-state.txt"

Confirm-SecureBootUEFI |
  Out-File "$ReportPath\client-secureboot-state.txt" `
  -ErrorAction SilentlyContinue

reagentc /info |
  Out-File "$ReportPath\client-reagentc-info.txt"

# Current BitLocker status.
Get-BitLockerVolume |
  Out-File "$ReportPath\client-bitlocker-status-before.txt"

manage-bde -status |
  Out-File "$ReportPath\client-manage-bde-status-before.txt"

# Refresh computer policy.
gpupdate /target:computer /force |
  Tee-Object "$ReportPath\client-gpupdate-computer.txt"

# Export RSOP.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\client-gpresult-computer-r.txt"

gpresult /h "$ReportPath\client-gpresult-bitlocker.html"
gpresult /x "$ReportPath\client-gpresult-bitlocker.xml"

# Check policy registry area.
Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\FVE" `
  -ErrorAction SilentlyContinue |
  Out-File "$ReportPath\client-fve-policy-registry.txt"

# Enable BitLocker on OS drive if not already encrypted.
$OsDrive = "C:"
$Volume = Get-BitLockerVolume -MountPoint $OsDrive

if ($Volume.ProtectionStatus -eq "Off") {
    Enable-BitLocker `
      -MountPoint $OsDrive `
      -EncryptionMethod XtsAes256 `
      -UsedSpaceOnly `
      -TpmProtector
}

# Ensure recovery password protector exists.
$Volume = Get-BitLockerVolume -MountPoint $OsDrive
$RecoveryProtector = $Volume.KeyProtector |
  Where-Object { $_.KeyProtectorType -eq "RecoveryPassword" } |
  Select-Object -First 1

if (-not $RecoveryProtector) {
    Add-BitLockerKeyProtector `
      -MountPoint $OsDrive `
      -RecoveryPasswordProtector

    $Volume = Get-BitLockerVolume -MountPoint $OsDrive
    $RecoveryProtector = $Volume.KeyProtector |
      Where-Object { $_.KeyProtectorType -eq "RecoveryPassword" } |
      Select-Object -First 1
}

# Back up recovery protector to AD DS.
if ($RecoveryProtector) {
    Backup-BitLockerKeyProtector `
      -MountPoint $OsDrive `
      -KeyProtectorId $RecoveryProtector.KeyProtectorId
}

# Capture final status.
Get-BitLockerVolume |
  Out-File "$ReportPath\client-bitlocker-status-after.txt"

manage-bde -status |
  Out-File "$ReportPath\client-manage-bde-status-after.txt"

manage-bde -protectors -get C: |
  Out-File "$ReportPath\client-manage-bde-protectors-c.txt"

# Event logs.
Get-WinEvent `
  -LogName "Microsoft-Windows-BitLocker/BitLocker Management" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\client-bitlocker-management-events.txt"

Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\client-group-policy-events.txt"

Write-Host "BitLocker client enablement and validation complete."
Write-Host "Report path: $ReportPath"
```

# 22_Configure_BitLocker_GPO_Settings_Recovery_Test_Skeleton
```powershell
# Run from management host and pilot client.
# Purpose: prove recovery key retrieval before relying on BitLocker at scale.

# Management host:
Import-Module ActiveDirectory

$TestComputer = "WIN11-01"
$ReportPath = "C:\GPOPrep\Reports\BitLocker"

New-Item -ItemType Directory -Force -Path $ReportPath

$Computer = Get-ADComputer `
  -Identity $TestComputer `
  -Properties DistinguishedName

$ComputerDN = $Computer.DistinguishedName

# Retrieve recovery object from AD DS.
$RecoveryObjects = Get-ADObject `
  -SearchBase $ComputerDN `
  -LDAPFilter "(objectClass=msFVE-RecoveryInformation)" `
  -Properties msFVE-RecoveryPassword,msFVE-RecoveryGuid,whenCreated |
  Sort-Object whenCreated -Descending

$RecoveryObjects |
  Select-Object Name,whenCreated,msFVE-RecoveryGuid,msFVE-RecoveryPassword |
  Out-File "$ReportPath\$TestComputer-recovery-passwords.txt"

# Pilot client:
# Use this to view protector IDs and confirm recovery password protector exists.
# manage-bde -protectors -get C:

# Safe recovery validation method:
# 1. Confirm AD DS recovery password exists.
# 2. Confirm recovery password matches the protector ID shown on the client.
# 3. Do not force recovery mode on production systems without console access.
# 4. For lab-only recovery test, document the recovery password and confirm console access first.

Write-Host "Recovery retrieval validation complete."
Write-Host "Computer DN: $ComputerDN"
Write-Host "Recovery object count: $($RecoveryObjects.Count)"
```

# 22_Configure_BitLocker_GPO_Settings_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final BitLocker GPO report, OU inheritance, permissions, recovery object state, and backup.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"
$BitLockerGpoName = "CORP-Workstation-BitLocker-Policy"

$ReportPath = "C:\GPOPrep\Reports\BitLocker"
$BackupPath = "C:\GPOPrep\Backup\BitLocker"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-final-bitlocker.txt"

# Capture GPO permissions.
Get-GPPermission `
  -Name $BitLockerGpoName `
  -All |
  Out-File "$ReportPath\$BitLockerGpoName-permissions.txt"

# Export final GPO reports.
Get-GPOReport `
  -Name $BitLockerGpoName `
  -ReportType Html `
  -Path "$ReportPath\$BitLockerGpoName-final.html"

Get-GPOReport `
  -Name $BitLockerGpoName `
  -ReportType Xml `
  -Path "$ReportPath\$BitLockerGpoName-final.xml"

# Search reports for BitLocker policy indicators.
Select-String `
  -Path "$ReportPath\$BitLockerGpoName-final.xml" `
  -Pattern "BitLocker","FVE","Recovery","AD DS","Encryption Method","Operating System Drives","Fixed Data Drives","Removable Data Drives","TPM","PIN" |
  Out-File "$ReportPath\$BitLockerGpoName-report-search.txt"

# Validate recovery object state for test computer.
$Computer = Get-ADComputer `
  -Identity $TestComputer `
  -Properties DistinguishedName

Get-ADObject `
  -SearchBase $Computer.DistinguishedName `
  -LDAPFilter "(objectClass=msFVE-RecoveryInformation)" `
  -Properties msFVE-RecoveryPassword,msFVE-RecoveryGuid,whenCreated |
  Select-Object Name,whenCreated,msFVE-RecoveryGuid,msFVE-RecoveryPassword |
  Out-File "$ReportPath\$TestComputer-final-recovery-objects.txt"

# Back up final BitLocker GPO.
Backup-GPO `
  -Name $BitLockerGpoName `
  -Path $BackupPath `
  -Comment "Final BitLocker GPO backup"

# Replication checks.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-bitlocker.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-bitlocker.txt"

Write-Host "BitLocker reports and backups complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 22_Configure_BitLocker_GPO_Settings_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-Tpm` | Validates TPM availability and readiness | TPM present and ready |
| `Confirm-SecureBootUEFI` | Checks Secure Boot state | Returns True or known state |
| `reagentc /info` | Checks Windows Recovery Environment state | WinRE state returns |
| `Get-BitLockerVolume` | Shows BitLocker volume status | OS volume state visible |
| `manage-bde -status` | Shows detailed BitLocker state | Volume encryption and protection state visible |
| `manage-bde -protectors -get C:` | Shows OS drive key protectors | TPM and Recovery Password protectors appear |
| `Get-GPO -Name "<bitlocker-gpo-name>"` | Confirms BitLocker GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-ou-dn>"` | Confirms BitLocker GPO link | GPO appears |
| `Get-GPPermission -Name "<bitlocker-gpo-name>" -All` | Confirms GPO apply permissions | Expected permissions appear |
| `Get-GPOReport -Name "<bitlocker-gpo-name>" -ReportType Html -Path "<path>"` | Exports readable GPO report | HTML report exists |
| `Backup-GPO -Name "<bitlocker-gpo-name>" -Path "<backup-path>"` | Backs up BitLocker GPO | Backup completes |
| `gpupdate /target:computer /force` | Refreshes computer policy | Refresh completes |
| `gpresult /scope computer /r` | Confirms BitLocker GPO applies | GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\BitLocker\gpresult-bitlocker.html` | Exports RSOP report | HTML report exists |
| `Get-ItemProperty "HKLM:\Software\Policies\Microsoft\FVE"` | Validates BitLocker policy registry | FVE policy values appear |
| `Enable-BitLocker -MountPoint C: -EncryptionMethod XtsAes256 -UsedSpaceOnly -TpmProtector` | Enables BitLocker with TPM protector | Encryption begins |
| `Add-BitLockerKeyProtector -MountPoint C: -RecoveryPasswordProtector` | Adds recovery password protector | Protector is created |
| `Backup-BitLockerKeyProtector -MountPoint C: -KeyProtectorId "<id>"` | Backs up protector to AD DS | Backup completes |
| `manage-bde -protectors -adbackup C: -id {<id>}` | Alternate AD backup method | Protector is backed up |
| `Get-ADObject -SearchBase "<computer-dn>" -LDAPFilter "(objectClass=msFVE-RecoveryInformation)" -Properties msFVE-RecoveryPassword` | Validates AD recovery object | Recovery object returns |
| `Get-WinEvent -LogName "Microsoft-Windows-BitLocker/BitLocker Management" -MaxEvents 100` | Reviews BitLocker events | Events return |
| `repadmin /replsummary` | Checks AD replication | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks SYSVOL and DC health | Tests pass |

# 22_Configure_BitLocker_GPO_Settings_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely roll back BitLocker GPO policy from the pilot scope.
# Disabling policy does not automatically decrypt existing drives.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$BitLockerGpoName = "CORP-Workstation-BitLocker-Policy"

$ReportPath = "C:\GPOPrep\Reports\BitLocker-Rollback"
$BackupPath = "C:\GPOPrep\Backup\BitLocker-Rollback"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture current state before rollback.
if (Get-GPO -Name $BitLockerGpoName -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $BitLockerGpoName `
      -ReportType Html `
      -Path "$ReportPath\$BitLockerGpoName-before-rollback.html"

    Backup-GPO `
      -Name $BitLockerGpoName `
      -Path $BackupPath `
      -Comment "Before BitLocker GPO rollback"
}

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-bitlocker-rollback.txt"

# Rollback option 1:
# Disable the BitLocker GPO link while preserving the GPO.
Set-GPLink `
  -Name $BitLockerGpoName `
  -Target $PilotOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Use GPMC to set BitLocker policy settings back to Not Configured.
gpmc.msc

# GUI rollback workflow:
# 1. Edit BitLocker GPO.
# 2. Set configured BitLocker Administrative Template settings back to Not Configured.
# 3. Export a new GPO report.
# 4. Run gpupdate /force on the pilot client.
#
# Rollback option 3:
# Suspend BitLocker temporarily on a pilot client.
# Run on pilot client:
# Suspend-BitLocker -MountPoint C: -RebootCount 1
#
# Rollback option 4:
# Fully decrypt only if intentionally removing BitLocker from a lab device.
# Run on pilot client:
# Disable-BitLocker -MountPoint C:
#
# Rollback option 5:
# Remove disposable lab GPO only after link is disabled and clients are validated.
# Remove-GPO -Name $BitLockerGpoName

# Capture state after rollback.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-bitlocker-rollback.txt"

if (Get-GPO -Name $BitLockerGpoName -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $BitLockerGpoName `
      -ReportType Html `
      -Path "$ReportPath\$BitLockerGpoName-after-rollback.html"
}

Write-Host "BitLocker GPO rollback workflow complete."
Write-Host "On pilot client: run gpupdate /force and validate BitLocker status."
```

# 22_Configure_BitLocker_GPO_Settings_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| BitLocker GPO does not apply | Computer outside pilot OU or filtering blocks it | `gpresult /scope computer /r`; `Get-GPInheritance` | Move computer or fix filtering |
| FVE registry policy missing | GPO not applied or wrong GPO edited | `gpresult /h`; `Get-GPOReport` | Edit correct GPO and refresh policy |
| TPM protector cannot be added | TPM not present, not ready, disabled, or firmware issue | `Get-Tpm` | Enable and initialize TPM |
| Secure Boot check fails | Legacy boot mode or unsupported firmware state | `Confirm-SecureBootUEFI`; `msinfo32` | Convert to UEFI/Secure Boot if required by baseline |
| BitLocker enablement blocked | Recovery backup required but AD DS backup failed | BitLocker event log and AD recovery object check | Fix AD backup requirement and retry |
| Recovery key not in AD | Recovery protector not backed up, GPO not applied, or permissions issue | `Get-ADObject` recovery search | Run `Backup-BitLockerKeyProtector` and fix policy |
| No recovery password protector | Only TPM protector exists | `manage-bde -protectors -get C:` | Add recovery password protector |
| Multiple recovery keys exist | Rotation, re-encryption, or repeated protector creation | AD recovery object list | Keep newest documented key and avoid duplicate workflows |
| Encryption method not as expected | Existing encryption method remains or policy applied after encryption | `manage-bde -status` | Decrypt and re-encrypt only if required and approved |
| BitLocker asks for recovery after change | TPM PCR state changed, BIOS update, boot change, Secure Boot change | Recovery screen and event logs | Enter recovery key, suspend before firmware changes next time |
| Startup PIN not accepted | Enhanced PIN policy mismatch or keyboard layout issue | GPO report | Fix PIN policy and document startup input requirements |
| Fixed drive cannot be written | Deny write to unencrypted fixed drives enabled too early | GPO report and client behavior | Encrypt fixed drive or disable write-deny policy |
| USB cannot be written | Removable drive write-deny policy enabled | GPO report and USB test | Encrypt USB or adjust removable policy |
| AD recovery read denied | Operator lacks permission to read recovery object | AD object read test | Delegate recovery read to approved group |
| Computer token stale | Computer newly added to filtering group | `gpresult /scope computer /r` | Reboot computer |
| GPMC lacks BitLocker settings | ADMX Central Store outdated or missing | GPMC Administrative Templates | Update ADMX Central Store |
| Client lacks BitLocker cmdlets | Edition, feature, or module availability issue | `Get-Command *BitLocker*` | Use supported Windows edition and feature set |
| Event log missing | Log path unavailable or no events generated | `Get-WinEvent -ListLog "*BitLocker*"` | Trigger BitLocker action and recheck |
| Recovery object written to another DC not visible yet | AD replication delay | `repadmin /replsummary` | Wait or fix AD replication |
| SYSVOL policy inconsistent | DFSR/SYSVOL replication issue | `dcdiag /test:sysvolcheck` | Fix SYSVOL replication |
| Rollback does not decrypt drive | GPO rollback does not decrypt existing BitLocker volumes | `Get-BitLockerVolume` | Use `Disable-BitLocker` only if decryption is intended |
| Suspended BitLocker stays suspended | Reboot count or manual suspension remains | `Get-BitLockerVolume` | `Resume-BitLocker -MountPoint C:` |
| Device locked out after policy test | Recovery key not documented before recovery test | AD recovery object check | Retrieve recovery key from AD before forcing recovery scenarios |

# 22_Configure_BitLocker_GPO_Settings_Related_Labs
| Related Lab                                                               | Relationship                                                                |
| ------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `00_Group_Policy_Index.md`                                                | Defines suite order and dependency path                                     |
| `01_Install_Group_Policy_Management_Tools.md`                             | Provides GPMC and GroupPolicy module                                        |
| `02_Create_And_Link_Baseline_GPO.md`                                      | Establishes GPO creation and linking workflow                               |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md`     | Controls precedence before BitLocker rollout                                |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md`                   | Controls which computers can apply the BitLocker GPO                        |
| `06_Configure_Computer_Administrative_Template_Settings.md`               | Provides computer-side Administrative Template foundation                   |
| `11_Configure_Security_Baseline_GPO_Settings.md`                          | Security baseline companion workbook                                        |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md`     | Ensures remote management remains available during encryption rollout       |
| `14_Backup_Restore_Import_And_Report_GPOs.md`                             | Provides backup and restore workflow before BitLocker policy changes        |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Validates BitLocker GPO application                                         |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md`              | Diagnoses scope, filtering, AD replication, and SYSVOL failures             |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md`               | Ensures BitLocker Administrative Templates are available                    |
| `20_Configure_LAPS_And_Local_Administrator_Password_Policy.md`            | Related endpoint security policy requiring AD-backed recovery data          |
| `21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings.md`          | Related policy that validates PKI and trust before other security workflows |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md`                     | Confirms domain infrastructure before BitLocker GPO rollout                 |
| `23_Configure_Advanced_Audit_Policy_And_Event_Log_Settings.md`            | Next workbook for collecting endpoint security and recovery-related events  |
