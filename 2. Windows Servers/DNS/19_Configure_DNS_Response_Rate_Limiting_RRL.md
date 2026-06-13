19_Configure_DNS_Response_Rate_Limiting_RRL.md
# 19_Configure_DNS_Response_Rate_Limiting_RRL

# Configure_DNS_Response_Rate_Limiting_RRL_Index
19_Configure_DNS_Response_Rate_Limiting_RRL.md
Configure_DNS_Response_Rate_Limiting_RRL
Configure_DNS_Response_Rate_Limiting_RRL_Source_Basis
Configure_DNS_Response_Rate_Limiting_RRL_Mental_Model
Configure_DNS_Response_Rate_Limiting_RRL_Planning_Table
Configure_DNS_Response_Rate_Limiting_RRL_Configuration_Checklist
Configure_DNS_Response_Rate_Limiting_RRL_Precheck_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_LogOnly_Baseline_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_Enable_RRL_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_Exception_Client_Subnet_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_Exception_FQDN_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_Exception_Server_Interface_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_Multi_Server_Copy_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_Test_And_Event_Log_Skeleton
Configure_DNS_Response_Rate_Limiting_RRL_Verification_Commands
Configure_DNS_Response_Rate_Limiting_RRL_Rollback
Configure_DNS_Response_Rate_Limiting_RRL_Failure_Checks
Configure_DNS_Response_Rate_Limiting_RRL_Related_Labs

# Configure_DNS_Response_Rate_Limiting_RRL_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Set-DnsServerResponseRateLimiting | Enabling RRL, setting mode, tuning response/error rate, prefix length, leak rate, truncate rate, and window |
| Microsoft Learn | Get-DnsServerResponseRateLimiting | Viewing current RRL settings on a DNS server |
| Microsoft Learn | Add-DnsServerResponseRateLimitingExceptionlist | Creating RRL exception lists for FQDNs, client subnets, server interfaces, or combined conditions |
| Microsoft Learn | Get-DnsServerResponseRateLimitingExceptionlist | Verifying configured RRL exception lists |
| Microsoft Learn | Set-DnsServerResponseRateLimitingExceptionlist | Updating an existing RRL exception list |
| Microsoft Learn | Remove-DnsServerResponseRateLimitingExceptionlist | Removing incorrect or lab-only exception lists |
| Microsoft Learn | Add-DnsServerClientSubnet | Creating named client subnet objects used by DNS policies and RRL exception matching |
| Microsoft Learn | Get-DnsServerClientSubnet | Validating named DNS client subnet objects |
| Microsoft Learn | Get-DnsServerStatistics | Reviewing DNS server statistics before and after RRL changes |
| Windows Server DNS operational practice | DNS Server service, event logs, controlled query validation | Confirming RRL behavior without creating unsafe query floods |

# Configure_DNS_Response_Rate_Limiting_RRL_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS RRL | Response Rate Limiting, used to reduce DNS amplification and abusive repeated responses |
| Repeated response | Same or similar DNS response being sent too often to a client subnet |
| Reflection risk | Attacker spoofs victim IP and causes DNS server to send many responses toward victim |
| Authoritative exposure | RRL matters most on DNS servers exposed to untrusted clients or the internet |
| Internal DNS caveat | Internal domain controllers and recursive DNS servers need careful tuning to avoid breaking normal clients |
| Mode Enable | RRL actively drops or truncates responses based on threshold behavior |
| Mode LogOnly | RRL calculates what it would do but does not enforce drops/truncation |
| Mode Disable | RRL is off |
| ResponsesPerSec | Maximum same-response rate allowed per second |
| ErrorsPerSec | Maximum error response rate allowed per second for errors such as REFUSED, FORMERR, and SERVFAIL |
| WindowInSec | Time window used to measure and average response rates |
| IPv4PrefixLength | Client grouping size for IPv4 source networks, default commonly `/24` behavior |
| IPv6PrefixLength | Client grouping size for IPv6 source networks, default commonly `/56` behavior |
| LeakRate | Allows a portion of responses through even when RRL would otherwise drop |
| TruncateRate | Sends truncated responses for some limited responses, allowing valid clients to retry over TCP |
| MaximumResponsesPerWindow | Hard cap on responses inside the RRL measurement window |
| Exception list | Explicit allow behavior for selected FQDNs, client subnets, server interfaces, or combinations |
| Client subnet object | Named subnet group that can be referenced in DNS policies and exception lists |
| First rule | Start with `LogOnly`, collect evidence, then move to `Enable` |
| Second rule | Do not tune RRL blindly on domain controllers or recursive resolvers used by large NATed client networks |
| Third rule | Create exceptions only for known trusted resolver/client paths, not broad internet ranges |
| Fourth rule | RRL is not a firewall replacement; still restrict recursion, zone transfers, and DNS exposure separately |

