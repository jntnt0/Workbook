16_Troubleshoot_File_Share_Access_And_SMB_Failures.md
# Troubleshoot_File_Share_Access_And_SMB_Failures

# Troubleshoot_File_Share_Access_And_SMB_Failures_Index
16_Troubleshoot_File_Share_Access_And_SMB_Failures.md
Troubleshoot_File_Share_Access_And_SMB_Failures
Troubleshoot_File_Share_Access_And_SMB_Failures_Source_Basis
Troubleshoot_File_Share_Access_And_SMB_Failures_Mental_Model
Troubleshoot_File_Share_Access_And_SMB_Failures_Planning_Table
Troubleshoot_File_Share_Access_And_SMB_Failures_Symptom_Map
Troubleshoot_File_Share_Access_And_SMB_Failures_Configuration_Checklist
Troubleshoot_File_Share_Access_And_SMB_Failures_Name_Resolution_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_Network_Path_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_Authentication_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_Authorization_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_SMB_Server_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_Client_Cache_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_DFS_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_Locked_File_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_Event_Log_Skeleton
Troubleshoot_File_Share_Access_And_SMB_Failures_Verification_Commands
Troubleshoot_File_Share_Access_And_SMB_Failures_Rollback
Troubleshoot_File_Share_Access_And_SMB_Failures_Failure_Checks
Troubleshoot_File_Share_Access_And_SMB_Failures_Related_Labs

