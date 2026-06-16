17_Configure_ADMX_Central_Store_And_Policy_Definitions.md
# 17_Configure_ADMX_Central_Store_And_Policy_Definitions

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Index
17_Configure_ADMX_Central_Store_And_Policy_Definitions.md
17_Configure_ADMX_Central_Store_And_Policy_Definitions
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Source_Basis
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Mental_Model
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Planning_Table
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Component_Map
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Configuration_Checklist
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Precheck_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Central_Store_Backup_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Create_Central_Store_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Copy_ADMX_ADML_From_Reference_Client_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Copy_ADMX_ADML_From_Downloaded_Package_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Validate_GPMC_Template_Loading_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_ADMX_Version_Inventory_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Report_And_Backup_Skeleton
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Verification_Commands
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Rollback
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Failure_Checks
17_Configure_ADMX_Central_Store_And_Policy_Definitions_Related_Labs

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Administrative Templates and ADMX files | Understanding ADMX and ADML policy definition files |
| Microsoft Learn | Group Policy Central Store | Creating `PolicyDefinitions` in SYSVOL |
| Microsoft Learn | Group Policy Management Console | Validating that GPMC loads policy definitions from the Central Store |
| Microsoft Learn | Group Policy Administrative Templates | Editing ADMX-backed settings in GPOs |
| Microsoft Learn | GroupPolicy PowerShell module | Reporting and backing up GPOs before and after template changes |
| Windows client reference system | Local `C:\Windows\PolicyDefinitions` | Source for ADMX and language-specific ADML files |
| SYSVOL and DFSR | Replicated central policy definition location | Making templates available to all GPMC editors in the domain |
| Windows Server operational practice | Back up existing Central Store before updating templates | Preventing broken GPMC display and lost custom templates |

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Mental_Model
| Concept | Operational Meaning |
|---|---|
| ADMX | Language-neutral Administrative Template file that defines policy setting structure |
| ADML | Language-specific resource file used by ADMX files for display text |
| Administrative Template | GPO editor category backed by ADMX and ADML files |
| Local PolicyDefinitions | Local template store at `C:\Windows\PolicyDefinitions` |
| Central Store | Domain-wide template store at `\\domain\SYSVOL\domain\Policies\PolicyDefinitions` |
| PolicyDefinitions folder | Folder name required for the Central Store |
| Language folder | Folder such as `en-US` containing ADML files |
| Reference client | Windows machine used as known source for policy definitions |
| GPMC template source | GPMC uses Central Store when present, otherwise local templates |
| Template mismatch | ADMX exists without required ADML, or ADML version does not match ADMX |
| Custom ADMX | Vendor or organization-provided template files such as Chrome, Edge, Office, OneDrive, or security products |
| Version inventory | Record of ADMX file names, hashes, and timestamps before and after changes |
| GPO setting storage | Actual GPO settings are stored in GPO files and AD metadata, not in the ADMX files |
| First rule | ADMX files control what GPMC can display and edit, not whether the client can read an already configured registry policy |
| Blunt rule | Never overwrite the Central Store without a backup and file inventory |

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Central Store path | `\\corp.local\SYSVOL\corp.local\Policies\PolicyDefinitions` | `<central-store-path>` |
| Reference client | `WIN11-REF01` | `<reference-client>` |
| Reference local PolicyDefinitions path | `C:\Windows\PolicyDefinitions` | `<reference-policydefinitions-path>` |
| Language folder | `en-US` | `<language-folder>` |
| Template source | Reference client or downloaded ADMX package | `<template-source>` |
| Download extraction path | `C:\GPOPrep\ADMX_Source` | `<downloaded-admx-path>` |
| Backup path | `C:\GPOPrep\Backup\ADMX-CentralStore` | `<backup-path>` |
| Report path | `C:\GPOPrep\Reports\ADMX-CentralStore` | `<report-path>` |
| Existing Central Store | Yes or No | `<yes-no>` |
| Custom ADMX files | Edge, Chrome, Office, OneDrive, vendor templates | `<custom-admx-list>` |
| Update strategy | Full copy, staged copy, or selective copy | `<update-strategy>` |
| GPMC validation host | `MGMT01` | `<management-host>` |
| Test GPO | `CORP-ADMX-Validation-Test` | `<test-gpo-name>` |
| Rollback method | Restore previous `PolicyDefinitions` folder | `<rollback-plan>` |

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Component_Map
| Component | Path | Purpose | Validation |
|---|---|---|---|
| Central Store root | `\\corp.local\SYSVOL\corp.local\Policies\PolicyDefinitions` | Domain-wide ADMX store | `Test-Path` returns True |
| ADMX files | `PolicyDefinitions\*.admx` | Language-neutral policy definitions | Files exist and are readable |
| ADML files | `PolicyDefinitions\en-US\*.adml` | Language-specific display strings | Matching ADML files exist |
| Local templates | `C:\Windows\PolicyDefinitions` | Local template source on reference computer | Files exist on reference client |
| Custom vendor ADMX | `PolicyDefinitions\<vendor>.admx` | Adds vendor policy nodes to GPMC | Vendor node appears in GPMC |
| Custom vendor ADML | `PolicyDefinitions\en-US\<vendor>.adml` | Display text for custom vendor policy | No GPMC parse error |
| GPMC editor | `gpmc.msc` | GUI validation point | Administrative Templates open without errors |
| GPO report | `Get-GPOReport` output | Confirms configured settings remain visible | HTML/XML exports succeed |
| ADMX inventory | CSV file of names, hashes, timestamps | Change control and rollback evidence | Inventory exists before and after |
| Central Store backup | Backup copy of previous `PolicyDefinitions` | Recovery path | Backup folder exists |

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 6 | Confirm GPMC opens | Management Host | `gpmc.msc` | GPMC opens |
| 7 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\ADMX-CentralStore` | Report path exists |
| 8 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup\ADMX-CentralStore` | Backup path exists |
| 9 | Check whether Central Store exists | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies\PolicyDefinitions"` | Existing state is known |
| 10 | Inventory existing Central Store | Management Host | `Get-ChildItem "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies\PolicyDefinitions" -Recurse` | Existing files are recorded |
| 11 | Back up existing Central Store if present | Management Host | `Copy-Item "<central-store-path>" "<backup-path>\PolicyDefinitions-before" -Recurse` | Existing Central Store backup exists |
| 12 | Confirm reference client source path | Reference Client | `Test-Path C:\Windows\PolicyDefinitions` | Local PolicyDefinitions source exists |
| 13 | Confirm language folder exists | Reference Client | `Test-Path C:\Windows\PolicyDefinitions\en-US` | Language folder exists |
| 14 | Create Central Store folder if missing | Management Host | `New-Item -ItemType Directory -Force -Path "<central-store-path>"` | Central Store root exists |
| 15 | Copy ADMX files | Management Host | `Copy-Item C:\Windows\PolicyDefinitions\*.admx "<central-store-path>" -Force` | ADMX files copied |
| 16 | Copy ADML language folder | Management Host | `Copy-Item C:\Windows\PolicyDefinitions\en-US "<central-store-path>" -Recurse -Force` | ADML files copied |
| 17 | Copy custom vendor ADMX files if required | Management Host | `Copy-Item "<vendor-source>\*.admx" "<central-store-path>" -Force` | Custom ADMX files copied |
| 18 | Copy custom vendor ADML files if required | Management Host | `Copy-Item "<vendor-source>\en-US\*.adml" "<central-store-path>\en-US" -Force` | Custom ADML files copied |
| 19 | Inventory updated Central Store | Management Host | `Get-ChildItem "<central-store-path>" -Recurse` | Updated files are recorded |
| 20 | Validate ADMX and ADML pairing | Management Host | `Compare ADMX names with language ADML names` | Missing ADML files are identified |
| 21 | Open GPMC editor | Management Host | `GPMC > Group Policy Objects > right-click test GPO > Edit` | Editor opens |
| 22 | Validate Administrative Templates loads | Management Host | `Computer Configuration > Policies > Administrative Templates` | No parse or namespace errors appear |
| 23 | Validate User Administrative Templates loads | Management Host | `User Configuration > Policies > Administrative Templates` | No parse or namespace errors appear |
| 24 | Validate expected new policy nodes | Management Host | `Search GPMC for expected Windows or vendor policy nodes` | Expected policy categories appear |
| 25 | Export test GPO report | Management Host | `Get-GPOReport -Name "<test-gpo-name>" -ReportType Html -Path "<report-path>\<test-gpo-name>.html"` | GPO report succeeds |
| 26 | Check replication summary | Domain Controller | `repadmin /replsummary` | No unexpected failures |
| 27 | Check SYSVOL health | Domain Controller | `dcdiag /test:sysvolcheck /test:advertising` | Tests pass |
| 28 | Confirm Central Store path from each DC | Management Host | `Test-Path "\\<dc>\SYSVOL\<domain-fqdn>\Policies\PolicyDefinitions"` | Central Store exists on each DC after replication |
| 29 | Document final inventory | Operator | `Record template source, date, files, backup, custom templates, validation host, and result` | Central Store change is documented |

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate domain, SYSVOL, GPMC tooling, and current Central Store state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$BasePath = "C:\GPOPrep"
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$ReportPath = "$BasePath\Reports\ADMX-CentralStore-$TimeStamp"
$BackupPath = "$BasePath\Backup\ADMX-CentralStore-$TimeStamp"

