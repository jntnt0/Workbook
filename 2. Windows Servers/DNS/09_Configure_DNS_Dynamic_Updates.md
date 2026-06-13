09_Configure_DNS_Dynamic_Updates.md
# Configure_DNS_Dynamic_Updates

# Configure_DNS_Dynamic_Updates_Index
09_Configure_DNS_Dynamic_Updates.md
Configure_DNS_Dynamic_Updates
Configure_DNS_Dynamic_Updates_Source_Basis
Configure_DNS_Dynamic_Updates_Mental_Model
Configure_DNS_Dynamic_Updates_Planning_Table
Configure_DNS_Dynamic_Updates_Configuration_Checklist
Configure_DNS_Dynamic_Updates_Precheck_Skeleton
Configure_DNS_Dynamic_Updates_Zone_Settings_Skeleton
Configure_DNS_Dynamic_Updates_Client_Registration_Skeleton
Configure_DNS_Dynamic_Updates_DHCP_DNS_Update_Skeleton
Configure_DNS_Dynamic_Updates_Record_Validation_Skeleton
Configure_DNS_Dynamic_Updates_Aging_And_Scavenging_Check_Skeleton
Configure_DNS_Dynamic_Updates_Event_Log_Skeleton
Configure_DNS_Dynamic_Updates_Verification_Commands
Configure_DNS_Dynamic_Updates_Rollback
Configure_DNS_Dynamic_Updates_Failure_Checks
Configure_DNS_Dynamic_Updates_Related_Labs

# Configure_DNS_Dynamic_Updates_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | Managing DNS zones, records, aging, and dynamic update behavior |
| Microsoft Learn | Set-DnsServerPrimaryZone | Configuring dynamic update mode on primary and AD-integrated zones |
| Microsoft Learn | Get-DnsServerZone | Validating DNS zone type, replication scope, and dynamic update setting |
| Microsoft Learn | Get-DnsServerResourceRecord | Validating dynamically registered A and PTR records |
| Microsoft Learn | Register-DnsClient | Forcing Windows client DNS registration |
| Microsoft Learn | ipconfig /registerdns | Forcing client or DC DNS registration |
| Microsoft Learn | Set-DhcpServerv4DnsSetting | Configuring DHCP-driven DNS registration behavior |
| Microsoft Learn | Get-DhcpServerv4DnsSetting | Validating DHCP DNS update behavior |
| Microsoft Learn | Set-DhcpServerDnsCredential | Configuring DHCP credentials for secure DNS updates |
| Microsoft Learn | Get-DnsServerScavenging | Reviewing DNS aging and scavenging settings |
| Windows Server operational practice | Secure dynamic updates, DHCP A/PTR ownership, stale record control | Ensuring domain clients and DHCP clients register accurate DNS records |

# Configure_DNS_Dynamic_Updates_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Dynamic DNS update | Automatic DNS record registration or modification by a client or DHCP server |
| Secure dynamic update | AD-integrated DNS update mode that allows authenticated updates only |
| Nonsecure dynamic update | Allows unauthenticated updates and is usually avoided in AD environments |
| A record | Name-to-IPv4 record dynamically registered by client or DHCP |
| PTR record | IPv4-to-name reverse lookup record dynamically registered by client or DHCP |
| Record ownership | Security descriptor on a DNS record that controls who can update it later |
| DHCP DNS update | DHCP server registers A/PTR records for DHCP clients |
| DHCP DNS credentials | Dedicated domain account used by DHCP servers to securely update DNS records |
| Client self-registration | Domain client registers its own A record through DNS client service |
| Reverse zone dependency | PTR updates require an existing reverse lookup zone |
| Aging | DNS record timestamping used to identify stale records |
| Scavenging | Cleanup process that removes stale dynamically registered records |
| AD-integrated zone dependency | Secure dynamic updates require an AD-integrated zone |
| First rule | In AD, use secure dynamic updates, make clients use AD DNS, configure DHCP update behavior deliberately, and validate both A and PTR records |

