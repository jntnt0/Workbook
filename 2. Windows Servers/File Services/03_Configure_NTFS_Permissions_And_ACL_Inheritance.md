03_Configure_NTFS_Permissions_And_ACL_Inheritance.md
# Configure_NTFS_Permissions_And_ACL_Inheritance

# Configure_NTFS_Permissions_And_ACL_Inheritance_Index
03_Configure_NTFS_Permissions_And_ACL_Inheritance.md
Configure_NTFS_Permissions_And_ACL_Inheritance
Configure_NTFS_Permissions_And_ACL_Inheritance_Source_Basis
Configure_NTFS_Permissions_And_ACL_Inheritance_Mental_Model
Configure_NTFS_Permissions_And_ACL_Inheritance_Planning_Table
Configure_NTFS_Permissions_And_ACL_Inheritance_Configuration_Checklist
Configure_NTFS_Permissions_And_ACL_Inheritance_Precheck_Skeleton
Configure_NTFS_Permissions_And_ACL_Inheritance_ACL_Backup_Skeleton
Configure_NTFS_Permissions_And_ACL_Inheritance_Share_Root_ACL_Skeleton
Configure_NTFS_Permissions_And_ACL_Inheritance_Department_ACL_Skeleton
Configure_NTFS_Permissions_And_ACL_Inheritance_Home_Root_ACL_Skeleton
Configure_NTFS_Permissions_And_ACL_Inheritance_Effective_Access_Test_Skeleton
Configure_NTFS_Permissions_And_ACL_Inheritance_Post_Change_Validation_Skeleton
Configure_NTFS_Permissions_And_ACL_Inheritance_Verification_Commands
Configure_NTFS_Permissions_And_ACL_Inheritance_Rollback
Configure_NTFS_Permissions_And_ACL_Inheritance_Failure_Checks
Configure_NTFS_Permissions_And_ACL_Inheritance_Related_Labs

# Configure_NTFS_Permissions_And_ACL_Inheritance_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | NTFS permissions | File and folder access control using discretionary access control lists |
| Microsoft Learn | Get-Acl | Reading existing ACLs before and after permission changes |
| Microsoft Learn | Set-Acl | Applying ACLs through PowerShell when required |
| Microsoft Learn | icacls | Backing up, restoring, granting, removing, and changing inheritance on ACLs |
| Microsoft Learn | Active Directory groups | Group-based access control for file server permissions |
| Microsoft Learn | Effective access | Validating how user and group membership produce final access |
| Microsoft Learn | Access control inheritance | Controlling whether child folders inherit permissions from parent folders |
| Microsoft Learn | SMB and NTFS interaction | Understanding that final network access is the most restrictive result of share and NTFS permissions |
| Windows Server operational practice | AGDLP / AGUDLP | Assign users to global groups, global groups to domain local resource groups, resource groups to ACLs |
| Windows Server operational practice | Least privilege file shares | Use groups, avoid user-specific ACLs, avoid deny permissions unless there is a strict reason |

# Configure_NTFS_Permissions_And_ACL_Inheritance_Mental_Model
| Concept | Operational Meaning |
|---|---|
| NTFS permissions | File system permissions stored directly on files and folders |
| ACL | Access Control List containing access rules for users, groups, and system principals |
| ACE | Access Control Entry inside an ACL |
| Inheritance | Permission flow from parent folders to child folders and files |
| Explicit permission | Permission applied directly to a folder or file |
| Inherited permission | Permission received from a parent folder |
| Owner | Security principal with ownership rights over the object |
| Effective access | Final access after group membership, allow entries, deny entries, inheritance, and share permissions are evaluated |
| Allow ACE | Permission that grants access |
| Deny ACE | Permission that blocks access. Avoid unless specifically required |
| Full Control | Permission to read, write, modify, delete, change permissions, and take ownership |
| Modify | Permission to read, write, create, change, and delete content without changing permissions |
| Read and Execute | Permission to read files, list folders, and execute files |
| List Folder | Permission to list folder contents without full read/write authority |
| This folder only | Permission applies only to the current folder |
| This folder, subfolders, and files | Permission applies to the current folder and all child objects |
| Inherit only | Permission applies only to child objects, not the current folder |
| SYSTEM | Local operating system identity that should retain Full Control |
| Administrators | Administrative group that should retain Full Control |
| File server admin group | Delegated admin group for file server operators |
| Resource group | AD group placed directly on NTFS ACLs |
| AGDLP | Accounts into Global groups, Global groups into Domain Local groups, Domain Local groups into Permissions |
| First rule | Back up ACLs before changing inheritance or removing inherited permissions |

