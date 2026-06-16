# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Index
10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md
10_Configure_Checkpoints_Production_Checkpoints_And_Restore
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Source_Basis
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Mental_Model
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Planning_Table
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Configuration_Checklist
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_PreChange_Capture_Skeleton
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Checkpoint_Policy_Skeleton
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Create_Checkpoint_Skeleton
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Apply_Restore_Checkpoint_Skeleton
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Remove_And_Merge_Checkpoints_Skeleton
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_PostChange_Verification_Skeleton
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Verification_Commands
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Rollback
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Failure_Checks
10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Related_Labs

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V checkpoints | Supports checkpoint types, creation, application, and deletion |
| Microsoft Learn | Production checkpoints | Supports application-consistent checkpoints through guest backup technologies |
| Microsoft Learn | Standard checkpoints | Supports saved-state checkpoints for lab and test rollback |
| Microsoft Learn | Hyper-V PowerShell `Checkpoint-VM`, `Get-VMCheckpoint`, `Restore-VMCheckpoint`, `Remove-VMCheckpoint` | Supports checkpoint creation, restore, inventory, and cleanup |
| Microsoft Learn | Hyper-V PowerShell `Set-VM` | Supports configuring checkpoint type, checkpoint path, and automatic checkpoints |
| Microsoft Learn | Hyper-V storage behavior | Supports checkpoint storage planning, AVHDX chain awareness, and merge behavior |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Checkpoint | A point-in-time state of a VM used for rollback |
| Production checkpoint | Uses guest backup integration to create an application-consistent checkpoint |
| Standard checkpoint | Captures VM state, memory, and device state, useful for labs but riskier for production workloads |
| Checkpoint type | VM-level policy that controls whether checkpoints are production, standard, or disabled |
| Automatic checkpoint | Hyper-V Manager feature that can create a checkpoint automatically when a VM starts |
| Checkpoint path | Location where checkpoint configuration and differencing disks are stored |
| AVHDX | Differencing disk created by a checkpoint |
| Checkpoint chain | Parent and child disk chain created when checkpoints exist |
| Apply checkpoint | Restores the VM to the selected checkpoint state |
| Delete checkpoint | Removes the checkpoint object and triggers disk merge behavior |
| Merge | Process where Hyper-V folds checkpoint differencing data back into the parent disk |
| Guest consistency | Whether the guest OS and applications are in a safe state after rollback |
| Backup boundary | Checkpoints are not backups and should not replace backup or replication workflows |
| Rollback boundary | This workbook handles VM checkpoint rollback, not full backup restore or disaster recovery |

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| VM workload | Lab server, domain controller, app server, test client | `<workload>` |
| Checkpoint type | Production, Standard, Disabled | `<checkpoint-type>` |
| Production checkpoint fallback | Enabled or disabled | `<enabled-disabled>` |
| Automatic checkpoints | Enabled or disabled | `<enabled-disabled>` |
| Checkpoint path | `D:\Hyper-V\Checkpoints\<vm-name>` | `<checkpoint-path>` |
| Checkpoint name | `Before-Role-Install` | `<checkpoint-name>` |
| Restore target | Latest checkpoint, named checkpoint | `<restore-target>` |
| VM state before checkpoint | Running, Off | `<vm-state>` |
| VM state before restore | Off preferred for controlled restore | `<vm-state>` |
| Application consistency required | Yes or No | `<yes-no>` |
| Domain controller caution required | Yes or No | `<yes-no>` |
| Free space required | `2x changed data expected` | `<free-space-requirement>` |
| Evidence path | `C:\Admin\HyperV-Checkpoints` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | Target VM is found |
| 3 | Confirm VM state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, State` | VM state is known |
| 4 | Capture current checkpoint policy | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, CheckpointType, AutomaticCheckpointsEnabled, Path` | Current checkpoint policy is documented |
| 5 | Capture existing checkpoints | Hyper-V host | `Get-VMCheckpoint -VMName '<vm-name>'` | Existing checkpoint tree is documented |
| 6 | Capture disk chain state | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>' \| ForEach-Object { Get-VHD -Path $_.Path }` | VHDX or AVHDX state is documented |
| 7 | Confirm checkpoint storage path exists | Hyper-V host | `Test-Path 'D:\Hyper-V\Checkpoints\<vm-name>'` | Checkpoint path status is known |
| 8 | Create checkpoint folder if required | Hyper-V host | `New-Item -Path 'D:\Hyper-V\Checkpoints\<vm-name>' -ItemType Directory -Force` | Checkpoint folder exists |
| 9 | Configure production checkpoints | Hyper-V host | `Set-VM -Name '<vm-name>' -CheckpointType Production` | VM uses production checkpoints |
| 10 | Configure standard checkpoints for lab-only VM if required | Hyper-V host | `Set-VM -Name '<vm-name>' -CheckpointType Standard` | VM uses standard checkpoints |
| 11 | Disable checkpoints if required | Hyper-V host | `Set-VM -Name '<vm-name>' -CheckpointType Disabled` | VM checkpoints are disabled |
| 12 | Configure checkpoint path | Hyper-V host | `Set-VM -Name '<vm-name>' -SnapshotFileLocation 'D:\Hyper-V\Checkpoints\<vm-name>'` | Checkpoint files use planned location |
| 13 | Disable automatic checkpoints unless intentionally used | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticCheckpointsEnabled $false` | Automatic checkpoints are disabled |
| 14 | Create named checkpoint | Hyper-V host | `Checkpoint-VM -Name '<vm-name>' -SnapshotName '<checkpoint-name>'` | Named checkpoint is created |
| 15 | Confirm checkpoint exists | Hyper-V host | `Get-VMCheckpoint -VMName '<vm-name>'` | Checkpoint appears in inventory |
| 16 | Apply checkpoint when restore is required | Hyper-V host | `Restore-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>' -Confirm:$false` | VM reverts to checkpoint state |
| 17 | Remove checkpoint after validation | Hyper-V host | `Remove-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>'` | Checkpoint is removed and merge begins or completes |
| 18 | Confirm no orphaned checkpoint chain remains | Hyper-V host | `Get-VHD -Path '<vhdx-or-avhdx-path>'` | Parent path and chain state are healthy |
| 19 | Confirm storage free space after merge | Hyper-V host | `Get-Volume -DriveLetter D` | Volume has expected free space |
| 20 | Capture post-change evidence | Hyper-V host | Run post-change verification skeleton | Checkpoint configuration and state are documented |

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_PreChange_Capture_Skeleton

