# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Index

## Purpose

Configure and validate custom domains for a Microsoft cloud tenant.

This workbook covers:

- Microsoft Entra tenant namespace review
- Microsoft 365 tenant namespace review
- Initial onmicrosoft.com domain review
- Custom domain planning
- Public DNS ownership verification
- TXT verification record creation
- MX verification record alternative
- Microsoft 365 domain verification
- Microsoft Entra domain verification
- Azure DNS hosted zone option
- External DNS provider option
- Domain purpose planning
- UPN namespace readiness
- Exchange Online namespace readiness
- SPF, DKIM, DMARC, MX, and Autodiscover planning
- Domain validation with portal, PowerShell, Azure CLI, and DNS tools
- Rollback and troubleshooting

## Outcome

By the end of this workbook, the admin should be able to:

- Identify the tenant initial domain
- Identify existing verified custom domains
- Add a new custom domain to the tenant
- Retrieve the required DNS verification records
- Publish DNS verification records
- Confirm DNS propagation
- Verify the domain in Microsoft 365 / Entra
- Decide whether the domain will be used for sign-in, mail, or both
- Document DNS provider ownership
- Document namespace risks before changing user UPNs or mail routing
- Validate that the domain is ready for later identity, Exchange, SharePoint, Teams, and licensing workbooks

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Source_Basis

| Source | Relevant Area | What It Supports |
|---|---|---|
| Microsoft 365 documentation fork | Microsoft 365 domains, admin center domain verification, domain services, tenant namespace | M365 custom domain workflow |
| Microsoft Entra documentation | Tenant ID, verified domains, users, UPNs, app sign-in, identity namespace | Entra custom domain and sign-in planning |
| Microsoft Graph documentation | Domains API, organization object, verified domains, domain DNS records | PowerShell and API validation |
| Azure DNS documentation | DNS zones, TXT records, MX records, CNAME records, NS delegation | Azure-hosted DNS option |
| Exchange Online documentation | Accepted domains, DKIM, mail routing, SPF, Autodiscover, recipient namespace | Mail readiness planning |
| Existing Workbook 01 | Tenant, directory, subscription, and domain comparison | Provides tenant and namespace baseline |
| Existing Workbook 02 | Admin portal, PowerShell, Azure CLI, and Graph tooling baseline | Provides tooling required for this workbook |
| Existing Workbook structure | Source basis, mental model, planning, checklist, skeletons, verification, rollback, failure checks | Obsidian-ready workbook format |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Mental_Model

| Concept | Plain Meaning | Admin Boundary |
|---|---|---|
| Tenant namespace | The DNS names the tenant can use for sign-in, mail, and services | Identity and service naming |
| Initial domain | The original tenantname.onmicrosoft.com domain created with the tenant | Permanent tenant namespace |
| Custom domain | Public DNS name you own and add to the tenant | Identity, mail, and workload namespace |
| Verified domain | Domain proven through public DNS ownership | Tenant trust boundary |
| DNS verification record | TXT or MX record Microsoft asks you to publish | Ownership proof |
| DNS host | Provider where public DNS records are managed | External dependency |
| Registrar | Provider where the domain is registered | Ownership and nameserver control |
| Authoritative nameserver | DNS server that actually answers for the domain | DNS truth source |
| Azure DNS zone | Azure-hosted public DNS zone | Azure DNS control plane |
| External DNS provider | Non-Azure DNS host such as registrar, Cloudflare, Route 53, GoDaddy, etc. | External DNS control plane |
| UPN | User Principal Name, usually user@domain.com | Sign-in namespace |
| Primary SMTP address | Main email address for mailbox | Mail namespace |
| Proxy address | Additional email alias on recipient | Mail namespace |
| Accepted domain | Exchange Online domain accepted for mail flow | Exchange mail boundary |
| SPF | DNS TXT record listing allowed mail senders | Sender authentication |
| DKIM | Cryptographic mail signing using DNS CNAME records | Sender authentication |
| DMARC | Domain policy for SPF/DKIM alignment and reporting | Mail authentication policy |
| MX | Mail exchanger DNS record | Inbound mail routing |
| Autodiscover | DNS record used by Outlook clients | Client configuration |
| Tenant default domain | Default domain used for new cloud users | Identity creation behavior |
| SharePoint tenant name | Tenant-specific SharePoint namespace | SharePoint/OneDrive service namespace |

## Core Rule

Adding a domain to Microsoft 365 or Microsoft Entra does not move DNS hosting.

Microsoft asks you to prove ownership by publishing a DNS record where the domain is authoritative.

Only after verification should you use the domain for UPNs, mail addresses, Exchange routing, DKIM, DMARC, or production user sign-in.

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Environment_Assumptions

| Item | Lab Example | Production Example |
|---|---|---|
| Tenant initial domain | contosolab.onmicrosoft.com | companyname.onmicrosoft.com |
| Custom domain | lab.contoso.com | contoso.com |
| DNS provider | Azure DNS | Registrar / Cloudflare / Route 53 / Azure DNS |
| Registrar | Example Registrar | Actual domain registrar |
| Target tenant ID | 00000000-0000-0000-0000-000000000000 | Production tenant ID |
| Admin account | cloudadmin@contosolab.onmicrosoft.com | named cloud admin account |
| Verification method | TXT record | TXT preferred, MX alternative |
| Mail use | No for initial lab | Yes if Exchange Online migration is planned |
| Sign-in use | Yes for test users | Yes after staged UPN planning |
| Azure subscription | Azure Lab Subscription | Shared services or DNS subscription |
| Azure DNS resource group | rg-dns-prod | rg-connectivity-dns |
| Public DNS zone | contoso.com | production public zone |
| TTL | 3600 seconds | Provider standard or lower during cutover |
| Evidence output | .\Custom_Domain_Baseline | secure admin evidence path |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Planning_Table

