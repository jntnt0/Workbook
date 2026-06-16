# 22_Configure_Nested_Virtualization

# 22_Configure_Nested_Virtualization_Index
22_Configure_Nested_Virtualization.md
22_Configure_Nested_Virtualization
22_Configure_Nested_Virtualization_Source_Basis
22_Configure_Nested_Virtualization_Mental_Model
22_Configure_Nested_Virtualization_Planning_Table
22_Configure_Nested_Virtualization_Configuration_Checklist
22_Configure_Nested_Virtualization_PreChange_Capture_Skeleton
22_Configure_Nested_Virtualization_Host_And_VM_Readiness_Skeleton
22_Configure_Nested_Virtualization_Enable_Expose_Virtualization_Extensions_Skeleton
22_Configure_Nested_Virtualization_Nested_Hyper-V_Guest_Config_Skeleton
22_Configure_Nested_Virtualization_Nested_Networking_MAC_Spoofing_And_NAT_Skeleton
22_Configure_Nested_Virtualization_Nested_Linux_KVM_Readiness_Skeleton
22_Configure_Nested_Virtualization_PostChange_Verification_Skeleton
22_Configure_Nested_Virtualization_Verification_Commands
22_Configure_Nested_Virtualization_Rollback
22_Configure_Nested_Virtualization_Failure_Checks
22_Configure_Nested_Virtualization_Related_Labs

# 22_Configure_Nested_Virtualization_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V nested virtualization | Supports exposing virtualization extensions to a VM |
| Microsoft Learn | Hyper-V PowerShell `Set-VMProcessor` | Supports `-ExposeVirtualizationExtensions` configuration |
| Microsoft Learn | Hyper-V virtual networking | Supports MAC spoofing and nested VM network behavior |
| Microsoft Learn | Hyper-V NAT networking | Supports internal switch and NAT design inside the nested Hyper-V guest |
| Microsoft Learn | Windows Server Hyper-V role installation | Supports installing Hyper-V inside a nested Windows Server VM |
| Microsoft Learn | Windows container and lab scenarios | Supports nested virtualization use for labs, containers, emulators, and test hypervisors |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 22_Configure_Nested_Virtualization_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Nested virtualization | Running a hypervisor inside a VM |
| L0 host | Physical Hyper-V host |
| L1 guest | VM running on the physical Hyper-V host that will itself run Hyper-V, KVM, containers, or another hypervisor |
| L2 guest | VM running inside the nested L1 guest |
| Virtualization extensions | CPU virtualization features exposed from L0 host into L1 guest |
| `ExposeVirtualizationExtensions` | Hyper-V VM processor setting that allows nested virtualization |
| Static memory | Fixed memory assigned to the L1 guest, preferred for nested Hyper-V stability |
| MAC spoofing | Hyper-V adapter setting often required so L2 guest MAC addresses can pass through L1 guest adapter |
| Nested NAT | NAT inside the L1 guest to provide network access to L2 guests |
| Internal nested switch | Hyper-V switch inside L1 guest used by L2 VMs |
| External nested switch | Switch inside L1 guest bridged through L1 adapter, usually requiring MAC spoofing |
| Hypervisor conflict | Guest OS cannot run another hypervisor if virtualization extensions are hidden |
| Mobility tradeoff | Nested virtualization may affect live migration, compatibility, performance, and resource planning |
| Lab use case | Commonly used for Hyper-V labs, Docker, Kubernetes, emulators, training, and test clusters |
| Rollback boundary | This workbook enables nested virtualization on a selected VM, not full nested cluster or container platform design |

