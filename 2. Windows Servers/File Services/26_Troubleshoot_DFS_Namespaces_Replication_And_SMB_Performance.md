26_Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance.md
# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Index
26_Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance.md
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Source_Basis
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Mental_Model
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Planning_Table
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Symptom_Map
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Configuration_Checklist
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSN_Referral_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSN_Topology_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSR_Backlog_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSR_State_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSR_Propagation_Test_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_SMB_Client_Baseline_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_SMB_Server_Baseline_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_SMB_Performance_Counter_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Network_Latency_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Event_Log_Collection_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Safe_Remediation_Skeleton
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Verification_Commands
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Rollback
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Failure_Checks
Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Related_Labs

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| DFS Namespaces overview | Namespace server, namespace root, folders, folder targets, referrals | Troubleshooting namespace path resolution and target selection |
| DFSN PowerShell module | Get-DfsnRoot, Get-DfsnFolder, Get-DfsnFolderTarget, Set-DfsnFolderTarget | Reviewing namespace structure and target state |
| DFS command-line tools | dfsutil, dfsdiag | Referral cache, namespace integrity, and namespace diagnostics |
| DFS Replication overview | Replication groups, members, replicated folders, topology | Troubleshooting replication design and state |
| DFSR PowerShell module | Get-DfsrBacklog, Get-DfsrState, Get-DfsrConnection, Get-DfsrMembership | Backlog, state, connection, and membership analysis |
| DFSR PowerShell module | Write-DfsrHealthReport, Start-DfsrPropagationTest, Write-DfsrPropagationReport | Health reporting and propagation validation |
| SMB Share PowerShell module | Get-SmbConnection, Get-SmbMultichannelConnection, Get-SmbSession, Get-SmbOpenFile | SMB client and server path validation |
| File server performance tuning | Client and SMB performance parameters | SMB latency, cache, signing, multichannel, and throughput review |
| Windows Event Logs | DFS Replication, DFS Namespace, SMBClient, SMBServer, System | Event-driven troubleshooting |
| Operational file server practice | User impact, cache clearing, target isolation, safe cutover | Production-safe troubleshooting flow |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DFS Namespace | Logical user-facing path that redirects clients to real SMB shares |
| Namespace root | Starting point of the DFS path, such as `\\corp.local\Shares` |
| Namespace server | Server hosting the namespace root |
| DFS folder | Logical folder under the namespace |
| Folder target | Real UNC path behind the DFS folder, such as `\\FS1\Departments` |
| DFS referral | Client response that tells the workstation which target to use |
| Referral cache | Client-side cached DFS target decision |
| Site costing | AD Sites and Services influence which target is considered closest |
| DFS Replication | Replicates folder content between servers |
| Replication group | Logical DFSR group containing replicated folders and members |
| Replicated folder | Data set replicated by DFSR |
| DFSR member | Server participating in a replication group |
| DFSR backlog | Pending updates from one member to another |
| DFSR state | Current DFSR processing state for files and database activity |
| SMB path | Actual file access path after DFS referral selects a target |
| SMB session | Authenticated client connection to the selected file server |
| SMB Multichannel | SMB uses multiple network paths when supported and enabled |
| SMB signing | Security feature that can add CPU and latency overhead |
| SMB encryption | Security feature that can add CPU overhead |
| Slow DFS path | Could be namespace referral, DFSR backlog, SMB target, network, disk, antivirus, client cache, or AD site mapping |
| Blunt rule | DFSN, DFSR, and SMB are different layers. Do not blame DFSR for an SMB performance issue until the actual target path is proven |
| Support rule | Always prove the client referral target first, then troubleshoot that real SMB server |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain | `corp.local` | `<domain-fqdn>` |
| Namespace root | `\\corp.local\Shares` | `<namespace-root>` |
| DFS folder path | `\\corp.local\Shares\Departments` | `<dfs-folder-path>` |
| Expected target 1 | `\\FS1\Departments` | `<target-1>` |
| Expected target 2 | `\\FS2\Departments` | `<target-2>` |
| Namespace server 1 | `FS1` | `<namespace-server-1>` |
| Namespace server 2 | `FS2` | `<namespace-server-2>` |
| Replication group | `RG_Departments` | `<replication-group>` |
| Replicated folder | `Departments` | `<replicated-folder>` |
| Source DFSR member | `FS1` | `<source-member>` |
| Destination DFSR member | `FS2` | `<destination-member>` |
| User reporting issue | `corp\jsmith` | `<username>` |
| Affected client | `WIN11-01` | `<client-name>` |
| Client site | `HQ` | `<client-site>` |
| Client IP | `10.10.20.25` | `<client-ip>` |
| Expected target site | `HQ` | `<expected-target-site>` |
| Reported symptom | `Slow open`, `Access denied`, `Wrong data`, `Missing file` | `<reported-symptom>` |
| Test file | `dfs-test.txt` | `<test-file>` |
| SMB target under test | `FS1` | `<smb-target>` |
| Performance sample duration | `60 seconds` | `<sample-duration>` |
| Event window | Last 8 hours | `<event-window>` |
| Report path | `C:\DFS-SMB-Troubleshooting\Reports` | `<report-path>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Symptom_Map
| Symptom | Most Likely Layer | First Check | Common Fix |
|---|---|---|---|
| DFS path opens wrong server | DFS referral, AD site, target priority, stale cache | `dfsutil /pktinfo`; `Get-DfsnFolderTarget` | Flush referral cache, fix target priority, fix AD Sites |
| DFS path fails but direct UNC works | Namespace root, referral, namespace server, DFSN config | `Get-DfsnRoot`; `dfsdiag /testreferral` | Repair namespace target or DFSN service |
| Direct UNC fails too | SMB, DNS, firewall, permissions, server health | `Test-NetConnection <server> -Port 445` | Fix SMB path before DFSN |
| Missing file on one target | DFSR backlog, conflict, staging, replicated folder filter | `Get-DfsrBacklog`; `Get-DfsrState` | Resolve DFSR health issue |
| File differs between targets | DFSR backlog, conflict, last-writer behavior | Hash file on both targets | Resolve backlog and conflict state |
| DFSR backlog grows | Replication stopped, schedule closed, bandwidth, database, staging, RPC/firewall | `Get-DfsrBacklog`; DFSR event log | Fix service, schedule, ports, staging, or database |
| DFSR does not replicate | DFSR service stopped, AD config stale, disabled membership | `Get-Service DFSR`; `Update-DfsrConfigurationFromAD` | Start service and poll AD |
| Client slow only through DFS path | Bad referral target or remote site target | `dfsutil /pktinfo` | Fix referral order, AD site, or target state |
| Client slow through direct target too | SMB performance, disk, network, signing, encryption, AV | `Get-SmbConnection`; counters | Troubleshoot SMB target |
| One client slow | Client cache, NIC, Wi-Fi, VPN, old mapping, DNS | `Get-SmbConnection`; `ipconfig /all` | Clear caches and validate network |
| All clients slow on one target | File server CPU, disk, network, SMB config, AV | SMB counters and server counters | Fix bottleneck on that target |
| Access denied after referral | Target SMB or NTFS ACL mismatch | `Get-SmbShareAccess`; `icacls` | Align permissions across targets |
| Users see old data | Referral points to stale target or DFSR backlog | `dfsutil /pktinfo`; `Get-DfsrBacklog` | Disable stale target or fix replication |
| Open files block change | SMB file handles active | `Get-SmbOpenFile` | Close exact handle only after approval |
| DFS namespace visible but empty | ABE, DFSN ACL, target ACL, referral failure | `Get-DfsnAccess`; `Get-DfsnFolderTarget` | Fix DFSN ACL or target access |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture exact user path | Client | `Write down exact path, error, time, user, and client` | Symptom is documented |
| 2 | Confirm client identity | Client | `hostname; whoami` | Correct user and client are known |
| 3 | Confirm client IP and DNS | Client | `ipconfig /all` | DNS and subnet are visible |
| 4 | Confirm AD site mapping | Client | `nltest /dsgetsite` | Client site is known |
| 5 | Confirm domain controller discovery | Client | `nltest /dsgetdc:<domain-fqdn>` | Client can locate domain controller |
| 6 | Test DFS path | Client | `Test-Path "\\corp.local\Shares\Departments"` | DFS path result is known |
| 7 | Inspect DFS referral cache | Client | `dfsutil /pktinfo` | Current DFS referral target is visible |
| 8 | Inspect DFS domain cache | Client | `dfsutil /spcinfo` | DFS namespace cache is visible |
| 9 | Flush DFS referral cache if needed | Client | `dfsutil /pktflush` | Client referral cache is cleared |
| 10 | Flush DFS domain cache if needed | Client | `dfsutil /spcflush` | DFS domain cache is cleared |
| 11 | Retest DFS path after cache flush | Client | `Test-Path "\\corp.local\Shares\Departments"` | Current referral behavior is validated |
| 12 | Test expected target directly | Client | `Test-Path "\\FS1\Departments"` | Direct SMB access to target is known |
| 13 | Test alternate target directly | Client | `Test-Path "\\FS2\Departments"` | Alternate SMB target access is known |
| 14 | Test SMB port to selected target | Client | `Test-NetConnection FS1 -Port 445` | TCP 445 succeeds or fails clearly |
| 15 | Inspect SMB connection | Client | `Get-SmbConnection` | Client SMB target, share, dialect, signing, and encryption are visible |
| 16 | Inspect SMB multichannel | Client | `Get-SmbMultichannelConnection` | Multichannel paths are visible if used |
| 17 | Confirm DFSN roots | Management Host | `Get-DfsnRoot` | Namespace roots are visible |
| 18 | Confirm DFSN root targets | Management Host | `Get-DfsnRootTarget -Path "\\corp.local\Shares"` | Namespace servers are visible |
| 19 | Confirm DFSN folder | Management Host | `Get-DfsnFolder -Path "\\corp.local\Shares\Departments"` | DFS folder settings are visible |
| 20 | Confirm DFSN folder targets | Management Host | `Get-DfsnFolderTarget -Path "\\corp.local\Shares\Departments"` | Real target list and states are visible |
| 21 | Confirm DFSN access | Management Host | `Get-DfsnAccess -Path "\\corp.local\Shares\Departments"` | DFS namespace ACL is visible |
| 22 | Run DFS referral diagnostic | Management Host | `dfsdiag /testreferral /dfspath:\\corp.local\Shares\Departments /full` | Referral diagnostic results are visible |
| 23 | Run DFS configuration diagnostic | Management Host | `dfsdiag /testdfsconfig /dfsroot:\\corp.local\Shares` | DFS namespace configuration is checked |
| 24 | Confirm DFSR service state | DFSR Members | `Get-Service DFSR` | DFSR service is running |
| 25 | Confirm DFSR replication groups | Management Host | `Get-DfsReplicationGroup` | Replication groups are visible |
| 26 | Confirm DFSR members | Management Host | `Get-DfsrMember -GroupName "RG_Departments"` | Members are visible |
| 27 | Confirm DFSR replicated folder | Management Host | `Get-DfsReplicatedFolder -GroupName "RG_Departments"` | Replicated folder is visible |
| 28 | Confirm DFSR memberships | Management Host | `Get-DfsrMembership -GroupName "RG_Departments"` | Local paths and enabled states are visible |
| 29 | Confirm DFSR connections | Management Host | `Get-DfsrConnection -GroupName "RG_Departments"` | Partner connections are visible |
| 30 | Check DFSR backlog FS1 to FS2 | Management Host | `Get-DfsrBacklog -GroupName "RG_Departments" -FolderName "Departments" -SourceComputerName FS1 -DestinationComputerName FS2` | Pending outbound items are visible |
| 31 | Check DFSR backlog FS2 to FS1 | Management Host | `Get-DfsrBacklog -GroupName "RG_Departments" -FolderName "Departments" -SourceComputerName FS2 -DestinationComputerName FS1` | Reverse backlog is visible |
| 32 | Check DFSR current state | DFSR Members | `Get-DfsrState` | Active DFSR work items are visible |
| 33 | Force DFSR AD config refresh | DFSR Members | `Update-DfsrConfigurationFromAD` | DFSR polls AD for latest configuration |
| 34 | Run DFSR health report | Management Host | `Write-DfsrHealthReport -GroupName "RG_Departments" -Path "C:\DFS-SMB-Troubleshooting\Reports"` | HTML health report is created |
| 35 | Run propagation test | Management Host | `Start-DfsrPropagationTest -GroupName "RG_Departments" -FolderName "Departments" -ReferenceComputerName FS1` | Test file is created |
| 36 | Generate propagation report | Management Host | `Write-DfsrPropagationReport -GroupName "RG_Departments" -FolderName "Departments" -Path "C:\DFS-SMB-Troubleshooting\Reports"` | Propagation timing is reported |
| 37 | Check SMB sessions on target | SMB Target | `Get-SmbSession` | Active users are visible |
| 38 | Check open files on target | SMB Target | `Get-SmbOpenFile` | Active file handles are visible |
| 39 | Check SMB server config | SMB Target | `Get-SmbServerConfiguration` | Signing, encryption, leasing, and SMB protocol state are visible |
| 40 | Check SMB server NICs | SMB Target | `Get-SmbServerNetworkInterface` | Server SMB NICs are visible |
| 41 | Check SMB client NICs | Client | `Get-SmbClientNetworkInterface` | Client SMB NICs are visible |
| 42 | Sample SMB counters | SMB Target | `Get-Counter "\SMB Server Shares(*)\*" -SampleInterval 5 -MaxSamples 12` | SMB share counters are captured |
| 43 | Sample disk counters | SMB Target | `Get-Counter "\LogicalDisk(*)\Avg. Disk sec/Read","\LogicalDisk(*)\Avg. Disk sec/Write" -SampleInterval 5 -MaxSamples 12` | Disk latency is captured |
| 44 | Sample network counters | SMB Target | `Get-Counter "\Network Interface(*)\Bytes Total/sec" -SampleInterval 5 -MaxSamples 12` | Network throughput is captured |
| 45 | Collect DFSR events | DFSR Members | `Get-WinEvent -LogName "DFS Replication" -MaxEvents 200` | DFSR events are visible |
| 46 | Collect DFSN events | Namespace Servers | `Get-WinEvent -ListLog "*DFS*"` | DFS-related logs are discoverable |
| 47 | Collect SMB server events | SMB Target | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 200` | SMB server events are visible |
| 48 | Collect SMB client events | Client | `Get-WinEvent -LogName "Microsoft-Windows-SMBClient/Connectivity" -MaxEvents 200` | SMB client events are visible |
| 49 | Isolate root cause layer | Operator | `Classify issue as DFSN, DFSR, SMB, DNS, AD site, permissions, or performance` | Troubleshooting path is narrowed |
| 50 | Document remediation and validation | Operator | `Record commands, evidence, fix, and retest result` | Ticket has proof, not guesses |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSN_Referral_Skeleton
```powershell
# Run on affected client.
# Purpose: prove which DFS target the client is using and whether cache is stale.

$DfsPath = "\\corp.local\Shares\Departments"
$ExpectedTargets = @(
  "\\FS1\Departments",
  "\\FS2\Departments"
)

Write-Host "Client identity"
hostname
whoami

Write-Host "`nClient IP and DNS"
ipconfig /all

