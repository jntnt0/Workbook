# 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Index

### Purpose

This workbook provides a structured troubleshooting workflow for Microsoft 365 and cloud domain issues involving domain verification, DNS records, Exchange Online mail routing, MX records, TXT records, CNAME records, SPF, DKIM, DMARC, and Autodiscover.

### Scope

Use this workbook when a custom domain or mail domain has one or more of the following symptoms:

- Microsoft 365 domain verification fails
- TXT verification record is not found
- DNS record appears correct at the registrar but Microsoft 365 does not detect it
- MX record is missing, wrong, duplicated, or pointing to old mail provider
- Users cannot receive mail after domain cutover
- Outbound mail is failing SPF checks
- DKIM selector records are missing or invalid
- DKIM cannot be enabled in Defender portal or Exchange Online PowerShell
- DMARC is missing, malformed, too strict, or not reporting
- Autodiscover fails for Outlook desktop or mobile clients
- Outlook prompts repeatedly for credentials after domain change
- Exchange Online accepted domain exists but mail routing is broken
- Domain is verified but users cannot use it as a sign-in or mail domain
- Hybrid Exchange Autodiscover or mail flow conflicts with Microsoft 365 DNS

### Assumptions

- You have access to Microsoft 365 admin center, Exchange admin center, Microsoft Defender portal, and public DNS hosting.
- You can query DNS from at least one external resolver.
- You know where DNS is hosted for the domain.
- You are not changing registrar ownership unless explicitly required.
- All DNS changes are recorded in a change log.

### Required Admin Roles

| Task Area | Minimum Role |
|---|---|
| View Microsoft 365 domains | Global Reader, Domain Name Administrator, Global Administrator |
| Add or verify Microsoft 365 domain | Domain Name Administrator or Global Administrator |
| Manage Exchange accepted domains | Exchange Administrator |
| Review Exchange mail flow | Exchange Administrator |
| Manage DKIM | Exchange Administrator or Security Administrator depending portal path |
| Review Defender email authentication | Security Reader, Security Administrator |
| Modify DNS records | DNS provider admin, registrar admin, or delegated DNS zone admin |
| Review audit logs | Reports Reader, Global Reader |
| Perform emergency tenant domain recovery | Global Administrator |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Mental_Model

Domain and mail authentication troubleshooting has two sides:

1. Microsoft 365 tenant configuration.
2. Public DNS reality.

Do not trust what the DNS provider page shows by itself. Always query public DNS.

A Microsoft 365 domain works only when all required layers line up:

1. Domain is owned by the organization.
2. Domain is added to the correct Microsoft 365 tenant.
3. Verification TXT record is published publicly.
4. Microsoft 365 sees the TXT record through public DNS.
5. Domain becomes verified.
6. Domain purpose is configured for Exchange, Teams, Intune, Entra, or other workloads.
7. Exchange Online accepted domain exists.
8. MX record points to Exchange Online Protection if cloud mail receipt is intended.
9. SPF authorizes the intended outbound mail systems.
10. DKIM CNAME selector records point to Microsoft 365 selector targets.
11. DKIM signing is enabled for the domain.
12. DMARC policy aligns SPF and DKIM with the visible From domain.
13. Autodiscover points clients to the right Exchange endpoint.
14. Old DNS records from previous mail providers do not conflict.

Traditional troubleshooting order:

1. Confirm domain and tenant.
2. Confirm authoritative DNS.
3. Confirm public DNS propagation.
4. Confirm Microsoft 365 domain state.
5. Confirm Exchange accepted domain state.
6. Confirm MX and inbound flow.
7. Confirm SPF and outbound authentication.
8. Confirm DKIM CNAMEs and signing.
9. Confirm DMARC syntax and alignment.
10. Confirm Autodiscover.
11. Remove legacy conflicts only after evidence.

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Symptom_Map

| Symptom | Likely Cause | Primary Evidence | First Check |
|---|---|---|---|
| Domain verification fails | TXT missing, wrong value, wrong DNS zone, propagation delay, wrong tenant | Public TXT lookup | `nslookup -type=txt <domain>` |
| Microsoft 365 says record not found | Record added at wrong DNS host or parent/child zone mismatch | Authoritative NS lookup | `nslookup -type=ns <domain>` |
| Domain verified in wrong tenant | Domain already added elsewhere | M365 domain status error | Check tenant ownership and Microsoft support path |
| MX points to old provider | DNS not updated or old record has lower preference | MX lookup | `nslookup -type=mx <domain>` |
| Mail not received | MX wrong, accepted domain missing, recipient missing, connector issue | Message trace and MX lookup | Exchange accepted domain |
| SPF hard fail | SPF missing Microsoft 365 include or too many includes | TXT lookup | SPF TXT record |
| SPF permerror | More than 10 DNS lookups, malformed syntax, multiple SPF records | SPF validator / TXT lookup | Count SPF records |
| DKIM cannot enable | Selector CNAME missing or invalid | CNAME lookup | `selector1._domainkey` and `selector2._domainkey` |
| DKIM fails after enablement | DNS target wrong, selector mismatch, domain not signed | Message headers | DKIM selector records |
| DMARC fail | DKIM/SPF not aligned, strict policy, wrong reporting record | Message headers and TXT lookup | `_dmarc.<domain>` |
| Outlook Autodiscover fails | CNAME missing, points to old Exchange, SRV conflict, internal DNS split-brain | Autodiscover tests | `autodiscover.<domain>` CNAME |
| Outlook credential loop | Old Autodiscover, stale profile, cached credentials, tenant mismatch | Outlook test email autoconfig | Autodiscover result |
| Teams or Entra sign-in domain issue | Domain not verified or not added as UPN suffix | Entra domain list | M365 domain status |
| Hybrid mail flow breaks | MX, connectors, accepted domain, or Autodiscover conflict | Message trace and connector config | Exchange connectors |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Evidence_Collection

