# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Index
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Source_Basis
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Mental_Model
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Planning_Table
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Configuration_Checklist
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_PreChange_Capture_Skeleton
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_CPU_Configuration_Skeleton
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Dynamic_Memory_Configuration_Skeleton
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_NUMA_And_Compatibility_Skeleton
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_PostChange_Verification_Skeleton
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Verification_Commands
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Rollback
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Failure_Checks
06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Related_Labs

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V processor configuration | Supports `Set-VMProcessor`, vCPU count, reserve, maximum, relative weight, and compatibility settings |
| Microsoft Learn | Hyper-V memory configuration | Supports `Set-VMMemory`, startup memory, dynamic memory, minimum memory, maximum memory, buffer, and memory weight |
| Microsoft Learn | Hyper-V NUMA configuration | Supports NUMA spanning, virtual NUMA visibility, and VM sizing decisions |
| Microsoft Learn | Hyper-V performance tuning | Supports avoiding overcommitment, right-sizing VM CPU and memory, and monitoring resource pressure |
| Microsoft Learn | Hyper-V PowerShell module | Supports configuration and verification through `Get-VM`, `Get-VMProcessor`, `Get-VMMemory`, and `Get-VMHostNumaNode` |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Virtual processor | A vCPU assigned to a VM from the host processor pool |
| Processor count | Number of vCPUs visible to the guest OS |
| CPU reserve | Minimum percentage of CPU capacity reserved for a VM |
| CPU maximum | Maximum percentage of CPU capacity a VM can consume |
| CPU relative weight | Priority value used when multiple VMs compete for CPU |
| Processor compatibility | Allows migration compatibility across hosts with different CPU generations, usually at some performance cost |
| Startup memory | Memory assigned to a VM when it boots |
| Static memory | Fixed memory allocation for the VM |
| Dynamic memory | Hyper-V feature that adjusts VM memory based on demand within minimum and maximum limits |
| Minimum memory | Lowest memory value Hyper-V may reduce a dynamic memory VM to after boot |
| Maximum memory | Highest memory value Hyper-V may assign to a dynamic memory VM |
| Memory buffer | Extra memory percentage Hyper-V tries to keep available above current guest demand |
| Memory weight | Priority value used when multiple VMs compete for memory |
| NUMA | Physical processor and memory locality model used by multi-socket or large memory systems |
| NUMA spanning | Allows VMs to use memory across physical NUMA nodes |
| Virtual NUMA | Presents NUMA topology to large VMs so the guest OS can optimize scheduling |
| Rollback boundary | This workbook changes VM CPU and memory settings, not guest OS roles or application tuning |

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| VM workload | Domain controller, app server, test client, Linux utility server | `<workload>` |
| Current VM state | Off, Running, Saved | `<vm-state>` |
| Startup memory | `2GB`, `4GB`, `8GB` | `<startup-memory>` |
| Dynamic memory enabled | Yes or No | `<yes-no>` |
| Minimum memory | `1GB`, `2GB` | `<minimum-memory>` |
| Maximum memory | `4GB`, `8GB`, `16GB` | `<maximum-memory>` |
| Memory buffer | `20` | `<memory-buffer-percent>` |
| Memory weight | `50`, `80`, `100` | `<memory-weight>` |
| Virtual processor count | `2`, `4` | `<processor-count>` |
| CPU reserve | `0`, `10`, `25` | `<cpu-reserve-percent>` |
| CPU maximum | `100` | `<cpu-maximum-percent>` |
| CPU relative weight | `100` | `<cpu-relative-weight>` |
| CPU compatibility mode | Enabled or disabled | `<enabled-disabled>` |
| NUMA spanning | Enabled or disabled | `<enabled-disabled>` |
| Host NUMA layout | `1 node`, `2 nodes`, `4 nodes` | `<host-numa-layout>` |
| Large VM expected | Yes or No | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-VMResources` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | Target VM is found |
| 3 | Confirm VM state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, State` | VM state is known before changes |
| 4 | Capture current CPU configuration | Hyper-V host | `Get-VMProcessor -VMName '<vm-name>' \| Format-List *` | Existing processor settings are documented |
| 5 | Capture current memory configuration | Hyper-V host | `Get-VMMemory -VMName '<vm-name>' \| Format-List *` | Existing memory settings are documented |
| 6 | Capture host processor and memory baseline | Hyper-V host | `Get-CimInstance Win32_Processor; Get-CimInstance Win32_ComputerSystem` | Host capacity is documented |
| 7 | Capture host NUMA state | Hyper-V host | `Get-VMHostNumaNode` | Host NUMA topology is documented |
| 8 | Stop VM if changing startup-only settings | Hyper-V host | `Stop-VM -Name '<vm-name>'` | VM is off before restricted changes |
| 9 | Configure vCPU count | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -Count 2` | VM has intended virtual processor count |
| 10 | Configure CPU reserve, maximum, and weight | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -Reserve 0 -Maximum 100 -RelativeWeight 100` | CPU scheduling policy is set |
| 11 | Configure CPU compatibility if migration requires it | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -CompatibilityForMigrationEnabled $true` | VM can migrate across older compatible CPU hosts |
| 12 | Configure static memory if dynamic memory is not desired | Hyper-V host | `Set-VMMemory -VMName '<vm-name>' -DynamicMemoryEnabled $false -StartupBytes 4GB` | VM uses fixed startup memory |
| 13 | Configure dynamic memory if desired | Hyper-V host | `Set-VMMemory -VMName '<vm-name>' -DynamicMemoryEnabled $true -MinimumBytes 2GB -StartupBytes 4GB -MaximumBytes 8GB` | VM uses dynamic memory range |
| 14 | Configure memory buffer | Hyper-V host | `Set-VMMemory -VMName '<vm-name>' -Buffer 20` | Hyper-V targets configured extra memory buffer |
| 15 | Configure memory priority | Hyper-V host | `Set-VMMemory -VMName '<vm-name>' -Priority 80` | VM memory priority is set |
| 16 | Confirm host NUMA spanning setting | Hyper-V host | `Get-VMHost \| Select NumaSpanningEnabled` | NUMA spanning state is known |
| 17 | Configure host NUMA spanning if required | Hyper-V host | `Set-VMHost -NumaSpanningEnabled $true` | Host NUMA spanning policy is set |
| 18 | Start VM after resource changes | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts successfully |
| 19 | Verify CPU and memory configuration | Hyper-V host | `Get-VMProcessor -VMName '<vm-name>'; Get-VMMemory -VMName '<vm-name>'` | VM resource settings match the plan |
| 20 | Capture post-change evidence | Hyper-V host | Run post-change verification skeleton | Resource configuration evidence is saved |

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_PreChange_Capture_Skeleton

~~~powershell
# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_PreChange_Capture_Skeleton
# Purpose:
# Capture current VM CPU, memory, NUMA, host capacity, and VM state before resource changes.

$EvidenceRoot = "C:\Admin\HyperV-VMResources"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, DynamicMemoryEnabled, MemoryAssigned, MemoryDemand, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.txt")

$VM |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.json") -Encoding UTF8

Write-Host "Capturing VM processor configuration..." -ForegroundColor Cyan

Get-VMProcessor -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMProcessor-Before.txt")

Get-VMProcessor -VMName $VMName |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "02-VMProcessor-Before.json") -Encoding UTF8

Write-Host "Capturing VM memory configuration..." -ForegroundColor Cyan

Get-VMMemory -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMMemory-Before.txt")

Get-VMMemory -VMName $VMName |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "03-VMMemory-Before.json") -Encoding UTF8

Write-Host "Capturing host CPU and memory capacity..." -ForegroundColor Cyan

Get-CimInstance Win32_Processor |
    Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed, VirtualizationFirmwareEnabled |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-HostProcessor-Before.txt")

Get-CimInstance Win32_ComputerSystem |
    Select-Object Manufacturer, Model, TotalPhysicalMemory, NumberOfProcessors, NumberOfLogicalProcessors, HypervisorPresent |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-HostComputerSystem-Before.txt")

Write-Host "Capturing host NUMA topology..." -ForegroundColor Cyan

Get-VMHostNumaNode |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "06-HostNumaNodes-Before.txt")

Write-Host "Capturing VMHost NUMA spanning setting..." -ForegroundColor Cyan

Get-VMHost |
    Select-Object NumaSpanningEnabled, ResourceMeteringSaveInterval, LogicalProcessorCount, MemoryCapacity |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "07-VMHost-Before.txt")

Write-Host "Capturing running VM memory demand snapshot..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, ProcessorCount, MemoryStartup, DynamicMemoryEnabled, MemoryAssigned, MemoryDemand, MemoryStatus |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-AllVM-MemoryDemand-Before.txt")

Write-Host "Pre-change VM resource evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_CPU_Configuration_Skeleton

~~~powershell
# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_CPU_Configuration_Skeleton
# Purpose:
# Configure VM processor count, CPU scheduling policy, and optional migration compatibility.

$VMName = "LAB-WIN-SRV01"

$ProcessorCount = 2
$ReservePercent = 0
$MaximumPercent = 100
$RelativeWeight = 100
$EnableMigrationCompatibility = $false
$ExposeVirtualizationExtensions = $false

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Current VM state: $($VM.State)" -ForegroundColor Cyan

if ($VM.State -ne "Off") {
    Write-Warning "Some processor changes may require the VM to be Off. Stop the VM if the setting fails."
}

Write-Host "Configuring processor count and CPU policy..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $VMName `
    -Count $ProcessorCount `
    -Reserve $ReservePercent `
    -Maximum $MaximumPercent `
    -RelativeWeight $RelativeWeight

Write-Host "Configuring migration compatibility..." -ForegroundColor Cyan

Set-VMProcessor `
    -VMName $VMName `
    -CompatibilityForMigrationEnabled $EnableMigrationCompatibility

Write-Host "Configuring nested virtualization exposure if required..." -ForegroundColor Cyan

if ($ExposeVirtualizationExtensions) {
    if ((Get-VM -Name $VMName).State -ne "Off") {
        throw "ExposeVirtualizationExtensions requires the VM to be Off. Stop the VM and rerun."
    }

    Set-VMProcessor `
        -VMName $VMName `
        -ExposeVirtualizationExtensions $true
}
else {
    if ((Get-VM -Name $VMName).State -eq "Off") {
        Set-VMProcessor `
            -VMName $VMName `
            -ExposeVirtualizationExtensions $false
    }
    else {
        Write-Host "Skipping ExposeVirtualizationExtensions because VM is not Off." -ForegroundColor Yellow
    }
}

Write-Host "Processor configuration after change:" -ForegroundColor Green

Get-VMProcessor -VMName $VMName |
    Select-Object `
        VMName,
        Count,
        Reserve,
        Maximum,
        RelativeWeight,
        CompatibilityForMigrationEnabled,
        ExposeVirtualizationExtensions |
    Format-List
