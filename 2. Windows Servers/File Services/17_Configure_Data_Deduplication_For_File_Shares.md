17_Configure_Data_Deduplication_For_File_Shares.md
# Configure_Data_Deduplication_For_File_Shares

# Configure_Data_Deduplication_For_File_Shares_Index
17_Configure_Data_Deduplication_For_File_Shares.md
Configure_Data_Deduplication_For_File_Shares
Configure_Data_Deduplication_For_File_Shares_Source_Basis
Configure_Data_Deduplication_For_File_Shares_Mental_Model
Configure_Data_Deduplication_For_File_Shares_Planning_Table
Configure_Data_Deduplication_For_File_Shares_Configuration_Checklist
Configure_Data_Deduplication_For_File_Shares_Feature_Install_Skeleton
Configure_Data_Deduplication_For_File_Shares_Volume_Enablement_Skeleton
Configure_Data_Deduplication_For_File_Shares_Policy_Tuning_Skeleton
Configure_Data_Deduplication_For_File_Shares_Schedule_Skeleton
Configure_Data_Deduplication_For_File_Shares_Manual_Optimization_Skeleton
Configure_Data_Deduplication_For_File_Shares_Status_And_Reporting_Skeleton
Configure_Data_Deduplication_For_File_Shares_Verification_Commands
Configure_Data_Deduplication_For_File_Shares_Rollback
Configure_Data_Deduplication_For_File_Shares_Failure_Checks
Configure_Data_Deduplication_For_File_Shares_Related_Labs

# Configure_Data_Deduplication_For_File_Shares_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server Data Deduplication | Data Deduplication overview | Deduplication purpose, workload fit, and space savings expectations |
| PowerShell Deduplication module | Enable-DedupVolume | Enabling deduplication on supported volumes |
| PowerShell Deduplication module | Set-DedupVolume | Tuning file age, excluded folders, excluded file types, and policy settings |
| PowerShell Deduplication module | New-DedupSchedule | Creating optimization, garbage collection, and scrubbing schedules |
| PowerShell Deduplication module | Start-DedupJob | Starting manual optimization, garbage collection, scrubbing, and unoptimization jobs |
| Operational file server practice | File share volume planning | Keeping deduplication volume-based, not share-based |
| Operational support practice | Monitoring and rollback | Validating savings, jobs, events, and safe unoptimization |

# Configure_Data_Deduplication_For_File_Shares_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Data Deduplication | Windows Server storage feature that reduces duplicate data stored on a volume |
| Volume-level feature | Deduplication is enabled on a volume, not directly on an SMB share |
| Share impact | Shares on the deduplicated volume benefit from savings if their files qualify |
| Chunk store | Deduplicated data is stored as shared chunks plus metadata |
| Optimization | Job that deduplicates and compresses eligible files |
| Garbage collection | Job that cleans up unreferenced chunks after data changes or deletion |
| Scrubbing | Job that validates deduplication metadata and chunk integrity |
| Unoptimization | Job that reverses deduplication for optimized files |
| Minimum file age | Policy that prevents very new or active files from being optimized too soon |
| Excluded folder | Folder tree skipped by deduplication |
| Excluded file type | File extension skipped by deduplication |
| General purpose file server | Normal user and department shares, usually `UsageType Default` |
| Hot files | Frequently changing files that should usually be delayed or excluded |
| Bad candidates | Databases, constantly changing application data, unsupported workloads, or volumes without enough free space |
| Blunt rule | Deduplication is not a substitute for quotas, cleanup, backups, or capacity planning |

