# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Index

02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Source_Basis  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Mental_Model  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Planning_Table  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Configuration_Checklist  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Public_DNS_Precheck_Skeleton  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_M365_Domain_Add_And_Verification_Skeleton  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Azure_DNS_Zone_Validation_Skeleton  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_M365_Service_Record_Validation_Skeleton  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Email_Authentication_Validation_Skeleton  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Verification_Commands  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Rollback  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Failure_Checks  
02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Related_Labs  

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft 365 Docs | Add a custom domain to Microsoft 365 | Adding a custom domain, Domain Connect, manual TXT or MX verification, and Microsoft 365 DNS record setup |
| Microsoft 365 Docs | Create DNS records at any DNS hosting provider | Manual DNS record planning for Exchange Online, Teams, Intune, and Microsoft 365 service records |
| Microsoft 365 Docs | Domains FAQ | Domain ownership, DNS host behavior, registrar behavior, and expected propagation delays |
| Microsoft Entra Docs | Custom domain names | Microsoft Entra custom domain verification and tenant sign-in namespace validation |
| Microsoft Graph PowerShell | Get-MgDomain, New-MgDomain, Confirm-MgDomain, Get-MgDomainVerificationDnsRecord, Get-MgDomainServiceConfigurationRecord | Scriptable domain creation, verification record retrieval, domain confirmation, and service record review |
| Azure DNS Docs | DNS zones and record sets | Hosting public DNS zones in Azure DNS and validating record sets |
| Exchange Online Docs | MX, Autodiscover, SPF, DKIM, DMARC | Mail flow and email authentication record validation |
| Windows PowerShell | Resolve-DnsName | Public DNS validation from the operator workstation |
| Operational practice | Do not change MX until mailboxes and cutover plan are ready | Prevents accidental mail outage during domain verification |

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Registrar | Company where the domain is registered, such as GoDaddy, Namecheap, or Cloudflare Registrar |
| DNS host | Platform that hosts the active authoritative DNS zone for the domain |
| Azure DNS zone | Azure-hosted authoritative DNS zone for a public domain |
| Microsoft 365 custom domain | Public DNS domain added to the Microsoft 365 tenant |
| Microsoft Entra custom domain | Public DNS domain verified in Microsoft Entra ID for user sign-in suffixes |
| Domain ownership verification | Proof that you control the domain, usually through TXT record |
| TXT verification record | Record such as `MS=ms########` used to prove domain ownership |
| MX verification fallback | Alternate domain ownership proof using MX record, not preferred if email already exists |
| MX record | Mail exchanger record controlling where inbound mail is delivered |
| Autodiscover CNAME | Exchange Online client discovery record, usually `autodiscover` to `autodiscover.outlook.com` |
| SPF TXT | Sender Policy Framework TXT record that authorizes mail senders for the domain |
| DKIM | DomainKeys Identified Mail signing records used by Exchange Online or third-party mail systems |
| DMARC | Policy record that tells receivers what to do with SPF/DKIM alignment failures |
| SRV record | Service locator record used by some Microsoft services |
| TTL | Time to live, controls how long resolvers cache a DNS answer |
| Domain Connect | Registrar integration that can automatically add Microsoft 365 DNS records |
| Manual DNS setup | Operator manually creates each DNS record at the DNS host |
| First rule | Find the authoritative DNS host before adding records anywhere |
| Blunt rule | Do not touch MX until mailboxes, migration, and cutover timing are ready |

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Custom domain | `contoso.com` | `<custom-domain>` |
| Microsoft 365 tenant | `contoso.onmicrosoft.com` | `<tenant-domain>` |
| Microsoft Entra tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| DNS host | Azure DNS / Cloudflare / GoDaddy | `<dns-host>` |
| Registrar | GoDaddy / Namecheap / Microsoft | `<registrar>` |
| Azure subscription | `Prod-Connectivity` | `<subscription-name>` |
| Azure DNS resource group | `rg-dns-prod` | `<dns-resource-group>` |
| Azure DNS zone name | `contoso.com` | `<azure-dns-zone>` |
| Existing authoritative NS records | `ns1-xx.azure-dns.com` | `<authoritative-ns-records>` |
| Domain verification method | TXT | `<TXT / MX / Website file>` |
| Verification TXT name | `@` or blank root | `<verification-record-name>` |
| Verification TXT value | `MS=ms12345678` | `<verification-record-value>` |
| Existing MX target | `mail.contoso.com` | `<existing-mx-target>` |
| Future Exchange Online MX target | `contoso-com.mail.protection.outlook.com` | `<m365-mx-target>` |
| Autodiscover target | `autodiscover.outlook.com` | `<autodiscover-target>` |
| SPF value | `v=spf1 include:spf.protection.outlook.com -all` | `<spf-value>` |
| DKIM selector 1 | `selector1._domainkey` | `<dkim-selector1>` |
| DKIM selector 2 | `selector2._domainkey` | `<dkim-selector2>` |
| DMARC policy | `v=DMARC1; p=none; rua=mailto:dmarc@contoso.com` | `<dmarc-policy>` |
| Evidence folder | `C:\M365-Domain-DNS` | `<evidence-path>` |
| DNS change window | After hours | `<change-window>` |
| Rollback stance | Restore previous DNS records from export | `<rollback-plan>` |

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\M365-Domain-DNS` | Evidence folder exists |
| 2 | Confirm internet DNS resolution works | Admin Workstation | `Resolve-DnsName microsoft.com` | Public DNS resolution succeeds |
| 3 | Confirm the public domain exists | Admin Workstation | `Resolve-DnsName contoso.com -Type SOA` | SOA record returns |
| 4 | Identify authoritative name servers | Admin Workstation | `Resolve-DnsName contoso.com -Type NS` | Authoritative NS records are visible |
| 5 | Confirm where DNS is actually hosted | Admin Workstation | `Resolve-DnsName contoso.com -Type NS | Select-Object NameHost` | DNS host is identified before changes |
| 6 | Export existing TXT records | Admin Workstation | `Resolve-DnsName contoso.com -Type TXT -ErrorAction SilentlyContinue` | Existing TXT records are documented |
| 7 | Export existing MX records | Admin Workstation | `Resolve-DnsName contoso.com -Type MX -ErrorAction SilentlyContinue` | Current mail routing is documented |
| 8 | Export existing CNAME records for Microsoft service names | Admin Workstation | `Resolve-DnsName autodiscover.contoso.com -Type CNAME -ErrorAction SilentlyContinue` | Existing Autodiscover state is known |
| 9 | Confirm Microsoft Graph PowerShell module | Admin Workstation | `Get-Module -ListAvailable Microsoft.Graph` | Microsoft Graph module is available |
| 10 | Connect to Microsoft Graph | Admin Workstation | `Connect-MgGraph -Scopes "Domain.ReadWrite.All","Directory.ReadWrite.All"` | Graph session connects |
| 11 | Confirm current tenant domains | Admin Workstation | `Get-MgDomain | Select-Object Id,IsVerified,IsDefault,AuthenticationType` | Existing domains are visible |
| 12 | Add custom domain to tenant if missing | Admin Workstation | `New-MgDomain -Id "contoso.com"` | Domain object is created in tenant |
| 13 | Retrieve verification DNS record | Admin Workstation | `Get-MgDomainVerificationDnsRecord -DomainId "contoso.com"` | Verification TXT or MX value is returned |
| 14 | Add TXT verification record at DNS host | DNS Host / Azure DNS | `Create TXT record at @ with MS=ms########` | Verification TXT is published |
| 15 | Confirm TXT verification record resolves publicly | Admin Workstation | `Resolve-DnsName contoso.com -Type TXT` | Expected `MS=ms########` value returns |
| 16 | Verify domain in Microsoft 365 / Entra | Admin Workstation | `Confirm-MgDomain -DomainId "contoso.com"` | Domain becomes verified |
| 17 | Confirm verified state | Admin Workstation | `Get-MgDomain -DomainId "contoso.com" | Select-Object Id,IsVerified` | `IsVerified` is `True` |
| 18 | Retrieve Microsoft 365 service DNS records | Admin Workstation | `Get-MgDomainServiceConfigurationRecord -DomainId "contoso.com"` | Required M365 service records are visible |
| 19 | Add Exchange Online records only when mail cutover is approved | DNS Host | `MX, Autodiscover CNAME, SPF TXT` | Exchange Online records are published only when appropriate |
| 20 | Add Teams / Skype service records if required | DNS Host | `SRV and CNAME records from M365 domain setup` | Required service discovery records are published |
| 21 | Add Intune / MDM records if required | DNS Host | `CNAME records from M365 domain setup` | Device enrollment discovery records are published |
| 22 | Validate SPF TXT record | Admin Workstation | `Resolve-DnsName contoso.com -Type TXT | Where-Object {$_.Strings -match "v=spf1"}` | SPF exists and has expected include values |
| 23 | Validate Autodiscover | Admin Workstation | `Resolve-DnsName autodiscover.contoso.com -Type CNAME` | Autodiscover points to Microsoft 365 when cutover is approved |
| 24 | Validate MX target | Admin Workstation | `Resolve-DnsName contoso.com -Type MX` | MX points to expected current or target mail system |
| 25 | Validate DKIM records if enabled | Admin Workstation | `Resolve-DnsName selector1._domainkey.contoso.com -Type CNAME` | DKIM selector CNAME resolves |
| 26 | Validate DMARC record | Admin Workstation | `Resolve-DnsName _dmarc.contoso.com -Type TXT` | DMARC TXT record resolves |
| 27 | Recheck Microsoft 365 domain health | Microsoft 365 Admin Center | `Settings > Domains > contoso.com > Check health` | Domain records show healthy or known exceptions |
| 28 | Export final DNS evidence | Admin Workstation | `Run service validation skeleton` | Final DNS validation files are saved |
| 29 | Document unresolved exceptions | Operator | `Record missing records, reason, owner, and next step` | Exceptions are known and not accidental |
| 30 | Mark domain validation complete | Operator | `Record verification date, DNS host, records changed, and rollback notes` | Workbook task is complete |

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Public_DNS_Precheck_Skeleton

