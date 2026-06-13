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
Configure_DHCPv4_Exclusions_And_Reservations_Activation_Decision_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Client_Validation_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Event_Log_Skeleton
Configure_DHCPv4_Exclusions_And_Reservations_Verification_Commands
Configure_DHCPv4_Exclusions_And_Reservations_Rollback
Configure_DHCPv4_Exclusions_And_Reservations_Failure_Checks
Configure_DHCPv4_Exclusions_And_Reservations_Related_Labs

# Configure_DHCPv4_Exclusions_And_Reservations_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DhcpServerv4ExclusionRange | Creating exclusion ranges inside an existing DHCPv4 scope |
| Microsoft Learn | Get-DhcpServerv4ExclusionRange | Verifying DHCPv4 exclusion ranges |
| Microsoft Learn | Remove-DhcpServerv4ExclusionRange | Removing incorrect DHCPv4 exclusion ranges |
| Microsoft Learn | Add-DhcpServerv4Reservation | Creating DHCPv4 reservations for fixed client leases |
| Microsoft Learn | Get-DhcpServerv4Reservation | Verifying DHCPv4 reservations |
| Microsoft Learn | Remove-DhcpServerv4Reservation | Removing incorrect DHCPv4 reservations |
| Microsoft Learn | Get-DhcpServerv4Lease | Checking active leases before and after reservations |
| Microsoft Learn | Get-DhcpServerv4Scope | Confirming target scope identity and state |
| Microsoft Learn | Set-DhcpServerv4Scope | Activating or deactivating a scope after exclusions and reservations are ready |
| Microsoft Learn | DHCP Server event logs | Reviewing reservation, scope, service, and lease assignment events |
| Windows Server operational practice | Static IP protection, printer reservations, server reservations, and lease conflict prevention | Preventing DHCP from leasing addresses that should remain fixed or controlled |

# Configure_DHCPv4_Exclusions_And_Reservations_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Exclusion range | IP range inside a DHCP scope that the server will not lease to clients |
| Reservation | Fixed IP assignment tied to a client identifier, usually a MAC address |
| Scope range | Total address range available to a scope before exclusions and reservations are applied |
| Static infrastructure range | Addresses commonly used by gateways, servers, printers, switches, APs, hypervisors, or management interfaces |
| Client ID | DHCP identifier used to match a reservation to a client, commonly the MAC address without separators |
| Reservation name | Human-readable label for the reserved client |
| Reservation description | Documentation field explaining what the reservation belongs to |
| Lease conflict | Condition where DHCP leases an address that is already statically assigned |
| Exclusion versus reservation | Exclusion prevents leasing; reservation leases a specific address only to a specific client |
| Existing lease impact | A reservation may not apply cleanly until an old lease is released, removed, or renewed |
| Scope activation dependency | Scope should stay inactive until required exclusions, reservations, and options are configured |
| First rule | Exclude infrastructure and static ranges before activating the scope, then reserve only the clients that need predictable DHCP-managed addresses |

# Configure_DHCPv4_Exclusions_And_Reservations_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server name | `DHCP1` | `<dhcp-server-name>` |
| DHCP server FQDN | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| Scope name | `Corp-Users-VLAN10` | `<scope-name>` |
| Scope ID | `10.10.10.0` | `<scope-id>` |
| Scope subnet mask | `255.255.255.0` | `<subnet-mask>` |
| Scope start range | `10.10.10.50` | `<start-range>` |
| Scope end range | `10.10.10.200` | `<end-range>` |
| Default gateway | `10.10.10.1` | `<gateway-ip>` |
| Static infrastructure range | `10.10.10.1` to `10.10.10.49` | `<static-exclusion-range>` |
| Printer reservation range | `10.10.10.201` to `10.10.10.220` | `<printer-reservation-range>` |
| Server reservation range | `10.10.10.221` to `10.10.10.230` | `<server-reservation-range>` |
| First exclusion start | `10.10.10.1` | `<exclusion-start>` |
| First exclusion end | `10.10.10.49` | `<exclusion-end>` |
| Reservation target | `PRN-FRONT-01` | `<reservation-name>` |
| Reservation IP | `10.10.10.210` | `<reserved-ip>` |
| Reservation MAC / Client ID | `001122334455` | `<client-id>` |
| Reservation description | `Front office printer` | `<reservation-description>` |
| Reservation type | `Both` | `<dhcp-both-dhcp-bootp>` |
| Scope activation timing | After exclusions, reservations, and options | `<activation-plan>` |
| Client test machine | `WIN11-01` | `<client-name>` |
| Evidence path | `C:\DHCP-Exclusions-Reservations` | `<evidence-path>` |
| Next workbook | `06_Validate_DHCP_Client_Lease_Assignment.md` | `<next-task>` |
| Rollback stance | Remove incorrect exclusion or reservation only after checking active leases | `<rollback-plan>` |

