04_Create_SMB_Shares_And_Share_Permissions.md
# Create_SMB_Shares_And_Share_Permissions

# Create_SMB_Shares_And_Share_Permissions_Index
04_Create_SMB_Shares_And_Share_Permissions.md
Create_SMB_Shares_And_Share_Permissions
Create_SMB_Shares_And_Share_Permissions_Source_Basis
Create_SMB_Shares_And_Share_Permissions_Mental_Model
Create_SMB_Shares_And_Share_Permissions_Planning_Table
Create_SMB_Shares_And_Share_Permissions_Configuration_Checklist
Create_SMB_Shares_And_Share_Permissions_Precheck_Skeleton
Create_SMB_Shares_And_Share_Permissions_Create_Core_Shares_Skeleton
Create_SMB_Shares_And_Share_Permissions_Share_Permissions_Skeleton
Create_SMB_Shares_And_Share_Permissions_Client_Access_Test_Skeleton
Create_SMB_Shares_And_Share_Permissions_Post_Change_Validation_Skeleton
Create_SMB_Shares_And_Share_Permissions_Verification_Commands
Create_SMB_Shares_And_Share_Permissions_Rollback
Create_SMB_Shares_And_Share_Permissions_Failure_Checks
Create_SMB_Shares_And_Share_Permissions_Related_Labs

# Create_SMB_Shares_And_Share_Permissions_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | New-SmbShare | Creating SMB shares from prepared folder paths |
| Microsoft Learn | Get-SmbShare | Validating existing and newly created SMB shares |
| Microsoft Learn | Set-SmbShare | Updating SMB share properties after creation |
| Microsoft Learn | Remove-SmbShare | Removing incorrect or lab SMB shares |
| Microsoft Learn | Grant-SmbShareAccess | Assigning SMB share permissions |
| Microsoft Learn | Revoke-SmbShareAccess | Removing incorrect SMB share permissions |
| Microsoft Learn | Get-SmbShareAccess | Reviewing share-level permissions |
| Microsoft Learn | SMBShare PowerShell module | Managing Windows SMB shares through PowerShell |
| Microsoft Learn | File and Storage Services | Publishing Windows Server file shares |
| Windows Server operational practice | Share plus NTFS permissions | Final access is the most restrictive result of share permissions and NTFS permissions |
| Windows Server operational practice | Least privilege access | Use groups, avoid user-specific share ACLs, and let NTFS enforce folder-level access |

# Create_SMB_Shares_And_Share_Permissions_Mental_Model
| Concept | Operational Meaning |
|---|---|
| SMB share | Network publication of a local folder path |
| Share name | Name users access through a UNC path such as `\\FS1\Departments` |
| Hidden share | Share ending in `$`, such as `Departments$`, not shown during casual browsing |
| UNC path | Network path format such as `\\FS1\Departments` |
| Local path | Server-side folder path such as `E:\Shares\Departments` |
| Share permissions | Network boundary permissions on the SMB share |
| NTFS permissions | File system permissions on folders and files |
| Effective access | Final access after SMB share permission and NTFS permission are both evaluated |
| Full share access | SMB permission that allows full share-level control |
| Change share access | SMB permission that allows read, write, create, modify, and delete through the share |
| Read share access | SMB permission that allows read-only access through the share |
| Broad share, strict NTFS | Common model where share permissions are broad enough and NTFS permissions enforce detailed access |
| Group-based access | Share permissions are assigned to AD groups, not individual users |
| Administrative share access | File server admins retain Full Control for management |
| User-facing share | Share intended for normal users, such as department or home shares |
| Administrative share | Share intended for IT operations or hidden root paths |
| Caching mode | Offline Files behavior for a share |
| Continuous availability | SMB property used with clustered file servers, not a normal standalone share setting |
| Access Based Enumeration | Hides inaccessible files/folders. Configured in task 05 |
| SMB encryption/signing/auditing | Security and audit posture. Configured in task 06 |
| First rule | Do not publish SMB shares until local folder paths and NTFS ACLs are already validated |

