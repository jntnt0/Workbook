15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md
# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Index
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Source_Basis
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Mental_Model
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Planning_Table
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Tool_Map
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Configuration_Checklist
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Precheck_Skeleton
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_GPUpdate_Skeleton
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_GPResult_Skeleton
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_RSoP_Skeleton
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Event_Log_Validation_Skeleton
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Remote_Validation_Skeleton
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Report_Collection_Skeleton
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Verification_Commands
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Rollback
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Failure_Checks
15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Related_Labs

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | `gpupdate` | Forcing computer and user Group Policy refresh |
| Microsoft Learn | `gpresult` | Viewing applied and denied GPOs from a client |
| Microsoft Learn | Resultant Set of Policy | Determining effective policy settings |
| Microsoft Learn | `Get-GPResultantSetOfPolicy` | Creating HTML or XML RSoP reports with PowerShell |
| Microsoft Learn | `Invoke-GPUpdate` | Triggering remote Group Policy refresh from a management host |
| Microsoft Learn | Group Policy Management Console | Running Group Policy Results and Group Policy Modeling |
| Microsoft Learn | GroupPolicy PowerShell module | GPO reports, inheritance checks, permissions checks, and RSoP reports |
| Microsoft Learn | Group Policy Operational event log | Reviewing client-side policy processing events |
| Windows Server operational practice | Validate user and computer policy separately | Preventing false assumptions about GPO scope and processing |

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Group Policy refresh | Client pulls policy from domain controllers and SYSVOL |
| Computer policy | Applies to the computer account based on computer OU scope |
| User policy | Applies to the user account based on user OU scope |
| `gpupdate` | Forces policy refresh on the local client |
| `gpupdate /force` | Reapplies all policy settings, not only changed settings |
| `gpupdate /target:computer` | Refreshes computer-side policy only |
| `gpupdate /target:user` | Refreshes user-side policy only |
| `gpupdate /boot` | Reboots if a setting requires restart |
| `gpupdate /logoff` | Logs off if a setting requires a new user session |
| `gpresult /r` | Quick text summary of applied and denied GPOs |
| `gpresult /h` | Full HTML report of effective policy |
| `gpresult /x` | XML report useful for parsing and archiving |
| RSoP | Effective policy result after link order, inheritance, filtering, WMI, loopback, and client processing |
| Applied GPO | GPO that passed scope, link, permission, WMI, and processing checks |
| Denied GPO | GPO in possible scope but denied by reason such as security filtering, WMI, disabled link, or empty settings |
| Winning GPO | GPO that provides the effective setting after precedence rules |
| Group Policy Operational log | Client event log showing policy processing details |
| First rule | Validate computer-side and user-side policy separately |
| Blunt rule | If `gpresult` does not show the GPO, the client did not apply it, no matter what GPMC looks like |

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Test user | `tuser` | `<test-user>` |
| Test user UPN | `tuser@corp.local` | `<test-user-upn>` |
| Management host | `MGMT01` | `<management-host>` |
| Computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<computer-ou-dn>` |
| User OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<user-ou-dn>` |
| Expected computer GPO | `CORP-Workstation-Security-Baseline` | `<expected-computer-gpo>` |
| Expected user GPO | `CORP-User-Administrative-Templates` | `<expected-user-gpo>` |
| Expected GPP GPO | `CORP-User-Preferences` | `<expected-gpp-gpo>` |
| Validation mode | Local client, remote client, or GPMC Results Wizard | `<validation-mode>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Client report path | `C:\GPOPrep\Reports\Client-GPResult` | `<client-report-path>` |
| Management report path | `C:\GPOPrep\Reports\Management-RSoP` | `<management-report-path>` |
| Refresh method | `gpupdate /force` | `<refresh-method>` |
| Reboot required | Depends on setting | `<yes-no>` |
| Logoff required | Depends on setting | `<yes-no>` |
| Validation evidence | `gpresult`, RSoP, event logs, registry, applied behavior | `<evidence>` |

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Tool_Map
| Tool | Runs From | Purpose | Best Use |
|---|---|---|---|
| `gpupdate /force` | Test client | Forces user and computer policy refresh | Quick local refresh |
| `gpupdate /target:computer /force` | Test client | Refreshes computer-side policy only | Computer GPO testing |
| `gpupdate /target:user /force` | Test client | Refreshes user-side policy only | User GPO testing |
| `gpresult /r` | Test client | Quick text view of applied and denied GPOs | Fast validation |
| `gpresult /h <path>` | Test client | Full HTML report | Evidence collection |
| `gpresult /x <path>` | Test client | Full XML report | Search and archive |
| `gpresult /z` | Test client | Verbose policy detail | Deep troubleshooting |
| `rsop.msc` | Test client | Graphical RSoP view | GUI validation |
| `Get-GPResultantSetOfPolicy` | Client or management host | PowerShell-generated RSoP report | Repeatable reporting |
| GPMC Group Policy Results Wizard | Management host | Remote RSoP collection | Admin-driven validation |
| GPMC Group Policy Modeling Wizard | Management host | Simulated policy result | Pre-change design checks |
| `Invoke-GPUpdate` | Management host | Remote policy refresh | Trigger refresh on another computer |
| GroupPolicy Operational log | Test client | Client-side policy processing evidence | Processing failure review |
| `Get-GPInheritance` | Management host | Checks link and inheritance state | Scope validation |
| `Get-GPPermission` | Management host | Checks security filtering and delegation | Denied GPO troubleshooting |
| `Get-GPOReport` | Management host | Exports configured GPO settings | Compare intended vs effective |

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 3 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 6 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled` | Test user returns |
| 7 | Confirm computer OU scope | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under expected OU |
| 8 | Confirm user OU scope | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName` | User DN is under expected OU |
| 9 | Confirm SYSVOL access | Test Client | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Confirm DC locator | Test Client | `nltest /dsgetdc:<domain-fqdn>` | Domain controller is located |
| 11 | Confirm time sync basics | Test Client | `w32tm /query /status` | Time status returns |
| 12 | Create report folder | Test Client | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\Client-GPResult` | Client report path exists |
| 13 | Create management report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\Management-RSoP` | Management report path exists |
| 14 | Confirm expected GPO exists | Management Host | `Get-GPO -Name "<expected-computer-gpo>"` | Expected GPO returns |
| 15 | Confirm computer OU inheritance | Management Host | `Get-GPInheritance -Target "<computer-ou-dn>"` | Expected computer GPO appears |
| 16 | Confirm user OU inheritance | Management Host | `Get-GPInheritance -Target "<user-ou-dn>"` | Expected user GPO appears |
| 17 | Confirm expected GPO permissions | Management Host | `Get-GPPermission -Name "<expected-computer-gpo>" -All` | Target computer has read and apply path |
| 18 | Force all policy refresh | Test Client | `gpupdate /force` | Policy refresh completes |
| 19 | Refresh computer policy only | Test Client | `gpupdate /target:computer /force` | Computer policy refresh completes |
| 20 | Refresh user policy only | Test Client | `gpupdate /target:user /force` | User policy refresh completes |
| 21 | Reboot if required by setting | Test Client | `Restart-Computer` | Computer restarts |
| 22 | Log off if required by user setting | Test Client | Sign out and sign back in | New user session starts |
| 23 | Validate applied computer GPOs | Test Client | `gpresult /scope computer /r` | Expected computer GPO appears under Applied GPOs |
| 24 | Validate denied computer GPOs | Test Client | `gpresult /scope computer /r` | Denied GPOs and reasons are visible |
| 25 | Validate applied user GPOs | Test Client | `gpresult /scope user /r` | Expected user GPO appears under Applied GPOs |
| 26 | Validate denied user GPOs | Test Client | `gpresult /scope user /r` | Denied GPOs and reasons are visible |
| 27 | Export HTML GPResult | Test Client | `gpresult /h C:\GPOPrep\Reports\Client-GPResult\gpresult-full.html` | HTML report exists |
| 28 | Export XML GPResult | Test Client | `gpresult /x C:\GPOPrep\Reports\Client-GPResult\gpresult-full.xml` | XML report exists |
| 29 | Export verbose text GPResult | Test Client | `gpresult /z > C:\GPOPrep\Reports\Client-GPResult\gpresult-verbose.txt` | Verbose text report exists |
| 30 | Open RSoP MMC locally | Test Client | `rsop.msc` | RSoP GUI opens |
| 31 | Generate PowerShell RSoP report | Test Client | `Get-GPResultantSetOfPolicy -ReportType Html -Path C:\GPOPrep\Reports\Client-GPResult\rsop-local.html` | RSoP HTML report exists |
| 32 | Generate remote RSoP report | Management Host | `Get-GPResultantSetOfPolicy -Computer "<test-computer>" -User "<domain>\<test-user>" -ReportType Html -Path C:\GPOPrep\Reports\Management-RSoP\rsop-remote.html` | Remote RSoP report exists |
| 33 | Trigger remote GPUpdate if needed | Management Host | `Invoke-GPUpdate -Computer "<test-computer>" -Force -RandomDelayInMinutes 0` | Remote refresh task is created |
| 34 | Review Group Policy operational events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Processing events are visible |
| 35 | Review System log for GP issues | Test Client | `Get-WinEvent -LogName System -MaxEvents 150` | Related system events are visible |
| 36 | Compare intended GPO report with effective policy | Management Host | `Get-GPOReport -Name "<expected-gpo>" -ReportType Html -Path C:\GPOPrep\Reports\Management-RSoP\<expected-gpo>.html` | Intended config report exists |
| 37 | Validate specific registry policy if applicable | Test Client | `Get-ItemProperty -Path "<policy-registry-path>"` | Expected policy value exists |
| 38 | Validate specific security policy if applicable | Test Client | `secedit /export /cfg C:\GPOPrep\Reports\Client-GPResult\effective-security-policy.inf` | Security policy export exists |
| 39 | Validate audit policy if applicable | Test Client | `auditpol /get /category:*` | Effective audit policy appears |
| 40 | Document result | Operator | `Record applied GPOs, denied GPOs, reports, event logs, reboot/logoff status, and final conclusion` | Validation is documented |

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: confirm the test user, test computer, expected GPOs, and OU scope before client validation.

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

$ExpectedComputerGpo = "CORP-Workstation-Security-Baseline"
$ExpectedUserGpo = "CORP-User-Administrative-Templates"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports\Management-RSoP"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm domain.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName |
  Out-File "$ReportPath\domain-info.txt"

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-path-check.txt"

Test-Path "\\$DomainFqdn\NETLOGON" |
  Out-File "$ReportPath\netlogon-path-check.txt"

# Confirm computer and user placement.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,Enabled,DistinguishedName |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-ad-state.txt"

Get-ADUser `
  -Identity $TestUser `
  -Properties UserPrincipalName,Enabled,DistinguishedName |
  Select-Object SamAccountName,UserPrincipalName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-user-ad-state.txt"

# Confirm OU inheritance.
Get-GPInheritance `
  -Target $ComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance.txt"

