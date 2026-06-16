19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs.md
# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Index
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs.md
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Source_Basis
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Mental_Model
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Planning_Table
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Control_Map
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Configuration_Checklist
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Precheck_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Create_And_Link_GPO_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Software_Restriction_Policies_GPMC_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_AppLocker_GPMC_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_AppLocker_PowerShell_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_WDAC_Audit_Policy_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_WDAC_Deployment_GPO_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Client_RSOP_Validation_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Report_And_Backup_Skeleton
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Verification_Commands
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Rollback
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Failure_Checks
19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Related_Labs

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Software Restriction Policies | Legacy application restriction through Security Settings |
| Microsoft Learn | AppLocker | Rule-based application control through GPO |
| Microsoft Learn | AppLocker PowerShell cmdlets | Building, testing, exporting, and importing AppLocker policy |
| Microsoft Learn | Application Identity service | Required service for AppLocker enforcement |
| Microsoft Learn | Windows Defender Application Control | Code integrity policy creation and deployment |
| Microsoft Learn | ConfigCI PowerShell module | Creating, converting, merging, and validating WDAC policies |
| Microsoft Learn | Group Policy Management Console | Configuring SRP, AppLocker, and deployment GPOs |
| Microsoft Learn | Event Viewer AppLocker and CodeIntegrity logs | Auditing allowed and blocked application activity |
| Windows Server operational practice | Audit before enforce | Preventing business application outages and endpoint lockouts |

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Software Restriction Policies | Older GPO application control method based on default security level and additional rules |
| AppLocker | GPO application control method using executable, script, MSI, DLL, and packaged app rules |
| WDAC | Stronger code integrity control that can restrict what code is allowed to run at the OS trust boundary |
| Audit mode | Logs what would be blocked without actually blocking execution |
| Enforcement mode | Blocks code that does not match allow rules |
| Default rules | Baseline AppLocker rules that allow Windows and Program Files paths for normal operation |
| Publisher rule | Allows or blocks software based on file signing publisher metadata |
| Path rule | Allows or blocks software based on file path |
| Hash rule | Allows or blocks a specific file hash |
| Script rule | AppLocker rule for scripts such as PowerShell, batch, VBScript, and JavaScript |
| MSI rule | AppLocker rule for Windows Installer packages |
| DLL rule | AppLocker rule for DLL loading control, powerful but higher noise and risk |
| Packaged app rule | AppLocker rule for Microsoft Store or packaged apps |
| Application Identity service | Service that must run for AppLocker policy evaluation |
| Code Integrity policy | WDAC policy that defines trusted signing, file, and rule behavior |
| Supplemental policy | WDAC policy that adds allow rules to a base policy |
| Managed Installer | Mechanism where software installed by trusted deployment tools can be trusted |
| First rule | Start with audit mode and default allow rules before enforcement |
| Blunt rule | App control can brick productivity fast, so pilot OU only, collect logs, then enforce |

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Test user | `tuser` | `<test-user>` |
| SRP GPO | `CORP-Workstation-SRP-Legacy-Test` | `<srp-gpo-name>` |
| AppLocker GPO | `CORP-Workstation-AppLocker-Audit` | `<applocker-gpo-name>` |
| WDAC GPO | `CORP-Workstation-WDAC-Audit` | `<wdac-gpo-name>` |
| App control mode | Audit first | `<audit-or-enforce>` |
| Target control | SRP, AppLocker, WDAC, or phased | `<control-type>` |
| Security filtering group | `GG_GPO_AppControl_Apply` | `<filtering-group>` |
| Approved local admin group | `GG_Workstation_Local_Admins` | `<admin-group>` |
| Application inventory source | Reference workstation | `<inventory-source>` |
| Approved software paths | `C:\Windows`, `C:\Program Files`, `C:\Program Files (x86)` | `<approved-paths>` |
| Approved publisher | Microsoft, Adobe, Google, vendor names | `<approved-publishers>` |
| Script policy stance | Audit or allow signed scripts | `<script-policy>` |
| DLL rule stance | Disabled until mature baseline | `<dll-rule-decision>` |
| WDAC policy source path | `C:\GPOPrep\WDAC` | `<wdac-source-path>` |
| WDAC policy share | `\\corp.local\NETLOGON\WDAC` | `<wdac-share-path>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Disable GPO link, set audit mode, remove WDAC policy, restore backup | `<rollback-plan>` |

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Control_Map
| Control | GPO Location / Tool | Best Use | Recommended Starting Mode | Validation |
|---|---|---|---|---|
| Software Restriction Policies | `Computer Configuration > Policies > Windows Settings > Security Settings > Software Restriction Policies` | Legacy restriction scenarios | Very limited pilot | `gpresult`, event logs, application launch tests |
| AppLocker EXE rules | `Security Settings > Application Control Policies > AppLocker > Executable Rules` | Control `.exe` and `.com` files | Audit only | AppLocker EXE and DLL log |
| AppLocker Script rules | `AppLocker > Script Rules` | Control PowerShell, batch, VBScript, JavaScript | Audit only | AppLocker MSI and Script log |
| AppLocker MSI rules | `AppLocker > Windows Installer Rules` | Control `.msi` and `.msp` installers | Audit only | AppLocker MSI and Script log |
| AppLocker DLL rules | `AppLocker > DLL Rules` | Control DLL loading | Usually defer | AppLocker EXE and DLL log |
| AppLocker Packaged app rules | `AppLocker > Packaged app Rules` | Control packaged apps | Audit only | Packaged app logs |
| AppLocker enforcement | `AppLocker > Configure rule enforcement` | Enforce after successful audit | Audit first, enforce later | `Get-AppLockerPolicy -Effective` |
| Application Identity service | Services policy or startup script | Required for AppLocker | Automatic | `Get-Service AppIDSvc` |
| WDAC base policy | ConfigCI PowerShell | Strong application control | Audit mode first | CodeIntegrity Operational log |
| WDAC deployment | GPO, startup script, or managed deployment | Deploy `.cip` or legacy policy file | Pilot only | `citool`, CodeIntegrity logs |
| WDAC supplemental policy | ConfigCI PowerShell | Add controlled allow rules | Audit then enforce | Policy inventory and CodeIntegrity logs |

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm pilot OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-ou-dn>"` | Pilot OU returns |
| 6 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 7 | Confirm test computer is in pilot OU | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under pilot OU |
| 8 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 9 | Confirm NETLOGON access | Management Host | `Test-Path "\\<domain-fqdn>\NETLOGON"` | Returns True |
| 10 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 11 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 12 | Back up all current GPOs | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 13 | Create AppLocker GPO | Management Host | `New-GPO -Name "<applocker-gpo-name>" -Comment "AppLocker audit policy for pilot workstations"` | AppLocker GPO exists |
| 14 | Link AppLocker GPO to pilot OU | Management Host | `New-GPLink -Name "<applocker-gpo-name>" -Target "<pilot-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 15 | Keep AppLocker GPO unenforced at link level | Management Host | `Set-GPLink -Name "<applocker-gpo-name>" -Target "<pilot-ou-dn>" -Enforced No` | Link is not enforced |
| 16 | Configure AppLocker default rules | Management Host | `GPMC > AppLocker > Create Default Rules` | Default allow rules exist |
| 17 | Configure AppLocker rule enforcement as audit | Management Host | `GPMC > AppLocker > Configure rule enforcement` | Collections set to Audit only |
| 18 | Configure EXE rules | Management Host | `GPMC > AppLocker > Executable Rules` | EXE rules exist |
| 19 | Configure Script rules | Management Host | `GPMC > AppLocker > Script Rules` | Script rules exist |
| 20 | Configure MSI rules | Management Host | `GPMC > AppLocker > Windows Installer Rules` | MSI rules exist |
| 21 | Defer DLL enforcement unless specifically testing | Management Host | `GPMC > AppLocker > DLL Rules` | DLL rules remain unconfigured or audit only |
| 22 | Configure Application Identity service | Management Host | `GPMC > System Services > Application Identity` | Service configured to start |
| 23 | Export AppLocker GPO report | Management Host | `Get-GPOReport -Name "<applocker-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<applocker-gpo-name>.html` | Report shows AppLocker settings |
| 24 | Back up AppLocker GPO | Management Host | `Backup-GPO -Name "<applocker-gpo-name>" -Path C:\GPOPrep\Backup` | GPO backup exists |
| 25 | Force policy refresh on client | Test Client | `gpupdate /force` | Policy refresh completes |
| 26 | Start Application Identity service | Test Client | `Start-Service AppIDSvc` | Service starts |
| 27 | Validate AppLocker policy | Test Client | `Get-AppLockerPolicy -Effective` | Effective AppLocker policy returns |
| 28 | Test normal applications | Test Client | Launch Windows tools and approved apps | Apps launch in audit mode |
| 29 | Review AppLocker logs | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 100` | Audit events are visible |
| 30 | Create WDAC working folder | Reference Client | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\WDAC` | WDAC folder exists |
| 31 | Create WDAC audit policy | Reference Client | `New-CIPolicy -FilePath C:\GPOPrep\WDAC\WDAC-Audit.xml -Level Publisher -Fallback Hash -UserPEs -ScanPath C:\` | WDAC XML policy is created |
| 32 | Set WDAC audit mode rule option | Reference Client | `Set-RuleOption -FilePath C:\GPOPrep\WDAC\WDAC-Audit.xml -Option 3` | WDAC policy remains audit mode |
| 33 | Convert WDAC XML to binary policy | Reference Client | `ConvertFrom-CIPolicy -XmlFilePath C:\GPOPrep\WDAC\WDAC-Audit.xml -BinaryFilePath C:\GPOPrep\WDAC\WDAC-Audit.cip` | WDAC binary policy exists |
| 34 | Stage WDAC policy in NETLOGON | Management Host | `Copy-Item C:\GPOPrep\WDAC\WDAC-Audit.cip "\\<domain-fqdn>\NETLOGON\WDAC" -Force` | WDAC policy is staged |
| 35 | Create WDAC deployment GPO | Management Host | `New-GPO -Name "<wdac-gpo-name>" -Comment "WDAC audit deployment for pilot workstations"` | WDAC GPO exists |
| 36 | Link WDAC GPO to pilot OU | Management Host | `New-GPLink -Name "<wdac-gpo-name>" -Target "<pilot-ou-dn>" -LinkEnabled Yes` | WDAC GPO is linked |
| 37 | Deploy WDAC through startup script or file copy workflow | Management Host | `GPMC > Computer Configuration > Policies > Windows Settings > Scripts > Startup` | WDAC policy deployment script is assigned |
| 38 | Refresh and reboot pilot client | Test Client | `gpupdate /force; Restart-Computer` | Client reboots and loads policy |
| 39 | Validate WDAC policy files | Test Client | `Get-ChildItem C:\Windows\System32\CodeIntegrity -Recurse` | WDAC policy file appears |
| 40 | Review CodeIntegrity logs | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-CodeIntegrity/Operational" -MaxEvents 100` | WDAC audit events appear |
| 41 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-app-control.html` | HTML RSOP report exists |
| 42 | Document audit results | Operator | `Record allowed apps, audit-only blocks, false positives, policy files, reports, and event logs` | App control audit is documented |
| 43 | Move to enforcement only after clean audit | Change Control | `Change mode from audit to enforce only after approval` | Enforcement is intentionally staged |

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring SRP, AppLocker, or WDAC GPOs.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$TestComputer = "WIN11-01"
$TestUser = "tuser"

$AppLockerGpoName = "CORP-Workstation-AppLocker-Audit"
$WdacGpoName = "CORP-Workstation-WDAC-Audit"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports\AppControl"
$BackupPath = "$BasePath\Backup\AppControl"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm domain identity.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName,PDCEmulator |
  Out-File "$ReportPath\domain-info.txt"

# Confirm SYSVOL and NETLOGON.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

Test-Path "\\$DomainFqdn\NETLOGON" |
  Out-File "$ReportPath\netlogon-check.txt"

# Confirm pilot OU and test objects.
Get-ADOrganizationalUnit -Identity $PilotOU |
  Out-File "$ReportPath\pilot-ou.txt"

Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,Enabled,DistinguishedName |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-before-app-control.txt"

Get-ADUser `
  -Identity $TestUser `
  -Properties UserPrincipalName,Enabled,DistinguishedName |
  Select-Object SamAccountName,UserPrincipalName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-user-before-app-control.txt"

# Capture current GPO state.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-app-control.txt"

Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-app-control.csv" -NoTypeInformation

# Back up current GPOs.
Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre application control GPO backup"

Write-Host "Application control precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link AppLocker and WDAC pilot GPOs.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$AppLockerGpoName = "CORP-Workstation-AppLocker-Audit"
$WdacGpoName = "CORP-Workstation-WDAC-Audit"

# Confirm pilot OU.
Get-ADOrganizationalUnit -Identity $PilotOU

# Create GPOs if missing.
foreach ($GpoName in @($AppLockerGpoName,$WdacGpoName)) {
    if (-not (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue)) {
        New-GPO `
          -Name $GpoName `
          -Comment "Pilot application control policy. Audit first before enforcement."
    }

    $Inheritance = Get-GPInheritance -Target $PilotOU

    if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $GpoName)) {
        New-GPLink `
          -Name $GpoName `
          -Target $PilotOU `
          -LinkEnabled Yes
    }

    Set-GPLink `
      -Name $GpoName `
      -Target $PilotOU `
      -LinkEnabled Yes `
      -Enforced No
}

