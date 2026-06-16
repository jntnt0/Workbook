# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Index
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration.md
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Source_Basis
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Mental_Model
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Planning_Table
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Configuration_Checklist
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_PreChange_Capture_Skeleton
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_GPU_Capability_Discovery_Skeleton
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_GPU_Partitioning_GPU-P_Skeleton
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_DDA_GPU_Passthrough_Skeleton
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Guest_Driver_And_Acceleration_Skeleton
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Remote_Graphics_And_RDS_Policy_Skeleton
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_PostChange_Verification_Skeleton
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Verification_Commands
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Rollback
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Failure_Checks
21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Related_Labs

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V Discrete Device Assignment for graphics devices | Supports DDA GPU passthrough workflow, VM preparation, PCI location path, dismount, assignment, removal, and host remount |
| Microsoft Learn | Hyper-V GPU partitioning cmdlets | Supports GPU-P inventory and VM partition adapter assignment where supported |
| Microsoft Learn | Hyper-V VM security and device assignment | Supports compatibility, isolation, and security planning for hardware-backed acceleration |
| Microsoft Learn | Hyper-V PowerShell `Get-VMHostPartitionableGpu`, `Add-VMGpuPartitionAdapter`, `Set-VMGpuPartitionAdapter`, `Remove-VMGpuPartitionAdapter` | Supports GPU partitioning operations |
| Microsoft Learn | Hyper-V PowerShell `Dismount-VMHostAssignableDevice`, `Add-VMAssignableDevice`, `Remove-VMAssignableDevice`, `Mount-VMHostAssignableDevice` | Supports DDA passthrough operations |
| Microsoft Learn | Remote Desktop Services graphics policies | Supports guest policy checks when GPU acceleration must be visible through RDP |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Mental_Model

