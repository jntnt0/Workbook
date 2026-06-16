# 02_Install_Hyper-V_Role_And_Management_Tools

# 02_Install_Hyper-V_Role_And_Management_Tools_Index
02_Install_Hyper-V_Role_And_Management_Tools.md
02_Install_Hyper-V_Role_And_Management_Tools
02_Install_Hyper-V_Role_And_Management_Tools_Source_Basis
02_Install_Hyper-V_Role_And_Management_Tools_Mental_Model
02_Install_Hyper-V_Role_And_Management_Tools_Planning_Table
02_Install_Hyper-V_Role_And_Management_Tools_Configuration_Checklist
02_Install_Hyper-V_Role_And_Management_Tools_PreInstall_Check_Skeleton
02_Install_Hyper-V_Role_And_Management_Tools_Role_Install_Skeleton
02_Install_Hyper-V_Role_And_Management_Tools_Management_Tools_Only_Skeleton
02_Install_Hyper-V_Role_And_Management_Tools_PostInstall_Verification_Skeleton
02_Install_Hyper-V_Role_And_Management_Tools_Remote_Management_Skeleton
02_Install_Hyper-V_Role_And_Management_Tools_Verification_Commands
02_Install_Hyper-V_Role_And_Management_Tools_Rollback
02_Install_Hyper-V_Role_And_Management_Tools_Failure_Checks
02_Install_Hyper-V_Role_And_Management_Tools_Related_Labs

# 02_Install_Hyper-V_Role_And_Management_Tools_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install the Hyper-V role on Windows Server | Supports role installation with Server Manager and PowerShell |
| Microsoft Learn | Hyper-V system requirements | Supports CPU virtualization, SLAT, DEP, and firmware checks before install |
| Microsoft Learn | Hyper-V PowerShell module | Supports management through `Hyper-V-PowerShell` |
| Microsoft Learn | Remote Server Administration Tools | Supports installing Hyper-V tools on management systems |
| Microsoft Learn | Windows Server roles and features | Supports `Install-WindowsFeature`, `Get-WindowsFeature`, and `Uninstall-WindowsFeature` |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this task aligned with AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 02_Install_Hyper-V_Role_And_Management_Tools_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Hyper-V role | The Windows Server role that turns the server into a virtualization host |
| Management tools | The MMC console and PowerShell module used to manage Hyper-V locally or remotely |
| Role install | Adds the hypervisor, Hyper-V services, VM management stack, and host components |
| Tools-only install | Adds Hyper-V management tools without making the system a Hyper-V host |
| Reboot requirement | Installing Hyper-V requires a reboot because the Windows hypervisor loads below the OS |
| Pre-install baseline | Confirms hardware, OS, network, storage, and reboot state before changing the host |
| Post-install validation | Confirms services, modules, role state, and hypervisor presence after reboot |
| Remote management | Allows another admin workstation or server to manage the Hyper-V host |
| Nested virtualization | Required when the Hyper-V host itself is running as a VM |
| Rollback boundary | Removing the role removes Hyper-V host capability but should not be treated as VM data cleanup |

# 02_Install_Hyper-V_Role_And_Management_Tools_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| Host purpose | Lab host, production host, cluster node | `<host-purpose>` |
| Install target | Local server, remote server, management workstation | `<install-target>` |
| Install type | Full role, tools only | `<role-or-tools-only>` |
| Server version | Windows Server 2022, Windows Server 2025 | `<server-version>` |
| Domain state | Domain joined, workgroup | `<domain-state>` |
| Admin account | `CORP\Domain Admin` or local admin | `<admin-account>` |
| Reboot window | Immediate, scheduled maintenance | `<reboot-window>` |
| Management method | Local PowerShell, Server Manager, Windows Admin Center | `<management-method>` |
| Hyper-V PowerShell module required | Yes | `<yes-no>` |
| MMC management console required | Yes | `<yes-no>` |
| Remote management required | Yes or No | `<yes-no>` |
| Baseline evidence path | `C:\Admin\HyperV-Baseline` | `<baseline-path>` |
| Install log path | `C:\Admin\HyperV-Install` | `<install-log-path>` |
| Rollback allowed | Yes or No | `<rollback-allowed>` |

