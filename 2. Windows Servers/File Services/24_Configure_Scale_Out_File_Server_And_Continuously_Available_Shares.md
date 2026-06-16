24_Configure_Scale_Out_File_Server_And_Continuously_Available_Shares.md
# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Index
24_Configure_Scale_Out_File_Server_And_Continuously_Available_Shares.md
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Source_Basis
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Mental_Model
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Planning_Table
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Configuration_Checklist
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Prereq_Validation_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Feature_Install_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Cluster_Validation_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Cluster_Creation_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_CSV_Preparation_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_SOFS_Role_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_CA_Share_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Application_ACL_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Client_Validation_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Failover_Test_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Status_And_Event_Review_Skeleton
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Verification_Commands
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Rollback
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Failure_Checks
Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Related_Labs

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Scale-Out File Server overview | Scale-out file shares | Active-active file shares from multiple cluster nodes |
| Microsoft Scale-Out File Server overview | When to use Scale-Out File Server | Workload fit for Hyper-V, SQL Server, IIS, and server applications |
| Microsoft Scale-Out File Server overview | Unsupported or not recommended workloads | Avoiding home folders, Offline Files, Work Folders, DFSR, FSRM, and metadata-heavy user shares |
| Microsoft Scale-Out File Server overview | SMB 3 support | SMB Transparent Failover, SMB Multichannel, SMB Direct, and continuous availability |
| Microsoft Deploy Scale-Out File Server | Configure Scale-Out File Server | Creating Scale-Out File Server clustered role |
| Microsoft Deploy Scale-Out File Server | Create continuously available file share | Creating SMB share on Cluster Shared Volume |
| Failover Clustering PowerShell | Test-Cluster, New-Cluster, Add-ClusterSharedVolume | Cluster validation, cluster creation, and CSV configuration |
| SMB Share PowerShell | New-SmbShare, Set-SmbPathAcl, Get-SmbShare | Creating and validating continuously available SMB shares |
| Operational file server practice | DR and application storage | Validating failover, client connection behavior, and app ACLs |

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Scale-Out File Server | Failover Cluster file server role designed for application data shares |
| SOFS name | Client access name used by applications to access the scale-out file server |
| Active-active shares | All cluster nodes can serve the same share at the same time |
| Continuously available share | SMB share setting that supports transparent failover for compatible SMB clients |
| Cluster Shared Volume | Shared cluster storage mounted at `C:\ClusterStorage\VolumeX` and accessible by all nodes |
| CSVFS | Cluster Shared Volume file system namespace used by SOFS workloads |
| Distributed Network Name | Cluster name model used by Scale-Out File Server |
| SMB Transparent Failover | SMB client capability that keeps application handles resilient during node failure |
| SMB Multichannel | SMB capability that uses multiple network paths when available |
| SMB Direct | SMB over RDMA for low-latency high-throughput file access |
| Coordinator node | Cluster node coordinating metadata for a CSV volume |
| Metadata-heavy workload | Workload with many open, close, create, rename, and directory operations |
| Application data share | Share intended for Hyper-V VHDX, SQL files, IIS app content, or similar server workload |
| Information worker share | Normal user department shares, home folders, redirected folders, and profile data |
| Blunt rule | Do not use SOFS as the default answer for normal user file shares |
| Design rule | SOFS is strongest when the workload is application data over SMB 3 and the backing storage can handle the combined node load |

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain | `corp.local` | `<domain-fqdn>` |
| Cluster name | `CLUS-FS01` | `<cluster-name>` |
| Cluster IP | `10.10.10.50` | `<cluster-ip>` |
| SOFS client access name | `SOFS01` | `<sofs-name>` |
| Node 1 | `FSNODE01` | `<node1-name>` |
| Node 2 | `FSNODE02` | `<node2-name>` |
| Optional node 3 | `FSNODE03` | `<node3-name>` |
| Cluster network | `10.10.10.0/24` | `<cluster-network>` |
| SMB client network | `10.10.20.0/24` | `<smb-client-network>` |
| Storage network | `10.10.30.0/24` | `<storage-network>` |
| CSV disk label | `CSV-AppData01` | `<csv-label>` |
| CSV path | `C:\ClusterStorage\Volume1` | `<csv-path>` |
| Share folder | `C:\ClusterStorage\Volume1\AppData` | `<share-folder-path>` |
| SMB share name | `AppData` | `<share-name>` |
| UNC path | `\\SOFS01\AppData` | `<unc-path>` |
| Workload type | Hyper-V / SQL / IIS / VMM Library | `<workload-type>` |
| CA share setting | Enabled | `True` |
| ABE setting | Disabled | `False` |
| Offline caching | Disabled | `None` |
| Encrypt data | Enabled / Disabled | `<encrypt-data-state>` |
| Access group | `CORP\GG_SOFS_AppData_Full` | `<access-group>` |
| App computer accounts | `CORP\HVHOST01$`, `CORP\SQL01$` | `<app-computer-accounts>` |
| Witness type | File Share Witness / Cloud Witness | `<witness-type>` |
| Witness path | `\\DC1\ClusterWitness$\CLUS-FS01` | `<witness-path>` |
| Validation report path | `C:\Cluster-Reports` | `<cluster-report-path>` |
| Test client | `APP01` | `<test-client>` |
| Report path | `C:\SOFS-Reports` | `<report-path>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm node identities | Each Node | `hostname` | Correct cluster nodes are identified |
| 2 | Confirm domain membership | Each Node | `(Get-CimInstance Win32_ComputerSystem).Domain` | All nodes are joined to expected domain |
| 3 | Confirm OS version | Each Node | `Get-ComputerInfo | Select WindowsProductName,WindowsVersion,OsName` | Supported Windows Server version is visible |
| 4 | Confirm node IP configuration | Each Node | `Get-NetIPConfiguration` | Management, SMB, and storage network configuration is visible |
| 5 | Confirm DNS resolution between nodes | Each Node | `Resolve-DnsName FSNODE01; Resolve-DnsName FSNODE02` | Nodes resolve each other |
| 6 | Confirm time sync | Each Node | `w32tm /query /status` | Domain time sync is healthy |
| 7 | Confirm local admin rights | Management Host | `whoami /groups` | Operator has required admin rights |
| 8 | Confirm shared storage visibility | Each Node | `Get-Disk` | Intended cluster disks are visible to all nodes |
| 9 | Confirm disk sector size | Each Node | `Get-Disk | Select Number,FriendlyName,LogicalSectorSize,PhysicalSectorSize` | Sector sizes are consistent |
| 10 | Confirm storage is not already in use | Each Node | `Get-Volume` | Intended disk is not hosting unrelated data |
| 11 | Install File Server role | Each Node | `Install-WindowsFeature FS-FileServer` | File Server role is installed |
| 12 | Install Failover Clustering | Each Node | `Install-WindowsFeature Failover-Clustering -IncludeManagementTools` | Failover clustering tools are installed |
| 13 | Confirm cluster cmdlets | Each Node | `Get-Command -Module FailoverClusters` | Cluster cmdlets are available |
| 14 | Enable required firewall rules | Each Node | `Enable-NetFirewallRule -DisplayGroup "Failover Clusters"; Enable-NetFirewallRule -DisplayGroup "File and Printer Sharing"` | Cluster and SMB firewall rules are enabled |
| 15 | Validate cluster topology | Management Host | `Test-Cluster -Node FSNODE01,FSNODE02 -Include "Storage","Inventory","Network","System Configuration" -ReportName C:\Cluster-Reports\CLUS-FS01-Validation.html` | Validation report is generated |
| 16 | Review validation report | Operator | `Open C:\Cluster-Reports\CLUS-FS01-Validation.html` | Blocking issues are resolved before cluster creation |
| 17 | Create failover cluster | Management Host | `New-Cluster -Name CLUS-FS01 -Node FSNODE01,FSNODE02 -StaticAddress 10.10.10.50 -NoStorage` | Cluster is created |
| 18 | Confirm cluster state | Management Host | `Get-Cluster -Name CLUS-FS01` | Cluster object is visible |
| 19 | Configure quorum witness | Management Host | `Set-ClusterQuorum -Cluster CLUS-FS01 -FileShareWitness "\\DC1\ClusterWitness$\CLUS-FS01"` | Quorum witness is configured |
| 20 | Add available disks to cluster | Management Host | `Get-ClusterAvailableDisk -Cluster CLUS-FS01 | Add-ClusterDisk` | Shared disks become cluster disks |
| 21 | Convert data disk to CSV | Management Host | `Add-ClusterSharedVolume -Name "Cluster Disk 1" -Cluster CLUS-FS01` | Disk becomes a Cluster Shared Volume |
| 22 | Confirm CSV state | Management Host | `Get-ClusterSharedVolume -Cluster CLUS-FS01` | CSV is online |
| 23 | Confirm CSV path | Cluster Node | `Get-ChildItem C:\ClusterStorage` | CSV path appears |
| 24 | Create SOFS role | Management Host | `Add-ClusterScaleOutFileServerRole -Cluster CLUS-FS01 -Name SOFS01` | Scale-Out File Server role is created |
| 25 | Confirm SOFS role | Management Host | `Get-ClusterGroup -Cluster CLUS-FS01 | Where-Object Name -like "*SOFS*"` | SOFS role is online |
| 26 | Confirm SOFS DNS name | Client or Management Host | `Resolve-DnsName SOFS01` | SOFS client access name resolves |
| 27 | Create share folder on CSV | Cluster Node | `New-Item -ItemType Directory -Path "C:\ClusterStorage\Volume1\AppData" -Force` | Share folder exists on CSV |
| 28 | Set NTFS ACL baseline | Cluster Node | `icacls "C:\ClusterStorage\Volume1\AppData" /grant "CORP\GG_SOFS_AppData_Full:(OI)(CI)F"` | Application access group has NTFS rights |
| 29 | Create continuously available SMB share | Cluster Node | `New-SmbShare -Name "AppData" -Path "C:\ClusterStorage\Volume1\AppData" -FullAccess "CORP\GG_SOFS_AppData_Full" -ContinuouslyAvailable $true -CachingMode None` | CA SMB share is created |
| 30 | Apply share ACL to path | Cluster Node | `Set-SmbPathAcl -ShareName "AppData"` | File system ACL is aligned with share permissions where applicable |
| 31 | Confirm share state | Cluster Node | `Get-SmbShare -Name "AppData" | Select Name,Path,ContinuouslyAvailable,CachingMode,ScopeName` | Share is continuously available and scoped to SOFS |
| 32 | Confirm share access | Cluster Node | `Get-SmbShareAccess -Name "AppData"` | Expected application accounts or groups have access |
| 33 | Confirm client SMB reachability | Test Client | `Test-NetConnection SOFS01 -Port 445` | TCP 445 succeeds |
| 34 | Confirm UNC path access | Test Client | `Test-Path "\\SOFS01\AppData"` | SOFS share is reachable |
| 35 | Write test file from client | Test Client | `"sofs-test" | Out-File "\\SOFS01\AppData\sofs-test.txt"` | Test file writes successfully |
| 36 | Confirm SMB connection details | Test Client | `Get-SmbConnection | Where-Object ServerName -like "*SOFS01*"` | SMB dialect and CA state are visible if exposed by client |
| 37 | Confirm SMB sessions across nodes | Cluster Nodes | `Get-SmbSession` | Client sessions are visible on servicing nodes |
| 38 | Move CSV owner for failover test | Management Host | `Move-ClusterSharedVolume -Name "Cluster Disk 1" -Node FSNODE02 -Cluster CLUS-FS01` | CSV ownership changes |
| 39 | Move SOFS role if needed | Management Host | `Move-ClusterGroup -Name "SOFS01" -Node FSNODE02 -Cluster CLUS-FS01` | SOFS group remains online |
| 40 | Retest client access after movement | Test Client | `Test-Path "\\SOFS01\AppData"` | Client access survives cluster movement |
| 41 | Review cluster events | Management Host | `Get-WinEvent -LogName System -MaxEvents 100 | Where-Object Message -like "*cluster*"` | Recent cluster events are visible |
| 42 | Review SMB server events | Cluster Nodes | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 100` | SMB operational events are visible |
| 43 | Export SOFS evidence | Management Host | `Get-ClusterGroup -Cluster CLUS-FS01 | Export-Clixml "C:\SOFS-Reports\cluster-groups.xml"` | Configuration evidence is saved |
| 44 | Document final state | Operator | `Record cluster, nodes, witness, CSV, SOFS name, share, ACLs, validation, and failover test` | Workbook evidence is complete |

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Prereq_Validation_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: validate nodes before building a SOFS cluster.

