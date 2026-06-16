19_Configure_BranchCache_For_File_Services.md
# Configure_BranchCache_For_File_Services

# Configure_BranchCache_For_File_Services_Index
19_Configure_BranchCache_For_File_Services.md
Configure_BranchCache_For_File_Services
Configure_BranchCache_For_File_Services_Source_Basis
Configure_BranchCache_For_File_Services_Mental_Model
Configure_BranchCache_For_File_Services_Planning_Table
Configure_BranchCache_For_File_Services_Configuration_Checklist
Configure_BranchCache_For_File_Services_Content_Server_Skeleton
Configure_BranchCache_For_File_Services_SMB_Share_BranchCache_Skeleton
Configure_BranchCache_For_File_Services_Distributed_Cache_Client_Skeleton
Configure_BranchCache_For_File_Services_Hosted_Cache_Server_Skeleton
Configure_BranchCache_For_File_Services_Hosted_Cache_Client_Skeleton
Configure_BranchCache_For_File_Services_GPO_Planning_Skeleton
Configure_BranchCache_For_File_Services_Content_Hash_Publication_Skeleton
Configure_BranchCache_For_File_Services_Status_And_Event_Review_Skeleton
Configure_BranchCache_For_File_Services_Verification_Commands
Configure_BranchCache_For_File_Services_Rollback
Configure_BranchCache_For_File_Services_Failure_Checks
Configure_BranchCache_For_File_Services_Related_Labs

# Configure_BranchCache_For_File_Services_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| BranchCache PowerShell module | Enable-BCDistributed | Configuring branch clients for distributed cache mode |
| BranchCache PowerShell module | Enable-BCHostedClient | Configuring branch clients for hosted cache mode |
| BranchCache PowerShell module | Enable-BCHostedServer | Configuring a hosted cache server |
| BranchCache PowerShell module | Get-BCStatus | Reviewing BranchCache status and configuration |
| BranchCache PowerShell module | Publish-BCFileContent | Generating BranchCache hashes for file share content |
| SMB Share PowerShell module | Set-SmbShare -CachingMode BranchCache | Enabling BranchCache caching mode on SMB shares |
| Windows Server File Services | BranchCache for Network Files | Allowing SMB content to participate in BranchCache |
| Windows Firewall | BranchCache firewall rules | Allowing client discovery and cache retrieval |
| Group Policy | BranchCache client policy | Controlling BranchCache behavior at scale |
| Operational file server practice | WAN file services | Reducing repeated WAN downloads from central file servers |

# Configure_BranchCache_For_File_Services_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BranchCache | WAN optimization feature that lets branch clients reuse cached content instead of repeatedly pulling the same data across the WAN |
| Content server | Central file server that hosts the original SMB share content |
| Branch client | Windows client at the remote site that requests file content |
| Distributed cache mode | Branch clients cache and share content with each other peer-to-peer |
| Hosted cache mode | Branch clients cache and retrieve content from a dedicated hosted cache server in the branch |
| Hosted cache server | Server in the branch office that stores cached content for clients |
| Hash publication | Content server generates content information so clients can discover cached data |
| SMB share caching mode | Per-share setting that allows BranchCache behavior on a file share |
| BranchCache for Network Files | File Services role service that lets SMB file content participate in BranchCache |
| Cache retrieval | Client obtains cached content from a peer or hosted cache server after validating content information |
| WAN threshold | BranchCache is useful when central file server access crosses a slow or high-latency link |
| GPO control | Normal enterprise deployment should use Group Policy instead of one-off local configuration |
| Blunt rule | BranchCache does not replace DFS, DFSR, OneDrive, or a local file server. It only reduces repeated WAN pulls for eligible content |
| Design rule | Use distributed cache for small branches and hosted cache for larger branches or where cache availability matters |

