12_Troubleshoot_DHCP_Client_Lease_Failures.md
# 12_Troubleshoot_DHCP_Client_Lease_Failures

# Troubleshoot_DHCP_Client_Lease_Failures_Index
12_Troubleshoot_DHCP_Client_Lease_Failures.md
Troubleshoot_DHCP_Client_Lease_Failures
Troubleshoot_DHCP_Client_Lease_Failures_Source_Basis
Troubleshoot_DHCP_Client_Lease_Failures_Mental_Model
Troubleshoot_DHCP_Client_Lease_Failures_Symptom_Map
Troubleshoot_DHCP_Client_Lease_Failures_Planning_Table
Troubleshoot_DHCP_Client_Lease_Failures_Configuration_Checklist
Troubleshoot_DHCP_Client_Lease_Failures_Client_Triage_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Server_Service_And_Authorization_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Scope_And_Pool_Triage_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Reservation_And_Bad_Lease_Triage_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Options_Triage_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Relay_And_Network_Path_Triage_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Event_Log_And_Audit_Triage_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Remediation_Skeleton
Troubleshoot_DHCP_Client_Lease_Failures_Verification_Commands
Troubleshoot_DHCP_Client_Lease_Failures_Rollback
Troubleshoot_DHCP_Client_Lease_Failures_Failure_Checks
Troubleshoot_DHCP_Client_Lease_Failures_Related_Labs

# Troubleshoot_DHCP_Client_Lease_Failures_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Get-DhcpServerv4Lease | Inspecting active leases, all leases, client-specific leases, IP-specific leases, expired leases, offered leases, declined leases, and bad leases |
| Microsoft Learn | Get-DhcpServerv4Scope | Confirming scope existence, scope state, subnet, lease duration, start range, and end range |
| Microsoft Learn | Get-DhcpServerv4Binding | Confirming which server interfaces the DHCP service is bound to |
| Microsoft Learn | Get-DhcpServerv4Statistics | Checking overall DHCP server statistics and scope utilization pressure |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Verifying server-level, scope-level, reservation-level, policy-level, vendor-class, and user-class DHCP options |
| Microsoft Learn | Get-DhcpServerv4Reservation | Checking reservations and MAC/client identifier mapping |
| Microsoft Learn | Get-DhcpServerv4ExclusionRange | Checking excluded IP ranges that reduce usable pool addresses |
| Microsoft Learn | Get-DhcpServerInDC | Confirming DHCP servers authorized in Active Directory |
| Windows operational practice | `ipconfig`, `Get-NetIPConfiguration`, `Test-NetConnection`, `Resolve-DnsName`, Event Viewer, DHCP audit logs | Isolating client, server, scope, relay, option, and event-log causes of lease failure |

# Troubleshoot_DHCP_Client_Lease_Failures_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP lease failure | Client does not receive a usable IPv4 lease from the DHCP server |
| DORA | DHCP Discover, Offer, Request, Acknowledge process |
| Discover | Client broadcast asking for DHCP service |
| Offer | DHCP server offers an address from a matching scope |
| Request | Client requests the offered address |
| ACK | Server confirms the lease and sends options |
| NAK | Server rejects the client request, often because the requested address is invalid for that network |
| APIPA | `169.254.x.x` self-assigned address, usually meaning no DHCP lease was obtained |
| Scope | Address pool for a subnet |
| Active scope | Scope enabled and able to hand out addresses |
| Exhausted scope | Scope has no usable addresses left after active leases, exclusions, reservations, and bad leases |
| Exclusion | Address range inside the scope that DHCP will not hand out |
| Reservation | Mapping of a client identifier or MAC address to a specific IP address |
| Bad lease | Address marked declined or conflicted and temporarily unavailable |
| DHCP authorization | AD DS protection that allows approved DHCP servers to serve clients in a domain environment |
| Binding | DHCP server interface selected to listen and serve leases |
| Relay / IP helper | Router or L3 device forwards DHCP broadcasts from a remote subnet to the DHCP server |
| VLAN mismatch | Client is on a different subnet/VLAN than the scope or relay path expects |
| Option issue | Lease succeeds, but gateway, DNS, domain suffix, or vendor options are wrong |
| First rule | Prove whether the client failed to get any lease or got a bad lease with bad options before touching server configuration |

