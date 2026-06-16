
07_Configure_User_Home_Folders_And_Department_Shares.md
# Configure_User_Home_Folders_And_Department_Shares

# Configure_User_Home_Folders_And_Department_Shares_Index
07_Configure_User_Home_Folders_And_Department_Shares.md
Configure_User_Home_Folders_And_Department_Shares
Configure_User_Home_Folders_And_Department_Shares_Source_Basis
Configure_User_Home_Folders_And_Department_Shares_Mental_Model
Configure_User_Home_Folders_And_Department_Shares_Planning_Table
Configure_User_Home_Folders_And_Department_Shares_Configuration_Checklist
Configure_User_Home_Folders_And_Department_Shares_Precheck_Skeleton
Configure_User_Home_Folders_And_Department_Shares_Department_Groups_And_Folders_Skeleton
Configure_User_Home_Folders_And_Department_Shares_Department_ACL_Skeleton
Configure_User_Home_Folders_And_Department_Shares_Home_Root_Preparation_Skeleton
Configure_User_Home_Folders_And_Department_Shares_Bulk_Home_Folder_Creation_Skeleton
Configure_User_Home_Folders_And_Department_Shares_AD_HomeDrive_Attribute_Skeleton
Configure_User_Home_Folders_And_Department_Shares_Client_Access_Test_Skeleton
Configure_User_Home_Folders_And_Department_Shares_Post_Change_Validation_Skeleton
Configure_User_Home_Folders_And_Department_Shares_Verification_Commands
Configure_User_Home_Folders_And_Department_Shares_Rollback
Configure_User_Home_Folders_And_Department_Shares_Failure_Checks
Configure_User_Home_Folders_And_Department_Shares_Related_Labs

# Configure_User_Home_Folders_And_Department_Shares_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | New-Item | Creating department folders and user home folders |
| Microsoft Learn | Get-Acl / Set-Acl | Reviewing and applying file system permissions |
| Microsoft Learn | icacls | Managing NTFS permissions, inheritance, ownership, and rollback evidence |
| Microsoft Learn | New-SmbShare | Creating SMB shares when missing |
| Microsoft Learn | Get-SmbShare | Validating existing department and home SMB shares |
| Microsoft Learn | Set-SmbShare | Confirming ABE and share properties |
| Microsoft Learn | Get-SmbShareAccess | Validating SMB share permissions |
| Microsoft Learn | Active Directory PowerShell module | Finding users and groups for home folder and department access |
| Microsoft Learn | Set-ADUser | Setting HomeDirectory and HomeDrive attributes |
| Microsoft Learn | Get-ADUser | Validating user attributes and user selection |
| Windows Server operational practice | Department share design | Use group-based NTFS ACLs on department folders under a common share |
| Windows Server operational practice | Home folder design | Use per-user folders with user-specific NTFS permissions and optional AD home drive mapping |
| Windows Server operational practice | AGDLP / AGUDLP | Assign users to identity groups, identity groups to resource groups, and resource groups to permissions |

# Configure_User_Home_Folders_And_Department_Shares_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Department share | Shared location for a department such as Accounting, HR, IT, or Operations |
| Home folder | User-specific private folder for individual files |
| Home drive | Drive letter mapped to a user's home folder, commonly `H:` |
| HomeDirectory attribute | AD user attribute containing the UNC path to the user's home folder |
| HomeDrive attribute | AD user attribute containing the drive letter assigned to the home folder |
| Common root share | A share such as `\\FS1\Departments` that contains many department folders |
| Hidden home share | A share such as `\\FS1\Home$` used to reduce casual browsing |
| Per-user folder | Folder named after the user's `SamAccountName`, such as `E:\Shares\Home\jsmith` |
| Department RW group | Group granted Modify rights to a department folder |
| Department RO group | Group granted Read and Execute rights to a department folder |
| File server admin group | Delegated group granted Full Control for file server operations |
| NTFS boundary | Folder level where inheritance is broken and explicit access is applied |
| ABE dependency | Access Based Enumeration hides folders only if NTFS permissions correctly block access |
| Share permission boundary | SMB layer that must allow users to reach the share before NTFS decides folder access |
| Effective access | Result of share permissions, NTFS permissions, group membership, and current logon token |
| Group token refresh | Users must sign out and back in after group membership changes |
| AGDLP | Accounts into Global groups, Global groups into Domain Local groups, Domain Local groups into Permissions |
| Bulk provisioning | Repeatable creation of many user folders or department folders from a CSV or list |
| First rule | Department folders and home folders should be group-based, repeatable, testable, and backed up before access goes live |

