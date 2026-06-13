08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md
# Validate_AD_DNS_SRV_Records_And_DC_Locator

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Index
08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md
Validate_AD_DNS_SRV_Records_And_DC_Locator
Validate_AD_DNS_SRV_Records_And_DC_Locator_Source_Basis
Validate_AD_DNS_SRV_Records_And_DC_Locator_Mental_Model
Validate_AD_DNS_SRV_Records_And_DC_Locator_Planning_Table
Validate_AD_DNS_SRV_Records_And_DC_Locator_Configuration_Checklist
Validate_AD_DNS_SRV_Records_And_DC_Locator_Server_Precheck_Skeleton
Validate_AD_DNS_SRV_Records_And_DC_Locator_SRV_Record_Validation_Skeleton
Validate_AD_DNS_SRV_Records_And_DC_Locator_DC_Locator_Validation_Skeleton
Validate_AD_DNS_SRV_Records_And_DC_Locator_Netlogon_DNS_Registration_Skeleton
Validate_AD_DNS_SRV_Records_And_DC_Locator_Client_Validation_Skeleton
Validate_AD_DNS_SRV_Records_And_DC_Locator_Replication_Validation_Skeleton
Validate_AD_DNS_SRV_Records_And_DC_Locator_Event_Log_Skeleton
Validate_AD_DNS_SRV_Records_And_DC_Locator_Verification_Commands
Validate_AD_DNS_SRV_Records_And_DC_Locator_Rollback
Validate_AD_DNS_SRV_Records_And_DC_Locator_Failure_Checks
Validate_AD_DNS_SRV_Records_And_DC_Locator_Related_Labs

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | Querying AD-integrated DNS zones and resource records |
| Microsoft Learn | Resolve-DnsName | Validating A, PTR, CNAME, and SRV record resolution |
| Microsoft Learn | Get-DnsServerResourceRecord | Inspecting DNS records inside AD-integrated zones |
| Microsoft Learn | Register-DnsClient | Forcing client DNS registration |
| Microsoft Learn | nltest | Validating domain controller discovery and DC locator behavior |
| Microsoft Learn | dcdiag /test:dns | Validating domain controller DNS health |
| Microsoft Learn | repadmin | Validating AD replication health for AD-integrated DNS data |
| Microsoft Learn | ipconfig /registerdns | Forcing host DNS record registration |
| Microsoft Learn | Netlogon service | Registering domain controller locator records |
| Windows Server operational practice | AD DNS, SRV records, _msdcs zone, DC locator, client DNS configuration | Proving that Active Directory DNS can locate domain controllers correctly |

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Mental_Model
| Concept | Operational Meaning |
|---|---|
| AD-integrated DNS | DNS zone data stored in Active Directory and replicated between domain controllers |
| SRV record | Service locator record that tells clients where domain services exist |
| DC locator | Client process used to discover a domain controller for logon, domain join, LDAP, Kerberos, and GPO |
| `_msdcs` zone | Forest-wide DNS zone used for domain controller locator records |
| `_ldap._tcp.dc._msdcs` | SRV query used to locate domain controllers for the domain |
| `_kerberos._tcp` | SRV query used to locate Kerberos services |
| `_ldap._tcp.gc._msdcs` | SRV query used to locate Global Catalog servers |
| A record | Hostname-to-IP mapping for domain controllers and clients |
| PTR record | IP-to-hostname mapping used for reverse lookup validation |
| Netlogon DNS registration | Process where domain controllers register SRV records in DNS |
| Client DNS setting | Domain clients must use AD DNS servers, not public DNS servers |
| Replication dependency | AD-integrated DNS records must replicate between domain controllers |
| Site-aware DC locator | Clients can locate domain controllers based on AD Sites and Services subnet mapping |
| First rule | If AD DNS SRV records or client DNS settings are wrong, domain joins, logons, GPO, Kerberos, LDAP, and replication troubleshooting all become unreliable |

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Primary domain controller | `DC1.corp.local` | `<dc1-fqdn>` |
| Secondary domain controller | `DC2.corp.local` | `<dc2-fqdn>` |
| Primary DNS server IP | `10.10.10.10` | `<dns-server-1>` |
| Secondary DNS server IP | `10.10.10.11` | `<dns-server-2>` |
| Forward lookup zone | `corp.local` | `<forward-zone-name>` |
| Forest locator zone | `_msdcs.corp.local` | `<msdcs-zone-name>` |
| Expected DC locator SRV | `_ldap._tcp.dc._msdcs.corp.local` | `<dc-locator-srv>` |
| Expected Kerberos SRV | `_kerberos._tcp.corp.local` | `<kerberos-srv>` |
| Expected GC SRV | `_ldap._tcp.gc._msdcs.corp.local` | `<gc-srv>` |
| Expected site name | `Default-First-Site-Name` | `<ad-site-name>` |
| Site-specific LDAP SRV | `_ldap._tcp.<site>._sites.dc._msdcs.corp.local` | `<site-ldap-srv>` |
| Reverse lookup zone | `10.10.10.in-addr.arpa` | `<reverse-zone-name>` |
| Test client | `WIN11-01.corp.local` | `<client-fqdn>` |
| Test client subnet | `10.10.10.0/24` | `<client-subnet>` |
| Expected client DNS suffix | `corp.local` | `<dns-suffix>` |
| Evidence path on DC | `C:\AD-DNS-DC-Locator-Validation` | `<dc-evidence-path>` |
| Evidence path on client | `C:\AD-DNS-Client-Validation` | `<client-evidence-path>` |
| Rollback stance | Validation-focused; rollback only forced registration or temporary records | `<rollback-plan>` |
| Next workbook | `09_Configure_DHCP_DNS_Dynamic_Updates.md` | `<next-task>` |

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Domain Controller | `whoami /groups` | Admin context is visible |
| 2 | Confirm DC identity | Domain Controller | `hostname`; `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Server identity and domain state are confirmed |
| 3 | Confirm AD DS role | Domain Controller | `Get-WindowsFeature AD-Domain-Services` | AD DS role is installed |
| 4 | Confirm DNS role | Domain Controller / DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 5 | Confirm Netlogon service | Domain Controller | `Get-Service Netlogon` | Netlogon service is Running |
| 6 | Confirm DNS service | DNS Server | `Get-Service DNS` | DNS service is Running |
| 7 | Import modules | Domain Controller | `Import-Module ActiveDirectory`; `Import-Module DnsServer` | AD and DNS modules import |
| 8 | Confirm domain and forest | Domain Controller | `Get-ADDomain`; `Get-ADForest` | Domain and forest metadata are visible |
| 9 | Confirm DNS zones | DNS Server | `Get-DnsServerZone` | Forward and `_msdcs` zones are visible |
| 10 | Confirm AD-integrated forward zone | DNS Server | `Get-DnsServerZone -Name "<domain-fqdn>"` | Domain forward zone exists |
| 11 | Confirm `_msdcs` zone | DNS Server | `Get-DnsServerZone -Name "_msdcs.<domain-fqdn>"` | Forest locator zone exists |
| 12 | Validate DC A records | DNS Server / Client | `Resolve-DnsName "<dc-fqdn>" -Server "<dns-server-ip>"` | DC resolves to expected IP |
| 13 | Validate LDAP DC SRV record | DNS Server / Client | `Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"` | Domain controller SRV records resolve |
| 14 | Validate Kerberos SRV record | DNS Server / Client | `Resolve-DnsName "_kerberos._tcp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"` | Kerberos SRV records resolve |
| 15 | Validate Global Catalog SRV record | DNS Server / Client | `Resolve-DnsName "_ldap._tcp.gc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"` | GC SRV records resolve if GC exists |
| 16 | Validate site-specific SRV records | DNS Server / Client | `Resolve-DnsName "_ldap._tcp.<site>._sites.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"` | Site-aware DC locator records resolve |
| 17 | Validate DC locator from DC | Domain Controller | `nltest /dsgetdc:<domain-fqdn>` | A domain controller is returned |
| 18 | Validate DC locator from client | Domain Client | `nltest /dsgetdc:<domain-fqdn>` | Client can locate a DC |
| 19 | Validate client DNS server settings | Domain Client | `ipconfig /all`; `Get-DnsClientServerAddress -AddressFamily IPv4` | Client uses AD DNS servers |
| 20 | Validate domain join DNS behavior | Domain Client | `Resolve-DnsName "<domain-fqdn>"`; `Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV` | Client resolves domain and SRV records |
| 21 | Run DNS diagnostic | Domain Controller | `dcdiag /test:dns /v` | DNS diagnostic output is clean or actionable |
| 22 | Validate AD replication | Domain Controller | `repadmin /replsummary`; `repadmin /showrepl` | Replication is healthy |
| 23 | Force DC DNS registration if records are missing | Domain Controller | `ipconfig /registerdns`; `nltest /dsregdns`; `Restart-Service Netlogon` | DC attempts to re-register DNS records |
| 24 | Review DNS and Netlogon events | Domain Controller | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100`; System Netlogon events | Event evidence is collected |
| 25 | Export validation evidence | Domain Controller / Client | Run validation skeletons | Evidence files are saved |
| 26 | Document final state | Operator | `Record DNS zones, SRV records, DC locator result, client DNS settings, dcdiag, repadmin, and event log status` | Validation record is complete |

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Server_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on a domain controller.
# Purpose: capture baseline AD, DNS, service, and zone state before SRV/DC locator validation.

$DomainFqdn = "corp.local"
$MsdcsZone = "_msdcs.$DomainFqdn"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\AD-DNS-DC-Locator-Validation"

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

# Confirm roles.
Get-WindowsFeature AD-Domain-Services,DNS |
  Tee-Object "$EvidencePath\windows-feature-state.txt"

# Confirm services.
Get-Service DNS,Netlogon,KDC,NTDS |
  Tee-Object "$EvidencePath\core-ad-dns-services.txt"

# Import modules.
Import-Module ActiveDirectory
Import-Module DnsServer

# Capture domain and forest state.
Get-ADDomain |
  Tee-Object "$EvidencePath\ad-domain.txt"

Get-ADForest |
  Tee-Object "$EvidencePath\ad-forest.txt"

# Capture domain controllers.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog,OperatingSystem |
  Tee-Object "$EvidencePath\domain-controllers.txt"

# Capture DNS zones.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones.txt"

# Capture expected AD zones.
Get-DnsServerZone -Name $DomainFqdn -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\domain-forward-zone.txt"

Get-DnsServerZone -Name $MsdcsZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\msdcs-zone.txt"

# Capture current resolver config on DC.
Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dc-dns-client-server-addresses.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\dc-ipconfig-all.txt"
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_SRV_Record_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a domain controller or admin host.
# Purpose: validate AD DNS SRV records used by domain clients and services.

$DomainFqdn = "corp.local"
$DnsServerIp = "10.10.10.10"
$SiteName = "Default-First-Site-Name"
$EvidencePath = "C:\AD-DNS-DC-Locator-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# LDAP domain controller locator records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$EvidencePath\srv-ldap-dc-msdcs.txt"

# Kerberos records.
Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$EvidencePath\srv-kerberos-domain.txt"

Resolve-DnsName "_kerberos._udp.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\srv-kerberos-udp-domain.txt"

# LDAP records under domain.
Resolve-DnsName "_ldap._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$EvidencePath\srv-ldap-domain.txt"

# Global Catalog records.
Resolve-DnsName "_ldap._tcp.gc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\srv-global-catalog.txt"

# Kpasswd records if present.
Resolve-DnsName "_kpasswd._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\srv-kpasswd-tcp.txt"

Resolve-DnsName "_kpasswd._udp.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\srv-kpasswd-udp.txt"

# Site-aware DC locator records.
Resolve-DnsName "_ldap._tcp.$SiteName._sites.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\srv-site-ldap-dc-msdcs.txt"

Resolve-DnsName "_kerberos._tcp.$SiteName._sites.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\srv-site-kerberos.txt"

# Export raw records from DNS zones for comparison.
Import-Module DnsServer

Get-DnsServerResourceRecord -ZoneName $DomainFqdn -RRType SRV -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\domain-zone-srv-records.txt"

Get-DnsServerResourceRecord -ZoneName "_msdcs.$DomainFqdn" -RRType SRV -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\msdcs-zone-srv-records.txt"
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_DC_Locator_Validation_Skeleton
```powershell
# Run on a domain controller and then on a domain client.
# Purpose: prove DC locator returns a usable domain controller.

