# 05_Create_Generation_1_And_Generation_2_VMs

# 05_Create_Generation_1_And_Generation_2_VMs_Index
05_Create_Generation_1_And_Generation_2_VMs.md
05_Create_Generation_1_And_Generation_2_VMs
05_Create_Generation_1_And_Generation_2_VMs_Source_Basis
05_Create_Generation_1_And_Generation_2_VMs_Mental_Model
05_Create_Generation_1_And_Generation_2_VMs_Planning_Table
05_Create_Generation_1_And_Generation_2_VMs_Configuration_Checklist
05_Create_Generation_1_And_Generation_2_VMs_PreCreation_Capture_Skeleton
05_Create_Generation_1_And_Generation_2_VMs_Generation_2_Windows_VM_Skeleton
05_Create_Generation_1_And_Generation_2_VMs_Generation_2_Linux_VM_Skeleton
05_Create_Generation_1_And_Generation_2_VMs_Generation_1_Legacy_VM_Skeleton
05_Create_Generation_1_And_Generation_2_VMs_ISO_And_Boot_Order_Skeleton
05_Create_Generation_1_And_Generation_2_VMs_PostCreation_Verification_Skeleton
05_Create_Generation_1_And_Generation_2_VMs_Verification_Commands
05_Create_Generation_1_And_Generation_2_VMs_Rollback
05_Create_Generation_1_And_Generation_2_VMs_Failure_Checks
05_Create_Generation_1_And_Generation_2_VMs_Related_Labs

# 05_Create_Generation_1_And_Generation_2_VMs_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V virtual machine generations | Supports choosing Generation 1 or Generation 2 based on guest OS and boot requirements |
| Microsoft Learn | Hyper-V PowerShell `New-VM` | Supports creating VMs from PowerShell |
| Microsoft Learn | Hyper-V PowerShell `Set-VM`, `Set-VMProcessor`, `Set-VMMemory` | Supports VM CPU, memory, and automatic action configuration |
| Microsoft Learn | Hyper-V PowerShell `New-VHD`, `Add-VMDvdDrive`, `Set-VMFirmware`, `Set-VMBios` | Supports disk creation, ISO attachment, and boot order configuration |
| Microsoft Learn | Hyper-V secure boot | Supports Generation 2 Secure Boot behavior for Windows and Linux guests |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 05_Create_Generation_1_And_Generation_2_VMs_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Generation 1 VM | Legacy BIOS based VM type used for older operating systems, legacy PXE, and compatibility scenarios |
| Generation 2 VM | UEFI based VM type used for modern Windows and Linux guests |
| VM configuration path | Folder where Hyper-V stores VM configuration files |
| VHDX path | Folder where Hyper-V stores the virtual hard disk files |
| Startup memory | Memory assigned to the VM at boot |
| Dynamic memory | Allows Hyper-V to adjust VM memory within configured minimum and maximum limits |
| Virtual processor count | Number of virtual CPUs assigned to the VM |
| Virtual switch | Hyper-V network switch that connects the VM network adapter to external, internal, or private networks |
| ISO attachment | Mounts an installer ISO to the VM DVD drive |
| Boot order | Determines whether the VM boots from DVD, disk, PXE, or other devices |
| Secure Boot | UEFI protection available to Generation 2 VMs |
| Secure Boot template | Certificate template used by Generation 2 VMs for Windows or Linux Secure Boot |
| Checkpoint policy | Decision point for whether checkpoints are allowed during initial build |
| Automatic actions | VM behavior when the Hyper-V host starts or stops |
| Rollback boundary | This workbook creates VM shells and boot media but does not complete guest OS installation |

