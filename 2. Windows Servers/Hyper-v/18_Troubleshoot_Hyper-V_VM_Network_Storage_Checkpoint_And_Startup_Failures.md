# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Index
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Source_Basis
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Mental_Model
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Symptom_Map
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Triage_Checklist
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_PreTriage_Capture_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Startup_Triage_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Network_Triage_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Storage_And_VHDX_Triage_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Checkpoint_And_AVHDX_Triage_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Cluster_And_Migration_Triage_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Remediation_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_PostFix_Verification_Skeleton
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Verification_Commands
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Rollback
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Failure_Checks
18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Related_Labs

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V troubleshooting | Supports VM startup, runtime, storage, checkpoint, and network troubleshooting |
| Microsoft Learn | Hyper-V event logs | Supports VMMS, Worker, VMSwitch, Hypervisor, and Config event log collection |
| Microsoft Learn | Hyper-V virtual hard disks | Supports VHDX path, parent chain, AVHDX, and disk integrity review |
| Microsoft Learn | Hyper-V checkpoints | Supports checkpoint inventory, merge behavior, AVHDX chains, and restore troubleshooting |
| Microsoft Learn | Hyper-V virtual networking | Supports virtual switch, adapter, VLAN, MAC, DHCP guard, router guard, and port mirroring checks |
| Microsoft Learn | Hyper-V failover clustering | Supports clustered VM, CSV, cluster group, resource, node, and failover troubleshooting |
| Microsoft Learn | Hyper-V Replica and migration | Supports Replica health, live migration, and storage migration failure review |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VM startup failure | VM cannot power on, resume, boot, or reach guest OS |
| VMMS failure | Hyper-V management plane problem that affects VM operations |
| Worker failure | Per-VM runtime problem involving memory, disk, device, or guest state |
| VMSwitch failure | Virtual switch, adapter, VLAN, MAC, or policy problem affecting connectivity |
| VHDX failure | Virtual disk file missing, locked, corrupt, inaccessible, or chained incorrectly |
| AVHDX failure | Checkpoint differencing disk problem, usually involving parent path or merge state |
| Broken checkpoint chain | VM points to a differencing disk whose parent cannot be found or opened |
| Storage pressure | Low free space, disk latency, offline disk, CSV issue, or permission problem |
| Saved state issue | VM cannot resume because saved memory or device state is stale or incompatible |
| Boot order issue | VM firmware or BIOS points to the wrong boot device |
| Integration service issue | Guest services such as heartbeat, shutdown, VSS, or time sync report unhealthy |
| Clustered VM issue | Cluster resource, CSV, node, network, or ownership problem affects VM availability |
| Replica issue | VM replication state or failover condition blocks normal operation |
| Evidence-first troubleshooting | Capture state before remediation so the fix does not destroy the root cause evidence |
| Rollback boundary | This workbook diagnoses and repairs Hyper-V VM problems, not guest application redesign |

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Symptom_Map

