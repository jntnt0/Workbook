# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Index
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Source_Basis
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Mental_Model
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Planning_Table
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Configuration_Checklist
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_PreChange_Capture_Skeleton
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Enable_Resource_Metering_Skeleton
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Host_And_VM_Inventory_Skeleton
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Event_Log_Collection_Skeleton
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Performance_Counter_Collection_Skeleton
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Resource_Usage_Report_Skeleton
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_PostChange_Verification_Skeleton
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Verification_Commands
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Rollback
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Failure_Checks
17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Related_Labs

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V monitoring and performance | Supports host and VM performance monitoring concepts |
| Microsoft Learn | Hyper-V resource metering | Supports `Enable-VMResourceMetering`, `Measure-VM`, and usage reporting |
| Microsoft Learn | Hyper-V event logs | Supports VMMS, Worker, VMSwitch, Hypervisor, and Failover Clustering event review |
| Microsoft Learn | Windows performance counters | Supports `Get-Counter`, Performance Monitor, and Data Collector Sets |
| Microsoft Learn | Failover cluster monitoring | Supports clustered VM, CSV, node, and cluster event monitoring where applicable |
| Microsoft Learn | Windows Server event log collection | Supports `Get-WinEvent`, filtering, exporting, and evidence capture |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Host monitoring | Tracks physical Hyper-V server CPU, memory, disk, network, services, and event health |
| VM monitoring | Tracks VM state, uptime, assigned memory, demand, CPU settings, disk paths, network adapters, and integration status |
| Resource metering | Hyper-V feature that records VM CPU, memory, disk, and network usage for reporting |
| Performance counter | Windows metric exposed through Performance Monitor and `Get-Counter` |
| VMMS | Hyper-V Virtual Machine Management Service, core management plane for VMs |
| Worker process | Per-VM runtime process that reports VM execution issues |
| VMSwitch events | Events related to Hyper-V virtual switch, VM adapter, VLAN, and network policy behavior |
| Hypervisor counters | Low-level CPU scheduling and virtualization metrics |
| Memory pressure | Condition where host or guest memory demand exceeds comfortable available capacity |
| Storage latency | Disk response delay that affects VM boot, checkpoint, backup, and workload performance |
| Network throughput | VM and host network traffic volume and possible bottlenecks |
| Cluster monitoring | Tracks cluster nodes, groups, CSVs, networks, quorum, and failover events |
| Baseline | Known-good performance and event snapshot used for comparison |
| Alert threshold | A value that indicates warning or critical condition |
| Evidence bundle | Exported logs, counters, inventory, and reports used for troubleshooting or documentation |
| Rollback boundary | This workbook enables monitoring and reporting, not workload tuning or incident remediation |

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| Cluster name if applicable | `HVCL01` | `<cluster-name>` |
| VM name scope | All VMs, selected VM | `<vm-scope>` |
| Selected VM | `LAB-WIN-SRV01` | `<vm-name>` |
| Monitoring interval | `5 seconds`, `30 seconds`, `60 seconds` | `<interval>` |
| Sample count | `12`, `60`, `120` | `<sample-count>` |
| Resource metering enabled | Yes or No | `<yes-no>` |
| Resource metering reset schedule | Daily, weekly, manual | `<reset-schedule>` |
| CPU warning threshold | `80% sustained` | `<cpu-warning-threshold>` |
| Memory warning threshold | `80% assigned`, `low available host memory` | `<memory-warning-threshold>` |
| Disk warning threshold | `20% free`, high latency | `<disk-warning-threshold>` |
| Network warning threshold | High throughput, packet drops, disconnected adapter | `<network-warning-threshold>` |
| Event lookback window | `24 hours`, `7 days` | `<event-window>` |
| Evidence path | `C:\Admin\HyperV-Monitoring` | `<evidence-path>` |
| Report path | `C:\Admin\HyperV-Monitoring\Reports` | `<report-path>` |
| Perf counter output path | `C:\Admin\HyperV-Monitoring\Counters` | `<counter-path>` |
| Export format | TXT, CSV, JSON, EVTX | `<export-format>` |
| Scheduled monitoring required | Yes or No | `<yes-no>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm Hyper-V services are running | Hyper-V host | `Get-Service vmms, vmcompute` | Required services are running |
| 3 | Capture host hardware and OS baseline | Hyper-V host | `Get-ComputerInfo; Get-CimInstance Win32_Processor; Get-CimInstance Win32_ComputerSystem` | Host baseline is documented |
| 4 | Capture Hyper-V host settings | Hyper-V host | `Get-VMHost` | Host settings are documented |
| 5 | Capture VM inventory | Hyper-V host | `Get-VM` | VM state, uptime, CPU, and memory are visible |
| 6 | Capture VM disk inventory | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>'` | VM disk paths are documented |
| 7 | Capture VM network inventory | Hyper-V host | `Get-VMNetworkAdapter -VMName '<vm-name>'` | VM adapter and IP reporting are documented |
| 8 | Capture integration services | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>'` | Guest service health is documented |
| 9 | Enable resource metering for selected VMs | Hyper-V host | `Enable-VMResourceMetering -VMName '<vm-name>'` | VM metering is enabled |
| 10 | Confirm resource metering data | Hyper-V host | `Measure-VM -Name '<vm-name>'` | Usage data is returned |
| 11 | Capture Hyper-V event logs | Hyper-V host | `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 100` | VMMS events are collected |
| 12 | Capture worker process events | Hyper-V host | `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 100` | Worker events are collected |
| 13 | Capture virtual switch events | Hyper-V host | `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 100` | VMSwitch events are collected |
| 14 | Capture host CPU, memory, disk, and network counters | Hyper-V host | `Get-Counter` counter set | Performance samples are collected |
| 15 | Capture VM resource usage report | Hyper-V host | `Measure-VM` report skeleton | VM resource usage report is generated |
| 16 | Capture cluster state if applicable | Cluster node | `Get-Cluster; Get-ClusterGroup; Get-ClusterSharedVolume` | Cluster state is documented |
| 17 | Capture recent failover cluster events if applicable | Cluster node | `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 100` | Cluster events are collected |
| 18 | Review warning and error events | Hyper-V host | Filter event output by warning and error | Operational problems are identified |
| 19 | Save reports to evidence folder | Hyper-V host | Run post-change verification skeleton | Monitoring evidence is stored |
| 20 | Reset resource metering only after report export | Hyper-V host | `Reset-VMResourceMetering -VMName '<vm-name>'` | Metering window is reset if approved |

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_PreChange_Capture_Skeleton