```powershell
# Run from an admin workstation with internet DNS resolution.
# Purpose: identify authoritative DNS hosting and capture the domain before-state.

$Domain = "contoso.com"
$EvidencePath = "C:\M365-Domain-DNS"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-public-dns-precheck-transcript.txt"

Resolve-DnsName $Domain -Type SOA -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-soa-before.txt"

Resolve-DnsName $Domain -Type NS -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-ns-before.txt"

Resolve-DnsName $Domain -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-txt-before.txt"

Resolve-DnsName $Domain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-mx-before.txt"

$NamesToCheck = @(
  "autodiscover.$Domain",
  "enterpriseenrollment.$Domain",
  "enterpriseregistration.$Domain",
  "sip.$Domain",
  "lyncdiscover.$Domain",
  "_sip._tls.$Domain",
  "_sipfederationtls._tcp.$Domain",
  "_dmarc.$Domain",
  "selector1._domainkey.$Domain",
  "selector2._domainkey.$Domain"
)

foreach ($Name in $NamesToCheck) {
  Write-Host "Checking $Name"

  Resolve-DnsName $Name -ErrorAction SilentlyContinue |
    Out-File "$EvidencePath\$($Name.Replace('*','wildcard')).dns-before.txt"
}

Stop-Transcript
```

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_M365_Domain_Add_And_Verification_Skeleton

