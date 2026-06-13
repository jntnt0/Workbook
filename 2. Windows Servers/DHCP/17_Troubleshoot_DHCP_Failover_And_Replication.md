17_Troubleshoot_DHCP_Failover_And_Replication.md
# 17_Troubleshoot_DHCP_Failover_And_Replication

# Troubleshoot_DHCP_Failover_And_Replication_Index
17_Troubleshoot_DHCP_Failover_And_Replication.md
Troubleshoot_DHCP_Failover_And_Replication
Troubleshoot_DHCP_Failover_And_Replication_Source_Basis
Troubleshoot_DHCP_Failover_And_Replication_Mental_Model
Troubleshoot_DHCP_Failover_And_Replication_State_Map
Troubleshoot_DHCP_Failover_And_Replication_Planning_Table
Troubleshoot_DHCP_Failover_And_Replication_Configuration_Checklist
Troubleshoot_DHCP_Failover_And_Replication_Precheck_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Relationship_Health_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Network_Path_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Scope_Membership_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Manual_Replication_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Lease_Consistency_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Partner_Down_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Recovery_Skeleton
Troubleshoot_DHCP_Failover_And_Replication_Verification_Commands
Troubleshoot_DHCP_Failover_And_Replication_Rollback
Troubleshoot_DHCP_Failover_And_Replication_Failure_Checks
Troubleshoot_DHCP_Failover_And_Replication_Related_Labs

# Troubleshoot_DHCP_Failover_And_Replication_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Get-DhcpServerv4Failover | Viewing failover relationships by name or by included scope |
| Microsoft Learn | Set-DhcpServerv4Failover | Modifying failover relationship parameters, including load-balance percent, MCLT, auto state transition, state switch interval, mode, and Partner Down |
| Microsoft Learn | Invoke-DhcpServerv4FailoverReplication | Manually replicating scope configuration to the partner DHCP server |
| Microsoft Learn | Add-DhcpServerv4FailoverScope | Adding existing scopes to an existing failover relationship |
| Microsoft Learn | Remove-DhcpServerv4FailoverScope | Removing scopes from a failover relationship while retaining them on the selected server |
| Microsoft Learn | Remove-DhcpServerv4Failover | Removing an entire failover relationship |
| Microsoft Learn | Repair-DhcpServerv4IPRecord | Reporting and repairing inconsistent lease records in the DHCP database |
| Microsoft Learn | Get-DhcpServerv4Lease | Comparing leases and bad leases across failover partners |
| Microsoft Learn | Get-DhcpServerv4ScopeStatistics | Checking utilization and free-address behavior during failover issues |
| Windows Server operational practice | Event Viewer, DHCP audit logs, service checks, network port checks | Validating relationship state, partner reachability, scope state, and lease behavior |

# Troubleshoot_DHCP_Failover_And_Replication_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP failover | IPv4 DHCP high availability relationship between two Windows DHCP servers |
| Relationship | Named failover pairing between two DHCP servers for one or more scopes |
| Partner server | The other DHCP server in the failover relationship |
| Load balance mode | Both servers actively serve clients according to load-balance percentage |
| Hot standby mode | One server actively serves clients while the standby keeps synchronized state |
| MCLT | Maximum Client Lead Time, limits how far one partner can extend leases without the other |
| State switch interval | Timer used for automatic transition from communication interrupted to partner down |
| Auto state transition | Whether the server can automatically move to Partner Down after timer expiration |
| Communication interrupted | Partners cannot communicate, but both may still exist |
| Partner down | Administrative or automatic state where one partner treats the other as unavailable |
| Replication | Manual replication of scope configuration, not a magic fix for network failure |
| Scope membership | Failover only protects scopes that are included in the relationship |
| Lease consistency | Lease records can become inconsistent and may need report-only reconciliation first |
| DHCP failover port | DHCP failover communication commonly depends on partner-to-partner reachability over TCP 647 |
| Split brain risk | Both servers thinking the other is down can create lease conflict risk |
| First rule | Prove partner reachability and relationship state before changing failover settings |
| Second rule | Do not force Partner Down unless the partner is actually unavailable or intentionally isolated |
| Third rule | Replicate configuration after scope/option/reservation/policy changes, but do not confuse replication with failover state repair |
| Fourth rule | DHCP failover is IPv4-focused; do not assume DHCPv6 is covered by these v4 failover cmdlets |