| Symptom | Primary Area | First Checks |
|---|---|---|
| VM will not start | Startup, storage, memory, saved state | `Get-VM`, event logs, VHDX paths, host memory |
| VM stuck in Starting | Startup, worker process, storage | Worker events, VMMS events, VHDX access, checkpoint chain |
| VM stuck in Saved | Saved state | VM state, saved state compatibility, host changes |
| VM fails after checkpoint restore | Checkpoint, AVHDX, guest consistency | `Get-VMCheckpoint`, `Get-VHD`, VMMS events |
| VM cannot delete checkpoint | Checkpoint merge, storage | AVHDX chain, free space, VMMS events |
| VM disk path missing | Storage | `Get-VMHardDiskDrive`, `Test-Path`, permissions |
| VM has no network | VM adapter, switch, VLAN, DHCP | `Get-VMNetworkAdapter`, `Get-VMSwitch`, VLAN state |
| VM has wrong VLAN | VLAN configuration | `Get-VMNetworkAdapterVlan`, physical switch trunk/access config |
| DHCP does not work inside VM | DHCP, VLAN, guard, switch | DHCP guard, VLAN, guest adapter, DHCP scope |
| Router or firewall VM cannot route | Router guard, MAC spoofing, VLAN trunk | Router guard, MAC spoofing, trunk VLAN list |
| VM loses network after migration | Switch mismatch, VLAN mismatch | Destination switch name, adapter mapping, VLAN config |
| Clustered VM fails over but will not start | Cluster, CSV, storage, switch | Cluster group, resources, CSV, node switch names |
| Live migration fails | Migration network, auth, CPU, storage | VMMS events, migration config, switch names, CPU compatibility |
| Replica test failover fails | Replica state, recovery point, switch | `Get-VMReplication`, recovery points, isolated switch |
| Backup fails because of checkpoint | VSS, production checkpoint, AVHDX | Integration services, guest VSS, checkpoint inventory |
| VM performance is terrible | CPU, memory, disk, network | `Measure-VM`, counters, memory demand, disk latency |

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Triage_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify affected VM and host | Hyper-V host | `Get-VM -Name '<vm-name>'` | VM state and host location are known |
| 2 | Capture current VM state before changing anything | Hyper-V host | Run pre-triage capture skeleton | Evidence is preserved |
| 3 | Confirm Hyper-V services | Hyper-V host | `Get-Service vmms, vmcompute` | Services are running |
| 4 | Check VMMS event log | Hyper-V host | `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 50` | Management errors are visible |
| 5 | Check worker event log | Hyper-V host | `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 50` | Runtime errors are visible |
| 6 | Confirm VM disk paths exist | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>' \| Select Path` | All referenced VHDX or AVHDX files exist |
| 7 | Inspect VHDX chain | Hyper-V host | `Get-VHD -Path '<vhdx-or-avhdx-path>'` | Parent path and disk state are known |
| 8 | Confirm host storage health | Hyper-V host | `Get-Volume; Get-Disk` | Volumes and disks are healthy |
| 9 | Confirm checkpoint inventory | Hyper-V host | `Get-VMCheckpoint -VMName '<vm-name>'` | Checkpoint tree is known |
| 10 | Search VM folder for AVHDX files | Hyper-V host | `Get-ChildItem '<vm-path>' -Recurse -Include '*.avhdx','*.vhdx'` | Checkpoint files are identified |
| 11 | Confirm VM network adapter state | Hyper-V host | `Get-VMNetworkAdapter -VMName '<vm-name>'` | Switch, MAC, IP reporting, and status are known |
| 12 | Confirm VLAN state | Hyper-V host | `Get-VMNetworkAdapterVlan -VMName '<vm-name>'` | VLAN mode and IDs are known |
| 13 | Confirm virtual switch state | Hyper-V host | `Get-VMSwitch` | Required switch exists |
| 14 | Check VMSwitch events | Hyper-V host | `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 50` | Switch events are visible |
| 15 | Confirm integration service state | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>'` | Heartbeat, VSS, shutdown, and guest service state are known |
| 16 | Use PowerShell Direct if guest is Windows and running | Hyper-V host | `Invoke-Command -VMName '<vm-name>' -Credential (Get-Credential) -ScriptBlock { hostname }` | Guest can be managed without network |
| 17 | Check cluster state if VM is clustered | Cluster node | `Get-ClusterGroup -Name '<vm-name>'; Get-ClusterResource` | Clustered VM role state is known |
| 18 | Check CSV state if VM is on CSV | Cluster node | `Get-ClusterSharedVolume` | CSV health is known |
| 19 | Check replication state if VM uses Replica | Primary or replica host | `Get-VMReplication -VMName '<vm-name>'` | Replica health and state are known |
| 20 | Apply one remediation at a time | Hyper-V host | Use remediation skeleton | Root cause remains traceable |
| 21 | Verify startup, network, storage, and event state after fix | Hyper-V host | Run post-fix verification skeleton | Fix is documented |

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_PreTriage_Capture_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_PreTriage_Capture_Skeleton
# Purpose:
# Capture evidence before changing VM state, disk paths, checkpoint chains, networking, or cluster resources.

$EvidenceRoot = "C:\Admin\HyperV-Troubleshooting"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreTriage-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing host identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Host-ComputerInfo.txt")

Write-Host "Capturing Hyper-V services..." -ForegroundColor Cyan

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperV-Services.txt")

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction SilentlyContinue

if ($VM) {
    $VM |
        Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SnapshotFileLocation, SmartPagingFilePath, CheckpointType, AutomaticCheckpointsEnabled |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "03-VM-Summary.txt")

    $VM |
        ConvertTo-Json -Depth 10 |
        Out-File -FilePath (Join-Path $OutputPath "03-VM-Summary.json") -Encoding UTF8

    Write-Host "Capturing VM processor and memory..." -ForegroundColor Cyan

    Get-VMProcessor -VMName $VMName -ErrorAction SilentlyContinue |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "04-VM-Processor.txt")

    Get-VMMemory -VMName $VMName -ErrorAction SilentlyContinue |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "05-VM-Memory.txt")

    Write-Host "Capturing VM disks and VHDX chain..." -ForegroundColor Cyan

    $HardDisks = Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue

    $HardDisks |
        Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "06-VM-HardDisks.txt")

    foreach ($Disk in $HardDisks) {
        if ($Disk.Path) {
            $SafeName = [IO.Path]::GetFileNameWithoutExtension($Disk.Path)

            [PSCustomObject]@{
                Path = $Disk.Path
                Exists = Test-Path $Disk.Path
            } |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath "07-VHDPathCheck-$SafeName.txt") -Encoding UTF8

            if (Test-Path $Disk.Path) {
                Get-VHD -Path $Disk.Path -ErrorAction SilentlyContinue |
                    Format-List * |
                    Out-File -FilePath (Join-Path $OutputPath "08-VHD-$SafeName.txt") -Encoding UTF8
            }
        }
    }

    Write-Host "Capturing checkpoint inventory..." -ForegroundColor Cyan

    Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, Name, Id, CreationTime, ParentSnapshotName, SnapshotType |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "09-VM-Checkpoints.txt")

    Write-Host "Searching VM folder for VHDX and AVHDX files..." -ForegroundColor Cyan

    $VMRoot = Split-Path $VM.Path -Parent

    Get-ChildItem -Path $VMRoot -Recurse -Include "*.vhdx","*.avhdx","*.vhds" -ErrorAction SilentlyContinue |
        Select-Object Name, FullName, Length, CreationTime, LastWriteTime |
        Sort-Object FullName |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "10-VMStorage-VHDX-AVHDX-Inventory.txt")

    Write-Host "Capturing VM network state..." -ForegroundColor Cyan

    Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "11-VM-NetworkAdapters.txt")

    Get-VMNetworkAdapterVlan -VMName $VMName -ErrorAction SilentlyContinue |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "12-VM-NetworkAdapterVLAN.txt")

    Write-Host "Capturing integration services..." -ForegroundColor Cyan

    Get-VMIntegrationService -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "13-VM-IntegrationServices.txt")
}
else {
    "VM not found: $VMName" | Out-File -FilePath (Join-Path $OutputPath "03-VM-NotFound.txt") -Encoding UTF8
}

