00_DNS_Index.md
# 00_DNS_Index

# 00_DNS_Index_Index
00_DNS_Index.md
00_DNS_Index
00_DNS_Index_Source_Basis
00_DNS_Index_Mental_Model
00_DNS_Index_Planning_Table
00_DNS_Index_Configuration_Checklist
00_DNS_Index_DNS_Suite_Map
00_DNS_Index_Build_Order
00_DNS_Index_Dependency_Matrix
00_DNS_Index_Validation_Matrix
00_DNS_Index_Naming_Standards
00_DNS_Index_Operator_Runbook_Skeleton
00_DNS_Index_Verification_Commands
00_DNS_Index_Rollback
00_DNS_Index_Failure_Checks
00_DNS_Index_Related_Labs

# 00_DNS_Index_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server DNS Server role | DNS role installation, zones, records, forwarding, diagnostics | Full DNS operational suite |
| Windows Server DnsServer PowerShell module | DNS Server cmdlets | Repeatable DNS configuration and verification |
| Windows Server ServerManager module | Role and RSAT installation | DNS Server role and management tool deployment |
| Active Directory Domain Services | AD-integrated DNS, secure dynamic updates, replication | Domain DNS behavior and DNS zone replication |
| Windows DNS client stack | Resolver settings, suffixes, cache, DoH behavior | Client-side validation and troubleshooting |
| DNS operational practice | Authoritative DNS, recursive DNS, forwarding, logging, DNSSEC, zone transfers | Real support workflow across production DNS |

# 00_DNS_Index_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS suite | Ordered set of workbooks that build, operate, secure, monitor, and troubleshoot Windows DNS |
| Authoritative DNS | DNS server owns zone data and answers for that namespace |
| Recursive DNS | DNS server resolves names on behalf of clients by using cache, root hints, or forwarders |
| Forward lookup zone | Name to IP or name to DNS data |
| Reverse lookup zone | IP to name using PTR records |
| Resource records | DNS data objects such as A, AAAA, CNAME, MX, NS, PTR, SRV, and TXT |
| Forwarders | Upstream DNS servers used for recursion |
| Conditional forwarders | Upstream DNS path for a specific namespace |
| AD-integrated DNS | DNS zone data stored in Active Directory |
| Dynamic updates | Clients or DHCP register records automatically |
| DNS client resolution | Client-side suffix, cache, server list, and query behavior |
| Diagnostics | Logs, events, query logging, packet flow, and resolver tests |
| DNS policies | Windows DNS response behavior controlled by policy rules |
| Zone transfers | Movement of zone data from primary/master to secondary servers |
| Replication | AD replication of AD-integrated DNS zones |
| DNSSEC | DNS data signing and validation |
| DoH | DNS transport encryption from the client resolver path |
| First rule | Build DNS in dependency order instead of jumping into advanced features |
| Blunt rule | If basic zones and records are broken, DNSSEC, DoH, policies, and logging will only make troubleshooting noisier |