# Create_SMB_Shares_And_Share_Permissions_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Data drive | `E:` | `<data-drive-letter>` |
| Share root | `E:\Shares` | `<share-root>` |
| Department root local path | `E:\Shares\Departments` | `<department-root>` |
| Home root local path | `E:\Shares\Home` | `<home-folder-root>` |
| Public root local path | `E:\Shares\Public` | `<public-share-root>` |
| Apps root local path | `E:\Shares\Apps` | `<app-share-root>` |
| Department share name | `Departments` or `Departments$` | `<department-share-name>` |
| Home share name | `Home` or `Home$` | `<home-share-name>` |
| Public share name | `Public` | `<public-share-name>` |
| Apps share name | `Apps` | `<apps-share-name>` |
| Department UNC path | `\\FS1\Departments` | `<department-unc-path>` |
| Home UNC path | `\\FS1\Home` | `<home-unc-path>` |
| Public UNC path | `\\FS1\Public` | `<public-unc-path>` |
| Apps UNC path | `\\FS1\Apps` | `<apps-unc-path>` |
| File server admins group | `CORP\GG-FileServer-Admins` | `<file-server-admins-group>` |
| Department RW group | `CORP\GG-FS-Accounting-RW` | `<department-rw-group>` |
| Department RO group | `CORP\GG-FS-Accounting-RO` | `<department-ro-group>` |
| General authenticated users principal | `Authenticated Users` | `<authenticated-users-principal>` |
| Public read/write group | `CORP\GG-FS-Public-RW` | `<public-rw-group>` |
| Apps read-only group | `CORP\GG-FS-Apps-RO` | `<apps-ro-group>` |
| Share permission model | Broad share, strict NTFS | `<share-permission-model>` |
| Folder enumeration | Leave default in this task | Configured in task 05 |
| SMB encryption | Leave default in this task | Configured in task 06 |
| SMB auditing | Leave default in this task | Configured in task 06 |
| Caching mode | `Manual` or `None` | `<caching-mode>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Next workbook | `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md` | `<next-task>` |

# Create_SMB_Shares_And_Share_Permissions_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm server identity | File Server | `hostname` | File server name matches plan |
| 3 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 4 | Confirm SMB server service is running | File Server | `Get-Service LanmanServer` | Server service is Running |
| 5 | Confirm SMBShare cmdlets exist | File Server | `Get-Command -Module SmbShare` | SMB share management cmdlets are available |
| 6 | Confirm local share paths exist | File Server | `Test-Path "<department-root>"`; `Test-Path "<home-folder-root>"`; `Test-Path "<public-share-root>"`; `Test-Path "<app-share-root>"` | Required local paths exist |
| 7 | Confirm NTFS ACLs from task 03 | File Server | `icacls "<department-root>"`; `icacls "<home-folder-root>"` | NTFS permissions are already configured |
| 8 | Capture existing SMB shares before changes | File Server | `Get-SmbShare` | Current share state is documented |
| 9 | Capture existing share permissions before changes | File Server | `Get-SmbShare \| ForEach-Object { Get-SmbShareAccess -Name $_.Name }` | Current share permissions are documented |
| 10 | Create Departments share | File Server | `New-SmbShare -Name "<department-share-name>" -Path "<department-root>" -FullAccess "<file-server-admins-group>" -ChangeAccess "Authenticated Users"` | Department share is created |
| 11 | Create Home share | File Server | `New-SmbShare -Name "<home-share-name>" -Path "<home-folder-root>" -FullAccess "<file-server-admins-group>" -ChangeAccess "Authenticated Users"` | Home share is created |
| 12 | Create Public share if planned | File Server | `New-SmbShare -Name "<public-share-name>" -Path "<public-share-root>" -FullAccess "<file-server-admins-group>" -ChangeAccess "<public-rw-group>"` | Public share is created |
| 13 | Create Apps share if planned | File Server | `New-SmbShare -Name "<apps-share-name>" -Path "<app-share-root>" -FullAccess "<file-server-admins-group>" -ReadAccess "<apps-ro-group>"` | Apps share is created |
| 14 | Remove unwanted default share permission if present | File Server | `Revoke-SmbShareAccess -Name "<share-name>" -AccountName "Everyone" -Force` | Unwanted broad share ACE is removed |
| 15 | Grant or confirm admin Full share access | File Server | `Grant-SmbShareAccess -Name "<share-name>" -AccountName "<file-server-admins-group>" -AccessRight Full -Force` | Admin group has Full share access |
| 16 | Grant or confirm user Change share access where NTFS will restrict | File Server | `Grant-SmbShareAccess -Name "<share-name>" -AccountName "Authenticated Users" -AccessRight Change -Force` | Users can reach share boundary |
| 17 | Grant read-only share access where required | File Server | `Grant-SmbShareAccess -Name "<share-name>" -AccountName "<ro-group>" -AccessRight Read -Force` | Read-only share access exists |
| 18 | Confirm share paths and settings | File Server | `Get-SmbShare -Name "<share-name>"` | Share path and settings match plan |
| 19 | Confirm share permissions | File Server | `Get-SmbShareAccess -Name "<share-name>"` | Share permissions match plan |
| 20 | Confirm firewall rules for SMB | File Server | `Get-NetFirewallRule -DisplayGroup "File and Printer Sharing"` | SMB firewall rule state is known |
| 21 | Test TCP 445 from client | Client | `Test-NetConnection "<file-server-name>" -Port 445` | Client can reach SMB port |
| 22 | Test UNC path from client | Client | `Test-Path "\\<file-server-name>\<share-name>"` | UNC path resolves and is reachable |
| 23 | Test create/delete with authorized RW user | Client | `New-Item "\\<file-server-name>\<share-name>\smb-test.txt"` then `Remove-Item` | RW user can write where NTFS allows |
| 24 | Test read-only behavior with RO user | Client | Attempt file create under read-only target | RO user is blocked from writing |
| 25 | Export final share inventory | File Server | `Get-SmbShare \| Export-Csv "<evidence-path>\smb-shares-after.csv" -NoTypeInformation` | Share inventory is saved |
| 26 | Export final share access inventory | File Server | Use post-change validation skeleton | Share permissions evidence is saved |
| 27 | Document final SMB share model | Operator | `Record share names, UNC paths, local paths, groups, and access rights` | SMB share build record is complete |

# Create_SMB_Shares_And_Share_Permissions_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: validate role, services, paths, NTFS ACLs, and current SMB share state before creating shares.

$EvidencePath = "C:\FileServices-Validation"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-smb-share-precheck-transcript.txt"

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-smb-shares.txt"

# Confirm server identity and File Server role.
hostname |
  Tee-Object "$EvidencePath\hostname-before-smb-shares.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-smb-shares.txt"

Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-smb-shares.txt"

# Confirm SMB services.
Get-Service LanmanServer,LanmanWorkstation |
  Tee-Object "$EvidencePath\smb-services-before-smb-shares.txt"

# Confirm SMB cmdlets.
Get-Command -Module SmbShare |
  Tee-Object "$EvidencePath\smbshare-cmdlets-before-smb-shares.txt"

# Confirm local paths exist.
$RequiredPaths = @(
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
  Tee-Object "$EvidencePath\smb-share-local-path-validation-before.txt"

# Capture NTFS ACLs before publishing shares.
foreach ($Path in $RequiredPaths) {
  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-smb-$($Path.Replace(':','').Replace('\','-')).txt"

    Get-Acl $Path |
      Format-List |
      Tee-Object "$EvidencePath\get-acl-before-smb-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Capture existing SMB shares and share permissions.
Get-SmbShare |
  Sort-Object Name |
  Tee-Object "$EvidencePath\smb-shares-before-create.txt"

Get-SmbShare |
  Sort-Object Name |
  Export-Csv "$EvidencePath\smb-shares-before-create.csv" -NoTypeInformation

Get-SmbShare |
  ForEach-Object {
    Get-SmbShareAccess -Name $_.Name
  } |
  Tee-Object "$EvidencePath\smb-share-access-before-create.txt"

# Confirm SMB firewall posture.
Get-NetFirewallRule -DisplayGroup "File and Printer Sharing" -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\file-and-printer-sharing-firewall-rules-before.txt"

Stop-Transcript
```