```powershell
# Run from an admin workstation with Microsoft Graph PowerShell.
# Purpose: add the custom domain to Microsoft 365 / Microsoft Entra and retrieve verification records.

$Domain = "contoso.com"
$EvidencePath = "C:\M365-Domain-DNS"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-m365-domain-add-verification-transcript.txt"

# Install only if needed:
# Install-Module Microsoft.Graph -Scope CurrentUser

Import-Module Microsoft.Graph.Identity.DirectoryManagement

Connect-MgGraph -Scopes "Domain.ReadWrite.All","Directory.ReadWrite.All"

Get-MgDomain |
  Select-Object Id,IsVerified,IsDefault,AuthenticationType,AvailabilityStatus |
  Export-Csv "$EvidencePath\m365-domains-before.csv" -NoTypeInformation

$ExistingDomain = Get-MgDomain -DomainId $Domain -ErrorAction SilentlyContinue

if (-not $ExistingDomain) {
  New-MgDomain -Id $Domain
}

Get-MgDomainVerificationDnsRecord -DomainId $Domain |
  Select-Object Id,RecordType,Label,SupportedService,Ttl |
  Export-Csv "$EvidencePath\$Domain-verification-records.csv" -NoTypeInformation

Get-MgDomainVerificationDnsRecord -DomainId $Domain |
  Format-List * |
  Out-File "$EvidencePath\$Domain-verification-records-full.txt"

# After the TXT verification record is added at the authoritative DNS host,
# uncomment the next command to confirm the domain.
# Confirm-MgDomain -DomainId $Domain

Get-MgDomain -DomainId $Domain |
  Format-List * |
  Out-File "$EvidencePath\$Domain-domain-object-after.txt"

Disconnect-MgGraph

Stop-Transcript
```

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Azure_DNS_Zone_Validation_Skeleton

