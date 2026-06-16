25_Migrate_File_Server_Data_Shares_And_Permissions.md
# Migrate_File_Server_Data_Shares_And_Permissions

# Migrate_File_Server_Data_Shares_And_Permissions_Index
25_Migrate_File_Server_Data_Shares_And_Permissions.md
Migrate_File_Server_Data_Shares_And_Permissions
Migrate_File_Server_Data_Shares_And_Permissions_Source_Basis
Migrate_File_Server_Data_Shares_And_Permissions_Mental_Model
Migrate_File_Server_Data_Shares_And_Permissions_Planning_Table
Migrate_File_Server_Data_Shares_And_Permissions_Configuration_Checklist
Migrate_File_Server_Data_Shares_And_Permissions_Source_Inventory_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Share_Export_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_NTFS_ACL_Export_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Target_Folder_Preparation_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Initial_Robocopy_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Delta_Robocopy_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Recreate_SMB_Shares_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Reapply_Share_Permissions_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Cutover_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_DFS_Or_DNS_Update_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Post_Migration_Validation_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Status_And_Event_Review_Skeleton
Migrate_File_Server_Data_Shares_And_Permissions_Verification_Commands
Migrate_File_Server_Data_Shares_And_Permissions_Rollback
Migrate_File_Server_Data_Shares_And_Permissions_Failure_Checks
Migrate_File_Server_Data_Shares_And_Permissions_Related_Labs

# Migrate_File_Server_Data_Shares_And_Permissions_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Storage Migration Service | Inventory, transfer, cutover | GUI-driven migration of files, shares, and security configuration |
| Robocopy | `/MIR`, `/COPYALL`, `/DCOPY`, `/SEC`, `/SECFIX`, `/TIMFIX`, `/MT`, `/LOG` | Manual data migration with NTFS ACLs, ownership, auditing, timestamps, and logs |
| SMBShare PowerShell module | Get-SmbShare | Source share inventory |
| SMBShare PowerShell module | Get-SmbShareAccess | Source share permission export |
| SMBShare PowerShell module | New-SmbShare | Target share recreation |
| SMBShare PowerShell module | Grant-SmbShareAccess | Target share permission recreation |
| NTFS ACL tooling | icacls save, restore, grant, verify | Exporting, comparing, and restoring NTFS ACLs |
| DFS Namespace tooling | Get-DfsnFolderTarget, Set-DfsnFolderTarget | Cutover through namespace target update |
| DNS tooling | Add-DnsServerResourceRecordCName, Set-DnsServerResourceRecord | Cutover through alias update |
| Operational file server practice | Pre-copy, delta copy, freeze, final sync, validation, rollback | Low-risk production migration workflow |

# Migrate_File_Server_Data_Shares_And_Permissions_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Source server | Current file server users access today |
| Destination server | New file server receiving data, shares, and permissions |
| Pre-copy | Large first data copy while users continue working |
| Delta copy | Later copy that only catches changes since the pre-copy |
| Freeze window | Short period where users are locked out or told to stop writes |
| Final sync | Last Robocopy pass during the freeze window |
| Cutover | Change user access path from old server to new server |
| Share permission | SMB-level permission on the share itself |
| NTFS permission | File system permission on folders and files |
| Effective access | Combined result of share permission and NTFS permission |
| Ownership | File or folder owner copied by Robocopy with owner flags |
| Auditing SACL | Audit entries copied by Robocopy when using audit flags and proper privileges |
| Local groups problem | Local groups on the old server do not automatically mean the same thing on the new server |
| SID history issue | Orphaned SIDs or unmigrated local accounts can break access on the destination |
| DFS cutover | Easiest user experience if users already access `\\domain\namespace\share` |
| DNS alias cutover | Works if clients use a stable alias instead of the physical server name |
| Server identity cutover | Storage Migration Service can take over the old server identity in supported scenarios |
| Blunt rule | Migrating files without shares, share permissions, NTFS ACLs, open-file handling, and cutover testing is not a real file server migration |

