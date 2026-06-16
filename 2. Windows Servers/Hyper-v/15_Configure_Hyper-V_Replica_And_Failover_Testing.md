# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Index
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Source_Basis
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Mental_Model
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Planning_Table
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Configuration_Checklist
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_PreChange_Capture_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Install_Clustering_Feature_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Cluster_Validation_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Create_Cluster_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Configure_CSV_And_Cluster_Networks_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Create_Clustered_VM_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Live_Migration_And_Failover_Test_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_PostChange_Verification_Skeleton
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Verification_Commands
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Rollback
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Failure_Checks
16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Related_Labs

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Failover clustering overview | Supports Windows Server cluster concepts, nodes, cluster name object, quorum, roles, and failover |
| Microsoft Learn | Hyper-V on failover clusters | Supports highly available virtual machines and clustered VM roles |
| Microsoft Learn | Failover cluster validation | Supports pre-cluster validation through `Test-Cluster` |
| Microsoft Learn | Cluster Shared Volumes | Supports shared VM storage under `C:\ClusterStorage` |
| Microsoft Learn | FailoverClusters PowerShell module | Supports `New-Cluster`, `Get-Cluster`, `Add-ClusterSharedVolume`, and clustered role operations |
| Microsoft Learn | Hyper-V live migration in clustered environments | Supports moving clustered VMs between nodes |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Failover cluster | Group of Windows Server nodes that can run clustered workloads with failover capability |
| Cluster node | Windows Server host that participates in the cluster |
| Cluster name object | AD computer object representing the cluster name |
| Quorum | Voting model that determines whether the cluster can stay online |
| Witness | File share, disk, or cloud witness that helps maintain quorum |
| Cluster role | Highly available workload managed by the cluster |
| Clustered VM | Hyper-V VM registered as a highly available cluster role |
| CSV | Cluster Shared Volume used by multiple cluster nodes for VM storage |
| Live migration | Planned movement of a running clustered VM between nodes |
| Failover | Movement of a clustered role after planned maintenance or node failure |
| Preferred owner | Node preference list for where a clustered VM should run |
| Possible owner | Nodes allowed to run a clustered VM |
| Anti-affinity | Cluster rule to keep selected workloads apart where possible |
| Cluster network | Network detected by the cluster for management, client, cluster, CSV, or live migration traffic |
| Validation report | Required health check before cluster creation or production support |
| Rollback boundary | This workbook builds the cluster and clustered VM workflow, not Storage Spaces Direct or Azure Site Recovery |

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Cluster name | `HVCL01` | `<cluster-name>` |
| Cluster static IP | `192.168.10.60` | `<cluster-ip>` |
| Cluster domain | `corp.contoso.com` | `<domain-name>` |
| Node 1 | `HV01` | `<node-1>` |
| Node 2 | `HV02` | `<node-2>` |
| Node 3 optional | `HV03` | `<node-3>` |
| Cluster OU | `OU=Servers,OU=Infrastructure,DC=corp,DC=contoso,DC=com` | `<cluster-ou>` |
| Witness type | File share, disk, cloud | `<witness-type>` |
| File share witness path | `\\FS01\HVCL01-Witness$` | `<witness-path>` |
| Shared storage type | iSCSI, FC, SAS, CSV, lab shared disk | `<shared-storage-type>` |
| CSV volume name | `Cluster Disk 1` | `<csv-disk-name>` |
| CSV path | `C:\ClusterStorage\Volume1` | `<csv-path>` |
| Management network | `192.168.10.0/24` | `<management-network>` |
| Live migration network | `192.168.50.0/24` | `<live-migration-network>` |
| CSV or cluster network | `192.168.60.0/24` | `<csv-network>` |
| VM switch name on every node | `vSwitch-External` | `<switch-name>` |
| Clustered VM name | `CL-VM01` | `<clustered-vm-name>` |
| VM config path | `C:\ClusterStorage\Volume1\VMs` | `<clustered-vm-path>` |
| VHDX path | `C:\ClusterStorage\Volume1\VHDX` | `<clustered-vhdx-path>` |
| Validation report path | `C:\Admin\HyperV-Cluster\Validation` | `<validation-path>` |
| Evidence path | `C:\Admin\HyperV-Cluster` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm all nodes are domain joined | Each node | `Get-ComputerInfo \| Select CsName,CsDomain,CsPartOfDomain` | Each node is joined to the same domain |
| 2 | Confirm Hyper-V role on each node | Each node | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 3 | Confirm matching virtual switch names | Each node | `Get-VMSwitch` | Required switch name exists on every node |
| 4 | Confirm Failover Clustering feature state | Each node | `Get-WindowsFeature Failover-Clustering` | Feature state is known |
| 5 | Install Failover Clustering feature and tools | Each node | `Install-WindowsFeature Failover-Clustering -IncludeManagementTools` | Feature and tools are installed |
| 6 | Confirm FailoverClusters module | Each node | `Get-Module -ListAvailable FailoverClusters` | Module is available |
| 7 | Confirm management network connectivity | Admin node | `Test-NetConnection <node-name>` | Nodes are reachable |
| 8 | Confirm live migration network connectivity | Each node | `Test-NetConnection <peer-live-migration-ip>` | Migration network works |
| 9 | Confirm shared storage visibility | Each node | `Get-Disk` | Shared disks are visible where applicable |
| 10 | Run full cluster validation | Admin node | `Test-Cluster -Node HV01,HV02` | Validation report completes |
| 11 | Review validation report | Admin node | Open generated validation report | Blocking failures are corrected before cluster creation |
| 12 | Create failover cluster | Admin node | `New-Cluster -Name HVCL01 -Node HV01,HV02 -StaticAddress 192.168.10.60` | Cluster is created |
| 13 | Confirm cluster state | Admin node | `Get-Cluster; Get-ClusterNode` | Cluster and nodes are online |
| 14 | Configure quorum witness | Admin node | `Set-ClusterQuorum -FileShareWitness '\\FS01\HVCL01-Witness$'` | Cluster quorum uses planned witness |
| 15 | Add available shared disks | Admin node | `Get-ClusterAvailableDisk \| Add-ClusterDisk` | Shared disks become cluster disks |
| 16 | Convert cluster disk to CSV | Admin node | `Add-ClusterSharedVolume -Name 'Cluster Disk 1'` | Disk appears under `C:\ClusterStorage` |
| 17 | Configure cluster network roles | Admin node | `Get-ClusterNetwork` | Management, cluster, and migration networks are identified |
| 18 | Configure live migration settings | Admin node | `Set-VMHost -VirtualMachineMigrationEnabled $true` | Live migration is enabled on nodes |
| 19 | Create VM on CSV storage | Owner node | `New-VM -Name '<vm-name>' -Path 'C:\ClusterStorage\Volume1\VMs' -NewVHDPath 'C:\ClusterStorage\Volume1\VHDX\<vm-name>.vhdx'` | VM files are placed on CSV |
| 20 | Make VM highly available | Owner node | `Add-ClusterVirtualMachineRole -VMName '<vm-name>'` | VM becomes clustered role |
| 21 | Confirm clustered VM role | Admin node | `Get-ClusterGroup -Name '<vm-name>'` | Clustered VM role exists |
| 22 | Test live migration | Admin node | `Move-ClusterVirtualMachineRole -Name '<vm-name>' -Node '<destination-node>'` | VM moves to another node |
| 23 | Test controlled failover | Admin node | `Move-ClusterGroup -Name '<vm-name>' -Node '<destination-node>'` | Cluster role fails over cleanly |
| 24 | Capture post-change evidence | Admin node | Run post-change verification skeleton | Cluster and clustered VM state are documented |

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_PreChange_Capture_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_PreChange_Capture_Skeleton
# Purpose:
# Capture node, Hyper-V, network, storage, cluster feature, and validation readiness before cluster work.

