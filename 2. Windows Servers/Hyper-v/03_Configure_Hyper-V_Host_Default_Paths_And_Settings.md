# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Index
03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md
03_Configure_Hyper-V_Host_Default_Paths_And_Settings
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Source_Basis
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Mental_Model
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Planning_Table
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Configuration_Checklist
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_PreChange_Capture_Skeleton
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Create_Storage_Paths_Skeleton
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Set_VMHost_Defaults_Skeleton
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_PostChange_Verification_Skeleton
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Verification_Commands
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Rollback
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Failure_Checks
03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Related_Labs

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V PowerShell `Get-VMHost` and `Set-VMHost` | Supports configuring default VM and VHDX paths, MAC range, migration limits, NUMA spanning, and enhanced session mode |
| Microsoft Learn | Hyper-V storage planning | Supports separating VM configuration files and virtual hard disks onto intended storage volumes |
| Microsoft Learn | Hyper-V host settings | Supports host-level configuration before VM creation |
| Microsoft Learn | Hyper-V live migration settings | Supports documenting migration defaults without fully configuring migration yet |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned to the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VM host defaults | Hyper-V host settings used when new VMs or VHDX files are created without specifying a custom path |
| Virtual machine path | Default location for VM configuration files |
| Virtual hard disk path | Default location for VHDX files |
| Checkpoint path | Usually configured per VM, but the host storage layout should still reserve space for checkpoints |
| Smart paging path | Used when a VM needs temporary paging during startup under memory pressure |
| MAC address range | Host-level dynamic MAC pool used for VM network adapters |
| Enhanced session mode | Allows richer local console interaction with supported guest operating systems |
| NUMA spanning | Allows VMs to span physical NUMA nodes when needed |
| Resource metering interval | Controls how often Hyper-V saves resource metering data |
| Migration limits | Host-level maximums for simultaneous live and storage migrations |
| Pre-change capture | The saved state of `Get-VMHost`, folders, ACLs, volumes, and role state before host settings are changed |
| Rollback boundary | This workbook changes host settings and folders, not VM configuration or virtual switch design |

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM storage root | `D:\Hyper-V` | `<vm-storage-root>` |
| Default VM configuration path | `D:\Hyper-V\Virtual Machines` | `<vm-config-path>` |
| Default VHDX path | `D:\Hyper-V\Virtual Hard Disks` | `<vhdx-path>` |
| Checkpoint holding path | `D:\Hyper-V\Checkpoints` | `<checkpoint-path>` |
| Smart paging path | `D:\Hyper-V\Smart Paging` | `<smart-paging-path>` |
| ISO library path | `D:\Hyper-V\ISO` | `<iso-path>` |
| Export path | `D:\Hyper-V\Exports` | `<export-path>` |
| Enhanced session mode | Enabled | `<enabled-disabled>` |
| NUMA spanning | Enabled for general lab use | `<enabled-disabled>` |
| Dynamic MAC minimum | `00155D000000` | `<mac-minimum>` |
| Dynamic MAC maximum | `00155DFFFFFF` | `<mac-maximum>` |
| Maximum live migrations | `2` | `<max-live-migrations>` |
| Maximum storage migrations | `2` | `<max-storage-migrations>` |
| Resource metering save interval | `01:00:00` | `<resource-metering-save-interval>` |
| Change evidence path | `C:\Admin\HyperV-HostSettings` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm Hyper-V PowerShell module exists | Hyper-V host | `Get-Module -ListAvailable Hyper-V` | Hyper-V module is available |
| 3 | Capture current host settings | Hyper-V host | `Get-VMHost \| Format-List *` | Current defaults are documented |
| 4 | Confirm target storage volume exists | Hyper-V host | `Get-Volume -DriveLetter D` | Storage volume is healthy and has free space |
| 5 | Create root Hyper-V folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V' -ItemType Directory -Force` | Root folder exists |
| 6 | Create VM configuration folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V\Virtual Machines' -ItemType Directory -Force` | VM configuration folder exists |
| 7 | Create VHDX folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V\Virtual Hard Disks' -ItemType Directory -Force` | VHDX folder exists |
| 8 | Create checkpoint folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V\Checkpoints' -ItemType Directory -Force` | Checkpoint holding folder exists |
| 9 | Create smart paging folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V\Smart Paging' -ItemType Directory -Force` | Smart paging folder exists |
| 10 | Create ISO library folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V\ISO' -ItemType Directory -Force` | ISO folder exists |
| 11 | Create VM export folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V\Exports' -ItemType Directory -Force` | Export folder exists |
| 12 | Confirm folder ACL inheritance | Hyper-V host | `Get-Acl 'D:\Hyper-V'` | Administrators and SYSTEM have appropriate access |
| 13 | Set default VM and VHDX paths | Hyper-V host | `Set-VMHost -VirtualMachinePath 'D:\Hyper-V\Virtual Machines' -VirtualHardDiskPath 'D:\Hyper-V\Virtual Hard Disks'` | New VMs and VHDX files use intended default paths |
| 14 | Configure enhanced session mode | Hyper-V host | `Set-VMHost -EnableEnhancedSessionMode $true` | Enhanced session mode is enabled |
| 15 | Configure NUMA spanning | Hyper-V host | `Set-VMHost -NumaSpanningEnabled $true` | NUMA spanning is enabled |
| 16 | Configure dynamic MAC pool if required | Hyper-V host | `Set-VMHost -MacAddressMinimum '00155D000000' -MacAddressMaximum '00155DFFFFFF'` | Dynamic MAC range is set |
| 17 | Configure migration concurrency defaults | Hyper-V host | `Set-VMHost -MaximumVirtualMachineMigrations 2 -MaximumStorageMigrations 2` | Migration limits are documented and set |
| 18 | Configure resource metering save interval | Hyper-V host | `Set-VMHost -ResourceMeteringSaveInterval '01:00:00'` | Resource metering interval is set |
| 19 | Capture final host settings | Hyper-V host | `Get-VMHost \| Format-List *` | Post-change configuration is documented |
| 20 | Confirm folders and settings are usable | Hyper-V host | Run verification skeleton | Evidence shows host defaults are ready |

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_PreChange_Capture_Skeleton

