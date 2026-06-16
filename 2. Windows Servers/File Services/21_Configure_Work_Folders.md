21_Configure_Work_Folders.md
# Configure_Work_Folders

# Configure_Work_Folders_Index
21_Configure_Work_Folders.md
Configure_Work_Folders
Configure_Work_Folders_Source_Basis
Configure_Work_Folders_Mental_Model
Configure_Work_Folders_Planning_Table
Configure_Work_Folders_Configuration_Checklist
Configure_Work_Folders_Feature_Install_Skeleton
Configure_Work_Folders_Certificate_And_HTTPS_Binding_Skeleton
Configure_Work_Folders_DNS_And_Discovery_Skeleton
Configure_Work_Folders_AD_Group_And_User_Targeting_Skeleton
Configure_Work_Folders_Sync_Share_Skeleton
Configure_Work_Folders_Client_GPO_Skeleton
Configure_Work_Folders_Client_Enrollment_Skeleton
Configure_Work_Folders_Status_And_Event_Review_Skeleton
Configure_Work_Folders_Verification_Commands
Configure_Work_Folders_Rollback
Configure_Work_Folders_Failure_Checks
Configure_Work_Folders_Related_Labs

# Configure_Work_Folders_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server Work Folders overview | Role service for file servers | Work Folders purpose and file server role |
| Windows Server Work Folders overview | Requirements | NTFS volume, certificate, AD DS, client requirements |
| Windows Server Work Folders deployment | Deployment steps | Certificate, DNS, role install, SSL binding, groups, sync shares |
| SyncShare PowerShell module | New-SyncShare | Creating Work Folders sync shares |
| SyncShare PowerShell module | Get-SyncShare | Reviewing existing sync shares |
| SyncShare PowerShell module | Set-SyncShare | Modifying sync share policy |
| SyncShare PowerShell module | Get-SyncServerSetting | Reviewing Work Folders server settings |
| SyncShare PowerShell module | Set-SyncServerSetting | Configuring administrator email and server settings |
| Active Directory PowerShell | New-ADGroup, Set-ADUser | Security groups and automatic server discovery |
| Group Policy | Work Folders client settings | Automatic Work Folders setup for domain-joined clients |
| Operational file server practice | User data, sync, restore, and troubleshooting | Safe deployment and support workflow |

# Configure_Work_Folders_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Work Folders | Windows feature that syncs user work files between a file server and user devices |
| Sync server | Windows Server file server hosting Work Folders |
| Sync share | Server-side folder root used to store user Work Folders data |
| User folder | Per-user folder created under the sync share |
| Client enrollment | User or policy-driven setup on Windows client |
| HTTPS sync | Work Folders sync traffic uses HTTPS to the sync server |
| Certificate binding | Server certificate must be bound to HTTPS for client trust |
| Work Folders discovery | Client locates sync server through DNS, user attribute, or configured URL |
| `workfolders.domain.com` | Common discovery name used by clients |
| `msDS-SyncServerURL` | AD user attribute that can point users to their sync server |
| Device policy | Sync share policy can request encryption and screen lock behavior |
| SyncShare module | PowerShell module used to manage Work Folders server configuration |
| FSRM compatibility | Quotas and file screens can manage user data stored in sync shares |
| Not SMB | Users do not normally browse Work Folders as a normal SMB share from the client side |
| Not OneDrive | Work Folders is file-server based, not Microsoft 365 cloud storage |
| Blunt rule | Work Folders is useful for controlled file-server sync, but it is not a replacement for SharePoint or OneDrive collaboration |