# Troubleshoot_DHCP_Failover_And_Replication_State_Map
| State / Symptom | Meaning | First Check | Safe Action |
|---|---|---|---|
| Normal | Partners communicate and relationship is healthy | `Get-DhcpServerv4Failover` from both servers | No failover repair needed |
| CommunicationInterrupted | Partner communication is broken | TCP 647, DNS, routing, firewall, service state | Fix connectivity first |
| PartnerDown | One server treats partner as unavailable | Confirm whether partner is truly down | Recover partner or return relationship to normal |
| Recover | Server is recovering failover state | Relationship state from both sides | Let recovery complete unless stuck |
| RecoverDone | Recovery completed but relationship may need normalization | Relationship state and leases | Confirm both sides agree |
| PotentialConflict | Possible split-brain or lease conflict condition | Lease comparison, event logs, partner state | Stop changing config, collect evidence, reconcile carefully |
| Paused | Relationship paused or not actively serving as expected | Relationship settings and event logs | Resume/repair relationship after verifying cause |
| Shutdown | DHCP service or relationship is down | `Get-Service DHCPServer` | Start service or investigate shutdown |
| Partner unreachable but service running | Network, firewall, name resolution, or port issue | `Test-NetConnection <partner> -Port 647` | Fix path before forcing state |
| Scope missing from relationship | Scope not protected by failover | `Get-DhcpServerv4Failover -ScopeId <scope-id>` | Add scope to relationship if intended |
| Options differ between partners | Config not replicated after changes | Compare option values | Invoke failover replication |
| Reservation differs between partners | Config not replicated after changes | Compare reservations | Invoke failover replication |
| Policy differs between partners | Config not replicated after changes | Compare policies | Invoke failover replication |
| Lease records inconsistent | Database inconsistency | `Repair-DhcpServerv4IPRecord -ReportOnly` | Repair only after evidence review |

# Troubleshoot_DHCP_Failover_And_Replication_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Primary DHCP server | `DHCP1` | `<primary-dhcp-server>` |
| Partner DHCP server | `DHCP2` | `<partner-dhcp-server>` |
| Failover relationship name | `FO-CORP-LAN` | `<failover-name>` |
| Failover mode | `LoadBalance` or `HotStandby` | `<failover-mode>` |
| Scope ID | `10.10.20.0` | `<scope-id>` |
| Protected scopes | `10.10.20.0`, `10.10.30.0` | `<protected-scopes>` |
| Load balance percent | `50` | `<load-balance-percent>` |
| Standby reserve percent | `5` | `<reserve-percent>` |
| MCLT | `01:00:00` | `<mclt>` |
| State switch interval | `01:00:00` | `<state-switch-interval>` |
| Auto state transition | `$true` or `$false` | `<auto-state-transition>` |
| Shared secret used | `Yes/No` | `<shared-secret-used>` |
| DHCP failover port | `TCP 647` | `<failover-port>` |
| DHCP client ports | `UDP 67/68` | `<dhcp-client-ports>` |
| Evidence path | `C:\DHCPPrep\dhcp-failover-triage` | `<evidence-path>` |
| Change type | Relationship repair / replication / scope membership / lease repair | `<change-type>` |
| Rollback plan | Disable change, restore export, or remove scope from relationship | `<rollback-plan>` |