# Confirm link state.
Get-GPInheritance `
  -Target $PilotOU |
  Format-List
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Software_Restriction_Policies_GPMC_Skeleton
```powershell
# Native GUI workflow for Software Restriction Policies.
# Use SRP only when specifically required for legacy coverage.
# AppLocker is usually a cleaner option for supported Windows editions.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit the intended SRP GPO:
#      CORP-Workstation-SRP-Legacy-Test
# 3. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Software Restriction Policies
# 4. If no policies exist:
#      Right-click Software Restriction Policies
#      Select New Software Restriction Policies
#
# Safer pilot approach:
# 5. Leave default security level as Unrestricted unless testing a controlled restriction.
# 6. Add specific Disallowed rules for known unwanted paths or files only.
#
# Example Additional Rules:
# 7. Path Rule:
#      %USERPROFILE%\Downloads\*
#      Security level: Disallowed
# 8. Path Rule:
#      %TEMP%\*
#      Security level: Disallowed
# 9. Hash Rule:
#      Select known test executable
#      Security level: Disallowed
#
# Enforcement:
# 10. Review:
#      Enforcement
# 11. Decide whether policy applies to:
#      All software files
#      or
#      All software files except libraries
#
# Design notes:
# - Do not set default security level to Disallowed unless you have a complete allowlist and console access.
# - Do not use broad path blocks without testing installer and update behavior.
# - Prefer AppLocker or WDAC for modern application control strategy.
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_AppLocker_GPMC_Skeleton
```powershell
# Native GUI workflow for AppLocker in audit mode.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-Workstation-AppLocker-Audit
# 3. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Application Control Policies
#      > AppLocker
#
# Configure rule enforcement:
# 4. Right-click AppLocker.
# 5. Select Properties.
# 6. Configure each rule collection:
#      Executable rules: Audit only
#      Windows Installer rules: Audit only
#      Script rules: Audit only
#      Packaged app rules: Audit only
#      DLL rules: Not configured or Audit only for advanced testing
#
# Create default rules:
# 7. Right-click Executable Rules.
# 8. Select Create Default Rules.
# 9. Right-click Windows Installer Rules.
# 10. Select Create Default Rules.
# 11. Right-click Script Rules.
# 12. Select Create Default Rules.
# 13. Right-click Packaged app Rules.
# 14. Select Create Default Rules.
#
# Create additional publisher rules:
# 15. Right-click Executable Rules.
# 16. Select Create New Rule.
# 17. Action:
#      Allow
# 18. User or group:
#      Everyone or approved group
# 19. Condition:
#      Publisher
# 20. Select a signed application file from approved software.
# 21. Choose publisher/product/file version scope carefully.
#
# Application Identity service:
# 22. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > System Services
# 23. Open:
#      Application Identity
# 24. Define this policy setting:
#      Automatic
#
# Close editor.
# Export GPO report.
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_AppLocker_PowerShell_Skeleton
```powershell
# Run on a reference workstation or management host.
# Purpose: create, test, and export AppLocker policy.

