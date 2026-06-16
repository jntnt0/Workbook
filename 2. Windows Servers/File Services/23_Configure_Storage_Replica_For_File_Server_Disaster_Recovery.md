23_Configure_Storage_Replica_For_File_Server_Disaster_Recovery.md
# Configure_Storage_Replica_For_File_Server_Disaster_Recovery

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Index
23_Configure_Storage_Replica_For_File_Server_Disaster_Recovery.md
Configure_Storage_Replica_For_File_Server_Disaster_Recovery
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Source_Basis
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Mental_Model
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Planning_Table
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Configuration_Checklist
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Prereq_Validation_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Feature_Install_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Volume_And_Log_Validation_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Network_And_Firewall_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Topology_Test_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Partnership_Create_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Replication_Monitoring_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Planned_Failover_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Test_Failover_Snapshot_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_File_Server_Cutover_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Status_And_Event_Review_Skeleton
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Verification_Commands
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Rollback
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Failure_Checks
Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Related_Labs

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server Storage Replica overview | Synchronous and asynchronous replication | DR mode selection and RPO behavior |
| Windows Server Storage Replica overview | Supported configurations | Server-to-server, cluster-to-cluster, stretch cluster options |
| Windows Server Storage Replica overview | Prerequisites | AD DS, storage, network, RAM, CPU, edition limits |
| Server-to-server Storage Replica guide | Prerequisites | Two servers, data and log volumes, firewall, latency, storage layout |
| Server-to-server Storage Replica guide | Test-SRTopology | Preflight validation and sizing report |
| Server-to-server Storage Replica guide | New-SRPartnership | Creating the replication partnership |
| Server-to-server Storage Replica guide | Get-SRGroup and Get-SRPartnership | Monitoring replication state |
| Server-to-server Storage Replica guide | Set-SRPartnership | Reversing replication direction for failover |
| StorageReplica PowerShell module | Cmdlet reference | Storage Replica operational commands |
| Operational file server practice | DR cutover | SMB share recreation, DNS aliasing, DFS namespace update, and user access validation |

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Storage Replica | Windows Server block-level volume replication for disaster recovery |
| Source | Server and volume currently allowing writes and replicating outbound |
| Destination | Server and volume receiving replicated blocks and normally blocked from local writes |
| Replication partnership | Relationship between source replication group and destination replication group |
| Replication group | Named group of one or more replicated volumes and its log on each server |
| Data volume | Volume containing file server data to protect |
| Log volume | Dedicated volume used by Storage Replica for replication logs |
| Synchronous replication | Source write completes only after remote site acknowledgment, giving zero data loss behavior on low-latency links |
| Asynchronous replication | Source write completes before remote acknowledgment, allowing longer distance but with possible data loss |
| Initial block copy | First replication pass that synchronizes destination volume with source volume |
| Destination dismount | Destination data volume is normally inaccessible while replication is active |
| Planned failover | Controlled direction reversal after replication is healthy and synchronized |
| Unplanned failover | Disaster action when the source site is unavailable |
| Test failover snapshot | Temporary mounted destination snapshot used to validate data without breaking replication |
| SMB 3 transport | Storage Replica uses SMB 3 transport for replication traffic |
| TCP 445 | Primary SMB transport port for Storage Replica |
| TCP 5445 | SMB Direct port when RDMA is used |
| TCP 5985 | WinRM management traffic used by PowerShell remoting |
| Not backup | Replication also replicates deletes, corruption, and ransomware damage |
| Blunt rule | Storage Replica protects against server or site loss. It does not replace backup, snapshots, retention, or tested restore procedures |

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Source file server | `FS1` | `<source-server-name>` |
| Destination file server | `FS2` | `<destination-server-name>` |
| Domain | `corp.local` | `<domain-fqdn>` |
| Source site | `PRIMARY` | `<source-site>` |
| Destination site | `DR` | `<destination-site>` |
| Replication mode | `Synchronous` or `Asynchronous` | `<replication-mode>` |
| Source data volume | `D:` | `<source-data-volume>` |
| Source log volume | `L:` | `<source-log-volume>` |
| Destination data volume | `D:` | `<destination-data-volume>` |
| Destination log volume | `L:` | `<destination-log-volume>` |
| Source replication group | `RG-FS1-DATA` | `<source-rg-name>` |
| Destination replication group | `RG-FS2-DATA` | `<destination-rg-name>` |
| Log size | `8GB` or topology report recommendation | `<log-size>` |
| Source SMB share | `Departments` | `<source-share-name>` |
| Source share path | `D:\Shares\Departments` | `<source-share-path>` |
| Destination share path | `D:\Shares\Departments` | `<destination-share-path>` |
| DFS namespace path | `\\corp.local\Shares\Departments` | `<dfs-namespace-path>` |
| DNS alias | `files.corp.local` | `<dns-alias>` |
| Test client | `WIN11-01` | `<test-client>` |
| Latency target | `5 ms or less for sync` | `<latency-target>` |
| Bandwidth estimate | Must handle write workload | `<bandwidth-plan>` |
| Test-SRTopology duration | `30 minutes` | `<topology-test-duration>` |
| Result path | `C:\SR-Reports` | `<report-path>` |
| Change ticket | `CHG000123` | `<ticket-id>` |
| Backup confirmed | Yes / No | `<backup-confirmed>` |
| Failback plan confirmed | Yes / No | `<failback-confirmed>` |

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm source identity | Source Server | `hostname` | Correct source file server is identified |
| 2 | Confirm destination identity | Destination Server | `hostname` | Correct DR file server is identified |
| 3 | Confirm domain membership | Both Servers | `(Get-WmiObject Win32_ComputerSystem).Domain` | Both servers are in expected AD domain |
| 4 | Confirm OS edition and version | Both Servers | `Get-ComputerInfo | Select WindowsProductName,WindowsVersion,OsName` | Supported Windows Server version and edition are visible |
| 5 | Confirm server resources | Both Servers | `Get-CimInstance Win32_ComputerSystem | Select Name,NumberOfLogicalProcessors,TotalPhysicalMemory` | CPU and RAM meet requirements |
| 6 | Confirm source volumes | Source Server | `Get-Volume` | Data and log volumes are visible |
| 7 | Confirm destination volumes | Destination Server | `Get-Volume` | Matching data and log volumes are visible |
| 8 | Confirm source data path | Source Server | `Test-Path "D:\Shares\Departments"` | Source file data path exists |
| 9 | Confirm destination data volume is clean or staged | Destination Server | `Get-ChildItem D:\ -Force` | Destination volume state is understood before replication |
| 10 | Confirm data volumes are not OS volumes | Both Servers | `$env:SystemDrive` | Replicated data volume is not the Windows system volume |
| 11 | Confirm log volumes are dedicated | Both Servers | `Get-ChildItem L:\ -Force` | Log volume has no unrelated workload |
| 12 | Confirm volume sizes match | Both Servers | `Get-Volume -DriveLetter D,L | Select DriveLetter,Size,SizeRemaining` | Source and destination data volumes match, log volumes match |
| 13 | Confirm disk partition style | Both Servers | `Get-Disk | Select Number,FriendlyName,PartitionStyle,OperationalStatus` | Replication disks use GPT |
| 14 | Confirm sector sizes | Both Servers | `Get-Disk | Select Number,FriendlyName,LogicalSectorSize,PhysicalSectorSize` | Matching sector sizes are confirmed |
| 15 | Confirm source SMB share | Source Server | `Get-SmbShare -Name "Departments"` | Existing file share is visible |
| 16 | Confirm current SMB access | Client | `Test-Path "\\FS1\Departments"` | Users can access source file server before DR setup |
| 17 | Confirm backup exists | Source Server | `wbadmin get versions` | Backup state is known if Windows Server Backup is used |
| 18 | Install File Server role | Both Servers | `Install-WindowsFeature FS-FileServer` | File Server role is installed |
| 19 | Install Storage Replica | Both Servers | `Install-WindowsFeature Storage-Replica -IncludeManagementTools -Restart` | Storage Replica feature and tools install |
| 20 | Confirm Storage Replica cmdlets | Both Servers | `Get-Command -Module StorageReplica` | Storage Replica commands are available |
| 21 | Configure WinRM | Both Servers | `winrm quickconfig` | WinRM is enabled |
| 22 | Test PowerShell remoting | Management Host | `Test-WSMan FS1; Test-WSMan FS2` | Both servers respond |
| 23 | Enable SMB firewall group | Both Servers | `Enable-NetFirewallRule -DisplayGroup "File and Printer Sharing"` | SMB firewall rules are enabled |
| 24 | Enable WinRM firewall group | Both Servers | `Enable-NetFirewallRule -DisplayGroup "Windows Remote Management"` | WinRM firewall rules are enabled |
| 25 | Enable ICMP if policy allows | Both Servers | `Enable-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)"` | ICMP test traffic is allowed |
| 26 | Test SMB port between servers | Source Server | `Test-NetConnection FS2 -Port 445` | Source reaches destination over TCP 445 |
| 27 | Test SMB port reverse direction | Destination Server | `Test-NetConnection FS1 -Port 445` | Destination reaches source over TCP 445 |
| 28 | Test WinRM port between servers | Management Host | `Test-NetConnection FS1 -Port 5985; Test-NetConnection FS2 -Port 5985` | WinRM is reachable |
| 29 | Run topology test | Management Host | `Test-SRTopology -SourceComputerName FS1 -SourceVolumeName D: -SourceLogVolumeName L: -DestinationComputerName FS2 -DestinationVolumeName D: -DestinationLogVolumeName L: -DurationInMinutes 30 -ResultPath C:\SR-Reports` | HTML report is generated |
| 30 | Review topology report | Operator | `Open C:\SR-Reports\TestSrTopologyReport.html` | Latency, bandwidth, log sizing, and requirements are validated |
| 31 | Create Storage Replica partnership | Management Host | `New-SRPartnership -SourceComputerName FS1 -SourceRGName "RG-FS1-DATA" -SourceVolumeName D: -SourceLogVolumeName L: -DestinationComputerName FS2 -DestinationRGName "RG-FS2-DATA" -DestinationVolumeName D: -DestinationLogVolumeName L: -ReplicationMode Synchronous` | Partnership is created |
| 32 | Confirm partnership | Management Host | `Get-SRPartnership` | Partnership is visible |
| 33 | Confirm replication groups | Management Host | `Get-SRGroup` | Source and destination groups are visible |
| 34 | Monitor initial sync | Destination Server | `(Get-SRGroup -Name "RG-FS2-DATA").Replicas | Select ReplicationStatus,NumOfBytesRemaining` | Initial block copy progress is visible |
| 35 | Review Storage Replica events | Both Servers | `Get-WinEvent -ProviderName Microsoft-Windows-StorageReplica -MaxEvents 50` | Storage Replica events are visible |
| 36 | Confirm destination volume behavior | Destination Server | `Get-Volume -DriveLetter D` | Destination data volume is not writable during replication |
| 37 | Test write on source SMB share | Client | `"sr-test" | Out-File "\\FS1\Departments\sr-test.txt"` | Source accepts writes |
| 38 | Confirm source local file exists | Source Server | `Get-Item "D:\Shares\Departments\sr-test.txt"` | File exists on source |
| 39 | Confirm sync catches up | Destination Server | `(Get-SRGroup -Name "RG-FS2-DATA").Replicas | Select ReplicationStatus,NumOfBytesRemaining` | Bytes remaining returns to 0 after sync |
| 40 | Optional test failover snapshot | Destination Server | `Mount-SRDestination -Name "RG-FS2-DATA" -ComputerName FS2` | Destination snapshot mounts for validation |
| 41 | Dismount test snapshot | Destination Server | `Dismount-SRDestination -Name "RG-FS2-DATA" -ComputerName FS2` | Test snapshot is removed |
| 42 | Planned failover direction reversal | Management Host | `Set-SRPartnership -NewSourceComputerName FS2 -SourceRGName "RG-FS2-DATA" -DestinationComputerName FS1 -DestinationRGName "RG-FS1-DATA"` | FS2 becomes new source after sync is healthy |
| 43 | Create or enable SMB share on DR source | Destination Server | `New-SmbShare -Name "Departments" -Path "D:\Shares\Departments" -FullAccess "CORP\Domain Admins" -ChangeAccess "CORP\GG_FS_Departments_Modify" -ReadAccess "CORP\GG_FS_Departments_Read"` | DR server publishes file share |
| 44 | Update DFS or DNS access path | DNS or DFS Server | `Update DFS target or DNS alias to point clients to FS2` | Clients reach active file server |
| 45 | Validate client access after failover | Client | `Test-Path "\\FS2\Departments"` | DR file server access works |
| 46 | Export configuration | Management Host | `Export-SRConfiguration -Path "C:\SR-Reports\sr-config.ps1"` | Replication config is exported |
| 47 | Document final state | Operator | `Record source, destination, groups, volumes, mode, report, failover plan, backup, and validation` | DR workbook evidence is complete |

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Prereq_Validation_Skeleton
```powershell
# Run in elevated PowerShell from a management host.
# Purpose: validate server identity, AD membership, OS, remoting, and basic requirements.

$SourceServer = "FS1"
$DestinationServer = "FS2"
$DomainFqdn = "corp.local"
$Servers = @($SourceServer, $DestinationServer)

foreach ($Server in $Servers) {
  Write-Host "===== $Server ====="

  Invoke-Command -ComputerName $Server -ScriptBlock {
    Write-Host "Computer name:"
    hostname

    Write-Host "Domain membership:"
    (Get-CimInstance Win32_ComputerSystem) |
      Select-Object Name, Domain, PartOfDomain

    Write-Host "Operating system:"
    Get-ComputerInfo |
      Select-Object WindowsProductName, WindowsVersion, OsName

    Write-Host "CPU and memory:"
    Get-CimInstance Win32_ComputerSystem |
      Select-Object NumberOfLogicalProcessors, TotalPhysicalMemory

    Write-Host "Volumes:"
    Get-Volume |
      Select-Object DriveLetter, FileSystem, HealthStatus, Size, SizeRemaining |
      Format-Table -AutoSize
  }
}

Write-Host "Testing WinRM:"
foreach ($Server in $Servers) {
  Test-WSMan $Server
}

Write-Host "Testing domain controller discovery from management host:"
nltest /dsgetdc:$DomainFqdn
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Feature_Install_Skeleton
```powershell
# Run in elevated PowerShell from a management host.
# Purpose: install File Server and Storage Replica on both file servers.

