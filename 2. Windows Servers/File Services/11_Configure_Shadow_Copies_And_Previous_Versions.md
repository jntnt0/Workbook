11_Configure_Shadow_Copies_And_Previous_Versions.md
# Configure_Shadow_Copies_And_Previous_Versions

# Configure_Shadow_Copies_And_Previous_Versions_Index
11_Configure_Shadow_Copies_And_Previous_Versions.md
Configure_Shadow_Copies_And_Previous_Versions
Configure_Shadow_Copies_And_Previous_Versions_Source_Basis
Configure_Shadow_Copies_And_Previous_Versions_Mental_Model
Configure_Shadow_Copies_And_Previous_Versions_Planning_Table
Configure_Shadow_Copies_And_Previous_Versions_Configuration_Checklist
Configure_Shadow_Copies_And_Previous_Versions_Precheck_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Shadow_Storage_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Manual_Shadow_Copy_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Scheduled_Shadow_Copy_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Client_Previous_Versions_Test_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Restore_Test_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Event_Validation_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Post_Change_Validation_Skeleton
Configure_Shadow_Copies_And_Previous_Versions_Verification_Commands
Configure_Shadow_Copies_And_Previous_Versions_Rollback
Configure_Shadow_Copies_And_Previous_Versions_Failure_Checks
Configure_Shadow_Copies_And_Previous_Versions_Related_Labs

# Configure_Shadow_Copies_And_Previous_Versions_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Volume Shadow Copy Service | Point-in-time volume snapshots while applications continue writing |
| Microsoft Learn | vssadmin | Listing shadow copies, writers, providers, and resizing shadow storage |
| Microsoft Learn | vssadmin list shadows | Validating existing shadow copies for a target volume |
| Microsoft Learn | vssadmin resize shadowstorage | Configuring maximum storage used by shadow copies |
| Microsoft Learn | vssadmin delete shadows | Removing client-accessible shadow copies during rollback |
| Microsoft Learn | Get-ScheduledTask | Validating scheduled shadow copy automation |
| Microsoft Learn | Register-ScheduledTask | Creating scheduled PowerShell-based snapshot tasks |
| Microsoft Learn | Get-WinEvent | Validating VSS, VolSnap, and Task Scheduler events |
| Windows Server operational practice | Shadow Copies for Shared Folders | Enabling user self-service restore through Previous Versions |
| Windows Server operational practice | Recovery layering | Shadow Copies help recover user mistakes but do not replace backup |
| Windows Server operational practice | Storage sizing | Shadow copy storage must be capped and monitored to avoid uncontrolled growth |
| Windows Server operational practice | User restore testing | Validate Previous Versions from a client against SMB shares |

# Configure_Shadow_Copies_And_Previous_Versions_Mental_Model
| Concept | Operational Meaning |
|---|---|
| VSS | Volume Shadow Copy Service, the Windows framework that creates point-in-time snapshots |
| Shadow copy | Read-only point-in-time snapshot of a volume |
| Previous Versions | Windows Explorer UI that lets users restore older versions of files or folders |
| Shadow Copies for Shared Folders | File server use case where users recover older SMB share content |
| Source volume | Volume being protected, such as `E:` |
| Shadow storage | Disk space used to store changed blocks for shadow copies |
| Diff area | Storage area holding changes needed to maintain snapshot history |
| Max shadow storage size | Cap that limits how much space shadow copies can consume |
| Snapshot schedule | Recurring task that creates shadow copies at planned times |
| Client-accessible snapshot | Shadow copy type that can appear through Previous Versions |
| Manual snapshot | On-demand shadow copy created for immediate testing |
| Persistent snapshot | Shadow copy that remains available until deleted or aged out |
| VSS writer | Application or service component that helps produce consistent snapshots |
| VSS provider | Software or hardware component that creates and maintains shadow copies |
| VolSnap | Windows component that logs shadow storage and snapshot-related events |
| Restore test | Controlled recovery of a deleted or modified test file |
| Backup difference | Shadow Copies are local convenience recovery, not off-server backup |
| Risk | If the protected volume or server is lost, local shadow copies are lost too |
| First rule | Enable Shadow Copies only after volume, share, ACL, and backup expectations are clear |

