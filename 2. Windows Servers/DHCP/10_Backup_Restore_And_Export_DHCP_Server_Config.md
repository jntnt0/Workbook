10_Backup_Restore_And_Export_DHCP_Server_Config.md
# Backup_Restore_And_Export_DHCP_Server_Config

# Backup_Restore_And_Export_DHCP_Server_Config_Index
10_Backup_Restore_And_Export_DHCP_Server_Config.md
Backup_Restore_And_Export_DHCP_Server_Config
Backup_Restore_And_Export_DHCP_Server_Config_Source_Basis
Backup_Restore_And_Export_DHCP_Server_Config_Mental_Model
Backup_Restore_And_Export_DHCP_Server_Config_Planning_Table
Backup_Restore_And_Export_DHCP_Server_Config_Configuration_Checklist
Backup_Restore_And_Export_DHCP_Server_Config_Precheck_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Inventory_Export_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_XML_Export_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Database_Backup_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Server_Config_Backup_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Failover_And_DNS_Credential_Backup_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Import_To_New_Server_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Database_Restore_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Post_Restore_Validation_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Event_Log_Skeleton
Backup_Restore_And_Export_DHCP_Server_Config_Verification_Commands
Backup_Restore_And_Export_DHCP_Server_Config_Rollback
Backup_Restore_And_Export_DHCP_Server_Config_Failure_Checks
Backup_Restore_And_Export_DHCP_Server_Config_Related_Labs

# Backup_Restore_And_Export_DHCP_Server_Config_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DHCP Server PowerShell module | DHCP server backup, export, import, restore, lease, scope, and configuration management |
| Microsoft Learn | Export-DhcpServer | Exporting DHCP server configuration and leases to XML |
| Microsoft Learn | Import-DhcpServer | Importing DHCP server configuration and leases from XML |
| Microsoft Learn | Backup-DhcpServer | Backing up the DHCP server database |
| Microsoft Learn | Restore-DhcpServer | Restoring the DHCP server database |
| Microsoft Learn | Get-DhcpServerv4Scope | Exporting DHCPv4 scope inventory |
| Microsoft Learn | Get-DhcpServerv4Lease | Exporting active lease inventory |
| Microsoft Learn | Get-DhcpServerv4Reservation | Exporting reservation inventory |
| Microsoft Learn | Get-DhcpServerv4ExclusionRange | Exporting exclusion inventory |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Exporting server-level and scope-level options |
| Microsoft Learn | Get-DhcpServerv4Failover | Exporting DHCPv4 failover relationship evidence |
| Microsoft Learn | Get-DhcpServerDnsCredential | Reviewing DHCP DNS update credential state |
| Microsoft Learn | Get-DhcpServerInDC | Validating DHCP authorization in Active Directory |
| Windows Server operational practice | XML export, database backup, registry backup, event evidence, import validation | Recovering DHCP service after migration, corruption, rebuild, accidental deletion, or server failure |

# Backup_Restore_And_Export_DHCP_Server_Config_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP export | Portable XML copy of DHCP scopes, options, reservations, policies, and optionally leases |
| DHCP backup | Backup of DHCP database and related service data from the local DHCP server |
| DHCP restore | Restoring DHCP database from a DHCP backup path |
| DHCP import | Loading exported DHCP configuration into a DHCP server, often for migration or rebuild |
| Lease export | Captures active client leases so clients do not all start over after migration |
| Scope inventory | Human-readable evidence of scopes, ranges, state, options, exclusions, and reservations |
| Server-level options | DHCP options inherited by scopes unless overridden |
| Scope-level options | DHCP options applied only to a specific scope |
| DHCP authorization | AD permission allowing a domain DHCP server to lease addresses |
| DHCP DNS credential | Credential DHCP uses for secure DNS dynamic updates; should be captured and reapplied manually |
| Failover relationship | DHCP HA relationship that must be documented and validated during backup/restore |
| Registry backup | Supporting copy of DHCP service configuration for evidence and same-server recovery scenarios |
| Database backup path | Folder containing DHCP backup data created by Backup-DhcpServer |
| Migration restore | Importing DHCP configuration into a different server, then authorizing and validating |
| Same-server restore | Restoring database/config after corruption or accidental change on the same server |
| First rule | Always export XML, backup database, export readable inventories, and document authorization/failover/DNS credentials before touching restore |

