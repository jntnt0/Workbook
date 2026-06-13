07_Configure_AD_Integrated_DNS_Zones.md
# Configure_AD_Integrated_DNS_Zones

# Configure_AD_Integrated_DNS_Zones_Index
07_Configure_AD_Integrated_DNS_Zones.md
Configure_AD_Integrated_DNS_Zones
Configure_AD_Integrated_DNS_Zones_Source_Basis
Configure_AD_Integrated_DNS_Zones_Mental_Model
Configure_AD_Integrated_DNS_Zones_Planning_Table
Configure_AD_Integrated_DNS_Zones_Configuration_Checklist
Configure_AD_Integrated_DNS_Zones_Precheck_Skeleton
Configure_AD_Integrated_DNS_Zones_Forward_Zone_Skeleton
Configure_AD_Integrated_DNS_Zones_Reverse_Zone_Skeleton
Configure_AD_Integrated_DNS_Zones_Secure_Dynamic_Update_Skeleton
Configure_AD_Integrated_DNS_Zones_Replication_Scope_Skeleton
Configure_AD_Integrated_DNS_Zones_Record_Validation_Skeleton
Configure_AD_Integrated_DNS_Zones_Client_Validation_Skeleton
Configure_AD_Integrated_DNS_Zones_Event_Log_Skeleton
Configure_AD_Integrated_DNS_Zones_Verification_Commands
Configure_AD_Integrated_DNS_Zones_Rollback
Configure_AD_Integrated_DNS_Zones_Failure_Checks
Configure_AD_Integrated_DNS_Zones_Related_Labs

# Configure_AD_Integrated_DNS_Zones_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | Managing Windows Server DNS zones and records |
| Microsoft Learn | Add-DnsServerPrimaryZone | Creating forward and reverse primary zones |
| Microsoft Learn | Set-DnsServerPrimaryZone | Configuring dynamic updates and zone properties |
| Microsoft Learn | Get-DnsServerZone | Listing and validating DNS zones |
| Microsoft Learn | Get-DnsServerResourceRecord | Validating A, PTR, NS, SOA, SRV, and CNAME records |
| Microsoft Learn | Register-DnsClient | Forcing client DNS registration |
| Microsoft Learn | Resolve-DnsName | Testing DNS resolution from server or client |
| Microsoft Learn | dcdiag /test:dns | Validating domain controller DNS health |
| Microsoft Learn | repadmin | Validating AD replication for AD-integrated DNS data |
| Windows Server operational practice | AD-integrated zones, secure dynamic updates, reverse zones, DC locator records | Building DNS zones that support Active Directory domain operations |

# Configure_AD_Integrated_DNS_Zones_Mental_Model
| Concept | Operational Meaning |
|---|---|
| AD-integrated DNS zone | DNS zone stored in Active Directory instead of a standalone zone file |
| Forward lookup zone | Zone used to resolve names to IP addresses |
| Reverse lookup zone | Zone used to resolve IP addresses back to names |
| Replication scope | Controls which domain controllers replicate the zone data |
| Secure dynamic updates | Allows only authenticated domain members to register or update DNS records |
| SRV records | Service locator records used by clients to find domain controllers, Kerberos, LDAP, and Global Catalog services |
| `_msdcs` zone | AD DNS zone used heavily for domain controller and forest-wide locator records |
| A record | Hostname-to-IPv4 mapping |
| PTR record | IPv4-to-hostname mapping inside a reverse lookup zone |
| NS record | Identifies authoritative DNS servers for the zone |
| SOA record | Defines authoritative zone metadata and versioning |
| DC locator | Client process that uses DNS SRV records to find domain controllers |
| Dynamic registration | Process where domain members register DNS records automatically |
| First rule | Active Directory depends on DNS; domain joins, logons, GPO, replication, and DC discovery all fail when AD DNS is wrong |