$Servers = @("FS1", "FS2")

foreach ($Server in $Servers) {
  Write-Host "Installing File Server and Storage Replica on $Server"

  Invoke-Command -ComputerName $Server -ScriptBlock {
    Install-WindowsFeature FS-FileServer, Storage-Replica -IncludeManagementTools

    Write-Host "Installed feature state:"
    Get-WindowsFeature FS-FileServer, Storage-Replica |
      Select-Object Name, DisplayName, InstallState

    Write-Host "Storage Replica cmdlets:"
    Get-Command -Module StorageReplica |
      Select-Object Name |
      Sort-Object Name
  }
}

Write-Host "Restart servers if the feature install requires it:"
Write-Host "Restart-Computer -ComputerName FS1,FS2 -Force"
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Volume_And_Log_Validation_Skeleton
```powershell
# Run in elevated PowerShell from a management host.
# Purpose: validate data and log volume layout before creating the partnership.

$SourceServer = "FS1"
$DestinationServer = "FS2"
$DataDrive = "D"
$LogDrive = "L"

$Servers = @($SourceServer, $DestinationServer)

foreach ($Server in $Servers) {
  Write-Host "===== $Server storage validation ====="

  Invoke-Command -ComputerName $Server -ArgumentList $DataDrive, $LogDrive -ScriptBlock {
    param($DataDrive, $LogDrive)

    Write-Host "Volumes:"
    Get-Volume -DriveLetter $DataDrive, $LogDrive |
      Select-Object DriveLetter, FileSystem, HealthStatus, Size, SizeRemaining |
      Format-Table -AutoSize

    Write-Host "Disks and sector sizes:"
    Get-Disk |
      Select-Object Number, FriendlyName, PartitionStyle, OperationalStatus, LogicalSectorSize, PhysicalSectorSize, Size |
      Format-Table -AutoSize

    Write-Host "Data volume path test:"
    Test-Path "$DataDrive`:\"

    Write-Host "Log volume path test:"
    Test-Path "$LogDrive`:\"

    Write-Host "Log volume contents:"
    Get-ChildItem "$LogDrive`:\" -Force -ErrorAction SilentlyContinue |
      Select-Object Name, Length, LastWriteTime |
      Format-Table -AutoSize
  }
}