# Troubleshoot_DHCP_Client_Lease_Failures_Symptom_Map
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client has `169.254.x.x` address | No DHCP offer / no ACK | `ipconfig /all`; `ipconfig /renew` | DNS troubleshooting |
| Client keeps old address after moving VLANs | Client cache or wrong relay/scope | `ipconfig /release`; `ipconfig /renew`; verify VLAN | Rebuilding DHCP server |
| Client gets lease from wrong subnet | Relay/VLAN mismatch or rogue DHCP | Compare client IP with expected scope | Editing DNS options |
| Client receives IP but no gateway | Scope option 003 missing or wrong | `Get-DhcpServerv4OptionValue -ScopeId <scope-id>` | Releasing lease repeatedly |
| Client receives IP but DNS fails | Scope option 006 or 015 wrong | `ipconfig /all`; `Resolve-DnsName` | Recreating the scope |
| Only one VLAN fails | Relay/IP helper/routing path | Check relay target and subnet scope | Restarting all DHCP services |
| All clients fail | DHCP service, authorization, binding, firewall, or server outage | `Get-Service DHCPServer`; `Get-DhcpServerInDC`; `Get-DhcpServerv4Binding` | Editing reservations |
| Only one client fails | Client NIC, MAC reservation, stale/bad lease, local firewall, driver | Check client ID, reservation, bad leases | Changing scope-wide options |
| Reservation client gets wrong IP | Bad MAC/client ID or duplicate reservation | `Get-DhcpServerv4Reservation` | Restarting client repeatedly |
| Scope shows available addresses but client fails | Exclusions/reservations/bad leases/policies | Check all leases, exclusions, policies | Expanding scope blindly |
| Client sees DHCP NAK | Wrong requested IP, wrong subnet, stale lease | Release/renew and confirm client VLAN | Changing DNS server list |
| Client renewal fails after lease worked before | Server unreachable, relay broken, scope inactive, failover issue | Test path to DHCP server and relay | Deleting client object |
| DHCP server has no leases | Service not listening, wrong binding, unauthorized, wrong scope | Service, binding, authorization, scope state | Changing client DNS |
| DHCP audit logs show no client activity | Packet never reaches server | Relay, VLAN, ACL, firewall | Editing server options |
| DHCP audit logs show offer but no ACK | Client path, conflict, firewall, duplicate DHCP | Client event logs and packet path | Reinstalling DHCP role |

# Troubleshoot_DHCP_Client_Lease_Failures_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server name | `DHCP1` | `<dhcp-server>` |
| DHCP server IP | `10.10.10.20` | `<dhcp-server-ip>` |
| Client name | `WIN11-01` | `<client-name>` |
| Client interface alias | `Ethernet` | `<client-interface>` |
| Client MAC / client ID | `00-11-22-33-44-55` | `<client-id>` |
| Expected client subnet | `10.10.20.0/24` | `<client-subnet>` |
| Expected scope ID | `10.10.20.0` | `<scope-id>` |
| Expected gateway option | `10.10.20.1` | `<router-option>` |
| Expected DNS servers | `10.10.10.10,10.10.10.11` | `<dns-servers>` |
| Expected DNS suffix | `corp.local` | `<dns-suffix>` |
| Relay / IP helper device | `CoreSW1` | `<relay-device>` |
| Relay target | `10.10.10.20` | `<relay-target>` |
| Failure type | No lease / wrong lease / bad options / renew failure | `<failure-type>` |
| Evidence path | `C:\DHCPPrep\lease-failure-triage` | `<evidence-path>` |
| Rollback plan | Restore old option/scope/relay values | `<rollback-plan>` |