# Troubleshoot_File_Share_Access_And_SMB_Failures_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server SMB PowerShell modules | Get-SmbShare, Get-SmbSession, Get-SmbOpenFile, Get-SmbServerConfiguration | Server-side SMB troubleshooting |
| Windows Client SMB PowerShell modules | Get-SmbConnection, Get-SmbMapping, New-SmbMapping, Remove-SmbMapping | Client-side SMB connection troubleshooting |
| Windows networking commands | Test-NetConnection, Resolve-DnsName, ipconfig, nltest | DNS, port, route, and domain locator validation |
| Windows permissions model | NTFS permissions plus SMB share permissions | Effective file share authorization |
| Windows Event Viewer | SMBServer, SMBClient, Security, System logs | Event-driven troubleshooting evidence |
| DFS tools | dfsutil, Get-DfsnFolderTarget, Get-DfsReplicationGroup | DFS namespace and target path troubleshooting |
| Operational practice | User impact, ticket notes, controlled remediation | Safe support workflow |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Mental_Model
| Concept | Operational Meaning |
|---|---|
| UNC path | Network path like `\\FS1\Departments` |
| DNS dependency | User must resolve the file server name before SMB can work |
| TCP 445 | Primary SMB port used for file sharing |
| Authentication | The user must prove identity to the server or domain |
| Authorization | NTFS and share permissions decide what the user can do |
| Most restrictive wins | Effective access is constrained by both share permissions and NTFS permissions |
| Access-based enumeration | Users only see folders they have permission to read |
| SMB session | Authenticated user connection to the file server |
| Open file handle | Active file access over SMB that can block rename, delete, edit, or save |
| DFS namespace | Friendly path that redirects clients to real SMB targets |
| DFS target | Actual file server and share behind a DFS namespace |
| Client cache | Cached DNS, Kerberos tickets, SMB mappings, and offline files can preserve old failures |
| Event logs | Evidence source for authentication, access denial, SMB transport, and service failures |
| Blunt rule | Do not start by changing permissions. Prove whether the problem is DNS, network, auth, share, NTFS, DFS, lock, or client cache first |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Planning_Table
| Item | Example | Decision |
|---|---|---|
| User reporting issue | `corp\jsmith` | `<username>` |
| User workstation | `WIN11-01` | `<client-name>` |
| File server | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain | `corp.local` | `<domain-fqdn>` |
| Target UNC path | `\\FS1\Departments` | `<unc-path>` |
| DFS path if used | `\\corp.local\Shares\Departments` | `<dfs-path>` |
| Target share | `Departments` | `<share-name>` |
| Local share path | `D:\Shares\Departments` | `<local-share-path>` |
| Target folder | `D:\Shares\Departments\Accounting` | `<target-folder>` |
| Reported error | `Access denied` | `<reported-error>` |
| Expected access | Read / Modify / Full Control | `<expected-access>` |
| User group | `GG_FS_Departments_Modify` | `<group-name>` |
| Test file | `smb-test.txt` | `<test-file>` |
| Time issue started | `2026-06-15 09:30` | `<start-time>` |
| Is issue one user or many | One / Many | `<scope>` |
| Is issue one share or many | One / Many | `<share-scope>` |
| Is issue one client or many | One / Many | `<client-scope>` |
| Is DFS involved | Yes / No | `<dfs-state>` |
| Change control approval | Ticket number | `<ticket-id>` |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Symptom_Map
| Symptom | Most Likely Layer | First Check | Common Fix |
|---|---|---|---|
| `The network path was not found` | DNS, routing, firewall, SMB service | `Resolve-DnsName`; `Test-NetConnection -Port 445` | Fix DNS, route, firewall, or LanmanServer |
| `Access is denied` | Share permissions, NTFS permissions, group membership | `Get-SmbShareAccess`; `icacls`; `whoami /groups` | Correct group membership or ACLs |
| User sees share but not folder | Access-based enumeration or NTFS read permission | `Get-SmbShare`; `icacls` | Grant required NTFS read/list access |
| User can read but not write | NTFS modify missing, share change missing, file locked, quota | `icacls`; `Get-SmbShareAccess`; `Get-SmbOpenFile`; FSRM quota | Correct ACL, close lock, or adjust quota |
| User cannot delete or rename | NTFS delete missing or file handle open | `icacls`; `Get-SmbOpenFile` | Grant delete or close handle with approval |
| Credential prompt appears repeatedly | Wrong saved credential, SPN/Kerberos issue, workgroup mismatch | `cmdkey /list`; `klist`; `nltest /dsgetdc:<domain>` | Remove cached credential or fix domain auth |
| Works by IP but not name | DNS problem | `Resolve-DnsName <server>` | Fix A record, DNS client settings, or suffix |
| Works by server name but not DFS path | DFS namespace or referral issue | `dfsutil /pktinfo`; `Get-DfsnFolderTarget` | Fix DFS target state or namespace |
| One client fails, others work | Client cache, firewall profile, DNS cache, mapping | `Get-SmbConnection`; `net use`; `ipconfig /displaydns` | Clear cache and remap |
| One user fails, others work | User permissions or group token | `whoami /groups`; `Get-ADPrincipalGroupMembership` | Correct group membership and refresh token |
| All users fail | Server service, share missing, firewall, disk offline, DNS | `Get-Service LanmanServer`; `Get-SmbShare`; `Test-NetConnection` | Restore server-side dependency |
| File says it is in use | Open file handle or app lock | `Get-SmbOpenFile` | Close specific file handle with approval |
| Very slow access | Network, SMB signing/encryption overhead, disk, antivirus, DFS referral, WAN | `Get-Counter`; `Test-NetConnection`; event logs | Isolate performance bottleneck |
| Offline files conflict | Client-side caching | Sync Center, Offline Files service | Resolve conflict or disable caching if policy allows |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record exact user error | Client | `Record screenshot or exact error text` | Error is documented before changes |
| 2 | Confirm exact path being used | Client | `echo <unc-path>` | UNC or DFS path is known |
| 3 | Confirm client identity | Client | `hostname` | Correct workstation is identified |
| 4 | Confirm logged-on user | Client | `whoami` | Correct user context is confirmed |
| 5 | Confirm user group token | Client | `whoami /groups` | Expected access group appears in current token |
| 6 | Confirm IP configuration | Client | `ipconfig /all` | DNS servers and domain suffix are correct |
| 7 | Resolve file server name | Client | `Resolve-DnsName <file-server-name>` | File server resolves to expected IP |
| 8 | Resolve file server FQDN | Client | `Resolve-DnsName <file-server-fqdn>` | FQDN resolves to expected IP |
| 9 | Test SMB port | Client | `Test-NetConnection <file-server-name> -Port 445` | TCP 445 succeeds |
| 10 | Test basic UNC path | Client | `Test-Path "\\<file-server-name>\<share-name>"` | Returns True if reachable and authorized |
| 11 | Check existing SMB mappings | Client | `Get-SmbMapping` | Current mappings are visible |
| 12 | Check legacy drive mappings | Client | `net use` | Legacy mapped drives are visible |
| 13 | Check client SMB connections | Client | `Get-SmbConnection` | Active SMB server/share connections are visible |
| 14 | Check cached credentials | Client | `cmdkey /list` | Saved credentials are visible |
| 15 | Check Kerberos tickets | Client | `klist` | Domain tickets are visible |
| 16 | Confirm domain controller discovery | Client | `nltest /dsgetdc:<domain-fqdn>` | Client can locate a DC |
| 17 | Confirm SMB server service | File Server | `Get-Service LanmanServer` | Service is running |
| 18 | Confirm file server shares | File Server | `Get-SmbShare` | Target share is present |
| 19 | Inspect target share | File Server | `Get-SmbShare -Name "<share-name>" | Format-List *` | Path and share configuration are correct |
| 20 | Inspect share permissions | File Server | `Get-SmbShareAccess -Name "<share-name>"` | Expected groups have required share access |
| 21 | Inspect NTFS permissions | File Server | `icacls "<local-share-path>"` | Expected groups have required NTFS permissions |
| 22 | Inspect target folder permissions | File Server | `icacls "<target-folder>"` | Folder-level inheritance and explicit ACLs are known |
| 23 | Confirm path exists locally | File Server | `Test-Path "<local-share-path>"` | Local backing path exists |
| 24 | Confirm volume state | File Server | `Get-Volume` | Volume is healthy and online |
| 25 | Confirm free space | File Server | `Get-PSDrive -PSProvider FileSystem` | Sufficient free space exists |
| 26 | Confirm SMB server config | File Server | `Get-SmbServerConfiguration | Select EnableSMB1Protocol,EnableSMB2Protocol,RequireSecuritySignature,EnableSecuritySignature,EncryptData,RejectUnencryptedAccess` | SMB posture is known |
| 27 | Check active sessions | File Server | `Get-SmbSession` | Client and user sessions are visible |
| 28 | Check open file handles | File Server | `Get-SmbOpenFile` | Active file handles are visible |
| 29 | Filter open files by user | File Server | `Get-SmbOpenFile | Where-Object ClientUserName -like "*<username>*"` | User handles are visible |
| 30 | Filter open files by file name | File Server | `Get-SmbOpenFile | Where-Object Path -like "*<filename>*"` | Locked file is located if present |
| 31 | Check SMBServer events | File Server | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 100` | Recent SMB server events are visible |
| 32 | Check SMBClient events | Client | `Get-WinEvent -LogName "Microsoft-Windows-SMBClient/Connectivity" -MaxEvents 100` | Recent SMB client connection events are visible |
| 33 | Check Security share access events | File Server | `Get-WinEvent -FilterHashtable @{LogName="Security"; Id=5140,5145; StartTime=(Get-Date).AddHours(-4)}` | Share access events are visible if auditing is enabled |
| 34 | If DFS path used, inspect DFS referral cache | Client | `dfsutil /pktinfo` | DFS referrals are visible |
| 35 | If DFS path used, inspect DFS target | File Server or DC | `Get-DfsnFolderTarget -Path "<dfs-path>"` | DFS targets and state are visible |
| 36 | Clear bad client mappings if needed | Client | `net use * /delete` | Existing SMB mappings are removed after approval |
| 37 | Clear DNS cache if needed | Client | `ipconfig /flushdns` | Client DNS cache is cleared |
| 38 | Clear Kerberos cache if needed | Client | `klist purge` | Current Kerberos tickets are removed |
| 39 | Remap target path | Client | `New-SmbMapping -LocalPath "Z:" -RemotePath "\\<file-server-name>\<share-name>"` | Mapping succeeds |
| 40 | Retest access | Client | `Test-Path "\\<file-server-name>\<share-name>"` | Access result matches expected state |
| 41 | Document root cause and fix | Operator | `Record failing layer, command proof, remediation, and validation` | Ticket has operational evidence |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Name_Resolution_Skeleton
```powershell
# Run on the affected client.
# Purpose: prove whether SMB failure is caused by name resolution.