# Configure_DNS_Dynamic_Updates_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Forward lookup zone | `corp.local` | `<forward-zone-name>` |
| Reverse lookup zone | `10.10.10.in-addr.arpa` | `<reverse-zone-name>` |
| DNS server 1 | `DC1.corp.local` | `<dns-server-1>` |
| DNS server 1 IP | `10.10.10.10` | `<dns-server-1-ip>` |
| DNS server 2 | `DC2.corp.local` | `<dns-server-2>` |
| DNS server 2 IP | `10.10.10.11` | `<dns-server-2-ip>` |
| DHCP server | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.20` | `<dhcp-server-ip>` |
| DHCP scope ID | `10.10.10.0` | `<scope-id>` |
| Dynamic update mode | Secure only | `<dynamic-update-mode>` |
| DHCP updates A/PTR records | Yes | `<yes-no>` |
| DHCP discards records on lease deletion | Yes | `<yes-no>` |
| DHCP updates records for clients that do not request updates | Yes | `<yes-no>` |
| DHCP DNS credential account | `CORP\svc-dhcpdns` | `<dhcp-dns-credential-account>` |
| Test client | `WIN11-01.corp.local` | `<client-fqdn>` |
| Test client IP | `10.10.10.55` | `<client-ip>` |
| Aging enabled | Lab optional / production change-controlled | `<yes-no>` |
| No-refresh interval | `7 days` | `<no-refresh-interval>` |
| Refresh interval | `7 days` | `<refresh-interval>` |
| Evidence path on DNS server | `C:\DNS-Dynamic-Updates` | `<dns-evidence-path>` |
| Evidence path on client | `C:\DNS-Client-Dynamic-Update` | `<client-evidence-path>` |
| Rollback stance | Restore prior zone update mode and DHCP DNS settings if behavior breaks | `<rollback-plan>` |
| Next workbook | `10_Troubleshoot_DNS_Dynamic_Update_Failures.md` | `<next-task>` |

# Configure_DNS_Dynamic_Updates_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DNS Server / DC | `whoami /groups` | Admin context is visible |
| 2 | Confirm DNS role installed | DNS Server / DC | `Get-WindowsFeature DNS` | DNS role is installed |
| 3 | Confirm DNS service running | DNS Server / DC | `Get-Service DNS` | DNS service is Running |
| 4 | Import DNS module | DNS Server / DC | `Import-Module DnsServer` | DNS module imports |
| 5 | Confirm forward zone exists | DNS Server / DC | `Get-DnsServerZone -Name "<forward-zone-name>"` | Forward zone exists |
| 6 | Confirm reverse zone exists | DNS Server / DC | `Get-DnsServerZone -Name "<reverse-zone-name>"` | Reverse zone exists |
| 7 | Confirm zone is AD-integrated | DNS Server / DC | `Get-DnsServerZone -Name "<zone-name>" \| Select-Object IsDsIntegrated` | Zone is AD-integrated |
| 8 | Capture current dynamic update mode | DNS Server / DC | `Get-DnsServerZone -Name "<zone-name>" \| Select-Object DynamicUpdate` | Existing setting is documented |
| 9 | Set forward zone secure updates | DNS Server / DC | `Set-DnsServerPrimaryZone -Name "<forward-zone-name>" -DynamicUpdate Secure` | Forward zone allows secure dynamic updates |
| 10 | Set reverse zone secure updates | DNS Server / DC | `Set-DnsServerPrimaryZone -Name "<reverse-zone-name>" -DynamicUpdate Secure` | Reverse zone allows secure dynamic updates |
| 11 | Confirm client uses AD DNS | Client | `ipconfig /all`; `Get-DnsClientServerAddress -AddressFamily IPv4` | Client points to AD DNS servers |
| 12 | Force client registration | Client | `Register-DnsClient`; `ipconfig /registerdns` | Client attempts DNS registration |
| 13 | Validate client A record | DNS Server / Client | `Resolve-DnsName "<client-fqdn>" -Server "<dns-server-ip>"` | Client name resolves to expected IP |
| 14 | Validate client PTR record | DNS Server / Client | `Resolve-DnsName "<client-ip>" -Type PTR -Server "<dns-server-ip>"` | Client IP resolves to expected name |
| 15 | Confirm DHCP DNS settings | DHCP Server | `Get-DhcpServerv4DnsSetting` | DHCP DNS update behavior is visible |
| 16 | Configure DHCP DNS update behavior | DHCP Server | `Set-DhcpServerv4DnsSetting -DynamicUpdates Always -DeleteDnsRROnLeaseExpiry $true -UpdateDnsRRForOlderClients $true` | DHCP is configured to update DNS records |
| 17 | Configure DHCP DNS credential | DHCP Server | `Set-DhcpServerDnsCredential` | DHCP server uses dedicated credentials for secure updates |
| 18 | Restart DHCP service if needed | DHCP Server | `Restart-Service DHCPServer` | DHCP service reloads settings |
| 19 | Renew DHCP client lease | Client | `ipconfig /release`; `ipconfig /renew` | Client receives lease and triggers DNS update path |
| 20 | Validate lease and DNS record | DHCP Server / DNS Server | `Get-DhcpServerv4Lease`; `Get-DnsServerResourceRecord` | Lease and DNS record agree |
| 21 | Review record timestamp | DNS Server / DC | `Get-DnsServerResourceRecord -ZoneName "<zone>" -Name "<client>"` | Dynamic record timestamp is visible |
| 22 | Review aging/scavenging stance | DNS Server / DC | `Get-DnsServerScavenging`; `Get-DnsServerZoneAging` | Stale record cleanup settings are known |
| 23 | Review DNS logs | DNS Server / DC | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | DNS update events are visible |
| 24 | Review client DNS logs | Client | `Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100` | Client DNS events are visible |
| 25 | Export evidence | DNS / DHCP / Client | Run validation skeletons | Evidence files are saved |
| 26 | Document final state | Operator | `Record zone update mode, DHCP DNS settings, credential account, A/PTR validation, aging stance, and event status` | Dynamic update record is complete |

# Configure_DNS_Dynamic_Updates_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: capture current DNS zone, record, and dynamic update state before changes.

$DomainFqdn = "corp.local"
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$DnsServerIp = "10.10.10.10"
$TestClientName = "WIN11-01"
$TestClientFqdn = "$TestClientName.$DomainFqdn"
$TestClientIp = "10.10.10.55"
$EvidencePath = "C:\DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Confirm DNS role and service.
Get-WindowsFeature DNS |
  Tee-Object "$EvidencePath\dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-state.txt"

# Import DNS module.
Import-Module DnsServer

# Capture DNS zones.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-before.txt"

# Capture forward zone properties.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\forward-zone-properties-before.txt"

# Capture reverse zone properties if present.
Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\reverse-zone-properties-before.txt"

# Capture existing test client records if present.
Get-DnsServerResourceRecord -ZoneName $ForwardZone -Name $TestClientName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\test-client-a-record-before.txt"

Resolve-DnsName $TestClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-test-client-before.txt"

Resolve-DnsName $TestClientIp -Type PTR -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-test-client-ptr-before.txt"

# Capture aging/scavenging baseline.
Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\dns-server-scavenging-before.txt"

Get-DnsServerZoneAging -Name $ForwardZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-aging-before.txt"

Get-DnsServerZoneAging -Name $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-aging-before.txt"
```

