
08_Configure_File_Screening_With_FSRM.md
# Configure_File_Screening_With_FSRM

# Configure_File_Screening_With_FSRM_Index
08_Configure_File_Screening_With_FSRM.md
Configure_File_Screening_With_FSRM
Configure_File_Screening_With_FSRM_Source_Basis
Configure_File_Screening_With_FSRM_Mental_Model
Configure_File_Screening_With_FSRM_Planning_Table
Configure_File_Screening_With_FSRM_Configuration_Checklist
Configure_File_Screening_With_FSRM_Precheck_Skeleton
Configure_File_Screening_With_FSRM_Install_And_Module_Validation_Skeleton
Configure_File_Screening_With_FSRM_File_Group_Skeleton
Configure_File_Screening_With_FSRM_Template_Skeleton
Configure_File_Screening_With_FSRM_Apply_File_Screen_Skeleton
Configure_File_Screening_With_FSRM_File_Screen_Exception_Skeleton
Configure_File_Screening_With_FSRM_Client_Block_Test_Skeleton
Configure_File_Screening_With_FSRM_Event_Validation_Skeleton
Configure_File_Screening_With_FSRM_Post_Change_Validation_Skeleton
Configure_File_Screening_With_FSRM_Verification_Commands
Configure_File_Screening_With_FSRM_Rollback
Configure_File_Screening_With_FSRM_Failure_Checks
Configure_File_Screening_With_FSRM_Related_Labs

# Configure_File_Screening_With_FSRM_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | File Server Resource Manager | Managing file screens, quotas, reports, and file management tasks |
| Microsoft Learn | Install-WindowsFeature | Installing the FSRM role service and management tools |
| Microsoft Learn | Get-FsrmFileGroup | Reviewing file groups used by file screens |
| Microsoft Learn | New-FsrmFileGroup | Creating custom file groups by extension or pattern |
| Microsoft Learn | New-FsrmFileScreenTemplate | Creating reusable file screen templates |
| Microsoft Learn | New-FsrmFileScreen | Applying file screens to target folders |
| Microsoft Learn | New-FsrmFileScreenException | Creating exceptions below screened paths |
| Microsoft Learn | New-FsrmAction | Creating notification actions for FSRM events |
| Microsoft Learn | Get-WinEvent | Validating FSRM-related events after blocked file attempts |
| Windows Server operational practice | File governance | Preventing unwanted file types from being stored on department, home, or public shares |
| Windows Server operational practice | Active versus passive screening | Blocking file types with active screens or monitoring them with passive screens |
| Windows Server operational practice | Defense in depth | File screening supports governance, but does not replace antivirus, EDR, or user training |

# Configure_File_Screening_With_FSRM_Mental_Model
| Concept | Operational Meaning |
|---|---|
| FSRM | File Server Resource Manager, a Windows Server role service for file screening, quotas, reports, and file management tasks |
| File screen | FSRM policy applied to a folder path that blocks or monitors selected file types |
| Active file screen | Blocks users from saving matching file types |
| Passive file screen | Allows matching files but logs or notifies for monitoring |
| File group | Named collection of file name patterns such as `*.mp3`, `*.exe`, or `*.iso` |
| Include pattern | File pattern included in the file group |
| Exclude pattern | File pattern exempted from the file group |
| File screen template | Reusable screen definition containing file groups, active/passive behavior, and notifications |
| File screen exception | Folder-level exception that allows or changes behavior under a screened parent path |
| Notification action | Event log, email, command, or report action triggered by a screening event |
| Department screen | File screen applied to department share paths |
| Home folder screen | File screen applied to user home folders |
| Public share screen | File screen applied to public drop or collaboration folders |
| Blocked file test | Attempt to save a file matching the screen pattern and confirm it fails |
| Monitoring screen | Passive screen used to discover behavior before blocking |
| User experience | User receives an access denied or save failure when active screening blocks a file |
| Event evidence | Application log or FSRM event output proving a file screen matched |
| Governance boundary | FSRM controls file type placement, not identity access |
| First rule | Test file screens in a lab or pilot folder before applying them to large production shares |

