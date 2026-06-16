# 04_Create_And_Configure_Virtual_Switches

# 04_Create_And_Configure_Virtual_Switches_Index
04_Create_And_Configure_Virtual_Switches.md
04_Create_And_Configure_Virtual_Switches
04_Create_And_Configure_Virtual_Switches_Source_Basis
04_Create_And_Configure_Virtual_Switches_Mental_Model
04_Create_And_Configure_Virtual_Switches_Planning_Table
04_Create_And_Configure_Virtual_Switches_Configuration_Checklist
04_Create_And_Configure_Virtual_Switches_PreChange_Capture_Skeleton
04_Create_And_Configure_Virtual_Switches_External_Switch_Skeleton
04_Create_And_Configure_Virtual_Switches_Internal_Private_Switch_Skeleton
04_Create_And_Configure_Virtual_Switches_Management_OS_VLAN_Skeleton
04_Create_And_Configure_Virtual_Switches_PostChange_Verification_Skeleton
04_Create_And_Configure_Virtual_Switches_Verification_Commands
04_Create_And_Configure_Virtual_Switches_Rollback
04_Create_And_Configure_Virtual_Switches_Failure_Checks
04_Create_And_Configure_Virtual_Switches_Related_Labs

# 04_Create_And_Configure_Virtual_Switches_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V virtual switch overview | Supports external, internal, and private switch behavior |
| Microsoft Learn | Hyper-V networking | Supports binding virtual switches to physical adapters |
| Microsoft Learn | Hyper-V PowerShell `New-VMSwitch` | Supports switch creation by PowerShell |
| Microsoft Learn | Hyper-V PowerShell `Get-VMSwitch`, `Set-VMSwitch`, `Remove-VMSwitch` | Supports verification, modification, and rollback |
| Microsoft Learn | Hyper-V VLAN configuration | Supports management OS and VM network adapter VLAN configuration |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 04_Create_And_Configure_Virtual_Switches_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Hyper-V virtual switch | Software switch used to connect VMs, the management OS, and physical networks |
| External switch | Binds to a physical NIC and allows VM traffic to reach the physical network |
| Internal switch | Allows communication between VMs and the Hyper-V host, but not directly to the physical network |
| Private switch | Allows communication between VMs only, not the host or physical network |
| Management OS sharing | Allows the host operating system to keep using the physical NIC after it is bound to an external switch |
| Physical NIC binding | The external switch takes control of the selected physical adapter |
| vEthernet adapter | Virtual adapter created in the management OS when the host participates in a virtual switch |
| VLAN ID | Logical network tag used to separate traffic on the same physical link |
| Switch naming | Clear switch names prevent mistakes when assigning VM network adapters |
| Dedicated uplink | Physical NIC reserved for VM traffic or shared host and VM traffic |
| Pre-change capture | Inventory of physical adapters, IP settings, routes, DNS, and existing switches before changes |
| Rollback boundary | Removing a switch can disconnect VMs or host management if the wrong adapter is used |

