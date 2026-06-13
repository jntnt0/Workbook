00_DHCP_Index.md
# DHCP_Index

# DHCP_Index_Index
00_DHCP_Index.md
DHCP_Index
DHCP_Index_Source_Basis
DHCP_Index_Mental_Model
DHCP_Index_Workbook_Map
DHCP_Index_Operational_Build_Order
DHCP_Index_Planning_Table
DHCP_Index_Configuration_Checklist
DHCP_Index_Core_Validation_Skeleton
DHCP_Index_Client_Validation_Skeleton
DHCP_Index_Server_Inventory_Skeleton
DHCP_Index_Verification_Commands
DHCP_Index_Rollback
DHCP_Index_Failure_Checks
DHCP_Index_Related_Labs

# DHCP_Index_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install-WindowsFeature | Installing the DHCP Server role and management tools |
| Microsoft Learn | Add-DhcpServerInDC | Authorizing a DHCP server in Active Directory |
| Microsoft Learn | Add-DhcpServerv4Scope | Creating IPv4 DHCP scopes |
| Microsoft Learn | Set-DhcpServerv4OptionValue | Configuring DHCP scope/server options |
| Microsoft Learn | Add-DhcpServerv4ExclusionRange | Creating exclusion ranges inside DHCP scopes |
| Microsoft Learn | Add-DhcpServerv4Reservation | Creating DHCP reservations |
| Microsoft Learn | Get-DhcpServerv4Lease | Verifying DHCP lease assignment |
| Microsoft Learn | Set-DhcpServerv4DnsSetting | Configuring DHCP DNS dynamic update behavior |
| Microsoft Learn | Add-DhcpServerv4Failover | Configuring DHCP failover partnerships |
| Microsoft Learn | Backup-DhcpServer / Restore-DhcpServer | Backing up and restoring DHCP server configuration |
| Microsoft Learn | Export-DhcpServer / Import-DhcpServer | Exporting and importing DHCP server configuration |
| Microsoft Learn | Get-DhcpServerv4ScopeStatistics | Monitoring lease utilization and scope health |
| Microsoft Learn | DHCP Server event logs | Troubleshooting service, scope, authorization, DNS, and failover issues |
| Windows Server operational practice | DHCP relay, IP helper, PXE, superscopes, policies, vendor classes | Integrating DHCP with routed networks and enterprise services |

# DHCP_Index_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP | Service that automatically leases IP configuration to clients |
| DHCP server | Windows Server role that owns scopes, leases, reservations, and options |
| DHCP authorization | AD safety control that allows only approved DHCP servers to lease addresses in a domain |
| Scope | A defined IPv4 or IPv6 address pool for one subnet |
| Lease | Temporary IP assignment given to a DHCP client |
| Scope options | Configuration handed to clients, such as gateway, DNS servers, DNS suffix, NTP, or PXE options |
| Exclusion | Address range inside a scope that DHCP will not lease |
| Reservation | Fixed lease mapping between a client MAC address and an IP address |
| Relay agent / IP helper | Router or L3 switch function that forwards DHCP broadcasts across subnets |
| DNS dynamic updates | DHCP server behavior that registers or updates client DNS records |
| DHCP failover | High availability partnership between two DHCP servers for one or more scopes |
| Backup/export | Operational safety mechanism for preserving DHCP scopes, options, reservations, and failover state |
| Scope utilization | Percentage of lease pool currently consumed |
| DHCP policies | Conditional assignment behavior based on user class, vendor class, MAC, relay agent, or other conditions |
| Superscope | Grouping of multiple scopes to support multinets on one physical segment |
| PXE DHCP options | DHCP options used to support network boot workflows |
| First rule | DHCP is simple only when routing, VLANs, DNS, authorization, and scope boundaries are clean |