$DomainFqdn = "corp.local"
$SiteName = "Default-First-Site-Name"
$EvidencePath = "C:\AD-DNS-DC-Locator-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Basic DC locator.
nltest /dsgetdc:$DomainFqdn |
  Tee-Object "$EvidencePath\nltest-dsgetdc-domain.txt"

# Force rediscovery instead of cached result.
nltest /dsgetdc:$DomainFqdn /force |
  Tee-Object "$EvidencePath\nltest-dsgetdc-domain-force.txt"

# Try site-specific locator.
nltest /dsgetdc:$DomainFqdn /site:$SiteName |
  Tee-Object "$EvidencePath\nltest-dsgetdc-site.txt"

# Locate a writable DC.
nltest /dsgetdc:$DomainFqdn /writable |
  Tee-Object "$EvidencePath\nltest-dsgetdc-writable.txt"

# Locate a Global Catalog.
nltest /dsgetdc:$DomainFqdn /gc |
  Tee-Object "$EvidencePath\nltest-dsgetdc-global-catalog.txt"

# Locate a KDC.
nltest /dsgetdc:$DomainFqdn /kdc |
  Tee-Object "$EvidencePath\nltest-dsgetdc-kdc.txt"

# Domain trust/domain controller status.
nltest /dclist:$DomainFqdn |
  Tee-Object "$EvidencePath\nltest-dclist.txt"