$ServerName = "<file-server-name>"
$ServerFqdn = "<file-server-fqdn>"
$DomainName = "<domain-fqdn>"

Write-Host "Client Identity"
hostname
whoami

Write-Host "`nIP Configuration"
ipconfig /all

Write-Host "`nDNS Resolution: short name"
Resolve-DnsName $ServerName -ErrorAction Continue

Write-Host "`nDNS Resolution: FQDN"
Resolve-DnsName $ServerFqdn -ErrorAction Continue

Write-Host "`nDomain Controller Discovery"
nltest /dsgetdc:$DomainName

Write-Host "`nDNS Cache Entries Matching Server"
ipconfig /displaydns | Select-String $ServerName
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_Network_Path_Skeleton
```powershell
# Run on the affected client.
# Purpose: prove whether the SMB server is reachable over TCP 445.

$ServerName = "<file-server-name>"
$ShareName = "<share-name>"
$UncPath = "\\$ServerName\$ShareName"

Write-Host "Testing ICMP reachability if allowed"
Test-Connection $ServerName -Count 4 -ErrorAction Continue

Write-Host "`nTesting SMB TCP 445"
Test-NetConnection $ServerName -Port 445

Write-Host "`nTesting UNC path"
Test-Path $UncPath

Write-Host "`nExisting SMB mappings"
Get-SmbMapping -ErrorAction SilentlyContinue