$ReportPath = "C:\GPOPrep\Reports\AppControl"
$PolicyPath = "C:\GPOPrep\AppLocker"

New-Item -ItemType Directory -Force -Path $ReportPath,$PolicyPath

# Create AppLocker policy from installed files.
# Start with common allow rules and audit mode before enforcement.
$Policy = New-AppLockerPolicy `
  -DefaultRule `
  -RuleType Executable,WindowsInstaller,Script,PackagedApp `
  -User Everyone

# Export policy to XML.
$Policy |
  Set-Content "$PolicyPath\AppLocker-DefaultRules.xml"

# Get effective policy on local machine after GPO applies.
Get-AppLockerPolicy `
  -Effective `
  -Xml |
  Out-File "$ReportPath\AppLocker-Effective.xml"

# Test policy against common paths.
$FilesToTest = Get-ChildItem `
  -Path "C:\Windows\System32","C:\Program Files" `
  -Include *.exe,*.ps1,*.msi `
  -Recurse `
  -ErrorAction SilentlyContinue |
  Select-Object -First 200

Test-AppLockerPolicy `
  -XmlPolicy "$PolicyPath\AppLocker-DefaultRules.xml" `
  -Path $FilesToTest.FullName `
  -User Everyone |
  Export-Csv "$ReportPath\AppLocker-Test-Results.csv" -NoTypeInformation

