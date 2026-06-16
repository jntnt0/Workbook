14_Configure_DFS_Replication.md
# Configure_DFS_Replication

# Configure_DFS_Replication_Index
14_Configure_DFS_Replication.md
Configure_DFS_Replication
Configure_DFS_Replication_Source_Basis
Configure_DFS_Replication_Mental_Model
Configure_DFS_Replication_Planning_Table
Configure_DFS_Replication_Configuration_Checklist
Configure_DFS_Replication_Precheck_Skeleton
Configure_DFS_Replication_Install_And_Module_Validation_Skeleton
Configure_DFS_Replication_Content_Preparation_Skeleton
Configure_DFS_Replication_Group_And_Members_Skeleton
Configure_DFS_Replication_Replicated_Folder_And_Membership_Skeleton
Configure_DFS_Replication_Connection_And_Schedule_Skeleton
Configure_DFS_Replication_Initial_Sync_And_Backlog_Skeleton
Configure_DFS_Replication_Health_Report_Skeleton
Configure_DFS_Replication_Client_DFSN_Target_Validation_Skeleton
Configure_DFS_Replication_Post_Change_Validation_Skeleton
Configure_DFS_Replication_Verification_Commands
Configure_DFS_Replication_Rollback
Configure_DFS_Replication_Failure_Checks
Configure_DFS_Replication_Related_Labs

# Configure_DFS_Replication_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DFS Replication | Multi-server replication of folder data between Windows file servers |
| Microsoft Learn | Install-WindowsFeature | Installing DFS Replication role service and management tools |
| Microsoft Learn | DFSR PowerShell module | Managing replication groups, members, folders, memberships, and connections |
| Microsoft Learn | New-DfsReplicationGroup | Creating DFS Replication groups |
| Microsoft Learn | Add-DfsrMember | Adding servers as replication group members |
| Microsoft Learn | New-DfsReplicatedFolder | Creating replicated folder definitions |
| Microsoft Learn | Set-DfsrMembership | Configuring member content paths, primary member, staging, and conflict settings |
| Microsoft Learn | Add-DfsrConnection | Creating replication connections between members |
| Microsoft Learn | Get-DfsrBacklog | Validating files waiting to replicate |
| Microsoft Learn | Sync-DfsReplicationGroup | Forcing AD polling and temporary replication sync |
| Microsoft Learn | Update-DfsrConfigurationFromAD | Forcing DFSR members to read updated AD configuration |
| Microsoft Learn | Write-DfsrHealthReport | Creating DFSR health reports |
| Windows Server operational practice | DFSN plus DFSR | DFS Namespaces gives users stable paths. DFSR keeps backend targets consistent |
| Windows Server operational practice | Initial sync discipline | Pick one authoritative primary member before enabling replication |
| Windows Server operational practice | Replication safety | DFSR is not a backup and is not for databases or high-churn open files |

# Configure_DFS_Replication_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DFSR | DFS Replication, a Windows role service that replicates folder content between servers |
| DFSN | DFS Namespaces, a separate role that gives users a logical path such as `\\corp.local\CorpFiles` |
| Replication group | DFSR container that defines a replication relationship between members |
| Replicated folder | Logical folder inside a replication group, such as `Departments` |
| Member | Server participating in the replication group |
| Membership | Per-server settings for a replicated folder, including content path and staging path |
| Primary member | Authoritative member used during initial replication seeding |
| Content path | Local folder path that contains replicated data |
| Staging folder | DFSR working area used to stage replicated file data |
| Staging quota | Maximum size of the staging folder |
| Conflict and Deleted folder | DFSR folder used to store conflict losers and deleted replicated files |
| Conflict quota | Maximum size of Conflict and Deleted storage |
| Connection | Directional replication path from one member to another |
| Replication topology | Pattern of connections, such as hub-and-spoke or full mesh |
| Schedule | Allowed replication time windows |
| Bandwidth throttle | Limit on replication bandwidth during scheduled windows |
| Backlog | Count/list of updates waiting to replicate from one member to another |
| Initial sync | First replication pass where non-primary members receive content |
| DFSR database | Per-volume DFSR metadata database used to track replicated files |
| DFSRPrivate | Hidden folder under the replicated folder containing staging/conflict data |
| USN journal | NTFS change journal DFSR uses to detect file changes |
| Pre-seeding | Copying data to secondary members before enabling DFSR to reduce replication traffic |
| Clone import/export | Advanced DFSR database cloning used for large pre-seeded datasets |
| First rule | Validate direct SMB paths and freeze conflicting writes before enabling initial DFSR sync |