Write-Host "`nClient AD site"
nltest /dsgetsite

Write-Host "`nDomain controller discovery"
nltest /dsgetdc:corp.local

Write-Host "`nCurrent DFS referral cache"
dfsutil /pktinfo

Write-Host "`nCurrent DFS domain cache"
dfsutil /spcinfo

Write-Host "`nTesting DFS path"
Test-Path $DfsPath

Write-Host "`nTesting expected direct targets"
foreach ($Target in $ExpectedTargets) {
  Write-Host "Testing $Target"
  Test-Path $Target
}

Write-Host "`nFlush DFS referral cache if referral appears wrong"
# dfsutil /pktflush
# dfsutil /spcflush

Write-Host "`nAfter cache flush, retest DFS path"
# Test-Path $DfsPath
# dfsutil /pktinfo
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSN_Topology_Skeleton
```powershell
# Run on management host with DFSN tools.
# Purpose: inventory namespace root, folder targets, target state, and DFSN ACL.

$NamespaceRoot = "\\corp.local\Shares"
$DfsFolderPath = "\\corp.local\Shares\Departments"
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "DFS namespace roots"
Get-DfsnRoot |
  Export-Csv "$ReportRoot\dfsn-roots-$Timestamp.csv" -NoTypeInformation

Write-Host "DFS namespace root targets"
Get-DfsnRootTarget -Path $NamespaceRoot |
  Export-Csv "$ReportRoot\dfsn-root-targets-$Timestamp.csv" -NoTypeInformation

Write-Host "DFS folder settings"
Get-DfsnFolder -Path $DfsFolderPath |
  Export-Clixml "$ReportRoot\dfsn-folder-$Timestamp.xml"

Write-Host "DFS folder targets"
Get-DfsnFolderTarget -Path $DfsFolderPath |
  Export-Csv "$ReportRoot\dfsn-folder-targets-$Timestamp.csv" -NoTypeInformation

Write-Host "DFS folder access"
Get-DfsnAccess -Path $DfsFolderPath |
  Export-Csv "$ReportRoot\dfsn-access-$Timestamp.csv" -NoTypeInformation

Write-Host "DFS referral diagnostic"
dfsdiag /testreferral /dfspath:$DfsFolderPath /full |
  Out-File "$ReportRoot\dfsdiag-testreferral-$Timestamp.txt"

Write-Host "DFS configuration diagnostic"
dfsdiag /testdfsconfig /dfsroot:$NamespaceRoot |
  Out-File "$ReportRoot\dfsdiag-testdfsconfig-$Timestamp.txt"

Write-Host "DFS topology reports exported to $ReportRoot"
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSR_Backlog_Skeleton
```powershell
# Run on management host with DFSR tools.
# Purpose: measure DFSR backlog in both directions.