# 02_Install_Hyper-V_Role_And_Management_Tools_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm administrative shell | Hyper-V host candidate | `whoami /groups` | User has local administrator rights |
| 2 | Confirm host baseline was completed | Hyper-V host candidate | `Test-Path 'C:\Admin\HyperV-Baseline'` | Prior baseline evidence exists |
| 3 | Confirm OS version and edition | Hyper-V host candidate | `Get-ComputerInfo \| Select WindowsProductName, WindowsVersion, OsBuildNumber, OsArchitecture` | Supported Windows Server OS is confirmed |
| 4 | Confirm pending reboot state | Hyper-V host candidate | `Test-Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending'` | No pending reboot before install |
| 5 | Confirm hardware virtualization readiness | Hyper-V host candidate | `Get-CimInstance Win32_Processor \| Select VirtualizationFirmwareEnabled, SecondLevelAddressTranslationExtensions, VMMonitorModeExtensions, DataExecutionPrevention_Available` | CPU and firmware readiness confirmed |
| 6 | Confirm current Hyper-V feature state | Hyper-V host candidate | `Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools` | Current role and tool state is documented |
| 7 | Create install evidence folder | Hyper-V host candidate | `New-Item -Path 'C:\Admin\HyperV-Install' -ItemType Directory -Force` | Install log folder exists |
| 8 | Install Hyper-V role with management tools | Hyper-V host candidate | `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools` | Hyper-V role is staged for install |
| 9 | Reboot host after install | Hyper-V host candidate | `Restart-Computer` | Host restarts and hypervisor loads |
| 10 | Confirm Hyper-V role installed after reboot | Hyper-V host | `Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools` | Role and tools show installed |
| 11 | Confirm Hyper-V PowerShell module available | Hyper-V host | `Get-Module -ListAvailable Hyper-V` | Hyper-V module is present |
| 12 | Confirm Hyper-V services exist | Hyper-V host | `Get-Service vmms, vmcompute -ErrorAction SilentlyContinue` | Hyper-V services are visible |
| 13 | Confirm hypervisor is present | Hyper-V host | `Get-CimInstance Win32_ComputerSystem \| Select HypervisorPresent` | `HypervisorPresent` shows `True` |
| 14 | Confirm Hyper-V Manager opens | Hyper-V host | `virtmgmt.msc` | Hyper-V Manager launches |
| 15 | Confirm no immediate Hyper-V errors | Hyper-V host | `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | No critical install-related failures |
| 16 | Export post-install evidence | Hyper-V host | Run post-install verification skeleton | Install evidence is saved |

# 02_Install_Hyper-V_Role_And_Management_Tools_PreInstall_Check_Skeleton

~~~powershell
# 02_Install_Hyper-V_Role_And_Management_Tools_PreInstall_Check_Skeleton
# Purpose:
# Confirm the host is safe to modify before installing Hyper-V.

$InstallRoot = "C:\Admin\HyperV-Install"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $InstallRoot "PreInstall-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Collecting OS and identity state..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, WindowsVersion, OsBuildNumber, OsArchitecture |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "01-ComputerInfo.txt") -Encoding UTF8

Write-Host "Collecting Hyper-V feature state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperVFeatureState.txt")

Write-Host "Checking processor virtualization readiness..." -ForegroundColor Cyan

$ProcessorChecks = Get-CimInstance Win32_Processor | Select-Object `
    Name,
    NumberOfCores,
    NumberOfLogicalProcessors,
    VirtualizationFirmwareEnabled,
    SecondLevelAddressTranslationExtensions,
    VMMonitorModeExtensions,
    DataExecutionPrevention_Available

$ProcessorChecks |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-ProcessorVirtualizationChecks.txt")

