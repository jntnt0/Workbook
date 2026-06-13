07_Configure_DHCP_Relay_And_IP_Helper.md
# Configure_DHCP_Relay_And_IP_Helper

# Configure_DHCP_Relay_And_IP_Helper_Index
07_Configure_DHCP_Relay_And_IP_Helper.md
Configure_DHCP_Relay_And_IP_Helper
Configure_DHCP_Relay_And_IP_Helper_Source_Basis
Configure_DHCP_Relay_And_IP_Helper_Mental_Model
Configure_DHCP_Relay_And_IP_Helper_Planning_Table
Configure_DHCP_Relay_And_IP_Helper_Configuration_Checklist
Configure_DHCP_Relay_And_IP_Helper_Precheck_Skeleton
Configure_DHCP_Relay_And_IP_Helper_DHCP_Server_Scope_Readiness_Skeleton
Configure_DHCP_Relay_And_IP_Helper_Network_Device_IP_Helper_Skeleton
Configure_DHCP_Relay_And_IP_Helper_Windows_RRAS_DHCP_Relay_Skeleton
Configure_DHCP_Relay_And_IP_Helper_Firewall_And_Path_Validation_Skeleton
Configure_DHCP_Relay_And_IP_Helper_Client_Lease_Validation_Skeleton
Configure_DHCP_Relay_And_IP_Helper_Server_Lease_Validation_Skeleton
Configure_DHCP_Relay_And_IP_Helper_Event_Log_Skeleton
Configure_DHCP_Relay_And_IP_Helper_Verification_Commands
Configure_DHCP_Relay_And_IP_Helper_Rollback
Configure_DHCP_Relay_And_IP_Helper_Failure_Checks
Configure_DHCP_Relay_And_IP_Helper_Related_Labs

# Configure_DHCP_Relay_And_IP_Helper_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DHCP Server PowerShell module | DHCP scope, lease, option, authorization, and server validation |
| Microsoft Learn | Get-DhcpServerv4Scope | Confirming the DHCP scope for the remote subnet exists |
| Microsoft Learn | Get-DhcpServerv4Lease | Validating lease assignment after relay configuration |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Confirming gateway, DNS, and domain options for relayed clients |
| Microsoft Learn | DHCP Server event logs | Reviewing lease assignment and DHCP server operational events |
| Microsoft Learn | Remote Access / RRAS DHCP Relay Agent | Optional Windows Server DHCP relay behavior |
| Cisco IOS operational practice | `ip helper-address` | Relaying DHCP broadcasts from client VLANs to DHCP server |
| Windows Server operational practice | DHCP relay, routed VLANs, scope selection by gateway interface | Supporting DHCP clients across subnets without placing DHCP server in every VLAN |

# Configure_DHCP_Relay_And_IP_Helper_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP broadcast | Client Discover traffic starts as local subnet broadcast |
| Routed boundary | Routers do not forward broadcasts by default |
| DHCP relay | Device that receives client broadcast and forwards it as unicast to DHCP server |
| IP helper address | Common network-device command that points DHCP broadcasts toward the DHCP server |
| Relay interface | Layer 3 interface/SVI/default gateway for the client subnet |
| GIADDR | Gateway IP address field inserted by relay so DHCP server knows which scope to use |
| DHCP server scope selection | DHCP server chooses the scope matching the relayed subnet/GIADDR |
| Remote subnet scope | DHCP scope for a subnet not directly connected to the DHCP server |
| UDP 67 | DHCP server/relay destination port |
| UDP 68 | DHCP client port |
| Option 003 | Router/default gateway option delivered to clients |
| Option 006 | DNS server option delivered to clients |
| Option 015 | DNS domain suffix option delivered to clients |
| Option 082 | Relay agent information option, sometimes used in enterprise networks |
| First rule | Build the DHCP scope first, then configure relay on the client VLAN gateway, then validate lease assignment from a real client |

