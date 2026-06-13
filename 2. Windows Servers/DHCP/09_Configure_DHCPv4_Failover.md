09_Configure_DHCPv4_Failover.md
# Configure_DHCPv4_Failover

# Configure_DHCPv4_Failover_Index
09_Configure_DHCPv4_Failover.md
Configure_DHCPv4_Failover
Configure_DHCPv4_Failover_Source_Basis
Configure_DHCPv4_Failover_Mental_Model
Configure_DHCPv4_Failover_Planning_Table
Configure_DHCPv4_Failover_Configuration_Checklist
Configure_DHCPv4_Failover_Precheck_Skeleton
Configure_DHCPv4_Failover_Partner_Server_Readiness_Skeleton
Configure_DHCPv4_Failover_Load_Balance_Mode_Skeleton
Configure_DHCPv4_Failover_Hot_Standby_Mode_Skeleton
Configure_DHCPv4_Failover_Add_Scope_To_Existing_Relationship_Skeleton
Configure_DHCPv4_Failover_Replication_Skeleton
Configure_DHCPv4_Failover_Client_Lease_Validation_Skeleton
Configure_DHCPv4_Failover_Partner_Failure_Test_Skeleton
Configure_DHCPv4_Failover_Event_Log_Skeleton
Configure_DHCPv4_Failover_Verification_Commands
Configure_DHCPv4_Failover_Rollback
Configure_DHCPv4_Failover_Failure_Checks
Configure_DHCPv4_Failover_Related_Labs

# Configure_DHCPv4_Failover_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DHCP Server PowerShell module | DHCPv4 scope, lease, failover, replication, and server validation |
| Microsoft Learn | Add-DhcpServerv4Failover | Creating DHCPv4 failover relationships |
| Microsoft Learn | Get-DhcpServerv4Failover | Reviewing DHCPv4 failover relationship state |
| Microsoft Learn | Set-DhcpServerv4Failover | Updating DHCPv4 failover settings |
| Microsoft Learn | Remove-DhcpServerv4Failover | Removing DHCPv4 failover relationships |
| Microsoft Learn | Add-DhcpServerv4FailoverScope | Adding scopes to an existing failover relationship |
| Microsoft Learn | Get-DhcpServerv4FailoverScope | Reviewing scopes assigned to failover relationships |
| Microsoft Learn | Invoke-DhcpServerv4FailoverReplication | Manually replicating DHCP scope/config data between partners |
| Microsoft Learn | Get-DhcpServerv4Scope | Validating scope state on each DHCP server |
| Microsoft Learn | Get-DhcpServerv4Lease | Validating client lease ownership and assignment |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Validating replicated scope options |
| Microsoft Learn | DHCP Server event logs | Reviewing partner communication, replication, and lease events |
| Windows Server operational practice | DHCP high availability, load balance, hot standby, MCLT, failover state handling | Keeping DHCP leases available if one DHCP server fails |

# Configure_DHCPv4_Failover_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP failover | Partnership between two DHCP servers that share lease responsibility for one or more IPv4 scopes |
| Failover relationship | Named configuration binding two DHCP servers together for selected scopes |
| Partner server | The second DHCP server in the failover pair |
| Load balance mode | Both servers actively lease addresses from the same scope based on percentage split |
| Hot standby mode | One server actively leases while partner stays ready with reserved capacity |
| MCLT | Maximum Client Lead Time, limiting how far one server can extend leases without partner confirmation |
| Auto state transition | Allows server to automatically move from communication-interrupted to partner-down after timer expires |
| State switch interval | Timer used for automatic failover state transition |
| Shared secret | Secret used to authenticate failover communication between DHCP partners |
| Scope replication | Copying scope configuration, options, reservations, policies, and lease state between partners |
| Communication interrupted | State where partners cannot communicate but do not yet assume full partner-down behavior |
| Partner down | State where one DHCP server assumes the partner is unavailable |
| Conflict risk | Bad failover setup can cause duplicate leases, stale options, or inconsistent DNS updates |
| DNS update credential risk | Both failover partners should use the same DHCP DNS update credential |
| First rule | Configure failover only after both DHCP servers are authorized, reachable, time-synced, and using matching DNS update credentials |