$Nodes = @("FSNODE01", "FSNODE02")
$DomainFqdn = "corp.local"
$ReportRoot = "C:\SOFS-Reports"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

foreach ($Node in $Nodes) {
  Write-Host "===== Validating $Node ====="

  Invoke-Command -ComputerName $Node -ScriptBlock {
    Write-Host "Identity:"
    hostname

    Write-Host "Domain membership:"
    Get-CimInstance Win32_ComputerSystem |
      Select-Object Name, Domain, PartOfDomain

    Write-Host "Operating system:"
    Get-ComputerInfo |
      Select-Object WindowsProductName, WindowsVersion, OsName

    Write-Host "Network configuration:"
    Get-NetIPConfiguration

    Write-Host "Disks:"
    Get-Disk |
      Select-Object Number, FriendlyName, OperationalStatus, PartitionStyle, Size, LogicalSectorSize, PhysicalSectorSize |
      Format-Table -AutoSize

    Write-Host "Volumes:"
    Get-Volume |
      Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, Size, SizeRemaining |
      Format-Table -AutoSize

    Write-Host "Time sync:"
    w32tm /query /status
  }
}

Write-Host "Domain controller discovery:"
nltest /dsgetdc:$DomainFqdn

Write-Host "Testing node name resolution:"
foreach ($Node in $Nodes) {
  Resolve-DnsName $Node
}
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Feature_Install_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: install Failover Clustering and File Server features on all SOFS nodes.