Get-GPInheritance `
  -Target $UserOU |
  Out-File "$ReportPath\user-ou-inheritance.txt"

# Confirm expected GPOs exist.
Get-GPO -Name $ExpectedComputerGpo |
  Format-List * |
  Out-File "$ReportPath\expected-computer-gpo-state.txt"

Get-GPO -Name $ExpectedUserGpo |
  Format-List * |
  Out-File "$ReportPath\expected-user-gpo-state.txt"

# Confirm expected GPO permissions.
Get-GPPermission `
  -Name $ExpectedComputerGpo `
  -All |
  Out-File "$ReportPath\expected-computer-gpo-permissions.txt"

Get-GPPermission `
  -Name $ExpectedUserGpo `
  -All |
  Out-File "$ReportPath\expected-user-gpo-permissions.txt"

# Export intended GPO reports.
Get-GPOReport `
  -Name $ExpectedComputerGpo `
  -ReportType Html `
  -Path "$ReportPath\$ExpectedComputerGpo-intended.html"

Get-GPOReport `
  -Name $ExpectedUserGpo `
  -ReportType Html `
  -Path "$ReportPath\$ExpectedUserGpo-intended.html"

Write-Host "Precheck complete."
Write-Host "Report path: $ReportPath"
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_GPUpdate_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: force Group Policy refresh and understand whether reboot or logoff is required.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\Client-GPResult"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm identity and domain join.
hostname |
  Out-File "$ReportPath\client-hostname.txt"

whoami |
  Out-File "$ReportPath\client-whoami.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Out-File "$ReportPath\client-domain-state.txt"

# Confirm DC locator and SYSVOL access.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\client-nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\client-sysvol-check.txt"

# Basic refresh.
gpupdate |
  Tee-Object "$ReportPath\gpupdate-basic.txt"

# Force both user and computer policy.
gpupdate /force |
  Tee-Object "$ReportPath\gpupdate-force.txt"

# Refresh computer policy only.
gpupdate /target:computer /force |
  Tee-Object "$ReportPath\gpupdate-computer-force.txt"

# Refresh user policy only.
gpupdate /target:user /force |
  Tee-Object "$ReportPath\gpupdate-user-force.txt"

# Optional:
# Force refresh and reboot if required by policy.
# gpupdate /force /boot

# Optional:
# Force refresh and logoff if required by policy.
# gpupdate /force /logoff

Write-Host "GPUpdate validation complete."
Write-Host "If settings still do not apply, reboot or sign out/sign in depending on setting type."
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_GPResult_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: collect quick, full, XML, and verbose GPResult evidence.

$ReportPath = "C:\GPOPrep\Reports\Client-GPResult"
New-Item -ItemType Directory -Force -Path $ReportPath

# Quick summary of computer policy.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-r.txt"

# Quick summary of user policy.
gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-r.txt"

# Full HTML report.
gpresult /h "$ReportPath\gpresult-full.html"

# XML report for searching or archival.
gpresult /x "$ReportPath\gpresult-full.xml"

# Verbose text report.
gpresult /z > "$ReportPath\gpresult-verbose-z.txt"

# Search for expected GPO names in collected text.
$ExpectedGpos = @(
  "CORP-Workstation-Security-Baseline",
  "CORP-User-Administrative-Templates",
  "CORP-User-Preferences",
  "CORP-Workstation-Update-Defender",
  "CORP-Workstation-Remote-Admin-Access"
)

foreach ($Gpo in $ExpectedGpos) {
    Select-String `
      -Path "$ReportPath\gpresult-computer-r.txt","$ReportPath\gpresult-user-r.txt","$ReportPath\gpresult-verbose-z.txt" `
      -Pattern $Gpo `
      -ErrorAction SilentlyContinue |
      Out-File "$ReportPath\search-$($Gpo.Replace(' ','_')).txt"
}