# Configure_DHCP_Relay_And_IP_Helper_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.5` | `<dhcp-server-ip>` |
| DHCP server VLAN | `Server-VLAN` | `<server-vlan>` |
| Client VLAN name | `Workstations-VLAN20` | `<client-vlan-name>` |
| Client subnet | `10.20.20.0/24` | `<client-subnet>` |
| Client gateway / SVI IP | `10.20.20.1` | `<client-gateway-ip>` |
| Scope ID | `10.20.20.0` | `<scope-id>` |
| Scope range | `10.20.20.50-10.20.20.250` | `<scope-range>` |
| Subnet mask | `255.255.255.0` | `<subnet-mask>` |
| Option 003 router | `10.20.20.1` | `<router-option>` |
| Option 006 DNS servers | `10.10.10.10,10.10.10.11` | `<dns-server-list>` |
| Option 015 DNS suffix | `corp.local` | `<domain-fqdn>` |
| Network relay device | `CORE-SW1` | `<relay-device>` |
| Relay interface | `Vlan20` | `<relay-interface>` |
| Relay command | `ip helper-address 10.10.10.5` | `<ip-helper-command>` |
| Routing required | Client subnet to DHCP server subnet | `<routing-requirement>` |
| Firewall required | UDP 67/68 between relay and DHCP server | `<firewall-requirement>` |
| Test client | `WIN11-01` | `<test-client>` |
| Expected client IP | `10.20.20.50+` | `<expected-client-ip>` |
| Evidence path | `C:\DHCP-Relay-IP-Helper` | `<evidence-path>` |
| Rollback stance | Remove helper only after confirming alternate DHCP path | `<rollback-plan>` |
| Next workbook | `08_Configure_DHCP_DNS_Dynamic_Updates.md` | `<next-task>` |

# Configure_DHCP_Relay_And_IP_Helper_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DHCP role installed | DHCP Server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 3 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | DHCP service is Running |
| 4 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 5 | Confirm DHCP server authorization | DHCP Server | `Get-DhcpServerInDC` | Server is authorized in AD |
| 6 | Confirm remote subnet scope exists | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Scope for client subnet exists |
| 7 | Confirm scope is active | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>" \| Select State` | Scope is Active |
| 8 | Confirm scope options | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Router, DNS, and suffix options are correct |
| 9 | Confirm DHCP server route to client subnet | DHCP Server | `Test-NetConnection "<client-gateway-ip>"` | Server can reach remote subnet gateway |
| 10 | Confirm relay device has route to DHCP server | Network Device | `ping <dhcp-server-ip>` | Relay device can reach DHCP server |
| 11 | Confirm relay interface is client gateway | Network Device | `show ip interface brief` | Client VLAN/SVI is up/up |
| 12 | Configure IP helper on client gateway interface | Network Device | `ip helper-address <dhcp-server-ip>` | DHCP broadcasts are relayed to DHCP server |
| 13 | Confirm helper configuration | Network Device | `show running-config interface <interface>` | Helper address appears under client VLAN interface |
| 14 | Confirm firewall permits DHCP relay path | Firewall / Network | Allow UDP 67/68 between relay and server | DHCP relay traffic is not blocked |
| 15 | Release client lease | Client | `ipconfig /release` | Client releases old lease |
| 16 | Renew client lease | Client | `ipconfig /renew` | Client receives lease from remote scope |
| 17 | Validate client IP configuration | Client | `ipconfig /all` | Client receives expected IP, gateway, DNS, suffix |
| 18 | Validate server-side lease | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Client lease appears in remote subnet scope |
| 19 | Validate DNS resolution from client | Client | `Resolve-DnsName "<domain-fqdn>"` | Client can resolve internal domain |
| 20 | Validate gateway reachability from client | Client | `Test-NetConnection "<client-gateway-ip>"` | Client can reach gateway |
| 21 | Validate DHCP server events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100` | DHCP lease events are visible |
| 22 | Export validation evidence | DHCP Server / Client / Network | Run validation skeletons | Evidence files are saved |
| 23 | Document final state | Operator | `Record scope, relay device, interface, helper IP, test client, and lease result` | Workbook record is complete |

# Configure_DHCP_Relay_And_IP_Helper_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: capture DHCP server, scope, lease, route, and service state before relay configuration.

$ScopeId = "10.20.20.0"
$ClientGatewayIp = "10.20.20.1"
$DhcpServerIp = "10.10.10.5"
$EvidencePath = "C:\DHCP-Relay-IP-Helper"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Capture DHCP server identity.
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

# Confirm AD authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Capture all scopes.
Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\dhcp-scopes-before.txt"

Get-DhcpServerv4Scope |
  Export-Csv "$EvidencePath\dhcp-scopes-before.csv" -NoTypeInformation

# Capture target remote subnet scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-remote-scope-before.txt"

# Capture scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-options-before.txt"

# Capture leases before relay validation.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-leases-before.txt"

# Capture scope statistics.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-statistics-before.txt"

# Confirm DHCP server network configuration.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\dhcp-server-net-ip-configuration.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\dhcp-server-ipconfig-all.txt"

# Test reachability to client subnet gateway.
Test-NetConnection $ClientGatewayIp |
  Tee-Object "$EvidencePath\test-client-gateway-reachability.txt"
```