$GroupName = "RG_Departments"
$FolderName = "Departments"
$MemberA = "FS1"
$MemberB = "FS2"
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Backlog from $MemberA to $MemberB"
$BacklogAB = Get-DfsrBacklog `
  -GroupName $GroupName `
  -FolderName $FolderName `
  -SourceComputerName $MemberA `
  -DestinationComputerName $MemberB

$BacklogAB |
  Export-Csv "$ReportRoot\dfsr-backlog-$MemberA-to-$MemberB-$Timestamp.csv" -NoTypeInformation

Write-Host "Backlog count $MemberA to $MemberB: $($BacklogAB.Count)"

Write-Host "Backlog from $MemberB to $MemberA"
$BacklogBA = Get-DfsrBacklog `
  -GroupName $GroupName `
  -FolderName $FolderName `
  -SourceComputerName $MemberB `
  -DestinationComputerName $MemberA

$BacklogBA |
  Export-Csv "$ReportRoot\dfsr-backlog-$MemberB-to-$MemberA-$Timestamp.csv" -NoTypeInformation

Write-Host "Backlog count $MemberB to $MemberA: $($BacklogBA.Count)"
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSR_State_Skeleton
```powershell
# Run on DFSR members.
# Purpose: inspect DFSR service state, AD polling, membership, connection, and active replication state.

$GroupName = "RG_Departments"
$Servers = @("FS1", "FS2")
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Replication group"
Get-DfsReplicationGroup -GroupName $GroupName |
  Export-Clixml "$ReportRoot\dfsr-group-$Timestamp.xml"

Write-Host "Replicated folders"
Get-DfsReplicatedFolder -GroupName $GroupName |
  Export-Csv "$ReportRoot\dfsr-replicated-folders-$Timestamp.csv" -NoTypeInformation

Write-Host "Members"
Get-DfsrMember -GroupName $GroupName |
  Export-Csv "$ReportRoot\dfsr-members-$Timestamp.csv" -NoTypeInformation

Write-Host "Memberships"
Get-DfsrMembership -GroupName $GroupName |
  Export-Csv "$ReportRoot\dfsr-memberships-$Timestamp.csv" -NoTypeInformation

Write-Host "Connections"
Get-DfsrConnection -GroupName $GroupName |
  Export-Csv "$ReportRoot\dfsr-connections-$Timestamp.csv" -NoTypeInformation

foreach ($Server in $Servers) {
  Write-Host "===== $Server DFSR state ====="

  Invoke-Command -ComputerName $Server -ArgumentList $ReportRoot, $Timestamp -ScriptBlock {
    param($ReportRoot, $Timestamp)

    Get-Service DFSR |
      Select-Object Name, DisplayName, Status, StartType |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-dfsr-service-$Timestamp.csv" -NoTypeInformation

    Update-DfsrConfigurationFromAD

    Get-DfsrState |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-dfsr-state-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -LogName "DFS Replication" -MaxEvents 300 -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-dfsr-events-$Timestamp.csv" -NoTypeInformation
  }
}
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_DFSR_Propagation_Test_Skeleton
```powershell
# Run on management host with DFSR tools.
# Purpose: create a DFSR propagation test and generate timing report.

