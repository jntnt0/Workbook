# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Index
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance.md
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Source_Basis
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Mental_Model
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Planning_Table
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Configuration_Checklist
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_PreChange_Capture_Skeleton
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Host_NIC_Capability_Skeleton
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_SET_vSwitch_Skeleton
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_SR-IOV_Configuration_Skeleton
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_RDMA_And_SMB_Direct_Skeleton
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_VMQ_vRSS_vMMQ_Performance_Skeleton
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_PostChange_Verification_Skeleton
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Verification_Commands
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Rollback
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Failure_Checks
20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Related_Labs

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V virtual switch performance | Supports advanced virtual switch tuning and performance validation |
| Microsoft Learn | Switch Embedded Teaming | Supports SET team creation, load balancing, and converged Hyper-V networking |
| Microsoft Learn | SR-IOV for Hyper-V | Supports hardware virtual functions and guest bypass acceleration |
| Microsoft Learn | RDMA and SMB Direct | Supports RDMA validation, SMB Direct testing, and storage network performance |
| Microsoft Learn | VMQ, vRSS, and vMMQ | Supports receive-side scaling and virtual network queue performance tuning |
| Microsoft Learn | Hyper-V PowerShell networking cmdlets | Supports `New-VMSwitch`, `Set-VMSwitch`, `Set-VMNetworkAdapter`, and adapter tuning cmdlets |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Advanced vSwitch performance | Tuning Hyper-V networking beyond basic VM adapter and VLAN settings |
| SET | Switch Embedded Teaming, a Hyper-V virtual switch team using multiple physical NICs |
| Team member | Physical NIC included in a SET virtual switch |
| Converged networking | Multiple traffic types share physical adapters through vNICs and QoS policies |
| SR-IOV | Single Root I/O Virtualization, allowing supported VMs to use physical NIC virtual functions |
| Virtual Function | Hardware-backed NIC path exposed to a VM through SR-IOV |
| RDMA | Remote Direct Memory Access, low-latency network data transfer used by SMB Direct and storage paths |
| SMB Direct | SMB over RDMA for high-throughput low-latency file, CSV, and storage traffic |
| VMQ | Virtual Machine Queue, hardware receive queue assignment for VM traffic |
| vRSS | Virtual Receive Side Scaling, spreads VM network processing across guest vCPUs |
| vMMQ | Virtual Machine Multi-Queue, modern receive-side scaling model for VM network traffic |
| RSS | Receive Side Scaling on the physical NIC and host |
| DCB | Data Center Bridging, QoS model often required for lossless RoCE RDMA designs |
| RoCE | RDMA over Converged Ethernet, often requiring careful QoS and switch configuration |
| iWARP | RDMA transport that usually does not require lossless Ethernet configuration |
| Offload | NIC hardware performs work that would otherwise consume CPU |
| Rollback boundary | This workbook configures host network performance features, not physical switch firmware or enterprise fabric design |

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| Physical NIC 1 | `NIC01` | `<nic-1>` |
| Physical NIC 2 | `NIC02` | `<nic-2>` |
| NIC vendor and model | Mellanox, Intel, Broadcom | `<nic-model>` |
| NIC driver version | `x.x.x.x` | `<driver-version>` |
| Firmware version | `x.x.x` | `<firmware-version>` |
| SET switch name | `vSwitch-SET-Converged` | `<set-switch-name>` |
| SET load balancing algorithm | Dynamic, HyperVPort | `<load-balancing-mode>` |
| Management OS access | Enabled or disabled | `<enabled-disabled>` |
| Management vNIC name | `vEthernet (Mgmt)` | `<management-vnic>` |
| Live migration vNIC name | `vEthernet (LiveMigration)` | `<lm-vnic>` |
| Storage vNIC name | `vEthernet (SMB)` | `<storage-vnic>` |
| VLANs | Management 10, LM 50, SMB 60 | `<vlan-plan>` |
| SR-IOV required | Yes or No | `<yes-no>` |
| SR-IOV switch support | Enabled or disabled | `<enabled-disabled>` |
| VM SR-IOV adapters | `APP01`, `NFV01` | `<vm-list>` |
| RDMA required | Yes or No | `<yes-no>` |
| RDMA transport | RoCE, iWARP, InfiniBand | `<rdma-transport>` |
| DCB required | Yes or No | `<yes-no>` |
| SMB Direct target | `\\SOFS01\VMStorage` | `<smb-target>` |
| VMQ required | Yes or No | `<yes-no>` |
| vRSS required | Yes or No | `<yes-no>` |
| vMMQ required | Yes or No | `<yes-no>` |
| Jumbo frames required | Yes or No | `<yes-no>` |
| MTU size | `1514`, `9014` | `<mtu-size>` |
| Evidence path | `C:\Admin\HyperV-vSwitchPerformance` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm physical NIC inventory | Hyper-V host | `Get-NetAdapter` | Target NICs are visible |
| 3 | Capture NIC driver and firmware state | Hyper-V host | `Get-NetAdapterHardwareInfo; Get-NetAdapterAdvancedProperty` | NIC capability baseline is documented |
| 4 | Confirm SR-IOV support | Hyper-V host | `Get-NetAdapterSriov` | SR-IOV capability is known |
| 5 | Confirm RDMA support | Hyper-V host | `Get-NetAdapterRdma` | RDMA capability is known |
| 6 | Confirm VMQ state | Hyper-V host | `Get-NetAdapterVmq` | VMQ state is known |
| 7 | Confirm RSS state | Hyper-V host | `Get-NetAdapterRss` | RSS state is known |
| 8 | Capture existing vSwitches | Hyper-V host | `Get-VMSwitch` | Existing virtual switches are documented |
| 9 | Capture existing VM adapter performance settings | Hyper-V host | `Get-VMNetworkAdapter` | Adapter state is documented |
| 10 | Create SET switch if required | Hyper-V host | `New-VMSwitch -Name '<set-switch-name>' -NetAdapterName '<nic-1>','<nic-2>' -EnableEmbeddedTeaming $true -AllowManagementOS $true` | SET switch is created |
| 11 | Configure SET load balancing | Hyper-V host | `Set-VMSwitchTeam -Name '<set-switch-name>' -LoadBalancingAlgorithm Dynamic` | SET load balancing mode is configured |
| 12 | Create management OS vNICs if using converged design | Hyper-V host | `Add-VMNetworkAdapter -ManagementOS -Name '<vnic-name>' -SwitchName '<set-switch-name>'` | Host vNICs are created |
| 13 | Configure management OS VLANs | Hyper-V host | `Set-VMNetworkAdapterVlan -ManagementOS -VMNetworkAdapterName '<vnic-name>' -Access -VlanId '<id>'` | Host vNIC VLANs are set |
| 14 | Enable SR-IOV on switch at creation time when required | Hyper-V host | `New-VMSwitch -Name '<switch>' -NetAdapterName '<nic>' -EnableIov $true` | vSwitch supports SR-IOV |
| 15 | Enable SR-IOV on VM adapter | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -IovWeight 100` | VM adapter can use SR-IOV |
| 16 | Enable RDMA on physical adapters | Hyper-V host | `Enable-NetAdapterRdma -Name '<nic-name>'` | RDMA is enabled on NIC |
| 17 | Enable RDMA on management OS vNIC if supported | Hyper-V host | `Enable-NetAdapterRdma -Name 'vEthernet (<vnic-name>)'` | RDMA is enabled on host vNIC |
| 18 | Validate SMB Direct | Hyper-V host | `Get-SmbClientNetworkInterface; Get-SmbMultichannelConnection` | SMB Direct interfaces and sessions are visible |
| 19 | Enable VMQ if required | Hyper-V host | `Enable-NetAdapterVmq -Name '<nic-name>'` | VMQ is enabled |
| 20 | Configure vRSS on VM adapter if required | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -VrssEnabled $true` | vRSS is enabled |
| 21 | Configure vMMQ if supported | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -VmmqEnabled $true` | vMMQ is enabled where supported |
| 22 | Validate advanced adapter state | Hyper-V host | `Get-VMNetworkAdapter -VMName '<vm-name>' \| Format-List *` | Adapter offload state is visible |
| 23 | Run performance validation | Hyper-V host | Use counters and SMB copy tests | Baseline performance is documented |
| 24 | Capture post-change evidence | Hyper-V host | Run post-change verification skeleton | SET, SR-IOV, RDMA, VMQ, vRSS, and vMMQ state are documented |

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_PreChange_Capture_Skeleton

~~~powershell
# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_PreChange_Capture_Skeleton
# Purpose:
# Capture NIC, vSwitch, SR-IOV, RDMA, VMQ, RSS, vRSS, vMMQ, VM adapter, event, and performance state before changes.

$EvidenceRoot = "C:\Admin\HyperV-vSwitchPerformance"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing host identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Host-ComputerInfo.txt")

Write-Host "Capturing Hyper-V feature and services..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperV-Features.txt")

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-HyperV-Services.txt")

Write-Host "Capturing physical NIC inventory..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverInformation, DriverFileName, DriverVersion |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-NetAdapter-Inventory.txt")

Write-Host "Capturing NIC hardware information..." -ForegroundColor Cyan

Get-NetAdapterHardwareInfo -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "05-NetAdapterHardwareInfo.txt")

Write-Host "Capturing NIC advanced properties..." -ForegroundColor Cyan

Get-NetAdapterAdvancedProperty -ErrorAction SilentlyContinue |
    Sort-Object Name, DisplayName |
    Select-Object Name, DisplayName, DisplayValue, RegistryKeyword, RegistryValue |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-NetAdapterAdvancedProperties.txt")

Write-Host "Capturing SR-IOV state..." -ForegroundColor Cyan

Get-NetAdapterSriov -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "07-NetAdapterSriov.txt")

Write-Host "Capturing RDMA state..." -ForegroundColor Cyan

Get-NetAdapterRdma -ErrorAction SilentlyContinue |
    Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-NetAdapterRdma.txt")

Write-Host "Capturing VMQ state..." -ForegroundColor Cyan

Get-NetAdapterVmq -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "09-NetAdapterVmq.txt")

Write-Host "Capturing RSS state..." -ForegroundColor Cyan

Get-NetAdapterRss -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "10-NetAdapterRss.txt")

Write-Host "Capturing vSwitch inventory..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, IovEnabled, BandwidthReservationMode |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "11-VMSwitch-Inventory.txt")

Write-Host "Capturing SET team state..." -ForegroundColor Cyan

Get-VMSwitchTeam -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "12-VMSwitchTeam.txt")

Write-Host "Capturing management OS adapters..." -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "13-ManagementOS-VMNetworkAdapters.txt")

Write-Host "Capturing VM network adapter state..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMNetworkAdapter -VMName $_.Name -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IovWeight, IovQueuePairsRequested, VrssEnabled, VmmqEnabled, DhcpGuard, RouterGuard, MacAddressSpoofing, BandwidthSetting
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "14-VMNetworkAdapters.txt")

Write-Host "Capturing VMSwitch events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VmSwitch-Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "15-HyperV-VmSwitch-Events.txt") -Encoding UTF8

Write-Host "Capturing quick performance counters..." -ForegroundColor Cyan

$Counters = @(
    "\Network Interface(*)\Bytes Total/sec",
    "\Network Interface(*)\Packets/sec",
    "\Hyper-V Virtual Network Adapter(*)\Bytes/sec",
    "\Hyper-V Virtual Network Adapter(*)\Packets/sec",
    "\Hyper-V Virtual Switch(*)\Bytes/sec",
    "\Hyper-V Virtual Switch(*)\Packets/sec",
    "\Processor(_Total)\% Processor Time"
)

Get-Counter -Counter $Counters -SampleInterval 2 -MaxSamples 5 -ErrorAction SilentlyContinue |
    Out-File -FilePath (Join-Path $OutputPath "16-QuickNetworkCounters.txt") -Encoding UTF8

Write-Host "Pre-change advanced vSwitch evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Host_NIC_Capability_Skeleton

~~~powershell
# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Host_NIC_Capability_Skeleton
# Purpose:
# Validate NIC capabilities before configuring SET, SR-IOV, RDMA, VMQ, vRSS, or vMMQ.

$TargetNics = @("NIC01", "NIC02")

Write-Host "Validating target NICs..." -ForegroundColor Cyan

foreach ($Nic in $TargetNics) {
    Get-NetAdapter -Name $Nic -ErrorAction Stop |
        Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverInformation |
        Format-List
}

Write-Host "Checking SR-IOV capability..." -ForegroundColor Cyan

Get-NetAdapterSriov -Name $TargetNics -ErrorAction SilentlyContinue |
    Select-Object Name, SriovSupport, Enabled, NumVFs, NumVFsInUse, IovSupportReasons |
    Format-List

Write-Host "Checking RDMA capability..." -ForegroundColor Cyan

Get-NetAdapterRdma -Name $TargetNics -ErrorAction SilentlyContinue |
    Select-Object Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS |
    Format-Table -AutoSize

Write-Host "Checking VMQ capability..." -ForegroundColor Cyan

Get-NetAdapterVmq -Name $TargetNics -ErrorAction SilentlyContinue |
    Select-Object Name, Enabled, BaseProcessorNumber, MaxProcessors, NumberOfReceiveQueues, MaxProcessorNumber |
    Format-Table -AutoSize

Write-Host "Checking RSS capability..." -ForegroundColor Cyan

Get-NetAdapterRss -Name $TargetNics -ErrorAction SilentlyContinue |
    Select-Object Name, Enabled, BaseProcessorNumber, MaxProcessors, NumberOfReceiveQueues, Profile |
    Format-Table -AutoSize

Write-Host "Checking advanced NIC properties likely relevant to performance..." -ForegroundColor Cyan

Get-NetAdapterAdvancedProperty -Name $TargetNics -ErrorAction SilentlyContinue |
    Where-Object {
        $_.DisplayName -match "Jumbo|MTU|RSS|VMQ|SR-IOV|RDMA|Interrupt|Offload|Checksum|Large Send|Receive|Transmit"
    } |
    Select-Object Name, DisplayName, DisplayValue, RegistryKeyword |
    Sort-Object Name, DisplayName |
    Format-Table -AutoSize

Write-Host "Checking current vSwitch and SET state..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, IovEnabled |
    Format-Table -AutoSize

Get-VMSwitchTeam -ErrorAction SilentlyContinue |
    Format-List *

Write-Host "NIC capability validation completed." -ForegroundColor Green
~~~

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_SET_vSwitch_Skeleton

~~~powershell
# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_SET_vSwitch_Skeleton
# Purpose:
# Create a Switch Embedded Teaming virtual switch and optional management OS vNICs.

$SetSwitchName = "vSwitch-SET-Converged"
$TeamMembers = @("NIC01", "NIC02")
$AllowManagementOS = $true
$LoadBalancingAlgorithm = "Dynamic"

$CreateHostVnics = $true

$HostVnics = @(
    @{ Name = "Mgmt"; VlanId = 10 },
    @{ Name = "LiveMigration"; VlanId = 50 },
    @{ Name = "SMB"; VlanId = 60 }
)

Write-Host "Validating team member NICs..." -ForegroundColor Cyan

foreach ($Nic in $TeamMembers) {
    Get-NetAdapter -Name $Nic -ErrorAction Stop |
        Select-Object Name, Status, LinkSpeed, InterfaceDescription |
        Format-List
}

Write-Host "Checking whether vSwitch already exists..." -ForegroundColor Cyan

if (Get-VMSwitch -Name $SetSwitchName -ErrorAction SilentlyContinue) {
    throw "vSwitch already exists: $SetSwitchName"
}

Write-Host "Creating SET vSwitch: $SetSwitchName" -ForegroundColor Cyan

New-VMSwitch `
    -Name $SetSwitchName `
    -NetAdapterName $TeamMembers `
    -EnableEmbeddedTeaming $true `
    -AllowManagementOS $AllowManagementOS

Write-Host "Configuring SET load balancing algorithm..." -ForegroundColor Cyan

Set-VMSwitchTeam `
    -Name $SetSwitchName `
    -LoadBalancingAlgorithm $LoadBalancingAlgorithm

if ($CreateHostVnics) {
    foreach ($HostVnic in $HostVnics) {
        Write-Host "Creating management OS vNIC: $($HostVnic.Name)" -ForegroundColor Cyan

        if (-not (Get-VMNetworkAdapter -ManagementOS -Name $HostVnic.Name -ErrorAction SilentlyContinue)) {
            Add-VMNetworkAdapter `
                -ManagementOS `
                -Name $HostVnic.Name `
                -SwitchName $SetSwitchName
        }

        Write-Host "Configuring VLAN $($HostVnic.VlanId) on vNIC $($HostVnic.Name)" -ForegroundColor Cyan

        Set-VMNetworkAdapterVlan `
            -ManagementOS `
            -VMNetworkAdapterName $HostVnic.Name `
            -Access `
            -VlanId $HostVnic.VlanId
    }
}

