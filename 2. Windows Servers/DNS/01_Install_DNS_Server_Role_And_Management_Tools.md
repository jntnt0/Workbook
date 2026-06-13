01_Install_DNS_Server_Role_And_Management_Tools.md
# 01_Install_DNS_Server_Role_And_Management_Tools

# 01_Install_DNS_Server_Role_And_Management_Tools_Index
01_Install_DNS_Server_Role_And_Management_Tools.md
01_Install_DNS_Server_Role_And_Management_Tools
01_Install_DNS_Server_Role_And_Management_Tools_Source_Basis
01_Install_DNS_Server_Role_And_Management_Tools_Mental_Model
01_Install_DNS_Server_Role_And_Management_Tools_Planning_Table
01_Install_DNS_Server_Role_And_Management_Tools_Configuration_Checklist
01_Install_DNS_Server_Role_And_Management_Tools_Local_Server_Install_Skeleton
01_Install_DNS_Server_Role_And_Management_Tools_Remote_Server_Install_Skeleton
01_Install_DNS_Server_Role_And_Management_Tools_Management_Workstation_Tools_Skeleton
01_Install_DNS_Server_Role_And_Management_Tools_Post_Install_Baseline_Skeleton
01_Install_DNS_Server_Role_And_Management_Tools_Verification_Commands
01_Install_DNS_Server_Role_And_Management_Tools_Rollback
01_Install_DNS_Server_Role_And_Management_Tools_Failure_Checks
01_Install_DNS_Server_Role_And_Management_Tools_Related_Labs

# 01_Install_DNS_Server_Role_And_Management_Tools_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install-WindowsFeature | Installing Windows Server roles, features, and management tools |
| Microsoft Learn | Get-WindowsFeature | Confirming available and installed Windows Server roles/features |
| Microsoft Learn | Uninstall-WindowsFeature | Removing the DNS Server role during rollback |
| Microsoft Learn | DnsServer PowerShell module | Managing DNS zones, records, policies, forwarders, DNSSEC, and diagnostics |
| Microsoft Learn | Server Manager | GUI installation path for DNS Server role and management tools |
| Windows Server operational practice | DNS service, firewall rules, event logs, RSAT, and remote management | Post-install validation and supportability baseline |

# 01_Install_DNS_Server_Role_And_Management_Tools_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS Server role | Windows Server role that hosts DNS zones and answers DNS queries |
| DNS management tools | GUI and PowerShell tools used to administer DNS Server |
| Server Manager | GUI tool for installing roles and features |
| Install-WindowsFeature | PowerShell cmdlet used to install server roles and features |
| IncludeManagementTools | Installs management tools along with the role |
| DNS service | Windows service named `DNS` that runs the DNS Server engine |
| DnsServer module | PowerShell module that exposes DNS Server management cmdlets |
| RSAT DNS tools | Remote Server Administration Tools used to manage DNS from another server or admin workstation |
| UDP 53 | Primary DNS query transport |
| TCP 53 | DNS zone transfers, large responses, and fallback query transport |
| DNS console | MMC snap-in used for GUI DNS management |
| Local install | Role installed on the current server |
| Remote install | Role installed on another Windows Server through Server Manager or PowerShell remoting |
| Management host | Admin workstation or jump server used to manage DNS remotely |
| First rule | Install the role and tools before building zones or records |
| Blunt rule | A server is not a functional DNS server just because tools are installed; the DNS role, service, firewall, and listening state must all validate |

# 01_Install_DNS_Server_Role_And_Management_Tools_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Target DNS server | `DC1` | `<dns-server>` |
| Target server role | DNS Server | `<role>` |
| Install method | PowerShell | `<PowerShell / Server Manager>` |
| Include management tools | Yes | `<yes-no>` |
| Management host | `MGMT01` | `<management-host>` |
| Server IP address | `10.10.10.10` | `<dns-server-ip>` |
| DNS client address on server | `127.0.0.1` or `10.10.10.10` | `<dns-client-setting>` |
| Firewall profile | Domain | `<Domain / Private / Public>` |
| Remote management required | Yes | `<yes-no>` |
| Admin credential | Domain admin or delegated DNS admin | `<admin-account>` |
| Initial forward zone | `corp.local` | `<forward-zone>` |
| Initial reverse zone | `10.10.10.in-addr.arpa` | `<reverse-zone>` |
| Event log review | DNS Server log | `<log-source>` |
| Rollback plan | Uninstall role if lab-only | `<rollback-plan>` |

