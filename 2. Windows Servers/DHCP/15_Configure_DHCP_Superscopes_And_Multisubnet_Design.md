15_Configure_DHCP_Superscopes_And_Multisubnet_Design.md
# 15_Configure_DHCP_Superscopes_And_Multisubnet_Design

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Index
15_Configure_DHCP_Superscopes_And_Multisubnet_Design.md
Configure_DHCP_Superscopes_And_Multisubnet_Design
Configure_DHCP_Superscopes_And_Multisubnet_Design_Source_Basis
Configure_DHCP_Superscopes_And_Multisubnet_Design_Mental_Model
Configure_DHCP_Superscopes_And_Multisubnet_Design_Design_Decision_Table
Configure_DHCP_Superscopes_And_Multisubnet_Design_Planning_Table
Configure_DHCP_Superscopes_And_Multisubnet_Design_Configuration_Checklist
Configure_DHCP_Superscopes_And_Multisubnet_Design_Precheck_Skeleton
Configure_DHCP_Superscopes_And_Multisubnet_Design_Create_Primary_Scope_Skeleton
Configure_DHCP_Superscopes_And_Multisubnet_Design_Create_Secondary_Scope_In_Superscope_Skeleton
Configure_DHCP_Superscopes_And_Multisubnet_Design_Attach_Existing_Scope_To_Superscope_Skeleton
Configure_DHCP_Superscopes_And_Multisubnet_Design_Configure_Per_Scope_Options_Skeleton
Configure_DHCP_Superscopes_And_Multisubnet_Design_Multisubnet_Gateway_Skeleton
Configure_DHCP_Superscopes_And_Multisubnet_Design_Client_Test_Skeleton
Configure_DHCP_Superscopes_And_Multisubnet_Design_Verification_Commands
Configure_DHCP_Superscopes_And_Multisubnet_Design_Rollback
Configure_DHCP_Superscopes_And_Multisubnet_Design_Failure_Checks
Configure_DHCP_Superscopes_And_Multisubnet_Design_Related_Labs

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DhcpServerv4Scope | Creating IPv4 scopes and placing a new scope into a superscope with `-SuperscopeName` |
| Microsoft Learn | Set-DhcpServerv4Scope | Modifying existing IPv4 scope properties and attaching an existing scope to a superscope with `-SuperscopeName` |
| Microsoft Learn | Get-DhcpServerv4Scope | Validating scope configuration, state, ranges, and superscope membership |
| Microsoft Learn | Remove-DhcpServerv4Scope | Removing incorrect lab scopes or rollback scopes |
| Microsoft Learn | Get-DhcpServerv4ScopeStatistics | Checking per-scope utilization and address pressure |
| Microsoft Learn | Set-DhcpServerv4OptionValue | Setting per-scope router, DNS, DNS suffix, and other options |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Verifying scope-level option values |
| Windows Server DHCP operational practice | DHCP Manager / PowerShell / Event Viewer / DHCP audit logs | Validating server behavior, leases, options, and client impact |
| Network design practice | Router secondary IPs, VLAN gateways, relay/IP-helper design | Distinguishing true superscope use from routed multi-VLAN DHCP relay design |

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Superscope | Logical grouping of multiple IPv4 scopes on one DHCP server |
| Multisubnet physical segment | One broadcast domain carrying more than one logical IPv4 subnet |
| Primary scope | Existing or first scope for the original subnet on the physical segment |
| Secondary scope | Additional scope for an added logical subnet on the same physical segment |
| Address exhaustion use case | Superscope can help when the original subnet is full but the physical segment remains the same |
| Same L2 boundary | Superscopes make sense when clients are on the same broadcast domain |
| Routed VLAN boundary | Superscopes are usually not the fix; use one scope per VLAN and DHCP relay/IP helper |
| Gateway requirement | Each logical subnet needs a usable default gateway IP in that subnet |
| Router secondary IP | Router/SVI can hold additional gateway IPs for secondary logical subnets on the same L2 segment |
| Per-scope options | Each scope in the superscope still needs its own router, DNS, DNS suffix, and lease settings |
| Relay agent behavior | DHCP relay uses gateway address / relay info to select the right scope for routed networks |
| Bad design smell | Using superscopes to avoid proper VLAN/routing design usually creates operational confusion |
| First rule | Use superscopes only for multiple logical IPv4 subnets on the same physical broadcast domain |
| Second rule | For separate VLANs, do not use superscope as the primary design; use relay/IP helper and separate scopes |

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Design_Decision_Table
| Scenario | Correct Design | Superscope? | Reason |
|---|---|---|---|
| One VLAN, one subnet | One DHCP scope | No | Normal scope is enough |
| One VLAN, original subnet exhausted, same L2 must remain | Add second logical subnet and group scopes in superscope | Yes | Multiple logical subnets exist on one broadcast domain |
| Multiple routed VLANs | One scope per VLAN plus DHCP relay/IP helper | No | Relay selects scope by routed interface/giaddr |
| Remote branch subnet | Scope plus relay/IP helper | No | Branch is routed, not same L2 segment |
| Voice VLAN and data VLAN | Separate scopes, relay/IP helper per VLAN | No | Different VLANs should stay separate |
| Temporary lab with two subnets on one flat switch | Superscope can be acceptable | Yes | Same L2, multiple logical IPv4 subnets |
| Printer subnet carved out of same flat LAN | Superscope only if it truly shares same L2 | Maybe | Better long-term design is usually VLAN separation |
| Trying to fix clients getting wrong subnet | Fix VLAN/relay/scope selection | No | Superscope can make this worse |
| Trying to expand address pool inside same subnet | Expand scope range or subnet if possible | No | Superscope is for multiple subnets, not simple range expansion |
| Production redesign opportunity exists | VLAN and route properly | Usually no | Cleaner control, security, and troubleshooting |

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server | `DHCP1` | `<dhcp-server>` |
| Superscope name | `SS-LAN20-EXPANSION` | `<superscope-name>` |
| Physical segment | `Access-LAN20` | `<physical-segment>` |
| VLAN ID | `20` | `<vlan-id>` |
| Primary subnet | `10.10.20.0/24` | `<primary-subnet>` |
| Primary scope ID | `10.10.20.0` | `<primary-scope-id>` |
| Primary range | `10.10.20.50-10.10.20.199` | `<primary-range>` |
| Primary gateway | `10.10.20.1` | `<primary-router>` |
| Secondary subnet | `10.10.21.0/24` | `<secondary-subnet>` |
| Secondary scope ID | `10.10.21.0` | `<secondary-scope-id>` |
| Secondary range | `10.10.21.50-10.10.21.199` | `<secondary-range>` |
| Secondary gateway | `10.10.21.1` | `<secondary-router>` |
| Subnet mask | `255.255.255.0` | `<subnet-mask>` |
| DNS servers | `10.10.10.10`, `10.10.10.11` | `<dns-servers>` |
| DNS domain | `corp.local` | `<dns-domain>` |
| Lease duration | `8.00:00:00` | `<lease-duration>` |
| Scope creation state | `Inactive first` | `<initial-state>` |
| Gateway device | `CoreSW1` | `<gateway-device>` |
| Gateway interface | `Vlan20` | `<gateway-interface>` |
| Test client | `WIN11-01` | `<test-client>` |
| Client interface | `Ethernet` | `<client-interface>` |
| Evidence path | `C:\DHCPPrep\dhcp-superscope-multisubnet` | `<evidence-path>` |

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm this is same-L2 multisubnet design | Operator | Review VLAN, gateway, and subnet design | Superscope is justified |
| 2 | Confirm this is not a routed VLAN relay problem | Operator / Network device | Check VLAN boundaries and DHCP relay path | Superscope is not misused |
| 3 | Create evidence folder | DHCP server | `New-Item -ItemType Directory -Force -Path C:\DHCPPrep\dhcp-superscope-multisubnet` | Evidence path exists |
| 4 | Import DHCP module | DHCP server | `Import-Module DhcpServer` | DHCP cmdlets available |
| 5 | Confirm DHCP service | DHCP server | `Get-Service DHCPServer` | DHCP service is running |
| 6 | Confirm server authorization | DHCP server or DC | `Get-DhcpServerInDC` | DHCP server is authorized |
| 7 | Capture existing scopes | DHCP server | `Get-DhcpServerv4Scope -ComputerName <dhcp-server>` | Existing scope map is known |
| 8 | Capture existing scope statistics | DHCP server | `Get-DhcpServerv4ScopeStatistics -ComputerName <dhcp-server>` | Utilization pressure is known |
| 9 | Confirm primary scope exists | DHCP server | `Get-DhcpServerv4Scope -ComputerName <dhcp-server> -ScopeId <primary-scope-id>` | Primary scope exists |
| 10 | Confirm secondary scope does not already exist | DHCP server | `Get-DhcpServerv4Scope -ComputerName <dhcp-server> -ScopeId <secondary-scope-id>` | Duplicate scope is not present |
| 11 | Confirm gateway has primary subnet IP | Router / L3 switch | `show running-config interface <gateway-interface>` | Primary gateway exists |
| 12 | Configure secondary gateway IP | Router / L3 switch | `ip address <secondary-router> <subnet-mask> secondary` | Router can route for secondary subnet |
| 13 | Add secondary DHCP scope into superscope | DHCP server | `Add-DhcpServerv4Scope -ComputerName <dhcp-server> -Name <scope-name> -StartRange <start> -EndRange <end> -SubnetMask <mask> -State InActive -SuperscopeName <superscope-name>` | Secondary scope exists and is grouped |
| 14 | Attach existing primary scope to superscope if needed | DHCP server | `Set-DhcpServerv4Scope -ComputerName <dhcp-server> -ScopeId <primary-scope-id> -SuperscopeName <superscope-name>` | Primary scope is grouped |
| 15 | Configure primary scope options | DHCP server | `Set-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <primary-scope-id> -Router <primary-router> -DnsServer <dns-servers> -DnsDomain <dns-domain>` | Primary clients get correct options |
| 16 | Configure secondary scope options | DHCP server | `Set-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <secondary-scope-id> -Router <secondary-router> -DnsServer <dns-servers> -DnsDomain <dns-domain>` | Secondary clients get correct options |
| 17 | Validate scope membership | DHCP server | `Get-DhcpServerv4Scope -ComputerName <dhcp-server> \| Select ScopeId,Name,State,SuperscopeName` | Both scopes show intended superscope |
| 18 | Activate secondary scope | DHCP server | `Set-DhcpServerv4Scope -ComputerName <dhcp-server> -ScopeId <secondary-scope-id> -State Active` | Secondary scope can serve leases |
| 19 | Renew a test client | Client | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Client receives valid lease and options |
| 20 | Validate lease on DHCP server | DHCP server | `Get-DhcpServerv4Lease -ComputerName <dhcp-server> -ScopeId <scope-id>` | Lease is visible |
| 21 | Validate routing from both subnets | Client / router | `Test-NetConnection <gateway>`; `show ip route` | Clients can route correctly |
| 22 | Capture final evidence | DHCP server and client | Export scopes, options, statistics, leases, and client output | Evidence is preserved |

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Precheck_Skeleton
~~~powershell
# Run on the DHCP server or management host with RSAT DHCP tools.