# Migrate_File_Server_Data_Shares_And_Permissions_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Source server | `FSOLD01` | `<source-server-name>` |
| Destination server | `FSNEW01` | `<destination-server-name>` |
| Domain | `corp.local` | `<domain-fqdn>` |
| Migration method | Robocopy / Storage Migration Service | `<migration-method>` |
| Source share root | `D:\Shares` | `<source-share-root>` |
| Destination share root | `D:\Shares` | `<destination-share-root>` |
| Source admin path | `\\FSOLD01\D$\Shares` | `<source-admin-path>` |
| Destination local path | `D:\Shares` | `<destination-local-path>` |
| Target share | `Departments` | `<share-name>` |
| Target source path | `D:\Shares\Departments` | `<source-share-path>` |
| Target destination path | `D:\Shares\Departments` | `<destination-share-path>` |
| User access path | `\\FSOLD01\Departments` | `<current-user-path>` |
| New access path | `\\FSNEW01\Departments` | `<new-user-path>` |
| DFS path | `\\corp.local\Shares\Departments` | `<dfs-path>` |
| DNS alias | `files.corp.local` | `<dns-alias>` |
| Cutover model | DFS / DNS alias / server rename / user path update | `<cutover-model>` |
| Freeze window | Friday 18:00 to 20:00 | `<freeze-window>` |
| Rollback deadline | Friday 19:30 | `<rollback-deadline>` |
| Migration account | `CORP\svc-filemig` | `<migration-account>` |
| Required privilege | Local admin on source and destination | `<required-privilege>` |
| Log path | `C:\FileMigration\Logs` | `<log-path>` |
| Export path | `C:\FileMigration\Exports` | `<export-path>` |
| Report path | `C:\FileMigration\Reports` | `<report-path>` |
| Robocopy threads | `16` | `<robocopy-mt>` |
| Retry count | `2` | `<robocopy-retry>` |
| Wait seconds | `5` | `<robocopy-wait>` |
| Backup confirmed | Yes / No | `<backup-confirmed>` |
| Share inventory captured | Yes / No | `<share-inventory-captured>` |
| NTFS ACL export captured | Yes / No | `<ntfs-acl-export-captured>` |
| Open files cleared | Yes / No | `<open-files-cleared>` |
| Test user validated | Yes / No | `<test-user-validated>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Migrate_File_Server_Data_Shares_And_Permissions_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm source identity | Source Server | `hostname` | Correct source server is identified |
| 2 | Confirm destination identity | Destination Server | `hostname` | Correct destination server is identified |
| 3 | Confirm domain membership | Both Servers | `(Get-CimInstance Win32_ComputerSystem).Domain` | Both servers are in expected domain |
| 4 | Confirm OS version | Both Servers | `Get-ComputerInfo | Select WindowsProductName,WindowsVersion,OsName` | OS versions are documented |
| 5 | Confirm source IP configuration | Source Server | `Get-NetIPConfiguration` | Source network state is visible |
| 6 | Confirm destination IP configuration | Destination Server | `Get-NetIPConfiguration` | Destination network state is visible |
| 7 | Confirm source volume state | Source Server | `Get-Volume` | Source data volume is healthy |
| 8 | Confirm destination volume state | Destination Server | `Get-Volume` | Destination data volume is healthy |
| 9 | Confirm destination free space | Destination Server | `Get-PSDrive -PSProvider FileSystem` | Enough space exists |
| 10 | Confirm File Server role on destination | Destination Server | `Install-WindowsFeature FS-FileServer -IncludeManagementTools` | File Server role is installed |
| 11 | Confirm backup before migration | Source Server | `wbadmin get versions` | Backup state is known if Windows Server Backup is used |
| 12 | Confirm source shares | Source Server | `Get-SmbShare | Where-Object Special -eq $false` | Non-admin shares are visible |
| 13 | Export source share inventory | Source Server | `Get-SmbShare | Where-Object Special -eq $false | Export-Clixml "C:\FileMigration\Exports\source-shares.xml"` | Share inventory is saved |
| 14 | Export source share permissions | Source Server | `Get-SmbShare | Where-Object Special -eq $false | ForEach-Object { Get-SmbShareAccess -Name $_.Name } | Export-Clixml "C:\FileMigration\Exports\source-share-access.xml"` | Share permissions are saved |
| 15 | Export source NTFS ACLs | Source Server | `icacls "D:\Shares" /save "C:\FileMigration\Exports\source-ntfs-acls.txt" /t /c` | NTFS ACL export is saved |
| 16 | Export source local groups | Source Server | `Get-LocalGroup | Export-Clixml "C:\FileMigration\Exports\source-local-groups.xml"` | Local group inventory is saved |
| 17 | Export source local group members | Source Server | `Get-LocalGroup | ForEach-Object { Get-LocalGroupMember $_.Name } | Export-Clixml "C:\FileMigration\Exports\source-local-group-members.xml"` | Local group membership is saved |
| 18 | Export source SMB settings | Source Server | `Get-SmbServerConfiguration | Export-Clixml "C:\FileMigration\Exports\source-smb-server-config.xml"` | SMB server settings are saved |
| 19 | Export source FSRM settings if present | Source Server | `dirquota template list` | FSRM quota state is documented if FSRM is used |
| 20 | Export source open files baseline | Source Server | `Get-SmbOpenFile | Export-Csv "C:\FileMigration\Reports\source-open-files-before.csv" -NoTypeInformation` | Open files are documented |
| 21 | Create destination folder root | Destination Server | `New-Item -ItemType Directory -Path "D:\Shares" -Force` | Destination root exists |
| 22 | Pre-create destination subfolder | Destination Server | `New-Item -ItemType Directory -Path "D:\Shares\Departments" -Force` | Destination target folder exists |
| 23 | Confirm admin share access from destination to source | Destination Server | `Test-Path "\\FSOLD01\D$\Shares"` | Destination can read source admin path |
| 24 | Confirm admin rights on destination | Destination Server | `Test-Path "D:\Shares"` | Migration account can access destination |
| 25 | Run dry-run Robocopy list | Destination Server | `robocopy "\\FSOLD01\D$\Shares\Departments" "D:\Shares\Departments" /MIR /COPYALL /DCOPY:DAT /L /R:0 /W:0 /LOG:C:\FileMigration\Logs\dryrun-Departments.log` | No-copy preview log is generated |
| 26 | Run initial Robocopy pre-copy | Destination Server | `robocopy "\\FSOLD01\D$\Shares\Departments" "D:\Shares\Departments" /MIR /COPYALL /DCOPY:DAT /ZB /R:2 /W:5 /MT:16 /XJ /TEE /LOG:C:\FileMigration\Logs\initial-Departments.log` | Data, ACLs, owner, audit, and timestamps copy |
| 27 | Capture initial Robocopy exit code | Destination Server | `$LASTEXITCODE` | Exit code under 8 means no Robocopy failure |
| 28 | Run security fix pass | Destination Server | `robocopy "\\FSOLD01\D$\Shares\Departments" "D:\Shares\Departments" /E /COPYALL /DCOPY:DAT /SECFIX /TIMFIX /R:2 /W:5 /MT:16 /XJ /LOG+:C:\FileMigration\Logs\secfix-Departments.log` | Security and timestamps are corrected |
| 29 | Compare source and destination file counts | Management Host | `Compare source and destination Get-ChildItem counts` | Counts are reasonably aligned |
| 30 | Recreate SMB share on destination | Destination Server | `New-SmbShare -Name "Departments" -Path "D:\Shares\Departments" -CachingMode None -FolderEnumerationMode AccessBased -Description "Migrated Departments share"` | Destination share exists |
| 31 | Apply share permissions | Destination Server | `Grant-SmbShareAccess -Name "Departments" -AccountName "CORP\GG_FS_Departments_Modify" -AccessRight Change -Force` | Share permissions are applied |
| 32 | Confirm destination share permissions | Destination Server | `Get-SmbShareAccess -Name "Departments"` | Expected share ACEs are visible |
| 33 | Confirm destination NTFS ACLs | Destination Server | `icacls "D:\Shares\Departments"` | Expected NTFS permissions are visible |
| 34 | Test destination share with admin | Management Host | `Test-Path "\\FSNEW01\Departments"` | Destination share is reachable |
| 35 | Test destination share with pilot user | Client | `Test-Path "\\FSNEW01\Departments"` | Pilot user has expected access |
| 36 | Test write on destination with approved test user | Client | `"migration-test" | Out-File "\\FSNEW01\Departments\migration-test.txt"` | Write succeeds only for expected users |
| 37 | Remove pilot test file | Client | `Remove-Item "\\FSNEW01\Departments\migration-test.txt" -Force` | Test file is removed |
| 38 | Notify users of freeze window | Operator | `Send change notification` | Users are told to stop file activity |
| 39 | Disable source share access during final sync | Source Server | `Block users by temporarily removing share access or setting read-only per change plan` | User writes are stopped |
| 40 | Confirm no active source open files | Source Server | `Get-SmbOpenFile | Where-Object Path -like "D:\Shares\Departments*"` | No blocking handles remain |
| 41 | Run final delta Robocopy | Destination Server | `robocopy "\\FSOLD01\D$\Shares\Departments" "D:\Shares\Departments" /MIR /COPYALL /DCOPY:DAT /SECFIX /TIMFIX /ZB /R:1 /W:1 /MT:16 /XJ /TEE /LOG:C:\FileMigration\Logs\final-Departments.log` | Final changes are synchronized |
| 42 | Capture final Robocopy exit code | Destination Server | `$LASTEXITCODE` | Exit code under 8 means no Robocopy failure |
| 43 | Cut over DFS path if used | DFS Server | `Set-DfsnFolderTarget -Path "\\corp.local\Shares\Departments" -TargetPath "\\FSNEW01\Departments" -State Online` | DFS namespace points to destination |
| 44 | Cut over DNS alias if used | DNS Server | `Update files.corp.local alias or A record to destination` | Alias points to new server |
| 45 | Update login scripts or GPO drive maps if used | Management Host | `Update mapped drive target from old UNC to new UNC` | Clients receive new path |
| 46 | Validate user access after cutover | Client | `Test-Path "\\corp.local\Shares\Departments"` | User path resolves to migrated data |
| 47 | Validate destination SMB sessions | Destination Server | `Get-SmbSession` | Users connect to destination server |
| 48 | Validate destination open files | Destination Server | `Get-SmbOpenFile` | Active files are on destination |
| 49 | Leave source data intact | Source Server | `Do not delete source data during validation window` | Rollback remains possible |
| 50 | Document final state | Operator | `Record inventory, Robocopy logs, share ACLs, NTFS ACLs, cutover time, validation, and rollback point` | Migration record is complete |

# Migrate_File_Server_Data_Shares_And_Permissions_Source_Inventory_Skeleton
```powershell
# Run in elevated PowerShell on the source file server.
# Purpose: inventory the source file server before migration.

