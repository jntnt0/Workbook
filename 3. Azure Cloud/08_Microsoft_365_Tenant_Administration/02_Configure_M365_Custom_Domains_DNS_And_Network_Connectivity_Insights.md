02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights.md
# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Index
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights.md
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Source_Basis
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Mental_Model
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Planning_Table
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Configuration_Checklist
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Custom_Domain_Portal_Skeleton
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Graph_Domain_Skeleton
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_DNS_Records_Skeleton
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_MX_Cutover_Skeleton
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Network_Connectivity_Insights_Skeleton
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Verification_Commands
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Rollback
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Failure_Checks
02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Related_Labs

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add a custom domain to Microsoft 365 | Domain add workflow, Domain Connect, manual DNS, verification, and DNS cutover risk |
| Microsoft Learn | Microsoft 365 domains FAQ | MX priority, SPF TXT record behavior, DNS provider limitations, and common domain constraints |
| Microsoft Learn | Microsoft 365 network connectivity principles | Local egress, local DNS, endpoint optimization, proxy bypass, and connectivity design |
| Microsoft Learn | Network connectivity in Microsoft 365 admin center | Health > Network connectivity page, network assessments, insights, recommendations, and location collection |
| Microsoft Learn | Microsoft 365 admin center overview | Settings, Setup, Health, Reports, and admin center navigation |
| Microsoft Graph PowerShell | Domain cmdlets | Repeatable tenant domain inventory, verification record discovery, and domain confirmation |
| Exchange Online PowerShell | Accepted domains and mail flow validation | Mail domain acceptance and Exchange Online cutover checks |
| DNS operational practice | Public DNS validation | Verifying TXT, MX, CNAME, SRV, SPF, DKIM, DMARC, Autodiscover, and Teams/Skype records |
| Network operational practice | Client connectivity testing | Verifying DNS path, latency, proxy behavior, endpoint reachability, and admin center network insights |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Custom domain | Public DNS name verified into Microsoft 365 for sign-in, email, Teams, SharePoint, and service branding |
| Initial domain | The original `<tenant>.onmicrosoft.com` namespace created with the tenant |
| Domain ownership verification | Microsoft 365 proves you control the public DNS zone by requiring a TXT record |
| Domain Connect | Registrar-supported method where Microsoft 365 can add required DNS records automatically |
| Manual DNS | Admin manually creates DNS records at the DNS hosting provider |
| Registrar | Company where the domain was purchased |
| DNS host | Provider authoritative for the domain DNS zone; may differ from registrar |
| Authoritative nameservers | DNS servers that own the live public DNS records for the domain |
| TXT verification record | Temporary Microsoft 365 record used to prove domain ownership |
| MX record | Mail exchanger record that controls where inbound mail for the domain is delivered |
| SPF record | TXT record that lists allowed mail senders for the domain |
| CNAME record | Alias record often used for Autodiscover, device registration, and service endpoints |
| SRV record | Service locator record used by some Microsoft communication services |
| Autodiscover | Exchange client discovery record used by Outlook and mail clients |
| Cutover | Moment where production DNS records, especially MX, move service traffic to Microsoft 365 |
| Split DNS | Internal DNS and public DNS use the same namespace but may return different answers |
| Local DNS egress | Microsoft 365 client DNS queries should resolve near the user location when possible |
| Local internet egress | Microsoft 365 traffic should exit close to the user, not hairpin through distant networks |
| Network connectivity insights | Microsoft 365 admin center health feature that measures and reports network quality by location |
| Network assessment | Score and diagnostics showing tenant Microsoft 365 network connectivity quality |
| First rule | Verify the domain before moving production services |
| Blunt rule | Do not change MX records until mailbox readiness and rollback are understood |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Microsoft 365 tenant | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Custom domain | `contoso.com` | `<custom-domain>` |
| DNS host | `Cloudflare` | `<dns-host>` |
| Registrar | `Network Solutions` | `<registrar>` |
| Authoritative nameservers | `ns1.example-dns.com`, `ns2.example-dns.com` | `<authoritative-nameservers>` |
| Admin account | `admin@contoso.onmicrosoft.com` | `<admin-upn>` |
| Required role | Domain Name Administrator or Global Administrator | `<role-name>` |
| Existing mail provider | `Google Workspace`, `Exchange on-prem`, `IMAP host` | `<current-mail-provider>` |
| Existing MX destination | `mail.contoso.com` | `<current-mx-target>` |
| Microsoft 365 MX target | `contoso-com.mail.protection.outlook.com` | `<m365-mx-target>` |
| SPF current value | `v=spf1 include:_spf.example.com -all` | `<current-spf>` |
| SPF target value | `v=spf1 include:spf.protection.outlook.com -all` | `<target-spf>` |
| Autodiscover target | `autodiscover.outlook.com` | `<autodiscover-target>` |
| TTL before cutover | `300` | `<dns-ttl>` |
| DNS change window | `Friday 8 PM` | `<change-window>` |
| Mail cutover owner | `Exchange Admin` | `<mail-owner>` |
| DNS rollback owner | `DNS Admin` | `<dns-owner>` |
| Network test location | `HQ`, `Branch01`, `Remote user` | `<test-location>` |
| Client public IP | `203.0.113.10` | `<client-public-ip>` |
| Proxy/firewall path | `Direct`, `Proxy`, `VPN`, `Secure Web Gateway` | `<egress-path>` |
| Evidence path | `.\Evidence\M365-Domain-Network` | `<evidence-path>` |
| Rollback standard | Restore prior DNS records | `<rollback-plan>` |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm admin account | Admin Workstation | Portal account menu | Correct admin UPN is signed in |
| 2 | Confirm tenant | Admin Workstation | Portal tenant switcher | Correct Microsoft 365 tenant is active |
| 3 | Confirm required admin role | Admin Workstation | Microsoft 365 admin center > Roles | Admin has required domain permissions |
| 4 | Confirm custom domain ownership owner | Admin Workstation | Notes | DNS/registrar owner is known |
| 5 | Identify authoritative nameservers | Admin Workstation | `nslookup -type=NS <custom-domain>` | Authoritative DNS host is known |
| 6 | Inventory current public DNS records | Admin Workstation | `nslookup -type=TXT <custom-domain>` | Current TXT records are documented |
| 7 | Inventory current MX records | Admin Workstation | `nslookup -type=MX <custom-domain>` | Current mail routing is documented |
| 8 | Inventory current Autodiscover record | Admin Workstation | `nslookup autodiscover.<custom-domain>` | Current Autodiscover target is documented |
| 9 | Lower DNS TTL before cutover if needed | DNS Provider Portal | Set relevant records to `300` seconds | DNS cutover and rollback are faster |
| 10 | Open Microsoft 365 domains page | Admin Workstation | Admin center > Settings > Domains | Domain management page opens |
| 11 | Start add domain workflow | Admin Workstation | Settings > Domains > Add domain | Domain wizard starts |
| 12 | Enter custom domain | Admin Workstation | `<custom-domain>` | Domain is accepted for verification workflow |
| 13 | Choose Domain Connect if supported | Admin Workstation | Domain wizard > Sign in to DNS provider | DNS records can be created automatically |
| 14 | Choose manual DNS if Domain Connect is unavailable | Admin Workstation | Domain wizard > Manual setup | TXT verification record is displayed |
| 15 | Copy TXT verification record | Admin Workstation | Portal-provided TXT value | Verification record value is captured |
| 16 | Create TXT verification record | DNS Provider Portal | TXT name/value from Microsoft 365 | Public DNS contains verification record |
| 17 | Validate TXT publication externally | Admin Workstation | `nslookup -type=TXT <custom-domain> 8.8.8.8` | TXT verification value is visible |
| 18 | Verify domain in Microsoft 365 | Admin Workstation | Domain wizard > Verify | Domain shows as verified |
| 19 | Decide whether to add DNS records now | Admin Workstation | Domain wizard | Admin chooses service DNS records intentionally |
| 20 | Add Exchange Online records only when mail cutover is approved | DNS Provider Portal | MX, SPF, Autodiscover CNAME | Mail-related records are created intentionally |
| 21 | Add Teams/Skype records if required by tenant services | DNS Provider Portal | CNAME/SRV records from wizard | Communication service DNS records exist if needed |
| 22 | Add device management records if required | DNS Provider Portal | CNAME records from wizard | Device/service discovery records exist if needed |
| 23 | Do not change MX early | DNS Provider Portal | Notes | Existing mail flow remains stable until cutover |
| 24 | Confirm domain state in Microsoft 365 | Admin Workstation | Settings > Domains | Domain shows healthy or lists remaining DNS issues |
| 25 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "Domain.ReadWrite.All","Organization.Read.All"` | Graph connects with domain permissions |
| 26 | Verify domain inventory with Graph | Admin Workstation | `Get-MgDomain` | Custom domain is listed |
| 27 | Check verification DNS records with Graph | Admin Workstation | `Get-MgDomainVerificationDnsRecord -DomainId "<custom-domain>"` | Verification record details are visible |
| 28 | Confirm Exchange accepted domain after mail records | Admin Workstation | `Get-AcceptedDomain` | Domain is accepted in Exchange Online if mail enabled |
| 29 | Verify MX record after mail cutover | Admin Workstation | `nslookup -type=MX <custom-domain> 8.8.8.8` | MX points to Microsoft 365 target |
| 30 | Verify SPF after mail cutover | Admin Workstation | `nslookup -type=TXT <custom-domain> 8.8.8.8` | SPF includes `spf.protection.outlook.com` |
| 31 | Verify Autodiscover | Admin Workstation | `nslookup autodiscover.<custom-domain> 8.8.8.8` | Autodiscover points to Microsoft 365 target |
| 32 | Validate Outlook connectivity path | Test Client | Open Outlook / create profile | Outlook discovers Exchange Online |
| 33 | Validate inbound mail after MX cutover | External Mailbox | Send test message to `<test-user>@<custom-domain>` | Message arrives in Exchange Online mailbox |
| 34 | Validate outbound mail after SPF update | Microsoft 365 Mailbox | Send test message to external mailbox | Message is delivered and not marked spoofed |
| 35 | Open network connectivity page | Admin Workstation | Admin center > Health > Network connectivity | Network connectivity dashboard opens |
| 36 | Review tenant network assessment | Admin Workstation | Health > Network connectivity | Network score and locations are visible |
| 37 | Review detected office locations | Admin Workstation | Network connectivity page | Locations are listed or missing |
| 38 | Add or correct office locations if required | Admin Workstation | Network connectivity location settings | Locations reflect real branch/HQ sites |
| 39 | Run Microsoft 365 connectivity test from client | Test Client | `https://connectivity.office.com` | Client-side test report is generated |
| 40 | Review local DNS behavior | Test Client | `nslookup outlook.office.com` | DNS resolution path is visible |
| 41 | Review local egress IP | Test Client | Browser public IP check or connectivity test | Public egress IP matches expected site |
| 42 | Check for VPN or proxy hairpinning | Test Client / Firewall | Compare public IP and site location | Microsoft 365 traffic exits near the user |
| 43 | Validate Microsoft 365 endpoints are not SSL-broken unnecessarily | Firewall / Proxy | Proxy policy review | Optimized endpoints are not damaged by inspection |
| 44 | Review admin center network insights | Admin Workstation | Health > Network connectivity > Recommendations | Recommendations are documented |
| 45 | Document all DNS records created | Operator | Notes / screenshots | DNS change record is complete |
| 46 | Document all network findings | Operator | Notes / screenshots | Network baseline is complete |
| 47 | Export tenant domain state | Admin Workstation | Graph export skeleton below | Domain state evidence is saved |
| 48 | Export DNS lookup outputs | Admin Workstation | `nslookup` outputs to files | DNS evidence is saved |
| 49 | Record rollback values | Operator | Prior DNS record table | Old DNS values are available |
| 50 | Document completion state | Operator | Record domain, DNS provider, records, MX cutover status, and network insight status | Domain and network baseline is supportable |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Custom_Domain_Portal_Skeleton
```text
Portal path:

Microsoft 365 admin center
Settings
Domains
Add domain
```