Create_SMB_Shares_And_Share_Permissions_Create_Core_Shares_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: create core SMB shares from already-prepared local folders.
# ABE is configured in task 05. SMB encryption, signing, and auditing are configured in task 06.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$AuthenticatedUsers = "Authenticated Users"

$DepartmentShareName = "Departments"
$HomeShareName = "Home"
$PublicShareName = "Public"
$AppsShareName = "Apps"

$DepartmentRoot = "E:\Shares\Departments"
$HomeRoot = "E:\Shares\Home"
$PublicRoot = "E:\Shares\Public"
$AppsRoot = "E:\Shares\Apps"

$PublicRWGroup = "$DomainNetbios\GG-FS-Public-RW"
$AppsROGroup = "$DomainNetbios\GG-FS-Apps-RO"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-create-core-smb-shares-transcript.txt"

# Create Departments share.
if (-not (Get-SmbShare -Name $DepartmentShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $DepartmentShareName `
    -Path $DepartmentRoot `
    -Description "Department file shares" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ChangeAccess $AuthenticatedUsers `
    -CachingMode Manual
}

# Create Home share.
if (-not (Get-SmbShare -Name $HomeShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $HomeShareName `
    -Path $HomeRoot `
    -Description "User home folders" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ChangeAccess $AuthenticatedUsers `
    -CachingMode Manual
}

# Create Public share.
if (-not (Get-SmbShare -Name $PublicShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $PublicShareName `
    -Path $PublicRoot `
    -Description "Public shared files" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ChangeAccess $PublicRWGroup `
    -CachingMode Manual
}

# Create Apps share.
if (-not (Get-SmbShare -Name $AppsShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $AppsShareName `
    -Path $AppsRoot `
    -Description "Application installers and shared tools" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ReadAccess $AppsROGroup `
    -CachingMode Manual
}

# Confirm created shares.
Get-SmbShare |
  Where-Object {
    $_.Name -in @($DepartmentShareName,$HomeShareName,$PublicShareName,$AppsShareName)
  } |
  Sort-Object Name |
  Tee-Object "$EvidencePath\core-smb-shares-after-create.txt"

Stop-Transcript
```

Create_SMB_Shares_And_Share_Permissions_Share_Permissions_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: normalize share permissions after SMB shares are created.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$AuthenticatedUsers = "Authenticated Users"

$DepartmentShareName = "Departments"
$HomeShareName = "Home"
$PublicShareName = "Public"
$AppsShareName = "Apps"

$PublicRWGroup = "$DomainNetbios\GG-FS-Public-RW"
$AppsROGroup = "$DomainNetbios\GG-FS-Apps-RO"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-share-permissions-transcript.txt"

$ManagedShares = @(
  $DepartmentShareName,
  $HomeShareName,
  $PublicShareName,
  $AppsShareName
)

# Capture share permissions before normalization.
foreach ($Share in $ManagedShares) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\smb-share-access-before-normalize-$Share.txt"
}

# Remove Everyone if present on managed shares.
foreach ($Share in $ManagedShares) {
  $EveryoneAce = Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Where-Object { $_.AccountName -eq "Everyone" }

  if ($EveryoneAce) {
    Revoke-SmbShareAccess `
      -Name $Share `
      -AccountName "Everyone" `
      -Force
  }
}

