10_Configure_DNS_Aging_And_Scavenging.md
# Configure_DNS_Aging_And_Scavenging

# Configure_DNS_Aging_And_Scavenging_Index
10_Configure_DNS_Aging_And_Scavenging.md
Configure_DNS_Aging_And_Scavenging
Configure_DNS_Aging_And_Scavenging_Source_Basis
Configure_DNS_Aging_And_Scavenging_Mental_Model
Configure_DNS_Aging_And_Scavenging_Planning_Table
Configure_DNS_Aging_And_Scavenging_Configuration_Checklist
Configure_DNS_Aging_And_Scavenging_Precheck_Skeleton
Configure_DNS_Aging_And_Scavenging_Record_Timestamp_Audit_Skeleton
Configure_DNS_Aging_And_Scavenging_Zone_Aging_Config_Skeleton
Configure_DNS_Aging_And_Scavenging_Server_Scavenging_Config_Skeleton
Configure_DNS_Aging_And_Scavenging_Scavenging_Server_Selection_Skeleton
Configure_DNS_Aging_And_Scavenging_Validation_Skeleton
Configure_DNS_Aging_And_Scavenging_Event_Log_Skeleton
Configure_DNS_Aging_And_Scavenging_Verification_Commands
Configure_DNS_Aging_And_Scavenging_Rollback
Configure_DNS_Aging_And_Scavenging_Failure_Checks
Configure_DNS_Aging_And_Scavenging_Related_Labs

# Configure_DNS_Aging_And_Scavenging_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | Managing DNS zones, records, timestamps, aging, and scavenging |
| Microsoft Learn | Get-DnsServerZone | Reviewing DNS zone properties |
| Microsoft Learn | Get-DnsServerResourceRecord | Auditing record timestamps and identifying static versus dynamic records |
| Microsoft Learn | Get-DnsServerZoneAging | Reviewing zone-level aging settings |
| Microsoft Learn | Set-DnsServerZoneAging | Configuring zone-level aging, no-refresh interval, and refresh interval |
| Microsoft Learn | Get-DnsServerScavenging | Reviewing DNS server scavenging settings |
| Microsoft Learn | Set-DnsServerScavenging | Configuring DNS server scavenging interval and scavenging state |
| Microsoft Learn | Clear-DnsServerCache | Clearing DNS server cache during controlled testing |
| Microsoft Learn | DNS Server event logs | Reviewing scavenging, update, and DNS service events |
| Windows Server operational practice | Stale record cleanup, controlled scavenging rollout, DHCP dynamic updates | Preventing stale DNS records without accidentally deleting valid records |

# Configure_DNS_Aging_And_Scavenging_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Aging | DNS record timestamp tracking used to determine whether a dynamic record is stale |
| Scavenging | DNS cleanup process that removes stale dynamically updated records |
| Dynamic record | DNS record created or refreshed by client or DHCP dynamic update |
| Static record | DNS record manually created without a timestamp; not scavenged by default |
| Timestamp | Date/time marker used by aging and scavenging to decide freshness |
| No-refresh interval | Period where timestamp refreshes are ignored to reduce AD replication churn |
| Refresh interval | Period after no-refresh where clients can refresh timestamps before record becomes stale |
| Scavenging interval | How often the DNS server checks for stale records to delete |
| Zone aging | Per-zone setting that allows timestamps to be used for stale record cleanup |
| Server scavenging | Server-level setting that actually runs cleanup against eligible zones |
| Scavenging server | DNS server selected to perform scavenging in an AD-integrated DNS environment |
| AD replication impact | DNS timestamp changes replicate through AD-integrated zones |
| Stale record risk | Old A/PTR records can point clients to wrong IPs and cause app, domain join, or access failures |
| First rule | Do not enable scavenging blindly; audit timestamps, dynamic update behavior, DHCP DNS settings, and record ownership first |