# Importing XML into a GPO is often done through GPMC:
# GPMC > AppLocker > right-click AppLocker > Import Policy
#
# You can also use Set-AppLockerPolicy for local or LDAP-based targets when fully validated.
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_WDAC_Audit_Policy_Skeleton
```powershell
# Run on a clean reference workstation.
# Purpose: create a WDAC audit policy from a known-good reference image.
# Start in audit mode. Do not deploy enforcement until audit data is clean.

$WdacRoot = "C:\GPOPrep\WDAC"
$ReportPath = "C:\GPOPrep\Reports\AppControl-WDAC"

New-Item -ItemType Directory -Force -Path $WdacRoot,$ReportPath

$PolicyXml = "$WdacRoot\WDAC-Audit.xml"
$PolicyBinary = "$WdacRoot\WDAC-Audit.cip"

# Create policy from reference system.
# Publisher level is easier to maintain than hash-only policy.
New-CIPolicy `
  -FilePath $PolicyXml `
  -Level Publisher `
  -Fallback Hash `
  -UserPEs `
  -ScanPath "C:\" `
  -NoShadowCopy `
  -ErrorAction SilentlyContinue

# Set audit mode rule option.
# Option 3 is commonly used for audit mode.
Set-RuleOption `
  -FilePath $PolicyXml `
  -Option 3

# Optional rule options should be reviewed before use.
# Examples depend on design:
# Set-RuleOption -FilePath $PolicyXml -Option <option-number>

# Convert XML policy to binary format.
ConvertFrom-CIPolicy `
  -XmlFilePath $PolicyXml `
  -BinaryFilePath $PolicyBinary

# Capture inventory.
Get-ChildItem `
  -Path $WdacRoot `
  -File |
  Select-Object FullName,Length,LastWriteTime |
  Out-File "$ReportPath\wdac-policy-files.txt"

Get-FileHash `
  -Path $PolicyXml,$PolicyBinary |
  Export-Csv "$ReportPath\wdac-policy-hashes.csv" -NoTypeInformation

