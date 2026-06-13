03_Create_Reverse_Lookup_Zone_And_PTR_Records.md
# 03_Create_Reverse_Lookup_Zone_And_PTR_Records

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Index
03_Create_Reverse_Lookup_Zone_And_PTR_Records.md
03_Create_Reverse_Lookup_Zone_And_PTR_Records
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Source_Basis
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Mental_Model
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Planning_Table
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Configuration_Checklist
03_Create_Reverse_Lookup_Zone_And_PTR_Records_AD_Integrated_Reverse_Zone_Skeleton
03_Create_Reverse_Lookup_Zone_And_PTR_Records_File_Backed_Reverse_Zone_Skeleton
03_Create_Reverse_Lookup_Zone_And_PTR_Records_PTR_Record_Creation_Skeleton
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Bulk_PTR_Audit_Skeleton
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Verification_Commands
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Rollback
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Failure_Checks
03_Create_Reverse_Lookup_Zone_And_PTR_Records_Related_Labs

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DnsServerPrimaryZone | Creating AD-integrated and file-backed reverse lookup zones |
| Microsoft Learn | Add-DnsServerResourceRecordPtr | Creating PTR records inside reverse lookup zones |
| Microsoft Learn | Get-DnsServerZone | Confirming reverse zone presence and zone type |
| Microsoft Learn | Get-DnsServerResourceRecord | Auditing PTR records inside reverse zones |
| Microsoft Learn | Remove-DnsServerResourceRecord | Removing stale or incorrect PTR records |
| Microsoft Learn | Resolve-DnsName | Validating reverse resolution from a client or server |
| Windows Server operational practice | AD-integrated DNS, DHCP dynamic updates, scavenging, and PTR hygiene | Maintaining correct reverse lookup behavior |

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Reverse lookup zone | DNS zone used to resolve IP addresses back to names |
| PTR record | Pointer record that maps an IP address to a hostname |
| Forward lookup | Name to IP address |
| Reverse lookup | IP address to name |
| `in-addr.arpa` | IPv4 reverse DNS namespace |
| `ip6.arpa` | IPv6 reverse DNS namespace |
| Network ID | IP network and prefix used by Windows to calculate the reverse zone name |
| `/24` IPv4 reverse zone | Common reverse zone size where the PTR record name is the last octet |
| PTR owner name | The IP portion stored inside the reverse zone, such as `50` for `10.10.10.50` in `10.10.10.in-addr.arpa` |
| PTR target | FQDN returned by reverse lookup, such as `app01.corp.local.` |
| AD-integrated reverse zone | Reverse zone stored in Active Directory and replicated between DNS-enabled domain controllers |
| File-backed reverse zone | Reverse zone stored in a DNS zone file on one DNS server |
| Dynamic update | DNS clients or DHCP can create or update records automatically |
| Secure dynamic update | AD-integrated zone update model requiring authenticated update ownership |
| Static PTR | Manually created PTR record |
| Aged PTR | PTR with timestamp, eligible for scavenging |
| Stale PTR | PTR pointing to a host or IP assignment that is no longer valid |
| Forward and reverse match | A record and PTR record agree with each other |
| First rule | Create the reverse zone before creating PTR records |
| Blunt rule | Reverse DNS is not automatic just because forward DNS works |

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server | `DC1` | `<dns-server>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Forward zone | `corp.local` | `<forward-zone>` |
| IPv4 subnet | `10.10.10.0/24` | `<network-id>` |
| Reverse zone name | `10.10.10.in-addr.arpa` | `<reverse-zone>` |
| Replication model | AD-integrated | `<AD-integrated / file-backed>` |
| Replication scope | Domain | `<Domain / Forest / Legacy / Custom>` |
| Dynamic updates | Secure | `<None / Secure / NonsecureAndSecure>` |
| Host name | `app01` | `<host-name>` |
| Host FQDN | `app01.corp.local.` | `<host-fqdn>` |
| IPv4 address | `10.10.10.50` | `<ipv4-address>` |
| PTR record name | `50` | `<ptr-record-name>` |
| TTL | `01:00:00` | `<ttl>` |
| Age record | No for static server records | `<yes-no>` |
| DHCP owns PTR updates | No for static server | `<yes-no>` |
| Rollback action | Remove PTR, remove zone if lab-only | `<rollback-plan>` |

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm DNS Server module | DNS Server | `Get-Module -ListAvailable DnsServer` | DnsServer module is available |
| 2 | Confirm DNS service is running | DNS Server | `Get-Service DNS` | DNS service is running |
| 3 | Inventory existing zones | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>"` | Existing forward and reverse zones are visible |
| 4 | Confirm forward zone exists | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>" -Name "<forward-zone>"` | Forward zone exists |
| 5 | Check whether reverse zone already exists | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>" -Name "<reverse-zone>" -ErrorAction SilentlyContinue` | Existing reverse zone state is known |
| 6 | Create AD-integrated reverse zone | DNS Server | `Add-DnsServerPrimaryZone -ComputerName "<dns-server>" -NetworkId "<network-id>" -ReplicationScope "Domain" -DynamicUpdate "Secure" -PassThru` | Reverse zone is created and stored in AD |
| 7 | Alternative: create file-backed reverse zone | DNS Server | `Add-DnsServerPrimaryZone -ComputerName "<dns-server>" -NetworkId "<network-id>" -ZoneFile "<reverse-zone>.dns" -PassThru` | File-backed reverse zone is created |
| 8 | Verify reverse zone | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>" -Name "<reverse-zone>"` | Reverse zone exists and shows expected type |
| 9 | Confirm A record exists before PTR | DNS Server | `Resolve-DnsName "<host-fqdn>" -Server "<dns-server>"` | Host resolves forward |
| 10 | Check for existing PTR | DNS Server | `Get-DnsServerResourceRecord -ComputerName "<dns-server>" -ZoneName "<reverse-zone>" -Name "<ptr-record-name>" -ErrorAction SilentlyContinue` | Duplicate PTR risk is known |
| 11 | Create PTR record | DNS Server | `Add-DnsServerResourceRecordPtr -ComputerName "<dns-server>" -ZoneName "<reverse-zone>" -Name "<ptr-record-name>" -PtrDomainName "<host-fqdn>" -TimeToLive 01:00:00 -PassThru` | PTR record exists |
| 12 | Optional: create aged PTR | DNS Server | `Add-DnsServerResourceRecordPtr -ComputerName "<dns-server>" -ZoneName "<reverse-zone>" -Name "<ptr-record-name>" -PtrDomainName "<host-fqdn>" -TimeToLive 01:00:00 -AgeRecord -PassThru` | PTR is timestamped and eligible for scavenging |
| 13 | Verify PTR object | DNS Server | `Get-DnsServerResourceRecord -ComputerName "<dns-server>" -ZoneName "<reverse-zone>" -Name "<ptr-record-name>" -RRType PTR` | PTR record data is correct |
| 14 | Verify reverse lookup by IP | Client / DNS Server | `Resolve-DnsName "<ipv4-address>" -Server "<dns-server>"` | IP resolves to expected FQDN |
| 15 | Verify reverse lookup by PTR FQDN | Client / DNS Server | `Resolve-DnsName "<ptr-record-name>.<reverse-zone>" -Type PTR -Server "<dns-server>"` | PTR query returns expected FQDN |
| 16 | Compare forward and reverse | Client / DNS Server | `Resolve-DnsName "<host-fqdn>" -Server "<dns-server>"; Resolve-DnsName "<ipv4-address>" -Server "<dns-server>"` | A and PTR records agree |
| 17 | Confirm replication if AD-integrated | Domain Controller | `repadmin /replsummary` | AD replication is healthy |
| 18 | Check reverse zone on another DNS DC | DNS Server | `Get-DnsServerZone -ComputerName "<other-dns-server>" -Name "<reverse-zone>"` | Zone replicated to expected DNS server |
| 19 | Export reverse zone PTR inventory | DNS Server | `Get-DnsServerResourceRecord -ComputerName "<dns-server>" -ZoneName "<reverse-zone>" -RRType PTR \| Export-Csv C:\DNS-Reverse-PTR.csv -NoTypeInformation` | PTR inventory is documented |
| 20 | Document owner and purpose | Operator | `Record subnet, zone name, PTR target, TTL, update model, and rollback plan` | Reverse DNS is supportable later |

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_AD_Integrated_Reverse_Zone_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server or management workstation with RSAT DNS tools.
# Purpose: create an AD-integrated IPv4 reverse lookup zone.