# Configure_DNS_Aging_And_Scavenging_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Forward lookup zone | `corp.local` | `<forward-zone-name>` |
| Reverse lookup zone | `10.10.10.in-addr.arpa` | `<reverse-zone-name>` |
| Primary DNS server | `DC1.corp.local` | `<dns-server-1>` |
| Secondary DNS server | `DC2.corp.local` | `<dns-server-2>` |
| Scavenging server | `DC1.corp.local` | `<scavenging-server>` |
| DNS server IP | `10.10.10.10` | `<dns-server-ip>` |
| DHCP server | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP DNS updates configured | Yes | `<yes-no>` |
| Forward zone aging enabled | Yes | `<yes-no>` |
| Reverse zone aging enabled | Yes | `<yes-no>` |
| No-refresh interval | `7 days` | `<no-refresh-interval>` |
| Refresh interval | `7 days` | `<refresh-interval>` |
| Server scavenging interval | `7 days` | `<scavenging-interval>` |
| Scavenge stale resource records | Enabled after audit | `<enabled-disabled>` |
| Excluded critical records | DC, server, app, static infrastructure | `<critical-record-list>` |
| Record audit window | Before enabling scavenging | `<audit-window>` |
| Evidence path | `C:\DNS-Aging-Scavenging` | `<evidence-path>` |
| Rollback stance | Disable scavenging first, then restore records from inventory/backup if needed | `<rollback-plan>` |
| Next workbook | `11_Monitor_DNS_Records_Events_And_Stale_Record_Cleanup.md` | `<next-task>` |

# Configure_DNS_Aging_And_Scavenging_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DNS Server / DC | `whoami /groups` | Admin context is visible |
| 2 | Confirm DNS role installed | DNS Server / DC | `Get-WindowsFeature DNS` | DNS role is installed |
| 3 | Confirm DNS service running | DNS Server / DC | `Get-Service DNS` | DNS service is Running |
| 4 | Import DNS module | DNS Server / DC | `Import-Module DnsServer` | DNS module imports |
| 5 | Confirm DNS zones | DNS Server / DC | `Get-DnsServerZone` | Forward and reverse zones are visible |
| 6 | Confirm AD-integrated zone state | DNS Server / DC | `Get-DnsServerZone -Name "<zone-name>" \| Select ZoneName,IsDsIntegrated,DynamicUpdate` | Zone state is known |
| 7 | Confirm dynamic updates are configured | DNS Server / DC | `Get-DnsServerZone -Name "<zone-name>" \| Select DynamicUpdate` | Secure dynamic updates are visible |
| 8 | Capture current zone aging settings | DNS Server / DC | `Get-DnsServerZoneAging -Name "<zone-name>"` | Current aging setting is documented |
| 9 | Capture current server scavenging settings | DNS Server / DC | `Get-DnsServerScavenging` | Current server scavenging setting is documented |
| 10 | Export current records | DNS Server / DC | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | Record inventory is saved |
| 11 | Audit timestamped records | DNS Server / DC | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" \| Where-Object {$_.Timestamp -ne $null}` | Dynamic records are identified |
| 12 | Audit static records | DNS Server / DC | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" \| Where-Object {$_.Timestamp -eq $null}` | Static records are identified |
| 13 | Confirm critical records are protected | DNS Server / DC | Review DC, server, application, and infrastructure records | Critical records are not accidentally targeted |
| 14 | Configure forward zone aging | DNS Server / DC | `Set-DnsServerZoneAging -Name "<forward-zone>" -Aging $true -NoRefreshInterval 7.00:00:00 -RefreshInterval 7.00:00:00` | Forward zone aging is enabled |
| 15 | Configure reverse zone aging | DNS Server / DC | `Set-DnsServerZoneAging -Name "<reverse-zone>" -Aging $true -NoRefreshInterval 7.00:00:00 -RefreshInterval 7.00:00:00` | Reverse zone aging is enabled |
| 16 | Confirm zone aging settings | DNS Server / DC | `Get-DnsServerZoneAging -Name "<zone-name>"` | Zone aging intervals match design |
| 17 | Configure server scavenging | DNS Server / DC | `Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00` | DNS server scavenging is enabled |
| 18 | Confirm scavenging server choice | DNS Server / DC | `Get-DnsServerScavenging` | Scavenging settings are visible |
| 19 | Force client registration for test client | Client | `Register-DnsClient`; `ipconfig /registerdns` | Dynamic record timestamp can refresh |
| 20 | Validate updated timestamp | DNS Server / DC | `Get-DnsServerResourceRecord -ZoneName "<zone>" -Name "<client-hostname>"` | Timestamp is visible for dynamic record |
| 21 | Review DNS logs | DNS Server / DC | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | DNS events are visible |
| 22 | Review DNS Server operational logs | DNS Server / DC | `Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100` | Operational events are visible if enabled |
| 23 | Export post-change evidence | DNS Server / DC | Run validation skeleton | Evidence files are saved |
| 24 | Document final state | Operator | `Record zones, intervals, scavenging server, timestamp audit, and event status` | Aging/scavenging record is complete |

