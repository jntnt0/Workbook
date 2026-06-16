14_Backup_Restore_Import_And_Report_GPOs.md
# 14_Backup_Restore_Import_And_Report_GPOs

# 14_Backup_Restore_Import_And_Report_GPOs_Index
14_Backup_Restore_Import_And_Report_GPOs.md
14_Backup_Restore_Import_And_Report_GPOs
14_Backup_Restore_Import_And_Report_GPOs_Source_Basis
14_Backup_Restore_Import_And_Report_GPOs_Mental_Model
14_Backup_Restore_Import_And_Report_GPOs_Planning_Table
14_Backup_Restore_Import_And_Report_GPOs_Operation_Map
14_Backup_Restore_Import_And_Report_GPOs_Configuration_Checklist
14_Backup_Restore_Import_And_Report_GPOs_Precheck_Skeleton
14_Backup_Restore_Import_And_Report_GPOs_GPO_Inventory_And_Report_Skeleton
14_Backup_Restore_Import_And_Report_GPOs_Backup_All_GPOs_Skeleton
14_Backup_Restore_Import_And_Report_GPOs_Restore_GPO_Skeleton
14_Backup_Restore_Import_And_Report_GPOs_Import_GPO_Skeleton
14_Backup_Restore_Import_And_Report_GPOs_Copy_GPO_Skeleton
14_Backup_Restore_Import_And_Report_GPOs_Compare_Before_After_Skeleton
14_Backup_Restore_Import_And_Report_GPOs_Verification_Commands
14_Backup_Restore_Import_And_Report_GPOs_Rollback
14_Backup_Restore_Import_And_Report_GPOs_Failure_Checks
14_Backup_Restore_Import_And_Report_GPOs_Related_Labs

# 14_Backup_Restore_Import_And_Report_GPOs_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | GroupPolicy PowerShell module | GPO backup, restore, import, copy, reporting, and inventory |
| Microsoft Learn | `Backup-GPO` | Backing up one GPO or all GPOs |
| Microsoft Learn | `Restore-GPO` | Restoring a backed-up GPO to its original identity |
| Microsoft Learn | `Import-GPO` | Importing backed-up settings into another GPO |
| Microsoft Learn | `Copy-GPO` | Copying GPOs inside a domain or between domains |
| Microsoft Learn | `Get-GPOReport` | Exporting GPO settings as HTML or XML |
| Microsoft Learn | `Get-GPO` | Listing GPO inventory and metadata |
| Microsoft Learn | `Get-GPInheritance` | Validating links and inheritance after restore or import |
| Microsoft Learn | GPMC backup and restore workflow | GUI-based backup, restore, import, copy, and migration operations |
| Windows Server operational practice | Export reports before and after every GPO change | Preserving evidence and enabling safe rollback |

# 14_Backup_Restore_Import_And_Report_GPOs_Mental_Model
| Concept | Operational Meaning |
|---|---|
| GPO report | Human-readable or machine-readable export of GPO settings |
| HTML report | Best for review and documentation |
| XML report | Best for searching, comparing, and parsing |
| GPO backup | File-based backup containing GPO settings and metadata |
| Backup ID | Unique identifier for a specific GPO backup instance |
| Backup comment | Operator note stored with the backup |
| Restore-GPO | Restores a backed-up GPO to its original GPO identity |
| Import-GPO | Imports settings from a backup into a different target GPO |
| Copy-GPO | Creates a copy of a GPO, optionally across domains |
| Migration table | Maps domain-specific references such as users, groups, UNC paths, and security principals during import or copy |
| GPO identity | GPO GUID and display name relationship |
| GPO link | OU, domain, or site link that determines where a GPO applies |
| Backup path | Storage location for GPO backups |
| Report path | Storage location for exported GPO reports |
| SYSVOL | Stores GPO policy files replicated across domain controllers |
| AD portion | Stores GPO container metadata in Active Directory |
| First rule | Export a report and back up the GPO before changing, restoring, importing, or copying anything |
| Blunt rule | Restore changes the original GPO; import changes the target GPO; copy creates another GPO |