$DhcpServer = "DHCP1"
$PrimaryScopeId = "10.10.20.0"
$SecondaryScopeId = "10.10.21.0"
$EvidencePath = "C:\DHCPPrep\dhcp-superscope-multisubnet"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm DHCP service.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\precheck-dhcp-service.txt"

# Confirm AD authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\precheck-authorized-dhcp-servers.txt"

# Capture all scopes.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer |
  Select-Object ScopeId,Name,State,StartRange,EndRange,SubnetMask,LeaseDuration,SuperscopeName |
  Tee-Object "$EvidencePath\precheck-all-scopes.txt"

# Capture all scope statistics.
Get-DhcpServerv4ScopeStatistics `
  -ComputerName $DhcpServer |
  Tee-Object "$EvidencePath\precheck-all-scope-statistics.txt"

# Confirm primary scope exists.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-primary-scope.txt"

# Confirm secondary scope does not already exist.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-secondary-scope-existing.txt"

# Capture primary scope options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-primary-scope-options.txt"

# Capture leases.
Get-DhcpServerv4Lease `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId `
  -AllLeases `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-primary-scope-leases.txt"
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Create_Primary_Scope_Skeleton
~~~powershell
# Run only if the primary scope does not already exist.
# Most production superscope work starts with an existing primary scope.