$Nodes = @("FSNODE01", "FSNODE02")

foreach ($Node in $Nodes) {
  Write-Host "===== Installing features on $Node ====="

  Invoke-Command -ComputerName $Node -ScriptBlock {
    Install-WindowsFeature `
      -Name FS-FileServer, Failover-Clustering `
      -IncludeManagementTools

    Get-WindowsFeature FS-FileServer, Failover-Clustering |
      Select-Object Name, DisplayName, InstallState |
      Format-Table -AutoSize

    Get-Command -Module FailoverClusters |
      Select-Object Name |
      Sort-Object Name |
      Select-Object -First 10

    Get-Command -Module SmbShare |
      Select-Object Name |
      Sort-Object Name |
      Select-Object -First 10
  }
}

Write-Host "Restart nodes if required:"
Write-Host "Restart-Computer -ComputerName FSNODE01,FSNODE02 -Force"
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Cluster_Validation_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: run Microsoft cluster validation before creating the cluster.

$Nodes = @("FSNODE01", "FSNODE02")
$ReportRoot = "C:\Cluster-Reports"
$ReportName = "CLUS-FS01-Validation.html"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Running cluster validation..."
Test-Cluster `
  -Node $Nodes `
  -Include "Storage","Inventory","Network","System Configuration" `
  -ReportName (Join-Path $ReportRoot $ReportName)

Write-Host "Cluster validation report path:"
Get-ChildItem $ReportRoot -Filter "*.html" |
  Sort-Object LastWriteTime -Descending |
  Select-Object FullName, LastWriteTime |
  Format-Table -AutoSize

Write-Host "Do not create the cluster until blocking validation failures are corrected."
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Cluster_Creation_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: create the failover cluster and configure quorum.

$ClusterName = "CLUS-FS01"
$ClusterIP = "10.10.10.50"
$Nodes = @("FSNODE01", "FSNODE02")
$WitnessPath = "\\DC1\ClusterWitness$\CLUS-FS01"

Write-Host "Creating failover cluster without automatically adding storage..."
New-Cluster `
  -Name $ClusterName `
  -Node $Nodes `
  -StaticAddress $ClusterIP `
  -NoStorage

Write-Host "Confirm cluster state:"
Get-Cluster -Name $ClusterName | Format-List *

Write-Host "Configuring file share witness..."
Set-ClusterQuorum `
  -Cluster $ClusterName `
  -FileShareWitness $WitnessPath

Write-Host "Cluster quorum:"
Get-ClusterQuorum -Cluster $ClusterName

Write-Host "Cluster nodes:"
Get-ClusterNode -Cluster $ClusterName |
  Format-Table Name, State, NodeWeight -AutoSize

Write-Host "Cluster networks:"
Get-ClusterNetwork -Cluster $ClusterName |
  Format-Table Name, Address, Role, State -AutoSize
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_CSV_Preparation_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: add available shared disk to the cluster and convert it to CSV.

$ClusterName = "CLUS-FS01"
$CsvDiskName = "Cluster Disk 1"

Write-Host "Available cluster disks:"
Get-ClusterAvailableDisk -Cluster $ClusterName |
  Format-Table Name, Size, Number -AutoSize

Write-Host "Adding available disks to the cluster..."
Get-ClusterAvailableDisk -Cluster $ClusterName |
  Add-ClusterDisk

Write-Host "Cluster disks after add:"
Get-ClusterResource -Cluster $ClusterName |
  Where-Object ResourceType -eq "Physical Disk" |
  Format-Table Name, State, OwnerGroup, OwnerNode -AutoSize

Write-Host "Converting selected disk to Cluster Shared Volume..."
Add-ClusterSharedVolume `
  -Cluster $ClusterName `
  -Name $CsvDiskName

Write-Host "CSV state:"
Get-ClusterSharedVolume -Cluster $ClusterName |
  Format-Table Name, State, OwnerNode -AutoSize

Write-Host "CSV local paths:"
Invoke-Command -ComputerName "FSNODE01" -ScriptBlock {
  Get-ChildItem C:\ClusterStorage
}
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_SOFS_Role_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: create the Scale-Out File Server clustered role.

$ClusterName = "CLUS-FS01"
$SofsName = "SOFS01"

Write-Host "Creating Scale-Out File Server role..."
Add-ClusterScaleOutFileServerRole `
  -Cluster $ClusterName `
  -Name $SofsName

Write-Host "Cluster groups:"
Get-ClusterGroup -Cluster $ClusterName |
  Format-Table Name, State, OwnerNode -AutoSize

Write-Host "SOFS resources:"
Get-ClusterResource -Cluster $ClusterName |
  Where-Object OwnerGroup -eq $SofsName |
  Format-Table Name, ResourceType, State, OwnerGroup, OwnerNode -AutoSize

Write-Host "DNS resolution for SOFS name:"
Resolve-DnsName $SofsName -ErrorAction Continue
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_CA_Share_Skeleton
```powershell
# Run from elevated PowerShell on one cluster node.
# Purpose: create a continuously available SMB share on a CSV path.