| Concept | Operational Meaning |
|---|---|
| GPU acceleration | Using GPU resources inside a VM for graphics, compute, rendering, AI, CAD, media, or RDS workloads |
| GPU-P | GPU partitioning, where a supported GPU is divided into assignable partitions for one or more VMs |
| DDA | Discrete Device Assignment, where an entire PCIe device is passed directly into a VM |
| Partitionable GPU | GPU that the Hyper-V host can divide into GPU partitions |
| GPU partition adapter | VM adapter object that represents assigned GPU-P resources |
| DDA location path | PCI location path used to dismount a device from the host and assign it to a VM |
| Guest driver | Vendor driver installed inside the VM so the guest OS can use the assigned GPU |
| Host driver | Vendor driver installed on the Hyper-V host so the device is visible and manageable |
| MMIO space | Memory-mapped I/O address space required by PCIe passthrough devices |
| Guest controlled cache types | VM setting often required for high-performance DDA graphics scenarios |
| SR-IOV relationship | Some hardware paths depend on firmware, IOMMU, and PCIe virtualization features |
| Live migration tradeoff | Hardware assignment can reduce or eliminate mobility depending on GPU mode and host support |
| Cluster compatibility | GPU acceleration must be validated per-node before clustered VM placement |
| RDS graphics policy | Guest policy may be required so RDP sessions use hardware graphics adapters |
| Rollback boundary | This workbook configures GPU acceleration paths, not vendor licensing or full VDI platform design |

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `GPU-VM01` | `<vm-name>` |
| Guest OS | Windows Server 2022, Windows 11 | `<guest-os>` |
| GPU model | NVIDIA A-series, AMD, Intel | `<gpu-model>` |
| GPU vendor | NVIDIA, AMD, Intel | `<gpu-vendor>` |
| GPU driver version on host | `x.x.x.x` | `<host-driver-version>` |
| GPU driver version in guest | `x.x.x.x` | `<guest-driver-version>` |
| Acceleration mode | GPU-P, DDA, None | `<gpu-mode>` |
| Workload type | RDS, VDI, AI, rendering, CAD, media | `<workload-type>` |
| VM generation | Generation 2 preferred | `<vm-generation>` |
| vCPU count | `4` | `<vcpu-count>` |
| Startup memory | `8GB` | `<startup-memory>` |
| Dynamic memory allowed | Yes or No | `<yes-no>` |
| Secure Boot state | On or Off | `<secure-boot-state>` |
| GPU-P required | Yes or No | `<yes-no>` |
| DDA required | Yes or No | `<yes-no>` |
| DDA location path | `PCIROOT(...)#PCI(...)` | `<location-path>` |
| MMIO low space | `3GB` | `<low-mmio>` |
| MMIO high space | `33280MB` | `<high-mmio>` |
| Guest controlled cache types | Enabled or disabled | `<enabled-disabled>` |
| Automatic stop action | `TurnOff` for DDA | `<automatic-stop-action>` |
| Mobility required | Yes or No | `<yes-no>` |
| Clustered VM required | Yes or No | `<yes-no>` |
| RDP hardware graphics policy required | Yes or No | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-GPU` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | VM is found |
| 3 | Confirm VM generation and state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name,Generation,State` | VM generation and state are known |
| 4 | Capture host GPU inventory | Hyper-V host | `Get-PnpDevice -PresentOnly \| Where-Object Class -eq 'Display'` | Display devices are listed |
| 5 | Capture partitionable GPU inventory | Hyper-V host | `Get-VMHostPartitionableGpu` | GPU-P capable devices are listed if supported |
| 6 | Capture assignable PCIe devices | Hyper-V host | `Get-VMHostAssignableDevice` | DDA assignable devices are listed if present |
| 7 | Capture current VM GPU-P adapter state | Hyper-V host | `Get-VMGpuPartitionAdapter -VMName '<vm-name>'` | Existing GPU-P assignment is known |
| 8 | Capture current DDA assigned device state | Hyper-V host | `Get-VMAssignableDevice -VMName '<vm-name>'` | Existing DDA assignment is known |
| 9 | Stop VM before GPU assignment change | Hyper-V host | `Stop-VM -Name '<vm-name>'` | VM is off |
| 10 | Configure GPU-P adapter if GPU-P is selected | Hyper-V host | `Add-VMGpuPartitionAdapter -VMName '<vm-name>'` | VM receives GPU partition adapter |
| 11 | Tune GPU-P resources if required | Hyper-V host | `Set-VMGpuPartitionAdapter -VMName '<vm-name>' ...` | GPU partition resource values are configured |
| 12 | Configure DDA VM prerequisites if DDA is selected | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticStopAction TurnOff -GuestControlledCacheTypes $true -LowMemoryMappedIoSpace 3GB -HighMemoryMappedIoSpace 33280MB` | VM is prepared for device passthrough |
| 13 | Locate DDA GPU location path | Hyper-V host | `Get-PnpDeviceProperty DEVPKEY_Device_LocationPaths` | PCI location path is captured |
| 14 | Disable device on host for DDA | Hyper-V host | `Disable-PnpDevice -InstanceId '<instance-id>' -Confirm:$false` | Device is disabled in host partition |
| 15 | Dismount device from host for DDA | Hyper-V host | `Dismount-VMHostAssignableDevice -Force -LocationPath '<location-path>'` | Device is assignable to VM |
| 16 | Assign DDA GPU to VM | Hyper-V host | `Add-VMAssignableDevice -LocationPath '<location-path>' -VMName '<vm-name>'` | GPU is attached to VM |
| 17 | Start VM | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts |
| 18 | Install or validate guest GPU driver | Guest VM | Vendor installer or Device Manager | Guest sees GPU and driver is healthy |
| 19 | Validate hardware acceleration inside guest | Guest VM | `Get-PnpDevice -Class Display`; application test | Guest sees expected accelerated GPU path |
| 20 | Configure RDS graphics policy if required | Guest VM | Local or domain GPO | RDP workload can use hardware adapter |
| 21 | Capture post-change evidence | Hyper-V host and guest | Run post-change verification skeleton | GPU-P or DDA state is documented |

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_PreChange_Capture_Skeleton

~~~powershell
# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_PreChange_Capture_Skeleton
# Purpose:
# Capture host GPU, VM, GPU-P, DDA, driver, firmware, PCIe, and event state before changes.

$EvidenceRoot = "C:\Admin\HyperV-GPU"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "GPU-VM01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing host identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Host-ComputerInfo.txt")

Write-Host "Capturing Hyper-V feature and services..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperV-Features.txt")

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-HyperV-Services.txt")

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

$VM |
    Select-Object Name, Id, State, Status, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path, AutomaticStopAction |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VM-Summary-Before.txt")

$VM |
    ConvertTo-Json -Depth 10 |
    Out-File -FilePath (Join-Path $OutputPath "04-VM-Summary-Before.json") -Encoding UTF8

Write-Host "Capturing VM firmware and security..." -ForegroundColor Cyan

if ($VM.Generation -eq 2) {
    Get-VMFirmware -VMName $VMName |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "05-VMFirmware-Before.txt")
}

Get-VMSecurity -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "06-VMSecurity-Before.txt")

Write-Host "Capturing host display devices..." -ForegroundColor Cyan

Get-PnpDevice -PresentOnly |
    Where-Object { $_.Class -eq "Display" } |
    Select-Object Status, Class, FriendlyName, InstanceId, Manufacturer |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-Host-DisplayDevices.txt")

Write-Host "Capturing display device location paths..." -ForegroundColor Cyan

Get-PnpDevice -PresentOnly |
    Where-Object { $_.Class -eq "Display" } |
    ForEach-Object {
        $Device = $_

        Get-PnpDeviceProperty -InstanceId $Device.InstanceId -KeyName DEVPKEY_Device_LocationPaths -ErrorAction SilentlyContinue |
            ForEach-Object {
                [PSCustomObject]@{
                    FriendlyName = $Device.FriendlyName
                    Manufacturer = $Device.Manufacturer
                    InstanceId = $Device.InstanceId
                    LocationPath = ($_.Data -join ";")
                }
            }
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "08-DisplayDevice-LocationPaths.txt")

Write-Host "Capturing GPU-P partitionable GPU state..." -ForegroundColor Cyan

Get-VMHostPartitionableGpu -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "09-VMHostPartitionableGpu-Before.txt")

Write-Host "Capturing VM GPU partition adapter state..." -ForegroundColor Cyan

Get-VMGpuPartitionAdapter -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "10-VMGpuPartitionAdapter-Before.txt")

Write-Host "Capturing host assignable devices..." -ForegroundColor Cyan

Get-VMHostAssignableDevice -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "11-VMHostAssignableDevices-Before.txt")

Write-Host "Capturing VM assignable devices..." -ForegroundColor Cyan

Get-VMAssignableDevice -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "12-VMAssignableDevices-Before.txt")

Write-Host "Capturing PCI and system device events..." -ForegroundColor Cyan

Get-WinEvent -LogName "System" -MaxEvents 150 -ErrorAction SilentlyContinue |
    Where-Object {
        $_.ProviderName -match "display|nvlddmkm|amdkmdag|igfx|pci|kernel-pnp|device"
    } |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "13-System-GPU-PCI-Events-Before.txt") -Encoding UTF8

Write-Host "Capturing Hyper-V events..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', "-"

    Get-WinEvent -LogName $Log -MaxEvents 75 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "14-$SafeLog-Before.txt") -Encoding UTF8
}

Write-Host "Pre-change GPU acceleration evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_GPU_Capability_Discovery_Skeleton

~~~powershell
# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_GPU_Capability_Discovery_Skeleton
# Purpose:
# Discover GPU hardware, partitionability, assignable PCIe devices, location paths, drivers, and VM readiness.

$VMName = "GPU-VM01"

Write-Host "Checking VM generation and state..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

$VM |
    Select-Object Name, State, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, AutomaticStopAction |
    Format-List

Write-Host "Checking host display devices..." -ForegroundColor Cyan

$DisplayDevices = Get-PnpDevice -PresentOnly |
    Where-Object { $_.Class -eq "Display" }

$DisplayDevices |
    Select-Object Status, Class, FriendlyName, InstanceId, Manufacturer |
    Format-Table -AutoSize

Write-Host "Checking GPU location paths for DDA..." -ForegroundColor Cyan

foreach ($Device in $DisplayDevices) {
    Write-Host "Device: $($Device.FriendlyName)" -ForegroundColor Yellow

    Get-PnpDeviceProperty `
        -InstanceId $Device.InstanceId `
        -KeyName DEVPKEY_Device_LocationPaths `
        -ErrorAction SilentlyContinue |
        Select-Object InstanceId, KeyName, Data |
        Format-List
}