# 05_Create_Generation_1_And_Generation_2_VMs_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-DC01`, `LAB-WIN11`, `LAB-LINUX01` | `<vm-name>` |
| VM generation | Generation 1 or Generation 2 | `<generation>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| VM configuration path | `D:\Hyper-V\Virtual Machines` | `<vm-config-path>` |
| VHDX path | `D:\Hyper-V\Virtual Hard Disks` | `<vhdx-path>` |
| VHDX size | `60GB`, `100GB` | `<vhdx-size>` |
| VHDX type | Dynamic, fixed | `<vhdx-type>` |
| Startup memory | `2GB`, `4GB` | `<startup-memory>` |
| Dynamic memory | Enabled or disabled | `<enabled-disabled>` |
| Minimum memory | `1GB` | `<minimum-memory>` |
| Maximum memory | `4GB`, `8GB` | `<maximum-memory>` |
| Virtual processors | `2` | `<processor-count>` |
| Virtual switch | `vSwitch-External`, `vSwitch-Internal-Lab` | `<switch-name>` |
| ISO path | `D:\Hyper-V\ISO\WindowsServer.iso` | `<iso-path>` |
| Secure Boot | Enabled or disabled | `<enabled-disabled>` |
| Secure Boot template | `MicrosoftWindows`, `MicrosoftUEFICertificateAuthority` | `<secure-boot-template>` |
| Automatic start action | Nothing, StartIfRunning, Start | `<automatic-start-action>` |
| Automatic stop action | Save, ShutDown, TurnOff | `<automatic-stop-action>` |
| Checkpoint type | Production, Standard, Disabled | `<checkpoint-type>` |
| Evidence path | `C:\Admin\HyperV-VMCreation` | `<evidence-path>` |

# 05_Create_Generation_1_And_Generation_2_VMs_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm Hyper-V PowerShell module exists | Hyper-V host | `Get-Module -ListAvailable Hyper-V` | Hyper-V module is available |
| 3 | Confirm VMHost default paths | Hyper-V host | `Get-VMHost \| Select VirtualMachinePath, VirtualHardDiskPath` | Default VM and VHDX paths are correct |
| 4 | Confirm target folders exist | Hyper-V host | `Test-Path 'D:\Hyper-V\Virtual Machines'; Test-Path 'D:\Hyper-V\Virtual Hard Disks'` | Required folders exist |
| 5 | Confirm virtual switch exists | Hyper-V host | `Get-VMSwitch -Name '<switch-name>'` | Intended switch exists |
| 6 | Confirm ISO exists | Hyper-V host | `Test-Path '<iso-path>'` | Installer ISO is available |
| 7 | Confirm VM name is unused | Hyper-V host | `Get-VM -Name '<vm-name>' -ErrorAction SilentlyContinue` | No existing VM uses the same name |
| 8 | Create Generation 2 VM for modern Windows guest | Hyper-V host | `New-VM -Name '<vm-name>' -Generation 2 -MemoryStartupBytes 4GB -NewVHDPath '<vhdx-path>\<vm-name>.vhdx' -NewVHDSizeBytes 80GB -Path '<vm-config-path>' -SwitchName '<switch-name>'` | Generation 2 VM is created |
| 9 | Create Generation 2 VM for modern Linux guest if required | Hyper-V host | `New-VM -Name '<vm-name>' -Generation 2 -MemoryStartupBytes 2GB -NewVHDPath '<vhdx-path>\<vm-name>.vhdx' -NewVHDSizeBytes 60GB -Path '<vm-config-path>' -SwitchName '<switch-name>'` | Generation 2 Linux VM is created |
| 10 | Create Generation 1 VM for legacy guest if required | Hyper-V host | `New-VM -Name '<vm-name>' -Generation 1 -MemoryStartupBytes 2GB -NewVHDPath '<vhdx-path>\<vm-name>.vhdx' -NewVHDSizeBytes 60GB -Path '<vm-config-path>' -SwitchName '<switch-name>'` | Generation 1 VM is created |
| 11 | Configure virtual processors | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -Count 2` | VM has planned vCPU count |
| 12 | Configure dynamic memory if required | Hyper-V host | `Set-VMMemory -VMName '<vm-name>' -DynamicMemoryEnabled $true -MinimumBytes 1GB -StartupBytes 2GB -MaximumBytes 4GB` | VM memory policy is set |
| 13 | Attach installer ISO | Hyper-V host | `Add-VMDvdDrive -VMName '<vm-name>' -Path '<iso-path>'` | ISO is attached to VM |
| 14 | Configure Generation 2 boot order | Hyper-V host | `$dvd = Get-VMDvdDrive -VMName '<vm-name>'; Set-VMFirmware -VMName '<vm-name>' -FirstBootDevice $dvd` | VM boots from ISO first |
| 15 | Configure Generation 1 boot order | Hyper-V host | `Set-VMBios -VMName '<vm-name>' -StartupOrder CD,IDE,LegacyNetworkAdapter,Floppy` | Legacy VM boots from ISO first |
| 16 | Configure Secure Boot for Windows Generation 2 VM | Hyper-V host | `Set-VMFirmware -VMName '<vm-name>' -EnableSecureBoot On -SecureBootTemplate MicrosoftWindows` | Windows Secure Boot is enabled |
| 17 | Configure Secure Boot for Linux Generation 2 VM if supported | Hyper-V host | `Set-VMFirmware -VMName '<vm-name>' -EnableSecureBoot On -SecureBootTemplate MicrosoftUEFICertificateAuthority` | Linux Secure Boot template is set |
| 18 | Configure automatic start and stop actions | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticStartAction Nothing -AutomaticStopAction Save` | VM host start and stop behavior is controlled |
| 19 | Configure checkpoint type | Hyper-V host | `Set-VM -Name '<vm-name>' -CheckpointType Production` | Production checkpoints are configured |
| 20 | Verify VM configuration | Hyper-V host | `Get-VM -Name '<vm-name>'; Get-VMHardDiskDrive -VMName '<vm-name>'; Get-VMNetworkAdapter -VMName '<vm-name>'` | VM shell, disk, and network adapter are correct |
| 21 | Start VM for installer boot test | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts successfully |
| 22 | Open VM console | Hyper-V host | `vmconnect localhost '<vm-name>'` | VM console opens |
| 23 | Capture post-creation evidence | Hyper-V host | Run post-creation verification skeleton | Evidence confirms VM creation state |

# 05_Create_Generation_1_And_Generation_2_VMs_PreCreation_Capture_Skeleton

~~~powershell
# 05_Create_Generation_1_And_Generation_2_VMs_PreCreation_Capture_Skeleton
# Purpose:
# Capture Hyper-V host, switch, path, ISO, and current VM state before creating a new VM.

$EvidenceRoot = "C:\Admin\HyperV-VMCreation"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PreCreation-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing Hyper-V feature state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-HyperVFeatureState.txt")

Write-Host "Capturing VMHost defaults..." -ForegroundColor Cyan

Get-VMHost |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMHost.txt")

Write-Host "Capturing existing VM inventory..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMInventory-Before.txt")

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, Path |
    Export-Csv -Path (Join-Path $OutputPath "03-VMInventory-Before.csv") -NoTypeInformation

Write-Host "Capturing virtual switches..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMSwitches.txt")

Write-Host "Capturing storage path state..." -ForegroundColor Cyan

$VMHost = Get-VMHost
$PathChecks = @(
    $VMHost.VirtualMachinePath,
    $VMHost.VirtualHardDiskPath,
    "D:\Hyper-V\ISO"
)

$PathChecks | ForEach-Object {
    [PSCustomObject]@{
        Path = $_
        Exists = Test-Path $_
    }
} |
Tee-Object -FilePath (Join-Path $OutputPath "05-PathChecks.txt") |
Export-Csv -Path (Join-Path $OutputPath "05-PathChecks.csv") -NoTypeInformation

Write-Host "Capturing ISO library contents..." -ForegroundColor Cyan

Get-ChildItem "D:\Hyper-V\ISO" -Filter "*.iso" -ErrorAction SilentlyContinue |
    Select-Object Name, FullName, Length, LastWriteTime |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-ISO-Library.txt")

Write-Host "Capturing volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-Volumes.txt")

Write-Host "Pre-creation evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 05_Create_Generation_1_And_Generation_2_VMs_Generation_2_Windows_VM_Skeleton

~~~powershell
# 05_Create_Generation_1_And_Generation_2_VMs_Generation_2_Windows_VM_Skeleton
# Purpose:
# Create a modern Windows Generation 2 VM with UEFI, Secure Boot, VHDX, memory, processor, switch, and ISO.

$VMName = "LAB-WIN-SRV01"
$VMConfigPath = "D:\Hyper-V\Virtual Machines"
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\$VMName.vhdx"
$ISOPath = "D:\Hyper-V\ISO\WindowsServer.iso"
$SwitchName = "vSwitch-External"

$StartupMemory = 4GB
$MinimumMemory = 2GB
$MaximumMemory = 8GB
$VHDXSize = 80GB
$ProcessorCount = 2

Write-Host "Validating prerequisites..." -ForegroundColor Cyan

if (Get-VM -Name $VMName -ErrorAction SilentlyContinue) {
    throw "VM already exists: $VMName"
}

if (-not (Test-Path $VMConfigPath)) {
    throw "VM configuration path does not exist: $VMConfigPath"
}

if (-not (Test-Path (Split-Path $VHDXPath))) {
    throw "VHDX folder does not exist: $(Split-Path $VHDXPath)"
}

if (-not (Test-Path $ISOPath)) {
    throw "ISO path does not exist: $ISOPath"
}

if (-not (Get-VMSwitch -Name $SwitchName -ErrorAction SilentlyContinue)) {
    throw "Virtual switch does not exist: $SwitchName"
}

Write-Host "Creating Generation 2 Windows VM: $VMName" -ForegroundColor Cyan

New-VM `
    -Name $VMName `
    -Generation 2 `
    -MemoryStartupBytes $StartupMemory `
    -NewVHDPath $VHDXPath `
    -NewVHDSizeBytes $VHDXSize `
    -Path $VMConfigPath `
    -SwitchName $SwitchName

