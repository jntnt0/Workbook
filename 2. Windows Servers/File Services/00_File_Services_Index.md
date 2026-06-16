00_File_Services_Index.md
# File_Services_Index

# File_Services_Index_Index
00_File_Services_Index.md
File_Services_Index
File_Services_Index_Source_Basis
File_Services_Index_Mental_Model
File_Services_Index_Workbook_Map
File_Services_Index_Operational_Build_Order
File_Services_Index_Planning_Table
File_Services_Index_Configuration_Checklist
File_Services_Index_Core_Validation_Skeleton
File_Services_Index_SMB_Validation_Skeleton
File_Services_Index_FSRM_Validation_Skeleton
File_Services_Index_DFS_Validation_Skeleton
File_Services_Index_Advanced_Features_Validation_Skeleton
File_Services_Index_Verification_Commands
File_Services_Index_Rollback
File_Services_Index_Failure_Checks
File_Services_Index_Related_Labs

# File_Services_Index_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install-WindowsFeature | Installing File Server, FSRM, DFS, NFS, iSCSI, BranchCache, Work Folders, Data Deduplication, and Failover Clustering features |
| Microsoft Learn | SMBShare PowerShell module | Creating, securing, enumerating, and auditing SMB shares |
| Microsoft Learn | NTFS permissions and ACL tooling | Managing folder permissions with File Explorer, `icacls`, `Get-Acl`, and `Set-Acl` |
| Microsoft Learn | File Server Resource Manager | File screening, quotas, reports, and file management tasks |
| Microsoft Learn | DFS Namespaces | Creating namespace roots, folders, targets, and referral behavior |
| Microsoft Learn | DFS Replication | Creating replication groups, replicated folders, members, connections, and backlog validation |
| Microsoft Learn | Shadow Copies / Previous Versions | Enabling point-in-time recovery for shared folders |
| Microsoft Learn | Data Deduplication | Enabling deduplication, scheduling optimization, and validating savings |
| Microsoft Learn | NFS Server | Supporting non-Windows clients with NFS shares and identity mapping considerations |
| Microsoft Learn | iSCSI Target Server | Creating virtual disks, targets, initiator access, and client connections |
| Microsoft Learn | BranchCache | Optimizing branch office access to file content |
| Microsoft Learn | Work Folders | Providing user file sync from Windows Server |
| Microsoft Learn | Azure File Sync | Syncing Windows Server file shares with Azure file shares |
| Microsoft Learn | Storage Replica | Synchronous or asynchronous server-to-server volume replication |
| Microsoft Learn | Scale-Out File Server | Continuously available file shares for clustered workloads |
| Windows Server operational practice | File share lifecycle | Planning storage, permissions, sharing, auditing, backup, migration, monitoring, and troubleshooting |

# File_Services_Index_Mental_Model
| Concept | Operational Meaning |
|---|---|
| File Server | Windows Server role that exposes storage to users, departments, applications, or non-Windows clients |
| Volume | Disk partition or mount point where file data and share folders live |
| Folder structure | The logical layout that separates departments, home folders, applications, profiles, and shared data |
| NTFS permissions | File system access control applied directly to files and folders |
| Share permissions | Network-level permissions applied at the SMB share boundary |
| Effective access | Final result of identity, group membership, NTFS ACLs, share permissions, inheritance, and deny entries |
| ACL inheritance | Permission flow from parent folder to child folders and files |
| Access Based Enumeration | SMB feature that hides files and folders users cannot access |
| SMB | Primary Windows file sharing protocol |
| SMB signing | Integrity protection for SMB traffic |
| SMB encryption | Confidentiality protection for SMB traffic |
| SMB auditing | Event visibility for share access, failures, and file operations |
| FSRM | File Server Resource Manager, used for quotas, file screens, reports, and file management tasks |
| File screen | Policy that blocks or audits unwanted file types |
| Quota | Storage limit applied to a folder or volume path |
| Storage report | FSRM-generated report showing usage, large files, duplicates, owners, or file groups |
| Shadow Copies | Point-in-time snapshots exposed to users as Previous Versions |
| DFS Namespace | Logical namespace that hides physical file server names behind a friendly path |
| DFS Replication | Multi-server replication engine for keeping folder targets synchronized |
| Data Deduplication | Storage optimization that removes duplicate chunks from supported volumes |
| NFS | File sharing protocol used by Linux, Unix, VMware, and other non-Windows clients |
| iSCSI Target | Server role that presents virtual disks as block storage to initiators |
| BranchCache | WAN optimization technology for branch office content retrieval |
| Work Folders | User file sync feature hosted on Windows Server |
| Azure File Sync | Cloud-backed sync between Windows Server and Azure Files |
| Storage Replica | Block-level volume replication for disaster recovery |
| Scale-Out File Server | Clustered file server pattern for continuously available application shares |
| First rule | File services are only clean when storage layout, identity groups, NTFS permissions, share permissions, DNS names, backup, and rollback are planned first |

