13_Configure_DHCPv6_Scopes_And_Options.md
# 13_Configure_DHCPv6_Scopes_And_Options

# Configure_DHCPv6_Scopes_And_Options_Index
13_Configure_DHCPv6_Scopes_And_Options.md
Configure_DHCPv6_Scopes_And_Options
Configure_DHCPv6_Scopes_And_Options_Source_Basis
Configure_DHCPv6_Scopes_And_Options_Mental_Model
Configure_DHCPv6_Scopes_And_Options_Planning_Table
Configure_DHCPv6_Scopes_And_Options_Configuration_Checklist
Configure_DHCPv6_Scopes_And_Options_Precheck_Skeleton
Configure_DHCPv6_Scopes_And_Options_Create_Stateful_Scope_Skeleton
Configure_DHCPv6_Scopes_And_Options_Configure_Exclusions_Skeleton
Configure_DHCPv6_Scopes_And_Options_Configure_Options_Skeleton
Configure_DHCPv6_Scopes_And_Options_Router_RA_Flag_Skeleton
Configure_DHCPv6_Scopes_And_Options_Client_Test_Skeleton
Configure_DHCPv6_Scopes_And_Options_Verification_Commands
Configure_DHCPv6_Scopes_And_Options_Rollback
Configure_DHCPv6_Scopes_And_Options_Failure_Checks
Configure_DHCPv6_Scopes_And_Options_Related_Labs

# Configure_DHCPv6_Scopes_And_Options_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DhcpServerv6Scope | Creating DHCPv6 scopes using prefix, scope name, state, preference, T1, T2, preferred lifetime, and valid lifetime |
| Microsoft Learn | Set-DhcpServerv6Scope | Modifying DHCPv6 scope name, description, state, preference, T1, T2, preferred lifetime, and valid lifetime |
| Microsoft Learn | Get-DhcpServerv6Scope | Validating DHCPv6 scope existence and scope properties |
| Microsoft Learn | Add-DhcpServerv6ExclusionRange | Adding excluded IPv6 address ranges inside a DHCPv6 scope |
| Microsoft Learn | Get-DhcpServerv6ExclusionRange | Validating excluded IPv6 address ranges |
| Microsoft Learn | Set-DhcpServerv6OptionValue | Setting DHCPv6 options at server, prefix/scope, or reservation level |
| Microsoft Learn | Get-DhcpServerv6OptionValue | Verifying DHCPv6 option values at server, prefix/scope, or reservation level |
| Microsoft Learn | Get-DhcpServerv6Lease | Verifying DHCPv6 leases by prefix or IPv6 address |
| RFC 4861 | Router Advertisement M and O flags | Explains why DHCPv6 depends on Router Advertisement signaling for managed address configuration and other configuration |
| RFC 4861 | Default router behavior | Explains why IPv6 default-router behavior comes from Router Advertisements, not a DHCPv6 default gateway option |
| Windows Server operational practice | DHCP Server module, Event Viewer, DHCP audit logs, Windows client `ipconfig /renew6` | End-to-end validation of DHCPv6 server, scope, options, leases, and client behavior |

# Configure_DHCPv6_Scopes_And_Options_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCPv6 scope | IPv6 prefix on the DHCP server that can lease IPv6 addresses or deliver options |
| Prefix | IPv6 subnet boundary used by the DHCPv6 scope, for example `2001:db8:10:20::` |
| Stateful DHCPv6 | DHCPv6 server assigns IPv6 addresses and options |
| Stateless DHCPv6 | Clients use SLAAC for address assignment but DHCPv6 for other options such as DNS |
| Router Advertisement | IPv6 router message that tells clients how to behave on the subnet |
| M flag | Managed address configuration flag, tells clients to use DHCPv6 for addresses |
| O flag | Other configuration flag, tells clients to use DHCPv6 for options such as DNS |
| Default gateway difference | DHCPv6 does not use an IPv4-style router/default-gateway option; IPv6 clients learn default routers through Router Advertisements |
| Preferred lifetime | Time during which a leased IPv6 address is preferred for normal use |
| Valid lifetime | Time during which a leased IPv6 address remains valid |
| T1 | Renewal timer, when client tries to renew with the original DHCPv6 server |
| T2 | Rebind timer, when client tries to renew with any available DHCPv6 server |
| Preference | DHCPv6 server preference used when clients receive replies from multiple DHCPv6 servers |
| DHCPv6 DNS option | DNS recursive server option, commonly configured through `-DnsServer` or option ID 23 |
| DHCPv6 domain search list | Domain search list option, commonly configured through `-DomainSearchList` |
| Exclusion range | IPv6 address range inside the prefix that the DHCPv6 server will not lease |
| First rule | Decide whether the subnet uses stateful DHCPv6, stateless DHCPv6, or SLAAC-only before configuring the scope |