Write-Host "Configuring CPU and memory..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $VMName `
    -Count $ProcessorCount

Set-VMMemory `
    -VMName $VMName `
    -DynamicMemoryEnabled $true `
    -MinimumBytes $MinimumMemory `
    -StartupBytes $StartupMemory `
    -MaximumBytes $MaximumMemory

Write-Host "Configuring firmware and Secure Boot..." -ForegroundColor Cyan

Set-VMFirmware `
    -VMName $VMName `
    -EnableSecureBoot On `
    -SecureBootTemplate MicrosoftWindows

Write-Host "Attaching installer ISO..." -ForegroundColor Cyan

Add-VMDvdDrive `
    -VMName $VMName `
    -Path $ISOPath

$DVDDrive = Get-VMDvdDrive -VMName $VMName

Set-VMFirmware `
    -VMName $VMName `
    -FirstBootDevice $DVDDrive

Write-Host "Configuring automatic actions and checkpoint type..." -ForegroundColor Cyan

Set-VM `
    -Name $VMName `
    -AutomaticStartAction Nothing `
    -AutomaticStopAction Save `
    -CheckpointType Production

Write-Host "Generation 2 Windows VM created:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, DynamicMemoryEnabled, Path |
    Format-List

Get-VMHardDiskDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize

Get-VMDvdDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize

Get-VMNetworkAdapter -VMName $VMName |
    Format-Table VMName, Name, SwitchName, MacAddress, Status -AutoSize
~~~

# 05_Create_Generation_1_And_Generation_2_VMs_Generation_2_Linux_VM_Skeleton

~~~powershell
# 05_Create_Generation_1_And_Generation_2_VMs_Generation_2_Linux_VM_Skeleton
# Purpose:
# Create a modern Linux Generation 2 VM with UEFI, optional Secure Boot template, VHDX, memory, processor, switch, and ISO.

$VMName = "LAB-LINUX01"
$VMConfigPath = "D:\Hyper-V\Virtual Machines"
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\$VMName.vhdx"
$ISOPath = "D:\Hyper-V\ISO\UbuntuServer.iso"
$SwitchName = "vSwitch-Internal-Lab"

$StartupMemory = 2GB
$MinimumMemory = 1GB
$MaximumMemory = 4GB
$VHDXSize = 60GB
$ProcessorCount = 2

$EnableLinuxSecureBoot = $true

Write-Host "Validating prerequisites..." -ForegroundColor Cyan

if (Get-VM -Name $VMName -ErrorAction SilentlyContinue) {
    throw "VM already exists: $VMName"
}

if (-not (Test-Path $VMConfigPath)) {
    throw "VM configuration path does not exist: $VMConfigPath"
}