~~~

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Dynamic_Memory_Configuration_Skeleton

~~~powershell
# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Dynamic_Memory_Configuration_Skeleton
# Purpose:
# Configure dynamic memory or static memory for a VM.

$VMName = "LAB-WIN-SRV01"

$UseDynamicMemory = $true

$StartupMemory = 4GB
$MinimumMemory = 2GB
$MaximumMemory = 8GB
$MemoryBufferPercent = 20
$MemoryPriority = 80

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Current VM state: $($VM.State)" -ForegroundColor Cyan

if ($UseDynamicMemory) {
    Write-Host "Configuring dynamic memory..." -ForegroundColor Cyan

    Set-VMMemory `
        -VMName $VMName `
        -DynamicMemoryEnabled $true `
        -StartupBytes $StartupMemory `
        -MinimumBytes $MinimumMemory `
        -MaximumBytes $MaximumMemory `
        -Buffer $MemoryBufferPercent `
        -Priority $MemoryPriority
}
else {
    Write-Host "Configuring static memory..." -ForegroundColor Cyan

    if ($VM.State -ne "Off") {
        throw "Static memory startup size changes should be made while the VM is Off. Stop the VM and rerun."
    }

    Set-VMMemory `
        -VMName $VMName `
        -DynamicMemoryEnabled $false `
        -StartupBytes $StartupMemory
}