# Search for common denial reasons.
Select-String `
  -Path "$ReportPath\gpresult-verbose-z.txt" `
  -Pattern "Denied","Filtering","WMI","Security","Empty","Inaccessible","Disabled","Loopback" |
  Out-File "$ReportPath\gpresult-denial-search.txt"

Write-Host "GPResult reports created at: $ReportPath"
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_RSoP_Skeleton
```powershell
# Run locally on the test client or remotely from a management host.
# Purpose: generate RSoP evidence.

$ReportPath = "C:\GPOPrep\Reports\Client-GPResult"
New-Item -ItemType Directory -Force -Path $ReportPath

# Local RSoP MMC.
# This opens the graphical Resultant Set of Policy snap-in.
rsop.msc

# Local PowerShell RSoP HTML report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-local.html" `
  -ErrorAction SilentlyContinue

# Local PowerShell RSoP XML report.
Get-GPResultantSetOfPolicy `
  -ReportType Xml `
  -Path "$ReportPath\rsop-local.xml" `
  -ErrorAction SilentlyContinue
```

```powershell
# Run on a management host.
# Purpose: generate a remote RSoP report for a specific computer and user.

Import-Module GroupPolicy

$Computer = "WIN11-01"
$User = "CORP\tuser"

$ReportPath = "C:\GPOPrep\Reports\Management-RSoP"
New-Item -ItemType Directory -Force -Path $ReportPath

# Remote RSoP HTML report.
Get-GPResultantSetOfPolicy `
  -Computer $Computer `
  -User $User `
  -ReportType Html `
  -Path "$ReportPath\rsop-$Computer-$($User.Replace('\','_')).html"

# Remote RSoP XML report.
Get-GPResultantSetOfPolicy `
  -Computer $Computer `
  -User $User `
  -ReportType Xml `
  -Path "$ReportPath\rsop-$Computer-$($User.Replace('\','_')).xml"

Write-Host "Remote RSoP reports written to: $ReportPath"
```

