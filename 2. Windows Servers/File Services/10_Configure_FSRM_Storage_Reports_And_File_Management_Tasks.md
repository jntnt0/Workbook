10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md
# Configure_FSRM_Storage_Reports_And_File_Management_Tasks

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Index
10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md
Configure_FSRM_Storage_Reports_And_File_Management_Tasks
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Source_Basis
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Mental_Model
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Planning_Table
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Configuration_Checklist
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Precheck_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Install_And_Module_Validation_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Interactive_Storage_Report_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Scheduled_Storage_Report_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Large_And_Duplicate_File_Report_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_File_Management_Expiration_Job_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_File_Management_Custom_Log_Job_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Manual_Job_Run_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Event_Validation_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Post_Change_Validation_Skeleton
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Verification_Commands
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Rollback
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Failure_Checks
Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Related_Labs

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | File Server Resource Manager | Managing reports, quotas, file screens, and file management tasks |
| Microsoft Learn | New-FsrmStorageReport | Creating on-demand or scheduled storage reports |
| Microsoft Learn | Get-FsrmStorageReport | Reviewing existing storage report jobs |
| Microsoft Learn | Remove-FsrmStorageReport | Removing report jobs during rollback |
| Microsoft Learn | New-FsrmScheduledTask | Creating weekly or monthly report and job schedules |
| Microsoft Learn | New-FsrmFileManagementJob | Creating scheduled file management jobs |
| Microsoft Learn | Get-FsrmFileManagementJob | Reviewing file management job configuration |
| Microsoft Learn | Set-FsrmFileManagementJob | Updating file management job configuration |
| Microsoft Learn | Remove-FsrmFileManagementJob | Removing file management jobs during rollback |
| Microsoft Learn | Start-FsrmFileManagementJob | Running a file management job immediately or queued |
| Microsoft Learn | New-FsrmFmjAction | Creating expiration, RMS, or custom file management actions |
| Microsoft Learn | New-FsrmFmjCondition | Creating file management conditions based on file name, date, or classification properties |
| Microsoft Learn | Get-WinEvent | Validating FSRM and scheduled job events |
| Windows Server operational practice | Storage reporting | Finding large files, stale files, duplicate files, file owners, quota usage, and file groups |
| Windows Server operational practice | File lifecycle management | Moving, logging, or processing stale files based on age or policy |
| Windows Server operational practice | Safe file management | Start with reports and logs before expiring or moving user data |

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Mental_Model
| Concept | Operational Meaning |
|---|---|
| FSRM | File Server Resource Manager, used for quotas, file screens, storage reports, and file management jobs |
| Storage report | FSRM report job that analyzes storage usage in one or more folder namespaces |
| Interactive report | Report generated immediately without a long-term schedule |
| Scheduled report | Report job that runs on a defined weekly or monthly schedule |
| Namespace | Folder path or classification expression analyzed by an FSRM report or file management job |
| Report type | Report category such as Large Files, Duplicate Files, Files by Owner, Files by File Group, Least Recently Accessed, or Quota Usage |
| Report format | Output type such as HTML, CSV, XML, DHTML, or text depending on configuration |
| File management job | FSRM job that acts on files matching one or more conditions |
| Expiration action | File management action that moves matching files to an expiration folder |
| Custom action | File management action that runs a command against matching files |
| RMS action | File management action that applies rights management when AD RMS is configured |
| Condition | Rule that decides whether a file qualifies for a file management job |
| Date condition | Condition based on File.DateCreated, File.DateLastModified, or File.DateLastAccessed |
| File name condition | Condition based on file name pattern matching |
| Classification condition | Condition based on a file classification property |
| Scheduled task object | FSRM object that defines when a report or file management job runs |
| Stale data | Old or inactive files that may need review, archive, or expiration |
| Expiration folder | Controlled folder where expired files are moved rather than deleted |
| Report evidence | HTML or CSV output used to prove storage usage and cleanup candidates |
| First rule | Generate reports and validate candidates before allowing file management jobs to move, expire, encrypt, or process user data |

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Department root | `E:\Shares\Departments` | `<department-root>` |
| Home root | `E:\Shares\Home` | `<home-folder-root>` |
| Public root | `E:\Shares\Public` | `<public-share-root>` |
| Apps root | `E:\Shares\Apps` | `<app-share-root>` |
| Report namespace | `E:\Shares` | `<report-namespace>` |
| Department report namespace | `E:\Shares\Departments` | `<department-report-namespace>` |
| Home report namespace | `E:\Shares\Home` | `<home-report-namespace>` |
| Report output root | `C:\StorageReports` | `<report-output-root>` |
| Interactive report name | `Interactive_Large_Files_Report` | `<interactive-report-name>` |
| Scheduled report name | `Monthly_File_Server_Storage_Report` | `<scheduled-report-name>` |
| Large file minimum | `100 MB` | `<large-file-minimum>` |
| Stale file age | `180 days` | `<stale-file-age>` |
| Duplicate file report | Enabled | `<duplicate-file-report-plan>` |
| Files by owner report | Enabled | `<files-by-owner-report-plan>` |
| Files by file group report | Enabled | `<files-by-file-group-report-plan>` |
| Quota usage report | Enabled | `<quota-usage-report-plan>` |
| File management job name | `Expire_Stale_Public_Files` | `<file-management-job-name>` |
| File management namespace | `E:\Shares\Public` | `<file-management-namespace>` |
| Expiration folder | `E:\FSRM-ExpiredFiles` | `<expiration-folder>` |
| Job mode | Report-only, custom log, or expiration | `<file-management-job-mode>` |
| Schedule frequency | Weekly Sunday at 1 AM | `<schedule-plan>` |
| Run duration | `4` hours | `<run-duration-hours>` |
| Notification type | Event log first | `<notification-plan>` |
| Email notification | Optional after SMTP config | `<email-notification-plan>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Rollback stance | Remove scheduled jobs before deleting report output or expired files | `<rollback-plan>` |
| Next workbook | `11_Configure_Shadow_Copies_And_Previous_Versions.md` | `<next-task>` |

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm target paths exist | File Server | `Test-Path "<report-namespace>"`; `Test-Path "<file-management-namespace>"` | Report and job target paths exist |
| 4 | Install FSRM if needed | File Server | `Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools` | FSRM is installed |
| 5 | Confirm FSRM service | File Server | `Get-Service SrmSvc` | FSRM service is visible |
| 6 | Start FSRM service if needed | File Server | `Start-Service SrmSvc` | FSRM service is running |
| 7 | Confirm FSRM cmdlets | File Server | `Get-Command -Module FileServerResourceManager` | FSRM cmdlets are available |
| 8 | Create report output folder | File Server | `New-Item -ItemType Directory -Path "<report-output-root>" -Force` | Report output path exists |
| 9 | Create expiration folder | File Server | `New-Item -ItemType Directory -Path "<expiration-folder>" -Force` | Expired files staging path exists |
| 10 | Capture existing storage reports | File Server | `Get-FsrmStorageReport` | Existing report jobs are documented |
| 11 | Capture existing file management jobs | File Server | `Get-FsrmFileManagementJob` | Existing file management jobs are documented |
| 12 | Run interactive large files report | File Server | `New-FsrmStorageReport -Name "<interactive-report-name>" -Namespace @("<report-namespace>") -Interactive -ReportType @("LargeFiles") -LargeFileMinimum <size>` | Immediate report is generated |
| 13 | Run interactive multi-report | File Server | `New-FsrmStorageReport -Name "<interactive-report-name>" -Namespace @("<report-namespace>") -Interactive -ReportType @("LargeFiles","DuplicateFiles","FilesByOwner","QuotaUsage")` | Immediate storage report evidence is generated |
| 14 | Create scheduled task object | File Server | `$Schedule = New-FsrmScheduledTask -Time (Get-Date "1:00am") -Weekly Sunday -RunDuration 4` | FSRM schedule object is ready |
| 15 | Create monthly scheduled storage report | File Server | `New-FsrmStorageReport -Name "<scheduled-report-name>" -Namespace @("<report-namespace>") -Schedule $Schedule -ReportType @("LargeFiles","DuplicateFiles","QuotaUsage")` | Scheduled storage report exists |
| 16 | Validate scheduled report | File Server | `Get-FsrmStorageReport -Name "<scheduled-report-name>"` | Scheduled report settings match plan |
| 17 | Create file management condition | File Server | `$Condition = New-FsrmFmjCondition -Property "File.DateLastModified" -Condition LessThan -Value "Date.Now" -DateOffset -180` | Condition identifies stale files |
| 18 | Create file management action | File Server | `$Action = New-FsrmFmjAction -Type Expiration -ExpirationFolder "<expiration-folder>"` | Expiration action is ready |
| 19 | Create file management job | File Server | `New-FsrmFileManagementJob -Name "<file-management-job-name>" -Namespace @("<file-management-namespace>") -Schedule $Schedule -Action $Action -Condition $Condition` | File management job exists |
| 20 | Validate file management job | File Server | `Get-FsrmFileManagementJob -Name "<file-management-job-name>"` | Job settings match plan |
| 21 | Run job manually in lab if safe | File Server | `Start-FsrmFileManagementJob -Name "<file-management-job-name>" -RunDuration 1` | Job starts immediately |
| 22 | Queue job manually if preferred | File Server | `Start-FsrmFileManagementJob -Name "<file-management-job-name>" -Queue -RunDuration 1` | Job is queued |
| 23 | Validate report output | File Server | `Get-ChildItem "<report-output-root>" -Recurse` | Report files exist |
| 24 | Validate expiration folder | File Server | `Get-ChildItem "<expiration-folder>" -Recurse` | Expired files are visible if job moved files |
| 25 | Validate FSRM events | File Server | `Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"}` | FSRM report or job events are visible |
| 26 | Export final FSRM report and job state | File Server | Use post-change validation skeleton | Final evidence is saved |
| 27 | Document reporting and file management model | Operator | `Record report names, report types, schedules, namespaces, jobs, actions, conditions, and output paths` | Workbook record is complete |

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate FSRM, target paths, reports, file management jobs, and output locations before configuration.

