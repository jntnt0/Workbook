05_Configure_DHCPv4_Exclusions_And_Reservations.md
# Configure_DHCPv4_Exclusions_And_Reservations

# Configure_DHCPv4_Exclusions_And_Reservations_Index
05_Configure_DHCPv4_Exclusions_And_Reservations.md
Configure_DHCPv4_Exclusions_And_Reservations
Configure_DHCPv4_Exclusions_And_Reservations_Source_Basis
Configure_DHCPv4_Exclusions_And_Reservations_Mental_Model
Configure_DHCPv4_Exclusions_And_Reservations_Planning_Table
Configure_DHCPv4_Exclusions_And_Reservations_Configuration_Checklist
Configure_DHCPv4_Exclusions_And_Reservations_Precheck_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Exclusion_Range_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Reservation_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Client_Validation_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Lease_And_Conflict_Check_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Event_Log_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Verification_Commands
Configure_DHCPv4_Exclusions_And_Reservations_Rollback
Configure_DHCPv4_Exclusions_And_Reservations_Failure_Checks
Configure_DHCPv4_Exclusions_And_Reservations_Related_Labs

# Configure_DHCPv4_Exclusions_And_Reservations_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DHCP Server PowerShell module | DHCPv4 scope, exclusion, reservation, lease, and option management |
| Microsoft Learn | Add-DhcpServerv4ExclusionRange | Creating excluded address ranges inside a DHCP scope |
| Microsoft Learn | Get-DhcpServerv4ExclusionRange | Reviewing configured exclusion ranges |
| Microsoft Learn | Remove-DhcpServerv4ExclusionRange | Removing incorrect exclusion ranges |
| Microsoft Learn | Add-DhcpServerv4Reservation | Creating DHCP reservations for known MAC addresses |
| Microsoft Learn | Get-DhcpServerv4Reservation | Reviewing DHCP reservations |
| Microsoft Learn | Set-DhcpServerv4Reservation | Updating reservation properties |
| Microsoft Learn | Remove-DhcpServerv4Reservation | Removing incorrect reservations |
| Microsoft Learn | Get-DhcpServerv4Lease | Validating active leases and reservation lease state |
| Microsoft Learn | Get-DhcpServerv4Scope | Validating DHCPv4 scope state |
| Windows Server operational practice | Static infrastructure protection, printer/server reservations, client lease validation | Preventing address conflicts while still using DHCP-managed addressing |

# Configure_DHCPv4_Exclusions_And_Reservations_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP scope | Pool of IPv4 addresses available for DHCP assignment |
| Exclusion range | Address or range inside a scope that DHCP must never lease automatically |
| Reservation | Specific IP address mapped to a specific client MAC address |
| Client identifier | MAC address or DHCP client identifier used to bind a reservation |
| Lease | Active DHCP assignment given to a client |
| Static IP | Manually configured IP on a device, outside normal DHCP assignment |
| Infrastructure address | IP used by gateways, servers, switches, printers, APs, firewalls, or appliances |
| Conflict prevention | Main purpose of exclusions and reservations |
| Exclusion versus reservation | Exclusion blocks DHCP from leasing an address; reservation leases a specific address only to one known client |
| Reservation option inheritance | Reserved clients can inherit scope options unless reservation-specific options override them |
| Active lease conflict | Existing dynamic lease may need to expire, be released, or be removed before assigning a reservation |
| First rule | Exclude static infrastructure addresses first, then reserve known DHCP-managed devices that need stable addresses |

# Configure_DHCPv4_Exclusions_And_Reservations_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.5` | `<dhcp-server-ip>` |
| Domain | `corp.local` | `<domain-fqdn>` |
| Scope ID | `10.10.10.0` | `<scope-id>` |
| Scope name | `Corp-LAN` | `<scope-name>` |
| Scope range | `10.10.10.50-10.10.10.250` | `<scope-range>` |
| Subnet mask | `255.255.255.0` | `<subnet-mask>` |
| Default gateway | `10.10.10.1` | `<router-ip>` |
| DNS servers | `10.10.10.10,10.10.10.11` | `<dns-server-list>` |
| Exclusion range 1 | `10.10.10.50-10.10.10.79` | `<exclusion-range-1>` |
| Exclusion purpose | Static servers, printers, network devices | `<exclusion-purpose>` |
| Reserved client name | `PRN-FRONTDESK-01` | `<reservation-name>` |
| Reserved client MAC | `00-11-22-33-44-55` | `<client-mac>` |
| Reserved IP | `10.10.10.80` | `<reservation-ip>` |
| Reservation description | `Front desk printer` | `<reservation-description>` |
| Existing lease check required | Yes | `<yes-no>` |
| Conflict detection enabled | Optional | `<yes-no>` |
| Evidence path | `C:\DHCP-Exclusions-Reservations` | `<evidence-path>` |
| Rollback stance | Remove reservation or exclusion only after confirming impact | `<rollback-plan>` |
| Next workbook | `06_Validate_DHCP_Client_Lease_Assignment.md` | `<next-task>` |

