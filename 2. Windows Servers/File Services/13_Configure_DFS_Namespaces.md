13_Configure_DFS_Namespaces.md
# Configure_DFS_Namespaces

# Configure_DFS_Namespaces_Index
13_Configure_DFS_Namespaces.md
Configure_DFS_Namespaces
Configure_DFS_Namespaces_Source_Basis
Configure_DFS_Namespaces_Mental_Model
Configure_DFS_Namespaces_Planning_Table
Configure_DFS_Namespaces_Configuration_Checklist
Configure_DFS_Namespaces_Precheck_Skeleton
Configure_DFS_Namespaces_Install_And_Module_Validation_Skeleton
Configure_DFS_Namespaces_Root_Share_Preparation_Skeleton
Configure_DFS_Namespaces_Domain_Based_Root_Skeleton
Configure_DFS_Namespaces_Folder_And_Targets_Skeleton
Configure_DFS_Namespaces_Referral_And_ABE_Skeleton
Configure_DFS_Namespaces_Client_Access_Test_Skeleton
Configure_DFS_Namespaces_Export_And_Documentation_Skeleton
Configure_DFS_Namespaces_Post_Change_Validation_Skeleton
Configure_DFS_Namespaces_Verification_Commands
Configure_DFS_Namespaces_Rollback
Configure_DFS_Namespaces_Failure_Checks
Configure_DFS_Namespaces_Related_Labs

# Configure_DFS_Namespaces_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DFS Namespaces | Creating unified namespace paths for shared folders |
| Microsoft Learn | Install-WindowsFeature | Installing DFS Namespace role service and management tools |
| Microsoft Learn | DFSN PowerShell module | Managing DFS namespace roots, folders, and targets |
| Microsoft Learn | New-DfsnRoot | Creating standalone or domain-based DFS namespace roots |
| Microsoft Learn | Get-DfsnRoot | Reviewing DFS namespace roots |
| Microsoft Learn | Set-DfsnRoot | Configuring namespace root properties |
| Microsoft Learn | New-DfsnFolder | Creating DFS namespace folders |
| Microsoft Learn | Get-DfsnFolder | Reviewing namespace folders |
| Microsoft Learn | New-DfsnFolderTarget | Adding folder targets to namespace folders |
| Microsoft Learn | Get-DfsnFolderTarget | Reviewing namespace folder targets |
| Microsoft Learn | Remove-DfsnFolder / Remove-DfsnRoot | Removing DFS namespace objects during rollback |
| Microsoft Learn | dfsutil | Legacy DFS namespace validation and export support |
| Windows Server operational practice | Domain-based namespace | Present user-facing paths under the domain name, not a specific server name |
| Windows Server operational practice | Namespace abstraction | Hide backend file server names behind stable logical paths |
| Windows Server operational practice | DFSN versus DFSR | DFS Namespaces provide logical paths. DFS Replication is configured separately |

# Configure_DFS_Namespaces_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DFS Namespace | Logical folder tree that presents shared folders through one stable UNC path |
| DFSN | DFS Namespaces role service |
| DFSR | DFS Replication, separate role used to replicate data between servers |
| Namespace root | Top-level DFS path such as `\\corp.local\CorpFiles` |
| Domain-based namespace | Namespace stored in AD and accessed through the domain name |
| Standalone namespace | Namespace hosted by a single server name, such as `\\FS1\CorpFiles` |
| Namespace server | Server that hosts a namespace root target |
| Root target | SMB share used by a namespace server to host the namespace root |
| Namespace folder | Logical folder below the root, such as `\\corp.local\CorpFiles\Departments` |
| Folder target | Real SMB share path that a namespace folder points to |
| Referral | Client response that tells the client which folder target to access |
| Referral ordering | Logic controlling preferred target order |
| Referral cache | Client-side cache of DFS referral results |
| ABE on DFS namespace | Access Based Enumeration behavior for namespace folders |
| Backend share | Existing SMB share such as `\\FS1\Departments` or `\\FS1\Home$` |
| Namespace abstraction | Users access `\\corp.local\CorpFiles\Departments` instead of `\\FS1\Departments` |
| High availability | A domain-based namespace can have multiple namespace servers |
| Data availability | DFSN does not replicate data by itself. Multiple folder targets require data consistency planning |
| First rule | Build namespace paths only after SMB shares and NTFS permissions already work directly |