$ExportRoot = "C:\FileMigration\Exports"
$ReportRoot = "C:\FileMigration\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ExportRoot | Out-Null
New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Source identity:"
hostname | Out-File "$ReportRoot\source-hostname-$Timestamp.txt"

Write-Host "OS version:"
Get-ComputerInfo |
  Select-Object WindowsProductName, WindowsVersion, OsName |
  Export-Csv "$ReportRoot\source-os-$Timestamp.csv" -NoTypeInformation

Write-Host "Volumes:"
Get-Volume |
  Select-Object DriveLetter, FileSystemLabel, FileSystem, HealthStatus, Size, SizeRemaining |
  Export-Csv "$ReportRoot\source-volumes-$Timestamp.csv" -NoTypeInformation

Write-Host "SMB server configuration:"
Get-SmbServerConfiguration |
  Export-Clixml "$ExportRoot\source-smb-server-config-$Timestamp.xml"

Write-Host "SMB shares:"
Get-SmbShare |
  Where-Object Special -eq $false |
  Export-Clixml "$ExportRoot\source-smb-shares-$Timestamp.xml"

Write-Host "SMB share access:"
$ShareAccess = foreach ($Share in Get-SmbShare | Where-Object Special -eq $false) {
  Get-SmbShareAccess -Name $Share.Name |
    Select-Object @{Name="ShareName";Expression={$Share.Name}}, AccountName, AccessControlType, AccessRight
}
$ShareAccess |
  Export-Csv "$ExportRoot\source-smb-share-access-$Timestamp.csv" -NoTypeInformation

