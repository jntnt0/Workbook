09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md
# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Index
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts.md
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Source_Basis
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Mental_Model
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Planning_Table
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Script_Type_Map
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Configuration_Checklist
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Precheck_Skeleton
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Create_And_Link_GPO_Skeleton
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_SYSVOL_Script_Staging_Skeleton
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_GPMC_Assignment_Skeleton
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Sample_Script_Skeletons
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Client_RSOP_Validation_Skeleton
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Report_And_Backup_Skeleton
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Verification_Commands
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Rollback
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Failure_Checks
09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Related_Labs

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy startup, shutdown, logon, and logoff scripts | Assigning scripts through GPO |
| Microsoft Learn | Group Policy Management Console | Configuring script assignments under Computer Configuration and User Configuration |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up the GPO that stores script assignments |
| Microsoft Learn | SYSVOL and NETLOGON | Hosting domain script content where clients can read it |
| Microsoft Learn | `gpupdate` and `gpresult` | Client-side policy refresh and validation |
| Microsoft Learn | Group Policy Operational event log | Troubleshooting script processing and policy application |
| Windows PowerShell | PowerShell script execution, transcript logging, and output logging | Building safe and verifiable scripts |
| Windows Server operational practice | Keep scripts short, logged, idempotent, and staged in SYSVOL | Avoiding slow logons, startup delays, and untraceable script behavior |

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Startup script | Computer-side script that runs when the computer starts |
| Shutdown script | Computer-side script that runs when the computer shuts down |
| Logon script | User-side script that runs when a user signs in |
| Logoff script | User-side script that runs when a user signs out |
| Computer Configuration | GPO half used for startup and shutdown scripts |
| User Configuration | GPO half used for logon and logoff scripts |
| Script assignment | GPO setting that points to a script file and optional parameters |
| Script storage | Script files are commonly stored in the GPO script folder under SYSVOL |
| SYSVOL path | Domain replicated path clients use to retrieve GPO scripts |
| Script order | Multiple scripts can run in an ordered sequence |
| Run context | Startup and shutdown scripts run under Local System; logon and logoff scripts run in user context |
| PowerShell script | `.ps1` script assigned through Group Policy script settings |
| Batch script | `.bat` or `.cmd` script assigned through Group Policy script settings |
| Execution policy | PowerShell policy behavior that may affect script execution |
| Idempotent script | Script safe to run repeatedly without breaking state |
| Logging | Script writes output to a known log path for troubleshooting |
| Slow logon risk | Long-running logon scripts can create bad user experience |
| First rule | Use scripts only when Administrative Templates or Preferences are not the cleaner tool |
| Blunt rule | If a script cannot log what it did, it is not ready for Group Policy deployment |

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target user OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<target-user-ou-dn>` |
| Target computer OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-computer-ou-dn>` |
| Test user | `tuser` | `<test-user>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Script GPO name | `CORP-Scripts-Logon-Startup` | `<script-gpo-name>` |
| Script type | Startup, Shutdown, Logon, Logoff | `<script-type>` |
| Script language | PowerShell or Batch | `<script-language>` |
| Script source folder | `C:\GPOPrep\Scripts` | `<local-script-source>` |
| GPO script folder | `\\corp.local\SYSVOL\corp.local\Policies\{GPO-GUID}\Machine\Scripts` or `User\Scripts` | `<gpo-script-folder>` |
| Log folder | `C:\ProgramData\Corp\GPO-Script-Logs` | `<log-folder>` |
| User log folder | `%LOCALAPPDATA%\Corp\GPO-Script-Logs` | `<user-log-folder>` |
| Startup script | `Startup-Baseline.ps1` | `<startup-script>` |
| Shutdown script | `Shutdown-Baseline.ps1` | `<shutdown-script>` |
| Logon script | `Logon-Baseline.ps1` | `<logon-script>` |
| Logoff script | `Logoff-Baseline.ps1` | `<logoff-script>` |
| Parameters | `-Environment Lab` | `<script-parameters>` |
| Link enabled | Yes | `<yes-no>` |
| Enforced | No | `<yes-no>` |
| Security filtering group | `GG_GPO_Scripts_Apply` | `<filtering-group>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Remove script assignment or disable link | `<rollback-plan>` |

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Script_Type_Map
| Script Type | GPO Location | Runs As | Runs When | Best Use | Validation |
|---|---|---|---|---|---|
| Startup | `Computer Configuration > Policies > Windows Settings > Scripts > Startup` | Local System | Computer boot | Machine setup, local folder creation, machine registry, service prep | Log file under `C:\ProgramData` |
| Shutdown | `Computer Configuration > Policies > Windows Settings > Scripts > Shutdown` | Local System | Computer shutdown | Cleanup tasks, final telemetry, local state capture | Shutdown log file or next boot validation |
| Logon | `User Configuration > Policies > Windows Settings > Scripts > Logon` | Signed-in user | User sign-in | User environment setup not better handled by GPP | User log file under profile |
| Logoff | `User Configuration > Policies > Windows Settings > Scripts > Logoff` | Signed-in user | User sign-out | User cleanup, session state capture | Logoff log file or next logon validation |
| PowerShell Script | Script assignment UI PowerShell Scripts tab | Depends on script type | Depends on assignment | Modern script logic | Transcript or custom log |
| Batch Script | Script assignment UI Scripts tab | Depends on script type | Depends on assignment | Simple legacy commands | Output redirection |
| GPP Scheduled Task | Group Policy Preferences | Configurable | Trigger-based | Better for delayed or recurring work | `Get-ScheduledTask` |
| Administrative Template | ADMX policy | Policy engine | Policy refresh | Better for actual policy settings | `gpresult` and registry policy |

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target computer OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-computer-ou-dn>"` | Computer OU returns |
| 6 | Confirm target user OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-user-ou-dn>"` | User OU returns |
| 7 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 8 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled` | Test user returns |
| 9 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 10 | Confirm NETLOGON access | Management Host | `Test-Path "\\<domain-fqdn>\NETLOGON"` | Returns True |
| 11 | Create local script source folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Scripts` | Local script folder exists |
| 12 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 13 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 14 | Back up existing GPOs before changes | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 15 | Create script GPO if missing | Management Host | `New-GPO -Name "<script-gpo-name>" -Comment "Startup, shutdown, logon, and logoff script assignments"` | GPO exists |
| 16 | Link script GPO to computer OU for startup and shutdown | Management Host | `New-GPLink -Name "<script-gpo-name>" -Target "<target-computer-ou-dn>" -LinkEnabled Yes` | GPO linked to computer OU |
| 17 | Link script GPO to user OU for logon and logoff | Management Host | `New-GPLink -Name "<script-gpo-name>" -Target "<target-user-ou-dn>" -LinkEnabled Yes` | GPO linked to user OU |
| 18 | Keep links unenforced by default | Management Host | `Set-GPLink -Name "<script-gpo-name>" -Target "<target-ou-dn>" -Enforced No` | Links are not enforced |
| 19 | Confirm inheritance state | Management Host | `Get-GPInheritance -Target "<target-computer-ou-dn>"; Get-GPInheritance -Target "<target-user-ou-dn>"` | Script GPO appears in link list |
| 20 | Confirm GPO permissions | Management Host | `Get-GPPermission -Name "<script-gpo-name>" -All` | Read and Apply scope is known |
| 21 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<script-gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 22 | Create startup script | Management Host | `New-Item -ItemType File -Force -Path C:\GPOPrep\Scripts\Startup-Baseline.ps1` | Startup script exists |
| 23 | Create shutdown script | Management Host | `New-Item -ItemType File -Force -Path C:\GPOPrep\Scripts\Shutdown-Baseline.ps1` | Shutdown script exists |
| 24 | Create logon script | Management Host | `New-Item -ItemType File -Force -Path C:\GPOPrep\Scripts\Logon-Baseline.ps1` | Logon script exists |
| 25 | Create logoff script | Management Host | `New-Item -ItemType File -Force -Path C:\GPOPrep\Scripts\Logoff-Baseline.ps1` | Logoff script exists |
| 26 | Open GPMC editor | Management Host | `gpmc.msc` | GPMC opens |
| 27 | Assign startup script | Management Host | `Computer Configuration > Policies > Windows Settings > Scripts > Startup > PowerShell Scripts` | Startup script is assigned |
| 28 | Assign shutdown script | Management Host | `Computer Configuration > Policies > Windows Settings > Scripts > Shutdown > PowerShell Scripts` | Shutdown script is assigned |
| 29 | Assign logon script | Management Host | `User Configuration > Policies > Windows Settings > Scripts > Logon > PowerShell Scripts` | Logon script is assigned |
| 30 | Assign logoff script | Management Host | `User Configuration > Policies > Windows Settings > Scripts > Logoff > PowerShell Scripts` | Logoff script is assigned |
| 31 | Configure script execution behavior if required | Management Host | `Computer Configuration > Policies > Administrative Templates > System > Scripts` | Script processing behavior is documented |
| 32 | Export GPO report | Management Host | `Get-GPOReport -Name "<script-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<script-gpo-name>.html` | Report shows script assignments |
| 33 | Back up configured script GPO | Management Host | `Backup-GPO -Name "<script-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 34 | Force client policy refresh | Test Client | `gpupdate /force` | Policy refresh completes |
| 35 | Reboot test client to trigger startup script | Test Client | `Restart-Computer` | Startup script runs at boot |
| 36 | Sign in as test user to trigger logon script | Test Client | `whoami` | Logon script runs |
| 37 | Sign out as test user to trigger logoff script | Test Client | Sign out from session | Logoff script runs |
| 38 | Shut down test client to trigger shutdown script | Test Client | `Stop-Computer` | Shutdown script runs |
| 39 | Validate applied computer GPOs | Test Client | `gpresult /scope computer /r` | Script GPO appears under Applied GPOs |
| 40 | Validate applied user GPOs | Test Client | `gpresult /scope user /r` | Script GPO appears under Applied GPOs |
| 41 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-scripts.html` | HTML RSOP report exists |
| 42 | Validate script log output | Test Client | `Get-ChildItem C:\ProgramData\Corp\GPO-Script-Logs -Recurse` | Script log files exist |
| 43 | Review Group Policy events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Script processing events are visible |
| 44 | Document result | Operator | `Record script GPO, target OUs, assigned scripts, parameters, reports, backups, and client validation` | Script deployment is documented |

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring startup, shutdown, logon, and logoff scripts.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$TestComputer = "WIN11-01"
$TestUser = "tuser"

