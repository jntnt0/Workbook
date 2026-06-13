16_Troubleshoot_DNS_Name_Resolution_Failures.md
# Troubleshoot_DNS_Name_Resolution_Failures

# Troubleshoot_DNS_Name_Resolution_Failures_Index
16_Troubleshoot_DNS_Name_Resolution_Failures.md
Troubleshoot_DNS_Name_Resolution_Failures
Troubleshoot_DNS_Name_Resolution_Failures_Source_Basis
Troubleshoot_DNS_Name_Resolution_Failures_Mental_Model
Troubleshoot_DNS_Name_Resolution_Failures_Symptom_Map
Troubleshoot_DNS_Name_Resolution_Failures_Planning_Table
Troubleshoot_DNS_Name_Resolution_Failures_Configuration_Checklist
Troubleshoot_DNS_Name_Resolution_Failures_Client_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Server_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Internal_Record_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_AD_SRV_And_DC_Locator_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Reverse_Lookup_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_External_Forwarder_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Cache_And_Stale_Record_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Zone_Health_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Event_Log_Triage_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Remediation_Skeleton
Troubleshoot_DNS_Name_Resolution_Failures_Verification_Commands
Troubleshoot_DNS_Name_Resolution_Failures_Rollback
Troubleshoot_DNS_Name_Resolution_Failures_Failure_Checks
Troubleshoot_DNS_Name_Resolution_Failures_Related_Labs

# Troubleshoot_DNS_Name_Resolution_Failures_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Client PowerShell module | Checking client DNS servers, cache, suffixes, and registration behavior |
| Microsoft Learn | DNS Server PowerShell module | Checking DNS zones, records, cache, forwarders, root hints, recursion, and server statistics |
| Microsoft Learn | Resolve-DnsName | Testing A, AAAA, CNAME, PTR, NS, SOA, MX, TXT, and SRV record resolution |
| Microsoft Learn | Test-NetConnection | Testing DNS server reachability over TCP 53 and other service ports |
| Microsoft Learn | Get-DnsClientServerAddress | Validating client resolver configuration |
| Microsoft Learn | Clear-DnsClientCache | Clearing stale client-side DNS cache |
| Microsoft Learn | Clear-DnsServerCache | Clearing stale server-side DNS cache during controlled troubleshooting |
| Microsoft Learn | Get-DnsServerZone | Validating zone presence, zone type, AD integration, update mode, and zone state |
| Microsoft Learn | Get-DnsServerResourceRecord | Validating authoritative DNS records |
| Microsoft Learn | Get-DnsServerForwarder | Checking upstream DNS forwarders |
| Microsoft Learn | Get-DnsServerRootHint | Checking recursion fallback path |
| Microsoft Learn | Get-DnsServerStatistics | Reviewing DNS server query and failure counters |
| Microsoft Learn | nltest | Validating AD domain controller locator and secure channel behavior |
| Microsoft Learn | dcdiag /test:dns | Validating domain controller DNS health |
| Microsoft Learn | repadmin | Validating AD replication for AD-integrated DNS zones |
| Windows Server operational practice | Resolver path isolation, authoritative zone checks, forwarder checks, cache checks, AD DNS checks | Root-causing DNS failures without randomly changing records or rebuilding DNS |

# Troubleshoot_DNS_Name_Resolution_Failures_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Name resolution path | The full path from client query to DNS server, cache, authoritative zone, forwarder, root hint, or referral |
| Client resolver issue | Client uses wrong DNS server, wrong suffix, stale cache, hosts override, or bad adapter configuration |
| Server resolver issue | DNS server is reachable but cannot resolve because of zone, forwarder, recursion, cache, or service state |
| Authoritative zone issue | DNS server hosts the zone but the record, zone, or zone state is wrong |
| Recursive lookup issue | DNS server must resolve names outside its zones through forwarders or root hints |
| AD DNS issue | Domain clients cannot find SRV records, domain controllers, Kerberos, LDAP, or Global Catalog |
| Stale record issue | DNS returns old IP because of stale record, stale cache, duplicate record, or scavenging/dynamic update drift |
| Split-horizon DNS issue | Internal and external DNS answer differently by design or misconfiguration |
| Delegation issue | Parent zone points to wrong or unreachable child authoritative servers |
| Forwarder issue | Internal DNS server cannot reach upstream resolver or recursion is disabled |
| Cache issue | Client or server cache returns outdated data after a record change |
| Hosts file issue | Local static override causes DNS tests to lie |
| First rule | Isolate failure by testing the same name from client, directly against intended DNS server, directly against authoritative server, and then through recursion path |

