16_Troubleshoot_Group_Policy_Processing_And_Replication.md
# 16_Troubleshoot_Group_Policy_Processing_And_Replication

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Index
16_Troubleshoot_Group_Policy_Processing_And_Replication.md
16_Troubleshoot_Group_Policy_Processing_And_Replication
16_Troubleshoot_Group_Policy_Processing_And_Replication_Source_Basis
16_Troubleshoot_Group_Policy_Processing_And_Replication_Mental_Model
16_Troubleshoot_Group_Policy_Processing_And_Replication_Planning_Table
16_Troubleshoot_Group_Policy_Processing_And_Replication_Symptom_Map
16_Troubleshoot_Group_Policy_Processing_And_Replication_Configuration_Checklist
16_Troubleshoot_Group_Policy_Processing_And_Replication_Precheck_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_Client_Processing_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_Scope_Filtering_And_Precedence_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_DC_Locator_DNS_And_Time_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_AD_Replication_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_SYSVOL_DFSR_Replication_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_GPO_Version_And_Report_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_Event_Log_Collection_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_Remediation_Workflow_Skeleton
16_Troubleshoot_Group_Policy_Processing_And_Replication_Verification_Commands
16_Troubleshoot_Group_Policy_Processing_And_Replication_Rollback
16_Troubleshoot_Group_Policy_Processing_And_Replication_Failure_Checks
16_Troubleshoot_Group_Policy_Processing_And_Replication_Related_Labs

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | `gpupdate` | Forcing local Group Policy processing |
| Microsoft Learn | `gpresult` | Viewing applied, denied, and effective GPOs |
| Microsoft Learn | Resultant Set of Policy | Validating effective user and computer policy |
| Microsoft Learn | Group Policy Operational event log | Client-side policy processing troubleshooting |
| Microsoft Learn | `Get-GPInheritance` | Validating GPO link order, inheritance, enforcement, and block inheritance |
| Microsoft Learn | `Get-GPPermission` | Validating security filtering and apply permissions |
| Microsoft Learn | `Get-GPOReport` | Comparing intended GPO settings against effective results |
| Microsoft Learn | `repadmin` | Diagnosing Active Directory replication |
| Microsoft Learn | `dcdiag` | Diagnosing domain controller, advertising, DNS, and SYSVOL health |
| Microsoft Learn | DFS Replication tools and logs | Diagnosing SYSVOL replication when DFSR is used |
| Windows Server operational practice | Separate client processing from AD/SYSVOL replication troubleshooting | Avoiding false conclusions when policy does not apply |

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Group Policy processing | Client-side process where Windows evaluates and applies user and computer policy |
| Computer policy | Policy evaluated for the computer account based on computer OU scope |
| User policy | Policy evaluated for the user account based on user OU scope |
| Applied GPO | GPO that passed scope, link, filtering, WMI, and processing checks |
| Denied GPO | GPO visible to the client but blocked by filtering, WMI, disabled state, empty settings, or access |
| GPO link | Site, domain, or OU link that places a GPO in processing scope |
| GPO precedence | Final effective setting is determined by LSDOU, link order, inheritance, enforcement, and conflicts |
| Security filtering | ACL-based gate requiring Read and Apply Group Policy permissions |
| WMI filtering | Client-side WMI query that must match before the GPO applies |
| Loopback processing | Computer-side setting that changes user policy behavior on specific computers |
| AD portion of GPO | GPO container metadata stored in Active Directory |
| SYSVOL portion of GPO | GPO policy files stored under SYSVOL |
| GPO version mismatch | AD metadata and SYSVOL `gpt.ini` are not in expected sync |
| DFSR | Common replication engine for SYSVOL on modern domains |
| DC locator | Client process for finding a domain controller |
| SYSVOL access | Client ability to read `\\domain\SYSVOL` and GPO files |
| Replication latency | Delay before AD and SYSVOL changes reach all DCs |
| First rule | Prove whether the problem is client scope, client processing, AD replication, or SYSVOL replication before changing anything |
| Blunt rule | If the client cannot locate a DC or read SYSVOL, Group Policy troubleshooting starts there, not inside the GPO editor |

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| NetBIOS domain | `CORP` | `<netbios-domain>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Test user | `tuser` | `<test-user>` |
| Computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<computer-ou-dn>` |
| User OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<user-ou-dn>` |
| Problem GPO | `CORP-Workstation-Security-Baseline` | `<problem-gpo-name>` |
| Expected computer GPO | `CORP-Workstation-Security-Baseline` | `<expected-computer-gpo>` |
| Expected user GPO | `CORP-User-Administrative-Templates` | `<expected-user-gpo>` |
| Primary DC | `DC1` | `<primary-dc>` |
| Secondary DC | `DC2` | `<secondary-dc>` |
| Client chosen DC | Output of `nltest /dsgetdc` | `<client-dc>` |
| SYSVOL path | `\\corp.local\SYSVOL\corp.local\Policies` | `<sysvol-path>` |
| Problem category | Client processing, scope, DNS, AD replication, SYSVOL replication, permissions | `<category>` |
| Report root | `C:\GPOPrep\Reports\GPO-Troubleshooting` | `<report-root>` |
| Backup root | `C:\GPOPrep\Backup` | `<backup-root>` |
| Remediation type | Refresh, reboot, fix link, fix filtering, fix DNS, fix replication, restore GPO | `<remediation>` |
| Rollback method | Restore backup, disable new link, revert permission change | `<rollback-plan>` |

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Symptom_Map
| Symptom | Most Likely Area | First Checks |
|---|---|---|
| `gpupdate` cannot complete | DC locator, DNS, network, SYSVOL | `nltest /dsgetdc`, `ipconfig /all`, `Test-Path \\domain\SYSVOL` |
| GPO not listed in `gpresult` at all | Link scope, OU placement, inheritance | `Get-GPInheritance`, AD user/computer DN |
| GPO listed as denied by security filtering | GPO ACL or group membership | `Get-GPPermission`, `whoami /groups`, computer reboot |
| GPO listed as denied by WMI filter | WMI query does not match | Test WMI query with `Get-CimInstance -Query` |
| GPO applies to wrong computers | Link too broad or filtering too broad | `Get-GPInheritance`, `Get-GPPermission` |
| User policy missing | User OU scope or loopback behavior | `gpresult /scope user /r`, user DN, loopback policy |
| Computer policy missing | Computer OU scope or computer filtering | `gpresult /scope computer /r`, computer DN |
| Setting configured but not effective | Precedence conflict or wrong policy side | `gpresult /h`, winning GPO section |
| GPP item missing | Item-level targeting or preference processing error | GPO report, GroupPolicy Operational log |
| Script GPO applies but script does not run | Script path, execution context, timing, or permissions | Script logs, SYSVOL path, Operational log |
| Different clients get different GPO state | Replication, OU scope, group token, WMI, local state | Compare `gpresult /h` and chosen DC |
| GPMC shows GPO but client cannot apply | SYSVOL access or AD/SYSVOL mismatch | `Test-Path \\domain\SYSVOL`, `gpt.ini`, event logs |
| GPO change visible on one DC but not another | AD replication latency or failure | `repadmin /replsummary`, `Get-ADReplicationFailure` |
| GPO files missing from SYSVOL | DFSR or SYSVOL replication failure | DFS Replication log, `dcdiag /test:sysvolcheck` |
| `The system cannot find the path specified` | SYSVOL path, DNS, DFS namespace, replication | `Test-Path`, `Resolve-DnsName`, DC SYSVOL share |
| `Access denied` reading GPO | GPO ACL, SYSVOL ACL, security filtering | `Get-GPPermission`, file ACL checks |
| Remote RSoP fails | Firewall, WMI, RPC, WinRM, permissions | `Test-WSMan`, `Test-NetConnection`, local RSoP |
| User group change not reflected | Stale user token | `whoami /groups`, sign out and sign in |
| Computer group change not reflected | Stale computer token | `gpresult /scope computer /r`, reboot |
| Setting reverts after refresh | Another GPO wins or local preference reprocessing | `gpresult /h`, link order, GPP action |
| GPO restore/import did not take effect | Client not refreshed or wrong target GPO | `gpupdate`, `Get-GPOReport`, `Get-GPInheritance` |

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled,DNSHostName` | Computer object returns |
| 6 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled,UserPrincipalName` | User object returns |
| 7 | Confirm expected GPO exists | Management Host | `Get-GPO -Name "<problem-gpo-name>"` | GPO object returns |
| 8 | Create report root | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\GPO-Troubleshooting` | Report path exists |
| 9 | Confirm DC locator from client | Test Client | `nltest /dsgetdc:<domain-fqdn>` | Client finds a DC |
| 10 | Confirm client DNS config | Test Client | `ipconfig /all` | Domain DNS servers are configured |
| 11 | Confirm time status | Test Client | `w32tm /query /status` | Time status returns |
| 12 | Confirm SYSVOL access from client | Test Client | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 13 | Force policy refresh | Test Client | `gpupdate /force` | Policy refresh completes or returns actionable error |
| 14 | Export quick computer GPResult | Test Client | `gpresult /scope computer /r` | Applied and denied computer GPOs are visible |
| 15 | Export quick user GPResult | Test Client | `gpresult /scope user /r` | Applied and denied user GPOs are visible |
| 16 | Export full GPResult HTML | Test Client | `gpresult /h C:\GPOPrep\Reports\GPO-Troubleshooting\gpresult-full.html` | HTML report exists |
| 17 | Export full GPResult XML | Test Client | `gpresult /x C:\GPOPrep\Reports\GPO-Troubleshooting\gpresult-full.xml` | XML report exists |
| 18 | Review Group Policy Operational events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 200` | Processing events are visible |
| 19 | Confirm computer OU inheritance | Management Host | `Get-GPInheritance -Target "<computer-ou-dn>"` | Expected computer GPO appears |
| 20 | Confirm user OU inheritance | Management Host | `Get-GPInheritance -Target "<user-ou-dn>"` | Expected user GPO appears |
| 21 | Confirm problem GPO permissions | Management Host | `Get-GPPermission -Name "<problem-gpo-name>" -All` | Read and Apply permissions are known |
| 22 | Export problem GPO report | Management Host | `Get-GPOReport -Name "<problem-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\GPO-Troubleshooting\<problem-gpo-name>.html` | Intended setting report exists |
| 23 | Check AD replication summary | Domain Controller | `repadmin /replsummary` | No unexpected failures |
| 24 | Check AD replication failures | Domain Controller | `Get-ADReplicationFailure -Scope Forest` | No unexpected failures |
| 25 | Check DC health | Domain Controller | `dcdiag /test:advertising /test:sysvolcheck /test:netlogons /test:dns` | Tests pass or failures are recorded |
| 26 | Check SYSVOL share on DCs | Domain Controller | `Get-SmbShare -Name SYSVOL,NETLOGON` | SYSVOL and NETLOGON shares exist |
| 27 | Check DFSR service | Domain Controller | `Get-Service DFSR` | DFSR service is running |
| 28 | Review DFS Replication events | Domain Controller | `Get-WinEvent -LogName "DFS Replication" -MaxEvents 100` | DFSR status or errors are visible |
| 29 | Compare GPO file presence across DCs | Domain Controller | `Test-Path "\\<dc>\SYSVOL\<domain-fqdn>\Policies\{<gpo-guid>}\gpt.ini"` | GPO files exist on expected DCs |
| 30 | Compare `gpt.ini` versions | Domain Controller | `Get-Content "\\<dc>\SYSVOL\<domain-fqdn>\Policies\{<gpo-guid>}\gpt.ini"` | Version data is visible |
| 31 | Identify if client uses stale DC | Test Client | `nltest /dsgetdc:<domain-fqdn>` | Chosen DC is known |
| 32 | Refresh client token if group change occurred | Test Client | `klist purge` plus sign out/sign in or reboot | New token is used |
| 33 | Reboot if computer group or startup policy changed | Test Client | `Restart-Computer` | Computer receives new token and startup policy |
| 34 | Validate after remediation | Test Client | `gpupdate /force; gpresult /h C:\GPOPrep\Reports\GPO-Troubleshooting\gpresult-after-fix.html` | Expected GPO behavior is proven |
| 35 | Document root cause | Operator | `Record symptom, failed layer, command evidence, remediation, and final validation` | Troubleshooting result is documented |

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create a troubleshooting workspace and capture domain, GPO, OU, and principal state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName
$DomainNetBIOS = $Domain.NetBIOSName

