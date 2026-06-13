08_Configure_DHCP_DNS_Dynamic_Updates.md
# Configure_DHCP_DNS_Dynamic_Updates

# Configure_DHCP_DNS_Dynamic_Updates_Index
08_Configure_DHCP_DNS_Dynamic_Updates.md
Configure_DHCP_DNS_Dynamic_Updates
Configure_DHCP_DNS_Dynamic_Updates_Source_Basis
Configure_DHCP_DNS_Dynamic_Updates_Mental_Model
Configure_DHCP_DNS_Dynamic_Updates_Planning_Table
Configure_DHCP_DNS_Dynamic_Updates_Configuration_Checklist
Configure_DHCP_DNS_Dynamic_Updates_Precheck_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_Server_Level_DNS_Update_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_Scope_Level_DNS_Update_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_DHCP_DNS_Credential_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_Name_Protection_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_Client_Lease_And_DNS_Registration_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_DNS_Record_Validation_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_Lease_Expiry_Cleanup_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_Event_Log_Skeleton
Configure_DHCP_DNS_Dynamic_Updates_Verification_Commands
Configure_DHCP_DNS_Dynamic_Updates_Rollback
Configure_DHCP_DNS_Dynamic_Updates_Failure_Checks
Configure_DHCP_DNS_Dynamic_Updates_Related_Labs

# Configure_DHCP_DNS_Dynamic_Updates_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DHCP Server PowerShell module | DHCP server, scope, lease, DNS update, and credential configuration |
| Microsoft Learn | Get-DhcpServerv4DnsSetting | Reviewing DHCP DNS dynamic update behavior |
| Microsoft Learn | Set-DhcpServerv4DnsSetting | Configuring DHCP DNS update behavior at server or scope level |
| Microsoft Learn | Set-DhcpServerDnsCredential | Configuring credentials DHCP uses for secure DNS updates |
| Microsoft Learn | Get-DhcpServerDnsCredential | Reviewing configured DHCP DNS update credential state |
| Microsoft Learn | Remove-DhcpServerDnsCredential | Removing DHCP DNS update credentials if rollback is required |
| Microsoft Learn | Get-DhcpServerv4Lease | Validating leases that should create DNS records |
| Microsoft Learn | Get-DnsServerResourceRecord | Validating A and PTR records created by DHCP |
| Microsoft Learn | Register-DnsClient | Forcing client DNS registration attempts |
| Microsoft Learn | DHCP Server event logs | Reviewing DHCP DNS update, lease, and cleanup events |
| Microsoft Learn | DNS Server event logs | Reviewing DNS dynamic update and secure update events |
| Windows Server operational practice | DHCP-owned A/PTR records, secure updates, credentials, name protection, lease cleanup | Ensuring DHCP leases create and clean up DNS records correctly |

# Configure_DHCP_DNS_Dynamic_Updates_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP DNS dynamic update | DHCP server creates or updates DNS A and PTR records when clients receive leases |
| A record | Forward DNS record mapping hostname to IP |
| PTR record | Reverse DNS record mapping IP to hostname |
| Secure dynamic update | DNS update allowed only to authenticated principals in AD-integrated DNS zones |
| DHCP update credential | Dedicated domain account DHCP uses to register secure DNS records |
| Server-level DNS update setting | Default DNS update behavior for the DHCP server |
| Scope-level DNS update setting | DNS update behavior applied only to one DHCP scope |
| Always update | DHCP updates DNS records for all clients |
| Update on client request | DHCP updates DNS records only when client requests DHCP-managed DNS update |
| Older client support | DHCP updates records for clients that cannot perform dynamic DNS updates |
| Delete on lease expiry | DHCP deletes A/PTR records when lease expires or is deleted |
| Name protection | DHCP helps prevent one client from overwriting another client name |
| Record ownership | Secure DNS records are owned by the creator; wrong ownership can block future updates |
| DHCP failover consideration | Failover partners should use the same DHCP DNS update credential |
| First rule | For AD DNS, use secure dynamic updates, a dedicated DHCP DNS credential, and validate both A and PTR records after lease renewal |

