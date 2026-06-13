11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md
# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Index
11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Source_Basis
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Mental_Model
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Planning_Table
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Configuration_Checklist
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Precheck_Skeleton
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Primary_Zone_Transfer_Skeleton
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Secondary_Zone_Skeleton
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Stub_Zone_Skeleton
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Notify_And_Refresh_Skeleton
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Validation_Skeleton
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Event_Log_Skeleton
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Verification_Commands
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Rollback
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Failure_Checks
Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Related_Labs

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | Managing Windows Server DNS zones, records, and transfer behavior |
| Microsoft Learn | Get-DnsServerZone | Reviewing primary, secondary, stub, AD-integrated, and file-backed zones |
| Microsoft Learn | Set-DnsServerPrimaryZone | Configuring primary zone transfer behavior and notification behavior |
| Microsoft Learn | Add-DnsServerSecondaryZone | Creating secondary zones from primary DNS master servers |
| Microsoft Learn | Add-DnsServerStubZone | Creating stub zones that contain delegation-focused records |
| Microsoft Learn | Remove-DnsServerZone | Removing incorrectly created secondary or stub zones |
| Microsoft Learn | Get-DnsServerResourceRecord | Validating SOA, NS, A, and glue records after zone replication or stub update |
| Microsoft Learn | Resolve-DnsName | Testing queries against primary, secondary, and stub-aware DNS servers |
| Microsoft Learn | DNS Server event logs | Reviewing zone transfer, notify, refresh, load, and failure events |
| Windows Server operational practice | Secure zone transfers, secondary zones, stub zones, conditional forwarding comparison | Replicating or delegating DNS resolution data without exposing transfers to untrusted hosts |

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Zone transfer | DNS mechanism where a secondary server copies zone data from a primary or master DNS server |
| AXFR | Full zone transfer |
| IXFR | Incremental zone transfer when supported |
| Primary zone | Writable source zone that owns the authoritative DNS data |
| Secondary zone | Read-only copy of a primary zone hosted on another DNS server |
| Stub zone | Zone that stores only SOA, NS, and glue A records for another zone |
| Master server | DNS server from which a secondary or stub zone pulls zone data |
| Notify | Mechanism where primary server informs secondary servers that zone data changed |
| SOA serial | Version number used by secondary servers to detect zone changes |
| Allowed transfer server | DNS server explicitly permitted to request zone transfers |
| Open zone transfer risk | Unsafe condition where any host can request full zone data |
| AD-integrated zone | DNS zone replicated through Active Directory; usually does not require classic secondary zone transfer between DCs |
| File-backed secondary zone | DNS zone stored locally as a read-only file-backed zone on the secondary server |
| Stub versus secondary | Secondary copies full zone data; stub copies only records needed to find authoritative servers |
| Conditional forwarder comparison | Conditional forwarders forward queries; stub zones maintain authoritative server awareness through NS/SOA/glue records |
| First rule | Never allow zone transfers to any server; allow only approved secondary DNS servers and validate the SOA serial, NS records, event logs, and query behavior |

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Primary DNS server | `DC1.corp.local` | `<primary-dns-server>` |
| Primary DNS server IP | `10.10.10.10` | `<primary-dns-ip>` |
| Secondary DNS server | `DNS2.corp.local` | `<secondary-dns-server>` |
| Secondary DNS server IP | `10.10.10.11` | `<secondary-dns-ip>` |
| Stub DNS server | `DNS3.branch.local` | `<stub-dns-server>` |
| Stub DNS server IP | `10.20.10.10` | `<stub-dns-ip>` |
| Zone name | `corp.local` | `<zone-name>` |
| Zone type on primary | Primary / AD-integrated primary | `<primary-zone-type>` |
| Zone type on secondary | Secondary | `<secondary-zone-type>` |
| Zone type for delegated awareness | Stub | `<stub-zone-type>` |
| Master server list | `10.10.10.10` | `<master-server-list>` |
| Zone transfer allowed to | `10.10.10.11` | `<allowed-transfer-server-ip>` |
| Notify server list | `10.10.10.11` | `<notify-server-list>` |
| Zone file name | `corp.local.dns` | `<zone-file-name>` |
| Stub zone file name | `corp.local.stub.dns` | `<stub-zone-file-name>` |
| Expected SOA serial | Primary and secondary match after transfer | `<soa-serial-expectation>` |
| Expected NS records | Primary authoritative servers | `<expected-ns-records>` |
| Expected test record | `dc1.corp.local` | `<test-record>` |
| Query source | Secondary DNS server and client | `<query-source>` |
| Firewall requirement | TCP/UDP 53 between DNS servers | `<firewall-requirement>` |
| Evidence path | `C:\DNS-Zone-Transfers` | `<evidence-path>` |
| Rollback stance | Disable unsafe transfer, remove secondary/stub zones if wrong | `<rollback-plan>` |
| Next workbook | `12_Troubleshoot_DNS_Zone_Transfer_And_Replication_Failures.md` | `<next-task>` |

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Primary DNS Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DNS role on primary | Primary DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 3 | Confirm DNS service on primary | Primary DNS Server | `Get-Service DNS` | DNS service is Running |
| 4 | Confirm DNS role on secondary | Secondary DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 5 | Confirm DNS service on secondary | Secondary DNS Server | `Get-Service DNS` | DNS service is Running |
| 6 | Import DNS module | Primary / Secondary | `Import-Module DnsServer` | DNS cmdlets are available |
| 7 | Confirm primary zone exists | Primary DNS Server | `Get-DnsServerZone -Name "<zone-name>"` | Zone exists on primary |
| 8 | Confirm primary zone records | Primary DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | Zone contains expected records |
| 9 | Confirm SOA on primary | Primary DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType SOA` | SOA record is visible |
| 10 | Confirm NS records on primary | Primary DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NS` | Authoritative NS records are visible |
| 11 | Confirm connectivity between DNS servers | Secondary DNS Server | `Test-NetConnection "<primary-dns-ip>" -Port 53` | DNS TCP 53 path is reachable |
| 12 | Configure primary zone transfers | Primary DNS Server | `Set-DnsServerPrimaryZone -Name "<zone-name>" -SecureSecondaries TransferToSecureServers -SecondaryServers "<secondary-dns-ip>"` | Only approved secondary can request transfer |
| 13 | Configure notify list | Primary DNS Server | `Set-DnsServerPrimaryZone -Name "<zone-name>" -Notify NotifyServers -NotifyServers "<secondary-dns-ip>"` | Primary notifies approved secondary |
| 14 | Create secondary zone | Secondary DNS Server | `Add-DnsServerSecondaryZone -Name "<zone-name>" -ZoneFile "<zone-file-name>" -MasterServers "<primary-dns-ip>"` | Secondary zone is created |
| 15 | Confirm secondary zone loaded | Secondary DNS Server | `Get-DnsServerZone -Name "<zone-name>"` | Zone appears as secondary |
| 16 | Confirm records on secondary | Secondary DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | Records copied from primary |
| 17 | Compare SOA serial | Primary / Secondary | Query SOA on both servers | SOA serial matches after transfer |
| 18 | Test resolution against secondary | Client / Admin Host | `Resolve-DnsName "<test-record>" -Server "<secondary-dns-ip>"` | Secondary answers correctly |
| 19 | Create stub zone if needed | Stub DNS Server | `Add-DnsServerStubZone -Name "<zone-name>" -ZoneFile "<stub-zone-file-name>" -MasterServers "<primary-dns-ip>"` | Stub zone is created |
| 20 | Confirm stub zone records | Stub DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | SOA, NS, and glue records are present |
| 21 | Validate stub-aware resolution | Stub DNS Server / Client | `Resolve-DnsName "<test-record>" -Server "<stub-dns-ip>"` | Server can locate authoritative DNS path |
| 22 | Review DNS logs on primary | Primary DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | Zone transfer and notify events are visible |
| 23 | Review DNS logs on secondary | Secondary DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | Zone load and transfer events are visible |
| 24 | Export evidence | Primary / Secondary / Stub | Run validation skeletons | Evidence files are saved |
| 25 | Document final state | Operator | `Record allowed transfer servers, notify list, secondary zones, stub zones, SOA serials, and query tests` | Build record is complete |

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the primary DNS server.
# Purpose: capture current DNS role, zone, record, SOA, NS, and transfer readiness state.