# 04_Create_And_Configure_Virtual_Switches_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| External switch name | `vSwitch-External` | `<external-switch-name>` |
| External uplink adapter | `Ethernet 2` or `VM-Uplink` | `<physical-uplink-adapter>` |
| Allow management OS on external switch | Yes or No | `<yes-no>` |
| Management OS vEthernet name | `vEthernet (vSwitch-External)` | `<management-vethernet-name>` |
| Management VLAN ID | `10` | `<management-vlan-id>` |
| VM production VLAN ID | `20` | `<vm-vlan-id>` |
| Lab VLAN ID | `30` | `<lab-vlan-id>` |
| Internal switch name | `vSwitch-Internal-Lab` | `<internal-switch-name>` |
| Private switch name | `vSwitch-Private-Isolated` | `<private-switch-name>` |
| Internal switch host IP | `172.16.10.1/24` | `<internal-host-ip>` |
| Physical switchport mode | Access or trunk | `<switchport-mode>` |
| Physical switchport VLANs | `10,20,30` | `<allowed-vlans>` |
| NIC rename plan | `MGMT`, `VM-UPLINK` | `<nic-name-plan>` |
| Change evidence path | `C:\Admin\HyperV-Switches` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 04_Create_And_Configure_Virtual_Switches_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm Hyper-V PowerShell module is available | Hyper-V host | `Get-Module -ListAvailable Hyper-V` | Hyper-V module exists |
| 3 | Capture physical NIC inventory | Hyper-V host | `Get-NetAdapter \| Format-Table Name,Status,LinkSpeed,MacAddress,InterfaceDescription` | NIC names, link state, and uplinks are documented |
| 4 | Capture current IP configuration | Hyper-V host | `Get-NetIPConfiguration` | Management IP, DNS, gateway, and interface mapping are documented |
| 5 | Capture existing virtual switches | Hyper-V host | `Get-VMSwitch` | Existing switches are documented |
| 6 | Rename physical NICs if needed | Hyper-V host | `Rename-NetAdapter -Name '<old-name>' -NewName 'VM-UPLINK'` | Adapter names match the plan |
| 7 | Confirm uplink adapter is not the only remote path unless approved | Hyper-V host | `Get-NetIPConfiguration -InterfaceAlias '<physical-uplink-adapter>'` | Risk of host management loss is understood |
| 8 | Create external virtual switch | Hyper-V host | `New-VMSwitch -Name 'vSwitch-External' -NetAdapterName 'VM-UPLINK' -AllowManagementOS $true` | External switch is created and bound to physical NIC |
| 9 | Confirm external switch | Hyper-V host | `Get-VMSwitch -Name 'vSwitch-External'` | Switch type shows external |
| 10 | Configure management OS VLAN if trunking is used | Hyper-V host | `Set-VMNetworkAdapterVlan -ManagementOS -VMNetworkAdapterName 'vSwitch-External' -Access -VlanId 10` | Host vEthernet adapter uses the correct VLAN |
| 11 | Confirm management OS vEthernet adapter exists | Hyper-V host | `Get-NetAdapter \| Where-Object Name -like 'vEthernet*'` | vEthernet adapter is present |
| 12 | Confirm host IP remains reachable | Hyper-V host or admin workstation | `Test-NetConnection <host-ip>` | Management connectivity still works |
| 13 | Create internal lab switch if required | Hyper-V host | `New-VMSwitch -Name 'vSwitch-Internal-Lab' -SwitchType Internal` | Internal switch exists |
| 14 | Assign host IP to internal vEthernet adapter if required | Hyper-V host | `New-NetIPAddress -InterfaceAlias 'vEthernet (vSwitch-Internal-Lab)' -IPAddress 172.16.10.1 -PrefixLength 24` | Host can communicate on internal lab network |
| 15 | Create private isolated switch if required | Hyper-V host | `New-VMSwitch -Name 'vSwitch-Private-Isolated' -SwitchType Private` | Private switch exists |
| 16 | Confirm all switches | Hyper-V host | `Get-VMSwitch \| Format-Table Name,SwitchType,NetAdapterInterfaceDescription,AllowManagementOS` | Switch inventory matches the plan |
| 17 | Confirm switch network adapters | Hyper-V host | `Get-VMNetworkAdapter -ManagementOS` | Management OS adapters are visible |
| 18 | Capture final switch configuration | Hyper-V host | Run post-change verification skeleton | Evidence confirms virtual switch configuration |

# 04_Create_And_Configure_Virtual_Switches_PreChange_Capture_Skeleton

~~~powershell
# 04_Create_And_Configure_Virtual_Switches_PreChange_Capture_Skeleton
# Purpose:
# Capture physical NICs, IP configuration, routes, DNS, firewall profile state, and existing Hyper-V switches before changes.

$EvidenceRoot = "C:\Admin\HyperV-Switches"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing Hyper-V feature and module state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-HyperVFeatureState.txt")

Get-Module -ListAvailable Hyper-V |
    Select-Object Name, Version, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperVModule.txt")

Write-Host "Capturing physical network adapters..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription, DriverInformation |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-NetAdapters-Before.txt")

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription, DriverInformation |
    Export-Csv -Path (Join-Path $OutputPath "03-NetAdapters-Before.csv") -NoTypeInformation

Write-Host "Capturing IP configuration..." -ForegroundColor Cyan

Get-NetIPConfiguration |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "04-NetIPConfiguration-Before.txt") -Encoding UTF8