# Configure_DNS_Dynamic_Updates_Zone_Settings_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: configure AD-integrated DNS zones for secure dynamic updates.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dns-zone-dynamic-update-config-transcript.txt"

Import-Module DnsServer

# Capture current forward zone update setting.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\forward-zone-dynamic-update-before.txt"

# Configure secure dynamic updates on forward zone.
Set-DnsServerPrimaryZone `
  -Name $ForwardZone `
  -DynamicUpdate Secure

# Capture current reverse zone update setting if present.
Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-dynamic-update-before.txt"

# Configure secure dynamic updates on reverse zone if present.
Set-DnsServerPrimaryZone `
  -Name $ReverseZone `
  -DynamicUpdate Secure `
  -ErrorAction SilentlyContinue

# Confirm final settings.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\forward-zone-dynamic-update-after.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-dynamic-update-after.txt"

Stop-Transcript
```

# Configure_DNS_Dynamic_Updates_Client_Registration_Skeleton
```powershell
# Run in elevated PowerShell on a domain client.
# Purpose: force and validate client-side dynamic DNS registration.

$DomainFqdn = "corp.local"
$DnsServerIp = "10.10.10.10"
$ClientEvidencePath = "C:\DNS-Client-Dynamic-Update"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Capture client identity.
hostname |
  Tee-Object "$ClientEvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$ClientEvidencePath\computer-domain-state.txt"

# Capture network and DNS state.
Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration-before.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\dns-client-server-addresses-before.txt"

Get-DnsClientGlobalSetting |
  Tee-Object "$ClientEvidencePath\dns-client-global-setting-before.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-before.txt"

# Clear DNS cache.
Clear-DnsClientCache

# Force dynamic DNS registration.
Register-DnsClient

ipconfig /registerdns |
  Tee-Object "$ClientEvidencePath\ipconfig-registerdns.txt"

Start-Sleep -Seconds 10

# Capture network and DNS state after registration attempt.
Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration-after.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-after.txt"

# Validate client FQDN resolution.
$ClientFqdn = "$env:COMPUTERNAME.$DomainFqdn"

Resolve-DnsName $ClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-client-fqdn-after-registerdns.txt"

# Validate domain and SRV resolution to ensure client is using AD DNS.
Resolve-DnsName $DomainFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-domain-fqdn.txt"

Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-domain-dc-locator-srv.txt"

# Review DNS client events.
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\dns-client-operational-events.txt"
```

