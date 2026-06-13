01_Install_DHCP_Server_Role_And_Management_Tools.md
# Install_DHCP_Server_Role_And_Management_Tools

# Install_DHCP_Server_Role_And_Management_Tools_Index
01_Install_DHCP_Server_Role_And_Management_Tools.md
Install_DHCP_Server_Role_And_Management_Tools
Install_DHCP_Server_Role_And_Management_Tools_Source_Basis
Install_DHCP_Server_Role_And_Management_Tools_Mental_Model
Install_DHCP_Server_Role_And_Management_Tools_Planning_Table
Install_DHCP_Server_Role_And_Management_Tools_Configuration_Checklist
Install_DHCP_Server_Role_And_Management_Tools_Precheck_Skeleton
Install_DHCP_Server_Role_And_Management_Tools_Install_Skeleton
Install_DHCP_Server_Role_And_Management_Tools_Post_Install_Skeleton
Install_DHCP_Server_Role_And_Management_Tools_Remote_Management_Skeleton
Install_DHCP_Server_Role_And_Management_Tools_Verification_Commands
Install_DHCP_Server_Role_And_Management_Tools_Rollback
Install_DHCP_Server_Role_And_Management_Tools_Failure_Checks
Install_DHCP_Server_Role_And_Management_Tools_Related_Labs

# Install_DHCP_Server_Role_And_Management_Tools_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install-WindowsFeature | Installing the DHCP Server role and optional management tools |
| Microsoft Learn | Get-WindowsFeature | Confirming whether DHCP Server and RSAT tools are installed |
| Microsoft Learn | Add-DhcpServerSecurityGroup | Creating DHCP Administrators and DHCP Users local security groups |
| Microsoft Learn | Restart-Service | Restarting the DHCP Server service after post-install security group creation |
| Microsoft Learn | Get-Service | Confirming DHCP Server service state |
| Microsoft Learn | Get-Command | Confirming the DhcpServer PowerShell module is available |
| Microsoft Learn | DHCP Server PowerShell module | Managing DHCP server configuration with PowerShell |
| Microsoft Learn | Server Manager / DHCP console | Managing DHCP with graphical tools |
| Windows Server operational practice | Static IP, hostname, domain membership, firewall, event logs | Preparing a Windows Server host before DHCP deployment |

# Install_DHCP_Server_Role_And_Management_Tools_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP Server role | Windows Server role that leases IP configuration to DHCP clients |
| Management tools | DHCP console and PowerShell module used to administer scopes, leases, options, and failover |
| RSAT DHCP tools | Remote Server Administration Tools used to manage DHCP locally or from an admin workstation |
| DhcpServer PowerShell module | PowerShell command set for DHCP installation validation and configuration |
| DHCPServer service | Windows service that performs DHCP server operations |
| DHCP Administrators group | Local group that can administer the DHCP Server service |
| DHCP Users group | Local group that can view DHCP server information |
| Static IP requirement | DHCP server should use a stable IP address before it begins leasing client addresses |
| Domain member server | Common DHCP deployment target in AD environments |
| Authorization dependency | In an AD domain, DHCP should be authorized before it leases to domain clients |
| First rule | Do not create scopes before confirming hostname, static IP, role install, management tools, service state, and post-install security groups |

# Install_DHCP_Server_Role_And_Management_Tools_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server hostname | `DHCP1` | `<dhcp-server-name>` |
| DHCP server FQDN | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.20` | `<dhcp-server-ip>` |
| Interface alias | `Ethernet` | `<interface-alias>` |
| Prefix length | `24` | `<prefix-length>` |
| Default gateway | `10.10.10.1` | `<gateway-ip>` |
| Preferred DNS server | `10.10.10.10` | `<dns-server-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain membership required | Yes | `<yes-no>` |
| Install method | PowerShell | `<powershell-gui>` |
| Include management tools | Yes | `<yes-no>` |
| Remote management host | `ADMIN1` | `<admin-workstation>` |
| Install evidence path | `C:\DHCP-Install` | `<evidence-path>` |
| Next workbook | `02_Authorize_DHCP_Server_In_Active_Directory.md` | `<next-task>` |
| Rollback stance | Remove role if install is wrong | `<rollback-plan>` |