# File_Services_Index_Workbook_Map
| Order | Workbook | Primary Goal | Required Before | Produces |
|---:|---|---|---|---|
| 00 | `00_File_Services_Index.md` | Map the File Services workbook suite and operational order | Existing Windows Server lab context | Navigation and build sequence |
| 01 | `01_Install_File_Server_Role_And_Management_Tools.md` | Install File Server role and management tools | Windows Server baseline complete | File Server role installed |
| 02 | `02_Prepare_File_Server_Storage_Volumes_And_Folders.md` | Prepare disks, volumes, folder paths, and base structure | File Server role installed | Storage and folder foundation |
| 03 | `03_Configure_NTFS_Permissions_And_ACL_Inheritance.md` | Configure file system permissions and inheritance | Folder structure planned | NTFS ACL baseline |
| 04 | `04_Create_SMB_Shares_And_Share_Permissions.md` | Create SMB shares and network access permissions | NTFS permissions defined | Published SMB shares |
| 05 | `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md` | Hide inaccessible content and tune share visibility | SMB shares exist | Cleaner user-facing share view |
| 06 | `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md` | Configure SMB security and auditing | SMB shares exist | Hardened and auditable SMB access |
| 07 | `07_Configure_User_Home_Folders_And_Department_Shares.md` | Build user and department share model | AD users/groups and SMB/NTFS baseline | Home and department shares |
| 08 | `08_Configure_File_Screening_With_FSRM.md` | Block or audit unwanted file types | FSRM installed and folders exist | File screen policy |
| 09 | `09_Configure_Storage_Quotas_With_FSRM.md` | Apply hard or soft storage limits | FSRM installed and folders exist | Quota policy |
| 10 | `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | Generate reports and automate file actions | FSRM installed and storage populated | Reporting and automation |
| 11 | `11_Configure_Shadow_Copies_And_Previous_Versions.md` | Enable user self-service recovery | Volumes and shares exist | Previous Versions capability |
| 12 | `12_Backup_Restore_And_Export_File_Server_Config.md` | Preserve shares, ACLs, FSRM, DFS, and server config | Working file server | Recovery evidence and exports |
| 13 | `13_Configure_DFS_Namespaces.md` | Create logical namespace paths for shares | SMB shares and AD DNS exist | DFS namespace |
| 14 | `14_Configure_DFS_Replication.md` | Replicate shared folders between servers | DFS targets and second file server | DFSR replication group |
| 15 | `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md` | Monitor active file server usage | Active shares | Operational visibility |
| 16 | `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md` | Triage user access and SMB failures | SMB shares and ACLs exist | Root cause workflow |
| 17 | `17_Configure_Data_Deduplication_For_File_Shares.md` | Reduce storage usage on supported volumes | File server volume with eligible data | Deduplicated volume |
| 18 | `18_Configure_NFS_Shares_For_Non_Windows_Clients.md` | Publish shares for Linux or Unix clients | File Server role and storage exist | NFS access path |
| 19 | `19_Configure_iSCSI_Target_Server_And_Virtual_Disks.md` | Present block storage to initiators | Storage volume prepared | iSCSI target and virtual disk |
| 20 | `20_Configure_BranchCache_For_File_Services.md` | Optimize branch office access to file data | SMB shares and branch clients | BranchCache-enabled access |
| 21 | `21_Configure_Work_Folders.md` | Provide user file sync from Windows Server | AD users/groups and storage path | Work Folders sync share |
| 22 | `22_Configure_Azure_File_Sync_For_Windows_File_Server.md` | Sync on-prem file server content to Azure Files | Azure subscription and file server share | Cloud sync path |
| 23 | `23_Configure_Storage_Replica_For_File_Server_Disaster_Recovery.md` | Replicate file server volumes for DR | Two servers and matching storage | Storage Replica partnership |
| 24 | `24_Configure_Scale_Out_File_Server_And_Continuously_Available_Shares.md` | Build clustered highly available shares | Failover cluster and shared storage | Continuously available SMB shares |
| 25 | `25_Migrate_File_Server_Data_Shares_And_Permissions.md` | Move data, shares, ACLs, and metadata safely | Source and destination file servers | Migrated file service |
| 26 | `26_Troubleshoot_DFS_Namespaces_Replication_And_SMB_Performance.md` | Triage DFS, DFSR, and SMB performance problems | DFS/SMB deployed | Advanced troubleshooting workflow |

# File_Services_Index_Operational_Build_Order
| Phase | Workbooks | Purpose |
|---:|---|---|
| 1 | `00` | Understand suite scope, naming, and build sequence |
| 2 | `01` to `02` | Install File Server role and prepare storage foundation |
| 3 | `03` to `07` | Build the core Windows share model with NTFS, SMB, ABE, security, home folders, and department shares |
| 4 | `08` to `10` | Add FSRM governance through file screens, quotas, reports, and file management tasks |
| 5 | `11` to `12` | Add recovery, backup, restore, and export safety |
| 6 | `13` to `14` | Add DFS namespace abstraction and replication |
| 7 | `15` to `16` | Monitor and troubleshoot production-style SMB access |
| 8 | `17` to `21` | Add advanced file service roles: deduplication, NFS, iSCSI, BranchCache, and Work Folders |
| 9 | `22` to `24` | Add hybrid cloud sync, disaster recovery, and high availability |
| 10 | `25` to `26` | Migrate existing file servers and troubleshoot DFS/SMB performance at depth |

# File_Services_Index_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| File server IP | `10.10.10.30` | `<file-server-ip>` |
| Management host | File server or admin workstation | `<management-host>` |
| Admin account | `CORP\Administrator` or delegated admin | `<admin-account>` |
| Data volume | `E:` | `<data-volume>` |
| Share root | `E:\Shares` | `<share-root>` |
| Department root | `E:\Shares\Departments` | `<department-root>` |
| Home folder root | `E:\Shares\Home` | `<home-folder-root>` |
| App share root | `E:\Shares\Apps` | `<app-share-root>` |
| Public share root | `E:\Shares\Public` | `<public-share-root>` |
| SMB share name | `Departments$` or `Departments` | `<share-name>` |
| UNC path | `\\FS1\Departments` | `<unc-path>` |
| DFS namespace root | `\\corp.local\Files` | `<dfs-root>` |
| DFS folder path | `\\corp.local\Files\Departments` | `<dfs-folder-path>` |
| Primary department group | `CORP\GG-FS-Accounting-RW` | `<rw-group>` |
| Read-only department group | `CORP\GG-FS-Accounting-RO` | `<ro-group>` |
| Denied group if used | Avoid unless required | `<deny-group>` |
| Permission model | AGDLP or AGUDLP | `<permission-model>` |
| NTFS inheritance stance | Inherit at root, break only where needed | `<inheritance-plan>` |
| Share permission stance | Broad share permission, restrictive NTFS | `<share-permission-plan>` |
| ABE stance | Enabled on user-facing shares | `<abe-plan>` |
| SMB encryption stance | Required for sensitive shares | `<smb-encryption-plan>` |
| SMB auditing stance | Audit failures and sensitive success events | `<audit-plan>` |
| FSRM file screen path | `E:\Shares\Departments` | `<file-screen-path>` |
| FSRM quota path | `E:\Shares\Home` | `<quota-path>` |
| Shadow copy volume | `E:` | `<shadow-copy-volume>` |
| Backup path | `D:\FileServerBackup` | `<backup-path>` |
| Export path | `C:\FileServer-Export` | `<export-path>` |
| Second file server | `FS2` | `<secondary-file-server>` |
| DFSR replicated folder | `Departments` | `<dfsr-folder-name>` |
| Deduplication target volume | `E:` | `<dedup-volume>` |
| NFS share path | `E:\Shares\NFS` | `<nfs-share-path>` |
| iSCSI virtual disk path | `E:\iSCSI\Disk01.vhdx` | `<iscsi-vhdx-path>` |
| Branch office subnet | `10.20.10.0/24` | `<branch-subnet>` |
| Work Folders path | `E:\SyncShare` | `<sync-share-path>` |
| Azure File Sync group | `sg-files-corp` | `<storage-sync-group>` |
| Storage Replica source volume | `E:` | `<sr-source-volume>` |
| Storage Replica destination volume | `E:` | `<sr-destination-volume>` |
| Cluster name if used | `FSCLUSTER1` | `<cluster-name>` |
| Scale-Out File Server role name | `SOFS1` | `<sofs-name>` |
| Migration source server | `OLD-FS1` | `<migration-source>` |
| Migration destination server | `FS1` | `<migration-destination>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Rollback stance | Lab rollback or production change control | `<rollback-plan>` |