# 00_DNS_Index_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS suite folder | `2. Windows Servers/DNS` | `<dns-suite-folder>` |
| DNS server | `DC1` | `<dns-server>` |
| Secondary DNS server | `DNS2` | `<secondary-dns-server>` |
| Management host | `MGMT01` | `<management-host>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Forward zone | `corp.local` | `<forward-zone>` |
| Reverse zone | `10.10.10.in-addr.arpa` | `<reverse-zone>` |
| DNS server IP | `10.10.10.10` | `<dns-server-ip>` |
| Secondary DNS IP | `10.10.10.20` | `<secondary-dns-ip>` |
| Client test host | `WIN11-01` | `<client-host>` |
| Test host record | `app01.corp.local` | `<test-fqdn>` |
| Test host IP | `10.10.10.50` | `<test-ip>` |
| AD replication scope | `Domain` | `<Domain / Forest / Legacy / Custom>` |
| Dynamic update model | `Secure` | `<None / Secure / NonsecureAndSecure>` |
| Forwarder target | `1.1.1.1` | `<forwarder-ip>` |
| Conditional namespace | `partner.local` | `<conditional-zone>` |
| Logging path | `C:\DNS-Diagnostics` | `<diagnostic-path>` |
| Rollback standard | Export before change | `<rollback-standard>` |

# 00_DNS_Index_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Create DNS suite folder | Management Host | `New-Item -ItemType Directory -Force -Path ".\2. Windows Servers\DNS"` | DNS suite folder exists |
| 2 | Create index workbook | Management Host | `New-Item -ItemType File -Force -Path ".\2. Windows Servers\DNS\00_DNS_Index.md"` | DNS index file exists |
| 3 | Confirm DNS Server role plan | DNS Server | `Get-WindowsFeature -Name DNS,RSAT-DNS-Server` | DNS role and tools state known |
| 4 | Confirm DNS service baseline | DNS Server | `Get-Service DNS -ErrorAction SilentlyContinue` | DNS service state known |
| 5 | Confirm network baseline | DNS Server | `Get-NetIPConfiguration` | DNS server IP plan is validated |
| 6 | Confirm AD status if domain DNS | Domain Controller | `Get-ADDomain; Get-ADForest` | Domain and forest context known |
| 7 | Confirm DNS zones baseline | DNS Server | `Get-DnsServerZone -ErrorAction SilentlyContinue` | Existing DNS zones documented |
| 8 | Confirm DNS client test host | Client | `ipconfig /all` | Client DNS server path is known |
| 9 | Build task 01 first | DNS Server | `Install-WindowsFeature -Name DNS -IncludeManagementTools` | DNS role and tools are installed |
| 10 | Build task 02 second | DNS Server | `Add-DnsServerPrimaryZone -Name "<forward-zone>" -ReplicationScope Domain -DynamicUpdate Secure -PassThru` | Forward zone exists |
| 11 | Build task 03 third | DNS Server | `Add-DnsServerPrimaryZone -NetworkId "<network-id>" -ReplicationScope Domain -DynamicUpdate Secure -PassThru` | Reverse zone exists |
| 12 | Build task 04 fourth | DNS Server | `Add-DnsServerResourceRecordA -ZoneName "<forward-zone>" -Name "<host>" -IPv4Address "<ip>" -PassThru` | Resource records can be created |
| 13 | Build recursion path | DNS Server | `Get-DnsServerForwarder; Get-DnsServerRootHint` | Forwarder and root hint posture known |
| 14 | Build AD-integrated DNS path | Domain Controller | `Get-DnsServerZone \| Where-Object IsDsIntegrated -eq $true` | AD-integrated zones known |
| 15 | Build dynamic update path | DNS Server / DHCP Server | `Get-DnsServerZone -Name "<forward-zone>" \| Format-List *` | Dynamic update model known |
| 16 | Build troubleshooting path | Client / DNS Server | `Resolve-DnsName "<test-fqdn>" -Server "<dns-server>"` | Client resolution validated |
| 17 | Build diagnostics path | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 50` | Event logging baseline exists |
| 18 | Build security and policy path | DNS Server | `Get-DnsServerQueryResolutionPolicy -ErrorAction SilentlyContinue` | Policy state known |
| 19 | Build advanced DNS path | DNS Server | `Get-DnsServerDnsSecZoneSetting -ZoneName "<forward-zone>" -ErrorAction SilentlyContinue` | DNSSEC state known |
| 20 | Document completion state | Operator | `Record completed workbooks, lab date, server names, and validation results` | DNS suite progress is tracked |

