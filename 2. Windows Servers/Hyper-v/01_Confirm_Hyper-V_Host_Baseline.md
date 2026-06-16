# 01_Confirm_Hyper-V_Host_Baseline

# 01_Confirm_Hyper-V_Host_Baseline_Index
01_Confirm_Hyper-V_Host_Baseline.md
01_Confirm_Hyper-V_Host_Baseline
01_Confirm_Hyper-V_Host_Baseline_Source_Basis
01_Confirm_Hyper-V_Host_Baseline_Mental_Model
01_Confirm_Hyper-V_Host_Baseline_Planning_Table
01_Confirm_Hyper-V_Host_Baseline_Configuration_Checklist
01_Confirm_Hyper-V_Host_Baseline_Host_Inventory_Skeleton
01_Confirm_Hyper-V_Host_Baseline_Hardware_Virtualization_Check_Skeleton
01_Confirm_Hyper-V_Host_Baseline_Network_Storage_Baseline_Skeleton
01_Confirm_Hyper-V_Host_Baseline_Readiness_Report_Skeleton
01_Confirm_Hyper-V_Host_Baseline_Verification_Commands
01_Confirm_Hyper-V_Host_Baseline_Rollback
01_Confirm_Hyper-V_Host_Baseline_Failure_Checks
01_Confirm_Hyper-V_Host_Baseline_Related_Labs

# 01_Confirm_Hyper-V_Host_Baseline_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V on Windows Server overview | Confirms Hyper-V role purpose, host responsibilities, and management tooling |
| Microsoft Learn | Hyper-V system requirements | Confirms CPU virtualization, SLAT, DEP, firmware virtualization, and supported OS requirements |
| Microsoft Learn | Install Hyper-V role | Supports the need to validate the server before role installation |
| Microsoft Learn | Hyper-V networking | Supports NIC inventory, virtual switch planning, and physical adapter baseline |
| Microsoft Learn | Hyper-V storage | Supports VHDX path planning, volume readiness, and VM storage baseline |
| Internal Workbook Standard | Windows Servers workbook structure | Keeps this lab aligned to DHCP, DNS, GPO, AD, and File Services workbook format |

# 01_Confirm_Hyper-V_Host_Baseline_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Hyper-V host baseline | The known-good state of the physical or nested server before the Hyper-V role is installed |
| Hardware-assisted virtualization | CPU and firmware support required for running virtual machines |
| SLAT | Second Level Address Translation, required for modern Hyper-V virtualization performance and support |
| DEP | Data Execution Prevention, required for Hyper-V processor protection requirements |
| Firmware virtualization | BIOS or UEFI setting that must be enabled before Hyper-V can run |
| Host OS readiness | The Windows Server installation must be stable, patched, named correctly, and joined to the right workgroup or domain |
| Network baseline | Physical NICs, IP configuration, DNS, gateway, VLAN expectations, and adapter names must be documented before virtual switches are created |
| Storage baseline | Volumes, free space, disk layout, and intended VM storage paths must be confirmed before VHDX placement |
| Role state | Hyper-V should not already be partially installed unless this is an audit or repair workflow |
| Pending reboot state | A pending reboot can break role installation, driver changes, NIC binding changes, or management tool installation |
| Baseline artifact | The exported text, CSV, and JSON output used as proof of pre-change state |