| Evidence | Where To Get It | Why It Matters |
|---|---|---|
| Domain name | User report / admin center | Defines troubleshooting target |
| Tenant name and tenant ID | Microsoft 365 admin center / Entra admin center | Prevents verifying in wrong tenant |
| DNS hosting provider | Registrar NS records | DNS must be changed at authoritative host |
| Registrar | WHOIS / registrar portal | Confirms ownership and delegation |
| Authoritative nameservers | Public NS lookup | Shows where records must exist |
| Microsoft 365 verification TXT value | Microsoft 365 admin center | Required to prove ownership |
| Current public TXT records | DNS lookup | Shows public reality |
| Current public MX records | DNS lookup | Determines inbound mail path |
| Current public SPF record | DNS lookup | Determines outbound authorization |
| Current DKIM selector CNAMEs | DNS lookup | Required for DKIM enablement |
| Current DMARC TXT record | DNS lookup | Determines authentication enforcement |
| Current Autodiscover record | DNS lookup | Determines Outlook discovery behavior |
| Exchange accepted domain state | Exchange admin center / PowerShell | Mail domain must exist in EXO |
| Message trace sample | Exchange admin center | Shows mail routing result |
| NDR headers | User-provided bounce | Shows recipient/routing/authentication error |
| Message headers | Delivered mail sample | Shows SPF/DKIM/DMARC alignment |
| Recent DNS changes | Change record / DNS provider logs | Finds cause of regression |
| TTL values | DNS lookup / DNS provider | Explains propagation delay |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Triage_Order

1. Confirm the exact domain.
2. Confirm the correct Microsoft 365 tenant.
3. Confirm where authoritative DNS is hosted.
4. Query public NS records.
5. Query current public TXT, MX, CNAME, SPF, DKIM, DMARC, and Autodiscover records.
6. Compare public DNS to Microsoft 365 required records.
7. Fix verification TXT first if domain is not verified.
8. Confirm Microsoft 365 domain status.
9. Confirm Exchange Online accepted domain.
10. Validate MX record and inbound mail routing.
11. Validate SPF syntax and includes.
12. Validate DKIM selector CNAMEs.
13. Enable or confirm DKIM signing.
14. Validate DMARC syntax, policy, and alignment.
15. Validate Autodiscover externally.
16. Check message trace and headers.
17. Remove legacy conflicting DNS records.
18. Record changes, TTL, verification output, and rollback plan.

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Planning_Table