# Configure_File_Screening_With_FSRM_Planning_Table
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
| Test screen path | `E:\Shares\Departments\Accounting` | `<screen-path>` |
| Exception path | `E:\Shares\Departments\Accounting\Approved-Installers` | `<exception-path>` |
| File group name | `Blocked_Risky_File_Types` | `<file-group-name>` |
| File screen template name | `Block_Risky_File_Types_Template` | `<template-name>` |
| File group include patterns | `*.exe`, `*.bat`, `*.cmd`, `*.vbs`, `*.js`, `*.iso` | `<include-patterns>` |
| File group exclude patterns | `README.exe.txt` if needed | `<exclude-patterns>` |
| Active or passive screen | Active for lab block test | `<screen-mode>` |
| Department screen stance | Block risky files | `<department-screen-plan>` |
| Home folder screen stance | Block risky files | `<home-screen-plan>` |
| Public share screen stance | Block risky files or monitor first | `<public-screen-plan>` |
| Apps share screen stance | Usually exception or tighter admin-only control | `<apps-screen-plan>` |
| Notification type | Event log first | `<notification-plan>` |
| SMTP/email notification | Optional after SMTP config exists | `<email-notification-plan>` |
| Test blocked file | `blocked-test.exe` | `<blocked-test-file>` |
| Test allowed file | `allowed-test.txt` | `<allowed-test-file>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Client evidence path | `C:\FileServices-Validation-Client` | `<client-evidence-path>` |
| Rollback stance | Remove screens/templates/groups if test fails | `<rollback-plan>` |
| Next workbook | `09_Configure_Storage_Quotas_With_FSRM.md` | `<next-task>` |

# Configure_File_Screening_With_FSRM_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm target folders exist | File Server | `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"` | Required folders exist |
| 4 | Install FSRM role service | File Server | `Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools` | FSRM is installed |
| 5 | Confirm FSRM service exists | File Server | `Get-Service SrmSvc` | FSRM service is visible |
| 6 | Start FSRM service if needed | File Server | `Start-Service SrmSvc` | FSRM service is running |
| 7 | Confirm FSRM cmdlets are available | File Server | `Get-Command -Module FileServerResourceManager` | FSRM PowerShell cmdlets are available |
| 8 | Capture existing file groups | File Server | `Get-FsrmFileGroup` | Existing file groups are documented |
| 9 | Create custom blocked file group | File Server | `New-FsrmFileGroup -Name "<file-group-name>" -IncludePattern <patterns>` | Custom file group exists |
| 10 | Confirm custom file group | File Server | `Get-FsrmFileGroup -Name "<file-group-name>"` | File group patterns match plan |
| 11 | Create event notification action | File Server | `New-FsrmAction -Type Event -EventType Warning -Body "<message>"` | Event action object is ready |
| 12 | Create file screen template | File Server | `New-FsrmFileScreenTemplate -Name "<template-name>" -IncludeGroup "<file-group-name>" -Active` | Template exists |
| 13 | Confirm file screen template | File Server | `Get-FsrmFileScreenTemplate -Name "<template-name>"` | Template matches file group and active/passive plan |
| 14 | Apply file screen to department path | File Server | `New-FsrmFileScreen -Path "<screen-path>" -Template "<template-name>"` | File screen exists on target path |
| 15 | Apply file screen to home root if planned | File Server | `New-FsrmFileScreen -Path "<home-folder-root>" -Template "<template-name>"` | Home folder screen exists |
| 16 | Apply file screen to public root if planned | File Server | `New-FsrmFileScreen -Path "<public-share-root>" -Template "<template-name>"` | Public screen exists |
| 17 | Create exception folder if needed | File Server | `New-Item -ItemType Directory -Path "<exception-path>" -Force` | Exception path exists |
| 18 | Create file screen exception if needed | File Server | `New-FsrmFileScreenException -Path "<exception-path>" -IncludeGroup "<file-group-name>"` | Exception exists |
| 19 | Validate final file screens | File Server | `Get-FsrmFileScreen` | Applied screens are visible |
| 20 | Validate final exceptions | File Server | `Get-FsrmFileScreenException` | Exceptions are visible |
| 21 | Test allowed file locally | File Server | `New-Item -ItemType File -Path "<screen-path>\allowed-test.txt"` | Allowed file is created |
| 22 | Test blocked file locally | File Server | `New-Item -ItemType File -Path "<screen-path>\blocked-test.exe"` | Active screen blocks file creation |
| 23 | Test blocked file from client UNC | Client | `New-Item "\\<file-server>\<share>\<path>\blocked-test.exe"` | Client write attempt is blocked |
| 24 | Validate FSRM events | File Server | `Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"}` | FSRM screening event evidence is visible |
| 25 | Export final FSRM state | File Server | Use post-change validation skeleton | File groups, templates, screens, and exceptions are saved |
| 26 | Document file screening model | Operator | `Record file groups, templates, paths, exceptions, and test results` | File screening record is complete |