# Backup_Restore_And_Export_DHCP_Server_Config_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Source DHCP server | `DHCP1.corp.local` | `<source-dhcp-server>` |
| Source DHCP server IP | `10.10.10.5` | `<source-dhcp-ip>` |
| Target DHCP server | `DHCP2.corp.local` | `<target-dhcp-server>` |
| Target DHCP server IP | `10.10.10.6` | `<target-dhcp-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Backup root path | `D:\DHCP-Backups` | `<backup-root>` |
| Backup run path | `D:\DHCP-Backups\20260612-1400` | `<backup-run-path>` |
| DHCP XML export file | `dhcp-export.xml` | `<dhcp-export-file>` |
| DHCP database backup path | `D:\DHCP-Backups\20260612-1400\DatabaseBackup` | `<dhcp-database-backup-path>` |
| Inventory export path | `D:\DHCP-Backups\20260612-1400\Inventory` | `<inventory-path>` |
| Registry export file | `dhcp-service-registry.reg` | `<registry-export-file>` |
| DHCP database path | `C:\Windows\System32\dhcp` | `<dhcp-database-path>` |
| Export leases | Yes | `<yes-no>` |
| Restore scope | Same server / new server / lab test | `<restore-scope>` |
| Import with leases | Yes / No | `<yes-no>` |
| Scope overwrite allowed | Lab only / approved restore only | `<yes-no>` |
| DHCP authorization required | Yes | `<yes-no>` |
| DHCP DNS credential account | `corp\svc-dhcpdns` | `<dhcp-dns-credential-account>` |
| DHCP failover relationship | `DHCP1-DHCP2-LB` | `<failover-name>` |
| Failover partner | `DHCP2.corp.local` | `<failover-partner>` |
| Change window required | Yes for restore/import | `<yes-no>` |
| Evidence path | `C:\DHCP-Backup-Restore` | `<evidence-path>` |
| Rollback stance | Export current target state before import or restore | `<rollback-plan>` |
| Next workbook | `11_Monitor_DHCP_Leases_Events_And_Scope_Utilization.md` | `<next-task>` |

# Backup_Restore_And_Export_DHCP_Server_Config_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DHCP role installed | DHCP Server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 3 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | DHCP service is Running |
| 4 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 5 | Confirm DHCP authorization | DHCP Server | `Get-DhcpServerInDC` | DHCP authorization state is documented |
| 6 | Create backup run folder | DHCP Server | `New-Item -ItemType Directory -Force -Path "<backup-run-path>"` | Backup folder exists |
| 7 | Capture server identity | DHCP Server | `hostname`; `Get-ComputerInfo` | Server identity is documented |
| 8 | Capture network state | DHCP Server | `Get-NetIPConfiguration`; `ipconfig /all` | IP and DNS config are documented |
| 9 | Export all scopes | DHCP Server | `Get-DhcpServerv4Scope` | Scope inventory is saved |
| 10 | Export all scope statistics | DHCP Server | `Get-DhcpServerv4ScopeStatistics` | Utilization evidence is saved |
| 11 | Export all leases | DHCP Server | `Get-DhcpServerv4Lease -ScopeId "<scope-id>"` | Lease inventory is saved |
| 12 | Export all reservations | DHCP Server | `Get-DhcpServerv4Reservation -ScopeId "<scope-id>"` | Reservation inventory is saved |
| 13 | Export all exclusions | DHCP Server | `Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"` | Exclusion inventory is saved |
| 14 | Export server-level options | DHCP Server | `Get-DhcpServerv4OptionValue` | Server option inventory is saved |
| 15 | Export scope-level options | DHCP Server | `Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"` | Scope option inventory is saved |
| 16 | Export DHCP DNS settings | DHCP Server | `Get-DhcpServerv4DnsSetting`; `Get-DhcpServerDnsCredential` | Dynamic DNS behavior is documented |
| 17 | Export DHCP policies/classes if used | DHCP Server | `Get-DhcpServerv4Policy`; class/filter cmdlets | Policy inventory is saved |
| 18 | Export failover state if used | DHCP Server | `Get-DhcpServerv4Failover`; `Get-DhcpServerv4FailoverScope` | Failover relationship evidence is saved |
| 19 | Create XML export with leases | DHCP Server | `Export-DhcpServer -File "<path>\dhcp-export.xml" -Leases -Force` | Portable DHCP export exists |
| 20 | Create DHCP database backup | DHCP Server | `Backup-DhcpServer -Path "<database-backup-path>"` | DHCP database backup exists |
| 21 | Copy DHCP database folder listing | DHCP Server | `Get-ChildItem "$env:SystemRoot\System32\dhcp"` | DHCP file evidence is saved |
| 22 | Export DHCP service registry key | DHCP Server | `reg export HKLM\SYSTEM\CurrentControlSet\Services\DHCPServer "<path>\dhcp-service-registry.reg"` | Registry evidence exists |
| 23 | Export DHCP event logs | DHCP Server | `Get-WinEvent` commands | Event evidence is saved |
| 24 | Validate backup files exist | DHCP Server | `Get-ChildItem "<backup-run-path>" -Recurse` | Export/backup artifacts are present |
| 25 | Test import in lab if possible | Lab DHCP Server | `Import-DhcpServer` | Import path is validated |
| 26 | Restore/import only in approved window | Target DHCP Server | Use restore/import skeleton | DHCP configuration is restored |
| 27 | Authorize target DHCP server if needed | Target DHCP Server | `Add-DhcpServerInDC` | Target can lease in AD domain |
| 28 | Reapply DHCP DNS credential manually | Target DHCP Server | `Set-DhcpServerDnsCredential -Credential (Get-Credential)` | Secure DNS updates can work |
| 29 | Recreate or validate failover if needed | DHCP Servers | Failover workbook commands | HA state is restored |
| 30 | Validate client lease | Client / DHCP Server | `ipconfig /renew`; `Get-DhcpServerv4Lease` | Client receives correct lease |
| 31 | Document backup and restore result | Operator | `Record backup path, file names, scopes, leases, authorization, import result, and validation` | Workbook record is complete |

