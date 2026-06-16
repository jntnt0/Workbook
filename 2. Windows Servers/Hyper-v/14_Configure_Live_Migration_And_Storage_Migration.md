# 14_Configure_Live_Migration_And_Storage_Migration

# 14_Configure_Live_Migration_And_Storage_Migration_Index
14_Configure_Live_Migration_And_Storage_Migration.md
14_Configure_Live_Migration_And_Storage_Migration
14_Configure_Live_Migration_And_Storage_Migration_Source_Basis
14_Configure_Live_Migration_And_Storage_Migration_Mental_Model
14_Configure_Live_Migration_And_Storage_Migration_Planning_Table
14_Configure_Live_Migration_And_Storage_Migration_Configuration_Checklist
14_Configure_Live_Migration_And_Storage_Migration_PreChange_Capture_Skeleton
14_Configure_Live_Migration_And_Storage_Migration_Enable_Live_Migration_Skeleton
14_Configure_Live_Migration_And_Storage_Migration_Migration_Networks_Skeleton
14_Configure_Live_Migration_And_Storage_Migration_Storage_Migration_Skeleton
14_Configure_Live_Migration_And_Storage_Migration_Shared_Nothing_Live_Migration_Skeleton
14_Configure_Live_Migration_And_Storage_Migration_PostChange_Verification_Skeleton
14_Configure_Live_Migration_And_Storage_Migration_Verification_Commands
14_Configure_Live_Migration_And_Storage_Migration_Rollback
14_Configure_Live_Migration_And_Storage_Migration_Failure_Checks
14_Configure_Live_Migration_And_Storage_Migration_Related_Labs

# 14_Configure_Live_Migration_And_Storage_Migration_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V live migration overview | Supports moving running VMs between compatible Hyper-V hosts |
| Microsoft Learn | Hyper-V live migration authentication | Supports CredSSP, Kerberos, and constrained delegation planning |
| Microsoft Learn | Hyper-V live migration performance options | Supports TCP/IP, compression, and SMB migration modes |
| Microsoft Learn | Hyper-V storage migration | Supports moving VM configuration, checkpoints, smart paging, and VHDX files between storage locations |
| Microsoft Learn | Hyper-V PowerShell `Set-VMHost`, `Add-VMMigrationNetwork`, `Move-VM`, `Move-VMStorage` | Supports PowerShell-driven migration configuration and execution |
| Microsoft Learn | Hyper-V event logs | Supports troubleshooting VMMS, worker, network, and storage migration failures |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 14_Configure_Live_Migration_And_Storage_Migration_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Live migration | Moves a running VM from one Hyper-V host to another with minimal interruption |
| Storage migration | Moves VM files from one storage location to another while the VM remains registered |
| Shared-nothing live migration | Moves VM compute and storage between hosts without shared storage |
| Source host | Hyper-V host where the VM currently runs |
| Destination host | Hyper-V host where the VM will move |
| Migration network | IP network allowed to carry live migration traffic |
| CredSSP | Live migration authentication method that works easily from an interactive session on the source host |
| Kerberos | Live migration authentication method better for remote management, requiring delegation configuration |
| Constrained delegation | AD configuration that allows one Hyper-V host to delegate credentials to another for migration |
| Performance option | Migration transport mode such as TCP/IP, compression, or SMB |
| VM compatibility | Processor, Hyper-V version, virtual switch names, storage paths, and security settings must be compatible |
| Storage path mapping | Destination folder layout for VM configuration, checkpoints, smart paging, and VHDX files |
| Bandwidth impact | Live migration can consume significant host and network bandwidth |
| Rollback boundary | This workbook configures and tests migration, not full failover clustering or Hyper-V Replica |