# 00_DNS_Index_DNS_Suite_Map
| Task | Workbook | Purpose | Depends On | Output |
|---:|---|---|---|---|
| 00 | `00_DNS_Index.md` | Suite map, build order, validation matrix, and dependency control | None | DNS suite control document |
| 01 | `01_Install_DNS_Server_Role_And_Management_Tools.md` | Install DNS Server role, RSAT tools, service baseline, and firewall sanity checks | 00 | DNS role ready |
| 02 | `02_Create_Forward_Lookup_Zone.md` | Create authoritative forward DNS namespace | 01 | Forward zone ready |
| 03 | `03_Create_Reverse_Lookup_Zone_And_PTR_Records.md` | Create reverse lookup zone and PTR baseline | 01, 02 | Reverse zone and PTRs ready |
| 04 | `04_Create_And_Manage_DNS_Resource_Records.md` | Create and manage A, AAAA, CNAME, MX, NS, PTR, SRV, and TXT records | 02, 03 | DNS records ready |
| 05 | `05_Configure_DNS_Forwarders_And_Root_Hints.md` | Configure recursive DNS path through forwarders or root hints | 01 | Recursion path ready |
| 06 | `06_Configure_DNS_Conditional_Forwarders.md` | Configure namespace-specific forwarding | 01, 05 | Conditional namespace resolution ready |
| 07 | `07_Configure_AD_Integrated_DNS_Zones.md` | Store DNS zones in AD and validate replication scope | 01, 02 | AD-integrated DNS ready |
| 08 | `08_Configure_DNS_Dynamic_Updates.md` | Configure secure dynamic updates and DHCP update behavior | 02, 07 | Dynamic record registration ready |
| 09 | `09_Troubleshoot_DNS_Client_Resolution.md` | Troubleshoot client DNS server list, suffix, cache, and query results | 01 through 08 | Client resolution workflow |
| 10 | `10_Configure_DNS_Client_Settings_And_Suffix_Search.md` | Configure resolver addresses and suffix search behavior | 01, 09 | Client resolver baseline |
| 11 | `11_Configure_DNS_Aging_And_Scavenging.md` | Configure stale record cleanup | 04, 08 | Record lifecycle control |
| 12 | `12_Configure_DNS_Zone_Delegation.md` | Delegate child DNS namespaces | 02, 04 | Delegated namespace ready |
| 13 | `13_Configure_DNS_Stub_Zones.md` | Use stub zones for delegated namespace awareness | 01, 12 | Stub zone resolution ready |
| 14 | `14_Configure_DNS_Secondary_Zones_And_Zone_Transfers.md` | Configure secondary zones and transfer permissions | 02, 04 | Secondary DNS ready |
| 15 | `15_Configure_DNS_Round_Robin_And_Netmask_Ordering.md` | Tune multi-record response behavior | 04 | Response behavior baseline |
| 16 | `16_Configure_DNS_Policies_And_Split_Brain_DNS.md` | Configure DNS policies and split response behavior | 02, 04 | Policy-based DNS ready |
| 17 | `17_Troubleshoot_DNS_Server_Startup_Zones_And_Events.md` | Troubleshoot DNS service startup, zone loading, and event logs | 01 through 04 | Server-side troubleshooting workflow |
| 18 | `18_Configure_DNS_Query_Logging_And_Diagnostics.md` | Enable and read DNS diagnostics | 01 | DNS diagnostic workflow |
| 19 | `19_Configure_DNS_Response_Rate_Limiting_RRL.md` | Protect authoritative DNS from query abuse | 01, 02 | RRL baseline |
| 20 | `20_Configure_DNS_DoH_And_Encryption_Settings.md` | Configure Windows client DNS over HTTPS behavior | 09, 10 | Encrypted client resolver path |
| 21 | `21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication.md` | Troubleshoot DNSSEC, DNS policies, transfers, and AD replication | 02, 04, 07, 14, 16, 18 | Advanced DNS troubleshooting workflow |