$ShareName = "AppData"
$CsvPath = "C:\ClusterStorage\Volume1"
$SharePath = Join-Path $CsvPath $ShareName
$FullAccessGroup = "CORP\GG_SOFS_AppData_Full"

Write-Host "Creating share folder on CSV..."
New-Item -ItemType Directory -Force -Path $SharePath | Out-Null

Write-Host "Setting NTFS ACL baseline..."
icacls $SharePath /grant "$FullAccessGroup:(OI)(CI)F"

Write-Host "Creating continuously available SMB share..."
New-SmbShare `
  -Name $ShareName `
  -Path $SharePath `
  -FullAccess $FullAccessGroup `
  -ContinuouslyAvailable $true `
  -CachingMode None

Write-Host "Applying SMB share ACL to NTFS path where applicable..."
Set-SmbPathAcl -ShareName $ShareName

Write-Host "Share state:"
Get-SmbShare -Name $ShareName |
  Select-Object Name, ScopeName, Path, ContinuouslyAvailable, CachingMode, EncryptData, FolderEnumerationMode |
  Format-List

Write-Host "Share permissions:"
Get-SmbShareAccess -Name $ShareName |
  Format-Table Name, AccountName, AccessControlType, AccessRight -AutoSize
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Application_ACL_Skeleton
```powershell
# Run from elevated PowerShell on a cluster node.
# Purpose: grant application servers or service accounts access to the SOFS application data share.