# Configure_DNS_Dynamic_Updates_DHCP_DNS_Update_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: configure DHCP to perform DNS dynamic updates for DHCP clients.
# Recommended: use a dedicated least-privilege domain account for DHCP DNS updates.

$ScopeId = "10.10.10.0"
$DhcpEvidencePath = "C:\DHCP-DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $DhcpEvidencePath

Start-Transcript -Path "$DhcpEvidencePath\dhcp-dns-update-config-transcript.txt"

Import-Module DhcpServer

# Confirm DHCP service.
Get-Service DHCPServer |
  Tee-Object "$DhcpEvidencePath\dhcp-service-state.txt"

# Capture existing DHCP DNS settings.
Get-DhcpServerv4DnsSetting |
  Tee-Object "$DhcpEvidencePath\dhcp-server-dns-settings-before.txt"

Get-DhcpServerv4DnsSetting -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$DhcpEvidencePath\dhcp-scope-dns-settings-before.txt"

# Configure DHCP server-level DNS dynamic update behavior.
# DynamicUpdates Always = DHCP server always dynamically updates DNS A and PTR records.
# DeleteDnsRROnLeaseExpiry = remove DNS records when lease expires or is deleted.
# UpdateDnsRRForOlderClients = update DNS for clients that do not request updates.
Set-DhcpServerv4DnsSetting `
  -DynamicUpdates Always `
  -DeleteDnsRROnLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true

# Optional: configure scope-level DNS dynamic update behavior.
Set-DhcpServerv4DnsSetting `
  -ScopeId $ScopeId `
  -DynamicUpdates Always `
  -DeleteDnsRROnLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true

# Configure DHCP DNS credentials.
# This prompts for credentials.
# Use a dedicated account such as CORP\svc-dhcpdns.
Set-DhcpServerDnsCredential

# Restart DHCP service so settings are fully applied.
Restart-Service DHCPServer

# Confirm settings after change.
Get-DhcpServerv4DnsSetting |
  Tee-Object "$DhcpEvidencePath\dhcp-server-dns-settings-after.txt"

Get-DhcpServerv4DnsSetting -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$DhcpEvidencePath\dhcp-scope-dns-settings-after.txt"

Get-Service DHCPServer |
  Tee-Object "$DhcpEvidencePath\dhcp-service-after-dns-update-config.txt"

Stop-Transcript
```