# Configure_DFS_Replication_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Primary file server | `FS1` | `<primary-file-server>` |
| Secondary file server | `FS2` | `<secondary-file-server>` |
| Optional hub server | `FS1` | `<hub-server>` |
| Replication group name | `RG-Departments` | `<replication-group-name>` |
| Replicated folder name | `Departments` | `<replicated-folder-name>` |
| Primary content path | `E:\Shares\Departments` | `<primary-content-path>` |
| Secondary content path | `E:\Shares\Departments` | `<secondary-content-path>` |
| Primary staging path | `E:\Shares\Departments\DfsrPrivate\Staging` | `<primary-staging-path>` |
| Secondary staging path | `E:\Shares\Departments\DfsrPrivate\Staging` | `<secondary-staging-path>` |
| Staging quota | `4096 MB` | `<staging-quota-mb>` |
| Conflict and deleted quota | `4096 MB` | `<conflict-quota-mb>` |
| Topology | Full mesh or hub-and-spoke | `<replication-topology>` |
| Source member | `FS1` | `<source-member>` |
| Destination member | `FS2` | `<destination-member>` |
| Initial primary member | `FS1` | `<initial-primary-member>` |
| Pre-seed method | Robocopy or clean empty secondary | `<preseed-method>` |
| Replication schedule | Always or business-hours restricted | `<replication-schedule>` |
| Bandwidth limit | Full or throttled | `<bandwidth-plan>` |
| DFS namespace path | `\\corp.local\CorpFiles\Departments` | `<dfsn-folder-path>` |
| DFS folder target 1 | `\\FS1\Departments` | `<dfsn-target-1>` |
| DFS folder target 2 | `\\FS2\Departments` | `<dfsn-target-2>` |
| Data type | User documents only | `<data-type>` |
| Excluded workloads | Databases, PSTs, live VHDX, application data | `<excluded-workloads>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Health report path | `C:\FileServices-Validation\DFSR-Health` | `<health-report-path>` |
| Export path | `C:\FileServices-Exports\DFSR` | `<dfsr-export-path>` |
| Rollback stance | Disable membership or remove group after preserving data | `<rollback-plan>` |
| Next workbook | `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md` | `<next-task>` |

# Configure_DFS_Replication_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | FS1 / FS2 | `whoami /groups` | Admin context is visible |
| 2 | Confirm both servers are domain joined | FS1 / FS2 | `Get-ComputerInfo \| Select CsName,CsDomain,CsPartOfDomain` | Both servers are in the domain |
| 3 | Confirm DNS resolution between members | FS1 / FS2 | `Resolve-DnsName FS1`; `Resolve-DnsName FS2` | Both names resolve correctly |
| 4 | Confirm time sync | FS1 / FS2 | `w32tm /query /status` | Time sync is healthy |
| 5 | Confirm File Server role | FS1 / FS2 | `Get-WindowsFeature FS-FileServer` | File Server role is installed |
| 6 | Install DFS Replication role | FS1 / FS2 | `Install-WindowsFeature FS-DFS-Replication -IncludeManagementTools` | DFSR role is installed |
| 7 | Confirm DFSR service | FS1 / FS2 | `Get-Service DFSR` | DFSR service exists and runs |
| 8 | Confirm DFSR module | FS1 / FS2 | `Get-Command -Module DFSR` | DFSR cmdlets are available |
| 9 | Confirm content paths | FS1 / FS2 | `Test-Path "<content-path>"` | Content folders exist on both servers |
| 10 | Pre-seed data if needed | FS1 to FS2 | `robocopy "<source>" "\\FS2\<target-share>" /MIR /COPYALL /DCOPY:DAT /R:1 /W:1` | Secondary content is pre-seeded |
| 11 | Validate no unsupported data type | FS1 / FS2 | Inventory content | No databases, PSTs, VHDX, or high-churn app files are selected |
| 12 | Create replication group | Admin Host | `New-DfsReplicationGroup -GroupName "<replication-group-name>"` | Replication group exists |
| 13 | Add members | Admin Host | `Add-DfsrMember -GroupName "<replication-group-name>" -ComputerName FS1,FS2` | Both servers are group members |
| 14 | Create replicated folder | Admin Host | `New-DfsReplicatedFolder -GroupName "<replication-group-name>" -FolderName "<replicated-folder-name>"` | Replicated folder exists |
| 15 | Configure primary membership | Admin Host | `Set-DfsrMembership -GroupName "<group>" -FolderName "<folder>" -ComputerName FS1 -ContentPath "<path>" -PrimaryMember $true` | FS1 is initial primary |
| 16 | Configure secondary membership | Admin Host | `Set-DfsrMembership -GroupName "<group>" -FolderName "<folder>" -ComputerName FS2 -ContentPath "<path>"` | FS2 membership is configured |
| 17 | Configure staging and conflict quotas | Admin Host | `Set-DfsrMembership ... -StagingPathQuotaInMB <mb> -ConflictAndDeletedQuotaInMB <mb>` | Staging and conflict settings match plan |
| 18 | Add DFSR connection FS1 to FS2 | Admin Host | `Add-DfsrConnection -GroupName "<group>" -SourceComputerName FS1 -DestinationComputerName FS2` | Replication connection exists |
| 19 | Add reverse DFSR connection if needed | Admin Host | `Add-DfsrConnection -GroupName "<group>" -SourceComputerName FS2 -DestinationComputerName FS1` | Reverse replication exists |
| 20 | Force AD config polling | FS1 / FS2 | `Update-DfsrConfigurationFromAD` or `dfsrdiag pollad` | Members read new DFSR config |
| 21 | Validate memberships | Admin Host | `Get-DfsrMembership -GroupName "<group>"` | Members show content path and state |
| 22 | Validate connections | Admin Host | `Get-DfsrConnection -GroupName "<group>"` | Connections are enabled |
| 23 | Monitor initial sync | Admin Host | `Get-DfsrBacklog -GroupName "<group>" -FolderName "<folder>" -SourceComputerName FS1 -DestinationComputerName FS2` | Backlog decreases |
| 24 | Create test file on primary | FS1 | `New-Item "<primary-content-path>\dfsr-test.txt"` | Test file exists on primary |
| 25 | Confirm test file replicates | FS2 | `Test-Path "<secondary-content-path>\dfsr-test.txt"` | Test file appears on secondary |
| 26 | Generate health report | Admin Host | `Write-DfsrHealthReport -GroupName "<group>" -Path "<health-report-path>"` | DFSR report is generated |
| 27 | Add secondary DFSN folder target if namespace is ready | Namespace Server | `New-DfsnFolderTarget -Path "<dfsn-folder-path>" -TargetPath "<dfsn-target-2>"` | DFSN can refer clients to replicated target |
| 28 | Validate DFSN target access | Client | `Test-Path "\\corp.local\CorpFiles\Departments"` | Namespace path works |
| 29 | Export DFSR config evidence | Admin Host | Use post-change validation skeleton | DFSR evidence is saved |
| 30 | Document replication model | Operator | `Record group, members, content paths, topology, schedule, backlog, and health report` | DFSR build record is complete |

# Configure_DFS_Replication_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on both file servers or from an admin host with remoting.
# Purpose: validate DNS, domain, features, paths, SMB, and DFSR state before creating replication.

$EvidencePath = "C:\FileServices-Validation"

$DomainFqdn = "corp.local"
$PrimaryMember = "FS1"
$SecondaryMember = "FS2"

$ReplicationGroupName = "RG-Departments"
$ReplicatedFolderName = "Departments"

$PrimaryContentPath = "E:\Shares\Departments"
$SecondaryContentPath = "E:\Shares\Departments"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-dfsr-precheck-transcript.txt"

# Admin context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-dfsr.txt"

# Local server state.
hostname |
  Tee-Object "$EvidencePath\hostname-before-dfsr.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-dfsr.txt"

# DNS and DC discovery.
Resolve-DnsName $DomainFqdn |
  Tee-Object "$EvidencePath\domain-dns-resolution-before-dfsr.txt"

Resolve-DnsName $PrimaryMember |
  Tee-Object "$EvidencePath\primary-member-dns-before-dfsr.txt"

Resolve-DnsName $SecondaryMember |
  Tee-Object "$EvidencePath\secondary-member-dns-before-dfsr.txt"

nltest /dsgetdc:$DomainFqdn |
  Tee-Object "$EvidencePath\nltest-dc-discovery-before-dfsr.txt"

# Local feature and service state.
Get-WindowsFeature FS-FileServer,FS-DFS-Replication,RSAT-DFS-Mgmt-Con |
  Tee-Object "$EvidencePath\dfs-features-before-dfsr.txt"

Get-Service DFSR -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsr-service-before.txt"

# Target path validation from this server.
foreach ($Path in @($PrimaryContentPath,$SecondaryContentPath)) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\dfsr-local-path-validation-before.txt" -Append
}

# Remote member validation.
$Members = @($PrimaryMember,$SecondaryMember)

foreach ($Member in $Members) {
  Invoke-Command -ComputerName $Member -ScriptBlock {
    $ContentPath = "E:\Shares\Departments"

    [PSCustomObject]@{
      ComputerName = $env:COMPUTERNAME
      DomainJoined = (Get-ComputerInfo).CsPartOfDomain
      Domain = (Get-ComputerInfo).CsDomain
      FileServerFeature = (Get-WindowsFeature FS-FileServer).InstallState
      DfsrFeature = (Get-WindowsFeature FS-DFS-Replication).InstallState
      DfsrService = (Get-Service DFSR -ErrorAction SilentlyContinue).Status
      ContentPath = $ContentPath
      ContentPathExists = Test-Path $ContentPath
    }

    Get-Volume | Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining
  } |
    Tee-Object "$EvidencePath\remote-member-validation-before-dfsr.txt" -Append
}

# SMB share and DFSN context.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-before-dfsr.txt"

Get-DfsnFolder -Path "\\$DomainFqdn\CorpFiles\*" -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-folders-before-dfsr.txt"

# Existing DFSR config if present.
Get-Command -Module DFSR -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsr-cmdlets-before-dfsr.txt"

Get-DfsReplicationGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsr-groups-before.txt"

Get-DfsReplicationGroup -GroupName $ReplicationGroupName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-dfsr-group-before.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Install_And_Module_Validation_Skeleton
```powershell
# Run in elevated PowerShell on each DFSR member.
# Purpose: install DFS Replication role service and management tools.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-dfsr-install-module-validation-transcript.txt"