# Secure channel validation if run from a domain client.
nltest /sc_verify:$DomainFqdn |
  Tee-Object "$EvidencePath\nltest-secure-channel-verify.txt"
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Netlogon_DNS_Registration_Skeleton
```powershell
# Run in elevated PowerShell on a domain controller.
# Purpose: refresh DC DNS registrations if SRV or A records are missing.
# Use only after confirming DNS zones and service state.

$DomainFqdn = "corp.local"
$DnsServerIp = "10.10.10.10"
$DcFqdn = "DC1.$DomainFqdn"
$EvidencePath = "C:\AD-DNS-DC-Locator-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture state before registration.
Get-Service Netlogon,DNS,KDC,NTDS |
  Tee-Object "$EvidencePath\services-before-dns-registration-refresh.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\dc-ipconfig-before-registerdns.txt"

# Force host DNS registration.
ipconfig /registerdns |
  Tee-Object "$EvidencePath\ipconfig-registerdns.txt"

# Force domain controller SRV record registration.
nltest /dsregdns |
  Tee-Object "$EvidencePath\nltest-dsregdns.txt"

# Restart Netlogon to re-register DC locator records.
Restart-Service Netlogon

Start-Sleep -Seconds 10

# Confirm services after restart.
Get-Service Netlogon,DNS,KDC,NTDS |
  Tee-Object "$EvidencePath\services-after-netlogon-restart.txt"

# Validate records after registration refresh.
Resolve-DnsName $DcFqdn -Server $DnsServerIp |
  Tee-Object "$EvidencePath\resolve-dc-a-record-after-refresh.txt"

Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$EvidencePath\srv-ldap-dc-msdcs-after-refresh.txt"

Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$EvidencePath\srv-kerberos-after-refresh.txt"

# Run DNS diagnostic after refresh.
dcdiag /test:dns /v |
  Tee-Object "$EvidencePath\dcdiag-dns-after-registration-refresh.txt"
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Client_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a domain client.
# Purpose: prove the client uses AD DNS and can locate a domain controller.

$DomainFqdn = "corp.local"
$DnsServerIp = "10.10.10.10"
$ClientEvidencePath = "C:\AD-DNS-Client-Validation"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Capture client identity.
hostname |
  Tee-Object "$ClientEvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$ClientEvidencePath\computer-domain-state.txt"

# Capture network and DNS client configuration.
Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\dns-client-server-addresses.txt"

Get-DnsClientGlobalSetting |
  Tee-Object "$ClientEvidencePath\dns-client-global-settings.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all.txt"

# Clear DNS cache.
Clear-DnsClientCache

# Validate domain FQDN.
Resolve-DnsName $DomainFqdn -Server $DnsServerIp |
  Tee-Object "$ClientEvidencePath\resolve-domain-fqdn.txt"

# Validate AD SRV records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$ClientEvidencePath\resolve-srv-ldap-dc-msdcs.txt"

Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV -Server $DnsServerIp |
  Tee-Object "$ClientEvidencePath\resolve-srv-kerberos.txt"

Resolve-DnsName "_ldap._tcp.gc._msdcs.$DomainFqdn" -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-srv-global-catalog.txt"

# Validate DC locator.
nltest /dsgetdc:$DomainFqdn |
  Tee-Object "$ClientEvidencePath\nltest-dsgetdc.txt"

nltest /dsgetdc:$DomainFqdn /force |
  Tee-Object "$ClientEvidencePath\nltest-dsgetdc-force.txt"

# Validate secure channel if domain joined.
nltest /sc_verify:$DomainFqdn |
  Tee-Object "$ClientEvidencePath\nltest-secure-channel-verify.txt"

# Force client DNS registration.
Register-DnsClient

# Validate client FQDN registration.
$ClientFqdn = "$env:COMPUTERNAME.$DomainFqdn"

Resolve-DnsName $ClientFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-client-fqdn.txt"
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Replication_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a domain controller.
# Purpose: validate that AD and AD-integrated DNS data can replicate between domain controllers.

$DomainFqdn = "corp.local"
$EvidencePath = "C:\AD-DNS-DC-Locator-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Validate replication summary.
repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary.txt"

# Show inbound replication details.
repadmin /showrepl |
  Tee-Object "$EvidencePath\repadmin-showrepl.txt"

# Show replication queue.
repadmin /queue |
  Tee-Object "$EvidencePath\repadmin-queue.txt"

# Show domain controllers from AD.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog,OperationMasterRoles |
  Tee-Object "$EvidencePath\ad-domain-controllers.txt"

# Test DNS records against each known DNS server/DC.
$DomainControllers = Get-ADDomainController -Filter *

foreach ($DC in $DomainControllers) {
  $Server = $DC.HostName

  Resolve-DnsName "_ldap._tcp.dc._msdcs.$DomainFqdn" -Type SRV -Server $Server -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\srv-ldap-dc-msdcs-against-$($DC.Name).txt"

  Resolve-DnsName "_kerberos._tcp.$DomainFqdn" -Type SRV -Server $Server -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\srv-kerberos-against-$($DC.Name).txt"
}

# Run DNS diagnostic.
dcdiag /test:dns /v |
  Tee-Object "$EvidencePath\dcdiag-dns-replication-validation.txt"
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on domain controllers and clients as applicable.
# Purpose: collect DNS, Netlogon, Directory Service, and client event evidence.

$EvidencePath = "C:\AD-DNS-DC-Locator-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events.txt"

# DNS Server operational events.
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events.txt"

# Directory Service events.
Get-WinEvent -LogName "Directory Service" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\directory-service-events.txt"

# System DNS and Netlogon related events.
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.ProviderName -like "*Netlogon*" -or
    $_.Message -like "*DNS*" -or
    $_.Message -like "*Netlogon*"
  } |
  Tee-Object "$EvidencePath\system-dns-netlogon-events.txt"

# KDC events.
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*KDC*" -or
    $_.Message -like "*Kerberos*"
  } |
  Tee-Object "$EvidencePath\system-kdc-kerberos-events.txt"

# Client DNS operational events if run on client.
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-client-operational-events.txt"

# Final service state.
Get-Service DNS,Netlogon,KDC,NTDS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\final-service-state.txt"
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Verification_Commands
```powershell
# Confirm roles and services.
Get-WindowsFeature AD-Domain-Services,DNS
Get-Service DNS,Netlogon,KDC,NTDS