# Configure_DNS_Dynamic_Updates_Record_Validation_Skeleton
```powershell
# Run in elevated PowerShell on DNS server/domain controller after client renew/register.
# Purpose: validate dynamically registered A and PTR records.

$DomainFqdn = "corp.local"
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$DnsServerIp = "10.10.10.10"
$TestClientName = "WIN11-01"
$TestClientFqdn = "$TestClientName.$DomainFqdn"
$TestClientIp = "10.10.10.55"
$EvidencePath = "C:\DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Query A record directly from zone.
Get-DnsServerResourceRecord `
  -ZoneName $ForwardZone `
  -Name $TestClientName `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\test-client-a-record-after-update.txt"

# Resolve client FQDN.
Resolve-DnsName $TestClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-test-client-fqdn-after-update.txt"

# Query reverse zone records.
Get-DnsServerResourceRecord `
  -ZoneName $ReverseZone `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-records-after-update.txt"

# Resolve PTR.
Resolve-DnsName $TestClientIp -Type PTR -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-test-client-ptr-after-update.txt"

# Show record timestamp and aging-related fields for target record.
Get-DnsServerResourceRecord `
  -ZoneName $ForwardZone `
  -Name $TestClientName `
  -ErrorAction SilentlyContinue |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\test-client-a-record-timestamp.txt"

# Export full zone record inventory for evidence.
Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Export-Csv "$EvidencePath\forward-zone-records-after-dynamic-update.csv" -NoTypeInformation

Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\reverse-zone-records-after-dynamic-update.csv" -NoTypeInformation
```

# Configure_DNS_Dynamic_Updates_Aging_And_Scavenging_Check_Skeleton
```powershell
# Run in elevated PowerShell on DNS server/domain controller.
# Purpose: inspect DNS aging and scavenging settings.
# Do not enable scavenging blindly in production. Validate stale record risk and intervals first.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Capture server scavenging settings.
Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\dns-server-scavenging-current.txt"

# Capture zone aging settings.
Get-DnsServerZoneAging -Name $ForwardZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-aging-current.txt"

Get-DnsServerZoneAging -Name $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-aging-current.txt"

# Optional examples for lab only.
# Enable aging on forward zone.
# Set-DnsServerZoneAging `
#   -Name $ForwardZone `
#   -Aging $true `
#   -NoRefreshInterval 7.00:00:00 `
#   -RefreshInterval 7.00:00:00

# Enable aging on reverse zone.
# Set-DnsServerZoneAging `
#   -Name $ReverseZone `
#   -Aging $true `
#   -NoRefreshInterval 7.00:00:00 `
#   -RefreshInterval 7.00:00:00

# Enable server scavenging in a controlled lab only.
# Set-DnsServerScavenging `
#   -ScavengingState $true `
#   -ScavengingInterval 7.00:00:00

# Confirm records with timestamps.
Get-DnsServerResourceRecord -ZoneName $ForwardZone |
  Where-Object {$_.Timestamp -ne $null} |
  Select-Object HostName,RecordType,Timestamp,RecordData |
  Tee-Object "$EvidencePath\forward-zone-timestamped-records.txt"
```

# Configure_DNS_Dynamic_Updates_Event_Log_Skeleton
```powershell
# Run on DNS server, DHCP server, and client as applicable.
# Purpose: collect event logs related to DNS dynamic updates.

$EvidencePath = "C:\DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events.txt"

# DNS Server operational events if available.
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events.txt"

# DHCP Server operational events if DHCP role exists.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-operational-events.txt"

# DHCP Server admin events if present.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-admin-events.txt"

# DNS Client operational events if run on client.
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-client-operational-events.txt"

# System DNS, DHCP, and Netlogon related events.
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.ProviderName -like "*DHCP*" -or
    $_.ProviderName -like "*Netlogon*" -or
    $_.Message -like "*dynamic update*" -or
    $_.Message -like "*DNS*"
  } |
  Tee-Object "$EvidencePath\system-dns-dhcp-netlogon-events.txt"