# Install DFS Replication and management tools.
Install-WindowsFeature `
  -Name FS-DFS-Replication `
  -IncludeManagementTools

Install-WindowsFeature `
  -Name RSAT-DFS-Mgmt-Con `
  -IncludeAllSubFeature

# Confirm role and tools.
Get-WindowsFeature FS-DFS-Replication,RSAT-DFS-Mgmt-Con |
  Tee-Object "$EvidencePath\dfsr-features-after-install.txt"

# Confirm DFSR service.
Get-Service DFSR |
  Tee-Object "$EvidencePath\dfsr-service-after-install.txt"

Start-Service DFSR -ErrorAction SilentlyContinue

Get-Service DFSR |
  Tee-Object "$EvidencePath\dfsr-service-after-start.txt"

# Confirm module and cmdlets.
Get-Module DFSR -ListAvailable |
  Tee-Object "$EvidencePath\dfsr-module-after-install.txt"

Import-Module DFSR

Get-Command -Module DFSR |
  Sort-Object Name |
  Tee-Object "$EvidencePath\dfsr-cmdlets-after-install.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Content_Preparation_Skeleton
```powershell
# Run before enabling DFSR.
# Purpose: prepare source and destination content paths and optionally pre-seed data.
# Warning: Do not run /MIR against the wrong destination. It can delete data.

$EvidencePath = "C:\FileServices-Validation"

$PrimaryMember = "FS1"
$SecondaryMember = "FS2"

$SourcePath = "E:\Shares\Departments"
$DestinationUNC = "\\FS2\Departments"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-dfsr-content-preparation-transcript.txt"

# Validate source and destination.
Test-Path $SourcePath |
  Tee-Object "$EvidencePath\primary-source-path-testpath-before-preseed.txt"

Test-Path $DestinationUNC |
  Tee-Object "$EvidencePath\secondary-destination-unc-testpath-before-preseed.txt"

# Capture source inventory.
Get-ChildItem $SourcePath -Recurse -Force -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Export-Csv "$EvidencePath\primary-content-inventory-before-preseed.csv" -NoTypeInformation

# Optional pre-seed with Robocopy.
# Use this when FS1 is authoritative and FS2 should receive the same data before DFSR initial sync.
robocopy $SourcePath $DestinationUNC /MIR /COPYALL /DCOPY:DAT /R:1 /W:1 /XJ /FFT /TEE /LOG:"$EvidencePath\robocopy-preseed-fs1-to-fs2.log"

# Validate destination inventory.
Get-ChildItem $DestinationUNC -Recurse -Force -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Export-Csv "$EvidencePath\secondary-content-inventory-after-preseed.csv" -NoTypeInformation

# Optional file hash sample for validation.
Get-ChildItem $SourcePath -File -Recurse -ErrorAction SilentlyContinue |
  Select-Object -First 25 |
  Get-FileHash |
  Export-Csv "$EvidencePath\primary-sample-hashes-before-dfsr.csv" -NoTypeInformation

Get-ChildItem $DestinationUNC -File -Recurse -ErrorAction SilentlyContinue |
  Select-Object -First 25 |
  Get-FileHash |
  Export-Csv "$EvidencePath\secondary-sample-hashes-after-preseed.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_DFS_Replication_Group_And_Members_Skeleton
```powershell
# Run from a DFSR management host or one DFSR member.
# Purpose: create replication group and add server members.