# Troubleshoot_DNS_Name_Resolution_Failures_Symptom_Map
| Symptom | Most Likely Area | Fastest First Test |
|---|---|---|
| Client cannot resolve any names | Client DNS server list, DNS service, network path | `ipconfig /all`; `Test-NetConnection <dns-ip> -Port 53` |
| Client resolves external but not internal | Client points to public DNS or internal zone missing | `Get-DnsClientServerAddress`; `Resolve-DnsName <internal-name> -Server <ad-dns-ip>` |
| Client resolves internal but not external | Forwarder, recursion, root hints, firewall | `Get-DnsServerForwarder`; `Resolve-DnsName www.microsoft.com -Server <dns-ip>` |
| FQDN works but short name fails | DNS suffix or search list | `Get-DnsClientGlobalSetting`; `Resolve-DnsName <short-name>` |
| One client fails but others work | Local client config/cache/hosts file | Check DNS server list, cache, hosts file |
| One DNS server fails but another works | Zone replication, forwarder, cache, server-specific issue | Query both DNS servers directly |
| Domain join fails | AD DNS SRV or client DNS server issue | `_ldap._tcp.dc._msdcs.<domain>` lookup |
| Logon/GPO/Kerberos fails | DC locator or SRV issue | `nltest /dsgetdc:<domain>` |
| Reverse lookup fails | Missing reverse zone or PTR record | `Resolve-DnsName <ip> -Type PTR` |
| Record resolves to old IP | Stale record or cache | `Get-DnsServerResourceRecord`; `Get-DnsClientCache`; `Get-DnsServerCache` |
| Delegated child name fails | Delegation NS/glue or child DNS reachability | `Resolve-DnsName <child-zone> -Type NS -Server <parent-dns>` |
| Secondary/stub stale | Zone transfer/refresh failure | Compare SOA serials |
| Dynamic update not working | Zone update mode, DHCP DNS settings, record ownership | Zone DynamicUpdate, DHCP DNS settings, DNS logs |
| DNS intermittent | Multiple DNS servers differ, cache, load, firewall, packet loss | Query exact server and compare logs |