Write-Host "Open SMB files:"
Get-SmbOpenFile |
  Export-Csv "$ReportRoot\source-open-files-$Timestamp.csv" -NoTypeInformation

Write-Host "Local groups:"
Get-LocalGroup |
  Export-Clixml "$ExportRoot\source-local-groups-$Timestamp.xml"

Write-Host "Local group members:"
$LocalGroupMembers = foreach ($Group in Get-LocalGroup) {
  Get-LocalGroupMember -Group $Group.Name -ErrorAction SilentlyContinue |
    Select-Object @{Name="GroupName";Expression={$Group.Name}}, Name, ObjectClass, PrincipalSource, SID
}
$LocalGroupMembers |
  Export-Csv "$ExportRoot\source-local-group-members-$Timestamp.csv" -NoTypeInformation

Write-Host "Source inventory exported to $ExportRoot and $ReportRoot"
```

# Migrate_File_Server_Data_Shares_And_Permissions_Share_Export_Skeleton
```powershell
# Run in elevated PowerShell on the source file server.
# Purpose: export SMB share definitions and share-level permissions.

$ExportRoot = "C:\FileMigration\Exports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ExportRoot | Out-Null

$Shares = Get-SmbShare | Where-Object Special -eq $false

$Shares |
  Select-Object `
    Name,
    Path,
    Description,
    CachingMode,
    FolderEnumerationMode,
    EncryptData,
    CompressData,
    ContinuouslyAvailable,
    ConcurrentUserLimit |
  Export-Csv "$ExportRoot\smb-shares-$Timestamp.csv" -NoTypeInformation

$ShareAccess = foreach ($Share in $Shares) {
  Get-SmbShareAccess -Name $Share.Name |
    Select-Object `
      @{Name="ShareName";Expression={$Share.Name}},
      AccountName,
      AccessControlType,
      AccessRight
}

$ShareAccess |
  Export-Csv "$ExportRoot\smb-share-access-$Timestamp.csv" -NoTypeInformation

$Shares |
  Export-Clixml "$ExportRoot\smb-shares-$Timestamp.xml"

$ShareAccess |
  Export-Clixml "$ExportRoot\smb-share-access-$Timestamp.xml"

Write-Host "Share export complete:"
Write-Host "$ExportRoot\smb-shares-$Timestamp.csv"
Write-Host "$ExportRoot\smb-share-access-$Timestamp.csv"
```

# Migrate_File_Server_Data_Shares_And_Permissions_NTFS_ACL_Export_Skeleton
```powershell
# Run in elevated PowerShell on the source file server.
# Purpose: export NTFS ACLs before migration.

$SourceRoot = "D:\Shares"
$ExportRoot = "C:\FileMigration\Exports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ExportRoot | Out-Null

Write-Host "Saving NTFS ACLs using icacls..."
icacls $SourceRoot /save "$ExportRoot\ntfs-acls-$Timestamp.txt" /t /c

Write-Host "Exporting top-level ACLs using PowerShell for review..."
Get-ChildItem $SourceRoot -Directory -Force |
  ForEach-Object {
    $Acl = Get-Acl $_.FullName
    [PSCustomObject]@{
      Path  = $_.FullName
      Owner = $Acl.Owner
      Access = ($Acl.Access | ForEach-Object {
        "$($_.IdentityReference):$($_.FileSystemRights):$($_.AccessControlType):$($_.IsInherited)"
      }) -join "; "
    }
  } |
  Export-Csv "$ExportRoot\top-level-ntfs-acls-$Timestamp.csv" -NoTypeInformation

Write-Host "NTFS ACL export complete."
```

# Migrate_File_Server_Data_Shares_And_Permissions_Target_Folder_Preparation_Skeleton
```powershell
# Run in elevated PowerShell on the destination file server.
# Purpose: prepare destination folder structure before copying data.

$DestinationRoot = "D:\Shares"
$MigrationRoot = "C:\FileMigration"
$Subfolders = @(
  "Departments",
  "Home",
  "Shared",
  "Archive"
)

New-Item -ItemType Directory -Force -Path "$MigrationRoot\Logs" | Out-Null
New-Item -ItemType Directory -Force -Path "$MigrationRoot\Exports" | Out-Null
New-Item -ItemType Directory -Force -Path "$MigrationRoot\Reports" | Out-Null

Write-Host "Creating destination root..."
New-Item -ItemType Directory -Force -Path $DestinationRoot | Out-Null

foreach ($Folder in $Subfolders) {
  $Path = Join-Path $DestinationRoot $Folder
  New-Item -ItemType Directory -Force -Path $Path | Out-Null
}

Write-Host "Destination folder structure:"
Get-ChildItem $DestinationRoot -Force |
  Select-Object FullName, LastWriteTime |
  Format-Table -AutoSize

Write-Host "Destination volume capacity:"
Get-PSDrive -Name D
```

# Migrate_File_Server_Data_Shares_And_Permissions_Initial_Robocopy_Skeleton
```powershell
# Run in elevated PowerShell on the destination file server.
# Purpose: perform initial bulk copy while users may still be active on source.

$SourceServer = "FSOLD01"
$SourcePath = "\\FSOLD01\D$\Shares\Departments"
$DestinationPath = "D:\Shares\Departments"
$LogRoot = "C:\FileMigration\Logs"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$LogFile = "$LogRoot\initial-Departments-$Timestamp.log"

New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null
New-Item -ItemType Directory -Force -Path $DestinationPath | Out-Null

Write-Host "Testing source path:"
if (-not (Test-Path $SourcePath)) {
  throw "Source path is not reachable: $SourcePath"
}