# Configure_DHCP_Relay_And_IP_Helper_DHCP_Server_Scope_Readiness_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: verify that the DHCP server has an active scope for the remote client subnet.
# The relay device GIADDR must match the subnet of this scope.

$ScopeId = "10.20.20.0"
$ScopeName = "Workstations-VLAN20"
$StartRange = "10.20.20.50"
$EndRange = "10.20.20.250"
$SubnetMask = "255.255.255.0"
$Router = "10.20.20.1"
$DnsServers = @("10.10.10.10","10.10.10.11")
$DomainName = "corp.local"
$EvidencePath = "C:\DHCP-Relay-IP-Helper"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-server-scope-readiness-transcript.txt"

Import-Module DhcpServer

# Check whether remote subnet scope exists.
$Scope = Get-DhcpServerv4Scope -ScopeId $ScopeId -ErrorAction SilentlyContinue

if ($Scope) {
  Write-Output "Scope $ScopeId already exists."
  $Scope |
    Tee-Object "$EvidencePath\remote-scope-existing.txt"
}
else {
  Add-DhcpServerv4Scope `
    -Name $ScopeName `
    -StartRange $StartRange `
    -EndRange $EndRange `
    -SubnetMask $SubnetMask `
    -State Active

  Get-DhcpServerv4Scope -ScopeId $ScopeId |
    Tee-Object "$EvidencePath\remote-scope-after-create.txt"
}

# Configure required scope options.
Set-DhcpServerv4OptionValue `
  -ScopeId $ScopeId `
  -Router $Router `
  -DnsServer $DnsServers `
  -DnsDomain $DomainName

# Confirm scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\remote-scope-options-after-config.txt"

# Confirm scope state and statistics.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\remote-scope-final.txt"

Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\remote-scope-statistics-final.txt"

Stop-Transcript
```

# Configure_DHCP_Relay_And_IP_Helper_Network_Device_IP_Helper_Skeleton
```text
# Run on the Layer 3 gateway for the client subnet.
# Purpose: configure DHCP relay on the interface/SVI that receives DHCP client broadcasts.
# Example shown for Cisco IOS/IOS XE style devices.

enable
configure terminal

interface Vlan20
 description Workstations VLAN 20 default gateway
 ip address 10.20.20.1 255.255.255.0
 ip helper-address 10.10.10.5
 no shutdown
exit

end
write memory

# Verification commands.
show running-config interface Vlan20
show ip interface brief
show ip route 10.10.10.5
ping 10.10.10.5
show ip dhcp relay information
show logging | include DHCP|BOOTP|relay|helper
```

# Configure_DHCP_Relay_And_IP_Helper_Windows_RRAS_DHCP_Relay_Skeleton
```powershell
# Run only if a Windows Server is acting as the router/relay device.
# Purpose: configure DHCP Relay Agent in RRAS.
# Most enterprise networks use IP helper on routers/switches instead.

$DhcpServerIp = "10.10.10.5"
$RelayInterfaceAlias = "Ethernet 2"
$EvidencePath = "C:\DHCP-Relay-IP-Helper\Windows-RRAS-Relay"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture server routing/network state first.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-before.txt"

Get-Service RemoteAccess -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\remoteaccess-service-before.txt"

# Install Remote Access tools if needed.
Install-WindowsFeature RemoteAccess -IncludeManagementTools |
  Tee-Object "$EvidencePath\install-remoteaccess-output.txt"

# RRAS DHCP Relay Agent is commonly managed through RRAS console or netsh.
# Use this only in lab or when Windows Server is intentionally used as the relay router.

# Example netsh-style flow:
netsh routing ip relay install

netsh routing ip relay add server $DhcpServerIp

netsh routing ip relay add interface "$RelayInterfaceAlias"

# Show relay configuration.
netsh routing ip relay show global |
  Tee-Object "$EvidencePath\rras-dhcp-relay-global.txt"

netsh routing ip relay show interface |
  Tee-Object "$EvidencePath\rras-dhcp-relay-interfaces.txt"

netsh routing ip relay show server |
  Tee-Object "$EvidencePath\rras-dhcp-relay-servers.txt"
```

# Configure_DHCP_Relay_And_IP_Helper_Firewall_And_Path_Validation_Skeleton
```powershell
# Run from DHCP server and from an admin host where possible.
# Purpose: validate routed path and firewall allowance for DHCP relay behavior.

$DhcpServerIp = "10.10.10.5"
$ClientGatewayIp = "10.20.20.1"
$ClientSubnetTestIp = "10.20.20.50"
$EvidencePath = "C:\DHCP-Relay-IP-Helper\Path-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture local routing table.
route print |
  Tee-Object "$EvidencePath\route-print.txt"

Get-NetRoute |
  Tee-Object "$EvidencePath\net-routes.txt"

# Test reachability to relay/client gateway.
Test-NetConnection $ClientGatewayIp |
  Tee-Object "$EvidencePath\test-client-gateway.txt"

# Test DNS/DHCP server local IP.
Test-NetConnection $DhcpServerIp |
  Tee-Object "$EvidencePath\test-dhcp-server-ip.txt"

# TCP 67 is not a perfect DHCP test because DHCP uses UDP,
# but this can still expose basic firewall/path issues.
Test-NetConnection $DhcpServerIp -Port 67 |
  Tee-Object "$EvidencePath\test-dhcp-server-tcp-67.txt"

# Windows firewall relevant rules on DHCP server.
Get-NetFirewallRule |
  Where-Object {$_.DisplayName -like "*DHCP*" -or $_.DisplayGroup -like "*DHCP*"} |
  Tee-Object "$EvidencePath\windows-firewall-dhcp-rules.txt"

# DHCP server listening endpoint check.
netstat -ano |
  findstr ":67" |
  Tee-Object "$EvidencePath\netstat-dhcp-ports.txt"

# DHCP service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-state-path-validation.txt"
```

# Configure_DHCP_Relay_And_IP_Helper_Client_Lease_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a client in the relayed subnet.
# Purpose: prove the client receives a DHCP lease through relay from the correct remote scope.

$ExpectedSubnet = "10.20.20."
$ExpectedGateway = "10.20.20.1"
$ExpectedDnsServer1 = "10.10.10.10"
$ExpectedDnsServer2 = "10.10.10.11"
$DomainFqdn = "corp.local"
$EvidencePath = "C:\DHCP-Relay-Client-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture client identity and initial IP state.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$EvidencePath\computer-domain-state.txt"

Get-NetAdapter |
  Tee-Object "$EvidencePath\net-adapters-before.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-before.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-before.txt"

# Release and renew lease.
ipconfig /release |
  Tee-Object "$EvidencePath\ipconfig-release.txt"

Start-Sleep -Seconds 5

ipconfig /renew |
  Tee-Object "$EvidencePath\ipconfig-renew.txt"

# Capture final IP state.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-after.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-after.txt"

# Validate gateway.
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

# DHCP client events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-client-admin-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-client-operational-events.txt"
```

# Configure_DHCP_Relay_And_IP_Helper_Server_Lease_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server after client lease renewal.
# Purpose: validate that the remote client lease appears in the correct DHCP scope.

$ScopeId = "10.20.20.0"
$ClientMac = "00-11-22-33-44-55"
$ExpectedClientIp = "10.20.20.50"
$EvidencePath = "C:\DHCP-Relay-IP-Helper\Server-Lease-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm scope state.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-state.txt"

# Confirm scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-options.txt"

# Confirm leases in remote scope.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\leases-after-client-renew.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\leases-after-client-renew.csv" -NoTypeInformation

# Check lease by client MAC if known.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ClientId $ClientMac -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-by-client-mac.txt"

# Check lease by expected IP if known.
Get-DhcpServerv4Lease -ScopeId $ScopeId -IPAddress $ExpectedClientIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-by-expected-ip.txt"

# Confirm utilization.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-statistics-after-relay-validation.txt"

# Confirm DHCP server audit/log event availability.
Get-DhcpServerAuditLog |
  Tee-Object "$EvidencePath\dhcp-audit-log-settings.txt"
```

# Configure_DHCP_Relay_And_IP_Helper_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: collect DHCP Server events related to relayed leases, scope use, authorization, and failures.

$EvidencePath = "C:\DHCP-Relay-IP-Helper\Events"
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
    $_.Message -like "*scope*" -or
    $_.Message -like "*lease*"
  } |
  Tee-Object "$EvidencePath\system-dhcp-events-last-24h.txt"

