18_Configure_NFS_Shares_For_Non_Windows_Clients.md
# Configure_NFS_Shares_For_Non_Windows_Clients

# Configure_NFS_Shares_For_Non_Windows_Clients_Index
18_Configure_NFS_Shares_For_Non_Windows_Clients.md
Configure_NFS_Shares_For_Non_Windows_Clients
Configure_NFS_Shares_For_Non_Windows_Clients_Source_Basis
Configure_NFS_Shares_For_Non_Windows_Clients_Mental_Model
Configure_NFS_Shares_For_Non_Windows_Clients_Planning_Table
Configure_NFS_Shares_For_Non_Windows_Clients_Configuration_Checklist
Configure_NFS_Shares_For_Non_Windows_Clients_Feature_Install_Skeleton
Configure_NFS_Shares_For_Non_Windows_Clients_Export_Path_Skeleton
Configure_NFS_Shares_For_Non_Windows_Clients_Unmapped_Access_Skeleton
Configure_NFS_Shares_For_Non_Windows_Clients_Client_Group_Skeleton
Configure_NFS_Shares_For_Non_Windows_Clients_Linux_Client_Mount_Skeleton
Configure_NFS_Shares_For_Non_Windows_Clients_Identity_And_Permission_Skeleton
Configure_NFS_Shares_For_Non_Windows_Clients_Event_And_Service_Check_Skeleton
Configure_NFS_Shares_For_Non_Windows_Clients_Verification_Commands
Configure_NFS_Shares_For_Non_Windows_Clients_Rollback
Configure_NFS_Shares_For_Non_Windows_Clients_Failure_Checks
Configure_NFS_Shares_For_Non_Windows_Clients_Related_Labs

# Configure_NFS_Shares_For_Non_Windows_Clients_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server NFS overview | Server for NFS and Client for NFS | Windows Server acting as an NFS file server for Linux and UNIX clients |
| Windows Server NFS overview | Supported protocol versions | NFSv2, NFSv3, and NFSv4.1 server-side support |
| NFS PowerShell module | New-NfsShare | Creating an NFS export from a Windows folder |
| NFS PowerShell module | Get-NfsShare | Reviewing configured NFS shares |
| NFS PowerShell module | Grant-NfsSharePermission | Granting host, client group, or netgroup access |
| NFS PowerShell module | New-NfsClientgroup | Grouping allowed NFS clients |
| Windows Firewall | Server for NFS rules | Allowing NFS traffic to the server |
| Linux NFS tools | showmount, mount, nfsstat | Validating access from non-Windows clients |
| Operational file server practice | NTFS plus NFS export rules | Preventing accidental wide-open cross-platform shares |

# Configure_NFS_Shares_For_Non_Windows_Clients_Mental_Model
| Concept | Operational Meaning |
|---|---|
| NFS | Network File System protocol used heavily by Linux and UNIX clients |
| Server for NFS | Windows Server role service that exports Windows folders over NFS |
| NFS export | NFS share presented to non-Windows clients |
| NFS client | Linux, UNIX, appliance, hypervisor, or other system mounting the export |
| Export path | Windows folder path shared over NFS |
| NFS share name | Export name visible from the client side |
| AUTH_SYS | Basic UID/GID-based NFS authentication model |
| Kerberos NFS | Stronger authentication model using Kerberos options such as krb5, krb5i, or krb5p |
| Unmapped UNIX User Access | Allows NFS users without Windows identity mapping to access the share using generated SIDs |
| Anonymous access | NFS access using anonymous UID and GID values |
| Root access | Controls whether UNIX root can access the export as root |
| Client group | Server-side NFS object used to group allowed clients |
| Host permission | Export permission granted to a specific hostname or IP |
| NTFS permissions | Windows file system ACLs still protect the backing folder |
| NFS permissions | Export-level permissions decide which NFS clients can connect |
| Most restrictive wins | The client must pass NFS export access and Windows folder permissions |
| Blunt rule | NFS is not SMB with a different name. Treat client identity, UID/GID behavior, root access, and export permissions as their own design problem |