Write-Host "WDAC audit policy created."
Write-Host "Policy XML: $PolicyXml"
Write-Host "Policy binary: $PolicyBinary"
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_WDAC_Deployment_GPO_Skeleton
```powershell
# Run on a management host.
# Purpose: stage WDAC policy and deploy it to pilot computers using GPO startup workflow.
# This workbook uses audit mode first.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$WdacGpoName = "CORP-Workstation-WDAC-Audit"

$WdacSource = "C:\GPOPrep\WDAC\WDAC-Audit.cip"
$WdacNetlogonFolder = "\\$DomainFqdn\NETLOGON\WDAC"
$WdacNetlogonPolicy = "$WdacNetlogonFolder\WDAC-Audit.cip"

$ScriptPath = "C:\GPOPrep\Scripts"
$DeployScript = "$ScriptPath\Deploy-WDAC-Audit.ps1"

New-Item -ItemType Directory -Force -Path $WdacNetlogonFolder,$ScriptPath

# Stage policy in NETLOGON.
Copy-Item `
  -Path $WdacSource `
  -Destination $WdacNetlogonPolicy `
  -Force

# Create deployment script.
# Modern WDAC multiple-policy deployments commonly use the CodeIntegrity CiPolicies Active folder.
# Validate policy format and OS support in a pilot before enforcement.
@"
`$SourcePolicy = "\\$DomainFqdn\NETLOGON\WDAC\WDAC-Audit.cip"
`$TargetFolder = "C:\Windows\System32\CodeIntegrity\CiPolicies\Active"
`$TargetPolicy = Join-Path `$TargetFolder "WDAC-Audit.cip"
`$LogRoot = "C:\ProgramData\Corp\WDAC"
`$LogFile = Join-Path `$LogRoot "Deploy-WDAC-Audit.log"

New-Item -ItemType Directory -Force -Path `$TargetFolder,`$LogRoot | Out-Null

try {
    Copy-Item -Path `$SourcePolicy -Destination `$TargetPolicy -Force
    "`$(Get-Date -Format o) Copied WDAC audit policy from `$SourcePolicy to `$TargetPolicy" | Out-File `$LogFile -Append
}
catch {
    "`$(Get-Date -Format o) WDAC policy copy failed: `$(`$_.Exception.Message)" | Out-File `$LogFile -Append
    exit 1
}
"@ | Set-Content -Path $DeployScript -Encoding UTF8

# Create and link WDAC GPO if missing.
if (-not (Get-GPO -Name $WdacGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $WdacGpoName `
      -Comment "WDAC audit policy deployment for pilot computers"
}