# 01_Confirm_Hyper-V_Host_Baseline_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hostname | `HV01` | `<host-name>` |
| Domain or workgroup | `corp.local` or `WORKGROUP` | `<domain-or-workgroup>` |
| Windows Server version | `Windows Server 2022` or `Windows Server 2025` | `<server-version>` |
| Host role intent | Standalone lab host, production host, cluster node | `<host-purpose>` |
| Physical or nested host | Physical server, nested VM, cloud VM | `<host-type>` |
| Management IP | `192.168.10.20/24` | `<management-ip>` |
| Management DNS servers | `192.168.10.10`, `192.168.10.11` | `<dns-servers>` |
| Default gateway | `192.168.10.1` | `<gateway>` |
| Physical NIC count | `2` | `<nic-count>` |
| NIC naming plan | `MGMT`, `VM-SWITCH-UPLINK`, `LIVE-MIGRATION` | `<nic-name-plan>` |
| VM storage volume | `D:\Hyper-V` | `<vm-storage-root>` |
| VHDX storage path | `D:\Hyper-V\Virtual Hard Disks` | `<vhdx-path>` |
| VM config path | `D:\Hyper-V\Virtual Machines` | `<vm-config-path>` |
| Checkpoint path | `D:\Hyper-V\Checkpoints` | `<checkpoint-path>` |
| Memory baseline | `16 GB lab minimum, 32 GB preferred` | `<memory-baseline>` |
| CPU baseline | `4 cores minimum lab, more preferred` | `<cpu-baseline>` |
| Nested virtualization required | Yes or No | `<nested-virtualization-required>` |
| Remote management method | Server Manager, Windows Admin Center, PowerShell Remoting | `<management-method>` |
| Baseline export location | `C:\Admin\HyperV-Baseline` | `<baseline-export-path>` |

# 01_Confirm_Hyper-V_Host_Baseline_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in with administrative rights | Hyper-V host candidate | `whoami /groups` | Current user has local administrator rights |
| 2 | Confirm hostname | Hyper-V host candidate | `hostname` | Hostname matches intended naming plan |
| 3 | Confirm OS version and edition | Hyper-V host candidate | `Get-ComputerInfo \| Select WindowsProductName, WindowsVersion, OsBuildNumber, OsArchitecture` | Supported Windows Server edition is installed |
| 4 | Confirm domain or workgroup state | Hyper-V host candidate | `Get-ComputerInfo \| Select CsName, CsDomain, CsPartOfDomain` | Host is joined to the expected domain or workgroup |
| 5 | Confirm server has no pending reboot | Hyper-V host candidate | `Test-Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending'` | Result is `False` before proceeding |
| 6 | Confirm Hyper-V role is not already installed unexpectedly | Hyper-V host candidate | `Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools` | Role state is known before changes |
| 7 | Confirm CPU virtualization capabilities | Hyper-V host candidate | `Get-CimInstance Win32_Processor \| Select Name, VirtualizationFirmwareEnabled, SecondLevelAddressTranslationExtensions, VMMonitorModeExtensions, DataExecutionPrevention_Available` | Virtualization, SLAT, VM monitor mode, and DEP are available |
| 8 | Confirm firmware virtualization status from systeminfo | Hyper-V host candidate | `systeminfo.exe` | Hyper-V requirements show acceptable values |
| 9 | Confirm physical memory | Hyper-V host candidate | `Get-CimInstance Win32_ComputerSystem \| Select TotalPhysicalMemory` | Memory meets lab or production baseline |
| 10 | Confirm CPU core and logical processor count | Hyper-V host candidate | `Get-CimInstance Win32_Processor \| Select Name, NumberOfCores, NumberOfLogicalProcessors` | CPU capacity meets intended VM workload |
| 11 | Confirm local disks | Hyper-V host candidate | `Get-Disk \| Sort Number \| Format-Table Number,FriendlyName,BusType,PartitionStyle,OperationalStatus,Size` | Disks are online and healthy |
| 12 | Confirm volumes and free space | Hyper-V host candidate | `Get-Volume \| Sort DriveLetter \| Format-Table DriveLetter,FileSystemLabel,FileSystem,HealthStatus,SizeRemaining,Size` | VM storage target has enough free space |
| 13 | Confirm VM storage root exists or document intended path | Hyper-V host candidate | `Test-Path 'D:\Hyper-V'` | Storage path status is known |
| 14 | Confirm physical NIC inventory | Hyper-V host candidate | `Get-NetAdapter \| Sort Name \| Format-Table Name,Status,LinkSpeed,MacAddress,InterfaceDescription` | NIC count, link speed, and status are documented |
| 15 | Confirm IP configuration | Hyper-V host candidate | `Get-NetIPConfiguration` | Management IP, DNS, and gateway are correct |
| 16 | Confirm DNS client servers | Hyper-V host candidate | `Get-DnsClientServerAddress -AddressFamily IPv4` | DNS points to expected internal DNS servers |
| 17 | Confirm network routes | Hyper-V host candidate | `Get-NetRoute -AddressFamily IPv4 \| Sort DestinationPrefix` | Default route and local routes are sane |
| 18 | Confirm firewall profiles | Hyper-V host candidate | `Get-NetFirewallProfile \| Format-Table Name,Enabled,DefaultInboundAction,DefaultOutboundAction` | Firewall state is known before remote management changes |
| 19 | Confirm PowerShell remoting status | Hyper-V host candidate | `Get-Service WinRM` | WinRM state is known for remote management planning |
| 20 | Export baseline evidence | Hyper-V host candidate | `C:\Admin\HyperV-Baseline` | Baseline artifacts exist before Hyper-V role installation |