# 00_DNS_Index_Build_Order
| Phase | Tasks | Goal | Stop Condition |
|---:|---|---|---|
| 1 | 00, 01 | Establish DNS suite and install DNS role/tools | DNS service runs and tools work |
| 2 | 02, 03, 04 | Build authoritative DNS foundation | Forward, reverse, and records resolve |
| 3 | 05, 06 | Build recursion and forwarding behavior | Internal and external namespaces resolve correctly |
| 4 | 07, 08 | Build AD-integrated DNS and dynamic updates | AD zones replicate and clients register records |
| 5 | 09, 10 | Build client resolver troubleshooting and suffix behavior | Clients query correct DNS path |
| 6 | 11, 12, 13, 14 | Build lifecycle, delegation, stub, and secondary zone behavior | Stale records, delegations, and transfers are controlled |
| 7 | 15, 16 | Build response behavior and DNS policies | Round robin, netmask ordering, split DNS, and policies work as expected |
| 8 | 17, 18 | Build diagnostics and server troubleshooting | Logs and events explain DNS behavior |
| 9 | 19, 20, 21 | Build advanced security, encryption, and DNSSEC troubleshooting | RRL, DoH, DNSSEC, transfers, and replication are supportable |

# 00_DNS_Index_Dependency_Matrix
| Feature Area | Required Beforehand | Do Not Start Until |
|---|---|---|
| DNS role install | Static IP and admin access | Server baseline is known |
| Forward zones | DNS role installed | DNS service responds locally |
| Reverse zones | DNS role installed and subnet plan known | Correct network ID is known |
| Resource records | Forward or reverse zone exists | Duplicate records are checked |
| Forwarders | DNS role installed | Upstream resolver IPs are approved |
| Conditional forwarders | Forwarder design known | Target namespace and master DNS IPs are known |
| AD-integrated DNS | AD DS healthy | Replication scope is chosen |
| Dynamic updates | AD-integrated zone for secure updates | DHCP and client update ownership is understood |
| Client troubleshooting | DNS server exists | Client DNS server list is known |
| Aging and scavenging | Dynamic update behavior understood | Static and dynamic records are inventoried |
| Delegation | Parent and child namespace plan known | Child authoritative DNS servers exist |
| Stub zones | Delegated namespace exists | Master DNS servers are known |
| Secondary zones | Primary zone exists | TCP 53 and transfer ACLs are ready |
| Round robin | Multiple same-name records exist | Load-sharing intent is documented |
| DNS policies | Base DNS resolution works | Policy match rules are defined |
| Query logging | DNS role installed | Log storage and event review process exist |
| RRL | Authoritative zone exists | Legitimate query patterns are understood |
| DoH | Client resolver design known | DoH endpoint and fallback behavior are approved |
| DNSSEC | Zone baseline is stable | Key, DS, signing, transfer, and replication plan is documented |

# 00_DNS_Index_Validation_Matrix
| Validation Area | Command | Good Result |
|---|---|---|
| DNS role installed | `Get-WindowsFeature DNS` | Installed |
| DNS tools installed | `Get-WindowsFeature RSAT-DNS-Server` | Installed |
| DNS service running | `Get-Service DNS` | Running |
| DNS cmdlets available | `Get-Command -Module DnsServer` | Cmdlets return |
| DNS server object responds | `Get-DnsServer -ComputerName "<dns-server>"` | Server object returns |
| Zones visible | `Get-DnsServerZone -ComputerName "<dns-server>"` | Expected zones show |
| Forward zone works | `Resolve-DnsName "<host>.<zone>" -Server "<dns-server>"` | Expected IP returns |
| Reverse zone works | `Resolve-DnsName "<ip-address>" -Server "<dns-server>"` | Expected PTR returns |
| Records exist | `Get-DnsServerResourceRecord -ZoneName "<zone>"` | Expected records return |
| Recursion works | `Resolve-DnsName "www.microsoft.com" -Server "<dns-server>"` | Public answer returns |
| Conditional forwarding works | `Resolve-DnsName "<host>.<conditional-zone>" -Server "<dns-server>"` | Expected remote namespace answer returns |
| AD DNS replicates | `repadmin /replsummary` | No replication failures |
| Client resolver correct | `ipconfig /all` | Expected DNS server list and suffixes |
| DNS logs readable | `Get-WinEvent -LogName "DNS Server" -MaxEvents 50` | Events return |
| TCP 53 reachable | `Test-NetConnection "<dns-server>" -Port 53` | TCP test succeeds |
| DNSSEC state readable | `Get-DnsServerDnsSecZoneSetting -ZoneName "<zone>"` | DNSSEC object returns if configured |
| DoH client state readable | `Get-DnsClientDohServerAddress` | DoH mappings return if configured |