# Configure_NTFS_Permissions_And_ACL_Inheritance_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Data drive | `E:` | `<data-drive-letter>` |
| Share root | `E:\Shares` | `<share-root>` |
| Department root | `E:\Shares\Departments` | `<department-root>` |
| Home folder root | `E:\Shares\Home` | `<home-folder-root>` |
| Public root | `E:\Shares\Public` | `<public-share-root>` |
| Apps root | `E:\Shares\Apps` | `<app-share-root>` |
| Sample department | `Accounting` | `<department-name>` |
| Sample department path | `E:\Shares\Departments\Accounting` | `<department-folder-path>` |
| File server admins group | `CORP\GG-FileServer-Admins` | `<file-server-admins-group>` |
| Department read/write group | `CORP\GG-FS-Accounting-RW` | `<department-rw-group>` |
| Department read-only group | `CORP\GG-FS-Accounting-RO` | `<department-ro-group>` |
| Department no-access group | Avoid if possible | `<department-deny-group>` |
| Home users group | `CORP\Domain Users` | `<home-users-group>` |
| ACL method | `icacls` primary, `Get-Acl` validation | `<acl-method>` |
| Inheritance stance at share root | Keep controlled inheritance | `<share-root-inheritance-plan>` |
| Inheritance stance at department folders | Break at department folder boundary | `<department-inheritance-plan>` |
| Inheritance stance at home root | Controlled root, per-user folders later | `<home-inheritance-plan>` |
| Deny permissions stance | Avoid unless required | `<deny-permission-plan>` |
| ACL backup path | `C:\FileServices-Validation\ACL-Backups` | `<acl-backup-path>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Next workbook | `04_Create_SMB_Shares_And_Share_Permissions.md` | `<next-task>` |

# Configure_NTFS_Permissions_And_ACL_Inheritance_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm folder structure exists | File Server | `Test-Path "<share-root>"`; `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"` | Required folders exist |
| 4 | Confirm AD module if validating groups locally | File Server / Admin Host | `Get-Module ActiveDirectory -ListAvailable` | AD group validation method is known |
| 5 | Confirm file server admin group exists | DC / Admin Host | `Get-ADGroup "<file-server-admins-group>"` | Admin group exists |
| 6 | Confirm department RW group exists | DC / Admin Host | `Get-ADGroup "<department-rw-group>"` | RW group exists |
| 7 | Confirm department RO group exists | DC / Admin Host | `Get-ADGroup "<department-ro-group>"` | RO group exists |
| 8 | Capture current ACLs before changes | File Server | `icacls "<share-root>" /save "<acl-backup-path>\share-root-acl.txt" /t /c` | ACL backup exists |
| 9 | Capture readable ACL evidence | File Server | `Get-Acl "<share-root>" \| Format-List` | Current ACL state is documented |
| 10 | Create sample department folder if needed | File Server | `New-Item -ItemType Directory -Path "<department-folder-path>" -Force` | Department folder exists |
| 11 | Apply share root baseline ACL | File Server | Use share root ACL skeleton | Admins and SYSTEM retain Full Control |
| 12 | Apply department root traversal ACL | File Server | Use department ACL skeleton | Users can traverse parent path without broad write access |
| 13 | Break inheritance at department folder boundary | File Server | `icacls "<department-folder-path>" /inheritance:r` | Department folder has explicit ACL boundary |
| 14 | Grant admins Full Control on department folder | File Server | `icacls "<department-folder-path>" /grant "<file-server-admins-group>:(OI)(CI)(F)"` | Admin group has Full Control |
| 15 | Grant RW group Modify on department folder | File Server | `icacls "<department-folder-path>" /grant "<department-rw-group>:(OI)(CI)(M)"` | RW group can create and modify content |
| 16 | Grant RO group Read and Execute on department folder | File Server | `icacls "<department-folder-path>" /grant "<department-ro-group>:(OI)(CI)(RX)"` | RO group can read content |
| 17 | Confirm no user-specific ACLs were added | File Server | `icacls "<department-folder-path>"` | Permissions are group-based |
| 18 | Apply home root baseline ACL | File Server | Use home root ACL skeleton | Home root is prepared for per-user home folders |
| 19 | Validate final ACLs on root paths | File Server | `icacls "<share-root>"`; `icacls "<department-root>"`; `icacls "<home-folder-root>"` | ACLs match planned baseline |
| 20 | Validate final ACL on sample department | File Server | `icacls "<department-folder-path>"` | Department folder has correct explicit ACLs |
| 21 | Test read/write access with RW user if available | Client / File Server | `runas /user:<rw-test-user> cmd` then create test file | RW user can create and modify files |
| 22 | Test read-only access with RO user if available | Client / File Server | `runas /user:<ro-test-user> cmd` then attempt write | RO user can read but cannot write |
| 23 | Test unauthorized user if available | Client / File Server | Attempt folder access | Unauthorized user cannot access protected folder |
| 24 | Export post-change ACL evidence | File Server | `icacls "<share-root>" /save "<acl-backup-path>\share-root-acl-after.txt" /t /c` | Post-change ACL backup exists |
| 25 | Document ACL model | Operator | `Record groups, folder paths, inheritance stance, and validation results` | Permission build record is complete |

