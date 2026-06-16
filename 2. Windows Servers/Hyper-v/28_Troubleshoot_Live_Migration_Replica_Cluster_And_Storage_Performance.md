

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Index
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance.md
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Source_Basis
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Mental_Model
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Symptom_Map
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Planning_Table
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Triage_Checklist
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_PreTriage_Capture_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Live_Migration_Triage_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Hyper-V_Replica_Triage_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Cluster_Health_Triage_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_CSV_And_Storage_Performance_Triage_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Network_And_SMB_RDMA_Triage_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Event_Log_And_Cluster_Log_Capture_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Remediation_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_PostFix_Verification_Skeleton
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Verification_Commands
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Rollback
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Failure_Checks
28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Related_Labs

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V live migration | Supports live migration authentication, network, SMB, compression, and migration troubleshooting |
| Microsoft Learn | Hyper-V Replica | Supports replica health, replication state, initial replication, failover, and resynchronization troubleshooting |
| Microsoft Learn | Failover Clustering | Supports cluster group, resource, node, quorum, network, and cluster log troubleshooting |
| Microsoft Learn | Cluster Shared Volumes | Supports CSV ownership, redirected mode, storage path, and VM placement troubleshooting |
| Microsoft Learn | Storage Spaces Direct | Supports S2D pool, physical disk, virtual disk, storage job, and health checks |
| Microsoft Learn | SMB Direct and SMB Multichannel | Supports RDMA, SMB transport, live migration over SMB, and CSV/S2D storage traffic checks |
| Microsoft Learn | Hyper-V performance counters | Supports CPU, memory, virtual switch, virtual storage, and migration performance evidence |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Live migration failure | Running VM cannot move from one Hyper-V host to another without downtime |
| Storage migration failure | VM files or VHDX disks cannot move between storage locations |
| Clustered live migration | Cluster-controlled movement of highly available VM role between nodes |
| Shared-nothing live migration | VM migration between hosts without shared storage |
| Migration authentication | Kerberos, CredSSP, or constrained delegation path used to authorize migration |
| Migration transport | TCP/IP, Compression, or SMB used to move VM memory and state |
| Replica failure | Hyper-V Replica VM cannot replicate, resynchronize, fail over, or maintain healthy RPO |
| Replication health | Normal, Warning, or Critical state for protected VM replication |
| Initial replication | First complete transfer of VM replica data |
| Resynchronization | Repair process after replica divergence or extended replication interruption |
| Cluster health | Combined state of cluster service, nodes, quorum, networks, groups, and resources |
| CSV health | State of Cluster Shared Volumes, including ownership, redirected I/O, and file system state |
| Storage performance | Disk latency, queue depth, throughput, free space, and CSV redirected traffic behavior |
| SMB Direct | RDMA-accelerated SMB transport often used by live migration, CSV, and S2D |
| SMB Multichannel | SMB using multiple network paths for throughput and resiliency |
| RDMA health | State of adapter, driver, firmware, DCB, PFC, ETS, and SMB Direct capability |
| Performance bottleneck | CPU, memory, disk, network, SMB, RDMA, cluster, or VM workload limit causing slow operations |
| Evidence-first triage | Capture cluster, host, VM, event, counter, and storage state before applying fixes |
| Rollback boundary | This workbook diagnoses and safely remediates migration, replica, cluster, and storage performance issues, not application redesign |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Symptom_Map