# Troubleshoot_DHCP_Failover_And_Replication_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | Both DHCP servers | `New-Item -ItemType Directory -Force -Path C:\DHCPPrep\dhcp-failover-triage` | Evidence path exists |
| 2 | Import DHCP module | Both DHCP servers | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 3 | Confirm DHCP service on both servers | Both DHCP servers | `Get-Service DHCPServer` | Service is running on both servers |
| 4 | Confirm both servers are authorized | Management host or DC | `Get-DhcpServerInDC` | Both expected DHCP servers are authorized |
| 5 | Confirm name resolution both directions | Both DHCP servers | `Resolve-DnsName <partner-dhcp-server>` | Partner resolves correctly |
| 6 | Confirm partner reachability | Both DHCP servers | `Test-NetConnection <partner-dhcp-server>` | Partner is reachable |
| 7 | Confirm failover port reachability | Both DHCP servers | `Test-NetConnection <partner-dhcp-server> -Port 647` | TCP 647 succeeds |
| 8 | Capture relationship from primary | Primary DHCP server | `Get-DhcpServerv4Failover -ComputerName <primary> -Name <failover-name>` | Primary relationship state is captured |
| 9 | Capture relationship from partner | Partner DHCP server | `Get-DhcpServerv4Failover -ComputerName <partner> -Name <failover-name>` | Partner relationship state is captured |
| 10 | Confirm scope is in relationship | Primary DHCP server | `Get-DhcpServerv4Failover -ComputerName <primary> -ScopeId <scope-id>` | Scope maps to intended relationship |
| 11 | Capture scopes from both servers | Both DHCP servers | `Get-DhcpServerv4Scope -ComputerName <server>` | Scope presence and state are known |
| 12 | Capture scope statistics | Both DHCP servers | `Get-DhcpServerv4ScopeStatistics -ComputerName <server> -ScopeId <scope-id>` | Utilization is visible |
| 13 | Compare scope options | Both DHCP servers | `Get-DhcpServerv4OptionValue -ComputerName <server> -ScopeId <scope-id>` | Options match after replication |
| 14 | Compare reservations | Both DHCP servers | `Get-DhcpServerv4Reservation -ComputerName <server> -ScopeId <scope-id>` | Reservations match after replication |
| 15 | Compare policies | Both DHCP servers | `Get-DhcpServerv4Policy -ComputerName <server> -ScopeId <scope-id>` | Policies match after replication |
| 16 | Invoke replication by relationship if config differs | Primary DHCP server | `Invoke-DhcpServerv4FailoverReplication -ComputerName <primary> -Name <failover-name> -Force` | Scope config replicates to partner |
| 17 | Invoke replication by scope if limited change | Primary DHCP server | `Invoke-DhcpServerv4FailoverReplication -ComputerName <primary> -ScopeId <scope-id> -Force` | Only selected scope config replicates |
| 18 | Report lease inconsistencies first | Both DHCP servers | `Repair-DhcpServerv4IPRecord -ComputerName <server> -ScopeId <scope-id> -ReportOnly` | Inconsistencies are reported without modification |
| 19 | Repair inconsistent lease records only if approved | Affected DHCP server | `Repair-DhcpServerv4IPRecord -ComputerName <server> -ScopeId <scope-id> -Force` | Inconsistent records are reconciled |
| 20 | Add missing scope to relationship if intended | Primary DHCP server | `Add-DhcpServerv4FailoverScope -ComputerName <primary> -Name <failover-name> -ScopeId <scope-id> -PassThru` | Scope becomes failover-protected |
| 21 | Use PartnerDown only after verification | Primary DHCP server | `Set-DhcpServerv4Failover -ComputerName <primary> -Name <failover-name> -PartnerDown -Force` | Primary treats partner as unavailable |
| 22 | Validate after recovery | Both DHCP servers | `Get-DhcpServerv4Failover`; leases; scope statistics; event logs | Relationship returns to expected state |
| 23 | Renew test client | Client | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Client receives expected lease |
| 24 | Capture final evidence | Both DHCP servers | Export relationship, scopes, options, reservations, policies, leases, events | Repair evidence is preserved |

# Troubleshoot_DHCP_Failover_And_Replication_Precheck_Skeleton
~~~powershell
# Run from an elevated PowerShell session on a management host or DHCP server.

$PrimaryDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$FailoverName = "FO-CORP-LAN"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Basic service checks.
Get-Service DHCPServer -ComputerName $PrimaryDhcp |
  Tee-Object "$EvidencePath\precheck-primary-service.txt"

Get-Service DHCPServer -ComputerName $PartnerDhcp |
  Tee-Object "$EvidencePath\precheck-partner-service.txt"

# Authorization check.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\precheck-authorized-dhcp-servers.txt"

# DNS resolution checks.
Resolve-DnsName $PrimaryDhcp |
  Tee-Object "$EvidencePath\precheck-resolve-primary.txt"

Resolve-DnsName $PartnerDhcp |
  Tee-Object "$EvidencePath\precheck-resolve-partner.txt"

# Basic network checks.
Test-NetConnection $PartnerDhcp |
  Tee-Object "$EvidencePath\precheck-primary-to-partner-basic.txt"

Test-NetConnection $PartnerDhcp -Port 647 |
  Tee-Object "$EvidencePath\precheck-primary-to-partner-tcp647.txt"