Write-Host "Checking GPU-P partitionable GPUs..." -ForegroundColor Cyan

Get-VMHostPartitionableGpu -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Checking existing GPU-P adapter on VM..." -ForegroundColor Cyan

Get-VMGpuPartitionAdapter -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Checking assignable devices for DDA..." -ForegroundColor Cyan

Get-VMHostAssignableDevice -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Checking currently assigned DDA devices on VM..." -ForegroundColor Cyan

Get-VMAssignableDevice -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Checking GPU-related driver events..." -ForegroundColor Cyan

Get-WinEvent -LogName "System" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Where-Object {
        $_.ProviderName -match "display|nvlddmkm|amdkmdag|igfx|pci|kernel-pnp|device"
    } |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Write-Host "GPU capability discovery completed." -ForegroundColor Green
~~~

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_GPU_Partitioning_GPU-P_Skeleton

~~~powershell
# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_GPU_Partitioning_GPU-P_Skeleton
# Purpose:
# Assign a GPU partition adapter to a VM where GPU-P is supported by the host OS, GPU, and driver.

$VMName = "GPU-VM01"

# Resource values are examples.
# Adjust to match GPU, host support, guest requirements, and vendor guidance.
$MinPartitionVRAM = 80000000
$MaxPartitionVRAM = 1000000000
$OptimalPartitionVRAM = 1000000000

