12_Backup_Restore_And_Export_File_Server_Config.md
# Backup_Restore_And_Export_File_Server_Config

# Backup_Restore_And_Export_File_Server_Config_Index
12_Backup_Restore_And_Export_File_Server_Config.md
Backup_Restore_And_Export_File_Server_Config
Backup_Restore_And_Export_File_Server_Config_Source_Basis
Backup_Restore_And_Export_File_Server_Config_Mental_Model
Backup_Restore_And_Export_File_Server_Config_Planning_Table
Backup_Restore_And_Export_File_Server_Config_Configuration_Checklist
Backup_Restore_And_Export_File_Server_Config_Precheck_Skeleton
Backup_Restore_And_Export_File_Server_Config_Windows_Server_Backup_Install_Skeleton
Backup_Restore_And_Export_File_Server_Config_Data_Backup_Skeleton
Backup_Restore_And_Export_File_Server_Config_File_Restore_Test_Skeleton
Backup_Restore_And_Export_File_Server_Config_SMB_Share_Config_Export_Skeleton
Backup_Restore_And_Export_File_Server_Config_NTFS_ACL_Backup_And_Restore_Skeleton
Backup_Restore_And_Export_File_Server_Config_FSRM_Config_Export_Skeleton
Backup_Restore_And_Export_File_Server_Config_Disaster_Rebuild_Export_Package_Skeleton
Backup_Restore_And_Export_File_Server_Config_Post_Change_Validation_Skeleton
Backup_Restore_And_Export_File_Server_Config_Verification_Commands
Backup_Restore_And_Export_File_Server_Config_Rollback
Backup_Restore_And_Export_File_Server_Config_Failure_Checks
Backup_Restore_And_Export_File_Server_Config_Related_Labs

# Backup_Restore_And_Export_File_Server_Config_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Windows Server Backup | Backing up and restoring Windows Server volumes and file data |
| Microsoft Learn | Install-WindowsFeature | Installing Windows Server Backup management tools |
| Microsoft Learn | wbadmin | Running backup and recovery operations from command line |
| Microsoft Learn | WindowsServerBackup PowerShell module | Building backup policies, backup targets, and backup jobs |
| Microsoft Learn | Get-SmbShare | Exporting SMB share names, paths, descriptions, and settings |
| Microsoft Learn | Get-SmbShareAccess | Exporting share-level permissions |
| Microsoft Learn | New-SmbShare | Recreating SMB shares during rebuild |
| Microsoft Learn | Grant-SmbShareAccess | Restoring share-level permissions |
| Microsoft Learn | icacls | Saving and restoring NTFS ACLs |
| Microsoft Learn | Get-FsrmQuota / Get-FsrmFileScreen / Get-FsrmStorageReport | Exporting FSRM quota, screening, and reporting state |
| Microsoft Learn | Export-Clixml / Import-Clixml | Preserving PowerShell object evidence for rebuild documentation |
| Windows Server operational practice | Restore testing | Backups are not trusted until restore has been tested |
| Windows Server operational practice | Configuration export | Data backup must be paired with SMB, ACL, FSRM, and schedule exports |
| Windows Server operational practice | Disaster rebuild package | Keep enough evidence to rebuild file services on a clean server |

# Backup_Restore_And_Export_File_Server_Config_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Data backup | Backup of user files, department folders, home folders, public shares, app shares, and staged data |
| Configuration export | Export of SMB shares, share permissions, NTFS ACLs, FSRM objects, schedules, and evidence |
| System state | Backup category for OS and role metadata, useful for server recovery but not a substitute for file-level restore testing |
| Volume backup | Backup of an entire volume such as `E:` |
| File-level restore | Recovery of a specific folder or file from backup |
| Alternate restore path | Safe restore location used to test recovery without overwriting production data |
| SMB share config | Share names, paths, descriptions, caching, ABE, encryption, and share permissions |
| NTFS ACL backup | Saved file system permissions using `icacls /save` |
| ACL restore | Reapplying saved NTFS permissions using `icacls /restore` from the correct parent path |
| FSRM config export | Documentation of quotas, auto quotas, file screens, templates, file groups, reports, and file management jobs |
| Backup target | External disk, backup volume, or network share where backups are stored |
| Recovery point | Specific backup version available for restore |
| Restore validation | Proof that backup content can be recovered and read |
| Rebuild package | Folder containing exports, scripts, ACLs, reports, and validation output needed to rebuild services |
| RPO | Recovery Point Objective, how much data loss is acceptable |
| RTO | Recovery Time Objective, how fast service must be restored |
| Shadow Copies difference | Shadow Copies provide local Previous Versions, but real backup must survive server or volume loss |
| First rule | A file server backup is incomplete unless data, ACLs, shares, FSRM settings, and restore evidence are all captured |