# 22_Configure_Nested_Virtualization_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Physical Hyper-V host | `HV01` | `<l0-host>` |
| L1 VM name | `LAB-NESTED-HV01` | `<l1-vm-name>` |
| L1 guest OS | Windows Server 2022, Windows Server 2025, Windows 11, Ubuntu | `<l1-guest-os>` |
| L2 guest type | Hyper-V VM, KVM VM, containers, emulator | `<l2-guest-type>` |
| VM generation | Generation 2 | `<generation>` |
| vCPU count | `4` | `<vcpu-count>` |
| Startup memory | `12GB` | `<startup-memory>` |
| Dynamic memory enabled | No | `<yes-no>` |
| Virtualization extensions required | Yes | `<yes-no>` |
| MAC spoofing required | Yes | `<yes-no>` |
| L1 VM adapter name | `Network Adapter` | `<adapter-name>` |
| L0 switch name | `vSwitch-External` | `<l0-switch-name>` |
| L1 nested switch type | Internal, NAT, external | `<nested-switch-type>` |
| Nested NAT subnet | `172.16.100.0/24` | `<nested-nat-subnet>` |
| Nested gateway IP | `172.16.100.1` | `<nested-gateway-ip>` |
| Nested NAT name | `NestedNAT` | `<nested-nat-name>` |
| L2 VM test name | `L2-TEST01` | `<l2-vm-name>` |
| L2 VHDX path | `D:\NestedVMs\VHDX` | `<l2-vhdx-path>` |
| Containers required | Yes or No | `<yes-no>` |
| KVM required | Yes or No | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-NestedVirtualization` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 22_Configure_Nested_Virtualization_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role on L0 host | L0 host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm L1 VM exists | L0 host | `Get-VM -Name '<l1-vm-name>'` | L1 VM is found |
| 3 | Confirm L1 VM generation | L0 host | `Get-VM -Name '<l1-vm-name>' \| Select Name,Generation` | VM generation is documented |
| 4 | Capture existing CPU and memory state | L0 host | `Get-VMProcessor -VMName '<l1-vm-name>'; Get-VMMemory -VMName '<l1-vm-name>'` | Baseline is documented |
| 5 | Shut down L1 VM | L0 host | `Stop-VM -Name '<l1-vm-name>'` | L1 VM is off |
| 6 | Disable dynamic memory for L1 VM | L0 host | `Set-VMMemory -VMName '<l1-vm-name>' -DynamicMemoryEnabled $false` | L1 VM uses static memory |
| 7 | Assign enough memory to L1 VM | L0 host | `Set-VMMemory -VMName '<l1-vm-name>' -StartupBytes 12GB` | L1 VM has enough memory for L2 workloads |
| 8 | Assign enough vCPU to L1 VM | L0 host | `Set-VMProcessor -VMName '<l1-vm-name>' -Count 4` | L1 VM has planned vCPU count |
| 9 | Expose virtualization extensions | L0 host | `Set-VMProcessor -VMName '<l1-vm-name>' -ExposeVirtualizationExtensions $true` | Nested virtualization extensions are exposed |
| 10 | Enable MAC spoofing for nested networking | L0 host | `Set-VMNetworkAdapter -VMName '<l1-vm-name>' -Name '<adapter-name>' -MacAddressSpoofing On` | L2 MAC addresses can pass when needed |
| 11 | Start L1 VM | L0 host | `Start-VM -Name '<l1-vm-name>'` | L1 VM starts |
| 12 | Confirm virtualization exposure inside L1 Windows guest | L1 guest | `systeminfo` or `Get-ComputerInfo` | Hyper-V requirements show available |
| 13 | Install Hyper-V inside L1 Windows guest | L1 guest | `Install-WindowsFeature Hyper-V -IncludeManagementTools -Restart` | Hyper-V role installs inside L1 guest |
| 14 | Create nested internal switch inside L1 guest | L1 guest | `New-VMSwitch -Name 'Nested-Internal' -SwitchType Internal` | Nested switch is created |
| 15 | Configure nested NAT inside L1 guest | L1 guest | `New-NetNat -Name 'NestedNAT' -InternalIPInterfaceAddressPrefix '172.16.100.0/24'` | L2 guests can NAT through L1 |
| 16 | Create L2 test VM | L1 guest | `New-VM -Name '<l2-vm-name>' -Generation 2 -SwitchName 'Nested-Internal'` | L2 test VM is created |
| 17 | Validate L2 VM starts | L1 guest | `Start-VM -Name '<l2-vm-name>'` | L2 VM starts |
| 18 | Validate L2 network path | L2 guest | `ipconfig /all` or `ping <gateway>` | L2 guest has expected network behavior |
| 19 | Capture post-change evidence | L0 host and L1 guest | Run post-change verification skeleton | Nested virtualization state is documented |

# 22_Configure_Nested_Virtualization_PreChange_Capture_Skeleton

~~~powershell
# 22_Configure_Nested_Virtualization_PreChange_Capture_Skeleton
# Purpose:
# Capture L0 host, L1 VM, CPU, memory, networking, firmware, security, and event state before nested virtualization changes.

$EvidenceRoot = "C:\Admin\HyperV-NestedVirtualization"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$L1VMName = "LAB-NESTED-HV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$L1VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing L0 host identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-L0-Host-ComputerInfo.txt")

Write-Host "Capturing Hyper-V feature and services..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-L0-HyperV-Features.txt")

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-L0-HyperV-Services.txt")

Write-Host "Capturing L0 CPU virtualization state..." -ForegroundColor Cyan

Get-CimInstance Win32_Processor |
    Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, VirtualizationFirmwareEnabled, SecondLevelAddressTranslationExtensions |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-L0-CPU-Virtualization.txt")

Get-CimInstance Win32_ComputerSystem |
    Select-Object Manufacturer, Model, TotalPhysicalMemory, HypervisorPresent |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-L0-ComputerSystem.txt")

Write-Host "Capturing L1 VM summary..." -ForegroundColor Cyan

$VM = Get-VM -Name $L1VMName -ErrorAction Stop

$VM |
    Select-Object Name, Id, State, Status, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path, ConfigurationLocation |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "06-L1-VM-Summary-Before.txt")

$VM |
    ConvertTo-Json -Depth 10 |
    Out-File -FilePath (Join-Path $OutputPath "06-L1-VM-Summary-Before.json") -Encoding UTF8

Write-Host "Capturing L1 VM processor state..." -ForegroundColor Cyan

Get-VMProcessor -VMName $L1VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "07-L1-VMProcessor-Before.txt")

Write-Host "Capturing L1 VM memory state..." -ForegroundColor Cyan

Get-VMMemory -VMName $L1VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "08-L1-VMMemory-Before.txt")

Write-Host "Capturing L1 VM network adapter state..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $L1VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, MacAddressSpoofing, DhcpGuard, RouterGuard, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "09-L1-VMNetworkAdapter-Before.txt")

Get-VMNetworkAdapterVlan -VMName $L1VMName |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "10-L1-VMNetworkAdapterVlan-Before.txt")

Write-Host "Capturing L1 VM firmware or BIOS state..." -ForegroundColor Cyan

if ($VM.Generation -eq 2) {
    Get-VMFirmware -VMName $L1VMName |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "11-L1-VMFirmware-Before.txt")
}
else {
    Get-VMBios -VMName $L1VMName |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "11-L1-VMBios-Before.txt")
}

Write-Host "Capturing L0 virtual switches..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS, EmbeddedTeamingEnabled, IovEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "12-L0-VMSwitches.txt")

Write-Host "Capturing recent Hyper-V events..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "Microsoft-Windows-Hyper-V-Hypervisor-Admin",
    "System"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', "-"

    Get-WinEvent -LogName $Log -MaxEvents 75 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "13-$SafeLog-Before.txt") -Encoding UTF8
}

Write-Host "Pre-change nested virtualization evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 22_Configure_Nested_Virtualization_Host_And_VM_Readiness_Skeleton

~~~powershell
# 22_Configure_Nested_Virtualization_Host_And_VM_Readiness_Skeleton
# Purpose:
# Validate L0 host and L1 VM readiness before enabling nested virtualization.

$L1VMName = "LAB-NESTED-HV01"

Write-Host "Checking L0 host Hyper-V role..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize

Write-Host "Checking L0 CPU virtualization support..." -ForegroundColor Cyan

Get-CimInstance Win32_Processor |
    Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, VirtualizationFirmwareEnabled, SecondLevelAddressTranslationExtensions |
    Format-Table -AutoSize

Write-Host "Checking L1 VM state..." -ForegroundColor Cyan

$VM = Get-VM -Name $L1VMName -ErrorAction Stop

$VM |
    Select-Object Name, State, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path |
    Format-List

Write-Host "Checking L1 VM processor settings..." -ForegroundColor Cyan

Get-VMProcessor -VMName $L1VMName |
    Select-Object VMName, Count, ExposeVirtualizationExtensions, CompatibilityForMigrationEnabled, Reserve, Maximum, RelativeWeight |
    Format-List

Write-Host "Checking L1 VM memory settings..." -ForegroundColor Cyan

Get-VMMemory -VMName $L1VMName |
    Select-Object VMName, DynamicMemoryEnabled, Startup, Minimum, Maximum, AssignedMemory, Demand, Status |
    Format-List

Write-Host "Checking L1 VM network settings..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $L1VMName |
    Select-Object VMName, Name, SwitchName, MacAddressSpoofing, Status, Connected, IPAddresses |
    Format-List

Write-Host "Readiness check completed." -ForegroundColor Green
~~~

# 22_Configure_Nested_Virtualization_Enable_Expose_Virtualization_Extensions_Skeleton

~~~powershell
# 22_Configure_Nested_Virtualization_Enable_Expose_Virtualization_Extensions_Skeleton
# Purpose:
# Enable nested virtualization by exposing virtualization extensions to the L1 VM.

$L1VMName = "LAB-NESTED-HV01"
$StartupMemory = 12GB
$ProcessorCount = 4

Write-Host "Validating L1 VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $L1VMName -ErrorAction Stop

Write-Host "Stopping L1 VM if needed..." -ForegroundColor Cyan

if ($VM.State -ne "Off") {
    Stop-VM -Name $L1VMName
}

Write-Host "Disabling dynamic memory for nested virtualization stability..." -ForegroundColor Cyan

Set-VMMemory `
    -VMName $L1VMName `
    -DynamicMemoryEnabled $false `
    -StartupBytes $StartupMemory