| Step | Task | Expected Result |
|---:|---|---|
| 1 | Open Settings > Domains | Domains page opens |
| 2 | Select Add domain | Add domain wizard starts |
| 3 | Enter `<custom-domain>` | Microsoft 365 accepts the domain for verification |
| 4 | Choose Domain Connect if supported | Microsoft 365 signs in to DNS provider and can add records |
| 5 | Choose manual setup if Domain Connect is unavailable | Microsoft 365 displays TXT verification record |
| 6 | Create TXT verification record at DNS host | Verification TXT record publishes publicly |
| 7 | Select Verify | Domain ownership is verified |
| 8 | Choose which service records to add | Only intended service records are created |
| 9 | Avoid MX cutover until mail readiness is confirmed | Existing mail flow is protected |
| 10 | Finish wizard | Domain appears in Microsoft 365 Domains page |

Manual verification record capture:

| Field | Value |
|---|---|
| Domain | `<custom-domain>` |
| TXT record name | `<txt-record-name>` |
| TXT record value | `<txt-record-value>` |
| TTL | `<ttl>` |
| DNS host | `<dns-host>` |
| Verification date | `<date>` |
| Verified by | `<admin-upn>` |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Graph_Domain_Skeleton
```powershell
# Run from an admin workstation.
# Purpose: inventory, add, and verify Microsoft 365 custom domain state with Microsoft Graph PowerShell.

$Domain = "<custom-domain>"
$EvidencePath = ".\Evidence\M365-Domain-Network"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Domain.ReadWrite.All","Organization.Read.All","Directory.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context.txt"

# Inventory current tenant domains.
Get-MgDomain |
    Sort-Object Id |
    Format-Table Id,IsDefault,IsInitial,IsVerified,AuthenticationType -AutoSize |
    Tee-Object "$EvidencePath\mg-domains-before.txt"

# Add domain if not already present.
Get-MgDomain -DomainId $Domain -ErrorAction SilentlyContinue

# If the domain does not exist, create it.
New-MgDomain -Id $Domain

# Retrieve verification DNS records.
Get-MgDomainVerificationDnsRecord -DomainId $Domain |
    Format-List * |
    Tee-Object "$EvidencePath\domain-verification-records.txt"

# After creating the DNS TXT verification record at the DNS host, confirm the domain.
Confirm-MgDomain -DomainId $Domain

# Verify final domain state.
Get-MgDomain -DomainId $Domain |
    Format-List * |
    Tee-Object "$EvidencePath\mg-domain-final.txt"
```