# Import modules.
Import-Module ActiveDirectory
Import-Module DnsServer

# Confirm domain and forest.
Get-ADDomain
Get-ADForest
Get-ADDomainController -Filter *

# Confirm DNS zones.
Get-DnsServerZone
Get-DnsServerZone -Name "<domain-fqdn>"
Get-DnsServerZone -Name "_msdcs.<domain-fqdn>"

# Confirm zone properties.
Get-DnsServerZone -Name "<domain-fqdn>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Validate DC A records.
Resolve-DnsName "<dc-fqdn>" -Server "<dns-server-ip>"

# Validate domain root resolution.
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"

# Validate SRV records.
Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_ldap._tcp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_kerberos._tcp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_kerberos._udp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_ldap._tcp.gc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_kpasswd._tcp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_kpasswd._udp.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"

# Validate site-aware SRV records.
Resolve-DnsName "_ldap._tcp.<site-name>._sites.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "_kerberos._tcp.<site-name>._sites.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"

# Inspect raw SRV records from zone.
Get-DnsServerResourceRecord -ZoneName "<domain-fqdn>" -RRType SRV
Get-DnsServerResourceRecord -ZoneName "_msdcs.<domain-fqdn>" -RRType SRV

# Validate DC locator.
nltest /dsgetdc:<domain-fqdn>
nltest /dsgetdc:<domain-fqdn> /force
nltest /dsgetdc:<domain-fqdn> /writable
nltest /dsgetdc:<domain-fqdn> /gc
nltest /dsgetdc:<domain-fqdn> /kdc
nltest /dclist:<domain-fqdn>