Test-NetConnection $PrimaryDhcp -Port 647 |
  Tee-Object "$EvidencePath\precheck-partner-to-primary-tcp647.txt"

# Relationship view from both servers.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-primary-failover-relationship.txt"

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-partner-failover-relationship.txt"

# Scope relationship lookup.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-failover-by-scope-primary.txt"

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-failover-by-scope-partner.txt"
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Relationship_Health_Skeleton
~~~powershell
# Run from a DHCP management host.
# Goal: compare relationship state from both sides before changing anything.

$PrimaryDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$FailoverName = "FO-CORP-LAN"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture all relationships from each server.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp |
  Format-List * |
  Tee-Object "$EvidencePath\relationship-all-primary.txt"

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp |
  Format-List * |
  Tee-Object "$EvidencePath\relationship-all-partner.txt"

# Capture target relationship from each server.
$PrimaryRel = Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp `
  -Name $FailoverName

$PartnerRel = Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp `
  -Name $FailoverName

$PrimaryRel |
  Format-List * |
  Tee-Object "$EvidencePath\relationship-target-primary.txt"

$PartnerRel |
  Format-List * |
  Tee-Object "$EvidencePath\relationship-target-partner.txt"

# Quick side-by-side comparison.
[PSCustomObject]@{
  Server = $PrimaryDhcp
  Name = $PrimaryRel.Name
  PartnerServer = $PrimaryRel.PartnerServer
  Mode = $PrimaryRel.Mode
  State = $PrimaryRel.State
  LoadBalancePercent = $PrimaryRel.LoadBalancePercent
  ReservePercent = $PrimaryRel.ReservePercent
  MaxClientLeadTime = $PrimaryRel.MaxClientLeadTime
  StateSwitchInterval = $PrimaryRel.StateSwitchInterval
  AutoStateTransition = $PrimaryRel.AutoStateTransition
} |
  Tee-Object "$EvidencePath\relationship-summary-primary.txt"

[PSCustomObject]@{
  Server = $PartnerDhcp
  Name = $PartnerRel.Name
  PartnerServer = $PartnerRel.PartnerServer
  Mode = $PartnerRel.Mode
  State = $PartnerRel.State
  LoadBalancePercent = $PartnerRel.LoadBalancePercent
  ReservePercent = $PartnerRel.ReservePercent
  MaxClientLeadTime = $PartnerRel.MaxClientLeadTime
  StateSwitchInterval = $PartnerRel.StateSwitchInterval
  AutoStateTransition = $PartnerRel.AutoStateTransition
} |
  Tee-Object "$EvidencePath\relationship-summary-partner.txt"
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Network_Path_Skeleton
~~~powershell
# Run on both DHCP servers.
# Goal: prove partner-to-partner path before changing failover state.

$PrimaryDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS checks.
Resolve-DnsName $PrimaryDhcp |
  Tee-Object "$EvidencePath\network-resolve-primary.txt"

Resolve-DnsName $PartnerDhcp |
  Tee-Object "$EvidencePath\network-resolve-partner.txt"

# Basic reachability.
Test-NetConnection $PrimaryDhcp |
  Tee-Object "$EvidencePath\network-test-primary-basic.txt"

Test-NetConnection $PartnerDhcp |
  Tee-Object "$EvidencePath\network-test-partner-basic.txt"

# DHCP failover protocol reachability.
Test-NetConnection $PrimaryDhcp -Port 647 |
  Tee-Object "$EvidencePath\network-test-primary-tcp647.txt"

Test-NetConnection $PartnerDhcp -Port 647 |
  Tee-Object "$EvidencePath\network-test-partner-tcp647.txt"

# DHCP client service ports are not the same as failover communication,
# but they are useful when clients also fail to lease.
Test-NetConnection $PrimaryDhcp -Port 67 |
  Tee-Object "$EvidencePath\network-test-primary-tcp67.txt"

Test-NetConnection $PartnerDhcp -Port 67 |
  Tee-Object "$EvidencePath\network-test-partner-tcp67.txt"

# Windows firewall rules containing DHCP.
Get-NetFirewallRule |
  Where-Object {
    $_.DisplayName -like "*DHCP*" -or
    $_.Name -like "*DHCP*"
  } |
  Select-Object DisplayName,Enabled,Direction,Action,Profile |
  Tee-Object "$EvidencePath\network-firewall-dhcp-rules.txt"