$ShareName = "AppData"
$SharePath = "C:\ClusterStorage\Volume1\AppData"

$Principals = @(
  "CORP\HVHOST01$",
  "CORP\HVHOST02$",
  "CORP\SQL01$",
  "CORP\GG_SOFS_AppData_Admins"
)

foreach ($Principal in $Principals) {
  Write-Host "Granting Full Control on SMB share to $Principal"
  Grant-SmbShareAccess `
    -Name $ShareName `
    -AccountName $Principal `
    -AccessRight Full `
    -Force

  Write-Host "Granting Full Control on NTFS path to $Principal"
  icacls $SharePath /grant "$Principal:(OI)(CI)F"
}

Write-Host "Final SMB share access:"
Get-SmbShareAccess -Name $ShareName |
  Format-Table Name, AccountName, AccessControlType, AccessRight -AutoSize

Write-Host "Final NTFS ACL:"
icacls $SharePath
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Client_Validation_Skeleton
```powershell
# Run on a workload server or test client.
# Purpose: validate SOFS UNC access and SMB connection properties.

$SofsName = "SOFS01"
$ShareName = "AppData"
$UncPath = "\\$SofsName\$ShareName"
$TestFile = Join-Path $UncPath "sofs-client-test.txt"

Write-Host "Client identity:"
hostname
whoami

Write-Host "DNS resolution:"
Resolve-DnsName $SofsName

Write-Host "SMB port test:"
Test-NetConnection $SofsName -Port 445

Write-Host "UNC path test:"
Test-Path $UncPath

Write-Host "Write test file:"
"SOFS test from $env:COMPUTERNAME by $env:USERNAME at $(Get-Date)" |
  Out-File -FilePath $TestFile -Encoding utf8

Write-Host "Read test file:"
Get-Content $TestFile

Write-Host "SMB connection details:"
Get-SmbConnection |
  Where-Object ServerName -like "*$SofsName*" |
  Select-Object ServerName, ShareName, Dialect, NumOpens, Signed, Encrypted, ContinuouslyAvailable |
  Format-Table -AutoSize
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Failover_Test_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: test SOFS behavior during planned cluster movement.

$ClusterName = "CLUS-FS01"
$SofsName = "SOFS01"
$CsvName = "Cluster Disk 1"
$TargetNode = "FSNODE02"

Write-Host "Cluster groups before movement:"
Get-ClusterGroup -Cluster $ClusterName |
  Format-Table Name, State, OwnerNode -AutoSize

Write-Host "CSV state before movement:"
Get-ClusterSharedVolume -Cluster $ClusterName |
  Format-Table Name, State, OwnerNode -AutoSize

Write-Host "Moving CSV owner to target node..."
Move-ClusterSharedVolume `
  -Cluster $ClusterName `
  -Name $CsvName `
  -Node $TargetNode