if (-not (Test-Path (Split-Path $VHDXPath))) {
    throw "VHDX folder does not exist: $(Split-Path $VHDXPath)"
}

if (-not (Test-Path $ISOPath)) {
    throw "ISO path does not exist: $ISOPath"
}

if (-not (Get-VMSwitch -Name $SwitchName -ErrorAction SilentlyContinue)) {
    throw "Virtual switch does not exist: $SwitchName"
}

Write-Host "Creating Generation 2 Linux VM: $VMName" -ForegroundColor Cyan

New-VM `
    -Name $VMName `
    -Generation 2 `
    -MemoryStartupBytes $StartupMemory `
    -NewVHDPath $VHDXPath `
    -NewVHDSizeBytes $VHDXSize `
    -Path $VMConfigPath `
    -SwitchName $SwitchName

Write-Host "Configuring CPU and memory..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $VMName `
    -Count $ProcessorCount

Set-VMMemory `
    -VMName $VMName `
    -DynamicMemoryEnabled $true `
    -MinimumBytes $MinimumMemory `
    -StartupBytes $StartupMemory `
    -MaximumBytes $MaximumMemory

Write-Host "Configuring Generation 2 firmware..." -ForegroundColor Cyan

if ($EnableLinuxSecureBoot) {
    Set-VMFirmware `
        -VMName $VMName `
        -EnableSecureBoot On `
        -SecureBootTemplate MicrosoftUEFICertificateAuthority
}
else {
    Set-VMFirmware `
        -VMName $VMName `
        -EnableSecureBoot Off
}

Write-Host "Attaching Linux installer ISO..." -ForegroundColor Cyan

Add-VMDvdDrive `
    -VMName $VMName `
    -Path $ISOPath

$DVDDrive = Get-VMDvdDrive -VMName $VMName

Set-VMFirmware `
    -VMName $VMName `
    -FirstBootDevice $DVDDrive

Write-Host "Configuring automatic actions and checkpoint type..." -ForegroundColor Cyan

Set-VM `
    -Name $VMName `
    -AutomaticStartAction Nothing `
    -AutomaticStopAction Save `
    -CheckpointType Production

Write-Host "Generation 2 Linux VM created:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, DynamicMemoryEnabled, Path |
    Format-List

Get-VMFirmware -VMName $VMName |
    Format-List

Get-VMHardDiskDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize

Get-VMDvdDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize

Get-VMNetworkAdapter -VMName $VMName |
    Format-Table VMName, Name, SwitchName, MacAddress, Status -AutoSize
~~~

# 05_Create_Generation_1_And_Generation_2_VMs_Generation_1_Legacy_VM_Skeleton

~~~powershell
# 05_Create_Generation_1_And_Generation_2_VMs_Generation_1_Legacy_VM_Skeleton
# Purpose:
# Create a Generation 1 VM for legacy BIOS based guests or older compatibility scenarios.

$VMName = "LAB-LEGACY01"
$VMConfigPath = "D:\Hyper-V\Virtual Machines"
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\$VMName.vhdx"
$ISOPath = "D:\Hyper-V\ISO\LegacyOS.iso"
$SwitchName = "vSwitch-Internal-Lab"

$StartupMemory = 2GB
$MinimumMemory = 1GB
$MaximumMemory = 4GB
$VHDXSize = 60GB
$ProcessorCount = 2

Write-Host "Validating prerequisites..." -ForegroundColor Cyan

if (Get-VM -Name $VMName -ErrorAction SilentlyContinue) {
    throw "VM already exists: $VMName"
}

if (-not (Test-Path $VMConfigPath)) {
    throw "VM configuration path does not exist: $VMConfigPath"
}

if (-not (Test-Path (Split-Path $VHDXPath))) {
    throw "VHDX folder does not exist: $(Split-Path $VHDXPath)"
}

if (-not (Test-Path $ISOPath)) {
    throw "ISO path does not exist: $ISOPath"
}

if (-not (Get-VMSwitch -Name $SwitchName -ErrorAction SilentlyContinue)) {
    throw "Virtual switch does not exist: $SwitchName"
}