$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm domain identity.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName,PDCEmulator |
  Out-File "$ReportPath\domain-info.txt"

# Confirm DC inventory.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog |
  Export-Csv "$ReportPath\domain-controllers.csv" -NoTypeInformation

# Confirm SYSVOL and NETLOGON.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-policies-path-check.txt"

Test-Path "\\$DomainFqdn\NETLOGON" |
  Out-File "$ReportPath\netlogon-path-check.txt"

# Confirm current Central Store state.
$CentralStoreExists = Test-Path $CentralStorePath

[PSCustomObject]@{
    CentralStorePath = $CentralStorePath
    Exists = $CentralStoreExists
} | Export-Csv "$ReportPath\central-store-state.csv" -NoTypeInformation

# Inventory current Central Store if present.
if ($CentralStoreExists) {
    Get-ChildItem `
      -Path $CentralStorePath `
      -Recurse `
      -ErrorAction SilentlyContinue |
      Select-Object FullName,Name,Extension,Length,LastWriteTime |
      Export-Csv "$ReportPath\central-store-existing-file-inventory.csv" -NoTypeInformation
}

# Confirm GPO report capability.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-admx-central-store.csv" -NoTypeInformation

Write-Host "Precheck complete."
Write-Host "Central Store path: $CentralStorePath"
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Central_Store_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: back up existing Central Store before any ADMX or ADML changes.

Import-Module ActiveDirectory

$DomainFqdn = (Get-ADDomain).DNSRoot

$BasePath = "C:\GPOPrep"
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$ReportPath = "$BasePath\Reports\ADMX-CentralStore-Backup-$TimeStamp"
$BackupRoot = "$BasePath\Backup\ADMX-CentralStore-Backup-$TimeStamp"

$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"
$CentralStoreBackupPath = "$BackupRoot\PolicyDefinitions"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupRoot

if (Test-Path $CentralStorePath) {
    # Inventory before backup.
    Get-ChildItem `
      -Path $CentralStorePath `
      -Recurse |
      Select-Object FullName,Name,Extension,Length,LastWriteTime |
      Export-Csv "$ReportPath\central-store-before-backup-inventory.csv" -NoTypeInformation

    # Hash inventory before backup.
    Get-ChildItem `
      -Path $CentralStorePath `
      -Recurse `
      -File |
      Get-FileHash |
      Select-Object Path,Algorithm,Hash |
      Export-Csv "$ReportPath\central-store-before-backup-hashes.csv" -NoTypeInformation

    # Backup the Central Store.
    Copy-Item `
      -Path $CentralStorePath `
      -Destination $CentralStoreBackupPath `
      -Recurse `
      -Force

    # Inventory backup.
    Get-ChildItem `
      -Path $CentralStoreBackupPath `
      -Recurse |
      Select-Object FullName,Name,Extension,Length,LastWriteTime |
      Export-Csv "$ReportPath\central-store-backup-inventory.csv" -NoTypeInformation

    Write-Host "Central Store backed up to: $CentralStoreBackupPath"
}
else {
    Write-Host "No existing Central Store found. Nothing to back up."
}