~~~powershell
# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_PreChange_Capture_Skeleton
# Purpose:
# Capture Hyper-V host settings, feature state, storage state, and folder state before changing defaults.

$EvidenceRoot = "C:\Admin\HyperV-HostSettings"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing Hyper-V feature state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-HyperVFeatureState.txt")

Write-Host "Capturing Hyper-V host settings..." -ForegroundColor Cyan

Get-VMHost |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMHost-Before.txt")

Get-VMHost |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "02-VMHost-Before.json") -Encoding UTF8

Write-Host "Capturing volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-Volumes-Before.txt")

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Export-Csv -Path (Join-Path $OutputPath "03-Volumes-Before.csv") -NoTypeInformation

Write-Host "Capturing disk state..." -ForegroundColor Cyan

Get-Disk |
    Sort-Object Number |
    Select-Object Number, FriendlyName, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Disks-Before.txt")

Write-Host "Capturing current Hyper-V folder candidates..." -ForegroundColor Cyan

$PathCandidates = @(
    "D:\Hyper-V",
    "D:\Hyper-V\Virtual Machines",
    "D:\Hyper-V\Virtual Hard Disks",
    "D:\Hyper-V\Checkpoints",
    "D:\Hyper-V\Smart Paging",
    "D:\Hyper-V\ISO",
    "D:\Hyper-V\Exports"
)

$PathCandidates | ForEach-Object {
    [PSCustomObject]@{
        Path = $_
        Exists = Test-Path $_
    }
} |
Export-Csv -Path (Join-Path $OutputPath "05-PathCandidates-Before.csv") -NoTypeInformation

Write-Host "Pre-change evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Create_Storage_Paths_Skeleton

~~~powershell
# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Create_Storage_Paths_Skeleton
# Purpose:
# Create the standard Hyper-V host folder layout before setting VMHost defaults.

$StorageRoot = "D:\Hyper-V"