$MinPartitionEncode = 80000000
$MaxPartitionEncode = 1000000000
$OptimalPartitionEncode = 1000000000

$MinPartitionDecode = 80000000
$MaxPartitionDecode = 1000000000
$OptimalPartitionDecode = 1000000000

$MinPartitionCompute = 80000000
$MaxPartitionCompute = 1000000000
$OptimalPartitionCompute = 1000000000

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if ($VM.State -ne "Off") {
    Write-Host "Stopping VM before GPU-P adapter changes..." -ForegroundColor Cyan
    Stop-VM -Name $VMName
}

Write-Host "Checking host partitionable GPUs..." -ForegroundColor Cyan

$PartitionableGpus = Get-VMHostPartitionableGpu -ErrorAction SilentlyContinue

if (-not $PartitionableGpus) {
    throw "No partitionable GPUs were returned by Get-VMHostPartitionableGpu. Verify GPU, driver, OS, and host support."
}

$PartitionableGpus |
    Format-List *

Write-Host "Checking existing VM GPU partition adapter..." -ForegroundColor Cyan

$ExistingAdapter = Get-VMGpuPartitionAdapter -VMName $VMName -ErrorAction SilentlyContinue

if (-not $ExistingAdapter) {
    Write-Host "Adding GPU partition adapter to VM..." -ForegroundColor Cyan

    Add-VMGpuPartitionAdapter -VMName $VMName
}
else {
    Write-Host "VM already has a GPU partition adapter." -ForegroundColor Yellow
}

Write-Host "Configuring GPU partition adapter resource values..." -ForegroundColor Cyan

Set-VMGpuPartitionAdapter `
    -VMName $VMName `
    -MinPartitionVRAM $MinPartitionVRAM `
    -MaxPartitionVRAM $MaxPartitionVRAM `
    -OptimalPartitionVRAM $OptimalPartitionVRAM `
    -MinPartitionEncode $MinPartitionEncode `
    -MaxPartitionEncode $MaxPartitionEncode `
    -OptimalPartitionEncode $OptimalPartitionEncode `
    -MinPartitionDecode $MinPartitionDecode `
    -MaxPartitionDecode $MaxPartitionDecode `
    -OptimalPartitionDecode $OptimalPartitionDecode `
    -MinPartitionCompute $MinPartitionCompute `
    -MaxPartitionCompute $MaxPartitionCompute `
    -OptimalPartitionCompute $OptimalPartitionCompute

Write-Host "GPU partition adapter after change:" -ForegroundColor Green

Get-VMGpuPartitionAdapter -VMName $VMName |
    Format-List *

Write-Host "Starting VM for guest validation..." -ForegroundColor Cyan

Start-VM -Name $VMName
~~~

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_DDA_GPU_Passthrough_Skeleton

~~~powershell
# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_DDA_GPU_Passthrough_Skeleton
# Purpose:
# Configure Discrete Device Assignment for passing an entire GPU into one VM.
# Use only with compatible hardware, supported drivers, and approved maintenance window.

$VMName = "GPU-VM01"
$GpuVendorMatch = "NVIDIA"

$LowMMIO = 3GB
$HighMMIO = 33280MB

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if ($VM.State -ne "Off") {
    Write-Host "Stopping VM before DDA configuration..." -ForegroundColor Cyan
    Stop-VM -Name $VMName
}

Write-Host "Configuring VM prerequisites for DDA..." -ForegroundColor Cyan

Set-VM `
    -Name $VMName `
    -AutomaticStopAction TurnOff `
    -GuestControlledCacheTypes $true `
    -LowMemoryMappedIoSpace $LowMMIO `
    -HighMemoryMappedIoSpace $HighMMIO

Write-Host "Finding display devices matching vendor: $GpuVendorMatch" -ForegroundColor Cyan

$GpuDevices = Get-PnpDevice -PresentOnly |
    Where-Object {
        $_.Class -eq "Display" -and
        $_.Manufacturer -like "*$GpuVendorMatch*"
    }

if (-not $GpuDevices) {
    throw "No matching display devices found for vendor match: $GpuVendorMatch"
}