$Inheritance = Get-GPInheritance -Target $PilotOU
if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $WdacGpoName)) {
    New-GPLink `
      -Name $WdacGpoName `
      -Target $PilotOU `
      -LinkEnabled Yes
}

Set-GPLink `
  -Name $WdacGpoName `
  -Target $PilotOU `
  -LinkEnabled Yes `
  -Enforced No

# Assign startup script through GPMC:
gpmc.msc

# GUI workflow:
# 1. Edit:
#      CORP-Workstation-WDAC-Audit
# 2. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Scripts
#      > Startup
# 3. PowerShell Scripts tab.
# 4. Add:
#      Deploy-WDAC-Audit.ps1
# 5. Place script in the GPO startup script folder or reference a safe SYSVOL path.
#
# After policy refresh, reboot the pilot client and validate CodeIntegrity logs.
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Client_RSOP_Validation_Skeleton
```powershell
# Run on the pilot test client.
# Purpose: validate SRP, AppLocker, and WDAC audit behavior.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\AppControl-Client"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm client identity.
hostname |
  Out-File "$ReportPath\hostname.txt"

whoami /all |
  Out-File "$ReportPath\whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$ReportPath\computer-info.txt"

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

# Refresh policy.
gpupdate /force |
  Tee-Object "$ReportPath\gpupdate-force.txt"

# Reboot after initial AppLocker service or WDAC deployment.
# Restart-Computer

# Validate applied GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-r.txt"

gpresult /h "$ReportPath\gpresult-app-control.html"
gpresult /x "$ReportPath\gpresult-app-control.xml"

# Validate Application Identity service.
Get-Service `
  -Name AppIDSvc `
  -ErrorAction SilentlyContinue |
  Select-Object Name,Status,StartType |
  Out-File "$ReportPath\application-identity-service.txt"

# Start service for immediate testing if policy has not started it yet.
Start-Service AppIDSvc -ErrorAction SilentlyContinue

# Validate effective AppLocker policy.
Get-AppLockerPolicy `
  -Effective `
  -Xml |
  Out-File "$ReportPath\applocker-effective.xml" `
  -ErrorAction SilentlyContinue

# Review AppLocker event logs.
$AppLockerLogs = @(
  "Microsoft-Windows-AppLocker/EXE and DLL",
  "Microsoft-Windows-AppLocker/MSI and Script",
  "Microsoft-Windows-AppLocker/Packaged app-Deployment",
  "Microsoft-Windows-AppLocker/Packaged app-Execution"
)

foreach ($Log in $AppLockerLogs) {
    Get-WinEvent `
      -LogName $Log `
      -MaxEvents 100 `
      -ErrorAction SilentlyContinue |
      Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
      Out-File "$ReportPath\$($Log.Replace('/','-')).txt"
}

# Validate WDAC policy file placement.
Get-ChildItem `
  -Path "C:\Windows\System32\CodeIntegrity" `
  -Recurse `
  -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,LastWriteTime |
  Out-File "$ReportPath\codeintegrity-policy-files.txt"

# Review Code Integrity logs.
Get-WinEvent `
  -LogName "Microsoft-Windows-CodeIntegrity/Operational" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\codeintegrity-operational-events.txt"

Get-WinEvent `
  -LogName "Microsoft-Windows-CodeIntegrity/Verbose" `
  -MaxEvents 200 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\codeintegrity-verbose-events.txt"

# Review Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 200 |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\group-policy-operational-events.txt"

Write-Host "Application control client validation complete."
Write-Host "Report path: $ReportPath"
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final reports and backups for app control GPOs.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoNames = @(
  "CORP-Workstation-AppLocker-Audit",
  "CORP-Workstation-WDAC-Audit",
  "CORP-Workstation-SRP-Legacy-Test"
)

$ReportPath = "C:\GPOPrep\Reports\AppControl"
$BackupPath = "C:\GPOPrep\Backup\AppControl"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-final-app-control.txt"

foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPPermission `
          -Name $GpoName `
          -All |
          Out-File "$ReportPath\$GpoName-permissions.txt"

        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-final.html"

        Get-GPOReport `
          -Name $GpoName `
          -ReportType Xml `
          -Path "$ReportPath\$GpoName-final.xml"

        Backup-GPO `
          -Name $GpoName `
          -Path $BackupPath `
          -Comment "Final backup after application control configuration"
    }
}

# Search reports for app control indicators.
Select-String `
  -Path "$ReportPath\*.xml" `
  -Pattern "AppLocker","Software Restriction","Safer","Application Control","Application Identity","WDAC","CodeIntegrity","Scripts","Executable","Windows Installer" |
  Out-File "$ReportPath\app-control-report-search.txt"

