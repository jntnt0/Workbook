01_Install_File_Server_Role_And_Management_Tools.md
# Install_File_Server_Role_And_Management_Tools

# Install_File_Server_Role_And_Management_Tools_Index
01_Install_File_Server_Role_And_Management_Tools.md
Install_File_Server_Role_And_Management_Tools
Install_File_Server_Role_And_Management_Tools_Source_Basis
Install_File_Server_Role_And_Management_Tools_Mental_Model
Install_File_Server_Role_And_Management_Tools_Planning_Table
Install_File_Server_Role_And_Management_Tools_Configuration_Checklist
Install_File_Server_Role_And_Management_Tools_Precheck_Skeleton
Install_File_Server_Role_And_Management_Tools_Core_Role_Install_Skeleton
Install_File_Server_Role_And_Management_Tools_Management_Tools_Skeleton
Install_File_Server_Role_And_Management_Tools_Remote_Management_Skeleton
Install_File_Server_Role_And_Management_Tools_Post_Install_Validation_Skeleton
Install_File_Server_Role_And_Management_Tools_Verification_Commands
Install_File_Server_Role_And_Management_Tools_Rollback
Install_File_Server_Role_And_Management_Tools_Failure_Checks
Install_File_Server_Role_And_Management_Tools_Related_Labs

# Install_File_Server_Role_And_Management_Tools_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install-WindowsFeature | Installing Windows Server roles and management tools |
| Microsoft Learn | Get-WindowsFeature | Confirming File Server role and related feature state |
| Microsoft Learn | File and Storage Services | Core Windows Server file server role family |
| Microsoft Learn | SMBShare PowerShell module | Validating SMB share cmdlet availability after role installation |
| Microsoft Learn | Server Manager | GUI-based File and Storage Services management |
| Microsoft Learn | Windows Admin Center | Browser-based server and file share management |
| Microsoft Learn | PowerShell remoting | Remote administration of Windows Server roles |
| Windows Server operational practice | File server baseline | Installing only the base role before storage, ACL, SMB, DFS, FSRM, and advanced features |

# Install_File_Server_Role_And_Management_Tools_Mental_Model
| Concept | Operational Meaning |
|---|---|
| File Server role | Windows Server role that allows the server to host and manage shared files |
| File and Storage Services | Parent role group that contains File Server and related file service components |
| FS-FileServer | Windows feature name for the core File Server role service |
| Management tools | Server Manager consoles, PowerShell modules, and RSAT tools used to manage file services |
| SMBShare module | PowerShell module used to create, inspect, secure, and remove SMB shares |
| Server Manager | GUI tool used to install roles and manage File and Storage Services |
| Windows Admin Center | Optional browser-based management tool for servers and shares |
| Remote management | WinRM and firewall posture that allows an admin workstation to manage the file server |
| Role separation | Install only the base File Server role first, then add DFS, FSRM, NFS, deduplication, iSCSI, Work Folders, or clustering later |
| Validation evidence | Text or CSV output proving the server state before and after role installation |
| First rule | Do not build shares until the server identity, IP configuration, domain membership, storage, and role state are proven clean |