Write-Host "Report path: $ReportPath"
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Create_Central_Store_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create the Central Store folder structure.

Import-Module ActiveDirectory

$DomainFqdn = (Get-ADDomain).DNSRoot

$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"
$Language = "en-US"
$LanguagePath = Join-Path $CentralStorePath $Language

# Create Central Store root and language folder.
New-Item `
  -ItemType Directory `
  -Force `
  -Path $CentralStorePath,$LanguagePath

# Confirm paths.
Test-Path $CentralStorePath
Test-Path $LanguagePath

# List initial state.
Get-ChildItem `
  -Path $CentralStorePath `
  -Force |
  Select-Object FullName,Name,Mode,LastWriteTime

Write-Host "Central Store root: $CentralStorePath"
Write-Host "Language folder: $LanguagePath"
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Copy_ADMX_ADML_From_Reference_Client_Skeleton
```powershell
# Run on the reference Windows client or management host.
# Purpose: copy ADMX and ADML files from a trusted Windows reference system to the Central Store.
# Use a reference system that matches the Windows version you intend to manage.

Import-Module ActiveDirectory

$DomainFqdn = (Get-ADDomain).DNSRoot

$SourcePolicyDefinitions = "C:\Windows\PolicyDefinitions"
$SourceLanguage = "C:\Windows\PolicyDefinitions\en-US"