$ZoneName = "corp.local"
$PrimaryDnsIp = "10.10.10.10"
$SecondaryDnsIp = "10.10.10.11"
$EvidencePath = "C:\DNS-Zone-Transfers"

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

# Capture all zones.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-before.txt"

# Capture target primary zone.
Get-DnsServerZone -Name $ZoneName |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\target-zone-properties-before.txt"

# Capture target zone records.
Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\target-zone-records-before.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Export-Csv "$EvidencePath\target-zone-records-before.csv" -NoTypeInformation

# Capture SOA and NS records.
Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType SOA |
  Tee-Object "$EvidencePath\target-zone-soa-before.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType NS |
  Tee-Object "$EvidencePath\target-zone-ns-before.txt"

# Confirm path to secondary DNS server.
Test-NetConnection $SecondaryDnsIp -Port 53 |
  Tee-Object "$EvidencePath\test-secondary-dns-tcp-53.txt"

# Capture DNS events before change.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-before.txt"
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Primary_Zone_Transfer_Skeleton
```powershell
# Run in elevated PowerShell on the primary DNS server.
# Purpose: allow zone transfers only to approved secondary DNS servers.
# Never use open transfer behavior in production.

$ZoneName = "corp.local"
$SecondaryDnsIp = "10.10.10.11"
$EvidencePath = "C:\DNS-Zone-Transfers"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\primary-zone-transfer-config-transcript.txt"

Import-Module DnsServer

# Capture current primary zone settings.
Get-DnsServerZone -Name $ZoneName |
  Tee-Object "$EvidencePath\primary-zone-before-transfer-config.txt"

# Configure transfer only to secure/approved secondary servers.
Set-DnsServerPrimaryZone `
  -Name $ZoneName `
  -SecureSecondaries TransferToSecureServers `
  -SecondaryServers $SecondaryDnsIp

