23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs.md
# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Index
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs.md
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Source_Basis
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Mental_Model
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Planning_Table
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Control_Map
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Configuration_Checklist
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Precheck_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Create_And_Link_GPO_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Advanced_Audit_GPMC_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Event_Log_Settings_GPMC_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Event_Collector_Server_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Source_Initiated_Subscription_GPO_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Client_Validation_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Collector_Validation_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Report_And_Backup_Skeleton
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Verification_Commands
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Rollback
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Failure_Checks
23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Related_Labs

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Advanced Security Audit Policy | Configuring detailed audit subcategories through GPO |
| Microsoft Learn | Auditpol command | Validating effective audit policy on clients |
| Microsoft Learn | Windows Event Forwarding | Collector-initiated and source-initiated event forwarding |
| Microsoft Learn | Windows Event Collector service | Configuring WEC collector service and subscriptions |
| Microsoft Learn | WinRM service | Transport used by Windows Event Forwarding |
| Microsoft Learn | Event log Group Policy settings | Controlling log size, retention, and access |
| Microsoft Learn | Group Policy Management Console | Configuring audit policy, event log, WinRM, and event forwarding GPO settings |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up audit and WEF GPOs |
| Windows Server operational practice | Use pilot OU and source-initiated WEF for scalable endpoint collection | Safe rollout and centralized security visibility |

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Advanced Audit Policy | Detailed audit subcategories under Security Settings that replace broad legacy audit categories |
| Audit subcategory | Specific audit area such as Logon, Account Lockout, Process Creation, Policy Change, or Directory Service Access |
| Success auditing | Logs successful activity |
| Failure auditing | Logs failed attempts |
| Audit policy precedence | Advanced Audit Policy should be forced over legacy audit policy to avoid conflicts |
| Security log | Primary log for audit events such as logons, privilege use, object access, and policy changes |
| Event log size | Maximum size of a log before retention behavior applies |
| Retention behavior | Determines whether logs overwrite old events or require manual clearing |
| Windows Event Forwarding | Native Windows mechanism for forwarding events from source computers to collector servers |
| Collector server | Server receiving forwarded events |
| Source computer | Endpoint or server that forwards events to the collector |
| Source-initiated subscription | Source computers discover and connect to collector by GPO |
| Collector-initiated subscription | Collector reaches out to explicitly listed source computers |
| Subscription manager | GPO setting telling clients where to send forwarded events |
| Forwarded Events log | Collector-side log where received events are stored by default |
| WinRM | Transport used by WEF source computers to communicate with collector |
| WEC service | Windows Event Collector service on the collector server |
| Event query | XPath or subscription filter defining which events are collected |
| Minimize bandwidth | WEF delivery mode optimized for lower network traffic |
| Minimize latency | WEF delivery mode optimized for faster event arrival |
| First rule | Configure audit policy first, then forward the events that matter |
| Blunt rule | WEF will not fix bad audit policy because unlogged events cannot be forwarded |

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-computer-ou-dn>` |
| Server OU | `OU=Servers,OU=Corp,DC=corp,DC=local` | `<server-ou-dn>` |
| Test client | `WIN11-01` | `<test-client>` |
| Event collector | `WEC01` | `<collector-server>` |
| Collector FQDN | `WEC01.corp.local` | `<collector-fqdn>` |
| Collector URL | `Server=http://WEC01:5985/wsman/SubscriptionManager/WEC,Refresh=60` | `<subscription-manager-url>` |
| Audit GPO | `CORP-Workstation-Advanced-Audit-Policy` | `<audit-gpo-name>` |
| Event Forwarding GPO | `CORP-Workstation-Event-Forwarding` | `<wef-gpo-name>` |
| Event Log GPO | `CORP-Workstation-Event-Log-Settings` | `<eventlog-gpo-name>` |
| Security filtering group | `GG_GPO_Audit_WEF_Apply` | `<filtering-group>` |
| Collector access group | `GG_WEF_Event_Sources` | `<event-source-group>` |
| Security log max size | `262144 KB` | `<security-log-size-kb>` |
| System log max size | `65536 KB` | `<system-log-size-kb>` |
| Application log max size | `65536 KB` | `<application-log-size-kb>` |
| Forwarded Events log size | `262144 KB` | `<forwarded-events-size-kb>` |
| Audit categories | Logon, Account Logon, Policy Change, Account Management, Process Creation | `<audit-scope>` |
| WEF subscription name | `CORP-Workstation-Security-Baseline-Events` | `<subscription-name>` |
| WEF delivery mode | Normal, Minimize Bandwidth, Minimize Latency | `<delivery-mode>` |
| Report path | `C:\GPOPrep\Reports\Audit-WEF` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup\Audit-WEF` | `<backup-path>` |
| Rollback method | Disable GPO link, delete subscription, restore GPO backup | `<rollback-plan>` |

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Control_Map
| Control | GPO Location / Tool | Purpose | Validation |
|---|---|---|---|
| Force Advanced Audit Policy | `Security Options > Audit: Force audit policy subcategory settings...` | Prevents legacy audit policy from overriding advanced subcategories | `auditpol /get /category:*` |
| Logon audit | `Advanced Audit Policy Configuration > Logon/Logoff` | Tracks sign-ins, RDP, logoff, lockout-related activity | Security log events |
| Account Logon audit | `Advanced Audit Policy Configuration > Account Logon` | Tracks credential validation and Kerberos activity | Security log on DCs |
| Account Management audit | `Advanced Audit Policy Configuration > Account Management` | Tracks user, group, and computer account changes | Security log |
| Policy Change audit | `Advanced Audit Policy Configuration > Policy Change` | Tracks audit policy, authentication policy, and authorization changes | Security log |
| Privilege Use audit | `Advanced Audit Policy Configuration > Privilege Use` | Tracks sensitive privilege use | Security log |
| Process Creation audit | `Advanced Audit Policy Configuration > Detailed Tracking` | Tracks process start events | Security log event 4688 |
| Command line logging | `Administrative Templates > System > Audit Process Creation` | Adds command line details to process creation events | Security event 4688 includes command line |
| Event log size | `Windows Components > Event Log Service` | Prevents logs from rolling too quickly | `wevtutil gl Security` |
| WinRM client | `Administrative Templates > Windows Remote Management > WinRM Client` | Allows source event forwarding transport | `winrm get winrm/config/client` |
| WinRM service | `Administrative Templates > Windows Remote Management > WinRM Service` | Allows remote management where required | `winrm get winrm/config/service` |
| Subscription Manager | `Event Forwarding > Configure target Subscription Manager` | Points sources to WEC collector | Registry and event forwarding log |
| WEC service | `wecutil qc` | Initializes event collector service | `Get-Service Wecsvc` |
| WEF subscription | Event Viewer or `wecutil cs` | Defines forwarded event query and source group | `wecutil es` and `wecutil gs` |
| Forwarded Events log | Collector Event Viewer | Stores received events | Events appear on collector |
| Event source membership | Collector subscription source computer group | Controls which endpoints can forward events | Runtime status shows active sources |

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 6 | Confirm pilot computer OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-computer-ou-dn>"` | Pilot OU returns |
| 7 | Confirm test client exists | Management Host | `Get-ADComputer -Identity "<test-client>" -Properties DistinguishedName,Enabled` | Test client returns |
| 8 | Confirm collector server exists | Management Host | `Get-ADComputer -Identity "<collector-server>" -Properties DNSHostName,Enabled` | Collector computer object returns |
| 9 | Confirm collector DNS resolves | Management Host | `Resolve-DnsName "<collector-fqdn>"` | Collector resolves |
| 10 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\Audit-WEF` | Report path exists |
| 11 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup\Audit-WEF` | Backup path exists |
| 12 | Back up current GPO state | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup\Audit-WEF` | Pre-change backup exists |
| 13 | Create audit GPO | Management Host | `New-GPO -Name "<audit-gpo-name>" -Comment "Advanced audit policy for pilot workstations"` | Audit GPO exists |
| 14 | Create WEF GPO | Management Host | `New-GPO -Name "<wef-gpo-name>" -Comment "Windows Event Forwarding source policy"` | WEF GPO exists |
| 15 | Create event log GPO | Management Host | `New-GPO -Name "<eventlog-gpo-name>" -Comment "Event log size and retention policy"` | Event log GPO exists |
| 16 | Link audit GPO to pilot OU | Management Host | `New-GPLink -Name "<audit-gpo-name>" -Target "<pilot-computer-ou-dn>" -LinkEnabled Yes` | Audit GPO is linked |
| 17 | Link WEF GPO to pilot OU | Management Host | `New-GPLink -Name "<wef-gpo-name>" -Target "<pilot-computer-ou-dn>" -LinkEnabled Yes` | WEF GPO is linked |
| 18 | Link event log GPO to pilot OU | Management Host | `New-GPLink -Name "<eventlog-gpo-name>" -Target "<pilot-computer-ou-dn>" -LinkEnabled Yes` | Event log GPO is linked |
| 19 | Keep GPO links unenforced by default | Management Host | `Set-GPLink -Name "<gpo-name>" -Target "<pilot-computer-ou-dn>" -Enforced No` | Links are not enforced |
| 20 | Configure force advanced audit policy | Management Host | `GPMC > Security Options > Audit: Force audit policy subcategory settings...` | Advanced audit precedence is configured |
| 21 | Configure Logon/Logoff audit subcategories | Management Host | `GPMC > Advanced Audit Policy Configuration > Logon/Logoff` | Logon auditing is configured |
| 22 | Configure Account Management audit subcategories | Management Host | `GPMC > Advanced Audit Policy Configuration > Account Management` | Account change auditing is configured |
| 23 | Configure Policy Change audit subcategories | Management Host | `GPMC > Advanced Audit Policy Configuration > Policy Change` | Policy change auditing is configured |
| 24 | Configure Detailed Tracking if required | Management Host | `GPMC > Advanced Audit Policy Configuration > Detailed Tracking` | Process creation auditing is configured |
| 25 | Configure command line in process creation if required | Management Host | `GPMC > Administrative Templates > System > Audit Process Creation` | Command line logging is configured |
| 26 | Configure event log sizes | Management Host | `GPMC > Administrative Templates > Windows Components > Event Log Service` | Security, System, and Application sizes are configured |
| 27 | Configure WinRM client if required | Management Host | `GPMC > Windows Remote Management > WinRM Client` | WinRM client behavior is configured |
| 28 | Configure target Subscription Manager | Management Host | `GPMC > Administrative Templates > Windows Components > Event Forwarding > Configure target Subscription Manager` | Client knows collector URL |
| 29 | Initialize WEC collector | Collector Server | `wecutil qc` | WEC service is configured |
| 30 | Confirm collector service | Collector Server | `Get-Service Wecsvc,WinRM` | WEC and WinRM services are running |
| 31 | Create WEF source group if used | Management Host | `New-ADGroup -Name "GG_WEF_Event_Sources" -GroupScope Global -GroupCategory Security -Path "<groups-ou-dn>"` | Event source group exists |
| 32 | Add pilot client to source group | Management Host | `Add-ADGroupMember -Identity "GG_WEF_Event_Sources" -Members "<test-client>$"` | Test client is member |
| 33 | Create WEF subscription | Collector Server | `Event Viewer > Subscriptions > Create Subscription` | Subscription exists |
| 34 | Configure subscription source computers | Collector Server | `Subscription Properties > Source Computers > Add Domain Computers or group` | Source group is assigned |
| 35 | Configure event query | Collector Server | `Subscription Properties > Select Events` | Desired events are selected |
| 36 | Refresh policy on test client | Test Client | `gpupdate /force` | Policy refresh completes |
| 37 | Reboot test client if group membership changed | Test Client | `Restart-Computer` | Computer token refreshes |
| 38 | Validate audit policy | Test Client | `auditpol /get /category:*` | Configured subcategories appear |
| 39 | Generate test events | Test Client | `whoami; gpupdate /force; run test logon/process activity` | Test events are generated |
| 40 | Validate local event logs | Test Client | `Get-WinEvent -LogName Security -MaxEvents 50` | Security events appear |
| 41 | Validate event forwarding client events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-Forwarding/Operational" -MaxEvents 100` | Forwarding activity appears |
| 42 | Validate collector subscription status | Collector Server | `wecutil es; wecutil gs "<subscription-name>"` | Subscription is visible |
| 43 | Validate forwarded events | Collector Server | `Get-WinEvent -LogName ForwardedEvents -MaxEvents 100` | Events from pilot client appear |
| 44 | Export GPO reports | Management Host | `Get-GPOReport -Name "<audit-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\Audit-WEF\<audit-gpo-name>.html` | Reports exist |
| 45 | Back up configured GPOs | Management Host | `Backup-GPO -Name "<audit-gpo-name>" -Path C:\GPOPrep\Backup\Audit-WEF` | GPO backups exist |
| 46 | Document result | Operator | `Record audit subcategories, log sizes, collector URL, subscription, source group, forwarded event proof, reports, and rollback path` | Audit and WEF deployment is documented |

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate domain, OU, client, collector, and GPO readiness before configuring audit and WEF policy.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$TestClient = "WIN11-01"
$CollectorServer = "WEC01"

