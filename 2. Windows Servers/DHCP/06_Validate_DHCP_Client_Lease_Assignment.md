06_Validate_DHCP_Client_Lease_Assignment.md
# Validate_DHCP_Client_Lease_Assignment

# Validate_DHCP_Client_Lease_Assignment_Index
06_Validate_DHCP_Client_Lease_Assignment.md
Validate_DHCP_Client_Lease_Assignment
Validate_DHCP_Client_Lease_Assignment_Source_Basis
Validate_DHCP_Client_Lease_Assignment_Mental_Model
Validate_DHCP_Client_Lease_Assignment_Planning_Table
Validate_DHCP_Client_Lease_Assignment_Configuration_Checklist
Validate_DHCP_Client_Lease_Assignment_Server_Precheck_Skeleton
Validate_DHCP_Client_Lease_Assignment_Client_Baseline_Skeleton
Validate_DHCP_Client_Lease_Assignment_Client_Renewal_Skeleton
Validate_DHCP_Client_Lease_Assignment_Server_Lease_Verification_Skeleton
Validate_DHCP_Client_Lease_Assignment_DNS_And_Gateway_Validation_Skeleton
Validate_DHCP_Client_Lease_Assignment_Event_Log_Skeleton
Validate_DHCP_Client_Lease_Assignment_Verification_Commands
Validate_DHCP_Client_Lease_Assignment_Rollback
Validate_DHCP_Client_Lease_Assignment_Failure_Checks
Validate_DHCP_Client_Lease_Assignment_Related_Labs

# Validate_DHCP_Client_Lease_Assignment_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Get-DhcpServerv4Scope | Confirming target DHCPv4 scope exists and is active |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Verifying scope options delivered to clients |
| Microsoft Learn | Get-DhcpServerv4ExclusionRange | Confirming excluded ranges are protected |
| Microsoft Learn | Get-DhcpServerv4Reservation | Confirming reserved clients have fixed assignments |
| Microsoft Learn | Get-DhcpServerv4Lease | Verifying DHCP lease records on the server |
| Microsoft Learn | Get-DhcpServerv4ScopeStatistics | Checking available and used addresses |
| Microsoft Learn | Get-Service | Confirming DHCP Server service state |
| Microsoft Learn | Resolve-DnsName | Validating DNS behavior after lease assignment |
| Microsoft Learn | Test-NetConnection | Testing gateway, DNS, and server reachability |
| Microsoft Learn | DHCP Client and DHCP Server event logs | Reviewing lease request, offer, ack, failure, and service events |
| Windows Server operational practice | Client lease renewal, APIPA detection, gateway validation, DNS validation | Proving that DHCP works end to end from server to client |

# Validate_DHCP_Client_Lease_Assignment_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP lease assignment | Client receives an IP address and network options from the DHCP server |
| DORA process | Discover, Offer, Request, Acknowledge sequence used by DHCPv4 clients |
| DHCP client | Endpoint requesting leased IP configuration |
| DHCP server | Server providing scope-based leases and options |
| Scope active state | Active scopes can lease addresses; inactive scopes cannot |
| Lease table | Server-side record of active, inactive, expired, or reserved leases |
| Client lease state | Client-side IP configuration shown through `ipconfig /all` and PowerShell networking cmdlets |
| APIPA | `169.254.x.x` self-assigned address used when DHCP fails |
| Scope options | Gateway, DNS servers, DNS suffix, and other values delivered with the lease |
| Gateway validation | Confirms the client can leave its local subnet |
| DNS validation | Confirms the client can resolve expected names with DHCP-provided DNS settings |
| Reservation validation | Confirms a specific client receives its planned reserved IP |
| Relay dependency | Clients outside the DHCP server subnet require relay or IP helper |
| First rule | A valid DHCP test proves server scope state, client lease state, options, gateway reachability, DNS resolution, and event logs agree with each other |

