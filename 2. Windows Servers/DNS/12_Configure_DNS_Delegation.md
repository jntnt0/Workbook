12_Configure_DNS_Delegation.md
# Configure_DNS_Delegation

# Configure_DNS_Delegation_Index
12_Configure_DNS_Delegation.md
Configure_DNS_Delegation
Configure_DNS_Delegation_Source_Basis
Configure_DNS_Delegation_Mental_Model
Configure_DNS_Delegation_Planning_Table
Configure_DNS_Delegation_Configuration_Checklist
Configure_DNS_Delegation_Precheck_Skeleton
Configure_DNS_Delegation_Child_Zone_Skeleton
Configure_DNS_Delegation_Parent_Delegation_Skeleton
Configure_DNS_Delegation_Glue_Record_Skeleton
Configure_DNS_Delegation_Resolution_Path_Validation_Skeleton
Configure_DNS_Delegation_Client_Validation_Skeleton
Configure_DNS_Delegation_Event_Log_Skeleton
Configure_DNS_Delegation_Verification_Commands
Configure_DNS_Delegation_Rollback
Configure_DNS_Delegation_Failure_Checks
Configure_DNS_Delegation_Related_Labs

# Configure_DNS_Delegation_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | Managing Windows Server DNS zones, records, and delegation behavior |
| Microsoft Learn | Add-DnsServerPrimaryZone | Creating authoritative child zones |
| Microsoft Learn | Add-DnsServerResourceRecord | Creating NS and glue records when delegation cmdlets are not used |
| Microsoft Learn | Add-DnsServerResourceRecordA | Creating glue A records for delegated DNS servers |
| Microsoft Learn | Get-DnsServerZone | Validating parent and child zone presence |
| Microsoft Learn | Get-DnsServerResourceRecord | Validating NS, SOA, A, and glue records |
| Microsoft Learn | Resolve-DnsName | Testing delegated namespace resolution |
| Microsoft Learn | Test-NetConnection | Validating DNS server reachability on TCP 53 |
| Microsoft Learn | DNS Server event logs | Reviewing delegation, query, recursion, and zone loading events |
| Windows Server operational practice | Parent zone, child zone, NS records, glue records, authoritative DNS path | Delegating DNS namespace authority without copying the entire zone |

# Configure_DNS_Delegation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS delegation | Parent zone points a child namespace to another authoritative DNS server |
| Parent zone | Zone that owns the higher namespace, such as `corp.local` |
| Child zone | Delegated namespace, such as `dev.corp.local` or `branch.corp.local` |
| Delegated server | DNS server authoritative for the child zone |
| NS record | Record in the parent zone that identifies authoritative DNS servers for the child zone |
| Glue record | A record in the parent zone that resolves the delegated DNS server name when needed |
| Authoritative answer | DNS response from a server that owns the zone being queried |
| Referral | DNS response that points the resolver toward another authoritative server |
| Conditional forwarder | Alternative design that forwards queries for a zone instead of delegating authority |
| Stub zone | Alternative design that tracks SOA, NS, and glue records for another zone |
| Split responsibility | Parent DNS team controls parent zone; child DNS team controls delegated child zone |
| Delegation boundary | Administrative line where one DNS zone hands off control to another zone |
| AD child domain DNS | Common internal use case where `child.corp.local` is delegated from `corp.local` |
| First rule | Delegation only works when parent NS records, glue records, child zone SOA/NS records, and network reachability all line up |