# Configure_NTFS_Permissions_And_ACL_Inheritance_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate folder paths, groups, and current ACL state before changes.

$EvidencePath = "C:\FileServices-Validation"
$AclBackupPath = "$EvidencePath\ACL-Backups"

$ShareRoot = "E:\Shares"
$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"
$DepartmentName = "Accounting"
$DepartmentPath = Join-Path $DepartmentRoot $DepartmentName

$FileServerAdminsGroup = "CORP\GG-FileServer-Admins"
$DepartmentRWGroup = "CORP\GG-FS-Accounting-RW"
$DepartmentROGroup = "CORP\GG-FS-Accounting-RO"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $AclBackupPath

Start-Transcript -Path "$EvidencePath\03-ntfs-acl-precheck-transcript.txt"

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-ntfs-acl.txt"

# Confirm server and role state.
hostname |
  Tee-Object "$EvidencePath\hostname-before-ntfs-acl.txt"

Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-ntfs-acl.txt"

# Confirm required folders.
$RequiredPaths = @(
  $ShareRoot,
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $AppsRoot
)

$RequiredPaths |
  ForEach-Object {
    [PSCustomObject]@{
      Path = $_
      Exists = Test-Path $_
    }
  } |
  Tee-Object "$EvidencePath\required-paths-before-ntfs-acl.txt"

# Create sample department folder if it does not exist.
New-Item -ItemType Directory -Force -Path $DepartmentPath |
  Tee-Object "$EvidencePath\department-folder-create-result.txt"

# Capture existing ACLs in readable form.
$AclPaths = @(
  $ShareRoot,
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $AppsRoot,
  $DepartmentPath
)

foreach ($Path in $AclPaths) {
  Get-Acl $Path |
    Format-List |
    Tee-Object "$EvidencePath\acl-before-$($Path.Replace(':','').Replace('\','-')).txt"
}