| Symptom | Primary Area | First Checks |
|---|---|---|
| Live migration fails immediately | Authentication, delegation, VM compatibility, switch mismatch | `Get-VMHost`, VMMS events, cluster events, vSwitch names |
| Live migration starts then times out | Network, bandwidth, SMB, CPU pressure | Counters, migration setting, SMB multichannel, RDMA |
| Live migration is slow | Network, compression CPU, SMB path, storage latency | Migration transport, CPU, NIC throughput, disk latency |
| VM fails after moving to another node | Missing vSwitch, VLAN mismatch, CSV issue, host compatibility | `Get-VMSwitch`, `Get-ClusterSharedVolume`, VMMS events |
| Storage migration fails | Storage permissions, disk lock, low space, path issue | `Get-VMHardDiskDrive`, `Get-VHD`, event logs, free space |
| Replica health warning | RPO breach, network delay, app-consistent snapshot issue | `Get-VMReplication`, Replica events, network tests |
| Replica health critical | Replication stopped, resync required, replica server unreachable | `Get-VMReplication`, `Test-VMReplicationConnection` |
| Initial replication never completes | Bandwidth, firewall, large disk, server auth | Replica server config, port reachability, event logs |
| Replica failover fails | Missing recovery point, replica VM unhealthy, network mapping issue | Recovery points, replica VM state, replica events |
| Cluster group fails over repeatedly | Resource failure, storage issue, network loss, node instability | `Get-ClusterGroup`, resource events, cluster log |
| Cluster node goes down unexpectedly | Cluster service issue, heartbeat loss, hardware fault | Cluster logs, System logs, network state |
| CSV redirected mode appears | Storage path issue, SMB/RDMA issue, node ownership issue | `Get-ClusterSharedVolume`, SMB counters, cluster logs |
| CSV storage is slow | Disk latency, S2D health, redirected I/O, network bottleneck | Storage counters, S2D health, CSV counters |
| S2D pool degraded | Disk failure, node issue, storage job, repair pending | `Get-StoragePool`, `Get-PhysicalDisk`, `Get-StorageJob` |
| SMB Direct not used | RDMA disabled, DCB issue, unsupported NIC, wrong network | `Get-NetAdapterRdma`, `Get-SmbClientNetworkInterface` |
| VM I/O latency high | Storage path, antivirus, backup, checkpoint, CSV, S2D | Disk counters, VHDX chain, storage events |
| VM network throughput low | vSwitch, NIC, VMQ, vRSS, RSS, driver | vSwitch events, NIC counters, advanced adapter state |
| Backup causes replica or storage slowdown | VSS, checkpoint merge, I/O pressure | Checkpoint state, backup window, disk counters |
| Checkpoint merge stalls | Storage pressure or chain issue | AVHDX inventory, free space, VMMS events |
| Failover test is inconsistent | DNS, network isolation, VM dependency, stale recovery point | Replica recovery point, test network, guest services |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Cluster name | `HVCL01` | `<cluster-name>` |
| Source node | `HV01` | `<source-node>` |
| Destination node | `HV02` | `<destination-node>` |
| VM name | `APP01` | `<vm-name>` |
| Replica VM name | `APP01` | `<replica-vm-name>` |
| Replica primary host | `HV01` | `<primary-host>` |
| Replica server | `HVDR01` | `<replica-server>` |
| Replica port | `80`, `443` | `<replica-port>` |
| Authentication method | Kerberos, certificate | `<auth-method>` |
| Live migration auth | Kerberos, CredSSP | `<lm-auth>` |
| Live migration transport | SMB, Compression, TCPIP | `<lm-transport>` |
| Live migration network | `192.168.50.0/24` | `<lm-network>` |
| Storage network | `192.168.60.0/24` | `<storage-network>` |
| CSV name | `CSV01-VMs` | `<csv-name>` |
| CSV path | `C:\ClusterStorage\CSV01-VMs` | `<csv-path>` |
| VM path | `C:\ClusterStorage\CSV01-VMs\VMs\APP01` | `<vm-path>` |
| VHDX path | `C:\ClusterStorage\CSV01-VMs\VHDX\APP01.vhdx` | `<vhdx-path>` |
| S2D used | Yes or No | `<yes-no>` |
| RDMA used | Yes or No | `<yes-no>` |
| SET switch name | `vSwitch-SET-Converged` | `<switch-name>` |
| Target RPO | `5 minutes`, `15 minutes` | `<rpo-target>` |
| Observed RPO | `<minutes>` | `<observed-rpo>` |
| Baseline migration time | `2 minutes` | `<baseline-time>` |
| Observed migration time | `<duration>` | `<observed-time>` |
| Evidence path | `C:\Admin\HyperV-Perf-Troubleshooting` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Triage_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Define exact symptom and scope | Admin workstation | Document VM, node, cluster, time, and error | Problem boundary is known |
| 2 | Capture state before changing anything | Cluster node | Run pre-triage capture skeleton | Evidence is preserved |
| 3 | Confirm Hyper-V services | Each node | `Get-Service vmms, vmcompute` | Services are running |
| 4 | Confirm cluster state | Cluster node | `Get-ClusterNode; Get-ClusterGroup` | Nodes and groups are healthy |
| 5 | Confirm CSV state | Cluster node | `Get-ClusterSharedVolume` | CSVs are online |
| 6 | Confirm VM location and owner | Cluster node | `Get-ClusterGroup -Name '<vm-name>'` | Owner node and role state known |
| 7 | Confirm live migration settings | Each node | `Get-VMHost \| Select *Migration*` | Auth and performance settings known |
| 8 | Confirm source and destination switches | Each node | `Get-VMSwitch` | Required switch exists on both nodes |
| 9 | Confirm VM network adapter mapping | Each node | `Get-VMNetworkAdapter -VMName '<vm-name>'` | VM has expected switch and VLAN |
| 10 | Confirm Replica health | Primary or replica host | `Get-VMReplication -VMName '<vm-name>'` | Replica state and health known |
| 11 | Test Replica server connection | Primary host | `Test-VMReplicationConnection` | Replica path reachable |
| 12 | Confirm storage free space | Each node | `Get-Volume` | CSV and VM storage have free space |
| 13 | Confirm VHDX and checkpoint chain | Owner node | `Get-VMHardDiskDrive; Get-VHD; Get-VMCheckpoint` | Disk path and chain healthy |
| 14 | Confirm S2D health if used | Cluster node | `Get-StoragePool; Get-PhysicalDisk; Get-VirtualDisk` | Storage is healthy |
| 15 | Confirm RDMA and SMB if used | Each node | `Get-NetAdapterRdma; Get-SmbMultichannelConnection` | SMB/RDMA path healthy |
| 16 | Capture performance counters | Each node | Run storage and network triage skeletons | Bottleneck evidence collected |
| 17 | Capture event logs and cluster log | Cluster node | Run event and cluster log skeleton | Relevant events captured |
| 18 | Apply one remediation at a time | Correct node | Use remediation skeleton | Root cause remains traceable |
| 19 | Re-test migration, replica, cluster, or storage path | Admin node | Use verification commands | Fix validated |
| 20 | Capture post-fix evidence | Admin node | Run post-fix verification skeleton | Final state documented |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_PreTriage_Capture_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_PreTriage_Capture_Skeleton
# Purpose:
# Capture cluster, node, VM, live migration, replica, CSV, storage, network, RDMA, and event state before troubleshooting.

$EvidenceRoot = "C:\Admin\HyperV-Perf-Troubleshooting"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$VMName = "APP01"
$Nodes = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "PreTriage-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing admin context..." -ForegroundColor Cyan

whoami |
    Out-File -FilePath (Join-Path $OutputPath "00-AdminContext.txt") -Encoding UTF8

Write-Host "Capturing cluster state..." -ForegroundColor Cyan

try {
    Get-Cluster -Name $ClusterName -ErrorAction Stop |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "01-Cluster-Summary.txt")

    Get-ClusterNode -Cluster $ClusterName |
        Select-Object Name, State, NodeWeight, FaultDomain |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "02-Cluster-Nodes.txt")

    Get-ClusterGroup -Cluster $ClusterName |
        Select-Object Name, OwnerNode, State, GroupType, Priority |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "03-Cluster-Groups.txt")

    Get-ClusterResource -Cluster $ClusterName |
        Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
        Sort-Object OwnerGroup, Name |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "04-Cluster-Resources.txt")

    Get-ClusterNetwork -Cluster $ClusterName |
        Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "05-Cluster-Networks.txt")

    Get-ClusterSharedVolume -Cluster $ClusterName -ErrorAction SilentlyContinue |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "06-Cluster-SharedVolumes.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "01-Cluster-Capture-Error.txt") -Encoding UTF8
}

foreach ($Node in $Nodes) {
    $NodePath = Join-Path $OutputPath $Node
    New-Item -Path $NodePath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing node state from $Node..." -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($VMName)

        "ComputerInfo"
        Get-ComputerInfo |
            Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
            Format-List

        "Hyper-V Services"
        Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
            Select-Object Name, DisplayName, Status, StartType |
            Format-Table -AutoSize

        "VMHost Migration Settings"
        Get-VMHost -ErrorAction SilentlyContinue |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption, MaximumVirtualMachineMigrations, MaximumStorageMigrations |
            Format-List

        "VM Inventory"
        Get-VM -ErrorAction SilentlyContinue |
            Sort-Object Name |
            Select-Object Name, State, Status, Generation, Uptime, ProcessorCount, MemoryAssigned, Path |
            Format-Table -AutoSize

        "Target VM Summary"
        Get-VM -Name $VMName -ErrorAction SilentlyContinue |
            Select-Object Name, Id, State, Status, Generation, Uptime, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path |
            Format-List

        "VM Disks"
        Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
            Format-Table -AutoSize

        "VM Network Adapters"
        Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses, DhcpGuard, RouterGuard, MacAddressSpoofing |
            Format-List

        "VM VLAN"
        Get-VMNetworkAdapterVlan -VMName $VMName -ErrorAction SilentlyContinue |
            Format-List

        "VM Checkpoints"
        Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
            Format-Table -AutoSize

        "VM Replication"
        Get-VMReplication -VMName $VMName -ErrorAction SilentlyContinue |
            Format-List *

        "Replication Server"
        Get-VMReplicationServer -ErrorAction SilentlyContinue |
            Format-List *

        "Virtual Switches"
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
            Format-Table -AutoSize

        "Physical NICs"
        Get-NetAdapter |
            Sort-Object Name |
            Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverVersion |
            Format-Table -AutoSize

        "IP Configuration"
        Get-NetIPConfiguration |
            Format-List

        "RDMA"
        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

        "SMB Client Interfaces"
        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        "SMB Multichannel"
        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize

        "Volumes"
        Get-Volume |
            Sort-Object DriveLetter |
            Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
            Format-Table -AutoSize

        "Disks"
        Get-Disk |
            Sort-Object Number |
            Select-Object Number, FriendlyName, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size, IsOffline, IsReadOnly |
            Format-Table -AutoSize
    } -ArgumentList $VMName |
    Out-File -FilePath (Join-Path $NodePath "Node-PreTriage-State.txt") -Encoding UTF8
}