# Backup_Restore_And_Export_File_Server_Config_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Data volume | `E:` | `<data-volume>` |
| Data root | `E:\Shares` | `<data-root>` |
| Department root | `E:\Shares\Departments` | `<department-root>` |
| Home root | `E:\Shares\Home` | `<home-folder-root>` |
| Public root | `E:\Shares\Public` | `<public-share-root>` |
| Apps root | `E:\Shares\Apps` | `<app-share-root>` |
| FSRM report path | `E:\FSRM-Reports` | `<fsrm-report-path>` |
| Shadow copy schedule task | `Create-ShadowCopy-E` | `<shadow-copy-task-name>` |
| Backup target type | Network share, local disk, or backup disk | `<backup-target-type>` |
| Backup target path | `\\BACKUP1\FileServerBackups\FS1` | `<backup-target-path>` |
| Backup credential | `CORP\backup.svc` | `<backup-credential>` |
| Backup schedule | Daily 10:00 PM | `<backup-schedule>` |
| Backup include set | `E:` and System State | `<backup-include-set>` |
| Restore test path | `E:\Restore-Test` | `<restore-test-path>` |
| Restore test file | `restore-validation.txt` | `<restore-test-file>` |
| Export root | `C:\FileServices-Exports` | `<export-root>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| ACL backup path | `C:\FileServices-Exports\ACL` | `<acl-backup-path>` |
| SMB export path | `C:\FileServices-Exports\SMB` | `<smb-export-path>` |
| FSRM export path | `C:\FileServices-Exports\FSRM` | `<fsrm-export-path>` |
| Rebuild package path | `C:\FileServices-Exports\RebuildPackage` | `<rebuild-package-path>` |
| RPO | 24 hours | `<rpo>` |
| RTO | 4 hours | `<rto>` |
| Restore validation cadence | Monthly | `<restore-test-cadence>` |
| Next workbook | `13_Configure_DFS_Namespaces.md` | `<next-task>` |

# Backup_Restore_And_Export_File_Server_Config_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm data volume | File Server | `Get-Volume -DriveLetter <data-drive-letter>` | Data volume is visible and healthy |
| 4 | Confirm share roots | File Server | `Test-Path "<data-root>"`; `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"` | File server data paths exist |
| 5 | Confirm SMB shares | File Server | `Get-SmbShare` | Shares are visible |
| 6 | Confirm FSRM state if used | File Server | `Get-WindowsFeature FS-Resource-Manager`; `Get-Service SrmSvc` | FSRM state is known |
| 7 | Confirm Shadow Copy state if used | File Server | `vssadmin list shadows /for=<data-volume>` | Snapshot state is documented |
| 8 | Install Windows Server Backup | File Server | `Install-WindowsFeature Windows-Server-Backup -IncludeManagementTools` | Backup tooling is installed |
| 9 | Confirm backup cmdlets | File Server | `Get-Command -Module WindowsServerBackup` | Backup cmdlets are available |
| 10 | Confirm backup target access | File Server | `Test-Path "<backup-target-path>"` | Backup target is reachable |
| 11 | Capture pre-backup evidence | File Server | Use precheck skeleton | Baseline state is saved |
| 12 | Run data volume backup | File Server | `wbadmin start backup -backupTarget:<target> -include:<data-volume> -quiet` | Backup completes successfully |
| 13 | List backup versions | File Server | `wbadmin get versions -backupTarget:<target>` | Recovery points are visible |
| 14 | Restore test file to alternate path | File Server | `wbadmin start recovery ... -recoveryTarget:<restore-test-path>` | File or folder restores safely |
| 15 | Validate restored content | File Server | `Get-ChildItem "<restore-test-path>" -Recurse` | Restored data is readable |
| 16 | Export SMB share inventory | File Server | Use SMB export skeleton | Share settings and permissions are exported |
| 17 | Export NTFS ACLs | File Server | `icacls "<data-root>" /save "<acl-backup-path>\data-root-acl.txt" /t /c` | ACL backup file exists |
| 18 | Export FSRM configuration | File Server | Use FSRM export skeleton | Quotas, screens, reports, and jobs are exported |
| 19 | Export scheduled tasks related to file services | File Server | `Get-ScheduledTask \| Where TaskName -like "*Shadow*"` | Related task state is saved |
| 20 | Export event evidence | File Server | `Get-WinEvent` queries | Backup, VSS, FSRM, and SMB events are saved |
| 21 | Build rebuild package | File Server | Use disaster rebuild package skeleton | Exports are collected into one package |
| 22 | Document restore test result | Operator | `Record backup version, source path, restore path, and validation output` | Restore evidence is complete |
| 23 | Document rebuild order | Operator | `Data volume, folders, ACLs, SMB shares, FSRM, Shadow Copies, scheduled tasks` | Disaster rebuild procedure is clear |

# Backup_Restore_And_Export_File_Server_Config_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture file server, data, share, FSRM, shadow copy, backup, and export state before changes.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"

$DataDriveLetter = "E"
$DataVolume = "$DataDriveLetter`:"
$DataRoot = "E:\Shares"
$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"
$RestoreTestPath = "E:\Restore-Test"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $ExportRoot

Start-Transcript -Path "$EvidencePath\12-backup-restore-export-precheck-transcript.txt"

# Identity and server state.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-backup-export.txt"

hostname |
  Tee-Object "$EvidencePath\hostname-before-backup-export.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-backup-export.txt"

# Roles and features.
Get-WindowsFeature FS-FileServer,FS-Resource-Manager,Windows-Server-Backup |
  Tee-Object "$EvidencePath\features-before-backup-export.txt"