# 01_Install_DNS_Server_Role_And_Management_Tools_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm elevated PowerShell | DNS Server | `whoami /groups` | Session has administrative rights |
| 2 | Confirm OS is Windows Server | DNS Server | `Get-ComputerInfo \| Select-Object WindowsProductName,WindowsVersion,OsBuildNumber` | Target is Windows Server |
| 3 | Confirm server name | DNS Server | `hostname` | Correct server is being configured |
| 4 | Confirm static IP configuration | DNS Server | `Get-NetIPConfiguration` | Server has stable IP address |
| 5 | Confirm DNS role availability | DNS Server | `Get-WindowsFeature -Name DNS` | DNS role is available |
| 6 | Confirm DNS management tools availability | DNS Server | `Get-WindowsFeature -Name RSAT-DNS-Server` | DNS tools feature is available |
| 7 | Preview DNS role install | DNS Server | `Install-WindowsFeature -Name DNS -IncludeManagementTools -WhatIf` | Install preview shows intended role/tools |
| 8 | Install DNS Server role and tools | DNS Server | `Install-WindowsFeature -Name DNS -IncludeManagementTools -Verbose` | DNS role and tools install successfully |
| 9 | Confirm installed state | DNS Server | `Get-WindowsFeature -Name DNS,RSAT-DNS-Server` | DNS and management tools show installed |
| 10 | Confirm DNS service exists | DNS Server | `Get-Service DNS` | DNS service exists |
| 11 | Start DNS service if needed | DNS Server | `Start-Service DNS` | DNS service is running |
| 12 | Set DNS service startup type | DNS Server | `Set-Service DNS -StartupType Automatic` | DNS service starts automatically |
| 13 | Confirm DnsServer module loads | DNS Server | `Import-Module DnsServer; Get-Command -Module DnsServer` | DNS Server cmdlets are available |
| 14 | Open DNS console from server | DNS Server | `dnsmgmt.msc` | DNS Manager opens |
| 15 | Confirm DNS server inventory | DNS Server | `Get-DnsServer -ComputerName "<dns-server>"` | DNS server object responds |
| 16 | Confirm DNS zones baseline | DNS Server | `Get-DnsServerZone -ComputerName "<dns-server>"` | Default zones or empty baseline are visible |
| 17 | Confirm listener ports | DNS Server | `Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue; Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue` | DNS is listening on port 53 |
| 18 | Confirm firewall DNS rules | DNS Server | `Get-NetFirewallRule -DisplayGroup "DNS Server" \| Select-Object DisplayName,Enabled,Profile` | DNS firewall rules are present and enabled |
| 19 | Test local DNS service response | DNS Server | `Resolve-DnsName localhost -Server 127.0.0.1` | Local DNS query succeeds |
| 20 | Review DNS Server event log | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 25` | No critical startup errors |
| 21 | Document install baseline | Operator | `Record server name, IP, install method, role state, tool state, service state, and rollback plan` | Install is supportable |

# 01_Install_DNS_Server_Role_And_Management_Tools_Local_Server_Install_Skeleton
```powershell
# Run in elevated PowerShell on the Windows Server that will host DNS.

$DnsServer = $env:COMPUTERNAME
$InstallLog = "C:\DNS-Install.log"

# Confirm server identity and OS.
hostname
Get-ComputerInfo |
  Select-Object WindowsProductName, WindowsVersion, OsBuildNumber

# Confirm network baseline.
Get-NetIPConfiguration

# Confirm role and tools availability.
Get-WindowsFeature -Name DNS
Get-WindowsFeature -Name RSAT-DNS-Server