Write-Host "Capturing storage pool state if available..." -ForegroundColor Cyan

try {
    Get-StoragePool -CimSession $ClusterName -ErrorAction Stop |
        Select-Object FriendlyName, HealthStatus, OperationalStatus, IsPrimordial, Size, AllocatedSize |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "07-Cluster-StoragePools.txt")

    Get-PhysicalDisk -CimSession $ClusterName |
        Select-Object FriendlyName, SerialNumber, MediaType, BusType, Usage, OperationalStatus, HealthStatus, Size |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "08-Cluster-PhysicalDisks.txt")

    Get-VirtualDisk -CimSession $ClusterName |
        Select-Object FriendlyName, ResiliencySettingName, HealthStatus, OperationalStatus, Size, FootprintOnPool |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "09-Cluster-VirtualDisks.txt")

    Get-StorageJob -CimSession $ClusterName -ErrorAction SilentlyContinue |
        Select-Object Name, JobState, PercentComplete, ElapsedTime, BytesProcessed, BytesTotal |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "10-Cluster-StorageJobs.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "07-Cluster-Storage-Capture-Error.txt") -Encoding UTF8
}

Write-Host "Pre-triage evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Live_Migration_Triage_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Live_Migration_Triage_Skeleton
# Purpose:
# Diagnose live migration failures and slow migrations between Hyper-V hosts or cluster nodes.

$ClusterName = "HVCL01"
$VMName = "APP01"
$SourceNode = "HV01"
$DestinationNode = "HV02"

Write-Host "Checking clustered VM role if VM is clustered..." -ForegroundColor Cyan

try {
    Get-ClusterGroup -Cluster $ClusterName -Name $VMName -ErrorAction Stop |
        Select-Object Name, OwnerNode, State, GroupType |
        Format-List

    Get-ClusterResource -Cluster $ClusterName |
        Where-Object { $_.OwnerGroup -eq $VMName } |
        Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
        Format-Table -AutoSize
}
catch {
    Write-Warning "VM may not be clustered or cluster query failed."
    $_
}

Write-Host "Checking live migration settings on source and destination..." -ForegroundColor Cyan

foreach ($Node in @($SourceNode, $DestinationNode)) {
    Write-Host "Node: $Node" -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        Get-VMHost |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption, MaximumVirtualMachineMigrations, MaximumStorageMigrations |
            Format-List

        Get-VMSwitch |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
            Format-Table -AutoSize

        Get-NetIPConfiguration |
            Format-List

        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize
    }
}

Write-Host "Checking source VM compatibility state..." -ForegroundColor Cyan

Invoke-Command -ComputerName $SourceNode -ScriptBlock {
    param($VMName)

    Get-VM -Name $VMName -ErrorAction SilentlyContinue |
        Select-Object Name, State, Status, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryAssigned, Path |
        Format-List

    Get-VMProcessor -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, Count, CompatibilityForMigrationEnabled, ExposeVirtualizationExtensions |
        Format-List

    Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, Name, SwitchName, Status, Connected, IPAddresses |
        Format-List

    Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
        Format-Table -AutoSize
} -ArgumentList $VMName

Write-Host "Testing node-to-node network reachability..." -ForegroundColor Cyan

Invoke-Command -ComputerName $SourceNode -ScriptBlock {
    param($DestinationNode)

    Test-NetConnection $DestinationNode |
        Format-List

    Test-NetConnection $DestinationNode -Port 445 |
        Format-List

    Test-NetConnection $DestinationNode -Port 6600 |
        Format-List
} -ArgumentList $DestinationNode

Write-Host "Checking migration-related events on source and destination..." -ForegroundColor Cyan

foreach ($Node in @($SourceNode, $DestinationNode)) {
    Write-Host "Events from $Node" -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
            Where-Object { $_.Message -match "migration|migrate|move|authentication|storage|network|failed" } |
            Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
            Format-List

        Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
            Where-Object { $_.Message -match "migration|migrate|move|storage|network|failed" } |
            Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
            Format-List
    }
}

Write-Host "Optional controlled migration test after evidence review:" -ForegroundColor Yellow
Write-Host "Clustered VM: Move-ClusterVirtualMachineRole -Name `"$VMName`" -Node `"$DestinationNode`"" -ForegroundColor Yellow
Write-Host "Non-clustered VM: Move-VM -Name `"$VMName`" -DestinationHost `"$DestinationNode`"" -ForegroundColor Yellow
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Hyper-V_Replica_Triage_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Hyper-V_Replica_Triage_Skeleton
# Purpose:
# Diagnose Hyper-V Replica health, connectivity, replication state, recovery points, and resync conditions.

$VMName = "APP01"
$PrimaryHost = "HV01"
$ReplicaServer = "HVDR01"
$ReplicaPort = 80
$AuthenticationType = "Kerberos"

Write-Host "Checking Replica state on primary host..." -ForegroundColor Cyan

Invoke-Command -ComputerName $PrimaryHost -ScriptBlock {
    param($VMName)

    Get-VMReplication -VMName $VMName -ErrorAction SilentlyContinue |
        Format-List *

    Get-VMReplicationServer -ErrorAction SilentlyContinue |
        Format-List *

    Get-VM -Name $VMName -ErrorAction SilentlyContinue |
        Select-Object Name, State, Status, Generation, Uptime, Path |
        Format-List

    Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
        Format-Table -AutoSize

    Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
        Format-Table -AutoSize
} -ArgumentList $VMName

Write-Host "Testing Replica server connectivity..." -ForegroundColor Cyan