# Route checks.
Get-NetRoute -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\network-ipv4-routes.txt"

Get-NetIPAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\network-ipv4-addresses.txt"
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Scope_Membership_Skeleton
~~~powershell
# Run from a DHCP management host.
# Goal: confirm the scope is actually protected by the failover relationship.

$PrimaryDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$FailoverName = "FO-CORP-LAN"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Find relationship by scope.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\scope-membership-primary-by-scope.txt"

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\scope-membership-partner-by-scope.txt"

# Confirm scope exists on both servers.
Get-DhcpServerv4Scope `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\scope-primary-details.txt"

Get-DhcpServerv4Scope `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Format-List * |
  Tee-Object "$EvidencePath\scope-partner-details.txt"

# Confirm statistics from both servers.
Get-DhcpServerv4ScopeStatistics `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\scope-primary-statistics.txt"

Get-DhcpServerv4ScopeStatistics `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-partner-statistics.txt"

# If the scope is not part of the relationship and should be,
# add it only after validating that the relationship is healthy.
# The scope should exist on the source server and should not already exist on the partner.

# Add-DhcpServerv4FailoverScope `
#   -ComputerName $PrimaryDhcp `
#   -Name $FailoverName `
#   -ScopeId $ScopeId `
#   -PassThru
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Manual_Replication_Skeleton
~~~powershell
# Run from the DHCP server that has the correct configuration.
# This replicates scope configuration to the partner.
# Replication includes scope properties, reservations, scope option values, and policies.
# It does not fix a broken network path.

$SourceDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$FailoverName = "FO-CORP-LAN"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Pre-replication evidence from source.
Get-DhcpServerv4OptionValue `
  -ComputerName $SourceDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-source-options-before.txt"

Get-DhcpServerv4Reservation `
  -ComputerName $SourceDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-source-reservations-before.txt"

Get-DhcpServerv4Policy `
  -ComputerName $SourceDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-source-policies-before.txt"

# Pre-replication evidence from partner.
Get-DhcpServerv4OptionValue `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-partner-options-before.txt"

Get-DhcpServerv4Reservation `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-partner-reservations-before.txt"

Get-DhcpServerv4Policy `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-partner-policies-before.txt"

# Replicate a specific scope.
Invoke-DhcpServerv4FailoverReplication `
  -ComputerName $SourceDhcp `
  -ScopeId $ScopeId `
  -Force

# Or replicate all scopes in a relationship.
# Invoke-DhcpServerv4FailoverReplication `
#   -ComputerName $SourceDhcp `
#   -Name $FailoverName `
#   -Force

# Post-replication comparison.
Get-DhcpServerv4OptionValue `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-partner-options-after.txt"

Get-DhcpServerv4Reservation `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-partner-reservations-after.txt"

Get-DhcpServerv4Policy `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\replication-partner-policies-after.txt"
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Lease_Consistency_Skeleton
~~~powershell
# Run carefully.
# Always use -ReportOnly before repairing lease records.
# Run against the affected scope.

$PrimaryDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture leases from both servers.
Get-DhcpServerv4Lease `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId `
  -AllLeases `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-primary-all-before.txt"

Get-DhcpServerv4Lease `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -AllLeases `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-partner-all-before.txt"

# Capture bad leases.
Get-DhcpServerv4Lease `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId `
  -BadLeases `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-primary-bad-before.txt"

Get-DhcpServerv4Lease `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -BadLeases `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\lease-partner-bad-before.txt"

# Report inconsistent records without changing them.
Repair-DhcpServerv4IPRecord `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId `
  -ReportOnly |
  Tee-Object "$EvidencePath\lease-primary-repair-reportonly.txt"

Repair-DhcpServerv4IPRecord `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ReportOnly |
  Tee-Object "$EvidencePath\lease-partner-repair-reportonly.txt"

# Repair only after review and approval.
# Repair-DhcpServerv4IPRecord `
#   -ComputerName $PrimaryDhcp `
#   -ScopeId $ScopeId `
#   -Force