# Configure_BranchCache_For_File_Services_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Central file server | `FS1` | `<content-server-name>` |
| Central file server FQDN | `FS1.corp.local` | `<content-server-fqdn>` |
| Target share | `Departments` | `<share-name>` |
| Target share path | `D:\Shares\Departments` | `<share-path>` |
| Branch site name | `BRANCH01` | `<branch-site-name>` |
| Branch subnet | `10.20.10.0/24` | `<branch-subnet>` |
| Branch client | `BRANCH-WIN11-01` | `<branch-client-name>` |
| Branch client IP | `10.20.10.25` | `<branch-client-ip>` |
| Hosted cache server | `BCHOST01` | `<hosted-cache-server-name>` |
| Hosted cache server FQDN | `BCHOST01.corp.local` | `<hosted-cache-server-fqdn>` |
| BranchCache mode | Distributed / Hosted | `<branchcache-mode>` |
| Cache size | `10%` or `20 GB` | `<cache-size>` |
| Cache path | `D:\BranchCache` | `<cache-path>` |
| SMB caching mode | `BranchCache` | `BranchCache` |
| Hash prepublication | Yes / No | `<publish-hashes-now>` |
| GPO name | `GPO_BranchCache_FileServices` | `<gpo-name>` |
| Client OU | `OU=BranchClients,OU=Workstations,DC=corp,DC=local` | `<client-ou>` |
| Hosted cache OU | `OU=BranchServers,DC=corp,DC=local` | `<server-ou>` |
| Firewall profile | Domain | `<firewall-profile>` |
| Test file | `branchcache-test.iso` | `<test-file>` |
| Report path | `C:\BranchCache-Reports` | `<report-path>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Configure_BranchCache_For_File_Services_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm central file server identity | Content Server | `hostname` | Correct file server is identified |
| 2 | Confirm target share exists | Content Server | `Get-SmbShare -Name "<share-name>"` | Target SMB share exists |
| 3 | Confirm share path exists | Content Server | `Test-Path "<share-path>"` | Backing folder exists |
| 4 | Confirm File Services feature state | Content Server | `Get-WindowsFeature FS-FileServer,FS-BranchCache` | File Server and BranchCache role service state are visible |
| 5 | Install BranchCache for Network Files | Content Server | `Install-WindowsFeature FS-BranchCache -IncludeManagementTools` | BranchCache for Network Files is installed |
| 6 | Import BranchCache module | Content Server | `Import-Module BranchCache` | BranchCache cmdlets are available |
| 7 | Confirm BranchCache cmdlets | Content Server | `Get-Command -Module BranchCache` | BranchCache commands are listed |
| 8 | Enable BranchCache on target SMB share | Content Server | `Set-SmbShare -Name "<share-name>" -CachingMode BranchCache -Force` | Share caching mode is BranchCache |
| 9 | Confirm share caching mode | Content Server | `Get-SmbShare -Name "<share-name>" | Select Name,Path,CachingMode` | Target share shows `BranchCache` |
| 10 | Confirm content server BranchCache status | Content Server | `Get-BCStatus` | Content server BranchCache state is visible |
| 11 | Generate hashes for existing content | Content Server | `Publish-BCFileContent -Path "<share-path>" -Recurse` | File content hashes are generated |
| 12 | Confirm BranchCache firewall rules | Content Server | `Get-NetFirewallRule -DisplayGroup "BranchCache"` | BranchCache firewall rules are visible |
| 13 | Enable BranchCache firewall rules if needed | Content Server | `Get-NetFirewallRule -DisplayGroup "BranchCache" | Enable-NetFirewallRule` | BranchCache firewall rules are enabled |
| 14 | Decide branch cache mode | Operator | `Distributed for small branches, hosted for larger branches` | Deployment mode is selected |
| 15 | Install BranchCache feature on hosted cache server if using hosted mode | Hosted Cache Server | `Install-WindowsFeature BranchCache -IncludeManagementTools` | BranchCache feature is installed |
| 16 | Enable hosted cache server if using hosted mode | Hosted Cache Server | `Enable-BCHostedServer -RegisterSCP -Force` | Hosted cache mode is enabled and SCP registration is attempted |
| 17 | Confirm hosted cache server status | Hosted Cache Server | `Get-BCStatus` | Hosted cache server configuration is visible |
| 18 | Enable BranchCache firewall rules on hosted cache server | Hosted Cache Server | `Get-NetFirewallRule -DisplayGroup "BranchCache" | Enable-NetFirewallRule` | Hosted cache traffic is allowed |
| 19 | Enable distributed cache mode on test client if selected | Branch Client | `Enable-BCDistributed -Force` | Client is configured for distributed cache mode |
| 20 | Enable hosted cache client mode on test client if selected | Branch Client | `Enable-BCHostedClient -ServerNames "<hosted-cache-server-fqdn>" -Force` | Client is configured for hosted cache mode |
| 21 | Confirm client BranchCache status | Branch Client | `Get-BCStatus` | Client mode and network settings are visible |
| 22 | Confirm client can reach content server SMB | Branch Client | `Test-NetConnection "<content-server-fqdn>" -Port 445` | TCP 445 succeeds |
| 23 | Confirm client can reach hosted cache if used | Branch Client | `Test-NetConnection "<hosted-cache-server-fqdn>" -Port 80` | Hosted cache HTTP retrieval path is reachable |
| 24 | Access test file from file share | Branch Client | `Copy-Item "\\<content-server-name>\<share-name>\<test-file>" "C:\Temp\<test-file>"` | First retrieval succeeds |
| 25 | Access same test file again from another branch client | Branch Client | `Copy-Item "\\<content-server-name>\<share-name>\<test-file>" "C:\Temp\<test-file>"` | Second retrieval succeeds and should benefit from cache if eligible |
| 26 | Review BranchCache status after access | Branch Client | `Get-BCStatus` | BranchCache counters and config are visible |
| 27 | Review BranchCache event logs | Branch Client | `Get-WinEvent -ListLog "*BranchCache*"` | BranchCache logs are discoverable |
| 28 | Export configuration evidence | All | `Get-BCStatus | Export-Clixml "C:\BranchCache-Reports\bc-status.xml"` | BranchCache configuration is documented |
| 29 | Document final state | Operator | `Record mode, share, clients, hosted cache server, firewall, test file, and verification result` | Change record is complete |

# Configure_BranchCache_For_File_Services_Content_Server_Skeleton
```powershell
# Run in elevated PowerShell on the central file server.
# Purpose: install BranchCache for Network Files and validate content server readiness.