Write-Host "Configuring vCPU count and exposing virtualization extensions..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $L1VMName `
    -Count $ProcessorCount `
    -ExposeVirtualizationExtensions $true

Write-Host "Enabling MAC spoofing for nested VM networking..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $L1VMName |
    ForEach-Object {
        Set-VMNetworkAdapter `
            -VMName $L1VMName `
            -Name $_.Name `
            -MacAddressSpoofing On
    }

Write-Host "Nested virtualization settings after change:" -ForegroundColor Green

Get-VMProcessor -VMName $L1VMName |
    Select-Object VMName, Count, ExposeVirtualizationExtensions, CompatibilityForMigrationEnabled |
    Format-List

Get-VMMemory -VMName $L1VMName |
    Select-Object VMName, DynamicMemoryEnabled, Startup, Minimum, Maximum |
    Format-List

Get-VMNetworkAdapter -VMName $L1VMName |
    Select-Object VMName, Name, SwitchName, MacAddressSpoofing, Status |
    Format-List

Write-Host "Starting L1 VM..." -ForegroundColor Cyan

Start-VM -Name $L1VMName
~~~

# 22_Configure_Nested_Virtualization_Nested_Hyper-V_Guest_Config_Skeleton

~~~powershell
# 22_Configure_Nested_Virtualization_Nested_Hyper-V_Guest_Config_Skeleton
# Purpose:
# Install and validate Hyper-V inside a Windows Server L1 guest.
# Run from L0 using PowerShell Direct or directly inside the L1 guest.