$GroupName = "RG_Departments"
$FolderName = "Departments"
$ReferenceComputerName = "FS1"
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$PropagationPath = Join-Path $ReportRoot "DFSR-Propagation-$Timestamp"

New-Item -ItemType Directory -Force -Path $PropagationPath | Out-Null

Write-Host "Starting DFSR propagation test"
Start-DfsrPropagationTest `
  -GroupName $GroupName `
  -FolderName $FolderName `
  -ReferenceComputerName $ReferenceComputerName

Write-Host "Wait for propagation to occur, then generate report"
Write-DfsrPropagationReport `
  -GroupName $GroupName `
  -FolderName $FolderName `
  -Path $PropagationPath

Write-Host "Generating DFSR health report"
Write-DfsrHealthReport `
  -GroupName $GroupName `
  -Path $PropagationPath

Get-ChildItem $PropagationPath -Recurse
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_SMB_Client_Baseline_Skeleton
```powershell
# Run on affected client.
# Purpose: prove which SMB server the DFS path resolved to and collect client SMB state.

$DfsPath = "\\corp.local\Shares\Departments"
$TargetServer = "FS1"
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Client identity"
hostname
whoami

Write-Host "`nTest DFS path"
Test-Path $DfsPath

Write-Host "`nDFS referral cache"
dfsutil /pktinfo |
  Out-File "$ReportRoot\client-dfs-pktinfo-$Timestamp.txt"