# Troubleshoot_DNS_Name_Resolution_Failures_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Failing client | `WIN11-01.corp.local` | `<client-fqdn>` |
| Client IP | `10.10.10.55` | `<client-ip>` |
| Client interface alias | `Ethernet` | `<interface-alias>` |
| Client subnet | `10.10.10.0/24` | `<client-subnet>` |
| Primary DNS server | `DC1.corp.local` | `<dns-server-1>` |
| Primary DNS server IP | `10.10.10.10` | `<dns-server-1-ip>` |
| Alternate DNS server | `DC2.corp.local` | `<dns-server-2>` |
| Alternate DNS server IP | `10.10.10.11` | `<dns-server-2-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Failing name | `app1.corp.local` | `<failing-name>` |
| Expected IP | `10.10.10.50` | `<expected-ip>` |
| Record type | `A` | `<record-type>` |
| Forward zone | `corp.local` | `<forward-zone-name>` |
| Reverse zone | `10.10.10.in-addr.arpa` | `<reverse-zone-name>` |
| External test name | `www.microsoft.com` | `<external-test-name>` |
| SRV test name | `_ldap._tcp.dc._msdcs.corp.local` | `<srv-test-name>` |
| Forwarder IP | `10.10.10.1` | `<forwarder-ip>` |
| Parent zone if delegated | `corp.local` | `<parent-zone-name>` |
| Child zone if delegated | `dev.corp.local` | `<child-zone-name>` |
| Authoritative DNS server | `DC1.corp.local` | `<authoritative-dns-server>` |
| Evidence path | `C:\DNS-Resolution-Troubleshooting` | `<evidence-path>` |
| Rollback stance | Export current state before changing DNS records, caches, forwarders, or zone settings | `<rollback-plan>` |
| Next workbook | `17_Troubleshoot_DNS_Server_Zone_And_Forwarder_Failures.md` | `<next-task>` |

# Troubleshoot_DNS_Name_Resolution_Failures_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture exact symptom | Operator | `Record failing name, client, expected result, actual result, time, DNS server used` | Scope is defined |
| 2 | Confirm client IP config | Client | `ipconfig /all`; `Get-NetIPConfiguration` | Client IP, gateway, DNS, suffix are visible |
| 3 | Confirm client DNS servers | Client | `Get-DnsClientServerAddress -AddressFamily IPv4` | Client uses intended DNS servers |
| 4 | Check hosts file | Client | `Get-Content "$env:SystemRoot\System32\drivers\etc\hosts"` | Local overrides are known |
| 5 | Clear client cache for retest | Client | `Clear-DnsClientCache` | Stale client cache is cleared |
| 6 | Test DNS server reachability | Client | `Test-NetConnection "<dns-server-ip>" -Port 53` | DNS server is reachable on TCP 53 |
| 7 | Query failing name using default resolver | Client | `Resolve-DnsName "<failing-name>"` | Actual default resolver result is captured |
| 8 | Query failing name against intended DNS server | Client/Admin Host | `Resolve-DnsName "<failing-name>" -Server "<dns-server-ip>"` | Intended DNS server result is captured |
| 9 | Query failing name against alternate DNS server | Client/Admin Host | `Resolve-DnsName "<failing-name>" -Server "<alternate-dns-ip>"` | Server-to-server difference is identified |
| 10 | Confirm DNS server role and service | DNS Server | `Get-WindowsFeature DNS`; `Get-Service DNS` | DNS server is installed and running |
| 11 | Confirm DNS server zones | DNS Server | `Get-DnsServerZone` | Hosted zones are visible |
| 12 | Confirm authoritative zone | DNS Server | `Get-DnsServerZone -Name "<zone-name>"` | Correct zone exists |
| 13 | Confirm expected record | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<record-name>"` | Expected record exists or is confirmed missing |
| 14 | Confirm reverse zone if PTR issue | DNS Server | `Get-DnsServerZone -Name "<reverse-zone-name>"` | Reverse zone exists |
| 15 | Test PTR if reverse failure | Client/Admin Host | `Resolve-DnsName "<ip-address>" -Type PTR -Server "<dns-server-ip>"` | PTR result is captured |
| 16 | Test AD SRV record if AD issue | Client/Admin Host | `Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain>" -Type SRV -Server "<dns-server-ip>"` | DC locator SRV result is captured |
| 17 | Test DC locator | Client | `nltest /dsgetdc:<domain-fqdn>` | Client can or cannot locate a DC |
| 18 | Confirm forwarders for external failure | DNS Server | `Get-DnsServerForwarder`; `Get-DnsServerRecursion` | Upstream resolver path is visible |
| 19 | Test forwarder reachability | DNS Server | `Test-NetConnection "<forwarder-ip>" -Port 53` | Forwarder path is reachable |
| 20 | Query external name from DNS server | DNS Server | `Resolve-DnsName "<external-test-name>"` | External recursive behavior is captured |
| 21 | Inspect DNS server cache | DNS Server | `Get-DnsServerCache` | Cached entries are visible |
| 22 | Clear server cache only if controlled | DNS Server | `Clear-DnsServerCache -Force` | Stale server cache is cleared |
| 23 | Review DNS server events | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | DNS errors/warnings are visible |
| 24 | Review DNS client events | Client | `Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100` | Client resolver events are visible |
| 25 | Check AD replication if AD-integrated zone differs | Domain Controller | `repadmin /replsummary`; `repadmin /showrepl` | AD replication state is known |
| 26 | Run DNS diagnostic if AD DNS involved | Domain Controller | `dcdiag /test:dns /v` | DC DNS diagnostic output is captured |
| 27 | Apply smallest safe remediation | Correct Layer | Use remediation skeleton | Fix targets root cause only |
| 28 | Retest exact failure path | Client/Admin Host | Same failing query and same DNS server path | Resolution result is corrected |
| 29 | Export evidence | Client / DNS Server | Run triage skeletons | Evidence files are saved |
| 30 | Document root cause and fix | Operator | `Record symptom, cause, changed setting/record, validation, rollback notes` | Troubleshooting record is complete |

