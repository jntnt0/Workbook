# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Index

03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Source_Basis  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Mental_Model  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Planning_Table  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Configuration_Checklist  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_AD_Object_Inventory_Precheck_Skeleton  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Duplicate_Attribute_Detection_Skeleton  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Invalid_Attribute_Detection_Skeleton  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_IdFix_Runbook_Skeleton  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Cleanup_Staging_And_Remediation_Skeleton  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Post_Cleanup_Validation_Skeleton  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Verification_Commands  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Rollback  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Failure_Checks  
03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Related_Labs  

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft 365 Docs | Prepare for directory synchronization to Microsoft 365 | AD DS cleanup, duplicate attributes, invalid characters, UPN readiness, and sync preparation |
| Microsoft 365 Docs | Prepare a nonroutable domain for directory synchronization | UPN suffix readiness and routable namespace alignment |
| Microsoft Entra Docs | Microsoft Entra Connect Sync readiness | Preparing AD DS before sync |
| Microsoft Entra Docs | IdFix | Discovery and remediation of object and attribute issues before synchronization |
| Windows Server AD DS | ActiveDirectory PowerShell module | Inventory, duplicate checks, invalid attribute discovery, and controlled remediation |
| Exchange / Microsoft 365 identity practice | proxyAddresses, mail, mailNickname, targetAddress | Mail-related identity attributes that commonly cause sync errors |
| Operational practice | Export first, pilot first, remediate in batches, preserve rollback maps | Prevents accidental directory-wide identity damage |

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Directory synchronization | Process that syncs selected AD DS users, groups, contacts, and attributes into Microsoft Entra ID |
| IdFix | Microsoft tool used to find and help correct common identity sync problems before Entra Connect Sync |
| Source of authority | For synced users, AD DS is usually authoritative for synced attributes |
| Dirty directory | AD DS state with duplicate values, invalid characters, blank UPNs, stale users, or bad mail attributes |
| Duplicate attribute | Same value used by multiple objects where cloud sync expects uniqueness |
| Invalid attribute | Attribute containing unsupported characters, broken format, or invalid value length |
| UPN | User sign-in name, ideally aligned with the user email address |
| proxyAddresses | Multi-value attribute that stores SMTP aliases and primary SMTP value |
| Primary SMTP | `SMTP:user@contoso.com`, uppercase `SMTP:` marks the primary reply address |
| Secondary SMTP alias | `smtp:alias@contoso.com`, lowercase `smtp:` marks a secondary alias |
| mail | Usually the primary email address value |
| mailNickname | Exchange alias value, should be unique and clean |
| targetAddress | Routing address used in some hybrid or migration scenarios |
| sAMAccountName | Legacy AD logon name, must be unique and under length limits |
| Object cleanup | Fixing user, group, contact, and mail attributes before first sync |
| Quarantine mindset | Do not sync everything first and fix later |
| First rule | Export the current state before changing anything |
| Blunt rule | If UPN, mail, proxyAddresses, and mailNickname are dirty, Entra Connect will expose the mess immediately |

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Planning_Table

