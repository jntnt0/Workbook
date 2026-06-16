# 07_Create_Attach_Resize_And_Compact_VHDX_Disks

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Index
07_Create_Attach_Resize_And_Compact_VHDX_Disks.md
07_Create_Attach_Resize_And_Compact_VHDX_Disks
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Source_Basis
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Mental_Model
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Planning_Table
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Configuration_Checklist
07_Create_Attach_Resize_And_Compact_VHDX_Disks_PreChange_Capture_Skeleton
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Create_VHDX_Skeleton
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Attach_VHDX_To_VM_Skeleton
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Resize_VHDX_Skeleton
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Compact_And_Optimize_VHDX_Skeleton
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Mount_Initialize_And_Format_Data_Disk_Skeleton
07_Create_Attach_Resize_And_Compact_VHDX_Disks_PostChange_Verification_Skeleton
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Verification_Commands
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Rollback
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Failure_Checks
07_Create_Attach_Resize_And_Compact_VHDX_Disks_Related_Labs

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V virtual hard disk overview | Supports VHDX format, dynamic disks, fixed disks, differencing disks, and disk placement |
| Microsoft Learn | Hyper-V PowerShell `New-VHD` | Supports creating VHDX files with dynamic, fixed, and differencing types |
| Microsoft Learn | Hyper-V PowerShell `Add-VMHardDiskDrive` | Supports attaching virtual disks to VMs |
| Microsoft Learn | Hyper-V PowerShell `Resize-VHD` | Supports growing or shrinking virtual hard disks where supported |
| Microsoft Learn | Hyper-V PowerShell `Optimize-VHD` | Supports compacting and optimizing VHDX files |
| Microsoft Learn | Hyper-V PowerShell `Mount-VHD` and `Dismount-VHD` | Supports offline inspection, initialization, formatting, and repair workflows |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VHDX | Modern Hyper-V virtual disk format with larger capacity and better resiliency than older VHD |
| Dynamic VHDX | Starts small and grows as data is written inside the guest |
| Fixed VHDX | Allocates the full disk size on the host immediately |
| Differencing VHDX | Child disk that stores changes against a parent disk |
| Parent disk | Base disk used by one or more differencing disks |
| Data disk | Extra VHDX attached to a VM for guest storage |
| Boot disk | VHDX used by the VM to boot the operating system |
| SCSI controller | Preferred controller for most modern Hyper-V data disks and Generation 2 boot disks |
| IDE controller | Required for Generation 1 boot disks |
| Online resize | Expanding a supported VHDX while attached to a running VM, normally when attached through SCSI |
| Offline resize | Resizing a disk while VM is off or disk is detached |
| Compacting | Reclaims unused space from dynamically expanding VHDX files |
| Mounting a VHDX | Attaches the VHDX to the host OS for inspection or disk management |
| Guest partition expansion | After growing a VHDX, the guest OS partition must also be extended |
| Rollback boundary | This workbook changes VHDX files and VM attachments, not guest workload data design |

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| Disk purpose | OS disk, data disk, logs disk, lab disk | `<disk-purpose>` |
| VHDX root path | `D:\Hyper-V\Virtual Hard Disks` | `<vhdx-root-path>` |
| VHDX file name | `LAB-WIN-SRV01-Data01.vhdx` | `<vhdx-file-name>` |
| Full VHDX path | `D:\Hyper-V\Virtual Hard Disks\LAB-WIN-SRV01-Data01.vhdx` | `<vhdx-path>` |
| Disk type | Dynamic, fixed, differencing | `<disk-type>` |
| Initial size | `60GB`, `100GB`, `500GB` | `<initial-size>` |
| Target resize size | `120GB`, `250GB` | `<target-size>` |
| Controller type | SCSI, IDE | `<controller-type>` |
| Controller number | `0` | `<controller-number>` |
| Controller location | `1` | `<controller-location>` |
| Guest drive letter | `E:` | `<guest-drive-letter>` |
| Guest file system | NTFS, ReFS | `<file-system>` |
| Allocation unit size | Default, 64K | `<allocation-unit-size>` |
| Volume label | `DATA01` | `<volume-label>` |
| VM must be offline | Yes or No | `<yes-no>` |
| Compact required | Yes or No | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-VHDX` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | Target VM is found |
| 3 | Confirm VM generation and state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, Generation, State` | VM generation and power state are known |
| 4 | Confirm VHDX storage path exists | Hyper-V host | `Test-Path 'D:\Hyper-V\Virtual Hard Disks'` | VHDX folder exists |
| 5 | Capture existing VM disk attachments | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>'` | Current VM disk layout is documented |
| 6 | Capture existing VHDX details | Hyper-V host | `Get-VHD -Path '<vhdx-path>'` | Current disk size, type, and state are documented |
| 7 | Create a dynamic VHDX if required | Hyper-V host | `New-VHD -Path '<vhdx-path>' -SizeBytes 100GB -Dynamic` | Dynamic VHDX is created |
| 8 | Create a fixed VHDX if required | Hyper-V host | `New-VHD -Path '<vhdx-path>' -SizeBytes 100GB -Fixed` | Fixed VHDX is created |
| 9 | Create a differencing VHDX if required | Hyper-V host | `New-VHD -Path '<child-vhdx-path>' -ParentPath '<parent-vhdx-path>' -Differencing` | Differencing disk is created |
| 10 | Attach VHDX as SCSI data disk | Hyper-V host | `Add-VMHardDiskDrive -VMName '<vm-name>' -ControllerType SCSI -ControllerNumber 0 -ControllerLocation 1 -Path '<vhdx-path>'` | Data disk is attached to VM |
| 11 | Attach VHDX as Generation 1 IDE boot disk if required | Hyper-V host | `Add-VMHardDiskDrive -VMName '<vm-name>' -ControllerType IDE -ControllerNumber 0 -ControllerLocation 0 -Path '<vhdx-path>'` | Generation 1 boot disk is attached |
| 12 | Confirm disk attachment | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>'` | Disk appears on the intended controller and location |
| 13 | Resize VHDX upward if required | Hyper-V host | `Resize-VHD -Path '<vhdx-path>' -SizeBytes 150GB` | VHDX maximum size increases |
| 14 | Expand guest partition after VHDX growth | Guest VM | `Resize-Partition -DriveLetter E -Size (Get-PartitionSupportedSize -DriveLetter E).SizeMax` | Guest volume expands to use added space |
| 15 | Shrink guest partition before VHDX shrink if required | Guest VM | `Resize-Partition -DriveLetter E -Size 80GB` | Guest partition is smaller than target VHDX size |
| 16 | Resize VHDX downward only after guest partition shrink | Hyper-V host | `Resize-VHD -Path '<vhdx-path>' -ToMinimumSize` | VHDX shrinks to the minimum safe size |
| 17 | Compact dynamic VHDX after cleanup | Hyper-V host | `Optimize-VHD -Path '<vhdx-path>' -Mode Full` | Unused host space is reclaimed |
| 18 | Mount VHDX for offline initialization if required | Hyper-V host | `Mount-VHD -Path '<vhdx-path>' -Passthru` | VHDX is mounted on the host |
| 19 | Initialize and format mounted data disk if required | Hyper-V host | `Initialize-Disk; New-Partition; Format-Volume` | Mounted VHDX receives a partition and file system |
| 20 | Dismount VHDX after offline work | Hyper-V host | `Dismount-VHD -Path '<vhdx-path>'` | VHDX is safely detached from host |
| 21 | Capture post-change disk evidence | Hyper-V host | Run post-change verification skeleton | VHDX and VM disk state are documented |

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_PreChange_Capture_Skeleton