Write-Host "`nExisting SMB connections"
Get-SmbConnection -ErrorAction SilentlyContinue

Write-Host "`nLegacy net use output"
net use
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_Authentication_Skeleton
```powershell
# Run on the affected client.
# Purpose: isolate authentication, cached credential, or Kerberos problems.

$DomainName = "<domain-fqdn>"
$ServerName = "<file-server-name>"

Write-Host "Logged-on user"
whoami

Write-Host "`nUser group token"
whoami /groups

Write-Host "`nDomain controller discovery"
nltest /dsgetdc:$DomainName

Write-Host "`nKerberos tickets"
klist

Write-Host "`nCached Windows credentials"
cmdkey /list

Write-Host "`nCurrent SMB connections"
Get-SmbConnection -ErrorAction SilentlyContinue

Write-Host "`nTarget SMB port"
Test-NetConnection $ServerName -Port 445

# Optional remediation after approval:
# klist purge
# cmdkey /delete:<target-name>
# net use * /delete
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_Authorization_Skeleton
```powershell
# Run on the file server and affected client.
# Purpose: compare user token, share permissions, and NTFS permissions.

$ShareName = "<share-name>"
$LocalPath = "<local-share-path>"
$TargetFolder = "<target-folder>"
$User = "<domain-fqdn>\<username>"

Write-Host "Server-side share permissions"
Get-SmbShareAccess -Name $ShareName

Write-Host "`nServer-side share details"
Get-SmbShare -Name $ShareName | Format-List *

Write-Host "`nNTFS permissions on share root"
icacls $LocalPath

Write-Host "`nNTFS permissions on target folder"
icacls $TargetFolder

Write-Host "`nLocal path exists"
Test-Path $LocalPath

Write-Host "`nOptional effective access check requires GUI or AccessChk-style tooling if available."
Write-Host "Compare expected group membership with share and NTFS ACLs."
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_SMB_Server_Skeleton
```powershell
# Run on the file server.
# Purpose: confirm SMB server health and share publication.

$ShareName = "<share-name>"

Write-Host "SMB server service"
Get-Service LanmanServer

Write-Host "`nWorkstation service"
Get-Service LanmanWorkstation

Write-Host "`nSMB server configuration"
Get-SmbServerConfiguration |
  Select-Object `
    EnableSMB1Protocol,
    EnableSMB2Protocol,
    EnableSecuritySignature,
    RequireSecuritySignature,
    EncryptData,
    RejectUnencryptedAccess,
    EnableLeasing,
    EnableOplocks |
  Format-List

Write-Host "`nPublished SMB shares"
Get-SmbShare