# Validate_DHCP_Client_Lease_Assignment_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server name | `DHCP1` | `<dhcp-server-name>` |
| DHCP server FQDN | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.20` | `<dhcp-server-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Scope name | `Corp-Users-VLAN10` | `<scope-name>` |
| Scope ID | `10.10.10.0` | `<scope-id>` |
| Scope subnet mask | `255.255.255.0` | `<subnet-mask>` |
| Scope start range | `10.10.10.50` | `<start-range>` |
| Scope end range | `10.10.10.200` | `<end-range>` |
| Expected default gateway | `10.10.10.1` | `<gateway-ip>` |
| Expected DNS server 1 | `10.10.10.10` | `<dns-server-1>` |
| Expected DNS server 2 | `10.10.10.11` | `<dns-server-2>` |
| Expected DNS suffix | `corp.local` | `<dns-suffix>` |
| Client test machine | `WIN11-01` | `<client-name>` |
| Client interface alias | `Ethernet` | `<client-interface-alias>` |
| Expected client address type | DHCP | `<dhcp-static>` |
| Expected client lease range | `10.10.10.50-10.10.10.200` | `<expected-lease-range>` |
| Reservation expected | No / Yes | `<yes-no>` |
| Reserved IP if applicable | `10.10.10.210` | `<reserved-ip>` |
| Relay required | No for same subnet, yes for remote subnet | `<yes-no>` |
| Evidence path on server | `C:\DHCP-Lease-Validation` | `<server-evidence-path>` |
| Evidence path on client | `C:\DHCP-Client-Lease-Validation` | `<client-evidence-path>` |
| Next workbook | `07_Configure_DHCP_Relay_And_IP_Helper.md` | `<next-task>` |
| Rollback stance | Return client to prior DNS/static state if validation changes client config | `<rollback-plan>` |

# Validate_DHCP_Client_Lease_Assignment_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm DHCP server role and tools | DHCP Server | `Get-WindowsFeature DHCP,RSAT-DHCP` | DHCP role and tools are installed |
| 2 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | Service state is Running |
| 3 | Confirm DHCP server authorization | DHCP Server / DC | `Get-DhcpServerInDC` | DHCP server appears in authorized list |
| 4 | Confirm target scope exists | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Target scope exists |
| 5 | Confirm target scope active | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>" \| Select-Object State` | State is Active |
| 6 | Confirm scope options | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Router, DNS, and DNS suffix are present |
| 7 | Confirm exclusions | DHCP Server | `Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"` | Static ranges are excluded |
| 8 | Confirm reservations if expected | DHCP Server | `Get-DhcpServerv4Reservation -ScopeId "<scope-id>"` | Reservation records are visible |
| 9 | Confirm scope utilization | DHCP Server | `Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"` | Available address count is not exhausted |
| 10 | Capture current leases | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Current lease state is known |
| 11 | Confirm client interface | Client | `Get-NetAdapter` | Correct test interface is known |
| 12 | Capture client baseline | Client | `ipconfig /all`; `Get-NetIPConfiguration` | Current client IP state is documented |
| 13 | Confirm client uses DHCP | Client | `Get-NetIPInterface`; `Get-NetIPConfiguration` | Interface is expected to use DHCP |
| 14 | Clear client DNS cache | Client | `Clear-DnsClientCache` | Stale DNS cache is cleared |
| 15 | Release current lease | Client | `ipconfig /release` | Current DHCP lease is released |
| 16 | Renew DHCP lease | Client | `ipconfig /renew` | Client receives DHCP lease |
| 17 | Capture client lease after renew | Client | `ipconfig /all`; `Get-NetIPConfiguration` | Client shows expected DHCP IP, gateway, DNS, and suffix |
| 18 | Confirm no APIPA address | Client | `Get-NetIPAddress -AddressFamily IPv4` | Client does not use `169.254.x.x` |
| 19 | Confirm gateway reachability | Client | `Test-NetConnection "<gateway-ip>"` | Gateway responds or TCP test path is reachable |
| 20 | Confirm DNS server reachability | Client | `Test-NetConnection "<dns-server-ip>" -Port 53` | DNS server is reachable on TCP 53 |
| 21 | Confirm DNS resolution | Client | `Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"` | Domain name resolves |
| 22 | Confirm server-side lease | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Client lease appears on DHCP server |
| 23 | Confirm reservation behavior if expected | DHCP Server / Client | Compare reservation IP to client `ipconfig /all` | Reserved client receives expected IP |
| 24 | Review client DHCP events | Client | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 50` | Lease events are visible |
| 25 | Review server DHCP events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50` | Server lease events are visible |
| 26 | Export validation evidence | DHCP Server / Client | Run validation skeletons | Evidence files are saved |
| 27 | Document final validation state | Operator | `Record client IP, lease time, DHCP server, gateway, DNS, suffix, lease ID, and pass/fail` | Lease assignment proof is complete |