# Install_DHCP_Server_Role_And_Management_Tools_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Local Administrators group is visible |
| 2 | Confirm hostname | DHCP Server | `hostname` | Server has intended name |
| 3 | Confirm server identity | DHCP Server | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Server name and domain state are known |
| 4 | Confirm IP configuration | DHCP Server | `Get-NetIPConfiguration` | Static IP, gateway, and DNS are visible |
| 5 | Confirm interface alias | DHCP Server | `Get-NetAdapter` | Correct network adapter is known |
| 6 | Configure static IP if needed | DHCP Server | `New-NetIPAddress -InterfaceAlias "<interface-alias>" -IPAddress "<dhcp-server-ip>" -PrefixLength <prefix-length> -DefaultGateway "<gateway-ip>"` | Server has stable IP |
| 7 | Configure DNS client if needed | DHCP Server | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ServerAddresses "<dns-server-ip>"` | Server uses intended DNS resolver |
| 8 | Confirm network path to domain/DNS | DHCP Server | `Resolve-DnsName <domain-fqdn>` | Domain DNS resolves |
| 9 | Confirm current DHCP role state | DHCP Server | `Get-WindowsFeature DHCP` | Install state is visible |
| 10 | Confirm DHCP tools state | DHCP Server | `Get-WindowsFeature RSAT-DHCP` | Management tools state is visible |
| 11 | Install DHCP Server role and tools | DHCP Server | `Install-WindowsFeature -Name DHCP -IncludeManagementTools` | DHCP role and tools are installed |
| 12 | Confirm role install result | DHCP Server | `Get-WindowsFeature DHCP,RSAT-DHCP` | Both show installed as expected |
| 13 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | Module imports without error |
| 14 | Confirm DHCP cmdlets | DHCP Server | `Get-Command -Module DhcpServer` | DHCP cmdlets are listed |
| 15 | Create DHCP local security groups | DHCP Server | `Add-DhcpServerSecurityGroup` | DHCP Administrators and DHCP Users groups are created |
| 16 | Restart DHCP service | DHCP Server | `Restart-Service DHCPServer` | DHCP service restarts |
| 17 | Confirm DHCP service state | DHCP Server | `Get-Service DHCPServer` | Service exists and is running |
| 18 | Confirm DHCP console availability | DHCP Server | `dhcpmgmt.msc` | DHCP console opens |
| 19 | Review DHCP Server events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50` | DHCP operational events are visible |
| 20 | Export install evidence | DHCP Server | Run evidence export skeleton | Install evidence files are saved |
| 21 | Document install state | Operator | `Record server name, IP, role state, tool state, service state, next authorization task` | Build record is complete |

# Install_DHCP_Server_Role_And_Management_Tools_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the planned DHCP server.
# Purpose: confirm the host is ready before installing DHCP Server.

$DhcpServerName = "DHCP1"
$DomainFqdn = "corp.local"
$InterfaceAlias = "Ethernet"
$DhcpServerIp = "10.10.10.20"
$PrefixLength = 24
$Gateway = "10.10.10.1"
$DnsServer = "10.10.10.10"
$EvidencePath = "C:\DHCP-Install"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Confirm hostname and domain state.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$EvidencePath\computer-domain-state.txt"

# Confirm network adapters.
Get-NetAdapter |
  Tee-Object "$EvidencePath\net-adapters.txt"

# Confirm current IP configuration.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-before.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dns-client-before.txt"

# Optional static IP configuration.
# Use only if the server does not already have the intended static IP.

# New-NetIPAddress `
#   -InterfaceAlias $InterfaceAlias `
#   -IPAddress $DhcpServerIp `
#   -PrefixLength $PrefixLength `
#   -DefaultGateway $Gateway

# Set-DnsClientServerAddress `
#   -InterfaceAlias $InterfaceAlias `
#   -ServerAddresses $DnsServer

# Confirm final resolver state.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-after.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dns-client-after.txt"

# Confirm DNS resolution for the domain if domain joined or planned for AD-integrated DHCP.
Resolve-DnsName $DomainFqdn -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-domain.txt"

