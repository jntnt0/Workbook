# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Index
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV.md
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Source_Basis
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Mental_Model
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Planning_Table
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Configuration_Checklist
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_PreChange_Capture_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Install_Features_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Cluster_Validation_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_Cluster_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Enable_S2D_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_CSV_Volumes_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Configure_Cluster_Networks_And_Live_Migration_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_Clustered_VM_On_CSV_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Health_And_Performance_Verification_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_PostChange_Verification_Skeleton
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Verification_Commands
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Rollback
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Failure_Checks
24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Related_Labs

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Storage Spaces Direct overview | Supports S2D architecture, pooled local disks, cache, capacity, and cluster requirements |
| Microsoft Learn | Deploy Storage Spaces Direct | Supports validation, cluster creation, enabling S2D, and volume creation workflow |
| Microsoft Learn | Hyper-V with Failover Clustering | Supports highly available VMs on clustered storage |
| Microsoft Learn | Cluster Shared Volumes | Supports CSV paths under `C:\ClusterStorage` for clustered VM storage |
| Microsoft Learn | Failover Cluster validation | Supports `Test-Cluster` before enabling production cluster workloads |
| Microsoft Learn | Storage Spaces PowerShell | Supports `Get-StoragePool`, `Get-PhysicalDisk`, `New-Volume`, and health validation |
| Microsoft Learn | FailoverClusters PowerShell | Supports `New-Cluster`, `Enable-ClusterS2D`, `Get-ClusterSharedVolume`, and clustered VM role workflows |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Storage Spaces Direct | Uses local disks in cluster nodes to create shared software-defined storage |
| S2D cluster | Failover cluster where local disks are pooled across nodes for resilient storage |
| Storage pool | Logical pool created from eligible physical disks across cluster nodes |
| Capacity disks | Physical disks that provide storage capacity to the S2D pool |
| Cache devices | Faster disks used to accelerate reads and writes depending on hardware mix |
| CSV | Cluster Shared Volume, a cluster-accessible volume mounted under `C:\ClusterStorage` |
| CSVFS_ReFS | File system commonly used for S2D CSV volumes |
| Resiliency | Mirror, parity, or nested resiliency layout protecting data across disks and nodes |
| Fault domain | Node, chassis, rack, or site boundary used for placement and resiliency awareness |
| Cluster network | Network used for management, cluster heartbeat, CSV, SMB, live migration, or storage traffic |
| SMB Direct | SMB over RDMA, often used for low-latency storage and CSV traffic |
| Live migration network | Dedicated or prioritized network for moving running VMs between nodes |
| Clustered VM | Hyper-V VM made highly available as a cluster role |
| Witness | Quorum helper such as file share, disk, or cloud witness |
| Validation report | Required health check before cluster and S2D enablement |
| Rollback boundary | This workbook configures Hyper-V with S2D and CSV, not Azure Stack HCI lifecycle management |

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Cluster name | `HV-S2D-CL01` | `<cluster-name>` |
| Cluster IP | `192.168.10.70` | `<cluster-ip>` |
| Domain | `corp.contoso.com` | `<domain-name>` |
| Node 1 | `HV01` | `<node-1>` |
| Node 2 | `HV02` | `<node-2>` |
| Node 3 optional | `HV03` | `<node-3>` |
| Node 4 optional | `HV04` | `<node-4>` |
| Cluster OU | `OU=Servers,DC=corp,DC=contoso,DC=com` | `<cluster-ou>` |
| Witness type | File share, cloud, disk | `<witness-type>` |
| Witness path | `\\FS01\HV-S2D-CL01-Witness$` | `<witness-path>` |
| Management network | `192.168.10.0/24` | `<management-network>` |
| Live migration network | `192.168.50.0/24` | `<live-migration-network>` |
| Storage network 1 | `192.168.60.0/24` | `<storage-network-1>` |
| Storage network 2 | `192.168.61.0/24` | `<storage-network-2>` |
| RDMA required | Yes or No | `<yes-no>` |
| Physical NICs | `NIC01`, `NIC02` | `<storage-nics>` |
| SET switch name | `vSwitch-SET-Converged` | `<set-switch-name>` |
| S2D disk type | NVMe, SSD, HDD, mixed | `<disk-type>` |
| Cache devices | NVMe, SSD, none | `<cache-devices>` |
| Capacity devices | SSD, HDD | `<capacity-devices>` |
| S2D pool name | `S2D on HV-S2D-CL01` | `<pool-name>` |
| CSV volume name | `CSV01-VMs` | `<csv-volume-name>` |
| CSV size | `1TB` | `<csv-size>` |
| CSV resiliency | Mirror, parity, nested mirror | `<resiliency>` |
| CSV path | `C:\ClusterStorage\CSV01-VMs` | `<csv-path>` |
| VM folder path | `C:\ClusterStorage\CSV01-VMs\VMs` | `<vm-folder>` |
| VHDX folder path | `C:\ClusterStorage\CSV01-VMs\VHDX` | `<vhdx-folder>` |
| Clustered VM name | `S2D-VM01` | `<clustered-vm-name>` |
| Evidence path | `C:\Admin\HyperV-S2D-CSV` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm all nodes are domain joined | Each node | `Get-ComputerInfo \| Select CsName,CsDomain,CsPartOfDomain` | Nodes are joined to same domain |
| 2 | Confirm Hyper-V role | Each node | `Get-WindowsFeature Hyper-V` | Hyper-V installed |
| 3 | Confirm Failover Clustering feature | Each node | `Get-WindowsFeature Failover-Clustering` | Feature state known |
| 4 | Confirm local disks are visible and healthy | Each node | `Get-PhysicalDisk; Get-Disk` | Eligible disks are visible |
| 5 | Confirm disks are not already pooled | Each node | `Get-PhysicalDisk \| Select FriendlyName,CanPool,HealthStatus` | Disks intended for S2D show usable state |
| 6 | Confirm NIC and storage network state | Each node | `Get-NetAdapter; Get-NetIPConfiguration` | Network baseline documented |
| 7 | Confirm RDMA if planned | Each node | `Get-NetAdapterRdma; Get-SmbClientNetworkInterface` | RDMA state known |
| 8 | Install required features | Each node | `Install-WindowsFeature Hyper-V,Failover-Clustering,FS-FileServer -IncludeManagementTools` | Required features installed |
| 9 | Run cluster validation with S2D tests | Admin node | `Test-Cluster -Node HV01,HV02 -Include 'Storage Spaces Direct','Inventory','Network','System Configuration'` | Validation report generated |
| 10 | Review validation report | Admin node | Open cluster validation report | Blocking errors fixed before proceeding |
| 11 | Create cluster without adding disks automatically | Admin node | `New-Cluster -Name '<cluster-name>' -Node HV01,HV02 -StaticAddress '<ip>' -NoStorage` | Cluster created |
| 12 | Configure quorum witness | Admin node | `Set-ClusterQuorum -FileShareWitness '<witness-path>'` | Witness configured |
| 13 | Confirm cluster health | Admin node | `Get-Cluster; Get-ClusterNode; Get-ClusterGroup` | Cluster online |
| 14 | Enable S2D | Admin node | `Enable-ClusterS2D -Confirm:$false` | S2D pool is created |
| 15 | Confirm S2D pool and disks | Admin node | `Get-StoragePool; Get-PhysicalDisk` | Storage pool and disks healthy |
| 16 | Create CSV volume | Admin node | `New-Volume -StoragePoolFriendlyName 'S2D*' -FriendlyName '<csv-name>' -FileSystem CSVFS_ReFS -Size '<size>' -ResiliencySettingName Mirror` | CSV volume created |
| 17 | Confirm CSV path | Admin node | `Get-ClusterSharedVolume` | CSV appears under `C:\ClusterStorage` |
| 18 | Configure cluster networks | Admin node | `Get-ClusterNetwork` | Management, storage, CSV, and live migration networks identified |
| 19 | Configure live migration | Each node | `Set-VMHost -VirtualMachineMigrationEnabled $true` | Live migration enabled |
| 20 | Create VM folders on CSV | Owner node | `New-Item -Path 'C:\ClusterStorage\<csv>\VMs','C:\ClusterStorage\<csv>\VHDX' -ItemType Directory -Force` | VM storage folders exist |
| 21 | Create VM on CSV | Owner node | `New-VM -Name '<vm-name>' -Path '<csv-vm-path>' -NewVHDPath '<csv-vhdx-path>'` | VM created on CSV |
| 22 | Make VM highly available | Owner node | `Add-ClusterVirtualMachineRole -VMName '<vm-name>'` | VM is clustered role |
| 23 | Test live migration of clustered VM | Admin node | `Move-ClusterVirtualMachineRole -Name '<vm-name>' -Node '<other-node>'` | VM moves successfully |
| 24 | Capture health and performance evidence | Admin node | Run health and post-change skeletons | S2D, CSV, and VM state documented |

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_PreChange_Capture_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_PreChange_Capture_Skeleton
# Purpose:
# Capture node, cluster, network, disk, storage, RDMA, Hyper-V, and event state before S2D and CSV work.