# 14_Backup_Restore_Import_And_Report_GPOs_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Source GPO | `CORP-Workstation-Security-Baseline` | `<source-gpo-name>` |
| Target GPO | `CORP-Workstation-Security-Baseline-Test` | `<target-gpo-name>` |
| Copy GPO | `CORP-Workstation-Security-Baseline-Copy` | `<copy-gpo-name>` |
| Target OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-ou-dn>` |
| Backup root | `C:\GPOPrep\Backup` | `<backup-root>` |
| Report root | `C:\GPOPrep\Reports` | `<report-root>` |
| Export date folder | `2026-06-13` | `<date-folder>` |
| Backup comment | `Pre-change backup before restore/import test` | `<backup-comment>` |
| Restore target | Original GPO | `<restore-target>` |
| Import target | New or existing test GPO | `<import-target>` |
| Migration table required | No for same-domain lab | `<yes-no>` |
| Link restored/imported GPO | Pilot OU only | `<target-link-scope>` |
| Validation client | `WIN11-01` | `<test-computer>` |
| Validation user | `tuser` | `<test-user>` |
| Rollback method | Restore prior backup or disable target link | `<rollback-plan>` |

# 14_Backup_Restore_Import_And_Report_GPOs_Operation_Map
| Operation | Cmdlet / Tool | Purpose | Changes Existing GPO? | Typical Use |
|---|---|---|---|---|
| Inventory | `Get-GPO -All` | List all GPOs and metadata | No | Baseline documentation |
| HTML Report | `Get-GPOReport -ReportType Html` | Human-readable GPO settings report | No | Review and evidence |
| XML Report | `Get-GPOReport -ReportType Xml` | Searchable GPO settings report | No | Compare and parsing |
| Backup One GPO | `Backup-GPO -Name <name>` | Back up one GPO | No | Before changing a specific GPO |
| Backup All GPOs | `Backup-GPO -All` | Back up every GPO in domain | No | Scheduled or pre-change backup |
| Restore GPO | `Restore-GPO` | Restore original GPO from backup | Yes | Roll back original GPO settings |
| Import GPO | `Import-GPO` | Import settings from backup into target GPO | Yes, target GPO | Build test copy or migrate settings |
| Copy GPO | `Copy-GPO` | Copy a GPO to new GPO | Creates or updates destination | Duplicate baseline |
| Link GPO | `New-GPLink` / `Set-GPLink` | Apply restored, imported, or copied GPO to scope | Link changes | Pilot validation |
| Compare | XML report diff or `Compare-Object` | Identify before/after setting differences | No | Audit and troubleshooting |
| GUI Backup | GPMC | Backup selected or all GPOs | No | Operator-friendly backup |
| GUI Restore | GPMC | Restore backed-up GPO | Yes | Operator-friendly rollback |
| GUI Import | GPMC | Import settings into a GPO | Yes, target GPO | Build GPO from backup |

# 14_Backup_Restore_Import_And_Report_GPOs_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 3 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 6 | Confirm GPMC opens | Management Host | `gpmc.msc` | GPMC opens |
| 7 | Create report root | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report root exists |
| 8 | Create backup root | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup root exists |
| 9 | Create date-stamped folders | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\GPO-$(Get-Date -Format yyyyMMdd),C:\GPOPrep\Backup\GPO-$(Get-Date -Format yyyyMMdd)` | Dated folders exist |
| 10 | Inventory all GPOs | Management Host | `Get-GPO -All` | All GPOs return |
| 11 | Export GPO inventory CSV | Management Host | `Get-GPO -All \| Select DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime \| Export-Csv C:\GPOPrep\Reports\gpo-inventory.csv -NoTypeInformation` | Inventory CSV exists |
| 12 | Export all GPO HTML reports | Management Host | `Get-GPO -All \| ForEach-Object { Get-GPOReport -Guid $_.Id -ReportType Html -Path "C:\GPOPrep\Reports\$($_.DisplayName).html" }` | HTML reports exist |
| 13 | Export all GPO XML reports | Management Host | `Get-GPO -All \| ForEach-Object { Get-GPOReport -Guid $_.Id -ReportType Xml -Path "C:\GPOPrep\Reports\$($_.DisplayName).xml" }` | XML reports exist |
| 14 | Back up all GPOs | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup -Comment "Full GPO backup"` | Backup set exists |
| 15 | Back up one source GPO | Management Host | `Backup-GPO -Name "<source-gpo-name>" -Path C:\GPOPrep\Backup -Comment "Single GPO backup"` | Source GPO backup exists |
| 16 | List backup folders | Management Host | `Get-ChildItem C:\GPOPrep\Backup -Recurse` | Backup files are visible |
| 17 | Capture source GPO report before test | Management Host | `Get-GPOReport -Name "<source-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<source-gpo-name>-before.html` | Before report exists |
| 18 | Create test target GPO for import | Management Host | `New-GPO -Name "<target-gpo-name>" -Comment "Import test target"` | Target GPO exists |
| 19 | Import source backup into test target GPO | Management Host | `Import-GPO -BackupGpoName "<source-gpo-name>" -TargetName "<target-gpo-name>" -Path C:\GPOPrep\Backup` | Source settings import into target GPO |
| 20 | Export target GPO report after import | Management Host | `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<target-gpo-name>-after-import.html` | Imported settings appear |
| 21 | Copy source GPO to copy GPO | Management Host | `Copy-GPO -SourceName "<source-gpo-name>" -TargetName "<copy-gpo-name>"` | Copy GPO is created |
| 22 | Export copy GPO report | Management Host | `Get-GPOReport -Name "<copy-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<copy-gpo-name>.html` | Copy report exists |
| 23 | Link imported or copied GPO to pilot OU if required | Management Host | `New-GPLink -Name "<target-gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | Test GPO is linked to pilot only |
| 24 | Confirm link state | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Target or copy GPO appears in link list |
| 25 | Force client refresh | Test Client | `gpupdate /force` | Policy refresh completes |
| 26 | Validate applied GPOs | Test Client | `gpresult /scope computer /r` | Expected GPO appears if linked to computer scope |
| 27 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-after-gpo-import.html` | Client RSOP report exists |
| 28 | Restore original source GPO from backup if testing rollback | Management Host | `Restore-GPO -Name "<source-gpo-name>" -Path C:\GPOPrep\Backup` | Original GPO is restored from backup |
| 29 | Export post-restore report | Management Host | `Get-GPOReport -Name "<source-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<source-gpo-name>-after-restore.html` | Restored report exists |
| 30 | Compare before and after XML reports | Management Host | `Compare-Object (Get-Content before.xml) (Get-Content after.xml)` | Differences are visible |
| 31 | Back up final state | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup -Comment "Post import restore final state"` | Final backup exists |
| 32 | Document operation | Operator | `Record source GPO, backup path, backup ID, target GPO, import/copy/restore action, reports, and validation result` | GPO lifecycle action is documented |

# 14_Backup_Restore_Import_And_Report_GPOs_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before backing up, reporting, restoring, importing, or copying GPOs.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$BasePath = "C:\GPOPrep"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$ReportRoot = "$BasePath\Reports\GPO-$DateStamp"
$BackupRoot = "$BasePath\Backup\GPO-$DateStamp"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportRoot,$BackupRoot

# Confirm domain and tool readiness.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
Get-Module -ListAvailable GroupPolicy
Get-Command -Module GroupPolicy | Sort-Object Name | Select-Object Name

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm current GPO inventory is readable.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportRoot\gpo-inventory-precheck.csv" -NoTypeInformation

# Confirm basic AD replication health before GPO lifecycle work.
repadmin /replsummary |
  Out-File "$ReportRoot\repadmin-replsummary-before-gpo-lifecycle.txt"

# Confirm SYSVOL health at a basic level.
dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportRoot\dcdiag-sysvol-advertising-before-gpo-lifecycle.txt"

Write-Host "Precheck complete."
Write-Host "Report root: $ReportRoot"
Write-Host "Backup root: $BackupRoot"
```