# Configure_DHCPv4_Failover_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Primary DHCP server | `DHCP1.corp.local` | `<primary-dhcp-server>` |
| Primary DHCP server IP | `10.10.10.5` | `<primary-dhcp-ip>` |
| Partner DHCP server | `DHCP2.corp.local` | `<partner-dhcp-server>` |
| Partner DHCP server IP | `10.10.10.6` | `<partner-dhcp-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Failover relationship name | `DHCP1-DHCP2-LB` | `<failover-name>` |
| Failover mode | LoadBalance / HotStandby | `<failover-mode>` |
| Scope ID | `10.20.20.0` | `<scope-id>` |
| Additional scope IDs | `10.30.30.0,10.40.40.0` | `<additional-scope-ids>` |
| Scope name | `Workstations-VLAN20` | `<scope-name>` |
| Load balance percent | `50` | `<load-balance-percent>` |
| Hot standby reserve percent | `5` | `<reserve-percent>` |
| Maximum client lead time | `01:00:00` | `<mclt>` |
| Auto state transition | Enabled / Disabled | `<auto-state-transition>` |
| State switch interval | `01:00:00` | `<state-switch-interval>` |
| Shared secret | Stored securely | `<shared-secret-source>` |
| DHCP DNS credential account | `corp\svc-dhcpdns` | `<dhcp-dns-credential-account>` |
| Scope options | Router, DNS, DNS suffix | `<scope-options>` |
| Relay/IP helper path | Client VLAN gateway to both DHCP servers | `<relay-path>` |
| Firewall requirement | DHCP failover traffic and DHCP relay traffic allowed | `<firewall-requirement>` |
| Test client | `WIN11-01` | `<test-client>` |
| Evidence path | `C:\DHCPv4-Failover` | `<evidence-path>` |
| Rollback stance | Remove failover relationship only after preserving scopes and leases | `<rollback-plan>` |
| Next workbook | `10_Backup_Restore_And_Export_DHCP_Server_Config.md` | `<next-task>` |

# Configure_DHCPv4_Failover_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Primary DHCP Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DHCP role installed on primary | Primary DHCP Server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 3 | Confirm DHCP role installed on partner | Partner DHCP Server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 4 | Confirm DHCP services running | Both DHCP Servers | `Get-Service DHCPServer` | DHCP service is Running |
| 5 | Import DHCP module | Primary DHCP Server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 6 | Confirm both servers authorized in AD | Primary DHCP Server | `Get-DhcpServerInDC` | Both DHCP servers are authorized |
| 7 | Confirm primary can reach partner | Primary DHCP Server | `Test-NetConnection "<partner-dhcp-server>"` | Partner is reachable |
| 8 | Confirm partner can reach primary | Partner DHCP Server | `Test-NetConnection "<primary-dhcp-server>"` | Primary is reachable |
| 9 | Confirm time sync | Both DHCP Servers | `w32tm /query /status` | Time is synchronized |
| 10 | Confirm target scope exists on primary | Primary DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Scope exists on primary |
| 11 | Confirm scope is active | Primary DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>" \| Select State` | Scope is Active |
| 12 | Capture scope options | Primary DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Scope options are documented |
| 13 | Capture leases | Primary DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Current leases are documented |
| 14 | Confirm DHCP DNS credential state | Both DHCP Servers | `Get-DhcpServerDnsCredential` | Credential state is known |
| 15 | Configure matching DHCP DNS credential if used | Both DHCP Servers | `Set-DhcpServerDnsCredential -Credential (Get-Credential)` | Both partners use same DNS update account |
| 16 | Create failover relationship | Primary DHCP Server | `Add-DhcpServerv4Failover ...` | Failover relationship is created |
| 17 | Confirm failover relationship | Primary DHCP Server | `Get-DhcpServerv4Failover -Name "<failover-name>"` | Relationship appears healthy |
| 18 | Confirm failover scope assignment | Primary DHCP Server | `Get-DhcpServerv4FailoverScope -Name "<failover-name>"` | Scope is assigned |
| 19 | Replicate failover configuration | Primary DHCP Server | `Invoke-DhcpServerv4FailoverReplication -Name "<failover-name>" -Force` | Scope and failover data replicate |
| 20 | Confirm scope exists on partner | Partner DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Scope appears on partner |
| 21 | Confirm options on partner | Partner DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Options match primary |
| 22 | Confirm leases on partner | Partner DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Lease state is visible |
| 23 | Confirm relay/IP helper reaches both servers if required | Network Device | Review helper addresses | Relay can reach DHCP pair |
| 24 | Renew test client lease | Client | `ipconfig /release`; `ipconfig /renew` | Client receives lease |
| 25 | Validate lease on DHCP servers | Both DHCP Servers | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Lease is visible in failover pair |
| 26 | Review DHCP failover events | Both DHCP Servers | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100` | Failover events are captured |
| 27 | Export final evidence | DHCP Servers / Client | Run validation skeletons | Evidence files are saved |
| 28 | Document final state | Operator | `Record relationship name, mode, partner, scopes, MCLT, state, and validation result` | Workbook record is complete |

# Configure_DHCPv4_Failover_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the primary DHCP server.
# Purpose: capture DHCP server, scope, lease, authorization, DNS credential, and existing failover state before changes.

$PrimaryDhcpServer = "DHCP1.corp.local"
$PartnerDhcpServer = "DHCP2.corp.local"
$ScopeId = "10.20.20.0"
$EvidencePath = "C:\DHCPv4-Failover"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Capture local server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion |
  Tee-Object "$EvidencePath\computer-info.txt"

# Confirm DHCP role and service locally.
Get-WindowsFeature DHCP |
  Tee-Object "$EvidencePath\primary-dhcp-role-state.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\primary-dhcp-service-state.txt"

# Import DHCP module.
Import-Module DhcpServer

# Confirm AD authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Confirm partner reachability.
Test-NetConnection $PartnerDhcpServer |
  Tee-Object "$EvidencePath\test-partner-reachability.txt"

# Capture existing failover relationships.
Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\failover-relationships-before.txt"

# Capture all scopes.
Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\primary-scopes-before.txt"

Get-DhcpServerv4Scope |
  Export-Csv "$EvidencePath\primary-scopes-before.csv" -NoTypeInformation

# Capture target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-before.txt"

# Capture target scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-options-before.txt"

# Capture target scope exclusions, reservations, policies, and leases.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-exclusions-before.txt"

Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-reservations-before.txt"

Get-DhcpServerv4Policy -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-policies-before.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-leases-before.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\target-scope-leases-before.csv" -NoTypeInformation

# Capture DHCP DNS credential state.
Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\primary-dhcp-dns-credential-before.txt"

# Capture time sync state.
w32tm /query /status |
  Tee-Object "$EvidencePath\primary-time-sync-status.txt"
```