# Capture existing ACLs in icacls form.
foreach ($Path in $AclPaths) {
  icacls $Path |
    Tee-Object "$EvidencePath\icacls-before-$($Path.Replace(':','').Replace('\','-')).txt"
}

Stop-Transcript
```


Configure_NTFS_Permissions_And_ACL_Inheritance_ACL_Backup_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: save ACLs before making inheritance or permission changes.

$EvidencePath = "C:\FileServices-Validation"
$AclBackupPath = "$EvidencePath\ACL-Backups"

$ShareRoot = "E:\Shares"
$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$DepartmentPath = "E:\Shares\Departments\Accounting"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $AclBackupPath

Start-Transcript -Path "$EvidencePath\03-acl-backup-transcript.txt"

# Save ACLs for rollback.
icacls $ShareRoot /save "$AclBackupPath\share-root-before.txt" /t /c
icacls $DepartmentRoot /save "$AclBackupPath\department-root-before.txt" /t /c
icacls $HomeRoot /save "$AclBackupPath\home-root-before.txt" /t /c

# Save single target folder ACL separately.
icacls $DepartmentPath /save "$AclBackupPath\accounting-folder-before.txt" /t /c

# Confirm backup files exist.
Get-ChildItem $AclBackupPath |
  Tee-Object "$EvidencePath\acl-backup-files.txt"

Stop-Transcript
```

Configure_NTFS_Permissions_And_ACL_Inheritance_Share_Root_ACL_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: apply controlled root ACLs to the top-level share root.
# This creates an administrative baseline, not final per-department access.

$EvidencePath = "C:\FileServices-Validation"
$ShareRoot = "E:\Shares"

$DomainNetbios = "CORP"
$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-share-root-acl-transcript.txt"

# Show current ACL.
icacls $ShareRoot |
  Tee-Object "$EvidencePath\share-root-icacls-before-change.txt"

# Disable inheritance and remove inherited ACEs from share root.
# Keep SYSTEM, Administrators, and delegated file server admins as explicit permissions.
icacls $ShareRoot /inheritance:r

# Grant administrative control.
icacls $ShareRoot /grant "SYSTEM:(OI)(CI)(F)"
icacls $ShareRoot /grant "BUILTIN\Administrators:(OI)(CI)(F)"
icacls $ShareRoot /grant "$FileServerAdminsGroup:(OI)(CI)(F)"

# Grant authenticated users root traversal only.
# No OI/CI flags means this applies to the current folder only.
icacls $ShareRoot /grant "Authenticated Users:(RX)"

# Show final ACL.
icacls $ShareRoot |
  Tee-Object "$EvidencePath\share-root-icacls-after-change.txt"

Get-Acl $ShareRoot |
  Format-List |
  Tee-Object "$EvidencePath\share-root-get-acl-after-change.txt"

