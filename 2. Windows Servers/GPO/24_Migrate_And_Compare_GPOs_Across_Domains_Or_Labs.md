24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs.md
# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Index
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs.md
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Source_Basis
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Mental_Model
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Planning_Table
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Migration_Map
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Configuration_Checklist
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Precheck_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Source_Domain_Export_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Migration_Table_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Target_Domain_Import_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Copy_GPO_Same_Domain_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Compare_GPOs_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Client_Validation_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Report_And_Backup_Skeleton
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Verification_Commands
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Rollback
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Failure_Checks
24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Related_Labs

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy backup and restore | Exporting source GPOs and preserving rollback points |
| Microsoft Learn | `Backup-GPO` | Creating source-domain backup packages |
| Microsoft Learn | `Import-GPO` | Importing backed-up settings into a target GPO |
| Microsoft Learn | `Copy-GPO` | Copying GPOs inside a domain or between trusted domains |
| Microsoft Learn | Migration Table Editor | Mapping users, groups, computers, UNC paths, and security principals during migration |
| Microsoft Learn | Group Policy Management Console | GUI-based backup, import, copy, and migration table workflow |
| Microsoft Learn | `Get-GPOReport` | Exporting source and target GPO settings for comparison |
| Microsoft Learn | `Get-GPInheritance` | Validating target OU link scope and precedence |
| Microsoft Learn | `Get-GPPermission` | Validating security filtering and delegation after import |
| Windows Server operational practice | Import into a test GPO before production linking | Preventing accidental broad policy impact during lab or domain migration |

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Mental_Model
| Concept | Operational Meaning |
|---|---|
| GPO migration | Moving or recreating GPO settings from one domain or lab to another |
| Source domain | Domain or lab where the original GPO exists |
| Target domain | Domain or lab where the GPO will be imported or recreated |
| Backup-GPO | Exports GPO settings into a file-based backup set |
| Import-GPO | Imports backed-up settings into a target GPO |
| Copy-GPO | Copies an existing GPO to another GPO, optionally across domains |
| Migration table | Mapping file used to translate domain-specific references |
| Domain-specific reference | User, group, computer, SID, UNC path, or security principal that may not exist in the target domain |
| Unmapped reference | Source-domain object or path that has not been translated |
| Test import | Import into a non-linked or pilot-linked target GPO before production |
| Target GPO | GPO that receives imported settings |
| Link migration | Recreating OU, domain, or site links in the target environment |
| Permission migration | Rebuilding security filtering and delegation in the target environment |
| WMI filter migration | Recreating WMI filters in the target domain and attaching them to target GPOs |
| ADMX dependency | Target GPMC needs matching policy definitions to display Administrative Template settings cleanly |
| Report comparison | Comparing source and target GPO XML or HTML reports after migration |
| First rule | Back up and report source GPOs before changing target GPOs |
| Blunt rule | GPO import migrates settings, not your entire OU design, group model, WMI filters, or business intent |

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Source domain FQDN | `corp.local` | `<source-domain-fqdn>` |
| Source domain DN | `DC=corp,DC=local` | `<source-domain-dn>` |
| Target domain FQDN | `lab.local` | `<target-domain-fqdn>` |
| Target domain DN | `DC=lab,DC=local` | `<target-domain-dn>` |
| Source GPO | `CORP-Workstation-Security-Baseline` | `<source-gpo-name>` |
| Target GPO | `LAB-Workstation-Security-Baseline` | `<target-gpo-name>` |
| Source OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<source-ou-dn>` |
| Target OU | `OU=Pilot,OU=Workstations,OU=Lab,DC=lab,DC=local` | `<target-ou-dn>` |
| Source backup path | `C:\GPOPrep\Migration\SourceBackup` | `<source-backup-path>` |
| Target import path | `C:\GPOPrep\Migration\SourceBackup` | `<target-import-path>` |
| Source report path | `C:\GPOPrep\Migration\SourceReports` | `<source-report-path>` |
| Target report path | `C:\GPOPrep\Migration\TargetReports` | `<target-report-path>` |
| Migration table path | `C:\GPOPrep\Migration\MigrationTable.migtable` | `<migration-table-path>` |
| Source security group | `CORP\GG_GPO_Workstation_Baseline_Apply` | `<source-security-group>` |
| Target security group | `LAB\GG_GPO_Workstation_Baseline_Apply` | `<target-security-group>` |
| Source UNC path | `\\FS1\Shared` | `<source-unc-path>` |
| Target UNC path | `\\LAB-FS1\Shared` | `<target-unc-path>` |
| Source printer | `\\PRINT1\HP-LaserJet-01` | `<source-printer-path>` |
| Target printer | `\\LAB-PRINT1\HP-LaserJet-01` | `<target-printer-path>` |
| WMI filters | Recreate manually | `<wmi-filter-plan>` |
| Link strategy | Import first, link to pilot only | `<link-strategy>` |
| Validation client | `LAB-WIN11-01` | `<target-test-client>` |
| Rollback method | Disable target link, restore target GPO backup, delete test GPO | `<rollback-plan>` |

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Migration_Map
| Migration Item | Migrates Automatically? | Needs Mapping? | Validation |
|---|---|---|---|
| Administrative Template registry settings | Usually yes | No, unless paths or values reference source domain | Target GPO report |
| Security Settings | Usually yes | Often yes for users/groups | Target report and RSOP |
| Group Policy Preferences drive maps | Settings import | UNC paths may need mapping | GPP report and client drive test |
| Group Policy Preferences printers | Settings import | Printer UNC paths need mapping | GPP report and printer test |
| Group Policy Preferences local groups | Settings import | Domain groups need mapping | Local group membership check |
| Scripts | Script references import | SYSVOL content and paths need review | SYSVOL script path and client logs |
| Folder Redirection | Settings import | UNC paths need mapping | Client folder path check |
| Software Installation | Package references import | MSI UNC paths need mapping | Client install test |
| Security Filtering | Not safely portable as-is | Yes | `Get-GPPermission` |
| Delegation | Not safely portable as-is | Yes | GPMC Delegation tab |
| GPO Links | No | Recreate in target | `Get-GPInheritance` |
| Link order | No | Recreate in target | `Get-GPInheritance` |
| Enforcement | No or must be recreated | Review manually | `Get-GPInheritance` |
| Block inheritance | OU property, not GPO setting | Recreate only if intended | `Get-GPInheritance` |
| WMI Filters | Usually manual | Recreate and attach | GPMC Scope tab |
| ADMX display | Depends on target Central Store | Target ADMX may need update | GPMC opens cleanly |
| Certificates or PKI references | Settings import | CA/template names may need review | PKI client validation |
| LAPS settings | Settings import | OU permissions and schema do not migrate through GPO import | LAPS cmdlets and client validation |
| BitLocker recovery policy | Settings import | AD recovery access and target OU validation needed | Recovery object check |
| WEF collector URL | Settings import | Collector URL usually needs mapping | SubscriptionManager registry |
| Firewall rules | Settings import | IP/subnet/source scope may need mapping | `Get-NetFirewallRule` |

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Source Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import GroupPolicy module | Source Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 3 | Import AD module | Source Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 4 | Confirm source domain | Source Management Host | `Get-ADDomain` | Source domain object returns |
| 5 | Confirm source GPO exists | Source Management Host | `Get-GPO -Name "<source-gpo-name>"` | Source GPO object returns |
| 6 | Create migration folders | Source Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Migration\SourceBackup,C:\GPOPrep\Migration\SourceReports` | Folders exist |
| 7 | Export source GPO HTML report | Source Management Host | `Get-GPOReport -Name "<source-gpo-name>" -ReportType Html -Path "<source-report-path>\<source-gpo-name>.html"` | HTML report exists |
| 8 | Export source GPO XML report | Source Management Host | `Get-GPOReport -Name "<source-gpo-name>" -ReportType Xml -Path "<source-report-path>\<source-gpo-name>.xml"` | XML report exists |
| 9 | Back up source GPO | Source Management Host | `Backup-GPO -Name "<source-gpo-name>" -Path "<source-backup-path>" -Comment "Source GPO migration backup"` | Source backup exists |
| 10 | Capture source GPO permissions | Source Management Host | `Get-GPPermission -Name "<source-gpo-name>" -All` | Source ACLs recorded |
| 11 | Capture source OU inheritance | Source Management Host | `Get-GPInheritance -Target "<source-ou-dn>"` | Source link context recorded |
| 12 | Search source report for domain references | Source Management Host | `Select-String -Path "<source-report-path>\<source-gpo-name>.xml" -Pattern "<source-domain>","\\\\"` | Source-specific references identified |
| 13 | Copy backup and reports to target lab | Operator | `Copy backup folder to target management host` | Target can access source backup |
| 14 | Open elevated PowerShell | Target Management Host | `whoami /groups` | Admin token is visible |
| 15 | Confirm target domain | Target Management Host | `Get-ADDomain` | Target domain object returns |
| 16 | Confirm target OU exists | Target Management Host | `Get-ADOrganizationalUnit -Identity "<target-ou-dn>"` | Target OU returns |
| 17 | Create target report and backup folders | Target Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Migration\TargetReports,C:\GPOPrep\Migration\TargetBackup` | Folders exist |
| 18 | Back up target GPOs before import | Target Management Host | `Backup-GPO -All -Path C:\GPOPrep\Migration\TargetBackup` | Target pre-change backup exists |
| 19 | Create migration table if needed | Target Management Host | `GPMC > Domains > right-click > Open Migration Table Editor` | Migration table file created |
| 20 | Map source groups to target groups | Target Management Host | `Migration Table Editor` | Security principals are mapped |
| 21 | Map source UNC paths to target UNC paths | Target Management Host | `Migration Table Editor` | UNC paths are mapped |
| 22 | Create target GPO | Target Management Host | `New-GPO -Name "<target-gpo-name>" -Comment "Imported from <source-gpo-name>"` | Target GPO exists |
| 23 | Import source backup into target GPO | Target Management Host | `Import-GPO -BackupGpoName "<source-gpo-name>" -TargetName "<target-gpo-name>" -Path "<target-import-path>"` | Settings import into target GPO |
| 24 | Import with migration table if required | Target Management Host | `Import-GPO -BackupGpoName "<source-gpo-name>" -TargetName "<target-gpo-name>" -Path "<target-import-path>" -MigrationTable "<migration-table-path>"` | Settings import with mapped references |
| 25 | Export target GPO HTML report | Target Management Host | `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path "<target-report-path>\<target-gpo-name>.html"` | Target report exists |
| 26 | Export target GPO XML report | Target Management Host | `Get-GPOReport -Name "<target-gpo-name>" -ReportType Xml -Path "<target-report-path>\<target-gpo-name>.xml"` | Target XML report exists |
| 27 | Compare source and target XML reports | Target Management Host | `Compare-Object (Get-Content source.xml) (Get-Content target.xml)` | Differences are visible |
| 28 | Search target report for unmapped source references | Target Management Host | `Select-String -Path "<target-report-path>\<target-gpo-name>.xml" -Pattern "<source-domain>","<source-server>","<source-unc>"` | No unwanted source references remain |
| 29 | Rebuild security filtering | Target Management Host | `Set-GPPermission -Name "<target-gpo-name>" -TargetName "<target-security-group>" -TargetType Group -PermissionLevel GpoApply` | Target group can apply GPO |
| 30 | Preserve required read access | Target Management Host | `Set-GPPermission -Name "<target-gpo-name>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | GPO remains readable |
| 31 | Recreate WMI filters if required | Target Management Host | `GPMC > WMI Filters > New` | WMI filters exist in target domain |
| 32 | Attach WMI filter if required | Target Management Host | `GPMC > Target GPO > Scope > WMI Filtering` | WMI filter is attached |
| 33 | Link target GPO to pilot OU | Target Management Host | `New-GPLink -Name "<target-gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | GPO linked to target pilot OU |
| 34 | Keep link unenforced by default | Target Management Host | `Set-GPLink -Name "<target-gpo-name>" -Target "<target-ou-dn>" -Enforced No` | Link is not enforced |
| 35 | Confirm target inheritance | Target Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Target GPO appears in link list |
| 36 | Refresh policy on target test client | Target Client | `gpupdate /force` | Policy refresh completes |
| 37 | Validate target GPO application | Target Client | `gpresult /scope computer /r` | Target GPO appears if computer-side policy applies |
| 38 | Export target client GPResult | Target Client | `gpresult /h C:\GPOPrep\Migration\TargetReports\gpresult-target-gpo.html` | Client RSOP report exists |
| 39 | Validate specific migrated settings | Target Client | `Check registry, firewall, local groups, drive maps, printers, scripts, or app behavior` | Migrated setting works |
| 40 | Back up final target GPO | Target Management Host | `Backup-GPO -Name "<target-gpo-name>" -Path C:\GPOPrep\Migration\TargetBackup` | Final target backup exists |
| 41 | Document migration result | Operator | `Record source GPO, target GPO, migration table, mapped references, reports, differences, links, and validation outcome` | Migration is documented |

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Precheck_Skeleton
```powershell
# Run in both source and target environments as appropriate.
# Purpose: validate modules, domain identity, SYSVOL, and GPO readiness.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$BasePath = "C:\GPOPrep\Migration"
$ReportPath = "$BasePath\PrecheckReports"
$BackupPath = "$BasePath\PrecheckBackup"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportPath,$BackupPath