# Configure_DNS_Response_Rate_Limiting_RRL_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server | `DNS1` | `<dns-server>` |
| Server role | Authoritative / recursive / AD-integrated DC DNS | `<dns-server-role>` |
| Zone protected | `corp.example.com` | `<zone-name>` |
| External exposure | Internal only / internet-facing / partner-facing | `<exposure-type>` |
| Initial RRL mode | `LogOnly` | `<initial-mode>` |
| Final RRL mode | `Enable` | `<final-mode>` |
| Responses per second | `8` | `<responses-per-sec>` |
| Errors per second | `8` | `<errors-per-sec>` |
| Window in seconds | `5` | `<window-in-sec>` |
| IPv4 prefix length | `24` | `<ipv4-prefix-length>` |
| IPv6 prefix length | `56` | `<ipv6-prefix-length>` |
| Leak rate | `3` | `<leak-rate>` |
| Truncate rate | `2` | `<truncate-rate>` |
| Maximum responses per window | `0` or environment-specific value | `<max-responses-per-window>` |
| Trusted resolver subnet name | `TrustedResolvers` | `<trusted-client-subnet-name>` |
| Trusted resolver IPv4 CIDR | `10.10.10.0/24` | `<trusted-resolver-ipv4>` |
| Trusted resolver IPv6 CIDR | `2001:db8:10:10::/64` | `<trusted-resolver-ipv6>` |
| Exception list name | `RRL-Trusted-Resolvers` | `<exception-list-name>` |
| Exception FQDN pattern | `EQ,*.corp.example.com` | `<exception-fqdn>` |
| Server interface exception | `EQ,10.10.10.20` | `<server-interface-ip>` |
| Evidence path | `C:\DNSPrep\dns-rrl` | `<evidence-path>` |
| Test client | `TESTCLIENT1` | `<test-client>` |
| Monitoring window | `24 hours` | `<monitoring-window>` |
| Rollback mode | `LogOnly` or `Disable` | `<rollback-mode>` |