| Item | Value To Record | Why It Matters |
|---|---|---|
| Tenant display name |  | Confirms correct tenant |
| Tenant ID |  | Prevents verifying domain in wrong tenant |
| Initial onmicrosoft.com domain |  | Permanent fallback namespace |
| Existing custom domains |  | Prevents duplicate or conflicting domain work |
| New custom domain |  | Domain being added |
| Domain registrar |  | Needed if nameservers must be changed |
| DNS host |  | Where TXT/MX/CNAME records are created |
| Authoritative nameservers |  | Confirms where DNS queries are answered |
| DNS zone location | Azure DNS / external | Determines tooling |
| Verification record type | TXT / MX | TXT preferred |
| Verification TXT name | @ or domain root | Provider-specific record name |
| Verification TXT value | MS=ms######## | Required ownership proof |
| TTL | 300 / 3600 / provider default | Impacts propagation and troubleshooting |
| Domain purpose | Sign-in / mail / both / future use | Determines required records |
| UPN change needed | Yes / No | Avoids breaking sign-in |
| Mail cutover needed | Yes / No | Avoids accidental MX changes |
| Exchange accepted domain needed | Yes / No | Required for mail flow |
| SPF needed | Yes / No | Required for outbound mail authentication |
| DKIM needed | Yes / No | Required for modern mail authentication |
| DMARC needed | Yes / No | Required for reporting and protection |
| Autodiscover needed | Yes / No | Required for Outlook client configuration |
| Break glass impact | None / Review | Avoids locking out emergency accounts |
| Conditional Access impact | None / Review | Avoids sign-in disruption |
| Application redirect impact | None / Review | App sign-in may depend on UPN/domain |
| Hybrid identity impact | None / Review | AD UPN suffix and Entra sync may be affected |
| Verification date |  | Evidence |
| Validation command output path |  | Evidence |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Domain_Purpose_Table

| Purpose | Required Tenant State | Required DNS Records | Notes |
|---|---|---|---|
| Identity sign-in only | Domain verified in Entra/M365 | TXT verification record | Use for UPNs after verification |
| Exchange Online mail | Domain verified and accepted in Exchange Online | MX, SPF, DKIM CNAMEs, DMARC, Autodiscover | Do not change MX until mail cutover is planned |
| Teams identity display | Domain verified and users assigned UPN/mail | Usually identity/mail records | Teams follows identity and Exchange namespace |
| SharePoint sharing | Domain verified and users exist | Usually none beyond identity/mail | Sharing policies are separate |
| Intune enrollment | Domain verified and users have UPNs | CNAME records may be needed for enrollment discovery depending on design | Validate Intune-specific records later |
| App registrations | Domain verified if publisher verification or redirect naming depends on domain | TXT verification may be enough | App ownership and publisher verification are separate |
| Hybrid identity | Domain verified and UPN suffix aligned with AD | TXT verification plus AD UPN suffix planning | Must coordinate with Entra Connect / cloud sync |
| Branding | Tenant/domain exists | May require separate branding config | Branding does not equal domain verification |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_DNS_Record_Planning_Table