# DHCP_Index_Workbook_Map
| Order | Workbook | Primary Goal | Required Before | Produces |
|---:|---|---|---|---|
| 00 | `00_DHCP_Index.md` | Map the DHCP workbook suite and operational order | Existing Windows Server / AD lab context | Navigation and build sequence |
| 01 | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | Install DHCP Server role and admin tools | Windows Server prepared with static IP | DHCP role installed |
| 02 | `02_Authorize_DHCP_Server_In_Active_Directory.md` | Authorize DHCP server in AD | Domain-joined server and AD admin rights | Authorized DHCP server |
| 03 | `03_Create_DHCPv4_Scope.md` | Create an IPv4 lease pool | Authorized DHCP server and subnet plan | Active IPv4 scope |
| 04 | `04_Configure_DHCPv4_Scope_Options.md` | Configure gateway, DNS, domain suffix, and lease settings | Existing IPv4 scope | Clients receive usable network config |
| 05 | `05_Configure_DHCPv4_Exclusions_And_Reservations.md` | Protect static IPs and reserve fixed leases | Existing IPv4 scope | Exclusions and reservations |
| 06 | `06_Validate_DHCP_Client_Lease_Assignment.md` | Prove clients can receive DHCP leases | Active scope and test client | Client lease evidence |
| 07 | `07_Configure_DHCP_Relay_And_IP_Helper.md` | Support DHCP across routed subnets | Routed VLAN/subnet and reachable DHCP server | Remote subnet DHCP lease path |
| 08 | `08_Configure_DHCP_DNS_Dynamic_Updates.md` | Integrate DHCP lease assignment with DNS registration | AD DNS and DHCP server | Dynamic DNS update behavior |
| 09 | `09_Configure_DHCPv4_Failover.md` | Add DHCP high availability | Two DHCP servers and existing scope | Failover relationship |
| 10 | `10_Backup_Restore_And_Export_DHCP_Server_Config.md` | Preserve and recover DHCP configuration | Working DHCP server | Backup/export recovery path |
| 11 | `11_Monitor_DHCP_Leases_Events_And_Scope_Utilization.md` | Monitor leases, events, and pool exhaustion risk | Active scopes | Operational visibility |
| 12 | `12_Troubleshoot_DHCP_Client_Lease_Failures.md` | Triage failed DHCP clients | Client, server, network path | Root cause workflow |
| 13 | `13_Configure_DHCPv6_Scopes_And_Options.md` | Configure DHCPv6 behavior | IPv6 addressing plan | IPv6 DHCP support |
| 14 | `14_Configure_DHCP_Policies_User_Classes_And_Vendor_Classes.md` | Apply conditional DHCP behavior | Existing scope and class plan | Policy-based DHCP assignment |
| 15 | `15_Configure_DHCP_Superscopes_And_Multisubnet_Design.md` | Support multinets and grouped scopes | Multiple subnet design | Superscope configuration |
| 16 | `16_Configure_PXE_Boot_DHCP_Options.md` | Support PXE boot clients | PXE/WDS/SCCM boot server details | PXE DHCP options |
| 17 | `17_Troubleshoot_DHCP_Failover_And_Replication.md` | Triage failover state and replication issues | DHCP failover pair | Failover recovery workflow |

# DHCP_Index_Operational_Build_Order
| Phase | Workbooks | Purpose |
|---:|---|---|
| 1 | `00` | Understand suite structure and build order |
| 2 | `01` to `02` | Install and authorize DHCP server |
| 3 | `03` to `05` | Build usable IPv4 scope with options, exclusions, and reservations |
| 4 | `06` | Validate local client lease assignment |
| 5 | `07` | Extend DHCP to routed client subnets |
| 6 | `08` | Integrate DHCP leases with DNS dynamic updates |
| 7 | `09` to `10` | Add failover and backup/export safety |
| 8 | `11` to `12` | Monitor and troubleshoot production-style DHCP issues |
| 9 | `13` to `17` | Add advanced DHCPv6, policies, superscopes, PXE, and failover troubleshooting |