Write-Host "Capturing host switches and storage..." -ForegroundColor Cyan

Get-VMSwitch -ErrorAction SilentlyContinue |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "14-Host-VMSwitches.txt")

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "15-Host-Volumes.txt")

Get-Disk |
    Sort-Object Number |
    Select-Object Number, FriendlyName, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size, IsOffline, IsReadOnly |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "16-Host-Disks.txt")

Write-Host "Capturing critical Hyper-V event logs..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-VmSwitch-Operational",
    "Microsoft-Windows-Hyper-V-Hypervisor-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "System"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', '-'

    Get-WinEvent -LogName $Log -MaxEvents 100 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, MachineName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "17-$SafeLog.txt") -Encoding UTF8
}

Write-Host "Pre-triage evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Startup_Triage_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Startup_Triage_Skeleton
# Purpose:
# Diagnose why a VM cannot start, resume, boot, or reach the guest OS.

$VMName = "LAB-WIN-SRV01"

Write-Host "Checking VM state..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

$VM |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation |
    Format-List

Write-Host "Checking Hyper-V services..." -ForegroundColor Cyan

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, Status, StartType |
    Format-Table -AutoSize

Write-Host "Checking host memory availability..." -ForegroundColor Cyan

Get-CimInstance Win32_OperatingSystem |
    Select-Object TotalVisibleMemorySize, FreePhysicalMemory |
    Format-List

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, MemoryStartup, MemoryAssigned, MemoryDemand, MemoryStatus |
    Format-Table -AutoSize

Write-Host "Checking VM boot firmware or BIOS..." -ForegroundColor Cyan

if ($VM.Generation -eq 2) {
    Get-VMFirmware -VMName $VMName |
        Format-List
}
else {
    Get-VMBios -VMName $VMName |
        Format-List
}

Write-Host "Checking VM disks..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

foreach ($Disk in $HardDisks) {
    if (-not (Test-Path $Disk.Path)) {
        Write-Warning "Missing virtual disk path: $($Disk.Path)"
    }
    else {
        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdType, VhdFormat, FileSize, Size, ParentPath, Attached |
            Format-List
    }
}

Write-Host "Checking DVD ISO paths..." -ForegroundColor Cyan

Get-VMDvdDrive -VMName $VMName -ErrorAction SilentlyContinue |
    ForEach-Object {
        [PSCustomObject]@{
            VMName = $_.VMName
            ControllerType = $_.ControllerType
            ControllerNumber = $_.ControllerNumber
            ControllerLocation = $_.ControllerLocation
            Path = $_.Path
            PathExists = if ($_.Path) { Test-Path $_.Path } else { $null }
        }
    } |
    Format-Table -AutoSize

Write-Host "Checking saved state scenario..." -ForegroundColor Cyan