# Install_File_Server_Role_And_Management_Tools_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| File server IP | `10.10.10.30` | `<file-server-ip>` |
| DNS server | `10.10.10.10` | `<dns-server-ip>` |
| Management host | `ADMIN1` or file server console | `<management-host>` |
| Admin account | `CORP\Administrator` or delegated admin | `<admin-account>` |
| Installation method | PowerShell / Server Manager | `<install-method>` |
| Core feature | `FS-FileServer` | `FS-FileServer` |
| Management feature | `RSAT-File-Services` | `<management-feature>` |
| Optional GUI management | Server Manager / Windows Admin Center | `<gui-management-tool>` |
| PowerShell remoting stance | Enabled for admin network | `<remoting-plan>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Reboot required | Usually no for base File Server role | `<reboot-plan>` |
| Rollback stance | Remove role only before shares are built | `<rollback-plan>` |
| Next workbook | `02_Prepare_File_Server_Storage_Volumes_And_Folders.md` | `<next-task>` |

# Install_File_Server_Role_And_Management_Tools_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm server hostname | File Server | `hostname` | Server name matches plan |
| 3 | Confirm domain membership | File Server | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Server is joined to expected domain if domain file server |
| 4 | Confirm IP configuration | File Server | `Get-NetIPConfiguration` | Static IP, gateway, and DNS are visible |
| 5 | Confirm DNS resolution for domain | File Server | `Resolve-DnsName "<domain-fqdn>"` | Domain name resolves |
| 6 | Confirm available storage before role install | File Server | `Get-Disk`; `Get-Volume` | Disk and volume layout is visible |
| 7 | Confirm current File Server feature state | File Server | `Get-WindowsFeature FS-FileServer` | Feature state is known before change |
| 8 | Confirm related file service feature state | File Server | `Get-WindowsFeature FS-*` | Related file service roles are visible |
| 9 | Confirm ServerManager module is available | File Server | `Get-Module ServerManager -ListAvailable` | ServerManager module exists |
| 10 | Install core File Server role | File Server | `Install-WindowsFeature FS-FileServer -IncludeManagementTools` | File Server role installs successfully |
| 11 | Install File Services management tools if needed | File Server | `Install-WindowsFeature RSAT-File-Services` | File services management tools install |
| 12 | Confirm File Server role installed | File Server | `Get-WindowsFeature FS-FileServer` | Install State shows Installed |
| 13 | Confirm management tools installed | File Server | `Get-WindowsFeature RSAT-File-Services` | Management tools show Installed or Available as expected |
| 14 | Confirm SMBShare cmdlets exist | File Server | `Get-Command -Module SmbShare` | SMB management cmdlets are available |
| 15 | Confirm existing SMB shares | File Server | `Get-SmbShare` | Default administrative shares are visible |
| 16 | Confirm SMB server service | File Server | `Get-Service LanmanServer` | Server service is Running |
| 17 | Confirm workstation service | File Server | `Get-Service LanmanWorkstation` | Workstation service is Running |
| 18 | Confirm SMB server configuration | File Server | `Get-SmbServerConfiguration` | SMB server settings are visible |
| 19 | Confirm firewall rules for File and Printer Sharing | File Server | `Get-NetFirewallRule -DisplayGroup "File and Printer Sharing"` | Firewall rule state is visible |
| 20 | Confirm WinRM remote management state | File Server | `Get-Service WinRM`; `Test-WSMan localhost` | Remote management state is known |
| 21 | Enable PowerShell remoting if required | File Server | `Enable-PSRemoting -Force` | WinRM remoting is enabled |
| 22 | Test remote management from admin host | Admin Host | `Test-WSMan "<file-server-name>"` | Admin host can reach WinRM on file server |
| 23 | Confirm Server Manager can see File and Storage Services | Management Host | `Open Server Manager > File and Storage Services` | File server appears manageable |
| 24 | Export installed feature state | File Server | `Get-WindowsFeature \| Export-Csv "<evidence-path>\windows-features-after.csv" -NoTypeInformation` | Feature inventory is saved |
| 25 | Document role installation result | Operator | `Record installed features, server name, IP, domain, remoting state, and evidence path` | Install record is complete |

# Install_File_Server_Role_And_Management_Tools_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the future file server.
# Purpose: capture baseline state before installing File Server role.

$EvidencePath = "C:\FileServices-Validation"
$DomainFqdn = "corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-file-server-role-precheck-transcript.txt"

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname-before.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName,WindowsVersion,OsHardwareAbstractionLayer |
  Tee-Object "$EvidencePath\computer-state-before.txt"

# Confirm network configuration.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration-before.txt"

Get-DnsClientServerAddress |
  Tee-Object "$EvidencePath\dns-client-server-addresses-before.txt"

# Confirm domain DNS if domain joined.
Resolve-DnsName $DomainFqdn -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\domain-resolution-before.txt"

# Confirm storage layout.
Get-Disk |
  Tee-Object "$EvidencePath\disks-before.txt"

Get-Volume |
  Tee-Object "$EvidencePath\volumes-before.txt"

# Capture current file service feature state.
Get-WindowsFeature FS-* |
  Sort-Object Name |
  Tee-Object "$EvidencePath\file-services-features-before.txt"

Get-WindowsFeature |
  Export-Csv "$EvidencePath\windows-features-before.csv" -NoTypeInformation

# Capture current SMB state.
Get-Service LanmanServer,LanmanWorkstation -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\smb-services-before.txt"

Get-SmbShare |
  Tee-Object "$EvidencePath\smb-shares-before.txt"

Stop-Transcript
```

Install_File_Server_Role_And_Management_Tools_Core_Role_Install_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: install the core File Server role.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-file-server-role-install-transcript.txt"

# Install the core File Server role.
Install-WindowsFeature `
  -Name FS-FileServer `
  -IncludeManagementTools

# Confirm role state.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-after.txt"

# Confirm whether restart is required.
Get-WindowsFeature FS-FileServer |
  Select-Object Name,DisplayName,InstallState,FeatureType,Path,Depth,DependsOn,Parent |
  Tee-Object "$EvidencePath\fs-fileserver-feature-detail-after.txt"

Stop-Transcript
```

Install_File_Server_Role_And_Management_Tools_Management_Tools_Skeleton
```
# Run in elevated PowerShell on the file server or Windows Server management host.
# Purpose: install or validate File Services management tools.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-file-services-management-tools-transcript.txt"