$DnsServer = "DC1"
$NetworkId = "10.10.10.0/24"
$ExpectedReverseZone = "10.10.10.in-addr.arpa"
$ReplicationScope = "Domain"
$DynamicUpdate = "Secure"

# Confirm DNS Server module and service.
Get-Module -ListAvailable DnsServer
Get-Service DNS -ComputerName $DnsServer

# Inventory existing zones.
Get-DnsServerZone -ComputerName $DnsServer

# Check whether the reverse zone already exists.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ExpectedReverseZone `
  -ErrorAction SilentlyContinue

# Create AD-integrated reverse lookup zone.
Add-DnsServerPrimaryZone `
  -ComputerName $DnsServer `
  -NetworkId $NetworkId `
  -ReplicationScope $ReplicationScope `
  -DynamicUpdate $DynamicUpdate `
  -PassThru

# Verify reverse zone.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ExpectedReverseZone |
  Format-List *

# Optional AD replication health check.
repadmin /replsummary
```

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_File_Backed_Reverse_Zone_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: create a file-backed IPv4 reverse lookup zone.

$DnsServer = "DNS1"
$NetworkId = "10.10.10.0/24"
$ReverseZone = "10.10.10.in-addr.arpa"
$ZoneFile = "10.10.10.in-addr.arpa.dns"