$DhcpServer = "DHCP1"
$PrimaryScopeName = "LAN20-PRIMARY-10.10.20.0"
$PrimaryScopeId = "10.10.20.0"
$PrimaryStart = "10.10.20.50"
$PrimaryEnd = "10.10.20.199"
$SubnetMask = "255.255.255.0"
$SuperscopeName = "SS-LAN20-EXPANSION"
$LeaseDuration = New-TimeSpan -Days 8

Import-Module DhcpServer

Add-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -Name $PrimaryScopeName `
  -StartRange $PrimaryStart `
  -EndRange $PrimaryEnd `
  -SubnetMask $SubnetMask `
  -State InActive `
  -LeaseDuration $LeaseDuration `
  -SuperscopeName $SuperscopeName `
  -PassThru

Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId |
  Format-List *
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Create_Secondary_Scope_In_Superscope_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# This creates the secondary logical subnet scope and places it directly into the superscope.

$DhcpServer = "DHCP1"
$SecondaryScopeName = "LAN20-SECONDARY-10.10.21.0"
$SecondaryScopeId = "10.10.21.0"
$SecondaryStart = "10.10.21.50"
$SecondaryEnd = "10.10.21.199"
$SubnetMask = "255.255.255.0"
$SuperscopeName = "SS-LAN20-EXPANSION"
$LeaseDuration = New-TimeSpan -Days 8

Import-Module DhcpServer

# Create secondary scope inactive first.
Add-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -Name $SecondaryScopeName `
  -StartRange $SecondaryStart `
  -EndRange $SecondaryEnd `
  -SubnetMask $SubnetMask `
  -State InActive `
  -LeaseDuration $LeaseDuration `
  -SuperscopeName $SuperscopeName `
  -PassThru