# 14_Configure_Live_Migration_And_Storage_Migration_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Source Hyper-V host | `HV01` | `<source-host>` |
| Destination Hyper-V host | `HV02` | `<destination-host>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| Source VM path | `D:\Hyper-V\Virtual Machines` | `<source-vm-path>` |
| Source VHDX path | `D:\Hyper-V\Virtual Hard Disks` | `<source-vhdx-path>` |
| Destination VM path | `D:\Hyper-V\Virtual Machines` | `<destination-vm-path>` |
| Destination VHDX path | `D:\Hyper-V\Virtual Hard Disks` | `<destination-vhdx-path>` |
| Destination checkpoint path | `D:\Hyper-V\Checkpoints` | `<destination-checkpoint-path>` |
| Migration network | `192.168.50.0/24` | `<migration-network>` |
| Source migration IP | `192.168.50.11` | `<source-migration-ip>` |
| Destination migration IP | `192.168.50.12` | `<destination-migration-ip>` |
| Authentication type | CredSSP, Kerberos | `<authentication-type>` |
| Performance option | TCPIP, Compression, SMB | `<performance-option>` |
| Maximum live migrations | `2` | `<max-live-migrations>` |
| Maximum storage migrations | `2` | `<max-storage-migrations>` |
| Virtual switch name on both hosts | `vSwitch-External` | `<switch-name>` |
| CPU compatibility required | Yes or No | `<yes-no>` |
| Shared storage present | Yes or No | `<yes-no>` |
| Shared-nothing migration required | Yes or No | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-Migration` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 14_Configure_Live_Migration_And_Storage_Migration_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role on source host | Source host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm Hyper-V role on destination host | Destination host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 3 | Confirm VM exists on source host | Source host | `Get-VM -Name '<vm-name>'` | Source VM is found |
| 4 | Capture source VM configuration | Source host | `Get-VM -Name '<vm-name>'; Get-VMHardDiskDrive -VMName '<vm-name>'; Get-VMNetworkAdapter -VMName '<vm-name>'` | VM state, disks, and network are documented |
| 5 | Capture source and destination host settings | Both hosts | `Get-VMHost \| Select *Migration*` | Current migration settings are documented |
| 6 | Confirm destination storage paths exist | Destination host | `Test-Path 'D:\Hyper-V\Virtual Machines'; Test-Path 'D:\Hyper-V\Virtual Hard Disks'` | Destination folders exist |
| 7 | Confirm virtual switch names match | Both hosts | `Get-VMSwitch` | Destination has matching switch name |
| 8 | Confirm migration network reachability | Both hosts | `Test-NetConnection <peer-migration-ip>` | Hosts can reach each other on migration network |
| 9 | Enable live migration on source host | Source host | `Set-VMHost -VirtualMachineMigrationEnabled $true` | Source host allows live migration |
| 10 | Enable live migration on destination host | Destination host | `Set-VMHost -VirtualMachineMigrationEnabled $true` | Destination host allows live migration |
| 11 | Configure migration authentication | Both hosts | `Set-VMHost -VirtualMachineMigrationAuthenticationType Kerberos` | Authentication method is configured |
| 12 | Configure migration performance option | Both hosts | `Set-VMHost -VirtualMachineMigrationPerformanceOption Compression` | Performance option is configured |
| 13 | Configure migration limits | Both hosts | `Set-VMHost -MaximumVirtualMachineMigrations 2 -MaximumStorageMigrations 2` | Migration concurrency is controlled |
| 14 | Add migration network | Both hosts | `Add-VMMigrationNetwork '192.168.50.0/24'` | Live migration is restricted to intended network |
| 15 | Configure CPU compatibility if required | Source host | `Set-VMProcessor -VMName '<vm-name>' -CompatibilityForMigrationEnabled $true` | VM can migrate across compatible CPU generations |
| 16 | Run storage migration only if moving storage on same host | Source host | `Move-VMStorage -VMName '<vm-name>' -DestinationStoragePath '<destination-path>'` | VM files move to new storage path |
| 17 | Run live migration without storage if shared storage exists | Source host | `Move-VM -Name '<vm-name>' -DestinationHost '<destination-host>'` | Running VM moves to destination host |
| 18 | Run shared-nothing live migration if storage must move too | Source host | `Move-VM -Name '<vm-name>' -DestinationHost '<destination-host>' -IncludeStorage -DestinationStoragePath '<destination-path>'` | VM compute and storage move to destination host |
| 19 | Confirm VM appears on destination host | Destination host | `Get-VM -Name '<vm-name>'` | VM is registered and running or migrated |
| 20 | Confirm VM no longer runs on source host if moved | Source host | `Get-VM -Name '<vm-name>' -ErrorAction SilentlyContinue` | VM is absent or no longer running on source depending on migration type |
| 21 | Capture post-migration evidence | Both hosts | Run post-change verification skeleton | Migration state and logs are documented |