Write-Host "Moving SOFS group to target node if needed..."
Move-ClusterGroup `
  -Cluster $ClusterName `
  -Name $SofsName `
  -Node $TargetNode

Write-Host "Cluster groups after movement:"
Get-ClusterGroup -Cluster $ClusterName |
  Format-Table Name, State, OwnerNode -AutoSize

Write-Host "CSV state after movement:"
Get-ClusterSharedVolume -Cluster $ClusterName |
  Format-Table Name, State, OwnerNode -AutoSize

Write-Host "Now retest from client:"
Write-Host "Test-Path \\$SofsName\AppData"
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Status_And_Event_Review_Skeleton
```powershell
# Run from elevated PowerShell on a management host.
# Purpose: export SOFS, cluster, SMB, and event evidence.

$ClusterName = "CLUS-FS01"
$SofsName = "SOFS01"
$ReportRoot = "C:\SOFS-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$Nodes = @("FSNODE01", "FSNODE02")

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Export cluster groups..."
Get-ClusterGroup -Cluster $ClusterName |
  Export-Clixml "$ReportRoot\cluster-groups-$Timestamp.xml"

Write-Host "Export cluster resources..."
Get-ClusterResource -Cluster $ClusterName |
  Export-Clixml "$ReportRoot\cluster-resources-$Timestamp.xml"

Write-Host "Export CSV state..."
Get-ClusterSharedVolume -Cluster $ClusterName |
  Export-Clixml "$ReportRoot\cluster-csv-$Timestamp.xml"

Write-Host "Export SMB shares from each node..."
foreach ($Node in $Nodes) {
  Invoke-Command -ComputerName $Node -ArgumentList $ReportRoot, $Timestamp -ScriptBlock {
    param($ReportRoot, $Timestamp)

    Get-SmbShare |
      Select-Object Name, ScopeName, Path, ContinuouslyAvailable, CachingMode, EncryptData |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-smb-shares-$Timestamp.csv" -NoTypeInformation

    Get-SmbSession -ErrorAction SilentlyContinue |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-smb-sessions-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-smbserver-events-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -LogName System -MaxEvents 300 -ErrorAction SilentlyContinue |
      Where-Object Message -like "*cluster*" |
      Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-cluster-system-events-$Timestamp.csv" -NoTypeInformation
  }
}