# 00_DNS_Index_Naming_Standards
| Object | Standard | Example |
|---|---|---|
| Workbook file | `##_Verb_Object_And_Context.md` | `02_Create_Forward_Lookup_Zone.md` |
| Workbook title | Same as filename without `.md` | `02_Create_Forward_Lookup_Zone` |
| Section heading | `<Workbook_Name>_<Section>` | `02_Create_Forward_Lookup_Zone_Mental_Model` |
| Forward zone | DNS namespace FQDN | `corp.local` |
| Reverse zone IPv4 | Reversed network plus `in-addr.arpa` | `10.10.10.in-addr.arpa` |
| Host A record | Short host label inside zone | `app01` |
| FQDN | Host plus zone | `app01.corp.local` |
| PTR owner | Host portion inside reverse zone | `50` |
| PTR target | FQDN with trailing dot preferred | `app01.corp.local.` |
| SRV record | `_service._protocol` | `_ldap._tcp` |
| TXT record | Provider or policy name | `_dmarc` |
| Evidence folder | `C:\DNS-<Purpose>` | `C:\DNS-Diagnostics` |
| Export file | `<zone>-<record-type>-<date>.csv` | `corp.local-A-2026-06-12.csv` |

# 00_DNS_Index_Operator_Runbook_Skeleton
```powershell
# DNS suite operator baseline.
# Run from an elevated PowerShell session on the DNS server or management host.

$DnsServer = "DC1"
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$TestHost = "app01"
$TestFqdn = "$TestHost.$ForwardZone"
$TestIp = "10.10.10.50"
$EvidencePath = "C:\DNS-Suite-Baseline"

New-Item -ItemType Directory -Force -Path $EvidencePath

# 01 Role and tools.
Get-WindowsFeature -Name DNS,RSAT-DNS-Server |
  Out-File "$EvidencePath\01-role-tools.txt"

Get-Service DNS |
  Format-List * |
  Out-File "$EvidencePath\01-dns-service.txt"

# 02 Forward zone.
Get-DnsServerZone -ComputerName $DnsServer -Name $ForwardZone -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\02-forward-zone.txt"

# 03 Reverse zone.
Get-DnsServerZone -ComputerName $DnsServer -Name $ReverseZone -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\03-reverse-zone.txt"

# 04 Records.
Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ForwardZone -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\04-forward-records.txt"

Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\04-reverse-records.txt"

# Resolution tests.
Resolve-DnsName $TestFqdn -Server $DnsServer -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\resolution-forward.txt"

Resolve-DnsName $TestIp -Server $DnsServer -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\resolution-reverse.txt"

Resolve-DnsName "www.microsoft.com" -Server $DnsServer -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\resolution-recursion.txt"

# AD replication if domain controller.
repadmin /replsummary |
  Out-File "$EvidencePath\ad-replsummary.txt"

# DNS logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 |
  Select-Object TimeCreated, Id, ProviderName, Message |
  Out-File "$EvidencePath\dns-server-events.txt"
```