Write-Host "`nSMB connections"
Get-SmbConnection |
  Export-Csv "$ReportRoot\client-smb-connections-$Timestamp.csv" -NoTypeInformation

Write-Host "`nSMB multichannel connections"
Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
  Export-Csv "$ReportRoot\client-smb-multichannel-$Timestamp.csv" -NoTypeInformation

Write-Host "`nSMB mappings"
Get-SmbMapping -ErrorAction SilentlyContinue |
  Export-Csv "$ReportRoot\client-smb-mappings-$Timestamp.csv" -NoTypeInformation

Write-Host "`nSMB client configuration"
Get-SmbClientConfiguration |
  Export-Clixml "$ReportRoot\client-smb-config-$Timestamp.xml"

Write-Host "`nSMB client network interfaces"
Get-SmbClientNetworkInterface |
  Export-Csv "$ReportRoot\client-smb-nics-$Timestamp.csv" -NoTypeInformation

Write-Host "`nTesting SMB port"
Test-NetConnection $TargetServer -Port 445 |
  Out-File "$ReportRoot\client-testnetconnection-$TargetServer-445-$Timestamp.txt"
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_SMB_Server_Baseline_Skeleton
```powershell
# Run on the selected SMB target server.
# Purpose: collect SMB server sessions, open files, share config, and server config.

$ShareName = "Departments"
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "SMB server configuration"
Get-SmbServerConfiguration |
  Export-Clixml "$ReportRoot\$env:COMPUTERNAME-smb-server-config-$Timestamp.xml"

Write-Host "SMB server network interfaces"
Get-SmbServerNetworkInterface |
  Export-Csv "$ReportRoot\$env:COMPUTERNAME-smb-server-nics-$Timestamp.csv" -NoTypeInformation

Write-Host "Target share"
Get-SmbShare -Name $ShareName |
  Export-Clixml "$ReportRoot\$env:COMPUTERNAME-smb-share-$ShareName-$Timestamp.xml"

Write-Host "Target share access"
Get-SmbShareAccess -Name $ShareName |
  Export-Csv "$ReportRoot\$env:COMPUTERNAME-smb-share-access-$ShareName-$Timestamp.csv" -NoTypeInformation

Write-Host "SMB sessions"
Get-SmbSession |
  Export-Csv "$ReportRoot\$env:COMPUTERNAME-smb-sessions-$Timestamp.csv" -NoTypeInformation

Write-Host "SMB open files"
Get-SmbOpenFile |
  Export-Csv "$ReportRoot\$env:COMPUTERNAME-smb-open-files-$Timestamp.csv" -NoTypeInformation

Write-Host "Recent SMB server events"
Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 300 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated, Id, LevelDisplayName, Message |
  Export-Csv "$ReportRoot\$env:COMPUTERNAME-smbserver-events-$Timestamp.csv" -NoTypeInformation
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_SMB_Performance_Counter_Skeleton
```powershell
# Run on selected SMB target server during the issue window.
# Purpose: capture SMB, disk, CPU, memory, and network counters.

