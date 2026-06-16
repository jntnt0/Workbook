# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Index

01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Source_Basis  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Mental_Model  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Planning_Table  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Configuration_Checklist  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_AD_DS_And_DNS_Precheck_Skeleton  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_UPN_Suffix_Add_Skeleton  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_UPN_Inventory_And_Dry_Run_Skeleton  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Bulk_UPN_Update_Skeleton  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_DNS_And_Domain_Validation_Skeleton  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Verification_Commands  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Rollback  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Failure_Checks  
01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Related_Labs  

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft 365 Docs | Prepare a nonroutable domain for directory synchronization | Explains why `.local` and other nonroutable UPN suffixes should be changed before syncing to Microsoft 365 |
| Microsoft 365 Docs | Prepare for directory synchronization to Microsoft 365 | Supports AD DS cleanup, UPN uniqueness, routable domain planning, and attribute preparation |
| Microsoft 365 Docs | Add a custom domain to Microsoft 365 | Supports verified domain planning and DNS ownership validation |
| Microsoft Entra Docs | Hybrid identity planning | Supports cloud identity, synchronized identity, and federated identity decision points |
| Windows Server AD DS | Active Directory Domains and Trusts | Supports adding alternate UPN suffixes |
| Windows Server ActiveDirectory PowerShell module | Get-ADForest, Set-ADForest, Get-ADUser, Set-ADUser | Supports inventory, adding UPN suffixes, and updating user UPN values |
| Windows DNS / public DNS tools | Resolve-DnsName, nslookup | Supports validation of public routable domains and DNS records |
| Operational practice | Change control, pilot OU, export before change, rollback CSV | Prevents bulk identity changes without evidence and rollback path |

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Mental_Model

| Concept | Operational Meaning |
|---|---|
| On-prem AD DS domain | Internal directory namespace such as `corp.local`, `ad.contoso.com`, or `corp.contoso.com` |
| Nonroutable domain | Internal-only DNS suffix such as `.local`, `.lan`, or `.internal` that cannot be verified as a public Microsoft 365 domain |
| Routable domain | Publicly registered DNS domain such as `contoso.com` that can be verified in Microsoft Entra ID / Microsoft 365 |
| UPN | User Principal Name, normally formatted like an email address, for example `user@contoso.com` |
| UPN suffix | Portion to the right of `@`, such as `contoso.com` |
| Alternate UPN suffix | Additional UPN suffix registered in AD DS so users can sign in with a cloud-ready routable suffix |
| Primary SMTP address | Main email address, often expected to match the UPN for user clarity |
| Verified cloud domain | Domain proven in Microsoft 365 or Microsoft Entra ID by adding DNS verification records |
| Initial tenant domain | Default `tenant.onmicrosoft.com` domain created with the tenant |
| Identity sync readiness | AD DS object state where user UPNs, proxyAddresses, mail, and names are unique, valid, and cloud-compatible |
| Pilot scope | Small test set of users used before changing every user |
| First rule | Do not sync users with `.local` UPNs unless you intentionally accept ugly `.onmicrosoft.com` sign-ins |
| Blunt rule | For most normal businesses, user UPN should match the primary email address unless there is a real reason not to |

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Planning_Table