# Ensure administrative Full access.
foreach ($Share in $ManagedShares) {
  Grant-SmbShareAccess `
    -Name $Share `
    -AccountName $FileServerAdminsGroup `
    -AccessRight Full `
    -Force

  Grant-SmbShareAccess `
    -Name $Share `
    -AccountName "BUILTIN\Administrators" `
    -AccessRight Full `
    -Force
}

# Ensure user access for broad-share, strict-NTFS model.
Grant-SmbShareAccess `
  -Name $DepartmentShareName `
  -AccountName $AuthenticatedUsers `
  -AccessRight Change `
  -Force

Grant-SmbShareAccess `
  -Name $HomeShareName `
  -AccountName $AuthenticatedUsers `
  -AccessRight Change `
  -Force

Grant-SmbShareAccess `
  -Name $PublicShareName `
  -AccountName $PublicRWGroup `
  -AccessRight Change `
  -Force

Grant-SmbShareAccess `
  -Name $AppsShareName `
  -AccountName $AppsROGroup `
  -AccessRight Read `
  -Force

# Capture final share permissions.
foreach ($Share in $ManagedShares) {
  Get-SmbShareAccess -Name $Share |
    Tee-Object "$EvidencePath\smb-share-access-after-normalize-$Share.txt"
}

Stop-Transcript
```

Create_SMB_Shares_And_Share_Permissions_Client_Access_Test_Skeleton
```
# Run from a Windows client or admin workstation.
# Purpose: validate UNC access to created SMB shares.
# Replace server and share names with the lab values.