# Configure_DHCPv4_Exclusions_And_Reservations_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DHCP role installed | DHCP Server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 3 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | DHCP service is Running |
| 4 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 5 | Confirm DHCP server authorization | DHCP Server | `Get-DhcpServerInDC` | Server is authorized in AD |
| 6 | Confirm target scope exists | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Scope is visible |
| 7 | Capture current exclusions | DHCP Server | `Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"` | Existing exclusions are documented |
| 8 | Capture current reservations | DHCP Server | `Get-DhcpServerv4Reservation -ScopeId "<scope-id>"` | Existing reservations are documented |
| 9 | Capture active leases | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Current leases are documented |
| 10 | Confirm reserved IP is inside scope | DHCP Server | Compare `<reservation-ip>` to scope range | Reservation IP is valid for the scope |
| 11 | Confirm excluded range is inside scope | DHCP Server | Compare exclusion start/end to scope range | Exclusion range is valid |
| 12 | Confirm reservation IP is not already leased to another client | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<reservation-ip>"` | IP is free or assigned to intended client |
| 13 | Add exclusion range | DHCP Server | `Add-DhcpServerv4ExclusionRange -ScopeId "<scope-id>" -StartRange "<start-ip>" -EndRange "<end-ip>"` | Exclusion range is created |
| 14 | Confirm exclusion range | DHCP Server | `Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"` | Exclusion appears in scope |
| 15 | Add reservation | DHCP Server | `Add-DhcpServerv4Reservation -ScopeId "<scope-id>" -IPAddress "<reservation-ip>" -ClientId "<mac>" -Name "<reservation-name>" -Description "<description>"` | Reservation is created |
| 16 | Confirm reservation | DHCP Server | `Get-DhcpServerv4Reservation -ScopeId "<scope-id>"` | Reservation appears in scope |
| 17 | Renew client lease | Client | `ipconfig /release`; `ipconfig /renew` | Client requests DHCP lease |
| 18 | Validate reserved client received reserved IP | Client | `ipconfig /all` | Client receives reserved IP |
| 19 | Validate DHCP server lease state | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>" -ClientId "<mac>"` | Lease maps to reservation |
| 20 | Validate DNS/gateway options still apply | Client | `ipconfig /all`; `Resolve-DnsName "<domain-fqdn>"` | Client receives correct DHCP options |
| 21 | Review DHCP events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100` | DHCP assignment events are visible |
| 22 | Export final evidence | DHCP Server | Run validation skeletons | Evidence files are saved |
| 23 | Document final state | Operator | `Record exclusions, reservations, client MACs, reserved IPs, and validation result` | Workbook record is complete |

# Configure_DHCPv4_Exclusions_And_Reservations_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: capture scope, lease, exclusion, and reservation state before changes.

$ScopeId = "10.10.10.0"
$ReservationIp = "10.10.10.80"
$ClientMac = "00-11-22-33-44-55"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Capture server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion |
  Tee-Object "$EvidencePath\computer-info.txt"

# Confirm DHCP role and service.
Get-WindowsFeature DHCP |
  Tee-Object "$EvidencePath\dhcp-role-state.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-state.txt"

# Import DHCP module.
Import-Module DhcpServer

# Confirm authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Capture target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-before.txt"

# Capture scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-options-before.txt"

# Capture current exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\exclusions-before.txt"

# Capture current reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservations-before.txt"

# Capture active leases.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\leases-before.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\leases-before.csv" -NoTypeInformation

# Check whether intended reservation IP is already leased.
Get-DhcpServerv4Lease -ScopeId $ScopeId -IPAddress $ReservationIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservation-ip-existing-lease-check.txt"

# Check whether intended client already has a lease.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ClientId $ClientMac -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservation-client-existing-lease-check.txt"
```