| Item | Example | Decision |
|---|---|---|
| AD forest | `corp.local` | `<ad-forest>` |
| AD domain | `corp.local` | `<ad-domain>` |
| Domain controller | `DC1.corp.local` | `<domain-controller>` |
| Existing internal UPN suffix | `corp.local` | `<current-upn-suffix>` |
| Target routable UPN suffix | `contoso.com` | `<target-upn-suffix>` |
| Microsoft 365 tenant initial domain | `contoso.onmicrosoft.com` | `<tenant-onmicrosoft-domain>` |
| Verified cloud domain target | `contoso.com` | `<verified-domain>` |
| Public DNS host | Cloudflare / GoDaddy / Azure DNS | `<public-dns-host>` |
| Registrar | GoDaddy / Namecheap / Microsoft | `<registrar>` |
| Admin workstation | `MGMT01` | `<admin-workstation>` |
| AD admin account | `CORP\Domain Admin` | `<ad-admin-account>` |
| Microsoft 365 admin role needed | Domain Name Administrator / Global Administrator | `<cloud-admin-role>` |
| Pilot OU | `OU=PilotUsers,OU=Users,DC=corp,DC=local` | `<pilot-ou>` |
| Production user OU | `OU=Users,DC=corp,DC=local` | `<production-ou>` |
| Excluded accounts | Break glass, service accounts, sync accounts | `<excluded-accounts>` |
| Evidence folder | `C:\HybridIdentity-Prep` | `<evidence-path>` |
| UPN export CSV | `C:\HybridIdentity-Prep\upn-inventory-before.csv` | `<upn-export>` |
| Rollback CSV | `C:\HybridIdentity-Prep\upn-rollback-map.csv` | `<rollback-csv>` |
| Change window | After business hours | `<change-window>` |
| Rollback stance | Restore original UPNs from CSV | `<rollback-plan>` |

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell with AD tools | Admin Workstation / DC | `whoami /groups` | Admin context is confirmed |
| 2 | Create evidence folder | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\HybridIdentity-Prep` | Evidence folder exists |
| 3 | Confirm Active Directory module is available | Admin Workstation / DC | `Get-Module -ListAvailable ActiveDirectory` | ActiveDirectory module is available |
| 4 | Import AD module | Admin Workstation / DC | `Import-Module ActiveDirectory` | AD cmdlets load |
| 5 | Confirm forest identity | Admin Workstation / DC | `Get-ADForest` | Forest name and UPN suffixes are visible |
| 6 | Confirm domain identity | Admin Workstation / DC | `Get-ADDomain` | Domain DNSRoot and NetBIOS name are visible |
| 7 | Export current forest UPN suffixes | Admin Workstation / DC | `(Get-ADForest).UPNSuffixes` | Current alternate UPN suffixes are recorded |
| 8 | Confirm target routable domain is not already present | Admin Workstation / DC | `(Get-ADForest).UPNSuffixes -contains "contoso.com"` | Duplicate UPN suffix risk is known |
| 9 | Add target alternate UPN suffix | Admin Workstation / DC | `Set-ADForest -Identity (Get-ADForest).Name -UPNSuffixes @{Add="contoso.com"}` | Target suffix is added to AD forest |
| 10 | Verify target UPN suffix exists | Admin Workstation / DC | `(Get-ADForest).UPNSuffixes` | `contoso.com` appears |
| 11 | Inventory users and current UPNs | Admin Workstation / DC | `Get-ADUser -Filter * -Properties UserPrincipalName,mail,proxyAddresses` | Current identity state is visible |
| 12 | Export full UPN inventory | Admin Workstation / DC | `Get-ADUser -Filter * -Properties UserPrincipalName,mail,proxyAddresses \| Export-Csv C:\HybridIdentity-Prep\upn-inventory-before.csv -NoTypeInformation` | Pre-change inventory exists |
| 13 | Find users with nonroutable UPN suffixes | Admin Workstation / DC | `Get-ADUser -Filter "UserPrincipalName -like '*corp.local'" -Properties UserPrincipalName` | Users needing UPN cleanup are identified |
| 14 | Find blank UPNs | Admin Workstation / DC | `Get-ADUser -Filter * -Properties UserPrincipalName \| Where-Object {-not $_.UserPrincipalName}` | Blank UPN accounts are identified |
| 15 | Find duplicate UPNs | Admin Workstation / DC | `Get-ADUser -Filter * -Properties UserPrincipalName \| Group-Object UserPrincipalName \| Where-Object Count -gt 1` | Duplicate UPN values are identified |
| 16 | Find likely service accounts | Admin Workstation / DC | `Get-ADUser -Filter * -Properties ServicePrincipalName,Description \| Where-Object {$_.ServicePrincipalName -or $_.Description -match "service"}` | Accounts that should not be bulk changed blindly are identified |
| 17 | Create rollback mapping CSV | Admin Workstation / DC | `Get-ADUser -Filter * -Properties UserPrincipalName \| Select SamAccountName,DistinguishedName,UserPrincipalName \| Export-Csv C:\HybridIdentity-Prep\upn-rollback-map.csv -NoTypeInformation` | Rollback map exists |
| 18 | Run pilot UPN dry run | Admin Workstation / DC | `Get-ADUser -SearchBase "<pilot-ou>" -Filter "UserPrincipalName -like '*corp.local'" -Properties UserPrincipalName` | Pilot change scope is visible |
| 19 | Update pilot users to routable suffix | Admin Workstation / DC | `Set-ADUser -Identity "<samaccountname>" -UserPrincipalName "<user>@contoso.com"` | Pilot user UPN changes |
| 20 | Verify pilot UPNs | Admin Workstation / DC | `Get-ADUser -SearchBase "<pilot-ou>" -Filter * -Properties UserPrincipalName \| Select Name,UserPrincipalName` | Pilot users show target suffix |
| 21 | Bulk update remaining approved users | Admin Workstation / DC | `Use controlled script from Bulk_UPN_Update_Skeleton` | Approved users receive routable UPN suffix |
| 22 | Export post-change inventory | Admin Workstation / DC | `Get-ADUser -Filter * -Properties UserPrincipalName,mail,proxyAddresses \| Export-Csv C:\HybridIdentity-Prep\upn-inventory-after.csv -NoTypeInformation` | After-state inventory exists |
| 23 | Validate no old suffix remains in target scope | Admin Workstation / DC | `Get-ADUser -Filter "UserPrincipalName -like '*corp.local'" -Properties UserPrincipalName` | No unexpected old UPN suffix remains |
| 24 | Validate target public domain resolves | Admin Workstation | `Resolve-DnsName contoso.com -Type NS` | Public namespace has authoritative NS records |
| 25 | Validate domain can be verified in cloud | Microsoft 365 Admin Center | `Settings > Domains > Add domain` | Domain verification path is ready |
| 26 | Record DNS verification requirements | Microsoft 365 Admin Center / DNS Host | `Document TXT or MX verification value` | Required DNS validation record is known |
| 27 | Confirm no MX cutover is performed in this task | DNS Host | `Review current MX records only` | Mail flow is not changed during identity prep |
| 28 | Document completion state | Operator | `Record suffix added, users changed, excluded accounts, validation results, rollback CSV` | Task evidence is complete |

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_AD_DS_And_DNS_Precheck_Skeleton

```powershell
# Run in elevated PowerShell on a domain controller or admin workstation with RSAT AD tools.
# Purpose: collect AD DS, DNS, and UPN readiness evidence before making changes.