$L1VMName = "LAB-NESTED-HV01"
$UsePowerShellDirect = $true

$NestedConfig = {
    Write-Host "Checking nested guest OS identity..." -ForegroundColor Cyan

    Get-ComputerInfo |
        Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
        Format-List

    Write-Host "Checking Hyper-V requirements visibility..." -ForegroundColor Cyan

    systeminfo.exe

    Write-Host "Checking Hyper-V feature state..." -ForegroundColor Cyan

    Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
        Select-Object Name, DisplayName, InstallState |
        Format-Table -AutoSize

    Write-Host "Installing Hyper-V inside L1 guest..." -ForegroundColor Cyan

    Install-WindowsFeature `
        -Name Hyper-V `
        -IncludeManagementTools

    Write-Host "Checking Hyper-V feature after install..." -ForegroundColor Green

    Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
        Select-Object Name, DisplayName, InstallState |
        Format-Table -AutoSize

    Write-Host "A reboot may be required inside the L1 guest." -ForegroundColor Yellow
}

if ($UsePowerShellDirect) {
    Invoke-Command `
        -VMName $L1VMName `
        -Credential (Get-Credential -Message "Enter L1 guest admin credentials") `
        -ScriptBlock $NestedConfig
}
else {
    Invoke-Command -ScriptBlock $NestedConfig
}
~~~

# 22_Configure_Nested_Virtualization_Nested_Networking_MAC_Spoofing_And_NAT_Skeleton

~~~powershell
# 22_Configure_Nested_Virtualization_Nested_Networking_MAC_Spoofing_And_NAT_Skeleton
# Purpose:
# Configure L1 nested Hyper-V networking using an internal switch and NAT.
# Run inside the L1 Windows guest after Hyper-V is installed.

$NestedSwitchName = "Nested-Internal"
$NestedNatName = "NestedNAT"
$NestedNatSubnet = "172.16.100.0/24"
$NestedGatewayIP = "172.16.100.1"
$NestedPrefixLength = 24

Write-Host "Checking Hyper-V service inside L1 guest..." -ForegroundColor Cyan

Get-Service vmms -ErrorAction SilentlyContinue |
    Select-Object Name, Status, StartType |
    Format-Table -AutoSize

Write-Host "Creating nested internal switch if missing..." -ForegroundColor Cyan

if (-not (Get-VMSwitch -Name $NestedSwitchName -ErrorAction SilentlyContinue)) {
    New-VMSwitch `
        -Name $NestedSwitchName `
        -SwitchType Internal
}
else {
    Write-Host "Nested switch already exists: $NestedSwitchName" -ForegroundColor Yellow
}

Write-Host "Configuring gateway IP on nested switch adapter..." -ForegroundColor Cyan

$AdapterAlias = "vEthernet ($NestedSwitchName)"

$ExistingIP = Get-NetIPAddress -InterfaceAlias $AdapterAlias -AddressFamily IPv4 -ErrorAction SilentlyContinue |
    Where-Object { $_.IPAddress -eq $NestedGatewayIP }

if (-not $ExistingIP) {
    New-NetIPAddress `
        -InterfaceAlias $AdapterAlias `
        -IPAddress $NestedGatewayIP `
        -PrefixLength $NestedPrefixLength
}