# Configure_Work_Folders_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Sync server | `FS1` | `<sync-server-name>` |
| Sync server FQDN | `FS1.corp.local` | `<sync-server-fqdn>` |
| Public Work Folders URL | `workfolders.contoso.com` | `<workfolders-public-name>` |
| Internal discovery name | `workfolders.corp.local` | `<workfolders-internal-name>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Sync share name | `DepartmentsWorkFolders` | `<sync-share-name>` |
| Sync share path | `D:\WorkFolders\Departments` | `<sync-share-path>` |
| User folder format | `[user]@[domain]` | `<user-folder-format>` |
| Allowed sync group | `CORP\GG_WorkFolders_Departments` | `<allowed-sync-group>` |
| Admin group | `CORP\GG_WorkFolders_Admins` | `<workfolders-admin-group>` |
| Certificate subject | `workfolders.contoso.com` | `<cert-subject>` |
| Certificate SAN | `FS1.corp.local`, `workfolders.contoso.com` | `<cert-san-list>` |
| Certificate thumbprint | `ABCD1234...` | `<cert-thumbprint>` |
| HTTPS binding IP | `0.0.0.0` or server IP | `<https-binding-ip>` |
| Sync server URL | `https://FS1.corp.local` | `<sync-server-url>` |
| Require client encryption | True / False | `<require-encryption>` |
| Require password auto lock | True / False | `<require-password-autolock>` |
| Admin support email | `helpdesk@corp.local` | `<support-email>` |
| Client OU | `OU=Workstations,DC=corp,DC=local` | `<client-ou>` |
| User OU | `OU=Users,DC=corp,DC=local` | `<user-ou>` |
| GPO name | `GPO_WorkFolders_Client_Config` | `<gpo-name>` |
| Report path | `C:\WorkFolders-Reports` | `<report-path>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Configure_Work_Folders_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm sync server identity | Sync Server | `hostname` | Correct server is identified |
| 2 | Confirm IP and DNS configuration | Sync Server | `Get-NetIPConfiguration` | Server IP and DNS configuration are visible |
| 3 | Confirm target volume | Sync Server | `Get-Volume -DriveLetter D` | NTFS data volume is present and healthy |
| 4 | Create sync root folder | Sync Server | `New-Item -ItemType Directory -Path "D:\WorkFolders\Departments" -Force` | Sync share root exists |
| 5 | Confirm folder path | Sync Server | `Test-Path "D:\WorkFolders\Departments"` | Returns True |
| 6 | Check Work Folders feature state | Sync Server | `Get-WindowsFeature FS-SyncShareService` | Feature state is visible |
| 7 | Install Work Folders role service | Sync Server | `Install-WindowsFeature FS-SyncShareService -IncludeManagementTools` | Work Folders role service and tools install |
| 8 | Import SyncShare module | Sync Server | `Import-Module SyncShare` | SyncShare cmdlets are available |
| 9 | Confirm SyncShare cmdlets | Sync Server | `Get-Command -Module SyncShare` | Work Folders cmdlets are listed |
| 10 | Confirm Sync Share service | Sync Server | `Get-Service SyncShareSvc` | Service is visible |
| 11 | Confirm IIS Hostable Web Core dependency | Sync Server | `Get-WindowsFeature Web-WHC` | Hostable Web Core state is visible if installed |
| 12 | Confirm certificate exists | Sync Server | `Get-ChildItem Cert:\LocalMachine\My | Where-Object Subject -like "*workfolders*"` | Certificate is present in local computer personal store |
| 13 | Record certificate thumbprint | Sync Server | `Get-ChildItem Cert:\LocalMachine\My | Select Subject,DnsNameList,Thumbprint,NotAfter` | Thumbprint and expiration are documented |
| 14 | Bind HTTPS certificate | Sync Server | `netsh http add sslcert ipport=0.0.0.0:443 certhash=<cert-thumbprint> appid="{<guid>}" certstorename=MY` | HTTPS certificate is bound |
| 15 | Confirm HTTPS binding | Sync Server | `netsh http show sslcert` | SSL binding appears |
| 16 | Enable HTTPS firewall rule | Sync Server | `New-NetFirewallRule -DisplayName "Work Folders HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow -Profile Domain` | TCP 443 is allowed |
| 17 | Create Work Folders security group | Domain Controller | `New-ADGroup -Name "GG_WorkFolders_Departments" -SamAccountName "GG_WorkFolders_Departments" -GroupCategory Security -GroupScope Global -Path "<user-ou>"` | Sync access group exists |
| 18 | Add test user to sync group | Domain Controller | `Add-ADGroupMember -Identity "GG_WorkFolders_Departments" -Members "<username>"` | User is in allowed sync group |
| 19 | Create Work Folders admin group | Domain Controller | `New-ADGroup -Name "GG_WorkFolders_Admins" -SamAccountName "GG_WorkFolders_Admins" -GroupCategory Security -GroupScope Global -Path "<user-ou>"` | Admin delegation group exists |
| 20 | Confirm group membership | Domain Controller | `Get-ADGroupMember "GG_WorkFolders_Departments"` | Expected test user is listed |
| 21 | Create sync share | Sync Server | `New-SyncShare -Name "DepartmentsWorkFolders" -Path "D:\WorkFolders\Departments" -User "CORP\GG_WorkFolders_Departments" -UserFolderName "[user]@[domain]" -RequireEncryption $true -RequirePasswordAutoLock $true` | Sync share is created |
| 22 | Confirm sync share | Sync Server | `Get-SyncShare -Name "DepartmentsWorkFolders" | Format-List *` | Sync share details are visible |
| 23 | Configure support email | Sync Server | `Set-SyncServerSetting -AdministratorEmail "helpdesk@corp.local"` | Support email is configured |
| 24 | Confirm server settings | Sync Server | `Get-SyncServerSetting | Format-List *` | Sync server settings are visible |
| 25 | Configure internal DNS discovery | DNS Server | `Add-DnsServerResourceRecordCName -ZoneName "corp.local" -Name "workfolders" -HostNameAlias "FS1.corp.local"` | `workfolders.corp.local` resolves to sync server |
| 26 | Test DNS discovery name | Client | `Resolve-DnsName workfolders.corp.local` | Discovery name resolves |
| 27 | Optionally set user sync server URL | Domain Controller | `Set-ADUser "<username>" -Add @{"msDS-SyncServerURL"="https://FS1.corp.local"}` | User object points to sync server |
| 28 | Confirm user sync server URL | Domain Controller | `Get-ADUser "<username>" -Properties msDS-SyncServerURL | Select Name,msDS-SyncServerURL` | Attribute is visible |
| 29 | Create Work Folders client GPO | Management Host | `Create GPO: GPO_WorkFolders_Client_Config` | GPO exists |
| 30 | Configure Work Folders user policy | Management Host | `User Configuration\Policies\Administrative Templates\Windows Components\Work Folders\Specify Work Folders settings` | Client points to Work Folders URL |
| 31 | Configure automatic setup if approved | Management Host | `Computer Configuration\Policies\Administrative Templates\Windows Components\Work Folders\Force automatic setup for all users` | Auto setup policy is configured |
| 32 | Link GPO to client OU | Management Host | `Link GPO to <client-ou>` | Client computers receive policy |
| 33 | Refresh client policy | Client | `gpupdate /force` | GPO refresh completes |
| 34 | Confirm GPO application | Client | `gpresult /h C:\Temp\workfolders-gpresult.html` | Work Folders policy appears |
| 35 | Test HTTPS reachability | Client | `Test-NetConnection FS1.corp.local -Port 443` | TCP 443 succeeds |
| 36 | Open Work Folders client UI | Client | `control /name Microsoft.WorkFolders` | Work Folders control panel opens |
| 37 | Configure client manually if needed | Client | `Use Work Folders URL: https://FS1.corp.local` | Client enrolls successfully |
| 38 | Confirm local Work Folders path | Client | `Test-Path "$env:USERPROFILE\Work Folders"` | Local sync folder exists after setup |
| 39 | Create test file | Client | `"wf-test" | Out-File "$env:USERPROFILE\Work Folders\wf-test.txt"` | Test file is created locally |
| 40 | Confirm server-side user folder | Sync Server | `Get-ChildItem "D:\WorkFolders\Departments" -Recurse` | User folder and synced file appear |
| 41 | Review user sync status | Sync Server | `Get-SyncUserStatus` | User sync status is visible |
| 42 | Review Work Folders events | Sync Server | `Get-WinEvent -ListLog "*Sync*","*WorkFolders*"` | Relevant logs are discoverable |
| 43 | Export evidence | Sync Server | `Get-SyncShare | Export-Clixml "C:\WorkFolders-Reports\syncshares.xml"` | Configuration evidence is saved |
| 44 | Document final state | Operator | `Record URL, certificate, DNS, sync share, group, GPO, client test, and rollback` | Change record is complete |

