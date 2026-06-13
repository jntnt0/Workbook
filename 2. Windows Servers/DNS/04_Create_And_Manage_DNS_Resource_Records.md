04_Create_And_Manage_DNS_Resource_Records.md
# 04_Create_And_Manage_DNS_Resource_Records

# 04_Create_And_Manage_DNS_Resource_Records_Index
04_Create_And_Manage_DNS_Resource_Records.md
04_Create_And_Manage_DNS_Resource_Records
04_Create_And_Manage_DNS_Resource_Records_Source_Basis
04_Create_And_Manage_DNS_Resource_Records_Mental_Model
04_Create_And_Manage_DNS_Resource_Records_Planning_Table
04_Create_And_Manage_DNS_Resource_Records_Configuration_Checklist
04_Create_And_Manage_DNS_Resource_Records_Common_Record_Creation_Skeleton
04_Create_And_Manage_DNS_Resource_Records_Modify_Record_Skeleton
04_Create_And_Manage_DNS_Resource_Records_Remove_Record_Skeleton
04_Create_And_Manage_DNS_Resource_Records_Bulk_Inventory_And_Audit_Skeleton
04_Create_And_Manage_DNS_Resource_Records_Verification_Commands
04_Create_And_Manage_DNS_Resource_Records_Rollback
04_Create_And_Manage_DNS_Resource_Records_Failure_Checks
04_Create_And_Manage_DNS_Resource_Records_Related_Labs

# 04_Create_And_Manage_DNS_Resource_Records_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DnsServerResourceRecord | Creating DNS resource records in a DNS server zone |
| Microsoft Learn | Get-DnsServerResourceRecord | Reading and auditing existing DNS records |
| Microsoft Learn | Set-DnsServerResourceRecord | Modifying an existing DNS resource record using old and new record objects |
| Microsoft Learn | Remove-DnsServerResourceRecord | Removing stale, duplicate, or incorrect DNS records |
| Microsoft Learn | Resolve-DnsName | Client-side DNS resolution testing |
| Windows Server operational practice | Forward zones, reverse zones, TTL, record ownership, scavenging, and AD-integrated DNS | Day-to-day DNS record administration |

# 04_Create_And_Manage_DNS_Resource_Records_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Resource record | A DNS object that maps a name to data such as an IP address, alias, mail server, service endpoint, or text value |
| Forward lookup record | Record used to resolve names to data, usually name to IP |
| Reverse lookup record | PTR record used to resolve IP address back to a name |
| A record | IPv4 host record |
| AAAA record | IPv6 host record |
| CNAME record | Alias pointing one DNS name to another canonical DNS name |
| MX record | Mail exchanger record for mail routing |
| NS record | Name server record for a zone or delegated namespace |
| PTR record | Reverse lookup pointer record |
| SRV record | Service locator record used heavily by Active Directory and application discovery |
| TXT record | Text record used for ownership validation, SPF, DKIM, DMARC, and application metadata |
| TTL | Time that resolvers may cache the answer |
| Record aging | Timestamp behavior used with scavenging |
| Static record | Manually created record that usually should not be scavenged unless aging is enabled |
| Dynamic record | Record created by a DNS client, DHCP server, or domain controller registration process |
| Zone apex | The root of the zone, often represented as the same-as-parent record |
| Duplicate record | Multiple records with the same name and type, sometimes valid for load sharing and sometimes a mistake |
| Stale record | Old record data that no longer reflects the actual host or service |
| First rule | Know the zone, record type, record name, and record data before changing anything |
| Blunt rule | Never delete a DNS record just because it looks old; verify ownership, clients, DHCP behavior, and application dependency first |