Write-Host "Testing destination path:"
if (-not (Test-Path $DestinationPath)) {
  throw "Destination path is not reachable: $DestinationPath"
}

Write-Host "Starting initial Robocopy pass..."
robocopy `
  $SourcePath `
  $DestinationPath `
  /MIR `
  /COPYALL `
  /DCOPY:DAT `
  /ZB `
  /R:2 `
  /W:5 `
  /MT:16 `
  /XJ `
  /TEE `
  /LOG:$LogFile

$ExitCode = $LASTEXITCODE
Write-Host "Robocopy exit code: $ExitCode"

if ($ExitCode -ge 8) {
  throw "Robocopy reported a failure. Review log: $LogFile"
}

Write-Host "Initial Robocopy complete. Log: $LogFile"
```

# Migrate_File_Server_Data_Shares_And_Permissions_Delta_Robocopy_Skeleton
```powershell
# Run in elevated PowerShell on the destination file server during the freeze window.
# Purpose: perform final delta copy after user writes are stopped.

$SourcePath = "\\FSOLD01\D$\Shares\Departments"
$DestinationPath = "D:\Shares\Departments"
$LogRoot = "C:\FileMigration\Logs"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$LogFile = "$LogRoot\final-delta-Departments-$Timestamp.log"

New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null

Write-Host "Starting final delta Robocopy pass..."
robocopy `
  $SourcePath `
  $DestinationPath `
  /MIR `
  /COPYALL `
  /DCOPY:DAT `
  /SECFIX `
  /TIMFIX `
  /ZB `
  /R:1 `
  /W:1 `
  /MT:16 `
  /XJ `
  /TEE `
  /LOG:$LogFile

$ExitCode = $LASTEXITCODE
Write-Host "Robocopy exit code: $ExitCode"

if ($ExitCode -ge 8) {
  throw "Final delta Robocopy failed. Review log: $LogFile"
}

Write-Host "Final delta Robocopy complete. Log: $LogFile"
```

# Migrate_File_Server_Data_Shares_And_Permissions_Recreate_SMB_Shares_Skeleton
```powershell
# Run in elevated PowerShell on the destination file server.
# Purpose: recreate SMB shares on the destination server.

$ShareDefinitions = @(
  @{
    Name                  = "Departments"
    Path                  = "D:\Shares\Departments"
    Description           = "Migrated Departments share"
    CachingMode           = "None"
    FolderEnumerationMode = "AccessBased"
    EncryptData           = $false
  },
  @{
    Name                  = "Shared"
    Path                  = "D:\Shares\Shared"
    Description           = "Migrated Shared share"
    CachingMode           = "None"
    FolderEnumerationMode = "AccessBased"
    EncryptData           = $false
  }
)

foreach ($Share in $ShareDefinitions) {
  Write-Host "Processing share $($Share.Name)"

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
      -EncryptData $Share.EncryptData
  }
  else {
    Write-Host "Share already exists: $($Share.Name)"
  }

  Get-SmbShare -Name $Share.Name |
    Select-Object Name, Path, Description, CachingMode, FolderEnumerationMode, EncryptData |
    Format-List
}
```

# Migrate_File_Server_Data_Shares_And_Permissions_Reapply_Share_Permissions_Skeleton
```powershell
# Run in elevated PowerShell on the destination file server.
# Purpose: apply SMB share permissions after share recreation.

$SharePermissions = @(
  @{
    ShareName   = "Departments"
    AccountName = "CORP\GG_FS_Departments_Read"
    AccessRight = "Read"
  },
  @{
    ShareName   = "Departments"
    AccountName = "CORP\GG_FS_Departments_Modify"
    AccessRight = "Change"
  },
  @{
    ShareName   = "Departments"
    AccountName = "CORP\Domain Admins"
    AccessRight = "Full"
  }
)

foreach ($Permission in $SharePermissions) {
  Write-Host "Granting $($Permission.AccessRight) on $($Permission.ShareName) to $($Permission.AccountName)"

  Grant-SmbShareAccess `
    -Name $Permission.ShareName `
    -AccountName $Permission.AccountName `
    -AccessRight $Permission.AccessRight `
    -Force
}

Write-Host "Final share access:"
Get-SmbShare | Where-Object Special -eq $false | ForEach-Object {
  Get-SmbShareAccess -Name $_.Name |
    Select-Object @{Name="ShareName";Expression={$_.Name}}, AccountName, AccessControlType, AccessRight
}
```

# Migrate_File_Server_Data_Shares_And_Permissions_Cutover_Skeleton
```powershell
# Run during approved cutover window.
# Purpose: stop source writes, perform final sync, validate destination, and cut users over.

$SourceServer = "FSOLD01"
$DestinationServer = "FSNEW01"
$ShareName = "Departments"
$SourcePath = "\\FSOLD01\D$\Shares\Departments"
$DestinationPath = "D:\Shares\Departments"
$LogRoot = "C:\FileMigration\Logs"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$FinalLog = "$LogRoot\cutover-final-$ShareName-$Timestamp.log"

Write-Host "Step 1: Check open files on source."
Invoke-Command -ComputerName $SourceServer -ArgumentList $ShareName -ScriptBlock {
  param($ShareName)

  Get-SmbOpenFile |
    Where-Object Path -like "*\$ShareName\*" |
    Select-Object FileId, SessionId, ClientComputerName, ClientUserName, Path, Locks |
    Format-Table -AutoSize
}

Write-Host "Step 2: Confirm user freeze is active before final copy."
Read-Host "Press Enter only after writes are stopped or source access is blocked"