# Final service states if available.
Get-Service DNS,DHCPServer,Dhcp,Netlogon -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\final-service-state.txt"
```

# Configure_DNS_Dynamic_Updates_Verification_Commands
```powershell
# DNS role and service.
Get-WindowsFeature DNS
Get-Service DNS

# Import DNS module.
Import-Module DnsServer

# Confirm zones.
Get-DnsServerZone
Get-DnsServerZone -Name "<forward-zone-name>"
Get-DnsServerZone -Name "<reverse-zone-name>"

# Confirm AD-integrated secure dynamic update settings.
Get-DnsServerZone -Name "<forward-zone-name>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

Get-DnsServerZone -Name "<reverse-zone-name>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Configure secure dynamic updates.
Set-DnsServerPrimaryZone -Name "<forward-zone-name>" -DynamicUpdate Secure
Set-DnsServerPrimaryZone -Name "<reverse-zone-name>" -DynamicUpdate Secure

# Client DNS registration.
ipconfig /all
Get-DnsClientServerAddress -AddressFamily IPv4
Clear-DnsClientCache
Register-DnsClient
ipconfig /registerdns

# Validate A record.
Resolve-DnsName "<client-fqdn>" -Server "<dns-server-ip>"
Get-DnsServerResourceRecord -ZoneName "<forward-zone-name>" -Name "<client-hostname>"

# Validate PTR record.
Resolve-DnsName "<client-ip>" -Type PTR -Server "<dns-server-ip>"
Get-DnsServerResourceRecord -ZoneName "<reverse-zone-name>"

# DHCP DNS dynamic update settings.
Import-Module DhcpServer
Get-DhcpServerv4DnsSetting
Get-DhcpServerv4DnsSetting -ScopeId "<scope-id>"

# Configure DHCP DNS update behavior.
Set-DhcpServerv4DnsSetting `
  -DynamicUpdates Always `
  -DeleteDnsRROnLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true

Set-DhcpServerv4DnsSetting `
  -ScopeId "<scope-id>" `
  -DynamicUpdates Always `
  -DeleteDnsRROnLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true

# Configure DHCP DNS credentials.
Set-DhcpServerDnsCredential

# Restart DHCP service if settings changed.
Restart-Service DHCPServer
Get-Service DHCPServer

# Renew DHCP client lease.
ipconfig /release
ipconfig /renew
ipconfig /all

# Validate DHCP lease and DNS agreement.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" | Where-Object {$_.HostName -like "*<client-name>*"}

# Aging and scavenging checks.
Get-DnsServerScavenging
Get-DnsServerZoneAging -Name "<forward-zone-name>"
Get-DnsServerZoneAging -Name "<reverse-zone-name>"

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
```

# Configure_DNS_Dynamic_Updates_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Forward zone update mode changed incorrectly | `Set-DnsServerPrimaryZone -Name "<forward-zone-name>" -DynamicUpdate Secure` | Wrong update mode can block client registration or allow unsafe updates |
| Reverse zone update mode changed incorrectly | `Set-DnsServerPrimaryZone -Name "<reverse-zone-name>" -DynamicUpdate Secure` | PTR updates may fail or become unsafe |
| DHCP DNS update behavior changed incorrectly | Reapply previous `Set-DhcpServerv4DnsSetting` values | DHCP clients may stop registering A/PTR records correctly |
| DHCP DNS credential set incorrectly | Re-run `Set-DhcpServerDnsCredential` with correct dedicated account | Wrong credentials can cause secure update failures |
| DHCP service restarted | No rollback normally required | Brief lease service interruption |
| Client registration forced | No rollback normally required | Client A/PTR records may update |
| Aging enabled accidentally | `Set-DnsServerZoneAging -Name "<zone-name>" -Aging $false` | Stale records may remain if disabled |
| Scavenging enabled accidentally | `Set-DnsServerScavenging -ScavengingState $false` | Disables automatic stale record cleanup |
| Incorrect dynamic record created | Remove incorrect record with `Remove-DnsServerResourceRecord` after confirming ownership and impact | Removing wrong record breaks name resolution |
| Evidence folder created | `Remove-Item C:\DNS-Dynamic-Updates -Recurse -Force` | Deletes validation evidence |