# Configure_DHCPv4_Exclusions_And_Reservations_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Local Administrators or DHCP Administrators context is visible |
| 2 | Confirm DHCP role and tools | DHCP Server | `Get-WindowsFeature DHCP,RSAT-DHCP` | DHCP role and tools are installed |
| 3 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | DHCP service is Running |
| 4 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | DhcpServer module imports |
| 5 | Confirm DHCP server authorization | DHCP Server / DC | `Get-DhcpServerInDC` | DHCP server appears in AD authorized list |
| 6 | Confirm target scope exists | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Target scope exists |
| 7 | Confirm scope range | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>" \| Select-Object ScopeId,StartRange,EndRange,SubnetMask,State` | Scope boundaries are visible |
| 8 | Capture current exclusions | DHCP Server | `Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"` | Existing exclusion ranges are documented |
| 9 | Capture current reservations | DHCP Server | `Get-DhcpServerv4Reservation -ScopeId "<scope-id>"` | Existing reservations are documented |
| 10 | Capture current leases | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Active leases are known before changes |
| 11 | Confirm planned static range is inside scope | Operator | `Review scope, gateway, static device IPs, and planned exclusions` | No accidental overlap or wrong subnet |
| 12 | Add infrastructure exclusion range | DHCP Server | `Add-DhcpServerv4ExclusionRange -ScopeId "<scope-id>" -StartRange "<exclusion-start>" -EndRange "<exclusion-end>"` | DHCP will not lease excluded range |
| 13 | Confirm exclusion range | DHCP Server | `Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"` | Exclusion appears under target scope |
| 14 | Confirm reservation target MAC/client ID | Client / Device | `ipconfig /all` or vendor label / device inventory | Correct client identifier is known |
| 15 | Add DHCP reservation | DHCP Server | `Add-DhcpServerv4Reservation -ScopeId "<scope-id>" -IPAddress "<reserved-ip>" -ClientId "<client-id>" -Name "<reservation-name>" -Description "<description>"` | Reservation exists |
| 16 | Confirm reservation | DHCP Server | `Get-DhcpServerv4Reservation -ScopeId "<scope-id>"` | Reservation appears with expected IP and client ID |
| 17 | Remove conflicting active lease if needed | DHCP Server | `Remove-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<reserved-ip>"` | Old lease is removed if blocking reservation behavior |
| 18 | Confirm scope options still exist | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Gateway, DNS, and suffix options remain present |
| 19 | Keep scope inactive if still building | DHCP Server | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive` | Scope does not lease yet |
| 20 | Activate scope when ready | DHCP Server | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State Active` | Scope can issue leases |
| 21 | Validate reservation from client | Reserved Client | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Reserved client receives reserved IP |
| 22 | Validate server-side lease | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<reserved-ip>"` | Lease appears for reserved client |
| 23 | Export exclusion and reservation evidence | DHCP Server | Run validation skeleton | Evidence files are saved |
| 24 | Review DHCP events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50` | Recent DHCP events are visible |
| 25 | Document final state | Operator | `Record exclusions, reservations, client IDs, reserved IPs, scope state, and validation result` | Build record is complete |

# Configure_DHCPv4_Exclusions_And_Reservations_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: confirm scope, lease, exclusion, and reservation state before making changes.

$ScopeId = "10.10.10.0"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

# Confirm DHCP role, tools, and service.
Get-WindowsFeature DHCP,RSAT-DHCP |
  Tee-Object "$EvidencePath\dhcp-feature-state.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-state.txt"

# Import DHCP module.
Import-Module DhcpServer

# Confirm DHCP authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Confirm target scope exists and capture boundaries.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-before.txt"

Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration |
  Tee-Object "$EvidencePath\target-scope-boundaries.txt"

# Capture current scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-options-before.txt"

# Capture current exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\exclusions-before.txt"

# Capture current reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservations-before.txt"

# Capture current leases.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\leases-before.txt"

# Capture scope utilization.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-statistics-before.txt"
```