# Validate client secure channel if domain joined.
nltest /sc_verify:<domain-fqdn>

# Force DC DNS registration.
ipconfig /registerdns
nltest /dsregdns
Restart-Service Netlogon

# DNS diagnostic.
dcdiag /test:dns /v

# AD replication validation.
repadmin /replsummary
repadmin /showrepl
repadmin /queue

# Client DNS validation.
ipconfig /all
Get-DnsClientServerAddress -AddressFamily IPv4
Clear-DnsClientCache
Register-DnsClient
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"
Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
nltest /dsgetdc:<domain-fqdn>

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Directory Service" -MaxEvents 100
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DNS*" -or $_.ProviderName -like "*Netlogon*" -or $_.ProviderName -like "*KDC*"}
Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
```

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| DNS validation only | No rollback required | No configuration changed |
| DNS cache cleared | No rollback required | Cache rebuilds during normal DNS use |
| Client DNS registration forced | No rollback usually required | Client A/PTR records may update |
| DC DNS registration forced | No rollback usually required | DC SRV/A records may refresh |
| Netlogon restarted | No rollback usually required | Brief DC locator registration interruption |
| Temporary test record created | Remove with `Remove-DnsServerResourceRecord` | Removing wrong record can break resolution |
| Incorrect DNS client server changed during testing | Restore previous DNS server list with `Set-DnsClientServerAddress` | Wrong DNS servers break domain lookup |
| Missing SRV records manually altered | Prefer Netlogon re-registration over manual edits | Manual mistakes can break DC locator |
| Evidence folder created | `Remove-Item C:\AD-DNS-DC-Locator-Validation -Recurse -Force` | Deletes validation evidence |

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| `nltest /dsgetdc` fails | DC locator, DNS SRV, or client DNS issue | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain> -Type SRV` | Rebuilding the domain |
| Domain join fails | Client DNS points to wrong resolver or SRV records missing | `ipconfig /all`; client DNS server list | Resetting the client first |
| Client uses public DNS | Client DHCP/static DNS misconfiguration | `Get-DnsClientServerAddress -AddressFamily IPv4` | Troubleshooting AD replication |
| `_ldap._tcp.dc._msdcs` does not resolve | Missing DC locator SRV records | Netlogon service, `_msdcs` zone, `nltest /dsregdns` | Manually creating random SRV records |
| `_kerberos._tcp` does not resolve | Missing Kerberos SRV record | Netlogon registration and DNS zones | Reinstalling KDC |
| GC SRV records missing | No Global Catalog or registration issue | `Get-ADDomainController -Filter *`; check IsGlobalCatalog | Assuming DNS is fully broken |
| DC A record missing | DC host registration issue | `ipconfig /registerdns`; DNS client settings on DC | Editing DHCP scope |
| Site-specific SRV missing | AD Sites and Services subnet/site mapping issue | AD site name and subnet mapping | Changing normal domain SRV records |
| Some DNS servers answer correctly and others do not | AD-integrated DNS replication issue | `repadmin /replsummary`; query each DC directly | Recreating zones |
| `dcdiag /test:dns` fails | DNS registration, delegation, or replication issue | Read specific failing dcdiag section | Restarting all DCs first |
| Netlogon errors in System log | DC cannot register DNS records | DNS zone availability and secure update permissions | Deleting `_msdcs` zone |
| Client resolves domain but not DC locator SRV | Partial DNS record issue | SRV queries directly | Changing gateway |
| Client finds remote/wrong-site DC | Sites/subnets issue | `nltest /dsgetdc:<domain> /force`; AD subnet mapping | Editing SRV records directly |
| PTR lookup fails | Reverse zone or PTR record missing | `Resolve-DnsName <ip> -Type PTR` | Troubleshooting Kerberos first |
| Secure channel fails | Client trust or DC locator issue | `nltest /sc_verify:<domain>` and DNS SRV checks | Rejoining immediately before DNS validation |
| GPO fails after lease works | DNS/DC locator issue, not DHCP lease itself | `gpresult /r`; `nltest /dsgetdc:<domain>` | Editing DHCP exclusions |
| LDAP/Kerberos ports unreachable | Firewall or DC service issue | `Test-NetConnection <dc-ip> -Port 389`; `Test-NetConnection <dc-ip> -Port 88` | Recreating DNS records |
| External DNS works but AD DNS fails | Wrong resolver or missing AD records | Check client DNS server and SRV records | Troubleshooting internet first |