# Configure_Shadow_Copies_And_Previous_Versions_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Data volume | `E:` | `<data-volume>` |
| Shadow storage volume | `E:` or `F:` | `<shadow-storage-volume>` |
| Shadow storage max size | `20%` or `20GB` | `<shadow-storage-size>` |
| Department root | `E:\Shares\Departments` | `<department-root>` |
| Home root | `E:\Shares\Home` | `<home-folder-root>` |
| Public root | `E:\Shares\Public` | `<public-share-root>` |
| Test share UNC | `\\FS1\Departments` | `<test-share-unc>` |
| Test folder path | `E:\Shares\Departments\Accounting` | `<test-folder-path>` |
| Test folder UNC | `\\FS1\Departments\Accounting` | `<test-folder-unc>` |
| Test file name | `previous-versions-test.txt` | `<test-file-name>` |
| Snapshot schedule 1 | `7:00 AM` | `<snapshot-time-1>` |
| Snapshot schedule 2 | `12:00 PM` | `<snapshot-time-2>` |
| Snapshot schedule 3 | `5:00 PM` | `<snapshot-time-3>` |
| Schedule task name | `Create-ShadowCopy-E` | `<scheduled-task-name>` |
| Retention stance | Based on shadow storage cap | `<retention-plan>` |
| User restore stance | Users restore files from Previous Versions | `<user-restore-plan>` |
| Admin restore stance | Admin validates with Explorer and snapshots | `<admin-restore-plan>` |
| Backup stance | Shadow copies supplement backup | `<backup-plan>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Client evidence path | `C:\FileServices-Validation-Client` | `<client-evidence-path>` |
| Rollback stance | Remove task, resize storage, optionally delete snapshots | `<rollback-plan>` |
| Next workbook | `12_Backup_Restore_And_Export_File_Server_Config.md` | `<next-task>` |

# Configure_Shadow_Copies_And_Previous_Versions_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm VSS service exists | File Server | `Get-Service VSS` | Volume Shadow Copy service is visible |
| 4 | Confirm target volume | File Server | `Get-Volume -DriveLetter <data-volume-letter>` | Protected data volume is visible |
| 5 | Confirm target paths exist | File Server | `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"` | Protected folders exist |
| 6 | Confirm SMB shares exist | File Server | `Get-SmbShare` | Shares exist for user testing |
| 7 | Capture current shadow storage | File Server | `vssadmin list shadowstorage` | Current shadow storage associations are documented |
| 8 | Capture current shadows | File Server | `vssadmin list shadows /for=<data-volume>` | Existing snapshots are documented |
| 9 | Capture VSS writers | File Server | `vssadmin list writers` | Writer state is documented |
| 10 | Configure shadow storage cap | File Server | `vssadmin resize shadowstorage /For=<data-volume> /On=<shadow-storage-volume> /MaxSize=<shadow-storage-size>` | Shadow storage association is configured |
| 11 | Create manual test shadow copy | File Server | Use manual shadow copy skeleton | Initial shadow copy exists |
| 12 | Validate shadow copy exists | File Server | `vssadmin list shadows /for=<data-volume>` | New snapshot appears |
| 13 | Create scheduled snapshot task | File Server | Use scheduled shadow copy skeleton | Recurring snapshot task exists |
| 14 | Validate scheduled task | File Server | `Get-ScheduledTask -TaskName "<scheduled-task-name>"` | Task is registered |
| 15 | Create test file before snapshot | Client | `New-Item "\\<file-server>\<share>\<folder>\<test-file-name>"` | Test file exists |
| 16 | Create or wait for snapshot | File Server | Manual or scheduled shadow copy | Snapshot captures test file state |
| 17 | Modify test file after snapshot | Client | `Set-Content "\\<file-server>\<share>\<folder>\<test-file-name>"` | File content changes after snapshot |
| 18 | Validate Previous Versions tab | Client | File or folder Properties > Previous Versions | Previous version appears |
| 19 | Restore test file to alternate path | Client | Previous Versions > Open or Copy | Recovery works without overwriting production data |
| 20 | Test deleted file recovery | Client | Delete test file, then restore folder Previous Version | Deleted test file can be recovered |
| 21 | Validate VSS and VolSnap events | File Server | Use event validation skeleton | Snapshot events are visible |
| 22 | Export final snapshot evidence | File Server | Use post-change validation skeleton | Final evidence is saved |
| 23 | Document recovery model | Operator | `Record volume, storage cap, schedule, test file, restore evidence, and caveats` | Previous Versions record is complete |