# Configure_DNS_Delegation_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Parent DNS zone | `corp.local` | `<parent-zone-name>` |
| Parent DNS server | `DC1.corp.local` | `<parent-dns-server>` |
| Parent DNS server IP | `10.10.10.10` | `<parent-dns-ip>` |
| Delegated child zone | `dev.corp.local` | `<child-zone-name>` |
| Delegation label | `dev` | `<delegation-label>` |
| Child DNS server 1 | `DNS1.dev.corp.local` | `<child-dns-server-1>` |
| Child DNS server 1 IP | `10.20.10.10` | `<child-dns-server-1-ip>` |
| Child DNS server 2 | `DNS2.dev.corp.local` | `<child-dns-server-2>` |
| Child DNS server 2 IP | `10.20.10.11` | `<child-dns-server-2-ip>` |
| Child zone type | Primary / AD-integrated primary | `<child-zone-type>` |
| Dynamic update mode | Secure only if AD-integrated | `<dynamic-update-mode>` |
| Delegation records in parent | NS plus glue A records | `<delegation-records>` |
| Test record in child zone | `app1.dev.corp.local` | `<test-record-fqdn>` |
| Test record IP | `10.20.10.50` | `<test-record-ip>` |
| Firewall requirement | TCP/UDP 53 from resolvers to child DNS servers | `<firewall-requirement>` |
| Query source | Parent DNS server, client, admin host | `<query-source>` |
| Expected result | Parent resolves child names through delegation | `<expected-result>` |
| Evidence path | `C:\DNS-Delegation` | `<evidence-path>` |
| Rollback stance | Remove delegation NS/glue records if wrong | `<rollback-plan>` |
| Next workbook | `13_Configure_DNS_Conditional_Forwarders.md` | `<next-task>` |

# Configure_DNS_Delegation_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Parent DNS Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DNS role on parent | Parent DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 3 | Confirm DNS service on parent | Parent DNS Server | `Get-Service DNS` | DNS service is Running |
| 4 | Confirm DNS role on child DNS server | Child DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 5 | Confirm DNS service on child DNS server | Child DNS Server | `Get-Service DNS` | DNS service is Running |
| 6 | Import DNS module | Parent / Child DNS Server | `Import-Module DnsServer` | DNS cmdlets are available |
| 7 | Confirm parent zone exists | Parent DNS Server | `Get-DnsServerZone -Name "<parent-zone-name>"` | Parent zone exists |
| 8 | Confirm child zone exists or create it | Child DNS Server | `Get-DnsServerZone -Name "<child-zone-name>"` | Child zone exists |
| 9 | Create child primary zone if missing | Child DNS Server | `Add-DnsServerPrimaryZone -Name "<child-zone-name>" -DynamicUpdate Secure` | Child DNS server becomes authoritative |
| 10 | Confirm child SOA record | Child DNS Server | `Get-DnsServerResourceRecord -ZoneName "<child-zone-name>" -RRType SOA` | SOA exists |
| 11 | Confirm child NS records | Child DNS Server | `Get-DnsServerResourceRecord -ZoneName "<child-zone-name>" -RRType NS` | Child authoritative DNS servers are listed |
| 12 | Add child test A record | Child DNS Server | `Add-DnsServerResourceRecordA -ZoneName "<child-zone-name>" -Name "<record-name>" -IPv4Address "<test-record-ip>"` | Test record exists in child zone |
| 13 | Validate direct child resolution | Admin Host / Parent DNS Server | `Resolve-DnsName "<test-record-fqdn>" -Server "<child-dns-server-ip>"` | Child DNS server resolves child record |
| 14 | Confirm connectivity to child DNS | Parent DNS Server / Client | `Test-NetConnection "<child-dns-server-ip>" -Port 53` | TCP 53 path is reachable |
| 15 | Add delegation NS records in parent | Parent DNS Server | Use parent delegation skeleton | Parent zone contains NS records for child label |
| 16 | Add glue records if needed | Parent DNS Server | Use glue record skeleton | Parent can resolve child DNS server names |
| 17 | Validate parent delegation records | Parent DNS Server | `Get-DnsServerResourceRecord -ZoneName "<parent-zone-name>" -Name "<delegation-label>"` | NS records are visible |
| 18 | Validate glue records | Parent DNS Server | `Get-DnsServerResourceRecord -ZoneName "<parent-zone-name>" -Name "<child-dns-host-label>"` | Glue A records are visible |
| 19 | Validate full resolution through parent DNS | Client / Admin Host | `Resolve-DnsName "<test-record-fqdn>" -Server "<parent-dns-ip>"` | Parent resolves delegated child record |
| 20 | Validate SOA through parent path | Client / Admin Host | `Resolve-DnsName "<child-zone-name>" -Type SOA -Server "<parent-dns-ip>"` | Query reaches child authoritative DNS server |
| 21 | Validate NS through parent path | Client / Admin Host | `Resolve-DnsName "<child-zone-name>" -Type NS -Server "<parent-dns-ip>"` | Child NS records are returned |
| 22 | Review DNS events | Parent / Child DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | Relevant query, zone, and service events are visible |
| 23 | Export evidence | Parent / Child DNS Server | Run validation skeletons | Evidence files are saved |
| 24 | Document final state | Operator | `Record parent zone, child zone, NS records, glue records, child SOA, test query results, and event status` | Delegation build record is complete |