Write-Host "Creating NAT if missing..." -ForegroundColor Cyan

if (-not (Get-NetNat -Name $NestedNatName -ErrorAction SilentlyContinue)) {
    New-NetNat `
        -Name $NestedNatName `
        -InternalIPInterfaceAddressPrefix $NestedNatSubnet
}
else {
    Write-Host "Nested NAT already exists: $NestedNatName" -ForegroundColor Yellow
}

Write-Host "Nested switch and NAT state:" -ForegroundColor Green

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription |
    Format-Table -AutoSize

Get-NetIPAddress -InterfaceAlias $AdapterAlias -AddressFamily IPv4 |
    Format-Table InterfaceAlias, IPAddress, PrefixLength -AutoSize

Get-NetNat |
    Format-Table Name, InternalIPInterfaceAddressPrefix -AutoSize

Write-Host "Create L2 VMs on switch: $NestedSwitchName" -ForegroundColor Yellow
~~~

# 22_Configure_Nested_Virtualization_Nested_Linux_KVM_Readiness_Skeleton

~~~powershell
# 22_Configure_Nested_Virtualization_Nested_Linux_KVM_Readiness_Skeleton
# Purpose:
# Validate Linux L1 guest readiness for nested KVM workloads after virtualization extensions are exposed.
# Run inside the Linux L1 guest shell.

# Linux shell commands:
# lscpu | egrep 'Virtualization|Hypervisor|Model name'
# grep -E --color 'vmx|svm' /proc/cpuinfo | head
# lsmod | egrep 'kvm|kvm_intel|kvm_amd'
# sudo modprobe kvm
# sudo modprobe kvm_intel || sudo modprobe kvm_amd
# sudo dmesg | egrep -i 'kvm|vmx|svm|virtualization' | tail -50
# sudo apt-get update
# sudo apt-get install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
# systemctl status libvirtd || systemctl status libvirt-daemon
# virsh list --all
# kvm-ok

Write-Host "This skeleton is intentionally command-reference only because it runs inside Linux L1 guest." -ForegroundColor Yellow
Write-Host "Copy the Linux commands into the L1 Linux guest shell." -ForegroundColor Yellow
~~~

# 22_Configure_Nested_Virtualization_PostChange_Verification_Skeleton

~~~powershell
# 22_Configure_Nested_Virtualization_PostChange_Verification_Skeleton
# Purpose:
# Capture L0 nested virtualization state and L1 guest Hyper-V, nested switch, NAT, and test VM state.

$EvidenceRoot = "C:\Admin\HyperV-NestedVirtualization"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$L1VMName = "LAB-NESTED-HV01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$L1VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing L1 VM state on L0 host..." -ForegroundColor Cyan