~~~powershell
# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_PreChange_Capture_Skeleton
# Purpose:
# Capture host, VM, services, storage, network, resource metering, and event log readiness before monitoring changes.

$EvidenceRoot = "C:\Admin\HyperV-Monitoring"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing host identity and OS baseline..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-ComputerInfo-Before.txt")

Write-Host "Capturing Hyper-V feature and services..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperV-Features-Before.txt")

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-HyperV-Services-Before.txt")

Write-Host "Capturing host CPU and memory..." -ForegroundColor Cyan

Get-CimInstance Win32_Processor |
    Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed, VirtualizationFirmwareEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Host-Processors-Before.txt")

Get-CimInstance Win32_ComputerSystem |
    Select-Object Manufacturer, Model, TotalPhysicalMemory, NumberOfProcessors, NumberOfLogicalProcessors, HypervisorPresent |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-Host-ComputerSystem-Before.txt")

Write-Host "Capturing Hyper-V host settings..." -ForegroundColor Cyan

Get-VMHost |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "06-VMHost-Before.txt")

Write-Host "Capturing VM inventory..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, Status, Generation, Uptime, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, MemoryDemand, MemoryStatus, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-VM-Inventory-Before.txt")

Get-VM |
    Sort-Object Name |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "07-VM-Inventory-Before.json") -Encoding UTF8

Write-Host "Capturing VM disk inventory..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMHardDiskDrive -VMName $_.Name -ErrorAction SilentlyContinue |
            Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path
    } |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-VM-DiskInventory-Before.txt")

Write-Host "Capturing VM network inventory..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMNetworkAdapter -VMName $_.Name -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "09-VM-NetworkInventory-Before.txt")

Write-Host "Capturing resource metering state..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Measure-VM -Name $_.Name -ErrorAction SilentlyContinue
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "10-MeasureVM-Before.txt")

Write-Host "Capturing host volumes..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "11-Volumes-Before.txt")