$GpuDevices |
    Select-Object Status, Class, FriendlyName, InstanceId, Manufacturer |
    Format-Table -AutoSize

$SelectedGpu = $GpuDevices | Select-Object -First 1

Write-Host "Selected GPU:" -ForegroundColor Yellow
$SelectedGpu | Format-List *

Write-Host "Getting PCI location path..." -ForegroundColor Cyan

$LocationPathProperty = Get-PnpDeviceProperty `
    -InstanceId $SelectedGpu.InstanceId `
    -KeyName DEVPKEY_Device_LocationPaths `
    -ErrorAction Stop

$LocationPath = $LocationPathProperty.Data[0]

Write-Host "LocationPath: $LocationPath" -ForegroundColor Green

Write-Warning "The next steps disable the GPU on the host and assign it to the VM."

Write-Host "Disabling GPU in host partition..." -ForegroundColor Cyan

Disable-PnpDevice `
    -InstanceId $SelectedGpu.InstanceId `
    -Confirm:$false

Write-Host "Dismounting GPU from host partition..." -ForegroundColor Cyan

Dismount-VMHostAssignableDevice `
    -Force `
    -LocationPath $LocationPath

Write-Host "Assigning GPU to VM..." -ForegroundColor Cyan

Add-VMAssignableDevice `
    -LocationPath $LocationPath `
    -VMName $VMName

Write-Host "DDA assignment after change:" -ForegroundColor Green

Get-VMAssignableDevice -VMName $VMName |
    Format-List *

Write-Host "Starting VM. Install vendor GPU driver inside the guest after boot." -ForegroundColor Cyan

Start-VM -Name $VMName
~~~

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Guest_Driver_And_Acceleration_Skeleton

~~~powershell
# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Guest_Driver_And_Acceleration_Skeleton
# Purpose:
# Validate GPU visibility and hardware acceleration inside a Windows guest.
# Run through PowerShell Direct where available or directly inside the guest.

$VMName = "GPU-VM01"
$UsePowerShellDirect = $true

if ($UsePowerShellDirect) {
    Write-Host "Running guest GPU checks through PowerShell Direct..." -ForegroundColor Cyan

    Invoke-Command -VMName $VMName -Credential (Get-Credential -Message "Enter guest admin credentials") -ScriptBlock {
        hostname
        whoami

        Write-Host "Guest OS:" -ForegroundColor Cyan
        Get-ComputerInfo |
            Select-Object CsName, WindowsProductName, WindowsVersion, OsBuildNumber |
            Format-List

        Write-Host "Display devices:" -ForegroundColor Cyan
        Get-PnpDevice -Class Display |
            Select-Object Status, Class, FriendlyName, InstanceId, Manufacturer |
            Format-Table -AutoSize

        Write-Host "Display driver signed drivers:" -ForegroundColor Cyan
        Get-CimInstance Win32_PnPSignedDriver |
            Where-Object { $_.DeviceClass -eq "DISPLAY" } |
            Select-Object DeviceName, Manufacturer, DriverVersion, DriverDate, InfName |
            Format-Table -AutoSize

        Write-Host "Video controller inventory:" -ForegroundColor Cyan
        Get-CimInstance Win32_VideoController |
            Select-Object Name, DriverVersion, AdapterRAM, VideoProcessor, Status |
            Format-List

        Write-Host "RDS graphics policy state if present:" -ForegroundColor Cyan
        $PolicyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services"
        if (Test-Path $PolicyPath) {
            Get-ItemProperty $PolicyPath |
                Format-List
        }
    }
}
else {
    Write-Host "Run the following inside the guest:" -ForegroundColor Yellow

    @'
hostname
Get-PnpDevice -Class Display
Get-CimInstance Win32_VideoController | Select Name,DriverVersion,AdapterRAM,VideoProcessor,Status
Get-CimInstance Win32_PnPSignedDriver | Where-Object DeviceClass -eq "DISPLAY" | Select DeviceName,Manufacturer,DriverVersion,DriverDate
'@
}
~~~

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Remote_Graphics_And_RDS_Policy_Skeleton

~~~powershell
# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Remote_Graphics_And_RDS_Policy_Skeleton
# Purpose:
# Configure guest policy so Remote Desktop Services sessions can use hardware graphics adapters where required.
# Run inside the Windows guest or through PowerShell Direct.

$VMName = "GPU-VM01"
$UsePowerShellDirect = $true

$PolicySettings = {
    $TsPolicyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services"

    New-Item -Path $TsPolicyPath -Force | Out-Null

    # Use hardware graphics adapters for all Remote Desktop Services sessions.
    New-ItemProperty `
        -Path $TsPolicyPath `
        -Name "bEnumerateHWBeforeSW" `
        -PropertyType DWord `
        -Value 1 `
        -Force | Out-Null

    # Prioritize H.264/AVC 444 graphics mode if desired for RDS graphics workloads.
    New-ItemProperty `
        -Path $TsPolicyPath `
        -Name "AVC444ModePreferred" `
        -PropertyType DWord `
        -Value 1 `
        -Force | Out-Null

    # Configure hardware graphics adapter use where policy is supported by OS edition and RDS role.
    New-ItemProperty `
        -Path $TsPolicyPath `
        -Name "UseHardwareDefaultGraphicsAdapter" `
        -PropertyType DWord `
        -Value 1 `
        -Force | Out-Null

    Get-ItemProperty $TsPolicyPath |
        Format-List

    gpupdate /force
}