~~~powershell
# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_PreChange_Capture_Skeleton
# Purpose:
# Capture current VM checkpoint policy, checkpoint inventory, disk chain, VM state, and storage state before changes.

$EvidenceRoot = "C:\Admin\HyperV-Checkpoints"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary and checkpoint policy..." -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Generation, CheckpointType, AutomaticCheckpointsEnabled, Path, SnapshotFileLocation |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-CheckpointPolicy-Before.txt")

$VM |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "01-VM-CheckpointPolicy-Before.json") -Encoding UTF8

Write-Host "Capturing existing checkpoints..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Id, CreationTime, ParentSnapshotName, SnapshotType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMCheckpoints-Before.txt")

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "02-VMCheckpoints-Before.json") -Encoding UTF8

Write-Host "Capturing VM disk attachments..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMHardDiskDrive-Before.txt")

Write-Host "Capturing VHD and AVHDX chain details..." -ForegroundColor Cyan

foreach ($Disk in $HardDisks) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, MinimumSize, ParentPath, Attached, FragmentationPercentage |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("04-VHDChain-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Searching for AVHDX files in VM storage paths..." -ForegroundColor Cyan

$VMRoot = Split-Path $VM.Path -Parent

Get-ChildItem -Path $VMRoot -Recurse -Include "*.avhdx","*.vhdx" -ErrorAction SilentlyContinue |
    Select-Object Name, FullName, Length, CreationTime, LastWriteTime |
    Sort-Object FullName |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMStorage-VHDX-AVHDX-Inventory-Before.txt")

Write-Host "Capturing volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-Volumes-Before.txt")

Write-Host "Capturing recent checkpoint related events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-VMMS-RecentEvents-Before.txt") -Encoding UTF8

Write-Host "Pre-change checkpoint evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Checkpoint_Policy_Skeleton

~~~powershell
# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Checkpoint_Policy_Skeleton
# Purpose:
# Configure checkpoint type, checkpoint path, and automatic checkpoint behavior for a VM.

$VMName = "LAB-WIN-SRV01"
$CheckpointType = "Production"
$CheckpointPath = "D:\Hyper-V\Checkpoints\$VMName"
$AutomaticCheckpointsEnabled = $false

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Creating checkpoint path if missing..." -ForegroundColor Cyan

New-Item -Path $CheckpointPath -ItemType Directory -Force | Out-Null

Write-Host "Configuring checkpoint policy..." -ForegroundColor Cyan