Write-Host "Capturing recent critical Hyper-V events..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-VmSwitch-Operational"
)

foreach ($Log in $Logs) {
    Get-WinEvent -LogName $Log -MaxEvents 50 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath ("12-" + ($Log -replace '[\\\/]', '-') + "-Before.txt")) -Encoding UTF8
}

Write-Host "Pre-change monitoring evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Enable_Resource_Metering_Skeleton

~~~powershell
# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Enable_Resource_Metering_Skeleton
# Purpose:
# Enable Hyper-V resource metering for selected VMs or all VMs.

$Scope = "All"

$SelectedVMs = @(
    "LAB-WIN-SRV01",
    "LAB-LINUX01"
)

Write-Host "Selecting VMs for resource metering..." -ForegroundColor Cyan

if ($Scope -eq "All") {
    $VMs = Get-VM
}
elseif ($Scope -eq "Selected") {
    $VMs = foreach ($VMName in $SelectedVMs) {
        Get-VM -Name $VMName -ErrorAction Stop
    }
}
else {
    throw "Unsupported scope: $Scope. Use All or Selected."
}

Write-Host "Enabling resource metering..." -ForegroundColor Cyan

foreach ($VM in $VMs) {
    Enable-VMResourceMetering -VMName $VM.Name

    Write-Host "Enabled resource metering for: $($VM.Name)" -ForegroundColor Green
}

Write-Host "Resource metering verification:" -ForegroundColor Cyan

$VMs |
    ForEach-Object {
        Measure-VM -Name $_.Name -ErrorAction SilentlyContinue
    } |
    Select-Object VMName, MeteringDuration, AverageProcessorUsage, AverageMemoryUsage, MaximumMemoryUsage, MinimumMemoryUsage, TotalDiskAllocation, NetworkMeteredTrafficReport |
    Format-List
~~~

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Host_And_VM_Inventory_Skeleton

~~~powershell
# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Host_And_VM_Inventory_Skeleton
# Purpose:
# Generate a host and VM inventory report for operational review.

$EvidenceRoot = "C:\Admin\HyperV-Monitoring"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ReportPath = Join-Path $EvidenceRoot "Reports\Inventory-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $ReportPath -ItemType Directory -Force | Out-Null

Write-Host "Generating host summary..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Export-Csv -Path (Join-Path $ReportPath "01-Host-ComputerInfo.csv") -NoTypeInformation

Get-CimInstance Win32_ComputerSystem |
    Select-Object Manufacturer, Model, TotalPhysicalMemory, NumberOfProcessors, NumberOfLogicalProcessors, HypervisorPresent |
    Export-Csv -Path (Join-Path $ReportPath "02-Host-ComputerSystem.csv") -NoTypeInformation

Get-CimInstance Win32_Processor |
    Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed, VirtualizationFirmwareEnabled |
    Export-Csv -Path (Join-Path $ReportPath "03-Host-Processors.csv") -NoTypeInformation

Write-Host "Generating Hyper-V host summary..." -ForegroundColor Cyan

Get-VMHost |
    Select-Object ComputerName, LogicalProcessorCount, MemoryCapacity, VirtualMachinePath, VirtualHardDiskPath, NumaSpanningEnabled, ResourceMeteringSaveInterval, VirtualMachineMigrationEnabled |
    Export-Csv -Path (Join-Path $ReportPath "04-VMHost.csv") -NoTypeInformation

Write-Host "Generating VM summary..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, Id, State, Status, Generation, Uptime, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, MemoryDemand, MemoryStatus, Path |
    Export-Csv -Path (Join-Path $ReportPath "05-VM-Summary.csv") -NoTypeInformation

Write-Host "Generating VM processor summary..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMProcessor -VMName $_.Name |
            Select-Object VMName, Count, Reserve, Maximum, RelativeWeight, CompatibilityForMigrationEnabled, ExposeVirtualizationExtensions
    } |
    Export-Csv -Path (Join-Path $ReportPath "06-VM-Processors.csv") -NoTypeInformation

Write-Host "Generating VM memory summary..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMMemory -VMName $_.Name |
            Select-Object VMName, DynamicMemoryEnabled, Startup, Minimum, Maximum, Buffer, Priority, AssignedMemory, Demand, Status
    } |
    Export-Csv -Path (Join-Path $ReportPath "07-VM-Memory.csv") -NoTypeInformation