# Configure_NFS_Shares_For_Non_Windows_Clients_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Windows file server | `FS1` | `<file-server-name>` |
| File server IP | `10.10.20.10` | `<file-server-ip>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Target volume | `D:` | `<target-volume>` |
| NFS folder path | `D:\NFS\LinuxShare01` | `<nfs-folder-path>` |
| NFS share name | `LinuxShare01` | `<nfs-share-name>` |
| Linux client hostname | `linux01` | `<linux-client-name>` |
| Linux client IP | `10.10.30.25` | `<linux-client-ip>` |
| Linux subnet | `10.10.30.0/24` | `<linux-subnet>` |
| NFS version target | `3` or `4.1` | `<nfs-version>` |
| Authentication model | `sys` | `<authentication-model>` |
| Permission model | `readonly` or `readwrite` | `<nfs-permission>` |
| Anonymous access | Disabled / Enabled | `<anonymous-access-state>` |
| Unmapped access | Disabled / Enabled | `<unmapped-access-state>` |
| Root access | Disabled / Enabled | `<root-access-state>` |
| Anonymous UID | `65534` | `<anonymous-uid>` |
| Anonymous GID | `65534` | `<anonymous-gid>` |
| Client group name | `LinuxNFSClients` | `<client-group-name>` |
| NTFS access group | `GG_NFS_LinuxShare01_Modify` | `<ntfs-group-name>` |
| Linux mount point | `/mnt/linuxshare01` | `<linux-mount-point>` |
| Mount options | `vers=3,rw` | `<mount-options>` |
| Firewall profile | Domain | `<firewall-profile>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Configure_NFS_Shares_For_Non_Windows_Clients_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Windows file server identity | File Server | `hostname` | Correct server is identified |
| 2 | Confirm IP configuration | File Server | `Get-NetIPConfiguration` | Server IP, DNS, and gateway are visible |
| 3 | Confirm target volume exists | File Server | `Get-Volume -DriveLetter D` | Target volume is present and healthy |
| 4 | Confirm target folder path plan | File Server | `Test-Path "D:\NFS\LinuxShare01"` | Path state is known |
| 5 | Create NFS root folder | File Server | `New-Item -ItemType Directory -Path "D:\NFS\LinuxShare01" -Force` | Folder exists |
| 6 | Confirm NFS feature state | File Server | `Get-WindowsFeature FS-NFS-Service,RSAT-NFS-Admin` | NFS role service and tools state are visible |
| 7 | Install Server for NFS | File Server | `Install-WindowsFeature FS-NFS-Service,RSAT-NFS-Admin -IncludeManagementTools` | NFS server and tools are installed |
| 8 | Import NFS module | File Server | `Import-Module NFS` | NFS PowerShell cmdlets are available |
| 9 | Inventory NFS cmdlets | File Server | `Get-Command -Module NFS` | NFS commands are listed |
| 10 | Confirm NFS service state | File Server | `Get-Service | Where-Object Name -like "*nfs*"` | NFS-related services are visible |
| 11 | Confirm NFS shares before change | File Server | `Get-NfsShare` | Existing NFS exports are visible |
| 12 | Confirm firewall rules | File Server | `Get-NetFirewallRule | Where-Object DisplayName -like "*NFS*"` | NFS firewall rules are visible |
| 13 | Enable NFS firewall rules if needed | File Server | `Get-NetFirewallRule | Where-Object DisplayName -like "*NFS*" | Enable-NetFirewallRule` | NFS rules are enabled |
| 14 | Set NTFS baseline permissions | File Server | `icacls "D:\NFS\LinuxShare01"` | Current ACL is documented |
| 15 | Create NFS share with no broad access | File Server | `New-NfsShare -Name "LinuxShare01" -Path "D:\NFS\LinuxShare01" -Authentication sys -EnableUnmappedAccess $true -Permission no-access` | NFS export is created with no broad access |
| 16 | Confirm NFS share exists | File Server | `Get-NfsShare -Name "LinuxShare01" | Format-List *` | Export details are visible |
| 17 | Create NFS client group | File Server | `New-NfsClientgroup -ClientGroupName "LinuxNFSClients"` | Client group is created |
| 18 | Add Linux client to NFS client group | File Server | `Add-NfsClientgroupMember -ClientGroupName "LinuxNFSClients" -MemberName "10.10.30.25"` | Linux client IP is added |
| 19 | Grant NFS permission to client group | File Server | `Grant-NfsSharePermission -Name "LinuxShare01" -ClientName "LinuxNFSClients" -ClientType clientgroup -Permission readwrite -AllowRootAccess $false` | Client group has read/write export access |
| 20 | Confirm NFS share permissions | File Server | `Get-NfsSharePermission -Name "LinuxShare01"` | Export permissions are visible |
| 21 | Confirm Linux client can resolve server | Linux Client | `getent hosts FS1` | Server resolves if DNS is configured |
| 22 | Confirm Linux client can reach server | Linux Client | `ping -c 4 10.10.20.10` | Basic reachability works if ICMP allowed |
| 23 | Confirm NFS RPC visibility | Linux Client | `showmount -e 10.10.20.10` | Export is visible to client |
| 24 | Create mount point | Linux Client | `sudo mkdir -p /mnt/linuxshare01` | Mount point exists |
| 25 | Mount NFS share | Linux Client | `sudo mount -t nfs -o vers=3 10.10.20.10:/LinuxShare01 /mnt/linuxshare01` | NFS export mounts |
| 26 | Confirm mounted file system | Linux Client | `mount | grep linuxshare01` | Mount is visible |
| 27 | Test read access | Linux Client | `ls -la /mnt/linuxshare01` | Directory listing works |
| 28 | Test write access | Linux Client | `echo "nfs-test" | sudo tee /mnt/linuxshare01/nfs-test.txt` | File write succeeds if readwrite access is intended |
| 29 | Confirm file appears on Windows path | File Server | `Get-ChildItem "D:\NFS\LinuxShare01"` | Linux-created file appears |
| 30 | Review NFS events | File Server | `Get-WinEvent -ListLog "*NFS*"` | NFS logs are discoverable |
| 31 | Review System events for NFS services | File Server | `Get-WinEvent -LogName System -MaxEvents 100 | Where-Object Message -like "*NFS*"` | Recent NFS-related system events are visible |
| 32 | Document export state | Operator | `Record path, client permissions, auth model, mount test, and rollback commands` | Change evidence is complete |

