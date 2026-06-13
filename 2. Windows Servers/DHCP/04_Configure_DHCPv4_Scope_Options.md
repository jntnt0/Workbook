04_Configure_DHCPv4_Scope_Options.md
# Configure_DHCPv4_Scope_Options

# Configure_DHCPv4_Scope_Options_Index
04_Configure_DHCPv4_Scope_Options.md
Configure_DHCPv4_Scope_Options
Configure_DHCPv4_Scope_Options_Source_Basis
Configure_DHCPv4_Scope_Options_Mental_Model
Configure_DHCPv4_Scope_Options_Planning_Table
Configure_DHCPv4_Scope_Options_Configuration_Checklist
Configure_DHCPv4_Scope_Options_Precheck_Skeleton
Configure_DHCPv4_Scope_Options_Core_Options_Skeleton
Configure_DHCPv4_Scope_Options_Optional_Options_Skeleton
Configure_DHCPv4_Scope_Options_Scope_Activation_Skeleton
Configure_DHCPv4_Scope_Options_Client_Validation_Skeleton
Configure_DHCPv4_Scope_Options_Event_Log_Skeleton
Configure_DHCPv4_Scope_Options_Verification_Commands
Configure_DHCPv4_Scope_Options_Rollback
Configure_DHCPv4_Scope_Options_Failure_Checks
Configure_DHCPv4_Scope_Options_Related_Labs

# Configure_DHCPv4_Scope_Options_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Set-DhcpServerv4OptionValue | Configuring DHCPv4 server-level and scope-level option values |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Verifying configured DHCPv4 option values |
| Microsoft Learn | Remove-DhcpServerv4OptionValue | Removing incorrect DHCPv4 option values |
| Microsoft Learn | Get-DhcpServerv4OptionDefinition | Viewing DHCPv4 option IDs and definitions |
| Microsoft Learn | Set-DhcpServerv4Scope | Adjusting DHCPv4 scope settings such as state and lease duration |
| Microsoft Learn | Get-DhcpServerv4Scope | Confirming DHCPv4 scope identity, state, range, and lease duration |
| Microsoft Learn | Get-DhcpServerv4Lease | Confirming client leases after options are configured |
| Microsoft Learn | Get-DhcpServerv4ScopeStatistics | Checking utilization after the scope begins leasing |
| Microsoft Learn | DHCP Server event logs | Reviewing DHCP service, scope, option, and lease events |
| Windows Server operational practice | Router, DNS server, DNS suffix, NTP, and client validation | Ensuring DHCP clients receive usable network configuration |

# Configure_DHCPv4_Scope_Options_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP option | Extra configuration handed to clients along with the leased IP address |
| Scope-level option | Option value applied only to clients leasing from one specific scope |
| Server-level option | Default option value inherited by scopes unless overridden |
| Option 003 Router | Default gateway provided to clients |
| Option 006 DNS Servers | DNS resolver addresses provided to clients |
| Option 015 DNS Domain Name | DNS suffix provided to clients |
| Option 042 NTP Servers | Time server addresses provided to clients when needed |
| Option precedence | Reservation options override scope options, and scope options override server options |
| Router option dependency | Clients may receive an IP but fail off-subnet traffic if option 003 is missing or wrong |
| DNS option dependency | Domain logon, GPO, AD lookup, and name resolution depend on correct option 006 and 015 values |
| Lease renewal | Clients may need release/renew before new options appear |
| Inactive scope stance | Keep a new scope inactive until required options, exclusions, and reservations are ready |
| First rule | A DHCP lease is only useful if the client receives the correct gateway, DNS servers, and DNS suffix for its subnet |

# Configure_DHCPv4_Scope_Options_Planning_Table
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
| Option 003 router | `10.10.10.1` | `<router-option>` |
| Option 006 DNS server 1 | `10.10.10.10` | `<dns-server-1>` |
| Option 006 DNS server 2 | `10.10.10.11` | `<dns-server-2>` |
| Option 015 DNS suffix | `corp.local` | `<dns-suffix>` |
| Option 042 NTP server | `10.10.10.10` | `<ntp-server>` |
| Lease duration | `8 days` | `<lease-duration>` |
| Scope activation timing | After options and exclusions | `<activation-plan>` |
| Client test machine | `WIN11-01` | `<client-name>` |
| Expected client IP range | `10.10.10.50-10.10.10.200` | `<expected-client-range>` |
| Evidence path | `C:\DHCP-Scope-Options` | `<evidence-path>` |
| Next workbook | `05_Configure_DHCPv4_Exclusions_And_Reservations.md` | `<next-task>` |
| Rollback stance | Restore previous option values if clients receive bad config | `<rollback-plan>` |