# Configure_Work_Folders_Feature_Install_Skeleton
```powershell
# Run in elevated PowerShell on the Work Folders sync server.
# Purpose: install Work Folders role service and confirm SyncShare management tooling.

$FeatureName = "FS-SyncShareService"
$ReportRoot = "C:\WorkFolders-Reports"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Checking Work Folders feature state..."
Get-WindowsFeature $FeatureName

Write-Host "Installing Work Folders role service..."
Install-WindowsFeature $FeatureName -IncludeManagementTools

Write-Host "Importing SyncShare module..."
Import-Module SyncShare

Write-Host "Available SyncShare cmdlets:"
Get-Command -Module SyncShare | Sort-Object Name

Write-Host "Work Folders service state:"
Get-Service SyncShareSvc -ErrorAction SilentlyContinue |
  Format-Table Name, DisplayName, Status, StartType -AutoSize

Write-Host "Current Sync Server settings:"
Get-SyncServerSetting -ErrorAction SilentlyContinue |
  Format-List *

Write-Host "Current Sync Shares:"
Get-SyncShare -ErrorAction SilentlyContinue |
  Format-Table Name, Path, Enabled, UserFolderName -AutoSize
```

# Configure_Work_Folders_Certificate_And_HTTPS_Binding_Skeleton
```powershell
# Run in elevated PowerShell on the Work Folders sync server.
# Purpose: validate certificate presence and bind it to HTTPS for Work Folders sync traffic.

$CertSubjectMatch = "workfolders"
$HttpsIp = "0.0.0.0"
$HttpsPort = "443"
$AppGuid = (New-Guid).Guid

Write-Host "Finding candidate Work Folders certificate..."
$Cert = Get-ChildItem Cert:\LocalMachine\My |
  Where-Object {
    $_.Subject -like "*$CertSubjectMatch*" -or
    ($_.DnsNameList -join ",") -like "*$CertSubjectMatch*"
  } |
  Sort-Object NotAfter -Descending |
  Select-Object -First 1

if (-not $Cert) {
  throw "No certificate found in Cert:\LocalMachine\My matching $CertSubjectMatch"
}

Write-Host "Selected certificate:"
$Cert | Select-Object Subject, DnsNameList, Thumbprint, NotAfter | Format-List

Write-Host "Existing HTTP SSL certificate bindings:"
netsh http show sslcert

Write-Host "Adding HTTPS certificate binding..."
netsh http add sslcert `
  ipport="$HttpsIp`:$HttpsPort" `
  certhash=$($Cert.Thumbprint) `
  appid="{$AppGuid}" `
  certstorename=MY

Write-Host "Updated HTTP SSL certificate bindings:"
netsh http show sslcert

Write-Host "Creating inbound firewall rule for HTTPS if missing..."
if (-not (Get-NetFirewallRule -DisplayName "Work Folders HTTPS" -ErrorAction SilentlyContinue)) {
  New-NetFirewallRule `
    -DisplayName "Work Folders HTTPS" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 443 `
    -Action Allow `
    -Profile Domain
}

