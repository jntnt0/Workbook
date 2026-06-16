# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Index
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Source_Basis
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Mental_Model
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Planning_Table
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Configuration_Checklist
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_PreChange_Capture_Skeleton
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Adapter_Connection_Skeleton
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_VLAN_Configuration_Skeleton
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_MAC_Address_Configuration_Skeleton
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Bandwidth_And_Protection_Skeleton
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_PostChange_Verification_Skeleton
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Verification_Commands
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Rollback
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Failure_Checks
08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Related_Labs

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V virtual network adapter configuration | Supports VM network adapter connection, naming, status, and switch assignment |
| Microsoft Learn | Hyper-V PowerShell `Get-VMNetworkAdapter`, `Set-VMNetworkAdapter`, `Add-VMNetworkAdapter`, `Remove-VMNetworkAdapter` | Supports VM network adapter lifecycle and adapter settings |
| Microsoft Learn | Hyper-V PowerShell `Connect-VMNetworkAdapter` and `Disconnect-VMNetworkAdapter` | Supports connecting and disconnecting VM adapters from virtual switches |
| Microsoft Learn | Hyper-V PowerShell `Set-VMNetworkAdapterVlan` and `Get-VMNetworkAdapterVlan` | Supports access VLAN, trunk VLAN, native VLAN, and untagged configurations |
| Microsoft Learn | Hyper-V bandwidth management | Supports minimum bandwidth weight and maximum bandwidth settings |
| Microsoft Learn | Hyper-V security features for virtual network adapters | Supports DHCP guard, router guard, MAC spoofing, port mirroring, and protected network behavior |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VM network adapter | The virtual NIC presented to the guest VM |
| Virtual switch | The Hyper-V switching fabric the VM adapter connects to |
| Access VLAN | A single VLAN assigned to a VM adapter |
| Trunk VLAN | Multiple VLANs allowed through one VM adapter, commonly for routers, firewalls, and nested labs |
| Native VLAN | Untagged VLAN used on a trunk configuration |
| Untagged adapter | VM adapter sends traffic without VLAN tagging |
| Dynamic MAC | Hyper-V automatically assigns the VM adapter MAC address |
| Static MAC | Administrator manually assigns the VM adapter MAC address |
| MAC spoofing | Allows the VM to send traffic using a MAC address other than the assigned adapter MAC |
| DHCP guard | Blocks rogue DHCP server traffic from a VM |
| Router guard | Blocks router advertisement traffic from a VM |
| Port mirroring | Copies traffic to or from a VM adapter for inspection |
| Protected network | Allows Hyper-V to react when network connectivity is lost in clustering or protected scenarios |
| Minimum bandwidth weight | Relative bandwidth priority when multiple adapters compete |
| Maximum bandwidth | Upper bandwidth cap applied to the VM adapter |
| Adapter name | Friendly VM-side adapter object name used by Hyper-V PowerShell |
| Rollback boundary | This workbook changes VM network adapter settings, not physical switch configuration or guest IP configuration |

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| Adapter name | `Network Adapter`, `ProdNIC`, `LabNIC` | `<adapter-name>` |
| Switch name | `vSwitch-External`, `vSwitch-Internal-Lab` | `<switch-name>` |
| Adapter purpose | Production, management, lab, backup, nested router | `<adapter-purpose>` |
| VLAN mode | Access, trunk, untagged | `<vlan-mode>` |
| Access VLAN ID | `20` | `<access-vlan-id>` |
| Trunk allowed VLANs | `10,20,30` | `<allowed-vlan-list>` |
| Native VLAN ID | `10` | `<native-vlan-id>` |
| MAC assignment | Dynamic or static | `<mac-mode>` |
| Static MAC address | `00155D010101` | `<static-mac>` |
| MAC spoofing | On or Off | `<on-off>` |
| DHCP guard | On or Off | `<on-off>` |
| Router guard | On or Off | `<on-off>` |
| Port mirroring | None, Source, Destination | `<port-mirroring-mode>` |
| Protected network | On or Off | `<on-off>` |
| Device naming | On or Off | `<on-off>` |
| Minimum bandwidth weight | `10`, `50`, `100` | `<minimum-bandwidth-weight>` |
| Maximum bandwidth | `1000000000` | `<maximum-bandwidth-bps>` |
| Evidence path | `C:\Admin\HyperV-VMNetworkAdapters` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | Target VM is found |
| 3 | Confirm VM state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, State` | VM state is known |
| 4 | Confirm target virtual switch exists | Hyper-V host | `Get-VMSwitch -Name '<switch-name>'` | Intended virtual switch exists |
| 5 | Capture existing VM adapter configuration | Hyper-V host | `Get-VMNetworkAdapter -VMName '<vm-name>'` | Existing adapter state is documented |
| 6 | Capture existing VLAN configuration | Hyper-V host | `Get-VMNetworkAdapterVlan -VMName '<vm-name>'` | Existing VLAN state is documented |
| 7 | Add a VM network adapter if required | Hyper-V host | `Add-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -SwitchName '<switch-name>'` | New VM network adapter is created and connected |
| 8 | Connect existing adapter to switch | Hyper-V host | `Connect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -SwitchName '<switch-name>'` | Adapter is connected to intended switch |
| 9 | Rename adapter if needed | Hyper-V host | `Rename-VMNetworkAdapter -VMName '<vm-name>' -Name '<old-name>' -NewName '<new-name>'` | Adapter name matches the plan |
| 10 | Configure access VLAN | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Access -VlanId 20` | Adapter sends traffic on the access VLAN |
| 11 | Configure trunk VLAN if required | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Trunk -AllowedVlanIdList '10,20,30' -NativeVlanId 10` | Adapter allows multiple VLANs |
| 12 | Configure untagged adapter if required | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Untagged` | Adapter sends untagged traffic |
| 13 | Configure dynamic MAC address | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -DynamicMacAddress` | Hyper-V assigns adapter MAC dynamically |
| 14 | Configure static MAC address | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -StaticMacAddress '00155D010101'` | Adapter uses planned static MAC |
| 15 | Configure MAC spoofing if required | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -MacAddressSpoofing On` | Adapter can send traffic from alternate MACs |
| 16 | Configure DHCP guard | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -DhcpGuard On` | Rogue DHCP from VM is blocked |
| 17 | Configure router guard | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -RouterGuard On` | Router advertisement traffic from VM is blocked |
| 18 | Configure port mirroring if required | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -PortMirroring Source` | Adapter traffic is mirrored according to plan |
| 19 | Configure bandwidth policy | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -MinimumBandwidthWeight 50 -MaximumBandwidth 1000000000` | Bandwidth priority and cap are set |
| 20 | Enable resource metering if required | Hyper-V host | `Enable-VMResourceMetering -VMName '<vm-name>'` | VM resource metering is enabled |
| 21 | Verify final adapter state | Hyper-V host | `Get-VMNetworkAdapter -VMName '<vm-name>'; Get-VMNetworkAdapterVlan -VMName '<vm-name>'` | Adapter, VLAN, MAC, and bandwidth match the plan |
| 22 | Capture post-change evidence | Hyper-V host | Run post-change verification skeleton | Evidence confirms final VM network adapter state |

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_PreChange_Capture_Skeleton

~~~powershell
# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_PreChange_Capture_Skeleton
# Purpose:
# Capture VM network adapters, VLAN settings, virtual switches, host adapters, and VM state before changes.

$EvidenceRoot = "C:\Admin\HyperV-VMNetworkAdapters"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Generation, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.txt")

Write-Host "Capturing VM network adapter configuration..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMNetworkAdapters-Before.txt")

Get-VMNetworkAdapter -VMName $VMName |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "02-VMNetworkAdapters-Before.json") -Encoding UTF8

Write-Host "Capturing VM network adapter VLAN configuration..." -ForegroundColor Cyan

Get-VMNetworkAdapterVlan -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMNetworkAdapterVlan-Before.txt")

Write-Host "Capturing virtual switch inventory..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMSwitches-Before.txt")

Write-Host "Capturing host network adapters..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-HostNetAdapters-Before.txt")

Write-Host "Capturing management OS network adapters..." -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS |
    Select-Object Name, SwitchName, MacAddress, Status, IsManagementOs |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-ManagementOSAdapters-Before.txt")

Write-Host "Capturing recent Hyper-V switch events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VmSwitch-Operational" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-VmSwitch-RecentEvents-Before.txt") -Encoding UTF8

Write-Host "Pre-change VM network adapter evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Adapter_Connection_Skeleton

~~~powershell
# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Adapter_Connection_Skeleton
# Purpose:
# Add, connect, disconnect, or rename a VM network adapter.

$VMName = "LAB-WIN-SRV01"
$AdapterName = "ProdNIC"
$SwitchName = "vSwitch-External"

$CreateAdapterIfMissing = $true
$ConnectAdapterToSwitch = $true

Write-Host "Validating VM and switch..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop
$Switch = Get-VMSwitch -Name $SwitchName -ErrorAction Stop

Write-Host "Checking existing VM network adapter..." -ForegroundColor Cyan

$Adapter = Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName -ErrorAction SilentlyContinue

if (-not $Adapter -and $CreateAdapterIfMissing) {
    Write-Host "Creating VM network adapter: $AdapterName" -ForegroundColor Cyan

    Add-VMNetworkAdapter `
        -VMName $VMName `
        -Name $AdapterName `
        -SwitchName $SwitchName
}
elseif (-not $Adapter) {
    throw "Adapter does not exist and CreateAdapterIfMissing is false: $AdapterName"
}
else {
    Write-Host "Adapter already exists: $AdapterName" -ForegroundColor Yellow
}