# Configure_DHCP_DNS_Dynamic_Updates_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.5` | `<dhcp-server-ip>` |
| DHCP partner server | `DHCP2.corp.local` | `<dhcp-failover-partner>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Forward DNS zone | `corp.local` | `<forward-zone-name>` |
| Reverse DNS zone | `10.10.10.in-addr.arpa` | `<reverse-zone-name>` |
| DNS server 1 | `DC1.corp.local` | `<dns-server-1>` |
| DNS server 1 IP | `10.10.10.10` | `<dns-server-1-ip>` |
| DNS server 2 | `DC2.corp.local` | `<dns-server-2>` |
| DNS server 2 IP | `10.10.10.11` | `<dns-server-2-ip>` |
| DHCP scope ID | `10.20.20.0` | `<scope-id>` |
| Scope name | `Workstations-VLAN20` | `<scope-name>` |
| DNS update behavior | Always update DNS records | `<dynamic-update-mode>` |
| Delete records on lease expiry | Enabled | `<yes-no>` |
| Update older clients | Enabled | `<yes-no>` |
| PTR update disabled | No | `<yes-no>` |
| Name protection | Enabled after validation | `<yes-no>` |
| DHCP DNS credential account | `corp\svc-dhcpdns` | `<dhcp-dns-credential-account>` |
| Credential location | AD service account | `<credential-location>` |
| Test client hostname | `WIN11-01` | `<test-client-hostname>` |
| Test client FQDN | `WIN11-01.corp.local` | `<test-client-fqdn>` |
| Expected client IP | `10.20.20.50` | `<expected-client-ip>` |
| Evidence path | `C:\DHCP-DNS-Dynamic-Updates` | `<evidence-path>` |
| Rollback stance | Restore previous DNS update settings and credential state if records fail | `<rollback-plan>` |
| Next workbook | `09_Configure_DHCPv4_Failover.md` | `<next-task>` |

# Configure_DHCP_DNS_Dynamic_Updates_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DHCP role installed | DHCP Server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 3 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | DHCP service is Running |
| 4 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 5 | Confirm DHCP authorization | DHCP Server | `Get-DhcpServerInDC` | DHCP server is authorized in AD |
| 6 | Confirm target scope exists | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Scope exists |
| 7 | Confirm scope options include DNS servers and suffix | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Options 006 and 015 are correct |
| 8 | Confirm forward DNS zone exists | DNS Server | `Get-DnsServerZone -Name "<forward-zone-name>"` | Forward zone exists |
| 9 | Confirm reverse DNS zone exists | DNS Server | `Get-DnsServerZone -Name "<reverse-zone-name>"` | Reverse zone exists |
| 10 | Confirm DNS zone dynamic update mode | DNS Server | `Get-DnsServerZone -Name "<zone-name>" \| Select DynamicUpdate` | Secure dynamic updates are configured |
| 11 | Capture current DHCP DNS settings | DHCP Server | `Get-DhcpServerv4DnsSetting` | Server-level DNS update state is documented |
| 12 | Capture current scope DNS settings | DHCP Server | `Get-DhcpServerv4DnsSetting -ScopeId "<scope-id>"` | Scope-level DNS update state is documented |
| 13 | Configure DHCP DNS update credential | DHCP Server | `Set-DhcpServerDnsCredential -Credential (Get-Credential)` | DHCP has secure update credential |
| 14 | Configure server-level DNS updates | DHCP Server | `Set-DhcpServerv4DnsSetting -DynamicUpdates Always -DeleteDnsRRonLeaseExpiry $true -UpdateDnsRRForOlderClients $true` | DHCP server can update and clean DNS records |
| 15 | Configure scope-level DNS updates if needed | DHCP Server | `Set-DhcpServerv4DnsSetting -ScopeId "<scope-id>" -DynamicUpdates Always -DeleteDnsRRonLeaseExpiry $true -UpdateDnsRRForOlderClients $true` | Scope DNS update behavior is set |
| 16 | Enable name protection if required | DHCP Server | `Set-DhcpServerv4DnsSetting -ScopeId "<scope-id>" -NameProtection $true` | Name protection is enabled |
| 17 | Renew test client lease | Client | `ipconfig /release`; `ipconfig /renew` | Client receives DHCP lease |
| 18 | Force client registration if needed | Client | `Register-DnsClient`; `ipconfig /registerdns` | Client attempts DNS registration |
| 19 | Validate DHCP lease | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Test client lease is visible |
| 20 | Validate A record | DNS Server | `Resolve-DnsName "<test-client-fqdn>" -Server "<dns-server-ip>"` | A record resolves to client IP |
| 21 | Validate PTR record | DNS Server | `Resolve-DnsName "<expected-client-ip>" -Type PTR -Server "<dns-server-ip>"` | PTR record resolves to client FQDN |
| 22 | Validate DNS record ownership symptom-free update | DHCP Server / DNS Server | Renew client again and recheck record | Record refresh does not fail |
| 23 | Review DHCP server events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100` | DHCP DNS update events are visible |
| 24 | Review DNS server events | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | DNS update events are visible |
| 25 | Export evidence | DHCP Server / DNS Server / Client | Run validation skeletons | Evidence files are saved |
| 26 | Document final state | Operator | `Record server settings, scope settings, credential account, A/PTR result, and event status` | Workbook record is complete |