Write-Host "SOFS report data exported to $ReportRoot"
```

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify File Server feature | `Get-WindowsFeature FS-FileServer` | Installed on every node |
| Verify Failover Clustering feature | `Get-WindowsFeature Failover-Clustering` | Installed on every node |
| Verify cluster validation | `Test-Cluster -Node FSNODE01,FSNODE02` | Validation completes without blocking failures |
| Verify cluster exists | `Get-Cluster -Name CLUS-FS01` | Cluster is visible |
| Verify cluster nodes | `Get-ClusterNode -Cluster CLUS-FS01` | Nodes are Up |
| Verify quorum | `Get-ClusterQuorum -Cluster CLUS-FS01` | Witness is configured |
| Verify cluster networks | `Get-ClusterNetwork -Cluster CLUS-FS01` | Networks are online and properly scoped |
| Verify CSV | `Get-ClusterSharedVolume -Cluster CLUS-FS01` | CSV is online |
| Verify CSV path | `Get-ChildItem C:\ClusterStorage` | CSV path is visible |
| Verify SOFS role | `Get-ClusterGroup -Cluster CLUS-FS01 | Where-Object Name -eq "SOFS01"` | SOFS role is online |
| Verify SOFS DNS | `Resolve-DnsName SOFS01` | SOFS client access name resolves |
| Verify SMB share | `Get-SmbShare -Name "AppData"` | Share exists |
| Verify CA flag | `Get-SmbShare -Name "AppData" | Select Name,ContinuouslyAvailable,CachingMode` | `ContinuouslyAvailable` is True and caching is None |
| Verify SMB permissions | `Get-SmbShareAccess -Name "AppData"` | Correct app principals have access |
| Verify NTFS ACL | `icacls "C:\ClusterStorage\Volume1\AppData"` | Correct app principals have NTFS rights |
| Verify client SMB port | `Test-NetConnection SOFS01 -Port 445` | TCP 445 succeeds |
| Verify client UNC path | `Test-Path "\\SOFS01\AppData"` | Client can access the share |
| Verify client SMB connection | `Get-SmbConnection | Where-Object ServerName -like "*SOFS01*"` | SMB connection is visible |
| Verify SMB sessions | `Get-SmbSession` on nodes | Sessions are visible |
| Verify failover movement | `Move-ClusterSharedVolume -Name "Cluster Disk 1" -Node FSNODE02` | CSV moves and share remains accessible |
| Verify events | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 50` | SMB operational events return |

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| Test file created | `Remove-Item "\\SOFS01\AppData\sofs-client-test.txt" -Force` | Test file is removed |
| SMB share created | `Remove-SmbShare -Name "AppData" -Force` | Continuously available share is removed |
| SMB access granted | `Revoke-SmbShareAccess -Name "AppData" -AccountName "<principal>" -Force` | Unwanted share permission is removed |
| NTFS access granted | `icacls "C:\ClusterStorage\Volume1\AppData" /remove "<principal>"` | Unwanted NTFS permission is removed |
| Share folder created | `Remove-Item "C:\ClusterStorage\Volume1\AppData" -Recurse -Force` | Folder is removed only after data is disposable |
| SOFS role created | `Remove-ClusterGroup -Cluster CLUS-FS01 -Name "SOFS01" -RemoveResources -Force` | SOFS role is removed |
| CSV added | `Remove-ClusterSharedVolume -Name "Cluster Disk 1"` | Disk is removed from CSV use |
| Disk added to cluster | `Remove-ClusterResource -Name "Cluster Disk 1" -Force` | Disk is removed from cluster resources |
| Cluster quorum witness configured | `Set-ClusterQuorum -Cluster CLUS-FS01 -NodeMajority` | Quorum returns to node majority if appropriate |
| Cluster created | `Remove-Cluster -Cluster CLUS-FS01 -Force -CleanupAD` | Cluster is removed and AD object cleanup is attempted |
| Failover Clustering feature installed | `Uninstall-WindowsFeature Failover-Clustering` | Feature is removed after cluster removal |
| File Server role installed only for lab | `Uninstall-WindowsFeature FS-FileServer` | File Server role is removed if not needed |
| Report folder created | `Remove-Item "C:\SOFS-Reports" -Recurse -Force` | Report folder is removed |
| Validation report folder created | `Remove-Item "C:\Cluster-Reports" -Recurse -Force` | Cluster report folder is removed |

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `Test-Cluster` fails storage validation | Shared storage not visible or not suitable | Validation report and `Get-Disk` on each node | Fix storage presentation before cluster creation |
| `New-Cluster` fails | DNS, AD permissions, firewall, node mismatch, or stale cluster object | `Resolve-DnsName`; AD computer object; validation report | Fix prerequisite and retry |
| Cluster name does not resolve | CNO or DNS issue | `Resolve-DnsName CLUS-FS01` | Check AD computer object and DNS registration |
| SOFS name does not resolve | Distributed network name registration issue | `Resolve-DnsName SOFS01`; cluster resources | Bring SOFS role online and verify DNS |
| CSV path missing | Disk not converted to CSV or wrong node | `Get-ClusterSharedVolume`; `Get-ChildItem C:\ClusterStorage` | Add disk to CSV and verify online state |
| `New-SmbShare` fails on CSV path | Path missing, permission issue, or not running on cluster node | `Test-Path <csv-path>`; `Get-ClusterSharedVolume` | Create path and run from proper node |
| Share not continuously available | Share created without CA flag or wrong share type | `Get-SmbShare -Name <share> | Select ContinuouslyAvailable` | Recreate or modify share with CA behavior |
| Client cannot access share | DNS, TCP 445, ACL, or cluster role offline | `Resolve-DnsName`; `Test-NetConnection`; `Get-SmbShareAccess` | Fix name, network, permission, or SOFS role |
| User workload performs poorly | SOFS used for metadata-heavy user data | Workload review and SMB counters | Move user share to general-purpose file server cluster |
| Access-based enumeration enabled | ABE not appropriate for SOFS workload | `Get-SmbShare | Select FolderEnumerationMode` | Disable ABE for SOFS application shares |
| Offline caching enabled | Client caching not appropriate for SOFS app share | `Get-SmbShare | Select CachingMode` | Set caching mode to None |
| FSRM quota or screen expected | FSRM is not supported with SOFS workload model | FSRM config and SOFS compatibility review | Use general file server cluster for FSRM workloads |
| DFSR expected | DFSR is not compatible with SOFS workload model | DFSR memberships | Do not use DFSR on SOFS share data |
| Home folders planned on SOFS | Wrong workload fit | Workload review | Use general file server cluster or standalone file server |
| Hyper-V host access denied | Computer account missing share or NTFS rights | `Get-SmbShareAccess`; `icacls` | Grant Hyper-V host computer accounts or group Full Control |
| SQL access denied | SQL service account missing rights | `Get-SmbShareAccess`; `icacls` | Grant SQL service account Full Control |
| Failover causes client interruption | Client not SMB 3 capable, workload not CA-aware, or share not CA | `Get-SmbConnection`; client OS; share CA state | Use supported SMB 3 clients and CA share |
| SMB Multichannel not used | Network paths not suitable or disabled | `Get-SmbMultichannelConnection` | Fix NIC, RSS, RDMA, or SMB configuration |
| SMB Direct not used | RDMA not present or disabled | `Get-SmbClientNetworkInterface`; `Get-NetAdapterRdma` | Enable and validate RDMA path |
| Cluster validation warning ignored | Real hardware, network, or storage problem | Validation report | Fix warning before production use |
| CSV owner movement disrupts performance | CSV redirection or network issue | CSV state and SMB counters | Fix cluster network and storage path |