# Volume and path state.
Get-Volume |
  Tee-Object "$EvidencePath\volumes-before-backup-export.txt"

Get-Volume -DriveLetter $DataDriveLetter |
  Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining |
  Tee-Object "$EvidencePath\data-volume-before-backup-export.txt"

$Paths = @(
  $DataRoot,
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $AppsRoot
)

foreach ($Path in $Paths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\file-server-path-validation-before-backup-export.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-backup-export-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# SMB state.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-before-backup-export.txt"

Get-SmbShare |
  ForEach-Object {
    Get-SmbShareAccess -Name $_.Name -ErrorAction SilentlyContinue
  } |
  Tee-Object "$EvidencePath\smb-share-access-before-backup-export.txt"

# FSRM state if available.
Get-Service SrmSvc -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-service-before-backup-export.txt"

Get-FsrmQuota -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quotas-before-backup-export.txt"

Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screens-before-backup-export.txt"

Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-storage-reports-before-backup-export.txt"

Get-FsrmFileManagementJob -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-management-jobs-before-backup-export.txt"

# VSS and shadow copy state.
Get-Service VSS |
  Tee-Object "$EvidencePath\vss-service-before-backup-export.txt"

vssadmin list writers |
  Tee-Object "$EvidencePath\vss-writers-before-backup-export.txt"

vssadmin list shadowstorage |
  Tee-Object "$EvidencePath\vss-shadowstorage-before-backup-export.txt"

vssadmin list shadows /for=$DataVolume |
  Tee-Object "$EvidencePath\vss-shadows-before-backup-export.txt"

# Backup state.
Get-Command wbadmin -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\wbadmin-command-before-backup-export.txt"

Get-Command -Module WindowsServerBackup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\windows-server-backup-cmdlets-before.txt"

# Scheduled tasks related to file services.
Get-ScheduledTask |
  Where-Object {
    $_.TaskName -like "*Shadow*" -or
    $_.TaskName -like "*Backup*" -or
    $_.TaskName -like "*FSRM*" -or
    $_.TaskPath -like "*FileServices*"
  } |
  Tee-Object "$EvidencePath\file-services-scheduled-tasks-before-backup-export.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_Windows_Server_Backup_Install_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: install and validate Windows Server Backup tooling.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\12-windows-server-backup-install-transcript.txt"

# Install Windows Server Backup.
Install-WindowsFeature `
  -Name Windows-Server-Backup `
  -IncludeManagementTools

# Confirm feature state.
Get-WindowsFeature Windows-Server-Backup |
  Tee-Object "$EvidencePath\windows-server-backup-feature-after-install.txt"

# Confirm command-line tooling.
Get-Command wbadmin |
  Tee-Object "$EvidencePath\wbadmin-command-after-install.txt"

# Confirm PowerShell module.
Get-Module WindowsServerBackup -ListAvailable |
  Tee-Object "$EvidencePath\windows-server-backup-module-after-install.txt"

Get-Command -Module WindowsServerBackup |
  Tee-Object "$EvidencePath\windows-server-backup-cmdlets-after-install.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_Data_Backup_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: run an immediate data volume backup using wbadmin.
# Replace the backup target with a real network share or backup disk path.

$EvidencePath = "C:\FileServices-Validation"

$DataVolume = "E:"
$BackupTarget = "\\BACKUP1\FileServerBackups\FS1"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\12-data-backup-transcript.txt"

# Confirm source volume.
Get-Volume -DriveLetter "E" |
  Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining |
  Tee-Object "$EvidencePath\data-volume-before-wbadmin-backup.txt"

# Confirm backup target reachability.
Test-Path $BackupTarget |
  Tee-Object "$EvidencePath\backup-target-testpath-before-wbadmin.txt"

# Capture backup versions before running backup.
wbadmin get versions -backupTarget:$BackupTarget |
  Tee-Object "$EvidencePath\wbadmin-versions-before-backup.txt"

# Run backup.
# For a lab, this backs up the data volume only.
# Add -allCritical for bare-metal/system recovery planning when appropriate.
wbadmin start backup `
  -backupTarget:$BackupTarget `
  -include:$DataVolume `
  -quiet |
  Tee-Object "$EvidencePath\wbadmin-start-backup-result.txt"

# Capture backup versions after running backup.
wbadmin get versions -backupTarget:$BackupTarget |
  Tee-Object "$EvidencePath\wbadmin-versions-after-backup.txt"

# Capture recent backup events.
Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\windows-backup-events-after-backup.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_File_Restore_Test_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: restore data from backup to an alternate path for validation.
# Replace <backup-version> with a version from wbadmin get versions.

$EvidencePath = "C:\FileServices-Validation"

$BackupTarget = "\\BACKUP1\FileServerBackups\FS1"
$SourceItem = "E:\Shares\Departments\Accounting"
$RestoreTarget = "E:\Restore-Test"
$BackupVersion = "<backup-version>"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $RestoreTarget

Start-Transcript -Path "$EvidencePath\12-file-restore-test-transcript.txt"

# List versions.
wbadmin get versions -backupTarget:$BackupTarget |
  Tee-Object "$EvidencePath\wbadmin-versions-before-restore.txt"

# Restore to alternate path.
# Do not restore over production data during the first validation test.
wbadmin start recovery `
  -version:$BackupVersion `
  -itemType:File `
  -items:$SourceItem `
  -recoveryTarget:$RestoreTarget `
  -recursive `
  -quiet |
  Tee-Object "$EvidencePath\wbadmin-file-restore-result.txt"

# Validate restored content.
Get-ChildItem $RestoreTarget -Recurse -Force |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\restore-test-file-inventory.txt"

# Capture ACLs on restored content.
icacls $RestoreTarget /t /c |
  Tee-Object "$EvidencePath\restore-test-icacls.txt"

# Backup event evidence.
Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\windows-backup-events-after-restore.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_SMB_Share_Config_Export_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: export SMB share definitions and share permissions for rebuild documentation.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"
$SmbExportPath = Join-Path $ExportRoot "SMB"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $SmbExportPath

Start-Transcript -Path "$EvidencePath\12-smb-share-config-export-transcript.txt"

# Export non-special SMB shares.
$Shares = Get-SmbShare |
  Where-Object { -not $_.Special } |
  Sort-Object Name

$Shares |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,ConcurrentUserLimit,Special |
  Export-Csv "$SmbExportPath\smb-shares.csv" -NoTypeInformation

$Shares |
  Export-Clixml "$SmbExportPath\smb-shares.clixml"

$Shares |
  ConvertTo-Json -Depth 5 |
  Out-File "$SmbExportPath\smb-shares.json" -Encoding UTF8

# Export share permissions.
$ShareAccess = foreach ($Share in $Shares) {
  Get-SmbShareAccess -Name $Share.Name |
    Select-Object @{Name="ShareName";Expression={$Share.Name}},AccountName,AccessControlType,AccessRight
}

$ShareAccess |
  Export-Csv "$SmbExportPath\smb-share-access.csv" -NoTypeInformation

$ShareAccess |
  Export-Clixml "$SmbExportPath\smb-share-access.clixml"

# Export SMB server/client posture.
Get-SmbServerConfiguration |
  Export-Clixml "$SmbExportPath\smb-server-configuration.clixml"

Get-SmbServerConfiguration |
  Out-File "$SmbExportPath\smb-server-configuration.txt"

Get-SmbClientConfiguration |
  Export-Clixml "$SmbExportPath\smb-client-configuration.clixml"

Get-SmbClientConfiguration |
  Out-File "$SmbExportPath\smb-client-configuration.txt"

# Generate a starter rebuild script.
$RebuildScript = @'
# Rebuild-SMB-Shares.ps1
# Review before running on a rebuilt file server.

$ImportRoot = "C:\FileServices-Exports\SMB"
$Shares = Import-Clixml "$ImportRoot\smb-shares.clixml"
$ShareAccess = Import-Csv "$ImportRoot\smb-share-access.csv"

foreach ($Share in $Shares) {
  if (-not (Test-Path $Share.Path)) {
    New-Item -ItemType Directory -Force -Path $Share.Path | Out-Null
  }

  if (-not (Get-SmbShare -Name $Share.Name -ErrorAction SilentlyContinue)) {
    New-SmbShare `
      -Name $Share.Name `
      -Path $Share.Path `
      -Description $Share.Description `
      -CachingMode $Share.CachingMode `
      -FolderEnumerationMode $Share.FolderEnumerationMode `
      -EncryptData ([bool]$Share.EncryptData) `
      -FullAccess "BUILTIN\Administrators"
  }

  $AccessRows = $ShareAccess | Where-Object { $_.ShareName -eq $Share.Name }

  foreach ($Ace in $AccessRows) {
    Grant-SmbShareAccess `
      -Name $Share.Name `
      -AccountName $Ace.AccountName `
      -AccessRight $Ace.AccessRight `
      -Force
  }
}
'@

