# 13_Configure_VM_Backup_And_Recovery_Workflows

# 13_Configure_VM_Backup_And_Recovery_Workflows_Index
13_Configure_VM_Backup_And_Recovery_Workflows.md
13_Configure_VM_Backup_And_Recovery_Workflows
13_Configure_VM_Backup_And_Recovery_Workflows_Source_Basis
13_Configure_VM_Backup_And_Recovery_Workflows_Mental_Model
13_Configure_VM_Backup_And_Recovery_Workflows_Planning_Table
13_Configure_VM_Backup_And_Recovery_Workflows_Configuration_Checklist
13_Configure_VM_Backup_And_Recovery_Workflows_PreChange_Capture_Skeleton
13_Configure_VM_Backup_And_Recovery_Workflows_Enable_Windows_Server_Backup_Skeleton
13_Configure_VM_Backup_And_Recovery_Workflows_Create_HyperV_Backup_Target_Skeleton
13_Configure_VM_Backup_And_Recovery_Workflows_Run_Manual_Backup_Skeleton
13_Configure_VM_Backup_And_Recovery_Workflows_Recovery_Test_Skeleton
13_Configure_VM_Backup_And_Recovery_Workflows_Backup_Health_Verification_Skeleton
13_Configure_VM_Backup_And_Recovery_Workflows_PostChange_Verification_Skeleton
13_Configure_VM_Backup_And_Recovery_Workflows_Verification_Commands
13_Configure_VM_Backup_And_Recovery_Workflows_Rollback
13_Configure_VM_Backup_And_Recovery_Workflows_Failure_Checks
13_Configure_VM_Backup_And_Recovery_Workflows_Related_Labs

# 13_Configure_VM_Backup_And_Recovery_Workflows_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V backup and restore planning | Supports host-level backup, guest-level backup, and recovery planning |
| Microsoft Learn | Hyper-V production checkpoints | Supports application-consistent backup behavior through guest integration |
| Microsoft Learn | Windows Server Backup | Supports installing backup tooling, creating backup policies, and running manual backups |
| Microsoft Learn | Windows Server Backup PowerShell and `wbadmin` | Supports backup status checks, manual backup runs, and recovery inventory |
| Microsoft Learn | Hyper-V export and import | Supports lab portability and recovery testing, while distinguishing export from backup |
| Microsoft Learn | Hyper-V event logs | Supports troubleshooting backup, checkpoint, VSS, VMMS, and worker failures |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 13_Configure_VM_Backup_And_Recovery_Workflows_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VM backup | A recoverable copy of VM configuration, virtual disks, and required state |
| Host-level backup | Backup taken from the Hyper-V host that protects VM files and Hyper-V-aware state |
| Guest-level backup | Backup agent or OS backup inside the VM, useful for application-aware restore |
| Application-consistent backup | Backup where guest applications are quiesced through VSS or equivalent integration |
| Crash-consistent backup | Backup that captures disk state as if power was lost |
| Production checkpoint | Temporary Hyper-V mechanism used by backup workflows for application-consistent capture |
| Standard checkpoint | Lab rollback feature, not a backup replacement |
| Export | VM portability copy, useful for labs and recovery tests, but not a backup strategy by itself |
| Recovery point | Specific backup version or exported VM state that can be restored |
| Restore test | Controlled recovery validation proving the backup can boot or data can be recovered |
| RPO | Maximum acceptable data loss measured in time |
| RTO | Maximum acceptable time to restore service |
| Backup target | Disk, share, repository, or backup appliance where backup data is stored |
| Backup evidence | Logs, backup set inventory, restore test notes, and event logs proving the workflow works |
| Rollback boundary | This workbook configures backup and test recovery workflow, not enterprise backup product deployment |

