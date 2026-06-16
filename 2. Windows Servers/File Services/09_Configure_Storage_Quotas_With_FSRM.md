


09_Configure_Storage_Quotas_With_FSRM.md
# Configure_Storage_Quotas_With_FSRM

# Configure_Storage_Quotas_With_FSRM_Index
09_Configure_Storage_Quotas_With_FSRM.md
Configure_Storage_Quotas_With_FSRM
Configure_Storage_Quotas_With_FSRM_Source_Basis
Configure_Storage_Quotas_With_FSRM_Mental_Model
Configure_Storage_Quotas_With_FSRM_Planning_Table
Configure_Storage_Quotas_With_FSRM_Configuration_Checklist
Configure_Storage_Quotas_With_FSRM_Precheck_Skeleton
Configure_Storage_Quotas_With_FSRM_Install_And_Module_Validation_Skeleton
Configure_Storage_Quotas_With_FSRM_Quota_Template_Skeleton
Configure_Storage_Quotas_With_FSRM_Department_Quota_Skeleton
Configure_Storage_Quotas_With_FSRM_Home_Auto_Quota_Skeleton
Configure_Storage_Quotas_With_FSRM_Soft_Quota_Monitoring_Skeleton
Configure_Storage_Quotas_With_FSRM_Client_Quota_Test_Skeleton
Configure_Storage_Quotas_With_FSRM_Event_Validation_Skeleton
Configure_Storage_Quotas_With_FSRM_Post_Change_Validation_Skeleton
Configure_Storage_Quotas_With_FSRM_Verification_Commands
Configure_Storage_Quotas_With_FSRM_Rollback
Configure_Storage_Quotas_With_FSRM_Failure_Checks
Configure_Storage_Quotas_With_FSRM_Related_Labs

# Configure_Storage_Quotas_With_FSRM_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | File Server Resource Manager | Managing quotas, file screens, storage reports, and file management tasks |
| Microsoft Learn | Install-WindowsFeature | Installing the FSRM role service and management tools |
| Microsoft Learn | Get-FsrmQuotaTemplate | Reviewing quota templates |
| Microsoft Learn | New-FsrmQuotaTemplate | Creating reusable hard or soft quota templates |
| Microsoft Learn | New-FsrmQuotaThreshold | Creating quota threshold levels |
| Microsoft Learn | New-FsrmAction | Creating event, email, command, or report actions for quota thresholds |
| Microsoft Learn | New-FsrmQuota | Applying a quota to a specific folder path |
| Microsoft Learn | New-FsrmAutoQuota | Applying quota templates automatically to child folders |
| Microsoft Learn | Get-FsrmQuota | Validating folder quota configuration |
| Microsoft Learn | Get-FsrmAutoQuota | Validating auto-apply quota configuration |
| Microsoft Learn | Remove-FsrmQuota | Removing folder quotas during rollback |
| Microsoft Learn | Remove-FsrmAutoQuota | Removing auto-apply quotas during rollback |
| Windows Server operational practice | Hard versus soft quotas | Hard quotas enforce a storage limit. Soft quotas monitor usage without blocking writes |
| Windows Server operational practice | Home folder quotas | Auto-apply quotas are useful when many user folders exist under one home root |
| Windows Server operational practice | Department quota governance | Department quotas prevent uncontrolled growth and provide threshold evidence |

# Configure_Storage_Quotas_With_FSRM_Mental_Model
| Concept | Operational Meaning |
|---|---|
| FSRM | File Server Resource Manager, the Windows Server role service used for quotas, file screens, reports, and file tasks |
| Quota | Storage limit applied to a folder path |
| Hard quota | Enforced quota that blocks users from saving more data after the limit is reached |
| Soft quota | Monitoring quota that records threshold events but does not block writes |
| Quota template | Reusable quota definition containing size, hard/soft behavior, thresholds, and notifications |
| Threshold | Usage percentage that triggers an action, such as 85 percent or 95 percent |
| Notification action | Event log, email, command, or report action triggered by a threshold |
| Department quota | Quota applied to a department folder or department root |
| Home folder quota | Quota applied to each user's home folder |
| Auto-apply quota | FSRM policy that applies a quota template to existing and future child folders |
| Derived quota | Quota instance created from an auto-apply quota for a child folder |
| Path-based quota | Quota tied to a folder path such as `E:\Shares\Departments\Accounting` |
| Volume-level free space | Remaining capacity on the underlying volume, separate from FSRM quota limits |
| Quota usage | How much of the quota limit is currently consumed |
| Quota enforcement | Blocking behavior when hard quota limit is reached |
| Quota monitoring | Visibility behavior when soft quota threshold is crossed |
| User experience | Hard quota usually appears as an access denied, insufficient space, or write failure |
| Event evidence | FSRM Application log entries proving threshold or quota behavior occurred |
| First rule | Use soft quotas to observe behavior before enforcing hard quotas on broad production paths |

