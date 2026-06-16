# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Index

04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Source_Basis  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Mental_Model  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Planning_Table  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Configuration_Checklist  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Sync_Method_Decision_Table  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Server_Precheck_Skeleton  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Entra_Connect_Sync_Install_Skeleton  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Entra_Connect_Sync_Validation_Skeleton  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Cloud_Sync_Agent_Baseline_Skeleton  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Sync_Scope_And_Pilot_Validation_Skeleton  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Verification_Commands  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Rollback  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Failure_Checks  
04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Related_Labs  

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft 365 Docs | Set up directory synchronization for Microsoft 365 | Hybrid identity baseline, Microsoft Entra Connect Sync role, PHS, PTA, federation, and tenant preparation |
| Microsoft 365 Docs | Prepare for directory synchronization to Microsoft 365 | AD DS preparation, cleanup, UPN readiness, and sync prerequisites |
| Microsoft Entra Docs | Microsoft Entra Connect Sync installation roadmap | Express installation, custom installation, staging mode, filtering, password hash sync, password writeback, device writeback, accidental delete protection, and upgrade planning |
| Microsoft Entra Docs | Microsoft Entra Cloud Sync | Lightweight agent-based provisioning, cloud-managed sync configuration, and multi-agent design |
| Microsoft Entra Docs | Connect Health | Monitoring, alerts, sync errors, object-level issues, and health reporting |
| Windows Server AD DS | ActiveDirectory PowerShell module | OU scope validation, user and group inventory, and pilot object verification |
| Microsoft Graph PowerShell | Domain, user, group, organization, and directory checks | Cloud-side verification of synchronized objects |
| ADSync PowerShell module | Start-ADSyncSyncCycle, Get-ADSyncScheduler, Set-ADSyncScheduler | Local Entra Connect Sync scheduler and sync cycle validation |
| Operational practice | Staging mode first, pilot OU first, export first, verify before enabling full sync | Prevents tenant-wide identity blast radius |

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Microsoft Entra Connect Sync | Traditional on-prem sync engine installed on a Windows Server to synchronize AD DS objects to Microsoft Entra ID |
| Microsoft Entra Cloud Sync | Lightweight provisioning agent model where configuration is managed mainly in the cloud |
| Hybrid identity | Identity design where on-prem AD DS objects are represented in Microsoft Entra ID |
| Source anchor | Immutable identity link between an on-prem object and its cloud object |
| Password Hash Synchronization | Password hash material is synchronized to Microsoft Entra ID so cloud can authenticate users |
| Pass-through Authentication | Microsoft Entra sign-in is validated by on-prem agents against AD DS |
| Federation | Microsoft Entra redirects authentication to another identity provider such as AD FS |
| Seamless SSO | Domain-joined users can sign in to cloud apps without retyping credentials on trusted corporate devices |
| Sync scope | Which OUs, users, groups, contacts, and attributes are allowed to sync |
| Pilot OU | Small test OU used to validate sync safely before expanding scope |
| Staging mode | Entra Connect Sync mode where import and sync run, but exports to Microsoft Entra ID are disabled |
| Delta sync | Incremental synchronization of recent changes |
| Initial sync | Full synchronization pass after major scope or rule changes |
| Accidental delete protection | Guardrail that blocks large unexpected deletion exports |
| Connect Health | Cloud monitoring layer for sync service health and errors |
| Sync scheduler | Local Entra Connect Sync schedule that controls automatic delta sync cycles |
| Provisioning agent | Cloud Sync on-prem component that talks to AD DS and Microsoft Entra provisioning service |
| First rule | Do not point sync at the whole directory on day one |
| Blunt rule | If AD cleanup was skipped, sync will turn your dirty directory into visible cloud problems |

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Sync method | Entra Connect Sync / Cloud Sync | `<sync-method>` |
| AD forest | `corp.local` | `<ad-forest>` |
| AD domain | `corp.local` | `<ad-domain>` |
| Verified cloud domain | `contoso.com` | `<verified-domain>` |
| Tenant initial domain | `contoso.onmicrosoft.com` | `<tenant-domain>` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Entra Connect server | `SYNC01.corp.local` | `<sync-server>` |
| Cloud Sync agent server | `PROVAGENT01.corp.local` | `<cloud-sync-agent-server>` |
| Sync service account | Microsoft Entra Connect-created account | `<sync-service-account>` |
| AD DS connector account | `MSOL_########` or gMSA/delegated account | `<ad-connector-account>` |
| Cloud admin role | Hybrid Identity Administrator | `<cloud-admin-role>` |
| AD admin role | Enterprise Admin / Domain Admin for setup | `<ad-admin-role>` |
| Authentication method | PHS / PTA / Federation | `<auth-method>` |
| Seamless SSO | Enabled / Disabled | `<seamless-sso>` |
| Password writeback | Enabled / Disabled | `<password-writeback>` |
| Device writeback | Enabled / Disabled | `<device-writeback>` |
| Group writeback | Enabled / Disabled | `<group-writeback>` |
| Pilot OU | `OU=PilotUsers,OU=Users,DC=corp,DC=local` | `<pilot-ou>` |
| Production OU | `OU=Users,DC=corp,DC=local` | `<production-ou>` |
| Excluded OUs | Service accounts, disabled users, stale objects | `<excluded-ous>` |
| Object types | Users, groups, contacts | `<object-types>` |
| Initial sync mode | Staging mode / export enabled | `<initial-sync-mode>` |
| Evidence folder | `C:\HybridIdentity-Sync` | `<evidence-path>` |
| Change window | After hours | `<change-window>` |
| Rollback plan | Disable scheduler, restore scope, staging mode, stop agent | `<rollback-plan>` |

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm AD cleanup workbook is complete | Admin Workstation | `Review cleanup reports from task 03` | UPN, mail, proxyAddresses, and duplicate issues are clean |
| 2 | Confirm custom domain is verified | Admin Workstation | `Get-MgDomain -DomainId "contoso.com"` | Domain exists and `IsVerified` is `True` |
| 3 | Create evidence folder | Sync Server | `New-Item -ItemType Directory -Force -Path C:\HybridIdentity-Sync` | Evidence folder exists |
| 4 | Confirm server identity | Sync Server | `hostname; whoami /all` | Correct server and admin context are confirmed |
| 5 | Confirm domain join | Sync Server | `(Get-CimInstance Win32_ComputerSystem).PartOfDomain` | Server is domain joined |
| 6 | Confirm OS and patch baseline | Sync Server | `Get-ComputerInfo \| Select WindowsProductName,OsVersion,OsBuildNumber` | Supported Windows Server build is documented |
| 7 | Confirm DNS client points to AD DNS | Sync Server | `Get-DnsClientServerAddress -AddressFamily IPv4` | AD DNS servers are configured |
| 8 | Confirm DC discovery | Sync Server | `nltest /dsgetdc:corp.local` | Domain controller discovery succeeds |
| 9 | Confirm AD module availability | Sync Server | `Get-Module -ListAvailable ActiveDirectory` | AD module is available |
| 10 | Confirm Graph module availability | Admin Workstation | `Get-Module -ListAvailable Microsoft.Graph` | Graph module is available |
| 11 | Connect to Microsoft Graph | Admin Workstation | `Connect-MgGraph -Scopes "Directory.Read.All","Domain.Read.All","User.Read.All","Group.Read.All"` | Cloud directory session connects |
| 12 | Export tenant domains | Admin Workstation | `Get-MgDomain \| Export-Csv C:\HybridIdentity-Sync\tenant-domains.csv -NoTypeInformation` | Tenant domains are documented |
| 13 | Export pilot OU users | Sync Server | `Get-ADUser -SearchBase "<pilot-ou>" -Filter * -Properties UserPrincipalName,mail` | Pilot users are visible |
| 14 | Decide sync method | Operator | `Use Sync_Method_Decision_Table` | Entra Connect Sync or Cloud Sync is selected |
| 15 | For Entra Connect Sync, download installer | Sync Server | `Download from official Microsoft source` | Installer is available locally |
| 16 | For Entra Connect Sync, start custom install | Sync Server | `AzureADConnect.msi` | Wizard starts |
| 17 | Select authentication method | Sync Server | `PHS / PTA / Federation` | Auth method matches plan |
| 18 | Configure OU filtering | Sync Server | `Select pilot OU first` | Only pilot scope is selected |
| 19 | Enable staging mode for first validation | Sync Server | `Enable staging mode in wizard` | No cloud exports occur yet |
| 20 | Complete Entra Connect Sync install | Sync Server | `Finish wizard` | Sync service installs successfully |
| 21 | Validate ADSync service | Sync Server | `Get-Service ADSync` | Service is running |
| 22 | Validate scheduler state | Sync Server | `Get-ADSyncScheduler` | Scheduler settings are visible |
| 23 | Run initial sync in staging mode | Sync Server | `Start-ADSyncSyncCycle -PolicyType Initial` | Initial cycle starts |
| 24 | Review Synchronization Service Manager | Sync Server | `miisclient.exe` | Import, sync, and connector status are reviewed |
| 25 | Disable staging mode only after validation | Sync Server | `Azure AD Connect wizard > Configure staging mode` | Exports are enabled only after approval |
| 26 | Run delta sync | Sync Server | `Start-ADSyncSyncCycle -PolicyType Delta` | Delta cycle succeeds |
| 27 | Verify cloud pilot users | Admin Workstation | `Get-MgUser -Filter "userPrincipalName eq 'pilot.user@contoso.com'"` | Pilot user appears in Microsoft Entra ID |
| 28 | For Cloud Sync, install provisioning agent | Cloud Sync Agent Server | `Install Microsoft Entra provisioning agent` | Agent registers and appears in portal |
| 29 | Configure Cloud Sync scoped provisioning | Entra Admin Center | `Hybrid identity > Cloud Sync > New configuration` | Pilot OU or scoped objects are selected |
| 30 | Enable Cloud Sync configuration for pilot | Entra Admin Center | `Enable provisioning configuration` | Pilot provisioning begins |
| 31 | Validate Cloud Sync agent service | Cloud Sync Agent Server | `Get-Service \| Where-Object {$_.DisplayName -like "*Provisioning*"}` | Provisioning agent service is running |
| 32 | Validate pilot object in cloud | Admin Workstation | `Get-MgUser -Filter "userPrincipalName eq 'pilot.user@contoso.com'"` | Pilot user appears |
| 33 | Review provisioning and sync errors | Entra Admin Center | `Identity > Hybrid management > Health / Provisioning logs` | No blocking pilot errors remain |
| 34 | Document chosen baseline | Operator | `Record sync method, scope, auth method, enabled features, server, agent, and rollback` | Baseline is documented |
| 35 | Expand scope only after pilot sign-off | Operator | `Add next OU batch` | Sync scope expands in controlled batches |

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Sync_Method_Decision_Table