Write-Host "SET vSwitch state after change:" -ForegroundColor Green

Get-VMSwitch -Name $SetSwitchName |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, IovEnabled |
    Format-List

Get-VMSwitchTeam -Name $SetSwitchName |
    Format-List *

Write-Host "Management OS adapters:" -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS |
    Select-Object Name, SwitchName, MacAddress, Status, IPAddresses |
    Format-List

Get-VMNetworkAdapterVlan -ManagementOS |
    Format-List
~~~

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_SR-IOV_Configuration_Skeleton

~~~powershell
# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_SR-IOV_Configuration_Skeleton
# Purpose:
# Configure SR-IOV on a supported vSwitch and VM network adapter.
# SR-IOV must be enabled when the vSwitch is created.

$SwitchName = "vSwitch-SRIOV"
$PhysicalNic = "NIC01"

$VMName = "NFV01"
$AdapterName = "Network Adapter"
$IovWeight = 100
$IovQueuePairsRequested = 4

Write-Host "Checking physical NIC SR-IOV capability..." -ForegroundColor Cyan

Get-NetAdapterSriov -Name $PhysicalNic -ErrorAction Stop |
    Format-List *

Write-Host "Checking existing vSwitch..." -ForegroundColor Cyan

$ExistingSwitch = Get-VMSwitch -Name $SwitchName -ErrorAction SilentlyContinue

