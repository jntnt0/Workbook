# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Index
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts.md
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Source_Basis
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Mental_Model
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Planning_Table
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configuration_Checklist
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_PreChange_Capture_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_NIC_And_Feature_Readiness_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Create_SET_vSwitch_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Create_ManagementOS_vNICs_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_IPs_VLANs_And_DNS_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_QoS_DCB_And_Bandwidth_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_RDMA_And_SMB_Direct_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_Cluster_Networks_And_Live_Migration_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_PostChange_Verification_Skeleton
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Verification_Commands
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Rollback
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Failure_Checks
25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Related_Labs

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V virtual switch | Supports external vSwitch and management OS vNIC design |
| Microsoft Learn | Switch Embedded Teaming | Supports SET-based converged networking on cluster hosts |
| Microsoft Learn | Hyper-V host networking | Supports host vNICs, VLANs, VM traffic, and management OS connectivity |
| Microsoft Learn | SMB Direct and RDMA | Supports storage, CSV, and live migration network acceleration |
| Microsoft Learn | Data Center Bridging | Supports PFC, ETS, QoS policy, and RoCE lossless network planning |
| Microsoft Learn | Failover cluster networks | Supports cluster network roles, metrics, and live migration network selection |
| Microsoft Learn | Hyper-V live migration | Supports migration authentication and performance option configuration |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Converged networking | Multiple host traffic types share the same physical NIC team through virtual adapters |
| SET | Switch Embedded Teaming, the preferred Hyper-V teaming model for modern cluster hosts |
| Team member | Physical NIC participating in the SET vSwitch |
| Management OS vNIC | Virtual NIC created on the Hyper-V host and connected to the vSwitch |
| Management traffic | Host administration, domain, DNS, monitoring, and general server access |
| Cluster traffic | Heartbeat and cluster control traffic between nodes |
| CSV traffic | Cluster Shared Volume coordination and redirected I/O path |
| Live migration traffic | Traffic used to move running VMs between Hyper-V hosts |
| SMB Direct traffic | SMB storage traffic accelerated with RDMA |
| Storage traffic | S2D, SMB, CSV, or file-server-backed VM storage traffic |
| VM traffic | Guest VM production network traffic |
| VLAN separation | Logical segmentation of traffic types over shared adapters |
| QoS | Bandwidth and priority control for traffic types sharing converged links |
| DCB | Data Center Bridging, commonly used with RoCE RDMA to create lossless traffic classes |
| PFC | Priority Flow Control, pauses selected traffic priorities instead of the full link |
| ETS | Enhanced Transmission Selection, bandwidth allocation per traffic class |
| RDMA | Low-latency network path used by SMB Direct, CSV, and live migration over SMB |
| Rollback boundary | This workbook configures converged host networking, not physical switch firmware or cabling redesign |

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Cluster name | `HVCL01` | `<cluster-name>` |
| Node 1 | `HV01` | `<node-1>` |
| Node 2 | `HV02` | `<node-2>` |
| Node 3 optional | `HV03` | `<node-3>` |
| SET switch name | `vSwitch-SET-Converged` | `<set-switch-name>` |
| Physical NIC 1 | `NIC01` | `<nic-1>` |
| Physical NIC 2 | `NIC02` | `<nic-2>` |
| Physical NIC speed | `10Gb`, `25Gb`, `40Gb`, `100Gb` | `<nic-speed>` |
| SET load balancing | Dynamic, HyperVPort | `<load-balancing-mode>` |
| Management OS access on switch | Enabled | `<enabled-disabled>` |
| Management vNIC name | `Mgmt` | `<mgmt-vnic>` |
| Cluster vNIC name | `Cluster` | `<cluster-vnic>` |
| Live migration vNIC name | `LiveMigration` | `<lm-vnic>` |
| SMB storage vNIC 1 | `SMB01` | `<smb1-vnic>` |
| SMB storage vNIC 2 | `SMB02` | `<smb2-vnic>` |
| Management VLAN | `10` | `<mgmt-vlan>` |
| Cluster VLAN | `20` | `<cluster-vlan>` |
| Live migration VLAN | `50` | `<lm-vlan>` |
| SMB01 VLAN | `60` | `<smb1-vlan>` |
| SMB02 VLAN | `61` | `<smb2-vlan>` |
| Management IP | `192.168.10.21/24` | `<mgmt-ip-prefix>` |
| Cluster IP | `192.168.20.21/24` | `<cluster-ip-prefix>` |
| Live migration IP | `192.168.50.21/24` | `<lm-ip-prefix>` |
| SMB01 IP | `192.168.60.21/24` | `<smb1-ip-prefix>` |
| SMB02 IP | `192.168.61.21/24` | `<smb2-ip-prefix>` |
| Default gateway vNIC | Management only | `<gateway-vnic>` |
| DNS vNIC | Management only | `<dns-vnic>` |
| RDMA required | Yes or No | `<yes-no>` |
| RDMA transport | RoCEv2, iWARP | `<rdma-transport>` |
| DCB required | Yes or No | `<yes-no>` |
| QoS policy model | SMB priority, cluster priority, default | `<qos-model>` |
| Live migration performance option | SMB, Compression, TCPIP | `<lm-performance-option>` |
| Evidence path | `C:\Admin\HyperV-ConvergedNetworking` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role | Each node | `Get-WindowsFeature Hyper-V` | Hyper-V installed |
| 2 | Confirm Failover Clustering feature | Each node | `Get-WindowsFeature Failover-Clustering` | Cluster feature installed |
| 3 | Capture current network baseline | Each node | Run pre-change capture skeleton | Existing NIC, IP, vSwitch, RDMA, and cluster state documented |
| 4 | Confirm target physical NICs | Each node | `Get-NetAdapter -Name '<nic-1>','<nic-2>'` | Team NICs are up and same speed |
| 5 | Confirm no conflicting vSwitch exists | Each node | `Get-VMSwitch` | Existing switch state known |
| 6 | Confirm RDMA capability if planned | Each node | `Get-NetAdapterRdma` | RDMA capability known |
| 7 | Confirm DCB feature if RoCE planned | Each node | `Get-WindowsFeature Data-Center-Bridging` | DCB state known |
| 8 | Install required features | Each node | `Install-WindowsFeature Hyper-V,Failover-Clustering,Data-Center-Bridging -IncludeManagementTools` | Required features installed |
| 9 | Create SET vSwitch | Each node | `New-VMSwitch -Name '<set-switch>' -NetAdapterName '<nic-1>','<nic-2>' -EnableEmbeddedTeaming $true -AllowManagementOS $true` | SET vSwitch created |
| 10 | Configure SET load balancing | Each node | `Set-VMSwitchTeam -Name '<set-switch>' -LoadBalancingAlgorithm Dynamic` | SET load balancing configured |
| 11 | Create host vNICs | Each node | `Add-VMNetworkAdapter -ManagementOS -Name '<vnic>' -SwitchName '<set-switch>'` | Host vNICs created |
| 12 | Configure host vNIC VLANs | Each node | `Set-VMNetworkAdapterVlan -ManagementOS -VMNetworkAdapterName '<vnic>' -Access -VlanId '<id>'` | VLANs assigned |
| 13 | Configure host vNIC IP settings | Each node | `New-NetIPAddress; Set-DnsClientServerAddress` | IP plan applied |
| 14 | Confirm only management vNIC has default gateway | Each node | `Get-NetRoute -DestinationPrefix '0.0.0.0/0'` | Default route exists only on management network |
| 15 | Configure QoS policies if required | Each node | `New-NetQosPolicy` | Traffic priorities applied |
| 16 | Configure DCB if RoCE required | Each node | `Enable-NetQosFlowControl; New-NetQosTrafficClass` | PFC and ETS configured |
| 17 | Enable RDMA on SMB vNICs | Each node | `Enable-NetAdapterRdma -Name 'vEthernet (SMB01)','vEthernet (SMB02)'` | RDMA enabled |
| 18 | Validate SMB Direct interfaces | Each node | `Get-SmbClientNetworkInterface` | SMB interfaces show RDMA capable where expected |
| 19 | Configure cluster network names and roles | Cluster node | `Get-ClusterNetwork` and set role values | Cluster networks labeled and scoped |
| 20 | Configure live migration | Each node | `Set-VMHost -VirtualMachineMigrationEnabled $true -VirtualMachineMigrationPerformanceOption SMB` | Live migration enabled |
| 21 | Set live migration network preference if required | Cluster node | `Set-ClusterNetwork` metrics or Failover Cluster Manager | LM network preference is controlled |
| 22 | Validate node-to-node connectivity | Each node | `Test-NetConnection <peer-ip>` | All planned networks pass |
| 23 | Validate SMB multichannel and RDMA sessions | Each node | `Get-SmbMultichannelConnection` | SMB channels visible after workload traffic |
| 24 | Capture post-change evidence | Each node | Run post-change verification skeleton | Converged network configuration documented |

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_PreChange_Capture_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_PreChange_Capture_Skeleton
# Purpose:
# Capture node networking, physical NICs, vSwitches, host vNICs, IP routes, DNS, RDMA, QoS, DCB, cluster networks, and events before changes.