$EvidenceRoot = "C:\Admin\HyperV-Cluster"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$Nodes = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "PreChange-$ClusterName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing local admin context..." -ForegroundColor Cyan

whoami | Out-File -FilePath (Join-Path $OutputPath "00-AdminContext.txt") -Encoding UTF8

Write-Host "Capturing node readiness..." -ForegroundColor Cyan

foreach ($Node in $Nodes) {
    $NodePath = Join-Path $OutputPath $Node
    New-Item -Path $NodePath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing readiness from $Node..." -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        "ComputerInfo"
        Get-ComputerInfo |
            Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
            Format-List

        "Installed Hyper-V and clustering features"
        Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, Failover-Clustering, RSAT-Clustering-PowerShell |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize

        "Hyper-V host settings"
        Get-VMHost |
            Select-Object ComputerName, VirtualMachinePath, VirtualHardDiskPath, NumaSpanningEnabled, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption |
            Format-List

        "Virtual switches"
        Get-VMSwitch |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
            Format-Table -AutoSize

        "Network adapters"
        Get-NetAdapter |
            Sort-Object Name |
            Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription |
            Format-Table -AutoSize

        "IP configuration"
        Get-NetIPConfiguration |
            Format-List

        "Disks"
        Get-Disk |
            Sort-Object Number |
            Select-Object Number, FriendlyName, BusType, PartitionStyle, OperationalStatus, HealthStatus, Size, IsOffline, IsReadOnly |
            Format-Table -AutoSize

        "Volumes"
        Get-Volume |
            Sort-Object DriveLetter |
            Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
            Format-Table -AutoSize
    } | Out-File -FilePath (Join-Path $NodePath "NodeReadiness.txt") -Encoding UTF8
}