Get-NetFirewallRule -DisplayName "Work Folders HTTPS" |
  Format-Table DisplayName, Enabled, Direction, Action, Profile -AutoSize
```

# Configure_Work_Folders_DNS_And_Discovery_Skeleton
```powershell
# Run DNS commands on a DNS server or management host with DNS tools.
# Run AD commands on a domain controller or management host with RSAT.

$ZoneName = "corp.local"
$WorkFoldersAlias = "workfolders"
$SyncServerFqdn = "FS1.corp.local"
$SyncServerUrl = "https://FS1.corp.local"
$TestUser = "jsmith"

Write-Host "Creating internal Work Folders CNAME record..."
Add-DnsServerResourceRecordCName `
  -ZoneName $ZoneName `
  -Name $WorkFoldersAlias `
  -HostNameAlias $SyncServerFqdn `
  -ErrorAction Stop

Write-Host "Testing DNS resolution..."
Resolve-DnsName "$WorkFoldersAlias.$ZoneName"

Write-Host "Optionally setting AD user Work Folders discovery URL..."
Set-ADUser $TestUser -Add @{
  "msDS-SyncServerURL" = $SyncServerUrl
}

Write-Host "Verifying user discovery URL..."
Get-ADUser $TestUser -Properties msDS-SyncServerURL |
  Select-Object SamAccountName, msDS-SyncServerURL