# Configure_Storage_Quotas_With_FSRM_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Data volume | `E:` | `<data-drive-letter>` |
| Department root | `E:\Shares\Departments` | `<department-root>` |
| Home root | `E:\Shares\Home` | `<home-folder-root>` |
| Public root | `E:\Shares\Public` | `<public-share-root>` |
| Sample department path | `E:\Shares\Departments\Accounting` | `<department-quota-path>` |
| Sample home folder path | `E:\Shares\Home\jsmith` | `<sample-home-folder-path>` |
| Department share UNC | `\\FS1\Departments` | `<department-unc>` |
| Home share UNC | `\\FS1\Home$` | `<home-unc-root>` |
| Department hard quota template | `Department_100GB_Hard_Quota` | `<department-hard-template-name>` |
| Department soft quota template | `Department_100GB_Soft_Monitor` | `<department-soft-template-name>` |
| Home hard quota template | `Home_5GB_Hard_Quota` | `<home-hard-template-name>` |
| Home soft quota template | `Home_5GB_Soft_Monitor` | `<home-soft-template-name>` |
| Department quota size | `100 GB` | `<department-quota-size>` |
| Home folder quota size | `5 GB` | `<home-quota-size>` |
| Public quota size | `20 GB` | `<public-quota-size>` |
| Quota mode for department | Hard or Soft | `<department-quota-mode>` |
| Quota mode for home folders | Hard or Soft | `<home-quota-mode>` |
| Auto-apply home quota | Enabled | `<home-auto-quota-plan>` |
| Threshold 1 | `85%` | `<threshold-1>` |
| Threshold 2 | `95%` | `<threshold-2>` |
| Threshold 3 | `100%` | `<threshold-3>` |
| Notification type | Event log first | `<notification-plan>` |
| SMTP notification | Optional after SMTP is configured | `<email-notification-plan>` |
| Test file path | `\\FS1\Home$\jsmith\quota-test.bin` | `<quota-test-file>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Client evidence path | `C:\FileServices-Validation-Client` | `<client-evidence-path>` |
| Rollback stance | Remove quotas or switch hard quotas to soft monitoring | `<rollback-plan>` |
| Next workbook | `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | `<next-task>` |

