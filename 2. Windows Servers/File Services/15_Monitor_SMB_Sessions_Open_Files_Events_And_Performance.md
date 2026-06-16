15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md
# Monitor_SMB_Sessions_Open_Files_Events_And_Performance

# Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Index
15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md
Monitor_SMB_Sessions_Open_Files_Events_And_Performance
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Source_Basis
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Mental_Model
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Planning_Table
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Configuration_Checklist
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Session_Inventory_Skeleton
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Open_File_Inventory_Skeleton
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Event_Log_Skeleton
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Performance_Counter_Skeleton
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Active_Triage_Skeleton
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Verification_Commands
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Rollback
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Failure_Checks
Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Related_Labs

# Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server SMB Server PowerShell module | Get-SmbSession, Get-SmbOpenFile, Get-SmbShare, Get-SmbServerConfiguration | Monitoring active SMB sessions, open files, shares, and server posture |
| Windows Server SMB Client PowerShell module | Get-SmbConnection, Get-SmbClientConfiguration | Client-side SMB connection validation |
| Windows Event Logs | SMBServer, SMBClient, Security audit events | Troubleshooting authentication, access, disconnects, lease/oplock, and share access issues |
| Windows Performance Counters | SMB Server Shares, SMB Server Sessions, SMB Direct Connection | Measuring SMB throughput, latency, request rate, and server load |
| Advanced Audit Policy | File Share and Detailed File Share auditing | Security event visibility for share access |
| Operational practice | Locked files, stale sessions, latency, throughput, user impact | Real support workflow for file server operations |

# Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Mental_Model
| Concept | Operational Meaning |
|---|---|
| SMB session | Authenticated client connection to the SMB server |
| Open file | File handle currently held open by a client over SMB |
| File lock | Active lock that can block other users or applications from modifying a file |
| SMB dialect | SMB protocol version negotiated between client and server |
| Signed session | SMB session protected with message signing |
| Encrypted session | SMB session protected with SMB encryption |
| Share access event | Event showing that a user or computer accessed a share |
| Detailed file share event | Event showing more granular access attempts against objects over a share |
| SMB operational event | SMBServer or SMBClient event used to troubleshoot protocol behavior |
| Performance counter | Time-series metric used to identify SMB latency, throughput, and load |
| Stale session | Session that remains after a client disconnect, crash, sleep state, or network interruption |
| Locked file problem | A user cannot edit, delete, move, or save a file because another session has an active handle |
| First rule | Do not close SMB sessions or open files blindly on a production file server |
| Blunt rule | Monitoring SMB is not just checking shares exist. You need sessions, open files, events, counters, and user impact together |

# Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Management host | `MGMT01` | `<management-host>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Target share | `Departments` | `<share-name>` |
| Target share path | `D:\Shares\Departments` | `<share-path>` |
| Expected client subnet | `10.10.20.0/24` | `<client-subnet>` |
| Expected SMB dialect | `3.1.1` | `<expected-smb-dialect>` |
| Expected signing state | Enabled / Required | `<signing-state>` |
| Expected encryption state | Disabled / Enabled per share | `<encryption-state>` |
| Audit policy stance | Enabled for File Share and Detailed File Share | `<audit-policy-state>` |
| SMB event export path | `C:\SMB-Monitoring\Events` | `<event-export-path>` |
| SMB report path | `C:\SMB-Monitoring\Reports` | `<report-path>` |
| Performance sample duration | `60 seconds` | `<sample-duration>` |
| Performance sample interval | `5 seconds` | `<sample-interval>` |
| Alert threshold | High latency or excessive opens | `<alert-threshold>` |
| Change control stance | Manual close only after approval | `<close-handle-policy>` |

# Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm file server identity | File Server | `hostname` | Correct file server is identified |
| 2 | Confirm server IP configuration | File Server | `Get-NetIPConfiguration` | Server IP, DNS, and gateway are visible |
| 3 | Confirm SMB Server service state | File Server | `Get-Service LanmanServer` | Server service is running |
| 4 | Confirm Workstation service state | File Server | `Get-Service LanmanWorkstation` | Workstation service is running |
| 5 | Confirm SMB shares | File Server | `Get-SmbShare` | Expected shares are listed |
| 6 | Confirm target share details | File Server | `Get-SmbShare -Name "<share-name>" | Format-List *` | Target share path and properties are visible |
| 7 | Confirm SMB server configuration | File Server | `Get-SmbServerConfiguration | Select EnableSMB1Protocol,EnableSMB2Protocol,RequireSecuritySignature,EnableSecuritySignature,EncryptData` | SMB protocol and security posture are known |
| 8 | Confirm active SMB sessions | File Server | `Get-SmbSession` | Active client sessions are listed |
| 9 | Review session users and clients | File Server | `Get-SmbSession | Select ClientComputerName,ClientUserName,Dialect,NumOpens,Signed,Encrypted,SecondsExists` | Session source, user, dialect, opens, signing, and encryption are visible |
| 10 | Filter sessions for a user | File Server | `Get-SmbSession | Where-Object ClientUserName -like "*<username>*"` | User sessions are isolated |
| 11 | Filter sessions for a client | File Server | `Get-SmbSession | Where-Object ClientComputerName -like "*<client-name>*"` | Client sessions are isolated |
| 12 | Confirm open files | File Server | `Get-SmbOpenFile` | Active SMB file handles are listed |
| 13 | Review open file paths | File Server | `Get-SmbOpenFile | Select ClientComputerName,ClientUserName,Path,Permissions,Locks` | Open files and locks are visible |
| 14 | Filter open files by share path | File Server | `Get-SmbOpenFile | Where-Object Path -like "D:\Shares\Departments*"` | Open files inside target share are isolated |
| 15 | Filter open files by username | File Server | `Get-SmbOpenFile | Where-Object ClientUserName -like "*<username>*"` | User file handles are isolated |
| 16 | Filter open files by filename | File Server | `Get-SmbOpenFile | Where-Object Path -like "*<filename>*"` | Target file handle is located |
| 17 | Confirm client-side SMB connection | Client | `Get-SmbConnection` | Client shows active SMB connection to file server |
| 18 | Confirm client-side SMB dialect | Client | `Get-SmbConnection | Select ServerName,ShareName,Dialect,Signed,Encrypted` | Client negotiated expected dialect and security state |
| 19 | List SMB event logs | File Server | `Get-WinEvent -ListLog "*SMB*" | Select LogName,RecordCount,IsEnabled` | SMB logs and enabled state are visible |
| 20 | Review SMBServer operational events | File Server | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 100` | Recent SMB server events are visible |
| 21 | Review SMBClient connectivity events | Client | `Get-WinEvent -LogName "Microsoft-Windows-SMBClient/Connectivity" -MaxEvents 100` | Client SMB connectivity events are visible |
| 22 | Review SMBClient operational events | Client | `Get-WinEvent -LogName "Microsoft-Windows-SMBClient/Operational" -MaxEvents 100` | Client SMB operational events are visible |
| 23 | Confirm file share audit policy | File Server | `auditpol /get /subcategory:"File Share"` | File Share auditing state is known |
| 24 | Confirm detailed file share audit policy | File Server | `auditpol /get /subcategory:"Detailed File Share"` | Detailed File Share auditing state is known |
| 25 | Enable File Share auditing if required | File Server | `auditpol /set /subcategory:"File Share" /success:enable /failure:enable` | File Share auditing is enabled |
| 26 | Enable Detailed File Share auditing if required | File Server | `auditpol /set /subcategory:"Detailed File Share" /success:enable /failure:enable` | Detailed File Share auditing is enabled |
| 27 | Review Security file share events | File Server | `Get-WinEvent -FilterHashtable @{LogName="Security"; Id=5140,5142,5143,5144,5145; StartTime=(Get-Date).AddHours(-4)}` | Recent share access and share change events are visible |
| 28 | List SMB performance counter sets | File Server | `Get-Counter -ListSet "*SMB*" | Select CounterSetName` | Available SMB counter sets are visible |
| 29 | Sample SMB Server Shares counters | File Server | `Get-Counter "\SMB Server Shares(*)\*" -SampleInterval 5 -MaxSamples 12` | Share-level SMB performance data is collected |
| 30 | Sample SMB Server Sessions counters | File Server | `Get-Counter "\SMB Server Sessions(*)\*" -SampleInterval 5 -MaxSamples 12` | Session-level SMB performance data is collected |
| 31 | Export session inventory | File Server | `Get-SmbSession | Export-Csv "C:\SMB-Monitoring\Reports\smb-sessions.csv" -NoTypeInformation` | Session report is saved |
| 32 | Export open file inventory | File Server | `Get-SmbOpenFile | Export-Csv "C:\SMB-Monitoring\Reports\smb-open-files.csv" -NoTypeInformation` | Open file report is saved |
| 33 | Export SMB events | File Server | `Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 500 | Export-Clixml "C:\SMB-Monitoring\Events\smbserver-operational.xml"` | SMB event export is saved |
| 34 | Export performance samples | File Server | `Get-Counter "\SMB Server Shares(*)\*" -SampleInterval 5 -MaxSamples 12 | Export-Clixml "C:\SMB-Monitoring\Reports\smb-performance.xml"` | Performance capture is saved |
| 35 | Document findings | Operator | `Record sessions, open files, top users, top clients, events, counters, and action taken` | Monitoring evidence is complete |

# Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Session_Inventory_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: collect SMB session inventory for operational review.

$ReportRoot = "C:\SMB-Monitoring\Reports"
New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$SessionCsv = Join-Path $ReportRoot "smb-sessions-$Timestamp.csv"
$SessionTxt = Join-Path $ReportRoot "smb-sessions-$Timestamp.txt"

Get-SmbSession |
  Select-Object `
    SessionId,
    ClientComputerName,
    ClientUserName,
    Dialect,
    NumOpens,
    Signed,
    Encrypted,
    SecondsExists |
  Sort-Object ClientComputerName, ClientUserName |
  Tee-Object -FilePath $SessionTxt |
  Export-Csv -Path $SessionCsv -NoTypeInformation