$RebuildScript |
  Out-File "$SmbExportPath\Rebuild-SMB-Shares.ps1" -Encoding UTF8 -Force

# Validate export files.
Get-ChildItem $SmbExportPath |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\smb-export-files.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_NTFS_ACL_Backup_And_Restore_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: save NTFS ACLs for restore and rebuild.
# Important: icacls restore must be run from the correct parent path.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"
$AclBackupPath = Join-Path $ExportRoot "ACL"

$DataRoot = "E:\Shares"
$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $AclBackupPath

Start-Transcript -Path "$EvidencePath\12-ntfs-acl-backup-restore-transcript.txt"

# Save ACLs.
icacls $DataRoot /save "$AclBackupPath\data-root-acl.txt" /t /c
icacls $DepartmentRoot /save "$AclBackupPath\department-root-acl.txt" /t /c
icacls $HomeRoot /save "$AclBackupPath\home-root-acl.txt" /t /c
icacls $PublicRoot /save "$AclBackupPath\public-root-acl.txt" /t /c
icacls $AppsRoot /save "$AclBackupPath\apps-root-acl.txt" /t /c

# Export readable ACL evidence.
$AclPaths = @(
  $DataRoot,
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $AppsRoot
)

foreach ($Path in $AclPaths) {
  Get-Acl $Path |
    Format-List |
    Out-File "$AclBackupPath\get-acl-$($Path.Replace(':','').Replace('\','-')).txt"

  icacls $Path |
    Out-File "$AclBackupPath\icacls-$($Path.Replace(':','').Replace('\','-')).txt"
}