~~~powershell
# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_PreChange_Capture_Skeleton
# Purpose:
# Capture current VM disk attachments, VHDX inventory, host storage, and VM state before disk changes.

$EvidenceRoot = "C:\Admin\HyperV-VHDX"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$VHDXRoot = "D:\Hyper-V\Virtual Hard Disks"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Generation, Path, ProcessorCount, MemoryStartup |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.txt")

Write-Host "Capturing VM hard disk attachments..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path, DiskNumber, ResourcePoolName |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMHardDiskDrive-Before.txt")

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path, DiskNumber, ResourcePoolName |
    Export-Csv -Path (Join-Path $OutputPath "02-VMHardDiskDrive-Before.csv") -NoTypeInformation

Write-Host "Capturing attached VHDX details..." -ForegroundColor Cyan

$AttachedDisks = Get-VMHardDiskDrive -VMName $VMName

foreach ($Disk in $AttachedDisks) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, MinimumSize, LogicalSectorSize, PhysicalSectorSize, ParentPath, FragmentationPercentage |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("03-VHD-Before-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Capturing VHDX folder inventory..." -ForegroundColor Cyan

Get-ChildItem $VHDXRoot -Filter "*.vhdx" -ErrorAction SilentlyContinue |
    Select-Object Name, FullName, Length, CreationTime, LastWriteTime |
    Sort-Object Name |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VHDXFolderInventory-Before.txt")

Write-Host "Capturing host volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-Volumes-Before.txt")