| Requirement | Entra Connect Sync | Cloud Sync | Decision |
|---|---|---|---|
| Full traditional sync engine | Strong fit | Not the model | Use Entra Connect Sync |
| Lightweight agent model | Not ideal | Strong fit | Use Cloud Sync |
| Complex sync rules and transformations | Strong fit | Limited fit | Use Entra Connect Sync |
| Large mature hybrid identity environment | Strong fit | Possible but validate limits | Usually Entra Connect Sync |
| Multiple simple AD forests | Possible | Strong fit in many cases | Consider Cloud Sync |
| Need Exchange hybrid attribute handling | Strong fit | Validate requirements | Usually Entra Connect Sync |
| Need password hash synchronization | Supported | Supported | Either |
| Need pass-through authentication | Supported through PTA agents | Not the same core model | Usually Entra Connect Sync plus PTA |
| Need federation with AD FS | Supported | Not primary use case | Entra Connect Sync |
| Need staged migration from existing Connect server | Strong fit | Depends on target design | Validate migration plan |
| Need minimal on-prem server footprint | Heavier | Lighter | Cloud Sync |
| Need scoping by pilot OU | Supported | Supported | Either |
| Need advanced connector space inspection | Strong fit with Synchronization Service Manager | Limited local visibility | Entra Connect Sync |
| Need cloud-managed provisioning configuration | Not primary | Strong fit | Cloud Sync |
| Operational skillset is classic AD and sync engine | Strong fit | Requires provisioning model understanding | Entra Connect Sync |
| First lab baseline | Strong fit for learning full surface area | Good second baseline | Build both if lab allows |

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Server_Precheck_Skeleton