# File_Services_Index_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm server identity | File Server | `hostname` | Server name is known |
| 2 | Confirm domain membership | File Server | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Server domain state is known |
| 3 | Confirm IP configuration | File Server | `Get-NetIPConfiguration` | Static IP, gateway, and DNS are visible |
| 4 | Confirm available disks and volumes | File Server | `Get-Disk`; `Get-Volume` | Data storage target is known |
| 5 | Install core File Server role | File Server | `Install-WindowsFeature FS-FileServer -IncludeManagementTools` | File Server role is installed |
| 6 | Install optional management tools | File Server | `Install-WindowsFeature RSAT-File-Services` | File services tools are available |
| 7 | Create share root folder | File Server | `New-Item -ItemType Directory -Path "<share-root>" -Force` | Base share folder exists |
| 8 | Create department and home folder roots | File Server | `New-Item -ItemType Directory -Path "<department-root>","<home-folder-root>" -Force` | Primary folder structure exists |
| 9 | Create or confirm AD security groups | DC / Admin Host | `Get-ADGroup "<group-name>"` | Permission groups exist |
| 10 | Apply NTFS baseline | File Server | `icacls "<share-root>"` and workbook-specific ACL commands | NTFS permissions match design |
| 11 | Validate inherited permissions | File Server | `Get-Acl "<share-root>" \| Format-List` | ACL inheritance state is visible |
| 12 | Create SMB share | File Server | `New-SmbShare -Name "<share-name>" -Path "<folder-path>"` | SMB share exists |
| 13 | Configure share permissions | File Server | `Grant-SmbShareAccess -Name "<share-name>" -AccountName "<group>" -AccessRight Change -Force` | Share permissions match design |
| 14 | Enable Access Based Enumeration | File Server | `Set-SmbShare -Name "<share-name>" -FolderEnumerationMode AccessBased` | Users only see accessible folders |
| 15 | Configure SMB encryption if required | File Server | `Set-SmbShare -Name "<share-name>" -EncryptData $true` | Share requires SMB encryption |
| 16 | Review SMB server configuration | File Server | `Get-SmbServerConfiguration` | SMB server posture is visible |
| 17 | Configure auditing if required | File Server / GPO | `auditpol /get /category:*` | Audit policy state is known |
| 18 | Create user home folders | File Server | `New-Item -ItemType Directory -Path "<home-folder-root>\<username>" -Force` | Home folders exist |
| 19 | Create department shares | File Server | `New-Item -ItemType Directory -Path "<department-root>\<department>" -Force` | Department paths exist |
| 20 | Install FSRM | File Server | `Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools` | FSRM is installed |
| 21 | Configure file screens | File Server | Use file screening workbook | Blocked or audited file types are enforced |
| 22 | Configure quotas | File Server | Use quota workbook | Storage limits are enforced |
| 23 | Configure FSRM reports | File Server | Use storage reports workbook | Reports generate on demand or schedule |
| 24 | Configure Shadow Copies | File Server | Use shadow copies workbook | Previous Versions are available |
| 25 | Export core configuration | File Server | Use backup/export workbook | Shares, ACLs, FSRM, DFS, and evidence are exported |
| 26 | Configure DFS Namespace if needed | File Server / DC | `Install-WindowsFeature FS-DFS-Namespace -IncludeManagementTools` | DFSN feature is available |
| 27 | Configure DFS Replication if needed | File Servers | `Install-WindowsFeature FS-DFS-Replication -IncludeManagementTools` | DFSR feature is available |
| 28 | Monitor SMB sessions | File Server | `Get-SmbSession`; `Get-SmbOpenFile` | Active usage is visible |
| 29 | Review file service events | File Server | `Get-WinEvent` against SMB, DFS, FSRM, and System logs | Recent events are visible |
| 30 | Document final state | Operator | `Record volumes, folders, groups, ACLs, shares, DFS, FSRM, backup, and validation results` | File services build record is complete |