# Domain identity.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName,PDCEmulator |
  Out-File "$ReportPath\domain-info.txt"

# DC and SYSVOL checks.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-policies-check.txt"

Test-Path "\\$DomainFqdn\NETLOGON" |
  Out-File "$ReportPath\netlogon-check.txt"

# GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime,UserVersion,ComputerVersion |
  Export-Csv "$ReportPath\gpo-inventory.csv" -NoTypeInformation

# AD replication and SYSVOL health.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising.txt"

# Backup current GPOs before migration work.
Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre GPO migration precheck backup"

Write-Host "GPO migration precheck complete."
Write-Host "Domain: $DomainFqdn"
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Source_Domain_Export_Skeleton
```powershell
# Run in the source domain.
# Purpose: export source GPO backup, reports, permissions, inheritance, and source-specific references.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$SourceDomain = Get-ADDomain
$SourceDomainFqdn = $SourceDomain.DNSRoot
$SourceDomainDN = $SourceDomain.DistinguishedName

$SourceGpoName = "CORP-Workstation-Security-Baseline"
$SourceOU = "OU=Pilot,OU=Workstations,OU=Corp,$SourceDomainDN"

$BasePath = "C:\GPOPrep\Migration"
$SourceBackupPath = "$BasePath\SourceBackup"
$SourceReportPath = "$BasePath\SourceReports"

New-Item -ItemType Directory -Force -Path $SourceBackupPath,$SourceReportPath

# Confirm source GPO and OU.
Get-GPO -Name $SourceGpoName |
  Format-List * |
  Out-File "$SourceReportPath\$SourceGpoName-source-gpo-metadata.txt"

Get-GPInheritance `
  -Target $SourceOU |
  Out-File "$SourceReportPath\$SourceGpoName-source-ou-inheritance.txt"