# Troubleshoot_DNS_Name_Resolution_Failures_Client_Triage_Skeleton
```powershell
# Run in elevated PowerShell on the failing Windows client.
# Purpose: isolate local client DNS resolver, cache, adapter, suffix, and hosts file issues.

$FailingName = "app1.corp.local"
$ShortName = "app1"
$DomainFqdn = "corp.local"
$DnsServer1 = "10.10.10.10"
$DnsServer2 = "10.10.10.11"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Client"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture client identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion |
  Tee-Object "$EvidencePath\computer-info.txt"

# Capture adapter and IP state.
Get-NetAdapter |
  Tee-Object "$EvidencePath\net-adapters.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration.txt"

Get-NetIPAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\net-ipv4-addresses.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all.txt"

# Capture DNS client settings.
Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dns-client-server-addresses.txt"

Get-DnsClient |
  Tee-Object "$EvidencePath\dns-client-settings.txt"

Get-DnsClientGlobalSetting |
  Tee-Object "$EvidencePath\dns-client-global-settings.txt"

# Capture hosts file.
Get-Content "$env:SystemRoot\System32\drivers\etc\hosts" |
  Tee-Object "$EvidencePath\hosts-file.txt"

# Capture cache before clear.
Get-DnsClientCache |
  Tee-Object "$EvidencePath\dns-client-cache-before-clear.txt"

# Clear client cache for clean retest.
Clear-DnsClientCache

# DNS server reachability.
Test-NetConnection $DnsServer1 -Port 53 |
  Tee-Object "$EvidencePath\test-dns-server1-tcp-53.txt"

Test-NetConnection $DnsServer2 -Port 53 |
  Tee-Object "$EvidencePath\test-dns-server2-tcp-53.txt"

# Query through default resolver path.
Resolve-DnsName $FailingName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-failing-name-default.txt"

# Query directly against expected DNS servers.
Resolve-DnsName $FailingName -Server $DnsServer1 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-failing-name-dns1.txt"

Resolve-DnsName $FailingName -Server $DnsServer2 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-failing-name-dns2.txt"

# Test short-name suffix behavior.
Resolve-DnsName $ShortName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-short-name-default.txt"

# Test domain root and AD SRV if domain environment.
Resolve-DnsName $DomainFqdn -Server $DnsServer1 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-domain-root-dns1.txt"

Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServer1 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-ad-dc-locator-srv-dns1.txt"

# DNS Client events.
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-client-operational-events.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_Server_Triage_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: validate DNS role, service, zones, records, forwarders, recursion, cache, and statistics.

$FailingName = "app1.corp.local"
$ZoneName = "corp.local"
$RecordName = "app1"
$DnsServerIp = "10.10.10.10"
$ExternalName = "www.microsoft.com"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Server"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm DNS role and service.
Get-WindowsFeature DNS |
  Tee-Object "$EvidencePath\dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-state.txt"

# Capture server identity and IP state.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all.txt"

# Import DNS module.
Import-Module DnsServer

# Capture zones.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones.txt"

Get-DnsServerZone |
  Export-Csv "$EvidencePath\dns-zones.csv" -NoTypeInformation

# Capture target zone.
Get-DnsServerZone -Name $ZoneName -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\target-zone-properties.txt"

# Capture target record.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-record.txt"

# Capture all zone records for comparison.
Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\target-zone-records.csv" -NoTypeInformation

# Query failing name locally against this server.
Resolve-DnsName $FailingName -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-failing-name-against-this-server.txt"

# Query external name from this DNS server context.
Resolve-DnsName $ExternalName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-external-from-server.txt"

# Capture forwarders and recursion.
Get-DnsServerForwarder |
  Tee-Object "$EvidencePath\dns-forwarders.txt"

Get-DnsServerRecursion |
  Tee-Object "$EvidencePath\dns-recursion.txt"

Get-DnsServerRootHint |
  Tee-Object "$EvidencePath\dns-root-hints.txt"

# Capture cache and statistics.
Get-DnsServerCache |
  Export-Csv "$EvidencePath\dns-server-cache.csv" -NoTypeInformation

Get-DnsServerStatistics |
  Tee-Object "$EvidencePath\dns-server-statistics.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_Internal_Record_Triage_Skeleton
```powershell
# Run in elevated PowerShell on the authoritative DNS server.
# Purpose: troubleshoot missing, wrong, stale, duplicate, or conflicting internal records.

$ZoneName = "corp.local"
$RecordName = "app1"
$ExpectedFqdn = "app1.corp.local"
$ExpectedIp = "10.10.10.50"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Internal-Record"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Confirm zone.
Get-DnsServerZone -Name $ZoneName |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\zone-properties.txt"

# Check exact record.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\exact-record.txt"

# Check duplicate records with same host label.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordName -ErrorAction SilentlyContinue |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\duplicate-host-record-check.txt"

# Search for expected IP across zone.
Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Where-Object {$_.RecordData -like "*$ExpectedIp*"} |
  Tee-Object "$EvidencePath\records-containing-expected-ip.txt"

# Search for stale records by hostname pattern.
Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Where-Object {$_.HostName -like "*$RecordName*"} |
  Tee-Object "$EvidencePath\records-matching-hostname-pattern.txt"

# Resolve from authoritative server.
Resolve-DnsName $ExpectedFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-expected-fqdn-authoritative.txt"

# Inspect cache for the failing name.
Get-DnsServerCache |
  Where-Object {$_.Name -like "*$ExpectedFqdn*"} |
  Tee-Object "$EvidencePath\server-cache-matching-failing-name.txt"

