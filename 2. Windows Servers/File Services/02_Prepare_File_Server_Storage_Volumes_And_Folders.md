02_Prepare_File_Server_Storage_Volumes_And_Folders.md
# Prepare_File_Server_Storage_Volumes_And_Folders

# Prepare_File_Server_Storage_Volumes_And_Folders_Index
02_Prepare_File_Server_Storage_Volumes_And_Folders.md
Prepare_File_Server_Storage_Volumes_And_Folders
Prepare_File_Server_Storage_Volumes_And_Folders_Source_Basis
Prepare_File_Server_Storage_Volumes_And_Folders_Mental_Model
Prepare_File_Server_Storage_Volumes_And_Folders_Planning_Table
Prepare_File_Server_Storage_Volumes_And_Folders_Configuration_Checklist
Prepare_File_Server_Storage_Volumes_And_Folders_Precheck_Skeleton
Prepare_File_Server_Storage_Volumes_And_Folders_Disk_Initialize_And_Format_Skeleton
Prepare_File_Server_Storage_Volumes_And_Folders_Folder_Structure_Skeleton
Prepare_File_Server_Storage_Volumes_And_Folders_Share_Root_Validation_Skeleton
Prepare_File_Server_Storage_Volumes_And_Folders_Post_Build_Validation_Skeleton
Prepare_File_Server_Storage_Volumes_And_Folders_Verification_Commands
Prepare_File_Server_Storage_Volumes_And_Folders_Rollback
Prepare_File_Server_Storage_Volumes_And_Folders_Failure_Checks
Prepare_File_Server_Storage_Volumes_And_Folders_Related_Labs

# Prepare_File_Server_Storage_Volumes_And_Folders_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Get-Disk | Discovering physical and virtual disks available to Windows Server |
| Microsoft Learn | Initialize-Disk | Initializing a new data disk before volume creation |
| Microsoft Learn | New-Partition | Creating a partition and assigning a drive letter |
| Microsoft Learn | Format-Volume | Formatting a volume with NTFS or ReFS |
| Microsoft Learn | Get-Volume | Validating volume label, file system, size, and health |
| Microsoft Learn | New-Item | Creating file server folder structure |
| Microsoft Learn | Get-Acl | Capturing default ACL state before permission design |
| Microsoft Learn | Test-Path | Confirming required folder paths exist |
| Microsoft Learn | File and Storage Services | Managing disks, volumes, shares, and storage layout |
| Windows Server operational practice | File server storage design | Separating OS volume from data volume and creating predictable share roots |

# Prepare_File_Server_Storage_Volumes_And_Folders_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OS volume | System volume that should not host production share data |
| Data volume | Dedicated volume used for shared folders, department data, home folders, and file service content |
| Disk number | Windows identifier for a disk. Must be verified before initialization or formatting |
| Partition style | GPT is the normal modern choice for Windows Server data disks |
| File system | NTFS is the default for general Windows file shares. ReFS is workload-specific |
| Allocation unit size | Cluster size used by the file system. Default is fine for most lab and general file server shares |
| Volume label | Human-readable label such as `FileData` that identifies the purpose of the volume |
| Drive letter | Stable path such as `E:` used by scripts, shares, backup, and documentation |
| Mount point | Folder path used instead of or alongside a drive letter for volume access |
| Share root | Top-level folder where user-facing share folders are created |
| Department root | Folder container for department shares |
| Home folder root | Folder container for user home directories |
| App share root | Folder container for application installers, packages, or shared tools |
| Staging folder | Temporary area used during migration, restore, testing, or Robocopy operations |
| Evidence folder | Local path where validation outputs are saved |
| Permission boundary | Folder level where later NTFS ACL design will begin |
| First rule | Never initialize, clean, format, or repartition a disk until disk number, size, bus type, and current partition state are verified |