$EvidencePath = "C:\FileServices-Validation"

$ReportNamespace = "E:\Shares"
$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$ReportOutputRoot = "C:\StorageReports"
$ExpirationFolder = "E:\FSRM-ExpiredFiles"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-fsrm-reports-file-management-precheck-transcript.txt"

# Confirm admin context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-fsrm-reports-jobs.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname-before-fsrm-reports-jobs.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-fsrm-reports-jobs.txt"

# Confirm File Server role and FSRM feature.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-fsrm-reports-jobs.txt"

Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-before-reports-jobs.txt"

Get-Service SrmSvc -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-service-before-reports-jobs.txt"

# Confirm target paths.
$TargetPaths = @(
  $ReportNamespace,
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot
)

foreach ($Path in $TargetPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\fsrm-report-job-target-path-validation-before.txt" -Append
}

# Create evidence and output folders.
New-Item -ItemType Directory -Force -Path $ReportOutputRoot |
  Tee-Object "$EvidencePath\report-output-root-create-result.txt"

New-Item -ItemType Directory -Force -Path $ExpirationFolder |
  Tee-Object "$EvidencePath\expiration-folder-create-result.txt"

# Capture current FSRM cmdlets and objects.
Get-Command -Module FileServerResourceManager -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-cmdlets-before-reports-jobs.txt"

Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-storage-reports-before.txt"

