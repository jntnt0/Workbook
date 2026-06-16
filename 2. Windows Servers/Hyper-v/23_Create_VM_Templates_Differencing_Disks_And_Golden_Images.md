# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Index
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images.md
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Source_Basis
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Mental_Model
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Planning_Table
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Configuration_Checklist
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_PreChange_Capture_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Golden_Image_Build_VM_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Generalize_And_Seal_Windows_Image_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Parent_VHDX_Preparation_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Differencing_Disk_Deployment_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Template_VM_Clone_And_Customization_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Offline_Servicing_And_Patching_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_PostChange_Verification_Skeleton
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Verification_Commands
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Rollback
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Failure_Checks
23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Related_Labs

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V virtual hard disks | Supports VHDX creation, differencing disks, parent-child chains, and disk optimization |
| Microsoft Learn | Hyper-V VM creation and management | Supports VM template-style creation with `New-VM`, `Set-VM`, and disk attachment |
| Microsoft Learn | Sysprep generalization | Supports Windows golden image sealing before cloning |
| Microsoft Learn | DISM offline servicing | Supports mounting and updating offline Windows images |
| Microsoft Learn | Hyper-V checkpoints | Supports safe lab checkpoints during image build but not as a template replacement |
| Microsoft Learn | Hyper-V export and import | Supports VM portability and template archive workflows |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Golden image | Clean, patched, generalized base OS image used to deploy repeatable VMs |
| Build VM | Temporary VM used to install, patch, configure, and seal the golden image |
| Parent VHDX | Read-only base disk used by one or more differencing disks |
| Differencing disk | Child VHDX that stores changes while reading unchanged blocks from parent VHDX |
| Child VM | VM deployed using a differencing disk or cloned template disk |
| Generalization | Sysprep process that removes unique Windows identity data such as SID and hardware-specific state |
| Sealed image | Golden image after Sysprep shutdown, ready to become parent or clone source |
| Specialized image | Image that has not been generalized; useful only for limited lab reuse |
| Offline servicing | Mounting a VHDX and applying updates, drivers, or packages without booting it |
| Parent chain | Dependency relationship between child AVHDX/VHDX and parent VHDX |
| Read-only parent | Best practice for differencing disk parents to prevent accidental chain corruption |
| Template drift | Changes made to child VMs but not the golden image baseline |
| Clone deployment | Copying a generalized VHDX to create independent VMs |
| Differencing deployment | Creating multiple thin child disks that depend on one parent |
| Rollback boundary | This workbook builds templates and image workflows, not full MDT, SCCM, Intune, or Azure Image Builder pipelines |

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host | `HV01` | `<host-name>` |
| Build VM name | `BUILD-WS2022-GOLD` | `<build-vm-name>` |
| Golden image OS | Windows Server 2022, Windows 11, Ubuntu | `<golden-os>` |
| VM generation | Generation 2 | `<generation>` |
| Secure Boot template | `MicrosoftWindows` | `<secure-boot-template>` |
| vCPU count | `2` | `<vcpu-count>` |
| Startup memory | `4GB` | `<startup-memory>` |
| Dynamic memory | Enabled or disabled | `<yes-no>` |
| Build VM switch | `vSwitch-External` | `<build-switch>` |
| ISO path | `D:\ISO\Windows_Server_2022.iso` | `<iso-path>` |
| Build VM path | `D:\Hyper-V\BuildVMs` | `<build-vm-path>` |
| Golden image folder | `D:\Hyper-V\GoldenImages` | `<golden-image-folder>` |
| Parent VHDX path | `D:\Hyper-V\GoldenImages\WS2022-GOLD.vhdx` | `<parent-vhdx-path>` |
| Differencing disk folder | `D:\Hyper-V\DifferencingDisks` | `<diff-disk-folder>` |
| Child VM folder | `D:\Hyper-V\VMs` | `<child-vm-folder>` |
| Child VM naming prefix | `LAB-WS2022-` | `<child-prefix>` |
| Number of child VMs | `3` | `<child-count>` |
| Sysprep required | Yes or No | `<yes-no>` |
| Offline servicing required | Yes or No | `<yes-no>` |
| Parent VHDX read-only | Yes | `<yes-no>` |
| Template metadata path | `D:\Hyper-V\GoldenImages\metadata` | `<metadata-path>` |
| Evidence path | `C:\Admin\HyperV-Templates` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm template folders exist | Hyper-V host | `New-Item -Path '<path>' -ItemType Directory -Force` | Folder structure exists |
| 3 | Confirm ISO path exists | Hyper-V host | `Test-Path '<iso-path>'` | Installation media is available |
| 4 | Create build VM | Hyper-V host | `New-VM -Name '<build-vm-name>' -Generation 2 -MemoryStartupBytes 4GB -Path '<build-vm-path>' -NewVHDPath '<path>' -NewVHDSizeBytes 80GB -SwitchName '<switch>'` | Build VM is created |
| 5 | Configure build VM CPU and memory | Hyper-V host | `Set-VMProcessor; Set-VMMemory` | Build VM resources are configured |
| 6 | Attach installation ISO | Hyper-V host | `Set-VMDvdDrive -VMName '<build-vm-name>' -Path '<iso-path>'` | ISO is attached |
| 7 | Configure boot order | Hyper-V host | `Set-VMFirmware -VMName '<build-vm-name>' -FirstBootDevice <dvd>` | Build VM boots to ISO |
| 8 | Install guest OS | Build VM console | Manual OS install or unattended install | OS is installed |
| 9 | Patch and configure baseline | Build VM guest | Windows Update, drivers, tools, baseline config | Golden image baseline is prepared |
| 10 | Remove temporary data | Build VM guest | Cleanup commands | Build image is clean |
| 11 | Run Sysprep if Windows clone image is required | Build VM guest | `sysprep.exe /generalize /oobe /shutdown /mode:vm` | Windows shuts down generalized |
| 12 | Copy or move sealed VHDX to golden image folder | Hyper-V host | `Copy-Item '<build-vhdx>' '<parent-vhdx-path>'` | Parent VHDX is created |
| 13 | Optimize golden VHDX | Hyper-V host | `Optimize-VHD -Path '<parent-vhdx-path>' -Mode Full` | VHDX is compacted |
| 14 | Mark parent VHDX read-only | Hyper-V host | `(Get-Item '<parent-vhdx-path>').IsReadOnly = $true` | Parent VHDX is protected from writes |
| 15 | Create differencing disks | Hyper-V host | `New-VHD -Path '<child.vhdx>' -ParentPath '<parent-vhdx-path>' -Differencing` | Child differencing disks are created |
| 16 | Create child VMs from differencing disks | Hyper-V host | `New-VM -Name '<child-vm>' -Generation 2 -VHDPath '<child.vhdx>' -SwitchName '<switch>'` | Child VMs are created |
| 17 | Boot child VM and complete OOBE | Child VM guest | Console or unattend workflow | Child VM gets unique identity |
| 18 | Configure child hostname and network | Child VM guest | PowerShell Direct, unattend, or manual | Child VM is customized |
| 19 | Verify parent-child chain | Hyper-V host | `Get-VHD -Path '<child.vhdx>'` | Parent path points to golden image |
| 20 | Capture template metadata | Hyper-V host | Export inventory to JSON/CSV | Template version is documented |
| 21 | Capture post-change evidence | Hyper-V host | Run post-change verification skeleton | Template workflow evidence is saved |

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_PreChange_Capture_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_PreChange_Capture_Skeleton
# Purpose:
# Capture host, folder, VM, VHDX, ISO, and Hyper-V baseline before template and golden image work.