| Item | Value |
|---|---|
| Domain | `<contoso.com>` |
| Tenant name | `<tenant-name>` |
| Tenant ID | `<tenant-id>` |
| DNS hosting provider | `<provider>` |
| Registrar | `<registrar>` |
| Authoritative nameservers | `<ns1, ns2>` |
| Microsoft 365 verification TXT | `<MS=ms########>` |
| Exchange Online MX target | `<domain-com.mail.protection.outlook.com>` |
| SPF target | `<v=spf1 include:spf.protection.outlook.com -all>` |
| DKIM selector 1 name | `selector1._domainkey.<domain>` |
| DKIM selector 2 name | `selector2._domainkey.<domain>` |
| DMARC record name | `_dmarc.<domain>` |
| Autodiscover record name | `autodiscover.<domain>` |
| Mail cutover date | `<yyyy-mm-dd>` |
| Current mail provider | `<Exchange Online / hybrid / third-party>` |
| Previous mail provider | `<provider>` |
| Hybrid Exchange present | `<yes-no>` |
| Third-party mail gateway present | `<yes-no>` |
| Change ticket | `<ticket-id>` |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Configuration_Checklist

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm domain spelling | Admin workstation | Review domain exactly as entered in Microsoft 365 | Not applicable | Domain has no typo, trailing space, or wrong suffix |
| 2 | Confirm correct Microsoft 365 tenant | Admin workstation | Microsoft 365 admin center > Settings > Domains | `Get-MgOrganization | Select-Object DisplayName,Id,VerifiedDomains` | Domain is being checked in correct tenant |
| 3 | Confirm domain status in Microsoft 365 | Admin workstation | Microsoft 365 admin center > Settings > Domains | `Get-MgDomain | Where-Object {$_.Id -eq "<domain>"}` | Domain is verified or pending verification |
| 4 | Identify authoritative nameservers | Admin workstation | `nslookup -type=ns <domain>` | `Resolve-DnsName <domain> -Type NS` | Nameservers identify real DNS host |
| 5 | Check public TXT records | Admin workstation | `nslookup -type=txt <domain>` | `Resolve-DnsName <domain> -Type TXT` | Verification TXT and SPF TXT are visible publicly |
| 6 | Check verification TXT value | Admin workstation | Compare public TXT to Microsoft 365 required value | `Resolve-DnsName <domain> -Type TXT | Select-Object -ExpandProperty Strings` | TXT contains exact `MS=ms########` value |
| 7 | Fix verification TXT if missing | DNS provider portal | Add TXT record at root host/name `@` | Not applicable | Public lookup returns Microsoft verification TXT |
| 8 | Re-run verification | Admin workstation | Microsoft 365 admin center > Domains > Verify | Not applicable | Domain changes to verified |
| 9 | Check for multiple SPF records | Admin workstation | `nslookup -type=txt <domain>` | `Resolve-DnsName <domain> -Type TXT | Where-Object {$_.Strings -match "v=spf1"}` | Only one SPF TXT record exists |
| 10 | Check MX record | Admin workstation | `nslookup -type=mx <domain>` | `Resolve-DnsName <domain> -Type MX` | MX points to intended inbound mail provider |
| 11 | Check Exchange Online accepted domain | Admin workstation | Exchange admin center > Mail flow > Accepted domains | `Get-AcceptedDomain | Where-Object {$_.DomainName -eq "<domain>"}` | Domain exists as accepted domain |
| 12 | Check accepted domain type | Admin workstation | Exchange admin center > Accepted domain details | `Get-AcceptedDomain <domain> | Select-Object DomainName,DomainType,Default` | DomainType matches Authoritative or InternalRelay design |
| 13 | Check recipient domain use | Admin workstation | Exchange admin center > Recipients | `Get-Recipient -ResultSize Unlimited | Where-Object {$_.EmailAddresses -like "*@<domain>"}` | Recipients have expected proxy addresses |
| 14 | Check inbound message trace | Admin workstation | Exchange admin center > Mail flow > Message trace | `Get-MessageTrace -RecipientAddress <user@domain> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Mail is accepted, rejected, deferred, or not reaching EXO |
| 15 | Check SPF syntax | Admin workstation | Review root TXT SPF | `Resolve-DnsName <domain> -Type TXT | Where-Object {$_.Strings -match "v=spf1"}` | SPF authorizes Microsoft 365 and any approved senders |
| 16 | Fix SPF for Microsoft 365 outbound only | DNS provider portal | Root TXT: `v=spf1 include:spf.protection.outlook.com -all` | Not applicable | SPF includes Microsoft 365 |
| 17 | Check DKIM configuration state | Admin workstation | Defender portal > Email & collaboration > Policies & rules > Threat policies > DKIM | `Get-DkimSigningConfig -Identity <domain>` | DKIM config exists and state is known |
| 18 | Check DKIM selector CNAMEs | Admin workstation | `nslookup -type=cname selector1._domainkey.<domain>` and `nslookup -type=cname selector2._domainkey.<domain>` | `Resolve-DnsName selector1._domainkey.<domain> -Type CNAME; Resolve-DnsName selector2._domainkey.<domain> -Type CNAME` | Both selectors resolve to Microsoft 365 DKIM targets |
| 19 | Fix missing DKIM CNAMEs | DNS provider portal | Add selector1 and selector2 CNAME records from Defender or EXO output | Not applicable | Public CNAME lookup resolves |
| 20 | Enable DKIM signing | Admin workstation | Defender portal > DKIM > select domain > Enable | `Set-DkimSigningConfig -Identity <domain> -Enabled $true` | DKIM enabled for domain |
| 21 | Check DMARC record | Admin workstation | `nslookup -type=txt _dmarc.<domain>` | `Resolve-DnsName _dmarc.<domain> -Type TXT` | DMARC TXT exists or absence is confirmed |
| 22 | Fix baseline DMARC monitoring record | DNS provider portal | `_dmarc` TXT: `v=DMARC1; p=none; rua=mailto:dmarc@<domain>; pct=100` | Not applicable | DMARC record resolves publicly |
| 23 | Validate Autodiscover DNS | Admin workstation | `nslookup -type=cname autodiscover.<domain>` | `Resolve-DnsName autodiscover.<domain> -Type CNAME` | Autodiscover points to intended endpoint |
| 24 | Fix Microsoft 365 Autodiscover | DNS provider portal | CNAME `autodiscover` to `autodiscover.outlook.com` | Not applicable | Autodiscover CNAME resolves correctly |
| 25 | Test Outlook Autodiscover externally | Admin workstation | Microsoft Remote Connectivity Analyzer or Outlook Test Email AutoConfiguration | Not applicable | Autodiscover reaches Exchange Online or intended hybrid endpoint |
| 26 | Check old mail provider conflicts | Admin workstation | Review MX, SPF includes, DKIM selectors, Autodiscover, SRV records | DNS provider portal and public lookups | Legacy records are identified |
| 27 | Remove stale records only after confirming cutover | DNS provider portal | Delete old MX/CNAME/TXT entries not required | Not applicable | Public DNS no longer points to obsolete provider |
| 28 | Verify inbound mail | Admin workstation | Send external test email to `<user@domain>` | `Get-MessageTrace -RecipientAddress <user@domain> -StartDate (Get-Date).AddHours(-1) -EndDate (Get-Date)` | Message is delivered |
| 29 | Verify outbound authentication | Admin workstation | Send test mail to external mailbox and inspect headers | Not applicable | SPF, DKIM, and DMARC pass or expected alignment result is known |
| 30 | Record final DNS state | Admin workstation | Save `nslookup` or `Resolve-DnsName` outputs | Use verification command section | Change record contains final proof |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Detailed_Workflow

### Phase 1: Confirm Domain, Tenant, And DNS Authority

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm domain name | Admin workstation | Ask for exact domain and copy from admin center | Not applicable | Domain is confirmed |
| 2 | Confirm tenant | Admin workstation | Microsoft 365 admin center > Settings > Org settings or Domains | `Get-MgOrganization | Select-Object DisplayName,Id` | Correct tenant is confirmed |
| 3 | Confirm domain exists in tenant | Admin workstation | Microsoft 365 admin center > Settings > Domains | `Get-MgDomain -DomainId <domain>` | Domain object exists or needs to be added |
| 4 | Confirm domain verification state | Admin workstation | Microsoft 365 admin center > Settings > Domains | `Get-MgDomain -DomainId <domain> | Select-Object Id,IsVerified,AuthenticationType,AvailabilityStatus` | Verification state is known |
| 5 | Find authoritative DNS | Admin workstation | `nslookup -type=ns <domain>` | `Resolve-DnsName <domain> -Type NS` | Authoritative nameservers are identified |
| 6 | Compare authoritative DNS to provider being edited | Admin workstation | Compare NS output to DNS provider portal | Not applicable | You are editing the correct DNS zone |
| 7 | Check parent/child zone issue | Admin workstation | Query NS for subdomain if troubleshooting subdomain | `Resolve-DnsName <subdomain.domain> -Type NS` | Delegated subdomain or parent zone behavior is understood |

### Phase 2: Domain Verification TXT

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Get required verification TXT | Admin workstation | Microsoft 365 admin center > Settings > Domains > setup incomplete domain | Not applicable | Required `MS=ms########` value is known |
| 2 | Query public TXT | Admin workstation | `nslookup -type=txt <domain>` | `Resolve-DnsName <domain> -Type TXT` | Current public TXT records are visible |
| 3 | Confirm exact TXT value | Admin workstation | Compare value character by character | `Resolve-DnsName <domain> -Type TXT | Select-Object -ExpandProperty Strings` | TXT matches Microsoft 365 requirement |
| 4 | Fix host/name error | DNS provider portal | Use host/name `@` or blank root depending provider | Not applicable | TXT is created at domain root, not `domain.domain` |
| 5 | Fix wrong DNS provider issue | Registrar/DNS provider portal | Update correct authoritative DNS zone | Not applicable | Public lookup changes after TTL |
| 6 | Wait for TTL only when public DNS still shows old value | Admin workstation | Query multiple resolvers if needed | `Resolve-DnsName <domain> -Type TXT -Server 8.8.8.8; Resolve-DnsName <domain> -Type TXT -Server 1.1.1.1` | Propagation status is understood |
| 7 | Retry verification | Admin workstation | Microsoft 365 admin center > Verify | Not applicable | Domain verifies successfully |