$AuditGpoName = "CORP-Workstation-Advanced-Audit-Policy"
$WefGpoName = "CORP-Workstation-Event-Forwarding"
$EventLogGpoName = "CORP-Workstation-Event-Log-Settings"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports\Audit-WEF"
$BackupPath = "$BasePath\Backup\Audit-WEF"

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

# Confirm OU and computers.
Get-ADOrganizationalUnit `
  -Identity $PilotComputerOU |
  Out-File "$ReportPath\pilot-computer-ou.txt"

Get-ADComputer `
  -Identity $TestClient `
  -Properties DNSHostName,Enabled,DistinguishedName,MemberOf |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-client-before-audit-wef.txt"

Get-ADComputer `
  -Identity $CollectorServer `
  -Properties DNSHostName,Enabled,DistinguishedName |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\collector-server-ad-state.txt"

Resolve-DnsName $CollectorServer |
  Out-File "$ReportPath\collector-dns-resolution.txt" `
  -ErrorAction SilentlyContinue

# Capture inheritance.
Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-audit-wef.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-audit-wef.csv" -NoTypeInformation

# Back up current GPOs.
Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre advanced audit and WEF configuration backup"

Write-Host "Advanced audit and WEF precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link audit, event log, and event forwarding GPOs to pilot OU.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoNames = @(
  "CORP-Workstation-Advanced-Audit-Policy",
  "CORP-Workstation-Event-Forwarding",
  "CORP-Workstation-Event-Log-Settings"
)