# 00_DNS_Index_Verification_Commands
```powershell
# Role and tools.
Get-WindowsFeature -Name DNS,RSAT-DNS-Server
Get-Service DNS
Import-Module DnsServer
Get-Command -Module DnsServer

# DNS server inventory.
Get-DnsServer
Get-DnsServerZone
Get-DnsServerForwarder
Get-DnsServerRootHint

# Forward zone and records.
Get-DnsServerZone -Name "corp.local"
Get-DnsServerResourceRecord -ZoneName "corp.local"
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType SOA
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType NS
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType A
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType CNAME
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType MX
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType TXT
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType SRV

# Reverse zone and PTR records.
Get-DnsServerZone -Name "10.10.10.in-addr.arpa"
Get-DnsServerResourceRecord -ZoneName "10.10.10.in-addr.arpa" -RRType PTR

# Resolution.
Resolve-DnsName "app01.corp.local" -Server "DC1"
Resolve-DnsName "10.10.10.50" -Server "DC1"
Resolve-DnsName "www.microsoft.com" -Server "DC1"

# Client resolver.
ipconfig /all
Clear-DnsClientCache
ipconfig /flushdns
Get-DnsClientServerAddress
Get-DnsClientGlobalSetting

# AD-integrated DNS.
repadmin /replsummary
repadmin /showrepl
dcdiag /test:dns /v

# Diagnostics.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Directory Service" -MaxEvents 100

# Port checks.
Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue
Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue
Test-NetConnection "DC1" -Port 53

# Advanced checks when configured.
Get-DnsServerQueryResolutionPolicy -ErrorAction SilentlyContinue
Get-DnsServerDnsSecZoneSetting -ZoneName "corp.local" -ErrorAction SilentlyContinue
Get-DnsClientDohServerAddress -ErrorAction SilentlyContinue
```

# 00_DNS_Index_Rollback
| Step | Action | PowerShell | Expected Result |
|---:|---|---|---|
| 1 | Capture DNS suite state | `Get-DnsServerZone; Get-DnsServerForwarder; Get-Service DNS` | Current DNS state is documented |
| 2 | Export zone records before rollback | `Get-DnsServerResourceRecord -ZoneName "<zone>" \| Export-Csv C:\DNS-Rollback-Records.csv -NoTypeInformation` | Record inventory is saved |
| 3 | Remove test records only | `Remove-DnsServerResourceRecord -ZoneName "<zone>" -InputObject $Record -Force` | Test record is removed |
| 4 | Remove lab-only reverse zone | `Remove-DnsServerZone -Name "<reverse-zone>" -Force` | Reverse zone is removed |
| 5 | Remove lab-only forward zone | `Remove-DnsServerZone -Name "<forward-zone>" -Force` | Forward zone is removed |
| 6 | Restore forwarders | `Set-DnsServerForwarder -IPAddress "<approved-forwarder-ip>"` | Approved forwarders restored |
| 7 | Clear DNS client cache | `Clear-DnsClientCache; ipconfig /flushdns` | Client cache is cleared |
| 8 | Restart DNS service if needed | `Restart-Service DNS` | DNS service restarts cleanly |
| 9 | Force AD replication if AD-integrated changes were made | `repadmin /syncall /AdeP` | AD replication triggered |
| 10 | Verify final state | `Get-DnsServerZone; Resolve-DnsName "<test-name>" -Server "<dns-server>"` | DNS state matches rollback target |
| 11 | Document rollback | `Record removed zones, removed records, restored settings, and unresolved issues` | Rollback is supportable later |