# Configure_AD_Integrated_DNS_Zones_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Primary domain controller | `DC1.corp.local` | `<dc1-fqdn>` |
| Secondary domain controller | `DC2.corp.local` | `<dc2-fqdn>` |
| DNS server IP 1 | `10.10.10.10` | `<dns-server-1>` |
| DNS server IP 2 | `10.10.10.11` | `<dns-server-2>` |
| Forward lookup zone | `corp.local` | `<forward-zone-name>` |
| Forest locator zone | `_msdcs.corp.local` | `<msdcs-zone-name>` |
| Reverse lookup network | `10.10.10.0/24` | `<reverse-zone-network>` |
| Reverse lookup zone | `10.10.10.in-addr.arpa` | `<reverse-zone-name>` |
| Replication scope | Domain or Forest | `<replication-scope>` |
| Dynamic update mode | Secure only | `<dynamic-update-mode>` |
| Zone type | AD-integrated primary | `<zone-type>` |
| Allow zone transfers | No by default | `<yes-no>` |
| DHCP DNS dynamic update dependency | DHCP server updates A/PTR for clients | `<yes-no>` |
| Client DNS suffix | `corp.local` | `<dns-suffix>` |
| Evidence path | `C:\DNS-AD-Integrated-Zones` | `<evidence-path>` |
| Rollback stance | Remove only test-created zones or records after confirming AD impact | `<rollback-plan>` |
| Next workbook | `08_Configure_DHCP_DNS_Dynamic_Updates.md` | `<next-task>` |

# Configure_AD_Integrated_DNS_Zones_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Domain Controller / DNS Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm domain controller identity | Domain Controller | `hostname`; `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Server is domain joined and expected DC |
| 3 | Confirm DNS role installed | Domain Controller / DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 4 | Confirm DNS service running | Domain Controller / DNS Server | `Get-Service DNS` | DNS service is Running |
| 5 | Import DNS module | Domain Controller / DNS Server | `Import-Module DnsServer` | DNS module imports |
| 6 | Confirm AD module available | Domain Controller / Admin Host | `Import-Module ActiveDirectory` | AD module imports |
| 7 | Confirm domain health baseline | Domain Controller | `dcdiag /test:dns /v` | DNS test output is available |
| 8 | Confirm existing zones | DNS Server | `Get-DnsServerZone` | Existing forward, reverse, and AD zones are visible |
| 9 | Confirm domain forward zone exists | DNS Server | `Get-DnsServerZone -Name "<domain-fqdn>"` | AD domain zone exists |
| 10 | Create AD-integrated forward zone if missing | DNS Server | `Add-DnsServerPrimaryZone -Name "<zone-name>" -ReplicationScope Domain -DynamicUpdate Secure` | Forward zone exists and is AD-integrated |
| 11 | Confirm `_msdcs` zone exists | DNS Server | `Get-DnsServerZone -Name "_msdcs.<domain-fqdn>"` | Forest locator zone exists |
| 12 | Create reverse lookup zone if missing | DNS Server | `Add-DnsServerPrimaryZone -NetworkId "<network-id>" -ReplicationScope Domain -DynamicUpdate Secure` | Reverse lookup zone exists |
| 13 | Confirm dynamic update setting | DNS Server | `Get-DnsServerZone -Name "<zone-name>" \| Select-Object ZoneName,DynamicUpdate` | Dynamic update mode is Secure |
| 14 | Confirm replication scope | DNS Server | `Get-DnsServerZone -Name "<zone-name>" \| Select-Object ZoneName,ReplicationScope` | Replication scope matches design |
| 15 | Validate NS and SOA records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NS`; `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType SOA` | Authoritative records exist |
| 16 | Validate domain controller A records | DNS Server | `Resolve-DnsName "<dc-fqdn>" -Server "<dns-server-ip>"` | DC resolves to expected IP |
| 17 | Validate SRV records | DNS Server / Client | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain-fqdn> -Type SRV` | DC locator records resolve |
| 18 | Validate reverse lookup | DNS Server / Client | `Resolve-DnsName "<dc-ip>" -Type PTR` | PTR record resolves if configured |
| 19 | Force DC DNS registration if needed | Domain Controller | `ipconfig /registerdns`; `nltest /dsregdns` | DC records are refreshed |
| 20 | Validate client DNS registration | Client | `Register-DnsClient`; `Resolve-DnsName <client-fqdn>` | Client A record resolves |
| 21 | Validate AD replication | Domain Controller | `repadmin /replsummary` | AD replication is healthy |
| 22 | Review DNS event logs | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | Recent DNS events are visible |
| 23 | Export DNS evidence | DNS Server | Run validation skeleton | Zone and record evidence files are saved |
| 24 | Document final DNS state | Operator | `Record zones, replication scope, update mode, DNS servers, reverse zone, SRV validation, and event status` | DNS zone build record is complete |