# Prepare_File_Server_Storage_Volumes_And_Folders_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Admin account | `CORP\Administrator` | `<admin-account>` |
| Target disk number | `1` | `<target-disk-number>` |
| Target disk size | `200 GB` | `<target-disk-size>` |
| Target disk bus type | `SCSI` | `<target-bus-type>` |
| Partition style | `GPT` | `<partition-style>` |
| Data drive letter | `E` | `<data-drive-letter>` |
| Data volume path | `E:\` | `<data-volume-path>` |
| File system | `NTFS` | `<file-system>` |
| Allocation unit size | Default | `<allocation-unit-size>` |
| Volume label | `FileData` | `<volume-label>` |
| Share root | `E:\Shares` | `<share-root>` |
| Department root | `E:\Shares\Departments` | `<department-root>` |
| Home folder root | `E:\Shares\Home` | `<home-folder-root>` |
| Public share root | `E:\Shares\Public` | `<public-share-root>` |
| App share root | `E:\Shares\Apps` | `<app-share-root>` |
| Profiles root if used | `E:\Shares\Profiles` | `<profiles-root>` |
| Redirected folders root if used | `E:\Shares\RedirectedFolders` | `<redirected-folders-root>` |
| DFS staging root | `E:\DFSRoots` | `<dfs-root-folder>` |
| FSRM report path | `E:\FSRM-Reports` | `<fsrm-report-path>` |
| Migration staging path | `E:\Staging` | `<staging-path>` |
| Backup staging path | `E:\Backup-Staging` | `<backup-staging-path>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Folder naming standard | Singular roots, department names below | `<folder-naming-standard>` |
| ACL stance in this workbook | Capture only, configure in task 03 | `<acl-plan>` |
| Next workbook | `03_Configure_NTFS_Permissions_And_ACL_Inheritance.md` | `<next-task>` |

# Prepare_File_Server_Storage_Volumes_And_Folders_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm server identity | File Server | `hostname` | Server name matches plan |
| 3 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 4 | Confirm current disks | File Server | `Get-Disk` | Disk list is visible |
| 5 | Confirm current volumes | File Server | `Get-Volume` | Existing volumes are visible |
| 6 | Identify target data disk | File Server | `Get-Disk -Number <target-disk-number>` | Correct disk number, size, and state are confirmed |
| 7 | Confirm target disk has no required data | File Server | `Get-Partition -DiskNumber <target-disk-number>` | Existing partition state is known |
| 8 | Initialize disk if raw | File Server | `Initialize-Disk -Number <target-disk-number> -PartitionStyle GPT` | Raw disk becomes initialized |
| 9 | Create data partition | File Server | `New-Partition -DiskNumber <target-disk-number> -UseMaximumSize -DriveLetter <data-drive-letter>` | Partition is created with target drive letter |
| 10 | Format data volume | File Server | `Format-Volume -DriveLetter <data-drive-letter> -FileSystem NTFS -NewFileSystemLabel "<volume-label>"` | Volume is formatted and labeled |
| 11 | Confirm volume health | File Server | `Get-Volume -DriveLetter <data-drive-letter>` | Volume shows healthy state |
| 12 | Create share root | File Server | `New-Item -ItemType Directory -Path "<share-root>" -Force` | Share root exists |
| 13 | Create department root | File Server | `New-Item -ItemType Directory -Path "<department-root>" -Force` | Department root exists |
| 14 | Create home folder root | File Server | `New-Item -ItemType Directory -Path "<home-folder-root>" -Force` | Home folder root exists |
| 15 | Create public root | File Server | `New-Item -ItemType Directory -Path "<public-share-root>" -Force` | Public root exists |
| 16 | Create apps root | File Server | `New-Item -ItemType Directory -Path "<app-share-root>" -Force` | Apps root exists |
| 17 | Create optional profiles root | File Server | `New-Item -ItemType Directory -Path "<profiles-root>" -Force` | Profiles root exists if used |
| 18 | Create optional redirected folders root | File Server | `New-Item -ItemType Directory -Path "<redirected-folders-root>" -Force` | Redirected folders root exists if used |
| 19 | Create DFS root folder if planned | File Server | `New-Item -ItemType Directory -Path "<dfs-root-folder>" -Force` | DFS root folder exists |
| 20 | Create FSRM report path | File Server | `New-Item -ItemType Directory -Path "<fsrm-report-path>" -Force` | FSRM report folder exists |
| 21 | Create migration staging path | File Server | `New-Item -ItemType Directory -Path "<staging-path>" -Force` | Staging folder exists |
| 22 | Create backup staging path | File Server | `New-Item -ItemType Directory -Path "<backup-staging-path>" -Force` | Backup staging folder exists |
| 23 | Capture default ACLs before task 03 | File Server | `Get-Acl "<share-root>" \| Format-List` | Default ACL state is documented |
| 24 | Confirm folder structure | File Server | `Get-ChildItem "<data-volume-path>" -Directory` | Expected root folders are visible |
| 25 | Confirm paths exist | File Server | `Test-Path "<share-root>"`; `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"` | Required paths return True |
| 26 | Export storage evidence | File Server | `Get-Disk`; `Get-Volume`; `Get-Partition` | Storage state is documented |
| 27 | Document final layout | Operator | `Record disk, drive letter, volume label, file system, root folders, and evidence path` | Storage and folder build record is complete |