# Create ACL restore notes.
$RestoreNotes = @'
# NTFS ACL Restore Notes

Run restore commands from the correct parent path.

Examples:
icacls E:\ /restore C:\FileServices-Exports\ACL\data-root-acl.txt
icacls E:\Shares /restore C:\FileServices-Exports\ACL\department-root-acl.txt
icacls E:\Shares /restore C:\FileServices-Exports\ACL\home-root-acl.txt

Validate after restore:
icacls E:\Shares
icacls E:\Shares\Departments
icacls E:\Shares\Home
'@

$RestoreNotes |
  Out-File "$AclBackupPath\NTFS-ACL-Restore-Notes.txt" -Encoding UTF8

# Validate export files.
Get-ChildItem $AclBackupPath |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\acl-backup-files.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_FSRM_Config_Export_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: export FSRM governance configuration for rebuild documentation.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"
$FsrmExportPath = Join-Path $ExportRoot "FSRM"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $FsrmExportPath

Start-Transcript -Path "$EvidencePath\12-fsrm-config-export-transcript.txt"

Import-Module FileServerResourceManager -ErrorAction SilentlyContinue

# Feature and service state.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$FsrmExportPath\fsrm-feature.txt"

Get-Service SrmSvc -ErrorAction SilentlyContinue |
  Tee-Object "$FsrmExportPath\fsrm-service.txt"

# Export FSRM objects as CLIXML and readable text.
Get-FsrmQuotaTemplate -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-quota-templates.clixml"

Get-FsrmQuotaTemplate -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-quota-templates.txt"

Get-FsrmQuota -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-quotas.clixml"

Get-FsrmQuota -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-quotas.txt"

Get-FsrmAutoQuota -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-auto-quotas.clixml"

Get-FsrmAutoQuota -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-auto-quotas.txt"

Get-FsrmFileGroup -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-file-groups.clixml"

Get-FsrmFileGroup -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-file-groups.txt"

Get-FsrmFileScreenTemplate -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-file-screen-templates.clixml"

Get-FsrmFileScreenTemplate -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-file-screen-templates.txt"

Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-file-screens.clixml"

Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-file-screens.txt"

Get-FsrmFileScreenException -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-file-screen-exceptions.clixml"

Get-FsrmFileScreenException -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-file-screen-exceptions.txt"

Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-storage-reports.clixml"

Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-storage-reports.txt"

Get-FsrmFileManagementJob -ErrorAction SilentlyContinue |
  Export-Clixml "$FsrmExportPath\fsrm-file-management-jobs.clixml"

Get-FsrmFileManagementJob -ErrorAction SilentlyContinue |
  Out-File "$FsrmExportPath\fsrm-file-management-jobs.txt"

# Export related FSRM report output folders if present.
Get-ChildItem "C:\StorageReports" -Recurse -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Export-Csv "$FsrmExportPath\storage-report-output-inventory.csv" -NoTypeInformation

# Validate export files.
Get-ChildItem $FsrmExportPath |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\fsrm-export-files.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_Disaster_Rebuild_Export_Package_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: build a single rebuild package containing data, ACL, SMB, FSRM, VSS, task, and event evidence.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"
$PackageRoot = Join-Path $ExportRoot "RebuildPackage"

$PackageName = "FS1-FileServices-RebuildPackage-$(Get-Date -Format yyyyMMdd-HHmmss)"
$PackagePath = Join-Path $PackageRoot $PackageName

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $PackagePath

Start-Transcript -Path "$EvidencePath\12-disaster-rebuild-export-package-transcript.txt"

# Create package folders.
$Subfolders = @(
  "System",
  "Volumes",
  "SMB",
  "ACL",
  "FSRM",
  "VSS",
  "ScheduledTasks",
  "Events",
  "Scripts",
  "RestoreEvidence"
)

foreach ($Folder in $Subfolders) {
  New-Item -ItemType Directory -Force -Path (Join-Path $PackagePath $Folder) | Out-Null
}

# System and feature state.
Get-ComputerInfo |
  Out-File "$PackagePath\System\computer-info.txt"

Get-WindowsFeature |
  Where-Object Installed |
  Out-File "$PackagePath\System\installed-windows-features.txt"

ipconfig /all |
  Out-File "$PackagePath\System\ipconfig-all.txt"

# Volume state.
Get-Disk |
  Out-File "$PackagePath\Volumes\disks.txt"

Get-Partition |
  Out-File "$PackagePath\Volumes\partitions.txt"

Get-Volume |
  Out-File "$PackagePath\Volumes\volumes.txt"

Get-Volume |
  Export-Csv "$PackagePath\Volumes\volumes.csv" -NoTypeInformation

# SMB export.
if (Test-Path "$ExportRoot\SMB") {
  Copy-Item "$ExportRoot\SMB\*" "$PackagePath\SMB" -Recurse -Force
}

# ACL export.
if (Test-Path "$ExportRoot\ACL") {
  Copy-Item "$ExportRoot\ACL\*" "$PackagePath\ACL" -Recurse -Force
}