# Configure_DHCPv4_Exclusions_And_Reservations_Exclusion_Range_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: create exclusion ranges for addresses that DHCP must not lease automatically.

$ScopeId = "10.10.10.0"
$StartRange = "10.10.10.50"
$EndRange = "10.10.10.79"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\exclusion-range-config-transcript.txt"

Import-Module DhcpServer

# Capture scope before change.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-before-exclusion.txt"

# Capture existing exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\exclusions-before-add.txt"

# Add exclusion range.
Add-DhcpServerv4ExclusionRange `
  -ScopeId $ScopeId `
  -StartRange $StartRange `
  -EndRange $EndRange

# Confirm exclusion range.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\exclusions-after-add.txt"

# Confirm active leases do not conflict with excluded range.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\leases-after-exclusion-add.txt"

Stop-Transcript
```

# Configure_DHCPv4_Exclusions_And_Reservations_Reservation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: create a reservation for a known client MAC address.

$ScopeId = "10.10.10.0"
$ReservationIp = "10.10.10.80"
$ClientMac = "00-11-22-33-44-55"
$ReservationName = "PRN-FRONTDESK-01"
$ReservationDescription = "Front desk printer DHCP reservation"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\reservation-config-transcript.txt"

Import-Module DhcpServer

# Capture current reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservations-before-add.txt"

# Check whether reservation IP is already leased.
Get-DhcpServerv4Lease -ScopeId $ScopeId -IPAddress $ReservationIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservation-ip-lease-check-before-add.txt"

# Check whether client MAC already has a lease.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ClientId $ClientMac -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservation-client-lease-check-before-add.txt"

# Create reservation.
Add-DhcpServerv4Reservation `
  -ScopeId $ScopeId `
  -IPAddress $ReservationIp `
  -ClientId $ClientMac `
  -Name $ReservationName `
  -Description $ReservationDescription

# Confirm reservation.
Get-DhcpServerv4Reservation -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\reservations-after-add.txt"

Get-DhcpServerv4Reservation -ScopeId $ScopeId -ClientId $ClientMac |
  Tee-Object "$EvidencePath\specific-reservation-after-add.txt"

Stop-Transcript
```

# Configure_DHCPv4_Exclusions_And_Reservations_Client_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the reserved Windows client.
# Purpose: confirm the client receives the reserved IP and correct DHCP options.

$ExpectedReservationIp = "10.10.10.80"
$ExpectedDnsServer1 = "10.10.10.10"
$ExpectedDnsServer2 = "10.10.10.11"
$ExpectedGateway = "10.10.10.1"
$DomainFqdn = "corp.local"
$EvidencePath = "C:\DHCP-Reservation-Client-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture client identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$EvidencePath\computer-domain-state.txt"

# Capture adapter and IP configuration before renew.
Get-NetAdapter |
  Tee-Object "$EvidencePath\net-adapters-before.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-before.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-before.txt"

# Release and renew DHCP lease.
ipconfig /release |
  Tee-Object "$EvidencePath\ipconfig-release.txt"

Start-Sleep -Seconds 3

ipconfig /renew |
  Tee-Object "$EvidencePath\ipconfig-renew.txt"

# Capture final IP configuration.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-after.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-after.txt"

# Validate expected IP.
Get-NetIPAddress -AddressFamily IPv4 |
  Where-Object {$_.IPAddress -eq $ExpectedReservationIp} |
  Tee-Object "$EvidencePath\expected-reservation-ip-check.txt"

# Validate gateway reachability.
Test-NetConnection $ExpectedGateway |
  Tee-Object "$EvidencePath\test-default-gateway.txt"

# Validate DNS server reachability.
Test-NetConnection $ExpectedDnsServer1 -Port 53 |
  Tee-Object "$EvidencePath\test-dns-server-1-tcp-53.txt"

Test-NetConnection $ExpectedDnsServer2 -Port 53 |
  Tee-Object "$EvidencePath\test-dns-server-2-tcp-53.txt"

# Validate DNS resolution.
Resolve-DnsName $DomainFqdn -Server $ExpectedDnsServer1 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-domain-fqdn.txt"