# Configure_Storage_Quotas_With_FSRM_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm target paths exist | File Server | `Test-Path "<department-quota-path>"`; `Test-Path "<home-folder-root>"` | Quota target paths exist |
| 4 | Install FSRM role service if needed | File Server | `Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools` | FSRM is installed |
| 5 | Confirm FSRM service | File Server | `Get-Service SrmSvc` | FSRM service is visible |
| 6 | Start FSRM service if needed | File Server | `Start-Service SrmSvc` | FSRM service is running |
| 7 | Confirm FSRM PowerShell cmdlets | File Server | `Get-Command -Module FileServerResourceManager` | FSRM cmdlets are available |
| 8 | Capture existing quota state | File Server | `Get-FsrmQuota`; `Get-FsrmAutoQuota`; `Get-FsrmQuotaTemplate` | Current FSRM quota state is documented |
| 9 | Create quota threshold actions | File Server | `New-FsrmAction -Type Event -EventType Warning -Body "<message>"` | Threshold action objects are ready |
| 10 | Create hard quota template for departments | File Server | `New-FsrmQuotaTemplate -Name "<department-hard-template-name>" -Size <size> -Threshold <thresholds>` | Hard quota template exists |
| 11 | Create soft quota template for monitoring | File Server | `New-FsrmQuotaTemplate -Name "<department-soft-template-name>" -Size <size> -SoftLimit -Threshold <thresholds>` | Soft quota template exists |
| 12 | Create home folder quota template | File Server | `New-FsrmQuotaTemplate -Name "<home-hard-template-name>" -Size <size> -Threshold <thresholds>` | Home quota template exists |
| 13 | Apply quota to sample department path | File Server | `New-FsrmQuota -Path "<department-quota-path>" -Template "<template-name>"` | Department quota exists |
| 14 | Apply auto quota to home root | File Server | `New-FsrmAutoQuota -Path "<home-folder-root>" -Template "<home-template-name>"` | Auto quota applies to home child folders |
| 15 | Confirm derived home quotas | File Server | `Get-FsrmQuota \| Where-Object Path -like "<home-folder-root>*"` | Per-user home quotas are visible |
| 16 | Apply public share quota if planned | File Server | `New-FsrmQuota -Path "<public-share-root>" -Template "<template-name>"` | Public quota exists if planned |
| 17 | Validate quota templates | File Server | `Get-FsrmQuotaTemplate` | Templates match planned size and soft/hard stance |
| 18 | Validate quotas | File Server | `Get-FsrmQuota` | Folder quotas match planned paths |
| 19 | Validate auto quotas | File Server | `Get-FsrmAutoQuota` | Auto quotas match planned paths |
| 20 | Test allowed write below quota | Client | Create a small test file in quota path | Small file write succeeds |
| 21 | Test hard quota behavior in lab path | Client | Create files until quota limit is reached | Hard quota blocks additional writes |
| 22 | Test soft quota behavior in lab path | Client | Create files past soft quota threshold | Write succeeds and event is generated |
| 23 | Validate FSRM quota events | File Server | `Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"}` | Quota threshold events are visible |
| 24 | Export final quota state | File Server | Use post-change validation skeleton | Quota evidence is saved |
| 25 | Document quota model | Operator | `Record paths, templates, sizes, thresholds, mode, and test results` | FSRM quota record is complete |

# Configure_Storage_Quotas_With_FSRM_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate File Server, target paths, SMB shares, FSRM role, and current quota state before quota changes.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$DepartmentQuotaPath = "E:\Shares\Departments\Accounting"
$SampleHomePath = "E:\Shares\Home\jsmith"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\09-fsrm-quota-precheck-transcript.txt"

# Confirm admin context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-fsrm-quotas.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname-before-fsrm-quotas.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-fsrm-quotas.txt"

# Confirm File Server role.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-fsrm-quotas.txt"

# Confirm storage capacity.
Get-Volume |
  Tee-Object "$EvidencePath\volumes-before-fsrm-quotas.txt"

Get-Volume -DriveLetter E -ErrorAction SilentlyContinue |
  Select-Object DriveLetter,FileSystemLabel,FileSystem,HealthStatus,Size,SizeRemaining |
  Tee-Object "$EvidencePath\data-volume-capacity-before-fsrm-quotas.txt"

# Confirm target paths.
$TargetPaths = @(
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $DepartmentQuotaPath,
  $SampleHomePath
)