# Troubleshoot_DHCP_Client_Lease_Failures_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | Client and DHCP server | `New-Item -ItemType Directory -Force -Path C:\DHCPPrep\lease-failure-triage` | Evidence path exists |
| 2 | Confirm current client address state | Client | `ipconfig /all` | Current IP, DHCP server, gateway, DNS, and lease times are visible |
| 3 | Confirm PowerShell network state | Client | `Get-NetIPConfiguration`; `Get-NetAdapter` | Interface, IP, gateway, and DNS state are visible |
| 4 | Determine if client has APIPA | Client | `Get-NetIPAddress -AddressFamily IPv4` | `169.254.x.x` confirms no usable DHCP lease |
| 5 | Release current lease | Client | `ipconfig /release` | Current DHCP lease is released |
| 6 | Renew lease and capture output | Client | `ipconfig /renew` | Lease succeeds or error is captured |
| 7 | Confirm DHCP is enabled on client NIC | Client | `Get-NetIPInterface -InterfaceAlias "<client-interface>" -AddressFamily IPv4` | DHCP state is enabled |
| 8 | Confirm server reachability by IP | Client | `Test-NetConnection <dhcp-server-ip>` | Server IP is reachable if routed path exists |
| 9 | Confirm DHCP server service | DHCP server | `Get-Service DHCPServer` | Service is running |
| 10 | Confirm DHCP server authorization | DHCP server or DC | `Get-DhcpServerInDC` | Expected DHCP server is listed |
| 11 | Confirm DHCP bindings | DHCP server | `Get-DhcpServerv4Binding -ComputerName "<dhcp-server>"` | Correct serving interface is enabled |
| 12 | Confirm scope exists | DHCP server | `Get-DhcpServerv4Scope -ComputerName "<dhcp-server>" -ScopeId <scope-id>` | Expected scope exists |
| 13 | Confirm scope is active | DHCP server | `Get-DhcpServerv4Scope -ComputerName "<dhcp-server>" \| Select ScopeId,Name,State,StartRange,EndRange,SubnetMask` | Scope state is Active |
| 14 | Check utilization pressure | DHCP server | `Get-DhcpServerv4Statistics -ComputerName "<dhcp-server>"` | Server statistics are visible |
| 15 | Check active leases in expected scope | DHCP server | `Get-DhcpServerv4Lease -ComputerName "<dhcp-server>" -ScopeId <scope-id>` | Active leases are visible |
| 16 | Check all lease states in expected scope | DHCP server | `Get-DhcpServerv4Lease -ComputerName "<dhcp-server>" -ScopeId <scope-id> -AllLeases` | Active, offered, declined, expired, or other lease states are visible |
| 17 | Check bad leases | DHCP server | `Get-DhcpServerv4Lease -ComputerName "<dhcp-server>" -ScopeId <scope-id> -BadLeases` | Bad or declined leases are identified |
| 18 | Check client-specific lease | DHCP server | `Get-DhcpServerv4Lease -ComputerName "<dhcp-server>" -ScopeId <scope-id> -ClientId "<client-id>"` | Client lease is found or confirmed absent |
| 19 | Check address-specific lease | DHCP server | `Get-DhcpServerv4Lease -ComputerName "<dhcp-server>" -IPAddress <client-ip>` | Lease record for target IP is visible |
| 20 | Check exclusions | DHCP server | `Get-DhcpServerv4ExclusionRange -ComputerName "<dhcp-server>" -ScopeId <scope-id>` | Exclusion ranges are known |
| 21 | Check reservations | DHCP server | `Get-DhcpServerv4Reservation -ComputerName "<dhcp-server>" -ScopeId <scope-id>` | Reservation mappings are visible |
| 22 | Check scope options | DHCP server | `Get-DhcpServerv4OptionValue -ComputerName "<dhcp-server>" -ScopeId <scope-id>` | Router, DNS, suffix, and other options are visible |
| 23 | Check server-level options | DHCP server | `Get-DhcpServerv4OptionValue -ComputerName "<dhcp-server>"` | Inherited defaults are visible |
| 24 | Check reservation-level options if reservation exists | DHCP server | `Get-DhcpServerv4OptionValue -ComputerName "<dhcp-server>" -ReservedIP <reserved-ip>` | Reservation-specific override options are visible |
| 25 | Check DHCP server logs | DHCP server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100` | Recent DHCP service events are visible if log exists |
| 26 | Check DHCP client logs | Client | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100` | Recent client DHCP events are visible if log exists |
| 27 | Check DHCP audit log files | DHCP server | `Get-ChildItem C:\Windows\System32\dhcp\DhcpSrvLog-*.log` | DHCP audit logs are present if enabled |
| 28 | Search audit logs for client ID or MAC | DHCP server | `Select-String -Path C:\Windows\System32\dhcp\DhcpSrvLog-*.log -Pattern "<client-id>","<client-ip>"` | Client DHCP activity is found or confirmed absent |
| 29 | Confirm relay path if client is remote | Relay device | `show running-config interface <vlan-interface>` / `show ip interface <vlan-interface>` | DHCP relay/IP helper points to DHCP server |
| 30 | Confirm no ACL blocks DHCP relay | Relay/firewall | Check UDP 67/68 and relay path | DHCP packets can reach server and replies can return |
| 31 | Confirm correct VLAN/subnet | Switch/router | Check access VLAN, SVI, gateway, relay, and scope subnet | Client subnet matches expected DHCP scope |
| 32 | Apply smallest remediation | Appropriate device | Fix service, authorization, binding, scope, pool, reservation, options, relay, or VLAN | Root cause is corrected |
| 33 | Force client renewal after fix | Client | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Client receives expected lease and options |
| 34 | Confirm server lease record after fix | DHCP server | `Get-DhcpServerv4Lease -ComputerName "<dhcp-server>" -ScopeId <scope-id> -ClientId "<client-id>"` | Lease appears under expected scope |
| 35 | Document root cause | Operator | `Record failure type, fix, affected scope, affected client, evidence files` | Troubleshooting evidence is preserved |