Get-DnsClientServerAddress |
    Export-Csv -Path (Join-Path $OutputPath "05-DnsClientServerAddress-Before.csv") -NoTypeInformation

Get-NetRoute |
    Sort-Object AddressFamily, DestinationPrefix, RouteMetric |
    Export-Csv -Path (Join-Path $OutputPath "06-NetRoutes-Before.csv") -NoTypeInformation

Write-Host "Capturing existing Hyper-V switches..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, Id, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-VMSwitches-Before.txt")

Get-VMSwitch |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "07-VMSwitches-Before.json") -Encoding UTF8

Write-Host "Capturing management OS VM network adapters..." -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS |
    Select-Object Name, SwitchName, MacAddress, Status, IsManagementOs |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-ManagementOS-VMNetworkAdapters-Before.txt")

Write-Host "Pre-change switch evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 04_Create_And_Configure_Virtual_Switches_External_Switch_Skeleton

~~~powershell
# 04_Create_And_Configure_Virtual_Switches_External_Switch_Skeleton
# Purpose:
# Create an external Hyper-V virtual switch bound to a physical NIC.

$SwitchName = "vSwitch-External"
$UplinkAdapter = "VM-UPLINK"
$AllowManagementOS = $true

Write-Host "Confirming uplink adapter exists and is up..." -ForegroundColor Cyan

$Adapter = Get-NetAdapter -Name $UplinkAdapter -ErrorAction Stop

if ($Adapter.Status -ne "Up") {
    Write-Warning "Adapter $UplinkAdapter is not currently Up. Current status: $($Adapter.Status)"
}

Write-Host "Confirming switch does not already exist..." -ForegroundColor Cyan

$ExistingSwitch = Get-VMSwitch -Name $SwitchName -ErrorAction SilentlyContinue

if ($ExistingSwitch) {
    throw "A VMSwitch named $SwitchName already exists. Choose another name or remove the existing switch after review."
}

Write-Host "Creating external switch: $SwitchName on adapter: $UplinkAdapter" -ForegroundColor Cyan

New-VMSwitch `
    -Name $SwitchName `
    -NetAdapterName $UplinkAdapter `
    -AllowManagementOS $AllowManagementOS

Write-Host "External switch created. Current switch state:" -ForegroundColor Green

Get-VMSwitch -Name $SwitchName |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
    Format-List

Write-Host "Management OS network adapters:" -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS |
    Select-Object Name, SwitchName, MacAddress, Status |
    Format-Table -AutoSize

Write-Host "Current host IP configuration:" -ForegroundColor Cyan

Get-NetIPConfiguration |
    Format-List
~~~

# 04_Create_And_Configure_Virtual_Switches_Internal_Private_Switch_Skeleton

~~~powershell
# 04_Create_And_Configure_Virtual_Switches_Internal_Private_Switch_Skeleton
# Purpose:
# Create internal and private Hyper-V switches for lab isolation.
# Internal: VM to host and VM to VM.
# Private: VM to VM only.

$InternalSwitchName = "vSwitch-Internal-Lab"
$PrivateSwitchName = "vSwitch-Private-Isolated"

$InternalHostIPAddress = "172.16.10.1"
$InternalPrefixLength = 24

Write-Host "Creating internal switch if missing..." -ForegroundColor Cyan

if (-not (Get-VMSwitch -Name $InternalSwitchName -ErrorAction SilentlyContinue)) {
    New-VMSwitch -Name $InternalSwitchName -SwitchType Internal
}
else {
    Write-Host "Internal switch already exists: $InternalSwitchName" -ForegroundColor Yellow
}

Write-Host "Creating private switch if missing..." -ForegroundColor Cyan

if (-not (Get-VMSwitch -Name $PrivateSwitchName -ErrorAction SilentlyContinue)) {
    New-VMSwitch -Name $PrivateSwitchName -SwitchType Private
}
else {
    Write-Host "Private switch already exists: $PrivateSwitchName" -ForegroundColor Yellow
}

Write-Host "Configuring host IP on internal switch adapter..." -ForegroundColor Cyan

$InternalAdapterAlias = "vEthernet ($InternalSwitchName)"

$ExistingInternalIP = Get-NetIPAddress `
    -InterfaceAlias $InternalAdapterAlias `
    -AddressFamily IPv4 `
    -ErrorAction SilentlyContinue |
    Where-Object { $_.IPAddress -eq $InternalHostIPAddress }

