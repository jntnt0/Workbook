25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays.md
# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Index
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays.md
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Source_Basis
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Mental_Model
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Planning_Table
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Symptom_Map
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Configuration_Checklist
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Precheck_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Client_Timing_And_Event_Log_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_CSE_Delay_Analysis_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_WMI_Filter_Testing_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Logon_Script_And_GPP_Delay_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Network_DC_And_SYSVOL_Delay_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Remediation_Workflow_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Report_And_Backup_Skeleton
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Verification_Commands
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Rollback
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Failure_Checks
25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Related_Labs

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy processing | Client-side policy processing, synchronous foreground processing, background refresh |
| Microsoft Learn | Group Policy Client-Side Extensions | Understanding CSE-driven delays from scripts, preferences, folder redirection, security, and software installation |
| Microsoft Learn | Group Policy Operational event log | Identifying processing duration, extension duration, and failure events |
| Microsoft Learn | `gpresult` and RSoP | Proving applied GPOs, denied GPOs, and winning policy |
| Microsoft Learn | WMI filters | Testing slow or failing WMI filters |
| Microsoft Learn | `Get-GPInheritance` | Validating link order, inheritance, and GPO count in scope |
| Microsoft Learn | `Get-GPPermission` | Validating security filtering delays and denied GPO behavior |
| Microsoft Learn | `repadmin` and `dcdiag` | Validating AD, SYSVOL, and DC health |
| Windows Server operational practice | Reduce GPO count, reduce synchronous work, and remove slow client-side extensions from broad scopes | Practical slow-logon remediation |

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Slow logon | User sign-in is delayed while Windows processes policy, scripts, network paths, printers, folders, or other extensions |
| CSE | Client-Side Extension that processes a category of Group Policy settings |
| Foreground processing | Policy processing during startup or sign-in |
| Background processing | Periodic policy refresh after the session is already active |
| Synchronous processing | Windows waits for a policy phase to complete before continuing startup or logon |
| Asynchronous processing | Windows continues startup or logon while policy finishes in background |
| Always wait for network | Policy that forces network availability before logon and can increase startup/logon time |
| WMI filter | Query evaluated by the client to decide whether a GPO applies |
| Slow WMI filter | WMI query that is expensive, broken, or slow on the client |
| GPP | Group Policy Preferences, often used for drive maps, printers, files, registry, scheduled tasks, and shortcuts |
| Item-level targeting | GPP targeting logic that can add delay when complex or network-dependent |
| Folder Redirection | User CSE that can heavily affect first logon and profile experience |
| Drive maps | GPP setting that can delay logon if file servers or DFS paths are slow |
| Printer deployment | GPP or policy setting that can delay logon if print servers, drivers, or permissions are slow |
| Scripts | Startup, shutdown, logon, and logoff scripts that can block or slow sessions |
| Software Installation CSE | Computer or user policy that can dramatically slow startup or logon |
| DC locator delay | Client delay while finding a domain controller |
| SYSVOL delay | Client delay while reading GPO files from SYSVOL |
| First rule | Prove which phase is slow before changing random GPOs |
| Blunt rule | If the Group Policy Operational log names a slow extension, start there, not with link order theory |

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Affected client | `WIN11-01` | `<affected-client>` |
| Affected user | `tuser` | `<affected-user>` |
| Affected computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<computer-ou-dn>` |
| Affected user OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<user-ou-dn>` |
| Slow phase | Startup, logon, background refresh, shutdown, logoff | `<slow-phase>` |
| Suspect GPO | `CORP-User-Preferences` | `<suspect-gpo>` |
| Suspect CSE | Scripts, Folder Redirection, Drive Maps, Printers, Security, Software Installation, WMI | `<suspect-cse>` |
| Suspect WMI filter | `Windows 11 Workstations Only` | `<suspect-wmi-filter>` |
| Baseline logon time | `45 seconds` | `<baseline-time>` |
| Current logon time | `4 minutes` | `<current-time>` |
| DC used by client | Output of `nltest /dsgetdc` | `<client-dc>` |
| SYSVOL path | `\\corp.local\SYSVOL\corp.local\Policies` | `<sysvol-path>` |
| Report path | `C:\GPOPrep\Reports\GPO-SlowLogon` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup\GPO-SlowLogon` | `<backup-path>` |
| Remediation type | Disable test link, remove WMI filter, simplify GPP, split GPO, fix network path | `<remediation-type>` |
| Rollback method | Restore backup, re-enable link, reattach WMI filter, restore setting | `<rollback-plan>` |

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Symptom_Map
| Symptom | Likely Area | First Checks |
|---|---|---|
| Long delay at `Applying Group Policy` | Foreground computer policy | GroupPolicy Operational log, `gpresult /scope computer /r` |
| Long delay after entering password | User policy, scripts, GPP, folder redirection | GroupPolicy Operational log, `gpresult /scope user /r` |
| First logon is slow but later logons are normal | Profile creation, folder redirection, printers, drive maps, first-time GPP | Folder redirection logs, GPP events, profile events |
| Every logon is slow | Scripts, network paths, printer deployment, WMI filters, synchronous policy | CSE duration events |
| Only one OU is slow | GPO link scope or inheritance | `Get-GPInheritance -Target <ou>` |
| Only one group of users is slow | Security filtering, item-level targeting, user GPP | `gpresult /scope user /r`, GPP report |
| Only one subnet/site is slow | DC locator, site mapping, network path, DFS referral | `nltest /dsgetdc`, `nltest /dsgetsite`, SYSVOL path timing |
| Slow before network is ready | `Always wait for the network` or network stack delay | GPO report and Operational log |
| Slow WMI filtering | Expensive or broken WMI query | Test WMI query locally with timing |
| Slow drive maps | File server, DFS, item-level targeting, reconnect behavior | `Test-Path`, GPP event logs |
| Slow printer mapping | Print server, drivers, point-and-print policy, unavailable printer | Printer events and GPP report |
| Slow folder redirection | File server, permissions, offline files, large data move | Folder redirection event logs and path test |
| Slow scripts | Script path, network share, PowerShell execution, timeout, blocking command | Script logs, SYSVOL access, Operational log |
| Slow software installation | MSI deployment, network path, install repair, app detection | Application log and GroupPolicy Operational log |
| Slow Security CSE | Large security baseline, restricted groups, user rights, audit policy | Operational log and GPO report |
| `gpupdate /force` takes very long | Same CSE issue outside logon | Time `gpupdate` and inspect CSE events |
| `gpupdate` hangs on user policy | User GPO, script, GPP, folder redirection | `gpresult /scope user /r` and user CSE events |
| GPO processing differs by DC | AD or SYSVOL replication inconsistency | `repadmin /replsummary`, `dcdiag /test:sysvolcheck` |
| User sees black screen after logon | Shell startup, scripts, profile, drive/printer mapping | User Profile Service and GroupPolicy logs |
| Slow only after adding a GPO | New GPO contains slow setting, WMI filter, or network dependency | Disable new link in pilot and retest |

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Create report folder | Management Host / Client | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\GPO-SlowLogon` | Report path exists |
| 6 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup\GPO-SlowLogon` | Backup path exists |
| 7 | Back up current GPOs | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup\GPO-SlowLogon` | Pre-troubleshooting backup exists |
| 8 | Confirm affected computer object | Management Host | `Get-ADComputer -Identity "<affected-client>" -Properties DistinguishedName,MemberOf,DNSHostName` | Computer state is known |
| 9 | Confirm affected user object | Management Host | `Get-ADUser -Identity "<affected-user>" -Properties DistinguishedName,MemberOf` | User state is known |
| 10 | Capture computer OU inheritance | Management Host | `Get-GPInheritance -Target "<computer-ou-dn>"` | Computer GPO path is recorded |
| 11 | Capture user OU inheritance | Management Host | `Get-GPInheritance -Target "<user-ou-dn>"` | User GPO path is recorded |
| 12 | Confirm client DC locator | Affected Client | `nltest /dsgetdc:<domain-fqdn>` | Client finds a DC |
| 13 | Confirm client AD site | Affected Client | `nltest /dsgetsite` | Client site is known |
| 14 | Confirm SYSVOL access | Affected Client | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 15 | Capture baseline GPResult | Affected Client | `gpresult /h C:\GPOPrep\Reports\GPO-SlowLogon\gpresult-before.html` | GPResult report exists |
| 16 | Capture quick applied GPOs | Affected Client | `gpresult /scope computer /r; gpresult /scope user /r` | Applied and denied GPOs are recorded |
| 17 | Force policy update and time it | Affected Client | `Measure-Command { gpupdate /force }` | Processing duration is recorded |
| 18 | Review GroupPolicy Operational log | Affected Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 300` | Processing events are visible |
| 19 | Identify slow CSE events | Affected Client | `Search GroupPolicy Operational events for extension names and duration` | Slow extension is identified |
| 20 | Export System events | Affected Client | `Get-WinEvent -LogName System -MaxEvents 300` | Network, DNS, time, profile, and service events are visible |
| 21 | Export Application events | Affected Client | `Get-WinEvent -LogName Application -MaxEvents 300` | Script, GPP, MSI, and profile events are visible |
| 22 | Test WMI filters locally | Affected Client | `Measure-Command { Get-CimInstance -Query "<wmi-query>" }` | Query time and result are known |
| 23 | Export all GPO reports in scope | Management Host | `Get-GPOReport -Guid <guid> -ReportType Html -Path <path>` | Reports for applied GPOs exist |
| 24 | Search reports for slow settings | Management Host | `Select-String -Path *.xml -Pattern "Scripts","Drive","Printer","Folder Redirection","Software Installation","WMI"` | Suspect settings are found |
| 25 | Test network paths used by GPOs | Affected Client | `Test-Path "\\server\share"` | Path availability and delay are known |
| 26 | Test DNS for servers used by GPOs | Affected Client | `Resolve-DnsName <server>` | Server names resolve |
| 27 | Check DFS or file server latency if relevant | Affected Client | `Measure-Command { Test-Path "\\domain\dfsroot\share" }` | Path timing is known |
| 28 | Temporarily disable suspect GPO link in pilot only | Management Host | `Set-GPLink -Name "<suspect-gpo>" -Target "<ou-dn>" -LinkEnabled No` | Suspect GPO is removed from pilot scope |
| 29 | Retest logon or gpupdate | Affected Client | `gpupdate /force; sign out/sign in` | Delay impact is measured |
| 30 | Re-enable link if suspect not confirmed | Management Host | `Set-GPLink -Name "<suspect-gpo>" -Target "<ou-dn>" -LinkEnabled Yes` | Original link restored |
| 31 | Apply targeted remediation | Management Host | `Simplify WMI, split GPO, remove slow GPP, fix path, fix script` | Slow component is corrected |
| 32 | Validate final policy application | Affected Client | `gpresult /h C:\GPOPrep\Reports\GPO-SlowLogon\gpresult-after.html` | Final RSOP report exists |
| 33 | Re-measure policy processing | Affected Client | `Measure-Command { gpupdate /force }` | Processing time improves |
| 34 | Document root cause | Operator | `Record slow phase, CSE, GPO, event IDs, timing before/after, remediation, and rollback` | Troubleshooting result is documented |

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: collect domain, OU, GPO, and principal state before changing anything.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$AffectedClient = "WIN11-01"
$AffectedUser = "tuser"

$ComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$UserOU = "OU=Users,OU=Corp,$DomainDN"

$BasePath = "C:\GPOPrep"
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$ReportPath = "$BasePath\Reports\GPO-SlowLogon-$TimeStamp"
$BackupPath = "$BasePath\Backup\GPO-SlowLogon-$TimeStamp"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Domain identity.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName,PDCEmulator |
  Out-File "$ReportPath\domain-info.txt"

# DC inventory.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog |
  Export-Csv "$ReportPath\domain-controllers.csv" -NoTypeInformation

# SYSVOL and DC health.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising.txt"

# Affected objects.
Get-ADComputer `
  -Identity $AffectedClient `
  -Properties DNSHostName,Enabled,DistinguishedName,MemberOf |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\affected-computer-ad-state.txt"

Get-ADUser `
  -Identity $AffectedUser `
  -Properties UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Select-Object SamAccountName,UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\affected-user-ad-state.txt"

# OU inheritance.
Get-GPInheritance `
  -Target $ComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance.txt"