Invoke-Command -ComputerName $PrimaryHost -ScriptBlock {
    param($ReplicaServer, $ReplicaPort, $AuthenticationType)

    Test-NetConnection $ReplicaServer -Port $ReplicaPort |
        Format-List

    try {
        Test-VMReplicationConnection `
            -ReplicaServerName $ReplicaServer `
            -ReplicaServerPort $ReplicaPort `
            -AuthenticationType $AuthenticationType
    }
    catch {
        Write-Warning "Test-VMReplicationConnection failed."
        $_
    }
} -ArgumentList $ReplicaServer, $ReplicaPort, $AuthenticationType

Write-Host "Checking Replica server configuration and VM state on replica host..." -ForegroundColor Cyan

Invoke-Command -ComputerName $ReplicaServer -ScriptBlock {
    param($VMName)

    Get-VMReplicationServer -ErrorAction SilentlyContinue |
        Format-List *

    Get-VM -Name $VMName -ErrorAction SilentlyContinue |
        Select-Object Name, State, Status, Generation, Uptime, Path |
        Format-List

    Get-VMReplication -VMName $VMName -ErrorAction SilentlyContinue |
        Format-List *

    Get-VMReplicationAuthorizationEntry -ErrorAction SilentlyContinue |
        Format-Table AllowedPrimaryServer, ReplicaStorageLocation, TrustGroup -AutoSize
} -ArgumentList $VMName

Write-Host "Checking replication events..." -ForegroundColor Cyan

foreach ($Node in @($PrimaryHost, $ReplicaServer)) {
    Write-Host "Replica events from $Node" -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
            Where-Object { $_.Message -match "replica|replication|resynchronization|recovery point|failover|replica server" } |
            Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
            Format-List
    }
}

Write-Host "Potential remediation commands after approval:" -ForegroundColor Yellow
Write-Host "Resume-VMReplication -VMName `"$VMName`"" -ForegroundColor Yellow
Write-Host "Resync-VMReplication -VMName `"$VMName`"" -ForegroundColor Yellow
Write-Host "Start-VMInitialReplication -VMName `"$VMName`"" -ForegroundColor Yellow
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Cluster_Health_Triage_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Cluster_Health_Triage_Skeleton
# Purpose:
# Diagnose cluster node, quorum, group, resource, CSV, and network health.

$ClusterName = "HVCL01"
$VMName = "APP01"

Write-Host "Cluster summary..." -ForegroundColor Cyan

Get-Cluster -Name $ClusterName |
    Format-List *

Write-Host "Cluster nodes..." -ForegroundColor Cyan

Get-ClusterNode -Cluster $ClusterName |
    Select-Object Name, State, NodeWeight, FaultDomain |
    Format-Table -AutoSize

Write-Host "Cluster quorum..." -ForegroundColor Cyan

Get-ClusterQuorum -Cluster $ClusterName |
    Format-List

Write-Host "Cluster groups..." -ForegroundColor Cyan

Get-ClusterGroup -Cluster $ClusterName |
    Select-Object Name, OwnerNode, State, GroupType, Priority |
    Format-Table -AutoSize

Write-Host "Target VM group..." -ForegroundColor Cyan

Get-ClusterGroup -Cluster $ClusterName -Name $VMName -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Resources for target VM group..." -ForegroundColor Cyan

Get-ClusterResource -Cluster $ClusterName |
    Where-Object { $_.OwnerGroup -eq $VMName } |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Format-Table -AutoSize

Write-Host "All failed or offline resources..." -ForegroundColor Cyan

Get-ClusterResource -Cluster $ClusterName |
    Where-Object { $_.State -ne "Online" } |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Format-Table -AutoSize

Write-Host "Cluster networks..." -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize

Write-Host "Cluster network interfaces..." -ForegroundColor Cyan

Get-ClusterNetworkInterface -Cluster $ClusterName |
    Select-Object Name, Node, Network, State, Address |
    Format-Table -AutoSize

Write-Host "Cluster Shared Volumes..." -ForegroundColor Cyan

Get-ClusterSharedVolume -Cluster $ClusterName -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Recent failover clustering events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-FailoverClustering/Operational" -MaxEvents 150 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Write-Host "Cluster health triage completed." -ForegroundColor Green
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_CSV_And_Storage_Performance_Triage_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_CSV_And_Storage_Performance_Triage_Skeleton
# Purpose:
# Diagnose CSV, S2D, disk latency, queue depth, VHDX chain, checkpoint merge, and storage performance issues.

$ClusterName = "HVCL01"
$VMName = "APP01"
$CsvPath = "C:\ClusterStorage\CSV01-VMs"
$SampleInterval = 2
$MaxSamples = 10

Write-Host "Checking CSV state..." -ForegroundColor Cyan

Get-ClusterSharedVolume -Cluster $ClusterName -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "Checking VM disk placement..." -ForegroundColor Cyan

$OwnerNode = (Get-ClusterGroup -Cluster $ClusterName -Name $VMName -ErrorAction SilentlyContinue).OwnerNode.Name

if ($OwnerNode) {
    Invoke-Command -ComputerName $OwnerNode -ScriptBlock {
        param($VMName)

        Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
            Format-Table -AutoSize

        foreach ($Disk in Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue) {
            if ($Disk.Path -and (Test-Path $Disk.Path)) {
                Get-VHD -Path $Disk.Path |
                    Select-Object Path, VhdFormat, VhdType, FileSize, Size, ParentPath, Attached, FragmentationPercentage |
                    Format-List
            }
        }

        Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
            Format-Table -AutoSize
    } -ArgumentList $VMName
}
else {
    Write-Warning "Unable to determine VM owner node."
}

Write-Host "Checking S2D and storage pool state if available..." -ForegroundColor Cyan

try {
    Get-ClusterS2D -CimSession $ClusterName -ErrorAction SilentlyContinue |
        Format-List *

    Get-StoragePool -CimSession $ClusterName -ErrorAction SilentlyContinue |
        Select-Object FriendlyName, HealthStatus, OperationalStatus, IsPrimordial, Size, AllocatedSize |
        Format-Table -AutoSize

    Get-PhysicalDisk -CimSession $ClusterName -ErrorAction SilentlyContinue |
        Select-Object FriendlyName, SerialNumber, MediaType, BusType, Usage, OperationalStatus, HealthStatus, Size |
        Format-Table -AutoSize

    Get-VirtualDisk -CimSession $ClusterName -ErrorAction SilentlyContinue |
        Select-Object FriendlyName, ResiliencySettingName, HealthStatus, OperationalStatus, Size, FootprintOnPool |
        Format-Table -AutoSize

    Get-StorageJob -CimSession $ClusterName -ErrorAction SilentlyContinue |
        Select-Object Name, JobState, PercentComplete, ElapsedTime, BytesProcessed, BytesTotal |
        Format-Table -AutoSize
}
catch {
    Write-Warning "S2D or cluster storage CIM query failed."
    $_
}

Write-Host "Checking CSV folder free space and file inventory..." -ForegroundColor Cyan

if (Test-Path $CsvPath) {
    Get-Item $CsvPath |
        Format-List FullName, CreationTime, LastWriteTime, Attributes

    Get-ChildItem -Path $CsvPath -Recurse -Include "*.vhdx","*.avhdx","*.vhds" -ErrorAction SilentlyContinue |
        Select-Object FullName, Length, CreationTime, LastWriteTime |
        Sort-Object Length -Descending |
        Select-Object -First 50 |
        Format-Table -AutoSize
}
else {
    Write-Warning "CSV path not found from current node: $CsvPath"
}

Write-Host "Sampling storage performance counters..." -ForegroundColor Cyan

$Counters = @(
    "\LogicalDisk(*)\Avg. Disk sec/Read",
    "\LogicalDisk(*)\Avg. Disk sec/Write",
    "\LogicalDisk(*)\Disk Reads/sec",
    "\LogicalDisk(*)\Disk Writes/sec",
    "\LogicalDisk(*)\Disk Read Bytes/sec",
    "\LogicalDisk(*)\Disk Write Bytes/sec",
    "\LogicalDisk(*)\Current Disk Queue Length",
    "\Cluster CSV File System(*)\IO Reads/sec",
    "\Cluster CSV File System(*)\IO Writes/sec",
    "\Cluster CSV File System(*)\IO Read Bytes/sec",
    "\Cluster CSV File System(*)\IO Write Bytes/sec",
    "\Hyper-V Virtual Storage Device(*)\Read Bytes/sec",
    "\Hyper-V Virtual Storage Device(*)\Write Bytes/sec",
    "\Hyper-V Virtual Storage Device(*)\Read Operations/sec",
    "\Hyper-V Virtual Storage Device(*)\Write Operations/sec"
)

Get-Counter -Counter $Counters -SampleInterval $SampleInterval -MaxSamples $MaxSamples -ErrorAction SilentlyContinue

Write-Host "Checking storage-related events..." -ForegroundColor Cyan

Get-WinEvent -LogName "System" -MaxEvents 150 -ErrorAction SilentlyContinue |
    Where-Object {
        $_.ProviderName -match "disk|stor|ntfs|refs|volmgr|volsnap|partmgr|spaceport|csv|smb"
    } |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List

Write-Host "CSV and storage performance triage completed." -ForegroundColor Green
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Network_And_SMB_RDMA_Triage_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Network_And_SMB_RDMA_Triage_Skeleton
# Purpose:
# Diagnose network, SMB, RDMA, vSwitch, SET, RSS, VMQ, and live migration path performance.

$Nodes = @("HV01", "HV02")
$PeerMap = @{
    "HV01" = "HV02"
    "HV02" = "HV01"
}
$SampleInterval = 2
$MaxSamples = 10

foreach ($Node in $Nodes) {
    Write-Host "Network and SMB triage on $Node..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($Peer, $SampleInterval, $MaxSamples)

        "Physical NICs"
        Get-NetAdapter |
            Sort-Object Name |
            Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverVersion |
            Format-Table -AutoSize

        "IP Configuration"
        Get-NetIPConfiguration |
            Format-List

        "Routes"
        Get-NetRoute -AddressFamily IPv4 |
            Sort-Object DestinationPrefix, RouteMetric |
            Format-Table DestinationPrefix, InterfaceAlias, NextHop, RouteMetric, ifMetric -AutoSize

        "vSwitches"
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
            Format-Table -AutoSize

        "SET Teams"
        Get-VMSwitchTeam -ErrorAction SilentlyContinue |
            Format-List *

        "RDMA"
        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

        "RSS"
        Get-NetAdapterRss -ErrorAction SilentlyContinue |
            Select-Object Name, Enabled, BaseProcessorNumber, MaxProcessors, NumberOfReceiveQueues, Profile |
            Format-Table -AutoSize

        "VMQ"
        Get-NetAdapterVmq -ErrorAction SilentlyContinue |
            Select-Object Name, Enabled, BaseProcessorNumber, MaxProcessors, NumberOfReceiveQueues |
            Format-Table -AutoSize

        "SMB Client Network Interfaces"
        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        "SMB Multichannel"
        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRSSCapable, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize

        "Peer reachability"
        Test-NetConnection $Peer |
            Format-List

        Test-NetConnection $Peer -Port 445 |
            Format-List

        "Network counters"
        $Counters = @(
            "\Network Interface(*)\Bytes Total/sec",
            "\Network Interface(*)\Packets/sec",
            "\Network Interface(*)\Output Queue Length",
            "\SMB Client Shares(*)\Avg. sec/Read",
            "\SMB Client Shares(*)\Avg. sec/Write",
            "\SMB Direct Connection(*)\RDMA Inbound Bytes/sec",
            "\SMB Direct Connection(*)\RDMA Outbound Bytes/sec",
            "\Hyper-V Virtual Switch(*)\Bytes/sec",
            "\Hyper-V Virtual Network Adapter(*)\Bytes/sec"
        )

        Get-Counter -Counter $Counters -SampleInterval $SampleInterval -MaxSamples $MaxSamples -ErrorAction SilentlyContinue
    } -ArgumentList $PeerMap[$Node], $SampleInterval, $MaxSamples
}

Write-Host "Network and SMB/RDMA triage completed." -ForegroundColor Green
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Event_Log_And_Cluster_Log_Capture_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Event_Log_And_Cluster_Log_Capture_Skeleton
# Purpose:
# Collect event logs and cluster logs for migration, replica, cluster, CSV, storage, and network troubleshooting.

$EvidenceRoot = "C:\Admin\HyperV-Perf-Troubleshooting"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$VMName = "APP01"
$Nodes = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "Logs-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Generating cluster log..." -ForegroundColor Cyan

try {
    Get-ClusterLog `
        -Cluster $ClusterName `
        -UseLocalTime `
        -TimeSpan 120 `
        -Destination $OutputPath
}
catch {
    Write-Warning "Cluster log generation failed."
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "ClusterLog-Error.txt") -Encoding UTF8
}