# File_Services_Index_Core_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: quick server-side File Services validation.

$FileServer = $env:COMPUTERNAME
$ShareRoot = "E:\Shares"
$EvidencePath = "C:\FileServices-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName,OsHardwareAbstractionLayer |
  Tee-Object "$EvidencePath\computer-state.txt"

# Confirm network configuration.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration.txt"

Get-DnsClientServerAddress |
  Tee-Object "$EvidencePath\dns-client-server-addresses.txt"

# Confirm storage.
Get-Disk |
  Tee-Object "$EvidencePath\disks.txt"

Get-Volume |
  Tee-Object "$EvidencePath\volumes.txt"

# Confirm File Server role and related features.
Get-WindowsFeature |
  Where-Object {
    $_.Name -like "FS-*" -or
    $_.Name -like "RSAT-File-*"
  } |
  Sort-Object Name |
  Tee-Object "$EvidencePath\file-services-features.txt"

# Confirm share root.
Test-Path $ShareRoot |
  Tee-Object "$EvidencePath\share-root-testpath.txt"

if (Test-Path $ShareRoot) {
  Get-ChildItem $ShareRoot -Force |
    Tee-Object "$EvidencePath\share-root-children.txt"

  Get-Acl $ShareRoot |
    Format-List |
    Tee-Object "$EvidencePath\share-root-acl.txt"
}

