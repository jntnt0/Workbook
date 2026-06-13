02_Authorize_DHCP_Server_In_Active_Directory.md
# Authorize_DHCP_Server_In_Active_Directory

# Authorize_DHCP_Server_In_Active_Directory_Index
02_Authorize_DHCP_Server_In_Active_Directory.md
Authorize_DHCP_Server_In_Active_Directory
Authorize_DHCP_Server_In_Active_Directory_Source_Basis
Authorize_DHCP_Server_In_Active_Directory_Mental_Model
Authorize_DHCP_Server_In_Active_Directory_Planning_Table
Authorize_DHCP_Server_In_Active_Directory_Configuration_Checklist
Authorize_DHCP_Server_In_Active_Directory_Precheck_Skeleton
Authorize_DHCP_Server_In_Active_Directory_Authorization_Skeleton
Authorize_DHCP_Server_In_Active_Directory_Server_Validation_Skeleton
Authorize_DHCP_Server_In_Active_Directory_DC_Validation_Skeleton
Authorize_DHCP_Server_In_Active_Directory_Event_Log_Skeleton
Authorize_DHCP_Server_In_Active_Directory_Verification_Commands
Authorize_DHCP_Server_In_Active_Directory_Rollback
Authorize_DHCP_Server_In_Active_Directory_Failure_Checks
Authorize_DHCP_Server_In_Active_Directory_Related_Labs

# Authorize_DHCP_Server_In_Active_Directory_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DhcpServerInDC | Authorizing a DHCP server in Active Directory |
| Microsoft Learn | Get-DhcpServerInDC | Listing DHCP servers authorized in Active Directory |
| Microsoft Learn | Remove-DhcpServerInDC | Removing DHCP server authorization from Active Directory |
| Microsoft Learn | Get-DhcpServerSetting | Reviewing DHCP server-level settings |
| Microsoft Learn | Get-Service | Confirming DHCP Server service state |
| Microsoft Learn | Restart-Service | Restarting DHCP Server service after authorization changes |
| Microsoft Learn | Resolve-DnsName | Confirming the DHCP server FQDN resolves correctly |
| Microsoft Learn | Get-ADComputer | Confirming the DHCP server computer account exists in AD |
| Microsoft Learn | Active Directory DHCP authorization | Preventing unauthorized DHCP servers from leasing addresses in an AD domain |
| Windows Server operational practice | DHCP authorization, DNS, domain membership, event logs | Validating that a domain DHCP server is approved to serve leases |

# Authorize_DHCP_Server_In_Active_Directory_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP authorization | AD-based approval that allows a Windows DHCP server to lease addresses in a domain environment |
| Authorized DHCP server list | List of DHCP servers stored in Active Directory that are allowed to provide DHCP leases |
| Rogue DHCP protection | Prevents unauthorized Windows DHCP servers from serving leases on an AD domain network |
| DHCP server FQDN | Fully qualified DNS name of the DHCP server, such as `DHCP1.corp.local` |
| DHCP server IP | Static IP address associated with the authorized DHCP server |
| Domain membership | DHCP server should normally be joined to the AD domain before authorization |
| Enterprise Admin or delegated rights | Required permission level to authorize DHCP in Active Directory |
| AD DS dependency | Authorization is written to AD and requires domain controller connectivity |
| DNS dependency | The DHCP server name should resolve correctly before authorization |
| DHCPServer service | Service that checks authorization state before serving clients in a domain |
| Local security groups | DHCP Administrators and DHCP Users local groups created during post-install setup |
| Authorized but not configured | Authorization only approves the server; scopes and options still need to be created separately |
| First rule | Do not troubleshoot client leases until the DHCP role is installed, service is running, server is authorized, and a scope exists |

# Authorize_DHCP_Server_In_Active_Directory_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server hostname | `DHCP1` | `<dhcp-server-name>` |
| DHCP server FQDN | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.20` | `<dhcp-server-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Domain controller to validate against | `DC1.corp.local` | `<dc-fqdn>` |
| Authorization account | `CORP\Administrator` or delegated DHCP authorization account | `<authorization-account>` |
| Required permission | Enterprise Admin or delegated equivalent | `<permission-model>` |
| DHCP server role installed | Yes | `<yes-no>` |
| DHCP local security groups created | Yes | `<yes-no>` |
| DHCP service restart allowed | Yes | `<yes-no>` |
| DNS record expected | `DHCP1.corp.local -> 10.10.10.20` | `<expected-dns-record>` |
| AD computer object expected | `CN=DHCP1,CN=Computers,DC=corp,DC=local` | `<dhcp-computer-object-dn>` |
| Evidence path | `C:\DHCP-Authorization` | `<evidence-path>` |
| Rollback stance | Remove authorization only if wrong server/IP was authorized | `<rollback-plan>` |
| Next workbook | `03_Create_DHCPv4_Scope.md` | `<next-task>` |