# Preview installation.
Install-WindowsFeature `
  -Name DNS `
  -IncludeManagementTools `
  -WhatIf

# Install DNS Server role and management tools.
Install-WindowsFeature `
  -Name DNS `
  -IncludeManagementTools `
  -LogPath $InstallLog `
  -Verbose

# Verify installed state.
Get-WindowsFeature -Name DNS,RSAT-DNS-Server

# Confirm and start DNS service.
Get-Service DNS

Start-Service DNS
Set-Service DNS -StartupType Automatic

# Load DNS Server PowerShell module.
Import-Module DnsServer

# Confirm DNS Server cmdlets are available.
Get-Command -Module DnsServer |
  Select-Object -First 20

# Confirm DNS server responds to management query.
Get-DnsServer -ComputerName $DnsServer

# Show current zones.
Get-DnsServerZone -ComputerName $DnsServer

# Confirm port listeners.
Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue
Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue

# Confirm DNS firewall rule group.
Get-NetFirewallRule -DisplayGroup "DNS Server" |
  Select-Object DisplayName, Enabled, Profile, Direction, Action

# Test local resolution path.
Resolve-DnsName localhost -Server 127.0.0.1

# Review recent DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 25 |
  Select-Object TimeCreated, Id, ProviderName, Message
```

# 01_Install_DNS_Server_Role_And_Management_Tools_Remote_Server_Install_Skeleton
```powershell
# Run in elevated PowerShell from a management server.
# Requires remote management and permissions on the target server.

$TargetServer = "DC1"
$InstallLog = "C:\DNS-Install.log"

# Confirm remote connectivity.
Test-Connection $TargetServer -Count 2
Test-WSMan $TargetServer

# Confirm remote feature state.
Get-WindowsFeature -ComputerName $TargetServer -Name DNS
Get-WindowsFeature -ComputerName $TargetServer -Name RSAT-DNS-Server

# Preview remote install.
Install-WindowsFeature `
  -ComputerName $TargetServer `
  -Name DNS `
  -IncludeManagementTools `
  -WhatIf

# Install DNS Server remotely.
Install-WindowsFeature `
  -ComputerName $TargetServer `
  -Name DNS `
  -IncludeManagementTools `
  -LogPath $InstallLog `
  -Verbose

# Verify remote role state.
Get-WindowsFeature -ComputerName $TargetServer -Name DNS,RSAT-DNS-Server

# Verify remote DNS service.
Get-Service -ComputerName $TargetServer -Name DNS

# Query DNS Server remotely.
Get-DnsServer -ComputerName $TargetServer
Get-DnsServerZone -ComputerName $TargetServer

# Confirm remote firewall rules for DNS.
Invoke-Command -ComputerName $TargetServer -ScriptBlock {
  Get-NetFirewallRule -DisplayGroup "DNS Server" |
    Select-Object DisplayName, Enabled, Profile, Direction, Action
}

# Confirm remote listeners.
Invoke-Command -ComputerName $TargetServer -ScriptBlock {
  Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue
  Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue
}

# Test DNS response from management host.
Resolve-DnsName localhost -Server $TargetServer
```

# 01_Install_DNS_Server_Role_And_Management_Tools_Management_Workstation_Tools_Skeleton
```powershell
# Run on a Windows Server management host or admin workstation.
# Purpose: install DNS management tools without making this machine a DNS server.

# Windows Server management host path.
Get-WindowsFeature -Name RSAT-DNS-Server

Install-WindowsFeature `
  -Name RSAT-DNS-Server `
  -Verbose

Get-WindowsFeature -Name RSAT-DNS-Server

# Confirm DNS PowerShell module.
Import-Module DnsServer
Get-Command -Module DnsServer | Select-Object -First 20

# Open DNS Manager.
dnsmgmt.msc

# Test remote DNS management.
$DnsServer = "DC1"

Get-DnsServer -ComputerName $DnsServer
Get-DnsServerZone -ComputerName $DnsServer

# Windows 10 or Windows 11 admin workstation path.
# Use only on client OS, not Windows Server.
Get-WindowsCapability -Online |
  Where-Object Name -like "Rsat.Dns.Tools*"

Add-WindowsCapability `
  -Online `
  -Name "Rsat.Dns.Tools~~~~0.0.1.0"

Get-WindowsCapability -Online |
  Where-Object Name -like "Rsat.Dns.Tools*"
```

# 01_Install_DNS_Server_Role_And_Management_Tools_Post_Install_Baseline_Skeleton
```powershell
# Run after DNS Server role installation.
# Purpose: prove the DNS role, tools, service, firewall, and logs are clean before zone work.