Write-Host "`nTarget share"
Get-SmbShare -Name $ShareName | Format-List *

Write-Host "`nFirewall rules for File and Printer Sharing"
Get-NetFirewallRule -DisplayGroup "File and Printer Sharing" |
  Select-Object DisplayName, Enabled, Direction, Action, Profile |
  Format-Table -AutoSize

Write-Host "`nActive SMB sessions"
Get-SmbSession |
  Select-Object ClientComputerName, ClientUserName, Dialect, NumOpens, Signed, Encrypted, SecondsExists |
  Format-Table -AutoSize
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_Client_Cache_Skeleton
```powershell
# Run on the affected client after approval.
# Purpose: remove stale client-side state that can preserve access failures.

$ServerName = "<file-server-name>"
$ShareName = "<share-name>"
$DriveLetter = "Z:"
$UncPath = "\\$ServerName\$ShareName"

Write-Host "Existing SMB mappings"
Get-SmbMapping -ErrorAction SilentlyContinue

Write-Host "`nExisting net use mappings"
net use

Write-Host "`nExisting SMB connections"
Get-SmbConnection -ErrorAction SilentlyContinue

Write-Host "`nFlush DNS cache"
ipconfig /flushdns

Write-Host "`nPurge Kerberos tickets"
klist purge

Write-Host "`nRemove existing mapping for drive letter if present"
Remove-SmbMapping -LocalPath $DriveLetter -Force -ErrorAction SilentlyContinue

Write-Host "`nRemove legacy mappings"
net use $DriveLetter /delete /y

Write-Host "`nCreate fresh SMB mapping"
New-SmbMapping -LocalPath $DriveLetter -RemotePath $UncPath

Write-Host "`nValidate mapping"
Get-SmbMapping
Test-Path $UncPath
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_DFS_Skeleton
```powershell
# Run on affected client and management host with DFS tools available.
# Purpose: isolate DFS namespace or referral problems.

$DfsPath = "<dfs-path>"

Write-Host "Client DFS referral cache"
dfsutil /pktinfo

Write-Host "`nDFS domain cache"
dfsutil /spcinfo

Write-Host "`nFlush DFS referral cache if needed"
# dfsutil /pktflush

Write-Host "`nInspect DFS namespace target"
Get-DfsnFolderTarget -Path $DfsPath -ErrorAction Continue

Write-Host "`nValidate DFS path"
Test-Path $DfsPath

Write-Host "`nClient SMB connections after DFS path access"
Get-SmbConnection
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_Locked_File_Skeleton
```powershell
# Run on the file server.
# Purpose: locate locked files and close only the exact handle after approval.

$TargetUser = "<username>"
$TargetFileName = "<filename>"

Write-Host "Open files by target user"
Get-SmbOpenFile |
  Where-Object ClientUserName -like "*$TargetUser*" |
  Select-Object FileId, SessionId, ClientComputerName, ClientUserName, Path, Permissions, Locks |
  Format-Table -AutoSize

Write-Host "`nOpen files by filename"
Get-SmbOpenFile |
  Where-Object Path -like "*$TargetFileName*" |
  Select-Object FileId, SessionId, ClientComputerName, ClientUserName, Path, Permissions, Locks |
  Format-Table -AutoSize

Write-Host "`nDo not close handles blindly."
Write-Host "Only after approval:"
Write-Host "Close-SmbOpenFile -FileId <file-id> -Force"
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_Event_Log_Skeleton
```powershell
# Run on the file server and affected client as appropriate.
# Purpose: collect event evidence around SMB and file share access.

$ExportRoot = "C:\SMB-Troubleshooting\Events"
$StartTime = (Get-Date).AddHours(-8)
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ExportRoot | Out-Null

Write-Host "Available SMB logs"
Get-WinEvent -ListLog "*SMB*" |
  Select-Object LogName, IsEnabled, RecordCount |
  Sort-Object LogName |
  Export-Csv "$ExportRoot\smb-log-inventory-$Timestamp.csv" -NoTypeInformation

Write-Host "Export SMBServer operational events"
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-SMBServer/Operational"
  StartTime = $StartTime
} -ErrorAction SilentlyContinue |
  Export-Clixml "$ExportRoot\smbserver-operational-$Timestamp.xml"

Write-Host "Export SMBClient connectivity events"
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-SMBClient/Connectivity"
  StartTime = $StartTime
} -ErrorAction SilentlyContinue |
  Export-Clixml "$ExportRoot\smbclient-connectivity-$Timestamp.xml"

Write-Host "Export Security file share events"
Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = 5140,5142,5143,5144,5145
  StartTime = $StartTime
} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated, Id, ProviderName, Message |
  Export-Csv "$ExportRoot\security-file-share-events-$Timestamp.csv" -NoTypeInformation

Write-Host "Exports saved to $ExportRoot"
```