# Configure_DFS_Namespaces_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Primary namespace server | `FS1` | `<primary-namespace-server>` |
| Secondary namespace server | `FS2` | `<secondary-namespace-server>` |
| Namespace root name | `CorpFiles` | `<namespace-root-name>` |
| Domain namespace path | `\\corp.local\CorpFiles` | `<namespace-root-path>` |
| Namespace type | `DomainV2` | `<namespace-type>` |
| Namespace root local folder | `E:\DFSRoots\CorpFiles` | `<namespace-root-local-folder>` |
| Namespace root share name | `CorpFiles$` | `<namespace-root-share-name>` |
| Namespace root target path | `\\FS1\CorpFiles$` | `<namespace-root-target-path>` |
| Department DFS folder | `\\corp.local\CorpFiles\Departments` | `<department-dfs-folder>` |
| Department folder target | `\\FS1\Departments` | `<department-folder-target>` |
| Home DFS folder | `\\corp.local\CorpFiles\Home` | `<home-dfs-folder>` |
| Home folder target | `\\FS1\Home$` | `<home-folder-target>` |
| Public DFS folder | `\\corp.local\CorpFiles\Public` | `<public-dfs-folder>` |
| Public folder target | `\\FS1\Public` | `<public-folder-target>` |
| Apps DFS folder | `\\corp.local\CorpFiles\Apps` | `<apps-dfs-folder>` |
| Apps folder target | `\\FS1\Apps` | `<apps-folder-target>` |
| Namespace ABE stance | Enabled | `<namespace-abe-plan>` |
| Referral ordering | Lowest cost or explicit target priority | `<referral-order-plan>` |
| Referral TTL | `300` seconds | `<referral-ttl>` |
| DFS Management Tools host | `FS1` or admin workstation | `<dfs-management-host>` |
| Direct SMB dependency | Tasks 04 to 06 complete | `<direct-smb-validation-state>` |
| Replication dependency | Not in this workbook | Task 14 |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Export path | `C:\FileServices-Exports\DFSN` | `<dfsn-export-path>` |
| Client evidence path | `C:\FileServices-Validation-Client` | `<client-evidence-path>` |
| Next workbook | `14_Configure_DFS_Replication.md` | `<next-task>` |

# Configure_DFS_Namespaces_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Namespace Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm domain membership | Namespace Server | `Get-ComputerInfo \| Select CsDomain,CsPartOfDomain` | Server is domain joined |
| 3 | Confirm DNS resolution | Namespace Server | `Resolve-DnsName <domain-fqdn>` | Domain resolves |
| 4 | Confirm File Server role | Namespace Server | `Get-WindowsFeature FS-FileServer` | File Server role is installed |
| 5 | Confirm existing backend SMB shares | Namespace Server | `Get-SmbShare` | Departments, Home, Public, and Apps shares exist |
| 6 | Test backend SMB paths locally | Namespace Server | `Test-Path "\\FS1\Departments"` | Backend shares are reachable |
| 7 | Install DFS Namespace role | Namespace Server | `Install-WindowsFeature FS-DFS-Namespace -IncludeManagementTools` | DFSN role is installed |
| 8 | Confirm DFSN PowerShell module | Namespace Server | `Get-Command -Module DFSN` | DFSN cmdlets are available |
| 9 | Create namespace root local folder | Namespace Server | `New-Item -ItemType Directory -Path "<namespace-root-local-folder>" -Force` | Root folder exists |
| 10 | Create namespace root SMB share | Namespace Server | `New-SmbShare -Name "<namespace-root-share-name>" -Path "<namespace-root-local-folder>" -FullAccess "<file-server-admins-group>"` | Hidden root share exists |
| 11 | Create domain-based namespace root | Namespace Server | `New-DfsnRoot -Path "<namespace-root-path>" -TargetPath "<namespace-root-target-path>" -Type DomainV2` | Domain namespace root exists |
| 12 | Validate namespace root | Namespace Server | `Get-DfsnRoot -Path "<namespace-root-path>"` | Namespace root is returned |
| 13 | Enable namespace root ABE if planned | Namespace Server | `Set-DfsnRoot -Path "<namespace-root-path>" -EnableAccessBasedEnumeration $true` | Namespace ABE is enabled |
| 14 | Create Departments DFS folder | Namespace Server | `New-DfsnFolder -Path "<department-dfs-folder>" -TargetPath "<department-folder-target>"` | Departments namespace folder exists |
| 15 | Create Home DFS folder | Namespace Server | `New-DfsnFolder -Path "<home-dfs-folder>" -TargetPath "<home-folder-target>"` | Home namespace folder exists |
| 16 | Create Public DFS folder | Namespace Server | `New-DfsnFolder -Path "<public-dfs-folder>" -TargetPath "<public-folder-target>"` | Public namespace folder exists |
| 17 | Create Apps DFS folder | Namespace Server | `New-DfsnFolder -Path "<apps-dfs-folder>" -TargetPath "<apps-folder-target>"` | Apps namespace folder exists |
| 18 | Validate folder targets | Namespace Server | `Get-DfsnFolderTarget -Path "<department-dfs-folder>"` | Folder target points to backend share |
| 19 | Configure referral TTL if planned | Namespace Server | `Set-DfsnFolder -Path "<dfs-folder-path>" -TimeToLiveSec <referral-ttl>` | Referral TTL matches plan |
| 20 | Add secondary namespace server if planned | Namespace Server | `New-DfsnRootTarget -Path "<namespace-root-path>" -TargetPath "\\FS2\CorpFiles$"` | Namespace root has another target |
| 21 | Validate namespace from client | Client | `Test-Path "\\corp.local\CorpFiles"` | Namespace root is reachable |
| 22 | Validate department namespace path | Client | `Test-Path "\\corp.local\CorpFiles\Departments"` | Department DFS path works |
| 23 | Validate home namespace path | Client | `Test-Path "\\corp.local\CorpFiles\Home"` | Home DFS path works |
| 24 | Validate target referral cache | Client | `dfsutil /pktinfo` | DFS referral cache shows namespace and target |
| 25 | Export DFS namespace config | Namespace Server | Use export skeleton | Namespace evidence is saved |
| 26 | Document final namespace design | Operator | `Record root, folders, targets, referral settings, and client tests` | DFS namespace build record is complete |

# Configure_DFS_Namespaces_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: validate AD, DNS, SMB, DFSN feature state, and backend shares before DFS Namespace configuration.

$EvidencePath = "C:\FileServices-Validation"