# Validate secondary scope and superscope membership.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId |
  Format-List *

# Do not activate until gateway and per-scope options are configured.
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Attach_Existing_Scope_To_Superscope_Skeleton
~~~powershell
# Run when the primary scope already exists and needs to be grouped with the secondary scope.

$DhcpServer = "DHCP1"
$PrimaryScopeId = "10.10.20.0"
$SecondaryScopeId = "10.10.21.0"
$SuperscopeName = "SS-LAN20-EXPANSION"

Import-Module DhcpServer

# Attach existing primary scope to superscope.
Set-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId `
  -SuperscopeName $SuperscopeName `
  -PassThru

# Confirm both member scopes.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer |
  Where-Object {
    $_.ScopeId -eq $PrimaryScopeId -or
    $_.ScopeId -eq $SecondaryScopeId
  } |
  Select-Object ScopeId,Name,State,StartRange,EndRange,SubnetMask,SuperscopeName |
  Format-Table -AutoSize
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Configure_Per_Scope_Options_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# Each scope in the superscope must have correct options for its own subnet.
# The router/default gateway option is different per logical subnet.

$DhcpServer = "DHCP1"

$PrimaryScopeId = "10.10.20.0"
$PrimaryRouter = "10.10.20.1"

$SecondaryScopeId = "10.10.21.0"
$SecondaryRouter = "10.10.21.1"

$DnsServers = @("10.10.10.10","10.10.10.11")
$DnsDomain = "corp.local"

Import-Module DhcpServer

# Primary scope options.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId `
  -Router $PrimaryRouter `
  -DnsServer $DnsServers `
  -DnsDomain $DnsDomain `
  -PassThru

# Secondary scope options.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId `
  -Router $SecondaryRouter `
  -DnsServer $DnsServers `
  -DnsDomain $DnsDomain `
  -PassThru

# Validate primary options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId

# Validate secondary options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId

# Activate secondary scope after options and gateway are ready.
Set-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId `
  -State Active `
  -PassThru
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Multisubnet_Gateway_Skeleton
~~~text
# Run on the router or L3 switch that is the default gateway for the shared physical segment.
# Example shown with Cisco IOS-style syntax.
# Use only when both logical subnets truly share the same L2 segment.

configure terminal
interface Vlan20
 description Access-LAN20 multisubnet gateway
 ip address 10.10.20.1 255.255.255.0
 ip address 10.10.21.1 255.255.255.0 secondary
 no shutdown
end
write memory

# Verification.
show running-config interface Vlan20
show ip interface Vlan20
show ip route connected
show arp

# If this is actually a routed VLAN design instead of a same-L2 multisubnet design,
# do not use this superscope model.
# Use one scope per VLAN and configure relay/IP helper instead.

configure terminal
interface Vlan20
 ip helper-address <dhcp-server-ip>
end
write memory
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Client_Test_Skeleton
~~~powershell
# Run on a Windows test client on the affected physical segment.

$EvidencePath = "C:\DHCPPrep\dhcp-superscope-multisubnet"
$InterfaceAlias = "Ethernet"
$DnsTestName = "corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture before state.
hostname |
  Tee-Object "$EvidencePath\client-hostname.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-before.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfig-before.txt"

Get-NetIPAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\client-ipv4-before.txt"

Get-NetRoute -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\client-ipv4-routes-before.txt"

# Renew DHCP.
ipconfig /release |
  Tee-Object "$EvidencePath\client-release.txt"

ipconfig /renew |
  Tee-Object "$EvidencePath\client-renew.txt"

# Capture after state.
ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-after.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfig-after.txt"

Get-NetIPAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\client-ipv4-after.txt"

Get-NetRoute -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\client-ipv4-routes-after.txt"

# Test default gateway and DNS after renewal.
$Config = Get-NetIPConfiguration -InterfaceAlias $InterfaceAlias

if ($Config.IPv4DefaultGateway) {
  Test-NetConnection $Config.IPv4DefaultGateway.NextHop |
    Tee-Object "$EvidencePath\client-test-default-gateway.txt"
}

Resolve-DnsName $DnsTestName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-resolve-dns-test.txt"

# DHCP client logs.
Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Client/Admin" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-dhcp-admin-events.txt"
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Verification_Commands
~~~powershell
# DHCP server verification.

Import-Module DhcpServer

Get-Service DHCPServer

Get-DhcpServerInDC

Get-DhcpServerv4Scope `
  -ComputerName <dhcp-server> |
  Select-Object ScopeId,Name,State,StartRange,EndRange,SubnetMask,SuperscopeName