# DHCP_Index_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server name | `DHCP1` | `<dhcp-server-name>` |
| DHCP server IP | `10.10.10.20` | `<dhcp-server-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| DHCP admin account | `CORP\Administrator` or delegated DHCP admin | `<admin-account>` |
| DHCP management host | DHCP server or admin workstation | `<management-host>` |
| Primary IPv4 scope name | `Corp-Users-VLAN10` | `<scope-name>` |
| Scope subnet | `10.10.10.0/24` | `<scope-subnet>` |
| Scope start IP | `10.10.10.50` | `<scope-start>` |
| Scope end IP | `10.10.10.200` | `<scope-end>` |
| Subnet mask | `255.255.255.0` | `<subnet-mask>` |
| Default gateway option | `10.10.10.1` | `<router-option>` |
| DNS server option | `10.10.10.10`, `10.10.10.11` | `<dns-servers>` |
| DNS suffix option | `corp.local` | `<dns-suffix>` |
| Lease duration | `8 days` | `<lease-duration>` |
| Exclusion range | `10.10.10.1` to `10.10.10.49` | `<exclusion-range>` |
| Reservation example | Printer / server / PXE client | `<reservation-target>` |
| Relay interface VLAN | `VLAN 20` | `<relay-vlan>` |
| Relay source subnet | `10.10.20.0/24` | `<relay-subnet>` |
| IP helper target | `10.10.10.20` | `<dhcp-helper-target>` |
| Failover partner | `DHCP2` | `<failover-partner>` |
| Backup path | `C:\DHCP-Backup` | `<backup-path>` |
| Export path | `C:\DHCP-Export\dhcp-export.xml` | `<export-path>` |
| Monitoring threshold | `80% utilization` | `<scope-threshold>` |
| Rollback stance | Lab restore / production change control | `<rollback-plan>` |

# DHCP_Index_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm DHCP server hostname | DHCP Server | `hostname` | Server name is known |
| 2 | Confirm server IP configuration | DHCP Server | `Get-NetIPConfiguration` | Static IP, gateway, and DNS are visible |
| 3 | Confirm domain membership | DHCP Server | `Get-ComputerInfo \| Select-Object CsDomain,CsPartOfDomain` | Server is joined to expected domain if AD-integrated |
| 4 | Install DHCP role | DHCP Server | `Install-WindowsFeature DHCP -IncludeManagementTools` | DHCP role and tools are installed |
| 5 | Confirm DHCP cmdlets exist | DHCP Server | `Get-Command -Module DhcpServer` | DHCP PowerShell module is available |
| 6 | Authorize DHCP in AD | DHCP Server / DC | `Add-DhcpServerInDC -DnsName "<dhcp-server-fqdn>" -IPAddress "<dhcp-server-ip>"` | DHCP server is authorized |
| 7 | Verify authorization | DHCP Server / DC | `Get-DhcpServerInDC` | DHCP server appears in authorized list |
| 8 | Create IPv4 scope | DHCP Server | `Add-DhcpServerv4Scope -Name "<scope-name>" -StartRange "<scope-start>" -EndRange "<scope-end>" -SubnetMask "<subnet-mask>"` | Scope exists |
| 9 | Configure default gateway option | DHCP Server | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -Router "<gateway-ip>"` | Clients receive gateway |
| 10 | Configure DNS server option | DHCP Server | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsServer "<dns-ip-1>","<dns-ip-2>"` | Clients receive DNS servers |
| 11 | Configure DNS suffix option | DHCP Server | `Set-DhcpServerv4OptionValue -ScopeId "<scope-id>" -DnsDomain "<domain-fqdn>"` | Clients receive DNS suffix |
| 12 | Configure exclusions | DHCP Server | `Add-DhcpServerv4ExclusionRange -ScopeId "<scope-id>" -StartRange "<exclude-start>" -EndRange "<exclude-end>"` | Static/reserved ranges are not leased |
| 13 | Configure reservations if needed | DHCP Server | `Add-DhcpServerv4Reservation -ScopeId "<scope-id>" -IPAddress "<reserved-ip>" -ClientId "<mac-address>" -Description "<description>"` | Reserved client receives fixed lease |
| 14 | Validate active scope | DHCP Server | `Get-DhcpServerv4Scope` | Scope is listed and active |
| 15 | Validate scope options | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Required options are present |
| 16 | Validate lease from client | Client | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Client receives expected DHCP lease |
| 17 | Validate leases from server | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Client lease appears |
| 18 | Configure relay if client is remote | Router / L3 Switch | `ip helper-address <dhcp-server-ip>` | DHCP broadcasts are forwarded to server |
| 19 | Validate DNS dynamic update behavior | DHCP Server / DNS | `Get-DhcpServerv4DnsSetting`; `Resolve-DnsName <client-fqdn>` | DNS registration behavior is known |
| 20 | Configure failover if needed | DHCP Server | `Add-DhcpServerv4Failover ...` | Failover relationship exists |
| 21 | Backup DHCP configuration | DHCP Server | `Backup-DhcpServer -Path "<backup-path>"` | DHCP backup exists |
| 22 | Export DHCP configuration | DHCP Server | `Export-DhcpServer -File "<export-path>" -Leases -Force` | DHCP export file exists |
| 23 | Monitor scope utilization | DHCP Server | `Get-DhcpServerv4ScopeStatistics` | Scope usage is visible |
| 24 | Review DHCP events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50` | Recent DHCP events are visible |
| 25 | Document DHCP build | Operator | `Record server, scopes, options, exclusions, reservations, relay, failover, backup path` | Build record is complete |