$EvidenceRoot = "C:\Admin\HyperV-S2D-CSV"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HV-S2D-CL01"
$Nodes = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "PreChange-$ClusterName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing admin context..." -ForegroundColor Cyan

whoami |
    Out-File -FilePath (Join-Path $OutputPath "00-AdminContext.txt") -Encoding UTF8

Write-Host "Capturing node readiness state..." -ForegroundColor Cyan

foreach ($Node in $Nodes) {
    $NodePath = Join-Path $OutputPath $Node
    New-Item -Path $NodePath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing from node: $Node" -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        "ComputerInfo"
        Get-ComputerInfo |
            Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
            Format-List

        "Features"
        Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, Failover-Clustering, RSAT-Clustering-PowerShell, FS-FileServer, Data-Center-Bridging |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize

        "Hyper-V Host"
        Get-VMHost -ErrorAction SilentlyContinue |
            Select-Object ComputerName, VirtualMachinePath, VirtualHardDiskPath, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption |
            Format-List

        "Physical Disks"
        Get-PhysicalDisk |
            Select-Object FriendlyName, SerialNumber, MediaType, BusType, CanPool, OperationalStatus, HealthStatus, Size, Usage |
            Format-Table -AutoSize

        "Disks"
        Get-Disk |
            Sort-Object Number |
            Select-Object Number, FriendlyName, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size, IsOffline, IsReadOnly |
            Format-Table -AutoSize

        "Storage Pools"
        Get-StoragePool -ErrorAction SilentlyContinue |
            Select-Object FriendlyName, HealthStatus, OperationalStatus, IsPrimordial, Size, AllocatedSize |
            Format-Table -AutoSize

        "Volumes"
        Get-Volume |
            Sort-Object DriveLetter |
            Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
            Format-Table -AutoSize

        "Network Adapters"
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

        "SMB Client Network Interfaces"
        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        "Virtual Switches"
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
            Format-Table -AutoSize

        "VM Inventory"
        Get-VM -ErrorAction SilentlyContinue |
            Sort-Object Name |
            Select-Object Name, State, Generation, ProcessorCount, MemoryAssigned, Path |
            Format-Table -AutoSize
    } |
    Out-File -FilePath (Join-Path $NodePath "Node-PreChange-State.txt") -Encoding UTF8
}

