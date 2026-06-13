02_Create_Forward_Lookup_Zone.md
# 02_Create_Forward_Lookup_Zone

# 02_Create_Forward_Lookup_Zone_Index
02_Create_Forward_Lookup_Zone.md
02_Create_Forward_Lookup_Zone
02_Create_Forward_Lookup_Zone_Source_Basis
02_Create_Forward_Lookup_Zone_Mental_Model
02_Create_Forward_Lookup_Zone_Planning_Table
02_Create_Forward_Lookup_Zone_Configuration_Checklist
02_Create_Forward_Lookup_Zone_AD_Integrated_Forward_Zone_Skeleton
02_Create_Forward_Lookup_Zone_File_Backed_Forward_Zone_Skeleton
02_Create_Forward_Lookup_Zone_Zone_Settings_Update_Skeleton
02_Create_Forward_Lookup_Zone_Verification_Commands
02_Create_Forward_Lookup_Zone_Rollback
02_Create_Forward_Lookup_Zone_Failure_Checks
02_Create_Forward_Lookup_Zone_Related_Labs

# 02_Create_Forward_Lookup_Zone_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DnsServerPrimaryZone | Creating AD-integrated and file-backed forward lookup zones |
| Microsoft Learn | Get-DnsServerZone | Verifying DNS zone existence, zone type, and zone properties |
| Microsoft Learn | Set-DnsServerPrimaryZone | Updating zone settings such as dynamic update, replication scope, notify, and zone transfer behavior |
| Microsoft Learn | Remove-DnsServerZone | Removing lab-only or incorrectly created zones |
| Microsoft Learn | Add-DnsServerResourceRecord | Creating initial records after the zone exists |
| Microsoft Learn | Resolve-DnsName | Testing name resolution against the authoritative DNS server |
| Windows Server operational practice | AD-integrated DNS, secure dynamic updates, SOA, NS records, and DNS replication | Building a supportable forward lookup zone baseline |

# 02_Create_Forward_Lookup_Zone_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Forward lookup zone | DNS namespace used to resolve names to IP addresses or other DNS data |
| Zone name | DNS namespace hosted by the server, such as `corp.local` |
| Authoritative DNS server | DNS server that owns the zone data and answers for that namespace |
| Primary zone | Writable copy of a DNS zone |
| AD-integrated zone | Primary zone stored in Active Directory instead of a local zone file |
| File-backed zone | Primary zone stored as a `.dns` file on one DNS server |
| Replication scope | AD partition that controls where an AD-integrated zone replicates |
| Domain replication scope | Replicates zone to DNS servers in the domain |
| Forest replication scope | Replicates zone to DNS servers in the forest |
| Legacy replication scope | Older Windows 2000 style domain partition behavior |
| Custom replication scope | Stores the zone in a custom application directory partition |
| Dynamic update | Allows DNS records to be registered or changed dynamically |
| Secure dynamic update | AD-integrated update mode that requires authenticated update ownership |
| Nonsecure and secure update | Permits unauthenticated and authenticated DNS updates, usually unsafe outside labs |
| No dynamic update | Manual record management only |
| SOA record | Start of Authority record defining zone authority and timing values |
| NS record | Name server record identifying authoritative DNS servers for the zone |
| Zone apex | Root of the zone, represented by the zone name itself |
| First rule | Create the zone before creating records inside it |
| Blunt rule | For domain DNS, AD-integrated plus secure dynamic updates is usually the correct baseline |

# 02_Create_Forward_Lookup_Zone_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server | `DC1` | `<dns-server>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Forward zone name | `corp.local` | `<forward-zone>` |
| Zone type | AD-integrated primary | `<AD-integrated / file-backed>` |
| Replication scope | Domain | `<Domain / Forest / Legacy / Custom>` |
| Dynamic update mode | Secure | `<None / Secure / NonsecureAndSecure>` |
| File-backed zone file | `corp.local.dns` | `<zone-file-name>` |
| Responsible person | `hostmaster.corp.local` | `<responsible-person>` |
| Initial host record | `dc1` | `<initial-record-name>` |
| Initial host IP | `10.10.10.10` | `<initial-record-ip>` |
| TTL baseline | `01:00:00` | `<ttl>` |
| DNS client test server | `DC1` | `<dns-server-to-query>` |
| Zone transfer required | No for AD-integrated baseline | `<yes-no>` |
| Rollback action | Remove lab-only zone | `<rollback-plan>` |