$EvidencePath = "C:\FileServices-Validation"

$ReplicationGroupName = "RG-Departments"
$PrimaryMember = "FS1"
$SecondaryMember = "FS2"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-dfsr-group-and-members-transcript.txt"

Import-Module DFSR

# Capture existing groups.
Get-DfsReplicationGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsr-groups-before-create.txt"

# Create replication group if missing.
if (-not (Get-DfsReplicationGroup -GroupName $ReplicationGroupName -ErrorAction SilentlyContinue)) {
  New-DfsReplicationGroup `
    -GroupName $ReplicationGroupName `
    -Description "DFS Replication group for department shares"
}

# Add members.
$Members = @($PrimaryMember,$SecondaryMember)

foreach ($Member in $Members) {
  $ExistingMember = Get-DfsrMember -GroupName $ReplicationGroupName -ErrorAction SilentlyContinue |
    Where-Object { $_.ComputerName -eq $Member }

  if (-not $ExistingMember) {
    Add-DfsrMember `
      -GroupName $ReplicationGroupName `
      -ComputerName $Member
  }
}

# Confirm group and members.
Get-DfsReplicationGroup -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-group-after-create.txt"

Get-DfsrMember -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-members-after-add.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Replicated_Folder_And_Membership_Skeleton
```powershell
# Run from a DFSR management host or one DFSR member.
# Purpose: create replicated folder and configure each member's content path.
# FS1 is marked primary only for initial sync.

$EvidencePath = "C:\FileServices-Validation"

$ReplicationGroupName = "RG-Departments"
$ReplicatedFolderName = "Departments"

$PrimaryMember = "FS1"
$SecondaryMember = "FS2"

$PrimaryContentPath = "E:\Shares\Departments"
$SecondaryContentPath = "E:\Shares\Departments"

$StagingQuotaMb = 4096
$ConflictQuotaMb = 4096

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-dfsr-replicated-folder-membership-transcript.txt"

Import-Module DFSR

# Create replicated folder if missing.
if (-not (Get-DfsReplicatedFolder -GroupName $ReplicationGroupName -FolderName $ReplicatedFolderName -ErrorAction SilentlyContinue)) {
  New-DfsReplicatedFolder `
    -GroupName $ReplicationGroupName `
    -FolderName $ReplicatedFolderName `
    -Description "Replicated department file share data"
}

# Configure primary member.
Set-DfsrMembership `
  -GroupName $ReplicationGroupName `
  -FolderName $ReplicatedFolderName `
  -ComputerName $PrimaryMember `
  -ContentPath $PrimaryContentPath `
  -StagingPathQuotaInMB $StagingQuotaMb `
  -ConflictAndDeletedQuotaInMB $ConflictQuotaMb `
  -PrimaryMember $true `
  -Force

# Configure secondary member.
Set-DfsrMembership `
  -GroupName $ReplicationGroupName `
  -FolderName $ReplicatedFolderName `
  -ComputerName $SecondaryMember `
  -ContentPath $SecondaryContentPath `
  -StagingPathQuotaInMB $StagingQuotaMb `
  -ConflictAndDeletedQuotaInMB $ConflictQuotaMb `
  -Force

# Confirm replicated folder and memberships.
Get-DfsReplicatedFolder -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-replicated-folder-after-create.txt"

Get-DfsrMembership -GroupName $ReplicationGroupName -FolderName $ReplicatedFolderName |
  Tee-Object "$EvidencePath\dfsr-memberships-after-config.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Connection_And_Schedule_Skeleton
```powershell
# Run from a DFSR management host or one DFSR member.
# Purpose: create DFSR connections and configure replication schedule behavior.

$EvidencePath = "C:\FileServices-Validation"

$ReplicationGroupName = "RG-Departments"
$PrimaryMember = "FS1"
$SecondaryMember = "FS2"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-dfsr-connection-schedule-transcript.txt"

Import-Module DFSR

# Create bidirectional connections if missing.
$ForwardConnection = Get-DfsrConnection `
  -GroupName $ReplicationGroupName `
  -SourceComputerName $PrimaryMember `
  -DestinationComputerName $SecondaryMember `
  -ErrorAction SilentlyContinue

if (-not $ForwardConnection) {
  Add-DfsrConnection `
    -GroupName $ReplicationGroupName `
    -SourceComputerName $PrimaryMember `
    -DestinationComputerName $SecondaryMember
}

$ReverseConnection = Get-DfsrConnection `
  -GroupName $ReplicationGroupName `
  -SourceComputerName $SecondaryMember `
  -DestinationComputerName $PrimaryMember `
  -ErrorAction SilentlyContinue

if (-not $ReverseConnection) {
  Add-DfsrConnection `
    -GroupName $ReplicationGroupName `
    -SourceComputerName $SecondaryMember `
    -DestinationComputerName $PrimaryMember
}

# Enable connections.
Set-DfsrConnection `
  -GroupName $ReplicationGroupName `
  -SourceComputerName $PrimaryMember `
  -DestinationComputerName $SecondaryMember `
  -Enabled $true

Set-DfsrConnection `
  -GroupName $ReplicationGroupName `
  -SourceComputerName $SecondaryMember `
  -DestinationComputerName $PrimaryMember `
  -Enabled $true

# Optional: leave default full schedule.
# For strict scheduling, use Set-DfsrGroupSchedule or Set-DfsrConnectionSchedule after planning.