# Authorize_DHCP_Server_In_Active_Directory_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server / Admin Host | `whoami /groups` | Account and group membership are visible |
| 2 | Confirm DHCP server hostname | DHCP Server | `hostname` | Expected DHCP server name appears |
| 3 | Confirm DHCP server domain membership | DHCP Server | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Server is joined to expected AD domain |
| 4 | Confirm static IP configuration | DHCP Server | `Get-NetIPConfiguration` | DHCP server has stable IP, gateway, and DNS |
| 5 | Confirm DHCP role installed | DHCP Server | `Get-WindowsFeature DHCP` | DHCP role shows installed |
| 6 | Confirm DHCP management tools installed | DHCP Server | `Get-WindowsFeature RSAT-DHCP` | DHCP tools show installed |
| 7 | Confirm DHCP service exists | DHCP Server | `Get-Service DHCPServer` | DHCP service is present |
| 8 | Confirm DHCP module loads | DHCP Server / Admin Host | `Import-Module DhcpServer` | DHCP module imports |
| 9 | Confirm DHCP cmdlets exist | DHCP Server / Admin Host | `Get-Command Add-DhcpServerInDC` | Authorization cmdlet is available |
| 10 | Confirm DHCP server FQDN resolves | DHCP Server / Admin Host | `Resolve-DnsName <dhcp-server-fqdn>` | FQDN resolves to DHCP server IP |
| 11 | Confirm DHCP server computer account exists | DC / Admin Host | `Get-ADComputer <dhcp-server-name>` | AD computer object exists |
| 12 | Check current authorized DHCP servers | DHCP Server / Admin Host | `Get-DhcpServerInDC` | Current authorized list is visible |
| 13 | Authorize DHCP server in AD | DHCP Server / Admin Host | `Add-DhcpServerInDC -DnsName "<dhcp-server-fqdn>" -IPAddress "<dhcp-server-ip>"` | DHCP server is added to authorized list |
| 14 | Confirm server appears in authorized list | DHCP Server / Admin Host | `Get-DhcpServerInDC` | DHCP server FQDN and IP appear |
| 15 | Restart DHCP service | DHCP Server | `Restart-Service DHCPServer` | DHCP service restarts cleanly |
| 16 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | Service state is Running |
| 17 | Confirm DHCP server setting query works | DHCP Server | `Get-DhcpServerSetting` | Server settings return without error |
| 18 | Review DHCP Server operational events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50` | No authorization-blocking errors remain |
| 19 | Review System log DHCP events | DHCP Server | `Get-WinEvent -LogName System -MaxEvents 100 \| Where-Object {$_.ProviderName -like "*DHCP*"}` | DHCP-related service events are visible |
| 20 | Document authorization | Operator | `Record DHCP server FQDN, IP, domain, account used, timestamp, and validation result` | Authorization record is complete |

# Authorize_DHCP_Server_In_Active_Directory_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server or an admin host with DHCP tools.
# Purpose: confirm the server is ready to be authorized in Active Directory.

$DhcpServerName = "DHCP1"
$DomainFqdn = "corp.local"
$DhcpServerFqdn = "$DhcpServerName.$DomainFqdn"
$DhcpServerIp = "10.10.10.20"
$EvidencePath = "C:\DHCP-Authorization"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Confirm local server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$EvidencePath\computer-domain-state.txt"

# Confirm network configuration.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$EvidencePath\dns-client-servers.txt"

# Confirm DHCP role and tools.
Get-WindowsFeature DHCP,RSAT-DHCP |
  Tee-Object "$EvidencePath\dhcp-feature-state.txt"

# Confirm DHCP service.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-before-authorization.txt"

# Confirm DHCP PowerShell module and authorization cmdlet.
Import-Module DhcpServer

Get-Command Add-DhcpServerInDC |
  Tee-Object "$EvidencePath\add-dhcpserverindc-command.txt"

Get-Command Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\get-dhcpserverindc-command.txt"

# Confirm DHCP server DNS record.
Resolve-DnsName $DhcpServerFqdn -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-dhcp-server-fqdn.txt"

# Confirm current authorized DHCP server list.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers-before.txt"
```