# Backup_Restore_And_Export_DHCP_Server_Config_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the source DHCP server.
# Purpose: capture baseline DHCP server state before backup, export, import, or restore.

$BackupRoot = "D:\DHCP-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$EvidencePath = Join-Path $BackupRunPath "Precheck"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Capture server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion |
  Tee-Object "$EvidencePath\computer-info.txt"

# Capture DHCP role and service.
Get-WindowsFeature DHCP |
  Tee-Object "$EvidencePath\dhcp-role-state.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-state.txt"

# Import DHCP module.
Import-Module DhcpServer

# Capture DHCP authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Capture network state.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all.txt"

# Capture DHCP server settings.
Get-DhcpServerSetting |
  Tee-Object "$EvidencePath\dhcp-server-settings.txt"

# Capture database settings if available.
Get-DhcpServerDatabase -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-database-settings.txt"

# Capture audit log settings if available.
Get-DhcpServerAuditLog -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-audit-log-settings.txt"

# Capture existing failover.
Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-failover-before.txt"

# Capture DNS credential state.
Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-dns-credential-before.txt"

# Capture recent events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-operational-events-before.txt"
```

# Backup_Restore_And_Export_DHCP_Server_Config_Inventory_Export_Skeleton
```powershell
# Run in elevated PowerShell on the source DHCP server.
# Purpose: export human-readable DHCP inventory for review, audit, and restore validation.

$BackupRoot = "D:\DHCP-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$InventoryPath = Join-Path $BackupRunPath "Inventory"

New-Item -ItemType Directory -Force -Path $InventoryPath

Import-Module DhcpServer

# Export server authorization and settings.
Get-DhcpServerInDC |
  Tee-Object "$InventoryPath\authorized-dhcp-servers.txt"

Get-DhcpServerSetting |
  Tee-Object "$InventoryPath\dhcp-server-settings.txt"

Get-DhcpServerDatabase -ErrorAction SilentlyContinue |
  Tee-Object "$InventoryPath\dhcp-database-settings.txt"

Get-DhcpServerAuditLog -ErrorAction SilentlyContinue |
  Tee-Object "$InventoryPath\dhcp-audit-log-settings.txt"

# Export scopes.
$Scopes = Get-DhcpServerv4Scope

$Scopes |
  Tee-Object "$InventoryPath\dhcp-v4-scopes.txt"

$Scopes |
  Export-Csv "$InventoryPath\dhcp-v4-scopes.csv" -NoTypeInformation

# Export server-level options.
Get-DhcpServerv4OptionValue -ErrorAction SilentlyContinue |
  Tee-Object "$InventoryPath\server-level-options.txt"

Get-DhcpServerv4OptionValue -ErrorAction SilentlyContinue |
  Export-Csv "$InventoryPath\server-level-options.csv" -NoTypeInformation

# Export server-level DNS update settings.
Get-DhcpServerv4DnsSetting -ErrorAction SilentlyContinue |
  Tee-Object "$InventoryPath\server-level-dns-update-settings.txt"

# Export classes and filters if used.
Get-DhcpServerv4Class -ErrorAction SilentlyContinue |
  Tee-Object "$InventoryPath\dhcp-v4-classes.txt"

Get-DhcpServerv4FilterList -ErrorAction SilentlyContinue |
  Tee-Object "$InventoryPath\dhcp-v4-filter-list.txt"

Get-DhcpServerv4Filter -List Allow -ErrorAction SilentlyContinue |
  Export-Csv "$InventoryPath\dhcp-v4-allow-filters.csv" -NoTypeInformation

Get-DhcpServerv4Filter -List Deny -ErrorAction SilentlyContinue |
  Export-Csv "$InventoryPath\dhcp-v4-deny-filters.csv" -NoTypeInformation