Expected output:

| Check | Expected Result |
|---|---|
| `Get-MgDomain` | Existing tenant domains are listed |
| `New-MgDomain` | Domain object is created if it did not already exist |
| `Get-MgDomainVerificationDnsRecord` | TXT verification record details are returned |
| `Confirm-MgDomain` | Domain verification completes after public TXT record exists |
| Final `Get-MgDomain` | `IsVerified` is true |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_DNS_Records_Skeleton
```powershell
# Run from any workstation with public DNS access.
# Purpose: collect public DNS evidence before and after Microsoft 365 domain configuration.

$Domain = "<custom-domain>"
$DnsServer = "8.8.8.8"
$EvidencePath = ".\Evidence\M365-Domain-Network"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Nameserver authority.
nslookup -type=NS $Domain $DnsServer | Tee-Object "$EvidencePath\ns-records.txt"

# TXT records, including Microsoft 365 verification and SPF.
nslookup -type=TXT $Domain $DnsServer | Tee-Object "$EvidencePath\txt-records.txt"

# MX records for inbound mail routing.
nslookup -type=MX $Domain $DnsServer | Tee-Object "$EvidencePath\mx-records.txt"

# Autodiscover.
nslookup "autodiscover.$Domain" $DnsServer | Tee-Object "$EvidencePath\autodiscover-cname.txt"

# Common Microsoft 365 discovery CNAME checks.
nslookup "enterpriseenrollment.$Domain" $DnsServer | Tee-Object "$EvidencePath\enterpriseenrollment-cname.txt"
nslookup "enterpriseregistration.$Domain" $DnsServer | Tee-Object "$EvidencePath\enterpriseregistration-cname.txt"
```