$Features = @(
  "FS-FileServer",
  "FS-BranchCache"
)

$ReportRoot = "C:\BranchCache-Reports"
New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Checking file server feature state..."
Get-WindowsFeature $Features

Write-Host "Installing BranchCache for Network Files..."
Install-WindowsFeature FS-BranchCache -IncludeManagementTools

Write-Host "Importing BranchCache module..."
Import-Module BranchCache

Write-Host "BranchCache cmdlets:"
Get-Command -Module BranchCache | Sort-Object Name

Write-Host "Current BranchCache status:"
Get-BCStatus | Format-List *

Write-Host "BranchCache firewall rules:"
Get-NetFirewallRule -DisplayGroup "BranchCache" -ErrorAction SilentlyContinue |
  Select-Object DisplayName, Enabled, Direction, Action, Profile |
  Format-Table -AutoSize
```

# Configure_BranchCache_For_File_Services_SMB_Share_BranchCache_Skeleton
```powershell
# Run in elevated PowerShell on the central file server.
# Purpose: enable BranchCache caching mode on a target SMB share.

$ShareName = "Departments"
$SharePath = "D:\Shares\Departments"

Write-Host "Confirming target SMB share..."
Get-SmbShare -Name $ShareName | Format-List *

Write-Host "Confirming target share path..."
Test-Path $SharePath

Write-Host "Current SMB share caching mode:"
Get-SmbShare -Name $ShareName |
  Select-Object Name, Path, CachingMode |
  Format-Table -AutoSize

Write-Host "Setting SMB share caching mode to BranchCache..."
Set-SmbShare -Name $ShareName -CachingMode BranchCache -Force

Write-Host "SMB share after BranchCache caching mode update:"
Get-SmbShare -Name $ShareName |
  Select-Object Name, Path, CachingMode, EncryptData, FolderEnumerationMode |
  Format-Table -AutoSize
