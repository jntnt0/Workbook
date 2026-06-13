Windows_Server_DNS_Troubleshooting_Workflow.md
# Windows_Server_DNS_Troubleshooting_Workflow

# Windows_Server_DNS_Troubleshooting_Workflow_Index
Windows_Server_DNS_Troubleshooting_Workflow.md
Windows_Server_DNS_Troubleshooting_Workflow
Windows_Server_DNS_Troubleshooting_Workflow_Source_Basis
Windows_Server_DNS_Troubleshooting_Workflow_Mental_Model
Windows_Server_DNS_Troubleshooting_Workflow_Symptom_Map
Windows_Server_DNS_Troubleshooting_Workflow_Configuration_Checklist
Windows_Server_DNS_Troubleshooting_Workflow_Triage_Skeleton
Windows_Server_DNS_Troubleshooting_Workflow_Client_Path_Skeleton
Windows_Server_DNS_Troubleshooting_Workflow_Server_Service_Skeleton
Windows_Server_DNS_Troubleshooting_Workflow_Zone_Record_Skeleton
Windows_Server_DNS_Troubleshooting_Workflow_Forwarder_Recursion_Skeleton
Windows_Server_DNS_Troubleshooting_Workflow_AD_Domain_DNS_Skeleton
Windows_Server_DNS_Troubleshooting_Workflow_Event_Log_Skeleton
Windows_Server_DNS_Troubleshooting_Workflow_Verification_Commands
Windows_Server_DNS_Troubleshooting_Workflow_Rollback
Windows_Server_DNS_Troubleshooting_Workflow_Failure_Checks
Windows_Server_DNS_Troubleshooting_Workflow_Related_Labs

# Windows_Server_DNS_Troubleshooting_Workflow_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Resolve-DnsName | Testing DNS name resolution from client or server |
| Microsoft Learn | Get-DnsClientServerAddress | Confirming which DNS servers a client interface is using |
| Microsoft Learn | Clear-DnsClientCache | Clearing stale client resolver cache |
| Microsoft Learn | Get-DnsServerZone | Listing zones hosted by the DNS server |
| Microsoft Learn | Get-DnsServerResourceRecord | Inspecting A, PTR, CNAME, SRV, NS, SOA, and other zone records |
| Microsoft Learn | Get-DnsServerForwarder | Checking configured DNS forwarders |
| Microsoft Learn | Clear-DnsServerCache | Clearing stale DNS server cache |
| Windows Server operational practice | DNS Server event log and service checks | Diagnosing DNS service, zone load, forwarding, recursion, and AD registration issues |

# Windows_Server_DNS_Troubleshooting_Workflow_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS troubleshooting order | Start at the client resolver path, then server reachability, then DNS service, then zones, then records, then forwarding, then AD-specific SRV records |
| Client resolver | The machine asking DNS questions; bad DNS client settings can break everything even when the server is healthy |
| DNS server reachability | Client must reach the DNS server on UDP 53 and sometimes TCP 53 |
| UDP 53 | Normal DNS query transport |
| TCP 53 | Used for large responses, zone transfers, and some fallback behavior |
| DNS Server service | Windows service named `DNS`; if stopped, the server cannot answer queries |
| Local zone authority | If the DNS server hosts the zone, it should answer authoritatively for records in that zone |
| Recursive lookup | DNS server tries to resolve names outside its hosted zones |
| Forwarder | Upstream resolver used for external or non-local names |
| Root hints | Fallback resolution path when no forwarder is used or reachable |
| Client cache | Local cached DNS answers on the client; stale entries can hide the real current answer |
| Server cache | Cached answers on the DNS server; stale external or forwarded answers can mislead troubleshooting |
| Forward lookup failure | Name does not resolve to IP |
| Reverse lookup failure | IP does not resolve back to name |
| NXDOMAIN | DNS server says the name does not exist |
| SERVFAIL | DNS server failed to complete resolution |
| Timeout | Client did not get a DNS response in time |
| Wrong answer | DNS resolves, but returns stale, duplicate, or incorrect data |
| AD-integrated DNS | DNS zones stored in Active Directory and replicated with AD |
| SRV records | Domain controller locator records required for domain join, logon, LDAP, Kerberos, and AD discovery |
| Dynamic update | Clients or domain controllers register records automatically |
| First rule | Prove the exact failing query against the exact DNS server before changing anything |