DNS record capture table:

| Record Purpose | Type | Name | Target / Value | TTL | Status |
|---|---|---|---|---:|---|
| Domain verification | TXT | `<custom-domain>` | `<ms-verification-value>` | `<ttl>` | `<planned/created/verified>` |
| Inbound mail | MX | `<custom-domain>` | `<m365-mx-target>` | `<ttl>` | `<planned/created/verified>` |
| SPF | TXT | `<custom-domain>` | `v=spf1 include:spf.protection.outlook.com -all` | `<ttl>` | `<planned/created/verified>` |
| Autodiscover | CNAME | `autodiscover` | `autodiscover.outlook.com` | `<ttl>` | `<planned/created/verified>` |
| Device registration | CNAME | `enterpriseregistration` | `<m365-target>` | `<ttl>` | `<planned/created/verified>` |
| Device enrollment | CNAME | `enterpriseenrollment` | `<m365-target>` | `<ttl>` | `<planned/created/verified>` |
| Teams/Skype service | SRV | `<srv-name>` | `<srv-target>` | `<ttl>` | `<planned/created/verified>` |
| DKIM selector 1 | CNAME | `selector1._domainkey` | `<dkim-target>` | `<ttl>` | `<planned/created/verified>` |
| DKIM selector 2 | CNAME | `selector2._domainkey` | `<dkim-target>` | `<ttl>` | `<planned/created/verified>` |
| DMARC | TXT | `_dmarc` | `v=DMARC1; p=none; rua=mailto:<dmarc-mailbox>` | `<ttl>` | `<planned/created/verified>` |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_MX_Cutover_Skeleton
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm domain is verified | Admin Workstation | `Get-MgDomain -DomainId "<custom-domain>"` | Domain shows verified |
| 2 | Confirm users/mailboxes are ready | Admin Workstation | Exchange admin center / `Get-EXOMailbox` | Target mailboxes exist |
| 3 | Confirm accepted domain exists | Admin Workstation | `Get-AcceptedDomain` | Domain is accepted in Exchange Online |
| 4 | Record current MX record | Admin Workstation | `nslookup -type=MX <custom-domain>` | Prior MX target is documented |
| 5 | Record current SPF record | Admin Workstation | `nslookup -type=TXT <custom-domain>` | Prior SPF is documented |
| 6 | Lower MX TTL before cutover | DNS Provider Portal | Set TTL to `300` if supported | Faster cutover and rollback |
| 7 | Change MX to Microsoft 365 target | DNS Provider Portal | Set MX to `<m365-mx-target>` | Inbound mail begins routing to Microsoft 365 |
| 8 | Update SPF | DNS Provider Portal | Add/include `spf.protection.outlook.com` | Microsoft 365 is authorized to send mail |
| 9 | Add Autodiscover CNAME | DNS Provider Portal | `autodiscover` to `autodiscover.outlook.com` | Outlook discovery uses Microsoft 365 |
| 10 | Validate public MX | Admin Workstation | `nslookup -type=MX <custom-domain> 8.8.8.8` | MX resolves to Microsoft 365 |
| 11 | Validate public SPF | Admin Workstation | `nslookup -type=TXT <custom-domain> 8.8.8.8` | SPF includes Microsoft 365 |
| 12 | Send inbound test mail | External Mailbox | Send to `<test-user>@<custom-domain>` | Test mailbox receives inbound mail |
| 13 | Send outbound test mail | Microsoft 365 Mailbox | Send to external recipient | External recipient receives mail |
| 14 | Validate Outlook profile | Test Client | Open Outlook / create profile | Autodiscover completes |
| 15 | Monitor message trace | Admin Workstation | Exchange admin center > Message trace | Message flow is visible |