### Phase 3: MX And Inbound Mail Flow

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Query MX records | Admin workstation | `nslookup -type=mx <domain>` | `Resolve-DnsName <domain> -Type MX` | Current MX targets and preference values are visible |
| 2 | Confirm intended mail receiver | Admin workstation | Compare MX to design: Exchange Online, hybrid, or third-party gateway | Not applicable | Inbound path is known |
| 3 | Fix Exchange Online MX | DNS provider portal | MX target `<domain-com.mail.protection.outlook.com>` with appropriate priority | Not applicable | MX points to Exchange Online Protection |
| 4 | Remove old lower-priority MX conflict | DNS provider portal | Delete stale MX records after cutover approval | Not applicable | Only intended MX path remains |
| 5 | Confirm Exchange accepted domain | Admin workstation | Exchange admin center > Mail flow > Accepted domains | `Get-AcceptedDomain | Where-Object {$_.DomainName -eq "<domain>"}` | Domain exists in Exchange Online |
| 6 | Confirm domain type | Admin workstation | Accepted domain details | `Get-AcceptedDomain <domain> | Select-Object Name,DomainName,DomainType,Default` | Authoritative or InternalRelay matches design |
| 7 | Confirm recipient exists | Admin workstation | Exchange admin center > Recipients | `Get-Recipient <user@domain>` | Recipient has expected address |
| 8 | Run message trace | Admin workstation | Exchange admin center > Mail flow > Message trace | `Get-MessageTrace -RecipientAddress <user@domain> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Delivery state is known |
| 9 | Review NDR if sender receives bounce | Admin workstation | Copy NDR error and enhanced status code | Not applicable | Failure reason is known |
| 10 | Send external inbound test | External mailbox | Send test to user at custom domain | `Get-MessageTrace -RecipientAddress <user@domain> -StartDate (Get-Date).AddMinutes(-30) -EndDate (Get-Date)` | Test mail reaches Exchange Online or fails with traceable status |

### Phase 4: SPF

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Query TXT records | Admin workstation | `nslookup -type=txt <domain>` | `Resolve-DnsName <domain> -Type TXT` | SPF TXT is visible |
| 2 | Confirm only one SPF record | Admin workstation | Count records beginning with `v=spf1` | `Resolve-DnsName <domain> -Type TXT | Where-Object {$_.Strings -match "v=spf1"}` | Exactly one SPF record exists |
| 3 | Confirm Microsoft 365 include | Admin workstation | Inspect SPF string | Not applicable | SPF contains `include:spf.protection.outlook.com` if M365 sends mail |
| 4 | Confirm third-party senders are authorized | Admin workstation | Compare SPF includes/IPs to approved systems | Not applicable | SPF includes all approved senders |
| 5 | Check for lookup limit risk | Admin workstation | Count include, a, mx, exists, redirect mechanisms | Not applicable | SPF does not exceed 10 DNS lookups |
| 6 | Fix duplicate SPF | DNS provider portal | Merge required mechanisms into one TXT record | Not applicable | One valid SPF TXT remains |
| 7 | Start with soft policy if migrating | DNS provider portal | Use `~all` during staged migration if approved | Not applicable | SPF failures are observable before hard enforcement |
| 8 | Use hard fail after validation | DNS provider portal | End SPF with `-all` when all senders are known | Not applicable | Unauthorized senders fail SPF |
| 9 | Validate outbound test headers | External mailbox | Send mail and inspect headers | Not applicable | SPF result is pass for Microsoft 365 outbound |

### Phase 5: DKIM

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check DKIM config | Admin workstation | Defender portal > Email & collaboration > Policies & rules > Threat policies > DKIM | `Get-DkimSigningConfig -Identity <domain>` | DKIM config state is visible |
| 2 | Get selector CNAME requirements | Admin workstation | Defender portal DKIM page or EXO PowerShell output | `Get-DkimSigningConfig -Identity <domain> | Format-List Selector1CNAME,Selector2CNAME,Enabled,Status` | Required CNAME target values are known |
| 3 | Query selector1 | Admin workstation | `nslookup -type=cname selector1._domainkey.<domain>` | `Resolve-DnsName selector1._domainkey.<domain> -Type CNAME` | selector1 resolves to Microsoft target |
| 4 | Query selector2 | Admin workstation | `nslookup -type=cname selector2._domainkey.<domain>` | `Resolve-DnsName selector2._domainkey.<domain> -Type CNAME` | selector2 resolves to Microsoft target |
| 5 | Fix missing selector CNAMEs | DNS provider portal | Add selector1 and selector2 CNAMEs exactly as shown | Not applicable | Selectors resolve publicly |
| 6 | Retry DKIM enablement | Admin workstation | Defender portal > DKIM > Enable | `Set-DkimSigningConfig -Identity <domain> -Enabled $true` | DKIM enables successfully |
| 7 | Confirm DKIM enabled | Admin workstation | Defender portal DKIM page | `Get-DkimSigningConfig -Identity <domain> | Select-Object Domain,Enabled,Status` | Enabled is True |
| 8 | Send outbound test | Admin workstation | Send email to external mailbox | Not applicable | Message headers show DKIM pass |
| 9 | Validate selector in headers | External mailbox | Inspect `DKIM-Signature` header | Not applicable | Header selector matches Microsoft 365 DKIM selector |

### Phase 6: DMARC

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Query DMARC | Admin workstation | `nslookup -type=txt _dmarc.<domain>` | `Resolve-DnsName _dmarc.<domain> -Type TXT` | DMARC record is visible or confirmed absent |
| 2 | Confirm syntax starts correctly | Admin workstation | Inspect DMARC TXT | Not applicable | Record starts with `v=DMARC1` |
| 3 | Confirm policy | Admin workstation | Inspect `p=` value | Not applicable | Policy is `none`, `quarantine`, or `reject` |
| 4 | Confirm reporting address | Admin workstation | Inspect `rua=` value | Not applicable | Aggregate reports go to monitored mailbox or service |
| 5 | Start with monitoring policy | DNS provider portal | TXT `_dmarc`: `v=DMARC1; p=none; rua=mailto:dmarc@<domain>; pct=100` | Not applicable | DMARC reports can be collected without enforcement |
| 6 | Confirm SPF alignment | External mailbox | Inspect headers: SPF domain aligns with From domain | Not applicable | SPF alignment passes when intended |
| 7 | Confirm DKIM alignment | External mailbox | Inspect headers: DKIM d= aligns with From domain | Not applicable | DKIM alignment passes when intended |
| 8 | Move to quarantine only after pass rates are known | DNS provider portal | Change `p=none` to `p=quarantine` when approved | Not applicable | Failing mail is quarantined by receivers |
| 9 | Move to reject only after validation | DNS provider portal | Change `p=quarantine` to `p=reject` when approved | Not applicable | Spoofed or unauthenticated mail is rejected |
| 10 | Record DMARC posture | Admin workstation | Save TXT and header test results | Not applicable | Change log records current enforcement state |

### Phase 7: Autodiscover

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Query Autodiscover CNAME | Admin workstation | `nslookup -type=cname autodiscover.<domain>` | `Resolve-DnsName autodiscover.<domain> -Type CNAME` | Autodiscover target is visible |
| 2 | Confirm Microsoft 365 target | Admin workstation | Compare target to `autodiscover.outlook.com` | Not applicable | Cloud-only tenant points to Microsoft 365 |
| 3 | Check for A record conflict | Admin workstation | `nslookup autodiscover.<domain>` | `Resolve-DnsName autodiscover.<domain> -Type A` | No old A record overrides intended CNAME |
| 4 | Check SRV fallback records | Admin workstation | `nslookup -type=srv _autodiscover._tcp.<domain>` | `Resolve-DnsName _autodiscover._tcp.<domain> -Type SRV` | SRV record is absent or intentional |
| 5 | Fix cloud Autodiscover | DNS provider portal | CNAME `autodiscover` to `autodiscover.outlook.com` | Not applicable | Public CNAME resolves correctly |
| 6 | Check internal DNS split-brain | Domain-joined workstation | Query internal DNS for `autodiscover.<domain>` | `Resolve-DnsName autodiscover.<domain> -Server <internal-dns-ip>` | Internal DNS does not point clients to obsolete server |
| 7 | Test Outlook Autodiscover | Client workstation | Outlook tray icon > Ctrl+right-click > Test Email AutoConfiguration | Not applicable | Autodiscover returns correct EXO endpoint |
| 8 | Use Remote Connectivity Analyzer | Admin workstation | Run Outlook Autodiscover test | Not applicable | External Autodiscover result is clean |
| 9 | Recreate Outlook profile if DNS is fixed but client remains broken | Client workstation | Control Panel > Mail > Show Profiles > Add | Not applicable | New profile discovers mailbox correctly |
| 10 | Clear stale Windows credentials if repeated prompts continue | Client workstation | Credential Manager > Windows Credentials | Not applicable | Old credentials removed and auth succeeds |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Verification_Commands

### Public DNS Checks

| Purpose | Command |
|---|---|
| Check authoritative nameservers | `nslookup -type=ns <domain>` |
| Check root TXT records | `nslookup -type=txt <domain>` |
| Check MX records | `nslookup -type=mx <domain>` |
| Check Autodiscover CNAME | `nslookup -type=cname autodiscover.<domain>` |
| Check Autodiscover A record | `nslookup -type=a autodiscover.<domain>` |
| Check Autodiscover SRV | `nslookup -type=srv _autodiscover._tcp.<domain>` |
| Check DKIM selector1 | `nslookup -type=cname selector1._domainkey.<domain>` |
| Check DKIM selector2 | `nslookup -type=cname selector2._domainkey.<domain>` |
| Check DMARC | `nslookup -type=txt _dmarc.<domain>` |
| Query Google DNS for TXT | `nslookup -type=txt <domain> 8.8.8.8` |
| Query Cloudflare DNS for TXT | `nslookup -type=txt <domain> 1.1.1.1` |

### PowerShell DNS Checks

| Purpose | PowerShell |
|---|---|
| Check authoritative nameservers | `Resolve-DnsName <domain> -Type NS` |
| Check root TXT records | `Resolve-DnsName <domain> -Type TXT` |
| Check MX records | `Resolve-DnsName <domain> -Type MX` |
| Check SPF records only | `Resolve-DnsName <domain> -Type TXT | Where-Object {$_.Strings -match "v=spf1"}` |
| Check verification TXT | `Resolve-DnsName <domain> -Type TXT | Where-Object {$_.Strings -match "MS=ms"}` |
| Check Autodiscover CNAME | `Resolve-DnsName autodiscover.<domain> -Type CNAME` |
| Check DKIM selector1 | `Resolve-DnsName selector1._domainkey.<domain> -Type CNAME` |
| Check DKIM selector2 | `Resolve-DnsName selector2._domainkey.<domain> -Type CNAME` |
| Check DMARC | `Resolve-DnsName _dmarc.<domain> -Type TXT` |
| Check Google DNS TXT | `Resolve-DnsName <domain> -Type TXT -Server 8.8.8.8` |
| Check Cloudflare DNS TXT | `Resolve-DnsName <domain> -Type TXT -Server 1.1.1.1` |

### Microsoft Graph Domain Checks

| Purpose | PowerShell |
|---|---|
| Connect to Graph | `Connect-MgGraph -Scopes "Domain.Read.All","Organization.Read.All","Directory.Read.All"` |
| Confirm tenant | `Get-MgOrganization | Select-Object DisplayName,Id,VerifiedDomains` |
| List domains | `Get-MgDomain | Select-Object Id,IsVerified,IsDefault,AuthenticationType,AvailabilityStatus` |
| Check specific domain | `Get-MgDomain -DomainId <domain> | Format-List` |
| Disconnect Graph | `Disconnect-MgGraph` |

### Exchange Online Checks

| Purpose | PowerShell |
|---|---|
| Connect to Exchange Online | `Connect-ExchangeOnline` |
| Check accepted domain | `Get-AcceptedDomain <domain> | Format-List Name,DomainName,DomainType,Default` |
| List accepted domains | `Get-AcceptedDomain | Select-Object Name,DomainName,DomainType,Default` |
| Check DKIM config | `Get-DkimSigningConfig -Identity <domain> | Format-List` |
| Enable DKIM | `Set-DkimSigningConfig -Identity <domain> -Enabled $true` |
| Check recipient address | `Get-Recipient <user@domain> | Select-Object DisplayName,PrimarySmtpAddress,RecipientType` |
| Find recipients using domain | `Get-Recipient -ResultSize Unlimited | Where-Object {$_.EmailAddresses -like "*@<domain>"} | Select-Object DisplayName,PrimarySmtpAddress,RecipientType` |
| Run message trace | `Get-MessageTrace -RecipientAddress <user@domain> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` |
| Disconnect Exchange Online | `Disconnect-ExchangeOnline` |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Known_Good_Baselines

### Microsoft 365 Cloud-Only DNS Baseline

| Record Type | Host / Name | Value | Purpose |
|---|---|---|---|
| TXT | `@` | `MS=ms########` | Microsoft 365 domain verification |
| MX | `@` | `<domain-com.mail.protection.outlook.com>` | Inbound mail to Exchange Online Protection |
| TXT | `@` | `v=spf1 include:spf.protection.outlook.com -all` | SPF authorization for Exchange Online outbound |
| CNAME | `autodiscover` | `autodiscover.outlook.com` | Outlook Autodiscover |
| CNAME | `selector1._domainkey` | Microsoft 365 selector1 target | DKIM signing |
| CNAME | `selector2._domainkey` | Microsoft 365 selector2 target | DKIM signing |
| TXT | `_dmarc` | `v=DMARC1; p=none; rua=mailto:dmarc@<domain>; pct=100` | DMARC monitoring baseline |