# Confirm services commonly involved in File Services.
Get-Service LanmanServer,LanmanWorkstation -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\smb-services.txt"

Get-Service SrmSvc,Dfs,DFSR,NfsService,WinTarget,PeerDistSvc,SyncShareSvc -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\optional-file-services.txt"
  
```


File_Services_Index_SMB_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: validate SMB shares, share permissions, active sessions, and open files.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm SMB server configuration.
Get-SmbServerConfiguration |
  Tee-Object "$EvidencePath\smb-server-configuration.txt"

# List SMB shares.
Get-SmbShare |
  Sort-Object Name |
  Tee-Object "$EvidencePath\smb-shares.txt"

Get-SmbShare |
  Sort-Object Name |
  Export-Csv "$EvidencePath\smb-shares.csv" -NoTypeInformation

# Capture share access rules.
Get-SmbShare |
  Where-Object { $_.Special -eq $false } |
  ForEach-Object {
    Get-SmbShareAccess -Name $_.Name
  } |
  Tee-Object "$EvidencePath\smb-share-access.txt"

# Capture share-specific configuration.
Get-SmbShare |
  Where-Object { $_.Special -eq $false } |
  Select-Object Name,Path,Description,FolderEnumerationMode,EncryptData,CachingMode,ContinuouslyAvailable |
  Export-Csv "$EvidencePath\smb-share-settings.csv" -NoTypeInformation

# Capture active SMB sessions and open files.
Get-SmbSession |
  Tee-Object "$EvidencePath\smb-sessions.txt"

Get-SmbOpenFile |
  Tee-Object "$EvidencePath\smb-open-files.txt"

# Basic SMB client/server cmdlet availability.
Get-Command -Module SmbShare |
  Tee-Object "$EvidencePath\smbshare-cmdlets.txt"
```


File_Services_Index_FSRM_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: validate File Server Resource Manager state.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm FSRM feature and service.
Get-WindowsFeature FS-Resource-Manager |
  Tee-Object "$EvidencePath\fsrm-feature.txt"

Get-Service SrmSvc -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-service.txt"

# Confirm FSRM cmdlets.
Get-Command -Module FileServerResourceManager -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-cmdlets.txt"

# Capture quotas.
Get-FsrmQuota -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quotas.txt"

Get-FsrmQuotaTemplate -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-quota-templates.txt"

# Capture file screens.
Get-FsrmFileScreen -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screens.txt"

Get-FsrmFileScreenTemplate -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-screen-templates.txt"

Get-FsrmFileGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-groups.txt"

# Capture reports and file management tasks.
Get-FsrmStorageReport -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-storage-reports.txt"

Get-FsrmFileManagementJob -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\fsrm-file-management-jobs.txt"
```


File_Services_Index_DFS_Validation_Skeleton
```
# Run in elevated PowerShell on a DFS management host.
# Purpose: validate DFS Namespace and DFS Replication state.