# Configure_DHCP_DNS_Dynamic_Updates_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: capture DHCP, scope, lease, DNS update, credential, and option state before changes.

$ScopeId = "10.20.20.0"
$DomainFqdn = "corp.local"
$ForwardZone = "corp.local"
$ReverseZone = "10.20.20.in-addr.arpa"
$TestClientName = "WIN11-01"
$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Capture server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion |
  Tee-Object "$EvidencePath\computer-info.txt"

# Confirm DHCP role and service.
Get-WindowsFeature DHCP |
  Tee-Object "$EvidencePath\dhcp-role-state.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-state.txt"

# Import DHCP module.
Import-Module DhcpServer

# Confirm DHCP authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Capture target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-before.txt"

# Capture scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-options-before.txt"

# Capture server-level DNS update settings.
Get-DhcpServerv4DnsSetting |
  Tee-Object "$EvidencePath\server-level-dns-update-settings-before.txt"

# Capture scope-level DNS update settings.
Get-DhcpServerv4DnsSetting -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-level-dns-update-settings-before.txt"

# Capture DHCP DNS credential state.
Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-dns-credential-before.txt"

# Capture leases before change.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\leases-before.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\leases-before.csv" -NoTypeInformation
```

# Configure_DHCP_DNS_Dynamic_Updates_Server_Level_DNS_Update_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: configure server-level DHCP DNS dynamic update behavior.
# Server-level settings are defaults unless scope-level settings override them.

$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\server-level-dns-update-config-transcript.txt"

Import-Module DhcpServer

# Capture current server-level DNS update settings.
Get-DhcpServerv4DnsSetting |
  Tee-Object "$EvidencePath\server-level-dns-update-settings-before-config.txt"

# Configure DHCP server to perform DNS updates for clients and delete records on lease expiry.
Set-DhcpServerv4DnsSetting `
  -DynamicUpdates Always `
  -DeleteDnsRRonLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true `
  -DisableDnsPtrRRUpdate $false

# Confirm final server-level DNS update settings.
Get-DhcpServerv4DnsSetting |
  Tee-Object "$EvidencePath\server-level-dns-update-settings-after-config.txt"

Stop-Transcript
```

# Configure_DHCP_DNS_Dynamic_Updates_Scope_Level_DNS_Update_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: configure DHCP DNS dynamic update behavior for a specific scope.
# Use scope-level settings when different scopes need different DNS behavior.

$ScopeId = "10.20.20.0"
$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\scope-level-dns-update-config-transcript.txt"

Import-Module DhcpServer

# Capture current scope-level DNS update settings.
Get-DhcpServerv4DnsSetting -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-level-dns-update-settings-before-config.txt"

# Configure scope-level DNS update behavior.
Set-DhcpServerv4DnsSetting `
  -ScopeId $ScopeId `
  -DynamicUpdates Always `
  -DeleteDnsRRonLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true `
  -DisableDnsPtrRRUpdate $false

# Confirm final scope-level DNS update settings.
Get-DhcpServerv4DnsSetting -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-level-dns-update-settings-after-config.txt"

Stop-Transcript
```