Write-Host "SMB session inventory exported:"
Write-Host $SessionCsv
Write-Host $SessionTxt
```


Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Open_File_Inventory_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: collect open file handles and lock visibility.

$ReportRoot = "C:\SMB-Monitoring\Reports"
New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OpenFileCsv = Join-Path $ReportRoot "smb-open-files-$Timestamp.csv"
$OpenFileTxt = Join-Path $ReportRoot "smb-open-files-$Timestamp.txt"

Get-SmbOpenFile |
  Select-Object `
    FileId,
    SessionId,
    ClientComputerName,
    ClientUserName,
    Path,
    Permissions,
    Locks |
  Sort-Object ClientComputerName, ClientUserName, Path |
  Tee-Object -FilePath $OpenFileTxt |
  Export-Csv -Path $OpenFileCsv -NoTypeInformation

Write-Host "SMB open file inventory exported:"
Write-Host $OpenFileCsv
Write-Host $OpenFileTxt
```


Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Event_Log_Skeleton
```

# Run in elevated PowerShell on the file server.
# Purpose: collect SMB operational and file share security events.

$EventRoot = "C:\SMB-Monitoring\Events"
New-Item -ItemType Directory -Force -Path $EventRoot | Out-Null

$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$StartTime = (Get-Date).AddHours(-8)

# SMB Server operational events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-SMBServer/Operational"
  StartTime = $StartTime
} -ErrorAction SilentlyContinue |
  Export-Clixml -Path (Join-Path $EventRoot "smbserver-operational-$Timestamp.xml")

# Security events commonly tied to file share access and share changes.
Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = 5140,5142,5143,5144,5145
  StartTime = $StartTime
} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated, Id, ProviderName, Message |
  Export-Csv -Path (Join-Path $EventRoot "security-file-share-events-$Timestamp.csv") -NoTypeInformation

# Show available SMB logs for operator review.
Get-WinEvent -ListLog "*SMB*" |
  Select-Object LogName, IsEnabled, RecordCount |
  Sort-Object LogName |
  Export-Csv -Path (Join-Path $EventRoot "smb-log-inventory-$Timestamp.csv") -NoTypeInformation