# Configure_Shadow_Copies_And_Previous_Versions_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture role, volume, share, VSS, and snapshot state before configuration.

$EvidencePath = "C:\FileServices-Validation"

$DataDriveLetter = "E"
$DataVolume = "$DataDriveLetter`:"
$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\11-shadow-copies-precheck-transcript.txt"

# Confirm admin context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-shadow-copies.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname-before-shadow-copies.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-shadow-copies.txt"

# Confirm File Server role.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-shadow-copies.txt"

# Confirm volume state.
Get-Volume |
  Tee-Object "$EvidencePath\volumes-before-shadow-copies.txt"

Get-Volume -DriveLetter $DataDriveLetter |
  Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining |
  Tee-Object "$EvidencePath\data-volume-before-shadow-copies.txt"

# Confirm target paths.
$TargetPaths = @(
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot
)

foreach ($Path in $TargetPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\shadow-copy-target-path-validation-before.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-shadow-copy-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Confirm SMB shares.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-before-shadow-copies.txt"

# Confirm VSS service.
Get-Service VSS |
  Tee-Object "$EvidencePath\vss-service-before-shadow-copies.txt"

# Capture VSS writers and providers.
vssadmin list writers |
  Tee-Object "$EvidencePath\vss-writers-before-shadow-copies.txt"

vssadmin list providers |
  Tee-Object "$EvidencePath\vss-providers-before-shadow-copies.txt"

# Capture shadow storage and existing shadows.
vssadmin list shadowstorage |
  Tee-Object "$EvidencePath\vss-shadowstorage-before.txt"

vssadmin list shadows /for=$DataVolume |
  Tee-Object "$EvidencePath\vss-shadows-before.txt"

# Capture existing scheduled tasks that look related.
Get-ScheduledTask |
  Where-Object {
    $_.TaskName -like "*Shadow*" -or
    $_.TaskName -like "*VSS*" -or
    $_.TaskName -like "*Previous*"
  } |
  Tee-Object "$EvidencePath\scheduled-shadow-tasks-before.txt"

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Shadow_Storage_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: configure shadow copy storage for the protected data volume.
# Warning: resizing shadow storage can cause existing shadow copies to disappear.

$EvidencePath = "C:\FileServices-Validation"

$ForVolume = "E:"
$OnVolume = "E:"
$MaxSize = "20%"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\11-shadow-storage-configuration-transcript.txt"

# Capture current storage.
vssadmin list shadowstorage |
  Tee-Object "$EvidencePath\vss-shadowstorage-before-resize.txt"

# Confirm volume capacity.
Get-Volume -DriveLetter "E" |
  Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining |
  Tee-Object "$EvidencePath\data-volume-capacity-before-shadowstorage-resize.txt"

# Configure shadow storage.
vssadmin resize shadowstorage /For=$ForVolume /On=$OnVolume /MaxSize=$MaxSize |
  Tee-Object "$EvidencePath\vss-shadowstorage-resize-result.txt"

# Confirm final storage.
vssadmin list shadowstorage |
  Tee-Object "$EvidencePath\vss-shadowstorage-after-resize.txt"

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Manual_Shadow_Copy_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create a manual client-accessible shadow copy for immediate Previous Versions testing.

$EvidencePath = "C:\FileServices-Validation"
$Volume = "E:\"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\11-manual-shadow-copy-transcript.txt"

# Confirm VSS service.
Get-Service VSS |
  Tee-Object "$EvidencePath\vss-service-before-manual-shadow.txt"

# Create a client-accessible shadow copy.
# Context is selected for Previous Versions style testing.
$Result = Invoke-CimMethod `
  -ClassName Win32_ShadowCopy `
  -MethodName Create `
  -Arguments @{
    Volume = $Volume
    Context = "ClientAccessible"
  }

$Result |
  Tee-Object "$EvidencePath\manual-shadow-copy-create-result.txt"

# Confirm shadow copies.
vssadmin list shadows /for=E: |
  Tee-Object "$EvidencePath\vss-shadows-after-manual-create.txt"

# Also capture CIM view.
Get-CimInstance Win32_ShadowCopy |
  Select-Object ID,ClientAccessible,ExposedLocally,ExposedRemotely,InstallDate,OriginatingMachine,ProviderID,ServiceMachine,VolumeName |
  Tee-Object "$EvidencePath\win32-shadowcopy-after-manual-create.txt"

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Scheduled_Shadow_Copy_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create recurring scheduled shadow copies for the data volume.
# This creates three daily snapshots. Adjust schedule for the lab.

$EvidencePath = "C:\FileServices-Validation"

$TaskName = "Create-ShadowCopy-E"
$TaskPath = "\FileServices\"
$Volume = "E:\"

$ScriptFolder = "C:\FileServices-Scripts"
$ScriptPath = Join-Path $ScriptFolder "Create-ShadowCopy-E.ps1"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $ScriptFolder

Start-Transcript -Path "$EvidencePath\11-scheduled-shadow-copy-transcript.txt"

# Create snapshot script.
$ScriptContent = @'
$Volume = "E:\"
$Result = Invoke-CimMethod -ClassName Win32_ShadowCopy -MethodName Create -Arguments @{
  Volume = $Volume
  Context = "ClientAccessible"
}

$LogPath = "C:\FileServices-Validation\scheduled-shadowcopy-run.log"
New-Item -ItemType Directory -Force -Path (Split-Path $LogPath) | Out-Null
"$(Get-Date -Format o) Result=$($Result.ReturnValue) ShadowID=$($Result.ShadowID)" | Out-File $LogPath -Append
'@

$ScriptContent |
  Out-File -FilePath $ScriptPath -Encoding UTF8 -Force

# Remove old lab task if recreating.
$ExistingTask = Get-ScheduledTask -TaskName $TaskName -TaskPath $TaskPath -ErrorAction SilentlyContinue

if ($ExistingTask) {
  Unregister-ScheduledTask `
    -TaskName $TaskName `
    -TaskPath $TaskPath `
    -Confirm:$false
}