# Confirm ServerManager module.
Get-Module ServerManager -ListAvailable |
  Tee-Object "$EvidencePath\servermanager-module.txt"

# Install File Services RSAT tools on Windows Server if needed.
Install-WindowsFeature `
  -Name RSAT-File-Services `
  -IncludeAllSubFeature

# Confirm management tools state.
Get-WindowsFeature RSAT-File-Services |
  Tee-Object "$EvidencePath\rsat-file-services-after.txt"

Get-WindowsFeature |
  Where-Object {
    $_.Name -like "RSAT-File*" -or
    $_.Name -like "FS-*"
  } |
  Sort-Object Name |
  Tee-Object "$EvidencePath\file-services-and-rsat-features-after.txt"

# Confirm management cmdlets.
Get-Command -Module SmbShare |
  Tee-Object "$EvidencePath\smbshare-cmdlets.txt"

Get-Command -Module ServerManager |
  Tee-Object "$EvidencePath\servermanager-cmdlets.txt"

Stop-Transcript
```

Install_File_Server_Role_And_Management_Tools_Remote_Management_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: validate and enable remote management for administration.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-remote-management-transcript.txt"

# Confirm WinRM service.
Get-Service WinRM |
  Tee-Object "$EvidencePath\winrm-service-before.txt"

# Test local WinRM listener.
Test-WSMan localhost |
  Tee-Object "$EvidencePath\test-wsman-localhost-before.txt"

# Enable PowerShell remoting if required.
Enable-PSRemoting -Force

# Confirm WinRM service after enabling remoting.
Get-Service WinRM |
  Tee-Object "$EvidencePath\winrm-service-after.txt"

Test-WSMan localhost |
  Tee-Object "$EvidencePath\test-wsman-localhost-after.txt"

# Confirm firewall rules related to remote management.
Get-NetFirewallRule |
  Where-Object {
    $_.DisplayGroup -like "*Windows Remote Management*" -or
    $_.DisplayGroup -like "*Remote Event Log Management*" -or
    $_.DisplayGroup -like "*File and Printer Sharing*"
  } |
  Sort-Object DisplayGroup,DisplayName |
  Tee-Object "$EvidencePath\remote-management-firewall-rules.txt"

Stop-Transcript
```

Install_File_Server_Role_And_Management_Tools_Post_Install_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: validate File Server role, services, shares, cmdlets, and evidence after install.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-post-install-validation-transcript.txt"

# Confirm File Server role state.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-final.txt"

# Confirm all file service feature states.
Get-WindowsFeature FS-* |
  Sort-Object Name |
  Tee-Object "$EvidencePath\file-services-features-final.txt"

Get-WindowsFeature |
  Export-Csv "$EvidencePath\windows-features-final.csv" -NoTypeInformation

# Confirm services.
Get-Service LanmanServer,LanmanWorkstation,WinRM -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\core-services-final.txt"

# Confirm SMB management cmdlets.
Get-Command -Module SmbShare |
  Tee-Object "$EvidencePath\smbshare-cmdlets-final.txt"

# Confirm current SMB shares.
Get-SmbShare |
  Sort-Object Name |
  Tee-Object "$EvidencePath\smb-shares-final.txt"

Get-SmbShare |
  Sort-Object Name |
  Export-Csv "$EvidencePath\smb-shares-final.csv" -NoTypeInformation

# Confirm SMB server configuration.
Get-SmbServerConfiguration |
  Tee-Object "$EvidencePath\smb-server-configuration-final.txt"

# Confirm no immediate Server or SMB errors in recent logs.
Get-WinEvent -LogName System -MaxEvents 100 |
  Where-Object {
    $_.ProviderName -like "*Server*" -or
    $_.ProviderName -like "*SMB*" -or
    $_.ProviderName -like "*Service Control Manager*"
  } |
  Tee-Object "$EvidencePath\recent-system-file-server-related-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Operational" -MaxEvents 50 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\recent-smbserver-operational-events.txt"

Stop-Transcript
```


Install_File_Server_Role_And_Management_Tools_Verification_Commands

```
# Identity and baseline
hostname
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName
Get-NetIPConfiguration
Get-Disk
Get-Volume

# File Server role and management tools
Get-WindowsFeature FS-FileServer
Get-WindowsFeature RSAT-File-Services
Get-WindowsFeature FS-*
Get-Module ServerManager -ListAvailable
Get-Command -Module ServerManager
Get-Command -Module SmbShare

# SMB service and default shares
Get-Service LanmanServer
Get-Service LanmanWorkstation
Get-SmbServerConfiguration
Get-SmbShare