$EvidenceRoot = "C:\Admin\HyperV-ConvergedNetworking"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$Nodes = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "PreChange-$ClusterName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing admin context..." -ForegroundColor Cyan

whoami |
    Out-File -FilePath (Join-Path $OutputPath "00-AdminContext.txt") -Encoding UTF8

foreach ($Node in $Nodes) {
    $NodePath = Join-Path $OutputPath $Node
    New-Item -Path $NodePath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing network baseline from $Node..." -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        "ComputerInfo"
        Get-ComputerInfo |
            Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
            Format-List

        "Features"
        Get-WindowsFeature Hyper-V, Failover-Clustering, Data-Center-Bridging |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize

        "Physical NICs"
        Get-NetAdapter |
            Sort-Object Name |
            Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverVersion |
            Format-Table -AutoSize

        "NIC Hardware Info"
        Get-NetAdapterHardwareInfo -ErrorAction SilentlyContinue |
            Format-List *

        "NIC Advanced Properties"
        Get-NetAdapterAdvancedProperty -ErrorAction SilentlyContinue |
            Sort-Object Name, DisplayName |
            Select-Object Name, DisplayName, DisplayValue, RegistryKeyword |
            Format-Table -AutoSize

        "IP Configuration"
        Get-NetIPConfiguration |
            Format-List

        "IPv4 Routes"
        Get-NetRoute -AddressFamily IPv4 |
            Sort-Object DestinationPrefix, RouteMetric |
            Format-Table DestinationPrefix, InterfaceAlias, NextHop, RouteMetric, ifMetric -AutoSize

        "DNS Client Server Addresses"
        Get-DnsClientServerAddress |
            Format-Table InterfaceAlias, AddressFamily, ServerAddresses -AutoSize

        "Virtual Switches"
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, IovEnabled, BandwidthReservationMode |
            Format-Table -AutoSize

        "SET Teams"
        Get-VMSwitchTeam -ErrorAction SilentlyContinue |
            Format-List *

        "Management OS vNICs"
        Get-VMNetworkAdapter -ManagementOS -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchName, MacAddress, Status, IPAddresses |
            Format-List

        "Management OS vNIC VLANs"
        Get-VMNetworkAdapterVlan -ManagementOS -ErrorAction SilentlyContinue |
            Format-List

        "RDMA"
        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

        "SMB Client Network Interfaces"
        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        "SMB Multichannel Connections"
        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize

        "QoS Policies"
        Get-NetQosPolicy -ErrorAction SilentlyContinue |
            Format-List *

        "QoS Flow Control"
        Get-NetQosFlowControl -ErrorAction SilentlyContinue |
            Format-Table Priority, Enabled -AutoSize

        "QoS Traffic Classes"
        Get-NetQosTrafficClass -ErrorAction SilentlyContinue |
            Format-Table Name, Algorithm, BandwidthPercentage, Priority -AutoSize

        "Hyper-V Host Migration Settings"
        Get-VMHost -ErrorAction SilentlyContinue |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption, MaximumVirtualMachineMigrations, MaximumStorageMigrations |
            Format-List
    } |
    Out-File -FilePath (Join-Path $NodePath "Node-ConvergedNetwork-Before.txt") -Encoding UTF8
}