# 14_Backup_Restore_Import_And_Report_GPOs_GPO_Inventory_And_Report_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: export GPO inventory plus HTML and XML reports for all GPOs.

Import-Module GroupPolicy

$BasePath = "C:\GPOPrep"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$ReportRoot = "$BasePath\Reports\GPO-Reports-$DateStamp"
$HtmlRoot = "$ReportRoot\Html"
$XmlRoot = "$ReportRoot\Xml"

New-Item -ItemType Directory -Force -Path $ReportRoot,$HtmlRoot,$XmlRoot

# Helper function to create filesystem-safe filenames.
function ConvertTo-SafeFileName {
    param(
        [Parameter(Mandatory)]
        [string]$Name
    )

    $Invalid = [System.IO.Path]::GetInvalidFileNameChars()
    $Safe = $Name

    foreach ($Char in $Invalid) {
        $Safe = $Safe.Replace($Char, "_")
    }

    return $Safe
}

# Get all GPOs.
$Gpos = Get-GPO -All | Sort-Object DisplayName

# Export inventory.
$Gpos |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime,UserVersion,ComputerVersion |
  Export-Csv "$ReportRoot\gpo-inventory.csv" -NoTypeInformation

# Export reports for every GPO.
foreach ($Gpo in $Gpos) {
    $SafeName = ConvertTo-SafeFileName -Name $Gpo.DisplayName

    Get-GPOReport `
      -Guid $Gpo.Id `
      -ReportType Html `
      -Path "$HtmlRoot\$SafeName.html"

    Get-GPOReport `
      -Guid $Gpo.Id `
      -ReportType Xml `
      -Path "$XmlRoot\$SafeName.xml"
}