Write-Host "Step 3: Run final Robocopy delta."
robocopy `
  $SourcePath `
  $DestinationPath `
  /MIR `
  /COPYALL `
  /DCOPY:DAT `
  /SECFIX `
  /TIMFIX `
  /ZB `
  /R:1 `
  /W:1 `
  /MT:16 `
  /XJ `
  /TEE `
  /LOG:$FinalLog

$ExitCode = $LASTEXITCODE
Write-Host "Robocopy exit code: $ExitCode"

if ($ExitCode -ge 8) {
  throw "Final copy failed. Do not cut over. Review $FinalLog"
}

Write-Host "Step 4: Validate destination share."
Test-Path "\\$DestinationServer\$ShareName"

Write-Host "Step 5: Validate destination SMB share and access."
Invoke-Command -ComputerName $DestinationServer -ArgumentList $ShareName -ScriptBlock {
  param($ShareName)

  Get-SmbShare -Name $ShareName | Format-List *
  Get-SmbShareAccess -Name $ShareName
}

Write-Host "Proceed with DFS, DNS, or drive-map cutover outside this script."
```

# Migrate_File_Server_Data_Shares_And_Permissions_DFS_Or_DNS_Update_Skeleton
```powershell
# Run on DFS or DNS management host as appropriate.
# Purpose: switch stable user path to the new destination file server.

$DfsPath = "\\corp.local\Shares\Departments"
$OldTarget = "\\FSOLD01\Departments"
$NewTarget = "\\FSNEW01\Departments"

$DnsZone = "corp.local"
$AliasName = "files"
$NewTargetFqdn = "FSNEW01.corp.local"

Write-Host "DFS cutover option:"
Write-Host "Disable old DFS target and enable new DFS target."

# Add new target if missing.
if (-not (Get-DfsnFolderTarget -Path $DfsPath -ErrorAction SilentlyContinue | Where-Object TargetPath -eq $NewTarget)) {
  New-DfsnFolderTarget -Path $DfsPath -TargetPath $NewTarget
}

Set-DfsnFolderTarget -Path $DfsPath -TargetPath $NewTarget -State Online
Set-DfsnFolderTarget -Path $DfsPath -TargetPath $OldTarget -State Offline

Get-DfsnFolderTarget -Path $DfsPath

Write-Host "DNS alias cutover option:"
Write-Host "Update CNAME manually if using DNS alias model."

# Example CNAME replacement pattern:
# Remove-DnsServerResourceRecord -ZoneName $DnsZone -RRType CName -Name $AliasName -Force
# Add-DnsServerResourceRecordCName -ZoneName $DnsZone -Name $AliasName -HostNameAlias $NewTargetFqdn
# Resolve-DnsName "$AliasName.$DnsZone"
```

# Migrate_File_Server_Data_Shares_And_Permissions_Post_Migration_Validation_Skeleton
```powershell
# Run from a management host and a pilot client.
# Purpose: validate migrated data, access, sessions, and common user workflows.

$SourceServer = "FSOLD01"
$DestinationServer = "FSNEW01"
$ShareName = "Departments"
$SourcePath = "\\FSOLD01\D$\Shares\Departments"
$DestinationPath = "\\FSNEW01\D$\Shares\Departments"
$UserPath = "\\FSNEW01\Departments"
$ReportRoot = "C:\FileMigration\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Compare file counts."
$SourceCount = (Get-ChildItem $SourcePath -Recurse -Force -ErrorAction SilentlyContinue | Measure-Object).Count
$DestinationCount = (Get-ChildItem $DestinationPath -Recurse -Force -ErrorAction SilentlyContinue | Measure-Object).Count

[PSCustomObject]@{
  SourcePath       = $SourcePath
  DestinationPath  = $DestinationPath
  SourceCount      = $SourceCount
  DestinationCount = $DestinationCount
} | Export-Csv "$ReportRoot\file-count-compare-$Timestamp.csv" -NoTypeInformation

Write-Host "Validate user path."
Test-Path $UserPath

Write-Host "Write test."
$TestFile = Join-Path $UserPath "migration-validation-$Timestamp.txt"
"Migration validation from $env:COMPUTERNAME by $env:USERNAME at $(Get-Date)" |
  Out-File $TestFile -Encoding utf8

Write-Host "Read test."
Get-Content $TestFile

Write-Host "Remove test file."
Remove-Item $TestFile -Force

Write-Host "Destination share state."
Invoke-Command -ComputerName $DestinationServer -ArgumentList $ShareName -ScriptBlock {
  param($ShareName)

  Get-SmbShare -Name $ShareName | Format-List *
  Get-SmbShareAccess -Name $ShareName
  Get-SmbSession
  Get-SmbOpenFile
}
```

# Migrate_File_Server_Data_Shares_And_Permissions_Status_And_Event_Review_Skeleton
```powershell
# Run on source and destination servers.
# Purpose: export file server, SMB, and migration evidence.

$Servers = @("FSOLD01", "FSNEW01")
$ReportRoot = "C:\FileMigration\Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