### SPF Patterns

| Scenario | SPF Example | Notes |
|---|---|---|
| Exchange Online only | `v=spf1 include:spf.protection.outlook.com -all` | Clean baseline |
| Exchange Online plus third-party sender | `v=spf1 include:spf.protection.outlook.com include:<vendor-spf-domain> -all` | Confirm vendor include does not exceed lookup limit |
| Migration monitoring | `v=spf1 include:spf.protection.outlook.com ~all` | Softer fail during testing |
| Bad duplicate SPF | Two separate TXT records both starting `v=spf1` | Causes SPF permerror |
| Bad missing all | `v=spf1 include:spf.protection.outlook.com` | Incomplete policy |
| Bad typo | `v-spf1 include:spf.protection.outlook.com -all` | Invalid syntax |

### DMARC Patterns

| Scenario | DMARC Example | Notes |
|---|---|---|
| Monitoring only | `v=DMARC1; p=none; rua=mailto:dmarc@<domain>; pct=100` | Start here |
| Quarantine staged | `v=DMARC1; p=quarantine; rua=mailto:dmarc@<domain>; pct=25` | Stage enforcement |
| Reject enforced | `v=DMARC1; p=reject; rua=mailto:dmarc@<domain>; pct=100` | Use after sender inventory is clean |
| Strict alignment | `v=DMARC1; p=reject; adkim=s; aspf=s; rua=mailto:dmarc@<domain>` | Only after confirming all systems align |
| Bad syntax | `v=DMARC1 p=reject rua=mailto:dmarc@<domain>` | Missing semicolons |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Rollback