# Configure_DHCPv4_Failover_Partner_Server_Readiness_Skeleton
```powershell
# Run in elevated PowerShell on both DHCP servers.
# Purpose: verify both servers are ready before creating a failover relationship.

$PrimaryDhcpServer = "DHCP1.corp.local"
$PartnerDhcpServer = "DHCP2.corp.local"
$EvidencePath = "C:\DHCPv4-Failover\Partner-Readiness"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Local DHCP role and service.
Get-WindowsFeature DHCP |
  Tee-Object "$EvidencePath\local-dhcp-role-state.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\local-dhcp-service-state.txt"

# Local network configuration.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\local-net-ip-configuration.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\local-ipconfig-all.txt"

# Authorization list.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Existing failover state.
Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\local-failover-relationships.txt"

# DNS update credential state.
Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\local-dhcp-dns-credential.txt"

# Time sync.
w32tm /query /status |
  Tee-Object "$EvidencePath\local-time-sync-status.txt"

# Connectivity between partners.
Test-NetConnection $PrimaryDhcpServer |
  Tee-Object "$EvidencePath\test-primary-reachability.txt"

Test-NetConnection $PartnerDhcpServer |
  Tee-Object "$EvidencePath\test-partner-reachability.txt"

# DNS name resolution for partners.
Resolve-DnsName $PrimaryDhcpServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-primary-dhcp-server.txt"

Resolve-DnsName $PartnerDhcpServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-partner-dhcp-server.txt"

# DHCP events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-operational-events-readiness.txt"
```