$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"
$CentralStoreLanguage = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions\en-US"

$BasePath = "C:\GPOPrep"
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ReportPath = "$BasePath\Reports\ADMX-Copy-Reference-$TimeStamp"

New-Item -ItemType Directory -Force -Path $ReportPath,$CentralStorePath,$CentralStoreLanguage

# Validate source.
Test-Path $SourcePolicyDefinitions |
  Out-File "$ReportPath\source-policydefinitions-check.txt"

Test-Path $SourceLanguage |
  Out-File "$ReportPath\source-language-check.txt"

# Inventory source ADMX and ADML files.
Get-ChildItem `
  -Path $SourcePolicyDefinitions `
  -Filter "*.admx" `
  -File |
  Select-Object Name,FullName,Length,LastWriteTime |
  Export-Csv "$ReportPath\source-admx-inventory.csv" -NoTypeInformation

Get-ChildItem `
  -Path $SourceLanguage `
  -Filter "*.adml" `
  -File |
  Select-Object Name,FullName,Length,LastWriteTime |
  Export-Csv "$ReportPath\source-adml-inventory.csv" -NoTypeInformation

# Copy ADMX files.
Copy-Item `
  -Path "$SourcePolicyDefinitions\*.admx" `
  -Destination $CentralStorePath `
  -Force

# Copy ADML files.
Copy-Item `
  -Path "$SourceLanguage\*.adml" `
  -Destination $CentralStoreLanguage `
  -Force

# Inventory destination.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Select-Object Name,FullName,Extension,Length,LastWriteTime |
  Export-Csv "$ReportPath\central-store-after-copy-inventory.csv" -NoTypeInformation

# Hash destination.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Get-FileHash |
  Select-Object Path,Algorithm,Hash |
  Export-Csv "$ReportPath\central-store-after-copy-hashes.csv" -NoTypeInformation

Write-Host "ADMX and ADML copy from reference client complete."
Write-Host "Central Store: $CentralStorePath"
Write-Host "Report path: $ReportPath"
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Copy_ADMX_ADML_From_Downloaded_Package_Skeleton
```powershell
# Run on a domain-joined management host.
# Purpose: copy ADMX and ADML files from an extracted Microsoft or vendor ADMX package.
# Example source path assumes the ADMX package was already downloaded and extracted.

Import-Module ActiveDirectory

$DomainFqdn = (Get-ADDomain).DNSRoot

$ExtractedAdmxRoot = "C:\GPOPrep\ADMX_Source"
$SourcePolicyDefinitions = "$ExtractedAdmxRoot\PolicyDefinitions"
$SourceLanguage = "$SourcePolicyDefinitions\en-US"

$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"
$CentralStoreLanguage = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions\en-US"

$BasePath = "C:\GPOPrep"
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ReportPath = "$BasePath\Reports\ADMX-Copy-Package-$TimeStamp"

New-Item -ItemType Directory -Force -Path $ReportPath,$CentralStorePath,$CentralStoreLanguage

# Validate source package structure.
Test-Path $SourcePolicyDefinitions |
  Out-File "$ReportPath\package-policydefinitions-check.txt"

Test-Path $SourceLanguage |
  Out-File "$ReportPath\package-language-check.txt"

# Inventory source.
Get-ChildItem `
  -Path $SourcePolicyDefinitions `
  -Filter "*.admx" `
  -File |
  Select-Object Name,FullName,Length,LastWriteTime |
  Export-Csv "$ReportPath\package-source-admx-inventory.csv" -NoTypeInformation