```powershell
# GPMC GUI workflow for Group Policy Results.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Right-click:
#      Group Policy Results
# 6. Select:
#      Group Policy Results Wizard
# 7. Choose:
#      Another computer
# 8. Enter:
#      WIN11-01
# 9. Choose:
#      Display policy settings for selected user
# 10. Select:
#      CORP\tuser
# 11. Finish wizard.
# 12. Review:
#      Summary
#      Settings
#      Policy Events
# 13. Right-click the results report.
# 14. Save report as HTML.
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Event_Log_Validation_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: collect Group Policy processing events and common supporting logs.

$ReportPath = "C:\GPOPrep\Reports\Client-GPResult"
New-Item -ItemType Directory -Force -Path $ReportPath

# Group Policy Operational log.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 250 |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-events.txt"

# Group Policy Operational warnings and errors only.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 500 |
  Where-Object { $_.LevelDisplayName -in @("Warning","Error") } |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-warnings-errors.txt"

# System log search for Group Policy, Netlogon, DNS, and time issues.
Get-WinEvent `
  -LogName System `
  -MaxEvents 500 |
  Where-Object {
    $_.ProviderName -like "*GroupPolicy*" -or
    $_.ProviderName -like "*NETLOGON*" -or
    $_.ProviderName -like "*DNS*" -or
    $_.ProviderName -like "*Time*"
  } |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\system-supporting-events.txt"

# Application log search for Group Policy Preferences issues.
Get-WinEvent `
  -LogName Application `
  -MaxEvents 500 |
  Where-Object {
    $_.ProviderName -like "*Group Policy*" -or
    $_.Message -like "*Group Policy Preferences*"
  } |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\application-gpp-events.txt"

# Confirm recent policy processing time.
Get-CimInstance `
  -Namespace root\rsop\computer `
  -ClassName RSOP_ComputerPolicySetting `
  -ErrorAction SilentlyContinue |
  Select-Object -First 20 |
  Out-File "$ReportPath\rsop-computer-wmi-sample.txt"