# Configure DNS notify to approved secondary servers.
Set-DnsServerPrimaryZone `
  -Name $ZoneName `
  -Notify NotifyServers `
  -NotifyServers $SecondaryDnsIp

# Confirm zone properties after transfer configuration.
Get-DnsServerZone -Name $ZoneName |
  Tee-Object "$EvidencePath\primary-zone-after-transfer-config.txt"

# Confirm SOA and NS records are still present.
Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType SOA |
  Tee-Object "$EvidencePath\primary-zone-soa-after-transfer-config.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType NS |
  Tee-Object "$EvidencePath\primary-zone-ns-after-transfer-config.txt"

Stop-Transcript
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Secondary_Zone_Skeleton
```powershell
# Run in elevated PowerShell on the secondary DNS server.
# Purpose: create a read-only secondary copy of the primary zone.

$ZoneName = "corp.local"
$ZoneFile = "corp.local.dns"
$PrimaryDnsIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Zone-Transfers"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\secondary-zone-create-transcript.txt"

Import-Module DnsServer

# Confirm DNS service.
Get-Service DNS |
  Tee-Object "$EvidencePath\secondary-dns-service-before.txt"

# Confirm connectivity to primary.
Test-NetConnection $PrimaryDnsIp -Port 53 |
  Tee-Object "$EvidencePath\secondary-to-primary-tcp-53.txt"

# Check whether secondary zone already exists.
$ExistingZone = Get-DnsServerZone -Name $ZoneName -ErrorAction SilentlyContinue