Write-Host "Creating Generation 1 legacy VM: $VMName" -ForegroundColor Cyan

New-VM `
    -Name $VMName `
    -Generation 1 `
    -MemoryStartupBytes $StartupMemory `
    -NewVHDPath $VHDXPath `
    -NewVHDSizeBytes $VHDXSize `
    -Path $VMConfigPath `
    -SwitchName $SwitchName

Write-Host "Configuring CPU and memory..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $VMName `
    -Count $ProcessorCount

Set-VMMemory `
    -VMName $VMName `
    -DynamicMemoryEnabled $true `
    -MinimumBytes $MinimumMemory `
    -StartupBytes $StartupMemory `
    -MaximumBytes $MaximumMemory

Write-Host "Attaching installer ISO..." -ForegroundColor Cyan

Add-VMDvdDrive `
    -VMName $VMName `
    -Path $ISOPath

Write-Host "Configuring Generation 1 BIOS boot order..." -ForegroundColor Cyan

Set-VMBios `
    -VMName $VMName `
    -StartupOrder CD,IDE,LegacyNetworkAdapter,Floppy

Write-Host "Configuring automatic actions and checkpoint type..." -ForegroundColor Cyan

Set-VM `
    -Name $VMName `
    -AutomaticStartAction Nothing `
    -AutomaticStopAction Save `
    -CheckpointType Production

Write-Host "Generation 1 VM created:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, DynamicMemoryEnabled, Path |
    Format-List

Get-VMBios -VMName $VMName |
    Format-List

Get-VMHardDiskDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize

Get-VMDvdDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize

Get-VMNetworkAdapter -VMName $VMName |
    Format-Table VMName, Name, SwitchName, MacAddress, Status -AutoSize
~~~

# 05_Create_Generation_1_And_Generation_2_VMs_ISO_And_Boot_Order_Skeleton

~~~powershell
# 05_Create_Generation_1_And_Generation_2_VMs_ISO_And_Boot_Order_Skeleton
# Purpose:
# Attach or replace ISO media and set correct boot order for either Generation 1 or Generation 2 VMs.

$VMName = "LAB-WIN-SRV01"
$ISOPath = "D:\Hyper-V\ISO\WindowsServer.iso"

Write-Host "Validating VM and ISO..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if (-not (Test-Path $ISOPath)) {
    throw "ISO path does not exist: $ISOPath"
}

Write-Host "Checking existing DVD drive..." -ForegroundColor Cyan

$DVDDrive = Get-VMDvdDrive -VMName $VMName -ErrorAction SilentlyContinue

if ($DVDDrive) {
    Write-Host "Updating existing DVD drive ISO path..." -ForegroundColor Cyan

    Set-VMDvdDrive `
        -VMName $VMName `
        -ControllerNumber $DVDDrive.ControllerNumber `
        -ControllerLocation $DVDDrive.ControllerLocation `
        -Path $ISOPath
}
else {
    Write-Host "Adding DVD drive..." -ForegroundColor Cyan

    Add-VMDvdDrive `
        -VMName $VMName `
        -Path $ISOPath
}

$DVDDrive = Get-VMDvdDrive -VMName $VMName

if ($VM.Generation -eq 2) {
    Write-Host "Setting Generation 2 first boot device to DVD..." -ForegroundColor Cyan

    Set-VMFirmware `
        -VMName $VMName `
        -FirstBootDevice $DVDDrive

    Get-VMFirmware -VMName $VMName |
        Format-List
}
elseif ($VM.Generation -eq 1) {
    Write-Host "Setting Generation 1 startup order to CD first..." -ForegroundColor Cyan

    Set-VMBios `
        -VMName $VMName `
        -StartupOrder CD,IDE,LegacyNetworkAdapter,Floppy

    Get-VMBios -VMName $VMName |
        Format-List
}
else {
    throw "Unknown VM generation for $VMName"
}

Write-Host "DVD drive state:" -ForegroundColor Green

Get-VMDvdDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize
~~~

# 05_Create_Generation_1_And_Generation_2_VMs_PostCreation_Verification_Skeleton

~~~powershell
# 05_Create_Generation_1_And_Generation_2_VMs_PostCreation_Verification_Skeleton
# Purpose:
# Verify VM configuration, disks, DVD, network, firmware, memory, CPU, and basic start behavior after VM creation.

$EvidenceRoot = "C:\Admin\HyperV-VMCreation"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostCreation-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, DynamicMemoryEnabled, Path, CheckpointType, AutomaticStartAction, AutomaticStopAction |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary.txt")

Get-VM -Name $VMName |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "01-VM-Summary.json") -Encoding UTF8

Write-Host "Capturing processor configuration..." -ForegroundColor Cyan

Get-VMProcessor -VMName $VMName |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMProcessor.txt")

Write-Host "Capturing memory configuration..." -ForegroundColor Cyan

Get-VMMemory -VMName $VMName |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMMemory.txt")

Write-Host "Capturing disk configuration..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMHardDiskDrive.txt")

Get-VHD -VMId (Get-VM -Name $VMName).VMId |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VHD-Details.txt")

Write-Host "Capturing DVD configuration..." -ForegroundColor Cyan

Get-VMDvdDrive -VMName $VMName |
    Format-Table VMName, ControllerType, ControllerNumber, ControllerLocation, Path -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-VMDvdDrive.txt")

Write-Host "Capturing network adapter configuration..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, DynamicMacAddressEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-VMNetworkAdapter.txt")

Write-Host "Capturing firmware or BIOS configuration..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName

if ($VM.Generation -eq 2) {
    Get-VMFirmware -VMName $VMName |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "08-VMFirmware.txt")
}
else {
    Get-VMBios -VMName $VMName |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "08-VMBios.txt")
}

Write-Host "Checking recent VMMS events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Write-Host "Optional boot test. Starting VM..." -ForegroundColor Cyan

Start-VM -Name $VMName

Start-Sleep -Seconds 5

Get-VM -Name $VMName |
    Select-Object Name, State, Uptime, Status |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "10-VM-BootTest.txt")

Write-Host "Post-creation verification evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 05_Create_Generation_1_And_Generation_2_VMs_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM` | Hyper-V host | Lists VM inventory |
| `Get-VM -Name '<vm-name>' \| Format-List *` | Hyper-V host | Shows full VM configuration |
| `Get-VM -Name '<vm-name>' \| Select Name, State, Generation, ProcessorCount, MemoryStartup, Path` | Hyper-V host | Confirms VM generation, CPU, memory, and path |
| `Get-VMProcessor -VMName '<vm-name>'` | Hyper-V host | Confirms vCPU configuration |
| `Get-VMMemory -VMName '<vm-name>'` | Hyper-V host | Confirms static or dynamic memory configuration |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Confirms attached VHDX |
| `Get-VHD -Path '<vhdx-path>'` | Hyper-V host | Confirms VHDX size, type, and location |
| `Get-VMDvdDrive -VMName '<vm-name>'` | Hyper-V host | Confirms ISO attachment |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Confirms VM network adapter and switch assignment |
| `Get-VMFirmware -VMName '<vm-name>'` | Hyper-V host | Confirms Generation 2 firmware, boot order, and Secure Boot |
| `Get-VMBios -VMName '<vm-name>'` | Hyper-V host | Confirms Generation 1 BIOS boot order |
| `Start-VM -Name '<vm-name>'` | Hyper-V host | Confirms VM can start |
| `vmconnect localhost '<vm-name>'` | Hyper-V host | Opens VM console for installer boot verification |
| `Stop-VM -Name '<vm-name>' -TurnOff` | Hyper-V host | Stops lab VM after boot test if needed |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks recent VM management errors |