if ($ConnectAdapterToSwitch) {
    Write-Host "Connecting adapter to switch: $SwitchName" -ForegroundColor Cyan

    Connect-VMNetworkAdapter `
        -VMName $VMName `
        -Name $AdapterName `
        -SwitchName $SwitchName
}

Write-Host "Adapter connection state after change:" -ForegroundColor Green

Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, DynamicMacAddressEnabled |
    Format-List
~~~

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_VLAN_Configuration_Skeleton

~~~powershell
# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_VLAN_Configuration_Skeleton
# Purpose:
# Configure access, trunk, or untagged VLAN mode for a VM network adapter.

$VMName = "LAB-WIN-SRV01"
$AdapterName = "ProdNIC"

# Valid modes:
# Access
# Trunk
# Untagged
$VlanMode = "Access"

$AccessVlanId = 20
$AllowedVlanIdList = "10,20,30"
$NativeVlanId = 10

Write-Host "Validating VM network adapter..." -ForegroundColor Cyan

$Adapter = Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName -ErrorAction Stop

Write-Host "Configuring VLAN mode: $VlanMode" -ForegroundColor Cyan

switch ($VlanMode) {
    "Access" {
        Set-VMNetworkAdapterVlan `
            -VMName $VMName `
            -VMNetworkAdapterName $AdapterName `
            -Access `
            -VlanId $AccessVlanId
    }

    "Trunk" {
        Set-VMNetworkAdapterVlan `
            -VMName $VMName `
            -VMNetworkAdapterName $AdapterName `
            -Trunk `
            -AllowedVlanIdList $AllowedVlanIdList `
            -NativeVlanId $NativeVlanId
    }

    "Untagged" {
        Set-VMNetworkAdapterVlan `
            -VMName $VMName `
            -VMNetworkAdapterName $AdapterName `
            -Untagged
    }

    default {
        throw "Unsupported VLAN mode: $VlanMode. Use Access, Trunk, or Untagged."
    }
}

Write-Host "VLAN configuration after change:" -ForegroundColor Green

Get-VMNetworkAdapterVlan -VMName $VMName -VMNetworkAdapterName $AdapterName |
    Format-List

Write-Host "Adapter switch connection:" -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected |
    Format-List
~~~

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_MAC_Address_Configuration_Skeleton

~~~powershell
# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_MAC_Address_Configuration_Skeleton
# Purpose:
# Configure dynamic or static MAC address assignment and MAC spoofing.

$VMName = "LAB-WIN-SRV01"
$AdapterName = "ProdNIC"

# Valid modes:
# Dynamic
# Static
$MacMode = "Dynamic"

$StaticMacAddress = "00155D010101"
$MacAddressSpoofing = "Off"

Write-Host "Validating VM network adapter..." -ForegroundColor Cyan

$Adapter = Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName -ErrorAction Stop

Write-Host "Configuring MAC mode: $MacMode" -ForegroundColor Cyan

switch ($MacMode) {
    "Dynamic" {
        Set-VMNetworkAdapter `
            -VMName $VMName `
            -Name $AdapterName `
            -DynamicMacAddress
    }

    "Static" {
        Set-VMNetworkAdapter `
            -VMName $VMName `
            -Name $AdapterName `
            -StaticMacAddress $StaticMacAddress
    }

    default {
        throw "Unsupported MAC mode: $MacMode. Use Dynamic or Static."
    }
}

Write-Host "Configuring MAC spoofing: $MacAddressSpoofing" -ForegroundColor Cyan

Set-VMNetworkAdapter `
    -VMName $VMName `
    -Name $AdapterName `
    -MacAddressSpoofing $MacAddressSpoofing

Write-Host "MAC configuration after change:" -ForegroundColor Green

Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName |
    Select-Object VMName, Name, SwitchName, MacAddress, DynamicMacAddressEnabled, MacAddressSpoofing, Status |
    Format-List
~~~

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Bandwidth_And_Protection_Skeleton

~~~powershell
# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Bandwidth_And_Protection_Skeleton
# Purpose:
# Configure bandwidth controls and common VM network adapter protection settings.

$VMName = "LAB-WIN-SRV01"
$AdapterName = "ProdNIC"

$MinimumBandwidthWeight = 50
$MaximumBandwidth = 1000000000

$DhcpGuard = "On"
$RouterGuard = "On"
$PortMirroring = "None"
$ProtectedNetwork = "On"
$DeviceNaming = "On"

Write-Host "Validating VM network adapter..." -ForegroundColor Cyan

$Adapter = Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName -ErrorAction Stop

Write-Host "Configuring bandwidth policy..." -ForegroundColor Cyan

Set-VMNetworkAdapter `
    -VMName $VMName `
    -Name $AdapterName `
    -MinimumBandwidthWeight $MinimumBandwidthWeight `
    -MaximumBandwidth $MaximumBandwidth

Write-Host "Configuring protection features..." -ForegroundColor Cyan

Set-VMNetworkAdapter `
    -VMName $VMName `
    -Name $AdapterName `
    -DhcpGuard $DhcpGuard `
    -RouterGuard $RouterGuard `
    -PortMirroring $PortMirroring `
    -ProtectedNetwork $ProtectedNetwork `
    -DeviceNaming $DeviceNaming

Write-Host "Bandwidth and protection configuration after change:" -ForegroundColor Green

Get-VMNetworkAdapter -VMName $VMName -Name $AdapterName |
    Select-Object `
        VMName,
        Name,
        SwitchName,
        Status,
        MinimumBandwidthWeight,
        MaximumBandwidth,
        DhcpGuard,
        RouterGuard,
        PortMirroring,
        ProtectedNetwork,
        DeviceNaming,
        MacAddressSpoofing |
    Format-List
~~~

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_PostChange_Verification_Skeleton

~~~powershell
# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_PostChange_Verification_Skeleton
# Purpose:
# Verify VM adapter connection, VLAN mode, MAC mode, protection settings, bandwidth settings, and recent events after changes.

$EvidenceRoot = "C:\Admin\HyperV-VMNetworkAdapters"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM summary after network adapter changes..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-After.txt")

Write-Host "Capturing VM network adapter configuration after changes..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMNetworkAdapters-After.txt")