```powershell
# Run on the planned sync server or Cloud Sync agent server.
# Purpose: prove the server can support hybrid identity sync components.

$EvidencePath = "C:\HybridIdentity-Sync"
$DomainFqdn = "corp.local"
$VerifiedDomain = "contoso.com"
$PilotOu = "OU=PilotUsers,OU=Users,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-server-precheck-transcript.txt"

# Identity and OS baseline.
hostname | Out-File "$EvidencePath\hostname.txt"
whoami /all | Out-File "$EvidencePath\whoami-all.txt"

Get-ComputerInfo |
  Select-Object WindowsProductName,WindowsVersion,OsVersion,OsBuildNumber,CsDomain,CsPartOfDomain |
  Format-List * |
  Out-File "$EvidencePath\computer-info.txt"

# Domain membership and secure channel.
(Get-CimInstance Win32_ComputerSystem) |
  Select-Object Name,Domain,PartOfDomain |
  Format-List * |
  Out-File "$EvidencePath\domain-membership.txt"

Test-ComputerSecureChannel -Verbose |
  Out-File "$EvidencePath\secure-channel.txt"

# DNS client state.
Get-DnsClientServerAddress -AddressFamily IPv4 |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\dns-client-servers.txt"

# Domain controller discovery.
nltest /dsgetdc:$DomainFqdn | Out-File "$EvidencePath\nltest-dsgetdc.txt"

# AD module and pilot object visibility.
Get-Module -ListAvailable ActiveDirectory |
  Out-File "$EvidencePath\ad-module-availability.txt"

Import-Module ActiveDirectory

Get-ADForest |
  Format-List * |
  Out-File "$EvidencePath\ad-forest.txt"

Get-ADDomain |
  Format-List * |
  Out-File "$EvidencePath\ad-domain.txt"

Get-ADUser -SearchBase $PilotOu -Filter * -Properties UserPrincipalName,mail,Enabled |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,DistinguishedName |
  Export-Csv "$EvidencePath\pilot-users-before-sync.csv" -NoTypeInformation

# Public DNS check for verified cloud domain.
Resolve-DnsName $VerifiedDomain -Type TXT -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\verified-domain-txt-records.txt"

Resolve-DnsName $VerifiedDomain -Type MX -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\verified-domain-mx-records.txt"

# Local service check before install.
Get-Service |
  Where-Object {
    $_.Name -like "*ADSync*" -or
    $_.DisplayName -like "*Microsoft Entra*" -or
    $_.DisplayName -like "*Azure AD*" -or
    $_.DisplayName -like "*Provisioning*"
  } |
  Select-Object Name,DisplayName,Status,StartType |
  Export-Csv "$EvidencePath\sync-related-services-before.csv" -NoTypeInformation

Stop-Transcript
```

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Entra_Connect_Sync_Install_Skeleton