# Create daily triggers.
$Trigger1 = New-ScheduledTaskTrigger -Daily -At 7:00am
$Trigger2 = New-ScheduledTaskTrigger -Daily -At 12:00pm
$Trigger3 = New-ScheduledTaskTrigger -Daily -At 5:00pm

# Create action.
$Action = New-ScheduledTaskAction `
  -Execute "PowerShell.exe" `
  -Argument "-NoProfile -ExecutionPolicy Bypass -File `"$ScriptPath`""

# Run as SYSTEM.
$Principal = New-ScheduledTaskPrincipal `
  -UserId "SYSTEM" `
  -RunLevel Highest

# Basic settings.
$Settings = New-ScheduledTaskSettingsSet `
  -Compatibility Win8 `
  -AllowStartIfOnBatteries `
  -StartWhenAvailable

# Register scheduled task.
Register-ScheduledTask `
  -TaskName $TaskName `
  -TaskPath $TaskPath `
  -Action $Action `
  -Trigger @($Trigger1,$Trigger2,$Trigger3) `
  -Principal $Principal `
  -Settings $Settings `
  -Description "Create client-accessible VSS shadow copies for E: data volume."

# Confirm task.
Get-ScheduledTask -TaskName $TaskName -TaskPath $TaskPath |
  Tee-Object "$EvidencePath\scheduled-shadow-copy-task-after-create.txt"

Get-ScheduledTaskInfo -TaskName $TaskName -TaskPath $TaskPath |
  Tee-Object "$EvidencePath\scheduled-shadow-copy-task-info-after-create.txt"

# Optional immediate test run.
Start-ScheduledTask -TaskName $TaskName -TaskPath $TaskPath

Start-Sleep -Seconds 10

Get-ScheduledTaskInfo -TaskName $TaskName -TaskPath $TaskPath |
  Tee-Object "$EvidencePath\scheduled-shadow-copy-task-info-after-test-run.txt"

vssadmin list shadows /for=E: |
  Tee-Object "$EvidencePath\vss-shadows-after-scheduled-task-test.txt"

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Client_Previous_Versions_Test_Skeleton
```powershell
# Run from a Windows client as an authorized user.
# Purpose: create, modify, and recover a test file through the Previous Versions workflow.