# Configure_DNS_Aging_And_Scavenging_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: capture DNS role, zone, aging, scavenging, and record state before changes.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-Aging-Scavenging"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$EvidencePath\computer-domain-state.txt"

# Confirm DNS role and service.
Get-WindowsFeature DNS |
  Tee-Object "$EvidencePath\dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-state.txt"

# Import DNS module.
Import-Module DnsServer

# Capture all DNS zones.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-before.txt"

# Capture target zone properties.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\forward-zone-properties-before.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-properties-before.txt"

# Capture current zone aging settings.
Get-DnsServerZoneAging -Name $ForwardZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-aging-before.txt"

Get-DnsServerZoneAging -Name $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-aging-before.txt"

# Capture server scavenging settings.
Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\dns-server-scavenging-before.txt"

# Capture DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-before.txt"
```

# Configure_DNS_Aging_And_Scavenging_Record_Timestamp_Audit_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: audit static records, dynamic records, and timestamp behavior before enabling scavenging.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-Aging-Scavenging"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Export full forward zone record inventory.
Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Tee-Object "$EvidencePath\forward-zone-records-full.txt"

Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Export-Csv "$EvidencePath\forward-zone-records-full.csv" -NoTypeInformation

# Export full reverse zone record inventory.
Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-records-full.txt"

Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\reverse-zone-records-full.csv" -NoTypeInformation

# Identify timestamped dynamic records.
Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Where-Object {$_.Timestamp -ne $null} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\forward-zone-dynamic-timestamped-records.txt"

# Identify static records.
Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Where-Object {$_.Timestamp -eq $null} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\forward-zone-static-records.txt"

# Identify potentially stale dynamic records older than 30 days.
$Cutoff = (Get-Date).AddDays(-30)

Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Where-Object {$_.Timestamp -ne $null -and $_.Timestamp -lt $Cutoff} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\forward-zone-potentially-stale-records-older-than-30-days.txt"

# Identify dynamic reverse records.
Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Where-Object {$_.Timestamp -ne $null} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\reverse-zone-dynamic-timestamped-records.txt"

# Identify potentially stale reverse records older than 30 days.
Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Where-Object {$_.Timestamp -ne $null -and $_.Timestamp -lt $Cutoff} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\reverse-zone-potentially-stale-records-older-than-30-days.txt"
```

# Configure_DNS_Aging_And_Scavenging_Zone_Aging_Config_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: enable zone-level aging after record audit.
# Use conservative intervals unless lab requirements say otherwise.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$NoRefreshInterval = "7.00:00:00"
$RefreshInterval = "7.00:00:00"
$EvidencePath = "C:\DNS-Aging-Scavenging"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dns-zone-aging-config-transcript.txt"

Import-Module DnsServer

# Capture zone aging before.
Get-DnsServerZoneAging -Name $ForwardZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-aging-before-change.txt"

Get-DnsServerZoneAging -Name $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-aging-before-change.txt"

# Enable aging on forward zone.
Set-DnsServerZoneAging `
  -Name $ForwardZone `
  -Aging $true `
  -NoRefreshInterval $NoRefreshInterval `
  -RefreshInterval $RefreshInterval