# 01_Confirm_Hyper-V_Host_Baseline_Host_Inventory_Skeleton

~~~powershell
# 01_Confirm_Hyper-V_Host_Baseline_Host_Inventory_Skeleton
# Purpose:
# Capture the Windows Server identity, OS, domain state, role state, and reboot state before Hyper-V installation.

$BaselineRoot = "C:\Admin\HyperV-Baseline"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $BaselineRoot "HostInventory-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Collecting host identity and OS baseline..." -ForegroundColor Cyan

$ComputerInfo = Get-ComputerInfo | Select-Object `
    CsName,
    CsDomain,
    CsPartOfDomain,
    WindowsProductName,
    WindowsVersion,
    OsBuildNumber,
    OsArchitecture,
    OsHardwareAbstractionLayer,
    CsManufacturer,
    CsModel,
    CsSystemType,
    CsProcessors,
    CsNumberOfLogicalProcessors,
    CsTotalPhysicalMemory

$ComputerInfo |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "01-ComputerInfo.txt") -Encoding UTF8

$ComputerInfo |
    ConvertTo-Json -Depth 4 |
    Out-File -FilePath (Join-Path $OutputPath "01-ComputerInfo.json") -Encoding UTF8

Write-Host "Collecting Windows feature state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Export-Csv -Path (Join-Path $OutputPath "02-HyperVFeatureState.csv") -NoTypeInformation

Write-Host "Checking pending reboot indicators..." -ForegroundColor Cyan

$PendingRebootChecks = [ordered]@{
    ComponentBasedServicing = Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending"
    WindowsUpdate           = Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired"
    PendingFileRename       = $null -ne (Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager" -Name PendingFileRenameOperations -ErrorAction SilentlyContinue)
}

$PendingRebootChecks.GetEnumerator() |
    ForEach-Object {
        [PSCustomObject]@{
            Check = $_.Key
            Pending = $_.Value
        }
    } |
    Export-Csv -Path (Join-Path $OutputPath "03-PendingRebootChecks.csv") -NoTypeInformation

Write-Host "Collecting systeminfo output..." -ForegroundColor Cyan

systeminfo.exe |
    Out-File -FilePath (Join-Path $OutputPath "04-SystemInfo.txt") -Encoding UTF8

Write-Host "Host inventory baseline exported to: $OutputPath" -ForegroundColor Green
~~~

# 01_Confirm_Hyper-V_Host_Baseline_Hardware_Virtualization_Check_Skeleton

~~~powershell
# 01_Confirm_Hyper-V_Host_Baseline_Hardware_Virtualization_Check_Skeleton
# Purpose:
# Confirm CPU, firmware, SLAT, VM monitor mode, DEP, memory, and BIOS data before Hyper-V installation.

$BaselineRoot = "C:\Admin\HyperV-Baseline"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $BaselineRoot "HardwareVirtualization-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Collecting processor virtualization data..." -ForegroundColor Cyan

$ProcessorChecks = Get-CimInstance Win32_Processor | Select-Object `
    Name,
    Manufacturer,
    NumberOfCores,
    NumberOfLogicalProcessors,
    VirtualizationFirmwareEnabled,
    SecondLevelAddressTranslationExtensions,
    VMMonitorModeExtensions,
    DataExecutionPrevention_Available