switch ($CheckpointType) {
    "Production" {
        Set-VM `
            -Name $VMName `
            -CheckpointType Production `
            -SnapshotFileLocation $CheckpointPath `
            -AutomaticCheckpointsEnabled $AutomaticCheckpointsEnabled
    }

    "Standard" {
        Set-VM `
            -Name $VMName `
            -CheckpointType Standard `
            -SnapshotFileLocation $CheckpointPath `
            -AutomaticCheckpointsEnabled $AutomaticCheckpointsEnabled
    }

    "Disabled" {
        Set-VM `
            -Name $VMName `
            -CheckpointType Disabled `
            -AutomaticCheckpointsEnabled $AutomaticCheckpointsEnabled
    }

    default {
        throw "Unsupported checkpoint type: $CheckpointType. Use Production, Standard, or Disabled."
    }
}

Write-Host "Checkpoint policy after change:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, State, CheckpointType, AutomaticCheckpointsEnabled, SnapshotFileLocation |
    Format-List
~~~

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Create_Checkpoint_Skeleton

~~~powershell
# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Create_Checkpoint_Skeleton
# Purpose:
# Create a named checkpoint after confirming policy, disk state, and free space.

$VMName = "LAB-WIN-SRV01"
$CheckpointName = "Before-Change-" + (Get-Date -Format "yyyyMMdd-HHmmss")

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Current checkpoint policy:" -ForegroundColor Cyan

$VM |
    Select-Object Name, State, CheckpointType, AutomaticCheckpointsEnabled, SnapshotFileLocation |
    Format-List

if ($VM.CheckpointType -eq "Disabled") {
    throw "Checkpoints are disabled for $VMName. Change CheckpointType before creating a checkpoint."
}

Write-Host "Checking storage volume free space..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize

Write-Host "Existing checkpoints:" -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, CreationTime, SnapshotType |
    Format-Table -AutoSize

Write-Host "Creating checkpoint: $CheckpointName" -ForegroundColor Cyan

Checkpoint-VM `
    -Name $VMName `
    -SnapshotName $CheckpointName

Write-Host "Checkpoint inventory after creation:" -ForegroundColor Green

Get-VMCheckpoint -VMName $VMName |
    Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
    Format-Table -AutoSize

Write-Host "VM disk attachments after checkpoint:" -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize
~~~

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Apply_Restore_Checkpoint_Skeleton

~~~powershell
# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Apply_Restore_Checkpoint_Skeleton
# Purpose:
# Restore a VM to a selected checkpoint in a controlled way.

$VMName = "LAB-WIN-SRV01"
$CheckpointName = "Before-Change-20260101-120000"
$StopVMBeforeRestore = $true
$CreateSafetyCheckpointBeforeRestore = $false

Write-Host "Validating target VM and checkpoint..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

$Checkpoint = Get-VMCheckpoint -VMName $VMName -Name $CheckpointName -ErrorAction Stop

Write-Host "Selected checkpoint:" -ForegroundColor Cyan

$Checkpoint |
    Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
    Format-List

if ($CreateSafetyCheckpointBeforeRestore) {
    $SafetyCheckpointName = "Safety-Before-Restore-" + (Get-Date -Format "yyyyMMdd-HHmmss")

    Write-Host "Creating safety checkpoint before restore: $SafetyCheckpointName" -ForegroundColor Yellow

    Checkpoint-VM `
        -Name $VMName `
        -SnapshotName $SafetyCheckpointName
}

if ($StopVMBeforeRestore) {
    $VM = Get-VM -Name $VMName

    if ($VM.State -ne "Off") {
        Write-Host "Stopping VM before restore..." -ForegroundColor Cyan

        Stop-VM -Name $VMName
    }
}

Write-Warning "Restore will revert VM state to checkpoint: $CheckpointName"

Restore-VMCheckpoint `
    -VMName $VMName `
    -Name $CheckpointName `
    -Confirm:$false

Write-Host "VM state after restore:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, CheckpointType, Path |
    Format-List

Write-Host "Checkpoint inventory after restore:" -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName |
    Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
    Format-Table -AutoSize
~~~

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Remove_And_Merge_Checkpoints_Skeleton

~~~powershell
# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Remove_And_Merge_Checkpoints_Skeleton
# Purpose:
# Remove one checkpoint or all checkpoints and monitor merge behavior.

$VMName = "LAB-WIN-SRV01"
$RemoveMode = "Named"

$CheckpointName = "Before-Change-20260101-120000"

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Checkpoint inventory before removal:" -ForegroundColor Cyan

$Checkpoints = Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue

$Checkpoints |
    Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
    Format-Table -AutoSize

if (-not $Checkpoints) {
    Write-Host "No checkpoints exist for $VMName." -ForegroundColor Yellow
    return
}

Write-Host "VM disk attachments before removal:" -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

switch ($RemoveMode) {
    "Named" {
        Write-Host "Removing named checkpoint: $CheckpointName" -ForegroundColor Cyan

        Remove-VMCheckpoint `
            -VMName $VMName `
            -Name $CheckpointName
    }

    "All" {
        Write-Host "Removing all checkpoints for VM: $VMName" -ForegroundColor Cyan

        Get-VMCheckpoint -VMName $VMName |
            Remove-VMCheckpoint
    }

    default {
        throw "Unsupported RemoveMode: $RemoveMode. Use Named or All."
    }
}

Write-Host "Checkpoint inventory after removal command:" -ForegroundColor Green

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
    Format-Table -AutoSize

Write-Host "VM disk attachments after removal command:" -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

Write-Host "Watch Hyper-V Manager or VMMS events if merge is still running." -ForegroundColor Yellow
~~~

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_PostChange_Verification_Skeleton

~~~powershell
# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_PostChange_Verification_Skeleton
# Purpose:
# Verify checkpoint policy, checkpoint inventory, disk chain, storage state, and recent events after checkpoint operations.

$EvidenceRoot = "C:\Admin\HyperV-Checkpoints"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM checkpoint policy after change..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

$VM |
    Select-Object Name, State, Generation, CheckpointType, AutomaticCheckpointsEnabled, SnapshotFileLocation, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-CheckpointPolicy-After.txt")

Write-Host "Capturing checkpoint inventory after change..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Id, CreationTime, ParentSnapshotName, SnapshotType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMCheckpoints-After.txt")

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "02-VMCheckpoints-After.json") -Encoding UTF8