# 02_Create_Forward_Lookup_Zone_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm DNS Server module exists | DNS Server | `Get-Module -ListAvailable DnsServer` | DnsServer module is available |
| 2 | Confirm DNS service is running | DNS Server | `Get-Service DNS` | DNS service is running |
| 3 | Inventory existing DNS zones | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>"` | Existing zones are visible |
| 4 | Check whether target zone already exists | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>" -Name "<forward-zone>" -ErrorAction SilentlyContinue` | Duplicate zone risk is known |
| 5 | Create AD-integrated forward lookup zone | DNS Server | `Add-DnsServerPrimaryZone -ComputerName "<dns-server>" -Name "<forward-zone>" -ReplicationScope "Domain" -DynamicUpdate "Secure" -PassThru` | AD-integrated forward zone exists |
| 6 | Alternative: create file-backed forward lookup zone | DNS Server | `Add-DnsServerPrimaryZone -ComputerName "<dns-server>" -Name "<forward-zone>" -ZoneFile "<zone-file-name>" -DynamicUpdate "None" -PassThru` | File-backed forward zone exists |
| 7 | Verify zone properties | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>" -Name "<forward-zone>" \| Format-List *` | Zone type, DS integration, and dynamic update mode are confirmed |
| 8 | Confirm SOA record | DNS Server | `Get-DnsServerResourceRecord -ComputerName "<dns-server>" -ZoneName "<forward-zone>" -RRType SOA` | SOA record exists |
| 9 | Confirm NS records | DNS Server | `Get-DnsServerResourceRecord -ComputerName "<dns-server>" -ZoneName "<forward-zone>" -RRType NS` | Authoritative NS records exist |
| 10 | Add initial A record if needed | DNS Server | `Add-DnsServerResourceRecordA -ComputerName "<dns-server>" -ZoneName "<forward-zone>" -Name "<initial-record-name>" -IPv4Address "<initial-record-ip>" -TimeToLive 01:00:00 -PassThru` | Initial host record exists |
| 11 | Verify initial A record | DNS Server | `Get-DnsServerResourceRecord -ComputerName "<dns-server>" -ZoneName "<forward-zone>" -Name "<initial-record-name>" -RRType A` | Record data is correct |
| 12 | Test zone apex lookup | Client / DNS Server | `Resolve-DnsName "<forward-zone>" -Server "<dns-server>"` | Zone query reaches authoritative server |
| 13 | Test host lookup | Client / DNS Server | `Resolve-DnsName "<initial-record-name>.<forward-zone>" -Server "<dns-server>"` | Host resolves to expected IP |
| 14 | Confirm AD replication if AD-integrated | Domain Controller | `repadmin /replsummary` | AD replication is healthy |
| 15 | Confirm zone on another DNS DC if applicable | DNS Server | `Get-DnsServerZone -ComputerName "<other-dns-server>" -Name "<forward-zone>"` | Zone appears on expected replica |
| 16 | Export zone inventory | DNS Server | `Get-DnsServerResourceRecord -ComputerName "<dns-server>" -ZoneName "<forward-zone>" \| Export-Csv C:\DNS-Forward-Zone-Records.csv -NoTypeInformation` | Zone baseline is documented |
| 17 | Document zone ownership | Operator | `Record zone name, type, replication scope, dynamic update mode, owner, and rollback plan` | Forward zone is supportable later |

# 02_Create_Forward_Lookup_Zone_AD_Integrated_Forward_Zone_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server or management workstation with RSAT DNS tools.
# Purpose: create an AD-integrated forward lookup zone.

$DnsServer = "DC1"
$ForwardZone = "corp.local"
$ReplicationScope = "Domain"
$DynamicUpdate = "Secure"

# Confirm DNS Server module and service.
Get-Module -ListAvailable DnsServer
Get-Service DNS -ComputerName $DnsServer

# Inventory current zones.
Get-DnsServerZone -ComputerName $DnsServer

# Check whether the zone already exists.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone `
  -ErrorAction SilentlyContinue

# Create AD-integrated forward lookup zone.
Add-DnsServerPrimaryZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone `
  -ReplicationScope $ReplicationScope `
  -DynamicUpdate $DynamicUpdate `
  -PassThru

# Verify zone properties.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone |
  Format-List *

# Confirm default SOA and NS records.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -RRType SOA

Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -RRType NS

# Optional AD replication health check.
repadmin /replsummary
```