```

# Configure_Work_Folders_AD_Group_And_User_Targeting_Skeleton
```powershell
# Run on domain controller or management host with Active Directory PowerShell.
# Purpose: create Work Folders access groups and add test users.

$DomainNetbios = "CORP"
$GroupOU = "OU=Groups,DC=corp,DC=local"
$AllowedGroupName = "GG_WorkFolders_Departments"
$AdminGroupName = "GG_WorkFolders_Admins"
$TestUsers = @("jsmith")

Write-Host "Creating allowed sync group..."
if (-not (Get-ADGroup -Filter "SamAccountName -eq '$AllowedGroupName'" -ErrorAction SilentlyContinue)) {
  New-ADGroup `
    -Name $AllowedGroupName `
    -SamAccountName $AllowedGroupName `
    -GroupCategory Security `
    -GroupScope Global `
    -Path $GroupOU
}

Write-Host "Creating Work Folders admin group..."
if (-not (Get-ADGroup -Filter "SamAccountName -eq '$AdminGroupName'" -ErrorAction SilentlyContinue)) {
  New-ADGroup `
    -Name $AdminGroupName `
    -SamAccountName $AdminGroupName `
    -GroupCategory Security `
    -GroupScope Global `
    -Path $GroupOU
}

Write-Host "Adding test users to allowed sync group..."
Add-ADGroupMember -Identity $AllowedGroupName -Members $TestUsers

Write-Host "Confirming allowed sync group membership..."
Get-ADGroupMember $AllowedGroupName |
  Select-Object Name, SamAccountName, objectClass
```

# Configure_Work_Folders_Sync_Share_Skeleton
```powershell
# Run in elevated PowerShell on the Work Folders sync server.
# Purpose: create a Work Folders sync share for a targeted user group.

$SyncShareName = "DepartmentsWorkFolders"
$SyncSharePath = "D:\WorkFolders\Departments"
$AllowedGroup = "CORP\GG_WorkFolders_Departments"
$SupportEmail = "helpdesk@corp.local"

Import-Module SyncShare

Write-Host "Creating sync share root..."
New-Item -ItemType Directory -Force -Path $SyncSharePath | Out-Null

Write-Host "Current NTFS ACL on sync share root:"
icacls $SyncSharePath

Write-Host "Creating Work Folders sync share..."
New-SyncShare `
  -Name $SyncShareName `
  -Path $SyncSharePath `
  -User $AllowedGroup `
  -UserFolderName "[user]@[domain]" `
  -RequireEncryption $true `
  -RequirePasswordAutoLock $true `
  -Description "Department Work Folders sync share"

Write-Host "Setting administrator support email..."
Set-SyncServerSetting -AdministratorEmail $SupportEmail

Write-Host "Sync share configuration:"
Get-SyncShare -Name $SyncShareName | Format-List *

Write-Host "Sync server settings:"
Get-SyncServerSetting | Format-List *
```

# Configure_Work_Folders_Client_GPO_Skeleton
```text
# Configure in Group Policy Management Console.
# Purpose: deploy Work Folders client settings to domain-joined Windows clients.

GPO Name:
GPO_WorkFolders_Client_Config

Recommended Scope:
Link to workstation OU first.
Do not immediately link to the full domain.

User Policy:
User Configuration
Policies
Administrative Templates
Windows Components
Work Folders
Specify Work Folders settings

Suggested Setting:
Enabled

Work Folders URL:
https://FS1.corp.local

Computer Policy:
Computer Configuration
Policies
Administrative Templates
Windows Components
Work Folders
Force automatic setup for all users

Suggested Setting:
Enabled only after pilot testing

Pilot Validation:
1. Link GPO to test workstation OU.
2. Run `gpupdate /force` on test client.
3. Run `gpresult /h C:\Temp\workfolders-gpresult.html`.
4. Confirm Work Folders policy appears.
5. Enroll a test user.
6. Confirm file sync to server.
7. Expand scope only after success.
```