# Configure_Data_Deduplication_For_File_Shares_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server | `FS1` | `<file-server-name>` |
| Target volume | `D:` | `<target-volume>` |
| Target share root | `D:\Shares` | `<share-root>` |
| Target SMB share | `Departments` | `<share-name>` |
| Workload type | General purpose file server | `Default` |
| Minimum file age | `3` days | `<minimum-file-age-days>` |
| Excluded folders | `D:\Shares\Databases`, `D:\Shares\Temp` | `<excluded-folders>` |
| Excluded file types | `tmp`, `mdf`, `ldf`, `edb` | `<excluded-file-types>` |
| Optimization window | Weekdays 23:00 for 6 hours | `<optimization-window>` |
| Garbage collection window | Sunday 02:00 for 5 hours | `<gc-window>` |
| Scrubbing window | Sunday 04:00 for 4 hours | `<scrubbing-window>` |
| Manual first optimization | Yes / No | `<manual-optimization>` |
| Memory limit | `50` percent | `<memory-percent>` |
| Stop when busy | Yes | `<stop-when-busy>` |
| Monitoring path | `C:\Dedup-Reports` | `<report-path>` |
| Rollback stance | Unoptimize before disabling | `<rollback-plan>` |
| Backup confirmed | Yes / No | `<backup-confirmed>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Configure_Data_Deduplication_For_File_Shares_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm file server identity | File Server | `hostname` | Correct server is identified |
| 2 | Confirm target volume exists | File Server | `Get-Volume -DriveLetter D` | Target volume is visible |
| 3 | Confirm target path exists | File Server | `Test-Path "D:\Shares"` | Share root exists |
| 4 | Confirm SMB shares on volume | File Server | `Get-SmbShare | Where-Object Path -like "D:\*"` | Shares hosted on target volume are visible |
| 5 | Confirm free space before enabling | File Server | `Get-PSDrive D` | Free space is documented |
| 6 | Confirm file system and health | File Server | `Get-Volume -DriveLetter D | Select DriveLetter,FileSystem,HealthStatus,Size,SizeRemaining` | Volume health and file system are known |
| 7 | Confirm backup exists | File Server | `wbadmin get versions` | Backup state is known if Windows Server Backup is used |
| 8 | Confirm deduplication feature state | File Server | `Get-WindowsFeature FS-Data-Deduplication` | Feature state is visible |
| 9 | Install Data Deduplication feature | File Server | `Install-WindowsFeature FS-Data-Deduplication -IncludeManagementTools` | Feature installs successfully |
| 10 | Import Deduplication module | File Server | `Import-Module Deduplication` | Cmdlets are available |
| 11 | Inventory deduplication cmdlets | File Server | `Get-Command -Module Deduplication` | Deduplication commands are listed |
| 12 | Confirm current dedup volumes | File Server | `Get-DedupVolume` | Existing dedup-enabled volumes are visible |
| 13 | Enable deduplication on target volume | File Server | `Enable-DedupVolume -Volume "D:" -UsageType Default` | Deduplication is enabled for general file server workload |
| 14 | Confirm deduplication volume policy | File Server | `Get-DedupVolume -Volume "D:" | Format-List *` | Deduplication policy is visible |
| 15 | Set minimum file age | File Server | `Set-DedupVolume -Volume "D:" -MinimumFileAgeDays 3` | Files younger than threshold are skipped |
| 16 | Set excluded folders | File Server | `Set-DedupVolume -Volume "D:" -ExcludeFolder "D:\Shares\Temp","D:\Shares\Databases"` | Excluded folders are configured |
| 17 | Set excluded file types | File Server | `Set-DedupVolume -Volume "D:" -ExcludeFileType "tmp","mdf","ldf","edb"` | Excluded extensions are configured |
| 18 | Review deduplication schedules | File Server | `Get-DedupSchedule` | Existing schedules are visible |
| 19 | Create weekday optimization schedule | File Server | `New-DedupSchedule -Name "WeekdayOptimization" -Type Optimization -Days Monday,Tuesday,Wednesday,Thursday,Friday -Start "23:00" -DurationHours 6 -StopWhenSystemBusy -Priority Low` | Optimization schedule is created |
| 20 | Create weekly garbage collection schedule | File Server | `New-DedupSchedule -Name "WeeklyGarbageCollection" -Type GarbageCollection -Days Sunday -Start "02:00" -DurationHours 5 -Priority Normal` | Garbage collection schedule is created |
| 21 | Create weekly scrubbing schedule | File Server | `New-DedupSchedule -Name "WeeklyScrubbing" -Type Scrubbing -Days Sunday -Start "04:00" -DurationHours 4 -Priority Normal` | Scrubbing schedule is created |
| 22 | Start manual optimization if approved | File Server | `Start-DedupJob -Volume "D:" -Type Optimization -Memory 50 -Priority Normal` | Manual optimization job starts or queues |
| 23 | Monitor active dedup jobs | File Server | `Get-DedupJob` | Running or queued jobs are visible |
| 24 | Review deduplication status | File Server | `Get-DedupStatus -Volume "D:"` | Savings and optimization state are visible |
| 25 | Review deduplication metadata | File Server | `Get-DedupMetadata -Volume "D:"` | Chunk store and metadata state are visible |
| 26 | Review event logs | File Server | `Get-WinEvent -LogName "Microsoft-Windows-Deduplication/Operational" -MaxEvents 100` | Deduplication operational events are visible |
| 27 | Export baseline report | File Server | `Get-DedupStatus -Volume "D:" | Export-Clixml "C:\Dedup-Reports\dedup-status.xml"` | Dedup status report is saved |
| 28 | Test SMB share access after enablement | Client | `Test-Path "\\FS1\Departments"` | Share remains accessible |
| 29 | Confirm files still open normally | Client | `notepad "\\FS1\Departments\smb-test.txt"` | User file access works |
| 30 | Document before and after state | Operator | `Record volume, policy, schedules, jobs, savings, and event results` | Change record is complete |

# Configure_Data_Deduplication_For_File_Shares_Feature_Install_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: install Data Deduplication role service and confirm cmdlet availability.

$FeatureName = "FS-Data-Deduplication"

Write-Host "Checking current Data Deduplication feature state..."
Get-WindowsFeature $FeatureName

Write-Host "Installing Data Deduplication..."
Install-WindowsFeature $FeatureName -IncludeManagementTools

Write-Host "Importing Deduplication module..."
Import-Module Deduplication

Write-Host "Available Deduplication cmdlets:"
Get-Command -Module Deduplication | Sort-Object Name

Write-Host "Feature state after install:"
Get-WindowsFeature $FeatureName
```