Write-Host "Checking existing cluster state if any..." -ForegroundColor Cyan

try {
    Get-Cluster -Name $ClusterName -ErrorAction Stop |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "ExistingCluster-State.txt")
}
catch {
    "Cluster not found or not reachable before change: $ClusterName" |
        Out-File -FilePath (Join-Path $OutputPath "ExistingCluster-State.txt") -Encoding UTF8
}

Write-Host "Testing node reachability..." -ForegroundColor Cyan

foreach ($Node in $Nodes) {
    Test-NetConnection $Node |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "Test-NetConnection-$Node.txt") -Encoding UTF8
}

Write-Host "Pre-change S2D and CSV evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Install_Features_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Install_Features_Skeleton
# Purpose:
# Install Hyper-V, Failover Clustering, File Server, and optional DCB tools on each S2D node.

$Nodes = @("HV01", "HV02")
$InstallDCB = $true

foreach ($Node in $Nodes) {
    Write-Host "Installing required features on $Node..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($InstallDCB)

        $FeatureList = @(
            "Hyper-V",
            "Hyper-V-PowerShell",
            "Failover-Clustering",
            "RSAT-Clustering-PowerShell",
            "RSAT-Clustering-Mgmt",
            "FS-FileServer"
        )

        if ($InstallDCB) {
            $FeatureList += "Data-Center-Bridging"
        }

        Install-WindowsFeature `
            -Name $FeatureList `
            -IncludeManagementTools

        Get-WindowsFeature $FeatureList |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize
    } -ArgumentList $InstallDCB
}

Write-Host "Feature installation completed. Reboot nodes if required." -ForegroundColor Green

foreach ($Node in $Nodes) {
    Invoke-Command -ComputerName $Node -ScriptBlock {
        Get-ComputerInfo |
            Select-Object CsName, CsDomain, WindowsProductName, OsBuildNumber |
            Format-List
    }
}
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Cluster_Validation_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Cluster_Validation_Skeleton
# Purpose:
# Run failover cluster validation with S2D-focused tests before creating or enabling S2D.

$ClusterName = "HV-S2D-CL01"
$Nodes = @("HV01", "HV02")
$ReportRoot = "C:\Admin\HyperV-S2D-CSV\Validation"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ReportName = "$ClusterName-S2D-Validation-$Timestamp"

New-Item -Path $ReportRoot -ItemType Directory -Force | Out-Null

Write-Host "Running cluster validation for nodes: $($Nodes -join ', ')" -ForegroundColor Cyan