| Item | Example | Decision |
|---|---|---|
| AD forest | `corp.local` | `<ad-forest>` |
| AD domain | `corp.local` | `<ad-domain>` |
| Domain controller | `DC1.corp.local` | `<domain-controller>` |
| Admin workstation | `MGMT01` | `<admin-workstation>` |
| Target synced OU | `OU=Users,DC=corp,DC=local` | `<sync-scope-ou>` |
| Pilot synced OU | `OU=PilotUsers,OU=Users,DC=corp,DC=local` | `<pilot-ou>` |
| Excluded OU | `OU=Service Accounts,DC=corp,DC=local` | `<excluded-ou>` |
| Target routable UPN suffix | `contoso.com` | `<target-upn-suffix>` |
| Current internal suffix | `corp.local` | `<current-upn-suffix>` |
| Primary SMTP domain | `contoso.com` | `<smtp-domain>` |
| Evidence folder | `C:\HybridIdentity-Prep` | `<evidence-path>` |
| IdFix workstation | `MGMT01` | `<idfix-workstation>` |
| IdFix export folder | `C:\HybridIdentity-Prep\IdFix` | `<idfix-export-path>` |
| AD backup status | System State backup completed | `<backup-status>` |
| Rollback CSV | `C:\HybridIdentity-Prep\rollback-identity-attributes.csv` | `<rollback-csv>` |
| Attribute owners | IAM / Messaging / AD admin | `<attribute-owner>` |
| Change batch size | 25 to 100 users | `<batch-size>` |
| Test users | `test.user1`, `test.user2` | `<pilot-users>` |
| Cleanup approval | Change ticket | `<change-ticket>` |
| Sync timing | Before Entra Connect install | `<sync-timing>` |

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folders | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\HybridIdentity-Prep,C:\HybridIdentity-Prep\IdFix` | Evidence folders exist |
| 2 | Confirm AD PowerShell tools | Admin Workstation / DC | `Get-Module -ListAvailable ActiveDirectory` | ActiveDirectory module is available |
| 3 | Import AD module | Admin Workstation / DC | `Import-Module ActiveDirectory` | AD cmdlets load |
| 4 | Confirm forest and domain | Admin Workstation / DC | `Get-ADForest; Get-ADDomain` | Correct AD DS environment is confirmed |
| 5 | Confirm UPN suffix readiness | Admin Workstation / DC | `(Get-ADForest).UPNSuffixes` | Target routable suffix appears |
| 6 | Export all candidate users | Admin Workstation / DC | `Get-ADUser -Filter * -Properties * \| Export-Csv C:\HybridIdentity-Prep\ad-users-before.csv -NoTypeInformation` | Full user export exists |
| 7 | Export groups | Admin Workstation / DC | `Get-ADGroup -Filter * -Properties mail,proxyAddresses,mailNickname \| Export-Csv C:\HybridIdentity-Prep\ad-groups-before.csv -NoTypeInformation` | Group export exists |
| 8 | Export contacts | Admin Workstation / DC | `Get-ADObject -LDAPFilter "(objectClass=contact)" -Properties mail,proxyAddresses,mailNickname \| Export-Csv C:\HybridIdentity-Prep\ad-contacts-before.csv -NoTypeInformation` | Contact export exists |
| 9 | Create rollback CSV | Admin Workstation / DC | `Run AD_Object_Inventory_Precheck_Skeleton` | Rollback map exists |
| 10 | Detect duplicate UPNs | Admin Workstation / DC | `Run Duplicate_Attribute_Detection_Skeleton` | Duplicate UPN report is created |
| 11 | Detect duplicate mail values | Admin Workstation / DC | `Run Duplicate_Attribute_Detection_Skeleton` | Duplicate mail report is created |
| 12 | Detect duplicate proxyAddresses values | Admin Workstation / DC | `Run Duplicate_Attribute_Detection_Skeleton` | Duplicate proxy address report is created |
| 13 | Detect duplicate mailNickname values | Admin Workstation / DC | `Run Duplicate_Attribute_Detection_Skeleton` | Duplicate alias report is created |
| 14 | Detect blank UPNs | Admin Workstation / DC | `Get-ADUser -Filter * -Properties UserPrincipalName \| Where-Object {-not $_.UserPrincipalName}` | Blank UPN accounts are listed |
| 15 | Detect nonroutable UPNs | Admin Workstation / DC | `Get-ADUser -Filter "UserPrincipalName -like '*.local'" -Properties UserPrincipalName` | Nonroutable UPN accounts are listed |
| 16 | Detect invalid characters | Admin Workstation / DC | `Run Invalid_Attribute_Detection_Skeleton` | Invalid attribute report is created |
| 17 | Download and run IdFix | IdFix Workstation | `Run IdFix as domain admin or delegated account` | IdFix query results are generated |
| 18 | Export IdFix results | IdFix Workstation | `Export results to C:\HybridIdentity-Prep\IdFix` | IdFix CSV evidence is saved |
| 19 | Review IdFix proposed updates | IdFix Workstation | `Review Update column and Action column` | Proposed changes are approved or corrected |
| 20 | Apply safe pilot fixes only | IdFix Workstation / AD | `Apply changes for pilot scope first` | Pilot objects are corrected |
| 21 | Validate pilot objects | Admin Workstation / DC | `Run Post_Cleanup_Validation_Skeleton` | Pilot shows no blocking errors |
| 22 | Apply approved cleanup batches | Admin Workstation / IdFix | `Use IdFix or controlled PowerShell remediation` | Approved objects are corrected |
| 23 | Re-run duplicate checks | Admin Workstation / DC | `Run Duplicate_Attribute_Detection_Skeleton` | Duplicate reports are empty or documented |
| 24 | Re-run invalid character checks | Admin Workstation / DC | `Run Invalid_Attribute_Detection_Skeleton` | Invalid character reports are empty or documented |
| 25 | Re-run IdFix query | IdFix Workstation | `Query` | No blocking sync errors remain |
| 26 | Export final cleaned inventory | Admin Workstation / DC | `Run Post_Cleanup_Validation_Skeleton` | Final clean-state evidence exists |
| 27 | Document excluded objects | Operator | `Record service accounts, disabled users, stale users, and exceptions` | Exclusions are intentional |
| 28 | Confirm readiness for sync baseline | Operator | `Review reports and sign off` | Directory is ready for Entra Connect or Cloud Sync planning |

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_AD_Object_Inventory_Precheck_Skeleton

```powershell
# Run in elevated PowerShell on a DC or admin workstation with RSAT AD tools.
# Purpose: export current identity attribute state before IdFix or manual cleanup.