$EvidencePath = "C:\HybridIdentity-Prep"
$TargetUpnSuffix = "contoso.com"
$CurrentInternalSuffix = "corp.local"
$DomainController = "DC1"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-precheck-transcript.txt"

# Confirm execution context.
whoami /all | Out-File "$EvidencePath\whoami-all.txt"

# Confirm AD module.
Get-Module -ListAvailable ActiveDirectory |
  Out-File "$EvidencePath\active-directory-module.txt"

Import-Module ActiveDirectory

# Capture forest and domain state.
Get-ADForest |
  Format-List * |
  Out-File "$EvidencePath\ad-forest-before.txt"

Get-ADDomain |
  Format-List * |
  Out-File "$EvidencePath\ad-domain-before.txt"

# Capture existing UPN suffixes.
(Get-ADForest).UPNSuffixes |
  Out-File "$EvidencePath\upn-suffixes-before.txt"

# Capture domain controller state.
Get-ADDomainController -Filter * |
  Select-Object HostName,Site,IPv4Address,IsGlobalCatalog,OperatingSystem |
  Export-Csv "$EvidencePath\domain-controllers.csv" -NoTypeInformation

# Capture DNS baseline.
Get-DnsClientServerAddress -AddressFamily IPv4 |
  Out-File "$EvidencePath\dns-client-server-addresses.txt"

Resolve-DnsName $TargetUpnSuffix -Type NS -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\public-domain-ns-lookup.txt"

Resolve-DnsName $TargetUpnSuffix -Type MX -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\public-domain-mx-lookup.txt"

# Capture users and identity attributes.
Get-ADUser -Filter * `
  -Properties UserPrincipalName,mail,proxyAddresses,Enabled,Description,ServicePrincipalName |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail,Description,ServicePrincipalName |
  Export-Csv "$EvidencePath\ad-users-identity-inventory-before.csv" -NoTypeInformation

# Identify nonroutable UPN users.
Get-ADUser -Filter "UserPrincipalName -like '*$CurrentInternalSuffix'" `
  -Properties UserPrincipalName,mail,Enabled |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,DistinguishedName |
  Export-Csv "$EvidencePath\users-with-current-internal-upn-suffix.csv" -NoTypeInformation