# Windows_Server_DNS_Troubleshooting_Workflow_Symptom_Map
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client cannot resolve any names | Client DNS setting or DNS server reachability | `Get-DnsClientServerAddress`; `Test-NetConnection <dns-server-ip> -Port 53` | Recreating zones |
| Client resolves public names but not internal names | Wrong DNS server or missing internal zone | `Resolve-DnsName <internal-name> -Server <dns-server-ip>` | Changing forwarders |
| Client resolves internal names but not public names | Forwarder, recursion, root hints, firewall | `Get-DnsServerForwarder`; `Resolve-DnsName microsoft.com -Server <dns-server-ip>` | Rebuilding internal records |
| One hostname returns wrong IP | Stale or duplicate A record | `Get-DnsServerResourceRecord -ZoneName <zone> -Name <host>` | Restarting DNS service |
| Reverse lookup fails | Missing reverse zone or PTR | `Get-DnsServerZone`; `Resolve-DnsName <ip> -Server <dns-server-ip>` | Editing forward A record only |
| Domain join fails | Client DNS points outside AD DNS or missing SRV records | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain> -Type SRV` | Joining by NetBIOS name |
| Logon is slow | DNS timeout, unreachable DNS server, broken SRV lookup | `Resolve-DnsName <domain> -Type SRV`; event logs | Clearing random records |
| DNS works on server but not client | Client path, firewall, wrong interface DNS | `Resolve-DnsName <name> -Server <dns-server-ip>` from client | Reinstalling DNS role |
| External lookup times out | Forwarder unreachable or outbound firewall issue | `Resolve-DnsName microsoft.com -Server <forwarder-ip>` | Editing local zone |
| Query returns NXDOMAIN | Record missing, wrong zone, wrong suffix | `Get-DnsServerResourceRecord` and exact FQDN test | Flushing cache only |
| Query returns SERVFAIL | Forwarder/recursion/DNSSEC/upstream failure | DNS event logs and forwarder test | Adding duplicate records |
| Zone not visible | Zone missing, not loaded, AD replication issue | `Get-DnsServerZone` | Editing client settings |
| Record exists but lookup fails | Wrong DNS server queried, zone scope, cache, or suffix issue | Query exact server with `Resolve-DnsName -Server` | Assuming GUI view proves client behavior |

# Windows_Server_DNS_Troubleshooting_Workflow_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Define the exact symptom | Operator | `Symptom: <name fails / IP fails / public fails / domain join fails>` | Troubleshooting target is specific |
| 2 | Identify failing client | Client | `hostname` | Client name is known |
| 3 | Identify expected DNS server | Operator | `<dns-server-ip>` | DNS server under test is known |
| 4 | Identify failing name or IP | Operator | `<fqdn-or-ip>` | Exact query target is known |
| 5 | Check client IP configuration | Client | `Get-NetIPConfiguration` | Client IP, gateway, and DNS servers are visible |
| 6 | Check client DNS server list | Client | `Get-DnsClientServerAddress -AddressFamily IPv4` | Client points to intended internal DNS server |
| 7 | Test DNS server network reachability | Client | `Test-NetConnection <dns-server-ip> -Port 53` | TCP 53 test succeeds or failure is identified |
| 8 | Test exact query against intended DNS server | Client | `Resolve-DnsName <fqdn> -Server <dns-server-ip> -DnsOnly` | Query result, NXDOMAIN, SERVFAIL, or timeout is captured |
| 9 | Test same query from DNS server itself | DNS Server | `Resolve-DnsName <fqdn> -Server 127.0.0.1 -DnsOnly` | Confirms whether failure is client-path or server-side |
| 10 | Confirm DNS role installed | DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 11 | Confirm DNS service state | DNS Server | `Get-Service DNS` | DNS service is running |
| 12 | Confirm DNS server listeners | DNS Server | `Get-NetUDPEndpoint -LocalPort 53`; `Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue` | DNS is listening on port 53 |
| 13 | Confirm local zones | DNS Server | `Get-DnsServerZone` | Expected forward and reverse zones exist |
| 14 | Confirm target forward zone | DNS Server | `Get-DnsServerZone -Name "<zone-name>"` | Target zone is present |
| 15 | Confirm target record | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<host-name>"` | Expected record exists |
| 16 | Confirm record type | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<host-name>" -RRType A` | Correct A record exists |
| 17 | Confirm reverse zone | DNS Server | `Get-DnsServerZone | Where-Object {$_.ZoneName -like "*in-addr.arpa*"}` | Reverse zone exists if PTR is expected |
| 18 | Confirm PTR record | DNS Server | `Resolve-DnsName <ip-address> -Server <dns-server-ip>` | IP resolves to expected hostname |
| 19 | Check duplicate records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<host-name>"` | No stale duplicate A records exist |
| 20 | Check forwarders | DNS Server | `Get-DnsServerForwarder` | Upstream DNS servers are listed if forwarding is used |
| 21 | Test forwarder directly | DNS Server | `Resolve-DnsName microsoft.com -Server <forwarder-ip> -DnsOnly` | Forwarder resolves public name |
| 22 | Test recursion through DNS server | DNS Server | `Resolve-DnsName microsoft.com -Server 127.0.0.1 -DnsOnly` | Local DNS server resolves external name |
| 23 | Clear client cache after record changes | Client | `Clear-DnsClientCache` | Stale client answers are removed |
| 24 | Clear server cache if stale upstream answer suspected | DNS Server | `Clear-DnsServerCache -Force` | Stale server-side cache is removed |
| 25 | Check DNS Server event log | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 50` | Zone load, service, forwarding, and update errors are visible |
| 26 | Check System log for service/network issues | DNS Server | `Get-WinEvent -LogName System -MaxEvents 50` | Service, network, or AD-related issues are visible |
| 27 | For AD DNS, check SRV records | DNS Server or Client | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain> -Type SRV -Server <dns-server-ip>` | Domain controller locator records resolve |
| 28 | For AD DNS, restart Netlogon only if DC registration is broken | Domain Controller | `Restart-Service Netlogon` | DC attempts SRV record registration |
| 29 | Re-test original failing query | Client | `Resolve-DnsName <fqdn-or-ip> -Server <dns-server-ip> -DnsOnly` | Original symptom is resolved or narrowed |
| 30 | Document root cause | Operator | `Root cause: <client setting / missing record / stale cache / forwarder / firewall / AD SRV>` | Fix is recorded for repeatability |