| Record | Type | Name / Host | Example Value | Purpose | Change Timing |
|---|---|---|---|---|---|
| Tenant verification | TXT | @ | MS=ms12345678 | Prove ownership to Microsoft | Safe before production use |
| Alternate verification | MX | @ | ms12345678.msv1.invalid | Prove ownership if TXT unavailable | Only if TXT not possible |
| Exchange inbound mail | MX | @ | contoso-com.mail.protection.outlook.com | Route inbound mail to Exchange Online | Only during planned mail cutover |
| SPF | TXT | @ | v=spf1 include:spf.protection.outlook.com -all | Authorize Microsoft 365 outbound mail | Before or during mail use |
| Autodiscover | CNAME | autodiscover | autodiscover.outlook.com | Outlook autodiscover | During Exchange readiness |
| DKIM selector 1 | CNAME | selector1._domainkey | selector1-contoso-com._domainkey.contoso.onmicrosoft.com | DKIM signing | After Exchange domain exists |
| DKIM selector 2 | CNAME | selector2._domainkey | selector2-contoso-com._domainkey.contoso.onmicrosoft.com | DKIM signing | After Exchange domain exists |
| DMARC monitor | TXT | _dmarc | v=DMARC1; p=none; rua=mailto:dmarc@contoso.com | Reporting and visibility | Before enforcement |
| DMARC quarantine | TXT | _dmarc | v=DMARC1; p=quarantine; rua=mailto:dmarc@contoso.com | Partial enforcement | After validation |
| DMARC reject | TXT | _dmarc | v=DMARC1; p=reject; rua=mailto:dmarc@contoso.com | Full enforcement | After confidence |
| Intune discovery | CNAME | enterpriseenrollment | enterpriseenrollment.manage.microsoft.com | Device enrollment discovery | If Intune enrollment requires it |
| Intune registration | CNAME | enterpriseregistration | enterpriseregistration.windows.net | Device registration discovery | If Entra join/registration discovery requires it |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm admin tooling baseline exists | Admin workstation | Review workbook 02 output | Azure, Graph, CLI, and portal access are ready |
| 2 | Confirm target tenant | Admin workstation | Get-MgOrganization | Correct tenant display name and tenant ID are shown |
| 3 | Confirm current Azure context | Admin workstation | Get-AzContext | Correct tenant and subscription are selected |
| 4 | Confirm current Graph context | Admin workstation | Get-MgContext | Correct tenant and scopes are shown |
| 5 | Inventory current tenant domains | Admin workstation | Get-MgDomain -All | Existing domains are listed |
| 6 | Identify initial onmicrosoft.com domain | Admin workstation | Get-MgDomain -All \| Where-Object {$_.IsInitial -eq $true} | Initial tenant namespace is recorded |
| 7 | Identify default domain | Admin workstation | Get-MgDomain -All \| Where-Object {$_.IsDefault -eq $true} | Default tenant domain is recorded |
| 8 | Confirm domain is not already verified elsewhere | Admin workstation | Portal / DNS / tenant review | Domain conflict risk is documented |
| 9 | Confirm DNS provider and authoritative nameservers | Admin workstation | nslookup -type=ns domain.com | Correct DNS host is identified |
| 10 | Add custom domain to tenant | Admin workstation | New-MgDomain -Id "contoso.com" | Domain object is created in tenant |
| 11 | Retrieve verification DNS records | Admin workstation | Get-MgDomainVerificationDnsRecord -DomainId "contoso.com" | TXT or MX verification values are displayed |
| 12 | Create TXT verification record at DNS host | DNS provider / Azure DNS | Provider UI or New-AzDnsRecordSet | TXT record exists in public DNS zone |
| 13 | Validate public DNS propagation | Admin workstation | Resolve-DnsName -Name "contoso.com" -Type TXT | Verification TXT record is returned |
| 14 | Confirm domain in Microsoft Graph | Admin workstation | Confirm-MgDomain -DomainId "contoso.com" | Domain becomes verified |
| 15 | Validate domain verification status | Admin workstation | Get-MgDomain -DomainId "contoso.com" | IsVerified is True |
| 16 | Review service configuration records | Admin workstation | Get-MgDomainServiceConfigurationRecord -DomainId "contoso.com" | Recommended service records are displayed |
| 17 | Decide domain purpose | Admin workstation | Planning table | Sign-in, mail, or both are documented |
| 18 | Configure identity-only use if planned | Admin workstation | Portal or Graph inventory | Domain is available for UPNs |
| 19 | Do not change production MX unless mail cutover is approved | Admin workstation | Planning decision | Existing mail flow is protected |
| 20 | If Exchange use is planned, review accepted domain | Admin workstation | Get-AcceptedDomain | Domain appears or required Exchange steps are documented |
| 21 | If Exchange use is planned, configure SPF | DNS provider / Azure DNS | Add TXT SPF record | SPF record includes Microsoft 365 sending source |
| 22 | If Exchange use is planned, configure Autodiscover | DNS provider / Azure DNS | Add CNAME autodiscover | Autodiscover points to Microsoft 365 |
| 23 | If Exchange use is planned, configure DKIM CNAMEs | DNS provider / Azure DNS | Add selector CNAME records | DKIM CNAMEs exist |
| 24 | If Exchange use is planned, enable DKIM | Admin workstation | Enable-DkimSigningConfig | DKIM signing is enabled |
| 25 | If Exchange use is planned, configure DMARC monitor policy | DNS provider / Azure DNS | Add TXT _dmarc record | DMARC record exists with p=none |
| 26 | Validate all required DNS records | Admin workstation | Resolve-DnsName / nslookup | Expected records resolve publicly |
| 27 | Export final domain baseline | Admin workstation | Run export skeleton | Domain status and DNS records are captured |
| 28 | Document namespace decisions | Admin workstation | Workbook notes | Sign-in/mail/use cases are recorded |
| 29 | Save final baseline | Admin workstation | Commit note to workbook repo or Obsidian vault | Domain baseline is ready for later labs |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Precheck_Skeleton

```powershell
# ============================================================
# Custom Domain Precheck Skeleton
# ============================================================

$TenantId = "<tenant-id>"
$DomainName = "contoso.com"
$OutputRoot = ".\Custom_Domain_Baseline"
$DateStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputPath = Join-Path $OutputRoot $DateStamp

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

# Confirm Microsoft Graph context
Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "Domain.ReadWrite.All"

$MgContext = Get-MgContext

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Format-List

$MgContext |
    Select-Object Account, TenantId, ClientId, Scopes |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "MgContext.csv")

# Confirm tenant organization
$Organization = Get-MgOrganization

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

$Organization |
    Select-Object Id, DisplayName, TenantType |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Organization.csv")

# Inventory current tenant domains
$Domains = Get-MgDomain -All

$Domains |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsDefault -Descending, Id |
    Format-Table -AutoSize

$Domains |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsDefault -Descending, Id |
    Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "ExistingDomains.csv")

# Identify initial domain
$InitialDomain = $Domains | Where-Object {$_.IsInitial -eq $true}

$InitialDomain |
    Select-Object Id, IsVerified, IsDefault, IsInitial |
    Format-List

# Identify default domain
$DefaultDomain = $Domains | Where-Object {$_.IsDefault -eq $true}

$DefaultDomain |
    Select-Object Id, IsVerified, IsDefault, IsInitial |
    Format-List

# Confirm target domain does not already exist in this tenant
$ExistingTargetDomain = $Domains | Where-Object {$_.Id -eq $DomainName}

if ($ExistingTargetDomain) {
    Write-Warning "Domain already exists in this tenant:"
    $ExistingTargetDomain | Format-List
}
else {
    Write-Host "Domain does not exist in this tenant yet: $DomainName"
}

# Check public DNS nameservers
try {
    Resolve-DnsName -Name $DomainName -Type NS -ErrorAction Stop |
        Select-Object NameHost, Name, Type |
        Format-Table -AutoSize

    Resolve-DnsName -Name $DomainName -Type NS -ErrorAction Stop |
        Select-Object NameHost, Name, Type |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Public_NS_Records.csv")
}
catch {
    Write-Warning "Could not resolve NS records for $DomainName"
    $_.Exception.Message | Out-File -FilePath (Join-Path $OutputPath "Public_NS_Error.txt")
}

# Check existing TXT records
try {
    Resolve-DnsName -Name $DomainName -Type TXT -ErrorAction Stop |
        Select-Object Name, Type, Strings |
        Format-Table -AutoSize

    Resolve-DnsName -Name $DomainName -Type TXT -ErrorAction Stop |
        Select-Object Name, Type, Strings |
        Export-Csv -NoTypeInformation -Path (Join-Path $OutputPath "Existing_TXT_Records.csv")
}
catch {
    Write-Warning "Could not resolve TXT records for $DomainName"
    $_.Exception.Message | Out-File -FilePath (Join-Path $OutputPath "Existing_TXT_Error.txt")
}

Write-Host "Precheck complete. Output path: $OutputPath"
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Graph_Domain_Add_And_Verify_Skeleton

```powershell
# ============================================================
# Add and Verify Custom Domain with Microsoft Graph PowerShell
# ============================================================