Write-Host "SMB event exports saved to $EventRoot"
```

Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Performance_Counter_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: capture SMB performance data for a short troubleshooting window.

$ReportRoot = "C:\SMB-Monitoring\Reports"
New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

# Inventory available SMB counter sets first.
Get-Counter -ListSet "*SMB*" |
  Select-Object CounterSetName, Description |
  Sort-Object CounterSetName |
  Export-Csv -Path (Join-Path $ReportRoot "smb-countersets-$Timestamp.csv") -NoTypeInformation

# Capture broad SMB Server Shares counters.
Get-Counter `
  -Counter "\SMB Server Shares(*)\*" `
  -SampleInterval 5 `
  -MaxSamples 12 |
  Export-Clixml -Path (Join-Path $ReportRoot "smb-server-shares-counters-$Timestamp.xml")

# Capture broad SMB Server Sessions counters if present.
Get-Counter `
  -Counter "\SMB Server Sessions(*)\*" `
  -SampleInterval 5 `
  -MaxSamples 12 `
  -ErrorAction SilentlyContinue |
  Export-Clixml -Path (Join-Path $ReportRoot "smb-server-sessions-counters-$Timestamp.xml")

Write-Host "SMB performance counter capture saved to $ReportRoot"
```


Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Active_Triage_Skeleton
```
# Run in elevated PowerShell on the file server.
# Purpose: locate a user, client, or locked file before deciding whether to close a file handle.

$TargetUser = "<username>"
$TargetClient = "<client-name>"
$TargetFileName = "<filename>"

# Find active sessions by user.
Get-SmbSession |
  Where-Object ClientUserName -like "*$TargetUser*" |
  Select-Object SessionId, ClientComputerName, ClientUserName, Dialect, NumOpens, Signed, Encrypted, SecondsExists |
  Format-Table -AutoSize

# Find active sessions by client.
Get-SmbSession |
  Where-Object ClientComputerName -like "*$TargetClient*" |
  Select-Object SessionId, ClientComputerName, ClientUserName, Dialect, NumOpens, Signed, Encrypted, SecondsExists |
  Format-Table -AutoSize

# Find open file handle by filename.
Get-SmbOpenFile |
  Where-Object Path -like "*$TargetFileName*" |
  Select-Object FileId, SessionId, ClientComputerName, ClientUserName, Path, Permissions, Locks |
  Format-Table -AutoSize

# Only use this after confirming impact and approval.
# Close-SmbOpenFile -FileId <file-id> -Force

# Only use this after confirming impact and approval.
# Close-SmbSession -SessionId <session-id> -Force
```

Monitor_SMB_Sessions_Open_Files_Events_And_Performance_Verification_Commands

|   |   |   |
|---|---|---|
|Task|Command|Expected Result|
|Verify SMB Server service|Get-Service LanmanServer|Service is running|
|Verify shares|Get-SmbShare|Expected shares are present|
|Verify SMB server settings|Get-SmbServerConfiguration|SMB posture is visible|
|Verify active sessions|Get-SmbSession|Active sessions are listed|
|Verify session security|`Get-SmbSession|Select ClientComputerName,ClientUserName,Dialect,Signed,Encrypted`|
|Verify open files|Get-SmbOpenFile|Active SMB file handles are listed|
|Verify open file locks|`Get-SmbOpenFile|Select Path,ClientUserName,Locks`|
|Verify client SMB connection|Get-SmbConnection|Client sees active server/share connection|
|Verify SMB logs exist|Get-WinEvent -ListLog "*SMB*"|SMB logs are discoverable|
|Verify SMBServer events|Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Operational" -MaxEvents 20|Recent events return|
|Verify File Share audit state|auditpol /get /subcategory:"File Share"|Audit state is visible|
|Verify Detailed File Share audit state|auditpol /get /subcategory:"Detailed File Share"|Audit state is visible|
|Verify share access events|Get-WinEvent -FilterHashtable @{LogName="Security"; Id=5140,5145; StartTime=(Get-Date).AddHours(-1)}|Recent share access events return if auditing is enabled|
|Verify SMB counters|Get-Counter -ListSet "*SMB*"|SMB counter sets return|
|Verify share performance capture|Get-Counter "\SMB Server Shares(*)\*" -SampleInterval 5 -MaxSamples 3|Counter samples return|