Get-FsrmFileManagementJob -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-management-jobs-before.txt"

Get-FsrmQuota -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quotas-before-reports-jobs.txt"

Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screens-before-reports-jobs.txt"

# Capture recent FSRM events.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddDays(-7)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-events-before-reports-jobs.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Install_And_Module_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: install FSRM and validate report and file management cmdlets.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-fsrm-install-module-validation-transcript.txt"

# Install FSRM if it is not already installed.
Install-WindowsFeature `
  -Name FS-Resource-Manager `
  -IncludeManagementTools

# Confirm feature and service state.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-after-report-job-install.txt"

Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-after-report-job-install.txt"

Start-Service SrmSvc -ErrorAction SilentlyContinue

Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-after-report-job-start.txt"

# Confirm module and relevant cmdlets.
Get-Module FileServerResourceManager -ListAvailable |
  Tee-Object "$EvidencePath\fsrm-module-available-for-reports-jobs.txt"

Import-Module FileServerResourceManager

Get-Command -Module FileServerResourceManager |
  Where-Object {
    $_.Name -like "*StorageReport*" -or
    $_.Name -like "*FileManagementJob*" -or
    $_.Name -like "*Fmj*" -or
    $_.Name -like "*ScheduledTask*"
  } |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-report-job-cmdlets.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Interactive_Storage_Report_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: generate immediate FSRM storage reports for quick analysis.

$EvidencePath = "C:\FileServices-Validation"