Exchange Online commands:

```powershell
# Run from an admin workstation with ExchangeOnlineManagement installed.
# Purpose: validate Exchange Online domain and mail readiness.

$Domain = "<custom-domain>"

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-AcceptedDomain |
    Sort-Object DomainName |
    Format-Table Name,DomainName,DomainType,Default -AutoSize

Get-AcceptedDomain |
    Where-Object {$_.DomainName -eq $Domain} |
    Format-List *

Get-EXOMailbox -ResultSize 20 |
    Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
    Format-Table -AutoSize
```

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Network_Connectivity_Insights_Skeleton
```text
Portal path:

Microsoft 365 admin center
Health
Network connectivity
```

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open Network connectivity | Admin Workstation | Admin center > Health > Network connectivity | Dashboard opens |
| 2 | Review tenant network assessment | Admin Workstation | Portal dashboard | Network score is visible |
| 3 | Review detected locations | Admin Workstation | Portal locations list | Office locations are visible or missing |
| 4 | Add missing office location if required | Admin Workstation | Network connectivity > Locations | Location is added |
| 5 | Review recommendations | Admin Workstation | Network connectivity > Recommendations | Network insights are visible |
| 6 | Run browser-based connectivity test | Test Client | `https://connectivity.office.com` | Client test report is generated |
| 7 | Record public egress IP | Test Client | Connectivity test / public IP check | Egress IP is documented |
| 8 | Record DNS resolver behavior | Test Client | `nslookup outlook.office.com` | DNS resolution path is documented |
| 9 | Identify proxy or VPN hairpin | Test Client / Firewall | Compare egress IP to client location | Traffic path is understood |
| 10 | Review Microsoft 365 optimized endpoint handling | Firewall / Proxy | Firewall/proxy policy review | Optimized endpoints avoid unnecessary inspection |
| 11 | Document network findings | Operator | Notes / screenshots | Network baseline is recorded |