```powershell
# Run if the domain is hosted in Azure DNS.
# Purpose: inventory Azure DNS zone and create verification TXT record when Azure DNS is authoritative.

$SubscriptionName = "Prod-Connectivity"
$ResourceGroupName = "rg-dns-prod"
$ZoneName = "contoso.com"
$VerificationTxtValue = "MS=ms12345678"
$EvidencePath = "C:\M365-Domain-DNS"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-azure-dns-zone-validation-transcript.txt"

Connect-AzAccount
Set-AzContext -Subscription $SubscriptionName

Get-AzDnsZone -ResourceGroupName $ResourceGroupName -Name $ZoneName |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-azure-dns-zone.txt"

Get-AzDnsRecordSet -ResourceGroupName $ResourceGroupName -ZoneName $ZoneName |
  Select-Object Name,RecordType,Ttl,Records |
  Export-Csv "$EvidencePath\$ZoneName-azure-dns-recordsets-before.csv" -NoTypeInformation

$TxtRecordSet = Get-AzDnsRecordSet `
  -ResourceGroupName $ResourceGroupName `
  -ZoneName $ZoneName `
  -Name "@" `
  -RecordType TXT `
  -ErrorAction SilentlyContinue

if (-not $TxtRecordSet) {
  New-AzDnsRecordSet `
    -ResourceGroupName $ResourceGroupName `
    -ZoneName $ZoneName `
    -Name "@" `
    -RecordType TXT `
    -Ttl 3600 `
    -DnsRecords (New-AzDnsRecordConfig -Value $VerificationTxtValue)
}
else {
  Add-AzDnsRecordConfig `
    -RecordSet $TxtRecordSet `
    -Value $VerificationTxtValue

  Set-AzDnsRecordSet -RecordSet $TxtRecordSet
}

Get-AzDnsRecordSet `
  -ResourceGroupName $ResourceGroupName `
  -ZoneName $ZoneName `
  -Name "@" `
  -RecordType TXT |
  Format-List * |
  Out-File "$EvidencePath\$ZoneName-verification-txt-in-azure.txt"