# 04_Create_And_Manage_DNS_Resource_Records_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server | `DC1` | `<dns-server>` |
| Forward zone | `corp.local` | `<forward-zone>` |
| Reverse zone | `10.10.10.in-addr.arpa` | `<reverse-zone>` |
| Hostname | `app01` | `<host-name>` |
| FQDN | `app01.corp.local` | `<fqdn>` |
| IPv4 address | `10.10.10.50` | `<ipv4-address>` |
| IPv6 address | `fd00:10:10:10::50` | `<ipv6-address>` |
| Alias name | `portal` | `<alias-name>` |
| Alias target | `app01.corp.local` | `<alias-target-fqdn>` |
| Mail host | `mail.corp.local` | `<mail-host-fqdn>` |
| MX preference | `10` | `<mx-preference>` |
| SRV service | `_ldap` | `<service-name>` |
| SRV protocol | `_tcp` | `<protocol>` |
| SRV target | `dc1.corp.local` | `<srv-target-fqdn>` |
| SRV port | `389` | `<srv-port>` |
| TXT name | `_dmarc` | `<txt-name>` |
| TXT value | `v=DMARC1; p=none` | `<txt-value>` |
| TTL | `01:00:00` | `<ttl>` |
| Aging enabled | No for static infra records | `<yes-no>` |
| Rollback action | Remove newly created record | `<rollback-plan>` |

# 04_Create_And_Manage_DNS_Resource_Records_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm DNS Server module | DNS Server | `Get-Module -ListAvailable DnsServer` | DnsServer module is available |
| 2 | Confirm target zone exists | DNS Server | `Get-DnsServerZone -Name "<forward-zone>" -ComputerName "<dns-server>"` | Forward zone exists |
| 3 | Confirm reverse zone exists if PTR is required | DNS Server | `Get-DnsServerZone -Name "<reverse-zone>" -ComputerName "<dns-server>"` | Reverse zone exists |
| 4 | Capture existing records for the name | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<forward-zone>" -Name "<record-name>" -ComputerName "<dns-server>"` | Existing records are known |
| 5 | Create A record | DNS Server | `Add-DnsServerResourceRecord -A -ZoneName "<forward-zone>" -Name "<host-name>" -IPv4Address "<ipv4-address>" -TimeToLive 01:00:00 -ComputerName "<dns-server>" -PassThru` | IPv4 host record exists |
| 6 | Create AAAA record if IPv6 is required | DNS Server | `Add-DnsServerResourceRecord -AAAA -ZoneName "<forward-zone>" -Name "<host-name>" -IPv6Address "<ipv6-address>" -TimeToLive 01:00:00 -ComputerName "<dns-server>" -PassThru` | IPv6 host record exists |
| 7 | Create PTR record | DNS Server | `Add-DnsServerResourceRecord -Ptr -ZoneName "<reverse-zone>" -Name "<last-octet>" -PtrDomainName "<fqdn>." -ComputerName "<dns-server>" -PassThru` | Reverse lookup resolves IP to name |
| 8 | Create CNAME alias | DNS Server | `Add-DnsServerResourceRecord -CName -ZoneName "<forward-zone>" -Name "<alias-name>" -HostNameAlias "<alias-target-fqdn>." -ComputerName "<dns-server>" -PassThru` | Alias resolves to canonical target |
| 9 | Create MX record | DNS Server | `Add-DnsServerResourceRecord -MX -ZoneName "<forward-zone>" -Name "." -MailExchange "<mail-host-fqdn>." -Preference <mx-preference> -ComputerName "<dns-server>" -PassThru` | Zone has mail exchanger record |
| 10 | Create TXT record | DNS Server | `Add-DnsServerResourceRecord -Txt -ZoneName "<forward-zone>" -Name "<txt-name>" -DescriptiveText "<txt-value>" -ComputerName "<dns-server>" -PassThru` | TXT record exists |
| 11 | Create SRV record | DNS Server | `Add-DnsServerResourceRecord -Srv -ZoneName "<forward-zone>" -Name "_service._protocol" -DomainName "<srv-target-fqdn>." -Priority 0 -Weight 100 -Port <srv-port> -ComputerName "<dns-server>" -PassThru` | SRV service locator exists |
| 12 | Create NS record if delegation or name server listing is required | DNS Server | `Add-DnsServerResourceRecord -NS -ZoneName "<forward-zone>" -Name "<node-or-subdomain>" -NameServer "<name-server-fqdn>." -ComputerName "<dns-server>" -PassThru` | NS record exists |
| 13 | Validate forward lookup | Client / DNS Server | `Resolve-DnsName "<fqdn>" -Server "<dns-server>"` | Expected IP address returns |
| 14 | Validate reverse lookup | Client / DNS Server | `Resolve-DnsName "<ipv4-address>" -Server "<dns-server>"` | Expected PTR name returns |
| 15 | Validate alias lookup | Client / DNS Server | `Resolve-DnsName "<alias-name>.<forward-zone>" -Server "<dns-server>"` | CNAME and target answer return |
| 16 | Validate MX lookup | Client / DNS Server | `Resolve-DnsName "<forward-zone>" -Type MX -Server "<dns-server>"` | MX answer returns |
| 17 | Validate TXT lookup | Client / DNS Server | `Resolve-DnsName "<txt-name>.<forward-zone>" -Type TXT -Server "<dns-server>"` | TXT answer returns |
| 18 | Validate SRV lookup | Client / DNS Server | `Resolve-DnsName "_service._protocol.<forward-zone>" -Type SRV -Server "<dns-server>"` | SRV target and port return |
| 19 | Export post-change record inventory | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<forward-zone>" -ComputerName "<dns-server>" \| Export-Csv C:\DNS-Records-After.csv -NoTypeInformation` | Record state is documented |
| 20 | Document owner and purpose | Operator | `Record owner, ticket, TTL, record type, dependency, and rollback plan` | Record is supportable later |