$EvidencePath = "C:\HybridIdentity-Prep"
$IdFixPath = "$EvidencePath\IdFix"
$SearchBase = "OU=Users,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath,$IdFixPath

Start-Transcript -Path "$EvidencePath\01-ad-object-inventory-precheck-transcript.txt"

Import-Module ActiveDirectory

# Forest and domain evidence.
Get-ADForest |
  Format-List * |
  Out-File "$EvidencePath\ad-forest.txt"

Get-ADDomain |
  Format-List * |
  Out-File "$EvidencePath\ad-domain.txt"

(Get-ADForest).UPNSuffixes |
  Out-File "$EvidencePath\upn-suffixes.txt"

# User inventory.
Get-ADUser -SearchBase $SearchBase -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,displayName,givenName,sn,sAMAccountName,Enabled,Description,ServicePrincipalName,whenCreated,whenChanged |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    targetAddress,
    displayName,
    givenName,
    sn,
    Description,
    ServicePrincipalName,
    whenCreated,
    whenChanged,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\ad-users-identity-before.csv" -NoTypeInformation

# Group inventory for mail-enabled or cloud-relevant groups.
Get-ADGroup -Filter * `
  -Properties mail,proxyAddresses,mailNickname,GroupCategory,GroupScope,Description |
  Select-Object `
    SamAccountName,
    Name,
    DistinguishedName,
    GroupCategory,
    GroupScope,
    mail,
    mailNickname,
    Description,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\ad-groups-identity-before.csv" -NoTypeInformation

# Contact inventory.
Get-ADObject -LDAPFilter "(objectClass=contact)" `
  -Properties mail,proxyAddresses,mailNickname,targetAddress,displayName,givenName,sn |
  Select-Object `
    Name,
    DistinguishedName,
    mail,
    mailNickname,
    targetAddress,
    displayName,
    givenName,
    sn,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\ad-contacts-identity-before.csv" -NoTypeInformation

# Rollback map for important attributes.
Get-ADUser -SearchBase $SearchBase -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,displayName,givenName,sn |
  Select-Object `
    SamAccountName,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    targetAddress,
    displayName,
    givenName,
    sn,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\rollback-identity-attributes.csv" -NoTypeInformation

Stop-Transcript
```

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Duplicate_Attribute_Detection_Skeleton