# Repair-DhcpServerv4IPRecord `
#   -ComputerName $PartnerDhcp `
#   -ScopeId $ScopeId `
#   -Force
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Partner_Down_Skeleton
~~~powershell
# Use this only when the partner is truly down, isolated, or intentionally unavailable.
# Do not use PartnerDown just because replication failed.
# PartnerDown can increase split-brain risk if both servers are alive but cannot communicate.

$SurvivingDhcp = "DHCP1"
$FailedPartner = "DHCP2"
$FailoverName = "FO-CORP-LAN"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Prove the partner is unreachable.
Resolve-DnsName $FailedPartner -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\partnerdown-resolve-partner.txt"

Test-NetConnection $FailedPartner |
  Tee-Object "$EvidencePath\partnerdown-basic-reachability.txt"

Test-NetConnection $FailedPartner -Port 647 |
  Tee-Object "$EvidencePath\partnerdown-tcp647.txt"

# Capture current relationship state.
Get-DhcpServerv4Failover `
  -ComputerName $SurvivingDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\partnerdown-relationship-before.txt"

# Move relationship to Partner Down only after proving this is the intended action.
Set-DhcpServerv4Failover `
  -ComputerName $SurvivingDhcp `
  -Name $FailoverName `
  -PartnerDown `
  -Force `
  -PassThru |
  Tee-Object "$EvidencePath\partnerdown-relationship-after.txt"

# Validate after change.
Get-DhcpServerv4Failover `
  -ComputerName $SurvivingDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\partnerdown-relationship-verify.txt"
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Recovery_Skeleton
~~~powershell
# Run when a failed partner returns.
# Goal: confirm both services are online, communication works, relationship state normalizes, and config is replicated.

$PrimaryDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$FailoverName = "FO-CORP-LAN"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm services.
Get-Service DHCPServer -ComputerName $PrimaryDhcp |
  Tee-Object "$EvidencePath\recovery-primary-service.txt"

Get-Service DHCPServer -ComputerName $PartnerDhcp |
  Tee-Object "$EvidencePath\recovery-partner-service.txt"

# Confirm partner-to-partner communication.
Test-NetConnection $PartnerDhcp -Port 647 |
  Tee-Object "$EvidencePath\recovery-primary-to-partner-tcp647.txt"

Test-NetConnection $PrimaryDhcp -Port 647 |
  Tee-Object "$EvidencePath\recovery-partner-to-primary-tcp647.txt"

# Capture current relationship state from both sides.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\recovery-primary-relationship-before.txt"

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\recovery-partner-relationship-before.txt"

# Replicate known-good scope configuration from the authoritative/source server.
Invoke-DhcpServerv4FailoverReplication `
  -ComputerName $PrimaryDhcp `
  -Name $FailoverName `
  -Force

# Re-check relationship state.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\recovery-primary-relationship-after.txt"

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp `
  -Name $FailoverName |
  Format-List * |
  Tee-Object "$EvidencePath\recovery-partner-relationship-after.txt"

# Check leases and scope utilization.
Get-DhcpServerv4ScopeStatistics `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\recovery-primary-scope-statistics.txt"

Get-DhcpServerv4ScopeStatistics `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\recovery-partner-scope-statistics.txt"

Get-DhcpServerv4Lease `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId `
  -AllLeases |
  Tee-Object "$EvidencePath\recovery-primary-leases.txt"

Get-DhcpServerv4Lease `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -AllLeases |
  Tee-Object "$EvidencePath\recovery-partner-leases.txt"
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Verification_Commands
~~~powershell
# Relationship checks.

Import-Module DhcpServer

Get-DhcpServerv4Failover `
  -ComputerName <primary-dhcp-server>

Get-DhcpServerv4Failover `
  -ComputerName <partner-dhcp-server>

Get-DhcpServerv4Failover `
  -ComputerName <primary-dhcp-server> `
  -Name <failover-name> |
  Format-List *

Get-DhcpServerv4Failover `
  -ComputerName <partner-dhcp-server> `
  -Name <failover-name> |
  Format-List *

Get-DhcpServerv4Failover `
  -ComputerName <primary-dhcp-server> `
  -ScopeId <scope-id>

# Service and authorization.

Get-Service DHCPServer -ComputerName <primary-dhcp-server>
Get-Service DHCPServer -ComputerName <partner-dhcp-server>
Get-DhcpServerInDC

# Network path.

Resolve-DnsName <primary-dhcp-server>
Resolve-DnsName <partner-dhcp-server>
Test-NetConnection <partner-dhcp-server> -Port 647
Test-NetConnection <primary-dhcp-server> -Port 647