# Windows_Server_DNS_Troubleshooting_Workflow_Triage_Skeleton
```powershell
# ============================================================
# Phase 1: Define the failing query
# ============================================================

$DnsServerIp = "10.10.10.10"
$ZoneName = "corp.local"
$HostName = "srv1"
$Fqdn = "$HostName.$ZoneName"
$HostIp = "10.10.10.20"

# ============================================================
# Phase 2: Client resolver baseline
# ============================================================

hostname
Get-NetIPConfiguration
Get-DnsClientServerAddress -AddressFamily IPv4

# Test direct DNS query against intended DNS server.
Resolve-DnsName $Fqdn -Server $DnsServerIp -DnsOnly

# Compare with default client resolver path.
Resolve-DnsName $Fqdn -DnsOnly

# ============================================================
# Phase 3: DNS server reachability from client
# ============================================================

Test-NetConnection $DnsServerIp -Port 53

# ============================================================
# Phase 4: Clear client cache and retest
# ============================================================

Clear-DnsClientCache
Resolve-DnsName $Fqdn -Server $DnsServerIp -DnsOnly
```

# Windows_Server_DNS_Troubleshooting_Workflow_Client_Path_Skeleton
```powershell
# Run on the failing client.

$DnsServerIp = "10.10.10.10"
$Fqdn = "srv1.corp.local"
$ExternalName = "microsoft.com"

# Show IP, gateway, DNS server, and interface data.
Get-NetIPConfiguration

# Show DNS servers assigned to each interface.
Get-DnsClientServerAddress -AddressFamily IPv4

# Test DNS server reachability.
Test-NetConnection $DnsServerIp -Port 53

# Query exact internal name using the intended DNS server.
Resolve-DnsName $Fqdn -Server $DnsServerIp -DnsOnly

# Query exact internal name using default interface DNS settings.
Resolve-DnsName $Fqdn -DnsOnly

# Query public name through intended internal DNS.
Resolve-DnsName $ExternalName -Server $DnsServerIp -DnsOnly

# Clear local resolver cache.
Clear-DnsClientCache

# Retest after cache clear.
Resolve-DnsName $Fqdn -Server $DnsServerIp -DnsOnly
```