Test-Cluster `
    -Node $Nodes `
    -Include "Storage Spaces Direct", "Inventory", "Network", "System Configuration" `
    -ReportName $ReportName

Write-Host "Validation completed. Review the generated report before continuing." -ForegroundColor Green

Write-Host "Recent validation reports:" -ForegroundColor Cyan

Get-ChildItem -Path "C:\Windows\Cluster\Reports" -Filter "*.htm" -ErrorAction SilentlyContinue |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 10 FullName, LastWriteTime, Length |
    Format-Table -AutoSize
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_Cluster_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_Cluster_Skeleton
# Purpose:
# Create the failover cluster for Storage Spaces Direct.
# Use -NoStorage so local disks are not automatically added as traditional cluster disks.

$ClusterName = "HV-S2D-CL01"
$ClusterIPAddress = "192.168.10.70"
$Nodes = @("HV01", "HV02")
$WitnessPath = "\\FS01\HV-S2D-CL01-Witness$"

Write-Host "Checking whether cluster already exists..." -ForegroundColor Cyan

if (Get-Cluster -Name $ClusterName -ErrorAction SilentlyContinue) {
    throw "Cluster already exists: $ClusterName"
}

Write-Host "Creating S2D failover cluster with -NoStorage..." -ForegroundColor Cyan

New-Cluster `
    -Name $ClusterName `
    -Node $Nodes `
    -StaticAddress $ClusterIPAddress `
    -NoStorage

Write-Host "Waiting for cluster service stabilization..." -ForegroundColor Cyan

Start-Sleep -Seconds 20

Write-Host "Configuring file share witness if path exists..." -ForegroundColor Cyan

if (Test-Path $WitnessPath) {
    Set-ClusterQuorum `
        -Cluster $ClusterName `
        -FileShareWitness $WitnessPath
}
else {
    Write-Warning "Witness path not reachable: $WitnessPath"
    Write-Warning "Configure quorum witness before production use."
}

Write-Host "Cluster state after creation:" -ForegroundColor Green

Get-Cluster -Name $ClusterName |
    Format-List *

Get-ClusterNode -Cluster $ClusterName |
    Select-Object Name, State, NodeWeight, FaultDomain |
    Format-Table -AutoSize

Get-ClusterQuorum -Cluster $ClusterName |
    Format-List

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Enable_S2D_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Enable_S2D_Skeleton
# Purpose:
# Enable Storage Spaces Direct on the cluster.
# Warning:
# This is a storage-changing operation. Validate disks and backups before running.

$ClusterName = "HV-S2D-CL01"
$SkipEligibilityChecks = $false

Write-Host "Checking cluster state before enabling S2D..." -ForegroundColor Cyan

Get-Cluster -Name $ClusterName |
    Format-List Name, Domain, State

Get-ClusterNode -Cluster $ClusterName |
    Format-Table Name, State, NodeWeight -AutoSize

Write-Host "Checking physical disks before S2D..." -ForegroundColor Cyan

Invoke-Command -ComputerName (Get-ClusterNode -Cluster $ClusterName | Select-Object -ExpandProperty Name) -ScriptBlock {
    "Node: $env:COMPUTERNAME"

    Get-PhysicalDisk |
        Select-Object FriendlyName, SerialNumber, MediaType, BusType, CanPool, OperationalStatus, HealthStatus, Size, Usage |
        Format-Table -AutoSize
}

Write-Warning "Enable-ClusterS2D will claim eligible local disks for the S2D storage pool."

if ($SkipEligibilityChecks) {
    Write-Host "Enabling S2D with SkipEligibilityChecks..." -ForegroundColor Yellow

    Enable-ClusterS2D `
        -CimSession $ClusterName `
        -SkipEligibilityChecks `
        -Confirm:$false
}
else {
    Write-Host "Enabling S2D..." -ForegroundColor Cyan

    Enable-ClusterS2D `
        -CimSession $ClusterName `
        -Confirm:$false
}

Write-Host "S2D state after enablement:" -ForegroundColor Green

Get-ClusterS2D -CimSession $ClusterName |
    Format-List *

Write-Host "Storage pools:" -ForegroundColor Cyan

Get-StoragePool -CimSession $ClusterName |
    Select-Object FriendlyName, HealthStatus, OperationalStatus, IsPrimordial, Size, AllocatedSize |
    Format-Table -AutoSize

Write-Host "Physical disks after S2D enablement:" -ForegroundColor Cyan

Get-PhysicalDisk -CimSession $ClusterName |
    Select-Object FriendlyName, SerialNumber, MediaType, BusType, CanPool, OperationalStatus, HealthStatus, Size, Usage |
    Format-Table -AutoSize
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_CSV_Volumes_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_CSV_Volumes_Skeleton
# Purpose:
# Create S2D-backed CSV volumes for Hyper-V VM storage.

$ClusterName = "HV-S2D-CL01"
$CsvVolumeName = "CSV01-VMs"
$CsvSize = 1TB

# Common values:
# Mirror
# Parity
# MirrorAcceleratedParity
$ResiliencySettingName = "Mirror"

Write-Host "Checking S2D storage pool..." -ForegroundColor Cyan

$StoragePool = Get-StoragePool -CimSession $ClusterName |
    Where-Object { $_.FriendlyName -like "S2D*" -and $_.IsPrimordial -eq $false } |
    Select-Object -First 1

if (-not $StoragePool) {
    throw "S2D storage pool not found."
}

$StoragePool |
    Select-Object FriendlyName, HealthStatus, OperationalStatus, Size, AllocatedSize |
    Format-List

Write-Host "Checking supported resiliency settings..." -ForegroundColor Cyan

Get-ResiliencySetting -StoragePool $StoragePool |
    Select-Object Name, NumberOfDataCopiesDefault, PhysicalDiskRedundancyDefault |
    Format-Table -AutoSize

Write-Host "Creating S2D CSV volume..." -ForegroundColor Cyan

New-Volume `
    -CimSession $ClusterName `
    -StoragePoolFriendlyName $StoragePool.FriendlyName `
    -FriendlyName $CsvVolumeName `
    -FileSystem CSVFS_ReFS `
    -Size $CsvSize `
    -ResiliencySettingName $ResiliencySettingName

Write-Host "Cluster Shared Volumes after creation:" -ForegroundColor Green

Get-ClusterSharedVolume -Cluster $ClusterName |
    Format-List *

Write-Host "Cluster storage resources:" -ForegroundColor Cyan

Get-ClusterResource -Cluster $ClusterName |
    Where-Object { $_.ResourceType -match "Physical Disk|Storage Pool|Virtual Machine" -or $_.Name -like "*$CsvVolumeName*" } |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Format-Table -AutoSize

Write-Host "CSV paths on local node:" -ForegroundColor Cyan

Get-ChildItem "C:\ClusterStorage" -ErrorAction SilentlyContinue |
    Select-Object FullName, CreationTime, LastWriteTime |
    Format-Table -AutoSize
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Configure_Cluster_Networks_And_Live_Migration_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Configure_Cluster_Networks_And_Live_Migration_Skeleton
# Purpose:
# Configure cluster network roles and Hyper-V live migration settings after S2D and CSV are online.