$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputFile = "$ReportRoot\$env:COMPUTERNAME-smb-performance-$Timestamp.xml"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

$Counters = @(
  "\SMB Server Shares(*)\*",
  "\SMB Server Sessions(*)\*",
  "\LogicalDisk(*)\Avg. Disk sec/Read",
  "\LogicalDisk(*)\Avg. Disk sec/Write",
  "\LogicalDisk(*)\Disk Reads/sec",
  "\LogicalDisk(*)\Disk Writes/sec",
  "\Network Interface(*)\Bytes Total/sec",
  "\Processor(_Total)\% Processor Time",
  "\Memory\Available MBytes"
)

Write-Host "Sampling performance counters"
Get-Counter `
  -Counter $Counters `
  -SampleInterval 5 `
  -MaxSamples 12 |
  Export-Clixml $OutputFile

Write-Host "Performance capture saved to $OutputFile"
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Network_Latency_Skeleton
```powershell
# Run from client and between DFSR members.
# Purpose: prove DNS, route, latency, and SMB transport health.

$Targets = @("FS1", "FS2")
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

foreach ($Target in $Targets) {
  Write-Host "===== $Target ====="

  Resolve-DnsName $Target |
    Out-File "$ReportRoot\resolve-$Target-$Timestamp.txt"

  Test-Connection $Target -Count 10 |
    Out-File "$ReportRoot\ping-$Target-$Timestamp.txt"

  Test-NetConnection $Target -Port 445 |
    Out-File "$ReportRoot\testnetconnection-$Target-445-$Timestamp.txt"

  tracert $Target |
    Out-File "$ReportRoot\tracert-$Target-$Timestamp.txt"
}

Write-Host "Network baseline complete."
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Event_Log_Collection_Skeleton
```powershell
# Run from management host.
# Purpose: collect DFSN, DFSR, SMBClient, SMBServer, and System evidence.

$Servers = @("FS1", "FS2")
$Client = "WIN11-01"
$ReportRoot = "C:\DFS-SMB-Troubleshooting\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$StartTime = (Get-Date).AddHours(-8)

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

foreach ($Server in $Servers) {
  Invoke-Command -ComputerName $Server -ArgumentList $ReportRoot, $Timestamp, $StartTime -ScriptBlock {
    param($ReportRoot, $Timestamp, $StartTime)

    Get-WinEvent -ListLog "*DFS*" -ErrorAction SilentlyContinue |
      Select-Object LogName, IsEnabled, RecordCount |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-dfs-log-inventory-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -FilterHashtable @{
      LogName = "DFS Replication"
      StartTime = $StartTime
    } -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-dfsr-events-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -FilterHashtable @{
      LogName = "Microsoft-Windows-SMBServer/Operational"
      StartTime = $StartTime
    } -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-smbserver-events-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -FilterHashtable @{
      LogName = "System"
      StartTime = $StartTime
    } -ErrorAction SilentlyContinue |
      Where-Object {
        $_.ProviderName -like "*DFSR*" -or
        $_.ProviderName -like "*DFS*" -or
        $_.ProviderName -like "*LanmanServer*"
      } |
      Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-system-file-events-$Timestamp.csv" -NoTypeInformation
  }
}

Invoke-Command -ComputerName $Client -ArgumentList $ReportRoot, $Timestamp, $StartTime -ScriptBlock {
  param($ReportRoot, $Timestamp, $StartTime)

  Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-SMBClient/Connectivity"
    StartTime = $StartTime
  } -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, Message |
    Export-Csv "$ReportRoot\$env:COMPUTERNAME-smbclient-connectivity-$Timestamp.csv" -NoTypeInformation

  Get-WinEvent -FilterHashtable @{
    LogName = "Microsoft-Windows-SMBClient/Operational"
    StartTime = $StartTime
  } -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, Message |
    Export-Csv "$ReportRoot\$env:COMPUTERNAME-smbclient-operational-$Timestamp.csv" -NoTypeInformation
}

Write-Host "Event log collection exported to $ReportRoot"
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Safe_Remediation_Skeleton
```powershell
# Use only after the failing layer has been proven.
# Purpose: safe remediation examples for common DFSN, DFSR, and SMB cases.

$DfsPath = "\\corp.local\Shares\Departments"
$BadTarget = "\\FS2\Departments"
$GoodTarget = "\\FS1\Departments"
$GroupName = "RG_Departments"
$FolderName = "Departments"

Write-Host "Option 1: Temporarily disable a bad DFS folder target"
# Set-DfsnFolderTarget -Path $DfsPath -TargetPath $BadTarget -State Offline