Stop-Transcript
```

Configure_NTFS_Permissions_And_ACL_Inheritance_Department_ACL_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: configure NTFS ACLs for department root and one sample department folder.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"
$ShareRoot = "E:\Shares"
$DepartmentRoot = "E:\Shares\Departments"
$DepartmentName = "Accounting"
$DepartmentPath = Join-Path $DepartmentRoot $DepartmentName

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$DepartmentRWGroup = "$DomainNetbios\GG-FS-Accounting-RW"
$DepartmentROGroup = "$DomainNetbios\GG-FS-Accounting-RO"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $DepartmentPath

Start-Transcript -Path "$EvidencePath\03-department-acl-transcript.txt"

# Show current ACLs.
icacls $DepartmentRoot |
  Tee-Object "$EvidencePath\department-root-icacls-before-change.txt"

icacls $DepartmentPath |
  Tee-Object "$EvidencePath\department-folder-icacls-before-change.txt"

# Department root should be traversable but not broadly writable.
icacls $DepartmentRoot /inheritance:r
icacls $DepartmentRoot /grant "SYSTEM:(OI)(CI)(F)"
icacls $DepartmentRoot /grant "BUILTIN\Administrators:(OI)(CI)(F)"
icacls $DepartmentRoot /grant "$FileServerAdminsGroup:(OI)(CI)(F)"
icacls $DepartmentRoot /grant "Authenticated Users:(RX)"

# Department folder becomes the access boundary.
icacls $DepartmentPath /inheritance:r
icacls $DepartmentPath /grant "SYSTEM:(OI)(CI)(F)"
icacls $DepartmentPath /grant "BUILTIN\Administrators:(OI)(CI)(F)"
icacls $DepartmentPath /grant "$FileServerAdminsGroup:(OI)(CI)(F)"
icacls $DepartmentPath /grant "$DepartmentRWGroup:(OI)(CI)(M)"
icacls $DepartmentPath /grant "$DepartmentROGroup:(OI)(CI)(RX)"

# Show final ACLs.
icacls $DepartmentRoot |
  Tee-Object "$EvidencePath\department-root-icacls-after-change.txt"

icacls $DepartmentPath |
  Tee-Object "$EvidencePath\department-folder-icacls-after-change.txt"

Get-Acl $DepartmentPath |
  Format-List |
  Tee-Object "$EvidencePath\department-folder-get-acl-after-change.txt"

Stop-Transcript
```

Configure_NTFS_Permissions_And_ACL_Inheritance_Home_Root_ACL_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: prepare the Home root for later per-user home folder creation.
# Per-user folders are built in task 07.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"
$HomeRoot = "E:\Shares\Home"
$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $HomeRoot

Start-Transcript -Path "$EvidencePath\03-home-root-acl-transcript.txt"

# Show current ACL.
icacls $HomeRoot |
  Tee-Object "$EvidencePath\home-root-icacls-before-change.txt"

# Break inheritance at Home root.
icacls $HomeRoot /inheritance:r

# Administrative permissions.
icacls $HomeRoot /grant "SYSTEM:(OI)(CI)(F)"
icacls $HomeRoot /grant "BUILTIN\Administrators:(OI)(CI)(F)"
icacls $HomeRoot /grant "$FileServerAdminsGroup:(OI)(CI)(F)"

# Allow authenticated users to reach the Home root only.
# Per-user create/list details are refined in the home folder workbook.
icacls $HomeRoot /grant "Authenticated Users:(RX)"

# Creator Owner applies to child folders and files only.
icacls $HomeRoot /grant "CREATOR OWNER:(OI)(CI)(IO)(F)"

# Show final ACL.
icacls $HomeRoot |
  Tee-Object "$EvidencePath\home-root-icacls-after-change.txt"

Get-Acl $HomeRoot |
  Format-List |
  Tee-Object "$EvidencePath\home-root-get-acl-after-change.txt"

Stop-Transcript
```

Configure_NTFS_Permissions_And_ACL_Inheritance_Effective_Access_Test_Skeleton
```
# Run from a test client or the file server.
# Purpose: validate expected access behavior using test accounts.
# Replace users with actual lab accounts.

$DepartmentPath = "E:\Shares\Departments\Accounting"
$TestFile = Join-Path $DepartmentPath "ntfs-acl-test.txt"

$RwTestUser = "CORP\test.accounting.rw"
$RoTestUser = "CORP\test.accounting.ro"
$NoAccessTestUser = "CORP\test.noaccess"

# Local direct-path tests on the file server require logging in or using RunAs.
# For network access, wait until task 04 creates SMB shares.

# Check current token group memberships for the logged-in test account.
whoami
whoami /groups

# Test path visibility.
Test-Path $DepartmentPath

# RW user expected result:
# This should succeed for RW group members.
"NTFS ACL write test $(Get-Date)" | Out-File $TestFile -Force
Get-Content $TestFile
Remove-Item $TestFile -Force