# Configure_Work_Folders_Client_Enrollment_Skeleton
```powershell
# Run on a Windows client as the target user.
# Purpose: validate DNS, HTTPS, GPO, and Work Folders client enrollment.

$SyncServer = "FS1.corp.local"
$WorkFoldersDiscovery = "workfolders.corp.local"
$LocalWorkFoldersPath = Join-Path $env:USERPROFILE "Work Folders"

Write-Host "Confirming logged-on user..."
whoami

Write-Host "Confirming domain group token..."
whoami /groups | Select-String "WorkFolders"

Write-Host "Testing discovery DNS..."
Resolve-DnsName $WorkFoldersDiscovery -ErrorAction Continue

Write-Host "Testing sync server DNS..."
Resolve-DnsName $SyncServer -ErrorAction Continue

Write-Host "Testing HTTPS connectivity..."
Test-NetConnection $SyncServer -Port 443

Write-Host "Refreshing Group Policy..."
gpupdate /force

Write-Host "Generating gpresult report..."
New-Item -ItemType Directory -Force -Path "C:\Temp" | Out-Null
gpresult /h C:\Temp\workfolders-gpresult.html

Write-Host "Opening Work Folders Control Panel..."
control /name Microsoft.WorkFolders

Write-Host "After enrollment, validate local path:"
Test-Path $LocalWorkFoldersPath

if (Test-Path $LocalWorkFoldersPath) {
  "Work Folders test from $env:COMPUTERNAME by $env:USERNAME at $(Get-Date)" |
    Out-File (Join-Path $LocalWorkFoldersPath "wf-client-test.txt") -Encoding utf8

  Get-ChildItem $LocalWorkFoldersPath
}
```

# Configure_Work_Folders_Status_And_Event_Review_Skeleton
```powershell
# Run on the sync server.
# Purpose: export Work Folders configuration, user status, and event evidence.

$ReportRoot = "C:\WorkFolders-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Import-Module SyncShare

Write-Host "Export sync server settings..."
Get-SyncServerSetting |
  Export-Clixml "$ReportRoot\sync-server-settings-$Timestamp.xml"

Write-Host "Export sync shares..."
Get-SyncShare |
  Export-Clixml "$ReportRoot\sync-shares-$Timestamp.xml"

Write-Host "Export sync user status..."
Get-SyncUserStatus -ErrorAction SilentlyContinue |
  Export-Clixml "$ReportRoot\sync-user-status-$Timestamp.xml"

Write-Host "Export Work Folders service state..."
Get-Service SyncShareSvc -ErrorAction SilentlyContinue |
  Select-Object Name, DisplayName, Status, StartType |
  Export-Csv "$ReportRoot\syncshare-service-$Timestamp.csv" -NoTypeInformation

Write-Host "Export Work Folders related event log inventory..."
Get-WinEvent -ListLog "*Sync*","*WorkFolders*" -ErrorAction SilentlyContinue |
  Select-Object LogName, IsEnabled, RecordCount |
  Export-Csv "$ReportRoot\workfolders-eventlog-inventory-$Timestamp.csv" -NoTypeInformation

Write-Host "Export recent Work Folders related events..."
Get-WinEvent -ListLog "*Sync*","*WorkFolders*" -ErrorAction SilentlyContinue |
  ForEach-Object {
    $LogName = $_.LogName
    Get-WinEvent -LogName $LogName -MaxEvents 100 -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$($LogName.Replace('/','-'))-$Timestamp.csv" -NoTypeInformation
  }

Write-Host "Work Folders report data exported to $ReportRoot"
```