Write-Host "Testing node reachability..." -ForegroundColor Cyan

foreach ($Node in $Nodes) {
    Test-NetConnection $Node |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "Test-NetConnection-$Node.txt") -Encoding UTF8
}

Write-Host "Checking whether cluster name already resolves..." -ForegroundColor Cyan

Resolve-DnsName $ClusterName -ErrorAction SilentlyContinue |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "ClusterName-DNS-Before.txt") -Encoding UTF8

Write-Host "Pre-change cluster evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Install_Clustering_Feature_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Install_Clustering_Feature_Skeleton
# Purpose:
# Install Failover Clustering feature and management tools on every Hyper-V node.

$Nodes = @("HV01", "HV02")

foreach ($Node in $Nodes) {
    Write-Host "Installing Failover Clustering on $Node..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        Install-WindowsFeature `
            -Name Failover-Clustering `
            -IncludeManagementTools

        Install-WindowsFeature `
            -Name RSAT-Clustering-PowerShell, RSAT-Clustering-Mgmt `
            -IncludeAllSubFeature

        Get-WindowsFeature Failover-Clustering, RSAT-Clustering-PowerShell, RSAT-Clustering-Mgmt |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize

        Get-Module -ListAvailable FailoverClusters |
            Select-Object Name, Version, Path |
            Format-Table -AutoSize
    }
}

Write-Host "Failover Clustering feature installation completed." -ForegroundColor Green
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Cluster_Validation_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Cluster_Validation_Skeleton
# Purpose:
# Run cluster validation before creating or changing a Hyper-V failover cluster.

$ClusterName = "HVCL01"
$Nodes = @("HV01", "HV02")
$ReportRoot = "C:\Admin\HyperV-Cluster\Validation"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ReportName = "$ClusterName-Validation-$Timestamp"

New-Item -Path $ReportRoot -ItemType Directory -Force | Out-Null

Write-Host "Running cluster validation for nodes: $($Nodes -join ', ')" -ForegroundColor Cyan

Test-Cluster `
    -Node $Nodes `
    -ReportName $ReportName

Write-Host "Cluster validation completed." -ForegroundColor Green
Write-Host "Review the validation report before creating the cluster." -ForegroundColor Yellow

Write-Host "Recent validation reports:" -ForegroundColor Cyan

Get-ChildItem -Path "C:\Windows\Cluster\Reports" -Filter "*.htm" -ErrorAction SilentlyContinue |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 10 FullName, LastWriteTime, Length |
    Format-Table -AutoSize
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Create_Cluster_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Create_Cluster_Skeleton
# Purpose:
# Create a new failover cluster for Hyper-V nodes.

$ClusterName = "HVCL01"
$ClusterIPAddress = "192.168.10.60"
$Nodes = @("HV01", "HV02")
$UseNoStorage = $false

Write-Host "Validating that cluster does not already exist..." -ForegroundColor Cyan

$ExistingCluster = Get-Cluster -Name $ClusterName -ErrorAction SilentlyContinue

if ($ExistingCluster) {
    throw "Cluster already exists: $ClusterName"
}

Write-Host "Testing cluster name DNS resolution before creation..." -ForegroundColor Cyan

Resolve-DnsName $ClusterName -ErrorAction SilentlyContinue

Write-Host "Creating failover cluster: $ClusterName" -ForegroundColor Cyan

if ($UseNoStorage) {
    New-Cluster `
        -Name $ClusterName `
        -Node $Nodes `
        -StaticAddress $ClusterIPAddress `
        -NoStorage
}
else {
    New-Cluster `
        -Name $ClusterName `
        -Node $Nodes `
        -StaticAddress $ClusterIPAddress
}