# Configure_DNS_Response_Rate_Limiting_RRL_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify DNS server role | Operator | Review DNS server purpose and exposure | RRL risk level is known |
| 2 | Create evidence path | DNS server | `New-Item -ItemType Directory -Force -Path C:\DNSPrep\dns-rrl` | Evidence folder exists |
| 3 | Import DNS module | DNS server | `Import-Module DnsServer` | DNS cmdlets are available |
| 4 | Confirm DNS service | DNS server | `Get-Service DNS` | DNS Server service is running |
| 5 | Capture DNS server settings | DNS server | `Get-DnsServer -ComputerName <dns-server>` | Baseline server configuration is captured |
| 6 | Capture existing RRL settings | DNS server | `Get-DnsServerResponseRateLimiting -ComputerName <dns-server>` | Current RRL mode/tuning is known |
| 7 | Capture existing RRL exceptions | DNS server | `Get-DnsServerResponseRateLimitingExceptionlist -ComputerName <dns-server>` | Existing exceptions are known |
| 8 | Capture DNS statistics | DNS server | `Get-DnsServerStatistics -ComputerName <dns-server>` | Baseline counters are captured |
| 9 | Capture zone list | DNS server | `Get-DnsServerZone -ComputerName <dns-server>` | Hosted zones are known |
| 10 | Configure RRL in LogOnly first | DNS server | `Set-DnsServerResponseRateLimiting -ComputerName <dns-server> -Mode LogOnly` | RRL evaluates behavior without enforcement |
| 11 | Set conservative tuning values | DNS server | `Set-DnsServerResponseRateLimiting -ResponsesPerSec <value> -ErrorsPerSec <value> -WindowInSec <value>` | RRL settings match plan |
| 12 | Create trusted client subnet object if needed | DNS server | `Add-DnsServerClientSubnet -ComputerName <dns-server> -Name <name> -IPv4Subnet <cidr>` | Named trusted subnet exists |
| 13 | Add RRL exception for trusted resolver subnet if needed | DNS server | `Add-DnsServerResponseRateLimitingExceptionlist -ComputerName <dns-server> -Name <exception-name> -ClientSubnet "EQ,<subnet-name>"` | Trusted subnet is exempt |
| 14 | Add RRL exception for FQDN if needed | DNS server | `Add-DnsServerResponseRateLimitingExceptionlist -ComputerName <dns-server> -Name <exception-name> -Fqdn "EQ,*.corp.example.com"` | Selected FQDN pattern is exempt |
| 15 | Add RRL exception for server interface if needed | DNS server | `Add-DnsServerResponseRateLimitingExceptionlist -ComputerName <dns-server> -Name <exception-name> -ServerInterfaceIP "EQ,<server-ip>"` | Queries received on selected interface are exempt |
| 16 | Monitor in LogOnly mode | DNS server | Event logs, DNS statistics, controlled client queries | No critical legitimate clients are flagged |
| 17 | Enable RRL enforcement | DNS server | `Set-DnsServerResponseRateLimiting -ComputerName <dns-server> -Mode Enable` | RRL enforcement is active |
| 18 | Validate final RRL settings | DNS server | `Get-DnsServerResponseRateLimiting -ComputerName <dns-server>` | Mode and thresholds match plan |
| 19 | Validate exception lists | DNS server | `Get-DnsServerResponseRateLimitingExceptionlist -ComputerName <dns-server>` | Exceptions match plan |
| 20 | Run controlled DNS validation | Client | `Resolve-DnsName <record> -Server <dns-server>` | Normal DNS resolution still works |
| 21 | Capture final evidence | DNS server | Export RRL settings, exceptions, statistics, service state, and event logs | Build evidence is preserved |
| 22 | Roll back if normal clients are affected | DNS server | `Set-DnsServerResponseRateLimiting -ComputerName <dns-server> -Mode LogOnly` | Enforcement stops while diagnostics continue |

# Configure_DNS_Response_Rate_Limiting_RRL_Precheck_Skeleton
~~~powershell
# Run on the DNS server or a management host with RSAT DNS tools.

$DnsServer = "DNS1"
$EvidencePath = "C:\DNSPrep\dns-rrl"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Confirm DNS Server service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\precheck-dns-service.txt"

# Capture DNS server configuration.
Get-DnsServer `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-dns-server-config.txt"

# Capture existing RRL settings.
Get-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-rrl-settings.txt"

# Capture existing RRL exception lists.
Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-rrl-exception-lists.txt"

# Capture DNS client subnet objects.
Get-DnsServerClientSubnet `
  -ComputerName $DnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-dns-client-subnets.txt"

# Capture hosted zones.
Get-DnsServerZone `
  -ComputerName $DnsServer |
  Tee-Object "$EvidencePath\precheck-dns-zones.txt"

# Capture DNS statistics.
Get-DnsServerStatistics `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-dns-statistics.txt"

# Capture recent DNS Server event log entries.
Get-WinEvent `
  -LogName "DNS Server" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-dns-server-events.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_LogOnly_Baseline_Skeleton
~~~powershell
# Start with LogOnly mode.
# RRL will calculate what it would do without dropping or truncating responses.

$DnsServer = "DNS1"
$EvidencePath = "C:\DNSPrep\dns-rrl"

$ResponsesPerSec = 8
$ErrorsPerSec = 8
$WindowInSec = 5
$IPv4PrefixLength = 24
$IPv6PrefixLength = 56
$LeakRate = 3
$TruncateRate = 2

Import-Module DnsServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Set RRL to LogOnly with planned values.
Set-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer `
  -Mode LogOnly `
  -ResponsesPerSec $ResponsesPerSec `
  -ErrorsPerSec $ErrorsPerSec `
  -WindowInSec $WindowInSec `
  -IPv4PrefixLength $IPv4PrefixLength `
  -IPv6PrefixLength $IPv6PrefixLength `
  -LeakRate $LeakRate `
  -TruncateRate $TruncateRate `
  -Force `
  -PassThru |
  Tee-Object "$EvidencePath\logonly-rrl-set-result.txt"

# Validate RRL settings.
Get-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\logonly-rrl-settings-after.txt"

# Capture DNS statistics after setting LogOnly.
Get-DnsServerStatistics `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\logonly-dns-statistics-after.txt"