# 04_Create_And_Manage_DNS_Resource_Records_Common_Record_Creation_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server or management workstation with RSAT DNS tools.

$DnsServer = "DC1"
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"

$HostName = "app01"
$Fqdn = "app01.corp.local"
$IPv4Address = "10.10.10.50"
$IPv6Address = "fd00:10:10:10::50"
$LastOctet = "50"

$AliasName = "portal"
$AliasTarget = "app01.corp.local."

$MailHost = "mail.corp.local."
$MxPreference = 10

$TxtName = "_dmarc"
$TxtValue = "v=DMARC1; p=none"

$SrvName = "_ldap._tcp"
$SrvTarget = "dc1.corp.local."
$SrvPriority = 0
$SrvWeight = 100
$SrvPort = 389

$TTL = [TimeSpan]::FromHours(1)

# Confirm DNS module and zones.
Get-Module -ListAvailable DnsServer
Get-DnsServerZone -ComputerName $DnsServer -Name $ForwardZone
Get-DnsServerZone -ComputerName $DnsServer -Name $ReverseZone

# Check for existing records before creating anything.
Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ForwardZone -Name $HostName -ErrorAction SilentlyContinue
Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ForwardZone -Name $AliasName -ErrorAction SilentlyContinue
Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ReverseZone -Name $LastOctet -ErrorAction SilentlyContinue

# Create A record.
Add-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -A `
  -Name $HostName `
  -IPv4Address $IPv4Address `
  -TimeToLive $TTL `
  -PassThru

# Create AAAA record.
Add-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -AAAA `
  -Name $HostName `
  -IPv6Address $IPv6Address `
  -TimeToLive $TTL `
  -PassThru

# Create PTR record.
Add-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -Ptr `
  -Name $LastOctet `
  -PtrDomainName "$Fqdn." `
  -TimeToLive $TTL `
  -PassThru

# Create CNAME record.
Add-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -CName `
  -Name $AliasName `
  -HostNameAlias $AliasTarget `
  -TimeToLive $TTL `
  -PassThru

# Create MX record at zone apex.
Add-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -MX `
  -Name "." `
  -MailExchange $MailHost `
  -Preference $MxPreference `
  -TimeToLive $TTL `
  -PassThru

# Create TXT record.
Add-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -Txt `
  -Name $TxtName `
  -DescriptiveText $TxtValue `
  -TimeToLive $TTL `
  -PassThru

# Create SRV record.
Add-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -Srv `
  -Name $SrvName `
  -DomainName $SrvTarget `
  -Priority $SrvPriority `
  -Weight $SrvWeight `
  -Port $SrvPort `
  -TimeToLive $TTL `
  -PassThru