# Configure_DNS_Delegation_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the parent DNS server.
# Purpose: capture parent zone, child DNS reachability, and existing delegation state before changes.

$ParentZone = "corp.local"
$ChildZone = "dev.corp.local"
$DelegationLabel = "dev"
$ParentDnsIp = "10.10.10.10"
$ChildDnsServer1 = "DNS1.dev.corp.local"
$ChildDnsServer1Ip = "10.20.10.10"
$ChildDnsServer2 = "DNS2.dev.corp.local"
$ChildDnsServer2Ip = "10.20.10.11"
$EvidencePath = "C:\DNS-Delegation"

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

# Capture parent zone state.
Get-DnsServerZone -Name $ParentZone |
  Tee-Object "$EvidencePath\parent-zone-before.txt"

Get-DnsServerResourceRecord -ZoneName $ParentZone |
  Tee-Object "$EvidencePath\parent-zone-records-before.txt"

Get-DnsServerResourceRecord -ZoneName $ParentZone |
  Export-Csv "$EvidencePath\parent-zone-records-before.csv" -NoTypeInformation

# Capture any existing delegation records.
Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $DelegationLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\existing-delegation-records-before.txt"

# Test DNS path to child DNS servers.
Test-NetConnection $ChildDnsServer1Ip -Port 53 |
  Tee-Object "$EvidencePath\test-child-dns-server-1-tcp-53.txt"

Test-NetConnection $ChildDnsServer2Ip -Port 53 |
  Tee-Object "$EvidencePath\test-child-dns-server-2-tcp-53.txt"

# Try direct child zone queries before parent delegation.
Resolve-DnsName $ChildZone -Type SOA -Server $ChildDnsServer1Ip -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\direct-child-zone-soa-before.txt"

Resolve-DnsName $ChildZone -Type NS -Server $ChildDnsServer1Ip -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\direct-child-zone-ns-before.txt"
```

# Configure_DNS_Delegation_Child_Zone_Skeleton
```powershell
# Run in elevated PowerShell on the child DNS server.
# Purpose: create or validate the child zone that will receive delegated authority.

$ChildZone = "dev.corp.local"
$ChildDnsServer1 = "DNS1.dev.corp.local"
$ChildDnsServer1Ip = "10.20.10.10"
$ChildDnsServer2 = "DNS2.dev.corp.local"
$ChildDnsServer2Ip = "10.20.10.11"
$TestRecordName = "app1"
$TestRecordIp = "10.20.10.50"
$EvidencePath = "C:\DNS-Delegation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\child-zone-config-transcript.txt"

Import-Module DnsServer

# Confirm DNS role and service.
Get-WindowsFeature DNS |
  Tee-Object "$EvidencePath\child-dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\child-dns-service-state.txt"

# Check whether child zone exists.
$ExistingChildZone = Get-DnsServerZone -Name $ChildZone -ErrorAction SilentlyContinue

if ($ExistingChildZone) {
  Write-Output "Child zone $ChildZone already exists. No new zone created."
  $ExistingChildZone |
    Tee-Object "$EvidencePath\child-zone-existing.txt"
}
else {
  # Create child primary zone.
  # For AD-integrated child zones on a domain controller, add -ReplicationScope Domain if appropriate.
  Add-DnsServerPrimaryZone `
    -Name $ChildZone `
    -DynamicUpdate Secure

  Get-DnsServerZone -Name $ChildZone |
    Tee-Object "$EvidencePath\child-zone-after-create.txt"
}

# Confirm SOA and NS records.
Get-DnsServerResourceRecord -ZoneName $ChildZone -RRType SOA |
  Tee-Object "$EvidencePath\child-zone-soa.txt"

Get-DnsServerResourceRecord -ZoneName $ChildZone -RRType NS |
  Tee-Object "$EvidencePath\child-zone-ns.txt"