# Configure_AD_Integrated_DNS_Zones_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on a domain controller or DNS server.
# Purpose: capture current DNS, AD, and DC locator state before modifying zones.

$DomainFqdn = "corp.local"
$MsdcsZone = "_msdcs.$DomainFqdn"
$DnsServer = "DC1"
$DnsServerIp = "10.10.10.10"
$ReverseNetworkId = "10.10.10.0/24"
$EvidencePath = "C:\DNS-AD-Integrated-Zones"

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

# Confirm role and service.
Get-WindowsFeature DNS |
  Tee-Object "$EvidencePath\dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-state.txt"

# Import modules.
Import-Module DnsServer
Import-Module ActiveDirectory

# Confirm domain and forest.
Get-ADDomain |
  Tee-Object "$EvidencePath\ad-domain.txt"

Get-ADForest |
  Tee-Object "$EvidencePath\ad-forest.txt"

# Capture existing DNS zones.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-before.txt"

# Capture specific expected zones if present.
Get-DnsServerZone -Name $DomainFqdn -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\domain-zone-before.txt"

Get-DnsServerZone -Name $MsdcsZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\msdcs-zone-before.txt"

# Capture DC DNS health.
dcdiag /test:dns /v |
  Tee-Object "$EvidencePath\dcdiag-dns-before.txt"

# Capture AD replication health.
repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary-before.txt"
```

# Configure_AD_Integrated_DNS_Zones_Forward_Zone_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: create or validate an AD-integrated forward lookup zone.

$ForwardZone = "corp.local"
$ReplicationScope = "Domain"
$EvidencePath = "C:\DNS-AD-Integrated-Zones"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\forward-zone-config-transcript.txt"

Import-Module DnsServer

# Check whether the forward zone already exists.
$ExistingZone = Get-DnsServerZone -Name $ForwardZone -ErrorAction SilentlyContinue

if ($ExistingZone) {
  Write-Output "Forward zone $ForwardZone already exists. No new zone created."
  $ExistingZone |
    Tee-Object "$EvidencePath\forward-zone-existing.txt"
}
else {
  # Create AD-integrated forward lookup zone.
  Add-DnsServerPrimaryZone `
    -Name $ForwardZone `
    -ReplicationScope $ReplicationScope `
    -DynamicUpdate Secure

  Get-DnsServerZone -Name $ForwardZone |
    Tee-Object "$EvidencePath\forward-zone-after-create.txt"
}

# Confirm zone properties.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsAutoCreated |
  Tee-Object "$EvidencePath\forward-zone-properties.txt"

# Confirm NS and SOA records.
Get-DnsServerResourceRecord -ZoneName $ForwardZone -RRType NS |
  Tee-Object "$EvidencePath\forward-zone-ns-records.txt"

Get-DnsServerResourceRecord -ZoneName $ForwardZone -RRType SOA |
  Tee-Object "$EvidencePath\forward-zone-soa-record.txt"