# Remote management
Get-Service WinRM
Test-WSMan localhost
Test-WSMan "<file-server-name>"

# Firewall visibility
Get-NetFirewallRule -DisplayGroup "File and Printer Sharing"
Get-NetFirewallRule -DisplayGroup "Windows Remote Management"

# Events
Get-WinEvent -LogName System -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Operational" -MaxEvents 50
```

Install_File_Server_Role_And_Management_Tools_Rollback

|      |                                                               |             |                                                                                                                       |                                         |
| ---- | ------------------------------------------------------------- | ----------- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Step | Task                                                          | Device      | PowerShell / Command                                                                                                  | Expected Result                         |
| 1    | Confirm no production shares were created yet                 | File Server | Get-SmbShare                                                                                                          | Only default or lab shares exist        |
| 2    | Confirm no later workbook features depend on File Server role | File Server | Get-WindowsFeature FS-*                                                                                               | Dependency risk is known                |
| 3    | Export current feature state before rollback                  | File Server | Get-WindowsFeature \| Export-Csv "C:\FileServices-Validation\windows-features-before-rollback.csv" -NoTypeInformation | Rollback evidence is saved              |
| 4    | Remove File Server role if still in early lab stage           | File Server | Uninstall-WindowsFeature FS-FileServer                                                                                | File Server role is removed             |
| 5    | Remove File Services management tools if required             | File Server | Uninstall-WindowsFeature RSAT-File-Services                                                                           | File Services RSAT tools are removed    |
| 6    | Reboot if requested                                           | File Server | Restart-Computer                                                                                                      | Server restarts cleanly                 |
| 7    | Confirm role removal                                          | File Server | Get-WindowsFeature FS-FileServer                                                                                      | Install State no longer shows Installed |
| 8    | Confirm SMB shares after rollback                             | File Server | Get-SmbShare                                                                                                          | No unintended user shares remain        |
| 9    | Document rollback result                                      | Operator    | Record removed features, reboot status, and validation output                                                         | Rollback record is complete             |

Install_File_Server_Role_And_Management_Tools_Failure_Checks

|                                                  |                                                                   |                                                              |                                                            |
| ------------------------------------------------ | ----------------------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| Symptom                                          | Likely Cause                                                      | Check                                                        | Corrective Action                                          |
| Install-WindowsFeature is not recognized         | ServerManager module unavailable or not running on Windows Server | Get-Module ServerManager -ListAvailable                      | Run from Windows Server or import ServerManager            |
| File Server role install fails                   | PowerShell not elevated                                           | whoami /groups                                               | Reopen PowerShell as Administrator                         |
| Feature source files unavailable                 | Component store or installation source issue                      | Review install error and System log                          | Repair component store or specify valid source media       |
| Server cannot resolve domain                     | DNS client points to wrong DNS server                             | Get-DnsClientServerAddress; Resolve-DnsName <domain-fqdn>    | Point server DNS to AD DNS                                 |
| Server is not domain joined                      | Server still in workgroup                                         | Get-ComputerInfo \| Select CsDomain,CsPartOfDomain           | Join domain before building domain-based file permissions  |
| SMB cmdlets missing                              | SMBShare module not loaded or OS mismatch                         | Get-Command -Module SmbShare                                 | Import module or confirm supported Windows Server version  |
| Get-SmbShare shows only admin shares             | No user shares created yet                                        | Get-SmbShare                                                 | Normal after role install. Create shares in later workbook |
| WinRM remote test fails                          | PowerShell remoting not enabled                                   | Test-WSMan <file-server>                                     | Run Enable-PSRemoting -Force                               |
| WinRM remote test fails from admin host          | Firewall or network path blocked                                  | Test-NetConnection <file-server> -Port 5985                  | Allow WinRM from admin network                             |
| Server Manager cannot manage server              | Remote management disabled or credential issue                    | Server Manager manageability status                          | Enable remoting and use domain admin or delegated admin    |
| File and Printer Sharing firewall rules disabled | Firewall profile or GPO blocks SMB                                | Get-NetFirewallRule -DisplayGroup "File and Printer Sharing" | Enable appropriate inbound SMB rules only where required   |
| Role installs but service is stopped             | Server service not running                                        | Get-Service LanmanServer                                     | Start service and check System logs                        |
| Unexpected reboot required                       | Feature dependency or pending reboot existed                      | Get-ItemProperty pending reboot checks                       | Reboot during maintenance window                           |
| Optional features installed too early            | DFS, FSRM, NFS, iSCSI, or dedup added before planning             | Get-WindowsFeature FS-*                                      | Leave them if planned, or uninstall before continuing      |
| Validation evidence missing                      | Evidence path not created or transcript failed                    | Test-Path C:\FileServices-Validation                         | Re-run validation skeleton with elevated PowerShell        |


