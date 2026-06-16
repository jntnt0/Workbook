05_Configure_Access_Based_Enumeration_And_Share_Visibility.md
# Configure_Access_Based_Enumeration_And_Share_Visibility

# Configure_Access_Based_Enumeration_And_Share_Visibility_Index
05_Configure_Access_Based_Enumeration_And_Share_Visibility.md
Configure_Access_Based_Enumeration_And_Share_Visibility
Configure_Access_Based_Enumeration_And_Share_Visibility_Source_Basis
Configure_Access_Based_Enumeration_And_Share_Visibility_Mental_Model
Configure_Access_Based_Enumeration_And_Share_Visibility_Planning_Table
Configure_Access_Based_Enumeration_And_Share_Visibility_Configuration_Checklist
Configure_Access_Based_Enumeration_And_Share_Visibility_Precheck_Skeleton
Configure_Access_Based_Enumeration_And_Share_Visibility_Enable_ABE_Skeleton
Configure_Access_Based_Enumeration_And_Share_Visibility_Hidden_Share_Skeleton
Configure_Access_Based_Enumeration_And_Share_Visibility_Client_Browse_Test_Skeleton
Configure_Access_Based_Enumeration_And_Share_Visibility_Post_Change_Validation_Skeleton
Configure_Access_Based_Enumeration_And_Share_Visibility_Verification_Commands
Configure_Access_Based_Enumeration_And_Share_Visibility_Rollback
Configure_Access_Based_Enumeration_And_Share_Visibility_Failure_Checks
Configure_Access_Based_Enumeration_And_Share_Visibility_Related_Labs

# Configure_Access_Based_Enumeration_And_Share_Visibility_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Set-SmbShare | Configuring SMB share properties such as FolderEnumerationMode |
| Microsoft Learn | Get-SmbShare | Validating share names, paths, and enumeration settings |
| Microsoft Learn | New-SmbShare | Creating visible and hidden SMB shares |
| Microsoft Learn | Remove-SmbShare | Removing incorrectly created visible or hidden shares |
| Microsoft Learn | Get-SmbShareAccess | Confirming share permissions that combine with ABE behavior |
| Microsoft Learn | NTFS permissions | ABE depends on NTFS permissions to determine what users can see |
| Microsoft Learn | SMBShare PowerShell module | Managing SMB share visibility through PowerShell |
| Windows Server operational practice | Access Based Enumeration | Hiding files and folders from users who do not have NTFS access |
| Windows Server operational practice | Hidden shares | Using `$` suffix to hide a share from casual browsing without changing permissions |
| Windows Server operational practice | Least privilege visibility | Reducing user confusion and information disclosure while keeping access control enforced by NTFS |

# Configure_Access_Based_Enumeration_And_Share_Visibility_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Access Based Enumeration | SMB feature that hides files and folders a user does not have permission to access |
| FolderEnumerationMode | SMB share property that controls whether ABE is enabled |
| AccessBased | Folder enumeration mode where inaccessible folders are hidden from users |
| Unrestricted | Folder enumeration mode where users can see folder names even if they cannot open them |
| Share visibility | Whether a share appears when browsing a server with tools like `net view \\FS1` |
| Hidden share | SMB share whose name ends in `$`, such as `Departments$` |
| Browsing | Listing shares or folders without typing the exact full UNC path |
| Direct UNC access | Accessing a share or folder by full path, such as `\\FS1\Departments\Accounting` |
| Information disclosure | Exposing names of folders, departments, projects, or shares to users who do not need to know they exist |
| ABE dependency | ABE only works correctly when NTFS permissions are already correct |
| Hidden share limitation | `$` hides the share from casual browsing but does not protect the share by itself |
| NTFS enforcement | Actual folder access is still enforced by NTFS permissions |
| Share permission enforcement | SMB share permissions still apply before NTFS permissions |
| User token | User group membership evaluated during access. Stale tokens can cause visibility testing confusion |
| Department share | ABE usually enabled so users only see departments they can access |
| Home share | ABE usually enabled so users do not browse other users' folders |
| Public share | ABE may be disabled if the share is intentionally visible to all users |
| Apps share | ABE depends on design. It may be visible to all users or restricted to IT/application groups |
| First rule | ABE is not a security substitute. It is visibility control layered on top of correct NTFS and SMB permissions |