# Configure_DHCPv4_Failover_Load_Balance_Mode_Skeleton
```powershell
# Run in elevated PowerShell on the primary DHCP server.
# Purpose: create DHCPv4 failover in load balance mode.
# Both servers actively lease addresses for the selected scope.

$FailoverName = "DHCP1-DHCP2-LB"
$PartnerDhcpServer = "DHCP2.corp.local"
$ScopeId = "10.20.20.0"
$SharedSecret = Read-Host -Prompt "Enter DHCP failover shared secret"
$EvidencePath = "C:\DHCPv4-Failover\Load-Balance"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\load-balance-failover-config-transcript.txt"

Import-Module DhcpServer

# Capture existing failover state.
Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\failover-before.txt"

# Capture target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-before-failover.txt"

# Create load balance failover relationship.
Add-DhcpServerv4Failover `
  -Name $FailoverName `
  -PartnerServer $PartnerDhcpServer `
  -ScopeId $ScopeId `
  -SharedSecret $SharedSecret `
  -Mode LoadBalance `
  -LoadBalancePercent 50 `
  -MaxClientLeadTime 01:00:00 `
  -AutoStateTransition $true `
  -StateSwitchInterval 01:00:00

# Confirm failover relationship.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-after-create.txt"

# Confirm failover scope mapping.
Get-DhcpServerv4FailoverScope -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-scopes-after-create.txt"

# Force replication after creation.
Invoke-DhcpServerv4FailoverReplication `
  -Name $FailoverName `
  -Force |
  Tee-Object "$EvidencePath\failover-replication-output.txt"

# Confirm final relationship state.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-final.txt"

Stop-Transcript
```

# Configure_DHCPv4_Failover_Hot_Standby_Mode_Skeleton
```powershell
# Run in elevated PowerShell on the primary DHCP server.
# Purpose: create DHCPv4 failover in hot standby mode.
# Primary serves clients; standby holds reserved capacity and takes over if primary fails.

$FailoverName = "DHCP1-DHCP2-HotStandby"
$PartnerDhcpServer = "DHCP2.corp.local"
$ScopeId = "10.30.30.0"
$SharedSecret = Read-Host -Prompt "Enter DHCP failover shared secret"
$EvidencePath = "C:\DHCPv4-Failover\Hot-Standby"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\hot-standby-failover-config-transcript.txt"

Import-Module DhcpServer

# Capture existing failover state.
Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\failover-before.txt"

# Capture target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-before-failover.txt"

# Create hot standby failover relationship.
Add-DhcpServerv4Failover `
  -Name $FailoverName `
  -PartnerServer $PartnerDhcpServer `
  -ScopeId $ScopeId `
  -SharedSecret $SharedSecret `
  -Mode HotStandby `
  -ReservePercent 5 `
  -MaxClientLeadTime 01:00:00 `
  -AutoStateTransition $true `
  -StateSwitchInterval 01:00:00

# Confirm failover relationship.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-after-create.txt"

# Confirm failover scope mapping.
Get-DhcpServerv4FailoverScope -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-scopes-after-create.txt"

# Force replication after creation.
Invoke-DhcpServerv4FailoverReplication `
  -Name $FailoverName `
  -Force |
  Tee-Object "$EvidencePath\failover-replication-output.txt"

# Confirm final relationship state.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-final.txt"

Stop-Transcript
```