# DHCP_Index_Core_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: quick server-side DHCP validation.

$DhcpServer = $env:COMPUTERNAME
$ScopeId = "10.10.10.0"
$ValidationPath = "C:\DHCP-Validation"

New-Item -ItemType Directory -Force -Path $ValidationPath

# Confirm server identity and IP configuration.
hostname | Tee-Object "$ValidationPath\hostname.txt"
Get-NetIPConfiguration | Tee-Object "$ValidationPath\net-ip-configuration.txt"

# Confirm DHCP role and tools.
Get-WindowsFeature DHCP | Tee-Object "$ValidationPath\dhcp-role-state.txt"
Get-Command -Module DhcpServer | Tee-Object "$ValidationPath\dhcp-cmdlets.txt"

# Confirm DHCP service.
Get-Service DHCPServer | Tee-Object "$ValidationPath\dhcp-service.txt"

# Confirm AD authorization.
Get-DhcpServerInDC | Tee-Object "$ValidationPath\authorized-dhcp-servers.txt"

# Confirm scopes.
Get-DhcpServerv4Scope | Tee-Object "$ValidationPath\dhcp-scopes.txt"

# Confirm target scope.
Get-DhcpServerv4Scope -ScopeId $ScopeId | Tee-Object "$ValidationPath\scope-$ScopeId.txt"

# Confirm options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId | Tee-Object "$ValidationPath\scope-options-$ScopeId.txt"

# Confirm exclusions.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId | Tee-Object "$ValidationPath\scope-exclusions-$ScopeId.txt"

# Confirm reservations.
Get-DhcpServerv4Reservation -ScopeId $ScopeId | Tee-Object "$ValidationPath\scope-reservations-$ScopeId.txt"

# Confirm leases.
Get-DhcpServerv4Lease -ScopeId $ScopeId | Tee-Object "$ValidationPath\scope-leases-$ScopeId.txt"

# Confirm utilization.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId | Tee-Object "$ValidationPath\scope-statistics-$ScopeId.txt"

# Review recent DHCP Server operational events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 |
  Tee-Object "$ValidationPath\dhcp-operational-events.txt"