# Export per-scope details.
foreach ($Scope in $Scopes) {
  $ScopeId = $Scope.ScopeId.IPAddressToString
  $ScopeFolderName = $ScopeId -replace '[\\/:*?"<>|]', '_'
  $ScopePath = Join-Path $InventoryPath "Scope-$ScopeFolderName"

  New-Item -ItemType Directory -Force -Path $ScopePath

  Get-DhcpServerv4Scope -ScopeId $ScopeId |
    Tee-Object "$ScopePath\scope.txt"

  Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
    Tee-Object "$ScopePath\scope-statistics.txt"

  Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$ScopePath\scope-options.txt"

  Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Export-Csv "$ScopePath\scope-options.csv" -NoTypeInformation

  Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$ScopePath\exclusions.txt"

  Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Export-Csv "$ScopePath\exclusions.csv" -NoTypeInformation

  Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$ScopePath\reservations.txt"

  Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Export-Csv "$ScopePath\reservations.csv" -NoTypeInformation

  Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$ScopePath\leases.txt"

  Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Export-Csv "$ScopePath\leases.csv" -NoTypeInformation

  Get-DhcpServerv4Policy -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$ScopePath\policies.txt"

  Get-DhcpServerv4DnsSetting -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$ScopePath\dns-update-settings.txt"
}
```

# Backup_Restore_And_Export_DHCP_Server_Config_XML_Export_Skeleton
```powershell
# Run in elevated PowerShell on the source DHCP server.
# Purpose: export DHCP server configuration and leases to a portable XML file.

$BackupRoot = "D:\DHCP-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$ExportPath = Join-Path $BackupRunPath "XML-Export"
$ExportFile = Join-Path $ExportPath "dhcp-export-with-leases.xml"

New-Item -ItemType Directory -Force -Path $ExportPath

Start-Transcript -Path "$ExportPath\dhcp-xml-export-transcript.txt"

Import-Module DhcpServer

# Export DHCP server config with leases.
Export-DhcpServer `
  -File $ExportFile `
  -Leases `
  -Force `
  -Verbose |
  Tee-Object "$ExportPath\export-dhcpserver-output.txt"

# Export config without leases as a second clean config copy.
Export-DhcpServer `
  -File (Join-Path $ExportPath "dhcp-export-config-only.xml") `
  -Force `
  -Verbose |
  Tee-Object "$ExportPath\export-dhcpserver-config-only-output.txt"

# Validate export files exist.
Get-ChildItem $ExportPath |
  Tee-Object "$ExportPath\export-files.txt"

# Hash export files for integrity evidence.
Get-FileHash $ExportFile |
  Tee-Object "$ExportPath\dhcp-export-with-leases-hash.txt"

Get-FileHash (Join-Path $ExportPath "dhcp-export-config-only.xml") |
  Tee-Object "$ExportPath\dhcp-export-config-only-hash.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_DHCP_Server_Config_Database_Backup_Skeleton
```powershell
# Run in elevated PowerShell on the source DHCP server.
# Purpose: create DHCP database backup using Backup-DhcpServer.

$BackupRoot = "D:\DHCP-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$DatabaseBackupPath = Join-Path $BackupRunPath "DatabaseBackup"
$EvidencePath = Join-Path $BackupRunPath "DatabaseBackup-Evidence"

New-Item -ItemType Directory -Force -Path $DatabaseBackupPath
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-database-backup-transcript.txt"

Import-Module DhcpServer

# Capture current database settings.
Get-DhcpServerDatabase -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\database-settings-before-backup.txt"

# Backup DHCP database.
Backup-DhcpServer `
  -Path $DatabaseBackupPath `
  -Verbose |
  Tee-Object "$EvidencePath\backup-dhcpserver-output.txt"

# Validate backup folder contents.
Get-ChildItem $DatabaseBackupPath -Recurse |
  Select-Object FullName,Length,LastWriteTime |
  Tee-Object "$EvidencePath\database-backup-files.txt"

# Copy DHCP database folder listing.
Get-ChildItem "$env:SystemRoot\System32\dhcp" -Recurse |
  Select-Object FullName,Length,LastWriteTime |
  Export-Csv "$EvidencePath\dhcp-system32-folder-file-list.csv" -NoTypeInformation

# Optional supporting copy of DHCP folder for evidence.
Copy-Item `
  -Path "$env:SystemRoot\System32\dhcp" `
  -Destination (Join-Path $EvidencePath "DHCP-Folder-Copy") `
  -Recurse `
  -Force

Stop-Transcript
```

# Backup_Restore_And_Export_DHCP_Server_Config_Server_Config_Backup_Skeleton
```powershell
# Run in elevated PowerShell on the source DHCP server.
# Purpose: capture server-level configuration outside simple scope export.

$BackupRoot = "D:\DHCP-Backups"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$BackupRunPath = Join-Path $BackupRoot $Timestamp
$ConfigPath = Join-Path $BackupRunPath "Server-Config"

New-Item -ItemType Directory -Force -Path $ConfigPath

Start-Transcript -Path "$ConfigPath\server-config-backup-transcript.txt"

Import-Module DhcpServer

# DHCP service state and role.
Get-WindowsFeature DHCP |
  Tee-Object "$ConfigPath\dhcp-role-state.txt"

Get-Service DHCPServer |
  Tee-Object "$ConfigPath\dhcp-service-state.txt"

# Server settings.
Get-DhcpServerSetting |
  Tee-Object "$ConfigPath\dhcp-server-settings.txt"

Get-DhcpServerDatabase -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-database-settings.txt"