Get-VMNetworkAdapter -VMName $VMName |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "02-VMNetworkAdapters-After.json") -Encoding UTF8

Write-Host "Capturing VLAN configuration after changes..." -ForegroundColor Cyan

Get-VMNetworkAdapterVlan -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMNetworkAdapterVlan-After.txt")

Write-Host "Capturing switch inventory after changes..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMSwitches-After.txt")

Write-Host "Capturing VM network adapter ACLs if present..." -ForegroundColor Cyan

Get-VMNetworkAdapterAcl -VMName $VMName -ErrorAction SilentlyContinue |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMNetworkAdapterAcl-After.txt")

Write-Host "Capturing VM network adapter routing guards and features through full adapter dump..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
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
        DeviceNaming,
        MinimumBandwidthWeight,
        MaximumBandwidth,
        Status,
        Connected |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-AdapterFeatureSummary-After.txt")

Write-Host "Capturing resource metering state if enabled..." -ForegroundColor Cyan

Measure-VM -Name $VMName -ErrorAction SilentlyContinue |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "07-MeasureVM-After.txt")

Write-Host "Capturing recent Hyper-V switch events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VmSwitch-Operational" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-HyperV-VmSwitch-RecentEvents-After.txt") -Encoding UTF8

Write-Host "Capturing recent Hyper-V VMMS events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-HyperV-VMMS-RecentEvents-After.txt") -Encoding UTF8

