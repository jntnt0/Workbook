15_Backup_Export_Restore_DNS_Zones_And_Server_Config.md
# Backup_Export_Restore_DNS_Zones_And_Server_Config

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Index
15_Backup_Export_Restore_DNS_Zones_And_Server_Config.md
Backup_Export_Restore_DNS_Zones_And_Server_Config
Backup_Export_Restore_DNS_Zones_And_Server_Config_Source_Basis
Backup_Export_Restore_DNS_Zones_And_Server_Config_Mental_Model
Backup_Export_Restore_DNS_Zones_And_Server_Config_Planning_Table
Backup_Export_Restore_DNS_Zones_And_Server_Config_Configuration_Checklist
Backup_Export_Restore_DNS_Zones_And_Server_Config_Precheck_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Zone_Inventory_Export_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Zone_File_Export_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Server_Config_Export_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_AD_Integrated_DNS_Backup_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_File_Backed_Zone_Restore_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Record_Level_Restore_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Server_Config_Restore_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Post_Restore_Validation_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Event_Log_Skeleton
Backup_Export_Restore_DNS_Zones_And_Server_Config_Verification_Commands
Backup_Export_Restore_DNS_Zones_And_Server_Config_Rollback
Backup_Export_Restore_DNS_Zones_And_Server_Config_Failure_Checks
Backup_Export_Restore_DNS_Zones_And_Server_Config_Related_Labs

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | DNS zone, record, forwarder, cache, recursion, diagnostics, and server setting inventory |
| Microsoft Learn | Export-DnsServerZone | Exporting zone contents to a zone file |
| Microsoft Learn | Get-DnsServerZone | Listing hosted zones and zone properties |
| Microsoft Learn | Get-DnsServerResourceRecord | Exporting record-level inventory for validation or manual restore |
| Microsoft Learn | Add-DnsServerPrimaryZone | Recreating primary zones, including from existing zone files |
| Microsoft Learn | Add-DnsServerResourceRecordA | Restoring A records manually |
| Microsoft Learn | Add-DnsServerResourceRecordPtr | Restoring PTR records manually |
| Microsoft Learn | Add-DnsServerResourceRecordCName | Restoring CNAME records manually |
| Microsoft Learn | Add-DnsServerResourceRecordMX | Restoring MX records manually |
| Microsoft Learn | Add-DnsServerResourceRecord | Restoring generic record types where supported |
| Microsoft Learn | Get-DnsServerForwarder / Set-DnsServerForwarder | Exporting and restoring DNS forwarder configuration |
| Microsoft Learn | Get-DnsServerRecursion / Set-DnsServerRecursion | Exporting and restoring recursion behavior |
| Microsoft Learn | Get-DnsServerScavenging / Set-DnsServerScavenging | Exporting and restoring scavenging configuration |
| Microsoft Learn | Get-DnsServerDiagnostics / Set-DnsServerDiagnostics | Exporting and restoring diagnostic logging stance |
| Windows Server operational practice | Registry export, DNS folder backup, AD System State backup, zone inventory, post-restore validation | Recovering DNS zones and server behavior after accidental deletion, corruption, migration, or rebuild |

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS zone backup | Copy of DNS zone contents, usually as exported zone files and record inventories |
| DNS server config backup | Copy of server-level behavior such as forwarders, recursion, diagnostics, scavenging, root hints, and registry settings |
| File-backed zone | DNS zone stored as a `.dns` file under the DNS server folder |
| AD-integrated zone | DNS zone stored in Active Directory and replicated through AD, not just a local file |
| Zone export | Portable text representation of a zone useful for backup, review, migration, or recovery |
| Record inventory | CSV/TXT export of DNS records for audit and manual rebuild |
| Server config inventory | Export of DNS server settings that define resolver and operational behavior |
| Registry backup | Export of DNS service registry key for server-level recovery evidence |
| DNS folder backup | Copy of `%SystemRoot%\System32\DNS` containing file-backed zones and boot/config artifacts |
| System State backup | Correct backup method for AD-integrated DNS because DNS data lives in Active Directory |
| Authoritative restore risk | AD restore operation that can overwrite replicated data if used incorrectly |
| Non-authoritative restore | Restore where AD replication updates the restored DC from partners |
| Record-level restore | Recreate individual DNS records instead of restoring an entire zone |
| Full-zone restore | Recreate or reload an entire zone from a zone export/file |
| First rule | Back up zone data, server settings, DNS folder, registry, and AD System State separately because they protect different failure modes |

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server name | `DC1.corp.local` | `<dns-server-fqdn>` |
| DNS server IP | `10.10.10.10` | `<dns-server-ip>` |
| Alternate DNS server | `DC2.corp.local` | `<alternate-dns-server>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Forward zone | `corp.local` | `<forward-zone-name>` |
| Reverse zone | `10.10.10.in-addr.arpa` | `<reverse-zone-name>` |
| AD-integrated zones present | Yes | `<yes-no>` |
| File-backed zones present | No / Yes | `<yes-no>` |
| Secondary zones present | No / Yes | `<yes-no>` |
| Stub zones present | No / Yes | `<yes-no>` |
| Backup root path | `D:\DNS-Backups` | `<backup-root>` |
| Backup run path | `D:\DNS-Backups\2026-06-12-1400` | `<backup-run-path>` |
| DNS service folder | `C:\Windows\System32\DNS` | `<dns-folder-path>` |
| Registry export file | `dns-service-registry.reg` | `<registry-export-file>` |
| Zone export naming | `<zone-name>.dns` | `<zone-export-file>` |
| Record inventory format | CSV and TXT | `<record-inventory-format>` |
| Server config format | TXT, CSV, CLIXML, JSON where useful | `<server-config-format>` |
| System State backup location | `E:\SystemStateBackup` | `<system-state-backup-location>` |
| Restore scope | Record / zone / server config / AD-integrated restore | `<restore-scope>` |
| Change window required | Yes for restore | `<yes-no>` |
| Evidence path | `C:\DNS-Backup-Restore` | `<evidence-path>` |
| Rollback stance | Export current broken state before restore, then restore only the smallest necessary scope | `<rollback-plan>` |
| Next workbook | `16_Troubleshoot_DNS_Backup_Restore_And_Record_Recovery.md` | `<next-task>` |

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DNS Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DNS role installed | DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 3 | Confirm DNS service running | DNS Server | `Get-Service DNS` | DNS service is Running |
| 4 | Import DNS module | DNS Server | `Import-Module DnsServer` | DNS cmdlets are available |
| 5 | Create backup run folder | DNS Server | `New-Item -ItemType Directory -Force -Path "<backup-run-path>"` | Backup folder exists |
| 6 | Capture server identity | DNS Server | `hostname`; `Get-ComputerInfo` | Server identity is documented |
| 7 | Capture IP configuration | DNS Server | `Get-NetIPConfiguration`; `ipconfig /all` | Resolver and NIC config are documented |
| 8 | Export DNS role and service state | DNS Server | `Get-WindowsFeature DNS`; `Get-Service DNS` | Role and service state are saved |
| 9 | Export zone inventory | DNS Server | `Get-DnsServerZone` | All hosted zones are listed |
| 10 | Export zone properties | DNS Server | `Get-DnsServerZone \| Select *` | Zone type, AD integration, update mode, and replication scope are documented |
| 11 | Export all zone records to CSV | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | Record inventory is saved |
| 12 | Export zone files | DNS Server | `Export-DnsServerZone -Name "<zone-name>" -FileName "<zone-file>"` | Zone text file is created |
| 13 | Copy DNS service folder | DNS Server | `Copy-Item "$env:SystemRoot\System32\DNS" "<backup-run-path>\DNS-Folder" -Recurse` | DNS folder backup exists |
| 14 | Export DNS service registry key | DNS Server | `reg export HKLM\SYSTEM\CurrentControlSet\Services\DNS "<backup-run-path>\dns-service-registry.reg"` | Registry export exists |
| 15 | Export forwarders | DNS Server | `Get-DnsServerForwarder` | Forwarder config is saved |
| 16 | Export recursion settings | DNS Server | `Get-DnsServerRecursion` | Recursion config is saved |
| 17 | Export scavenging settings | DNS Server | `Get-DnsServerScavenging` | Scavenging config is saved |
| 18 | Export zone aging settings | DNS Server | `Get-DnsServerZoneAging -Name "<zone-name>"` | Aging config is saved |
| 19 | Export diagnostics settings | DNS Server | `Get-DnsServerDiagnostics` | Diagnostic config is saved |
| 20 | Export root hints | DNS Server | `Get-DnsServerRootHint` | Root hints are saved |
| 21 | Export DNS statistics | DNS Server | `Get-DnsServerStatistics` | DNS stats are saved |
| 22 | Export event logs | DNS Server | `Get-WinEvent -LogName "DNS Server"` | DNS event evidence is saved |
| 23 | Run AD DNS health checks if AD-integrated | Domain Controller | `dcdiag /test:dns /v`; `repadmin /replsummary` | AD DNS health and replication evidence are saved |
| 24 | Run System State backup if AD-integrated DNS | Domain Controller | `wbadmin start systemstatebackup -backuptarget:<target> -quiet` | System State backup is created |
| 25 | Validate backup files exist | DNS Server | `Get-ChildItem "<backup-run-path>" -Recurse` | Backup artifacts are visible |
| 26 | Test restore path in lab or isolated copy | Lab DNS Server | Use restore skeletons | Restore procedure is validated |
| 27 | Document backup result | Operator | `Record backup path, zones, record counts, config exports, event logs, and validation status` | Backup record is complete |

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: capture baseline state before backup or restore.

$BackupRoot = "D:\DNS-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"

New-Item -ItemType Directory -Force -Path $BackupRunPath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$BackupRunPath\whoami-groups.txt"

# Capture server identity.
hostname |
  Tee-Object "$BackupRunPath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,WindowsVersion |
  Tee-Object "$BackupRunPath\computer-info.txt"

# Capture DNS role and service.
Get-WindowsFeature DNS |
  Tee-Object "$BackupRunPath\dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$BackupRunPath\dns-service-state.txt"

# Capture network state.
Get-NetIPConfiguration |
  Tee-Object "$BackupRunPath\net-ip-configuration.txt"

Get-DnsClientServerAddress -AddressFamily IPv4 |
  Tee-Object "$BackupRunPath\dns-client-server-addresses.txt"

ipconfig /all |
  Tee-Object "$BackupRunPath\ipconfig-all.txt"

# Import DNS module.
Import-Module DnsServer

# Capture zones.
Get-DnsServerZone |
  Tee-Object "$BackupRunPath\dns-zones.txt"

Get-DnsServerZone |
  Export-Csv "$BackupRunPath\dns-zones.csv" -NoTypeInformation

# Capture critical zone properties.
Get-DnsServerZone -Name $ForwardZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$BackupRunPath\forward-zone-properties.txt"

Get-DnsServerZone -Name $ReverseZone -ErrorAction SilentlyContinue |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$BackupRunPath\reverse-zone-properties.txt"
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Zone_Inventory_Export_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: export DNS zone inventory and all records from every zone.

$BackupRoot = "D:\DNS-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$ZoneInventoryPath = Join-Path $BackupRunPath "Zone-Inventory"

New-Item -ItemType Directory -Force -Path $ZoneInventoryPath

Import-Module DnsServer

# Export zone list.
$Zones = Get-DnsServerZone

$Zones |
  Tee-Object "$ZoneInventoryPath\dns-zones.txt"

$Zones |
  Export-Csv "$ZoneInventoryPath\dns-zones.csv" -NoTypeInformation

# Export each zone's records.
foreach ($Zone in $Zones) {
  $SafeZoneName = $Zone.ZoneName -replace '[\\/:*?"<>|]', '_'
  $ZonePath = Join-Path $ZoneInventoryPath $SafeZoneName

  New-Item -ItemType Directory -Force -Path $ZonePath

  # Zone properties.
  $Zone |
    Select-Object * |
    Tee-Object "$ZonePath\zone-properties.txt"

  $Zone |
    Select-Object * |
    Export-Clixml "$ZonePath\zone-properties.xml"

  # Zone records.
  Get-DnsServerResourceRecord -ZoneName $Zone.ZoneName -ErrorAction SilentlyContinue |
    Tee-Object "$ZonePath\records.txt"

  Get-DnsServerResourceRecord -ZoneName $Zone.ZoneName -ErrorAction SilentlyContinue |
    Export-Csv "$ZonePath\records.csv" -NoTypeInformation

  # Common record-type snapshots.
  foreach ($Type in @("SOA","NS","A","AAAA","CNAME","MX","SRV","PTR","TXT")) {
    Get-DnsServerResourceRecord -ZoneName $Zone.ZoneName -RRType $Type -ErrorAction SilentlyContinue |
      Export-Csv "$ZonePath\records-$Type.csv" -NoTypeInformation
  }

  # Record count.
  $RecordCount = (Get-DnsServerResourceRecord -ZoneName $Zone.ZoneName -ErrorAction SilentlyContinue).Count
  $RecordCount |
    Tee-Object "$ZonePath\record-count.txt"
}
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Zone_File_Export_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: export zones to zone files where supported.
# Exported files are created under the DNS server folder unless copied out after export.

$BackupRoot = "D:\DNS-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$ZoneFileExportPath = Join-Path $BackupRunPath "Zone-Files"
$DnsFolder = "$env:SystemRoot\System32\DNS"

New-Item -ItemType Directory -Force -Path $ZoneFileExportPath

Import-Module DnsServer

$Zones = Get-DnsServerZone

foreach ($Zone in $Zones) {
  $ZoneName = $Zone.ZoneName
  $SafeZoneName = $ZoneName -replace '[\\/:*?"<>|]', '_'
  $ExportFileName = "$SafeZoneName.dns"

  # Skip cache and special zones if export fails.
  try {
    Export-DnsServerZone `
      -Name $ZoneName `
      -FileName $ExportFileName `
      -ErrorAction Stop

    # Copy exported zone file from DNS folder into backup run path.
    $ExportedFilePath = Join-Path $DnsFolder $ExportFileName

    if (Test-Path $ExportedFilePath) {
      Copy-Item `
        -Path $ExportedFilePath `
        -Destination (Join-Path $ZoneFileExportPath $ExportFileName) `
        -Force
    }

    "Exported $ZoneName to $ExportFileName" |
      Tee-Object "$ZoneFileExportPath\zone-export-success.log" -Append
  }
  catch {
    "FAILED to export $ZoneName : $($_.Exception.Message)" |
      Tee-Object "$ZoneFileExportPath\zone-export-failures.log" -Append
  }
}

# Copy the complete DNS folder as supporting evidence.
Copy-Item `
  -Path $DnsFolder `
  -Destination (Join-Path $BackupRunPath "DNS-Folder-Copy") `
  -Recurse `
  -Force
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Server_Config_Export_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: export DNS server-level configuration and supporting registry/service state.

$BackupRoot = "D:\DNS-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$ConfigPath = Join-Path $BackupRunPath "Server-Config"

New-Item -ItemType Directory -Force -Path $ConfigPath

Import-Module DnsServer

# DNS server forwarders.
Get-DnsServerForwarder |
  Tee-Object "$ConfigPath\dns-forwarders.txt"

Get-DnsServerForwarder |
  Export-Clixml "$ConfigPath\dns-forwarders.xml"

# DNS recursion settings.
Get-DnsServerRecursion |
  Tee-Object "$ConfigPath\dns-recursion.txt"

Get-DnsServerRecursion |
  Export-Clixml "$ConfigPath\dns-recursion.xml"

# DNS scavenging settings.
Get-DnsServerScavenging |
  Tee-Object "$ConfigPath\dns-scavenging.txt"

Get-DnsServerScavenging |
  Export-Clixml "$ConfigPath\dns-scavenging.xml"

# DNS diagnostic settings.
Get-DnsServerDiagnostics |
  Tee-Object "$ConfigPath\dns-diagnostics.txt"

Get-DnsServerDiagnostics |
  Export-Clixml "$ConfigPath\dns-diagnostics.xml"

# DNS root hints.
Get-DnsServerRootHint |
  Tee-Object "$ConfigPath\dns-root-hints.txt"

Get-DnsServerRootHint |
  Export-Csv "$ConfigPath\dns-root-hints.csv" -NoTypeInformation

# DNS server cache and statistics.
Get-DnsServerCache |
  Export-Csv "$ConfigPath\dns-cache.csv" -NoTypeInformation

Get-DnsServerStatistics |
  Tee-Object "$ConfigPath\dns-statistics.txt"

Get-DnsServerStatistics |
  Export-Clixml "$ConfigPath\dns-statistics.xml"

# Zone aging settings.
$Zones = Get-DnsServerZone

foreach ($Zone in $Zones) {
  $SafeZoneName = $Zone.ZoneName -replace '[\\/:*?"<>|]', '_'

  Get-DnsServerZoneAging -Name $Zone.ZoneName -ErrorAction SilentlyContinue |
    Tee-Object "$ConfigPath\zone-aging-$SafeZoneName.txt"

  Get-DnsServerZoneAging -Name $Zone.ZoneName -ErrorAction SilentlyContinue |
    Export-Clixml "$ConfigPath\zone-aging-$SafeZoneName.xml"
}

# Export DNS service registry key.
reg export "HKLM\SYSTEM\CurrentControlSet\Services\DNS" "$ConfigPath\dns-service-registry.reg" /y

# Export DNS service folder file listing.
Get-ChildItem "$env:SystemRoot\System32\DNS" -Recurse |
  Select-Object FullName,Length,LastWriteTime |
  Export-Csv "$ConfigPath\dns-folder-file-list.csv" -NoTypeInformation

# Copy DNS folder.
Copy-Item `
  -Path "$env:SystemRoot\System32\DNS" `
  -Destination "$ConfigPath\DNS-Folder-Copy" `
  -Recurse `
  -Force

# Export Windows event logs relevant to DNS.
Get-WinEvent -LogName "DNS Server" -MaxEvents 500 -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dns-server-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 500 -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dns-server-operational-events.txt"
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_AD_Integrated_DNS_Backup_Skeleton
```powershell
# Run in elevated PowerShell on a domain controller hosting AD-integrated DNS.
# Purpose: capture AD-integrated DNS evidence and trigger System State backup.
# AD-integrated DNS is stored in Active Directory, so System State backup is the recovery anchor.

$BackupTarget = "E:"
$BackupRoot = "D:\DNS-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$AdDnsPath = Join-Path $BackupRunPath "AD-Integrated-DNS"

New-Item -ItemType Directory -Force -Path $AdDnsPath

Import-Module DnsServer
Import-Module ActiveDirectory

# Confirm DC and AD state.
Get-ADDomain |
  Tee-Object "$AdDnsPath\ad-domain.txt"

Get-ADForest |
  Tee-Object "$AdDnsPath\ad-forest.txt"

Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog,OperationMasterRoles |
  Tee-Object "$AdDnsPath\domain-controllers.txt"

# Capture AD-integrated zones.
Get-DnsServerZone |
  Where-Object {$_.IsDsIntegrated -eq $true} |
  Tee-Object "$AdDnsPath\ad-integrated-zones.txt"

Get-DnsServerZone |
  Where-Object {$_.IsDsIntegrated -eq $true} |
  Export-Csv "$AdDnsPath\ad-integrated-zones.csv" -NoTypeInformation

# Export records from AD-integrated zones.
$AdZones = Get-DnsServerZone | Where-Object {$_.IsDsIntegrated -eq $true}

foreach ($Zone in $AdZones) {
  $SafeZoneName = $Zone.ZoneName -replace '[\\/:*?"<>|]', '_'
  $ZonePath = Join-Path $AdDnsPath $SafeZoneName

  New-Item -ItemType Directory -Force -Path $ZonePath

  Get-DnsServerResourceRecord -ZoneName $Zone.ZoneName -ErrorAction SilentlyContinue |
    Tee-Object "$ZonePath\records.txt"

  Get-DnsServerResourceRecord -ZoneName $Zone.ZoneName -ErrorAction SilentlyContinue |
    Export-Csv "$ZonePath\records.csv" -NoTypeInformation

  Export-DnsServerZone -Name $Zone.ZoneName -FileName "$SafeZoneName.dns" -ErrorAction SilentlyContinue
}

# Validate AD replication health before backup.
repadmin /replsummary |
  Tee-Object "$AdDnsPath\repadmin-replsummary-before-backup.txt"

dcdiag /test:dns /v |
  Tee-Object "$AdDnsPath\dcdiag-dns-before-backup.txt"

# Start System State backup.
# Backup target must be a local volume or supported backup target.
wbadmin start systemstatebackup `
  -backuptarget:$BackupTarget `
  -quiet |
  Tee-Object "$AdDnsPath\wbadmin-systemstatebackup-output.txt"

# List backups after completion.
wbadmin get versions |
  Tee-Object "$AdDnsPath\wbadmin-versions-after-backup.txt"
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_File_Backed_Zone_Restore_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server during a restore window.
# Purpose: restore a file-backed primary zone from an exported .dns zone file.
# Use in lab first. Export current state before restore.

$ZoneName = "corp.local"
$ZoneFileName = "corp.local.dns"
$RestoreSourceFile = "D:\DNS-Backups\Latest\Zone-Files\corp.local.dns"
$DnsFolder = "$env:SystemRoot\System32\DNS"
$EvidencePath = "C:\DNS-Restore"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\file-backed-zone-restore-transcript.txt"

Import-Module DnsServer

# Capture current zone state before restore.
Get-DnsServerZone -Name $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\zone-before-restore.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\records-before-restore.csv" -NoTypeInformation

# Copy restore zone file into DNS folder.
Copy-Item `
  -Path $RestoreSourceFile `
  -Destination (Join-Path $DnsFolder $ZoneFileName) `
  -Force

# If zone already exists and needs full rebuild, remove only after backup and approval.
# Remove-DnsServerZone -Name $ZoneName -Force

# Recreate primary zone from existing zone file.
# Use LoadExisting when restoring from an existing DNS zone file.
Add-DnsServerPrimaryZone `
  -Name $ZoneName `
  -ZoneFile $ZoneFileName `
  -LoadExisting

# Confirm restored zone.
Get-DnsServerZone -Name $ZoneName |
  Tee-Object "$EvidencePath\zone-after-restore.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\records-after-restore.txt"

Stop-Transcript
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Record_Level_Restore_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server during a restore window.
# Purpose: restore selected records manually.
# Use when only a few records were deleted or corrupted.

$ZoneName = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$EvidencePath = "C:\DNS-Restore"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\record-level-restore-transcript.txt"

Import-Module DnsServer

# Capture current records before manual restore.
Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\records-before-manual-restore.csv" -NoTypeInformation

# Restore A record example.
Add-DnsServerResourceRecordA `
  -ZoneName $ZoneName `
  -Name "app1" `
  -IPv4Address "10.10.10.50" `
  -TimeToLive 01:00:00

# Restore CNAME record example.
Add-DnsServerResourceRecordCName `
  -ZoneName $ZoneName `
  -Name "portal" `
  -HostNameAlias "app1.corp.local" `
  -TimeToLive 01:00:00

# Restore MX record example.
Add-DnsServerResourceRecordMX `
  -ZoneName $ZoneName `
  -Name "." `
  -MailExchange "mail.corp.local" `
  -Preference 10 `
  -TimeToLive 01:00:00

# Restore PTR record example.
# For 10.10.10.50 in 10.10.10.in-addr.arpa, PTR host label is 50.
Add-DnsServerResourceRecordPtr `
  -ZoneName $ReverseZone `
  -Name "50" `
  -PtrDomainName "app1.corp.local" `
  -TimeToLive 01:00:00

# Restore TXT record example using generic record cmdlet if needed.
Add-DnsServerResourceRecord `
  -ZoneName $ZoneName `
  -Name "txt-test" `
  -Txt `
  -DescriptiveText "restored record validation" `
  -TimeToLive 01:00:00

# Validate restored records.
Resolve-DnsName "app1.$ZoneName" -Server "127.0.0.1" |
  Tee-Object "$EvidencePath\resolve-restored-a-record.txt"

Resolve-DnsName "portal.$ZoneName" -Type CNAME -Server "127.0.0.1" |
  Tee-Object "$EvidencePath\resolve-restored-cname-record.txt"

Resolve-DnsName "10.10.10.50" -Type PTR -Server "127.0.0.1" |
  Tee-Object "$EvidencePath\resolve-restored-ptr-record.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Export-Csv "$EvidencePath\records-after-manual-restore.csv" -NoTypeInformation

Stop-Transcript
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Server_Config_Restore_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server during a restore window.
# Purpose: restore selected DNS server-level settings from documented backup values.
# Do not blindly import registry config onto a different OS/server without lab validation.

$EvidencePath = "C:\DNS-Restore"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\server-config-restore-transcript.txt"

Import-Module DnsServer

# Capture current config before restore.
Get-DnsServerForwarder |
  Tee-Object "$EvidencePath\forwarders-before-restore.txt"

Get-DnsServerRecursion |
  Tee-Object "$EvidencePath\recursion-before-restore.txt"

Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\scavenging-before-restore.txt"

Get-DnsServerDiagnostics |
  Tee-Object "$EvidencePath\diagnostics-before-restore.txt"

# Restore forwarders example.
Set-DnsServerForwarder `
  -IPAddress "10.10.10.1","10.10.10.2" `
  -UseRootHint $true

# Restore recursion example.
Set-DnsServerRecursion `
  -Enable $true

# Restore scavenging example.
Set-DnsServerScavenging `
  -ScavengingState $true `
  -ScavengingInterval 7.00:00:00

# Restore diagnostics to quiet/default troubleshooting stance.
Set-DnsServerDiagnostics `
  -Queries $false `
  -Answers $false `
  -Notifications $false `
  -Update $false `
  -QuestionTransactions $false `
  -UnmatchedResponse $false `
  -SendPackets $false `
  -ReceivePackets $false `
  -TcpPackets $false `
  -UdpPackets $false

# Optional registry restore from same-server backup only after approval.
# reg import "D:\DNS-Backups\Latest\Server-Config\dns-service-registry.reg"
# Restart-Service DNS

# Confirm restored settings.
Get-DnsServerForwarder |
  Tee-Object "$EvidencePath\forwarders-after-restore.txt"

Get-DnsServerRecursion |
  Tee-Object "$EvidencePath\recursion-after-restore.txt"

Get-DnsServerScavenging |
  Tee-Object "$EvidencePath\scavenging-after-restore.txt"

Get-DnsServerDiagnostics |
  Tee-Object "$EvidencePath\diagnostics-after-restore.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-after-restore.txt"

Stop-Transcript
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Post_Restore_Validation_Skeleton
```powershell
# Run in elevated PowerShell on DNS server and from at least one client/admin host.
# Purpose: validate DNS service, zones, records, AD locator, recursion, and event logs after restore.

$DomainFqdn = "corp.local"
$ForwardZone = "corp.local"
$ReverseZone = "10.10.10.in-addr.arpa"
$DnsServerIp = "10.10.10.10"
$InternalRecord = "dc1.corp.local"
$SrvRecord = "_ldap._tcp.dc._msdcs.corp.local"
$PtrTestIp = "10.10.10.10"
$ExternalRecord = "www.microsoft.com"
$EvidencePath = "C:\DNS-Restore"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Confirm DNS service.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-post-restore.txt"

# Confirm zones.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-post-restore.txt"

# Confirm records.
Get-DnsServerResourceRecord -ZoneName $ForwardZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\forward-zone-records-post-restore.txt"

Get-DnsServerResourceRecord -ZoneName $ReverseZone -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\reverse-zone-records-post-restore.txt"

# Internal lookup.
Resolve-DnsName $InternalRecord -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-internal-record-post-restore.txt"

# Domain root lookup.
Resolve-DnsName $DomainFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-domain-root-post-restore.txt"

# AD SRV lookup.
Resolve-DnsName $SrvRecord -Type SRV -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-srv-post-restore.txt"

# PTR lookup.
Resolve-DnsName $PtrTestIp -Type PTR -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-ptr-post-restore.txt"

# External lookup if recursion/forwarding is expected.
Resolve-DnsName $ExternalRecord -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-external-post-restore.txt"

# AD DNS diagnostic if on DC.
dcdiag /test:dns /v |
  Tee-Object "$EvidencePath\dcdiag-dns-post-restore.txt"

# AD replication if AD-integrated DNS.
repadmin /replsummary |
  Tee-Object "$EvidencePath\repadmin-replsummary-post-restore.txt"

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-post-restore.txt"

Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events-post-restore.txt"
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: collect DNS, System, Directory Service, and backup/restore related events.

$EvidencePath = "C:\DNS-Backup-Restore-Events"
$Since = (Get-Date).AddHours(-24)

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server events.
Get-WinEvent -FilterHashtable @{
  LogName = "DNS Server"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-last-24h.txt"

# DNS Server operational events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-DNSServer/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events-last-24h.txt"

# System DNS and service events.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*DNS*" -or
    $_.Message -like "*zone*" -or
    $_.Message -like "*backup*" -or
    $_.Message -like "*restore*"
  } |
  Tee-Object "$EvidencePath\system-dns-backup-restore-events-last-24h.txt"

# Directory Service events for AD-integrated DNS.
Get-WinEvent -FilterHashtable @{
  LogName = "Directory Service"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\directory-service-events-last-24h.txt"

# Windows Backup events if System State backup is used.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Backup"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\windows-backup-events-last-24h.txt"

# Final DNS service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-final.txt"
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Verification_Commands
```powershell
# Confirm DNS role and service.
Get-WindowsFeature DNS
Get-Service DNS

# Import DNS module.
Import-Module DnsServer

# Zone inventory.
Get-DnsServerZone
Get-DnsServerZone -Name "<zone-name>"
Get-DnsServerZone -Name "<zone-name>" | Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Record inventory.
Get-DnsServerResourceRecord -ZoneName "<zone-name>"
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType SOA
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NS
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType A
Get-DnsServerResourceRecord -ZoneName "<reverse-zone-name>" -RRType PTR

# Export records.
Get-DnsServerResourceRecord -ZoneName "<zone-name>" |
  Export-Csv "<backup-path>\<zone-name>-records.csv" -NoTypeInformation

# Export zone file.
Export-DnsServerZone -Name "<zone-name>" -FileName "<zone-name>.dns"

# Copy DNS folder.
Copy-Item "$env:SystemRoot\System32\DNS" "<backup-path>\DNS-Folder" -Recurse -Force

# Export DNS registry key.
reg export "HKLM\SYSTEM\CurrentControlSet\Services\DNS" "<backup-path>\dns-service-registry.reg" /y

# Export server settings.
Get-DnsServerForwarder
Get-DnsServerRecursion
Get-DnsServerScavenging
Get-DnsServerDiagnostics
Get-DnsServerRootHint
Get-DnsServerStatistics
Get-DnsServerZoneAging -Name "<zone-name>"

# Restore file-backed primary zone from existing file.
Copy-Item "<restore-source>\<zone-name>.dns" "$env:SystemRoot\System32\DNS\<zone-name>.dns" -Force

Add-DnsServerPrimaryZone `
  -Name "<zone-name>" `
  -ZoneFile "<zone-name>.dns" `
  -LoadExisting

# Restore common records manually.
Add-DnsServerResourceRecordA `
  -ZoneName "<zone-name>" `
  -Name "<host-label>" `
  -IPv4Address "<ip-address>"

Add-DnsServerResourceRecordCName `
  -ZoneName "<zone-name>" `
  -Name "<alias-label>" `
  -HostNameAlias "<target-fqdn>"

Add-DnsServerResourceRecordPtr `
  -ZoneName "<reverse-zone-name>" `
  -Name "<ptr-host-label>" `
  -PtrDomainName "<target-fqdn>"

Add-DnsServerResourceRecordMX `
  -ZoneName "<zone-name>" `
  -Name "." `
  -MailExchange "<mail-server-fqdn>" `
  -Preference 10

# Restore forwarders.
Set-DnsServerForwarder -IPAddress "<forwarder-ip-1>","<forwarder-ip-2>" -UseRootHint $true

# Restore recursion.
Set-DnsServerRecursion -Enable $true

# Restore scavenging.
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00

# System State backup for AD-integrated DNS.
wbadmin start systemstatebackup -backuptarget:<backup-target-drive> -quiet
wbadmin get versions

# DNS resolution validation.
Resolve-DnsName "<internal-test-record>" -Server "<dns-server-ip>"
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"
Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV -Server "<dns-server-ip>"
Resolve-DnsName "<ptr-test-ip>" -Type PTR -Server "<dns-server-ip>"
Resolve-DnsName "<external-test-record>" -Server "<dns-server-ip>"

# AD-integrated DNS validation.
dcdiag /test:dns /v
repadmin /replsummary
repadmin /showrepl

# Event logs.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Backup" -MaxEvents 100 -ErrorAction SilentlyContinue
```

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Zone export only | No rollback required | Export operation should not alter zone data |
| Record inventory export only | No rollback required | Inventory only |
| DNS folder copied | No rollback required | Backup copy only |
| Registry export only | No rollback required | Export only |
| File-backed zone restored incorrectly | Remove restored zone, restore prior zone file or recreate from pre-restore export | Zone outage until restored correctly |
| Wrong record restored | Remove incorrect record with `Remove-DnsServerResourceRecord` | Removing wrong record can break resolution |
| Missing record after restore | Recreate record from inventory/export | Name resolution remains broken until restored |
| Forwarders restored incorrectly | Reapply previous forwarder list from backup evidence | External resolution may fail |
| Recursion setting restored incorrectly | Reapply previous `Set-DnsServerRecursion` state | Resolver behavior may be too open or too restricted |
| Scavenging restored incorrectly | Reapply previous `Set-DnsServerScavenging` state | Stale records may remain or valid records may be removed |
| Diagnostics restored incorrectly | Disable noisy diagnostics with `Set-DnsServerDiagnostics` flags set to `$false` | Excessive logging overhead |
| Registry imported incorrectly | Restore from known-good backup or rebuild DNS role in lab-tested procedure | Registry import can destabilize DNS service |
| AD-integrated DNS restore issue | Use AD/System State recovery workflow and replication validation | Incorrect AD restore can replicate bad data |
| Evidence folder created | `Remove-Item C:\DNS-Backup-Restore -Recurse -Force` | Deletes restore evidence |

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Export-DnsServerZone fails | Zone type, permissions, path, or special zone issue | Zone type and error output | Reinstalling DNS |
| Export file not found | Export created under DNS folder, not backup path | Check `%SystemRoot%\System32\DNS` | Re-running random exports |
| CSV record inventory cannot be directly imported | CSV is audit format, not full restore format | Use manual record restore or zone file | Assuming CSV is a zone backup |
| AD-integrated zone missing after DC restore | AD/System State/replication issue | `dcdiag /test:dns`; `repadmin /replsummary` | Creating duplicate zones immediately |
| Restored file-backed zone does not load | Bad zone file path or syntax | DNS Server event log and zone file location | Editing records blindly |
| Secondary zone restored as primary | Wrong restore method | Zone type and source design | Leaving wrong authoritative copy |
| Stub zone restored as full primary | Wrong restore method | `Get-DnsServerZone` ZoneType | Copying all records into stub |
| Forwarders missing after rebuild | Server config not restored | `Get-DnsServerForwarder` and backup config | Troubleshooting internal zones |
| External resolution fails after restore | Forwarder/recursion/root hints issue | Forwarders and recursion settings | Recreating AD zones |
| Internal resolution fails after restore | Zone or record restore issue | Zone inventory and record count | Changing forwarders |
| SRV records missing | AD DNS/Netlogon registration issue | `dcdiag /test:dns`; `nltest /dsregdns` | Manually creating every SRV record |
| PTR records missing | Reverse zone not restored or records missing | Reverse zone and PTR inventory | Editing forward zone |
| DNS service fails to start | Registry/config/zone file issue | DNS Server and System event logs | Importing more registry files |
| Restored records have wrong TTL | Manual restore omitted original TTL | Record inventory and restored record output | Ignoring TTL drift |
| Restored records have wrong timestamp | Manual/static restore changed dynamic behavior | Record timestamp and aging/scavenging settings | Enabling scavenging immediately |
| Clients still resolve old records | Client or server cache | `Clear-DnsClientCache`; `Clear-DnsServerCache` | Re-restoring zone immediately |
| One DC has restored data but another does not | AD replication issue | `repadmin /showrepl`; query each DNS server directly | Manually editing each DC |
| Backup too old | RPO issue | Backup timestamp and zone serial/record inventory | Restoring without impact review |
| Restore broke delegation/forwarders | Missing non-zone server config | Delegation records, conditional forwarders, server settings | Blaming zone file only |

# Backup_Export_Restore_DNS_Zones_And_Server_Config_Related_Labs
| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Configure AD-integrated DNS zones | `07_Configure_AD_Integrated_DNS_Zones.md` | AD DNS zone baseline before backup |
| Validate AD DNS SRV records and DC locator | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md` | Critical AD DNS validation before and after restore |
| Configure DNS dynamic updates | `09_Configure_DNS_Dynamic_Updates.md` | Dynamic record behavior and ownership awareness |
| Configure DNS aging and scavenging | `10_Configure_DNS_Aging_And_Scavenging.md` | Timestamp and stale record cleanup considerations |
| Configure DNS zone transfers, secondary, and stub zones | `11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md` | Secondary/stub recovery and zone copy awareness |
| Configure DNS delegation | `12_Configure_DNS_Delegation.md` | Delegation records that must be preserved during restore |
| Configure DNS client settings and name resolution | `13_Configure_DNS_Client_Settings_And_Name_Resolution.md` | Client validation after DNS restore |
| Monitor DNS events, logs, statistics, and cache | `14_Monitor_DNS_Events_Logs_Statistics_And_Cache.md` | Evidence collection before and after restore |
| Backup, export, restore DNS zones and server config | `15_Backup_Export_Restore_DNS_Zones_And_Server_Config.md` | DNS backup, restore, and post-restore validation |
| Troubleshoot DNS backup, restore, and record recovery | `16_Troubleshoot_DNS_Backup_Restore_And_Record_Recovery.md` | Failed restore and missing-record root-cause workflow |