# 13_Configure_VM_Backup_And_Recovery_Workflows_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| VM role | DC, file server, app server, test client | `<vm-role>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| Backup scope | Full host, selected VM, selected volume, guest-level | `<backup-scope>` |
| Backup method | Windows Server Backup, backup product, guest backup, export test | `<backup-method>` |
| Backup target | `E:\Backups`, `\\NAS01\HyperVBackups`, backup appliance | `<backup-target>` |
| Backup schedule | Daily 21:00, weekly Sunday, manual lab run | `<backup-schedule>` |
| Retention goal | `7 daily`, `4 weekly`, lab manual only | `<retention-goal>` |
| RPO | `24 hours`, `4 hours`, lab only | `<rpo>` |
| RTO | `2 hours`, `same day`, lab only | `<rto>` |
| Application consistency required | Yes or No | `<yes-no>` |
| Guest VSS required | Yes or No | `<yes-no>` |
| Production checkpoint allowed | Yes or No | `<yes-no>` |
| Standard checkpoint allowed | Lab only, no, yes | `<checkpoint-policy>` |
| Restore test VM name | `RESTORE-LAB-WIN-SRV01` | `<restore-test-vm-name>` |
| Isolated restore switch | `vSwitch-Private-Isolated` | `<restore-test-switch>` |
| Export test path | `D:\Hyper-V\Exports` | `<export-test-path>` |
| Evidence path | `C:\Admin\HyperV-BackupRecovery` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 13_Configure_VM_Backup_And_Recovery_Workflows_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | VM is found |
| 3 | Capture VM state and storage paths | Hyper-V host | `Get-VM -Name '<vm-name>'; Get-VMHardDiskDrive -VMName '<vm-name>'` | VM configuration and disk paths are documented |
| 4 | Capture checkpoint state | Hyper-V host | `Get-VMCheckpoint -VMName '<vm-name>'` | Existing checkpoints are documented |
| 5 | Confirm integration services state | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>'` | Heartbeat, VSS, shutdown, and time sync state are known |
| 6 | Confirm production checkpoint policy | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select CheckpointType` | VM is configured for production checkpoints where required |
| 7 | Install Windows Server Backup if using built-in backup | Hyper-V host | `Install-WindowsFeature Windows-Server-Backup` | Windows Server Backup feature is installed |
| 8 | Confirm backup target exists | Hyper-V host | `Test-Path '<backup-target>'` | Backup destination is reachable |
| 9 | Confirm target free space | Hyper-V host | `Get-Volume` or `Get-ChildItem '<backup-target>'` | Backup target has adequate capacity |
| 10 | Create backup evidence folder | Hyper-V host | `New-Item -Path 'C:\Admin\HyperV-BackupRecovery' -ItemType Directory -Force` | Evidence path exists |
| 11 | Configure selected VM backup method | Hyper-V host | Use backup product or Windows Server Backup workflow | VM protection method is defined |
| 12 | Run manual backup test | Hyper-V host | `wbadmin start backup -backupTarget:<target> -include:<volume> -quiet` | Manual backup job starts |
| 13 | Monitor backup job | Hyper-V host | `Get-WBJob` or backup product console | Backup status is visible |
| 14 | Confirm backup summary | Hyper-V host | `wbadmin get versions` | Backup version is listed |
| 15 | Export VM as lab portability copy if required | Hyper-V host | `Export-VM -Name '<vm-name>' -Path '<export-path>'` | VM export copy is created |
| 16 | Perform isolated restore test | Hyper-V host | `Import-VM -Path '<vmcx-path>' -Copy -GenerateNewId` | Restore test VM imports without overwriting source |
| 17 | Connect restore test VM to isolated switch | Hyper-V host | `Connect-VMNetworkAdapter -VMName '<restore-vm>' -SwitchName '<isolated-switch>'` | Restore VM is isolated from production network |
| 18 | Boot restore test VM | Hyper-V host | `Start-VM -Name '<restore-vm>'` | Restore test VM boots |
| 19 | Validate guest services or data | Guest VM | Application or OS validation commands | Recovery point is proven usable |
| 20 | Capture backup and restore evidence | Hyper-V host | Run post-change verification skeleton | Backup and recovery workflow evidence is saved |
| 21 | Remove restore test VM after validation if required | Hyper-V host | `Remove-VM -Name '<restore-vm>' -Force` | Test VM registration is removed |
| 22 | Preserve or delete restored files according to plan | Hyper-V host | Manual cleanup after verification | Restore test data is handled safely |

# 13_Configure_VM_Backup_And_Recovery_Workflows_PreChange_Capture_Skeleton

~~~powershell
# 13_Configure_VM_Backup_And_Recovery_Workflows_PreChange_Capture_Skeleton
# Purpose:
# Capture VM, disk, checkpoint, integration service, backup feature, storage, and event state before backup workflow changes.

$EvidenceRoot = "C:\Admin\HyperV-BackupRecovery"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, ConfigurationLocation, SnapshotFileLocation, CheckpointType, AutomaticCheckpointsEnabled |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.txt")

$VM |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.json") -Encoding UTF8

Write-Host "Capturing VM disk attachments..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMHardDiskDrive-Before.txt")

Write-Host "Capturing VHDX chain state..." -ForegroundColor Cyan

foreach ($Disk in $HardDisks) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, MinimumSize, ParentPath, Attached, FragmentationPercentage |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("03-VHD-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Capturing checkpoint inventory..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Id, CreationTime, ParentSnapshotName, SnapshotType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMCheckpoints-Before.txt")

Write-Host "Capturing integration services..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-IntegrationServices-Before.txt")

Write-Host "Capturing Windows Server Backup feature state..." -ForegroundColor Cyan

Get-WindowsFeature Windows-Server-Backup |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-WindowsServerBackupFeature-Before.txt")

Write-Host "Capturing volume state..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-Volumes-Before.txt")

Write-Host "Capturing recent backup and Hyper-V events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-WindowsBackup-RecentEvents-Before.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-HyperV-VMMS-RecentEvents-Before.txt") -Encoding UTF8

Write-Host "Pre-change backup and recovery evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 13_Configure_VM_Backup_And_Recovery_Workflows_Enable_Windows_Server_Backup_Skeleton

~~~powershell
# 13_Configure_VM_Backup_And_Recovery_Workflows_Enable_Windows_Server_Backup_Skeleton
# Purpose:
# Install Windows Server Backup tooling for built-in host-level backup workflows.

Write-Host "Checking Windows Server Backup feature state..." -ForegroundColor Cyan

Get-WindowsFeature Windows-Server-Backup |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize

Write-Host "Installing Windows Server Backup..." -ForegroundColor Cyan

Install-WindowsFeature Windows-Server-Backup

Write-Host "Confirming Windows Server Backup feature state..." -ForegroundColor Green

Get-WindowsFeature Windows-Server-Backup |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize

Write-Host "Confirming wbadmin availability..." -ForegroundColor Cyan

wbadmin /?
~~~

# 13_Configure_VM_Backup_And_Recovery_Workflows_Create_HyperV_Backup_Target_Skeleton

~~~powershell
# 13_Configure_VM_Backup_And_Recovery_Workflows_Create_HyperV_Backup_Target_Skeleton
# Purpose:
# Validate or create backup target folders for Hyper-V backup and export testing.
# This does not replace enterprise backup product configuration.

$BackupRoot = "E:\HyperVBackups"
$ExportRoot = "D:\Hyper-V\Exports"
$EvidenceRoot = "C:\Admin\HyperV-BackupRecovery"

$Paths = @(
    $BackupRoot,
    $ExportRoot,
    $EvidenceRoot
)

Write-Host "Creating and validating backup workflow folders..." -ForegroundColor Cyan

foreach ($Path in $Paths) {
    New-Item -Path $Path -ItemType Directory -Force | Out-Null

    [PSCustomObject]@{
        Path = $Path
        Exists = Test-Path $Path
        Root = [System.IO.Path]::GetPathRoot($Path)
    }
}

Write-Host "Capturing volume capacity..." -ForegroundColor Cyan

Get-Volume |
    Sort-Object DriveLetter |
    Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, SizeRemaining, Size |
    Format-Table -AutoSize

Write-Host "Testing write access to backup root..." -ForegroundColor Cyan

$TestFile = Join-Path $BackupRoot ("BackupTargetWriteTest-" + (Get-Date -Format "yyyyMMdd-HHmmss") + ".txt")

"Backup target write test from $env:COMPUTERNAME at $(Get-Date)" |
    Out-File -FilePath $TestFile -Encoding UTF8

Get-Item $TestFile |
    Select-Object FullName, Length, LastWriteTime |
    Format-List

Remove-Item $TestFile -Force

Write-Host "Backup target folder validation completed." -ForegroundColor Green
~~~

# 13_Configure_VM_Backup_And_Recovery_Workflows_Run_Manual_Backup_Skeleton

~~~powershell
# 13_Configure_VM_Backup_And_Recovery_Workflows_Run_Manual_Backup_Skeleton
# Purpose:
# Run a manual Windows Server Backup job for the volume that contains Hyper-V VM files.
# Adjust volume and target values to match the lab.
# For production, use an approved backup platform and perform documented restore tests.

$BackupTarget = "E:"
$IncludedVolume = "D:"

Write-Host "Confirming Windows Server Backup feature..." -ForegroundColor Cyan

$Feature = Get-WindowsFeature Windows-Server-Backup

if ($Feature.InstallState -ne "Installed") {
    throw "Windows Server Backup is not installed. Run the enable skeleton first."
}

Write-Host "Confirming backup target volume exists..." -ForegroundColor Cyan

$TargetDriveLetter = $BackupTarget.TrimEnd(":")
$IncludeDriveLetter = $IncludedVolume.TrimEnd(":")

Get-Volume -DriveLetter $TargetDriveLetter -ErrorAction Stop |
    Select-Object DriveLetter, FileSystemLabel, HealthStatus, SizeRemaining, Size |
    Format-List

Get-Volume -DriveLetter $IncludeDriveLetter -ErrorAction Stop |
    Select-Object DriveLetter, FileSystemLabel, HealthStatus, SizeRemaining, Size |
    Format-List

Write-Warning "This command starts a manual backup of the included volume."
Write-Warning "Make sure the backup target is not the same storage being protected."

$Command = "wbadmin start backup -backupTarget:$BackupTarget -include:$IncludedVolume -quiet"

Write-Host "Running: $Command" -ForegroundColor Cyan

cmd.exe /c $Command

Write-Host "Backup versions after job:" -ForegroundColor Green

wbadmin get versions

Write-Host "Windows Backup event summary:" -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 20 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, Message |
    Format-List
~~~

# 13_Configure_VM_Backup_And_Recovery_Workflows_Recovery_Test_Skeleton

~~~powershell
# 13_Configure_VM_Backup_And_Recovery_Workflows_Recovery_Test_Skeleton
# Purpose:
# Perform a safe lab recovery test by exporting a VM, importing a copy with a new ID, connecting it to an isolated switch, and boot testing.
# This validates the recovery workflow without overwriting the source VM.

$SourceVMName = "LAB-WIN-SRV01"
$RestoreVMName = "RESTORE-LAB-WIN-SRV01"
$ExportRoot = "D:\Hyper-V\Exports"
$RestoreVMPath = "D:\Hyper-V\Virtual Machines"
$RestoreVHDXPath = "D:\Hyper-V\Virtual Hard Disks"
$RestoreCheckpointPath = "D:\Hyper-V\Checkpoints"
$RestoreSmartPagingPath = "D:\Hyper-V\Smart Paging"
$IsolatedSwitchName = "vSwitch-Private-Isolated"

$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ExportPath = Join-Path $ExportRoot "$SourceVMName-RecoveryTest-$Timestamp"

Write-Host "Validating source VM..." -ForegroundColor Cyan

$SourceVM = Get-VM -Name $SourceVMName -ErrorAction Stop

Write-Host "Validating isolated switch..." -ForegroundColor Cyan

if (-not (Get-VMSwitch -Name $IsolatedSwitchName -ErrorAction SilentlyContinue)) {
    New-VMSwitch -Name $IsolatedSwitchName -SwitchType Private
}

Write-Host "Creating export path..." -ForegroundColor Cyan

New-Item -Path $ExportPath -ItemType Directory -Force | Out-Null

Write-Host "Exporting source VM for recovery test..." -ForegroundColor Cyan

Export-VM -Name $SourceVMName -Path $ExportPath

Write-Host "Finding exported VM configuration file..." -ForegroundColor Cyan

$VMConfigFile = Get-ChildItem -Path $ExportPath -Recurse -Include "*.vmcx" -ErrorAction SilentlyContinue |
    Select-Object -First 1

if (-not $VMConfigFile) {
    throw "Exported VM configuration file was not found under: $ExportPath"
}

Write-Host "Comparing import compatibility..." -ForegroundColor Cyan

$Report = Compare-VM -Path $VMConfigFile.FullName

if ($Report.Incompatibilities) {
    Write-Warning "Import compatibility issues found. Review before continuing."

    $Report.Incompatibilities |
        Format-Table -AutoSize
}

Write-Host "Importing VM copy with new ID..." -ForegroundColor Cyan

$ImportedVM = Import-VM `
    -Path $VMConfigFile.FullName `
    -Copy `
    -GenerateNewId `
    -VirtualMachinePath $RestoreVMPath `
    -VhdDestinationPath $RestoreVHDXPath `
    -SnapshotFilePath $RestoreCheckpointPath `
    -SmartPagingFilePath $RestoreSmartPagingPath