Client-side capture:

```powershell
# Run from a test client at each major location.
# Purpose: capture DNS and route hints for Microsoft 365 connectivity baseline.

$EvidencePath = ".\Evidence\M365-Network-Connectivity"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

ipconfig /all | Tee-Object "$EvidencePath\ipconfig-all.txt"

nslookup outlook.office.com | Tee-Object "$EvidencePath\nslookup-outlook-office.txt"
nslookup teams.microsoft.com | Tee-Object "$EvidencePath\nslookup-teams.txt"
nslookup login.microsoftonline.com | Tee-Object "$EvidencePath\nslookup-login.txt"

tracert outlook.office.com | Tee-Object "$EvidencePath\tracert-outlook-office.txt"
tracert teams.microsoft.com | Tee-Object "$EvidencePath\tracert-teams.txt"
tracert login.microsoftonline.com | Tee-Object "$EvidencePath\tracert-login.txt"
```

Network finding capture table:

| Location | Client IP | Public Egress IP | DNS Resolver | Proxy/VPN Path | Assessment Finding | Action |
|---|---|---|---|---|---|---|
| `<location>` | `<client-ip>` | `<public-ip>` | `<dns-resolver>` | `<direct/proxy/vpn>` | `<finding>` | `<action>` |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Verification_Commands
```powershell
# Microsoft Graph domain verification.

$Domain = "<custom-domain>"

Connect-MgGraph -Scopes "Domain.ReadWrite.All","Organization.Read.All","Directory.Read.All"

Get-MgDomain |
    Sort-Object Id |
    Format-Table Id,IsDefault,IsInitial,IsVerified,AuthenticationType -AutoSize

Get-MgDomain -DomainId $Domain |
    Format-List *

Get-MgDomainVerificationDnsRecord -DomainId $Domain |
    Format-List *
```

```powershell
# Public DNS verification.

$Domain = "<custom-domain>"
$DnsServer = "8.8.8.8"

nslookup -type=NS $Domain $DnsServer
nslookup -type=TXT $Domain $DnsServer
nslookup -type=MX $Domain $DnsServer
nslookup "autodiscover.$Domain" $DnsServer
nslookup "enterpriseenrollment.$Domain" $DnsServer
nslookup "enterpriseregistration.$Domain" $DnsServer
```

```powershell
# Exchange Online mail domain validation.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-AcceptedDomain |
    Sort-Object DomainName |
    Format-Table Name,DomainName,DomainType,Default -AutoSize

Get-AcceptedDomain |
    Where-Object {$_.DomainName -eq "<custom-domain>"} |
    Format-List *
```