# Confirm current Windows feature state.
Get-WindowsFeature DHCP,RSAT-DHCP |
  Tee-Object "$EvidencePath\windows-feature-state-before.txt"
```

# Install_DHCP_Server_Role_And_Management_Tools_Install_Skeleton
```powershell
# Run in elevated PowerShell on the planned DHCP server.
# Purpose: install the DHCP Server role and management tools.

$EvidencePath = "C:\DHCP-Install"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-role-install-transcript.txt"

# Install DHCP Server role and DHCP management tools.
Install-WindowsFeature `
  -Name DHCP `
  -IncludeManagementTools

# Confirm installed feature state.
Get-WindowsFeature DHCP,RSAT-DHCP |
  Tee-Object "$EvidencePath\windows-feature-state-after.txt"

# Confirm DHCP PowerShell module availability.
Import-Module DhcpServer

Get-Command -Module DhcpServer |
  Tee-Object "$EvidencePath\dhcpserver-module-commands.txt"

# Confirm DHCP service exists.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-after-install.txt"

Stop-Transcript
```

# Install_DHCP_Server_Role_And_Management_Tools_Post_Install_Skeleton
```powershell
# Run in elevated PowerShell after the DHCP role install completes.
# Purpose: complete local DHCP post-install setup and verify service state.

$EvidencePath = "C:\DHCP-Install"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Import DHCP module.
Import-Module DhcpServer

# Create DHCP local security groups.
# This creates DHCP Administrators and DHCP Users local groups if needed.
Add-DhcpServerSecurityGroup

# Restart DHCP service so security group changes are recognized.
Restart-Service DHCPServer

# Confirm DHCP service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-post-install.txt"

# Confirm local DHCP groups exist.
Get-LocalGroup |
  Where-Object {$_.Name -like "DHCP*"} |
  Tee-Object "$EvidencePath\local-dhcp-groups.txt"

# Confirm DHCP module still loads.
Get-Command -Module DhcpServer |
  Select-Object Name,CommandType,Source |
  Tee-Object "$EvidencePath\dhcp-module-command-summary.txt"

# Confirm DHCP console can be launched manually if GUI is available.
# dhcpmgmt.msc

# Review recent DHCP Server operational events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50 |
  Tee-Object "$EvidencePath\dhcp-operational-events-post-install.txt"
```

# Install_DHCP_Server_Role_And_Management_Tools_Remote_Management_Skeleton
```powershell
# Run from an admin workstation or management server.
# Purpose: confirm DHCP can be inspected remotely after tools are installed.
# Replace DHCP1 with the target DHCP server.

$DhcpServer = "DHCP1"
$EvidencePath = "C:\DHCP-Remote-Management-Check"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm name resolution to DHCP server.
Resolve-DnsName $DhcpServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-dhcp-server.txt"

# Confirm basic connectivity.
Test-NetConnection $DhcpServer |
  Tee-Object "$EvidencePath\test-dhcp-server-connectivity.txt"

# Confirm DHCP Server service can be queried remotely.
Get-Service -ComputerName $DhcpServer -Name DHCPServer |
  Tee-Object "$EvidencePath\remote-dhcp-service.txt"

# Confirm DHCP module is available on management host.
Get-Command -Module DhcpServer |
  Select-Object Name,CommandType,Source |
  Tee-Object "$EvidencePath\local-dhcp-cmdlets.txt"

# Confirm DHCP server object can be queried.
# This may fail until authorization and scopes are configured in later workbooks.
Get-DhcpServerSetting -ComputerName $DhcpServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\remote-dhcp-server-settings.txt"
```

# Install_DHCP_Server_Role_And_Management_Tools_Verification_Commands
```powershell
# Confirm server identity.
hostname
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm IP configuration.
Get-NetIPConfiguration
Get-DnsClientServerAddress -AddressFamily IPv4

# Confirm DHCP role and tools.
Get-WindowsFeature DHCP
Get-WindowsFeature RSAT-DHCP
Get-WindowsFeature DHCP,RSAT-DHCP

# Confirm DHCP PowerShell module.
Import-Module DhcpServer
Get-Command -Module DhcpServer