if (-not $ExistingSwitch) {
    Write-Host "Creating SR-IOV enabled vSwitch..." -ForegroundColor Cyan

    New-VMSwitch `
        -Name $SwitchName `
        -NetAdapterName $PhysicalNic `
        -EnableIov $true `
        -AllowManagementOS $true
}
else {
    Write-Host "vSwitch already exists. Verify IovEnabled state." -ForegroundColor Yellow
}

Write-Host "vSwitch SR-IOV state:" -ForegroundColor Cyan

Get-VMSwitch -Name $SwitchName |
    Select-Object Name, IovEnabled, IovSupport, IovSupportReasons, NetAdapterInterfaceDescription |
    Format-List

Write-Host "Connecting VM adapter to SR-IOV switch..." -ForegroundColor Cyan

Connect-VMNetworkAdapter `
    -VMName $VMName `
    -Name $AdapterName `
    -SwitchName $SwitchName

Write-Host "Configuring SR-IOV weight and queue pairs..." -ForegroundColor Cyan

Set-VMNetworkAdapter `
    -VMName $VMName `
    -Name $AdapterName `
    -IovWeight $IovWeight `
    -IovQueuePairsRequested $IovQueuePairsRequested

Write-Host "VM adapter SR-IOV state after change:" -ForegroundColor Green

Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName |
    Select-Object VMName, Name, SwitchName, IovWeight, IovQueuePairsRequested, IovQueuePairsAssigned, Status, Connected |
    Format-List

Write-Host "SR-IOV NIC state after change:" -ForegroundColor Cyan

Get-NetAdapterSriov -Name $PhysicalNic |
    Select-Object Name, Enabled, SriovSupport, NumVFs, NumVFsInUse, IovSupportReasons |
    Format-List
~~~

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_RDMA_And_SMB_Direct_Skeleton

~~~powershell
# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_RDMA_And_SMB_Direct_Skeleton
# Purpose:
# Enable RDMA on supported physical NICs or host vNICs and validate SMB Direct.
# Physical switch QoS and DCB requirements must be handled separately for RoCE designs.

$RdmaAdapters = @("NIC01", "NIC02")

$HostRdmaVnics = @(
    "vEthernet (SMB)"
)

$SmbTarget = "\\SOFS01\VMStorage"

Write-Host "Checking RDMA adapters before change..." -ForegroundColor Cyan

Get-NetAdapterRdma -Name $RdmaAdapters -ErrorAction SilentlyContinue |
    Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

Write-Host "Enabling RDMA on physical adapters..." -ForegroundColor Cyan

foreach ($Adapter in $RdmaAdapters) {
    Enable-NetAdapterRdma -Name $Adapter -ErrorAction SilentlyContinue
}

Write-Host "Enabling RDMA on host vNICs if present..." -ForegroundColor Cyan

foreach ($Vnic in $HostRdmaVnics) {
    if (Get-NetAdapter -Name $Vnic -ErrorAction SilentlyContinue) {
        Enable-NetAdapterRdma -Name $Vnic -ErrorAction SilentlyContinue
    }
    else {
        Write-Host "Host vNIC not found: $Vnic" -ForegroundColor Yellow
    }
}

Write-Host "RDMA state after change:" -ForegroundColor Green

Get-NetAdapterRdma -ErrorAction SilentlyContinue |
    Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize

Write-Host "Checking SMB client network interfaces..." -ForegroundColor Cyan

Get-SmbClientNetworkInterface |
    Select-Object InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses |
    Format-Table -AutoSize

Write-Host "Checking SMB server network interfaces if this host serves SMB..." -ForegroundColor Cyan

Get-SmbServerNetworkInterface -ErrorAction SilentlyContinue |
    Select-Object InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddress |
    Format-Table -AutoSize

Write-Host "Testing SMB target path if reachable..." -ForegroundColor Cyan

if (Test-Path $SmbTarget) {
    Get-ChildItem $SmbTarget -ErrorAction SilentlyContinue | Select-Object -First 5

    Write-Host "Creating SMB Direct test file..." -ForegroundColor Cyan

    $TestFile = Join-Path $SmbTarget ("SMBDirectTest-" + $env:COMPUTERNAME + "-" + (Get-Date -Format "yyyyMMdd-HHmmss") + ".txt")

    "SMB Direct test from $env:COMPUTERNAME at $(Get-Date)" |
        Out-File -FilePath $TestFile -Encoding UTF8

    Remove-Item $TestFile -Force
}
else {
    Write-Host "SMB target not reachable or not configured: $SmbTarget" -ForegroundColor Yellow
}

Write-Host "Checking SMB multichannel connections..." -ForegroundColor Cyan

Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
    Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRSSCapable, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize
~~~

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_VMQ_vRSS_vMMQ_Performance_Skeleton

~~~powershell
# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_VMQ_vRSS_vMMQ_Performance_Skeleton
# Purpose:
# Configure VMQ, RSS, vRSS, and vMMQ where supported for advanced VM network performance.

$PhysicalNics = @("NIC01", "NIC02")

$VMName = "APP01"
$AdapterName = "Network Adapter"

$EnableVMQ = $true
$EnableRSS = $true
$EnableVRSS = $true
$EnableVMMQ = $true

$BaseProcessorNumber = 2
$MaxProcessors = 8

Write-Host "Checking current VMQ and RSS state..." -ForegroundColor Cyan

Get-NetAdapterVmq -Name $PhysicalNics -ErrorAction SilentlyContinue |
    Format-List *

Get-NetAdapterRss -Name $PhysicalNics -ErrorAction SilentlyContinue |
    Format-List *

foreach ($Nic in $PhysicalNics) {
    if ($EnableVMQ) {
        Write-Host "Enabling VMQ on $Nic" -ForegroundColor Cyan
        Enable-NetAdapterVmq -Name $Nic -ErrorAction SilentlyContinue

        Write-Host "Configuring VMQ processor range on $Nic" -ForegroundColor Cyan
        Set-NetAdapterVmq `
            -Name $Nic `
            -BaseProcessorNumber $BaseProcessorNumber `
            -MaxProcessors $MaxProcessors `
            -ErrorAction SilentlyContinue
    }
    else {
        Disable-NetAdapterVmq -Name $Nic -ErrorAction SilentlyContinue
    }

    if ($EnableRSS) {
        Write-Host "Enabling RSS on $Nic" -ForegroundColor Cyan
        Enable-NetAdapterRss -Name $Nic -ErrorAction SilentlyContinue

        Set-NetAdapterRss `
            -Name $Nic `
            -BaseProcessorNumber $BaseProcessorNumber `
            -MaxProcessors $MaxProcessors `
            -ErrorAction SilentlyContinue
    }
    else {
        Disable-NetAdapterRss -Name $Nic -ErrorAction SilentlyContinue
    }
}

Write-Host "Configuring VM adapter vRSS and vMMQ..." -ForegroundColor Cyan

$VMAdapter = Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName -ErrorAction Stop

if ($EnableVRSS) {
    Set-VMNetworkAdapter `
        -VMName $VMName `
        -Name $AdapterName `
        -VrssEnabled $true `
        -ErrorAction SilentlyContinue
}
else {
    Set-VMNetworkAdapter `
        -VMName $VMName `
        -Name $AdapterName `
        -VrssEnabled $false `
        -ErrorAction SilentlyContinue
}

if ($EnableVMMQ) {
    Set-VMNetworkAdapter `
        -VMName $VMName `
        -Name $AdapterName `
        -VmmqEnabled $true `
        -ErrorAction SilentlyContinue
}
else {
    Set-VMNetworkAdapter `
        -VMName $VMName `
        -Name $AdapterName `
        -VmmqEnabled $false `
        -ErrorAction SilentlyContinue
}

Write-Host "VMQ and RSS state after change:" -ForegroundColor Green

Get-NetAdapterVmq -Name $PhysicalNics -ErrorAction SilentlyContinue |
    Select-Object Name, Enabled, BaseProcessorNumber, MaxProcessors, NumberOfReceiveQueues, MaxProcessorNumber |
    Format-Table -AutoSize

Get-NetAdapterRss -Name $PhysicalNics -ErrorAction SilentlyContinue |
    Select-Object Name, Enabled, BaseProcessorNumber, MaxProcessors, NumberOfReceiveQueues, Profile |
    Format-Table -AutoSize

Write-Host "VM adapter offload state after change:" -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName |
    Select-Object VMName, Name, SwitchName, VrssEnabled, VmmqEnabled, IovWeight, IovQueuePairsRequested, Status |
    Format-List
~~~

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_PostChange_Verification_Skeleton

~~~powershell
# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_PostChange_Verification_Skeleton
# Purpose:
# Capture final SET, SR-IOV, RDMA, VMQ, RSS, vRSS, vMMQ, SMB Direct, event, and performance evidence.

$EvidenceRoot = "C:\Admin\HyperV-vSwitchPerformance"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing physical NIC state..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, InterfaceDescription, Status, LinkSpeed, MacAddress, DriverInformation, DriverVersion |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-NetAdapter-Inventory-After.txt")

Write-Host "Capturing NIC advanced properties..." -ForegroundColor Cyan

Get-NetAdapterAdvancedProperty -ErrorAction SilentlyContinue |
    Sort-Object Name, DisplayName |
    Select-Object Name, DisplayName, DisplayValue, RegistryKeyword |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-NetAdapterAdvancedProperties-After.txt")

Write-Host "Capturing vSwitch and SET state..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, IovEnabled, BandwidthReservationMode |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMSwitch-After.txt")

Get-VMSwitchTeam -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMSwitchTeam-After.txt")

Write-Host "Capturing SR-IOV state..." -ForegroundColor Cyan

Get-NetAdapterSriov -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "05-NetAdapterSriov-After.txt")

Write-Host "Capturing RDMA state..." -ForegroundColor Cyan

Get-NetAdapterRdma -ErrorAction SilentlyContinue |
    Format-Table Name, InterfaceDescription, Enabled, OperationalState, PFC, ETS -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-NetAdapterRdma-After.txt")

Write-Host "Capturing SMB Direct state..." -ForegroundColor Cyan

Get-SmbClientNetworkInterface -ErrorAction SilentlyContinue |
    Format-Table InterfaceIndex, RSSCapable, RDMA Capable, Speed, IpAddresses -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-SmbClientNetworkInterface-After.txt")

Get-SmbMultichannelConnection -ErrorAction SilentlyContinue |
    Format-Table ServerName, ClientInterfaceIndex, ServerInterfaceIndex, ClientRSSCapable, ClientRdmaCapable, ServerRdmaCapable, CurrentChannels -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-SmbMultichannelConnection-After.txt")

Write-Host "Capturing VMQ and RSS state..." -ForegroundColor Cyan

Get-NetAdapterVmq -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "09-NetAdapterVmq-After.txt")

Get-NetAdapterRss -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "10-NetAdapterRss-After.txt")

Write-Host "Capturing management OS vNIC state..." -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "11-ManagementOS-Adapters-After.txt")

Get-VMNetworkAdapterVlan -ManagementOS -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "12-ManagementOS-VLANs-After.txt")

Write-Host "Capturing VM adapter advanced state..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        Get-VMNetworkAdapter -VMName $_.Name -ErrorAction SilentlyContinue |
            Select-Object VMName, Name, SwitchName, Status, Connected, IovWeight, IovQueuePairsRequested, IovQueuePairsAssigned, VrssEnabled, VmmqEnabled, MacAddressSpoofing, DhcpGuard, RouterGuard, MinimumBandwidthWeight, MaximumBandwidth
    } |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "13-VMNetworkAdapters-Advanced-After.txt")

Write-Host "Capturing quick network performance counters..." -ForegroundColor Cyan

$Counters = @(
    "\Network Interface(*)\Bytes Total/sec",
    "\Network Interface(*)\Packets/sec",
    "\Hyper-V Virtual Network Adapter(*)\Bytes/sec",
    "\Hyper-V Virtual Network Adapter(*)\Packets/sec",
    "\Hyper-V Virtual Switch(*)\Bytes/sec",
    "\Hyper-V Virtual Switch(*)\Packets/sec",
    "\Processor(_Total)\% Processor Time"
)

Get-Counter -Counter $Counters -SampleInterval 2 -MaxSamples 10 -ErrorAction SilentlyContinue |
    Out-File -FilePath (Join-Path $OutputPath "14-QuickNetworkCounters-After.txt") -Encoding UTF8

Write-Host "Capturing Hyper-V VMSwitch events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VmSwitch-Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "15-HyperV-VmSwitch-Events-After.txt") -Encoding UTF8

Write-Host "Capturing System events related to NIC drivers..." -ForegroundColor Cyan

Get-WinEvent -LogName "System" -MaxEvents 150 -ErrorAction SilentlyContinue |
    Where-Object {
        $_.ProviderName -match "net|ndis|tcpip|mlx|intel|broadcom|qlogic|rdma|smb"
    } |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "16-System-NetworkDriver-Events-After.txt") -Encoding UTF8

Write-Host "Post-change advanced vSwitch evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-NetAdapter` | Hyper-V host | Shows physical and virtual NIC inventory |
| `Get-NetAdapterHardwareInfo` | Hyper-V host | Shows hardware capability information where available |
| `Get-NetAdapterAdvancedProperty` | Hyper-V host | Shows NIC driver advanced settings |
| `Get-NetAdapterSriov` | Hyper-V host | Shows SR-IOV support, enabled state, and VF usage |
| `Get-NetAdapterRdma` | Hyper-V host | Shows RDMA enabled and operational state |
| `Get-NetAdapterVmq` | Hyper-V host | Shows VMQ state and processor assignment |
| `Get-NetAdapterRss` | Hyper-V host | Shows RSS state and processor assignment |
| `Get-VMSwitch` | Hyper-V host | Shows vSwitch inventory, SET, and SR-IOV flags |
| `Get-VMSwitchTeam` | Hyper-V host | Shows SET team members and load balancing |
| `New-VMSwitch -Name '<switch>' -NetAdapterName '<nic1>','<nic2>' -EnableEmbeddedTeaming $true` | Hyper-V host | Creates SET vSwitch |
| `Set-VMSwitchTeam -Name '<switch>' -LoadBalancingAlgorithm Dynamic` | Hyper-V host | Configures SET load balancing |
| `New-VMSwitch -Name '<switch>' -NetAdapterName '<nic>' -EnableIov $true` | Hyper-V host | Creates SR-IOV enabled vSwitch |
| `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -IovWeight 100` | Hyper-V host | Enables SR-IOV use for VM adapter |
| `Enable-NetAdapterRdma -Name '<adapter>'` | Hyper-V host | Enables RDMA on supported adapter |
| `Get-SmbClientNetworkInterface` | Hyper-V host | Shows SMB Direct capable client interfaces |
| `Get-SmbServerNetworkInterface` | SMB server | Shows SMB Direct capable server interfaces |
| `Get-SmbMultichannelConnection` | Hyper-V host | Shows SMB multichannel and RDMA session state |
| `Enable-NetAdapterVmq -Name '<adapter>'` | Hyper-V host | Enables VMQ |
| `Set-NetAdapterVmq -Name '<adapter>' -BaseProcessorNumber '<n>' -MaxProcessors '<n>'` | Hyper-V host | Tunes VMQ processor range |
| `Enable-NetAdapterRss -Name '<adapter>'` | Hyper-V host | Enables RSS |
| `Set-NetAdapterRss -Name '<adapter>' -BaseProcessorNumber '<n>' -MaxProcessors '<n>'` | Hyper-V host | Tunes RSS processor range |
| `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -VrssEnabled $true` | Hyper-V host | Enables vRSS for VM adapter |
| `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -VmmqEnabled $true` | Hyper-V host | Enables vMMQ where supported |
| `Get-VMNetworkAdapter -VMName '<vm-name>' \| Format-List *` | Hyper-V host | Shows advanced VM adapter state |
| `Get-Counter '\Hyper-V Virtual Network Adapter(*)\Bytes/sec'` | Hyper-V host | Samples VM adapter throughput |
| `Get-Counter '\Hyper-V Virtual Switch(*)\Bytes/sec'` | Hyper-V host | Samples virtual switch throughput |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 20` | Hyper-V host | Checks virtual switch events |
| `Get-WinEvent -LogName 'System' -MaxEvents 50` | Hyper-V host | Checks NIC driver, NDIS, RDMA, and SMB events |

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture post-issue evidence before rollback | Hyper-V host | Run post-change verification skeleton | Current broken state is documented |
| 2 | Disable vRSS on VM adapter if it caused issues | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -VrssEnabled $false` | vRSS is disabled |
| 3 | Disable vMMQ on VM adapter if it caused issues | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -VmmqEnabled $false` | vMMQ is disabled |
| 4 | Disable SR-IOV use for VM adapter | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -IovWeight 0` | VM no longer requests SR-IOV |
| 5 | Move VM adapter back to non-SR-IOV switch | Hyper-V host | `Connect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter>' -SwitchName '<old-switch>'` | VM uses baseline switch |
| 6 | Disable RDMA on specific adapter if required | Hyper-V host | `Disable-NetAdapterRdma -Name '<adapter>'` | RDMA is disabled on selected adapter |
| 7 | Disable VMQ if driver instability appears | Hyper-V host | `Disable-NetAdapterVmq -Name '<adapter>'` | VMQ is disabled |
| 8 | Disable RSS if required for troubleshooting | Hyper-V host | `Disable-NetAdapterRss -Name '<adapter>'` | RSS is disabled |
| 9 | Restore VMQ processor settings from baseline | Hyper-V host | `Set-NetAdapterVmq -Name '<adapter>' -BaseProcessorNumber '<old>' -MaxProcessors '<old>'` | VMQ processor placement returns to baseline |
| 10 | Restore RSS processor settings from baseline | Hyper-V host | `Set-NetAdapterRss -Name '<adapter>' -BaseProcessorNumber '<old>' -MaxProcessors '<old>'` | RSS processor placement returns to baseline |
| 11 | Remove management OS vNIC if created by mistake | Hyper-V host | `Remove-VMNetworkAdapter -ManagementOS -Name '<vnic-name>'` | Extra host vNIC is removed |
| 12 | Remove SET vSwitch only during maintenance | Hyper-V host | `Remove-VMSwitch -Name '<set-switch-name>' -Force` | SET switch is removed |
| 13 | Recreate original vSwitch | Hyper-V host | `New-VMSwitch -Name '<old-switch>' -NetAdapterName '<old-nic>' -AllowManagementOS $true` | Baseline switch is restored |
| 14 | Restore management OS IP settings | Hyper-V host | `New-NetIPAddress`, `Set-DnsClientServerAddress` | Host management connectivity is restored |
| 15 | Capture rollback evidence | Hyper-V host | Run post-change verification skeleton | Rollback state is documented |

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| SET switch creation fails | NICs already bound, used by another vSwitch, or unsupported | Remove conflicting switch, validate NIC state, use supported adapters |
| Host loses network after SET creation | Management OS access, VLAN, or IP was not restored | Use console access, restore management vNIC VLAN and IP settings |
| SET team shows degraded state | One member down or driver issue | Check `Get-NetAdapter`, cables, switch ports, drivers, and firmware |
| SR-IOV cannot be enabled on existing switch | SR-IOV must be enabled when vSwitch is created | Recreate vSwitch with `-EnableIov $true` during maintenance |
| `Get-NetAdapterSriov` shows unsupported | NIC, BIOS, firmware, or driver does not support SR-IOV | Enable BIOS virtualization and I/O MMU, update firmware and driver, verify NIC support |
| VM does not use SR-IOV | IOV weight is 0, switch not IOV-enabled, or VF unavailable | Set IOV weight, verify switch IOV, check VF capacity |
| SR-IOV breaks VM mobility expectations | Hardware-specific direct path reduces portability | Use standard vSwitch path for VMs requiring broad mobility |
| RDMA enabled but not operational | NIC or driver issue, wrong adapter, DCB issue, or unsupported transport | Check `Get-NetAdapterRdma`, driver, firmware, DCB, and physical switch config |
| SMB Direct not used | RDMA not available on both sides or SMB multichannel path not selected | Check `Get-SmbClientNetworkInterface` and `Get-SmbMultichannelConnection` |
| RoCE traffic drops or performs badly | Missing lossless Ethernet, PFC, ETS, or switch QoS config | Configure DCB consistently on host and physical switches |
| VMQ causes poor performance | Bad queue placement or driver issue | Tune VMQ CPU range or disable VMQ for testing |
| RSS causes poor performance | RSS queue placement conflicts with workload or NUMA | Tune RSS base processor and max processor settings |
| vRSS does not improve guest throughput | Guest OS or adapter does not support it, low vCPU count, or bottleneck elsewhere | Validate guest support, vCPU count, and counters |
| vMMQ setting not accepted | OS, NIC, driver, or VM adapter does not support vMMQ | Update driver or leave vMMQ disabled |
| Jumbo frames break connectivity | MTU mismatch across host, switch, and peer | Set consistent MTU end to end or revert to default MTU |
| VMSwitch event log shows policy errors | Unsupported adapter offload or bad advanced setting | Revert one setting at a time and review event details |
| Live migration fails after performance tuning | SR-IOV, switch mismatch, RDMA, or network design issue | Validate destination switch, adapter settings, and migration compatibility |
| Clustered VM network fails on another node | SET or vSwitch names differ between hosts | Standardize switch names, SET design, VLANs, and host vNICs |
| Storage traffic slows after converged switch changes | QoS, RDMA, DCB, or bandwidth policy issue | Review SMB Direct, QoS, PFC, ETS, and counters |
| Management host becomes unreachable | Incorrect VLAN, host vNIC, or IP configuration | Use out-of-band console and restore management path |

# 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this advanced vSwitch performance task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms physical NIC, driver, firmware, and host readiness |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before vSwitch and VM adapter cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Host defaults support consistent Hyper-V operations |
| 04_Create_And_Configure_Virtual_Switches.md | Basic vSwitch configuration comes before SET, SR-IOV, RDMA, and VMQ tuning |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | CPU and NUMA planning affects RSS, VMQ, and network queue placement |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | VM adapter, VLAN, MAC, and bandwidth settings are the base layer for advanced performance |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Migration design depends on vSwitch, RDMA, SMB Direct, and SET choices |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md | Cluster nodes need consistent vSwitch, SET, RDMA, and live migration networking |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md | Monitoring validates throughput, CPU cost, RDMA use, and VMSwitch health |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Troubleshooting uses SR-IOV, RDMA, SET, VMQ, vRSS, vMMQ, and VMSwitch evidence |
| 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration_For_VMs.md | Hardware passthrough and acceleration planning uses similar compatibility and mobility tradeoffs |