$HyperVPaths = [ordered]@{
    Root            = $StorageRoot
    VirtualMachines = Join-Path $StorageRoot "Virtual Machines"
    VirtualHardDisks = Join-Path $StorageRoot "Virtual Hard Disks"
    Checkpoints     = Join-Path $StorageRoot "Checkpoints"
    SmartPaging     = Join-Path $StorageRoot "Smart Paging"
    ISO             = Join-Path $StorageRoot "ISO"
    Exports         = Join-Path $StorageRoot "Exports"
    Logs            = Join-Path $StorageRoot "Logs"
}

Write-Host "Creating Hyper-V folder layout..." -ForegroundColor Cyan

foreach ($Path in $HyperVPaths.GetEnumerator()) {
    New-Item -Path $Path.Value -ItemType Directory -Force | Out-Null

    [PSCustomObject]@{
        Name = $Path.Key
        Path = $Path.Value
        Exists = Test-Path $Path.Value
    }
}

Write-Host "Reviewing folder ACLs..." -ForegroundColor Cyan

Get-Acl $StorageRoot |
    Format-List |
    Out-File -FilePath (Join-Path $StorageRoot "Logs\HyperVRoot-Acl.txt") -Encoding UTF8

Write-Host "Reviewing volume free space..." -ForegroundColor Cyan

Get-Volume -DriveLetter D |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-List |
    Out-File -FilePath (Join-Path $StorageRoot "Logs\Volume-D-BeforeVMHostDefaults.txt") -Encoding UTF8

Write-Host "Hyper-V folder layout is ready under: $StorageRoot" -ForegroundColor Green
~~~

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Set_VMHost_Defaults_Skeleton

~~~powershell
# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Set_VMHost_Defaults_Skeleton
# Purpose:
# Configure Hyper-V host defaults after the role is installed and storage folders are created.

$VMConfigPath = "D:\Hyper-V\Virtual Machines"
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks"

$MacMinimum = "00155D000000"
$MacMaximum = "00155DFFFFFF"

$MaximumLiveMigrations = 2
$MaximumStorageMigrations = 2
$ResourceMeteringSaveInterval = New-TimeSpan -Hours 1

Write-Host "Confirming required folders exist..." -ForegroundColor Cyan

$RequiredPaths = @(
    $VMConfigPath,
    $VHDXPath
)

foreach ($Path in $RequiredPaths) {
    if (-not (Test-Path $Path)) {
        throw "Required path does not exist: $Path"
    }
}

Write-Host "Setting default VM and VHDX paths..." -ForegroundColor Cyan

Set-VMHost `
    -VirtualMachinePath $VMConfigPath `
    -VirtualHardDiskPath $VHDXPath

Write-Host "Setting enhanced session mode..." -ForegroundColor Cyan

Set-VMHost -EnableEnhancedSessionMode $true

Write-Host "Setting NUMA spanning..." -ForegroundColor Cyan

Set-VMHost -NumaSpanningEnabled $true

Write-Host "Setting dynamic MAC address range..." -ForegroundColor Cyan

Set-VMHost `
    -MacAddressMinimum $MacMinimum `
    -MacAddressMaximum $MacMaximum

Write-Host "Setting migration concurrency defaults..." -ForegroundColor Cyan

Set-VMHost `
    -MaximumVirtualMachineMigrations $MaximumLiveMigrations `
    -MaximumStorageMigrations $MaximumStorageMigrations

Write-Host "Setting resource metering save interval..." -ForegroundColor Cyan

Set-VMHost -ResourceMeteringSaveInterval $ResourceMeteringSaveInterval

Write-Host "Final VMHost settings:" -ForegroundColor Green

