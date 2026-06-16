01_Install_Group_Policy_Management_Tools.md
# 01_Install_Group_Policy_Management_Tools

# 01_Install_Group_Policy_Management_Tools_Index
01_Install_Group_Policy_Management_Tools.md
01_Install_Group_Policy_Management_Tools
01_Install_Group_Policy_Management_Tools_Source_Basis
01_Install_Group_Policy_Management_Tools_Mental_Model
01_Install_Group_Policy_Management_Tools_Planning_Table
01_Install_Group_Policy_Management_Tools_Configuration_Checklist
01_Install_Group_Policy_Management_Tools_Server_GPMC_Install_Skeleton
01_Install_Group_Policy_Management_Tools_Windows_Client_RSAT_Install_Skeleton
01_Install_Group_Policy_Management_Tools_Module_And_Console_Validation_Skeleton
01_Install_Group_Policy_Management_Tools_Remote_Domain_Admin_Validation_Skeleton
01_Install_Group_Policy_Management_Tools_Verification_Commands
01_Install_Group_Policy_Management_Tools_Rollback
01_Install_Group_Policy_Management_Tools_Failure_Checks
01_Install_Group_Policy_Management_Tools_Related_Labs

# 01_Install_Group_Policy_Management_Tools_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install-WindowsFeature | Installing Group Policy Management Console on Windows Server |
| Microsoft Learn | Add-WindowsCapability | Installing RSAT Group Policy tools on Windows client operating systems |
| Microsoft Learn | Get-WindowsFeature | Verifying Windows Server feature installation state |
| Microsoft Learn | Get-WindowsCapability | Verifying Windows client RSAT capability installation state |
| Microsoft Learn | GroupPolicy PowerShell module | Validating `Get-GPO`, `New-GPO`, `New-GPLink`, `Backup-GPO`, and report cmdlets |
| Microsoft Learn | Group Policy Management Console | Managing GPOs, links, inheritance, delegation, backups, and reporting |
| Windows Server operational practice | GPMC, RSAT, domain DNS, DC locator, SYSVOL, and admin permissions | Preparing a usable Group Policy management workstation or server |

# 01_Install_Group_Policy_Management_Tools_Mental_Model
| Concept | Operational Meaning |
|---|---|
| GPMC | Group Policy Management Console, the GUI tool for managing domain GPOs |
| GroupPolicy module | PowerShell module used to create, link, report, back up, restore, and inspect GPOs |
| RSAT | Remote Server Administration Tools used to manage Windows Server roles from a workstation |
| Server install path | Windows Server uses `Install-WindowsFeature GPMC` |
| Client install path | Windows 10 or Windows 11 uses `Add-WindowsCapability` for RSAT Group Policy tools |
| Domain dependency | GPMC is useful only when the admin host can reach AD DS and domain controllers |
| DNS dependency | GPMC depends on domain DNS and DC locator working correctly |
| SYSVOL dependency | GPO policy files live under SYSVOL and must be reachable |
| Admin rights | Installing tools requires local administrator rights; managing GPOs requires domain permissions |
| Delegated GPO admin | Non-Domain Admin account granted rights to create, edit, link, or report GPOs |
| Management host | Workstation or server where GPMC and GroupPolicy module are installed |
| First rule | Install tools on a stable management host, not randomly on every server |
| Blunt rule | If DNS, domain join, DC locator, or SYSVOL is broken, GPMC installation is not the real problem |

# 01_Install_Group_Policy_Management_Tools_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Management host | `MGMT01` | `<management-host>` |
| Management host OS | `Windows Server 2022` or `Windows 11` | `<server/client-version>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Primary domain controller | `DC1.corp.local` | `<primary-dc-fqdn>` |
| Domain controller IP | `10.10.10.10` | `<dc-ip>` |
| Admin account | `CORP\Administrator` | `<admin-account>` |
| Local admin required | Yes | `<yes-no>` |
| Install method | Server feature or Windows capability | `<server-feature/client-capability>` |
| RSAT capability name | `Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0` | `<rsat-capability-name>` |
| GPMC console command | `gpmc.msc` | `<gpmc-command>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Test GPO for validation | None or `TEST-GPO-Tool-Validation` | `<test-gpo-name>` |
| Rollback plan | Remove GPMC or RSAT only if lab cleanup requires it | `<rollback-plan>` |