Write-Host "Cluster state after creation:" -ForegroundColor Green

Get-Cluster -Name $ClusterName |
    Format-List *

Write-Host "Cluster nodes:" -ForegroundColor Cyan

Get-ClusterNode -Cluster $ClusterName |
    Format-Table Name, State, NodeWeight, FaultDomain -AutoSize

Write-Host "Cluster groups:" -ForegroundColor Cyan

Get-ClusterGroup -Cluster $ClusterName |
    Format-Table Name, OwnerNode, State, GroupType -AutoSize

Write-Host "Cluster networks:" -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName |
    Format-Table Name, Address, AddressMask, Role, Metric, AutoMetric -AutoSize
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Configure_CSV_And_Cluster_Networks_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Configure_CSV_And_Cluster_Networks_Skeleton
# Purpose:
# Add available shared storage to the cluster, convert to CSV, and label cluster networks.

$ClusterName = "HVCL01"
$CsvDiskName = "Cluster Disk 1"

$ManagementNetworkName = "Cluster Network 1"
$LiveMigrationNetworkName = "Cluster Network 2"
$CsvNetworkName = "Cluster Network 3"

Write-Host "Cluster available disks:" -ForegroundColor Cyan

Get-ClusterAvailableDisk -Cluster $ClusterName |
    Format-Table Number, FriendlyName, Size, PartitionStyle -AutoSize

Write-Host "Adding available disks to cluster storage..." -ForegroundColor Cyan

$AvailableDisks = Get-ClusterAvailableDisk -Cluster $ClusterName

if ($AvailableDisks) {
    $AvailableDisks | Add-ClusterDisk
}
else {
    Write-Warning "No available cluster disks found. Confirm shared storage is visible to all nodes."
}

Write-Host "Cluster disks after add:" -ForegroundColor Cyan

Get-ClusterResource -Cluster $ClusterName |
    Where-Object { $_.ResourceType -eq "Physical Disk" } |
    Format-Table Name, State, OwnerGroup, OwnerNode -AutoSize

Write-Host "Adding disk to Cluster Shared Volumes if present: $CsvDiskName" -ForegroundColor Cyan

$DiskResource = Get-ClusterResource -Cluster $ClusterName -Name $CsvDiskName -ErrorAction SilentlyContinue

if ($DiskResource) {
    Add-ClusterSharedVolume -Name $CsvDiskName -Cluster $ClusterName
}
else {
    Write-Warning "Disk resource not found: $CsvDiskName"
}

Write-Host "CSV inventory:" -ForegroundColor Green

Get-ClusterSharedVolume -Cluster $ClusterName |
    Format-List *