# Configure_DHCPv4_Failover_Add_Scope_To_Existing_Relationship_Skeleton
```powershell
# Run in elevated PowerShell on the primary DHCP server.
# Purpose: add another DHCPv4 scope to an existing failover relationship.

$FailoverName = "DHCP1-DHCP2-LB"
$NewScopeId = "10.40.40.0"
$EvidencePath = "C:\DHCPv4-Failover\Add-Scope"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\add-scope-to-failover-transcript.txt"

Import-Module DhcpServer

# Capture existing failover relationship.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-before-add-scope.txt"

# Capture existing failover scopes.
Get-DhcpServerv4FailoverScope -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-scopes-before-add.txt"

# Confirm new scope exists and is ready.
Get-DhcpServerv4Scope -ScopeId $NewScopeId |
  Tee-Object "$EvidencePath\new-scope-before-add.txt"

Get-DhcpServerv4OptionValue -ScopeId $NewScopeId |
  Tee-Object "$EvidencePath\new-scope-options-before-add.txt"

# Add scope to failover relationship.
Add-DhcpServerv4FailoverScope `
  -Name $FailoverName `
  -ScopeId $NewScopeId

# Replicate failover data.
Invoke-DhcpServerv4FailoverReplication `
  -Name $FailoverName `
  -Force |
  Tee-Object "$EvidencePath\failover-replication-after-add-scope.txt"

# Confirm scope membership.
Get-DhcpServerv4FailoverScope -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-scopes-after-add.txt"

Stop-Transcript
```

# Configure_DHCPv4_Failover_Replication_Skeleton
```powershell
# Run in elevated PowerShell on either DHCP failover partner.
# Purpose: manually trigger and validate DHCP failover replication.

$FailoverName = "DHCP1-DHCP2-LB"
$ScopeId = "10.20.20.0"
$PartnerDhcpServer = "DHCP2.corp.local"
$EvidencePath = "C:\DHCPv4-Failover\Replication"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Capture failover relationship state before replication.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-before-replication.txt"

# Capture failover scopes.
Get-DhcpServerv4FailoverScope -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-scopes-before-replication.txt"

# Capture local scope configuration.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\local-scope-before-replication.txt"

Get-DhcpServerv4OptionValue -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\local-scope-options-before-replication.txt"

Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\local-reservations-before-replication.txt"

# Trigger replication.
Invoke-DhcpServerv4FailoverReplication `
  -Name $FailoverName `
  -Force |
  Tee-Object "$EvidencePath\invoke-failover-replication-output.txt"

# Validate relationship after replication.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-after-replication.txt"

# Validate partner scope state remotely if permitted.
Get-DhcpServerv4Scope `
  -ComputerName $PartnerDhcpServer `
  -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\partner-scope-after-replication.txt"

Get-DhcpServerv4OptionValue `
  -ComputerName $PartnerDhcpServer `
  -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\partner-scope-options-after-replication.txt"

Get-DhcpServerv4Reservation `
  -ComputerName $PartnerDhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\partner-reservations-after-replication.txt"
```

# Configure_DHCPv4_Failover_Client_Lease_Validation_Skeleton
```powershell
# Run in elevated PowerShell on a DHCP client in the failover-protected scope.
# Purpose: verify client can renew lease and receive correct options after failover relationship is configured.

$ExpectedSubnetPrefix = "10.20.20."
$ExpectedGateway = "10.20.20.1"
$ExpectedDnsServer1 = "10.10.10.10"
$ExpectedDnsServer2 = "10.10.10.11"
$DomainFqdn = "corp.local"
$EvidencePath = "C:\DHCPv4-Failover\Client-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture client identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$EvidencePath\computer-domain-state.txt"