Write-Host "Capturing cluster networks if cluster exists..." -ForegroundColor Cyan

try {
    Get-ClusterNetwork -Cluster $ClusterName -ErrorAction Stop |
        Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "ClusterNetworks-Before.txt")

    Get-ClusterNode -Cluster $ClusterName |
        Select-Object Name, State, NodeWeight |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "ClusterNodes-Before.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "Cluster-State-Before-Error.txt") -Encoding UTF8
}

Write-Host "Pre-change converged networking evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_NIC_And_Feature_Readiness_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_NIC_And_Feature_Readiness_Skeleton
# Purpose:
# Validate physical NICs, drivers, RDMA, RSS, VMQ, DCB, and required Windows features before creating converged networking.

$Nodes = @("HV01", "HV02")
$TeamNics = @("NIC01", "NIC02")
$RequireDCB = $true

foreach ($Node in $Nodes) {
    Write-Host "Validating node: $Node" -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($TeamNics, $RequireDCB)

        Write-Host "Feature state..." -ForegroundColor Cyan
        Get-WindowsFeature Hyper-V, Failover-Clustering, Data-Center-Bridging |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize

        if ($RequireDCB) {
            Install-WindowsFeature Data-Center-Bridging -IncludeManagementTools
        }

        Write-Host "Physical NICs..." -ForegroundColor Cyan
        foreach ($Nic in $TeamNics) {
            Get-NetAdapter -Name $Nic -ErrorAction Stop |
                Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverVersion |
                Format-List
        }

        Write-Host "RDMA capability..." -ForegroundColor Cyan
        Get-NetAdapterRdma -Name $TeamNics -ErrorAction SilentlyContinue |
            Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

        Write-Host "RSS state..." -ForegroundColor Cyan
        Get-NetAdapterRss -Name $TeamNics -ErrorAction SilentlyContinue |
            Select-Object Name, Enabled, NumberOfReceiveQueues, BaseProcessorNumber, MaxProcessors, Profile |
            Format-Table -AutoSize

        Write-Host "VMQ state..." -ForegroundColor Cyan
        Get-NetAdapterVmq -Name $TeamNics -ErrorAction SilentlyContinue |
            Select-Object Name, Enabled, NumberOfReceiveQueues, BaseProcessorNumber, MaxProcessors |
            Format-Table -AutoSize

        Write-Host "Existing vSwitch state..." -ForegroundColor Cyan
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
            Format-Table -AutoSize

        Write-Host "Existing IP configuration on target NICs..." -ForegroundColor Cyan
        foreach ($Nic in $TeamNics) {
            Get-NetIPConfiguration -InterfaceAlias $Nic -ErrorAction SilentlyContinue |
                Format-List
        }
    } -ArgumentList $TeamNics, $RequireDCB
}
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Create_SET_vSwitch_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Create_SET_vSwitch_Skeleton
# Purpose:
# Create a consistent SET vSwitch across cluster hosts.
# Run from an admin workstation or one cluster node.

$Nodes = @("HV01", "HV02")
$SetSwitchName = "vSwitch-SET-Converged"
$TeamNics = @("NIC01", "NIC02")
$LoadBalancingAlgorithm = "Dynamic"
$AllowManagementOS = $true