# Windows_Server_DNS_Troubleshooting_Workflow_Server_Service_Skeleton
```powershell
# Run on the DNS server.

$DnsServerIp = "10.10.10.10"
$Fqdn = "srv1.corp.local"

# Confirm role installation.
Get-WindowsFeature DNS

# Confirm service state.
Get-Service DNS

# Start service if stopped.
Start-Service DNS

# Confirm DNS listener state.
Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue
Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue

# Query local DNS service through loopback.
Resolve-DnsName $Fqdn -Server 127.0.0.1 -DnsOnly

# Query local DNS service through server IP.
Resolve-DnsName $Fqdn -Server $DnsServerIp -DnsOnly

# Check recent DNS Server log entries.
Get-WinEvent -LogName "DNS Server" -MaxEvents 50
```

# Windows_Server_DNS_Troubleshooting_Workflow_Zone_Record_Skeleton
```powershell
# Run on the DNS server.

$ZoneName = "corp.local"
$HostName = "srv1"
$Fqdn = "$HostName.$ZoneName"
$DnsServerIp = "10.10.10.10"
$HostIp = "10.10.10.20"

# List all zones.
Get-DnsServerZone

# Check target forward lookup zone.
Get-DnsServerZone -Name $ZoneName

# List all records in the target zone.
Get-DnsServerResourceRecord -ZoneName $ZoneName

# Check target host records.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $HostName

# Check A record specifically.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $HostName -RRType A

# Query forward lookup.
Resolve-DnsName $Fqdn -Server $DnsServerIp -Type A -DnsOnly

# Query reverse lookup.
Resolve-DnsName $HostIp -Server $DnsServerIp -Type PTR -DnsOnly

# Find reverse zones.
Get-DnsServerZone | Where-Object {$_.ZoneName -like "*.in-addr.arpa"}
```

# Windows_Server_DNS_Troubleshooting_Workflow_Forwarder_Recursion_Skeleton
```powershell
# Run on the DNS server.

$ExternalName = "microsoft.com"
$DnsServerIp = "10.10.10.10"
$Forwarder1 = "1.1.1.1"
$Forwarder2 = "8.8.8.8"

# Show configured forwarders.
Get-DnsServerForwarder

# Test forwarders directly.
Resolve-DnsName $ExternalName -Server $Forwarder1 -DnsOnly
Resolve-DnsName $ExternalName -Server $Forwarder2 -DnsOnly

# Test recursion through local DNS server.
Resolve-DnsName $ExternalName -Server 127.0.0.1 -DnsOnly
Resolve-DnsName $ExternalName -Server $DnsServerIp -DnsOnly

# Clear DNS server cache if stale or bad cached answer is suspected.
Clear-DnsServerCache -Force

# Retest after cache clear.
Resolve-DnsName $ExternalName -Server $DnsServerIp -DnsOnly
```