# Configure_DHCP_DNS_Dynamic_Updates_DHCP_DNS_Credential_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: configure a dedicated domain credential for DHCP secure DNS updates.
# Use the same credential on failover partner DHCP servers.

$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-dns-credential-config-transcript.txt"

Import-Module DhcpServer

# Capture existing credential state.
Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-dns-credential-before-config.txt"

# Prompt for credential.
# Example account: corp\svc-dhcpdns
$Credential = Get-Credential -Message "Enter dedicated DHCP DNS update credential, example: corp\svc-dhcpdns"

# Configure DHCP DNS update credential.
Set-DhcpServerDnsCredential `
  -Credential $Credential

# Confirm credential state after configuration.
# This should show credential metadata, not the password.
Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-dns-credential-after-config.txt"

# Restart DHCP Server service during approved window if credential behavior does not take effect immediately.
# Restart-Service DHCPServer

Stop-Transcript
```

# Configure_DHCP_DNS_Dynamic_Updates_Name_Protection_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: enable DHCP name protection to reduce name squatting and record overwrite issues.
# Validate in lab first, especially in mixed client environments.

$ScopeId = "10.20.20.0"
$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\name-protection-config-transcript.txt"

Import-Module DhcpServer

# Capture current DNS setting.
Get-DhcpServerv4DnsSetting -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-dns-settings-before-name-protection.txt"

# Enable name protection for the scope.
Set-DhcpServerv4DnsSetting `
  -ScopeId $ScopeId `
  -NameProtection $true

# Confirm name protection state.
Get-DhcpServerv4DnsSetting -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-dns-settings-after-name-protection.txt"

Stop-Transcript
```

# Configure_DHCP_DNS_Dynamic_Updates_Client_Lease_And_DNS_Registration_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP client.
# Purpose: renew lease and trigger DNS registration so DHCP/DNS update behavior can be validated.

$ExpectedDomain = "corp.local"
$ExpectedDnsServer = "10.10.10.10"
$ClientEvidencePath = "C:\DHCP-DNS-Client-Validation"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Capture client identity.
hostname |
  Tee-Object "$ClientEvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$ClientEvidencePath\computer-domain-state.txt"

# Capture initial network state.
Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration-before.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\dns-client-server-addresses-before.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-before.txt"

# Release and renew lease.
ipconfig /release |
  Tee-Object "$ClientEvidencePath\ipconfig-release.txt"

Start-Sleep -Seconds 5

ipconfig /renew |
  Tee-Object "$ClientEvidencePath\ipconfig-renew.txt"

# Force DNS registration attempt.
Register-DnsClient

ipconfig /registerdns |
  Tee-Object "$ClientEvidencePath\ipconfig-registerdns.txt"

# Capture final network state.
Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration-after.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-after.txt"

# Validate DNS server reachability.
Test-NetConnection $ExpectedDnsServer -Port 53 |
  Tee-Object "$ClientEvidencePath\test-dns-server-tcp-53.txt"

# Validate domain resolution.
Resolve-DnsName $ExpectedDomain -Server $ExpectedDnsServer -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-domain.txt"

# Capture DHCP and DNS client events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\dhcp-client-admin-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\dns-client-operational-events.txt"
```