```powershell
# Run in elevated PowerShell with AD module.
# Purpose: find duplicate values that commonly block or damage sync.

$EvidencePath = "C:\HybridIdentity-Prep"
$SearchBase = "OU=Users,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-duplicate-attribute-detection-transcript.txt"

Import-Module ActiveDirectory

$Users = Get-ADUser -SearchBase $SearchBase -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,Enabled

# Duplicate UPNs.
$Users |
  Where-Object { $_.UserPrincipalName } |
  Group-Object UserPrincipalName |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicates-userPrincipalName.csv" -NoTypeInformation

# Duplicate mail values.
$Users |
  Where-Object { $_.mail } |
  Group-Object mail |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicates-mail.csv" -NoTypeInformation

# Duplicate mailNickname values.
$Users |
  Where-Object { $_.mailNickname } |
  Group-Object mailNickname |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicates-mailNickname.csv" -NoTypeInformation

# Duplicate targetAddress values.
$Users |
  Where-Object { $_.targetAddress } |
  Group-Object targetAddress |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicates-targetAddress.csv" -NoTypeInformation

# Flatten proxyAddresses and detect duplicates across users.
$ProxyRows = foreach ($User in $Users) {
  foreach ($Proxy in $User.proxyAddresses) {
    [PSCustomObject]@{
      SamAccountName = $User.SamAccountName
      DistinguishedName = $User.DistinguishedName
      ProxyAddress = $Proxy.ToLowerInvariant()
      OriginalProxyAddress = $Proxy
    }
  }
}

$ProxyRows |
  Group-Object ProxyAddress |
  Where-Object { $_.Count -gt 1 } |
  ForEach-Object {
    $DuplicateValue = $_.Name
    $_.Group | Select-Object `
      @{Name="DuplicateProxyAddress";Expression={$DuplicateValue}},
      SamAccountName,
      DistinguishedName,
      OriginalProxyAddress
  } |
  Export-Csv "$EvidencePath\duplicates-proxyAddresses.csv" -NoTypeInformation

# Include groups and contacts in proxyAddress duplicate check.
$Groups = Get-ADGroup -Filter * -Properties proxyAddresses,mail,mailNickname
$Contacts = Get-ADObject -LDAPFilter "(objectClass=contact)" -Properties proxyAddresses,mail,mailNickname

$AllProxyRows = @()

foreach ($User in $Users) {
  foreach ($Proxy in $User.proxyAddresses) {
    $AllProxyRows += [PSCustomObject]@{
      ObjectType = "User"
      Name = $User.Name
      SamAccountName = $User.SamAccountName
      DistinguishedName = $User.DistinguishedName
      ProxyAddress = $Proxy.ToLowerInvariant()
      OriginalProxyAddress = $Proxy
    }
  }
}

foreach ($Group in $Groups) {
  foreach ($Proxy in $Group.proxyAddresses) {
    $AllProxyRows += [PSCustomObject]@{
      ObjectType = "Group"
      Name = $Group.Name
      SamAccountName = $Group.SamAccountName
      DistinguishedName = $Group.DistinguishedName
      ProxyAddress = $Proxy.ToLowerInvariant()
      OriginalProxyAddress = $Proxy
    }
  }
}

foreach ($Contact in $Contacts) {
  foreach ($Proxy in $Contact.proxyAddresses) {
    $AllProxyRows += [PSCustomObject]@{
      ObjectType = "Contact"
      Name = $Contact.Name
      SamAccountName = ""
      DistinguishedName = $Contact.DistinguishedName
      ProxyAddress = $Proxy.ToLowerInvariant()
      OriginalProxyAddress = $Proxy
    }
  }
}