# Configure_DHCPv4_Scope_Options_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Local Administrators or DHCP Administrators context is visible |
| 2 | Confirm DHCP role and tools | DHCP Server | `Get-WindowsFeature DHCP,RSAT-DHCP` | DHCP role and tools are installed |
| 3 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | DHCP service is Running |
| 4 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | DhcpServer module imports |
| 5 | Confirm DHCP server authorization | DHCP Server / DC | `Get-DhcpServerInDC` | DHCP server appears in AD authorized list |
| 6 | Confirm target scope exists | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Target scope exists |
| 7 | Confirm current scope state | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>" \| Select-Object ScopeId,Name,State` | Scope state is visible |
| 8 | Capture current option values | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Existing options are documented |
| 9 | Confirm option definitions if needed | DHCP Server | `Get-DhcpServerv4OptionDefinition \| Where-Object {$_.OptionId -in 3,6,15,42}` | Core option definitions are visible |
| 10 | Configure router option | DHCP Server | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -Router "<gateway-ip>"` | Clients receive default gateway |
| 11 | Configure DNS server option | DHCP Server | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsServer "<dns-server-1>","<dns-server-2>"` | Clients receive DNS servers |
| 12 | Configure DNS suffix option | DHCP Server | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsDomain "<domain-fqdn>"` | Clients receive DNS suffix |
| 13 | Configure NTP option if needed | DHCP Server | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -OptionId 42 -Value "<ntp-server-ip>"` | Clients receive NTP server if supported |
| 14 | Confirm option values after change | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Expected scope options are present |
| 15 | Configure lease duration if needed | DHCP Server | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -LeaseDuration 8.00:00:00` | Scope has intended lease duration |
| 16 | Keep scope inactive until exclusions/reservations are complete | DHCP Server | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive` | Scope does not lease yet |
| 17 | Activate scope only when ready | DHCP Server | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State Active` | Scope can lease addresses |
| 18 | Validate scope statistics | DHCP Server | `Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"` | Utilization is visible |
| 19 | Validate from client after activation | Client | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Client receives lease and expected options |
| 20 | Confirm server-side lease | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Client lease appears in scope |
| 21 | Export option evidence | DHCP Server | Run option validation skeleton | Option evidence files are saved |
| 22 | Review DHCP events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50` | Recent DHCP events are visible |
| 23 | Document final option state | Operator | `Record router, DNS servers, DNS suffix, optional NTP, lease duration, and scope state` | Scope option record is complete |

# Configure_DHCPv4_Scope_Options_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: confirm the scope exists and capture current option state before changes.

$ScopeId = "10.10.10.0"
$EvidencePath = "C:\DHCP-Scope-Options"

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

# Confirm DHCP server authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Confirm target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-before-options.txt"

# Capture all scopes.
Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\all-scopes-before-options.txt"

# Capture current scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-options-before.txt"

# Capture server-level options for inheritance awareness.
Get-DhcpServerv4OptionValue -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\server-options-before.txt"

# Confirm common option definitions.
Get-DhcpServerv4OptionDefinition |
  Where-Object {$_.OptionId -in 3,6,15,42} |
  Tee-Object "$EvidencePath\common-option-definitions.txt"
```

# Configure_DHCPv4_Scope_Options_Core_Options_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: configure the required DHCPv4 scope options for normal client operation.

$ScopeId = "10.10.10.0"
$Router = "10.10.10.1"
$DnsServers = @("10.10.10.10","10.10.10.11")
$DnsDomain = "corp.local"
$LeaseDuration = "8.00:00:00"
$EvidencePath = "C:\DHCP-Scope-Options"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcpv4-scope-options-transcript.txt"

Import-Module DhcpServer

# Confirm target scope exists before changing options.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-pre-change.txt"

# Configure option 003 Router.
Set-DhcpServerv4OptionValue `
  -ScopeId $ScopeId `
  -Router $Router

# Configure option 006 DNS Servers.
Set-DhcpServerv4OptionValue `
  -ScopeId $ScopeId `
  -DnsServer $DnsServers

# Configure option 015 DNS Domain Name.
Set-DhcpServerv4OptionValue `
  -ScopeId $ScopeId `
  -DnsDomain $DnsDomain

# Configure lease duration if needed.
Set-DhcpServerv4Scope `
  -ScopeId $ScopeId `
  -LeaseDuration $LeaseDuration

# Confirm option values.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-options-after-core-change.txt"

# Confirm scope state and lease duration.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration |
  Tee-Object "$EvidencePath\target-scope-after-core-change.txt"