foreach ($Node in $Nodes) {
    Write-Host "Creating SET vSwitch on $Node..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($SetSwitchName, $TeamNics, $LoadBalancingAlgorithm, $AllowManagementOS)

        Write-Host "Checking existing vSwitches..." -ForegroundColor Cyan

        if (Get-VMSwitch -Name $SetSwitchName -ErrorAction SilentlyContinue) {
            Write-Warning "vSwitch already exists: $SetSwitchName"
        }
        else {
            Write-Host "Creating SET switch: $SetSwitchName" -ForegroundColor Cyan

            New-VMSwitch `
                -Name $SetSwitchName `
                -NetAdapterName $TeamNics `
                -EnableEmbeddedTeaming $true `
                -AllowManagementOS $AllowManagementOS
        }

        Write-Host "Configuring SET load balancing..." -ForegroundColor Cyan

        Set-VMSwitchTeam `
            -Name $SetSwitchName `
            -LoadBalancingAlgorithm $LoadBalancingAlgorithm

        Write-Host "SET switch state:" -ForegroundColor Green

        Get-VMSwitch -Name $SetSwitchName |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, BandwidthReservationMode |
            Format-List

        Get-VMSwitchTeam -Name $SetSwitchName |
            Format-List *
    } -ArgumentList $SetSwitchName, $TeamNics, $LoadBalancingAlgorithm, $AllowManagementOS
}
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Create_ManagementOS_vNICs_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Create_ManagementOS_vNICs_Skeleton
# Purpose:
# Create host vNICs for management, cluster, live migration, and SMB storage traffic on each node.

$Nodes = @("HV01", "HV02")
$SetSwitchName = "vSwitch-SET-Converged"

$HostVnics = @(
    @{ Name = "Mgmt";          VlanId = 10 },
    @{ Name = "Cluster";       VlanId = 20 },
    @{ Name = "LiveMigration"; VlanId = 50 },
    @{ Name = "SMB01";         VlanId = 60 },
    @{ Name = "SMB02";         VlanId = 61 }
)

foreach ($Node in $Nodes) {
    Write-Host "Creating management OS vNICs on $Node..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($SetSwitchName, $HostVnics)

        foreach ($Vnic in $HostVnics) {
            Write-Host "Processing vNIC: $($Vnic.Name)" -ForegroundColor Yellow

            if (-not (Get-VMNetworkAdapter -ManagementOS -Name $Vnic.Name -ErrorAction SilentlyContinue)) {
                Add-VMNetworkAdapter `
                    -ManagementOS `
                    -Name $Vnic.Name `
                    -SwitchName $SetSwitchName
            }
            else {
                Write-Host "vNIC already exists: $($Vnic.Name)" -ForegroundColor Yellow
            }

            Set-VMNetworkAdapterVlan `
                -ManagementOS `
                -VMNetworkAdapterName $Vnic.Name `
                -Access `
                -VlanId $Vnic.VlanId
        }

        Write-Host "Management OS vNICs after change:" -ForegroundColor Green

        Get-VMNetworkAdapter -ManagementOS |
            Select-Object Name, SwitchName, MacAddress, Status, IPAddresses |
            Format-List

        Get-VMNetworkAdapterVlan -ManagementOS |
            Format-List
    } -ArgumentList $SetSwitchName, $HostVnics
}
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_IPs_VLANs_And_DNS_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_IPs_VLANs_And_DNS_Skeleton
# Purpose:
# Configure IP addressing on host vNICs.
# Only the management vNIC should normally receive a default gateway and DNS servers.

$NodeConfigs = @(
    @{
        Node = "HV01"
        MgmtIP = "192.168.10.21"; MgmtPrefix = 24; MgmtGateway = "192.168.10.1"; Dns = @("192.168.10.10","192.168.10.11")
        ClusterIP = "192.168.20.21"; ClusterPrefix = 24
        LMIP = "192.168.50.21"; LMPrefix = 24
        SMB01IP = "192.168.60.21"; SMB01Prefix = 24
        SMB02IP = "192.168.61.21"; SMB02Prefix = 24
    },
    @{
        Node = "HV02"
        MgmtIP = "192.168.10.22"; MgmtPrefix = 24; MgmtGateway = "192.168.10.1"; Dns = @("192.168.10.10","192.168.10.11")
        ClusterIP = "192.168.20.22"; ClusterPrefix = 24
        LMIP = "192.168.50.22"; LMPrefix = 24
        SMB01IP = "192.168.60.22"; SMB01Prefix = 24
        SMB02IP = "192.168.61.22"; SMB02Prefix = 24
    }
)

foreach ($Config in $NodeConfigs) {
    Write-Host "Configuring IPs on $($Config.Node)..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Config.Node -ScriptBlock {
        param($Config)

        function Set-StaticIPv4 {
            param(
                [string]$Alias,
                [string]$IPAddress,
                [int]$PrefixLength,
                [string]$Gateway = $null,
                [string[]]$DnsServers = $null
            )

            Write-Host "Configuring $Alias -> $IPAddress/$PrefixLength" -ForegroundColor Yellow

            Get-NetIPAddress -InterfaceAlias $Alias -AddressFamily IPv4 -ErrorAction SilentlyContinue |
                Where-Object { $_.IPAddress -notlike "169.254*" } |
                Remove-NetIPAddress -Confirm:$false -ErrorAction SilentlyContinue

            Get-NetRoute -InterfaceAlias $Alias -DestinationPrefix "0.0.0.0/0" -ErrorAction SilentlyContinue |
                Remove-NetRoute -Confirm:$false -ErrorAction SilentlyContinue

            if ($Gateway) {
                New-NetIPAddress `
                    -InterfaceAlias $Alias `
                    -IPAddress $IPAddress `
                    -PrefixLength $PrefixLength `
                    -DefaultGateway $Gateway
            }
            else {
                New-NetIPAddress `
                    -InterfaceAlias $Alias `
                    -IPAddress $IPAddress `
                    -PrefixLength $PrefixLength
            }

            if ($DnsServers) {
                Set-DnsClientServerAddress `
                    -InterfaceAlias $Alias `
                    -ServerAddresses $DnsServers
            }
            else {
                Set-DnsClientServerAddress `
                    -InterfaceAlias $Alias `
                    -ResetServerAddresses
            }
        }

        Set-StaticIPv4 -Alias "vEthernet (Mgmt)" -IPAddress $Config.MgmtIP -PrefixLength $Config.MgmtPrefix -Gateway $Config.MgmtGateway -DnsServers $Config.Dns
        Set-StaticIPv4 -Alias "vEthernet (Cluster)" -IPAddress $Config.ClusterIP -PrefixLength $Config.ClusterPrefix
        Set-StaticIPv4 -Alias "vEthernet (LiveMigration)" -IPAddress $Config.LMIP -PrefixLength $Config.LMPrefix
        Set-StaticIPv4 -Alias "vEthernet (SMB01)" -IPAddress $Config.SMB01IP -PrefixLength $Config.SMB01Prefix
        Set-StaticIPv4 -Alias "vEthernet (SMB02)" -IPAddress $Config.SMB02IP -PrefixLength $Config.SMB02Prefix

        Write-Host "IP configuration after change:" -ForegroundColor Green

        Get-NetIPConfiguration |
            Format-List

        Write-Host "Default route check:" -ForegroundColor Cyan

        Get-NetRoute -DestinationPrefix "0.0.0.0/0" |
            Format-Table DestinationPrefix, InterfaceAlias, NextHop, RouteMetric, ifMetric -AutoSize

        Write-Host "DNS check:" -ForegroundColor Cyan

        Get-DnsClientServerAddress |
            Format-Table InterfaceAlias, AddressFamily, ServerAddresses -AutoSize
    } -ArgumentList $Config
}
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_QoS_DCB_And_Bandwidth_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_QoS_DCB_And_Bandwidth_Skeleton
# Purpose:
# Configure host QoS and DCB policy for converged networking.
# Use DCB especially for RoCE designs. Coordinate priorities with physical switch configuration.