Get-GPInheritance `
  -Target $UserOU |
  Out-File "$ReportPath\user-ou-inheritance.txt"

# GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime,UserVersion,ComputerVersion |
  Export-Csv "$ReportPath\gpo-inventory.csv" -NoTypeInformation

# Backup current GPOs.
Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre slow-logon GPO troubleshooting backup"

Write-Host "Slow logon precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Client_Timing_And_Event_Log_Skeleton
```powershell
# Run on the affected client.
# Purpose: collect timing, GPResult, Group Policy Operational logs, and system context.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-Client"

New-Item -ItemType Directory -Force -Path $ReportPath

# Identity and system state.
hostname |
  Out-File "$ReportPath\hostname.txt"

whoami /all |
  Out-File "$ReportPath\whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$ReportPath\computer-info.txt"

# Network and domain state.
ipconfig /all |
  Out-File "$ReportPath\ipconfig-all.txt"

nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

nltest /dsgetsite |
  Out-File "$ReportPath\nltest-dsgetsite.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

# Time gpupdate.
$GpupdateTime = Measure-Command {
    gpupdate /force | Out-File "$ReportPath\gpupdate-force-output.txt"
}

$GpupdateTime |
  Out-File "$ReportPath\gpupdate-force-duration.txt"

# Export GPResult.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-r.txt"

gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-r.txt"

gpresult /h "$ReportPath\gpresult-slow-logon.html"
gpresult /x "$ReportPath\gpresult-slow-logon.xml"
gpresult /z > "$ReportPath\gpresult-slow-logon-verbose.txt"

# Group Policy Operational log.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 500 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-events.txt"

# Warnings and errors only.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 1000 `
  -ErrorAction SilentlyContinue |
  Where-Object { $_.LevelDisplayName -in @("Warning","Error") } |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-warnings-errors.txt"

# Search for duration and extension clues.
Select-String `
  -Path "$ReportPath\group-policy-operational-events.txt" `
  -Pattern "duration","milliseconds","extension","CSE","completed","failed","timeout","script","WMI","Folder Redirection","Drive Maps","Printers","Software Installation" |
  Out-File "$ReportPath\group-policy-duration-and-cse-search.txt"

# Supporting logs.
Get-WinEvent `
  -LogName System `
  -MaxEvents 500 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Out-File "$ReportPath\system-events.txt"

Get-WinEvent `
  -LogName Application `
  -MaxEvents 500 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Out-File "$ReportPath\application-events.txt"

Write-Host "Client timing and event log collection complete."
Write-Host "gpupdate duration:"
$GpupdateTime
Write-Host "Report path: $ReportPath"
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_CSE_Delay_Analysis_Skeleton
```powershell
# Run on the affected client.
# Purpose: identify which Client-Side Extensions are slow or failing.

$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-CSE"
New-Item -ItemType Directory -Force -Path $ReportPath

# Pull recent Group Policy Operational events.
$Events = Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 1000 `
  -ErrorAction SilentlyContinue