# Configure_NFS_Shares_For_Non_Windows_Clients_Feature_Install_Skeleton
```powershell
# Run in elevated PowerShell on the Windows file server.
# Purpose: install Server for NFS and management tooling.

$Features = @(
  "FS-NFS-Service",
  "RSAT-NFS-Admin"
)

Write-Host "Checking current feature state..."
Get-WindowsFeature $Features

Write-Host "Installing Server for NFS and management tools..."
Install-WindowsFeature $Features -IncludeManagementTools

Write-Host "Importing NFS module..."
Import-Module NFS

Write-Host "Available NFS cmdlets:"
Get-Command -Module NFS | Sort-Object Name

Write-Host "NFS-related services:"
Get-Service | Where-Object {
  $_.Name -like "*nfs*" -or $_.DisplayName -like "*NFS*"
} | Format-Table Name, DisplayName, Status, StartType -AutoSize

Write-Host "NFS firewall rules:"
Get-NetFirewallRule | Where-Object DisplayName -like "*NFS*" |
  Select-Object DisplayName, Enabled, Direction, Action, Profile |
  Format-Table -AutoSize
```

# Configure_NFS_Shares_For_Non_Windows_Clients_Export_Path_Skeleton
```powershell
# Run in elevated PowerShell on the Windows file server.
# Purpose: create and validate the Windows folder backing the NFS export.

$NfsPath = "D:\NFS\LinuxShare01"
$NtfsGroup = "CORP\GG_NFS_LinuxShare01_Modify"

Write-Host "Creating NFS folder path..."
New-Item -ItemType Directory -Path $NfsPath -Force | Out-Null

Write-Host "Current NTFS ACL:"
icacls $NfsPath

Write-Host "Granting baseline NTFS Modify rights to intended Windows group..."
icacls $NfsPath /grant "$NtfsGroup:(OI)(CI)M"

Write-Host "NTFS ACL after change:"
icacls $NfsPath

Write-Host "Creating test marker file..."
"Created on Windows file server for NFS validation" |
  Out-File -FilePath (Join-Path $NfsPath "windows-marker.txt") -Encoding utf8

Get-ChildItem $NfsPath
```

# Configure_NFS_Shares_For_Non_Windows_Clients_Unmapped_Access_Skeleton
```powershell
# Run in elevated PowerShell on the Windows file server.
# Purpose: create an NFS export using unmapped UNIX user access for a simple lab or mixed-client scenario.

$ShareName = "LinuxShare01"
$NfsPath = "D:\NFS\LinuxShare01"

Write-Host "Creating NFS share with no broad default access..."
New-NfsShare `
  -Name $ShareName `
  -Path $NfsPath `
  -Authentication sys `
  -EnableUnmappedAccess $true `
  -EnableAnonymousAccess $false `
  -AllowRootAccess $false `
  -Permission no-access