```text
# Microsoft Entra Connect Sync install skeleton.
# This part is wizard-driven by design. Keep evidence screenshots and exports in C:\HybridIdentity-Sync.

1. Sign in to SYNC01 using the approved setup account.
2. Confirm AD cleanup and domain validation workbooks are complete.
3. Confirm the server is domain joined.
4. Confirm DNS client points to AD DNS.
5. Download Microsoft Entra Connect Sync from the official Microsoft download source.
6. Start the installer.
7. Accept license terms.
8. Choose Customize.
9. Use existing SQL only if planned. Otherwise use the default local database for lab or small baseline.
10. Choose sign-in method:
    - Password Hash Synchronization for the default baseline
    - Pass-through Authentication only if planned
    - Federation only if AD FS or third-party federation is planned
11. Enable Seamless SSO only if planned and supported by the environment.
12. Connect to Microsoft Entra ID using an account with the required hybrid identity role.
13. Connect to AD DS using the approved AD account.
14. Verify the AD forest is detected.
15. Confirm the verified custom domain appears.
16. Choose source anchor defaults unless there is a documented reason not to.
17. Configure domain and OU filtering.
18. Select only the pilot OU first:
    OU=PilotUsers,OU=Users,DC=corp,DC=local
19. Configure optional features only if approved:
    - Password writeback
    - Device writeback
    - Group writeback
    - Exchange hybrid deployment
    - Directory extension attribute sync
20. Enable staging mode for first validation if this is a production or cautious baseline.
21. Finish installation.
22. Do not expand to all production OUs yet.
23. Open Synchronization Service Manager:
    C:\Program Files\Microsoft Azure AD Sync\UIShell\miisclient.exe
24. Review connectors:
    - AD DS connector
    - Microsoft Entra connector
25. Review Operations tab after first sync.
26. Export install evidence:
    - Wizard choices
    - selected OUs
    - scheduler state
    - service state
    - connector status
27. Proceed to validation skeleton.
```

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Entra_Connect_Sync_Validation_Skeleton