Get-VM -Name $L1VMName |
    Select-Object Name, Id, State, Status, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-L1-VM-Summary-After.txt")

Write-Host "Capturing L1 VM processor state on L0 host..." -ForegroundColor Cyan

Get-VMProcessor -VMName $L1VMName |
    Select-Object VMName, Count, ExposeVirtualizationExtensions, CompatibilityForMigrationEnabled, Reserve, Maximum, RelativeWeight |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "02-L1-VMProcessor-After.txt")

Write-Host "Capturing L1 VM memory state on L0 host..." -ForegroundColor Cyan

Get-VMMemory -VMName $L1VMName |
    Select-Object VMName, DynamicMemoryEnabled, Startup, Minimum, Maximum, AssignedMemory, Demand, Status |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-L1-VMMemory-After.txt")

Write-Host "Capturing L1 VM network adapter state on L0 host..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $L1VMName |
    Select-Object VMName, Name, SwitchName, MacAddressSpoofing, DhcpGuard, RouterGuard, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-L1-VMNetworkAdapter-After.txt")

Write-Host "Capturing L1 guest nested Hyper-V state through PowerShell Direct..." -ForegroundColor Cyan

try {
    Invoke-Command -VMName $L1VMName -Credential (Get-Credential -Message "Enter L1 guest admin credentials") -ScriptBlock {
        "L1 Guest Identity"
        Get-ComputerInfo |
            Select-Object CsName, WindowsProductName, OsBuildNumber |
            Format-List

        "L1 Hyper-V Features"
        Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize

        "L1 Hyper-V Services"
        Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
            Select-Object Name, DisplayName, Status, StartType |
            Format-Table -AutoSize

        "L1 VMHost"
        Get-VMHost -ErrorAction SilentlyContinue |
            Format-List *

        "L1 Nested Switches"
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
            Format-Table -AutoSize

        "L1 Nested NAT"
        Get-NetNat -ErrorAction SilentlyContinue |
            Format-Table Name, InternalIPInterfaceAddressPrefix -AutoSize

        "L2 VM Inventory"
        Get-VM -ErrorAction SilentlyContinue |
            Select-Object Name, State, Status, Generation, Uptime, Path |
            Format-Table -AutoSize

        "L1 IP Configuration"
        Get-NetIPConfiguration |
            Format-List
    } |
    Out-File -FilePath (Join-Path $OutputPath "05-L1-Guest-NestedHyperV-State.txt") -Encoding UTF8
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "05-L1-Guest-NestedHyperV-State-Error.txt") -Encoding UTF8
}

Write-Host "Capturing recent Hyper-V events after nested virtualization changes..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-Hypervisor-Admin",
    "System"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', "-"

    Get-WinEvent -LogName $Log -MaxEvents 100 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "06-$SafeLog-After.txt") -Encoding UTF8
}