$ClusterName = "HV-S2D-CL01"
$Nodes = @("HV01", "HV02")

$ManagementNetworkName = "Cluster Network 1"
$StorageNetwork1Name = "Cluster Network 2"
$StorageNetwork2Name = "Cluster Network 3"
$LiveMigrationNetworkName = "Cluster Network 4"

Write-Host "Current cluster networks:" -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize

Write-Host "Renaming and setting cluster network roles where names match planned values..." -ForegroundColor Cyan

$Mgmt = Get-ClusterNetwork -Cluster $ClusterName -Name $ManagementNetworkName -ErrorAction SilentlyContinue
$Storage1 = Get-ClusterNetwork -Cluster $ClusterName -Name $StorageNetwork1Name -ErrorAction SilentlyContinue
$Storage2 = Get-ClusterNetwork -Cluster $ClusterName -Name $StorageNetwork2Name -ErrorAction SilentlyContinue
$LM = Get-ClusterNetwork -Cluster $ClusterName -Name $LiveMigrationNetworkName -ErrorAction SilentlyContinue

# Role values:
# 0 = None
# 1 = Cluster only
# 3 = Cluster and Client
if ($Mgmt) {
    $Mgmt.Name = "Management"
    $Mgmt.Role = 3
}

if ($Storage1) {
    $Storage1.Name = "Storage-CSV-01"
    $Storage1.Role = 1
}

if ($Storage2) {
    $Storage2.Name = "Storage-CSV-02"
    $Storage2.Role = 1
}

if ($LM) {
    $LM.Name = "LiveMigration"
    $LM.Role = 1
}

Write-Host "Cluster networks after configuration:" -ForegroundColor Green

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize

Write-Host "Configuring Hyper-V live migration on nodes..." -ForegroundColor Cyan