# Configure_DHCP_DNS_Dynamic_Updates_DNS_Record_Validation_Skeleton
```powershell
# Run in elevated PowerShell on DNS server or admin host with DNS tools.
# Purpose: validate DHCP-created A and PTR records after lease renewal.

$ForwardZone = "corp.local"
$ReverseZone = "20.20.10.in-addr.arpa"
$ClientHostName = "WIN11-01"
$ClientFqdn = "WIN11-01.corp.local"
$ExpectedClientIp = "10.20.20.50"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates\DNS-Record-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Validate A record through query.
Resolve-DnsName $ClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-client-a-record.txt"

# Validate PTR record through query.
Resolve-DnsName $ExpectedClientIp -Type PTR -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-client-ptr-record.txt"

# Validate A record directly in DNS zone.
Get-DnsServerResourceRecord -ZoneName $ForwardZone -Name $ClientHostName -ErrorAction SilentlyContinue |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\dns-zone-client-a-record.txt"

# Validate reverse zone records.
Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Where-Object {$_.RecordData -like "*$ClientFqdn*"} |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\dns-zone-client-ptr-record.txt"

# Capture zone dynamic update state.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,DynamicUpdate,ReplicationScope |
  Tee-Object "$EvidencePath\forward-zone-dynamic-update-state.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,DynamicUpdate,ReplicationScope |
  Tee-Object "$EvidencePath\reverse-zone-dynamic-update-state.txt"

# Capture recent DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events.txt"
```

# Configure_DHCP_DNS_Dynamic_Updates_Lease_Expiry_Cleanup_Skeleton
```powershell
# Run in a lab or controlled maintenance window.
# Purpose: validate whether DHCP deletes DNS records when lease expires or lease is removed.
# Do not remove production leases unless impact is approved.

$ScopeId = "10.20.20.0"
$ClientMac = "00-11-22-33-44-55"
$ClientIp = "10.20.20.50"
$ClientHostName = "WIN11-01"
$ClientFqdn = "WIN11-01.corp.local"
$ForwardZone = "corp.local"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates\Lease-Cleanup"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer
Import-Module DnsServer

# Confirm DHCP DNS cleanup setting.
Get-DhcpServerv4DnsSetting -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-dns-settings-before-cleanup-test.txt"

# Capture lease before removal.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ClientId $ClientMac -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-lease-before-cleanup-test.txt"

# Capture DNS record before removal.
Resolve-DnsName $ClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-a-record-before-cleanup-test.txt"

# Lab-only lease removal example.
# Remove-DhcpServerv4Lease `
#   -ScopeId $ScopeId `
#   -ClientId $ClientMac

# Wait for cleanup processing.
# Start-Sleep -Seconds 30

# Validate whether DNS record remains or is deleted.
Resolve-DnsName $ClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-a-record-after-cleanup-test.txt"

Get-DnsServerResourceRecord -ZoneName $ForwardZone -Name $ClientHostName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\zone-record-after-cleanup-test.txt"

# Review DHCP and DNS events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-events-after-cleanup-test.txt"

Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-after-cleanup-test.txt"
```

# Configure_DHCP_DNS_Dynamic_Updates_Event_Log_Skeleton
```powershell
# Run on DHCP server and DNS server as applicable.
# Purpose: collect event evidence for DHCP DNS dynamic updates.

$EvidencePath = "C:\DHCP-DNS-Dynamic-Updates\Events"
$Since = (Get-Date).AddHours(-24)

New-Item -ItemType Directory -Force -Path $EvidencePath

# DHCP Server operational events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Dhcp-Server/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-operational-events-last-24h.txt"

# DHCP Server admin events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Dhcp-Server/Admin"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-admin-events-last-24h.txt"

# DNS Server events.
Get-WinEvent -FilterHashtable @{
  LogName = "DNS Server"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-last-24h.txt"

# DNS Server operational events if available.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-DNSServer/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events-last-24h.txt"

# System DHCP/DNS/Netlogon events.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DHCP*" -or
    $_.ProviderName -like "*DNS*" -or
    $_.ProviderName -like "*Netlogon*" -or
    $_.Message -like "*dynamic update*" -or
    $_.Message -like "*DNS*"
  } |
  Tee-Object "$EvidencePath\system-dhcp-dns-events-last-24h.txt"

# Final service state.
Get-Service DHCPServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"

Get-Service DNS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-service-final.txt"
```

# Configure_DHCP_DNS_Dynamic_Updates_Verification_Commands
```powershell
# DHCP role and service.
Get-WindowsFeature DHCP
Get-Service DHCPServer
Import-Module DhcpServer

# DHCP authorization.
Get-DhcpServerInDC

# DHCP scopes and options.
Get-DhcpServerv4Scope
Get-DhcpServerv4Scope -ScopeId "<scope-id>"
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"