Get-CimInstance `
  -Namespace root\rsop\user `
  -ClassName RSOP_UserPolicySetting `
  -ErrorAction SilentlyContinue |
  Select-Object -First 20 |
  Out-File "$ReportPath\rsop-user-wmi-sample.txt"

Write-Host "Event log validation complete."
Write-Host "Report path: $ReportPath"
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Remote_Validation_Skeleton
```powershell
# Run on a domain-joined management host.
# Purpose: remotely trigger policy refresh and collect validation reports where possible.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Computer = "WIN11-01"
$User = "CORP\tuser"

$ReportPath = "C:\GPOPrep\Reports\Management-RSoP"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm computer exists and resolves.
Get-ADComputer `
  -Identity $Computer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\$Computer-ad-state.txt"

Resolve-DnsName $Computer |
  Out-File "$ReportPath\$Computer-dns-resolution.txt"

Test-Connection $Computer -Count 2 |
  Out-File "$ReportPath\$Computer-ping.txt"

# Trigger remote Group Policy update.
Invoke-GPUpdate `
  -Computer $Computer `
  -Force `
  -RandomDelayInMinutes 0

# Optional connectivity checks for remote reporting.
Test-WSMan $Computer |
  Out-File "$ReportPath\$Computer-test-wsman.txt" `
  -ErrorAction SilentlyContinue

Test-NetConnection `
  -ComputerName $Computer `
  -Port 135 |
  Out-File "$ReportPath\$Computer-rpc-135-test.txt"

# Generate remote RSoP report.
Get-GPResultantSetOfPolicy `
  -Computer $Computer `
  -User $User `
  -ReportType Html `
  -Path "$ReportPath\rsop-$Computer.html" `
  -ErrorAction SilentlyContinue