foreach ($Server in $Servers) {
  Invoke-Command -ComputerName $Server -ArgumentList $ReportRoot, $Timestamp -ScriptBlock {
    param($ReportRoot, $Timestamp)

    New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

    Get-SmbShare |
      Where-Object Special -eq $false |
      Select-Object Name, Path, Description, CachingMode, FolderEnumerationMode, EncryptData |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-shares-$Timestamp.csv" -NoTypeInformation

    $Access = foreach ($Share in Get-SmbShare | Where-Object Special -eq $false) {
      Get-SmbShareAccess -Name $Share.Name |
        Select-Object @{Name="ShareName";Expression={$Share.Name}}, AccountName, AccessControlType, AccessRight
    }

    $Access |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-share-access-$Timestamp.csv" -NoTypeInformation

    Get-SmbSession -ErrorAction SilentlyContinue |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-smb-sessions-$Timestamp.csv" -NoTypeInformation

    Get-SmbOpenFile -ErrorAction SilentlyContinue |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-open-files-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-smbserver-events-$Timestamp.csv" -NoTypeInformation

    Get-WinEvent -LogName System -MaxEvents 300 -ErrorAction SilentlyContinue |
      Where-Object Message -like "*LanmanServer*" |
      Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$env:COMPUTERNAME-lanmanserver-events-$Timestamp.csv" -NoTypeInformation
  }
}

Write-Host "Migration status and event reports exported under $ReportRoot"
```

# Migrate_File_Server_Data_Shares_And_Permissions_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify source shares | `Get-SmbShare | Where-Object Special -eq $false` | Source shares are known |
| Verify source share access | `Get-SmbShareAccess -Name "Departments"` | Source share permissions are documented |
| Verify source NTFS ACL | `icacls "D:\Shares\Departments"` | Source NTFS permissions are visible |
| Verify destination folder | `Test-Path "D:\Shares\Departments"` | Destination folder exists |
| Verify destination share | `Get-SmbShare -Name "Departments"` | Destination share exists |
| Verify destination share access | `Get-SmbShareAccess -Name "Departments"` | Destination share permissions match design |
| Verify destination NTFS ACL | `icacls "D:\Shares\Departments"` | Destination NTFS permissions match design |
| Verify admin path to source | `Test-Path "\\FSOLD01\D$\Shares"` | Source admin path is reachable |
| Verify Robocopy log | `Select-String "ERROR" "C:\FileMigration\Logs\final-Departments.log"` | No blocking errors appear |
| Verify Robocopy exit code | `$LASTEXITCODE` | Value under 8 is acceptable |
| Verify source file count | `(Get-ChildItem "\\FSOLD01\D$\Shares\Departments" -Recurse -Force | Measure-Object).Count` | Source count is captured |
| Verify destination file count | `(Get-ChildItem "\\FSNEW01\D$\Shares\Departments" -Recurse -Force | Measure-Object).Count` | Destination count is captured |
| Verify destination user UNC | `Test-Path "\\FSNEW01\Departments"` | Destination user path works |
| Verify DFS path if used | `Get-DfsnFolderTarget -Path "\\corp.local\Shares\Departments"` | DFS target points to destination |
| Verify DNS alias if used | `Resolve-DnsName files.corp.local` | Alias resolves to destination |
| Verify client SMB connection | `Get-SmbConnection | Where-Object ServerName -like "*FSNEW01*"` | Client is connected to destination |
| Verify active sessions | `Get-SmbSession` on destination | Users are connecting to destination |
| Verify open files | `Get-SmbOpenFile` on destination | Active file handles are on destination |
| Verify source quiesced | `Get-SmbSession` on source | Source has no normal user activity after cutover |
| Verify event logs | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 50` | SMB events are visible |

# Migrate_File_Server_Data_Shares_And_Permissions_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| DFS target moved to destination | `Set-DfsnFolderTarget -Path "\\corp.local\Shares\Departments" -TargetPath "\\FSOLD01\Departments" -State Online` | DFS path returns to source |
| Destination DFS target enabled | `Set-DfsnFolderTarget -Path "\\corp.local\Shares\Departments" -TargetPath "\\FSNEW01\Departments" -State Offline` | Destination target stops receiving users |
| DNS alias moved to destination | Restore DNS CNAME or A record to old server | Alias returns to source |
| GPO drive map updated | Restore old UNC path in Group Policy Preferences | Drive maps return to source |
| Login script updated | Revert script path to `\\FSOLD01\Departments` | Users map old path again |
| Source share access blocked | Reapply original `Grant-SmbShareAccess` entries | Source share is usable again |
| Source share set read-only | Restore original Change or Full share permissions | Source writes are restored |
| Destination SMB share created | `Remove-SmbShare -Name "Departments" -Force` | Destination share is removed |
| Destination share permissions changed | `Revoke-SmbShareAccess -Name "Departments" -AccountName "<account>" -Force` | Incorrect permission is removed |
| Destination folder created | `Remove-Item "D:\Shares\Departments" -Recurse -Force` | Destination data is removed only after rollback decision |
| Robocopy mirrored wrong data | Restore from source, backup, or previous clean destination snapshot | Destination is corrected |
| NTFS ACLs incorrect | `icacls "D:\Shares" /restore "C:\FileMigration\Exports\ntfs-acls.txt"` | Saved ACLs are restored if path mapping is valid |
| Test files created | `Remove-Item "\\FSNEW01\Departments\migration-validation*.txt" -Force` | Test files are removed |
| Migration report folder created | `Remove-Item "C:\FileMigration" -Recurse -Force` | Local migration artifacts are removed after retention window |