# Scope, option, reservation, policy comparison.

Get-DhcpServerv4Scope -ComputerName <primary-dhcp-server>
Get-DhcpServerv4Scope -ComputerName <partner-dhcp-server>

Get-DhcpServerv4ScopeStatistics `
  -ComputerName <primary-dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4ScopeStatistics `
  -ComputerName <partner-dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4OptionValue `
  -ComputerName <primary-dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4OptionValue `
  -ComputerName <partner-dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4Reservation `
  -ComputerName <primary-dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4Reservation `
  -ComputerName <partner-dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4Policy `
  -ComputerName <primary-dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4Policy `
  -ComputerName <partner-dhcp-server> `
  -ScopeId <scope-id>

# Manual replication.

Invoke-DhcpServerv4FailoverReplication `
  -ComputerName <source-dhcp-server> `
  -Name <failover-name> `
  -Force

Invoke-DhcpServerv4FailoverReplication `
  -ComputerName <source-dhcp-server> `
  -ScopeId <scope-id> `
  -Force

# Lease consistency.

Get-DhcpServerv4Lease `
  -ComputerName <primary-dhcp-server> `
  -ScopeId <scope-id> `
  -AllLeases

Get-DhcpServerv4Lease `
  -ComputerName <partner-dhcp-server> `
  -ScopeId <scope-id> `
  -AllLeases

Repair-DhcpServerv4IPRecord `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -ReportOnly

# Event logs.

Get-WinEvent `
  -ComputerName <primary-dhcp-server> `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

Get-WinEvent `
  -ComputerName <partner-dhcp-server> `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

Select-String `
  -Path "\\<primary-dhcp-server>\C$\Windows\System32\dhcp\DhcpSrvLog-*.log" `
  -Pattern "<scope-id>","<client-ip>","failover","partner" `
  -ErrorAction SilentlyContinue
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Rollback
~~~powershell
# Run only the rollback matching the change made.
# Always capture current relationship and scope state first.

$PrimaryDhcp = "DHCP1"
$PartnerDhcp = "DHCP2"
$FailoverName = "FO-CORP-LAN"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\dhcp-failover-triage"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current state.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp |
  Export-Csv "$EvidencePath\rollback-primary-failover-before.csv" -NoTypeInformation

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp |
  Export-Csv "$EvidencePath\rollback-partner-failover-before.csv" -NoTypeInformation

Get-DhcpServerv4Scope `
  -ComputerName $PrimaryDhcp |
  Export-Csv "$EvidencePath\rollback-primary-scopes-before.csv" -NoTypeInformation

Get-DhcpServerv4Scope `
  -ComputerName $PartnerDhcp |
  Export-Csv "$EvidencePath\rollback-partner-scopes-before.csv" -NoTypeInformation

Get-DhcpServerv4OptionValue `
  -ComputerName $PrimaryDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-primary-options-before.csv" -NoTypeInformation

Get-DhcpServerv4OptionValue `
  -ComputerName $PartnerDhcp `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-partner-options-before.csv" -NoTypeInformation

# Rollback type 1: undo an accidental scope addition to failover.
# Warning: this removes the scope from the relationship and deletes the scope from the partner server.
# The scope remains on the server specified in -ComputerName.

# Remove-DhcpServerv4FailoverScope `
#   -ComputerName $PrimaryDhcp `
#   -Name $FailoverName `
#   -ScopeId $ScopeId `
#   -Confirm

# Rollback type 2: restore conservative failover timing values.
# Replace values with previously captured settings.

# Set-DhcpServerv4Failover `
#   -ComputerName $PrimaryDhcp `
#   -Name $FailoverName `
#   -MaxClientLeadTime 01:00:00 `
#   -AutoStateTransition $false `
#   -PassThru

# Rollback type 3: restore load-balance percentage.
# Set-DhcpServerv4Failover `
#   -ComputerName $PrimaryDhcp `
#   -Name $FailoverName `
#   -Mode LoadBalance `
#   -LoadBalancePercent 50 `
#   -PassThru

# Rollback type 4: remove entire failover relationship only in lab or rebuild scenario.
# Warning: this is destructive to the failover relationship.