# Configure_DHCPv6_Scopes_And_Options_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server name | `DHCP1` | `<dhcp-server>` |
| DHCP server FQDN | `dhcp1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IPv6 address | `2001:db8:10:10::20` | `<dhcp-server-ipv6>` |
| DHCPv6 prefix | `2001:db8:10:20::` | `<dhcpv6-prefix>` |
| Prefix length | `/64` | `<prefix-length>` |
| Scope name | `VLAN20-CLIENTS-DHCPv6` | `<scope-name>` |
| Scope description | `Stateful DHCPv6 for VLAN20 clients` | `<scope-description>` |
| Scope state at creation | `Inactive` first, then `Active` after validation | `<scope-state>` |
| Addressing model | Stateful DHCPv6 / stateless DHCPv6 / SLAAC-only | `<addressing-model>` |
| RA M flag | Enabled for stateful DHCPv6 | `<managed-flag>` |
| RA O flag | Enabled for stateless option delivery | `<other-config-flag>` |
| Preferred lifetime | `4.00:00:00` | `<preferred-lifetime>` |
| Valid lifetime | `6.00:00:00` | `<valid-lifetime>` |
| T1 renewal time | `2.00:00:00` | `<t1>` |
| T2 rebind time | `3.00:00:00` | `<t2>` |
| DHCPv6 server preference | `0` | `<preference>` |
| Exclusion start | `2001:db8:10:20::1` | `<exclusion-start>` |
| Exclusion end | `2001:db8:10:20::ff` | `<exclusion-end>` |
| DNS recursive servers | `2001:db8:10:10::10`, `2001:db8:10:10::11` | `<dns-servers>` |
| Domain search list | `corp.local` | `<domain-search-list>` |
| Test client | `WIN11-01` | `<test-client>` |
| Client interface | `Ethernet` | `<client-interface>` |
| Evidence path | `C:\DHCPPrep\dhcpv6-scope-options` | `<evidence-path>` |

# Configure_DHCPv6_Scopes_And_Options_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm DHCP role is installed | DHCP server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 2 | Confirm DHCP Server service is running | DHCP server | `Get-Service DHCPServer` | Service is running |
| 3 | Import DHCP module | DHCP server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 4 | Confirm DHCP authorization if domain joined | DHCP server or DC | `Get-DhcpServerInDC` | Expected DHCP server is listed |
| 5 | Confirm IPv6 is enabled on server NIC | DHCP server | `Get-NetAdapterBinding -ComponentID ms_tcpip6` | IPv6 binding is enabled |
| 6 | Confirm server has expected IPv6 address | DHCP server | `Get-NetIPAddress -AddressFamily IPv6` | Expected DHCP server IPv6 address is present |
| 7 | Confirm DHCP server IPv6 bindings | DHCP server | `Get-DhcpServerv6Binding -ComputerName <dhcp-server>` | Correct IPv6 serving interface is enabled |
| 8 | Confirm no duplicate DHCPv6 prefix exists | DHCP server | `Get-DhcpServerv6Scope -ComputerName <dhcp-server>` | Prefix is not already configured |
| 9 | Create DHCPv6 scope as inactive first | DHCP server | `Add-DhcpServerv6Scope -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix> -Name <scope-name> -State Inactive` | Scope exists but does not serve clients yet |
| 10 | Set scope lifetimes and preference | DHCP server | `Set-DhcpServerv6Scope -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix> -PreferredLifeTime <preferred-lifetime> -ValidLifeTime <valid-lifetime> -T1 <t1> -T2 <t2> -Preference <preference>` | Scope timers and preference are configured |
| 11 | Add exclusions if required | DHCP server | `Add-DhcpServerv6ExclusionRange -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix> -StartRange <exclusion-start> -EndRange <exclusion-end>` | Infrastructure/static IPv6 range is not leased |
| 12 | Configure scope-level DNS servers | DHCP server | `Set-DhcpServerv6OptionValue -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix> -DnsServer <dns-servers>` | Scope delivers DNS recursive server option |
| 13 | Configure scope-level domain search list | DHCP server | `Set-DhcpServerv6OptionValue -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix> -DomainSearchList <domain-search-list>` | Scope delivers domain search list |
| 14 | Validate scope object | DHCP server | `Get-DhcpServerv6Scope -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix>` | Scope exists with expected properties |
| 15 | Validate exclusions | DHCP server | `Get-DhcpServerv6ExclusionRange -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix>` | Exclusions match plan |
| 16 | Validate options | DHCP server | `Get-DhcpServerv6OptionValue -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix>` | DNS and domain options match plan |
| 17 | Configure router RA flags | Router / L3 gateway | `ipv6 nd managed-config-flag` or `ipv6 nd other-config-flag` | Clients know whether to use DHCPv6 for addresses or options |
| 18 | Activate DHCPv6 scope | DHCP server | `Set-DhcpServerv6Scope -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix> -State Active` | Scope is ready to serve |
| 19 | Renew IPv6 on test client | Client | `ipconfig /release6` then `ipconfig /renew6` | Client receives expected IPv6 behavior |
| 20 | Verify client IPv6 config | Client | `ipconfig /all`; `Get-NetIPConfiguration`; `Get-NetIPAddress -AddressFamily IPv6` | Client receives address/options as expected |
| 21 | Verify DHCPv6 lease | DHCP server | `Get-DhcpServerv6Lease -ComputerName <dhcp-server> -Prefix <dhcpv6-prefix>` | Client lease appears if using stateful DHCPv6 |
| 22 | Verify DNS behavior | Client | `Resolve-DnsName <domain-fqdn>` | DNS resolution works using expected IPv6 DNS servers |
| 23 | Capture evidence | DHCP server and client | Export scope, options, leases, client output, and logs | Build evidence is preserved |

# Configure_DHCPv6_Scopes_And_Options_Precheck_Skeleton
~~~powershell
# Run on the DHCP server or from an elevated management host with RSAT DHCP tools.

$DhcpServer = "DHCP1"
$Prefix = "2001:db8:10:20::"
$EvidencePath = "C:\DHCPPrep\dhcpv6-scope-options"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm DHCP role and service.
Get-WindowsFeature DHCP |
  Tee-Object "$EvidencePath\precheck-windows-feature-dhcp.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\precheck-dhcp-service.txt"

# Confirm AD authorization if domain joined.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\precheck-dhcp-authorized-servers.txt"

# Confirm IPv6 binding on NICs.
Get-NetAdapterBinding -ComponentID ms_tcpip6 |
  Tee-Object "$EvidencePath\precheck-ipv6-bindings.txt"

# Confirm server IPv6 addresses.
Get-NetIPAddress -AddressFamily IPv6 |
  Tee-Object "$EvidencePath\precheck-server-ipv6-addresses.txt"

# Confirm DHCPv6 bindings.
Get-DhcpServerv6Binding -ComputerName $DhcpServer |
  Tee-Object "$EvidencePath\precheck-dhcpv6-bindings.txt"

# Confirm existing DHCPv6 scopes.
Get-DhcpServerv6Scope -ComputerName $DhcpServer |
  Tee-Object "$EvidencePath\precheck-existing-dhcpv6-scopes.txt"

# Confirm target prefix does not already exist.
Get-DhcpServerv6Scope -ComputerName $DhcpServer -Prefix $Prefix -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-target-prefix-existing.txt"

# Confirm server-level DHCPv6 options.
Get-DhcpServerv6OptionValue -ComputerName $DhcpServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-server-level-dhcpv6-options.txt"

# Check DHCP Server event logs.
Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-dhcp-server-operational-events.txt"
~~~

# Configure_DHCPv6_Scopes_And_Options_Create_Stateful_Scope_Skeleton
~~~powershell
# Run on the DHCP server or from an elevated management host with RSAT DHCP tools.
# This creates a stateful DHCPv6 scope.
# Create inactive first, validate, then activate.

$DhcpServer = "DHCP1"
$Prefix = "2001:db8:10:20::"
$ScopeName = "VLAN20-CLIENTS-DHCPv6"
$ScopeDescription = "Stateful DHCPv6 scope for VLAN20 client subnet"
$PreferredLifetime = New-TimeSpan -Days 4
$ValidLifetime = New-TimeSpan -Days 6
$T1 = New-TimeSpan -Days 2
$T2 = New-TimeSpan -Days 3
$Preference = 0

Import-Module DhcpServer

# Create the scope inactive first.
Add-DhcpServerv6Scope `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -Name $ScopeName `
  -Description $ScopeDescription `
  -State Inactive `
  -PreferredLifeTime $PreferredLifetime `
  -ValidLifeTime $ValidLifetime `
  -T1 $T1 `
  -T2 $T2 `
  -Preference $Preference `
  -PassThru