Write-Host "Option 2: Re-enable a healthy DFS folder target"
# Set-DfsnFolderTarget -Path $DfsPath -TargetPath $GoodTarget -State Online

Write-Host "Option 3: Force DFSR service to poll AD"
# Invoke-Command -ComputerName FS1,FS2 -ScriptBlock { Update-DfsrConfigurationFromAD }

Write-Host "Option 4: Sync DFSR group outside schedule after approval"
# Sync-DfsReplicationGroup -GroupName $GroupName -SourceComputerName FS1 -DestinationComputerName FS2 -DurationInMinutes 15

Write-Host "Option 5: Suspend DFSR only during approved emergency containment"
# Suspend-DfsReplicationGroup -GroupName $GroupName -DurationInMinutes 30

Write-Host "Option 6: Close exact SMB open file only after user approval"
# Close-SmbOpenFile -FileId <file-id> -Force

Write-Host "Option 7: Clear client DFS cache after target correction"
# dfsutil /pktflush
# dfsutil /spcflush

Write-Host "Do not use these as guesses. Use them only after evidence points to the specific layer."
```

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify client site | `nltest /dsgetsite` | Client site is expected |
| Verify DFS path | `Test-Path "\\corp.local\Shares\Departments"` | DFS path works |
| Verify referral cache | `dfsutil /pktinfo` | Client target referral is visible |
| Verify DFS root | `Get-DfsnRoot` | Namespace root is visible |
| Verify root targets | `Get-DfsnRootTarget -Path "\\corp.local\Shares"` | Namespace servers are visible |
| Verify folder targets | `Get-DfsnFolderTarget -Path "\\corp.local\Shares\Departments"` | Targets and states are visible |
| Verify DFSN ACL | `Get-DfsnAccess -Path "\\corp.local\Shares\Departments"` | Namespace ACL is visible |
| Verify direct target FS1 | `Test-Path "\\FS1\Departments"` | Direct target works |
| Verify direct target FS2 | `Test-Path "\\FS2\Departments"` | Alternate target works |
| Verify SMB port | `Test-NetConnection FS1 -Port 445` | TCP 445 succeeds |
| Verify SMB connection | `Get-SmbConnection` | Client target and dialect are visible |
| Verify SMB multichannel | `Get-SmbMultichannelConnection` | Multichannel paths are visible if applicable |
| Verify SMB server sessions | `Get-SmbSession` | Active sessions are visible |
| Verify SMB open files | `Get-SmbOpenFile` | Open handles are visible |
| Verify SMB server config | `Get-SmbServerConfiguration` | Server SMB settings are visible |
| Verify DFSR service | `Get-Service DFSR` | DFSR is running |
| Verify DFSR group | `Get-DfsReplicationGroup -GroupName "RG_Departments"` | Replication group exists |
| Verify DFSR membership | `Get-DfsrMembership -GroupName "RG_Departments"` | Members and local paths are visible |
| Verify DFSR connection | `Get-DfsrConnection -GroupName "RG_Departments"` | Connections are visible |
| Verify DFSR backlog | `Get-DfsrBacklog -GroupName "RG_Departments" -FolderName "Departments" -SourceComputerName FS1 -DestinationComputerName FS2` | Backlog is visible |
| Verify DFSR state | `Get-DfsrState` | Current replication work is visible |
| Verify DFSR health report | `Write-DfsrHealthReport -GroupName "RG_Departments" -Path "C:\DFS-SMB-Troubleshooting\Reports"` | HTML health report is generated |
| Verify SMB performance | `Get-Counter "\SMB Server Shares(*)\*" -SampleInterval 5 -MaxSamples 3` | SMB counters return |
| Verify DFSR events | `Get-WinEvent -LogName "DFS Replication" -MaxEvents 50` | DFSR events return |
| Verify SMB events | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 50` | SMB events return |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| DFS referral cache flushed | No rollback needed | Client rebuilds referral cache |
| DFS domain cache flushed | No rollback needed | Client rebuilds DFS domain cache |
| DFS target disabled | `Set-DfsnFolderTarget -Path "\\corp.local\Shares\Departments" -TargetPath "\\FS2\Departments" -State Online` | Target is re-enabled |
| DFS target enabled | `Set-DfsnFolderTarget -Path "\\corp.local\Shares\Departments" -TargetPath "\\FS2\Departments" -State Offline` | Target is disabled again |
| DFS target priority changed | Restore previous `Set-DfsnFolderTarget` priority settings | Referral behavior returns to baseline |
| DFSR suspended | Wait for duration or resume with approved sync operation | Replication resumes |
| DFSR forced sync | No direct rollback | Replication catches up according to topology |
| DFSR service restarted | No direct rollback | Service returns to normal running state |
| SMB open file closed | No clean rollback | User must reopen file or application |
| SMB session closed | No clean rollback | Client reconnects on next access |
| SMB tuning changed | Restore captured `Get-SmbServerConfiguration` or `Get-SmbClientConfiguration` baseline | SMB settings return to baseline |
| Firewall rule enabled | Disable only the rule changed in the ticket | Firewall returns to baseline |
| Reports created | `Remove-Item "C:\DFS-SMB-Troubleshooting" -Recurse -Force` | Report folder is removed |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Failure_Checks
| Failure | Likely Cause | Proof Command | Fix |
|---|---|---|---|
| DFS path fails but direct UNC works | DFSN issue | `dfsdiag /testreferral`; `Get-DfsnFolderTarget` | Fix namespace target, root, or referral state |
| Direct UNC fails | SMB, DNS, firewall, ACL, server down | `Test-NetConnection -Port 445`; `Get-SmbShareAccess`; `icacls` | Fix SMB layer first |
| Client gets remote target | AD site mapping or referral ordering issue | `nltest /dsgetsite`; `dfsutil /pktinfo` | Fix AD subnet/site or target priority |
| Client keeps old target | Referral cache stale | `dfsutil /pktinfo` | `dfsutil /pktflush` |
| Namespace root missing | DFSN service, AD metadata, or wrong path | `Get-DfsnRoot`; `dfsdiag /testdfsconfig` | Repair namespace or root target |
| Folder target offline | Target disabled or server unavailable | `Get-DfsnFolderTarget` | Bring target online only if healthy |
| ABE hides DFS folder | DFSN ACL or target ACL denies access | `Get-DfsnAccess`; `icacls` | Fix namespace or NTFS permissions |
| Missing file on one target | DFSR backlog or conflict | `Get-DfsrBacklog`; DFSR events | Fix DFSR health and wait for convergence |
| File differs on two targets | DFSR conflict, lag, or last-writer issue | File hash on both targets | Resolve conflict and educate users |
| DFSR backlog high | Replication disabled, schedule closed, service stopped, network blocked | `Get-DfsrBacklog`; `Get-DfsrConnectionSchedule` | Fix service, schedule, network, or membership |
| DFSR service stopped | Service crash or disabled startup | `Get-Service DFSR` | Start service and inspect events |
| DFSR AD config stale | Member did not poll AD | `Update-DfsrConfigurationFromAD` | Force poll and verify membership |
| DFSR RPC blocked | Firewall or network issue | Test RPC path and events | Open required ports or static DFSR port |
| DFSR staging full | Staging quota too small or high churn | DFSR events and membership settings | Increase staging quota and clean backlog |
| DFSR conflict and deleted data | Last writer wins or preserved files | `Get-DfsrPreservedFiles` | Restore preserved files if appropriate |
| SMB slow on one target | Disk, CPU, NIC, antivirus, signing, encryption, open handles | SMB counters, disk counters, server config | Fix bottleneck on target |
| SMB slow on all targets | Client network, VPN, Wi-Fi, DNS, WAN, endpoint security | Client baseline and network tests | Fix client or network path |
| SMB multichannel not active | NIC capability, RSS/RDMA disabled, policy constraint | `Get-SmbMultichannelConnection`; `Get-SmbClientNetworkInterface` | Fix NIC, RSS, RDMA, or constraints |
| SMB signing CPU issue | Signing required and CPU constrained | `Get-SmbConnection`; CPU counters | Keep required security, add CPU, or tune design |
| SMB encryption overhead | Encryption enabled on high-throughput share | `Get-SmbShare`; CPU counters | Keep if required, otherwise review share policy |
| Open files cause app hangs | Locked SMB file handles | `Get-SmbOpenFile` | Close exact handle only after approval |
| Users see old data after failover | DFS referral target points to stale server | `dfsutil /pktinfo`; target file timestamp | Disable stale target or fix DFSR |
| Events missing | Wrong log name or no activity | `Get-WinEvent -ListLog "*DFS*","*SMB*"` | Query discovered logs |
| Reports show no issue | Intermittent issue not captured during window | Repeat capture during user impact | Correlate event time with user report |

# Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance_Related_Labs
| Lab                                                                 | Relationship                                                              |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `02_Create_And_Publish_SMB_Shares.md`                               | DFS targets point to real SMB shares                                      |
| `03_Configure_NTFS_And_Share_Permissions.md`                        | Target ACL mismatches cause DFS access inconsistencies                    |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`      | SMB security settings can affect performance and connectivity             |
| `13_Configure_DFS_Namespaces.md`                                    | Base namespace creation and target structure                              |
| `14_Configure_DFS_Replication.md`                                   | Base DFSR replication group and member configuration                      |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`      | SMB monitoring provides the performance and open file layer               |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`             | Direct SMB troubleshooting before blaming DFS                             |
| `19_Configure_BranchCache_For_File_Services.md`                     | Branch cache can affect perceived WAN file performance                    |
| `22_Configure_Azure_File_Sync_For_Windows_File_Server.md`           | Azure File Sync is a different sync model than DFSR                       |
| `23_Configure_Storage_Replica_For_File_Server_Disaster_Recovery.md` | Storage Replica is block-level DR, not namespace or DFSR troubleshooting  |
| `25_Migrate_File_Server_Data_Shares_And_Permissions.md`             | Migration cutovers commonly expose DFSN, DFSR, and SMB performance issues |