Write-Host "Manual checks required:"
Write-Host "1. Source and destination data volumes must be same size."
Write-Host "2. Source and destination log volumes should be same size."
Write-Host "3. Log volumes must not host normal workloads."
Write-Host "4. Replicated data volume must not be the OS volume."
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Network_And_Firewall_Skeleton
```powershell
# Run in elevated PowerShell from a management host.
# Purpose: validate and enable required network paths for Storage Replica.

$SourceServer = "FS1"
$DestinationServer = "FS2"
$Servers = @($SourceServer, $DestinationServer)

foreach ($Server in $Servers) {
  Write-Host "===== Enabling firewall rules on $Server ====="

  Invoke-Command -ComputerName $Server -ScriptBlock {
    Enable-NetFirewallRule -DisplayGroup "File and Printer Sharing" -ErrorAction SilentlyContinue
    Enable-NetFirewallRule -DisplayGroup "Windows Remote Management" -ErrorAction SilentlyContinue

    Get-NetFirewallRule -DisplayGroup "File and Printer Sharing" -ErrorAction SilentlyContinue |
      Select-Object DisplayName, Enabled, Direction, Action, Profile |
      Format-Table -AutoSize

    Get-NetFirewallRule -DisplayGroup "Windows Remote Management" -ErrorAction SilentlyContinue |
      Select-Object DisplayName, Enabled, Direction, Action, Profile |
      Format-Table -AutoSize
  }
}

Write-Host "Testing required ports from source to destination:"
Invoke-Command -ComputerName $SourceServer -ArgumentList $DestinationServer -ScriptBlock {
  param($DestinationServer)

  Test-NetConnection $DestinationServer -Port 445
  Test-NetConnection $DestinationServer -Port 5985
}

Write-Host "Testing required ports from destination to source:"
Invoke-Command -ComputerName $DestinationServer -ArgumentList $SourceServer -ScriptBlock {
  param($SourceServer)

  Test-NetConnection $SourceServer -Port 445
  Test-NetConnection $SourceServer -Port 5985
}

Write-Host "If RDMA or SMB Direct is used, also validate TCP 5445 and RDMA adapter state."
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Topology_Test_Skeleton
```powershell
# Run in elevated PowerShell from a management host with Storage Replica tools.
# Purpose: run Microsoft topology validation before creating the partnership.