Write-Host "Renaming imported VM for restore test..." -ForegroundColor Cyan

Rename-VM -VM $ImportedVM -NewName $RestoreVMName

Write-Host "Connecting restore VM to isolated switch..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $RestoreVMName |
    ForEach-Object {
        Connect-VMNetworkAdapter `
            -VMName $RestoreVMName `
            -Name $_.Name `
            -SwitchName $IsolatedSwitchName
    }

Write-Host "Disabling automatic start on restore test VM..." -ForegroundColor Cyan

Set-VM `
    -Name $RestoreVMName `
    -AutomaticStartAction Nothing `
    -AutomaticStopAction ShutDown

Write-Host "Starting restore test VM..." -ForegroundColor Cyan

Start-VM -Name $RestoreVMName

Start-Sleep -Seconds 15

Write-Host "Restore test VM state:" -ForegroundColor Green

Get-VM -Name $RestoreVMName |
    Select-Object Name, Id, State, Status, Uptime, Path |
    Format-List

Write-Host "Restore test network adapter state:" -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $RestoreVMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
    Format-List

Write-Host "Open VM console for validation:" -ForegroundColor Yellow
Write-Host "vmconnect localhost `"$RestoreVMName`"" -ForegroundColor Yellow
~~~

# 13_Configure_VM_Backup_And_Recovery_Workflows_Backup_Health_Verification_Skeleton

~~~powershell
# 13_Configure_VM_Backup_And_Recovery_Workflows_Backup_Health_Verification_Skeleton
# Purpose:
# Review backup feature state, backup versions, backup events, Hyper-V checkpoint behavior, and restore readiness.

$EvidenceRoot = "C:\Admin\HyperV-BackupRecovery"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $EvidenceRoot "BackupHealth-$env:COMPUTERNAME-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Checking Windows Server Backup feature..." -ForegroundColor Cyan

Get-WindowsFeature Windows-Server-Backup |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "01-WindowsServerBackupFeature.txt")

Write-Host "Checking wbadmin backup versions..." -ForegroundColor Cyan

cmd.exe /c "wbadmin get versions" |
    Out-File -FilePath (Join-Path $OutputPath "02-wbadmin-get-versions.txt") -Encoding UTF8

Write-Host "Checking backup job state if available..." -ForegroundColor Cyan

try {
    Get-WBJob |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "03-WBJob.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "03-WBJob-Error.txt") -Encoding UTF8
}

Write-Host "Capturing backup summary if available..." -ForegroundColor Cyan

try {
    Get-WBSummary |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "04-WBSummary.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "04-WBSummary-Error.txt") -Encoding UTF8
}

Write-Host "Capturing backup event logs..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "05-WindowsBackup-RecentEvents.txt") -Encoding UTF8

Write-Host "Capturing Hyper-V VMMS and Worker events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "06-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-Worker-RecentEvents.txt") -Encoding UTF8

Write-Host "Capturing VM protection state snapshot..." -ForegroundColor Cyan

Get-VM |
    Sort-Object Name |
    Select-Object Name, State, Generation, CheckpointType, AutomaticCheckpointsEnabled, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-VMProtectionSnapshot.txt")

Write-Host "Backup health evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 13_Configure_VM_Backup_And_Recovery_Workflows_PostChange_Verification_Skeleton

~~~powershell
# 13_Configure_VM_Backup_And_Recovery_Workflows_PostChange_Verification_Skeleton
# Purpose:
# Capture final VM backup, recovery test, export, integration service, checkpoint, and event evidence.

$EvidenceRoot = "C:\Admin\HyperV-BackupRecovery"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$RestoreVMName = "RESTORE-LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing source VM state..." -ForegroundColor Cyan

Get-VM -Name $VMName -ErrorAction SilentlyContinue |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path, CheckpointType, AutomaticCheckpointsEnabled |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-SourceVM-State.txt")

Write-Host "Capturing source VM disk state..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-SourceVM-Disks.txt")

Write-Host "Capturing source VM integration services..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-SourceVM-IntegrationServices.txt")

Write-Host "Capturing source VM checkpoint state..." -ForegroundColor Cyan

Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, Id, CreationTime, ParentSnapshotName, SnapshotType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-SourceVM-Checkpoints.txt")

Write-Host "Capturing restore test VM state if present..." -ForegroundColor Cyan

Get-VM -Name $RestoreVMName -ErrorAction SilentlyContinue |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-RestoreTestVM-State.txt")

Get-VMNetworkAdapter -VMName $RestoreVMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "06-RestoreTestVM-Network.txt")

Write-Host "Capturing export folder inventory..." -ForegroundColor Cyan

$ExportRoot = "D:\Hyper-V\Exports"

if (Test-Path $ExportRoot) {
    Get-ChildItem -Path $ExportRoot -Recurse -ErrorAction SilentlyContinue |
        Select-Object FullName, Length, LastWriteTime |
        Sort-Object FullName |
        Out-File -FilePath (Join-Path $OutputPath "07-ExportFolderInventory.txt") -Encoding UTF8
}

Write-Host "Capturing backup versions..." -ForegroundColor Cyan

cmd.exe /c "wbadmin get versions" |
    Out-File -FilePath (Join-Path $OutputPath "08-wbadmin-get-versions.txt") -Encoding UTF8

Write-Host "Capturing Windows Backup events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "09-WindowsBackup-RecentEvents.txt") -Encoding UTF8

Write-Host "Capturing Hyper-V events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "10-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "11-HyperV-Worker-RecentEvents.txt") -Encoding UTF8

Write-Host "Post-change backup and recovery evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 13_Configure_VM_Backup_And_Recovery_Workflows_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-WindowsFeature Windows-Server-Backup` | Hyper-V host | Confirms Windows Server Backup feature state |
| `Install-WindowsFeature Windows-Server-Backup` | Hyper-V host | Installs built-in backup tooling |
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms source VM state |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Confirms protected VM disk paths |
| `Get-VMIntegrationService -VMName '<vm-name>'` | Hyper-V host | Confirms integration services needed for clean backup behavior |
| `Get-VMCheckpoint -VMName '<vm-name>'` | Hyper-V host | Confirms checkpoint inventory before and after backup tests |
| `wbadmin start backup -backupTarget:<target> -include:<volume> -quiet` | Hyper-V host | Starts manual volume-level backup |
| `wbadmin get versions` | Hyper-V host | Lists available backup versions |
| `Get-WBJob` | Hyper-V host | Shows active Windows Server Backup job state |
| `Get-WBSummary` | Hyper-V host | Shows Windows Server Backup summary where available |
| `Export-VM -Name '<vm-name>' -Path '<export-path>'` | Hyper-V host | Creates VM export for lab recovery testing |
| `Compare-VM -Path '<vmcx-path>'` | Hyper-V host | Checks import compatibility |
| `Import-VM -Path '<vmcx-path>' -Copy -GenerateNewId` | Hyper-V host | Imports a restore test copy |
| `Connect-VMNetworkAdapter -VMName '<restore-vm>' -SwitchName '<isolated-switch>'` | Hyper-V host | Isolates restore test VM |
| `Start-VM -Name '<restore-vm>'` | Hyper-V host | Confirms restore test VM can boot |
| `vmconnect localhost '<restore-vm>'` | Hyper-V host | Opens console for restore validation |
| `Get-WinEvent -LogName 'Microsoft-Windows-Backup' -MaxEvents 20` | Hyper-V host | Checks Windows Backup events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks Hyper-V backup and checkpoint events |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks VM worker backup and restore issues |

# 13_Configure_VM_Backup_And_Recovery_Workflows_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop restore test VM | Hyper-V host | `Stop-VM -Name '<restore-vm>' -TurnOff` | Restore test VM is powered off |
| 2 | Remove restore test VM registration | Hyper-V host | `Remove-VM -Name '<restore-vm>' -Force` | Restore test VM object is removed |
| 3 | Delete restore test VM files only if approved | Hyper-V host | `Remove-Item '<restore-vm-folder>' -Recurse -Force` | Restore test files are removed |
| 4 | Delete export copy only if approved | Hyper-V host | `Remove-Item '<export-folder>' -Recurse -Force` | Export folder is removed |
| 5 | Restore VM checkpoint policy | Hyper-V host | `Set-VM -Name '<vm-name>' -CheckpointType '<old-type>'` | Checkpoint policy returns to baseline |
| 6 | Disable automatic checkpoints if enabled by mistake | Hyper-V host | `Set-VM -Name '<vm-name>' -AutomaticCheckpointsEnabled $false` | Automatic checkpoints are disabled |
| 7 | Remove temporary backup evidence only if approved | Hyper-V host | `Remove-Item 'C:\Admin\HyperV-BackupRecovery' -Recurse -Force` | Evidence folder is removed |
| 8 | Remove Windows Server Backup feature if lab cleanup requires it | Hyper-V host | `Uninstall-WindowsFeature Windows-Server-Backup` | Built-in backup feature is removed |
| 9 | Confirm original VM still exists and starts | Hyper-V host | `Get-VM -Name '<vm-name>'; Start-VM -Name '<vm-name>'` | Source VM is intact |
| 10 | Capture rollback evidence | Hyper-V host | `Get-VM; wbadmin get versions; Get-WinEvent -LogName 'Microsoft-Windows-Backup' -MaxEvents 10` | Rollback state is documented |

# 13_Configure_VM_Backup_And_Recovery_Workflows_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Windows Server Backup feature missing | Feature not installed | Run `Install-WindowsFeature Windows-Server-Backup` |
| Backup target not found | Wrong drive letter, disconnected disk, or unreachable share | Validate path with `Test-Path` and confirm storage connectivity |
| Backup target has insufficient space | Target volume too small or old backups consuming capacity | Free space, expand target, or adjust retention |
| Backup fails with VSS error | Guest VSS, host VSS, or integration service issue | Check VSS writers, integration services, and Windows Backup event log |
| VM backup is crash-consistent only | Guest integration or application quiescing failed | Fix guest integration services or use guest-aware backup agent |
| Production checkpoint fails during backup | Guest backup integration unavailable | Check guest OS support, VSS, and Hyper-V integration services |
| Existing checkpoints block clean backup | Long checkpoint chain or merge issue | Remove stale checkpoints and allow merges to complete |
| Export fails | VM files locked, storage full, or checkpoint chain issue | Free space, stop conflicting jobs, resolve checkpoints |
| Import test fails | Missing virtual switch, path mismatch, or incompatible configuration | Run `Compare-VM`, fix paths and switch mappings |
| Restore test VM conflicts with source VM | Imported without new ID or attached to production network | Use `-GenerateNewId` and isolated switch |
| Restore test VM causes duplicate hostname or IP | Guest booted on production network | Keep restore test on private isolated switch |
| Restore test VM does not boot | Missing disk, broken checkpoint chain, or import path issue | Check VM disks, VHDX paths, and VMMS events |
| `wbadmin get versions` shows no backups | Backup never completed or target not attached | Confirm backup job success and target availability |
| `Get-WBJob` returns nothing | No active job | Use Windows Backup event log and `wbadmin get versions` for completed jobs |
| Backup completes but restore was never tested | Paper backup only, no proof of recovery | Perform isolated restore test and capture evidence |
| Guest application data missing after restore | Backup scope did not include application data or app-consistent backup failed | Use application-aware guest backup or fix backup scope |
| Backup window too long | Storage slow, VM too large, or changed data too high | Improve storage, split workloads, schedule off-hours, or use incremental backup product |
| Backup impacts VM performance | Snapshot or checkpoint activity stressing storage | Schedule backups off-hours and monitor storage latency |
| Cleanup deletes source VM files | Wrong path selected during cleanup | Use pre-change evidence, confirm VM paths, and never delete source paths during restore test cleanup |

# 13_Configure_VM_Backup_And_Recovery_Workflows_Related_Labs

| Lab                                                                           | Relationship                                                                           |
| ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| 00_Hyper-V_Index.md                                                           | Places this backup and recovery task in the full Hyper-V suite                         |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Confirms host storage, memory, and role readiness before backup design                 |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Required before Hyper-V backup validation cmdlets are available                        |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md                       | Provides VM, VHDX, checkpoint, smart paging, and export path planning                  |
| 04_Create_And_Configure_Virtual_Switches.md                                   | Provides isolated switch options for safe restore testing                              |
| 05_Create_Generation_1_And_Generation_2_VMs.md                                | Creates the VMs protected by backup workflows                                          |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md                             | Documents VHDX layout and disk state needed for recovery                               |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md                     | Integration services support clean backup and guest consistency                        |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md                | Explains checkpoint behavior used by backup workflows                                  |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md                 | Provides export, import, and restore test operations                                   |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md                        | Helps validate restored Windows guests without network dependency                      |
| 14_Configure_Live_Migration_And_Storage_Migration.md                          | Migration planning affects backup windows and recovery placement                       |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses backup, restore, checkpoint, and VM startup state during recovery troubleshooting |