$TestComputer = "WIN11-01"
$TestUser = "tuser"

$ComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$UserOU = "OU=Users,OU=Corp,$DomainDN"

$ProblemGpo = "CORP-Workstation-Security-Baseline"

$BasePath = "C:\GPOPrep"
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ReportPath = "$BasePath\Reports\GPO-Troubleshooting-$TimeStamp"
$BackupPath = "$BasePath\Backup\GPO-Troubleshooting-$TimeStamp"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Domain identity.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName,PDCEmulator,RIDMaster,InfrastructureMaster |
  Out-File "$ReportPath\domain-info.txt"

# DC inventory.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog,OperationMasterRoles |
  Export-Csv "$ReportPath\domain-controllers.csv" -NoTypeInformation

# Principal placement.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,Enabled,DistinguishedName,MemberOf |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-computer-ad-state.txt"

Get-ADUser `
  -Identity $TestUser `
  -Properties UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Select-Object SamAccountName,UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-user-ad-state.txt"

# OU inheritance.
Get-GPInheritance `
  -Target $ComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance.txt"

Get-GPInheritance `
  -Target $UserOU |
  Out-File "$ReportPath\user-ou-inheritance.txt"

# Problem GPO state.
Get-GPO `
  -Name $ProblemGpo |
  Format-List * |
  Out-File "$ReportPath\problem-gpo-state.txt"

Get-GPPermission `
  -Name $ProblemGpo `
  -All |
  Out-File "$ReportPath\problem-gpo-permissions.txt"

Get-GPOReport `
  -Name $ProblemGpo `
  -ReportType Html `
  -Path "$ReportPath\$ProblemGpo-intended.html"