# Confirm pilot OU.
Get-ADOrganizationalUnit -Identity $PilotComputerOU

foreach ($GpoName in $GpoNames) {
    if (-not (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue)) {
        New-GPO `
          -Name $GpoName `
          -Comment "Advanced audit, event log, or Windows Event Forwarding policy"
    }

    $Inheritance = Get-GPInheritance -Target $PilotComputerOU

    if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $GpoName)) {
        New-GPLink `
          -Name $GpoName `
          -Target $PilotComputerOU `
          -LinkEnabled Yes
    }

    Set-GPLink `
      -Name $GpoName `
      -Target $PilotComputerOU `
      -LinkEnabled Yes `
      -Enforced No
}

# Confirm.
Get-GPInheritance `
  -Target $PilotComputerOU |
  Format-List
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Advanced_Audit_GPMC_Skeleton
```powershell
# Native GUI workflow for Advanced Audit Policy.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-Workstation-Advanced-Audit-Policy
# 3. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Local Policies
#      > Security Options
# 4. Open:
#      Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings
# 5. Set:
#      Enabled
#
# Advanced Audit Policy path:
# 6. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Advanced Audit Policy Configuration
#      > Audit Policies
#
# Recommended workstation pilot baseline:
# 7. Account Logon:
#      Audit Credential Validation: Success and Failure
#
# 8. Account Management:
#      Audit User Account Management: Success and Failure
#      Audit Security Group Management: Success and Failure
#      Audit Computer Account Management: Success and Failure
#
# 9. Logon/Logoff:
#      Audit Logon: Success and Failure
#      Audit Logoff: Success
#      Audit Account Lockout: Success and Failure
#      Audit Special Logon: Success
#      Audit Other Logon/Logoff Events: Success and Failure
#
# 10. Policy Change:
#      Audit Audit Policy Change: Success and Failure
#      Audit Authentication Policy Change: Success and Failure
#      Audit Authorization Policy Change: Success and Failure
#
# 11. Privilege Use:
#      Audit Sensitive Privilege Use: Success and Failure
#
# 12. Detailed Tracking:
#      Audit Process Creation: Success
#      Audit Process Termination: Success only if required
#
# 13. Object Access:
#      Configure only when needed.
#      Object Access can generate large volume and often requires SACLs.
#
# Command line in process creation:
# 14. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > System
#      > Audit Process Creation
# 15. Open:
#      Include command line in process creation events
# 16. Set:
#      Enabled
#
# Notes:
# - Start with pilot workstations.
# - Monitor Security log growth.
# - Do not enable every subcategory blindly.
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Event_Log_Settings_GPMC_Skeleton
```powershell
# Native GUI workflow for event log sizing and retention.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Edit:
#      CORP-Workstation-Event-Log-Settings
# 2. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > Event Log Service
#
# Security log:
# 3. Go to:
#      Security
# 4. Configure:
#      Specify the maximum log file size
#      Example: 262144 KB
# 5. Configure retention behavior based on lab objective.
#      Typical endpoint baseline: overwrite events as needed.
#
# System log:
# 6. Go to:
#      System
# 7. Configure:
#      Specify the maximum log file size
#      Example: 65536 KB
#
# Application log:
# 8. Go to:
#      Application
# 9. Configure:
#      Specify the maximum log file size
#      Example: 65536 KB
#
# Forwarded Events log on collector:
# 10. Configure directly on the collector or through a server-targeted GPO.
# 11. Example command on collector:
#      wevtutil sl ForwardedEvents /ms:268435456
#
# Notes:
# - Audit policy without enough log size causes rapid event loss.
# - WEF reduces endpoint retention pressure but does not replace local log sizing.
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Event_Collector_Server_Skeleton
```powershell
# Run on the Windows Event Collector server.
# Purpose: initialize WinRM and Windows Event Collector, create source group, and prepare collector log.

Import-Module ActiveDirectory

$Domain = Get-ADDomain
$DomainDN = $Domain.DistinguishedName

$CollectorServer = "WEC01"
$SourceGroup = "GG_WEF_Event_Sources"
$GroupsOU = "OU=Groups,OU=Corp,$DomainDN"

$ReportPath = "C:\GPOPrep\Reports\Audit-WEF-Collector"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm identity.
hostname |
  Out-File "$ReportPath\collector-hostname.txt"

whoami /all |
  Out-File "$ReportPath\collector-whoami-all.txt"

# Initialize WinRM if needed.
winrm quickconfig

# Initialize Windows Event Collector.
wecutil qc

# Confirm services.
Get-Service `
  -Name Wecsvc,WinRM |
  Select-Object Name,Status,StartType |
  Out-File "$ReportPath\collector-services.txt"

# Increase Forwarded Events log size.
wevtutil sl ForwardedEvents /ms:268435456

# Confirm Forwarded Events log configuration.
wevtutil gl ForwardedEvents |
  Out-File "$ReportPath\forwarded-events-log-config.txt"

# Create source group if missing.
if (-not (Get-ADGroup -Identity $SourceGroup -ErrorAction SilentlyContinue)) {
    New-ADGroup `
      -Name $SourceGroup `
      -SamAccountName $SourceGroup `
      -GroupScope Global `
      -GroupCategory Security `
      -Path $GroupsOU `
      -Description "Computers allowed to forward events to Windows Event Collector"
}

# Record source group.
Get-ADGroup `
  -Identity $SourceGroup `
  -Properties DistinguishedName,Members |
  Select-Object Name,DistinguishedName,Members |
  Out-File "$ReportPath\wef-source-group.txt"

# Event Viewer subscription setup is usually GUI-based:
eventvwr.msc

# GUI workflow:
# 1. Open Event Viewer.
# 2. Go to Subscriptions.
# 3. If prompted, allow the Windows Event Collector service to start.
# 4. Create Subscription.
# 5. Name:
#      CORP-Workstation-Security-Baseline-Events
# 6. Destination log:
#      Forwarded Events
# 7. Subscription type:
#      Source computer initiated
# 8. Source computers:
#      Add domain computer group CORP\GG_WEF_Event_Sources
# 9. Events to collect:
#      Select Events
#      Security log
#      Key Event IDs or XML query
# 10. Advanced:
#      Delivery mode: Normal or Minimize Bandwidth
#      Protocol: HTTP
#      Port: 5985
#
# Example key Security Event IDs:
# 4624,4625,4634,4648,4672,4688,4697,4700,4702,4719,4720,4722,4723,4724,4725,4726,4732,4733,4738,4740,4768,4769,4771,4776

# Validate current subscriptions.
wecutil es |
  Out-File "$ReportPath\wecutil-enum-subscriptions.txt"

Write-Host "Collector initialization complete."
Write-Host "Create or validate subscriptions in Event Viewer."
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Source_Initiated_Subscription_GPO_Skeleton
```powershell
# Native GUI workflow for WEF source client GPO.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Edit:
#      CORP-Workstation-Event-Forwarding
# 2. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > Event Forwarding
# 3. Open:
#      Configure target Subscription Manager
# 4. Set:
#      Enabled
# 5. Add subscription manager value:
#      Server=http://WEC01:5985/wsman/SubscriptionManager/WEC,Refresh=60
#
# WinRM client policy:
# 6. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > Windows Remote Management (WinRM)
#      > WinRM Client
# 7. Configure trusted hosts only if required by design.
#      Domain Kerberos scenarios usually should not need broad TrustedHosts.
#
# Windows Remote Management service:
# 8. Go to:
#      Windows Remote Management (WinRM)
#      > WinRM Service
# 9. Open:
#      Allow remote server management through WinRM
# 10. Set:
#      Enabled
# 11. IPv4 filter:
#      *
#      or a scoped management range if required
#
# Firewall:
# 12. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Windows Defender Firewall with Advanced Security
#      > Inbound Rules
# 13. Ensure Windows Remote Management rules are allowed if required.
#
# Service startup:
# 14. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > System Services
# 15. Configure:
#      Windows Remote Management
#      Startup mode: Automatic
#
# Notes:
# - Source-initiated subscriptions scale better than collector-initiated subscriptions.
# - Pilot clients must be in the source group configured on the collector subscription.
# - Reboot clients after adding them to source computer groups.
```

```powershell
# Optional PowerShell registry policy for Subscription Manager.
# Use GPMC where possible, but this documents the backing policy value.

Import-Module GroupPolicy

$WefGpoName = "CORP-Workstation-Event-Forwarding"
$Collector = "WEC01"
$SubscriptionManager = "Server=http://$Collector`:5985/wsman/SubscriptionManager/WEC,Refresh=60"

Set-GPRegistryValue `
  -Name $WefGpoName `
  -Key "HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager" `
  -ValueName "1" `
  -Type String `
  -Value $SubscriptionManager

Get-GPOReport `
  -Name $WefGpoName `
  -ReportType Html `
  -Path "C:\GPOPrep\Reports\Audit-WEF\$WefGpoName-subscription-manager.html"
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Client_Validation_Skeleton
```powershell
# Run on the pilot source client.
# Purpose: validate audit policy, event log settings, WinRM, and WEF source behavior.

$DomainFqdn = "corp.local"
$Collector = "WEC01"
$ReportPath = "C:\GPOPrep\Reports\Audit-WEF-Client"

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

# Confirm collector resolution and connectivity.
Resolve-DnsName $Collector |
  Out-File "$ReportPath\collector-dns-resolution.txt" `
  -ErrorAction SilentlyContinue

Test-NetConnection `
  -ComputerName $Collector `
  -Port 5985 |
  Out-File "$ReportPath\collector-5985-test.txt"

# Refresh policy.
gpupdate /force |
  Tee-Object "$ReportPath\gpupdate-force.txt"

# Reboot if computer group membership changed.
# Restart-Computer

# Export GPResult.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-r.txt"

gpresult /h "$ReportPath\gpresult-audit-wef.html"
gpresult /x "$ReportPath\gpresult-audit-wef.xml"

# Validate effective audit policy.
auditpol /get /category:* |
  Out-File "$ReportPath\auditpol-effective-all.txt"

auditpol /get /subcategory:"Process Creation" |
  Out-File "$ReportPath\auditpol-process-creation.txt"

auditpol /get /subcategory:"Logon" |
  Out-File "$ReportPath\auditpol-logon.txt"

# Validate event log sizes.
wevtutil gl Security |
  Out-File "$ReportPath\security-log-config.txt"

wevtutil gl System |
  Out-File "$ReportPath\system-log-config.txt"

wevtutil gl Application |
  Out-File "$ReportPath\application-log-config.txt"

# Validate Subscription Manager policy.
Get-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager" `
  -ErrorAction SilentlyContinue |
  Out-File "$ReportPath\subscription-manager-policy.txt"

# Validate WinRM.
winrm get winrm/config/client |
  Out-File "$ReportPath\winrm-client-config.txt"

winrm get winrm/config/service |
  Out-File "$ReportPath\winrm-service-config.txt" `
  -ErrorAction SilentlyContinue

Get-Service `
  -Name WinRM `
  -ErrorAction SilentlyContinue |
  Select-Object Name,Status,StartType |
  Out-File "$ReportPath\winrm-service-state.txt"

# Generate test events.
whoami |
  Out-File "$ReportPath\test-whoami-command.txt"

cmd /c echo Audit-WEF process creation test |
  Out-File "$ReportPath\process-creation-test.txt"

# Review local Security log.
Get-WinEvent `
  -LogName Security `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Out-File "$ReportPath\security-events-recent.txt"

# Review event forwarding client log.
Get-WinEvent `
  -LogName "Microsoft-Windows-Forwarding/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\forwarding-operational-events.txt"

# Review WinRM operational log.
Get-WinEvent `
  -LogName "Microsoft-Windows-WinRM/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\winrm-operational-events.txt"

Write-Host "Audit and WEF client validation complete."
Write-Host "Report path: $ReportPath"
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Collector_Validation_Skeleton
```powershell
# Run on the Windows Event Collector server.
# Purpose: validate subscriptions, source runtime status, and forwarded events.

$SubscriptionName = "CORP-Workstation-Security-Baseline-Events"
$ReportPath = "C:\GPOPrep\Reports\Audit-WEF-Collector"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm collector identity.
hostname |
  Out-File "$ReportPath\collector-hostname.txt"

whoami /all |
  Out-File "$ReportPath\collector-whoami-all.txt"

# Confirm services.
Get-Service `
  -Name Wecsvc,WinRM |
  Select-Object Name,Status,StartType |
  Out-File "$ReportPath\collector-services.txt"

# Confirm WinRM listener.
winrm enumerate winrm/config/listener |
  Out-File "$ReportPath\winrm-listeners.txt"

# List subscriptions.
wecutil es |
  Out-File "$ReportPath\wecutil-subscriptions.txt"

# Get subscription configuration.
wecutil gs "$SubscriptionName" |
  Out-File "$ReportPath\wecutil-$SubscriptionName-config.txt" `
  -ErrorAction SilentlyContinue

# Get runtime subscription status.
wecutil gr "$SubscriptionName" |
  Out-File "$ReportPath\wecutil-$SubscriptionName-runtime.txt" `
  -ErrorAction SilentlyContinue

# Confirm Forwarded Events log configuration.
wevtutil gl ForwardedEvents |
  Out-File "$ReportPath\forwarded-events-log-config.txt"

# Review forwarded events.
Get-WinEvent `
  -LogName ForwardedEvents `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,MachineName,LevelDisplayName,Message |
  Out-File "$ReportPath\forwarded-events-recent.txt"

# Count forwarded events by source.
Get-WinEvent `
  -LogName ForwardedEvents `
  -MaxEvents 1000 `
  -ErrorAction SilentlyContinue |
  Group-Object MachineName |
  Sort-Object Count -Descending |
  Select-Object Name,Count |
  Out-File "$ReportPath\forwarded-events-by-source.txt"

# Review collector-side WEC logs.
Get-WinEvent `
  -LogName "Microsoft-Windows-EventCollector/Operational" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\eventcollector-operational-events.txt"

Write-Host "Collector validation complete."
Write-Host "Report path: $ReportPath"
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final audit and WEF GPO reports, inheritance, permissions, and backups.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoNames = @(
  "CORP-Workstation-Advanced-Audit-Policy",
  "CORP-Workstation-Event-Forwarding",
  "CORP-Workstation-Event-Log-Settings"
)

$ReportPath = "C:\GPOPrep\Reports\Audit-WEF"
$BackupPath = "C:\GPOPrep\Backup\Audit-WEF"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture inheritance.
Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-ou-inheritance-final-audit-wef.txt"

# Export reports and backups.
foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPPermission `
          -Name $GpoName `
          -All |
          Out-File "$ReportPath\$GpoName-permissions.txt"

        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-final.html"

        Get-GPOReport `
          -Name $GpoName `
          -ReportType Xml `
          -Path "$ReportPath\$GpoName-final.xml"

        Backup-GPO `
          -Name $GpoName `
          -Path $BackupPath `
          -Comment "Final advanced audit and WEF GPO backup"
    }
}

# Search GPO XML reports.
Select-String `
  -Path "$ReportPath\*.xml" `
  -Pattern "Audit","Advanced Audit","Event Forwarding","SubscriptionManager","WinRM","Event Log Service","Process Creation","Command Line","Security" |
  Out-File "$ReportPath\audit-wef-gpo-report-search.txt"

# Replication checks.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-audit-wef.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-audit-wef.txt"

Write-Host "Advanced audit and WEF reports and backups complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<audit-gpo-name>"` | Confirms audit GPO exists | GPO object returns |
| `Get-GPO -Name "<wef-gpo-name>"` | Confirms WEF GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-computer-ou-dn>"` | Confirms GPO links | Audit, WEF, and event log GPOs appear |
| `Get-GPPermission -Name "<gpo-name>" -All` | Confirms target computers can apply GPO | Expected permissions appear |
| `gpupdate /force` | Forces client policy refresh | Policy refresh completes |
| `gpresult /scope computer /r` | Shows applied computer GPOs | Audit and WEF GPOs appear |
| `gpresult /h C:\GPOPrep\Reports\Audit-WEF\gpresult-audit-wef.html` | Exports RSOP report | HTML report exists |
| `auditpol /get /category:*` | Shows effective advanced audit policy | Configured subcategories appear |
| `auditpol /get /subcategory:"Process Creation"` | Validates process creation audit | Success is enabled if configured |
| `wevtutil gl Security` | Validates Security log size and retention | Configured max size appears |
| `Get-WinEvent -LogName Security -MaxEvents 100` | Validates local Security events | Security events appear |
| `Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager"` | Validates subscription manager policy | Collector URL appears |
| `Test-NetConnection <collector> -Port 5985` | Tests client to collector WinRM port | TCP succeeds |
| `winrm get winrm/config/client` | Validates WinRM client config | WinRM client config returns |
| `Get-Service WinRM` | Confirms WinRM service state | Service is running if required |
| `wecutil qc` | Initializes collector service | WEC configured |
| `Get-Service Wecsvc,WinRM` | Validates collector services | Services running |
| `wecutil es` | Lists WEF subscriptions | Subscription name appears |
| `wecutil gs "<subscription-name>"` | Shows subscription configuration | Subscription settings return |
| `wecutil gr "<subscription-name>"` | Shows subscription runtime status | Source status appears |
| `Get-WinEvent -LogName ForwardedEvents -MaxEvents 100` | Validates forwarded events on collector | Events from source computers appear |
| `Get-WinEvent -LogName "Microsoft-Windows-Forwarding/Operational" -MaxEvents 100` | Validates source forwarding activity | Forwarding events appear |
| `Get-WinEvent -LogName "Microsoft-Windows-EventCollector/Operational" -MaxEvents 100` | Validates collector activity | Collector events appear |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path "<path>"` | Exports readable GPO report | HTML report exists |
| `Backup-GPO -Name "<gpo-name>" -Path "<backup-path>"` | Backs up configured GPO | Backup completes |
| `repadmin /replsummary` | Checks AD replication | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks SYSVOL and DC health | Tests pass |

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely roll back advanced audit, event log, and WEF GPO policy from pilot scope.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoNames = @(
  "CORP-Workstation-Advanced-Audit-Policy",
  "CORP-Workstation-Event-Forwarding",
  "CORP-Workstation-Event-Log-Settings"
)

$ReportPath = "C:\GPOPrep\Reports\Audit-WEF-Rollback"
$BackupPath = "C:\GPOPrep\Backup\Audit-WEF-Rollback"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture current state before rollback.
foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-before-rollback.html"

        Backup-GPO `
          -Name $GpoName `
          -Path $BackupPath `
          -Comment "Before advanced audit and WEF rollback"
    }
}

Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-audit-wef-rollback.txt"