# 01_Install_Group_Policy_Management_Tools_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Local Administrators membership is visible |
| 2 | Confirm host identity | Management Host | `hostname` | Management host name is known |
| 3 | Confirm domain membership | Management Host | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Host is joined to expected domain |
| 4 | Confirm network configuration | Management Host | `Get-NetIPConfiguration` | IP, gateway, and DNS settings are visible |
| 5 | Confirm domain DNS resolution | Management Host | `Resolve-DnsName <domain-fqdn>` | Domain resolves through AD DNS |
| 6 | Confirm DC locator | Management Host | `nltest /dsgetdc:<domain-fqdn>` | Domain controller is returned |
| 7 | Confirm SYSVOL path | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | SYSVOL Policies path is reachable |
| 8 | Confirm NETLOGON path | Management Host | `Test-Path "\\<domain-fqdn>\NETLOGON"` | NETLOGON share is reachable |
| 9 | Check server GPMC feature state if using Windows Server | Management Host | `Get-WindowsFeature GPMC` | GPMC install state is known |
| 10 | Install GPMC on Windows Server | Management Host | `Install-WindowsFeature GPMC -IncludeManagementTools` | GPMC installs successfully |
| 11 | Check RSAT Group Policy capability if using Windows 10 or 11 | Management Host | `Get-WindowsCapability -Online -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0` | RSAT capability state is known |
| 12 | Install RSAT Group Policy tools on Windows 10 or 11 | Management Host | `Add-WindowsCapability -Online -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0` | RSAT Group Policy tools install successfully |
| 13 | Confirm GroupPolicy module exists | Management Host | `Get-Module -ListAvailable GroupPolicy` | GroupPolicy module appears |
| 14 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | Module imports without error |
| 15 | Confirm GroupPolicy commands | Management Host | `Get-Command -Module GroupPolicy` | GPO cmdlets are listed |
| 16 | Confirm `Get-GPO` works | Management Host | `Get-GPO -All` | Domain GPO inventory returns |
| 17 | Launch GPMC console | Management Host | `gpmc.msc` | Group Policy Management Console opens |
| 18 | Confirm current GPO inventory | Management Host | `Get-GPO -All \| Select-Object DisplayName,Id,Owner,GpoStatus` | Existing GPOs are visible |
| 19 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 20 | Export Default Domain Policy report | Management Host | `Get-GPOReport -Name "Default Domain Policy" -ReportType Html -Path C:\GPOPrep\Reports\Default-Domain-Policy.html` | HTML report is created |
| 21 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 22 | Back up a known GPO | Management Host | `Backup-GPO -Name "Default Domain Policy" -Path C:\GPOPrep\Backup` | Backup completes |
| 23 | Confirm OU inheritance query works | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | GPO links and inheritance state are visible |
| 24 | Validate client-side tools if run from a test client | Test Client | `gpupdate /force`; `gpresult /r` | Policy refresh and result reporting work |
| 25 | Document tool installation | Operator | `Record host, OS, install method, module state, GPMC state, and validation output` | Management tool baseline is documented |

# 01_Install_Group_Policy_Management_Tools_Server_GPMC_Install_Skeleton
```powershell
# Run in elevated PowerShell on Windows Server.
# Purpose: install Group Policy Management Console and validate the GroupPolicy module.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm server identity and domain state.
hostname
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
Get-NetIPConfiguration

# Confirm domain and DC locator.
Resolve-DnsName $DomainFqdn
nltest /dsgetdc:$DomainFqdn

# Confirm SYSVOL and NETLOGON access.
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Check current GPMC feature state.
Get-WindowsFeature GPMC

# Install GPMC.
Install-WindowsFeature `
  -Name GPMC `
  -IncludeManagementTools

# Verify feature state.
Get-WindowsFeature GPMC

# Validate GroupPolicy module.
Get-Module -ListAvailable GroupPolicy
Import-Module GroupPolicy
Get-Command -Module GroupPolicy

# Validate GPO inventory access.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime

# Export a known report.
Get-GPOReport `
  -Name "Default Domain Policy" `
  -ReportType Html `
  -Path "$ReportPath\Default-Domain-Policy.html"

# Back up a known GPO.
Backup-GPO `
  -Name "Default Domain Policy" `
  -Path $BackupPath