Write-Host "Current cluster networks:" -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName |
    Format-Table Name, Address, AddressMask, Role, Metric, AutoMetric -AutoSize

Write-Host "Configuring cluster network roles." -ForegroundColor Cyan
Write-Host "Role values: 0 = None, 1 = Cluster only, 3 = Cluster and Client." -ForegroundColor Yellow

$Mgmt = Get-ClusterNetwork -Cluster $ClusterName -Name $ManagementNetworkName -ErrorAction SilentlyContinue
$LM = Get-ClusterNetwork -Cluster $ClusterName -Name $LiveMigrationNetworkName -ErrorAction SilentlyContinue
$CSV = Get-ClusterNetwork -Cluster $ClusterName -Name $CsvNetworkName -ErrorAction SilentlyContinue

if ($Mgmt) {
    $Mgmt.Role = 3
    $Mgmt.Name = "Management"
}

if ($LM) {
    $LM.Role = 1
    $LM.Name = "LiveMigration"
}

if ($CSV) {
    $CSV.Role = 1
    $CSV.Name = "CSV-Cluster"
}

Write-Host "Cluster networks after role configuration:" -ForegroundColor Green

Get-ClusterNetwork -Cluster $ClusterName |
    Format-Table Name, Address, AddressMask, Role, Metric, AutoMetric -AutoSize
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Create_Clustered_VM_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Create_Clustered_VM_Skeleton
# Purpose:
# Create a VM on CSV storage and make it highly available as a clustered VM role.

$ClusterName = "HVCL01"
$VMName = "CL-VM01"
$SwitchName = "vSwitch-External"

$VMRoot = "C:\ClusterStorage\Volume1\VMs"
$VHDXRoot = "C:\ClusterStorage\Volume1\VHDX"
$VMPath = Join-Path $VMRoot $VMName
$VHDXPath = Join-Path $VHDXRoot "$VMName.vhdx"

$StartupMemory = 4GB
$VHDXSize = 80GB
$Generation = 2
$ProcessorCount = 2

Write-Host "Validating CSV paths..." -ForegroundColor Cyan

New-Item -Path $VMRoot -ItemType Directory -Force | Out-Null
New-Item -Path $VHDXRoot -ItemType Directory -Force | Out-Null

if (-not (Test-Path "C:\ClusterStorage\Volume1")) {
    throw "CSV path not found: C:\ClusterStorage\Volume1"
}

Write-Host "Validating virtual switch..." -ForegroundColor Cyan

Get-VMSwitch -Name $SwitchName -ErrorAction Stop | Out-Null

Write-Host "Checking VM name availability..." -ForegroundColor Cyan

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

Write-Host "Configuring VM CPU and checkpoint type..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $VMName `
    -Count $ProcessorCount

Set-VM `
    -Name $VMName `
    -CheckpointType Production `
    -AutomaticStartAction Nothing `
    -AutomaticStopAction ShutDown

Write-Host "Making VM highly available..." -ForegroundColor Cyan

Add-ClusterVirtualMachineRole `
    -VMName $VMName `
    -Cluster $ClusterName

Write-Host "Clustered VM role state:" -ForegroundColor Green

Get-ClusterGroup -Cluster $ClusterName -Name $VMName |
    Format-List *

Write-Host "Clustered VM resources:" -ForegroundColor Cyan

Get-ClusterResource -Cluster $ClusterName |
    Where-Object { $_.OwnerGroup -eq $VMName } |
    Format-Table Name, ResourceType, State, OwnerGroup, OwnerNode -AutoSize
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Live_Migration_And_Failover_Test_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Live_Migration_And_Failover_Test_Skeleton
# Purpose:
# Test live migration and controlled failover for a clustered VM.

$ClusterName = "HVCL01"
$VMName = "CL-VM01"
$DestinationNode = "HV02"

Write-Host "Validating clustered VM role..." -ForegroundColor Cyan

$Group = Get-ClusterGroup -Cluster $ClusterName -Name $VMName -ErrorAction Stop