$ReportNamespace = "E:\Shares"
$ReportName = "Interactive_File_Server_Storage_Report"
$LargeFileMinimum = 100MB

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-interactive-storage-report-transcript.txt"

Import-Module FileServerResourceManager

# Confirm namespace.
Test-Path $ReportNamespace |
  Tee-Object "$EvidencePath\interactive-report-namespace-testpath.txt"

# Run an immediate multi-report.
New-FsrmStorageReport `
  -Name $ReportName `
  -Namespace @($ReportNamespace) `
  -Interactive `
  -ReportType @(
    "LargeFiles",
    "DuplicateFiles",
    "FilesByOwner",
    "QuotaUsage"
  ) `
  -LargeFileMinimum $LargeFileMinimum `
  -ReportFormat @(
    "Html",
    "Csv"
  )

# Capture report job list after interactive report creation.
Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-storage-reports-after-interactive.txt"

# Capture default report output locations if present.
Get-ChildItem "C:\StorageReports" -Recurse -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\storage-report-output-after-interactive.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Scheduled_Storage_Report_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create a scheduled FSRM storage report job.

$EvidencePath = "C:\FileServices-Validation"

$ReportNamespace = "E:\Shares"
$ScheduledReportName = "Monthly_File_Server_Storage_Report"
$LargeFileMinimum = 100MB

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-scheduled-storage-report-transcript.txt"

Import-Module FileServerResourceManager

# Create a monthly schedule.
# -1 means the last day of the month.
$ReportTime = Get-Date "1:00am"

$Schedule = New-FsrmScheduledTask `
  -Time $ReportTime `
  -Monthly @(-1) `
  -RunDuration 4

# Remove old lab report job with the same name if recreating cleanly.
$ExistingReport = Get-FsrmStorageReport -Name $ScheduledReportName -ErrorAction SilentlyContinue

if ($ExistingReport) {
  Remove-FsrmStorageReport `
    -Name $ScheduledReportName `
    -Confirm:$false
}

# Create scheduled storage report.
New-FsrmStorageReport `
  -Name $ScheduledReportName `
  -Namespace @($ReportNamespace) `
  -Schedule $Schedule `
  -ReportType @(
    "LargeFiles",
    "DuplicateFiles",
    "FilesByOwner",
    "FilesByFileGroup",
    "QuotaUsage"
  ) `
  -LargeFileMinimum $LargeFileMinimum `
  -ReportFormat @(
    "Html",
    "Csv"
  )

# Confirm scheduled report.
Get-FsrmStorageReport -Name $ScheduledReportName |
  Tee-Object "$EvidencePath\scheduled-storage-report-after-create.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Large_And_Duplicate_File_Report_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: generate focused reports for large files, duplicate files, stale files, and quota usage.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"

$LargeFileMinimum = 50MB

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-large-duplicate-stale-report-transcript.txt"

Import-Module FileServerResourceManager

# Immediate large files report for Departments.
New-FsrmStorageReport `
  -Name "Interactive_Department_Large_Files" `
  -Namespace @($DepartmentRoot) `
  -Interactive `
  -ReportType @("LargeFiles") `
  -LargeFileMinimum $LargeFileMinimum `
  -ReportFormat @("Html","Csv")

# Immediate duplicate files report for Departments.
New-FsrmStorageReport `
  -Name "Interactive_Department_Duplicate_Files" `
  -Namespace @($DepartmentRoot) `
  -Interactive `
  -ReportType @("DuplicateFiles") `
  -ReportFormat @("Html","Csv")

# Immediate least recently accessed files report for Home folders.
New-FsrmStorageReport `
  -Name "Interactive_Home_Least_Recently_Accessed" `
  -Namespace @($HomeRoot) `
  -Interactive `
  -ReportType @("LeastRecentlyAccessed") `
  -LeastAccessedMinimum 180 `
  -ReportFormat @("Html","Csv")

# Immediate quota usage report for all shares.
New-FsrmStorageReport `
  -Name "Interactive_File_Server_Quota_Usage" `
  -Namespace @("E:\Shares") `
  -Interactive `
  -ReportType @("QuotaUsage") `
  -QuotaMinimumUsage 50 `
  -ReportFormat @("Html","Csv")

# Capture report outputs.
Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-storage-reports-after-focused-reports.txt"