```

# Configure_BranchCache_For_File_Services_Distributed_Cache_Client_Skeleton
```powershell
# Run in elevated PowerShell on a branch Windows client.
# Purpose: configure distributed cache mode for a small branch or lab scenario.

$ContentServer = "FS1.corp.local"
$ShareName = "Departments"
$TestFile = "branchcache-test.iso"
$LocalTestPath = "C:\Temp"

New-Item -ItemType Directory -Force -Path $LocalTestPath | Out-Null

Write-Host "Importing BranchCache module..."
Import-Module BranchCache

Write-Host "Enabling distributed cache mode..."
Enable-BCDistributed -Force

Write-Host "Enabling BranchCache firewall rules..."
Get-NetFirewallRule -DisplayGroup "BranchCache" -ErrorAction SilentlyContinue |
  Enable-NetFirewallRule

Write-Host "BranchCache status after distributed mode configuration:"
Get-BCStatus | Format-List *

Write-Host "Testing SMB connectivity to content server..."
Test-NetConnection $ContentServer -Port 445

Write-Host "Copying test file from SMB share..."
Copy-Item "\\$ContentServer\$ShareName\$TestFile" "$LocalTestPath\$TestFile" -Force

Write-Host "BranchCache status after test file retrieval:"
Get-BCStatus | Format-List *
```

# Configure_BranchCache_For_File_Services_Hosted_Cache_Server_Skeleton
```powershell
# Run in elevated PowerShell on the hosted cache server in the branch office.
# Purpose: configure hosted cache server mode.

$ReportRoot = "C:\BranchCache-Reports"
New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Installing BranchCache feature..."
Install-WindowsFeature BranchCache -IncludeManagementTools

Write-Host "Importing BranchCache module..."
Import-Module BranchCache

Write-Host "Enabling hosted cache server mode and registering SCP in AD if permitted..."
Enable-BCHostedServer -RegisterSCP -Force

Write-Host "Enabling BranchCache firewall rules..."
Get-NetFirewallRule -DisplayGroup "BranchCache" -ErrorAction SilentlyContinue |
  Enable-NetFirewallRule

Write-Host "Hosted cache server status:"
Get-BCStatus | Format-List *

Write-Host "Hosted cache server configuration:"
Get-BCHostedCacheServerConfiguration | Format-List * -ErrorAction SilentlyContinue

Write-Host "BranchCache network configuration:"
Get-BCNetworkConfiguration | Format-List * -ErrorAction SilentlyContinue

Get-BCStatus |
  Export-Clixml "$ReportRoot\hosted-cache-server-bc-status.xml"
```

# Configure_BranchCache_For_File_Services_Hosted_Cache_Client_Skeleton
```powershell
# Run in elevated PowerShell on a branch Windows client.
# Purpose: configure a branch client to use a hosted cache server.

$HostedCacheServer = "BCHOST01.corp.local"
$ContentServer = "FS1.corp.local"
$ShareName = "Departments"
$TestFile = "branchcache-test.iso"
$LocalTestPath = "C:\Temp"

New-Item -ItemType Directory -Force -Path $LocalTestPath | Out-Null

Write-Host "Importing BranchCache module..."
Import-Module BranchCache

Write-Host "Testing SMB access to central content server..."
Test-NetConnection $ContentServer -Port 445

Write-Host "Testing hosted cache server reachability..."
Test-NetConnection $HostedCacheServer -Port 80

Write-Host "Configuring hosted cache client mode..."
Enable-BCHostedClient -ServerNames $HostedCacheServer -Force

Write-Host "Enabling BranchCache firewall rules..."
Get-NetFirewallRule -DisplayGroup "BranchCache" -ErrorAction SilentlyContinue |
  Enable-NetFirewallRule

Write-Host "Client BranchCache status:"
Get-BCStatus | Format-List *

Write-Host "Copying test file from SMB share..."
Copy-Item "\\$ContentServer\$ShareName\$TestFile" "$LocalTestPath\$TestFile" -Force