Get-ChildItem `
  -Path $SourceLanguage `
  -Filter "*.adml" `
  -File |
  Select-Object Name,FullName,Length,LastWriteTime |
  Export-Csv "$ReportPath\package-source-adml-inventory.csv" -NoTypeInformation

# Copy ADMX files.
Copy-Item `
  -Path "$SourcePolicyDefinitions\*.admx" `
  -Destination $CentralStorePath `
  -Force

# Copy ADML files for the selected language.
Copy-Item `
  -Path "$SourceLanguage\*.adml" `
  -Destination $CentralStoreLanguage `
  -Force

# Optional custom vendor templates:
# Copy-Item "C:\GPOPrep\VendorADMX\*.admx" $CentralStorePath -Force
# Copy-Item "C:\GPOPrep\VendorADMX\en-US\*.adml" $CentralStoreLanguage -Force

# Inventory destination.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Select-Object Name,FullName,Extension,Length,LastWriteTime |
  Export-Csv "$ReportPath\central-store-after-package-copy-inventory.csv" -NoTypeInformation

Write-Host "ADMX and ADML copy from package complete."
Write-Host "Central Store: $CentralStorePath"
Write-Host "Report path: $ReportPath"
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Validate_GPMC_Template_Loading_Skeleton
```powershell
# Run on a domain-joined management host.
# Purpose: validate that GPMC can load Administrative Templates from the Central Store.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainFqdn = (Get-ADDomain).DNSRoot
$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"

$TestGpoName = "CORP-ADMX-Validation-Test"
$ReportPath = "C:\GPOPrep\Reports\ADMX-CentralStore-Validation"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm Central Store path.
Test-Path $CentralStorePath |
  Out-File "$ReportPath\central-store-check.txt"

# Create test GPO if missing.
if (-not (Get-GPO -Name $TestGpoName -ErrorAction SilentlyContinue)) {
    New-GPO `
      -Name $TestGpoName `
      -Comment "Temporary GPO for ADMX Central Store validation"
}

# Export empty or current test GPO report.
Get-GPOReport `
  -Name $TestGpoName `
  -ReportType Html `
  -Path "$ReportPath\$TestGpoName-before-validation.html"

# Open GPMC for manual validation.
gpmc.msc

# Manual validation workflow:
# 1. Open Group Policy Management Console.
# 2. Expand the domain.
# 3. Expand Group Policy Objects.
# 4. Right-click:
#      CORP-ADMX-Validation-Test
# 5. Select Edit.
# 6. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
# 7. Confirm templates load without namespace, parse, or missing resource errors.
# 8. Go to:
#      User Configuration
#      > Policies
#      > Administrative Templates
# 9. Confirm templates load without errors.
# 10. Confirm expected policy nodes exist.
# 11. Close editor without making changes unless testing a specific setting.

# Export report after validation.
Get-GPOReport `
  -Name $TestGpoName `
  -ReportType Html `
  -Path "$ReportPath\$TestGpoName-after-validation.html"