Write-Host "NFS share state:"
Get-NfsShare -Name $ShareName | Format-List *

Write-Host "NFS share permissions:"
Get-NfsSharePermission -Name $ShareName
```

# Configure_NFS_Shares_For_Non_Windows_Clients_Client_Group_Skeleton
```powershell
# Run in elevated PowerShell on the Windows file server.
# Purpose: limit NFS export access to approved non-Windows clients.

$ClientGroupName = "LinuxNFSClients"
$LinuxClientIp = "10.10.30.25"
$ShareName = "LinuxShare01"

Write-Host "Creating NFS client group..."
New-NfsClientgroup -ClientGroupName $ClientGroupName

Write-Host "Adding Linux client IP to client group..."
Add-NfsClientgroupMember `
  -ClientGroupName $ClientGroupName `
  -MemberName $LinuxClientIp

Write-Host "Granting read/write NFS access to client group..."
Grant-NfsSharePermission `
  -Name $ShareName `
  -ClientName $ClientGroupName `
  -ClientType clientgroup `
  -Permission readwrite `
  -AllowRootAccess $false

Write-Host "Client group state:"
Get-NfsClientgroup -ClientGroupName $ClientGroupName | Format-List *

Write-Host "NFS share permissions:"
Get-NfsSharePermission -Name $ShareName
```

# Configure_NFS_Shares_For_Non_Windows_Clients_Linux_Client_Mount_Skeleton
```bash
# Run on the Linux client.
# Purpose: validate NFS export visibility, mount behavior, read, and write.

SERVER_IP="10.10.20.10"
SERVER_NAME="FS1"
EXPORT_NAME="LinuxShare01"
MOUNT_POINT="/mnt/linuxshare01"

echo "Install NFS client tools if needed"
# Debian or Ubuntu:
sudo apt-get update
sudo apt-get install -y nfs-common

# RHEL, Rocky, Alma, Fedora:
# sudo dnf install -y nfs-utils

echo "Confirm name resolution if DNS is expected"
getent hosts "$SERVER_NAME"

echo "Confirm basic server reachability"
ping -c 4 "$SERVER_IP"

echo "Show NFS exports from server"
showmount -e "$SERVER_IP"

echo "Create local mount point"
sudo mkdir -p "$MOUNT_POINT"

echo "Mount NFS export"
sudo mount -t nfs -o vers=3 "$SERVER_IP:/$EXPORT_NAME" "$MOUNT_POINT"

echo "Confirm mounted filesystem"
mount | grep "$MOUNT_POINT"

echo "List mounted path"
ls -la "$MOUNT_POINT"

echo "Write test file"
echo "nfs write test from $(hostname) at $(date)" | sudo tee "$MOUNT_POINT/nfs-client-test.txt"

echo "Read test file"
cat "$MOUNT_POINT/nfs-client-test.txt"

echo "Unmount after test if this is not meant to persist"
# sudo umount "$MOUNT_POINT"
```

# Configure_NFS_Shares_For_Non_Windows_Clients_Identity_And_Permission_Skeleton
```powershell
# Run on the Windows file server.
# Purpose: inspect NTFS, NFS export permission, and share behavior together.

$ShareName = "LinuxShare01"
$NfsPath = "D:\NFS\LinuxShare01"

Write-Host "NFS export state:"
Get-NfsShare -Name $ShareName | Format-List *

Write-Host "NFS export permissions:"
Get-NfsSharePermission -Name $ShareName | Format-Table -AutoSize

Write-Host "NTFS permissions:"
icacls $NfsPath

Write-Host "Files created through NFS or Windows:"
Get-ChildItem $NfsPath -Force |
  Select-Object Name, Length, LastWriteTime, Attributes |
  Format-Table -AutoSize

Write-Host "NFS client groups:"
Get-NfsClientgroup | Format-Table -AutoSize
```