# Confirm connections.
Get-DfsrConnection -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-connections-after-config.txt"

# Capture group schedule.
Get-DfsrGroupSchedule -GroupName $ReplicationGroupName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsr-group-schedule-after-config.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Initial_Sync_And_Backlog_Skeleton
```powershell
# Run from a DFSR management host after AD replication has had time to converge.
# Purpose: force DFSR members to read configuration, start initial sync, and monitor backlog.

$EvidencePath = "C:\FileServices-Validation"

$ReplicationGroupName = "RG-Departments"
$ReplicatedFolderName = "Departments"

$PrimaryMember = "FS1"
$SecondaryMember = "FS2"

$PrimaryContentPath = "E:\Shares\Departments"
$SecondaryContentPath = "\\FS2\Departments"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-dfsr-initial-sync-backlog-transcript.txt"

Import-Module DFSR

# Force local AD polling from this machine.
Update-DfsrConfigurationFromAD |
  Tee-Object "$EvidencePath\update-dfsr-configuration-from-ad-local.txt"

# Force AD polling on both members.
foreach ($Member in @($PrimaryMember,$SecondaryMember)) {
  Invoke-Command -ComputerName $Member -ScriptBlock {
    dfsrdiag pollad
    Get-Service DFSR
  } |
    Tee-Object "$EvidencePath\dfsr-pollad-$Member.txt"
}

# Trigger sync for a limited window.
Sync-DfsReplicationGroup `
  -GroupName $ReplicationGroupName `
  -SourceComputerName $PrimaryMember `
  -DestinationComputerName $SecondaryMember `
  -DurationInMinutes 15 |
  Tee-Object "$EvidencePath\sync-dfsr-group-fs1-to-fs2.txt"

# Check backlog from primary to secondary.
Get-DfsrBacklog `
  -GroupName $ReplicationGroupName `
  -FolderName $ReplicatedFolderName `
  -SourceComputerName $PrimaryMember `
  -DestinationComputerName $SecondaryMember |
  Tee-Object "$EvidencePath\dfsr-backlog-fs1-to-fs2.txt"

# Check reverse backlog.
Get-DfsrBacklog `
  -GroupName $ReplicationGroupName `
  -FolderName $ReplicatedFolderName `
  -SourceComputerName $SecondaryMember `
  -DestinationComputerName $PrimaryMember |
  Tee-Object "$EvidencePath\dfsr-backlog-fs2-to-fs1.txt"

# Create test file on primary.
$TestFile = Join-Path $PrimaryContentPath "dfsr-replication-test.txt"

"DFSR test from $PrimaryMember at $(Get-Date)" |
  Out-File $TestFile -Force

# Wait and validate on secondary UNC.
Start-Sleep -Seconds 30

Test-Path "\\$SecondaryMember\Departments\dfsr-replication-test.txt" |
  Tee-Object "$EvidencePath\dfsr-test-file-replication-result.txt"

# Capture DFSR replicated folder state through WMI on both members.
foreach ($Member in @($PrimaryMember,$SecondaryMember)) {
  Invoke-Command -ComputerName $Member -ScriptBlock {
    Get-WmiObject -Namespace root\MicrosoftDFS -Class DfsrReplicatedFolderInfo |
      Select-Object ReplicationGroupName,ReplicatedFolderName,State,UpdatesReceived,UpdatesSent,ConflictBytesCleaned,ConflictFolderCleanupsCompleted
  } |
    Tee-Object "$EvidencePath\dfsr-wmi-replicated-folder-info-$Member.txt"
}

Stop-Transcript
```

# Configure_DFS_Replication_Health_Report_Skeleton
```powershell
# Run from a DFSR management host.
# Purpose: generate DFSR health report and capture event evidence.

$EvidencePath = "C:\FileServices-Validation"
$HealthReportPath = "$EvidencePath\DFSR-Health"

$ReplicationGroupName = "RG-Departments"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $HealthReportPath

Start-Transcript -Path "$EvidencePath\14-dfsr-health-report-transcript.txt"

Import-Module DFSR

# Generate health report.
Write-DfsrHealthReport `
  -GroupName $ReplicationGroupName `
  -Path $HealthReportPath

# Capture report files.
Get-ChildItem $HealthReportPath -Recurse |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\dfsr-health-report-files.txt"

# Capture DFS Replication event log.
Get-WinEvent -LogName "DFS Replication" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\dfsr-events-health-report.txt"

# Capture common DFSR warnings/errors.
Get-WinEvent -LogName "DFS Replication" -MaxEvents 500 -ErrorAction SilentlyContinue |
  Where-Object { $_.LevelDisplayName -in @("Warning","Error") } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\dfsr-warning-error-events.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Client_DFSN_Target_Validation_Skeleton
```powershell
# Run from a Windows client after DFSR is healthy and DFSN has multiple targets.
# Purpose: validate user namespace path and backend target behavior.

$EvidencePath = "C:\FileServices-Validation-Client"

$DomainFqdn = "corp.local"
$NamespaceRootName = "CorpFiles"
$DfsDepartmentPath = "\\$DomainFqdn\$NamespaceRootName\Departments"

$TestFileName = "dfsn-dfsr-client-test-$env:USERNAME.txt"
$TestFilePath = Join-Path $DfsDepartmentPath $TestFileName

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\14-client-dfsn-dfsr-target-validation-transcript.txt"

# Client identity and token.
hostname |
  Tee-Object "$EvidencePath\client-hostname-dfsr.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami-dfsr.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups-dfsr.txt"

# DNS and namespace tests.
Resolve-DnsName $DomainFqdn |
  Tee-Object "$EvidencePath\client-domain-resolution-dfsr.txt"

Test-Path $DfsDepartmentPath |
  Tee-Object "$EvidencePath\client-dfs-department-path-test.txt"

Get-ChildItem $DfsDepartmentPath -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-dfs-department-listing.txt"