$PendingRebootChecks = [ordered]@{
    ComponentBasedServicing = Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending"
    WindowsUpdate           = Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired"
    PendingFileRename       = $null -ne (Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager" -Name PendingFileRenameOperations -ErrorAction SilentlyContinue)
}

$PendingRebootObjects = $PendingRebootChecks.GetEnumerator() | ForEach-Object {
    [PSCustomObject]@{
        Check = $_.Key
        Pending = $_.Value
    }
}

$PendingRebootObjects |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-PendingRebootChecks.txt")

$BlockingIssues = @()

foreach ($Processor in $ProcessorChecks) {
    if ($Processor.VirtualizationFirmwareEnabled -ne $true) {
        $BlockingIssues += "Firmware virtualization is not enabled."
    }

    if ($Processor.SecondLevelAddressTranslationExtensions -ne $true) {
        $BlockingIssues += "SLAT support is not available."
    }

    if ($Processor.VMMonitorModeExtensions -ne $true) {
        $BlockingIssues += "VM monitor mode extensions are not available."
    }

    if ($Processor.DataExecutionPrevention_Available -ne $true) {
        $BlockingIssues += "DEP is not available."
    }
}

if ($PendingRebootObjects | Where-Object { $_.Pending -eq $true }) {
    $BlockingIssues += "Pending reboot detected."
}

$BlockingIssues |
    Out-File -FilePath (Join-Path $OutputPath "05-BlockingIssues.txt") -Encoding UTF8

if ($BlockingIssues.Count -gt 0) {
    Write-Warning "Pre-install checks failed. Resolve blocking issues before installing Hyper-V."
    $BlockingIssues | ForEach-Object { Write-Warning $_ }
}
else {
    Write-Host "Pre-install checks passed. Host is ready for Hyper-V role installation." -ForegroundColor Green
}

Write-Host "Pre-install evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 02_Install_Hyper-V_Role_And_Management_Tools_Role_Install_Skeleton

~~~powershell
# 02_Install_Hyper-V_Role_And_Management_Tools_Role_Install_Skeleton
# Purpose:
# Install the Hyper-V role and management tools on the local Windows Server host.

$InstallRoot = "C:\Admin\HyperV-Install"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $InstallRoot "Install-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Starting Hyper-V role installation..." -ForegroundColor Cyan

$InstallResult = Install-WindowsFeature -Name Hyper-V -IncludeManagementTools

$InstallResult |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-InstallResult.txt")

$InstallResult |
    ConvertTo-Json -Depth 4 |
    Out-File -FilePath (Join-Path $OutputPath "01-InstallResult.json") -Encoding UTF8

Write-Host "Current Hyper-V role state after installation command:" -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-FeatureStateAfterInstallCommand.txt")

if ($InstallResult.RestartNeeded -eq "Yes") {
    Write-Warning "Restart is required before Hyper-V is usable."
    Write-Host "Run this during the approved reboot window:" -ForegroundColor Yellow
    Write-Host "Restart-Computer" -ForegroundColor Yellow
}
else {
    Write-Host "Install completed without a reported restart requirement. Verify anyway before creating VMs." -ForegroundColor Green
}

Write-Host "Install evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 02_Install_Hyper-V_Role_And_Management_Tools_Management_Tools_Only_Skeleton

~~~powershell
# 02_Install_Hyper-V_Role_And_Management_Tools_Management_Tools_Only_Skeleton
# Purpose:
# Install Hyper-V management tools only on an admin workstation or management server.
# Use this when the system should manage Hyper-V but should not become a Hyper-V host.

$InstallRoot = "C:\Admin\HyperV-Tools"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $InstallRoot "ToolsOnly-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Installing Hyper-V management tools only..." -ForegroundColor Cyan

$Tools = @(
    "Hyper-V-PowerShell",
    "RSAT-Hyper-V-Tools"
)

$Results = foreach ($Tool in $Tools) {
    Install-WindowsFeature -Name $Tool
}

$Results |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-ToolsInstallResult.txt")

Write-Host "Confirming tools state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-ToolsFeatureState.txt")

Write-Host "Confirming Hyper-V PowerShell module availability..." -ForegroundColor Cyan

Get-Module -ListAvailable Hyper-V |
    Select-Object Name, Version, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-HyperVModule.txt")

Write-Host "Tools-only installation evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 02_Install_Hyper-V_Role_And_Management_Tools_PostInstall_Verification_Skeleton

~~~powershell
# 02_Install_Hyper-V_Role_And_Management_Tools_PostInstall_Verification_Skeleton
# Purpose:
# Verify Hyper-V role, management tools, services, module, logs, and hypervisor state after reboot.

$InstallRoot = "C:\Admin\HyperV-Install"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $InstallRoot "PostInstall-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Checking feature state..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-HyperVFeatureState.txt")