# Configure_NFS_Shares_For_Non_Windows_Clients_Event_And_Service_Check_Skeleton
```powershell
# Run in elevated PowerShell on the Windows file server.
# Purpose: collect basic NFS service, firewall, and event evidence.

$ExportRoot = "C:\NFS-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ExportRoot | Out-Null

Write-Host "NFS-related services:"
Get-Service | Where-Object {
  $_.Name -like "*nfs*" -or $_.DisplayName -like "*NFS*"
} |
  Select-Object Name, DisplayName, Status, StartType |
  Export-Csv "$ExportRoot\nfs-services-$Timestamp.csv" -NoTypeInformation

Write-Host "NFS firewall rules:"
Get-NetFirewallRule | Where-Object DisplayName -like "*NFS*" |
  Select-Object DisplayName, Enabled, Direction, Action, Profile |
  Export-Csv "$ExportRoot\nfs-firewall-rules-$Timestamp.csv" -NoTypeInformation

Write-Host "NFS share exports:"
Get-NfsShare |
  Export-Clixml "$ExportRoot\nfs-shares-$Timestamp.xml"

Write-Host "Available NFS event logs:"
Get-WinEvent -ListLog "*NFS*" -ErrorAction SilentlyContinue |
  Select-Object LogName, IsEnabled, RecordCount |
  Export-Csv "$ExportRoot\nfs-eventlogs-$Timestamp.csv" -NoTypeInformation

Write-Host "Recent System events mentioning NFS:"
Get-WinEvent -LogName System -MaxEvents 300 |
  Where-Object Message -like "*NFS*" |
  Select-Object TimeCreated, Id, ProviderName, Message |
  Export-Csv "$ExportRoot\nfs-system-events-$Timestamp.csv" -NoTypeInformation

Write-Host "NFS report data exported to $ExportRoot"
```

# Configure_NFS_Shares_For_Non_Windows_Clients_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify NFS feature | `Get-WindowsFeature FS-NFS-Service,RSAT-NFS-Admin` | NFS service and tools are installed |
| Verify NFS module | `Get-Command -Module NFS` | NFS cmdlets are available |
| Verify NFS services | `Get-Service | Where-Object DisplayName -like "*NFS*"` | NFS services are present and running as needed |
| Verify export exists | `Get-NfsShare -Name "LinuxShare01"` | NFS share exists |
| Verify export path | `Get-NfsShare -Name "LinuxShare01" | Select Name,Path,Online` | Export points to correct folder and is online |
| Verify export permissions | `Get-NfsSharePermission -Name "LinuxShare01"` | Expected client group or host has access |
| Verify client group | `Get-NfsClientgroup -ClientGroupName "LinuxNFSClients"` | Client group exists |
| Verify NTFS permissions | `icacls "D:\NFS\LinuxShare01"` | Expected NTFS access is present |
| Verify firewall rules | `Get-NetFirewallRule | Where-Object DisplayName -like "*NFS*"` | NFS firewall rules are enabled |
| Verify Linux export visibility | `showmount -e 10.10.20.10` | Linux client sees export |
| Verify Linux mount | `mount | grep linuxshare01` | NFS export is mounted |
| Verify Linux read | `ls -la /mnt/linuxshare01` | Directory listing succeeds |
| Verify Linux write | `echo test | sudo tee /mnt/linuxshare01/test.txt` | Write succeeds if readwrite is expected |
| Verify Windows sees Linux file | `Get-ChildItem "D:\NFS\LinuxShare01"` | File written by Linux appears |
| Verify NFS events | `Get-WinEvent -ListLog "*NFS*"` | NFS logs are discoverable |
| Verify System events | `Get-WinEvent -LogName System -MaxEvents 100 | Where-Object Message -like "*NFS*"` | NFS-related system events can be reviewed |

# Configure_NFS_Shares_For_Non_Windows_Clients_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| Linux mount created | `sudo umount /mnt/linuxshare01` | Linux client unmounts NFS export |
| Persistent Linux fstab entry added | `sudo nano /etc/fstab` and remove NFS line | Mount no longer persists |
| NFS share permission granted | `Revoke-NfsSharePermission -Name "LinuxShare01" -ClientName "LinuxNFSClients" -ClientType clientgroup` | Client group access is removed |
| Client group member added | `Remove-NfsClientgroupMember -ClientGroupName "LinuxNFSClients" -MemberName "10.10.30.25"` | Linux client is removed from group |
| NFS client group created | `Remove-NfsClientgroup -ClientGroupName "LinuxNFSClients"` | Client group is removed |
| NFS export created | `Remove-NfsShare -Name "LinuxShare01" -Force` | NFS export is removed |
| NTFS permission granted | `icacls "D:\NFS\LinuxShare01" /remove "CORP\GG_NFS_LinuxShare01_Modify"` | NTFS group permission is removed |
| NFS firewall rules enabled | `Get-NetFirewallRule | Where-Object DisplayName -like "*NFS*" | Disable-NetFirewallRule` | NFS firewall rules are disabled |
| NFS role installed | `Uninstall-WindowsFeature FS-NFS-Service,RSAT-NFS-Admin` | NFS role service and tools are removed |
| Test folder created | `Remove-Item "D:\NFS\LinuxShare01" -Recurse -Force` | Folder is deleted after data is backed up or confirmed disposable |
| Report folder created | `Remove-Item "C:\NFS-Reports" -Recurse -Force` | Report folder is removed |