# Replication checks.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-app-control.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-app-control.txt"

Write-Host "Application control reports and backups complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<applocker-gpo-name>"` | Confirms AppLocker GPO exists | GPO object returns |
| `Get-GPO -Name "<wdac-gpo-name>"` | Confirms WDAC GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-ou-dn>"` | Confirms app control GPO links | GPOs appear in pilot OU link list |
| `Get-GPPermission -Name "<gpo-name>" -All` | Confirms target computers can apply GPO | Expected permissions appear |
| `gpupdate /force` | Forces computer policy refresh | Policy update completes |
| `Restart-Computer` | Refreshes startup and code integrity policy state | Client restarts |
| `gpresult /scope computer /r` | Shows applied computer GPOs | App control GPO appears |
| `gpresult /h C:\GPOPrep\Reports\gpresult-app-control.html` | Exports full RSOP report | HTML report exists |
| `Get-Service AppIDSvc` | Validates Application Identity service | Service exists and expected startup state is visible |
| `Start-Service AppIDSvc` | Starts Application Identity service for testing | Service starts |
| `Get-AppLockerPolicy -Effective` | Shows effective AppLocker policy | Policy object returns |
| `Get-AppLockerPolicy -Effective -Xml` | Exports effective AppLocker XML | XML output returns |
| `Test-AppLockerPolicy -XmlPolicy <xml> -Path <file> -User <user>` | Tests a file against AppLocker policy | Allowed or denied result returns |
| `Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 100` | Reviews EXE and DLL AppLocker events | Audit or enforcement events appear |
| `Get-WinEvent -LogName "Microsoft-Windows-AppLocker/MSI and Script" -MaxEvents 100` | Reviews MSI and Script AppLocker events | Audit or enforcement events appear |
| `New-CIPolicy -FilePath <xml> -Level Publisher -Fallback Hash -ScanPath C:\` | Creates WDAC XML policy | XML policy file is created |
| `Set-RuleOption -FilePath <xml> -Option 3` | Enables WDAC audit option | Policy remains audit mode |
| `ConvertFrom-CIPolicy -XmlFilePath <xml> -BinaryFilePath <cip>` | Converts WDAC XML to binary | `.cip` file is created |
| `Get-ChildItem C:\Windows\System32\CodeIntegrity -Recurse` | Checks WDAC policy placement | Policy files are visible |
| `Get-WinEvent -LogName "Microsoft-Windows-CodeIntegrity/Operational" -MaxEvents 100` | Reviews WDAC CodeIntegrity events | Audit events appear |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path <path>` | Exports intended GPO report | Report exists |
| `Backup-GPO -Name "<gpo-name>" -Path "<backup-path>"` | Backs up app control GPO | Backup completes |
| `repadmin /replsummary` | Checks AD replication | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks SYSVOL and DC health | Tests pass |

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely roll back SRP, AppLocker, and WDAC pilot deployment.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoNames = @(
  "CORP-Workstation-AppLocker-Audit",
  "CORP-Workstation-WDAC-Audit",
  "CORP-Workstation-SRP-Legacy-Test"
)