# Configure_User_Home_Folders_And_Department_Shares_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Data drive | `E:` | `<data-drive-letter>` |
| Department root local path | `E:\Shares\Departments` | `<department-root>` |
| Home root local path | `E:\Shares\Home` | `<home-folder-root>` |
| Department share name | `Departments` | `<department-share-name>` |
| Home share name | `Home$` | `<home-share-name>` |
| Department UNC | `\\FS1\Departments` | `<department-unc>` |
| Home UNC root | `\\FS1\Home$` | `<home-unc-root>` |
| Home drive letter | `H:` | `<home-drive-letter>` |
| Sample department | `Accounting` | `<department-name>` |
| Sample department path | `E:\Shares\Departments\Accounting` | `<department-folder-path>` |
| Sample department UNC | `\\FS1\Departments\Accounting` | `<department-folder-unc>` |
| Department RW group | `CORP\GG-FS-Accounting-RW` | `<department-rw-group>` |
| Department RO group | `CORP\GG-FS-Accounting-RO` | `<department-ro-group>` |
| File server admins group | `CORP\GG-FileServer-Admins` | `<file-server-admins-group>` |
| Home folder user group | `CORP\Domain Users` | `<home-users-group>` |
| Sample user | `jsmith` | `<sample-samaccountname>` |
| Sample home folder path | `E:\Shares\Home\jsmith` | `<sample-home-folder-path>` |
| Sample home folder UNC | `\\FS1\Home$\jsmith` | `<sample-home-folder-unc>` |
| User selection method | OU, group, or CSV | `<user-selection-method>` |
| User source OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<user-source-ou>` |
| Department source list | CSV or manual list | `<department-source>` |
| ABE stance | Enabled on Departments and Home | `<abe-plan>` |
| SMB encryption stance | Enabled on sensitive shares | `<smb-encryption-plan>` |
| ACL backup path | `C:\FileServices-Validation\ACL-Backups` | `<acl-backup-path>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Client evidence path | `C:\FileServices-Validation-Client` | `<client-evidence-path>` |
| Next workbook | `08_Configure_File_Screening_With_FSRM.md` | `<next-task>` |

# Configure_User_Home_Folders_And_Department_Shares_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm SMB service is running | File Server | `Get-Service LanmanServer` | SMB server service is Running |
| 4 | Confirm department and home roots exist | File Server | `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"` | Required local roots exist |
| 5 | Confirm department and home shares exist | File Server | `Get-SmbShare -Name "<department-share-name>"`; `Get-SmbShare -Name "<home-share-name>"` | SMB shares exist |
| 6 | Confirm ABE on department and home shares | File Server | `Get-SmbShare -Name "<share-name>" \| Select Name,FolderEnumerationMode` | ABE state is known |
| 7 | Confirm share permissions | File Server | `Get-SmbShareAccess -Name "<share-name>"` | Share permissions allow intended access boundary |
| 8 | Confirm AD module access | File Server / Admin Host | `Get-Module ActiveDirectory -ListAvailable` | AD cmdlets are available where needed |
| 9 | Confirm file server admin group exists | Admin Host | `Get-ADGroup "<file-server-admins-group>"` | Admin group exists |
| 10 | Confirm department RW group exists | Admin Host | `Get-ADGroup "<department-rw-group>"` | RW group exists |
| 11 | Confirm department RO group exists | Admin Host | `Get-ADGroup "<department-ro-group>"` | RO group exists |
| 12 | Capture ACL backup before changes | File Server | `icacls "<department-root>" /save "<acl-backup-path>\department-root-before-task07.txt" /t /c` | Department ACL backup exists |
| 13 | Capture home ACL backup before changes | File Server | `icacls "<home-folder-root>" /save "<acl-backup-path>\home-root-before-task07.txt" /t /c` | Home ACL backup exists |
| 14 | Create department folder | File Server | `New-Item -ItemType Directory -Path "<department-folder-path>" -Force` | Department folder exists |
| 15 | Break inheritance on department folder | File Server | `icacls "<department-folder-path>" /inheritance:r` | Department folder becomes ACL boundary |
| 16 | Grant admins Full Control on department folder | File Server | `icacls "<department-folder-path>" /grant "<file-server-admins-group>:(OI)(CI)(F)"` | Admin group has Full Control |
| 17 | Grant RW group Modify on department folder | File Server | `icacls "<department-folder-path>" /grant "<department-rw-group>:(OI)(CI)(M)"` | RW group can create and modify content |
| 18 | Grant RO group Read and Execute on department folder | File Server | `icacls "<department-folder-path>" /grant "<department-ro-group>:(OI)(CI)(RX)"` | RO group can read content |
| 19 | Validate department ACL | File Server | `icacls "<department-folder-path>"` | Department permissions match design |
| 20 | Prepare home root ACL | File Server | Use home root preparation skeleton | Home root is ready for per-user folders |
| 21 | Create per-user home folder | File Server | `New-Item -ItemType Directory -Path "<sample-home-folder-path>" -Force` | User home folder exists |
| 22 | Break inheritance on user home folder | File Server | `icacls "<sample-home-folder-path>" /inheritance:r` | User folder becomes ACL boundary |
| 23 | Grant user Modify on own home folder | File Server | `icacls "<sample-home-folder-path>" /grant "<netbios-name>\<sam>:(OI)(CI)(M)"` | User can manage own files |
| 24 | Grant admins Full Control on user home folder | File Server | `icacls "<sample-home-folder-path>" /grant "<file-server-admins-group>:(OI)(CI)(F)"` | Admin group can manage user folder |
| 25 | Set AD home folder attributes if used | Admin Host | `Set-ADUser "<sam>" -HomeDirectory "\\<file-server>\<home-share>\<sam>" -HomeDrive "H:"` | User has home drive attributes |
| 26 | Test department access as RW user | Client | `New-Item "\\<file-server>\<department-share-name>\<department>\rw-test.txt"` | RW user can write |
| 27 | Test department access as RO user | Client | `Get-ChildItem "\\<file-server>\<department-share-name>\<department>"` then attempt write | RO user can read but cannot write |
| 28 | Test home folder access as owner | Client | `New-Item "\\<file-server>\<home-share-name>\<sam>\home-test.txt"` | User can write to own home folder |
| 29 | Test home folder isolation | Client | Attempt access to another user's home folder | User cannot access another user's folder |
| 30 | Export final ACL and share evidence | File Server | Use post-change validation skeleton | Final evidence is saved |
| 31 | Document final department and home folder model | Operator | `Record groups, users, folder paths, shares, and validation results` | Workbook record is complete |