$DomainFqdn = "corp.local"
$DfsRoot = "\\corp.local\Files"
$EvidencePath = "C:\FileServices-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm DFS features.
Get-WindowsFeature FS-DFS-Namespace,FS-DFS-Replication |
  Tee-Object "$EvidencePath\dfs-features.txt"

# Confirm DFS services.
Get-Service Dfs,DFSR -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfs-services.txt"

# Confirm DFS cmdlets.
Get-Command -Module DFSN -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-cmdlets.txt"

Get-Command -Module DFSR -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsr-cmdlets.txt"

# Capture DFS namespace roots.
Get-DfsnRoot -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-roots.txt"

# Capture DFS namespace folders under target root.
Get-DfsnFolder -Path "$DfsRoot\*" -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsn-folders.txt"

# Capture DFS namespace targets.
Get-DfsnFolder -Path "$DfsRoot\*" -ErrorAction SilentlyContinue |
  ForEach-Object {
    Get-DfsnFolderTarget -Path $_.Path -ErrorAction SilentlyContinue
  } |
  Tee-Object "$EvidencePath\dfsn-folder-targets.txt"

# Capture DFSR groups and memberships.
Get-DfsReplicationGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dfsr-replication-groups.txt"

Get-DfsReplicationGroup -ErrorAction SilentlyContinue |
  ForEach-Object {
    Get-DfsrMember -GroupName $_.GroupName -ErrorAction SilentlyContinue
  } |
  Tee-Object "$EvidencePath\dfsr-members.txt"

Get-DfsReplicationGroup -ErrorAction SilentlyContinue |
  ForEach-Object {
    Get-DfsReplicatedFolder -GroupName $_.GroupName -ErrorAction SilentlyContinue
  } |
  Tee-Object "$EvidencePath\dfsr-replicated-folders.txt"
```


File_Services_Index_Advanced_Features_Validation_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: quick inventory of advanced File Services features.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

# Data Deduplication.
Get-WindowsFeature FS-Data-Deduplication |
  Tee-Object "$EvidencePath\dedup-feature.txt"

Get-DedupVolume -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dedup-volumes.txt"

Get-DedupStatus -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dedup-status.txt"

# NFS.
Get-WindowsFeature FS-NFS-Service |
  Tee-Object "$EvidencePath\nfs-feature.txt"

Get-NfsShare -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\nfs-shares.txt"

# iSCSI Target Server.
Get-WindowsFeature FS-iSCSITarget-Server |
  Tee-Object "$EvidencePath\iscsi-target-feature.txt"

Get-IscsiServerTarget -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\iscsi-targets.txt"

Get-IscsiVirtualDisk -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\iscsi-virtual-disks.txt"

# BranchCache.
Get-WindowsFeature BranchCache |
  Tee-Object "$EvidencePath\branchcache-feature.txt"

Get-BCStatus -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\branchcache-status.txt"

# Work Folders.
Get-WindowsFeature FS-SyncShareService |
  Tee-Object "$EvidencePath\work-folders-feature.txt"

Get-SyncShare -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\work-folders-sync-shares.txt"

# Storage Replica.
Get-WindowsFeature Storage-Replica |
  Tee-Object "$EvidencePath\storage-replica-feature.txt"

Get-SRGroup -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\storage-replica-groups.txt"

Get-SRPartnership -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\storage-replica-partnerships.txt"

# Failover clustering / SOFS.
Get-WindowsFeature Failover-Clustering,FS-FileServer |
  Tee-Object "$EvidencePath\cluster-sofs-features.txt"

Get-Cluster -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\cluster-state.txt"

Get-ClusterResource -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\cluster-resources.txt"
```