$FileServerName = "FS1"
$ShareName = "Departments"
$DepartmentName = "Accounting"

$TestFolderUNC = "\\$FileServerName\$ShareName\$DepartmentName"
$TestFileName = "previous-versions-test.txt"
$TestFileUNC = Join-Path $TestFolderUNC $TestFileName

$EvidencePath = "C:\FileServices-Validation-Client"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\11-client-previous-versions-test-transcript.txt"

# Capture client identity.
hostname |
  Tee-Object "$EvidencePath\client-hostname-previous-versions.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami-previous-versions.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups-previous-versions.txt"

# Confirm SMB reachability.
Resolve-DnsName $FileServerName |
  Tee-Object "$EvidencePath\client-resolve-fileserver-previous-versions.txt"

Test-NetConnection $FileServerName -Port 445 |
  Tee-Object "$EvidencePath\client-test-smb-445-previous-versions.txt"

Test-Path $TestFolderUNC |
  Tee-Object "$EvidencePath\client-test-folder-unc-previous-versions.txt"

# Create baseline test file.
"Version 1 created at $(Get-Date)" |
  Out-File $TestFileUNC -Force

Get-Content $TestFileUNC |
  Tee-Object "$EvidencePath\previous-versions-test-file-version1.txt"

# Stop here and create a manual or scheduled shadow copy on the file server.
# After shadow copy exists, modify the file.

"Version 2 modified at $(Get-Date)" |
  Out-File $TestFileUNC -Force

Get-Content $TestFileUNC |
  Tee-Object "$EvidencePath\previous-versions-test-file-version2.txt"

# Manual GUI validation:
# 1. In File Explorer, browse to $TestFolderUNC.
# 2. Right-click the test file or parent folder.
# 3. Open Properties.
# 4. Select Previous Versions.
# 5. Open or Copy the previous version to a safe alternate location.
# 6. Do not overwrite production data during the first test.

# Capture current SMB connection state.
Get-SmbConnection |
  Where-Object { $_.ServerName -like "$FileServerName*" -or $_.ServerName -eq $FileServerName } |
  Select-Object ServerName,ShareName,Dialect,Signed,Encrypted,UserName |
  Tee-Object "$EvidencePath\client-smb-connection-previous-versions.txt"

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Restore_Test_Skeleton
```powershell
# Run from a Windows client or admin workstation.
# Purpose: validate deleted file recovery through Previous Versions.

$FileServerName = "FS1"
$ShareName = "Departments"
$DepartmentName = "Accounting"

$TestFolderUNC = "\\$FileServerName\$ShareName\$DepartmentName"
$DeletedFileName = "previous-versions-delete-restore-test.txt"
$DeletedFileUNC = Join-Path $TestFolderUNC $DeletedFileName

$EvidencePath = "C:\FileServices-Validation-Client"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\11-client-deleted-file-restore-test-transcript.txt"

# Create a file before snapshot.
"Deleted file restore test created at $(Get-Date)" |
  Out-File $DeletedFileUNC -Force

Get-Content $DeletedFileUNC |
  Tee-Object "$EvidencePath\deleted-file-restore-test-before-delete.txt"

# Stop here and create a shadow copy on the file server.
# After shadow copy exists, delete the file.

Remove-Item $DeletedFileUNC -Force

Test-Path $DeletedFileUNC |
  Tee-Object "$EvidencePath\deleted-file-testpath-after-delete.txt"

# Manual GUI validation:
# 1. In File Explorer, browse to $TestFolderUNC.
# 2. Right-click the parent folder.
# 3. Open Properties.
# 4. Select Previous Versions.
# 5. Open a version from before the deletion.
# 6. Copy the deleted file back to a safe restore location or original location.
# 7. Confirm recovered file content.

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Event_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate VSS, VolSnap, and Task Scheduler events after snapshot creation.

$EvidencePath = "C:\FileServices-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\11-shadow-copy-event-validation-transcript.txt"

# VSS events in Application log.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "VSS"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\vss-application-events-last-8-hours.txt"

# VolSnap events in System log.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "System"
    ProviderName = "VolSnap"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\volsnap-system-events-last-8-hours.txt"

# Task Scheduler Operational log for scheduled snapshot task.
Get-WinEvent `
  -LogName "Microsoft-Windows-TaskScheduler/Operational" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Where-Object {
    $_.Message -like "*Create-ShadowCopy-E*" -or
    $_.Message -like "*FileServices*"
  } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\task-scheduler-shadow-copy-events.txt"