# Prepare_File_Server_Storage_Volumes_And_Folders_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture disk, volume, role, and folder state before storage changes.

$EvidencePath = "C:\FileServices-Validation"
$DataDriveLetter = "E"
$ShareRoot = "E:\Shares"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-storage-precheck-transcript.txt"

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-storage.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname-before-storage.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-storage.txt"

# Confirm File Server role.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-storage.txt"

# Capture disk, partition, and volume state.
Get-Disk |
  Tee-Object "$EvidencePath\disks-before-storage.txt"

Get-Partition |
  Tee-Object "$EvidencePath\partitions-before-storage.txt"

Get-Volume |
  Tee-Object "$EvidencePath\volumes-before-storage.txt"

Get-Disk |
  Export-Csv "$EvidencePath\disks-before-storage.csv" -NoTypeInformation

Get-Partition |
  Export-Csv "$EvidencePath\partitions-before-storage.csv" -NoTypeInformation

Get-Volume |
  Export-Csv "$EvidencePath\volumes-before-storage.csv" -NoTypeInformation

# Check whether target drive letter already exists.
Get-Volume -DriveLetter $DataDriveLetter -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-driveletter-before-storage.txt"

# Check whether share root already exists.
Test-Path $ShareRoot |
  Tee-Object "$EvidencePath\share-root-testpath-before-storage.txt"

if (Test-Path $ShareRoot) {
  Get-ChildItem $ShareRoot -Force |
    Tee-Object "$EvidencePath\share-root-children-before-storage.txt"

  Get-Acl $ShareRoot |
    Format-List |
    Tee-Object "$EvidencePath\share-root-acl-before-storage.txt"
}

Stop-Transcript
```


Prepare_File_Server_Storage_Volumes_And_Folders_Disk_Initialize_And_Format_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: initialize, partition, and format a new dedicated data disk.
# Warning: verify the disk number before running. Formatting the wrong disk destroys data.

$EvidencePath = "C:\FileServices-Validation"
$TargetDiskNumber = 1
$DataDriveLetter = "E"
$VolumeLabel = "FileData"
$FileSystem = "NTFS"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-disk-initialize-format-transcript.txt"

# Show target disk before changes.
Get-Disk -Number $TargetDiskNumber |
  Tee-Object "$EvidencePath\target-disk-before-format.txt"

Get-Partition -DiskNumber $TargetDiskNumber -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-disk-partitions-before-format.txt"

# Safety check: stop if the disk is not RAW and already has partitions.
$TargetDisk = Get-Disk -Number $TargetDiskNumber
$TargetPartitions = Get-Partition -DiskNumber $TargetDiskNumber -ErrorAction SilentlyContinue

if ($TargetDisk.PartitionStyle -ne "RAW" -and $TargetPartitions) {
  Write-Error "Target disk is not RAW or already has partitions. Stop and verify disk selection before continuing."
  Stop-Transcript
  return
}

# Initialize raw disk.
if ($TargetDisk.PartitionStyle -eq "RAW") {
  Initialize-Disk `
    -Number $TargetDiskNumber `
    -PartitionStyle GPT
}

# Create partition and assign drive letter.
New-Partition `
  -DiskNumber $TargetDiskNumber `
  -UseMaximumSize `
  -DriveLetter $DataDriveLetter