```powershell
# Run on the Entra Connect Sync server after installation.
# Purpose: validate local sync engine, scheduler, service state, and pilot sync.

$EvidencePath = "C:\HybridIdentity-Sync"
$PilotUserUpn = "pilot.user@contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-entra-connect-sync-validation-transcript.txt"

# ADSync service status.
Get-Service ADSync |
  Select-Object Name,DisplayName,Status,StartType |
  Tee-Object "$EvidencePath\adsync-service-status.txt"

# Load ADSync module if available.
$ADSyncModulePath = "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1"

if (Test-Path $ADSyncModulePath) {
  Import-Module $ADSyncModulePath
}
else {
  Import-Module ADSync -ErrorAction SilentlyContinue
}

# Scheduler state.
Get-ADSyncScheduler |
  Format-List * |
  Tee-Object "$EvidencePath\adsync-scheduler.txt"

# Connector inventory.
Get-ADSyncConnector |
  Select-Object Name,Type,SubType,Identifier |
  Export-Csv "$EvidencePath\adsync-connectors.csv" -NoTypeInformation

# Run a delta sync after install validation.
Start-ADSyncSyncCycle -PolicyType Delta

Start-Sleep -Seconds 30

# Capture scheduler again.
Get-ADSyncScheduler |
  Format-List * |
  Out-File "$EvidencePath\adsync-scheduler-after-delta.txt"

# Capture recent event logs.
Get-WinEvent -LogName Application -MaxEvents 200 |
  Where-Object {
    $_.ProviderName -like "*Directory Synchronization*" -or
    $_.ProviderName -like "*ADSync*" -or
    $_.Message -like "*sync*"
  } |
  Select-Object TimeCreated,ProviderName,Id,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\adsync-application-events.csv" -NoTypeInformation

# Capture local sync-related services.
Get-Service |
  Where-Object {
    $_.Name -like "*ADSync*" -or
    $_.DisplayName -like "*Microsoft Entra*" -or
    $_.DisplayName -like "*Azure AD*"
  } |
  Select-Object Name,DisplayName,Status,StartType |
  Export-Csv "$EvidencePath\sync-related-services-after-install.csv" -NoTypeInformation

Stop-Transcript
```

```powershell
# Run from an admin workstation with Microsoft Graph after sync cycle completes.
# Purpose: validate pilot cloud object.

$EvidencePath = "C:\HybridIdentity-Sync"
$PilotUserUpn = "pilot.user@contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Identity.DirectoryManagement

Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All","Domain.Read.All"

# Validate domain.
Get-MgDomain -DomainId "contoso.com" |
  Select-Object Id,IsVerified,IsDefault,AuthenticationType |
  Format-List * |
  Out-File "$EvidencePath\cloud-domain-validation.txt"

# Validate pilot user.
Get-MgUser -Filter "userPrincipalName eq '$PilotUserUpn'" -Property Id,DisplayName,UserPrincipalName,OnPremisesSyncEnabled,OnPremisesImmutableId,AccountEnabled |
  Select-Object Id,DisplayName,UserPrincipalName,OnPremisesSyncEnabled,OnPremisesImmutableId,AccountEnabled |
  Format-List * |
  Out-File "$EvidencePath\pilot-user-cloud-validation.txt"

Disconnect-MgGraph
```

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Cloud_Sync_Agent_Baseline_Skeleton

```text
# Microsoft Entra Cloud Sync baseline skeleton.
# This part is mostly portal-driven. Use this when Cloud Sync is the selected sync method.

1. Confirm AD cleanup and domain validation workbooks are complete.
2. Confirm the planned agent server is domain joined.
3. Confirm DNS client points to AD DNS.
4. Confirm outbound internet access required for Microsoft Entra provisioning.
5. Sign in to Microsoft Entra admin center.
6. Go to:
   Identity > Hybrid management > Microsoft Entra Cloud Sync
7. Download the Microsoft Entra provisioning agent.
8. Install the provisioning agent on the planned on-prem server.
9. Register the agent to the tenant using the approved cloud admin role.
10. Connect the agent to the target AD DS forest.
11. Confirm the agent appears healthy in the portal.
12. Create a new Cloud Sync configuration.
13. Select the AD domain.
14. Scope the first configuration to the pilot OU or pilot group.
15. Configure attribute mappings only if required.
16. Enable password hash synchronization if planned.
17. Validate provisioning configuration.
18. Enable the configuration.
19. Monitor provisioning logs.
20. Confirm pilot users appear in Microsoft Entra ID.
21. Do not expand production scope until pilot provisioning is healthy.
22. Add additional agents for high availability if required.
23. Document:
    - agent server
    - AD forest
    - selected scope
    - enabled features
    - provisioning status
    - known limitations
    - rollback steps
```