$GpoName = "CORP-Scripts-Logon-Startup"

$BasePath = "C:\GPOPrep"
$ScriptSourcePath = "$BasePath\Scripts"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $BasePath,$ScriptSourcePath,$ReportPath,$BackupPath

# Confirm domain and tool readiness.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
Get-Module -ListAvailable GroupPolicy
Get-Command -Module GroupPolicy | Select-Object Name | Sort-Object Name

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm target OUs.
Get-ADOrganizationalUnit -Identity $TargetComputerOU
Get-ADOrganizationalUnit -Identity $TargetUserOU

# Confirm test principals.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-before-scripts.txt"

Get-ADUser `
  -Identity $TestUser `
  -Properties DistinguishedName,Enabled |
  Select-Object SamAccountName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-user-before-scripts.txt"

# Capture current inheritance.
Get-GPInheritance `
  -Target $TargetComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance-before-scripts.txt"

Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\user-ou-inheritance-before-scripts.txt"

# Capture current GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-scripts.csv" -NoTypeInformation

# Back up current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link the script assignment GPO.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-Scripts-Logon-Startup"
$GpoComment = "GPO for startup, shutdown, logon, and logoff script assignments."

# Confirm target OUs exist.
Get-ADOrganizationalUnit -Identity $TargetComputerOU
Get-ADOrganizationalUnit -Identity $TargetUserOU

# Create the GPO if missing.
$ExistingGpo = Get-GPO `
  -Name $GpoName `
  -ErrorAction SilentlyContinue

if ($ExistingGpo) {
    Write-Host "GPO already exists: $GpoName"
}
else {
    New-GPO `
      -Name $GpoName `
      -Comment $GpoComment

    Write-Host "Created GPO: $GpoName"
}

# Link to computer OU for startup and shutdown.
$ComputerInheritance = Get-GPInheritance -Target $TargetComputerOU
$ComputerLink = $ComputerInheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $GpoName }

if (-not $ComputerLink) {
    New-GPLink `
      -Name $GpoName `
      -Target $TargetComputerOU `
      -LinkEnabled Yes
}