Write-Host "Checking Hyper-V PowerShell module..." -ForegroundColor Cyan

Get-Module -ListAvailable Hyper-V |
    Select-Object Name, Version, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperVModule.txt")

Write-Host "Checking Hyper-V services..." -ForegroundColor Cyan

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-HyperVServices.txt")

Write-Host "Checking hypervisor presence..." -ForegroundColor Cyan

Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, HypervisorPresent, Manufacturer, Model |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-HypervisorPresence.txt")

Write-Host "Checking VM inventory..." -ForegroundColor Cyan

try {
    Get-VM |
        Select-Object Name, State, Generation, ProcessorCount, MemoryStartup |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "05-VMInventory.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "05-VMInventory-Error.txt") -Encoding UTF8
}

Write-Host "Checking recent Hyper-V VMMS admin events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 30 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "06-HyperV-VMMS-Admin-RecentEvents.txt") -Encoding UTF8

Write-Host "Checking recent Hyper-V hypervisor admin events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Hypervisor-Admin" -MaxEvents 30 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-Hypervisor-Admin-RecentEvents.txt") -Encoding UTF8

Write-Host "Post-install verification evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 02_Install_Hyper-V_Role_And_Management_Tools_Remote_Management_Skeleton

~~~powershell
# 02_Install_Hyper-V_Role_And_Management_Tools_Remote_Management_Skeleton
# Purpose:
# Enable basic PowerShell remote management for later Hyper-V administration.
# Use only if remote management is part of the lab or operational plan.

$InstallRoot = "C:\Admin\HyperV-Install"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $InstallRoot "RemoteManagement-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing WinRM service state before change..." -ForegroundColor Cyan

Get-Service WinRM |
    Select-Object Name, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-WinRM-Before.txt")

Write-Host "Enabling PowerShell remoting..." -ForegroundColor Cyan

Enable-PSRemoting -Force

Write-Host "Confirming WinRM service state after change..." -ForegroundColor Cyan

Get-Service WinRM |
    Select-Object Name, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-WinRM-After.txt")

Write-Host "Checking firewall rules for remote management..." -ForegroundColor Cyan

Get-NetFirewallRule |
    Where-Object {
        $_.DisplayGroup -like "*Remote Management*" -or
        $_.DisplayGroup -like "*Windows Remote Management*"
    } |
    Select-Object DisplayName, DisplayGroup, Enabled, Direction, Action, Profile |
    Sort-Object DisplayGroup, DisplayName |
    Export-Csv -Path (Join-Path $OutputPath "03-RemoteManagementFirewallRules.csv") -NoTypeInformation

Write-Host "Testing local remoting endpoint..." -ForegroundColor Cyan

Test-WSMan localhost |
    Out-File -FilePath (Join-Path $OutputPath "04-TestWSMan-Localhost.txt") -Encoding UTF8

Write-Host "Remote management evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 02_Install_Hyper-V_Role_And_Management_Tools_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, RSAT-Hyper-V-Tools` | Hyper-V host | Confirms role and management tools installation state |
| `Get-Module -ListAvailable Hyper-V` | Hyper-V host or management server | Confirms Hyper-V PowerShell module is available |
| `Get-Command -Module Hyper-V` | Hyper-V host or management server | Confirms Hyper-V cmdlets are available |
| `Get-Service vmms, vmcompute -ErrorAction SilentlyContinue` | Hyper-V host | Confirms core Hyper-V services exist |
| `Get-CimInstance Win32_ComputerSystem \| Select HypervisorPresent` | Hyper-V host | Confirms the hypervisor is loaded after reboot |
| `systeminfo.exe` | Hyper-V host | Confirms hypervisor state and system information |
| `virtmgmt.msc` | Hyper-V host or management server | Confirms Hyper-V Manager can launch |
| `Get-VM` | Hyper-V host | Confirms Hyper-V PowerShell can query VM inventory |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Confirms VMMS event log is present and readable |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Hypervisor-Admin' -MaxEvents 20` | Hyper-V host | Confirms hypervisor event log is present and readable |
| `Get-Service WinRM` | Hyper-V host | Confirms remote management service state |
| `Test-WSMan <host-name>` | Management server | Confirms WinRM reachability to Hyper-V host |