Write-Host "Client BranchCache status after retrieval:"
Get-BCStatus | Format-List *
```

# Configure_BranchCache_For_File_Services_GPO_Planning_Skeleton
```text
# Use this section for domain-scale deployment.
# Configure these settings through Group Policy Management Console.

GPO Name:
GPO_BranchCache_FileServices

Suggested Scope:
Branch client computer OU
Hosted cache server OU if hosted mode is used

Policy Path:
Computer Configuration
Policies
Administrative Templates
Network
BranchCache

Client Policy Decisions:
1. Turn on BranchCache
2. Choose Distributed Cache mode or Hosted Cache mode
3. If hosted mode, configure hosted cache server name
4. Configure client cache size
5. Configure firewall rules through Windows Defender Firewall policy if needed
6. Configure network files latency threshold if using policy controls for SMB caching behavior

File Server Policy Decisions:
1. Install BranchCache for Network Files on the content server
2. Set SMB share CachingMode to BranchCache
3. Publish content hashes for large existing datasets if needed
4. Confirm SMB signing and encryption policy does not conflict with the intended test model

Verification:
gpupdate /force
gpresult /h C:\Temp\branchcache-gpresult.html
Get-BCStatus
Get-SmbShare -Name "<share-name>" | Select Name,Path,CachingMode
```

# Configure_BranchCache_For_File_Services_Content_Hash_Publication_Skeleton
```powershell
# Run in elevated PowerShell on the central file server.
# Purpose: generate BranchCache content information for existing files in a target share path.

$ShareName = "Departments"
$SharePath = "D:\Shares\Departments"
$ReportRoot = "C:\BranchCache-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Confirming BranchCache caching mode on SMB share..."
Get-SmbShare -Name $ShareName |
  Select-Object Name, Path, CachingMode |
  Format-Table -AutoSize

Write-Host "Generating BranchCache hashes for files in $SharePath..."
Publish-BCFileContent -Path $SharePath -Recurse -Force

Write-Host "BranchCache content server configuration:"
Get-BCContentServerConfiguration |
  Format-List * |
  Out-File "$ReportRoot\bc-content-server-config-$Timestamp.txt"

Write-Host "BranchCache status:"
Get-BCStatus |
  Format-List * |
  Out-File "$ReportRoot\bc-status-content-server-$Timestamp.txt"

Write-Host "Hash publication pass completed for $SharePath"
```

# Configure_BranchCache_For_File_Services_Status_And_Event_Review_Skeleton
```powershell
# Run in elevated PowerShell on content server, hosted cache server, and branch clients as applicable.
# Purpose: export BranchCache status, firewall state, and event evidence.

$ReportRoot = "C:\BranchCache-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Export BranchCache status..."
Get-BCStatus |
  Export-Clixml "$ReportRoot\bc-status-$Timestamp.xml"

Write-Host "Export BranchCache client configuration if present..."
Get-BCClientConfiguration -ErrorAction SilentlyContinue |
  Export-Clixml "$ReportRoot\bc-client-config-$Timestamp.xml"

Write-Host "Export BranchCache content server configuration if present..."
Get-BCContentServerConfiguration -ErrorAction SilentlyContinue |
  Export-Clixml "$ReportRoot\bc-content-server-config-$Timestamp.xml"

Write-Host "Export hosted cache server configuration if present..."
Get-BCHostedCacheServerConfiguration -ErrorAction SilentlyContinue |
  Export-Clixml "$ReportRoot\bc-hosted-cache-server-config-$Timestamp.xml"

Write-Host "Export BranchCache network configuration..."
Get-BCNetworkConfiguration -ErrorAction SilentlyContinue |
  Export-Clixml "$ReportRoot\bc-network-config-$Timestamp.xml"

Write-Host "Export BranchCache firewall rules..."
Get-NetFirewallRule -DisplayGroup "BranchCache" -ErrorAction SilentlyContinue |
  Select-Object DisplayName, Enabled, Direction, Action, Profile |
  Export-Csv "$ReportRoot\bc-firewall-rules-$Timestamp.csv" -NoTypeInformation