# Windows_Server_DNS_Troubleshooting_Workflow_AD_Domain_DNS_Skeleton
```powershell
# Run on a domain client, member server, or domain controller.

$DomainName = "corp.local"
$DnsServerIp = "10.10.10.10"

# Confirm client points to internal AD DNS.
Get-DnsClientServerAddress -AddressFamily IPv4

# Test domain DNS name.
Resolve-DnsName $DomainName -Server $DnsServerIp -DnsOnly

# Test domain controller locator records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainName" -Type SRV -Server $DnsServerIp -DnsOnly
Resolve-DnsName "_kerberos._tcp.$DomainName" -Type SRV -Server $DnsServerIp -DnsOnly
Resolve-DnsName "_ldap._tcp.$DomainName" -Type SRV -Server $DnsServerIp -DnsOnly

# Test domain controller host records if known.
Resolve-DnsName "dc1.$DomainName" -Server $DnsServerIp -DnsOnly

# On a domain controller only, force DC locator registration if records are missing.
Restart-Service Netlogon

# Retest SRV records after Netlogon restart.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainName" -Type SRV -Server $DnsServerIp -DnsOnly
```

# Windows_Server_DNS_Troubleshooting_Workflow_Event_Log_Skeleton
```powershell
# Run on DNS server.

# DNS Server log.
Get-WinEvent -LogName "DNS Server" -MaxEvents 50 |
  Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message

# System log, filtered for DNS service related messages.
Get-WinEvent -LogName System -MaxEvents 100 |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*DNS*" -or
    $_.Message -like "*Netlogon*"
  } |
  Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message

# Optional: export DNS events for case notes.
Get-WinEvent -LogName "DNS Server" -MaxEvents 200 |
  Export-Csv ".\dns-server-events.csv" -NoTypeInformation
```

# Windows_Server_DNS_Troubleshooting_Workflow_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Client DNS server list | Client | `Get-DnsClientServerAddress -AddressFamily IPv4` | Client points to intended DNS server |
| Client IP configuration | Client | `Get-NetIPConfiguration` | Correct IP, gateway, and DNS server shown |
| DNS server TCP reachability | Client | `Test-NetConnection <dns-server-ip> -Port 53` | TCP test succeeds |
| Exact internal lookup against target DNS | Client | `Resolve-DnsName <fqdn> -Server <dns-server-ip> -DnsOnly` | Correct record or clear error returned |
| Exact internal lookup through default path | Client | `Resolve-DnsName <fqdn> -DnsOnly` | Same correct answer as direct server query |
| Client cache cleared | Client | `Clear-DnsClientCache` | Command completes |
| DNS role installed | DNS Server | `Get-WindowsFeature DNS` | DNS install state is installed |
| DNS service running | DNS Server | `Get-Service DNS` | Status is Running |
| UDP 53 listener | DNS Server | `Get-NetUDPEndpoint -LocalPort 53` | UDP endpoint exists |
| TCP 53 listener | DNS Server | `Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue` | TCP listener appears when active |
| Zones visible | DNS Server | `Get-DnsServerZone` | Expected zones listed |
| Forward zone visible | DNS Server | `Get-DnsServerZone -Name "<zone-name>"` | Zone exists |
| Host record visible | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<host>"` | Expected host record exists |
| A record visible | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<host>" -RRType A` | Correct IPv4 value exists |
| PTR lookup works | Client or Server | `Resolve-DnsName <ip-address> -Server <dns-server-ip> -Type PTR` | Expected hostname returned |
| Forwarders visible | DNS Server | `Get-DnsServerForwarder` | Expected upstream DNS servers listed |
| External lookup through DNS server | Client or Server | `Resolve-DnsName microsoft.com -Server <dns-server-ip> -DnsOnly` | Public name resolves |
| DNS server cache cleared | DNS Server | `Clear-DnsServerCache -Force` | Command completes |
| DNS logs readable | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 50` | DNS events are returned |
| AD LDAP SRV records resolve | Client or Server | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain> -Type SRV -Server <dns-server-ip>` | Domain controller SRV records returned |
| AD Kerberos SRV records resolve | Client or Server | `Resolve-DnsName _kerberos._tcp.<domain> -Type SRV -Server <dns-server-ip>` | Kerberos SRV records returned |