# Configure_User_Home_Folders_And_Department_Shares_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate folder roots, SMB shares, ABE, ACLs, and AD tooling before creating department and home folders.

$EvidencePath = "C:\FileServices-Validation"
$AclBackupPath = "$EvidencePath\ACL-Backups"

$DomainNetbios = "CORP"
$FileServerName = $env:COMPUTERNAME

$DepartmentShareName = "Departments"
$HomeShareName = "Home$"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$SampleDepartment = "Accounting"
$DepartmentRWGroup = "$DomainNetbios\GG-FS-Accounting-RW"
$DepartmentROGroup = "$DomainNetbios\GG-FS-Accounting-RO"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $AclBackupPath

Start-Transcript -Path "$EvidencePath\07-home-department-precheck-transcript.txt"

# Confirm admin context and server identity.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-home-department.txt"

hostname |
  Tee-Object "$EvidencePath\hostname-before-home-department.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-home-department.txt"

# Confirm File Server role and SMB services.
Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-home-department.txt"

Get-Service LanmanServer,LanmanWorkstation |
  Tee-Object "$EvidencePath\smb-services-before-home-department.txt"

# Confirm required roots.
$RequiredPaths = @(
  $DepartmentRoot,
  $HomeRoot
)

foreach ($Path in $RequiredPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\required-paths-before-home-department.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-home-department-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Confirm SMB shares and visibility settings.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,CachingMode,Special |
  Tee-Object "$EvidencePath\smb-shares-before-home-department.txt"

foreach ($Share in @($DepartmentShareName,$HomeShareName)) {
  Get-SmbShare -Name $Share -ErrorAction SilentlyContinue |
    Select-Object Name,Path,FolderEnumerationMode,EncryptData,CachingMode,Special |
    Tee-Object "$EvidencePath\smb-share-state-before-home-department-$($Share.Replace('$','hidden')).txt"

  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\smb-share-access-before-home-department-$($Share.Replace('$','hidden')).txt"
}

# Capture ACL backups.
if (Test-Path $DepartmentRoot) {
  icacls $DepartmentRoot /save "$AclBackupPath\department-root-before-task07.txt" /t /c
}

if (Test-Path $HomeRoot) {
  icacls $HomeRoot /save "$AclBackupPath\home-root-before-task07.txt" /t /c
}

# Confirm Active Directory module availability if present locally.
Get-Module ActiveDirectory -ListAvailable |
  Tee-Object "$EvidencePath\active-directory-module-availability.txt"

Stop-Transcript
````

# Configure_User_Home_Folders_And_Department_Shares_Department_Groups_And_Folders_Skeleton

```powershell
# Run on a DC or admin host with the ActiveDirectory module.
# Purpose: validate or create department access groups, then create the matching department folder on the file server.

$EvidencePath = "C:\FileServices-Validation"
$DomainNetbios = "CORP"

$DepartmentName = "Accounting"
$DepartmentRoot = "E:\Shares\Departments"
$DepartmentPath = Join-Path $DepartmentRoot $DepartmentName

$DepartmentRWGroupName = "GG-FS-$DepartmentName-RW"
$DepartmentROGroupName = "GG-FS-$DepartmentName-RO"

$GroupPath = "OU=Groups,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\07-department-groups-and-folders-transcript.txt"

# Confirm AD module.
Import-Module ActiveDirectory

# Create RW group if missing.
if (-not (Get-ADGroup -Filter "Name -eq '$DepartmentRWGroupName'" -ErrorAction SilentlyContinue)) {
  New-ADGroup `
    -Name $DepartmentRWGroupName `
    -SamAccountName $DepartmentRWGroupName `
    -GroupScope Global `
    -GroupCategory Security `
    -Path $GroupPath `
    -Description "Read/write access to $DepartmentName department file share"
}

# Create RO group if missing.
if (-not (Get-ADGroup -Filter "Name -eq '$DepartmentROGroupName'" -ErrorAction SilentlyContinue)) {
  New-ADGroup `
    -Name $DepartmentROGroupName `
    -SamAccountName $DepartmentROGroupName `
    -GroupScope Global `
    -GroupCategory Security `
    -Path $GroupPath `
    -Description "Read-only access to $DepartmentName department file share"
}

# Confirm groups.
Get-ADGroup $DepartmentRWGroupName |
  Tee-Object "$EvidencePath\ad-group-$DepartmentRWGroupName.txt"

Get-ADGroup $DepartmentROGroupName |
  Tee-Object "$EvidencePath\ad-group-$DepartmentROGroupName.txt"

# Create department folder.
New-Item `
  -ItemType Directory `
  -Path $DepartmentPath `
  -Force |
  Tee-Object "$EvidencePath\department-folder-create-$DepartmentName.txt"

# Confirm folder.
Test-Path $DepartmentPath |
  Tee-Object "$EvidencePath\department-folder-testpath-$DepartmentName.txt"

Stop-Transcript
```

# Configure_User_Home_Folders_And_Department_Shares_Department_ACL_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: configure NTFS permissions for department folders under the Departments share.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"
$DepartmentRoot = "E:\Shares\Departments"

$DepartmentName = "Accounting"
$DepartmentPath = Join-Path $DepartmentRoot $DepartmentName

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$DepartmentRWGroup = "$DomainNetbios\GG-FS-$DepartmentName-RW"
$DepartmentROGroup = "$DomainNetbios\GG-FS-$DepartmentName-RO"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $DepartmentPath

Start-Transcript -Path "$EvidencePath\07-department-acl-$DepartmentName-transcript.txt"

# Capture current ACL.
icacls $DepartmentPath |
  Tee-Object "$EvidencePath\department-acl-before-$DepartmentName.txt"

# Break inheritance on the department folder.
icacls $DepartmentPath /inheritance:r

# Administrative control.
icacls $DepartmentPath /grant "SYSTEM:(OI)(CI)(F)"
icacls $DepartmentPath /grant "BUILTIN\Administrators:(OI)(CI)(F)"
icacls $DepartmentPath /grant "${FileServerAdminsGroup}:(OI)(CI)(F)"

# Department access.
icacls $DepartmentPath /grant "${DepartmentRWGroup}:(OI)(CI)(M)"
icacls $DepartmentPath /grant "${DepartmentROGroup}:(OI)(CI)(RX)"

# Remove broad access if present.
icacls $DepartmentPath /remove:g "Users" 2>$null
icacls $DepartmentPath /remove:g "Authenticated Users" 2>$null
icacls $DepartmentPath /remove:g "Everyone" 2>$null

# Confirm final ACL.
icacls $DepartmentPath |
  Tee-Object "$EvidencePath\department-acl-after-$DepartmentName.txt"

Get-Acl $DepartmentPath |
  Format-List |
  Tee-Object "$EvidencePath\department-get-acl-after-$DepartmentName.txt"

Stop-Transcript
```

# Configure_User_Home_Folders_And_Department_Shares_Home_Root_Preparation_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: prepare the Home root and Home$ share for per-user private folders.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"
$FileServerName = $env:COMPUTERNAME

$HomeRoot = "E:\Shares\Home"
$HomeShareName = "Home$"

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$AuthenticatedUsers = "Authenticated Users"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $HomeRoot

Start-Transcript -Path "$EvidencePath\07-home-root-preparation-transcript.txt"

# Capture current ACL.
icacls $HomeRoot |
  Tee-Object "$EvidencePath\home-root-acl-before-task07.txt"

# Create hidden Home$ share if missing.
if (-not (Get-SmbShare -Name $HomeShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $HomeShareName `
    -Path $HomeRoot `
    -Description "User home folders" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ChangeAccess $AuthenticatedUsers `
    -FolderEnumerationMode AccessBased `
    -CachingMode Manual
}

# Normalize Home$ share visibility and access.
Set-SmbShare `
  -Name $HomeShareName `
  -FolderEnumerationMode AccessBased `
  -Force

Grant-SmbShareAccess `
  -Name $HomeShareName `
  -AccountName $FileServerAdminsGroup `
  -AccessRight Full `
  -Force

Grant-SmbShareAccess `
  -Name $HomeShareName `
  -AccountName "BUILTIN\Administrators" `
  -AccessRight Full `
  -Force

Grant-SmbShareAccess `
  -Name $HomeShareName `
  -AccountName $AuthenticatedUsers `
  -AccessRight Change `
  -Force

# Prepare root ACL.
# Users need to reach the root, but individual user folders enforce privacy.
icacls $HomeRoot /inheritance:r
icacls $HomeRoot /grant "SYSTEM:(OI)(CI)(F)"
icacls $HomeRoot /grant "BUILTIN\Administrators:(OI)(CI)(F)"
icacls $HomeRoot /grant "${FileServerAdminsGroup}:(OI)(CI)(F)"
icacls $HomeRoot /grant "Authenticated Users:(RX)"
icacls $HomeRoot /grant "CREATOR OWNER:(OI)(CI)(IO)(F)"

# Confirm final root and share state.
icacls $HomeRoot |
  Tee-Object "$EvidencePath\home-root-acl-after-task07.txt"

Get-SmbShare -Name $HomeShareName |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,CachingMode,Special |
  Tee-Object "$EvidencePath\home-share-final-task07.txt"

Get-SmbShareAccess -Name $HomeShareName |
  Tee-Object "$EvidencePath\home-share-access-final-task07.txt"

Stop-Transcript
```

# Configure_User_Home_Folders_And_Department_Shares_Bulk_Home_Folder_Creation_Skeleton

```powershell
# Run in elevated PowerShell on the file server or from an admin host with access to the file server path.
# Purpose: create per-user home folders and apply private NTFS ACLs.
# Replace the sample list with a CSV import or AD query for real bulk provisioning.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"
$HomeRoot = "E:\Shares\Home"

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"

# Sample users. Replace with Import-Csv or Get-ADUser.
$Users = @(
  [PSCustomObject]@{ SamAccountName = "jsmith" },
  [PSCustomObject]@{ SamAccountName = "adoe" },
  [PSCustomObject]@{ SamAccountName = "bwilliams" }
)

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $HomeRoot

Start-Transcript -Path "$EvidencePath\07-bulk-home-folder-creation-transcript.txt"

foreach ($User in $Users) {
  $Sam = $User.SamAccountName
  $UserIdentity = "$DomainNetbios\$Sam"
  $UserHomePath = Join-Path $HomeRoot $Sam

  # Create user folder.
  New-Item `
    -ItemType Directory `
    -Path $UserHomePath `
    -Force |
    Out-Null

  # Configure private ACL.
  icacls $UserHomePath /inheritance:r

  icacls $UserHomePath /grant "SYSTEM:(OI)(CI)(F)"
  icacls $UserHomePath /grant "BUILTIN\Administrators:(OI)(CI)(F)"
  icacls $UserHomePath /grant "${FileServerAdminsGroup}:(OI)(CI)(F)"
  icacls $UserHomePath /grant "${UserIdentity}:(OI)(CI)(M)"

  # Remove broad access if present.
  icacls $UserHomePath /remove:g "Users" 2>$null
  icacls $UserHomePath /remove:g "Authenticated Users" 2>$null
  icacls $UserHomePath /remove:g "Everyone" 2>$null

  # Capture result.
  icacls $UserHomePath |
    Tee-Object "$EvidencePath\home-folder-acl-$Sam.txt"
}

# Final inventory.
Get-ChildItem $HomeRoot -Directory |
  Sort-Object Name |
  Select-Object Name,FullName,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\home-folder-inventory-after-create.txt"

Stop-Transcript
```

# Configure_User_Home_Folders_And_Department_Shares_AD_HomeDrive_Attribute_Skeleton

```powershell
# Run on a DC or admin host with the ActiveDirectory module.
# Purpose: set each user's HomeDirectory and HomeDrive attributes in Active Directory.
# This does not replace GPO mapped drives. It sets classic AD home folder attributes.

$EvidencePath = "C:\FileServices-Validation"

$FileServerName = "FS1"
$HomeShareName = "Home$"
$HomeDriveLetter = "H:"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\07-ad-homedrive-attributes-transcript.txt"

Import-Module ActiveDirectory

# Sample users. Replace with Import-Csv or Get-ADUser search.
$Users = @(
  [PSCustomObject]@{ SamAccountName = "jsmith" },
  [PSCustomObject]@{ SamAccountName = "adoe" },
  [PSCustomObject]@{ SamAccountName = "bwilliams" }
)

foreach ($User in $Users) {
  $Sam = $User.SamAccountName
  $HomeDirectory = "\\$FileServerName\$HomeShareName\$Sam"

  # Capture before state.
  Get-ADUser $Sam -Properties HomeDirectory,HomeDrive |
    Select-Object SamAccountName,Enabled,HomeDirectory,HomeDrive |
    Tee-Object "$EvidencePath\ad-user-home-before-$Sam.txt"

  # Set home folder attributes.
  Set-ADUser `
    -Identity $Sam `
    -HomeDirectory $HomeDirectory `
    -HomeDrive $HomeDriveLetter

  # Capture after state.
  Get-ADUser $Sam -Properties HomeDirectory,HomeDrive |
    Select-Object SamAccountName,Enabled,HomeDirectory,HomeDrive |
    Tee-Object "$EvidencePath\ad-user-home-after-$Sam.txt"
}

# Export final state.
$Users |
  ForEach-Object {
    Get-ADUser $_.SamAccountName -Properties HomeDirectory,HomeDrive |
      Select-Object SamAccountName,Enabled,HomeDirectory,HomeDrive
  } |
  Export-Csv "$EvidencePath\ad-user-home-attributes-final.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_User_Home_Folders_And_Department_Shares_Client_Access_Test_Skeleton

```powershell
# Run from a Windows client as a test user.
# Purpose: validate department and home folder access from the user perspective.

$FileServerName = "FS1"

$DepartmentShareName = "Departments"
$HomeShareName = "Home$"

$DepartmentName = "Accounting"
$SamAccountName = $env:USERNAME

$DepartmentUNC = "\\$FileServerName\$DepartmentShareName\$DepartmentName"
$HomeUNC = "\\$FileServerName\$HomeShareName\$SamAccountName"

$OtherUserHomeUNC = "\\$FileServerName\$HomeShareName\otheruser"

$EvidencePath = "C:\FileServices-Validation-Client"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\07-client-home-department-access-test-transcript.txt"

# Capture client identity and token.
hostname |
  Tee-Object "$EvidencePath\client-hostname-task07.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami-task07.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups-task07.txt"

# Confirm server resolution and SMB reachability.
Resolve-DnsName $FileServerName |
  Tee-Object "$EvidencePath\client-resolve-fileserver-task07.txt"

Test-NetConnection $FileServerName -Port 445 |
  Tee-Object "$EvidencePath\client-test-smb-445-task07.txt"

# Confirm share reachability.
Test-Path "\\$FileServerName\$DepartmentShareName" |
  Tee-Object "$EvidencePath\department-share-testpath-task07.txt"

Test-Path "\\$FileServerName\$HomeShareName" |
  Tee-Object "$EvidencePath\home-share-testpath-task07.txt"

# Department folder test.
Test-Path $DepartmentUNC |
  Tee-Object "$EvidencePath\department-folder-testpath-task07.txt"

Get-ChildItem $DepartmentUNC -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\department-folder-listing-task07.txt"

$DepartmentTestFile = Join-Path $DepartmentUNC "department-access-test-$SamAccountName.txt"

"Department access test from $SamAccountName at $(Get-Date)" |
  Out-File $DepartmentTestFile -Force

Get-Content $DepartmentTestFile |
  Tee-Object "$EvidencePath\department-test-file-readback-task07.txt"

Remove-Item $DepartmentTestFile -Force

# Home folder test.
Test-Path $HomeUNC |
  Tee-Object "$EvidencePath\home-folder-testpath-task07.txt"

Get-ChildItem $HomeUNC -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\home-folder-listing-task07.txt"

$HomeTestFile = Join-Path $HomeUNC "home-access-test-$SamAccountName.txt"

"Home folder access test from $SamAccountName at $(Get-Date)" |
  Out-File $HomeTestFile -Force

Get-Content $HomeTestFile |
  Tee-Object "$EvidencePath\home-test-file-readback-task07.txt"

Remove-Item $HomeTestFile -Force

# Isolation test.
# Expected result: access to another user's folder should fail unless this test account is an admin.
Test-Path $OtherUserHomeUNC |
  Tee-Object "$EvidencePath\other-user-home-testpath-task07.txt"

Get-ChildItem $OtherUserHomeUNC -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\other-user-home-listing-task07.txt"

# SMB connection details.
Get-SmbConnection |
  Where-Object { $_.ServerName -like "$FileServerName*" -or $_.ServerName -eq $FileServerName } |
  Select-Object ServerName,ShareName,Dialect,Signed,Encrypted,UserName |
  Tee-Object "$EvidencePath\client-smb-connection-task07.txt"

Stop-Transcript
```

# Configure_User_Home_Folders_And_Department_Shares_Post_Change_Validation_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture final folder, ACL, share, ABE, SMB security, and user home folder evidence.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"

$DepartmentShareName = "Departments"
$HomeShareName = "Home$"

$DepartmentName = "Accounting"
$DepartmentPath = Join-Path $DepartmentRoot $DepartmentName

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\07-post-change-home-department-validation-transcript.txt"

# Folder inventory.
Get-ChildItem $DepartmentRoot -Directory -Force |
  Sort-Object Name |
  Select-Object Name,FullName,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\department-folder-inventory-final.txt"

Get-ChildItem $HomeRoot -Directory -Force |
  Sort-Object Name |
  Select-Object Name,FullName,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\home-folder-inventory-final.txt"

# ACL evidence.
icacls $DepartmentRoot |
  Tee-Object "$EvidencePath\department-root-acl-final-task07.txt"

icacls $DepartmentPath |
  Tee-Object "$EvidencePath\department-sample-acl-final-task07.txt"

icacls $HomeRoot |
  Tee-Object "$EvidencePath\home-root-acl-final-task07.txt"

Get-ChildItem $HomeRoot -Directory -Force |
  ForEach-Object {
    icacls $_.FullName
  } |
  Tee-Object "$EvidencePath\home-folder-acls-final-task07.txt"

# Share evidence.
Get-SmbShare -Name $DepartmentShareName,$HomeShareName -ErrorAction SilentlyContinue |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,CachingMode,Special |
  Tee-Object "$EvidencePath\department-home-share-settings-final-task07.txt"

foreach ($Share in @($DepartmentShareName,$HomeShareName)) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\smb-share-access-final-task07-$($Share.Replace('$','hidden')).txt"
}

# SMB sessions and open files.
Get-SmbSession |
  Tee-Object "$EvidencePath\smb-sessions-final-task07.txt"

Get-SmbOpenFile |
  Tee-Object "$EvidencePath\smb-open-files-final-task07.txt"

# Recent related events.
Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = @(5140,5145,4663)
  StartTime = (Get-Date).AddHours(-4)
} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,Message |
  Tee-Object "$EvidencePath\security-events-final-task07.txt"

Stop-Transcript
```

# Configure_User_Home_Folders_And_Department_Shares_Verification_Commands

```powershell
# File Server and SMB baseline
Get-WindowsFeature FS-FileServer
Get-Service LanmanServer
Get-SmbShare
Get-SmbShare -Name "<department-share-name>"
Get-SmbShare -Name "<home-share-name>"

# Share settings
Get-SmbShare -Name "<department-share-name>" | Select-Object Name,Path,FolderEnumerationMode,EncryptData
Get-SmbShare -Name "<home-share-name>" | Select-Object Name,Path,FolderEnumerationMode,EncryptData
Get-SmbShareAccess -Name "<department-share-name>"
Get-SmbShareAccess -Name "<home-share-name>"

# Folder paths
Test-Path "<department-root>"
Test-Path "<home-folder-root>"
Test-Path "<department-folder-path>"
Test-Path "<sample-home-folder-path>"

# Folder inventory
Get-ChildItem "<department-root>" -Directory
Get-ChildItem "<home-folder-root>" -Directory

# NTFS ACLs
icacls "<department-root>"
icacls "<department-folder-path>"
icacls "<home-folder-root>"
icacls "<sample-home-folder-path>"

Get-Acl "<department-folder-path>" | Format-List
Get-Acl "<sample-home-folder-path>" | Format-List

# AD groups and users
Get-ADGroup "<department-rw-group>"
Get-ADGroup "<department-ro-group>"
Get-ADGroupMember "<department-rw-group>"
Get-ADGroupMember "<department-ro-group>"

Get-ADUser "<sample-samaccountname>" -Properties HomeDirectory,HomeDrive |
  Select-Object SamAccountName,HomeDirectory,HomeDrive

# Client UNC tests
Test-NetConnection "<file-server-name>" -Port 445
Test-Path "\\<file-server-name>\<department-share-name>"
Test-Path "\\<file-server-name>\<department-share-name>\<department-name>"
Test-Path "\\<file-server-name>\<home-share-name>\<sample-samaccountname>"

# User token validation
whoami
whoami /groups

# Home folder write test
New-Item -ItemType File -Path "\\<file-server-name>\<home-share-name>\<sample-samaccountname>\home-test.txt"
Remove-Item "\\<file-server-name>\<home-share-name>\<sample-samaccountname>\home-test.txt"

# Department write test
New-Item -ItemType File -Path "\\<file-server-name>\<department-share-name>\<department-name>\department-test.txt"
Remove-Item "\\<file-server-name>\<department-share-name>\<department-name>\department-test.txt"

# SMB connection state
Get-SmbConnection | Select-Object ServerName,ShareName,Dialect,Signed,Encrypted,UserName

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*task07*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*home*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*department*"
```

# Configure_User_Home_Folders_And_Department_Shares_Rollback

|Step|Task|Device|PowerShell / Command|Expected Result|
|--:|---|---|---|---|
|1|Capture current department and home folder state|File Server|`Get-ChildItem "<department-root>","<home-folder-root>" -Directory`|Current folder inventory is saved|
|2|Capture current ACLs before rollback|File Server|`icacls "<department-root>" /save "<acl-backup-path>\department-root-before-task07-rollback.txt" /t /c`|Current ACL state is saved|
|3|Remove test files|File Server|`Remove-Item "<path>\*-test*.txt" -Force`|Test artifacts are removed|
|4|Remove incorrect department folder if empty or lab-only|File Server|`Remove-Item "<department-folder-path>" -Recurse -Force`|Incorrect department folder is removed|
|5|Remove incorrect user home folder if empty or lab-only|File Server|`Remove-Item "<sample-home-folder-path>" -Recurse -Force`|Incorrect home folder is removed|
|6|Restore department ACL backup|File Server|`icacls "<restore-parent-path>" /restore "<acl-backup-path>\department-root-before-task07.txt"`|Previous department ACLs are restored|
|7|Restore home ACL backup|File Server|`icacls "<restore-parent-path>" /restore "<acl-backup-path>\home-root-before-task07.txt"`|Previous home ACLs are restored|
|8|Remove incorrect department group membership|Admin Host|`Remove-ADGroupMember "<group-name>" -Members "<user>" -Confirm:$false`|User is removed from incorrect group|
|9|Remove AD home folder attributes if rollback requires it|Admin Host|`Set-ADUser "<sam>" -HomeDirectory $null -HomeDrive $null`|User home attributes are cleared|
|10|Revert home drive to prior path if known|Admin Host|`Set-ADUser "<sam>" -HomeDirectory "<old-home-unc>" -HomeDrive "<old-drive-letter>"`|User home attributes point to previous location|
|11|Remove incorrect Home$ share if created by mistake|File Server|`Remove-SmbShare -Name "Home$" -Force`|Hidden home share is removed|
|12|Recreate intended Home share if removed by mistake|File Server|`New-SmbShare -Name "<home-share-name>" -Path "<home-folder-root>" -ChangeAccess "Authenticated Users"`|Intended home share is restored|
|13|Validate user access after rollback|Client|`Test-Path "\\<file-server>\<share>\<path>"`|User access matches rollback target|
|14|Document rollback result|Operator|`Record removed folders, restored ACLs, changed AD attributes, and test results`|Rollback record is complete|

# Configure_User_Home_Folders_And_Department_Shares_Failure_Checks

|Symptom|Likely Cause|Check|Corrective Action|
|---|---|---|---|
|User cannot access department folder|User missing department RW or RO group|`whoami /groups`; `Get-ADPrincipalGroupMembership <user>`|Add user to correct department group and refresh token|
|User can see department but cannot write|User is in RO group only or NTFS Modify missing|`icacls "<department-folder-path>"`; `whoami /groups`|Add RW group or correct NTFS ACL|
|RO user can write|User is also in RW group|`Get-ADPrincipalGroupMembership <user>`|Remove overlapping RW membership|
|Unauthorized user can see department folder|ABE disabled or NTFS allows list access|`Get-SmbShare`; `icacls "<department-folder-path>"`|Enable ABE and remove broad NTFS access|
|Unauthorized user can open department folder|Broad NTFS or share permission allows access|`icacls`; `Get-SmbShareAccess`|Remove broad NTFS ACE and validate group model|
|User cannot access own home folder|User-specific ACL missing or wrong folder name|`icacls "<home-folder-path>"`|Grant user Modify on correct folder|
|User can access another user's home folder|Home folder inherited broad access|`icacls "<other-home-folder-path>"`|Break inheritance and remove broad access|
|Home drive does not map at sign-in|AD HomeDirectory/HomeDrive not set or client did not process it|`Get-ADUser <sam> -Properties HomeDirectory,HomeDrive`|Set AD attributes and have user sign out/in|
|Home drive maps to wrong server|Old AD attribute still present|`Get-ADUser <sam> -Properties HomeDirectory,HomeDrive`|Update HomeDirectory to correct UNC|
|`Set-ADUser` is not recognized|Active Directory module missing|`Get-Module ActiveDirectory -ListAvailable`|Run from DC/admin host or install RSAT AD tools|
|Group creation fails|Wrong OU path or insufficient rights|`Get-ADOrganizationalUnit`; admin token|Correct group path or use delegated admin|
|`icacls` group grant fails|Group name typo or domain prefix issue|`Get-ADGroup "<group-name>"`|Use exact group name and NetBIOS prefix|
|User still has old access after group change|Kerberos token has not refreshed|`whoami /groups`|Sign out and sign back in or purge tickets|
|Hidden Home$ share not reachable|Share missing or typed without `$`|`Get-SmbShare -Name "Home$"`|Create share or use exact UNC path|
|Home$ appears in `Get-SmbShare`|Server-side tools show hidden shares|`Get-SmbShare`|Normal behavior. Test visibility with client `net view`|
|Client cannot reach any share|DNS or SMB firewall issue|`Resolve-DnsName`; `Test-NetConnection <server> -Port 445`|Fix DNS or firewall path|
|ABE hides expected folder|User does not have NTFS access|`icacls`; `whoami /groups`|Correct NTFS ACL or group membership|
|Department folder created under wrong root|Variable or path typo|`Get-ChildItem E:\Shares -Recurse -Directory`|Move or remove wrong folder and rerun with correct path|
|Evidence missing|Validation skeleton not run|`Test-Path "<evidence-path>"`|Re-run precheck or post-change validation skeleton|

# Configure_User_Home_Folders_And_Department_Shares_Related_Labs

|Related Lab|Relationship|
|---|---|
|`00_File_Services_Index.md`|Defines the File Services suite order and home/department share placement|
|`01_Install_File_Server_Role_And_Management_Tools.md`|Provides File Server role and management tools|
|`02_Prepare_File_Server_Storage_Volumes_And_Folders.md`|Creates the root paths used for department and home folders|
|`03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`|Provides baseline ACL model used by this workbook|
|`04_Create_SMB_Shares_And_Share_Permissions.md`|Provides SMB shares used for department and home access|
|`05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`|Hides inaccessible department and home folders from users|
|`06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`|Provides secure and auditable SMB transport for user data|
|`08_Configure_File_Screening_With_FSRM.md`|Applies file type restrictions to department and home folder paths|
|`09_Configure_Storage_Quotas_With_FSRM.md`|Applies storage limits to home folders and department roots|
|`10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md`|Reports on user and department storage consumption|
|`11_Configure_Shadow_Copies_And_Previous_Versions.md`|Enables user self-service recovery for department and home data|
|`12_Backup_Restore_And_Export_File_Server_Config.md`|Protects folder data, ACLs, shares, and user home attributes|
|`15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`|Monitors active department and home share usage|
|`16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`|Troubleshoots user access failures across SMB, NTFS, ABE, and group membership|
|`25_Migrate_File_Server_Data_Shares_And_Permissions.md`|Migrates department and home folders while preserving ACLs and attributes|