Write-Host "Capturing disk management state..." -ForegroundColor Cyan

Get-Disk |
    Sort-Object Number |
    Select-Object Number, FriendlyName, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-HostDisks-Before.txt")

Write-Host "Pre-change VHDX evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Create_VHDX_Skeleton

~~~powershell
# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Create_VHDX_Skeleton
# Purpose:
# Create dynamic, fixed, or differencing VHDX files for Hyper-V VMs.

$VHDXRoot = "D:\Hyper-V\Virtual Hard Disks"
$DiskName = "LAB-WIN-SRV01-Data01.vhdx"
$VHDXPath = Join-Path $VHDXRoot $DiskName

$DiskType = "Dynamic"
$DiskSize = 100GB

$ParentVHDXPath = "D:\Hyper-V\Virtual Hard Disks\Base-WindowsServer.vhdx"

Write-Host "Validating VHDX root path..." -ForegroundColor Cyan

if (-not (Test-Path $VHDXRoot)) {
    throw "VHDX root path does not exist: $VHDXRoot"
}

if (Test-Path $VHDXPath) {
    throw "VHDX already exists: $VHDXPath"
}

Write-Host "Creating VHDX type: $DiskType" -ForegroundColor Cyan

switch ($DiskType) {
    "Dynamic" {
        New-VHD `
            -Path $VHDXPath `
            -SizeBytes $DiskSize `
            -Dynamic
    }

    "Fixed" {
        New-VHD `
            -Path $VHDXPath `
            -SizeBytes $DiskSize `
            -Fixed
    }

    "Differencing" {
        if (-not (Test-Path $ParentVHDXPath)) {
            throw "Parent VHDX does not exist: $ParentVHDXPath"
        }

        New-VHD `
            -Path $VHDXPath `
            -ParentPath $ParentVHDXPath `
            -Differencing
    }

    default {
        throw "Unsupported disk type: $DiskType. Use Dynamic, Fixed, or Differencing."
    }
}

Write-Host "Created VHDX details:" -ForegroundColor Green

Get-VHD -Path $VHDXPath |
    Select-Object Path, VhdFormat, VhdType, FileSize, Size, MinimumSize, ParentPath, LogicalSectorSize, PhysicalSectorSize |
    Format-List
~~~

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Attach_VHDX_To_VM_Skeleton

~~~powershell
# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Attach_VHDX_To_VM_Skeleton
# Purpose:
# Attach a VHDX to a VM as a SCSI data disk or Generation 1 IDE boot disk.

$VMName = "LAB-WIN-SRV01"
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\LAB-WIN-SRV01-Data01.vhdx"

$ControllerType = "SCSI"
$ControllerNumber = 0
$ControllerLocation = 1

Write-Host "Validating VM and VHDX..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if (-not (Test-Path $VHDXPath)) {
    throw "VHDX does not exist: $VHDXPath"
}

Write-Host "Checking whether this VHDX is already attached..." -ForegroundColor Cyan

$ExistingAttachment = Get-VMHardDiskDrive -VMName $VMName |
    Where-Object { $_.Path -eq $VHDXPath }

if ($ExistingAttachment) {
    throw "VHDX is already attached to $VMName at $($ExistingAttachment.ControllerType) $($ExistingAttachment.ControllerNumber):$($ExistingAttachment.ControllerLocation)"
}

Write-Host "Checking controller location availability..." -ForegroundColor Cyan

$LocationInUse = Get-VMHardDiskDrive -VMName $VMName |
    Where-Object {
        $_.ControllerType -eq $ControllerType -and
        $_.ControllerNumber -eq $ControllerNumber -and
        $_.ControllerLocation -eq $ControllerLocation
    }

if ($LocationInUse) {
    throw "Controller location is already in use: $ControllerType $ControllerNumber:$ControllerLocation"
}

Write-Host "Attaching VHDX to VM..." -ForegroundColor Cyan

Add-VMHardDiskDrive `
    -VMName $VMName `
    -ControllerType $ControllerType `
    -ControllerNumber $ControllerNumber `
    -ControllerLocation $ControllerLocation `
    -Path $VHDXPath

Write-Host "Disk attachment after change:" -ForegroundColor Green

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Sort-Object ControllerType, ControllerNumber, ControllerLocation |
    Format-Table -AutoSize

Write-Host "VHDX details:" -ForegroundColor Cyan

Get-VHD -Path $VHDXPath |
    Format-List
~~~

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Resize_VHDX_Skeleton

~~~powershell
# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Resize_VHDX_Skeleton
# Purpose:
# Expand or shrink a VHDX. Expansion is safer. Shrink requires guest partition shrink first.

$VMName = "LAB-WIN-SRV01"
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\LAB-WIN-SRV01-Data01.vhdx"

$ResizeMode = "Expand"
$TargetSize = 150GB

Write-Host "Validating VHDX..." -ForegroundColor Cyan

if (-not (Test-Path $VHDXPath)) {
    throw "VHDX does not exist: $VHDXPath"
}

$VHD = Get-VHD -Path $VHDXPath

Write-Host "Current VHDX state:" -ForegroundColor Cyan

$VHD |
    Select-Object Path, VhdType, FileSize, Size, MinimumSize, Attached |
    Format-List

Write-Host "Checking VM attachment..." -ForegroundColor Cyan

$Attachment = Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
    Where-Object { $_.Path -eq $VHDXPath }

if ($Attachment) {
    Write-Host "Disk is attached to VM: $VMName" -ForegroundColor Yellow
    Write-Host "Controller: $($Attachment.ControllerType) $($Attachment.ControllerNumber):$($Attachment.ControllerLocation)" -ForegroundColor Yellow
}

switch ($ResizeMode) {
    "Expand" {
        Write-Host "Expanding VHDX to $TargetSize..." -ForegroundColor Cyan

        Resize-VHD `
            -Path $VHDXPath `
            -SizeBytes $TargetSize
    }

    "ShrinkToMinimum" {
        Write-Warning "Shrink requires the guest partition to be smaller than the target VHDX size."
        Write-Warning "Confirm guest partition shrink was already completed before running this."

        Resize-VHD `
            -Path $VHDXPath `
            -ToMinimumSize
    }

    "ShrinkToSize" {
        Write-Warning "Shrink requires the guest partition to be smaller than the target VHDX size."
        Write-Warning "Confirm guest partition shrink was already completed before running this."

        Resize-VHD `
            -Path $VHDXPath `
            -SizeBytes $TargetSize
    }

    default {
        throw "Unsupported resize mode: $ResizeMode. Use Expand, ShrinkToMinimum, or ShrinkToSize."
    }
}

Write-Host "VHDX state after resize:" -ForegroundColor Green

Get-VHD -Path $VHDXPath |
    Select-Object Path, VhdType, FileSize, Size, MinimumSize, Attached |
    Format-List
~~~

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Compact_And_Optimize_VHDX_Skeleton

~~~powershell
# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Compact_And_Optimize_VHDX_Skeleton
# Purpose:
# Compact and optimize a dynamic VHDX after guest cleanup.
# Best practice:
# 1. Clean up files inside guest.
# 2. Run guest defrag or retrim if appropriate.
# 3. Shut down VM or detach disk.
# 4. Run Optimize-VHD from host.

$VMName = "LAB-WIN-SRV01"
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\LAB-WIN-SRV01-Data01.vhdx"
$OptimizeMode = "Full"

Write-Host "Validating VHDX..." -ForegroundColor Cyan

if (-not (Test-Path $VHDXPath)) {
    throw "VHDX does not exist: $VHDXPath"
}

$VHD = Get-VHD -Path $VHDXPath

Write-Host "Current VHDX state:" -ForegroundColor Cyan

$VHD |
    Select-Object Path, VhdType, FileSize, Size, MinimumSize, Attached, FragmentationPercentage |
    Format-List

if ($VHD.VhdType -ne "Dynamic") {
    Write-Warning "Compaction is mainly useful for dynamically expanding VHDX files. Current type: $($VHD.VhdType)"
}

Write-Host "Checking whether VHDX is attached to VM..." -ForegroundColor Cyan

$Attachment = Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
    Where-Object { $_.Path -eq $VHDXPath }

if ($Attachment) {
    $VM = Get-VM -Name $VMName

    if ($VM.State -ne "Off") {
        throw "VM is not Off. Shut down the VM before full compaction. Current state: $($VM.State)"
    }
}

Write-Host "Optimizing VHDX using mode: $OptimizeMode" -ForegroundColor Cyan

Optimize-VHD `
    -Path $VHDXPath `
    -Mode $OptimizeMode

Write-Host "VHDX state after optimization:" -ForegroundColor Green

Get-VHD -Path $VHDXPath |
    Select-Object Path, VhdType, FileSize, Size, MinimumSize, Attached, FragmentationPercentage |
    Format-List
~~~

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Mount_Initialize_And_Format_Data_Disk_Skeleton

~~~powershell
# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Mount_Initialize_And_Format_Data_Disk_Skeleton
# Purpose:
# Mount a VHDX on the host, initialize it, create a partition, format it, then dismount it.
# Use this for offline preparation of data disks before attaching to a VM.

$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\LAB-WIN-SRV01-Data01.vhdx"
$DriveLetter = "V"
$FileSystem = "NTFS"
$VolumeLabel = "DATA01"

Write-Host "Validating VHDX..." -ForegroundColor Cyan

if (-not (Test-Path $VHDXPath)) {
    throw "VHDX does not exist: $VHDXPath"
}

Write-Host "Mounting VHDX..." -ForegroundColor Cyan

$MountedVHD = Mount-VHD -Path $VHDXPath -Passthru

Start-Sleep -Seconds 2

$Disk = $MountedVHD | Get-Disk

Write-Host "Mounted disk:" -ForegroundColor Cyan

$Disk |
    Format-List Number, FriendlyName, OperationalStatus, PartitionStyle, Size

if ($Disk.PartitionStyle -eq "RAW") {
    Write-Host "Initializing disk as GPT..." -ForegroundColor Cyan

    Initialize-Disk `
        -Number $Disk.Number `
        -PartitionStyle GPT
}

Write-Host "Creating partition and formatting volume..." -ForegroundColor Cyan

$ExistingPartition = Get-Partition -DiskNumber $Disk.Number -ErrorAction SilentlyContinue |
    Where-Object { $_.Type -ne "Reserved" }

if (-not $ExistingPartition) {
    $Partition = New-Partition `
        -DiskNumber $Disk.Number `
        -UseMaximumSize `
        -DriveLetter $DriveLetter

    Format-Volume `
        -Partition $Partition `
        -FileSystem $FileSystem `
        -NewFileSystemLabel $VolumeLabel `
        -Confirm:$false
}
else {
    Write-Host "Disk already contains a partition. Skipping partition creation." -ForegroundColor Yellow
}

Write-Host "Volume state before dismount:" -ForegroundColor Green

Get-Volume -DriveLetter $DriveLetter -ErrorAction SilentlyContinue |
    Format-List

Write-Host "Dismounting VHDX..." -ForegroundColor Cyan

Dismount-VHD -Path $VHDXPath

Write-Host "VHDX prepared and dismounted: $VHDXPath" -ForegroundColor Green
~~~

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_PostChange_Verification_Skeleton

~~~powershell
# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_PostChange_Verification_Skeleton
# Purpose:
# Verify VHDX details, VM disk attachments, storage usage, and recent Hyper-V events after disk work.

$EvidenceRoot = "C:\Admin\HyperV-VHDX"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$VHDXRoot = "D:\Hyper-V\Virtual Hard Disks"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM disk attachments after change..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path, DiskNumber, ResourcePoolName |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VMHardDiskDrive-After.txt")

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path, DiskNumber, ResourcePoolName |
    Export-Csv -Path (Join-Path $OutputPath "01-VMHardDiskDrive-After.csv") -NoTypeInformation

Write-Host "Capturing attached VHDX details after change..." -ForegroundColor Cyan

$AttachedDisks = Get-VMHardDiskDrive -VMName $VMName

foreach ($Disk in $AttachedDisks) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, MinimumSize, LogicalSectorSize, PhysicalSectorSize, ParentPath, FragmentationPercentage |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("02-VHD-After-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Capturing VHDX folder inventory after change..." -ForegroundColor Cyan

Get-ChildItem $VHDXRoot -Filter "*.vhdx" -ErrorAction SilentlyContinue |
    Select-Object Name, FullName, Length, CreationTime, LastWriteTime |
    Sort-Object Name |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VHDXFolderInventory-After.txt")

Write-Host "Capturing host volume state after change..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Volumes-After.txt")

Write-Host "Checking mounted VHDs..." -ForegroundColor Cyan

Get-VHD -Path (Join-Path $VHDXRoot "*.vhdx") -ErrorAction SilentlyContinue |
    Select-Object Path, Attached, VhdType, FileSize, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VHDXAttachedState-After.txt")

Write-Host "Capturing recent Hyper-V storage related events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "06-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-Worker-RecentEvents.txt") -Encoding UTF8

Write-Host "Post-change VHDX evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms target VM exists and shows state |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Shows all VHDX attachments for the VM |
| `Get-VMHardDiskDrive -VMName '<vm-name>' \| Select ControllerType,ControllerNumber,ControllerLocation,Path` | Hyper-V host | Confirms controller placement |
| `Get-VHD -Path '<vhdx-path>'` | Hyper-V host | Shows VHDX type, size, file size, parent, and attachment state |
| `Test-Path '<vhdx-path>'` | Hyper-V host | Confirms VHDX file exists |
| `Get-ChildItem 'D:\Hyper-V\Virtual Hard Disks' -Filter '*.vhdx'` | Hyper-V host | Lists VHDX inventory |
| `Get-Volume -DriveLetter D` | Hyper-V host | Confirms host storage volume health and free space |
| `Mount-VHD -Path '<vhdx-path>' -Passthru` | Hyper-V host | Mounts VHDX for offline inspection |
| `Get-Disk` | Hyper-V host | Confirms mounted VHDX appears as a disk |
| `Dismount-VHD -Path '<vhdx-path>'` | Hyper-V host | Safely dismounts VHDX |
| `Resize-VHD -Path '<vhdx-path>' -SizeBytes '<size>'` | Hyper-V host | Resizes VHDX to target size |
| `Optimize-VHD -Path '<vhdx-path>' -Mode Full` | Hyper-V host | Compacts and optimizes VHDX |
| `Get-PartitionSupportedSize -DriveLetter <letter>` | Guest VM | Shows valid partition expansion range inside the guest |
| `Resize-Partition -DriveLetter <letter> -Size <size>` | Guest VM | Expands or shrinks guest partition |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks recent VMMS storage errors |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM worker storage errors |

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm VM disk attachments | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>'` | Current disk attachment state is known |
| 2 | Shut down VM if disk change requires offline state | Hyper-V host | `Stop-VM -Name '<vm-name>'` | VM is off |
| 3 | Detach added data disk | Hyper-V host | `Remove-VMHardDiskDrive -VMName '<vm-name>' -ControllerType SCSI -ControllerNumber 0 -ControllerLocation 1` | VHDX is detached from VM |
| 4 | Confirm VHDX file still exists after detach | Hyper-V host | `Test-Path '<vhdx-path>'` | Disk file is preserved |
| 5 | Delete newly created VHDX only if approved | Hyper-V host | `Remove-Item '<vhdx-path>' -Force` | VHDX file is removed |
| 6 | Restore from exported copy if one exists | Hyper-V host | `Copy-Item '<backup-vhdx-path>' '<vhdx-path>'` | Previous VHDX copy is restored |
| 7 | Reattach original VHDX if needed | Hyper-V host | `Add-VMHardDiskDrive -VMName '<vm-name>' -ControllerType '<type>' -ControllerNumber '<number>' -ControllerLocation '<location>' -Path '<vhdx-path>'` | Original disk attachment is restored |
| 8 | Restore guest partition size from backup if shrink was wrong | Guest VM | Use guest backup or disk image restore | Guest volume is recovered |
| 9 | Dismount any mounted VHDX | Hyper-V host | `Dismount-VHD -Path '<vhdx-path>'` | VHDX is not left mounted on host |
| 10 | Start VM after rollback | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts with restored disk state |
| 11 | Capture rollback evidence | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>'; Get-VHD -Path '<vhdx-path>'` | Rollback state is documented |

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `New-VHD` fails | Folder missing, access denied, or insufficient free space | Confirm path, permissions, and volume free space |
| VHDX already exists | Duplicate disk name | Choose a unique file name or verify whether existing disk should be reused |
| `Add-VMHardDiskDrive` fails | Controller location already in use | Run `Get-VMHardDiskDrive` and choose an unused controller location |
| VM cannot boot after disk change | Boot disk detached, wrong controller, or wrong boot order | Reattach boot disk to correct controller and verify firmware or BIOS boot order |
| Generation 1 VM will not boot from SCSI disk | Generation 1 requires IDE boot disk | Attach boot disk to IDE controller |
| Generation 2 VM disk boot issue | Firmware boot order wrong | Use `Set-VMFirmware` to set correct first boot device |
| Guest does not see new data disk | Disk not initialized inside guest or guest needs rescan | Use Disk Management or `Get-Disk`, initialize and format |
| VHDX resize succeeds but guest volume size does not change | Guest partition was not extended | Run `Resize-Partition` inside the guest |
| `Resize-VHD -ToMinimumSize` fails | Guest partition still too large or disk attached in unsupported state | Shrink guest partition first and perform resize offline |
| Compact does not reduce file much | Guest free space was not zeroed, trimmed, or cleaned | Clean up guest, run retrim or defrag as appropriate, then optimize again |
| `Optimize-VHD` fails because disk is in use | VM is running or VHDX is attached or mounted | Shut down VM, detach disk, or dismount VHDX |
| Mounted VHDX does not appear in Disk Management | Mount failed or disk is offline | Use `Mount-VHD -Passthru`, then `Get-Disk` |
| Disk initializes on host but not in guest | Disk was prepared offline but not attached to VM | Attach VHDX to VM and rescan guest disks |
| Differencing disk fails | Parent path missing or parent disk changed | Restore parent disk or recreate differencing disk chain |
| Parent disk was modified after child creation | Differencing disk chain is damaged | Restore from backup or rebuild from known-good parent |
| VHDX file grows unexpectedly | Dynamic disk expands as guest writes data | Clean guest data, compact disk, or move to larger volume |
| Host volume fills up | Dynamic disks, checkpoints, or exports consumed space | Expand host volume, move disks, delete stale checkpoints after verification |
| VM fails to start with storage error | Missing VHDX path, permissions issue, or locked file | Confirm path exists, ACLs are correct, and no host process has the file locked |

# 07_Create_Attach_Resize_And_Compact_VHDX_Disks_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this VHDX task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host storage and volume readiness |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before VHDX cmdlets and VM disk attachment cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Provides the default VHDX path used in this workbook |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates VMs that use the disks managed here |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | Complements storage configuration with VM compute sizing |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Complements disk configuration with VM networking |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Uses attached VHDX disks during guest installation and configuration |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Checkpoints depend heavily on VHDX storage layout and available space |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Export, import, and move workflows include VM disk files |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | VHDX files are core backup and recovery targets |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses VHDX attachment, resize, and storage state during troubleshooting |