# Identify blank UPN users.
Get-ADUser -Filter * -Properties UserPrincipalName,mail,Enabled |
  Where-Object { [string]::IsNullOrWhiteSpace($_.UserPrincipalName) } |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,DistinguishedName |
  Export-Csv "$EvidencePath\users-with-blank-upn.csv" -NoTypeInformation

# Identify duplicate UPN values.
Get-ADUser -Filter * -Properties UserPrincipalName |
  Where-Object { $_.UserPrincipalName } |
  Group-Object UserPrincipalName |
  Where-Object { $_.Count -gt 1 } |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\duplicate-upn-values.csv" -NoTypeInformation

Stop-Transcript
```

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_UPN_Suffix_Add_Skeleton

```powershell
# Run in elevated PowerShell using an account allowed to modify forest UPN suffixes.
# Purpose: add the target routable UPN suffix to AD DS.

$EvidencePath = "C:\HybridIdentity-Prep"
$TargetUpnSuffix = "contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-add-upn-suffix-transcript.txt"

Import-Module ActiveDirectory

$Forest = Get-ADForest
$ForestName = $Forest.Name

# Show current UPN suffixes.
$Forest.UPNSuffixes |
  Tee-Object "$EvidencePath\upn-suffixes-before-add.txt"

# Add the suffix only if it does not already exist.
if ($Forest.UPNSuffixes -notcontains $TargetUpnSuffix) {
  Set-ADForest `
    -Identity $ForestName `
    -UPNSuffixes @{Add = $TargetUpnSuffix}
}

# Verify suffix after change.
(Get-ADForest).UPNSuffixes |
  Tee-Object "$EvidencePath\upn-suffixes-after-add.txt"

Stop-Transcript
```

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_UPN_Inventory_And_Dry_Run_Skeleton

```powershell
# Run in elevated PowerShell on a DC or admin workstation.
# Purpose: create a safe dry-run list before bulk changing UPNs.

$EvidencePath = "C:\HybridIdentity-Prep"
$CurrentInternalSuffix = "corp.local"
$TargetUpnSuffix = "contoso.com"
$PilotSearchBase = "OU=PilotUsers,OU=Users,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module ActiveDirectory

# Build rollback map before any user change.
Get-ADUser -Filter * -Properties UserPrincipalName,mail,Enabled |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\upn-rollback-map.csv" -NoTypeInformation

# Pilot dry run.
$PilotUsers = Get-ADUser `
  -SearchBase $PilotSearchBase `
  -Filter "UserPrincipalName -like '*$CurrentInternalSuffix'" `
  -Properties UserPrincipalName,mail,Enabled

$PilotUsers |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    DistinguishedName,
    UserPrincipalName,
    mail,
    @{Name="ProposedUPN";Expression={$_.UserPrincipalName.Replace("@$CurrentInternalSuffix","@$TargetUpnSuffix")}} |
  Export-Csv "$EvidencePath\pilot-upn-change-dry-run.csv" -NoTypeInformation

# Full dry run, excluding common service style accounts by description keyword.
$AllTargetUsers = Get-ADUser `
  -Filter "UserPrincipalName -like '*$CurrentInternalSuffix'" `
  -Properties UserPrincipalName,mail,Enabled,Description,ServicePrincipalName

$AllTargetUsers |
  Where-Object {
    -not $_.ServicePrincipalName -and
    $_.Description -notmatch "service|sync|break glass|emergency"
  } |
  Select-Object `
    SamAccountName,
    Name,
    Enabled,
    DistinguishedName,
    UserPrincipalName,
    mail,
    Description,
    @{Name="ProposedUPN";Expression={$_.UserPrincipalName.Replace("@$CurrentInternalSuffix","@$TargetUpnSuffix")}} |
  Export-Csv "$EvidencePath\approved-style-upn-change-dry-run.csv" -NoTypeInformation
```

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Bulk_UPN_Update_Skeleton

```powershell
# Run only after dry-run CSV has been reviewed.
# Purpose: change approved user UPN suffixes from nonroutable suffix to routable suffix.

$EvidencePath = "C:\HybridIdentity-Prep"
$CurrentInternalSuffix = "corp.local"
$TargetUpnSuffix = "contoso.com"
$ApprovedInputCsv = "$EvidencePath\approved-style-upn-change-dry-run.csv"