# 14_Configure_Live_Migration_And_Storage_Migration_PreChange_Capture_Skeleton

~~~powershell
# 14_Configure_Live_Migration_And_Storage_Migration_PreChange_Capture_Skeleton
# Purpose:
# Capture source host, destination host, VM, network, storage, and migration settings before migration changes.

$EvidenceRoot = "C:\Admin\HyperV-Migration"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$DestinationHost = "HV02"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing local host identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-LocalHost-ComputerInfo.txt")

Write-Host "Capturing local VMHost migration settings..." -ForegroundColor Cyan

Get-VMHost |
    Select-Object `
        ComputerName,
        VirtualMachineMigrationEnabled,
        VirtualMachineMigrationAuthenticationType,
        VirtualMachineMigrationPerformanceOption,
        MaximumVirtualMachineMigrations,
        MaximumStorageMigrations,
        NumaSpanningEnabled,
        VirtualMachinePath,
        VirtualHardDiskPath |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "02-Local-VMHost-MigrationSettings.txt")

Write-Host "Capturing local migration networks..." -ForegroundColor Cyan

Get-VMMigrationNetwork |
    Format-Table Subnet, Priority -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-Local-MigrationNetworks.txt")

Write-Host "Capturing source VM state..." -ForegroundColor Cyan

Get-VM -Name $VMName -ErrorAction Stop |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SnapshotFileLocation |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-SourceVM-Summary.txt")

Write-Host "Capturing source VM CPU, memory, disk, network, and checkpoints..." -ForegroundColor Cyan

Get-VMProcessor -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "05-SourceVM-Processor.txt")

Get-VMMemory -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "06-SourceVM-Memory.txt")

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-SourceVM-HardDisks.txt")

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "08-SourceVM-NetworkAdapters.txt")

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "09-SourceVM-Checkpoints.txt")

Write-Host "Capturing local virtual switches..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "10-Local-VMSwitches.txt")

Write-Host "Capturing local network adapters and IP configuration..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "11-Local-NetAdapters.txt")

Get-NetIPConfiguration |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "12-Local-NetIPConfiguration.txt") -Encoding UTF8

Write-Host "Testing destination host reachability..." -ForegroundColor Cyan

Test-NetConnection $DestinationHost |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "13-Test-DestinationHost.txt")

Write-Host "Capturing local volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "14-Local-Volumes.txt")

Write-Host "Pre-change migration evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 14_Configure_Live_Migration_And_Storage_Migration_Enable_Live_Migration_Skeleton

~~~powershell
# 14_Configure_Live_Migration_And_Storage_Migration_Enable_Live_Migration_Skeleton
# Purpose:
# Enable and configure host-level live migration settings.
# Run on each Hyper-V host that will participate in live migration.

$AuthenticationType = "Kerberos"
$PerformanceOption = "Compression"
$MaximumLiveMigrations = 2
$MaximumStorageMigrations = 2

Write-Host "Current VMHost migration settings:" -ForegroundColor Cyan

Get-VMHost |
    Select-Object `
        ComputerName,
        VirtualMachineMigrationEnabled,
        VirtualMachineMigrationAuthenticationType,
        VirtualMachineMigrationPerformanceOption,
        MaximumVirtualMachineMigrations,
        MaximumStorageMigrations |
    Format-List

Write-Host "Enabling live migration..." -ForegroundColor Cyan