# Authorize_DHCP_Server_In_Active_Directory_Authorization_Skeleton
```powershell
# Run in elevated PowerShell using an account with rights to authorize DHCP in AD.
# Commonly requires Enterprise Admin or delegated equivalent.
# Purpose: authorize the DHCP server in Active Directory.

$DhcpServerName = "DHCP1"
$DomainFqdn = "corp.local"
$DhcpServerFqdn = "$DhcpServerName.$DomainFqdn"
$DhcpServerIp = "10.10.10.20"
$EvidencePath = "C:\DHCP-Authorization"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-authorization-transcript.txt"

Import-Module DhcpServer

# Show authorized list before change.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers-before-change.txt"

# Authorize DHCP server in Active Directory.
Add-DhcpServerInDC `
  -DnsName $DhcpServerFqdn `
  -IPAddress $DhcpServerIp

# Show authorized list after change.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers-after-change.txt"

# Restart DHCP Server service so authorization state is refreshed.
Restart-Service DHCPServer

# Confirm DHCP service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-after-authorization.txt"

Stop-Transcript
```

# Authorize_DHCP_Server_In_Active_Directory_Server_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: confirm the local DHCP server sees clean service and authorization state.

$DhcpServerName = "DHCP1"
$DomainFqdn = "corp.local"
$DhcpServerFqdn = "$DhcpServerName.$DomainFqdn"
$DhcpServerIp = "10.10.10.20"
$EvidencePath = "C:\DHCP-Authorization"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm DHCP service.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-validation.txt"

# Confirm DHCP server settings are readable.
Get-DhcpServerSetting |
  Tee-Object "$EvidencePath\dhcp-server-settings.txt"

# Confirm this server appears in the AD authorized list.
Get-DhcpServerInDC |
  Where-Object {
    $_.DnsName -eq $DhcpServerFqdn -or
    $_.IPAddress -eq $DhcpServerIp
  } |
  Tee-Object "$EvidencePath\this-server-authorized-entry.txt"

# Confirm current feature state.
Get-WindowsFeature DHCP,RSAT-DHCP |
  Tee-Object "$EvidencePath\dhcp-feature-validation.txt"

# Confirm DHCP console command exists if GUI tools are installed.
Get-Command dhcpmgmt.msc -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-console-command.txt"
```

# Authorize_DHCP_Server_In_Active_Directory_DC_Validation_Skeleton
```powershell
# Run on a domain controller or admin host with RSAT Active Directory tools.
# Purpose: validate the DHCP server object and authorization from the AD side.

$DhcpServerName = "DHCP1"
$DomainFqdn = "corp.local"
$DhcpServerFqdn = "$DhcpServerName.$DomainFqdn"
$DhcpServerIp = "10.10.10.20"
$EvidencePath = "C:\DHCP-Authorization-AD-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module ActiveDirectory
Import-Module DhcpServer

# Confirm domain identity.
Get-ADDomain |
  Tee-Object "$EvidencePath\ad-domain.txt"

Get-ADForest |
  Tee-Object "$EvidencePath\ad-forest.txt"

# Confirm DHCP server computer account exists.
Get-ADComputer -Identity $DhcpServerName -Properties DNSHostName,IPv4Address,Enabled,DistinguishedName |
  Select-Object Name,DNSHostName,IPv4Address,Enabled,DistinguishedName |
  Tee-Object "$EvidencePath\dhcp-server-ad-computer.txt"

# Confirm DHCP server DNS resolution.
Resolve-DnsName $DhcpServerFqdn -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-dhcp-server-fqdn.txt"

# Confirm DHCP authorization list.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Confirm target DHCP server is in the authorized list.
Get-DhcpServerInDC |
  Where-Object {
    $_.DnsName -eq $DhcpServerFqdn -or
    $_.IPAddress -eq $DhcpServerIp
  } |
  Tee-Object "$EvidencePath\target-dhcp-server-authorized.txt"
```

# Authorize_DHCP_Server_In_Active_Directory_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: collect DHCP authorization and service evidence from event logs.

$EvidencePath = "C:\DHCP-Authorization"

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

# Service state after event review.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"
```

# Authorize_DHCP_Server_In_Active_Directory_Verification_Commands
```powershell
# Confirm DHCP role and tools.
Get-WindowsFeature DHCP,RSAT-DHCP

# Confirm DHCP service.
Get-Service DHCPServer

# Confirm DHCP module.
Import-Module DhcpServer
Get-Command Add-DhcpServerInDC
Get-Command Get-DhcpServerInDC
Get-Command Remove-DhcpServerInDC

# Confirm DHCP server identity.
hostname
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
Get-NetIPConfiguration

# Confirm DHCP server DNS resolution.
Resolve-DnsName "<dhcp-server-fqdn>"

# Confirm DHCP server AD computer object from DC/admin host.
Get-ADComputer "<dhcp-server-name>" -Properties DNSHostName,IPv4Address,Enabled,DistinguishedName

# Authorize DHCP server.
Add-DhcpServerInDC -DnsName "<dhcp-server-fqdn>" -IPAddress "<dhcp-server-ip>"

# List authorized DHCP servers.
Get-DhcpServerInDC

# Confirm specific server appears.
Get-DhcpServerInDC | Where-Object {$_.DnsName -eq "<dhcp-server-fqdn>" -or $_.IPAddress -eq "<dhcp-server-ip>"}

# Restart DHCP service after authorization.
Restart-Service DHCPServer
Get-Service DHCPServer

# Confirm DHCP server settings.
Get-DhcpServerSetting

# Review DHCP events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DHCP*" -or $_.Message -like "*DHCP*"}
```