Get-ChildItem "C:\StorageReports" -Recurse -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\focused-storage-report-output-files.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_File_Management_Expiration_Job_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create a file management job that moves stale files to an expiration folder.
# Warning: test in a lab namespace first. This job can move user data.

$EvidencePath = "C:\FileServices-Validation"

$JobName = "Expire_Stale_Public_Files"
$Namespace = "E:\Shares\Public"
$ExpirationFolder = "E:\FSRM-ExpiredFiles"
$StaleDays = 180

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $ExpirationFolder

Start-Transcript -Path "$EvidencePath\10-file-management-expiration-job-transcript.txt"

Import-Module FileServerResourceManager

# Confirm namespace and expiration folder.
Test-Path $Namespace |
  Tee-Object "$EvidencePath\fmj-namespace-testpath.txt"

Test-Path $ExpirationFolder |
  Tee-Object "$EvidencePath\fmj-expiration-folder-testpath.txt"

# Create a monthly schedule.
$JobTime = Get-Date "2:00am"

$Schedule = New-FsrmScheduledTask `
  -Time $JobTime `
  -Monthly @(-1) `
  -RunDuration 4

# Create stale file condition.
# Select files whose last modified date is older than Date.Now minus $StaleDays.
$Condition = New-FsrmFmjCondition `
  -Property "File.DateLastModified" `
  -Condition LessThan `
  -Value "Date.Now" `
  -DateOffset (-1 * $StaleDays)

# Create expiration action.
$Action = New-FsrmFmjAction `
  -Type Expiration `
  -ExpirationFolder $ExpirationFolder

# Remove old lab job with same name if recreating cleanly.
$ExistingJob = Get-FsrmFileManagementJob -Name $JobName -ErrorAction SilentlyContinue

if ($ExistingJob) {
  Remove-FsrmFileManagementJob `
    -Name $JobName `
    -Confirm:$false
}

# Create file management job.
New-FsrmFileManagementJob `
  -Name $JobName `
  -Description "Moves files older than $StaleDays days from Public share to expiration folder." `
  -Namespace @($Namespace) `
  -Schedule $Schedule `
  -Action $Action `
  -Condition @($Condition) `
  -ReportFormat @("Html","Csv")

# Confirm job.
Get-FsrmFileManagementJob -Name $JobName |
  Tee-Object "$EvidencePath\file-management-expiration-job-after-create.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_File_Management_Custom_Log_Job_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create a safer file management job that logs matching files instead of moving them.
# Use this before using expiration jobs on production data.

$EvidencePath = "C:\FileServices-Validation"

$JobName = "Log_Stale_Public_Files"
$Namespace = "E:\Shares\Public"
$LogPath = "C:\FileServices-Validation\stale-public-files.log"
$StaleDays = 180

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-file-management-custom-log-job-transcript.txt"

Import-Module FileServerResourceManager

# Create a weekly schedule.
$JobTime = Get-Date "1:30am"

$Schedule = New-FsrmScheduledTask `
  -Time $JobTime `
  -Weekly Sunday `
  -RunDuration 4

# Create stale file condition.
$Condition = New-FsrmFmjCondition `
  -Property "File.DateLastModified" `
  -Condition LessThan `
  -Value "Date.Now" `
  -DateOffset (-1 * $StaleDays)

# Create a custom logging action.
# FSRM supports macros such as [source file path] in command parameters.
$Action = New-FsrmFmjAction `
  -Type Custom `
  -Command "C:\Windows\System32\cmd.exe" `
  -CommandParameters "/c echo [source file path] >> `"$LogPath`"" `
  -SecurityLevel LocalSystem

# Remove old lab job with same name if recreating cleanly.
$ExistingJob = Get-FsrmFileManagementJob -Name $JobName -ErrorAction SilentlyContinue

if ($ExistingJob) {
  Remove-FsrmFileManagementJob `
    -Name $JobName `
    -Confirm:$false
}