foreach ($Path in $TargetPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\quota-target-path-validation-before.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-fsrm-quota-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Confirm SMB shares because quota testing usually happens over SMB.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-before-fsrm-quotas.txt"

# Capture FSRM state if available.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-before-quotas.txt"

Get-Service SrmSvc -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-service-before-quotas.txt"

Get-Command -Module FileServerResourceManager -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-cmdlets-before-quotas.txt"

Get-FsrmQuotaTemplate -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quota-templates-before.txt"

Get-FsrmQuota -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quotas-before.txt"

Get-FsrmAutoQuota -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-auto-quotas-before.txt"

Stop-Transcript


# Configure_Storage_Quotas_With_FSRM_Install_And_Module_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: install FSRM and validate the quota cmdlets.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\09-fsrm-install-module-validation-transcript.txt"

# Install FSRM if it is not already installed.
Install-WindowsFeature `
  -Name FS-Resource-Manager `
  -IncludeManagementTools

# Confirm feature.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-after-quota-install.txt"

# Confirm service.
Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-after-quota-install.txt"

Start-Service SrmSvc -ErrorAction SilentlyContinue

Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-after-quota-start.txt"

# Confirm module.
Get-Module FileServerResourceManager -ListAvailable |
  Tee-Object "$EvidencePath\fsrm-module-available-for-quotas.txt"

Import-Module FileServerResourceManager

# Confirm quota-related cmdlets.
Get-Command -Module FileServerResourceManager |
  Where-Object {
    $_.Name -like "*Quota*" -or
    $_.Name -like "*FsrmAction*"
  } |
  Tee-Object "$EvidencePath\fsrm-quota-cmdlets.txt"

Stop-Transcript
```

# Configure_Storage_Quotas_With_FSRM_Quota_Template_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: apply an FSRM quota to a department folder.
# Use soft quota first for monitoring, then switch to hard quota when ready.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentQuotaPath = "E:\Shares\Departments\Accounting"
$DepartmentTemplateName = "Department_100GB_Hard_Quota"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $DepartmentQuotaPath

Start-Transcript -Path "$EvidencePath\09-department-quota-transcript.txt"

Import-Module FileServerResourceManager

# Confirm path and template.
Test-Path $DepartmentQuotaPath |
  Tee-Object "$EvidencePath\department-quota-path-testpath.txt"

Get-FsrmQuotaTemplate -Name $DepartmentTemplateName |
  Tee-Object "$EvidencePath\department-quota-template-before-apply.txt"

# Capture existing quota on path.
Get-FsrmQuota -Path $DepartmentQuotaPath -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\department-quota-before-apply.txt"

# Create or update quota.
$ExistingQuota = Get-FsrmQuota -Path $DepartmentQuotaPath -ErrorAction SilentlyContinue

if (-not $ExistingQuota) {
  New-FsrmQuota `
    -Path $DepartmentQuotaPath `
    -Template $DepartmentTemplateName
}
else {
  Set-FsrmQuota `
    -Path $DepartmentQuotaPath `
    -Template $DepartmentTemplateName
}

# Confirm quota.
Get-FsrmQuota -Path $DepartmentQuotaPath |
  Tee-Object "$EvidencePath\department-quota-after-apply.txt"

Stop-Transcript
```



# Configure_Storage_Quotas_With_FSRM_Home_Auto_Quota_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: apply an auto quota to the Home root so each user home folder gets a quota.

$EvidencePath = "C:\FileServices-Validation"

$HomeRoot = "E:\Shares\Home"
$HomeTemplateName = "Home_5GB_Hard_Quota"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $HomeRoot

Start-Transcript -Path "$EvidencePath\09-home-auto-quota-transcript.txt"

Import-Module FileServerResourceManager

# Confirm path and template.
Test-Path $HomeRoot |
  Tee-Object "$EvidencePath\home-root-testpath-before-auto-quota.txt"

Get-FsrmQuotaTemplate -Name $HomeTemplateName |
  Tee-Object "$EvidencePath\home-quota-template-before-auto-quota.txt"

# Capture existing auto quota.
Get-FsrmAutoQuota -Path $HomeRoot -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\home-auto-quota-before.txt"

# Create or update auto quota on Home root.
$ExistingAutoQuota = Get-FsrmAutoQuota -Path $HomeRoot -ErrorAction SilentlyContinue

if (-not $ExistingAutoQuota) {
  New-FsrmAutoQuota `
    -Path $HomeRoot `
    -Template $HomeTemplateName
}
else {
  Set-FsrmAutoQuota `
    -Path $HomeRoot `
    -Template $HomeTemplateName
}

# Confirm auto quota.
Get-FsrmAutoQuota -Path $HomeRoot |
  Tee-Object "$EvidencePath\home-auto-quota-after.txt"

# Confirm derived quotas under Home root.
Get-FsrmQuota |
  Where-Object { $_.Path -like "$HomeRoot*" } |
  Sort-Object Path |
  Tee-Object "$EvidencePath\home-derived-quotas-after.txt"

Stop-Transcript
```



```
# Run in elevated PowerShell on the file server.
# Purpose: apply a soft quota for monitoring before enforcing a hard quota.

$EvidencePath = "C:\FileServices-Validation"

$MonitorPath = "E:\Shares\Public"
$SoftTemplateName = "Department_100GB_Soft_Monitor"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $MonitorPath

Start-Transcript -Path "$EvidencePath\09-soft-quota-monitoring-transcript.txt"

Import-Module FileServerResourceManager

# Confirm template and path.
Get-FsrmQuotaTemplate -Name $SoftTemplateName |
  Tee-Object "$EvidencePath\soft-quota-template-before-apply.txt"

Test-Path $MonitorPath |
  Tee-Object "$EvidencePath\soft-quota-monitor-path-testpath.txt"

# Apply soft quota.
$ExistingQuota = Get-FsrmQuota -Path $MonitorPath -ErrorAction SilentlyContinue

if (-not $ExistingQuota) {
  New-FsrmQuota `
    -Path $MonitorPath `
    -Template $SoftTemplateName
}
else {
  Set-FsrmQuota `
    -Path $MonitorPath `
    -Template $SoftTemplateName
}

# Confirm soft quota.
Get-FsrmQuota -Path $MonitorPath |
  Tee-Object "$EvidencePath\soft-quota-after-apply.txt"

Stop-Transcript
```


# Configure_Storage_Quotas_With_FSRM_Client_Quota_Test_Skeleton
```
# Run in elevated PowerShell on the file server after quota tests.
# Purpose: validate FSRM quota threshold and quota limit events.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\09-fsrm-quota-event-validation-transcript.txt"

# FSRM quota events commonly appear in the Application log under SRMSVC.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-4)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-quota-application-events-last-4-hours.txt"

Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-4)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\fsrm-quota-application-events-last-4-hours.csv" -NoTypeInformation