# Confirm module and service.
Get-Module -ListAvailable DnsServer
Get-Service DNS -ComputerName $DnsServer

# Check existing zone state.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ReverseZone `
  -ErrorAction SilentlyContinue

# Create file-backed reverse lookup zone.
Add-DnsServerPrimaryZone `
  -ComputerName $DnsServer `
  -NetworkId $NetworkId `
  -ZoneFile $ZoneFile `
  -PassThru

# Verify zone.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ReverseZone |
  Format-List *

# Confirm zone records.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone
```

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_PTR_Record_Creation_Skeleton
```powershell
# Run in elevated PowerShell.
# Purpose: create matching A and PTR validation path for a static server.

$DnsServer = "DC1"
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"

$HostName = "app01"
$HostFqdn = "app01.corp.local."
$IPv4Address = "10.10.10.50"
$PtrRecordName = "50"
$TTL = [TimeSpan]::FromHours(1)

# Confirm forward and reverse zones.
Get-DnsServerZone -ComputerName $DnsServer -Name $ForwardZone
Get-DnsServerZone -ComputerName $DnsServer -Name $ReverseZone

# Confirm forward record exists.
Resolve-DnsName `
  -Name $HostFqdn `
  -Server $DnsServer

# Check for existing PTR record.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -Name $PtrRecordName `
  -RRType PTR `
  -ErrorAction SilentlyContinue

# Create PTR record.
Add-DnsServerResourceRecordPtr `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -Name $PtrRecordName `
  -PtrDomainName $HostFqdn `
  -TimeToLive $TTL `
  -PassThru

# Verify PTR object.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -Name $PtrRecordName `
  -RRType PTR |
  Format-List *

# Verify reverse lookup by IP.
Resolve-DnsName `
  -Name $IPv4Address `
  -Server $DnsServer