$DomainFqdn = "corp.local"
$NamespaceServer = $env:COMPUTERNAME

$NamespaceRootName = "CorpFiles"
$NamespaceRootPath = "\\$DomainFqdn\$NamespaceRootName"
$NamespaceRootLocalFolder = "E:\DFSRoots\CorpFiles"
$NamespaceRootShareName = "CorpFiles$"
$NamespaceRootTargetPath = "\\$NamespaceServer\$NamespaceRootShareName"

$BackendTargets = @(
  "\\FS1\Departments",
  "\\FS1\Home$",
  "\\FS1\Public",
  "\\FS1\Apps"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-dfsn-precheck-transcript.txt"

# Identity and server state.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-dfsn.txt"

hostname |
  Tee-Object "$EvidencePath\hostname-before-dfsn.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-dfsn.txt"

# DNS and domain validation.
Resolve-DnsName $DomainFqdn |
  Tee-Object "$EvidencePath\domain-dns-resolution-before-dfsn.txt"

nltest /dsgetdc:$DomainFqdn |
  Tee-Object "$EvidencePath\nltest-dc-discovery-before-dfsn.txt"

# Feature state.
Get-WindowsFeature FS-FileServer,FS-DFS-Namespace,RSAT-DFS-Mgmt-Con |
  Tee-Object "$EvidencePath\dfs-features-before-dfsn.txt"

# SMB service and share state.
Get-Service LanmanServer |
  Tee-Object "$EvidencePath\smb-service-before-dfsn.txt"

Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-before-dfsn.txt"

# Backend target validation.
foreach ($Target in $BackendTargets) {
  [PSCustomObject]@{
    Target = $Target
    Reachable = Test-Path $Target
  } |
    Tee-Object "$EvidencePath\backend-target-validation-before-dfsn.txt" -Append
}

# Existing DFS namespace state if cmdlets are already available.
Get-Command -Module DFSN -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-cmdlets-before-dfsn.txt"

Get-DfsnRoot -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-roots-before-dfsn.txt"

Get-DfsnRoot -Path $NamespaceRootPath -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-namespace-root-before-dfsn.txt"

# Existing local namespace root folder/share state.
Test-Path $NamespaceRootLocalFolder |
  Tee-Object "$EvidencePath\namespace-root-local-folder-testpath-before.txt"

Get-SmbShare -Name $NamespaceRootShareName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\namespace-root-share-before-dfsn.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Install_And_Module_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: install DFS Namespace role service and validate management tooling.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-dfsn-install-module-validation-transcript.txt"

# Install DFS Namespace role service and management tools.
Install-WindowsFeature `
  -Name FS-DFS-Namespace `
  -IncludeManagementTools

# Optional management console component.
Install-WindowsFeature `
  -Name RSAT-DFS-Mgmt-Con `
  -IncludeAllSubFeature

# Confirm feature state.
Get-WindowsFeature FS-DFS-Namespace,RSAT-DFS-Mgmt-Con |
  Tee-Object "$EvidencePath\dfs-features-after-install.txt"

# Confirm DFSN PowerShell module and cmdlets.
Get-Module DFSN -ListAvailable |
  Tee-Object "$EvidencePath\dfsn-module-after-install.txt"

Import-Module DFSN

Get-Command -Module DFSN |
  Sort-Object Name |
  Tee-Object "$EvidencePath\dfsn-cmdlets-after-install.txt"

# Confirm DFS service state if present.
Get-Service DFS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfs-service-after-install.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Root_Share_Preparation_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: prepare the local folder and hidden SMB share used as the DFS namespace root target.

$EvidencePath = "C:\FileServices-Validation"

$DomainNetbios = "CORP"
$NamespaceRootLocalFolder = "E:\DFSRoots\CorpFiles"
$NamespaceRootShareName = "CorpFiles$"

$FileServerAdminsGroup = "$DomainNetbios\GG-FileServer-Admins"
$AuthenticatedUsers = "Authenticated Users"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-dfsn-root-share-preparation-transcript.txt"

# Create DFS root local folder.
New-Item `
  -ItemType Directory `
  -Path $NamespaceRootLocalFolder `
  -Force |
  Tee-Object "$EvidencePath\dfsn-root-local-folder-create.txt"

# Configure root folder ACL.
icacls $NamespaceRootLocalFolder /inheritance:r
icacls $NamespaceRootLocalFolder /grant "SYSTEM:(OI)(CI)(F)"
icacls $NamespaceRootLocalFolder /grant "BUILTIN\Administrators:(OI)(CI)(F)"
icacls $NamespaceRootLocalFolder /grant "${FileServerAdminsGroup}:(OI)(CI)(F)"
icacls $NamespaceRootLocalFolder /grant "${AuthenticatedUsers}:(RX)"

# Create hidden SMB share for DFS namespace root target.
if (-not (Get-SmbShare -Name $NamespaceRootShareName -ErrorAction SilentlyContinue)) {
  New-SmbShare `
    -Name $NamespaceRootShareName `
    -Path $NamespaceRootLocalFolder `
    -Description "DFS Namespace root target for CorpFiles" `
    -FullAccess $FileServerAdminsGroup,"BUILTIN\Administrators" `
    -ReadAccess $AuthenticatedUsers `
    -CachingMode Manual
}

# Validate root target share.
Get-SmbShare -Name $NamespaceRootShareName |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,Special |
  Tee-Object "$EvidencePath\dfsn-root-smb-share-after-create.txt"

Get-SmbShareAccess -Name $NamespaceRootShareName |
  Tee-Object "$EvidencePath\dfsn-root-smb-share-access-after-create.txt"

icacls $NamespaceRootLocalFolder |
  Tee-Object "$EvidencePath\dfsn-root-local-folder-icacls-after-create.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Domain_Based_Root_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: create a domain-based DFS namespace root.

$EvidencePath = "C:\FileServices-Validation"

$DomainFqdn = "corp.local"
$NamespaceServer = $env:COMPUTERNAME

$NamespaceRootName = "CorpFiles"
$NamespaceRootPath = "\\$DomainFqdn\$NamespaceRootName"
$NamespaceRootShareName = "CorpFiles$"
$NamespaceRootTargetPath = "\\$NamespaceServer\$NamespaceRootShareName"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-domain-based-dfsn-root-transcript.txt"

Import-Module DFSN

# Capture existing namespace roots.
Get-DfsnRoot -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-roots-before-create-root.txt"

# Create domain-based namespace root if missing.
if (-not (Get-DfsnRoot -Path $NamespaceRootPath -ErrorAction SilentlyContinue)) {
  New-DfsnRoot `
    -Path $NamespaceRootPath `
    -TargetPath $NamespaceRootTargetPath `
    -Type DomainV2 `
    -Description "Corporate file services DFS namespace"
}

# Enable Access Based Enumeration on the namespace root.
Set-DfsnRoot `
  -Path $NamespaceRootPath `
  -EnableAccessBasedEnumeration $true

# Validate root.
Get-DfsnRoot -Path $NamespaceRootPath |
  Tee-Object "$EvidencePath\dfsn-root-after-create.txt"

Get-DfsnRootTarget -Path $NamespaceRootPath |
  Tee-Object "$EvidencePath\dfsn-root-targets-after-create.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Folder_And_Targets_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: create DFS namespace folders and link them to existing SMB share targets.
# This does not configure DFS Replication. Replication is task 14.

$EvidencePath = "C:\FileServices-Validation"

$DomainFqdn = "corp.local"
$NamespaceRootName = "CorpFiles"
$NamespaceRootPath = "\\$DomainFqdn\$NamespaceRootName"

$Folders = @(
  [PSCustomObject]@{
    Name = "Departments"
    DfsPath = "$NamespaceRootPath\Departments"
    TargetPath = "\\FS1\Departments"
    Description = "Department file shares"
  },
  [PSCustomObject]@{
    Name = "Home"
    DfsPath = "$NamespaceRootPath\Home"
    TargetPath = "\\FS1\Home$"
    Description = "User home folders"
  },
  [PSCustomObject]@{
    Name = "Public"
    DfsPath = "$NamespaceRootPath\Public"
    TargetPath = "\\FS1\Public"
    Description = "Public shared files"
  },
  [PSCustomObject]@{
    Name = "Apps"
    DfsPath = "$NamespaceRootPath\Apps"
    TargetPath = "\\FS1\Apps"
    Description = "Application installers and shared tools"
  }
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-dfsn-folders-and-targets-transcript.txt"

Import-Module DFSN

# Confirm namespace root exists.
Get-DfsnRoot -Path $NamespaceRootPath |
  Tee-Object "$EvidencePath\dfsn-root-before-folder-create.txt"

foreach ($Folder in $Folders) {
  # Validate backend target.
  [PSCustomObject]@{
    DfsPath = $Folder.DfsPath
    TargetPath = $Folder.TargetPath
    TargetReachable = Test-Path $Folder.TargetPath
  } |
    Tee-Object "$EvidencePath\dfsn-target-reachability-before-folder-create.txt" -Append

  # Create namespace folder if missing.
  if (-not (Get-DfsnFolder -Path $Folder.DfsPath -ErrorAction SilentlyContinue)) {
    New-DfsnFolder `
      -Path $Folder.DfsPath `
      -TargetPath $Folder.TargetPath `
      -Description $Folder.Description
  }
  else {
    # Add target if folder exists but target is missing.
    $ExistingTarget = Get-DfsnFolderTarget -Path $Folder.DfsPath -ErrorAction SilentlyContinue |
      Where-Object { $_.TargetPath -eq $Folder.TargetPath }

    if (-not $ExistingTarget) {
      New-DfsnFolderTarget `
        -Path $Folder.DfsPath `
        -TargetPath $Folder.TargetPath
    }
  }

  # Confirm folder and target.
  Get-DfsnFolder -Path $Folder.DfsPath |
    Tee-Object "$EvidencePath\dfsn-folder-after-create-$($Folder.Name).txt"

  Get-DfsnFolderTarget -Path $Folder.DfsPath |
    Tee-Object "$EvidencePath\dfsn-folder-target-after-create-$($Folder.Name).txt"
}