# Configure_Access_Based_Enumeration_And_Share_Visibility_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Department share name | `Departments` | `<department-share-name>` |
| Home share name | `Home` | `<home-share-name>` |
| Public share name | `Public` | `<public-share-name>` |
| Apps share name | `Apps` | `<apps-share-name>` |
| Optional hidden department share | `Departments$` | `<hidden-department-share-name>` |
| Optional hidden home share | `Home$` | `<hidden-home-share-name>` |
| Department local path | `E:\Shares\Departments` | `<department-root>` |
| Home local path | `E:\Shares\Home` | `<home-folder-root>` |
| Public local path | `E:\Shares\Public` | `<public-share-root>` |
| Apps local path | `E:\Shares\Apps` | `<app-share-root>` |
| ABE stance for Departments | Enabled | `<department-abe-plan>` |
| ABE stance for Home | Enabled | `<home-abe-plan>` |
| ABE stance for Public | Optional, usually Unrestricted | `<public-abe-plan>` |
| ABE stance for Apps | Optional | `<apps-abe-plan>` |
| Hidden share stance | Use `$` only where browsing should be suppressed | `<hidden-share-plan>` |
| Share permission model | Broad share, strict NTFS | `<share-permission-model>` |
| NTFS dependency | Task 03 complete | `<ntfs-validation-state>` |
| SMB share dependency | Task 04 complete | `<smb-share-validation-state>` |
| RW test user | `CORP\test.accounting.rw` | `<rw-test-user>` |
| RO test user | `CORP\test.accounting.ro` | `<ro-test-user>` |
| No-access test user | `CORP\test.noaccess` | `<no-access-test-user>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Client evidence path | `C:\FileServices-Validation-Client` | `<client-evidence-path>` |
| Next workbook | `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md` | `<next-task>` |

# Configure_Access_Based_Enumeration_And_Share_Visibility_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm SMB server service is running | File Server | `Get-Service LanmanServer` | Server service is Running |
| 4 | Confirm shares from task 04 exist | File Server | `Get-SmbShare` | Department, Home, Public, and Apps shares are visible |
| 5 | Confirm local paths exist | File Server | `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"` | Local paths exist |
| 6 | Confirm NTFS permissions before ABE | File Server | `icacls "<department-root>"`; `icacls "<home-folder-root>"` | Folder ACLs are already configured |
| 7 | Confirm share permissions before ABE | File Server | `Get-SmbShareAccess -Name "<share-name>"` | Share permissions are already configured |
| 8 | Capture current share visibility settings | File Server | `Get-SmbShare \| Select Name,Path,FolderEnumerationMode,Special` | Current ABE state is documented |
| 9 | Enable ABE on Departments share | File Server | `Set-SmbShare -Name "<department-share-name>" -FolderEnumerationMode AccessBased -Force` | Inaccessible department folders are hidden |
| 10 | Enable ABE on Home share | File Server | `Set-SmbShare -Name "<home-share-name>" -FolderEnumerationMode AccessBased -Force` | Inaccessible home folders are hidden |
| 11 | Decide Public share enumeration mode | File Server | `Set-SmbShare -Name "<public-share-name>" -FolderEnumerationMode Unrestricted -Force` | Public share remains browseable if intended |
| 12 | Decide Apps share enumeration mode | File Server | `Set-SmbShare -Name "<apps-share-name>" -FolderEnumerationMode AccessBased -Force` | Apps folder visibility matches NTFS access |
| 13 | Create hidden share if required | File Server | `New-SmbShare -Name "<hidden-share-name>" -Path "<folder-path>"` | Hidden share exists and ends with `$` |
| 14 | Confirm hidden share does not appear in casual browse | Client | `net view \\<file-server-name>` | Hidden `$` share is not listed |
| 15 | Confirm hidden share is reachable by exact UNC if authorized | Client | `Test-Path "\\<file-server-name>\<hidden-share-name>"` | Authorized user can reach exact hidden UNC |
| 16 | Test Departments share as RW user | Client | `Get-ChildItem "\\<file-server-name>\<department-share-name>"` | RW user sees only accessible folders |
| 17 | Test Departments share as RO user | Client | `Get-ChildItem "\\<file-server-name>\<department-share-name>"` | RO user sees only accessible folders |
| 18 | Test Departments share as no-access user | Client | `Get-ChildItem "\\<file-server-name>\<department-share-name>"` | No-access user does not see protected department folder |
| 19 | Test Home share visibility | Client | `Get-ChildItem "\\<file-server-name>\<home-share-name>"` | User sees only permitted home folder content |
| 20 | Confirm direct unauthorized folder path is still denied | Client | `Test-Path "\\<file-server-name>\<department-share-name>\<protected-folder>"` | Unauthorized direct access fails |
| 21 | Export final share settings | File Server | `Get-SmbShare \| Export-Csv "<evidence-path>\smb-share-visibility-after.csv" -NoTypeInformation` | Share visibility evidence is saved |
| 22 | Export final share permissions | File Server | `Get-SmbShareAccess -Name "<share-name>"` | Share permissions evidence is saved |
| 23 | Capture client browse evidence | Client | `net view \\<file-server-name>` | Client-visible share list is documented |
| 24 | Document visibility model | Operator | `Record ABE shares, hidden shares, visible shares, and test users` | Share visibility record is complete |