if ($ExistingZone) {
  Write-Output "Zone $ZoneName already exists on this server. Review before creating a secondary zone."
  $ExistingZone |
    Tee-Object "$EvidencePath\secondary-zone-existing.txt"
}
else {
  # Create secondary zone from primary master.
  Add-DnsServerSecondaryZone `
    -Name $ZoneName `
    -ZoneFile $ZoneFile `
    -MasterServers $PrimaryDnsIp

  # Confirm secondary zone after creation.
  Get-DnsServerZone -Name $ZoneName |
    Tee-Object "$EvidencePath\secondary-zone-after-create.txt"
}

# Validate records loaded on secondary.
Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\secondary-zone-records-after-create.txt"

# Capture SOA and NS records on secondary.
Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType SOA -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\secondary-zone-soa-after-create.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType NS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\secondary-zone-ns-after-create.txt"

Stop-Transcript
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Stub_Zone_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server that should host the stub zone.
# Purpose: create a stub zone that tracks authoritative DNS servers for another zone.
# Stub zones store SOA, NS, and glue records, not the full zone contents.

$ZoneName = "corp.local"
$StubZoneFile = "corp.local.stub.dns"
$MasterServer = "10.10.10.10"
$EvidencePath = "C:\DNS-Stub-Zone"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\stub-zone-create-transcript.txt"

Import-Module DnsServer

# Confirm DNS role and service.
Get-WindowsFeature DNS |
  Tee-Object "$EvidencePath\dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-state.txt"

# Confirm connectivity to master server.
Test-NetConnection $MasterServer -Port 53 |
  Tee-Object "$EvidencePath\stub-to-master-tcp-53.txt"

# Check whether stub zone already exists.
$ExistingZone = Get-DnsServerZone -Name $ZoneName -ErrorAction SilentlyContinue

if ($ExistingZone) {
  Write-Output "Zone $ZoneName already exists on this server. Review before creating stub zone."
  $ExistingZone |
    Tee-Object "$EvidencePath\stub-zone-existing.txt"
}
else {
  # Create stub zone.
  Add-DnsServerStubZone `
    -Name $ZoneName `
    -ZoneFile $StubZoneFile `
    -MasterServers $MasterServer

  # Confirm stub zone after creation.
  Get-DnsServerZone -Name $ZoneName |
    Tee-Object "$EvidencePath\stub-zone-after-create.txt"
}

# Confirm stub zone records.
Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\stub-zone-records-after-create.txt"

# Stub zone should contain SOA, NS, and glue A records.
Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType SOA -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\stub-zone-soa-record.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType NS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\stub-zone-ns-records.txt"

Stop-Transcript
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Notify_And_Refresh_Skeleton
```powershell
# Run in elevated PowerShell on primary and secondary DNS servers as needed.
# Purpose: validate SOA serial, notify behavior, and secondary refresh behavior.

$ZoneName = "corp.local"
$PrimaryDnsIp = "10.10.10.10"
$SecondaryDnsIp = "10.10.10.11"
$TestRecordName = "zonexfer-test"
$TestRecordIp = "10.10.10.250"
$EvidencePath = "C:\DNS-Zone-Transfers"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Capture SOA on primary.
Resolve-DnsName $ZoneName -Type SOA -Server $PrimaryDnsIp |
  Tee-Object "$EvidencePath\soa-primary-before-test.txt"

# Capture SOA on secondary.
Resolve-DnsName $ZoneName -Type SOA -Server $SecondaryDnsIp |
  Tee-Object "$EvidencePath\soa-secondary-before-test.txt"

# Optional lab-only test:
# Add a temporary record on the primary to increment zone serial / trigger notify.
# Do not use in production zones without change control.
# Add-DnsServerResourceRecordA `
#   -ZoneName $ZoneName `
#   -Name $TestRecordName `
#   -IPv4Address $TestRecordIp `
#   -TimeToLive 00:05:00

# Wait for notify/refresh.
Start-Sleep -Seconds 30