# Export event evidence.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "VSS"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\vss-application-events-last-8-hours.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Post_Change_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture final shadow copy, storage, schedule, event, and share evidence.

$EvidencePath = "C:\FileServices-Validation"

$DataDriveLetter = "E"
$DataVolume = "$DataDriveLetter`:"
$TaskName = "Create-ShadowCopy-E"
$TaskPath = "\FileServices\"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\11-post-change-shadow-copy-validation-transcript.txt"

# VSS service, writers, providers.
Get-Service VSS |
  Tee-Object "$EvidencePath\vss-service-final.txt"

vssadmin list writers |
  Tee-Object "$EvidencePath\vss-writers-final.txt"

vssadmin list providers |
  Tee-Object "$EvidencePath\vss-providers-final.txt"

# Shadow storage and shadows.
vssadmin list shadowstorage |
  Tee-Object "$EvidencePath\vss-shadowstorage-final.txt"

vssadmin list shadows /for=$DataVolume |
  Tee-Object "$EvidencePath\vss-shadows-final.txt"

# CIM view.
Get-CimInstance Win32_ShadowCopy |
  Select-Object ID,ClientAccessible,ExposedLocally,ExposedRemotely,InstallDate,OriginatingMachine,ProviderID,ServiceMachine,VolumeName |
  Tee-Object "$EvidencePath\win32-shadowcopy-final.txt"

# Volume capacity.
Get-Volume -DriveLetter $DataDriveLetter |
  Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining |
  Tee-Object "$EvidencePath\data-volume-capacity-final-shadow-copies.txt"

# SMB shares and target paths.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-final-shadow-copies.txt"

# Scheduled task evidence.
Get-ScheduledTask -TaskName $TaskName -TaskPath $TaskPath -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scheduled-shadow-copy-task-final.txt"

Get-ScheduledTaskInfo -TaskName $TaskName -TaskPath $TaskPath -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scheduled-shadow-copy-task-info-final.txt"

# Script evidence.
Get-ChildItem "C:\FileServices-Scripts" -Filter "*ShadowCopy*" -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\scheduled-shadow-copy-script-files-final.txt"

# Event evidence.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "VSS"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\vss-events-final-shadow-copies.txt"

Get-WinEvent `
  -FilterHashtable @{
    LogName = "System"
    ProviderName = "VolSnap"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\volsnap-events-final-shadow-copies.txt"

Stop-Transcript
```