Resolve-DnsName $ZoneName -Type TXT -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\$ZoneName-verification-txt-public-query.txt"

Stop-Transcript
```

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_M365_Service_Record_Validation_Skeleton

```powershell
# Run after domain verification.
# Purpose: retrieve and validate Microsoft 365 service configuration DNS records.

$Domain = "contoso.com"
$EvidencePath = "C:\M365-Domain-DNS"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-m365-service-record-validation-transcript.txt"

Import-Module Microsoft.Graph.Identity.DirectoryManagement

Connect-MgGraph -Scopes "Domain.Read.All","Directory.Read.All"

Get-MgDomainServiceConfigurationRecord -DomainId $Domain |
  Format-List * |
  Out-File "$EvidencePath\$Domain-m365-service-configuration-records-full.txt"

Get-MgDomainServiceConfigurationRecord -DomainId $Domain |
  Select-Object Id,RecordType,Label,SupportedService,Ttl |
  Export-Csv "$EvidencePath\$Domain-m365-service-configuration-records.csv" -NoTypeInformation

Disconnect-MgGraph

Resolve-DnsName $Domain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-mx-current.txt"

Resolve-DnsName "autodiscover.$Domain" -Type CNAME -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-autodiscover-cname-current.txt"

Resolve-DnsName $Domain -Type TXT -ErrorAction SilentlyContinue |
  Where-Object { $_.Strings -match "v=spf1" } |
  Tee-Object "$EvidencePath\$Domain-spf-current.txt"

Resolve-DnsName "sip.$Domain" -Type CNAME -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-sip-cname-current.txt"

Resolve-DnsName "lyncdiscover.$Domain" -Type CNAME -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-lyncdiscover-cname-current.txt"

Resolve-DnsName "_sip._tls.$Domain" -Type SRV -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-sip-tls-srv-current.txt"

Resolve-DnsName "_sipfederationtls._tcp.$Domain" -Type SRV -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-sipfederationtls-srv-current.txt"

Resolve-DnsName "enterpriseenrollment.$Domain" -Type CNAME -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-enterpriseenrollment-cname-current.txt"

Resolve-DnsName "enterpriseregistration.$Domain" -Type CNAME -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-enterpriseregistration-cname-current.txt"

Stop-Transcript
```

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Email_Authentication_Validation_Skeleton

```powershell
# Run after Exchange Online records are planned or deployed.
# Purpose: validate SPF, DKIM, and DMARC DNS records.

$Domain = "contoso.com"
$EvidencePath = "C:\M365-Domain-DNS"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-email-authentication-validation-transcript.txt"

Resolve-DnsName $Domain -Type TXT -ErrorAction SilentlyContinue |
  Where-Object { $_.Strings -match "v=spf1" } |
  Tee-Object "$EvidencePath\$Domain-spf.txt"

Resolve-DnsName "_dmarc.$Domain" -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-dmarc.txt"

Resolve-DnsName "selector1._domainkey.$Domain" -Type CNAME -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-dkim-selector1.txt"

Resolve-DnsName "selector2._domainkey.$Domain" -Type CNAME -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\$Domain-dkim-selector2.txt"

$SpfRecords = Resolve-DnsName $Domain -Type TXT -ErrorAction SilentlyContinue |
  Where-Object { $_.Strings -match "v=spf1" }

$SpfRecords |
  Out-File "$EvidencePath\$Domain-spf-record-count.txt"

if (($SpfRecords | Measure-Object).Count -gt 1) {
  Write-Warning "Multiple SPF records found. Consolidate SPF into one TXT record."
}