# Configure_Scale_Out_File_Server_And_Continuously_Available_Shares_Related_Labs
| Lab                                                                 | Relationship                                                                                          |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `01_Install_File_Server_Role_And_Management_Tools.md`               | Establishes the Windows file server role baseline                                                     |
| `02_Create_And_Publish_SMB_Shares.md`                               | SOFS shares are still SMB shares, but clustered and continuously available                            |
| `03_Configure_NTFS_And_Share_Permissions.md`                        | Application accounts need matching SMB and NTFS rights                                                |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`      | SMB security posture affects SOFS client connections                                                  |
| `12_Backup_Restore_And_Export_File_Server_Config.md`                | SOFS configuration and clustered shares need recovery documentation                                   |
| `13_Configure_DFS_Namespaces.md`                                    | DFS namespaces can front some clustered share designs, but DFSR is not SOFS replication               |
| `14_Configure_DFS_Replication.md`                                   | DFSR is not the mechanism for SOFS data availability                                                  |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`      | SMB monitoring validates application connections to SOFS                                              |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`             | Troubleshooting covers DNS, SMB, ACL, and client access failures                                      |
| `17_Configure_Data_Deduplication_For_File_Shares.md`                | Deduplication support must be checked carefully before using with SOFS workloads                      |
| `19_Configure_BranchCache_For_File_Services.md`                     | BranchCache is not a SOFS workload fit                                                                |
| `21_Configure_Work_Folders.md`                                      | Work Folders is not supported as a SOFS workload pattern                                              |
| `23_Configure_Storage_Replica_For_File_Server_Disaster_Recovery.md` | Storage Replica provides DR replication, while SOFS provides clustered application share availability |