# Capture recent DNS events.
Get-WinEvent `
  -LogName "DNS Server" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\logonly-dns-server-events-after.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Enable_RRL_Skeleton
~~~powershell
# Enable enforcement only after LogOnly evidence is reviewed.
# This activates dropping/truncating behavior based on RRL thresholds.

$DnsServer = "DNS1"
$EvidencePath = "C:\DNSPrep\dns-rrl"

Import-Module DnsServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current state before enforcement.
Get-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\enable-before-rrl-settings.txt"

Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\enable-before-exceptions.txt"

# Enable RRL enforcement.
Set-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer `
  -Mode Enable `
  -Force `
  -PassThru |
  Tee-Object "$EvidencePath\enable-rrl-result.txt"

# Validate enforcement mode.
Get-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\enable-after-rrl-settings.txt"

# Capture DNS Server event log after enabling.
Get-WinEvent `
  -LogName "DNS Server" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\enable-after-dns-events.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Exception_Client_Subnet_Skeleton
~~~powershell
# Use this when trusted resolver/client subnets must be exempt from RRL.
# ClientSubnet values in exception lists reference DNS client subnet object names.

$DnsServer = "DNS1"
$ClientSubnetName = "TrustedResolvers"
$TrustedIPv4Subnet = "10.10.10.0/24"
$TrustedIPv6Subnet = "2001:db8:10:10::/64"
$ExceptionListName = "RRL-Trusted-Resolvers"
$EvidencePath = "C:\DNSPrep\dns-rrl"

Import-Module DnsServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Create named DNS client subnet object.
Add-DnsServerClientSubnet `
  -ComputerName $DnsServer `
  -Name $ClientSubnetName `
  -IPv4Subnet $TrustedIPv4Subnet `
  -IPv6Subnet $TrustedIPv6Subnet `
  -PassThru |
  Tee-Object "$EvidencePath\exception-client-subnet-create.txt"

# Validate client subnet object.
Get-DnsServerClientSubnet `
  -ComputerName $DnsServer `
  -Name $ClientSubnetName |
  Format-List * |
  Tee-Object "$EvidencePath\exception-client-subnet-verify.txt"

# Create RRL exception list using the named client subnet.
Add-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -Name $ExceptionListName `
  -ClientSubnet "EQ,$ClientSubnetName" `
  -PassThru |
  Tee-Object "$EvidencePath\exception-client-subnet-rrl-create.txt"

# Validate RRL exception.
Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -Name $ExceptionListName |
  Format-List * |
  Tee-Object "$EvidencePath\exception-client-subnet-rrl-verify.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Exception_FQDN_Skeleton
~~~powershell
# Use FQDN exceptions sparingly.
# This exempts responses for selected FQDN patterns from RRL behavior.

$DnsServer = "DNS1"
$ExceptionListName = "RRL-FQDN-SafeList"
$FqdnMatch = "EQ,*.corp.example.com"
$EvidencePath = "C:\DNSPrep\dns-rrl"

Import-Module DnsServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Create exception list for FQDN pattern.
Add-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -Name $ExceptionListName `
  -Fqdn $FqdnMatch `
  -PassThru |
  Tee-Object "$EvidencePath\exception-fqdn-create.txt"

# Validate exception list.
Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -Name $ExceptionListName |
  Format-List * |
  Tee-Object "$EvidencePath\exception-fqdn-verify.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Exception_Server_Interface_Skeleton