# Capture related System service events.
Get-WinEvent -LogName System -MaxEvents 100 |
  Where-Object {
    $_.ProviderName -like "*Service Control Manager*" -or
    $_.Message -like "*File Server Resource Manager*" -or
    $_.Message -like "*SrmSvc*"
  } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-quota-system-events-last-100.txt"

Stop-Transcript
```








Configure_Storage_Quotas_With_FSRM_Post_Change_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: capture final FSRM quota state, templates, auto quotas, quota usage, and event evidence.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentQuotaPath = "E:\Shares\Departments\Accounting"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\09-post-change-fsrm-quota-validation-transcript.txt"

Import-Module FileServerResourceManager

# Feature, service, and cmdlet state.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-final-quotas.txt"

Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-final-quotas.txt"

Get-Command -Module FileServerResourceManager |
  Where-Object {
    $_.Name -like "*Quota*" -or
    $_.Name -like "*FsrmAction*"
  } |
  Tee-Object "$EvidencePath\fsrm-quota-cmdlets-final.txt"

# Quota templates.
Get-FsrmQuotaTemplate |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-quota-templates-final.txt"

# Folder quotas.
Get-FsrmQuota |
  Sort-Object Path |
  Tee-Object "$EvidencePath\fsrm-quotas-final.txt"

Get-FsrmQuota |
  Sort-Object Path |
  Select-Object Path,Template,Size,PeakUsage,Usage,SoftLimit,Disabled |
  Export-Csv "$EvidencePath\fsrm-quotas-final.csv" -NoTypeInformation

# Auto quotas.
Get-FsrmAutoQuota -ErrorAction SilentlyContinue |
  Sort-Object Path |
  Tee-Object "$EvidencePath\fsrm-auto-quotas-final.txt"

Get-FsrmAutoQuota -ErrorAction SilentlyContinue |
  Sort-Object Path |
  Select-Object Path,Template,Size,SoftLimit,Disabled |
  Export-Csv "$EvidencePath\fsrm-auto-quotas-final.csv" -NoTypeInformation

# Specific target validation.
Get-FsrmQuota -Path $DepartmentQuotaPath -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\department-quota-final.txt"

Get-FsrmAutoQuota -Path $HomeRoot -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\home-auto-quota-final.txt"

Get-FsrmQuota -Path $PublicRoot -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\public-quota-final.txt"

# SMB and path evidence.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-final-fsrm-quotas.txt"

foreach ($Path in @($DepartmentQuotaPath,$HomeRoot,$PublicRoot)) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\quota-paths-final.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-final-fsrm-quota-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Recent FSRM events.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-4)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-quota-events-final.txt"

Stop-Transcript
```