# Verify created records.
Resolve-DnsName "$HostName.$ForwardZone" -Server $DnsServer
Resolve-DnsName $IPv4Address -Server $DnsServer
Resolve-DnsName "$AliasName.$ForwardZone" -Server $DnsServer
Resolve-DnsName $ForwardZone -Type MX -Server $DnsServer
Resolve-DnsName "$TxtName.$ForwardZone" -Type TXT -Server $DnsServer
Resolve-DnsName "$SrvName.$ForwardZone" -Type SRV -Server $DnsServer
```

# 04_Create_And_Manage_DNS_Resource_Records_Modify_Record_Skeleton
```powershell
# Run in elevated PowerShell.
# Set-DnsServerResourceRecord uses old and new record objects.
# Clone the old record, modify the clone, then submit both objects.

$DnsServer = "DC1"
$ZoneName = "corp.local"
$RecordName = "app01"
$OldIPv4Address = "10.10.10.50"
$NewIPv4Address = "10.10.10.55"
$NewTTL = [TimeSpan]::FromMinutes(30)

# Get the existing A record.
$OldRecord = Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName `
  -Name $RecordName `
  -RRType A |
  Where-Object {
    $_.RecordData.IPv4Address.IPAddressToString -eq $OldIPv4Address
  }

# Clone and modify record data.
$NewRecord = $OldRecord.Clone()
$NewRecord.RecordData.IPv4Address = [System.Net.IPAddress]::Parse($NewIPv4Address)
$NewRecord.TimeToLive = $NewTTL

# Apply change.
Set-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName `
  -OldInputObject $OldRecord `
  -NewInputObject $NewRecord `
  -PassThru

# Verify.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName `
  -Name $RecordName `
  -RRType A

Resolve-DnsName "$RecordName.$ZoneName" -Server $DnsServer
```

# 04_Create_And_Manage_DNS_Resource_Records_Remove_Record_Skeleton
```powershell
# Run in elevated PowerShell.
# Prefer object-based removal when possible because it avoids removing the wrong duplicate record.

$DnsServer = "DC1"
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"

$RecordName = "app01"
$IPv4Address = "10.10.10.50"
$LastOctet = "50"
$AliasName = "portal"

# Remove a specific A record by object.
$ARecord = Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -Name $RecordName `
  -RRType A |
  Where-Object {
    $_.RecordData.IPv4Address.IPAddressToString -eq $IPv4Address
  }

if ($ARecord) {
  Remove-DnsServerResourceRecord `
    -ComputerName $DnsServer `
    -ZoneName $ForwardZone `
    -InputObject $ARecord `
    -Force
}

# Remove matching PTR record by object.
$PtrRecord = Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ReverseZone `
  -Name $LastOctet `
  -RRType PTR |
  Where-Object {
    $_.RecordData.PtrDomainName -eq "$RecordName.$ForwardZone."
  }

if ($PtrRecord) {
  Remove-DnsServerResourceRecord `
    -ComputerName $DnsServer `
    -ZoneName $ReverseZone `
    -InputObject $PtrRecord `
    -Force
}

# Remove a CNAME record by object.
$CNameRecord = Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ForwardZone `
  -Name $AliasName `
  -RRType CNAME

if ($CNameRecord) {
  Remove-DnsServerResourceRecord `
    -ComputerName $DnsServer `
    -ZoneName $ForwardZone `
    -InputObject $CNameRecord `
    -Force
}

# Verify removal.
Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ForwardZone -Name $RecordName -ErrorAction SilentlyContinue
Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ReverseZone -Name $LastOctet -ErrorAction SilentlyContinue
Get-DnsServerResourceRecord -ComputerName $DnsServer -ZoneName $ForwardZone -Name $AliasName -ErrorAction SilentlyContinue
```

# 04_Create_And_Manage_DNS_Resource_Records_Bulk_Inventory_And_Audit_Skeleton
```powershell
# Run on a DNS server or management host.
# Purpose: inventory records before and after record changes.

$DnsServer = "DC1"
$ZoneName = "corp.local"
$ExportPath = "C:\DNS-Audit"

New-Item -ItemType Directory -Force -Path $ExportPath

# Full zone record inventory.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName |
  Select-Object `
    HostName,
    RecordType,
    Timestamp,
    TimeToLive,
    DistinguishedName,
    RecordData |
  Export-Csv "$ExportPath\$ZoneName-all-records.csv" -NoTypeInformation

# A records only.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName `
  -RRType A |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData |
  Export-Csv "$ExportPath\$ZoneName-a-records.csv" -NoTypeInformation