# Validate_DHCP_Client_Lease_Assignment_Server_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: confirm the DHCP server and target scope are ready before testing a client lease.

$ScopeId = "10.10.10.0"
$EvidencePath = "C:\DHCP-Lease-Validation"

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

# Confirm target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope.txt"

# Confirm target scope boundaries and state.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration |
  Tee-Object "$EvidencePath\target-scope-summary.txt"

# Confirm scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-options.txt"

# Confirm exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-exclusions.txt"

# Confirm reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-reservations.txt"

# Confirm utilization and current leases.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-statistics-before-client-test.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-leases-before-client-test.txt"
```

# Validate_DHCP_Client_Lease_Assignment_Client_Baseline_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP client.
# Purpose: capture the client state before release and renew.

$ClientEvidencePath = "C:\DHCP-Client-Lease-Validation"
$InterfaceAlias = "Ethernet"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$ClientEvidencePath\whoami-groups.txt"

# Capture client identity.
hostname |
  Tee-Object "$ClientEvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$ClientEvidencePath\computer-domain-state.txt"

# Capture adapters and interface details.
Get-NetAdapter |
  Tee-Object "$ClientEvidencePath\net-adapters.txt"

Get-NetIPInterface |
  Tee-Object "$ClientEvidencePath\net-ip-interfaces.txt"

# Capture current IP and DNS state.
Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration-before.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\dns-client-servers-before.txt"

Get-DnsClientGlobalSetting |
  Tee-Object "$ClientEvidencePath\dns-client-global-setting-before.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-before.txt"

# Confirm DHCP client service.
Get-Service Dhcp |
  Tee-Object "$ClientEvidencePath\dhcp-client-service.txt"

# Clear DNS cache before validation.
Clear-DnsClientCache

# Optional: confirm interface is DHCP-enabled.
Get-NetIPInterface -InterfaceAlias $InterfaceAlias -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\target-interface-ip-state.txt"
```

# Validate_DHCP_Client_Lease_Assignment_Client_Renewal_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP client.
# Purpose: force the client to request a fresh DHCP lease and capture the result.

$ClientEvidencePath = "C:\DHCP-Client-Lease-Validation"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Capture state before release.
ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-immediately-before-release.txt"

# Release current IPv4 DHCP lease.
ipconfig /release |
  Tee-Object "$ClientEvidencePath\ipconfig-release.txt"

# Optional pause for readability.
Start-Sleep -Seconds 3

# Capture state after release.
ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-after-release.txt"

# Renew DHCP lease.
ipconfig /renew |
  Tee-Object "$ClientEvidencePath\ipconfig-renew.txt"

# Optional pause for DHCP processing.
Start-Sleep -Seconds 5

# Capture final state after renew.
ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-after-renew.txt"

Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration-after-renew.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\dns-client-servers-after-renew.txt"

Get-NetIPAddress -AddressFamily IPv4 |
  Tee-Object "$ClientEvidencePath\net-ipv4-addresses-after-renew.txt"

# Detect APIPA.
Get-NetIPAddress -AddressFamily IPv4 |
  Where-Object {$_.IPAddress -like "169.254.*"} |
  Tee-Object "$ClientEvidencePath\apipa-check.txt"