# Validate the scope.
Get-DhcpServerv6Scope `
  -ComputerName $DhcpServer `
  -Prefix $Prefix |
  Format-List *

# Activate only after options, exclusions, and RA behavior are planned.
Set-DhcpServerv6Scope `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -State Active `
  -PassThru

# Final scope validation.
Get-DhcpServerv6Scope `
  -ComputerName $DhcpServer `
  -Prefix $Prefix |
  Format-List *
~~~

# Configure_DHCPv6_Scopes_And_Options_Configure_Exclusions_Skeleton
~~~powershell
# Run on the DHCP server or from an elevated management host with RSAT DHCP tools.
# Use exclusions for infrastructure/static IPv6 addresses that should not be leased.

$DhcpServer = "DHCP1"
$Prefix = "2001:db8:10:20::"
$ExclusionStart = "2001:db8:10:20::1"
$ExclusionEnd = "2001:db8:10:20::ff"

Import-Module DhcpServer

# Add DHCPv6 exclusion range.
Add-DhcpServerv6ExclusionRange `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -StartRange $ExclusionStart `
  -EndRange $ExclusionEnd `
  -PassThru

# Validate exclusions.
Get-DhcpServerv6ExclusionRange `
  -ComputerName $DhcpServer `
  -Prefix $Prefix
~~~

# Configure_DHCPv6_Scopes_And_Options_Configure_Options_Skeleton
~~~powershell
# Run on the DHCP server or from an elevated management host with RSAT DHCP tools.
# DHCPv6 common options:
# Option 23 = DNS recursive name server
# Option 24 = Domain search list
# Prefer the named parameters when possible.

$DhcpServer = "DHCP1"
$Prefix = "2001:db8:10:20::"
$DnsServers = @(
  "2001:db8:10:10::10",
  "2001:db8:10:10::11"
)
$DomainSearchList = @(
  "corp.local"
)

Import-Module DhcpServer

# Configure DHCPv6 DNS servers and domain search list at scope level.
Set-DhcpServerv6OptionValue `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -DnsServer $DnsServers `
  -DomainSearchList $DomainSearchList `
  -PassThru