# Confirm DHCP service.
Get-Service DHCPServer

# Confirm DHCP local groups.
Get-LocalGroup | Where-Object {$_.Name -like "DHCP*"}

# Confirm DHCP console launch path if GUI is available.
dhcpmgmt.msc

# Confirm DHCP events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50

# Confirm server is not necessarily authorized yet.
# Authorization is handled in the next workbook.
Get-DhcpServerInDC
```

# Install_DHCP_Server_Role_And_Management_Tools_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Static IP configured incorrectly | `Remove-NetIPAddress -InterfaceAlias "<interface-alias>" -IPAddress "<dhcp-server-ip>"` then reapply correct config | Can disconnect server if done remotely |
| DNS client server changed incorrectly | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ServerAddresses "<correct-dns-ip>"` | Wrong DNS breaks AD and name resolution |
| DHCP Server role installed on wrong host | `Remove-WindowsFeature DHCP` | Removes DHCP role from server |
| DHCP management tools installed but not needed | `Remove-WindowsFeature RSAT-DHCP` | Removes DHCP console and tools |
| DHCP service restarted | No rollback normally needed | Brief service interruption if already in production |
| DHCP security groups created | Usually leave in place | Removing groups manually can break DHCP admin delegation |
| Install evidence folder created | `Remove-Item C:\DHCP-Install -Recurse -Force` | Deletes install evidence |

# Install_DHCP_Server_Role_And_Management_Tools_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| `Install-WindowsFeature` not recognized | PowerShell session or OS mismatch | Confirm Windows Server and elevated PowerShell | Reinstalling Windows |
| DHCP role install fails | Feature store, permissions, pending reboot, OS issue | `Get-WindowsFeature DHCP`; check install error | Creating scopes manually |
| DHCP service missing after install | Role did not install correctly | `Get-WindowsFeature DHCP`; reinstall role | Editing firewall rules |
| DHCP service stopped | Service state or post-install issue | `Get-Service DHCPServer` | Rebuilding the server |
| DHCP cmdlets unavailable | RSAT/tools/module missing | `Get-WindowsFeature RSAT-DHCP`; `Import-Module DhcpServer` | Reinstalling DHCP role first |
| DHCP console will not open | Management tools missing or GUI limitation | `Get-WindowsFeature RSAT-DHCP`; try PowerShell module | Assuming DHCP role failed |
| `Add-DhcpServerSecurityGroup` fails | Permission or local security group issue | Confirm elevated shell and local admin rights | Deleting DHCP service |
| Server has DHCP role but no leases | No scope or not authorized yet | Confirm this is only workbook 01 | Troubleshooting clients too early |
| `Get-DhcpServerInDC` does not show server | DHCP authorization not done yet | Proceed to workbook 02 | Reinstalling DHCP role |
| Domain resolution fails | DNS client or domain connectivity issue | `Get-DnsClientServerAddress`; `Resolve-DnsName <domain-fqdn>` | Installing DHCP anyway in AD workflow |
| Server has dynamic IP | Host preparation issue | `Get-NetIPConfiguration` | Creating DHCP scopes |
| Remote management fails | Firewall, WinRM, DNS, permissions, tools | Test name resolution, service query, RSAT tools | Reinstalling DHCP role |
| Event log shows DHCP warnings | Role installed but not fully configured or not authorized | Review DHCP operational events | Ignoring authorization step |

# Install_DHCP_Server_Role_And_Management_Tools_Related_Labs
| Lab                         | Related Workbook                                      | Skill Proven                         |
| --------------------------- | ----------------------------------------------------- | ------------------------------------ |
| Install DHCP Server role    | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | Windows Server role deployment       |
| Authorize DHCP server in AD | `02_Authorize_DHCP_Server_In_Active_Directory.md`     | AD-integrated DHCP safety control    |
| Create DHCPv4 scope         | `03_Create_DHCPv4_Scope.md`                           | IPv4 lease pool creation             |
| Configure DHCPv4 options    | `04_Configure_DHCPv4_Scope_Options.md`                | DHCP client network configuration    |
| Validate client DHCP lease  | `06_Validate_DHCP_Client_Lease_Assignment.md`         | Client and server lease verification |