# Capture DHCP Client events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-client-admin-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-client-operational-events.txt"
```

# Configure_DHCPv4_Exclusions_And_Reservations_Lease_And_Conflict_Check_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: verify reservation lease state and detect conflicts or wrong assignments.

$ScopeId = "10.10.10.0"
$ReservationIp = "10.10.10.80"
$ClientMac = "00-11-22-33-44-55"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm scope state.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-state-validation.txt"

# Confirm exclusion ranges.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\exclusions-validation.txt"

# Confirm reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\reservations-validation.txt"

# Confirm specific reservation.
Get-DhcpServerv4Reservation -ScopeId $ScopeId -ClientId $ClientMac |
  Tee-Object "$EvidencePath\specific-reservation-validation.txt"

# Confirm lease by IP.
Get-DhcpServerv4Lease -ScopeId $ScopeId -IPAddress $ReservationIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-by-reservation-ip.txt"

# Confirm lease by client MAC.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ClientId $ClientMac -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-by-client-mac.txt"

# Export active leases.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\leases-after-config.csv" -NoTypeInformation

# Scope utilization.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-statistics-after-config.txt"

# Optional conflict detection state.
Get-DhcpServerSetting |
  Tee-Object "$EvidencePath\dhcp-server-settings-conflict-detection.txt"
```

# Configure_DHCPv4_Exclusions_And_Reservations_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: collect DHCP Server events related to leases, reservations, scope changes, and conflicts.

$EvidencePath = "C:\DHCP-Exclusions-Reservations\Events"
$Since = (Get-Date).AddHours(-24)

New-Item -ItemType Directory -Force -Path $EvidencePath

# DHCP Server operational events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Dhcp-Server/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-operational-events-last-24h.txt"

# DHCP Server admin events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Dhcp-Server/Admin"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-admin-events-last-24h.txt"

# System events related to DHCP.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DHCP*" -or
    $_.Message -like "*DHCP*" -or
    $_.Message -like "*reservation*" -or
    $_.Message -like "*conflict*"
  } |
  Tee-Object "$EvidencePath\system-dhcp-events-last-24h.txt"

# Final DHCP service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"
```

# Configure_DHCPv4_Exclusions_And_Reservations_Verification_Commands
```powershell
# Confirm DHCP role and service.
Get-WindowsFeature DHCP
Get-Service DHCPServer

# Import DHCP module.
Import-Module DhcpServer

# Confirm DHCP authorization.
Get-DhcpServerInDC

# Confirm scope.
Get-DhcpServerv4Scope
Get-DhcpServerv4Scope -ScopeId "<scope-id>"

# Confirm scope options.
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"

# View exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"

# Add exclusion range.
Add-DhcpServerv4ExclusionRange `
  -ScopeId "<scope-id>" `
  -StartRange "<start-ip>" `
  -EndRange "<end-ip>"

# Remove exclusion range.
Remove-DhcpServerv4ExclusionRange `
  -ScopeId "<scope-id>" `
  -StartRange "<start-ip>" `
  -EndRange "<end-ip>"

# View reservations.
Get-DhcpServerv4Reservation -ScopeId "<scope-id>"

# Add reservation.
Add-DhcpServerv4Reservation `
  -ScopeId "<scope-id>" `
  -IPAddress "<reservation-ip>" `
  -ClientId "<client-mac>" `
  -Name "<reservation-name>" `
  -Description "<reservation-description>"

# Update reservation.
Set-DhcpServerv4Reservation `
  -ScopeId "<scope-id>" `
  -IPAddress "<reservation-ip>" `
  -Name "<new-reservation-name>" `
  -Description "<new-description>"

# Remove reservation.
Remove-DhcpServerv4Reservation `
  -ScopeId "<scope-id>" `
  -IPAddress "<reservation-ip>"

# Check leases.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<reservation-ip>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -ClientId "<client-mac>"

# Scope statistics.
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"

# Client renewal.
ipconfig /release
ipconfig /renew
ipconfig /all

# Client DNS and gateway validation.
Test-NetConnection "<gateway-ip>"
Test-NetConnection "<dns-server-ip>" -Port 53
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"

# DHCP logs.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
```