# Rollback option 1:
# Disable GPO links from pilot OU.
foreach ($GpoName in $GpoNames) {
    Set-GPLink `
      -Name $GpoName `
      -Target $PilotComputerOU `
      -LinkEnabled No `
      -ErrorAction SilentlyContinue
}

# Rollback option 2:
# Use GPMC to set configured settings back to Not Configured.
gpmc.msc

# GUI rollback workflow:
# 1. Edit Advanced Audit GPO.
# 2. Set configured audit subcategories back to Not Configured if required.
# 3. Edit Event Forwarding GPO.
# 4. Set Configure target Subscription Manager to Not Configured.
# 5. Edit Event Log GPO.
# 6. Set event log size and retention settings to Not Configured if required.
# 7. Export reports.
# 8. Run gpupdate /force on pilot client.
#
# Rollback option 3:
# Remove WEF subscription from collector if lab-only.
# Run on collector:
# wecutil ds "CORP-Workstation-Security-Baseline-Events"
#
# Rollback option 4:
# Remove pilot client from WEF source group if lab-only.
# Remove-ADGroupMember -Identity "GG_WEF_Event_Sources" -Members "WIN11-01$"

# Capture state after rollback.
Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-audit-wef-rollback.txt"

foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-after-rollback.html"
    }
}