```powershell
# Run on the Cloud Sync agent server.
# Purpose: validate local provisioning agent service and basic server posture.

$EvidencePath = "C:\HybridIdentity-Sync"
$DomainFqdn = "corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-cloud-sync-agent-validation-transcript.txt"

hostname | Out-File "$EvidencePath\cloud-sync-agent-hostname.txt"

Get-ComputerInfo |
  Select-Object WindowsProductName,WindowsVersion,OsVersion,OsBuildNumber,CsDomain,CsPartOfDomain |
  Format-List * |
  Out-File "$EvidencePath\cloud-sync-agent-computer-info.txt"

nltest /dsgetdc:$DomainFqdn |
  Out-File "$EvidencePath\cloud-sync-agent-dc-discovery.txt"

Get-Service |
  Where-Object {
    $_.DisplayName -like "*Provisioning*" -or
    $_.DisplayName -like "*Microsoft Entra*" -or
    $_.DisplayName -like "*Azure AD*"
  } |
  Select-Object Name,DisplayName,Status,StartType |
  Export-Csv "$EvidencePath\cloud-sync-agent-services.csv" -NoTypeInformation

Get-WinEvent -LogName Application -MaxEvents 200 |
  Where-Object {
    $_.Message -like "*provision*" -or
    $_.ProviderName -like "*Provisioning*" -or
    $_.ProviderName -like "*Microsoft Entra*" -or
    $_.ProviderName -like "*Azure AD*"
  } |
  Select-Object TimeCreated,ProviderName,Id,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\cloud-sync-agent-events.csv" -NoTypeInformation

Stop-Transcript
```

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Sync_Scope_And_Pilot_Validation_Skeleton

```powershell
# Run before expanding sync scope.
# Purpose: validate pilot OU users, groups, and cloud appearance.

$EvidencePath = "C:\HybridIdentity-Sync"
$PilotOu = "OU=PilotUsers,OU=Users,DC=corp,DC=local"
$VerifiedDomain = "contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-sync-scope-pilot-validation-transcript.txt"

Import-Module ActiveDirectory

# Export pilot users from AD.
$PilotUsers = Get-ADUser -SearchBase $PilotOu -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,Enabled,ObjectGUID

$PilotUsers |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    ObjectGUID,
    UserPrincipalName,
    mail,
    mailNickname,
    DistinguishedName,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\pilot-users-ad-validation.csv" -NoTypeInformation

# Check pilot users have the verified domain UPN suffix.
$PilotUsers |
  Where-Object { $_.UserPrincipalName -notlike "*@$VerifiedDomain" } |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,DistinguishedName |
  Export-Csv "$EvidencePath\pilot-users-wrong-upn-suffix.csv" -NoTypeInformation

# Export pilot groups if applicable.
Get-ADGroup -SearchBase $PilotOu -Filter * -Properties mail,proxyAddresses,mailNickname |
  Select-Object `
    SamAccountName,
    Name,
    GroupCategory,
    GroupScope,
    mail,
    mailNickname,
    DistinguishedName,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\pilot-groups-ad-validation.csv" -NoTypeInformation

Stop-Transcript
```

```powershell
# Run from admin workstation after sync or provisioning completes.
# Purpose: compare pilot AD users to cloud users.

$EvidencePath = "C:\HybridIdentity-Sync"
$PilotUsersCsv = "$EvidencePath\pilot-users-ad-validation.csv"

Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All"

$PilotUsers = Import-Csv $PilotUsersCsv