$SourceServer = "FS1"
$DestinationServer = "FS2"
$SourceDataVolume = "D:"
$SourceLogVolume = "L:"
$DestinationDataVolume = "D:"
$DestinationLogVolume = "L:"
$ResultPath = "C:\SR-Reports"
$DurationInMinutes = 30

New-Item -ItemType Directory -Force -Path $ResultPath | Out-Null

Write-Host "Running Test-SRTopology..."
Test-SRTopology `
  -SourceComputerName $SourceServer `
  -SourceVolumeName $SourceDataVolume `
  -SourceLogVolumeName $SourceLogVolume `
  -DestinationComputerName $DestinationServer `
  -DestinationVolumeName $DestinationDataVolume `
  -DestinationLogVolumeName $DestinationLogVolume `
  -DurationInMinutes $DurationInMinutes `
  -ResultPath $ResultPath

Write-Host "Topology test complete. Review the HTML report:"
Get-ChildItem $ResultPath -Filter "*.html" |
  Sort-Object LastWriteTime -Descending |
  Select-Object FullName, LastWriteTime |
  Format-Table -AutoSize
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Partnership_Create_Skeleton
```powershell
# Run in elevated PowerShell from a management host.
# Purpose: create server-to-server Storage Replica partnership.

$SourceServer = "FS1"
$DestinationServer = "FS2"

$SourceRG = "RG-FS1-DATA"
$DestinationRG = "RG-FS2-DATA"

$SourceDataVolume = "D:"
$SourceLogVolume = "L:"
$DestinationDataVolume = "D:"
$DestinationLogVolume = "L:"

$ReplicationMode = "Synchronous"

Write-Host "Creating Storage Replica partnership..."
New-SRPartnership `
  -SourceComputerName $SourceServer `
  -SourceRGName $SourceRG `
  -SourceVolumeName $SourceDataVolume `
  -SourceLogVolumeName $SourceLogVolume `
  -DestinationComputerName $DestinationServer `
  -DestinationRGName $DestinationRG `
  -DestinationVolumeName $DestinationDataVolume `
  -DestinationLogVolumeName $DestinationLogVolume `
  -ReplicationMode $ReplicationMode