# FSRM export.
if (Test-Path "$ExportRoot\FSRM") {
  Copy-Item "$ExportRoot\FSRM\*" "$PackagePath\FSRM" -Recurse -Force
}

# VSS state.
vssadmin list writers |
  Out-File "$PackagePath\VSS\vss-writers.txt"

vssadmin list shadowstorage |
  Out-File "$PackagePath\VSS\vss-shadowstorage.txt"

vssadmin list shadows |
  Out-File "$PackagePath\VSS\vss-shadows.txt"

# Scheduled tasks related to file services.
Get-ScheduledTask |
  Where-Object {
    $_.TaskName -like "*Shadow*" -or
    $_.TaskName -like "*Backup*" -or
    $_.TaskName -like "*FSRM*" -or
    $_.TaskPath -like "*FileServices*"
  } |
  Export-Clixml "$PackagePath\ScheduledTasks\file-services-scheduled-tasks.clixml"

Get-ScheduledTask |
  Where-Object {
    $_.TaskName -like "*Shadow*" -or
    $_.TaskName -like "*Backup*" -or
    $_.TaskName -like "*FSRM*" -or
    $_.TaskPath -like "*FileServices*"
  } |
  Out-File "$PackagePath\ScheduledTasks\file-services-scheduled-tasks.txt"

# Events.
Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$PackagePath\Events\windows-backup-events.csv" -NoTypeInformation

Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="VSS"; StartTime=(Get-Date).AddDays(-14)} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$PackagePath\Events\vss-events.csv" -NoTypeInformation

Get-WinEvent -FilterHashtable @{LogName="System"; ProviderName="VolSnap"; StartTime=(Get-Date).AddDays(-14)} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$PackagePath\Events\volsnap-events.csv" -NoTypeInformation

Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"; StartTime=(Get-Date).AddDays(-14)} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$PackagePath\Events\fsrm-events.csv" -NoTypeInformation

# Rebuild order notes.
$RebuildOrder = @'
# File Server Rebuild Order

1. Install Windows Server and join domain.
2. Install File Server role and management tools.
3. Attach or restore data volume.
4. Recreate folder roots if data volume is empty.
5. Restore NTFS ACLs using ACL export.
6. Recreate SMB shares using SMB export and rebuild script.
7. Reapply SMB security settings, ABE, encryption, and signing posture.
8. Install FSRM.
9. Recreate or reconfigure FSRM quotas, file screens, reports, and file management jobs.
10. Recreate Shadow Copy schedule.
11. Configure backup schedule.
12. Restore test data to alternate path.
13. Validate user access from client.
14. Document final state.
'@

$RebuildOrder |
  Out-File "$PackagePath\Rebuild-Order.md" -Encoding UTF8

# Compress package.
$ZipPath = "$PackagePath.zip"

Compress-Archive `
  -Path $PackagePath `
  -DestinationPath $ZipPath `
  -Force

# Validate package.
Get-ChildItem $PackageRoot |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\rebuild-package-files.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_Post_Change_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture final backup, restore, export, and rebuild package evidence.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"
$BackupTarget = "\\BACKUP1\FileServerBackups\FS1"
$RestoreTestPath = "E:\Restore-Test"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\12-post-change-backup-restore-export-validation-transcript.txt"

# Feature and tool state.
Get-WindowsFeature FS-FileServer,FS-Resource-Manager,Windows-Server-Backup |
  Tee-Object "$EvidencePath\features-final-backup-restore-export.txt"

Get-Command wbadmin |
  Tee-Object "$EvidencePath\wbadmin-final.txt"

Get-Command -Module WindowsServerBackup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\windows-server-backup-cmdlets-final.txt"

# Backup versions.
wbadmin get versions -backupTarget:$BackupTarget |
  Tee-Object "$EvidencePath\wbadmin-versions-final.txt"

# Restore test evidence.
if (Test-Path $RestoreTestPath) {
  Get-ChildItem $RestoreTestPath -Recurse -Force |
    Select-Object FullName,Length,CreationTime,LastWriteTime |
    Tee-Object "$EvidencePath\restore-test-inventory-final.txt"

  icacls $RestoreTestPath /t /c |
    Tee-Object "$EvidencePath\restore-test-icacls-final.txt"
}

# SMB export evidence.
if (Test-Path "$ExportRoot\SMB") {
  Get-ChildItem "$ExportRoot\SMB" |
    Select-Object FullName,Length,CreationTime,LastWriteTime |
    Tee-Object "$EvidencePath\smb-export-final.txt"
}

# ACL export evidence.
if (Test-Path "$ExportRoot\ACL") {
  Get-ChildItem "$ExportRoot\ACL" |
    Select-Object FullName,Length,CreationTime,LastWriteTime |
    Tee-Object "$EvidencePath\acl-export-final.txt"
}

# FSRM export evidence.
if (Test-Path "$ExportRoot\FSRM") {
  Get-ChildItem "$ExportRoot\FSRM" |
    Select-Object FullName,Length,CreationTime,LastWriteTime |
    Tee-Object "$EvidencePath\fsrm-export-final.txt"
}

# Rebuild package evidence.
if (Test-Path "$ExportRoot\RebuildPackage") {
  Get-ChildItem "$ExportRoot\RebuildPackage" -Recurse |
    Select-Object FullName,Length,CreationTime,LastWriteTime |
    Tee-Object "$EvidencePath\rebuild-package-final.txt"
}

# Live configuration evidence.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-final-backup-restore-export.txt"

Get-SmbShare |
  ForEach-Object {
    Get-SmbShareAccess -Name $_.Name -ErrorAction SilentlyContinue
  } |
  Tee-Object "$EvidencePath\smb-share-access-final-backup-restore-export.txt"

Get-FsrmQuota -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quotas-final-backup-restore-export.txt"

Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screens-final-backup-restore-export.txt"

# Event evidence.
Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\windows-backup-events-final.txt"

Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="VSS"; StartTime=(Get-Date).AddDays(-7)} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\vss-events-final-backup-restore-export.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_File_Server_Config_Verification_Commands
```powershell
# File Server baseline
Get-WindowsFeature FS-FileServer
Get-WindowsFeature FS-Resource-Manager
Get-WindowsFeature Windows-Server-Backup
Get-Volume
Get-Volume -DriveLetter <data-drive-letter>

