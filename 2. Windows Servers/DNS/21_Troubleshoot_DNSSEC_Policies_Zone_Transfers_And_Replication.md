21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication.md
# 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication

# 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Index
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication.md
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Source_Basis
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Mental_Model
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Planning_Table
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Configuration_Checklist
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_DNSSEC_Health_Check_Skeleton
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Zone_Transfer_Check_Skeleton
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_AD_Integrated_Replication_Check_Skeleton
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Resign_And_Metadata_Check_Skeleton
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Verification_Commands
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Rollback
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Failure_Checks
21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Related_Labs

# 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Get-DnsServerDnsSecZoneSetting | Reading DNSSEC zone settings, signing metadata, key master state, NSEC/NSEC3 settings, and rollover metadata |
| Microsoft Learn | Test-DnsServerDnsSecZoneSetting | Validating DNSSEC configuration health for a zone |
| Microsoft Learn | Get-DnsServerSigningKey | Inspecting KSK and ZSK state |
| Microsoft Learn | Add-DnsServerSigningKey | Adding missing KSK or ZSK records to a signed zone |
| Microsoft Learn | Invoke-DnsServerZoneSign | Signing unsigned zones and re-signing signed zones |
| Microsoft Learn | Invoke-DnsServerZoneUnsign | Removing DNSSEC signing when rollback is required |
| Microsoft Learn | Set-DnsServerPrimaryZone | Configuring primary-zone transfer, notify, secondary server, dynamic update, and AD replication scope settings |
| Windows Server operational practice | dcdiag, repadmin, Resolve-DnsName, DNS Server event logs | Troubleshooting DNSSEC, zone transfer, AD-integrated DNS replication, and client validation behavior |

# 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNSSEC | DNS data integrity and authenticity mechanism using signed DNS records |
| Signed zone | Zone containing DNSSEC records such as DNSKEY, RRSIG, NSEC/NSEC3, and sometimes DS |
| KSK | Key Signing Key; signs the DNSKEY record set |
| ZSK | Zone Signing Key; signs ordinary zone record sets |
| RRSIG | Signature record proving a record set was signed |
| DNSKEY | Public key material used by validating resolvers |
| DS record | Delegation signer record placed in the parent zone to build chain of trust |
| NSEC / NSEC3 | Authenticated denial-of-existence methods |
| Key master | DNS server responsible for DNSSEC key management for the zone |
| Signing metadata | DNSSEC signing state that can be exported or compared between DNS servers |
| Re-sign | Refreshing or rebuilding DNSSEC signatures on an already signed zone |
| Zone transfer | AXFR or IXFR movement of zone data from primary/master to secondary DNS servers |
| Notify | DNS master notification to secondary servers that zone data changed |
| Secure secondaries | Zone transfer restriction model on the Windows DNS primary zone |
| SecondaryServers | Explicit list of servers allowed to receive zone transfers |
| NotifyServers | Explicit list of secondary servers to notify after changes |
| AD-integrated zone | DNS zone stored in Active Directory instead of a flat zone file |
| ReplicationScope | AD partition scope used by an AD-integrated DNS zone: Domain, Forest, Legacy, or Custom |
| DomainDnsZones | Domain-wide DNS application partition |
| ForestDnsZones | Forest-wide DNS application partition |
| First rule | DNSSEC failure, transfer failure, and AD replication failure are separate problems until proven connected |
| Blunt rule | Do not re-sign, unsign, or change replication scope until you have captured current zone state and event evidence |

# 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Zone name | `corp.local` | `<zone-name>` |
| Primary DNS server | `DC1` | `<primary-dns-server>` |
| Secondary DNS server | `DNS2` | `<secondary-dns-server>` |
| Primary DNS IP | `10.10.10.10` | `<primary-dns-ip>` |
| Secondary DNS IP | `10.10.10.20` | `<secondary-dns-ip>` |
| Zone type | AD-integrated primary | `<primary / secondary / stub / AD-integrated>` |
| Zone storage | AD DS | `<AD DS / file-backed>` |
| Replication scope | Domain | `<Domain / Forest / Legacy / Custom>` |
| Expected transfer mode | Transfer to secure servers | `<NoTransfer / TransferAnyServer / TransferToZoneNameServer / TransferToSecureServers>` |
| Expected notify mode | NotifyServers | `<NoNotify / Notify / NotifyServers>` |
| DNSSEC status | Signed | `<signed / unsigned>` |
| Key master | `DC1` | `<key-master-server>` |
| Expected KSK algorithm | `RsaSha256` | `<ksk-algorithm>` |
| Expected ZSK algorithm | `RsaSha256` | `<zsk-algorithm>` |
| Expected denial method | `NSec3` | `<NSEC / NSEC3>` |
| Parent DS required | No for internal AD zone | `<yes-no>` |
| Client validation test | `Resolve-DnsName <record> -DnssecOk` | `<test-command>` |
| Event log capture path | `C:\DNS-Troubleshooting` | `<evidence-path>` |
| Safe rollback action | Restore transfer settings only | `<rollback-plan>` |