Write-Host "Replication partnership:"
Get-SRPartnership | Format-List *

Write-Host "Replication groups:"
Get-SRGroup | Format-List *

Write-Host "Replica status:"
(Get-SRGroup).Replicas |
  Select-Object DataVolume, ReplicationStatus, ReplicationMode, NumOfBytesRemaining, LastInSyncTime |
  Format-Table -AutoSize
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Replication_Monitoring_Skeleton
```powershell
# Run in elevated PowerShell on source and destination, or from management host.
# Purpose: monitor initial sync, replication health, and performance counters.

$SourceRG = "RG-FS1-DATA"
$DestinationRG = "RG-FS2-DATA"
$ReportRoot = "C:\SR-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Current partnerships:"
Get-SRPartnership |
  Tee-Object -FilePath "$ReportRoot\sr-partnerships-$Timestamp.txt"

Write-Host "Current replication groups:"
Get-SRGroup |
  Tee-Object -FilePath "$ReportRoot\sr-groups-$Timestamp.txt"

Write-Host "Replica state:"
(Get-SRGroup).Replicas |
  Select-Object `
    DataVolume,
    ReplicationStatus,
    ReplicationMode,
    NumOfBytesRemaining,
    LastInSyncTime,
    LastOutOfSyncTime |
  Tee-Object -FilePath "$ReportRoot\sr-replicas-$Timestamp.txt" |
  Format-Table -AutoSize

Write-Host "Recent Storage Replica events:"
Get-WinEvent -ProviderName Microsoft-Windows-StorageReplica -MaxEvents 100 |
  Select-Object TimeCreated, Id, LevelDisplayName, Message |
  Export-Csv "$ReportRoot\sr-events-$Timestamp.csv" -NoTypeInformation