### Rollback Principles

- Lower TTL before planned DNS cutovers when possible.
- Capture pre-change DNS values before editing.
- Do not remove old provider DNS records until the new path is verified.
- Do not jump DMARC from none to reject without report review.
- Do not set SPF hard fail until all legitimate senders are inventoried.
- Do not point MX directly to Exchange Online if a required third-party gateway is still in path.
- Keep rollback focused on the failed change, not the whole domain.

### DNS Rollback Table

| Change Made | Rollback Action | Verification |
|---|---|---|
| Changed MX to Exchange Online | Restore previous MX target and priority if cutover fails | `nslookup -type=mx <domain>` |
| Removed old MX | Recreate old MX record from pre-change record | Public MX lookup shows old path |
| Changed SPF | Restore previous SPF TXT value | SPF TXT lookup returns previous value |
| Merged SPF records | Recreate prior SPF only if merged record breaks mail; avoid duplicate SPF long-term | SPF validates with one record |
| Added DKIM selectors | Remove only if wrong target was added; otherwise leave in place | Selector CNAMEs resolve correctly |
| Enabled DKIM | Disable DKIM only if signing causes confirmed issue | `Get-DkimSigningConfig -Identity <domain>` |
| Changed DMARC to quarantine/reject | Return to `p=none` during investigation | `_dmarc` TXT lookup shows monitoring policy |
| Changed Autodiscover to Microsoft 365 | Restore previous Autodiscover target if hybrid/on-prem design requires it | Autodiscover test follows intended endpoint |
| Deleted SRV Autodiscover | Recreate SRV only if required by design | SRV lookup resolves |
| Changed NS delegation | Restore previous NS records at registrar | NS lookup shows prior authoritative DNS |