# Validate all standard scope-level options.
Get-DhcpServerv6OptionValue `
  -ComputerName $DhcpServer `
  -Prefix $Prefix

# Validate option 23 DNS recursive name server.
Get-DhcpServerv6OptionValue `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -OptionId 23

# Validate option 24 domain search list.
Get-DhcpServerv6OptionValue `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -OptionId 24

# Validate all options, including class-specific values if used.
Get-DhcpServerv6OptionValue `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -All
~~~

# Configure_DHCPv6_Scopes_And_Options_Router_RA_Flag_Skeleton
~~~text
# Run on the router or Layer 3 gateway for the client VLAN.
# DHCPv6 client behavior depends on Router Advertisement flags.
# This is not configured on the Windows DHCP server.

# =========================================================
# Stateful DHCPv6 model
# Client should use DHCPv6 for address assignment and options.
# =========================================================

configure terminal
interface <client-vlan-svi>
 ipv6 address 2001:db8:10:20::1/64
 ipv6 nd managed-config-flag
 ipv6 nd other-config-flag
end
write memory

# =========================================================
# Stateless DHCPv6 model
# Client uses SLAAC for address assignment and DHCPv6 for options.
# =========================================================

configure terminal
interface <client-vlan-svi>
 ipv6 address 2001:db8:10:20::1/64
 no ipv6 nd managed-config-flag
 ipv6 nd other-config-flag
end
write memory

# =========================================================
# SLAAC-only model
# Client uses Router Advertisements only, not DHCPv6.
# Do not use this if DHCPv6 options are required.
# =========================================================

configure terminal
interface <client-vlan-svi>
 ipv6 address 2001:db8:10:20::1/64
 no ipv6 nd managed-config-flag
 no ipv6 nd other-config-flag
end
write memory

# Verification examples.
show running-config interface <client-vlan-svi>
show ipv6 interface <client-vlan-svi>
show ipv6 neighbors
show ipv6 route
~~~

# Configure_DHCPv6_Scopes_And_Options_Client_Test_Skeleton
~~~powershell
# Run on a Windows test client.
# This validates whether the client receives DHCPv6 address/options as expected.

$EvidencePath = "C:\DHCPPrep\dhcpv6-scope-options"
$InterfaceAlias = "Ethernet"
$ExpectedDnsName = "corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture pre-test state.
hostname | Tee-Object "$EvidencePath\client-hostname.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-before.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfiguration-before.txt"

Get-NetIPAddress -AddressFamily IPv6 |
  Tee-Object "$EvidencePath\client-ipv6-addresses-before.txt"

Get-DnsClientServerAddress -AddressFamily IPv6 |
  Tee-Object "$EvidencePath\client-ipv6-dns-before.txt"

Get-NetRoute -AddressFamily IPv6 |
  Tee-Object "$EvidencePath\client-ipv6-routes-before.txt"

# Renew IPv6 configuration.
ipconfig /release6 |
  Tee-Object "$EvidencePath\client-release6.txt"

ipconfig /renew6 |
  Tee-Object "$EvidencePath\client-renew6.txt"

# Capture post-test state.
ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-after.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfiguration-after.txt"

Get-NetIPAddress -AddressFamily IPv6 |
  Tee-Object "$EvidencePath\client-ipv6-addresses-after.txt"

Get-DnsClientServerAddress -AddressFamily IPv6 |
  Tee-Object "$EvidencePath\client-ipv6-dns-after.txt"

Get-NetRoute -AddressFamily IPv6 |
  Tee-Object "$EvidencePath\client-ipv6-routes-after.txt"

# DNS validation.
Resolve-DnsName $ExpectedDnsName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-resolve-expected-domain.txt"

# DHCP client event logs.
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
~~~

# Configure_DHCPv6_Scopes_And_Options_Verification_Commands
~~~powershell
# DHCP server verification.

Import-Module DhcpServer

Get-Service DHCPServer

Get-DhcpServerInDC

Get-DhcpServerv6Binding -ComputerName <dhcp-server>

Get-DhcpServerv6Scope -ComputerName <dhcp-server>

Get-DhcpServerv6Scope `
  -ComputerName <dhcp-server> `
  -Prefix <dhcpv6-prefix> |
  Format-List *

Get-DhcpServerv6ExclusionRange `
  -ComputerName <dhcp-server> `
  -Prefix <dhcpv6-prefix>

Get-DhcpServerv6OptionValue `
  -ComputerName <dhcp-server>