# Search all XML reports for common high-risk settings.
Select-String `
  -Path "$XmlRoot\*.xml" `
  -Pattern "Administrators","Remote Desktop","Firewall","WindowsUpdate","Defender","Folder Redirection","Scripts","Drive Maps","DisableCMD","NoControlPanel" |
  Out-File "$ReportRoot\gpo-report-keyword-search.txt"

Write-Host "GPO inventory and reports exported."
Write-Host "Report root: $ReportRoot"
```

# 14_Backup_Restore_Import_And_Report_GPOs_Backup_All_GPOs_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: back up all GPOs and create a backup manifest.

Import-Module GroupPolicy

$BasePath = "C:\GPOPrep"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$BackupRoot = "$BasePath\Backup\GPO-Backup-$DateStamp"
$ReportRoot = "$BasePath\Reports\GPO-Backup-$DateStamp"

New-Item -ItemType Directory -Force -Path $BackupRoot,$ReportRoot

$BackupComment = "Full GPO backup created $DateStamp before GPO lifecycle operation"

# Back up all GPOs.
$BackupResult = Backup-GPO `
  -All `
  -Path $BackupRoot `
  -Comment $BackupComment

# Export backup result manifest.
$BackupResult |
  Select-Object DisplayName,GpoId,Id,CreationTime,DomainName,Comment |
  Export-Csv "$ReportRoot\gpo-backup-result.csv" -NoTypeInformation

# Export current inventory beside the backup.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportRoot\gpo-inventory-at-backup-time.csv" -NoTypeInformation

# List backup files and folders.
Get-ChildItem `
  -Path $BackupRoot `
  -Recurse |
  Select-Object FullName,Length,LastWriteTime |
  Out-File "$ReportRoot\gpo-backup-file-list.txt"