Set-VMHost `
    -VirtualMachineMigrationEnabled $true `
    -VirtualMachineMigrationAuthenticationType $AuthenticationType `
    -VirtualMachineMigrationPerformanceOption $PerformanceOption `
    -MaximumVirtualMachineMigrations $MaximumLiveMigrations `
    -MaximumStorageMigrations $MaximumStorageMigrations

Write-Host "VMHost migration settings after change:" -ForegroundColor Green

Get-VMHost |
    Select-Object `
        ComputerName,
        VirtualMachineMigrationEnabled,
        VirtualMachineMigrationAuthenticationType,
        VirtualMachineMigrationPerformanceOption,
        MaximumVirtualMachineMigrations,
        MaximumStorageMigrations |
    Format-List
~~~

# 14_Configure_Live_Migration_And_Storage_Migration_Migration_Networks_Skeleton

~~~powershell
# 14_Configure_Live_Migration_And_Storage_Migration_Migration_Networks_Skeleton
# Purpose:
# Restrict live migration to the intended migration network.
# Run on each participating Hyper-V host.

$MigrationSubnet = "192.168.50.0/24"
$PeerMigrationIP = "192.168.50.12"

Write-Host "Current migration networks:" -ForegroundColor Cyan

Get-VMMigrationNetwork |
    Format-Table Subnet, Priority -AutoSize

Write-Host "Adding migration subnet if missing: $MigrationSubnet" -ForegroundColor Cyan

$Existing = Get-VMMigrationNetwork |
    Where-Object { $_.Subnet -eq $MigrationSubnet }

if (-not $Existing) {
    Add-VMMigrationNetwork $MigrationSubnet
}
else {
    Write-Host "Migration subnet already exists: $MigrationSubnet" -ForegroundColor Yellow
}

Write-Host "Migration networks after change:" -ForegroundColor Green

Get-VMMigrationNetwork |
    Format-Table Subnet, Priority -AutoSize

Write-Host "Testing peer migration IP reachability..." -ForegroundColor Cyan

Test-NetConnection $PeerMigrationIP |
    Format-List

Write-Host "Testing SMB port if SMB migration performance is planned..." -ForegroundColor Cyan

Test-NetConnection $PeerMigrationIP -Port 445 |
    Format-List
~~~

# 14_Configure_Live_Migration_And_Storage_Migration_Storage_Migration_Skeleton

~~~powershell
# 14_Configure_Live_Migration_And_Storage_Migration_Storage_Migration_Skeleton
# Purpose:
# Move VM storage to a different path on the same Hyper-V host.

$VMName = "LAB-WIN-SRV01"
$DestinationStoragePath = "E:\Hyper-V-MovedStorage\$VMName"

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Creating destination storage path..." -ForegroundColor Cyan

New-Item -Path $DestinationStoragePath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM paths before storage migration..." -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Path, ConfigurationLocation, SnapshotFileLocation, SmartPagingFilePath |
    Format-List

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

Write-Host "Starting storage migration to: $DestinationStoragePath" -ForegroundColor Cyan

Move-VMStorage `
    -VMName $VMName `
    -DestinationStoragePath $DestinationStoragePath

Write-Host "VM paths after storage migration:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, State, Path, ConfigurationLocation, SnapshotFileLocation, SmartPagingFilePath |
    Format-List

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

Write-Host "Destination folder inventory:" -ForegroundColor Cyan

Get-ChildItem -Path $DestinationStoragePath -Recurse |
    Select-Object FullName, Length, LastWriteTime |
    Format-Table -AutoSize
~~~

# 14_Configure_Live_Migration_And_Storage_Migration_Shared_Nothing_Live_Migration_Skeleton

~~~powershell
# 14_Configure_Live_Migration_And_Storage_Migration_Shared_Nothing_Live_Migration_Skeleton
# Purpose:
# Move a running VM from one Hyper-V host to another and include VM storage.
# Run from the source host or from a management session with correct delegation.

$VMName = "LAB-WIN-SRV01"
$DestinationHost = "HV02"
$DestinationStoragePath = "D:\Hyper-V\Virtual Machines\$VMName"