$AllProxyRows |
  Group-Object ProxyAddress |
  Where-Object { $_.Count -gt 1 } |
  ForEach-Object {
    $DuplicateValue = $_.Name
    $_.Group | Select-Object `
      @{Name="DuplicateProxyAddress";Expression={$DuplicateValue}},
      ObjectType,
      Name,
      SamAccountName,
      DistinguishedName,
      OriginalProxyAddress
  } |
  Export-Csv "$EvidencePath\duplicates-proxyAddresses-users-groups-contacts.csv" -NoTypeInformation

Stop-Transcript
```

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Invalid_Attribute_Detection_Skeleton

```powershell
# Run in elevated PowerShell with AD module.
# Purpose: find blank, malformed, nonroutable, or invalid sync-related values.

$EvidencePath = "C:\HybridIdentity-Prep"
$SearchBase = "OU=Users,DC=corp,DC=local"
$TargetUpnSuffix = "contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-invalid-attribute-detection-transcript.txt"

Import-Module ActiveDirectory

$Users = Get-ADUser -SearchBase $SearchBase -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,displayName,givenName,sn,sAMAccountName,Enabled,Description

# Blank UPN.
$Users |
  Where-Object { [string]::IsNullOrWhiteSpace($_.UserPrincipalName) } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\invalid-blank-upn.csv" -NoTypeInformation

# Nonroutable UPN suffixes.
$Users |
  Where-Object {
    $_.UserPrincipalName -match "\.local$|\.lan$|\.internal$|\.corp$"
  } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\invalid-nonroutable-upn-suffix.csv" -NoTypeInformation

# UPN does not use target suffix.
$Users |
  Where-Object {
    $_.UserPrincipalName -and
    $_.UserPrincipalName -notlike "*@$TargetUpnSuffix"
  } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\upn-not-using-target-suffix.csv" -NoTypeInformation

# UPN contains spaces or obvious invalid characters.
$Users |
  Where-Object {
    $_.UserPrincipalName -match "\s|[\\%&\*\+/=\?\{\}\|<>\(\);:,\[\]`"]"
  } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\invalid-upn-characters.csv" -NoTypeInformation

# mail is blank while user is enabled.
$Users |
  Where-Object {
    $_.Enabled -eq $true -and [string]::IsNullOrWhiteSpace($_.mail)
  } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\enabled-users-blank-mail.csv" -NoTypeInformation

# mail format looks wrong.
$Users |
  Where-Object {
    $_.mail -and $_.mail -notmatch "^[^@\s]+@[^@\s]+\.[^@\s]+$"
  } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\invalid-mail-format.csv" -NoTypeInformation

# mailNickname begins with period or contains bad characters.
$Users |
  Where-Object {
    $_.mailNickname -and (
      $_.mailNickname.StartsWith(".") -or
      $_.mailNickname -match "[\\`"\|,/:<>\+=;\?\*'\s]"
    )
  } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,mailNickname,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\invalid-mailNickname.csv" -NoTypeInformation

# sAMAccountName longer than 20 characters.
$Users |
  Where-Object {
    $_.sAMAccountName.Length -gt 20
  } |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName |
  Export-Csv "$EvidencePath\invalid-samaccountname-length.csv" -NoTypeInformation

# proxyAddresses containing spaces or obvious invalid characters.
$ProxyIssues = foreach ($User in $Users) {
  foreach ($Proxy in $User.proxyAddresses) {
    if ($Proxy -match "\s|[<>\(\);,\[\]`"]") {
      [PSCustomObject]@{
        SamAccountName = $User.SamAccountName
        Name = $User.Name
        DistinguishedName = $User.DistinguishedName
        ProxyAddress = $Proxy
        Issue = "Invalid character or space"
      }
    }
  }
}

$ProxyIssues |
  Export-Csv "$EvidencePath\invalid-proxyAddresses.csv" -NoTypeInformation

# Multiple primary SMTP values.
$PrimarySmtpIssues = foreach ($User in $Users) {
  $PrimaryValues = @($User.proxyAddresses | Where-Object { $_ -cmatch "^SMTP:" })

  if ($PrimaryValues.Count -gt 1) {
    [PSCustomObject]@{
      SamAccountName = $User.SamAccountName
      Name = $User.Name
      DistinguishedName = $User.DistinguishedName
      PrimarySmtpValues = ($PrimaryValues -join ";")
      Issue = "Multiple primary SMTP values"
    }
  }
}

$PrimarySmtpIssues |
  Export-Csv "$EvidencePath\invalid-multiple-primary-smtp.csv" -NoTypeInformation

Stop-Transcript
```

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_IdFix_Runbook_Skeleton

```text
# IdFix operational runbook.
# This is intentionally written as operator steps because IdFix is normally an interactive tool.

1. Sign in to the IdFix workstation with an account that can read the target AD DS objects.
2. Confirm the workstation can reach a domain controller.
3. Confirm .NET and required Windows desktop components are present.
4. Download IdFix from the official Microsoft source.
5. Start IdFix.
6. Select Query.
7. Wait for IdFix to scan the directory.
8. Export results to:
   C:\HybridIdentity-Prep\IdFix\idfix-results-before.csv