# Final inventory.
Get-DfsnFolder -Path "$NamespaceRootPath\*" |
  Tee-Object "$EvidencePath\dfsn-folders-after-create-all.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Referral_And_ABE_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: configure namespace ABE and referral behavior for DFS folders.

$EvidencePath = "C:\FileServices-Validation"

$DomainFqdn = "corp.local"
$NamespaceRootName = "CorpFiles"
$NamespaceRootPath = "\\$DomainFqdn\$NamespaceRootName"

$ReferralTtlSeconds = 300

$DfsFolders = @(
  "$NamespaceRootPath\Departments",
  "$NamespaceRootPath\Home",
  "$NamespaceRootPath\Public",
  "$NamespaceRootPath\Apps"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-dfsn-referral-abe-transcript.txt"

Import-Module DFSN

# Enable ABE on root.
Set-DfsnRoot `
  -Path $NamespaceRootPath `
  -EnableAccessBasedEnumeration $true

Get-DfsnRoot -Path $NamespaceRootPath |
  Tee-Object "$EvidencePath\dfsn-root-after-abe-enable.txt"

# Configure referral TTL on folders.
foreach ($FolderPath in $DfsFolders) {
  if (Get-DfsnFolder -Path $FolderPath -ErrorAction SilentlyContinue) {
    Set-DfsnFolder `
      -Path $FolderPath `
      -TimeToLiveSec $ReferralTtlSeconds

    Get-DfsnFolder -Path $FolderPath |
      Tee-Object "$EvidencePath\dfsn-folder-after-referral-config-$($FolderPath.Split('\')[-1]).txt"

    Get-DfsnFolderTarget -Path $FolderPath |
      Tee-Object "$EvidencePath\dfsn-folder-targets-after-referral-config-$($FolderPath.Split('\')[-1]).txt"
  }
}

# Optional target priority example.
# Use only when multiple folder targets exist.
# Set-DfsnFolderTarget `
#   -Path "$NamespaceRootPath\Departments" `
#   -TargetPath "\\FS1\Departments" `
#   -ReferralPriorityClass GlobalHigh

Stop-Transcript
```

# Configure_DFS_Namespaces_Client_Access_Test_Skeleton
```powershell
# Run from a Windows client as a normal domain user.
# Purpose: validate DFS namespace access, referrals, SMB connectivity, and user-facing paths.

$EvidencePath = "C:\FileServices-Validation-Client"

$DomainFqdn = "corp.local"
$NamespaceRootName = "CorpFiles"

$NamespaceRoot = "\\$DomainFqdn\$NamespaceRootName"
$DepartmentsPath = "$NamespaceRoot\Departments"
$HomePath = "$NamespaceRoot\Home"
$PublicPath = "$NamespaceRoot\Public"
$AppsPath = "$NamespaceRoot\Apps"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-client-dfsn-access-test-transcript.txt"

# Client identity and token.
hostname |
  Tee-Object "$EvidencePath\client-hostname-dfsn.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami-dfsn.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups-dfsn.txt"

# DNS and domain validation.
Resolve-DnsName $DomainFqdn |
  Tee-Object "$EvidencePath\client-domain-dns-resolution-dfsn.txt"

nltest /dsgetdc:$DomainFqdn |
  Tee-Object "$EvidencePath\client-dc-discovery-dfsn.txt"

# Namespace path tests.
$Paths = @(
  $NamespaceRoot,
  $DepartmentsPath,
  $HomePath,
  $PublicPath,
  $AppsPath
)

foreach ($Path in $Paths) {
  [PSCustomObject]@{
    Path = $Path
    Exists = Test-Path $Path
  } |
    Tee-Object "$EvidencePath\client-dfsn-path-test-results.txt" -Append

  Get-ChildItem $Path -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\client-dfsn-listing-$($Path.Replace('\','-').Replace('$','hidden')).txt"
}

# Write test where user is authorized.
$PublicTestFile = Join-Path $PublicPath "dfsn-public-write-test-$env:USERNAME.txt"

try {
  "DFS Namespace write test from $env:USERNAME at $(Get-Date)" |
    Out-File $PublicTestFile -Force

  Get-Content $PublicTestFile |
    Tee-Object "$EvidencePath\client-dfsn-public-write-readback.txt"

  Remove-Item $PublicTestFile -Force
}
catch {
  "DFS Namespace write test failed: $($_.Exception.Message)" |
    Tee-Object "$EvidencePath\client-dfsn-public-write-test-result.txt"
}

# Referral cache.
dfsutil /pktinfo |
  Tee-Object "$EvidencePath\client-dfsutil-pktinfo.txt"

dfsutil /spcinfo |
  Tee-Object "$EvidencePath\client-dfsutil-spcinfo.txt"

# SMB connection state.
Get-SmbConnection |
  Select-Object ServerName,ShareName,Dialect,Signed,Encrypted,UserName |
  Tee-Object "$EvidencePath\client-smb-connections-after-dfsn.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Export_And_Documentation_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: export DFS namespace configuration for backup and rebuild documentation.

$EvidencePath = "C:\FileServices-Validation"
$ExportRoot = "C:\FileServices-Exports"
$DfsnExportPath = Join-Path $ExportRoot "DFSN"

$DomainFqdn = "corp.local"
$NamespaceRootName = "CorpFiles"
$NamespaceRootPath = "\\$DomainFqdn\$NamespaceRootName"

New-Item -ItemType Directory -Force -Path $EvidencePath
New-Item -ItemType Directory -Force -Path $DfsnExportPath

Start-Transcript -Path "$EvidencePath\13-dfsn-export-documentation-transcript.txt"

Import-Module DFSN

# Export root, root targets, folders, and folder targets.
Get-DfsnRoot -Path $NamespaceRootPath |
  Tee-Object "$DfsnExportPath\dfsn-root.txt"

Get-DfsnRoot -Path $NamespaceRootPath |
  Export-Clixml "$DfsnExportPath\dfsn-root.clixml"

Get-DfsnRootTarget -Path $NamespaceRootPath |
  Tee-Object "$DfsnExportPath\dfsn-root-targets.txt"

Get-DfsnRootTarget -Path $NamespaceRootPath |
  Export-Clixml "$DfsnExportPath\dfsn-root-targets.clixml"

Get-DfsnFolder -Path "$NamespaceRootPath\*" |
  Tee-Object "$DfsnExportPath\dfsn-folders.txt"

Get-DfsnFolder -Path "$NamespaceRootPath\*" |
  Export-Clixml "$DfsnExportPath\dfsn-folders.clixml"

$Folders = Get-DfsnFolder -Path "$NamespaceRootPath\*" -ErrorAction SilentlyContinue

$FolderTargets = foreach ($Folder in $Folders) {
  Get-DfsnFolderTarget -Path $Folder.Path -ErrorAction SilentlyContinue |
    Select-Object @{Name="FolderPath";Expression={$Folder.Path}},TargetPath,State,ReferralPriorityClass,ReferralPriorityRank
}

$FolderTargets |
  Tee-Object "$DfsnExportPath\dfsn-folder-targets.txt"

$FolderTargets |
  Export-Csv "$DfsnExportPath\dfsn-folder-targets.csv" -NoTypeInformation

# Export SMB root share evidence.
Get-SmbShare |
  Where-Object { $_.Name -like "*CorpFiles*" -or $_.Path -like "*DFSRoots*" } |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,Special |
  Export-Csv "$DfsnExportPath\dfsn-root-smb-shares.csv" -NoTypeInformation

# Optional dfsutil export if available.
dfsutil root export $NamespaceRootPath "$DfsnExportPath\dfsn-dfsutil-export.xml" verbose |
  Tee-Object "$DfsnExportPath\dfsn-dfsutil-export-result.txt"

# Generate rebuild notes.
$RebuildNotes = @"
# DFS Namespace Rebuild Notes

Namespace root:
$NamespaceRootPath

Rebuild order:
1. Install File Server role and DFS Namespace role.
2. Recreate namespace root local folder.
3. Recreate hidden root SMB share.
4. Recreate domain-based namespace root.
5. Recreate namespace folders.
6. Recreate folder targets.
7. Reapply referral settings and ABE.
8. Validate from a client with Test-Path and dfsutil /pktinfo.
9. Configure DFS Replication separately in task 14 if multi-target data consistency is required.
"@

$RebuildNotes |
  Out-File "$DfsnExportPath\DFSN-Rebuild-Notes.md" -Encoding UTF8

# Validate export files.
Get-ChildItem $DfsnExportPath |
  Select-Object FullName,Length,CreationTime,LastWriteTime |
  Tee-Object "$EvidencePath\dfsn-export-files.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Post_Change_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the namespace server.
# Purpose: capture final DFS namespace, SMB, referral, and event evidence.

$EvidencePath = "C:\FileServices-Validation"

$DomainFqdn = "corp.local"
$NamespaceRootName = "CorpFiles"
$NamespaceRootPath = "\\$DomainFqdn\$NamespaceRootName"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\13-post-change-dfsn-validation-transcript.txt"

Import-Module DFSN

# Feature and module state.
Get-WindowsFeature FS-DFS-Namespace,RSAT-DFS-Mgmt-Con |
  Tee-Object "$EvidencePath\dfs-features-final.txt"

Get-Command -Module DFSN |
  Sort-Object Name |
  Tee-Object "$EvidencePath\dfsn-cmdlets-final.txt"

# Namespace root and root targets.
Get-DfsnRoot -Path $NamespaceRootPath |
  Tee-Object "$EvidencePath\dfsn-root-final.txt"

Get-DfsnRootTarget -Path $NamespaceRootPath |
  Tee-Object "$EvidencePath\dfsn-root-targets-final.txt"

# Namespace folders and targets.
Get-DfsnFolder -Path "$NamespaceRootPath\*" |
  Tee-Object "$EvidencePath\dfsn-folders-final.txt"

$Folders = Get-DfsnFolder -Path "$NamespaceRootPath\*" -ErrorAction SilentlyContinue

foreach ($Folder in $Folders) {
  Get-DfsnFolderTarget -Path $Folder.Path |
    Tee-Object "$EvidencePath\dfsn-folder-targets-final-$($Folder.Path.Split('\')[-1]).txt"
}

# Root SMB share and backend shares.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,Special |
  Tee-Object "$EvidencePath\smb-shares-final-dfsn.txt"

# Namespace path validation from server.
Test-Path $NamespaceRootPath |
  Tee-Object "$EvidencePath\server-testpath-namespace-root-final.txt"

Test-Path "$NamespaceRootPath\Departments" |
  Tee-Object "$EvidencePath\server-testpath-departments-final.txt"

Test-Path "$NamespaceRootPath\Home" |
  Tee-Object "$EvidencePath\server-testpath-home-final.txt"

Test-Path "$NamespaceRootPath\Public" |
  Tee-Object "$EvidencePath\server-testpath-public-final.txt"

Test-Path "$NamespaceRootPath\Apps" |
  Tee-Object "$EvidencePath\server-testpath-apps-final.txt"

# DFS service and events.
Get-Service DFS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfs-service-final.txt"

Get-WinEvent -LogName "DFS Replication" -MaxEvents 50 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfs-replication-log-note-final.txt"

Get-WinEvent -LogName System -MaxEvents 200 |
  Where-Object {
    $_.ProviderName -like "*DFS*" -or
    $_.Message -like "*DFS Namespace*" -or
    $_.Message -like "*DfsSvc*"
  } |
  Select-Object TimeCreated,Id,ProviderName,LevelDisplayName,Message |
  Tee-Object "$EvidencePath\dfs-system-events-final.txt"

Stop-Transcript
```

# Configure_DFS_Namespaces_Verification_Commands
```powershell
# Feature and module
Get-WindowsFeature FS-DFS-Namespace
Get-WindowsFeature RSAT-DFS-Mgmt-Con
Install-WindowsFeature FS-DFS-Namespace -IncludeManagementTools
Get-Module DFSN -ListAvailable
Import-Module DFSN
Get-Command -Module DFSN

# Domain and DNS validation
Resolve-DnsName "<domain-fqdn>"
nltest /dsgetdc:<domain-fqdn>

# SMB root share and backend shares
Get-SmbShare
Get-SmbShare -Name "<namespace-root-share-name>"
Get-SmbShareAccess -Name "<namespace-root-share-name>"
Test-Path "<namespace-root-target-path>"
Test-Path "<department-folder-target>"
Test-Path "<home-folder-target>"

# DFS namespace root
Get-DfsnRoot
Get-DfsnRoot -Path "<namespace-root-path>"
New-DfsnRoot -Path "<namespace-root-path>" -TargetPath "<namespace-root-target-path>" -Type DomainV2
Set-DfsnRoot -Path "<namespace-root-path>" -EnableAccessBasedEnumeration $true
Get-DfsnRootTarget -Path "<namespace-root-path>"

# DFS namespace folders
Get-DfsnFolder -Path "<namespace-root-path>\*"
New-DfsnFolder -Path "<department-dfs-folder>" -TargetPath "<department-folder-target>" -Description "Department file shares"
New-DfsnFolder -Path "<home-dfs-folder>" -TargetPath "<home-folder-target>" -Description "User home folders"

# DFS folder targets
Get-DfsnFolderTarget -Path "<department-dfs-folder>"
New-DfsnFolderTarget -Path "<dfs-folder-path>" -TargetPath "<folder-target-path>"
Set-DfsnFolder -Path "<dfs-folder-path>" -TimeToLiveSec <referral-ttl>
Set-DfsnFolderTarget -Path "<dfs-folder-path>" -TargetPath "<folder-target-path>" -ReferralPriorityClass GlobalHigh

# Client validation
Test-Path "\\<domain-fqdn>\<namespace-root-name>"
Test-Path "\\<domain-fqdn>\<namespace-root-name>\Departments"
Test-Path "\\<domain-fqdn>\<namespace-root-name>\Home"
Get-ChildItem "\\<domain-fqdn>\<namespace-root-name>"
Get-ChildItem "\\<domain-fqdn>\<namespace-root-name>\Departments"

# DFS client referral cache
dfsutil /pktinfo
dfsutil /spcinfo
dfsutil /pktflush

# Export
dfsutil root export "\\<domain-fqdn>\<namespace-root-name>" "<dfsn-export-path>\dfsn-export.xml" verbose
Get-DfsnRoot -Path "<namespace-root-path>" | Export-Clixml "<dfsn-export-path>\dfsn-root.clixml"
Get-DfsnFolder -Path "<namespace-root-path>\*" | Export-Clixml "<dfsn-export-path>\dfsn-folders.clixml"

# Rollback
Remove-DfsnFolderTarget -Path "<dfs-folder-path>" -TargetPath "<folder-target-path>" -Force
Remove-DfsnFolder -Path "<dfs-folder-path>" -Force
Remove-DfsnRootTarget -Path "<namespace-root-path>" -TargetPath "<namespace-root-target-path>" -Force
Remove-DfsnRoot -Path "<namespace-root-path>" -Force

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*dfsn*"
Get-ChildItem "<dfsn-export-path>"
```

# Configure_DFS_Namespaces_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Export current namespace before rollback | Namespace Server | `dfsutil root export "<namespace-root-path>" "<dfsn-export-path>\pre-rollback.xml" verbose` | Current namespace is preserved |
| 2 | Capture current namespace objects | Namespace Server | `Get-DfsnRoot`; `Get-DfsnFolder`; `Get-DfsnFolderTarget` | Rollback evidence is saved |
| 3 | Flush DFS referral cache on test client | Client | `dfsutil /pktflush` | Client drops cached referrals |
| 4 | Remove incorrect folder target | Namespace Server | `Remove-DfsnFolderTarget -Path "<dfs-folder-path>" -TargetPath "<folder-target-path>" -Force` | Incorrect target is removed |
| 5 | Remove incorrect namespace folder | Namespace Server | `Remove-DfsnFolder -Path "<dfs-folder-path>" -Force` | Incorrect DFS folder is removed |
| 6 | Remove secondary root target if added incorrectly | Namespace Server | `Remove-DfsnRootTarget -Path "<namespace-root-path>" -TargetPath "<root-target-path>" -Force` | Incorrect root target is removed |
| 7 | Disable ABE on namespace root if rollback requires it | Namespace Server | `Set-DfsnRoot -Path "<namespace-root-path>" -EnableAccessBasedEnumeration $false` | Namespace ABE is disabled |
| 8 | Remove namespace root only in lab or approved teardown | Namespace Server | `Remove-DfsnRoot -Path "<namespace-root-path>" -Force` | Namespace root is removed |
| 9 | Remove root SMB share if namespace root removed | Namespace Server | `Remove-SmbShare -Name "<namespace-root-share-name>" -Force` | Hidden root share is removed |
| 10 | Remove local root folder if lab-only and empty | Namespace Server | `Remove-Item "<namespace-root-local-folder>" -Recurse -Force` | Root folder is removed |
| 11 | Validate backend shares still exist | File Server | `Get-SmbShare`; `Test-Path "<folder-target-path>"` | Backend SMB shares remain intact |
| 12 | Validate namespace rollback from client | Client | `Test-Path "<namespace-root-path>"`; `dfsutil /pktinfo` | Client view matches rollback target |
| 13 | Document rollback result | Operator | `Record removed DFS folders, targets, root targets, and client validation` | Rollback record is complete |

# Configure_DFS_Namespaces_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| `New-DfsnRoot` fails | DFS Namespace role not installed | `Get-WindowsFeature FS-DFS-Namespace` | Install `FS-DFS-Namespace` |
| `New-DfsnRoot` fails | Root target share does not exist | `Get-SmbShare -Name "<namespace-root-share-name>"` | Create hidden namespace root share first |
| `New-DfsnRoot` fails | Server not domain joined or domain unavailable | `Get-ComputerInfo`; `nltest /dsgetdc:<domain>` | Fix domain join, DNS, or DC connectivity |
| Namespace path does not resolve | DNS or AD namespace issue | `Resolve-DnsName <domain>`; `Get-DfsnRoot` | Fix DNS/domain availability and verify namespace root |
| Client gets access denied to namespace root | Root share or NTFS permissions too restrictive | `Get-SmbShareAccess`; `icacls "<namespace-root-local-folder>"` | Grant appropriate read/traverse access |
| DFS folder exists but target fails | Backend SMB share unreachable | `Test-Path "<folder-target-path>"` | Fix backend SMB share, permissions, DNS, or firewall |
| User can access direct SMB but not DFS path | DFS folder target or namespace permission issue | `Get-DfsnFolderTarget`; `dfsutil /pktinfo` | Correct folder target and flush referral cache |
| User sees old target | DFS referral cache still active | `dfsutil /pktinfo` | Run `dfsutil /pktflush` or wait for TTL |
| ABE does not hide folders | Namespace ABE disabled or NTFS still allows access | `Get-DfsnRoot`; `icacls backend path` | Enable ABE and correct NTFS ACLs |
| Hidden Home$ target fails through DFS | Share target path typed incorrectly | `Get-DfsnFolderTarget -Path "<home-dfs-folder>"` | Use exact target `\\server\Home$` |
| Multiple targets show inconsistent data | DFSN does not replicate data | Compare folder targets | Configure DFS Replication in task 14 or use single target |
| Users randomly hit stale server | Multiple folder targets with bad referral priority | `Get-DfsnFolderTarget`; `dfsutil /pktinfo` | Disable bad target or adjust priority |
| Root target removal breaks namespace | Last root target removed | `Get-DfsnRootTarget` | Add healthy root target before removing old one |
| DFS Management GUI shows stale info | Console cache or replication delay | Refresh GUI and use PowerShell validation | Trust PowerShell and client tests |
| `dfsutil` not recognized | DFS management tools missing | `Get-WindowsFeature RSAT-DFS-Mgmt-Con` | Install DFS management tools |
| Namespace export fails | Export path missing or permissions issue | `Test-Path "<dfsn-export-path>"` | Create export folder and rerun elevated |
| Evidence missing | Validation skeleton not run | `Test-Path "<evidence-path>"` | Re-run precheck or post-change validation skeleton |

# Configure_DFS_Namespaces_Related_Labs
| Related Lab                                                      | Relationship                                                              |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `00_File_Services_Index.md`                                      | Defines where DFS Namespaces fits in the File Services suite              |
| `01_Install_File_Server_Role_And_Management_Tools.md`            | Provides File Server role and base management tools                       |
| `02_Prepare_File_Server_Storage_Volumes_And_Folders.md`          | Provides root folders used by namespace root shares and backend targets   |
| `03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`           | Provides NTFS ACLs that control effective access through DFS paths        |
| `04_Create_SMB_Shares_And_Share_Permissions.md`                  | Creates backend SMB shares used as DFS folder targets                     |
| `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`  | Provides share visibility behavior that interacts with namespace browsing |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md`   | Secures SMB sessions used by DFS namespace referrals                      |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`        | Provides real department and home folder targets for DFS namespace paths  |
| `08_Configure_File_Screening_With_FSRM.md`                       | Applies file screening to backend targets exposed through DFS             |
| `09_Configure_Storage_Quotas_With_FSRM.md`                       | Applies quotas to backend targets exposed through DFS                     |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | Reports on backend storage regardless of DFS namespace abstraction        |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md`            | Provides restore capability on backend shares accessed through DFS        |
| `12_Backup_Restore_And_Export_File_Server_Config.md`             | Exports DFS namespace configuration for rebuild and recovery              |
| `14_Configure_DFS_Replication.md`                                | Adds data replication for multi-target DFS namespace designs              |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`   | Helps identify access patterns through DFS-backed SMB shares              |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`          | Troubleshoots DFS, SMB, NTFS, DNS, and referral path failures             |
| `25_Migrate_File_Server_Data_Shares_And_Permissions.md`          | Uses DFS namespace abstraction to reduce user-facing migration impact     |