# 02_Create_Forward_Lookup_Zone_File_Backed_Forward_Zone_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: create a file-backed forward lookup zone.

$DnsServer = "DNS1"
$ForwardZone = "lab.local"
$ZoneFile = "lab.local.dns"
$DynamicUpdate = "None"

# Confirm DNS Server module and service.
Get-Module -ListAvailable DnsServer
Get-Service DNS -ComputerName $DnsServer

# Check current zones.
Get-DnsServerZone -ComputerName $DnsServer

# Check whether zone already exists.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone `
  -ErrorAction SilentlyContinue

# Create file-backed forward lookup zone.
Add-DnsServerPrimaryZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone `
  -ZoneFile $ZoneFile `
  -DynamicUpdate $DynamicUpdate `
  -PassThru

# Verify zone.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone |
  Format-List *

# Confirm SOA and NS records.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone
```

# 02_Create_Forward_Lookup_Zone_Zone_Settings_Update_Skeleton
```powershell
# Run in elevated PowerShell.
# Purpose: correct zone settings after creation.

$DnsServer = "DC1"
$ForwardZone = "corp.local"

# Review current zone state.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone |
  Format-List *

# For AD-integrated domain baseline, set secure dynamic updates.
Set-DnsServerPrimaryZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone `
  -DynamicUpdate Secure `
  -PassThru

# For AD-integrated zone, adjust replication scope if required.
# Use with care. Do not change replication scope blindly.
Set-DnsServerPrimaryZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone `
  -ReplicationScope Domain `
  -PassThru

# Verify after update.
Get-DnsServerZone `
  -ComputerName $DnsServer `
  -Name $ForwardZone |
  Format-List *
```

# 02_Create_Forward_Lookup_Zone_Verification_Commands
```powershell
# DNS Server module and service.
Get-Module -ListAvailable DnsServer
Get-Service DNS

# Zone inventory.
Get-DnsServerZone
Get-DnsServerZone -Name "corp.local"

# Zone property review.
Get-DnsServerZone -Name "corp.local" | Format-List *

# SOA and NS records.
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType SOA
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType NS

# All records in zone.
Get-DnsServerResourceRecord -ZoneName "corp.local"

# Create a basic test A record if needed.
Add-DnsServerResourceRecordA `
  -ZoneName "corp.local" `
  -Name "testhost01" `
  -IPv4Address "10.10.10.50" `
  -TimeToLive 01:00:00 `
  -PassThru

# Verify record object.
Get-DnsServerResourceRecord `
  -ZoneName "corp.local" `
  -Name "testhost01" `
  -RRType A

# Client resolution test.
Resolve-DnsName "testhost01.corp.local" -Server "DC1"

# Classic nslookup check.
nslookup testhost01.corp.local DC1

# Client cache reset.
Clear-DnsClientCache
ipconfig /flushdns

# AD-integrated zone replication checks.
repadmin /replsummary
repadmin /showrepl
dcdiag /test:dns /v

# DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 50 |
  Select-Object TimeCreated, Id, ProviderName, Message
```

# 02_Create_Forward_Lookup_Zone_Rollback
| Step | Action | PowerShell | Expected Result |
|---:|---|---|---|
| 1 | Capture current zone state | `Get-DnsServerZone -Name "<forward-zone>" \| Format-List *` | Zone state is documented |
| 2 | Export records before removal | `Get-DnsServerResourceRecord -ZoneName "<forward-zone>" \| Export-Csv C:\DNS-Zone-Rollback.csv -NoTypeInformation` | Record state is saved |
| 3 | Remove test record only | `$Rec = Get-DnsServerResourceRecord -ZoneName "<forward-zone>" -Name "<record-name>"; Remove-DnsServerResourceRecord -ZoneName "<forward-zone>" -InputObject $Rec -Force` | Test record is removed |
| 4 | Remove lab-only forward zone | `Remove-DnsServerZone -Name "<forward-zone>" -Force` | Zone is deleted |
| 5 | Confirm zone removal | `Get-DnsServerZone -Name "<forward-zone>" -ErrorAction SilentlyContinue` | Zone no longer appears |
| 6 | Flush client resolver cache | `Clear-DnsClientCache; ipconfig /flushdns` | Client cache is cleared |
| 7 | Confirm expected lookup failure | `Resolve-DnsName "<record>.<forward-zone>" -Server "<dns-server>"` | Name no longer resolves if zone was removed |
| 8 | Force AD replication if AD-integrated zone was removed | `repadmin /syncall /AdeP` | Zone deletion replicates |
| 9 | Document rollback | `Record removed zone, removed records, and remaining dependencies` | Rollback notes are complete |