# Final DHCP service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"
```

# Configure_DHCP_Relay_And_IP_Helper_Verification_Commands
```powershell
# DHCP server role and service.
Get-WindowsFeature DHCP
Get-Service DHCPServer

# DHCP module.
Import-Module DhcpServer

# Authorization.
Get-DhcpServerInDC

# Scope readiness.
Get-DhcpServerv4Scope
Get-DhcpServerv4Scope -ScopeId "<scope-id>"
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"

# Create remote subnet scope if missing.
Add-DhcpServerv4Scope `
  -Name "<scope-name>" `
  -StartRange "<start-ip>" `
  -EndRange "<end-ip>" `
  -SubnetMask "<subnet-mask>" `
  -State Active

# Configure remote subnet options.
Set-DhcpServerv4OptionValue `
  -ScopeId "<scope-id>" `
  -Router "<client-gateway-ip>" `
  -DnsServer "<dns-server-1>","<dns-server-2>" `
  -DnsDomain "<domain-fqdn>"

# Lease validation.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -ClientId "<client-mac>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>" -IPAddress "<client-ip>"

# DHCP server path checks.
Test-NetConnection "<client-gateway-ip>"
Test-NetConnection "<dhcp-server-ip>" -Port 67
route print

# Client validation.
ipconfig /release
ipconfig /renew
ipconfig /all
Get-NetIPConfiguration
Test-NetConnection "<client-gateway-ip>"
Test-NetConnection "<dns-server-ip>" -Port 53
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"

# DHCP server events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DHCP*" -or $_.Message -like "*DHCP*"}

# DHCP client events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
```