Stop-Transcript
```

# Configure_AD_Integrated_DNS_Zones_Reverse_Zone_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: create or validate an AD-integrated reverse lookup zone.

$ReverseNetworkId = "10.10.10.0/24"
$ExpectedReverseZone = "10.10.10.in-addr.arpa"
$ReplicationScope = "Domain"
$EvidencePath = "C:\DNS-AD-Integrated-Zones"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\reverse-zone-config-transcript.txt"

Import-Module DnsServer

# Check whether the reverse zone already exists.
$ExistingReverseZone = Get-DnsServerZone -Name $ExpectedReverseZone -ErrorAction SilentlyContinue

if ($ExistingReverseZone) {
  Write-Output "Reverse zone $ExpectedReverseZone already exists. No new reverse zone created."
  $ExistingReverseZone |
    Tee-Object "$EvidencePath\reverse-zone-existing.txt"
}
else {
  # Create AD-integrated reverse lookup zone from network ID.
  Add-DnsServerPrimaryZone `
    -NetworkId $ReverseNetworkId `
    -ReplicationScope $ReplicationScope `
    -DynamicUpdate Secure

  Get-DnsServerZone -Name $ExpectedReverseZone |
    Tee-Object "$EvidencePath\reverse-zone-after-create.txt"
}

# Confirm reverse zone properties.
Get-DnsServerZone -Name $ExpectedReverseZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsAutoCreated |
  Tee-Object "$EvidencePath\reverse-zone-properties.txt"

# Confirm NS and SOA records.
Get-DnsServerResourceRecord -ZoneName $ExpectedReverseZone -RRType NS |
  Tee-Object "$EvidencePath\reverse-zone-ns-records.txt"

Get-DnsServerResourceRecord -ZoneName $ExpectedReverseZone -RRType SOA |
  Tee-Object "$EvidencePath\reverse-zone-soa-record.txt"

Stop-Transcript
```

# Configure_AD_Integrated_DNS_Zones_Secure_Dynamic_Update_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: set AD-integrated zones to secure dynamic updates.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-AD-Integrated-Zones"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Capture current zone dynamic update settings.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,IsDsIntegrated,DynamicUpdate |
  Tee-Object "$EvidencePath\forward-zone-dynamic-update-before.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,IsDsIntegrated,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-dynamic-update-before.txt"

# Set forward zone to secure dynamic updates.
Set-DnsServerPrimaryZone `
  -Name $ForwardZone `
  -DynamicUpdate Secure

# Set reverse zone to secure dynamic updates if present.
Set-DnsServerPrimaryZone `
  -Name $ReverseZone `
  -DynamicUpdate Secure

# Confirm final dynamic update settings.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,IsDsIntegrated,DynamicUpdate |
  Tee-Object "$EvidencePath\forward-zone-dynamic-update-after.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,IsDsIntegrated,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-dynamic-update-after.txt"
```

# Configure_AD_Integrated_DNS_Zones_Replication_Scope_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: inspect AD-integrated DNS zone replication scope.
# Change replication scope only when you understand the AD topology and zone purpose.

$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-AD-Integrated-Zones"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer
Import-Module ActiveDirectory

# Capture zone replication scope.
Get-DnsServerZone -Name $ForwardZone |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\forward-zone-replication-scope.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate |
  Tee-Object "$EvidencePath\reverse-zone-replication-scope.txt"

# Capture DNS application partitions visible in AD.
Get-ADObject `
  -SearchBase (Get-ADRootDSE).ConfigurationNamingContext `
  -LDAPFilter "(objectClass=crossRef)" `
  -Properties dnsRoot,nCName |
  Where-Object {$_.dnsRoot -like "*DnsZones*"} |
  Select-Object Name,dnsRoot,nCName |
  Tee-Object "$EvidencePath\dns-application-partitions.txt"

# Capture replication health.
repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary-after-zone-config.txt"

# Optional examples for changing replication scope.
# Use only with change control.
# Set-DnsServerPrimaryZone -Name $ForwardZone -ReplicationScope Domain
# Set-DnsServerPrimaryZone -Name $ForwardZone -ReplicationScope Forest
```