Configure_Storage_Quotas_With_FSRM_Verification_Commands
```
# Feature and service
Get-WindowsFeature FS-Resource-Manager
Get-Service SrmSvc
Start-Service SrmSvc

# FSRM module and quota cmdlets
Get-Module FileServerResourceManager -ListAvailable
Import-Module FileServerResourceManager
Get-Command -Module FileServerResourceManager | Where-Object Name -like "*Quota*"

# Quota templates
Get-FsrmQuotaTemplate
Get-FsrmQuotaTemplate -Name "<template-name>"

# Quotas
Get-FsrmQuota
Get-FsrmQuota -Path "<quota-path>"
New-FsrmQuota -Path "<quota-path>" -Template "<template-name>"
Set-FsrmQuota -Path "<quota-path>" -Template "<template-name>"
Remove-FsrmQuota -Path "<quota-path>" -Confirm:$false

# Auto quotas
Get-FsrmAutoQuota
Get-FsrmAutoQuota -Path "<home-folder-root>"
New-FsrmAutoQuota -Path "<home-folder-root>" -Template "<home-template-name>"
Set-FsrmAutoQuota -Path "<home-folder-root>" -Template "<home-template-name>"
Remove-FsrmAutoQuota -Path "<home-folder-root>" -Confirm:$false

# Target path and SMB validation
Test-Path "<quota-path>"
icacls "<quota-path>"
Get-SmbShare
Get-SmbShare -Name "<share-name>"
Get-SmbShareAccess -Name "<share-name>"

# Storage capacity
Get-Volume
Get-Volume -DriveLetter <data-drive-letter>

# Client path test
Test-NetConnection "<file-server-name>" -Port 445
Test-Path "\\<file-server-name>\<share-name>\<folder>"

# Client quota write test
fsutil file createnew "\\<file-server-name>\<share-name>\<folder>\quota-test.bin" 104857600
Remove-Item "\\<file-server-name>\<share-name>\<folder>\quota-test.bin" -Force

# Events
Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"; StartTime=(Get-Date).AddHours(-4)}
Get-WinEvent -LogName System -MaxEvents 100

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*quota*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*fsrm*"
```




# Configure_Storage_Quotas_With_FSRM_Rollback

|Step|Task|Device|PowerShell / Command|Expected Result|
|---|---|---|---|---|
|1|Capture current quota state before rollback|File Server|`Get-FsrmQuota`; `Get-FsrmAutoQuota`; `Get-FsrmQuotaTemplate`|Current quota state is saved|
|2|Remove test files|File Server|`Remove-Item "<path>\quota-test.bin" -Force -ErrorAction SilentlyContinue`|Test data is removed|
|3|Remove quota from department path|File Server|`Remove-FsrmQuota -Path "<department-quota-path>" -Confirm:$false`|Department quota is removed|
|4|Remove quota from public path|File Server|`Remove-FsrmQuota -Path "<public-share-root>" -Confirm:$false`|Public quota is removed|
|5|Remove auto quota from home root|File Server|`Remove-FsrmAutoQuota -Path "<home-folder-root>" -Confirm:$false`|Home auto quota is removed|
|6|Remove derived home quotas if needed|File Server|`Get-FsrmQuota \| Where-Object Path -like "<home-folder-root>*" \| Remove-FsrmQuota -Confirm:$false`|Derived child quotas are removed|
|7|Change hard quota to soft monitoring instead of removing|File Server|`Set-FsrmQuota -Path "<quota-path>" -Template "<soft-template-name>"`|Quota remains but no longer blocks writes|
|8|Remove custom quota template if no longer used|File Server|`Remove-FsrmQuotaTemplate -Name "<template-name>" -Confirm:$false`|Template is removed|
|9|Leave FSRM installed if reports and file tasks are next|File Server|`Get-WindowsFeature FS-Resource-Manager`|FSRM remains installed|
|10|Uninstall FSRM only if abandoning FSRM labs|File Server|`Uninstall-WindowsFeature FS-Resource-Manager`|FSRM role service is removed|
|11|Validate rollback state|File Server|`Get-FsrmQuota`; `Get-FsrmAutoQuota`; `Get-FsrmQuotaTemplate`|Quota state matches rollback target|
|12|Retest file write after rollback|Client|Create test file in affected path|Write behavior matches rollback target|
|13|Document rollback result|Operator|`Record removed quotas, templates, auto quotas, and test results`|Rollback record is complete|




# Configure_Storage_Quotas_With_FSRM_Failure_Checks