Get-VMHost |
    Select-Object `
        VirtualMachinePath,
        VirtualHardDiskPath,
        EnableEnhancedSessionMode,
        NumaSpanningEnabled,
        MacAddressMinimum,
        MacAddressMaximum,
        MaximumVirtualMachineMigrations,
        MaximumStorageMigrations,
        ResourceMeteringSaveInterval |
    Format-List
~~~

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_PostChange_Verification_Skeleton

~~~powershell
# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_PostChange_Verification_Skeleton
# Purpose:
# Verify Hyper-V host defaults, storage folders, ACLs, and event logs after configuration.

$EvidenceRoot = "C:\Admin\HyperV-HostSettings"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing final VMHost settings..." -ForegroundColor Cyan

Get-VMHost |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VMHost-After.txt")

Get-VMHost |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "01-VMHost-After.json") -Encoding UTF8

Write-Host "Confirming configured paths exist..." -ForegroundColor Cyan

$VMHost = Get-VMHost

$ConfiguredPaths = @(
    $VMHost.VirtualMachinePath,
    $VMHost.VirtualHardDiskPath,
    "D:\Hyper-V\Checkpoints",
    "D:\Hyper-V\Smart Paging",
    "D:\Hyper-V\ISO",
    "D:\Hyper-V\Exports"
)

$ConfiguredPaths | ForEach-Object {
    [PSCustomObject]@{
        Path = $_
        Exists = Test-Path $_
    }
} |
Tee-Object -FilePath (Join-Path $OutputPath "02-ConfiguredPaths.txt") |
Export-Csv -Path (Join-Path $OutputPath "02-ConfiguredPaths.csv") -NoTypeInformation

Write-Host "Capturing ACLs for Hyper-V root..." -ForegroundColor Cyan

Get-Acl "D:\Hyper-V" |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "03-HyperVRoot-Acl.txt") -Encoding UTF8

Write-Host "Capturing volume state after configuration..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Volumes-After.txt")

Write-Host "Capturing recent Hyper-V VMMS events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 30 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "05-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Write-Host "Checking for obvious path mismatch..." -ForegroundColor Cyan

$ExpectedVMPath = "D:\Hyper-V\Virtual Machines"
$ExpectedVHDXPath = "D:\Hyper-V\Virtual Hard Disks"

$Validation = [PSCustomObject]@{
    ComputerName = $env:COMPUTERNAME
    VirtualMachinePath = $VMHost.VirtualMachinePath
    VirtualMachinePathMatches = $VMHost.VirtualMachinePath -eq $ExpectedVMPath
    VirtualHardDiskPath = $VMHost.VirtualHardDiskPath
    VirtualHardDiskPathMatches = $VMHost.VirtualHardDiskPath -eq $ExpectedVHDXPath
    EnhancedSessionMode = $VMHost.EnableEnhancedSessionMode
    NumaSpanningEnabled = $VMHost.NumaSpanningEnabled
    MacAddressMinimum = $VMHost.MacAddressMinimum
    MacAddressMaximum = $VMHost.MacAddressMaximum
    MaximumVirtualMachineMigrations = $VMHost.MaximumVirtualMachineMigrations
    MaximumStorageMigrations = $VMHost.MaximumStorageMigrations
}

$Validation |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "06-ValidationSummary.txt")

$Validation |
    ConvertTo-Json -Depth 4 |
    Out-File -FilePath (Join-Path $OutputPath "06-ValidationSummary.json") -Encoding UTF8

Write-Host "Post-change evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VMHost \| Format-List *` | Hyper-V host | Shows all current host-level Hyper-V settings |
| `Get-VMHost \| Select VirtualMachinePath, VirtualHardDiskPath` | Hyper-V host | Confirms default VM and VHDX paths |
| `Get-VMHost \| Select EnableEnhancedSessionMode, NumaSpanningEnabled` | Hyper-V host | Confirms enhanced session and NUMA settings |
| `Get-VMHost \| Select MacAddressMinimum, MacAddressMaximum` | Hyper-V host | Confirms dynamic MAC address pool |
| `Get-VMHost \| Select MaximumVirtualMachineMigrations, MaximumStorageMigrations` | Hyper-V host | Confirms migration concurrency defaults |
| `Test-Path 'D:\Hyper-V\Virtual Machines'` | Hyper-V host | Confirms VM configuration folder exists |
| `Test-Path 'D:\Hyper-V\Virtual Hard Disks'` | Hyper-V host | Confirms VHDX folder exists |
| `Test-Path 'D:\Hyper-V\Checkpoints'` | Hyper-V host | Confirms checkpoint holding folder exists |
| `Get-Acl 'D:\Hyper-V'` | Hyper-V host | Confirms root folder ACL state |
| `Get-Volume -DriveLetter D` | Hyper-V host | Confirms storage volume health and free space |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Confirms no immediate Hyper-V VMMS path or configuration errors |
| `Get-Module -ListAvailable Hyper-V` | Hyper-V host | Confirms Hyper-V PowerShell support |

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Review pre-change VMHost settings | Hyper-V host | `Get-Content '<pre-change-path>\02-VMHost-Before.txt'` | Previous settings are identified |
| 2 | Restore default VM path | Hyper-V host | `Set-VMHost -VirtualMachinePath '<old-vm-config-path>'` | VM configuration default is restored |
| 3 | Restore default VHDX path | Hyper-V host | `Set-VMHost -VirtualHardDiskPath '<old-vhdx-path>'` | VHDX default is restored |
| 4 | Restore enhanced session mode setting | Hyper-V host | `Set-VMHost -EnableEnhancedSessionMode $false` | Enhanced session mode returns to prior value |
| 5 | Restore NUMA spanning setting | Hyper-V host | `Set-VMHost -NumaSpanningEnabled $true` | NUMA spanning returns to prior value |
| 6 | Restore MAC address range | Hyper-V host | `Set-VMHost -MacAddressMinimum '<old-min>' -MacAddressMaximum '<old-max>'` | MAC pool returns to prior range |
| 7 | Restore migration limits | Hyper-V host | `Set-VMHost -MaximumVirtualMachineMigrations <old-value> -MaximumStorageMigrations <old-value>` | Migration concurrency returns to prior values |
| 8 | Remove newly created empty folders only if unused | Hyper-V host | `Remove-Item 'D:\Hyper-V\<folder-name>' -Recurse -Force` | Empty test folders are removed |
| 9 | Do not delete folders containing VM data | Hyper-V host | `Get-ChildItem 'D:\Hyper-V' -Recurse` | VM data is protected from accidental deletion |
| 10 | Capture rollback evidence | Hyper-V host | `Get-VMHost \| Format-List *` | Rollback state is documented |

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Set-VMHost` is not recognized | Hyper-V PowerShell module missing | Install `Hyper-V-PowerShell` or run on the Hyper-V host |
| `Get-VMHost` fails | Hyper-V role not installed or management stack unavailable | Confirm Hyper-V role installation and reboot state |
| Default paths do not change | Command was not run elevated or wrong host was targeted | Open elevated PowerShell and confirm `hostname` before rerunning |
| Path does not exist error | Storage folder was not created first | Create the folder, confirm `Test-Path`, then rerun `Set-VMHost` |
| Access denied creating folders | Insufficient permissions or locked-down ACLs | Run as local administrator and review parent folder ACLs |
| VHDX files still appear in old path | Existing VMs keep their existing paths | New defaults apply to future VM and VHDX creation, not existing files |
| Checkpoints do not use the new folder | Checkpoint path is normally per VM | Configure checkpoint paths when creating or modifying individual VMs |
| Smart paging does not use expected folder | Smart paging path is per VM | Configure smart paging path during VM creation or VM settings |
| Dynamic MAC conflict occurs | MAC pool overlaps another Hyper-V host or existing infrastructure | Assign a unique MAC pool range per host |
| Live migration setting seems incomplete | Network and authentication settings are configured in later migration workbook | Configure detailed live migration settings in the migration task |
| Storage volume fills quickly | VM files, checkpoints, ISOs, or exports share the same volume | Move ISOs or exports, expand volume, or split storage paths |
| VMMS event log shows storage errors | Path, ACL, disk, or volume issue | Confirm folder exists, ACLs are correct, and volume is healthy |
| Folder layout was created on wrong drive | Incorrect planning value or mistyped path | Move or recreate folders on correct storage volume, then reset VMHost paths |

# 03_Configure_Hyper-V_Host_Default_Paths_And_Settings_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this host settings task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms storage, OS, and role readiness before host settings are changed |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before `Get-VMHost` and `Set-VMHost` can be used |
| 04_Create_And_Configure_Virtual_Switches.md | Follows host defaults by configuring the networking foundation |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Uses the default VM and VHDX paths configured here |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Depends on the default VHDX path configured here |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Builds on the checkpoint storage planning captured here |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Uses export and storage path conventions from this workbook |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Uses migration host defaults documented here |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses these defaults as the known-good host storage baseline |