~~~powershell
# Use server-interface exceptions only when a specific listener/interface must be exempt.
# Example: internal management/listener IP exempt, public interface not exempt.

$DnsServer = "DNS1"
$ExceptionListName = "RRL-Internal-Interface"
$ServerInterfaceIp = "EQ,10.10.10.20"
$EvidencePath = "C:\DNSPrep\dns-rrl"

Import-Module DnsServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Create exception list for server interface IP.
Add-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -Name $ExceptionListName `
  -ServerInterfaceIP $ServerInterfaceIp `
  -PassThru |
  Tee-Object "$EvidencePath\exception-interface-create.txt"

# Validate exception list.
Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -Name $ExceptionListName |
  Format-List * |
  Tee-Object "$EvidencePath\exception-interface-verify.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Multi_Server_Copy_Skeleton
~~~powershell
# Use this to copy known-good RRL settings from one DNS server to another.
# Validate exceptions separately, because exception lists and client subnet objects must also exist on the target.

$SourceDnsServer = "DNS1"
$TargetDnsServer = "DNS2"
$EvidencePath = "C:\DNSPrep\dns-rrl"

Import-Module DnsServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture source settings.
Get-DnsServerResponseRateLimiting `
  -ComputerName $SourceDnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\copy-source-rrl-settings.txt"

# Capture target settings before copy.
Get-DnsServerResponseRateLimiting `
  -ComputerName $TargetDnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\copy-target-rrl-settings-before.txt"

# Copy RRL settings from source to target.
Get-DnsServerResponseRateLimiting `
  -ComputerName $SourceDnsServer |
  Set-DnsServerResponseRateLimiting `
    -ComputerName $TargetDnsServer `
    -Force `
    -PassThru |
  Tee-Object "$EvidencePath\copy-target-rrl-set-result.txt"

# Validate target settings after copy.
Get-DnsServerResponseRateLimiting `
  -ComputerName $TargetDnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\copy-target-rrl-settings-after.txt"

# Capture exception lists from both servers for comparison.
Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $SourceDnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\copy-source-exceptions.txt"

Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $TargetDnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\copy-target-exceptions.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Test_And_Event_Log_Skeleton
~~~powershell
# Use controlled validation only.
# Do not create a query flood against production DNS.
# Goal: confirm normal lookups still work and event visibility exists.

$DnsServer = "DNS1"
$RecordToTest = "www.corp.example.com"
$EvidencePath = "C:\DNSPrep\dns-rrl"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Normal controlled lookup tests.
Resolve-DnsName `
  -Name $RecordToTest `
  -Server $DnsServer |
  Tee-Object "$EvidencePath\test-resolve-record.txt"

Resolve-DnsName `
  -Name $RecordToTest `
  -Server $DnsServer `
  -Type A `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\test-resolve-a.txt"

Resolve-DnsName `
  -Name $RecordToTest `
  -Server $DnsServer `
  -Type AAAA `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\test-resolve-aaaa.txt"

# Small repeated sample, not a stress test.
1..5 | ForEach-Object {
  Resolve-DnsName `
    -Name $RecordToTest `
    -Server $DnsServer `
    -ErrorAction SilentlyContinue
} |
  Tee-Object "$EvidencePath\test-small-repeat-sample.txt"