# CNAME records only.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName `
  -RRType CNAME |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData |
  Export-Csv "$ExportPath\$ZoneName-cname-records.csv" -NoTypeInformation

# MX records only.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName `
  -RRType MX |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData |
  Export-Csv "$ExportPath\$ZoneName-mx-records.csv" -NoTypeInformation

# TXT records only.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName `
  -RRType TXT |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData |
  Export-Csv "$ExportPath\$ZoneName-txt-records.csv" -NoTypeInformation

# Records with aging timestamps.
Get-DnsServerResourceRecord `
  -ComputerName $DnsServer `
  -ZoneName $ZoneName |
  Where-Object {
    $_.Timestamp -ne $null
  } |
  Select-Object HostName, RecordType, Timestamp, TimeToLive, RecordData |
  Export-Csv "$ExportPath\$ZoneName-aged-records.csv" -NoTypeInformation
```

# 04_Create_And_Manage_DNS_Resource_Records_Verification_Commands
```powershell
# Zone checks.
Get-DnsServerZone
Get-DnsServerZone -Name "corp.local"
Get-DnsServerZone -Name "10.10.10.in-addr.arpa"

# Record inventory.
Get-DnsServerResourceRecord -ZoneName "corp.local"
Get-DnsServerResourceRecord -ZoneName "corp.local" -Name "app01"
Get-DnsServerResourceRecord -ZoneName "corp.local" -Name "app01" -RRType A
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType CNAME
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType MX
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType TXT
Get-DnsServerResourceRecord -ZoneName "corp.local" -RRType SRV
Get-DnsServerResourceRecord -ZoneName "10.10.10.in-addr.arpa" -RRType PTR

# Client resolution tests.
Resolve-DnsName "app01.corp.local" -Server "DC1"
Resolve-DnsName "portal.corp.local" -Server "DC1"
Resolve-DnsName "corp.local" -Type MX -Server "DC1"
Resolve-DnsName "_dmarc.corp.local" -Type TXT -Server "DC1"
Resolve-DnsName "_ldap._tcp.corp.local" -Type SRV -Server "DC1"
Resolve-DnsName "10.10.10.50" -Server "DC1"

# Classic nslookup fallback.
nslookup app01.corp.local DC1
nslookup 10.10.10.50 DC1
nslookup -type=mx corp.local DC1
nslookup -type=txt _dmarc.corp.local DC1
nslookup -type=srv _ldap._tcp.corp.local DC1

# Cache reset on test client.
Clear-DnsClientCache
ipconfig /flushdns