Write-Host "Sampling Storage Replica counters:"
Get-Counter `
  -Counter "\Storage Replica Statistics(*)\Current RPO",
           "\Storage Replica Statistics(*)\Avg. Network Send Latency",
           "\Storage Replica Statistics(*)\Current Log Queue Length",
           "\Storage Replica Statistics(*)\Total Bytes Sent",
           "\Storage Replica Statistics(*)\Total Bytes Received" `
  -SampleInterval 5 `
  -MaxSamples 6 |
  Export-Clixml "$ReportRoot\sr-counters-$Timestamp.xml"

Write-Host "Reports exported to $ReportRoot"
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Planned_Failover_Skeleton
```powershell
# Run in elevated PowerShell from a management host.
# Purpose: perform controlled failover by reversing replication direction.
# Do not run until initial sync is complete and the change window is approved.

$OldSourceServer = "FS1"
$NewSourceServer = "FS2"

$OldSourceRG = "RG-FS1-DATA"
$NewSourceRG = "RG-FS2-DATA"

Write-Host "Pre-failover partnership state:"
Get-SRPartnership | Format-List *

Write-Host "Pre-failover replica state:"
(Get-SRGroup).Replicas |
  Select-Object DataVolume, ReplicationStatus, NumOfBytesRemaining, LastInSyncTime |
  Format-Table -AutoSize

Write-Host "Validate NumOfBytesRemaining is 0 before planned failover."
Write-Host "Reversing replication direction so FS2 becomes source..."

Set-SRPartnership `
  -NewSourceComputerName $NewSourceServer `
  -SourceRGName $NewSourceRG `
  -DestinationComputerName $OldSourceServer `
  -DestinationRGName $OldSourceRG

Write-Host "Post-failover partnership state:"
Get-SRPartnership | Format-List *

Write-Host "Post-failover replica state:"
(Get-SRGroup).Replicas |
  Select-Object DataVolume, ReplicationStatus, NumOfBytesRemaining, LastInSyncTime |
  Format-Table -AutoSize
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Test_Failover_Snapshot_Skeleton
```powershell
# Run in elevated PowerShell on the destination server.
# Purpose: temporarily mount a destination snapshot for validation without breaking replication.

$DestinationServer = "FS2"
$DestinationRG = "RG-FS2-DATA"

Write-Host "Mounting destination snapshot for test validation..."
Mount-SRDestination `
  -Name $DestinationRG `
  -ComputerName $DestinationServer

Write-Host "Review available volumes after snapshot mount:"
Get-Volume |
  Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, Size, SizeRemaining |
  Format-Table -AutoSize

Write-Host "Validate data from mounted snapshot, then dismount it."
Write-Host "Dismounting destination snapshot..."
Dismount-SRDestination `
  -Name $DestinationRG `
  -ComputerName $DestinationServer

Write-Host "Snapshot dismounted."
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_File_Server_Cutover_Skeleton
```powershell
# Run after planned failover when the DR server is now the source.
# Purpose: publish the SMB share on the new active file server and validate user access.

$NewActiveServer = "FS2"
$ShareName = "Departments"
$SharePath = "D:\Shares\Departments"
$ChangeGroup = "CORP\GG_FS_Departments_Modify"
$ReadGroup = "CORP\GG_FS_Departments_Read"
$AdminGroup = "CORP\Domain Admins"