# Capture record timestamp to identify dynamic/stale behavior.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordName -ErrorAction SilentlyContinue |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\record-timestamp.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_AD_SRV_And_DC_Locator_Triage_Skeleton
```powershell
# Run on domain client and domain controller as needed.
# Purpose: troubleshoot AD DNS SRV, DC locator, Kerberos, LDAP, and secure channel failures.

$DomainFqdn = "corp.local"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\AD-SRV-DC-Locator"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture local DNS config.
ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dns-client-server-addresses.txt"

# Clear DNS cache before AD locator tests.
Clear-DnsClientCache

# Validate AD SRV records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-ldap-dc-msdcs-srv.txt"

Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-kerberos-srv.txt"

Resolve-DnsName "_ldap._tcp.gc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-global-catalog-srv.txt"

# DC locator.
nltest /dsgetdc:$DomainFqdn |
  Tee-Object "$EvidencePath\nltest-dsgetdc.txt"

nltest /dsgetdc:$DomainFqdn /force |
  Tee-Object "$EvidencePath\nltest-dsgetdc-force.txt"

nltest /dsgetdc:$DomainFqdn /writable |
  Tee-Object "$EvidencePath\nltest-dsgetdc-writable.txt"

nltest /dsgetdc:$DomainFqdn /gc |
  Tee-Object "$EvidencePath\nltest-dsgetdc-gc.txt"

# Secure channel if domain joined.
nltest /sc_verify:$DomainFqdn |
  Tee-Object "$EvidencePath\nltest-secure-channel-verify.txt"

# GPO evidence.
gpresult /r |
  Tee-Object "$EvidencePath\gpresult-r.txt"

# DC-side diagnostics if this runs on DC.
dcdiag /test:dns /v |
  Tee-Object "$EvidencePath\dcdiag-dns.txt"

repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_Reverse_Lookup_Triage_Skeleton
```powershell
# Run in elevated PowerShell on DNS server or admin host.
# Purpose: troubleshoot PTR and reverse lookup failures.

$ReverseZone = "10.10.10.in-addr.arpa"
$PtrTestIp = "10.10.10.50"
$ExpectedPtrName = "app1.corp.local"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Reverse-Lookup"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Confirm reverse zone exists.
Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-properties.txt"

# Export reverse zone records.
Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-records.txt"

Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\reverse-zone-records.csv" -NoTypeInformation

# Test PTR lookup.
Resolve-DnsName $PtrTestIp -Type PTR -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-ptr-test-ip.txt"

# Search for expected PTR name.
Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Where-Object {$_.RecordData -like "*$ExpectedPtrName*"} |
  Tee-Object "$EvidencePath\ptr-records-matching-expected-name.txt"

# Check DHCP DNS update settings if DHCP manages PTR.
Get-DhcpServerv4DnsSetting -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-dns-settings.txt"

# Check zone dynamic update mode.
Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-dynamic-update-setting.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_External_Forwarder_Triage_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: troubleshoot external DNS failures caused by forwarders, recursion, root hints, firewall, or upstream DNS.

$ExternalName = "www.microsoft.com"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\External-Forwarder"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Capture forwarders.
Get-DnsServerForwarder |
  Tee-Object "$EvidencePath\dns-forwarders.txt"

# Capture recursion.
Get-DnsServerRecursion |
  Tee-Object "$EvidencePath\dns-recursion.txt"

# Capture root hints.
Get-DnsServerRootHint |
  Tee-Object "$EvidencePath\dns-root-hints.txt"

# Test external lookup through local/default DNS path.
Resolve-DnsName $ExternalName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-external-default.txt"

# Test external lookup explicitly against this DNS server.
Resolve-DnsName $ExternalName -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-external-against-this-dns.txt"

# Test each configured forwarder.
$Forwarders = (Get-DnsServerForwarder).IPAddress

foreach ($Forwarder in $Forwarders) {
  $ForwarderIp = $Forwarder.IPAddressToString

  Test-NetConnection $ForwarderIp -Port 53 |
    Tee-Object "$EvidencePath\test-forwarder-$ForwarderIp-tcp-53.txt"

  Resolve-DnsName $ExternalName -Server $ForwarderIp -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\resolve-external-via-forwarder-$ForwarderIp.txt"
}