$EventLogs = @(
    "Microsoft-Windows-FailoverClustering/Operational",
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-VmSwitch-Operational",
    "Microsoft-Windows-Hyper-V-Hypervisor-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "System",
    "Application"
)

foreach ($Node in $Nodes) {
    $NodePath = Join-Path $OutputPath $Node
    New-Item -Path $NodePath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing events from $Node..." -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($EventLogs)

        foreach ($Log in $EventLogs) {
            $SafeLog = $Log -replace '[\\\/]', "-"

            Get-WinEvent -LogName $Log -MaxEvents 200 -ErrorAction SilentlyContinue |
                Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
                Format-List |
                Out-File -FilePath "C:\Windows\Temp\$SafeLog.txt" -Encoding UTF8
        }

        Get-ChildItem C:\Windows\Temp\*.txt |
            Where-Object { $_.Name -match "Hyper-V|Failover|System|Application" } |
            Select-Object FullName
    } -ArgumentList $EventLogs |
    Out-File -FilePath (Join-Path $NodePath "RemoteEventCapture-Manifest.txt") -Encoding UTF8

    foreach ($Log in $EventLogs) {
        $SafeLog = $Log -replace '[\\\/]', "-"

        try {
            Copy-Item `
                -Path "\\$Node\C$\Windows\Temp\$SafeLog.txt" `
                -Destination (Join-Path $NodePath "$SafeLog.txt") `
                -Force `
                -ErrorAction SilentlyContinue
        }
        catch {
            $_ |
                Out-File -FilePath (Join-Path $NodePath "Copy-$SafeLog-Error.txt") -Encoding UTF8
        }
    }
}