# Write test if authorized.
try {
  "Client DFSN/DFSR test from $env:USERNAME at $(Get-Date)" |
    Out-File $TestFilePath -Force

  Get-Content $TestFilePath |
    Tee-Object "$EvidencePath\client-dfsn-dfsr-test-file-readback.txt"

  Remove-Item $TestFilePath -Force
}
catch {
  "Client DFSN/DFSR write test failed: $($_.Exception.Message)" |
    Tee-Object "$EvidencePath\client-dfsn-dfsr-write-test-result.txt"
}

# DFS referral cache.
dfsutil /pktinfo |
  Tee-Object "$EvidencePath\client-dfsutil-pktinfo-dfsr.txt"

dfsutil /spcinfo |
  Tee-Object "$EvidencePath\client-dfsutil-spcinfo-dfsr.txt"

# SMB connection state.
Get-SmbConnection |
  Select-Object ServerName,ShareName,Dialect,Signed,Encrypted,UserName |
  Tee-Object "$EvidencePath\client-smb-connections-dfsr.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Post_Change_Validation_Skeleton
```powershell
# Run from a DFSR management host.
# Purpose: capture final DFSR group, membership, connection, backlog, health, event, and namespace evidence.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"
$DfsrExportPath = Join-Path $ExportRoot "DFSR"

$ReplicationGroupName = "RG-Departments"
$ReplicatedFolderName = "Departments"

$PrimaryMember = "FS1"
$SecondaryMember = "FS2"

$NamespacePath = "\\corp.local\CorpFiles\Departments"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $DfsrExportPath

Start-Transcript -Path "$EvidencePath\14-post-change-dfsr-validation-transcript.txt"

Import-Module DFSR

# DFSR feature and service state on both members.
foreach ($Member in @($PrimaryMember,$SecondaryMember)) {
  Invoke-Command -ComputerName $Member -ScriptBlock {
    Get-WindowsFeature FS-DFS-Replication
    Get-Service DFSR
    dfsrdiag pollad
  } |
    Tee-Object "$EvidencePath\dfsr-member-final-state-$Member.txt"
}

# Replication group objects.
Get-DfsReplicationGroup -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-group-final.txt"

Get-DfsrMember -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-members-final.txt"

Get-DfsReplicatedFolder -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-replicated-folders-final.txt"

Get-DfsrMembership -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-memberships-final.txt"

Get-DfsrConnection -GroupName $ReplicationGroupName |
  Tee-Object "$EvidencePath\dfsr-connections-final.txt"

# Export objects.
Get-DfsReplicationGroup -GroupName $ReplicationGroupName |
  Export-Clixml "$DfsrExportPath\dfsr-group.clixml"

Get-DfsrMember -GroupName $ReplicationGroupName |
  Export-Clixml "$DfsrExportPath\dfsr-members.clixml"

Get-DfsReplicatedFolder -GroupName $ReplicationGroupName |
  Export-Clixml "$DfsrExportPath\dfsr-replicated-folders.clixml"

Get-DfsrMembership -GroupName $ReplicationGroupName |
  Export-Clixml "$DfsrExportPath\dfsr-memberships.clixml"

Get-DfsrConnection -GroupName $ReplicationGroupName |
  Export-Clixml "$DfsrExportPath\dfsr-connections.clixml"

# Backlog checks.
Get-DfsrBacklog `
  -GroupName $ReplicationGroupName `
  -FolderName $ReplicatedFolderName `
  -SourceComputerName $PrimaryMember `
  -DestinationComputerName $SecondaryMember |
  Tee-Object "$EvidencePath\dfsr-backlog-final-fs1-to-fs2.txt"

Get-DfsrBacklog `
  -GroupName $ReplicationGroupName `
  -FolderName $ReplicatedFolderName `
  -SourceComputerName $SecondaryMember `
  -DestinationComputerName $PrimaryMember |
  Tee-Object "$EvidencePath\dfsr-backlog-final-fs2-to-fs1.txt"

# DFSR WMI state.
foreach ($Member in @($PrimaryMember,$SecondaryMember)) {
  Invoke-Command -ComputerName $Member -ScriptBlock {
    Get-WmiObject -Namespace root\MicrosoftDFS -Class DfsrReplicatedFolderInfo |
      Select-Object ReplicationGroupName,ReplicatedFolderName,State,UpdatesReceived,UpdatesSent,ConflictBytesCleaned,ConflictFolderCleanupsCompleted
  } |
    Tee-Object "$EvidencePath\dfsr-wmi-final-$Member.txt"
}

# DFSN path context.
Test-Path $NamespacePath |
  Tee-Object "$EvidencePath\dfsn-department-path-final-testpath.txt"

Get-DfsnFolderTarget -Path $NamespacePath -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-department-folder-targets-final-dfsr.txt"

# Events.
Get-WinEvent -LogName "DFS Replication" -MaxEvents 300 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\dfsr-events-final.txt"

# Export files.
Get-ChildItem $DfsrExportPath |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\dfsr-export-files-final.txt"

Stop-Transcript
```