# Enable aging on reverse zone if present.
Set-DnsServerZoneAging `
  -Name $ReverseZone `
  -Aging $true `
  -NoRefreshInterval $NoRefreshInterval `
  -RefreshInterval $RefreshInterval `
  -ErrorAction SilentlyContinue

# Confirm zone aging after.
Get-DnsServerZoneAging -Name $ForwardZone |
  Tee-Object "$EvidencePath\forward-zone-aging-after-change.txt"

Get-DnsServerZoneAging -Name $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-aging-after-change.txt"

Stop-Transcript
```

# Configure_DNS_Aging_And_Scavenging_Server_Scavenging_Config_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server chosen to perform scavenging.
# Purpose: enable server-level scavenging after zone aging is configured.

$ScavengingInterval = "7.00:00:00"
$EvidencePath = "C:\DNS-Aging-Scavenging"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dns-server-scavenging-config-transcript.txt"

Import-Module DnsServer

# Capture server scavenging before.
Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\dns-server-scavenging-before-change.txt"

# Enable DNS server scavenging.
Set-DnsServerScavenging `
  -ScavengingState $true `
  -ScavengingInterval $ScavengingInterval

# Confirm server scavenging after.
Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\dns-server-scavenging-after-change.txt"

Stop-Transcript
```

# Configure_DNS_Aging_And_Scavenging_Scavenging_Server_Selection_Skeleton
```powershell
# Run in elevated PowerShell on each DNS server/domain controller if needed.
# Purpose: avoid enabling scavenging everywhere without design.
# Recommended: select one or a controlled set of DNS servers to scavenge AD-integrated zones.

$EvidencePath = "C:\DNS-Aging-Scavenging"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer
Import-Module ActiveDirectory

# List domain controllers and DNS role assumption.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog,OperatingSystem |
  Tee-Object "$EvidencePath\domain-controllers-for-scavenging-selection.txt"

# Capture local DNS server scavenging state.
Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\local-dns-server-scavenging-state.txt"

# Capture AD replication health before relying on AD-integrated DNS scavenging behavior.
repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary-before-scavenging.txt"

repadmin /showrepl |
  Tee-Object "$EvidencePath\repadmin-showrepl-before-scavenging.txt"

# Example policy note:
# Enable scavenging only on the selected DNS server first.
# Keep other DNS servers with ScavengingState disabled unless design says otherwise.
```

# Configure_DNS_Aging_And_Scavenging_Validation_Skeleton
```powershell
# Run in elevated PowerShell on DNS server/domain controller after configuration.
# Purpose: validate zone aging, server scavenging, timestamps, and test client refresh behavior.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$TestClientName = "WIN11-01"
$DomainFqdn = "corp.local"
$TestClientFqdn = "$TestClientName.$DomainFqdn"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Aging-Scavenging"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Confirm zone aging settings.
Get-DnsServerZoneAging -Name $ForwardZone |
  Tee-Object "$EvidencePath\forward-zone-aging-validation.txt"

Get-DnsServerZoneAging -Name $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-aging-validation.txt"

# Confirm server scavenging settings.
Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\dns-server-scavenging-validation.txt"

# Confirm target client record and timestamp.
Get-DnsServerResourceRecord -ZoneName $ForwardZone -Name $TestClientName -ErrorAction SilentlyContinue |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\test-client-record-timestamp-validation.txt"

# Resolve target client.
Resolve-DnsName $TestClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-test-client-after-aging-config.txt"

# Re-audit old records.
$Cutoff = (Get-Date).AddDays(-30)

Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Where-Object {$_.Timestamp -ne $null -and $_.Timestamp -lt $Cutoff} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\forward-zone-stale-candidates-after-config.txt"

Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Where-Object {$_.Timestamp -ne $null -and $_.Timestamp -lt $Cutoff} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\reverse-zone-stale-candidates-after-config.txt"