Get-GPOReport `
  -Name $ProblemGpo `
  -ReportType Xml `
  -Path "$ReportPath\$ProblemGpo-intended.xml"

# Back up the problem GPO before any changes.
Backup-GPO `
  -Name $ProblemGpo `
  -Path $BackupPath `
  -Comment "Troubleshooting backup before any remediation"

Write-Host "Troubleshooting precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Client_Processing_Skeleton
```powershell
# Run on the affected test client.
# Purpose: determine whether the client can process Group Policy and what it actually applied.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-Client"

New-Item -ItemType Directory -Force -Path $ReportPath

# Identity and domain state.
hostname |
  Out-File "$ReportPath\hostname.txt"

whoami /all |
  Out-File "$ReportPath\whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$ReportPath\computer-info.txt"

# DNS and DC locator.
ipconfig /all |
  Out-File "$ReportPath\ipconfig-all.txt"

nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Resolve-DnsName $DomainFqdn |
  Out-File "$ReportPath\resolve-domain.txt" `
  -ErrorAction SilentlyContinue

# Time status.
w32tm /query /status |
  Out-File "$ReportPath\w32tm-status.txt"

# SYSVOL and NETLOGON access.
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-policies-access.txt"

Test-Path "\\$DomainFqdn\NETLOGON" |
  Out-File "$ReportPath\netlogon-access.txt"

# Force policy refresh.
gpupdate /force |
  Tee-Object "$ReportPath\gpupdate-force.txt"

# Export GPResult views.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-r.txt"

gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-r.txt"

gpresult /h "$ReportPath\gpresult-full.html"
gpresult /x "$ReportPath\gpresult-full.xml"
gpresult /z > "$ReportPath\gpresult-verbose-z.txt"