# 05_Create_Generation_1_And_Generation_2_VMs_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm VM state | Hyper-V host | `Get-VM -Name '<vm-name>'` | VM exists and state is known |
| 2 | Stop VM if running | Hyper-V host | `Stop-VM -Name '<vm-name>' -TurnOff` | VM is off |
| 3 | Export VM first if rollback needs preservation | Hyper-V host | `Export-VM -Name '<vm-name>' -Path 'D:\Hyper-V\Exports'` | VM is preserved before removal |
| 4 | Remove VM registration | Hyper-V host | `Remove-VM -Name '<vm-name>' -Force` | VM configuration is removed from Hyper-V |
| 5 | Confirm VM is gone | Hyper-V host | `Get-VM -Name '<vm-name>' -ErrorAction SilentlyContinue` | No VM object returns |
| 6 | Remove VHDX only if deletion is approved | Hyper-V host | `Remove-Item '<vhdx-path>' -Force` | VHDX file is deleted only after confirmation |
| 7 | Remove VM folder only if empty and approved | Hyper-V host | `Remove-Item '<vm-folder-path>' -Recurse -Force` | Empty VM folder is removed |
| 8 | Remove generated evidence if not needed | Hyper-V host | `Remove-Item 'C:\Admin\HyperV-VMCreation' -Recurse -Force` | Evidence folder is removed |
| 9 | Confirm switch was not altered | Hyper-V host | `Get-VMSwitch` | Network foundation remains intact |
| 10 | Capture rollback state | Hyper-V host | `Get-VM; Get-ChildItem 'D:\Hyper-V\Virtual Hard Disks'` | Rollback state is documented |