# Configure_AD_Integrated_DNS_Zones_Record_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: validate AD DNS records required for domain operations.

$DomainFqdn = "corp.local"
$MsdcsZone = "_msdcs.$DomainFqdn"
$DnsServerIp = "10.10.10.10"
$Dc1Fqdn = "DC1.$DomainFqdn"
$Dc1Ip = "10.10.10.10"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-AD-Integrated-Zones"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Validate forward zone records.
Get-DnsServerResourceRecord -ZoneName $DomainFqdn |
  Tee-Object "$EvidencePath\forward-zone-all-records.txt"

# Validate NS and SOA.
Get-DnsServerResourceRecord -ZoneName $DomainFqdn -RRType NS |
  Tee-Object "$EvidencePath\forward-zone-ns-records-validation.txt"

Get-DnsServerResourceRecord -ZoneName $DomainFqdn -RRType SOA |
  Tee-Object "$EvidencePath\forward-zone-soa-validation.txt"

# Validate DC A record.
Resolve-DnsName $Dc1Fqdn -Server $DnsServerIp |
  Tee-Object "$EvidencePath\resolve-dc-a-record.txt"

# Validate AD SRV records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$EvidencePath\resolve-ldap-dc-msdcs-srv.txt"

Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$EvidencePath\resolve-kerberos-domain-srv.txt"

Resolve-DnsName "_ldap._tcp.gc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-global-catalog-srv.txt"

# Validate _msdcs zone if separate.
Get-DnsServerZone -Name $MsdcsZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\msdcs-zone-validation.txt"

# Validate reverse lookup if zone exists.
Resolve-DnsName $Dc1Ip -Type PTR -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-dc-ptr-record.txt"

# Run domain controller DNS diagnostic.
dcdiag /test:dns /v |
  Tee-Object "$EvidencePath\dcdiag-dns-after-zone-config.txt"
```

# Configure_AD_Integrated_DNS_Zones_Client_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a domain client.
# Purpose: validate client DNS behavior against AD-integrated DNS zones.

$DomainFqdn = "corp.local"
$DnsServerIp = "10.10.10.10"
$ClientEvidencePath = "C:\DNS-Client-Validation"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Capture client identity and DNS configuration.
hostname |
  Tee-Object "$ClientEvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$ClientEvidencePath\computer-domain-state.txt"

Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\dns-client-server-addresses.txt"

Get-DnsClientGlobalSetting |
  Tee-Object "$ClientEvidencePath\dns-client-global-settings.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all.txt"

# Clear DNS cache and force client registration.
Clear-DnsClientCache
Register-DnsClient

# Test domain resolution.
Resolve-DnsName $DomainFqdn -Server $DnsServerIp |
  Tee-Object "$ClientEvidencePath\resolve-domain-fqdn.txt"

# Test AD locator SRV records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$ClientEvidencePath\resolve-ldap-dc-msdcs-srv.txt"

Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$ClientEvidencePath\resolve-kerberos-srv.txt"

# Test DC locator.
nltest /dsgetdc:$DomainFqdn |
  Tee-Object "$ClientEvidencePath\nltest-dsgetdc.txt"

# Test current client FQDN resolution.
$ClientFqdn = "$env:COMPUTERNAME.$DomainFqdn"

Resolve-DnsName $ClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-client-fqdn.txt"

# Review DNS client events.
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\dns-client-operational-events.txt"
```

# Configure_AD_Integrated_DNS_Zones_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on a DNS server/domain controller.
# Purpose: collect DNS, Directory Service, and System events after zone configuration.

$EvidencePath = "C:\DNS-AD-Integrated-Zones"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server log.
Get-WinEvent -LogName "DNS Server" -MaxEvents 200 |
  Tee-Object "$EvidencePath\dns-server-events.txt"

# DNS Server analytical/operational logs may vary by OS and enabled state.
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events.txt"