$TenantId = "<tenant-id>"
$DomainName = "contoso.com"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "Domain.ReadWrite.All"

# Confirm target tenant
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

# Create domain object in tenant
# Run only after confirming domain name and tenant are correct
New-MgDomain -Id $DomainName

# Confirm domain exists
Get-MgDomain -DomainId $DomainName |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Format-List

# Retrieve verification DNS records
$VerificationRecords = Get-MgDomainVerificationDnsRecord -DomainId $DomainName

$VerificationRecords |
    Select-Object Id, RecordType, Label, Text, MailExchange, Preference, Ttl |
    Format-List

# Export verification records for DNS provider entry
$VerificationRecords |
    Select-Object Id, RecordType, Label, Text, MailExchange, Preference, Ttl |
    Export-Csv -NoTypeInformation -Path ".\Domain_Verification_Records_$($DomainName.Replace('.','_')).csv"

# After DNS record has been created and resolves publicly, confirm domain
Confirm-MgDomain -DomainId $DomainName

# Validate final status
Get-MgDomain -DomainId $DomainName |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Format-List

# Retrieve service configuration records after verification
Get-MgDomainServiceConfigurationRecord -DomainId $DomainName |
    Select-Object Id, RecordType, Label, Text, MailExchange, Preference, Ttl |
    Format-List
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Azure_DNS_Verification_Record_Skeleton

```powershell
# ============================================================
# Azure DNS Verification Record Skeleton
# Use only if the public DNS zone is hosted in Azure DNS
# ============================================================

$SubscriptionId = "<subscription-id>"
$DnsResourceGroupName = "rg-dns-prod"
$DnsZoneName = "contoso.com"

# Example TXT value from Microsoft verification record
$VerificationTxtValue = "MS=ms12345678"

Connect-AzAccount -Tenant "<tenant-id>"
Set-AzContext -SubscriptionId $SubscriptionId

# Confirm zone exists
Get-AzDnsZone -ResourceGroupName $DnsResourceGroupName -Name $DnsZoneName

# Create root TXT verification record
# In Azure DNS, the root record set name is "@"
$TxtRecord = New-AzDnsRecordConfig -Value $VerificationTxtValue

New-AzDnsRecordSet `
    -Name "@" `
    -RecordType TXT `
    -ZoneName $DnsZoneName `
    -ResourceGroupName $DnsResourceGroupName `
    -Ttl 3600 `
    -DnsRecords $TxtRecord

# Validate the TXT record in Azure DNS
Get-AzDnsRecordSet `
    -ZoneName $DnsZoneName `
    -ResourceGroupName $DnsResourceGroupName `
    -RecordType TXT `
    -Name "@"

# Validate public DNS resolution
Resolve-DnsName -Name $DnsZoneName -Type TXT

# If the TXT record set already exists, append the verification value carefully
$ExistingTxtSet = Get-AzDnsRecordSet `
    -ZoneName $DnsZoneName `
    -ResourceGroupName $DnsResourceGroupName `
    -RecordType TXT `
    -Name "@"

$ExistingTxtSet.Records.Add((New-AzDnsRecordConfig -Value $VerificationTxtValue))

Set-AzDnsRecordSet -RecordSet $ExistingTxtSet
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_External_DNS_Verification_Record_Skeleton

```text
# ============================================================
# External DNS Provider Verification Record Skeleton
# ============================================================

Domain:
contoso.com

Record type:
TXT

Host / Name:
@
or
contoso.com

Value:
MS=ms12345678

TTL:
3600
or
provider default

Validation commands:
nslookup -type=txt contoso.com
Resolve-DnsName -Name contoso.com -Type TXT

Expected result:
The TXT answer includes MS=ms12345678.

Notes:
Some DNS providers require @ for the root record.
Some DNS providers require the full domain name.
Some DNS providers automatically append the zone name.
Do not create contoso.com.contoso.com by accident.
Do not remove existing SPF, DMARC, or other TXT records.
If a TXT record already exists at root, add the Microsoft verification string as another TXT value.
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_DNS_Validation_Skeleton