$Events |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Export-Csv "$ReportPath\group-policy-operational-events.csv" -NoTypeInformation

# Search for known CSE names and delay language.
$Patterns = @(
  "Group Policy Registry",
  "Security",
  "Scripts",
  "Folder Redirection",
  "Group Policy Drive Maps",
  "Group Policy Printers",
  "Software Installation",
  "Internet Explorer",
  "Scheduled Tasks",
  "Files",
  "Folders",
  "Registry",
  "Shortcuts",
  "Data Sources",
  "WMI",
  "slow link",
  "synchronous",
  "asynchronous",
  "completed",
  "failed",
  "took",
  "milliseconds",
  "timeout"
)

foreach ($Pattern in $Patterns) {
    $SafePattern = $Pattern.Replace(" ","_").Replace("/","_")
    $Events |
      Where-Object { $_.Message -like "*$Pattern*" } |
      Select-Object TimeCreated,Id,LevelDisplayName,Message |
      Out-File "$ReportPath\search-$SafePattern.txt"
}

# Count recent event IDs to find repeated failures.
$Events |
  Group-Object Id |
  Sort-Object Count -Descending |
  Select-Object Name,Count |
  Out-File "$ReportPath\group-policy-event-id-counts.txt"

# Extract warning/error timeline.
$Events |
  Where-Object { $_.LevelDisplayName -in @("Warning","Error") } |
  Sort-Object TimeCreated |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-warning-error-timeline.txt"

# Export GPResult verbose to correlate CSE with applied GPOs.
gpresult /z > "$ReportPath\gpresult-verbose.txt"

Select-String `
  -Path "$ReportPath\gpresult-verbose.txt" `
  -Pattern "Applied Group Policy Objects","Denied GPOs","Folder Redirection","Scripts","Drive Maps","Printers","Software Installation","WMI","Slow Link" |
  Out-File "$ReportPath\gpresult-cse-search.txt"

Write-Host "CSE delay analysis complete."
Write-Host "Report path: $ReportPath"
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_WMI_Filter_Testing_Skeleton
```powershell
# Run on management host and affected client.
# Purpose: find WMI filters in use and test suspect WMI queries locally.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-WMI"

New-Item -ItemType Directory -Force -Path $ReportPath

# List WMI filter objects from AD.
$WmiSearchBase = "CN=SOM,CN=WMIPolicy,CN=System,$DomainDN"

Get-ADObject `
  -LDAPFilter "(objectClass=msWMI-Som)" `
  -SearchBase $WmiSearchBase `
  -Properties * |
  Select-Object Name,DistinguishedName,msWMI-Name,msWMI-Parm1,msWMI-Parm2,whenChanged |
  Export-Csv "$ReportPath\domain-wmi-filters.csv" -NoTypeInformation

# Export all GPO reports and search for WMI filter references.
$AllGpos = Get-GPO -All

foreach ($Gpo in $AllGpos) {
    $SafeName = ($Gpo.DisplayName -replace '[\\/:*?"<>|]', '_')
    Get-GPOReport `
      -Guid $Gpo.Id `
      -ReportType Xml `
      -Path "$ReportPath\$SafeName.xml"
}

Select-String `
  -Path "$ReportPath\*.xml" `
  -Pattern "WMI","Filter","msWMI","SELECT","Win32_" |
  Out-File "$ReportPath\gpo-wmi-filter-reference-search.txt"

Write-Host "WMI filter inventory complete."
Write-Host "Now test suspect WMI queries on affected client."
```

```powershell
# Run on the affected client.
# Purpose: time a suspect WMI query locally.

$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-WMI-Client"
New-Item -ItemType Directory -Force -Path $ReportPath

# Replace with actual WMI query from the WMI filter.
$WmiQuery = 'SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1'

# Test WMI query with timing.
$Result = $null
$Duration = Measure-Command {
    $Result = Get-CimInstance `
      -Namespace root\CIMV2 `
      -Query $WmiQuery `
      -ErrorAction SilentlyContinue
}

[PSCustomObject]@{
    Query = $WmiQuery
    DurationMilliseconds = [math]::Round($Duration.TotalMilliseconds,2)
    ResultCount = ($Result | Measure-Object).Count
} | Export-Csv "$ReportPath\wmi-query-test-result.csv" -NoTypeInformation

$Result |
  Out-File "$ReportPath\wmi-query-output.txt"

# Test common lightweight alternatives.
$Tests = @(
  'SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1',
  'SELECT * FROM Win32_ComputerSystem WHERE DomainRole < 2',
  'SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.%"'
)

foreach ($Query in $Tests) {
    $Output = $null
    $Time = Measure-Command {
        $Output = Get-CimInstance -Namespace root\CIMV2 -Query $Query -ErrorAction SilentlyContinue
    }

    [PSCustomObject]@{
        Query = $Query
        DurationMilliseconds = [math]::Round($Time.TotalMilliseconds,2)
        ResultCount = ($Output | Measure-Object).Count
    } | Export-Csv "$ReportPath\wmi-query-benchmark.csv" -NoTypeInformation -Append
}