# Configure_Data_Deduplication_For_File_Shares_Volume_Enablement_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate target volume and enable deduplication for a general file share workload.

$Volume = "D:"
$DriveLetter = "D"
$ShareRoot = "D:\Shares"

Write-Host "Confirming target volume..."
Get-Volume -DriveLetter $DriveLetter |
  Select-Object DriveLetter, FileSystem, HealthStatus, Size, SizeRemaining |
  Format-List

Write-Host "Confirming share root path..."
Test-Path $ShareRoot

Write-Host "Confirming SMB shares hosted on target volume..."
Get-SmbShare |
  Where-Object Path -like "$Volume*" |
  Select-Object Name, Path, Description |
  Format-Table -AutoSize

Write-Host "Enabling Data Deduplication on $Volume..."
Enable-DedupVolume -Volume $Volume -UsageType Default

Write-Host "Deduplication volume state:"
Get-DedupVolume -Volume $Volume | Format-List *
```

# Configure_Data_Deduplication_For_File_Shares_Policy_Tuning_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: tune deduplication policy for normal file shares.

$Volume = "D:"
$MinimumFileAgeDays = 3

$ExcludedFolders = @(
  "D:\Shares\Temp",
  "D:\Shares\Databases",
  "D:\Shares\DoNotDedup"
)

$ExcludedFileTypes = @(
  "tmp",
  "mdf",
  "ldf",
  "edb"
)

Write-Host "Applying deduplication policy..."
Set-DedupVolume `
  -Volume $Volume `
  -MinimumFileAgeDays $MinimumFileAgeDays `
  -ExcludeFolder $ExcludedFolders `
  -ExcludeFileType $ExcludedFileTypes

Write-Host "Deduplication policy after tuning:"
Get-DedupVolume -Volume $Volume |
  Select-Object `
    Volume,
    UsageType,
    MinimumFileAgeDays,
    ExcludeFolder,
    ExcludeFileType,
    NoCompress,
    OptimizeInUseFiles |
  Format-List
```

# Configure_Data_Deduplication_For_File_Shares_Schedule_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create predictable off-hours deduplication schedules.

Write-Host "Existing deduplication schedules:"
Get-DedupSchedule | Format-Table -AutoSize

# Weekday optimization.
$OptimizationParams = @{
  Name               = "WeekdayOptimization"
  Type               = "Optimization"
  Days               = @("Monday","Tuesday","Wednesday","Thursday","Friday")
  Start              = "23:00"
  DurationHours      = 6
  StopWhenSystemBusy = $true
  Priority           = "Low"
}

# Weekly garbage collection.
$GarbageCollectionParams = @{
  Name          = "WeeklyGarbageCollection"
  Type          = "GarbageCollection"
  Days          = "Sunday"
  Start         = "02:00"
  DurationHours = 5
  Priority      = "Normal"
}

# Weekly scrubbing.
$ScrubbingParams = @{
  Name          = "WeeklyScrubbing"
  Type          = "Scrubbing"
  Days          = "Sunday"
  Start         = "04:00"
  DurationHours = 4
  Priority      = "Normal"
}

New-DedupSchedule @OptimizationParams
New-DedupSchedule @GarbageCollectionParams
New-DedupSchedule @ScrubbingParams

Write-Host "Schedules after creation:"
Get-DedupSchedule | Format-Table Name, Type, Enabled, Start, Days, DurationHours, Priority -AutoSize
```