# Final zone export.
Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Export-Csv "$EvidencePath\forward-zone-records-after-aging-config.csv" -NoTypeInformation

Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\reverse-zone-records-after-aging-config.csv" -NoTypeInformation
```

# Configure_DNS_Aging_And_Scavenging_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on DNS server/domain controller.
# Purpose: collect DNS events related to aging, scavenging, updates, and service health.

$EvidencePath = "C:\DNS-Aging-Scavenging"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server log.
Get-WinEvent -LogName "DNS Server" -MaxEvents 300 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events.txt"

# DNS Server operational log if available.
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 300 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events.txt"

# System DNS-related events.
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*DNS*" -or
    $_.Message -like "*scaveng*"
  } |
  Tee-Object "$EvidencePath\system-dns-scavenging-events.txt"

# Directory Service events because AD-integrated DNS depends on AD replication.
Get-WinEvent -LogName "Directory Service" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\directory-service-events.txt"

# Final service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-final.txt"

# Final replication check.
repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary-after-aging-scavenging-config.txt"
```

# Configure_DNS_Aging_And_Scavenging_Verification_Commands
```powershell
# Confirm DNS role and service.
Get-WindowsFeature DNS
Get-Service DNS

# Import DNS module.
Import-Module DnsServer

# Confirm zones.
Get-DnsServerZone
Get-DnsServerZone -Name "<forward-zone-name>"
Get-DnsServerZone -Name "<reverse-zone-name>"

# Confirm zone dynamic update and AD-integrated state.
Get-DnsServerZone -Name "<forward-zone-name>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Capture zone aging settings.
Get-DnsServerZoneAging -Name "<forward-zone-name>"
Get-DnsServerZoneAging -Name "<reverse-zone-name>"

# Configure zone aging.
Set-DnsServerZoneAging `
  -Name "<forward-zone-name>" `
  -Aging $true `
  -NoRefreshInterval 7.00:00:00 `
  -RefreshInterval 7.00:00:00

Set-DnsServerZoneAging `
  -Name "<reverse-zone-name>" `
  -Aging $true `
  -NoRefreshInterval 7.00:00:00 `
  -RefreshInterval 7.00:00:00

# Capture server scavenging settings.
Get-DnsServerScavenging

# Configure server scavenging.
Set-DnsServerScavenging `
  -ScavengingState $true `
  -ScavengingInterval 7.00:00:00

# Audit timestamped dynamic records.
Get-DnsServerResourceRecord -ZoneName "<forward-zone-name>" |
  Where-Object {$_.Timestamp -ne $null} |
  Select-Object HostName,RecordType,Timestamp,RecordData

# Audit static records.
Get-DnsServerResourceRecord -ZoneName "<forward-zone-name>" |
  Where-Object {$_.Timestamp -eq $null} |
  Select-Object HostName,RecordType,Timestamp,RecordData

# Find dynamic records older than 30 days.
$Cutoff = (Get-Date).AddDays(-30)

Get-DnsServerResourceRecord -ZoneName "<forward-zone-name>" |
  Where-Object {$_.Timestamp -ne $null -and $_.Timestamp -lt $Cutoff} |
  Select-Object HostName,RecordType,Timestamp,RecordData

# Client refresh.
Register-DnsClient
ipconfig /registerdns

# Validate client record.
Resolve-DnsName "<client-fqdn>" -Server "<dns-server-ip>"
Get-DnsServerResourceRecord -ZoneName "<forward-zone-name>" -Name "<client-hostname>"

# Validate reverse record.
Resolve-DnsName "<client-ip>" -Type PTR -Server "<dns-server-ip>"

# AD replication health.
repadmin /replsummary
repadmin /showrepl

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DNS*" -or $_.Message -like "*scaveng*"}
```

# Configure_DNS_Aging_And_Scavenging_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Zone aging enabled accidentally | `Set-DnsServerZoneAging -Name "<zone-name>" -Aging $false` | Stale records remain until manually cleaned |
| Server scavenging enabled accidentally | `Set-DnsServerScavenging -ScavengingState $false` | Stops server from automatically deleting stale records |
| No-refresh interval set too short | Reapply safer interval, such as `7.00:00:00` | Excessive timestamp refreshes can increase AD replication churn |
| Refresh interval set too short | Reapply safer interval, such as `7.00:00:00` | Valid records may become stale too quickly |
| Scavenging interval set too aggressive | Reapply safer interval, such as `7.00:00:00` | Stale cleanup may run too frequently |
| Valid record scavenged | Restore record from exported CSV/inventory or recreate manually | Name resolution outage until restored |
| Reverse PTR record scavenged | Recreate PTR or force DHCP/client DNS update | Reverse lookup fails until restored |
| Client record timestamp changed | No rollback normally needed | Timestamp refresh is expected behavior |
| DNS service restarted | No rollback normally needed | Brief DNS service interruption |
| Evidence folder created | `Remove-Item C:\DNS-Aging-Scavenging -Recurse -Force` | Deletes audit and rollback evidence |