# Configure_DHCPv4_Exclusions_And_Reservations_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Exclusion range added incorrectly | `Remove-DhcpServerv4ExclusionRange -ScopeId "<scope-id>" -StartRange "<start-ip>" -EndRange "<end-ip>"` | DHCP may begin leasing those addresses if they remain inside active pool |
| Reservation added incorrectly | `Remove-DhcpServerv4Reservation -ScopeId "<scope-id>" -IPAddress "<reservation-ip>"` | Client may receive dynamic address instead of reserved address |
| Reservation mapped to wrong MAC | Remove reservation and recreate with correct ClientId | Wrong device may receive reserved IP |
| Reservation IP conflicts with static device | Remove reservation or choose different IP | IP conflict can disrupt both devices |
| Reservation description/name wrong | `Set-DhcpServerv4Reservation` with corrected metadata | Low technical risk |
| Client lease stuck on old IP | `ipconfig /release`; `ipconfig /renew`; optionally remove old lease | Temporary client network disruption |
| Active lease assigned to wrong client | Remove lease only after confirming impact | Client may lose network until renewed |
| Evidence folder created | `Remove-Item C:\DHCP-Exclusions-Reservations -Recurse -Force` | Deletes validation evidence |

# Configure_DHCPv4_Exclusions_And_Reservations_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client does not receive reserved IP | Wrong MAC/client ID, existing lease, wrong scope, or client not renewing | `Get-DhcpServerv4Reservation`; `Get-DhcpServerv4Lease`; client `ipconfig /all` | Rebuilding DHCP scope |
| Reservation created but lease still dynamic | Client has not renewed or lease still active | Release/renew client and check lease by ClientId | Deleting scope |
| Reserved IP already leased to another device | Existing active lease conflict | `Get-DhcpServerv4Lease -IPAddress <reservation-ip>` | Creating duplicate reservation |
| DHCP leases excluded address | Exclusion missing or wrong scope | `Get-DhcpServerv4ExclusionRange -ScopeId <scope-id>` | Restarting server first |
| Static device conflicts with DHCP client | Static IP not excluded or reservation overlaps | Exclusion ranges and active leases | Blaming DNS |
| Printer gets different IP | Wrong printer MAC, multiple NICs, or old lease | Printer MAC and reservation ClientId | Changing DNS records |
| Exclusion range blocks too much of scope | Exclusion too broad | Scope statistics and exclusion ranges | Expanding scope blindly |
| Scope runs out of addresses | Too many exclusions/reservations or high lease usage | `Get-DhcpServerv4ScopeStatistics` | Removing random reservations |
| Client shows APIPA | DHCP reachability or scope issue | DHCP service, relay, client VLAN, scope state | Editing reservation first |
| Client gets correct IP but wrong DNS/gateway | Scope options issue | `Get-DhcpServerv4OptionValue -ScopeId <scope-id>` | Editing reservation IP |
| Reservation in wrong scope | Scope ID mismatch | `Get-DhcpServerv4Reservation` across scopes | Recreating DHCP server |
| MAC format rejected | ClientId format issue | Use hyphenated MAC format like `00-11-22-33-44-55` | Changing scope range |
| Duplicate reservation exists | Same MAC or IP reserved elsewhere | Search all reservations | Disabling DHCP service |
| DHCP server not leasing at all | Authorization, service, scope state, or relay issue | `Get-DhcpServerInDC`; `Get-Service DHCPServer`; scope state | Fixing reservation only |
| Client renew fails across VLAN | Relay/IP helper issue | DHCP relay path and server events | Changing exclusion range |

# Configure_DHCPv4_Exclusions_And_Reservations_Related_Labs
| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Install DHCP Server role and management tools | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | DHCP role baseline |
| Authorize DHCP Server in Active Directory | `02_Authorize_DHCP_Server_In_Active_Directory.md` | AD-authorized DHCP service |
| Create DHCPv4 scope | `03_Create_DHCPv4_Scope.md` | DHCP address pool creation |
| Configure DHCPv4 scope options | `04_Configure_DHCPv4_Scope_Options.md` | Gateway, DNS, and domain option delivery |
| Configure DHCPv4 exclusions and reservations | `05_Configure_DHCPv4_Exclusions_And_Reservations.md` | Static address protection and stable DHCP-assigned devices |
| Validate DHCP client lease assignment | `06_Validate_DHCP_Client_Lease_Assignment.md` | Client-side lease verification |
| Configure DHCP relay and IP helper | `07_Configure_DHCP_Relay_And_IP_Helper.md` | Cross-subnet DHCP lease delivery |
| Troubleshoot DHCP client lease failures | `12_Troubleshoot_DHCP_Client_Lease_Failures.md` | Reservation, exclusion, lease, relay, and scope triage |
```