### Pre-Change Capture Commands

| Purpose | Command |
|---|---|
| Capture NS | `nslookup -type=ns <domain>` |
| Capture TXT | `nslookup -type=txt <domain>` |
| Capture MX | `nslookup -type=mx <domain>` |
| Capture SPF | `nslookup -type=txt <domain>` |
| Capture DKIM selector1 | `nslookup -type=cname selector1._domainkey.<domain>` |
| Capture DKIM selector2 | `nslookup -type=cname selector2._domainkey.<domain>` |
| Capture DMARC | `nslookup -type=txt _dmarc.<domain>` |
| Capture Autodiscover | `nslookup -type=cname autodiscover.<domain>` |
| Capture SRV Autodiscover | `nslookup -type=srv _autodiscover._tcp.<domain>` |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Failure_Checks

| Failure | Check | Fix |
|---|---|---|
| Domain verification TXT not found | Wrong DNS provider or wrong host/name | Confirm authoritative NS and add TXT at root |
| TXT visible at provider but not public | Wrong zone, DNSSEC issue, provider delay, unsaved change | Query public DNS and provider audit/change status |
| Microsoft 365 still fails after TXT appears | Propagation, wrong tenant, wrong TXT value | Compare exact `MS=ms########` value and retry |
| Multiple SPF records | More than one TXT starts with `v=spf1` | Merge into one SPF record |
| SPF permerror | Too many DNS lookups or bad syntax | Flatten carefully or remove unnecessary includes |
| SPF fail for Microsoft 365 mail | Missing `include:spf.protection.outlook.com` | Add Microsoft 365 include |
| DKIM enable fails | Selector CNAME missing, wrong target, not propagated | Fix selector CNAMEs and retry |
| DKIM fails in headers | DKIM not enabled, wrong domain, third-party sending unsigned mail | Enable DKIM and validate sending source |
| DMARC fails while SPF passes | SPF domain not aligned with visible From domain | Use DKIM alignment or fix sender envelope/header configuration |
| DMARC fails while DKIM passes | DKIM d= domain not aligned or strict alignment blocks subdomain | Adjust sender DKIM domain or DMARC alignment policy |
| Inbound mail not received | MX points wrong, accepted domain missing, recipient missing | Fix MX, accepted domain, or recipient address |
| Mail routes to old provider | Old MX has lower priority or DNS cached | Remove old MX after cutover and wait TTL |
| Autodiscover points old server | Stale CNAME/A/SRV or internal DNS override | Fix public/internal Autodiscover |
| Outlook still fails after DNS fix | Cached profile or credentials | Create new Outlook profile and clear credentials |
| Hybrid Autodiscover breaks | Cloud-only Autodiscover applied to hybrid design | Restore hybrid Autodiscover target per design |
| Domain stuck in another tenant | Domain already verified elsewhere | Use tenant admin path or Microsoft support domain removal process |
| DNS changes not taking effect | Registrar points to different nameservers | Edit authoritative DNS provider or update delegation |
| DMARC reports not arriving | `rua` mailbox absent, external reporting authorization issue | Create mailbox or use reporting service instructions |
| DKIM CNAME host duplicated | Provider appends domain automatically | Use host `selector1._domainkey`, not full duplicated FQDN |
| MX target typo | Misspelled mail protection host | Copy exact Microsoft 365 MX target from domain setup page |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Escalation_Handoff

