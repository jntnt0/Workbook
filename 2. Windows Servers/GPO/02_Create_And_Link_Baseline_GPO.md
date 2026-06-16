02_Create_And_Link_Baseline_GPO.md
# 02_Create_And_Link_Baseline_GPO

# 02_Create_And_Link_Baseline_GPO_Index
02_Create_And_Link_Baseline_GPO.md
02_Create_And_Link_Baseline_GPO
02_Create_And_Link_Baseline_GPO_Source_Basis
02_Create_And_Link_Baseline_GPO_Mental_Model
02_Create_And_Link_Baseline_GPO_Planning_Table
02_Create_And_Link_Baseline_GPO_Configuration_Checklist
02_Create_And_Link_Baseline_GPO_Precheck_Skeleton
02_Create_And_Link_Baseline_GPO_Create_GPO_Skeleton
02_Create_And_Link_Baseline_GPO_Link_GPO_To_OU_Skeleton
02_Create_And_Link_Baseline_GPO_Validation_Marker_Setting_Skeleton
02_Create_And_Link_Baseline_GPO_Client_Policy_Refresh_Skeleton
02_Create_And_Link_Baseline_GPO_Report_And_Backup_Skeleton
02_Create_And_Link_Baseline_GPO_Verification_Commands
02_Create_And_Link_Baseline_GPO_Rollback
02_Create_And_Link_Baseline_GPO_Failure_Checks
02_Create_And_Link_Baseline_GPO_Related_Labs

# 02_Create_And_Link_Baseline_GPO_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | New-GPO | Creating a new Group Policy Object |
| Microsoft Learn | New-GPLink | Linking a GPO to a site, domain, or OU |
| Microsoft Learn | Set-GPLink | Enabling, disabling, enforcing, and ordering GPO links |
| Microsoft Learn | Get-GPO | Finding and validating existing GPOs |
| Microsoft Learn | Get-GPInheritance | Viewing linked and inherited GPOs on a target container |
| Microsoft Learn | Set-GPRegistryValue | Adding a harmless registry policy marker for validation |
| Microsoft Learn | Get-GPOReport | Exporting GPO configuration reports |
| Microsoft Learn | Backup-GPO | Backing up GPO state before and after changes |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, GroupPolicy event logs | Proving the linked baseline GPO applies to a test computer |
| Windows Server operational practice | Create, link, test, report, backup, and document | Safe first GPO deployment workflow |

# 02_Create_And_Link_Baseline_GPO_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Baseline GPO | First controlled GPO used to prove creation, link, inheritance, client processing, reporting, and rollback |
| Empty GPO | GPO with no settings; useful structurally but weak for client validation |
| Validation marker | Harmless registry policy setting used to prove the GPO actually reached the client |
| GPO link | Connection between the GPO and a target OU, domain, or site |
| Target OU | Container where the GPO link is attached |
| Scope | Who or what can receive the GPO after link, inheritance, filtering, and WMI evaluation |
| Computer scope | Computer object must be in or under the linked OU for computer policy to apply |
| User scope | User object must be in or under the linked OU for user policy to apply |
| Link enabled | GPO link is active and can be processed |
| Enforced | Link cannot be blocked by child OU block inheritance |
| Link order | Precedence order when multiple GPOs are linked to the same OU |
| GPO status | Controls whether user settings, computer settings, all settings, or no settings are enabled |
| SYSVOL portion | File-based GPT content of the GPO |
| AD portion | GPC object stored in Active Directory |
| GPResult | Client-side proof showing applied and denied GPOs |
| First rule | Create and link to a test OU before touching production OUs |
| Blunt rule | A GPO existing in GPMC means nothing until the correct client actually applies it |