Stop-Transcript
```

# Configure_DHCPv4_Scope_Options_Optional_Options_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: configure optional DHCPv4 scope options only when required by the environment.
# Keep PXE-specific options for 16_Configure_PXE_Boot_DHCP_Options.md unless required here.

$ScopeId = "10.10.10.0"
$NtpServer = "10.10.10.10"
$EvidencePath = "C:\DHCP-Scope-Options"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Option 042 NTP Servers.
# Use only if clients need DHCP-provided NTP.
Set-DhcpServerv4OptionValue `
  -ScopeId $ScopeId `
  -OptionId 42 `
  -Value $NtpServer

# Optional legacy WINS examples.
# Use only in environments that still require NetBIOS/WINS.
# Set-DhcpServerv4OptionValue `
#   -ScopeId $ScopeId `
#   -OptionId 44 `
#   -Value "10.10.10.30"

# Set-DhcpServerv4OptionValue `
#   -ScopeId $ScopeId `
#   -OptionId 46 `
#   -Value "0x8"

# Confirm all configured scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-options-after-optional-change.txt"

# Confirm specific optional option.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -OptionId 42 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-option-042-ntp.txt"
```

# Configure_DHCPv4_Scope_Options_Scope_Activation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: activate the scope only after required options are configured.
# If exclusions and reservations are not yet configured, keep the scope inactive until workbook 05 is complete.

$ScopeId = "10.10.10.0"
$EvidencePath = "C:\DHCP-Scope-Options"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm scope and option state before activation.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-before-activation.txt"

Get-DhcpServerv4OptionValue -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-options-before-activation.txt"

# Recommended during initial build:
# keep inactive until exclusions and reservations are complete.
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

# Configure_DHCPv4_Scope_Options_Client_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a DHCP client after the scope is active.
# Purpose: confirm the client receives IP, gateway, DNS servers, and DNS suffix from DHCP.

$EvidencePath = "C:\DHCP-Client-Option-Validation"
$ExpectedGateway = "10.10.10.1"
$ExpectedDnsServer = "10.10.10.10"
$ExpectedDnsSuffix = "corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture client state before renew.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-before.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-before.txt"

# Release and renew lease.
ipconfig /release |
  Tee-Object "$EvidencePath\ipconfig-release.txt"

ipconfig /renew |
  Tee-Object "$EvidencePath\ipconfig-renew.txt"

# Capture client state after renew.
ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all-after.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-after.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dns-client-servers-after.txt"

# Test gateway reachability.
Test-NetConnection $ExpectedGateway |
  Tee-Object "$EvidencePath\test-expected-gateway.txt"

# Test DNS server reachability.
Test-NetConnection $ExpectedDnsServer -Port 53 |
  Tee-Object "$EvidencePath\test-expected-dns-server-tcp-53.txt"

# Test DNS suffix/domain resolution.
Resolve-DnsName $ExpectedDnsSuffix -Server $ExpectedDnsServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-expected-dns-suffix.txt"

# Review DHCP client events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 50 |
  Tee-Object "$EvidencePath\dhcp-client-admin-events.txt"
```

# Configure_DHCPv4_Scope_Options_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: collect DHCP Server events after configuring scope options.

$EvidencePath = "C:\DHCP-Scope-Options"

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

# Configure_DHCPv4_Scope_Options_Verification_Commands
```powershell
# Confirm DHCP role, tools, service, and authorization.
Get-WindowsFeature DHCP,RSAT-DHCP
Get-Service DHCPServer
Get-DhcpServerInDC

# Import DHCP module.
Import-Module DhcpServer

# Confirm target scope.
Get-DhcpServerv4Scope -ScopeId "<scope-id>"

# View all scope options.
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"

# View server-level options.
Get-DhcpServerv4OptionValue

# View common option definitions.
Get-DhcpServerv4OptionDefinition | Where-Object {$_.OptionId -in 3,6,15,42}

# Configure core scope options.
Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -Router "<gateway-ip>"
Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsServer "<dns-server-1>","<dns-server-2>"
Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsDomain "<domain-fqdn>"

# Configure optional NTP option.
Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -OptionId 42 -Value "<ntp-server-ip>"

# Configure lease duration if needed.
Set-DhcpServerv4Scope -ScopeId "<scope-id>" -LeaseDuration 8.00:00:00

# Keep scope inactive during build.
Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive

# Activate scope when ready.
Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State Active

# Validate scope state and utilization.
Get-DhcpServerv4Scope -ScopeId "<scope-id>" | Select-Object ScopeId,Name,State,LeaseDuration
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"

# Validate leases.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"

# Client validation.
ipconfig /release
ipconfig /renew
ipconfig /all
Get-NetIPConfiguration
Get-DnsClientServerAddress -AddressFamily IPv4
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"
Test-NetConnection "<gateway-ip>"

# Event logs.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 50
```