Get-GPResultantSetOfPolicy `
  -Computer $Computer `
  -User $User `
  -ReportType Xml `
  -Path "$ReportPath\rsop-$Computer.xml" `
  -ErrorAction SilentlyContinue

Write-Host "Remote validation workflow complete."
Write-Host "Some remote RSoP operations may require firewall, WMI, RPC, and permissions to be configured."
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Report_Collection_Skeleton
```powershell
# Run on the test client.
# Purpose: collect a complete validation evidence bundle after gpupdate, gpresult, and RSoP.

$ReportPath = "C:\GPOPrep\Reports\Client-GPResult"
$BundlePath = "C:\GPOPrep\Reports\Bundles"

New-Item -ItemType Directory -Force -Path $ReportPath,$BundlePath

$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BundleRoot = "$BundlePath\GPValidation-$env:COMPUTERNAME-$TimeStamp"

New-Item -ItemType Directory -Force -Path $BundleRoot

# Identity.
hostname |
  Out-File "$BundleRoot\hostname.txt"

whoami /all |
  Out-File "$BundleRoot\whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$BundleRoot\computer-info.txt"

# Policy refresh.
gpupdate /force |
  Out-File "$BundleRoot\gpupdate-force.txt"

# GPResult reports.
gpresult /scope computer /r |
  Out-File "$BundleRoot\gpresult-computer-r.txt"

gpresult /scope user /r |
  Out-File "$BundleRoot\gpresult-user-r.txt"

gpresult /h "$BundleRoot\gpresult-full.html"
gpresult /x "$BundleRoot\gpresult-full.xml"
gpresult /z > "$BundleRoot\gpresult-verbose-z.txt"

# RSoP reports.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$BundleRoot\rsop-local.html" `
  -ErrorAction SilentlyContinue

Get-GPResultantSetOfPolicy `
  -ReportType Xml `
  -Path "$BundleRoot\rsop-local.xml" `
  -ErrorAction SilentlyContinue

# Logs.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 250 |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$BundleRoot\group-policy-operational-events.txt"

Get-WinEvent `
  -LogName System `
  -MaxEvents 250 |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$BundleRoot\system-events.txt"

# Common effective policy exports.
secedit /export /cfg "$BundleRoot\effective-security-policy.inf" 2>$null

auditpol /get /category:* |
  Out-File "$BundleRoot\auditpol-effective.txt" 2>$null

Get-NetFirewallProfile |
  Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction |
  Out-File "$BundleRoot\firewall-profiles.txt" 2>$null

# Compress bundle.
Compress-Archive `
  -Path "$BundleRoot\*" `
  -DestinationPath "$BundlePath\GPValidation-$env:COMPUTERNAME-$TimeStamp.zip" `
  -Force