# Configure_DHCP_Relay_And_IP_Helper_Network_Device_Verification_Commands
```text
# Cisco IOS / IOS XE verification examples.

show running-config interface <relay-interface>
show ip interface brief
show ip route <dhcp-server-ip>
ping <dhcp-server-ip>
traceroute <dhcp-server-ip>

# Relay/helper visibility.
show running-config | include ip helper-address
show running-config interface <relay-interface> | include helper
show ip dhcp relay information
show logging | include DHCP|BOOTP|relay|helper

# Example expected interface config.
interface Vlan20
 description Workstations VLAN 20 default gateway
 ip address 10.20.20.1 255.255.255.0
 ip helper-address 10.10.10.5
 no shutdown
```

# Configure_DHCP_Relay_And_IP_Helper_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| IP helper added to wrong interface | Remove helper from wrong interface | Clients on that VLAN stop relaying DHCP to that server |
| Wrong DHCP server IP configured | Replace helper address with correct DHCP server IP | Clients may fail DHCP until corrected |
| Duplicate helper addresses added | Remove unneeded helper address | DHCP requests may reach unintended servers |
| Remote subnet scope created incorrectly | Deactivate or remove scope after confirming no active leases | Clients may lose lease availability |
| Scope router option wrong | Correct option 003 with `Set-DhcpServerv4OptionValue` | Clients may receive wrong default gateway |
| Scope DNS option wrong | Correct option 006 with `Set-DhcpServerv4OptionValue` | Clients may fail name resolution |
| Windows RRAS relay added accidentally | Remove relay interface/server from RRAS DHCP Relay Agent | Windows relay stops forwarding DHCP |
| Firewall rule opened too broadly | Restrict UDP 67/68 to approved relay and DHCP server paths | DHCP relay may fail if overly restricted |
| Client lease renewed during test | No rollback usually needed | Client may keep new lease until expiry/release |
| Evidence folder created | `Remove-Item C:\DHCP-Relay-IP-Helper -Recurse -Force` | Deletes validation evidence |