# 02_Install_Hyper-V_Role_And_Management_Tools_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm no VMs are running before role removal | Hyper-V host | `Get-VM` | VM state is known before rollback |
| 2 | Stop or export lab VMs if any were created later | Hyper-V host | `Stop-VM -Name '<vm-name>'` | VMs are safely stopped before role removal |
| 3 | Remove Hyper-V role | Hyper-V host | `Uninstall-WindowsFeature -Name Hyper-V` | Hyper-V role is removed from Windows feature state |
| 4 | Reboot after role removal | Hyper-V host | `Restart-Computer` | Host restarts without Hyper-V role loaded |
| 5 | Confirm role removal | Hyper-V host | `Get-WindowsFeature Hyper-V` | `InstallState` shows available or removed |
| 6 | Remove management tools only if desired | Hyper-V host or management server | `Uninstall-WindowsFeature -Name Hyper-V-PowerShell, RSAT-Hyper-V-Tools` | Hyper-V management tools are removed |
| 7 | Remove generated install evidence if not needed | Hyper-V host | `Remove-Item 'C:\Admin\HyperV-Install' -Recurse -Force` | Install evidence folder is removed |
| 8 | Keep VM files unless deliberate cleanup is approved | Hyper-V host | `N/A` | VM configuration and VHDX data are not deleted by accident |

# 02_Install_Hyper-V_Role_And_Management_Tools_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Install-WindowsFeature` fails | Unsupported OS edition, corrupt component store, or missing installation source | Confirm OS edition, run `DISM /Online /Cleanup-Image /RestoreHealth`, retry |
| Install requires reboot | Normal Hyper-V behavior | Reboot during approved maintenance window |
| `HypervisorPresent` remains `False` after reboot | Firmware virtualization disabled or boot configuration issue | Enable virtualization in BIOS or UEFI, confirm `bcdedit /enum` hypervisor launch settings |
| Host blue screens or fails after enabling Hyper-V | Driver, firmware, nested virtualization, or hardware compatibility issue | Update firmware and drivers, validate nested virtualization support, remove role if needed |
| `Get-VM` is not recognized | Hyper-V PowerShell module not installed or not loaded | Install `Hyper-V-PowerShell`, then open a new elevated PowerShell session |
| Hyper-V Manager missing | RSAT Hyper-V tools not installed | Install `RSAT-Hyper-V-Tools` |
| `vmms` service missing | Hyper-V role did not install correctly | Recheck `Get-WindowsFeature Hyper-V`, reinstall role |
| `vmms` service stopped | Service startup failure or host issue | Start service with `Start-Service vmms`, then review VMMS Admin event log |
| Hyper-V event logs missing | Role not fully installed or reboot not completed | Reboot and confirm role state |
| Remote Hyper-V management fails | WinRM disabled, firewall blocked, DNS issue, or permissions issue | Enable PowerShell remoting, confirm firewall rules, test DNS, use admin credentials |
| Cannot install on nested VM | Parent hypervisor is not exposing virtualization extensions | Enable nested virtualization on parent host |
| Network breaks after install | Existing NIC binding or driver issue | Review NIC state, IP config, and driver after reboot |
| Server Manager cannot manage host | Firewall, WinRM, or trust issue | Test `Test-WSMan`, verify DNS, credentials, firewall, and TrustedHosts if workgroup |
| Tools-only install accidentally used | Hyper-V role was not installed, only tools were installed | Install full `Hyper-V` role on the actual host |
| Role removal does not delete VM files | Normal behavior | Clean VM files manually only after confirming they are no longer needed |

# 02_Install_Hyper-V_Role_And_Management_Tools_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this install task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Required prerequisite before installing the Hyper-V role |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Follows role installation by setting default VM and VHDX paths |
| 04_Create_And_Configure_Virtual_Switches.md | Follows role installation by binding virtual switches to physical NICs |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Depends on successful Hyper-V role installation |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | Depends on working Hyper-V cmdlets and VM creation capability |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Depends on Hyper-V storage components and PowerShell module |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Depends on installed Hyper-V host and VM creation |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md | Depends on Hyper-V role and supported guest VMs |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses this install state as the starting point for host-level troubleshooting |