$UseIncludeStorage = $true

Write-Host "Validating source VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Source VM state:" -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Generation, Path, ConfigurationLocation |
    Format-List

Write-Host "Validating destination host reachability..." -ForegroundColor Cyan

Test-NetConnection $DestinationHost |
    Format-List

Write-Host "Validating destination Hyper-V service and switch names through PowerShell remoting..." -ForegroundColor Cyan

Invoke-Command -ComputerName $DestinationHost -ScriptBlock {
    Get-Service vmms |
        Select-Object Name, Status, StartType

    Get-VMSwitch |
        Select-Object Name, SwitchType, NetAdapterInterfaceDescription
}

Write-Host "Capturing source VM network adapters before migration..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status |
    Format-Table -AutoSize

Write-Host "Starting migration..." -ForegroundColor Cyan

if ($UseIncludeStorage) {
    Move-VM `
        -Name $VMName `
        -DestinationHost $DestinationHost `
        -IncludeStorage `
        -DestinationStoragePath $DestinationStoragePath
}
else {
    Move-VM `
        -Name $VMName `
        -DestinationHost $DestinationHost
}

Write-Host "Checking VM on destination host..." -ForegroundColor Green

Invoke-Command -ComputerName $DestinationHost -ScriptBlock {
    param($VMName)

    Get-VM -Name $VMName |
        Select-Object Name, State, Status, Generation, Uptime, Path |
        Format-List

    Get-VMHardDiskDrive -VMName $VMName |
        Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
        Format-Table -AutoSize

    Get-VMNetworkAdapter -VMName $VMName |
        Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
        Format-List
} -ArgumentList $VMName
~~~

# 14_Configure_Live_Migration_And_Storage_Migration_PostChange_Verification_Skeleton

~~~powershell
# 14_Configure_Live_Migration_And_Storage_Migration_PostChange_Verification_Skeleton
# Purpose:
# Capture migration configuration, VM location, disks, networking, storage, and event logs after live or storage migration.

$EvidenceRoot = "C:\Admin\HyperV-Migration"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$DestinationHost = "HV02"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing local VMHost migration settings..." -ForegroundColor Cyan