# Configure_Shadow_Copies_And_Previous_Versions_Verification_Commands
```powershell
# File Server and volume baseline
Get-WindowsFeature FS-FileServer
Get-Service VSS
Get-Volume
Get-Volume -DriveLetter <data-drive-letter>
Get-SmbShare

# Target paths
Test-Path "<department-root>"
Test-Path "<home-folder-root>"
Test-Path "<test-folder-path>"
icacls "<test-folder-path>"

# VSS writers and providers
vssadmin list writers
vssadmin list providers

# Shadow storage
vssadmin list shadowstorage
vssadmin resize shadowstorage /For=<data-volume> /On=<shadow-storage-volume> /MaxSize=<shadow-storage-size>

# Existing shadow copies
vssadmin list shadows
vssadmin list shadows /for=<data-volume>

# Manual shadow copy using CIM
Invoke-CimMethod -ClassName Win32_ShadowCopy -MethodName Create -Arguments @{Volume="<data-volume>\"; Context="ClientAccessible"}

# CIM shadow copy inventory
Get-CimInstance Win32_ShadowCopy |
  Select-Object ID,ClientAccessible,InstallDate,VolumeName

# Scheduled task validation
Get-ScheduledTask -TaskName "<scheduled-task-name>"
Get-ScheduledTaskInfo -TaskName "<scheduled-task-name>"
Start-ScheduledTask -TaskName "<scheduled-task-name>"

# Client SMB path validation
Test-NetConnection "<file-server-name>" -Port 445
Test-Path "\\<file-server-name>\<share-name>\<folder>"
New-Item -ItemType File -Path "\\<file-server-name>\<share-name>\<folder>\previous-versions-test.txt"
Set-Content "\\<file-server-name>\<share-name>\<folder>\previous-versions-test.txt" "Modified after snapshot"

# Previous Versions GUI validation
# File Explorer > UNC path > right-click file or folder > Properties > Previous Versions

# Delete shadow copies during rollback
vssadmin delete shadows /for=<data-volume> /oldest
vssadmin delete shadows /for=<data-volume> /all

# Events
Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="VSS"; StartTime=(Get-Date).AddHours(-8)}
Get-WinEvent -FilterHashtable @{LogName="System"; ProviderName="VolSnap"; StartTime=(Get-Date).AddHours(-8)}
Get-WinEvent -LogName "Microsoft-Windows-TaskScheduler/Operational" -MaxEvents 100

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*shadow*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*vss*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*volsnap*"
```

# Configure_Shadow_Copies_And_Previous_Versions_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current snapshot state before rollback | File Server | `vssadmin list shadows /for=<data-volume>` | Existing shadow copies are documented |
| 2 | Capture current shadow storage state | File Server | `vssadmin list shadowstorage` | Storage association is documented |
| 3 | Capture scheduled task state | File Server | `Get-ScheduledTask -TaskName "<scheduled-task-name>"` | Snapshot task state is documented |
| 4 | Disable scheduled snapshot task | File Server | `Disable-ScheduledTask -TaskName "<scheduled-task-name>" -TaskPath "\FileServices\"` | New scheduled snapshots stop |
| 5 | Remove scheduled snapshot task if lab rollback requires it | File Server | `Unregister-ScheduledTask -TaskName "<scheduled-task-name>" -TaskPath "\FileServices\" -Confirm:$false` | Task is removed |
| 6 | Remove snapshot script if no longer used | File Server | `Remove-Item "C:\FileServices-Scripts\Create-ShadowCopy-E.ps1" -Force` | Script is removed |
| 7 | Delete oldest shadow copy if freeing space | File Server | `vssadmin delete shadows /for=<data-volume> /oldest` | Oldest snapshot is removed |
| 8 | Delete all lab shadow copies if required | File Server | `vssadmin delete shadows /for=<data-volume> /all` | All client-accessible snapshots for the volume are removed |
| 9 | Reduce or remove shadow storage cap | File Server | `vssadmin resize shadowstorage /For=<data-volume> /On=<shadow-storage-volume> /MaxSize=<new-size>` | Shadow storage is resized |
| 10 | Restore test file state if needed | Client / File Server | Copy from saved test output or backup | Test data returns to expected state |
| 11 | Validate rollback | File Server | `vssadmin list shadows /for=<data-volume>`; `Get-ScheduledTask` | Snapshot and schedule state match rollback target |
| 12 | Document rollback result | Operator | `Record deleted shadows, task removal, storage resize, and test result` | Rollback record is complete |