Write-Host "Generating VM disk summary..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMHardDiskDrive -VMName $_.Name |
            Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path
    } |
    Export-Csv -Path (Join-Path $ReportPath "08-VM-Disks.csv") -NoTypeInformation

Write-Host "Generating VM network summary..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMNetworkAdapter -VMName $_.Name |
            Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses
    } |
    Export-Csv -Path (Join-Path $ReportPath "09-VM-NetworkAdapters.csv") -NoTypeInformation

Write-Host "Generating integration services summary..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMIntegrationService -VMName $_.Name |
            Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription
    } |
    Export-Csv -Path (Join-Path $ReportPath "10-VM-IntegrationServices.csv") -NoTypeInformation

Write-Host "Inventory report exported to: $ReportPath" -ForegroundColor Green
~~~

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Event_Log_Collection_Skeleton

~~~powershell
# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Event_Log_Collection_Skeleton
# Purpose:
# Collect Hyper-V and related event logs for monitoring and troubleshooting evidence.

$EvidenceRoot = "C:\Admin\HyperV-Monitoring"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EventPath = Join-Path $EvidenceRoot "Events\$env:COMPUTERNAME-$Timestamp"
$LookbackHours = 24

New-Item -Path $EventPath -ItemType Directory -Force | Out-Null

$StartTime = (Get-Date).AddHours(-$LookbackHours)

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-VmSwitch-Operational",
    "Microsoft-Windows-Hyper-V-Hypervisor-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "System",
    "Application"
)

Write-Host "Collecting events since: $StartTime" -ForegroundColor Cyan

foreach ($Log in $Logs) {
    $SafeName = $Log -replace '[\\\/]', '-'

    Write-Host "Collecting log: $Log" -ForegroundColor Yellow

    $Events = Get-WinEvent -FilterHashtable @{
        LogName = $Log
        StartTime = $StartTime
    } -ErrorAction SilentlyContinue

    $Events |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, MachineName, Message |
        Export-Csv -Path (Join-Path $EventPath "$SafeName.csv") -NoTypeInformation

    $Events |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, MachineName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $EventPath "$SafeName.txt") -Encoding UTF8

    Write-Host "Collected $($Events.Count) events from $Log" -ForegroundColor Green
}

Write-Host "Collecting warning and error rollup..." -ForegroundColor Cyan

foreach ($Log in $Logs) {
    Get-WinEvent -FilterHashtable @{
        LogName = $Log
        StartTime = $StartTime
        Level = 1,2,3
    } -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, MachineName, Message
} |
Export-Csv -Path (Join-Path $EventPath "Warning-Error-Critical-Rollup.csv") -NoTypeInformation

Write-Host "Event collection exported to: $EventPath" -ForegroundColor Green
~~~

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Performance_Counter_Collection_Skeleton

~~~powershell
# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Performance_Counter_Collection_Skeleton
# Purpose:
# Collect host and Hyper-V performance counter samples for CPU, memory, disk, network, and Hyper-V behavior.

$EvidenceRoot = "C:\Admin\HyperV-Monitoring"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$CounterPath = Join-Path $EvidenceRoot "Counters\$env:COMPUTERNAME-$Timestamp"

$SampleIntervalSeconds = 5
$MaxSamples = 12

New-Item -Path $CounterPath -ItemType Directory -Force | Out-Null

$Counters = @(
    "\Processor(_Total)\% Processor Time",
    "\System\Processor Queue Length",
    "\Memory\Available MBytes",
    "\Memory\Pages/sec",
    "\LogicalDisk(_Total)\% Free Space",
    "\LogicalDisk(_Total)\Avg. Disk sec/Read",
    "\LogicalDisk(_Total)\Avg. Disk sec/Write",
    "\LogicalDisk(_Total)\Disk Reads/sec",
    "\LogicalDisk(_Total)\Disk Writes/sec",
    "\Network Interface(*)\Bytes Total/sec",
    "\Network Interface(*)\Packets/sec",
    "\Hyper-V Hypervisor Logical Processor(_Total)\% Total Run Time",
    "\Hyper-V Hypervisor Logical Processor(_Total)\% Guest Run Time",
    "\Hyper-V Hypervisor Logical Processor(_Total)\% Hypervisor Run Time",
    "\Hyper-V Dynamic Memory VM(*)\Average Pressure",
    "\Hyper-V Dynamic Memory VM(*)\Current Pressure",
    "\Hyper-V Virtual Storage Device(*)\Read Bytes/sec",
    "\Hyper-V Virtual Storage Device(*)\Write Bytes/sec",
    "\Hyper-V Virtual Network Adapter(*)\Bytes/sec"
)