Invoke-Command -ComputerName $NewActiveServer -ArgumentList $ShareName, $SharePath, $AdminGroup, $ChangeGroup, $ReadGroup -ScriptBlock {
  param($ShareName, $SharePath, $AdminGroup, $ChangeGroup, $ReadGroup)

  Write-Host "Confirming replicated path exists:"
  Test-Path $SharePath

  Write-Host "Creating SMB share if missing:"
  if (-not (Get-SmbShare -Name $ShareName -ErrorAction SilentlyContinue)) {
    New-SmbShare `
      -Name $ShareName `
      -Path $SharePath `
      -FullAccess $AdminGroup `
      -ChangeAccess $ChangeGroup `
      -ReadAccess $ReadGroup
  }

  Get-SmbShare -Name $ShareName | Format-List *
  Get-SmbShareAccess -Name $ShareName
}

Write-Host "Client validation command:"
Write-Host "Test-Path \\$NewActiveServer\$ShareName"
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Status_And_Event_Review_Skeleton
```powershell
# Run from a management host.
# Purpose: export Storage Replica configuration, events, and counters for evidence.

$Servers = @("FS1", "FS2")
$ReportRoot = "C:\SR-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Exporting Storage Replica configuration script..."
Export-SRConfiguration -Path "$ReportRoot\sr-config-$Timestamp.ps1"

foreach ($Server in $Servers) {
  Write-Host "===== Exporting from $Server ====="

  Invoke-Command -ComputerName $Server -ArgumentList $ReportRoot, $Timestamp -ScriptBlock {
    param($ReportRoot, $Timestamp)

    $LocalReportRoot = $ReportRoot
    New-Item -ItemType Directory -Force -Path $LocalReportRoot | Out-Null

    Get-SRPartnership -ErrorAction SilentlyContinue |
      Export-Clixml "$LocalReportRoot\$env:COMPUTERNAME-sr-partnerships-$Timestamp.xml"

    Get-SRGroup -ErrorAction SilentlyContinue |
      Export-Clixml "$LocalReportRoot\$env:COMPUTERNAME-sr-groups-$Timestamp.xml"

    Get-WinEvent -ProviderName Microsoft-Windows-StorageReplica -MaxEvents 300 -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, LevelDisplayName, Message |
      Export-Csv "$LocalReportRoot\$env:COMPUTERNAME-sr-events-$Timestamp.csv" -NoTypeInformation

    Get-Counter `
      -Counter "\Storage Replica Statistics(*)\Current RPO",
               "\Storage Replica Statistics(*)\Avg. Network Send Latency",
               "\Storage Replica Statistics(*)\Current Log Queue Length" `
      -SampleInterval 5 `
      -MaxSamples 3 `
      -ErrorAction SilentlyContinue |
      Export-Clixml "$LocalReportRoot\$env:COMPUTERNAME-sr-counters-$Timestamp.xml"
  }
}

Write-Host "Storage Replica evidence exported under $ReportRoot on the relevant hosts."
```

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify feature installed | `Get-WindowsFeature Storage-Replica` | Storage Replica is installed |
| Verify cmdlets | `Get-Command -Module StorageReplica` | Storage Replica cmdlets are available |
| Verify source volume | `Get-Volume -DriveLetter D` | Source data volume is healthy |
| Verify destination volume | `Get-Volume -DriveLetter D` | Destination data volume exists and matches source size |
| Verify log volume | `Get-Volume -DriveLetter L` | Dedicated log volume exists |
| Verify disk sector sizes | `Get-Disk | Select Number,LogicalSectorSize,PhysicalSectorSize` | Sector sizes are compatible |
| Verify WinRM | `Test-WSMan FS1; Test-WSMan FS2` | Both servers respond |
| Verify SMB transport | `Test-NetConnection FS2 -Port 445` | TCP 445 succeeds |
| Verify topology | `Test-SRTopology -SourceComputerName FS1 -SourceVolumeName D: -SourceLogVolumeName L: -DestinationComputerName FS2 -DestinationVolumeName D: -DestinationLogVolumeName L: -DurationInMinutes 30 -ResultPath C:\SR-Reports` | Report is generated |
| Verify partnership | `Get-SRPartnership` | Partnership exists |
| Verify groups | `Get-SRGroup` | Replication groups exist |
| Verify replica progress | `(Get-SRGroup).Replicas | Select ReplicationStatus,NumOfBytesRemaining` | Initial sync progress is visible |
| Verify events | `Get-WinEvent -ProviderName Microsoft-Windows-StorageReplica -MaxEvents 20` | Storage Replica events return |
| Verify counters | `Get-Counter "\Storage Replica Statistics(*)\Current RPO"` | Counter data returns |
| Verify source SMB share | `Get-SmbShare -Name "Departments"` | Source file share exists |
| Verify source client access | `Test-Path "\\FS1\Departments"` | Client can reach active source share |
| Verify DR share after failover | `Test-Path "\\FS2\Departments"` | Client can reach DR share after cutover |
| Verify configuration export | `Export-SRConfiguration -Path "C:\SR-Reports\sr-config.ps1"` | Configuration script is exported |

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| Planned failover reversed direction | `Set-SRPartnership -NewSourceComputerName FS1 -SourceRGName "RG-FS1-DATA" -DestinationComputerName FS2 -DestinationRGName "RG-FS2-DATA"` | FS1 becomes source again |
| Test failover snapshot mounted | `Dismount-SRDestination -Name "RG-FS2-DATA" -ComputerName FS2` | Snapshot is removed |
| SMB share created on DR server | `Remove-SmbShare -Name "Departments" -Force` | DR SMB share is removed |
| DNS alias moved to DR server | Restore DNS alias target to source server | Clients resolve original active file server |
| DFS target moved to DR server | Re-enable source DFS target and disable DR target | DFS namespace points back to original target |
| Storage Replica partnership created | `Remove-SRPartnership -SourceComputerName FS1 -SourceRGName "RG-FS1-DATA" -DestinationComputerName FS2 -DestinationRGName "RG-FS2-DATA" -Confirm:$false` | Partnership is removed from current source |
| Source replication group remains | `Remove-SRGroup -Name "RG-FS1-DATA" -Confirm:$false` | Source replication group is removed |
| Destination replication group remains | `Remove-SRGroup -Name "RG-FS2-DATA" -Confirm:$false` | Destination replication group is removed |
| Storage Replica feature installed | `Uninstall-WindowsFeature Storage-Replica` | Feature is removed after partnerships and groups are removed |
| File Server role installed only for test | `Uninstall-WindowsFeature FS-FileServer` | File Server role is removed only if not needed |
| Firewall rules enabled | Disable only rules enabled for this lab | Firewall posture returns to baseline |
| Topology report created | `Remove-Item "C:\SR-Reports" -Recurse -Force` | Report folder is removed |

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `Storage-Replica` feature not available | Unsupported OS or edition | `Get-ComputerInfo`; `Get-WindowsFeature Storage-Replica` | Use supported Windows Server version and edition |
| Standard edition fails design limits | Standard edition supports limited scenario | OS edition and volume size | Use Datacenter or keep within single volume and size limits |
| `Test-SRTopology` fails | Missing feature, wrong volume, firewall, latency, or storage mismatch | Test report HTML | Fix failed prerequisites before partnership creation |
| Data volumes do not match | Destination data volume not same size | `Get-Volume -DriveLetter D` on both servers | Resize or recreate matching destination volume |
| Sector size mismatch | Disk hardware or virtual disk mismatch | `Get-Disk | Select LogicalSectorSize,PhysicalSectorSize` | Use compatible storage |
| Log volume too small | Default or selected log size not supported for workload | Test-SRTopology report | Increase log volume or use recommended log size |
| Log volume used by other workload | Bad design | `Get-ChildItem L:\ -Force` | Dedicate log volume to Storage Replica |
| Partnership creation fails on port | SMB or WinRM blocked | `Test-NetConnection <server> -Port 445`; `Test-WSMan` | Fix firewall, routing, or service |
| Partnership creation fails on permissions | User not local admin on both servers | `whoami /groups` | Use domain account with local admin rights |
| Destination drive disappears | Expected behavior during replication | `Get-SRGroup`; event logs | Do not write to destination volume while it is a destination |
| Initial sync never completes | Large dataset, slow link, bad disk, insufficient bandwidth, or high write churn | `(Get-SRGroup).Replicas`; counters | Fix storage or network bottleneck |
| Planned failover blocked | Initial sync incomplete | `NumOfBytesRemaining` | Wait for sync completion |
| Forced failover risks data loss | Source unavailable and destination may be behind | Replication status and events | Accept DR decision based on RPO and business approval |
| Users cannot access after failover | SMB share missing, DNS or DFS not updated, ACL issue | `Get-SmbShare`; `Resolve-DnsName`; `Test-Path` | Publish share and update access path |
| DFS users still hit old server | DFS target priority or cache | `dfsutil /pktinfo`; DFS management | Update DFS target and flush client referral cache |
| DNS alias points wrong | DNS CNAME or A record stale | `Resolve-DnsName files.corp.local` | Update DNS and flush cache |
| Replication performance poor | Latency, bandwidth, log disk, data disk, SMB path | Storage Replica counters | Improve network, log storage, SMB Direct, or use async |
| Synchronous replication hurts file server writes | Normal synchronous write latency effect | App write latency counters | Use lower-latency link or async mode |
| Deletes replicated to DR | Expected block-level replication behavior | File history, backups | Restore from backup, not replica |
| Ransomware replicated to DR | Replication is not point-in-time protection | Backup and security logs | Use backups, snapshots, immutable storage, and security controls |
| Cannot remove partnership | Running command on wrong server or wrong current source | `Get-SRPartnership` | Remove partnership from current source, then remove groups on both servers |

# Configure_Storage_Replica_For_File_Server_Disaster_Recovery_Related_Labs
| Lab                                                            | Relationship                                                                            |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `01_Install_File_Server_Role_And_Management_Tools.md`          | Establishes the Windows file server baseline                                            |
| `02_Create_And_Publish_SMB_Shares.md`                          | SMB shares must be recreated or redirected after DR failover                            |
| `03_Configure_NTFS_And_Share_Permissions.md`                   | Replicated data still requires correct NTFS and share permissions                       |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md`          | VSS snapshots are distinct from block replication and help recovery                     |
| `12_Backup_Restore_And_Export_File_Server_Config.md`           | Storage Replica does not replace backup                                                 |
| `13_Configure_DFS_Namespaces.md`                               | DFS namespaces can abstract the active file server target during DR                     |
| `14_Configure_DFS_Replication.md`                              | DFS-R is different from block-level Storage Replica and has different DR behavior       |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md` | SMB session monitoring helps confirm user impact before and after failover              |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`        | Troubleshooting validates DNS, SMB, auth, and permissions after cutover                 |
| `17_Configure_Data_Deduplication_For_File_Shares.md`           | Deduplication may exist on volumes being protected and must be validated                |
| `22_Configure_Azure_File_Sync_For_Windows_File_Server.md`      | Azure File Sync is a different hybrid sync model and not the same as Storage Replica DR |