Write-Host "Full GPO backup complete."
Write-Host "Backup root: $BackupRoot"
Write-Host "Report root: $ReportRoot"
```

# 14_Backup_Restore_Import_And_Report_GPOs_Restore_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: restore an original GPO from a backup.
# Restore-GPO is for restoring the original GPO identity, not importing settings into a different GPO.

Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Security-Baseline"
$BackupRoot = "C:\GPOPrep\Backup\GPO-Backup-20260613-120000"
$ReportRoot = "C:\GPOPrep\Reports\GPO-Restore-Test"

New-Item -ItemType Directory -Force -Path $ReportRoot

# Confirm the GPO currently exists.
Get-GPO -Name $GpoName

# Export current state before restore.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportRoot\$GpoName-before-restore.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportRoot\$GpoName-before-restore.xml"

# Review available backups in the backup path.
Get-ChildItem `
  -Path $BackupRoot `
  -Recurse |
  Select-Object FullName,Length,LastWriteTime |
  Out-File "$ReportRoot\available-backup-files.txt"

# Restore the GPO from backup.
Restore-GPO `
  -Name $GpoName `
  -Path $BackupRoot

# Export restored state.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportRoot\$GpoName-after-restore.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportRoot\$GpoName-after-restore.xml"

# Compare before and after XML as raw text.
Compare-Object `
  -ReferenceObject (Get-Content "$ReportRoot\$GpoName-before-restore.xml") `
  -DifferenceObject (Get-Content "$ReportRoot\$GpoName-after-restore.xml") |
  Out-File "$ReportRoot\$GpoName-restore-compare.txt"

Write-Host "Restore complete for GPO: $GpoName"
Write-Host "Reports written to: $ReportRoot"
```

# 14_Backup_Restore_Import_And_Report_GPOs_Import_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: import backed-up GPO settings into a separate target GPO.
# Import-GPO changes the target GPO settings but does not restore the original GPO identity.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$SourceBackupGpoName = "CORP-Workstation-Security-Baseline"
$TargetGpoName = "CORP-Workstation-Security-Baseline-Imported-Test"
$TargetOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$BackupRoot = "C:\GPOPrep\Backup\GPO-Backup-20260613-120000"
$ReportRoot = "C:\GPOPrep\Reports\GPO-Import-Test"

New-Item -ItemType Directory -Force -Path $ReportRoot

# Confirm target OU exists.
Get-ADOrganizationalUnit -Identity $TargetOU