# Configure_Access_Based_Enumeration_And_Share_Visibility_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture SMB share, share permission, NTFS, and visibility state before ABE changes.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentShareName = "Departments"
$HomeShareName = "Home"
$PublicShareName = "Public"
$AppsShareName = "Apps"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"

$ManagedShares = @(
  $DepartmentShareName,
  $HomeShareName,
  $PublicShareName,
  $AppsShareName
)

$ManagedPaths = @(
  $DepartmentRoot,
  $HomeRoot,
  $PublicRoot,
  $AppsRoot
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-abe-share-visibility-precheck-transcript.txt"

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-abe.txt"

# Confirm server and File Server role.
hostname |
  Tee-Object "$EvidencePath\hostname-before-abe.txt"

Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-abe.txt"

# Confirm SMB services.
Get-Service LanmanServer,LanmanWorkstation |
  Tee-Object "$EvidencePath\smb-services-before-abe.txt"

# Capture current SMB share settings.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,Special |
  Tee-Object "$EvidencePath\smb-share-settings-before-abe.txt"

Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,Special |
  Export-Csv "$EvidencePath\smb-share-settings-before-abe.csv" -NoTypeInformation

# Capture managed share permissions.
foreach ($Share in $ManagedShares) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\smb-share-access-before-abe-$Share.txt"
}