$ProcessorChecks |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "01-ProcessorChecks.txt") -Encoding UTF8

$ProcessorChecks |
    Export-Csv -Path (Join-Path $OutputPath "01-ProcessorChecks.csv") -NoTypeInformation

Write-Host "Collecting BIOS and firmware data..." -ForegroundColor Cyan

Get-CimInstance Win32_BIOS |
    Select-Object Manufacturer, SMBIOSBIOSVersion, ReleaseDate, SerialNumber |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "02-BIOS.txt") -Encoding UTF8

Get-CimInstance Win32_ComputerSystem |
    Select-Object Manufacturer, Model, SystemType, TotalPhysicalMemory, HypervisorPresent |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "03-ComputerSystem.txt") -Encoding UTF8

Write-Host "Calculating readiness result..." -ForegroundColor Cyan

$ReadinessResults = foreach ($CPU in $ProcessorChecks) {
    [PSCustomObject]@{
        ComputerName = $env:COMPUTERNAME
        ProcessorName = $CPU.Name
        VirtualizationFirmwareEnabled = $CPU.VirtualizationFirmwareEnabled
        SLAT = $CPU.SecondLevelAddressTranslationExtensions
        VMMonitorModeExtensions = $CPU.VMMonitorModeExtensions
        DEPAvailable = $CPU.DataExecutionPrevention_Available
        ReadyForHyperV = (
            $CPU.VirtualizationFirmwareEnabled -eq $true -and
            $CPU.SecondLevelAddressTranslationExtensions -eq $true -and
            $CPU.VMMonitorModeExtensions -eq $true -and
            $CPU.DataExecutionPrevention_Available -eq $true
        )
    }
}

$ReadinessResults |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-HyperVHardwareReadiness.txt")

$ReadinessResults |
    Export-Csv -Path (Join-Path $OutputPath "04-HyperVHardwareReadiness.csv") -NoTypeInformation

$Failed = $ReadinessResults | Where-Object { $_.ReadyForHyperV -ne $true }

if ($Failed) {
    Write-Warning "Host failed one or more Hyper-V hardware readiness checks. Review firmware virtualization, SLAT, VM monitor mode, and DEP."
}
else {
    Write-Host "Host passed Hyper-V hardware readiness checks." -ForegroundColor Green
}

Write-Host "Hardware virtualization baseline exported to: $OutputPath" -ForegroundColor Green
~~~

# 01_Confirm_Hyper-V_Host_Baseline_Network_Storage_Baseline_Skeleton

~~~powershell
# 01_Confirm_Hyper-V_Host_Baseline_Network_Storage_Baseline_Skeleton
# Purpose:
# Capture NIC, IP, DNS, route, disk, and volume baseline before creating Hyper-V switches or VM storage paths.

$BaselineRoot = "C:\Admin\HyperV-Baseline"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $BaselineRoot "NetworkStorage-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Collecting network adapter inventory..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription, DriverInformation |
    Export-Csv -Path (Join-Path $OutputPath "01-NetAdapter.csv") -NoTypeInformation

Get-NetAdapter |
    Sort-Object Name |
    Format-Table Name, Status, LinkSpeed, MacAddress, InterfaceDescription -AutoSize |
    Out-File -FilePath (Join-Path $OutputPath "01-NetAdapter.txt") -Encoding UTF8

if (Get-Command Get-NetAdapterHardwareInfo -ErrorAction SilentlyContinue) {
    Get-NetAdapterHardwareInfo |
        Export-Csv -Path (Join-Path $OutputPath "02-NetAdapterHardwareInfo.csv") -NoTypeInformation
}

Write-Host "Collecting IP configuration..." -ForegroundColor Cyan