$FileServerName = "FS1"
$DepartmentShareName = "Departments"
$HomeShareName = "Home"
$PublicShareName = "Public"
$AppsShareName = "Apps"

$DepartmentUNC = "\\$FileServerName\$DepartmentShareName"
$HomeUNC = "\\$FileServerName\$HomeShareName"
$PublicUNC = "\\$FileServerName\$PublicShareName"
$AppsUNC = "\\$FileServerName\$AppsShareName"

$EvidencePath = "C:\FileServices-Validation-Client"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-client-smb-access-test-transcript.txt"

# Confirm client identity.
hostname |
  Tee-Object "$EvidencePath\client-hostname.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups.txt"

# Confirm name resolution and SMB reachability.
Resolve-DnsName $FileServerName |
  Tee-Object "$EvidencePath\file-server-name-resolution.txt"

Test-NetConnection $FileServerName -Port 445 |
  Tee-Object "$EvidencePath\test-netconnection-smb-445.txt"

# Test UNC path visibility.
$UNCs = @(
  $DepartmentUNC,
  $HomeUNC,
  $PublicUNC,
  $AppsUNC
)

foreach ($UNC in $UNCs) {
  [PSCustomObject]@{
    UNC = $UNC
    Exists = Test-Path $UNC
  } |
  Tee-Object "$EvidencePath\unc-testpath-results.txt" -Append
}