# Create target GPO if missing.
if (-not (Get-GPO -Name $TargetGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $TargetGpoName `
      -Comment "Imported test GPO from backup of $SourceBackupGpoName"
}

# Export target before import.
Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Html `
  -Path "$ReportRoot\$TargetGpoName-before-import.html"

Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Xml `
  -Path "$ReportRoot\$TargetGpoName-before-import.xml"

# Import settings from backup into target GPO.
Import-GPO `
  -BackupGpoName $SourceBackupGpoName `
  -TargetName $TargetGpoName `
  -Path $BackupRoot

# Export target after import.
Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Html `
  -Path "$ReportRoot\$TargetGpoName-after-import.html"

Get-GPOReport `
  -Name $TargetGpoName `
  -ReportType Xml `
  -Path "$ReportRoot\$TargetGpoName-after-import.xml"

# Link imported GPO to pilot OU if testing application.
$Inheritance = Get-GPInheritance -Target $TargetOU
$ExistingLink = $Inheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $TargetGpoName }

if (-not $ExistingLink) {
    New-GPLink `
      -Name $TargetGpoName `
      -Target $TargetOU `
      -LinkEnabled Yes
}

# Keep imported test GPO unenforced by default.
Set-GPLink `
  -Name $TargetGpoName `
  -Target $TargetOU `
  -LinkEnabled Yes `
  -Enforced No

# Confirm link state.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportRoot\target-ou-inheritance-after-import.txt"

Write-Host "Import complete."
Write-Host "Source backup GPO: $SourceBackupGpoName"
Write-Host "Target GPO: $TargetGpoName"
Write-Host "Report root: $ReportRoot"
```

# 14_Backup_Restore_Import_And_Report_GPOs_Copy_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: copy an existing GPO to a new GPO for testing or reuse.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$SourceGpoName = "CORP-Workstation-Security-Baseline"
$CopyGpoName = "CORP-Workstation-Security-Baseline-Copy"
$TargetOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$ReportRoot = "C:\GPOPrep\Reports\GPO-Copy-Test"
$BackupRoot = "C:\GPOPrep\Backup\GPO-Copy-Test"

New-Item -ItemType Directory -Force -Path $ReportRoot,$BackupRoot

# Confirm source GPO and target OU.
Get-GPO -Name $SourceGpoName
Get-ADOrganizationalUnit -Identity $TargetOU

# Export source report before copy.
Get-GPOReport `
  -Name $SourceGpoName `
  -ReportType Html `
  -Path "$ReportRoot\$SourceGpoName-source-before-copy.html"

# Copy source GPO to a new GPO.
Copy-GPO `
  -SourceName $SourceGpoName `
  -TargetName $CopyGpoName

# Export copied GPO reports.
Get-GPOReport `
  -Name $CopyGpoName `
  -ReportType Html `
  -Path "$ReportRoot\$CopyGpoName-after-copy.html"

Get-GPOReport `
  -Name $CopyGpoName `
  -ReportType Xml `
  -Path "$ReportRoot\$CopyGpoName-after-copy.xml"

# Back up copied GPO.
Backup-GPO `
  -Name $CopyGpoName `
  -Path $BackupRoot `
  -Comment "Backup of copied GPO"

# Optional: link copied GPO to pilot OU.
$Inheritance = Get-GPInheritance -Target $TargetOU
$ExistingLink = $Inheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $CopyGpoName }

if (-not $ExistingLink) {
    New-GPLink `
      -Name $CopyGpoName `
      -Target $TargetOU `
      -LinkEnabled Yes
}

Set-GPLink `
  -Name $CopyGpoName `
  -Target $TargetOU `
  -Enforced No

Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportRoot\target-ou-inheritance-after-copy.txt"

Write-Host "Copy complete."
Write-Host "Source GPO: $SourceGpoName"
Write-Host "Copy GPO: $CopyGpoName"
```

# 14_Backup_Restore_Import_And_Report_GPOs_Compare_Before_After_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: compare before and after XML reports for a GPO lifecycle operation.

$ReportRoot = "C:\GPOPrep\Reports\GPO-Compare"

$BeforeXml = "$ReportRoot\before.xml"
$AfterXml = "$ReportRoot\after.xml"
$CompareOutput = "$ReportRoot\gpo-xml-compare.txt"

New-Item -ItemType Directory -Force -Path $ReportRoot

# Example:
# Copy or rename XML reports into before.xml and after.xml before running this.
# Or update the variables above to point directly to existing report files.

if ((Test-Path $BeforeXml) -and (Test-Path $AfterXml)) {
    Compare-Object `
      -ReferenceObject (Get-Content $BeforeXml) `
      -DifferenceObject (Get-Content $AfterXml) |
      Out-File $CompareOutput

    Write-Host "Compare output written to: $CompareOutput"
}
else {
    Write-Host "Before or after XML report is missing."
}

# Search for key policy categories in after report.
if (Test-Path $AfterXml) {
    Select-String `
      -Path $AfterXml `
      -Pattern "Security","Registry","Scripts","Drive","Printer","Folder Redirection","WindowsUpdate","Defender","Firewall","Remote Desktop" |
      Out-File "$ReportRoot\after-report-keyword-search.txt"
}
```

# 14_Backup_Restore_Import_And_Report_GPOs_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -All` | Lists all GPOs | All GPOs return |
| `Get-GPO -Name "<gpo-name>"` | Confirms a specific GPO exists | GPO object returns |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path "<path>\<gpo>.html"` | Exports readable GPO report | HTML report exists |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Xml -Path "<path>\<gpo>.xml"` | Exports searchable GPO report | XML report exists |
| `Backup-GPO -Name "<gpo-name>" -Path "<backup-path>" -Comment "<comment>"` | Backs up a single GPO | Backup result returns |
| `Backup-GPO -All -Path "<backup-path>" -Comment "<comment>"` | Backs up all GPOs | Backup result returns for all GPOs |
| `Restore-GPO -Name "<gpo-name>" -Path "<backup-path>"` | Restores original GPO from backup | GPO settings restored |
| `Import-GPO -BackupGpoName "<source-gpo>" -TargetName "<target-gpo>" -Path "<backup-path>"` | Imports backup settings into target GPO | Target GPO receives imported settings |
| `Copy-GPO -SourceName "<source-gpo>" -TargetName "<copy-gpo>"` | Copies source GPO into new GPO | Copy GPO appears |
| `Get-GPInheritance -Target "<target-ou-dn>"` | Confirms restored, imported, or copied GPO link scope | Expected links appear |
| `New-GPLink -Name "<gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | Links a test GPO to pilot OU | Link appears |
| `Set-GPLink -Name "<gpo-name>" -Target "<target-ou-dn>" -LinkEnabled No` | Disables test GPO link | Link remains but disabled |
| `gpupdate /force` | Refreshes client policy | Policy update completes |
| `gpresult /scope computer /r` | Validates computer-side applied GPOs | Expected GPO appears if in scope |
| `gpresult /scope user /r` | Validates user-side applied GPOs | Expected GPO appears if in scope |
| `gpresult /h C:\GPOPrep\Reports\gpresult-after-gpo-lifecycle.html` | Exports client RSOP report | HTML report exists |
| `repadmin /replsummary` | Checks AD replication before and after lifecycle work | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks DC and SYSVOL health | Tests pass |

# 14_Backup_Restore_Import_And_Report_GPOs_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: rollback GPO lifecycle test actions.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$ImportedGpoName = "CORP-Workstation-Security-Baseline-Imported-Test"
$CopyGpoName = "CORP-Workstation-Security-Baseline-Copy"
$OriginalGpoName = "CORP-Workstation-Security-Baseline"

$BackupRoot = "C:\GPOPrep\Backup\GPO-Backup-20260613-120000"
$ReportRoot = "C:\GPOPrep\Reports\GPO-Lifecycle-Rollback"

New-Item -ItemType Directory -Force -Path $ReportRoot

# Capture state before rollback.
foreach ($GpoName in @($ImportedGpoName,$CopyGpoName,$OriginalGpoName)) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportRoot\$GpoName-before-rollback.html"
    }
}

# Rollback option 1:
# Disable pilot links for imported or copied GPOs.
foreach ($GpoName in @($ImportedGpoName,$CopyGpoName)) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Set-GPLink `
          -Name $GpoName `
          -Target $TargetOU `
          -LinkEnabled No `
          -ErrorAction SilentlyContinue
    }
}

# Rollback option 2:
# Remove imported or copied test GPOs if they were created only for this lab.
# Remove-GPO -Name $ImportedGpoName -Confirm:$false
# Remove-GPO -Name $CopyGpoName -Confirm:$false

# Rollback option 3:
# Restore original GPO from backup if it was changed during testing.
# Confirm backup path before running.
# Restore-GPO `
#   -Name $OriginalGpoName `
#   -Path $BackupRoot

# Capture state after rollback.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportRoot\target-ou-inheritance-after-lifecycle-rollback.txt"

foreach ($GpoName in @($ImportedGpoName,$CopyGpoName,$OriginalGpoName)) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportRoot\$GpoName-after-rollback.html"
    }
}