Get-DhcpServerv6OptionValue `
  -ComputerName <dhcp-server> `
  -Prefix <dhcpv6-prefix>

Get-DhcpServerv6OptionValue `
  -ComputerName <dhcp-server> `
  -Prefix <dhcpv6-prefix> `
  -OptionId 23

Get-DhcpServerv6OptionValue `
  -ComputerName <dhcp-server> `
  -Prefix <dhcpv6-prefix> `
  -OptionId 24

Get-DhcpServerv6Lease `
  -ComputerName <dhcp-server> `
  -Prefix <dhcpv6-prefix>

Get-DhcpServerv6Lease `
  -ComputerName <dhcp-server> `
  -IPAddress <client-ipv6-address>

Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

# Client verification.

ipconfig /all

ipconfig /release6

ipconfig /renew6

Get-NetIPConfiguration

Get-NetIPAddress -AddressFamily IPv6

Get-DnsClientServerAddress -AddressFamily IPv6

Get-NetRoute -AddressFamily IPv6

Resolve-DnsName <domain-fqdn>

Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Client/Admin" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

# Router / L3 gateway verification.

show running-config interface <client-vlan-svi>
show ipv6 interface <client-vlan-svi>
show ipv6 neighbors
show ipv6 route
~~~

# Configure_DHCPv6_Scopes_And_Options_Rollback
~~~powershell
# Run only the rollback that matches the change being reversed.
# Capture current state before rollback.