# Troubleshoot_DHCP_Client_Lease_Failures_Client_Triage_Skeleton
```powershell
# Run in elevated PowerShell or CMD on the affected Windows client.

$EvidencePath = "C:\DHCPPrep\lease-failure-triage"
$InterfaceAlias = "Ethernet"
$DhcpServerIp = "10.10.10.20"
$ExpectedDnsName = "corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture identity and network baseline.
hostname | Tee-Object "$EvidencePath\client-hostname.txt"

whoami | Tee-Object "$EvidencePath\client-whoami.txt"

ipconfig /all | Tee-Object "$EvidencePath\client-ipconfig-before.txt"

Get-NetAdapter |
  Tee-Object "$EvidencePath\client-netadapter.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfiguration-before.txt"

Get-NetIPAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\client-ipv4-addresses-before.txt"

Get-NetIPInterface `
  -InterfaceAlias $InterfaceAlias `
  -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\client-ip-interface-before.txt"

# Confirm whether the interface is DHCP-enabled.
Get-NetIPInterface `
  -InterfaceAlias $InterfaceAlias `
  -AddressFamily IPv4 |
  Select-Object InterfaceAlias,Dhcp,ConnectionState,NlMtu,InterfaceMetric

# Basic reachability to DHCP server by IP.
Test-NetConnection $DhcpServerIp |
  Tee-Object "$EvidencePath\client-test-dhcp-server-ip.txt"

# Release and renew.
ipconfig /release | Tee-Object "$EvidencePath\client-ipconfig-release.txt"
ipconfig /renew   | Tee-Object "$EvidencePath\client-ipconfig-renew.txt"

# Capture after-state.
ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-after-renew.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfiguration-after-renew.txt"

Get-NetIPAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\client-ipv4-addresses-after-renew.txt"

# If a lease was received, test gateway and DNS behavior.
$CurrentConfig = Get-NetIPConfiguration -InterfaceAlias $InterfaceAlias

$CurrentConfig

if ($CurrentConfig.IPv4DefaultGateway) {
    Test-NetConnection $CurrentConfig.IPv4DefaultGateway.NextHop |
      Tee-Object "$EvidencePath\client-test-default-gateway.txt"
}

Resolve-DnsName $ExpectedDnsName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-resolve-expected-domain.txt"

# Client DHCP event logs, if present.
Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Client/Admin" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-dhcp-admin-events.txt"

Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Client/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-dhcp-operational-events.txt"