# Authorize_DHCP_Server_In_Active_Directory_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Wrong DHCP server authorized | `Remove-DhcpServerInDC -DnsName "<wrong-dhcp-server-fqdn>" -IPAddress "<wrong-dhcp-server-ip>"` | Removes authorization for that server |
| Correct server authorized with wrong IP | Remove wrong entry, then re-add correct FQDN/IP | DHCP server may remain unauthorized until corrected |
| DHCP service restarted | No rollback normally needed | Brief DHCP service interruption |
| Evidence folder created | `Remove-Item C:\DHCP-Authorization -Recurse -Force` | Deletes validation evidence |
| DNS issue discovered | Correct DNS A record or client resolver settings | Wrong DNS can break AD and DHCP management |
| Server authorized before scope creation | No rollback needed if server is correct | Server still needs scopes before leasing |
| Unauthorized server still leasing | Stop service or remove role on rogue server | Client outages possible if clients depend on rogue leases |

# Authorize_DHCP_Server_In_Active_Directory_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| `Add-DhcpServerInDC` is not recognized | DHCP tools/module missing | `Get-WindowsFeature RSAT-DHCP`; `Import-Module DhcpServer` | Reinstalling Windows |
| Access denied during authorization | Insufficient AD permissions | Confirm Enterprise Admin or delegated rights | Reinstalling DHCP role |
| Server does not appear in `Get-DhcpServerInDC` | Authorization failed or AD replication delay | Re-run `Get-DhcpServerInDC` from DC/admin host | Creating scopes anyway |
| DHCP service says server is unauthorized | AD authorization missing or not refreshed | `Get-DhcpServerInDC`; restart DHCP service | Troubleshooting client NICs |
| DHCP server FQDN does not resolve | DNS issue | `Resolve-DnsName <dhcp-server-fqdn>` | Removing DHCP role |
| DHCP server IP is wrong in authorized list | Wrong authorization entry | Remove wrong entry and add correct FQDN/IP | Editing scope options |
| DHCP server is not domain joined | Host preparation issue | `Get-ComputerInfo \| Select CsDomain,CsPartOfDomain` | Authorizing random hostname |
| DHCP role is installed but no leases are issued | No scope, no authorization, or service stopped | Check role, service, authorization, scopes | Rebuilding AD |
| `Get-DhcpServerInDC` fails from member server | Module, DNS, DC connectivity, permissions | Test domain DNS and run from DC/admin host | Assuming AD is broken |
| DHCP console shows warning icon | Server not authorized or service issue | Check authorization and DHCP service | Creating additional scopes |
| Authorization succeeds but clients still fail | Scope not created or wrong network path | Proceed to scope and client validation workbooks | Reauthorizing repeatedly |
| Event log shows authorization errors | AD authorization or domain communication issue | Review DHCP operational log and `Get-DhcpServerInDC` | Changing lease duration |
| Multiple DHCP servers listed unexpectedly | Possible stale or rogue DHCP authorization | Identify each FQDN/IP before removing | Removing all entries blindly |
| DHCP server cannot contact domain | DNS, network, firewall, or DC issue | `Resolve-DnsName <domain-fqdn>`; `nltest /dsgetdc:<domain-fqdn>` | Editing DHCP options |

# Authorize_DHCP_Server_In_Active_Directory_Related_Labs
| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Install DHCP Server role | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | DHCP role and tools installation |
| Authorize DHCP server in AD | `02_Authorize_DHCP_Server_In_Active_Directory.md` | AD-integrated DHCP authorization |
| Create DHCPv4 scope | `03_Create_DHCPv4_Scope.md` | Lease pool creation after authorization |
| Configure DHCPv4 scope options | `04_Configure_DHCPv4_Scope_Options.md` | DHCP client configuration delivery |
| Validate DHCP client lease | `06_Validate_DHCP_Client_Lease_Assignment.md` | End-to-end DHCP lease proof |
| Troubleshoot DHCP client lease failures | `12_Troubleshoot_DHCP_Client_Lease_Failures.md` | Authorization and scope troubleshooting |