# Configure_DFS_Replication_Verification_Commands
```powershell
# Feature and module
Get-WindowsFeature FS-DFS-Replication
Install-WindowsFeature FS-DFS-Replication -IncludeManagementTools
Get-Service DFSR
Start-Service DFSR
Get-Module DFSR -ListAvailable
Import-Module DFSR
Get-Command -Module DFSR

# Domain, DNS, and server reachability
Resolve-DnsName <domain-fqdn>
Resolve-DnsName <primary-file-server>
Resolve-DnsName <secondary-file-server>
nltest /dsgetdc:<domain-fqdn>
Test-NetConnection <secondary-file-server> -Port 445
Test-NetConnection <secondary-file-server> -Port 5722

# Content path validation
Test-Path "<primary-content-path>"
Test-Path "\\<secondary-file-server>\<share-name>"
icacls "<primary-content-path>"

# DFSR group, members, folders
Get-DfsReplicationGroup
Get-DfsReplicationGroup -GroupName "<replication-group-name>"
New-DfsReplicationGroup -GroupName "<replication-group-name>"
Add-DfsrMember -GroupName "<replication-group-name>" -ComputerName <primary-file-server>,<secondary-file-server>
Get-DfsrMember -GroupName "<replication-group-name>"

New-DfsReplicatedFolder -GroupName "<replication-group-name>" -FolderName "<replicated-folder-name>"
Get-DfsReplicatedFolder -GroupName "<replication-group-name>"

# Memberships
Set-DfsrMembership -GroupName "<replication-group-name>" -FolderName "<replicated-folder-name>" -ComputerName <primary-file-server> -ContentPath "<primary-content-path>" -PrimaryMember $true -Force
Set-DfsrMembership -GroupName "<replication-group-name>" -FolderName "<replicated-folder-name>" -ComputerName <secondary-file-server> -ContentPath "<secondary-content-path>" -Force
Get-DfsrMembership -GroupName "<replication-group-name>"

# Connections
Add-DfsrConnection -GroupName "<replication-group-name>" -SourceComputerName <primary-file-server> -DestinationComputerName <secondary-file-server>
Add-DfsrConnection -GroupName "<replication-group-name>" -SourceComputerName <secondary-file-server> -DestinationComputerName <primary-file-server>
Get-DfsrConnection -GroupName "<replication-group-name>"

# AD polling and sync
Update-DfsrConfigurationFromAD
dfsrdiag pollad
Sync-DfsReplicationGroup -GroupName "<replication-group-name>" -SourceComputerName <primary-file-server> -DestinationComputerName <secondary-file-server> -DurationInMinutes 15

# Backlog
Get-DfsrBacklog -GroupName "<replication-group-name>" -FolderName "<replicated-folder-name>" -SourceComputerName <primary-file-server> -DestinationComputerName <secondary-file-server>
Get-DfsrBacklog -GroupName "<replication-group-name>" -FolderName "<replicated-folder-name>" -SourceComputerName <secondary-file-server> -DestinationComputerName <primary-file-server>

# Legacy dfsrdiag backlog
dfsrdiag backlog /rgname:<replication-group-name> /rfname:<replicated-folder-name> /smem:<primary-file-server> /rmem:<secondary-file-server>
dfsrdiag backlog /rgname:<replication-group-name> /rfname:<replicated-folder-name> /smem:<secondary-file-server> /rmem:<primary-file-server>

# WMI replicated folder state
Get-WmiObject -Namespace root\MicrosoftDFS -Class DfsrReplicatedFolderInfo |
  Select-Object ReplicationGroupName,ReplicatedFolderName,State,UpdatesReceived,UpdatesSent

# Health report
Write-DfsrHealthReport -GroupName "<replication-group-name>" -Path "<health-report-path>"
Get-ChildItem "<health-report-path>" -Recurse

# DFSN context after DFSR
Get-DfsnFolderTarget -Path "<dfsn-folder-path>"
New-DfsnFolderTarget -Path "<dfsn-folder-path>" -TargetPath "<dfsn-target-2>"
Test-Path "<dfsn-folder-path>"
dfsutil /pktinfo
dfsutil /pktflush

# Events
Get-WinEvent -LogName "DFS Replication" -MaxEvents 200
Get-WinEvent -LogName "DFS Replication" -MaxEvents 500 | Where-Object LevelDisplayName -in "Warning","Error"

# Rollback
Remove-DfsrConnection -GroupName "<replication-group-name>" -SourceComputerName <source> -DestinationComputerName <destination> -Force
Remove-DfsrMember -GroupName "<replication-group-name>" -ComputerName <member> -Force
Remove-DfsReplicatedFolder -GroupName "<replication-group-name>" -FolderName "<replicated-folder-name>" -Force
Remove-DfsReplicationGroup -GroupName "<replication-group-name>" -Force

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*dfsr*"
Get-ChildItem "<dfsr-export-path>"
```

# Configure_DFS_Replication_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop user writes before rollback | File Servers / Clients | Communicate freeze or remove DFSN target temporarily | Data changes stop during rollback |
| 2 | Capture DFSR state before rollback | Admin Host | `Get-DfsReplicationGroup`; `Get-DfsrMembership`; `Get-DfsrConnection` | Current DFSR state is saved |
| 3 | Capture backlog before rollback | Admin Host | `Get-DfsrBacklog -GroupName "<group>" -FolderName "<folder>" -SourceComputerName FS1 -DestinationComputerName FS2` | Pending changes are documented |
| 4 | Capture latest content inventory | FS1 / FS2 | `Get-ChildItem "<content-path>" -Recurse` | File inventory is saved |
| 5 | Remove secondary DFSN folder target first if used | Namespace Server | `Remove-DfsnFolderTarget -Path "<dfsn-folder-path>" -TargetPath "<dfsn-target-2>" -Force` | Clients stop being referred to secondary target |
| 6 | Disable DFSR membership instead of deleting if troubleshooting | Admin Host | `Set-DfsrMembership -GroupName "<group>" -FolderName "<folder>" -ComputerName "<member>" -Enabled $false -Force` | Member stops participating |
| 7 | Remove DFSR connection if rollback requires it | Admin Host | `Remove-DfsrConnection -GroupName "<group>" -SourceComputerName "<source>" -DestinationComputerName "<destination>" -Force` | Connection is removed |
| 8 | Remove secondary DFSR member if needed | Admin Host | `Remove-DfsrMember -GroupName "<group>" -ComputerName "<secondary-member>" -Force` | Secondary member is removed from group |
| 9 | Remove replicated folder if lab teardown | Admin Host | `Remove-DfsReplicatedFolder -GroupName "<group>" -FolderName "<folder>" -Force` | Replicated folder definition is removed |
| 10 | Remove replication group if lab teardown | Admin Host | `Remove-DfsReplicationGroup -GroupName "<group>" -Force` | Replication group is removed |
| 11 | Preserve or manually reconcile data | FS1 / FS2 | Robocopy selected direction after deciding authoritative copy | Data remains protected |
| 12 | Clean DFSRPrivate only after DFSR is fully removed and data is backed up | FS1 / FS2 | Remove `DfsrPrivate` only in lab or approved cleanup | DFSR private data is removed safely |
| 13 | Force AD config polling | FS1 / FS2 | `dfsrdiag pollad`; `Update-DfsrConfigurationFromAD` | Members receive rollback config |
| 14 | Validate final DFSR state | Admin Host | `Get-DfsReplicationGroup`; `Get-DfsrMembership`; `Get-WinEvent -LogName "DFS Replication"` | DFSR state matches rollback target |
| 15 | Validate client path | Client | `dfsutil /pktflush`; `Test-Path "<dfsn-folder-path>"` | Client access follows intended target |
| 16 | Document rollback result | Operator | `Record removed targets, memberships, group state, content decision, and validation output` | Rollback record is complete |