if ($VM.State -eq "Saved") {
    Write-Warning "VM is in Saved state. If the saved state is stale or incompatible, remove saved state only after approval."
    Write-Host "Potential command after approval: Remove-VMSavedState -VMName `"$VMName`"" -ForegroundColor Yellow
}

Write-Host "Checking recent startup-related Hyper-V events..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "System"
)

foreach ($Log in $Logs) {
    Write-Host "Log: $Log" -ForegroundColor Yellow

    Get-WinEvent -LogName $Log -MaxEvents 25 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List
}

Write-Host "Startup triage completed. Do not force destructive fixes until disk, checkpoint, and event evidence is reviewed." -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Network_Triage_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Network_Triage_Skeleton
# Purpose:
# Diagnose VM network failures involving switch connection, VLAN, MAC, DHCP, router guard, or migration switch mismatch.

$VMName = "LAB-WIN-SRV01"
$GuestTestIP = "192.168.10.51"
$GatewayIP = "192.168.10.1"
$DNSServerIP = "192.168.10.10"

Write-Host "Checking VM network adapters..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName -ErrorAction Stop |
    Select-Object `
        VMName,
        Name,
        SwitchName,
        MacAddress,
        DynamicMacAddressEnabled,
        MacAddressSpoofing,
        DhcpGuard,
        RouterGuard,
        PortMirroring,
        ProtectedNetwork,
        Status,
        Connected,
        IPAddresses |
    Format-List

Write-Host "Checking VM VLAN configuration..." -ForegroundColor Cyan

Get-VMNetworkAdapterVlan -VMName $VMName |
    Format-List

Write-Host "Checking host virtual switches..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
    Format-Table -AutoSize

Write-Host "Checking management OS adapters..." -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS -ErrorAction SilentlyContinue |
    Select-Object Name, SwitchName, MacAddress, Status, IPAddresses |
    Format-List

Write-Host "Checking host physical adapters..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription |
    Format-Table -AutoSize

Write-Host "Checking host IP configuration..." -ForegroundColor Cyan

Get-NetIPConfiguration |
    Format-List

Write-Host "Testing host network reachability..." -ForegroundColor Cyan

Test-NetConnection $GatewayIP |
    Format-List

Test-NetConnection $DNSServerIP |
    Format-List

Write-Host "Testing guest IP from host if known..." -ForegroundColor Cyan

Test-NetConnection $GuestTestIP |
    Format-List

Write-Host "Checking VMSwitch events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VmSwitch-Operational" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Write-Host "Guest-side Windows network checks through PowerShell Direct if available..." -ForegroundColor Cyan

try {
    Invoke-Command -VMName $VMName -Credential (Get-Credential -Message "Enter guest credentials") -ScriptBlock {
        hostname
        Get-NetAdapter | Select-Object Name, Status, LinkSpeed, MacAddress
        Get-NetIPConfiguration
        Get-DnsClientServerAddress
        route print
        ipconfig /all
    }
}
catch {
    Write-Warning "PowerShell Direct guest check failed. Use VM console if needed."
    $_
}

Write-Host "Network triage completed." -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Storage_And_VHDX_Triage_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Storage_And_VHDX_Triage_Skeleton
# Purpose:
# Diagnose missing, locked, inaccessible, corrupt, or poorly performing VHDX storage.

$VMName = "LAB-WIN-SRV01"

Write-Host "Checking VM disk attachments..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName -ErrorAction Stop

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

Write-Host "Checking each disk path and VHD metadata..." -ForegroundColor Cyan

foreach ($Disk in $HardDisks) {
    Write-Host "Disk path: $($Disk.Path)" -ForegroundColor Yellow

    [PSCustomObject]@{
        Path = $Disk.Path
        Exists = Test-Path $Disk.Path
        Extension = [IO.Path]::GetExtension($Disk.Path)
        Directory = Split-Path $Disk.Path -Parent
    } |
    Format-List

    if (Test-Path $Disk.Path) {
        Get-Item $Disk.Path |
            Select-Object FullName, Length, CreationTime, LastWriteTime, Attributes |
            Format-List

        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, MinimumSize, ParentPath, Attached, FragmentationPercentage |
            Format-List

        try {
            Test-VHD -Path $Disk.Path
        }
        catch {
            Write-Warning "Test-VHD failed or is unavailable for path: $($Disk.Path)"
            $_
        }
    }
}

Write-Host "Checking host storage volumes..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize

Write-Host "Checking host disk state..." -ForegroundColor Cyan

Get-Disk |
    Sort-Object Number |
    Select-Object Number, FriendlyName, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size, IsOffline, IsReadOnly |
    Format-Table -AutoSize

Write-Host "Checking for file locks with openfiles if enabled..." -ForegroundColor Cyan
Write-Host "If openfiles is not enabled, skip this and use Resource Monitor or Sysinternals Handle." -ForegroundColor Yellow

cmd.exe /c "openfiles /query /fo table" 2>$null

Write-Host "Checking storage-related event logs..." -ForegroundColor Cyan

Get-WinEvent -LogName "System" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Where-Object {
        $_.ProviderName -match "disk|stor|ntfs|refs|volmgr|volsnap|partmgr"
    } |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Write-Host "Storage and VHDX triage completed." -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Checkpoint_And_AVHDX_Triage_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Checkpoint_And_AVHDX_Triage_Skeleton
# Purpose:
# Diagnose checkpoint failures, AVHDX chains, merge issues, and broken parent disk references.

$VMName = "LAB-WIN-SRV01"

Write-Host "Checking VM checkpoint policy..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

$VM |
    Select-Object Name, State, CheckpointType, AutomaticCheckpointsEnabled, SnapshotFileLocation, Path |
    Format-List

Write-Host "Checking checkpoint inventory..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Id, CreationTime, SnapshotType, ParentSnapshotName |
    Format-Table -AutoSize

Write-Host "Checking current hard disk paths..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

Write-Host "Walking current disk parent chain..." -ForegroundColor Cyan

foreach ($Disk in $HardDisks) {
    $CurrentPath = $Disk.Path
    $Depth = 0

    while ($CurrentPath -and (Test-Path $CurrentPath)) {
        $VHD = Get-VHD -Path $CurrentPath

        [PSCustomObject]@{
            Depth = $Depth
            Path = $VHD.Path
            Type = $VHD.VhdType
            Format = $VHD.VhdFormat
            FileSize = $VHD.FileSize
            Size = $VHD.Size
            ParentPath = $VHD.ParentPath
            Attached = $VHD.Attached
        } |
        Format-List

        $CurrentPath = $VHD.ParentPath
        $Depth++
    }

    if ($CurrentPath -and -not (Test-Path $CurrentPath)) {
        Write-Warning "Broken parent path found: $CurrentPath"
    }
}

Write-Host "Searching VM storage for AVHDX and VHDX files..." -ForegroundColor Cyan

$VMRoot = Split-Path $VM.Path -Parent

Get-ChildItem -Path $VMRoot -Recurse -Include "*.avhdx","*.vhdx" -ErrorAction SilentlyContinue |
    Select-Object Name, FullName, Length, CreationTime, LastWriteTime |
    Sort-Object FullName |
    Format-Table -AutoSize

Write-Host "Checking for automatic checkpoints and stale checkpoint files..." -ForegroundColor Cyan

Get-ChildItem -Path $VMRoot -Recurse -Include "*.avhdx" -ErrorAction SilentlyContinue |
    ForEach-Object {
        $File = $_

        try {
            $VHD = Get-VHD -Path $File.FullName -ErrorAction Stop

            [PSCustomObject]@{
                Name = $File.Name
                FullName = $File.FullName
                FileSize = $File.Length
                ParentPath = $VHD.ParentPath
                ParentExists = if ($VHD.ParentPath) { Test-Path $VHD.ParentPath } else { $null }
                Attached = $VHD.Attached
            }
        }
        catch {
            [PSCustomObject]@{
                Name = $File.Name
                FullName = $File.FullName
                FileSize = $File.Length
                ParentPath = "Unable to read"
                ParentExists = $false
                Attached = "Unknown"
            }
        }
    } |
    Format-Table -AutoSize

Write-Host "Checking recent checkpoint and merge events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Where-Object {
        $_.Message -match "checkpoint|snapshot|merge|avhdx|vhdx|virtual hard disk"
    } |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Write-Host "Checkpoint and AVHDX triage completed." -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Cluster_And_Migration_Triage_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Cluster_And_Migration_Triage_Skeleton