Write-Host "Event and cluster logs exported to: $OutputPath" -ForegroundColor Green
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Remediation_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Remediation_Skeleton
# Purpose:
# Apply one approved remediation at a time.
# Do not run all actions blindly. Capture evidence first.

$Action = "ReviewOnly"

$VMName = "APP01"
$ClusterName = "HVCL01"
$SourceNode = "HV01"
$DestinationNode = "HV02"
$ReplicaServer = "HVDR01"
$ReplicaPort = 80
$AuthenticationType = "Kerberos"

$SwitchName = "vSwitch-External"
$AdapterName = "Network Adapter"

Write-Host "Selected remediation action: $Action" -ForegroundColor Cyan

switch ($Action) {
    "ReviewOnly" {
        Write-Host "No remediation applied." -ForegroundColor Yellow
    }

    "EnableLiveMigration" {
        foreach ($Node in @($SourceNode, $DestinationNode)) {
            Invoke-Command -ComputerName $Node -ScriptBlock {
                Set-VMHost `
                    -VirtualMachineMigrationEnabled $true `
                    -VirtualMachineMigrationAuthenticationType Kerberos `
                    -VirtualMachineMigrationPerformanceOption SMB `
                    -MaximumVirtualMachineMigrations 2 `
                    -MaximumStorageMigrations 2
            }
        }
    }

    "SwitchLiveMigrationToCompression" {
        foreach ($Node in @($SourceNode, $DestinationNode)) {
            Invoke-Command -ComputerName $Node -ScriptBlock {
                Set-VMHost `
                    -VirtualMachineMigrationPerformanceOption Compression
            }
        }
    }

    "SwitchLiveMigrationToSMB" {
        foreach ($Node in @($SourceNode, $DestinationNode)) {
            Invoke-Command -ComputerName $Node -ScriptBlock {
                Set-VMHost `
                    -VirtualMachineMigrationPerformanceOption SMB
            }
        }
    }

    "EnableProcessorCompatibility" {
        Invoke-Command -ComputerName $SourceNode -ScriptBlock {
            param($VMName)

            if ((Get-VM -Name $VMName).State -ne "Off") {
                Write-Warning "Processor compatibility usually requires VM off. Stop VM during approved window."
            }

            Set-VMProcessor `
                -VMName $VMName `
                -CompatibilityForMigrationEnabled $true
        } -ArgumentList $VMName
    }

    "ReconnectVMNetworkAdapter" {
        Invoke-Command -ComputerName $SourceNode -ScriptBlock {
            param($VMName, $AdapterName, $SwitchName)

            Connect-VMNetworkAdapter `
                -VMName $VMName `
                -Name $AdapterName `
                -SwitchName $SwitchName
        } -ArgumentList $VMName, $AdapterName, $SwitchName
    }

    "ResumeReplica" {
        Invoke-Command -ComputerName $SourceNode -ScriptBlock {
            param($VMName)

            Resume-VMReplication `
                -VMName $VMName
        } -ArgumentList $VMName
    }

    "ResyncReplica" {
        Invoke-Command -ComputerName $SourceNode -ScriptBlock {
            param($VMName)

            Resync-VMReplication `
                -VMName $VMName
        } -ArgumentList $VMName
    }

    "StartInitialReplication" {
        Invoke-Command -ComputerName $SourceNode -ScriptBlock {
            param($VMName)

            Start-VMInitialReplication `
                -VMName $VMName
        } -ArgumentList $VMName
    }

    "MoveClusteredVM" {
        Move-ClusterVirtualMachineRole `
            -Cluster $ClusterName `
            -Name $VMName `
            -Node $DestinationNode
    }

    "DrainClusterNode" {
        Suspend-ClusterNode `
            -Cluster $ClusterName `
            -Name $SourceNode `
            -Drain
    }

    "ResumeClusterNode" {
        Resume-ClusterNode `
            -Cluster $ClusterName `
            -Name $SourceNode
    }

    "BringCSVOnline" {
        Get-ClusterSharedVolume -Cluster $ClusterName |
            Start-ClusterResource
    }

    "UpdateClusterNetworkRoles" {
        Get-ClusterNetwork -Cluster $ClusterName |
            Select-Object Name, Address, Role, Metric |
            Format-Table -AutoSize

        Write-Warning "Set cluster network roles manually after confirming subnet purpose."
    }

    "EnableRDMAOnSMBVnics" {
        foreach ($Node in @($SourceNode, $DestinationNode)) {
            Invoke-Command -ComputerName $Node -ScriptBlock {
                Get-NetAdapterRdma |
                    Where-Object { $_.Name -match "SMB|Storage|vEthernet" } |
                    ForEach-Object {
                        Enable-NetAdapterRdma -Name $_.Name -ErrorAction SilentlyContinue
                    }
            }
        }
    }

    default {
        throw "Unsupported remediation action: $Action"
    }
}

Write-Host "Remediation action completed. Run post-fix verification." -ForegroundColor Green
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_PostFix_Verification_Skeleton

~~~powershell
# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_PostFix_Verification_Skeleton
# Purpose:
# Verify live migration, replica, cluster, CSV, storage, SMB, RDMA, and VM state after remediation.

$EvidenceRoot = "C:\Admin\HyperV-Perf-Troubleshooting"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$VMName = "APP01"
$Nodes = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "PostFix-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing cluster post-fix state..." -ForegroundColor Cyan

Get-ClusterNode -Cluster $ClusterName |
    Select-Object Name, State, NodeWeight |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-ClusterNodes-After.txt")

Get-ClusterGroup -Cluster $ClusterName |
    Select-Object Name, OwnerNode, State, GroupType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-ClusterGroups-After.txt")

Get-ClusterResource -Cluster $ClusterName |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Sort-Object OwnerGroup, Name |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-ClusterResources-After.txt")

Get-ClusterSharedVolume -Cluster $ClusterName -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "04-CSV-After.txt")

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-ClusterNetworks-After.txt")

foreach ($Node in $Nodes) {
    $NodePath = Join-Path $OutputPath $Node
    New-Item -Path $NodePath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing post-fix state from $Node..." -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($VMName)

        "VMHost Migration Settings"
        Get-VMHost |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption, MaximumVirtualMachineMigrations, MaximumStorageMigrations |
            Format-List

        "VM State"
        Get-VM -Name $VMName -ErrorAction SilentlyContinue |
            Select-Object Name, State, Status, Generation, Uptime, ProcessorCount, MemoryAssigned, Path |
            Format-List

        "Replica State"
        Get-VMReplication -VMName $VMName -ErrorAction SilentlyContinue |
            Format-List *

        "Network"
        Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, SwitchName, Status, Connected, IPAddresses |
            Format-List

        "Disks"
        Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
            Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
            Format-Table -AutoSize

        "RDMA"
        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, Enabled, OperationalState, PFC, ETS -AutoSize

        "SMB Interfaces"
        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        "SMB Multichannel"
        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize

        "Volumes"
        Get-Volume |
            Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
            Format-Table -AutoSize
    } -ArgumentList $VMName |
    Out-File -FilePath (Join-Path $NodePath "Node-PostFix-State.txt") -Encoding UTF8
}

Write-Host "Capturing quick post-fix counters..." -ForegroundColor Cyan

$Counters = @(
    "\Processor(_Total)\% Processor Time",
    "\Network Interface(*)\Bytes Total/sec",
    "\LogicalDisk(*)\Avg. Disk sec/Read",
    "\LogicalDisk(*)\Avg. Disk sec/Write",
    "\Cluster CSV File System(*)\IO Reads/sec",
    "\Cluster CSV File System(*)\IO Writes/sec",
    "\Hyper-V Virtual Storage Device(*)\Read Bytes/sec",
    "\Hyper-V Virtual Storage Device(*)\Write Bytes/sec"
)

Get-Counter -Counter $Counters -SampleInterval 2 -MaxSamples 10 -ErrorAction SilentlyContinue |
    Out-File -FilePath (Join-Path $OutputPath "06-PostFix-Counters.txt") -Encoding UTF8

Write-Host "Post-fix verification evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-ClusterNode` | Cluster node | Cluster node state |
| `Get-ClusterGroup` | Cluster node | Cluster role ownership and state |
| `Get-ClusterGroup -Name '<vm-name>'` | Cluster node | Target clustered VM role state |
| `Get-ClusterResource` | Cluster node | Resource state and ownership |
| `Get-ClusterNetwork` | Cluster node | Cluster network role, subnet, and metrics |
| `Get-ClusterNetworkInterface` | Cluster node | Cluster NIC state per node |
| `Get-ClusterSharedVolume` | Cluster node | CSV online, owner, and redirected state indicators |
| `Get-ClusterQuorum` | Cluster node | Quorum and witness state |
| `Get-ClusterLog -UseLocalTime -TimeSpan 120 -Destination '<path>'` | Cluster node | Generates cluster log for recent events |
| `Get-VMHost \| Select *Migration*` | Each Hyper-V host | Live migration settings |
| `Set-VMHost -VirtualMachineMigrationEnabled $true -VirtualMachineMigrationPerformanceOption SMB` | Each Hyper-V host | Enables SMB live migration transport |
| `Move-ClusterVirtualMachineRole -Name '<vm-name>' -Node '<node>'` | Cluster node | Tests clustered live migration |
| `Move-VM -Name '<vm-name>' -DestinationHost '<host>'` | Source Hyper-V host | Tests non-clustered live migration |
| `Move-VMStorage -VMName '<vm-name>' -DestinationStoragePath '<path>'` | Hyper-V host | Tests storage migration |
| `Get-VMProcessor -VMName '<vm-name>'` | Hyper-V host | CPU compatibility and virtualization extension state |
| `Set-VMProcessor -VMName '<vm-name>' -CompatibilityForMigrationEnabled $true` | Hyper-V host | Enables CPU compatibility for cross-generation hosts |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | VM switch, MAC, and IP reporting |
| `Get-VMSwitch` | Each Hyper-V host | Confirms matching switch names across nodes |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | VM disk paths |
| `Get-VHD -Path '<vhdx-path>'` | Hyper-V host | VHDX type, parent path, and disk metadata |
| `Get-VMCheckpoint -VMName '<vm-name>'` | Hyper-V host | Checkpoint and AVHDX state |
| `Get-VMReplication -VMName '<vm-name>'` | Primary or replica host | Replica state, health, and statistics |
| `Get-VMReplicationServer` | Replica host | Replica server listener and auth state |
| `Test-VMReplicationConnection -ReplicaServerName '<server>' -ReplicaServerPort '<port>' -AuthenticationType Kerberos` | Primary host | Replica connectivity |
| `Resume-VMReplication -VMName '<vm-name>'` | Primary host | Resumes paused replication |
| `Resync-VMReplication -VMName '<vm-name>'` | Primary host | Starts replica resynchronization |
| `Start-VMInitialReplication -VMName '<vm-name>'` | Primary host | Starts initial replication |
| `Get-StoragePool` | Cluster node | Storage pool health |
| `Get-PhysicalDisk` | Cluster node | Physical disk health |
| `Get-VirtualDisk` | Cluster node | Virtual disk health |
| `Get-StorageJob` | Cluster node | Storage repair, optimization, or regeneration jobs |
| `Get-NetAdapterRdma` | Each node | RDMA state |
| `Get-SmbClientNetworkInterface` | Each node | SMB Direct capability |
| `Get-SmbMultichannelConnection` | Each node | Active SMB channels and RDMA status |
| `Test-NetConnection '<peer-node>' -Port 445` | Each node | SMB path reachability |
| `Get-Counter '\LogicalDisk(*)\Avg. Disk sec/Read'` | Each node | Disk read latency |
| `Get-Counter '\LogicalDisk(*)\Avg. Disk sec/Write'` | Each node | Disk write latency |
| `Get-Counter '\Cluster CSV File System(*)\IO Reads/sec'` | Cluster node | CSV read activity |
| `Get-Counter '\Hyper-V Virtual Storage Device(*)\Read Bytes/sec'` | Hyper-V host | VM storage throughput |
| `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 50` | Cluster node | Cluster events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 50` | Hyper-V host | VMMS and migration events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 50` | Hyper-V host | VM worker events |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current state before rollback | Cluster node | Run post-fix verification skeleton | Current state documented |
| 2 | Restore live migration transport | Each host | `Set-VMHost -VirtualMachineMigrationPerformanceOption '<old-value>'` | Migration transport returns to baseline |
| 3 | Restore live migration auth type | Each host | `Set-VMHost -VirtualMachineMigrationAuthenticationType '<old-value>'` | Auth type returns to baseline |
| 4 | Restore migration concurrency | Each host | `Set-VMHost -MaximumVirtualMachineMigrations '<old>' -MaximumStorageMigrations '<old>'` | Migration limits return to baseline |
| 5 | Disable CPU compatibility if it was test-only | VM host | `Set-VMProcessor -VMName '<vm-name>' -CompatibilityForMigrationEnabled $false` | Processor setting returns to baseline |
| 6 | Move VM back to original owner if required | Cluster node | `Move-ClusterVirtualMachineRole -Name '<vm-name>' -Node '<original-node>'` | VM role returns to original node |
| 7 | Resume drained node | Cluster node | `Resume-ClusterNode -Name '<node>'` | Node resumes cluster ownership |
| 8 | Restore VM network adapter switch | Hyper-V host | `Connect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -SwitchName '<old-switch>'` | Adapter returns to baseline switch |
| 9 | Restore VLAN configuration | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter>' -Access -VlanId '<old-vlan>'` | VLAN returns to baseline |
| 10 | Suspend Replica only if approved | Primary host | `Suspend-VMReplication -VMName '<vm-name>'` | Replication pauses |
| 11 | Resume Replica after rollback | Primary host | `Resume-VMReplication -VMName '<vm-name>'` | Replication resumes |
| 12 | Cancel test failover if active | Replica host | `Stop-VMFailover -VMName '<vm-name>'` | Test failover cleanup starts |
| 13 | Restore cluster network roles and metrics | Cluster node | Use baseline `Get-ClusterNetwork` values | Cluster networks return to baseline |
| 14 | Disable RDMA test change if required | Each node | `Disable-NetAdapterRdma -Name '<adapter>'` | RDMA state returns to baseline |
| 15 | Restore storage path if migration changed it | Hyper-V host | `Move-VMStorage -VMName '<vm-name>' -DestinationStoragePath '<old-path>'` | VM files return to baseline storage |
| 16 | Capture rollback evidence | Cluster node | Run post-fix verification skeleton | Rollback documented |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Live migration fails with authentication error | Kerberos delegation missing, CredSSP issue, wrong auth type | Configure delegation or use approved auth method |
| Live migration fails with network error | Migration network unreachable or blocked | Test ports, routes, DNS, and cluster network roles |
| Live migration fails after memory copy starts | Network timeout, CPU pressure, or bandwidth issue | Review counters and change migration transport if needed |
| Live migration fails between different CPU generations | CPU compatibility disabled | Enable processor compatibility after outage review |
| VM fails on destination node | vSwitch missing or name mismatch | Create consistent vSwitch on all nodes |
| VM network breaks after migration | VLAN or switch mismatch | Standardize VLAN and adapter configuration |
| Storage migration fails | Destination path missing, permissions, low space, or file lock | Fix storage path, free space, and permissions |
| Replica health warning | RPO breach or app-consistent snapshot failure | Check network, VSS, integration services, and replica jobs |
| Replica health critical | Replication stopped or resync required | Resume or resync replication after root cause review |
| Replica server unreachable | DNS, firewall, port, auth, or service issue | Test connectivity and replica server config |
| Initial replication slow | Large disks, limited bandwidth, host load | Schedule off-hours, throttle properly, or use initial copy method |
| Planned failover fails | Replica not healthy or recovery point unavailable | Repair replication and confirm recovery points |
| Cluster group fails to come online | Resource, storage, network, or dependency issue | Review group resources and cluster log |
| Cluster node isolated | Heartbeat network issue or cluster service problem | Fix network and validate cluster service |
| Quorum lost | Witness unavailable or too many nodes offline | Restore witness or node majority |
| CSV enters redirected mode | Direct I/O path issue, SMB/RDMA failure, storage issue | Check CSV owner, SMB, RDMA, and storage health |
| CSV performance poor | Disk latency, redirected I/O, repair job, or network bottleneck | Review storage counters and S2D jobs |
| S2D pool degraded | Disk or node fault | Review physical disks and storage jobs |
| Storage job consumes performance | Repair, optimize, or regeneration running | Monitor job and avoid extra workload during repair |
| VHDX latency high | Disk queue, fragmentation, AVHDX chain, backup, or antivirus | Check counters, checkpoint chain, and exclusions |
| Checkpoint merge slow | Storage pressure or active workload churn | Wait, reduce I/O, ensure free space |
| SMB Direct not active | RDMA disabled, unsupported, DCB issue, wrong network | Validate RDMA and SMB interfaces |
| SMB Multichannel uses wrong path | Route metrics, NIC capability, or SMB interface selection | Review IP routes and SMB client interfaces |
| RDMA enabled but not operational | Driver, firmware, PFC, ETS, or switch config issue | Fix NIC and physical network configuration |
| VMQ or RSS causes uneven CPU | Bad processor placement or driver issue | Tune RSS/VMQ or disable for testing |
| Events show access denied | Permissions on storage, cluster object, or delegation | Fix ACLs, CNO permissions, and delegation |
| Fix made issue worse | Multiple settings changed at once | Roll back one change at a time using captured baseline |

# 28_Troubleshoot_Live_Migration_Replica_Cluster_And_Storage_Performance_Related_Labs

| Lab                                                                           | Relationship                                                                        |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| 00_Hyper-V_Index.md                                                           | Places this troubleshooting task in the full Hyper-V suite                          |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Baseline host health is required before performance triage                          |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Hyper-V and clustering tools are required for troubleshooting                       |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md                       | Host paths affect VM storage and migration behavior                                 |
| 04_Create_And_Configure_Virtual_Switches.md                                   | vSwitch consistency is required for live migration success                          |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md                         | CPU and memory settings affect migration and runtime performance                    |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md                             | VHDX health and pathing affect storage performance                                  |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md                   | VM network adapter configuration affects post-migration connectivity                |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md                | Checkpoint chains and merges affect storage performance                             |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md                 | VM lifecycle control supports remediation and rollback                              |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md                              | Backup workload can affect checkpoints, storage, and replica health                 |
| 14_Configure_Live_Migration_And_Storage_Migration.md                          | Provides the baseline migration workflow this task troubleshoots                    |
| 15_Configure_Hyper-V_Replica_And_Failover_Testing.md                          | Provides the baseline Replica workflow this task troubleshoots                      |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md                 | Clustered VM roles are central to this troubleshooting workflow                     |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md         | Monitoring counters feed this triage workflow                                       |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | General VM troubleshooting overlaps with migration and storage failures             |
| 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance.md              | RDMA, SET, SMB, VMQ, RSS, and vSwitch performance affect migration and CSV behavior |
| 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV.md                    | S2D and CSV storage health are major troubleshooting domains                        |
| 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts.md                | Converged cluster networking affects live migration, CSV, SMB, and RDMA performance |
| 26_Configure_Azure_Site_Recovery_For_Hyper-V.md                               | ASR and Replica workflows share DR replication health concepts                      |
| 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM.md                      | WAC and SCVMM provide alternate management evidence and job history                 |
