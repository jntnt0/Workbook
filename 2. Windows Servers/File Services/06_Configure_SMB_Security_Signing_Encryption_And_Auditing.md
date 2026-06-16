
06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md
# Configure_SMB_Security_Signing_Encryption_And_Auditing

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Index
06_Configure_SMB_Security_Signing_Encryption_And_Auditing.md
Configure_SMB_Security_Signing_Encryption_And_Auditing
Configure_SMB_Security_Signing_Encryption_And_Auditing_Source_Basis
Configure_SMB_Security_Signing_Encryption_And_Auditing_Mental_Model
Configure_SMB_Security_Signing_Encryption_And_Auditing_Planning_Table
Configure_SMB_Security_Signing_Encryption_And_Auditing_Configuration_Checklist
Configure_SMB_Security_Signing_Encryption_And_Auditing_Precheck_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_SMB_Signing_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_SMB_Encryption_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_Audit_Policy_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_Folder_SACL_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_Client_Test_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_Event_Validation_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_Post_Change_Validation_Skeleton
Configure_SMB_Security_Signing_Encryption_And_Auditing_Verification_Commands
Configure_SMB_Security_Signing_Encryption_And_Auditing_Rollback
Configure_SMB_Security_Signing_Encryption_And_Auditing_Failure_Checks
Configure_SMB_Security_Signing_Encryption_And_Auditing_Related_Labs

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Get-SmbServerConfiguration | Reviewing SMB server signing, encryption, reject-unencrypted, SMB1, and related server settings |
| Microsoft Learn | Set-SmbServerConfiguration | Configuring SMB server signing and encryption posture |
| Microsoft Learn | Get-SmbClientConfiguration | Reviewing SMB client signing posture |
| Microsoft Learn | Set-SmbClientConfiguration | Configuring SMB client signing posture |
| Microsoft Learn | Get-SmbShare | Reviewing per-share encryption settings |
| Microsoft Learn | Set-SmbShare | Enabling or disabling SMB encryption on selected shares |
| Microsoft Learn | Get-SmbConnection | Validating SMB dialect, signing, encryption, and user connection state from clients |
| Microsoft Learn | auditpol | Configuring Windows Advanced Audit Policy for File Share and File System auditing |
| Microsoft Learn | Get-WinEvent | Reviewing SMB, Security, and file access event logs |
| Microsoft Learn | FileSystemAuditRule | Applying SACL audit rules to folders with PowerShell |
| Windows Server operational practice | SMB hardening | Require signing where compatibility allows, encrypt sensitive shares, disable SMB1, and audit access to important file paths |
| Windows Server operational practice | Evidence-based validation | Prove hardening with server settings, share settings, client connection state, and event logs |

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Mental_Model
| Concept | Operational Meaning |
|---|---|
| SMB signing | Adds integrity protection so SMB traffic cannot be silently tampered with in transit |
| SMB encryption | Encrypts SMB data in transit for selected shares or all shares |
| SMB dialect | SMB protocol version negotiated between client and server |
| RequireSecuritySignature | Requires signed SMB communication when set on server or client |
| EnableSecuritySignature | Allows SMB signing support |
| EncryptData | SMB share or server setting that enables SMB encryption |
| RejectUnencryptedAccess | Server setting that rejects clients that cannot use encryption for encrypted shares |
| SMB1 | Legacy SMB version that should remain disabled unless a strict legacy requirement exists |
| Share-level encryption | Encrypts selected shares such as `Departments` or `Home` |
| Server-level encryption | Makes encryption the default across SMB shares |
| File Share audit policy | Windows audit policy that records network share access events |
| Detailed File Share audit policy | Windows audit policy that records more detailed share access checks |
| File System audit policy | Windows audit policy that records object access when folder/file SACLs exist |
| SACL | System Access Control List that defines what file/folder access should be audited |
| DACL | Discretionary Access Control List that defines who is allowed or denied access |
| Security log | Event log where Object Access events such as 5140, 5145, and 4663 appear |
| SMBServer Operational log | SMB-specific operational log for server-side SMB activity |
| Event 5140 | Network share object was accessed |
| Event 5145 | Network share object was checked for requested access |
| Event 4663 | File system object access attempt occurred when SACL is configured |
| Compatibility risk | Older clients, appliances, scanners, and embedded systems may fail if signing or encryption is required |
| First rule | Hardening must be validated from both server settings and real client SMB connection state |

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Planning_Table
| Item | Example | Decision |
|---|---|---|
| File server name | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Management host | `ADMIN1` | `<management-host>` |
| Client test host | `CLIENT1` | `<client-test-host>` |
| Department share name | `Departments` | `<department-share-name>` |
| Home share name | `Home` | `<home-share-name>` |
| Public share name | `Public` | `<public-share-name>` |
| Apps share name | `Apps` | `<apps-share-name>` |
| Department local path | `E:\Shares\Departments` | `<department-root>` |
| Home local path | `E:\Shares\Home` | `<home-folder-root>` |
| Sensitive share list | `Departments`, `Home` | `<encrypted-share-list>` |
| Non-sensitive share list | `Public`, `Apps` | `<non-sensitive-share-list>` |
| SMB signing server stance | Require on file server | `<server-signing-plan>` |
| SMB signing client stance | Require on managed clients | `<client-signing-plan>` |
| SMB encryption stance | Per-share encryption | `<smb-encryption-plan>` |
| Reject unencrypted access | Enabled if all clients support encryption | `<reject-unencrypted-plan>` |
| SMB1 stance | Disabled | `<smb1-plan>` |
| File Share audit stance | Success and Failure | `<file-share-audit-plan>` |
| Detailed File Share audit stance | Success and Failure for testing or sensitive shares | `<detailed-file-share-audit-plan>` |
| File System audit stance | Success and Failure where SACLs exist | `<file-system-audit-plan>` |
| Audit target principal | `Authenticated Users` | `<audit-principal>` |
| Audit target path | `E:\Shares\Departments\Accounting` | `<audit-target-path>` |
| Audit rights | Read, Write, Delete | `<audit-rights>` |
| Evidence path | `C:\FileServices-Validation` | `<evidence-path>` |
| Client evidence path | `C:\FileServices-Validation-Client` | `<client-evidence-path>` |
| Rollback stance | Restore previous SMB and audit settings if compatibility breaks | `<rollback-plan>` |
| Next workbook | `07_Configure_User_Home_Folders_And_Department_Shares.md` | `<next-task>` |

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | File Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm File Server role is installed | File Server | `Get-WindowsFeature FS-FileServer` | File Server role shows Installed |
| 3 | Confirm SMB service is running | File Server | `Get-Service LanmanServer` | Server service is Running |
| 4 | Confirm SMB shares exist | File Server | `Get-SmbShare` | Managed shares are visible |
| 5 | Capture current SMB server configuration | File Server | `Get-SmbServerConfiguration` | Current SMB server posture is documented |
| 6 | Capture current SMB client configuration | File Server | `Get-SmbClientConfiguration` | Current SMB client posture is documented |
| 7 | Capture current share encryption state | File Server | `Get-SmbShare \| Select Name,Path,EncryptData,FolderEnumerationMode` | Per-share encryption state is documented |
| 8 | Confirm SMB1 server state | File Server | `Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol` | SMB1 state is known |
| 9 | Disable SMB1 if enabled | File Server | `Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart` | SMB1 is disabled or marked disabled |
| 10 | Enable SMB server signing support | File Server | `Set-SmbServerConfiguration -EnableSecuritySignature $true -Force` | Server supports SMB signing |
| 11 | Require SMB server signing if planned | File Server | `Set-SmbServerConfiguration -RequireSecuritySignature $true -Force` | Server requires signed SMB sessions |
| 12 | Enable SMB client signing support | File Server | `Set-SmbClientConfiguration -EnableSecuritySignature $true -Force` | Server acts as SMB client with signing support |
| 13 | Require SMB client signing if planned | File Server | `Set-SmbClientConfiguration -RequireSecuritySignature $true -Force` | Server requires signing when acting as SMB client |
| 14 | Enable SMB encryption on sensitive shares | File Server | `Set-SmbShare -Name "<share-name>" -EncryptData $true -Force` | Sensitive share requires encrypted SMB data |
| 15 | Leave non-sensitive shares unencrypted if planned | File Server | `Set-SmbShare -Name "<share-name>" -EncryptData $false -Force` | Non-sensitive share encryption remains disabled if intended |
| 16 | Reject unencrypted access if required | File Server | `Set-SmbServerConfiguration -RejectUnencryptedAccess $true -Force` | Server rejects unencrypted access where encryption is required |
| 17 | Capture final SMB server configuration | File Server | `Get-SmbServerConfiguration` | Signing and encryption posture is visible |
| 18 | Capture final share encryption state | File Server | `Get-SmbShare \| Select Name,EncryptData` | Encrypted share list matches plan |
| 19 | Enable File Share audit policy | File Server / GPO | `auditpol /set /subcategory:"File Share" /success:enable /failure:enable` | Share access auditing is enabled |
| 20 | Enable Detailed File Share audit policy | File Server / GPO | `auditpol /set /subcategory:"Detailed File Share" /success:enable /failure:enable` | Detailed share access auditing is enabled |
| 21 | Enable File System audit policy | File Server / GPO | `auditpol /set /subcategory:"File System" /success:enable /failure:enable` | File system object access auditing is enabled |
| 22 | Apply SACL to sensitive folder | File Server | Use folder SACL skeleton | File/folder access attempts can generate 4663 events |
| 23 | Test encrypted SMB connection from client | Client | `Get-SmbConnection` after accessing share | Client shows signed/encrypted connection where expected |
| 24 | Generate read/write test events | Client | Create, read, and delete test file on audited share | Security events are generated |
| 25 | Validate Security log events | File Server | `Get-WinEvent -FilterHashtable @{LogName="Security"; Id=5140,5145,4663}` | File share and file system events are visible |
| 26 | Validate SMBServer logs | File Server | `Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Operational"` | SMB operational events are visible |
| 27 | Export final SMB and audit evidence | File Server | Use post-change validation skeleton | Final configuration evidence is saved |
| 28 | Document final hardening state | Operator | `Record signing, encryption, audit policy, SACLs, client result, and event IDs` | SMB security workbook record is complete |

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture SMB security, share encryption, SMB1, audit policy, and event log state before changes.

$EvidencePath = "C:\FileServices-Validation"

$ManagedShares = @(
  "Departments",
  "Home",
  "Public",
  "Apps"
)

$ManagedPaths = @(
  "E:\Shares\Departments",
  "E:\Shares\Home",
  "E:\Shares\Public",
  "E:\Shares\Apps"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-smb-security-precheck-transcript.txt"

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups-before-smb-security.txt"

# Confirm server identity and File Server role.
hostname |
  Tee-Object "$EvidencePath\hostname-before-smb-security.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,WindowsProductName |
  Tee-Object "$EvidencePath\computer-state-before-smb-security.txt"

Get-WindowsFeature FS-FileServer |
  Tee-Object "$EvidencePath\fs-fileserver-feature-before-smb-security.txt"

# Confirm SMB services.
Get-Service LanmanServer,LanmanWorkstation |
  Tee-Object "$EvidencePath\smb-services-before-security.txt"

# Capture SMB server and client configuration.
Get-SmbServerConfiguration |
  Tee-Object "$EvidencePath\smb-server-configuration-before-security.txt"

Get-SmbClientConfiguration |
  Tee-Object "$EvidencePath\smb-client-configuration-before-security.txt"

# Capture SMB1 state.
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\smb1-optional-feature-before-security.txt"

Get-SmbServerConfiguration |
  Select-Object EnableSMB1Protocol,EnableSMB2Protocol,EnableSecuritySignature,RequireSecuritySignature,EncryptData,RejectUnencryptedAccess |
  Tee-Object "$EvidencePath\smb-core-security-settings-before.txt"

# Capture share settings and share permissions.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,Special |
  Tee-Object "$EvidencePath\smb-share-settings-before-security.txt"

Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,Description,FolderEnumerationMode,CachingMode,EncryptData,ContinuouslyAvailable,Special |
  Export-Csv "$EvidencePath\smb-share-settings-before-security.csv" -NoTypeInformation

foreach ($Share in $ManagedShares) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\smb-share-access-before-security-$Share.txt"
}

# Capture NTFS ACLs for target paths.
foreach ($Path in $ManagedPaths) {
  if (Test-Path $Path) {
    icacls $Path |
      Tee-Object "$EvidencePath\icacls-before-security-$($Path.Replace(':','').Replace('\','-')).txt"
  }
}

# Capture audit policy state.
auditpol /get /subcategory:"File Share" |
  Tee-Object "$EvidencePath\auditpol-file-share-before.txt"

auditpol /get /subcategory:"Detailed File Share" |
  Tee-Object "$EvidencePath\auditpol-detailed-file-share-before.txt"

auditpol /get /subcategory:"File System" |
  Tee-Object "$EvidencePath\auditpol-file-system-before.txt"

# Capture recent related events.
Get-WinEvent -LogName Security -MaxEvents 200 -ErrorAction SilentlyContinue |
  Where-Object { $_.Id -in 5140,5145,4663,4656,4658 } |
  Tee-Object "$EvidencePath\recent-security-file-events-before.txt"

Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\recent-smbserver-operational-before-security.txt"

Stop-Transcript
````

# Configure_SMB_Security_Signing_Encryption_And_Auditing_SMB_Signing_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: configure SMB signing posture.
# Warning: requiring signing can break or slow incompatible legacy clients. Validate with a test client first.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-smb-signing-transcript.txt"

# Capture current signing settings.
Get-SmbServerConfiguration |
  Select-Object EnableSecuritySignature,RequireSecuritySignature,EnableSMB1Protocol,EnableSMB2Protocol |
  Tee-Object "$EvidencePath\smb-server-signing-before.txt"

Get-SmbClientConfiguration |
  Select-Object EnableSecuritySignature,RequireSecuritySignature,EnableInsecureGuestLogons |
  Tee-Object "$EvidencePath\smb-client-signing-before.txt"

# Enable and require SMB signing for inbound SMB server sessions.
Set-SmbServerConfiguration `
  -EnableSecuritySignature $true `
  -RequireSecuritySignature $true `
  -Force

# Enable and require SMB signing when this server acts as an SMB client.
Set-SmbClientConfiguration `
  -EnableSecuritySignature $true `
  -RequireSecuritySignature $true `
  -Force

# Confirm final signing settings.
Get-SmbServerConfiguration |
  Select-Object EnableSecuritySignature,RequireSecuritySignature,EnableSMB1Protocol,EnableSMB2Protocol |
  Tee-Object "$EvidencePath\smb-server-signing-after.txt"

Get-SmbClientConfiguration |
  Select-Object EnableSecuritySignature,RequireSecuritySignature,EnableInsecureGuestLogons |
  Tee-Object "$EvidencePath\smb-client-signing-after.txt"

Stop-Transcript
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_SMB_Encryption_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: enable SMB encryption on sensitive shares and reject unencrypted access if required.

$EvidencePath = "C:\FileServices-Validation"

$EncryptedShares = @(
  "Departments",
  "Home"
)

$UnencryptedShares = @(
  "Public",
  "Apps"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-smb-encryption-transcript.txt"

# Capture current share encryption state.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,EncryptData,FolderEnumerationMode,Special |
  Tee-Object "$EvidencePath\smb-share-encryption-before.txt"

# Enable encryption on sensitive shares.
foreach ($Share in $EncryptedShares) {
  if (Get-SmbShare -Name $Share -ErrorAction SilentlyContinue) {
    Set-SmbShare `
      -Name $Share `
      -EncryptData $true `
      -Force
  }
}

# Leave non-sensitive shares unencrypted if that is the lab design.
foreach ($Share in $UnencryptedShares) {
  if (Get-SmbShare -Name $Share -ErrorAction SilentlyContinue) {
    Set-SmbShare `
      -Name $Share `
      -EncryptData $false `
      -Force
  }
}

# Reject unencrypted access where encryption is required.
# Validate client compatibility before enabling this in production.
Set-SmbServerConfiguration `
  -RejectUnencryptedAccess $true `
  -Force

# Optional: server-wide encryption default.
# Use only if all shares should require encryption by default.
# Set-SmbServerConfiguration -EncryptData $true -Force

# Confirm final encryption posture.
Get-SmbServerConfiguration |
  Select-Object EncryptData,RejectUnencryptedAccess |
  Tee-Object "$EvidencePath\smb-server-encryption-after.txt"

Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,EncryptData,FolderEnumerationMode,Special |
  Tee-Object "$EvidencePath\smb-share-encryption-after.txt"

Stop-Transcript
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Audit_Policy_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: enable audit policy required for SMB share and file system access events.
# In domain environments, prefer GPO for persistent audit policy.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-audit-policy-transcript.txt"

# Capture current audit policy.
auditpol /get /subcategory:"File Share" |
  Tee-Object "$EvidencePath\auditpol-file-share-before-enable.txt"

auditpol /get /subcategory:"Detailed File Share" |
  Tee-Object "$EvidencePath\auditpol-detailed-file-share-before-enable.txt"

auditpol /get /subcategory:"File System" |
  Tee-Object "$EvidencePath\auditpol-file-system-before-enable.txt"

# Enable file share auditing.
auditpol /set /subcategory:"File Share" /success:enable /failure:enable

# Enable detailed file share auditing.
auditpol /set /subcategory:"Detailed File Share" /success:enable /failure:enable

# Enable file system auditing.
# File System events require SACLs on folders/files.
auditpol /set /subcategory:"File System" /success:enable /failure:enable

# Optional: capture all Object Access audit settings.
auditpol /get /category:"Object Access" |
  Tee-Object "$EvidencePath\auditpol-object-access-after-enable.txt"

# Confirm final audit policy.
auditpol /get /subcategory:"File Share" |
  Tee-Object "$EvidencePath\auditpol-file-share-after-enable.txt"

auditpol /get /subcategory:"Detailed File Share" |
  Tee-Object "$EvidencePath\auditpol-detailed-file-share-after-enable.txt"

auditpol /get /subcategory:"File System" |
  Tee-Object "$EvidencePath\auditpol-file-system-after-enable.txt"

Stop-Transcript
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Folder_SACL_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: apply file system auditing SACLs to a sensitive folder.
# Audit policy alone does not create 4663 file system events. The folder also needs SACL entries.

$EvidencePath = "C:\FileServices-Validation"

$AuditPath = "E:\Shares\Departments\Accounting"
$AuditPrincipal = "Authenticated Users"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-folder-sacl-transcript.txt"

# Confirm target path exists.
if (-not (Test-Path $AuditPath)) {
  Write-Error "Audit path does not exist: $AuditPath"
  Stop-Transcript
  return
}

# Capture current ACL and audit rules before change.
Get-Acl -Path $AuditPath |
  Format-List |
  Tee-Object "$EvidencePath\acl-before-sacl-$($AuditPath.Replace(':','').Replace('\','-')).txt"

# Build audit rule.
$Rights = [System.Security.AccessControl.FileSystemRights]"ReadAndExecute,Write,Delete"
$InheritanceFlags = [System.Security.AccessControl.InheritanceFlags]"ContainerInherit,ObjectInherit"
$PropagationFlags = [System.Security.AccessControl.PropagationFlags]"None"
$AuditFlags = [System.Security.AccessControl.AuditFlags]"Success,Failure"

$AuditRule = New-Object System.Security.AccessControl.FileSystemAuditRule(
  $AuditPrincipal,
  $Rights,
  $InheritanceFlags,
  $PropagationFlags,
  $AuditFlags
)

# Apply SACL rule.
$Acl = Get-Acl -Path $AuditPath
$Acl.AddAuditRule($AuditRule)
Set-Acl -Path $AuditPath -AclObject $Acl

# Capture final ACL and audit rules.
Get-Acl -Path $AuditPath |
  Format-List |
  Tee-Object "$EvidencePath\acl-after-sacl-$($AuditPath.Replace(':','').Replace('\','-')).txt"

(Get-Acl -Path $AuditPath).Audit |
  Tee-Object "$EvidencePath\sacl-rules-after-$($AuditPath.Replace(':','').Replace('\','-')).txt"

Stop-Transcript
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Client_Test_Skeleton

```powershell
# Run from a Windows client after accessing the hardened shares.
# Purpose: validate SMB connection signing and encryption from the client side.

$FileServerName = "FS1"

$DepartmentShare = "\\$FileServerName\Departments"
$HomeShare = "\\$FileServerName\Home"
$PublicShare = "\\$FileServerName\Public"

$DepartmentTestPath = "\\$FileServerName\Departments\Accounting"
$DepartmentTestFile = Join-Path $DepartmentTestPath "smb-security-audit-test.txt"

$EvidencePath = "C:\FileServices-Validation-Client"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-client-smb-security-test-transcript.txt"

# Capture client identity.
hostname |
  Tee-Object "$EvidencePath\client-hostname-smb-security.txt"

whoami |
  Tee-Object "$EvidencePath\client-whoami-smb-security.txt"

whoami /groups |
  Tee-Object "$EvidencePath\client-whoami-groups-smb-security.txt"

# Confirm name resolution and SMB reachability.
Resolve-DnsName $FileServerName |
  Tee-Object "$EvidencePath\resolve-file-server-smb-security.txt"

Test-NetConnection $FileServerName -Port 445 |
  Tee-Object "$EvidencePath\test-smb-445-smb-security.txt"

# Access shares to create SMB connections.
Test-Path $DepartmentShare |
  Tee-Object "$EvidencePath\department-share-testpath.txt"

Test-Path $HomeShare |
  Tee-Object "$EvidencePath\home-share-testpath.txt"

Test-Path $PublicShare |
  Tee-Object "$EvidencePath\public-share-testpath.txt"

# Generate file access events if authorized.
if (Test-Path $DepartmentTestPath) {
  "SMB audit write test $(Get-Date)" |
    Out-File $DepartmentTestFile -Force

  Get-Content $DepartmentTestFile |
    Tee-Object "$EvidencePath\department-audit-test-readback.txt"

  Remove-Item $DepartmentTestFile -Force
}

# Validate client-side SMB connection details.
Get-SmbConnection |
  Where-Object { $_.ServerName -like "$FileServerName*" -or $_.ServerName -eq $FileServerName } |
  Select-Object ServerName,ShareName,Dialect,NumOpens,Signed,Encrypted,UserName |
  Tee-Object "$EvidencePath\client-smb-connections-after-access.txt"

Stop-Transcript
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Event_Validation_Skeleton

```powershell
# Run in elevated PowerShell on the file server after client access tests.
# Purpose: validate Security and SMBServer events created by share access and file access.

$EvidencePath = "C:\FileServices-Validation"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-event-validation-transcript.txt"

# Security events commonly relevant to SMB/file access:
# 5140: network share object accessed
# 5145: network share object checked for desired access
# 4663: object access attempt, requires File System audit policy and SACL
# 4656: handle requested, may appear with object access auditing
# 4658: handle closed

$SecurityEventIds = @(5140,5145,4663,4656,4658)

Get-WinEvent `
  -FilterHashtable @{
    LogName = "Security"
    Id = $SecurityEventIds
    StartTime = (Get-Date).AddHours(-4)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,Message |
  Tee-Object "$EvidencePath\security-file-share-events-last-4-hours.txt"

# Export compact event evidence to CSV.
Get-WinEvent `
  -FilterHashtable @{
    LogName = "Security"
    Id = $SecurityEventIds
    StartTime = (Get-Date).AddHours(-4)
  } `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,MachineName,Message |
  Export-Csv "$EvidencePath\security-file-share-events-last-4-hours.csv" -NoTypeInformation

# SMBServer Operational log.
Get-WinEvent `
  -LogName "Microsoft-Windows-SmbServer/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,Message |
  Tee-Object "$EvidencePath\smbserver-operational-events-after-security.txt"

# SMBServer Audit log if available.
Get-WinEvent `
  -LogName "Microsoft-Windows-SMBServer/Audit" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,Message |
  Tee-Object "$EvidencePath\smbserver-audit-events-after-security.txt"

Stop-Transcript
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Post_Change_Validation_Skeleton

```powershell
# Run in elevated PowerShell on the file server.
# Purpose: capture final SMB hardening, share encryption, audit policy, SACL, sessions, and event evidence.

$EvidencePath = "C:\FileServices-Validation"

$ManagedShares = @(
  "Departments",
  "Home",
  "Public",
  "Apps"
)

$AuditPath = "E:\Shares\Departments\Accounting"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-post-change-smb-security-validation-transcript.txt"

# Final SMB server security configuration.
Get-SmbServerConfiguration |
  Tee-Object "$EvidencePath\smb-server-configuration-final-security.txt"

Get-SmbServerConfiguration |
  Select-Object EnableSMB1Protocol,EnableSMB2Protocol,EnableSecuritySignature,RequireSecuritySignature,EncryptData,RejectUnencryptedAccess,AuditSmb1Access |
  Tee-Object "$EvidencePath\smb-core-security-settings-final.txt"

# Final SMB client security configuration.
Get-SmbClientConfiguration |
  Tee-Object "$EvidencePath\smb-client-configuration-final-security.txt"

Get-SmbClientConfiguration |
  Select-Object EnableSecuritySignature,RequireSecuritySignature,EnableInsecureGuestLogons |
  Tee-Object "$EvidencePath\smb-client-core-security-settings-final.txt"

# SMB1 optional feature state.
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\smb1-optional-feature-final.txt"

# Final share encryption and visibility state.
Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,CachingMode,ContinuouslyAvailable,Special |
  Tee-Object "$EvidencePath\smb-share-security-final.txt"

Get-SmbShare |
  Sort-Object Name |
  Select-Object Name,Path,FolderEnumerationMode,EncryptData,CachingMode,ContinuouslyAvailable,Special |
  Export-Csv "$EvidencePath\smb-share-security-final.csv" -NoTypeInformation

# Final share permissions.
foreach ($Share in $ManagedShares) {
  Get-SmbShareAccess -Name $Share -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\smb-share-access-final-security-$Share.txt"
}

# Final audit policy.
auditpol /get /category:"Object Access" |
  Tee-Object "$EvidencePath\auditpol-object-access-final.txt"

auditpol /get /subcategory:"File Share" |
  Tee-Object "$EvidencePath\auditpol-file-share-final.txt"

auditpol /get /subcategory:"Detailed File Share" |
  Tee-Object "$EvidencePath\auditpol-detailed-file-share-final.txt"

auditpol /get /subcategory:"File System" |
  Tee-Object "$EvidencePath\auditpol-file-system-final.txt"

# Final SACL evidence.
if (Test-Path $AuditPath) {
  (Get-Acl -Path $AuditPath).Audit |
    Tee-Object "$EvidencePath\sacl-rules-final-$($AuditPath.Replace(':','').Replace('\','-')).txt"
}

# Active SMB sessions, open files, and connections.
Get-SmbSession |
  Tee-Object "$EvidencePath\smb-sessions-final-security.txt"

Get-SmbOpenFile |
  Tee-Object "$EvidencePath\smb-open-files-final-security.txt"

Get-SmbConnection -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\smb-connections-final-security.txt"

# Recent events.
Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = @(5140,5145,4663,4656,4658)
  StartTime = (Get-Date).AddHours(-4)
} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,Message |
  Tee-Object "$EvidencePath\security-events-final-security.txt"

Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,ProviderName,Message |
  Tee-Object "$EvidencePath\smbserver-operational-final-security.txt"

Stop-Transcript
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Verification_Commands

```powershell
# SMB role and services
Get-WindowsFeature FS-FileServer
Get-Service LanmanServer
Get-Service LanmanWorkstation

# SMB server security posture
Get-SmbServerConfiguration
Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol,EnableSMB2Protocol,EnableSecuritySignature,RequireSecuritySignature,EncryptData,RejectUnencryptedAccess,AuditSmb1Access

# SMB client security posture
Get-SmbClientConfiguration
Get-SmbClientConfiguration | Select-Object EnableSecuritySignature,RequireSecuritySignature,EnableInsecureGuestLogons

# SMB1 optional feature
Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# Disable SMB1 if required
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart

# Configure SMB signing
Set-SmbServerConfiguration -EnableSecuritySignature $true -RequireSecuritySignature $true -Force
Set-SmbClientConfiguration -EnableSecuritySignature $true -RequireSecuritySignature $true -Force

# Share encryption state
Get-SmbShare | Select-Object Name,Path,EncryptData,FolderEnumerationMode
Get-SmbShare -Name "<share-name>" | Select-Object Name,Path,EncryptData

# Enable or disable encryption on a share
Set-SmbShare -Name "<share-name>" -EncryptData $true -Force
Set-SmbShare -Name "<share-name>" -EncryptData $false -Force

# Reject unencrypted access
Set-SmbServerConfiguration -RejectUnencryptedAccess $true -Force
Set-SmbServerConfiguration -RejectUnencryptedAccess $false -Force

# Share permissions and NTFS permissions
Get-SmbShareAccess -Name "<share-name>"
icacls "<folder-path>"

# Audit policy
auditpol /get /category:"Object Access"
auditpol /get /subcategory:"File Share"
auditpol /get /subcategory:"Detailed File Share"
auditpol /get /subcategory:"File System"

auditpol /set /subcategory:"File Share" /success:enable /failure:enable
auditpol /set /subcategory:"Detailed File Share" /success:enable /failure:enable
auditpol /set /subcategory:"File System" /success:enable /failure:enable

# SACL check
(Get-Acl -Path "<audit-target-path>").Audit

# Client-side SMB validation
Test-NetConnection "<file-server-name>" -Port 445
Test-Path "\\<file-server-name>\<share-name>"
Get-SmbConnection | Select-Object ServerName,ShareName,Dialect,Signed,Encrypted,UserName

# Security log event validation
Get-WinEvent -FilterHashtable @{LogName="Security"; Id=5140,5145,4663; StartTime=(Get-Date).AddHours(-4)}

# SMB log event validation
Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Operational" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Audit" -MaxEvents 100

# Evidence
Test-Path "<evidence-path>"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*security*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*audit*"
Get-ChildItem "<evidence-path>" | Where-Object Name -like "*smb*"
```

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Rollback

|Step|Task|Device|PowerShell / Command|Expected Result|
|--:|---|---|---|---|
|1|Export current SMB security state|File Server|`Get-SmbServerConfiguration > "<evidence-path>\smb-server-before-rollback.txt"`|Current server posture is saved|
|2|Export current share encryption state|File Server|`Get-SmbShare \| Select Name,EncryptData \| Export-Csv "<evidence-path>\share-encryption-before-rollback.csv" -NoTypeInformation`|Current share encryption state is saved|
|3|Relax server signing requirement if compatibility breaks|File Server|`Set-SmbServerConfiguration -RequireSecuritySignature $false -Force`|Server no longer requires signed sessions|
|4|Keep signing support enabled while relaxing requirement|File Server|`Set-SmbServerConfiguration -EnableSecuritySignature $true -Force`|Server can still sign when negotiated|
|5|Relax client signing requirement if required|File Server|`Set-SmbClientConfiguration -RequireSecuritySignature $false -Force`|Server as client no longer requires signing|
|6|Disable share encryption on a specific share|File Server|`Set-SmbShare -Name "<share-name>" -EncryptData $false -Force`|Share no longer requires encryption|
|7|Allow unencrypted access if compatibility requires rollback|File Server|`Set-SmbServerConfiguration -RejectUnencryptedAccess $false -Force`|Unencrypted access is no longer rejected|
|8|Do not re-enable SMB1 unless explicitly required|File Server|`Get-WindowsOptionalFeature -Online -FeatureName SMB1Protocol`|SMB1 remains disabled by default|
|9|Disable File Share audit policy if lab rollback requires it|File Server / GPO|`auditpol /set /subcategory:"File Share" /success:disable /failure:disable`|File Share auditing disabled|
|10|Disable Detailed File Share audit policy if lab rollback requires it|File Server / GPO|`auditpol /set /subcategory:"Detailed File Share" /success:disable /failure:disable`|Detailed File Share auditing disabled|
|11|Disable File System audit policy if lab rollback requires it|File Server / GPO|`auditpol /set /subcategory:"File System" /success:disable /failure:disable`|File System auditing disabled|
|12|Remove incorrect folder SACL rule|File Server|Use `Get-Acl`, remove matching audit rule, then `Set-Acl`|Incorrect audit rule is removed|
|13|Validate client access after rollback|Client|`Test-Path "\\<file-server-name>\<share-name>"`; `Get-SmbConnection`|Client access works under rollback posture|
|14|Document rollback result|Operator|`Record changed signing, encryption, audit, and event state`|Rollback record is complete|

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Failure_Checks

|Symptom|Likely Cause|Check|Corrective Action|
|---|---|---|---|
|Client cannot connect after signing required|Client does not support or allow required signing|`Get-SmbConnection` on client if connection exists|Relax signing requirement or update client policy|
|Client cannot connect after encryption enabled|Client does not support SMB encryption|`Get-SmbShare -Name <share>`; client OS version|Disable encryption for that share or upgrade client|
|Access fails only on encrypted shares|`RejectUnencryptedAccess` blocks client|`Get-SmbServerConfiguration \| Select RejectUnencryptedAccess`|Fix client encryption support or set rollback value|
|`Set-SmbServerConfiguration` fails|PowerShell not elevated|`whoami /groups`|Reopen PowerShell as Administrator|
|`Set-SmbShare` fails|Share name typo or share missing|`Get-SmbShare`|Use exact share name|
|SMB1-dependent device stops working|SMB1 disabled or signing/encryption not supported|Device logs and server SMB settings|Replace/update device. Avoid re-enabling SMB1 unless approved|
|Signing appears enabled but client says unsigned|Client connected before setting changed|`Get-SmbSession`; `Get-SmbConnection`|Disconnect session and reconnect|
|Encryption appears enabled but client says not encrypted|Client is connected to different share or cached session|`Get-SmbConnection`; UNC path check|Reconnect to target encrypted share|
|Public share unexpectedly encrypted|Share included in encrypted share list|`Get-SmbShare -Name Public \| Select EncryptData`|Set Public `EncryptData` to `$false` if planned|
|No 5140 events appear|File Share audit policy not enabled|`auditpol /get /subcategory:"File Share"`|Enable File Share auditing|
|No 5145 events appear|Detailed File Share audit policy not enabled|`auditpol /get /subcategory:"Detailed File Share"`|Enable Detailed File Share auditing|
|No 4663 events appear|Folder SACL missing or File System audit disabled|`(Get-Acl <path>).Audit`; `auditpol /get /subcategory:"File System"`|Add SACL and enable File System auditing|
|Too many audit events|SACL too broad or Detailed File Share enabled broadly|Security log volume and audit settings|Narrow SACL principal/path/rights or adjust policy|
|Security log overwrites quickly|Log size too small for audit volume|Event Viewer Security log properties|Increase log size and forward events|
|Audit policy resets after reboot or GPUpdate|Domain GPO overrides local audit policy|`gpresult /h report.html`; `auditpol /get /category:*`|Configure audit policy in GPO|
|Access denied after hardening|Share permission, NTFS, signing, or encryption issue|`Get-SmbShareAccess`; `icacls`; `Get-SmbConnection`|Isolate one layer at a time|
|`Get-WinEvent` cannot read Security log|Not elevated or insufficient rights|Run elevated PowerShell|Use admin context or Event Log Readers where appropriate|
|SMBServer/Audit log missing|Log unavailable or disabled on OS|Event Viewer Applications and Services Logs|Use Security log events and SMB Operational log|
|Evidence missing|Validation skeleton not run|`Test-Path "<evidence-path>"`|Re-run precheck or post-change validation skeleton|

# Configure_SMB_Security_Signing_Encryption_And_Auditing_Related_Labs

|Related Lab|Relationship|
|---|---|
|`00_File_Services_Index.md`|Defines where SMB security fits in the full File Services sequence|
|`01_Install_File_Server_Role_And_Management_Tools.md`|Provides the File Server role and SMB PowerShell tooling|
|`02_Prepare_File_Server_Storage_Volumes_And_Folders.md`|Provides local paths used by SMB shares and SACLs|
|`03_Configure_NTFS_Permissions_And_ACL_Inheritance.md`|Provides NTFS DACLs that combine with SMB access|
|`04_Create_SMB_Shares_And_Share_Permissions.md`|Creates the SMB shares hardened in this workbook|
|`05_Configure_Access_Based_Enumeration_And_Share_Visibility.md`|Configures visibility before deeper SMB security and audit validation|
|`07_Configure_User_Home_Folders_And_Department_Shares.md`|Uses encrypted and audited shares for real user and department data patterns|
|`12_Backup_Restore_And_Export_File_Server_Config.md`|Exports share security, audit settings, and evidence for recovery|
|`15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md`|Uses SMB connection, session, and event data for operations|
|`16_Troubleshoot_File_Share_Access_And_SMB_Failures.md`|Troubleshoots failures caused by signing, encryption, audit, share, or NTFS mismatch|
|`23_Configure_Storage_Replica_For_File_Server_Disaster_Recovery.md`|Requires SMB and storage security planning for replicated file services|
|`24_Configure_Scale_Out_File_Server_And_Continuously_Available_Shares.md`|Applies SMB security concepts to clustered file shares|
|`25_Migrate_File_Server_Data_Shares_And_Permissions.md`|Preserves SMB share security and audit posture during migration|