# Purpose:
# Diagnose clustered VM, CSV, live migration, storage migration, and failover state.

$VMName = "CL-VM01"
$ClusterName = "HVCL01"

Write-Host "Importing FailoverClusters module if available..." -ForegroundColor Cyan

if (-not (Get-Module -ListAvailable FailoverClusters)) {
    Write-Warning "FailoverClusters module not available on this host."
    return
}

Import-Module FailoverClusters

Write-Host "Checking cluster state..." -ForegroundColor Cyan

Get-Cluster -Name $ClusterName -ErrorAction SilentlyContinue |
    Format-List *

Get-ClusterNode -Cluster $ClusterName -ErrorAction SilentlyContinue |
    Select-Object Name, State, NodeWeight, FaultDomain |
    Format-Table -AutoSize

Write-Host "Checking clustered VM role..." -ForegroundColor Cyan

Get-ClusterGroup -Cluster $ClusterName -Name $VMName -ErrorAction SilentlyContinue |
    Select-Object Name, OwnerNode, State, GroupType, Priority |
    Format-List

Write-Host "Checking clustered VM resources..." -ForegroundColor Cyan

Get-ClusterResource -Cluster $ClusterName -ErrorAction SilentlyContinue |
    Where-Object { $_.OwnerGroup -eq $VMName } |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Format-Table -AutoSize

Write-Host "Checking CSV state..." -ForegroundColor Cyan

Get-ClusterSharedVolume -Cluster $ClusterName -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Checking cluster networks..." -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName -ErrorAction SilentlyContinue |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize

Write-Host "Checking VM presence on cluster nodes..." -ForegroundColor Cyan

$Nodes = Get-ClusterNode -Cluster $ClusterName | Select-Object -ExpandProperty Name

foreach ($Node in $Nodes) {
    Write-Host "Node: $Node" -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($VMName)

        Get-VM -Name $VMName -ErrorAction SilentlyContinue |
            Select-Object Name, State, Status, Uptime, Path |
            Format-List

        Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
            Format-List

        Get-VMSwitch |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription |
            Format-Table -AutoSize
    } -ArgumentList $VMName
}

Write-Host "Checking cluster event log..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-FailoverClustering/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Write-Host "Cluster and migration triage completed." -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Remediation_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Remediation_Skeleton
# Purpose:
# Apply one approved remediation at a time.
# Do not run every block blindly.
# Capture pre-triage evidence before remediation.

$VMName = "LAB-WIN-SRV01"

# Valid remediation actions:
# StartVM
# StopGracefully
# TurnOff
# RemoveSavedState
# ReconnectNetworkAdapter
# SetAccessVLAN
# SetUntaggedVLAN
# EnableMacSpoofing
# DisableDhcpGuard
# DisableRouterGuard
# EjectMissingISO
# RemoveAllCheckpoints
# MergeCheckpointByDeleting
# MoveVMStorage
# ReattachVHDX
# ResetBootOrderGen2
# ResetBootOrderGen1

$Action = "StartVM"

$SwitchName = "vSwitch-External"
$AdapterName = "Network Adapter"
$AccessVlanId = 20
$VHDXPath = "D:\Hyper-V\Virtual Hard Disks\LAB-WIN-SRV01.vhdx"
$DestinationStoragePath = "E:\Hyper-V-RecoveredStorage\LAB-WIN-SRV01"

Write-Host "Selected remediation action: $Action" -ForegroundColor Cyan