# Directory Service events.
Get-WinEvent -LogName "Directory Service" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\directory-service-events.txt"

# System DNS-related events.
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*DNS*" -or
    $_.Message -like "*Netlogon*"
  } |
  Tee-Object "$EvidencePath\system-dns-netlogon-events.txt"

# Netlogon event review.
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object {$_.ProviderName -like "*Netlogon*"} |
  Tee-Object "$EvidencePath\netlogon-system-events.txt"

# Final service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-final.txt"

Get-Service Netlogon |
  Tee-Object "$EvidencePath\netlogon-service-final.txt"
```

# Configure_AD_Integrated_DNS_Zones_Verification_Commands
```powershell
# Confirm role and service.
Get-WindowsFeature DNS
Get-Service DNS

# Import modules.
Import-Module DnsServer
Import-Module ActiveDirectory

# Domain and forest baseline.
Get-ADDomain
Get-ADForest

# List DNS zones.
Get-DnsServerZone

# Confirm zone properties.
Get-DnsServerZone -Name "<domain-fqdn>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

Get-DnsServerZone -Name "_msdcs.<domain-fqdn>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

Get-DnsServerZone -Name "<reverse-zone-name>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Create AD-integrated forward zone if missing.
Add-DnsServerPrimaryZone `
  -Name "<zone-name>" `
  -ReplicationScope Domain `
  -DynamicUpdate Secure

# Create AD-integrated reverse zone if missing.
Add-DnsServerPrimaryZone `
  -NetworkId "<network-id-cidr>" `
  -ReplicationScope Domain `
  -DynamicUpdate Secure

# Set secure dynamic updates.
Set-DnsServerPrimaryZone -Name "<zone-name>" -DynamicUpdate Secure

# Validate NS and SOA.
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NS
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType SOA

# Validate A records.
Get-DnsServerResourceRecord -ZoneName "<domain-fqdn>" -RRType A
Resolve-DnsName "<dc-fqdn>" -Server "<dns-server-ip>"

# Validate SRV records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_kerberos._tcp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_ldap._tcp.gc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"

# Validate PTR records.
Resolve-DnsName "<ip-address>" -Type PTR -Server "<dns-server-ip>"

# Force DC DNS registration.
ipconfig /registerdns
nltest /dsregdns
Restart-Service Netlogon

# Validate domain controller DNS health.
dcdiag /test:dns /v

# Validate AD replication.
repadmin /replsummary
repadmin /showrepl