Write-Host "GPMC validation workflow complete."
Write-Host "If GPMC showed ADMX or ADML errors, capture screenshots and check missing/mismatched files."
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_ADMX_Version_Inventory_Skeleton
```powershell
# Run on a domain-joined management host.
# Purpose: create an inventory of ADMX and ADML files, including file hashes for change control.

Import-Module ActiveDirectory

$DomainFqdn = (Get-ADDomain).DNSRoot
$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"

$ReportPath = "C:\GPOPrep\Reports\ADMX-CentralStore-Inventory"
New-Item -ItemType Directory -Force -Path $ReportPath

# File inventory.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Select-Object FullName,Name,Extension,Length,CreationTime,LastWriteTime |
  Export-Csv "$ReportPath\central-store-file-inventory.csv" -NoTypeInformation

# Hash inventory.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Get-FileHash |
  Select-Object Path,Algorithm,Hash |
  Export-Csv "$ReportPath\central-store-file-hashes.csv" -NoTypeInformation

# ADMX files only.
Get-ChildItem `
  -Path $CentralStorePath `
  -Filter "*.admx" `
  -File |
  Select-Object Name,FullName,Length,LastWriteTime |
  Export-Csv "$ReportPath\central-store-admx-files.csv" -NoTypeInformation

# ADML files only.
Get-ChildItem `
  -Path $CentralStorePath `
  -Filter "*.adml" `
  -Recurse `
  -File |
  Select-Object Name,FullName,Length,LastWriteTime |
  Export-Csv "$ReportPath\central-store-adml-files.csv" -NoTypeInformation

# Identify ADMX files without matching en-US ADML files.
$Language = "en-US"
$LanguagePath = Join-Path $CentralStorePath $Language

$AdmxBaseNames = Get-ChildItem -Path $CentralStorePath -Filter "*.admx" -File |
  Select-Object -ExpandProperty BaseName

$AdmlBaseNames = Get-ChildItem -Path $LanguagePath -Filter "*.adml" -File |
  Select-Object -ExpandProperty BaseName

$MissingAdml = $AdmxBaseNames |
  Where-Object { $_ -notin $AdmlBaseNames }

$MissingAdml |
  Out-File "$ReportPath\admx-without-matching-en-us-adml.txt"

# Identify ADML files without matching ADMX files.
$OrphanAdml = $AdmlBaseNames |
  Where-Object { $_ -notin $AdmxBaseNames }

$OrphanAdml |
  Out-File "$ReportPath\adml-without-matching-admx.txt"

Write-Host "ADMX Central Store inventory complete."
Write-Host "Report path: $ReportPath"
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final Central Store inventory, GPO report validation, and SYSVOL health.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot

$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"

$TestGpoName = "CORP-ADMX-Validation-Test"

$BasePath = "C:\GPOPrep"
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$ReportPath = "$BasePath\Reports\ADMX-CentralStore-Final-$TimeStamp"
$BackupPath = "$BasePath\Backup\ADMX-CentralStore-Final-$TimeStamp"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Final Central Store inventory.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Select-Object FullName,Name,Extension,Length,LastWriteTime |
  Export-Csv "$ReportPath\central-store-final-file-inventory.csv" -NoTypeInformation

# Final Central Store hashes.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Get-FileHash |
  Select-Object Path,Algorithm,Hash |
  Export-Csv "$ReportPath\central-store-final-file-hashes.csv" -NoTypeInformation

# Backup final Central Store.
Copy-Item `
  -Path $CentralStorePath `
  -Destination "$BackupPath\PolicyDefinitions" `
  -Recurse `
  -Force

# Export test GPO report to prove GPMC and reporting still work.
if (Get-GPO -Name $TestGpoName -ErrorAction SilentlyContinue) {
    Get-GPOReport `
      -Name $TestGpoName `
      -ReportType Html `
      -Path "$ReportPath\$TestGpoName-final.html"

    Get-GPOReport `
      -Name $TestGpoName `
      -ReportType Xml `
      -Path "$ReportPath\$TestGpoName-final.xml"
}

# AD and SYSVOL health.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-admx.txt"

dcdiag /test:sysvolcheck /test:advertising /test:netlogons |
  Out-File "$ReportPath\dcdiag-sysvol-after-admx.txt"

# Validate Central Store path per DC.
$Dcs = Get-ADDomainController -Filter * | Select-Object -ExpandProperty HostName

foreach ($Dc in $Dcs) {
    [PSCustomObject]@{
        DC = $Dc
        Path = "\\$Dc\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"
        Exists = Test-Path "\\$Dc\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"
    } | Export-Csv "$ReportPath\central-store-path-by-dc.csv" -NoTypeInformation -Append
}