Write-Host "Post-change nested virtualization evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 22_Configure_Nested_Virtualization_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-WindowsFeature Hyper-V` | L0 host | Confirms physical host Hyper-V role |
| `Get-VM -Name '<l1-vm-name>'` | L0 host | Confirms L1 VM state |
| `Get-VMProcessor -VMName '<l1-vm-name>'` | L0 host | Shows `ExposeVirtualizationExtensions` state |
| `Set-VMProcessor -VMName '<l1-vm-name>' -ExposeVirtualizationExtensions $true` | L0 host | Enables nested virtualization for L1 VM |
| `Get-VMMemory -VMName '<l1-vm-name>'` | L0 host | Shows static or dynamic memory state |
| `Set-VMMemory -VMName '<l1-vm-name>' -DynamicMemoryEnabled $false` | L0 host | Disables dynamic memory |
| `Set-VMNetworkAdapter -VMName '<l1-vm-name>' -MacAddressSpoofing On` | L0 host | Allows nested L2 MAC addresses through L1 |
| `Get-VMNetworkAdapter -VMName '<l1-vm-name>'` | L0 host | Confirms MAC spoofing and switch state |
| `systeminfo` | L1 Windows guest | Shows Hyper-V requirement visibility |
| `Install-WindowsFeature Hyper-V -IncludeManagementTools` | L1 Windows guest | Installs nested Hyper-V |
| `Get-WindowsFeature Hyper-V` | L1 Windows guest | Confirms nested Hyper-V role state |
| `Get-Service vmms` | L1 Windows guest | Confirms nested Hyper-V management service |
| `Get-VMHost` | L1 Windows guest | Confirms nested Hyper-V host configuration |
| `New-VMSwitch -Name 'Nested-Internal' -SwitchType Internal` | L1 Windows guest | Creates nested internal switch |
| `New-NetIPAddress -InterfaceAlias 'vEthernet (Nested-Internal)' -IPAddress 172.16.100.1 -PrefixLength 24` | L1 Windows guest | Assigns nested NAT gateway IP |
| `New-NetNat -Name 'NestedNAT' -InternalIPInterfaceAddressPrefix '172.16.100.0/24'` | L1 Windows guest | Creates nested NAT |
| `Get-NetNat` | L1 Windows guest | Confirms nested NAT |
| `New-VM -Name '<l2-vm-name>' -Generation 2 -SwitchName 'Nested-Internal'` | L1 Windows guest | Creates L2 nested VM |
| `Start-VM -Name '<l2-vm-name>'` | L1 Windows guest | Starts L2 nested VM |
| `Get-VM` | L1 Windows guest | Shows L2 VM inventory |
| `lscpu \| grep Virtualization` | L1 Linux guest | Confirms virtualization extensions are visible |
| `grep -E 'vmx\|svm' /proc/cpuinfo` | L1 Linux guest | Confirms Intel VT-x or AMD-V flags |
| `lsmod \| grep kvm` | L1 Linux guest | Confirms KVM modules loaded |
| `virsh list --all` | L1 Linux guest | Confirms libvirt can enumerate VMs |

# 22_Configure_Nested_Virtualization_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture post-issue evidence before rollback | L0 host | Run post-change verification skeleton | Broken state is documented |
| 2 | Stop L2 VMs | L1 guest | `Get-VM \| Stop-VM -TurnOff` | Nested VMs are powered off |
| 3 | Remove L2 test VM if lab cleanup requires it | L1 guest | `Remove-VM -Name '<l2-vm-name>' -Force` | L2 VM registration is removed |
| 4 | Remove nested NAT if created for test | L1 guest | `Remove-NetNat -Name 'NestedNAT' -Confirm:$false` | Nested NAT is removed |
| 5 | Remove nested switch if created for test | L1 guest | `Remove-VMSwitch -Name 'Nested-Internal' -Force` | Nested switch is removed |
| 6 | Uninstall nested Hyper-V role if rollback requires it | L1 guest | `Uninstall-WindowsFeature Hyper-V -Restart` | Hyper-V is removed from L1 guest |
| 7 | Shut down L1 VM before L0 rollback | L0 host | `Stop-VM -Name '<l1-vm-name>'` | L1 VM is off |
| 8 | Hide virtualization extensions from L1 VM | L0 host | `Set-VMProcessor -VMName '<l1-vm-name>' -ExposeVirtualizationExtensions $false` | Nested virtualization is disabled |
| 9 | Restore original vCPU count | L0 host | `Set-VMProcessor -VMName '<l1-vm-name>' -Count '<old-count>'` | vCPU count returns to baseline |
| 10 | Restore original memory settings | L0 host | `Set-VMMemory -VMName '<l1-vm-name>' -DynamicMemoryEnabled '<old-value>' -StartupBytes '<old-memory>'` | Memory settings return to baseline |
| 11 | Disable MAC spoofing if baseline required it | L0 host | `Set-VMNetworkAdapter -VMName '<l1-vm-name>' -MacAddressSpoofing Off` | MAC spoofing returns to baseline |
| 12 | Start L1 VM after rollback | L0 host | `Start-VM -Name '<l1-vm-name>'` | L1 VM starts |
| 13 | Capture rollback evidence | L0 host | Run post-change verification skeleton | Rollback state is documented |