# Capture initial IP state.
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

# Validate DNS.
Test-NetConnection $ExpectedDnsServer1 -Port 53 |
  Tee-Object "$EvidencePath\test-dns-server-1-tcp-53.txt"

Test-NetConnection $ExpectedDnsServer2 -Port 53 |
  Tee-Object "$EvidencePath\test-dns-server-2-tcp-53.txt"

Resolve-DnsName $DomainFqdn -Server $ExpectedDnsServer1 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-domain-fqdn.txt"

# DHCP client logs.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-client-admin-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Client/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-client-operational-events.txt"
```

# Configure_DHCPv4_Failover_Partner_Failure_Test_Skeleton
```powershell
# Run only in lab or an approved maintenance window.
# Purpose: validate client lease behavior when one DHCP partner is unavailable.
# Do not stop production DHCP services without approval.

$FailoverName = "DHCP1-DHCP2-LB"
$ScopeId = "10.20.20.0"
$EvidencePath = "C:\DHCPv4-Failover\Partner-Failure-Test"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Capture failover state before any failure test.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-before-failure-test.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\leases-before-failure-test.txt"

# Lab-only controlled action:
# Stop DHCP service on one partner from its own console or through approved remote admin.
# Stop-Service DHCPServer

# Wait and observe failover state.
# Start-Sleep -Seconds 60

Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-during-failure-test.txt"

# Renew a client lease during the test from the client machine:
# ipconfig /release
# ipconfig /renew
# ipconfig /all

# Capture leases during failure test.
Get-DhcpServerv4Lease -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\leases-during-failure-test.txt"

# Lab-only restore action:
# Start-Service DHCPServer

# Trigger replication after service restoration.
# Invoke-DhcpServerv4FailoverReplication -Name $FailoverName -Force

# Capture final state after recovery.
Get-DhcpServerv4Failover -Name $FailoverName |
  Tee-Object "$EvidencePath\failover-after-failure-test.txt"
```

# Configure_DHCPv4_Failover_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on both DHCP servers.
# Purpose: collect failover, replication, lease, service, and authorization event evidence.

$EvidencePath = "C:\DHCPv4-Failover\Events"
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

# System events related to DHCP service and connectivity.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DHCP*" -or
    $_.Message -like "*DHCP*" -or
    $_.Message -like "*failover*" -or
    $_.Message -like "*replication*" -or
    $_.Message -like "*lease*"
  } |
  Tee-Object "$EvidencePath\system-dhcp-failover-events-last-24h.txt"

# Final DHCP service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"

# Final failover state.
Import-Module DhcpServer

Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\failover-final-state.txt"
```

# Configure_DHCPv4_Failover_Verification_Commands
```powershell
# DHCP role and service.
Get-WindowsFeature DHCP
Get-Service DHCPServer
Import-Module DhcpServer

# Authorization.
Get-DhcpServerInDC

# Existing scopes and leases.
Get-DhcpServerv4Scope
Get-DhcpServerv4Scope -ScopeId "<scope-id>"
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>"

# DNS credential consistency.
Get-DhcpServerDnsCredential
Set-DhcpServerDnsCredential -Credential (Get-Credential)

# Existing failover relationships.
Get-DhcpServerv4Failover
Get-DhcpServerv4Failover -Name "<failover-name>"
Get-DhcpServerv4FailoverScope -Name "<failover-name>"

# Create load balance failover.
Add-DhcpServerv4Failover `
  -Name "<failover-name>" `
  -PartnerServer "<partner-dhcp-server>" `
  -ScopeId "<scope-id>" `
  -SharedSecret "<shared-secret>" `
  -Mode LoadBalance `
  -LoadBalancePercent 50 `
  -MaxClientLeadTime 01:00:00 `
  -AutoStateTransition $true `
  -StateSwitchInterval 01:00:00