$DnsServer = "DC1"
$EvidencePath = "C:\DNS-Install-Baseline"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Role and tool state.
Get-WindowsFeature -Name DNS,RSAT-DNS-Server |
  Out-File "$EvidencePath\windows-feature-dns.txt"

# Service state.
Get-Service DNS |
  Format-List * |
  Out-File "$EvidencePath\dns-service.txt"

# DNS Server object.
Get-DnsServer -ComputerName $DnsServer |
  Format-List * |
  Out-File "$EvidencePath\dns-server-object.txt"

# Existing zones.
Get-DnsServerZone -ComputerName $DnsServer |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\dns-zones.txt"

# DNS cmdlets available.
Get-Command -Module DnsServer |
  Select-Object Name, CommandType, Source |
  Out-File "$EvidencePath\dnsserver-module-commands.txt"

# Listener state.
Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\tcp-53-listener.txt"

Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\udp-53-listener.txt"

# Firewall state.
Get-NetFirewallRule -DisplayGroup "DNS Server" |
  Select-Object DisplayName, Enabled, Profile, Direction, Action |
  Out-File "$EvidencePath\dns-firewall-rules.txt"

# Local resolver test.
Resolve-DnsName localhost -Server 127.0.0.1 |
  Out-File "$EvidencePath\localhost-dns-test.txt"

# DNS Server event log.
Get-WinEvent -LogName "DNS Server" -MaxEvents 50 |
  Select-Object TimeCreated, Id, ProviderName, Message |
  Out-File "$EvidencePath\dns-server-events.txt"
```

# 01_Install_DNS_Server_Role_And_Management_Tools_Verification_Commands
```powershell
# Confirm role and tools.
Get-WindowsFeature -Name DNS
Get-WindowsFeature -Name RSAT-DNS-Server
Get-WindowsFeature -Name DNS,RSAT-DNS-Server

# Confirm DNS service.
Get-Service DNS
Start-Service DNS
Set-Service DNS -StartupType Automatic

# Confirm DNS PowerShell module.
Import-Module DnsServer
Get-Command -Module DnsServer

# Confirm DNS Server object and zones.
Get-DnsServer
Get-DnsServerZone

# Confirm DNS Manager can open.
dnsmgmt.msc

# Confirm local port listeners.
Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue
Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue

# Confirm firewall rule group.
Get-NetFirewallRule -DisplayGroup "DNS Server" |
  Select-Object DisplayName, Enabled, Profile, Direction, Action

# Local query tests.
Resolve-DnsName localhost -Server 127.0.0.1
Resolve-DnsName $env:COMPUTERNAME -Server 127.0.0.1

# Remote query test from another host.
Resolve-DnsName localhost -Server "DC1"

# DNS Server event log.
Get-WinEvent -LogName "DNS Server" -MaxEvents 50 |
  Select-Object TimeCreated, Id, ProviderName, Message

# System event log for service startup issues.
Get-WinEvent -LogName "System" -MaxEvents 50 |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*DNS*"
  }