# Configure_DHCPv4_Exclusions_And_Reservations_Exclusion_Range_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: add exclusion ranges that DHCP should never lease.
# Use for gateways, static servers, network devices, printers with true static IPs, and management interfaces.

$ScopeId = "10.10.10.0"
$ExclusionStart = "10.10.10.1"
$ExclusionEnd = "10.10.10.49"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-exclusion-range-transcript.txt"

Import-Module DhcpServer

# Confirm scope before adding exclusion.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-before-exclusion.txt"

# Show existing exclusions before change.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\exclusions-before-change.txt"

# Add exclusion range.
Add-DhcpServerv4ExclusionRange `
  -ScopeId $ScopeId `
  -StartRange $ExclusionStart `
  -EndRange $ExclusionEnd

# Confirm exclusions after change.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\exclusions-after-change.txt"

# Export exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId |
  Export-Csv "$EvidencePath\exclusions-after-change.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_DHCPv4_Exclusions_And_Reservations_Reservation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: create DHCP reservations for clients that need predictable DHCP-managed addresses.

$ScopeId = "10.10.10.0"
$ReservedIp = "10.10.10.210"
$ClientId = "001122334455"
$ReservationName = "PRN-FRONT-01"
$ReservationDescription = "Front office printer"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-reservation-transcript.txt"

Import-Module DhcpServer

# Confirm scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-before-reservation.txt"

# Capture existing reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservations-before-change.txt"

# Capture existing lease for the reserved IP if present.
Get-DhcpServerv4Lease -ScopeId $ScopeId -IPAddress $ReservedIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reserved-ip-existing-lease-before.txt"

# Optional: remove an existing dynamic lease for the reserved IP if it conflicts.
# Use only after confirming the lease is safe to remove.
# Remove-DhcpServerv4Lease `
#   -ScopeId $ScopeId `
#   -IPAddress $ReservedIp

# Add reservation.
Add-DhcpServerv4Reservation `
  -ScopeId $ScopeId `
  -IPAddress $ReservedIp `
  -ClientId $ClientId `
  -Name $ReservationName `
  -Description $ReservationDescription

# Confirm reservation after change.
Get-DhcpServerv4Reservation -ScopeId $ScopeId |
  Where-Object {$_.IPAddress -eq $ReservedIp -or $_.ClientId -eq $ClientId} |
  Tee-Object "$EvidencePath\target-reservation-after-change.txt"

# Export all reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId |
  Export-Csv "$EvidencePath\reservations-after-change.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_DHCPv4_Exclusions_And_Reservations_Activation_Decision_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: keep the scope inactive during build or activate it when options, exclusions, and reservations are complete.

$ScopeId = "10.10.10.0"
$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm final scope configuration before activation decision.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration |
  Tee-Object "$EvidencePath\scope-before-activation-decision.txt"

Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-options-before-activation-decision.txt"

Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\exclusions-before-activation-decision.txt"

Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reservations-before-activation-decision.txt"

# Recommended during staged build:
# keep inactive until all required settings are complete.
Set-DhcpServerv4Scope `
  -ScopeId $ScopeId `
  -State InActive

# Use this only when scope options, exclusions, reservations, and relay path are ready.
# Set-DhcpServerv4Scope `
#   -ScopeId $ScopeId `
#   -State Active

# Confirm final scope state.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Select-Object ScopeId,Name,State,LeaseDuration,StartRange,EndRange |
  Tee-Object "$EvidencePath\scope-after-activation-decision.txt"