$DhcpServer = "DHCP1"
$Prefix = "2001:db8:10:20::"
$ExclusionStart = "2001:db8:10:20::1"
$ExclusionEnd = "2001:db8:10:20::ff"
$EvidencePath = "C:\DHCPPrep\dhcpv6-scope-options"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current state.
Get-DhcpServerv6Scope `
  -ComputerName $DhcpServer |
  Export-Csv "$EvidencePath\rollback-dhcpv6-scopes-before.csv" -NoTypeInformation

Get-DhcpServerv6Scope `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-target-scope-before.csv" -NoTypeInformation

Get-DhcpServerv6ExclusionRange `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-exclusions-before.csv" -NoTypeInformation

Get-DhcpServerv6OptionValue `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-scope-options-before.csv" -NoTypeInformation

Get-DhcpServerv6Lease `
  -ComputerName $DhcpServer `
  -Prefix $Prefix `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-leases-before.csv" -NoTypeInformation

# Rollback option values.
# Remove specific DHCPv6 option values only if they were newly added incorrectly.

# Remove-DhcpServerv6OptionValue `
#   -ComputerName $DhcpServer `
#   -Prefix $Prefix `
#   -OptionId 23

# Remove-DhcpServerv6OptionValue `
#   -ComputerName $DhcpServer `
#   -Prefix $Prefix `
#   -OptionId 24

# Rollback exclusion range.
# Remove-DhcpServerv6ExclusionRange `
#   -ComputerName $DhcpServer `
#   -Prefix $Prefix `
#   -StartRange $ExclusionStart `
#   -EndRange $ExclusionEnd `
#   -Passthru

# Rollback active scope to inactive.
# Set-DhcpServerv6Scope `
#   -ComputerName $DhcpServer `
#   -Prefix $Prefix `
#   -State Inactive `
#   -PassThru

# Remove the scope only if this was a lab build or a wrong prefix.
# Warning: Removing a DHCPv6 scope deletes associated settings and leases.

# Remove-DhcpServerv6Scope `
#   -ComputerName $DhcpServer `
#   -Prefix $Prefix `
#   -Confirm

# Client reset after rollback.
# ipconfig /release6
# ipconfig /renew6
# ipconfig /all
~~~