Write-Host "Current clustered VM owner:" -ForegroundColor Cyan

$Group |
    Select-Object Name, OwnerNode, State, GroupType |
    Format-List

Write-Host "Starting clustered VM if offline..." -ForegroundColor Cyan

if ($Group.State -ne "Online") {
    Start-ClusterGroup -Cluster $ClusterName -Name $VMName
}

Write-Host "Moving clustered VM role to: $DestinationNode" -ForegroundColor Cyan

Move-ClusterVirtualMachineRole `
    -Cluster $ClusterName `
    -Name $VMName `
    -Node $DestinationNode

Start-Sleep -Seconds 10

Write-Host "Clustered VM owner after move:" -ForegroundColor Green

Get-ClusterGroup -Cluster $ClusterName -Name $VMName |
    Select-Object Name, OwnerNode, State, GroupType |
    Format-List

Write-Host "VM state on destination node:" -ForegroundColor Cyan

Invoke-Command -ComputerName $DestinationNode -ScriptBlock {
    param($VMName)

    Get-VM -Name $VMName |
        Select-Object Name, State, Status, Uptime, Path |
        Format-List

    Get-VMNetworkAdapter -VMName $VMName |
        Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
        Format-List
} -ArgumentList $VMName

Write-Host "Controlled failover test completed." -ForegroundColor Green
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_PostChange_Verification_Skeleton

~~~powershell
# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_PostChange_Verification_Skeleton
# Purpose:
# Capture cluster, nodes, networks, quorum, CSVs, clustered VMs, and event logs after cluster configuration.

$EvidenceRoot = "C:\Admin\HyperV-Cluster"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$ClusterName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing cluster summary..." -ForegroundColor Cyan

Get-Cluster -Name $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Cluster-Summary.txt")

Write-Host "Capturing cluster nodes..." -ForegroundColor Cyan

Get-ClusterNode -Cluster $ClusterName |
    Select-Object Name, State, NodeWeight, FaultDomain |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-Cluster-Nodes.txt")

Write-Host "Capturing quorum configuration..." -ForegroundColor Cyan

Get-ClusterQuorum -Cluster $ClusterName |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-Cluster-Quorum.txt")

Write-Host "Capturing cluster networks..." -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-Cluster-Networks.txt")

Write-Host "Capturing cluster groups..." -ForegroundColor Cyan

Get-ClusterGroup -Cluster $ClusterName |
    Select-Object Name, OwnerNode, State, GroupType, Priority |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-Cluster-Groups.txt")

Write-Host "Capturing cluster resources..." -ForegroundColor Cyan

Get-ClusterResource -Cluster $ClusterName |
    Select-Object Name, ResourceType, State, OwnerGroup, OwnerNode |
    Sort-Object OwnerGroup, Name |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-Cluster-Resources.txt")

Write-Host "Capturing Cluster Shared Volumes..." -ForegroundColor Cyan

Get-ClusterSharedVolume -Cluster $ClusterName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "07-Cluster-SharedVolumes.txt")

Write-Host "Capturing clustered VM state from all nodes..." -ForegroundColor Cyan

$Nodes = Get-ClusterNode -Cluster $ClusterName | Select-Object -ExpandProperty Name

foreach ($Node in $Nodes) {
    Invoke-Command -ComputerName $Node -ScriptBlock {
        "Node: $env:COMPUTERNAME"

        Get-VM |
            Sort-Object Name |
            Select-Object Name, State, Status, Generation, Uptime, Path |
            Format-Table -AutoSize

        Get-VMSwitch |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription |
            Format-Table -AutoSize
    } | Out-File -FilePath (Join-Path $OutputPath "08-Node-$Node-VMState.txt") -Encoding UTF8
}

Write-Host "Capturing failover clustering events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-FailoverClustering/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-FailoverClustering-Operational-RecentEvents.txt") -Encoding UTF8

Write-Host "Capturing Hyper-V events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "10-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Write-Host "Post-change cluster evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-WindowsFeature Failover-Clustering` | Each node | Confirms clustering feature state |
| `Install-WindowsFeature Failover-Clustering -IncludeManagementTools` | Each node | Installs clustering feature and tools |
| `Test-Cluster -Node HV01,HV02` | Admin node | Runs cluster validation |
| `New-Cluster -Name HVCL01 -Node HV01,HV02 -StaticAddress <ip>` | Admin node | Creates the failover cluster |
| `Get-Cluster` | Admin node | Confirms cluster exists |
| `Get-ClusterNode` | Admin node | Confirms node membership and node state |
| `Get-ClusterQuorum` | Admin node | Confirms quorum mode and witness |
| `Set-ClusterQuorum -FileShareWitness '<path>'` | Admin node | Configures file share witness |
| `Get-ClusterNetwork` | Admin node | Shows cluster networks, roles, and metrics |
| `Get-ClusterAvailableDisk` | Admin node | Shows disks that can be added to the cluster |
| `Get-ClusterAvailableDisk \| Add-ClusterDisk` | Admin node | Adds shared disks to the cluster |
| `Get-ClusterSharedVolume` | Admin node | Shows CSV inventory |
| `Add-ClusterSharedVolume -Name '<cluster-disk-name>'` | Admin node | Converts cluster disk to CSV |
| `New-VM -Name '<vm-name>' -Path 'C:\ClusterStorage\Volume1\VMs'` | Owner node | Creates VM on CSV storage |
| `Add-ClusterVirtualMachineRole -VMName '<vm-name>'` | Owner node | Makes VM highly available |
| `Get-ClusterGroup -Name '<vm-name>'` | Admin node | Confirms clustered VM role |
| `Get-ClusterResource \| Where-Object OwnerGroup -eq '<vm-name>'` | Admin node | Confirms VM cluster resources |
| `Move-ClusterVirtualMachineRole -Name '<vm-name>' -Node '<node>'` | Admin node | Tests clustered VM live migration |
| `Move-ClusterGroup -Name '<vm-name>' -Node '<node>'` | Admin node | Tests controlled role failover |
| `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 20` | Admin node | Checks cluster event history |

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm clustered VM role state | Admin node | `Get-ClusterGroup -Name '<vm-name>'` | Current owner and state are known |
| 2 | Stop clustered VM role if required | Admin node | `Stop-ClusterGroup -Name '<vm-name>'` | Clustered VM role is offline |
| 3 | Remove clustered VM role but keep VM files if approved | Admin node | `Remove-ClusterGroup -Name '<vm-name>' -RemoveResources` | Cluster role is removed |
| 4 | Remove VM registration if rollback requires it | Owner node | `Remove-VM -Name '<vm-name>' -Force` | VM registration is removed |
| 5 | Delete VM files only after path verification | Owner node | `Remove-Item 'C:\ClusterStorage\Volume1\VMs\<vm-name>' -Recurse -Force` | VM files are removed only if approved |
| 6 | Remove CSV if cluster storage rollback requires it | Admin node | `Remove-ClusterSharedVolume -Name '<csv-name>'` | CSV role is removed |
| 7 | Remove cluster disk from cluster if required | Admin node | `Remove-ClusterResource -Name '<cluster-disk-name>' -Force` | Disk resource is removed from cluster |
| 8 | Restore cluster network roles | Admin node | Set `Get-ClusterNetwork` role values from pre-change evidence | Network roles return to baseline |
| 9 | Restore quorum if changed | Admin node | `Set-ClusterQuorum -FileShareWitness '<old-path>'` | Quorum returns to baseline |
| 10 | Destroy lab cluster only if full rollback is approved | Admin node | `Remove-Cluster -Cluster '<cluster-name>' -Force -CleanupAD` | Cluster is removed |
| 11 | Remove clustering feature only if lab cleanup requires it | Each node | `Uninstall-WindowsFeature Failover-Clustering` | Clustering feature is removed |
| 12 | Capture rollback evidence | Admin node | `Get-Cluster -ErrorAction SilentlyContinue; Get-ClusterNode -ErrorAction SilentlyContinue` | Rollback state is documented |

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Test-Cluster` fails inventory checks | Node OS, driver, update, hardware, or role mismatch | Standardize nodes and rerun validation |
| `Test-Cluster` fails network checks | Missing IP connectivity, DNS issue, or incorrect NIC config | Fix IP, DNS, routes, firewall, and NIC naming |
| `Test-Cluster` fails storage checks | Shared disk not visible to every node or storage misconfigured | Present shared storage to all nodes and validate MPIO or iSCSI settings |
| Cluster creation fails | Cluster name object permission, DNS, IP conflict, or validation issue | Fix AD permissions, DNS, static IP, and rerun validation |
| Cluster name does not come online | CNO creation failure or DNS update failure | Prestage CNO or grant OU permissions |
| Node cannot join cluster | Firewall, domain, DNS, time skew, or cluster service issue | Fix domain connectivity, DNS, time, and cluster service |
| CSV does not appear under `C:\ClusterStorage` | Disk not added as CSV or storage not suitable | Add disk to cluster storage and then add as CSV |
| CSV enters redirected mode | Storage path, network, or CSV communication issue | Check storage health, cluster network, and FailoverClustering events |
| Clustered VM creation fails | VM files not on shared storage or VM config invalid | Place VM on CSV and validate VM paths |
| `Add-ClusterVirtualMachineRole` fails | VM is running, unsupported config, or path issue | Stop VM if needed, verify VM config and CSV path |
| Clustered VM fails to start on another node | Switch name missing or VM storage not accessible | Create matching switch on every node and confirm CSV access |
| Live migration fails | Network, authentication, CPU compatibility, or switch mismatch | Fix migration networks, auth, CPU compatibility, and switch naming |
| Controlled failover works but VM loses network | Destination node switch or VLAN mismatch | Standardize virtual switch names and VLANs |
| Quorum warning appears | Witness missing or node vote issue | Configure file share, disk, or cloud witness |
| File share witness fails | Share permissions or NTFS permissions wrong | Grant cluster computer object appropriate permissions |
| Cluster validation warns about single network | Lab has insufficient network separation | Add dedicated management or migration networks where possible |
| Event log shows cluster resource failure | Clustered VM resource, storage, or network issue | Review resource dependencies and cluster operational logs |
| Removing cluster leaves AD object | CNO cleanup not completed | Remove stale AD object manually only after confirming cluster removal |
| Removing VM role deletes wrong resources | Poor role identification | Validate role name, owner group, and VM paths before removal |

# 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs_Related_Labs

| Lab                                                                           | Relationship                                                                                     |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| 00_Hyper-V_Index.md                                                           | Places this failover clustering task in the full Hyper-V suite                                   |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Confirms node readiness, CPU, memory, network, storage, and domain state                         |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Required before Hyper-V clustering workloads are possible                                        |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md                       | Establishes VM, VHDX, checkpoint, and smart paging paths                                         |
| 04_Create_And_Configure_Virtual_Switches.md                                   | Matching virtual switch names are required on every cluster node                                 |
| 05_Create_Generation_1_And_Generation_2_VMs.md                                | Creates VMs that can later be made highly available                                              |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md                         | VM sizing affects migration and failover behavior                                                |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md                             | VHDX placement and health affect clustered VM storage                                            |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md                   | VM adapter and VLAN settings must be consistent across nodes                                     |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md                | Checkpoint chains affect clustered VM storage and movement                                       |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md                 | VM lifecycle operations are required during cluster testing                                      |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md                              | Backups should exist before cluster role movement or failover testing                            |
| 14_Configure_Live_Migration_And_Storage_Migration.md                          | Live migration concepts apply directly to clustered VM movement                                  |
| 15_Configure_Hyper-V_Replica_And_Failover_Testing.md                          | Replica is DR-style recovery, while clustering is local HA                                       |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses cluster events, CSV state, VM resources, networks, and storage paths during troubleshooting |