Write-Host "Export available BranchCache logs..."
Get-WinEvent -ListLog "*BranchCache*" -ErrorAction SilentlyContinue |
  Select-Object LogName, IsEnabled, RecordCount |
  Export-Csv "$ReportRoot\bc-eventlog-inventory-$Timestamp.csv" -NoTypeInformation

Write-Host "Export recent BranchCache events if logs exist..."
Get-WinEvent -ListLog "*BranchCache*" -ErrorAction SilentlyContinue |
  ForEach-Object {
    $LogName = $_.LogName
    Get-WinEvent -LogName $LogName -MaxEvents 100 -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$($LogName.Replace('/','-'))-$Timestamp.csv" -NoTypeInformation
  }

Write-Host "BranchCache report data exported to $ReportRoot"
```

# Configure_BranchCache_For_File_Services_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BranchCache module | `Get-Command -Module BranchCache` | BranchCache cmdlets are available |
| Verify content server feature | `Get-WindowsFeature FS-BranchCache` | BranchCache for Network Files is installed |
| Verify content server status | `Get-BCStatus` | BranchCache status is visible |
| Verify target SMB share | `Get-SmbShare -Name "<share-name>"` | Target SMB share exists |
| Verify share caching mode | `Get-SmbShare -Name "<share-name>" | Select Name,Path,CachingMode` | CachingMode shows `BranchCache` |
| Verify content hash command | `Publish-BCFileContent -Path "<share-path>" -Recurse -WhatIf` | Command validates target path without making changes |
| Verify content server SMB reachability | `Test-NetConnection "<content-server-fqdn>" -Port 445` | TCP 445 succeeds from branch client |
| Verify hosted cache server status | `Get-BCHostedCacheServerConfiguration` | Hosted cache configuration is visible |
| Verify hosted cache reachability | `Test-NetConnection "<hosted-cache-server-fqdn>" -Port 80` | Client can reach hosted cache server |
| Verify distributed client mode | `Get-BCClientConfiguration` | Client shows BranchCache client settings |
| Verify full BranchCache status | `Get-BCStatus` | Client, content server, network, and hosted cache info are visible |
| Verify firewall rules | `Get-NetFirewallRule -DisplayGroup "BranchCache"` | BranchCache firewall rules are enabled where needed |
| Verify GPO application | `gpresult /h C:\Temp\branchcache-gpresult.html` | BranchCache GPO appears in applied computer policy |
| Verify event logs | `Get-WinEvent -ListLog "*BranchCache*"` | BranchCache event logs are discoverable |
| Verify file access | `Copy-Item "\\<content-server-name>\<share-name>\<test-file>" "C:\Temp\<test-file>" -Force` | SMB file retrieval succeeds |
| Verify SMB share still works | `Test-Path "\\<content-server-name>\<share-name>"` | Share remains reachable and authorized |

# Configure_BranchCache_For_File_Services_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| SMB share caching mode set to BranchCache | `Set-SmbShare -Name "<share-name>" -CachingMode Manual -Force` | Share no longer uses BranchCache caching mode |
| BranchCache distributed mode enabled on client | `Disable-BC -Force` | BranchCache is disabled on client |
| Hosted cache client mode enabled | `Disable-BC -Force` | Client stops using hosted cache mode |
| Hosted cache server enabled | `Disable-BC -Force` | Hosted cache server mode is disabled |
| BranchCache cache populated | `Clear-BCCache -Force` | Local BranchCache cache is cleared |
| BranchCache firewall rules enabled | `Get-NetFirewallRule -DisplayGroup "BranchCache" | Disable-NetFirewallRule` | BranchCache firewall rules are disabled |
| BranchCache for Network Files installed | `Uninstall-WindowsFeature FS-BranchCache` | Content server role service is removed |
| BranchCache feature installed on hosted cache server | `Uninstall-WindowsFeature BranchCache` | BranchCache feature is removed |
| GPO created for BranchCache | Unlink or disable `GPO_BranchCache_FileServices` | BranchCache policy no longer applies |
| Reports created | `Remove-Item "C:\BranchCache-Reports" -Recurse -Force` | Report folder is removed |

# Configure_BranchCache_For_File_Services_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| BranchCache cmdlets not found | BranchCache feature or management tools missing | `Get-Command -Module BranchCache` | Install BranchCache feature or role service |
| `FS-BranchCache` missing or not installed | Content server role service not installed | `Get-WindowsFeature FS-BranchCache` | Install `FS-BranchCache` |
| Share does not cache | SMB share caching mode not set to BranchCache | `Get-SmbShare -Name "<share-name>" | Select CachingMode` | Set `-CachingMode BranchCache` |
| Clients always pull across WAN | BranchCache disabled, wrong mode, threshold not met, or no eligible content | `Get-BCStatus`; `Get-BCClientConfiguration` | Enable client mode and validate policy |
| Hosted cache client cannot find server | Wrong hosted cache name, DNS issue, SCP issue, or firewall | `Resolve-DnsName`; `Test-NetConnection <hosted-cache> -Port 80` | Fix DNS, GPO, firewall, or specify server directly |
| Distributed cache clients do not share content | Firewall disabled, multicast/peer discovery blocked, clients on different networks | `Get-NetFirewallRule -DisplayGroup "BranchCache"` | Enable firewall rules and validate network design |
| Hosted cache server status blank | Hosted cache mode not enabled | `Get-BCHostedCacheServerConfiguration` | Run `Enable-BCHostedServer` |
| Hash publication fails | Path wrong, feature missing, access denied, or unsupported file state | `Test-Path <share-path>`; `Publish-BCFileContent -WhatIf` | Correct path, permissions, or feature install |
| GPO says applied but client still wrong | Policy conflict or local override | `gpresult /h C:\Temp\branchcache.html`; `Get-BCStatus` | Fix GPO scope, link order, security filtering |
| SMB access fails after change | Separate SMB permission, DNS, or firewall issue | `Test-Path "\\server\share"`; `Test-NetConnection -Port 445` | Troubleshoot SMB path separately |
| No visible benefit in lab | File too small, same client test only, low latency, no repeated access, or cache not warmed | Repeat from second branch client with large file | Test with realistic branch traffic |
| BranchCache events missing | Wrong log or no BranchCache activity | `Get-WinEvent -ListLog "*BranchCache*"` | Generate file access and query discovered logs |
| Cache consumes too much disk | Cache size not controlled | `Get-BCStatus`; `Get-BCDataCache` | Configure cache size through policy or `Set-BCCache` |
| Security concern about cached data | Misunderstanding of cache behavior or unsupported client placement | Review client placement and policy scope | Limit BranchCache to managed domain clients |
| Users complain file is stale | Normal SMB issue, offline files issue, or app cache issue | `Get-SmbConnection`; file timestamp; Offline Files status | Do not blame BranchCache first. Validate SMB and client cache state |

# Configure_BranchCache_For_File_Services_Related_Labs
| Lab                                                            | Relationship                                                               |
| -------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `01_Install_File_Server_Role_And_Management_Tools.md`          | Establishes the Windows file server baseline                               |
| `02_Create_And_Publish_SMB_Shares.md`                          | BranchCache is enabled against existing SMB shares                         |
| `03_Configure_NTFS_And_Share_Permissions.md`                   | BranchCache does not replace NTFS or share permissions                     |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md` | SMB security posture must remain compatible with file access requirements  |
| `13_Configure_DFS_Namespaces.md`                               | DFS namespace paths can front BranchCache-enabled SMB shares               |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md` | SMB monitoring helps confirm file access behavior during BranchCache tests |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`        | Separates BranchCache behavior from ordinary SMB failures                  |
| `17_Configure_Data_Deduplication_For_File_Shares.md`           | Deduplication and BranchCache can affect the same file server datasets     |
| `18_Configure_NFS_Shares_For_Non_Windows_Clients.md`           | Contrasts SMB BranchCache with non-Windows NFS access models               |