# 05_Create_Generation_1_And_Generation_2_VMs_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `New-VM` fails because VM already exists | Duplicate VM name | Choose a unique VM name or remove the old VM after validation |
| `New-VM` fails because switch not found | Wrong switch name or switch not created | Run `Get-VMSwitch` and use the exact switch name |
| `New-VM` fails because path does not exist | VM or VHDX folder missing | Create folders or fix `Set-VMHost` defaults |
| VHDX creation fails | Insufficient space or permissions issue | Confirm volume free space and folder ACLs |
| ISO attachment fails | ISO path is wrong or file missing | Copy ISO to library and validate with `Test-Path` |
| Generation 2 VM does not boot ISO | Boot order wrong or ISO not UEFI bootable | Set DVD as first boot device or use a UEFI capable ISO |
| Linux Generation 2 VM will not boot with Secure Boot | Wrong Secure Boot template or unsupported distro | Use `MicrosoftUEFICertificateAuthority` or disable Secure Boot |
| Generation 1 VM does not boot ISO | BIOS startup order wrong | Use `Set-VMBios -StartupOrder CD,IDE,LegacyNetworkAdapter,Floppy` |
| VM starts but shows no network | Wrong virtual switch or VLAN issue | Confirm `Get-VMNetworkAdapter`, switch assignment, and VLAN configuration |
| Dynamic memory behaves poorly | Minimum or maximum memory set too low | Increase minimum, startup, or maximum memory |
| Guest installer reports no disk | Disk not attached or unsupported controller scenario | Confirm `Get-VMHardDiskDrive` and VM generation compatibility |
| VM console does not open | VMConnect issue or permissions issue | Run as admin, confirm Hyper-V tools, use `vmconnect localhost '<vm-name>'` |
| Start-VM fails with insufficient memory | Host does not have enough available memory | Lower startup memory or stop other VMs |
| Secure Boot option unavailable | VM is Generation 1 | Use Generation 2 for UEFI Secure Boot |
| Legacy OS will not install on Generation 2 | Guest requires BIOS or legacy devices | Recreate as Generation 1 VM |
| VM was created in wrong path | Host defaults or explicit path were wrong | Stop VM, export and import to correct path, or recreate before guest install |
| Wrong VHDX size selected | Planning mistake | Resize before installing guest OS or recreate VM |
| VM created with wrong generation | VM generation cannot be changed after creation | Remove and recreate VM with correct generation |

# 05_Create_Generation_1_And_Generation_2_VMs_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this VM creation task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host CPU, memory, storage, and network readiness |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before VMs can be created |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Provides default VM and VHDX paths used by this workbook |
| 04_Create_And_Configure_Virtual_Switches.md | Provides the virtual switches used by new VMs |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | Builds on the basic CPU and memory settings configured here |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Expands storage management beyond initial VHDX creation |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Builds on the initial VM network adapter configuration |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Follows VM shell creation by installing the guest operating system |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Adds checkpoint strategy after the VM exists |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Uses the VMs created here for lifecycle operations |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses VM creation details for troubleshooting boot, storage, and network failures |