Get-DhcpServerAuditLog -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-audit-log-settings.txt"

# Authorization.
Get-DhcpServerInDC |
  Tee-Object "$ConfigPath\authorized-dhcp-servers.txt"

# DNS update settings and credential state.
Get-DhcpServerv4DnsSetting -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\server-level-dns-update-settings.txt"

Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-dns-credential-state.txt"

# IPv4 bindings.
Get-DhcpServerv4Binding -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-v4-bindings.txt"

# IPv4 policies, filters, classes.
Get-DhcpServerv4Class -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-v4-classes.txt"

Get-DhcpServerv4FilterList -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-v4-filter-list.txt"

Get-DhcpServerv4Filter -List Allow -ErrorAction SilentlyContinue |
  Export-Csv "$ConfigPath\dhcp-v4-allow-filters.csv" -NoTypeInformation

Get-DhcpServerv4Filter -List Deny -ErrorAction SilentlyContinue |
  Export-Csv "$ConfigPath\dhcp-v4-deny-filters.csv" -NoTypeInformation

# Registry export for same-server evidence.
reg export "HKLM\SYSTEM\CurrentControlSet\Services\DHCPServer" "$ConfigPath\dhcp-service-registry.reg" /y

# DHCP server event export.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 500 -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-operational-events.txt"

Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 500 -ErrorAction SilentlyContinue |
  Tee-Object "$ConfigPath\dhcp-admin-events.txt"

Stop-Transcript
```

# Backup_Restore_And_Export_DHCP_Server_Config_Failover_And_DNS_Credential_Backup_Skeleton
```powershell
# Run in elevated PowerShell on each DHCP failover partner.
# Purpose: document failover relationships and DHCP DNS credential state before backup/restore/migration.

$EvidencePath = "D:\DHCP-Backups\Failover-And-DNS-Credential"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Capture local server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

# Capture DHCP DNS credential state.
Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-dns-credential-state.txt"

# Capture server-level DNS update settings.
Get-DhcpServerv4DnsSetting -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\server-dns-update-settings.txt"

# Capture failover relationships.
Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-failover-relationships.txt"

Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\dhcp-failover-relationships.csv" -NoTypeInformation

# Capture failover scopes per relationship.
$Failovers = Get-DhcpServerv4Failover -ErrorAction SilentlyContinue

foreach ($Failover in $Failovers) {
  $SafeName = $Failover.Name -replace '[\\/:*?"<>|]', '_'

  Get-DhcpServerv4FailoverScope -Name $Failover.Name -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\failover-scopes-$SafeName.txt"

  Get-DhcpServerv4FailoverScope -Name $Failover.Name -ErrorAction SilentlyContinue |
    Export-Csv "$EvidencePath\failover-scopes-$SafeName.csv" -NoTypeInformation
}

# Manual note:
# DHCP DNS credential passwords are not exported as plaintext.
# Reapply credential manually on restored or migrated DHCP servers.
# Use same credential on failover partners.
```

# Backup_Restore_And_Export_DHCP_Server_Config_Import_To_New_Server_Skeleton
```powershell
# Run in elevated PowerShell on the target DHCP server during approved migration window.
# Purpose: import DHCP configuration and leases from XML into a new or rebuilt DHCP server.
# Export current target state first if target already has DHCP config.

$TargetDhcpServer = "DHCP2.corp.local"
$SourceExportFile = "D:\DHCP-Backups\Latest\XML-Export\dhcp-export-with-leases.xml"
$ImportBackupPath = "D:\DHCP-Import-Backup"
$EvidencePath = "C:\DHCP-Import-Restore"

New-Item -ItemType Directory -Force -Path $ImportBackupPath
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-import-to-new-server-transcript.txt"

Import-Module DhcpServer

# Confirm DHCP role and service.
Get-WindowsFeature DHCP |
  Tee-Object "$EvidencePath\target-dhcp-role-state-before-import.txt"

Get-Service DHCPServer |
  Tee-Object "$EvidencePath\target-dhcp-service-state-before-import.txt"

# Capture target state before import.
Get-DhcpServerv4Scope -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scopes-before-import.txt"

Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-failover-before-import.txt"

# Import DHCP configuration and leases.
# Use -ScopeOverwrite only when approved because it overwrites existing target scope configuration.
Import-DhcpServer `
  -ComputerName $TargetDhcpServer `
  -File $SourceExportFile `
  -BackupPath $ImportBackupPath `
  -Leases `
  -ScopeOverwrite `
  -Force `
  -Verbose |
  Tee-Object "$EvidencePath\import-dhcpserver-output.txt"

# Confirm imported scopes.
Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\target-scopes-after-import.txt"

Get-DhcpServerv4Scope |
  Export-Csv "$EvidencePath\target-scopes-after-import.csv" -NoTypeInformation

# Confirm options and leases per scope.
$Scopes = Get-DhcpServerv4Scope

foreach ($Scope in $Scopes) {
  $ScopeId = $Scope.ScopeId.IPAddressToString
  $SafeScope = $ScopeId -replace '[\\/:*?"<>|]', '_'

  Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\scope-options-$SafeScope-after-import.txt"

  Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\leases-$SafeScope-after-import.txt"
}

# Authorize the target DHCP server if this is a domain DHCP server and not already authorized.
# Add-DhcpServerInDC -DnsName $TargetDhcpServer -IPAddress "<target-dhcp-ip>"

# Reapply DHCP DNS credential manually.
# Set-DhcpServerDnsCredential -Credential (Get-Credential)

Stop-Transcript
```