Get-VMHost |
    Select-Object `
        ComputerName,
        VirtualMachineMigrationEnabled,
        VirtualMachineMigrationAuthenticationType,
        VirtualMachineMigrationPerformanceOption,
        MaximumVirtualMachineMigrations,
        MaximumStorageMigrations,
        VirtualMachinePath,
        VirtualHardDiskPath |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Local-VMHost-MigrationSettings-After.txt")

Write-Host "Capturing local migration networks..." -ForegroundColor Cyan

Get-VMMigrationNetwork |
    Format-Table Subnet, Priority -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-Local-MigrationNetworks-After.txt")

Write-Host "Capturing local VM state if present..." -ForegroundColor Cyan

Get-VM -Name $VMName -ErrorAction SilentlyContinue |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SnapshotFileLocation |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-Local-VM-State-After.txt")

Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Local-VM-Disks-After.txt")

Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-Local-VM-Network-After.txt")

Write-Host "Capturing destination VM state if destination is reachable..." -ForegroundColor Cyan

try {
    Invoke-Command -ComputerName $DestinationHost -ScriptBlock {
        param($VMName)

        "Destination VMHost migration settings"
        Get-VMHost |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption, MaximumVirtualMachineMigrations, MaximumStorageMigrations |
            Format-List

        "Destination migration networks"
        Get-VMMigrationNetwork |
            Format-Table Subnet, Priority -AutoSize

        "Destination VM state"
        Get-VM -Name $VMName -ErrorAction SilentlyContinue |
            Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SnapshotFileLocation |
            Format-List

        "Destination VM disks"
        Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
            Format-Table -AutoSize

        "Destination VM network"
        Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
            Format-List
    } -ArgumentList $VMName |
    Out-File -FilePath (Join-Path $OutputPath "06-DestinationHost-VM-State-After.txt") -Encoding UTF8
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "06-DestinationHost-VM-State-After-Error.txt") -Encoding UTF8
}

Write-Host "Capturing local volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-Local-Volumes-After.txt")

Write-Host "Capturing migration-related event logs..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-HyperV-VMMS-RecentEvents-After.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-HyperV-Worker-RecentEvents-After.txt") -Encoding UTF8

Write-Host "Post-change migration evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 14_Configure_Live_Migration_And_Storage_Migration_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VMHost \| Select *Migration*` | Both hosts | Confirms migration settings |
| `Get-VMMigrationNetwork` | Both hosts | Confirms allowed migration networks |
| `Set-VMHost -VirtualMachineMigrationEnabled $true` | Both hosts | Enables live migration |
| `Set-VMHost -VirtualMachineMigrationAuthenticationType Kerberos` | Both hosts | Sets Kerberos authentication |
| `Set-VMHost -VirtualMachineMigrationPerformanceOption Compression` | Both hosts | Sets migration performance option |
| `Add-VMMigrationNetwork '<subnet>'` | Both hosts | Adds allowed live migration subnet |
| `Get-VM -Name '<vm-name>'` | Source or destination host | Confirms VM registration and state |
| `Get-VMProcessor -VMName '<vm-name>' \| Select CompatibilityForMigrationEnabled` | Source host | Confirms CPU compatibility setting |
| `Set-VMProcessor -VMName '<vm-name>' -CompatibilityForMigrationEnabled $true` | Source host | Enables CPU compatibility mode |
| `Test-NetConnection <destination-host>` | Source host | Confirms destination host reachability |
| `Test-NetConnection <migration-ip> -Port 445` | Source or destination host | Confirms SMB reachability if SMB transport is used |
| `Get-VMSwitch` | Both hosts | Confirms matching virtual switch names |
| `Move-VM -Name '<vm-name>' -DestinationHost '<destination-host>'` | Source host | Performs live migration when storage is already accessible |
| `Move-VM -Name '<vm-name>' -DestinationHost '<destination-host>' -IncludeStorage -DestinationStoragePath '<path>'` | Source host | Performs shared-nothing live migration with storage |
| `Move-VMStorage -VMName '<vm-name>' -DestinationStoragePath '<path>'` | Current VM host | Moves VM storage to another path |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Source or destination host | Confirms VHDX paths after migration |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Destination host | Confirms network adapter switch mapping and IP reporting |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Both hosts | Checks VMMS migration events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Both hosts | Checks VM worker migration events |

# 14_Configure_Live_Migration_And_Storage_Migration_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm current VM location | Source and destination hosts | `Get-VM -Name '<vm-name>' -ErrorAction SilentlyContinue` | VM location is identified |
| 2 | Move VM back to original host if rollback requires it | Current VM host | `Move-VM -Name '<vm-name>' -DestinationHost '<original-host>' -IncludeStorage -DestinationStoragePath '<original-storage-path>'` | VM returns to original host and storage |
| 3 | Move storage back to original path if only storage migration changed | Current VM host | `Move-VMStorage -VMName '<vm-name>' -DestinationStoragePath '<original-path>'` | VM files return to original storage path |
| 4 | Restore CPU compatibility setting | Current VM host | `Set-VMProcessor -VMName '<vm-name>' -CompatibilityForMigrationEnabled '<old-value>'` | CPU compatibility returns to baseline |
| 5 | Restore migration authentication type | Both hosts | `Set-VMHost -VirtualMachineMigrationAuthenticationType '<old-auth-type>'` | Authentication type returns to baseline |
| 6 | Restore migration performance option | Both hosts | `Set-VMHost -VirtualMachineMigrationPerformanceOption '<old-performance-option>'` | Performance option returns to baseline |
| 7 | Restore migration concurrency | Both hosts | `Set-VMHost -MaximumVirtualMachineMigrations '<old-value>' -MaximumStorageMigrations '<old-value>'` | Migration limits return to baseline |
| 8 | Remove migration subnet if added by mistake | Both hosts | `Remove-VMMigrationNetwork '<subnet>'` | Migration network list returns to baseline |
| 9 | Disable live migration if lab cleanup requires it | Both hosts | `Set-VMHost -VirtualMachineMigrationEnabled $false` | Live migration is disabled |
| 10 | Confirm VM starts and has network after rollback | Final VM host | `Start-VM -Name '<vm-name>'; Get-VMNetworkAdapter -VMName '<vm-name>'` | VM is functional after rollback |
| 11 | Capture rollback evidence | Both hosts | `Get-VMHost \| Select *Migration*; Get-VM -Name '<vm-name>'` | Rollback state is documented |