# Windows_Server_DNS_Troubleshooting_Workflow_Rollback
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current client DNS settings before change | Client | `Get-DnsClientServerAddress -AddressFamily IPv4 | Format-List *` | Original resolver settings are recorded |
| 2 | Capture current zones before change | DNS Server | `Get-DnsServerZone | Export-Csv .\dns-zones-before.csv -NoTypeInformation` | Zone inventory is backed up |
| 3 | Capture target zone records before change | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" | Export-Csv .\dns-records-before.csv -NoTypeInformation` | Record inventory is backed up |
| 4 | Revert client DNS to DHCP if manual DNS change was bad | Client | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ResetServerAddresses` | Client no longer uses manual DNS |
| 5 | Restore previous manual DNS server list if known | Client | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ServerAddresses "<old-dns-1>","<old-dns-2>"` | Client uses previous DNS servers |
| 6 | Remove accidentally added A record | DNS Server | `Remove-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<host>" -RRType A` | Wrong A record removed |
| 7 | Remove accidentally added PTR record | DNS Server | `Remove-DnsServerResourceRecord -ZoneName "<reverse-zone>" -Name "<ptr-node>" -RRType PTR` | Wrong PTR record removed |
| 8 | Remove bad forwarder | DNS Server | `Remove-DnsServerForwarder -IPAddress "<bad-forwarder-ip>" -Force` | Bad forwarder removed |
| 9 | Restore correct forwarder | DNS Server | `Add-DnsServerForwarder -IPAddress "<good-forwarder-ip>" -PassThru` | Correct forwarder restored |
| 10 | Clear client cache after rollback | Client | `Clear-DnsClientCache` | Client does not keep stale answer |
| 11 | Clear DNS server cache after rollback | DNS Server | `Clear-DnsServerCache -Force` | Server does not keep stale answer |
| 12 | Restart DNS service only if service state is bad | DNS Server | `Restart-Service DNS` | DNS service restarts cleanly |
| 13 | Retest original query | Client | `Resolve-DnsName <fqdn> -Server <dns-server-ip> -DnsOnly` | Query behavior returns to expected state |
| 14 | Record final state | Operator | `Root cause / Fix / Rollback notes` | Troubleshooting history is preserved |