if ($UsePowerShellDirect) {
    Invoke-Command `
        -VMName $VMName `
        -Credential (Get-Credential -Message "Enter guest admin credentials") `
        -ScriptBlock $PolicySettings
}
else {
    Invoke-Command -ScriptBlock $PolicySettings
}

Write-Host "RDS graphics policy configured. Reboot the guest if required." -ForegroundColor Green
~~~

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_PostChange_Verification_Skeleton

~~~powershell
# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_PostChange_Verification_Skeleton
# Purpose:
# Capture final GPU-P, DDA, VM, guest driver, acceleration, and event evidence after changes.

$EvidenceRoot = "C:\Admin\HyperV-GPU"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "GPU-VM01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM summary after GPU configuration..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, Id, State, Status, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path, AutomaticStopAction |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-After.txt")

Write-Host "Capturing GPU-P host state..." -ForegroundColor Cyan

Get-VMHostPartitionableGpu -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMHostPartitionableGpu-After.txt")

Write-Host "Capturing VM GPU partition adapter..." -ForegroundColor Cyan

Get-VMGpuPartitionAdapter -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMGpuPartitionAdapter-After.txt")

Write-Host "Capturing host assignable devices..." -ForegroundColor Cyan

Get-VMHostAssignableDevice -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMHostAssignableDevices-After.txt")

Write-Host "Capturing VM assignable devices..." -ForegroundColor Cyan

Get-VMAssignableDevice -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMAssignableDevices-After.txt")

Write-Host "Capturing host display device state..." -ForegroundColor Cyan

Get-PnpDevice -PresentOnly |
    Where-Object { $_.Class -eq "Display" } |
    Select-Object Status, Class, FriendlyName, InstanceId, Manufacturer |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-Host-DisplayDevices-After.txt")

Write-Host "Capturing VM firmware and security..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName

if ($VM.Generation -eq 2) {
    Get-VMFirmware -VMName $VMName |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "07-VMFirmware-After.txt")
}

Get-VMSecurity -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "08-VMSecurity-After.txt")

Write-Host "Capturing guest GPU state through PowerShell Direct if available..." -ForegroundColor Cyan

try {
    Invoke-Command -VMName $VMName -Credential (Get-Credential -Message "Enter guest admin credentials for GPU verification") -ScriptBlock {
        hostname

        "Display devices"
        Get-PnpDevice -Class Display |
            Select-Object Status, Class, FriendlyName, InstanceId, Manufacturer |
            Format-Table -AutoSize

        "Video controllers"
        Get-CimInstance Win32_VideoController |
            Select-Object Name, DriverVersion, AdapterRAM, VideoProcessor, Status |
            Format-List

        "Display signed drivers"
        Get-CimInstance Win32_PnPSignedDriver |
            Where-Object { $_.DeviceClass -eq "DISPLAY" } |
            Select-Object DeviceName, Manufacturer, DriverVersion, DriverDate, InfName |
            Format-Table -AutoSize
    } |
    Out-File -FilePath (Join-Path $OutputPath "09-Guest-GPU-State-After.txt") -Encoding UTF8
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "09-Guest-GPU-State-After-Error.txt") -Encoding UTF8
}

Write-Host "Capturing Hyper-V and GPU-related events..." -ForegroundColor Cyan

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
        Out-File -FilePath (Join-Path $OutputPath "10-$SafeLog-After.txt") -Encoding UTF8
}

Write-Host "Post-change GPU acceleration evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>' \| Select Name,State,Generation` | Hyper-V host | Confirms target VM and generation |
| `Get-PnpDevice -PresentOnly \| Where-Object Class -eq 'Display'` | Hyper-V host | Lists host display devices |
| `Get-PnpDeviceProperty -InstanceId '<instance-id>' -KeyName DEVPKEY_Device_LocationPaths` | Hyper-V host | Finds PCI location path for DDA |
| `Get-VMHostPartitionableGpu` | Hyper-V host | Shows GPU-P capable GPUs where supported |
| `Get-VMGpuPartitionAdapter -VMName '<vm-name>'` | Hyper-V host | Shows GPU-P adapter assigned to VM |
| `Add-VMGpuPartitionAdapter -VMName '<vm-name>'` | Hyper-V host | Adds GPU-P adapter to VM |
| `Set-VMGpuPartitionAdapter -VMName '<vm-name>'` | Hyper-V host | Configures GPU-P resource values |
| `Remove-VMGpuPartitionAdapter -VMName '<vm-name>'` | Hyper-V host | Removes GPU-P adapter |
| `Get-VMHostAssignableDevice` | Hyper-V host | Shows host devices assignable by DDA |
| `Get-VMAssignableDevice -VMName '<vm-name>'` | Hyper-V host | Shows DDA devices assigned to VM |
| `Set-VM -Name '<vm-name>' -AutomaticStopAction TurnOff` | Hyper-V host | Configures required DDA stop behavior |
| `Set-VM -Name '<vm-name>' -GuestControlledCacheTypes $true` | Hyper-V host | Enables guest controlled cache types for DDA graphics scenarios |
| `Set-VM -Name '<vm-name>' -LowMemoryMappedIoSpace 3GB -HighMemoryMappedIoSpace 33280MB` | Hyper-V host | Configures MMIO space for DDA GPU |
| `Disable-PnpDevice -InstanceId '<instance-id>' -Confirm:$false` | Hyper-V host | Disables GPU in host partition before DDA |
| `Dismount-VMHostAssignableDevice -Force -LocationPath '<location-path>'` | Hyper-V host | Dismounts PCIe device from host |
| `Add-VMAssignableDevice -LocationPath '<location-path>' -VMName '<vm-name>'` | Hyper-V host | Assigns DDA device to VM |
| `Remove-VMAssignableDevice -LocationPath '<location-path>' -VMName '<vm-name>'` | Hyper-V host | Removes DDA device from VM |
| `Mount-VMHostAssignableDevice -LocationPath '<location-path>'` | Hyper-V host | Returns DDA device to host |
| `Get-PnpDevice -Class Display` | Windows guest | Confirms guest sees display device |
| `Get-CimInstance Win32_VideoController` | Windows guest | Shows guest video controller and driver |
| `Get-CimInstance Win32_PnPSignedDriver \| Where-Object DeviceClass -eq 'DISPLAY'` | Windows guest | Confirms display driver version |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks VMMS assignment and startup events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM worker runtime errors |
| `Get-WinEvent -LogName 'System' -MaxEvents 50` | Hyper-V host | Checks GPU, PCI, driver, and PnP events |

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture post-issue state before rollback | Hyper-V host | Run post-change verification skeleton | Broken state is documented |
| 2 | Stop VM before GPU rollback | Hyper-V host | `Stop-VM -Name '<vm-name>' -TurnOff` | VM is off |
| 3 | Remove GPU-P adapter if GPU-P rollback is required | Hyper-V host | `Remove-VMGpuPartitionAdapter -VMName '<vm-name>'` | GPU-P adapter is removed |
| 4 | Remove DDA GPU from VM if DDA rollback is required | Hyper-V host | `Remove-VMAssignableDevice -LocationPath '<location-path>' -VMName '<vm-name>'` | GPU is detached from VM |
| 5 | Remount DDA GPU to host | Hyper-V host | `Mount-VMHostAssignableDevice -LocationPath '<location-path>'` | GPU returns to host partition |
| 6 | Re-enable host GPU device | Hyper-V host | `Enable-PnpDevice -InstanceId '<instance-id>' -Confirm:$false` | Host can use GPU again |
| 7 | Restore VM automatic stop action | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticStopAction '<old-value>'` | Stop behavior returns to baseline |
| 8 | Restore guest controlled cache setting | Hyper-V host | `Set-VM -Name '<vm-name>' -GuestControlledCacheTypes '<old-value>'` | Cache setting returns to baseline |
| 9 | Restore MMIO values from baseline | Hyper-V host | `Set-VM -Name '<vm-name>' -LowMemoryMappedIoSpace '<old-value>' -HighMemoryMappedIoSpace '<old-value>'` | MMIO settings return to baseline |
| 10 | Remove guest GPU driver if required | Guest VM | Vendor uninstall workflow | Guest driver state is cleaned up |
| 11 | Remove RDS graphics policy if it was test-only | Guest VM | Delete or revert registry/GPO settings | RDS graphics policy returns to baseline |
| 12 | Start VM after rollback | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts without assigned GPU |
| 13 | Capture rollback evidence | Hyper-V host and guest | Run post-change verification skeleton | Rollback state is documented |

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Get-VMHostPartitionableGpu` returns nothing | GPU, driver, OS, or firmware does not support GPU-P | Verify supported hardware, driver, OS build, and firmware |
| `Add-VMGpuPartitionAdapter` fails | GPU-P unsupported or VM state invalid | Shut down VM and validate host GPU-P support |
| GPU-P adapter exists but guest sees no GPU | Missing guest driver or unsupported guest path | Install supported guest driver and validate Device Manager |
| GPU-P resource setting fails | Values unsupported by GPU or driver | Use supported defaults or vendor guidance |
| DDA location path cannot be found | Wrong device selected or PnP property unavailable | Confirm Display class device and retrieve location path again |
| DDA dismount fails | Device still enabled, in use, or mitigation driver issue | Disable device, close GPU workloads, use approved `-Force` if no mitigation driver |
| DDA assignment fails | VM not configured for DDA, MMIO too small, or device not dismounted | Configure DDA VM settings and dismount device first |
| VM fails to start after DDA | MMIO too small or incompatible GPU | Increase MMIO values and validate vendor support |
| Guest shows Code 12 or resource error | Not enough MMIO resources | Increase `LowMemoryMappedIoSpace` or `HighMemoryMappedIoSpace` |
| Guest shows GPU but driver errors | Wrong guest driver or unsupported GPU virtualization mode | Install supported driver and confirm vendor support |
| Host loses GPU display | DDA removed GPU from host partition | Use alternate management path, remove DDA assignment, remount GPU |
| VM cannot live migrate | DDA hardware assignment blocks mobility | Remove DDA or use compatible GPU-P or cluster GPU design |
| Clustered VM cannot start on another node | GPU hardware not identical or not available on target node | Validate GPU availability and supported clustered GPU configuration |
| RDP session does not use GPU | Guest RDS policy not configured or app does not use GPU | Enable RDS hardware graphics policy and reboot guest |
| App does not detect GPU | App allowlist, driver, licensing, or virtualization detection issue | Validate app support and licensing |
| Secure Boot blocks GPU driver | Driver signing or Secure Boot template issue | Use supported signed driver and correct template |
| Checkpoint or backup fails | GPU assignment mode restricts save/checkpoint behavior | Use supported backup workflow and avoid unsupported checkpoint modes |
| Host driver crashes after rollback | Driver did not recover after remount | Reboot host or reinstall vendor driver during maintenance |
| Device cannot be re-enabled on host | Host still has device dismounted or assigned | Remove VM assignment, mount device, then enable PnP device |
| Evidence missing after failure | Fix applied before capture | Reproduce in lab if possible and capture pre-change evidence next time |

# 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this GPU acceleration task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host hardware, firmware, driver, and virtualization readiness |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before Hyper-V GPU assignment cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Host path and VM policy settings affect hardware assignment workflows |
| 05_Create_Generation_1_And_Generation_2_VMs.md | GPU-P, Secure Boot, and modern acceleration scenarios usually require Generation 2 VM planning |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | GPU workloads need correct vCPU, memory, and NUMA sizing |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Accelerated workloads still depend on correct network configuration |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Guest OS and integration services affect validation and management |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | DDA and GPU assignment require controlled VM shutdown and startup workflows |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md | PowerShell Direct helps validate Windows guest driver and device state |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Backup and recovery must be validated before hardware assignment changes |
| 14_Configure_Live_Migration_And_Storage_Migration.md | GPU assignment affects mobility and migration design |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md | Cluster compatibility must be validated for GPU-enabled VMs |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md | Monitoring confirms GPU assignment events and performance state |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Startup and device assignment troubleshooting uses GPU, PCI, VMMS, and Worker event evidence |
| 19_Configure_Shielded_VMs_vTPM_Secure_Boot_And_Guardian_Settings.md | Secure Boot and protected VM settings can affect driver and device assignment behavior |
| 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance.md | Hardware acceleration tradeoffs overlap with SR-IOV, mobility, and host compatibility |