Get-GPPermission `
  -Name $SourceGpoName `
  -All |
  Out-File "$SourceReportPath\$SourceGpoName-source-permissions.txt"

# Export source reports.
Get-GPOReport `
  -Name $SourceGpoName `
  -ReportType Html `
  -Path "$SourceReportPath\$SourceGpoName-source.html"

Get-GPOReport `
  -Name $SourceGpoName `
  -ReportType Xml `
  -Path "$SourceReportPath\$SourceGpoName-source.xml"

# Back up source GPO.
Backup-GPO `
  -Name $SourceGpoName `
  -Path $SourceBackupPath `
  -Comment "Source GPO backup for cross-domain or lab migration"

# Search for source-domain references and UNC paths.
Select-String `
  -Path "$SourceReportPath\$SourceGpoName-source.xml" `
  -Pattern $SourceDomain.NetBIOSName,$SourceDomainFqdn,"\\\\","S-1-5-21","Printer","Drive","Script","Folder Redirection","Security" |
  Out-File "$SourceReportPath\$SourceGpoName-source-reference-search.txt"

# Inventory backup files.
Get-ChildItem `
  -Path $SourceBackupPath `
  -Recurse |
  Select-Object FullName,Length,LastWriteTime |
  Out-File "$SourceReportPath\$SourceGpoName-source-backup-file-list.txt"

Write-Host "Source GPO export complete."
Write-Host "Source backup path: $SourceBackupPath"
Write-Host "Source report path: $SourceReportPath"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Migration_Table_Skeleton
```powershell
# Native GUI workflow for migration table creation.
# Run on target management host with the source GPO backup available.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Right-click the domain or Group Policy Objects container.
# 3. Open Migration Table Editor.
# 4. Create a new migration table.
# 5. Use Tools or Populate from Backup.
# 6. Select the source GPO backup folder:
#      C:\GPOPrep\Migration\SourceBackup
# 7. Review every source reference found.
#
# Map source-domain references:
# 8. Source:
#      CORP\GG_GPO_Workstation_Baseline_Apply
#    Destination:
#      LAB\GG_GPO_Workstation_Baseline_Apply
#
# Map source UNC paths:
# 9. Source:
#      \\FS1\Shared
#    Destination:
#      \\LAB-FS1\Shared
#
# Map printer paths:
# 10. Source:
#      \\PRINT1\HP-LaserJet-01
#     Destination:
#      \\LAB-PRINT1\HP-LaserJet-01
#
# Map script paths if needed:
# 11. Source:
#      \\corp.local\SYSVOL\corp.local\scripts\Logon.ps1
#     Destination:
#      \\lab.local\SYSVOL\lab.local\scripts\Logon.ps1
#
# Save migration table:
# 12. Save as:
#      C:\GPOPrep\Migration\MigrationTable.migtable
#
# Notes:
# - Migration tables are most important when GPOs contain security principals or UNC paths.
# - They do not recreate OU links.
# - They do not create missing target groups or shares.
# - Create target groups, shares, and printers before import validation.
```

```powershell
# Optional target-side reference search to build a mapping checklist.