# Create custom log job.
New-FsrmFileManagementJob `
  -Name $JobName `
  -Description "Logs stale files older than $StaleDays days without moving them." `
  -Namespace @($Namespace) `
  -Schedule $Schedule `
  -Action $Action `
  -Condition @($Condition) `
  -ReportFormat @("Html","Csv")

# Confirm job.
Get-FsrmFileManagementJob -Name $JobName |
  Tee-Object "$EvidencePath\file-management-custom-log-job-after-create.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Manual_Job_Run_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: manually run or queue file management jobs in a lab.

$EvidencePath = "C:\FileServices-Validation"

$JobName = "Log_Stale_Public_Files"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-manual-file-management-job-run-transcript.txt"

Import-Module FileServerResourceManager

# Confirm job exists.
Get-FsrmFileManagementJob -Name $JobName |
  Tee-Object "$EvidencePath\manual-run-job-before-start.txt"

# Start job immediately with a one-hour run duration.
Start-FsrmFileManagementJob `
  -Name $JobName `
  -RunDuration 1

# Optional: queue instead of starting immediately.
# Start-FsrmFileManagementJob -Name $JobName -Queue -RunDuration 1

# Capture job state after start.
Get-FsrmFileManagementJob -Name $JobName |
  Tee-Object "$EvidencePath\manual-run-job-after-start.txt"

# Capture likely report and log output.
Get-ChildItem "C:\StorageReports" -Recurse -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\manual-run-storage-report-output.txt"

Get-ChildItem $EvidencePath -Filter "*.log" -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\manual-run-custom-log-output.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Event_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the file server after reports or file management jobs run.
# Purpose: validate FSRM events and scheduled job behavior.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-fsrm-report-job-event-validation-transcript.txt"

# FSRM events commonly appear in the Application log under SRMSVC.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-events-last-8-hours-reports-jobs.txt"

Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\fsrm-events-last-8-hours-reports-jobs.csv" -NoTypeInformation

# Capture System log entries related to service activity.
Get-WinEvent -LogName System -MaxEvents 100 |
  Where-Object {
    $_.ProviderName -like "*Service Control Manager*" -or
    $_.Message -like "*File Server Resource Manager*" -or
    $_.Message -like "*SrmSvc*"
  } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-system-events-reports-jobs.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Post_Change_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture final FSRM reports, file management jobs, schedules, output files, and events.

$EvidencePath = "C:\FileServices-Validation"

$ReportOutputRoot = "C:\StorageReports"
$ExpirationFolder = "E:\FSRM-ExpiredFiles"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\10-post-change-fsrm-reports-jobs-validation-transcript.txt"

Import-Module FileServerResourceManager

# Feature and service state.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-final-reports-jobs.txt"

Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-final-reports-jobs.txt"

# Relevant cmdlets.
Get-Command -Module FileServerResourceManager |
  Where-Object {
    $_.Name -like "*StorageReport*" -or
    $_.Name -like "*FileManagementJob*" -or
    $_.Name -like "*Fmj*" -or
    $_.Name -like "*ScheduledTask*"
  } |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-report-job-cmdlets-final.txt"

# Storage reports.
Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-storage-reports-final.txt"

Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Sort-Object Name |
  Select-Object Name,Namespace,ReportType,ReportFormat,MailTo |
  Export-Csv "$EvidencePath\fsrm-storage-reports-final.csv" -NoTypeInformation

# File management jobs.
Get-FsrmFileManagementJob -ErrorAction SilentlyContinue |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-file-management-jobs-final.txt"

Get-FsrmFileManagementJob -ErrorAction SilentlyContinue |
  Sort-Object Name |
  Select-Object Name,Namespace,Disabled,Continuous,ReportFormat,MailTo |
  Export-Csv "$EvidencePath\fsrm-file-management-jobs-final.csv" -NoTypeInformation

# Report output evidence.
Get-ChildItem $ReportOutputRoot -Recurse -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\storage-report-output-files-final.txt"

# Expiration folder evidence.
Get-ChildItem $ExpirationFolder -Recurse -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\expiration-folder-output-files-final.txt"

# FSRM related objects from previous workbooks.
Get-FsrmQuota -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quotas-final-reports-jobs.txt"

Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screens-final-reports-jobs.txt"

# Recent events.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-8)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-events-final-reports-jobs.txt"

Stop-Transcript
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Verification_Commands
```powershell
# Feature and service
Get-WindowsFeature FS-Resource-Manager
Get-Service SrmSvc
Start-Service SrmSvc

# Module and cmdlets
Get-Module FileServerResourceManager -ListAvailable
Import-Module FileServerResourceManager
Get-Command -Module FileServerResourceManager | Where-Object Name -like "*StorageReport*"
Get-Command -Module FileServerResourceManager | Where-Object Name -like "*FileManagementJob*"
Get-Command -Module FileServerResourceManager | Where-Object Name -like "*Fmj*"
Get-Command -Module FileServerResourceManager | Where-Object Name -like "*ScheduledTask*"