$CloudResults = foreach ($User in $PilotUsers) {
  $CloudUser = Get-MgUser `
    -Filter "userPrincipalName eq '$($User.UserPrincipalName)'" `
    -Property Id,DisplayName,UserPrincipalName,OnPremisesSyncEnabled,OnPremisesImmutableId,AccountEnabled `
    -ErrorAction SilentlyContinue

  [PSCustomObject]@{
    SamAccountName = $User.SamAccountName
    AdUserPrincipalName = $User.UserPrincipalName
    CloudFound = [bool]$CloudUser
    CloudDisplayName = $CloudUser.DisplayName
    CloudUserPrincipalName = $CloudUser.UserPrincipalName
    OnPremisesSyncEnabled = $CloudUser.OnPremisesSyncEnabled
    OnPremisesImmutableId = $CloudUser.OnPremisesImmutableId
    AccountEnabled = $CloudUser.AccountEnabled
  }
}

$CloudResults |
  Export-Csv "$EvidencePath\pilot-users-cloud-comparison.csv" -NoTypeInformation

Disconnect-MgGraph
```

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-ADForest` | Confirms AD forest and UPN suffix baseline | Correct forest and verified suffix appear |
| `Get-ADDomain` | Confirms AD domain context | Expected domain returns |
| `nltest /dsgetdc:corp.local` | Confirms domain controller discovery | DC discovery succeeds |
| `Test-ComputerSecureChannel` | Confirms server domain trust | Returns `True` |
| `Get-Service ADSync` | Confirms Entra Connect Sync service | Service is running |
| `Get-ADSyncScheduler` | Reviews sync scheduler | Scheduler state returns |
| `Start-ADSyncSyncCycle -PolicyType Delta` | Triggers delta sync | Sync cycle starts successfully |
| `Start-ADSyncSyncCycle -PolicyType Initial` | Triggers full initial sync | Initial cycle starts when required |
| `Get-ADSyncConnector` | Lists AD and Microsoft Entra connectors | Expected connectors appear |
| `miisclient.exe` | Opens Synchronization Service Manager | Operations and connectors are visible |
| `Get-Service \| Where-Object {$_.DisplayName -like "*Provisioning*"}` | Checks Cloud Sync agent services | Provisioning agent service is running |
| `Get-MgDomain -DomainId "contoso.com"` | Confirms verified cloud domain | `IsVerified` is `True` |
| `Get-MgUser -Filter "userPrincipalName eq 'pilot.user@contoso.com'"` | Confirms synced pilot user | Pilot user returns |
| `Get-MgUser -Filter "userPrincipalName eq 'pilot.user@contoso.com'" -Property OnPremisesSyncEnabled,OnPremisesImmutableId` | Confirms cloud user is synced from AD | Sync flags and anchor data appear |
| `repadmin /replsummary` | Confirms AD replication health before sync expansion | No major replication failures |
| `dcdiag /test:dns` | Confirms AD DNS baseline | DNS tests pass or known exceptions are documented |

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Rollback

```powershell
# Entra Connect Sync safe rollback controls.
# Use only with an approved change decision.

$EvidencePath = "C:\HybridIdentity-Sync"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\rollback-entra-connect-sync-controls.txt"

# Run on Entra Connect Sync server.
Import-Module ADSync -ErrorAction SilentlyContinue

# 1. Stop automatic sync scheduler.
Set-ADSyncScheduler -SyncCycleEnabled $false

# 2. Confirm scheduler disabled.
Get-ADSyncScheduler |
  Format-List * |
  Out-File "$EvidencePath\adsync-scheduler-disabled.txt"

# 3. Stop ADSync service only if required.
# Stopping service is more disruptive than disabling scheduler.
# Stop-Service ADSync

# 4. Put the server back into staging mode using the Entra Connect wizard if exports must be blocked:
# Azure AD Connect > Configure > Configure staging mode > Enable staging mode

Stop-Transcript
```

```text
# Entra Connect Sync rollback procedure.

1. Disable the sync scheduler:
   Set-ADSyncScheduler -SyncCycleEnabled $false

2. If bad exports have not occurred yet:
   - Enable staging mode in the Entra Connect wizard.
   - Review pending exports in Synchronization Service Manager.

3. If wrong objects were scoped:
   - Reopen Entra Connect wizard.
   - Configure domain and OU filtering.
   - Remove the bad OU from scope.
   - Run initial sync after approval.

4. If incorrect attributes were synced:
   - Correct source AD DS attributes.
   - Run delta sync.
   - Validate cloud object state.

5. If a pilot configuration is being abandoned:
   - Disable scheduler.
   - Document cloud objects created.
   - Remove test objects only if safe.
   - Do not delete production cloud users without explicit approval.

6. If uninstalling:
   - Confirm no other sync server is active.
   - Export configuration first.
   - Uninstall Microsoft Entra Connect from Programs and Features.
   - Clean up service accounts only after confirming they are unused.
```

```text
# Cloud Sync rollback procedure.

1. In Microsoft Entra admin center, go to:
   Identity > Hybrid management > Microsoft Entra Cloud Sync

2. Disable the affected provisioning configuration.

3. Confirm provisioning stops.

4. If the agent is unhealthy:
   - Stop the provisioning agent service on the server.
   - Check local events.
   - Repair or reinstall the agent only after preserving logs.

5. If the wrong OU or group was scoped:
   - Edit the Cloud Sync configuration.
   - Correct scoping filters.
   - Re-enable provisioning after validation.

6. If bad objects were provisioned:
   - Correct source AD DS attributes first.
   - Let provisioning repair the cloud objects.
   - Avoid deleting cloud identities manually unless the deletion impact is fully understood.

7. Export final health and provisioning logs.
```

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Entra Connect installer cannot contact AD DS | DNS client not pointing to AD DNS or secure channel issue | `nltest /dsgetdc:corp.local`; `Test-ComputerSecureChannel` | Fix DNS client, domain join, or secure channel |
| Verified domain missing during install | Domain not verified in tenant | `Get-MgDomain -DomainId "contoso.com"` | Complete custom domain verification first |
| Pilot users do not appear in cloud | Wrong OU selected, scheduler disabled, staging mode enabled, or sync cycle not run | `Get-ADSyncScheduler`; `miisclient.exe` | Correct OU scope, run sync, disable staging only after approval |
| Users sync with `.onmicrosoft.com` UPN | UPN suffix not routable or not verified | Check AD UPN and tenant domains | Fix AD UPN suffix and verified domain |
| Sync shows duplicate attribute errors | AD cleanup missed duplicate UPN, mail, or proxyAddresses | Review task 03 reports and sync errors | Correct duplicates in AD DS |
| ADSync service stopped | Service failure or local server issue | `Get-Service ADSync` | Start service and review event logs |
| Scheduler disabled unexpectedly | Prior rollback or staging configuration | `Get-ADSyncScheduler` | Re-enable only after validating scope |
| Large deletion export blocked | Accidental delete protection triggered | Synchronization Service Manager operations | Investigate scope changes before raising threshold |
| Cloud user exists but is not synced | Existing cloud-only object did not match source AD object | `Get-MgUser` sync properties | Review soft match/hard match and source anchor plan |
| Password hash sync not working | PHS not enabled or sync not completed | Entra Connect wizard, sign-in logs | Enable PHS and run sync |
| PTA sign-in fails | PTA agents unavailable or AD cannot validate credentials | Agent status in portal and local services | Restore PTA agents or fail back to planned auth method |
| Seamless SSO fails | Kerberos object, browser, or intranet zone issue | Entra Connect SSO status and client browser settings | Reconfigure Seamless SSO and client policy |
| Cloud Sync agent not visible in portal | Agent registration failed or outbound connectivity blocked | Agent service status and portal health | Re-register agent and fix outbound access |
| Cloud Sync provisions nothing | Configuration disabled or scoping filter excludes objects | Entra portal provisioning config | Correct scope and enable configuration |
| Cloud Sync errors on attributes | Dirty AD attributes or unsupported values | Provisioning logs and AD cleanup reports | Correct AD source attributes |
| Sync server CPU or disk spikes | Initial sync, SQL/local DB load, or undersized server | Task Manager, Event Viewer | Wait for initial sync or resize server |
| Production users sync too early | OU filtering too broad | Synchronization Service Manager pending exports | Enable staging mode, fix scope, then validate |
| Admin cannot change sync settings | Wrong cloud role or AD rights | `whoami /groups`; Entra role check | Use Hybrid Identity Administrator and proper AD rights |

# 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline_Related_Labs

| Lab                                                                                   | Relationship                                                              |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md           | Provides routable UPN suffixes for synced identities                      |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md                           | Verifies the custom domain before user UPNs sync                          |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md               | Cleans AD DS before the sync engine or Cloud Sync provisions objects      |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md | Troubleshoots sync errors after baseline deployment                       |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md               | Builds user recovery and password protection after hybrid identity exists |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md       | Protects synced cloud identities with access controls                     |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments.md          | Uses synchronized users and groups for licensing and access assignment    |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md             | Validates sign-in, MFA, CA, and licensing issues after sync               |