```powershell
# ============================================================
# DNS Validation Skeleton
# ============================================================

$DomainName = "contoso.com"

# Nameserver validation
Resolve-DnsName -Name $DomainName -Type NS |
    Select-Object NameHost, Name, Type |
    Format-Table -AutoSize

# TXT validation
Resolve-DnsName -Name $DomainName -Type TXT |
    Select-Object Name, Type, Strings |
    Format-Table -AutoSize

# MX validation
Resolve-DnsName -Name $DomainName -Type MX |
    Select-Object Name, Type, NameExchange, Preference |
    Format-Table -AutoSize

# Autodiscover validation
Resolve-DnsName -Name "autodiscover.$DomainName" -Type CNAME |
    Select-Object Name, Type, NameHost |
    Format-Table -AutoSize

# DMARC validation
Resolve-DnsName -Name "_dmarc.$DomainName" -Type TXT |
    Select-Object Name, Type, Strings |
    Format-Table -AutoSize

# DKIM selector validation
Resolve-DnsName -Name "selector1._domainkey.$DomainName" -Type CNAME |
    Select-Object Name, Type, NameHost |
    Format-Table -AutoSize

Resolve-DnsName -Name "selector2._domainkey.$DomainName" -Type CNAME |
    Select-Object Name, Type, NameHost |
    Format-Table -AutoSize

# Legacy nslookup alternatives
nslookup -type=ns $DomainName
nslookup -type=txt $DomainName
nslookup -type=mx $DomainName
nslookup -type=txt "_dmarc.$DomainName"
nslookup -type=cname "autodiscover.$DomainName"
nslookup -type=cname "selector1._domainkey.$DomainName"
nslookup -type=cname "selector2._domainkey.$DomainName"
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Microsoft_365_Portal_Skeleton

```text
# ============================================================
# Microsoft 365 Admin Center Domain Verification
# ============================================================

Portal:
https://admin.microsoft.com

Navigation:
Settings > Domains > Add domain

Steps:
1. Sign in with the cloud admin account.
2. Confirm the tenant display name.
3. Go to Settings > Domains.
4. Select Add domain.
5. Enter the custom domain name.
6. Select Use this domain.
7. Copy the TXT verification record Microsoft provides.
8. Go to the authoritative DNS provider.
9. Add the TXT verification record.
10. Wait for DNS propagation.
11. Return to Microsoft 365 admin center.
12. Select Verify.
13. Choose domain services:
    - Exchange and Exchange Online Protection
    - Teams / Skype service records if shown
    - Intune / device records if shown
14. Do not update MX for production mail unless mail cutover is approved.
15. Save the domain status.

Expected result:
The domain appears as verified in Settings > Domains.

Important:
Verification proves ownership only.
It does not automatically change user sign-in names.
It does not automatically migrate mail.
It does not automatically change MX routing.
It does not remove the initial onmicrosoft.com domain.
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Entra_Portal_Skeleton

```text
# ============================================================
# Entra Admin Center Domain Review
# ============================================================

Portal:
https://entra.microsoft.com

Navigation:
Identity > Settings > Domain names

Steps:
1. Sign in with the cloud admin account.
2. Confirm the tenant ID.
3. Go to Identity > Settings > Domain names.
4. Review existing domains.
5. Identify:
   - Initial domain
   - Default domain
   - Verified custom domains
6. Add custom domain if not already added.
7. Copy DNS verification record.
8. Publish DNS verification record at authoritative DNS host.
9. Verify the domain.
10. Confirm IsVerified status.
11. Decide whether the domain should become the default domain for new users.

Expected result:
The custom domain is visible and verified in Entra.

Important:
Changing the default domain affects new object defaults.
It does not automatically rename existing users.
Existing UPN changes must be planned separately.
Hybrid users may be overwritten by on-premises directory sync.
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Exchange_Online_Readiness_Skeleton

```powershell
# ============================================================
# Exchange Online Domain Readiness Skeleton
# Use only if domain will support mail
# ============================================================

$AdminUpn = "<admin-upn>"
$DomainName = "contoso.com"

Connect-ExchangeOnline -UserPrincipalName $AdminUpn

# Review accepted domains
Get-AcceptedDomain |
    Select-Object Name, DomainName, DomainType, Default |
    Sort-Object DomainName |
    Format-Table -AutoSize

# Confirm target domain accepted status
Get-AcceptedDomain |
    Where-Object {$_.DomainName -eq $DomainName} |
    Select-Object Name, DomainName, DomainType, Default |
    Format-List

# Review remote domains
Get-RemoteDomain |
    Select-Object Name, DomainName, AutoReplyEnabled, AutoForwardEnabled |
    Format-Table -AutoSize

# Review DKIM status
Get-DkimSigningConfig |
    Where-Object {$_.Domain -eq $DomainName} |
    Select-Object Domain, Enabled, Status, Selector1CNAME, Selector2CNAME |
    Format-List

# If DKIM config does not exist and domain is ready for Exchange Online
# New-DkimSigningConfig -DomainName $DomainName -Enabled $false

# After selector CNAMEs exist in DNS
# Enable-DkimSigningConfig -Identity $DomainName

# Validate DKIM again
Get-DkimSigningConfig |
    Where-Object {$_.Domain -eq $DomainName} |
    Select-Object Domain, Enabled, Status, Selector1CNAME, Selector2CNAME |
    Format-List

Disconnect-ExchangeOnline -Confirm:$false
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_UPN_Readiness_Skeleton

```powershell
# ============================================================
# UPN Namespace Readiness Skeleton
# This is read-only unless update commands are explicitly uncommented
# ============================================================

$TenantId = "<tenant-id>"
$DomainName = "contoso.com"

Connect-MgGraph -TenantId $TenantId -Scopes `
    "Directory.Read.All", `
    "User.Read.All", `
    "User.ReadWrite.All"