Write-Host "Post-change VM network adapter evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms VM exists and shows state |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Shows VM adapter inventory |
| `Get-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' \| Format-List *` | Hyper-V host | Shows full adapter configuration |
| `Get-VMNetworkAdapterVlan -VMName '<vm-name>'` | Hyper-V host | Shows VLAN configuration for all VM adapters |
| `Get-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>'` | Hyper-V host | Confirms access, trunk, native, or untagged mode |
| `Get-VMSwitch` | Hyper-V host | Confirms available virtual switches |
| `Get-VMNetworkAdapter -VMName '<vm-name>' \| Select Name,SwitchName,MacAddress,Status,Connected` | Hyper-V host | Confirms switch connection and MAC address |
| `Get-VMNetworkAdapter -VMName '<vm-name>' \| Select Name,DynamicMacAddressEnabled,MacAddressSpoofing` | Hyper-V host | Confirms dynamic or static MAC and spoofing state |
| `Get-VMNetworkAdapter -VMName '<vm-name>' \| Select Name,DhcpGuard,RouterGuard,PortMirroring,ProtectedNetwork` | Hyper-V host | Confirms protection features |
| `Get-VMNetworkAdapter -VMName '<vm-name>' \| Select Name,MinimumBandwidthWeight,MaximumBandwidth` | Hyper-V host | Confirms bandwidth settings |
| `Measure-VM -Name '<vm-name>'` | Hyper-V host | Shows resource metering data if enabled |
| `Get-VMNetworkAdapterAcl -VMName '<vm-name>'` | Hyper-V host | Shows adapter ACLs if configured |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 20` | Hyper-V host | Checks recent virtual switch events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks recent VM management network events |

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Review pre-change adapter state | Hyper-V host | `Get-Content '<prechange-path>\02-VMNetworkAdapters-Before.txt'` | Original adapter settings are identified |
| 2 | Review pre-change VLAN state | Hyper-V host | `Get-Content '<prechange-path>\03-VMNetworkAdapterVlan-Before.txt'` | Original VLAN settings are identified |
| 3 | Disconnect adapter if wrong switch was assigned | Hyper-V host | `Disconnect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>'` | Adapter is disconnected from the wrong switch |
| 4 | Reconnect adapter to original switch | Hyper-V host | `Connect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -SwitchName '<old-switch-name>'` | Adapter is connected to original switch |
| 5 | Restore untagged VLAN mode | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Untagged` | VLAN tagging is removed |
| 6 | Restore access VLAN mode | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Access -VlanId '<old-vlan-id>'` | Access VLAN is restored |
| 7 | Restore trunk VLAN mode | Hyper-V host | `Set-VMNetworkAdapterVlan -VMName '<vm-name>' -VMNetworkAdapterName '<adapter-name>' -Trunk -AllowedVlanIdList '<old-list>' -NativeVlanId '<old-native-vlan>'` | Trunk VLAN settings are restored |
| 8 | Restore dynamic MAC | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -DynamicMacAddress` | Adapter returns to dynamic MAC assignment |
| 9 | Restore static MAC | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -StaticMacAddress '<old-static-mac>'` | Adapter returns to original static MAC |
| 10 | Restore protection settings | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -DhcpGuard Off -RouterGuard Off -MacAddressSpoofing Off -PortMirroring None` | Protection settings return to baseline |
| 11 | Restore bandwidth settings | Hyper-V host | `Set-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -MinimumBandwidthWeight 0 -MaximumBandwidth 0` | Bandwidth settings are cleared or reset |
| 12 | Remove newly added adapter if rollback requires it | Hyper-V host | `Remove-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>'` | Extra adapter is removed |
| 13 | Capture rollback evidence | Hyper-V host | `Get-VMNetworkAdapter -VMName '<vm-name>'; Get-VMNetworkAdapterVlan -VMName '<vm-name>'` | Rollback state is documented |

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Adapter is not connected | Adapter was never connected or was disconnected from switch | Run `Connect-VMNetworkAdapter` with the correct switch name |
| Switch not found | Wrong switch name or switch was not created | Run `Get-VMSwitch` and use exact switch name |
| VM has no network after VLAN change | VLAN mismatch between VM adapter, virtual switch uplink, and physical switchport | Confirm access VLAN, trunk VLAN list, native VLAN, and physical switchport config |
| Trunked VM cannot reach expected VLANs | Allowed VLAN list missing required VLANs | Update `AllowedVlanIdList` on the VM adapter |
| Untagged VM cannot reach network | Physical switchport expects tagged traffic | Configure access VLAN on VM adapter or adjust physical port design |
| Static MAC causes duplicate MAC issue | MAC address reused on another VM or device | Assign unique static MAC or return to dynamic MAC |
| Guest OS still shows old MAC | Guest cache or adapter reset needed | Disable and enable guest NIC or restart VM |
| MAC spoofing required workload fails | MAC spoofing is Off | Enable MAC spoofing for nested virtualization, NLB, firewall, router, or appliance scenarios |
| Rogue DHCP lab server cannot hand out leases | DHCP guard is On | Turn DHCP guard Off only for approved DHCP server VMs |
| Router appliance does not route | Router guard is On | Turn router guard Off only for approved router or firewall VMs |
| Bandwidth cap throttles VM too hard | MaximumBandwidth value too low | Increase or clear maximum bandwidth |
| Minimum bandwidth setting appears ineffective | Switch minimum bandwidth mode or contention scenario does not use the setting as expected | Confirm virtual switch design and test under contention |
| Port mirroring destination sees no traffic | Source and destination mirroring roles are not configured correctly | Configure one adapter as Source and one as Destination |
| VM network works on one host but not another | Switch names or VLAN settings differ across hosts | Standardize switch names, VLANs, and adapter settings |
| Adapter rename breaks scripts | Scripts reference old adapter name | Update scripts or rename adapter back |
| VM loses network after connecting to wrong switch | Adapter connected to internal or private switch by mistake | Reconnect to correct external or lab switch |
| Resource metering returns no data | Resource metering not enabled | Run `Enable-VMResourceMetering -VMName '<vm-name>'` |
| Event log shows VMSwitch policy errors | Invalid adapter policy or unsupported feature combination | Review recent Hyper-V VMSwitch events and simplify adapter settings |

# 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this VM network adapter task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host NIC, IP, DNS, and physical network baseline |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before Hyper-V networking cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Establishes host defaults before VM creation and adapter configuration |
| 04_Create_And_Configure_Virtual_Switches.md | Provides the virtual switches used by VM adapters |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates the VMs whose adapters are configured here |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | Complements network settings with VM compute settings |
| 09_Configure_Hyper-V_NAT_Internal_And_Private_Lab_Networks.md | Uses internal and private adapter configurations for NAT lab networks |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md | Uses working VM networking as one guest management path |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Depends on consistent host and VM network design |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses adapter, VLAN, MAC, and bandwidth state during VM network troubleshooting |