|Symptom|Likely Cause|Check|Corrective Action|
|---|---|---|---|
|`Get-FsrmQuota` is not recognized|FSRM module not installed or imported|`Get-WindowsFeature FS-Resource-Manager`; `Get-Module FileServerResourceManager -ListAvailable`|Install FSRM and import module|
|FSRM service missing|FS-Resource-Manager not installed|`Get-WindowsFeature FS-Resource-Manager`|Install `FS-Resource-Manager`|
|FSRM service stopped|Service did not start or failed|`Get-Service SrmSvc`|Start service and review System/Application logs|
|Quota template creation fails|Invalid size, duplicate name, or bad threshold object|`Get-FsrmQuotaTemplate`; review error|Correct template name, size, or threshold input|
|Quota creation fails|Target path does not exist|`Test-Path "<quota-path>"`|Create or correct folder path|
|Quota creation fails|Quota already exists on path|`Get-FsrmQuota -Path "<quota-path>"`|Use `Set-FsrmQuota` or remove old quota|
|Auto quota does not create child quotas|No child folders exist or auto quota not applied to correct root|`Get-ChildItem "<home-root>" -Directory`; `Get-FsrmAutoQuota`|Create child folders and verify auto quota path|
|User can exceed expected quota|Soft quota used instead of hard quota|`Get-FsrmQuota -Path "<path>"`|Switch to hard quota template|
|User is blocked unexpectedly|Hard quota applied to wrong path or size too small|`Get-FsrmQuota`; `Get-Volume`|Adjust path, template, or quota size|
|User gets generic write failure|Hard quota limit reached|FSRM events and `Get-FsrmQuota`|Confirm quota status and communicate limit|
|Quota events do not appear|Threshold not crossed or notification missing|`Get-FsrmQuotaTemplate`; Application log|Add event notification and retest|
|Email alerts do not send|SMTP not configured|FSRM email settings|Configure SMTP before relying on email actions|
|Test file creation fails before quota is reached|NTFS or share permission issue|`icacls`; `Get-SmbShareAccess`; `whoami /groups`|Fix access permissions first|
|Quota path is correct but client tests wrong path|UNC maps to different folder or DFS target|`Get-SmbShare`; `Resolve-DnsName`; DFS checks|Test the exact path where quota is applied|
|Derived home quotas missing for new users|Auto quota not refreshing or folder created outside root|`Get-FsrmAutoQuota`; `Get-FsrmQuota`|Create folder under auto quota root and refresh validation|
|Quota size too large for lab test|Hard to generate enough data|`Get-FsrmQuotaTemplate`|Create a temporary small lab quota template|
|Quota cleanup leaves test files|Test file was created before quota blocked|`Get-ChildItem "<path>" -Filter "quota-test*"`|Remove test files|
|Evidence missing|Validation skeleton not run|`Test-Path "<evidence-path>"`|Re-run precheck or post-change validation skeleton|



# Configure_Storage_Quotas_With_FSRM_Related_Labs

|Related Lab|Relationship|
|---|---|
|`00_File_Services_Index.md`|Defines where FSRM quotas fit in the File Services suite|
|`01_Install_File_Server_Role_And_Management_Tools.md`|Provides File Server role and management baseline|
|`02_Prepare_File_Server_Storage_Volumes_And_Folders.md`|Creates folder paths where quotas are applied|
|`03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`|Provides ACLs controlling who can write into quota paths|
|`04_Create_SMB_Shares_And_Share_Permissions.md`|Publishes quota-controlled folders over SMB|
|`05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`|Controls visibility of folders that may have quotas|
|`06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`|Provides audit visibility around SMB writes and quota tests|
|`07_Configure_User_Home_Folders_And_Department_Shares.md`|Provides home and department folders that receive quota policy|
|`08_Configure_File_Screening_With_FSRM.md`|Uses the same FSRM role service for file type governance|
|`10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md`|Extends quota visibility into reports and automated cleanup|
|`11_Configure_Shadow_Copies_And_Previous_Versions.md`|Requires quota planning because snapshots consume volume storage|
|`12_Backup_Restore_And_Export_File_Server_Config.md`|Exports FSRM quota configuration for recovery|
|`15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`|Helps identify users and activity related to quota growth|
|`16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`|Distinguishes quota blocks from SMB and NTFS access failures|
|`25_Migrate_File_Server_Data_Shares_And_Permissions.md`|Requires recreating or exporting FSRM quota policy during migration|

Okay yes can you now build task