# Server-level DHCP DNS settings.
Get-DhcpServerv4DnsSetting

Set-DhcpServerv4DnsSetting `
  -DynamicUpdates Always `
  -DeleteDnsRRonLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true `
  -DisableDnsPtrRRUpdate $false

# Scope-level DHCP DNS settings.
Get-DhcpServerv4DnsSetting -ScopeId "<scope-id>"

Set-DhcpServerv4DnsSetting `
  -ScopeId "<scope-id>" `
  -DynamicUpdates Always `
  -DeleteDnsRRonLeaseExpiry $true `
  -UpdateDnsRRForOlderClients $true `
  -DisableDnsPtrRRUpdate $false

# Name protection.
Set-DhcpServerv4DnsSetting `
  -ScopeId "<scope-id>" `
  -NameProtection $true

# DHCP DNS credential.
Get-DhcpServerDnsCredential
Set-DhcpServerDnsCredential -Credential (Get-Credential)
Remove-DhcpServerDnsCredential

# DHCP leases.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -ClientId "<client-mac>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<client-ip>"

# DNS zone validation.
Import-Module DnsServer
Get-DnsServerZone -Name "<forward-zone-name>"
Get-DnsServerZone -Name "<reverse-zone-name>"
Get-DnsServerZone -Name "<forward-zone-name>" | Select-Object ZoneName,DynamicUpdate,IsDsIntegrated,ReplicationScope
Get-DnsServerResourceRecord -ZoneName "<forward-zone-name>" -Name "<client-hostname>"
Get-DnsServerResourceRecord -ZoneName "<reverse-zone-name>"

# DNS query validation.
Resolve-DnsName "<client-fqdn>" -Server "<dns-server-ip>"
Resolve-DnsName "<client-ip>" -Type PTR -Server "<dns-server-ip>"

# Client renewal and registration.
ipconfig /release
ipconfig /renew
ipconfig /all
Register-DnsClient
ipconfig /registerdns
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"