Write-Host "WMI filter testing complete."
Write-Host "Report path: $ReportPath"
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Logon_Script_And_GPP_Delay_Skeleton
```powershell
# Run on management host and affected client.
# Purpose: identify scripts, drive maps, printers, files, shortcuts, scheduled tasks, and item-level targeting that delay logon.

Import-Module GroupPolicy

$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-Scripts-GPP"
New-Item -ItemType Directory -Force -Path $ReportPath

# Export all GPO XML reports for searching.
foreach ($Gpo in Get-GPO -All) {
    $SafeName = ($Gpo.DisplayName -replace '[\\/:*?"<>|]', '_')
    Get-GPOReport `
      -Guid $Gpo.Id `
      -ReportType Xml `
      -Path "$ReportPath\$SafeName.xml"
}

# Search for known slow logon policy areas.
$Patterns = @(
  "Scripts",
  "Logon",
  "Startup",
  "Shutdown",
  "Logoff",
  "Drive Maps",
  "Printers",
  "Folder Redirection",
  "Files",
  "Folders",
  "Shortcuts",
  "Scheduled Tasks",
  "ItemLevelTargeting",
  "Filter",
  "\\\\",
  ".ps1",
  ".bat",
  ".cmd",
  ".vbs",
  ".msi"
)

foreach ($Pattern in $Patterns) {
    $SafePattern = $Pattern.Replace("\","_").Replace(".","_").Replace(" ","_")
    Select-String `
      -Path "$ReportPath\*.xml" `
      -Pattern $Pattern `
      -ErrorAction SilentlyContinue |
      Out-File "$ReportPath\search-$SafePattern.txt"
}

Write-Host "Script and GPP report search complete."
Write-Host "Report path: $ReportPath"
```

```powershell
# Run on affected client.
# Purpose: test network paths that scripts, drive maps, printers, and folder redirection depend on.

$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-PathTests"
New-Item -ItemType Directory -Force -Path $ReportPath

# Replace with paths discovered in GPO reports.
$PathsToTest = @(
  "\\FS1\Shared",
  "\\FS1\Profiles",
  "\\FS1\Home",
  "\\PRINT1\HP-LaserJet-01",
  "\\corp.local\SYSVOL\corp.local\scripts"
)

foreach ($Path in $PathsToTest) {
    $Result = $null
    $Duration = Measure-Command {
        $Result = Test-Path $Path
    }

    [PSCustomObject]@{
        Path = $Path
        Exists = $Result
        DurationMilliseconds = [math]::Round($Duration.TotalMilliseconds,2)
    } | Export-Csv "$ReportPath\network-path-timing.csv" -NoTypeInformation -Append
}

# DNS timing for referenced servers.
$Servers = @("FS1","PRINT1")

foreach ($Server in $Servers) {
    $Result = $null
    $Duration = Measure-Command {
        $Result = Resolve-DnsName $Server -ErrorAction SilentlyContinue
    }

    [PSCustomObject]@{
        Server = $Server
        DurationMilliseconds = [math]::Round($Duration.TotalMilliseconds,2)
        Resolved = [bool]$Result
    } | Export-Csv "$ReportPath\dns-resolution-timing.csv" -NoTypeInformation -Append
}

# Review GPP related events.
Get-WinEvent `
  -LogName Application `
  -MaxEvents 500 `
  -ErrorAction SilentlyContinue |
  Where-Object {
    $_.Message -like "*Group Policy Preferences*" -or
    $_.Message -like "*Drive Maps*" -or
    $_.Message -like "*Printers*" -or
    $_.Message -like "*Folder Redirection*"
  } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Out-File "$ReportPath\application-gpp-related-events.txt"

Write-Host "Script, GPP, and network path timing complete."
Write-Host "Report path: $ReportPath"
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Network_DC_And_SYSVOL_Delay_Skeleton
```powershell
# Run on affected client.
# Purpose: test whether slow logon comes from DC locator, DNS, site mapping, SYSVOL, DFS, or network paths.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-Network"

New-Item -ItemType Directory -Force -Path $ReportPath

# DNS and network state.
ipconfig /all |
  Out-File "$ReportPath\ipconfig-all.txt"

route print |
  Out-File "$ReportPath\route-print.txt"

# DC locator timing.
$DcLocatorTime = Measure-Command {
    nltest /dsgetdc:$DomainFqdn | Out-File "$ReportPath\nltest-dsgetdc.txt"
}

$DcLocatorTime |
  Out-File "$ReportPath\nltest-dsgetdc-duration.txt"

# AD site.
nltest /dsgetsite |
  Out-File "$ReportPath\nltest-dsgetsite.txt"

# Domain SRV DNS.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV |
  Out-File "$ReportPath\domain-dc-srv-records.txt" `
  -ErrorAction SilentlyContinue

# SYSVOL timing through domain namespace.
$SysvolDomainPath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"

$SysvolDomainTime = Measure-Command {
    $SysvolDomainResult = Test-Path $SysvolDomainPath
}

[PSCustomObject]@{
    Path = $SysvolDomainPath
    Exists = $SysvolDomainResult
    DurationMilliseconds = [math]::Round($SysvolDomainTime.TotalMilliseconds,2)
} | Export-Csv "$ReportPath\sysvol-domain-path-timing.csv" -NoTypeInformation

# Test individual DC SYSVOL paths if DC names are known.
$Dcs = @("DC1","DC2")

foreach ($Dc in $Dcs) {
    $Path = "\\$Dc\SYSVOL\$DomainFqdn\Policies"
    $Result = $null
    $Duration = Measure-Command {
        $Result = Test-Path $Path
    }

    [PSCustomObject]@{
        DC = $Dc
        Path = $Path
        Exists = $Result
        DurationMilliseconds = [math]::Round($Duration.TotalMilliseconds,2)
    } | Export-Csv "$ReportPath\sysvol-by-dc-path-timing.csv" -NoTypeInformation -Append
}

# Test NETLOGON path.
$NetlogonPath = "\\$DomainFqdn\NETLOGON"
$NetlogonDuration = Measure-Command {
    $NetlogonResult = Test-Path $NetlogonPath
}

[PSCustomObject]@{
    Path = $NetlogonPath
    Exists = $NetlogonResult
    DurationMilliseconds = [math]::Round($NetlogonDuration.TotalMilliseconds,2)
} | Export-Csv "$ReportPath\netlogon-path-timing.csv" -NoTypeInformation

# Network profile and adapter state.
Get-NetConnectionProfile |
  Out-File "$ReportPath\net-connection-profile.txt"

Get-NetAdapter |
  Select-Object Name,Status,LinkSpeed,MacAddress |
  Out-File "$ReportPath\net-adapter-state.txt"

Write-Host "Network, DC locator, and SYSVOL delay checks complete."
Write-Host "Report path: $ReportPath"
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Remediation_Workflow_Skeleton
```powershell
# This is a decision workflow.
# Apply only the remediation that matches collected evidence.

Import-Module GroupPolicy
Import-Module ActiveDirectory

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

# 1. If WMI filter is slow:
#    - Replace expensive query with simpler query.
#    - Avoid broad wildcard queries.
#    - Avoid querying slow or unstable WMI classes.
#    - Remove WMI filter from pilot GPO and retest.

# GUI:
# GPMC > WMI Filters > edit query
# GPMC > GPO > Scope > WMI Filtering > None

# 2. If GPP drive maps are slow:
#    - Remove reconnect if unnecessary.
#    - Avoid unavailable file servers.
#    - Simplify item-level targeting.
#    - Prefer DFS paths only if DFS is healthy.
#    - Split drive maps into a separate GPO for targeted users.

# 3. If printer deployment is slow:
#    - Remove unavailable printers.
#    - Pre-stage drivers.
#    - Reduce printer item-level targeting complexity.
#    - Use separate printer GPO by site or OU.

# 4. If folder redirection is slow:
#    - Validate file server permissions.
#    - Validate path availability.
#    - Avoid moving large existing content during peak logon.
#    - Test with one pilot user.

# 5. If logon scripts are slow:
#    - Add logging and timing inside scripts.
#    - Avoid blocking network calls.
#    - Move long-running work to scheduled task where possible.
#    - Use PowerShell with timeouts where appropriate.

# 6. If too many GPOs apply:
#    - Consolidate baseline settings.
#    - Remove duplicate or obsolete GPOs.
#    - Reduce broad domain-level links.
#    - Keep high-latency GPP in narrowly scoped GPOs.

# 7. If DC locator or SYSVOL is slow:
#    - Fix DNS client settings.
#    - Fix AD Sites and Services subnet mapping.
#    - Fix DFSR or SYSVOL replication.
#    - Fix remote site network path.

# 8. Safe pilot test:
$SuspectGpo = "CORP-User-Preferences"

# Disable suspect GPO link in pilot OU for testing.
# Set-GPLink `
#   -Name $SuspectGpo `
#   -Target $PilotOU `
#   -LinkEnabled No

# Retest:
# On affected client:
# gpupdate /force
# sign out and sign in
# collect GroupPolicy Operational log

# Re-enable if not root cause:
# Set-GPLink `
#   -Name $SuspectGpo `
#   -Target $PilotOU `
#   -LinkEnabled Yes

# 9. Final proof:
# - Save before and after timing.
# - Save before and after gpresult.
# - Save before and after GroupPolicy Operational events.
# - Document the exact CSE or GPO that caused delay.
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Report_And_Backup_Skeleton
```powershell
# Run on management host after remediation.
# Purpose: capture final reports, backups, and comparison evidence.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$ComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$UserOU = "OU=Users,OU=Corp,$DomainDN"

$SuspectGpo = "CORP-User-Preferences"

$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-Final"
$BackupPath = "C:\GPOPrep\Backup\GPO-SlowLogon-Final"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture inheritance after remediation.
Get-GPInheritance `
  -Target $ComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance-final.txt"

Get-GPInheritance `
  -Target $UserOU |
  Out-File "$ReportPath\user-ou-inheritance-final.txt"

# Capture suspect GPO state if it exists.
if (Get-GPO -Name $SuspectGpo -ErrorAction SilentlyContinue) {
    Get-GPPermission `
      -Name $SuspectGpo `
      -All |
      Out-File "$ReportPath\$SuspectGpo-permissions-final.txt"

    Get-GPOReport `
      -Name $SuspectGpo `
      -ReportType Html `
      -Path "$ReportPath\$SuspectGpo-final.html"

    Get-GPOReport `
      -Name $SuspectGpo `
      -ReportType Xml `
      -Path "$ReportPath\$SuspectGpo-final.xml"

    Backup-GPO `
      -Name $SuspectGpo `
      -Path $BackupPath `
      -Comment "Final backup after slow logon remediation"
}

# Export all GPO reports in affected scope for archive.
$AllGpos = Get-GPO -All

foreach ($Gpo in $AllGpos) {
    $SafeName = ($Gpo.DisplayName -replace '[\\/:*?"<>|]', '_')
    Get-GPOReport `
      -Guid $Gpo.Id `
      -ReportType Xml `
      -Path "$ReportPath\$SafeName.xml" `
      -ErrorAction SilentlyContinue
}

# Search for remaining slow-logon indicators.
Select-String `
  -Path "$ReportPath\*.xml" `
  -Pattern "Scripts","Drive Maps","Printers","Folder Redirection","Software Installation","WMI","ItemLevelTargeting","Always wait for the network" |
  Out-File "$ReportPath\remaining-slow-logon-indicator-search.txt"

# Replication checks.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-slow-logon-remediation.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-slow-logon-remediation.txt"

Write-Host "Slow logon final report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Measure-Command { gpupdate /force }` | Measures policy refresh duration | Duration is acceptable after remediation |
| `gpresult /scope computer /r` | Shows applied and denied computer GPOs | Expected GPOs appear only |
| `gpresult /scope user /r` | Shows applied and denied user GPOs | Expected GPOs appear only |
| `gpresult /h C:\GPOPrep\Reports\GPO-SlowLogon\gpresult.html` | Exports full effective policy report | HTML report exists |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 300` | Reviews Group Policy processing timeline | Slow CSE or errors are visible |
| `Get-WinEvent -LogName System -MaxEvents 300` | Reviews system delays | DNS, network, profile, or service issues are visible |
| `Get-WinEvent -LogName Application -MaxEvents 300` | Reviews app, script, GPP, and MSI events | Relevant events are visible |
| `Get-GPInheritance -Target "<ou-dn>"` | Shows GPO link count and precedence | Only intended GPOs apply |
| `Get-GPPermission -Name "<gpo-name>" -All` | Shows filtering and delegation | Target permissions are correct |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path "<path>"` | Exports intended GPO settings | Report exists |
| `Get-ADComputer -Identity "<computer>" -Properties DistinguishedName,MemberOf` | Confirms computer scope | Computer placement and groups are correct |
| `Get-ADUser -Identity "<user>" -Properties DistinguishedName,MemberOf` | Confirms user scope | User placement and groups are correct |
| `nltest /dsgetdc:<domain-fqdn>` | Confirms DC locator | Client finds a DC |
| `nltest /dsgetsite` | Confirms AD site | Correct site returns |
| `Test-Path "\\<domain>\SYSVOL\<domain>\Policies"` | Confirms SYSVOL access | Returns True quickly |
| `Measure-Command { Test-Path "\\server\share" }` | Measures file share response | Path responds quickly |
| `Resolve-DnsName <server>` | Tests DNS for file or print servers | Server resolves |
| `Measure-Command { Get-CimInstance -Query "<wmi-query>" }` | Measures WMI filter query speed | Query returns quickly |
| `repadmin /replsummary` | Checks AD replication | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks SYSVOL and DC advertising | Tests pass |
| `whoami /groups` | Confirms user token | Expected groups appear |
| `klist purge` | Clears user Kerberos tickets | Tickets are purged |
| `Restart-Computer` | Refreshes computer token and startup policy | Computer restarts |

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: roll back test changes made during slow-logon troubleshooting.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$SuspectGpo = "CORP-User-Preferences"

$ReportPath = "C:\GPOPrep\Reports\GPO-SlowLogon-Rollback"
$BackupPath = "C:\GPOPrep\Backup\GPO-SlowLogon"

New-Item -ItemType Directory -Force -Path $ReportPath

# Capture current state before rollback.
if (Get-GPO -Name $SuspectGpo -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $SuspectGpo `
      -ReportType Html `
      -Path "$ReportPath\$SuspectGpo-before-rollback.html"

    Get-GPPermission `
      -Name $SuspectGpo `
      -All |
      Out-File "$ReportPath\$SuspectGpo-permissions-before-rollback.txt"
}

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-rollback.txt"

# Rollback option 1:
# Re-enable a GPO link disabled during testing.
Set-GPLink `
  -Name $SuspectGpo `
  -Target $PilotOU `
  -LinkEnabled Yes `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Disable a temporary test GPO link.
# Set-GPLink `
#   -Name "TEMP-GPO-SlowLogon-Test" `
#   -Target $PilotOU `
#   -LinkEnabled No

# Rollback option 3:
# Restore a GPO from backup if settings were changed.
# Confirm backup identity before running.
# Restore-GPO `
#   -Name $SuspectGpo `
#   -Path $BackupPath

# Rollback option 4:
# Reattach a WMI filter if it was removed during testing.
# Use GPMC:
# GPO > Scope > WMI Filtering > select original filter

# Rollback option 5:
# Restore original GPP item, script path, drive map, or printer setting from the before report.

# Capture state after rollback.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-rollback.txt"

if (Get-GPO -Name $SuspectGpo -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $SuspectGpo `
      -ReportType Html `
      -Path "$ReportPath\$SuspectGpo-after-rollback.html"
}

Write-Host "Slow-logon troubleshooting rollback workflow complete."
Write-Host "On affected client: run gpupdate /force, sign out/sign in, and validate timing."
```

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `gpupdate /force` is slow | Slow CSE, network path, WMI filter, or DC/SYSVOL issue | `Measure-Command { gpupdate /force }`; Operational log | Identify slow CSE and remediate |
| Logon is slow but `gpupdate` is normal | Logon-only script, profile, printer, drive map, or shell issue | Logon events and profile logs | Focus on user session components |
| Startup is slow before sign-in | Computer policy, startup script, software install, security CSE | Operational log and System log | Remediate computer-side CSE |
| WMI filter takes seconds to evaluate | Expensive WMI query | `Measure-Command { Get-CimInstance -Query "<query>" }` | Simplify or remove WMI filter |
| WMI filter returns no result | Query does not match client | Query test output | Fix query or targeting method |
| Drive maps delay logon | Unavailable share, DFS delay, reconnect behavior, targeting | `Measure-Command { Test-Path "\\server\share" }` | Fix path or simplify GPP |
| Printers delay logon | Unavailable print server, driver install, bad printer path | Printer logs and GPP report | Remove bad printer or pre-stage driver |
| Folder redirection delays first logon | Data move, permissions, offline files, slow file server | Folder redirection events and path test | Fix permissions and stage migration |
| Scripts delay logon | Blocking script logic, slow network call, missing timeout | Script logs and Operational log | Add logging, timeout, or move to scheduled task |
| Software installation delays startup | MSI install, repair, unavailable package path | Application log and GPO report | Remove or fix package deployment |
| Security CSE takes long | Large security baseline or repeated application | Operational log | Split baseline or reduce repeated processing |
| Too many GPOs apply | GPO sprawl and broad linking | `gpresult /r`; `Get-GPInheritance` | Consolidate or narrow GPO scope |
| User policy slow only on RDS/kiosk | Loopback applies extra user GPOs | `gpresult /h` | Tune loopback GPOs and link order |
| Slow only at one site | Wrong AD site, remote DC, DFS referral, WAN path | `nltest /dsgetsite`; `nltest /dsgetdc` | Fix AD Sites and Services subnet |
| SYSVOL access slow | DFSR, DC, SMB, network, or DNS issue | SYSVOL timing tests | Fix SYSVOL/DC/network path |
| GPO differs between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication |
| Source path in GPP is stale | Server renamed, share removed, source lab path remains | GPO report search for `\\` paths | Update or remove stale paths |
| Old group membership affects targeting | User or computer token stale | `whoami /groups`; `gpresult /r` | Sign out or reboot |
| Item-level targeting slow | Complex targeting, LDAP queries, file checks, WMI conditions | GPO report and GPP events | Simplify targeting |
| `Always wait for the network` causes delay | Synchronous network wait policy enabled | GPO report search | Disable unless required |
| Slow link detection changes processing | Client detects slow link and skips/extensions change behavior | Operational log | Fix network or configure policy intentionally |
| Black screen after logon | Shell, profile, scripts, mapped drives, printer CSE | Application, System, User Profile logs | Remove slow user-session dependencies |
| Remediation made behavior worse | GPO changed without backup or pilot isolation | Before/after reports | Restore backup or re-enable original link |
| Rollback does not restore timing | Another GPO or local state still causing delay | `gpresult /h`; event logs | Identify remaining winning GPO or CSE |

# 25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Helps reduce excessive or conflicting GPO processing |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Helps identify denied or filtered GPOs |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Directly related to WMI filter delay diagnosis |
| `08_Configure_Group_Policy_Preferences.md` | GPP items commonly cause slow logon through drives, printers, files, and targeting |
| `09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md` | Scripts are a common slow logon and slow startup cause |
| `10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md` | Folder redirection, drive maps, and printer deployment are major slow logon sources |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Provides backup and report workflow before remediation |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Provides validation method for applied and denied GPOs |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Broader processing and replication troubleshooting foundation |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md` | Ensures GPMC can display settings correctly during review |
| `18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers.md` | Loopback can add user-policy complexity and logon delay |
| `23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs.md` | Provides event collection strategy for endpoint troubleshooting |
| `24_Migrate_And_Compare_GPOs_Across_Domains_Or_Labs.md` | Helps find stale migrated paths and references that slow policy |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms domain infrastructure before deep slow-logon troubleshooting |