File_Services_Index_Verification_Commands
```
# Server identity and baseline
hostname
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName
Get-NetIPConfiguration
Get-Disk
Get-Volume

# Installed file services features
Get-WindowsFeature FS-*
Get-WindowsFeature RSAT-File-Services
Get-WindowsFeature BranchCache,Storage-Replica,Failover-Clustering

# SMB
Get-SmbServerConfiguration
Get-SmbShare
Get-SmbShareAccess -Name "<share-name>"
Get-SmbSession
Get-SmbOpenFile

# NTFS and ACLs
Get-Acl "<folder-path>" | Format-List
icacls "<folder-path>"

# Access tests
Test-Path "\\<file-server>\<share-name>"
New-Item -ItemType File -Path "\\<file-server>\<share-name>\test.txt"
Remove-Item "\\<file-server>\<share-name>\test.txt"

# FSRM
Get-WindowsFeature FS-Resource-Manager
Get-Service SrmSvc
Get-FsrmQuota
Get-FsrmFileScreen
Get-FsrmStorageReport
Get-FsrmFileManagementJob

# DFS Namespace
Get-WindowsFeature FS-DFS-Namespace
Get-DfsnRoot
Get-DfsnFolder -Path "\\<domain-fqdn>\<namespace-root>\*"
Get-DfsnFolderTarget -Path "\\<domain-fqdn>\<namespace-root>\<folder>"

# DFS Replication
Get-WindowsFeature FS-DFS-Replication
Get-Service DFSR
Get-DfsReplicationGroup
Get-DfsrMember -GroupName "<replication-group>"
Get-DfsReplicatedFolder -GroupName "<replication-group>"
dfsrdiag backlog /rgname:"<replication-group>" /rfname:"<replicated-folder>" /smem:<source-server> /rmem:<destination-server>

# Data Deduplication
Get-DedupVolume
Get-DedupStatus
Get-DedupSchedule

# NFS
Get-NfsShare
Get-NfsSharePermission -Name "<nfs-share-name>"

# iSCSI
Get-IscsiServerTarget
Get-IscsiVirtualDisk

# BranchCache
Get-BCStatus

# Work Folders
Get-SyncShare

# Storage Replica
Get-SRGroup
Get-SRPartnership

# Events
Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Operational" -MaxEvents 50
Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Audit" -MaxEvents 50
Get-WinEvent -LogName "DFS Replication" -MaxEvents 50
Get-WinEvent -LogName "File Server Resource Manager" -MaxEvents 50
Get-WinEvent -LogName System -MaxEvents 100
```


File_Services_Index_Rollback


|   |   |   |   |   |
|---|---|---|---|---|
|Scope|Rollback Action|Device|PowerShell / Command|Expected Result|
|SMB share|Remove test share|File Server|Remove-SmbShare -Name "<share-name>" -Force|Share is unpublished|
|Share permission|Revoke incorrect share permission|File Server|Revoke-SmbShareAccess -Name "<share-name>" -AccountName "<account>" -Force|Incorrect access is removed|
|NTFS ACL|Restore prior ACL from saved evidence|File Server|icacls "<folder-path>" /restore "<acl-backup-file>"|Previous ACL is restored|
|ABE|Disable ABE if needed|File Server|Set-SmbShare -Name "<share-name>" -FolderEnumerationMode Unrestricted|Share shows normal enumeration|
|SMB encryption|Disable share encryption if not required|File Server|Set-SmbShare -Name "<share-name>" -EncryptData $false|Encryption requirement removed|
|FSRM quota|Remove incorrect quota|File Server|Remove-FsrmQuota -Path "<quota-path>"|Quota is removed|
|FSRM file screen|Remove incorrect file screen|File Server|Remove-FsrmFileScreen -Path "<screen-path>"|File screen is removed|
|DFS namespace folder|Remove incorrect namespace folder|DFS Management Host|Remove-DfsnFolder -Path "<dfs-folder-path>" -Force|DFS folder is removed|
|DFS namespace target|Remove incorrect folder target|DFS Management Host|Remove-DfsnFolderTarget -Path "<dfs-folder-path>" -TargetPath "\\<server>\<share>" -Force|Target is removed|
|DFSR group|Remove lab replication group|DFS Management Host|Remove-DfsReplicationGroup -GroupName "<replication-group>" -Force|Replication group is removed|
|Deduplication|Disable dedup on lab volume|File Server|Disable-DedupVolume -Volume "<volume-letter>"|Dedup no longer processes volume|
|NFS share|Remove NFS share|File Server|Remove-NfsShare -Name "<nfs-share-name>"|NFS share is removed|
|iSCSI target|Remove lab iSCSI target|File Server|Remove-IscsiServerTarget -TargetName "<target-name>"|Target is removed|
|iSCSI virtual disk|Remove lab virtual disk mapping|File Server|Remove-IscsiVirtualDisk -Path "<vhdx-path>"|Virtual disk is removed from iSCSI config|
|BranchCache|Disable BranchCache|File Server|Disable-BC|BranchCache is disabled|
|Work Folders|Remove sync share|File Server|Remove-SyncShare -Name "<sync-share-name>"|Work Folders sync share is removed|
|Storage Replica|Remove partnership|File Servers|Remove-SRPartnership -SourceComputerName "<source>" -SourceRGName "<source-rg>" -DestinationComputerName "<destination>" -DestinationRGName "<destination-rg>"|Replication partnership is removed|
|SOFS share|Remove continuously available share|Cluster Node|Remove-SmbShare -Name "<share-name>" -Force|Clustered share is removed|
|Migration|Fall back to source server|Client / DNS / DFS|Restore prior DFS target, DNS alias, or client mapping|Users return to source file server|