# RO user expected result:
# This should allow read/list but fail write.
Get-ChildItem $DepartmentPath
"RO write test $(Get-Date)" | Out-File $TestFile -Force

# No-access user expected result:
# This should fail to list or access the protected department folder.
Get-ChildItem $DepartmentPath
```

Configure_NTFS_Permissions_And_ACL_Inheritance_Post_Change_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: capture final ACL state after NTFS permission changes.

$EvidencePath = "C:\FileServices-Validation"
$AclBackupPath = "$EvidencePath\ACL-Backups"

$AclPaths = @(
  "E:\Shares",
  "E:\Shares\Departments",
  "E:\Shares\Departments\Accounting",
  "E:\Shares\Home",
  "E:\Shares\Public",
  "E:\Shares\Apps"
)

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $AclBackupPath

Start-Transcript -Path "$EvidencePath\03-post-change-validation-transcript.txt"

# Save post-change ACLs for documentation.
icacls "E:\Shares" /save "$AclBackupPath\share-root-after.txt" /t /c

# Capture readable ACLs.
foreach ($Path in $AclPaths) {
  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-after-$($Path.Replace(':','').Replace('\','-')).txt"

    Get-Acl $Path |
      Format-List |
      Tee-Object "$EvidencePath\get-acl-after-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Export ACL summaries.
$AclSummary = foreach ($Path in $AclPaths) {
  if (Test-Path $Path) {
    $Acl = Get-Acl $Path
    [PSCustomObject]@{
      Path = $Path
      Owner = $Acl.Owner
      Group = $Acl.Group
      Access = $Acl.AccessToString
    }
  }
}

$AclSummary |
  Export-Csv "$EvidencePath\ntfs-acl-summary-after.csv" -NoTypeInformation

# Confirm no SMB shares were created in this task.
Get-SmbShare |
  Sort-Object Name |
  Tee-Object "$EvidencePath\smb-shares-after-ntfs-acl-task.txt"

Stop-Transcript
```


Configure_NTFS_Permissions_And_ACL_Inheritance_Verification_Commands
```
# Required paths
Test-Path "<share-root>"
Test-Path "<department-root>"
Test-Path "<department-folder-path>"
Test-Path "<home-folder-root>"

# ACL verification
icacls "<share-root>"
icacls "<department-root>"
icacls "<department-folder-path>"
icacls "<home-folder-root>"

Get-Acl "<share-root>" | Format-List
Get-Acl "<department-root>" | Format-List
Get-Acl "<department-folder-path>" | Format-List
Get-Acl "<home-folder-root>" | Format-List

# Group validation from a DC or admin host with AD module
Get-ADGroup "<file-server-admins-group>"
Get-ADGroup "<department-rw-group>"
Get-ADGroup "<department-ro-group>"

# User membership validation
Get-ADPrincipalGroupMembership "<test-user>" | Select-Object Name
whoami /groups

# Basic direct path tests from the file server
Get-ChildItem "<department-folder-path>"
New-Item -ItemType File -Path "<department-folder-path>\acl-test.txt"
Remove-Item "<department-folder-path>\acl-test.txt"

# Confirm SMB shares are not the scope of this workbook
Get-SmbShare

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>\ACL-Backups"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*acl*"
```

Configure_NTFS_Permissions_And_ACL_Inheritance_Rollback