9. Review each result before applying changes.
10. Pay special attention to:
   - duplicate userPrincipalName
   - duplicate proxyAddresses
   - duplicate mail
   - duplicate mailNickname
   - invalid characters
   - format errors
   - top-level domain issues
11. Do not apply bulk fixes blindly.
12. For each result, decide:
   - Edit
   - Remove
   - Complete
   - Do not change
13. Pilot changes first against known test users.
14. Export IdFix results after pilot cleanup.
15. Re-run Query.
16. Confirm pilot errors are gone.
17. Apply approved production batches.
18. Export final IdFix results to:
   C:\HybridIdentity-Prep\IdFix\idfix-results-after.csv
19. Save screenshots or export files into the evidence folder.
20. Do not proceed to Entra Connect Sync until blocking IdFix findings are cleared or documented as intentional exceptions.
```

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Cleanup_Staging_And_Remediation_Skeleton

```powershell
# Controlled remediation examples.
# Do not run this as a blind bulk cleanup script.
# Edit object names and values intentionally.

$EvidencePath = "C:\HybridIdentity-Prep"

Start-Transcript -Path "$EvidencePath\04-cleanup-staging-remediation-transcript.txt"

Import-Module ActiveDirectory

# Example 1: Set a blank UPN to a routable UPN.
Set-ADUser `
  -Identity "jdoe" `
  -UserPrincipalName "jdoe@contoso.com"

# Example 2: Align UPN to primary email address.
$User = Get-ADUser -Identity "asmith" -Properties mail
Set-ADUser `
  -Identity $User.SamAccountName `
  -UserPrincipalName $User.mail

# Example 3: Correct mailNickname.
Set-ADUser `
  -Identity "bwhite" `
  -Replace @{mailNickname = "bwhite"}

# Example 4: Correct mail attribute.
Set-ADUser `
  -Identity "bwhite" `
  -EmailAddress "bwhite@contoso.com"

# Example 5: Replace proxyAddresses safely.
# Preserve all known valid aliases, and only remove the bad or duplicate value.
$User = Get-ADUser -Identity "bwhite" -Properties proxyAddresses

$NewProxyAddresses = @(
  "SMTP:bwhite@contoso.com",
  "smtp:ben.white@contoso.com"
)

Set-ADUser `
  -Identity $User.SamAccountName `
  -Replace @{proxyAddresses = $NewProxyAddresses}

# Example 6: Remove one bad proxyAddress without destroying the entire attribute.
Set-ADUser `
  -Identity "svc-oldapp" `
  -Remove @{proxyAddresses = "smtp:duplicate@contoso.com"}

# Example 7: Add a corrected proxyAddress.
Set-ADUser `
  -Identity "svc-oldapp" `
  -Add @{proxyAddresses = "smtp:svc-oldapp@contoso.com"}

# Example 8: Disable or move stale users out of sync scope instead of cleaning every abandoned object.
# Move-ADObject -Identity "<distinguished-name>" -TargetPath "OU=Excluded From Sync,DC=corp,DC=local"

Stop-Transcript
```

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Post_Cleanup_Validation_Skeleton

```powershell
# Run after IdFix and manual remediation.
# Purpose: prove that the directory is ready for sync scoping and Entra Connect planning.

$EvidencePath = "C:\HybridIdentity-Prep"
$SearchBase = "OU=Users,DC=corp,DC=local"
$TargetUpnSuffix = "contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-post-cleanup-validation-transcript.txt"

Import-Module ActiveDirectory

$Users = Get-ADUser -SearchBase $SearchBase -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress,displayName,givenName,sn,sAMAccountName,Enabled,Description

# Export final inventory.
$Users |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    targetAddress,
    displayName,
    givenName,
    sn,
    Description,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\ad-users-identity-after-cleanup.csv" -NoTypeInformation