# Search for common denied GPO reasons.
Select-String `
  -Path "$ReportPath\gpresult-verbose-z.txt" `
  -Pattern "Denied","Security Filtering","WMI","Disabled","Inaccessible","Empty","Loopback","Slow Link" |
  Out-File "$ReportPath\gpresult-denied-reason-search.txt"

# Group Policy client service.
Get-Service gpsvc |
  Select-Object Name,Status,StartType |
  Out-File "$ReportPath\gpsvc-service.txt"

# Local Group Policy cache and history indicators.
Get-ChildItem `
  -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Group Policy\History" `
  -Recurse `
  -ErrorAction SilentlyContinue |
  Select-Object Name |
  Out-File "$ReportPath\group-policy-history-registry.txt"

Write-Host "Client processing collection complete."
Write-Host "Report path: $ReportPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Scope_Filtering_And_Precedence_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: determine whether the GPO is actually in scope and allowed to apply.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainDN = $Domain.DistinguishedName

$TestComputer = "WIN11-01"
$TestUser = "tuser"

$ComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$UserOU = "OU=Users,OU=Corp,$DomainDN"

$ProblemGpo = "CORP-Workstation-Security-Baseline"

$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-Scope"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm AD placement.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DistinguishedName,MemberOf |
  Select-Object Name,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-computer-placement-and-groups.txt"

Get-ADUser `
  -Identity $TestUser `
  -Properties DistinguishedName,MemberOf |
  Select-Object SamAccountName,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-user-placement-and-groups.txt"

# Confirm GPO links and precedence.
Get-GPInheritance `
  -Target $ComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance.txt"

Get-GPInheritance `
  -Target $UserOU |
  Out-File "$ReportPath\user-ou-inheritance.txt"

# Confirm GPO status.
Get-GPO `
  -Name $ProblemGpo |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime,UserVersion,ComputerVersion |
  Out-File "$ReportPath\problem-gpo-status.txt"

# Confirm GPO permissions and security filtering.
Get-GPPermission `
  -Name $ProblemGpo `
  -All |
  Out-File "$ReportPath\problem-gpo-permissions.txt"

# Export report and search for WMI filter, disabled settings, and configured sections.
Get-GPOReport `
  -Name $ProblemGpo `
  -ReportType Xml `
  -Path "$ReportPath\$ProblemGpo.xml"

Select-String `
  -Path "$ReportPath\$ProblemGpo.xml" `
  -Pattern "WMI","Filter","SecurityDescriptor","Computer","User","Enabled","Disabled" |
  Out-File "$ReportPath\problem-gpo-report-search.txt"

# Check for GPOs that may override the problem setting.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,ModificationTime |
  Export-Csv "$ReportPath\all-gpo-inventory.csv" -NoTypeInformation

Write-Host "Scope, filtering, and precedence collection complete."
Write-Host "Report path: $ReportPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_DC_Locator_DNS_And_Time_Skeleton
```powershell
# Run on the affected client and management host as needed.
# Purpose: validate DNS, DC locator, secure channel, and time before blaming the GPO.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-DC-Locator"

New-Item -ItemType Directory -Force -Path $ReportPath

# Client DNS configuration.
ipconfig /all |
  Out-File "$ReportPath\ipconfig-all.txt"

# Domain and SRV record resolution.
Resolve-DnsName $DomainFqdn |
  Out-File "$ReportPath\resolve-domain.txt" `
  -ErrorAction SilentlyContinue

Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV |
  Out-File "$ReportPath\resolve-dc-srv-records.txt" `
  -ErrorAction SilentlyContinue

Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV |
  Out-File "$ReportPath\resolve-kerberos-srv-records.txt" `
  -ErrorAction SilentlyContinue

# DC locator.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

nltest /dclist:$DomainFqdn |
  Out-File "$ReportPath\nltest-dclist.txt"

# Secure channel.
nltest /sc_verify:$DomainFqdn |
  Out-File "$ReportPath\nltest-secure-channel.txt"

# Time status.
w32tm /query /status |
  Out-File "$ReportPath\w32tm-status.txt"

w32tm /query /source |
  Out-File "$ReportPath\w32tm-source.txt"

# SYSVOL access against domain namespace.
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\domain-sysvol-policies-check.txt"

# SYSVOL access against each DC returned by nltest needs manual substitution if needed:
# Test-Path "\\DC1\SYSVOL\$DomainFqdn\Policies"
# Test-Path "\\DC2\SYSVOL\$DomainFqdn\Policies"

Write-Host "DNS, DC locator, secure channel, and time checks complete."
Write-Host "Report path: $ReportPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_AD_Replication_Skeleton
```powershell
# Run on a domain controller or management host with RSAT.
# Purpose: diagnose Active Directory replication problems that can affect GPO metadata.

Import-Module ActiveDirectory

$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-AD-Replication"
New-Item -ItemType Directory -Force -Path $ReportPath

# Domain controller inventory.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog,OperationMasterRoles |
  Export-Csv "$ReportPath\domain-controllers.csv" -NoTypeInformation

# Replication summary.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary.txt"

# Detailed replication failures.
repadmin /showrepl * /csv |
  Out-File "$ReportPath\repadmin-showrepl.csv"