# Configure_Data_Deduplication_For_File_Shares_Manual_Optimization_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: manually start first optimization after change approval.

$Volume = "D:"

Write-Host "Starting manual optimization job on $Volume..."
Start-DedupJob `
  -Volume $Volume `
  -Type Optimization `
  -Memory 50 `
  -Priority Normal

Write-Host "Current deduplication jobs:"
Get-DedupJob | Format-Table -AutoSize

Write-Host "Current deduplication status:"
Get-DedupStatus -Volume $Volume | Format-List
```

# Configure_Data_Deduplication_For_File_Shares_Status_And_Reporting_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: export deduplication state, jobs, schedules, and event evidence.

$Volume = "D:"
$ReportRoot = "C:\Dedup-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Get-DedupVolume -Volume $Volume |
  Export-Clixml -Path "$ReportRoot\dedup-volume-$Timestamp.xml"

Get-DedupStatus -Volume $Volume |
  Export-Clixml -Path "$ReportRoot\dedup-status-$Timestamp.xml"

Get-DedupMetadata -Volume $Volume |
  Export-Clixml -Path "$ReportRoot\dedup-metadata-$Timestamp.xml"

Get-DedupSchedule |
  Export-Clixml -Path "$ReportRoot\dedup-schedules-$Timestamp.xml"

Get-DedupJob |
  Export-Clixml -Path "$ReportRoot\dedup-jobs-$Timestamp.xml"

Get-WinEvent -LogName "Microsoft-Windows-Deduplication/Operational" -MaxEvents 300 -ErrorAction SilentlyContinue |
  Export-Clixml -Path "$ReportRoot\dedup-operational-events-$Timestamp.xml"

Write-Host "Deduplication reports exported to $ReportRoot"
```

# Configure_Data_Deduplication_For_File_Shares_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify feature installed | `Get-WindowsFeature FS-Data-Deduplication` | Install State shows installed |
| Verify module commands | `Get-Command -Module Deduplication` | Deduplication cmdlets are listed |
| Verify target volume | `Get-Volume -DriveLetter D` | Volume is healthy and online |
| Verify hosted shares | `Get-SmbShare | Where-Object Path -like "D:\*"` | Expected shares are on target volume |
| Verify dedup volume enabled | `Get-DedupVolume -Volume "D:"` | Volume appears as dedup-enabled |
| Verify policy settings | `Get-DedupVolume -Volume "D:" | Format-List *` | Minimum age, exclusions, and usage type are visible |
| Verify schedules | `Get-DedupSchedule` | Optimization, garbage collection, and scrubbing schedules exist |
| Verify running jobs | `Get-DedupJob` | Running or queued jobs are visible |
| Verify savings status | `Get-DedupStatus -Volume "D:"` | Savings, optimized files, and status are visible |
| Verify metadata | `Get-DedupMetadata -Volume "D:"` | Metadata and chunk store info are visible |
| Verify events | `Get-WinEvent -LogName "Microsoft-Windows-Deduplication/Operational" -MaxEvents 50` | Deduplication events are visible |
| Verify SMB access | `Test-Path "\\FS1\Departments"` | Share remains accessible |
| Verify file open | `notepad "\\FS1\Departments\smb-test.txt"` | File opens normally |
| Verify free space trend | `Get-PSDrive D` | Space is visible for before and after comparison |

