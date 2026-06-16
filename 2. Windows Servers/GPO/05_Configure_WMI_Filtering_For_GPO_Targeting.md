05_Configure_WMI_Filtering_For_GPO_Targeting.md
# 05_Configure_WMI_Filtering_For_GPO_Targeting

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Index
05_Configure_WMI_Filtering_For_GPO_Targeting.md
05_Configure_WMI_Filtering_For_GPO_Targeting
05_Configure_WMI_Filtering_For_GPO_Targeting_Source_Basis
05_Configure_WMI_Filtering_For_GPO_Targeting_Mental_Model
05_Configure_WMI_Filtering_For_GPO_Targeting_Planning_Table
05_Configure_WMI_Filtering_For_GPO_Targeting_Configuration_Checklist
05_Configure_WMI_Filtering_For_GPO_Targeting_Precheck_Skeleton
05_Configure_WMI_Filtering_For_GPO_Targeting_WMI_Query_Test_Skeleton
05_Configure_WMI_Filtering_For_GPO_Targeting_GPMC_Create_Filter_Skeleton
05_Configure_WMI_Filtering_For_GPO_Targeting_AD_WMI_Filter_Inventory_Skeleton
05_Configure_WMI_Filtering_For_GPO_Targeting_GPO_Report_Validation_Skeleton
05_Configure_WMI_Filtering_For_GPO_Targeting_Client_RSOP_Validation_Skeleton
05_Configure_WMI_Filtering_For_GPO_Targeting_Report_And_Backup_Skeleton
05_Configure_WMI_Filtering_For_GPO_Targeting_Verification_Commands
05_Configure_WMI_Filtering_For_GPO_Targeting_Rollback
05_Configure_WMI_Filtering_For_GPO_Targeting_Failure_Checks
05_Configure_WMI_Filtering_For_GPO_Targeting_Related_Labs

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy Management Console | Creating WMI filters and linking WMI filters to GPOs |
| Microsoft Learn | WMI filters for Group Policy | Targeting GPO application based on WMI query results |
| Microsoft Learn | `Get-GPOReport` | Exporting GPO reports to confirm WMI filter association |
| Microsoft Learn | `Backup-GPO` | Backing up GPO state before and after targeting changes |
| Microsoft Learn | `Get-GPInheritance` | Confirming GPO link path before troubleshooting WMI targeting |
| Microsoft Learn | `gpupdate` and `gpresult` | Client-side proof of applied or denied GPOs |
| Active Directory PowerShell | `Get-ADObject` | Inventorying WMI filter objects stored in Active Directory |
| Windows WMI / CIM | `Get-CimInstance` and WQL queries | Testing whether a client matches a proposed WMI filter |
| Windows Server operational practice | Prefer OU and security filtering before WMI filtering | Avoiding slow or fragile Group Policy processing |

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Mental_Model
| Concept | Operational Meaning |
|---|---|
| WMI filter | A WQL query attached to a GPO that decides whether the GPO applies |
| WQL | WMI Query Language used to query client properties |
| Namespace | WMI location queried by the filter, usually `root\CIMv2` |
| Query result true | If the query returns at least one object, the WMI filter matches |
| Query result false | If the query returns no objects, the GPO is denied by WMI filter |
| GPO link | The GPO must still be linked to an OU, domain, or site |
| Security filtering | The target must still pass security filtering |
| WMI filtering | Additional condition evaluated after the client is already in GPO scope |
| WMI filter storage | WMI filters are stored in AD under `CN=SOM,CN=WMIPolicy,CN=System,<domain DN>` |
| GPMC role | Safest native tool for creating and attaching WMI filters |
| PowerShell limitation | Built-in GroupPolicy cmdlets do not provide the same clean create/link workflow for WMI filters as they do for GPOs |
| Client cost | WMI filters are evaluated by the client during policy processing |
| Slow filter | Expensive or broken WMI queries can slow logon or policy refresh |
| Best use case | OS family, OS version, hardware class, server vs workstation targeting |
| Bad use case | Replacing clean OU design or basic security filtering |
| First rule | Test the WQL query directly on target and non-target clients before attaching it to a GPO |
| Blunt rule | If OU targeting or security filtering can do the job, do not use WMI filtering |

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-ou-dn>` |
| Target GPO | `CORP-Workstation-Baseline` | `<target-gpo-name>` |
| WMI filter name | `WMI-Windows-11-Workstations` | `<wmi-filter-name>` |
| WMI filter description | `Applies only to Windows 11 workstation clients` | `<wmi-filter-description>` |
| WMI namespace | `root\CIMv2` | `<wmi-namespace>` |
| WMI query | `SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1 AND BuildNumber >= "22000"` | `<wmi-query>` |
| Positive test client | `WIN11-01` | `<matching-client>` |
| Negative test client | `WIN10-01` or `SERVER01` | `<nonmatching-client>` |
| Filtering group | `GG_GPO_Workstation_Baseline_Apply` | `<security-filter-group>` |
| Expected result | GPO applies only when link, ACL, and WMI all match | `<expected-result>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Detach WMI filter from GPO | `<rollback-plan>` |

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target GPO exists | Management Host | `Get-GPO -Name "<target-gpo-name>"` | GPO returns |
| 6 | Confirm target OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-ou-dn>"` | OU returns |
| 7 | Confirm GPO is linked to target OU | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Target GPO appears in link list |
| 8 | Confirm current GPO permissions | Management Host | `Get-GPPermission -Name "<target-gpo-name>" -All` | Read and Apply permissions are known |
| 9 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 11 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 12 | Back up GPO before WMI change | Management Host | `Backup-GPO -Name "<target-gpo-name>" -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 13 | Export pre-change GPO report | Management Host | `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<target-gpo-name>-before-wmi.html` | Before report exists |
| 14 | Test WMI query on matching client | Matching Client | `Get-CimInstance -Namespace root\CIMv2 -Query '<wmi-query>'` | Query returns object |
| 15 | Test WMI query on nonmatching client | Nonmatching Client | `Get-CimInstance -Namespace root\CIMv2 -Query '<wmi-query>'` | Query returns no object |
| 16 | Open GPMC | Management Host | `gpmc.msc` | Group Policy Management Console opens |
| 17 | Create WMI filter in GPMC | Management Host | `GPMC: Forest > Domains > <domain> > WMI Filters > New` | New WMI filter exists |
| 18 | Name WMI filter | Management Host | `GPMC: Name = <wmi-filter-name>` | Filter name matches standard |
| 19 | Add WMI query | Management Host | `GPMC: Namespace = root\CIMv2; Query = <wmi-query>` | Query is saved |
| 20 | Attach WMI filter to target GPO | Management Host | `GPMC: Select GPO > Scope tab > WMI Filtering > select <wmi-filter-name>` | GPO references WMI filter |
| 21 | Confirm WMI filter inventory in AD | Management Host | `Get-ADObject -LDAPFilter "(objectClass=msWMI-Som)" -SearchBase "CN=SOM,CN=WMIPolicy,CN=System,<domain-dn>" -Properties *` | WMI filter object appears |
| 22 | Export post-change GPO report | Management Host | `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<target-gpo-name>-after-wmi.html` | Report includes WMI filter association |
| 23 | Force policy refresh on matching client | Matching Client | `gpupdate /force` | Policy refresh completes |
| 24 | Validate matching client applies GPO | Matching Client | `gpresult /scope computer /r` | GPO appears under Applied Group Policy Objects |
| 25 | Force policy refresh on nonmatching client | Nonmatching Client | `gpupdate /force` | Policy refresh completes |
| 26 | Validate nonmatching client does not apply GPO | Nonmatching Client | `gpresult /scope computer /r` | GPO appears denied by WMI filter or is absent from applied list |
| 27 | Export matching client GPResult report | Matching Client | `gpresult /h C:\GPOPrep\Reports\gpresult-wmi-matching.html` | HTML report exists |
| 28 | Export nonmatching client GPResult report | Nonmatching Client | `gpresult /h C:\GPOPrep\Reports\gpresult-wmi-nonmatching.html` | HTML report exists |
| 29 | Review Group Policy events | Test Clients | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Processing events show success or denial reason |
| 30 | Back up final GPO state | Management Host | `Backup-GPO -Name "<target-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 31 | Document targeting result | Operator | `Record GPO, WMI filter, query, matching client, nonmatching client, reports, and rollback plan` | WMI targeting design is documented |

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture current GPO, OU, permission, and WMI filter state before WMI targeting changes.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"
$GpoName = "CORP-Workstation-Baseline"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportPath,$BackupPath