```

# Configure_DHCPv4_Exclusions_And_Reservations_Client_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a reserved DHCP client.
# Purpose: prove that the reserved client receives the reserved address after release/renew.

$EvidencePath = "C:\DHCP-Reservation-Client-Validation"
$ExpectedReservedIp = "10.10.10.210"
$ExpectedGateway = "10.10.10.1"
$ExpectedDnsServer = "10.10.10.10"
$ExpectedDnsSuffix = "corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture identity and current config.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-NetAdapter |
  Tee-Object "$EvidencePath\net-adapters.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-before.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-before.txt"

# Release and renew DHCP lease.
ipconfig /release |
  Tee-Object "$EvidencePath\ipconfig-release.txt"

ipconfig /renew |
  Tee-Object "$EvidencePath\ipconfig-renew.txt"

# Capture final client state.
ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-after.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-after.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dns-client-servers-after.txt"

# Test expected gateway and DNS.
Test-NetConnection $ExpectedGateway |
  Tee-Object "$EvidencePath\test-expected-gateway.txt"

Test-NetConnection $ExpectedDnsServer -Port 53 |
  Tee-Object "$EvidencePath\test-expected-dns-server-tcp-53.txt"

Resolve-DnsName $ExpectedDnsSuffix -Server $ExpectedDnsServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-expected-dns-suffix.txt"

# Review DHCP client events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 50 |
  Tee-Object "$EvidencePath\dhcp-client-admin-events.txt"

# Manual check:
# Confirm ipconfig /all shows the expected reserved IP:
# $ExpectedReservedIp
```

# Configure_DHCPv4_Exclusions_And_Reservations_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: collect DHCP Server events after configuring exclusions and reservations.

$EvidencePath = "C:\DHCP-Exclusions-Reservations"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DHCP Server operational events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 |
  Tee-Object "$EvidencePath\dhcp-server-operational-events.txt"

# DHCP Server admin events if present.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-admin-events.txt"

# System log DHCP-related events.
Get-WinEvent -LogName System -MaxEvents 200 |
  Where-Object {
    $_.ProviderName -like "*DHCP*" -or
    $_.Message -like "*DHCP*"
  } |
  Tee-Object "$EvidencePath\system-dhcp-events.txt"

# Confirm service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"
```

# Configure_DHCPv4_Exclusions_And_Reservations_Verification_Commands
```powershell
# Confirm DHCP role, tools, service, and authorization.
Get-WindowsFeature DHCP,RSAT-DHCP
Get-Service DHCPServer
Get-DhcpServerInDC

# Import DHCP module.
Import-Module DhcpServer

# Confirm target scope.
Get-DhcpServerv4Scope -ScopeId "<scope-id>"

# Confirm scope boundaries and state.
Get-DhcpServerv4Scope -ScopeId "<scope-id>" |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration

# View current scope options.
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"

# View exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"

# Add exclusion range.
Add-DhcpServerv4ExclusionRange `
  -ScopeId "<scope-id>" `
  -StartRange "<exclusion-start>" `
  -EndRange "<exclusion-end>"

# Remove exclusion range if wrong.
Remove-DhcpServerv4ExclusionRange `
  -ScopeId "<scope-id>" `
  -StartRange "<exclusion-start>" `
  -EndRange "<exclusion-end>"

# View reservations.
Get-DhcpServerv4Reservation -ScopeId "<scope-id>"

# Add reservation.
Add-DhcpServerv4Reservation `
  -ScopeId "<scope-id>" `
  -IPAddress "<reserved-ip>" `
  -ClientId "<client-id>" `
  -Name "<reservation-name>" `
  -Description "<reservation-description>"

# Remove reservation if wrong.
Remove-DhcpServerv4Reservation `
  -ScopeId "<scope-id>" `
  -ClientId "<client-id>"

# View leases.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<reserved-ip>"

# Remove conflicting lease if needed.
Remove-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<reserved-ip>"

# Check scope statistics.
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"

# Scope state control.
Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive
Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State Active

# Client validation.
ipconfig /all
ipconfig /release
ipconfig /renew
Get-NetIPConfiguration
Get-DnsClientServerAddress -AddressFamily IPv4

# Event logs.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 50
```