# Confirm domain is verified before UPN use
Get-MgDomain -DomainId $DomainName |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Format-List

# Inventory users currently using onmicrosoft.com namespace
Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled |
    Where-Object {$_.UserPrincipalName -like "*.onmicrosoft.com"} |
    Select-Object DisplayName, UserPrincipalName, UserType, AccountEnabled, Id |
    Sort-Object UserPrincipalName |
    Format-Table -AutoSize

# Example staged UPN mapping report
$Users = Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,UserType,AccountEnabled

$UpnMapping = $Users |
    Where-Object {$_.UserPrincipalName -like "*.onmicrosoft.com" -and $_.UserType -eq "Member"} |
    Select-Object DisplayName,
        UserPrincipalName,
        @{Name="ProposedUPN";Expression={
            ($_.UserPrincipalName.Split("@")[0]) + "@$DomainName"
        }},
        AccountEnabled,
        Id

$UpnMapping |
    Format-Table -AutoSize

$UpnMapping |
    Export-Csv -NoTypeInformation -Path ".\Proposed_UPN_Mapping_$($DomainName.Replace('.','_')).csv"

# Do not bulk update UPNs without planning Conditional Access, MFA, apps, hybrid sync, and user communications

# Example single-user update pattern
# Update-MgUser -UserId "<user-object-id-or-upn>" -UserPrincipalName "user@contoso.com"
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Azure_CLI_Skeleton

```bash
# ============================================================
# Azure CLI and Graph REST Domain Inventory Skeleton
# ============================================================

TENANT_ID="<tenant-id>"
DOMAIN_NAME="contoso.com"

# Login to target tenant
az login --tenant "$TENANT_ID"

# Confirm account context
az account show --output table

# Get organization
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json

# List domains
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains" \
  --output table

# Get target domain
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains/$DOMAIN_NAME" \
  --output json

# Create domain object
# Run only after confirming tenant and domain are correct
az rest \
  --method POST \
  --url "https://graph.microsoft.com/v1.0/domains" \
  --headers "Content-Type=application/json" \
  --body "{\"id\":\"$DOMAIN_NAME\"}"

# Get verification DNS records
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains/$DOMAIN_NAME/verificationDnsRecords" \
  --output json

# After DNS record is published, verify domain
az rest \
  --method POST \
  --url "https://graph.microsoft.com/v1.0/domains/$DOMAIN_NAME/verify" \
  --headers "Content-Type=application/json" \
  --body "{}"

# Get service configuration records
az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains/$DOMAIN_NAME/serviceConfigurationRecords" \
  --output json
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Verification_Commands

```powershell
# ============================================================
# Verification Commands
# ============================================================

$TenantId = "<tenant-id>"
$DomainName = "contoso.com"

# Connect Graph
Connect-MgGraph -TenantId $TenantId -Scopes `
    "Organization.Read.All", `
    "Directory.Read.All", `
    "Domain.Read.All"

# Verify tenant
Get-MgOrganization |
    Select-Object Id, DisplayName, TenantType |
    Format-List

# Verify all tenant domains
Get-MgDomain -All |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Sort-Object IsDefault -Descending, Id |
    Format-Table -AutoSize

# Verify target domain
Get-MgDomain -DomainId $DomainName |
    Select-Object Id, IsVerified, IsDefault, IsInitial, SupportedServices |
    Format-List

# Verify verification DNS records
Get-MgDomainVerificationDnsRecord -DomainId $DomainName |
    Select-Object Id, RecordType, Label, Text, MailExchange, Preference, Ttl |
    Format-List

# Verify service configuration records
Get-MgDomainServiceConfigurationRecord -DomainId $DomainName |
    Select-Object Id, RecordType, Label, Text, MailExchange, Preference, Ttl |
    Format-List

# Public DNS verification
Resolve-DnsName -Name $DomainName -Type NS
Resolve-DnsName -Name $DomainName -Type TXT
Resolve-DnsName -Name $DomainName -Type MX

# Optional mail readiness verification
Resolve-DnsName -Name "autodiscover.$DomainName" -Type CNAME
Resolve-DnsName -Name "_dmarc.$DomainName" -Type TXT
Resolve-DnsName -Name "selector1._domainkey.$DomainName" -Type CNAME
Resolve-DnsName -Name "selector2._domainkey.$DomainName" -Type CNAME

# nslookup alternatives
nslookup -type=ns $DomainName
nslookup -type=txt $DomainName
nslookup -type=mx $DomainName
nslookup -type=txt "_dmarc.$DomainName"
nslookup -type=cname "autodiscover.$DomainName"
```