# 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | DNS Server | `New-Item -ItemType Directory -Force -Path C:\DNS-Troubleshooting` | Evidence path exists |
| 2 | Confirm DNS Server module | DNS Server | `Get-Module -ListAvailable DnsServer` | DnsServer module is available |
| 3 | Confirm target zone exists | DNS Server | `Get-DnsServerZone -Name "<zone-name>" -ComputerName "<primary-dns-server>"` | Zone exists on expected server |
| 4 | Capture zone baseline | DNS Server | `Get-DnsServerZone -Name "<zone-name>" -ComputerName "<primary-dns-server>" \| Format-List *` | Zone type, DS integration, and signed state are captured |
| 5 | Capture SOA record | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType SOA -ComputerName "<primary-dns-server>"` | SOA serial and master data are visible |
| 6 | Capture NS records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NS -ComputerName "<primary-dns-server>"` | Authoritative servers are listed |
| 7 | Capture DNSSEC settings | DNS Server | `Get-DnsServerDnsSecZoneSetting -ZoneName "<zone-name>" -ComputerName "<primary-dns-server>"` | DNSSEC setting object returns |
| 8 | Validate DNSSEC settings | DNS Server | `Test-DnsServerDnsSecZoneSetting -ZoneName "<zone-name>" -ComputerName "<primary-dns-server>"` | Validation object returns without obvious failure |
| 9 | Capture signing metadata | DNS Server | `Get-DnsServerDnsSecZoneSetting -ZoneName "<zone-name>" -SigningMetadata -IncludeKSKMetadata -ComputerName "<primary-dns-server>"` | Signing metadata and KSK metadata are visible |
| 10 | Capture signing keys | DNS Server | `Get-DnsServerSigningKey -ZoneName "<zone-name>" -ComputerName "<primary-dns-server>"` | KSK and ZSK are visible |
| 11 | Confirm DNSSEC record sets | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType DNSKEY`; `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType RRSIG` | DNSKEY and RRSIG records exist if zone is signed |
| 12 | Test signed record resolution | Client / DNS Server | `Resolve-DnsName "<record>.<zone-name>" -Server "<primary-dns-ip>" -DnssecOk` | Record resolves and DNSSEC data is requested |
| 13 | Check zone transfer settings | DNS Server | `Get-DnsServerZone -Name "<zone-name>" \| Format-List *` | Transfer and notify settings are reviewed |
| 14 | Confirm transfer ACL intent | DNS Server | `Set-DnsServerPrimaryZone -Name "<zone-name>" -SecureSecondaries TransferToSecureServers -SecondaryServers "<secondary-dns-ip>" -Notify NotifyServers -NotifyServers "<secondary-dns-ip>" -WhatIf` | Intended change preview is sane |
| 15 | Test transfer port reachability | Secondary DNS | `Test-NetConnection "<primary-dns-ip>" -Port 53` | TCP 53 is reachable from secondary to primary |
| 16 | Confirm secondary zone configuration | Secondary DNS | `Get-DnsServerZone -Name "<zone-name>" -ComputerName "<secondary-dns-server>"` | Secondary zone exists if file-backed transfer design is used |
| 17 | Compare SOA serial primary vs secondary | DNS Servers | `Resolve-DnsName "<zone-name>" -Type SOA -Server "<primary-dns-ip>"; Resolve-DnsName "<zone-name>" -Type SOA -Server "<secondary-dns-ip>"` | SOA serials match or expected lag is identified |
| 18 | Check AD-integrated replication scope | DNS Server | `Get-DnsServerZone -Name "<zone-name>" \| Select-Object ZoneName,IsDsIntegrated,ReplicationScope` | AD integration and scope are confirmed |
| 19 | Check AD replication health | Domain Controller | `repadmin /showrepl`; `repadmin /replsummary` | AD replication is healthy or failures are visible |
| 20 | Run DNS diagnostics | Domain Controller | `dcdiag /test:dns /v` | DNS registration, delegation, and service checks are visible |
| 21 | Compare zone visibility across DC DNS servers | DNS Servers | `Get-DnsServerZone -ComputerName "<dc1>"; Get-DnsServerZone -ComputerName "<dc2>"` | Zone exists on expected DNS servers |
| 22 | Compare DNSSEC records across replicas | DNS Servers | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType DNSKEY -ComputerName "<dc1>"; Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType DNSKEY -ComputerName "<dc2>"` | DNSKEY data matches across replicas |
| 23 | Review DNS Server logs | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | Transfer, signing, loading, or replication errors are visible |
| 24 | Review Directory Service logs if AD-integrated | Domain Controller | `Get-WinEvent -LogName "Directory Service" -MaxEvents 100` | AD replication or directory partition errors are visible |
| 25 | Correct transfer settings only if wrong | Primary DNS | `Set-DnsServerPrimaryZone -Name "<zone-name>" -SecureSecondaries TransferToSecureServers -SecondaryServers "<secondary-dns-ip>" -Notify NotifyServers -NotifyServers "<secondary-dns-ip>" -PassThru` | Zone transfer is restricted to intended secondary servers |
| 26 | Re-sign only if DNSSEC signatures are stale or broken | DNS Server | `Invoke-DnsServerZoneSign -ZoneName "<zone-name>" -DoResign -PassThru -Verbose` | Signed zone is re-signed |
| 27 | Force AD replication only after confirming AD-integrated DNS issue | Domain Controller | `repadmin /syncall /AdeP` | AD replication cycle is triggered |
| 28 | Re-test resolution, DNSSEC, transfer, and replication | DNS / Client | Run verification command block | DNSSEC, transfer, and replication state are clean |
| 29 | Document root cause and final state | Operator | `Record failed layer: DNSSEC / transfer / AD replication / client validation` | Troubleshooting record is complete |