```

# Validate_DHCP_Client_Lease_Assignment_Server_Lease_Verification_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server after the client renews.
# Purpose: confirm the DHCP server recorded the client lease.

$ScopeId = "10.10.10.0"
$ClientName = "WIN11-01"
$ClientMacOrId = "001122334455"
$EvidencePath = "C:\DHCP-Lease-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Capture all leases after client renew.
Get-DhcpServerv4Lease -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-leases-after-client-renew.txt"

# Search for client by hostname.
Get-DhcpServerv4Lease -ScopeId $ScopeId |
  Where-Object {$_.HostName -like "*$ClientName*"} |
  Tee-Object "$EvidencePath\client-lease-by-hostname.txt"

# Search for client by client ID if known.
Get-DhcpServerv4Lease -ScopeId $ScopeId |
  Where-Object {$_.ClientId -eq $ClientMacOrId} |
  Tee-Object "$EvidencePath\client-lease-by-clientid.txt"

# Confirm utilization after lease.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-statistics-after-client-renew.txt"

# Confirm reservations in case the client should receive a fixed lease.
Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-reservations-after-client-renew.txt"

# Export lease table.
Get-DhcpServerv4Lease -ScopeId $ScopeId |
  Export-Csv "$EvidencePath\target-scope-leases-after-client-renew.csv" -NoTypeInformation
```

# Validate_DHCP_Client_Lease_Assignment_DNS_And_Gateway_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP client after lease renewal.
# Purpose: prove the DHCP-provided network configuration is usable.

$ClientEvidencePath = "C:\DHCP-Client-Lease-Validation"
$ExpectedGateway = "10.10.10.1"
$ExpectedDnsServer = "10.10.10.10"
$ExpectedDnsSuffix = "corp.local"
$DhcpServerIp = "10.10.10.20"

New-Item -ItemType Directory -Force -Path $ClientEvidencePath

# Capture final IP configuration.
Get-NetIPConfiguration |
  Tee-Object "$ClientEvidencePath\net-ip-configuration-final.txt"

ipconfig /all |
  Tee-Object "$ClientEvidencePath\ipconfig-all-final.txt"

# Test gateway reachability.
Test-NetConnection $ExpectedGateway |
  Tee-Object "$ClientEvidencePath\test-default-gateway.txt"

# Test DNS server reachability.
Test-NetConnection $ExpectedDnsServer -Port 53 |
  Tee-Object "$ClientEvidencePath\test-dns-server-tcp-53.txt"

# Test DHCP server reachability.
# DHCP uses UDP 67/68, so this TCP test only proves general path/name reachability if available.
Test-NetConnection $DhcpServerIp |
  Tee-Object "$ClientEvidencePath\test-dhcp-server-ip.txt"

# Test DNS suffix/domain resolution.
Resolve-DnsName $ExpectedDnsSuffix -Server $ExpectedDnsServer -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-domain-fqdn.txt"

# Test domain controller locator if the client is in an AD environment.
nltest /dsgetdc:$ExpectedDnsSuffix |
  Tee-Object "$ClientEvidencePath\nltest-dsgetdc.txt"

# Test internet or upstream resolution only if allowed in the lab.
Resolve-DnsName "www.microsoft.com" -Server $ExpectedDnsServer -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\resolve-external-test-name.txt"
```

# Validate_DHCP_Client_Lease_Assignment_Event_Log_Skeleton
```powershell
# Run on both DHCP server and DHCP client as applicable.
# Purpose: collect DHCP event logs for lease assignment validation.

$ServerEvidencePath = "C:\DHCP-Lease-Validation"
$ClientEvidencePath = "C:\DHCP-Client-Lease-Validation"

# DHCP server event collection.
if (Get-Service DHCPServer -ErrorAction SilentlyContinue) {
  New-Item -ItemType Directory -Force -Path $ServerEvidencePath

  Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 |
    Tee-Object "$ServerEvidencePath\dhcp-server-operational-events.txt"

  Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Tee-Object "$ServerEvidencePath\dhcp-server-admin-events.txt"

  Get-WinEvent -LogName System -MaxEvents 200 |
    Where-Object {
      $_.ProviderName -like "*DHCP*" -or
      $_.Message -like "*DHCP*"
    } |
    Tee-Object "$ServerEvidencePath\system-dhcp-events.txt"

  Get-Service DHCPServer |
    Tee-Object "$ServerEvidencePath\dhcp-server-service-final.txt"
}