# Windows_Server_DNS_Troubleshooting_Workflow_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `Resolve-DnsName` times out | DNS server unreachable, firewall, service stopped, wrong DNS server IP | `Test-NetConnection <dns-server-ip> -Port 53`; `Get-Service DNS` | Start DNS service, correct client DNS, permit DNS traffic |
| Client uses public DNS | Client DNS server list is wrong | `Get-DnsClientServerAddress` | Set client DNS to internal DNS server |
| Internal hostname returns NXDOMAIN | Record missing or wrong zone queried | `Get-DnsServerResourceRecord -ZoneName <zone> -Name <host>` | Add or correct record in authoritative zone |
| Internal hostname returns old IP | Stale A record, duplicate A record, client cache, server cache | Check record list, `Clear-DnsClientCache`, `Clear-DnsServerCache -Force` | Remove stale record and clear caches |
| Reverse lookup fails | Reverse zone or PTR missing | `Get-DnsServerZone`; `Resolve-DnsName <ip> -Type PTR` | Create reverse zone or PTR record |
| Public lookup fails but internal lookup works | Forwarder unreachable, recursion problem, outbound firewall | `Get-DnsServerForwarder`; test forwarder directly | Fix forwarder, route, firewall, or recursion path |
| Public lookup works from DNS server but not client | Client path or client DNS setting problem | Query with `-Server <dns-server-ip>` from client | Correct client DNS or firewall path |
| DNS server resolves locally but clients fail | Windows Firewall, network ACL, routing, wrong client subnet path | `Test-NetConnection <dns-server-ip> -Port 53` | Permit TCP/UDP 53 and fix routing |
| DNS service stopped | Service crash, manual stop, role issue | `Get-Service DNS`; event logs | Start service and review DNS Server log |
| Zone missing | Zone was not created, not loaded, or AD replication issue | `Get-DnsServerZone` | Recreate zone or fix AD replication/load problem |
| Zone exists but record lookup fails | Wrong record name, wrong FQDN, wrong suffix, wrong server queried | Query exact FQDN against exact server | Correct record, suffix, or client DNS server |
| `-CreatePtr` did not create PTR | Reverse lookup zone missing at record creation time | Check reverse zone | Create reverse zone and add PTR manually |
| Domain join cannot find domain | Client not using AD DNS or missing DC SRV records | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain> -Type SRV` | Point client to AD DNS and re-register DC records |
| LDAP/Kerberos SRV records missing | Netlogon registration issue on domain controller | Run SRV lookups and check Netlogon/System logs | Restart Netlogon on DC after DNS is healthy |
| One client fails but others work | Local cache, hosts file, wrong interface DNS, VPN adapter DNS priority | `Resolve-DnsName -NoHostsFile`; `Get-DnsClientServerAddress` | Clear cache, fix hosts file, fix interface DNS |
| VPN clients resolve wrong names | VPN DNS suffix or DNS server assignment wrong | `Get-DnsClientServerAddress`; `ipconfig /all` | Push correct DNS server and suffix through VPN profile |
| Query works with short name but not FQDN | Search suffix or different record path issue | Test both short name and FQDN | Fix suffix list or create correct FQDN record |
| Query works with FQDN but not short name | DNS suffix missing | `Get-DnsClientGlobalSetting`; `ipconfig /all` | Configure DNS suffix or use FQDN |
| SERVFAIL on external names | Upstream resolver failure, DNSSEC issue, recursion failure | Query forwarder directly and check logs | Fix forwarder or upstream path |
| Slow lookup | First DNS server unreachable, timeout before fallback | `Get-DnsClientServerAddress`; test each DNS server | Remove dead DNS server or fix reachability |
| Duplicate A records | Static plus dynamic registration conflict | `Get-DnsServerResourceRecord -ZoneName <zone> -Name <host>` | Remove stale duplicate and correct dynamic update behavior |
| DNS Manager shows record but PowerShell query fails | Looking at wrong server, wrong zone, wrong scope, replication delay | Use `-ComputerName` and exact zone | Query correct DNS server and wait/fix replication |
| Record changes do not appear on another DC | AD-integrated DNS replication delay or AD replication issue | Compare `Get-DnsServerResourceRecord` on each DC | Troubleshoot AD replication, then retest DNS |
| External domain shadowed internally | Internal zone has same name as public zone | `Get-DnsServerZone`; compare public vs internal record needs | Add required internal records or redesign split DNS |
| Wildcard or CNAME confusion | Alias points to wrong name or target missing | Check CNAME target and A record | Correct CNAME or add target A record |
| DNS logs show zone load errors | Bad zone file, AD issue, permission or corruption | DNS Server event log | Restore zone, fix AD, or recreate lab zone |
| DNS recursion disabled unexpectedly | Server will answer authoritative zones but not external names | Check server recursion/forwarder behavior | Re-enable recursion only if design requires it |
| Firewall blocks UDP but TCP test passes | `Test-NetConnection` only proves TCP 53 | Use packet capture or firewall review | Permit UDP 53 as well as TCP 53 |

# Windows_Server_DNS_Troubleshooting_Workflow_Related_Labs
| Lab | Relationship |
|---|---|
| `windows-dns-troubleshooting-001` | Client resolver path and DNS server selection |
| `windows-dns-troubleshooting-002` | DNS service and listener validation |
| `windows-dns-troubleshooting-003` | Forward lookup zone and A record failure |
| `windows-dns-troubleshooting-004` | Reverse lookup zone and PTR failure |
| `windows-dns-troubleshooting-005` | Forwarder and external recursion failure |
| `windows-dns-troubleshooting-006` | Stale DNS cache and wrong answer correction |
| `windows-dns-troubleshooting-007` | AD DNS SRV record and domain join troubleshooting |
| `windows-dns-troubleshooting-008` | Duplicate records and stale dynamic registration cleanup |
| `windows-dns-troubleshooting-009` | DNS event log triage |
| `windows-dns-troubleshooting-010` | Split DNS and internal zone shadowing |