switch ($Action) {
    "StartVM" {
        Start-VM -Name $VMName
    }

    "StopGracefully" {
        Stop-VM -Name $VMName
    }

    "TurnOff" {
        Write-Warning "Hard power off selected. Use only after approval."
        Stop-VM -Name $VMName -TurnOff
    }

    "RemoveSavedState" {
        Write-Warning "Removing saved state discards saved memory/device state."
        Remove-VMSavedState -VMName $VMName
    }

    "ReconnectNetworkAdapter" {
        Connect-VMNetworkAdapter `
            -VMName $VMName `
            -Name $AdapterName `
            -SwitchName $SwitchName
    }

    "SetAccessVLAN" {
        Set-VMNetworkAdapterVlan `
            -VMName $VMName `
            -VMNetworkAdapterName $AdapterName `
            -Access `
            -VlanId $AccessVlanId
    }

    "SetUntaggedVLAN" {
        Set-VMNetworkAdapterVlan `
            -VMName $VMName `
            -VMNetworkAdapterName $AdapterName `
            -Untagged
    }

    "EnableMacSpoofing" {
        Set-VMNetworkAdapter `
            -VMName $VMName `
            -Name $AdapterName `
            -MacAddressSpoofing On
    }

    "DisableDhcpGuard" {
        Set-VMNetworkAdapter `
            -VMName $VMName `
            -Name $AdapterName `
            -DhcpGuard Off
    }

    "DisableRouterGuard" {
        Set-VMNetworkAdapter `
            -VMName $VMName `
            -Name $AdapterName `
            -RouterGuard Off
    }

    "EjectMissingISO" {
        Get-VMDvdDrive -VMName $VMName |
            Where-Object { $_.Path -and -not (Test-Path $_.Path) } |
            ForEach-Object {
                Set-VMDvdDrive `
                    -VMName $VMName `
                    -ControllerNumber $_.ControllerNumber `
                    -ControllerLocation $_.ControllerLocation `
                    -Path $null
            }
    }

    "RemoveAllCheckpoints" {
        Write-Warning "This deletes checkpoint objects and starts merge operations. It does not delete backups."
        Get-VMCheckpoint -VMName $VMName |
            Remove-VMCheckpoint
    }

    "MergeCheckpointByDeleting" {
        Write-Warning "The supported merge path is to remove checkpoints and allow Hyper-V to merge."
        Get-VMCheckpoint -VMName $VMName |
            Remove-VMCheckpoint
    }

    "MoveVMStorage" {
        New-Item -Path $DestinationStoragePath -ItemType Directory -Force | Out-Null

        Move-VMStorage `
            -VMName $VMName `
            -DestinationStoragePath $DestinationStoragePath
    }

    "ReattachVHDX" {
        if (-not (Test-Path $VHDXPath)) {
            throw "VHDX path not found: $VHDXPath"
        }

        Add-VMHardDiskDrive `
            -VMName $VMName `
            -ControllerType SCSI `
            -ControllerNumber 0 `
            -ControllerLocation 0 `
            -Path $VHDXPath
    }

    "ResetBootOrderGen2" {
        $Disk = Get-VMHardDiskDrive -VMName $VMName | Select-Object -First 1

        Set-VMFirmware `
            -VMName $VMName `
            -FirstBootDevice $Disk
    }

    "ResetBootOrderGen1" {
        Set-VMBios `
            -VMName $VMName `
            -StartupOrder IDE,CD,LegacyNetworkAdapter,Floppy
    }

    default {
        throw "Unsupported remediation action: $Action"
    }
}

Write-Host "Remediation action completed. Run post-fix verification next." -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_PostFix_Verification_Skeleton

~~~powershell
# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_PostFix_Verification_Skeleton
# Purpose:
# Verify VM startup, network, storage, checkpoint, integration service, and event state after remediation.

$EvidenceRoot = "C:\Admin\HyperV-Troubleshooting"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostFix-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing final VM state..." -ForegroundColor Cyan

Get-VM -Name $VMName -ErrorAction SilentlyContinue |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SnapshotFileLocation |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-State-AfterFix.txt")

Write-Host "Capturing final disk state..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VM-HardDisks-AfterFix.txt")

foreach ($Disk in $HardDisks) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path -ErrorAction SilentlyContinue |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, ParentPath, Attached |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("03-VHD-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + "-AfterFix.txt")) -Encoding UTF8
    }
}

Write-Host "Capturing final checkpoint state..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Checkpoints-AfterFix.txt")

Write-Host "Capturing final network state..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses, DhcpGuard, RouterGuard, MacAddressSpoofing |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-NetworkAdapters-AfterFix.txt")

Get-VMNetworkAdapterVlan -VMName $VMName -ErrorAction SilentlyContinue |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "06-NetworkAdapterVlan-AfterFix.txt")

Write-Host "Capturing integration services..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-IntegrationServices-AfterFix.txt")

Write-Host "Running optional VM start check if VM is Off..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction SilentlyContinue

if ($VM -and $VM.State -eq "Off") {
    Start-VM -Name $VMName
    Start-Sleep -Seconds 15

    Get-VM -Name $VMName |
        Select-Object Name, State, Status, Uptime, MemoryAssigned, MemoryDemand, MemoryStatus |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "08-StartTest-AfterFix.txt")
}