# Capture DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_Cache_And_Stale_Record_Triage_Skeleton
```powershell
# Run on DNS server and client as appropriate.
# Purpose: identify stale cache, stale records, duplicate records, and timestamp issues.

$FailingName = "app1.corp.local"
$ZoneName = "corp.local"
$RecordName = "app1"
$ExpectedIp = "10.10.10.50"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Cache-Stale"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Capture server cache before clearing.
Get-DnsServerCache |
  Where-Object {$_.Name -like "*$FailingName*"} |
  Tee-Object "$EvidencePath\dns-server-cache-matching-failing-name.txt"

# Capture authoritative record.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordName -ErrorAction SilentlyContinue |
  Select-Object HostName,RecordType,Timestamp,TimeToLive,RecordData |
  Tee-Object "$EvidencePath\authoritative-record.txt"

# Search for records with expected IP.
Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Where-Object {$_.RecordData -like "*$ExpectedIp*"} |
  Tee-Object "$EvidencePath\records-containing-expected-ip.txt"

# Query before cache clear.
Resolve-DnsName $FailingName -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-before-server-cache-clear.txt"

# Optional controlled server cache clear.
# Clear-DnsServerCache -Force

# Query after optional clear.
Resolve-DnsName $FailingName -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-after-server-cache-clear.txt"

# Client-side cache commands to run on affected client.
# Get-DnsClientCache | Where-Object {$_.Entry -like "*app1*"}
# Clear-DnsClientCache
# Resolve-DnsName app1.corp.local
```

# Troubleshoot_DNS_Name_Resolution_Failures_Zone_Health_Triage_Skeleton
```powershell
# Run in elevated PowerShell on DNS server/domain controller.
# Purpose: check zone presence, AD integration, replication, delegation, aging, and dynamic update settings.

$ZoneName = "corp.local"
$MsdcsZone = "_msdcs.corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Zone-Health"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Zone inventory.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones.txt"

# Target zone properties.
Get-DnsServerZone -Name $ZoneName -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\forward-zone-properties.txt"

Get-DnsServerZone -Name $MsdcsZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\msdcs-zone-properties.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\reverse-zone-properties.txt"

# SOA and NS records.
Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType SOA -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-soa.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType NS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-ns.txt"

# Aging and scavenging.
Get-DnsServerZoneAging -Name $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-aging.txt"

Get-DnsServerScavenging -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-scavenging.txt"

# AD replication if applicable.
repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary.txt"

repadmin /showrepl |
  Tee-Object "$EvidencePath\repadmin-showrepl.txt"

# DC DNS diagnostic.
dcdiag /test:dns /v |
  Tee-Object "$EvidencePath\dcdiag-dns.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_Event_Log_Triage_Skeleton
```powershell
# Run on DNS server and failing client as applicable.
# Purpose: collect DNS, System, DHCP, Netlogon, Directory Service, and DNS Client event evidence.

$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Event-Logs"
$Since = (Get-Date).AddHours(-24)

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server events.
Get-WinEvent -FilterHashtable @{
  LogName = "DNS Server"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-last-24h.txt"

# DNS Server operational events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-DNSServer/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events-last-24h.txt"

# DNS Client events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-DNS-Client/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-client-operational-events-last-24h.txt"

# DHCP Client events because DHCP often delivers DNS server/suffix options.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Dhcp-Client/Admin"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-client-admin-events-last-24h.txt"

# System DNS/DHCP/Netlogon events.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.ProviderName -like "*DHCP*" -or
    $_.ProviderName -like "*Netlogon*" -or
    $_.Message -like "*DNS*" -or
    $_.Message -like "*name resolution*" -or
    $_.Message -like "*domain controller*"
  } |
  Tee-Object "$EvidencePath\system-dns-dhcp-netlogon-events-last-24h.txt"