Write-Host "Capturing VM disk attachments after checkpoint operations..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMHardDiskDrive-After.txt")

Write-Host "Capturing VHD and AVHDX chain details after checkpoint operations..." -ForegroundColor Cyan

foreach ($Disk in $HardDisks) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, MinimumSize, ParentPath, Attached, FragmentationPercentage |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("04-VHDChain-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Searching VM storage for VHDX and AVHDX files..." -ForegroundColor Cyan

$VMRoot = Split-Path $VM.Path -Parent

Get-ChildItem -Path $VMRoot -Recurse -Include "*.avhdx","*.vhdx" -ErrorAction SilentlyContinue |
    Select-Object Name, FullName, Length, CreationTime, LastWriteTime |
    Sort-Object FullName |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMStorage-VHDX-AVHDX-Inventory-After.txt")

Write-Host "Capturing volume state after checkpoint operations..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-Volumes-After.txt")

Write-Host "Capturing recent Hyper-V VMMS events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-VMMS-RecentEvents-After.txt") -Encoding UTF8

Write-Host "Capturing recent Hyper-V worker events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-HyperV-Worker-RecentEvents-After.txt") -Encoding UTF8

Write-Host "Post-change checkpoint evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>' \| Select Name, State, CheckpointType, AutomaticCheckpointsEnabled, SnapshotFileLocation` | Hyper-V host | Confirms VM checkpoint policy |
| `Get-VMCheckpoint -VMName '<vm-name>'` | Hyper-V host | Lists checkpoints for the VM |
| `Get-VMCheckpoint -VMName '<vm-name>' \| Select Name, CreationTime, SnapshotType, ParentSnapshotName` | Hyper-V host | Confirms checkpoint tree and type |
| `Checkpoint-VM -Name '<vm-name>' -SnapshotName '<checkpoint-name>'` | Hyper-V host | Creates a named checkpoint |
| `Restore-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>'` | Hyper-V host | Restores VM to selected checkpoint |
| `Remove-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>'` | Hyper-V host | Removes selected checkpoint and triggers merge |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Shows current VM disk attachments |
| `Get-VHD -Path '<vhdx-or-avhdx-path>'` | Hyper-V host | Shows disk type, parent path, file size, and chain state |
| `Get-ChildItem '<vm-storage-path>' -Recurse -Include '*.avhdx','*.vhdx'` | Hyper-V host | Finds checkpoint differencing disks |
| `Get-Volume -DriveLetter D` | Hyper-V host | Confirms storage free space and health |
| `Get-VMIntegrationService -VMName '<vm-name>'` | Hyper-V host | Confirms guest integration services that production checkpoints rely on |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks recent checkpoint and merge events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM worker checkpoint or restore errors |

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Review current checkpoint inventory | Hyper-V host | `Get-VMCheckpoint -VMName '<vm-name>'` | Existing checkpoints are identified |
| 2 | Restore VM to prior checkpoint if rollback is required | Hyper-V host | `Restore-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>' -Confirm:$false` | VM returns to selected checkpoint state |
| 3 | Recreate checkpoint policy from baseline | Hyper-V host | `Set-VM -Name '<vm-name>' -CheckpointType '<old-type>'` | Checkpoint type is restored |
| 4 | Restore automatic checkpoint setting | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticCheckpointsEnabled '<old-value>'` | Automatic checkpoint behavior is restored |
| 5 | Restore checkpoint path | Hyper-V host | `Set-VM -Name '<vm-name>' -SnapshotFileLocation '<old-path>'` | Snapshot file location returns to baseline |
| 6 | Remove unwanted checkpoint | Hyper-V host | `Remove-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>'` | Unwanted checkpoint is removed |
| 7 | Remove all lab checkpoints if cleanup is approved | Hyper-V host | `Get-VMCheckpoint -VMName '<vm-name>' \| Remove-VMCheckpoint` | All checkpoints are removed and merges complete |
| 8 | Confirm disk chain after cleanup | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>' \| ForEach-Object { Get-VHD -Path $_.Path }` | Disk chain is healthy |
| 9 | Confirm VM starts after restore or cleanup | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts successfully |
| 10 | Capture rollback evidence | Hyper-V host | `Get-VM -Name '<vm-name>'; Get-VMCheckpoint -VMName '<vm-name>'` | Rollback state is documented |

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Checkpoint creation fails | Checkpoints disabled for the VM | Set `CheckpointType` to `Production` or `Standard` |
| Production checkpoint fails | Guest integration services, VSS, or guest backup support issue | Verify integration services, guest VSS writers, and guest OS support |
| Production checkpoint falls back to standard behavior | Production fallback allowed and guest consistency failed | Fix guest backup integration or disable fallback by using stricter policy where supported |
| Standard checkpoint works but production checkpoint fails | Guest application-consistent checkpoint path is broken | Check VSS, guest services, Linux integration support, and event logs |
| Checkpoint consumes too much storage | Long-lived AVHDX chain and high data churn | Remove stale checkpoints and allow merge to complete |
| VM performance drops after checkpoints | AVHDX chain is deep or storage is under pressure | Merge checkpoints and improve storage capacity or performance |
| VM cannot start after applying checkpoint | Restored state has boot, disk, or guest corruption issue | Try another checkpoint or restore from backup |
| Restore selected wrong checkpoint | Similar checkpoint names or unclear tree | Use unique names with date and purpose |
| Remove checkpoint appears slow | Merge is still running | Monitor disk activity and VMMS events, do not interrupt host |
| AVHDX remains after removing checkpoint | Merge incomplete, VM running, or orphaned chain | Confirm VMMS events, shut down VM if required, allow merge to complete |
| Host volume fills during checkpoint | Not enough free space for changed data or merge operation | Free space, move VM storage, or remove stale checkpoints |
| Checkpoint path wrong | Snapshot file location misconfigured | Set `SnapshotFileLocation` to planned path |
| Automatic checkpoints keep appearing | Automatic checkpoints enabled | Run `Set-VM -Name '<vm-name>' -AutomaticCheckpointsEnabled $false` |
| Guest time or domain state breaks after restore | Checkpoint restore rolled back domain or time-sensitive state | Avoid checkpoints for certain domain controller and distributed workload scenarios |
| Backup software conflicts with checkpoints | Backup job uses Hyper-V checkpoint mechanics | Coordinate with backup window and check backup provider behavior |
| VM is stuck in saved or restoring state | Interrupted checkpoint, restore, or merge operation | Review VMMS and Worker logs before forcing actions |
| `Get-VHD` shows parent path missing | Broken checkpoint disk chain | Restore missing parent disk from backup or rebuild VM from known-good copy |
| Checkpoint cannot be deleted | File lock, storage issue, or VMMS issue | Confirm no backup job is active, verify storage health, restart VMMS only during maintenance |

# 10_Configure_Checkpoints_Production_Checkpoints_And_Restore_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this checkpoint task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host storage and readiness before checkpoint-heavy work |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before checkpoint cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Provides checkpoint path planning and host defaults |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates the VMs that receive checkpoints |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | VM memory and state affect checkpoint behavior |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Checkpoints create AVHDX differencing disks tied to VHDX chains |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Network changes are common reasons to checkpoint before rollback testing |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Integration services support production checkpoint behavior |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Lifecycle operations interact with checkpoint state |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Clarifies the difference between checkpoints and backups |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses checkpoint state, AVHDX chains, and restore behavior during troubleshooting |