# DHCP client event collection.
New-Item -ItemType Directory -Force -Path $ClientEvidencePath

Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 |
  Tee-Object "$ClientEvidencePath\dhcp-client-admin-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$ClientEvidencePath\dhcp-client-operational-events.txt"

Get-WinEvent -LogName System -MaxEvents 200 |
  Where-Object {
    $_.ProviderName -like "*DHCP*" -or
    $_.Message -like "*DHCP*"
  } |
  Tee-Object "$ClientEvidencePath\system-dhcp-events.txt"

Get-Service Dhcp |
  Tee-Object "$ClientEvidencePath\dhcp-client-service-final.txt"
```

# Validate_DHCP_Client_Lease_Assignment_Verification_Commands
```powershell
# Server-side role, service, authorization.
Get-WindowsFeature DHCP,RSAT-DHCP
Get-Service DHCPServer
Get-DhcpServerInDC

# Server-side scope validation.
Import-Module DhcpServer
Get-DhcpServerv4Scope
Get-DhcpServerv4Scope -ScopeId "<scope-id>"
Get-DhcpServerv4Scope -ScopeId "<scope-id>" | Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"
Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"
Get-DhcpServerv4Reservation -ScopeId "<scope-id>"
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"

# Server-side lease validation.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" | Where-Object {$_.HostName -like "*<client-name>*"}
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<client-ip>"

# Client-side baseline.
hostname
Get-NetAdapter
Get-NetIPConfiguration
Get-DnsClientServerAddress -AddressFamily IPv4
ipconfig /all

# Client-side renewal.
Clear-DnsClientCache
ipconfig /release
ipconfig /renew
ipconfig /all

# Client-side APIPA check.
Get-NetIPAddress -AddressFamily IPv4 | Where-Object {$_.IPAddress -like "169.254.*"}

# Client-side gateway and DNS tests.
Test-NetConnection "<gateway-ip>"
Test-NetConnection "<dns-server-ip>" -Port 53
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"

# AD domain locator test if applicable.
nltest /dsgetdc:<domain-fqdn>

# Event logs.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 50
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DHCP*" -or $_.Message -like "*DHCP*"}
```

# Validate_DHCP_Client_Lease_Assignment_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Client lease released during test | `ipconfig /renew` | Client may temporarily lose network connectivity |
| Client fails to renew | Reconnect to correct VLAN, restore static config if previously static, or troubleshoot DHCP path | Client remains offline until network config is restored |
| Client was changed from static to DHCP | Reapply previous static IP, gateway, DNS, and suffix settings | Wrong static settings can cause conflict |
| Client DNS cache cleared | No rollback needed | Cached entries are rebuilt through DNS queries |
| Conflicting lease removed | Client may need renew or reboot | Removing wrong lease can disrupt another client |
| Scope activated for validation | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive` | Stops new lease assignment |
| Scope option changed during validation | Reapply previous option value with `Set-DhcpServerv4OptionValue` | Clients may receive bad network config until corrected and renewed |
| Reservation adjusted for validation | Remove incorrect reservation and recreate correct reservation | Reserved device may receive wrong IP |
| Evidence folders created | Remove validation folders if no longer needed | Deletes troubleshooting evidence |