Write-Host "Capturing recent warning and error events after fix..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-VmSwitch-Operational",
    "System"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', '-'

    Get-WinEvent -FilterHashtable @{
        LogName = $Log
        Level = 1,2,3
        StartTime = (Get-Date).AddHours(-2)
    } -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "09-$SafeLog-WarningsErrors-AfterFix.txt") -Encoding UTF8
}

Write-Host "Post-fix verification evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms VM exists and shows current state |
| `Get-VM -Name '<vm-name>' \| Select Name, State, Status, Uptime` | Hyper-V host | Confirms VM runtime state |
| `Start-VM -Name '<vm-name>'` | Hyper-V host | Tests whether VM can start |
| `Stop-VM -Name '<vm-name>'` | Hyper-V host | Requests graceful shutdown |
| `Stop-VM -Name '<vm-name>' -TurnOff` | Hyper-V host | Forces power off after approval |
| `Remove-VMSavedState -VMName '<vm-name>'` | Hyper-V host | Removes stale saved state after approval |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Shows attached virtual disks |
| `Test-Path '<vhdx-path>'` | Hyper-V host | Confirms disk path exists |
| `Get-VHD -Path '<vhdx-or-avhdx-path>'` | Hyper-V host | Shows VHDX type, parent path, file size, and attachment state |
| `Test-VHD -Path '<vhdx-path>'` | Hyper-V host | Tests virtual hard disk integrity where supported |
| `Get-ChildItem '<vm-path>' -Recurse -Include '*.avhdx','*.vhdx'` | Hyper-V host | Finds checkpoint and disk files |
| `Get-VMCheckpoint -VMName '<vm-name>'` | Hyper-V host | Shows checkpoint inventory |
| `Remove-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>'` | Hyper-V host | Deletes checkpoint and triggers merge |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Shows VM adapter state, switch, MAC, status, and IP reporting |
| `Get-VMNetworkAdapterVlan -VMName '<vm-name>'` | Hyper-V host | Shows VM VLAN configuration |
| `Get-VMSwitch` | Hyper-V host | Confirms switch inventory |
| `Connect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -SwitchName '<switch-name>'` | Hyper-V host | Reconnects VM adapter to a switch |
| `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Access -VlanId '<id>'` | Hyper-V host | Restores access VLAN |
| `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -DhcpGuard Off` | Hyper-V host | Allows DHCP server behavior from approved VM |
| `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -RouterGuard Off` | Hyper-V host | Allows router behavior from approved VM |
| `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -MacAddressSpoofing On` | Hyper-V host | Allows nested, NLB, firewall, or router MAC behavior |
| `Get-VMIntegrationService -VMName '<vm-name>'` | Hyper-V host | Shows heartbeat, shutdown, time sync, VSS, and guest services |
| `Invoke-Command -VMName '<vm-name>' -Credential (Get-Credential) -ScriptBlock { hostname }` | Hyper-V host | Confirms PowerShell Direct into Windows guest |
| `Get-Volume` | Hyper-V host | Confirms volume health and free space |
| `Get-Disk` | Hyper-V host | Confirms disk health and online state |
| `Get-ClusterGroup -Name '<vm-name>'` | Cluster node | Confirms clustered VM role state |
| `Get-ClusterSharedVolume` | Cluster node | Confirms CSV state |
| `Get-VMReplication -VMName '<vm-name>'` | Primary or replica host | Confirms Replica health |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks management errors |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM runtime errors |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 20` | Hyper-V host | Checks virtual switch errors |
| `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 20` | Cluster node | Checks cluster errors |

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Review pre-triage evidence before rollback | Hyper-V host | `Get-ChildItem 'C:\Admin\HyperV-Troubleshooting'` | Original state evidence is available |
| 2 | Restore VM network switch assignment | Hyper-V host | `Connect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -SwitchName '<old-switch>'` | Adapter returns to original switch |
| 3 | Restore VLAN mode | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Access -VlanId '<old-vlan>'` | VLAN returns to baseline |
| 4 | Restore untagged VLAN mode if that was baseline | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Untagged` | Adapter returns to untagged mode |
| 5 | Restore MAC spoofing state | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -MacAddressSpoofing '<old-value>'` | MAC spoofing returns to baseline |
| 6 | Restore DHCP guard state | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -DhcpGuard '<old-value>'` | DHCP guard returns to baseline |
| 7 | Restore router guard state | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -RouterGuard '<old-value>'` | Router guard returns to baseline |
| 8 | Reattach original VHDX path | Hyper-V host | `Add-VMHardDiskDrive -VMName '<vm-name>' -ControllerType '<type>' -ControllerNumber '<number>' -ControllerLocation '<location>' -Path '<old-path>'` | Original disk attachment is restored |
| 9 | Move VM storage back to original path | Hyper-V host | `Move-VMStorage -VMName '<vm-name>' -DestinationStoragePath '<old-path>'` | VM storage returns to original location |
| 10 | Restore Generation 2 boot order | Hyper-V host | `$disk = Get-VMHardDiskDrive -VMName '<vm-name>' \| Select-Object -First 1; Set-VMFirmware -VMName '<vm-name>' -FirstBootDevice $disk` | VM boots from intended disk |
| 11 | Restore Generation 1 boot order | Hyper-V host | `Set-VMBios -VMName '<vm-name>' -StartupOrder IDE,CD,LegacyNetworkAdapter,Floppy` | VM boots from IDE disk first |
| 12 | Restore checkpoint from known-good checkpoint only if approved | Hyper-V host | `Restore-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>' -Confirm:$false` | VM returns to checkpoint state |
| 13 | Move clustered VM back to original owner | Cluster node | `Move-ClusterVirtualMachineRole -Name '<vm-name>' -Node '<old-owner-node>'` | Clustered VM returns to prior node |
| 14 | Restart VM after rollback | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts successfully |
| 15 | Capture rollback evidence | Hyper-V host | Run post-fix verification skeleton | Rollback state is documented |

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| VM will not start | Missing VHDX, low memory, bad checkpoint chain, or stale saved state | Check event logs, disk paths, memory, and saved state |
| VM stuck in saved state | Saved state incompatible after host or VM change | Remove saved state only after approval |
| VM boots to ISO repeatedly | DVD still first boot device | Eject ISO and reset boot order |
| Generation 2 VM does not boot | Wrong firmware boot order or Secure Boot template | Set first boot device and correct Secure Boot template |
| Generation 1 VM does not boot | Boot disk not on IDE or BIOS order wrong | Attach boot disk to IDE and reset BIOS order |
| VHDX path missing | Disk moved, deleted, or path changed | Restore file, reattach correct VHDX, or restore from backup |
| `Get-VHD` shows missing parent | Broken AVHDX or differencing chain | Restore missing parent or recover from backup |
| Checkpoint deletion is slow | Merge still running | Wait for merge, monitor VMMS events and disk activity |
| Checkpoint cannot be removed | File lock, storage issue, or broken chain | Stop backup jobs, check storage health, then retry |
| Host volume is full | Checkpoints, exports, dynamic VHDX growth, or logs consumed space | Free space, move storage, merge checkpoints |
| VM has no network | Adapter disconnected or wrong virtual switch | Reconnect to correct switch |
| VM network works on host A but not host B | Switch names or VLANs differ | Standardize switch names and VLANs |
| VM gets no DHCP lease | DHCP guard on, VLAN mismatch, DHCP unavailable | Disable DHCP guard only for DHCP server VM, fix VLAN or DHCP service |
| Router VM cannot route | Router guard on or MAC spoofing off | Disable router guard and enable MAC spoofing for approved router VM |
| Firewall or NLB VM fails | MAC spoofing disabled | Enable MAC spoofing where the workload requires it |
| VM loses network after migration | Destination switch or VLAN missing | Reconnect adapter and standardize destination host networking |
| VM fails to start on cluster node | CSV inaccessible, switch missing, or resource failed | Check cluster group, resources, CSV, and switch names |
| CSV is redirected or degraded | Storage or cluster network issue | Review CSV state, cluster networks, and storage events |
| Live migration fails | Authentication, CPU compatibility, storage, or network issue | Check migration settings, switch names, CPU compatibility, and VMMS events |
| Replica failover test fails | No recovery point or replica health critical | Check `Get-VMReplication` and complete initial replication |
| Production checkpoint fails | Guest VSS or integration service issue | Check integration services, guest VSS writers, and backup logs |
| VM starts but guest is hung | Guest OS issue, disk latency, or resource pressure | Use console or PowerShell Direct, check counters and guest logs |
| Worker events show device failure | Disk, network adapter, or saved state problem | Review Worker Admin event message and isolate device |
| VMMS events show access denied | NTFS permissions, SMB permissions, or cluster object permissions | Fix permissions on VM paths, shares, or cluster resources |
| Repair action made issue worse | Multiple changes applied without evidence | Roll back one change at a time using pre-triage evidence |