# Validate_AD_DNS_SRV_Records_And_DC_Locator_Related_Labs
| Lab                                                 | Related Workbook                                         | Skill Proven                                                         |
| --------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------- |
| Create new AD forest and root domain                | `04_Create_New_AD_Forest_And_Root_Domain.md`             | AD DNS creation baseline                                             |
| Validate AD-integrated DNS and SRV records          | `05_Validate_AD_Integrated_DNS_And_SRV_Records.md`       | AD DNS and service locator validation                                |
| Configure AD-integrated DNS zones                   | `07_Configure_AD_Integrated_DNS_Zones.md`                | Forward zone, `_msdcs`, reverse zone, and secure dynamic updates     |
| Validate AD DNS SRV records and DC locator          | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md`       | DC locator, SRV records, Netlogon registration, and client discovery |
| Verify SYSVOL, NETLOGON, DC locator, and FSMO roles | `09_Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Full post-DC AD health validation                                    |
| Troubleshoot failed domain join                     | `10_Troubleshoot_Failed_Domain_Join.md`                  | Client DNS and DC locator troubleshooting                            |
| Configure DHCP DNS dynamic updates                  | `09_Configure_DHCP_DNS_Dynamic_Updates.md`               | DHCP-driven A/PTR record registration                                |
| Validate DHCP client lease assignment               | `06_Validate_DHCP_Client_Lease_Assignment.md`            | Client DNS option and resolver validation                            |