# Configure_File_Screening_With_FSRM_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate File Server, target folders, SMB shares, and current FSRM state before file screening changes.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"

$ScreenPath = "E:\Shares\Departments\Accounting"
$ExceptionPath = "E:\Shares\Departments\Accounting\Approved-Installers"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-fsrm-file-screening-precheck-transcript.txt"

# Confirm admin context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-fsrm-screening.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname-before-fsrm-screening.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-fsrm-screening.txt"

# Confirm File Server role.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-fsrm-screening.txt"

# Confirm FSRM state before install/config.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-before-screening.txt"

Get-Service SrmSvc -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-service-before-screening.txt"

# Confirm target paths.
$TargetPaths = @(
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $AppsRoot,
  $ScreenPath
)

foreach ($Path in $TargetPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\fsrm-target-path-validation-before.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-fsrm-screening-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Capture SMB shares because screening will be tested over SMB.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-before-fsrm-screening.txt"

# Capture current FSRM objects if cmdlets exist.
Get-Command -Module FileServerResourceManager -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-cmdlets-before-screening.txt"

Get-FsrmFileGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-groups-before-screening.txt"

Get-FsrmFileScreenTemplate -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screen-templates-before-screening.txt"

Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screens-before-screening.txt"

Get-FsrmFileScreenException -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screen-exceptions-before-screening.txt"

Stop-Transcript
````

# Configure_File_Screening_With_FSRM_Install_And_Module_Validation_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: install FSRM and validate the service and PowerShell module.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-fsrm-install-validation-transcript.txt"

# Install File Server Resource Manager.
Install-WindowsFeature `
  -Name FS-Resource-Manager `
  -IncludeManagementTools

# Confirm feature state.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-after-install.txt"

# Confirm service state.
Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-after-install.txt"

# Start service if needed.
Start-Service SrmSvc -ErrorAction SilentlyContinue

Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-after-start.txt"

# Confirm PowerShell module and cmdlets.
Get-Module FileServerResourceManager -ListAvailable |
  Tee-Object "$EvidencePath\fsrm-module-available.txt"

Import-Module FileServerResourceManager

Get-Command -Module FileServerResourceManager |
  Tee-Object "$EvidencePath\fsrm-cmdlets-after-install.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_File_Group_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create a custom FSRM file group for risky or unwanted file types.

$EvidencePath = "C:\FileServices-Validation"

$FileGroupName = "Blocked_Risky_File_Types"

$IncludePatterns = @(
  "*.exe",
  "*.bat",
  "*.cmd",
  "*.com",
  "*.scr",
  "*.ps1",
  "*.vbs",
  "*.js",
  "*.jse",
  "*.wsf",
  "*.msi",
  "*.iso"
)

$ExcludePatterns = @(
  "README.exe.txt"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-fsrm-file-group-transcript.txt"

Import-Module FileServerResourceManager

# Capture existing file groups.
Get-FsrmFileGroup |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-file-groups-before-custom-create.txt"

# Create or update custom file group.
$ExistingFileGroup = Get-FsrmFileGroup -Name $FileGroupName -ErrorAction SilentlyContinue

if (-not $ExistingFileGroup) {
  New-FsrmFileGroup `
    -Name $FileGroupName `
    -IncludePattern $IncludePatterns `
    -ExcludePattern $ExcludePatterns
}
else {
  Set-FsrmFileGroup `
    -Name $FileGroupName `
    -IncludePattern $IncludePatterns `
    -ExcludePattern $ExcludePatterns
}

# Confirm custom file group.
Get-FsrmFileGroup -Name $FileGroupName |
  Tee-Object "$EvidencePath\fsrm-file-group-$FileGroupName-after.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_Template_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create an FSRM file screen template using the custom file group.
# Event notification is used first because it does not require SMTP configuration.

$EvidencePath = "C:\FileServices-Validation"

$FileGroupName = "Blocked_Risky_File_Types"
$TemplateName = "Block_Risky_File_Types_Template"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-fsrm-template-transcript.txt"

Import-Module FileServerResourceManager

# Confirm file group exists.
Get-FsrmFileGroup -Name $FileGroupName |
  Tee-Object "$EvidencePath\fsrm-file-group-before-template.txt"

# Create event notification action.
$EventAction = New-FsrmAction `
  -Type Event `
  -EventType Warning `
  -Body "FSRM blocked or detected a file matching the $FileGroupName file group."

# Create or update active file screen template.
$ExistingTemplate = Get-FsrmFileScreenTemplate -Name $TemplateName -ErrorAction SilentlyContinue

if (-not $ExistingTemplate) {
  New-FsrmFileScreenTemplate `
    -Name $TemplateName `
    -Description "Blocks risky executable or script file types on user file shares." `
    -IncludeGroup $FileGroupName `
    -Active `
    -Notification $EventAction
}
else {
  Set-FsrmFileScreenTemplate `
    -Name $TemplateName `
    -Description "Blocks risky executable or script file types on user file shares." `
    -IncludeGroup $FileGroupName `
    -Active `
    -Notification $EventAction
}

# Confirm template.
Get-FsrmFileScreenTemplate -Name $TemplateName |
  Tee-Object "$EvidencePath\fsrm-file-screen-template-$TemplateName-after.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_Apply_File_Screen_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: apply FSRM file screens to selected share paths.

$EvidencePath = "C:\FileServices-Validation"

$TemplateName = "Block_Risky_File_Types_Template"

$ScreenPaths = @(
  "E:\Shares\Departments\Accounting",
  "E:\Shares\Home",
  "E:\Shares\Public"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-apply-file-screens-transcript.txt"

Import-Module FileServerResourceManager

# Confirm template exists.
Get-FsrmFileScreenTemplate -Name $TemplateName |
  Tee-Object "$EvidencePath\fsrm-template-before-apply-screens.txt"

foreach ($Path in $ScreenPaths) {
  if (-not (Test-Path $Path)) {
    New-Item -ItemType Directory -Force -Path $Path | Out-Null
  }

  $ExistingScreen = Get-FsrmFileScreen -Path $Path -ErrorAction SilentlyContinue

  if (-not $ExistingScreen) {
    New-FsrmFileScreen `
      -Path $Path `
      -Template $TemplateName
  }
  else {
    Set-FsrmFileScreen `
      -Path $Path `
      -Template $TemplateName
  }

  Get-FsrmFileScreen -Path $Path |
    Tee-Object "$EvidencePath\fsrm-file-screen-after-$($Path.Replace(':','').Replace('\','-')).txt"
}

# Confirm all file screens.
Get-FsrmFileScreen |
  Sort-Object Path |
  Tee-Object "$EvidencePath\fsrm-file-screens-after-apply.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_File_Screen_Exception_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: create a controlled exception folder under a screened path.
# Example use: allow approved installers only in a tightly controlled folder.

$EvidencePath = "C:\FileServices-Validation"

$FileGroupName = "Blocked_Risky_File_Types"
$ExceptionPath = "E:\Shares\Departments\Accounting\Approved-Installers"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $ExceptionPath

Start-Transcript -Path "$EvidencePath\08-file-screen-exception-transcript.txt"

Import-Module FileServerResourceManager

# Capture existing exceptions.
Get-FsrmFileScreenException -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screen-exceptions-before-create.txt"

# Create exception if missing.
$ExistingException = Get-FsrmFileScreenException -Path $ExceptionPath -ErrorAction SilentlyContinue

if (-not $ExistingException) {
  New-FsrmFileScreenException `
    -Path $ExceptionPath `
    -IncludeGroup $FileGroupName
}
else {
  Set-FsrmFileScreenException `
    -Path $ExceptionPath `
    -IncludeGroup $FileGroupName
}

# Confirm exception.
Get-FsrmFileScreenException -Path $ExceptionPath |
  Tee-Object "$EvidencePath\fsrm-file-screen-exception-after.txt"

# Capture ACL because exceptions should be tightly controlled.
icacls $ExceptionPath |
  Tee-Object "$EvidencePath\exception-path-icacls-after.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_Client_Block_Test_Skeleton

```powershell
# Run from a Windows client as a normal test user.
# Purpose: prove allowed file types succeed and blocked file types fail over SMB.

$FileServerName = "FS1"
$DepartmentShareName = "Departments"
$DepartmentName = "Accounting"

$TargetUNC = "\\$FileServerName\$DepartmentShareName\$DepartmentName"

$AllowedFile = Join-Path $TargetUNC "fsrm-allowed-test.txt"
$BlockedFile = Join-Path $TargetUNC "fsrm-blocked-test.exe"
$BlockedScript = Join-Path $TargetUNC "fsrm-blocked-test.vbs"

$EvidencePath = "C:\FileServices-Validation-Client"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-client-fsrm-file-screen-test-transcript.txt"

# Capture client identity and token.
hostname |
  Tee-Object "$EvidencePath\client-hostname-fsrm-screening.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami-fsrm-screening.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups-fsrm-screening.txt"

# Confirm SMB reachability.
Resolve-DnsName $FileServerName |
  Tee-Object "$EvidencePath\client-resolve-file-server-fsrm-screening.txt"

Test-NetConnection $FileServerName -Port 445 |
  Tee-Object "$EvidencePath\client-test-smb-445-fsrm-screening.txt"

Test-Path $TargetUNC |
  Tee-Object "$EvidencePath\client-test-target-unc-fsrm-screening.txt"

# Allowed file test. Expected to succeed.
"Allowed FSRM test file $(Get-Date)" |
  Out-File $AllowedFile -Force

Get-Content $AllowedFile |
  Tee-Object "$EvidencePath\allowed-file-readback-fsrm-screening.txt"

Remove-Item $AllowedFile -Force

# Blocked EXE test. Expected to fail when active file screen is applied.
try {
  "Blocked EXE FSRM test $(Get-Date)" |
    Out-File $BlockedFile -Force

  "Unexpected: blocked EXE file was created." |
    Tee-Object "$EvidencePath\blocked-exe-test-result.txt"
}
catch {
  "Expected block for EXE file: $($_.Exception.Message)" |
    Tee-Object "$EvidencePath\blocked-exe-test-result.txt"
}

# Blocked script test. Expected to fail when active file screen is applied.
try {
  "Blocked VBS FSRM test $(Get-Date)" |
    Out-File $BlockedScript -Force

  "Unexpected: blocked VBS file was created." |
    Tee-Object "$EvidencePath\blocked-vbs-test-result.txt"
}
catch {
  "Expected block for VBS file: $($_.Exception.Message)" |
    Tee-Object "$EvidencePath\blocked-vbs-test-result.txt"
}

# Confirm no blocked test files remain.
Get-ChildItem $TargetUNC -Force |
  Where-Object { $_.Name -like "fsrm-blocked-test*" -or $_.Name -like "fsrm-allowed-test*" } |
  Tee-Object "$EvidencePath\test-file-inventory-after-fsrm-screening.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_Event_Validation_Skeleton

```powershell
# Run in elevated PowerShell on the file server after blocked file tests.
# Purpose: validate FSRM event evidence.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-fsrm-event-validation-transcript.txt"

# FSRM commonly writes file screening events to the Application log under SRMSVC.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-4)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-application-events-last-4-hours.txt"

Get-WinEvent `
  -FilterHashtable @{
    LogName = "Application"
    ProviderName = "SRMSVC"
    StartTime = (Get-Date).AddHours(-4)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\fsrm-application-events-last-4-hours.csv" -NoTypeInformation

# Capture recent System events related to service status.
Get-WinEvent -LogName System -MaxEvents 100 |
  Where-Object {
    $_.ProviderName -like "*Service Control Manager*" -or
    $_.Message -like "*File Server Resource Manager*" -or
    $_.Message -like "*SrmSvc*"
  } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\fsrm-system-events-last-100.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_Post_Change_Validation_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture final FSRM file screening state and related evidence.

$EvidencePath = "C:\FileServices-Validation"

$FileGroupName = "Blocked_Risky_File_Types"
$TemplateName = "Block_Risky_File_Types_Template"

$ScreenPaths = @(
  "E:\Shares\Departments\Accounting",
  "E:\Shares\Home",
  "E:\Shares\Public"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\08-post-change-fsrm-screening-validation-transcript.txt"

Import-Module FileServerResourceManager

# Feature, service, and cmdlet state.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature-final-screening.txt"

Get-Service SrmSvc |
  Tee-Object "$EvidencePath\fsrm-service-final-screening.txt"

Get-Command -Module FileServerResourceManager |
  Tee-Object "$EvidencePath\fsrm-cmdlets-final-screening.txt"

# File groups.
Get-FsrmFileGroup |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-file-groups-final.txt"

Get-FsrmFileGroup -Name $FileGroupName |
  Tee-Object "$EvidencePath\fsrm-custom-file-group-final.txt"

# Templates.
Get-FsrmFileScreenTemplate |
  Sort-Object Name |
  Tee-Object "$EvidencePath\fsrm-file-screen-templates-final.txt"

Get-FsrmFileScreenTemplate -Name $TemplateName |
  Tee-Object "$EvidencePath\fsrm-custom-file-screen-template-final.txt"

# File screens.
Get-FsrmFileScreen |
  Sort-Object Path |
  Tee-Object "$EvidencePath\fsrm-file-screens-final.txt"

foreach ($Path in $ScreenPaths) {
  Get-FsrmFileScreen -Path $Path -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\fsrm-file-screen-final-$($Path.Replace(':','').Replace('\','-')).txt"
}

# Exceptions.
Get-FsrmFileScreenException -ErrorAction SilentlyContinue |
  Sort-Object Path |
  Tee-Object "$EvidencePath\fsrm-file-screen-exceptions-final.txt"

# SMB and path evidence.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-final-fsrm-screening.txt"

foreach ($Path in $ScreenPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\fsrm-screen-paths-final.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-final-fsrm-screening-$($Path.Replace(':','').Replace('\','-')).txt"
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
  Tee-Object "$EvidencePath\fsrm-events-final-screening.txt"

Stop-Transcript
```

# Configure_File_Screening_With_FSRM_Verification_Commands

```powershell
# Feature and service
Get-WindowsFeature FS-Resource-Manager
Get-Service SrmSvc
Start-Service SrmSvc

# FSRM module and cmdlets
Get-Module FileServerResourceManager -ListAvailable
Import-Module FileServerResourceManager
Get-Command -Module FileServerResourceManager

# File groups
Get-FsrmFileGroup
Get-FsrmFileGroup -Name "<file-group-name>"
New-FsrmFileGroup -Name "<file-group-name>" -IncludePattern "*.exe","*.bat","*.cmd","*.vbs","*.js","*.iso"

# File screen templates
Get-FsrmFileScreenTemplate
Get-FsrmFileScreenTemplate -Name "<template-name>"

# File screens
Get-FsrmFileScreen
Get-FsrmFileScreen -Path "<screen-path>"
New-FsrmFileScreen -Path "<screen-path>" -Template "<template-name>"
Set-FsrmFileScreen -Path "<screen-path>" -Template "<template-name>"

# File screen exceptions
Get-FsrmFileScreenException
Get-FsrmFileScreenException -Path "<exception-path>"
New-FsrmFileScreenException -Path "<exception-path>" -IncludeGroup "<file-group-name>"

# Target path and SMB validation
Test-Path "<screen-path>"
icacls "<screen-path>"
Get-SmbShare
Get-SmbShare -Name "<share-name>"
Get-SmbShareAccess -Name "<share-name>"

# Local allowed file test
New-Item -ItemType File -Path "<screen-path>\allowed-test.txt"
Remove-Item "<screen-path>\allowed-test.txt"

# Local blocked file test
New-Item -ItemType File -Path "<screen-path>\blocked-test.exe"

# Client UNC blocked file test
Test-NetConnection "<file-server-name>" -Port 445
Test-Path "\\<file-server-name>\<share-name>\<folder>"
New-Item -ItemType File -Path "\\<file-server-name>\<share-name>\<folder>\blocked-test.exe"

# Events
Get-WinEvent -FilterHashtable @{LogName="Application"; ProviderName="SRMSVC"; StartTime=(Get-Date).AddHours(-4)}
Get-WinEvent -LogName System -MaxEvents 100

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*fsrm*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*screen*"
```

# Configure_File_Screening_With_FSRM_Rollback

|Step|Task|Device|PowerShell / Command|Expected Result|
|--:|---|---|---|---|
|1|Capture current FSRM state before rollback|File Server|`Get-FsrmFileScreen`; `Get-FsrmFileScreenTemplate`; `Get-FsrmFileGroup`|Current screening state is saved|
|2|Remove test blocked files if any were created|File Server|`Remove-Item "<screen-path>\fsrm-blocked-test.*" -Force -ErrorAction SilentlyContinue`|Test artifacts are removed|
|3|Remove file screen from test path|File Server|`Remove-FsrmFileScreen -Path "<screen-path>" -Confirm:$false`|File screen is removed|
|4|Remove file screen from home root if needed|File Server|`Remove-FsrmFileScreen -Path "<home-folder-root>" -Confirm:$false`|Home screen is removed|
|5|Remove file screen from public root if needed|File Server|`Remove-FsrmFileScreen -Path "<public-share-root>" -Confirm:$false`|Public screen is removed|
|6|Remove file screen exception if created incorrectly|File Server|`Remove-FsrmFileScreenException -Path "<exception-path>" -Confirm:$false`|Exception is removed|
|7|Remove custom file screen template if no longer used|File Server|`Remove-FsrmFileScreenTemplate -Name "<template-name>" -Confirm:$false`|Template is removed|
|8|Remove custom file group if no longer used|File Server|`Remove-FsrmFileGroup -Name "<file-group-name>" -Confirm:$false`|File group is removed|
|9|Leave FSRM installed if quotas/reports are next|File Server|`Get-WindowsFeature FS-Resource-Manager`|FSRM remains ready for task 09|
|10|Uninstall FSRM only if abandoning FSRM labs|File Server|`Uninstall-WindowsFeature FS-Resource-Manager`|FSRM role service is removed|
|11|Validate rollback|File Server|`Get-FsrmFileScreen`; `Get-FsrmFileScreenTemplate`; `Get-FsrmFileGroup`|FSRM state matches rollback target|
|12|Retest allowed file creation|Client / File Server|`New-Item "<path>\rollback-test.exe"` if screen removed|File behavior matches rollback target|
|13|Document rollback result|Operator|`Record removed screens, templates, file groups, exceptions, and test results`|Rollback record is complete|

# Configure_File_Screening_With_FSRM_Failure_Checks

|Symptom|Likely Cause|Check|Corrective Action|
|---|---|---|---|
|`Get-FsrmFileGroup` is not recognized|FSRM module not installed or imported|`Get-WindowsFeature FS-Resource-Manager`; `Get-Module FileServerResourceManager -ListAvailable`|Install FSRM and import module|
|FSRM service is missing|FS-Resource-Manager role not installed|`Get-WindowsFeature FS-Resource-Manager`|Install `FS-Resource-Manager`|
|FSRM service is stopped|Service did not start after install|`Get-Service SrmSvc`|Start `SrmSvc`|
|File screen creation fails|Target path does not exist|`Test-Path "<screen-path>"`|Create or correct the target path|
|File screen creation fails|Template name typo or missing template|`Get-FsrmFileScreenTemplate`|Create template or use exact template name|
|Template creation fails|File group missing|`Get-FsrmFileGroup -Name "<file-group-name>"`|Create file group first|
|Blocked file is allowed|Screen is passive instead of active|`Get-FsrmFileScreen -Path "<screen-path>"`|Use active template or set active behavior|
|Blocked file is allowed|Screen applied to wrong folder|`Get-FsrmFileScreen`; `Test-Path "<path>"`|Apply screen to correct folder path|
|Blocked file is allowed|File pattern not included in file group|`Get-FsrmFileGroup -Name "<file-group-name>"`|Add missing pattern to include list|
|Blocked file is allowed in subfolder|File screen exception exists below screen path|`Get-FsrmFileScreenException`|Remove or adjust exception|
|Allowed file is blocked|File group pattern too broad|`Get-FsrmFileGroup -Name "<file-group-name>"`|Refine include and exclude patterns|
|Apps share breaks after screening|Executable file types blocked on installer path|`Get-FsrmFileScreen -Path "<apps-path>"`|Remove screen from Apps path or create controlled exception|
|User sees generic access denied|Active file screen blocked the save|FSRM events and client test result|Confirm event and communicate policy behavior|
|No FSRM events appear|Notification action missing or wrong log query|`Get-FsrmFileScreenTemplate`; Application log|Add event action and query `SRMSVC` provider|
|Email notifications do not send|SMTP not configured for FSRM|FSRM email settings|Configure SMTP before relying on email actions|
|File screen exception too permissive|Exception path has broad NTFS access|`icacls "<exception-path>"`|Lock down exception folder ACLs|
|Production users disrupted|Active screen deployed without pilot|FSRM events, helpdesk reports|Switch to passive screen or remove screen while testing|
|Evidence missing|Validation skeleton not run|`Test-Path "<evidence-path>"`|Re-run precheck or post-change validation skeleton|

# Configure_File_Screening_With_FSRM_Related_Labs

|Related Lab|Relationship|
|---|---|
|`00_File_Services_Index.md`|Defines where FSRM file screening fits in the File Services suite|
|`01_Install_File_Server_Role_And_Management_Tools.md`|Provides File Server role and management baseline|
|`02_Prepare_File_Server_Storage_Volumes_And_Folders.md`|Creates folder paths where file screens are applied|
|`03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`|Provides ACLs that control who can write to screened paths|
|`04_Create_SMB_Shares_And_Share_Permissions.md`|Publishes screened folders over SMB|
|`05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`|Controls visibility of folders that may also be screened|
|`06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`|Provides audit trail and secure SMB transport for screened shares|
|`07_Configure_User_Home_Folders_And_Department_Shares.md`|Provides department and home folder targets for file screening|
|`09_Configure_Storage_Quotas_With_FSRM.md`|Uses the same FSRM role service for quota enforcement|
|`10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md`|Extends FSRM governance into reports and automated file tasks|
|`15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`|Helps identify active users and file activity when screening events occur|
|`16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`|Distinguishes FSRM blocking from NTFS or SMB access failures|
|`25_Migrate_File_Server_Data_Shares_And_Permissions.md`|Requires preserving or recreating FSRM screens during migration|