# Format volume.
Format-Volume `
  -DriveLetter $DataDriveLetter `
  -FileSystem $FileSystem `
  -NewFileSystemLabel $VolumeLabel `
  -Confirm:$false

# Confirm final state.
Get-Disk -Number $TargetDiskNumber |
  Tee-Object "$EvidencePath\target-disk-after-format.txt"

Get-Partition -DiskNumber $TargetDiskNumber |
  Tee-Object "$EvidencePath\target-disk-partitions-after-format.txt"

Get-Volume -DriveLetter $DataDriveLetter |
  Tee-Object "$EvidencePath\target-volume-after-format.txt"

Stop-Transcript
```


Prepare_File_Server_Storage_Volumes_And_Folders_Folder_Structure_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: create the base file server folder structure.
# This workbook creates folders only. NTFS permission design happens in task 03.

$EvidencePath = "C:\FileServices-Validation"

$DataRoot = "E:\"
$ShareRoot = "E:\Shares"
$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"
$ProfilesRoot = "E:\Shares\Profiles"
$RedirectedFoldersRoot = "E:\Shares\RedirectedFolders"
$DfsRootFolder = "E:\DFSRoots"
$FsrmReportPath = "E:\FSRM-Reports"
$StagingPath = "E:\Staging"
$BackupStagingPath = "E:\Backup-Staging"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-folder-structure-transcript.txt"

# Confirm data volume exists.
Test-Path $DataRoot |
  Tee-Object "$EvidencePath\data-root-testpath.txt"

# Create required root folders.
$Folders = @(
  $ShareRoot,
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $AppsRoot,
  $ProfilesRoot,
  $RedirectedFoldersRoot,
  $DfsRootFolder,
  $FsrmReportPath,
  $StagingPath,
  $BackupStagingPath
)

foreach ($Folder in $Folders) {
  New-Item `
    -ItemType Directory `
    -Path $Folder `
    -Force
}

# Confirm folder structure.
Get-ChildItem $DataRoot -Directory -Force |
  Sort-Object FullName |
  Tee-Object "$EvidencePath\data-root-folders-after-create.txt"

Get-ChildItem $ShareRoot -Directory -Force |
  Sort-Object FullName |
  Tee-Object "$EvidencePath\share-root-folders-after-create.txt"

# Capture default ACLs for later permission design.
foreach ($Folder in $Folders) {
  Get-Acl $Folder |
    Select-Object @{Name="Path";Expression={$Folder}},Owner,Group,AccessToString |
    Tee-Object "$EvidencePath\folder-acl-summary.txt" -Append
}

Stop-Transcript
```


Prepare_File_Server_Storage_Volumes_And_Folders_Share_Root_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: validate required folder paths before ACL and SMB share configuration.

$EvidencePath = "C:\FileServices-Validation"

$RequiredPaths = @(
  "E:\Shares",
  "E:\Shares\Departments",
  "E:\Shares\Home",
  "E:\Shares\Public",
  "E:\Shares\Apps",
  "E:\DFSRoots",
  "E:\FSRM-Reports",
  "E:\Staging",
  "E:\Backup-Staging"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-share-root-validation-transcript.txt"

$ValidationResults = foreach ($Path in $RequiredPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
    ItemType = if (Test-Path $Path) { (Get-Item $Path).PSIsContainer } else { $null }
  }
}

$ValidationResults |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\required-path-validation.txt"

$ValidationResults |
  Export-Csv "$EvidencePath\required-path-validation.csv" -NoTypeInformation

# Capture directory inventory.
Get-ChildItem "E:\" -Directory -Force |
  Sort-Object FullName |
  Tee-Object "$EvidencePath\e-drive-directory-inventory.txt"

Get-ChildItem "E:\Shares" -Directory -Force |
  Sort-Object FullName |
  Tee-Object "$EvidencePath\shares-directory-inventory.txt"

# Capture ACL state for task 03.
Get-Acl "E:\Shares" |
  Format-List |
  Tee-Object "$EvidencePath\shares-root-acl-before-task-03.txt"

Get-Acl "E:\Shares\Departments" |
  Format-List |
  Tee-Object "$EvidencePath\departments-root-acl-before-task-03.txt"

Get-Acl "E:\Shares\Home" |
  Format-List |
  Tee-Object "$EvidencePath\home-root-acl-before-task-03.txt"

Stop-Transcript
```