# Configure_DNS_Dynamic_Updates_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client A record does not appear | Client DNS setting, zone update mode, or permissions | `ipconfig /all`; zone DynamicUpdate; DNS client events | Rebuilding DNS |
| Client PTR record does not appear | Missing reverse zone or DHCP/client update setting | `Get-DnsServerZone`; reverse zone check | Editing forward zone only |
| Secure updates fail | Record ownership or credential issue | DNS event logs and DHCP DNS credential | Setting zone to nonsecure updates |
| DHCP clients do not register DNS | DHCP DNS update settings | `Get-DhcpServerv4DnsSetting` | Recreating DHCP scope |
| DHCP-created records cannot update later | Record ownership conflict | Check record security owner and DHCP credential use | Deleting all records blindly |
| Client uses public DNS | DHCP option 006 or static client DNS wrong | `ipconfig /all`; `Get-DnsClientServerAddress` | Troubleshooting DNS server first |
| Dynamic update works for A but not PTR | Reverse zone missing or not secure-update enabled | Reverse zone properties | Rejoining client |
| Old DNS records remain | Aging/scavenging disabled or lease cleanup disabled | `Get-DnsServerScavenging`; DHCP delete-on-expiry setting | Manually deleting random records |
| Duplicate A records exist | Reused hostname, stale record, or multiple NICs | DNS records and client hostname/IP inventory | Flushing DNS cache only |
| Client resolves old IP | Stale record or caching | `Resolve-DnsName`; record timestamp; DNS cache | Changing DHCP lease duration |
| Dynamic update denied | ACL/ownership issue on record | Record permissions and DHCP credential | Restarting DNS repeatedly |
| Reverse lookup fails | No PTR or wrong reverse zone | `Resolve-DnsName <ip> -Type PTR` | Troubleshooting Kerberos first |
| DHCP server logs DNS update errors | Credential, zone security, DNS reachability | DHCP operational log and DNS server log | Removing authorization |
| DNS event log shows update refusal | Zone mode, permissions, or record ownership | Zone DynamicUpdate and record owner | Changing gateway |
| Stale records deleted unexpectedly | Scavenging intervals too aggressive | Aging/scavenging settings and timestamps | Disabling DNS service |
| Domain join fails after DNS changes | Client cannot locate AD DNS/SRV | `_ldap._tcp.dc._msdcs` lookup and client DNS settings | Blaming DHCP reservation |
| DC records affected | Netlogon registration or zone issue | `dcdiag /test:dns`; `nltest /dsregdns` | Manually recreating all SRV records |

# Configure_DNS_Dynamic_Updates_Related_Labs
| Lab                                          | Related Workbook                                            | Skill Proven                                                   |
| -------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------- |
| Configure AD-integrated DNS zones            | `07_Configure_AD_Integrated_DNS_Zones.md`                   | AD-integrated forward/reverse zone baseline                    |
| Validate AD DNS SRV records and DC locator   | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md`          | DC locator and AD DNS validation                               |
| Configure DNS dynamic updates                | `09_Configure_DNS_Dynamic_Updates.md`                       | Secure dynamic updates, client registration, DHCP DNS behavior |
| Validate DHCP client lease assignment        | `06_Validate_DHCP_Client_Lease_Assignment.md`               | Client lease and DNS option validation                         |
| Configure DHCPv4 scope options               | `04_Configure_DHCPv4_Scope_Options.md`                      | DNS server and suffix delivery through DHCP                    |
| Configure DHCPv4 exclusions and reservations | `05_Configure_DHCPv4_Exclusions_And_Reservations.md`        | Predictable client identity and lease behavior                 |
| Troubleshoot failed domain join              | `10_Troubleshoot_Failed_Domain_Join.md`                     | DNS and DC locator failure triage                              |
| Monitor DNS records and events               | `11_Monitor_DNS_Records_Events_And_Stale_Record_Cleanup.md` | Operational DNS visibility and stale record control            |