$SourceXml = "C:\GPOPrep\Migration\SourceReports\CORP-Workstation-Security-Baseline-source.xml"
$MappingChecklist = "C:\GPOPrep\Migration\SourceReports\migration-reference-checklist.txt"

Select-String `
  -Path $SourceXml `
  -Pattern "CORP\\","corp.local","\\\\","S-1-5-21","Drive","Printer","Folder Redirection","Scripts","Groups","Users" |
  Out-File $MappingChecklist

Write-Host "Migration mapping checklist written to: $MappingChecklist"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Target_Domain_Import_Skeleton
```powershell
# Run in the target domain.
# Purpose: import source GPO backup into a target GPO, optionally using a migration table.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$TargetDomain = Get-ADDomain
$TargetDomainFqdn = $TargetDomain.DNSRoot
$TargetDomainDN = $TargetDomain.DistinguishedName

$SourceBackupGpoName = "CORP-Workstation-Security-Baseline"
$TargetGpoName = "LAB-Workstation-Security-Baseline"

$TargetOU = "OU=Pilot,OU=Workstations,OU=Lab,$TargetDomainDN"

$SourceBackupPath = "C:\GPOPrep\Migration\SourceBackup"
$MigrationTablePath = "C:\GPOPrep\Migration\MigrationTable.migtable"

$TargetReportPath = "C:\GPOPrep\Migration\TargetReports"
$TargetBackupPath = "C:\GPOPrep\Migration\TargetBackup"

New-Item -ItemType Directory -Force -Path $TargetReportPath,$TargetBackupPath

# Confirm target OU.
Get-ADOrganizationalUnit -Identity $TargetOU

# Back up target environment before import.
Backup-GPO `
  -All `
  -Path $TargetBackupPath `
  -Comment "Pre target GPO import backup"