Write-Host "Advanced audit and WEF rollback workflow complete."
Write-Host "On pilot client: run gpupdate /force and validate auditpol plus subscription manager registry."
```

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Audit GPO does not apply | Computer outside pilot OU or filtering blocks it | `gpresult /scope computer /r`; `Get-GPInheritance` | Move computer or fix filtering |
| `auditpol` does not show expected settings | Advanced audit policy not configured or legacy policy conflict | `auditpol /get /category:*`; GPO report | Enable force advanced audit policy and configure subcategories |
| Security log fills too fast | Too many audit categories or log size too small | `wevtutil gl Security` | Increase log size or reduce noisy subcategories |
| Process command line missing | Command line process creation policy not enabled | GPO report and event 4688 | Enable command line in process creation events |
| Event ID 4688 missing | Process Creation audit not enabled | `auditpol /get /subcategory:"Process Creation"` | Enable Audit Process Creation success |
| WEF client has no subscription manager | GPO not applied or wrong registry path | SubscriptionManager registry check | Fix WEF GPO scope and setting |
| Client cannot reach collector | DNS, firewall, WinRM, or network issue | `Resolve-DnsName`; `Test-NetConnection <collector> -Port 5985` | Fix name resolution, firewall, or WinRM |
| Collector has no subscriptions | Subscription not created or WEC not initialized | `wecutil es` | Run `wecutil qc` and create subscription |
| WEC service stopped | Collector service not running | `Get-Service Wecsvc` | Start and set WEC service startup |
| WinRM service stopped | WinRM not configured on source or collector | `Get-Service WinRM`; `winrm quickconfig` | Enable WinRM where required |
| Forwarded Events log empty | Source not in subscription, no matching events, or client has not refreshed | `wecutil gr "<subscription>"`; source log checks | Add source group, refresh policy, generate test events |
| Source not listed in runtime status | Source group token stale or subscription source list wrong | `wecutil gr`; source group membership | Reboot source and verify group assignment |
| Forwarding events show access denied | Collector subscription permissions or source group issue | Forwarding and EventCollector logs | Fix subscription source computer group |
| Events collected but not expected IDs | Subscription query too narrow or audit policy not generating events | Subscription query and local Security log | Fix audit policy or query |
| Too many forwarded events | Subscription query too broad | ForwardedEvents volume | Narrow query or split subscriptions |
| Collector disk usage grows quickly | Forwarded Events log too large or subscription too noisy | `wevtutil gl ForwardedEvents` | Tune event query and log retention |
| Remote event forwarding works for some clients only | OU, GPO scope, group membership, or network segmentation differs | Compare `gpresult /h` and network tests | Normalize scope and firewall rules |
| Group membership change not reflected | Computer token stale | `gpresult /scope computer /r` | Reboot source computer |
| GPO report missing settings | Wrong GPO edited or stale report | `Get-GPOReport` | Edit correct GPO and export fresh report |
| Client points to old collector | Old Subscription Manager GPO still winning | `gpresult /h`; registry check | Remove old policy or fix precedence |
| Source-initiated subscription not supported by current setup | Subscription type mismatch | Subscription properties | Use source-initiated subscription with source group |
| Collector-initiated subscription fails | Firewall, RPC, permissions, or offline source | Collector logs and network tests | Prefer source-initiated for scalable endpoint collection |
| AD replication delay | GPO, group, or source membership not replicated | `repadmin /replsummary` | Wait or fix replication |
| SYSVOL issue | GPO files inconsistent across DCs | `dcdiag /test:sysvolcheck` | Fix SYSVOL/DFSR replication |
| Rollback does not stop forwarding | Client has not refreshed or another GPO configures Subscription Manager | `gpupdate /force`; `gpresult /h` | Refresh client and remove winning policy |

# 23_Configure_Advanced_Audit_Policy_And_Event_Forwarding_GPOs_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before audit and WEF rollout |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which computers receive audit and WEF policy |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Provides computer-side policy foundation |
| `11_Configure_Security_Baseline_GPO_Settings.md` | Includes security options that pair with advanced auditing |
| `12_Configure_Windows_Update_And_Defender_GPO_Settings.md` | Generates endpoint security events that can be forwarded |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md` | Ensures remote management and firewall settings do not block WEF |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Provides GPO backup and reporting workflow |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Validates audit and WEF GPO application |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Diagnoses scope, filtering, AD replication, and SYSVOL failures |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md` | Ensures Event Forwarding, WinRM, and Event Log ADMX settings are available |
| `19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs.md` | Generates AppLocker and CodeIntegrity events that can be forwarded |
| `20_Configure_LAPS_And_Local_Administrator_Password_Policy.md` | Related admin credential activity should be audited |
| `21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings.md` | Certificate client events can be collected through WEF |
| `22_Configure_BitLocker_GPO_Settings.md` | BitLocker and recovery-related endpoint events can be audited and forwarded |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms domain infrastructure before audit and WEF rollout |
| `24_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays.md` | Next workbook for diagnosing slow Group Policy processing and client-side extension delays |