# Confirm domain and tools.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
Get-Module -ListAvailable GroupPolicy
Get-Command -Module GroupPolicy | Select-Object Name | Sort-Object Name

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm target OU and GPO.
Get-ADOrganizationalUnit -Identity $TargetOU
Get-GPO -Name $GpoName

# Capture GPO link path.
Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\inheritance-before-wmi-filtering.txt"

# Capture permissions.
Get-GPPermission -Name $GpoName -All |
  Out-File "$ReportPath\$GpoName-permissions-before-wmi.txt"

# Inventory existing WMI filters in AD.
$WmiFilterSearchBase = "CN=SOM,CN=WMIPolicy,CN=System,$DomainDN"

if (Get-ADObject -Identity $WmiFilterSearchBase -ErrorAction SilentlyContinue) {
    Get-ADObject `
      -LDAPFilter "(objectClass=msWMI-Som)" `
      -SearchBase $WmiFilterSearchBase `
      -Properties msWMI-Name,msWMI-ID,msWMI-Author,msWMI-Parm1,msWMI-Parm2,whenCreated,whenChanged |
      Select-Object Name,msWMI-Name,msWMI-ID,msWMI-Author,msWMI-Parm1,msWMI-Parm2,whenCreated,whenChanged |
      Export-Csv "$ReportPath\wmi-filters-before.csv" -NoTypeInformation
}

# Export current GPO report and backup.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-before-wmi-filtering.html"

Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_WMI_Query_Test_Skeleton
```powershell
# Run on matching and nonmatching clients before attaching the filter to a GPO.
# Purpose: prove whether the WMI query returns expected results.

$ReportPath = "C:\GPOPrep\Reports"
New-Item -ItemType Directory -Force -Path $ReportPath

# Example 1:
# Workstation only.
$QueryWorkstationOnly = 'SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1'

# Example 2:
# Server only.
$QueryServerOnly = 'SELECT * FROM Win32_OperatingSystem WHERE ProductType <> 1'

# Example 3:
# Windows 11 style client targeting by build number.
# Windows 11 starts at build 22000.
$QueryWindows11Workstation = 'SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1 AND BuildNumber >= "22000"'

# Example 4:
# 64-bit operating system.
$Query64BitOS = 'SELECT * FROM Win32_OperatingSystem WHERE OSArchitecture = "64-bit"'

# Example 5:
# Laptop style targeting. Returns results only if a battery class is present.
$QueryLaptopBattery = 'SELECT * FROM Win32_Battery'

# Choose the query being tested.
$SelectedQuery = $QueryWindows11Workstation

# Show basic OS facts.
Get-CimInstance `
  -ClassName Win32_OperatingSystem |
  Select-Object Caption,Version,BuildNumber,ProductType,OSArchitecture |
  Tee-Object "$ReportPath\client-os-facts.txt"

# Test WQL query.
$Result = Get-CimInstance `
  -Namespace root\CIMv2 `
  -Query $SelectedQuery `
  -ErrorAction SilentlyContinue

if ($Result) {
    Write-Host "WMI filter query MATCHED this client."
    $Result |
      Select-Object * |
      Out-File "$ReportPath\wmi-query-match-result.txt"
}
else {
    Write-Host "WMI filter query DID NOT MATCH this client."
    "No WMI objects returned for query: $SelectedQuery" |
      Out-File "$ReportPath\wmi-query-no-match-result.txt"
}

# Save tested query.
$SelectedQuery |
  Out-File "$ReportPath\wmi-query-tested.txt"
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_GPMC_Create_Filter_Skeleton
```powershell
# Native creation and attachment of WMI filters should be done in GPMC for this workbook.
# Built-in GroupPolicy PowerShell cmdlets do not provide the same clean workflow for WMI filters
# that they provide for GPO creation, links, permissions, reports, and backups.

# Run this command on the management host:
gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Select WMI Filters.
# 6. Right-click WMI Filters.
# 7. Select New.
# 8. Name:
#      WMI-Windows-11-Workstations
# 9. Description:
#      Applies only to Windows 11 workstation clients.
# 10. Click Add.
# 11. Namespace:
#      root\CIMv2
# 12. Query:
#      SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1 AND BuildNumber >= "22000"
# 13. Save the filter.
# 14. Select the target GPO:
#      CORP-Workstation-Baseline
# 15. Open the Scope tab.
# 16. Under WMI Filtering, select:
#      WMI-Windows-11-Workstations
# 17. Confirm the warning prompt if shown.
# 18. Close and reopen GPMC or refresh the console.
# 19. Confirm the GPO Scope tab shows the selected WMI filter.
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_AD_WMI_Filter_Inventory_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: inventory WMI filter objects stored in Active Directory.

Import-Module ActiveDirectory

$Domain = Get-ADDomain
$DomainDN = $Domain.DistinguishedName

$WmiFilterSearchBase = "CN=SOM,CN=WMIPolicy,CN=System,$DomainDN"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm the WMI policy container exists.
Get-ADObject -Identity $WmiFilterSearchBase

# Inventory WMI filters.
$Filters = Get-ADObject `
  -LDAPFilter "(objectClass=msWMI-Som)" `
  -SearchBase $WmiFilterSearchBase `
  -Properties msWMI-Name,msWMI-ID,msWMI-Author,msWMI-Parm1,msWMI-Parm2,whenCreated,whenChanged

$Filters |
  Select-Object Name,msWMI-Name,msWMI-ID,msWMI-Author,msWMI-Parm1,msWMI-Parm2,whenCreated,whenChanged |
  Format-List

$Filters |
  Select-Object Name,msWMI-Name,msWMI-ID,msWMI-Author,msWMI-Parm1,msWMI-Parm2,whenCreated,whenChanged |
  Export-Csv "$ReportPath\wmi-filters-inventory.csv" -NoTypeInformation

# Search for a specific WMI filter by display name.
$TargetFilterName = "WMI-Windows-11-Workstations"

$Filters |
  Where-Object { $_.'msWMI-Name' -eq $TargetFilterName } |
  Select-Object Name,DistinguishedName,msWMI-Name,msWMI-ID,msWMI-Parm1,msWMI-Parm2 |
  Format-List
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_GPO_Report_Validation_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: export GPO reports and inspect whether the target GPO references a WMI filter.

Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Baseline"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

$HtmlReport = "$ReportPath\$GpoName-after-wmi-filter.html"
$XmlReport = "$ReportPath\$GpoName-after-wmi-filter.xml"

# Export reports.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path $HtmlReport

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path $XmlReport

# Load XML for basic review.
[xml]$GpoXml = Get-Content $XmlReport

# Print top-level XML nodes to help locate WMI filter data in the report.
$GpoXml.GPO |
  Format-List *

# Quick string search for WMI filter name or query fragments.
Select-String `
  -Path $XmlReport `
  -Pattern "WMI","Win32_OperatingSystem","BuildNumber","ProductType" |
  Out-File "$ReportPath\$GpoName-wmi-filter-report-search.txt"

Write-Host "GPO reports exported:"
Write-Host $HtmlReport
Write-Host $XmlReport
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Client_RSOP_Validation_Skeleton
```powershell
# Run on the matching and nonmatching test clients.
# Purpose: prove the GPO applies only when the WMI query matches.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"
$GpoName = "CORP-Workstation-Baseline"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Capture OS facts that should explain WMI filter behavior.
Get-CimInstance Win32_OperatingSystem |
  Select-Object Caption,Version,BuildNumber,ProductType,OSArchitecture |
  Tee-Object "$ReportPath\client-os-facts-for-wmi-filter.txt"

# Force policy refresh.
gpupdate /force

# Show computer-side applied and denied GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-wmi.txt"

# Show user-side applied and denied GPOs.
gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-wmi.txt"

# Export HTML RSOP.
gpresult /h "$ReportPath\gpresult-wmi-filtering.html"

# Optional PowerShell RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-wmi-filtering.html" `
  -ErrorAction SilentlyContinue

# Search text result for target GPO name.
Select-String `
  -Path "$ReportPath\gpresult-computer-wmi.txt" `
  -Pattern $GpoName `
  -ErrorAction SilentlyContinue

# Review recent Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 120 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-operational-events-wmi.txt"
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final GPO, WMI filter, inheritance, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainDN = $Domain.DistinguishedName

$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"
$GpoName = "CORP-Workstation-Baseline"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

$WmiFilterSearchBase = "CN=SOM,CN=WMIPolicy,CN=System,$DomainDN"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\inheritance-after-wmi-filtering.txt"

# Capture final GPO permissions.
Get-GPPermission `
  -Name $GpoName `
  -All |
  Out-File "$ReportPath\$GpoName-permissions-after-wmi.txt"

# Export final GPO reports.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-after-wmi-filtering.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-after-wmi-filtering.xml"

# Capture WMI filter inventory.
Get-ADObject `
  -LDAPFilter "(objectClass=msWMI-Som)" `
  -SearchBase $WmiFilterSearchBase `
  -Properties msWMI-Name,msWMI-ID,msWMI-Author,msWMI-Parm1,msWMI-Parm2,whenCreated,whenChanged |
  Select-Object Name,DistinguishedName,msWMI-Name,msWMI-ID,msWMI-Author,msWMI-Parm1,msWMI-Parm2,whenCreated,whenChanged |
  Export-Csv "$ReportPath\wmi-filters-after.csv" -NoTypeInformation

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

Write-Host "WMI filtering report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<target-gpo-name>"` | Confirms target GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<target-ou-dn>"` | Confirms GPO is linked in expected scope | Target GPO appears in link path |
| `Get-GPPermission -Name "<target-gpo-name>" -All` | Confirms security filtering still permits the target | Expected read and apply permissions exist |
| `Get-CimInstance -Namespace root\CIMv2 -Query '<wmi-query>'` | Tests WMI query directly on a client | Matching client returns object, nonmatching client returns no object |
| `Get-CimInstance Win32_OperatingSystem \| Select Caption,Version,BuildNumber,ProductType,OSArchitecture` | Shows OS properties used by common filters | OS data explains query match or no match |
| `gpmc.msc` | Opens GPMC for native WMI filter creation and attachment | Console opens |
| `Get-ADObject -LDAPFilter "(objectClass=msWMI-Som)" -SearchBase "CN=SOM,CN=WMIPolicy,CN=System,<domain-dn>" -Properties *` | Inventories AD WMI filter objects | Filter object appears |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Html -Path "<report-path>\<target-gpo-name>-wmi.html"` | Exports readable GPO report | HTML report exists and shows WMI filter association |
| `Get-GPOReport -Name "<target-gpo-name>" -ReportType Xml -Path "<report-path>\<target-gpo-name>-wmi.xml"` | Exports machine-readable GPO report | XML report exists |
| `Select-String -Path "<report-path>\<target-gpo-name>-wmi.xml" -Pattern "WMI","Win32_OperatingSystem","ProductType","BuildNumber"` | Searches report for WMI details | WMI filter text appears |
| `gpupdate /force` | Forces policy refresh | Policy refresh completes |
| `gpresult /scope computer /r` | Validates applied and denied computer GPOs | Matching target applies GPO; nonmatching target does not |
| `gpresult /h C:\GPOPrep\Reports\gpresult-wmi-filtering.html` | Creates full client policy report | HTML report exists |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 120` | Reviews client processing events | Events show policy processing status |
| `repadmin /replsummary` | Checks replication if filter or GPO association differs by DC | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks DC and SYSVOL health | Tests pass |

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Rollback
```powershell
# Rollback note:
# Detaching a WMI filter from a GPO is safest through GPMC.
# Built-in GroupPolicy cmdlets do not provide the same clean first-party detach workflow
# that they provide for GPO links and permissions.

# Run this command on the management host:
gpmc.msc

# GUI rollback workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Select the target GPO:
#      CORP-Workstation-Baseline
# 6. Open the Scope tab.
# 7. Under WMI Filtering, change the selected filter to:
#      <none>
# 8. Confirm the prompt if shown.
# 9. Refresh GPMC.
# 10. Export a new GPO report.
# 11. Run gpupdate and gpresult on the test client again.

# PowerShell rollback support: report and validate after GUI detach.

Import-Module GroupPolicy
Import-Module ActiveDirectory

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$GpoName = "CORP-Workstation-Baseline"
$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"

$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture post-rollback reports.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-after-wmi-rollback.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-after-wmi-rollback.xml"

# Confirm link and inheritance remain unchanged.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\inheritance-after-wmi-rollback.txt"

# Backup GPO after rollback.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Validate SYSVOL and replication if behavior is inconsistent.
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
repadmin /replsummary
```

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| WMI filter option not visible | Not using GPMC or insufficient permissions | `gpmc.msc`; `whoami /groups` | Use GPMC with authorized domain admin or delegated GPO admin |
| WMI query works on one client but not another | Different OS, build, hardware, or WMI repository state | `Get-CimInstance Win32_OperatingSystem` | Adjust query or validate client class data |
| Matching client does not apply GPO | Link, security filtering, WMI filter, or GPO status issue | `gpresult /scope computer /r`; `Get-GPInheritance`; `Get-GPPermission` | Fix link scope, permissions, WMI query, or GPO status |
| GPO appears denied by WMI filter | Query returned no objects on client | `Get-CimInstance -Namespace root\CIMv2 -Query '<wmi-query>'` | Correct WQL or attach filter to correct GPO |
| GPO disappears from applied list after adding WMI filter | Filter is too restrictive | Test WQL directly on client | Loosen query or remove WMI filter |
| GPO applies to servers unexpectedly | Query does not exclude servers | `Get-CimInstance Win32_OperatingSystem | Select ProductType` | Add `ProductType = 1` for workstations |
| GPO applies to workstations unexpectedly | Query does not exclude workstations | `Get-CimInstance Win32_OperatingSystem | Select ProductType` | Use `ProductType <> 1` for servers |
| Windows 11 query misses some machines | BuildNumber comparison or WMI value mismatch | `Get-CimInstance Win32_OperatingSystem | Select Caption,BuildNumber` | Validate build logic and simplify query |
| WMI filter slows logon | Expensive query or broken WMI provider | GroupPolicy Operational event log | Simplify query or remove WMI filter |
| WMI filter works but GPO still denied | Security filtering blocks target | `Get-GPPermission -Name "<gpo-name>" -All`; `gpresult /r` | Grant `GpoApply` to intended target |
| WMI filter works but no settings apply | GPO has no settings or wrong policy side | `Get-GPOReport -Name "<gpo-name>"` | Add computer/user setting and validate correct side |
| Report does not show WMI filter | Filter not attached to GPO or report was generated before change | GPMC Scope tab; rerun `Get-GPOReport` | Attach filter and export fresh report |
| WMI filter object not found in AD inventory | Filter not created, wrong domain, or replication delay | `Get-ADObject -LDAPFilter "(objectClass=msWMI-Som)"` | Create filter in correct domain or wait/fix replication |
| GPMC shows stale WMI filter data | Console cache or replication delay | Refresh GPMC; `repadmin /replsummary` | Refresh console or fix replication |
| Client event log shows WMI failure | WMI service, repository, or query issue | `Get-Service Winmgmt`; GroupPolicy Operational log | Repair WMI issue or remove filter |
| Nonmatching client still applies GPO | WMI filter not attached to GPO or client actually matches query | GPMC Scope tab; test WQL locally | Attach correct filter or correct query |
| Rollback does not restore behavior immediately | Client has not refreshed policy | `gpupdate /force`; reboot | Force refresh or reboot client |
| WMI filter used where OU targeting was enough | Overcomplicated design | Review OU and security group scope | Replace WMI filter with OU link or security filtering |

# 05_Configure_WMI_Filtering_For_GPO_Targeting_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines the full GPO suite and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy module |
| `02_Create_And_Link_Baseline_GPO.md` | Creates and links the GPO that can receive WMI filtering |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Confirms inheritance and link path before adding WMI targeting |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Confirms ACL-based scope before WMI targeting |
| `Create_Baseline_OU_Structure.md` | Provides OU structure for cleaner targeting before WMI filters are considered |
| `Create_Baseline_Groups_And_Test_Users.md` | Provides test principals for security filtering and RSOP validation |
| `Join_Windows_Client_To_Domain.md` | Provides client required for WMI query testing, `gpupdate`, and `gpresult` |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms domain policy infrastructure before troubleshooting WMI filters |
| `06_Configure_Computer_Security_Baseline_With_GPO.md` | Next workbook where actual computer security policy can be targeted after scope controls are known |