File_Services_Index_Failure_Checks

|   |   |   |   |
|---|---|---|---|
|Symptom|Likely Cause|Check|Corrective Action|
|Client cannot access share|DNS name does not resolve|Resolve-DnsName <file-server-fqdn>|Fix DNS record or use correct server name|
|Client cannot access share|SMB blocked by firewall|Test-NetConnection <file-server> -Port 445|Allow SMB TCP 445 between client and server|
|User sees access denied|NTFS ACL does not allow user or group|icacls "<folder-path>"; whoami /groups|Add correct group to NTFS ACL|
|User sees access denied|Share permission is too restrictive|Get-SmbShareAccess -Name "<share-name>"|Grant correct SMB share permission|
|User sees folders they should not see|ABE disabled|Get-SmbShare -Name "<share-name>" \| Select FolderEnumerationMode|Enable Access Based Enumeration|
|User cannot see folder they should see|User is missing group membership or token is stale|whoami /groups; AD group check|Add group membership and have user sign out/in|
|SMB share missing|Share was not created or was removed|Get-SmbShare|Recreate share with correct path|
|SMB encryption breaks old clients|Client does not support required SMB encryption|Get-SmbConnection from client|Disable encryption for non-sensitive share or upgrade client|
|File screen not blocking files|FSRM service stopped or path mismatch|Get-Service SrmSvc; Get-FsrmFileScreen|Start service and correct screen path|
|Quota not applying|Quota assigned to wrong folder|Get-FsrmQuota|Apply quota to correct path|
|Shadow copies unavailable|Volume not configured or schedule missing|vssadmin list shadows; shadow copy settings|Configure shadow copies on the correct volume|
|Previous Versions empty|No snapshots exist yet|vssadmin list shadows|Create snapshot or wait for schedule|
|DFS path unavailable|Namespace root or target offline|Get-DfsnRoot; Get-DfsnFolderTarget|Restore namespace server or target|
|DFS target referral wrong|Old target still active|Get-DfsnFolderTarget|Disable or remove stale target|
|DFSR not replicating|Service stopped or replication backlog exists|Get-Service DFSR; dfsrdiag backlog|Start DFSR and troubleshoot backlog|
|DFSR conflict files appear|Same files changed on multiple members|DFSR conflict folder and event logs|Review conflict resolution and user workflow|
|Dedup not saving space|Dataset not eligible or job not run|Get-DedupStatus; Get-DedupJob|Confirm workload type and run optimization|
|NFS client cannot mount|NFS share permissions or client mapping wrong|Get-NfsShare; Get-NfsSharePermission|Correct NFS client permissions and identity mapping|
|iSCSI initiator cannot connect|IQN not allowed or portal unreachable|Get-IscsiServerTarget; Test-NetConnection <server> -Port 3260|Add initiator ID and allow TCP 3260|
|BranchCache no benefit|Clients or server not enabled|Get-BCStatus|Enable BranchCache and verify policy|
|Work Folders sync fails|Certificate, URL, or user access issue|Work Folders event logs|Fix certificate, URL, and sync share permissions|
|Azure File Sync fails|Agent, cloud endpoint, or sync group issue|Azure File Sync portal and agent events|Repair agent registration and sync endpoints|
|Storage Replica not replicating|Network, log volume, or partnership issue|Get-SRPartnership; Get-WinEvent|Validate topology and repair partnership|
|SOFS share fails over poorly|Share not continuously available or cluster issue|Get-SmbShare; Get-ClusterResource|Configure CA share and fix cluster health|
|Migration lost permissions|Robocopy or migration path omitted security|icacls; migration logs|Re-run migration preserving security and owner metadata|
|SMB performance poor|Network, signing/encryption, disk, or antivirus bottleneck|PerfMon, SMB counters, disk latency|Isolate bottleneck and tune one layer at a time|