# Backup_Restore_And_Export_DHCP_Server_Config_Database_Restore_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server during approved restore window.
# Purpose: restore DHCP database from Backup-DhcpServer output.
# Best suited for same-server or controlled restore scenarios.

$DatabaseBackupPath = "D:\DHCP-Backups\Latest\DatabaseBackup"
$EvidencePath = "C:\DHCP-Database-Restore"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcp-database-restore-transcript.txt"

Import-Module DhcpServer

# Capture current state before restore.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-before-restore.txt"

Get-DhcpServerv4Scope -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scopes-before-restore.txt"

Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\failover-before-restore.txt"

# Stop DHCP service before database restore if required by restore procedure.
Stop-Service DHCPServer

# Restore DHCP database.
Restore-DhcpServer `
  -Path $DatabaseBackupPath `
  -Force `
  -Verbose |
  Tee-Object "$EvidencePath\restore-dhcpserver-output.txt"

# Start DHCP service.
Start-Service DHCPServer

# Confirm service and scopes after restore.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-after-restore.txt"

Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\scopes-after-restore.txt"

Get-DhcpServerv4Scope |
  Export-Csv "$EvidencePath\scopes-after-restore.csv" -NoTypeInformation

# Confirm leases after restore.
$Scopes = Get-DhcpServerv4Scope

foreach ($Scope in $Scopes) {
  $ScopeId = $Scope.ScopeId.IPAddressToString
  $SafeScope = $ScopeId -replace '[\\/:*?"<>|]', '_'

  Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\leases-$SafeScope-after-restore.txt"
}

Stop-Transcript
```

# Backup_Restore_And_Export_DHCP_Server_Config_Post_Restore_Validation_Skeleton
```powershell
# Run after import or restore on DHCP server, DNS server, and one DHCP client.
# Purpose: validate scopes, options, leases, authorization, DNS updates, and client lease behavior.

$ScopeId = "10.20.20.0"
$TestClientMac = "00-11-22-33-44-55"
$ExpectedGateway = "10.20.20.1"
$ExpectedDnsServer = "10.10.10.10"
$DomainFqdn = "corp.local"
$EvidencePath = "C:\DHCP-Post-Restore-Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm DHCP service.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-post-restore.txt"

# Confirm authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers-post-restore.txt"

# Confirm scopes.
Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\scopes-post-restore.txt"

Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-post-restore.txt"

# Confirm scope options.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-options-post-restore.txt"

# Confirm exclusions/reservations/leases.
Get-DhcpServerv4ExclusionRange -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-exclusions-post-restore.txt"

Get-DhcpServerv4Reservation -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-reservations-post-restore.txt"

Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-leases-post-restore.txt"

Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-statistics-post-restore.txt"

# Confirm DHCP DNS settings.
Get-DhcpServerv4DnsSetting -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\server-dns-update-settings-post-restore.txt"

Get-DhcpServerv4DnsSetting -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\scope-dns-update-settings-post-restore.txt"

Get-DhcpServerDnsCredential -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-dns-credential-post-restore.txt"

# Confirm failover state if applicable.
Get-DhcpServerv4Failover -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\failover-post-restore.txt"

# Confirm test client lease if MAC is known.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ClientId $TestClientMac -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\test-client-lease-post-restore.txt"

# Event logs.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-operational-events-post-restore.txt"

# Client-side commands to run on test client:
# ipconfig /release
# ipconfig /renew
# ipconfig /all
# Test-NetConnection $ExpectedGateway
# Test-NetConnection $ExpectedDnsServer -Port 53
# Resolve-DnsName $DomainFqdn -Server $ExpectedDnsServer
```

# Backup_Restore_And_Export_DHCP_Server_Config_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on DHCP server.
# Purpose: collect DHCP backup, export, import, restore, service, and lease event evidence.

$EvidencePath = "C:\DHCP-Backup-Restore\Events"
$Since = (Get-Date).AddHours(-24)

New-Item -ItemType Directory -Force -Path $EvidencePath

# DHCP Server operational events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Dhcp-Server/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-operational-events-last-24h.txt"

# DHCP Server admin events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-Dhcp-Server/Admin"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-admin-events-last-24h.txt"

# System DHCP/service events.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DHCP*" -or
    $_.Message -like "*DHCP*" -or
    $_.Message -like "*backup*" -or
    $_.Message -like "*restore*" -or
    $_.Message -like "*database*" -or
    $_.Message -like "*lease*"
  } |
  Tee-Object "$EvidencePath\system-dhcp-backup-restore-events-last-24h.txt"