# Final blocking checks.
$BlankUpn = $Users | Where-Object { [string]::IsNullOrWhiteSpace($_.UserPrincipalName) }
$NonTargetUpn = $Users | Where-Object { $_.Enabled -and $_.UserPrincipalName -notlike "*@$TargetUpnSuffix" }
$DuplicateUpn = $Users | Where-Object { $_.UserPrincipalName } | Group-Object UserPrincipalName | Where-Object Count -gt 1
$DuplicateMail = $Users | Where-Object { $_.mail } | Group-Object mail | Where-Object Count -gt 1
$DuplicateAlias = $Users | Where-Object { $_.mailNickname } | Group-Object mailNickname | Where-Object Count -gt 1

$Summary = [PSCustomObject]@{
  TotalUsers = ($Users | Measure-Object).Count
  BlankUpnCount = ($BlankUpn | Measure-Object).Count
  NonTargetUpnCount = ($NonTargetUpn | Measure-Object).Count
  DuplicateUpnCount = ($DuplicateUpn | Measure-Object).Count
  DuplicateMailCount = ($DuplicateMail | Measure-Object).Count
  DuplicateMailNicknameCount = ($DuplicateAlias | Measure-Object).Count
}

$Summary |
  Format-List * |
  Tee-Object "$EvidencePath\post-cleanup-summary.txt"

$BlankUpn |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,DistinguishedName |
  Export-Csv "$EvidencePath\post-cleanup-blank-upn.csv" -NoTypeInformation

$NonTargetUpn |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,DistinguishedName |
  Export-Csv "$EvidencePath\post-cleanup-non-target-upn.csv" -NoTypeInformation

$DuplicateUpn |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\post-cleanup-duplicate-upn.csv" -NoTypeInformation

$DuplicateMail |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\post-cleanup-duplicate-mail.csv" -NoTypeInformation

$DuplicateAlias |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\post-cleanup-duplicate-mailNickname.csv" -NoTypeInformation

Stop-Transcript
```

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-ADForest` | Confirms forest and UPN suffix state | Correct forest and target UPN suffix are visible |
| `(Get-ADForest).UPNSuffixes` | Confirms routable UPN suffix exists | `contoso.com` appears |
| `Get-ADUser -Filter * -Properties UserPrincipalName \| Where-Object {-not $_.UserPrincipalName}` | Finds blank UPNs | No normal synced users return |
| `Get-ADUser -Filter "UserPrincipalName -like '*.local'" -Properties UserPrincipalName` | Finds nonroutable UPNs | No in-scope synced users return |
| `Get-ADUser -Filter * -Properties UserPrincipalName \| Group-Object UserPrincipalName \| Where-Object Count -gt 1` | Detects duplicate UPN values | No duplicates return |
| `Get-ADUser -Filter * -Properties mail \| Group-Object mail \| Where-Object Count -gt 1` | Detects duplicate mail values | No duplicates return |
| `Get-ADUser -Filter * -Properties mailNickname \| Group-Object mailNickname \| Where-Object Count -gt 1` | Detects duplicate aliases | No duplicates return |
| `Get-ADUser -Filter * -Properties proxyAddresses` | Reviews proxy address state | Values are unique and valid |
| `Resolve-DnsName contoso.com -Type TXT` | Confirms public domain DNS is queryable | Domain records resolve |
| `repadmin /replsummary` | Confirms AD replication health | No major replication failures |
| `dcdiag /test:dns` | Confirms domain DNS baseline | DNS tests pass or known issues are documented |
| `Import-Csv C:\HybridIdentity-Prep\rollback-identity-attributes.csv` | Confirms rollback map exists | Original attribute values are available |
| `Get-Content C:\HybridIdentity-Prep\post-cleanup-summary.txt` | Reviews cleanup summary | Blocking issue counts are zero or documented |

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Rollback