Prepare_File_Server_Storage_Volumes_And_Folders_Post_Build_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: final validation after preparing storage volume and folder structure.

$EvidencePath = "C:\FileServices-Validation"
$DataDriveLetter = "E"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-post-build-validation-transcript.txt"

# Confirm disk, partition, and volume state.
Get-Disk |
  Tee-Object "$EvidencePath\disks-after-storage-build.txt"

Get-Partition |
  Tee-Object "$EvidencePath\partitions-after-storage-build.txt"

Get-Volume |
  Tee-Object "$EvidencePath\volumes-after-storage-build.txt"

Get-Volume -DriveLetter $DataDriveLetter |
  Tee-Object "$EvidencePath\data-volume-final.txt"

# Confirm file system and free space.
Get-Volume -DriveLetter $DataDriveLetter |
  Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining |
  Tee-Object "$EvidencePath\data-volume-capacity-final.txt"

# Confirm folder structure.
Get-ChildItem "$($DataDriveLetter):\" -Directory -Force |
  Sort-Object FullName |
  Tee-Object "$EvidencePath\data-volume-root-folders-final.txt"

Get-ChildItem "$($DataDriveLetter):\Shares" -Directory -Force |
  Sort-Object FullName |
  Tee-Object "$EvidencePath\share-root-folders-final.txt"

# Confirm no user SMB shares were created in this workbook.
Get-SmbShare |
  Sort-Object Name |
  Tee-Object "$EvidencePath\smb-shares-after-folder-prep.txt"

# Capture ACLs for handoff to task 03.
$AclPaths = @(
  "E:\Shares",
  "E:\Shares\Departments",
  "E:\Shares\Home",
  "E:\Shares\Public",
  "E:\Shares\Apps"
)

foreach ($Path in $AclPaths) {
  Get-Acl $Path |
    Format-List |
    Tee-Object "$EvidencePath\acl-$($Path.Replace(':','').Replace('\','-')).txt"
}

Stop-Transcript
```

Prepare_File_Server_Storage_Volumes_And_Folders_Verification_Commands
```
# Role and server baseline
hostname
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
Get-WindowsFeature FS-FileServer

# Disk and volume validation
Get-Disk
Get-Disk -Number <target-disk-number>
Get-Partition -DiskNumber <target-disk-number>
Get-Volume
Get-Volume -DriveLetter <data-drive-letter>

# Folder validation
Test-Path "<share-root>"
Test-Path "<department-root>"
Test-Path "<home-folder-root>"
Test-Path "<public-share-root>"
Test-Path "<app-share-root>"
Get-ChildItem "<data-volume-path>" -Directory -Force
Get-ChildItem "<share-root>" -Directory -Force

# ACL capture for next workbook
Get-Acl "<share-root>" | Format-List
Get-Acl "<department-root>" | Format-List
Get-Acl "<home-folder-root>" | Format-List
icacls "<share-root>"
icacls "<department-root>"
icacls "<home-folder-root>"

# Confirm shares have not been created yet, except defaults
Get-SmbShare