# 22_Configure_Nested_Virtualization_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| L1 guest says virtualization is not available | `ExposeVirtualizationExtensions` is false or VM was not restarted | Shut down L1 VM, enable setting, start VM |
| `Set-VMProcessor -ExposeVirtualizationExtensions` fails | VM is running or unsupported host scenario | Stop VM and validate host Hyper-V support |
| Hyper-V role will not install inside L1 guest | Virtualization extensions hidden or unsupported guest OS | Enable nested virtualization and confirm guest OS support |
| L1 guest Hyper-V installs but VMs fail to start | Not enough memory, CPU, or hypervisor feature exposure | Increase static memory and vCPU, confirm settings |
| L1 VM has dynamic memory enabled | Nested virtualization unstable or unsupported with current design | Disable dynamic memory and assign static memory |
| L2 VM has no network | MAC spoofing disabled on L1 adapter or nested switch misconfigured | Enable MAC spoofing and verify nested switch or NAT |
| L2 VM cannot reach outside network | NAT missing, gateway wrong, DNS missing, or firewall issue | Validate `Get-NetNat`, gateway IP, DNS, and firewall |
| L2 DHCP does not work | No DHCP service on nested network | Use static IP or configure DHCP inside L1 nested network |
| L1 host loses connectivity after switch work | Wrong vNIC, VLAN, or IP configuration | Use console, restore L1 network adapter settings |
| Nested NAT already exists with overlapping subnet | Subnet conflict | Choose a different nested NAT subnet |
| KVM modules do not load in Linux L1 guest | CPU flags hidden or unsupported kernel/module path | Confirm `vmx` or `svm` flags and load correct KVM module |
| Docker or Kubernetes complains virtualization unavailable | Nested virtualization not exposed or feature conflict | Confirm virtualization flags and required Windows/Linux features |
| Android emulator or WSL2 fails inside L1 | Hypervisor features hidden or guest OS missing required features | Enable nested virtualization and install required guest features |
| L2 VM performance is poor | CPU, memory, storage, or nested overhead | Increase resources and reduce overcommitment |
| Live migration fails for L1 VM | CPU compatibility or nested virtualization constraint | Review migration compatibility and disable nested features if mobility matters |
| Checkpoints fail or are slow | Nested VMs running and storage churn high | Shut down L2 VMs or use production checkpoint-aware workflow |
| Backup takes longer | L2 VM storage and nested Hyper-V files increase churn | Schedule backup windows and validate restore |
| Clustered nested host behaves inconsistently | L1 VM moved to host without same nested support | Standardize L0 hosts and validate settings |
| L1 guest cannot use Hyper-V Manager | Tools missing or service not running | Install management tools and check `vmms` service |
| Rollback breaks L2 workloads | Nested Hyper-V removed before exporting L2 VMs | Export or shut down L2 VMs before rollback |

# 22_Configure_Nested_Virtualization_Related_Labs

| Lab                                                                           | Relationship                                                                                     |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| 00_Hyper-V_Index.md                                                           | Places this nested virtualization task in the full Hyper-V suite                                 |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Confirms L0 host CPU, firmware, memory, and Hyper-V readiness                                    |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Required before Hyper-V cmdlets and nested lab setup are available                               |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md                       | Host defaults affect L1 VM pathing and nested lab storage                                        |
| 04_Create_And_Configure_Virtual_Switches.md                                   | L0 virtual switch design affects L1 and L2 network paths                                         |
| 05_Create_Generation_1_And_Generation_2_VMs.md                                | Creates the L1 VM used for nested virtualization                                                 |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md                         | vCPU and memory planning is critical for L1 nested hypervisor workloads                          |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md                             | L1 and L2 VM storage planning depends on VHDX layout                                             |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md                   | MAC spoofing and VM adapter configuration support nested networking                              |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md                     | L1 guest OS readiness and integration services affect nested management                          |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md                 | Nested virtualization changes require controlled L1 and L2 VM lifecycle handling                 |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md                        | PowerShell Direct helps configure nested Hyper-V inside Windows L1 guests                        |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md                              | Backups should exist before nested lab expansion                                                 |
| 14_Configure_Live_Migration_And_Storage_Migration.md                          | Nested virtualization affects migration compatibility and performance                            |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md                 | Nested labs can simulate clustered Hyper-V scenarios                                             |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md         | Monitoring validates nested workload resource pressure                                           |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Troubleshooting nested labs requires L0, L1, and L2 evidence separation                          |
| 20_Configure_SR-IOV_RDMA_SET_And_Advanced_vSwitch_Performance.md              | Advanced networking and offload choices affect nested networking behavior                        |
| 21_Configure_GPU_Partitioning_DDA_And_Hardware_Acceleration.md                | Hardware acceleration and nested virtualization both require compatibility and mobility planning |