# Final DHCP service state.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"

# Final scope state.
Import-Module DhcpServer

Get-DhcpServerv4Scope -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-scopes-final.txt"
```

# Backup_Restore_And_Export_DHCP_Server_Config_Verification_Commands
```powershell
# DHCP role and service.
Get-WindowsFeature DHCP
Get-Service DHCPServer
Import-Module DhcpServer

# Authorization.
Get-DhcpServerInDC
Add-DhcpServerInDC -DnsName "<dhcp-server-fqdn>" -IPAddress "<dhcp-server-ip>"

# Server settings.
Get-DhcpServerSetting
Get-DhcpServerDatabase
Get-DhcpServerAuditLog
Get-DhcpServerv4Binding

# Scopes.
Get-DhcpServerv4Scope
Get-DhcpServerv4Scope -ScopeId "<scope-id>"
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"

# Options.
Get-DhcpServerv4OptionValue
Get-DhcpServerv4OptionValue -ScopeId "<scope-id>"

# Exclusions, reservations, leases.
Get-DhcpServerv4ExclusionRange -ScopeId "<scope-id>"
Get-DhcpServerv4Reservation -ScopeId "<scope-id>"
Get-DhcpServerv4Lease -ScopeId "<scope-id>"

# Policies, classes, filters.
Get-DhcpServerv4Policy -ScopeId "<scope-id>"
Get-DhcpServerv4Class
Get-DhcpServerv4FilterList
Get-DhcpServerv4Filter -List Allow
Get-DhcpServerv4Filter -List Deny

# DNS update settings and credential.
Get-DhcpServerv4DnsSetting
Get-DhcpServerv4DnsSetting -ScopeId "<scope-id>"
Get-DhcpServerDnsCredential
Set-DhcpServerDnsCredential -Credential (Get-Credential)
Remove-DhcpServerDnsCredential

# Failover.
Get-DhcpServerv4Failover
Get-DhcpServerv4Failover -Name "<failover-name>"
Get-DhcpServerv4FailoverScope -Name "<failover-name>"
Invoke-DhcpServerv4FailoverReplication -Name "<failover-name>" -Force

# XML export with leases.
Export-DhcpServer `
  -File "<backup-path>\dhcp-export-with-leases.xml" `
  -Leases `
  -Force `
  -Verbose

# XML export config only.
Export-DhcpServer `
  -File "<backup-path>\dhcp-export-config-only.xml" `
  -Force `
  -Verbose

# XML import with leases.
Import-DhcpServer `
  -ComputerName "<target-dhcp-server>" `
  -File "<backup-path>\dhcp-export-with-leases.xml" `
  -BackupPath "<import-backup-path>" `
  -Leases `
  -ScopeOverwrite `
  -Force `
  -Verbose

# DHCP database backup.
Backup-DhcpServer `
  -Path "<database-backup-path>" `
  -Verbose

# DHCP database restore.
Restore-DhcpServer `
  -Path "<database-backup-path>" `
  -Force `
  -Verbose

# Registry export.
reg export "HKLM\SYSTEM\CurrentControlSet\Services\DHCPServer" "<backup-path>\dhcp-service-registry.reg" /y

# File integrity.
Get-ChildItem "<backup-path>" -Recurse
Get-FileHash "<backup-path>\dhcp-export-with-leases.xml"

# Client validation.
ipconfig /release
ipconfig /renew
ipconfig /all
Test-NetConnection "<gateway-ip>"
Test-NetConnection "<dns-server-ip>" -Port 53
Resolve-DnsName "<domain-fqdn>" -Server "<dns-server-ip>"

# Events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DHCP*" -or $_.Message -like "*DHCP*"}
```

# Backup_Restore_And_Export_DHCP_Server_Config_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| XML export created | No rollback required | Export is read-only |
| Database backup created | No rollback required | Backup is read-only |
| Inventory exported | No rollback required | Inventory is read-only |
| DHCP registry exported | No rollback required | Export is read-only |
| Import overwrote target scope | Restore target from pre-import export or database backup | Existing target config may be lost |
| Import included wrong leases | Re-import correct XML or clear affected leases carefully | Clients may receive unexpected renewals |
| Database restored incorrectly | Restore from prior known-good backup | DHCP may lease wrong scope data |
| DHCP server authorized accidentally | `Remove-DhcpServerInDC -DnsName "<server>" -IPAddress "<ip>"` | Removing active server authorization breaks leasing |
| DHCP DNS credential removed | Reapply credential with `Set-DhcpServerDnsCredential` | Secure DNS updates fail until restored |
| Failover relationship lost | Recreate failover relationship from inventory evidence | HA unavailable until rebuilt |
| DHCP service stopped for restore | `Start-Service DHCPServer` | Clients cannot lease until service starts |
| Scope imported inactive | Activate scope with `Set-DhcpServerv4Scope -State Active` | Clients cannot receive leases |
| Wrong options imported | Restore option values from inventory | Clients may get wrong gateway/DNS/suffix |
| Evidence folder created | `Remove-Item C:\DHCP-Backup-Restore -Recurse -Force` | Deletes restore evidence |