$Nodes = @("HV01", "HV02")
$EnableDCB = $true

foreach ($Node in $Nodes) {
    Write-Host "Configuring QoS and DCB on $Node..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($EnableDCB)

        Write-Host "Removing old lab QoS policies if present..." -ForegroundColor Cyan

        Get-NetQosPolicy -Name "SMB" -ErrorAction SilentlyContinue | Remove-NetQosPolicy -Confirm:$false
        Get-NetQosPolicy -Name "Cluster" -ErrorAction SilentlyContinue | Remove-NetQosPolicy -Confirm:$false
        Get-NetQosPolicy -Name "LiveMigration" -ErrorAction SilentlyContinue | Remove-NetQosPolicy -Confirm:$false

        Write-Host "Creating QoS policies..." -ForegroundColor Cyan

        New-NetQosPolicy `
            -Name "SMB" `
            -NetDirectPortMatchCondition 445 `
            -PriorityValue8021Action 3

        New-NetQosPolicy `
            -Name "Cluster" `
            -Cluster `
            -PriorityValue8021Action 7

        New-NetQosPolicy `
            -Name "LiveMigration" `
            -LiveMigration `
            -PriorityValue8021Action 5

        if ($EnableDCB) {
            Write-Host "Enabling QoS flow control for selected priorities..." -ForegroundColor Cyan

            Disable-NetQosFlowControl -Priority 0,1,2,4,6 -ErrorAction SilentlyContinue
            Enable-NetQosFlowControl -Priority 3,5,7

            Write-Host "Removing old lab traffic classes if present..." -ForegroundColor Cyan

            Get-NetQosTrafficClass -Name "SMB" -ErrorAction SilentlyContinue | Remove-NetQosTrafficClass -Confirm:$false
            Get-NetQosTrafficClass -Name "LiveMigration" -ErrorAction SilentlyContinue | Remove-NetQosTrafficClass -Confirm:$false
            Get-NetQosTrafficClass -Name "Cluster" -ErrorAction SilentlyContinue | Remove-NetQosTrafficClass -Confirm:$false

            Write-Host "Creating traffic classes..." -ForegroundColor Cyan

            New-NetQosTrafficClass `
                -Name "SMB" `
                -Priority 3 `
                -BandwidthPercentage 50 `
                -Algorithm ETS

            New-NetQosTrafficClass `
                -Name "LiveMigration" `
                -Priority 5 `
                -BandwidthPercentage 30 `
                -Algorithm ETS

            New-NetQosTrafficClass `
                -Name "Cluster" `
                -Priority 7 `
                -BandwidthPercentage 10 `
                -Algorithm ETS
        }

        Write-Host "QoS policy state:" -ForegroundColor Green

        Get-NetQosPolicy |
            Format-List Name, Template, NetDirectPort, PriorityValue, PriorityValue8021Action

        Write-Host "Flow control state:" -ForegroundColor Cyan

        Get-NetQosFlowControl |
            Format-Table Priority, Enabled -AutoSize

        Write-Host "Traffic class state:" -ForegroundColor Cyan

        Get-NetQosTrafficClass |
            Format-Table Name, Priority, Algorithm, BandwidthPercentage -AutoSize
    } -ArgumentList $EnableDCB
}
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_RDMA_And_SMB_Direct_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_RDMA_And_SMB_Direct_Skeleton
# Purpose:
# Enable RDMA on SMB host vNICs and validate SMB Direct readiness.

$Nodes = @("HV01", "HV02")
$RdmaVnics = @("vEthernet (SMB01)", "vEthernet (SMB02)")

foreach ($Node in $Nodes) {
    Write-Host "Configuring RDMA on $Node..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $Node -ScriptBlock {
        param($RdmaVnics)

        Write-Host "Current RDMA state:" -ForegroundColor Cyan

        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

        foreach ($Vnic in $RdmaVnics) {
            if (Get-NetAdapter -Name $Vnic -ErrorAction SilentlyContinue) {
                Write-Host "Enabling RDMA on $Vnic" -ForegroundColor Yellow

                Enable-NetAdapterRdma -Name $Vnic -ErrorAction SilentlyContinue
            }
            else {
                Write-Warning "RDMA vNIC not found: $Vnic"
            }
        }

        Write-Host "RDMA state after change:" -ForegroundColor Green

        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

        Write-Host "SMB client network interfaces:" -ForegroundColor Cyan

        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        Write-Host "SMB multichannel connections:" -ForegroundColor Cyan

        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRSSCapable, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize
    } -ArgumentList $RdmaVnics
}

Write-Host "Run SMB traffic such as CSV, S2D, or file copy before expecting active SMB multichannel connections." -ForegroundColor Yellow
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_Cluster_Networks_And_Live_Migration_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Configure_Cluster_Networks_And_Live_Migration_Skeleton
# Purpose:
# Label cluster networks, assign cluster network roles, and configure Hyper-V live migration settings.

$ClusterName = "HVCL01"
$Nodes = @("HV01", "HV02")

# Role values:
# 0 = None
# 1 = Cluster only
# 3 = Cluster and Client
$NetworkPlan = @(
    @{ Match = "192.168.10."; NewName = "Management";    Role = 3 },
    @{ Match = "192.168.20."; NewName = "Cluster";       Role = 1 },
    @{ Match = "192.168.50."; NewName = "LiveMigration"; Role = 1 },
    @{ Match = "192.168.60."; NewName = "SMB01";         Role = 1 },
    @{ Match = "192.168.61."; NewName = "SMB02";         Role = 1 }
)

Write-Host "Current cluster networks:" -ForegroundColor Cyan

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize

Write-Host "Applying cluster network naming and roles..." -ForegroundColor Cyan

foreach ($Network in Get-ClusterNetwork -Cluster $ClusterName) {
    foreach ($Plan in $NetworkPlan) {
        if ($Network.Address -like "$($Plan.Match)*") {
            Write-Host "Configuring cluster network $($Network.Name) -> $($Plan.NewName)" -ForegroundColor Yellow

            $Network.Name = $Plan.NewName
            $Network.Role = $Plan.Role
        }
    }
}

Write-Host "Cluster networks after configuration:" -ForegroundColor Green

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
    Format-Table -AutoSize

Write-Host "Configuring Hyper-V live migration settings on nodes..." -ForegroundColor Cyan

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

Write-Host "Optional: adjust cluster network metrics if live migration chooses the wrong path." -ForegroundColor Yellow

Get-ClusterNetwork -Cluster $ClusterName |
    Select-Object Name, Address, Role, Metric, AutoMetric |
    Format-Table -AutoSize
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_PostChange_Verification_Skeleton

~~~powershell
# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_PostChange_Verification_Skeleton
# Purpose:
# Capture final converged networking state across cluster hosts.

$EvidenceRoot = "C:\Admin\HyperV-ConvergedNetworking"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$Nodes = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "PostChange-$ClusterName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

foreach ($Node in $Nodes) {
    $NodePath = Join-Path $OutputPath $Node
    New-Item -Path $NodePath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing post-change state from $Node..." -ForegroundColor Yellow

    Invoke-Command -ComputerName $Node -ScriptBlock {
        "Physical NICs"
        Get-NetAdapter |
            Sort-Object Name |
            Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverVersion |
            Format-Table -AutoSize

        "Virtual Switches"
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, IovEnabled, BandwidthReservationMode |
            Format-Table -AutoSize

        "SET Teams"
        Get-VMSwitchTeam -ErrorAction SilentlyContinue |
            Format-List *

        "Management OS vNICs"
        Get-VMNetworkAdapter -ManagementOS -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchName, MacAddress, Status, IPAddresses |
            Format-List

        "Management OS vNIC VLANs"
        Get-VMNetworkAdapterVlan -ManagementOS -ErrorAction SilentlyContinue |
            Format-List

        "IP Configuration"
        Get-NetIPConfiguration |
            Format-List

        "Default Routes"
        Get-NetRoute -DestinationPrefix "0.0.0.0/0" -ErrorAction SilentlyContinue |
            Format-Table DestinationPrefix, InterfaceAlias, NextHop, RouteMetric, ifMetric -AutoSize

        "DNS Client Server Addresses"
        Get-DnsClientServerAddress |
            Format-Table InterfaceAlias, AddressFamily, ServerAddresses -AutoSize

        "RDMA"
        Get-NetAdapterRdma -ErrorAction SilentlyContinue |
            Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

        "SMB Client Interfaces"
        Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
            Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize

        "SMB Multichannel Connections"
        Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
            Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRSSCapable, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize

        "QoS Policies"
        Get-NetQosPolicy -ErrorAction SilentlyContinue |
            Format-List *

        "QoS Flow Control"
        Get-NetQosFlowControl -ErrorAction SilentlyContinue |
            Format-Table Priority, Enabled -AutoSize

        "QoS Traffic Classes"
        Get-NetQosTrafficClass -ErrorAction SilentlyContinue |
            Format-Table Name, Priority, Algorithm, BandwidthPercentage -AutoSize

        "Hyper-V Host Migration Settings"
        Get-VMHost -ErrorAction SilentlyContinue |
            Select-Object ComputerName, VirtualMachineMigrationEnabled, VirtualMachineMigrationAuthenticationType, VirtualMachineMigrationPerformanceOption, MaximumVirtualMachineMigrations, MaximumStorageMigrations |
            Format-List

        "Quick Network Counters"
        Get-Counter -Counter "\Network Interface(*)\Bytes Total/sec","\Hyper-V Virtual Switch(*)\Bytes/sec","\SMB Direct Connection(*)\RDMA Inbound Bytes/sec","\SMB Direct Connection(*)\RDMA Outbound Bytes/sec" -SampleInterval 2 -MaxSamples 5 -ErrorAction SilentlyContinue
    } |
    Out-File -FilePath (Join-Path $NodePath "Node-ConvergedNetwork-After.txt") -Encoding UTF8
}

Write-Host "Capturing cluster network state..." -ForegroundColor Cyan

try {
    Get-ClusterNetwork -Cluster $ClusterName -ErrorAction Stop |
        Select-Object Name, Address, AddressMask, Role, Metric, AutoMetric |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "ClusterNetworks-After.txt")

    Get-ClusterNode -Cluster $ClusterName |
        Select-Object Name, State, NodeWeight |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "ClusterNodes-After.txt")

    Get-ClusterGroup -Cluster $ClusterName |
        Select-Object Name, OwnerNode, State, GroupType |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "ClusterGroups-After.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "Cluster-State-After-Error.txt") -Encoding UTF8
}

Write-Host "Capturing event logs..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-FailoverClustering/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "FailoverClustering-Operational-Events.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VmSwitch-Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "HyperV-VmSwitch-Events.txt") -Encoding UTF8

Write-Host "Post-change converged networking evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-NetAdapter` | Each node | Physical NIC state, speed, MAC, and driver |
| `Get-NetAdapterHardwareInfo` | Each node | NIC hardware details where available |
| `Get-NetAdapterAdvancedProperty` | Each node | NIC driver advanced properties |
| `Get-VMSwitch` | Each node | vSwitch inventory and SET state |
| `New-VMSwitch -Name '<switch>' -NetAdapterName '<nic1>','<nic2>' -EnableEmbeddedTeaming $true -AllowManagementOS $true` | Each node | Creates SET vSwitch |
| `Get-VMSwitchTeam` | Each node | SET team members and load balancing |
| `Set-VMSwitchTeam -Name '<switch>' -LoadBalancingAlgorithm Dynamic` | Each node | Configures SET load balancing |
| `Add-VMNetworkAdapter -ManagementOS -Name '<vnic>' -SwitchName '<switch>'` | Each node | Creates host vNIC |
| `Get-VMNetworkAdapter -ManagementOS` | Each node | Shows host vNIC inventory |
| `Set-VMNetworkAdapterVlan -ManagementOS -VMNetworkAdapterName '<vnic>' -Access -VlanId '<id>'` | Each node | Sets host vNIC VLAN |
| `Get-VMNetworkAdapterVlan -ManagementOS` | Each node | Shows host vNIC VLAN state |
| `New-NetIPAddress -InterfaceAlias 'vEthernet (<vnic>)' -IPAddress '<ip>' -PrefixLength '<prefix>'` | Each node | Assigns static IP |
| `Set-DnsClientServerAddress -InterfaceAlias 'vEthernet (Mgmt)' -ServerAddresses '<dns1>','<dns2>'` | Each node | Assigns DNS servers to management vNIC |
| `Get-NetRoute -DestinationPrefix '0.0.0.0/0'` | Each node | Confirms default gateway path |
| `New-NetQosPolicy` | Each node | Creates QoS policy |
| `Get-NetQosPolicy` | Each node | Shows QoS policies |
| `Enable-NetQosFlowControl -Priority '<priority>'` | Each node | Enables PFC for selected priorities |
| `Get-NetQosFlowControl` | Each node | Shows flow control state |
| `New-NetQosTrafficClass -Name '<name>' -Priority '<priority>' -BandwidthPercentage '<percent>' -Algorithm ETS` | Each node | Creates ETS traffic class |
| `Get-NetQosTrafficClass` | Each node | Shows traffic class state |
| `Enable-NetAdapterRdma -Name 'vEthernet (<vnic>)'` | Each node | Enables RDMA on vNIC |
| `Get-NetAdapterRdma` | Each node | Shows RDMA enabled and operational state |
| `Get-SmbClientNetworkInterface` | Each node | Shows SMB Direct capability |
| `Get-SmbMultichannelConnection` | Each node | Shows active SMB multichannel and RDMA sessions |
| `Get-ClusterNetwork` | Cluster node | Shows cluster networks, roles, and metrics |
| `Set-VMHost -VirtualMachineMigrationEnabled $true -VirtualMachineMigrationPerformanceOption SMB` | Each node | Enables SMB-based live migration |
| `Get-VMHost \| Select *Migration*` | Each node | Shows migration settings |
| `Test-NetConnection '<peer-ip>'` | Each node | Confirms node-to-node path |
| `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 20` | Cluster node | Checks cluster network events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 20` | Each node | Checks vSwitch events |

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current state before rollback | Each node | Run post-change verification skeleton | Broken state documented |
| 2 | Restore management access first | Each node console | Validate `vEthernet (Mgmt)` IP, gateway, and DNS | Admin access restored |
| 3 | Disable RDMA on test vNIC if needed | Each node | `Disable-NetAdapterRdma -Name 'vEthernet (<vnic>)'` | RDMA disabled |
| 4 | Remove test QoS policies | Each node | `Get-NetQosPolicy -Name '<name>' \| Remove-NetQosPolicy -Confirm:$false` | QoS policy removed |
| 5 | Remove test traffic classes | Each node | `Get-NetQosTrafficClass -Name '<name>' \| Remove-NetQosTrafficClass -Confirm:$false` | Traffic class removed |
| 6 | Disable selected flow control priorities | Each node | `Disable-NetQosFlowControl -Priority '<priority>'` | PFC disabled for selected priority |
| 7 | Remove non-management host vNIC if rollback requires it | Each node | `Remove-VMNetworkAdapter -ManagementOS -Name '<vnic>'` | Host vNIC removed |
| 8 | Remove SET vSwitch only during maintenance | Each node | `Remove-VMSwitch -Name '<set-switch>' -Force` | SET switch removed |
| 9 | Recreate original vSwitch if required | Each node | `New-VMSwitch -Name '<old-switch>' -NetAdapterName '<old-nic>' -AllowManagementOS $true` | Original switch restored |
| 10 | Restore management IP settings | Each node | `New-NetIPAddress; Set-DnsClientServerAddress` | Host management restored |
| 11 | Restore live migration settings | Each node | `Set-VMHost -VirtualMachineMigrationEnabled '<old-value>' -VirtualMachineMigrationPerformanceOption '<old-value>'` | Migration settings restored |
| 12 | Restore cluster network names and roles | Cluster node | Set `Get-ClusterNetwork` names and role values from baseline | Cluster network state restored |
| 13 | Validate cluster node connectivity | Cluster node | `Get-ClusterNode; Get-ClusterNetwork` | Cluster remains healthy |
| 14 | Capture rollback evidence | Each node | Run post-change verification skeleton | Rollback documented |

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Host loses network after SET creation | Management vNIC IP, VLAN, gateway, or DNS not restored | Use console access and restore management vNIC settings |
| SET switch creation fails | NIC already bound to another vSwitch or wrong NIC name | Remove conflicting vSwitch or correct NIC names |
| SET team degraded | One physical NIC down, cable issue, switch port issue, or driver problem | Check link state, cabling, physical switch, driver, and firmware |
| Host has multiple default gateways | Gateway assigned to non-management vNICs | Remove default routes from cluster, live migration, and SMB vNICs |
| DNS fails after converged setup | DNS not assigned to management vNIC | Set DNS servers only on management vNIC |
| Cluster network names are wrong | Cluster discovered subnets before naming plan | Rename networks based on subnet and assign correct role |
| Cluster heartbeat unstable | Cluster network VLAN or IP path broken | Validate cluster vNIC IPs and node-to-node reachability |
| Live migration uses wrong network | Cluster metrics or migration settings not preferred | Adjust network metrics or live migration network preference |
| Live migration fails | Auth, network, switch, CPU compatibility, or SMB path issue | Review VMMS events, migration settings, and cluster networks |
| SMB Direct not used | RDMA disabled, unsupported, or no SMB traffic | Enable RDMA and generate SMB traffic to validate |
| RDMA operational state is false | Driver, firmware, DCB, or physical switch issue | Fix NIC driver/firmware and DCB configuration |
| RoCE drops traffic | PFC or ETS mismatch on host and physical switch | Align DCB config end to end |
| iWARP path works without DCB but RoCE does not | RoCE requires lossless network design | Configure DCB or use iWARP-capable adapters |
| SMB multichannel missing expected paths | IPs, routes, firewall, or SMB interface capability issue | Check SMB interfaces, IP paths, and multichannel policy |
| VM traffic fails after SET | vSwitch name mismatch, VLAN issue, or uplink trunk issue | Confirm switch name, VM VLAN, and physical switch trunk |
| Cluster CSV redirected mode appears | SMB or storage network issue | Validate SMB Direct, cluster networks, and storage path |
| Node cannot join cluster after networking change | Management or cluster path broken | Restore management IP/DNS and cluster network reachability |
| QoS policy not matching traffic | Wrong template, port, or priority | Confirm policy conditions and traffic type |
| PFC enabled on wrong priority | Priority mismatch between host and physical switch | Align priority values across host and network fabric |
| Rollback removes management path | Removing SET before restoring management adapter | Restore management path before deleting switch or vNICs |

# 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts_Related_Labs

| Lab                                                                           | Relationship                                                                    |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 00_Hyper-V_Index.md                                                           | Places converged cluster networking in the full Hyper-V suite                   |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Confirms host NIC, driver, firmware, CPU, memory, and domain readiness          |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Required before Hyper-V networking cmdlets are available                        |
| 04_Create_And_Configure_Virtual_Switches.md                                   | Base vSwitch design comes before converged SET networking                       |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md                   | VM adapter and VLAN behavior must align with converged host networking          |
| 14_Configure_Live_Migration_And_Storage_Migration.md                          | Live migration uses the converged networking design                             |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md                 | Clustered VMs depend on consistent network configuration across nodes           |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md         | Monitoring validates throughput, RDMA, SMB, vSwitch, and cluster network health |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Troubleshooting uses vSwitch, VLAN, cluster network, RDMA, and VMMS evidence    |
| 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance.md              | Advanced performance features overlap with SET, RDMA, QoS, and vSwitch tuning   |
| 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV.md                    | S2D and CSV rely heavily on converged SMB, RDMA, and cluster networks           |