# Backup tooling
Get-Command wbadmin
Get-Module WindowsServerBackup -ListAvailable
Get-Command -Module WindowsServerBackup

# Backup target
Test-Path "<backup-target-path>"
wbadmin get versions -backupTarget:<backup-target-path>

# Start immediate backup
wbadmin start backup -backupTarget:<backup-target-path> -include:<data-volume> -quiet

# Restore test
wbadmin get versions -backupTarget:<backup-target-path>
wbadmin start recovery -version:<backup-version> -itemType:File -items:<source-path> -recoveryTarget:<restore-test-path> -recursive -quiet

# Restore validation
Test-Path "<restore-test-path>"
Get-ChildItem "<restore-test-path>" -Recurse
icacls "<restore-test-path>" /t /c

# SMB export validation
Get-SmbShare
Get-SmbShare | Where-Object { -not $_.Special }
Get-SmbShareAccess -Name "<share-name>"
Import-Clixml "<smb-export-path>\smb-shares.clixml"
Import-Csv "<smb-export-path>\smb-share-access.csv"

# NTFS ACL backup and restore
icacls "<data-root>" /save "<acl-backup-path>\data-root-acl.txt" /t /c
icacls "<restore-parent-path>" /restore "<acl-backup-path>\data-root-acl.txt"

# FSRM export validation
Get-FsrmQuota
Get-FsrmAutoQuota
Get-FsrmQuotaTemplate
Get-FsrmFileGroup
Get-FsrmFileScreenTemplate
Get-FsrmFileScreen
Get-FsrmFileScreenException
Get-FsrmStorageReport
Get-FsrmFileManagementJob

# Shadow copy and VSS context
vssadmin list writers
vssadmin list shadowstorage
vssadmin list shadows /for=<data-volume>

# Scheduled tasks
Get-ScheduledTask | Where-Object TaskName -like "*Shadow*"
Get-ScheduledTask | Where-Object TaskName -like "*Backup*"
Get-ScheduledTask | Where-Object TaskPath -like "*FileServices*"

# Events
Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 100
Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="VSS"; StartTime=(Get-Date).AddDays(-7)}
Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"; StartTime=(Get-Date).AddDays(-7)}
Get-WinEvent -FilterHashtable @{LogName="System"; ProviderName="VolSnap"; StartTime=(Get-Date).AddDays(-7)}