if (-not $ExistingInternalIP) {
    New-NetIPAddress `
        -InterfaceAlias $InternalAdapterAlias `
        -IPAddress $InternalHostIPAddress `
        -PrefixLength $InternalPrefixLength
}
else {
    Write-Host "Internal switch host IP already exists: $InternalHostIPAddress/$InternalPrefixLength" -ForegroundColor Yellow
}

Write-Host "Current virtual switches:" -ForegroundColor Green

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
    Format-Table -AutoSize

Write-Host "Current vEthernet adapters:" -ForegroundColor Cyan

Get-NetAdapter |
    Where-Object { $_.Name -like "vEthernet*" } |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription |
    Format-Table -AutoSize

Write-Host "Current internal switch IP configuration:" -ForegroundColor Cyan

Get-NetIPConfiguration -InterfaceAlias $InternalAdapterAlias
~~~

# 04_Create_And_Configure_Virtual_Switches_Management_OS_VLAN_Skeleton

~~~powershell
# 04_Create_And_Configure_Virtual_Switches_Management_OS_VLAN_Skeleton
# Purpose:
# Configure VLAN tagging for the management OS adapter connected to an external virtual switch.
# Use this only when the physical switchport is a trunk and the host management network requires a VLAN tag.

$ExternalSwitchName = "vSwitch-External"
$ManagementVlanId = 10

Write-Host "Confirming external switch exists..." -ForegroundColor Cyan

$Switch = Get-VMSwitch -Name $ExternalSwitchName -ErrorAction Stop

if ($Switch.SwitchType -ne "External") {
    throw "$ExternalSwitchName is not an external switch."
}

Write-Host "Confirming management OS adapter exists on switch..." -ForegroundColor Cyan

$MgmtAdapter = Get-VMNetworkAdapter -ManagementOS |
    Where-Object { $_.SwitchName -eq $ExternalSwitchName -or $_.Name -eq $ExternalSwitchName }

if (-not $MgmtAdapter) {
    throw "No management OS VM network adapter was found for $ExternalSwitchName. Confirm AllowManagementOS is enabled."
}

Write-Host "Configuring management OS VLAN ID $ManagementVlanId on switch $ExternalSwitchName..." -ForegroundColor Cyan

Set-VMNetworkAdapterVlan `
    -ManagementOS `
    -VMNetworkAdapterName $MgmtAdapter.Name `
    -Access `
    -VlanId $ManagementVlanId

Write-Host "Current management OS VLAN configuration:" -ForegroundColor Green

Get-VMNetworkAdapterVlan -ManagementOS |
    Format-Table VMName, VMNetworkAdapterName, Mode, AccessVlanId, AllowedVlanIdList -AutoSize

Write-Host "Current management OS network adapters:" -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS |
    Select-Object Name, SwitchName, MacAddress, Status |
    Format-Table -AutoSize

Write-Host "Current IP configuration:" -ForegroundColor Cyan

Get-NetIPConfiguration
~~~

# 04_Create_And_Configure_Virtual_Switches_PostChange_Verification_Skeleton

~~~powershell
# 04_Create_And_Configure_Virtual_Switches_PostChange_Verification_Skeleton
# Purpose:
# Verify virtual switch configuration, management OS adapters, VLANs, IP settings, routes, and recent Hyper-V events.

$EvidenceRoot = "C:\Admin\HyperV-Switches"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing final virtual switch inventory..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, Id, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VMSwitches-After.txt")

Get-VMSwitch |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "01-VMSwitches-After.json") -Encoding UTF8

Write-Host "Capturing management OS virtual network adapters..." -ForegroundColor Cyan

Get-VMNetworkAdapter -ManagementOS |
    Select-Object Name, SwitchName, MacAddress, Status, IsManagementOs |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-ManagementOS-VMNetworkAdapters-After.txt")

Write-Host "Capturing management OS VLAN state..." -ForegroundColor Cyan

Get-VMNetworkAdapterVlan -ManagementOS |
    Format-Table VMName, VMNetworkAdapterName, Mode, AccessVlanId, AllowedVlanIdList -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-ManagementOS-VLANs-After.txt")

Write-Host "Capturing host network adapters..." -ForegroundColor Cyan

Get-NetAdapter |
    Sort-Object Name |
    Select-Object Name, Status, LinkSpeed, MacAddress, InterfaceDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-NetAdapters-After.txt")

Write-Host "Capturing host IP configuration..." -ForegroundColor Cyan

Get-NetIPConfiguration |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "05-NetIPConfiguration-After.txt") -Encoding UTF8

Write-Host "Capturing DNS and routes..." -ForegroundColor Cyan

Get-DnsClientServerAddress |
    Export-Csv -Path (Join-Path $OutputPath "06-DnsClientServerAddress-After.csv") -NoTypeInformation

Get-NetRoute |
    Sort-Object AddressFamily, DestinationPrefix, RouteMetric |
    Export-Csv -Path (Join-Path $OutputPath "07-NetRoutes-After.csv") -NoTypeInformation

Write-Host "Capturing recent Hyper-V networking events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VmSwitch-Operational" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-HyperV-VmSwitch-Operational-RecentEvents.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-HyperV-VMMS-Admin-RecentEvents.txt") -Encoding UTF8

Write-Host "Testing basic host connectivity if gateway exists..." -ForegroundColor Cyan

$DefaultRoute = Get-NetRoute -DestinationPrefix "0.0.0.0/0" -ErrorAction SilentlyContinue |
    Sort-Object RouteMetric |
    Select-Object -First 1

if ($DefaultRoute) {
    Test-NetConnection $DefaultRoute.NextHop |
        Out-File -FilePath (Join-Path $OutputPath "10-Test-DefaultGateway.txt") -Encoding UTF8
}

Write-Host "Post-change switch evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 04_Create_And_Configure_Virtual_Switches_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VMSwitch` | Hyper-V host | Lists all virtual switches |
| `Get-VMSwitch -Name 'vSwitch-External' \| Format-List *` | Hyper-V host | Confirms external switch details |
| `Get-VMSwitch \| Format-Table Name,SwitchType,NetAdapterInterfaceDescription,AllowManagementOS` | Hyper-V host | Confirms switch type, uplink, and management OS sharing |
| `Get-VMNetworkAdapter -ManagementOS` | Hyper-V host | Confirms management OS virtual adapters |
| `Get-VMNetworkAdapterVlan -ManagementOS` | Hyper-V host | Confirms management OS VLAN tagging |
| `Get-NetAdapter \| Sort Name` | Hyper-V host | Confirms physical and vEthernet adapter state |
| `Get-NetIPConfiguration` | Hyper-V host | Confirms host IP, DNS, and gateway after switch binding |
| `Get-DnsClientServerAddress -AddressFamily IPv4` | Hyper-V host | Confirms DNS client settings were preserved |
| `Get-NetRoute -DestinationPrefix '0.0.0.0/0'` | Hyper-V host | Confirms default route still exists |
| `Test-NetConnection <default-gateway>` | Hyper-V host | Confirms gateway reachability |
| `Test-NetConnection <domain-controller-or-dns-server>` | Hyper-V host | Confirms internal network reachability |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VmSwitch-Operational' -MaxEvents 20` | Hyper-V host | Confirms recent virtual switch events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Confirms no immediate VMMS switch errors |

# 04_Create_And_Configure_Virtual_Switches_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current switch state before rollback | Hyper-V host | `Get-VMSwitch \| Format-Table Name,SwitchType,NetAdapterInterfaceDescription,AllowManagementOS` | Current switch state is documented |
| 2 | Confirm no VMs depend on the switch | Hyper-V host | `Get-VMNetworkAdapter -All \| Where-Object SwitchName -eq '<switch-name>'` | Dependent VM adapters are identified |
| 3 | Disconnect VM adapters if required | Hyper-V host | `Disconnect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>'` | VM adapter is no longer attached to the switch |
| 4 | Remove external virtual switch | Hyper-V host | `Remove-VMSwitch -Name 'vSwitch-External' -Force` | External switch is removed |
| 5 | Remove internal virtual switch | Hyper-V host | `Remove-VMSwitch -Name 'vSwitch-Internal-Lab' -Force` | Internal switch is removed |
| 6 | Remove private virtual switch | Hyper-V host | `Remove-VMSwitch -Name 'vSwitch-Private-Isolated' -Force` | Private switch is removed |
| 7 | Remove internal switch host IP if needed | Hyper-V host | `Remove-NetIPAddress -InterfaceAlias 'vEthernet (vSwitch-Internal-Lab)' -IPAddress 172.16.10.1 -Confirm:$false` | Internal host IP is removed |
| 8 | Restore physical NIC IP if the external switch was removed | Hyper-V host | `New-NetIPAddress -InterfaceAlias '<physical-nic>' -IPAddress '<host-ip>' -PrefixLength '<prefix>' -DefaultGateway '<gateway>'` | Physical NIC regains management IP if needed |
| 9 | Restore DNS servers if needed | Hyper-V host | `Set-DnsClientServerAddress -InterfaceAlias '<physical-nic>' -ServerAddresses '<dns1>','<dns2>'` | DNS settings are restored |
| 10 | Confirm host network reachability | Hyper-V host | `Test-NetConnection <gateway-or-dns-server>` | Host networking is restored |
| 11 | Capture rollback evidence | Hyper-V host | `Get-NetIPConfiguration; Get-VMSwitch` | Rollback state is documented |

# 04_Create_And_Configure_Virtual_Switches_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Host loses network after creating external switch | Management OS sharing disabled or VLAN mismatch | Re-enable management OS adapter, correct VLAN, or restore physical NIC IP |
| `New-VMSwitch` fails because adapter is already bound | Physical NIC is already attached to another virtual switch | Use `Get-VMSwitch` to identify existing binding, remove or choose another adapter |
| `New-VMSwitch` says adapter not found | Wrong adapter name | Run `Get-NetAdapter` and use the exact adapter name |
| External switch created on wrong NIC | Adapter naming was unclear | Remove switch, rename NICs clearly, recreate switch on correct uplink |
| Host cannot reach DNS after switch creation | DNS settings moved or lost during adapter rebinding | Set DNS on the new `vEthernet` adapter |
| Host cannot reach gateway after VLAN tagging | Physical switchport is access mode or wrong trunk VLAN | Correct physical switchport mode or remove management VLAN tag |
| `Set-VMNetworkAdapterVlan -ManagementOS` fails | Wrong management OS adapter name | Run `Get-VMNetworkAdapter -ManagementOS` and use the listed adapter name |
| Internal switch has no host connectivity | No IP assigned to the internal `vEthernet` adapter | Assign host IP with `New-NetIPAddress` |
| Private switch cannot reach host | Private switch design blocks host communication | Use internal switch instead if host communication is required |
| VM attached to switch has no network | VM VLAN, physical trunk, DHCP, or switch assignment mismatch | Verify VM adapter switch name, VLAN, DHCP scope, and physical switchport |
| Event log shows VMSwitch errors | Driver, NIC, binding, or switch creation issue | Update NIC driver, review adapter state, recreate switch if needed |
| Remote session disconnects mid-change | The management NIC was rebound to an external switch | Use console access or out-of-band access for switch changes |
| Duplicate switch name error | Switch already exists | Pick a unique name or remove the existing switch after verification |
| VLAN settings appear blank | VLAN tagging was not configured or adapter is untagged | Configure VLAN only where trunking design requires it |
| Default route disappears | IP configuration was not preserved on vEthernet adapter | Recreate default gateway on the management OS network adapter |

# 04_Create_And_Configure_Virtual_Switches_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this virtual switch task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Provides NIC, IP, DNS, route, and physical network baseline |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before Hyper-V switch cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Establishes host defaults before networking configuration |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Uses these switches when connecting new VMs |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Builds on these switches by configuring VM adapter settings |
| 09_Configure_Hyper-V_NAT_Internal_And_Private_Lab_Networks.md | Uses internal and private switch concepts for NAT lab networking |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md | Uses working VM networking as one guest management path |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Depends on stable host networking and correct adapter planning |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses switch inventory and host network state during VM network troubleshooting |