Start-Transcript -Path "$EvidencePath\03-bulk-upn-update-transcript.txt"

Import-Module ActiveDirectory

$ApprovedUsers = Import-Csv $ApprovedInputCsv

foreach ($User in $ApprovedUsers) {
  $OldUpn = $User.UserPrincipalName
  $NewUpn = $User.ProposedUPN

  if ($OldUpn -and $NewUpn -and $OldUpn -ne $NewUpn) {
    Write-Host "Updating $($User.SamAccountName): $OldUpn to $NewUpn"

    Set-ADUser `
      -Identity $User.SamAccountName `
      -UserPrincipalName $NewUpn
  }
}

# Export post-change state.
Get-ADUser -Filter * -Properties UserPrincipalName,mail,Enabled |
  Select-Object SamAccountName,Name,Enabled,DistinguishedName,UserPrincipalName,mail |
  Export-Csv "$EvidencePath\ad-users-identity-inventory-after.csv" -NoTypeInformation

# Verify old suffix remains only where expected.
Get-ADUser -Filter "UserPrincipalName -like '*$CurrentInternalSuffix'" `
  -Properties UserPrincipalName,mail,Enabled,Description |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,Description,DistinguishedName |
  Export-Csv "$EvidencePath\remaining-users-with-old-upn-suffix.csv" -NoTypeInformation

Stop-Transcript
```

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_DNS_And_Domain_Validation_Skeleton

```powershell
# Run from any management host with internet DNS resolution.
# Purpose: validate routable public domain DNS posture before Microsoft 365 verification.

$EvidencePath = "C:\HybridIdentity-Prep"
$TargetDomain = "contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-domain-dns-validation-transcript.txt"

# Confirm public NS records.
Resolve-DnsName $TargetDomain -Type NS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$TargetDomain-ns-records.txt"

# Confirm public SOA record.
Resolve-DnsName $TargetDomain -Type SOA -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$TargetDomain-soa-record.txt"

# Review current MX. Do not change MX in this task.
Resolve-DnsName $TargetDomain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$TargetDomain-mx-records-before-m365.txt"

# Review current TXT records.
Resolve-DnsName $TargetDomain -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$TargetDomain-txt-records-before-m365.txt"

# Placeholder for Microsoft 365 verification TXT record after it is issued by the portal.
# Example only:
# Resolve-DnsName $TargetDomain -Type TXT | Where-Object {$_.Strings -match "MS=ms"}

Stop-Transcript
```

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-ADForest` | Confirms forest and alternate UPN suffixes | Forest returns and target UPN suffix appears |
| `(Get-ADForest).UPNSuffixes` | Lists alternate UPN suffixes | `contoso.com` appears |
| `Get-ADDomain` | Confirms AD domain identity | Expected DNSRoot and NetBIOS name return |
| `Get-ADUser -Filter "UserPrincipalName -like '*corp.local'" -Properties UserPrincipalName` | Finds users still using old internal suffix | Only excluded accounts remain, or no users return |
| `Get-ADUser -Filter * -Properties UserPrincipalName \| Where-Object {-not $_.UserPrincipalName}` | Finds blank UPNs | No normal user accounts return |
| `Get-ADUser -Filter * -Properties UserPrincipalName \| Group-Object UserPrincipalName \| Where-Object Count -gt 1` | Finds duplicate UPNs | No duplicate UPN groups return |
| `Resolve-DnsName contoso.com -Type NS` | Confirms public domain has authoritative name servers | NS records return |
| `Resolve-DnsName contoso.com -Type TXT` | Confirms TXT records can be queried | TXT records return after configured |
| `Resolve-DnsName contoso.com -Type MX` | Reviews current mail routing | Existing mail routing is understood |
| `Get-Content C:\HybridIdentity-Prep\upn-suffixes-after-add.txt` | Confirms evidence was saved | Target suffix appears in evidence |
| `Import-Csv C:\HybridIdentity-Prep\upn-rollback-map.csv` | Confirms rollback data exists | Original user UPN values are available |

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Rollback

```powershell
# Rollback user UPN values from the rollback CSV.
# Run only if the UPN change caused sign-in or application impact and rollback is approved.