# Create target GPO if missing.
if (-not (Get-GPO -Name $TargetGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $TargetGpoName `
      -Comment "Imported from $SourceBackupGpoName"
}

# Export target before import.
Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Html `
  -Path "$TargetReportPath\$TargetGpoName-before-import.html"

Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Xml `
  -Path "$TargetReportPath\$TargetGpoName-before-import.xml"

# Import with migration table if present, otherwise import without one.
if (Test-Path $MigrationTablePath) {
    Import-GPO `
      -BackupGpoName $SourceBackupGpoName `
      -TargetName $TargetGpoName `
      -Path $SourceBackupPath `
      -MigrationTable $MigrationTablePath
}
else {
    Import-GPO `
      -BackupGpoName $SourceBackupGpoName `
      -TargetName $TargetGpoName `
      -Path $SourceBackupPath
}

# Export target after import.
Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Html `
  -Path "$TargetReportPath\$TargetGpoName-after-import.html"

Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Xml `
  -Path "$TargetReportPath\$TargetGpoName-after-import.xml"

# Rebuild target security filtering.
# Example:
# Set-GPPermission `
#   -Name $TargetGpoName `
#   -TargetName "GG_GPO_Workstation_Baseline_Apply" `
#   -TargetType Group `
#   -PermissionLevel GpoApply

# Preserve broad read if apply permission was removed from Authenticated Users.
Set-GPPermission `
  -Name $TargetGpoName `
  -TargetName "Authenticated Users" `
  -TargetType Group `
  -PermissionLevel GpoRead `
  -ErrorAction SilentlyContinue

# Link to pilot OU only.
$Inheritance = Get-GPInheritance -Target $TargetOU
if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $TargetGpoName)) {
    New-GPLink `
      -Name $TargetGpoName `
      -Target $TargetOU `
      -LinkEnabled Yes
}

Set-GPLink `
  -Name $TargetGpoName `
  -Target $TargetOU `
  -LinkEnabled Yes `
  -Enforced No

# Confirm.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$TargetReportPath\$TargetGpoName-target-ou-inheritance-after-import.txt"

Get-GPPermission `
  -Name $TargetGpoName `
  -All |
  Out-File "$TargetReportPath\$TargetGpoName-target-permissions-after-import.txt"

Write-Host "Target GPO import complete."
Write-Host "Target GPO: $TargetGpoName"
Write-Host "Target report path: $TargetReportPath"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Copy_GPO_Same_Domain_Skeleton
```powershell
# Run in a single domain when creating a lab copy or staging copy of a GPO.
# Purpose: copy a GPO inside the same domain before modifying or testing it.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$SourceGpoName = "CORP-Workstation-Security-Baseline"
$CopyGpoName = "CORP-Workstation-Security-Baseline-LabCopy"
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$ReportPath = "C:\GPOPrep\Migration\CopyReports"
$BackupPath = "C:\GPOPrep\Migration\CopyBackup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm source GPO.
Get-GPO -Name $SourceGpoName

# Export source report.
Get-GPOReport `
  -Name $SourceGpoName `
  -ReportType Html `
  -Path "$ReportPath\$SourceGpoName-before-copy.html"

Get-GPOReport `
  -Name $SourceGpoName `
  -ReportType Xml `
  -Path "$ReportPath\$SourceGpoName-before-copy.xml"

# Copy GPO.
Copy-GPO `
  -SourceName $SourceGpoName `
  -TargetName $CopyGpoName

# Export copied GPO report.
Get-GPOReport `
  -Name $CopyGpoName `
  -ReportType Html `
  -Path "$ReportPath\$CopyGpoName-after-copy.html"

Get-GPOReport `
  -Name $CopyGpoName `
  -ReportType Xml `
  -Path "$ReportPath\$CopyGpoName-after-copy.xml"

# Back up copied GPO.
Backup-GPO `
  -Name $CopyGpoName `
  -Path $BackupPath `
  -Comment "Copied GPO for lab validation"

# Optional link to pilot OU.
$Inheritance = Get-GPInheritance -Target $PilotOU
if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $CopyGpoName)) {
    New-GPLink `
      -Name $CopyGpoName `
      -Target $PilotOU `
      -LinkEnabled Yes
}

Set-GPLink `
  -Name $CopyGpoName `
  -Target $PilotOU `
  -Enforced No

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\$CopyGpoName-pilot-ou-inheritance.txt"

Write-Host "Same-domain GPO copy complete."
Write-Host "Copy GPO: $CopyGpoName"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Compare_GPOs_Skeleton
```powershell
# Run on target management host after source and target XML reports are available.
# Purpose: compare source and target GPO reports and identify unmapped references.

$SourceXml = "C:\GPOPrep\Migration\SourceReports\CORP-Workstation-Security-Baseline-source.xml"
$TargetXml = "C:\GPOPrep\Migration\TargetReports\LAB-Workstation-Security-Baseline-after-import.xml"

$ComparePath = "C:\GPOPrep\Migration\CompareReports"
New-Item -ItemType Directory -Force -Path $ComparePath

# Raw XML compare.
if ((Test-Path $SourceXml) -and (Test-Path $TargetXml)) {
    Compare-Object `
      -ReferenceObject (Get-Content $SourceXml) `
      -DifferenceObject (Get-Content $TargetXml) |
      Out-File "$ComparePath\source-vs-target-raw-xml-compare.txt"
}

# Search for leftover source-domain references in target.
$SourceIndicators = @(
  "CORP\\",
  "corp.local",
  "\\\\FS1",
  "\\\\PRINT1",
  "S-1-5-21"
)

foreach ($Pattern in $SourceIndicators) {
    Select-String `
      -Path $TargetXml `
      -Pattern $Pattern `
      -ErrorAction SilentlyContinue |
      Out-File "$ComparePath\target-leftover-source-reference-$($Pattern.Replace('\','_').Replace(':','_')).txt"
}