# Query test record against primary and secondary.
Resolve-DnsName "$TestRecordName.$ZoneName" -Server $PrimaryDnsIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-test-record-primary.txt"

Resolve-DnsName "$TestRecordName.$ZoneName" -Server $SecondaryDnsIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-test-record-secondary.txt"

# Compare SOA after change/refresh.
Resolve-DnsName $ZoneName -Type SOA -Server $PrimaryDnsIp |
  Tee-Object "$EvidencePath\soa-primary-after-test.txt"

Resolve-DnsName $ZoneName -Type SOA -Server $SecondaryDnsIp |
  Tee-Object "$EvidencePath\soa-secondary-after-test.txt"

# Optional lab cleanup.
# Remove-DnsServerResourceRecord `
#   -ZoneName $ZoneName `
#   -Name $TestRecordName `
#   -RRType A `
#   -RecordData $TestRecordIp `
#   -Force
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Validation_Skeleton
```powershell
# Run from an admin host that can query primary, secondary, and stub DNS servers.
# Purpose: validate primary, secondary, and stub zone behavior.

$ZoneName = "corp.local"
$PrimaryDnsIp = "10.10.10.10"
$SecondaryDnsIp = "10.10.10.11"
$StubDnsIp = "10.20.10.10"
$TestRecord = "dc1.corp.local"
$EvidencePath = "C:\DNS-Zone-Transfers"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Query zone SOA from primary and secondary.
Resolve-DnsName $ZoneName -Type SOA -Server $PrimaryDnsIp |
  Tee-Object "$EvidencePath\validate-soa-primary.txt"

Resolve-DnsName $ZoneName -Type SOA -Server $SecondaryDnsIp |
  Tee-Object "$EvidencePath\validate-soa-secondary.txt"

# Query NS records from primary and secondary.
Resolve-DnsName $ZoneName -Type NS -Server $PrimaryDnsIp |
  Tee-Object "$EvidencePath\validate-ns-primary.txt"

Resolve-DnsName $ZoneName -Type NS -Server $SecondaryDnsIp |
  Tee-Object "$EvidencePath\validate-ns-secondary.txt"

# Query normal record from primary and secondary.
Resolve-DnsName $TestRecord -Server $PrimaryDnsIp |
  Tee-Object "$EvidencePath\validate-test-record-primary.txt"

Resolve-DnsName $TestRecord -Server $SecondaryDnsIp |
  Tee-Object "$EvidencePath\validate-test-record-secondary.txt"

# Query through stub DNS server if stub zone exists.
Resolve-DnsName $ZoneName -Type SOA -Server $StubDnsIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\validate-soa-stub-server.txt"

Resolve-DnsName $ZoneName -Type NS -Server $StubDnsIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\validate-ns-stub-server.txt"

Resolve-DnsName $TestRecord -Server $StubDnsIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\validate-test-record-through-stub-server.txt"

# Test DNS TCP 53 path to each server.
Test-NetConnection $PrimaryDnsIp -Port 53 |
  Tee-Object "$EvidencePath\test-primary-tcp-53.txt"

Test-NetConnection $SecondaryDnsIp -Port 53 |
  Tee-Object "$EvidencePath\test-secondary-tcp-53.txt"

Test-NetConnection $StubDnsIp -Port 53 |
  Tee-Object "$EvidencePath\test-stub-tcp-53.txt"
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Event_Log_Skeleton
```powershell
# Run on primary, secondary, and stub DNS servers as applicable.
# Purpose: collect DNS Server events related to zone transfer, notify, zone load, and failures.

$EvidencePath = "C:\DNS-Zone-Transfers"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server event log.
Get-WinEvent -LogName "DNS Server" -MaxEvents 300 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events.txt"

# DNS Server operational log if available.
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 300 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events.txt"

# System DNS-related events.
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*zone transfer*" -or
    $_.Message -like "*secondary*" -or
    $_.Message -like "*stub zone*" -or
    $_.Message -like "*notify*"
  } |
  Tee-Object "$EvidencePath\system-dns-zone-transfer-events.txt"