$EvidencePath = "C:\HybridIdentity-Prep"
$RollbackCsv = "$EvidencePath\upn-rollback-map.csv"

Start-Transcript -Path "$EvidencePath\rollback-upn-values-transcript.txt"

Import-Module ActiveDirectory

$RollbackUsers = Import-Csv $RollbackCsv

foreach ($User in $RollbackUsers) {
  if ($User.SamAccountName -and $User.UserPrincipalName) {
    Write-Host "Restoring $($User.SamAccountName) to $($User.UserPrincipalName)"

    Set-ADUser `
      -Identity $User.SamAccountName `
      -UserPrincipalName $User.UserPrincipalName
  }
}

Get-ADUser -Filter * -Properties UserPrincipalName |
  Select-Object SamAccountName,Name,DistinguishedName,UserPrincipalName |
  Export-Csv "$EvidencePath\upn-inventory-after-rollback.csv" -NoTypeInformation

Stop-Transcript
```

```powershell
# Optional rollback for alternate UPN suffix.
# Use only if no users should keep the suffix and cloud prep is being abandoned.

$TargetUpnSuffix = "contoso.com"
$EvidencePath = "C:\HybridIdentity-Prep"

Start-Transcript -Path "$EvidencePath\rollback-remove-upn-suffix-transcript.txt"

Import-Module ActiveDirectory

$ForestName = (Get-ADForest).Name

Set-ADForest `
  -Identity $ForestName `
  -UPNSuffixes @{Remove = $TargetUpnSuffix}

(Get-ADForest).UPNSuffixes |
  Out-File "$EvidencePath\upn-suffixes-after-remove.txt"

Stop-Transcript
```

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Target suffix does not appear in ADUC | UPN suffix not added to forest | `(Get-ADForest).UPNSuffixes` | Add suffix with `Set-ADForest -UPNSuffixes @{Add="contoso.com"}` |
| `Set-ADForest` fails | Insufficient rights or AD module missing | `whoami /groups`; `Get-Module -ListAvailable ActiveDirectory` | Use proper admin account and install RSAT AD tools |
| Users still show `.local` UPNs | Bulk update skipped OU or excluded users | `Get-ADUser -Filter "UserPrincipalName -like '*corp.local'"` | Review scope and update approved users |
| Duplicate UPN detected | Two AD users share same sign-in name | Group by `UserPrincipalName` | Correct one or both UPNs before sync |
| Blank UPN detected | User object has no UPN value | Search blank UPN query | Assign valid routable UPN |
| User UPN differs from email | Historical AD mismatch or mail alias design | Compare `UserPrincipalName`, `mail`, and `proxyAddresses` | Align UPN to primary SMTP unless exception is documented |
| Service account breaks after UPN change | Account was changed in bulk without application review | Review `Description`, SPNs, app configs | Roll back that account and document exclusion |
| Microsoft 365 domain verification fails | DNS TXT/MX not published, wrong host, or propagation delay | `Resolve-DnsName <domain> -Type TXT` | Fix DNS record at authoritative DNS host and retry |
| Mail flow changes unexpectedly | MX was changed too early | `Resolve-DnsName <domain> -Type MX` | Restore previous MX until mail migration task |
| User sign-in confusion after change | Users were not told their new UPN | Compare old and new UPN CSVs | Communicate new sign-in format and update documentation |
| AD replication delay | UPN suffix or user changes not replicated | `repadmin /replsummary` | Fix AD replication before sync work |
| Public domain is not owned | Domain is not registered or controlled | Registrar lookup and DNS NS query | Register domain or use a domain you control |

# 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud_Related_Labs

| Lab                                                                                   | Relationship                                                                                     |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md                           | Validates the public DNS records required to prove ownership and prepare Microsoft 365 services  |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md               | Cleans duplicate and invalid AD attributes before sync                                           |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md                            | Uses this UPN and domain preparation as the identity sync foundation                             |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md | Troubleshoots sync failures caused by bad UPNs, duplicates, or invalid attributes                |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md               | Builds user sign-in protection after identities are cloud-ready                                  |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md       | Protects cloud sign-ins after domain and identity baseline is in place                           |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments.md          | Maps synced users and groups into cloud access, licensing, app assignment, and admin role models |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md             | Troubleshoots post-change hybrid sign-in problems                                                |