# Migrate_File_Server_Data_Shares_And_Permissions_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Robocopy access denied | Migration account lacks rights, backup mode unavailable, locked file, or admin share blocked | Robocopy log and `whoami /priv` | Run elevated, use local admin, use `/ZB`, confirm admin shares |
| Robocopy exit code 8 or higher | One or more copy failures | `$LASTEXITCODE`; Robocopy log | Fix errors and rerun delta |
| Destination missing ACLs | Wrong Robocopy copy flags | Review command for `/COPYALL` or `/SEC` | Rerun with `/COPYALL /SECFIX` |
| Destination missing owner | Owner flag not copied or privileges missing | `Get-Acl` source and destination | Rerun as admin with `/COPYALL` |
| Destination missing audit entries | SACL not copied or privilege missing | `Get-Acl -Audit` where applicable | Rerun with proper privilege and `/COPYALL` |
| Timestamps changed | Missing `/DCOPY:DAT` or `/TIMFIX` | Compare timestamps | Rerun with `/DCOPY:DAT /TIMFIX` |
| Extra files remain on destination | Used `/E` instead of `/MIR` or skipped purge | Compare destination extras | Use `/MIR` only after confirming correct source and destination |
| Files deleted unexpectedly | `/MIR` used against wrong destination | Robocopy log and path review | Stop, restore from backup or source |
| Local group permissions break | Source local groups do not exist on destination | `icacls`; orphaned SIDs | Replace local groups with domain groups or recreate local groups carefully |
| Orphaned SIDs appear | Deleted users, old local groups, or domain migration leftovers | `icacls` output | Clean ACLs after approval |
| Share permissions missing | Share permissions were not recreated | `Get-SmbShareAccess` | Apply exported share permissions |
| User can access share but not folder | NTFS ACL missing or ABE behavior | `icacls`; `Get-SmbShare` | Correct NTFS ACL or ABE setting |
| User can read but not write | Share Change or NTFS Modify missing | `Get-SmbShareAccess`; `icacls` | Grant correct least-privilege group |
| User gets credential prompt | Wrong path, cached credentials, SPN, or DNS alias issue | `cmdkey /list`; `klist`; `Resolve-DnsName` | Clear cached credentials and fix DNS/SPN |
| Client still hits old server | DFS referral cache, DNS cache, mapped drive, or Offline Files | `Get-SmbConnection`; `dfsutil /pktinfo`; `ipconfig /displaydns` | Flush cache and update mapping |
| DFS path points wrong | Target state or referral priority wrong | `Get-DfsnFolderTarget` | Set correct target online and old target offline |
| DNS alias points wrong | CNAME or A record stale | `Resolve-DnsName` | Update DNS and flush client DNS cache |
| Open files block final sync | Users still connected | `Get-SmbOpenFile` on source | Close specific handles after approval |
| Copy is too slow | Low bandwidth, small files, antivirus, storage bottleneck, too few threads | Robocopy throughput and performance counters | Tune `/MT`, schedule off-hours, exclude AV scanning if approved |
| Destination fills up | Underestimated space or duplicate data | `Get-PSDrive D` | Expand volume or reduce scope |
| Long paths fail | Legacy path handling or application issue | Robocopy log | Enable long paths where appropriate and review failing paths |
| EFS files fail | Encrypted files need special handling | Robocopy log | Use `/EFSRAW` only when design requires it |
| Offline Files conflicts | Client cache stale | Sync Center and client cache | Disable or reset Offline Files cache as approved |
| FSRM settings missing | FSRM not migrated by Robocopy | FSRM reports and quota config | Export/import FSRM config separately |
| Shadow Copies missing | VSS settings not migrated by Robocopy | `vssadmin list shadowstorage` | Reconfigure shadow copies on destination |
| Users complain after cutover | App hardcoded old UNC or path | App config and recent file lists | Update app paths, DFS, DNS alias, or drive maps |

# Migrate_File_Server_Data_Shares_And_Permissions_Related_Labs
| Lab                                                                       | Relationship                                                                        |
| ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `01_Install_File_Server_Role_And_Management_Tools.md`                     | Destination server must have file server role and management tools                  |
| `02_Create_And_Publish_SMB_Shares.md`                                     | Migrated shares must be recreated and published correctly                           |
| `03_Configure_NTFS_And_Share_Permissions.md`                              | NTFS and share permissions are the core migration risk                              |
| `04_Configure_Access_Based_Enumeration_And_Share_Visibility.md`           | ABE settings must be preserved where used                                           |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`            | SMB security settings should be compared between source and destination             |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`                 | Home folders and department shares are common migration candidates                  |
| `08_Configure_File_Screening_With_FSRM.md`                                | FSRM file screens may need separate export and import                               |
| `09_Configure_Storage_Quotas_With_FSRM.md`                                | FSRM quotas may need separate export and import                                     |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md`          | Storage reports help scope the migration                                            |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md`                     | Previous Versions settings should be rebuilt on destination                         |
| `12_Backup_Restore_And_Export_File_Server_Config.md`                      | Backup and export are mandatory before migration                                    |
| `13_Configure_DFS_Namespaces.md`                                          | DFS namespace cutover reduces user path changes                                     |
| `14_Configure_DFS_Replication.md`                                         | DFSR may be involved in staged migrations but is not the same as Robocopy           |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`            | Open files and SMB sessions must be monitored before final sync                     |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`                   | Post-cutover issues usually resolve through SMB troubleshooting                     |
| `17_Configure_Data_Deduplication_For_File_Shares.md`                      | Deduplication state may need review before and after migration                      |
| `22_Configure_Azure_File_Sync_For_Windows_File_Server.md`                 | Azure File Sync can be part of hybrid migration design                              |
| `23_Configure_Storage_Replica_For_File_Server_Disaster_Recovery.md`       | Storage Replica is a DR strategy, not a normal permission-preserving file migration |
| `24_Configure_Scale_Out_File_Server_And_Continuously_Available_Shares.md` | SOFS migrations require cluster and application workload validation                 |