# 14_Configure_Live_Migration_And_Storage_Migration_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Live migration option unavailable | Live migration disabled on one or both hosts | Enable live migration with `Set-VMHost -VirtualMachineMigrationEnabled $true` |
| Migration fails from remote console | CredSSP used from non-source host or delegation not configured | Use Kerberos with constrained delegation or run from source host with CredSSP |
| Kerberos migration fails | AD constrained delegation missing or wrong service types | Configure delegation for Microsoft Virtual System Migration Service and CIFS where storage is included |
| Destination host not found | DNS, firewall, or connectivity issue | Fix DNS, routing, firewall, and host reachability |
| Migration network not used | Migration subnet missing or priority wrong | Configure `Add-VMMigrationNetwork` on both hosts |
| Migration is slow | Compression disabled, SMB not optimized, network congested, or storage slow | Review performance option, migration network, SMB, and storage performance |
| SMB migration fails | SMB port blocked or permissions issue | Test port 445 and confirm administrative access |
| VM fails compatibility check | CPU, Hyper-V version, VM generation, or device mismatch | Enable CPU compatibility or migrate to compatible host |
| Destination switch missing | Virtual switch names do not match | Create matching switch name on destination host or reconnect adapter after migration |
| VM has no network after migration | Switch mapping mismatch or VLAN missing | Reconnect adapter and confirm destination switch VLAN design |
| Storage migration fails | Destination path missing or insufficient free space | Create folders and confirm free space |
| Shared-nothing migration fails | Storage copy, authentication, or destination path issue | Confirm Kerberos delegation, CIFS delegation, destination path, and permissions |
| VM remains on source after failed migration | Migration rolled back | Review event logs and retry after fixing cause |
| VM partially imported or registered | Interrupted migration or import conflict | Remove broken registration only after validating VM files |
| Migration fails with checkpoints | Checkpoint chain or AVHDX issue | Merge stale checkpoints or verify disk chain with `Get-VHD` |
| Migration fails with saved state | Saved state not compatible | Start and cleanly shut down VM, or discard saved state only if approved |
| VM cannot start after storage migration | VHDX path moved incorrectly or missing | Verify `Get-VMHardDiskDrive` paths and move storage back if needed |
| Event logs show VMMS migration errors | Host-level migration service or auth issue | Review VMMS Admin events on both hosts |
| Event logs show worker errors | VM runtime, storage, or device issue | Review Worker Admin logs and VM configuration |
| Rollback migration fails | Same underlying auth, network, or storage issue | Fix root cause first, then migrate back |

# 14_Configure_Live_Migration_And_Storage_Migration_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this migration task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host CPU, memory, storage, network, and domain readiness |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before migration cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Provides default VM, VHDX, checkpoint, and smart paging paths |
| 04_Create_And_Configure_Virtual_Switches.md | Matching switch names are required for clean VM migration |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates the VM objects moved during migration |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | CPU compatibility and memory sizing affect live migration |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | VHDX paths and disk health affect storage migration |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Network adapter and VLAN settings must survive migration |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Checkpoint chains affect migration and storage movement |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Export and import are alternate portability workflows |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Backups should exist before moving important VMs |
| 15_Configure_Hyper-V_Replica_And_Failover_Testing.md | Replication is a recovery workflow, not the same as migration |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses migration logs, storage paths, switches, and VM state during troubleshooting |