# 00_DNS_Index_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| DNS suite feels out of order | Tasks were built without dependency order | Review build order table | Start at 01, then 02, 03, 04 before advanced work |
| Role installed but cmdlets missing | Management tools not installed | `Get-WindowsFeature RSAT-DNS-Server` | Install management tools |
| DNS service not running | Role install or service startup failure | `Get-Service DNS`; DNS Server event log | Start service and fix startup events |
| Forward lookup fails | Missing forward zone or A record | `Get-DnsServerZone`; `Get-DnsServerResourceRecord` | Create zone or record |
| Reverse lookup fails | Missing reverse zone or PTR record | `Get-DnsServerZone`; `Resolve-DnsName <ip>` | Create reverse zone or PTR |
| Public recursion fails | Forwarder or root hint issue | `Get-DnsServerForwarder`; `Resolve-DnsName www.microsoft.com -Server <dns>` | Correct forwarder or root hint path |
| Conditional namespace fails | Conditional forwarder missing or wrong master IP | `Get-DnsServerConditionalForwarderZone` | Correct conditional forwarder |
| Client resolves using wrong DNS | Client DNS server list wrong | `ipconfig /all` | Correct client DNS server settings |
| AD DNS not replicating | AD replication issue | `repadmin /replsummary` | Fix AD replication before DNS changes |
| Dynamic records not updating | Zone dynamic update or DHCP update issue | Check zone settings and DHCP DNS settings | Configure secure dynamic updates and DHCP ownership |
| Stale records remain | Aging or scavenging not configured | Check timestamps and scavenging settings | Configure aging and scavenging carefully |
| Zone transfer fails | TCP 53, transfer ACL, or notify issue | `Test-NetConnection <server> -Port 53`; SOA compare | Fix ACL, notify, and firewall |
| DNS policy gives unexpected answer | Policy match rule too broad | `Get-DnsServerQueryResolutionPolicy` | Narrow or disable policy |
| DNSSEC breaks resolution | Broken signing, stale DS, or validation issue | `Test-DnsServerDnsSecZoneSetting` | Fix DNSSEC chain or re-sign |
| DoH bypasses internal DNS | Client configured for external DoH resolver | `Get-DnsClientDohServerAddress`; client DNS settings | Restore internal DNS or design split resolver path |
| Logs do not show queries | Query logging not enabled or wrong server queried | Check client DNS path and diagnostics config | Query correct server and enable diagnostics |

# 00_DNS_Index_Related_Labs
| Lab                                                            | Relationship                                                                    |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 01_Install_DNS_Server_Role_And_Management_Tools                | Installs the DNS role and management tools used by every later DNS workbook     |
| 02_Create_Forward_Lookup_Zone                                  | Builds the authoritative forward namespace                                      |
| 03_Create_Reverse_Lookup_Zone_And_PTR_Records                  | Builds reverse lookup support                                                   |
| 04_Create_And_Manage_DNS_Resource_Records                      | Adds the actual DNS data objects                                                |
| 05_Configure_DNS_Forwarders_And_Root_Hints                     | Adds recursive resolution behavior                                              |
| 06_Configure_DNS_Conditional_Forwarders                        | Adds namespace-specific forwarding behavior                                     |
| 07_Configure_AD_Integrated_DNS_Zones                           | Adds AD storage and replication behavior                                        |
| 08_Configure_DNS_Dynamic_Updates                               | Adds automatic client and DHCP registration behavior                            |
| 09_Troubleshoot_DNS_Client_Resolution                          | Troubleshoots resolver path from the client side                                |
| 10_Configure_DNS_Client_Settings_And_Suffix_Search             | Controls DNS client server list and suffix behavior                             |
| 11_Configure_DNS_Aging_And_Scavenging                          | Controls stale record cleanup                                                   |
| 12_Configure_DNS_Zone_Delegation                               | Builds delegated child namespaces                                               |
| 13_Configure_DNS_Stub_Zones                                    | Builds delegated namespace awareness through stub zones                         |
| 14_Configure_DNS_Secondary_Zones_And_Zone_Transfers            | Builds secondary DNS and transfer behavior                                      |
| 15_Configure_DNS_Round_Robin_And_Netmask_Ordering              | Controls multi-record answer behavior                                           |
| 16_Configure_DNS_Policies_And_Split_Brain_DNS                  | Controls policy-based DNS answers                                               |
| 17_Troubleshoot_DNS_Server_Startup_Zones_And_Events            | Troubleshoots DNS service and zone loading                                      |
| 18_Configure_DNS_Query_Logging_And_Diagnostics                 | Builds DNS logging and evidence collection                                      |
| 19_Configure_DNS_Response_Rate_Limiting_RRL                    | Adds authoritative DNS rate limiting                                            |
| 20_Configure_DNS_DoH_And_Encryption_Settings                   | Adds DNS client transport encryption context                                    |
| 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication | Advanced troubleshooting across DNSSEC, policies, transfers, and AD replication |