# Client validation.
ipconfig /all
Register-DnsClient
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"
nltest /dsgetdc:<domain-fqdn>

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Directory Service" -MaxEvents 100
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DNS*" -or $_.ProviderName -like "*Netlogon*"}
```

# Configure_AD_Integrated_DNS_Zones_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Test forward zone created incorrectly | `Remove-DnsServerZone -Name "<zone-name>" -Force` | Dangerous if zone is used by AD or clients |
| Test reverse zone created incorrectly | `Remove-DnsServerZone -Name "<reverse-zone-name>" -Force` | Removes PTR zone and all records inside it |
| Dynamic update mode changed incorrectly | `Set-DnsServerPrimaryZone -Name "<zone-name>" -DynamicUpdate Secure` | Wrong mode can block or allow unsafe updates |
| Replication scope changed incorrectly | Restore intended replication scope with `Set-DnsServerPrimaryZone` after change control | DNS data replication impact across DCs |
| Wrong A record created | `Remove-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType A -Name "<record-name>" -RecordData "<ip>" -Force` | Removing wrong A record can break name resolution |
| Wrong PTR record created | Remove incorrect PTR record from reverse zone | Reverse lookup may fail until corrected |
| DC DNS registration broken | Run `ipconfig /registerdns`, `nltest /dsregdns`, and restart Netlogon | DC locator records may remain missing until replicated |
| Evidence folder created | `Remove-Item C:\DNS-AD-Integrated-Zones -Recurse -Force` | Deletes validation evidence |
| DNS service restarted | No rollback normally needed | Brief DNS service disruption |
| Client DNS registration forced | No rollback normally needed | Client record may update or appear in DNS |

# Configure_AD_Integrated_DNS_Zones_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Domain clients cannot log on | DNS SRV or DC locator issue | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain>` | Rebuilding the domain |
| Domain join fails | Client DNS points wrong or SRV records missing | `ipconfig /all`; `nltest /dsgetdc:<domain>` | Resetting computer account first |
| `Get-DnsServerZone` does not show domain zone | DNS role or wrong server | Confirm DNS role and query correct DNS server | Creating random duplicate zones |
| Zone is not AD-integrated | Zone type or server role issue | `Get-DnsServerZone -Name <zone> \| Select IsDsIntegrated` | Editing records blindly |
| Dynamic updates fail | Zone update mode, permissions, or client registration issue | Zone DynamicUpdate setting and client event log | Reinstalling DNS |
| Client A record missing | Client did not register or DNS suffix wrong | `Register-DnsClient`; `ipconfig /all` | Manually creating every client record |
| PTR records missing | Reverse zone missing or DHCP/client not configured to update PTR | `Get-DnsServerZone`; reverse zone check | Changing forward zone |
| `_msdcs` records missing | Netlogon/DC registration issue | `nltest /dsregdns`; restart Netlogon; `dcdiag /test:dns` | Deleting DNS zones |
| SRV records missing | Netlogon registration or zone issue | `_ldap`, `_kerberos`, `_gc` SRV lookups | Recreating DHCP scope |
| `dcdiag /test:dns` fails | DNS zone, delegation, registration, or replication issue | Read failing section of dcdiag output | Restarting all DCs first |
| Reverse lookup fails | Reverse zone or PTR missing | `Resolve-DnsName <ip> -Type PTR` | Editing A records |
| Some DCs have records and others do not | AD replication issue | `repadmin /replsummary`; `repadmin /showrepl` | Recreating records on every DC manually |
| Client resolves external names but not AD names | Client using wrong DNS server | `ipconfig /all` DNS server list | Troubleshooting internet |
| DNS server starts but zone missing | AD replication/application partition issue | DNS event log and AD replication | Removing DNS role |
| Secure updates reject a device | Device not domain joined or lacks rights | Client domain membership and DNS event logs | Setting zone to nonsecure updates |
| DHCP clients do not update DNS | DHCP DNS dynamic update settings | DHCP server DNS settings and credentials | Blaming AD DNS zone immediately |
| DNS console shows stale records | Aging/scavenging or stale clients | Record timestamp and scavenging design | Deleting records without inventory |

# Configure_AD_Integrated_DNS_Zones_Related_Labs
| Lab                                                 | Related Workbook                                         | Skill Proven                                                      |
| --------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------- |
| Create new AD forest and root domain                | `04_Create_New_AD_Forest_And_Root_Domain.md`             | AD DS and DNS dependency baseline                                 |
| Validate AD-integrated DNS and SRV records          | `05_Validate_AD_Integrated_DNS_And_SRV_Records.md`       | DC locator and service record validation                          |
| Configure AD-integrated DNS zones                   | `07_Configure_AD_Integrated_DNS_Zones.md`                | Forward zone, reverse zone, secure updates, and replication scope |
| Verify SYSVOL, NETLOGON, DC locator, and FSMO roles | `09_Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | AD DNS health and locator validation                              |
| Troubleshoot failed domain join                     | `10_Troubleshoot_Failed_Domain_Join.md`                  | DNS-driven domain join troubleshooting                            |
| Configure DHCP DNS dynamic updates                  | `08_Configure_DHCP_DNS_Dynamic_Updates.md`               | DHCP-managed A/PTR record registration                            |
| Validate DHCP client lease assignment               | `06_Validate_DHCP_Client_Lease_Assignment.md`            | Client-side DNS option and registration validation                |