# Configure_DHCPv6_Scopes_And_Options_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Client receives no DHCPv6 address | RA M flag not set for stateful DHCPv6 | Router interface RA config | Enable managed-config flag or change design to SLAAC/stateless |
| Client receives SLAAC address only | Stateless or SLAAC-only design active | `ipconfig /all`; router RA flags | Confirm design or enable stateful DHCPv6 |
| Client receives address but no DNS servers | DHCPv6 DNS option missing or RA O flag wrong | `Get-DhcpServerv6OptionValue -Prefix <prefix>`; client `ipconfig /all` | Set DNS option and ensure O flag is enabled where needed |
| Client receives DNS but no DHCPv6 address | Stateless DHCPv6 behavior | RA M/O flags | Enable M flag if stateful addressing is required |
| Client has no IPv6 default gateway | Router Advertisement issue, not DHCPv6 scope issue | `Get-NetRoute -AddressFamily IPv6`; router RA config | Fix router RA/default-router behavior |
| Scope exists but clients do not lease | Scope inactive | `Get-DhcpServerv6Scope -Prefix <prefix>` | Activate the DHCPv6 scope |
| Scope missing | Prefix not created | `Get-DhcpServerv6Scope` | Create DHCPv6 scope with correct prefix |
| Wrong prefix configured | Scope prefix does not match client VLAN | Compare router interface prefix and DHCPv6 prefix | Remove or disable wrong scope and create correct scope |
| DHCPv6 options set at server level but not expected on scope | Scope-level override missing or wrong | `Get-DhcpServerv6OptionValue` with and without `-Prefix` | Set options at the correct level |
| DNS option points to IPv4 DNS only | IPv6 clients need reachable IPv6 DNS or dual-stack design | `Get-DnsClientServerAddress -AddressFamily IPv6` | Configure IPv6 DNS servers or confirm dual-stack resolver design |
| Excluded range blocks expected leases | Exclusion range too broad | `Get-DhcpServerv6ExclusionRange -Prefix <prefix>` | Narrow or remove incorrect exclusion |
| Server not responding | DHCP Server service stopped or binding wrong | `Get-Service DHCPServer`; `Get-DhcpServerv6Binding` | Start service and correct binding |
| Server not authorized | DHCP server not authorized in AD DS | `Get-DhcpServerInDC` | Authorize DHCP server |
| Client renew fails | Client did not receive RA or cannot reach DHCPv6 server | Client event logs, router RA config, DHCP server logs | Fix RA, VLAN, ACL, firewall, relay, or DHCP service |
| Remote subnet fails only | DHCPv6 relay missing or wrong | Router relay config | Configure DHCPv6 relay toward server |
| Leases appear but DNS fails | Option or DNS server reachability issue | `Get-DhcpServerv6OptionValue`; `Resolve-DnsName` | Correct DNS option or DNS server reachability |
| Client gets IPv6 address but cannot route off-link | Default router not learned through RA | `Get-NetRoute -AddressFamily IPv6` | Fix router advertisements on the gateway |
| DHCPv6 audit/event logs show no client activity | Packet never reaches server | Server logs, relay, VLAN, firewall | Fix network path or relay |
| DHCPv6 lease exists but wrong client behavior remains | Client cache or policy issue | `ipconfig /release6`; `ipconfig /renew6`; client event logs | Renew client, reboot, or check local policy/security agent |

# Configure_DHCPv6_Scopes_And_Options_Related_Labs
| Lab                                                 | Relationship                                                       |
| --------------------------------------------------- | ------------------------------------------------------------------ |
| 01_Install_DHCP_Server_Role_And_Management_Tools    | DHCPv6 cmdlets require the DHCP Server role/tools                  |
| 04_Activate_Authorize_And_Verify_DHCP_Server        | DHCPv6 still depends on service state, authorization, and bindings |
| 07_Configure_DHCP_Relay_And_IP_Helper               | Remote IPv6 client networks may need DHCPv6 relay behavior         |
| 11_Monitor_DHCP_Leases_Events_And_Scope_Utilization | DHCPv6 scope validation uses leases, events, and server state      |
| 12_Troubleshoot_DHCP_Client_Lease_Failures          | Client lease troubleshooting extends naturally into DHCPv6         |