# AD replication failures through PowerShell.
Get-ADReplicationFailure -Scope Forest |
  Select-Object Server,FirstFailureTime,FailureCount,LastError,Partner,PartnerGuid |
  Export-Csv "$ReportPath\ad-replication-failures.csv" -NoTypeInformation

# Replication partner metadata.
Get-ADReplicationPartnerMetadata -Target * -Scope Forest |
  Select-Object Server,Partner,Partition,LastReplicationSuccess,LastReplicationAttempt,ConsecutiveReplicationFailures,LastReplicationResult |
  Export-Csv "$ReportPath\ad-replication-partner-metadata.csv" -NoTypeInformation

# Force replication only after recording failures and confirming this is safe.
# repadmin /syncall /AdeP

# Check Directory Service events on the local DC.
Get-WinEvent `
  -LogName "Directory Service" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\directory-service-events.txt"

Write-Host "AD replication diagnostics complete."
Write-Host "Report path: $ReportPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_SYSVOL_DFSR_Replication_Skeleton
```powershell
# Run on domain controllers.
# Purpose: diagnose SYSVOL and DFSR replication problems that can affect GPO files.

Import-Module ActiveDirectory

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot

$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-SYSVOL-DFSR"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm SYSVOL and NETLOGON shares on the local DC.
Get-SmbShare `
  -Name SYSVOL,NETLOGON `
  -ErrorAction SilentlyContinue |
  Select-Object Name,Path,Description |
  Out-File "$ReportPath\smb-shares-sysvol-netlogon.txt"

# Confirm SYSVOL path.
Test-Path "C:\Windows\SYSVOL\domain\Policies" |
  Out-File "$ReportPath\local-sysvol-policies-path-check.txt"

# DFSR service state.
Get-Service DFSR |
  Select-Object Name,Status,StartType |
  Out-File "$ReportPath\dfsr-service.txt"

# DFSR migration state.
dfsrMig /getglobalstate |
  Out-File "$ReportPath\dfsr-migration-global-state.txt"

dfsrMig /getmigrationstate |
  Out-File "$ReportPath\dfsr-migration-state.txt"

# DFS Replication events.
Get-WinEvent `
  -LogName "DFS Replication" `
  -MaxEvents 250 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\dfs-replication-events.txt"

# Basic DC and SYSVOL checks.
dcdiag /test:sysvolcheck /test:advertising /test:netlogons |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-netlogons.txt"

# Compare SYSVOL policy folder presence across DCs.
$Dcs = Get-ADDomainController -Filter * | Select-Object -ExpandProperty HostName

foreach ($Dc in $Dcs) {
    $Path = "\\$Dc\SYSVOL\$DomainFqdn\Policies"
    $Result = [PSCustomObject]@{
        DC = $Dc
        Path = $Path
        Exists = Test-Path $Path
    }

    $Result |
      Export-Csv "$ReportPath\sysvol-policy-path-by-dc.csv" -NoTypeInformation -Append
}

# DFSR backlog example:
# Replace DC1 and DC2 with actual DC names.
# dfsrdiag Backlog /RGName:"Domain System Volume" /RFName:"SYSVOL Share" /SMem:DC1 /RMem:DC2 |
#   Out-File "$ReportPath\dfsr-backlog-dc1-to-dc2.txt"

Write-Host "SYSVOL and DFSR diagnostics complete."
Write-Host "Report path: $ReportPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_GPO_Version_And_Report_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: compare GPO AD metadata, reports, SYSVOL file presence, and gpt.ini state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot

$ProblemGpo = "CORP-Workstation-Security-Baseline"

$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-Version"
New-Item -ItemType Directory -Force -Path $ReportPath

# Get GPO metadata.
$Gpo = Get-GPO -Name $ProblemGpo

$Gpo |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime,UserVersion,ComputerVersion |
  Format-List |
  Out-File "$ReportPath\problem-gpo-metadata.txt"

$GpoGuid = $Gpo.Id.Guid

# Export intended reports.
Get-GPOReport `
  -Guid $Gpo.Id `
  -ReportType Html `
  -Path "$ReportPath\$ProblemGpo-intended.html"

Get-GPOReport `
  -Guid $Gpo.Id `
  -ReportType Xml `
  -Path "$ReportPath\$ProblemGpo-intended.xml"

# Check GPO folder and gpt.ini across all DCs.
$Dcs = Get-ADDomainController -Filter * | Select-Object -ExpandProperty HostName

foreach ($Dc in $Dcs) {
    $GpoPath = "\\$Dc\SYSVOL\$DomainFqdn\Policies\{$GpoGuid}"
    $GptIni = Join-Path $GpoPath "gpt.ini"

    $Row = [PSCustomObject]@{
        DC = $Dc
        GpoPath = $GpoPath
        GpoPathExists = Test-Path $GpoPath
        GptIniExists = Test-Path $GptIni
        GptIniContent = if (Test-Path $GptIni) { (Get-Content $GptIni -Raw) } else { "<missing>" }
    }

    $Row |
      Export-Csv "$ReportPath\gpo-gptini-by-dc.csv" -NoTypeInformation -Append
}