Write-Host "Validation bundle created:"
Write-Host "$BundlePath\GPValidation-$env:COMPUTERNAME-$TimeStamp.zip"
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `nltest /dsgetdc:<domain-fqdn>` | Confirms client can locate a domain controller | DC details return |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms SYSVOL access | Returns True |
| `w32tm /query /status` | Confirms time service status | Time status returns |
| `gpupdate /force` | Forces all policy refresh | User and computer policy update completes |
| `gpupdate /target:computer /force` | Refreshes computer policy only | Computer policy update completes |
| `gpupdate /target:user /force` | Refreshes user policy only | User policy update completes |
| `gpresult /scope computer /r` | Shows applied and denied computer GPOs | Expected computer GPOs appear |
| `gpresult /scope user /r` | Shows applied and denied user GPOs | Expected user GPOs appear |
| `gpresult /h C:\GPOPrep\Reports\Client-GPResult\gpresult-full.html` | Exports full HTML effective policy report | HTML report exists |
| `gpresult /x C:\GPOPrep\Reports\Client-GPResult\gpresult-full.xml` | Exports full XML effective policy report | XML report exists |
| `gpresult /z` | Shows verbose effective policy output | Detailed GPO and setting data appears |
| `rsop.msc` | Opens local RSoP GUI | RSoP snap-in opens |
| `Get-GPResultantSetOfPolicy -ReportType Html -Path <path>` | Generates PowerShell RSoP report | HTML report exists |
| `Get-GPResultantSetOfPolicy -Computer <computer> -User <domain\user> -ReportType Html -Path <path>` | Generates remote RSoP report | Remote report exists if permissions and firewall allow |
| `Invoke-GPUpdate -Computer <computer> -Force -RandomDelayInMinutes 0` | Triggers remote policy refresh | Remote refresh request is created |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews client-side Group Policy events | Recent processing events appear |
| `Get-GPInheritance -Target "<ou-dn>"` | Confirms linked and inherited GPO path | Expected GPO appears in OU scope |
| `Get-GPPermission -Name "<gpo-name>" -All` | Confirms security filtering and permissions | Target principal has read and apply route |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path <path>` | Exports intended GPO configuration | Intended GPO report exists |
| `secedit /export /cfg <path>` | Exports effective security policy | INF export exists |
| `auditpol /get /category:*` | Shows effective audit policy | Audit subcategories appear |
| `whoami /groups` | Shows current user group token | Expected security groups appear |
| `klist purge` | Purges Kerberos tickets for current user | Tickets are purged |
| `Restart-Computer` | Forces new computer policy session | Client restarts |
| Sign out / sign in | Forces new user token and user policy session | User receives new policy token |

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Rollback
```powershell
# This workbook is validation-focused and normally does not change GPO configuration.
# Rollback here means cleanup of generated reports or undoing remote refresh/test artifacts.

$ReportRoot = "C:\GPOPrep\Reports"

# Optional cleanup:
# Remove generated local validation reports.
# Remove-Item "$ReportRoot\Client-GPResult" -Recurse -Force -ErrorAction SilentlyContinue
# Remove-Item "$ReportRoot\Management-RSoP" -Recurse -Force -ErrorAction SilentlyContinue
# Remove-Item "$ReportRoot\Bundles" -Recurse -Force -ErrorAction SilentlyContinue

# Optional cleanup:
# Remove local temporary validation folder if created.
# Remove-Item "C:\GPOPrep" -Recurse -Force -ErrorAction SilentlyContinue

# If remote Invoke-GPUpdate created scheduled tasks and they remain visible,
# inspect Task Scheduler on the client under Microsoft > Windows > GroupPolicy.
# Usually the task runs and cleans itself up.