# Event logs.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
```

# Configure_DHCP_DNS_Dynamic_Updates_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Server-level DNS updates changed | Reapply previous `Set-DhcpServerv4DnsSetting` values from evidence | Clients may stop creating or cleaning DNS records |
| Scope-level DNS updates changed | Reapply previous scope-level DNS update settings | Only that scope is affected |
| DHCP DNS credential added incorrectly | `Remove-DhcpServerDnsCredential` or re-run `Set-DhcpServerDnsCredential` with correct account | Secure updates may fail until corrected |
| Name protection enabled and causes conflict | `Set-DhcpServerv4DnsSetting -ScopeId "<scope-id>" -NameProtection $false` | Name collision protection is reduced |
| PTR updates disabled accidentally | `Set-DhcpServerv4DnsSetting -ScopeId "<scope-id>" -DisableDnsPtrRRUpdate $false` | PTR records may not update until corrected |
| Delete on lease expiry disabled accidentally | Re-enable `-DeleteDnsRRonLeaseExpiry $true` | Stale records can accumulate |
| Manual DNS record changed | Restore record from DNS export or recreate expected A/PTR | Wrong DNS answers may continue until fixed |
| Client lease removed during test | Client renews lease with `ipconfig /renew` | Client may lose network temporarily |
| Evidence folder created | `Remove-Item C:\DHCP-DNS-Dynamic-Updates -Recurse -Force` | Deletes validation evidence |

# Configure_DHCP_DNS_Dynamic_Updates_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client gets lease but no A record | DHCP DNS setting, DNS credential, zone update mode, or client suffix issue | `Get-DhcpServerv4DnsSetting`; DNS Server events | Rebuilding DHCP scope |
| Client gets lease but no PTR record | PTR update disabled or reverse zone missing | `-DisableDnsPtrRRUpdate`; reverse zone existence | Editing forward record only |
| A record exists but points to old IP | Stale DNS record or cleanup disabled | DHCP lease, DNS timestamp, delete-on-expiry setting | Recreating DHCP server |
| PTR points to wrong hostname | Old PTR record or lease cleanup issue | Reverse zone record and DHCP lease history | Changing scope DNS option |
| Secure update fails | DHCP DNS credential missing/wrong or record ownership conflict | DHCP credential and DNS event logs | Setting zone to nonsecure updates |
| DHCP cannot update existing record | Record owned by old computer or wrong DHCP credential | DNS record ACL/ownership symptom and events | Deleting entire zone |
| Works for new records but not old records | Existing record ownership issue | Remove/recreate one stale test record after backup | Changing all DHCP settings |
| Older clients do not register | `UpdateDnsRRForOlderClients` disabled | DHCP DNS settings | Troubleshooting client DNS first |
| Records do not delete after lease expiry | `DeleteDnsRRonLeaseExpiry` disabled or DHCP did not own record | DHCP DNS settings and record ownership | Enabling scavenging immediately |
| DHCP failover partner updates inconsistently | Partners use different DNS credentials/settings | Compare settings on both DHCP servers | Blaming DNS replication first |
| Name protection blocks update | DHCID/name protection conflict | Name protection setting and DHCP events | Removing all DNS records |
| Client registers own record instead of DHCP | Client-side DNS registration behavior | Client `ipconfig /all`; DHCP DNS mode | Disabling DHCP DNS updates blindly |
| Reverse zone missing | PTR cannot be created | `Get-DnsServerZone -Name <reverse-zone>` | Changing DHCP credential |
| Forward zone not secure dynamic | DNS zone update mode wrong | `Get-DnsServerZone | Select DynamicUpdate` | Editing DHCP lease only |
| DNS server unreachable from DHCP server | Network/firewall issue | `Test-NetConnection <dns-ip> -Port 53` | Changing DNS update settings |
| DHCP events show credential failure | Password/account problem | Reset service account password and reapply credential | Recreating scope |
| DNS events show update refused | Secure update or ownership issue | DNS Server log and record ownership symptoms | Flushing cache only |
| Client FQDN wrong | DNS suffix option or client suffix wrong | DHCP option 015 and client `ipconfig /all` | Editing A/PTR manually |
| Record exists on one DC but not another | AD-integrated DNS replication | `repadmin /replsummary`; query each DNS server | Recreating record repeatedly |

# Configure_DHCP_DNS_Dynamic_Updates_Related_Labs
| Lab                                           | Related Workbook                                      | Skill Proven                                            |
| --------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| Install DHCP Server role and management tools | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | DHCP role baseline                                      |
| Authorize DHCP Server in Active Directory     | `02_Authorize_DHCP_Server_In_Active_Directory.md`     | AD-authorized DHCP service                              |
| Create DHCPv4 scope                           | `03_Create_DHCPv4_Scope.md`                           | Scope creation for client leases                        |
| Configure DHCPv4 scope options                | `04_Configure_DHCPv4_Scope_Options.md`                | DNS server and suffix delivery                          |
| Configure DHCPv4 exclusions and reservations  | `05_Configure_DHCPv4_Exclusions_And_Reservations.md`  | Stable device addressing before DNS validation          |
| Validate DHCP client lease assignment         | `06_Validate_DHCP_Client_Lease_Assignment.md`         | Client lease validation                                 |
| Configure DHCP relay and IP helper            | `07_Configure_DHCP_Relay_And_IP_Helper.md`            | Cross-subnet lease delivery before DNS updates          |
| Configure DHCP DNS dynamic updates            | `08_Configure_DHCP_DNS_Dynamic_Updates.md`            | DHCP-owned A/PTR registration and cleanup               |
| Configure DHCPv4 failover                     | `09_Configure_DHCPv4_Failover.md`                     | Consistent DNS update behavior across failover partners |
| Troubleshoot DHCP client lease failures       | `12_Troubleshoot_DHCP_Client_Lease_Failures.md`       | Lease and registration failure triage                   |
| Configure DNS dynamic updates                 | `09_Configure_DNS_Dynamic_Updates.md`                 | DNS-side secure update behavior                         |
| Configure DNS aging and scavenging            | `10_Configure_DNS_Aging_And_Scavenging.md`            | Long-term stale record cleanup                          |