Get-DhcpServerv4Scope `
  -ComputerName <dhcp-server> `
  -ScopeId <primary-scope-id> |
  Format-List *

Get-DhcpServerv4Scope `
  -ComputerName <dhcp-server> `
  -ScopeId <secondary-scope-id> |
  Format-List *

Get-DhcpServerv4ScopeStatistics `
  -ComputerName <dhcp-server>

Get-DhcpServerv4ScopeStatistics `
  -ComputerName <dhcp-server> `
  -ScopeId <primary-scope-id>

Get-DhcpServerv4ScopeStatistics `
  -ComputerName <dhcp-server> `
  -ScopeId <secondary-scope-id>

Get-DhcpServerv4OptionValue `
  -ComputerName <dhcp-server> `
  -ScopeId <primary-scope-id>

Get-DhcpServerv4OptionValue `
  -ComputerName <dhcp-server> `
  -ScopeId <secondary-scope-id>

Get-DhcpServerv4Lease `
  -ComputerName <dhcp-server> `
  -ScopeId <primary-scope-id> `
  -AllLeases

Get-DhcpServerv4Lease `
  -ComputerName <dhcp-server> `
  -ScopeId <secondary-scope-id> `
  -AllLeases

Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

# Client verification.

ipconfig /all
ipconfig /release
ipconfig /renew
ipconfig /all
Get-NetIPConfiguration
Get-NetIPAddress -AddressFamily IPv4
Get-NetRoute -AddressFamily IPv4
Test-NetConnection <default-gateway-ip>
Resolve-DnsName <domain-fqdn>

# Network gateway verification.

show running-config interface <gateway-interface>
show ip interface <gateway-interface>
show ip route connected
show arp
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Rollback
~~~powershell
# Run only the rollback matching the change being reversed.
# Capture current state first.

$DhcpServer = "DHCP1"
$PrimaryScopeId = "10.10.20.0"
$SecondaryScopeId = "10.10.21.0"
$EvidencePath = "C:\DHCPPrep\dhcp-superscope-multisubnet"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current scope state.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer |
  Select-Object ScopeId,Name,State,StartRange,EndRange,SubnetMask,LeaseDuration,SuperscopeName |
  Export-Csv "$EvidencePath\rollback-scopes-before.csv" -NoTypeInformation

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $PrimaryScopeId `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-primary-options-before.csv" -NoTypeInformation

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-secondary-options-before.csv" -NoTypeInformation

Get-DhcpServerv4Lease `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId `
  -AllLeases `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-secondary-leases-before.csv" -NoTypeInformation

# Safest first rollback: deactivate secondary scope.
Set-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $SecondaryScopeId `
  -State InActive `
  -ErrorAction SilentlyContinue `
  -PassThru

# Optional: remove the secondary scope if it was created incorrectly.
# Warning: this deletes the scope and its settings.

# Remove-DhcpServerv4Scope `
#   -ComputerName $DhcpServer `
#   -ScopeId $SecondaryScopeId `
#   -Confirm

# Optional: detach a scope from superscope by setting a blank superscope name.
# Validate behavior in lab before production use.