# Final DNS service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-final.txt"

# Final zone list.
Import-Module DnsServer

Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-final.txt"
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Verification_Commands
```powershell
# Confirm DNS role and service.
Get-WindowsFeature DNS
Get-Service DNS

# Import DNS module.
Import-Module DnsServer

# List zones.
Get-DnsServerZone

# Confirm primary zone.
Get-DnsServerZone -Name "<zone-name>"
Get-DnsServerZone -Name "<zone-name>" | Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Confirm primary zone records.
Get-DnsServerResourceRecord -ZoneName "<zone-name>"
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType SOA
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NS

# Configure primary zone transfer to approved secondary.
Set-DnsServerPrimaryZone `
  -Name "<zone-name>" `
  -SecureSecondaries TransferToSecureServers `
  -SecondaryServers "<secondary-dns-ip>"

# Configure notify to approved secondary.
Set-DnsServerPrimaryZone `
  -Name "<zone-name>" `
  -Notify NotifyServers `
  -NotifyServers "<secondary-dns-ip>"

# Create secondary zone.
Add-DnsServerSecondaryZone `
  -Name "<zone-name>" `
  -ZoneFile "<zone-file-name>" `
  -MasterServers "<primary-dns-ip>"

# Create stub zone.
Add-DnsServerStubZone `
  -Name "<zone-name>" `
  -ZoneFile "<stub-zone-file-name>" `
  -MasterServers "<primary-dns-ip>"

# Confirm secondary or stub zone.
Get-DnsServerZone -Name "<zone-name>"
Get-DnsServerResourceRecord -ZoneName "<zone-name>"

# Query primary and secondary.
Resolve-DnsName "<zone-name>" -Type SOA -Server "<primary-dns-ip>"
Resolve-DnsName "<zone-name>" -Type SOA -Server "<secondary-dns-ip>"
Resolve-DnsName "<zone-name>" -Type NS -Server "<primary-dns-ip>"
Resolve-DnsName "<zone-name>" -Type NS -Server "<secondary-dns-ip>"
Resolve-DnsName "<test-record-fqdn>" -Server "<primary-dns-ip>"
Resolve-DnsName "<test-record-fqdn>" -Server "<secondary-dns-ip>"

# Query through stub DNS server if applicable.
Resolve-DnsName "<zone-name>" -Type SOA -Server "<stub-dns-ip>"
Resolve-DnsName "<zone-name>" -Type NS -Server "<stub-dns-ip>"
Resolve-DnsName "<test-record-fqdn>" -Server "<stub-dns-ip>"

# Connectivity tests.
Test-NetConnection "<primary-dns-ip>" -Port 53
Test-NetConnection "<secondary-dns-ip>" -Port 53
Test-NetConnection "<stub-dns-ip>" -Port 53

# Remove wrong secondary or stub zone.
Remove-DnsServerZone -Name "<zone-name>" -Force

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DNS*" -or $_.Message -like "*zone transfer*"}
```

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Zone transfers allowed to wrong server | Re-run `Set-DnsServerPrimaryZone` with correct `-SecondaryServers` list | Wrong server may retain copied zone data |
| Zone transfers opened too broadly | Restrict transfer to secure/approved servers or disable transfer | Full zone disclosure risk |
| Notify configured to wrong server | Re-run `Set-DnsServerPrimaryZone -Notify NotifyServers -NotifyServers "<correct-ip>"` | Wrong server may receive unnecessary notifications |
| Secondary zone created on wrong server | `Remove-DnsServerZone -Name "<zone-name>" -Force` on wrong server | Removes the secondary copy only |
| Stub zone created on wrong server | `Remove-DnsServerZone -Name "<zone-name>" -Force` on wrong server | Removes stub zone only |
| Wrong master server configured | Remove and recreate secondary/stub zone with correct master list | Zone may not refresh until fixed |
| Temporary test record created | Remove test record from primary zone | Wrong removal can break real records |
| Firewall opened too broadly | Restrict TCP/UDP 53 to approved DNS servers | Transfer or query path may break if too restrictive |
| Evidence folder created | `Remove-Item C:\DNS-Zone-Transfers -Recurse -Force` | Deletes validation evidence |
| DNS service restarted | No rollback normally needed | Brief DNS outage possible |

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Secondary zone does not load | Transfer blocked, wrong master, firewall, or primary disallows transfer | DNS event log and `Test-NetConnection <primary> -Port 53` | Recreating the primary zone |
| Secondary has old records | Notify/refresh issue or SOA serial not updated | Compare SOA serial on primary and secondary | Reinstalling DNS role |
| Transfer refused | Primary zone transfer ACL does not allow secondary | Primary `Set-DnsServerPrimaryZone` transfer settings | Opening transfer to any server |
| Secondary can query primary but cannot transfer | TCP 53 or zone transfer setting issue | Firewall and primary transfer policy | Editing client DNS settings |
| Stub zone has no NS records | Master unreachable or zone not authoritative on master | `Resolve-DnsName <zone> -Type NS -Server <master>` | Creating full secondary zone blindly |
| Stub zone resolves SOA/NS but not host records | Stub zone only stores delegation data | Confirm recursion/forwarding path | Treating stub like a secondary |
| Client receives SERVFAIL through stub server | Recursion, authoritative path, or firewall issue | Query SOA/NS and test authoritative server reachability | Deleting stub zone first |
| SOA serial differs after long wait | Transfer/notify failure or stale secondary | Event logs and transfer settings | Editing random records |
| Zone transfer works from one secondary but not another | ACL or firewall only permits one IP | Compare secondary IPs and allowed list | Disabling secure transfer |
| Event log shows zone transfer denied | Primary ACL/permission issue | Confirm secondary IP in allowed transfer list | Restarting all DNS servers |
| Event log shows timeout | Network/firewall/path issue | TCP/UDP 53 path and routing | Changing SOA manually |
| Secondary zone is writable | Wrong zone type | `Get-DnsServerZone -Name <zone>` | Editing records on secondary |
| AD-integrated zone expected but secondary created | Design mismatch | Determine whether AD replication should be used instead | Mixing AD-integrated and secondary casually |
| Public zone exposes internal data | Zone transfer/data classification issue | Review zone contents and transfer ACL | Publishing zone externally |
| Notify not working | Notify list wrong or blocked | Primary zone notify settings and DNS events | Assuming transfer itself is broken |
| Master server IP changed | Secondary/stub still points to old master | Review master server list | Rebuilding secondary DNS server |

# Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones_Related_Labs
| Lab                                                     | Related Workbook                                                | Skill Proven                                                  |
| ------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------- |
| Configure AD-integrated DNS zones                       | `07_Configure_AD_Integrated_DNS_Zones.md`                       | Primary AD DNS zone baseline                                  |
| Validate AD DNS SRV records and DC locator              | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md`              | Critical AD DNS validation before transfer design             |
| Configure DNS dynamic updates                           | `09_Configure_DNS_Dynamic_Updates.md`                           | Dynamic record behavior before replication/transfer decisions |
| Configure DNS aging and scavenging                      | `10_Configure_DNS_Aging_And_Scavenging.md`                      | Stale record cleanup before copying records elsewhere         |
| Configure DNS zone transfers, secondary, and stub zones | `11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md`   | Secure zone transfer, secondary zone, and stub zone operation |
| Troubleshoot DNS zone transfer and replication failures | `12_Troubleshoot_DNS_Zone_Transfer_And_Replication_Failures.md` | Failed secondary/stub/transfer root-cause workflow            |
| Monitor DNS records, events, and stale record cleanup   | `13_Monitor_DNS_Records_Events_And_Stale_Record_Cleanup.md`     | Long-term DNS operational visibility                          |