# 02_Create_Forward_Lookup_Zone_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Zone creation fails | DNS Server role or module missing | `Get-Service DNS`; `Get-Module -ListAvailable DnsServer` | Install DNS Server role and RSAT DNS tools |
| Zone already exists | Duplicate namespace on same DNS server | `Get-DnsServerZone -Name "<zone>"` | Use existing zone or choose correct namespace |
| Access denied | Not elevated or insufficient DNS rights | Run PowerShell as admin and check group membership | Use Domain Admin, DNSAdmins, or delegated DNS permissions |
| Secure dynamic update unavailable | Zone is file-backed, not AD-integrated | `Get-DnsServerZone -Name "<zone>" \| Format-List IsDsIntegrated` | Use AD-integrated zone for secure updates |
| Zone created but clients cannot resolve names | Client points at wrong DNS server | `ipconfig /all`; `Resolve-DnsName <name> -Server <dns-server>` | Correct client DNS server assignment |
| Zone appears on one DC only | AD replication delay or wrong replication scope | `repadmin /replsummary`; `Get-DnsServerZone -ComputerName <dc>` | Fix AD replication or set expected scope |
| SOA or NS records look wrong | Zone created on wrong server or stale server metadata | `Get-DnsServerResourceRecord -RRType SOA`; `Get-DnsServerResourceRecord -RRType NS` | Correct zone owner, NS records, or server registration |
| Dynamic records do not appear | Dynamic update disabled or DHCP/client update issue | Check `DynamicUpdate` and client registration | Set secure dynamic update and validate DHCP DNS settings |
| Records disappear later | Scavenging removed aged records | Check record `Timestamp` | Adjust aging/scavenging or create static records |
| Resolution works on DNS server but not client | Client cache or resolver path issue | `Clear-DnsClientCache`; query with `-Server` | Flush cache and test authoritative server directly |
| Wrong zone name used | Namespace typo or missing suffix | `Get-DnsServerZone`; `Resolve-DnsName` | Recreate with correct FQDN or add correct zone |
| File-backed zone file issue | Bad file name or existing stale file | Check `%systemroot%\System32\dns` | Use correct `-ZoneFile` or remove stale lab file |
| AD-integrated zone fails to replicate | Directory Services issue, not DNS record issue | `repadmin /showrepl`; Directory Service logs | Fix AD replication first |
| `NonsecureAndSecure` used accidentally | Unsafe dynamic update mode | `Get-DnsServerZone -Name "<zone>" \| Format-List *` | Set `DynamicUpdate Secure` for AD-integrated domain zones |

# 02_Create_Forward_Lookup_Zone_Related_Labs
| Lab | Relationship |
|---|---|
| 01_Install_DNS_Server_Role_And_Management_Tools | Required DNS Server and RSAT tooling baseline |
| 03_Create_Reverse_Lookup_Zone_And_PTR_Records | Adds reverse lookup namespace after forward zone exists |
| 04_Create_And_Manage_DNS_Resource_Records | Adds A, AAAA, CNAME, MX, TXT, SRV, NS, and PTR records after zone creation |
| 05_Configure_DNS_Forwarders_And_Root_Hints | Separates authoritative zone hosting from recursive lookup behavior |
| 06_Configure_DNS_Conditional_Forwarders | Handles namespaces not hosted locally |
| 07_Configure_AD_Integrated_DNS_Zones | Expands AD-integrated zone replication and secure update behavior |
| 08_Configure_DNS_Dynamic_Updates | Controls automatic client and DHCP registration inside the zone |
| 09_Troubleshoot_DNS_Client_Resolution | Validates zone resolution from client perspective |
| 18_Configure_DNS_Query_Logging_And_Diagnostics | Confirms clients are querying the expected zone |
| 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication | Extends zone troubleshooting into DNSSEC, transfer, and replication behavior |