foreach ($Node in $Nodes) {
    Invoke-Command -ComputerName $Node -ScriptBlock {
        Set-VMHost `
            -VirtualMachineMigrationEnabled $true `
            -VirtualMachineMigrationAuthenticationType Kerberos `
            -VirtualMachineMigrationPerformanceOption SMB `
            -MaximumVirtualMachineMigrations 2 `
            -MaximumStorageMigrations 2

        Get-VMHost |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption, MaximumVirtualMachineMigrations, MaximumStorageMigrations |
            Format-List
    }
}

Write-Host "Checking SMB Direct and multichannel state..." -ForegroundColor Cyan

foreach ($Node in $Nodes) {
    Invoke-Command -ComputerName $Node -ScriptBlock {
        "Node: $env:COMPUTERNAME"

        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRSSCapable, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize
    }
}
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_Clustered_VM_On_CSV_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Create_Clustered_VM_On_CSV_Skeleton
# Purpose:
# Create a VM on S2D-backed CSV storage and make it highly available.

$ClusterName = "HV-S2D-CL01"
$VMName = "S2D-VM01"
$SwitchName = "vSwitch-External"

$CsvRoot = "C:\ClusterStorage\CSV01-VMs"
$VMRoot = Join-Path $CsvRoot "VMs"
$VHDXRoot = Join-Path $CsvRoot "VHDX"

$VMPath = Join-Path $VMRoot $VMName
$VHDXPath = Join-Path $VHDXRoot "$VMName.vhdx"

$Generation = 2
$StartupMemory = 4GB
$VHDXSize = 80GB
$ProcessorCount = 2

Write-Host "Validating CSV path..." -ForegroundColor Cyan

if (-not (Test-Path $CsvRoot)) {
    throw "CSV path not found: $CsvRoot"
}

New-Item -Path $VMRoot -ItemType Directory -Force | Out-Null
New-Item -Path $VHDXRoot -ItemType Directory -Force | Out-Null

Write-Host "Validating vSwitch..." -ForegroundColor Cyan

Get-VMSwitch -Name $SwitchName -ErrorAction Stop | Out-Null

Write-Host "Checking whether VM already exists..." -ForegroundColor Cyan

if (Get-VM -Name $VMName -ErrorAction SilentlyContinue) {
    throw "VM already exists on this node: $VMName"
}

Write-Host "Creating VM on CSV storage..." -ForegroundColor Cyan

New-VM `
    -Name $VMName `
    -Generation $Generation `
    -MemoryStartupBytes $StartupMemory `
    -Path $VMPath `
    -NewVHDPath $VHDXPath `
    -NewVHDSizeBytes $VHDXSize `
    -SwitchName $SwitchName

Write-Host "Configuring VM baseline..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $VMName `
    -Count $ProcessorCount

Set-VM `
    -Name $VMName `
    -CheckpointType Production `
    -AutomaticCheckpointsEnabled $false `
    -AutomaticStartAction Nothing `
    -AutomaticStopAction ShutDown

if ($Generation -eq 2) {
    $BootDisk = Get-VMHardDiskDrive -VMName $VMName | Select-Object -First 1

    Set-VMFirmware `
        -VMName $VMName `
        -EnableSecureBoot On `
        -SecureBootTemplate MicrosoftWindows `
        -FirstBootDevice $BootDisk
}

Write-Host "Adding VM as clustered role..." -ForegroundColor Cyan

Add-ClusterVirtualMachineRole `
    -Cluster $ClusterName `
    -VMName $VMName

Write-Host "Starting clustered VM role..." -ForegroundColor Cyan

Start-ClusterGroup `
    -Cluster $ClusterName `
    -Name $VMName

Write-Host "Clustered VM state:" -ForegroundColor Green

Get-ClusterGroup -Cluster $ClusterName -Name $VMName |
    Select-Object Name, OwnerNode, State, GroupType |
    Format-List

Get-ClusterResource -Cluster $ClusterName |
    Where-Object { $_.OwnerGroup -eq $VMName } |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Format-Table -AutoSize
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Health_And_Performance_Verification_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Health_And_Performance_Verification_Skeleton
# Purpose:
# Validate S2D, cluster, CSV, physical disks, storage pool, virtual disks, SMB, and VM performance state.

$ClusterName = "HV-S2D-CL01"
$EvidenceRoot = "C:\Admin\HyperV-S2D-CSV"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "Health-$ClusterName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing cluster health..." -ForegroundColor Cyan

Get-Cluster -Name $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Cluster.txt")

Get-ClusterNode -Cluster $ClusterName |
    Select-Object Name, State, NodeWeight, FaultDomain |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-ClusterNodes.txt")

Get-ClusterGroup -Cluster $ClusterName |
    Select-Object Name, OwnerNode, State, GroupType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-ClusterGroups.txt")

Write-Host "Capturing S2D health..." -ForegroundColor Cyan

Get-ClusterS2D -CimSession $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "04-ClusterS2D.txt")

Get-StorageSubSystem -CimSession $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "05-StorageSubSystem.txt")

Get-StoragePool -CimSession $ClusterName |
    Select-Object FriendlyName, HealthStatus, OperationalStatus, IsPrimordial, Size, AllocatedSize |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-StoragePools.txt")

Get-PhysicalDisk -CimSession $ClusterName |
    Select-Object FriendlyName, SerialNumber, MediaType, BusType, Usage, OperationalStatus, HealthStatus, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-PhysicalDisks.txt")

Get-VirtualDisk -CimSession $ClusterName |
    Select-Object FriendlyName, ResiliencySettingName, HealthStatus, OperationalStatus, Size, FootprintOnPool |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-VirtualDisks.txt")

Write-Host "Capturing CSV state..." -ForegroundColor Cyan

Get-ClusterSharedVolume -Cluster $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "09-ClusterSharedVolumes.txt")

Write-Host "Capturing cluster networks..." -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "10-ClusterNetworks.txt")

Write-Host "Capturing SMB and RDMA state from nodes..." -ForegroundColor Cyan

$Nodes = Get-ClusterNode -Cluster $ClusterName | Select-Object -ExpandProperty Name

foreach ($Node in $Nodes) {
    Invoke-Command -ComputerName $Node -ScriptBlock {
        "Node: $env:COMPUTERNAME"

        "SMB Client Interfaces"
        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        "SMB Multichannel"
        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRSSCapable, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize

        "RDMA"
        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, Enabled, OperationalState, PFC, ETS -AutoSize

        "Quick Counters"
        Get-Counter -Counter "\Cluster CSV File System(*)\IO Reads/sec","\Cluster CSV File System(*)\IO Writes/sec","\SMB Direct Connection(*)\RDMA Inbound Bytes/sec","\SMB Direct Connection(*)\RDMA Outbound Bytes/sec" -SampleInterval 2 -MaxSamples 5 -ErrorAction SilentlyContinue
    } |
    Out-File -FilePath (Join-Path $OutputPath "11-Node-$Node-SMB-RDMA-Counters.txt") -Encoding UTF8
}

Write-Host "S2D health evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_PostChange_Verification_Skeleton

~~~powershell
# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_PostChange_Verification_Skeleton
# Purpose:
# Capture final S2D, CSV, cluster, Hyper-V, VM, disk, network, and event evidence after configuration.

$EvidenceRoot = "C:\Admin\HyperV-S2D-CSV"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HV-S2D-CL01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$ClusterName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing cluster summary..." -ForegroundColor Cyan

Get-Cluster -Name $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Cluster-Summary.txt")

Get-ClusterNode -Cluster $ClusterName |
    Select-Object Name, State, NodeWeight, FaultDomain |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-Cluster-Nodes.txt")

Get-ClusterQuorum -Cluster $ClusterName |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-Cluster-Quorum.txt")

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Cluster-Networks.txt")

Write-Host "Capturing S2D and storage state..." -ForegroundColor Cyan

Get-ClusterS2D -CimSession $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "05-ClusterS2D.txt")

Get-StoragePool -CimSession $ClusterName |
    Select-Object FriendlyName, HealthStatus, OperationalStatus, IsPrimordial, Size, AllocatedSize |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-StoragePools.txt")

Get-PhysicalDisk -CimSession $ClusterName |
    Select-Object FriendlyName, SerialNumber, MediaType, BusType, Usage, OperationalStatus, HealthStatus, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-PhysicalDisks.txt")

Get-VirtualDisk -CimSession $ClusterName |
    Select-Object FriendlyName, ResiliencySettingName, HealthStatus, OperationalStatus, Size, FootprintOnPool |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-VirtualDisks.txt")

Get-Volume -CimSession $ClusterName -ErrorAction SilentlyContinue |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "09-Volumes.txt")

Write-Host "Capturing CSV state..." -ForegroundColor Cyan

Get-ClusterSharedVolume -Cluster $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "10-ClusterSharedVolumes.txt")

Write-Host "Capturing clustered VM roles..." -ForegroundColor Cyan

Get-ClusterGroup -Cluster $ClusterName |
    Select-Object Name, OwnerNode, State, GroupType, Priority |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "11-ClusterGroups.txt")

Get-ClusterResource -Cluster $ClusterName |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Sort-Object OwnerGroup, Name |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "12-ClusterResources.txt")

Write-Host "Capturing VM state from cluster nodes..." -ForegroundColor Cyan

$Nodes = Get-ClusterNode -Cluster $ClusterName | Select-Object -ExpandProperty Name

foreach ($Node in $Nodes) {
    Invoke-Command -ComputerName $Node -ScriptBlock {
        "Node: $env:COMPUTERNAME"

        Get-VM -ErrorAction SilentlyContinue |
            Sort-Object Name |
            Select-Object Name, State, Status, Generation, Uptime, Path |
            Format-Table -AutoSize

        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
            Format-Table -AutoSize

        Get-VMHost -ErrorAction SilentlyContinue |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption |
            Format-List
    } |
    Out-File -FilePath (Join-Path $OutputPath "13-Node-$Node-HyperV-State.txt") -Encoding UTF8
}

Write-Host "Capturing event logs..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-FailoverClustering/Operational" -MaxEvents 150 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "14-FailoverClustering-Operational-Events.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "15-HyperV-VMMS-Events.txt") -Encoding UTF8

Get-WinEvent -LogName "System" -MaxEvents 150 -ErrorAction SilentlyContinue |
    Where-Object {
        $_.ProviderName -match "stor|disk|ntfs|refs|smb|csv|cluster|net|rdma"
    } |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "16-System-Storage-Network-Events.txt") -Encoding UTF8

Write-Host "Post-change S2D and CSV evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-WindowsFeature Hyper-V,Failover-Clustering,FS-FileServer` | Each node | Required feature state |
| `Get-PhysicalDisk` | Each node or cluster CIM session | Physical disk health, media type, usage, and pool eligibility |
| `Get-Disk` | Each node | Disk online, partition, and health state |
| `Get-StoragePool` | Admin node | S2D storage pool state |
| `Get-StorageSubSystem` | Admin node | Storage subsystem state |
| `Get-VirtualDisk` | Admin node | S2D virtual disk state |
| `Test-Cluster -Node HV01,HV02 -Include 'Storage Spaces Direct','Inventory','Network','System Configuration'` | Admin node | Cluster and S2D validation |
| `New-Cluster -Name '<cluster-name>' -Node '<nodes>' -StaticAddress '<ip>' -NoStorage` | Admin node | Creates failover cluster for S2D |
| `Get-Cluster` | Admin node | Cluster state |
| `Get-ClusterNode` | Admin node | Node membership and state |
| `Get-ClusterQuorum` | Admin node | Quorum and witness state |
| `Set-ClusterQuorum -FileShareWitness '<path>'` | Admin node | Configures file share witness |
| `Enable-ClusterS2D -Confirm:$false` | Admin node | Enables Storage Spaces Direct |
| `Get-ClusterS2D` | Admin node | S2D state and configuration |
| `New-Volume -StoragePoolFriendlyName 'S2D*' -FriendlyName '<name>' -FileSystem CSVFS_ReFS -Size '<size>' -ResiliencySettingName Mirror` | Admin node | Creates S2D CSV volume |
| `Get-ClusterSharedVolume` | Admin node | CSV state and path |
| `Get-ClusterNetwork` | Admin node | Cluster network role and metric state |
| `Get-SmbClientNetworkInterface` | Each node | SMB Direct and RDMA-capable network interfaces |
| `Get-SmbMultichannelConnection` | Each node | SMB multichannel and RDMA session state |
| `Get-NetAdapterRdma` | Each node | RDMA enabled and operational state |
| `Get-VMHost \| Select *Migration*` | Each node | Live migration host settings |
| `Set-VMHost -VirtualMachineMigrationEnabled $true -VirtualMachineMigrationPerformanceOption SMB` | Each node | Enables SMB-based live migration |
| `New-VM -Name '<vm-name>' -Path 'C:\ClusterStorage\<csv>\VMs'` | Owner node | Creates VM on CSV |
| `Add-ClusterVirtualMachineRole -VMName '<vm-name>'` | Owner node | Makes VM highly available |
| `Move-ClusterVirtualMachineRole -Name '<vm-name>' -Node '<node>'` | Admin node | Tests clustered VM movement |
| `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 20` | Admin node | Cluster events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Admin node | Hyper-V events |

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current failure state before rollback | Admin node | Run post-change verification skeleton | Current state documented |
| 2 | Stop test clustered VMs | Admin node | `Stop-ClusterGroup -Name '<vm-name>'` | VM role is offline |
| 3 | Remove test clustered VM role if required | Admin node | `Remove-ClusterGroup -Name '<vm-name>' -RemoveResources` | Cluster role removed |
| 4 | Remove test VM registration if required | Owner node | `Remove-VM -Name '<vm-name>' -Force` | VM registration removed |
| 5 | Delete test VM files only after path verification | Owner node | `Remove-Item 'C:\ClusterStorage\<csv>\VMs\<vm-name>' -Recurse -Force` | Test VM files removed |
| 6 | Remove S2D test volume if approved | Admin node | `Remove-VirtualDisk -FriendlyName '<csv-volume-name>'` | Virtual disk removed |
| 7 | Disable S2D only in lab or rebuild scenario | Admin node | `Disable-ClusterS2D -Confirm:$false` | S2D disabled |
| 8 | Clear storage pool only if full rebuild approved | Admin node | Remove virtual disks and pool after validation | Storage reset for rebuild |
| 9 | Remove cluster only if full rollback approved | Admin node | `Remove-Cluster -Cluster '<cluster-name>' -Force -CleanupAD` | Cluster removed |
| 10 | Remove cluster feature only if lab cleanup requires it | Each node | `Uninstall-WindowsFeature Failover-Clustering` | Feature removed |
| 11 | Restore VMHost migration settings from baseline | Each node | `Set-VMHost -VirtualMachineMigrationEnabled '<old-value>'` | Migration settings restored |
| 12 | Restore cluster network naming and roles from baseline | Admin node | Set `Get-ClusterNetwork` values manually | Network settings restored |
| 13 | Capture rollback evidence | Admin node | Run post-change verification skeleton | Rollback state documented |

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Test-Cluster` fails S2D validation | Unsupported hardware, driver, firmware, disk, or network config | Fix validation failures before enabling S2D |
| Disks do not show `CanPool = True` | Disks already partitioned, pooled, reserved, or system disks | Clean only approved data disks and rerun discovery |
| `Enable-ClusterS2D` fails | Cluster not healthy, validation failed, or disks not eligible | Fix cluster, disk, and validation issues |
| S2D pool is not created | No eligible disks or S2D enable failed | Confirm eligible local disks on all nodes |
| Physical disk shows unhealthy | Failed disk, firmware issue, or connectivity issue | Replace disk or resolve hardware/firmware problem |
| Storage pool degraded | Disk or node health issue | Review `Get-PhysicalDisk`, `Get-StoragePool`, and cluster events |
| CSV does not appear | Volume creation failed or cluster resource offline | Check virtual disk, volume, and cluster resource state |
| CSV path missing under `C:\ClusterStorage` | CSV offline or mounted on another state incorrectly | Review `Get-ClusterSharedVolume` and cluster events |
| CSV enters redirected mode | Storage network or CSV communication issue | Fix cluster networks, SMB, RDMA, and storage path |
| `New-Volume` fails | Not enough capacity, wrong resiliency, or pool unhealthy | Review pool capacity and supported resiliency settings |
| ReFS or CSVFS_ReFS format fails | Unsupported parameter or OS mismatch | Validate OS support and use supported file system option |
| Cluster creation fails | CNO permissions, DNS, IP conflict, or node issue | Fix AD permissions, DNS, IP, and node connectivity |
| Witness cannot be configured | Share missing or permissions incorrect | Create witness share and grant cluster computer object rights |
| Live migration fails | Network, auth, switch, CPU, or storage issue | Review migration settings, cluster networks, and VMMS events |
| SMB Direct not used | RDMA unavailable or SMB multichannel path not selected | Validate RDMA state, SMB interfaces, and network path |
| RDMA operational state is false | Driver, firmware, DCB, or switch configuration issue | Fix NIC driver/firmware and physical network configuration |
| VM cannot start on another node | vSwitch missing or CSV inaccessible | Standardize vSwitch names and validate CSV access |
| S2D performance poor | Cache not configured, RDMA issue, disk bottleneck, or network contention | Review counters, RDMA, disk media, cache, and cluster networks |
| Clustered VM role fails | VM config, storage, or resource issue | Review cluster resource dependencies and Hyper-V events |
| Cleanup risks data loss | S2D operations are destructive if wrong disks are selected | Stop and validate disk identity before removing pools or volumes |

# 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV_Related_Labs

| Lab                                                                           | Relationship                                                          |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| 00_Hyper-V_Index.md                                                           | Places this S2D and CSV task in the full Hyper-V suite                |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Confirms node CPU, memory, disk, network, and domain readiness        |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Required before Hyper-V and Failover Clustering work                  |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md                       | Host paths and defaults affect VM placement                           |
| 04_Create_And_Configure_Virtual_Switches.md                                   | Cluster nodes require consistent vSwitch design                       |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md                         | VM resource sizing affects clustered workload behavior                |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md                             | VHDX placement and health are central to CSV-backed VMs               |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md                   | VM network settings must work consistently across nodes               |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md                              | Backups are required before S2D, CSV, or clustered VM changes         |
| 14_Configure_Live_Migration_And_Storage_Migration.md                          | Live migration is used to move clustered VMs between S2D nodes        |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md                 | S2D depends on failover clustering and clustered VM roles             |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md         | Monitoring validates S2D, CSV, RDMA, SMB, and VM performance          |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Troubleshooting uses cluster, CSV, storage, and VMMS evidence         |
| 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance.md              | RDMA, SET, and SMB Direct tuning improve S2D and CSV traffic          |
| 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images.md                | Golden images and templates can be stored and deployed on CSV volumes |