# Target paths
Test-Path "<report-namespace>"
Test-Path "<file-management-namespace>"
Test-Path "<report-output-root>"
Test-Path "<expiration-folder>"

# Storage reports
Get-FsrmStorageReport
Get-FsrmStorageReport -Name "<scheduled-report-name>"
New-FsrmStorageReport -Name "<interactive-report-name>" -Namespace @("<report-namespace>") -Interactive -ReportType @("LargeFiles") -LargeFileMinimum 100MB

# Scheduled task object
$Schedule = New-FsrmScheduledTask -Time (Get-Date "1:00am") -Weekly Sunday -RunDuration 4

# Scheduled report
New-FsrmStorageReport -Name "<scheduled-report-name>" -Namespace @("<report-namespace>") -Schedule $Schedule -ReportType @("LargeFiles","DuplicateFiles","QuotaUsage") -ReportFormat @("Html","Csv")

# File management job condition and action
$Condition = New-FsrmFmjCondition -Property "File.DateLastModified" -Condition LessThan -Value "Date.Now" -DateOffset -180
$Action = New-FsrmFmjAction -Type Expiration -ExpirationFolder "<expiration-folder>"

# File management jobs
Get-FsrmFileManagementJob
Get-FsrmFileManagementJob -Name "<file-management-job-name>"
New-FsrmFileManagementJob -Name "<file-management-job-name>" -Namespace @("<file-management-namespace>") -Schedule $Schedule -Action $Action -Condition @($Condition)

# Manual job run
Start-FsrmFileManagementJob -Name "<file-management-job-name>" -RunDuration 1
Start-FsrmFileManagementJob -Name "<file-management-job-name>" -Queue -RunDuration 1

# Report and expiration output
Get-ChildItem "C:\StorageReports" -Recurse
Get-ChildItem "<expiration-folder>" -Recurse

# Existing FSRM governance context
Get-FsrmQuota
Get-FsrmFileScreen
Get-FsrmFileGroup