# Configure_Work_Folders_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify role service | `Get-WindowsFeature FS-SyncShareService` | Work Folders role service is installed |
| Verify SyncShare module | `Get-Command -Module SyncShare` | SyncShare cmdlets are available |
| Verify service | `Get-Service SyncShareSvc` | Sync Share service is visible |
| Verify sync server settings | `Get-SyncServerSetting` | Server settings are visible |
| Verify sync share | `Get-SyncShare -Name "DepartmentsWorkFolders"` | Sync share exists |
| Verify sync share path | `Test-Path "D:\WorkFolders\Departments"` | Sync root exists |
| Verify user sync group | `Get-ADGroupMember "GG_WorkFolders_Departments"` | Target user is listed |
| Verify DNS discovery | `Resolve-DnsName workfolders.corp.local` | Discovery name resolves |
| Verify AD discovery attribute | `Get-ADUser "<username>" -Properties msDS-SyncServerURL` | User has expected sync server URL if configured |
| Verify HTTPS binding | `netsh http show sslcert` | Certificate binding exists on port 443 |
| Verify HTTPS firewall | `Get-NetFirewallRule -DisplayName "Work Folders HTTPS"` | Rule is enabled |
| Verify client HTTPS reachability | `Test-NetConnection FS1.corp.local -Port 443` | TCP 443 succeeds |
| Verify GPO application | `gpresult /h C:\Temp\workfolders-gpresult.html` | Work Folders GPO is applied |
| Verify client local folder | `Test-Path "$env:USERPROFILE\Work Folders"` | Local Work Folders folder exists after setup |
| Verify test sync server side | `Get-ChildItem "D:\WorkFolders\Departments" -Recurse` | Client test file appears |
| Verify user status | `Get-SyncUserStatus` | User sync state is visible |
| Verify event logs | `Get-WinEvent -ListLog "*Sync*","*WorkFolders*"` | Relevant logs are discoverable |

# Configure_Work_Folders_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| Client test file created | `Remove-Item "$env:USERPROFILE\Work Folders\wf-client-test.txt" -Force` | Test file is removed from client sync folder |
| User AD discovery URL set | `Set-ADUser "<username>" -Remove @{"msDS-SyncServerURL"="https://FS1.corp.local"}` | User discovery URL is removed |
| DNS CNAME created | `Remove-DnsServerResourceRecord -ZoneName "corp.local" -RRType CName -Name "workfolders" -Force` | Internal discovery record is removed |
| Sync share created | `Remove-SyncShare -Name "DepartmentsWorkFolders" -Force` | Sync share definition is removed |
| Sync share disabled for testing | `Disable-SyncShare -Name "DepartmentsWorkFolders"` | Sync share stops accepting sync activity |
| Sync share disabled accidentally | `Enable-SyncShare -Name "DepartmentsWorkFolders"` | Sync share is re-enabled |
| Support email changed | `Set-SyncServerSetting -AdministratorEmail ""` | Support email is cleared |
| User added to allowed group | `Remove-ADGroupMember -Identity "GG_WorkFolders_Departments" -Members "<username>" -Confirm:$false` | User loses sync share entitlement |
| Group created | `Remove-ADGroup "GG_WorkFolders_Departments" -Confirm:$false` | Allowed group is removed |
| Admin group created | `Remove-ADGroup "GG_WorkFolders_Admins" -Confirm:$false` | Admin group is removed |
| GPO linked | Unlink `GPO_WorkFolders_Client_Config` from pilot OU | Policy no longer applies |
| GPO created | Delete or disable `GPO_WorkFolders_Client_Config` | Client policy is removed |
| HTTPS firewall rule created | `Remove-NetFirewallRule -DisplayName "Work Folders HTTPS"` | Custom firewall rule is removed |
| HTTPS cert binding created | `netsh http delete sslcert ipport=0.0.0.0:443` | HTTPS certificate binding is removed |
| Work Folders role installed | `Uninstall-WindowsFeature FS-SyncShareService` | Work Folders role service is removed |
| Sync folder created | `Remove-Item "D:\WorkFolders\Departments" -Recurse -Force` | Data folder is removed only after data is confirmed disposable |
| Report folder created | `Remove-Item "C:\WorkFolders-Reports" -Recurse -Force` | Report folder is removed |

# Configure_Work_Folders_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| SyncShare cmdlets missing | Work Folders role service not installed | `Get-WindowsFeature FS-SyncShareService` | Install `FS-SyncShareService` |
| `New-SyncShare` fails | Path missing, non-NTFS volume, nested sync share, or permission issue | `Test-Path`; `Get-Volume`; `Get-SyncShare` | Use valid NTFS local path |
| Client cannot discover Work Folders | DNS missing, GPO missing, or AD attribute missing | `Resolve-DnsName workfolders.<domain>`; `gpresult`; `Get-ADUser -Properties msDS-SyncServerURL` | Fix DNS, GPO, or discovery attribute |
| Client cannot connect | HTTPS blocked, certificate missing, binding missing, or firewall issue | `Test-NetConnection <server> -Port 443`; `netsh http show sslcert` | Bind cert and allow TCP 443 |
| Certificate warning | Client does not trust certificate or name mismatch | Check certificate subject, SAN, chain, and expiration | Install proper trusted certificate |
| User not authorized | User not in allowed sync group or stale token | `whoami /groups`; `Get-ADGroupMember` | Add user to group and have user sign out and back in |
| Sync share created but user folder not present | User has not completed first sync | `Get-SyncUserStatus`; client setup | Enroll client and create test file |
| Client policy not applying | GPO not linked, security filtering, wrong OU, or stale policy | `gpresult /h`; `gpupdate /force` | Correct GPO scope and filtering |
| Password policy prompt unwanted | RequirePasswordAutoLock enabled | `Get-SyncShare | Format-List RequirePasswordAutoLock` | Disable on sync share only if policy allows |
| Encryption prompt unwanted | RequireEncryption enabled | `Get-SyncShare | Format-List RequireEncryption` | Adjust sync share policy carefully |
| Sync is slow | Large initial dataset, client bandwidth, server load, antivirus, or FSRM scans | `Get-SyncUserStatus`; performance counters; events | Stage rollout and monitor load |
| File fails to sync | File too large, file locked, blocked type, path issue, or client problem | Client Work Folders UI, event logs, FSRM screens | Fix file issue or policy conflict |
| Events are hard to find | Log names vary or no activity yet | `Get-WinEvent -ListLog "*Sync*","*WorkFolders*"` | Query discovered logs after generating activity |
| Internet sync fails | Reverse proxy, public DNS, certificate, or authentication design incomplete | Public DNS, proxy logs, HTTPS test | Publish Work Folders correctly through approved reverse proxy |
| Users confuse Work Folders with SMB | Wrong user expectation | Check client path and server sync share | Train users on local sync folder usage |
| Admin cannot see user data | Exclusive user access is default behavior | `Get-SyncShare | Format-List *` | Use documented recovery process, do not break ACLs casually |

# Configure_Work_Folders_Related_Labs
| Lab                                                              | Relationship                                                              |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `01_Install_File_Server_Role_And_Management_Tools.md`            | Establishes the Windows file server baseline                              |
| `02_Create_And_Publish_SMB_Shares.md`                            | Contrasts SMB shares with Work Folders sync shares                        |
| `03_Configure_NTFS_And_Share_Permissions.md`                     | Work Folders still depends on NTFS storage and permissions                |
| `07_Configure_User_Home_Folders_And_Department_Shares.md`        | Work Folders can coexist with home folders and existing user data designs |
| `08_Configure_File_Screening_With_FSRM.md`                       | FSRM file screens can control what Work Folders stores or syncs           |
| `09_Configure_Storage_Quotas_With_FSRM.md`                       | FSRM quotas can limit user storage under sync shares                      |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | Storage reports help manage Work Folders user data                        |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md`            | Previous Versions can help recover server-side user data                  |
| `12_Backup_Restore_And_Export_File_Server_Config.md`             | Work Folders configuration and user data require backup planning          |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`   | Provides monitoring style for file server operations                      |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`          | Separates SMB access issues from Work Folders sync issues                 |
| `17_Configure_Data_Deduplication_For_File_Shares.md`             | Deduplication may apply to volumes that store Work Folders data           |
| `20_Configure_BranchCache_For_File_Services.md`                  | Contrasts WAN cache optimization with actual file sync                    |