# 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this troubleshooting task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Provides host CPU, memory, storage, network, and role baseline for troubleshooting |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before Hyper-V cmdlets and tools are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Host paths affect VM storage, checkpoint, smart paging, and startup behavior |
| 04_Create_And_Configure_Virtual_Switches.md | Switch configuration is the base layer for VM network troubleshooting |
| 05_Create_Generation_1_And_Generation_2_VMs.md | VM generation, firmware, and boot configuration affect startup troubleshooting |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | CPU and memory sizing affect startup and performance |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | VHDX path, parent chain, and disk health are central to storage troubleshooting |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Adapter, VLAN, MAC, guard, and bandwidth settings are central to network troubleshooting |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Guest services affect heartbeat, shutdown, VSS, backup, and PowerShell Direct |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Checkpoint and AVHDX behavior is central to restore and merge troubleshooting |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Lifecycle commands are used during startup and recovery troubleshooting |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md | PowerShell Direct is a break-glass path when networking is broken |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Backups are the recovery path when repair is unsafe or disk chains are broken |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Migration failures often surface as network, storage, or startup issues |
| 15_Configure_Hyper-V_Replica_And_Failover_Testing.md | Replica state and failover testing can affect VM recovery troubleshooting |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md | Cluster groups, CSVs, and node ownership affect clustered VM troubleshooting |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md | Monitoring evidence feeds this troubleshooting workflow |