# Troubleshoot_File_Share_Access_And_SMB_Failures_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify client identity | `hostname; whoami` | Correct client and user are confirmed |
| Verify DNS | `Resolve-DnsName <file-server-name>` | Correct server IP is returned |
| Verify SMB port | `Test-NetConnection <file-server-name> -Port 445` | `TcpTestSucceeded : True` |
| Verify UNC reachability | `Test-Path "\\<file-server-name>\<share-name>"` | Returns True when reachable and authorized |
| Verify client mappings | `Get-SmbMapping` | Mappings are visible and correct |
| Verify client connections | `Get-SmbConnection` | Active SMB connections show server and share |
| Verify cached credentials | `cmdkey /list` | No wrong saved credential for file server |
| Verify Kerberos tickets | `klist` | Tickets are present for expected domain resources |
| Verify DC discovery | `nltest /dsgetdc:<domain-fqdn>` | Domain controller is discovered |
| Verify server service | `Get-Service LanmanServer` | Service is running |
| Verify share exists | `Get-SmbShare -Name "<share-name>"` | Share is present |
| Verify share permissions | `Get-SmbShareAccess -Name "<share-name>"` | Correct groups have access |
| Verify NTFS ACL | `icacls "<local-share-path>"` | Correct groups have expected NTFS rights |
| Verify sessions | `Get-SmbSession` | Active user and client sessions appear |
| Verify open files | `Get-SmbOpenFile` | Active file handles are visible |
| Verify SMB server events | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 20` | Recent server SMB events return |
| Verify share audit events | `Get-WinEvent -FilterHashtable @{LogName="Security"; Id=5140,5145; StartTime=(Get-Date).AddHours(-1)}` | Share access events return if auditing is enabled |
| Verify DFS referral | `dfsutil /pktinfo` | DFS referral cache shows target server |
| Verify DFS target | `Get-DfsnFolderTarget -Path "<dfs-path>"` | DFS target is online and expected |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| Removed SMB mapping | `New-SmbMapping -LocalPath "Z:" -RemotePath "\\<file-server-name>\<share-name>"` | Mapping is recreated |
| Removed legacy drive mapping | `net use Z: \\<file-server-name>\<share-name> /persistent:yes` | Drive mapping is recreated |
| Cleared Kerberos tickets | No direct rollback | User obtains new tickets automatically |
| Flushed DNS cache | No direct rollback | Client rebuilds DNS cache automatically |
| Deleted cached credential | `cmdkey /add:<target> /user:<user> /pass:<password>` | Credential is restored only if explicitly needed |
| Changed share permission | `Grant-SmbShareAccess -Name "<share-name>" -AccountName "<group>" -AccessRight <right> -Force` | Share permission is restored |
| Removed share permission | `Revoke-SmbShareAccess -Name "<share-name>" -AccountName "<group>" -Force` | Unwanted share permission is removed |
| Changed NTFS ACL | Restore from ACL backup or documented `icacls` command | NTFS permissions return to prior state |
| Closed open file handle | No clean rollback | User must reopen the file or application |
| Closed SMB session | No clean rollback | Client reconnects on next share access |
| Flushed DFS referral cache | No direct rollback | Client rebuilds referral cache |
| Created troubleshooting export folder | `Remove-Item "C:\SMB-Troubleshooting" -Recurse -Force` | Export folder is removed |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Failure_Checks
| Failure | Likely Cause | Proof Command | Fix |
|---|---|---|---|
| Name does not resolve | DNS A record missing, wrong DNS server, suffix issue | `Resolve-DnsName <file-server-name>` | Fix DNS record or client DNS settings |
| FQDN resolves but short name fails | DNS suffix search issue | `ipconfig /all` | Correct DNS suffix or use FQDN |
| TCP 445 fails | Firewall, routing, server down, SMB service stopped | `Test-NetConnection <server> -Port 445` | Fix network path, firewall, or LanmanServer |
| Server responds but share missing | Share deleted, renamed, or wrong server | `Get-SmbShare` | Recreate or correct share path |
| Share exists but path fails | Local folder missing or volume offline | `Test-Path <local-path>`; `Get-Volume` | Restore folder or volume |
| User lacks access | Missing group membership or ACL mismatch | `whoami /groups`; `Get-SmbShareAccess`; `icacls` | Add proper group or ACL |
| User recently added to group but still denied | User token stale | `whoami /groups` | Sign out and back in, or purge tickets |
| Repeated credential prompt | Bad cached credential or auth mismatch | `cmdkey /list`; `klist` | Delete cached credential and purge tickets |
| Works for admin only | NTFS or share permissions too restrictive | `icacls`; `Get-SmbShareAccess` | Grant least-privilege group access |
| Can read but cannot write | Missing modify rights or share change rights | `icacls`; `Get-SmbShareAccess` | Grant Modify NTFS and Change share access as designed |
| Cannot delete file | Delete permission missing or file locked | `icacls`; `Get-SmbOpenFile` | Grant Delete or close handle with approval |
| File is locked | Open SMB handle | `Get-SmbOpenFile` | Close exact handle only after approval |
| DFS path fails but direct server path works | Bad DFS referral or target offline | `dfsutil /pktinfo`; `Get-DfsnFolderTarget` | Correct DFS target state or namespace |
| Direct server path fails but DFS path works | User is testing wrong server | `Get-SmbConnection` | Identify real target and troubleshoot that server |
| Access-based enumeration hides folder | User lacks list/read permission | `icacls <folder>` | Grant proper read/list permissions |
| Offline files cause stale view | Client-side cache conflict | Sync Center, Offline Files status | Resolve conflict or disable caching per policy |
| Slow access | Disk, network, SMB signing/encryption, antivirus, quota, DFS over WAN | `Get-Counter`; event logs; `Test-NetConnection` | Isolate bottleneck and remediate |
| Events missing | Auditing disabled | `auditpol /get /subcategory:"File Share"` | Enable File Share and Detailed File Share auditing |
| SMB client events missing | Wrong client or log unavailable | `Get-WinEvent -ListLog "*SMBClient*"` | Query correct client and available logs |

# Troubleshoot_File_Share_Access_And_SMB_Failures_Related_Labs
| Lab | Relationship |
|---|---|
| `01_Install_File_Server_Role_And_Management_Tools.md` | Provides baseline file server role and management tools |
| `02_Create_And_Publish_SMB_Shares.md` | Share creation must be validated during troubleshooting |
| `03_Configure_NTFS_And_Share_Permissions.md` | Most access denied cases trace back to NTFS plus share permission mismatch |
| `04_Configure_Access_Based_Enumeration_And_Share_Visibility.md` | Explains why users can be authorized to a share but unable to see folders |
| `05_Configure_Access_Based_Enumeration_And_Share_Visibility.md` | Related share visibility and folder enumeration behavior |
| `06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md` | SMB signing, encryption, and auditing affect connection and event evidence |
| `07_Configure_User_Home_Folders_And_Department_Shares.md` | User-specific paths commonly create access and mapping issues |
| `08_Configure_File_Screening_With_FSRM.md` | File screens can look like SMB write failures |
| `09_Configure_Storage_Quotas_With_FSRM.md` | Quotas can appear as denied writes or failed saves |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md` | Restore operations can be confused with access or lock issues |
| `13_Configure_DFS_Namespaces.md` | DFS referrals determine which file server actually receives SMB traffic |
| `14_Configure_DFS_Replication.md` | DFSR problems can be mistaken for SMB share failure |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md` | Monitoring workbook provides the evidence-gathering layer for this troubleshooting workflow |