# Remove-DhcpServerv4Failover `
#   -ComputerName $PrimaryDhcp `
#   -Name $FailoverName `
#   -Confirm

# Post-rollback validation.
Get-DhcpServerv4Failover `
  -ComputerName $PrimaryDhcp `
  -ErrorAction SilentlyContinue

Get-DhcpServerv4Failover `
  -ComputerName $PartnerDhcp `
  -ErrorAction SilentlyContinue
~~~

# Troubleshoot_DHCP_Failover_And_Replication_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Relationship stuck in CommunicationInterrupted | Partner path broken | `Test-NetConnection <partner> -Port 647` | Fix DNS, routing, firewall, service, or server availability |
| Relationship says PartnerDown but partner is online | Manual or automatic transition happened while path was broken | Relationship state from both servers | Restore communication and normalize relationship |
| Replication command fails | Partner unreachable or relationship unhealthy | TCP 647 and failover state | Fix partner path before retrying replication |
| Scope options differ between servers | Scope config changed but not replicated | Compare `Get-DhcpServerv4OptionValue` | Run failover replication from correct source |
| Reservations differ between servers | Reservation created on one partner only | Compare reservations | Run failover replication |
| Policies differ between servers | Policy created or edited on one partner only | Compare policies | Run failover replication |
| Scope not protected by failover | Scope never added to relationship | `Get-DhcpServerv4Failover -ScopeId <scope-id>` | Add scope to relationship if intended |
| Scope exists on one partner only | Scope membership or replication issue | Scope list from both servers | Add scope to failover or rebuild from known-good source |
| Clients still lease during partner outage | Expected in failover design | Scope statistics and relationship mode | No action unless lease behavior is wrong |
| Clients cannot lease during partner outage | Surviving server not serving or scope unavailable | Service, relationship state, scope state | Fix service, relationship, scope, or PartnerDown state |
| Hot standby server serving too many clients | Role or state changed | Relationship mode and role | Validate active/standby role and recover failed active |
| Load-balance split wrong | LoadBalancePercent changed | Relationship properties | Set intended load-balance percentage |
| Auto transition happened unexpectedly | AutoStateTransition enabled with short timer | Relationship properties | Adjust state switch interval or disable auto transition |
| Lease conflicts appear | Split-brain or inconsistent records | Event logs, bad leases, repair report | Stop changes, collect evidence, reconcile lease records |
| Repair command would change records | Lease database inconsistency | `Repair-DhcpServerv4IPRecord -ReportOnly` | Review report before repair |
| Event logs show failover auth errors | Shared secret mismatch or relationship mismatch | Relationship properties and event logs | Correct relationship/shared secret design |
| Partner name resolves wrong | DNS stale or wrong record | `Resolve-DnsName` | Fix DNS record and retry partner connectivity |
| Partner path works by IP but not name | DNS resolution issue | Name versus IP tests | Fix DNS suffix, records, or host resolution |
| DHCP works locally but failover is broken | Client DHCP and failover protocol are separate | UDP 67/68 versus TCP 647 checks | Fix failover communication path |
| Removing failover scope deleted partner scope | Expected behavior of remove-scope operation | Partner scope list | Recreate/restore partner scope only if required |
| Relationship rebuild considered too early | Root cause not isolated | Evidence completeness | Export config, identify source of truth, then rebuild only if necessary |

# Troubleshoot_DHCP_Failover_And_Replication_Related_Labs
| Lab                                                        | Relationship                                                                          |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 04_Activate_Authorize_And_Verify_DHCP_Server               | Failover depends on both DHCP servers being authorized and healthy                    |
| 09_Configure_DHCPv4_Failover                               | Baseline failover relationship creation and expected state                            |
| 10_Backup_Restore_And_Export_DHCP_Server_Config            | Export evidence and restore paths before destructive failover repair                  |
| 11_Monitor_DHCP_Leases_Events_And_Scope_Utilization        | Scope statistics, event logs, and lease visibility are core failover evidence         |
| 12_Troubleshoot_DHCP_Client_Lease_Failures                 | Client lease symptoms are often how failover problems are discovered                  |
| 14_Configure_DHCP_Policies_User_Classes_And_Vendor_Classes | Policies must replicate between partners when scopes are in failover                  |
| 15_Configure_DHCP_Superscopes_And_Multisubnet_Design       | Multiscope designs must be checked carefully before failover scope membership changes |