# Configure_DHCPv4_Exclusions_And_Reservations_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Wrong exclusion range added | `Remove-DhcpServerv4ExclusionRange -ScopeId "<scope-id>" -StartRange "<start>" -EndRange "<end>"` | Addresses may become leasable again |
| Exclusion range too large | Remove wrong range, then add smaller correct range | Clients may be unable to lease if too many addresses are excluded |
| Missing exclusion for static range | Add correct exclusion range | DHCP may lease addresses already used by static devices |
| Wrong reservation added | `Remove-DhcpServerv4Reservation -ScopeId "<scope-id>" -ClientId "<client-id>"` | Client may receive a different IP after renew |
| Reservation added with wrong MAC/client ID | Remove wrong reservation, then add correct client ID | Intended client will not receive reserved IP until fixed |
| Reservation IP conflicts with active lease | Remove conflicting lease only after confirming safe | Active client may lose lease |
| Reserved IP should become dynamic again | Remove reservation and confirm IP is not excluded | Address can be leased dynamically |
| Scope activated too early | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive` | Stops new lease assignment |
| Evidence folder created | `Remove-Item C:\DHCP-Exclusions-Reservations -Recurse -Force` | Deletes validation evidence |

# Configure_DHCPv4_Exclusions_And_Reservations_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Exclusion command fails | Range outside scope or invalid boundaries | Compare scope range, mask, start, and end | Recreating the DHCP server |
| Reservation command fails | Bad client ID, duplicate reservation, or IP outside scope | `Get-DhcpServerv4Reservation`; check reserved IP and client ID | Deleting the scope |
| Reserved client does not get reserved IP | Wrong MAC/client ID or old lease still active | `ipconfig /all`; `Get-DhcpServerv4Lease`; reservation ClientId | Changing DNS options |
| Client gets APIPA | Scope inactive, no relay, no lease path, or service issue | Scope state, DHCP service, relay, client VLAN | Editing reservations first |
| DHCP leases an address that should be static | Missing exclusion | `Get-DhcpServerv4ExclusionRange -ScopeId <scope-id>` | Reinstalling DHCP |
| Scope has no available addresses | Exclusion too large or scope exhausted | `Get-DhcpServerv4ScopeStatistics` | Restarting every client |
| Reservation IP is inside exclusion range | Design conflict | Compare reservations and exclusions | Assuming DHCP is broken |
| Printer still has old IP | Client did not renew or is statically configured | Printer network config and DHCP lease table | Recreating reservation repeatedly |
| Duplicate IP conflict appears | Static device and DHCP lease overlap | Ping, ARP table, lease table, exclusion plan | Expanding scope immediately |
| Reservation uses dashed MAC but server expects client ID format | Client ID formatting issue | Normalize MAC/client ID and compare with lease | Changing subnet mask |
| Scope options missing after reservation | Separate scope options issue | `Get-DhcpServerv4OptionValue -ScopeId <scope-id>` | Removing reservation |
| Reservation exists but no lease shown | Client has not requested lease yet | Renew client or restart DHCP client service | Deleting reservation |
| Remote subnet reservation fails | Relay, VLAN, or wrong scope issue | Confirm client subnet and relay path | Editing local scope only |
| DHCP service event warnings appear | Service, authorization, conflict, or scope issue | Review DHCP operational log | Ignoring event logs |

# Configure_DHCPv4_Exclusions_And_Reservations_Related_Labs
| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Install DHCP Server role | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | DHCP role and tools installation |
| Authorize DHCP server in AD | `02_Authorize_DHCP_Server_In_Active_Directory.md` | AD-integrated DHCP authorization |
| Create DHCPv4 scope | `03_Create_DHCPv4_Scope.md` | IPv4 lease pool creation |
| Configure DHCPv4 scope options | `04_Configure_DHCPv4_Scope_Options.md` | Gateway, DNS, DNS suffix, and lease duration assignment |
| Configure DHCPv4 exclusions and reservations | `05_Configure_DHCPv4_Exclusions_And_Reservations.md` | Static range protection and fixed client leases |
| Validate DHCP client lease assignment | `06_Validate_DHCP_Client_Lease_Assignment.md` | End-to-end DHCP lease and reservation validation |
| Configure DHCP relay and IP helper | `07_Configure_DHCP_Relay_And_IP_Helper.md` | Routed subnet DHCP support |
| Troubleshoot DHCP client lease failures | `12_Troubleshoot_DHCP_Client_Lease_Failures.md` | Exclusion, reservation, and lease conflict triage |