# Create hot standby failover.
Add-DhcpServerv4Failover `
  -Name "<failover-name>" `
  -PartnerServer "<partner-dhcp-server>" `
  -ScopeId "<scope-id>" `
  -SharedSecret "<shared-secret>" `
  -Mode HotStandby `
  -ReservePercent 5 `
  -MaxClientLeadTime 01:00:00 `
  -AutoStateTransition $true `
  -StateSwitchInterval 01:00:00

# Add scope to existing failover relationship.
Add-DhcpServerv4FailoverScope `
  -Name "<failover-name>" `
  -ScopeId "<scope-id>"

# Replicate failover data.
Invoke-DhcpServerv4FailoverReplication `
  -Name "<failover-name>" `
  -Force

# Remove failover relationship.
Remove-DhcpServerv4Failover `
  -Name "<failover-name>" `
  -Force

# Partner remote validation.
Get-DhcpServerv4Scope -ComputerName "<partner-dhcp-server>" -ScopeId "<scope-id>"
Get-DhcpServerv4OptionValue -ComputerName "<partner-dhcp-server>" -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ComputerName "<partner-dhcp-server>" -ScopeId "<scope-id>"
Get-DhcpServerv4Failover -ComputerName "<partner-dhcp-server>"

# Connectivity and name resolution.
Test-NetConnection "<partner-dhcp-server>"
Resolve-DnsName "<partner-dhcp-server>"
w32tm /query /status

# Client validation.
ipconfig /release
ipconfig /renew
ipconfig /all
Get-NetIPConfiguration

# DHCP events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DHCP*" -or $_.Message -like "*failover*"}
```

# Configure_DHCPv4_Failover_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Failover relationship created incorrectly | `Remove-DhcpServerv4Failover -Name "<failover-name>" -Force` | Scope ownership/lease behavior changes after removal |
| Wrong partner server selected | Remove relationship and recreate with correct partner | Leases may not replicate to intended server |
| Wrong scope added to relationship | Remove failover relationship or remove scope from relationship if supported by environment | Scope may remain protected by wrong relationship |
| Wrong failover mode selected | Remove and recreate relationship with correct mode | Client lease behavior may differ during outage |
| Wrong load balance percentage | `Set-DhcpServerv4Failover` with corrected percentage where supported | Lease distribution may be uneven |
| Wrong reserve percent in hot standby | Update or recreate relationship with correct reserve percent | Standby capacity may be too small |
| Shared secret wrong | Recreate or reset relationship with correct shared secret | Partners may fail authentication |
| Auto transition too aggressive | Adjust state transition settings | Server may enter partner-down too quickly |
| MCLT too long or short | Update during approved window | Lease extension behavior may not match design |
| DHCP DNS credential mismatch | Reapply same credential on both partners | DNS records may be inconsistently owned |
| Partner service stopped during test | `Start-Service DHCPServer` | DHCP availability reduced until service returns |
| Evidence folder created | `Remove-Item C:\DHCPv4-Failover -Recurse -Force` | Deletes validation evidence |