# Events
Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"; StartTime=(Get-Date).AddHours(-8)}
Get-WinEvent -LogName System -MaxEvents 100

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*report*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*job*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*fsrm*"
```

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current report and job state | File Server | `Get-FsrmStorageReport`; `Get-FsrmFileManagementJob` | Current FSRM report and job state is saved |
| 2 | Stop active file management job if required | File Server | `Stop-FsrmFileManagementJob -Name "<file-management-job-name>"` | Running job stops |
| 3 | Remove scheduled storage report | File Server | `Remove-FsrmStorageReport -Name "<scheduled-report-name>" -Confirm:$false` | Scheduled report is removed |
| 4 | Remove file management job | File Server | `Remove-FsrmFileManagementJob -Name "<file-management-job-name>" -Confirm:$false` | File management job is removed |
| 5 | Disable file management job instead of removing | File Server | `Set-FsrmFileManagementJob -Name "<file-management-job-name>" -Disabled $true` | Job remains but does not run |
| 6 | Preserve report output before cleanup | File Server | `Copy-Item "<report-output-root>" "<evidence-path>\ReportOutput-Backup" -Recurse -Force` | Report output is backed up |
| 7 | Remove lab report output if needed | File Server | `Remove-Item "<report-output-root>\*" -Recurse -Force` | Lab report files are removed |
| 8 | Preserve expiration folder before cleanup | File Server | `Copy-Item "<expiration-folder>" "<evidence-path>\ExpiredFiles-Backup" -Recurse -Force` | Expired files are backed up |
| 9 | Restore expired files if required | File Server | `Move-Item "<expiration-folder>\*" "<original-path>"` | Files are returned to original location if known |
| 10 | Leave FSRM installed if quotas or file screens remain | File Server | `Get-FsrmQuota`; `Get-FsrmFileScreen` | FSRM remains available for other governance functions |
| 11 | Uninstall FSRM only if abandoning all FSRM labs | File Server | `Uninstall-WindowsFeature FS-Resource-Manager` | FSRM role service is removed |
| 12 | Validate rollback | File Server | `Get-FsrmStorageReport`; `Get-FsrmFileManagementJob` | Reports and jobs match rollback target |
| 13 | Document rollback | Operator | `Record removed reports, removed jobs, preserved output, and restored files` | Rollback record is complete |

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| `Get-FsrmStorageReport` is not recognized | FSRM module missing or not imported | `Get-WindowsFeature FS-Resource-Manager`; `Get-Module FileServerResourceManager -ListAvailable` | Install FSRM and import module |
| Storage report creation fails | Namespace path does not exist | `Test-Path "<report-namespace>"` | Correct or create namespace path |
| Scheduled report fails | Invalid schedule object | `$Schedule`; `Get-Command New-FsrmScheduledTask` | Recreate schedule object |
| Report output not found | Report still running or output path not checked | `Get-ChildItem C:\StorageReports -Recurse` | Wait for report completion and check default report folder |
| Large file report empty | Minimum size too high | `Get-ChildItem "<path>" -Recurse | Sort Length -Descending` | Lower `LargeFileMinimum` |
| Duplicate file report empty | No duplicates exist or namespace too small | Review target namespace | Test with controlled duplicate files |
| Quota usage report empty | No FSRM quotas exist | `Get-FsrmQuota` | Configure quotas in task 09 |
| File management job creation fails | Missing action object | `$Action`; `New-FsrmFmjAction` | Create action before creating job |
| File management job creation fails | Missing schedule object | `$Schedule`; `New-FsrmScheduledTask` | Create schedule before creating job |
| File management job creation fails | Invalid condition object | `$Condition`; `New-FsrmFmjCondition` | Recreate condition with supported property and condition |
| File management job moves too many files | Condition too broad or namespace too high-level | `Get-FsrmFileManagementJob -Name "<job>"` | Disable job, restore files, narrow namespace or condition |
| Expiration folder empty | No files met condition | Job report and condition settings | Create lab stale file or adjust age threshold |
| Custom log job output empty | Command parameters or macro path wrong | Job settings and log path | Correct `CommandParameters` and rerun job |
| Job does not start immediately | Queued job waits or another job is running | `Start-FsrmFileManagementJob -Queue` behavior | Run without `-Queue` in lab or wait |
| Job stops early | RunDuration too low | Job event logs and schedule | Increase RunDuration |
| Email notifications fail | SMTP not configured | FSRM email settings | Use event/log notifications first |
| Report generation impacts server | Large namespace or many report types | CPU, disk, and job runtime | Run off-hours, narrow namespace, or limit report types |
| Report or job overwritten by duplicate name | Same report/job name reused | `Get-FsrmStorageReport`; `Get-FsrmFileManagementJob` | Use unique names or remove old lab objects |
| Event evidence missing | Wrong provider or time window | Application log query | Query `SRMSVC` over a larger time range |
| Evidence missing | Validation skeleton not run | `Test-Path "<evidence-path>"` | Re-run precheck or post-change validation skeleton |

# Configure_FSRM_Storage_Reports_And_File_Management_Tasks_Related_Labs
| Related Lab                                                     | Relationship                                                                             |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `00_File_Services_Index.md`                                     | Defines where FSRM reporting and file management fits in the File Services suite         |
| `01_Install_File_Server_Role_And_Management_Tools.md`           | Provides File Server role and management baseline                                        |
| `02_Prepare_File_Server_Storage_Volumes_And_Folders.md`         | Creates the folder paths analyzed by reports and jobs                                    |
| `03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`          | Controls who can create data that later appears in reports or jobs                       |
| `04_Create_SMB_Shares_And_Share_Permissions.md`                 | Publishes folders whose storage usage is analyzed                                        |
| `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md` | Controls user visibility while reports analyze actual storage                            |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`  | Provides audit evidence around file access and job impact                                |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`       | Provides department and home folder targets for reports and lifecycle jobs               |
| `08_Configure_File_Screening_With_FSRM.md`                      | Uses FSRM file groups and screens that reports can analyze                               |
| `09_Configure_Storage_Quotas_With_FSRM.md`                      | Provides quota usage data for FSRM quota reports                                         |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md`           | Adds recovery capability before aggressive file management actions                       |
| `12_Backup_Restore_And_Export_File_Server_Config.md`            | Protects FSRM configuration, report jobs, and file management job definitions            |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`  | Provides operational context around users and active files                               |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`         | Helps distinguish FSRM movement, quota, screening, SMB, and NTFS issues                  |
| `25_Migrate_File_Server_Data_Shares_And_Permissions.md`         | Requires exporting and recreating FSRM reports and file management jobs during migration |