# Server-side statistics.
Get-DnsServerStatistics `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\test-dns-statistics.txt"

# DNS Server event log.
Get-WinEvent `
  -LogName "DNS Server" `
  -MaxEvents 300 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\test-dns-server-events.txt"

# Current RRL state.
Get-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\test-current-rrl-settings.txt"

# Current RRL exceptions.
Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\test-current-rrl-exceptions.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Verification_Commands
~~~powershell
# DNS service and module.
Import-Module DnsServer
Get-Service DNS

# Current RRL settings.
Get-DnsServerResponseRateLimiting -ComputerName <dns-server>
Get-DnsServerResponseRateLimiting -ComputerName <dns-server> | Format-List *

# RRL exception lists.
Get-DnsServerResponseRateLimitingExceptionlist -ComputerName <dns-server>
Get-DnsServerResponseRateLimitingExceptionlist -ComputerName <dns-server> -Name <exception-list-name> | Format-List *

# DNS client subnet objects used by exception lists.
Get-DnsServerClientSubnet -ComputerName <dns-server>
Get-DnsServerClientSubnet -ComputerName <dns-server> -Name <client-subnet-name> | Format-List *

# DNS server configuration and zones.
Get-DnsServer -ComputerName <dns-server> | Format-List *
Get-DnsServerZone -ComputerName <dns-server>

# DNS statistics.
Get-DnsServerStatistics -ComputerName <dns-server> | Format-List *

# DNS event log.
Get-WinEvent -LogName "DNS Server" -MaxEvents 200 -ErrorAction SilentlyContinue

# Controlled client lookup.
Resolve-DnsName <record-name> -Server <dns-server>
Resolve-DnsName <record-name> -Server <dns-server> -Type A
Resolve-DnsName <record-name> -Server <dns-server> -Type AAAA

# Switch modes.
Set-DnsServerResponseRateLimiting -ComputerName <dns-server> -Mode LogOnly -Force -PassThru
Set-DnsServerResponseRateLimiting -ComputerName <dns-server> -Mode Enable -Force -PassThru
Set-DnsServerResponseRateLimiting -ComputerName <dns-server> -Mode Disable -Force -PassThru

# Reset all RRL parameters to default.
Set-DnsServerResponseRateLimiting -ComputerName <dns-server> -ResetToDefault -Force -PassThru
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Rollback
~~~powershell
# Run only the rollback that matches the change being reversed.
# Safest rollback is usually Mode LogOnly first, then Disable if needed.

$DnsServer = "DNS1"
$EvidencePath = "C:\DNSPrep\dns-rrl"

$ExceptionListName1 = "RRL-Trusted-Resolvers"
$ExceptionListName2 = "RRL-FQDN-SafeList"
$ExceptionListName3 = "RRL-Internal-Interface"

Import-Module DnsServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current RRL state before rollback.
Get-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\rollback-rrl-settings-before.txt"

Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\rollback-exceptions-before.txt"

Get-DnsServerClientSubnet `
  -ComputerName $DnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\rollback-client-subnets-before.txt"

# Rollback option 1: move from enforcement to LogOnly.
Set-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer `
  -Mode LogOnly `
  -Force `
  -PassThru |
  Tee-Object "$EvidencePath\rollback-set-logonly.txt"

# Rollback option 2: disable RRL completely if LogOnly is not enough.
# Set-DnsServerResponseRateLimiting `
#   -ComputerName $DnsServer `
#   -Mode Disable `
#   -Force `
#   -PassThru

# Rollback option 3: reset RRL tuning to Microsoft defaults.
# Set-DnsServerResponseRateLimiting `
#   -ComputerName $DnsServer `
#   -ResetToDefault `
#   -Force `
#   -PassThru

# Rollback option 4: remove lab or incorrect exception lists.
# Remove-DnsServerResponseRateLimitingExceptionlist `
#   -ComputerName $DnsServer `
#   -Name $ExceptionListName1 `
#   -Force `
#   -PassThru

# Remove-DnsServerResponseRateLimitingExceptionlist `
#   -ComputerName $DnsServer `
#   -Name $ExceptionListName2 `
#   -Force `
#   -PassThru

# Remove-DnsServerResponseRateLimitingExceptionlist `
#   -ComputerName $DnsServer `
#   -Name $ExceptionListName3 `
#   -Force `
#   -PassThru