Get-NetIPConfiguration |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "03-NetIPConfiguration.txt") -Encoding UTF8

Get-DnsClientServerAddress |
    Export-Csv -Path (Join-Path $OutputPath "04-DnsClientServerAddress.csv") -NoTypeInformation

Get-NetRoute |
    Sort-Object AddressFamily, DestinationPrefix, RouteMetric |
    Export-Csv -Path (Join-Path $OutputPath "05-NetRoute.csv") -NoTypeInformation

Write-Host "Collecting storage inventory..." -ForegroundColor Cyan

Get-Disk |
    Sort-Object Number |
    Select-Object Number, FriendlyName, SerialNumber, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size |
    Export-Csv -Path (Join-Path $OutputPath "06-DiskInventory.csv") -NoTypeInformation

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Export-Csv -Path (Join-Path $OutputPath "07-VolumeInventory.csv") -NoTypeInformation

Get-Volume |
    Sort-Object DriveLetter |
    Format-Table DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size -AutoSize |
    Out-File -FilePath (Join-Path $OutputPath "07-VolumeInventory.txt") -Encoding UTF8

Write-Host "Checking intended Hyper-V storage paths..." -ForegroundColor Cyan

$IntendedPaths = @(
    "D:\Hyper-V",
    "D:\Hyper-V\Virtual Machines",
    "D:\Hyper-V\Virtual Hard Disks",
    "D:\Hyper-V\Checkpoints"
)

$IntendedPaths |
    ForEach-Object {
        [PSCustomObject]@{
            Path = $_
            Exists = Test-Path $_
        }
    } |
    Export-Csv -Path (Join-Path $OutputPath "08-IntendedHyperVPaths.csv") -NoTypeInformation

Write-Host "Network and storage baseline exported to: $OutputPath" -ForegroundColor Green
~~~

# 01_Confirm_Hyper-V_Host_Baseline_Readiness_Report_Skeleton

~~~powershell
# 01_Confirm_Hyper-V_Host_Baseline_Readiness_Report_Skeleton
# Purpose:
# Produce a practical pass/fail style readiness report before installing Hyper-V.

$BaselineRoot = "C:\Admin\HyperV-Baseline"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $BaselineRoot "ReadinessReport-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

$Computer = Get-ComputerInfo
$CPU = Get-CimInstance Win32_Processor
$Features = Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools
$Volumes = Get-Volume
$Adapters = Get-NetAdapter