$ReportPath = "C:\GPOPrep\Reports\AppControl-Rollback"
$BackupPath = "C:\GPOPrep\Backup\AppControl-Rollback"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture current state before rollback.
foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-before-rollback.html"

        Backup-GPO `
          -Name $GpoName `
          -Path $BackupPath `
          -Comment "Before application control rollback"
    }
}

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-rollback.txt"

# Rollback option 1:
# Disable app control GPO links.
foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Set-GPLink `
          -Name $GpoName `
          -Target $PilotOU `
          -LinkEnabled No `
          -ErrorAction SilentlyContinue
    }
}

# Rollback option 2:
# Use GPMC to set AppLocker collections back to Audit only or remove rules.
gpmc.msc

# GUI rollback workflow:
# 1. Edit AppLocker GPO.
# 2. Go to AppLocker > Properties.
# 3. Set collections to Audit only or Not configured.
# 4. Remove test block rules.
# 5. Export report.
# 6. Refresh pilot client and validate.

# Rollback option 3:
# Remove WDAC deployed audit policy file using a controlled startup script or manual pilot cleanup.
# Run on pilot client only after disabling WDAC GPO link:
# Remove-Item "C:\Windows\System32\CodeIntegrity\CiPolicies\Active\WDAC-Audit.cip" -Force -ErrorAction SilentlyContinue
# Restart-Computer

# Rollback option 4:
# Remove disposable lab GPOs only after links are disabled and clients are validated.
# foreach ($GpoName in $GpoNames) {
#     if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
#         Remove-GPO -Name $GpoName
#     }
# }

# Capture state after rollback.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-rollback.txt"

Write-Host "Application control rollback workflow complete."
Write-Host "On pilot client: run gpupdate /force, reboot, and review AppLocker plus CodeIntegrity logs."
```

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| App control GPO does not apply | Computer not in pilot OU or filtering blocks it | `gpresult /scope computer /r`; `Get-GPInheritance` | Move computer to pilot OU or fix filtering |
| AppLocker policy does not work | Application Identity service not running | `Get-Service AppIDSvc` | Configure service to Automatic and start it |
| AppLocker events missing | Policy not applied, service stopped, or wrong log checked | `Get-AppLockerPolicy -Effective`; Event Viewer logs | Fix policy scope and service |
| AppLocker blocks approved app | Missing allow rule or rule too narrow | AppLocker event logs | Add publisher, path, or hash allow rule |
| AppLocker audit shows too many events | Default rules missing or DLL audit too noisy | AppLocker logs and policy report | Create default rules and defer DLL rules |
| AppLocker script rule blocks admin scripts | Script rules enforced too early | MSI and Script log | Return to audit mode or add signed script rules |
| SRP blocks too much | Default security level set to Disallowed without full allowlist | GPO report and client behavior | Disable SRP link or restore backup |
| SRP does nothing | SRP not linked or rule does not match | `gpresult /h`; SRP GPO report | Fix link, rule path, or target scope |
| WDAC policy does not load | Policy file not copied to correct path or reboot missing | CodeIntegrity logs and file check | Fix deployment path and reboot |
| WDAC audit events missing | Policy not loaded or no relevant execution tested | CodeIntegrity Operational log | Confirm policy file, reboot, and test apps |
| WDAC blocks apps unexpectedly | Enforcement policy deployed instead of audit policy | CodeIntegrity logs and WDAC policy options | Roll back policy and redeploy audit mode |
| WDAC prevents boot or login | Enforced policy too restrictive | Recovery console or safe rollback path | Remove policy through recovery method and restore audit baseline |
| Policy works on one client but not another | Different OU, OS, installed apps, or replication state | Compare `gpresult /h` reports | Normalize scope and inventory differences |
| GPMC report missing AppLocker settings | Wrong GPO edited or stale report | `Get-GPOReport -Name "<gpo>"` | Edit correct GPO and export fresh report |
| Client gets old policy | Replication delay or stale client cache | `repadmin /replsummary`; `gpupdate /force` | Fix replication and refresh client |
| Computer group filtering not reflected | Computer token stale | `gpresult /scope computer /r` | Reboot computer |
| User group filtering not reflected | User token stale | `whoami /groups` | Sign out and sign back in |
| Application allowed in audit mode | Audit mode logs only and does not block | Event log mode review | Switch to enforcement only after approval |
| App allowed because default path rule is broad | Default rules allow Program Files or Windows | AppLocker policy report | Use publisher rules or narrower design |
| DLL rules create performance or noise issues | DLL collection enabled too early | AppLocker EXE and DLL log | Disable DLL rules until mature baseline |
| WDAC policy source is untrusted or unknown | No clean reference image or package control | Policy creation notes and hashes | Rebuild policy from known-good reference |
| Rollback does not remove WDAC behavior | Policy file remains or reboot missing | CodeIntegrity folder and logs | Remove policy file and reboot |
| Remote troubleshooting blocked | Firewall or app control blocks admin tools | RDP/WinRM tests and event logs | Use console, disable pilot link, or restore backup |

# 19_Configure_Software_Restriction_AppLocker_And_WDAC_GPOs_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before app control rollout |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which computers receive app control policy |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Adds optional OS targeting for AppLocker or WDAC pilots |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Provides computer-side policy foundation |
| `11_Configure_Security_Baseline_GPO_Settings.md` | Security baseline companion workbook |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md` | Ensures remote admin access remains available during app control testing |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Provides backup and restore path before risky app control changes |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Validates AppLocker and WDAC GPO application |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Diagnoses scope, filtering, SYSVOL, and client processing failures |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md` | Ensures current Administrative Templates are available |
| `18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers.md` | Related for kiosk and RDS application control behavior |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms domain infrastructure before app control deployment |
| `20_Configure_Advanced_Auditing_And_Event_Log_GPO_Settings.md` | Next workbook for collecting security event evidence |