# Configure_DNS_Aging_And_Scavenging_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Valid records disappear | Scavenging interval too aggressive or records had old timestamps | Event logs and exported pre-change record inventory | Rebuilding DNS |
| No records are scavenged | Zone aging disabled, server scavenging disabled, or records are static | `Get-DnsServerZoneAging`; `Get-DnsServerScavenging`; record timestamps | Deleting records manually first |
| Records have no timestamp | Static record or aging not active when created | `Get-DnsServerResourceRecord` timestamp field | Assuming scavenging is broken |
| Dynamic records do not refresh | Client registration, DHCP DNS update, or secure update issue | `Register-DnsClient`; DHCP DNS settings; DNS events | Changing scavenging interval |
| PTR records stay stale | Reverse zone aging/update issue | Reverse zone aging and DHCP DNS update settings | Editing forward zone only |
| AD replication spikes | No-refresh interval too low or many timestamp updates | AD replication logs and interval settings | Disabling DNS service |
| Some DCs show different records | AD-integrated DNS replication issue | `repadmin /replsummary`; query each DNS server | Recreating zone |
| DNS scavenging event not seen | Scavenging may not have run yet | Scavenging interval and event log | Forcing random record deletion |
| Static server record disappears | It may have been timestamped or manually deleted | Record inventory and event logs | Blaming scavenging without evidence |
| Clients resolve old IPs | Stale records, DNS cache, or scavenging not yet active | Record timestamp, DNS cache, dynamic update behavior | Rebooting domain controllers |
| DHCP-created records do not age properly | DHCP DNS settings or credential issue | `Get-DhcpServerv4DnsSetting`; DHCP DNS credential | Changing DNS zone replication scope |
| Scavenging enabled on many DCs | Poor scavenging server design | `Get-DnsServerScavenging` on each DC | Leaving all DCs scavenging by accident |
| Critical AD records look old | Do not delete blindly | Validate DC locator, Netlogon registration, and SRV records | Removing `_msdcs` records |
| Record timestamp looks wrong | Time sync issue or old dynamic record | Check DC/client time and NTP | Changing scavenging immediately |
| DNS clients fail after cleanup | Valid record was removed or stale cache remains | Restore record and flush DNS cache | Reinstalling DNS role |

# Configure_DNS_Aging_And_Scavenging_Related_Labs
| Lab                                                   | Related Workbook                                            | Skill Proven                                       |
| ----------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------- |
| Configure AD-integrated DNS zones                     | `07_Configure_AD_Integrated_DNS_Zones.md`                   | AD-integrated forward and reverse zone baseline    |
| Validate AD DNS SRV records and DC locator            | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md`          | DC locator and critical AD DNS validation          |
| Configure DNS dynamic updates                         | `09_Configure_DNS_Dynamic_Updates.md`                       | Dynamic record registration and DHCP DNS behavior  |
| Configure DNS aging and scavenging                    | `10_Configure_DNS_Aging_And_Scavenging.md`                  | Stale dynamic record cleanup and timestamp control |
| Monitor DNS records, events, and stale record cleanup | `11_Monitor_DNS_Records_Events_And_Stale_Record_Cleanup.md` | Ongoing DNS hygiene and event monitoring           |
| Troubleshoot failed domain join                       | `10_Troubleshoot_Failed_Domain_Join.md`                     | DNS and DC locator troubleshooting                 |
| Validate DHCP client lease assignment                 | `06_Validate_DHCP_Client_Lease_Assignment.md`               | DHCP lease and client DNS registration validation  |
| Configure DHCP DNS dynamic updates                    | `09_Configure_DNS_Dynamic_Updates.md`                       | DHCP-managed A/PTR update behavior                 |