# Configure_NFS_Shares_For_Non_Windows_Clients_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `New-NfsShare` command not found | NFS module or role missing | `Get-WindowsFeature FS-NFS-Service,RSAT-NFS-Admin` | Install Server for NFS and tools |
| NFS export not visible from Linux | Firewall, share missing, client not allowed, service issue | `Get-NfsShare`; `showmount -e <server-ip>` | Enable firewall rules and verify export |
| Linux mount times out | Network path, firewall, service stopped, wrong server IP | `Test-NetConnection`; Linux `ping`; Windows service check | Fix network, firewall, or NFS service |
| Linux sees export but cannot mount | Export permission or NFS version mismatch | `Get-NfsSharePermission`; Linux mount verbose output | Grant client access or specify version |
| Linux can mount but cannot write | NFS permission readonly, NTFS denies write, root squash behavior | `Get-NfsSharePermission`; `icacls` | Grant readwrite export permission and correct NTFS ACL |
| Linux root cannot write | Root access disabled | `Get-NfsSharePermission -Name <share>` | Enable root access only if required and approved |
| All machines accidentally have access | Broad builtin permission used | `Get-NfsSharePermission` | Revoke broad permission and grant host or client group only |
| Client group does not work | Wrong client IP, DNS mismatch, member not added | `Get-NfsClientgroup`; Linux `ip addr` | Add correct IP or hostname |
| Works by IP but not hostname | DNS issue | Linux `getent hosts <server>` | Fix DNS or use IP in lab |
| Permission behavior looks strange | UID/GID, unmapped user access, anonymous access, or NTFS ACL mismatch | `Get-NfsShare`; `icacls`; Linux `id` | Choose identity model and align folder permissions |
| Files show unexpected ownership | Unmapped or anonymous access behavior | Linux `ls -ln`; Windows ACL view | Configure mapped identities or document unmapped behavior |
| NFSv4.1 client fails but v3 works | Version, identity, firewall, or implementation mismatch | Linux `mount -t nfs -o vers=3` and `vers=4.1` | Use supported version for workload |
| `showmount` fails | RPC/NFS service or firewall issue | Windows firewall rules and services | Enable rules and confirm services |
| Write creates file but app fails | Locking, permissions, line endings, app expectation | Linux app logs, Windows events | Confirm app compatibility with Windows-backed NFS |
| SMB and NFS users conflict | Multi-protocol access without aligned identity and permissions | NTFS ACLs, NFS identity model, SMB sessions | Avoid mixed writes or design identity mapping carefully |
| NFS events missing | Wrong log name or no events generated | `Get-WinEvent -ListLog "*NFS*"` | Use discovered logs and System log |

# Configure_NFS_Shares_For_Non_Windows_Clients_Related_Labs
| Lab                                                            | Relationship                                                                               |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `01_Install_File_Server_Role_And_Management_Tools.md`          | Establishes baseline Windows file server management                                        |
| `02_Create_And_Publish_SMB_Shares.md`                          | Contrasts SMB sharing with NFS export behavior                                             |
| `03_Configure_NTFS_And_Share_Permissions.md`                   | NTFS permissions still control backing folder access                                       |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md` | Shows Windows-native file sharing security compared with NFS                               |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`      | Helps separate Windows user share design from non-Windows client exports                   |
| `12_Backup_Restore_And_Export_File_Server_Config.md`           | NFS configuration should be documented and recoverable                                     |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md` | Monitoring pattern carries over to protocol visibility and event evidence                  |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`        | Troubleshooting logic applies to NFS too: name resolution, port, auth, permissions, events |
| `17_Configure_Data_Deduplication_For_File_Shares.md`           | Deduplication can affect the same backing volumes used by SMB or NFS exports               |