Write-Host "Final ADMX Central Store report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Confirms SYSVOL Policies path exists | Returns True |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies\PolicyDefinitions"` | Confirms Central Store exists | Returns True |
| `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies\PolicyDefinitions\en-US"` | Confirms language folder exists | Returns True |
| `Get-ChildItem "<central-store-path>" -Filter "*.admx"` | Lists ADMX files | ADMX files appear |
| `Get-ChildItem "<central-store-path>\en-US" -Filter "*.adml"` | Lists ADML files | ADML files appear |
| `Get-ChildItem "<central-store-path>" -Recurse -File \| Get-FileHash` | Creates file hash inventory | Hashes return |
| `gpmc.msc` | Opens GPMC | Console opens |
| `GPMC > Edit GPO > Administrative Templates` | Validates templates load | No ADMX/ADML errors appear |
| `Get-GPOReport -Name "<test-gpo-name>" -ReportType Html -Path "<report-path>\<test-gpo-name>.html"` | Validates GPO reporting still works | HTML report exists |
| `Get-GPOReport -Name "<test-gpo-name>" -ReportType Xml -Path "<report-path>\<test-gpo-name>.xml"` | Validates XML report generation | XML report exists |
| `repadmin /replsummary` | Checks AD replication | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising /test:netlogons` | Checks DC and SYSVOL state | Tests pass |
| `Get-SmbShare -Name SYSVOL,NETLOGON` | Confirms DC shares | Shares exist |
| `Get-Service DFSR` | Confirms DFSR state on DCs | Service is running |
| `Get-WinEvent -LogName "DFS Replication" -MaxEvents 100` | Reviews SYSVOL replication health | No unexpected critical errors |
| `Test-Path "\\<dc>\SYSVOL\<domain-fqdn>\Policies\PolicyDefinitions"` | Confirms Central Store appears on each DC | Returns True after replication |
| `Select-String -Path "<report-path>\*.txt" -Pattern "error","missing","namespace"` | Searches report notes for problems | No unresolved issues |

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: restore the previous Central Store if new ADMX or ADML files break GPMC.

Import-Module ActiveDirectory

$DomainFqdn = (Get-ADDomain).DNSRoot

$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqDN\Policies\PolicyDefinitions"

# Correct variable if needed.
$CentralStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions"

$BackupPolicyDefinitions = "C:\GPOPrep\Backup\ADMX-CentralStore-Backup-20260613-120000\PolicyDefinitions"

$ReportPath = "C:\GPOPrep\Reports\ADMX-CentralStore-Rollback"
New-Item -ItemType Directory -Force -Path $ReportPath

# Capture broken state before rollback.
if (Test-Path $CentralStorePath) {
    Get-ChildItem `
      -Path $CentralStorePath `
      -Recurse `
      -File |
      Select-Object FullName,Name,Extension,Length,LastWriteTime |
      Export-Csv "$ReportPath\central-store-before-rollback-inventory.csv" -NoTypeInformation

    Get-ChildItem `
      -Path $CentralStorePath `
      -Recurse `
      -File |
      Get-FileHash |
      Select-Object Path,Algorithm,Hash |
      Export-Csv "$ReportPath\central-store-before-rollback-hashes.csv" -NoTypeInformation
}

# Validate backup exists.
if (-not (Test-Path $BackupPolicyDefinitions)) {
    Write-Host "Backup path not found: $BackupPolicyDefinitions"
    Write-Host "Stop and locate the correct backup before continuing."
    return
}

# Rollback option 1:
# Rename current Central Store and restore backup.
$TimeStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BrokenStorePath = "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies\PolicyDefinitions-Broken-$TimeStamp"

if (Test-Path $CentralStorePath) {
    Rename-Item `
      -Path $CentralStorePath `
      -NewName "PolicyDefinitions-Broken-$TimeStamp"
}

Copy-Item `
  -Path $BackupPolicyDefinitions `
  -Destination $CentralStorePath `
  -Recurse `
  -Force

# Capture restored state.
Get-ChildItem `
  -Path $CentralStorePath `
  -Recurse `
  -File |
  Select-Object FullName,Name,Extension,Length,LastWriteTime |
  Export-Csv "$ReportPath\central-store-after-rollback-inventory.csv" -NoTypeInformation

# Validate GPMC manually after rollback.
gpmc.msc