Write-Host "Memory configuration after change:" -ForegroundColor Green

Get-VMMemory -VMName $VMName |
    Select-Object `
        VMName,
        DynamicMemoryEnabled,
        Startup,
        Minimum,
        Maximum,
        Buffer,
        Priority,
        AssignedMemory,
        Demand |
    Format-List

Write-Host "VM summary after memory configuration:" -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, State, DynamicMemoryEnabled, MemoryStartup, MemoryMinimum, MemoryMaximum, MemoryAssigned, MemoryDemand, MemoryStatus |
    Format-List
~~~

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_NUMA_And_Compatibility_Skeleton

~~~powershell
# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_NUMA_And_Compatibility_Skeleton
# Purpose:
# Review NUMA topology and configure host NUMA spanning policy.
# Use this carefully on production hosts because NUMA settings affect VM placement and performance.

$SetHostNumaSpanning = $true
$EnableNumaSpanning = $true

Write-Host "Capturing host NUMA nodes..." -ForegroundColor Cyan

Get-VMHostNumaNode |
    Select-Object NodeId, ProcessorCount, MemoryCapacity, MemoryAvailable |
    Format-Table -AutoSize

Write-Host "Capturing current VMHost NUMA spanning setting..." -ForegroundColor Cyan

Get-VMHost |
    Select-Object NumaSpanningEnabled, LogicalProcessorCount, MemoryCapacity |
    Format-List

if ($SetHostNumaSpanning) {
    Write-Host "Setting host NUMA spanning to: $EnableNumaSpanning" -ForegroundColor Cyan

    Set-VMHost -NumaSpanningEnabled $EnableNumaSpanning
}
else {
    Write-Host "No NUMA spanning change requested." -ForegroundColor Yellow
}

Write-Host "Reviewing large VMs against host NUMA capacity..." -ForegroundColor Cyan

$HostNumaNodes = Get-VMHostNumaNode
$LargestNodeMemory = ($HostNumaNodes | Measure-Object MemoryCapacity -Maximum).Maximum
$LargestNodeProcessorCount = ($HostNumaNodes | Measure-Object ProcessorCount -Maximum).Maximum

Get-VM |
    Sort-Object Name |
    ForEach-Object {
        $Memory = Get-VMMemory -VMName $_.Name
        $Processor = Get-VMProcessor -VMName $_.Name

        [PSCustomObject]@{
            VMName = $_.Name
            State = $_.State
            Generation = $_.Generation
            ProcessorCount = $Processor.Count
            StartupMemoryGB = [math]::Round($Memory.Startup / 1GB, 2)
            MaximumMemoryGB = [math]::Round($Memory.Maximum / 1GB, 2)
            LargerThanSingleNumaNodeMemory = $Memory.Maximum -gt $LargestNodeMemory
            MoreVCPUThanSingleNumaNode = $Processor.Count -gt $LargestNodeProcessorCount
        }
    } |
    Format-Table -AutoSize

Write-Host "Final VMHost NUMA spanning state:" -ForegroundColor Green

Get-VMHost |
    Select-Object NumaSpanningEnabled, LogicalProcessorCount, MemoryCapacity |
    Format-List
~~~

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_PostChange_Verification_Skeleton

~~~powershell
# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_PostChange_Verification_Skeleton
# Purpose:
# Verify VM CPU, memory, dynamic memory, NUMA state, and event logs after configuration.

$EvidenceRoot = "C:\Admin\HyperV-VMResources"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM summary after resource configuration..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryMinimum, MemoryMaximum, MemoryAssigned, MemoryDemand, MemoryStatus |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-After.txt")

Get-VM -Name $VMName |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "01-VM-Summary-After.json") -Encoding UTF8

Write-Host "Capturing VM processor configuration after change..." -ForegroundColor Cyan

Get-VMProcessor -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMProcessor-After.txt")

Get-VMProcessor -VMName $VMName |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "02-VMProcessor-After.json") -Encoding UTF8

Write-Host "Capturing VM memory configuration after change..." -ForegroundColor Cyan

Get-VMMemory -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMMemory-After.txt")

Get-VMMemory -VMName $VMName |
    ConvertTo-Json -Depth 6 |
    Out-File -FilePath (Join-Path $OutputPath "03-VMMemory-After.json") -Encoding UTF8

Write-Host "Capturing host NUMA state after change..." -ForegroundColor Cyan

Get-VMHostNumaNode |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "04-HostNumaNodes-After.txt")

Get-VMHost |
    Select-Object NumaSpanningEnabled, LogicalProcessorCount, MemoryCapacity, ResourceMeteringSaveInterval |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMHost-After.txt")

Write-Host "Capturing all VM resource snapshot..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryMinimum, MemoryMaximum, MemoryAssigned, MemoryDemand, MemoryStatus |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-AllVM-ResourceSnapshot-After.txt")

Write-Host "Capturing recent Hyper-V VMMS admin events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Write-Host "Capturing recent Hyper-V worker admin events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-HyperV-Worker-RecentEvents.txt") -Encoding UTF8

Write-Host "Optional VM start test if VM is Off..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName

if ($VM.State -eq "Off") {
    Start-VM -Name $VMName
    Start-Sleep -Seconds 10

    Get-VM -Name $VMName |
        Select-Object Name, State, Uptime, Status, MemoryAssigned, MemoryDemand, MemoryStatus |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "09-StartTest.txt")
}
else {
    Write-Host "VM is not Off. Skipping start test." -ForegroundColor Yellow
}

Write-Host "Post-change VM resource evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms VM exists and shows state |
| `Get-VM -Name '<vm-name>' \| Select Name, State, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, MemoryDemand, MemoryStatus` | Hyper-V host | Confirms practical CPU and memory state |
| `Get-VMProcessor -VMName '<vm-name>' \| Format-List *` | Hyper-V host | Shows full VM processor configuration |
| `Get-VMProcessor -VMName '<vm-name>' \| Select Count, Reserve, Maximum, RelativeWeight, CompatibilityForMigrationEnabled, ExposeVirtualizationExtensions` | Hyper-V host | Confirms key CPU policy settings |
| `Get-VMMemory -VMName '<vm-name>' \| Format-List *` | Hyper-V host | Shows full memory configuration |
| `Get-VMMemory -VMName '<vm-name>' \| Select DynamicMemoryEnabled, Startup, Minimum, Maximum, Buffer, Priority` | Hyper-V host | Confirms dynamic memory configuration |
| `Get-VMHostNumaNode` | Hyper-V host | Shows host NUMA nodes and capacity |
| `Get-VMHost \| Select NumaSpanningEnabled, LogicalProcessorCount, MemoryCapacity` | Hyper-V host | Confirms host NUMA spanning and capacity |
| `Get-CimInstance Win32_Processor \| Select Name, NumberOfCores, NumberOfLogicalProcessors` | Hyper-V host | Confirms physical CPU capacity |
| `Get-CimInstance Win32_ComputerSystem \| Select TotalPhysicalMemory, HypervisorPresent` | Hyper-V host | Confirms host memory and hypervisor state |
| `Start-VM -Name '<vm-name>'` | Hyper-V host | Confirms VM can start with configured resources |
| `Measure-VM -Name '<vm-name>'` | Hyper-V host | Confirms metering data if resource metering is enabled |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks recent VMMS resource configuration errors |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM worker process resource issues |

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Review pre-change processor settings | Hyper-V host | `Get-Content '<prechange-path>\02-VMProcessor-Before.txt'` | Original CPU settings are identified |
| 2 | Review pre-change memory settings | Hyper-V host | `Get-Content '<prechange-path>\03-VMMemory-Before.txt'` | Original memory settings are identified |
| 3 | Stop VM if required | Hyper-V host | `Stop-VM -Name '<vm-name>'` | VM is off for restricted changes |
| 4 | Restore processor count | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -Count '<old-count>'` | vCPU count is restored |
| 5 | Restore CPU reserve, maximum, and weight | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -Reserve '<old-reserve>' -Maximum '<old-maximum>' -RelativeWeight '<old-weight>'` | CPU scheduling policy is restored |
| 6 | Restore CPU compatibility setting | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -CompatibilityForMigrationEnabled $false` | Compatibility mode returns to prior state |
| 7 | Restore nested virtualization exposure | Hyper-V host | `Set-VMProcessor -VMName '<vm-name>' -ExposeVirtualizationExtensions $false` | Nested virtualization exposure returns to prior state |
| 8 | Restore static memory | Hyper-V host | `Set-VMMemory -VMName '<vm-name>' -DynamicMemoryEnabled $false -StartupBytes '<old-startup-memory>'` | Static memory configuration is restored |
| 9 | Restore dynamic memory | Hyper-V host | `Set-VMMemory -VMName '<vm-name>' -DynamicMemoryEnabled $true -MinimumBytes '<old-min>' -StartupBytes '<old-startup>' -MaximumBytes '<old-max>' -Buffer '<old-buffer>' -Priority '<old-priority>'` | Dynamic memory configuration is restored |
| 10 | Restore host NUMA spanning | Hyper-V host | `Set-VMHost -NumaSpanningEnabled '<old-value>'` | Host NUMA spanning state is restored |
| 11 | Start VM after rollback | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts with restored settings |
| 12 | Capture rollback evidence | Hyper-V host | `Get-VMProcessor -VMName '<vm-name>'; Get-VMMemory -VMName '<vm-name>'` | Rollback settings are documented |

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Set-VMProcessor` fails | VM state does not allow the requested change | Stop the VM, rerun the command, then start the VM |
| `Set-VMMemory` fails | Startup, minimum, or maximum values are invalid | Ensure minimum is less than or equal to startup and startup is less than or equal to maximum |
| VM fails to start after memory increase | Host lacks available memory | Lower startup memory, stop other VMs, or add host memory |
| VM starts slowly or pages heavily | Startup memory or minimum memory too low | Increase startup memory and minimum memory |
| Dynamic memory guest performs poorly | Maximum memory too low or buffer too small | Increase maximum memory and buffer |
| VM consumes too much host memory | Maximum memory too high or memory pressure from guest workload | Lower maximum memory or tune guest workload |
| CPU usage is high inside guest | Too few vCPUs or guest workload issue | Increase vCPU count carefully or tune application workload |
| Host CPU is saturated | Too many vCPUs across VMs or high workload contention | Reduce vCPU overcommitment or rebalance workloads |
| VM becomes less responsive after adding vCPUs | Guest OS or workload does not benefit from extra vCPUs | Reduce vCPU count and monitor performance |
| Migration compatibility causes performance drop | CPU compatibility masks newer CPU features | Disable compatibility unless cross-host migration requires it |
| Nested virtualization does not work | Virtualization extensions are not exposed | Stop VM and set `ExposeVirtualizationExtensions` to true |
| NUMA spanning causes inconsistent performance | VM memory is crossing NUMA nodes | Right-size VM to fit within a NUMA node or review host NUMA policy |
| Large VM cannot start | VM exceeds single NUMA node and NUMA spanning is disabled | Enable NUMA spanning or reduce VM size |
| `Get-VMHostNumaNode` shows unexpected layout | Host hardware or BIOS presents different NUMA topology | Review BIOS, firmware, processor layout, and memory population |
| Memory demand stays above assigned memory | Dynamic memory cannot satisfy workload demand | Increase maximum memory or reduce guest workload |
| Event logs show worker process resource errors | Invalid resource setting or host pressure | Review VMMS and Worker Admin logs, then adjust CPU or memory settings |
| Resource metering shows no data | Resource metering not enabled | Run `Enable-VMResourceMetering -VMName '<vm-name>'` if metering is required |

# 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this resource configuration task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host CPU, memory, virtualization, and NUMA readiness |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before VM resource cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Establishes host defaults and NUMA spanning baseline |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates the VMs that receive CPU and memory settings here |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Complements compute tuning with VM disk configuration |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Complements CPU and memory settings with VM network resource settings |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Uses resource-aware planning because checkpoints can affect memory and storage behavior |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Uses VM start and stop behavior impacted by CPU and memory sizing |
| 14_Configure_Live_Migration_And_Storage_Migration.md | CPU compatibility and memory sizing can affect live migration success |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses CPU, memory, and NUMA state during VM startup and performance troubleshooting |