$PendingReboot = (
    (Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending") -or
    (Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired") -or
    ($null -ne (Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager" -Name PendingFileRenameOperations -ErrorAction SilentlyContinue))
)

$CPUReady = $true
foreach ($Processor in $CPU) {
    if (
        $Processor.VirtualizationFirmwareEnabled -ne $true -or
        $Processor.SecondLevelAddressTranslationExtensions -ne $true -or
        $Processor.VMMonitorModeExtensions -ne $true -or
        $Processor.DataExecutionPrevention_Available -ne $true
    ) {
        $CPUReady = $false
    }
}

$MemoryGB = [math]::Round($Computer.CsTotalPhysicalMemory / 1GB, 2)
$OnlineNICs = $Adapters | Where-Object { $_.Status -eq "Up" }
$CandidateVMVolume = $Volumes | Where-Object { $_.DriveLetter -eq "D" }

$Report = [PSCustomObject]@{
    ComputerName = $env:COMPUTERNAME
    WindowsProductName = $Computer.WindowsProductName
    WindowsVersion = $Computer.WindowsVersion
    OSBuild = $Computer.OsBuildNumber
    Domain = $Computer.CsDomain
    PartOfDomain = $Computer.CsPartOfDomain
    MemoryGB = $MemoryGB
    ProcessorCount = ($CPU | Measure-Object).Count
    LogicalProcessorCount = ($CPU | Measure-Object NumberOfLogicalProcessors -Sum).Sum
    HardwareVirtualizationReady = $CPUReady
    PendingReboot = $PendingReboot
    HyperVInstallState = ($Features | Where-Object Name -eq "Hyper-V").InstallState
    HyperVPowerShellInstallState = ($Features | Where-Object Name -eq "Hyper-V-PowerShell").InstallState
    OnlineNICCount = ($OnlineNICs | Measure-Object).Count
    CandidateVMVolume = if ($CandidateVMVolume) { "D:" } else { "NotFound" }
    CandidateVMVolumeFreeGB = if ($CandidateVMVolume) { [math]::Round($CandidateVMVolume.SizeRemaining / 1GB, 2) } else { 0 }
}

$Report |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-HyperVReadinessReport.txt")

$Report |
    ConvertTo-Json -Depth 4 |
    Out-File -FilePath (Join-Path $OutputPath "01-HyperVReadinessReport.json") -Encoding UTF8

$BlockingIssues = @()

if ($Report.HardwareVirtualizationReady -ne $true) {
    $BlockingIssues += "Hardware virtualization readiness failed."
}

if ($Report.PendingReboot -eq $true) {
    $BlockingIssues += "Pending reboot detected."
}

if ($Report.OnlineNICCount -lt 1) {
    $BlockingIssues += "No online network adapters detected."
}

if ($Report.MemoryGB -lt 8) {
    $BlockingIssues += "Memory is below lab baseline."
}

if ($Report.CandidateVMVolume -eq "D:" -and $Report.CandidateVMVolumeFreeGB -lt 60) {
    $BlockingIssues += "D: exists but free space is low for VM storage."
}

$BlockingIssues |
    Out-File -FilePath (Join-Path $OutputPath "02-BlockingIssues.txt") -Encoding UTF8

if ($BlockingIssues.Count -gt 0) {
    Write-Warning "Host is not cleanly ready for Hyper-V. Review blocking issues:"
    $BlockingIssues | ForEach-Object { Write-Warning $_ }
}
else {
    Write-Host "Host baseline looks ready for Hyper-V role installation." -ForegroundColor Green
}

Write-Host "Readiness report exported to: $OutputPath" -ForegroundColor Green
~~~

# 01_Confirm_Hyper-V_Host_Baseline_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-ComputerInfo \| Select CsName, WindowsProductName, WindowsVersion, OsBuildNumber, CsDomain, CsPartOfDomain` | Hyper-V host candidate | Confirms host identity, OS, build, and domain state |
| `Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools` | Hyper-V host candidate | Confirms current Hyper-V role and tools state |
| `Get-CimInstance Win32_Processor \| Select Name, VirtualizationFirmwareEnabled, SecondLevelAddressTranslationExtensions, VMMonitorModeExtensions, DataExecutionPrevention_Available` | Hyper-V host candidate | Confirms hardware virtualization readiness |
| `systeminfo.exe` | Hyper-V host candidate | Confirms Windows-reported Hyper-V requirement status |
| `Get-NetAdapter \| Format-Table Name,Status,LinkSpeed,MacAddress,InterfaceDescription` | Hyper-V host candidate | Confirms physical NIC status and link speed |
| `Get-NetIPConfiguration` | Hyper-V host candidate | Confirms management IP, DNS, gateway, and interface mapping |
| `Get-DnsClientServerAddress -AddressFamily IPv4` | Hyper-V host candidate | Confirms DNS client configuration |
| `Get-Disk \| Format-Table Number,FriendlyName,BusType,OperationalStatus,HealthStatus,Size` | Hyper-V host candidate | Confirms disk health and visibility |
| `Get-Volume \| Format-Table DriveLetter,FileSystemLabel,FileSystem,HealthStatus,SizeRemaining,Size` | Hyper-V host candidate | Confirms volume health and VM storage capacity |
| `Test-Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending'` | Hyper-V host candidate | Confirms whether a component servicing reboot is pending |
| `Get-Service WinRM` | Hyper-V host candidate | Confirms whether PowerShell remoting can be used for later management |
| `Get-NetFirewallProfile` | Hyper-V host candidate | Confirms firewall profile state before remote management changes |

# 01_Confirm_Hyper-V_Host_Baseline_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove generated baseline folder if test output is not needed | Hyper-V host candidate | `Remove-Item 'C:\Admin\HyperV-Baseline' -Recurse -Force` | Baseline output folder is removed |
| 2 | Revert accidental directory creation only | Hyper-V host candidate | `Remove-Item 'D:\Hyper-V' -Recurse -Force` | Empty test directory is removed if created by mistake |
| 3 | Revert accidental NIC rename | Hyper-V host candidate | `Rename-NetAdapter -Name '<new-name>' -NewName '<old-name>'` | NIC name returns to prior value |
| 4 | Revert accidental DNS server edit | Hyper-V host candidate | `Set-DnsClientServerAddress -InterfaceAlias '<adapter-name>' -ServerAddresses '<original-dns-1>','<original-dns-2>'` | DNS servers return to original baseline |
| 5 | Reboot only if a previous patch or feature operation requires it | Hyper-V host candidate | `Restart-Computer` | Pending reboot clears after restart |
| 6 | Do not uninstall Hyper-V in this workbook | Hyper-V host candidate | `N/A` | This workbook should not install or remove the Hyper-V role |

# 01_Confirm_Hyper-V_Host_Baseline_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `VirtualizationFirmwareEnabled` shows `False` | Intel VT-x or AMD-V disabled in BIOS or UEFI | Enable virtualization in firmware, save settings, power cycle host |
| `SecondLevelAddressTranslationExtensions` shows `False` | CPU lacks SLAT support or nested virtualization is not exposed | Use supported hardware or expose virtualization extensions from parent hypervisor |
| `VMMonitorModeExtensions` shows `False` | CPU virtualization feature unavailable | Enable virtualization in firmware or use compatible CPU |
| `DataExecutionPrevention_Available` shows `False` | DEP/NX unavailable or disabled | Enable DEP/NX/XD in firmware |
| `systeminfo` says hypervisor already detected | Host is already running under a hypervisor or Hyper-V is already installed | Confirm whether this is intended nested virtualization or an existing Hyper-V install |
| `Get-WindowsFeature Hyper-V` shows installed unexpectedly | Host was previously configured | Treat this as audit/remediation instead of pre-install baseline |
| OS edition is not appropriate | Wrong Windows Server edition or client OS used | Rebuild or reinstall with intended Windows Server edition |
| Host has pending reboot | Updates, role changes, or driver installs are incomplete | Reboot before installing Hyper-V |
| No NICs show `Up` | Disconnected cable, driver issue, disabled adapter, or wrong VLAN | Fix physical network, driver, switchport, or adapter state |
| DNS points to public resolvers only | Domain-joined host is not using internal DNS | Set DNS to AD DNS servers before role installation |
| VM storage volume has low free space | VM storage target too small | Add storage, expand volume, or choose another path |
| Disk health is not healthy | Disk, controller, driver, or storage pool issue | Resolve storage health before placing VMs |
| WinRM service is stopped | Remote management is not enabled | Enable remoting later if remote Hyper-V management is required |
| Firewall blocks management | Firewall profile or inbound rules block remote tools | Plan firewall rule changes in the management workbook |
| Nested host fails readiness | Parent hypervisor does not expose virtualization extensions | Enable nested virtualization on the parent platform |

# 01_Confirm_Hyper-V_Host_Baseline_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this baseline task in the full Hyper-V suite |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Uses this baseline as the prerequisite before installing the role |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Uses the storage path decisions captured here |
| 04_Create_And_Configure_Virtual_Switches.md | Uses the NIC inventory and network baseline captured here |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Depends on a validated host, CPU, memory, and storage baseline |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Depends on confirmed disk and volume readiness |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Depends on physical NIC and IP baseline |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Depends on host, network, and storage readiness |
| 15_Configure_Hyper-V_Replica_And_Failover_Testing.md | Depends on stable host identity, networking, and storage |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses this baseline as the known-good comparison point |