# List each share if access allows it.
foreach ($UNC in $UNCs) {
  Get-ChildItem $UNC -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\directory-listing-$($UNC.Replace('\','-').Replace(':','')).txt"
}

# Test write against Public share if this user is authorized.
$PublicTestFile = Join-Path $PublicUNC "smb-public-write-test.txt"

"SMB public write test $(Get-Date)" |
  Out-File $PublicTestFile -Force

Get-Content $PublicTestFile |
  Tee-Object "$EvidencePath\public-write-test-readback.txt"

Remove-Item $PublicTestFile -Force

# Test read-only apps share behavior.
Get-ChildItem $AppsUNC -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\apps-share-listing.txt"

Stop-Transcript
```

Create_SMB_Shares_And_Share_Permissions_Post_Change_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: capture final SMB share and share permission state after configuration.

$EvidencePath = "C:\FileServices-Validation"

$ManagedShares = @(
  "Departments",
  "Home",
  "Public",
  "Apps"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-post-change-smb-share-validation-transcript.txt"

# Capture final SMB server configuration.
Get-SmbServerConfiguration |
  Tee-Object "$EvidencePath\smb-server-configuration-after-share-create.txt"

# Capture final shares.
Get-SmbShare |
  Sort-Object Name |
  Tee-Object "$EvidencePath\smb-shares-after-share-create.txt"

Get-SmbShare |
  Sort-Object Name |
  Export-Csv "$EvidencePath\smb-shares-after-share-create.csv" -NoTypeInformation

# Capture managed share settings.
foreach ($Share in $ManagedShares) {
  Get-SmbShare -Name $Share -ErrorAction SilentlyContinue |
    Select-Object Name,Path,Description,ScopeName,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,Special |
    Tee-Object "$EvidencePath\smb-share-settings-$Share.txt"
}

# Capture final share permissions.
foreach ($Share in $ManagedShares) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\smb-share-access-final-$Share.txt"
}

# Export share permissions as CSV.
$ShareAccess = foreach ($Share in $ManagedShares) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Select-Object @{Name="ShareName";Expression={$Share}},AccountName,AccessControlType,AccessRight
}

$ShareAccess |
  Export-Csv "$EvidencePath\smb-share-access-final.csv" -NoTypeInformation

# Confirm local paths and NTFS ACLs still exist.
foreach ($Share in $ManagedShares) {
  $SmbShare = Get-SmbShare -Name $Share -ErrorAction SilentlyContinue

  if ($SmbShare) {
    [PSCustomObject]@{
      ShareName = $SmbShare.Name
      Path = $SmbShare.Path
      PathExists = Test-Path $SmbShare.Path
    } |
    Tee-Object "$EvidencePath\smb-share-path-validation-final.txt" -Append

    icacls $SmbShare.Path |
      Tee-Object "$EvidencePath\icacls-final-$Share.txt"
  }
}

# Capture active sessions and open files.
Get-SmbSession |
  Tee-Object "$EvidencePath\smb-sessions-after-share-create.txt"

Get-SmbOpenFile |
  Tee-Object "$EvidencePath\smb-open-files-after-share-create.txt"

Stop-Transcript
```

Create_SMB_Shares_And_Share_Permissions_Verification_Commands
```
# File Server role and SMB services
Get-WindowsFeature FS-FileServer
Get-Service LanmanServer
Get-Service LanmanWorkstation
Get-Command -Module SmbShare

# Local folder paths
Test-Path "<department-root>"
Test-Path "<home-folder-root>"
Test-Path "<public-share-root>"
Test-Path "<app-share-root>"

# NTFS ACLs from task 03
icacls "<department-root>"
icacls "<home-folder-root>"
icacls "<public-share-root>"
icacls "<app-share-root>"

# SMB shares
Get-SmbShare
Get-SmbShare -Name "<department-share-name>"
Get-SmbShare -Name "<home-share-name>"
Get-SmbShare -Name "<public-share-name>"
Get-SmbShare -Name "<apps-share-name>"

# SMB share permissions
Get-SmbShareAccess -Name "<department-share-name>"
Get-SmbShareAccess -Name "<home-share-name>"
Get-SmbShareAccess -Name "<public-share-name>"
Get-SmbShareAccess -Name "<apps-share-name>"

# SMB server configuration
Get-SmbServerConfiguration

# Firewall and network
Get-NetFirewallRule -DisplayGroup "File and Printer Sharing"
Test-NetConnection "<file-server-name>" -Port 445

# Client UNC tests
Test-Path "\\<file-server-name>\<department-share-name>"
Test-Path "\\<file-server-name>\<home-share-name>"
Test-Path "\\<file-server-name>\<public-share-name>"
Test-Path "\\<file-server-name>\<apps-share-name>"

# Client write test where authorized
New-Item -ItemType File -Path "\\<file-server-name>\<public-share-name>\smb-write-test.txt"
Remove-Item "\\<file-server-name>\<public-share-name>\smb-write-test.txt"

# Sessions and open files
Get-SmbSession
Get-SmbOpenFile

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*smb*"
```


Create_SMB_Shares_And_Share_Permissions_Rollback

|      |                                                   |             |                                                                                                                                      |                                           |
| ---- | ------------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| Step | Task                                              | Device      | PowerShell / Command                                                                                                                 | Expected Result                           |
| 1    | Capture current shares before rollback            | File Server | Get-SmbShare \| Export-Csv "<evidence-path>\smb-shares-before-rollback.csv" -NoTypeInformation                                       | Current share state is saved              |
| 2    | Capture current share permissions before rollback | File Server | Get-SmbShareAccess -Name "<share-name>"                                                                                              | Current share permission state is visible |
| 3    | Close active test open files if needed            | File Server | Get-SmbOpenFile; Close-SmbOpenFile -FileId <file-id> -Force                                                                          | Test files are closed                     |
| 4    | Disconnect test sessions if needed                | File Server | Get-SmbSession; Close-SmbSession -SessionId <session-id> -Force                                                                      | Test sessions are disconnected            |
| 5    | Remove incorrect share permission                 | File Server | Revoke-SmbShareAccess -Name "<share-name>" -AccountName "<account>" -Force                                                           | Incorrect share ACE is removed            |
| 6    | Restore expected admin share permission           | File Server | Grant-SmbShareAccess -Name "<share-name>" -AccountName "<file-server-admins-group>" -AccessRight Full -Force                         | Admin access is restored                  |
| 7    | Restore expected user share permission            | File Server | Grant-SmbShareAccess -Name "<share-name>" -AccountName "Authenticated Users" -AccessRight Change -Force                              | User boundary access is restored          |
| 8    | Remove incorrect SMB share                        | File Server | Remove-SmbShare -Name "<share-name>" -Force                                                                                          | Share is unpublished                      |
| 9    | Recreate correct SMB share if needed              | File Server | New-SmbShare -Name "<share-name>" -Path "<folder-path>" -FullAccess "<file-server-admins-group>" -ChangeAccess "Authenticated Users" | Correct share is recreated                |
| 10   | Confirm local folder remains intact               | File Server | Test-Path "<folder-path>"; Get-ChildItem "<folder-path>"                                                                             | Local data path still exists              |
| 11   | Validate final share state                        | File Server | Get-SmbShare; Get-SmbShareAccess -Name "<share-name>"                                                                                | Share state matches rollback target       |
| 12   | Document rollback result                          | Operator    | Record removed shares, restored permissions, and evidence files                                                                      | Rollback record is complete               |

Create_SMB_Shares_And_Share_Permissions_Failure_Checks

|   |   |   |   |
|---|---|---|---|
|Symptom|Likely Cause|Check|Corrective Action|
|New-SmbShare fails|PowerShell not elevated|whoami /groups|Reopen PowerShell as Administrator|
|New-SmbShare fails|Local path does not exist|Test-Path "<folder-path>"|Create or correct local folder path|
|New-SmbShare fails|Share name already exists|Get-SmbShare -Name "<share-name>"|Use existing share, remove incorrect share, or choose correct name|
|Share appears but client cannot connect|SMB firewall or network path blocked|Test-NetConnection <file-server> -Port 445|Allow TCP 445 from client network|
|Client cannot resolve server name|DNS issue|Resolve-DnsName <file-server-name>|Fix DNS record or use FQDN|
|Client gets access denied|Share permission too restrictive|Get-SmbShareAccess -Name "<share-name>"|Grant correct share permission|
|Client gets access denied|NTFS permission denies access|icacls "<folder-path>"; whoami /groups|Fix NTFS ACL or group membership from task 03|
|User can access share but not subfolder|NTFS inheritance or department ACL blocks user|icacls "<subfolder-path>"|Add user to correct resource group or correct ACL|
|User can write when they should only read|Share Change access combined with NTFS Modify|Get-SmbShareAccess; icacls "<folder-path>"|Correct NTFS ACL or use Read share permission for that share|
|User cannot write even with NTFS Modify|Share permission is Read|Get-SmbShareAccess -Name "<share-name>"|Grant Change share permission where appropriate|
|Revoke-SmbShareAccess fails|Account name does not match ACE|Get-SmbShareAccess -Name "<share-name>"|Use exact AccountName shown in output|
|Group name not accepted|AD group typo or not resolvable|Get-ADGroup "<group-name>"|Correct group name or domain prefix|
|Administrative group loses access|Admin Full share permission missing|Get-SmbShareAccess -Name "<share-name>"|Grant admins Full share access|
|UNC path prompts repeatedly for credentials|Cached wrong credentials|cmdkey /list; net use|Remove cached credentials and reconnect|
|UNC path points to old server|DNS alias, DFS, or mapped drive points elsewhere|Resolve-DnsName; net use|Correct mapping, DNS, or DFS target|
|Share browsing exposes too much|ABE not configured yet|Get-SmbShare -Name "<share-name>" \| Select FolderEnumerationMode|Configure ABE in task 05|
|Sensitive traffic not encrypted|SMB encryption not configured yet|Get-SmbShare -Name "<share-name>" \| Select EncryptData|Configure SMB encryption in task 06|
|No audit events appear|SMB/file access auditing not configured yet|Audit policy and event logs|Configure auditing in task 06|
|Evidence missing|Validation skeleton not run|Test-Path "<evidence-path>"|Re-run precheck or post-change validation skeleton|