$EvidenceRoot = "C:\Admin\HyperV-Templates"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BuildVMName = "BUILD-WS2022-GOLD"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$BuildVMName-$Timestamp"

$PathsToCheck = @(
    "D:\ISO",
    "D:\Hyper-V\BuildVMs",
    "D:\Hyper-V\GoldenImages",
    "D:\Hyper-V\DifferencingDisks",
    "D:\Hyper-V\VMs"
)

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing host identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Host-ComputerInfo.txt")

Write-Host "Capturing Hyper-V feature and service state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperV-Features.txt")

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-HyperV-Services.txt")

Write-Host "Capturing Hyper-V host defaults..." -ForegroundColor Cyan

Get-VMHost |
    Select-Object ComputerName, VirtualMachinePath, VirtualHardDiskPath, EnableEnhancedSessionMode, NumaSpanningEnabled |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMHost-Defaults.txt")

Write-Host "Capturing existing VM inventory..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VM-Inventory.txt")

Write-Host "Capturing existing VHDX inventory under planned paths..." -ForegroundColor Cyan

foreach ($Path in $PathsToCheck) {
    [PSCustomObject]@{
        Path = $Path
        Exists = Test-Path $Path
    } |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath ("06-PathCheck-" + (($Path -replace "[:\\]", "_")) + ".txt")) -Encoding UTF8

    if (Test-Path $Path) {
        Get-ChildItem -Path $Path -Recurse -Include "*.vhdx","*.avhdx","*.iso" -ErrorAction SilentlyContinue |
            Select-Object FullName, Length, CreationTime, LastWriteTime, Attributes |
            Sort-Object FullName |
            Out-File -FilePath (Join-Path $OutputPath ("07-Inventory-" + (($Path -replace "[:\\]", "_")) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Capturing existing virtual switches..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-VMSwitches.txt")

Write-Host "Capturing recent Hyper-V events..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "System"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', "-"

    Get-WinEvent -LogName $Log -MaxEvents 75 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "09-$SafeLog-Before.txt") -Encoding UTF8
}

Write-Host "Pre-change template evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Golden_Image_Build_VM_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Golden_Image_Build_VM_Skeleton
# Purpose:
# Create a temporary build VM used to install, patch, configure, and seal a golden image.

$BuildVMName = "BUILD-WS2022-GOLD"
$Generation = 2
$StartupMemory = 4GB
$ProcessorCount = 2
$BuildVMRoot = "D:\Hyper-V\BuildVMs"
$BuildVMPath = Join-Path $BuildVMRoot $BuildVMName
$BuildVHDXPath = Join-Path $BuildVMPath "$BuildVMName.vhdx"
$VHDXSize = 80GB
$SwitchName = "vSwitch-External"
$ISOPath = "D:\ISO\Windows_Server_2022.iso"

Write-Host "Validating paths and switch..." -ForegroundColor Cyan

New-Item -Path $BuildVMPath -ItemType Directory -Force | Out-Null

if (-not (Test-Path $ISOPath)) {
    throw "ISO path not found: $ISOPath"
}

Get-VMSwitch -Name $SwitchName -ErrorAction Stop | Out-Null

if (Get-VM -Name $BuildVMName -ErrorAction SilentlyContinue) {
    throw "Build VM already exists: $BuildVMName"
}

Write-Host "Creating build VM..." -ForegroundColor Cyan

New-VM `
    -Name $BuildVMName `
    -Generation $Generation `
    -MemoryStartupBytes $StartupMemory `
    -Path $BuildVMPath `
    -NewVHDPath $BuildVHDXPath `
    -NewVHDSizeBytes $VHDXSize `
    -SwitchName $SwitchName

Write-Host "Configuring build VM resources..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $BuildVMName `
    -Count $ProcessorCount

Set-VMMemory `
    -VMName $BuildVMName `
    -DynamicMemoryEnabled $true `
    -MinimumBytes 2GB `
    -StartupBytes $StartupMemory `
    -MaximumBytes 8GB

Set-VM `
    -Name $BuildVMName `
    -CheckpointType Production `
    -AutomaticCheckpointsEnabled $false `
    -AutomaticStartAction Nothing `
    -AutomaticStopAction ShutDown

Write-Host "Attaching ISO..." -ForegroundColor Cyan

Set-VMDvdDrive `
    -VMName $BuildVMName `
    -Path $ISOPath

Write-Host "Configuring firmware boot order..." -ForegroundColor Cyan

$DVDDrive = Get-VMDvdDrive -VMName $BuildVMName

Set-VMFirmware `
    -VMName $BuildVMName `
    -EnableSecureBoot On `
    -SecureBootTemplate MicrosoftWindows `
    -FirstBootDevice $DVDDrive

Write-Host "Build VM created. Start VM and install OS." -ForegroundColor Green

Start-VM -Name $BuildVMName

Write-Host "Open console with:" -ForegroundColor Yellow
Write-Host "vmconnect localhost `"$BuildVMName`"" -ForegroundColor Yellow
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Generalize_And_Seal_Windows_Image_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Generalize_And_Seal_Windows_Image_Skeleton
# Purpose:
# Run inside the Windows build VM before capturing the golden image.
# This generalizes the OS and shuts down the VM.

# Run this inside the build VM as local administrator after:
# - Windows updates are installed
# - Required base tools are installed
# - Temporary files are removed
# - Local build notes are saved outside the VM
# - No domain join remains unless intentionally building a specialized lab image
# - Recovery keys, secrets, installer caches, and user-specific data are removed

$SysprepPath = "C:\Windows\System32\Sysprep\Sysprep.exe"

Write-Host "Capturing pre-Sysprep guest summary..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, WindowsProductName, WindowsVersion, OsBuildNumber, CsDomain, CsPartOfDomain |
    Format-List

Write-Host "Clearing common temporary locations..." -ForegroundColor Cyan

Remove-Item "C:\Windows\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue

Write-Host "Checking Sysprep path..." -ForegroundColor Cyan

if (-not (Test-Path $SysprepPath)) {
    throw "Sysprep not found at: $SysprepPath"
}

Write-Warning "Sysprep will generalize this Windows installation and shut down the VM."
Write-Warning "Do not boot the parent image after sealing unless you intend to rebuild and reseal."

Start-Process `
    -FilePath $SysprepPath `
    -ArgumentList "/generalize /oobe /shutdown /mode:vm" `
    -Wait
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Parent_VHDX_Preparation_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Parent_VHDX_Preparation_Skeleton
# Purpose:
# Copy sealed build VHDX to golden image library, optimize it, and mark it read-only for differencing disk use.

$BuildVMName = "BUILD-WS2022-GOLD"
$GoldenImageName = "WS2022-GOLD-v1"
$GoldenImageFolder = "D:\Hyper-V\GoldenImages"
$MetadataFolder = Join-Path $GoldenImageFolder "metadata"

$BuildVHDXPath = "D:\Hyper-V\BuildVMs\BUILD-WS2022-GOLD\BUILD-WS2022-GOLD.vhdx"
$ParentVHDXPath = Join-Path $GoldenImageFolder "$GoldenImageName.vhdx"

Write-Host "Validating build VM is off..." -ForegroundColor Cyan

$BuildVM = Get-VM -Name $BuildVMName -ErrorAction Stop

if ($BuildVM.State -ne "Off") {
    throw "Build VM must be Off before copying sealed VHDX. Current state: $($BuildVM.State)"
}

Write-Host "Creating golden image folders..." -ForegroundColor Cyan

New-Item -Path $GoldenImageFolder -ItemType Directory -Force | Out-Null
New-Item -Path $MetadataFolder -ItemType Directory -Force | Out-Null

Write-Host "Validating build VHDX..." -ForegroundColor Cyan

if (-not (Test-Path $BuildVHDXPath)) {
    throw "Build VHDX not found: $BuildVHDXPath"
}

Get-VHD -Path $BuildVHDXPath |
    Format-List *

Write-Host "Copying sealed VHDX to golden image library..." -ForegroundColor Cyan

Copy-Item `
    -Path $BuildVHDXPath `
    -Destination $ParentVHDXPath `
    -Force

Write-Host "Optimizing golden parent VHDX..." -ForegroundColor Cyan

Optimize-VHD `
    -Path $ParentVHDXPath `
    -Mode Full

Write-Host "Marking parent VHDX read-only..." -ForegroundColor Cyan

(Get-Item $ParentVHDXPath).IsReadOnly = $true

Write-Host "Creating metadata file..." -ForegroundColor Cyan

$Metadata = [PSCustomObject]@{
    GoldenImageName = $GoldenImageName
    ParentVHDXPath = $ParentVHDXPath
    BuildVMName = $BuildVMName
    BuildVHDXPath = $BuildVHDXPath
    CreatedOn = Get-Date
    Host = $env:COMPUTERNAME
    OSFamily = "Windows Server"
    Generalized = $true
    Notes = "Sysprep generalized golden image for Hyper-V differencing disks and clone deployments."
}

$Metadata |
    ConvertTo-Json -Depth 5 |
    Out-File -FilePath (Join-Path $MetadataFolder "$GoldenImageName.json") -Encoding UTF8

Write-Host "Golden parent VHDX state:" -ForegroundColor Green

Get-Item $ParentVHDXPath |
    Select-Object FullName, Length, IsReadOnly, CreationTime, LastWriteTime |
    Format-List

Get-VHD -Path $ParentVHDXPath |
    Select-Object Path, VhdFormat, VhdType, FileSize, Size, ParentPath, Attached |
    Format-List
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Differencing_Disk_Deployment_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Differencing_Disk_Deployment_Skeleton
# Purpose:
# Deploy multiple child VMs using differencing disks from a read-only golden parent VHDX.

$ParentVHDXPath = "D:\Hyper-V\GoldenImages\WS2022-GOLD-v1.vhdx"
$ChildVMRoot = "D:\Hyper-V\VMs"
$DiffDiskRoot = "D:\Hyper-V\DifferencingDisks"
$SwitchName = "vSwitch-External"

$ChildPrefix = "LAB-WS2022-"
$StartNumber = 1
$ChildCount = 3

$StartupMemory = 4GB
$ProcessorCount = 2
$Generation = 2

Write-Host "Validating parent VHDX..." -ForegroundColor Cyan

if (-not (Test-Path $ParentVHDXPath)) {
    throw "Parent VHDX not found: $ParentVHDXPath"
}

$ParentItem = Get-Item $ParentVHDXPath

if (-not $ParentItem.IsReadOnly) {
    Write-Warning "Parent VHDX is not read-only. Marking it read-only."
    $ParentItem.IsReadOnly = $true
}

Get-VHD -Path $ParentVHDXPath |
    Format-List *

Write-Host "Validating vSwitch..." -ForegroundColor Cyan

Get-VMSwitch -Name $SwitchName -ErrorAction Stop | Out-Null

Write-Host "Creating child VMs..." -ForegroundColor Cyan

for ($i = $StartNumber; $i -lt ($StartNumber + $ChildCount); $i++) {
    $ChildVMName = "{0}{1:D2}" -f $ChildPrefix, $i
    $ChildVMPath = Join-Path $ChildVMRoot $ChildVMName
    $ChildDiskFolder = Join-Path $DiffDiskRoot $ChildVMName
    $ChildVHDXPath = Join-Path $ChildDiskFolder "$ChildVMName.vhdx"

    Write-Host "Creating child VM: $ChildVMName" -ForegroundColor Yellow

    if (Get-VM -Name $ChildVMName -ErrorAction SilentlyContinue) {
        Write-Warning "VM already exists, skipping: $ChildVMName"
        continue
    }

    New-Item -Path $ChildVMPath -ItemType Directory -Force | Out-Null
    New-Item -Path $ChildDiskFolder -ItemType Directory -Force | Out-Null

    New-VHD `
        -Path $ChildVHDXPath `
        -ParentPath $ParentVHDXPath `
        -Differencing

    New-VM `
        -Name $ChildVMName `
        -Generation $Generation `
        -MemoryStartupBytes $StartupMemory `
        -Path $ChildVMPath `
        -VHDPath $ChildVHDXPath `
        -SwitchName $SwitchName

    Set-VMProcessor `
        -VMName $ChildVMName `
        -Count $ProcessorCount

    Set-VM `
        -Name $ChildVMName `
        -CheckpointType Production `
        -AutomaticCheckpointsEnabled $false `
        -AutomaticStartAction Nothing `
        -AutomaticStopAction ShutDown

    if ($Generation -eq 2) {
        $BootDisk = Get-VMHardDiskDrive -VMName $ChildVMName | Select-Object -First 1

        Set-VMFirmware `
            -VMName $ChildVMName `
            -EnableSecureBoot On `
            -SecureBootTemplate MicrosoftWindows `
            -FirstBootDevice $BootDisk
    }

    Write-Host "Created: $ChildVMName" -ForegroundColor Green
}

Write-Host "Child VM inventory:" -ForegroundColor Cyan

Get-VM |
    Where-Object { $_.Name -like "$ChildPrefix*" } |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, Path |
    Format-Table -AutoSize

Write-Host "Differencing disk chain check:" -ForegroundColor Cyan

Get-ChildItem -Path $DiffDiskRoot -Recurse -Filter "*.vhdx" |
    ForEach-Object {
        Get-VHD -Path $_.FullName |
            Select-Object Path, VhdType, ParentPath, FileSize, Size
    } |
    Format-List
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Template_VM_Clone_And_Customization_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Template_VM_Clone_And_Customization_Skeleton
# Purpose:
# Deploy independent clone VMs by copying the generalized parent VHDX instead of using differencing disks.

$ParentVHDXPath = "D:\Hyper-V\GoldenImages\WS2022-GOLD-v1.vhdx"
$CloneVMRoot = "D:\Hyper-V\VMs"
$SwitchName = "vSwitch-External"

$ClonePrefix = "CLONE-WS2022-"
$StartNumber = 1
$CloneCount = 2

$StartupMemory = 4GB
$ProcessorCount = 2
$Generation = 2

Write-Host "Validating parent VHDX..." -ForegroundColor Cyan

if (-not (Test-Path $ParentVHDXPath)) {
    throw "Parent VHDX not found: $ParentVHDXPath"
}

Write-Host "Creating independent clone VMs..." -ForegroundColor Cyan

for ($i = $StartNumber; $i -lt ($StartNumber + $CloneCount); $i++) {
    $CloneVMName = "{0}{1:D2}" -f $ClonePrefix, $i
    $CloneVMPath = Join-Path $CloneVMRoot $CloneVMName
    $CloneVHDXPath = Join-Path $CloneVMPath "$CloneVMName.vhdx"

    Write-Host "Creating clone VM: $CloneVMName" -ForegroundColor Yellow

    if (Get-VM -Name $CloneVMName -ErrorAction SilentlyContinue) {
        Write-Warning "VM already exists, skipping: $CloneVMName"
        continue
    }

    New-Item -Path $CloneVMPath -ItemType Directory -Force | Out-Null

    Copy-Item `
        -Path $ParentVHDXPath `
        -Destination $CloneVHDXPath `
        -Force

    (Get-Item $CloneVHDXPath).IsReadOnly = $false

    New-VM `
        -Name $CloneVMName `
        -Generation $Generation `
        -MemoryStartupBytes $StartupMemory `
        -Path $CloneVMPath `
        -VHDPath $CloneVHDXPath `
        -SwitchName $SwitchName

    Set-VMProcessor `
        -VMName $CloneVMName `
        -Count $ProcessorCount

    Set-VM `
        -Name $CloneVMName `
        -CheckpointType Production `
        -AutomaticCheckpointsEnabled $false `
        -AutomaticStartAction Nothing `
        -AutomaticStopAction ShutDown

    if ($Generation -eq 2) {
        $BootDisk = Get-VMHardDiskDrive -VMName $CloneVMName | Select-Object -First 1

        Set-VMFirmware `
            -VMName $CloneVMName `
            -EnableSecureBoot On `
            -SecureBootTemplate MicrosoftWindows `
            -FirstBootDevice $BootDisk
    }

    Write-Host "Created independent clone: $CloneVMName" -ForegroundColor Green
}

Write-Host "Clone VM inventory:" -ForegroundColor Cyan

Get-VM |
    Where-Object { $_.Name -like "$ClonePrefix*" } |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, Path |
    Format-Table -AutoSize
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Offline_Servicing_And_Patching_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Offline_Servicing_And_Patching_Skeleton
# Purpose:
# Mount a golden VHDX offline for inspection or servicing.
# Do not patch a parent VHDX that active differencing children depend on unless the chain impact is understood.
# Best practice: create a new versioned parent image instead of modifying an in-use parent.

$ParentVHDXPath = "D:\Hyper-V\GoldenImages\WS2022-GOLD-v1.vhdx"
$MountReadOnly = $true
$PackagePath = "D:\Updates\update.cab"

Write-Host "Validating parent VHDX..." -ForegroundColor Cyan

if (-not (Test-Path $ParentVHDXPath)) {
    throw "Parent VHDX not found: $ParentVHDXPath"
}

Write-Warning "If child differencing disks depend on this parent, avoid modifying it in place."
Write-Warning "Create a new versioned image for patching whenever possible."

Write-Host "Mounting VHDX..." -ForegroundColor Cyan

if ($MountReadOnly) {
    Mount-VHD -Path $ParentVHDXPath -ReadOnly
}
else {
    (Get-Item $ParentVHDXPath).IsReadOnly = $false
    Mount-VHD -Path $ParentVHDXPath
}

Start-Sleep -Seconds 3

Write-Host "Finding mounted volume..." -ForegroundColor Cyan

$MountedDisk = Get-DiskImage -ImagePath $ParentVHDXPath | Get-Disk
$Volumes = $MountedDisk | Get-Partition | Get-Volume

$Volumes |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, SizeRemaining, Size |
    Format-Table -AutoSize

$WindowsVolume = $Volumes |
    Where-Object { $_.DriveLetter } |
    Select-Object -First 1

if (-not $WindowsVolume) {
    throw "No mounted drive letter found. Assign a drive letter before servicing."
}

$MountRoot = "$($WindowsVolume.DriveLetter):\"

Write-Host "Mounted root: $MountRoot" -ForegroundColor Green

Write-Host "Optional DISM inspection:" -ForegroundColor Cyan

dism.exe /Image:$MountRoot /Get-Packages

if ((Test-Path $PackagePath) -and -not $MountReadOnly) {
    Write-Host "Adding package to offline image..." -ForegroundColor Cyan

    dism.exe /Image:$MountRoot /Add-Package /PackagePath:$PackagePath
}
else {
    Write-Host "Package add skipped. Either package not found or image mounted read-only." -ForegroundColor Yellow
}

Write-Host "Dismounting VHDX..." -ForegroundColor Cyan

if ($MountReadOnly) {
    Dismount-VHD -Path $ParentVHDXPath
}
else {
    Dismount-VHD -Path $ParentVHDXPath
    Optimize-VHD -Path $ParentVHDXPath -Mode Full
    (Get-Item $ParentVHDXPath).IsReadOnly = $true
}

Write-Host "Offline servicing workflow completed." -ForegroundColor Green
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_PostChange_Verification_Skeleton

~~~powershell
# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_PostChange_Verification_Skeleton
# Purpose:
# Capture golden image, parent VHDX, differencing disk chain, template metadata, child VM, and event evidence after deployment.

$EvidenceRoot = "C:\Admin\HyperV-Templates"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

$GoldenImageFolder = "D:\Hyper-V\GoldenImages"
$DiffDiskRoot = "D:\Hyper-V\DifferencingDisks"
$ChildVMPrefix = "LAB-WS2022-"

$OutputPath = Join-Path $EvidenceRoot "PostChange-Templates-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing golden image folder inventory..." -ForegroundColor Cyan

if (Test-Path $GoldenImageFolder) {
    Get-ChildItem -Path $GoldenImageFolder -Recurse -ErrorAction SilentlyContinue |
        Select-Object FullName, Length, IsReadOnly, CreationTime, LastWriteTime, Attributes |
        Sort-Object FullName |
        Out-File -FilePath (Join-Path $OutputPath "01-GoldenImageFolderInventory.txt") -Encoding UTF8
}

Write-Host "Capturing golden parent VHDX metadata..." -ForegroundColor Cyan

Get-ChildItem -Path $GoldenImageFolder -Filter "*.vhdx" -ErrorAction SilentlyContinue |
    ForEach-Object {
        $File = $_

        Get-VHD -Path $File.FullName |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, ParentPath, Attached
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "02-GoldenParentVHDXMetadata.txt")

Write-Host "Capturing differencing disk chain metadata..." -ForegroundColor Cyan

if (Test-Path $DiffDiskRoot) {
    Get-ChildItem -Path $DiffDiskRoot -Recurse -Filter "*.vhdx" -ErrorAction SilentlyContinue |
        ForEach-Object {
            $File = $_

            try {
                $VHD = Get-VHD -Path $File.FullName -ErrorAction Stop

                [PSCustomObject]@{
                    Path = $VHD.Path
                    VhdType = $VHD.VhdType
                    FileSize = $VHD.FileSize
                    Size = $VHD.Size
                    ParentPath = $VHD.ParentPath
                    ParentExists = if ($VHD.ParentPath) { Test-Path $VHD.ParentPath } else { $null }
                    Attached = $VHD.Attached
                }
            }
            catch {
                [PSCustomObject]@{
                    Path = $File.FullName
                    VhdType = "UnableToRead"
                    FileSize = $File.Length
                    Size = $null
                    ParentPath = $null
                    ParentExists = $false
                    Attached = $null
                }
            }
        } |
        Export-Csv -Path (Join-Path $OutputPath "03-DifferencingDiskChain.csv") -NoTypeInformation
}

Write-Host "Capturing child VM inventory..." -ForegroundColor Cyan

Get-VM |
    Where-Object { $_.Name -like "$ChildVMPrefix*" } |
    Sort-Object Name |
    Select-Object Name, State, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-ChildVMInventory.txt")

Write-Host "Capturing child VM disks..." -ForegroundColor Cyan

Get-VM |
    Where-Object { $_.Name -like "$ChildVMPrefix*" } |
    ForEach-Object {
        Get-VMHardDiskDrive -VMName $_.Name |
            Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path
    } |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-ChildVMDisks.txt")

Write-Host "Capturing child VM network adapters..." -ForegroundColor Cyan

Get-VM |
    Where-Object { $_.Name -like "$ChildVMPrefix*" } |
    ForEach-Object {
        Get-VMNetworkAdapter -VMName $_.Name |
            Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "06-ChildVMNetworkAdapters.txt")

Write-Host "Capturing Hyper-V events..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "System"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', "-"

    Get-WinEvent -LogName $Log -MaxEvents 100 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "07-$SafeLog-After.txt") -Encoding UTF8
}

Write-Host "Post-change template evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<build-vm-name>'` | Hyper-V host | Confirms build VM state |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Shows attached VHDX path |
| `Get-VHD -Path '<parent-vhdx-path>'` | Hyper-V host | Shows parent VHDX type, size, and metadata |
| `Get-VHD -Path '<child-vhdx-path>'` | Hyper-V host | Shows child differencing disk and parent path |
| `Test-Path '<parent-vhdx-path>'` | Hyper-V host | Confirms golden image exists |
| `(Get-Item '<parent-vhdx-path>').IsReadOnly` | Hyper-V host | Confirms parent VHDX is protected |
| `New-VHD -Path '<child-vhdx>' -ParentPath '<parent-vhdx>' -Differencing` | Hyper-V host | Creates differencing disk |
| `New-VM -Name '<child-vm>' -Generation 2 -VHDPath '<child-vhdx>'` | Hyper-V host | Creates VM from child disk |
| `Set-VMFirmware -VMName '<vm-name>' -FirstBootDevice '<disk>'` | Hyper-V host | Sets Generation 2 boot order |
| `Optimize-VHD -Path '<vhdx-path>' -Mode Full` | Hyper-V host | Compacts VHDX |
| `Mount-VHD -Path '<vhdx-path>' -ReadOnly` | Hyper-V host | Mounts VHDX for inspection |
| `Dismount-VHD -Path '<vhdx-path>'` | Hyper-V host | Dismounts VHDX |
| `dism.exe /Image:<mount-root> /Get-Packages` | Hyper-V host | Lists offline Windows packages |
| `sysprep.exe /generalize /oobe /shutdown /mode:vm` | Windows build VM | Generalizes and shuts down Windows image |
| `Start-VM -Name '<child-vm>'` | Hyper-V host | Tests child VM startup |
| `Invoke-Command -VMName '<child-vm>' -Credential (Get-Credential) -ScriptBlock { hostname }` | Hyper-V host | Confirms child VM guest management |
| `Get-VMNetworkAdapter -VMName '<child-vm>'` | Hyper-V host | Confirms child VM network adapter |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks VMMS events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM runtime events |

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop child VMs created from template | Hyper-V host | `Get-VM -Name '<prefix>*' \| Stop-VM -TurnOff` | Child VMs are powered off |
| 2 | Remove child VM registration if lab cleanup requires it | Hyper-V host | `Remove-VM -Name '<child-vm>' -Force` | Child VM object is removed |
| 3 | Delete child differencing disk only after VM removal | Hyper-V host | `Remove-Item '<child-vhdx-path>' -Force` | Child disk is removed |
| 4 | Delete child VM folder if approved | Hyper-V host | `Remove-Item '<child-vm-folder>' -Recurse -Force` | Child VM files are removed |
| 5 | Remove independent clone VM if required | Hyper-V host | `Remove-VM -Name '<clone-vm>' -Force` | Clone VM object is removed |
| 6 | Delete independent clone VHDX if approved | Hyper-V host | `Remove-Item '<clone-vhdx-path>' -Force` | Clone disk is removed |
| 7 | Remove build VM if image capture is complete | Hyper-V host | `Remove-VM -Name '<build-vm-name>' -Force` | Build VM object is removed |
| 8 | Delete build VM folder if approved | Hyper-V host | `Remove-Item '<build-vm-folder>' -Recurse -Force` | Build files are removed |
| 9 | Remove golden parent only if no child depends on it | Hyper-V host | Verify `Get-VHD` parent paths first, then `Remove-Item '<parent-vhdx-path>'` | Parent is removed safely |
| 10 | Clear read-only flag if image must be rebuilt | Hyper-V host | `(Get-Item '<parent-vhdx-path>').IsReadOnly = $false` | Parent can be modified |
| 11 | Restore original template metadata | Hyper-V host | Copy previous metadata JSON back to metadata folder | Template catalog returns to baseline |
| 12 | Capture rollback evidence | Hyper-V host | Run post-change verification skeleton | Rollback state is documented |

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Child VM fails to start | Parent VHDX missing, moved, renamed, or modified | Restore parent path or recreate child disk from valid parent |
| `Get-VHD` shows broken parent path | Parent disk path changed after child creation | Move parent back or repair by recreating child from correct parent |
| Parent VHDX changed accidentally | Parent was not read-only | Restore parent from backup and mark read-only |
| All child VMs fail after parent change | Shared parent chain corrupted | Restore parent VHDX from known-good golden image backup |
| Sysprep fails | AppX package issue, pending reboot, domain join, or unsupported state | Resolve Sysprep log errors and rebuild image if needed |
| Cloned VMs have duplicate identity | Image was not generalized | Run Sysprep on build image and redeploy clones |
| Child VMs ask for OOBE repeatedly | Generalized image not customized by unattend or manual setup incomplete | Complete OOBE or add unattend workflow |
| Child VM boots to ISO | DVD still first boot device | Eject ISO and set VHDX as first boot device |
| Generation 2 child VM will not boot | Secure Boot template or boot order issue | Set correct Secure Boot template and first boot device |
| Differencing disk grows too large | High write churn in child VM | Use full clone for heavy-write workloads |
| Parent image patching breaks children | Parent was modified while children depended on it | Version new parent images instead of modifying active parent |
| Offline servicing cannot find Windows volume | VHDX mounted without drive letter or wrong partition selected | Assign drive letter or identify OS partition |
| `Optimize-VHD` fails | VHDX mounted, VM running, or disk locked | Shut down VM and dismount VHDX |
| Build VM cannot boot after install | Firmware boot order wrong or OS not installed to disk | Reset boot device to VHDX |
| Build VM has temporary secrets | Cleanup was skipped before capture | Remove credentials, keys, temp files, logs, and installer caches |
| Guest activation issues | Cloned image licensing or activation mismatch | Use proper volume licensing or lab-appropriate activation process |
| Child VM network unavailable | Wrong switch, VLAN, or adapter state | Reconnect adapter and validate VLAN |
| PowerShell Direct fails to child VM | Guest not booted, credentials wrong, or unsupported guest | Use console, validate guest state and credentials |
| Template metadata is missing | Metadata file was not generated | Recreate JSON metadata from image inventory |
| Cleanup deletes parent used by children | Parent dependency was not checked | Always inspect child `ParentPath` before deleting parent |

# 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images_Related_Labs

| Lab                                                                           | Relationship                                                                      |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| 00_Hyper-V_Index.md                                                           | Places this template and image workflow in the full Hyper-V suite                 |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Confirms host readiness before building image libraries                           |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Required before template VM and VHDX cmdlets are available                        |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md                       | Default VM and VHDX paths influence template library structure                    |
| 04_Create_And_Configure_Virtual_Switches.md                                   | Build and child VMs need correct virtual switch assignment                        |
| 05_Create_Generation_1_And_Generation_2_VMs.md                                | Template VM creation builds on Gen1 and Gen2 VM workflows                         |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md                         | Template defaults should include reasonable CPU and memory baselines              |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md                             | Differencing disks and parent VHDX chains rely on VHDX management skills          |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md                   | Child VMs inherit or require correct adapter and VLAN configuration               |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md                     | Golden image build depends on a clean guest OS and integration services           |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md                | Checkpoints can help during build but are not a template replacement              |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md                 | Template build and deployment require lifecycle and export/import skills          |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md                        | PowerShell Direct helps customize child Windows VMs after first boot              |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md                              | Golden images and parents should be backed up before deployment                   |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md         | Monitoring confirms child VM deployment and template workflow health              |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Troubleshooting focuses on parent paths, child disks, boot order, and VMMS events |
| 22_Configure_Nested_Virtualization.md                                         | Nested lab environments benefit from repeatable templates and golden images       |