# Validate rollback state.
Get-DnsServerResponseRateLimiting `
  -ComputerName $DnsServer |
  Format-List * |
  Tee-Object "$EvidencePath\rollback-rrl-settings-after.txt"

Get-DnsServerResponseRateLimitingExceptionlist `
  -ComputerName $DnsServer `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\rollback-exceptions-after.txt"
~~~

# Configure_DNS_Response_Rate_Limiting_RRL_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Legitimate clients intermittently fail DNS lookups | RRL thresholds too aggressive | RRL mode/settings, event logs, client subnet size | Move to LogOnly, increase thresholds, or add narrow trusted exception |
| Large NATed client network affected | Many clients grouped into one source prefix | IPv4PrefixLength, client NAT design | Adjust prefix length or exempt trusted resolver subnet |
| RRL appears to do nothing | Mode is LogOnly or Disable | `Get-DnsServerResponseRateLimiting` | Set Mode to Enable after validation |
| RRL enforcement started too soon | LogOnly monitoring skipped | Change history and event logs | Revert to LogOnly and monitor |
| Exception list does not match | Wrong comparator, subnet name, FQDN pattern, or condition | `Get-DnsServerResponseRateLimitingExceptionlist` | Correct exception list syntax |
| Client subnet exception does not match | Client subnet object missing or wrong CIDR | `Get-DnsServerClientSubnet` | Create or fix client subnet object |
| FQDN exception too broad | Wildcard pattern exempts more than intended | Exception list FQDN value | Narrow FQDN match |
| Server interface exception too broad | Exempted public interface | Exception list ServerInterfaceIP | Exempt only trusted/internal listener if needed |
| DNS amplification risk remains | RRL is not a substitute for exposure controls | Recursion settings, firewall, external scans | Restrict recursion, ACL DNS exposure, and harden firewall |
| Authoritative zone still abused | RRL too loose or exceptions too broad | DNS statistics and event logs | Tighten RRL values and remove broad exceptions |
| Recursive DNS clients break | RRL applied to busy recursive resolver path | Client traffic pattern and NAT behavior | Use trusted resolver architecture and tune conservatively |
| Internal DC DNS issues appear | RRL applied too aggressively to AD DNS | Client and DC event logs | Use LogOnly, tune carefully, or disable for DC DNS if required |
| Copy to second DNS server incomplete | RRL settings copied but exceptions/subnets missing | Compare RRL, exceptions, client subnets | Recreate matching exception/client subnet objects |
| `Set-DnsServerResponseRateLimiting` fails | DNS module missing or insufficient rights | Module import, role, admin privileges | Install RSAT/DNS role tools and run elevated |
| No DNS Server event logs found | Log not present, disabled, or different server context | Event Viewer and `Get-WinEvent -ListLog *DNS*` | Enable/check proper DNS logs |
| Controlled lookup fails after RRL | DNS zone/record problem, not necessarily RRL | Query same record with RRL LogOnly | Troubleshoot DNS record/zone separately |
| Rollback does not restore behavior | Client cache or resolver cache still stale | `ipconfig /flushdns`, DNS cache checks | Flush client/server cache and retest |
| Truncated responses confuse clients | TruncateRate behavior exposes TCP fallback problems | Client behavior and firewall TCP 53 | Permit TCP 53 or adjust truncation behavior |
| Error responses throttled unexpectedly | ErrorsPerSec threshold too low | Query failures and RRL settings | Increase ErrorsPerSec or fix underlying DNS errors |
| Broad exception weakens protection | Exception design too permissive | Exception list scope | Remove or narrow exception list |

# Configure_DNS_Response_Rate_Limiting_RRL_Related_Labs
| Lab                                                    | Relationship                                                               |
| ------------------------------------------------------ | -------------------------------------------------------------------------- |
| 01_Install_DNS_Server_Role_And_Management_Tools        | RRL requires DNS Server role/tools and DnsServer PowerShell module         |
| 02_Create_Forward_Lookup_Zone                          | RRL protects responses for hosted zones, especially authoritative exposure |
| 04_Create_And_Manage_DNS_Resource_Records              | Repeated records can become part of repeated-response behavior             |
| 05_Configure_DNS_Forwarders_And_Conditional_Forwarders | RRL does not replace correct forwarding/recursive design                   |
| 06_Configure_DNS_Recursion_And_Root_Hints              | Recursion exposure must be controlled separately from RRL                  |
| 07_Configure_DNS_Zone_Transfers                        | RRL does not replace zone transfer restrictions                            |
| 11_Monitor_DNS_Events_Queries_And_Server_Health        | RRL validation depends on DNS statistics, event logs, and query behavior   |
| 12_Troubleshoot_DNS_Client_Resolution_Failures         | Over-aggressive RRL can look like intermittent client DNS failure          |
| 18_Configure_DNS_Policies_And_Client_Subnets           | RRL exception lists can use named DNS client subnet objects                |