# Add test record in child zone if missing.
$ExistingTestRecord = Get-DnsServerResourceRecord `
  -ZoneName $ChildZone `
  -Name $TestRecordName `
  -ErrorAction SilentlyContinue

if ($ExistingTestRecord) {
  Write-Output "Test record $TestRecordName already exists in $ChildZone."
  $ExistingTestRecord |
    Tee-Object "$EvidencePath\child-zone-test-record-existing.txt"
}
else {
  Add-DnsServerResourceRecordA `
    -ZoneName $ChildZone `
    -Name $TestRecordName `
    -IPv4Address $TestRecordIp `
    -TimeToLive 00:15:00

  Get-DnsServerResourceRecord -ZoneName $ChildZone -Name $TestRecordName |
    Tee-Object "$EvidencePath\child-zone-test-record-after-create.txt"
}

# Validate direct child zone resolution.
Resolve-DnsName "$TestRecordName.$ChildZone" -Server $ChildDnsServer1Ip |
  Tee-Object "$EvidencePath\resolve-test-record-direct-child-dns.txt"

Stop-Transcript
```

# Configure_DNS_Delegation_Parent_Delegation_Skeleton
```powershell
# Run in elevated PowerShell on the parent DNS server.
# Purpose: create delegation records in the parent zone.
# Parent zone corp.local delegates dev.corp.local to child DNS servers.

$ParentZone = "corp.local"
$DelegationLabel = "dev"
$ChildZone = "dev.corp.local"
$ChildDnsServer1 = "DNS1.dev.corp.local."
$ChildDnsServer2 = "DNS2.dev.corp.local."
$EvidencePath = "C:\DNS-Delegation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\parent-delegation-config-transcript.txt"

Import-Module DnsServer

# Confirm parent zone.
Get-DnsServerZone -Name $ParentZone |
  Tee-Object "$EvidencePath\parent-zone-before-delegation.txt"

# Capture any existing delegation records.
Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $DelegationLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\delegation-records-before.txt"

# Create NS delegation records using generic DNS resource record objects.
# This method is explicit and works cleanly in operational notes.
Add-DnsServerResourceRecord `
  -ZoneName $ParentZone `
  -Name $DelegationLabel `
  -NS `
  -NameServer $ChildDnsServer1

Add-DnsServerResourceRecord `
  -ZoneName $ParentZone `
  -Name $DelegationLabel `
  -NS `
  -NameServer $ChildDnsServer2

# Confirm delegation NS records.
Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $DelegationLabel |
  Tee-Object "$EvidencePath\delegation-records-after.txt"

Stop-Transcript
```

# Configure_DNS_Delegation_Glue_Record_Skeleton
```powershell
# Run in elevated PowerShell on the parent DNS server.
# Purpose: create glue A records when child DNS server names live inside the delegated child namespace.
# Example: DNS1.dev.corp.local and DNS2.dev.corp.local need A records reachable from parent delegation.

$ParentZone = "corp.local"
$ChildDnsHostLabel1 = "DNS1.dev"
$ChildDnsHostLabel2 = "DNS2.dev"
$ChildDnsServer1Ip = "10.20.10.10"
$ChildDnsServer2Ip = "10.20.10.11"
$EvidencePath = "C:\DNS-Delegation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\glue-record-config-transcript.txt"

Import-Module DnsServer

# Capture existing glue records if present.
Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $ChildDnsHostLabel1 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\glue-record-1-before.txt"

Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $ChildDnsHostLabel2 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\glue-record-2-before.txt"

# Add glue A record for first child DNS server.
if (-not (Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $ChildDnsHostLabel1 -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ParentZone `
    -Name $ChildDnsHostLabel1 `
    -IPv4Address $ChildDnsServer1Ip `
    -TimeToLive 01:00:00
}