# Link to user OU for logon and logoff.
$UserInheritance = Get-GPInheritance -Target $TargetUserOU
$UserLink = $UserInheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $GpoName }

if (-not $UserLink) {
    New-GPLink `
      -Name $GpoName `
      -Target $TargetUserOU `
      -LinkEnabled Yes
}

# Safe default: links enabled, not enforced.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetComputerOU `
  -LinkEnabled Yes `
  -Enforced No

Set-GPLink `
  -Name $GpoName `
  -Target $TargetUserOU `
  -LinkEnabled Yes `
  -Enforced No

# Confirm final link state.
Get-GPInheritance -Target $TargetComputerOU | Format-List
Get-GPInheritance -Target $TargetUserOU | Format-List
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_SYSVOL_Script_Staging_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create sample scripts and stage them for GPO assignment.

Import-Module GroupPolicy

$DomainFqdn = (Get-ADDomain).DNSRoot
$GpoName = "CORP-Scripts-Logon-Startup"

$BasePath = "C:\GPOPrep"
$ScriptSourcePath = "$BasePath\Scripts"
$ReportPath = "$BasePath\Reports"

New-Item -ItemType Directory -Force -Path $ScriptSourcePath,$ReportPath

# Create local script files.
$StartupScript = "$ScriptSourcePath\Startup-Baseline.ps1"
$ShutdownScript = "$ScriptSourcePath\Shutdown-Baseline.ps1"
$LogonScript = "$ScriptSourcePath\Logon-Baseline.ps1"
$LogoffScript = "$ScriptSourcePath\Logoff-Baseline.ps1"

@'
$LogRoot = "C:\ProgramData\Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null
$LogFile = Join-Path $LogRoot "Startup-Baseline.log"
"$(Get-Date -Format o) Startup script ran as $env:USERNAME on $env:COMPUTERNAME" | Out-File $LogFile -Append
New-Item -ItemType Directory -Force -Path "C:\Corp" | Out-Null
'@ | Set-Content -Path $StartupScript -Encoding UTF8

@'
$LogRoot = "C:\ProgramData\Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null
$LogFile = Join-Path $LogRoot "Shutdown-Baseline.log"
"$(Get-Date -Format o) Shutdown script ran as $env:USERNAME on $env:COMPUTERNAME" | Out-File $LogFile -Append
'@ | Set-Content -Path $ShutdownScript -Encoding UTF8

@'
$LogRoot = Join-Path $env:LOCALAPPDATA "Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null
$LogFile = Join-Path $LogRoot "Logon-Baseline.log"
"$(Get-Date -Format o) Logon script ran as $env:USERNAME on $env:COMPUTERNAME" | Out-File $LogFile -Append
New-Item -ItemType Directory -Force -Path (Join-Path $env:USERPROFILE "Corp") | Out-Null
'@ | Set-Content -Path $LogonScript -Encoding UTF8

@'
$LogRoot = Join-Path $env:LOCALAPPDATA "Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null
$LogFile = Join-Path $LogRoot "Logoff-Baseline.log"
"$(Get-Date -Format o) Logoff script ran as $env:USERNAME on $env:COMPUTERNAME" | Out-File $LogFile -Append
'@ | Set-Content -Path $LogoffScript -Encoding UTF8

# Confirm scripts were created.
Get-ChildItem $ScriptSourcePath |
  Select-Object Name,Length,LastWriteTime |
  Out-File "$ReportPath\script-source-files.txt"

# Get GPO GUID and SYSVOL script path reference.
$Gpo = Get-GPO -Name $GpoName
$GpoGuid = $Gpo.Id.Guid

$GpoSysvolRoot = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\{$GpoGuid}"
$MachineScriptsPath = Join-Path $GpoSysvolRoot "Machine\Scripts"
$UserScriptsPath = Join-Path $GpoSysvolRoot "User\Scripts"

# Create script folders if needed.
New-Item -ItemType Directory -Force -Path $MachineScriptsPath,$UserScriptsPath

# Copy scripts into the GPO script storage area.
Copy-Item $StartupScript -Destination $MachineScriptsPath -Force
Copy-Item $ShutdownScript -Destination $MachineScriptsPath -Force
Copy-Item $LogonScript -Destination $UserScriptsPath -Force
Copy-Item $LogoffScript -Destination $UserScriptsPath -Force

# Confirm staged scripts.
Get-ChildItem $MachineScriptsPath,$UserScriptsPath |
  Select-Object FullName,Length,LastWriteTime |
  Out-File "$ReportPath\script-files-staged-in-sysvol.txt"

Write-Host "GPO SYSVOL root: $GpoSysvolRoot"
Write-Host "Machine scripts: $MachineScriptsPath"
Write-Host "User scripts: $UserScriptsPath"
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_GPMC_Assignment_Skeleton
```powershell
# Native GUI workflow for assigning startup, shutdown, logon, and logoff scripts.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-Scripts-Logon-Startup
# 7. Select Edit.
#
# Startup script:
# 8. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Scripts
#      > Startup
# 9. Open Startup.
# 10. Select the PowerShell Scripts tab.
# 11. Click Add.
# 12. Script Name:
#      Startup-Baseline.ps1
# 13. Script Parameters:
#      <optional parameters>
#
# Shutdown script:
# 14. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Scripts
#      > Shutdown
# 15. Open Shutdown.
# 16. Select the PowerShell Scripts tab.
# 17. Click Add.
# 18. Script Name:
#      Shutdown-Baseline.ps1
#
# Logon script:
# 19. Go to:
#      User Configuration
#      > Policies
#      > Windows Settings
#      > Scripts
#      > Logon
# 20. Open Logon.
# 21. Select the PowerShell Scripts tab.
# 22. Click Add.
# 23. Script Name:
#      Logon-Baseline.ps1
#
# Logoff script:
# 24. Go to:
#      User Configuration
#      > Policies
#      > Windows Settings
#      > Scripts
#      > Logoff
# 25. Open Logoff.
# 26. Select the PowerShell Scripts tab.
# 27. Click Add.
# 28. Script Name:
#      Logoff-Baseline.ps1
#
# Optional script processing settings:
# 29. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > System
#      > Scripts
# 30. Review:
#      Run startup scripts asynchronously
#      Run shutdown scripts asynchronously
#      Run logon scripts synchronously
#      Run logon scripts visible
#      Specify maximum wait time for Group Policy scripts
#
# Close editor.
# Refresh GPMC.
# Export a GPO report.
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Sample_Script_Skeletons
```powershell
# Startup-Baseline.ps1
# Runs as Local System at computer startup.

$LogRoot = "C:\ProgramData\Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null

$LogFile = Join-Path $LogRoot "Startup-Baseline.log"

try {
    "$(Get-Date -Format o) Startup script started on $env:COMPUTERNAME as $env:USERNAME" |
      Out-File $LogFile -Append

    New-Item -ItemType Directory -Force -Path "C:\Corp" | Out-Null

    "$(Get-Date -Format o) Startup script completed successfully." |
      Out-File $LogFile -Append
}
catch {
    "$(Get-Date -Format o) Startup script failed: $($_.Exception.Message)" |
      Out-File $LogFile -Append
    exit 1
}
```

```powershell
# Shutdown-Baseline.ps1
# Runs as Local System at computer shutdown.

$LogRoot = "C:\ProgramData\Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null

$LogFile = Join-Path $LogRoot "Shutdown-Baseline.log"

try {
    "$(Get-Date -Format o) Shutdown script started on $env:COMPUTERNAME as $env:USERNAME" |
      Out-File $LogFile -Append

    "$(Get-Date -Format o) Shutdown script completed successfully." |
      Out-File $LogFile -Append
}
catch {
    "$(Get-Date -Format o) Shutdown script failed: $($_.Exception.Message)" |
      Out-File $LogFile -Append
    exit 1
}
```

```powershell
# Logon-Baseline.ps1
# Runs in the signed-in user's context at logon.

$LogRoot = Join-Path $env:LOCALAPPDATA "Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null

$LogFile = Join-Path $LogRoot "Logon-Baseline.log"

try {
    "$(Get-Date -Format o) Logon script started on $env:COMPUTERNAME as $env:USERNAME" |
      Out-File $LogFile -Append

    New-Item -ItemType Directory -Force -Path (Join-Path $env:USERPROFILE "Corp") | Out-Null

    "$(Get-Date -Format o) Logon script completed successfully." |
      Out-File $LogFile -Append
}
catch {
    "$(Get-Date -Format o) Logon script failed: $($_.Exception.Message)" |
      Out-File $LogFile -Append
    exit 1
}
```

```powershell
# Logoff-Baseline.ps1
# Runs in the signed-in user's context at logoff.

$LogRoot = Join-Path $env:LOCALAPPDATA "Corp\GPO-Script-Logs"
New-Item -ItemType Directory -Force -Path $LogRoot | Out-Null

$LogFile = Join-Path $LogRoot "Logoff-Baseline.log"

try {
    "$(Get-Date -Format o) Logoff script started on $env:COMPUTERNAME as $env:USERNAME" |
      Out-File $LogFile -Append

    "$(Get-Date -Format o) Logoff script completed successfully." |
      Out-File $LogFile -Append
}
catch {
    "$(Get-Date -Format o) Logoff script failed: $($_.Exception.Message)" |
      Out-File $LogFile -Append
    exit 1
}
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: prove script GPO applies and scripts leave verifiable logs.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm user and client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Force policy refresh.
gpupdate /force

# Show applied computer GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-scripts.txt"

# Show applied user GPOs.
gpresult /scope user /r |
  Tee-Object "$ReportPath\gpresult-user-scripts.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-scripts.html"

# Optional RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-scripts.html" `
  -ErrorAction SilentlyContinue

# Validate startup or shutdown script logs.
Get-ChildItem `
  -Path "C:\ProgramData\Corp\GPO-Script-Logs" `
  -Recurse `
  -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,LastWriteTime

# Validate logon or logoff script logs.
Get-ChildItem `
  -Path "$env:LOCALAPPDATA\Corp\GPO-Script-Logs" `
  -Recurse `
  -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,LastWriteTime

# Read log content.
Get-Content `
  -Path "C:\ProgramData\Corp\GPO-Script-Logs\Startup-Baseline.log" `
  -ErrorAction SilentlyContinue

Get-Content `
  -Path "$env:LOCALAPPDATA\Corp\GPO-Script-Logs\Logon-Baseline.log" `
  -ErrorAction SilentlyContinue

# Review Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 150 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-scripts.txt"
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final script GPO report, inheritance, permissions, SYSVOL files, and backup.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-Scripts-Logon-Startup"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $TargetComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance-after-scripts.txt"

Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\user-ou-inheritance-after-scripts.txt"

# Capture final GPO permissions.
Get-GPPermission `
  -Name $GpoName `
  -All |
  Out-File "$ReportPath\$GpoName-permissions-after.txt"

# Export final reports.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-final.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-final.xml"

# Search XML report for script content.
Select-String `
  -Path "$ReportPath\$GpoName-final.xml" `
  -Pattern "Startup","Shutdown","Logon","Logoff","PowerShell","Startup-Baseline.ps1","Logon-Baseline.ps1" |
  Out-File "$ReportPath\$GpoName-script-report-search.txt"

# Capture staged SYSVOL script files.
$Gpo = Get-GPO -Name $GpoName
$GpoGuid = $Gpo.Id.Guid
$GpoSysvolRoot = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\{$GpoGuid}"

Get-ChildItem `
  -Path $GpoSysvolRoot `
  -Recurse `
  -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,LastWriteTime |
  Out-File "$ReportPath\$GpoName-sysvol-script-files.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-scripts.csv" -NoTypeInformation

Write-Host "Script GPO report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<script-gpo-name>"` | Confirms script GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<target-computer-ou-dn>"` | Confirms computer-side script GPO link | GPO appears in link list |
| `Get-GPInheritance -Target "<target-user-ou-dn>"` | Confirms user-side script GPO link | GPO appears in link list |
| `Get-GPPermission -Name "<script-gpo-name>" -All` | Confirms users and computers can read and apply GPO | Expected permissions appear |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms SYSVOL is reachable | Returns True |
| `Get-GPOReport -Name "<script-gpo-name>" -ReportType Html -Path "<report-path>\<script-gpo-name>.html"` | Exports readable GPO report | HTML report exists |
| `Get-GPOReport -Name "<script-gpo-name>" -ReportType Xml -Path "<report-path>\<script-gpo-name>.xml"` | Exports searchable GPO report | XML report exists |
| `Select-String -Path "<report-path>\<script-gpo-name>.xml" -Pattern "Startup","Shutdown","Logon","Logoff"` | Searches report for script assignments | Script assignment entries appear |
| `Backup-GPO -Name "<script-gpo-name>" -Path "<backup-path>"` | Backs up configured script GPO | Backup completes |
| `gpupdate /force` | Forces policy refresh | Policy refresh completes |
| `Restart-Computer` | Triggers startup script on next boot | Startup script log appears |
| `Stop-Computer` | Triggers shutdown script | Shutdown script log appears |
| `gpresult /scope computer /r` | Shows applied computer-side GPOs | Script GPO appears under Applied GPOs |
| `gpresult /scope user /r` | Shows applied user-side GPOs | Script GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-scripts.html` | Exports full RSOP report | HTML report exists |
| `Get-ChildItem C:\ProgramData\Corp\GPO-Script-Logs -Recurse` | Validates computer-side script logs | Startup or shutdown logs appear |
| `Get-ChildItem "$env:LOCALAPPDATA\Corp\GPO-Script-Logs" -Recurse` | Validates user-side script logs | Logon or logoff logs appear |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews script processing events | Recent policy processing events appear |
| `repadmin /replsummary` | Checks replication if script settings differ between DCs | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks DC and SYSVOL health | Tests pass |

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: rollback script deployment by disabling the link, removing script assignments in GPMC, or removing staged files.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"
$TargetUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoName = "CORP-Scripts-Logon-Startup"

$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture state before rollback.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-before-rollback.html"

Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Rollback option 1:
# Disable links while preserving the GPO for review.
Set-GPLink `
  -Name $GpoName `
  -Target $TargetComputerOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

Set-GPLink `
  -Name $GpoName `
  -Target $TargetUserOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Use GPMC to remove individual script assignments.
gpmc.msc

# GUI rollback workflow:
# 1. Open the GPO in Group Policy Management Editor.
# 2. Go to Computer Configuration > Policies > Windows Settings > Scripts.
# 3. Remove Startup and Shutdown script assignments.
# 4. Go to User Configuration > Policies > Windows Settings > Scripts.
# 5. Remove Logon and Logoff script assignments.
# 6. Export a fresh GPO report.
# 7. Run gpupdate /force on the client.
# 8. Reboot or sign out/sign in as needed to confirm scripts no longer run.

# Rollback option 3:
# Remove staged script files from this GPO SYSVOL folder.
# Use only after removing script assignments.
$Gpo = Get-GPO -Name $GpoName -ErrorAction SilentlyContinue

if ($Gpo) {
    $GpoGuid = $Gpo.Id.Guid
    $GpoSysvolRoot = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\{$GpoGuid}"

    # Lab cleanup examples:
    # Remove-Item "$GpoSysvolRoot\Machine\Scripts\Startup-Baseline.ps1" -Force -ErrorAction SilentlyContinue
    # Remove-Item "$GpoSysvolRoot\Machine\Scripts\Shutdown-Baseline.ps1" -Force -ErrorAction SilentlyContinue
    # Remove-Item "$GpoSysvolRoot\User\Scripts\Logon-Baseline.ps1" -Force -ErrorAction SilentlyContinue
    # Remove-Item "$GpoSysvolRoot\User\Scripts\Logoff-Baseline.ps1" -Force -ErrorAction SilentlyContinue
}

# Rollback option 4:
# Remove the whole GPO if it was created only for this lab.
# Remove-GPO `
#   -Name $GpoName

# Capture after rollback.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-after-rollback.html" `
  -ErrorAction SilentlyContinue

Get-GPInheritance `
  -Target $TargetComputerOU |
  Out-File "$ReportPath\computer-ou-inheritance-after-script-rollback.txt"

Get-GPInheritance `
  -Target $TargetUserOU |
  Out-File "$ReportPath\user-ou-inheritance-after-script-rollback.txt"

Write-Host "Rollback action complete. Run gpupdate /force, reboot, and sign out/sign in to validate."
```

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Script GPO does not apply | GPO not linked to correct user or computer OU | `Get-GPInheritance -Target "<target-ou-dn>"` | Link GPO to the correct OU |
| Startup script does not run | Computer object not in linked OU or reboot not performed | `gpresult /scope computer /r` | Move computer into scope and reboot |
| Shutdown script does not run | Computer-side script not assigned or shutdown not completed cleanly | GPO report and event logs | Confirm shutdown assignment and test controlled shutdown |
| Logon script does not run | User object not in linked OU or user policy not refreshed | `gpresult /scope user /r` | Move user into scope and sign out/sign in |
| Logoff script does not run | Script not assigned or user session did not fully log off | GPO report and log file check | Confirm logoff assignment and test clean sign out |
| GPO appears denied | Security filtering or WMI filter blocks target | `gpresult /r`; `Get-GPPermission` | Fix filtering or WMI filter |
| PowerShell script blocked | Execution policy or script processing setting issue | GroupPolicy Operational log; script policy settings | Configure script processing policy or use supported GPO PowerShell Scripts tab |
| Script file not found | File not staged in GPO SYSVOL script folder | `Get-ChildItem "\\<domain>\SYSVOL\<domain>\Policies\{GUID}" -Recurse` | Copy script into correct GPO script folder or reselect script in GPMC |
| Script works manually but not through GPO | Wrong run context | Check whether startup runs as System or logon runs as user | Rewrite script for correct context |
| Startup script cannot access network share | Local System lacks share or network access timing issue | Script log and event logs | Use domain computer permissions or copy required content to SYSVOL |
| Logon script cannot write to ProgramData | User lacks permission | Script log or lack of log | Write user logs to `%LOCALAPPDATA%` |
| Script causes slow logon | Long-running script, network dependency, or synchronous processing | GroupPolicy Operational event timing | Shorten script, move work to scheduled task, or run asynchronously |
| Script causes slow startup | Long-running startup script or unavailable network resource | Startup timing and event logs | Add timeout handling and avoid slow network calls |
| Script runs repeatedly and breaks state | Script is not idempotent | Review script logic and logs | Make script check current state before changing anything |
| Script produces no logs | Script lacks logging or fails before log folder creation | Check script content | Add try/catch and log creation at top of script |
| GPO report does not show scripts | Wrong GPO edited or stale report | `Get-GPOReport -Name "<script-gpo-name>"` | Edit correct GPO and export fresh report |
| Script settings differ between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before more testing |
| Script path has quoting or spaces issue | Bad script path or parameters | GPO report | Quote parameters correctly or simplify path |
| User script runs but mapped resources fail | GPP would be better or permissions are wrong | Test share path as user | Use GPP drive maps or fix share/NTFS permissions |
| Legacy batch script works but PowerShell script does not | PowerShell syntax, execution policy, or script tab issue | Event logs and manual test | Test script manually, then assign through PowerShell Scripts tab |
| Script continues after rollback | Client has not refreshed, startup needs reboot, user needs new session | `gpupdate /force`; reboot; sign out/sign in | Refresh policy and start a new session |

# 09_Configure_Logon_Logoff_Startup_And_Shutdown_Scripts_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before script deployment |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which users and computers can apply script GPO |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Adds optional targeting before script processing |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Computer-side policy companion workbook |
| `07_Configure_User_Administrative_Template_Settings.md` | User-side policy companion workbook |
| `08_Configure_Group_Policy_Preferences.md` | Preferred tool for many user environment tasks before scripting |
| `Create_Baseline_OU_Structure.md` | Provides OU scope for script GPO links |
| `Create_Baseline_Groups_And_Test_Users.md` | Provides users, computers, and filtering groups for script testing |
| `Join_Windows_Client_To_Domain.md` | Provides domain-joined test client for script validation |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms SYSVOL and DC locator before troubleshooting scripts |
| `10_Configure_Folder_Redirection_And_User_Profile_Settings.md` | Next workbook for user profile and data-path configuration |