```bash
# ============================================================
# Azure CLI Verification Commands
# ============================================================

TENANT_ID="<tenant-id>"
DOMAIN_NAME="contoso.com"

az login --tenant "$TENANT_ID"

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/organization" \
  --output json

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains" \
  --output table

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains/$DOMAIN_NAME" \
  --output json

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains/$DOMAIN_NAME/verificationDnsRecords" \
  --output json

az rest \
  --method GET \
  --url "https://graph.microsoft.com/v1.0/domains/$DOMAIN_NAME/serviceConfigurationRecords" \
  --output json

nslookup -type=txt "$DOMAIN_NAME"
nslookup -type=mx "$DOMAIN_NAME"
nslookup -type=ns "$DOMAIN_NAME"
```

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---|---|---|---|---|
| 1 | Confirm no users were renamed | Admin workstation | Get-MgUser -All | Existing UPNs remain unchanged unless intentionally changed |
| 2 | Confirm no MX cutover occurred accidentally | Admin workstation | Resolve-DnsName -Name domain.com -Type MX | MX still points to intended mail provider |
| 3 | Remove verification TXT record if domain verification is abandoned | DNS provider / Azure DNS | Remove TXT record | Public DNS no longer contains verification value |
| 4 | Remove unverified domain from tenant if abandoned | Admin workstation | Remove-MgDomain -DomainId "contoso.com" | Domain object is removed if unused |
| 5 | Do not remove verified production domain if users/mail depend on it | Admin workstation | Review users and Exchange recipients first | Prevents sign-in and mail disruption |
| 6 | Disable DKIM only if it was enabled by mistake | Admin workstation | Disable-DkimSigningConfig -Identity "contoso.com" | DKIM signing disabled for that domain |
| 7 | Restore previous DMARC policy if changed | DNS provider / Azure DNS | Replace _dmarc TXT value | Mail policy returns to previous state |
| 8 | Restore previous SPF value if changed | DNS provider / Azure DNS | Replace root SPF TXT value | Sender authorization returns to previous state |
| 9 | Restore previous MX if changed by mistake | DNS provider / Azure DNS | Replace MX record | Inbound mail routes to previous provider |
| 10 | Disconnect sessions | Admin workstation | Disconnect-MgGraph; Disconnect-AzAccount; az logout | Admin sessions are closed |
| 11 | Remove sensitive exports if needed | Admin workstation | Remove-Item ".\Custom_Domain_Baseline" -Recurse -Force | Local evidence files are removed |

## Rollback Notes

Domain verification by itself is low risk.

Risk increases when you:

- Change user UPNs
- Change primary SMTP addresses
- Change Exchange accepted domain behavior
- Change MX records
- Change SPF records
- Enable DKIM
- Enforce DMARC
- Change production Autodiscover
- Change nameservers at the registrar
- Move a DNS zone between providers

Do not remove a verified domain until you confirm:

- No user UPN uses the domain
- No group email address uses the domain
- No mailbox primary SMTP or proxy address uses the domain
- No application redirect URI or publisher verification depends on the domain
- No Exchange accepted domain depends on it
- No SharePoint, Teams, or device enrollment workflow depends on it

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Failure_Checks

| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| Domain add fails | Domain already exists in tenant or another Microsoft tenant | Admin workstation | Get-MgDomain -All | Confirm ownership and remove from old tenant if appropriate |
| Domain verification fails | TXT record not published or wrong DNS host used | Admin workstation | Resolve-DnsName -Name domain.com -Type TXT | Add record at authoritative DNS provider |
| TXT record not visible publicly | DNS propagation delay or record created in wrong zone | Admin workstation | nslookup -type=ns domain.com | Confirm authoritative nameservers |
| TXT record value looks correct but still fails | Extra quotes, wrong host, duplicate zone suffix, or stale DNS | Admin workstation | Resolve-DnsName -Type TXT domain.com | Correct record name/value and wait for TTL |
| Provider created contoso.com.contoso.com | DNS UI appended zone automatically | DNS provider | nslookup -type=txt contoso.com.contoso.com | Recreate record at root using @ or blank host |
| Verification asks for MX instead of TXT | TXT unsupported or alternate method selected | Admin workstation | Get-MgDomainVerificationDnsRecord | Use TXT if possible, otherwise publish exact MX verification record |
| New domain not available for UPN | Domain not verified yet | Admin workstation | Get-MgDomain -DomainId domain.com | Verify domain first |
| User cannot sign in after UPN change | UPN changed without user communication, cached token issue, CA issue, or hybrid sync overwrite | Admin workstation | Get-MgUser -UserId user@domain.com | Validate UPN, communicate new sign-in, check CA and sync |
| Hybrid user UPN reverts | On-premises AD is source of authority | Domain controller / sync server | Check AD userPrincipalName and Entra Connect rules | Change UPN suffix on-premises and sync |
| MX changed and mail stopped | MX cutover done before Exchange readiness | Admin workstation | Resolve-DnsName -Type MX domain.com | Restore previous MX or complete Exchange cutover |
| SPF breaks third-party senders | SPF replaced instead of merged | Admin workstation | Resolve-DnsName -Type TXT domain.com | Merge all authorized senders into one SPF record |
| Multiple SPF records exist | More than one root TXT begins with v=spf1 | Admin workstation | Resolve-DnsName -Type TXT domain.com | Combine into a single SPF record |
| DKIM enable fails | Selector CNAMEs missing or wrong | Admin workstation | Resolve-DnsName selector1._domainkey.domain.com -Type CNAME | Publish exact selector records |
| DMARC reports missing | rua address invalid or mailbox not configured | Admin workstation | Resolve-DnsName _dmarc.domain.com -Type TXT | Correct DMARC rua and mailbox |
| Autodiscover fails | CNAME missing or conflicting A record exists | Admin workstation | Resolve-DnsName autodiscover.domain.com -Type CNAME | Remove conflict and add correct CNAME |
| Domain visible in M365 but not Entra portal | Portal cache or viewing wrong tenant | Admin workstation | Get-MgContext; Get-MgDomain -All | Switch tenant and refresh |
| Graph command fails with authorization error | Missing Graph scope or admin consent | Admin workstation | Get-MgContext | Reconnect with Domain.ReadWrite.All or proper scope |
| Remove domain fails | Domain still used by user, group, app, or service | Admin workstation | Search users/groups/proxy addresses | Remove dependencies before deleting domain |
| Azure DNS record creation fails | Wrong subscription, RG, or zone name | Admin workstation | Get-AzContext; Get-AzDnsZone | Set correct context and zone |
| DNS validation differs between machines | Resolver cache or split DNS | Admin workstation | Resolve-DnsName -Server 8.8.8.8 | Test public resolver and flush local cache |
| Domain verification succeeds but mail records incomplete | Verification only proves ownership | Admin workstation | Get-MgDomainServiceConfigurationRecord | Configure required service records |
| Break glass account impacted | Emergency account renamed or placed under custom domain | Admin workstation | Get-MgUser | Keep break glass on stable onmicrosoft.com domain unless design says otherwise |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Documentation_Output_TEMPLATE