# Add glue A record for second child DNS server.
if (-not (Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $ChildDnsHostLabel2 -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ParentZone `
    -Name $ChildDnsHostLabel2 `
    -IPv4Address $ChildDnsServer2Ip `
    -TimeToLive 01:00:00
}

# Confirm glue records after change.
Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $ChildDnsHostLabel1 |
  Tee-Object "$EvidencePath\glue-record-1-after.txt"

Get-DnsServerResourceRecord -ZoneName $ParentZone -Name $ChildDnsHostLabel2 |
  Tee-Object "$EvidencePath\glue-record-2-after.txt"

Stop-Transcript
```

# Configure_DNS_Delegation_Resolution_Path_Validation_Skeleton
```powershell
# Run from an admin host or parent DNS server.
# Purpose: validate child zone resolution directly and through parent DNS.

$ParentZone = "corp.local"
$ChildZone = "dev.corp.local"
$ParentDnsIp = "10.10.10.10"
$ChildDnsServer1Ip = "10.20.10.10"
$ChildDnsServer2Ip = "10.20.10.11"
$TestRecordFqdn = "app1.dev.corp.local"
$EvidencePath = "C:\DNS-Delegation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Test direct child authoritative resolution.
Resolve-DnsName $ChildZone -Type SOA -Server $ChildDnsServer1Ip |
  Tee-Object "$EvidencePath\direct-child-soa-dns1.txt"

Resolve-DnsName $ChildZone -Type NS -Server $ChildDnsServer1Ip |
  Tee-Object "$EvidencePath\direct-child-ns-dns1.txt"

Resolve-DnsName $TestRecordFqdn -Server $ChildDnsServer1Ip |
  Tee-Object "$EvidencePath\direct-child-test-record-dns1.txt"

# Test second child DNS server if available.
Resolve-DnsName $ChildZone -Type SOA -Server $ChildDnsServer2Ip -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\direct-child-soa-dns2.txt"

Resolve-DnsName $ChildZone -Type NS -Server $ChildDnsServer2Ip -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\direct-child-ns-dns2.txt"

Resolve-DnsName $TestRecordFqdn -Server $ChildDnsServer2Ip -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\direct-child-test-record-dns2.txt"

# Test through parent DNS server.
Resolve-DnsName $ChildZone -Type SOA -Server $ParentDnsIp |
  Tee-Object "$EvidencePath\through-parent-child-soa.txt"

Resolve-DnsName $ChildZone -Type NS -Server $ParentDnsIp |
  Tee-Object "$EvidencePath\through-parent-child-ns.txt"

Resolve-DnsName $TestRecordFqdn -Server $ParentDnsIp |
  Tee-Object "$EvidencePath\through-parent-test-record.txt"

# Connectivity tests.
Test-NetConnection $ParentDnsIp -Port 53 |
  Tee-Object "$EvidencePath\test-parent-dns-tcp-53.txt"

Test-NetConnection $ChildDnsServer1Ip -Port 53 |
  Tee-Object "$EvidencePath\test-child-dns1-tcp-53.txt"

Test-NetConnection $ChildDnsServer2Ip -Port 53 |
  Tee-Object "$EvidencePath\test-child-dns2-tcp-53.txt"
```

# Configure_DNS_Delegation_Client_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a DNS client that uses the parent DNS server.
# Purpose: prove normal clients can resolve delegated child zone records.

$ParentDnsIp = "10.10.10.10"
$ChildZone = "dev.corp.local"
$TestRecordFqdn = "app1.dev.corp.local"
$ClientEvidencePath = "C:\DNS-Delegation-Client-Validation"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Capture client identity and resolver settings.
hostname |
  Tee-Object "$ClientEvidencePath\hostname.txt"

Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\dns-client-server-addresses.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all.txt"

# Clear DNS cache before testing.
Clear-DnsClientCache

# Validate child zone SOA through the configured parent resolver.
Resolve-DnsName $ChildZone -Type SOA -Server $ParentDnsIp |
  Tee-Object "$ClientEvidencePath\resolve-child-soa-through-parent.txt"

# Validate child zone NS records.
Resolve-DnsName $ChildZone -Type NS -Server $ParentDnsIp |
  Tee-Object "$ClientEvidencePath\resolve-child-ns-through-parent.txt"

# Validate test host in delegated child zone.
Resolve-DnsName $TestRecordFqdn -Server $ParentDnsIp |
  Tee-Object "$ClientEvidencePath\resolve-test-record-through-parent.txt"

# Test application-layer path if the test record represents a host.
Test-NetConnection $TestRecordFqdn |
  Tee-Object "$ClientEvidencePath\test-testrecord-connectivity.txt"

# DNS client events.
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\dns-client-operational-events.txt"
```

# Configure_DNS_Delegation_Event_Log_Skeleton
```powershell
# Run on parent and child DNS servers.
# Purpose: collect DNS Server events related to delegation, queries, zone loading, and failures.

$EvidencePath = "C:\DNS-Delegation"

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
    $_.Message -like "*delegat*" -or
    $_.Message -like "*recursion*"
  } |
  Tee-Object "$EvidencePath\system-dns-delegation-events.txt"

# Final service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-final.txt"

# Final zones and relevant records.
Import-Module DnsServer

Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-final.txt"
```

# Configure_DNS_Delegation_Verification_Commands
```powershell
# Confirm DNS role and service.
Get-WindowsFeature DNS
Get-Service DNS

# Import DNS module.
Import-Module DnsServer

# Confirm parent zone.
Get-DnsServerZone -Name "<parent-zone-name>"
Get-DnsServerResourceRecord -ZoneName "<parent-zone-name>"

# Confirm child zone on child DNS server.
Get-DnsServerZone -Name "<child-zone-name>"
Get-DnsServerResourceRecord -ZoneName "<child-zone-name>" -RRType SOA
Get-DnsServerResourceRecord -ZoneName "<child-zone-name>" -RRType NS

# Create child zone if needed.
Add-DnsServerPrimaryZone `
  -Name "<child-zone-name>" `
  -DynamicUpdate Secure

# Add child test record.
Add-DnsServerResourceRecordA `
  -ZoneName "<child-zone-name>" `
  -Name "<test-record-name>" `
  -IPv4Address "<test-record-ip>"

# Add parent delegation NS records.
Add-DnsServerResourceRecord `
  -ZoneName "<parent-zone-name>" `
  -Name "<delegation-label>" `
  -NS `
  -NameServer "<child-dns-server-1-fqdn>."

Add-DnsServerResourceRecord `
  -ZoneName "<parent-zone-name>" `
  -Name "<delegation-label>" `
  -NS `
  -NameServer "<child-dns-server-2-fqdn>."

# Add glue records if child DNS server names live inside delegated namespace.
Add-DnsServerResourceRecordA `
  -ZoneName "<parent-zone-name>" `
  -Name "<child-dns-host-label-1>" `
  -IPv4Address "<child-dns-server-1-ip>"

Add-DnsServerResourceRecordA `
  -ZoneName "<parent-zone-name>" `
  -Name "<child-dns-host-label-2>" `
  -IPv4Address "<child-dns-server-2-ip>"

# Validate parent delegation records.
Get-DnsServerResourceRecord -ZoneName "<parent-zone-name>" -Name "<delegation-label>"

# Validate glue records.
Get-DnsServerResourceRecord -ZoneName "<parent-zone-name>" -Name "<child-dns-host-label-1>"
Get-DnsServerResourceRecord -ZoneName "<parent-zone-name>" -Name "<child-dns-host-label-2>"

# Direct child authoritative queries.
Resolve-DnsName "<child-zone-name>" -Type SOA -Server "<child-dns-server-1-ip>"
Resolve-DnsName "<child-zone-name>" -Type NS -Server "<child-dns-server-1-ip>"
Resolve-DnsName "<test-record-fqdn>" -Server "<child-dns-server-1-ip>"

# Queries through parent DNS.
Resolve-DnsName "<child-zone-name>" -Type SOA -Server "<parent-dns-ip>"
Resolve-DnsName "<child-zone-name>" -Type NS -Server "<parent-dns-ip>"
Resolve-DnsName "<test-record-fqdn>" -Server "<parent-dns-ip>"

# Client resolver validation.
ipconfig /all
Clear-DnsClientCache
Resolve-DnsName "<test-record-fqdn>" -Server "<parent-dns-ip>"

# Connectivity.
Test-NetConnection "<parent-dns-ip>" -Port 53
Test-NetConnection "<child-dns-server-1-ip>" -Port 53
Test-NetConnection "<child-dns-server-2-ip>" -Port 53

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
```

# Configure_DNS_Delegation_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Wrong delegation NS record added | Remove incorrect NS record from parent zone | Child namespace may stop resolving if all NS records are removed |
| Wrong glue record added | Remove incorrect A glue record from parent zone | Delegated server name may fail to resolve |
| Child zone created on wrong server | `Remove-DnsServerZone -Name "<child-zone-name>" -Force` on wrong server | Removes child authoritative zone from that server |
| Test A record created | Remove test record from child zone | Test validation record disappears |
| Delegation points to unreachable DNS server | Replace NS/glue with reachable child DNS server | Queries may timeout until corrected |
| Firewall opened too broadly | Restrict TCP/UDP 53 to approved DNS paths | Delegation may fail if DNS traffic is blocked |
| Parent zone record inventory changed | Restore from exported record inventory or zone backup | Manual record restore can be error-prone |
| DNS cache polluted during testing | `Clear-DnsServerCache` and `Clear-DnsClientCache` | Temporary cache flush impact |
| Evidence folder created | `Remove-Item C:\DNS-Delegation -Recurse -Force` | Deletes validation evidence |

# Configure_DNS_Delegation_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Parent cannot resolve child records | Delegation NS/glue missing or child DNS unreachable | Parent delegation records and `Test-NetConnection <child-dns-ip> -Port 53` | Recreating the parent zone |
| Direct child DNS query works but parent query fails | Parent delegation issue | NS and glue records in parent zone | Editing child records first |
| Parent returns NS but host lookup fails | Child DNS zone missing target record or recursion path issue | Query test record directly against child DNS | Removing delegation |
| SOA query for child fails | Child zone not authoritative or child DNS unreachable | `Resolve-DnsName <child-zone> -Type SOA -Server <child-dns-ip>` | Changing parent SOA |
| NS record points to wrong child DNS name | Bad delegation record | Parent delegation NS records | Rebuilding child DNS server |
| Glue record points to wrong IP | Bad glue A record | Parent glue record and child DNS server IP | Changing client DNS settings |
| Client fails but parent DNS server works | Client resolver or cache issue | `ipconfig /all`; `Clear-DnsClientCache`; query parent DNS directly | Editing delegation immediately |
| Timeout instead of NXDOMAIN | Firewall/routing/path to child DNS | TCP/UDP 53 path to child DNS servers | Deleting zones |
| NXDOMAIN for child zone | Missing child zone or wrong delegation label | Zone name and delegation label | Restarting DNS service repeatedly |
| Stub/conditional forwarder conflict | Competing DNS resolution design | Check existing stub zones and conditional forwarders | Adding more delegation records blindly |
| AD child domain join fails | Delegation or child DC DNS missing | Child zone SOA/SRV records and parent delegation | Reinstalling AD DS |
| Delegated DNS server name cannot resolve | Glue missing or child DNS name inside child zone | Parent glue records | Changing SOA serial |
| Some clients resolve and others fail | Resolver path/cache/site difference | Query exact DNS server each client uses | Assuming DNS delegation is globally broken |
| External/public lookup fails | Internal delegation not published externally | Check which DNS namespace is authoritative | Mixing internal and public DNS changes |
| DNS event log shows recursion failure | Server cannot reach delegated authoritative server | Routing/firewall/forwarder path | Editing child A records |
| Zone exists on parent and child with same name | Namespace conflict | `Get-DnsServerZone` on all servers | Leaving duplicate authoritative zones |

# Configure_DNS_Delegation_Related_Labs
| Lab                                                     | Related Workbook                                              | Skill Proven                                               |
| ------------------------------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------- |
| Configure AD-integrated DNS zones                       | `07_Configure_AD_Integrated_DNS_Zones.md`                     | Parent and child zone baseline                             |
| Validate AD DNS SRV records and DC locator              | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md`            | AD DNS health before child namespace delegation            |
| Configure DNS dynamic updates                           | `09_Configure_DNS_Dynamic_Updates.md`                         | Dynamic record behavior inside delegated namespaces        |
| Configure DNS aging and scavenging                      | `10_Configure_DNS_Aging_And_Scavenging.md`                    | Stale record cleanup before delegating or exposing records |
| Configure DNS zone transfers, secondary, and stub zones | `11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md` | Alternative DNS namespace distribution patterns            |
| Configure DNS delegation                                | `12_Configure_DNS_Delegation.md`                              | Parent-to-child namespace authority handoff                |
| Configure DNS conditional forwarders                    | `13_Configure_DNS_Conditional_Forwarders.md`                  | Alternative to delegation for query forwarding             |
| Troubleshoot DNS delegation and forwarding failures     | `14_Troubleshoot_DNS_Delegation_And_Forwarding_Failures.md`   | Delegation, glue, forwarder, and resolver path triage      |