```

# 01_Install_DNS_Server_Role_And_Management_Tools_Rollback
| Step | Action | PowerShell | Expected Result |
|---:|---|---|---|
| 1 | Capture current DNS role state | `Get-WindowsFeature -Name DNS,RSAT-DNS-Server` | Installed state is documented |
| 2 | Export DNS zones before uninstall if zones exist | `Get-DnsServerZone \| Export-Csv C:\DNS-Zones-Before-Uninstall.csv -NoTypeInformation` | Zone inventory is saved |
| 3 | Export DNS records if lab zones exist | `Get-DnsServerZone \| ForEach-Object { Get-DnsServerResourceRecord -ZoneName $_.ZoneName }` | Record data is captured |
| 4 | Stop DNS service | `Stop-Service DNS -Force` | DNS service stops |
| 5 | Uninstall DNS Server role | `Uninstall-WindowsFeature -Name DNS -Restart:$false` | DNS Server role is removed |
| 6 | Optionally remove DNS management tools | `Uninstall-WindowsFeature -Name RSAT-DNS-Server -Restart:$false` | DNS tools are removed |
| 7 | Confirm removal | `Get-WindowsFeature -Name DNS,RSAT-DNS-Server` | Feature state shows removed |
| 8 | Confirm service removal or stopped state | `Get-Service DNS -ErrorAction SilentlyContinue` | DNS service no longer runs |
| 9 | Confirm port 53 no longer listening | `Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue; Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue` | No DNS listener remains |
| 10 | Document rollback | `Record uninstall time, zones impacted, and whether restart is pending` | Rollback notes are complete |

# 01_Install_DNS_Server_Role_And_Management_Tools_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `Install-WindowsFeature` not recognized | Not on Windows Server or ServerManager module unavailable | `Get-Module -ListAvailable ServerManager` | Run on Windows Server or use correct RSAT/client tooling path |
| Install fails with access denied | PowerShell not elevated or account lacks rights | Check admin token and group membership | Run elevated as local admin or domain admin |
| DNS role installs but DNS console missing | Management tools were not installed | `Get-WindowsFeature RSAT-DNS-Server` | Run `Install-WindowsFeature DNS -IncludeManagementTools` or install `RSAT-DNS-Server` |
| DNS service not found | Role did not install | `Get-WindowsFeature DNS` | Re-run install and review install log |
| DNS service stopped | Service startup issue or manual stop | `Get-Service DNS`; DNS Server event log | Start service and review event errors |
| Port 53 not listening | DNS service not running or binding issue | `Get-NetTCPConnection -LocalPort 53`; `Get-NetUDPEndpoint -LocalPort 53` | Start DNS service and check DNS Server settings |
| Remote DNS query fails | Firewall, routing, or service issue | `Test-NetConnection <dns-server> -Port 53` | Enable DNS firewall rules and verify network path |
| Remote management fails | WinRM disabled or permissions issue | `Test-WSMan <server>` | Enable PS remoting or use Server Manager with proper rights |
| `Get-DnsServer` fails remotely | DNS tools missing on management host or remote access blocked | `Get-Command Get-DnsServer`; `Test-WSMan` | Install RSAT DNS tools and fix remoting |
| DNS Manager opens but cannot connect | Wrong server name, permissions, or firewall | Try `Get-DnsServer -ComputerName <server>` | Use correct server and admin rights |
| DNS feature source missing | Component store or source path problem | Review install error and `-Source` requirement | Provide valid Windows source path or repair component store |
| Server has dynamic IP | DNS server address may change | `Get-NetIPConfiguration` | Configure static IP before production DNS use |
| DNS client points to external resolver only | Server cannot resolve internal future zones | `Get-DnsClientServerAddress` | Point server DNS client to internal DNS design |
| Public firewall profile blocks DNS | Wrong network profile | `Get-NetConnectionProfile`; `Get-NetFirewallRule -DisplayGroup "DNS Server"` | Correct network profile or firewall rules |
| Event log shows startup errors | Bad config, port conflict, or service issue | `Get-WinEvent -LogName "DNS Server"` | Fix specific event ID cause before zone creation |

# 01_Install_DNS_Server_Role_And_Management_Tools_Related_Labs
| Lab                                                            | Relationship                                                        |
| -------------------------------------------------------------- | ------------------------------------------------------------------- |
| 02_Create_Forward_Lookup_Zone                                  | First authoritative namespace created after DNS role installation   |
| 03_Create_Reverse_Lookup_Zone_And_PTR_Records                  | Adds reverse DNS after DNS Server baseline exists                   |
| 04_Create_And_Manage_DNS_Resource_Records                      | Uses DNS management tools and DnsServer cmdlets installed here      |
| 05_Configure_DNS_Forwarders_And_Root_Hints                     | Builds recursive lookup behavior after server installation          |
| 06_Configure_DNS_Conditional_Forwarders                        | Adds namespace-specific forwarding after base DNS role is installed |
| 07_Configure_AD_Integrated_DNS_Zones                           | Extends DNS role into AD-integrated zone storage and replication    |
| 08_Configure_DNS_Dynamic_Updates                               | Requires DNS role and zones to already exist                        |
| 09_Troubleshoot_DNS_Client_Resolution                          | Uses installed DNS server as the authoritative or recursive target  |
| 18_Configure_DNS_Query_Logging_And_Diagnostics                 | Uses DNS Server logs and diagnostics after role installation        |
| 21_Troubleshoot_DNSSEC_Policies_Zone_Transfers_And_Replication | Depends on a working DNS Server role baseline                       |