### Escalate To DNS Provider When

- Authoritative DNS does not publish saved records
- DNSSEC validation failures occur
- Registrar delegation does not match configured nameservers
- Provider UI shows records but public authoritative queries do not
- Zone file appears corrupted or locked

### Escalate To Microsoft 365 / Exchange Admin Team When

- Domain is verified but Exchange accepted domain is missing
- Mail flow fails after DNS is correct
- DKIM signing config is missing or cannot enable after correct CNAMEs
- Message trace shows unexpected Exchange Online processing
- Hybrid connector or accepted domain behavior is unclear

### Escalate To Security / Email Authentication Team When

- SPF, DKIM, or DMARC failures affect external deliverability
- DMARC reject policy blocks legitimate business mail
- Third-party senders are unknown or unmanaged
- Spoofing or phishing activity is suspected
- DKIM private key / signing source outside Microsoft 365 is involved

### Escalate To Microsoft Support When

- Domain is stuck in another tenant and no admin can release it
- Microsoft 365 verification fails despite public TXT being correct
- DKIM cannot enable despite correct selectors and propagation
- Exchange Online accepted domain state is inconsistent across portal and PowerShell
- Microsoft 365 domain service state appears corrupted

### Handoff Data To Include

| Field | Value |
|---|---|
| Incident ID | `<ticket-id>` |
| Domain | `<domain>` |
| Tenant ID | `<tenant-id>` |
| DNS provider | `<provider>` |
| Registrar | `<registrar>` |
| Authoritative NS | `<nameservers>` |
| Verification TXT expected | `<MS=ms########>` |
| Verification TXT observed | `<observed-value>` |
| MX observed | `<mx-output>` |
| SPF observed | `<spf-output>` |
| DKIM selector1 observed | `<selector1-output>` |
| DKIM selector2 observed | `<selector2-output>` |
| DMARC observed | `<dmarc-output>` |
| Autodiscover observed | `<autodiscover-output>` |
| Message trace ID | `<message-trace-id>` |
| NDR code | `<ndr-code>` |
| Recent DNS changes | `<changes>` |
| Current blocker | `<blocker>` |
| Business impact | `<impact>` |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Incident_Record_Template

| Field | Entry |
|---|---|
| Date opened | `<yyyy-mm-dd>` |
| Opened by | `<name>` |
| Domain | `<domain>` |
| Tenant | `<tenant-name>` |
| Symptom | `<symptom>` |
| Root cause category | `<Verification / MX / SPF / DKIM / DMARC / Autodiscover / Exchange / DNS provider / Hybrid>` |
| Root cause | `<confirmed-root-cause>` |
| Evidence | `<dns-output, screenshots, message trace, headers>` |
| DNS changes made | `<records changed>` |
| Microsoft 365 changes made | `<tenant or Exchange changes>` |
| Security impact | `<impact>` |
| Mail flow impact | `<impact>` |
| Rollback action | `<rollback>` |
| Verification performed | `<verification>` |
| Preventive action | `<preventive-action>` |
| Closed by | `<name>` |
| Date closed | `<yyyy-mm-dd>` |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Preventive_Controls

| Control | Target State |
|---|---|
| DNS ownership inventory | Registrar, DNS host, and admin contacts are documented |
| Tenant domain inventory | All custom domains are listed with purpose and owner |
| DNS change control | MX, SPF, DKIM, DMARC, and Autodiscover changes require ticket and rollback |
| TTL management | TTL is lowered before planned cutover |
| SPF hygiene | Exactly one SPF record per domain |
| DKIM | DKIM enabled for all sending domains |
| DMARC | DMARC monitoring enabled before enforcement |
| DMARC reporting | Reports route to monitored mailbox or platform |
| Third-party senders | All legitimate senders are inventoried |
| Mail cutover runbook | MX and Autodiscover changes follow a tested sequence |
| Hybrid documentation | Hybrid mail flow and Autodiscover design are documented |
| Header validation | Post-change header checks are required |
| DNS backup | Pre-change record values captured before edits |

---

## 02_Troubleshoot_Domain_Verification_DNS_MX_TXT_CNAME_SPF_DKIM_DMARC_And_Autodiscover_Related_Labs

| Lab                                        | Purpose                                       |
| ------------------------------------------ | --------------------------------------------- |
| Add and verify Microsoft 365 custom domain | Practice domain ownership verification        |
| Configure Exchange Online MX record        | Route inbound mail to Microsoft 365           |
| Configure Exchange Online accepted domain  | Validate mail domain inside Exchange Online   |
| Configure SPF for Exchange Online          | Authorize Microsoft 365 outbound mail         |
| Configure DKIM for custom domain           | Publish selectors and enable signing          |
| Configure DMARC monitoring                 | Start reporting without enforcement           |
| Enforce DMARC quarantine and reject        | Harden email authentication after validation  |
| Troubleshoot Autodiscover for Outlook      | Validate client discovery path                |
| Perform message trace in Exchange Online   | Confirm mail routing and delivery state       |
| Analyze message headers                    | Confirm SPF, DKIM, and DMARC results          |
| Document DNS rollback record               | Preserve pre-change and post-change DNS state |