Write-Host "Collecting performance counters..." -ForegroundColor Cyan

$CounterData = Get-Counter `
    -Counter $Counters `
    -SampleInterval $SampleIntervalSeconds `
    -MaxSamples $MaxSamples `
    -ErrorAction SilentlyContinue

Write-Host "Exporting raw counter sample text..." -ForegroundColor Cyan

$CounterData |
    Out-File -FilePath (Join-Path $CounterPath "01-RawCounterData.txt") -Encoding UTF8

Write-Host "Flattening counter samples to CSV..." -ForegroundColor Cyan

$Flattened = foreach ($Sample in $CounterData) {
    foreach ($CounterSample in $Sample.CounterSamples) {
        [PSCustomObject]@{
            Timestamp = $Sample.Timestamp
            Path = $CounterSample.Path
            InstanceName = $CounterSample.InstanceName
            CookedValue = $CounterSample.CookedValue
            Status = $CounterSample.Status
        }
    }
}

$Flattened |
    Export-Csv -Path (Join-Path $CounterPath "02-CounterSamples.csv") -NoTypeInformation

Write-Host "Counter summary by path:" -ForegroundColor Cyan

$Flattened |
    Group-Object Path |
    ForEach-Object {
        [PSCustomObject]@{
            CounterPath = $_.Name
            Samples = $_.Count
            Average = [math]::Round(($_.Group | Measure-Object CookedValue -Average).Average, 2)
            Minimum = [math]::Round(($_.Group | Measure-Object CookedValue -Minimum).Minimum, 2)
            Maximum = [math]::Round(($_.Group | Measure-Object CookedValue -Maximum).Maximum, 2)
        }
    } |
    Sort-Object CounterPath |
    Export-Csv -Path (Join-Path $CounterPath "03-CounterSummary.csv") -NoTypeInformation

Write-Host "Performance counter data exported to: $CounterPath" -ForegroundColor Green
~~~

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Resource_Usage_Report_Skeleton

~~~powershell
# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Resource_Usage_Report_Skeleton
# Purpose:
# Generate a VM resource usage report using Hyper-V resource metering.

$EvidenceRoot = "C:\Admin\HyperV-Monitoring"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ReportPath = Join-Path $EvidenceRoot "Reports\ResourceUsage-$env:COMPUTERNAME-$Timestamp"

$ResetMeteringAfterExport = $false

New-Item -Path $ReportPath -ItemType Directory -Force | Out-Null

Write-Host "Collecting resource metering data..." -ForegroundColor Cyan

$MeteringData = Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Measure-VM -Name $_.Name -ErrorAction SilentlyContinue
    }

$MeteringData |
    Export-Csv -Path (Join-Path $ReportPath "01-MeasureVM-Raw.csv") -NoTypeInformation

Write-Host "Building readable usage report..." -ForegroundColor Cyan