# Configure_Shadow_Copies_And_Previous_Versions_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Previous Versions tab shows no versions | No client-accessible shadow copy exists | `vssadmin list shadows /for=<data-volume>` | Create a manual or scheduled shadow copy |
| Previous Versions tab shows no versions | User is checking wrong share or folder | Compare UNC path to protected volume | Test the folder on the protected volume |
| Manual shadow copy fails | VSS service problem | `Get-Service VSS`; `vssadmin list writers` | Start VSS and repair writer errors |
| Manual shadow copy fails | Volume argument is wrong | `Get-Volume`; CIM command arguments | Use volume format such as `E:\` |
| Shadow copies disappear | Shadow storage too small or resized | `vssadmin list shadowstorage`; VolSnap events | Increase MaxSize and recreate snapshots |
| Shadow copies consume too much space | Shadow storage cap too large or UNBOUNDED | `vssadmin list shadowstorage`; `Get-Volume` | Set a bounded MaxSize |
| Scheduled snapshots do not run | Scheduled task disabled or bad trigger | `Get-ScheduledTask`; `Get-ScheduledTaskInfo` | Enable task and correct triggers |
| Scheduled snapshots fail | Script path or execution policy issue | Task Scheduler Operational log | Correct script path and action arguments |
| Scheduled snapshots fail | Task not running as elevated principal | `Get-ScheduledTask` principal details | Run task as SYSTEM or approved admin service account |
| Users cannot restore files | NTFS or share permissions block access | `icacls`; `Get-SmbShareAccess`; user token | Fix access permissions |
| User restores wrong version | Too many similar snapshots or unclear test process | Snapshot times and file timestamps | Document snapshot time and test file content |
| Deleted file not visible in Previous Versions | Snapshot was created after deletion | Snapshot time versus deletion time | Create snapshots before changes and retest |
| VSS writer reports failed state | Application writer issue | `vssadmin list writers` | Restart related service or server during maintenance |
| VolSnap warns about low shadow storage | Diff area too small | VolSnap System events | Increase shadow storage or reduce churn |
| High-change folders lose older versions quickly | Volume churn exceeds shadow storage | `vssadmin list shadowstorage`; report file change rate | Increase storage or adjust schedule |
| Shadow copies treated as backup | Misunderstanding recovery scope | Backup inventory and restore plan | Use task 12 backup/export for real recovery |
| `vssadmin delete shadows` cannot delete snapshot | Snapshot is outside allowed context | Command output and snapshot type | Use proper VSS management tool for that snapshot type |
| Evidence missing | Validation skeleton not run | `Test-Path "<evidence-path>"` | Re-run precheck or post-change validation skeleton |

# Configure_Shadow_Copies_And_Previous_Versions_Related_Labs
| Related Lab                                                      | Relationship                                                                           |
| ---------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `00_File_Services_Index.md`                                      | Defines where Shadow Copies fit in the File Services suite                             |
| `01_Install_File_Server_Role_And_Management_Tools.md`            | Provides File Server role and management baseline                                      |
| `02_Prepare_File_Server_Storage_Volumes_And_Folders.md`          | Provides the data volume protected by shadow copies                                    |
| `03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`           | Controls who can read, modify, and restore files                                       |
| `04_Create_SMB_Shares_And_Share_Permissions.md`                  | Publishes protected folders over SMB for Previous Versions testing                     |
| `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`  | Controls user visibility of folders that may have previous versions                    |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`   | Provides secure SMB transport and audit evidence around restore tests                  |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`        | Provides common data targets for user self-service restore                             |
| `08_Configure_File_Screening_With_FSRM.md`                       | Adds file governance to protected shared folders                                       |
| `09_Configure_Storage_Quotas_With_FSRM.md`                       | Requires quota and snapshot storage planning because both consume volume capacity      |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | Identifies stale or large data before snapshot and backup planning                     |
| `12_Backup_Restore_And_Export_File_Server_Config.md`             | Provides real backup and restore beyond local Previous Versions                        |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`   | Helps validate active users and open files during restore testing                      |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`          | Helps distinguish Previous Versions issues from SMB and NTFS access issues             |
| `25_Migrate_File_Server_Data_Shares_And_Permissions.md`          | Requires planning because local shadow copies usually do not migrate like normal files |