## Tenant Namespace Baseline

| Field | Value |
|---|---|
| Tenant display name |  |
| Tenant ID |  |
| Initial onmicrosoft.com domain |  |
| Default domain before change |  |
| Default domain after change |  |
| Custom domain added |  |
| Domain verified | Yes / No |
| Verification method | TXT / MX |
| Verification date |  |
| Admin account used |  |

## DNS Ownership Baseline

| Field | Value |
|---|---|
| Domain registrar |  |
| DNS host |  |
| Authoritative nameservers |  |
| Azure DNS zone used | Yes / No |
| Azure DNS subscription |  |
| Azure DNS resource group |  |
| DNS TTL used |  |
| DNS change owner |  |

## Verification Record Baseline

| Record Type | Name / Host | Value | TTL | Status |
|---|---|---|---|---|
| TXT |  |  |  | Present / Missing |
| MX alternative |  |  |  | Present / Missing / Not used |

## Service Record Baseline

| Record | Type | Name / Host | Value | Status | Notes |
|---|---|---|---|---|---|
| MX | MX | @ |  | Present / Missing / Deferred |  |
| SPF | TXT | @ |  | Present / Missing / Deferred |  |
| Autodiscover | CNAME | autodiscover |  | Present / Missing / Deferred |  |
| DKIM selector 1 | CNAME | selector1._domainkey |  | Present / Missing / Deferred |  |
| DKIM selector 2 | CNAME | selector2._domainkey |  | Present / Missing / Deferred |  |
| DMARC | TXT | _dmarc |  | Present / Missing / Deferred |  |
| Intune enrollment | CNAME | enterpriseenrollment |  | Present / Missing / Deferred |  |
| Device registration | CNAME | enterpriseregistration |  | Present / Missing / Deferred |  |

## Namespace Usage Decision

| Use Case | Decision | Notes |
|---|---|---|
| User sign-in UPNs | Use / Do not use / Future |  |
| Exchange Online mail | Use / Do not use / Future |  |
| SharePoint / OneDrive sharing | Use / Do not use / Future |  |
| Teams identity | Use / Do not use / Future |  |
| Intune enrollment | Use / Do not use / Future |  |
| App registrations | Use / Do not use / Future |  |
| Hybrid identity | Use / Do not use / Future |  |
| Break glass accounts | Keep on onmicrosoft.com / Review |  |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories | Supplies tenant ID, initial domain, existing verified domains, and subscription context |
| 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline | Supplies Graph, Az, Azure CLI, and portal access needed for domain verification |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles | Required to understand Domain Name Administrator, Global Administrator, Exchange Administrator, and Azure DNS permissions |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline | Uses stable tenant namespace decisions for emergency accounts |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline | Uses verified domain for user UPNs and group naming |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking | Tracks domain verification, DNS changes, admin actions, and service health |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Uses this workbook to troubleshoot sign-in, DNS verification, UPN, and portal tenant issues |
| Exchange Online Administration Workbook | Extends accepted domain, MX, SPF, DKIM, DMARC, and mail routing work |
| SharePoint OneDrive Workbook | Extends tenant namespace and sharing behavior |
| Teams Administration Workbook | Extends identity and service namespace impact |
| Intune Workbook | Extends enrollment and device registration DNS records |
| Hybrid Identity Workbook | Extends AD UPN suffix, Entra Connect, cloud sync, and verified domain planning |
| Azure DNS Workbook | Extends public DNS zone hosting, record management, delegation, and validation |

---

# 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces_Completion_Criteria

| Requirement | Complete |
|---|---|
| Target tenant confirmed |  |
| Tenant ID documented |  |
| Initial onmicrosoft.com domain documented |  |
| Existing verified domains exported |  |
| New custom domain selected |  |
| Registrar identified |  |
| DNS host identified |  |
| Authoritative nameservers validated |  |
| Domain added to tenant |  |
| Verification DNS record retrieved |  |
| TXT verification record published |  |
| TXT record validated publicly |  |
| Domain verified in tenant |  |
| Domain verification status exported |  |
| Service configuration records reviewed |  |
| Domain purpose documented |  |
| UPN impact reviewed |  |
| Break glass impact reviewed |  |
| Hybrid identity impact reviewed |  |
| Exchange mail use decision documented |  |
| MX change deferred unless approved |  |
| SPF plan documented |  |
| DKIM plan documented |  |
| DMARC plan documented |  |
| Autodiscover plan documented |  |
| Intune DNS records deferred or documented |  |
| Rollback notes documented |  |
| Evidence output stored securely |  |

## Final Expected State

The admin can clearly state:

- This is the tenant where the custom domain was added.
- This is the initial onmicrosoft.com domain.
- This is the verified custom domain.
- This is the DNS provider that hosts the authoritative records.
- This is the verification record used.
- This is the domain verification status.
- This domain is approved for sign-in, mail, both, or future use.
- Mail routing was not changed unless explicitly planned.
- UPN changes were not performed unless separately planned.
- The domain namespace is ready for later user, licensing, Exchange, Teams, SharePoint, Intune, and hybrid identity workbooks.