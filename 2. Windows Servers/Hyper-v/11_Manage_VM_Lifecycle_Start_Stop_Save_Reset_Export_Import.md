# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Index
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Source_Basis
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Mental_Model
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Planning_Table
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Configuration_Checklist
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_PreChange_Capture_Skeleton
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Start_Stop_Save_Reset_Skeleton
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Automatic_Actions_Skeleton
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Export_VM_Skeleton
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Import_VM_Register_Copy_Restore_Skeleton
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_PostChange_Verification_Skeleton
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Verification_Commands
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Rollback
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Failure_Checks
11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Related_Labs

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V VM lifecycle management | Supports starting, stopping, saving, pausing, resuming, and resetting VMs |
| Microsoft Learn | Hyper-V PowerShell `Start-VM`, `Stop-VM`, `Save-VM`, `Restart-VM`, `Suspend-VM`, `Resume-VM` | Supports VM power-state operations |
| Microsoft Learn | Hyper-V PowerShell `Export-VM` | Supports exporting VM configuration, disks, and checkpoint state |
| Microsoft Learn | Hyper-V PowerShell `Import-VM`, `Compare-VM`, and `Repair-VM` | Supports importing, registering, copying, restoring, and repairing VM imports |
| Microsoft Learn | Hyper-V automatic start and stop actions | Supports host boot and shutdown behavior for VMs |
| Microsoft Learn | Hyper-V event logs | Supports lifecycle troubleshooting through VMMS and Worker event logs |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VM lifecycle | The normal operational handling of a VM from power on through shutdown, save, reset, export, and import |
| Start | Powers on the VM |
| Stop | Turns off or shuts down the VM depending on command options and guest integration state |
| Shut down | Requests a graceful guest OS shutdown through integration services |
| Turn off | Hard power-off equivalent, used when the guest is hung or shutdown is unavailable |
| Save | Writes VM memory and device state to disk so the VM can resume later |
| Pause | Temporarily suspends VM execution while keeping state in memory |
| Reset | Restarts VM power state without a clean guest OS shutdown |
| Export | Copies VM configuration, disks, and related state to an export folder |
| Import register | Registers an existing VM in-place without copying its files |
| Import restore | Restores VM files back to their original layout where possible |
| Import copy | Copies VM files to a new location and can generate a new VM ID |
| VM ID | Unique identifier for a VM, important when importing duplicates |
| Automatic start action | Controls whether the VM starts when the Hyper-V host starts |
| Automatic stop action | Controls whether the VM saves, shuts down, or turns off when the host stops |
| Saved state risk | Saved state depends on memory and device state, not a clean app-level shutdown |
| Rollback boundary | This workbook manages VM lifecycle and portability, not backup software or disaster recovery orchestration |

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| VM role | Domain controller, member server, client, Linux utility VM | `<vm-role>` |
| Normal start behavior | Manual, automatic if previously running, always automatic | `<start-behavior>` |
| Automatic start action | Nothing, StartIfRunning, Start | `<automatic-start-action>` |
| Automatic start delay | `0`, `30`, `60` seconds | `<automatic-start-delay>` |
| Automatic stop action | Save, ShutDown, TurnOff | `<automatic-stop-action>` |
| Graceful shutdown preferred | Yes or No | `<yes-no>` |
| Save state allowed | Yes or No | `<yes-no>` |
| Hard reset allowed | Lab only, break-glass only, never | `<reset-policy>` |
| Export path | `D:\Hyper-V\Exports` | `<export-path>` |
| Export folder naming | `<vm-name>-yyyyMMdd-HHmmss` | `<export-folder-format>` |
| Import mode | Register, restore, copy | `<import-mode>` |
| Import destination VM path | `D:\Hyper-V\Virtual Machines` | `<vm-config-destination>` |
| Import destination VHDX path | `D:\Hyper-V\Virtual Hard Disks` | `<vhdx-destination>` |
| Generate new VM ID | Yes for duplicates, no for same VM restore | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-VMLifecycle` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | Target VM is found |
| 3 | Capture current VM state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, State, Status, Uptime` | VM state is documented |
| 4 | Capture VM hardware and path state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Format-List *` | Current VM configuration is documented |
| 5 | Capture disk attachment state | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>'` | VHDX paths are documented |
| 6 | Capture checkpoint state | Hyper-V host | `Get-VMCheckpoint -VMName '<vm-name>'` | Checkpoint tree is documented |
| 7 | Start the VM | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM powers on |
| 8 | Gracefully stop the VM through guest shutdown | Hyper-V host | `Stop-VM -Name '<vm-name>'` | VM shuts down cleanly if integration services are working |
| 9 | Force power off only when approved | Hyper-V host | `Stop-VM -Name '<vm-name>' -TurnOff` | VM is powered off without guest shutdown |
| 10 | Save VM state if allowed | Hyper-V host | `Save-VM -Name '<vm-name>'` | VM enters saved state |
| 11 | Resume saved VM | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM resumes from saved state |
| 12 | Restart VM if guest reset is required | Hyper-V host | `Restart-VM -Name '<vm-name>' -Force` | VM restarts |
| 13 | Pause VM if required | Hyper-V host | `Suspend-VM -Name '<vm-name>'` | VM state changes to paused |
| 14 | Resume paused VM | Hyper-V host | `Resume-VM -Name '<vm-name>'` | VM resumes execution |
| 15 | Configure automatic start and stop actions | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticStartAction StartIfRunning -AutomaticStopAction ShutDown` | VM host lifecycle behavior is configured |
| 16 | Create export folder | Hyper-V host | `New-Item -Path 'D:\Hyper-V\Exports' -ItemType Directory -Force` | Export path exists |
| 17 | Export VM | Hyper-V host | `Export-VM -Name '<vm-name>' -Path 'D:\Hyper-V\Exports'` | VM export is created |
| 18 | Validate export files | Hyper-V host | `Get-ChildItem 'D:\Hyper-V\Exports' -Recurse` | Export contains VM configuration and disk data |
| 19 | Compare exported VM before import | Hyper-V host | `Compare-VM -Path '<exported-vmcx-path>'` | Compatibility report is generated |
| 20 | Import VM by copy with new ID if duplicate lab VM is required | Hyper-V host | `Import-VM -Path '<exported-vmcx-path>' -Copy -GenerateNewId -VirtualMachinePath '<vm-path>' -VhdDestinationPath '<vhdx-path>'` | New VM copy is imported |
| 21 | Import VM by register if files stay in-place | Hyper-V host | `Import-VM -Path '<vmcx-path>' -Register` | Existing VM files are registered |
| 22 | Capture post-change evidence | Hyper-V host | Run post-change verification skeleton | VM lifecycle state is documented |

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_PreChange_Capture_Skeleton

~~~powershell
# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_PreChange_Capture_Skeleton
# Purpose:
# Capture VM state, configuration, disks, checkpoints, integration services, and host storage before lifecycle operations.

$EvidenceRoot = "C:\Admin\HyperV-VMLifecycle"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SmartPagingFilePath, AutomaticStartAction, AutomaticStartDelay, AutomaticStopAction |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.txt")

$VM |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.json") -Encoding UTF8

Write-Host "Capturing full VM configuration..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Format-List * |
    Out-File -FilePath (Join-Path $OutputPath "02-VM-FullConfig-Before.txt") -Encoding UTF8

Write-Host "Capturing VM processor and memory state..." -ForegroundColor Cyan

Get-VMProcessor -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMProcessor-Before.txt")

Get-VMMemory -VMName $VMName |
    Format-List * |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMMemory-Before.txt")

Write-Host "Capturing VM disk attachments..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMHardDiskDrive-Before.txt")

Write-Host "Capturing VHD details..." -ForegroundColor Cyan

foreach ($Disk in Get-VMHardDiskDrive -VMName $VMName) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("06-VHD-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Capturing network adapter state..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "07-VMNetworkAdapter-Before.txt")

Write-Host "Capturing checkpoint inventory..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Id, CreationTime, ParentSnapshotName, SnapshotType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-VMCheckpoints-Before.txt")

Write-Host "Capturing integration service state..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "09-IntegrationServices-Before.txt")

Write-Host "Capturing volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "10-Volumes-Before.txt")

Write-Host "Pre-change lifecycle evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Start_Stop_Save_Reset_Skeleton

~~~powershell
# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Start_Stop_Save_Reset_Skeleton
# Purpose:
# Perform controlled VM lifecycle power operations.

$VMName = "LAB-WIN-SRV01"

# Valid actions:
# Start
# GracefulStop
# TurnOff
# Save
# ResumeFromSaved
# Pause
# ResumeFromPaused
# Restart
# Reset
$Action = "Start"

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Current VM state before action:" -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Status, Uptime |
    Format-List

switch ($Action) {
    "Start" {
        if ($VM.State -eq "Running") {
            Write-Host "VM is already running." -ForegroundColor Yellow
        }
        else {
            Start-VM -Name $VMName
        }
    }

    "GracefulStop" {
        Write-Host "Requesting graceful guest shutdown..." -ForegroundColor Cyan

        Stop-VM -Name $VMName
    }

    "TurnOff" {
        Write-Warning "This is a hard power-off. Use only when graceful shutdown is not possible or approved."

        Stop-VM -Name $VMName -TurnOff
    }

    "Save" {
        Write-Host "Saving VM state..." -ForegroundColor Cyan

        Save-VM -Name $VMName
    }

    "ResumeFromSaved" {
        Write-Host "Starting saved VM..." -ForegroundColor Cyan

        Start-VM -Name $VMName
    }

    "Pause" {
        Write-Host "Pausing VM..." -ForegroundColor Cyan

        Suspend-VM -Name $VMName
    }

    "ResumeFromPaused" {
        Write-Host "Resuming paused VM..." -ForegroundColor Cyan

        Resume-VM -Name $VMName
    }

    "Restart" {
        Write-Host "Restarting VM..." -ForegroundColor Cyan

        Restart-VM -Name $VMName -Force
    }

    "Reset" {
        Write-Warning "Reset is disruptive. It is similar to pressing reset on a physical machine."

        Restart-VM -Name $VMName -Force
    }

    default {
        throw "Unsupported action: $Action"
    }
}

Start-Sleep -Seconds 5

Write-Host "VM state after action:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, State, Status, Uptime |
    Format-List
~~~

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Automatic_Actions_Skeleton

~~~powershell
# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Automatic_Actions_Skeleton
# Purpose:
# Configure how a VM behaves when the Hyper-V host starts or shuts down.

$VMName = "LAB-WIN-SRV01"

# Common values:
# AutomaticStartAction: Nothing, StartIfRunning, Start
# AutomaticStopAction: Save, ShutDown, TurnOff

$AutomaticStartAction = "StartIfRunning"
$AutomaticStartDelay = 30
$AutomaticStopAction = "ShutDown"

Write-Host "Validating target VM..." -ForegroundColor Cyan

Get-VM -Name $VMName -ErrorAction Stop | Out-Null

Write-Host "Current automatic action settings:" -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, AutomaticStartAction, AutomaticStartDelay, AutomaticStopAction |
    Format-List

Write-Host "Configuring automatic actions..." -ForegroundColor Cyan

Set-VM `
    -Name $VMName `
    -AutomaticStartAction $AutomaticStartAction `
    -AutomaticStartDelay $AutomaticStartDelay `
    -AutomaticStopAction $AutomaticStopAction

Write-Host "Automatic action settings after change:" -ForegroundColor Green

Get-VM -Name $VMName |
    Select-Object Name, AutomaticStartAction, AutomaticStartDelay, AutomaticStopAction |
    Format-List
~~~

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Export_VM_Skeleton

~~~powershell
# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Export_VM_Skeleton
# Purpose:
# Export a VM to a planned export folder.

$VMName = "LAB-WIN-SRV01"
$ExportRoot = "D:\Hyper-V\Exports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ExportPath = Join-Path $ExportRoot "$VMName-$Timestamp"

$StopBeforeExport = $false

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Creating export folder..." -ForegroundColor Cyan

New-Item -Path $ExportPath -ItemType Directory -Force | Out-Null

if ($StopBeforeExport) {
    if ($VM.State -ne "Off") {
        Write-Host "Stopping VM before export..." -ForegroundColor Cyan

        Stop-VM -Name $VMName
    }
}
else {
    Write-Host "Exporting VM in current state: $($VM.State)" -ForegroundColor Yellow
}

Write-Host "Capturing VM state before export..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, Id, State, Generation, Path, ConfigurationLocation |
    Format-List

Write-Host "Starting export to: $ExportPath" -ForegroundColor Cyan

Export-VM `
    -Name $VMName `
    -Path $ExportPath

Write-Host "Export complete. Export inventory:" -ForegroundColor Green

Get-ChildItem -Path $ExportPath -Recurse |
    Select-Object FullName, Length, LastWriteTime |
    Format-Table -AutoSize

Write-Host "Exported VM configuration files:" -ForegroundColor Cyan

Get-ChildItem -Path $ExportPath -Recurse -Include "*.vmcx","*.xml" |
    Select-Object FullName, Length, LastWriteTime |
    Format-Table -AutoSize
~~~

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Import_VM_Register_Copy_Restore_Skeleton

~~~powershell
# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Import_VM_Register_Copy_Restore_Skeleton
# Purpose:
# Import an exported VM by register, restore, or copy mode.

$ExportedVMRoot = "D:\Hyper-V\Exports\LAB-WIN-SRV01-20260101-120000"

# Valid modes:
# Register
# Restore
# CopyNewId
$ImportMode = "CopyNewId"

$DestinationVMPath = "D:\Hyper-V\Virtual Machines"
$DestinationVHDPath = "D:\Hyper-V\Virtual Hard Disks"
$DestinationCheckpointPath = "D:\Hyper-V\Checkpoints"
$DestinationSmartPagingPath = "D:\Hyper-V\Smart Paging"

Write-Host "Locating exported VM configuration file..." -ForegroundColor Cyan

$VMConfigFile = Get-ChildItem -Path $ExportedVMRoot -Recurse -Include "*.vmcx" -ErrorAction SilentlyContinue |
    Select-Object -First 1

if (-not $VMConfigFile) {
    $VMConfigFile = Get-ChildItem -Path $ExportedVMRoot -Recurse -Include "*.xml" -ErrorAction SilentlyContinue |
        Select-Object -First 1
}

if (-not $VMConfigFile) {
    throw "No VM configuration file was found under: $ExportedVMRoot"
}

Write-Host "Using VM config file: $($VMConfigFile.FullName)" -ForegroundColor Green

Write-Host "Comparing VM import compatibility..." -ForegroundColor Cyan

$CompatibilityReport = Compare-VM -Path $VMConfigFile.FullName

$CompatibilityReport |
    Format-List |
    Out-Host

if ($CompatibilityReport.Incompatibilities) {
    Write-Warning "Import incompatibilities were found. Review before import."

    $CompatibilityReport.Incompatibilities |
        Format-Table -AutoSize
}

Write-Host "Import mode selected: $ImportMode" -ForegroundColor Cyan

switch ($ImportMode) {
    "Register" {
        Import-VM `
            -Path $VMConfigFile.FullName `
            -Register
    }

    "Restore" {
        Import-VM `
            -Path $VMConfigFile.FullName
    }

    "CopyNewId" {
        Import-VM `
            -Path $VMConfigFile.FullName `
            -Copy `
            -GenerateNewId `
            -VirtualMachinePath $DestinationVMPath `
            -VhdDestinationPath $DestinationVHDPath `
            -SnapshotFilePath $DestinationCheckpointPath `
            -SmartPagingFilePath $DestinationSmartPagingPath
    }

    default {
        throw "Unsupported import mode: $ImportMode. Use Register, Restore, or CopyNewId."
    }
}

Write-Host "VM inventory after import:" -ForegroundColor Green

Get-VM |
    Sort-Object Name |
    Select-Object Name, Id, State, Generation, Path |
    Format-Table -AutoSize
~~~

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_PostChange_Verification_Skeleton

~~~powershell
# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_PostChange_Verification_Skeleton
# Purpose:
# Capture VM lifecycle, automatic action, export, import, disk, checkpoint, and event evidence after lifecycle work.

$EvidenceRoot = "C:\Admin\HyperV-VMLifecycle"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM summary after lifecycle operation..." -ForegroundColor Cyan

Get-VM -Name $VMName -ErrorAction SilentlyContinue |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SmartPagingFilePath, AutomaticStartAction, AutomaticStartDelay, AutomaticStopAction |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-After.txt")

Write-Host "Capturing full VM configuration after lifecycle operation..." -ForegroundColor Cyan

Get-VM -Name $VMName -ErrorAction SilentlyContinue |
    Format-List * |
    Out-File -FilePath (Join-Path $OutputPath "02-VM-FullConfig-After.txt") -Encoding UTF8

Write-Host "Capturing VM disks after lifecycle operation..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMHardDiskDrive-After.txt")

Write-Host "Capturing network adapter state after lifecycle operation..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMNetworkAdapter-After.txt")

Write-Host "Capturing checkpoint state after lifecycle operation..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Id, CreationTime, ParentSnapshotName, SnapshotType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMCheckpoints-After.txt")

Write-Host "Capturing integration services after lifecycle operation..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-IntegrationServices-After.txt")

Write-Host "Capturing export folder inventory if present..." -ForegroundColor Cyan

$ExportRoot = "D:\Hyper-V\Exports"

if (Test-Path $ExportRoot) {
    Get-ChildItem -Path $ExportRoot -Recurse -ErrorAction SilentlyContinue |
        Select-Object FullName, Length, LastWriteTime |
        Sort-Object FullName |
        Format-Table -AutoSize |
        Out-File -FilePath (Join-Path $OutputPath "07-ExportFolderInventory.txt") -Encoding UTF8
}

Write-Host "Capturing recent VMMS events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Write-Host "Capturing recent worker events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-HyperV-Worker-RecentEvents.txt") -Encoding UTF8

Write-Host "Post-change VM lifecycle evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM` | Hyper-V host | Lists all VMs and current states |
| `Get-VM -Name '<vm-name>' \| Select Name, Id, State, Status, Uptime` | Hyper-V host | Confirms target VM identity and state |
| `Start-VM -Name '<vm-name>'` | Hyper-V host | Starts a powered off or saved VM |
| `Stop-VM -Name '<vm-name>'` | Hyper-V host | Requests a graceful shutdown |
| `Stop-VM -Name '<vm-name>' -TurnOff` | Hyper-V host | Forces a hard power off |
| `Save-VM -Name '<vm-name>'` | Hyper-V host | Saves VM runtime state |
| `Suspend-VM -Name '<vm-name>'` | Hyper-V host | Pauses VM execution |
| `Resume-VM -Name '<vm-name>'` | Hyper-V host | Resumes a paused VM |
| `Restart-VM -Name '<vm-name>' -Force` | Hyper-V host | Resets or restarts VM |
| `Get-VM -Name '<vm-name>' \| Select AutomaticStartAction, AutomaticStartDelay, AutomaticStopAction` | Hyper-V host | Confirms automatic host start and stop behavior |
| `Set-VM -Name '<vm-name>' -AutomaticStartAction StartIfRunning -AutomaticStopAction ShutDown` | Hyper-V host | Configures host lifecycle actions |
| `Export-VM -Name '<vm-name>' -Path '<export-path>'` | Hyper-V host | Exports VM files to a folder |
| `Get-ChildItem '<export-path>' -Recurse` | Hyper-V host | Verifies export file inventory |
| `Compare-VM -Path '<vmcx-path>'` | Hyper-V host | Checks whether an exported VM can be imported cleanly |
| `Import-VM -Path '<vmcx-path>' -Register` | Hyper-V host | Registers an existing VM in place |
| `Import-VM -Path '<vmcx-path>' -Copy -GenerateNewId` | Hyper-V host | Imports a duplicate copy with a new VM ID |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Confirms disk attachments after import |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Confirms network adapter state after import |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks VMMS lifecycle and import/export events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM worker lifecycle errors |

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Review current VM state | Hyper-V host | `Get-VM -Name '<vm-name>'` | Current state is known |
| 2 | Stop VM if bad import or lifecycle state must be reversed | Hyper-V host | `Stop-VM -Name '<vm-name>' -TurnOff` | VM is off |
| 3 | Restore automatic start action | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticStartAction '<old-action>' -AutomaticStartDelay '<old-delay>'` | Start behavior returns to baseline |
| 4 | Restore automatic stop action | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticStopAction '<old-action>'` | Stop behavior returns to baseline |
| 5 | Remove mistakenly imported VM registration | Hyper-V host | `Remove-VM -Name '<imported-vm-name>' -Force` | Imported VM object is removed |
| 6 | Delete copied imported files only if approved | Hyper-V host | `Remove-Item '<imported-vm-folder>' -Recurse -Force` | Copied VM files are removed |
| 7 | Reimport from export if original VM was damaged | Hyper-V host | `Import-VM -Path '<vmcx-path>' -Copy -GenerateNewId -VirtualMachinePath '<vm-path>' -VhdDestinationPath '<vhdx-path>'` | VM is restored from export |
| 8 | Reconnect imported VM to correct switch | Hyper-V host | `Connect-VMNetworkAdapter -VMName '<vm-name>' -Name '<adapter-name>' -SwitchName '<switch-name>'` | VM network is restored |
| 9 | Restore checkpoint if lifecycle action caused guest damage | Hyper-V host | `Restore-VMCheckpoint -VMName '<vm-name>' -Name '<checkpoint-name>' -Confirm:$false` | VM returns to checkpoint state |
| 10 | Start VM after rollback | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM starts successfully |
| 11 | Capture rollback evidence | Hyper-V host | `Get-VM -Name '<vm-name>'; Get-VMHardDiskDrive -VMName '<vm-name>'; Get-VMNetworkAdapter -VMName '<vm-name>'` | Rollback state is documented |

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| VM does not start | Insufficient memory, missing VHDX, bad checkpoint chain, or invalid configuration | Check VMMS events, VHDX paths, memory availability, and checkpoint state |
| VM is stuck in saved state | Saved state incompatible after host update or config change | Delete saved state only after approval, then boot cleanly |
| Graceful shutdown fails | Guest integration shutdown service disabled or guest OS hung | Enable Shutdown integration service or use console, then force off only if approved |
| `Stop-VM` hangs | Guest is unresponsive | Use `Stop-VM -TurnOff` as break-glass action |
| Reset causes guest corruption | Reset was used instead of graceful restart | Use guest OS restart or `Stop-VM` when possible |
| VM cannot pause or save | Host storage issue or VM state does not support the operation | Check storage free space and VM state |
| Automatic start does not work | AutomaticStartAction set to Nothing or host did not shut down with VM running | Configure Start or StartIfRunning as required |
| Automatic stop saves instead of shutting down | AutomaticStopAction set to Save | Set AutomaticStopAction to ShutDown if clean guest shutdown is required |
| Export fails | Insufficient space, VM files locked, or bad checkpoint chain | Free storage, stop backup jobs, resolve checkpoint chain issues |
| Export folder is incomplete | Export interrupted | Delete incomplete export and rerun export |
| Import fails compatibility check | Missing virtual switch, invalid path, or hardware mismatch | Run `Compare-VM`, fix incompatibilities, then import |
| Imported VM has duplicate identity | Imported without generating new VM ID or guest identity was not sysprepped | Use `-GenerateNewId` for VM ID, then address guest OS identity separately |
| Imported VM has no network | Switch name missing or different on target host | Reconnect adapter to correct switch |
| Imported VM cannot find disk | VHDX path did not copy or restore correctly | Repair import path, verify files, or reimport with destination paths |
| Register import fails | VM files are not in a valid in-place layout | Use Copy or Restore mode instead |
| Copy import fails | Destination path missing or insufficient space | Create destination folders and confirm free space |
| VM starts but guest services broken after import | Guest OS identity, drivers, network, or integration issue | Check guest console, integration services, and guest network config |
| VMMS event log shows import error | Configuration incompatibility or missing resource | Review `Compare-VM`, repair mapping, then retry import |
| Remove-VM deleted registration but files remain | Normal Hyper-V behavior | Delete VM files manually only after confirming they are no longer needed |
| Deleting files removed the wrong VM data | Poor path validation | Restore from backup/export and use pre-change evidence to identify correct paths |

# 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this lifecycle task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host readiness before lifecycle and export/import operations |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before lifecycle PowerShell cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Provides VM, VHDX, checkpoint, export, and smart paging path planning |
| 04_Create_And_Configure_Virtual_Switches.md | Provides switch names needed after import |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates the VM objects managed by lifecycle operations |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | Resource settings affect VM start, save, and import behavior |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Disk paths and VHDX state are critical to export and import |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Network adapter configuration must be validated after import |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Integration services support graceful shutdown and guest-aware lifecycle operations |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Checkpoint chains affect export, import, and VM startup behavior |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Export/import is portability, not a replacement for real backup |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses lifecycle, export, import, disk, and event state during troubleshooting |