# Verify reverse lookup by full PTR query name.
Resolve-DnsName `
  -Name "$PtrRecordName.$ReverseZone" `
  -Type PTR `
  -Server $DnsServer
```

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Bulk_PTR_Audit_Skeleton
```powershell
# Run on DNS server or management workstation.
# Purpose: export PTR records and identify stale reverse DNS candidates.

$DnsServer = "DC1"
$ReverseZone = "10.10.10.in-addr.arpa"
$ExportPath = "C:\DNS-Reverse-Audit"

New-Item -ItemType Directory -Force -Path $ExportPath

# Export all PTR records.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -RRType PTR |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData |
  Export-Csv "$ExportPath\$ReverseZone-ptr-records.csv" -NoTypeInformation

# Display PTR records with timestamps.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -RRType PTR |
  Where-Object {
    $_.Timestamp -ne $null
  } |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData

# Display static PTR records.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -RRType PTR |
  Where-Object {
    $_.Timestamp -eq $null
  } |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData

# Resolve each PTR target forward to check for mismatches.
$PtrRecords = Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -RRType PTR

foreach ($Record in $PtrRecords) {
  $PtrName = $Record.HostName
  $Target = $Record.RecordData.PtrDomainName

  [PSCustomObject]@{
    PtrName = $PtrName
    PtrTarget = $Target
    ForwardLookup = try {
      (Resolve-DnsName -Name $Target -Server $DnsServer -ErrorAction Stop |
        Where-Object Type -in "A","AAAA" |
        Select-Object -ExpandProperty IPAddress) -join ","
    } catch {
      "FORWARD_LOOKUP_FAILED"
    }
  }
}
```

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Verification_Commands
```powershell
# DNS module and service.
Get-Module -ListAvailable DnsServer
Get-Service DNS

# Zone inventory.
Get-DnsServerZone
Get-DnsServerZone -Name "10.10.10.in-addr.arpa"
Get-DnsServerZone -Name "corp.local"

# Reverse zone record inventory.
Get-DnsServerResourceRecord -ZoneName "10.10.10.in-addr.arpa"
Get-DnsServerResourceRecord -ZoneName "10.10.10.in-addr.arpa" -RRType PTR
Get-DnsServerResourceRecord -ZoneName "10.10.10.in-addr.arpa" -Name "50" -RRType PTR

# Reverse lookup by IP.
Resolve-DnsName "10.10.10.50" -Server "DC1"

# Reverse lookup by PTR owner name.
Resolve-DnsName "50.10.10.10.in-addr.arpa" -Type PTR -Server "DC1"

# Forward lookup comparison.
Resolve-DnsName "app01.corp.local" -Server "DC1"

# Classic nslookup checks.
nslookup 10.10.10.50 DC1
nslookup app01.corp.local DC1

# Client cache reset.
Clear-DnsClientCache
ipconfig /flushdns

# AD-integrated zone replication checks.
repadmin /replsummary
repadmin /showrepl
dcdiag /test:dns /v

# Check DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 50 |
  Select-Object TimeCreated, Id, ProviderName, Message
```

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Rollback
| Step | Action | PowerShell | Expected Result |
|---:|---|---|---|
| 1 | Capture current PTR before rollback | `Get-DnsServerResourceRecord -ZoneName "<reverse-zone>" -Name "<ptr-record-name>" -RRType PTR` | PTR state is documented |
| 2 | Remove PTR record by object | `$Ptr = Get-DnsServerResourceRecord -ZoneName "<reverse-zone>" -Name "<ptr-record-name>" -RRType PTR; Remove-DnsServerResourceRecord -ZoneName "<reverse-zone>" -InputObject $Ptr -Force` | PTR record is removed |
| 3 | Confirm PTR removal | `Get-DnsServerResourceRecord -ZoneName "<reverse-zone>" -Name "<ptr-record-name>" -ErrorAction SilentlyContinue` | No PTR record returns |
| 4 | Remove lab-only reverse zone | `Remove-DnsServerZone -Name "<reverse-zone>" -Force` | Reverse zone is removed |
| 5 | Flush client resolver cache | `Clear-DnsClientCache; ipconfig /flushdns` | Cached reverse lookup answers are cleared |
| 6 | Verify reverse lookup no longer resolves | `Resolve-DnsName "<ipv4-address>" -Server "<dns-server>"` | Expected failure or previous production answer appears |
| 7 | Confirm AD replication after AD-integrated rollback | `repadmin /syncall /AdeP` | Zone or record removal replicates |
| 8 | Document rollback | `Record removed PTRs, removed zone if applicable, and remaining dependencies` | Rollback notes are complete |

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Reverse lookup fails | Reverse zone does not exist | `Get-DnsServerZone -Name "<reverse-zone>"` | Create reverse zone with correct `-NetworkId` |
| PTR creation fails | Wrong reverse zone name | `Get-DnsServerZone` | Use actual reverse zone generated from `-NetworkId` |
| PTR created but IP does not resolve | Wrong PTR record name | `Get-DnsServerResourceRecord -ZoneName "<reverse-zone>" -RRType PTR` | Use correct final octet for `/24` reverse zone |
| PTR points to wrong host | Incorrect `PtrDomainName` | `Resolve-DnsName "<ip>" -Server "<dns-server>"` | Remove and recreate PTR with correct FQDN |
| Forward and reverse do not match | A record and PTR record maintained separately | Compare `Resolve-DnsName "<fqdn>"` and `Resolve-DnsName "<ip>"` | Correct A or PTR record |
| PTR duplicated | Multiple PTRs exist for same IP node | `Get-DnsServerResourceRecord -Name "<ptr-record-name>" -RRType PTR` | Remove stale duplicate |
| PTR disappears later | Scavenging removed aged record | Check `Timestamp` | Recreate static PTR or adjust aging and scavenging |
| PTR not created by DHCP | DHCP DNS dynamic update settings not configured | Check DHCP DNS tab and lease behavior | Configure DHCP to update PTR records |
| Access denied | Insufficient DNS permissions | Run elevated and check DNSAdmins membership | Use delegated DNS rights |
| Reverse zone not on another DC | AD replication delay or wrong replication scope | `repadmin /replsummary`; `Get-DnsServerZone -ComputerName <dc>` | Fix replication or scope |
| `Add-DnsServerPrimaryZone -NetworkId` creates unexpected zone | Prefix not aligned with IPv4 class boundary | Review generated zone from command output | Use expected `/24`, `/16`, or `/8` network ID |
| IPv6 reverse zone name looks strange | IPv6 uses nibble-based `ip6.arpa` format | `Get-DnsServerZone` | Let `-NetworkId` generate the zone and document it |
| Client still sees old PTR | DNS client or resolver cache | `Clear-DnsClientCache`; test with `-Server` | Flush cache and query authoritative DNS server |
| Reverse lookup works on server but not client | Client points to wrong DNS server | `ipconfig /all` | Fix client DNS server assignment |
| Reverse zone exists but no records | Zone only creates container, not host PTRs | `Get-DnsServerResourceRecord -RRType PTR` | Add PTR records manually or through DHCP dynamic updates |

# 03_Create_Reverse_Lookup_Zone_And_PTR_Records_Related_Labs
| Lab                                                            | Relationship                                                                               |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 01_Install_DNS_Server_Role_And_Management_Tools                | Required DNS Server and RSAT tooling baseline                                              |
| 02_Create_Forward_Lookup_Zone                                  | Provides forward zone used to compare A records with PTR records                           |
| 04_Create_And_Manage_DNS_Resource_Records                      | Expands record management to A, AAAA, CNAME, MX, SRV, TXT, NS, and PTR                     |
| 05_Configure_DNS_Forwarders_And_Root_Hints                     | Separates reverse authoritative lookup from recursive lookup behavior                      |
| 07_Configure_AD_Integrated_DNS_Zones                           | Explains reverse zone replication when stored in AD                                        |
| 08_Configure_DNS_Dynamic_Updates                               | Explains DHCP and client-driven PTR registration behavior                                  |
| 09_Troubleshoot_DNS_Client_Resolution                          | Validates reverse lookup from client perspective                                           |
| 18_Configure_DNS_Query_Logging_And_Diagnostics                 | Confirms whether clients are querying PTR names                                            |
| 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication | Extends reverse-zone troubleshooting into signed zones, zone transfers, and AD replication |