# DNS Server service state.
Get-Service DNS
Get-WinEvent -LogName "DNS Server" -MaxEvents 50
```

# 04_Create_And_Manage_DNS_Resource_Records_Rollback
| Step | Action | PowerShell | Expected Result |
|---:|---|---|---|
| 1 | Capture current record before rollback | `Get-DnsServerResourceRecord -ZoneName "<zone>" -Name "<name>"` | Current state is documented |
| 2 | Remove newly created A record | `Remove-DnsServerResourceRecord -ZoneName "<zone>" -InputObject $ARecord -Force` | A record is removed |
| 3 | Remove newly created AAAA record | `Remove-DnsServerResourceRecord -ZoneName "<zone>" -InputObject $AAAARecord -Force` | AAAA record is removed |
| 4 | Remove newly created PTR record | `Remove-DnsServerResourceRecord -ZoneName "<reverse-zone>" -InputObject $PtrRecord -Force` | PTR record is removed |
| 5 | Remove newly created CNAME record | `Remove-DnsServerResourceRecord -ZoneName "<zone>" -InputObject $CNameRecord -Force` | Alias is removed |
| 6 | Restore old A record IP if modified | `Set-DnsServerResourceRecord -ZoneName "<zone>" -OldInputObject $CurrentRecord -NewInputObject $RestoredRecord` | Original IP address is restored |
| 7 | Restore old TTL if changed | `$RestoredRecord.TimeToLive = [TimeSpan]::FromHours(<hours>)` | Original TTL is restored |
| 8 | Flush client resolver cache | `Clear-DnsClientCache; ipconfig /flushdns` | Client stops using cached stale answer |
| 9 | Verify rollback | `Resolve-DnsName "<fqdn>" -Server "<dns-server>"` | Expected old answer returns |
| 10 | Document rollback | `Record removed records, restored values, and remaining dependencies` | Support notes are complete |

# 04_Create_And_Manage_DNS_Resource_Records_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Record creation fails | Wrong zone name | `Get-DnsServerZone -Name "<zone>"` | Use correct forward or reverse zone |
| Record resolves on server but not client | Client cache or wrong DNS server | `ipconfig /all`; `Resolve-DnsName <fqdn> -Server <dns-server>` | Flush cache and point client at correct DNS |
| PTR lookup fails | Missing reverse zone or wrong PTR node name | `Get-DnsServerZone`; `Get-DnsServerResourceRecord -RRType PTR` | Create reverse zone or correct PTR record |
| CNAME does not resolve | Alias target missing or target lacks trailing FQDN clarity | `Resolve-DnsName <target>` | Create target record or correct alias target |
| MX lookup returns no records | MX created under wrong name | `Get-DnsServerResourceRecord -RRType MX` | Use zone apex name or correct mail domain node |
| TXT value looks wrong | Quotes, semicolons, or policy string pasted incorrectly | `Resolve-DnsName <name> -Type TXT` | Recreate TXT record with exact provider value |
| SRV lookup fails | Wrong service label or protocol label | `Resolve-DnsName _service._protocol.<zone> -Type SRV` | Use exact `_service._tcp` or `_service._udp` format |
| Duplicate A records appear | Existing record was not checked first | `Get-DnsServerResourceRecord -Name <name> -RRType A` | Remove wrong duplicate or document intentional load sharing |
| Wrong IP still returned | Resolver cache or multiple records | `Resolve-DnsName <fqdn> -Server <dns-server>` | Flush cache and remove stale duplicate |
| Record disappears later | DHCP dynamic update or scavenging behavior | Check timestamp and DHCP ownership | Correct DHCP DNS update settings or create static record |
| Static record gets scavenged | Aging was enabled on the record | Check `Timestamp` | Recreate as static or adjust aging/scavenging |
| Access denied | Not running elevated or insufficient DNS permissions | Run elevated and check group membership | Use DNSAdmins or delegated rights |
| AD-integrated DNS record not on all DCs | AD replication delay or failure | `repadmin /replsummary`; compare records across DCs | Fix AD replication first |
| Record added to wrong DNS server | Managing non-authoritative server | `Get-DnsServerZone -ComputerName <server>` | Add record to authoritative zone owner |
| Record exists but application fails | Application expects different name, port, or target | Test exact app lookup | Correct record data based on application requirement |

# 04_Create_And_Manage_DNS_Resource_Records_Related_Labs
| Lab                                                            | Relationship                                                                      |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| 01_Install_DNS_Server_Role_And_Management_Tools                | Required DNS Server and RSAT tooling baseline                                     |
| 02_Create_Forward_Lookup_Zone                                  | Provides the forward zone where A, AAAA, CNAME, MX, TXT, NS, and SRV records live |
| 03_Create_Reverse_Lookup_Zone_And_PTR_Records                  | Provides the reverse zone used by PTR records                                     |
| 05_Configure_DNS_Forwarders_And_Root_Hints                     | Separates authoritative record management from recursion behavior                 |
| 06_Configure_DNS_Conditional_Forwarders                        | Helps identify when the record belongs in another DNS namespace                   |
| 07_Configure_AD_Integrated_DNS_Zones                           | Explains AD replication behavior for records stored in AD-integrated zones        |
| 08_Configure_DNS_Dynamic_Updates                               | Explains why DHCP or clients may create, update, or overwrite records             |
| 09_Troubleshoot_DNS_Client_Resolution                          | Validates records from the client perspective                                     |
| 18_Configure_DNS_Query_Logging_And_Diagnostics                 | Confirms whether clients are querying the expected names                          |
| 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication | Extends record troubleshooting into signed zones, transfers, and AD replication   |