|   |   |   |   |   |
|---|---|---|---|---|
|Step|Task|Device|PowerShell / Command|Expected Result|
|1|Confirm rollback backup exists|File Server|Get-ChildItem "<acl-backup-path>"|ACL backup files are visible|
|2|Stop active access testing|Client / File Server|Close open files and test sessions|No active test writes continue|
|3|Review current ACL before rollback|File Server|icacls "<folder-path>"|Current permission state is visible|
|4|Restore share root ACL from backup|File Server|icacls "<restore-parent-path>" /restore "<acl-backup-path>\share-root-before.txt"|Prior ACLs are restored|
|5|Restore department root ACL from backup|File Server|icacls "<restore-parent-path>" /restore "<acl-backup-path>\department-root-before.txt"|Prior department root ACLs are restored|
|6|Restore home root ACL from backup|File Server|icacls "<restore-parent-path>" /restore "<acl-backup-path>\home-root-before.txt"|Prior home root ACLs are restored|
|7|Re-enable inheritance if rollback requires default inheritance|File Server|icacls "<folder-path>" /inheritance:e|Folder inherits from parent again|
|8|Remove incorrect explicit ACE|File Server|icacls "<folder-path>" /remove "<account-or-group>"|Incorrect group is removed|
|9|Grant emergency admin access if locked out|File Server console|takeown /f "<folder-path>" /a /r /d y; icacls "<folder-path>" /grant "BUILTIN\Administrators:(OI)(CI)(F)" /t|Administrators regain control|
|10|Validate ACL after rollback|File Server|icacls "<folder-path>"; Get-Acl "<folder-path>"|ACL matches intended rollback state|
|11|Document rollback result|Operator|Record restored backup file, folder path, and validation output|Rollback record is complete|
Configure_NTFS_Permissions_And_ACL_Inheritance_Failure_Checks

|   |   |   |   |
|---|---|---|---|
|Symptom|Likely Cause|Check|Corrective Action|
|Access denied while changing ACL|PowerShell not elevated or admin lacks rights|whoami /groups; Get-Acl "<folder-path>"|Use elevated PowerShell or administrative account|
|Admin locked out of folder|Inheritance removed and admin ACE missing|File server console, icacls "<folder-path>"|Use ownership recovery and restore admin Full Control|
|User has access unexpectedly|User belongs to another allowed group|whoami /groups; Get-ADPrincipalGroupMembership <user>|Remove user from extra group or adjust ACL|
|User has access unexpectedly|Parent folder still grants inherited access|icacls "<folder-path>"|Break inheritance at correct boundary and remove broad inherited ACE|
|User cannot access folder|User missing required AD group|Get-ADPrincipalGroupMembership <user>|Add user to correct group and refresh logon token|
|User cannot access folder|User has not signed out after group change|whoami /groups|Sign out and sign back in or purge Kerberos tickets|
|RO user can write|RO user is also in RW group|whoami /groups|Remove overlapping group membership|
|RW user cannot write|Modify permission missing or applied to wrong folder|icacls "<department-folder-path>"|Grant RW group (OI)(CI)(M) on correct folder|
|New files do not inherit permissions|OI/CI inheritance flags missing|icacls "<folder-path>"|Reapply ACE with (OI)(CI) flags|
|Child folders have old permissions|Existing child ACLs were not updated|icacls "<folder-path>" /t|Reapply permissions recursively after confirming design|
|Permission command fails on group name|Group name typo or wrong domain prefix|Get-ADGroup "<group-name>"|Correct group name and rerun|
|Get-ADGroup not recognized|AD module missing on file server|Get-Module ActiveDirectory -ListAvailable|Run from DC/admin host or install RSAT AD tools|
|icacls /restore does not restore as expected|Restore run from wrong parent path|Review saved ACL file paths|Run restore from the parent path matching saved ACL structure|
|Deny permission blocks intended access|Deny ACE overrides allow ACE|icacls "<folder-path>"|Remove deny ACE unless strictly required|
|Home root lets users see other names|Home root read/list too broad|icacls "<home-folder-root>"|Refine home folder model in task 07|
|Direct file server test works but UNC test fails later|Share permissions or SMB settings differ|Get-SmbShareAccess after task 04|Fix share permissions in SMB workbook|
|ACL evidence missing|Backup skeleton not run or path wrong|Test-Path "<acl-backup-path>"|Re-run ACL backup before further changes|