# Export folders
Test-Path "<export-root>"
Test-Path "<smb-export-path>"
Test-Path "<acl-backup-path>"
Test-Path "<fsrm-export-path>"
Get-ChildItem "<export-root>" -Recurse

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*backup*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*restore*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*export*"
```

# Backup_Restore_And_Export_File_Server_Config_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current backup/export state | File Server | `Get-ChildItem "<export-root>" -Recurse`; `wbadmin get versions -backupTarget:<target>` | Current state is documented |
| 2 | Remove restore test files | File Server | `Remove-Item "<restore-test-path>" -Recurse -Force` | Restore test folder is removed |
| 3 | Remove temporary export package if needed | File Server | `Remove-Item "<rebuild-package-path>" -Recurse -Force` | Lab export package is removed |
| 4 | Remove compressed rebuild package if needed | File Server | `Remove-Item "<rebuild-package-path>.zip" -Force` | Lab zip package is removed |
| 5 | Preserve ACL backup before deleting | File Server | `Copy-Item "<acl-backup-path>" "<evidence-path>\ACL-Backup-Preserved" -Recurse -Force` | ACL backup is preserved |
| 6 | Preserve SMB export before deleting | File Server | `Copy-Item "<smb-export-path>" "<evidence-path>\SMB-Export-Preserved" -Recurse -Force` | SMB export is preserved |
| 7 | Preserve FSRM export before deleting | File Server | `Copy-Item "<fsrm-export-path>" "<evidence-path>\FSRM-Export-Preserved" -Recurse -Force` | FSRM export is preserved |
| 8 | Remove lab export folder if required | File Server | `Remove-Item "<export-root>" -Recurse -Force` | Export folder is removed |
| 9 | Remove scheduled backup task if one was created | File Server | `Unregister-ScheduledTask -TaskName "<backup-task-name>" -Confirm:$false` | Lab backup task is removed |
| 10 | Do not delete production backups casually | Backup Target | Backup console or retention policy | Production backup retention remains governed |
| 11 | Restore ACLs if rollback requires it | File Server | `icacls "<restore-parent-path>" /restore "<acl-backup-file>"` | Prior ACLs are restored |
| 12 | Restore SMB share config if rollback requires it | File Server | Run reviewed `Rebuild-SMB-Shares.ps1` | Shares are recreated |
| 13 | Validate after rollback | File Server | `Get-SmbShare`; `icacls`; `Get-FsrmQuota`; `wbadmin get versions` | State matches rollback target |
| 14 | Document rollback | Operator | `Record removed test artifacts, preserved exports, and validation output` | Rollback record is complete |

# Backup_Restore_And_Export_File_Server_Config_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| `wbadmin` not recognized | Windows Server Backup not installed | `Get-WindowsFeature Windows-Server-Backup` | Install `Windows-Server-Backup` |
| Backup target inaccessible | DNS, permissions, firewall, or path issue | `Test-Path "<backup-target-path>"`; `Resolve-DnsName` | Fix path, DNS, credentials, or share permissions |
| Backup fails with access denied | Backup account lacks write access to target | Target share and NTFS ACLs | Grant backup account required access |
| Backup fails due to insufficient space | Backup target full | Target free space check | Free space, expand target, or adjust retention |
| Backup succeeds but no version appears | Wrong backup target queried | `wbadmin get versions -backupTarget:<target>` | Query correct target path |
| Restore fails | Wrong backup version format | `wbadmin get versions` | Copy exact backup version value |
| Restore overwrites production accidentally | Recovery target omitted or wrong | Command history and restored path | Always restore to alternate path first |
| Restored files missing ACLs | File-level restore or target path behavior changed permissions | `icacls "<restore-test-path>"` | Restore ACLs from `icacls /save` backup if needed |
| SMB export missing shares | Only special shares exist or filter too strict | `Get-SmbShare` | Verify share list and filter |
| SMB rebuild script fails | Paths do not exist on rebuilt server | `Test-Path <share-path>` | Restore data/folders before recreating shares |
| SMB permissions not restored correctly | Account names differ or groups missing | `Get-SmbShareAccess`; `Get-ADGroup` | Recreate groups or correct account names |
| ACL restore fails | `icacls /restore` run from wrong parent path | Saved ACL file path format | Run restore from correct parent path |
| ACL restore creates access problems | ACL backup was old or wrong | Compare backup timestamp and current path | Restore correct ACL file or recover admin ownership |
| FSRM export empty | FSRM not installed or cmdlets unavailable | `Get-WindowsFeature FS-Resource-Manager` | Install/import FSRM or accept empty export |
| FSRM rebuild incomplete | Export is evidence, not full automatic import | Export files and rebuild notes | Recreate objects from documented settings |
| Shadow copies not included in backup | Shadow copies are local volume state, not normal portable config | VSS/shadowcopy evidence | Use backup for data recovery, not local snapshot migration |
| Event logs missing | Wrong log or time window | `Get-WinEvent` with larger window | Query correct logs and expand time range |
| Rebuild package too large | Report output or restore data included | `Get-ChildItem <package> -Recurse | Sort Length` | Exclude bulky report/restore folders or compress separately |
| Evidence missing | Skeleton not run or wrong path | `Test-Path "<evidence-path>"` | Re-run precheck or post-change validation skeleton |

# Backup_Restore_And_Export_File_Server_Config_Related_Labs
| Related Lab                                                      | Relationship                                                                      |
| ---------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| `00_File_Services_Index.md`                                      | Defines where backup, restore, and export fits in the File Services suite         |
| `01_Install_File_Server_Role_And_Management_Tools.md`            | Provides File Server role and management baseline                                 |
| `02_Prepare_File_Server_Storage_Volumes_And_Folders.md`          | Defines volumes and folders that must be protected                                |
| `03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`           | Provides ACL model that must be exported and restorable                           |
| `04_Create_SMB_Shares_And_Share_Permissions.md`                  | Provides SMB shares that must be exported and rebuildable                         |
| `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`  | Adds share visibility settings that must be captured                              |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`   | Adds SMB hardening and audit settings that must be documented                     |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`        | Provides high-value user data that must be backed up and restore-tested           |
| `08_Configure_File_Screening_With_FSRM.md`                       | Adds FSRM screening state that must be exported                                   |
| `09_Configure_Storage_Quotas_With_FSRM.md`                       | Adds FSRM quota state that must be exported                                       |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | Adds FSRM report and file management jobs that must be exported                   |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md`            | Provides local restore convenience but does not replace backup                    |
| `13_Configure_DFS_Namespaces.md`                                 | Adds namespace configuration that should later be included in export and recovery |
| `14_Configure_DFS_Replication.md`                                | Adds replicated data state that affects backup and restore strategy               |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`   | Provides operational evidence during backup and restore windows                   |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`          | Helps validate restored access after recovery                                     |
| `25_Migrate_File_Server_Data_Shares_And_Permissions.md`          | Uses these export and restore methods during migration                            |