# Validate_DHCP_Client_Lease_Assignment_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client gets `169.254.x.x` | DHCP request failed | `ipconfig /all`; scope state; DHCP service; VLAN/relay path | Editing DNS suffix |
| Client keeps old IP after renew | Lease cache, reservation, wrong VLAN, or alternate DHCP server | `ipconfig /release`; `ipconfig /renew`; server lease table | Rebuilding the DHCP server |
| Client gets IP from wrong subnet | Wrong VLAN, wrong relay, rogue DHCP, or overlapping scope | Client switchport/VLAN and DHCP server lease table | Changing lease duration |
| Client gets IP but no gateway | Missing or wrong option 003 | `Get-DhcpServerv4OptionValue -ScopeId <scope-id>` | Removing reservation |
| Client gets IP but DNS fails | Missing or wrong option 006 | `ipconfig /all`; DNS server option | Recreating scope |
| Client gets DNS but wrong suffix | Missing or wrong option 015 | `ipconfig /all` DNS suffix section | Reauthorizing DHCP |
| Client cannot reach gateway | Wrong gateway option, VLAN issue, L2 issue, or gateway down | `Test-NetConnection <gateway-ip>` | Editing DHCP leases first |
| Client cannot reach DNS server | DNS server path or option issue | `Test-NetConnection <dns-ip> -Port 53` | Changing scope range |
| Server shows no lease for client | Request never reached server, wrong scope, or rogue server answered | DHCP server lease table and client DHCP server in `ipconfig /all` | Removing exclusions |
| Server shows lease but client lacks address | Client-side renew issue, adapter issue, or stale output | `ipconfig /all`; DHCP Client events | Recreating reservation |
| Reserved client gets dynamic IP | Wrong ClientId or existing lease conflict | `Get-DhcpServerv4Reservation`; `ipconfig /all` MAC | Changing router option |
| Scope has no available addresses | Scope exhaustion | `Get-DhcpServerv4ScopeStatistics` | Restarting DHCP service repeatedly |
| Remote client fails but local client works | Relay/IP helper or routing issue | Router/L3 SVI helper and route path | Editing local server role |
| DHCP server unauthorized | AD authorization missing or stale | `Get-DhcpServerInDC` | Troubleshooting client adapter |
| DHCP service stopped | Server service issue | `Get-Service DHCPServer` | Releasing clients repeatedly |
| DHCP events show NACK | Client requested invalid address or wrong subnet | DHCP client event log and server lease table | Recreating entire server |
| Client domain logon fails after lease | DNS option or AD locator issue | `Resolve-DnsName`; `nltest /dsgetdc:<domain>` | Blaming DHCP lease itself |
| Firewall blocks test traffic | Client/server/network firewall issue | Test path and event logs | Changing scope options blindly |

# Validate_DHCP_Client_Lease_Assignment_Related_Labs
| Lab                                                | Related Workbook                                         | Skill Proven                                      |
| -------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------- |
| Install DHCP Server role                           | `01_Install_DHCP_Server_Role_And_Management_Tools.md`    | DHCP role and tools installation                  |
| Authorize DHCP server in AD                        | `02_Authorize_DHCP_Server_In_Active_Directory.md`        | AD-integrated DHCP authorization                  |
| Create DHCPv4 scope                                | `03_Create_DHCPv4_Scope.md`                              | IPv4 lease pool creation                          |
| Configure DHCPv4 scope options                     | `04_Configure_DHCPv4_Scope_Options.md`                   | Gateway, DNS, suffix, and lease duration delivery |
| Configure DHCPv4 exclusions and reservations       | `05_Configure_DHCPv4_Exclusions_And_Reservations.md`     | Static range protection and fixed lease mapping   |
| Validate DHCP client lease assignment              | `06_Validate_DHCP_Client_Lease_Assignment.md`            | End-to-end DHCP client lease proof                |
| Configure DHCP relay and IP helper                 | `07_Configure_DHCP_Relay_And_IP_Helper.md`               | Remote subnet DHCP forwarding                     |
| Configure DHCP DNS dynamic updates                 | `08_Configure_DHCP_DNS_Dynamic_Updates.md`               | DHCP-to-DNS registration validation               |
| Monitor DHCP leases, events, and scope utilization | `11_Monitor_DHCP_Leases_Events_And_Scope_Utilization.md` | Operational DHCP visibility                       |
| Troubleshoot DHCP client lease failures            | `12_Troubleshoot_DHCP_Client_Lease_Failures.md`          | Failed lease root-cause workflow                  |