# Evidence
Test-Path "C:\FileServices-Validation"
Get-ChildItem "C:\FileServices-Validation"
```

Prepare_File_Server_Storage_Volumes_And_Folders_Rollback

|      |                                                                 |             |                                                                                                                                     |                                             |
| ---- | --------------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| Step | Task                                                            | Device      | PowerShell / Command                                                                                                                | Expected Result                             |
| 1    | Confirm no production shares point to folders                   | File Server | Get-SmbShare                                                                                                                        | No created SMB shares depend on the folders |
| 2    | Export current folder inventory                                 | File Server | Get-ChildItem "<data-volume-path>" -Recurse \| Export-Csv "<evidence-path>\folder-inventory-before-rollback.csv" -NoTypeInformation | Folder inventory is saved                   |
| 3    | Remove empty staging folder if created incorrectly              | File Server | Remove-Item "<staging-path>" -Recurse -Force                                                                                        | Staging folder is removed                   |
| 4    | Remove empty backup staging folder if created incorrectly       | File Server | Remove-Item "<backup-staging-path>" -Recurse -Force                                                                                 | Backup staging folder is removed            |
| 5    | Remove optional profiles root if not needed                     | File Server | Remove-Item "<profiles-root>" -Recurse -Force                                                                                       | Optional folder is removed                  |
| 6    | Remove optional redirected folders root if not needed           | File Server | Remove-Item "<redirected-folders-root>" -Recurse -Force                                                                             | Optional folder is removed                  |
| 7    | Remove DFS root folder if no DFS namespace uses it              | File Server | Remove-Item "<dfs-root-folder>" -Recurse -Force                                                                                     | DFS root folder is removed                  |
| 8    | Remove FSRM report folder if no reports use it                  | File Server | Remove-Item "<fsrm-report-path>" -Recurse -Force                                                                                    | FSRM report path is removed                 |
| 9    | Remove share root only in early lab stage                       | File Server | Remove-Item "<share-root>" -Recurse -Force                                                                                          | Share root and child folders are removed    |
| 10   | Remove data partition only in lab and only after verifying disk | File Server | Remove-Partition -DiskNumber <target-disk-number> -PartitionNumber <partition-number> -Confirm:$false                               | Partition is removed                        |
| 11   | Clear disk only in lab and only after verifying disk            | File Server | Clear-Disk -Number <target-disk-number> -RemoveData -Confirm:$false                                                                 | Disk returns to unallocated state           |
| 12   | Document rollback                                               | Operator    | Record removed folders, partition changes, and evidence path                                                                        | Rollback record is complete                 |

Prepare_File_Server_Storage_Volumes_And_Folders_Failure_Checks

|   |   |   |   |
|---|---|---|---|
|Symptom|Likely Cause|Check|Corrective Action|
|Target disk is not visible|Disk not attached, offline, or storage controller issue|Get-Disk|Attach disk, bring disk online, or fix VM/storage configuration|
|Disk shows Offline|SAN policy or admin action left disk offline|Get-Disk -Number <disk-number>|Run Set-Disk -Number <disk-number> -IsOffline $false|
|Disk is read-only|Disk readonly flag is set|Get-Disk -Number <disk-number> \| Select IsReadOnly|Run Set-Disk -Number <disk-number> -IsReadOnly $false|
|Initialize fails|Wrong disk state or permissions issue|Get-Disk; elevated PowerShell check|Verify disk is RAW and run as Administrator|
|Safety script stops before formatting|Disk already has partitions|Get-Partition -DiskNumber <disk-number>|Stop and verify this is not a disk with needed data|
|Drive letter already in use|Another volume owns the chosen letter|Get-Volume -DriveLetter <letter>|Choose another letter or reassign safely|
|Format fails|Volume is in use or wrong drive letter|Get-Volume; Get-Partition|Confirm target volume and retry|
|Folder creation fails|Parent path does not exist|Test-Path "<parent-path>"|Create parent path or correct drive letter|
|Folder creation fails|Permission issue|whoami /groups; Get-Acl "<parent-path>"|Use elevated PowerShell or correct admin permissions|
|Expected folder missing|Variable typo or wrong root path|Review transcript and Get-ChildItem|Correct variable and rerun folder skeleton|
|ACLs look too permissive|Default inherited permissions still present|Get-Acl "<folder-path>"|Leave as captured. Configure real ACLs in task 03|
|User shares already exist|Previous lab work created shares|Get-SmbShare|Document existing shares before continuing|
|Evidence files missing|Evidence path not created or transcript failed|Test-Path "<evidence-path>"|Re-run skeleton as Administrator|
|Data volume full or too small|Disk size was undersized|Get-Volume -DriveLetter <letter>|Expand disk or choose larger data disk|
|Wrong disk was selected|Disk number not validated before command|Compare evidence files|Stop, preserve evidence, restore from backup if data was lost|
|ReFS chosen accidentally|Wrong file system variable|Get-Volume -DriveLetter <letter>|Reformat only in lab before data is placed|
|Staging folders placed on OS drive|Wrong path variables|Get-ChildItem C:\ and Get-ChildItem E:\|Move staging paths to data volume|