# Search for common setting categories.
$CategoryPatterns = @(
  "Registry",
  "Security",
  "Drive",
  "Printer",
  "Folder Redirection",
  "Scripts",
  "Firewall",
  "WindowsUpdate",
  "Defender",
  "LAPS",
  "BitLocker",
  "AppLocker",
  "Event Forwarding",
  "Certificate"
)

foreach ($Pattern in $CategoryPatterns) {
    Select-String `
      -Path $SourceXml,$TargetXml `
      -Pattern $Pattern `
      -ErrorAction SilentlyContinue |
      Out-File "$ComparePath\category-search-$($Pattern.Replace(' ','_')).txt"
}

# Produce simple file hashes for source and target reports.
Get-FileHash `
  -Path $SourceXml,$TargetXml `
  -ErrorAction SilentlyContinue |
  Export-Csv "$ComparePath\source-target-report-hashes.csv" -NoTypeInformation

Write-Host "GPO comparison complete."
Write-Host "Compare path: $ComparePath"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Client_Validation_Skeleton
```powershell
# Run on the target test client after target GPO import and pilot link.
# Purpose: prove the migrated GPO applies and settings behave correctly.

$DomainFqdn = "lab.local"
$ReportPath = "C:\GPOPrep\Migration\TargetClientReports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm identity and domain state.
hostname |
  Out-File "$ReportPath\hostname.txt"

whoami /all |
  Out-File "$ReportPath\whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$ReportPath\computer-info.txt"

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

# Refresh policy.
gpupdate /force |
  Tee-Object "$ReportPath\gpupdate-force.txt"

# Reboot or sign out if the migrated GPO requires it.
# Restart-Computer

# Export GPResult.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-r.txt"

gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-r.txt"

gpresult /h "$ReportPath\gpresult-migrated-gpo.html"
gpresult /x "$ReportPath\gpresult-migrated-gpo.xml"
gpresult /z > "$ReportPath\gpresult-migrated-gpo-verbose.txt"

# Review Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-events.txt"

# Example targeted validation checks.
# Registry policies:
Get-ChildItem `
  -Path "HKLM:\Software\Policies" `
  -ErrorAction SilentlyContinue |
  Select-Object Name |
  Out-File "$ReportPath\hklm-policy-roots.txt"

Get-ChildItem `
  -Path "HKCU:\Software\Policies" `
  -ErrorAction SilentlyContinue |
  Select-Object Name |
  Out-File "$ReportPath\hkcu-policy-roots.txt"

# Firewall policies if relevant:
Get-NetFirewallProfile |
  Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction |
  Out-File "$ReportPath\firewall-profile-state.txt" `
  -ErrorAction SilentlyContinue

# Local group membership if relevant:
Get-LocalGroupMember `
  -Group Administrators `
  -ErrorAction SilentlyContinue |
  Select-Object Name,ObjectClass,PrincipalSource |
  Out-File "$ReportPath\local-administrators.txt"

# Drive maps if relevant:
Get-PSDrive |
  Out-File "$ReportPath\psdrives.txt"

# Printers if relevant:
Get-Printer `
  -ErrorAction SilentlyContinue |
  Select-Object Name,DriverName,PortName,Shared |
  Out-File "$ReportPath\printers.txt"

Write-Host "Target client migrated GPO validation complete."
Write-Host "Report path: $ReportPath"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Report_And_Backup_Skeleton
```powershell
# Run in the target domain after validation.
# Purpose: capture final reports, permissions, inheritance, comparison output, and final backup.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$TargetDomain = Get-ADDomain
$TargetDomainDN = $TargetDomain.DistinguishedName

$TargetGpoName = "LAB-Workstation-Security-Baseline"
$TargetOU = "OU=Pilot,OU=Workstations,OU=Lab,$TargetDomainDN"

$ReportPath = "C:\GPOPrep\Migration\FinalReports"
$BackupPath = "C:\GPOPrep\Migration\FinalBackup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Target GPO metadata and permissions.
Get-GPO `
  -Name $TargetGpoName |
  Format-List * |
  Out-File "$ReportPath\$TargetGpoName-final-metadata.txt"

Get-GPPermission `
  -Name $TargetGpoName `
  -All |
  Out-File "$ReportPath\$TargetGpoName-final-permissions.txt"

# Target OU inheritance.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\$TargetGpoName-final-target-ou-inheritance.txt"

# Final reports.
Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Html `
  -Path "$ReportPath\$TargetGpoName-final.html"

Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Xml `
  -Path "$ReportPath\$TargetGpoName-final.xml"

# Search for leftover source references.
Select-String `
  -Path "$ReportPath\$TargetGpoName-final.xml" `
  -Pattern "CORP\\","corp.local","\\\\FS1","\\\\PRINT1","S-1-5-21" |
  Out-File "$ReportPath\$TargetGpoName-final-leftover-source-reference-search.txt"

# Final backup.
Backup-GPO `
  -Name $TargetGpoName `
  -Path $BackupPath `
  -Comment "Final migrated target GPO backup after validation"

# Replication checks.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-gpo-migration.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-gpo-migration.txt"

Write-Host "Final GPO migration reports and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<source-gpo-name>"` | Confirms source GPO exists | Source GPO returns |
| `Get-GPOReport -Name "<source-gpo-name>" -ReportType Html -Path "<path>"` | Exports source readable report | HTML report exists |
| `Get-GPOReport -Name "<source-gpo-name>" -ReportType Xml -Path "<path>"` | Exports source comparison report | XML report exists |
| `Backup-GPO -Name "<source-gpo-name>" -Path "<source-backup-path>"` | Backs up source GPO | Source backup exists |
| `Get-GPPermission -Name "<source-gpo-name>" -All` | Records source permissions | Permission list returns |
| `Get-GPInheritance -Target "<source-ou-dn>"` | Records source link context | Link context returns |
| `Import-GPO -BackupGpoName "<source-gpo-name>" -TargetName "<target-gpo-name>" -Path "<backup-path>"` | Imports settings into target GPO | Import completes |
| `Import-GPO -BackupGpoName "<source-gpo-name>" -TargetName "<target-gpo-name>" -Path "<backup-path>" -MigrationTable "<migtable>"` | Imports settings with mapped references | Import completes with mappings |
| `Copy-GPO -SourceName "<source-gpo-name>" -TargetName "<copy-gpo-name>"` | Copies GPO inside same domain | Copy GPO exists |
| `Get-GPO -Name "<target-gpo-name>"` | Confirms target GPO exists | Target GPO returns |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path "<path>"` | Exports target readable report | HTML report exists |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Xml -Path "<path>"` | Exports target comparison report | XML report exists |
| `Compare-Object (Get-Content source.xml) (Get-Content target.xml)` | Compares source and target XML reports | Differences are visible |
| `Select-String -Path target.xml -Pattern "<source-domain>","<source-server>","S-1-5-21"` | Finds leftover source references | No unexpected hits |
| `Set-GPPermission -Name "<target-gpo-name>" -TargetName "<target-group>" -TargetType Group -PermissionLevel GpoApply` | Rebuilds target security filtering | Target group can apply |
| `New-GPLink -Name "<target-gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | Links target GPO to pilot OU | Link appears |
| `Get-GPInheritance -Target "<target-ou-dn>"` | Validates target link and precedence | Target GPO appears |
| `gpupdate /force` | Refreshes target client policy | Refresh completes |
| `gpresult /scope computer /r` | Validates computer-side application | Target GPO appears if scoped |
| `gpresult /scope user /r` | Validates user-side application | Target GPO appears if scoped |
| `gpresult /h C:\GPOPrep\Migration\TargetReports\gpresult-migrated-gpo.html` | Exports client RSOP evidence | HTML report exists |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews target client processing | Events show success or actionable errors |
| `Backup-GPO -Name "<target-gpo-name>" -Path "<target-backup-path>"` | Backs up final target GPO | Final backup exists |
| `repadmin /replsummary` | Checks target AD replication | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks target SYSVOL and DC health | Tests pass |

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Rollback
```powershell
# Run in the target domain.
# Purpose: safely roll back migrated or imported GPO changes.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$TargetDomainDN = (Get-ADDomain).DistinguishedName

$TargetGpoName = "LAB-Workstation-Security-Baseline"
$TargetOU = "OU=Pilot,OU=Workstations,OU=Lab,$TargetDomainDN"

$TargetBackupPath = "C:\GPOPrep\Migration\TargetBackup"
$ReportPath = "C:\GPOPrep\Migration\RollbackReports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Capture current target state before rollback.
if (Get-GPO -Name $TargetGpoName -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $TargetGpoName `
      -ReportType Html `
      -Path "$ReportPath\$TargetGpoName-before-rollback.html"

    Get-GPPermission `
      -Name $TargetGpoName `
      -All |
      Out-File "$ReportPath\$TargetGpoName-permissions-before-rollback.txt"
}

Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\target-ou-inheritance-before-rollback.txt"

# Rollback option 1:
# Disable the pilot link while preserving the target GPO.
Set-GPLink `
  -Name $TargetGpoName `
  -Target $TargetOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Restore target GPO from a pre-import backup if it existed before import.
# Confirm the backup path and backup identity before running.
# Restore-GPO `
#   -Name $TargetGpoName `
#   -Path $TargetBackupPath

# Rollback option 3:
# Delete imported target GPO if it was created only for this migration test.
# Remove-GPO -Name $TargetGpoName

# Rollback option 4:
# Remove target permissions or filtering changes manually if needed.
# Set-GPPermission -Name $TargetGpoName -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoApply

# Capture after rollback.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\target-ou-inheritance-after-rollback.txt"

if (Get-GPO -Name $TargetGpoName -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $TargetGpoName `
      -ReportType Html `
      -Path "$ReportPath\$TargetGpoName-after-rollback.html"
}

Write-Host "GPO migration rollback workflow complete."
Write-Host "Run gpupdate /force and gpresult /h on the target test client to validate rollback."
```

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Source backup fails | No permissions or invalid path | `Backup-GPO`; path check | Use authorized account and writable backup path |
| Source GPO report fails | GPO name wrong or report path invalid | `Get-GPO -Name "<source-gpo>"` | Correct GPO name and create report folder |
| Import fails | Source backup path wrong or backup GPO name mismatch | `Get-ChildItem <backup-path> -Recurse` | Use correct backup path and `BackupGpoName` |
| Target GPO overwritten unexpectedly | Imported into wrong existing GPO | Target before report | Restore target backup or import into new test GPO |
| Migration table not accepted | Invalid path or malformed migration table | Check `.migtable` file path | Recreate migration table in GPMC |
| Source groups remain in target report | Missing or incomplete security principal mapping | `Select-String target.xml -Pattern "CORP\\"` | Update migration table and reimport |
| Source UNC paths remain | UNC path mapping missing | `Select-String target.xml -Pattern "\\\\FS1"` | Map paths and reimport or edit target GPO |
| GPO imports but does not apply | Not linked, wrong OU, filtering, WMI, or disabled GPO | `Get-GPInheritance`; `gpresult /r` | Link to pilot OU and fix filtering |
| User settings missing | User-side GPO not linked to user OU or loopback not configured | `gpresult /scope user /r` | Link correctly or configure loopback |
| Computer settings missing | Computer object outside target OU | `Get-ADComputer -Identity "<client>" -Properties DistinguishedName` | Move computer or link GPO correctly |
| Security filtering fails | Source group not valid in target or target group missing | `Get-GPPermission -Name "<target-gpo>" -All` | Rebuild filtering with target groups |
| Delegation missing | Delegation does not migrate as intended | GPMC Delegation tab | Recreate target delegation |
| WMI filter missing | WMI filters not recreated or not attached | GPMC Scope tab | Recreate WMI filter in target domain |
| GPMC shows template errors | Target ADMX Central Store missing or mismatched | GPMC Administrative Templates | Update ADMX Central Store |
| GPP drive map fails | Target share missing or permissions wrong | `Test-Path \\server\share` | Create share and permissions |
| GPP printer fails | Target printer path invalid or driver issue | `Get-Printer`; client event logs | Fix print share and driver |
| Script fails | Script path references source domain or file missing | GPO report and SYSVOL path check | Copy scripts and update paths |
| Software installation fails | MSI path references source domain or target cannot access share | Event logs and UNC path check | Host MSI in target share and update path |
| Firewall rules block target management | Imported source IP scopes do not fit target network | `Get-NetFirewallRule`; report search | Adjust subnet scopes |
| Certificate policy imports but fails | CA, template, or trust references do not exist in target | PKI reports and client events | Rebuild PKI references |
| LAPS policy imports but does not work | Schema, OU permissions, or LAPS delegation not configured in target | LAPS cmdlets and event logs | Configure LAPS in target domain |
| BitLocker policy imports but recovery missing | AD DS recovery validation not performed | AD recovery object search | Validate recovery backup before scaling |
| WEF imports but forwards to old collector | Subscription Manager still points to source collector | Registry and GPO report | Update collector URL |
| AppLocker/WDAC imports but blocks target apps | Target app inventory differs from source lab | Event logs | Audit first and adjust rules |
| Comparison is noisy | XML includes metadata and ordering differences | Targeted searches | Compare key settings and source references instead of raw diff only |
| Target client gets old settings | Replication or GP cache delay | `repadmin /replsummary`; `gpupdate /force` | Fix replication and refresh client |
| Rollback does not remove effect | Another GPO wins or client has not refreshed | `gpresult /h` | Disable winning GPO and refresh |

# 24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Rebuilds target link order and inheritance behavior |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Rebuilds target filtering and delegation after migration |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Recreates WMI filters in target domain or lab |
| `08_Configure_Group_Policy_Preferences.md` | Helps validate migrated GPP items such as drive maps and printers |
| `09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md` | Helps validate migrated script paths and SYSVOL content |
| `10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md` | Helps validate migrated UNC paths, printers, and folder redirection |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Foundation workbook for backup, import, restore, copy, and reporting |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Validates migrated GPO application on target clients |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Diagnoses target scope, filtering, replication, and SYSVOL issues |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md` | Ensures target GPMC can display imported Administrative Template settings |
| `18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers.md` | Relevant when migrating loopback user settings |
| `19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs.md` | Relevant when migrating app control rules across different app inventories |
| `20_Configure_LAPS_And_Local_Administrator_Password_Policy.md` | Relevant when migrating LAPS policy but rebuilding AD permissions separately |
| `21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings.md` | Relevant when migrating certificate policies that depend on target CA and templates |
| `22_Configure_BitLocker_GPO_Settings.md` | Relevant when migrating encryption policy and recovery validation |
| `23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs.md` | Relevant when migrating audit and WEF collector settings |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms target domain infrastructure before migration validation |