# Set-DhcpServerv4Scope `
#   -ComputerName $DhcpServer `
#   -ScopeId $PrimaryScopeId `
#   -SuperscopeName "" `
#   -PassThru

# Set-DhcpServerv4Scope `
#   -ComputerName $DhcpServer `
#   -ScopeId $SecondaryScopeId `
#   -SuperscopeName "" `
#   -PassThru

# Client renewal after rollback.
# Run on client.

# ipconfig /release
# ipconfig /renew
# ipconfig /all
~~~

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Superscope design does not fix remote VLAN clients | Superscope used instead of relay/IP-helper | Check VLAN boundaries and routed interfaces | Use one scope per VLAN and configure DHCP relay |
| Clients receive wrong subnet | Same-L2 multisubnet design is confusing scope selection | `ipconfig /all`; DHCP leases; gateway config | Confirm superscope membership and per-scope options |
| Clients receive secondary subnet but cannot route | Missing secondary gateway IP | Router interface config | Add gateway IP for secondary subnet |
| Clients receive secondary IP but wrong gateway | Secondary scope option 003 wrong | `Get-DhcpServerv4OptionValue -ScopeId <secondary-scope-id>` | Set router option to secondary subnet gateway |
| Secondary scope does not lease | Scope inactive | `Get-DhcpServerv4Scope -ScopeId <secondary-scope-id>` | Activate secondary scope |
| Secondary scope not grouped | SuperscopeName missing or typo | `Get-DhcpServerv4Scope | Select ScopeId,SuperscopeName` | Set correct `-SuperscopeName` |
| Scope creation fails | Overlapping range or duplicate scope | `Get-DhcpServerv4Scope` | Correct subnet/range or remove duplicate |
| Existing scope loses expected behavior | Options changed at wrong scope level | Compare primary and secondary option values | Restore correct per-scope options |
| Clients get IP but DNS fails | DNS option missing on one member scope | `Get-DhcpServerv4OptionValue -ScopeId <scope-id>` | Set DNS servers and DNS suffix on each scope |
| Clients stay in original scope only | Original scope still has available addresses | Scope statistics and leases | Expected until original pool pressure changes |
| Scope exhaustion persists | Secondary scope not reachable or not active | Scope statistics and leases | Activate scope, confirm superscope membership, confirm gateway |
| DHCP logs show no client activity | Client packets not reaching DHCP server | DHCP audit logs, VLAN, relay, firewall | Fix network path before changing scopes |
| Clients have duplicate IP conflicts | Static addresses inside DHCP ranges | Bad leases, ARP table, DHCP logs | Exclude static ranges and clean bad leases |
| Superscope name typo creates split grouping | Different SuperscopeName strings | Scope list grouped by SuperscopeName | Standardize name and reassign scope membership |
| Rollback leaves clients with old lease | Client lease not renewed | `ipconfig /all` lease time | Release/renew or wait for lease expiration |
| Removing scope would delete active leases | Scope still serving clients | `Get-DhcpServerv4Lease -ScopeId <scope-id>` | Deactivate first, migrate clients, then remove if safe |
| Multisubnet design grows operationally messy | Flat network design issue | Incident history and troubleshooting complexity | Move toward VLAN segmentation and routed DHCP relay |

# Configure_DHCP_Superscopes_And_Multisubnet_Design_Related_Labs
| Lab                                                        | Relationship                                                                          |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 02_Create_DHCPv4_Scope                                     | Superscope membership is built from normal IPv4 scopes                                |
| 03_Configure_DHCPv4_Scope_Options                          | Each member scope still requires correct router, DNS, and suffix options              |
| 04_Activate_Authorize_And_Verify_DHCP_Server               | Superscope behavior depends on working DHCP service, authorization, and active scopes |
| 05_Configure_DHCPv4_Exclusions_And_Reservations            | Exclusions and reservations still apply inside member scopes                          |
| 07_Configure_DHCP_Relay_And_IP_Helper                      | Shows the design alternative for routed VLANs and remote subnets                      |
| 11_Monitor_DHCP_Leases_Events_And_Scope_Utilization        | Superscope validation depends on scope statistics and lease pressure                  |
| 12_Troubleshoot_DHCP_Client_Lease_Failures                 | Superscope mistakes usually present as wrong lease, no lease, or bad gateway behavior |
| 14_Configure_DHCP_Policies_User_Classes_And_Vendor_Classes | Policies can coexist with scopes, but should not be confused with superscope design   |