Write-Host "Validation rollback is normally just report cleanup."
Write-Host "Do not delete reports until the validation evidence has been reviewed or archived."
```

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `gpupdate` cannot contact domain controller | DNS, network, domain join, or DC locator issue | `nltest /dsgetdc:<domain>`; `ipconfig /all` | Fix DNS, network, or domain join |
| SYSVOL cannot be reached | DNS, SMB, DC, or SYSVOL replication issue | `Test-Path "\\<domain>\SYSVOL\<domain>\Policies"` | Fix DNS, SMB, DC, or SYSVOL |
| `gpupdate /force` succeeds but GPO missing | GPO not in scope, filtering denies it, or wrong user/computer OU | `gpresult /r`; `Get-GPInheritance` | Fix OU link, target object location, or filtering |
| Computer GPO missing | Computer account is not in linked OU | `Get-ADComputer -Identity "<computer>" -Properties DistinguishedName` | Move computer or link GPO to correct OU |
| User GPO missing | User account is not in linked OU | `Get-ADUser -Identity "<user>" -Properties DistinguishedName` | Move user or link GPO to correct OU |
| GPO appears under denied list | Security filtering, WMI filter, disabled link, or empty GPO | `gpresult /r`; `Get-GPPermission`; GPO report | Fix filtering, WMI, link state, or add settings |
| GPO denied due to security filtering | Target lacks Apply Group Policy permission | `Get-GPPermission -Name "<gpo>" -All` | Grant `GpoApply` to correct group |
| GPO denied due to WMI filter | Client does not match WMI query | Test WMI query with `Get-CimInstance -Query` | Correct WMI filter or remove it |
| GPO not listed at all | GPO not linked or wrong OU path | `Get-GPInheritance -Target "<ou-dn>"` | Link GPO to correct scope |
| GPO listed but settings not visible | Wrong policy side or setting not configured | GPO report and `gpresult /h` | Configure correct Computer or User side |
| User group membership not reflected | User Kerberos token stale | `whoami /groups` | Sign out and sign back in |
| Computer group membership not reflected | Computer token stale | `gpresult /scope computer /r` | Reboot computer |
| `gpresult /h` access denied | Output path permission issue or not elevated for full report | Run elevated; change path | Use writable path such as `C:\GPOPrep\Reports` |
| `gpresult` says user does not have RSoP data | User has not logged on or no user policy processed | `whoami`; sign in as test user | Log on as user and run `gpupdate /force` |
| Remote RSoP fails | Firewall, RPC, WMI, permissions, or remote registry issue | `Test-NetConnection <computer> -Port 135`; `Test-WSMan` | Enable required remote management access or run locally |
| `Invoke-GPUpdate` fails | Firewall, scheduled task remote management, permissions, or offline computer | `Test-Connection <computer>` | Fix remote management or run local `gpupdate` |
| `rsop.msc` shows stale data | Policy not refreshed or WMI/RSoP repository stale | `gpupdate /force`; rerun RSoP | Refresh policy and reopen RSoP |
| Setting requires reboot | Computer-side setting needs startup processing | `gpupdate /boot` or reboot | Reboot client |
| Setting requires logoff | User-side setting needs new session | `gpupdate /logoff` or sign out | Sign out and sign back in |
| Loopback changes user GPO result | Loopback processing enabled on computer GPO | `gpresult /h` | Review loopback mode and computer OU policy |
| Another GPO wins | Link order, enforcement, inheritance, or closer OU GPO overrides setting | `gpresult /h`; `Get-GPInheritance` | Adjust precedence intentionally |
| Block inheritance hides parent GPO | OU has block inheritance enabled | `Get-GPInheritance -Target "<ou-dn>"` | Remove block or enforce parent only if justified |
| Enforced GPO overrides child setting | Parent GPO link is enforced | `Get-GPInheritance` | Remove enforcement or accept intended override |
| GPP item missing but GPO applied | Item-level targeting failed or preference processing error | GPO report and event log | Fix targeting or item configuration |
| Script GPO applied but script did not run | Script path, execution, timing, or context issue | Script logs and GroupPolicy Operational log | Fix script path, context, and logging |
| Policy differs between clients | OU placement, group token, WMI, replication, or local state differs | Compare `gpresult /h` reports | Normalize scope and client state |
| GPO differs between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before further validation |
| No clear conclusion | Reports were not collected before changes | Evidence bundle missing | Collect `gpupdate`, `gpresult`, RSoP, event logs, and intended GPO report |

# 15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GroupPolicy module and GPMC tools |
| `02_Create_And_Link_Baseline_GPO.md` | Creates GPOs that must be validated |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Explains precedence conditions visible in `gpresult` |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Explains denied GPOs caused by filtering |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Explains denied GPOs caused by WMI filters |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Produces computer policies validated with `gpresult /scope computer` |
| `07_Configure_User_Administrative_Template_Settings.md` | Produces user policies validated with `gpresult /scope user` |
| `08_Configure_Group_Policy_Preferences.md` | Produces GPP items validated with `gpresult`, event logs, and behavior checks |
| `09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md` | Produces script GPOs validated with logs and events |
| `10_Configure_Folder_Redirection_Drive_Maps_And_Printer_Deployment.md` | Produces user resource GPOs validated with RSoP and client behavior |
| `11_Configure_Security_Baseline_GPO_Settings.md` | Produces security baseline settings validated with `secedit`, `auditpol`, and RSoP |
| `12_Configure_Windows_Update_And_Defender_GPO_Settings.md` | Produces update and Defender policies validated with registry, service, and event checks |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md` | Produces remote admin policies validated with local group, firewall, and remote access checks |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Provides intended GPO reports used for before and after comparison |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms domain policy infrastructure before deep validation |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Next workbook for diagnosing failed processing, SYSVOL, replication, and client-side failures |