# Confirm local paths and NTFS ACLs.
foreach ($Path in $ManagedPaths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\abe-local-path-validation-before.txt" -Append

  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-abe-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

Stop-Transcript
```

Configure_Access_Based_Enumeration_And_Share_Visibility_Enable_ABE_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: configure Access Based Enumeration on selected SMB shares.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentShareName = "Departments"
$HomeShareName = "Home"
$PublicShareName = "Public"
$AppsShareName = "Apps"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-enable-abe-transcript.txt"

# Capture current state.
Get-SmbShare |
  Where-Object { $_.Name -in @($DepartmentShareName,$HomeShareName,$PublicShareName,$AppsShareName) } |
  Select-Object Name,Path,FolderEnumerationMode,Special |
  Tee-Object "$EvidencePath\managed-share-folder-enumeration-before.txt"

# Enable ABE on user and department shares.
Set-SmbShare `
  -Name $DepartmentShareName `
  -FolderEnumerationMode AccessBased `
  -Force

Set-SmbShare `
  -Name $HomeShareName `
  -FolderEnumerationMode AccessBased `
  -Force

# Public share is intentionally browseable in many labs.
# Change to AccessBased if public subfolders have different NTFS permissions.
Set-SmbShare `
  -Name $PublicShareName `
  -FolderEnumerationMode Unrestricted `
  -Force

# Apps can be AccessBased if only some users should see some app folders.
Set-SmbShare `
  -Name $AppsShareName `
  -FolderEnumerationMode AccessBased `
  -Force

# Confirm final state.
Get-SmbShare |
  Where-Object { $_.Name -in @($DepartmentShareName,$HomeShareName,$PublicShareName,$AppsShareName) } |
  Select-Object Name,Path,FolderEnumerationMode,Special |
  Tee-Object "$EvidencePath\managed-share-folder-enumeration-after.txt"

Stop-Transcript
```

Configure_Access_Based_Enumeration_And_Share_Visibility_Hidden_Share_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: optionally create hidden SMB shares using the $ suffix.
# Hidden shares are not security controls. NTFS and share permissions still enforce access.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"
$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$AuthenticatedUsers = "Authenticated Users"

$HiddenDepartmentShareName = "Departments$"
$HiddenHomeShareName = "Home$"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-hidden-share-transcript.txt"

# Create hidden Departments share if required.
if (-not (Get-SmbShare -Name $HiddenDepartmentShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $HiddenDepartmentShareName `
    -Path $DepartmentRoot `
    -Description "Hidden department share" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ChangeAccess $AuthenticatedUsers `
    -FolderEnumerationMode AccessBased `
    -CachingMode Manual
}

# Create hidden Home share if required.
if (-not (Get-SmbShare -Name $HiddenHomeShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $HiddenHomeShareName `
    -Path $HomeRoot `
    -Description "Hidden home folder share" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ChangeAccess $AuthenticatedUsers `
    -FolderEnumerationMode AccessBased `
    -CachingMode Manual
}

# Confirm hidden share settings.
Get-SmbShare |
  Where-Object { $_.Name -in @($HiddenDepartmentShareName,$HiddenHomeShareName) } |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,Special |
  Tee-Object "$EvidencePath\hidden-shares-after-create.txt"

foreach ($Share in @($HiddenDepartmentShareName,$HiddenHomeShareName)) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\hidden-share-access-$Share.txt"
}

Stop-Transcript
```

Configure_Access_Based_Enumeration_And_Share_Visibility_Client_Browse_Test_Skeleton
```
# Run from a Windows client as different test users.
# Purpose: validate visible shares, hidden shares, and ABE behavior from the client side.

$FileServerName = "FS1"

$DepartmentShareName = "Departments"
$HomeShareName = "Home"
$PublicShareName = "Public"
$AppsShareName = "Apps"

$HiddenDepartmentShareName = "Departments$"
$HiddenHomeShareName = "Home$"

$DepartmentUNC = "\\$FileServerName\$DepartmentShareName"
$HomeUNC = "\\$FileServerName\$HomeShareName"
$PublicUNC = "\\$FileServerName\$PublicShareName"
$AppsUNC = "\\$FileServerName\$AppsShareName"

$HiddenDepartmentUNC = "\\$FileServerName\$HiddenDepartmentShareName"
$HiddenHomeUNC = "\\$FileServerName\$HiddenHomeShareName"

$EvidencePath = "C:\FileServices-Validation-Client"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-client-abe-browse-test-transcript.txt"

# Capture client identity and token.
hostname |
  Tee-Object "$EvidencePath\client-hostname-abe-test.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami-abe-test.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups-abe-test.txt"

# Confirm name resolution and SMB reachability.
Resolve-DnsName $FileServerName |
  Tee-Object "$EvidencePath\client-resolve-file-server-abe-test.txt"

Test-NetConnection $FileServerName -Port 445 |
  Tee-Object "$EvidencePath\client-test-smb-445-abe-test.txt"

# Browse visible shares.
net view "\\$FileServerName" |
  Tee-Object "$EvidencePath\net-view-file-server-visible-shares.txt"

# Test visible share paths.
$VisibleUNCs = @(
  $DepartmentUNC,
  $HomeUNC,
  $PublicUNC,
  $AppsUNC
)

foreach ($UNC in $VisibleUNCs) {
  [PSCustomObject]@{
    UNC = $UNC
    Exists = Test-Path $UNC
  } |
    Tee-Object "$EvidencePath\visible-unc-testpath-results.txt" -Append

  Get-ChildItem $UNC -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\listing-$($UNC.Replace('\','-').Replace('$','hidden')).txt"
}

# Hidden shares should not show in net view, but should work by exact UNC if authorized.
$HiddenUNCs = @(
  $HiddenDepartmentUNC,
  $HiddenHomeUNC
)

foreach ($UNC in $HiddenUNCs) {
  [PSCustomObject]@{
    UNC = $UNC
    ExactPathExistsForThisUser = Test-Path $UNC
  } |
    Tee-Object "$EvidencePath\hidden-unc-testpath-results.txt" -Append

  Get-ChildItem $UNC -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\hidden-listing-$($UNC.Replace('\','-').Replace('$','hidden')).txt"
}

Stop-Transcript
```

Configure_Access_Based_Enumeration_And_Share_Visibility_Post_Change_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: capture final share visibility and ABE state.

$EvidencePath = "C:\FileServices-Validation"

$ManagedShares = @(
  "Departments",
  "Home",
  "Public",
  "Apps",
  "Departments$",
  "Home$"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-post-change-abe-validation-transcript.txt"

# Capture final share settings.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,Special |
  Tee-Object "$EvidencePath\smb-share-visibility-final.txt"

Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,Special |
  Export-Csv "$EvidencePath\smb-share-visibility-final.csv" -NoTypeInformation

# Capture managed share permissions.
foreach ($Share in $ManagedShares) {
  Get-SmbShare -Name $Share -ErrorAction SilentlyContinue |
    Select-Object Name,Path,Description,FolderEnumerationMode,Special |
    Tee-Object "$EvidencePath\managed-share-final-$($Share.Replace('$','hidden')).txt"

  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\managed-share-access-final-$($Share.Replace('$','hidden')).txt"
}

# Confirm local paths still exist and NTFS permissions remain in place.
foreach ($Share in $ManagedShares) {
  $SmbShare = Get-SmbShare -Name $Share -ErrorAction SilentlyContinue

  if ($SmbShare) {
    [PSCustomObject]@{
      ShareName = $SmbShare.Name
      Path = $SmbShare.Path
      PathExists = Test-Path $SmbShare.Path
      FolderEnumerationMode = $SmbShare.FolderEnumerationMode
    } |
      Tee-Object "$EvidencePath\managed-share-path-and-abe-final.txt" -Append

    icacls $SmbShare.Path |
      Tee-Object "$EvidencePath\icacls-for-share-final-$($Share.Replace('$','hidden')).txt"
  }
}

# Capture current SMB sessions and open files.
Get-SmbSession |
  Tee-Object "$EvidencePath\smb-sessions-after-abe.txt"

Get-SmbOpenFile |
  Tee-Object "$EvidencePath\smb-open-files-after-abe.txt"

Stop-Transcript
```

Configure_Access_Based_Enumeration_And_Share_Visibility_Verification_Commands
```
# Server-side role and SMB state
Get-WindowsFeature FS-FileServer
Get-Service LanmanServer
Get-SmbShare
Get-SmbShare | Select-Object Name,Path,FolderEnumerationMode,Special

# Specific share visibility settings
Get-SmbShare -Name "<department-share-name>" | Select-Object Name,Path,FolderEnumerationMode
Get-SmbShare -Name "<home-share-name>" | Select-Object Name,Path,FolderEnumerationMode
Get-SmbShare -Name "<public-share-name>" | Select-Object Name,Path,FolderEnumerationMode
Get-SmbShare -Name "<apps-share-name>" | Select-Object Name,Path,FolderEnumerationMode

# Share permissions
Get-SmbShareAccess -Name "<department-share-name>"
Get-SmbShareAccess -Name "<home-share-name>"
Get-SmbShareAccess -Name "<public-share-name>"
Get-SmbShareAccess -Name "<apps-share-name>"

# NTFS dependency checks
icacls "<department-root>"
icacls "<home-folder-root>"
icacls "<public-share-root>"
icacls "<app-share-root>"

# Enable ABE
Set-SmbShare -Name "<department-share-name>" -FolderEnumerationMode AccessBased -Force
Set-SmbShare -Name "<home-share-name>" -FolderEnumerationMode AccessBased -Force

# Disable ABE
Set-SmbShare -Name "<share-name>" -FolderEnumerationMode Unrestricted -Force

# Hidden shares
Get-SmbShare -Name "<hidden-share-name>"
Get-SmbShareAccess -Name "<hidden-share-name>"

# Client-side browse and access
net view \\<file-server-name>
Test-Path "\\<file-server-name>\<department-share-name>"
Test-Path "\\<file-server-name>\<hidden-share-name>"
Get-ChildItem "\\<file-server-name>\<department-share-name>"
Get-ChildItem "\\<file-server-name>\<home-share-name>"

# Network path
Resolve-DnsName "<file-server-name>"
Test-NetConnection "<file-server-name>" -Port 445

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*abe*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*visibility*"
```

Configure_Access_Based_Enumeration_And_Share_Visibility_Rollback

|   |   |   |   |   |
|---|---|---|---|---|
|Step|Task|Device|PowerShell / Command|Expected Result|
|1|Capture current share visibility state|File Server|Get-SmbShare \| Select Name,Path,FolderEnumerationMode,Special|Current visibility state is saved|
|2|Disable ABE on a share if required|File Server|Set-SmbShare -Name "<share-name>" -FolderEnumerationMode Unrestricted -Force|Share returns to unrestricted folder enumeration|
|3|Re-enable ABE if rollback target requires it|File Server|Set-SmbShare -Name "<share-name>" -FolderEnumerationMode AccessBased -Force|Share hides inaccessible folders again|
|4|Remove incorrect hidden share|File Server|Remove-SmbShare -Name "<hidden-share-name>" -Force|Hidden share is removed|
|5|Recreate visible share if hidden share replaced it incorrectly|File Server|New-SmbShare -Name "<visible-share-name>" -Path "<folder-path>" -ChangeAccess "Authenticated Users"|Visible share is restored|
|6|Recreate hidden share if needed|File Server|New-SmbShare -Name "<hidden-share-name>" -Path "<folder-path>" -ChangeAccess "Authenticated Users"|Hidden share is restored|
|7|Confirm share permissions after rollback|File Server|Get-SmbShareAccess -Name "<share-name>"|Share permissions match expected state|
|8|Confirm NTFS ACLs were not changed|File Server|icacls "<folder-path>"|NTFS permissions remain intact|
|9|Validate client browse state|Client|net view \\<file-server-name>|Visible and hidden shares behave as expected|
|10|Validate direct UNC access|Client|Test-Path "\\<file-server-name>\<share-name>"|Authorized users can still access exact UNC path|
|11|Document rollback result|Operator|Record restored FolderEnumerationMode, removed shares, and client test result|Rollback record is complete|


Configure_Access_Based_Enumeration_And_Share_Visibility_Failure_Checks

|                                              |                                                                         |                                                        |                                                          |
| -------------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------ | -------------------------------------------------------- |
| Symptom                                      | Likely Cause                                                            | Check                                                  | Corrective Action                                        |
| ABE setting does not hide folders            | NTFS permissions still allow list/read access                           | icacls "<folder-path>"; whoami /groups                 | Correct NTFS ACLs from task 03                           |
| ABE appears not to work during test          | User token still has old group membership                               | whoami /groups                                         | Sign out and sign back in or purge Kerberos tickets      |
| User still sees department folder            | User belongs to RW or RO group for that folder                          | Get-ADPrincipalGroupMembership <user>                  | Remove unintended group membership                       |
| User cannot see folder they should see       | Missing NTFS permission or wrong group                                  | icacls "<folder-path>"; whoami /groups                 | Add user to correct group or fix ACL                     |
| User cannot access exact UNC path            | Share permission or NTFS permission blocks access                       | Get-SmbShareAccess; icacls                             | Correct share and NTFS permissions                       |
| Hidden share appears in Get-SmbShare         | Server-side tools show all shares                                       | Get-SmbShare                                           | Normal behavior. Test hidden shares with client net view |
| Hidden share appears in client browse        | Share name does not end in $                                            | Get-SmbShare -Name "<share-name>"                      | Recreate share with $ suffix                             |
| Hidden share is inaccessible by direct path  | User lacks share or NTFS permission                                     | Get-SmbShareAccess; icacls                             | Grant correct group access                               |
| Set-SmbShare fails                           | PowerShell not elevated                                                 | whoami /groups                                         | Reopen PowerShell as Administrator                       |
| Set-SmbShare fails                           | Share name typo                                                         | Get-SmbShare                                           | Use exact share name                                     |
| New-SmbShare hidden share fails              | Share name already exists                                               | Get-SmbShare -Name "<hidden-share-name>"               | Remove or reuse existing hidden share                    |
| Client cannot browse server at all           | SMB port blocked or name resolution issue                               | Resolve-DnsName; Test-NetConnection <server> -Port 445 | Fix DNS or SMB firewall path                             |
| net view shows access denied                 | User lacks permission to enumerate server or SMB policy blocks browsing | Client credential and SMB policy checks                | Test direct UNC and validate share permissions           |
| Public share hides folders unexpectedly      | Public share set to AccessBased with restrictive NTFS                   | Get-SmbShare -Name Public; icacls                      | Set Public to Unrestricted or adjust NTFS design         |
| Apps share exposes folder names unexpectedly | Apps share set to Unrestricted                                          | Get-SmbShare -Name Apps                                | Set Apps share to AccessBased if needed                  |
| ABE confused with security                   | Hidden names mistaken for denied access                                 | Direct UNC and ACL tests                               | Treat ABE as visibility only, not access control         |
| Evidence missing                             | Validation skeleton not run                                             | Test-Path "<evidence-path>"                            | Re-run precheck or post-change validation skeleton       |