# List SYSVOL contents for the problem GPO from each DC.
foreach ($Dc in $Dcs) {
    $GpoPath = "\\$Dc\SYSVOL\$DomainFqdn\Policies\{$GpoGuid}"

    if (Test-Path $GpoPath) {
        Get-ChildItem `
          -Path $GpoPath `
          -Recurse `
          -ErrorAction SilentlyContinue |
          Select-Object @{Name="DC";Expression={$Dc}},FullName,Length,LastWriteTime |
          Export-Csv "$ReportPath\gpo-files-by-dc.csv" -NoTypeInformation -Append
    }
}

# Search intended GPO report for common key sections.
Select-String `
  -Path "$ReportPath\$ProblemGpo-intended.xml" `
  -Pattern "Registry","Security","Scripts","Drive","Printer","Folder Redirection","WindowsUpdate","Defender","Firewall","WMI" |
  Out-File "$ReportPath\problem-gpo-report-keyword-search.txt"

Write-Host "GPO version and report diagnostics complete."
Write-Host "Problem GPO GUID: {$GpoGuid}"
Write-Host "Report path: $ReportPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Event_Log_Collection_Skeleton
```powershell
# Run on affected client and domain controllers as needed.
# Purpose: collect relevant event logs for Group Policy processing and replication.

$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-Events"
New-Item -ItemType Directory -Force -Path $ReportPath

# Client-side Group Policy processing events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 500 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-events.txt"

# Group Policy warnings and errors.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 1000 `
  -ErrorAction SilentlyContinue |
  Where-Object { $_.LevelDisplayName -in @("Warning","Error") } |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-warnings-errors.txt"

# System log supporting events.
Get-WinEvent `
  -LogName System `
  -MaxEvents 1000 `
  -ErrorAction SilentlyContinue |
  Where-Object {
    $_.ProviderName -like "*GroupPolicy*" -or
    $_.ProviderName -like "*NETLOGON*" -or
    $_.ProviderName -like "*DNS*" -or
    $_.ProviderName -like "*Time*" -or
    $_.ProviderName -like "*DFSR*"
  } |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\system-supporting-events.txt"

# DFS Replication log on DCs.
Get-WinEvent `
  -LogName "DFS Replication" `
  -MaxEvents 500 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\dfs-replication-events.txt"

# Directory Service log on DCs.
Get-WinEvent `
  -LogName "Directory Service" `
  -MaxEvents 500 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\directory-service-events.txt"

# Application log for GPP related events.
Get-WinEvent `
  -LogName Application `
  -MaxEvents 1000 `
  -ErrorAction SilentlyContinue |
  Where-Object {
    $_.ProviderName -like "*Group Policy*" -or
    $_.Message -like "*Group Policy Preferences*" -or
    $_.Message -like "*The user*preferences*"
  } |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\application-gpp-events.txt"

Write-Host "Event collection complete."
Write-Host "Report path: $ReportPath"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Remediation_Workflow_Skeleton
```powershell
# This is a decision workflow.
# Run the relevant section only after evidence points to that root cause.

# 1. If client cannot find a DC:
#    - Fix DNS client settings.
#    - Confirm domain DNS SRV records.
#    - Confirm network path to DCs.
#    - Confirm secure channel.

nltest /dsgetdc:corp.local
nltest /sc_verify:corp.local
ipconfig /all

# 2. If SYSVOL is inaccessible:
#    - Fix DNS, SMB, firewall, DC health, or SYSVOL share state.
#    - Validate both domain namespace and individual DC paths.

Test-Path "\\corp.local\SYSVOL\corp.local\Policies"
Test-Path "\\DC1\SYSVOL\corp.local\Policies"
Test-Path "\\DC2\SYSVOL\corp.local\Policies"

# 3. If GPO is not in scope:
#    - Fix OU placement or GPO link.
#    - Avoid moving production objects until pilot behavior is known.

Get-GPInheritance -Target "OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local"

# Example only:
# New-GPLink -Name "CORP-Workstation-Security-Baseline" -Target "OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local" -LinkEnabled Yes

# 4. If security filtering denies the GPO:
#    - Confirm intended group membership.
#    - Add GpoApply only to the intended group.
#    - Preserve GpoRead for Authenticated Users or Domain Computers where required.

Get-GPPermission -Name "CORP-Workstation-Security-Baseline" -All

# Example only:
# Set-GPPermission -Name "CORP-Workstation-Security-Baseline" -TargetName "GG_GPO_Workstation_Security_Baseline_Apply" -TargetType Group -PermissionLevel GpoApply

# 5. If WMI filter denies the GPO:
#    - Test WMI query locally.
#    - Fix query or detach filter in GPMC.

Get-CimInstance `
  -Namespace root\CIMv2 `
  -Query 'SELECT * FROM Win32_OperatingSystem WHERE ProductType = 1'

# 6. If AD replication fails:
#    - Fix AD replication before expecting consistent GPO metadata.

repadmin /replsummary
Get-ADReplicationFailure -Scope Forest

# Example only after review:
# repadmin /syncall /AdeP

# 7. If SYSVOL or DFSR fails:
#    - Fix DFSR and SYSVOL before expecting consistent GPO files.
#    - Do not manually copy GPO folders between DCs as a first response.

dcdiag /test:sysvolcheck /test:advertising /test:netlogons
Get-WinEvent -LogName "DFS Replication" -MaxEvents 100

# 8. If client token is stale:
#    - User group change requires sign out and sign in.
#    - Computer group change usually requires reboot.

whoami /groups
klist purge

# 9. Final validation:
#    - Refresh policy.
#    - Reboot or sign out if needed.
#    - Export fresh GPResult and RSoP evidence.

gpupdate /force
gpresult /h "C:\GPOPrep\Reports\GPO-Troubleshooting\gpresult-after-remediation.html"
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `gpupdate /force` | Forces policy refresh on client | Policy refresh completes |
| `gpresult /scope computer /r` | Shows applied and denied computer GPOs | Expected computer GPO appears |
| `gpresult /scope user /r` | Shows applied and denied user GPOs | Expected user GPO appears |
| `gpresult /h C:\GPOPrep\Reports\GPO-Troubleshooting\gpresult.html` | Exports full effective policy report | HTML report exists |
| `gpresult /x C:\GPOPrep\Reports\GPO-Troubleshooting\gpresult.xml` | Exports XML effective policy report | XML report exists |
| `Get-GPResultantSetOfPolicy -ReportType Html -Path <path>` | Exports RSoP report | HTML report exists |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews client-side policy processing | Events show success or actionable errors |
| `nltest /dsgetdc:<domain-fqdn>` | Confirms DC locator | DC details return |
| `nltest /sc_verify:<domain-fqdn>` | Confirms secure channel | Secure channel verifies |
| `ipconfig /all` | Confirms DNS client configuration | Domain DNS servers are used |
| `Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV` | Confirms DC SRV records | DC SRV records return |
| `w32tm /query /status` | Confirms time status | Time data returns |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms domain SYSVOL access | Returns True |
| `Test-Path "\\<dc>\SYSVOL\<domain-fqdn>\Policies"` | Confirms specific DC SYSVOL access | Returns True |
| `Get-GPInheritance -Target "<ou-dn>"` | Confirms link, order, enforcement, and inheritance | Expected GPO appears |
| `Get-GPPermission -Name "<gpo-name>" -All` | Confirms security filtering and delegation | Expected apply/read permissions appear |
| `Get-GPO -Name "<gpo-name>"` | Confirms GPO metadata and status | GPO object returns |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path <path>` | Confirms intended GPO configuration | Report exists |
| `repadmin /replsummary` | Checks AD replication summary | No unexpected failures |
| `repadmin /showrepl * /csv` | Captures detailed AD replication state | CSV output returns |
| `Get-ADReplicationFailure -Scope Forest` | Lists AD replication failures | No failures or known failures only |
| `dcdiag /test:advertising /test:sysvolcheck /test:netlogons /test:dns` | Checks DC advertising, SYSVOL, NETLOGON, and DNS | Tests pass |
| `Get-SmbShare -Name SYSVOL,NETLOGON` | Confirms DC shares exist | SYSVOL and NETLOGON shares return |
| `Get-Service DFSR` | Confirms DFSR service state | DFSR running on DCs using DFSR SYSVOL |
| `dfsrMig /getglobalstate` | Confirms SYSVOL migration state | State is known |
| `Get-WinEvent -LogName "DFS Replication" -MaxEvents 100` | Reviews DFSR replication events | No unexpected critical errors |
| `Get-Content "\\<dc>\SYSVOL\<domain>\Policies\{<gpo-guid>}\gpt.ini"` | Reviews GPO file version on a DC | `Version=` line appears |
| `whoami /groups` | Confirms user group token | Expected groups appear |
| `klist purge` | Clears user Kerberos tickets | Tickets purge successfully |
| `Restart-Computer` | Refreshes computer token and startup policy | Computer restarts |

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Rollback
```powershell
# Troubleshooting should be evidence-first.
# Rollback applies only if you changed links, permissions, filters, or restored/imported GPOs during remediation.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$ProblemGpo = "CORP-Workstation-Security-Baseline"
$BackupPath = "C:\GPOPrep\Backup\GPO-Troubleshooting-Backup"
$ReportPath = "C:\GPOPrep\Reports\GPO-Troubleshooting-Rollback"

New-Item -ItemType Directory -Force -Path $ReportPath

# Capture current state before rollback.
Get-GPOReport `
  -Name $ProblemGpo `
  -ReportType Html `
  -Path "$ReportPath\$ProblemGpo-before-rollback.html" `
  -ErrorAction SilentlyContinue

Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\target-ou-inheritance-before-rollback.txt"

Get-GPPermission `
  -Name $ProblemGpo `
  -All |
  Out-File "$ReportPath\$ProblemGpo-permissions-before-rollback.txt" `
  -ErrorAction SilentlyContinue

# Rollback option 1:
# Disable a test link created during troubleshooting.
# Set-GPLink `
#   -Name $ProblemGpo `
#   -Target $TargetOU `
#   -LinkEnabled No

# Rollback option 2:
# Revert enforcement if it was enabled during troubleshooting.
# Set-GPLink `
#   -Name $ProblemGpo `
#   -Target $TargetOU `
#   -Enforced No

# Rollback option 3:
# Restore original GPO permissions manually from recorded report.
# Example only:
# Set-GPPermission -Name $ProblemGpo -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoApply

# Rollback option 4:
# Restore GPO from backup if troubleshooting changed settings.
# Confirm backup path before running.
# Restore-GPO `
#   -Name $ProblemGpo `
#   -Path $BackupPath

# Capture state after rollback.
Get-GPOReport `
  -Name $ProblemGpo `
  -ReportType Html `
  -Path "$ReportPath\$ProblemGpo-after-rollback.html" `
  -ErrorAction SilentlyContinue

Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\target-ou-inheritance-after-rollback.txt"

Write-Host "Rollback workflow complete."
Write-Host "Run gpupdate /force and gpresult /h on the affected client to validate rollback."
```

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `gpupdate` fails with DC error | DNS, network, DC locator, or secure channel issue | `nltest /dsgetdc:<domain>`; `nltest /sc_verify:<domain>` | Fix DNS, network, or domain secure channel |
| `gpupdate` fails with SYSVOL error | Client cannot access policy files | `Test-Path "\\<domain>\SYSVOL\<domain>\Policies"` | Fix SYSVOL, SMB, DFSR, DNS, or DC health |
| GPO missing from `gpresult` | GPO not linked to user or computer scope | `Get-GPInheritance -Target "<ou-dn>"` | Link GPO to correct OU or move object |
| Computer GPO missing | Computer object outside linked OU | `Get-ADComputer -Identity "<computer>" -Properties DistinguishedName` | Move computer or link GPO correctly |
| User GPO missing | User object outside linked OU | `Get-ADUser -Identity "<user>" -Properties DistinguishedName` | Move user or link GPO correctly |
| GPO denied by security | Missing Apply permission or group token stale | `Get-GPPermission`; `whoami /groups` | Fix ACL, sign out user, or reboot computer |
| GPO denied by WMI | WMI filter query returns no objects | `Get-CimInstance -Namespace root\CIMv2 -Query '<query>'` | Correct WMI query or detach filter |
| GPO denied because empty | No settings configured on relevant side | `Get-GPOReport` | Add computer or user setting as appropriate |
| GPO applies but setting absent | Wrong side, conflicting GPO, or setting requires reboot/logoff | `gpresult /h` | Fix policy side, precedence, or refresh session |
| GPP item missing | Item-level targeting failed or preference error | GPO report and Operational log | Correct targeting or item configuration |
| Script does not run | Bad SYSVOL path, execution context, or script error | Script logs and Operational log | Fix path, permissions, and script logging |
| Client picks wrong DC | Site/subnet mapping or DNS issue | `nltest /dsgetdc:<domain>` | Fix AD Sites and Services subnet or DNS |
| GPO differs by DC | AD or SYSVOL replication failure | `repadmin /replsummary`; gpt.ini compare | Fix replication before more GPO changes |
| GPO folder missing from one DC | DFSR/SYSVOL replication issue | `Test-Path "\\<dc>\SYSVOL\<domain>\Policies\{guid}"` | Fix DFSR or DC SYSVOL health |
| SYSVOL share missing | DC not advertising SYSVOL/NETLOGON | `Get-SmbShare SYSVOL,NETLOGON`; `dcdiag /test:sysvolcheck` | Fix SYSVOL replication and DC health |
| AD replication failing | Network, DNS, permissions, or DC health issue | `Get-ADReplicationFailure -Scope Forest` | Resolve AD replication root cause |
| DFSR has errors | DFSR backlog, journal wrap, service issue, or replication conflict | DFS Replication event log | Resolve DFSR issue using controlled recovery process |
| GPO report differs from GPResult | Intended config differs from effective config due precedence or filtering | Compare `Get-GPOReport` and `gpresult /h` | Find winning GPO and adjust precedence |
| User group update not reflected | Stale user Kerberos token | `whoami /groups` | Sign out and sign back in |
| Computer group update not reflected | Stale computer token | `gpresult /scope computer /r` | Reboot computer |
| Remote RSoP fails | Firewall, WMI, RPC, WinRM, or permission issue | `Test-WSMan`; `Test-NetConnection <client> -Port 135` | Run locally or enable required remote management |
| Slow GPO processing | WMI filter, scripts, folder redirection, network delay, or DC issues | Operational log timing | Remove slow filters/scripts or fix network/DC issue |
| Loopback changes user policy | Loopback enabled on computer OU | `gpresult /h` | Review loopback processing mode |
| Block inheritance hides parent GPO | OU has inheritance blocked | `Get-GPInheritance` | Remove block or redesign GPO placement |
| Enforced GPO overrides child | Parent link is enforced | `Get-GPInheritance` | Remove enforcement if not intended |
| Troubleshooting change made problem worse | No backup or rollback path used | GPO report and backup inventory | Restore backup or disable test link |

# 16_Troubleshoot_Group_Policy_Processing_And_Replication_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Creates baseline GPOs that may fail to apply |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Explains precedence, enforcement, and block inheritance failures |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Explains permission-based GPO denial |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Explains WMI filter denial and slow processing |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Produces computer-side policy validated in this workbook |
| `07_Configure_User_Administrative_Template_Settings.md` | Produces user-side policy validated in this workbook |
| `08_Configure_Group_Policy_Preferences.md` | Produces GPP items that may fail through item-level targeting or preference errors |
| `09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md` | Produces script processing issues diagnosed through logs and SYSVOL |
| `10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md` | Produces user resource policy that may fail due paths, permissions, or GPP |
| `11_Configure_Security_Baseline_GPO_Settings.md` | Produces high-impact settings where precedence and RSOP matter |
| `12_Configure_Windows_Update_And_Defender_GPO_Settings.md` | Produces update and Defender policies that require client event validation |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md` | Produces firewall and remote management settings that can affect remote troubleshooting |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Provides reports and backups needed before remediation |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Primary validation workbook before deep troubleshooting |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms domain controller, SYSVOL, NETLOGON, and locator health |