Write-Host "Central Store rollback complete."
Write-Host "Restored Central Store: $CentralStorePath"
Write-Host "Broken store preserved as: $BrokenStorePath"
Write-Host "Validate Administrative Templates in GPMC."
```

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Administrative Templates missing in GPMC | Central Store exists but is empty or incomplete | `Get-ChildItem "<central-store-path>"` | Copy ADMX and ADML files into Central Store |
| GPMC shows namespace error | Conflicting or mismatched ADMX files | Note error file name, inventory ADMX files | Restore backup or replace conflicting ADMX set |
| GPMC shows missing resource error | ADMX file exists but matching ADML is missing | Check `PolicyDefinitions\en-US\<file>.adml` | Copy matching ADML file |
| GPMC works on one admin machine but not another | One machine uses Central Store and another uses local templates, or console cache issue | Confirm Central Store path and GPMC error details | Standardize Central Store and reopen GPMC |
| New Windows policy settings do not appear | Old ADMX files in Central Store | Check ADMX inventory timestamps | Update Central Store from approved source |
| Vendor policy node missing | Vendor ADMX or ADML not copied | Check vendor file names in Central Store | Copy vendor ADMX and matching ADML |
| GPO setting still applies even after ADMX removed | ADMX only controls editor display, not existing registry policy application | `gpresult /h`; registry check | Remove or change the configured GPO setting |
| GPO report fails | ADMX/ADML parsing issue or corrupt GPO | `Get-GPOReport`; GPMC error | Restore Central Store backup or fix template files |
| Central Store not visible on all DCs | SYSVOL/DFSR replication delay or failure | `Test-Path "\\<dc>\SYSVOL\<domain>\Policies\PolicyDefinitions"` | Fix DFSR/SYSVOL replication |
| ADMX copied but ADML not copied | Only root files were copied | `Get-ChildItem "<central-store-path>\en-US"` | Copy language folder |
| Wrong language folder used | GPMC language does not match available ADML | Check language folder names | Add required language folder |
| Custom templates overwritten | Full copy replaced vendor templates | Compare backup and current inventory | Restore custom files from backup |
| Central Store permissions changed accidentally | Manual copy changed ACLs or folder ownership | `icacls "<central-store-path>"` | Restore from backup or correct permissions |
| SYSVOL path inaccessible | DNS, SMB, DC, or SYSVOL issue | `Test-Path "\\<domain>\SYSVOL"` | Fix domain connectivity or SYSVOL |
| DFSR backlog delays template visibility | SYSVOL replication not complete | DFSR events and backlog check | Wait or remediate DFSR |
| Admin copied templates from wrong OS version | Reference machine did not match managed OS | ADMX inventory and source documentation | Replace with approved version source |
| GPMC becomes unusable after update | Bad or mismatched template package | GPMC error and backup inventory | Roll back Central Store immediately |
| Central Store backup missing | Backup step skipped | Check backup path | Stop and manually preserve current state before changing more |
| Reports lack enough evidence | No inventory or hashes captured | Report folder check | Run inventory and hash collection skeleton |
| Multiple ADMX sources mixed randomly | Inconsistent package strategy | File timestamps and source notes | Rebuild Central Store from a clean approved source plus documented custom templates |

# 17_Configure_ADMX_Central_Store_And_Policy_Definitions_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy module used for validation |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Uses computer-side ADMX-backed policy settings |
| `07_Configure_User_Administrative_Template_Settings.md` | Uses user-side ADMX-backed policy settings |
| `11_Configure_Security_Baseline_GPO_Settings.md` | Uses security baseline Administrative Template settings |
| `12_Configure_Windows_Update_And_Defender_GPO_Settings.md` | Depends on current Windows Update and Defender ADMX availability |
| `13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md` | Uses RDP, WinRM, and firewall ADMX policy nodes |
| `14_Backup_Restore_Import_And_Report_GPOs.md` | Provides report and backup lifecycle around GPO changes |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Validates GPO settings after template updates |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Troubleshoots SYSVOL and replication issues that affect Central Store availability |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms SYSVOL, NETLOGON, DC locator, and replication health |
| `18_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers.md` | Next advanced policy behavior workbook using Administrative Template settings |