```powershell
# Client network connectivity baseline.

ipconfig /all
nslookup outlook.office.com
nslookup teams.microsoft.com
nslookup login.microsoftonline.com
tracert outlook.office.com
tracert teams.microsoft.com
tracert login.microsoftonline.com
```

| Validation Area | PowerShell / Command | Good Result |
|---|---|---|
| Domain listed in tenant | `Get-MgDomain` | Custom domain appears |
| Domain verified | `Get-MgDomain -DomainId "<custom-domain>"` | `IsVerified` is true |
| Verification TXT record visible | `nslookup -type=TXT <custom-domain> 8.8.8.8` | Microsoft verification TXT appears before verification |
| MX record correct after cutover | `nslookup -type=MX <custom-domain> 8.8.8.8` | MX points to Microsoft 365 |
| SPF record correct after cutover | `nslookup -type=TXT <custom-domain> 8.8.8.8` | SPF includes Microsoft 365 sender |
| Autodiscover correct | `nslookup autodiscover.<custom-domain> 8.8.8.8` | Autodiscover points to Microsoft 365 |
| Accepted domain exists | `Get-AcceptedDomain` | Custom domain appears in Exchange Online |
| Inbound mail works | External mail test | Message arrives in Microsoft 365 mailbox |
| Outbound mail works | Microsoft 365 mail test | External mailbox receives message |
| Network connectivity page opens | Admin center > Health > Network connectivity | Network assessment loads |
| Connectivity test works | `https://connectivity.office.com` | Report generates |
| DNS path known | `nslookup outlook.office.com` | Resolver and answer are visible |
| Egress path known | Connectivity test / public IP check | Public IP maps to expected site |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify changed DNS records | Operator | Compare DNS evidence files | Changed records are known |
| 2 | Restore previous MX record if mail cutover failed | DNS Provider Portal | Set MX back to `<previous-mx-target>` | Inbound mail returns to prior provider |
| 3 | Restore previous SPF record if outbound mail failed | DNS Provider Portal | Set TXT SPF back to `<previous-spf>` | Prior sender authorization is restored |
| 4 | Restore previous Autodiscover record if client discovery failed | DNS Provider Portal | Set Autodiscover back to `<previous-target>` | Client discovery returns to prior provider |
| 5 | Remove incorrect CNAME records | DNS Provider Portal | Delete incorrect record | Bad service discovery records are removed |
| 6 | Remove incorrect SRV records | DNS Provider Portal | Delete incorrect SRV | Bad communication records are removed |
| 7 | Restore prior TTL values | DNS Provider Portal | Set TTL back to standard | DNS posture returns to normal |
| 8 | Verify public DNS rollback | Admin Workstation | `nslookup -type=MX <custom-domain> 8.8.8.8` | Previous MX is visible |
| 9 | Verify prior SPF | Admin Workstation | `nslookup -type=TXT <custom-domain> 8.8.8.8` | Previous SPF is visible |
| 10 | Verify prior Autodiscover | Admin Workstation | `nslookup autodiscover.<custom-domain> 8.8.8.8` | Previous target is visible |
| 11 | Confirm mail flow to prior provider | External Mailbox | Send inbound test | Prior mail provider receives mail |
| 12 | Document rollback | Operator | Notes | Rollback time, owner, and validation are recorded |

Rollback values:

| Record | Previous Value | Microsoft 365 Value | Rollback Needed |
|---|---|---|---|
| MX | `<previous-mx-target>` | `<m365-mx-target>` | `<yes-no>` |
| SPF | `<previous-spf>` | `<m365-spf>` | `<yes-no>` |
| Autodiscover | `<previous-autodiscover>` | `autodiscover.outlook.com` | `<yes-no>` |
| TXT verification | `<none-or-previous>` | `<ms-verification-value>` | `<yes-no>` |
| CNAME | `<previous-cname>` | `<m365-target>` | `<yes-no>` |
| SRV | `<previous-srv>` | `<m365-target>` | `<yes-no>` |

Domain removal note:

```text
Do not remove a verified custom domain from Microsoft 365 unless all users, groups, aliases, mailboxes, and service references have been moved off the domain.
For a failed DNS cutover, rollback DNS first.
Removing the domain object is usually not the correct immediate rollback.
```

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Domain verification fails | `nslookup -type=TXT <custom-domain> 8.8.8.8` | TXT record missing, wrong value, wrong DNS host, or propagation delay |
| TXT record not visible publicly | `nslookup -type=NS <custom-domain>` | Record was added at registrar but DNS is hosted somewhere else |
| Domain already exists in another tenant | Microsoft 365 domain wizard | Domain is attached to another Microsoft 365 tenant |
| Domain wizard shows DNS errors | Settings > Domains > Check health | Required DNS records are missing or wrong |
| MX still points to old provider | `nslookup -type=MX <custom-domain> 8.8.8.8` | MX was not changed, wrong DNS zone was edited, or TTL/cache delay exists |
| Inbound mail still goes to old provider | Message trace / MX lookup | Public MX still old or sender is caching old MX |
| Inbound mail bounces after cutover | Exchange message trace | Mailboxes, accepted domain, licensing, or recipient objects are not ready |
| Outbound mail marked as spam | `nslookup -type=TXT <custom-domain>` | SPF missing Microsoft 365 or DKIM/DMARC not configured |
| SPF has multiple TXT records | `nslookup -type=TXT <custom-domain>` | More than one SPF record exists; combine into one SPF record |
| Outlook profile does not auto-configure | `nslookup autodiscover.<custom-domain>` | Autodiscover CNAME missing, wrong, or blocked by old record |
| Teams or Skype records fail | `nslookup -type=SRV <srv-record>` | SRV record missing, wrong priority, wrong target, or wrong name |
| Device enrollment discovery fails | `nslookup enterpriseenrollment.<custom-domain>` | Device management CNAME missing or wrong |
| Graph domain cmdlets fail | `Get-MgContext` | Missing Graph scopes or admin consent |
| `Confirm-MgDomain` fails | `Get-MgDomainVerificationDnsRecord` | DNS verification record is not publicly visible |
| Exchange accepted domain missing | `Get-AcceptedDomain` | Domain not enabled for Exchange or service provisioning not complete |
| Admin center network page missing | Admin center > Health | Role, license, or tenant feature availability issue |
| Network score is poor | Network connectivity recommendations | Hairpin VPN, proxy inspection, distant DNS, or distant egress |
| Branch users see worse performance | Connectivity test from branch | Traffic exits through HQ or distant proxy instead of local internet |
| Connectivity test location is wrong | `https://connectivity.office.com` | Public IP geolocation, proxy egress, or NAT path is misleading |
| Microsoft 365 endpoint blocked | Firewall/proxy logs | Required Microsoft 365 endpoints are blocked or inspected incorrectly |
| DNS results differ inside and outside | Internal DNS vs public DNS lookups | Split DNS mismatch |
| External DNS correct but internal clients fail | `nslookup <record> <internal-dns>` | Internal DNS zone lacks matching Microsoft 365 records |
| Rollback does not appear immediately | `nslookup <record> 8.8.8.8` | TTL/cache delay or wrong DNS host edited |

# 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights_Related_Labs
| Lab                                                                                        | Relationship                                                                          |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings              | Tenant baseline must exist before domain and DNS changes                              |
| 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup                     | Uses Health and Message center after domain/network configuration                     |
| 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests                        | Users can receive UPNs and email addresses using the verified custom domain           |
| 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups | Groups and shared mailboxes can use the verified custom domain                        |
| 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage                     | Domain users need licensing for Exchange, Teams, OneDrive, and apps                   |
| 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation                               | Domain and DNS administration requires appropriate delegated admin roles              |
| 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues              | Troubleshoots failed verification, DNS, licensing, role, and service health issues    |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces                         | Cloud foundation workbook for custom domain concepts                                  |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365                                   | Hybrid identity bridge workbook for domain DNS validation                             |
| Windows Server DNS suite                                                                   | Supports split DNS, internal DNS resolution, delegation, records, and troubleshooting |