$UsageReport = $MeteringData |
    Select-Object `
        VMName,
        MeteringDuration,
        AverageProcessorUsage,
        AverageMemoryUsage,
        MaximumMemoryUsage,
        MinimumMemoryUsage,
        TotalDiskAllocation,
        AggregatedAverageNormalizedIOPS,
        AggregatedAverageLatency,
        AggregatedDiskDataRead,
        AggregatedDiskDataWritten,
        AggregatedNormalizedIOCount

$UsageReport |
    Export-Csv -Path (Join-Path $ReportPath "02-VM-ResourceUsage-Summary.csv") -NoTypeInformation

$UsageReport |
    Format-Table -AutoSize |
    Out-File -FilePath (Join-Path $ReportPath "02-VM-ResourceUsage-Summary.txt") -Encoding UTF8

Write-Host "Collecting VM live state summary..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, Status, Uptime, ProcessorCount, DynamicMemoryEnabled, MemoryAssigned, MemoryDemand, MemoryStatus |
    Export-Csv -Path (Join-Path $ReportPath "03-VM-LiveState.csv") -NoTypeInformation

Write-Host "Collecting VM network meter report details..." -ForegroundColor Cyan

$NetworkReports = foreach ($VM in Get-VM | Sort-Object Name) {
    $Measure = Measure-VM -Name $VM.Name -ErrorAction SilentlyContinue

    foreach ($NetworkReport in $Measure.NetworkMeteredTrafficReport) {
        [PSCustomObject]@{
            VMName = $VM.Name
            MeteringDuration = $Measure.MeteringDuration
            NetworkAdapter = $NetworkReport.NetworkAdapter
            LocalAddress = $NetworkReport.LocalAddress
            RemoteAddress = $NetworkReport.RemoteAddress
            Direction = $NetworkReport.Direction
            TotalTraffic = $NetworkReport.TotalTraffic
        }
    }
}

$NetworkReports |
    Export-Csv -Path (Join-Path $ReportPath "04-VM-NetworkMetering.csv") -NoTypeInformation

if ($ResetMeteringAfterExport) {
    Write-Warning "Resetting resource metering data after export."

    Get-VM |
        ForEach-Object {
            Reset-VMResourceMetering -VMName $_.Name
        }
}

Write-Host "Resource usage report exported to: $ReportPath" -ForegroundColor Green
~~~

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_PostChange_Verification_Skeleton

~~~powershell
# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_PostChange_Verification_Skeleton
# Purpose:
# Capture final monitoring state, reports, resource metering, events, counters, and optional cluster state.

$EvidenceRoot = "C:\Admin\HyperV-Monitoring"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing Hyper-V services..." -ForegroundColor Cyan

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-HyperV-Services-After.txt")

Write-Host "Capturing VMHost final state..." -ForegroundColor Cyan

Get-VMHost |
    Select-Object ComputerName, LogicalProcessorCount, MemoryCapacity, ResourceMeteringSaveInterval, VirtualMachinePath, VirtualHardDiskPath, VirtualMachineMigrationEnabled |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMHost-After.txt")

Write-Host "Capturing VM final state..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, Status, Generation, Uptime, ProcessorCount, DynamicMemoryEnabled, MemoryAssigned, MemoryDemand, MemoryStatus |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VM-State-After.txt")

Write-Host "Capturing resource metering final state..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Measure-VM -Name $_.Name -ErrorAction SilentlyContinue
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-MeasureVM-After.txt")

Write-Host "Capturing warning and error event rollup..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-VmSwitch-Operational",
    "Microsoft-Windows-Hyper-V-Hypervisor-Admin",
    "System"
)

foreach ($Log in $Logs) {
    Get-WinEvent -FilterHashtable @{
        LogName = $Log
        Level = 1,2,3
        StartTime = (Get-Date).AddHours(-24)
    } -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Export-Csv -Path (Join-Path $OutputPath (($Log -replace '[\\\/]', '-') + "-WarningsErrors.csv")) -NoTypeInformation
}

Write-Host "Capturing quick performance counter sample..." -ForegroundColor Cyan

$Counters = @(
    "\Processor(_Total)\% Processor Time",
    "\Memory\Available MBytes",
    "\LogicalDisk(_Total)\Avg. Disk sec/Read",
    "\LogicalDisk(_Total)\Avg. Disk sec/Write",
    "\Network Interface(*)\Bytes Total/sec",
    "\Hyper-V Hypervisor Logical Processor(_Total)\% Total Run Time",
    "\Hyper-V Dynamic Memory VM(*)\Current Pressure"
)

Get-Counter -Counter $Counters -SampleInterval 2 -MaxSamples 5 -ErrorAction SilentlyContinue |
    Out-File -FilePath (Join-Path $OutputPath "05-QuickCounterSample.txt") -Encoding UTF8

Write-Host "Capturing cluster state if FailoverClusters module and cluster exist..." -ForegroundColor Cyan

if (Get-Module -ListAvailable FailoverClusters) {
    try {
        Import-Module FailoverClusters -ErrorAction Stop

        Get-Cluster -ErrorAction Stop |
            Format-List |
            Tee-Object -FilePath (Join-Path $OutputPath "06-Cluster-State.txt")

        Get-ClusterGroup |
            Select-Object Name, OwnerNode, State, GroupType |
            Format-Table -AutoSize |
            Tee-Object -FilePath (Join-Path $OutputPath "07-Cluster-Groups.txt")

        Get-ClusterSharedVolume |
            Format-List |
            Tee-Object -FilePath (Join-Path $OutputPath "08-Cluster-SharedVolumes.txt")
    }
    catch {
        $_ |
            Out-File -FilePath (Join-Path $OutputPath "06-Cluster-State-NotAvailable.txt") -Encoding UTF8
    }
}

Write-Host "Post-change monitoring evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-Service vmms, vmcompute` | Hyper-V host | Confirms Hyper-V management and compute services |
| `Get-VMHost` | Hyper-V host | Shows host-level Hyper-V settings |
| `Get-VM` | Hyper-V host | Shows VM state, uptime, CPU, and memory summary |
| `Get-VM -Name '<vm-name>' \| Select Name, State, Status, Uptime, MemoryAssigned, MemoryDemand` | Hyper-V host | Confirms selected VM runtime state |
| `Get-VMProcessor -VMName '<vm-name>'` | Hyper-V host | Shows VM CPU configuration |
| `Get-VMMemory -VMName '<vm-name>'` | Hyper-V host | Shows VM memory configuration and demand |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Shows VM disk paths |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Shows VM adapter status, switch, MAC, and IP reporting |
| `Get-VMIntegrationService -VMName '<vm-name>'` | Hyper-V host | Shows heartbeat, shutdown, VSS, time sync, and guest service state |
| `Enable-VMResourceMetering -VMName '<vm-name>'` | Hyper-V host | Enables metering for selected VM |
| `Measure-VM -Name '<vm-name>'` | Hyper-V host | Shows VM usage metrics |
| `Reset-VMResourceMetering -VMName '<vm-name>'` | Hyper-V host | Resets metering window after report export |
| `Get-Counter '\Processor(_Total)\% Processor Time'` | Hyper-V host | Samples host CPU usage |
| `Get-Counter '\Memory\Available MBytes'` | Hyper-V host | Samples host available memory |
| `Get-Counter '\LogicalDisk(_Total)\Avg. Disk sec/Read'` | Hyper-V host | Samples storage read latency |
| `Get-Counter '\LogicalDisk(_Total)\Avg. Disk sec/Write'` | Hyper-V host | Samples storage write latency |
| `Get-Counter '\Hyper-V Hypervisor Logical Processor(_Total)\% Total Run Time'` | Hyper-V host | Samples hypervisor logical processor utilization |
| `Get-Counter '\Hyper-V Dynamic Memory VM(*)\Current Pressure'` | Hyper-V host | Samples VM dynamic memory pressure |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks VMMS management events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM runtime worker events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 20` | Hyper-V host | Checks virtual switch events |
| `Get-WinEvent -LogName 'System' -MaxEvents 20` | Hyper-V host | Checks host system-level errors |
| `Get-ClusterGroup` | Cluster node | Shows clustered VM and role state |
| `Get-ClusterSharedVolume` | Cluster node | Shows CSV state |
| `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 20` | Cluster node | Checks cluster operational events |

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Export current metering data before changes | Hyper-V host | `Measure-VM \| Export-Csv '<path>\MeasureVM-BeforeRollback.csv' -NoTypeInformation` | Metering data is preserved |
| 2 | Disable resource metering for one VM if no longer required | Hyper-V host | `Disable-VMResourceMetering -VMName '<vm-name>'` | Metering is disabled for selected VM |
| 3 | Disable resource metering for all VMs if lab cleanup requires it | Hyper-V host | `Get-VM \| Disable-VMResourceMetering` | Metering is disabled for all VMs |
| 4 | Remove scheduled monitoring task if created | Hyper-V host | `Unregister-ScheduledTask -TaskName '<task-name>' -Confirm:$false` | Scheduled collection is removed |
| 5 | Stop active data collector set if created | Hyper-V host | `logman stop '<collector-name>'` | Counter collection stops |
| 6 | Delete data collector set if created | Hyper-V host | `logman delete '<collector-name>'` | Collector definition is removed |
| 7 | Archive evidence folder before cleanup | Hyper-V host | `Compress-Archive -Path 'C:\Admin\HyperV-Monitoring' -DestinationPath '<archive-path>'` | Evidence is preserved |
| 8 | Delete evidence folder only if approved | Hyper-V host | `Remove-Item 'C:\Admin\HyperV-Monitoring' -Recurse -Force` | Evidence folder is removed |
| 9 | Confirm no monitoring collector remains active | Hyper-V host | `logman query` | No unwanted collector is running |
| 10 | Capture rollback evidence | Hyper-V host | `Get-VM; Measure-VM -ErrorAction SilentlyContinue; logman query` | Rollback state is documented |

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Measure-VM` returns no useful data | Resource metering not enabled or just enabled | Run `Enable-VMResourceMetering` and allow time to collect data |
| Resource metering data is stale | Metering window has not been reset in a long time | Export data, then use `Reset-VMResourceMetering` if a fresh window is required |
| `Get-Counter` path fails | Counter name unavailable, typo, or role not installed | Confirm counters with `Get-Counter -ListSet *Hyper-V*` |
| Hyper-V event log query fails | Log does not exist on that OS or role not installed | Confirm log name with `Get-WinEvent -ListLog *Hyper-V*` |
| VM shows high memory demand | Guest needs more memory or workload is active | Review `Get-VMMemory`, dynamic memory settings, and guest workload |
| Host memory is low | Too many VMs or oversized startup memory | Right-size VMs or move workloads |
| High processor queue length | CPU contention on host | Review vCPU allocation and host CPU load |
| High hypervisor processor runtime | Host is saturated or VMs are CPU-heavy | Balance workloads, reduce overcommitment, or migrate VMs |
| High disk read or write latency | Storage bottleneck, checkpoint chain, or backup activity | Check VHDX paths, AVHDX files, storage health, and backup windows |
| VM disk usage report seems wrong | Metering not enabled long enough or disk reporting scope mismatch | Confirm metering duration and VHDX allocation separately |
| VM has no IP reporting | Guest integration not ready, unsupported guest, or network issue | Check guest network and integration services |
| VMSwitch events show adapter errors | VLAN, switch, MAC, or policy issue | Review VM adapter settings and switch configuration |
| VMMS events show checkpoint errors | Checkpoint chain or storage issue | Review checkpoint inventory and VHDX parent paths |
| Worker events show runtime failures | VM device, storage, memory, or guest issue | Review selected VM logs, state, and recent changes |
| Cluster group is offline | Cluster role failed or storage/network issue | Review cluster group, resources, CSV, and cluster events |
| CSV shows redirected or degraded state | Storage or cluster network issue | Review CSV state, storage events, and cluster networks |
| Reports folder grows too large | Excessive event or counter retention | Archive and prune old monitoring evidence |
| Scheduled task does not run | Credential, execution policy, or path issue | Test script manually and review Task Scheduler history |
| Counter collection causes overhead | Too many counters or too frequent samples | Increase interval and reduce counter count |
| No baseline exists for comparison | First monitoring run | Save report as baseline and compare future reports to it |

# 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this monitoring task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Provides the baseline host inventory used for monitoring comparison |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before Hyper-V monitoring cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Host paths affect storage monitoring and evidence interpretation |
| 04_Create_And_Configure_Virtual_Switches.md | Virtual switch state affects VMSwitch monitoring |
| 05_Create_Generation_1_And_Generation_2_VMs.md | VM objects created there are monitored here |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | CPU, memory, and NUMA settings are measured and validated here |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | VHDX paths and storage behavior are key monitoring inputs |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | VM adapter and bandwidth settings are monitored here |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Integration service status affects heartbeat, VSS, shutdown, and guest reporting |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Checkpoints can create storage and performance pressure |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Lifecycle operations are visible through VM state and event logs |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Backup jobs affect performance counters, checkpoints, and events |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Migration events and performance are monitored here |
| 15_Configure_Hyper-V_Replica_And_Failover_Testing.md | Replica health and failover events feed monitoring evidence |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md | Cluster groups, CSVs, node state, and cluster logs are monitored here |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses this monitoring evidence to troubleshoot failures |