Stop-Transcript
```

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Resolve-DnsName contoso.com -Type SOA` | Confirms public DNS zone exists | SOA record returns |
| `Resolve-DnsName contoso.com -Type NS` | Identifies authoritative DNS host | Expected authoritative NS records return |
| `Resolve-DnsName contoso.com -Type TXT` | Checks verification, SPF, and other TXT records | Expected TXT records return |
| `Resolve-DnsName contoso.com -Type MX` | Checks inbound mail routing | MX target is expected for current migration stage |
| `Resolve-DnsName autodiscover.contoso.com -Type CNAME` | Checks Exchange Autodiscover | Points to expected current or Microsoft 365 target |
| `Resolve-DnsName _dmarc.contoso.com -Type TXT` | Checks DMARC | DMARC TXT policy returns |
| `Resolve-DnsName selector1._domainkey.contoso.com -Type CNAME` | Checks DKIM selector 1 | DKIM CNAME returns after enabled |
| `Resolve-DnsName selector2._domainkey.contoso.com -Type CNAME` | Checks DKIM selector 2 | DKIM CNAME returns after enabled |
| `Resolve-DnsName enterpriseenrollment.contoso.com -Type CNAME` | Checks Intune enrollment discovery | CNAME exists if Intune enrollment requires it |
| `Resolve-DnsName enterpriseregistration.contoso.com -Type CNAME` | Checks device registration discovery | CNAME exists if required |
| `Get-MgDomain` | Lists tenant domains | Custom domain appears |
| `Get-MgDomain -DomainId "contoso.com"` | Checks exact custom domain state | `IsVerified` is `True` after verification |
| `Get-MgDomainVerificationDnsRecord -DomainId "contoso.com"` | Retrieves required verification records | TXT or MX verification values return |
| `Get-MgDomainServiceConfigurationRecord -DomainId "contoso.com"` | Retrieves Microsoft 365 service DNS records | Required record list returns |
| `Get-AzDnsZone -Name "contoso.com" -ResourceGroupName "rg-dns-prod"` | Confirms Azure DNS zone exists | Zone object returns |
| `Get-AzDnsRecordSet -ZoneName "contoso.com" -ResourceGroupName "rg-dns-prod"` | Lists Azure DNS records | Record sets return |

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Rollback

```powershell
# Rollback approach for Azure DNS-hosted domains.
# Restore record values from the before-state CSV manually or by scripted change control.
# Do not run blindly in production.

$SubscriptionName = "Prod-Connectivity"
$ResourceGroupName = "rg-dns-prod"
$ZoneName = "contoso.com"
$EvidencePath = "C:\M365-Domain-DNS"

Connect-AzAccount
Set-AzContext -Subscription $SubscriptionName

Get-AzDnsRecordSet -ResourceGroupName $ResourceGroupName -ZoneName $ZoneName |
  Select-Object Name,RecordType,Ttl,Records |
  Export-Csv "$EvidencePath\$ZoneName-azure-dns-recordsets-before-rollback.csv" -NoTypeInformation

$VerificationTxtValue = "MS=ms12345678"

$TxtRecordSet = Get-AzDnsRecordSet `
  -ResourceGroupName $ResourceGroupName `
  -ZoneName $ZoneName `
  -Name "@" `
  -RecordType TXT `
  -ErrorAction SilentlyContinue

if ($TxtRecordSet) {
  $TxtRecordSet.Records = @(
    $TxtRecordSet.Records | Where-Object {
      $_.Value -notcontains $VerificationTxtValue
    }
  )

  Set-AzDnsRecordSet -RecordSet $TxtRecordSet
}

Resolve-DnsName $ZoneName -Type TXT -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\$ZoneName-txt-after-rollback.txt"
```

```powershell
# Microsoft 365 domain object rollback.
# Use only if the custom domain was added by mistake and is not used by users, groups, aliases, apps, or services.

Import-Module Microsoft.Graph.Identity.DirectoryManagement

Connect-MgGraph -Scopes "Domain.ReadWrite.All","Directory.ReadWrite.All"

$Domain = "contoso.com"

Get-MgDomain -DomainId $Domain | Format-List *

# Removal command may fail if the domain is assigned to users, groups, email addresses, apps, or service records.
# Remove-MgDomain -DomainId $Domain