# Configure_Data_Deduplication_For_File_Shares_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| Manual optimization job started | `Stop-DedupJob -Volume "D:"` | Running deduplication job stops |
| Optimization schedule created | `Remove-DedupSchedule -Name "WeekdayOptimization"` | Optimization schedule is removed |
| Garbage collection schedule created | `Remove-DedupSchedule -Name "WeeklyGarbageCollection"` | Garbage collection schedule is removed |
| Scrubbing schedule created | `Remove-DedupSchedule -Name "WeeklyScrubbing"` | Scrubbing schedule is removed |
| Excluded folders configured | `Set-DedupVolume -Volume "D:" -ExcludeFolder @()` | Excluded folder list is cleared |
| Excluded file types configured | `Set-DedupVolume -Volume "D:" -ExcludeFileType @()` | Excluded file type list is cleared |
| Minimum file age changed | `Set-DedupVolume -Volume "D:" -MinimumFileAgeDays 3` | Minimum file age returns to baseline |
| Deduplication enabled | `Start-DedupJob -Volume "D:" -Type Unoptimization -Wait` | Optimized files are rehydrated |
| Deduplication enabled | `Disable-DedupVolume -Volume "D:"` | Deduplication is disabled after unoptimization |
| Feature installed | `Uninstall-WindowsFeature FS-Data-Deduplication` | Feature is removed after deduplication is no longer needed |
| Reports created | `Remove-Item "C:\Dedup-Reports" -Recurse -Force` | Report folder is removed |

# Configure_Data_Deduplication_For_File_Shares_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `Enable-DedupVolume` fails | Unsupported volume, wrong drive, feature missing, or insufficient volume size | `Get-Volume`; `Get-WindowsFeature FS-Data-Deduplication` | Use supported data volume and install feature |
| Dedup cmdlets not found | Feature or module missing | `Get-Command -Module Deduplication` | Install feature and import module |
| No savings after enablement | Files too new, low duplication, excluded folders, or job has not run | `Get-DedupStatus`; `Get-DedupVolume`; `Get-DedupJob` | Wait for schedule or start optimization |
| Optimization never starts | Schedule missing, server busy, job queued, or policy excludes files | `Get-DedupSchedule`; `Get-DedupJob` | Fix schedule or run manual job |
| Optimization impacts users | Job running during business hours or too much resource usage | `Get-DedupJob`; performance counters | Move schedule off-hours, lower priority, use stop when busy |
| Share access breaks after change | Separate SMB or permission issue, not normal dedup behavior | `Test-Path "\\server\share"`; `Get-SmbShare`; event logs | Troubleshoot SMB path separately |
| Database files perform poorly | Active database files were optimized | `Get-DedupVolume`; review exclusions | Exclude database folders and unoptimize if needed |
| Chunk store grows unexpectedly | Heavy file churn or deleted optimized data not garbage collected | `Get-DedupMetadata`; `Get-DedupStatus` | Run garbage collection off-hours |
| Scrubbing reports corruption | Storage, metadata, or chunk issue | Deduplication operational log | Run scrubbing and verify storage health |
| Event log missing | Wrong log name or no dedup activity yet | `Get-WinEvent -ListLog "*Dedup*"` | Use discovered log and generate activity |
| Rollback takes a long time | Unoptimization is data-heavy | `Get-DedupJob`; `Get-DedupStatus` | Allow off-hours completion |
| Free space too low for rollback | Not enough room to rehydrate files | `Get-PSDrive D`; `Get-DedupStatus` | Add capacity before unoptimization |
| Backup size increases unexpectedly | Backup solution not dedup-aware or backs up rehydrated data | Backup job report | Confirm backup application behavior |
| Users save large duplicate files but no savings | Minimum file age not met yet | `Get-DedupVolume -Volume "D:"` | Wait for threshold or adjust minimum file age carefully |

# Configure_Data_Deduplication_For_File_Shares_Related_Labs
| Lab                                                              | Relationship                                                                     |
| ---------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `01_Install_File_Server_Role_And_Management_Tools.md`            | Provides baseline file server role and management tools                          |
| `02_Create_And_Publish_SMB_Shares.md`                            | Deduplication applies to the volumes hosting SMB shares                          |
| `03_Configure_NTFS_And_Share_Permissions.md`                     | Deduplication does not replace file access control                               |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`        | User home folders and department shares are common deduplication candidates      |
| `09_Configure_Storage_Quotas_With_FSRM.md`                       | Quotas and deduplication both affect storage capacity management                 |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | FSRM reports help identify duplicate-heavy or stale data                         |
| `12_Backup_Restore_And_Export_File_Server_Config.md`             | Backups must be confirmed before enabling or rolling back deduplication          |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`   | SMB monitoring validates user impact after deduplication enablement              |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`          | SMB troubleshooting separates access failures from storage optimization behavior |