# Backup_Restore_And_Export_DHCP_Server_Config_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Export-DhcpServer fails | Permissions, path, service, or module issue | Elevated PowerShell and file path | Reinstalling DHCP |
| XML file missing after export | Wrong export path or permission issue | `Get-ChildItem <backup-path>` | Running import |
| Import-DhcpServer fails | Existing scope conflict, bad XML, version/path issue | Import output and target scope state | Deleting DHCP role |
| Target has duplicate scopes | Imported into server with existing scopes | Pre-import target inventory | Authorizing target first |
| Imported scopes inactive | Scope state imported or disabled | `Get-DhcpServerv4Scope` | Editing leases |
| Clients do not receive leases after import | Authorization, service, scope state, relay, or firewall | Service, authorization, scope state, IP helper | Re-importing immediately |
| Reservations missing | Export/import issue or wrong XML file | Reservation inventory and target reservations | Recreating all scopes |
| Leases missing | Export without `-Leases` or import without `-Leases` | Export/import command history | Blaming clients |
| Options missing | Server/scope options not imported or overwritten | `Get-DhcpServerv4OptionValue` | Editing reservations |
| DNS updates fail after restore | DHCP DNS credential not reapplied | `Get-DhcpServerDnsCredential` | Changing DNS zones first |
| Failover missing after migration | Relationship not recreated or not validated | Failover inventory | Assuming XML fully rebuilt HA |
| Partner scope missing | Failover replication not run | `Invoke-DhcpServerv4FailoverReplication` | Recreating scope manually |
| Authorization missing | New DHCP server not authorized in AD | `Get-DhcpServerInDC` | Restarting client |
| Rogue old DHCP server still leasing | Old server still authorized/running | Authorization list and network checks | Editing target scopes |
| DHCP service will not start | Database restore/config issue | System and DHCP event logs | Importing registry blindly |
| Database restore fails | Bad backup path or locked service/database | Backup folder contents and service state | Deleting database files manually |
| Import overwrites wrong target | Wrong ComputerName or wrong file | Confirm host and XML hash | Continuing restore |
| Client gets old IP | Existing lease or client not renewed | `ipconfig /release`; `ipconfig /renew` | Editing server config |
| Client gets APIPA | Scope inactive, relay missing, server unauthorized | Scope/service/authorization/relay | Re-importing XML |
| Event logs show database error | Corrupt database or restore issue | DHCP/Admin and System logs | Expanding scope |

# Backup_Restore_And_Export_DHCP_Server_Config_Related_Labs
| Lab                                                | Related Workbook                                         | Skill Proven                                                  |
| -------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------- |
| Install DHCP Server role and management tools      | `01_Install_DHCP_Server_Role_And_Management_Tools.md`    | DHCP role baseline before restore                             |
| Authorize DHCP Server in Active Directory          | `02_Authorize_DHCP_Server_In_Active_Directory.md`        | Post-restore domain authorization                             |
| Create DHCPv4 scope                                | `03_Create_DHCPv4_Scope.md`                              | Scope validation after import                                 |
| Configure DHCPv4 scope options                     | `04_Configure_DHCPv4_Scope_Options.md`                   | Option validation after restore                               |
| Configure DHCPv4 exclusions and reservations       | `05_Configure_DHCPv4_Exclusions_And_Reservations.md`     | Reservation and exclusion validation                          |
| Validate DHCP client lease assignment              | `06_Validate_DHCP_Client_Lease_Assignment.md`            | Client-side validation after restore                          |
| Configure DHCP relay and IP helper                 | `07_Configure_DHCP_Relay_And_IP_Helper.md`               | Relay validation after DHCP migration                         |
| Configure DHCP DNS dynamic updates                 | `08_Configure_DHCP_DNS_Dynamic_Updates.md`               | DNS credential and A/PTR update validation                    |
| Configure DHCPv4 failover                          | `09_Configure_DHCPv4_Failover.md`                        | Failover relationship backup and recovery awareness           |
| Backup, restore, and export DHCP Server config     | `10_Backup_Restore_And_Export_DHCP_Server_Config.md`     | DHCP backup, export, import, restore, and validation workflow |
| Monitor DHCP leases, events, and scope utilization | `11_Monitor_DHCP_Leases_Events_And_Scope_Utilization.md` | Post-restore monitoring                                       |
| Troubleshoot DHCP client lease failures            | `12_Troubleshoot_DHCP_Client_Lease_Failures.md`          | Restore-related lease failure triage                          |
| Troubleshoot DHCP failover and replication         | `17_Troubleshoot_DHCP_Failover_And_Replication.md`       | Failover recovery and replication validation                  |