```powershell
# Rollback selected user identity attributes from rollback CSV.
# Use only after approval.
# This example restores userPrincipalName, mail, mailNickname, targetAddress, displayName, givenName, sn, and proxyAddresses.

$EvidencePath = "C:\HybridIdentity-Prep"
$RollbackCsv = "$EvidencePath\rollback-identity-attributes.csv"

Start-Transcript -Path "$EvidencePath\rollback-identity-attributes-transcript.txt"

Import-Module ActiveDirectory

$RollbackRows = Import-Csv $RollbackCsv

foreach ($Row in $RollbackRows) {
  Write-Host "Restoring identity attributes for $($Row.SamAccountName)"

  $Replace = @{}

  if ($Row.UserPrincipalName) { $Replace["userPrincipalName"] = $Row.UserPrincipalName }
  if ($Row.mail) { $Replace["mail"] = $Row.mail }
  if ($Row.mailNickname) { $Replace["mailNickname"] = $Row.mailNickname }
  if ($Row.targetAddress) { $Replace["targetAddress"] = $Row.targetAddress }
  if ($Row.displayName) { $Replace["displayName"] = $Row.displayName }
  if ($Row.givenName) { $Replace["givenName"] = $Row.givenName }
  if ($Row.sn) { $Replace["sn"] = $Row.sn }

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

Get-ADUser -Filter * -Properties UserPrincipalName,mail,proxyAddresses,mailNickname,targetAddress |
  Select-Object `
    SamAccountName,
    Name,
    DistinguishedName,
    UserPrincipalName,
    mail,
    mailNickname,
    targetAddress,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\ad-users-identity-after-rollback.csv" -NoTypeInformation

Stop-Transcript
```

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| IdFix shows duplicate UPN | Two users share the same `userPrincipalName` | Duplicate UPN report | Change one UPN to a valid unique routable value |
| IdFix shows duplicate proxy address | Alias exists on more than one user, group, or contact | `duplicates-proxyAddresses-users-groups-contacts.csv` | Remove or correct the duplicate alias on the wrong object |
| IdFix shows invalid characters | Attribute contains unsupported characters | Invalid attribute reports | Replace invalid values with supported format |
| User has `.local` UPN | AD still uses internal-only suffix | `invalid-nonroutable-upn-suffix.csv` | Change UPN to verified routable suffix |
| User has blank UPN | Legacy account was created without UPN | `invalid-blank-upn.csv` | Assign valid `user@contoso.com` UPN |
| mail and UPN do not match | Historical naming mismatch | Compare `mail` and `UserPrincipalName` | Align unless there is a documented exception |
| Multiple primary SMTP values | More than one `SMTP:` value exists in proxyAddresses | `invalid-multiple-primary-smtp.csv` | Keep one primary `SMTP:` and make others lowercase `smtp:` |
| SPF or DNS looks unrelated but sync readiness fails later | Domain was not verified or UPN suffix is not cloud-ready | `Resolve-DnsName contoso.com -Type TXT` and M365 domain check | Verify custom domain before sync |
| Cleanup breaks application login | Service account UPN or alias changed without app owner approval | Review rollback CSV and app config | Restore service account value and exclude it |
| Cleanup does not appear on another DC | AD replication delay or failure | `repadmin /replsummary` | Fix replication and wait before sync |
| IdFix query fails | Tool cannot contact DC or account lacks rights | Event logs, network, DNS, domain auth | Run from domain-joined workstation with proper rights |
| PowerShell export is empty | Wrong SearchBase or insufficient permissions | Check `$SearchBase`; run `Get-ADUser -Filter *` | Correct OU path or permissions |
| Duplicate report includes disabled stale users | Old accounts are still in sync scope | Review `Enabled` and `DistinguishedName` | Move stale users to excluded OU or clean them |
| mailNickname is duplicated | Alias reused across users or groups | Duplicate alias report | Assign unique alias values |
| targetAddress conflict | Hybrid or migration routing address duplicated | Duplicate targetAddress report | Correct routing address before hybrid mail work |
| Rollback fails for proxyAddresses | Existing values conflict with another object | Duplicate proxy report | Remove conflicting value first, then restore intended object |

# 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md | Provides the routable UPN suffix that cleaned users should use |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md | Confirms the cloud domain is verified before sync |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md | Uses the cleaned AD DS objects as the sync source |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md | Troubleshoots remaining sync issues after first sync |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md | Depends on clean identity attributes for user recovery and sign-in |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md | Protects sign-ins after users are synced |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments.md | Builds on cleaned groups and users for cloud access assignment |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md | Uses cleanup evidence when diagnosing sign-in and licensing issues |