Disconnect-MgGraph
```

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Failure_Checks

| Symptom                                                               | Likely Cause                                                             | Check Command                                                  | Fix                                                    |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------- | ------------------------------------------------------ |
| Microsoft 365 cannot verify domain                                    | TXT record missing, wrong value, wrong DNS host, or propagation delay    | `Resolve-DnsName contoso.com -Type TXT`                        | Add exact TXT record at authoritative DNS host         |
| Verification TXT exists in portal DNS host but not public DNS         | DNS was edited at registrar but authoritative DNS is somewhere else      | `Resolve-DnsName contoso.com -Type NS`                         | Add record at authoritative DNS host                   |
| TXT record split incorrectly                                          | DNS host wrapped or changed TXT value                                    | `Resolve-DnsName contoso.com -Type TXT`                        | Recreate exact value from Microsoft 365                |
| MX verification breaks mail                                           | MX verification record used with low priority or mail cutover done early | `Resolve-DnsName contoso.com -Type MX`                         | Restore original MX or set verification MX safely      |
| Inbound mail goes to wrong place                                      | MX record changed before mailboxes and cutover were ready                | `Resolve-DnsName contoso.com -Type MX`                         | Restore previous MX until migration task               |
| Autodiscover fails                                                    | Missing or wrong CNAME                                                   | `Resolve-DnsName autodiscover.contoso.com -Type CNAME`         | Point Autodiscover to correct current platform         |
| SPF fails                                                             | Multiple SPF TXT records or missing include                              | `Resolve-DnsName contoso.com -Type TXT`                        | Consolidate into one SPF record                        |
| DKIM fails                                                            | DKIM not enabled or selector CNAMEs missing                              | `Resolve-DnsName selector1._domainkey.contoso.com -Type CNAME` | Add selector CNAMEs and enable DKIM in Exchange Online |
| DMARC missing                                                         | No `_dmarc` TXT record                                                   | `Resolve-DnsName _dmarc.contoso.com -Type TXT`                 | Add DMARC policy record                                |
| Teams federation records fail                                         | SRV records missing or wrong                                             | `Resolve-DnsName _sipfederationtls._tcp.contoso.com -Type SRV` | Add service records from Microsoft 365 domain setup    |
| Intune enrollment discovery fails                                     | `enterpriseenrollment` or `enterpriseregistration` CNAME missing         | `Resolve-DnsName enterpriseenrollment.contoso.com -Type CNAME` | Add required Intune CNAMEs                             |
| Microsoft 365 shows unhealthy records that were intentionally skipped | Domain health expects services not in use                                | `Microsoft 365 admin center > Settings > Domains`              | Document exception and do not add unnecessary records  |
| Azure DNS record does not resolve publicly                            | Domain registrar not delegated to Azure DNS name servers                 | `Resolve-DnsName contoso.com -Type NS`                         | Update registrar NS delegation to Azure DNS NS records |
| Domain cannot be removed from tenant                                  | Domain is assigned to users, aliases, groups, apps, or service configs   | `Get-MgDomain -DomainId contoso.com`                           | Move objects back to another verified domain first     |
| Public DNS checks vary by resolver                                    | DNS cache or TTL delay                                                   | `Resolve-DnsName contoso.com -Server 8.8.8.8`                  | Wait for TTL or query authoritative name server        |

# 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365_Related_Labs

| Lab                                                                                   | Relationship                                                                       |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md           | Uses the verified custom domain as the target routable UPN suffix                  |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md               | Requires clean UPN and proxyAddress values before sync                             |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md                            | Depends on verified domains before synchronized users can use the right UPN suffix |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md | Troubleshoots domain and UPN sync issues after verification                        |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md               | Uses verified domains for user sign-in and recovery flows                          |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md       | Protects sign-ins after custom domain identity is active                           |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md             | Validates sign-in failures caused by domain, DNS, MFA, CA, or licensing issues     |