# 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication_DNSSEC_Health_Check_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server.
# Purpose: determine whether the problem is DNSSEC signing, missing records, stale signatures, or client validation.

$ZoneName = "corp.local"
$DnsServer = "DC1"
$TestRecord = "dc1.corp.local"
$EvidencePath = "C:\DNS-Troubleshooting"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm zone exists and capture baseline.
Get-DnsServerZone `
  -Name $ZoneName `
  -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-zone-baseline.txt"

# Capture SOA and NS records.
Get-DnsServerResourceRecord `
  -ZoneName $ZoneName `
  -RRType SOA `
  -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-soa.txt"

Get-DnsServerResourceRecord `
  -ZoneName $ZoneName `
  -RRType NS `
  -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-ns.txt"

# Capture DNSSEC settings.
Get-DnsServerDnsSecZoneSetting `
  -ZoneName $ZoneName `
  -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-dnssec-settings.txt"

# Validate DNSSEC settings.
Test-DnsServerDnsSecZoneSetting `
  -ZoneName $ZoneName `
  -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-dnssec-validation.txt"

# Capture signing metadata and KSK metadata.
Get-DnsServerDnsSecZoneSetting `
  -ZoneName $ZoneName `
  -ComputerName $DnsServer `
  -SigningMetadata `
  -IncludeKSKMetadata |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-signing-metadata.txt"

# Capture signing keys.
Get-DnsServerSigningKey `
  -ZoneName $ZoneName `
  -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-signing-keys.txt"

# Check DNSSEC record types.
Get-DnsServerResourceRecord `
  -ZoneName $ZoneName `
  -RRType DNSKEY `
  -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-dnskey-records.txt"

Get-DnsServerResourceRecord `
  -ZoneName $ZoneName `
  -RRType RRSIG `
  -ComputerName $DnsServer |
  Select-Object -First 20 |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-rrsig-sample.txt"

# Test resolution with DNSSEC OK bit.
Resolve-DnsName `
  -Name $TestRecord `
  -Server $DnsServer `
  -DnssecOk |
  Out-File "$EvidencePath\$ZoneName-resolve-dnssecok.txt"

# Review DNS Server events.
Get-WinEvent `
  -LogName "DNS Server" `
  -MaxEvents 100 |
  Select-Object TimeCreated, Id, ProviderName, Message |
  Out-File "$EvidencePath\dns-server-events.txt"