# 02_Create_And_Link_Baseline_GPO_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Management host | `MGMT01` | `<management-host>` |
| Primary DC | `DC1.corp.local` | `<primary-dc-fqdn>` |
| Target OU name | `Workstations` | `<target-ou-name>` |
| Target OU DN | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Test user | `tuser` | `<test-user>` |
| Baseline GPO name | `CORP-Workstation-Baseline` | `<baseline-gpo-name>` |
| Baseline GPO purpose | Workstation baseline policy anchor | `<baseline-purpose>` |
| Link enabled | Yes | `<yes-no>` |
| Enforced | No | `<yes-no>` |
| Initial link order | 1 or current last | `<link-order>` |
| GPO status | AllSettingsEnabled | `<gpo-status>` |
| Validation marker key | `HKLM\Software\Policies\CORP\GPOBaseline` | `<marker-key>` |
| Validation marker value | `BaselineApplied` | `<marker-value-name>` |
| Validation marker data | `1` | `<marker-value-data>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Unlink then delete test GPO if needed | `<rollback-plan>` |

# 02_Create_And_Link_Baseline_GPO_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm management tool availability | Management Host | `Get-Command -Module GroupPolicy` | Group Policy cmdlets are listed |
| 6 | Confirm target OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-ou-dn>"` | Target OU returns |
| 7 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer object exists |
| 8 | Confirm test computer is under target OU or child OU | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | DistinguishedName is inside target OU path |
| 9 | Confirm SYSVOL is reachable | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 11 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 12 | Export current inheritance state | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Existing GPO link state is known |
| 13 | Back up current domain GPO state | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 14 | Check whether baseline GPO already exists | Management Host | `Get-GPO -Name "<baseline-gpo-name>" -ErrorAction SilentlyContinue` | Existing or missing state is known |
| 15 | Create baseline GPO | Management Host | `New-GPO -Name "<baseline-gpo-name>" -Comment "Baseline GPO anchor for workstation policy"` | GPO object is created |
| 16 | Confirm baseline GPO exists | Management Host | `Get-GPO -Name "<baseline-gpo-name>"` | GPO returns |
| 17 | Set GPO status | Management Host | `(Get-GPO -Name "<baseline-gpo-name>").GpoStatus` | Status is visible |
| 18 | Link GPO to target OU | Management Host | `New-GPLink -Name "<baseline-gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | GPO link is created |
| 19 | Confirm link on target OU | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Baseline GPO appears under links |
| 20 | Keep link unenforced unless intentionally required | Management Host | `Set-GPLink -Name "<baseline-gpo-name>" -Target "<target-ou-dn>" -Enforced No` | Link is not enforced |
| 21 | Add harmless computer registry validation marker | Management Host | `Set-GPRegistryValue -Name "<baseline-gpo-name>" -Key "HKLM\Software\Policies\CORP\GPOBaseline" -ValueName "BaselineApplied" -Type DWord -Value 1` | GPO has a verifiable computer setting |
| 22 | Export GPO report | Management Host | `Get-GPOReport -Name "<baseline-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<baseline-gpo-name>.html` | Report is created |
| 23 | Back up new baseline GPO | Management Host | `Backup-GPO -Name "<baseline-gpo-name>" -Path C:\GPOPrep\Backup` | Baseline GPO backup exists |
| 24 | Force policy refresh on test client | Test Client | `gpupdate /force` | Policy refresh completes |
| 25 | Confirm applied computer GPOs | Test Client | `gpresult /scope computer /r` | Baseline GPO appears as applied |
| 26 | Create HTML GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-baseline.html` | Client policy report is created |
| 27 | Confirm registry marker on client | Test Client | `Get-ItemProperty -Path "HKLM:\Software\Policies\CORP\GPOBaseline"` | `BaselineApplied` equals `1` |
| 28 | Review client Group Policy events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 50` | Recent processing events are visible |
| 29 | Document result | Operator | `Record GPO name, target OU, test client, report path, backup path, and validation result` | Baseline GPO deployment is documented |

# 02_Create_And_Link_Baseline_GPO_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before creating the first baseline GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"
$GpoName = "CORP-Workstation-Baseline"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportPath,$BackupPath

# Confirm identity.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
hostname
whoami

# Confirm Group Policy tooling.
Get-Module -ListAvailable GroupPolicy
Get-Command -Module GroupPolicy | Select-Object Name | Sort-Object Name

# Confirm domain location and DC locator.
Resolve-DnsName $DomainFqdn
nltest /dsgetdc:$DomainFqdn

# Confirm SYSVOL.
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm target OU.
Get-ADOrganizationalUnit `
  -Identity $TargetOU `
  -Properties DistinguishedName

# Confirm test computer exists and is in scope.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName

# Capture current inheritance.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\inheritance-before.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before.csv" -NoTypeInformation

# Backup current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath