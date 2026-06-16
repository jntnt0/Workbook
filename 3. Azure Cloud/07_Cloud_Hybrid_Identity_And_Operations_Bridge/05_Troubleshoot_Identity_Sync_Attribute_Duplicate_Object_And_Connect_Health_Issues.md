# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Index

05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Source_Basis  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Mental_Model  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Planning_Table  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Configuration_Checklist  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Triage_Flow  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Local_ADSync_Health_Skeleton  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Connect_Health_Portal_Review_Skeleton  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Duplicate_Attribute_Triage_Skeleton  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Duplicate_Object_And_Matching_Triage_Skeleton  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Object_Sync_Error_Export_Skeleton  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Remediation_Skeleton  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Post_Remediation_Validation_Skeleton  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Verification_Commands  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Rollback  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Failure_Checks  
05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Related_Labs  

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Entra Docs | Microsoft Entra Connect Sync troubleshooting | Sync service, scheduler, connector, export, import, and synchronization error triage |
| Microsoft Entra Docs | Microsoft Entra Connect Health | Sync health alerts, sync latency, object-level errors, and agent health |
| Microsoft Entra Docs | Duplicate attribute resiliency | Duplicate UPN, proxyAddress, mail, and conflicting cloud object troubleshooting |
| Microsoft Entra Docs | Microsoft Graph users and directory objects | Cloud-side object validation, sync flags, and provisioning error inspection |
| Microsoft 365 Docs | Prepare for directory synchronization | Duplicate and invalid AD DS attribute cleanup |
| Windows Server AD DS | ActiveDirectory PowerShell module | Source object inspection and remediation |
| ADSync PowerShell module | Get-ADSyncScheduler, Start-ADSyncSyncCycle, Get-ADSyncConnector | Local sync engine validation and cycle control |
| Synchronization Service Manager | `miisclient.exe` | Connector space, run history, pending exports, and object-level sync operations |
| Windows Event Viewer | Application and Directory Synchronization logs | Local sync service and connector event triage |
| Operational practice | Triage first, disable scheduler if blast radius exists, fix source of authority, validate delta sync | Prevents making identity conflicts worse |

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Sync error | A failure during import, sync, or export between AD DS and Microsoft Entra ID |
| Attribute error | Sync problem caused by bad, duplicate, invalid, or unsupported attribute values |
| Duplicate attribute | Same value exists on more than one object where Microsoft Entra ID expects uniqueness |
| Duplicate object | On-prem object and cloud object represent the same person or entity but are not correctly joined |
| Soft match | Matching an on-prem object to a cloud object based on values such as UPN or primary SMTP |
| Hard match | Matching an on-prem object to a cloud object using the immutable source anchor |
| Source anchor | Stable identifier linking the on-prem object to the cloud object |
| ImmutableId | Cloud-side value used by older tooling to represent source anchor relationship |
| OnPremisesSyncEnabled | Cloud-side indicator that the object is synchronized from on-prem AD DS |
| Connector space | Local sync engine database view of objects from each connected directory |
| Metaverse | Local sync engine joined object space between connectors |
| Import | Sync engine reads source directory changes |
| Synchronization | Sync engine processes joins, projections, rules, and attribute flows |
| Export | Sync engine writes changes to target directory |
| Pending export | Change waiting to be written to Microsoft Entra ID or AD DS |
| Export error | Failure writing an update to the target directory |
| Connect Health | Cloud monitoring for Microsoft Entra Connect Sync and related identity components |
| Provisioning error | Cloud-side object provisioning failure visible through portal or Graph |
| First rule | Do not delete cloud users to fix sync until matching and source anchor are understood |
| Blunt rule | Duplicate proxyAddresses and UPNs are usually AD hygiene failures, not a Microsoft cloud mystery |

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Sync method | Entra Connect Sync / Cloud Sync | `<sync-method>` |
| Sync server | `SYNC01.corp.local` | `<sync-server>` |
| Cloud Sync agent server | `PROVAGENT01.corp.local` | `<cloud-sync-agent-server>` |
| AD forest | `corp.local` | `<ad-forest>` |
| AD domain | `corp.local` | `<ad-domain>` |
| Verified domain | `contoso.com` | `<verified-domain>` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Tenant domain | `contoso.onmicrosoft.com` | `<tenant-domain>` |
| Affected user | `jdoe@contoso.com` | `<affected-upn>` |
| Affected object type | User / Group / Contact | `<object-type>` |
| Suspected attribute | UPN / mail / proxyAddresses / mailNickname / targetAddress | `<suspected-attribute>` |
| Error source | Connect Health / miisclient / Graph / Event Viewer | `<error-source>` |
| Error type | Duplicate attribute / export error / matching issue / agent health | `<error-type>` |
| Pilot OU | `OU=PilotUsers,OU=Users,DC=corp,DC=local` | `<pilot-ou>` |
| Sync scope | Pilot OU / production OU / selected groups | `<sync-scope>` |
| Evidence folder | `C:\HybridIdentity-Troubleshooting` | `<evidence-path>` |
| Rollback CSV | `C:\HybridIdentity-Troubleshooting\rollback-sync-attribute-map.csv` | `<rollback-csv>` |
| Scheduler state | Enabled / Disabled | `<scheduler-state>` |
| Emergency action | Disable scheduler / enable staging mode / stop agent | `<emergency-action>` |
| Change ticket | `CHG000000` | `<change-ticket>` |
| Remediation owner | IAM / AD / Messaging / Cloud Admin | `<owner>` |

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | Sync Server / Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\HybridIdentity-Troubleshooting` | Evidence folder exists |
| 2 | Confirm affected object and symptom | Operator | `Record UPN, object type, error message, time, and source` | Troubleshooting target is clear |
| 3 | Confirm sync method | Operator | `Review deployment notes from task 04` | Entra Connect Sync or Cloud Sync path is known |
| 4 | If blast radius exists, disable scheduler | Sync Server | `Set-ADSyncScheduler -SyncCycleEnabled $false` | Automatic sync stops |
| 5 | Confirm local ADSync service | Sync Server | `Get-Service ADSync` | Service state is known |
| 6 | Confirm ADSync scheduler | Sync Server | `Get-ADSyncScheduler` | Scheduler and sync cycle state are visible |
| 7 | Export ADSync connectors | Sync Server | `Get-ADSyncConnector \| Export-Csv C:\HybridIdentity-Troubleshooting\adsync-connectors.csv -NoTypeInformation` | Connector inventory exists |
| 8 | Open Synchronization Service Manager | Sync Server | `miisclient.exe` | Run history and connector errors are visible |
| 9 | Review recent sync runs | Sync Server | `Operations tab in miisclient.exe` | Failed import, sync, or export stage is identified |
| 10 | Export local sync events | Sync Server | `Get-WinEvent -LogName Application -MaxEvents 500` | Local sync event evidence exists |
| 11 | Review Connect Health | Entra Admin Center | `Identity > Monitoring & health > Connect Health` | Health alerts and object errors are reviewed |
| 12 | Review provisioning logs if Cloud Sync | Entra Admin Center | `Identity > Monitoring & health > Provisioning logs` | Cloud Sync errors are reviewed |
| 13 | Connect to Microsoft Graph | Admin Workstation | `Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All"` | Cloud-side validation session connects |
| 14 | Query affected cloud object | Admin Workstation | `Get-MgUser -UserId "<affected-upn>" -Property *` | Cloud object properties are visible |
| 15 | Query affected AD object | Admin Workstation / DC | `Get-ADUser -Filter "UserPrincipalName -eq '<affected-upn>'" -Properties *` | Source AD object properties are visible |
| 16 | Export rollback values before remediation | Admin Workstation / DC | `Run Duplicate_Attribute_Triage_Skeleton` | Original attribute values are saved |
| 17 | Check duplicate UPN values | Admin Workstation / DC | `Group-Object UserPrincipalName` | Duplicate UPNs are identified |
| 18 | Check duplicate mail values | Admin Workstation / DC | `Group-Object mail` | Duplicate mail values are identified |
| 19 | Check duplicate proxyAddresses values | Admin Workstation / DC | `Run duplicate proxy report` | Duplicate aliases are identified |
| 20 | Check duplicate mailNickname values | Admin Workstation / DC | `Group-Object mailNickname` | Duplicate aliases are identified |
| 21 | Compare cloud object sync flags | Admin Workstation | `Get-MgUser -Property OnPremisesSyncEnabled,OnPremisesImmutableId` | Cloud object match state is known |
| 22 | Determine source of authority | Operator | `Cloud-only or synced from AD DS` | Fix location is known |
| 23 | Remediate duplicate attributes in AD DS first | Admin Workstation / DC | `Set-ADUser`, `Set-ADObject`, or IdFix | Source duplicate is corrected |
| 24 | Remediate cloud-only conflict only after approval | Admin Workstation | `Update or remove conflicting cloud object value` | Cloud conflict is resolved safely |
| 25 | Run delta sync | Sync Server | `Start-ADSyncSyncCycle -PolicyType Delta` | Sync cycle starts |
| 26 | Validate run history | Sync Server | `miisclient.exe` | No blocking export errors remain |
| 27 | Validate affected object in cloud | Admin Workstation | `Get-MgUser -UserId "<affected-upn>"` | Cloud object reflects expected state |
| 28 | Re-enable scheduler if disabled | Sync Server | `Set-ADSyncScheduler -SyncCycleEnabled $true` | Automatic sync resumes |
| 29 | Recheck Connect Health | Entra Admin Center | `Review health and alerts` | Health state is clean or exceptions documented |
| 30 | Document root cause and fix | Operator | `Record cause, source object, changed attributes, sync cycle, validation result` | Troubleshooting evidence is complete |

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Triage_Flow

```text
1. Identify the exact failing object.
   - User, group, contact, or device
   - UPN, ObjectId, DistinguishedName, or proxy address
   - Error text and source

2. Identify the failing stage.
   - AD import
   - Microsoft Entra import
   - Synchronization
   - Microsoft Entra export
   - AD export
   - Cloud Sync provisioning

3. Decide whether to freeze sync.
   - Disable scheduler if many objects are affected.
   - Use staging mode if exports need to be blocked.
   - Disable Cloud Sync configuration if provisioning is causing repeated bad writes.

4. Determine source of authority.
   - If object is synced, fix AD DS for synced attributes.
   - If object is cloud-only, fix the cloud object or matching conflict.
   - Do not edit cloud values that are mastered on-prem unless the object is cloud-only.

5. Classify the problem.
   - Duplicate UPN
   - Duplicate mail
   - Duplicate proxyAddresses
   - Duplicate mailNickname
   - Invalid characters
   - Nonroutable UPN
   - Soft match conflict
   - Hard match/source anchor conflict
   - Connector or agent health issue
   - Permission issue
   - Accidental delete protection
   - Sync scope issue

6. Export before-state.
   - AD object properties
   - Cloud object properties
   - Connector/run history evidence
   - Health alert evidence

7. Fix the source.
   - Correct AD attributes first for synced objects.
   - Correct cloud conflicts only when they are cloud-only or blocking match.
   - Use pilot object validation before broad remediation.

8. Run delta sync or provisioning cycle.

9. Validate.
   - AD source object
   - Sync run history
   - Connect Health or provisioning logs
   - Cloud object properties
   - User sign-in, license, and access impact if relevant

10. Re-enable scheduler only after validation.
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Local_ADSync_Health_Skeleton

```powershell
# Run on the Entra Connect Sync server.
# Purpose: collect local ADSync health, scheduler, connector, service, and event evidence.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-local-adsync-health-transcript.txt"

# Basic server context.
hostname | Out-File "$EvidencePath\sync-server-hostname.txt"
whoami /all | Out-File "$EvidencePath\sync-server-whoami-all.txt"

Get-ComputerInfo |
  Select-Object WindowsProductName,WindowsVersion,OsVersion,OsBuildNumber,CsDomain,CsPartOfDomain |
  Format-List * |
  Out-File "$EvidencePath\sync-server-computer-info.txt"

# Service state.
Get-Service ADSync -ErrorAction SilentlyContinue |
  Select-Object Name,DisplayName,Status,StartType |
  Tee-Object "$EvidencePath\adsync-service-status.txt"

# Import ADSync module.
$ADSyncModulePath = "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1"

if (Test-Path $ADSyncModulePath) {
  Import-Module $ADSyncModulePath
}
else {
  Import-Module ADSync -ErrorAction SilentlyContinue
}

# Scheduler.
Get-ADSyncScheduler |
  Format-List * |
  Tee-Object "$EvidencePath\adsync-scheduler.txt"

# Connectors.
Get-ADSyncConnector |
  Select-Object Name,Type,SubType,Identifier |
  Export-Csv "$EvidencePath\adsync-connectors.csv" -NoTypeInformation

# Optional: disable scheduler if many bad exports are suspected.
# Set-ADSyncScheduler -SyncCycleEnabled $false

# Recent events.
Get-WinEvent -LogName Application -MaxEvents 1000 |
  Where-Object {
    $_.ProviderName -like "*ADSync*" -or
    $_.ProviderName -like "*Directory Synchronization*" -or
    $_.Message -like "*ADSync*" -or
    $_.Message -like "*sync*" -or
    $_.Message -like "*export*" -or
    $_.Message -like "*duplicate*"
  } |
  Select-Object TimeCreated,ProviderName,Id,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\adsync-application-events.csv" -NoTypeInformation

# Launch Synchronization Service Manager manually for connector/run details.
# C:\Program Files\Microsoft Azure AD Sync\UIShell\miisclient.exe

Stop-Transcript
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Connect_Health_Portal_Review_Skeleton

```text
# Microsoft Entra Connect Health review skeleton.
# This is portal-driven because Connect Health data is primarily reviewed in Microsoft Entra admin center.

1. Sign in to Microsoft Entra admin center.
2. Go to:
   Identity > Monitoring & health > Connect Health
3. Select the Microsoft Entra Connect Sync service.
4. Confirm the expected sync server appears.
5. Review service status:
   - Healthy
   - Warning
   - Critical
   - Not reporting
6. Review active alerts.
7. Record:
   - alert name
   - severity
   - affected server
   - first detected time
   - last detected time
   - suggested remediation
8. Review sync latency.
9. Review object-level synchronization errors.
10. Classify each error:
    - duplicate attribute
    - data validation failure
    - large attribute
    - federated domain issue
    - permissions issue
    - export failure
11. Open affected object details.
12. Record:
    - object display name
    - source anchor or immutable ID if visible
    - UPN
    - object type
    - failing attribute
    - error text
13. Export or screenshot evidence into:
    C:\HybridIdentity-Troubleshooting\ConnectHealth
14. If Cloud Sync is used, also review:
    Identity > Monitoring & health > Provisioning logs
15. Confirm whether errors are increasing, stable, or resolved.
16. Do not mark resolved until a new sync cycle or provisioning cycle validates cleanly.
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Duplicate_Attribute_Triage_Skeleton

```powershell
# Run from a DC or admin workstation with RSAT AD tools.
# Purpose: find duplicate UPN, mail, proxyAddresses, mailNickname, and targetAddress values.
# This does not remediate. It exports evidence and rollback values.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"
$SearchBase = "OU=Users,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-duplicate-attribute-triage-transcript.txt"

Import-Module ActiveDirectory

# User inventory.
$Users = Get-ADUser -SearchBase $SearchBase -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,Enabled,Description,ObjectGUID

$Users |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    ObjectGUID,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    targetAddress,
    Description,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\users-identity-before-triage.csv" -NoTypeInformation

# Rollback map.
$Users |
  Select-Object `
    SamAccountName,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    targetAddress,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\rollback-sync-attribute-map.csv" -NoTypeInformation

# Duplicate UPN.
$Users |
  Where-Object { $_.UserPrincipalName } |
  Group-Object UserPrincipalName |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicate-userPrincipalName-summary.csv" -NoTypeInformation

# Duplicate mail.
$Users |
  Where-Object { $_.mail } |
  Group-Object mail |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicate-mail-summary.csv" -NoTypeInformation

# Duplicate mailNickname.
$Users |
  Where-Object { $_.mailNickname } |
  Group-Object mailNickname |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicate-mailNickname-summary.csv" -NoTypeInformation

# Duplicate targetAddress.
$Users |
  Where-Object { $_.targetAddress } |
  Group-Object targetAddress |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicate-targetAddress-summary.csv" -NoTypeInformation

# Flatten users, groups, and contacts for duplicate proxyAddresses.
$ProxyRows = @()

foreach ($User in $Users) {
  foreach ($Proxy in $User.proxyAddresses) {
    $ProxyRows += [PSCustomObject]@{
      ObjectType = "User"
      Name = $User.Name
      SamAccountName = $User.SamAccountName
      DistinguishedName = $User.DistinguishedName
      ProxyAddressNormalized = $Proxy.ToLowerInvariant()
      ProxyAddressOriginal = $Proxy
    }
  }
}

$Groups = Get-ADGroup -Filter * -Properties proxyAddresses,mail,mailNickname
foreach ($Group in $Groups) {
  foreach ($Proxy in $Group.proxyAddresses) {
    $ProxyRows += [PSCustomObject]@{
      ObjectType = "Group"
      Name = $Group.Name
      SamAccountName = $Group.SamAccountName
      DistinguishedName = $Group.DistinguishedName
      ProxyAddressNormalized = $Proxy.ToLowerInvariant()
      ProxyAddressOriginal = $Proxy
    }
  }
}

$Contacts = Get-ADObject -LDAPFilter "(objectClass=contact)" -Properties proxyAddresses,mail,mailNickname
foreach ($Contact in $Contacts) {
  foreach ($Proxy in $Contact.proxyAddresses) {
    $ProxyRows += [PSCustomObject]@{
      ObjectType = "Contact"
      Name = $Contact.Name
      SamAccountName = ""
      DistinguishedName = $Contact.DistinguishedName
      ProxyAddressNormalized = $Proxy.ToLowerInvariant()
      ProxyAddressOriginal = $Proxy
    }
  }
}

$ProxyRows |
  Export-Csv "$EvidencePath\all-proxyAddresses-flattened.csv" -NoTypeInformation

$ProxyRows |
  Group-Object ProxyAddressNormalized |
  Where-Object { $_.Count -gt 1 } |
  ForEach-Object {
    $DuplicateValue = $_.Name
    $_.Group | Select-Object `
      @{Name="DuplicateProxyAddress";Expression={$DuplicateValue}},
      ObjectType,
      Name,
      SamAccountName,
      DistinguishedName,
      ProxyAddressOriginal
  } |
  Export-Csv "$EvidencePath\duplicate-proxyAddresses-detail.csv" -NoTypeInformation

Stop-Transcript
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Duplicate_Object_And_Matching_Triage_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell and AD module if available.
# Purpose: compare affected AD object and cloud object for matching, source anchor, and duplicate object state.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"
$AffectedUpn = "jdoe@contoso.com"
$AdSamAccountName = "jdoe"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-duplicate-object-matching-triage-transcript.txt"

# AD source object.
Import-Module ActiveDirectory

$AdUser = Get-ADUser -Identity $AdSamAccountName `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,ObjectGUID,ObjectSID,Enabled,whenCreated,whenChanged

$AdUser |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    ObjectGUID,
    ObjectSID,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    whenCreated,
    whenChanged,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\$AdSamAccountName-ad-object.csv" -NoTypeInformation

# Convert AD ObjectGUID to common immutable ID style for comparison in older workflows.
$ImmutableIdCandidate = [System.Convert]::ToBase64String($AdUser.ObjectGUID.ToByteArray())

[PSCustomObject]@{
  SamAccountName = $AdUser.SamAccountName
  ObjectGUID = $AdUser.ObjectGUID
  ImmutableIdCandidate = $ImmutableIdCandidate
} |
  Export-Csv "$EvidencePath\$AdSamAccountName-immutableid-candidate.csv" -NoTypeInformation

# Cloud object.
Import-Module Microsoft.Graph.Users
Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All"

$CloudUser = Get-MgUser `
  -UserId $AffectedUpn `
  -Property Id,DisplayName,UserPrincipalName,Mail,ProxyAddresses,OnPremisesSyncEnabled,OnPremisesImmutableId,OnPremisesSecurityIdentifier,OnPremisesDistinguishedName,OnPremisesDomainName,OnPremisesSamAccountName,OnPremisesProvisioningErrors,AccountEnabled,CreatedDateTime `
  -ErrorAction SilentlyContinue

if ($CloudUser) {
  $CloudUser |
    Select-Object `
      Id,
      DisplayName,
      UserPrincipalName,
      Mail,
      AccountEnabled,
      OnPremisesSyncEnabled,
      OnPremisesImmutableId,
      OnPremisesSecurityIdentifier,
      OnPremisesDistinguishedName,
      OnPremisesDomainName,
      OnPremisesSamAccountName,
      CreatedDateTime,
      @{Name="ProxyAddresses";Expression={($_.ProxyAddresses -join ";")}},
      @{Name="OnPremisesProvisioningErrors";Expression={($_.OnPremisesProvisioningErrors | ConvertTo-Json -Depth 5)}} |
    Export-Csv "$EvidencePath\$AdSamAccountName-cloud-object.csv" -NoTypeInformation
}

# Search cloud for same mail.
if ($AdUser.mail) {
  Get-MgUser `
    -Filter "mail eq '$($AdUser.mail)'" `
    -Property Id,DisplayName,UserPrincipalName,Mail,OnPremisesSyncEnabled,OnPremisesImmutableId |
    Select-Object Id,DisplayName,UserPrincipalName,Mail,OnPremisesSyncEnabled,OnPremisesImmutableId |
    Export-Csv "$EvidencePath\$AdSamAccountName-cloud-same-mail-search.csv" -NoTypeInformation
}

# Search cloud for same UPN.
Get-MgUser `
  -Filter "userPrincipalName eq '$($AdUser.UserPrincipalName)'" `
  -Property Id,DisplayName,UserPrincipalName,Mail,OnPremisesSyncEnabled,OnPremisesImmutableId |
  Select-Object Id,DisplayName,UserPrincipalName,Mail,OnPremisesSyncEnabled,OnPremisesImmutableId |
  Export-Csv "$EvidencePath\$AdSamAccountName-cloud-same-upn-search.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Object_Sync_Error_Export_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph.
# Purpose: export cloud-side users with provisioning error data and sync flags for review.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"
$VerifiedDomain = "contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-object-sync-error-export-transcript.txt"

Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All"

# Export synced users with relevant on-premises properties.
Get-MgUser -All `
  -Property Id,DisplayName,UserPrincipalName,Mail,ProxyAddresses,OnPremisesSyncEnabled,OnPremisesImmutableId,OnPremisesSecurityIdentifier,OnPremisesSamAccountName,OnPremisesDistinguishedName,OnPremisesDomainName,OnPremisesProvisioningErrors,AccountEnabled |
  Select-Object `
    Id,
    DisplayName,
    UserPrincipalName,
    Mail,
    AccountEnabled,
    OnPremisesSyncEnabled,
    OnPremisesImmutableId,
    OnPremisesSecurityIdentifier,
    OnPremisesSamAccountName,
    OnPremisesDistinguishedName,
    OnPremisesDomainName,
    @{Name="ProxyAddresses";Expression={($_.ProxyAddresses -join ";")}},
    @{Name="ProvisioningErrors";Expression={($_.OnPremisesProvisioningErrors | ConvertTo-Json -Depth 5)}} |
  Export-Csv "$EvidencePath\cloud-users-sync-state.csv" -NoTypeInformation

# Export cloud users with provisioning errors.
Get-MgUser -All `
  -Property Id,DisplayName,UserPrincipalName,Mail,OnPremisesSyncEnabled,OnPremisesProvisioningErrors |
  Where-Object { $_.OnPremisesProvisioningErrors -and $_.OnPremisesProvisioningErrors.Count -gt 0 } |
  Select-Object `
    Id,
    DisplayName,
    UserPrincipalName,
    Mail,
    OnPremisesSyncEnabled,
    @{Name="ProvisioningErrors";Expression={($_.OnPremisesProvisioningErrors | ConvertTo-Json -Depth 10)}} |
  Export-Csv "$EvidencePath\cloud-users-with-provisioning-errors.csv" -NoTypeInformation

# Export users not using expected verified domain.
Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,Mail,OnPremisesSyncEnabled |
  Where-Object { $_.UserPrincipalName -notlike "*@$VerifiedDomain" } |
  Select-Object Id,DisplayName,UserPrincipalName,Mail,OnPremisesSyncEnabled |
  Export-Csv "$EvidencePath\cloud-users-not-using-verified-domain.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Remediation_Skeleton

```powershell
# Controlled remediation examples.
# Do not run as a blind script.
# Fix the source of authority, then run sync and validate.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"

Start-Transcript -Path "$EvidencePath\05-remediation-transcript.txt"

Import-Module ActiveDirectory

# Example 1: Fix duplicate UPN on the wrong AD user.
Set-ADUser `
  -Identity "jdoe2" `
  -UserPrincipalName "jdoe2@contoso.com"

# Example 2: Fix duplicate mail value.
Set-ADUser `
  -Identity "jdoe2" `
  -EmailAddress "jdoe2@contoso.com"

# Example 3: Remove a duplicate proxyAddress from the wrong object.
Set-ADUser `
  -Identity "jdoe2" `
  -Remove @{proxyAddresses = "smtp:jdoe@contoso.com"}

# Example 4: Add the correct proxyAddress to the intended object.
Set-ADUser `
  -Identity "jdoe" `
  -Add @{proxyAddresses = "smtp:jdoe.alias@contoso.com"}

# Example 5: Replace all proxyAddresses only when the full intended list is known.
$CorrectProxyAddresses = @(
  "SMTP:jdoe@contoso.com",
  "smtp:john.doe@contoso.com"
)

Set-ADUser `
  -Identity "jdoe" `
  -Replace @{proxyAddresses = $CorrectProxyAddresses}

# Example 6: Correct mailNickname.
Set-ADUser `
  -Identity "jdoe" `
  -Replace @{mailNickname = "jdoe"}

# Example 7: Clear a bad targetAddress.
Set-ADUser `
  -Identity "jdoe" `
  -Clear targetAddress

Stop-Transcript
```

```powershell
# Run on Entra Connect Sync server after AD remediation.
# Purpose: force delta sync and capture result.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"

Start-Transcript -Path "$EvidencePath\06-post-remediation-sync-cycle-transcript.txt"

Import-Module ADSync -ErrorAction SilentlyContinue

Get-ADSyncScheduler |
  Format-List * |
  Out-File "$EvidencePath\adsync-scheduler-before-remediation-delta.txt"

Start-ADSyncSyncCycle -PolicyType Delta

Start-Sleep -Seconds 60

Get-ADSyncScheduler |
  Format-List * |
  Out-File "$EvidencePath\adsync-scheduler-after-remediation-delta.txt"

Get-WinEvent -LogName Application -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*ADSync*" -or
    $_.Message -like "*sync*" -or
    $_.Message -like "*export*" -or
    $_.Message -like "*duplicate*"
  } |
  Select-Object TimeCreated,ProviderName,Id,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\adsync-events-after-remediation.csv" -NoTypeInformation

Stop-Transcript
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Post_Remediation_Validation_Skeleton

```powershell
# Run after remediation and sync cycle.
# Purpose: prove duplicate issues are gone and affected object sync state is healthy.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"
$AffectedUpn = "jdoe@contoso.com"
$AdSamAccountName = "jdoe"
$SearchBase = "OU=Users,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\07-post-remediation-validation-transcript.txt"

Import-Module ActiveDirectory

# Recheck duplicate values.
$Users = Get-ADUser -SearchBase $SearchBase -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,Enabled

$Users |
  Where-Object { $_.UserPrincipalName } |
  Group-Object UserPrincipalName |
  Where-Object Count -gt 1 |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\post-remediation-duplicate-upn.csv" -NoTypeInformation

$Users |
  Where-Object { $_.mail } |
  Group-Object mail |
  Where-Object Count -gt 1 |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\post-remediation-duplicate-mail.csv" -NoTypeInformation

$Users |
  Where-Object { $_.mailNickname } |
  Group-Object mailNickname |
  Where-Object Count -gt 1 |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\post-remediation-duplicate-mailNickname.csv" -NoTypeInformation

# Validate affected AD object.
Get-ADUser -Identity $AdSamAccountName `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,ObjectGUID,Enabled |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    ObjectGUID,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    targetAddress,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\$AdSamAccountName-ad-object-after-remediation.csv" -NoTypeInformation

# Validate affected cloud object.
Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All"

Get-MgUser `
  -UserId $AffectedUpn `
  -Property Id,DisplayName,UserPrincipalName,Mail,ProxyAddresses,OnPremisesSyncEnabled,OnPremisesImmutableId,OnPremisesSecurityIdentifier,OnPremisesSamAccountName,OnPremisesDistinguishedName,OnPremisesProvisioningErrors,AccountEnabled |
  Select-Object `
    Id,
    DisplayName,
    UserPrincipalName,
    Mail,
    AccountEnabled,
    OnPremisesSyncEnabled,
    OnPremisesImmutableId,
    OnPremisesSecurityIdentifier,
    OnPremisesSamAccountName,
    OnPremisesDistinguishedName,
    @{Name="ProxyAddresses";Expression={($_.ProxyAddresses -join ";")}},
    @{Name="ProvisioningErrors";Expression={($_.OnPremisesProvisioningErrors | ConvertTo-Json -Depth 5)}} |
  Export-Csv "$EvidencePath\$AdSamAccountName-cloud-object-after-remediation.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-Service ADSync` | Confirms local sync service state | ADSync service is running |
| `Get-ADSyncScheduler` | Confirms scheduler state | Scheduler enabled unless intentionally paused |
| `Set-ADSyncScheduler -SyncCycleEnabled $false` | Emergency pause for sync cycles | Scheduler is disabled |
| `Set-ADSyncScheduler -SyncCycleEnabled $true` | Resume sync cycles after remediation | Scheduler is enabled |
| `Start-ADSyncSyncCycle -PolicyType Delta` | Runs incremental sync | Delta sync starts |
| `Start-ADSyncSyncCycle -PolicyType Initial` | Runs full sync after major scope or rule changes | Initial sync starts |
| `Get-ADSyncConnector` | Lists sync connectors | AD DS and Microsoft Entra connectors appear |
| `miisclient.exe` | Opens local Sync Service Manager | Operations, connector space, and run history are visible |
| `Get-WinEvent -LogName Application -MaxEvents 500` | Reviews local sync events | No repeated critical sync failures |
| `Get-ADUser -Filter * -Properties UserPrincipalName \| Group-Object UserPrincipalName \| Where-Object Count -gt 1` | Detects duplicate UPNs | No duplicates return |
| `Get-ADUser -Filter * -Properties mail \| Group-Object mail \| Where-Object Count -gt 1` | Detects duplicate mail values | No duplicates return |
| `Get-ADUser -Filter * -Properties mailNickname \| Group-Object mailNickname \| Where-Object Count -gt 1` | Detects duplicate aliases | No duplicates return |
| `Get-ADUser -Identity jdoe -Properties proxyAddresses` | Reviews AD proxy addresses | Intended aliases only |
| `Get-MgUser -UserId "jdoe@contoso.com" -Property OnPremisesSyncEnabled,OnPremisesImmutableId` | Checks cloud sync linkage | Synced object shows expected on-premises properties |
| `Get-MgUser -UserId "jdoe@contoso.com" -Property OnPremisesProvisioningErrors` | Checks cloud provisioning errors | No unresolved provisioning errors |
| `repadmin /replsummary` | Confirms AD replication health | No major replication failures |
| `dcdiag /test:dns` | Confirms AD DNS health | DNS tests pass or exceptions documented |
| `Resolve-DnsName contoso.com -Type TXT` | Confirms verified domain DNS query | TXT records resolve |

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Rollback

```powershell
# Roll back AD identity attributes from saved rollback CSV.
# Use only with an approved rollback decision.
# This restores values captured before duplicate remediation.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"
$RollbackCsv = "$EvidencePath\rollback-sync-attribute-map.csv"

Start-Transcript -Path "$EvidencePath\rollback-sync-attribute-map-transcript.txt"

Import-Module ActiveDirectory

$Rows = Import-Csv $RollbackCsv

foreach ($Row in $Rows) {
  Write-Host "Restoring $($Row.SamAccountName)"

  $Replace = @{}

  if ($Row.UserPrincipalName) { $Replace["userPrincipalName"] = $Row.UserPrincipalName }
  if ($Row.mail) { $Replace["mail"] = $Row.mail }
  if ($Row.mailNickname) { $Replace["mailNickname"] = $Row.mailNickname }
  if ($Row.targetAddress) { $Replace["targetAddress"] = $Row.targetAddress }

  if ($Replace.Count -gt 0) {
    Set-ADUser `
      -Identity $Row.SamAccountName `
      -Replace $Replace
  }

  if ($Row.proxyAddresses) {
    $ProxyValues = $Row.proxyAddresses -split ";"

    Set-ADUser `
      -Identity $Row.SamAccountName `
      -Replace @{proxyAddresses = $ProxyValues}
  }
}

Stop-Transcript
```

```powershell
# Sync control rollback.
# Use when remediation created unexpected cloud impact.

$EvidencePath = "C:\HybridIdentity-Troubleshooting"

Start-Transcript -Path "$EvidencePath\rollback-sync-control-transcript.txt"

Import-Module ADSync -ErrorAction SilentlyContinue

# Disable scheduled sync while impact is reviewed.
Set-ADSyncScheduler -SyncCycleEnabled $false

Get-ADSyncScheduler |
  Format-List * |
  Out-File "$EvidencePath\adsync-scheduler-disabled-during-rollback.txt"

# Run delta sync only after AD rollback is complete and approved.
# Start-ADSyncSyncCycle -PolicyType Delta

Stop-Transcript
```

```text
# Portal rollback notes.

If the problem is Cloud Sync:
1. Go to Microsoft Entra admin center.
2. Open Cloud Sync configuration.
3. Disable affected provisioning configuration.
4. Correct AD source attributes.
5. Re-enable only after pilot validation.

If the problem is Entra Connect Sync export:
1. Disable scheduler.
2. Open Synchronization Service Manager.
3. Review pending exports.
4. Fix source objects.
5. Run delta sync.
6. Re-enable scheduler only after run history is clean.

If the problem is duplicate cloud object matching:
1. Do not delete the cloud object immediately.
2. Determine if it is cloud-only or synced.
3. Export cloud properties.
4. Confirm source anchor and matching state.
5. Fix source values or matching conflict.
6. Validate sign-in and licensing impact.
```

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Connect Health shows duplicate attribute | Same UPN, mail, or proxyAddress exists on multiple objects | Duplicate attribute triage reports | Correct duplicate value in AD DS or cloud-only object |
| Export error for user | Bad attribute or cloud conflict prevents export | `miisclient.exe` Operations tab | Fix source object and rerun delta sync |
| User appears cloud-only instead of synced | Soft match failed or source anchor mismatch | `Get-MgUser -Property OnPremisesSyncEnabled,OnPremisesImmutableId` | Resolve matching conflict before deleting anything |
| Two users represent one person | Existing cloud user and AD user did not join | Compare UPN, mail, proxyAddresses, immutable ID | Plan soft match or hard match remediation carefully |
| UPN changed but cloud still shows old value | Sync not run, scheduler disabled, or AD replication delay | `Get-ADSyncScheduler`; `repadmin /replsummary` | Fix replication and run delta sync |
| proxyAddress duplicate persists | Duplicate exists on group or contact, not just users | `duplicate-proxyAddresses-detail.csv` | Remove duplicate from wrong object |
| mailNickname duplicate persists | Same alias value exists on multiple objects | `duplicate-mailNickname-summary.csv` | Assign unique alias |
| Multiple primary SMTP values | More than one uppercase `SMTP:` value exists | Inspect `proxyAddresses` | Keep one primary, make other aliases lowercase `smtp:` |
| Connect Health server not reporting | Agent or service issue, network issue, old agent | Portal health and local services | Restart service, update agent, fix outbound access |
| ADSync service stopped | Service failure or server problem | `Get-Service ADSync` | Start service and review event logs |
| Scheduler disabled after fix | Emergency pause was not reverted | `Get-ADSyncScheduler` | Re-enable scheduler after validation |
| Large deletion threshold blocks export | Scope changed and many objects would delete | `miisclient.exe` pending export | Investigate scope before changing threshold |
| Cloud Sync provisioning stopped | Configuration disabled or agent unhealthy | Entra portal provisioning logs | Re-enable config or repair agent |
| Cloud object has provisioning errors | Invalid source attribute or duplicate cloud conflict | Graph export and provisioning logs | Correct source object and rerun provisioning |
| AD object not in sync scope | OU filtering excludes object | Entra Connect wizard scope or Cloud Sync config | Add correct OU only after validation |
| Sign-in broken after remediation | UPN or auth method changed unexpectedly | AD object, cloud object, sign-in logs | Restore UPN or fix user communication and auth config |
| License assignment fails after sync | Usage location missing or group licensing error | Microsoft 365 admin center or Graph license details | Set usage location and fix licensing group |
| Admin cannot view Connect Health | Missing role or permissions | Entra role assignment | Assign appropriate role such as Hybrid Identity Administrator or report reader role |
| Fix made in cloud disappears | Attribute is mastered from AD DS | Cloud object sync flags | Fix source AD DS attribute instead |
| IdFix and Connect Health disagree | Different scan timing or scope | Rerun IdFix and delta sync | Compare timestamps and object scope |

# 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md | Prevents nonroutable UPN and domain-related sync failures |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md | Confirms verified domain state before UPN-based sync troubleshooting |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md | Provides the cleanup baseline used before troubleshooting duplicate sync errors |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md | Provides the sync engine, Cloud Sync agent, scheduler, and scope baseline |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md | Depends on healthy synced users and correct sign-in names |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md | Protects identities after sync problems are resolved |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments.md | Depends on clean users and groups for licensing and access assignment |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md | Uses sync health evidence when diagnosing sign-in, MFA, CA, licensing, and admin access issues |