Write-Host "Rollback workflow complete."
Write-Host "Validate with gpupdate and gpresult on any test clients in scope."
```

# 14_Backup_Restore_Import_And_Report_GPOs_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `Backup-GPO` is not recognized | GroupPolicy module missing | `Get-Module -ListAvailable GroupPolicy` | Complete `01_Install_Group_Policy_Management_Tools.md` |
| Backup fails with access denied | User lacks GPO permissions or backup path write access | `whoami /groups`; `Test-Path <backup-path>` | Use authorized account and writable path |
| Report export fails | Report path missing or invalid filename | `Test-Path <report-path>` | Create path and sanitize GPO display names |
| Some GPO reports fail | GPO name contains invalid filename characters | Use GUID-based report export | Export by `-Guid` and safe filename |
| Restore-GPO fails | Wrong backup path, wrong GPO name, or backup missing | Check backup folder and backup result CSV | Use correct backup root and source GPO name |
| Restore changed the wrong GPO | Restore used original GPO identity | `Get-GPOReport` before and after | Use Import-GPO for target GPO testing instead |
| Import-GPO fails | Target GPO missing or backup source not found | `Get-GPO -Name <target>`; inspect backup path | Create target GPO and verify backup source name |
| Imported GPO does not apply | GPO was imported but not linked | `Get-GPInheritance -Target <ou>` | Link imported GPO to pilot OU |
| Copied GPO does not apply | Copy created but not linked or filtering denies it | `Get-GPInheritance`; `Get-GPPermission` | Link copy GPO and fix filtering |
| Copied or imported GPO references old groups | Domain-specific references need migration | GPO report search for old domain names | Use migration table or manually remap references |
| Copied GPO has old UNC paths | File share paths copied as-is | Search XML report for `\\` paths | Update paths or use migration table |
| Imported GPO overwrote target settings | Import replaces target settings with backup settings | Target before/after reports | Restore target backup or use a blank test GPO next time |
| GPO link state not restored by import | Import transfers settings, not link placement | `Get-GPInheritance` | Recreate links manually |
| Backup does not include intended latest state | Backup was taken before latest edits or replication delay existed | GPO report and backup timestamp | Back up after confirming replication |
| GPO differs between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before restore/import/copy |
| Client still sees old settings | Policy not refreshed or winning GPO differs | `gpupdate /force`; `gpresult /h` | Refresh policy and check precedence |
| Restore appears successful but setting remains changed | Another GPO wins or local tattooed setting remains | `gpresult /h`; registry/security export | Identify winning GPO and remove tattooed state if applicable |
| XML compare is noisy | XML contains metadata or ordering changes | Compare setting-specific sections or keyword search | Use reports plus targeted validation |
| Backup folder is hard to audit | No manifest or comments | Backup result CSV missing | Export backup result and use meaningful comments |
| GPMC and PowerShell show different state | Console cache or replication delay | Refresh GPMC; `repadmin /replsummary` | Refresh console and verify DC consistency |
| Rollback removes too much | Imported or copied GPO linked too broadly | `Get-GPInheritance`; `Get-GPPermission` | Disable pilot link first, then remove test GPO only after validation |

# 14_Backup_Restore_Import_And_Report_GPOs_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes baseline GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Confirms link and precedence before restoring or importing GPOs |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Confirms who can apply and administer restored or imported GPOs |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Produces GPOs that should be backed up and reported |
| `07_Configure_User_Administrative_Template_Settings.md` | Produces user GPOs that should be backed up and reported |
| `08_Configure_Group_Policy_Preferences.md` | Produces GPP items that should be included in reports and backups |
| `09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md` | Produces script GPOs where SYSVOL content should be reviewed in reports and backups |
| `11_Configure_Security_Baseline_GPO_Settings.md` | High-risk baseline GPO requiring backup before changes |
| `12_Configure_Windows_Update_And_Defender_GPO_Settings.md` | Update and Defender policy GPO requiring report and backup history |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md` | Remote access GPO requiring careful backup and rollback path |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms GPO infrastructure health before lifecycle work |
| `15_Troubleshoot_Group_Policy_Processing_And_RSOP.md` | Next workbook for diagnosing effective policy after backup, restore, import, or copy |