# Configure_DHCPv4_Scope_Options_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Wrong router option | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -Router "<correct-gateway-ip>"` | Clients may lose off-subnet access until renewal |
| Wrong DNS server option | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsServer "<correct-dns-1>","<correct-dns-2>"` | AD logon, GPO, and name resolution may fail |
| Wrong DNS suffix option | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsDomain "<correct-domain-fqdn>"` | Clients may fail short-name and domain lookups |
| Wrong NTP option | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -OptionId 42 -Value "<correct-ntp-ip>"` | Time-sensitive services may drift if clients rely on DHCP NTP |
| Option should not exist | `Remove-DhcpServerv4OptionValue -ScopeId "<scope-id>" -OptionId <option-id>` | Clients stop receiving that option after renewal |
| Lease duration wrong | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -LeaseDuration <correct-duration>` | Clients apply new lease duration on renewal |
| Scope activated too early | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive` | Stops new lease assignment |
| Server-level option accidentally changed | Reapply previous server-level option value or remove incorrect server-level option | Multiple scopes may be affected |
| Evidence folder created | `Remove-Item C:\DHCP-Scope-Options -Recurse -Force` | Deletes validation evidence |

# Configure_DHCPv4_Scope_Options_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client gets IP but no default gateway | Option 003 missing or wrong | `Get-DhcpServerv4OptionValue -ScopeId <scope-id>` | Rebuilding the DHCP server |
| Client gets IP but cannot resolve names | Option 006 missing or wrong | `ipconfig /all`; check DNS server option | Deleting the scope |
| Client cannot resolve short names | Option 015 missing or wrong | Check DNS suffix in `ipconfig /all` | Reauthorizing DHCP |
| Client still has old options | Lease not renewed or stale client state | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Changing scope range |
| Scope options look correct but client gets different options | Reservation-level or server-level override | Check reservation options and server-level options | Removing the DHCP role |
| Remote subnet clients get wrong gateway | Wrong scope selected or relay points to wrong server | Confirm client subnet, scope ID, and relay path | Editing DNS first |
| Option command fails | Module syntax or option type issue | `Get-DhcpServerv4OptionDefinition` | Restarting the server repeatedly |
| Scope is inactive | Scope state intentionally or accidentally inactive | `Get-DhcpServerv4Scope -ScopeId <scope-id>` | Troubleshooting client NIC |
| Client gets APIPA | No lease path, inactive scope, relay issue, or service issue | DHCP service, scope state, relay, client VLAN | Editing DNS suffix |
| DHCP server not authorized | AD authorization missing | `Get-DhcpServerInDC` | Changing options |
| DNS option uses public DNS | AD domain clients cannot locate DCs | Check option 006 values | Troubleshooting GPO first |
| Wrong lease duration | Scope property issue | `Get-DhcpServerv4Scope -ScopeId <scope-id>` | Removing options |
| NTP option not used by client | Client OS may not consume DHCP NTP option | Confirm client time configuration | Assuming DHCP failed |
| Event log shows DHCP warnings | Service, authorization, or scope issue | Review DHCP operational log | Recreating all options |

# Configure_DHCPv4_Scope_Options_Related_Labs
| Lab                                          | Related Workbook                                      | Skill Proven                                          |
| -------------------------------------------- | ----------------------------------------------------- | ----------------------------------------------------- |
| Install DHCP Server role                     | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | DHCP role and tools installation                      |
| Authorize DHCP server in AD                  | `02_Authorize_DHCP_Server_In_Active_Directory.md`     | AD-integrated DHCP authorization                      |
| Create DHCPv4 scope                          | `03_Create_DHCPv4_Scope.md`                           | IPv4 lease pool creation                              |
| Configure DHCPv4 scope options               | `04_Configure_DHCPv4_Scope_Options.md`                | Gateway, DNS, DNS suffix, and optional NTP assignment |
| Configure DHCPv4 exclusions and reservations | `05_Configure_DHCPv4_Exclusions_And_Reservations.md`  | Static address protection and fixed client leases     |
| Validate DHCP client lease assignment        | `06_Validate_DHCP_Client_Lease_Assignment.md`         | End-to-end DHCP option validation                     |
| Configure DHCP relay and IP helper           | `07_Configure_DHCP_Relay_And_IP_Helper.md`            | Routed subnet DHCP support                            |
| Configure DHCP DNS dynamic updates           | `08_Configure_DHCP_DNS_Dynamic_Updates.md`            | DHCP and DNS registration behavior                    |
| Troubleshoot DHCP client lease failures      | `12_Troubleshoot_DHCP_Client_Lease_Failures.md`       | Scope option troubleshooting                          |