# Configure_DHCP_Relay_And_IP_Helper_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client gets APIPA | Relay missing, DHCP unreachable, scope missing, or VLAN issue | Client `ipconfig /all`; relay interface config | Rebuilding DHCP server |
| Client does not receive lease across VLAN | Missing `ip helper-address` | `show running-config interface <client-gateway-interface>` | Changing DNS options |
| Client receives lease on local VLAN but not remote VLAN | Relay/routing path issue | Helper address, route to DHCP server, firewall | Recreating scope blindly |
| DHCP server sees no lease request | Relay not forwarding or firewall blocking | Network device logs and packet path | Editing reservations |
| DHCP server receives request but no offer | Missing/inactive scope for GIADDR subnet | `Get-DhcpServerv4Scope -ScopeId <remote-subnet>` | Changing helper address |
| Client receives wrong subnet IP | Wrong GIADDR/interface, wrong scope, or misconfigured relay | Relay interface IP and DHCP scope ID | Changing client manually |
| Client receives IP but wrong gateway | Scope option 003 wrong | `Get-DhcpServerv4OptionValue -ScopeId <scope-id>` | Editing relay config |
| Client receives IP but wrong DNS | Scope option 006 wrong | Scope options | Changing firewall |
| Relay device cannot ping DHCP server | Routing/firewall issue | `show ip route <dhcp-server-ip>` and ACLs | Rebuilding DHCP scope |
| DHCP server cannot reach client gateway | Routing issue from server side | `Test-NetConnection <client-gateway-ip>` | Changing client adapter |
| Multiple DHCP servers answer | Rogue DHCP or duplicate helper | DHCP server events and network relay config | Expanding scope |
| Client gets old lease | Client cache/lease not released | `ipconfig /release`; `ipconfig /renew` | Changing helper immediately |
| Client MAC not visible in expected scope | Request hitting wrong scope or server | Lease inventory across scopes | Creating reservation first |
| Scope exhausted | No available addresses | `Get-DhcpServerv4ScopeStatistics` | Changing relay first |
| Exclusions too broad | Address pool blocked by exclusions | `Get-DhcpServerv4ExclusionRange` | Removing helper |
| Reservation conflicts | Reserved IP already leased/wrong MAC | `Get-DhcpServerv4Reservation`; `Get-DhcpServerv4Lease` | Recreating VLAN |
| Firewall allows ping but not DHCP | UDP 67/68 blocked | Firewall/ACL rules for DHCP relay path | Trusting ICMP only |
| Option 82 breaks assignment | Relay information policy mismatch | Relay Option 82 behavior and DHCP policy | Removing scope |
| Wireless clients fail only | WLAN/VLAN mapping or DHCP snooping issue | SSID VLAN and relay path | Editing DHCP server first |
| DHCP snooping drops replies | Switch security feature | DHCP snooping trust/uplink config | Changing Windows DHCP settings |

# Configure_DHCP_Relay_And_IP_Helper_Related_Labs
| Lab                                           | Related Workbook                                      | Skill Proven                                     |
| --------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------ |
| Install DHCP Server role and management tools | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | DHCP role baseline                               |
| Authorize DHCP Server in Active Directory     | `02_Authorize_DHCP_Server_In_Active_Directory.md`     | AD-authorized DHCP service                       |
| Create DHCPv4 scope                           | `03_Create_DHCPv4_Scope.md`                           | Remote subnet scope creation                     |
| Configure DHCPv4 scope options                | `04_Configure_DHCPv4_Scope_Options.md`                | Gateway, DNS, and domain option delivery         |
| Configure DHCPv4 exclusions and reservations  | `05_Configure_DHCPv4_Exclusions_And_Reservations.md`  | Lease pool protection before relay testing       |
| Validate DHCP client lease assignment         | `06_Validate_DHCP_Client_Lease_Assignment.md`         | Client-side lease validation                     |
| Configure DHCP relay and IP helper            | `07_Configure_DHCP_Relay_And_IP_Helper.md`            | Cross-subnet DHCP relay operation                |
| Configure DHCP DNS dynamic updates            | `08_Configure_DHCP_DNS_Dynamic_Updates.md`            | DNS registration after relayed lease assignment  |
| Troubleshoot DHCP client lease failures       | `12_Troubleshoot_DHCP_Client_Lease_Failures.md`       | Relay, scope, lease, firewall, and client triage |