# Directory Service events for AD-integrated DNS.
Get-WinEvent -FilterHashtable @{
  LogName = "Directory Service"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\directory-service-events-last-24h.txt"
```

# Troubleshoot_DNS_Name_Resolution_Failures_Remediation_Skeleton
```powershell
# Run only after triage identifies the specific root cause.
# Purpose: apply smallest safe DNS remediation.
# Export current state before changing anything.

$ZoneName = "corp.local"
$RecordName = "app1"
$CorrectIp = "10.10.10.50"
$DnsServer1 = "10.10.10.10"
$DnsServer2 = "10.10.10.11"
$InterfaceAlias = "Ethernet"
$EvidencePath = "C:\DNS-Resolution-Troubleshooting\Remediation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Export current zone records before remediation.
Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\zone-records-before-remediation.csv" -NoTypeInformation

# Example 1: fix client DNS server list.
# Run on client.
# Set-DnsClientServerAddress `
#   -InterfaceAlias $InterfaceAlias `
#   -ServerAddresses $DnsServer1,$DnsServer2
# Clear-DnsClientCache

# Example 2: remove wrong A record.
# Remove-DnsServerResourceRecord `
#   -ZoneName $ZoneName `
#   -Name $RecordName `
#   -RRType A `
#   -RecordData "<wrong-ip>" `
#   -Force

# Example 3: create missing A record.
# Add-DnsServerResourceRecordA `
#   -ZoneName $ZoneName `
#   -Name $RecordName `
#   -IPv4Address $CorrectIp `
#   -TimeToLive 01:00:00

# Example 4: fix DNS forwarders.
# Set-DnsServerForwarder `
#   -IPAddress "<forwarder-ip-1>","<forwarder-ip-2>" `
#   -UseRootHint $true

# Example 5: force DC DNS registration for missing AD SRV records.
# ipconfig /registerdns
# nltest /dsregdns
# Restart-Service Netlogon

# Example 6: clear server cache after confirmed stale cached answer.
# Clear-DnsServerCache -Force

# Post-remediation query validation.
Resolve-DnsName "$RecordName.$ZoneName" -Server $DnsServer1 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-record-after-remediation.txt"

# Export records after remediation.
Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\zone-records-after-remediation.csv" -NoTypeInformation
```

# Troubleshoot_DNS_Name_Resolution_Failures_Verification_Commands
```powershell
# Client baseline.
hostname
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
Get-NetAdapter
Get-NetIPConfiguration
Get-DnsClientServerAddress -AddressFamily IPv4
Get-DnsClient
Get-DnsClientGlobalSetting
ipconfig /all

# Hosts file and cache.
Get-Content "$env:SystemRoot\System32\drivers\etc\hosts"
Get-DnsClientCache
Clear-DnsClientCache

# Client query tests.
Resolve-DnsName "<failing-name>"
Resolve-DnsName "<failing-name>" -Server "<dns-server-1-ip>"
Resolve-DnsName "<failing-name>" -Server "<dns-server-2-ip>"
Resolve-DnsName "<short-name>"
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"

# DNS server reachability.
Test-NetConnection "<dns-server-1-ip>" -Port 53
Test-NetConnection "<dns-server-2-ip>" -Port 53

# DNS server baseline.
Get-WindowsFeature DNS
Get-Service DNS
Import-Module DnsServer
Get-DnsServerZone
Get-DnsServerZone -Name "<zone-name>"
Get-DnsServerZone -Name "<zone-name>" | Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Record checks.
Get-DnsServerResourceRecord -ZoneName "<zone-name>"
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<record-name>"
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType SOA
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NS
Resolve-DnsName "<fqdn>" -Server "<authoritative-dns-ip>"

# Reverse lookup checks.
Get-DnsServerZone -Name "<reverse-zone-name>"
Get-DnsServerResourceRecord -ZoneName "<reverse-zone-name>" -RRType PTR
Resolve-DnsName "<ip-address>" -Type PTR -Server "<dns-server-ip>"

# AD SRV and DC locator checks.
Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_kerberos._tcp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_ldap._tcp.gc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
nltest /dsgetdc:<domain-fqdn>
nltest /dsgetdc:<domain-fqdn> /force
nltest /sc_verify:<domain-fqdn>

# Forwarder and recursion checks.
Get-DnsServerForwarder
Get-DnsServerRecursion
Get-DnsServerRootHint
Test-NetConnection "<forwarder-ip>" -Port 53
Resolve-DnsName "<external-test-name>" -Server "<dns-server-ip>"
Resolve-DnsName "<external-test-name>" -Server "<forwarder-ip>"

# Cache and stats.
Get-DnsServerCache
Get-DnsServerCache | Where-Object {$_.Name -like "*<name-pattern>*"}
Clear-DnsServerCache -Force
Get-DnsServerStatistics
Get-Counter "\DNS\*"

# AD-integrated DNS health.
dcdiag /test:dns /v
repadmin /replsummary
repadmin /showrepl

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DNS*" -or $_.ProviderName -like "*DHCP*" -or $_.ProviderName -like "*Netlogon*"}
```

# Troubleshoot_DNS_Name_Resolution_Failures_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Client DNS server list changed | Restore previous DNS server list with `Set-DnsClientServerAddress` | Client may lose AD DNS resolution until restored |
| Client DNS set back to DHCP | Reapply static DNS list if DHCP options are wrong | Client depends on DHCP option 006 |
| Client cache cleared | No rollback; cache rebuilds naturally | First lookups may be slower |
| Server cache cleared | No rollback; cache rebuilds naturally | First recursive lookups may be slower |
| A record changed | Restore previous A record from exported inventory | Wrong IP breaks application access |
| CNAME changed | Restore previous alias target | Alias may point to wrong host until fixed |
| PTR record changed | Restore previous PTR record | Reverse lookup may remain broken |
| Forwarder changed | Restore previous forwarder list | External resolution may fail |
| Recursion changed | Restore previous recursion setting | Resolver may be too open or too restricted |
| Dynamic update setting changed | Restore previous zone DynamicUpdate mode | Clients may fail or unsafe updates may be allowed |
| Netlogon restarted | No rollback normally needed | Brief DC locator registration interruption |
| DNS service restarted | No rollback normally needed | Brief DNS outage possible |
| Evidence folder created | `Remove-Item C:\DNS-Resolution-Troubleshooting -Recurse -Force` | Deletes troubleshooting evidence |

# Troubleshoot_DNS_Name_Resolution_Failures_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client cannot resolve anything | Client DNS server list or DNS server reachability | `ipconfig /all`; `Test-NetConnection <dns-ip> -Port 53` | Editing DNS records |
| Client resolves external but not internal | Client using public DNS or missing internal zone | `Get-DnsClientServerAddress` | Changing forwarders |
| Client resolves internal but not external | Forwarder, recursion, root hints, firewall | `Get-DnsServerForwarder`; external lookup from DNS server | Editing AD SRV records |
| FQDN works but short name fails | DNS suffix search list | `Get-DnsClientGlobalSetting` | Recreating records |
| One client fails only | Client cache, hosts file, adapter DNS, VPN DNS | Hosts file and client resolver settings | Rebuilding DNS server |
| One DNS server fails only | Zone replication, cache, forwarder, or server issue | Query both DNS servers directly | Rejoining client |
| Wrong IP returned | Stale record, duplicate record, stale cache | Authoritative record and cache checks | Flushing cache only |
| NXDOMAIN returned | Missing record or wrong zone queried | Authoritative zone and record check | Restarting DNS first |
| SERVFAIL returned | Server recursion, delegation, DNSSEC, or upstream issue | DNS Server logs and forwarder path | Creating new record randomly |
| Timeout returned | Firewall, routing, service down, or unreachable server | `Test-NetConnection <dns-ip> -Port 53` | Editing zone data |
| Reverse lookup fails | Reverse zone or PTR missing | `Resolve-DnsName <ip> -Type PTR` | Changing A record first |
| Domain join fails | Client DNS or AD SRV issue | `_ldap._tcp.dc._msdcs` SRV lookup | Resetting computer account first |
| Logon/GPO fails | DC locator, Kerberos SRV, or secure channel | `nltest /dsgetdc`; Kerberos SRV lookup | Editing DHCP exclusions |
| Dynamic update fails | Zone update mode, DHCP DNS settings, permissions, ownership | DNS/DHCP event logs and zone DynamicUpdate | Making zone nonsecure blindly |
| Delegated child zone fails | NS/glue or child DNS reachability | Parent delegation and child SOA/NS direct lookup | Deleting parent zone |
| External forwarder fails | Upstream resolver/firewall | Forwarder TCP 53 and direct query | Editing internal records |
| Cache keeps old answer | Client or server cache | Clear correct cache and retest authoritative answer | Changing records again |
| Secondary DNS stale | Zone transfer/AD replication issue | Compare SOA serials and replication status | Recreating secondary zone immediately |
| AD-integrated DNS differs by DC | AD replication issue | `repadmin /replsummary` | Manual record edits on every DC |
| DNS service logs zone load error | Zone file, AD replication, or corruption issue | DNS Server event log | Clearing client cache |

# Troubleshoot_DNS_Name_Resolution_Failures_Related_Labs
| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Configure AD-integrated DNS zones | `07_Configure_AD_Integrated_DNS_Zones.md` | Authoritative internal DNS zone baseline |
| Validate AD DNS SRV records and DC locator | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md` | AD SRV and DC locator troubleshooting |
| Configure DNS dynamic updates | `09_Configure_DNS_Dynamic_Updates.md` | Dynamic DNS registration troubleshooting |
| Configure DNS aging and scavenging | `10_Configure_DNS_Aging_And_Scavenging.md` | Stale record and timestamp troubleshooting |
| Configure DNS zone transfers, secondary, and stub zones | `11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md` | Secondary/stub/transfer troubleshooting |
| Configure DNS delegation | `12_Configure_DNS_Delegation.md` | Delegation, NS, and glue troubleshooting |
| Configure DNS client settings and name resolution | `13_Configure_DNS_Client_Settings_And_Name_Resolution.md` | Client resolver and suffix troubleshooting |
| Monitor DNS events, logs, statistics, and cache | `14_Monitor_DNS_Events_Logs_Statistics_And_Cache.md` | Evidence collection for DNS incidents |
| Backup, export, restore DNS zones and server config | `15_Backup_Export_Restore_DNS_Zones_And_Server_Config.md` | Restore path for deleted or corrupted DNS records |
| Troubleshoot DNS name resolution failures | `16_Troubleshoot_DNS_Name_Resolution_Failures.md` | End-to-end DNS client, server, zone, cache, forwarder, and AD DNS root-cause workflow |