# Configure_DHCPv4_Failover_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Failover relationship creation fails | Partner reachability, authorization, existing relationship, or shared secret issue | `Test-NetConnection`; `Get-DhcpServerInDC`; existing failover state | Reinstalling DHCP |
| Partner not reachable | DNS, routing, firewall, or service issue | `Resolve-DnsName`; `Test-NetConnection`; service state | Recreating scope |
| Scope cannot be added | Scope already in another relationship or scope missing | `Get-DhcpServerv4FailoverScope`; `Get-DhcpServerv4Scope` | Removing all scopes |
| Partner does not show scope | Replication issue | `Invoke-DhcpServerv4FailoverReplication`; partner-side `Get-DhcpServerv4Scope` | Manually recreating scope on partner |
| Leases do not replicate | Failover communication or state issue | Failover state and DHCP event logs | Editing client settings |
| Options differ between partners | Replication not run or scope option drift | Compare `Get-DhcpServerv4OptionValue` on both servers | Changing reservations first |
| Reservations missing on partner | Replication issue | Trigger replication and compare reservations | Recreating failover relationship immediately |
| Clients fail when one server is down | Relay only points to failed server or failover state issue | IP helper addresses and failover state | Blaming client OS |
| Clients get APIPA during test | Relay/path/firewall/scope issue | Client logs, relay config, scope state | Editing DNS update settings |
| Duplicate leases appear | Incorrect failover state or independent scopes | Failover relationship and lease state | Expanding scope |
| External DNS records wrong after failover | DHCP DNS credential mismatch | `Get-DhcpServerDnsCredential` on both partners | Editing DNS zone first |
| Partner enters communication interrupted | Network path or service issue | Event logs and partner reachability | Removing relationship immediately |
| Partner enters partner down too early | Auto state transition settings too aggressive | `Get-DhcpServerv4Failover` timers | Restarting both servers |
| Hot standby leases too few | Reserve percent too small | Failover mode and reserve percent | Expanding scope blindly |
| Load balance uneven | Load balance percent or client distribution | Relationship mode and percent | Rebuilding scope |
| Failover state stuck | Replication/state issue | Event logs and manual replication | Rebooting servers first |
| Cannot remove relationship | Active state or permission issue | Elevated PowerShell and relationship name | Deleting DHCP role |
| DHCP service running but no leases | Scope inactive, relay path, failover state, or exhaustion | Scope state/statistics and relay path | Recreating failover first |
| Time skew warnings | Time sync issue between partners | `w32tm /query /status` | Changing DHCP options |
| Failover port blocked | Firewall/network policy issue | Firewall rules and partner communication events | Editing scope ranges |

# Configure_DHCPv4_Failover_Related_Labs
| Lab                                                | Related Workbook                                         | Skill Proven                                                        |
| -------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------- |
| Install DHCP Server role and management tools      | `01_Install_DHCP_Server_Role_And_Management_Tools.md`    | DHCP role baseline                                                  |
| Authorize DHCP Server in Active Directory          | `02_Authorize_DHCP_Server_In_Active_Directory.md`        | AD-authorized DHCP partners                                         |
| Create DHCPv4 scope                                | `03_Create_DHCPv4_Scope.md`                              | Scope readiness for failover                                        |
| Configure DHCPv4 scope options                     | `04_Configure_DHCPv4_Scope_Options.md`                   | Scope option replication validation                                 |
| Configure DHCPv4 exclusions and reservations       | `05_Configure_DHCPv4_Exclusions_And_Reservations.md`     | Reservation and exclusion replication validation                    |
| Validate DHCP client lease assignment              | `06_Validate_DHCP_Client_Lease_Assignment.md`            | Lease verification before and after failover                        |
| Configure DHCP relay and IP helper                 | `07_Configure_DHCP_Relay_And_IP_Helper.md`               | Relay path to DHCP failover pair                                    |
| Configure DHCP DNS dynamic updates                 | `08_Configure_DHCP_DNS_Dynamic_Updates.md`               | Matching DNS update credentials across partners                     |
| Configure DHCPv4 failover                          | `09_Configure_DHCPv4_Failover.md`                        | DHCP high availability through load balance or hot standby failover |
| Backup, restore, and export DHCP Server config     | `10_Backup_Restore_And_Export_DHCP_Server_Config.md`     | Failover backup and recovery readiness                              |
| Monitor DHCP leases, events, and scope utilization | `11_Monitor_DHCP_Leases_Events_And_Scope_Utilization.md` | Failover lease and event visibility                                 |
| Troubleshoot DHCP client lease failures            | `12_Troubleshoot_DHCP_Client_Lease_Failures.md`          | Scope, relay, partner, and client triage                            |
| Troubleshoot DHCP failover and replication         | `17_Troubleshoot_DHCP_Failover_And_Replication.md`       | Partner communication, replication, and failover state triage       |