# Configure_DFS_Replication_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| `New-DfsReplicationGroup` fails | DFSR tools missing | `Get-WindowsFeature FS-DFS-Replication`; `Get-Module DFSR -ListAvailable` | Install DFSR and management tools |
| DFSR service missing | DFS Replication role not installed | `Get-Service DFSR` | Install `FS-DFS-Replication` |
| Members do not receive config | AD replication delay or DFSR has not polled AD | `dfsrdiag pollad`; `Update-DfsrConfigurationFromAD` | Force AD poll and wait for AD replication |
| Backlog never decreases | Connection missing or disabled | `Get-DfsrConnection -GroupName "<group>"` | Create or enable connections |
| Backlog never decreases | RPC/firewall problem | `Test-NetConnection <member> -Port 5722` | Allow DFSR RPC/DFSR firewall traffic |
| Initial sync does not start | Primary member not set or membership incomplete | `Get-DfsrMembership -GroupName "<group>"` | Configure primary membership and content paths |
| Data conflict after initial sync | Both members had different changed data before replication | Conflict and Deleted folder, DFSR events | Choose authoritative data, restore/reconcile files |
| Users see inconsistent data through DFSN | DFSN has multiple targets before DFSR is healthy | `Get-DfsnFolderTarget`; backlog checks | Remove secondary DFSN target until DFSR is caught up |
| File does not replicate | File is open, locked, filtered, or unsupported workload | DFSR event log, file type check | Close file, avoid databases/PSTs/VHDX, or use another replication method |
| Large files replicate slowly | Staging quota too small or WAN limited | `Get-DfsrMembership`; backlog | Increase staging quota and review schedule/bandwidth |
| DFSR warning about staging cleanup | Staging quota undersized | DFS Replication log | Increase staging quota |
| DFSR warning about Conflict and Deleted | Conflict quota undersized or conflicts frequent | DFSR events and conflict folder | Increase quota and resolve conflict cause |
| DFSR database recovery events appear | Dirty shutdown, volume issue, or database recovery | DFSR event IDs and service state | Let recovery complete, verify volume health |
| DFSR journal wrap or USN issue | Long outage or NTFS journal problem | DFSR event log | Follow DFSR recovery process, consider reinitialization |
| `Get-DfsrBacklog` fails | Wrong group/folder/member names | `Get-DfsReplicationGroup`; `Get-DfsReplicatedFolder`; `Get-DfsrMember` | Use exact names |
| Pre-seed created duplicate/extra content | Robocopy destination wrong or `/MIR` misuse | Robocopy log and inventory | Stop, restore from backup, choose authoritative source |
| DFSR replicates deletes too quickly | DFSR is not a backup | Deleted files on both members | Restore from backup or shadow copy |
| DFSR report generation fails | Health report path missing or permissions issue | `Test-Path "<health-report-path>"` | Create path and rerun elevated |
| Event log missing | Wrong log or member queried | `Get-WinEvent -LogName "DFS Replication"` on each member | Query both members |
| Evidence missing | Validation skeleton not run | `Test-Path "<evidence-path>"` | Re-run precheck or post-change validation skeleton |

# Configure_DFS_Replication_Related_Labs
| Related Lab                                                         | Relationship                                                                    |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `00_File_Services_Index.md`                                         | Defines where DFS Replication fits in the File Services suite                   |
| `01_Install_File_Server_Role_And_Management_Tools.md`               | Provides File Server role and management baseline                               |
| `02_Prepare_File_Server_Storage_Volumes_And_Folders.md`             | Provides data volumes and folder paths replicated by DFSR                       |
| `03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`              | Provides ACLs that DFSR replicates with file data                               |
| `04_Create_SMB_Shares_And_Share_Permissions.md`                     | Provides backend shares used by DFSN and DFSR members                           |
| `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`     | Provides visibility behavior for replicated share targets                       |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`      | Secures SMB access to DFSR-backed shares                                        |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`           | Provides department and home data patterns that may be replicated               |
| `08_Configure_File_Screening_With_FSRM.md`                          | File screens may need matching configuration on replicated members              |
| `09_Configure_Storage_Quotas_With_FSRM.md`                          | Quotas may need matching configuration on replicated members                    |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md`    | Reports help assess replicated data size and stale data                         |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md`               | Provides local recovery on each member but does not replace backup              |
| `12_Backup_Restore_And_Export_File_Server_Config.md`                | Protects replicated data and exports DFSR configuration evidence                |
| `13_Configure_DFS_Namespaces.md`                                    | Provides user-facing DFS paths that can point to replicated targets             |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`      | Monitors usage and open files on replicated SMB targets                         |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`             | Helps isolate DFSR, DFSN, SMB, NTFS, DNS, and referral failures                 |
| `17_Configure_Data_Deduplication_For_File_Shares.md`                | Requires compatibility planning when used on replicated file data               |
| `25_Migrate_File_Server_Data_Shares_And_Permissions.md`             | DFSR can support staged migration, but backup and validation are still required |
| `26_Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance.md` | Deep troubleshooting workbook for DFSN/DFSR and SMB performance                 |